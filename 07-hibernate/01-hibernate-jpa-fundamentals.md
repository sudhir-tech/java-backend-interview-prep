# Hibernate & JPA — File 01: Fundamentals

This is the first Hibernate/JPA interview-preparation file.

Core topics:

```text
JPA
Hibernate
JPA vs Hibernate
ORM
Entity
Persistence Context
EntityManager
Session
Entity Lifecycle
@Id
@GeneratedValue
@Table
@Column
Basic Mapping
Primary Keys
Repositories
Spring Data JPA
Transactions
Dirty Checking
```

---

# 1. What Is ORM?

ORM means:

```text
Object Relational Mapping
```

It maps:

```text
Java Objects
      ↕
Database Tables
```

Example:

```text
Java
User
 ├── id
 ├── name
 └── email

Database
users
 ├── id
 ├── name
 └── email
```

Instead of manually converting every database row into a Java object, an ORM framework can handle much of that mapping.

---

# 2. Why Use ORM?

Without ORM, application code often needs to handle:

```text
SQL
ResultSet
Object creation
Type conversion
Connection handling
```

ORM reduces repetitive database-access code.

Benefits:

```text
Less boilerplate
Object-oriented programming model
Automatic mapping
Transaction support
Caching
Dirty checking
Relationship mapping
```

---

# 3. What Is JPA?

JPA stands for:

```text
Java Persistence API
```

It is a Java specification for persistence and ORM.

JPA defines concepts and APIs such as:

```text
@Entity
@Id
EntityManager
Persistence Context
JPQL
Relationships
```

JPA itself is not the implementation.

---

# 4. What Is Hibernate?

Hibernate is an ORM framework and one of the most widely used JPA implementations.

Think:

```text
JPA
 ↓
Specification / API
 ↓
Hibernate
 ↓
Actual ORM implementation
 ↓
Database
```

---

# 5. JPA vs Hibernate

This is a very common interview question.

Answer:

> "JPA is a specification that defines a standard persistence API and ORM model, while Hibernate is an implementation of JPA. Hibernate also provides additional features beyond the JPA specification."

Simple mental model:

```text
JPA = contract

Hibernate = implementation
```

---

# 6. Spring Data JPA

Spring Data JPA sits above JPA.

Conceptually:

```text
Application
     ↓
Spring Data JPA
     ↓
JPA
     ↓
Hibernate
     ↓
JDBC
     ↓
Database
```

Spring Data JPA reduces repository boilerplate.

---

# 7. Hibernate Architecture

A simplified flow:

```text
Spring Boot Application
          ↓
Spring Data JPA
          ↓
EntityManager / JPA
          ↓
Hibernate
          ↓
JDBC
          ↓
Database
```

---

# 8. Entity

An entity is a Java class mapped to a database table.

Example:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    private Long id;

    private String name;
    private String email;
}
```

The class represents persistent data.

---

# 9. @Entity

`@Entity` tells JPA:

> This class is a persistent entity.

Example:

```java
@Entity
public class User {
}
```

Without `@Entity`, JPA does not treat the class as a normal persistent entity.

---

# 10. Entity Naming

By default, the entity name is typically the class name:

```java
@Entity
public class User {
}
```

JPQL can refer to:

```text
User
```

rather than directly using the database table name.

---

# 11. @Table

`@Table` specifies database table details.

Example:

```java
@Entity
@Table(name = "users")
public class User {
}
```

Now:

```text
User entity
      ↓
users table
```

---

# 12. @Id

Every entity needs an identifier.

Example:

```java
@Id
private Long id;
```

The ID uniquely identifies an entity instance.

---

# 13. @GeneratedValue

The primary key can be generated automatically.

Example:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

The exact generation strategy should match the database and application requirements.

---

# 14. GenerationType.IDENTITY

Example:

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

The database generates the identifier, commonly using an identity/auto-increment mechanism.

Common example:

```text
MySQL AUTO_INCREMENT
```

---

# 15. GenerationType.SEQUENCE

Example:

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```

Uses a database sequence where supported.

Common with databases such as:

```text
PostgreSQL
Oracle
```

---

# 16. GenerationType.AUTO

```java
@GeneratedValue(strategy = GenerationType.AUTO)
```

Lets the persistence provider choose a suitable strategy.

For production systems, understand what strategy your database/provider actually selects.

---

# 17. @Column

`@Column` controls column mapping.

Example:

```java
@Column(name = "email_address", nullable = false)
private String email;
```

This maps:

```text
Java:
email

Database:
email_address
```

---

# 18. Common @Column Attributes

Examples:

```java
@Column(
    name = "email_address",
    nullable = false,
    unique = true,
    length = 255
)
```

Common attributes:

```text
name
nullable
unique
length
insertable
updatable
```

---

# 19. Basic Mapping

Example:

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

Conceptually:

```text
users
--------------------------------
id | name | email
--------------------------------
```

---

# 20. Java Types and Database Types

Hibernate maps Java types to appropriate database types.

Examples:

```text
String
Integer
Long
Boolean
BigDecimal
LocalDate
LocalDateTime
```

The exact SQL type depends on the database and mapping configuration.

---

# 21. Why Prefer LocalDate/LocalDateTime?

Modern Java applications commonly use:

```java
LocalDate
LocalDateTime
Instant
```

instead of older date APIs.

For distributed systems, `Instant` is often useful when representing an absolute point in time.

---

# 22. Entity Constructors

JPA entities generally need a no-argument constructor.

Example:

```java
@Entity
public class User {

    protected User() {
    }

    // other constructors
}
```

A protected/package-private no-argument constructor is commonly used so application code doesn't accidentally create an incomplete entity.

---

# 23. Entity Class Rules

A JPA entity should generally:

```text
Be annotated with @Entity
Have an identifier
Have a no-argument constructor
Not be final
Avoid final persistent fields/methods when proxying is needed
```

Exact requirements can vary with provider features and bytecode enhancement.

---

# 24. Why Hibernate Uses Proxies?

Hibernate can use proxies/lazy-loading mechanisms to delay loading associated data.

Example:

```text
Order loaded
   ↓
Customer not loaded yet
   ↓
Access customer
   ↓
Hibernate loads it
```

This is one reason entity classes need to be compatible with Hibernate's proxy/enhancement mechanisms.

---

# 25. Entity Lifecycle

A JPA entity can move through states:

```text
Transient
Managed
Detached
Removed
```

This is extremely important for interviews.

---

# 26. Transient

A newly created object that is not associated with a persistence context.

Example:

```java
User user = new User();
user.setName("Sudhir");
```

At this point:

```text
Java object exists
Database row does not
```

---

# 27. Managed

An entity becomes managed when associated with a persistence context.

Example:

```java
entityManager.persist(user);
```

Now Hibernate tracks it.

```text
Java object
   ↓
Persistence Context
   ↓
Hibernate tracks changes
```

---

# 28. Detached

A previously managed entity becomes detached when it is no longer associated with the persistence context.

Example situations:

```text
Transaction ends
EntityManager is cleared
EntityManager is closed
```

The exact behavior depends on the surrounding persistence context configuration.

---

# 29. Removed

An entity marked for deletion.

Example:

```java
entityManager.remove(user);
```

The corresponding row will be deleted when the persistence operation is flushed/committed.

---

# 30. Lifecycle Summary

```text
Transient
   |
   | persist()
   ↓
Managed
   |
   | detach()/clear()/close
   ↓
Detached

Managed
   |
   | remove()
   ↓
Removed
```

---

# 31. Persistence Context

The persistence context is a set of entity instances managed by an `EntityManager`.

Think of it as:

```text
EntityManager
      ↓
Persistence Context
      ↓
Managed Entities
```

Hibernate tracks these entities.

---

# 32. Why Is Persistence Context Important?

It provides behavior such as:

```text
First-level cache
Identity guarantee within context
Dirty checking
Entity lifecycle management
```

---

# 33. First-Level Cache

The persistence context acts as a first-level cache.

Example:

```java
User u1 = entityManager.find(User.class, 1L);
User u2 = entityManager.find(User.class, 1L);
```

Within the same persistence context, Hibernate can return the same managed entity instance rather than issuing another database query.

Conceptually:

```text
find(1)
 ↓
DB query

find(1)
 ↓
Persistence Context
 ↓
existing managed entity
```

---

# 34. First-Level Cache Is Per Persistence Context

Important:

```text
Persistence Context A
→ Cache A

Persistence Context B
→ Cache B
```

It is not a global application-wide cache.

---

# 35. Second-Level Cache

Hibernate can also support a second-level cache.

Conceptually:

```text
Application
   ↓
Persistence Context
   ↓
Second-Level Cache
   ↓
Database
```

Unlike first-level cache, it can be shared across persistence contexts.

It requires explicit configuration and a cache provider.

---

# 36. First-Level vs Second-Level Cache

```text
First-Level
→ Built into persistence context
→ Per EntityManager/session
→ Always present

Second-Level
→ Optional
→ Shared across persistence contexts
→ Requires configuration
```

---

# 37. EntityManager

`EntityManager` is a standard JPA API for interacting with the persistence context.

Common operations:

```java
persist()
find()
merge()
remove()
detach()
clear()
flush()
```

---

# 38. persist()

Makes a new entity managed.

Example:

```java
entityManager.persist(user);
```

The SQL `INSERT` may be executed at flush/commit rather than immediately, depending on the ID strategy and flush behavior.

---

# 39. find()

Loads an entity by primary key.

Example:

```java
User user =
    entityManager.find(User.class, 1L);
```

It checks the persistence context before hitting the database.

---

# 40. remove()

Marks a managed entity for deletion.

Example:

```java
entityManager.remove(user);
```

Deletion is generally executed during flush/commit.

---

# 41. merge()

`merge()` copies the state of a detached/new entity into a managed entity.

Important:

```java
User managed = entityManager.merge(detachedUser);
```

The returned object is the managed instance.

Do not assume the original object becomes managed.

---

# 42. Common merge Interview Trap

Wrong mental model:

```text
merge()
↓
original object becomes managed
```

Better:

```text
detached object
      ↓
merge()
      ↓
managed copy
```

Use the returned instance when you need the managed object.

---

# 43. detach()

Removes an entity from the persistence context.

```java
entityManager.detach(user);
```

After that:

```text
Hibernate stops automatically tracking that entity.
```

---

# 44. clear()

Clears the persistence context.

```java
entityManager.clear();
```

All currently managed entities become detached.

---

# 45. flush()

Synchronizes changes in the persistence context with the database.

```java
entityManager.flush();
```

Important:

> `flush()` is not the same as `commit()`.

---

# 46. Flush vs Commit

```text
flush
→ Synchronize persistence changes with DB

commit
→ Complete the database transaction
```

A transaction can flush SQL and still roll back before commit.

---

# 47. Dirty Checking

One of Hibernate's most important features.

Suppose:

```java
@Transactional
public void updateUser(Long id) {

    User user = repository.findById(id).orElseThrow();

    user.setName("New Name");
}
```

You don't explicitly call:

```java
repository.save(user);
```

after modifying a managed entity.

Hibernate can detect the change and generate:

```sql
UPDATE users
SET name = ?
WHERE id = ?
```

during flush.

---

# 48. How Dirty Checking Works

Conceptually:

```text
Load entity
   ↓
Hibernate remembers state
   ↓
Application changes entity
   ↓
Hibernate detects difference
   ↓
UPDATE during flush
```

---

# 49. Dirty Checking Requires a Managed Entity

If the entity is detached:

```text
Detached entity
 ↓
Change field
 ↓
Hibernate isn't tracking it
```

No automatic dirty checking occurs for that detached instance.

---

# 50. Transaction Boundary

Dirty checking is commonly used inside:

```java
@Transactional
```

Example:

```java
@Transactional
public void updateUser(Long id) {

    User user = repository.findById(id)
                           .orElseThrow();

    user.setEmail("new@example.com");
}
```

At transaction completion:

```text
Dirty checking
 ↓
Flush
 ↓
UPDATE
 ↓
Commit
```

---

# 51. Spring Data JPA Repository

A common repository:

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

Spring Data provides methods such as:

```text
save()
findById()
findAll()
deleteById()
existsById()
```

---

# 52. Repository Abstraction

Application:

```text
Service
  ↓
Repository
  ↓
JPA
  ↓
Hibernate
  ↓
JDBC
  ↓
Database
```

This keeps database-access code cleaner.

---

# 53. save() Is Not Always "INSERT"

A common interview trap.

```java
repository.save(entity);
```

may result in:

```text
INSERT
```

or:

```text
UPDATE
```

depending on whether Spring Data JPA considers the entity new.

The exact behavior depends on entity identity/newness detection and mapping.

---

# 54. findById()

Example:

```java
Optional<User> user =
    userRepository.findById(id);
```

It generally performs an entity lookup by primary key.

---

# 55. existsById()

Useful when you only need existence:

```java
if (userRepository.existsById(id)) {
    ...
}
```

This can avoid loading the entire entity.

---

# 56. Derived Query Methods

Spring Data JPA can derive queries from method names.

Example:

```java
List<User> findByName(String name);
```

Or:

```java
Optional<User> findByEmail(String email);
```

---

# 57. Multiple Conditions

Example:

```java
List<User> findByNameAndEmail(
    String name,
    String email
);
```

Spring Data derives the query from the method name.

---

# 58. When Derived Queries Become Too Long

Avoid creating methods like:

```text
findByNameAndEmailAndAgeGreaterThanAndStatusAnd...
```

For complex queries, consider:

```text
JPQL
Specifications
Criteria API
QueryDSL
Native SQL
```

depending on requirements.

---

# 59. Entity Relationships

JPA supports:

```text
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

These are extremely important interview topics.

We will cover them deeply in the next files.

---

# 60. Example: Many Orders Belong to One Customer

```text
Customer
   |
   +---- Order
   +---- Order
   +---- Order
```

Typical mapping:

```java
@ManyToOne
private Customer customer;
```

---

# 61. Example: Order Has Many Items

```text
Order
 |
 +-- OrderItem
 +-- OrderItem
 +-- OrderItem
```

Possible mapping:

```java
@OneToMany
private List<OrderItem> items;
```

The exact ownership and join-column design matter and will be covered separately.

---

# 62. Lazy Loading

Lazy loading means related data is loaded when accessed rather than immediately.

Conceptually:

```text
Load Order
 ↓
Order available

Items?
 ↓
Not loaded yet

order.getItems()
 ↓
Hibernate loads items
```

---

# 63. Eager Loading

Eager loading means associated data is loaded as part of the fetch strategy.

Conceptually:

```text
Load Order
 ↓
Order + association loaded
```

Eager loading can cause unnecessary database work if overused.

---

# 64. Lazy vs Eager

General rule:

> Prefer lazy loading for large or optional associations and explicitly fetch what a use case needs.

Do not blindly make everything eager.

---

# 65. N+1 Problem

Suppose:

```text
1 query → orders
N queries → customer for each order
```

Total:

```text
1 + N queries
```

This can seriously hurt performance.

We will cover solutions in detail later.

---

# 66. Common N+1 Solutions

Depending on the use case:

```text
JOIN FETCH
EntityGraph
Batch fetching
DTO projection
Explicit query design
```

Don't automatically solve every N+1 issue by making relationships eager.

---

# 67. SQL Generated by Hibernate

Hibernate can generate SQL automatically.

Example Java:

```java
userRepository.findById(1L);
```

may result in:

```sql
SELECT
    u.id,
    u.name,
    u.email
FROM users u
WHERE u.id = ?
```

The exact SQL depends on mapping, Hibernate version and database dialect.

---

# 68. SQL Logging

During development, you can inspect generated SQL.

Spring Boot configuration can enable SQL logging.

For example:

```properties
spring.jpa.show-sql=true
```

For more readable/logged SQL, Hibernate-specific logging configuration is often preferable in real projects.

---

# 69. Don't Enable Excessive SQL Logging in Production

SQL logs can:

```text
Create huge log volume
Expose sensitive values
Increase overhead
Make debugging harder
```

Use controlled logging.

---

# 70. Hibernate Dialect

Hibernate needs to generate SQL appropriate for the target database.

Examples:

```text
MySQL
PostgreSQL
Oracle
SQL Server
```

Modern Hibernate/Spring Boot configurations can often infer the database dialect, so explicit configuration is not always necessary.

---

# 71. Hibernate and JDBC

Hibernate ultimately communicates with relational databases through JDBC.

Conceptually:

```text
Hibernate
 ↓
JDBC
 ↓
Database Driver
 ↓
Database
```

---

# 72. Hibernate Does Not Eliminate SQL Knowledge

This is an important interview point.

Even with Hibernate, developers need to understand:

```text
Joins
Indexes
Transactions
Execution plans
Locks
Query performance
Normalization
```

ORM does not remove database fundamentals.

---

# 73. ORM Impedance Mismatch

Object models and relational models are different.

Object model:

```text
Objects
Inheritance
References
Collections
```

Relational model:

```text
Tables
Rows
Columns
Foreign Keys
```

ORM attempts to bridge these differences.

---

# 74. Common Hibernate Problems

Interviewers may ask about:

```text
N+1 queries
LazyInitializationException
Unexpected eager loading
Large persistence contexts
Slow queries
Too many joins
Missing indexes
Incorrect cascade
Incorrect transaction boundaries
```

---

# 75. LazyInitializationException

A common scenario:

```text
Transaction ends
 ↓
Entity becomes detached
 ↓
Lazy association accessed
 ↓
LazyInitializationException
```

This is why transaction boundaries and fetch planning matter.

---

# 76. Bad Fix for LazyInitializationException

Don't blindly change everything to:

```text
EAGER
```

That may create:

```text
Large joins
More queries
Memory usage
Performance problems
```

Better:

```text
Define the data needed by the use case
 ↓
Fetch it explicitly
```

---

# 77. Open Session in View

Open Session in View (OSIV) keeps the persistence context available longer, commonly through the web request.

It can make lazy loading in the web layer appear convenient.

But it can also hide poor fetch planning and cause unexpected database access during response generation.

Use it intentionally rather than treating it as a fix for every lazy-loading issue.

---

# 78. Transaction Management

In Spring:

```java
@Transactional
```

is commonly used around service-layer business operations.

Example:

```java
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order);
}
```

---

# 79. Transaction Atomicity

Suppose:

```text
Create Order
+
Reserve Inventory
```

inside one local transaction.

If one operation fails:

```text
Rollback
```

Both changes can be rolled back if they are in the same transactional resource.

Across multiple services/databases, use distributed workflow patterns such as Saga rather than assuming one local transaction can cover everything.

---

# 80. Transaction Propagation

Spring supports propagation modes such as:

```text
REQUIRED
REQUIRES_NEW
SUPPORTS
MANDATORY
NOT_SUPPORTED
NEVER
NESTED
```

The most common is:

```text
REQUIRED
```

It joins an existing transaction or creates one if necessary.

---

# 81. Read-Only Transactions

Example:

```java
@Transactional(readOnly = true)
public User getUser(Long id) {
    ...
}
```

This communicates that the operation is intended for reads.

It can help with transaction/provider behavior, but it should not be treated as a universal performance switch.

---

# 82. Flush Modes

Flush timing can be influenced by the persistence provider and flush mode.

Conceptually:

```text
AUTO
COMMIT
```

The default behavior and exact semantics should be understood for the provider/version in use.

---

# 83. Optimistic Locking

Useful when concurrent updates are possible.

Example:

```java
@Version
private Long version;
```

Conceptually:

```text
User
version = 5
```

Two transactions read version 5.

First update:

```text
5 → 6
```

Second update still expects:

```text
version = 5
```

and detects a conflict.

---

# 84. Why Optimistic Locking?

It prevents silent lost updates without requiring a database lock for the entire transaction.

Good when:

```text
Conflicts are relatively uncommon
```

---

# 85. Pessimistic Locking

Pessimistic locking asks the database to lock rows during the transaction.

Conceptually:

```text
Transaction A
 ↓
Locks row
 ↓
Transaction B waits/fails
```

Useful when contention is high and strong serialization is needed.

---

# 86. Optimistic vs Pessimistic

```text
Optimistic
→ Assume conflicts are uncommon
→ Detect conflicts

Pessimistic
→ Assume conflicts may happen
→ Lock database rows
```

---

# 87. Hibernate Interview Question

### "What is JPA?"

Answer:

> "JPA is a Java specification that defines a standard API and ORM model for persistence. It is not itself an ORM implementation."

---

# 88. Hibernate Interview Question

### "What is Hibernate?"

Answer:

> "Hibernate is an ORM framework and a major implementation of JPA. It maps Java objects to relational data and handles persistence operations, dirty checking, fetching and other ORM functionality."

---

# 89. Hibernate Interview Question

### "JPA vs Hibernate?"

Answer:

> "JPA is the specification, while Hibernate is an implementation. I generally use JPA-standard APIs in application code and let Hibernate provide the underlying implementation."

---

# 90. Hibernate Interview Question

### "What is Persistence Context?"

Answer:

> "It is the set of entities currently managed by an EntityManager. Hibernate uses it for entity lifecycle management, first-level caching and dirty checking."

---

# 91. Hibernate Interview Question

### "What is dirty checking?"

Answer:

> "Hibernate tracks managed entities and detects changes to their state. During flush, it can generate the required SQL updates automatically."

---

# 92. Hibernate Interview Question

### "What is the first-level cache?"

Answer:

> "The persistence context acts as Hibernate's first-level cache. It is associated with an EntityManager or session and ensures repeated lookups of the same entity can reuse the managed instance."

---

# 93. Hibernate Interview Question

### "What is second-level cache?"

Answer:

> "It is an optional Hibernate cache shared across persistence contexts. It can reduce database reads for suitable data when configured with an appropriate cache provider."

---

# 94. Hibernate Interview Question

### "persist() vs merge()?"

Answer:

> "`persist()` makes a new entity managed. `merge()` copies the state of a detached or new entity into a managed instance and returns that managed instance."

---

# 95. Hibernate Interview Question

### "flush() vs commit()?"

Answer:

> "`flush()` synchronizes pending persistence changes with the database, while `commit()` completes the transaction. A flush can occur before commit, and the transaction can still roll back."

---

# 96. Hibernate Interview Question

### "Why does an entity need a no-argument constructor?"

Answer:

> "JPA requires an accessible no-argument constructor so the persistence provider can instantiate entities. It is common to make it protected or package-private."

---

# 97. Hibernate Interview Question

### "What is LazyInitializationException?"

Answer:

> "It typically occurs when Hibernate tries to initialize a lazy association after the entity is no longer attached to an active persistence context. The better solution is usually to define the required fetch plan and transaction boundary rather than simply making everything eager."

---

# 98. Hibernate Interview Question

### "What is N+1?"

Answer:

> "N+1 occurs when one query loads a collection of entities and then additional queries are executed for each entity's related data. It can cause serious performance problems."

---

# 99. Hibernate Interview Question

### "How do you solve N+1?"

Answer:

> "Depending on the use case, I can use JOIN FETCH, EntityGraph, batch fetching or DTO projections. I would choose based on the data actually required rather than making every relationship eager."

---

# 100. Hibernate Interview Question

### "What is @Version?"

Answer:

> "`@Version` enables optimistic locking. Hibernate uses the version value to detect whether another transaction changed the entity before an update."

---

# 101. Interview Scenario

### "The API became slow after loading 100 orders."

Think:

```text
Check SQL logs
 ↓
Count queries
 ↓
Look for N+1
 ↓
Check joins
 ↓
Check indexes
 ↓
Inspect execution plan
 ↓
Design an appropriate fetch strategy
```

---

# 102. Interview Scenario

### "You update an entity but don't call save(). Will it update?"

Answer:

> "If the entity is managed inside an active transaction, Hibernate's dirty checking can detect the change and flush an UPDATE automatically. An explicit save isn't necessarily required for that managed entity."

---

# 103. Interview Scenario

### "You call save() but don't see INSERT immediately."

Answer:

> "Persistence operations are synchronized with the database during flush. Depending on the ID generation strategy and flush behavior, SQL may execute before transaction commit, but `save()` itself should not be treated as a guarantee that the database transaction has already committed."

---

# 104. Interview Scenario

### "Two users update the same product."

Solution:

```text
@Version
```

or appropriate pessimistic locking depending on the business requirements and contention.

---

# 105. Interview Scenario

### "Why is Hibernate generating many SQL queries?"

Investigate:

```text
N+1
Lazy loading
Eager loading
Cascade behavior
Fetch joins
Entity graphs
Application loops
```

Don't assume Hibernate itself is broken.

---

# 106. Best Practices

```text
Use JPA interfaces in application code where practical
Keep entities focused on persistence/domain concerns
Define explicit fetch plans
Avoid unnecessary EAGER associations
Use transactions at meaningful service boundaries
Understand generated SQL
Index frequently queried columns
Watch for N+1
Use pagination for large datasets
Use projections for read-heavy use cases
Avoid huge persistence contexts
Use optimistic locking where appropriate
Keep secrets out of entity data/logs
```

---

# 107. Common Interview Traps

```text
JPA = Hibernate
```

Wrong.

Correct:

```text
JPA = specification
Hibernate = implementation
```

---

# 108. Common Interview Trap

```text
save() always means INSERT
```

Wrong.

It can result in:

```text
INSERT
or
UPDATE
```

depending on entity state/newness detection.

---

# 109. Common Interview Trap

```text
flush() = commit()
```

Wrong.

```text
flush
→ synchronize

commit
→ finish transaction
```

---

# 110. Common Interview Trap

```text
Lazy loading = bad
```

Wrong.

Lazy loading can reduce unnecessary database work, but it must be used with a well-defined fetch plan and transaction boundary.

---

# 111. Common Interview Trap

```text
Fix N+1 by making everything EAGER
```

Usually a poor solution.

Prefer:

```text
JOIN FETCH
EntityGraph
DTO projection
Batch fetching
```

as appropriate.

---

# 112. Common Interview Trap

```text
Hibernate means I don't need SQL knowledge.
```

Wrong.

A strong Java backend developer should understand:

```text
SQL
Indexes
Joins
Transactions
Locks
Execution plans
```

---

# 113. Hibernate Mental Model

Remember:

```text
Java Entity
     ↓
Persistence Context
     ↓
Hibernate
     ↓
JDBC
     ↓
Database
```

And:

```text
Entity becomes managed
     ↓
Hibernate tracks it
     ↓
Entity changes
     ↓
Dirty checking
     ↓
Flush
     ↓
SQL
     ↓
Commit
```

---

# 114. Final Interview Answer

If asked:

> "Explain how Hibernate works in a Spring Boot application."

Use:

> "In a Spring Boot application, Spring Data JPA provides repository abstractions over the JPA API, and Hibernate commonly acts as the JPA implementation. Entities are managed inside a persistence context associated with an EntityManager. Hibernate tracks managed entities, performs dirty checking and translates persistence operations into SQL through JDBC. Transactions define when those changes are flushed and committed. For relationships and performance, I pay particular attention to lazy loading, fetch plans, N+1 queries, indexing and transaction boundaries."

---

# 115. Revision Checklist

```text
□ ORM
□ JPA
□ Hibernate
□ JPA vs Hibernate
□ Spring Data JPA
□ Entity
□ @Entity
□ @Table
□ @Id
□ @GeneratedValue
□ GenerationType
□ @Column
□ Entity lifecycle
□ Transient
□ Managed
□ Detached
□ Removed
□ Persistence Context
□ EntityManager
□ persist()
□ find()
□ merge()
□ remove()
□ detach()
□ clear()
□ flush()
□ First-level cache
□ Second-level cache
□ Dirty checking
□ Spring Data repositories
□ Derived queries
□ Lazy loading
□ Eager loading
□ N+1
□ Transactions
□ Flush vs commit
□ Optimistic locking
□ Pessimistic locking
□ @Version
□ LazyInitializationException
□ Hibernate SQL
□ Hibernate/JDBC
□ ORM impedance mismatch
□ Interview scenarios
```

---

# 116. What Comes Next

This first file gives the foundation.

Next files should go deeper into:

```text
File 02 → Entity Relationships & Mapping
File 03 → Fetching, Lazy/Eager & N+1
File 04 → JPQL, Native SQL & Projections
File 05 → Transactions & Locking
File 06 → Hibernate Performance & Optimization
File 07 → Caching & Persistence Context
File 08 → Advanced Hibernate & Interview Scenarios
```

The key interview lesson is:

> **Hibernate is not magic. It is an ORM layer that manages entity state and translates object operations into database operations. To use it well, you need to understand both the Java object model and what SQL/database work Hibernate is actually performing.**
