# Week 47 — Exercises: Indexes and Query Performance

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises walk you through measuring, creating, and evaluating indexes on your TrailShop database. You will use PostgreSQL's `EXPLAIN ANALYZE` to gather real execution data and make evidence-based decisions about indexing.

---

## 1. TrailShop Project Task

### Step 1 — Baseline Measurement

Before creating any indexes, run `EXPLAIN ANALYZE` on the following five queries. For each, record the **scan type**, **execution time (ms)**, and **rows removed by filter**.

```sql
-- Query A: Find a product by name
SELECT * FROM products WHERE name = 'Trail Runner Pro 3';

-- Query B: Get all items for a specific order
SELECT * FROM order_items WHERE order_id = 42;

-- Query C: Get all orders for a customer
SELECT * FROM orders WHERE customer_id = 7;

-- Query D: Get products in a category
SELECT * FROM product_categories WHERE category_id = 3;

-- Query E: Join order_items to products
SELECT oi.quantity, p.name, p.price
FROM order_items oi
JOIN products p ON p.id = oi.product_id
WHERE oi.order_id = 42;
```

Record your results in a table like this:

| Query | Scan Type   | Exec Time (ms) | Rows Removed by Filter |
| ----- | ----------- | -------------- | ---------------------- |
| A     | Seq Scan    | 1.2            | 10                     |
| B     | Bitmap Heap | 0.190          | -                      |
| C     | Seq Scan    | 0.493          | 8                      |
| D     | Bitmap Heap | 5.372          | -                      |
| E     | Hash Join   | 0.126          | -                      |

### Step 2 — Create Indexes

Now create the following indexes:

```sql
CREATE INDEX idx_product_name ON products (name);
CREATE INDEX idx_order_item_order_id ON order_items (order_id);
CREATE INDEX idx_order_item_product_id ON order_items (product_id);
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
CREATE INDEX idx_product_categories_category_id ON product_categories (category_id);
```

### Step 3 — Re-measure

Because the project database is so small, the PostgreSQL optimizer will most likely default to Seq Scan even after creating indexes.

You can force PostgreSQL to use indexes by disabling the Seq scan using following commands:

```sql
--- Turn the seq scan off
SET enable_seqscan = OFF;

--- Turn the seq scan back on
SET enable_seqscan = ON;
```

Run the same five queries with `EXPLAIN ANALYZE` again. Fill in your comparison table:

| Query | Before Scan | Before Time (ms) | After Scan  | After Time (ms) | Improvement Factor |
| ----- | ----------- | ---------------- | ----------- | --------------- | ------------------ |
| A     | Seq Scan    | 1.2              | Index Scan  | 0.148           | 8.1                |
| B     | Bitmap Heap | 0.190            | Index Scan  | 0.176           | 1.1                |
| C     | Seq Scan    | 0.493            | Index Scan  | 0.055           | 9                  |
| D     | Bitmap Heap | 5.372            | Index scan  | 0.084           | 64                 |
| E     | Hash Join   | 0.126            | nested loop | 0.071           | 1.8                |

### Step 4 — Recommendation

Write a paragraph (4–6 sentences) recommending which indexes to keep in production and why. Consider table size, query frequency, and write overhead. If any index showed minimal improvement, explain why you might drop it.

> [!NOTE]
> **_Your Answer_**
>
> I recommend keeping the indexes for Queries A, C, and D because they delivered substantial speedups between 8x and 64x. In contrast query B showed a negligible improvement of only 1.1x, making the additional write overhead on every frequent order insert unjustified.

---

## 2. Theory Review

Answer the following questions in your own words. See the corresponding theory sections for reference.

**Question 1** — What is a database index, and why is it needed? (See Theory Section 1)

> [!NOTE]
> **_Your Answer_**
>
> A database index provides a shortcut so the database doesn't have to read everything. It is needed mainly to safe time and to speed up reads.

**Question 2** — Explain the difference between B-tree search at O(log n) and sequential scan at O(n). If a table has 1,000,000 rows, approximately how many comparisons does each approach need? (See Theory Section 2)

> [!NOTE]
> **_Your Answer_**
>
> While B trees organizes the data into a tree structure with three levels of nodes (Root node, Internal node and leaf node) a seq scan starts at the first row of the table and checks every single row until it reaches the end. At a table with 1,000,000 rows, O(log n) would need ~ 20 lookups, O(n) 400ms.

**Question 3** — PostgreSQL automatically creates indexes on primary keys. Why doesn't it also auto-index foreign key columns? (See Theory Section 3)

> [!NOTE]
> **_Your Answer_**
>
> Because every foreign key refers to a primary key in the parent table. So PostgresSQL checks the primary keys in the parent table that refers to the foreign key to also minimize storage and write overhead.

**Question 4** — What is the difference between an Index Scan and an Index Only Scan? When can PostgreSQL use Index Only Scan? (See Theory Section 4)

> [!NOTE]
> **_Your Answer_**
>
> When executing an Index Scan, PostgresSQL traverses the index to find matching entries, then fetches each matching row from the table (the "heap"). If the index contains **all columns** needed by the query, PostgreSQL can skip the table entirely using an Index Only Scan. The Index Only Scan requires a check against the visbility map to ensure the data is up-to-date and is also the fastest scan type.

**Question 5** — Give three situations where you should NOT create an index. Explain each briefly. (See Theory Section 5)

> [!NOTE]
> **_Your Answer_**
>
> 1. Small tables: If a table only have few hundred rows, the overhead of maintaining an index is simply not worth. Also is a seq scan almost instant with this amount of rows.
> 2. Low selectivity columns: Indexing columns with very few distinct values (like boolean types) simply doesnt make any sense at all. Also it is slower than a seq scan.
> 3. Write-Heavy tables: Tables receiving a lot of inserts per second are not good for indexes, because every `NSERT`, `UPDATE`, and `DELETE` must also update **every index** on the table which is very ressource intensive. 

**Question 6** — What does `EXPLAIN ANALYZE` show that plain `EXPLAIN` does not? Why does this matter? (See Theory Section 6)

> [!NOTE]
> **_Your Answer_**
>
> `EXPLAIN ANALYZE` shows the plan and also the execution statistics. EXPLAIN only shows the plan, how SQL would execute the query. It matters, because if you just want to see, how SQL executes the query, but dont want to actually execute the query, you can only use EXPLAIN.

**Question 7** — What is a partial index? Give one concrete example where a partial index on a TrailShop table would be useful. (See Theory Section 7)

> [!NOTE]
> **_Your Answer_**
>
> A partial index only indexes rows that match a condition. This makes the index smaller and more focused. It is especially useful, when you freuquently query a specific subset of data, in a trailshop example for example pending orders.

**Question 8** — What is the leftmost prefix rule for multi-column indexes? If you create `CREATE INDEX idx ON orders (customer_id, order_date)`, which of the following queries can use the index and why?

```sql
-- a)
SELECT * FROM orders WHERE customer_id = 5;
-- b)
SELECT * FROM orders WHERE order_date = '2026-01-15';
-- c)
SELECT * FROM orders WHERE customer_id = 5 AND order_date = '2026-01-15';
```

(See Theory Section 8)

> [!NOTE]
> **_Your Answer_**
>
> Its a) and c). The rule says that it filters on the leftmost column of the index (in this case the customer_id). b) doesnt filter on the the leftmost column instead it skips it. 

---

## 3. EXPLAIN Reading Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Below are four `EXPLAIN ANALYZE` outputs. For each one, identify:

1. The scan type used
2. Estimated rows vs actual rows
3. Total execution time
4. Whether an index would help (and which column to index)

### Output A — Sequential Scan

```
Seq Scan on products  (cost=0.00..25.00 rows=5 width=72) (actual time=0.015..0.285 rows=3 loops=1)
  Filter: (category_id = 4)
  Rows Removed by Filter: 497
Planning Time: 0.065 ms
Execution Time: 0.312 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Would an index help? Which one?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Output B — Index Scan

```
Index Scan using idx_orders_customer_id on orders  (cost=0.28..8.30 rows=1 width=48) (actual time=0.021..0.024 rows=4 loops=1)
  Index Cond: (customer_id = 12)
Planning Time: 0.082 ms
Execution Time: 0.041 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Is the index effective here?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Output C — Bitmap Index Scan

```
Bitmap Heap Scan on order_items  (cost=4.32..35.76 rows=25 width=24) (actual time=0.045..0.098 rows=28 loops=1)
  Recheck Cond: (product_id = 15)
  Heap Blocks: exact=12
  ->  Bitmap Index Scan on idx_order_item_product_id  (cost=0.00..4.31 rows=25 width=0) (actual time=0.030..0.030 rows=28 loops=1)
        Index Cond: (product_id = 15)
Planning Time: 0.091 ms
Execution Time: 0.132 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Why did PostgreSQL choose a Bitmap scan instead of a plain Index Scan?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Output D — Hash Join

```
Hash Join  (cost=33.50..75.82 rows=120 width=52) (actual time=0.452..1.205 rows=115 loops=1)
  Hash Cond: (oi.product_id = p.id)
  ->  Seq Scan on order_items oi  (cost=0.00..30.40 rows=2040 width=16) (actual time=0.008..0.195 rows=2040 loops=1)
  ->  Hash  (cost=20.00..20.00 rows=500 width=40) (actual time=0.410..0.411 rows=500 loops=1)
        Buckets: 1024  Batches: 1  Memory Usage: 45kB
        ->  Seq Scan on products p  (cost=0.00..20.00 rows=500 width=40) (actual time=0.005..0.180 rows=500 loops=1)
Planning Time: 0.215 ms
Execution Time: 1.340 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Would adding an index improve this join? Which column?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

---

## 4. Index Design Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, recommend the best index(es), explain why, and predict the scan type PostgreSQL will use after indexing.

**Query 1**

```sql
SELECT * FROM customers WHERE email = 'anna.korhonen@email.fi';
```

**Query 2**

```sql
SELECT * FROM orders WHERE customer_id = 5 ORDER BY order_date DESC;
```

**Query 3**

```sql
SELECT product_id, SUM(quantity) FROM order_items GROUP BY product_id;
```

**Query 4**

```sql
SELECT * FROM products WHERE price BETWEEN 50 AND 100;
```

**Query 5**

```sql
SELECT p.name, c.name AS category
FROM products p
JOIN product_categories pc ON pc.product_id = p.product_id
JOIN categories c ON c.category_id = pc.category_id
WHERE pc.category_id = 2;
```

**Query 6**

```sql
SELECT * FROM payments WHERE order_id = 88;
```

**Query 7**

```sql
SELECT * FROM orders WHERE status = 'shipped' AND order_date > '2026-06-01';
```

**Query 8**

```sql
SELECT customer_id, COUNT(*) FROM orders GROUP BY customer_id HAVING COUNT(*) > 5;
```

**Query 9**

```sql
SELECT oi.quantity, p.name
FROM order_items oi
JOIN products p ON p.id = oi.product_id
WHERE oi.order_id = 100;
```

**Query 10**

```sql
SELECT * FROM products WHERE name LIKE 'Trail%';
```

For each query, fill in:

| #  | Recommended Index | Why It Helps | Expected Scan Type |
| -- | ----------------- | ------------ | ------------------ |
| 1  |                   |              |                    |
| 2  |                   |              |                    |
| 3  |                   |              |                    |
| 4  |                   |              |                    |
| 5  |                   |              |                    |
| 6  |                   |              |                    |
| 7  |                   |              |                    |
| 8  |                   |              |                    |
| 9  |                   |              |                    |
| 10 |                   |              |                    |

---

## 5. Performance Comparison Exercise

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, follow these steps:

1. Run `EXPLAIN ANALYZE` **without** an index — record the full output
2. Create the recommended index
3. Run `EXPLAIN ANALYZE` **again** — record the full output
4. Calculate the improvement factor (before time ÷ after time)
5. Write one sentence explaining why the improvement occurred (in the **Explanation** field under each comparison)

### Comparison 1

```sql
-- Query
SELECT * FROM order_items WHERE order_id = 55;

-- Index to create
CREATE INDEX idx_oi_order_id ON order_items (order_id);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —     |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 2

```sql
-- Query
SELECT * FROM products p
JOIN product_categories pc ON pc.product_id = p.product_id
WHERE pc.category_id = 2 AND p.price < 80;

CREATE INDEX idx_product_categories_category_id ON product_categories (category_id);
CREATE INDEX idx_products_price ON products (price);

-- Index to create
CREATE INDEX idx_products_cat_price ON products (category_id, price);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —     |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 3

```sql
-- Query
SELECT * FROM customers WHERE email = 'test.user@example.com';

-- Index to create
CREATE UNIQUE INDEX idx_customers_email ON customers (email);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —     |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 4

```sql
-- Query
SELECT order_id, SUM(quantity * unit_price) AS total
FROM order_items
WHERE order_id IN (10, 20, 30, 40, 50)
GROUP BY order_id;

-- Index to create
CREATE INDEX idx_oi_order_id_covering ON order_items (order_id) INCLUDE (quantity, unit_price);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —     |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 5

```sql
-- Query
SELECT * FROM orders WHERE status = 'pending';

-- Index to create (partial index)
CREATE INDEX idx_orders_pending ON orders (status) WHERE status = 'pending';
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —     |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

---

## Submission Checklist

**Required**

- [ ] Baseline measurements for all 5 project queries
- [ ] All 5 indexes created and re-measured
- [ ] Comparison table filled with before/after data
- [ ] Recommendation paragraph written
- [ ] All 8 theory questions answered

**Recommended practice**

- [ ] All 4 EXPLAIN outputs analyzed
- [ ] Index recommendations for all 10 queries
- [ ] All 5 performance comparisons completed with explanations
