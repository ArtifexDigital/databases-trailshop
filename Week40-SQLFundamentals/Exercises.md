# Week 40 — Exercises: SQL Fundamentals

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task

This week you'll build the TrailShop database from scratch and practice manipulating data.

### Task 1.1: Create the Database

1. Open your PostgreSQL terminal (psql) or pgAdmin
2. Create a new database called `trailshop`
3. Connect to it

### Task 1.2: Create All Tables

Write and execute the CREATE TABLE statements for all six TrailShop tables in the correct order:

- categories
- customers
- products
- product_categories
- orders
- order_items

**Requirements:**

- Use appropriate data types for each column
- Include all constraints from the theory (NOT NULL, UNIQUE, CHECK, FOREIGN KEY, DEFAULT)
- Use SERIAL for primary keys
- Ensure foreign keys reference the correct parent tables

**Verify** by running `\dt` in psql to list all tables.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> trailshop=# \dt
                 List of tables
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | categories         | table | postgres
 public | customers          | table | postgres
 public | order_items        | table | postgres
 public | orders             | table | postgres
 public | product_categories | table | postgres
 public | products           | table | postgres
(6 rows)
>
>
> ```

### Task 1.3: Insert Sample Data

Insert the following data:

**Categories** (at least 5):

- Footwear, Backpacks, Tents, Clothing, Accessories

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> trailshop=# SELECT * FROM categories;
 category_id | category_name |          description
-------------+---------------+-------------------------------
           2 | Backpacks     | Daypacks and expedition packs
           1 | Footwear      | Hiking Boots for trails
           6 | Clothing      | Clothing for trails
           7 | Accessories   | Accessories for hiking
           3 | Tents         | Tents for outdoor camping
(5 rows)
>
>
> ```

**Customers** (at least 5):

- Use easy to write names with realistic email addresses

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> INSERT INTO customers (customer_id, first_name, last_name, email, phone, street, city, postal_code, country, registered_at) VALUES
(1, 'Anna',   'Müller',     'anna.mueller@example.com',     '+49 151 2345678',  'Hauptstraße 12',    'Heilbronn', '74072', 'Germany', '2025-03-14 09:23:00'),
(2, 'Jonas',  'Schmidt',    'jonas.schmidt@example.com',    '+49 160 9876543',  'Bahnhofstraße 5',   'Aalen',     '73430', 'Germany', '2025-06-02 14:45:00'),
(3, 'Laura',  'Virtanen',   'laura.virtanen@example.com',   '+358 40 1234567',  'Porrassalmenkatu 8','Mikkeli',   '50100', 'Finland', '2025-09-21 11:10:00'),
(4, 'Lukas',  'Weber',      'lukas.weber@example.com',      '+43 664 5551234',  'Mariahilfer Str. 45','Wien',     '1060',  'Austria', '2026-01-08 16:30:00'),
(5, 'Sophie', 'Dubois',     'sophie.dubois@example.com',    '+33 6 12345678',   'Rue de Rivoli 22',  'Paris',     '75004', 'France',  '2026-04-17 08:05:00');

>
>
> ```

**Products** (at least 10):

- At least 2 products per category
- At least one product assigned to **two or more** categories
- Prices ranging from €20 to €500
- Various stock levels

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> INSERT INTO products (name, description, price, weight_kg, stock_quantity) VALUES
('Trail Runner Pro Running Shoe', 'Lightweight trail running shoe with aggressive grip outsole', 149.90, 0.62, 42),
('Alpine GTX Hiking Boot', 'Waterproof leather hiking boot with Gore-Tex membrane', 229.00, 1.45, 8),
('Daypack 22L Backpack', 'Compact daypack with ventilated back panel and rain cover', 79.90, 0.75, 65),
('Expedition 65+10 Trekking Backpack', 'Large trekking backpack with adjustable back length for multi-day tours', 289.00, 2.30, 3),
('Merino Base Layer Shirt', 'Odour-resistant merino wool shirt for all seasons', 59.90, 0.18, 120),
('Stormshell 3-Layer Rain Jacket', 'Fully taped 3-layer hardshell jacket with helmet-compatible hood', 349.00, 0.48, 0),
('Ultralight 2P Trekking Tent', 'Freestanding two-person tent with two vestibules', 499.00, 1.35, 5),
('Down Sleeping Bag -5 °C', '700 fill power down sleeping bag, comfort rating -5 °C', 279.00, 0.98, 14),
('Carbon Trekking Poles (Pair)', 'Foldable carbon poles with cork grips', 119.00, 0.42, 27),
('Hydration Bladder 2L', 'BPA-free hydration reservoir with insulated drinking tube', 24.90, 0.20, 210);
>
>
> ```

**Product categories:**

- Insert rows into `product_categories` so every sample product is linked to at least one category

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> INSERT INTO product_categories (product_id, category_id) VALUES
(1, 1),
(2, 1),
(3, 2),
(3, 7),
(4, 2),
(5, 6),
(6, 6),
(7, 3),
(8, 3),
(9, 7),
(10, 2),
(10, 7);
>
>
> ```

**Orders** (at least 5):

- Different customers, different statuses

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> INSERT INTO orders (customer_id, status, shipping_street, shipping_city, shipping_postal_code, shipping_country) VALUES
(3, 'Delivered',  'Porrassalmenkatu 8',  'Mikkeli',   '50100', 'Finland'),
(1, 'Shipped',    'Kaiserstraße 30',     'Stuttgart', '70173', 'Germany'),
(5, 'Processing', 'Rue de Rivoli 22',    'Paris',     '75004', 'France'),
(2, 'Cancelled',  'Bahnhofstraße 5',     'Aalen',     '73430', 'Germany'),
(4, 'Pending',    'Mariahilfer Str. 45', 'Wien',      '1060',  'Austria');
>
>
> ```

**Order Items** (at least 10):

- Multiple items in some orders, single items in others

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(2, 1,  1, 149.90),  
(2, 10, 1, 24.90),   
(2, 5,  1, 59.90),   
(3, 7,  1, 499.00),  
(3, 8,  1, 279.00), 
(3, 4,  1, 289.00),  
(4, 5,  2, 59.90),   
(4, 9,  1, 119.00),  
(5, 6,  1, 349.00),  
(6, 3,  1, 79.90);   
>
>
> ```

**Verify** each insert with `SELECT * FROM table_name;`

### Task 1.4: Practice UPDATE

> [!TIP]
> **Recommended practice.** Do Tasks 1.4–1.6. They are not required to finish the TrailShop project. They prepare you for the exams. Task 1.6 renames `stock` to `quantity_in_stock`. Later weeks still use `stock`, so after you practice the rename, change the column name back.

Perform the following updates and verify each one:

1. Increase the price of all products in the Footwear category by 10% (join through `product_categories`)
2. Change customer #3's email to a new address
3. Update the status of order #2 from 'shipped' to 'delivered'
4. Set the stock of 'HydroFlask 1L' to 85
5. Add a description to any product that currently has NULL in description

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> 1. UPDATE products
SET price = price * 1.10
FROM product_categories
WHERE products.product_id = product_categories.product_id
  AND product_categories.category_id = 1;

2. UPDATE customers
SET email = 'laura.virtanen1@example.com'
WHERE customer_id = 3;

3. UPDATE order
SET status = 'Delivered'
WHERE order_id = 2;

4. UPDATE products
SET stock_quantity = 85
WHERE name = 'HydroFlask 1L';

5. UPDATE products
SET description = 'Waterproof leather hiking boot with Gore-Tex membrane'
WHERE product_id = 2;
>
>
> ```

### Task 1.5: Practice DELETE

1. Delete the most recently created order (and observe what happens to its order_items if you used CASCADE)
2. Try to delete a product that appears in `order_items` — what error do you get?
3. Delete a category that has products linked through `product_categories`. The products should remain; only the link rows should disappear. Confirm this.
4. Delete a customer who has no orders

### Task 1.6: Practice ALTER TABLE

1. Add a column `phone VARCHAR(20)` to the customers table
2. Add a column `weight_grams INTEGER` to the products table
3. Add a CHECK constraint to ensure `weight_grams > 0` (allow NULL though — not all products have weight recorded yet)
4. Rename the `stock` column in products to `quantity_in_stock`

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> 1. ALTER TABLE customers
ADD COLUMN phone VARCHAR(20);

2. ALTER TABLE products
ADD COLUMN weight_grams INTEGER;

3. ALTER TABLE products
ADD CONSTRAINT check_weight_positive CHECK (weight_grams > 0);

4. ALTER TABLE products
RENAME COLUMN stock TO quantity_in_stock;
>
>
> ```

---

## Exercise 2: Theory Review Questions

Answer the following questions in your own words using the answer fields below:

1. What does SQL stand for, and why was the language designed to look like English?

> [!NOTE]
> **_Your Answer_**
>
> It stands for Structured Query Language it was designed to look like englisch to keep it simple so also non programmers can use it.

2. Explain the difference between DDL and DML. Give two example commands for each.

> [!NOTE]
> **_Your Answer_**
>
> DDL stands for Data Definition Language. It is used to define the structure of the database. For example: CREATE TABLE, ALTER TABLE, DROP TABLE etc. DML stands for Data Manipulation Language. It is used to work with the data inside the tables. For example: INSERT INTO, UPDATE, DELETE, SELECT etc.

3. What is the difference between DCL and TCL? When would you use each?

> [!NOTE]
> **_Your Answer_**
>
> TCL stands for Transaction Control Language. It manages transactions: Groups of operations that must succeed or fail as a unit. With TCL you have something like a version control. When something went wrong between BEGIN and COMMIT you can still use ROLLBACK to undo everything. DCL is all  about permissions: Who can do what. You can use it to manage the access rights of users.

4. Why must you create tables in a specific order? What determines that order?

> [!NOTE]
> **_Your Answer_**
>
> You must create tables in a specific order because of foreign keys. You have to create the table that is being referenced first and then the table that is referencing it. So you always have to start with the tables that don't have any foreign keys.

5. What is the difference between a column-level constraint and a table-level constraint? When _must_ you use a table-level constraint?

> [!NOTE]
> **_Your Answer_**
>
> A column level constraint is defined on a single column. A table level constraint is defined on multiple columns. You must use a table level constraint when a constraint applies to multiple columns. For example: CHECK constraints referencing multiple columns.

6. Explain the difference between `DELETE FROM products;` and `TRUNCATE TABLE products;`. When would you prefer each?

> [!NOTE]
> **_Your Answer_**
>
> DELETE is more precise, you have to add the WHERE add on. TRUNCATE just deletes every row. TRUNCATE is faster than DELETE. So you should use TRUNCATE when you want to delete all rows and DELETE when you want to delete specific rows.

7. What does `ON DELETE CASCADE` do on a foreign key? Give a real-world scenario where it's appropriate and one where it would be dangerous.

> [!NOTE]
> **_Your Answer_**
>
> ON DELETE CASCADE means that if a row is deleted from the parent table, the corresponding rows in the child table are also deleted. For example, if you delete a customer, all of their orders will be deleted, thats dangerous because online shops are forced to store order history for a certain amount of time. An example where its less dangerous is when you have a table of product reviews and you delete a product, then all of its reviews will be deleted.

8. Why should you store `unit_price` in the `order_items` table instead of just looking it up from the `products` table?

> [!NOTE]
> **_Your Answer_**
>
> You should store unit_price in the order_items table instead of just looking it up from the products table because the price of a product can change over time. If you store the unit_price in the order_items table, you can ensure that the price of a product is always the same for a given order.

9. What is the difference between SERIAL and GENERATED ALWAYS AS IDENTITY? Which would you use in a new project and why?

> [!NOTE]
> **_Your Answer_**
>
> SERIAL is shorthand for creating columns with INTEGER NOT NULL, a sequence, and a trigger to auto-increment. GENERATED ALWAYS AS IDENTITY is the modern SQL standard way to create auto-incrementing columns. It's safer because it prevents accidental manual inserts and is clearer about intent. For a new project, I would use GENERATED ALWAYS AS IDENTITY because it's the standard and safer. GENERATED ALWAYS... can also be used in various SQL Languages like PostgreSQL, MySQL, Oracle etc.  

10. Explain why `UPDATE products SET price = 9.99;` is dangerous. What steps should you take before running any UPDATE statement?

> [!NOTE]
> **_Your Answer_**
>
> Its dangerous because when you running a UPDATE statement without a WHERE clause it will update every row in the table. So you should always use a WHERE clause when running a UPDATE statement or using the TCL language so you can always roll back. 

---

## Exercise 3: SQL Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write the SQL statements for each task in the **Your SQL** fields below. Verify by running them when ready.

### 3.1 CREATE TABLE

Write a CREATE TABLE statement for a `suppliers` table with the following columns:

- supplier_id (auto-incrementing primary key)
- company_name (required, max 200 characters, must be unique)
- contact_name (max 150 characters)
- email (max 255 characters, required, unique)
- phone (max 20 characters)
- country (max 100 characters, required, default 'Finland')

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.2 CREATE TABLE with Foreign Key

Write a CREATE TABLE statement for a `product_reviews` table:

- review_id (auto-incrementing primary key)
- product_id (required, references products)
- customer_id (required, references customers)
- rating (required integer, must be between 1 and 5 inclusive)
- review_text (optional, unlimited length)
- created_at (required, defaults to current timestamp)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.3 INSERT — Single Row

Write an INSERT statement to add a new category called 'Electronics' with description 'GPS devices, solar chargers, and tech gear'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.4 INSERT — Multiple Rows

Write a single INSERT statement that adds three new customers:

- Eero Lahtinen, eero.l@email.com
- Maria Salminen, maria.s@email.com
- Petri Kallio, petri.k@email.com

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.5 INSERT with RETURNING

Write an INSERT statement that adds a new product called 'NorthStar GPS' priced at €229.99 with stock of 12, then assign it to category 'Electronics' (assume `category_id = 6`) using `product_categories`. Return the product_id and created_at.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.6 UPDATE — Simple

Write an UPDATE statement that changes the email of the customer with customer_id = 2 to 'mikko.korhonen@newmail.com'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.7 UPDATE — Expression

Write an UPDATE statement that reduces the stock of all products by 1 where the stock is currently greater than 0.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.8 UPDATE — Multiple Columns

Write an UPDATE statement that changes order #3 to status 'cancelled' and sets a (hypothetical) cancelled_at timestamp to the current time. (Assume you've already added a cancelled_at column.)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.9 DELETE — With Condition

Write a DELETE statement that removes all orders with status 'cancelled'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.10 ALTER TABLE

Write the ALTER TABLE statements to:
a) Add a `discount_percent NUMERIC(5,2) DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100)` column to products
b) Drop the `description` column from categories
c) Add a composite unique constraint on (customer_id, product_id) in the product_reviews table (preventing a customer from reviewing the same product twice)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Exercise 4: Error Diagnosis (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Each of the following SQL statements contains one or more errors. Identify the error(s) and write the corrected version.

### 4.1

```sql
CREATE TABLE warehouses
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(100
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
INSERT INTO products (name, price, stock)
VALUES ("Alpine Sleeping Bag", 89.99, 20);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
CREATE TABLE shipments (
    shipment_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id)
    shipped_date DATE NOT NULL,
    carrier VARCHAR(100)
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
UPDATE products
SET price = price * 0.9
SET stock = stock + 10
WHERE product_id = 3;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
CREATE TABLE wishlists (
    wishlist_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    added_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id, product_id)
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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

- [ ] All 6 TrailShop tables created successfully
- [ ] Sample data inserted (at least 5 categories, 5 customers, 10 products, product_categories links, 5 orders, 10 order items)
- [ ] Theory review questions answered

**Recommended practice**

- [ ] UPDATE exercises completed and verified
- [ ] DELETE exercises completed and verified
- [ ] ALTER TABLE exercises completed, then `quantity_in_stock` renamed back to `stock`
- [ ] SQL writing exercises completed
- [ ] Error diagnosis completed with corrections
