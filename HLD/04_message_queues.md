# Message Queues & Event-Driven Architecture

---

## 1. Why Message Queues?

**Problems they solve:**
- **Decoupling:** Producer doesn't need to know about consumers
- **Buffering:** Handle traffic spikes without dropping requests
- **Async processing:** Return HTTP 202 immediately; process in background
- **Fan-out:** One message → multiple consumers
- **Retry:** Failed processing retried without bothering the producer

---

## 2. Core Concepts

| Term | Meaning |
|------|---------|
| **Producer** | Sends messages to the queue |
| **Consumer** | Reads and processes messages |
| **Queue** | Buffer holding messages (FIFO) |
| **Topic** | Named channel (Kafka); multiple queues subscribe to a topic |
| **Exchange** | RabbitMQ routing component |
| **Dead Letter Queue (DLQ)** | Messages that failed processing N times go here |
| **Acknowledgment (ACK)** | Consumer confirms successful processing |

---

## 3. Kafka Architecture

```
Producer → Topic (partitioned log) → Consumer Group

Topic: ordered_events
  Partition 0: [msg1] [msg4] [msg7]  ← Consumer A processes
  Partition 1: [msg2] [msg5] [msg8]  ← Consumer B processes
  Partition 2: [msg3] [msg6] [msg9]  ← Consumer C processes

Consumer Group: each partition is consumed by exactly ONE consumer in group
Offset: position in partition log (consumers commit offsets)
```

**Key Kafka properties:**
- Messages are **persistent** (retained configurable duration, default 7 days)
- Multiple consumer groups can independently read the same topic
- **Ordering guaranteed within a partition** (not globally)
- **High throughput:** Sequential disk writes + zero-copy networking

**Kafka partition key:** `hash(key) % num_partitions` — same key always to same partition → ensures ordering for related events (e.g., all events for user_id 123).

---

## 4. RabbitMQ vs Kafka

| Feature | RabbitMQ | Kafka |
|---------|----------|-------|
| Model | Message queue (push) | Distributed log (pull) |
| Message retention | Deleted after ACK | Retained for configured duration |
| Ordering | Per queue | Per partition |
| Consumer model | Competing consumers | Consumer groups |
| Throughput | Moderate (~50K msg/s) | Very high (~1M msg/s) |
| Routing | Flexible exchanges (fanout, direct, topic) | Topic-based |
| Replay | ❌ | ✅ (replay from offset) |
| Use for | Task queues, RPC, complex routing | Event streaming, audit log, ML pipelines |

---

## 5. AWS SQS / SNS

| | SQS | SNS |
|---|-----|-----|
| Type | Queue | Pub/Sub |
| Retention | Up to 14 days | No retention |
| Fan-out | 1 consumer at a time | Multiple subscribers |
| Use | Background jobs, rate limiting | Notifications, fan-out to multiple SQS |

**Common pattern — SNS + SQS Fan-out:**
```
OrderCreated event
    → SNS Topic
        → SQS Queue 1 → Email Service (sends confirmation)
        → SQS Queue 2 → Inventory Service (reserve stock)
        → SQS Queue 3 → Analytics Service (track event)
```

**SQS message visibility timeout:** When consumer picks up a message, it's hidden from others for N seconds. If not ACK'd in time, it becomes visible again (another consumer retries).

---

## 6. Exactly-Once vs At-Least-Once vs At-Most-Once

| Semantic | Description | Risk |
|----------|-------------|------|
| **At-Most-Once** | Message sent once; if lost, gone. | Data loss |
| **At-Least-Once** | Retried until ACK; duplicates possible | Duplicate processing |
| **Exactly-Once** | No duplicates, no loss | Complex, slower |

**Most systems default to at-least-once.** Make consumers **idempotent** (processing same message twice has same effect as once) — this is the practical solution.

```python
def process_order(order_id: str):
    # Idempotency check: skip if already processed
    if db.exists(f"processed:{order_id}"):
        return
    # Process...
    db.set(f"processed:{order_id}", "1", ex=86400)
```

---

## 7. Outbox Pattern (Transactional Messaging)

**Problem:** How to atomically update DB and publish an event? They're two separate systems.

```python
# WRONG — not atomic
db.update(order, status="PAID")   # succeeds
kafka.publish(OrderPaid(order))   # crashes here → event never sent!

# RIGHT — Outbox Pattern
BEGIN TRANSACTION:
    db.update(order, status="PAID")
    db.insert(outbox, event=OrderPaid)   ← same transaction
COMMIT

# Separate process (CDC or polling) reads outbox table and publishes to Kafka
```

**Benefits:** Guaranteed at-least-once delivery using DB ACID. Event published iff transaction commits.

---

## 8. Backpressure

**Problem:** Producer is faster than consumer → queue grows unboundedly → OOM.

**Solutions:**
- Bounded queues (block or drop when full)
- Consumer tells producer to slow down (reactive streams)
- Scale consumers (auto-scaling on queue depth)
- Shed load (drop low-priority messages)

---

## 9. Event-Driven Patterns

| Pattern | Description |
|---------|-------------|
| **Event Notification** | "Something happened" — receiver decides what to do |
| **Event-Carried State Transfer** | Event contains full state; consumer doesn't query source |
| **Event Sourcing** | Store sequence of events as source of truth; derive current state |
| **CQRS** | Separate Command (write) and Query (read) models |
| **Saga** | Distributed transaction via sequence of events + compensating transactions |

---

## 10. Quick Reference

| Question | Answer |
|----------|--------|
| Kafka vs RabbitMQ for event replay? | Kafka (persistent log); RabbitMQ deletes after ACK |
| What is a consumer lag? | Difference between latest offset and consumer's current offset; high lag = consumer too slow |
| How to handle poison pill messages? | Move to DLQ after N retries; alert and inspect manually |
| What is a message broker? | Middleware that routes messages (RabbitMQ, ActiveMQ) vs log (Kafka) |
| DLQ best practice? | Monitor DLQ size; set alerts; have replay mechanism |
