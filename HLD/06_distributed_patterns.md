# Distributed Systems Patterns: Interview Notes

Each pattern states the problem, the idea, trade-offs, and where it appears in your projects.

## 1. CAP and PACELC
- **CAP:** during a network partition you choose Consistency or Availability (partition tolerance is not optional in a distributed system).
- **PACELC:** else (no partition), you trade Latency against Consistency.
- CP examples: ZooKeeper, etcd, HBase. AP examples: Cassandra, DynamoDB (tunable), CouchDB.
- **Consistency models:** strong (linearizable), sequential, causal, eventual, read-your-writes.

## 2. Idempotency
**Problem:** retries and duplicate deliveries must not double-apply an effect (double charge, double booking).
**Idea:** the client sends an idempotency key; the server stores key -> result and returns the stored result on replay. Implement with a unique constraint on the key plus `INSERT ... ON CONFLICT DO NOTHING`.
**Used in:** Healthcare booking, ticket booking confirm.

## 3. Optimistic vs pessimistic concurrency
- **Optimistic (OCC / compare-and-set):** read a version, write only if it is unchanged (`UPDATE ... WHERE version = ?`). Best when conflicts are rare; retry on conflict.
- **Pessimistic:** lock first (`SELECT ... FOR UPDATE`). Best when conflicts are frequent; risks deadlocks and reduced throughput.
- **Used in:** Healthcare (CAS state transitions), ticket booking (both modes).

## 4. Distributed transactions: 2PC vs Saga
- **Two-phase commit:** coordinator asks all participants to prepare, then commit. Strong consistency, but blocking, slow, coordinator is a single point of failure.
- **Saga:** a sequence of local transactions, each with a **compensating action** if a later step fails (refund, release seat). Eventually consistent.
  - **Orchestration:** a central orchestrator tells services what to do (clearer, central logic).
  - **Choreography:** services react to events (loose coupling, harder to trace).
- **Used in:** ticket booking: hold seat -> charge payment -> confirm; on payment failure release the hold.

## 5. Transactional outbox and CDC
**Problem:** "write to DB and publish an event" is two operations; a crash between them loses or duplicates events.
**Idea:** write the event into an `outbox` table in the same DB transaction; a relay (poller or CDC such as Debezium) publishes it and marks it sent. Consumers must be idempotent (at-least-once).
**Used in:** Healthcare email outbox (retries, dead state), chat message relay.

## 6. Caching strategies and stampede control
- **Cache-aside, read-through, write-through, write-behind** (see `HLD/02_caching.md` and `Backend/05_redis.md`).
- **Single-flight / request coalescing:** only one request recomputes a missing key while others wait and share the result (URL shortener).
- **TTL jitter, early refresh, negative caching, Bloom filters.**

## 7. Rate limiting
| Algorithm | Behaviour | Notes |
|---|---|---|
| Fixed window | counter per window | simple; burst at boundaries |
| Sliding window log | timestamps in a sorted set | exact, memory heavy |
| Sliding window counter | weighted blend of two windows | good approximation |
| Token bucket | tokens refill at a rate; bursts allowed | most common |
| Leaky bucket | constant outflow; smooths traffic | queue-like |
Distributed: store counters in Redis with Lua for atomicity; shard hot keys; decide **fail-open vs fail-closed** if the limiter store is down. Return `429` with `Retry-After`.

## 8. Load balancing and service discovery
L4 (TCP) vs L7 (HTTP) balancers; algorithms: round robin, least connections, weighted, IP hash, **consistent hashing** (minimal reshuffling when nodes change, virtual nodes for balance). Health checks, connection draining, sticky sessions only when unavoidable.

## 9. Sharding and replication
- **Replication:** leader-follower (async lag, failover risk), multi-leader, leaderless (quorum reads/writes: `R + W > N`).
- **Sharding:** hash (even spread, hard range queries), range (good ranges, hot spots), directory-based. Cross-shard joins and transactions are expensive. Choose the shard key for access patterns; plan resharding.

## 10. Fan-out (news feed)
- **Fan-out on write (push):** precompute each follower's feed; fast reads, write amplification.
- **Fan-out on read (pull):** compute on request; cheap writes, slow reads.
- **Hybrid:** push for normal users, pull for celebrities above a follower threshold. Cursor pagination, ranking with time decay.
- **Used in:** your news feed project.

## 11. Real-time messaging (WebSockets)
Each server holds some connections; a **pub/sub backplane** (Redis) delivers messages across nodes. Needs: per-room monotonic sequence numbers for ordering, de-duplication by message id, offline queue, heartbeats and presence, backpressure for slow consumers, reconnect with replay from last sequence.
- **Used in:** your chat project.

## 12. Resilience patterns
| Pattern | Purpose |
|---|---|
| Timeout | never wait forever |
| Retry with exponential backoff and jitter | transient failures without thundering retries |
| Circuit breaker | stop calling a failing dependency, fail fast, probe to recover |
| Bulkhead | isolate resource pools so one failure does not sink everything |
| Fallback / graceful degradation | return cached/default/synchronous result when a dependency fails |
| Load shedding / backpressure | reject excess load early |
| Idempotent retries | safe repeats |
| Health checks and readiness probes | route traffic only to healthy instances |

**Used in:** Redis -> in-memory, Celery -> synchronous, LLM -> deterministic fallback.

## 13. Leader election, locks, consensus
Raft/Paxos (etcd, ZooKeeper) give consensus. Use leases with TTL; **fencing tokens** prevent a paused old leader from writing. Redis locks (SET NX PX) are fine for efficiency but not for strict safety.

## 14. Event-driven architecture
Events (facts about the past) vs commands; message brokers (Kafka log with partitions and consumer groups; RabbitMQ queues with routing). Delivery: at-most-once, at-least-once, exactly-once (effectively via idempotency or transactions). Ordering only within a partition/key. Dead-letter queues, schema evolution, replay.

## 15. Observability
Logs (structured, with request IDs), metrics (RED: rate, errors, duration; USE: utilisation, saturation, errors), traces (OpenTelemetry), alerts on SLOs. SLI/SLO/SLA definitions; error budgets.

## 16. Unique ID generation
Auto-increment (single DB), UUID (no coordination, poor locality), **Snowflake-style** (timestamp + machine id + sequence, sortable), ULID/UUIDv7 (time-ordered).

## 17. Consistent hashing sketch
```
ring = sorted(hash(node + "#" + i) for node in nodes for i in range(vnodes))
node_for(key): first ring point >= hash(key), wrapping around
```
Adding or removing a node moves only ~1/N of keys.

## Quick interview Q&A
- **Saga vs 2PC?** Local transactions with compensation (available, eventual) vs atomic blocking commit.
- **How do you avoid double booking in a distributed system?** Atomic conditional write on one source of truth, idempotency key, expiring holds.
- **What is the outbox pattern?** Same-transaction event row plus a relay.
- **How do you handle a cache stampede?** Single-flight, jittered TTL.
- **Consistency vs availability in CAP?** On partition choose one.
- **How do you design for failures?** Timeouts, retries with backoff, circuit breakers, fallbacks, idempotency.
- **What is consistent hashing for?** Distributing keys with minimal movement as nodes change.
