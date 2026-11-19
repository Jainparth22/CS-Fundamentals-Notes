# System Design Interview: Framework and Worked Designs

## 1. A repeatable framework (45 minutes)
1. **Clarify requirements (5 min).** Functional (what it does), non-functional (latency, availability, consistency, scale), out-of-scope. Ask users, read/write ratio, data size.
2. **Estimate capacity (3 min).** QPS, storage, bandwidth. Example: 100M DAU x 10 reads/day = 1B reads/day ~ 12K QPS average, peak 3x.
3. **API design (3 min).** Key endpoints, request/response, pagination, auth.
4. **Data model (5 min).** Entities, relations, SQL vs NoSQL choice, indexes, shard key.
5. **High-level design (10 min).** Clients, load balancer, app servers, cache, DB, queue, workers, CDN, storage.
6. **Deep dive (10 min).** The 1 or 2 hardest parts (concurrency, ordering, hot keys, consistency).
7. **Bottlenecks and trade-offs (5 min).** Failures, scaling, monitoring, alternatives.
Always state trade-offs out loud.

## 2. Handy numbers
- 1 day = 86,400 s ~ 10^5 s. 1M requests/day ~ 12 QPS. 1B/day ~ 12K QPS.
- Memory read ~100 ns; SSD read ~100 us; network round trip in a datacenter ~0.5 ms; cross-continent ~100 ms.
- 1 char = 1 byte (ASCII). 1 KB = 10^3, 1 MB = 10^6, 1 GB = 10^9 (approx).
- A single Redis node: ~100K ops/s. A single SQL node: thousands to tens of thousands QPS depending on queries.

## 3. Building blocks cheat sheet
Load balancer, reverse proxy (Nginx), CDN, API gateway, app servers (stateless), cache (Redis/Memcached), relational DB (+ replicas), NoSQL (Cassandra/DynamoDB/Mongo), object storage (S3), message queue (Kafka/RabbitMQ/SQS), workers (Celery), search (Elasticsearch), blob/CDN, rate limiter, monitoring, service discovery, config store.

## 4. Design 1: URL shortener
- **Requirements:** create short link, redirect, optional custom alias, analytics, expiry. Read-heavy (100:1).
- **API:** `POST /shorten {url}` -> `{code}`; `GET /{code}` -> 301/302.
- **Key generation:** Base62 of an auto-increment/ID generator (7 chars = 62^7 ~ 3.5 trillion), or hash + collision check, or a pre-generated key service with ranges handed to app servers.
- **Storage:** key-value (code -> url, user, created, expiry); SQL or DynamoDB is fine.
- **Read path:** CDN/edge -> cache (LRU in Redis) -> DB. **Single-flight** to stop stampedes on viral links.
- **Redirect code:** 301 (cacheable, less load, less analytics) vs 302 (every click hits you, analytics accurate).
- **Analytics:** push click events to a queue, aggregate asynchronously.
- **Rate limit** creation; abuse and malware checks.
- **Scaling:** shard by code; replicas; cache hit rates of 90%+.
(Your repo: `distributed-url-shortener`.)

## 5. Design 2: Rate limiter
- Place at the API gateway or middleware; key by user/IP/API key.
- Token bucket per key in Redis (Lua for atomic refill-and-take); sliding window counter alternative.
- Return 429 + `Retry-After`; headers with remaining quota.
- Distributed correctness: central Redis or local limits with periodic sync; hot-key sharding; fail-open vs fail-closed decision.
(Your repo: `distributed-rate-limiter`.)

## 6. Design 3: Chat / messaging
- **Connections:** WebSockets via gateway servers; user -> server mapping in Redis; pub/sub across servers.
- **Message flow:** client sends -> server assigns per-conversation sequence number -> persist -> fan out to online recipients, queue for offline (push notification on mobile).
- **Storage:** messages in a wide-column/partitioned store keyed by conversation and time; recent messages cached.
- **Guarantees:** at-least-once delivery with client-side dedup by message id; ordering by sequence number; delivery/read receipts.
- **Presence:** heartbeats with TTL. **Groups:** fan-out on write for small groups, on read for large channels.
- **Scaling:** shard by conversation id; backpressure for slow consumers; transactional outbox for durability.
(Your repo: `scalable-realtime-chat`.)

## 7. Design 4: News feed
- **Requirements:** post, follow, see a ranked feed fast.
- **Fan-out strategy:** push for typical users, pull for celebrities (threshold), hybrid merge at read time.
- **Data:** posts table, follow graph, per-user feed list (Redis sorted set of post ids by score/time) with a cap; hydrate post details from cache/DB.
- **Pagination:** cursor-based (score + id).
- **Ranking:** recency decay plus engagement; ML ranker later.
- **Scaling issues:** write amplification, hot celebrity posts (cache), eventual consistency of feeds.
(Your repo: `scalable-newsfeed-system`.)

## 8. Design 5: Ticket / appointment booking (no double booking)
- **Requirements:** browse availability, hold a seat/slot, pay, confirm; no double booking; handle payment failure and abandonment.
- **Core:** seat/slot row with `status` and `version`. Hold with atomic conditional update (`WHERE status='free' AND version=?`); hold expires (TTL, 5 to 10 minutes) via scheduled sweep.
- **Confirm:** idempotency key + saga: charge payment, confirm booking; on failure compensate (release hold, refund).
- **Webhooks:** verify signature, idempotent handling.
- **Scaling:** hot events -> queue-based admission (virtual waiting room), shard by event, cache availability read-only with short TTL, source of truth is the DB.
- **Edge cases:** payment succeeds after hold expired (reconcile and refund), duplicate webhooks, partial failure.
(Your repos: `high-concurrency-ticket-booking`, Healthcare Appointment Manager.)

## 9. Design 6: File storage / Google Drive (outline)
Metadata DB (files, folders, permissions) + object storage (S3) for blocks; chunk and dedup files by hash; resumable uploads; sync clients with change events and versioning; sharing via ACLs; notifications via a queue; CDN for downloads.

## 10. Design 7: Notification system (outline)
API -> validation -> priority queues (per channel: email, SMS, push) -> workers with provider adapters -> retries with backoff and dead-letter -> delivery status store; templates; user preferences and rate limits; idempotency keys to prevent duplicates.

## 11. Design 8: Search autocomplete (outline)
Trie or prefix index of popular queries with top-k per node; built offline from logs; served from memory/Redis; update periodically; cache hot prefixes; personalise lightly.

## 12. Design 9: ML inference service (relevant to your profile)
- Model registry and versioning; containerised service (FastAPI) behind a load balancer.
- **Latency/throughput:** batching requests, GPU/CPU sizing, model quantisation/ONNX, warm-up, autoscaling on queue depth/latency.
- **Caching** of repeated predictions; async jobs for slow models (queue + status endpoint).
- **Reliability:** timeouts, fallback model/heuristic, canary/shadow deploys, rollback.
- **Monitoring:** latency percentiles, error rate, input drift, prediction distribution, feedback loop.
- **Data:** feature store for consistent training and serving features.

## 13. Design 10: RAG question-answering service (relevant to your profile)
Ingestion (parse, chunk, embed, store in a vector DB with metadata) -> query path (guard input, rewrite, hybrid retrieve BM25 + dense, rerank, build prompt with citations, LLM call, output guard) -> evaluation (RAGAS, golden set) -> caching, provider fallback, cost and latency controls, access control per document.

## 14. Common deep-dive answers
- **How to scale reads?** Cache, replicas, CDN, denormalised read models.
- **How to scale writes?** Shard, queue and batch, append-only logs, async processing.
- **How to handle hot keys?** Replicate or split the key, local cache, request coalescing.
- **Consistency in a booking system?** Single source of truth with conditional writes; cache only for display.
- **How to ensure message ordering?** Partition by key; sequence numbers per entity.
- **How to deal with failures?** Timeouts, retries with backoff and jitter, circuit breakers, idempotency, graceful degradation.
- **SQL or NoSQL?** SQL for relations and transactions; NoSQL for massive scale, simple access patterns, flexible schema.

## 15. Practice list
Design: URL shortener, pastebin, rate limiter, key-value store, chat, news feed, notification service, ride sharing (Uber), video streaming (YouTube/Netflix), file sync, ticket booking, web crawler, search autocomplete, distributed cache, job scheduler, payment system, ML inference platform, RAG assistant. For each: write requirements, API, data model, diagram, bottlenecks in 30 minutes, then compare with your notes.
