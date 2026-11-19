# SQL Joins, CTEs and Window Functions: Interview Notes

## 1. Joins
```sql
-- INNER: only matching rows
SELECT e.name, d.name FROM employees e JOIN departments d ON e.dept_id = d.id;
-- LEFT: all left rows, NULLs for no match
SELECT c.name, o.id FROM customers c LEFT JOIN orders o ON o.customer_id = c.id;
-- RIGHT: mirror of LEFT.  FULL OUTER: all rows from both (not in MySQL; emulate with UNION)
-- CROSS: cartesian product (m x n rows)
SELECT * FROM sizes CROSS JOIN colors;
-- SELF: join a table to itself
SELECT e.name AS emp, m.name AS manager FROM employees e LEFT JOIN employees m ON e.manager_id = m.id;
```
| Join | Returns |
|---|---|
| INNER | rows with a match on both sides |
| LEFT | all left rows plus matches |
| RIGHT | all right rows plus matches |
| FULL | all rows from both |
| CROSS | every combination |
| NATURAL | joins on same-named columns (avoid) |
| SEMI (EXISTS) / ANTI (NOT EXISTS) | filter by existence, no columns from the other table |

**Filter placement matters with outer joins:** a condition on the right table in WHERE turns a LEFT JOIN into an INNER JOIN. Put it in the ON clause to keep unmatched left rows.
```sql
-- customers with no orders (anti-join)
SELECT c.* FROM customers c LEFT JOIN orders o ON o.customer_id = c.id WHERE o.id IS NULL;
SELECT c.* FROM customers c WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```
**Join algorithms:** nested loop (good with an index on the inner side), hash join (large unsorted sets, equality joins), merge join (both sides sorted).
**Many-to-many** needs a junction table: `student_course(student_id, course_id, PRIMARY KEY(student_id, course_id))`.
**Join duplication trap:** joining a one-to-many then summing the "one" side double counts. Aggregate first, then join.

## 2. CTEs (WITH)
```sql
WITH dept_avg AS (
  SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id
)
SELECT e.name, e.salary, d.avg_sal
FROM employees e JOIN dept_avg d ON e.dept_id = d.dept_id
WHERE e.salary > d.avg_sal;
```
CTEs improve readability and can be chained. In PostgreSQL 12+ non-recursive CTEs are inlined unless `MATERIALIZED`.

### Recursive CTE (hierarchies, org charts, series)
```sql
WITH RECURSIVE chain AS (
  SELECT id, name, manager_id, 1 AS lvl FROM employees WHERE manager_id IS NULL   -- anchor
  UNION ALL
  SELECT e.id, e.name, e.manager_id, c.lvl + 1
  FROM employees e JOIN chain c ON e.manager_id = c.id                              -- recursive step
)
SELECT * FROM chain ORDER BY lvl;
-- generate dates / numbers
WITH RECURSIVE n(i) AS (SELECT 1 UNION ALL SELECT i + 1 FROM n WHERE i < 10) SELECT i FROM n;
```
Guard against cycles with a depth limit or a visited path.

## 3. Window functions
A window function computes over a set of rows related to the current row **without collapsing them** (unlike GROUP BY).
```sql
function() OVER (PARTITION BY col ORDER BY col [frame])
```
### Ranking
```sql
SELECT name, dept_id, salary,
  ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn,   -- 1,2,3,4 unique
  RANK()       OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk,  -- 1,2,2,4 gaps after ties
  DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS drnk, -- 1,2,2,3 no gaps
  NTILE(4)     OVER (ORDER BY salary DESC) AS quartile
FROM employees;
```
### Offset functions
```sql
LAG(amount, 1)  OVER (PARTITION BY customer_id ORDER BY created_at)   -- previous row
LEAD(amount, 1) OVER (...)                                            -- next row
FIRST_VALUE(x) OVER (...), LAST_VALUE(x) OVER (... ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```
### Aggregates as windows
```sql
SUM(amount) OVER (PARTITION BY customer_id ORDER BY created_at)                          -- running total
AVG(amount) OVER (ORDER BY created_at ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)          -- 7-row moving average
SUM(amount) OVER ()                                                                      -- grand total on every row
amount * 100.0 / SUM(amount) OVER ()                                                     -- percent of total
```
### Frames
`ROWS` counts physical rows; `RANGE` groups rows with equal ORDER BY values. The default frame with ORDER BY is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which is why `LAST_VALUE` often surprises people.

### Where you can use them
Window functions are evaluated after WHERE/GROUP BY/HAVING, so you cannot filter on them directly. Wrap in a subquery or CTE:
```sql
WITH r AS (
  SELECT *, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rk FROM employees
)
SELECT * FROM r WHERE rk <= 3;       -- top 3 salaries per department
```
(`QUALIFY` exists in Snowflake/BigQuery.)

## 4. Pivot / unpivot with conditional aggregation
```sql
SELECT customer_id,
  SUM(CASE WHEN EXTRACT(MONTH FROM created_at) = 1 THEN amount END) AS jan,
  SUM(CASE WHEN EXTRACT(MONTH FROM created_at) = 2 THEN amount END) AS feb
FROM orders GROUP BY customer_id;
```

## 5. JSON, arrays, text search (practical extras)
- PostgreSQL JSONB: `data->'k'`, `data->>'k'`, `@>`, GIN index; MySQL JSON: `JSON_EXTRACT`, `->>`.
- Full-text: PostgreSQL `tsvector`/`tsquery`, MySQL `FULLTEXT`; use a search engine (Elasticsearch) for heavy needs.

## Quick interview Q&A
- **RANK vs DENSE_RANK vs ROW_NUMBER?** Gaps after ties, no gaps, always unique.
- **GROUP BY vs window function?** GROUP BY collapses rows; window keeps each row and adds an aggregate column.
- **How do you get the nth highest salary?** `DENSE_RANK() OVER (ORDER BY salary DESC)` and filter `= n`, or `LIMIT 1 OFFSET n-1` on distinct salaries.
- **INNER vs LEFT JOIN?** Matches only vs keeps all left rows.
- **How do you find rows in A that are not in B?** `LEFT JOIN ... WHERE b.id IS NULL` or `NOT EXISTS`.
- **What is a CTE, and when use a recursive one?** Named temporary result; recursive for trees, hierarchies, graph walks.
- **What is a self join?** Joining a table to itself, e.g. employee to manager.
- **Why does a WHERE on the right table break a LEFT JOIN?** It filters out the NULL-extended rows; move the condition into ON.
