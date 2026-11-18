# Scalability — Horizontal Scaling, Consistent Hashing, CAP Theorem

---

## 1. Vertical vs Horizontal Scaling

| | Vertical (Scale Up) | Horizontal (Scale Out) |
|---|--------------------|-----------------------|
| Approach | Add more CPU/RAM/storage to one server | Add more servers |
| Limit | Hardware ceiling (biggest machine) | Practically unlimited |
| Cost | Expensive per unit | Cheaper commodity hardware |
| Failure | Single point of failure | Fault-tolerant by design |
| Complexity | Simple (no code change) | Complex (distributed coordination) |
| Downtime | Restart required for upgrade | Rolling updates, zero downtime |
| Examples | Upgrade RDS from db.t3 to db.r6g | Add more API pods in Kubernetes |

**When to vertical scale:** Databases (less complexity), early-stage products.
**When to horizontal scale:** Stateless services (API servers), when traffic grows beyond one machine.

---

## 2. Load Balancing

Distributes incoming traffic across multiple servers.

### Algorithms

| Algorithm | Description | Use Case |
|-----------|-------------|----------|
| Round Robin | Requests distributed cyclically | Uniform servers, uniform requests |
| Weighted Round Robin | Servers with more capacity get more | Heterogeneous server capacity |
| Least Connections | Route to server with fewest active connections | Variable request duration |
| IP Hash | Same client IP always → same server | Session affinity (stateful apps) |
| Random | Random selection | Simple, works well at scale |

### L4 vs L7 Load Balancer

- **L4 (Transport):** Routes based on IP + port, doesn't inspect payload. Faster. (AWS NLB)
- **L7 (Application):** Routes based on HTTP headers, URL path, cookies. Smarter. (AWS ALB, nginx, Envoy)

---

## 3. Consistent Hashing

**Problem:** With N servers, `hash(key) % N` causes massive re-distribution when N changes (adding/removing servers).

**Solution:** Map both keys and servers onto a ring (0 to 2^32). A key is served by the first server clockwise on the ring.

```
Ring: 0 ──────── A ──────── B ──────── C ──────── 2^32
           ↑                     ↑
      Keys here go to A     Keys here go to B

Add server D between A and B:
  Only keys between A and D now move to D (not all keys)
  On average: N/M keys move (where N=keys, M=servers)
```

**Virtual nodes:** Each physical server maps to K positions on the ring (K=100-200). Improves load distribution and handles heterogeneous servers.

```python
import hashlib, bisect

class ConsistentHash:
    def __init__(self, servers, replicas=100):
        self.ring = {}
        self.sorted_keys = []
        for s in servers:
            for i in range(replicas):
                key = self._hash(f"{s}:{i}")
                self.ring[key] = s
                self.sorted_keys.append(key)
        self.sorted_keys.sort()

    def _hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)

    def get_server(self, key):
        h = self._hash(key)
        idx = bisect.bisect(self.sorted_keys, h) % len(self.sorted_keys)
        return self.ring[self.sorted_keys[idx]]
```

**Used in:** Cassandra, DynamoDB, Redis Cluster, CDN routing.

---

## 4. CAP Theorem

**A distributed system can guarantee only 2 of 3:**

| Property | Meaning |
|----------|---------|
| **C**onsistency | Every read returns the most recent write (or an error) |
| **A**vailability | Every request receives a response (not necessarily most recent) |
| **P**artition Tolerance | System continues despite network partition (dropped messages) |

**Key insight:** Network partitions are unavoidable in real distributed systems. So the real choice is **CP vs AP** when a partition occurs.

| System | Chooses | Behavior During Partition |
|--------|---------|--------------------------|
| HBase, Zookeeper, MongoDB (default) | CP | Refuse writes to stay consistent |
| Cassandra, CouchDB, DynamoDB | AP | Accept writes, serve potentially stale reads |
| Traditional RDBMS (single node) | CA | (Not distributed, so partition not applicable) |

---

## 5. PACELC Theorem (extends CAP)

Even without a partition, there's a trade-off:
- During **P**artition: **A**vailability vs **C**onsistency
- **E**lse (normal operation): **L**atency vs **C**onsistency

| System | P choice | E choice |
|--------|----------|----------|
| DynamoDB | AP | EL (eventual, low latency) |
| BigTable/HBase | CP | EC (consistent) |
| PostgreSQL | CP | EC |

---

## 6. Stateless vs Stateful Services

| | Stateless | Stateful |
|---|-----------|---------|
| Session data | In shared store (Redis, DB) | In-process memory |
| Horizontal scaling | Trivial (add more instances) | Hard (sticky sessions or data sync) |
| Failure recovery | Any replica can handle | Need state transfer or failover |
| Examples | REST APIs, microservices | WebSocket servers, game servers |

**Best practice:** Keep services stateless; push state to dedicated stores (Redis, DB).

---

## 7. Fault Tolerance Patterns

| Pattern | Description |
|---------|-------------|
| Replication | N copies of data on N servers; tolerate N-1 failures |
| Circuit Breaker | Stop calling a failing service; fail fast; retry after cooldown |
| Bulkhead | Isolate failures with separate thread pools / resources per service |
| Retry with backoff | Exponential backoff + jitter to avoid thundering herd |
| Timeout | Never wait forever; set a timeout and fail gracefully |
| Health checks | Load balancer removes unhealthy instances from rotation |

---

## 8. Numbers Every Engineer Should Know

| Operation | Latency |
|-----------|---------|
| L1 cache reference | 1 ns |
| L2 cache reference | 4 ns |
| Main memory reference | 100 ns |
| SSD random read | 100 µs |
| HDD seek | 10 ms |
| Round trip within same DC | 0.5 ms |
| Round trip US → Europe | ~150 ms |

| Throughput | Value |
|------------|-------|
| Gigabit ethernet | 125 MB/s |
| NVMe SSD sequential read | 3-7 GB/s |
| Typical API server | 1K-10K req/s per instance |
