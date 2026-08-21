# SQL — Views, Stored Procedures & Triggers

This file covers database features commonly discussed in SQL and backend interviews:

```text
Views
Materialized views
Stored procedures
Functions
Triggers
Events
Advantages
Disadvantages
When to use them
Spring Boot considerations
Interview questions
```

---

# 1. What Is a View?

A view is a virtual table based on a SQL query.

Example:

```sql
CREATE VIEW active_users AS
SELECT id, name, email
FROM users
WHERE status = 'ACTIVE';
```

You can then query it:

```sql
SELECT *
FROM active_users;
```

A normal view generally stores the query definition rather than a separate copy of the result data.

---

# 2. Why Use a View?

Views can provide:

```text
Simplified queries
Abstraction
Reusable query logic
Restricted access to columns/rows
Cleaner reporting queries
```

Example:

Instead of repeatedly writing:

```sql
SELECT id, name, email
FROM users
WHERE status = 'ACTIVE';
```

you can use:

```sql
SELECT *
FROM active_users;
```

---

# 3. View Example with JOIN

Suppose:

```text
users
orders
```

You can create:

```sql
CREATE VIEW user_order_summary AS
SELECT
    u.id AS user_id,
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name;
```

Then:

```sql
SELECT *
FROM user_order_summary;
```

This can simplify reporting queries.

---

# 4. Does a View Store Data?

A normal view generally does not store a separate copy of the query result.

Conceptually:

```text
SELECT from view
       ↓
Database uses view definition
       ↓
Underlying tables
```

However, materialized views are different.

---

# 5. Materialized View

A materialized view stores the result of a query.

Conceptually:

```text
Complex query
     ↓
Materialized result
     ↓
Stored data
```

This can make repeated reads faster.

But the stored result needs to be refreshed when underlying data changes.

---

# 6. Normal View vs Materialized View

| Feature | View | Materialized View |
|---|---|---|
| Stores result data | Usually no | Yes |
| Read performance | Depends on underlying query | Can be much faster |
| Freshness | Usually reflects current data | Depends on refresh strategy |
| Storage | Low | Requires storage |
| Maintenance | Query definition | Refresh/maintenance required |

Materialized views are database-specific. Not every database supports them in the same way.

---

# 7. View for Security

A view can expose only selected columns.

Suppose:

```text
users
```

contains:

```text
id
name
email
password_hash
salary
```

You could create:

```sql
CREATE VIEW public_users AS
SELECT id, name, email
FROM users;
```

Applications/users querying the view don't need direct access to sensitive columns.

Views can therefore be part of a database access-control strategy.

They are not a replacement for proper application and database security.

---

# 8. Updating a View

Some views are updatable.

For example, a simple view based on one table may allow:

```sql
UPDATE active_users
SET name = 'Sudhir'
WHERE id = 10;
```

But not every view is updateable.

Views involving:

```text
GROUP BY
Aggregations
DISTINCT
Complex joins
UNION
```

may have restrictions.

Exact rules depend on the database.

---

# 9. Dropping a View

```sql
DROP VIEW active_users;
```

This removes the view definition.

It does not normally delete the underlying table data.

---

# 10. CREATE OR REPLACE VIEW

Some databases support:

```sql
CREATE OR REPLACE VIEW active_users AS
SELECT id, name, email
FROM users
WHERE status = 'ACTIVE';
```

This is useful when modifying a view definition.

Syntax and capabilities vary by database.

---

# 11. Stored Procedure

A stored procedure is a named program stored in the database that can execute database operations.

Conceptually:

```text
Application
    ↓
CALL procedure
    ↓
Database executes logic
```

Example syntax varies by database.

MySQL example:

```sql
DELIMITER //

CREATE PROCEDURE get_user_orders(IN userId BIGINT)
BEGIN
    SELECT *
    FROM orders
    WHERE user_id = userId;
END //

DELIMITER ;
```

Call it:

```sql
CALL get_user_orders(10);
```

---

# 12. Why Use Stored Procedures?

Possible reasons:

```text
Complex database-side operations
Reusable database logic
Reduced application/database round trips
Centralized database logic
Legacy enterprise systems
Security/access-control patterns
```

But stored procedures also have trade-offs.

---

# 13. Stored Procedure Disadvantages

Potential disadvantages:

```text
Database-specific syntax
Harder application-level testing
Logic split between application and database
Deployment complexity
Potential vendor lock-in
Harder version control in some workflows
```

Modern applications often keep most business logic in application services, but stored procedures can still be appropriate in certain systems.

---

# 14. Stored Procedure vs Java Service

Java service:

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

Stored procedure:

```text
Application
   ↓
Stored Procedure
   ↓
Database
```

A procedure can contain multiple database operations.

---

# 15. Stored Function

A stored function generally returns a value.

Example concept:

```sql
CREATE FUNCTION calculate_discount(price DECIMAL(10,2))
RETURNS DECIMAL(10,2)
...
```

You might then use:

```sql
SELECT calculate_discount(1000);
```

Exact syntax depends on the database.

---

# 16. Procedure vs Function

A common interview distinction:

```text
Procedure
→ performs an operation
→ may return result sets/output parameters depending on DB

Function
→ generally returns a value
→ often usable inside SQL expressions
```

The exact rules vary across database systems.

---

# 17. Trigger

A trigger is database logic that automatically executes when a specified database event occurs.

Common events:

```text
INSERT
UPDATE
DELETE
```

Example concept:

```text
INSERT user
   ↓
Trigger executes
   ↓
Audit record created
```

---

# 18. Trigger Example

Suppose:

```text
users
user_audit
```

A trigger could automatically record changes to users.

Conceptually:

```sql
CREATE TRIGGER user_audit_trigger
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
    INSERT INTO user_audit(user_id, changed_at)
    VALUES (NEW.id, CURRENT_TIMESTAMP);
END;
```

Syntax differs by database.

---

# 19. BEFORE Trigger

A `BEFORE` trigger runs before the triggering operation is completed.

Conceptual use cases:

```text
Validate data
Transform values
Set derived fields
```

Example:

```text
BEFORE INSERT
```

can modify/validate values before insertion, depending on the database.

---

# 20. AFTER Trigger

An `AFTER` trigger runs after the triggering operation.

Common use:

```text
Audit
History
Derived side effects
```

Example:

```text
UPDATE users
   ↓
AFTER UPDATE trigger
   ↓
Insert audit record
```

---

# 21. Trigger Events

Typical combinations:

```text
BEFORE INSERT
AFTER INSERT

BEFORE UPDATE
AFTER UPDATE

BEFORE DELETE
AFTER DELETE
```

Availability and behavior vary by database.

---

# 22. OLD and NEW

Triggers often expose:

```text
OLD
NEW
```

Conceptually:

```text
INSERT
→ NEW available

DELETE
→ OLD available

UPDATE
→ OLD + NEW
```

Example:

```text
OLD.status = 'PENDING'
NEW.status = 'PAID'
```

This is useful for audit logic.

Exact trigger syntax differs by database.

---

# 23. Audit Trigger Example

Suppose:

```text
orders.status
```

changes.

A trigger can record:

```text
order_id
old_status
new_status
changed_at
```

This provides a database-level audit trail.

---

# 24. Trigger Advantages

Triggers can provide:

```text
Automatic execution
Centralized database-side enforcement
Audit trails
Automatic derived updates
Protection even when multiple applications write to the same database
```

---

# 25. Trigger Disadvantages

Triggers can also cause problems:

```text
Hidden side effects
Harder debugging
Unexpected performance costs
Complex transaction behavior
Application developers may not know they exist
Testing becomes harder
Database portability decreases
```

This is why triggers should be used deliberately.

---

# 26. Hidden Side Effect Example

Application code:

```sql
UPDATE users
SET status = 'ACTIVE'
WHERE id = 10;
```

Developer may think:

```text
One UPDATE
```

But a trigger could also execute:

```text
INSERT audit
UPDATE statistics
INSERT notification
```

So the real database work may be much larger.

---

# 27. Trigger and Transaction

A trigger generally executes as part of the transaction that caused it.

Conceptually:

```text
BEGIN
  ↓
UPDATE users
  ↓
Trigger executes
  ↓
Trigger changes
  ↓
COMMIT
```

If the transaction rolls back, trigger changes that participate in that transaction generally roll back too.

Exact behavior depends on the database and trigger type.

---

# 28. Trigger vs Application Event

Application event:

```text
Java code
   ↓
Event
   ↓
Listener
```

Database trigger:

```text
SQL operation
   ↓
Trigger
   ↓
Database-side logic
```

Application events are easier to keep visible in application code.

Triggers can protect data regardless of which application performs the database write.

---

# 29. Trigger vs @Transactional

These solve different problems.

`@Transactional`:

```text
Defines transaction behavior in the application
```

Trigger:

```text
Automatically executes database logic in response to database events
```

They can work together.

---

# 30. Stored Procedure and Spring Boot

Spring applications can call stored procedures.

For example, using JPA:

```java
@Procedure("get_user_orders")
List<Order> getUserOrders(Long userId);
```

Or using:

```text
JdbcTemplate
```

depending on the procedure and result structure.

---

# 31. JdbcTemplate Example

Conceptually:

```java
jdbcTemplate.call(
    connection -> {
        CallableStatement statement =
            connection.prepareCall("{call get_user_orders(?)}");

        statement.setLong(1, userId);

        return statement;
    },
    List.of()
);
```

The exact implementation depends on the procedure's parameters and return values.

---

# 32. When Procedures Make Sense

Stored procedures can make sense when:

```text
Heavy database-side processing
Legacy database systems
Existing enterprise procedures
Complex reporting
Strict database-side access patterns
Multiple applications share the same database logic
```

---

# 33. When Java Service Logic Is Better

Application-side logic is often preferable when:

```text
Business logic is complex
Logic needs strong unit testing
Application has rich domain behavior
Portability matters
Developers want one main codebase
Business rules change frequently
```

This is not an absolute rule.

Architecture depends on the system.

---

# 34. View vs Table

Table:

```text
Stores actual data
```

View:

```text
Usually stores a query definition
```

Example:

```text
users → table

active_users → view
```

---

# 35. View vs Materialized View

View:

```text
Query definition
↓
Result generated when queried
```

Materialized view:

```text
Query
↓
Stored result
↓
Refresh
```

Materialized views trade storage and refresh complexity for potentially faster reads.

---

# 36. View vs Stored Procedure

View:

```text
Represents queryable data
```

Procedure:

```text
Executes a sequence of database operations
```

You query:

```sql
SELECT *
FROM active_users;
```

You call:

```sql
CALL get_user_orders(10);
```

---

# 37. Function vs Stored Procedure

Function:

```text
Returns a value
```

Procedure:

```text
Performs an operation
```

But database-specific implementations can blur the exact distinction.

Always know the behavior of the database you're using.

---

# 38. Trigger vs Stored Procedure

Stored procedure:

```text
Explicitly called
```

Trigger:

```text
Automatically executed by a database event
```

Example:

```sql
CALL update_order_summary();
```

versus:

```text
UPDATE orders
↓
trigger automatically fires
```

---

# 39. Should We Put Business Logic in Triggers?

Usually, avoid putting large amounts of business logic into triggers.

Why?

```text
Hidden execution
Harder debugging
Harder testing
Tight database coupling
```

A small, clearly justified audit/data-integrity trigger can be reasonable.

---

# 40. Views in Backend APIs

Suppose an API needs:

```text
User ID
Name
Total Orders
Total Spent
```

Instead of repeating a complicated query across multiple repositories, a view can expose:

```text
user_order_summary
```

Then the backend can query:

```sql
SELECT *
FROM user_order_summary
WHERE user_id = ?;
```

---

# 41. Reporting Use Case

Views are especially useful for reporting queries.

Example:

```text
Monthly sales report
Customer order summary
Product performance
```

A view can encapsulate the underlying joins and calculations.

---

# 42. Performance Warning with Views

A view is not automatically a performance optimization.

If:

```text
View
↓
Complex joins
↓
Millions of rows
```

the query can still be expensive.

A normal view generally does not magically cache its results.

Use:

```text
EXPLAIN
```

to inspect the actual query plan.

---

# 43. Materialized View for Reporting

Suppose:

```text
Daily sales report
```

requires expensive aggregation over millions of rows.

A materialized view can store the calculated result.

Then:

```text
Dashboard
↓
Materialized view
```

can be much faster.

But you need a refresh strategy.

---

# 44. Stored Procedure and Transactions

A stored procedure can execute multiple operations.

For example:

```text
Create invoice
Update account
Insert audit
```

Depending on the database and procedure design, these can participate in transaction handling.

Do not assume procedure calls automatically have the same semantics across databases.

---

# 45. Trigger Performance

A trigger can make a simple write expensive.

Example:

```text
INSERT order
↓
Trigger
↓
INSERT audit
↓
UPDATE summary
↓
additional work
```

At high traffic:

```text
10,000 inserts
↓
10,000 trigger executions
```

This hidden work can become a bottleneck.

---

# 46. Trigger Debugging

If an apparently simple SQL statement is unexpectedly slow:

```text
Check triggers
Check indexes
Check foreign keys
Check locks
Check constraints
Check execution plan
```

Triggers are easy to overlook.

---

# 47. Database Portability

SQL features such as:

```text
Triggers
Stored procedures
Functions
Materialized views
```

often have database-specific syntax and behavior.

Therefore:

```text
MySQL
PostgreSQL
Oracle
SQL Server
```

may implement them differently.

This can increase vendor lock-in.

---

# 48. Version Control

Database objects should ideally be version-controlled through migration tools such as:

```text
Flyway
Liquibase
```

Example:

```text
V10__create_user_view.sql
V11__create_order_trigger.sql
```

This keeps schema and database-object changes reproducible across environments.

---

# 49. Spring Boot + Flyway

A common architecture:

```text
Spring Boot
    ↓
Flyway migrations
    ↓
Database schema
    ↓
Tables / indexes / views / procedures
```

For production systems, database changes should be repeatable and traceable.

---

# 50. Database Migration

Instead of manually changing production:

```text
CREATE VIEW ...
```

you can put the change in a migration:

```sql
-- V10__create_active_users_view.sql

CREATE VIEW active_users AS
SELECT id, name, email
FROM users
WHERE status = 'ACTIVE';
```

Then Flyway applies it consistently.

---

# 51. Security Consideration

Do not expose sensitive database objects unnecessarily.

For example:

```text
password_hash
payment information
internal audit data
```

should not accidentally become accessible through a public view.

Use:

```text
Least privilege
Database roles
Restricted views
Application authorization
```

---

# 52. Interview: What Is a View?

> A view is a virtual table defined by a SQL query. It usually doesn't store a separate copy of the result and can simplify complex queries, provide abstraction and restrict exposed columns.

---

# 53. Interview: View vs Table

> A table stores data, while a normal view generally stores a query definition that presents data from underlying tables.

---

# 54. Interview: What Is a Materialized View?

> A materialized view stores the result of a query so repeated reads can be faster. The trade-off is additional storage and the need to refresh the result when source data changes.

---

# 55. Interview: What Is a Stored Procedure?

> A stored procedure is a named program stored in the database that can execute one or more database operations. It can be useful for complex database-side processing or legacy enterprise systems.

---

# 56. Interview: Procedure vs Function

> A procedure is generally designed to perform an operation, while a function generally returns a value and can often be used within SQL expressions. The exact capabilities depend on the database.

---

# 57. Interview: What Is a Trigger?

> A trigger is database logic that automatically executes when a specified event such as INSERT, UPDATE or DELETE occurs.

---

# 58. Interview: Why Can Triggers Be Dangerous?

> Triggers can introduce hidden side effects. A simple SQL statement may execute additional database operations that aren't obvious from the application code, making debugging, testing and performance analysis harder.

---

# 59. Interview: When Would You Use a Trigger?

> I would use a trigger when database-level enforcement or auditing is genuinely required, especially when multiple applications write to the same database. I would avoid putting large amounts of business logic into triggers.

---

# 60. Interview: Do Views Improve Performance?

> Not automatically. A normal view usually just encapsulates a query. Its performance depends on the underlying query and execution plan. A materialized view can improve read performance by storing the result, but it introduces refresh and storage costs.

---

# 61. Interview: Can Spring Boot Call Stored Procedures?

> Yes. Spring Boot can call stored procedures using JPA features such as `@Procedure` or lower-level APIs such as `JdbcTemplate`.

---

# 62. Interview: How Do You Manage Database Objects in Production?

> I prefer version-controlled database migrations using tools such as Flyway or Liquibase so schema changes, indexes and other database objects can be applied consistently across environments.

---

# 63. Interview: Would You Put Business Logic in a Trigger?

> Usually no. I prefer keeping most business logic in the application service layer because it's easier to test, maintain and version. I'd use triggers selectively for things such as auditing or database-level invariants.

---

# 64. Interview: View vs Stored Procedure

> A view is primarily a queryable representation of data, while a stored procedure is executable database-side logic that can perform multiple operations.

---

# 65. Interview: Trigger vs Stored Procedure

> A stored procedure is explicitly invoked, while a trigger is automatically invoked by a database event such as INSERT, UPDATE or DELETE.

---

# 66. Interview: Does a Trigger Run Inside the Transaction?

> Generally, trigger work participates in the transaction that caused the trigger event, so it follows the transaction's commit or rollback behavior. Exact semantics depend on the database.

---

# 67. Backend Architecture Mental Model

A practical Java backend might look like:

```text
Client
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database

                    ┌──────────────┐
                    │ Views        │
                    │ Procedures   │
                    │ Functions    │
                    │ Triggers     │
                    └──────────────┘
                           ↓
                       Database
```

The important question isn't:

```text
"Can I use a stored procedure?"
```

It is:

```text
"Where should this logic live,
and what are the maintenance,
performance, testing and consistency trade-offs?"
```

---

# 68. Final Checklist

```text
□ View
□ CREATE VIEW
□ DROP VIEW
□ Updatable views
□ Materialized views
□ View security
□ Stored procedure
□ Stored function
□ Procedure vs function
□ Trigger
□ BEFORE trigger
□ AFTER trigger
□ INSERT / UPDATE / DELETE triggers
□ OLD / NEW
□ Trigger advantages
□ Trigger disadvantages
□ Trigger performance
□ Trigger transactions
□ View performance
□ Spring Boot procedure calls
□ JdbcTemplate
□ @Procedure
□ Flyway
□ Liquibase
□ Database migrations
□ Database portability
□ Hidden side effects
□ Business logic placement
```

---

# 69. Final Mental Model

Remember the distinction:

```text
TABLE
→ stores data

VIEW
→ presents data through a query

MATERIALIZED VIEW
→ stores query results

FUNCTION
→ returns a value

STORED PROCEDURE
→ executes database-side operations

TRIGGER
→ automatically reacts to database events
```

For a Java backend interview, don't just define these features.

Be ready to explain:

```text
Why would you use it?
What are the trade-offs?
How does it affect Spring Boot?
How does it affect performance?
How would you maintain it in production?
```

> **Interview shortcut:** A strong answer is usually more valuable than a definition: "I prefer application-level business logic for maintainability, but I use database features such as views, procedures or triggers when they solve a specific database-level problem and the trade-offs are understood."
