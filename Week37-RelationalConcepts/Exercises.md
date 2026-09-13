# Week 37 — Exercises & Project Task

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 37 Theory material. Complete all sections.

---

## Part 1: TrailShop Project Task

### Task 1: Identify Keys

Using the `products`, `categories`, and `customers` tables shown in Section 2 of this week's Theory material, answer:

1. What is the primary key of the `products` table? Why is it a good choice?

> [!NOTE]
> ***Your Answer***
>
> Its the product_id because its unique for each product and isnt likely to change that much. The value is also never NULL.
>
>
>
>


2. What is the primary key of the `categories` table?

> [!NOTE]
> ***Your Answer***
>
> Its the categorie_id because for the same reasons as the product_id.
>
>
>
>


3. What is the foreign key in the `products` table? What does it reference?

> [!NOTE]
> ***Your Answer***
>
> Its the product_id for the same reasons as the product_id and categorie_id.
>
>
>
>


4. Is `name` in `products` a candidate key? Under what assumption? What would make it unsuitable as a primary key?


> [!NOTE]
> ***Your Answer***
>
> Its only suitable, if we assume that all products will have a unique name which is not realistic and risky. If we assume, that the names arent unique, then its not suitable for a primary key. 
>
>
>
>
5. Give an example of a **superkey** for the `products` table that is NOT a candidate key. Explain why it's not minimal.

> [!NOTE]
> ***Your Answer***
>
> {product_id, name} is a superkey, but not a candidate key, because its not minimal, because name is not adding any value to it.
>
>
>
>

6. Give an example of a **composite key** using a hypothetical `order_items` table. Explain why neither column alone would be sufficient.

> [!NOTE]
> ***Your Answer***
>
> (order_id, product_id) is a composite key, because order_id alone can appear for example if you buy more than one product in the same order. Product_id alone can appear for example if you order the same product more than once. So individually, they are not unique alone.
>
>
>
>

7. Is `email` in `customers` a candidate key? What makes it different from `customer_id` as a PK choice? *(See Section 6.9 on natural vs surrogate keys.)*

> [!NOTE]
> ***Your Answer***
>
> email can be a candidate key, but again risky. It can change easiely and can also appear twice. Thats why its not a good choice as a primary key. customer_id is better because its a surrogate key and is guaranteed to be unique and never change. 
>
>
>
>

### Task 2: Define Business Rules

List **5 business rules** for TrailShop. For each rule, specify:
- The rule in plain English
- Which constraint type(s) would enforce it
- Which table and column the constraint applies to
- The SQL syntax for the constraint

Example:

| Business Rule | Constraint Type | Table.Column | SQL |
|---|---|---|---|
| Every product must have a price greater than zero | CHECK | products.price | `CHECK (price > 0)` |
| ... | ... | ... | ... |

Think about rules for customers, orders, and categories — not just products.

> [!NOTE]
> ***Your Answer***
>
> | Business Rule | Constraint Type | Table.Column | SQL |
|---|---|---|---|
| Every customer must have a unique ID | PRIMARY KEY | customer_id | `PRIMARY KEY (customer_id)` |
| Every email must be unique | UNIQUE | customers.email | `UNIQUE (email)` |
| Every order_item must have a unique order_id and product_id combination | PRIMARY KEY | order_items | `PRIMARY KEY (order_id, product_id)` |
| Product price must be greater than zero | CHECK | products.price | `CHECK (price > 0)` |
| category_id in products must exist in categories | FOREIGN KEY | products.category_id | `FOREIGN KEY (category_id) REFERENCES categories(category_id)` |
| ... | ... | ... | ... |
>
>
>
>

### Task 3: Integrity Violations

For each SQL statement below, predict whether it will **succeed** or **fail**. If it fails, explain which integrity rule or constraint is violated and what error message you'd expect. Assume the schema from Section 9.8 of the Theory material.

```sql
-- Statement A
INSERT INTO categories (category_id, category_name)
VALUES (NULL, 'Cycling');

-- Statement B
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (109, 'AeroLite Tent', 279.00, 10, 2);

-- Statement C
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (110, 'BudgetBoots', -5.00, 25, 1);

-- Statement D
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (103, 'Duplicate Shoes', 99.99, 5, 3);

-- Statement E
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (111, 'CloudWalker Sandals', 65.00, 40, 10);

-- Statement F
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (112, NULL, 89.99, 20, 1);

-- Statement G
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (113, 'LightStep Shoes', 149.00, -3, 1);

-- Statement H
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (1001, 101, 0, 189.50);
```

> [!NOTE]
> ***Your Answer***
>
> Statement A: FAIL, because categorie_id can't be null.
Statement B: Succeed
Statement C: Fail, because price is negative.
Statement D: Succeed, unless 103 isnt used (otherwise it would fail, becuase product_id is already used and has to be unique).
Statement E: Fail, because category_id 10 doesnt exist.
Statement F: Fail, because name cant be NULL
Statement G: Fail, because stock_quantity is negative.
Statement H: Fail, because quantity is 0.
>
>
>

### Task 4: Foreign Key Actions

Consider the following scenario using the schema from Theory Section 9.8:

1. You want to delete category 2 ("Camping") from the `categories` table. Products 102 and 106 reference this category. What happens with:
   - `ON DELETE RESTRICT`?
   - `ON DELETE CASCADE`?
   - `ON DELETE SET NULL`? (Assume `category_id` in `products` allows NULL for this question)

2. Which foreign key action would you recommend for the TrailShop `products.category_id` → `categories.category_id` relationship? Justify your choice in 2–3 sentences.

> [!NOTE]
> ***Your Answer***
>
> On delete restrict: prevent the category from beeing deleted
On delete cascade: Deleting the referenced rows too
On delete set null: Set the Values to NULL

I would recommend ON DELETE RESTRICT or ON DELETE SET NULL, because it deletes the categorie, but keeps the products you maybe want to add to another categorie.
>
>
>




---

## Part 2: Theory Review Questions

Answer each question in 2–4 sentences unless otherwise specified. Reference the Theory material sections as needed.

### Short-Answer Questions

**Q1.** Define the following terms in your own words: relation, tuple, attribute, domain. Give one TrailShop example for each.

> [!NOTE]
> ***Your Answer***
>
> Section 2 & 3
Relation: a table, a set of tuples
tuples: Rows in the table (relation)
attribute: column in a table (relation)
Domain: Set of all allowed values for a attribute (column)

Relation: /customers/
tuples: 1 | test@xamk.fi | Test | 0401234567 | 123 Main St
attribute: customer id, email, name, phone, address
Domain: VARCHAR (n), INTEGER

>
>
>
>

*(See Sections 2 and 3 of this week's Theory material.)*

**Q2.** What makes a candidate key different from a primary key? Can a table have more than one candidate key?


> [!NOTE]
> ***Your Answer***
>
> Section 6
>
> A candidate key has to be unique and minimal. A primary key doesnt have to be minimal, but must be unique. Every candidate key is a primary key, but not vice versa. You can have more than one candidate key, for example product id and usually name.
> 
>
> 
> 
>  
>

*(See Section 6 of this week's Theory material.)*

**Q3.** Explain entity integrity in your own words. Why can't a primary key be NULL?


> [!NOTE]
> ***Your Answer***
>
> Section 8.1 

It says, that every relation must have a primary key and it cannot be NULL.
Imagine it would be NULL, you couldnt uniquely identify the tuples (rows) without the primary key and the system would break down. You also cant find the row to update or delete it.
>
>
>
>

*(See Section 8.1 of this week's Theory material.)*

**Q4.** What happens when referential integrity is violated? Give a concrete TrailShop example — show the SQL statement and the expected error.

> [!NOTE]
> ***Your Answer***
>
> Section 8.2

Then you would have orphan records, for example rows that refer to something, that doesnt exist. 

INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (109, 'Ghost Product', 59.99, 5, 99);

ERROR:  insert or update on table "products" violates foreign key constraint
        "products_category_id_fkey"
DETAIL:  Key (category_id)=(99) is not present in table "categories".
>
>
>
>

*(See Section 8.2 of this week's Theory material.)*

**Q5.** Explain the difference between a surrogate key and a natural key. Give an example of each for a `books` table in a library database.

> [!NOTE]
> ***Your Answer***
>
> A surrogate key is an artificial key, that has no business meaning, for example an auto generated product id. A natural key is derived from the business domain, for example a product's ISBN number.
>
>
>

*(See Section 6.8–6.9 of this week's Theory material.)*

**Q6.** What is a NULL value? Why is `WHERE price = NULL` wrong? What should you write instead?


> [!NOTE]
> ***Your Answer***
>
> A NULL value cant be identified. Its wrong, because you can only use an equal sign in combination with numbers. You should write IS NULL instead.
>
>
>
>

*(See Section 7 of this week's Theory material.)*

**Q7.** What is a junction table? When is it needed? Give an example.

> [!NOTE]
> ***Your Answer***
>
> A Junction table is a table where you can see the references between two tables. Its needed when you have a M:N (many-to-many) relationship between two tables. 

CREATE TABLE book_authors (
    isbn      CHAR(13) REFERENCES books(isbn),
    author_id INTEGER  REFERENCES authors(author_id),
    PRIMARY KEY (isbn, author_id)
);
>
>
>
>

*(See Section 12.3 of this week's Theory material.)*

**Q8.** Describe the three types of relationships (1:1, 1:N, M:N). For each, give one TrailShop example.

> [!NOTE]
> ***Your Answer***
>
> 1:1: One row in Table A has exactly one connection to a row in Table B
products (1) ──── (1) product_details
1:N: One row in Table A has multiple connections to rows in Table B
products (1) ──── (N) product_reviews
M:N: Many rows in Table A have multiple connections to rows in Table B
customers (M) ──── (N) orders
>
>
>

*(See Section 12 of this week's Theory material.)*

**Q9.** What is the difference between `ON DELETE CASCADE` and `ON DELETE RESTRICT`? When would you use each?


> [!NOTE]
> ***Your Answer***
>
> ON DELETE RESTRICT prevents you from deleting a referenced value
ON DELETE CASCADE deletes the referenced value too. On default you should use ON DELETE RESTRICT, because otherwise the system deletes related values and you lose data.
> 
>
>

*(See Section 10 of this week's Theory material.)*

**Q10.** Explain what "atomic entries" means in the context of relation properties. Give an example of a violation.

> [!NOTE]
> ***Your Answer***
>
> It means that every value in a cell is atomic, in other words, it contains only a single value, that cant be split again. 

For Example

product_id | name                    | categories  
101        | Alpine Pro Hiking Boots | Footwear, Hiking

The cell "Footwear, Hiking" is not atomic — it contains two values, separated by a comma, which can cause violation, because its not defined in the table schema.
>
>
>
>

*(See Section 5.3 of this week's Theory material.)*

### True/False

For each statement, write **True** or **False** and correct any false statements.

1. A superkey is always a candidate key.
2. A primary key can consist of more than one column.
3. NULL = NULL evaluates to TRUE in SQL.
4. A foreign key must always be NOT NULL.
5. Referential integrity ensures that every FK value matches an existing PK value (or is NULL).
6. The degree of a relation is the number of rows.

### Matching Exercise

Match each term (1–12) with its definition (A–L).

| # | Term |
|---|---|
| 1 | Superkey |
| 2 | Candidate key |
| 3 | Composite key |
| 4 | Foreign key |
| 5 | Alternate key |
| 6 | Surrogate key |
| 7 | Natural key |
| 8 | Orphan record |
| 9 | Domain |
| 10 | Junction table |
| 11 | Cardinality |
| 12 | COALESCE |

| Letter | Definition |
|---|---|
| A | The set of all permitted values for an attribute |
| B | A key composed of two or more attributes |
| C | A row whose FK references a non-existent PK — forbidden by referential integrity |
| D | An artificial key with no business meaning (e.g., auto-generated ID) |
| E | A candidate key not chosen as the primary key |
| F | Any set of attributes that uniquely identifies every tuple |
| G | A minimal superkey — no attribute can be removed without losing uniqueness |
| H | A column that references the primary key of another table |
| I | The number of tuples (rows) in a relation |
| J | A key drawn from real-world data with business meaning |
| K | A table implementing a many-to-many relationship |
| L | A SQL function that returns the first non-NULL argument |


> [!NOTE]
> ***Your Answers***
>
> | # | Your Match |
> |---|---|
> | 1 |F|
> | 2 |G|
> | 3 |B|
> | 4 |H|
> | 5 |E|
> | 6 |D|
> | 7 |J|
> | 8 |C|
> | 9 |A|
> | 10 |K|
> | 11 |I|
> | 12 |L|
>

---

## Part 3: SQL Practice — Constraints in Action

These exercises test your understanding of constraints. You do NOT need to run these in PostgreSQL (but you may if you'd like to verify your answers).

### Exercise 3.1: Predict the Outcome

Given the following table definitions:

```sql
CREATE TABLE departments (
    dept_id   INTEGER      PRIMARY KEY,
    dept_name VARCHAR(50)  NOT NULL UNIQUE
);

CREATE TABLE employees (
    emp_id    INTEGER       PRIMARY KEY,
    name      VARCHAR(100)  NOT NULL,
    salary    NUMERIC(10,2) NOT NULL CHECK (salary >= 0),
    dept_id   INTEGER       NOT NULL REFERENCES departments(dept_id)
);
```

Assume these rows already exist:

```sql
INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO departments VALUES (2, 'Marketing');
INSERT INTO employees VALUES (100, 'Alice', 75000, 1);
INSERT INTO employees VALUES (101, 'Bob', 65000, 2);
```

For each statement below, predict: **SUCCESS** or **FAIL**? If fail, name the violated constraint.

```sql
-- 1
INSERT INTO employees VALUES (102, 'Carol', 70000, 1);
--> SUCCESS
-- 2
INSERT INTO employees VALUES (103, 'Dan', -5000, 1);
--> FAIL, salary must be positive
-- 3
INSERT INTO employees VALUES (100, 'Eve', 80000, 2);
--> FAIL, employee_id 100 already exists

-- 4
INSERT INTO employees VALUES (104, 'Frank', 60000, 5);
--> FAIL, dept_id 5 does not exist

-- 5
INSERT INTO departments VALUES (3, 'Engineering');
--> FAIL, dept_name 'Engineering' already exists

-- 6
INSERT INTO employees VALUES (105, NULL, 55000, 2);
--> FAIL, name must be not null

-- 7
DELETE FROM departments WHERE dept_id = 1;
--> FAIL, referential integrity violation

-- 8
INSERT INTO employees VALUES (106, 'Grace', 0, 2);
--> SUCCESS
```

### Exercise 3.2: Write the Constraints

Given these business rules for a **bookstore database**, write the `CREATE TABLE` statements with appropriate constraints:

1. Every book has a unique ISBN (13 characters), a title (required), a price (must be positive), and a publication year.
2. Every author has an ID, a first name (required), and a last name (required).
3. A book can have multiple authors, and an author can write multiple books.
4. Every book belongs to exactly one genre. Genres have an ID and a unique name.
5. Publication year must be between 1450 and the current year.

*(Hint: you'll need at least 4 tables, including a junction table for the M:N relationship.)*

> [!NOTE]
> ***Your Answer***
>
> CREATE TABLE books (
    isbn             CHAR(13)      PRIMARY KEY,
    title            VARCHAR(255)  NOT NULL,
    price            NUMERIC(10,2) NOT NULL CHECK (price > 0),
    publication_year INTEGER       CHECK (publication_year >= 1450
                                      AND publication_year <= EXTRACT(YEAR FROM CURRENT_DATE)),
    genre_id         INTEGER       NOT NULL REFERENCES genres(genre_id)
);

>CREATE TABLE authors (
    author_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL
);
>
>CREATE TABLE book_authors (
    isbn      CHAR(13) REFERENCES books(isbn),
    author_id INTEGER  REFERENCES authors(author_id),
    PRIMARY KEY (isbn, author_id)
);

> CREATE TABLE genres (
    genre_id   INTEGER     PRIMARY KEY,
    genre_name VARCHAR(50) NOT NULL UNIQUE
);

---

## Part 4: Design Exercise — Library System

A small public library needs a database. Here is a description of their requirements:

> The library has a collection of **books**. Each book has an ISBN, a title, a publication year, and belongs to one genre (Fiction, Non-Fiction, Science, History, etc.). The library may own multiple **copies** of the same book — each copy has a unique barcode sticker.
>
> The library has registered **members**. Each member has a member number, name, email, and phone. Members can **borrow** copies. Each borrowing records which member borrowed which copy, the borrow date, the due date, and the return date (NULL if not yet returned).
>
> **Rules:**
> - A member can borrow at most 5 copies at any given time.
> - The due date is always 14 days after the borrow date.
> - A copy cannot be borrowed if it's currently not returned (return_date IS NULL).

### Your Tasks

1. **Identify the tables** you would need (list them with their columns).
2. **Identify the primary key** for each table. Are they surrogate or natural keys? Justify your choices.
3. **Identify all foreign keys** and the tables they reference.
4. **Identify any candidate keys** beyond the primary key (alternate keys).
5. **List the business rules** from the description and map each to a constraint type. Which rules cannot be enforced by simple constraints?


> [!NOTE]
> ***Your Answer***
> Tables:
> books: isbn (natural primary Key), title, publication_year, genre_id 
> genre: genre_id (surrogate primary Key), name (candidate key)
> copies: copy_id (surrogate primary Key), barcode (candidate key)
> members: member_id (surrogate primary Key), name, email (candidate key), phone
> borrowings: borrow_id (surrogate primary Key), member_id (foreign Key, references members), copy_id (foreign Key, references copies), borrow_date, due_date, return_date
> 
6. **Write the CREATE TABLE statements** for at least the `books`, `copies`, and `borrowings` tables with full constraints.

> [!NOTE]
> ***Your Answer***
> CREATE TABLE books(
    isbn CHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    publication_year INTEGER NOT NULL,
    genre_id INTEGER NOT NULL REFERENCES genres(genre_id)
);

CREATE TABLE copies(
    copy_id INTEGER PRIMARY KEY,
    barcode INTEGER NOT NULL
)

CREATE TABLE borrowings(
    borrow_id INTEGER PRIMARY KEY,
    member_id INTEGER NOT NULL REFERENCES memebers(member_id),
    copy_id INTEGER NOT NULL REFERENCES copies(copy_id),
    borrow_date DATE NOT NULL,
    due_date DATE NOT NULL CHECK (due_date = borrow_date + 14 days),
    return_date DATE
)
> 

---

## Submission Checklist

- [ ] Task 1: Key identification answers (Part 1)
- [ ] Task 2: Business rules table with 5 rules (Part 1)
- [ ] Task 3: Integrity violation predictions with explanations (Part 1)
- [ ] Task 4: Foreign key action analysis (Part 1)
- [ ] Theory Review Questions answered (Part 2)
- [ ] SQL Practice — constraint predictions and bookstore CREATE TABLE (Part 3)
- [ ] Library System design exercise (Part 4)
