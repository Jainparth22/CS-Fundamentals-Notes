# Redis: Interview Notes

## 1. What Redis is
An in-memory data-structure store used as a cache, session store, message broker, rate limiter, leaderboard and lock service. Single-threaded command execution (I/O threads exist in newer versions), so each command is atomic. Sub-millisecond latency.

## 2. Data types and typical uses
| Type | Commands | Use |
|---|---|---|
| String | `SET/GET/INCR/EXPIRE/SETNX` | cache values, counters, locks, rate limits |
| Hash | `HSET/HGET/HGETALL/HINCRBY` | objects, sessions |
| List | `LPUSH/RPOP/BRPOP/LRANGE` | queues, recent items |
| Set | `SADD/SISMEMBER/SINTER` | unique members, tags, follows |
| Sorted set | `ZADD/ZRANGE/ZREVRANK/ZREMRANGEBYSCORE` | leaderboards, sliding-window rate limit, delayed jobs |
| Bitmap / HyperLogLog | `SETBIT/PFADD/PFCOUNT` | daily active flags, approximate unique counts |
| Stream | `XADD/XREADGROUP/XACK` | durable event log with consumer groups |
| Geo | `GEOADD/GEOSEARCH` | nearby search |
| Pub/Sub | `PUBLISH/SUBSCRIBE` | fire-and-forget fan-out (not durable) |

## 3. Caching patterns
- **Cache-aside (lazy loading):** read cache; on miss read DB and populate with a TTL; on write update the DB and invalidate (or update) the cache. Most common; what your Placement Portal uses.
- **Read-through / write-through / write-behind:** cache sits in front of the store; write-through writes both synchronously; write-behind writes to the DB asynchronously (fast, risks loss).
- **TTL and eviction:** always set TTLs. `maxmemory-policy`: `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, `noeviction`.
- **Invalidation** is the hard part: prefer short TTLs plus explicit deletes on write.

### Cache problems and fixes
| Problem | What happens | Fix |
|---|---|---|
| Cache stampede (thundering herd) | a hot key expires and many requests hit the DB | locking / single-flight, early refresh, jittered TTLs |
| Cache penetration | requests for keys that never exist hit the DB | cache negative results, Bloom filter |
| Cache avalanche | many keys expire together | randomise TTLs, warm caches, multi-layer cache |
| Hot key | one key overloads a node | local in-process cache, key replication/sharding |
| Stale data | cache differs from DB | TTL, event-driven invalidation, version in key |

## 4. Persistence
- **RDB snapshots:** periodic point-in-time dumps; fast restart, may lose recent writes.
- **AOF (append-only file):** logs every write; `appendfsync always/everysec/no` trades durability for speed. AOF rewrite compacts it.
- Use both for durability; for a pure cache you can disable persistence.

## 5. High availability and scaling
- **Replication:** primary to replicas (asynchronous), so a failover can lose recent writes.
- **Sentinel:** monitors and promotes a replica automatically.
- **Redis Cluster:** shards data across nodes using **16384 hash slots**; multi-key operations must be in one slot (use hash tags `{user:1}`).
- Managed options: ElastiCache, Upstash, Redis Cloud.

## 6. Transactions, scripting, atomicity
- `MULTI ... EXEC` queues commands and runs them atomically (no rollback on logic errors). `WATCH` gives optimistic locking (CAS).
- **Lua scripts (`EVAL`)** run atomically; ideal for read-modify-write like a token bucket.
- **Pipelining** batches round trips for throughput.

## 7. Distributed lock
```python
token = uuid4().hex
ok = r.set("lock:slot:42", token, nx=True, px=10_000)      # set if not exists, with expiry
try:
    if ok: do_work()
finally:
    # release only if we still own it (atomic via Lua)
    r.eval("if redis.call('get',KEYS[1])==ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end", 1, "lock:slot:42", token)
```
Always set an expiry so a crashed holder does not block forever. For strong correctness use database constraints or fencing tokens; Redlock is debated for safety-critical locks.

## 8. Rate limiting with Redis
- **Fixed window:** `INCR key; EXPIRE key window` (bursty at boundaries).
- **Sliding window log:** sorted set of timestamps, `ZREMRANGEBYSCORE` old, `ZCARD` to count.
- **Token bucket:** hash with tokens and last refill, updated in Lua; allows bursts.
- **Fail-open vs fail-closed** if Redis is down; decide per endpoint (your rate limiter project supports both).

## 9. Redis as a queue / broker
- Lists (`LPUSH`/`BRPOP`) are simple but lose messages on worker crash.
- **Streams with consumer groups** provide acknowledgements and replays.
- **Celery with Redis** uses lists plus visibility timeouts; for guaranteed delivery RabbitMQ is stronger.
- Pub/Sub does not store messages; offline subscribers miss them (used as the chat backplane).

## 10. Session store and JWT blacklist
Store session data in a hash with TTL. A JWT blacklist is a key `blacklist:<jti>` with TTL equal to the token's remaining life, so the set cleans itself.

## 11. Operational notes
- Avoid `KEYS *` in production (blocks); use `SCAN`.
- Large keys and big values hurt latency; keep values small, compress if needed.
- Monitor memory, evictions, hit rate, latency (`INFO`, `SLOWLOG`), connected clients; use connection pools.
- Redis is not a primary database for critical data unless you accept its durability trade-offs.
- Security: bind to private networks, set a password/ACLs, TLS.

## Quick interview Q&A
- **Why is Redis fast?** In-memory, simple data structures, single-threaded command loop without lock contention, efficient I/O multiplexing.
- **Redis vs Memcached?** Redis has rich types, persistence, replication, scripting; Memcached is simple multithreaded caching.
- **How do you prevent a cache stampede?** Single-flight/lock around recompute, jittered TTL, early refresh.
- **How does Redis handle expiry?** Lazy deletion on access plus periodic sampling of expiring keys.
- **What happens if Redis goes down?** Fall back to the DB or in-memory cache; fail-open/closed for limiters; design for graceful degradation (your portal does).
- **Pub/Sub vs Streams?** Pub/Sub is transient; Streams are durable with consumer groups.
- **How would you build a leaderboard?** Sorted set: `ZADD`, `ZREVRANGE`, `ZREVRANK`.
- **Redis persistence options?** RDB, AOF, or both.
- **How do you shard Redis?** Redis Cluster hash slots or client-side consistent hashing.
