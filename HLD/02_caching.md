# Caching

> Cache = fast, small storage layer between the application and slower data source. Goal: serve repeated reads from cache (sub-ms) instead of DB (10-100ms).

---

## 1. Cache Benefits & Costs

**Benefits:** Reduced latency, reduced DB load, higher throughput, can serve traffic during DB downtime.

**Costs:** Cache invalidation complexity, stale data risk, additional infrastructure, cold start problem.

> "There are only two hard things in CS: cache invalidation and naming things." — Phil Karlton

---

## 2. Cache Write Strategies

### Write-Through
```
Write → Cache AND DB simultaneously
Pros: No stale data in cache
Cons: Higher write latency (wait for both)
Use: Read-heavy workloads where freshness matters
```

### Write-Back (Write-Behind)
```
Write → Cache immediately, write to DB asynchronously (later)
Pros: Low write latency
Cons: Risk of data loss if cache fails before async write
Use: Write-heavy workloads where some data loss is acceptable
```

### Write-Around
```
Write → DB directly (bypass cache)
Cache populated only on read (lazy)
Pros: Cache not polluted with write-once data
Cons: First read after write is a cache miss
Use: Bulk imports, infrequently read data
```

---

## 3. Cache Read Strategies

### Cache-Aside (Lazy Loading) — Most Common
```
App → Check cache
      HIT:  Return cached value
      MISS: Read from DB, store in cache, return value

Code:
  value = cache.get(key)
  if not value:
      value = db.query(key)
      cache.set(key, value, ttl=300)
  return value
```

**Pros:** Only cache what's actually needed. Resilient to cache failure (fall through to DB).
**Cons:** Cache miss = 3 trips (cache + DB + cache write). Stale data possible (TTL-based expiry).

### Read-Through
```
App → Cache
      HIT:  Cache returns value
      MISS: Cache itself fetches from DB, caches, returns
App never talks to DB directly
```

---

## 4. Cache Eviction Policies

| Policy | Description | Use Case |
|--------|-------------|----------|
| **LRU** (Least Recently Used) | Evict item not accessed for longest time | General purpose |
| **LFU** (Least Frequently Used) | Evict item accessed least often | Popularity-based access |
| **FIFO** | Evict oldest item | Simple queues |
| **TTL** (Time To Live) | Evict after fixed duration | Time-sensitive data |
| **Random** | Evict random item | Approximates LRU cheaply |

**Redis default:** Configurable (`allkeys-lru`, `volatile-lru`, `allkeys-random`, etc.)

---

## 5. Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Strings, Hash, List, Set, ZSet, Stream, Bitmap, HLL | Strings only |
| Persistence | ✅ RDB snapshots, AOF log | ❌ |
| Replication | ✅ Primary-replica | ❌ |
| Clustering | ✅ Redis Cluster (hash slots) | ✅ Client-side sharding |
| Pub/Sub | ✅ | ❌ |
| Lua scripting | ✅ | ❌ |
| Atomic operations | ✅ INCR, MULTI/EXEC | Partial |
| Use for | Sessions, rate limiting, pub/sub, leaderboards, queues | Pure high-speed cache |

**Choose Redis** almost always unless you need Memcached's simpler multi-threaded model for pure string caching at extreme scale.

---

## 6. Cache Stampede (Thundering Herd)

**Problem:** Key expires → many concurrent requests all miss cache simultaneously → all hit DB at once → DB overload.

**Solutions:**
1. **Mutex lock:** Only one thread fetches; others wait.
2. **Probabilistic early expiry (PER):** Start refreshing slightly before TTL expires.
3. **Background refresh:** Async worker refreshes hot keys before they expire.
4. **Singleflight pattern (Go):** Multiple goroutines waiting for same key merge into one DB call.

```python
import threading
_locks = {}

def get_or_set(key, fetch_fn, ttl=300):
    value = cache.get(key)
    if value is not None:
        return value
    lock = _locks.setdefault(key, threading.Lock())
    with lock:
        value = cache.get(key)  # check again after acquiring lock
        if value is None:
            value = fetch_fn()
            cache.set(key, value, ttl=ttl)
    return value
```

---

## 7. CDN (Content Delivery Network)

**Idea:** Cache static assets (images, CSS, JS, videos) at edge nodes close to users globally.

```
User in Mumbai → CDN edge in Mumbai (cache HIT, 5ms)
                                      vs
User in Mumbai → Origin server in US (cache MISS, 300ms)
```

**Cache-Control headers:**
```
Cache-Control: max-age=86400, public         ← CDN + browsers cache for 1 day
Cache-Control: no-cache                       ← always revalidate
Cache-Control: no-store                       ← never cache (sensitive data)
Cache-Control: private, max-age=3600          ← browser only, not CDN
ETag: "abc123"                               ← content fingerprint for conditional GETs
```

**Cache invalidation on CDN:** Send purge request or use versioned URLs (`/js/app.v3.js`).

---

## 8. Distributed Cache Architecture

```
Application Layer  →  Local L1 Cache (in-process, HashMap, 100µs)
                   →  Shared L2 Cache (Redis cluster, 1ms)
                   →  Database (PostgreSQL, 10-100ms)
```

**Cache hierarchy:**
- L1: Per-instance in-memory cache (Guava Cache, Caffeine). Fastest, but not shared.
- L2: Shared distributed cache (Redis). All instances see same data.
- Each layer is a fallback for the next.

---

## 9. Rate Limiting with Redis

```python
# Sliding window rate limit: max 100 req per minute per user
import time

def is_allowed(user_id: str, limit: int = 100, window: int = 60) -> bool:
    key = f"ratelimit:{user_id}"
    now = time.time()
    window_start = now - window

    pipe = redis.pipeline()
    pipe.zremrangebyscore(key, 0, window_start)   # remove old entries
    pipe.zadd(key, {str(now): now})                # add current request
    pipe.zcard(key)                                # count in window
    pipe.expire(key, window)                       # set TTL
    _, _, count, _ = pipe.execute()

    return count <= limit
```

---

## 10. Quick Reference

| Question | Answer |
|----------|--------|
| Cache-aside vs read-through? | Cache-aside: app manages cache; read-through: cache fetches from DB itself |
| When to avoid caching? | Real-time financial data, frequently updating data with strong consistency needs |
| What is a hot key? | A cache key accessed so frequently it overwhelms one Redis node; solution: local cache or key sharding |
| Redis persistence options? | RDB (snapshot at intervals) + AOF (append-only log of every write) |
