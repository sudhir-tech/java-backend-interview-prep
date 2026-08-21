# SQL — Constraints, Keys & Relationships

Database design is a major part of backend development. Constraints protect data integrity, keys identify records, and relationships connect tables.

For Java/Spring Boot applications, these concepts map directly to:

```text
Entity
↓
Table

@Id
↓
Primary Key

@ManyToOne / @OneToMany
↓
Foreign Key relationship

@NotNull
↓
NOT NULL

@UniqueConstraint
↓
UNIQUE
```

---

# 1. What Is a Constraint?

A constraint is a database rule that controls what data can be stored.

Common constraints:

```text
PRIMARY KEY
FOREIGN KEY
NOT NULL
UNIQUE
CHECK
DEFAULT
```

Constraints protect the database from invalid data.

---

# 2. PRIMARY KEY

A primary key uniquely identifies each row.

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
);
```

Here:

```text
id
```

is the primary key.

A primary key must uniquely identify a row and cannot be NULL.

---

# 3. Primary Key Example

```text
users

id   name
---  ------
1    Sudhir
2    Rahul
3    Priya
```

Each:

```text
id
```

is unique.

You cannot have:

```text
1  Sudhir
1  Rahul
```

because the primary key would be duplicated.

---

# 4. Can a Table Have Multiple Primary Keys?

A table can have only one primary key constraint.

However, that primary key can contain multiple columns.

This is called a:

```text
Composite Primary Key
```

---

# 5. Composite Primary Key

Example:

```sql
CREATE TABLE student_courses (
    student_id BIGINT,
    course_id BIGINT,
    PRIMARY KEY (student_id, course_id)
);
```

The combination:

```text
student_id + course_id
```

must be unique.

Example:

```text
1, 101
1, 102
2, 101
```

is valid.

But:

```text
1, 101
1, 101
```

is not.

---

# 6. FOREIGN KEY

A foreign key creates a relationship between tables.

Example:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

This means:

```text
orders.user_id
       ↓
users.id
```

---

# 7. Why Foreign Keys Matter

Without a foreign key, the database might allow:

```text
orders.user_id = 999999
```

even if user 999999 doesn't exist.

A foreign key can prevent this invalid reference.

This is called:

```text
Referential Integrity
```

---

# 8. Referential Integrity

Referential integrity means relationships between tables remain valid.

For example:

```text
users
  1
  2
  3

orders
  user_id = 1
  user_id = 2
```

The order references existing users.

An order referencing:

```text
user_id = 999
```

would violate the foreign key if user 999 does not exist.

---

# 9. NOT NULL

`NOT NULL` requires a value.

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

This is invalid:

```sql
INSERT INTO users (id, name)
VALUES (1, NULL);
```

---

# 10. UNIQUE

`UNIQUE` prevents duplicate values.

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

Two users cannot have the same email value under this constraint.

---

# 11. PRIMARY KEY vs UNIQUE

Both enforce uniqueness, but they have different roles.

Primary key:

```text
Main identifier of the row
Cannot be NULL
One primary-key constraint per table
```

UNIQUE:

```text
Prevents duplicate values
Can have multiple UNIQUE constraints
NULL behavior depends on the database
```

---

# 12. Multiple UNIQUE Constraints

A table can have multiple unique constraints.

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    username VARCHAR(100) UNIQUE
);
```

Now both:

```text
email
username
```

must be unique.

---

# 13. CHECK

`CHECK` validates a condition.

Example:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    price DECIMAL(10,2),
    CHECK (price >= 0)
);
```

The database rejects a negative price.

---

# 14. CHECK Example

```sql
CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    age INT,
    CHECK (age >= 18)
);
```

This prevents:

```text
age = 15
```

assuming the database enforces the constraint.

---

# 15. DEFAULT

`DEFAULT` provides a value when one isn't supplied.

Example:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    status VARCHAR(30) DEFAULT 'PENDING'
);
```

If the application inserts an order without specifying status, the database can use:

```text
PENDING
```

---

# 16. DEFAULT vs NOT NULL

These are different.

```sql
status VARCHAR(30) DEFAULT 'PENDING'
```

means:

```text
Use PENDING when no value is provided.
```

It does not necessarily prevent explicit NULL.

To require a value:

```sql
status VARCHAR(30) NOT NULL DEFAULT 'PENDING'
```

---

# 17. AUTO_INCREMENT

MySQL commonly uses:

```sql
AUTO_INCREMENT
```

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL
);
```

The database generates IDs automatically.

---

# 18. Generated IDs and Backend Applications

In JPA/Hibernate, this might look like:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

The database generates the identity value.

The exact strategy depends on the database and application requirements.

---

# 19. Natural Key vs Surrogate Key

A:

```text
Natural key
```

comes from business data.

Example:

```text
email
ISBN
passport number
```

A:

```text
Surrogate key
```

is generated specifically to identify the record.

Example:

```text
id = 1001
```

Many backend systems use surrogate IDs because they are stable and simple for relationships.

---

# 20. Primary Key Selection

A good primary key should generally be:

```text
Unique
Stable
Non-null
Suitable for relationships
```

Avoid choosing a value that can change frequently as the primary key.

---

# 21. Foreign Key Example

```text
users
+----+--------+
| id | name   |
+----+--------+
| 1  | Sudhir |
| 2  | Rahul  |
+----+--------+

orders
+-----+---------+--------+
| id  | user_id | amount |
+-----+---------+--------+
| 101 | 1       | 50000  |
| 102 | 1       | 20000  |
| 103 | 2       | 10000  |
+-----+---------+--------+
```

Relationship:

```text
users.id
   ↑
orders.user_id
```

---

# 22. One-to-One Relationship

Example:

```text
User
  |
  1
  |
  1
Profile
```

A user has one profile.

One way to model this:

```sql
CREATE TABLE profiles (
    id BIGINT PRIMARY KEY,
    user_id BIGINT UNIQUE,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

The `UNIQUE` constraint prevents multiple profiles from referencing the same user.

---

# 23. One-to-Many Relationship

Common e-commerce relationship:

```text
User
 |
 1
 |
 N
Orders
```

One user can have many orders.

Database:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

The foreign key is stored on the many side.

---

# 24. Many-to-Many Relationship

Example:

```text
Students
   |
   N
   |
student_courses
   |
   N
   |
Courses
```

A student can take many courses.

A course can have many students.

Use a junction table:

```sql
CREATE TABLE student_courses (
    student_id BIGINT,
    course_id BIGINT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

---

# 25. Junction Table

A junction table is also called:

```text
Bridge table
Association table
Join table
```

It converts:

```text
many-to-many
```

into two:

```text
one-to-many
```

relationships.

---

# 26. E-Commerce Many-to-Many Example

An order can contain many products.

A product can appear in many orders.

Instead of:

```text
orders ↔ products
```

use:

```text
orders
   |
   1
   |
   N
order_items
   |
   N
   |
   1
products
```

---

# 27. Why order_items Exists

`order_items` can store additional relationship data:

```text
order_id
product_id
quantity
price
```

This is important because the relationship itself has attributes.

For example:

```text
Order 101
Product 500
Quantity 3
Price 799
```

---

# 28. Foreign Key Constraint

Example:

```sql
FOREIGN KEY (product_id)
REFERENCES products(id)
```

This means the referenced product must exist unless the relationship allows NULL and no value is supplied.

---

# 29. ON DELETE

Foreign keys can define what happens when the referenced row is deleted.

Example:

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE CASCADE
```

---

# 30. ON DELETE CASCADE

Suppose:

```text
User 1
 ↓
Order 101
 ↓
Order 102
```

If the user is deleted and the relationship uses:

```text
ON DELETE CASCADE
```

the database can automatically delete the dependent orders.

Use cascading deletes carefully.

They can remove many rows from a single operation.

---

# 31. ON DELETE SET NULL

Example:

```sql
FOREIGN KEY (manager_id)
REFERENCES employees(id)
ON DELETE SET NULL
```

If the manager is deleted:

```text
manager_id
```

can become:

```text
NULL
```

The column must allow NULL for this behavior.

---

# 32. ON DELETE RESTRICT

A restrictive behavior prevents deletion of the parent while dependent rows exist.

The exact syntax and default behavior can vary by database.

Conceptually:

```text
Parent has children
      ↓
Cannot delete parent
```

---

# 33. ON UPDATE

Foreign keys can also define behavior when the referenced key changes.

Example:

```sql
ON UPDATE CASCADE
```

However, primary keys are generally designed to be stable, so updating them should be uncommon.

---

# 34. Cascade Caution

Consider:

```text
User
 ↓
Orders
 ↓
Order Items
 ↓
Payments
```

A cascading delete from user to all dependent records could affect a very large amount of data.

For business-critical systems, deletion may instead be handled through:

```text
Soft delete
Retention policies
Archiving
Explicit application workflows
```

---

# 35. Soft Delete

Instead of physically deleting:

```sql
DELETE FROM users
WHERE id = 1;
```

you might use:

```sql
UPDATE users
SET deleted_at = CURRENT_TIMESTAMP
WHERE id = 1;
```

Then queries filter:

```sql
WHERE deleted_at IS NULL
```

This is an application/data-model decision, not a universal rule.

---

# 36. Constraint Naming

Constraints can be explicitly named.

Example:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
);
```

Named constraints make schema maintenance and debugging easier.

---

# 37. Composite UNIQUE Constraint

You can enforce uniqueness across multiple columns.

Example:

```sql
CREATE TABLE enrollments (
    student_id BIGINT,
    course_id BIGINT,
    UNIQUE (student_id, course_id)
);
```

This prevents duplicate enrollment for the same student and course.

---

# 38. Composite UNIQUE vs Composite PRIMARY KEY

Composite primary key:

```sql
PRIMARY KEY (student_id, course_id)
```

Composite unique constraint:

```sql
UNIQUE (student_id, course_id)
```

Both enforce uniqueness of the combination.

The primary key additionally defines the table's main row identity and cannot be NULL.

---

# 39. Candidate Key

A candidate key is a minimal set of columns that can uniquely identify a row.

Example:

```text
id
email
```

If both are guaranteed unique, both may be candidate keys.

One is selected as the primary key.

---

# 40. Alternate Key

A candidate key not chosen as the primary key is commonly called an:

```text
Alternate key
```

Example:

```text
id      → primary key
email   → alternate unique key
```

---

# 41. Superkey

A superkey is any set of columns that uniquely identifies a row.

Example:

```text
id
```

may be a superkey.

So could:

```text
id + name
```

because `id` already uniquely identifies the row.

But `id + name` is not minimal, so it is not a candidate key.

---

# 42. Primary Key vs Foreign Key

Primary key:

```text
Identifies a row in its own table
```

Foreign key:

```text
References a key in another table
```

Example:

```text
users.id
→ primary key

orders.user_id
→ foreign key
```

---

# 43. Can a Foreign Key Reference a UNIQUE Column?

Yes.

A foreign key can reference a candidate/unique key depending on database rules.

Example:

```sql
users.email UNIQUE
```

A related table could reference the unique email column.

However, primary keys are usually preferable for stable relationships.

---

# 44. Can a Foreign Key Be NULL?

Yes, unless the column is declared:

```sql
NOT NULL
```

Example:

```sql
manager_id BIGINT NULL
```

can represent:

```text
employee has no manager
```

---

# 45. Foreign Key and Indexes

Foreign-key columns are often indexed for:

```text
JOIN performance
Parent lookup
DELETE/UPDATE checks
```

The exact indexing behavior depends on the database engine and schema.

Do not assume every database automatically creates every desired index.

---

# 46. Constraints and Transactions

Constraints are checked as part of database operations.

Example:

```text
BEGIN
   ↓
Insert order
   ↓
Insert order item
   ↓
Constraint violation
   ↓
ROLLBACK
```

In a Spring Boot application, transaction management can coordinate multiple database operations.

---

# 47. Constraint Violation in Spring Boot

A database constraint can cause an exception.

Examples:

```text
Duplicate key
Foreign key violation
NOT NULL violation
CHECK violation
```

The application should translate these into meaningful API responses where appropriate.

---

# 48. Entity Example in Spring Boot

A simple entity:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;
}
```

Database concepts:

```text
@Id
→ primary key

@GeneratedValue
→ generated identifier

nullable = false
→ NOT NULL

unique = true
→ UNIQUE
```

---

# 49. JPA Relationship Example

```java
@ManyToOne
@JoinColumn(name = "user_id", nullable = false)
private User user;
```

This commonly corresponds to:

```text
orders.user_id
```

being a foreign key referencing:

```text
users.id
```

---

# 50. One-to-Many in JPA

Example:

```java
@OneToMany(mappedBy = "user")
private List<Order> orders;
```

The owning foreign-key relationship is commonly represented on the `Order` side:

```java
@ManyToOne
@JoinColumn(name = "user_id")
private User user;
```

Understanding the database relationship makes JPA mappings much easier.

---

# 51. Constraint vs Application Validation

You may validate in Java:

```java
@NotNull
private String name;
```

and also enforce:

```sql
name VARCHAR(100) NOT NULL
```

Why both?

Application validation gives:

```text
Fast user-friendly validation
```

Database constraints provide:

```text
Final data-integrity protection
```

The database should not blindly rely on application code to maintain critical invariants.

---

# 52. NOT NULL vs Java Validation

For a request DTO:

```java
@NotBlank
private String name;
```

This validates an incoming API request.

A database:

```sql
name VARCHAR(100) NOT NULL
```

protects the persisted data.

These operate at different layers.

---

# 53. UNIQUE and Race Conditions

Suppose two users simultaneously register:

```text
same@email.com
```

Application code might check:

```text
Does email exist?
```

Both requests could see:

```text
No
```

and then attempt to insert.

A database `UNIQUE` constraint is still necessary to guarantee uniqueness.

The application should handle the resulting constraint violation appropriately.

---

# 54. Primary Key and Distributed Systems

In distributed systems, ID generation may use:

```text
Database identity
UUID
Application-generated IDs
Snowflake-style IDs
```

The choice depends on:

```text
Scale
Ordering requirements
Database
Distributed architecture
Storage/indexing characteristics
```

Do not choose an ID strategy only because it is convenient.

---

# 55. UUID Example

A UUID can be used as an identifier:

```sql
CREATE TABLE users (
    id CHAR(36) PRIMARY KEY,
    name VARCHAR(100)
);
```

Advantages can include:

```text
Can be generated without a central sequence
Harder to guess than sequential IDs
Useful in distributed systems
```

Trade-offs can include:

```text
Larger storage
Larger indexes
Potentially worse locality depending on representation
```

The database-specific implementation matters.

---

# 56. Data Integrity

Database integrity can be thought of as:

```text
Entity integrity
Referential integrity
Domain integrity
```

Entity integrity:

```text
Primary key identifies rows
```

Referential integrity:

```text
Foreign keys maintain relationships
```

Domain integrity:

```text
Column values obey allowed rules
```

Examples:

```text
NOT NULL
CHECK
UNIQUE
```

---

# 57. Constraint Order of Thought

When designing a table, ask:

```text
1. What identifies a row?
2. Which fields are required?
3. Which fields must be unique?
4. Which tables does this reference?
5. What values are allowed?
6. What happens when a parent is deleted?
7. Which columns need indexes?
```

---

# 58. E-Commerce Schema Example

A simplified model:

```text
users
-----
id PK
name
email UNIQUE

products
--------
id PK
name
price CHECK(price >= 0)

orders
------
id PK
user_id FK
status
created_at

order_items
-----------
order_id FK
product_id FK
quantity
price
```

Relationships:

```text
users 1 ─── N orders

orders 1 ─── N order_items

products 1 ─── N order_items
```

---

# 59. Why Store Price in order_items?

Suppose a product costs:

```text
₹500
```

Today.

A customer buys it.

Later the product price becomes:

```text
₹700
```

If the order only references the product's current price, historical order totals can become incorrect.

So an order item often stores:

```text
product_id
quantity
price_at_purchase
```

This is a business/data-modeling decision.

---

# 60. CHECK Constraint for Quantity

Example:

```sql
CREATE TABLE order_items (
    order_id BIGINT,
    product_id BIGINT,
    quantity INT NOT NULL,
    CHECK (quantity > 0)
);
```

This prevents:

```text
quantity = 0
quantity = -5
```

---

# 61. CHECK Constraint for Order Status

Depending on database/version, you could use:

```sql
CHECK (status IN ('PENDING', 'PAID', 'CANCELLED'))
```

This restricts the allowed values.

Another approach is a reference table or application/domain enum, depending on the system's needs.

---

# 62. Foreign Key vs Index

A foreign key and an index are different.

Foreign key:

```text
Enforces relationship
```

Index:

```text
Improves lookup/query performance
```

A foreign key does not mean:

```text
"this column is automatically the best index for every query"
```

---

# 63. Can Two Tables Have Multiple Foreign Keys?

Yes.

Example:

```text
orders
-----
created_by
approved_by
```

Both could reference:

```text
users.id
```

SQL:

```sql
FOREIGN KEY (created_by) REFERENCES users(id),
FOREIGN KEY (approved_by) REFERENCES users(id)
```

This represents two different relationships to the same table.

---

# 64. Self-Referencing Foreign Key

Example:

```sql
CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    manager_id BIGINT,
    FOREIGN KEY (manager_id)
        REFERENCES employees(id)
);
```

The table references itself.

This supports hierarchical structures.

---

# 65. Circular Foreign Keys

Be careful when:

```text
Table A references B
Table B references A
```

Insertion and deletion can become complicated.

Prefer simpler relationships where possible.

If circular dependencies are necessary, transaction/order-of-insertion strategies may be required.

---

# 66. Constraint Naming Convention

A common style:

```text
pk_users
fk_orders_user
uk_users_email
chk_products_price
```

Naming conventions make schema errors easier to understand.

Exact naming standards vary by organization.

---

# 67. Schema Migration

In real Spring Boot projects, database changes should generally be managed through migration tools such as:

```text
Flyway
Liquibase
```

Example change:

```text
V2__add_order_status.sql
```

Migrations provide:

```text
Versioning
Repeatability
Deployment consistency
Auditability
```

---

# 68. Why Not Manually Change Production Tables?

Manual production changes can cause:

```text
Environment differences
Lost changes
Deployment problems
Rollback difficulty
```

Migration tools allow database schema changes to be part of the application's deployment process.

---

# 69. Interview: What Is a Primary Key?

> A primary key uniquely identifies each row in a table. It cannot be NULL, and a table has one primary key constraint, although that key can contain multiple columns.

---

# 70. Interview: What Is a Foreign Key?

> A foreign key establishes a relationship between tables by referencing a key in another table. It helps enforce referential integrity, such as ensuring an order references an existing user.

---

# 71. Interview: Primary Key vs Foreign Key

> A primary key identifies a row in its own table, while a foreign key references a key in another table to represent a relationship.

---

# 72. Interview: Primary Key vs UNIQUE

> Both can enforce uniqueness, but the primary key is the table's main identifier and cannot be NULL. A table can have multiple unique constraints for other fields such as email or username.

---

# 73. Interview: What Is a Composite Key?

> A composite key uses multiple columns together to uniquely identify a row. A common example is a junction table where `student_id` and `course_id` together identify an enrollment.

---

# 74. Interview: What Is Referential Integrity?

> Referential integrity means relationships between tables remain valid. For example, if `orders.user_id` references `users.id`, the database can prevent an order from referencing a user that doesn't exist.

---

# 75. Interview: What Is ON DELETE CASCADE?

> It tells the database to automatically delete dependent rows when the referenced parent row is deleted. I use it carefully because a single delete can cascade into many records.

---

# 76. Interview: What Is a Many-to-Many Relationship?

> It means multiple rows on each side can be related. Relational databases usually represent it using a junction table containing foreign keys to both entities.

---

# 77. Interview: Why Do We Need Database Constraints if We Validate in Java?

> Application validation improves the API experience, but database constraints provide the final protection for persisted data. They also protect against concurrent requests, scripts, admin tools or other applications that might bypass the application validation.

---

# 78. Interview: Can a Foreign Key Be NULL?

> Yes, unless the foreign-key column is declared NOT NULL. A nullable foreign key can represent an optional relationship, such as an employee who currently has no manager.

---

# 79. Interview: Why Is UNIQUE Important for Email?

> Application-level checks alone can have race conditions when two requests arrive simultaneously. A database UNIQUE constraint guarantees that duplicate emails cannot be committed, and the application can handle the constraint violation.

---

# 80. Interview: What Is a Surrogate Key?

> A surrogate key is an artificial identifier created specifically for the database, such as a numeric ID or UUID. It doesn't represent business meaning and can provide a stable identifier for relationships.

---

# 81. Interview: What Is a Natural Key?

> A natural key comes from real business data, such as an ISBN or an email address. It can be useful when the business field is guaranteed to be unique and stable, but many systems still use a surrogate primary key.

---

# 82. Interview: What Is a Junction Table?

> A junction table represents a many-to-many relationship by storing foreign keys to both entities. It can also store attributes of the relationship, such as quantity or price in an order-item relationship.

---

# 83. Constraints Checklist

```text
□ PRIMARY KEY
□ Composite primary key
□ FOREIGN KEY
□ Referential integrity
□ NOT NULL
□ UNIQUE
□ CHECK
□ DEFAULT
□ AUTO_INCREMENT
□ Surrogate key
□ Natural key
□ Candidate key
□ Alternate key
□ Composite UNIQUE
□ One-to-one
□ One-to-many
□ Many-to-many
□ Junction table
□ ON DELETE
□ ON UPDATE
□ CASCADE
□ SET NULL
□ RESTRICT
□ Soft delete
□ Foreign-key indexes
□ Constraint naming
□ Spring/JPA mapping
□ Flyway/Liquibase
```

---

# 84. Final Mental Model

Think of database design like this:

```text
PRIMARY KEY
    ↓
"What identifies this row?"

FOREIGN KEY
    ↓
"What other row does this belong to?"

NOT NULL
    ↓
"Is this value required?"

UNIQUE
    ↓
"Can two rows have this value?"

CHECK
    ↓
"Is this value valid?"

DEFAULT
    ↓
"What should happen when no value is supplied?"
```

Relationships:

```text
1 : 1
→ one-to-one

1 : N
→ one-to-many

N : N
→ junction table
```

For backend development:

```text
Java Entity
    ↓
Database Table
    ↓
Primary Key
    ↓
Foreign Keys
    ↓
Constraints
    ↓
Indexes
    ↓
Transactions
```

> **Good backend code protects data at multiple layers. Validate requests in the application, model relationships correctly, and use database constraints to enforce the rules that must never be violated.**
