# System Design Template + URL Shortener Worked Example

---

## 1. The 8-Step System Design Template

Use this structure for every HLD interview. Spend the first 5-10 minutes on steps 1-3.

---

### Step 1: Clarify Requirements (5 min)

Ask functional + non-functional requirements before designing anything.

**Functional (what it does):**
- Core features (write, then read, then admin)
- What's out of scope?

**Non-functional (how well it does it):**
- Scale: DAU, QPS, storage
- Latency: p99 read/write SLA
- Availability: 99.9% vs 99.99% (8.7h vs 52min downtime/year)
- Consistency: strong vs eventual
- Durability: data loss tolerance

---

### Step 2: Capacity Estimation (3 min)

Back-of-envelope math. Orders of magnitude are enough.

```
Given: 100M DAU, 10 reads/day per user, 1 write/day per 10 users
  Read QPS  = 100M × 10 / 86400 ≈ 11,500 reads/sec
  Write QPS = 10M / 86400 ≈ 115 writes/sec
  Peak (10x) = 115K reads/sec, 1150 writes/sec

Storage: 1 write = 1KB, 10M writes/day
  Daily storage = 10M × 1KB = 10GB/day
  5-year storage = 10GB × 365 × 5 = 18TB
```

---

### Step 3: Define API (2 min)

```
POST /shorten        { long_url: "...", custom_alias: "..." }  → { short_url: "..." }
GET  /{short_code}   → 301/302 redirect to long URL
DELETE /{short_code} → 200/404
GET  /stats/{short_code} → { clicks: N, ... }
```

---

### Step 4: High-Level Architecture

Draw the main components and data flow:
```
Client → Load Balancer → API Servers → Cache (Redis) → DB (PostgreSQL)
                                    ↘ Analytics Queue (Kafka) → Analytics DB
```

---

### Step 5: Data Model

```sql
CREATE TABLE urls (
    short_code VARCHAR(8) PRIMARY KEY,
    long_url   TEXT NOT NULL,
    user_id    BIGINT,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    click_count BIGINT DEFAULT 0
);
CREATE INDEX idx_urls_user ON urls(user_id);
```

---

### Step 6: Deep Dive Key Components

Pick the hardest 2-3 problems and design them in detail. The interviewer often guides this.

---

### Step 7: Handle Scale

- Read replicas for heavy read load
- Caching hot URLs
- Sharding if write volume too high
- CDN for redirect responses (cacheable with 301)

---

### Step 8: Failure Scenarios & Trade-offs

- What if cache goes down? (fall-through to DB)
- What if DB goes down? (read replicas + queued writes)
- Single points of failure? (HA deployment, health checks)
- Trade-offs you made and why

---

## 2. URL Shortener — Full Worked Example

### Requirements
**Functional:** Shorten URLs, redirect, optional custom alias, optional expiry, click stats.
**Non-functional:** 100M DAU, 1 write per 10 users = 10M writes/day, reads 10x. 100ms p99 redirect. 99.99% availability.

### Capacity
```
Read QPS  ≈ 100M × 10 / 86400 ≈ 11,600/s   (peak: ~100K/s)
Write QPS ≈ 10M / 86400 ≈ 115/s
Storage   ≈ 500 bytes/record × 10M/day × 5yr = 9TB
```

### Short Code Generation

**Option A — Counter + Base62 encoding:**
```python
CHARS = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

def encode_base62(num: int) -> str:
    if num == 0: return CHARS[0]
    result = []
    while num:
        result.append(CHARS[num % 62])
        num //= 62
    return ''.join(reversed(result))

# 7-char Base62 = 62^7 ≈ 3.5 trillion unique codes
```

Use a distributed counter (Zookeeper, Redis INCR) for unique IDs.

**Option B — Hash (MD5/SHA256) + take first 7 chars:**
- Risk of collision (1/62^7 probability per pair)
- Retry with salt if collision detected

**Chosen: Option A** — predictable, no collision risk, sortable.

### Architecture

```
Client
  ↓
CDN (cache 301 redirects for popular URLs)
  ↓
Load Balancer (AWS ALB)
  ↓
API Servers (stateless, horizontally scaled)
  │
  ├── Write path:
  │     ID Generator → DB write → Cache write → return short_code
  │
  ├── Read path:
  │     Check L1 cache (local) → L2 Redis → DB
  │     → 301 redirect (browser caches) or 302 (track every click)
  │
  └── Analytics:
        Click event → Kafka → Spark → Analytics DB (ClickHouse)
        Stats API reads from ClickHouse

ID Generator: Snowflake-based (Twitter Snowflake):
  64-bit = timestamp(41) + datacenter(5) + machine(5) + sequence(12)
  → 4096 unique IDs per millisecond per machine, globally unique, sortable
```

### Database Schema

```sql
-- Primary store
CREATE TABLE urls (
    id         BIGINT PRIMARY KEY,          -- Snowflake ID
    short_code VARCHAR(8) UNIQUE NOT NULL,  -- Base62(id)
    long_url   TEXT NOT NULL,
    user_id    BIGINT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Click tracking (high write)
CREATE TABLE clicks (
    short_code VARCHAR(8),
    clicked_at TIMESTAMPTZ DEFAULT NOW(),
    ip_hash    VARCHAR(64),
    user_agent TEXT,
    country    VARCHAR(3)
) PARTITION BY RANGE (clicked_at);  -- partition by month
```

### Caching Strategy

```
Cache-aside with Redis:
  Key: "url:{short_code}"
  Value: long_url (string)
  TTL: 24h (refresh on hit)
  
  On write: cache.set("url:{short_code}", long_url, 86400)
  On read:  long_url = cache.get("url:{short_code}") or db_lookup()
  On delete: cache.delete("url:{short_code}")
  
Cache size: Top 20% of URLs handle 80% of traffic (Zipf's law)
  10M active URLs × 500 bytes = 5GB → fits in Redis
```

### Custom Alias Flow

```
POST /shorten { long_url, custom_alias: "mylink" }
  → Check if "mylink" exists in DB (index on short_code)
  → If taken: 409 Conflict
  → If free: insert with custom short_code
```

### Expiry

```
expires_at field in DB.
Two approaches:
  1. Lazy deletion: check expires_at on read; return 410 Gone if expired
  2. Active deletion: background job sweeps expired rows daily

Use approach 1 (lazy) + approach 2 (periodic cleanup for storage recovery)
```

### Availability & Failure

```
API Servers: 3+ instances, auto-scaling, health checks
Redis: Redis Cluster (3 masters + 3 replicas), read from replicas
DB: PostgreSQL with 1 primary + 2 read replicas
  Failover: automatic with Patroni or RDS Multi-AZ

What if Redis is down?
  → Fall through to DB (slower but functional)
  → Circuit breaker prevents cascade

What if DB primary fails?
  → Read replicas still serve reads
  → Writes queued or return 503 briefly (failover in ~30s with Patroni)
```

### Trade-offs Made

| Decision | Alternative | Why chosen |
|----------|-------------|-----------|
| 301 redirect (permanent) | 302 (temporary) | 301 cached by browser = fewer server hits; 302 needed for accurate analytics |
| Counter-based ID | Hash-based | No collisions, simpler, sortable |
| Redis for cache | Memcached | Redis supports persistence + richer data types |
| SQL for primary | NoSQL | Schema is well-defined, relations are simple, ACID needed |
