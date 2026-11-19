# Resources and references: SQL Interview

Accessed 19 Nov 2026.

| Title | Link | Used for | Status |
|---|---|---|---|
| PostgreSQL docs: Transaction Isolation | https://www.postgresql.org/docs/current/transaction-iso.html | Isolation-level table and anomalies; PostgreSQL implements three distinct levels; default Read Committed; Repeatable Read is snapshot isolation; retry on serialization failure | Opened |
| PostgreSQL docs: Window Functions (tutorial) | https://www.postgresql.org/docs/current/tutorial-window.html | `OVER`, `PARTITION BY`, rows not collapsed, allowed only in SELECT list and ORDER BY, filter via subquery | Opened |
| PostgreSQL docs: Multicolumn Indexes | https://www.postgresql.org/docs/current/indexes-multicolumn.html | Leading-column rule, equality plus one inequality, later columns checked inside the index, skip scan | Opened |
| MySQL 8.0 Reference: InnoDB Transaction Isolation Levels | https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html | Default REPEATABLE READ; consistent reads use the first-read snapshot; gap/next-key locks; READ COMMITTED disables gap locking | Opened; it does not restate the standard's definition of REPEATABLE READ |

Not read / not verified (marked **Ex.** in the notes): logical execution order beyond what the window docs imply, NULL ordering per engine, `FULL JOIN` and window-function support by MySQL/SQLite version, `FILTER`/`DISTINCT ON`/`ON CONFLICT` syntax across engines, `EXPLAIN ANALYZE` availability in MySQL, normal forms, MVCC internals, keyset pagination, and every sample query (not run against a database in this pass). SQL:2016 standard text was not consulted.
