# SQL Interview Notes (ELI15 to expert)

Compiled reference notes, unassessed (M0). **Est.** = checked against official PostgreSQL/MySQL documentation (see `SOURCES.md`); **Ex.** = standard knowledge, not verified in this pass. SQL varies by engine: examples use standard/PostgreSQL syntax; dialect differences are flagged. Sample tables: `employees(id, name, dept_id, manager_id, salary, hire_date)`, `departments(id, name)`, `customers(id, name, city)`, `orders(id, customer_id, amount, status, created_at)`, `logins(user_id, login_date)`.

---
## 1. Relational model and SELECT basics
**ELI15.** A database table is a spreadsheet with strict rules. SQL is how you ask questions: "show me these columns, from this table, only the rows like this, in this order."

**Foundations.** Tables hold rows; columns have types; a **primary key** uniquely identifies a row; a **foreign key** points to another table's key. Command groups: DDL (CREATE, ALTER, DROP, TRUNCATE), DML (INSERT, UPDATE, DELETE), DQL (SELECT), DCL (GRANT, REVOKE), TCL (BEGIN, COMMIT, ROLLBACK).
```sql
SELECT dept_id, COUNT(*) AS n
FROM employees
WHERE salary > 30000
GROUP BY dept_id
HAVING COUNT(*) > 2
ORDER BY n DESC
LIMIT 5 OFFSET 10;
```
**Logical execution order (Ex.).** FROM/JOIN, WHERE, GROUP BY, HAVING, window functions, SELECT (aliases created), DISTINCT, ORDER BY, LIMIT. This is why a SELECT alias cannot be used in WHERE, and why **window functions cannot appear in WHERE/GROUP BY/HAVING** (**Est.**: PostgreSQL docs: they run after those clauses and are allowed only in the SELECT list and ORDER BY).

**WHERE vs HAVING.** WHERE filters rows before grouping; HAVING filters groups after aggregation.
**DELETE vs TRUNCATE vs DROP (Ex.).** DELETE removes chosen rows (WHERE, per-row logging, triggers); TRUNCATE empties the table quickly and usually resets identity; DROP removes the table. Transactional behaviour of TRUNCATE varies by engine.

**Pitfalls.** `SELECT *` in production queries; `LIMIT` without `ORDER BY` returns an arbitrary set; `COUNT(*)` vs `COUNT(col)` confusion (see section 2).

---
## 2. NULL and three-valued logic
**ELI15.** NULL means "unknown", not zero or empty. Asking "is unknown equal to unknown?" gets the answer "unknown", not "yes".

**Foundations (Ex.).** Comparisons with NULL yield UNKNOWN; WHERE keeps only TRUE rows. Use `IS NULL`/`IS NOT NULL`. `COALESCE(a, b, ...)` returns the first non-NULL; `NULLIF(a, b)` returns NULL when they are equal (guards division by zero).
- Aggregates ignore NULLs: `COUNT(*)` counts rows, `COUNT(col)` counts non-NULL values, `COUNT(DISTINCT col)` distinct non-NULL.
- `x NOT IN (1, 2, NULL)` returns no rows (UNKNOWN); prefer `NOT EXISTS`.
- `UNION` treats NULLs as equal when deduplicating; `GROUP BY` groups NULLs together; ORDER BY NULL placement differs by engine (PostgreSQL: NULLS LAST for ASC by default, others first) (**Ex.**, verify per engine).

---
## 3. Joins
**ELI15.** A join lines up rows from two tables where they share something, like matching students to their classes by student ID.

**Foundations (Ex.).**
| Join | Returns |
|---|---|
| INNER | rows with a match in both tables |
| LEFT / RIGHT | all rows from one side, NULLs when no match |
| FULL OUTER | all rows from both (MySQL has no FULL JOIN; emulate with `UNION` of LEFT and RIGHT) |
| CROSS | every combination (m x n) |
| SELF | a table joined to itself (employee to manager) |
```sql
SELECT e.name, m.name AS manager
FROM employees e LEFT JOIN employees m ON e.manager_id = m.id;
-- customers who never ordered (anti-join)
SELECT c.* FROM customers c LEFT JOIN orders o ON o.customer_id = c.id WHERE o.id IS NULL;
SELECT c.* FROM customers c WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```
**Interview / expert.** A WHERE condition on the right table of a LEFT JOIN removes the NULL-extended rows and turns it into an INNER JOIN; put such conditions in `ON`. Join algorithms: nested loop (good with an index), hash join (large equality joins), merge join (sorted inputs). Joining a one-to-many then summing the "one" side double counts: aggregate first, then join. Many-to-many needs a junction table.

**Pitfalls.** Accidental cross joins (missing ON); duplicate rows from non-unique join keys; NULL join keys never match.

---
## 4. Aggregation: GROUP BY, HAVING, conditional aggregates
**Foundations.** `COUNT, SUM, AVG, MIN, MAX`; every selected non-aggregated column must be in `GROUP BY` (strict engines). Conditional aggregation pivots rows to columns:
```sql
SELECT customer_id,
  SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS paid_total,
  COUNT(*) FILTER (WHERE status = 'paid') AS paid_n          -- FILTER is PostgreSQL/SQLite
FROM orders GROUP BY customer_id;
```
`ROLLUP`, `CUBE`, `GROUPING SETS` produce subtotals (**Ex.**). `STRING_AGG` (PostgreSQL) / `GROUP_CONCAT` (MySQL) concatenate values.

---
## 5. Subqueries, CTEs, set operations
**Foundations (Ex.).** A subquery can be scalar, a list (`IN`), a table (derived table in FROM), or **correlated** (references the outer row, conceptually run per row; often rewritable as a join). **EXISTS** stops at the first match and is NULL-safe. A **CTE** (`WITH name AS (...)`) names an intermediate result; a **recursive CTE** walks hierarchies:
```sql
WITH RECURSIVE chain AS (
  SELECT id, name, manager_id, 1 AS lvl FROM employees WHERE manager_id IS NULL
  UNION ALL
  SELECT e.id, e.name, e.manager_id, c.lvl + 1 FROM employees e JOIN chain c ON e.manager_id = c.id
) SELECT * FROM chain;
```
Set operations: `UNION` (deduplicates), `UNION ALL` (keeps duplicates, faster), `INTERSECT`, `EXCEPT` (Oracle `MINUS`); column counts and types must match.

---
## 6. Window functions
**ELI15.** A window function looks at the other rows around each row to calculate something (rank, running total) while keeping every row visible.

**Foundations (Est., PostgreSQL docs).** A window function is marked by `OVER`. `PARTITION BY` splits rows into groups; `ORDER BY` inside `OVER` orders rows within each partition; with no `PARTITION BY` all rows form one partition. Unlike `GROUP BY`, window functions **do not collapse rows**. They are allowed only in the SELECT list and ORDER BY; to filter on one, compute it in a subquery or CTE and filter outside:
```sql
SELECT * FROM (
  SELECT depname, empno, salary,
         row_number() OVER (PARTITION BY depname ORDER BY salary DESC, empno) AS pos
  FROM empsalary) ss
WHERE pos < 3;                                 -- top two earners per department
```
**Common functions (Ex.).**
- Ranking: `ROW_NUMBER` (unique 1,2,3,4), `RANK` (ties share rank, gaps: 1,2,2,4), `DENSE_RANK` (ties, no gaps: 1,2,2,3), `NTILE(n)`.
- Offset: `LAG(x, n)`, `LEAD(x, n)`, `FIRST_VALUE`, `LAST_VALUE` (default frame ends at the current row; widen it with `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`).
- Aggregates as windows: running total `SUM(amount) OVER (PARTITION BY c ORDER BY created_at)`; moving average with `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW`; percent of total `amount * 100.0 / SUM(amount) OVER ()`.
Engine support: MySQL 8.0+, SQLite 3.25+ (**Ex.**, verify versions).

---
## 7. DDL, constraints, keys, views
**Foundations (Ex.).** Constraints: PRIMARY KEY (unique, not null, one per table, may be composite), UNIQUE (many allowed; NULL handling varies by engine), NOT NULL, CHECK, DEFAULT, FOREIGN KEY with `ON DELETE CASCADE | SET NULL | RESTRICT`. Key types: candidate, primary, composite, surrogate (auto id/UUID) vs natural (email). Views are stored queries; **materialised views** store results and must be refreshed. Money uses `DECIMAL(p,s)` or integer minor units, never FLOAT. Store timestamps in UTC.
```sql
CREATE TABLE orders (
  id BIGINT PRIMARY KEY, customer_id BIGINT NOT NULL REFERENCES customers(id) ON DELETE RESTRICT,
  amount DECIMAL(12,2) CHECK (amount >= 0), created_at TIMESTAMP NOT NULL DEFAULT now());
```
`ALTER TABLE` on big tables can lock; use online migrations (`CREATE INDEX CONCURRENTLY` in PostgreSQL).

---
## 8. DML: insert, update, delete, upsert
**Foundations (Ex.).** Multi-row `INSERT`, `INSERT ... SELECT`, `UPDATE ... WHERE`, `DELETE ... WHERE`; run the same `WHERE` as a SELECT first. **Upsert** atomically instead of select-then-insert (which races):
```sql
INSERT INTO counters(k, v) VALUES ('hits', 1) ON CONFLICT (k) DO UPDATE SET v = counters.v + 1;   -- PostgreSQL/SQLite
INSERT INTO counters(k, v) VALUES ('hits', 1) ON DUPLICATE KEY UPDATE v = v + 1;                  -- MySQL
```
Use `RETURNING` (PostgreSQL) to get changed rows. Delete duplicates by keeping `MIN(id)` per key or numbering rows with `ROW_NUMBER()` and deleting those above 1.

---
## 9. Indexes and query plans
**ELI15.** An index is the index at the back of a book: jump to the page instead of reading every page.

**Foundations.** Usually a **B-tree** (B+ tree): lookups O(log n) instead of a full scan; speeds reads, slows writes, uses space. Types: unique, composite, covering (index-only scan), partial/filtered (PostgreSQL), expression, hash, GIN/GiST for JSON, arrays, full text (PostgreSQL) (**Ex.**). Clustered index (rows stored in index order: InnoDB primary key, SQL Server) vs secondary.
**Composite index rule (Est., PostgreSQL docs).** A multicolumn B-tree index can serve conditions on any subset of its columns but works best on the **leading (leftmost) columns**: equality conditions on leading columns plus one inequality on the first column without an equality condition limit the scanned range; conditions on later columns are checked inside the index and avoid table visits but usually do not shrink the scanned range. PostgreSQL may use a **skip scan** on a later column when the leading column has few distinct values. Put equality columns first and the range column last; match your `ORDER BY`.
**Why an index may be ignored (Ex.).** Function or expression on the column (`WHERE YEAR(d) = 2026`); leading wildcard `LIKE '%x'`; implicit type conversion; low selectivity; stale statistics; many `OR`s across columns. Keep predicates sargable.
**Plans.** `EXPLAIN` shows the plan; `EXPLAIN ANALYZE` also runs it and shows real timings (PostgreSQL; MySQL 8 has `EXPLAIN ANALYZE` in recent 8.0 releases, verify). Look for sequential scans on large tables, bad row estimates, expensive sorts, join types.
**Optimisation checklist (Ex.).** Select fewer columns; filter early on indexed columns; avoid N+1 queries; use `EXISTS` for existence; **keyset pagination** (`WHERE (created_at, id) < (:c, :i) ORDER BY created_at DESC, id DESC LIMIT 20`) instead of large `OFFSET`; batch writes; update statistics; cache hot reads; partition huge tables.

---
## 10. Transactions, ACID, isolation, locking
**ELI15.** A transaction is a bundle of steps that must all happen or none, like moving money: take from A and give to B, never only one.

**Foundations.** ACID: atomicity (all or nothing), consistency (constraints hold), isolation (concurrent transactions do not see each other's partial work), durability (committed data survives crashes, via write-ahead logging). `BEGIN ... COMMIT / ROLLBACK`, `SAVEPOINT`.

**Isolation levels and anomalies (Est., PostgreSQL docs).**
| Level | Dirty read | Nonrepeatable read | Phantom read | Serialization anomaly |
|---|---|---|---|---|
| Read uncommitted | allowed by the standard, **not in PostgreSQL** | possible | possible | possible |
| Read committed | no | possible | possible | possible |
| Repeatable read | no | no | allowed by the standard, **not in PostgreSQL** | possible |
| Serializable | no | no | no | no |
PostgreSQL implements three distinct levels (Read Uncommitted behaves as Read Committed), defaults to **Read Committed** (each command sees a snapshot as of its start), and its Repeatable Read is snapshot isolation; applications must retry on serialization failures (`40001`). Serializable adds non-blocking predicate-lock monitoring.
**MySQL/InnoDB (Est.).** Default is **REPEATABLE READ**: plain consistent reads use the snapshot from the first read; locking reads, UPDATE and DELETE use gap/next-key locks to prevent phantoms; READ COMMITTED disables most gap locking so phantoms can occur. Defaults therefore differ between engines: state the engine in an answer.
**MVCC (Ex.).** Keeps several versions of a row so readers do not block writers.
**Locking (Ex.).** `SELECT ... FOR UPDATE` locks rows (pessimistic); `FOR UPDATE SKIP LOCKED` is the usual job-queue claim; **optimistic locking** uses a `version` column: `UPDATE ... SET ..., version = version + 1 WHERE id = ? AND version = ?` and check affected rows (compare-and-set). **Deadlock**: circular waits; avoid with consistent lock order, short transactions, retries.
```sql
-- safe money transfer: lock in a consistent order
BEGIN;
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
UPDATE accounts SET balance = balance - 500 WHERE id = 1 AND balance >= 500;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;
```
**Pitfalls.** Long-running transactions; assuming the same default isolation everywhere; read-modify-write in application code without locking or a conditional update.

---
## 11. Normalisation and design
**Foundations (Ex.).** 1NF atomic values; 2NF no partial dependency on part of a composite key; 3NF no transitive dependency; BCNF every determinant is a candidate key. Normalise to avoid insert/update/delete anomalies; denormalise deliberately for read performance (precomputed totals) accepting write complexity. Snapshot prices in `order_items` so history does not change. Soft deletes (`deleted_at`) need partial indexes and filtered views. SQL vs NoSQL: relational integrity and transactions vs flexible schema and horizontal scale; replication (lag) and sharding (cross-shard joins are hard).

---
## 12. Interview problem patterns (all Ex.; run on a sample dataset to verify)
```sql
-- 2nd / Nth highest salary
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;
-- duplicates
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
-- top 3 salaries per department
WITH r AS (SELECT *, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) rk FROM employees)
SELECT * FROM r WHERE rk <= 3;
-- running total
SELECT customer_id, created_at, SUM(amount) OVER (PARTITION BY customer_id ORDER BY created_at) AS running FROM orders;
-- month-over-month change
WITH m AS (SELECT DATE_TRUNC('month', created_at) mon, SUM(amount) rev FROM orders GROUP BY 1)
SELECT mon, rev, rev - LAG(rev) OVER (ORDER BY mon) AS diff FROM m;
-- consecutive-day streaks (gaps and islands): date minus row number is constant within a run
WITH d AS (SELECT DISTINCT user_id, login_date FROM logins),
g AS (SELECT user_id, login_date,
             login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) * INTERVAL '1 day' AS grp FROM d)
SELECT user_id, MIN(login_date) AS start_day, MAX(login_date) AS end_day, COUNT(*) AS streak FROM g GROUP BY user_id, grp;
-- overlapping bookings
SELECT a.id, b.id FROM bookings a JOIN bookings b
  ON a.resource_id = b.resource_id AND a.id < b.id AND a.start_time < b.end_time AND b.start_time < a.end_time;
```
Other staples: employees earning more than their manager (self join), products never sold (anti-join), first order per customer (`ROW_NUMBER() = 1`; `DISTINCT ON` is PostgreSQL-specific), median (`PERCENTILE_CONT` in PostgreSQL), pivot with conditional aggregation, retention (Day-1/Day-7), sessionisation (gaps over 30 minutes using `LAG` and a running sum of new-session flags).

**Checklist of question topics.** WHERE vs HAVING; execution order; joins and the LEFT JOIN filter trap; NULL behaviour and `NOT IN`; `UNION` vs `UNION ALL`; DELETE/TRUNCATE/DROP; PK vs UNIQUE; EXISTS vs IN; correlated subquery; CTE and recursive CTE; ROW_NUMBER/RANK/DENSE_RANK; LAG/LEAD; running totals; top-N per group; duplicates; indexes (B-tree, composite order, covering, why ignored); EXPLAIN; ACID and isolation levels (name the engine); deadlocks and optimistic vs pessimistic locking; keyset vs OFFSET pagination; normalisation vs denormalisation; upsert.
