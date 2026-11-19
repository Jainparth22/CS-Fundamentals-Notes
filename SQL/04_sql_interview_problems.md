# SQL Interview Problems (with solutions)

Schema used:
```
employees(id, name, dept_id, manager_id, salary, hire_date)
departments(id, name)
customers(id, name, city)
orders(id, customer_id, amount, status, created_at)
order_items(order_id, product_id, qty, price)
products(id, name, category)
logins(user_id, login_date)
```

## Easy
1. **Second highest salary**
```sql
SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);
-- nth highest (works with ties): 
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET n-1;
```
2. **Find duplicate emails**
```sql
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
```
3. **Delete duplicates, keep the lowest id**
```sql
DELETE FROM users WHERE id NOT IN (SELECT MIN(id) FROM users GROUP BY email);
-- MySQL requires wrapping the subquery in a derived table; or use ROW_NUMBER():
WITH d AS (SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) rn FROM users)
DELETE FROM users WHERE id IN (SELECT id FROM d WHERE rn > 1);
```
4. **Customers who never ordered**
```sql
SELECT c.* FROM customers c LEFT JOIN orders o ON o.customer_id = c.id WHERE o.id IS NULL;
```
5. **Employees earning more than their manager**
```sql
SELECT e.name FROM employees e JOIN employees m ON e.manager_id = m.id WHERE e.salary > m.salary;
```
6. **Department-wise employee count including empty departments**
```sql
SELECT d.name, COUNT(e.id) FROM departments d LEFT JOIN employees e ON e.dept_id = d.id GROUP BY d.id, d.name;
```
7. **Employees with no department**
```sql
SELECT * FROM employees WHERE dept_id IS NULL;
```

## Medium
8. **Top 3 salaries in each department**
```sql
WITH r AS (SELECT *, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) rk FROM employees)
SELECT * FROM r WHERE rk <= 3;
```
9. **Departments whose average salary exceeds the company average**
```sql
SELECT dept_id, AVG(salary) FROM employees GROUP BY dept_id HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```
10. **Running total of orders per customer**
```sql
SELECT customer_id, created_at, amount, SUM(amount) OVER (PARTITION BY customer_id ORDER BY created_at) AS running
FROM orders;
```
11. **Month-over-month revenue growth**
```sql
WITH m AS (SELECT DATE_TRUNC('month', created_at) mon, SUM(amount) rev FROM orders GROUP BY 1)
SELECT mon, rev, rev - LAG(rev) OVER (ORDER BY mon) AS diff,
       ROUND(100.0 * (rev - LAG(rev) OVER (ORDER BY mon)) / LAG(rev) OVER (ORDER BY mon), 2) AS pct
FROM m;
```
12. **Customers with orders in consecutive months / consecutive login days (gaps and islands)**
```sql
WITH d AS (SELECT DISTINCT user_id, login_date FROM logins),
g AS (SELECT user_id, login_date,
             login_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date)) * INTERVAL '1 day' AS grp
      FROM d)
SELECT user_id, MIN(login_date) AS start_day, MAX(login_date) AS end_day, COUNT(*) AS streak
FROM g GROUP BY user_id, grp ORDER BY streak DESC;
```
The idea: dates minus their row number are constant inside a consecutive run. (MySQL: `DATE_SUB(login_date, INTERVAL rn DAY)`.)
13. **Users who logged in 3 days in a row**
```sql
SELECT DISTINCT a.user_id FROM logins a
JOIN logins b ON a.user_id = b.user_id AND b.login_date = a.login_date + 1
JOIN logins c ON a.user_id = c.user_id AND c.login_date = a.login_date + 2;
```
14. **Top customer by total spend per city**
```sql
WITH t AS (SELECT c.city, c.name, SUM(o.amount) tot,
                  RANK() OVER (PARTITION BY c.city ORDER BY SUM(o.amount) DESC) rk
           FROM customers c JOIN orders o ON o.customer_id = c.id GROUP BY c.city, c.id, c.name)
SELECT * FROM t WHERE rk = 1;
```
15. **Products never sold**
```sql
SELECT p.* FROM products p WHERE NOT EXISTS (SELECT 1 FROM order_items i WHERE i.product_id = p.id);
```
16. **Median salary** (PostgreSQL)
```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) FROM employees;
-- portable: average of the middle one or two rows via ROW_NUMBER and COUNT
```
17. **Pivot: monthly revenue by category** (conditional aggregation, see joins notes).
18. **Percent of total per category**
```sql
SELECT category, SUM(amount) AS s, 100.0 * SUM(amount) / SUM(SUM(amount)) OVER () AS pct FROM sales GROUP BY category;
```
19. **Employees hired in the last 90 days / each month's hires**
```sql
SELECT * FROM employees WHERE hire_date >= CURRENT_DATE - INTERVAL '90 days';
SELECT DATE_TRUNC('month', hire_date), COUNT(*) FROM employees GROUP BY 1 ORDER BY 1;
```
20. **First order per customer**
```sql
SELECT DISTINCT ON (customer_id) * FROM orders ORDER BY customer_id, created_at;       -- PostgreSQL
-- portable: ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at) = 1
```

## Hard
21. **Organization hierarchy with depth and path** (recursive CTE, see joins notes).
22. **Swap seats of adjacent students**
```sql
SELECT CASE WHEN id % 2 = 1 AND id = (SELECT MAX(id) FROM seat) THEN id
            WHEN id % 2 = 1 THEN id + 1 ELSE id - 1 END AS id, student
FROM seat ORDER BY id;
```
23. **Cumulative distinct customers per month** (set-based, usually window or self join).
24. **Rolling 7-day active users**
```sql
SELECT d, COUNT(DISTINCT user_id) FROM (
  SELECT c.d, l.user_id FROM calendar c JOIN logins l ON l.login_date BETWEEN c.d - 6 AND c.d
) x GROUP BY d;
```
25. **Retention: users who came back the day after signup**
```sql
SELECT COUNT(DISTINCT s.user_id) FILTER (WHERE l.login_date = s.signup_date + 1) * 1.0 / COUNT(DISTINCT s.user_id)
FROM signups s LEFT JOIN logins l ON l.user_id = s.user_id;
```
26. **Find overlapping bookings (double-booking audit)**
```sql
SELECT a.id, b.id FROM bookings a JOIN bookings b
  ON a.resource_id = b.resource_id AND a.id < b.id AND a.start_time < b.end_time AND b.start_time < a.end_time;
```
Also enforce with a PostgreSQL exclusion constraint: `EXCLUDE USING gist (resource_id WITH =, tstzrange(start_time, end_time) WITH &&)`.
27. **Job queue claim without conflicts**
```sql
UPDATE jobs SET status='running', worker=:w
WHERE id = (SELECT id FROM jobs WHERE status='pending' ORDER BY created_at LIMIT 1 FOR UPDATE SKIP LOCKED)
RETURNING *;
```
28. **Transfer money safely** (transaction with row locks, ordered by account id to avoid deadlock).
```sql
BEGIN;
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
UPDATE accounts SET balance = balance - 500 WHERE id = 1 AND balance >= 500;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;
```

## Conceptual questions with short answers
- **Find the 2nd highest without LIMIT/TOP?** Subquery with MAX, or `DENSE_RANK`.
- **Why can't you use an alias in WHERE?** SELECT is processed after WHERE.
- **Delete vs truncate?** See fundamentals.
- **What is a correlated subquery?** Inner query refers to outer row; runs per row (often rewritten as a join).
- **How do you count rows fast on a huge table?** Approximate counts from statistics, or a maintained counter; `COUNT(*)` full scans in some engines.
- **UNION ALL vs UNION performance?** UNION ALL is faster (no dedup sort).
- **How would you design a schema for orders and products?** `customers`, `orders`, `order_items` (many-to-many with qty and price snapshot), `products`; keep the price in `order_items` so history does not change.
- **How do you handle soft deletes?** `deleted_at` column plus partial indexes and filtered views.
- **How do you store money?** `DECIMAL(p, s)` or integer minor units, never FLOAT.

## Practice plan
Do 5 problems a day from this list, then repeat the hard ones from memory. Run each on a small dataset (SQLite/PostgreSQL) and read `EXPLAIN` output for the join and window problems.
