# Indexing — B+ Trees, Types, and Query Optimization

---

## 1. Why Indexes?

Without an index, finding a row requires a **full table scan** — O(n) disk reads. An index on the search column turns this into O(log n).

**Trade-off:** Indexes speed up reads but slow down writes (must update index on INSERT/UPDATE/DELETE) and consume disk space.

---

## 2. B+ Tree Index (Most Common)

A balanced tree where:
- All **data** lives in **leaf nodes** only
- Leaf nodes are linked in a **doubly linked list** (enables range scans)
- Internal nodes store keys only (for routing)
- Tree height is O(log_B n), where B = branching factor (~100-1000 for typical block size)

```
Internal nodes:   [20 | 40 | 60]
                 /    |    |    \
Leaf nodes: [10,15] [20,30] [40,50] [60,70,80]
            ←→       ←→       ←→       ←→        (doubly linked)
```

**Height example:** 1 million rows, B=200 → height ≈ 3. Only 3 disk reads to find any row!

---

## 3. Clustered vs Non-Clustered Index

| | Clustered | Non-Clustered |
|---|-----------|---------------|
| Data rows | Stored **in** index order | Stored separately |
| Per table | Only **one** (usually PK) | Many allowed |
| Lookup | Direct (index leaf = row) | Two steps: index → row pointer → row |
| Range scan | Very efficient | Less efficient (scattered row reads) |
| Row update (changing key) | Expensive (move row) | Only update index |

**InnoDB (MySQL):** PK = clustered index. Secondary indexes store the PK value as the row pointer.
**PostgreSQL:** Heap-based; `CLUSTER` command physically orders rows, but non-persistent.

---

## 4. Index Types

| Type | Description | Use Case |
|------|-------------|----------|
| B-Tree | Balanced tree, default | Equality, range queries, ORDER BY |
| Hash | Hash map, O(1) exact lookup | Only equality (`=`), no range |
| Bitmap | Bit array per value | Low-cardinality columns (gender, status) |
| Full-text | Inverted index on words | `LIKE '%word%'`, text search |
| Spatial | R-tree for 2D/3D data | Geo queries (PostGIS) |
| Composite | Index on multiple columns | Covers multi-column WHERE / ORDER BY |
| Covering | Index includes all needed columns | Avoids row lookup entirely |

---

## 5. Composite Index Column Order

```sql
CREATE INDEX idx ON orders (customer_id, order_date, status);
```

**Leftmost prefix rule:** Index usable if WHERE uses leftmost columns:

| Query | Uses Index? |
|-------|------------|
| `WHERE customer_id = 5` | ✅ (prefix match) |
| `WHERE customer_id = 5 AND order_date > '2024-01-01'` | ✅ |
| `WHERE customer_id = 5 AND status = 'OPEN'` | ✅ for customer_id, ❌ for status (gap) |
| `WHERE order_date > '2024-01-01'` | ❌ (skips first column) |
| `WHERE customer_id = 5 ORDER BY order_date` | ✅ avoids filesort |

---

## 6. Covering Index

An index that contains **all columns** a query needs. No need to look up the actual row (index-only scan).

```sql
-- Query
SELECT order_date, total FROM orders WHERE customer_id = 5;

-- Covering index
CREATE INDEX idx_cust_date_total ON orders (customer_id, order_date, total);
-- All three needed columns are in the index → zero row lookups
```

---

## 7. When NOT to Index

- **High-cardinality but rarely queried** — wasted space
- **Very small tables** — full scan is faster than index + row lookup
- **Columns with many NULLs** — NULLs usually not indexed (B-tree) or waste space
- **Columns in frequent bulk inserts** — index maintenance overhead dominates
- **Function on the column in WHERE:** `WHERE YEAR(created_at) = 2024` doesn't use index on `created_at`; rewrite as range.

---

## 8. Query Optimization Tips

```sql
-- BAD: function prevents index use
SELECT * FROM users WHERE UPPER(name) = 'ALICE';
-- GOOD: use a functional index (Postgres)
CREATE INDEX idx_upper_name ON users (UPPER(name));

-- BAD: leading wildcard can't use B-tree index
SELECT * FROM products WHERE name LIKE '%phone%';
-- GOOD: use full-text search
SELECT * FROM products WHERE to_tsvector(name) @@ to_tsquery('phone');

-- Check actual plan
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
-- Look for: Seq Scan (bad on large table), Index Scan (good), Index Only Scan (best)
```

---

## 9. Index Internals — Space Analysis

```
Page size: 16KB
Row size: 128 bytes → 128 rows/page
1M rows → ~8000 data pages → needs 8000*128/page_size disk reads for full scan

B-tree index (int key, 8 byte pointer) → ~1000 keys/page
1M rows → 3 levels: root(1) + internal(1000) + leaf(1M/1000=1000)
Lookup: 3 disk reads vs 8000
```

---

## 10. Quick Reference

| Question | Answer |
|----------|--------|
| Why B+ tree not B-tree? | B+ tree data only in leaves → higher branching → lower height; linked leaves enable range scans |
| Difference: index scan vs index-only scan? | Index scan: index + row lookup; index-only scan: all data in index (covering) |
| What is a clustered index? | Physical ordering of rows matches index order; only one per table |
| When does MySQL use a full scan despite index? | selectivity too low (e.g., 50% of rows match); optimizer chooses scan |
| What is index selectivity? | Distinct values / total rows; high selectivity → more useful index |
