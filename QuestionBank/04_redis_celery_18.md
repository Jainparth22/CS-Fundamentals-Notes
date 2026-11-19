# Redis (10) and Celery (8): Most-Asked Questions

## Redis

**Q1. What is Redis and why is it fast?**
An in-memory data-structure store used for caching, sessions, queues, rate limiting and leaderboards. It is fast because data lives in RAM, structures are simple and efficient, and a single-threaded event loop avoids lock contention.

**Q2. Which data types does Redis support and what are they for?**
Strings (counters, cache), hashes (objects), lists (queues), sets (unique items), sorted sets (leaderboards, sliding windows), bitmaps/HyperLogLog, streams, geo, and Pub/Sub channels.

**Q3. EXPIRE vs PERSIST, and how does expiry work?**
`EXPIRE` sets a time to live; `PERSIST` removes it. Redis deletes expired keys lazily on access and by periodically sampling keys.

**Q4. What eviction policies exist?**
`noeviction`, `allkeys-lru`, `allkeys-lfu`, `volatile-lru`, `volatile-lfu`, `volatile-ttl`, `allkeys-random`, `volatile-random`; chosen with `maxmemory-policy` when memory is full.

**Q5. Cache-aside vs write-through (and write-behind)?**
Cache-aside: the app reads the cache, on a miss loads from the DB and fills the cache. Write-through writes to cache and DB together (consistent, slower writes). Write-behind writes to the cache first and the DB later (fast, risk of loss).

**Q6. RDB vs AOF persistence?**
RDB takes periodic snapshots (compact, fast restart, may lose recent writes). AOF logs every write (more durable, larger, `appendfsync` trade-offs). Use both for durability, or neither for a pure cache.

**Q7. How do you handle a cache stampede (thundering herd)?**
Use a lock or single-flight so one request recomputes, jitter the TTLs, refresh early before expiry, and cache negative results. Related: cache penetration (Bloom filter) and avalanche (random TTLs).

**Q8. What is pipelining and when do you use it?**
Sending many commands without waiting for each reply, cutting network round trips. Use it for bulk operations; it is not atomic (use `MULTI/EXEC` or Lua for atomicity).

**Q9. How do you build a rate limiter or distributed lock with Redis?**
Rate limit: `INCR` + `EXPIRE` (fixed window), sorted-set timestamps (sliding window), or a Lua token bucket. Lock: `SET key token NX PX ttl`, release only if the token matches (Lua). Use database constraints when strict safety is required.

**Q10. How does Redis scale and stay available?**
Replication (async) for reads and failover, Sentinel for automatic promotion, and Redis Cluster to shard across 16,384 hash slots. Multi-key operations need keys in the same slot (hash tags).

## Celery

**Q11. How does Celery work end to end?**
Producer sends a task message to a broker (Redis/RabbitMQ); workers pull and execute it; results optionally go to a result backend; Celery Beat schedules periodic tasks.

**Q12. How do you handle failures and retries?**
`autoretry_for`, `retry_backoff` with jitter, `max_retries`, time limits, and a dead-letter/failed-state store. Tasks must be idempotent because delivery is at-least-once.

**Q13. What happens if a worker crashes mid-task?**
With `acks_late=True` the message is redelivered (task may run twice); with early ack it is lost. Choose per task and design for idempotency.

**Q14. Redis vs RabbitMQ as a broker?**
Redis is simple and fast with weaker delivery guarantees (visibility timeout caveats). RabbitMQ gives acknowledgements, routing, priorities and durability.

**Q15. What are chains, groups and chords?**
`chain` runs tasks in sequence passing results; `group` runs tasks in parallel; `chord` runs a group then a callback with all results.

**Q16. How do you route, prioritise and scale tasks?**
Separate queues per workload (`emails`, `reports`, `ml`) with dedicated workers, `task_routes`, `worker_prefetch_multiplier=1` for long tasks, autoscale on queue depth, and rate limits per task.

**Q17. Celery with Flask or FastAPI, and eager mode?**
Create the Celery app from the web app config; run tasks inside an app context (Flask). In FastAPI enqueue and return a job id; do not block on `.get()`. `task_always_eager=True` runs tasks inline, useful for tests and local demos.

**Q18. When would you avoid Celery, and how do you monitor it?**
Avoid it for short synchronous work or tiny scale (use threads or `BackgroundTasks`). Monitor with Flower, logs with task and request ids, and alerts on queue length and failure rate; run exactly one Beat instance.
