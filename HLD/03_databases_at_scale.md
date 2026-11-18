# Databases at Scale

---

## 1. SQL vs NoSQL

| Dimension | SQL (Relational) | NoSQL |
|-----------|-----------------|-------|
| Schema | Fixed (DDL) | Flexible / schemaless |
| Relationships | Foreign keys, joins | Denormalized, embedded |
| ACID | ✅ Full | Varies (often BASE) |
| Scaling | Primarily vertical; horizontal via read replicas | Horizontal native |
| Query language | SQL (standard) | Proprietary APIs / limited query |
| Consistency | Strong | Often eventual |
| Use when | Complex queries, relations, transactions | High write throughput, flexible schema, massive scale |
| Examples | PostgreSQL, MySQL, SQLite | MongoDB, Cassandra, DynamoDB, HBase |

---

## 2. NoSQL Types

| Type | Model | Best For | Examples |
|------|-------|---------|---------|
| Document | JSON/BSON documents | CMS, catalogs, user profiles | MongoDB, CouchDB |
| Key-Value | Simple key → value | Sessions, caching, counters | Redis, DynamoDB, Riak |
| Wide-Column | Tables with dynamic columns | Time-series, write-heavy analytics | Cassandra, HBase |
| Graph | Nodes and edges | Social networks, recommendations | Neo4j, Amazon Neptune |

---

## 3. Read Replicas & Replication

```
Primary (read + write)
    │
    ├── Replica 1 (read only)
    ├── Replica 2 (read only)
    └── Replica 3 (read only)

Application: writes → Primary, reads → any Replica

Replication lag: Replica may be milliseconds to seconds behind Primary
  → Problem: read-your-writes consistency (write to primary, immediately read from replica → see old data)
  → Fix: read from primary immediately after write, or sticky reads to same replica
```

**Replication modes:**
- **Synchronous:** Primary waits for at least one replica to confirm write. Guarantees no data loss on failover. Higher write latency.
- **Asynchronous:** Primary doesn't wait. Lower latency. Risk of data loss on failover.
- **Semi-synchronous (MySQL):** Wait for one replica, rest are async.

---

## 4. Sharding (Horizontal Partitioning)

Split data across multiple DB nodes. Each shard is an independent DB holding a subset of data.

### Sharding Strategies

**Range-based:**
```
Shard 1: user_id 1–1M
Shard 2: user_id 1M–2M
Shard 3: user_id 2M–3M

Pros: Range queries efficient
Cons: Hot shards (sequential IDs all hit shard 1)
```

**Hash-based:**
```
Shard = hash(user_id) % N

Pros: Uniform distribution
Cons: Range queries hit all shards; resharding is expensive
```

**Directory-based (shard map):**
```
Lookup service: user_id → shard_id
Pros: Flexible rebalancing
Cons: Lookup service = bottleneck / SPOF
```

### Resharding Problem
When adding/removing shards with hash-based, almost all keys move. **Consistent hashing** (see scalability notes) minimizes key movement.

---

## 5. Write-Ahead Log (WAL)

```
Before ANY data modification:
  1. Write change to WAL (append-only, sequential disk write → fast)
  2. fsync WAL to disk
  3. Apply change to data page (can be delayed — "lazy write")

On crash:
  1. Scan WAL from last checkpoint
  2. Redo committed transactions from WAL
  3. Undo uncommitted transactions
```

WAL enables both **durability** (committed data survives crash) and **fast writes** (sequential log writes are much faster than random page writes).

**PostgreSQL:** WAL = `pg_wal/` directory. `wal_level`, `synchronous_commit`, `checkpoint_timeout` are key params.

---

## 6. Connection Pooling

```
Problem: Creating a new DB connection = 20-100ms (TCP + auth + TLS)
         Web request = 10ms + 100ms connection = absurd

Solution: Maintain a pool of persistent connections

App servers ──► Pool (10-50 connections) ──► PostgreSQL (max_connections=200)

PgBouncer: connection pooler for PostgreSQL
  - Transaction mode: connection returned to pool after each transaction (best)
  - Session mode: connection held for entire client session
```

---

## 7. Indexing at Scale (Quick Reference)

| Scenario | Solution |
|----------|---------|
| Slow `WHERE col = ?` on large table | Add index on col |
| Slow `ORDER BY col` | Add index on col |
| Index exists but query still slow | Check cardinality; rebuild index; check for implicit casting |
| Write throughput dropping | Too many indexes; remove unused ones; use partial indexes |
| Need full-text search | Elasticsearch or PostgreSQL `tsvector/tsquery` |

---

## 8. NewSQL — Best of Both Worlds

Distributed SQL databases with ACID and horizontal scaling:

| System | Description |
|--------|-------------|
| **CockroachDB** | Distributed PostgreSQL-compatible, strong consistency, global distribution |
| **Google Spanner** | Google's globally distributed SQL database; uses TrueTime for external consistency |
| **TiDB** | MySQL-compatible, distributed, HTAP (hybrid OLTP+OLAP) |
| **YugabyteDB** | PostgreSQL-compatible, distributed, cloud-native |

---

## 9. OLTP vs OLAP

| | OLTP (Online Transaction Processing) | OLAP (Online Analytical Processing) |
|---|--------------------------------------|--------------------------------------|
| Queries | Simple, row-level (read/write) | Complex, aggregate (read-heavy) |
| Data freshness | Real-time | Near real-time to batch |
| Table design | Normalized (3NF/BCNF) | Denormalized (star/snowflake schema) |
| Row/column store | Row-oriented | Column-oriented |
| Examples | PostgreSQL, MySQL | BigQuery, Redshift, ClickHouse, Snowflake |

**Data pipeline:** OLTP → ETL/CDC → Data Warehouse (OLAP) → BI dashboard

---

## 10. Quick Reference

| Question | Answer |
|----------|--------|
| What is N+1 query problem? | Fetching N records + 1 extra query per record (instead of one JOIN) |
| Fix for N+1? | Eager loading with JOIN or batch fetch |
| What is CDC? | Change Data Capture — stream DB changes (Debezium, Kafka Connect) |
| Vertical vs horizontal sharding? | Vertical = split by columns (feature decomp); Horizontal = split by rows (sharding) |
| What is a hot shard? | One shard receives disproportionate traffic; use hash + virtual shards to distribute |
