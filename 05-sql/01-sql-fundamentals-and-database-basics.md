# SQL — Fundamentals & Database Basics

This file builds the foundation for SQL and relational databases before moving into joins, aggregation, subqueries, indexes, transactions, optimization and advanced interview queries.

---

# 1. What Is SQL?

SQL stands for:

```text
Structured Query Language
```

It is used to communicate with relational databases.

Typical operations:

```text
Create data
Read data
Update data
Delete data
```

These are commonly called:

```text
CRUD
```

---

# 2. What Is a Relational Database?

A relational database stores data in tables.

Example:

```text
users
+----+--------+----------------------+
| id | name   | email                |
+----+--------+----------------------+
| 1  | Sudhir | sudhir@example.com   |
| 2  | Rahul  | rahul@example.com    |
+----+--------+----------------------+
```

A table contains:

```text
Rows    → records
Columns → attributes
```

---

# 3. Database vs Table

A database is a container for related data.

Example:

```text
ecommerce
│
├── users
├── products
├── orders
├── order_items
└── payments
```

A table stores a particular type of entity or relationship.

---

# 4. Row

A row represents one record.

Example:

```text
id = 101
name = Laptop
price = 50000
```

This represents one product.

---

# 5. Column

A column represents an attribute.

Example:

```text
products
-------------------------
id
name
price
category
created_at
```

Each column has a defined data type and usually a business meaning.

---

# 6. Primary Key

A primary key uniquely identifies a row.

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);
```

Here:

```text
id
```

is the primary key.

A primary key must uniquely identify each row.

---

# 7. Primary Key Properties

A primary key should be:

```text
Unique
Not NULL
Stable enough for identification
```

A table has one primary key constraint, although that key can contain multiple columns.

---

# 8. Composite Primary Key

A primary key can contain multiple columns.

Example:

```sql
CREATE TABLE order_items (
    order_id BIGINT,
    product_id BIGINT,
    quantity INT,

    PRIMARY KEY (order_id, product_id)
);
```

Here:

```text
(order_id, product_id)
```

together identify one row.

---

# 9. Candidate Key

A candidate key is a column or combination of columns that can uniquely identify a row.

Example:

```text
users
----------------
id
email
```

If both are unique:

```text
id    → candidate key
email → candidate key
```

One candidate key is selected as the primary key.

---

# 10. Alternate Key

A candidate key that is not selected as the primary key can be considered an alternate key.

Example:

```text
id     → PRIMARY KEY
email  → UNIQUE
```

Here, `email` can serve as an alternate unique identifier.

---

# 11. Foreign Key

A foreign key creates a relationship between tables.

Example:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Here:

```text
orders.user_id
        ↓
users.id
```

---

# 12. Why Foreign Keys Matter

Foreign keys help maintain referential integrity.

For example, an order should not normally reference:

```text
user_id = 999999
```

if that user does not exist.

The database can enforce this relationship.

---

# 13. UNIQUE Constraint

A `UNIQUE` constraint prevents duplicate values.

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

Two users cannot have the same email value under this constraint.

---

# 14. NOT NULL

`NOT NULL` means a column must contain a value.

Example:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

This prevents a product from having a NULL name.

---

# 15. DEFAULT

A default value is used when a value is not explicitly supplied.

Example:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    stock INT DEFAULT 0
);
```

If stock is omitted:

```text
stock = 0
```

---

# 16. CHECK Constraint

A `CHECK` constraint can enforce a condition.

Example:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    price DECIMAL(10,2),
    CHECK (price >= 0)
);
```

This prevents negative prices when the database enforces the constraint.

---

# 17. Common SQL Data Types

Common MySQL types include:

```text
INT
BIGINT
DECIMAL
FLOAT
DOUBLE
CHAR
VARCHAR
TEXT
DATE
TIME
DATETIME
TIMESTAMP
BOOLEAN
JSON
```

Choose types according to the data and business requirements.

---

# 18. INT vs BIGINT

`INT` uses less storage and supports a smaller range.

`BIGINT` supports a much larger integer range.

For high-volume tables, choose the smallest type that safely supports the expected values.

Do not automatically use `BIGINT` for every column without considering requirements.

---

# 19. DECIMAL vs FLOAT

For monetary values, prefer:

```text
DECIMAL
```

Example:

```sql
price DECIMAL(10,2)
```

Floating-point types can introduce representation and rounding issues.

For financial calculations, exact decimal arithmetic is generally safer.

---

# 20. CHAR vs VARCHAR

`CHAR(n)` is fixed-length.

```text
CHAR(10)
```

`VARCHAR(n)` is variable-length.

```text
VARCHAR(255)
```

For most variable-length text such as names and emails:

```text
VARCHAR
```

is commonly appropriate.

---

# 21. DATE vs DATETIME vs TIMESTAMP

`DATE`:

```text
2026-08-21
```

`DATETIME`:

```text
2026-08-21 13:30:00
```

`TIMESTAMP` has MySQL-specific behavior around storage/time-zone conversion and automatic timestamp features.

Choose based on the application's time semantics rather than treating them as interchangeable.

---

# 22. NULL

`NULL` means:

```text
Unknown
Missing
Not applicable
```

It is not the same as:

```text
0
```

or:

```text
''
```

---

# 23. NULL Comparison

Incorrect:

```sql
WHERE email = NULL
```

Use:

```sql
WHERE email IS NULL
```

And:

```sql
WHERE email IS NOT NULL
```

---

# 24. Three-Valued Logic

SQL commonly works with:

```text
TRUE
FALSE
UNKNOWN
```

Because of NULL, a comparison may evaluate to UNKNOWN.

This is an important reason SQL filtering behaves differently from ordinary two-valued Boolean logic.

---

# 25. CREATE DATABASE

Example:

```sql
CREATE DATABASE ecommerce;
```

Then:

```sql
USE ecommerce;
```

---

# 26. CREATE TABLE

Example:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0
);
```

---

# 27. INSERT

Insert one row:

```sql
INSERT INTO products
(name, price, stock)
VALUES
('Laptop', 50000.00, 10);
```

---

# 28. Insert Multiple Rows

```sql
INSERT INTO products
(name, price, stock)
VALUES
('Laptop', 50000.00, 10),
('Mouse', 1000.00, 50),
('Keyboard', 2000.00, 30);
```

This is generally preferable to issuing many individual insert statements when inserting a batch of known rows.

---

# 29. SELECT

Retrieve all columns:

```sql
SELECT *
FROM products;
```

In production queries, prefer explicitly selecting the columns you need when practical.

Example:

```sql
SELECT id, name, price
FROM products;
```

---

# 30. WHERE

Filter rows:

```sql
SELECT id, name, price
FROM products
WHERE price > 10000;
```

Only rows satisfying the condition are returned.

---

# 31. Comparison Operators

Common operators:

```text
=
<>
!=
>
<
>=
<=
```

Example:

```sql
SELECT *
FROM products
WHERE stock > 0;
```

---

# 32. AND

```sql
SELECT *
FROM products
WHERE price > 10000
  AND stock > 0;
```

Both conditions must evaluate to TRUE.

---

# 33. OR

```sql
SELECT *
FROM products
WHERE category = 'Laptop'
   OR category = 'Monitor';
```

At least one condition must be TRUE.

---

# 34. NOT

Example:

```sql
SELECT *
FROM products
WHERE NOT category = 'Laptop';
```

Use parentheses when combining complex Boolean expressions so the intended logic is obvious.

---

# 35. BETWEEN

Example:

```sql
SELECT *
FROM products
WHERE price BETWEEN 10000 AND 50000;
```

`BETWEEN` is inclusive of both boundaries.

Conceptually:

```text
price >= 10000
AND
price <= 50000
```

---

# 36. IN

Instead of:

```sql
WHERE category = 'Laptop'
   OR category = 'Monitor'
   OR category = 'Keyboard'
```

you can write:

```sql
WHERE category IN (
    'Laptop',
    'Monitor',
    'Keyboard'
);
```

---

# 37. LIKE

Pattern matching:

```sql
SELECT *
FROM users
WHERE name LIKE 'Sud%';
```

`%` means zero or more characters.

Example:

```text
Sudhir
Sudha
Sud
```

could match.

---

# 38. LIKE Underscore

`_` matches one character.

Example:

```sql
WHERE name LIKE 'S_d%'
```

could match:

```text
Sudhir
Sadi
```

depending on the remaining characters.

---

# 39. ORDER BY

Sort results:

```sql
SELECT *
FROM products
ORDER BY price ASC;
```

Descending:

```sql
ORDER BY price DESC;
```

---

# 40. Multiple Sort Columns

Example:

```sql
SELECT *
FROM products
ORDER BY category ASC,
         price DESC;
```

Rows are first ordered by category, then price within each category.

---

# 41. LIMIT

Return only a certain number of rows:

```sql
SELECT *
FROM products
LIMIT 10;
```

Useful for:

```text
Pagination
Top-N queries
Testing
```

---

# 42. OFFSET

Example:

```sql
SELECT *
FROM products
LIMIT 10 OFFSET 20;
```

Conceptually:

```text
Skip first 20
Return next 10
```

Large offsets can become inefficient on large datasets, which is one reason cursor/keyset pagination is sometimes preferred.

---

# 43. DISTINCT

Remove duplicate result values:

```sql
SELECT DISTINCT category
FROM products;
```

It applies to the selected column combination.

---

# 44. Aliases

Column alias:

```sql
SELECT
    name AS product_name,
    price AS product_price
FROM products;
```

Table alias:

```sql
SELECT p.name
FROM products p;
```

Aliases improve readability, especially in joins.

---

# 45. SQL Comments

Single-line:

```sql
-- Get active products
SELECT *
FROM products;
```

Multi-line:

```sql
/*
  Get products
  with available stock
*/
SELECT *
FROM products
WHERE stock > 0;
```

---

# 46. UPDATE

Example:

```sql
UPDATE products
SET price = 55000
WHERE id = 1;
```

Always be careful with the `WHERE` clause.

---

# 47. Dangerous UPDATE

This:

```sql
UPDATE products
SET price = 55000;
```

updates every row.

Before running an update, confirm:

```text
Which rows should change?
```

---

# 48. DELETE

Example:

```sql
DELETE FROM products
WHERE id = 1;
```

This permanently removes the matching row unless the operation is rolled back or another recovery mechanism exists.

---

# 49. Dangerous DELETE

This:

```sql
DELETE FROM products;
```

deletes all rows from the table.

Always verify the intended scope before executing destructive SQL.

---

# 50. TRUNCATE

Example:

```sql
TRUNCATE TABLE products;
```

It removes all rows and has different transactional, logging and identity/auto-increment behavior from `DELETE`, depending on the database.

Do not treat `TRUNCATE` as simply a faster `DELETE`.

---

# 51. DELETE vs TRUNCATE

General comparison:

```text
DELETE
- Removes rows
- Can use WHERE
- Row-level operation semantics
- Transaction behavior depends on database

TRUNCATE
- Removes all rows
- No WHERE
- Usually treated as a DDL-like operation by databases
- Database-specific transaction/identity behavior
```

For MySQL, understand the storage engine and transaction behavior before making assumptions.

---

# 52. DROP

Example:

```sql
DROP TABLE products;
```

This removes the table definition and its data.

It is far more destructive than deleting rows.

---

# 53. DDL

DDL stands for:

```text
Data Definition Language
```

Common commands:

```text
CREATE
ALTER
DROP
TRUNCATE
```

These define or change database structures.

---

# 54. DML

DML stands for:

```text
Data Manipulation Language
```

Common operations:

```text
INSERT
UPDATE
DELETE
```

They manipulate stored data.

---

# 55. DQL

DQL is commonly used to describe:

```text
SELECT
```

It retrieves data.

The exact classification of SQL statements can vary by textbook, but the terminology is common in interviews.

---

# 56. DCL

DCL stands for:

```text
Data Control Language
```

Common examples:

```text
GRANT
REVOKE
```

These manage database privileges.

---

# 57. TCL

TCL stands for:

```text
Transaction Control Language
```

Common commands include:

```text
COMMIT
ROLLBACK
SAVEPOINT
```

These manage transaction boundaries.

---

# 58. SQL Execution Order

A useful logical processing order is:

```text
FROM
JOIN
WHERE
GROUP BY
HAVING
SELECT
DISTINCT
ORDER BY
LIMIT
```

This is the logical order used to understand query semantics, not necessarily the physical order used internally by the database engine.

---

# 59. Example Query

```sql
SELECT category, COUNT(*) AS product_count
FROM products
WHERE price > 10000
GROUP BY category
HAVING COUNT(*) > 5
ORDER BY product_count DESC
LIMIT 10;
```

Logical flow:

```text
FROM products
↓
WHERE price > 10000
↓
GROUP BY category
↓
HAVING count > 5
↓
SELECT category, count
↓
ORDER BY count
↓
LIMIT 10
```

---

# 60. Schema

A schema describes database structures such as:

```text
Tables
Columns
Constraints
Indexes
Views
```

In MySQL, `schema` and `database` are commonly used interchangeably in many contexts.

---

# 61. Relational Model

A relational database represents relationships using tables and keys.

Example:

```text
users
  |
  | 1
  |
  | N
orders
```

One user can have many orders.

---

# 62. One-to-One

Example:

```text
user
  |
  | 1
  |
  | 1
user_profile
```

A user may have one profile.

A foreign key plus an appropriate unique constraint can enforce the relationship.

---

# 63. One-to-Many

Example:

```text
user
  |
  | 1
  |
  | N
orders
```

Typically:

```text
orders.user_id
```

references:

```text
users.id
```

---

# 64. Many-to-Many

Example:

```text
students
    ↕
student_courses
    ↕
courses
```

A junction/association table represents the relationship.

Example:

```sql
CREATE TABLE student_courses (
    student_id BIGINT,
    course_id BIGINT,
    PRIMARY KEY (student_id, course_id)
);
```

---

# 65. Referential Integrity

Referential integrity ensures relationships remain valid.

Example:

```text
orders.user_id
```

should reference an existing:

```text
users.id
```

when the foreign-key constraint requires it.

---

# 66. ON DELETE

Foreign keys can define delete behavior.

Example:

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE CASCADE
```

Deleting a user may then delete dependent rows.

Use cascading deletes carefully, especially in production systems.

---

# 67. ON UPDATE

Foreign keys can also define update behavior.

Example:

```sql
ON UPDATE CASCADE
```

The exact behavior should be selected according to business requirements.

---

# 68. Surrogate Key

A surrogate key is an artificial identifier.

Example:

```text
id = 100
```

It has no direct business meaning.

Common implementation:

```sql
id BIGINT AUTO_INCREMENT PRIMARY KEY
```

---

# 69. Natural Key

A natural key comes from business data.

Example:

```text
email
ISBN
national product code
```

Natural keys can change or have business-specific constraints, so many systems use a surrogate primary key and enforce uniqueness separately.

---

# 70. AUTO_INCREMENT

MySQL can automatically generate numeric identifiers:

```sql
id BIGINT AUTO_INCREMENT PRIMARY KEY
```

Example:

```text
1
2
3
4
...
```

Do not assume IDs will always be gapless. Deletes, rollbacks, failed inserts and allocation behavior can create gaps.

---

# 71. Constraints

Important constraints:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
DEFAULT
```

Constraints allow the database to enforce data integrity.

---

# 72. Why Constraints Matter

Application validation is useful:

```text
Spring Boot
   ↓
Validation
```

but database constraints provide another layer:

```text
Database
   ↓
Integrity enforcement
```

For important invariants, defense in depth is valuable.

---

# 73. Normalization Preview

Normalization reduces unnecessary duplication and update anomalies.

Common normal forms:

```text
1NF
2NF
3NF
BCNF
```

We will cover normalization in detail later.

---

# 74. Denormalization Preview

Denormalization intentionally duplicates or restructures data to improve read performance or simplify access.

Trade-off:

```text
Faster/easier reads
        vs
More storage and consistency complexity
```

Do not denormalize without a reason.

---

# 75. SQL vs NoSQL

SQL databases:

```text
Relational
Tables
Strong schema
Joins
Transactions
```

NoSQL databases can include:

```text
Document
Key-value
Column-family
Graph
```

Neither is universally better.

Choose based on:

```text
Access patterns
Consistency requirements
Scale
Data model
Operational needs
```

---

# 76. MySQL

MySQL is a relational database management system.

A typical Spring Boot backend might use:

```text
Spring Boot
    ↓
Spring Data JPA
    ↓
Hibernate
    ↓
JDBC
    ↓
MySQL
```

Understanding SQL remains important even when using JPA.

---

# 77. SQL vs JPA

JPA lets Java applications work with persistent entities.

Example:

```java
productRepository.findById(id);
```

Under the hood, Hibernate may generate SQL.

Therefore a Java backend engineer should understand:

```text
SQL
Indexes
Joins
Transactions
Execution plans
```

rather than relying entirely on ORM abstractions.

---

# 78. ORM Problem

An ORM can hide database behavior.

For example:

```java
products.forEach(
    product -> product.getCategory().getName()
);
```

can potentially trigger many database queries depending on mappings and fetch strategy.

This leads to problems such as:

```text
N+1 queries
```

We will cover this later in the SQL + JPA section.

---

# 79. SQL Injection

Never build SQL using unsafe string concatenation.

Bad:

```java
String sql =
    "SELECT * FROM users WHERE email = '" + email + "'";
```

Prefer:

```text
Prepared statements
Parameterized queries
Safe ORM APIs
```

---

# 80. Prepared Statements

Conceptually:

```sql
SELECT *
FROM users
WHERE email = ?;
```

The value is supplied separately.

This prevents user input from being interpreted as SQL syntax in the normal parameter-binding model.

---

# 81. Transactions Preview

A transaction groups operations into a logical unit.

Example:

```text
Create order
   +
Reduce stock
   +
Create order items
```

If one required operation fails, the transaction may need to roll back the related changes.

---

# 82. ACID Preview

Transactions are commonly described using:

```text
Atomicity
Consistency
Isolation
Durability
```

We will cover each in detail later.

---

# 83. Interview: What Is SQL?

> SQL is a language used to interact with relational databases. I use it to define database structures, query data, modify records, manage transactions and work with constraints and permissions.

---

# 84. Interview: Primary Key vs Foreign Key?

> A primary key uniquely identifies a row within its table. A foreign key references a key in another table and is used to represent relationships and maintain referential integrity.

---

# 85. Interview: DELETE vs TRUNCATE vs DROP?

> DELETE removes rows and can use a WHERE condition. TRUNCATE removes all rows and has database-specific transaction and identity behavior. DROP removes the table itself, including its definition.

---

# 86. Interview: What Is NULL?

> NULL represents missing, unknown or not-applicable data. It is not the same as zero or an empty string, and it should be checked using `IS NULL` or `IS NOT NULL` rather than `= NULL`.

---

# 87. Interview: CHAR vs VARCHAR?

> CHAR is fixed-length while VARCHAR is variable-length. For most variable-length values such as names and emails, VARCHAR is usually more appropriate.

---

# 88. Interview: DECIMAL vs FLOAT?

> DECIMAL provides exact decimal arithmetic and is generally preferred for monetary values. Floating-point types can introduce representation and rounding issues.

---

# 89. Interview: What Is a Composite Key?

> A composite key uses multiple columns together to uniquely identify a row. For example, an order-items table can use `(order_id, product_id)` as a composite primary key.

---

# 90. Interview: Why Are Database Constraints Important?

> Constraints let the database enforce data integrity. Primary keys prevent duplicate identities, foreign keys protect relationships, unique constraints prevent duplicates and check/not-null constraints enforce valid data.

---

# 91. Interview: SQL vs JPA?

> SQL directly expresses database operations, while JPA provides a Java persistence abstraction over relational databases. I use JPA for application development but still need SQL knowledge for joins, optimization, indexes, transactions and debugging generated queries.

---

# 92. Interview: What Is Referential Integrity?

> Referential integrity means relationships between tables remain valid. For example, an order's user ID should reference an existing user when a foreign-key constraint enforces that relationship.

---

# 93. Interview: What Is Normalization?

> Normalization is the process of organizing relational data to reduce unnecessary duplication and update anomalies. Common levels include 1NF, 2NF and 3NF.

---

# 94. Quick SQL Cheat Sheet

```sql
-- Create
CREATE TABLE users (...);

-- Insert
INSERT INTO users (...) VALUES (...);

-- Read
SELECT id, name
FROM users;

-- Filter
SELECT *
FROM users
WHERE id = 10;

-- Sort
SELECT *
FROM users
ORDER BY name ASC;

-- Update
UPDATE users
SET name = 'Sudhir'
WHERE id = 10;

-- Delete
DELETE FROM users
WHERE id = 10;

-- Add constraint
ALTER TABLE users
ADD CONSTRAINT uk_users_email UNIQUE (email);

-- Drop table
DROP TABLE users;
```

---

# 95. Fundamentals Checklist

```text
□ SQL
□ Relational database
□ Tables
□ Rows
□ Columns
□ Primary keys
□ Composite keys
□ Candidate keys
□ Foreign keys
□ UNIQUE
□ NOT NULL
□ DEFAULT
□ CHECK
□ NULL
□ Data types
□ SELECT
□ INSERT
□ UPDATE
□ DELETE
□ WHERE
□ ORDER BY
□ DISTINCT
□ LIKE
□ IN
□ BETWEEN
□ LIMIT
□ OFFSET
□ DDL
□ DML
□ DQL
□ DCL
□ TCL
□ Relationships
□ Referential integrity
□ Normalization basics
□ Transactions basics
□ SQL vs JPA
```

---

# 96. Final Mental Model

Think about SQL in layers:

```text
DATABASE
    ↓
TABLES
    ↓
COLUMNS + DATA TYPES
    ↓
KEYS + CONSTRAINTS
    ↓
RELATIONSHIPS
    ↓
SQL QUERIES
    ↓
TRANSACTIONS
    ↓
INDEXES
    ↓
QUERY OPTIMIZATION
```

For a Java backend developer:

```text
Spring Boot
     ↓
JPA / Hibernate
     ↓
JDBC
     ↓
SQL
     ↓
MySQL
```

> **SQL is not just about writing SELECT queries. Strong backend engineers understand data modeling, constraints, relationships, transactions and how application code ultimately interacts with the database.**
