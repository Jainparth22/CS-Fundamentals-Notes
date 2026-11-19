# SQL Indexes, Transactions, Normalization and Query Optimization

## 1. Indexes
An index is a separate sorted data structure (usually a **B+ tree**) that lets the database find rows without scanning the whole table.

| Index | Use |
|---|---|
| B-tree / B+ tree | default; equality, range, ORDER BY, prefix LIKE |
| Hash | equality only |
| Composite (multi-column) | follows the **leftmost prefix rule** |
| Unique | enforce uniqueness |
| Covering | contains every column the query needs, so no table lookup (index-only scan) |
| Partial / filtered | index only rows matching a condition (PostgreSQL) |
| Expression / functional | index on `LOWER(email)` |
| GIN / GiST | JSONB, arrays, full text, geometry (PostgreSQL) |
| Clustered | table rows stored in index order (InnoDB primary key; SQL Server clustered index) |
| Non-clustered / secondary | separate structure pointing to the row or the PK |

### Composite index and leftmost prefix
`INDEX(a, b, c)` helps `WHERE a=?`, `WHERE a=? AND b=?`, `WHERE a=? AND b>?`, but not `WHERE b=?` alone. Put equality columns first, range column last, and match ORDER BY.

### Why an index may NOT be used
- Function on the column: `WHERE YEAR(created_at) = 2026` (use a range instead).
- Leading wildcard: `LIKE '%abc'`.
- Implicit type conversion (string vs int).
- Low selectivity (the optimizer prefers a scan, e.g. a boolean column).
- `OR` across different columns, or `!=` / `NOT IN` in many cases.
- Stale statistics (run `ANALYZE`).

### Costs
Indexes speed reads and slow writes (every INSERT/UPDATE/DELETE maintains them) and use disk. Do not index everything; index foreign keys, join columns, frequent filters and sorts.

### Reading a plan
```sql
EXPLAIN SELECT ...;          -- plan
EXPLAIN ANALYZE SELECT ...;  -- plan plus real timings (PostgreSQL / MySQL 8)
```
Look for: Seq Scan / Full Table Scan on big tables, estimated vs actual rows, join type, sort spills, index-only scan. In MySQL watch `type` (`ALL` is a full scan; `ref`, `range`, `const` are good), `key`, `rows`, `Extra` (`Using filesort`, `Using temporary`).

## 2. Query optimization checklist
1. Select only needed columns (not `SELECT *`).
2. Filter early and on indexed columns; keep predicates sargable (search-argument-able).
3. Avoid N+1 queries from the application (use a join or `IN` batch / ORM eager loading).
4. Use `EXISTS` for existence checks; avoid `DISTINCT` as a bug fix for bad joins.
5. Paginate with **keyset (seek) pagination** instead of large OFFSETs:
```sql
SELECT * FROM orders WHERE (created_at, id) < (:last_created, :last_id) ORDER BY created_at DESC, id DESC LIMIT 20;
```
6. Batch inserts; use `COPY`/bulk loaders; commit in chunks.
7. Denormalize or cache (Redis) hot read paths; add read replicas.
8. Partition huge tables (range by date, hash by key); archive old data.
9. Keep statistics fresh; fix data types; use connection pooling.

## 3. Transactions and ACID
A transaction is a unit of work: all statements succeed or none do.
```sql
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;      -- or ROLLBACK
```
| Property | Meaning | Mechanism |
|---|---|---|
| Atomicity | all or nothing | undo log / rollback |
| Consistency | constraints always hold | constraints, triggers, application rules |
| Isolation | concurrent transactions do not see each other's partial work | locks, MVCC |
| Durability | committed data survives crashes | write-ahead log (WAL), fsync |

### Concurrency problems
| Problem | What happens |
|---|---|
| Dirty read | read uncommitted data of another transaction |
| Non-repeatable read | same row read twice gives different values |
| Phantom read | same range query returns different rows |
| Lost update | two transactions overwrite each other's changes |
| Write skew | two transactions each read overlapping data and write disjoint rows, breaking an invariant |

### Isolation levels (SQL standard)
| Level | Dirty | Non-repeatable | Phantom |
|---|---|---|---|
| READ UNCOMMITTED | possible | possible | possible |
| READ COMMITTED | no | possible | possible |
| REPEATABLE READ | no | no | possible (MySQL InnoDB prevents most via gap locks; PostgreSQL's snapshot isolation prevents them) |
| SERIALIZABLE | no | no | no |

Defaults: PostgreSQL READ COMMITTED; MySQL InnoDB REPEATABLE READ.

### MVCC
Multi-Version Concurrency Control keeps several versions of a row so readers do not block writers. PostgreSQL stores versions in the table (VACUUM cleans them); InnoDB uses undo logs.

### Locking
- **Shared (read) vs exclusive (write)** locks; row-level vs table-level.
- `SELECT ... FOR UPDATE` locks selected rows until commit (pessimistic locking). `FOR UPDATE SKIP LOCKED` is the standard pattern for job queues.
- **Optimistic locking:** add a `version` column; `UPDATE ... SET ..., version = version + 1 WHERE id = ? AND version = ?` and check affected rows (compare-and-set).
- **Deadlock:** each transaction waits for a lock the other holds; the DB aborts one. Reduce with consistent lock ordering, short transactions, retry on deadlock error.
- **Idempotency** for retried writes: unique key on an idempotency token with `ON CONFLICT DO NOTHING`.

### Double-booking example (ties to your Healthcare project)
```sql
UPDATE slots SET status='held', version=version+1, held_by=:u, hold_until=:t
WHERE id=:slot AND status='free' AND version=:v;       -- 1 row affected = you won; 0 = someone else did
```

## 4. Normalization
| Form | Rule |
|---|---|
| 1NF | atomic values, no repeating groups |
| 2NF | 1NF and no partial dependency on part of a composite key |
| 3NF | 2NF and no transitive dependency (non-key depends on non-key) |
| BCNF | every determinant is a candidate key |
| 4NF / 5NF | multivalued and join dependencies |

Normalize to remove redundancy and update anomalies. **Denormalize deliberately** for read performance (precomputed totals, duplicated names), accepting write complexity.

Anomalies: insertion, update, deletion.

## 5. Keys
Super key, candidate key, primary key, alternate key, composite key, foreign key, surrogate key (auto id / UUID) vs natural key (email). UUIDv7 or ULID keep indexes more insert-friendly than random UUIDv4.

## 6. Stored procedures, functions, triggers
- **Stored procedure:** saved SQL routine, can have side effects. **Function:** returns a value, usable in queries.
- **Trigger:** runs automatically on INSERT/UPDATE/DELETE (audit logs, derived data). Hidden logic can hurt maintainability and performance.

## 7. SQL vs NoSQL, scaling
| | SQL | NoSQL (document, key-value, wide-column, graph) |
|---|---|---|
| Schema | fixed | flexible |
| Transactions | strong ACID | varies, often limited |
| Scaling | vertical first, read replicas, sharding is manual | designed for horizontal scale |
| Best for | relational, consistent data | massive scale, flexible or denormalized access patterns |

**Replication:** primary plus replicas (async lag means stale reads). **Sharding:** split data across nodes by key (hash or range); cross-shard joins and transactions are hard. **CAP:** under a partition choose consistency or availability.

## 8. Connection pooling and ORMs
Pool connections (PgBouncer, SQLAlchemy pool) because connections are expensive. ORMs (SQLAlchemy, Django ORM) speed development; watch for N+1, lazy loading, unbounded `SELECT *`, and use `EXPLAIN`.

## Quick interview Q&A
- **What is an index and how does it work?** A B+ tree on column(s) so lookups are O(log n) instead of O(n).
- **Clustered vs non-clustered?** Rows physically ordered by the index vs a separate structure pointing to rows.
- **Why might a query ignore an index?** See section 1.
- **What are ACID properties?** Section 3.
- **Explain isolation levels and the anomalies each prevents.** Section 3 table.
- **Optimistic vs pessimistic locking?** Version check at write time vs lock rows up front.
- **How do you avoid double booking?** Atomic conditional update (compare-and-set), `FOR UPDATE`, or a unique constraint, plus idempotency keys.
- **What is a deadlock and how do you handle it?** Circular lock wait; consistent ordering, short transactions, retry.
- **What is normalization? When denormalize?** Remove redundancy; denormalize for read-heavy hot paths.
- **How do you paginate a huge table?** Keyset pagination.
- **How do you find slow queries?** Slow query log, `pg_stat_statements`, `EXPLAIN ANALYZE`.
- **What is WAL?** Changes are written to a log before data files so the database can recover after a crash.
