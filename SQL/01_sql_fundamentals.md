# SQL Fundamentals: Interview Notes

Examples use standard SQL; MySQL / PostgreSQL / SQLite differences are noted. Sample tables:
```
employees(id, name, dept_id, manager_id, salary, hire_date)
departments(id, name)
orders(id, customer_id, amount, status, created_at)
customers(id, name, city)
```

## 1. Categories of SQL
| Group | Commands |
|---|---|
| DDL (structure) | CREATE, ALTER, DROP, TRUNCATE, RENAME |
| DML (data) | INSERT, UPDATE, DELETE, (MERGE / UPSERT) |
| DQL (query) | SELECT |
| DCL (permissions) | GRANT, REVOKE |
| TCL (transactions) | BEGIN, COMMIT, ROLLBACK, SAVEPOINT |

**DELETE vs TRUNCATE vs DROP:** DELETE removes rows (WHERE allowed, logged per row, triggers fire, can roll back); TRUNCATE removes all rows fast (resets identity, usually no triggers; rollback depends on the engine); DROP removes the table itself.

## 2. Creating tables and constraints
```sql
CREATE TABLE employees (
  id          INT PRIMARY KEY AUTO_INCREMENT,       -- SERIAL / GENERATED ... in PostgreSQL
  name        VARCHAR(100) NOT NULL,
  email       VARCHAR(255) UNIQUE,
  dept_id     INT,
  salary      DECIMAL(10,2) CHECK (salary >= 0),
  hire_date   DATE DEFAULT (CURRENT_DATE),
  manager_id  INT,
  FOREIGN KEY (dept_id) REFERENCES departments(id) ON DELETE SET NULL,
  FOREIGN KEY (manager_id) REFERENCES employees(id)
);
```
Constraints: PRIMARY KEY (unique and not null, one per table), UNIQUE, NOT NULL, CHECK, FOREIGN KEY (with ON DELETE CASCADE / SET NULL / RESTRICT), DEFAULT.

**Primary key vs unique key:** one primary key per table and no NULLs; many unique keys and (usually) one NULL allowed.

## 3. SELECT basics and logical order of execution
```sql
SELECT DISTINCT dept_id, COUNT(*) AS n
FROM employees
WHERE salary > 30000
GROUP BY dept_id
HAVING COUNT(*) > 2
ORDER BY n DESC
LIMIT 5 OFFSET 10;
```
**Logical processing order:** FROM / JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT (aliases defined here) -> DISTINCT -> ORDER BY -> LIMIT. This is why you cannot use a SELECT alias in WHERE (but you can in ORDER BY in most engines).

**WHERE vs HAVING:** WHERE filters rows before grouping; HAVING filters groups after aggregation.

## 4. Operators and expressions
```sql
WHERE salary BETWEEN 30000 AND 60000      -- inclusive
WHERE name LIKE 'A%'  -- % any chars, _ one char
WHERE dept_id IN (1, 2, 3)
WHERE manager_id IS NULL                  -- never = NULL
SELECT CASE WHEN salary > 80000 THEN 'high' WHEN salary > 40000 THEN 'mid' ELSE 'low' END AS band;
SELECT COALESCE(manager_id, 0), NULLIF(a, 0), IFNULL(x, 'n/a');   -- IFNULL is MySQL
SELECT CAST(salary AS INT), salary::int;    -- :: is PostgreSQL
```
Strings: `CONCAT`, `SUBSTRING`, `LENGTH`, `TRIM`, `UPPER`, `REPLACE`. Dates: `NOW()`, `DATE_TRUNC('month', d)` (PostgreSQL), `DATE_FORMAT`/`EXTRACT`, `DATEDIFF`, interval arithmetic.

## 5. NULL semantics (very common question)
- NULL means unknown. `NULL = NULL` is unknown (not true); use `IS NULL`.
- Aggregates ignore NULL (`COUNT(col)` skips NULLs, `COUNT(*)` counts rows). `AVG` ignores NULLs.
- `NOT IN` with a NULL in the list returns no rows. Prefer `NOT EXISTS`.
- `ORDER BY` NULL position differs: PostgreSQL puts NULLs last on ASC; MySQL first.
- Three-valued logic: TRUE, FALSE, UNKNOWN. WHERE keeps only TRUE.

## 6. Aggregates and GROUP BY
`COUNT, SUM, AVG, MIN, MAX`, plus `GROUP_CONCAT` (MySQL) / `STRING_AGG` (PostgreSQL), `COUNT(DISTINCT x)`.
- Every non-aggregated selected column must be in GROUP BY (strict engines).
- `GROUP BY ROLLUP(a, b)`, `CUBE`, `GROUPING SETS` produce subtotals.
- Conditional aggregation:
```sql
SELECT dept_id,
       SUM(CASE WHEN status='paid' THEN amount ELSE 0 END) AS paid_total,
       COUNT(*) FILTER (WHERE status='paid') AS paid_n      -- PostgreSQL
FROM orders GROUP BY dept_id;
```

## 7. Subqueries
```sql
-- scalar
SELECT name FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);
-- IN / EXISTS
SELECT name FROM customers c WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
-- correlated: runs per outer row (optimizers often rewrite as a join)
SELECT name FROM employees e WHERE salary > (SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id);
-- derived table
SELECT * FROM (SELECT dept_id, AVG(salary) a FROM employees GROUP BY dept_id) t WHERE a > 50000;
```
**IN vs EXISTS:** EXISTS stops at the first match and handles NULLs safely; IN is fine for small static lists.

## 8. Set operations
`UNION` (distinct, sorted/deduped), `UNION ALL` (keeps duplicates, faster), `INTERSECT`, `EXCEPT` (`MINUS` in Oracle). Column count and types must match.

## 9. INSERT / UPDATE / DELETE / UPSERT
```sql
INSERT INTO employees (name, dept_id) VALUES ('Asha', 2), ('Ravi', 3);
INSERT INTO archive SELECT * FROM employees WHERE hire_date < '2015-01-01';
UPDATE employees SET salary = salary * 1.1 WHERE dept_id = 2;
UPDATE e SET e.salary = d.bonus FROM employees e JOIN depts d ON ...;   -- vendor syntax varies
DELETE FROM employees WHERE id IN (SELECT id FROM temp);
-- upsert
INSERT INTO counters(k, v) VALUES ('hits', 1)
ON CONFLICT (k) DO UPDATE SET v = counters.v + 1;                       -- PostgreSQL / SQLite
INSERT INTO counters(k, v) VALUES ('hits', 1)
ON DUPLICATE KEY UPDATE v = v + 1;                                      -- MySQL
```
Always run a SELECT with the same WHERE before UPDATE/DELETE. Use transactions.

## 10. ALTER
```sql
ALTER TABLE employees ADD COLUMN phone VARCHAR(20);
ALTER TABLE employees DROP COLUMN phone;
ALTER TABLE employees ALTER COLUMN salary TYPE NUMERIC(12,2);     -- PostgreSQL
ALTER TABLE employees ADD CONSTRAINT fk FOREIGN KEY (dept_id) REFERENCES departments(id);
CREATE INDEX idx_emp_dept ON employees(dept_id);
```
On big tables, schema changes can lock; use online migration tools (pt-online-schema-change, `CREATE INDEX CONCURRENTLY` in PostgreSQL).

## 11. Views
```sql
CREATE VIEW high_paid AS SELECT id, name, salary FROM employees WHERE salary > 80000;
```
A view is a stored query; it does not store data (a **materialized view** does and must be refreshed). Use views for abstraction and security.

## 12. Data types to know
INT/BIGINT, DECIMAL (money: never FLOAT), VARCHAR/TEXT, BOOLEAN, DATE/TIMESTAMP (with or without time zone), JSON/JSONB, UUID, ENUM, BLOB.
Store timestamps in UTC.

## Quick interview Q&A
- **Difference between WHERE and HAVING?** Rows before grouping vs groups after.
- **COUNT(*) vs COUNT(col) vs COUNT(DISTINCT col)?** All rows; non-null values; distinct non-null values.
- **UNION vs UNION ALL?** Dedup vs keep duplicates.
- **Why is `= NULL` wrong?** Comparison with NULL is unknown; use `IS NULL`.
- **DELETE vs TRUNCATE vs DROP?** See section 1.
- **What is a foreign key?** A column referencing another table's key to enforce referential integrity.
- **What is a view?** A named stored query; no data of its own.
- **Primary key vs unique?** See section 2.
- **Can a table have two primary keys?** One primary key, possibly composite (several columns).
- **What is ON DELETE CASCADE?** Deleting the parent row deletes its child rows.
