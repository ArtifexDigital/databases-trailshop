# Week 39 — Logical Database Design: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 39 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Build the Schema

**Goal:** Convert the TrailShop ER diagram (from Week 38) into a complete PostgreSQL relational schema.

### Instructions

Write `CREATE TABLE` statements for all six TrailShop tables:

1. `categories`
2. `customers`
3. `products`
4. `product_categories`
5. `orders`
6. `order_items`

### Requirements

For each table, you must:

- Choose appropriate PostgreSQL data types for every column (justify at least 3 choices in writing)
- Define primary keys (surrogate or composite as appropriate)
- Define foreign keys with explicit `ON DELETE` and `ON UPDATE` actions (justify each choice)
- Add `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` constraints where appropriate
- Create tables in the correct dependency order
- Follow the naming conventions from Theory Section 8

### Deliverables

1. A single `.sql` file with all six `CREATE TABLE` statements (executable in PostgreSQL)
2. A short written document (1–2 pages) containing:
   - Justification for 3 data type choices (e.g., why `NUMERIC(10,2)` for price instead of `REAL`)
   - Justification for each FK action choice (e.g., why CASCADE on `order_items.order_id`)
   - One design decision you made that wasn't specified in the requirements (e.g., whether shipping address is optional)

### Bonus Challenge

After creating the tables, insert sample data:
- At least 5 categories
- At least 8 products (across at least 3 categories)
- At least one product assigned to **two or more** categories via `product_categories`
- At least 3 customers
- At least 4 orders (across at least 2 customers)
- At least 10 order items

Verify that your constraints work by attempting at least 2 invalid inserts and showing the error messages.

> [!NOTE]
> ***Your SQL***
>
> (Paste key CREATE TABLE statements or link to your .sql file contents here)
> 
> See [`trailshop_schema.sql`](./trailshop_schema.sql)
> 

> [!NOTE]
> ***Your Answer***
>
> (Paste written justifications for data types, FK actions, and design decisions here.)
>
>## 1. Data Type Choices

### 1.1 NUMERIC(10,2) for price and unit_price

Both price columns use NUMERIC(10,2) instead of a floating point type such as REAL. The reason is that REAL and DOUBLE PRECISION store values in binary floating point, which cannot represent most decimal fractions exactly. A value like 19.99 is stored as the closest binary approximation, not as the number itself. For a single product that difference is invisible, but as soon as you sum up order items or calculate VAT, the small errors add up and the total can be off by a cent. In a shop that is not acceptable, because the amount shown to the customer and the amount stored in the database have to match exactly.Money is never a floating point number.

### 1.2 VARCHAR(n) instead of TEXT

For most character columns, such as first_name, email, postal_code and phone, I used VARCHAR with an explicit length instead of TEXT. In PostgreSQL there is no performance advantage in doing so, because both types are stored the same way internally. The benefit is a different one. The length limit works as a cheap additional constraint and documents what the column is meant to hold. A postal code of 200 characters is not a postal code, it is a data entry error, and VARCHAR(10) rejects it at insert time instead of letting it into the table.

The lengths are therefore chosen per column rather than copied everywhere. phone gets 20 characters so that international formats with a country prefix still fit, street gets 100, and postal_code only 10.

### 1.3 SERIAL as a surrogate key

Every main table uses a SERIAL surrogate key as its primary key instead of a natural key. There would have been candidates for natural keys, for example email in customers or category_name in categories, and both are unique in practice. The problem is that both can change. Customers switch email providers and marketing renames a category from "Tents" to "Tents & Shelters".

## 2. Foreign Key Actions

order_items.order_id references orders, ON DELETE CASCADE. order_items is a weak entity, and its relationship to orders is identifying, which is why order_id is part of its composite primary key. An order item has no meaning on its own. If order 42 is removed, its line items describe nothing anymore and would only stay behind as orphaned rows. CASCADE is therefore not a convenience here, it follows directly from the fact that the child cannot exist without the parent.

order_items.product_id references products, ON DELETE RESTRICT. This is the interesting contrast, because the same table uses two different actions. The relationship to products is a plain reference, not a composition. A product exists independently of any order, and the order item only points at it. Deleting a product that has already been sold would destroy sales history, so the database blocks it. If the shop wants to remove a product from the catalogue, the correct move is to mark it as inactive or set stock_quantity to zero, not to delete the row. This also explains why unit_price is stored in order_items rather than looked up from products: the price paid at the time of the order has to stay correct even when the current product price changes later.

orders.customer_id references customers, ON DELETE RESTRICT. Orders are financial records, and in most countries they have to be kept for several years. If deleting a customer silently deleted their orders, the shop would lose revenue data and, because of the cascade on order_items, the line items as well.

product_categories.product_id and product_categories.category_id, both ON DELETE CASCADE. A junction table row carries no information of its own beyond the assignment itself. If a category is discontinued, the statement "this product belongs to that category" simply stops being true, and the row should disappear with it. Importantly, the cascade only removes the assignment, not the product, because the delete travels from the parent to the junction table and stops there. 

ON UPDATE CASCADE on all five. Because all primary keys are SERIAL values that are never reused or edited, this action will realistically never fire. I still set it explicitly rather than relying on the default. If a key ever does change, the children follow automatically instead of the update being blocked, and writing it out makes the intention visible to anyone reading the schema. The cost of declaring it is zero.

## 3. Design Decision: DEFAULT Finland for customers.country

The requirements say nothing about how the country column should behave, so this was my own decision. I declared it NOT NULL DEFAULT 'Finland'.

The reasoning is that TrailShop is a Finnish shop and the large majority of its customers will be Finnish. Combining NOT NULL with a default means the column can never be empty, but the application does not have to supply a value for the common case. Without the default I would have had two weaker options. Leaving the column nullable would allow delivery addresses without a country, which is not a usable address. Keeping it NOT NULL without a default would push the responsibility onto every insert statement and make the most frequent case the most verbose one.

If TrailShop later starts shipping across Europe on a larger scale, I would revisit this.

>
>

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section.

1. List the seven phases of the database development lifecycle in order. Which phase is this week's focus? *(Section 1)*

> [!NOTE]
> ***Your Answer***
>
> 1. Requirements Gathering
2. Conceprual Design
3. Logical Design --> Focus this week
4. Physical Design
5. Implementation
6. Testing & Validation
7. Maintenance & Evolution
>
>
>
>

2. Explain the transformation rule for mapping a 1:N relationship to the relational model. Why is the foreign key placed on the "many" side? *(Section 3.2)*

> [!NOTE]
> ***Your Answer***
>
> Lets take the example of customers and orders. Because each order belongs to ONE customer only you can store that single reference in the order row. On the other side, one customer can have more than one order. So you cant store the order to the customer table because than you would need to write every order in one row which would clash with the atomity principle. 
>
>
>
>

3. What is a junction table? When is it needed? Give an example not from TrailShop. *(Section 3.3)*

> [!NOTE]
> ***Your Answer***
>
> A junction table manages M:N relationships by containing the primary key of both participating entities as foreign keys. Its needed when many-to-many relationships appear in a model. For example when a student can have many courses and a course can have many students, you need a junction table to manage this relationship.
>
>
>
>

4. When mapping a 1:1 relationship, how do you decide which table gets the foreign key? *(Section 3.4)*

> [!NOTE]
> ***Your Answer***
>
> You have 4 decision criterias: 
>- **If one side has mandatory participation and the other optional:** Put the FK on the mandatory side (it will always have a value).
- **If both sides are mandatory:** Either side works; choose the side that makes queries more natural.
- **If both sides are optional:** Put the FK on the side that is more likely to have the value. Mark the FK column as NULL-able.
- **Alternative:** Merge both entities into one table if they always exist together.
>
>
>

5. How does the mapping of a weak entity differ from a strong entity? What happens to the primary key? *(Section 3.5)*

> [!NOTE]
> ***Your Answer***
>
> You have to include the owner entitys primary key as both a foreign key and as part of the composite key in the weak entity. The primary key of a strong entity is not changed.
>
>
>
>

6. Why should you never use `REAL` or `DOUBLE PRECISION` for monetary values? What should you use instead? *(Section 4.1)*

> [!NOTE]
> ***Your Answer***
>
> Because REAL and DOUBLE PRECISION are simply not precise enough and have rounding errors, which can cause trouble with multiple orders and customers. You should use NUMERIC instead for money values because of its exact precision.
>
>
>
>

7. What is the difference between `TIMESTAMP` and `TIMESTAMPTZ`? Which should you prefer and why? *(Section 4.3)*
> [!NOTE]
> ***Your Answer***
>
> TIMESTAMPTZ stores the time with timezone information, while TIMESTAMP stores it without. You should prefer TIMESTAMPTZ because it stores the time with the timezone information, so you can compare times from different timezones.
>




8. Explain the difference between `CASCADE` and `RESTRICT` as foreign key delete actions. Give a scenario where each is appropriate. *(Section 6)*
> [!NOTE]
> ***Your Answer***
>
> CASCADE is the setting, that either deletes or updates automatically the child rows if the parent row is deleted or updated. RESTRICT is the setting, that prevents the deletion or update of a parent row if there are child rows. A scenario for CASCADE is when you delete a product category and you want to delete all products in that category. A scenario for RESTRICT is when you delete a customer and you want to keep all orders of that customer, because it might be that you need them later.
>




9. What is an insertion anomaly? Give an example and explain how proper schema design prevents it. *(Section 7)*
> [!NOTE]
> ***Your Answer***
>
> Its the issue caused through poorly designed tbales/databases. You cannot insert certain data without inserting other unrelated data.
> For example if you want to add a new product category you also have to add a product to it.




10. What is the difference between a surrogate key and a natural key? Give one advantage of each. *(Section 9)*
> [!NOTE]
> ***Your Answer***
>
> A natural Key is a key with real world meaning behind it. An advantage of it is, that it is easy to understand and to use. 
A surrogate Key is a key with no real world meaning behind it. An advantage of it is, that it is always unique and easy to use.
>




11. Why does PostgreSQL fold unquoted identifiers to lowercase? How does `snake_case` naming help? *(Section 8)*

> [!NOTE]
> ***Your Answer***
>
> Because it prevents the problem of mixing lower and upper case letters in your database. You always know that the column name is lowercase. snake_case is doing this exactly and gives some more rules you have to stick to.
>
>
>
>

12. What does `SET NULL` do as a foreign key action? When would you use it instead of `CASCADE`? *(Section 6)*
> [!NOTE]
> ***Your Answer***
>
> Instead of deleting a value, it sets it to NULL so the row still exists. You use it instead of CASCADE if you want to keep the row but don't want to keep the value. For example if a customer cancels his membership, you don't want to delete his orders, but you want to set his customer_id to NULL.
>




---

## Exercise 3: Transformation Exercise — Hotel Booking System

### Given ER Diagram

A hotel booking system has the following entities and relationships:

**Entities:**

1. **Hotel** — hotel_id (PK), name, city, star_rating, phone
2. **Room** (weak entity, owned by Hotel) — room_number (partial key), room_type, floor, price_per_night, has_balcony
3. **Guest** — guest_id (PK), first_name, last_name, email, phone, passport_number
4. **Booking** — booking_id (PK), check_in_date, check_out_date, total_amount, status
5. **Service** — service_id (PK), name, description, price (e.g., "Room Service", "Spa", "Airport Shuttle")

**Relationships:**

- Hotel (1) → Room (N): A hotel has many rooms. Each room belongs to exactly one hotel. (Identifying relationship — Room is weak.)
- Guest (1) → Booking (N): A guest can make many bookings. Each booking belongs to one guest.
- Booking (M) ↔ Room (N): A booking can include multiple rooms, and a room can appear in many bookings (over time). The junction records the specific dates.
- Booking (M) ↔ Service (N): A booking can use multiple services, and a service can be used by many bookings. The junction records the date used and quantity.

### Task

1. Write `CREATE TABLE` statements for ALL tables (including junction tables).
2. For each table:
   - Choose appropriate data types
   - Define PK, FK, NOT NULL, UNIQUE, CHECK, and DEFAULT constraints
   - Specify ON DELETE and ON UPDATE actions for all FKs
3. Create the tables in the correct dependency order.
4. Explain why Room is a weak entity and how its PK reflects this.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE TABLE hotels(
   hotel_id SERIAL PRIMARY KEY,
   name VARCHAR(100) NOT NULL,
   city VARCHAR(100) NOT NULL,
   star_rating INTEGER NOT NULL CHECK (star_rating BETWEEN 1 AND 5),
   phone VARCHAR(20) NOT NULL
); 

CREATE TABLE rooms(
   room_number VARCHAR(10) NOT NULL,
   room_type VARCHAR(50) NOT NULL,
   floor INTEGER NOT NULL,
   price_per_night NUMERIC(10,2) NOT NULL CHECK(price_per_night > 0),
   has_balcony BOOLEAN NOT NULL DEFAULT FALSE,
   hotel_id INTEGER NOT NULL,
   PRIMARY KEY (room_number, hotel_id),
   FOREIGN KEY (hotel_id, room_number) REFERENCES rooms(hotel_id, rooms_number) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE guests(
   guest_id SERIAL PRIMARY KEY,
   first_name VARCHAR(50) NOT NULL,
   last_name VARCHAR(50) NOT NULL,
   email VARCHAR(50) NOT NULL UNIQUE,
   phone VARCHAR(20) NOT NULL,
   passport_number VARCHAR(20) NOT NULL UNIQUE
);

CREATE TABLE bookings(
   booking_id SERIAL PRIMARY KEY,
   check_in_date DATE NOT NULL,
   check_out_date DATE NOT NULL CHECK (check_out_date > check_in_date),
   total_amount NUMERIC(10,2) NOT NULL CHECK(total_amount > 0),
   status VARCHAR(20) NOT NULL DEFAULT 'PENDING'
            CHECK(status IN ('PENDING','PROCESSING','BOOKED','CANCELLED','CHECKED_IN', 'CHECKED_OUT')),
   guest_id INTEGER NOT NULL,
   FOREIGN KEY (guest_id) REFERENCES guests(guest_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE services(
   service_id SERIAL PRIMARY KEY,
   name VARCHAR(50) NOT NULL,
   description TEXT,
   price NUMERIC(10,2) NOT NULL CHECK (price >= 0)
);

CREATE TABLE booking_rooms(
   booking_id INTEGER NOT NULL
               REFERENCES bookings(booking_id)
               ON DELETE CASCADE
               ON UPDATE CASCADE,
   room_number INTEGER NOT NULL
               REFERENCES rooms(Room_number)
               ON DELETE CASCADE
               ON UPDATE CASCADE,
   PRIMARY KEY(booking_id, room_number)
);

CREATE TABLE booking_services(
   booking_id INTEGER NOT NULL
            REFERENCES bookings(booking_id)
            ON DELETE CASCADE
            ON UPDATE CASCADE,
   service_id INTEGER NOT NULL
            REFERENCES services(service_id)
            ON DELETE CASCADE
            ON UPDATE CASCADE,
   PRIMARY KEY(booking_id, service_id)
);
>
>
> ```

> [!NOTE]
> ***Your Answer***
>
> *(Explain why Room is a weak entity and how its PK reflects this.)*
>
>
>
>

---

## Exercise 4: Data Type Selection

> [!NOTE]
> ***Your Answers***
> Fill in the **Your Data Type** and **Justification** columns in the table below.
>

For each column described below, choose the best PostgreSQL data type and write a brief justification (1–2 sentences). Do NOT just pick `VARCHAR` or `TEXT` for everything — think carefully about validation, storage, and query needs.

| # | Column Description | Your Data Type | Justification |
|---|---|---|---|
| 1 | Employee salary (exact, up to €999,999.99) | NUMERIC(10,2) | Its the most precise one for money, because it avoids rounding errors |
| 2 | Number of items in stock (never negative, max ~50,000) | INTEGER CHECK(>0) | Stores whole numbers and makes sure that the number is never negative |
| 3 | Whether a user's email is verified | BOOLEAN | Stores true/false values and makes sure that the value is either true or false |
| 4 | Customer's date of birth | DATE | Stores dates and makes sure that the date is in the correct format |
| 5 | Product description (variable length, could be several paragraphs) | TEXT | Stores text and makes sure that the text is in the correct format |
| 6 | Country code (always exactly 2 letters, like "FI", "US") | CHAR(2) | Stores fixed length strings and makes sure that the string is in the correct format |
| 7 | IP address of a login attempt | VARCHAR(45) | Stores variable length strings and makes sure that the string is in the correct format |
| 8 | Order total (exact, up to €9,999,999.99) | NUMERIC(10,2) | Like employee salary but for larger values |
| 9 | GPS latitude of a store location | NUMERIC(10,8) | Stores decimal numbers with high precision |
| 10 | A unique identifier for API tokens that must be globally unique across distributed systems | UUID UNIQUE NOT NULL | Stores universally unique identifiers and makes sure that the identifier is in the correct format |
| 11 | Duration of a video in seconds (always a whole number) | INTEGER | Stores whole numbers and makes sure that the number is in the correct format |
| 12 | Timestamp of when a record was last modified (users in multiple time zones) | TIMESTAMPTZ | Stores timestamps with time zone support and makes sure that the timestamp is in the correct format |
| 13 | A Finnish phone number like "+358 40 123 4567" | VARCHAR(20) | Stores variable length strings and makes sure that the string is in the correct format |
| 14 | A percentage discount (0.00% to 100.00%) | DECIMAL(5,2) | Stores decimal numbers with high precision and makes sure that the number is in the correct format |
| 15 | A product's color options (e.g., a product comes in "red", "blue", "green") | VARCHAR(20) | Stores variable length strings and makes sure that the string is in the correct format |

---

## Exercise 5: Constraint Design

For each business rule below, write the appropriate PostgreSQL constraint. Provide the constraint as it would appear inside a `CREATE TABLE` statement or as an `ALTER TABLE` statement.

### Part A: Single-Column Constraints

1. "A product's weight must be greater than zero (if provided)."

2. "Every customer must have an email address."

3. "Product names must be unique."

4. "An employee's hire date defaults to today if not specified."

5. "Order status can only be one of: 'new', 'confirmed', 'shipped', 'delivered', 'returned'."

> [!NOTE]
> ***Your SQL***
>
> ```sql
1. ALTER TABLE products ADD COLUMN weight_kg NUMERIC(10,2) CHECK (weight_kg > 0);
2. ALTER TABLE customers ALTER COLUMN email SET NOT NULL;
3. ALTER TABLE products ADD CONSTRAINT uq_product_name UNIQUE (product_name);
4. ALTER TABLE employees ALTER COLUMN hire_date SET DEFAULT CURRENT_DATE;
5. ALTER TABLE orders ADD CONSTRAINT chk_order_status CHECK (status IN ('new', 'confirmed', 'shipped', 'delivered', 'returned'));
>
> ```

### Part B: Multi-Column Constraints

6. "A flight's arrival time must be after its departure time."

7. "In the `enrollments` table, the combination of `student_id` and `course_id` must be unique (a student can only enroll in a course once)."

8. "A discount percentage must be between 0 and 100, inclusive."

> [!NOTE]
> ***Your SQL***
>
> ```sql
6. ALTER TABLE flights ADD CONSTRAINT chk_arrival_after_departure CHECK (arrival_time > departure_time);
7. ALTER TABLE enrollments ADD CONSTRAINT uq_student_course UNIQUE (student_id, course_id);
8. ALTER TABLE discounts ADD CONSTRAINT chk_discount_percentage CHECK (discount_percentage BETWEEN 0 AND 100);
>
>
> ```

### Part C: Foreign Key Constraints with Actions

9. "When a department is deleted, all employees in that department should have their `department_id` set to NULL (they become unassigned)."

10. "When a customer is deleted, prevent the deletion if the customer has any orders."

11. "When an author is deleted, all their blog posts should be deleted automatically."

12. "When a course is deleted, all enrollments for that course should be removed."

> [!NOTE]
> ***Your SQL***
>
> ```sql
9. ALTER TABLE employees ADD CONSTRAINT fk_employees_department FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE SET NULL;
10. ALTER TABLE orders ADD CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE RESTRICT;
11. ALTER TABLE blog_posts ADD CONSTRAINT fk_blog_posts_author FOREIGN KEY (author_id) REFERENCES authors(author_id) ON DELETE CASCADE;
12. ALTER TABLE enrollments ADD CONSTRAINT fk_enrollments_course FOREIGN KEY (course_id) REFERENCES course(course_id) ON DELETE CASCADE;
> ```

---

## Submission Checklist

- [X] Exercise 1: `.sql` file with all CREATE TABLE statements + written justifications
- [X] Exercise 2: All 12 theory review answers
- [X] Exercise 3: Hotel booking schema with all tables and explanations
- [X] Exercise 4: Data type selections with justifications for all 15 columns
- [X] Exercise 5: All 12 constraints written in valid PostgreSQL syntax
