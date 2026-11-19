# System Design: 20 Most-Asked Questions

## Concepts (10)

**Q1. Vertical vs horizontal scaling?**
Vertical adds resources to one machine (simple, limited, a single point of failure). Horizontal adds machines behind a load balancer (needs stateless services and data partitioning, scales further).

**Q2. What does a load balancer do and which algorithms exist?**
Distributes traffic across healthy servers. Algorithms: round robin, least connections, weighted, IP hash, consistent hashing. L4 works on TCP, L7 on HTTP content; add health checks and draining.

**Q3. Explain the CAP theorem.**
During a network partition a distributed store must choose consistency (all nodes agree) or availability (every request gets a response). PACELC adds the latency/consistency trade-off when there is no partition.

**Q4. Strong vs eventual consistency?**
Strong: reads see the latest write. Eventual: replicas converge over time, so reads may be stale. Use strong for money and inventory, eventual for feeds and counters.

**Q5. What is consistent hashing and why use it?**
Keys and nodes are placed on a hash ring; a key belongs to the next node clockwise. Adding or removing a node moves only about 1/N of keys; virtual nodes balance load.

**Q6. Caching: where and what strategies?**
Client, CDN, API gateway, application cache (Redis), database cache. Strategies: cache-aside, read/write-through, write-behind; manage TTLs and invalidation; prevent stampedes with single-flight and jitter.

**Q7. SQL vs NoSQL?**
SQL for relational data and transactions; NoSQL for massive scale, flexible schema or simple access patterns (key-value, document, wide-column, graph). Often you use both.

**Q8. What is sharding and replication?**
Replication copies data for availability and read scaling (leader-follower, multi-leader, leaderless quorum). Sharding splits data by key across nodes for write/storage scaling; cross-shard joins and transactions get hard.

**Q9. How do message queues help (Kafka, RabbitMQ)?**
They decouple producers from consumers, absorb spikes, enable async processing and retries. Kafka is a partitioned log with consumer groups and replay; RabbitMQ is a broker with routing and acknowledgements. Delivery is usually at-least-once, so consumers must be idempotent.

**Q10. How do you design for failure?**
Timeouts, retries with exponential backoff and jitter, circuit breakers, bulkheads, fallbacks, idempotency keys, health checks, load shedding, graceful degradation, and observability (logs, metrics, traces).

## Design questions (10)

**Q11. Design a URL shortener.**
Base62 codes from an ID generator or hash; key-value store; read path CDN -> Redis -> DB with single-flight; 301 vs 302 trade-off; async click analytics; rate limiting; shard by code.

**Q12. Design a rate limiter.**
Token bucket or sliding window per key in Redis (Lua for atomicity); at the gateway; return 429 + `Retry-After`; decide fail-open vs fail-closed; shard hot keys.

**Q13. Design a news feed / social timeline.**
Fan-out on write for normal users, on read for celebrities (hybrid); feed lists in Redis sorted sets; cursor pagination; ranking with time decay; accept eventual consistency.

**Q14. Design a chat system (WhatsApp/Slack).**
WebSocket gateways, a pub/sub backplane across servers, per-conversation sequence numbers, persistence, offline queue and push notifications, presence via heartbeats, at-least-once delivery with client dedup.

**Q15. Design a notification system.**
API -> queue per channel (push, SMS, email) -> workers with provider adapters -> retries with backoff and dead-letter -> delivery status; user preferences, templates, rate limits, idempotency.

**Q16. Design Google Drive / Dropbox.**
Metadata DB plus object storage; chunking and deduplication by hash; resumable uploads; sync clients with change events and versioning; sharing ACLs; notifications; CDN for downloads.

**Q17. Design a search autocomplete.**
Trie or prefix index of popular queries with top-k per node, built offline from logs, served from memory/Redis, hot prefixes cached, periodic refresh.

**Q18. Design a payment system or e-commerce checkout.**
Idempotency keys, a state machine for orders and payments, saga with compensations, webhooks with signature verification, reconciliation jobs, ledger as the source of truth, no double charging.

**Q19. Design a ticket or appointment booking system.**
Atomic conditional update (compare-and-set) for holds, expiring holds, idempotent confirm, saga for payment failure, queue-based admission for hot events, DB as the source of truth.

**Q20. Design a video streaming or ride-sharing backend (outline).**
Streaming: upload -> transcode queue -> object storage + CDN with adaptive bitrate. Ride-sharing: location updates via geospatial index (geohash/quadtree), matching service, ETA service, surge pricing, WebSocket updates.
