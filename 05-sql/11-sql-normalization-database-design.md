# SQL — Normalization & Database Design

Database design is a core SQL and Java backend interview topic.

You should understand:

```text
Entities
Relationships
Primary keys
Foreign keys
Constraints
Normalization
1NF
2NF
3NF
BCNF
Denormalization
One-to-one
One-to-many
Many-to-many
Junction tables
Surrogate keys
Natural keys
Cascade behavior
Indexes
Database design trade-offs
```

---

# 1. What Is Database Design?

Database design is the process of deciding:

```text
What data to store
How data is related
How tables should be structured
What constraints should exist
How data should be queried efficiently
```

A good design balances:

```text
Correctness
Consistency
Performance
Maintainability
Scalability
```

---

# 2. Entity

An entity represents something the application needs to store information about.

E-commerce examples:

```text
User
Product
Order
OrderItem
Cart
Payment
Address
```

Usually, an entity becomes a table.

Example:

```text
Product
----------------
id
name
price
stock
category_id
```

---

# 3. Attribute

An attribute describes an entity.

For:

```text
Product
```

attributes might be:

```text
id
name
price
stock
description
```

These commonly become table columns.

---

# 4. Primary Key

A primary key uniquely identifies a row.

Example:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);
```

Properties:

```text
Unique
Not NULL
Identifies one row
```

A table has one primary-key constraint, which can consist of one or multiple columns.

---

# 5. Composite Primary Key

A primary key can contain multiple columns.

Example:

```sql
PRIMARY KEY (order_id, product_id)
```

This means the combination must be unique.

Useful for certain relationship tables.

---

# 6. Foreign Key

A foreign key creates a relationship to another table.

Example:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Conceptually:

```text
users.id
   ↑
   |
orders.user_id
```

This helps enforce referential integrity.

---

# 7. Referential Integrity

Referential integrity means relationships between related tables remain valid according to the defined foreign-key constraints.

For example:

```text
orders.user_id
```

should reference an existing:

```text
users.id
```

when the constraint requires it.

---

# 8. NOT NULL

`NOT NULL` requires a value.

Example:

```sql
email VARCHAR(255) NOT NULL
```

This prevents:

```text
email = NULL
```

But it does not necessarily prevent:

```text
email = ''
```

Application validation may still be required.

---

# 9. UNIQUE

A unique constraint prevents duplicate values according to the database's NULL and constraint semantics.

Example:

```sql
email VARCHAR(255) UNIQUE
```

Common use:

```text
Email
Username
External reference
Order number
```

---

# 10. CHECK

A `CHECK` constraint enforces a condition.

Example:

```sql
price DECIMAL(10,2)
CHECK (price >= 0)
```

This protects data at the database level.

Support and enforcement details vary by database/version.

---

# 11. DEFAULT

A default value is used when a value is omitted from an insert.

Example:

```sql
status VARCHAR(20) DEFAULT 'ACTIVE'
```

Then:

```sql
INSERT INTO users(name)
VALUES ('Sudhir');
```

can result in:

```text
status = ACTIVE
```

---

# 12. Entity Relationships

Common relationships:

```text
One-to-One
One-to-Many
Many-to-Many
```

These are fundamental to relational database design.

---

# 13. One-to-One

Example:

```text
User
 ↓
UserProfile
```

One user has one profile.

Example:

```sql
CREATE TABLE user_profiles (
    id BIGINT PRIMARY KEY,
    user_id BIGINT UNIQUE,
    bio TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

The `UNIQUE` constraint ensures one profile per user.

---

# 14. One-to-Many

Example:

```text
User
 ↓
Orders
```

One user can have many orders.

Database design:

```text
users
-----
id

orders
------
id
user_id
```

The foreign key lives on the many side:

```text
orders.user_id → users.id
```

---

# 15. Many-to-One

From the `orders` perspective:

```text
Many orders
   ↓
One user
```

So:

```text
User = one
Order = many
```

These are two perspectives of the same relationship.

---

# 16. Many-to-Many

Example:

```text
Students
   ↕
Courses
```

A student can take many courses.

A course can have many students.

A relational database typically uses a junction table.

---

# 17. Junction Table

Example:

```text
student
-------
id
name

course
------
id
name

student_course
--------------
student_id
course_id
```

Then:

```sql
PRIMARY KEY (student_id, course_id)
```

can prevent duplicate relationships.

---

# 18. E-Commerce Many-to-Many

Consider:

```text
Orders
Products
```

One order has many products.

One product can appear in many orders.

Use:

```text
order_items
```

Example:

```text
order_items
-----------
order_id
product_id
quantity
price
```

This table represents the relationship plus relationship-specific attributes.

---

# 19. Why OrderItem Is Important

Do not simply store:

```text
order_id
product_id
```

because the relationship itself has information:

```text
quantity
price_at_purchase
discount
```

So:

```text
Order
 ↓
OrderItem
 ↓
Product
```

is a better domain model.

---

# 20. Normalization

Normalization is the process of structuring relational data to reduce unnecessary redundancy and undesirable dependencies.

Main goals:

```text
Reduce duplication
Improve consistency
Prevent update anomalies
Improve data integrity
```

---

# 21. Why Normalization?

Imagine:

```text
orders
-----------------------------------------
order_id | user_name | user_email | item
```

If the same user has:

```text
100 orders
```

their name and email may be repeated:

```text
100 times
```

This creates unnecessary duplication.

Instead:

```text
users
orders
```

store user information once.

---

# 22. Data Anomalies

Poorly normalized designs can cause:

```text
Insert anomaly
Update anomaly
Delete anomaly
```

These are common normalization interview questions.

---

# 23. Insert Anomaly

Suppose one table stores:

```text
student
course
instructor
```

If you cannot add a new course unless a student enrolls in it, the design has an insertion problem.

Separating entities can solve this.

---

# 24. Update Anomaly

Suppose:

```text
customer_name
customer_email
```

is repeated across 50 order rows.

The email changes.

You must update:

```text
50 rows
```

If one row is missed, the database contains inconsistent values.

Normalization reduces this problem.

---

# 25. Delete Anomaly

Suppose the only record of a course exists inside a student's enrollment row.

If you delete the student's enrollment:

```text
Course information disappears too.
```

That is a delete anomaly.

Separating independent entities prevents this.

---

# 26. First Normal Form — 1NF

A table is in first normal form when its columns contain atomic values according to the chosen relational model, rather than repeating groups or multi-valued fields in a single cell.

Bad design:

```text
user_id | phone_numbers
1       | 9999,8888,7777
```

Better:

```text
user_id | phone
1       | 9999
1       | 8888
1       | 7777
```

or a separate phone table.

---

# 27. 1NF Interview Answer

> First Normal Form requires attributes to contain atomic values and avoids repeating groups or multi-valued fields in a single column.

---

# 28. Second Normal Form — 2NF

2NF applies to tables using a composite key.

A table is in 2NF when:

```text
It is in 1NF
+
No non-key attribute depends on only part of a composite candidate key
```

This is called:

```text
No partial dependency.
```

---

# 29. 2NF Example

Suppose:

```text
order_items
--------------------------------
order_id
product_id
product_name
quantity
```

Primary key:

```text
(order_id, product_id)
```

But:

```text
product_name
```

depends only on:

```text
product_id
```

not on the whole composite key.

This is a partial dependency.

---

# 30. Fixing the 2NF Problem

Separate:

```text
products
--------
product_id
product_name
```

and:

```text
order_items
-----------
order_id
product_id
quantity
```

Now:

```text
product_name
```

belongs to the product entity.

---

# 31. 2NF Interview Answer

> Second Normal Form means the table is already in 1NF and every non-key attribute depends on the whole candidate key, not just part of a composite key.

---

# 32. Third Normal Form — 3NF

A table is in 3NF when it is in 2NF and non-key attributes do not depend transitively on another non-key attribute.

In simpler terms:

```text
Non-key attributes should depend on the key,
the whole key,
and nothing but the key.
```

The phrase is useful for interviews, but remember that the formal definition is based on functional dependencies.

---

# 33. 3NF Example

Suppose:

```text
employees
--------------------------------
employee_id
employee_name
department_id
department_name
```

Primary key:

```text
employee_id
```

But:

```text
employee_id
    ↓
department_id
    ↓
department_name
```

`department_name` depends on `department_id`, not directly on the employee key.

This is a transitive dependency.

---

# 34. Fixing 3NF

Use:

```text
employees
---------
employee_id
employee_name
department_id
```

and:

```text
departments
-----------
department_id
department_name
```

Now department information is stored separately.

---

# 35. 3NF Interview Answer

> Third Normal Form means the table is in 2NF and non-key attributes do not have transitive dependencies on the key through another non-key attribute.

---

# 36. BCNF

BCNF stands for:

```text
Boyce-Codd Normal Form
```

It is stronger than 3NF.

A relation is in BCNF when:

```text
For every non-trivial functional dependency X → Y,
X is a superkey.
```

For most Java backend interviews:

```text
Know the concept
Know it is stricter than 3NF
```

You usually don't need advanced decomposition proofs unless the role is database-heavy.

---

# 37. Normalization Summary

```text
1NF
↓
Atomic values

2NF
↓
1NF + no partial dependency

3NF
↓
2NF + no transitive dependency

BCNF
↓
Stronger functional-dependency requirement
```

---

# 38. Denormalization

Denormalization intentionally introduces some redundancy to improve read performance or simplify queries.

Example:

Instead of calculating:

```text
customer
+
orders
+
order_items
```

for every request, you may store:

```text
customer.total_spent
```

and update it carefully.

---

# 39. Why Denormalize?

Possible reasons:

```text
High read volume
Expensive joins
Reporting performance
Reducing repeated calculations
Read-heavy systems
```

But denormalization creates a consistency problem:

```text
Duplicated data
↓
More update paths
↓
Higher chance of stale values
```

---

# 40. Normalization vs Denormalization

Normalization:

```text
Less duplication
Better consistency
More joins
```

Denormalization:

```text
More duplication
Potentially faster reads
Fewer joins
More complex updates
```

---

# 41. When to Normalize

Prefer normalization when:

```text
Data changes frequently
Consistency is critical
Write operations matter
Relationships are important
Duplication would create anomalies
```

---

# 42. When to Denormalize

Consider denormalization when:

```text
Read performance is critical
Data is read much more often than written
Joins are expensive
Access patterns are well understood
The consistency trade-off is acceptable
```

Don't denormalize just because:

```text
"Joins are bad."
```

Good indexes and query design can make joins efficient.

---

# 43. Surrogate Key

A surrogate key is an artificial identifier generated for a row.

Example:

```text
id = 1001
```

rather than using business information as the primary key.

Common examples:

```text
BIGINT ID
UUID
database-generated sequence
```

---

# 44. Natural Key

A natural key comes from real business data.

Examples:

```text
email
ISBN
country_code
tax_identifier
```

Natural keys can be meaningful but may change or have business-specific constraints.

---

# 45. Surrogate vs Natural Key

Surrogate:

```text
Stable
Simple joins
Usually compact with BIGINT
Independent of business meaning
```

Natural:

```text
Meaningful
Can enforce business uniqueness
May change
Can be larger
```

A common design is:

```text
id → surrogate primary key
email → UNIQUE business constraint
```

---

# 46. UUID vs BIGINT

BIGINT:

```text
Small
Efficient indexing
Simple
Sequential values can be useful
```

UUID:

```text
Globally unique
Useful across distributed systems
Harder to guess than sequential IDs in some contexts
Larger indexes/storage
Potentially less locality depending on UUID generation
```

Choose based on system requirements.

---

# 47. Composite Key vs Surrogate Key

Composite:

```text
PRIMARY KEY (order_id, product_id)
```

Useful when the combination naturally identifies the relationship.

Surrogate:

```text
id BIGINT PRIMARY KEY
```

and then:

```text
UNIQUE(order_id, product_id)
```

can also be used.

The right choice depends on domain and access patterns.

---

# 48. Cascade Delete

A foreign key can define behavior when a parent row is deleted.

Conceptually:

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE CASCADE
```

Then deleting a user can automatically delete dependent rows.

Use carefully.

---

# 49. Cascade Risks

Consider:

```text
User
 ↓
Orders
 ↓
OrderItems
 ↓
Products
```

A careless cascade can delete a large amount of data.

For business-critical data such as orders, hard deletion may not even be appropriate.

---

# 50. Soft Delete

Instead of:

```sql
DELETE FROM users
WHERE id = 10;
```

you may use:

```text
deleted_at
```

or:

```text
is_deleted
```

Example:

```sql
UPDATE users
SET deleted_at = CURRENT_TIMESTAMP
WHERE id = 10;
```

Then normal queries filter:

```sql
WHERE deleted_at IS NULL
```

---

# 51. Soft Delete Trade-offs

Advantages:

```text
Recoverability
Audit/history
Avoid accidental data loss
```

Disadvantages:

```text
Every query needs filtering
Indexes may need consideration
Unique constraints become more complex
Storage grows
```

Some databases support specialized indexing/constraint techniques that can help.

---

# 52. E-Commerce Schema

A simple normalized e-commerce design:

```text
users
-----
id
name
email

products
--------
id
name
price
stock

orders
------
id
user_id
status
created_at

order_items
-----------
order_id
product_id
quantity
price_at_purchase
```

Relationships:

```text
User
 ↓ 1:N
Orders
 ↓ 1:N
OrderItems
 ↑ N:1
Product
```

---

# 53. Why price_at_purchase?

Do not rely only on:

```text
products.price
```

for historical orders.

Suppose:

```text
Product price today = ₹1200
```

but customer bought it for:

```text
₹999
```

The order should preserve:

```text
price_at_purchase = ₹999
```

This is an example of intentional historical data storage.

---

# 54. Database Constraints for E-Commerce

Possible constraints:

```sql
users.email UNIQUE

products.price CHECK (price >= 0)

products.stock CHECK (stock >= 0)

orders.user_id FOREIGN KEY

order_items.order_id FOREIGN KEY

order_items.product_id FOREIGN KEY
```

Constraints protect data even if a bug occurs in application code.

---

# 55. Database Design and Java Entities

A Spring/JPA model might contain:

```java
@Entity
public class OrderEntity {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    private User user;
}
```

But JPA mappings should reflect actual database relationships rather than being added only for convenience.

---

# 56. @OneToMany Caution

A common mistake:

```java
@OneToMany
private List<OrderEntity> orders;
```

Developers may assume:

```text
@OneToMany = automatically efficient
```

It isn't.

Be careful with:

```text
Lazy vs eager loading
N+1 queries
Cascade
Orphan removal
Large collections
Serialization
```

---

# 57. Lazy vs Eager

Lazy:

```text
Load relationship when needed
```

Eager:

```text
Load relationship immediately
```

For large collections, eager loading can cause:

```text
Large queries
Memory usage
Unexpected database calls
```

A deliberate fetch strategy is usually better.

---

# 58. Database Design and API Design

Don't automatically expose database entities directly as API responses.

Prefer DTOs when appropriate:

```text
Database Entity
      ↓
Service
      ↓
DTO
      ↓
REST API
```

This prevents database structure from unnecessarily becoming API structure.

---

# 59. Schema Evolution

Production schemas change.

Examples:

```text
Add column
Add index
Create table
Rename column
Create view
Change constraint
```

Use migration tools:

```text
Flyway
Liquibase
```

rather than manually making undocumented production changes.

---

# 60. Backward-Compatible Changes

A safer migration pattern:

```text
1. Add new column
2. Deploy application that can use both versions
3. Backfill data
4. Start writing new field
5. Stop using old field
6. Remove old field later
```

This is useful in systems where old and new application versions may temporarily coexist.

---

# 61. Database Design Checklist

Before creating tables, ask:

```text
What are the entities?
What are their attributes?
What are the relationships?
What is the primary key?
What are the foreign keys?
Which fields must be unique?
Which fields can be NULL?
What constraints are required?
Is the data normalized?
Are there intentional denormalizations?
What are the common queries?
Which indexes are needed?
How will the schema evolve?
```

---

# 62. Interview: What Is Normalization?

> Normalization is the process of structuring relational data to reduce unnecessary redundancy and prevent update, insert and delete anomalies while maintaining data integrity.

---

# 63. Interview: Explain 1NF

> 1NF requires atomic values and avoids storing repeating groups or multiple values in a single column.

---

# 64. Interview: Explain 2NF

> 2NF means the table is in 1NF and every non-key attribute depends on the entire candidate key, so there are no partial dependencies on part of a composite key.

---

# 65. Interview: Explain 3NF

> 3NF means the table is in 2NF and non-key attributes don't depend transitively on the key through another non-key attribute.

---

# 66. Interview: What Is Denormalization?

> Denormalization intentionally introduces some redundancy to improve read performance or simplify frequently used queries. The trade-off is more complex updates and a higher risk of inconsistent duplicated data.

---

# 67. Interview: Why Use a Junction Table?

> A junction table represents a many-to-many relationship in a relational database. For example, `order_items` connects orders and products and can also store relationship-specific attributes such as quantity and price at purchase time.

---

# 68. Interview: Surrogate vs Natural Key

> A surrogate key is an artificial identifier such as a generated numeric ID or UUID. A natural key comes from business data such as an email or ISBN. I commonly use a surrogate primary key and enforce business uniqueness separately when appropriate.

---

# 69. Interview: Why Use Foreign Keys?

> Foreign keys enforce referential integrity between related tables and prevent invalid references according to the defined constraint rules.

---

# 70. Interview: Should Every Relationship Use CASCADE DELETE?

> No. Cascades should be used deliberately. They're useful for tightly owned dependent data, but cascading through business-critical records such as orders can cause unintended large-scale deletion.

---

# 71. Interview: What Is Soft Delete?

> Soft delete marks a record as deleted using a field such as `deleted_at` instead of physically removing it. It helps with recovery and history, but every relevant query must account for deleted rows.

---

# 72. Interview: How Would You Design an E-Commerce Database?

> I'd separate users, products, orders and order items. Users have a one-to-many relationship with orders, and orders and products are connected through order items. I'd use foreign keys and constraints for integrity, keep historical order values such as purchase price, and add indexes based on actual query patterns.

---

# 73. Interview: Would You Normalize Everything?

> Not necessarily. I start with a normalized design for correctness and maintainability, then consider targeted denormalization when measured performance requirements justify the additional consistency and update complexity.

---

# 74. Interview: How Do You Decide Which Indexes to Add?

> I look at real query patterns, filtering and join columns, ordering requirements and data volume. I use `EXPLAIN` and measurements rather than adding indexes to every column because indexes also increase storage and write costs.

---

# 75. Interview: Why Store price_at_purchase?

> Product prices can change after an order is placed. Storing the purchase price on the order item preserves the historical financial value of the transaction.

---

# 76. Interview: Entity vs DTO

> An entity represents persistence/database state, while a DTO represents data exchanged between application layers or through an API. DTOs help avoid exposing persistence details and let the API contract evolve independently.

---

# 77. Interview: What Is Referential Integrity?

> Referential integrity ensures that relationships represented by foreign keys remain valid according to the database constraints, such as an order referencing an existing user.

---

# 78. Interview: What Is an Anomaly?

> An anomaly is an undesirable data-management problem caused by a poor schema design. Common types are insert, update and delete anomalies, which normalization helps reduce.

---

# 79. Interview: Normalization vs Performance

> Normalization improves consistency and reduces duplication, but it can require more joins. I would start with a well-normalized design and optimize measured bottlenecks with indexes, query improvements, caching or carefully chosen denormalization.

---

# 80. Final Mental Model

Think of database design as:

```text
Requirements
    ↓
Entities
    ↓
Relationships
    ↓
Tables
    ↓
Primary / Foreign Keys
    ↓
Constraints
    ↓
Normalize
    ↓
Design common queries
    ↓
Add indexes
    ↓
Measure performance
    ↓
Denormalize only when justified
```

For an e-commerce backend:

```text
User
 │
 │ 1:N
 ↓
Order
 │
 │ 1:N
 ↓
OrderItem
 │
 │ N:1
 ↓
Product
```

The key interview principle is:

> **Start with correctness and a clean normalized model. Then optimize based on actual access patterns and measurements rather than prematurely duplicating data.**
