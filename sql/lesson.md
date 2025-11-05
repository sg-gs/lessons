# SQL Queries & Efficiency

This lesson outlines the key points for a lesson on **query efficiency in SQL**, along with a **practical exercise** to analyze and optimize queries in a realistic scenario.

---

## Key Concepts (3–5’)

### 1. Query Execution Flow

Before optimizing, understand how SQL databases **execute a query**:

* **Parsing & Planning**: SQL is parsed into an execution plan.
* **Optimization**: The planner chooses the best way to retrieve data.
* **Execution**: The plan runs, using indexes, joins, scans, etc.

*Tool*: Use `EXPLAIN` or `EXPLAIN ANALYZE` (PostgreSQL/MySQL) to see how queries are executed.

---

### 2. Scans and Indexes

* **Full Table Scan**: The DB reads every row — simple but slow for large tables.
* **Index Scan**: Uses indexes to locate rows faster.
* **Sequential vs. Random I/O**: Index lookups can be fast for small subsets, but random disk access can hurt large queries.

📘 *Example*:

```sql
-- Full scan (no index)
SELECT * FROM users WHERE last_name = 'Smith';

-- Add an index
CREATE INDEX idx_users_lastname ON users(last_name);

-- Now the same query uses an index scan
SELECT * FROM users WHERE last_name = 'Smith';
```

---

### 3. Filtering and Sargability

A query is **sargable** (Search ARGument able) if the database can use an index to filter rows directly in the storage engine, instead of having to load all rows into memory and then apply the condition.

When a function or operation is applied **to a column**, the index on that column becomes unusable because the DB must compute the expression for every row. In contrast, when the comparison is **performed directly on the indexed value**, the index can be used to narrow the search space immediately.

⚠️ *Bad example* (non-sargable):

```sql
SELECT * FROM orders WHERE YEAR(order_date) = 2024;
```

Here, `YEAR(order_date)` is computed for every row — the database cannot use an index on `order_date` because the expression modifies the column’s value before comparison.

✅ *Better*:

```sql
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

This version is **sargable**: the database can use the index on `order_date` to directly locate the relevant rows in the range, avoiding a full table scan.

---

### 4. Joins and Their Cost

Different join algorithms have distinct performance characteristics depending on data size, indexes, and sorting.

* **Nested Loop Join**: For each row in the outer table, the DB searches matching rows in the inner table. Fast when the inner table is indexed or small, but can degrade to O(n²) for large datasets.

* **Hash Join**: The DB builds a hash table in memory for one dataset (usually the smaller one) and probes it for matches from the other dataset. Great for large, unsorted data but requires memory.

* **Merge Join**: Both tables must be sorted by the join key. The DB then scans both in order, merging matching rows efficiently. It’s ideal for large sorted inputs but expensive if sorting is needed first.

🧩 *Example*:

```sql
-- Expensive join (no index)
SELECT c.name, o.id
FROM customers c
JOIN orders o ON c.id = o.customer_id;

-- Improved (add index)
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```

By indexing `orders.customer_id`, the join can switch from a slow nested loop with full scans to a faster index lookup join.

---

### 5. Aggregations and Grouping

Aggregations (`COUNT`, `SUM`, `AVG`) often cause **temporary table scans** or **sorts**.
Using **partial indexes** or **materialized views** can help.

#### What Materialized Views Do

A **materialized view** is a precomputed result of a query stored as a physical table. Unlike a regular view (which runs the query every time), a materialized view saves the results, allowing future queries to read pre-aggregated data instantly. It’s ideal for dashboards, reporting, and analytics where data doesn’t change every second.

You can refresh it manually or automatically:

```sql
REFRESH MATERIALIZED VIEW user_country_counts;
```

📊 *Example*:

```sql
-- Inefficient
SELECT country, COUNT(*) FROM users GROUP BY country;

-- Pre-aggregate popular countries in a materialized view
CREATE MATERIALIZED VIEW user_country_counts AS
SELECT country, COUNT(*) as user_count FROM users GROUP BY country;
```

---

### 6. Query Optimization Techniques

* **Use `EXPLAIN ANALYZE`** to measure real execution time.
* **Avoid SELECT *** when you don’t need all columns.
* **Use proper indexes** for WHERE, JOIN, and ORDER BY clauses.
* **Denormalization** can sometimes improve read-heavy workloads.
* **Caching** at the app layer can avoid repetitive queries.

---

## Practical Exercise: Query Optimization on an E-commerce Database (15’)

### 🧩 Objective

You’ll analyze and optimize slow SQL queries on a simulated e-commerce database.

---

### 🧱 Dataset Description

Imagine three tables:

```sql
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name TEXT,
  country TEXT
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INT REFERENCES customers(id),
  order_date TIMESTAMP,
  total_amount NUMERIC
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INT REFERENCES orders(id),
  product_name TEXT,
  quantity INT,
  price NUMERIC
);
```

---

### 🔍 Part 1 – Identify Inefficiencies

Run these queries and analyze them using `EXPLAIN ANALYZE`:

```sql
-- Q1: Find customers with total spend > 5000
SELECT c.name, SUM(oi.quantity * oi.price) AS total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
GROUP BY c.name
HAVING SUM(oi.quantity * oi.price) > 5000;

-- Q2: Count orders from customers in 'USA' during 2024
SELECT COUNT(*)
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.country = 'USA' AND YEAR(o.order_date) = 2024;
```

👉 **Task:** Use `EXPLAIN ANALYZE` to find why they’re slow.
Note which parts of the plan take most of the time (full scan? join? sort?).

---

### ⚙️ Part 2 – Optimize

Propose and test improvements such as:

1. **Adding relevant indexes**:

   ```sql
   CREATE INDEX idx_orders_date ON orders(order_date);
   CREATE INDEX idx_customers_country ON customers(country);
   CREATE INDEX idx_order_items_order_id ON order_items(order_id);
   ```

2. **Making queries sargable**:

   ```sql
   WHERE o.order_date >= '2024-01-01' AND o.order_date < '2025-01-01'
   ```

3. **Aggregating smartly**:

   * Precompute customer total spend in a materialized view or summary table.
   * Cache frequent queries with Redis (discussion-level).

4. **Test again** with `EXPLAIN ANALYZE` and compare total runtime.

---

### 🧠 Wrap-up Discussion

Each team shares:

* Which queries improved and why.
* What type of optimization (index, rewrite, caching) helped the most.
* Trade-offs discovered (e.g., index write cost vs. read benefit).
* One additional optimization idea for a production environment.

---

### 📈 Deliverables

By the end of the session, each group should have:

* An annotated `EXPLAIN ANALYZE` output (before/after).
* A short note explaining **why** the optimized version is faster.
* At least one visualization or sketch of the **query plan flow**.

---

### 💡 Optional Challenge (if time allows)

Try to **rank top 5 customers by spend in the last 3 months**, using both:

1. A single heavy query.
2. A pre-aggregated monthly summary table.

Compare execution times — this teaches **batch vs. on-demand computation**.

---

## Instructor Notes: Why Part 1 Queries Are Inefficient

**Query 1**

* Performs **three joins** without indexes. The DB likely uses nested loops, reading all rows from each table.
* The `SUM(oi.quantity * oi.price)` forces a full aggregation across potentially millions of rows.
* The `HAVING` clause filters after aggregation — meaning all data must be processed first.

**Query 2**

* The `YEAR(order_date)` makes the condition non-sargable, forcing a **full scan** of the `orders` table.
* The join to `customers` uses `country` but lacks an index, so filtering by `'USA'` requires scanning the whole table.
* Overall result: no index usage, excessive scans, and redundant computations.

Optimizing both queries with indexes, sargable predicates, and pre-aggregations dramatically reduces execution time.
