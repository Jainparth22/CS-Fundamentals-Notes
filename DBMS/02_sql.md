# SQL — Joins, Window Functions & Classic Problems

---

## 1. Basic Query Structure (Order of Execution)

```sql
SELECT   columns          -- 6. Project columns
FROM     table            -- 1. Get rows
JOIN     other ON cond    -- 2. Join
WHERE    condition        -- 3. Filter rows
GROUP BY columns          -- 4. Group
HAVING   condition        -- 5. Filter groups
ORDER BY column           -- 7. Sort
LIMIT    n                -- 8. Limit rows
```

---

## 2. All Joins

```sql
-- INNER JOIN: only matching rows
SELECT e.name, d.name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT JOIN: all rows from left, NULLs for no match
SELECT e.name, d.name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- RIGHT JOIN: all rows from right
-- FULL OUTER JOIN: all rows from both (NULL if no match)
-- CROSS JOIN: cartesian product (every combination)
-- SELF JOIN: table joined with itself

-- Find employees with no department
SELECT e.name FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id
WHERE d.id IS NULL;
```

---

## 3. Aggregation & GROUP BY

```sql
SELECT dept_id, COUNT(*) as headcount, AVG(salary) as avg_sal
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 5          -- filter groups (not individual rows)
ORDER BY avg_sal DESC;
```

**Common aggregate functions:** `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `COUNT(DISTINCT col)`

---

## 4. Subqueries

```sql
-- Correlated subquery: employees earning more than their dept avg
SELECT name, salary, dept_id
FROM employees e
WHERE salary > (
    SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id
);

-- EXISTS
SELECT name FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.cust_id = c.id);

-- IN vs EXISTS: EXISTS short-circuits, better for large subqueries
```

---

## 5. Window Functions

Window functions operate over a set of rows related to the current row (the "window"), without collapsing rows like GROUP BY.

```sql
function() OVER (
    PARTITION BY column     -- separate windows per partition
    ORDER BY column         -- order within window
    ROWS/RANGE BETWEEN ...  -- frame specification
)
```

### Ranking Functions
```sql
SELECT name, salary, dept_id,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) as rn,
    RANK()       OVER (PARTITION BY dept_id ORDER BY salary DESC) as rnk,
    DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as drnk
FROM employees;

-- ROW_NUMBER: unique sequential (no ties)
-- RANK: gaps on ties (1,1,3)
-- DENSE_RANK: no gaps (1,1,2)
```

### Running Total / Moving Average
```sql
SELECT date, amount,
    SUM(amount) OVER (ORDER BY date) as running_total,
    AVG(amount) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as 7day_avg
FROM sales;
```

### LAG / LEAD
```sql
SELECT date, amount,
    LAG(amount, 1, 0)  OVER (ORDER BY date) as prev_day,
    LEAD(amount, 1, 0) OVER (ORDER BY date) as next_day,
    amount - LAG(amount) OVER (ORDER BY date) as day_over_day_change
FROM sales;
```

### NTILE
```sql
SELECT name, salary,
    NTILE(4) OVER (ORDER BY salary) as quartile
FROM employees;
```

---

## 6. CTEs (Common Table Expressions)

```sql
WITH dept_avg AS (
    SELECT dept_id, AVG(salary) as avg_sal
    FROM employees
    GROUP BY dept_id
),
top_earners AS (
    SELECT e.*, d.avg_sal
    FROM employees e
    JOIN dept_avg d ON e.dept_id = d.dept_id
    WHERE e.salary > d.avg_sal * 1.2
)
SELECT * FROM top_earners ORDER BY salary DESC;
```

---

## 7. Classic SQL Interview Problems

### 1. Second Highest Salary
```sql
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Or with DENSE_RANK (handles NULLs if only 1 distinct salary):
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as dr FROM employees
) WHERE dr = 2;
```

### 2. Employees Earning More Than Manager
```sql
SELECT e.name as employee, m.name as manager
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### 3. Duplicate Emails
```sql
SELECT email FROM users
GROUP BY email HAVING COUNT(*) > 1;
```

### 4. Consecutive Rows / Streaks
```sql
-- Find user IDs with 3+ consecutive logins
WITH numbered AS (
    SELECT user_id, login_date,
           login_date - INTERVAL ROW_NUMBER() OVER (
               PARTITION BY user_id ORDER BY login_date
           ) DAY as grp
    FROM logins
)
SELECT user_id FROM numbered
GROUP BY user_id, grp
HAVING COUNT(*) >= 3;
```

### 5. Department Top 3 Salaries
```sql
SELECT dept, name, salary FROM (
    SELECT d.name as dept, e.name, e.salary,
           DENSE_RANK() OVER (PARTITION BY d.id ORDER BY e.salary DESC) as dr
    FROM employees e JOIN departments d ON e.dept_id = d.id
) WHERE dr <= 3;
```

### 6. Delete Duplicates (keep one)
```sql
DELETE FROM emails
WHERE id NOT IN (
    SELECT MIN(id) FROM emails GROUP BY email
);
```

### 7. Pivot Table (manual)
```sql
SELECT
    SUM(CASE WHEN quarter = 'Q1' THEN revenue END) as Q1,
    SUM(CASE WHEN quarter = 'Q2' THEN revenue END) as Q2,
    SUM(CASE WHEN quarter = 'Q3' THEN revenue END) as Q3,
    SUM(CASE WHEN quarter = 'Q4' THEN revenue END) as Q4
FROM sales GROUP BY year;
```

### 8. Running Rank Reset by Condition
```sql
-- Rank products by sales within each category
SELECT category, product, sales,
    RANK() OVER (PARTITION BY category ORDER BY sales DESC) as cat_rank
FROM products;
```

### 9. Find Missing IDs
```sql
-- IDs 1 to N, find gaps
SELECT t1.id + 1 as missing_start
FROM table1 t1
WHERE NOT EXISTS (SELECT 1 FROM table1 t2 WHERE t2.id = t1.id + 1)
AND t1.id < (SELECT MAX(id) FROM table1);
```

### 10. Median Salary
```sql
-- PostgreSQL
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) as median
FROM employees;
```

---

## 8. Index Usage in Queries

```sql
-- Check query plan
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;

-- Index helps when: WHERE col = ?, ORDER BY col, JOIN ON col
-- Index doesn't help: functions on indexed col (WHERE YEAR(date)=2024 — bad)
-- Fix: WHERE date >= '2024-01-01' AND date < '2025-01-01'
```
