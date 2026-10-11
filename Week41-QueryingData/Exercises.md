# Week 41 — Exercises: Querying Data

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task — Business Questions

Use the TrailShop database you created in Week 40. Write SQL queries to answer each business question below. Run each query and verify the results make sense.

> [!IMPORTANT]
> **_Tools to use_**
> You can use both the pgAdmin or psql (terminal) to test the queries

### Basic Queries (SELECT + WHERE)

1. List all products in the 'Footwear' category (show name, price, stock).

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT name, price, stock_quantity
   > FROM products
   > WHERE product_id IN (
   >     SELECT pc.product_id
   >     FROM product_categories pc
   >     JOIN categories c ON pc.category_id = c.category_id
   >     WHERE c.category_name = 'Footwear'
   > );
   > ```

2. Find all products priced between €50 and €150, sorted by price ascending.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT price, name
   > FROM products
   > WHERE price BETWEEN 50 AND 150
   > ORDER BY price ASC;
   > ```

3. Show all customers whose last name starts with the letter 'M' or 'K'.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT last_name, first_name
   > FROM customers
   > WHERE last_name LIKE 'M%' OR last_name LIKE 'K%';
   > ```

4. List all orders with status 'pending' or 'shipped', sorted by order date (most recent first).

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT order_id, status, order_date, customer_id
   > FROM orders
   > WHERE LOWER(status) IN ('pending', 'shipped')
   > ORDER BY order_date DESC;
   > ```

5. Find all products that have the word 'Pro' or 'pro' somewhere in their name.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT name
   > FROM products
   > WHERE name ILIKE '%pro%';
   > ```

### Aggregate Queries

6. What is the total number of products in the database?

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT COUNT (*) FROM products;
>
>
> ```

7. What is the average price of all products? Round to 2 decimal places.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT ROUND(AVG(price), 2) AS average_price FROM products; 
   >
   >
   > ```

8. Which is the most expensive product and which is the cheapest? Show both in one query.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT MIN(price) AS cheapest, 
   >MAX(price) AS most_expensive
   > FROM products;
   >
   >
   > ```

9. How many orders does each customer have? Show customer name and order count, sorted by count descending.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT c.first_name,
   >        c.last_name,
   >         COUNT(o.order_id) AS order_count
   > FROM customers c 
   >JOIN orders o ON c.customer_id = o.customer_id
   >GROUP BY c.customer_id, c.first_name, c.last_name
   >ORDER BY order_count DESC;
   > ```

10. What is the total revenue (sum of quantity × unit_price from order_items) for each order status?

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT o.status, 
>        ROUND(SUM(oi.quantity * oi.unit_price),2) AS total_revenue
> FROM orders o
>JOIN order_items oi ON o.order_id = oi.order_id
>GROUP BY o.status
>ORDER BY total_revenue DESC; 
> ```

### JOIN Queries

11. List all products with their category names (join through `product_categories`). A product in two categories should appear twice. Sort by category name, then product name.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT c.name AS category_name, p.name AS product_name
    > FROM products p
    > INNER JOIN product_categories pc ON pc.product_id = p.product_id
    > INNER JOIN categories c ON c.category_id = pc.category_id
    > ORDER BY category_name, product_name; 
    >
    > ```

12. Show each order with the customer's full name, order date, and status.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT o.order_id, o.order_date, c.first_name || ' ' || c.last_name AS customer_full_name, o.status
    > FROM orders o
    > INNER JOIN customers c ON c.customer_id = o.customer_id;
    >
    >
    > ```

13. Show a detailed breakdown of order #1: product name, quantity, unit price, and line total.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT p.name, oi.quantity, oi.unit_price, (oi.quantity*oi.unit_price) AS line_total
    > FROM order_items oi
    > JOIN products p ON oi.product_id = p.product_id
    > WHERE oi.order_id = 1;
    > ```

14. Find all customers who have NOT placed any orders. (Hint: use LEFT JOIN + IS NULL pattern.)

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT c.customer_id, c.first_name, c.last_name 
    > FROM customers c
    > LEFT JOIN orders o ON c.customer_id = o.customer_id
    > WHERE o.customer_id is NULL; 
    > ```

15. For each category, show the category name, number of products, average price, and total inventory value (price × stock summed). Only include categories with total inventory value greater than €500.
    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT c.category_name, COUNT(pc.product_id) as product_count, ROUND(AVG(p.price),2) AS average_price, SUM(p.price * p.stock_quantity) AS total_inventory_value
    > FROM categories c 
    > JOIN product_categories pc ON c.category_id = pc.category_id
    > JOIN products p ON p.product_id = pc.product_id
    > GROUP BY c.category_name
    > HAVING SUM(p.price * p.stock_quantity) > 500;
    > ```

---

## Exercise 2: Theory Review Questions

Answer in your own words:

1. What is the logical execution order of a SQL query? Why does it matter?

> [!NOTE]
> **_Your Answer_**
>
> The logical execution order of SQL is the order in which the database processes the clauses of a SQL query. It matters because the SQL refers to logical thinking. For example you cant run a WHERE clause before the FROM clause, because first you have to define the table you want to query from, otherwise that wouldnt make sense from a logical perspective.

2. What is the difference between WHERE and HAVING? Give an example of when you would use each.

> [!NOTE]
> **_Your Answer_**
>
> WHERE is the clause that filters rows based on conditions. HAVING is the clause that filters groups after aggregation. For example, you would use WHERE to filter orders based on their status, and you would use HAVING to filter categories based on the average price of their products.

3. Explain the difference between COUNT(\*), COUNT(column), and COUNT(DISTINCT column).

> [!NOTE]
> **_Your Answer_**
>
> COUNT(\*) counts all rows including those with NULLs. COUNT(column) counts all rows where column is not NULL. COUNT(DISTINCT column) counts all unique non-NULL values in a column.

4. What is the difference between INNER JOIN and LEFT JOIN? When would you choose one over the other?

> [!NOTE]
> **_Your Answer_**
>
> INNER JOIN only returns rows that have matching values in both tables. LEFT JOIN returns all rows from the left table and matching rows from the right table. If there is no match, the result is NULL on the right side, but would still be counted. You use INNER JOIN when you only want to see rows that have matching values in both tables (for example: Which products are in which orders?). You use LEFT JOIN when you want to see all rows from the left table and matching rows from the right table (For example: Show me all products even those that are not in any orders).

5. Why should you avoid SELECT \* in production code?

> [!NOTE]
> 1. It takes a lot time to load all the rows in database with 1000+ rows.
> 2. If the columns in the database are changed (added/removed/renamed) the query might break or return unexpected results.
> 3. It makes it harder to understand the query.
> 4. Readability is much lower, espacially for people who dont know the database structure.
> 

6. What does DISTINCT do? On what level does it operate (columns or entire rows)?

> [!NOTE]
> **_Your Answer_**
>
> DISTINCT eliminates duplicate rows from the result set. It operates on the entire row. This means that all columns in the result must match for two rows to be considered duplicates.

7. Can you use a column alias in a WHERE clause? Why or why not?

> [!NOTE]
> **_Your Answer_**
>
> No you can not, because the WHERE clause doesnt know the alias because its not a defined row/column that can be referenced. Its just a label/name for the result.
>
> 8. Explain what happens when you GROUP BY a column and there's a column in SELECT that isn't aggregated and isn't in GROUP BY.

> [!NOTE]
> **_Your Answer_**
>
> You would get an error message, because GROUP BY combines many rows into one group, Which means, every column in SELECT needs exactly one value per group. That only can happen when either it is in GROUP BY or its in an aggregate function.

---

## Exercise 3: Query Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write the SQL for each task. Use the TrailShop schema (categories, customers, products, product_categories, orders, order_items).

### Simple SELECT + WHERE

**3.1** — Select product names and prices for all products with stock less than 15.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.2** — Find all customers who registered (created_at) in 2026. Show first name, last name, and registration date.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.3** — Show all products that do NOT have a description (description IS NULL).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Multi-condition Filtering

**3.4** — Find products in category 1 OR category 2, priced above €100, with stock greater than 0. Sort by price descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.5** — Find orders that are either 'delivered' or placed by customer_id 1. Show order_id, customer_id, status.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Aggregation

**3.6** — For each category, show the category name, minimum, maximum, and average product price. Round averages to 2 decimal places. Join through `product_categories`.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.7** — Count how many distinct customers have placed at least one order.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.8** — Find the total quantity of items sold across all orders.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### JOIN Queries

**3.9** — Show each product name alongside its category name. Include all products (even if somehow a category was deleted — use LEFT JOIN).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.10** — List all orders showing: order_id, customer full name, order date, number of items in the order, and order total cost.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.11** — Show all products that have NEVER been ordered. (Hint: LEFT JOIN order_items, then IS NULL on order_item_id.)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### GROUP BY + HAVING

**3.12** — Show categories where the average product price exceeds €100. Display category name and average price.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.13** — Find customers who have placed more than 1 order. Show their name and order count.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Pagination

**3.14** — Write a paginated query that returns products 4 through 6 (page 2, page size 3), ordered by product_id.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Complex

**3.15** — Write a "sales report" query that shows: category name, total units sold (from order_items), total revenue, and number of distinct products sold — for each category. Sort by revenue descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Exercise 4: Query Reading Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, explain in **plain English** what it does and what the result would look like.

### 4.1

```sql
SELECT c.name, COUNT(p.product_id) AS num_products
FROM categories c
LEFT JOIN product_categories pc ON pc.category_id = c.category_id
LEFT JOIN products p ON p.product_id = pc.product_id
GROUP BY c.name
ORDER BY num_products DESC;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.2

```sql
SELECT first_name, last_name
FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM orders
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.3

```sql
SELECT p.name, p.price, p.stock,
       p.price * p.stock AS inventory_value
FROM products p
WHERE p.stock > 0
ORDER BY inventory_value DESC
LIMIT 3;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.4

```sql
SELECT o.order_id,
       SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders o
INNER JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.status <> 'cancelled'
GROUP BY o.order_id
HAVING SUM(oi.quantity * oi.unit_price) > 200
ORDER BY order_total DESC;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.5

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       COUNT(DISTINCT o.order_id) AS num_orders,
       SUM(oi.quantity) AS total_items,
       ROUND(SUM(oi.quantity * oi.unit_price), 2) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC NULLS LAST;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

---

## Submission Checklist

**Required**

- [ ] All 15 business questions answered with working SQL
- [ ] Theory review questions answered in your own words
- [ ] Results of the business questions verified by running them against your TrailShop database

**Recommended practice**

- [ ] All 15 query writing exercises completed
- [ ] All 5 query reading explanations written
