# SQL + Spring Boot / JPA — Integration Interview Questions

This file connects SQL knowledge with the way Java backend applications actually use databases.

For Java backend interviews, know how:

```text
REST API
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
JPA / Hibernate
   ↓
JDBC
   ↓
Connection Pool
   ↓
Database
```

works.

---

# 1. What Is JPA?

JPA stands for:

```text
Java Persistence API
```

It is a Java specification for object-relational mapping.

JPA defines APIs and concepts such as:

```text
@Entity
@Id
@OneToMany
@ManyToOne
@OneToOne
@ManyToMany
EntityManager
JPQL
```

Hibernate is a popular JPA implementation.

---

# 2. What Is Hibernate?

Hibernate is an ORM framework and a common implementation of JPA.

Conceptually:

```text
Java Entity
   ↓
Hibernate
   ↓
SQL
   ↓
Database
```

For example:

```java
@Entity
public class Product {
    @Id
    private Long id;

    private String name;
    private BigDecimal price;
}
```

Hibernate maps this object to a database table.

---

# 3. JPA vs Hibernate

JPA:

```text
Specification
```

Hibernate:

```text
Implementation / ORM framework
```

Interview answer:

> JPA defines the standard persistence API and ORM concepts, while Hibernate is a popular implementation that provides those capabilities.

---

# 4. What Is Spring Data JPA?

Spring Data JPA provides a repository abstraction on top of JPA.

Example:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {
}
```

You automatically get methods such as:

```text
save()
findById()
findAll()
deleteById()
existsById()
```

This reduces boilerplate repository code.

---

# 5. Repository Layer

Typical Spring Boot architecture:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

The repository is responsible for persistence/data access.

Business rules generally belong in the service layer rather than the repository.

---

# 6. Entity

Example:

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private BigDecimal price;
}
```

This maps:

```text
Product
   ↓
products table
```

---

# 7. @Id

`@Id` identifies the entity's primary key.

Example:

```java
@Id
private Long id;
```

The corresponding database column should represent the table's primary key.

---

# 8. @GeneratedValue

Example:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

This tells the persistence provider that the database-generated identifier strategy is being used.

Exact behavior depends on the database and generation strategy.

---

# 9. Entity Relationships

JPA supports:

```text
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

These should correspond to actual database relationships.

---

# 10. @ManyToOne

Example:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id")
private User user;
```

Database:

```text
orders.user_id
        ↓
users.id
```

Many orders belong to one user.

---

# 11. @OneToMany

Example:

```java
@OneToMany(mappedBy = "user")
private List<OrderEntity> orders;
```

This represents:

```text
One User
   ↓
Many Orders
```

The foreign key is still stored on the many side:

```text
orders.user_id
```

---

# 12. Owning Side

In a bidirectional relationship, the owning side is the side that controls the relationship mapping, typically through the foreign-key mapping.

Example:

```java
@ManyToOne
@JoinColumn(name = "user_id")
private User user;
```

This side owns the relationship.

The inverse side can use:

```java
@OneToMany(mappedBy = "user")
```

---

# 13. mappedBy

Example:

```java
@OneToMany(mappedBy = "user")
private List<OrderEntity> orders;
```

`mappedBy` tells JPA:

```text
The relationship is mapped by the "user" field
on the OrderEntity side.
```

It avoids creating an unnecessary separate relationship mapping.

---

# 14. LAZY vs EAGER

LAZY:

```text
Load relationship when accessed
```

EAGER:

```text
Load relationship immediately
```

For large collections:

```java
@OneToMany(fetch = FetchType.LAZY)
```

is generally safer than blindly loading everything.

But LAZY loading still needs careful query design.

---

# 15. N+1 Query Problem

Example:

```text
Query 1:
SELECT users

Then:

Query 2:
SELECT orders for user 1

Query 3:
SELECT orders for user 2

Query 4:
SELECT orders for user 3
```

For 100 users:

```text
101 queries
```

This is the N+1 problem.

---

# 16. Solving N+1 with JOIN FETCH

Example:

```java
@Query("""
    SELECT DISTINCT u
    FROM User u
    JOIN FETCH u.orders
""")
List<User> findUsersWithOrders();
```

This can fetch users and their orders together.

But fetching large collections can produce a large result set.

Use it deliberately.

---

# 17. EntityGraph

Another option:

```java
@EntityGraph(attributePaths = "orders")
List<User> findAll();
```

Entity graphs let you control fetching for specific queries.

---

# 18. DTO Projection

Instead of loading the entire entity graph:

```java
@Query("""
    SELECT new com.example.dto.ProductSummaryDTO(
        p.id,
        p.name,
        p.price
    )
    FROM Product p
""")
List<ProductSummaryDTO> findProductSummaries();
```

This fetches only the fields needed.

---

# 19. Why DTO Projection?

Benefits can include:

```text
Less data
Less memory
Less entity overhead
Smaller result sets
Clear API-specific data
```

Especially useful for:

```text
List APIs
Reporting
Dashboards
Read-heavy endpoints
```

---

# 20. Entity vs DTO

Entity:

```text
Persistence model
```

DTO:

```text
Data transfer model
```

Avoid exposing entities directly from every REST endpoint.

A common flow is:

```text
Entity
 ↓
Service
 ↓
DTO
 ↓
Controller
 ↓
JSON
```

---

# 21. JPQL

JPQL stands for:

```text
Java Persistence Query Language
```

It queries entities rather than database tables directly.

Example:

```java
@Query("""
    SELECT p
    FROM Product p
    WHERE p.price > :price
""")
List<Product> findExpensiveProducts(
    @Param("price") BigDecimal price
);
```

Notice:

```text
Product
p.price
```

rather than:

```text
products
price_column
```

---

# 22. Native SQL Query

Spring Data JPA can execute native SQL.

Example:

```java
@Query(
    value = """
        SELECT *
        FROM products
        WHERE price > :price
    """,
    nativeQuery = true
)
List<Product> findExpensiveProducts(
    @Param("price") BigDecimal price
);
```

Use native SQL when database-specific features or complex SQL make JPQL unsuitable.

---

# 23. JPQL vs Native SQL

JPQL:

```text
Entity-oriented
Database-independent to a greater degree
Works with JPA mappings
```

Native SQL:

```text
Database-specific
Full SQL capabilities
Useful for complex/optimized queries
```

---

# 24. Derived Query Methods

Spring Data can generate queries from method names.

Example:

```java
List<Product> findByCategoryId(Long categoryId);
```

Another:

```java
List<Product> findByPriceGreaterThan(
    BigDecimal price
);
```

Another:

```java
Optional<User> findByEmail(String email);
```

---

# 25. Derived Query Advantages

Advantages:

```text
Simple
Readable
Low boilerplate
Fast to write
```

But extremely long method names can become difficult to maintain.

For complex queries, use:

```text
@Query
Specifications
Criteria API
QueryDSL
native SQL
```

depending on project requirements.

---

# 26. @Query

Example:

```java
@Query("""
    SELECT o
    FROM OrderEntity o
    WHERE o.user.id = :userId
      AND o.status = :status
""")
List<OrderEntity> findOrders(
    @Param("userId") Long userId,
    @Param("status") OrderStatus status
);
```

This is JPQL.

---

# 27. Parameter Binding

Use named parameters:

```java
WHERE o.user.id = :userId
```

and:

```java
@Param("userId")
```

This is safer and clearer than constructing query strings manually.

---

# 28. Never Concatenate User Input

Bad:

```java
String query =
    "SELECT u FROM User u WHERE u.email = '" + email + "'";
```

Potential SQL injection risk.

Better:

```java
@Query("""
    SELECT u
    FROM User u
    WHERE u.email = :email
""")
Optional<User> findByEmail(
    @Param("email") String email
);
```

---

# 29. @Transactional

Spring's:

```java
@Transactional
```

defines transaction behavior around a method/class.

Example:

```java
@Transactional
public void placeOrder(...) {

    // create order
    // create order items
    // update inventory
}
```

The operations are treated as one transaction according to the configured transaction manager and database behavior.

---

# 30. Why Use @Transactional?

Suppose placing an order requires:

```text
1. Create order
2. Insert order items
3. Reduce stock
```

If step 3 fails:

```text
Order should not remain partially created
```

The transaction can roll back the database work.

---

# 31. Transaction Boundary

A good transaction boundary usually surrounds one business operation.

Example:

```text
placeOrder()
```

rather than:

```text
controller method calls random repository methods
```

The service layer is commonly a good place for transaction boundaries.

---

# 32. Read-Only Transaction

Spring supports:

```java
@Transactional(readOnly = true)
```

for read operations.

Example:

```java
@Transactional(readOnly = true)
public Product getProduct(Long id) {
    return repository.findById(id)
        .orElseThrow();
}
```

It communicates intent and may allow optimizations depending on the stack.

It is not a universal guarantee that no write can happen.

---

# 33. Transaction Propagation

Common propagation modes include:

```text
REQUIRED
REQUIRES_NEW
MANDATORY
SUPPORTS
NOT_SUPPORTED
NEVER
NESTED
```

Most important:

```text
REQUIRED
REQUIRES_NEW
```

---

# 34. REQUIRED

Default Spring propagation is generally:

```text
REQUIRED
```

Meaning:

```text
Use existing transaction
OR
create one if none exists
```

Example:

```text
Service A
  ↓
@Transactional
Service B
  ↓
@Transactional
```

B normally participates in A's transaction.

---

# 35. REQUIRES_NEW

`REQUIRES_NEW` suspends the existing transaction and starts a separate transaction when transaction management supports it.

Example:

```text
Main transaction
      ↓
   suspend
      ↓
Audit transaction
      ↓
   commit
      ↓
resume main transaction
```

This can be useful when an independent operation must commit separately, but it should be used carefully.

---

# 36. Isolation Levels

Common isolation levels:

```text
READ_UNCOMMITTED
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```

They control visibility/concurrency trade-offs.

Exact behavior depends on the database.

---

# 37. Dirty Read

A dirty read happens when one transaction reads data written by another transaction that has not committed.

Conceptually:

```text
T1 writes 100
T2 reads 100
T1 rolls back
```

T2 saw data that never committed.

---

# 38. Non-Repeatable Read

Within one transaction:

```text
T1 reads row → 100

T2 updates row → 200
T2 commits

T1 reads again → 200
```

The same row returned different values within T1.

---

# 39. Phantom Read

A query returns a different set of rows when repeated because another transaction inserted/deleted matching rows.

Example:

```text
T1:
SELECT orders WHERE amount > 1000

T2:
INSERT order amount = 2000
COMMIT

T1:
SELECT again
```

The second query may contain an additional row depending on isolation semantics.

---

# 40. Optimistic Locking

JPA supports optimistic locking using:

```java
@Version
private Long version;
```

Conceptually:

```text
Read version = 5
      ↓
Update WHERE id = 10 AND version = 5
      ↓
If successful → version becomes 6
```

If another transaction already changed the row:

```text
version is no longer 5
```

and the update can fail with an optimistic-lock exception.

---

# 41. Why Optimistic Locking?

Useful when:

```text
Conflicts are relatively rare
Reads are frequent
You don't want to hold database locks for long periods
```

Example:

```text
Product inventory
Order state
User profile
```

depending on concurrency requirements.

---

# 42. Pessimistic Locking

Pessimistic locking acquires database locks to prevent conflicting operations.

Example JPA:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Product> findById(Long id);
```

This can be useful when:

```text
Conflicts are frequent
Correctness requires serialized access
```

But locks reduce concurrency and can contribute to deadlocks.

---

# 43. Optimistic vs Pessimistic

Optimistic:

```text
Assume conflicts are rare
Detect conflict during update
```

Pessimistic:

```text
Lock early
Prevent conflicting access
```

Choose based on:

```text
Contention
Transaction length
Business requirements
Database behavior
```

---

# 44. Flush

Hibernate may synchronize pending entity changes with the database through a flush.

Conceptually:

```text
Java entity changed
      ↓
Persistence context
      ↓
flush
      ↓
SQL sent to database
```

Flush does not necessarily mean commit.

---

# 45. Flush vs Commit

Important distinction:

```text
Flush
→ synchronize persistence context with database

Commit
→ finalize transaction
```

A flushed SQL statement can still be rolled back before the transaction commits.

---

# 46. Persistence Context

The JPA persistence context manages entity instances.

It provides concepts such as:

```text
First-level cache
Dirty checking
Identity management
Entity lifecycle
```

---

# 47. First-Level Cache

Within a persistence context:

```text
find(User.class, 10)
```

followed by another lookup for the same entity may return the same managed instance rather than issuing another SQL query.

This is the JPA first-level cache.

It is associated with the persistence context.

---

# 48. Dirty Checking

Suppose:

```java
@Transactional
public void updateProduct(Long id) {

    Product product =
        repository.findById(id).orElseThrow();

    product.setPrice(newPrice);
}
```

You may not need:

```java
repository.save(product);
```

for a managed entity in the transaction.

Hibernate can detect the change during flush and generate:

```sql
UPDATE products
SET price = ?
WHERE id = ?;
```

Exact behavior depends on entity state and persistence context.

---

# 49. Entity Lifecycle

Common states:

```text
Transient
Managed
Detached
Removed
```

Transient:

```text
new object
not managed
```

Managed:

```text
associated with persistence context
```

Detached:

```text
was managed but is no longer attached
```

Removed:

```text
scheduled for deletion
```

---

# 50. save() in Spring Data JPA

Example:

```java
productRepository.save(product);
```

Depending on whether the entity is considered new, Spring Data/JPA may persist or merge it.

Don't think of `save()` as simply:

```text
always INSERT
```

or:

```text
always UPDATE
```

---

# 51. findById()

Example:

```java
Optional<Product> product =
    productRepository.findById(id);
```

This generally results in a database lookup when the entity isn't already available in the persistence context.

---

# 52. LazyInitializationException

A common problem:

```text
Transaction ends
      ↓
Entity detached
      ↓
Access LAZY relationship
      ↓
LazyInitializationException
```

Example:

```java
user.getOrders().size();
```

outside the appropriate persistence context.

---

# 53. How to Avoid LazyInitializationException

Prefer explicit data-fetching strategies:

```text
JOIN FETCH
@EntityGraph
DTO projection
Service-level transaction
```

Avoid blindly making everything:

```java
FetchType.EAGER
```

because that can create performance problems.

---

# 54. Open Session in View

Open Session in View can keep the persistence context available through web request processing.

It can make lazy loading in controllers/views possible.

But it can also hide database access and cause unexpected queries.

A deliberate service-layer fetch strategy is often easier to reason about.

---

# 55. Cascade Types

JPA supports cascade operations such as:

```text
PERSIST
MERGE
REMOVE
REFRESH
DETACH
ALL
```

Example:

```java
@OneToMany(
    mappedBy = "order",
    cascade = CascadeType.ALL
)
private List<OrderItem> items;
```

Cascade should represent ownership/lifecycle semantics.

Don't use `CascadeType.ALL` automatically.

---

# 56. orphanRemoval

Example:

```java
@OneToMany(
    mappedBy = "order",
    orphanRemoval = true
)
private List<OrderItem> items;
```

When a child is removed from the relationship, JPA may schedule it for deletion depending on the mapping and operation.

Use only when the child lifecycle is truly owned by the parent.

---

# 57. Many-to-Many Caution

A direct:

```java
@ManyToMany
```

mapping can become awkward when the relationship has attributes.

For example:

```text
Order
Product
```

needs:

```text
quantity
priceAtPurchase
discount
```

Prefer an explicit entity:

```text
OrderItem
```

rather than a plain many-to-many mapping.

---

# 58. Native SQL for Complex Queries

Use native SQL when:

```text
Database-specific features are required
Complex reporting query
Vendor-specific functions
Performance requires a carefully tuned query
```

But understand the portability trade-off.

---

# 59. SQL Injection with Native Queries

Parameterized:

```java
@Query(
    value = """
        SELECT *
        FROM users
        WHERE email = :email
    """,
    nativeQuery = true
)
```

is safer than concatenating:

```java
"... WHERE email = '" + email + "'"
```

Always parameterize untrusted input.

---

# 60. Pagination with Spring Data

Example:

```java
Pageable pageable =
    PageRequest.of(
        page,
        size,
        Sort.by("createdAt").descending()
    );

Page<Product> result =
    repository.findAll(pageable);
```

For large datasets, consider cursor/keyset pagination rather than deep OFFSET pagination.

---

# 61. Slice vs Page

`Page`:

```text
Content
Total elements
Total pages
Page metadata
```

It commonly requires a count query.

`Slice`:

```text
Content
Whether another slice exists
```

It can avoid the total-count requirement in appropriate cases.

---

# 62. Specification

Spring Data JPA Specifications can build dynamic predicates.

Useful when filters are optional:

```text
category
minPrice
maxPrice
status
createdAfter
```

Instead of creating many repository methods:

```text
findByCategoryAndPriceAndStatus...
```

a specification can compose conditions dynamically.

---

# 63. Example Dynamic Search

Requirements:

```text
category optional
price optional
status optional
```

Conceptually:

```text
Specification<Product>
      ↓
category predicate
+
price predicate
+
status predicate
```

This can keep complex filtering more maintainable.

---

# 64. QueryDSL

QueryDSL is another option for type-safe dynamic queries.

Conceptually:

```text
Java code
 ↓
Type-safe query construction
 ↓
SQL
```

It can be useful when dynamic query complexity becomes high.

---

# 65. JDBC vs JPA

JPA/Hibernate:

```text
Higher abstraction
Entity mapping
Automatic persistence behavior
Relationships
Dirty checking
```

JDBC:

```text
Lower-level
Explicit SQL
More control
Less ORM overhead
```

Choose based on use case.

---

# 66. JdbcTemplate

Spring's `JdbcTemplate` simplifies JDBC access.

Example:

```java
String sql = """
    SELECT id, name, price
    FROM products
    WHERE category_id = ?
""";

List<Product> products =
    jdbcTemplate.query(
        sql,
        productRowMapper,
        categoryId
    );
```

---

# 67. When JdbcTemplate Can Be Better

Useful for:

```text
Complex SQL
Reporting
Bulk operations
Fine-grained SQL control
Queries where ORM mapping is unnecessary
```

---

# 68. JPA Performance Pitfall

Don't assume:

```text
One repository method
=
One SQL query
```

ORMs can generate multiple queries because of:

```text
Lazy loading
Collections
Cascades
Entity relationships
Dirty checking
Fetch configuration
```

Always inspect generated SQL when debugging performance.

---

# 69. SQL Logging

During development, SQL logging can help identify:

```text
Unexpected queries
N+1
Wrong joins
Slow query patterns
```

Avoid exposing sensitive data in production logs.

---

# 70. Database Indexes and JPA

JPA can define indexes through table metadata in some setups.

Example:

```java
@Table(
    indexes = {
        @Index(
            name = "idx_user_email",
            columnList = "email"
        )
    }
)
```

But production database schema should ideally be managed through migration tooling such as:

```text
Flyway
Liquibase
```

rather than relying solely on ORM auto-generation.

---

# 71. ddl-auto

Spring Boot commonly supports:

```properties
spring.jpa.hibernate.ddl-auto=
```

Common values:

```text
none
validate
update
create
create-drop
```

Be careful with:

```text
create
create-drop
```

because they can destroy schema/data.

For production, controlled migrations are generally safer.

---

# 72. validate

`validate` can check whether the entity mappings are compatible with the existing schema.

It does not create the schema.

This can be useful when migrations are managed separately.

---

# 73. update

`update` asks Hibernate to attempt schema updates.

It can be convenient during development.

It is generally not the preferred production schema-management strategy for serious systems because schema evolution should be controlled and versioned.

---

# 74. Flyway + JPA

A common production setup:

```text
Flyway
 ↓
Database schema
 ↓
Hibernate/JPA
 ↓
Application
```

Flyway manages:

```text
Tables
Indexes
Constraints
Views
Other database objects
```

Hibernate maps Java entities to the existing schema.

---

# 75. Transaction + Multiple Repositories

Example:

```java
@Transactional
public void placeOrder(...) {

    orderRepository.save(order);

    orderItemRepository.saveAll(items);

    productRepository.decreaseStock(...);
}
```

All participate in the same transaction when they use the same transactional resource/context.

---

# 76. Transaction Rollback

By default, Spring's rollback behavior is primarily based on unchecked exceptions.

For checked exceptions, configure rollback behavior when required.

Example:

```java
@Transactional(
    rollbackFor = Exception.class
)
```

Use explicit rollback rules when the business operation requires them.

---

# 77. Self-Invocation Problem

A common Spring interview question:

```java
public void methodA() {
    methodB();
}

@Transactional
public void methodB() {
}
```

If `methodB()` is called through the same object instance, the call may bypass Spring's proxy and therefore not apply the transactional interceptor as expected.

This is one reason transactional boundaries should be designed carefully.

---

# 78. Transactional on Private Methods

Spring's proxy-based transaction management generally does not apply `@Transactional` to private methods in the way developers often expect.

Prefer transactional boundaries on appropriate externally invoked service methods.

---

# 79. Transaction and Async

Be careful combining:

```text
@Transactional
```

with:

```text
@Async
```

The asynchronous method executes in a different thread, so it does not automatically participate in the caller's thread-bound transaction.

Design transaction boundaries explicitly.

---

# 80. Transaction and REST Controller

You can technically put:

```java
@Transactional
```

on a controller method.

But a common design is:

```text
Controller
 ↓
Service @Transactional
 ↓
Repository
```

The service layer is usually a clearer place for business transaction boundaries.

---

# 81. Read-Only API Flow

Example:

```text
GET /products
     ↓
Controller
     ↓
Service @Transactional(readOnly = true)
     ↓
Repository
     ↓
Database
```

The exact need for a transaction depends on the operation and persistence behavior.

---

# 82. Write API Flow

Example:

```text
POST /orders
     ↓
Controller
     ↓
OrderService @Transactional
     ↓
OrderRepository
OrderItemRepository
InventoryRepository
     ↓
Database
```

All related writes should satisfy the business transaction requirement.

---

# 83. Optimistic Locking Example

Entity:

```java
@Version
private Long version;
```

Two requests:

```text
Request A reads version 5
Request B reads version 5
```

A updates:

```text
version → 6
```

B tries using:

```text
version = 5
```

and may fail because the row has changed.

---

# 84. Inventory Concurrency

Suppose stock is:

```text
10
```

Two customers request:

```text
7
7
```

A naive read-then-update can oversell.

Possible approaches:

```text
Atomic conditional update
Pessimistic lock
Optimistic locking
Database transaction
Queueing
```

Choose based on throughput and consistency requirements.

---

# 85. Atomic Inventory Update

Example:

```sql
UPDATE products
SET stock = stock - :quantity
WHERE id = :productId
  AND stock >= :quantity;
```

Then:

```text
affected rows = 1
→ success

affected rows = 0
→ insufficient stock / condition failed
```

This can avoid a separate vulnerable read-check-update sequence.

---

# 86. Repository Method for Atomic Update

Conceptually:

```java
@Modifying
@Query("""
    UPDATE Product p
    SET p.stock = p.stock - :quantity
    WHERE p.id = :productId
      AND p.stock >= :quantity
""")
int decreaseStock(
    @Param("productId") Long productId,
    @Param("quantity") int quantity
);
```

Then check:

```java
int updated = repository.decreaseStock(...);

if (updated == 0) {
    throw new InsufficientStockException();
}
```

The transaction boundary should cover the broader order operation as required.

---

# 87. Batch Fetching

If you need many related entities, batch fetching can reduce query count.

Hibernate supports configuration/options for batching associations.

But don't use it blindly.

Measure:

```text
Query count
Result size
Memory
Latency
```

---

# 88. Entity Graph vs JOIN FETCH

Both can control fetching.

`JOIN FETCH`:

```text
Explicit query-level fetch
```

`EntityGraph`:

```text
Declarative fetch plan
```

Choose based on readability, reuse and query requirements.

---

# 89. Bulk Operations and Persistence Context

Bulk JPQL/SQL updates operate directly on database rows.

They do not necessarily synchronize already-managed entity objects in the persistence context.

Therefore:

```text
Bulk update
 ↓
Persistence context may be stale
```

Clear/refresh appropriately when necessary.

---

# 90. First-Level vs Second-Level Cache

First-level cache:

```text
Built into JPA persistence context
Scoped to persistence context
```

Second-level cache:

```text
Shared across persistence contexts
Optional
Requires configuration/provider support
```

Don't assume a second-level cache exists by default.

---

# 91. Redis vs Hibernate Cache

Redis:

```text
Application-level distributed cache
```

Hibernate second-level cache:

```text
ORM-level entity/query caching
```

They solve related but different problems.

---

# 92. Database Connection Pool and Transactions

A transaction generally holds database resources while it is active.

Long transaction:

```text
Transaction starts
 ↓
Connection occupied
 ↓
Slow processing
 ↓
Commit
```

This can reduce pool availability.

Keep transactions focused.

---

# 93. Common JPA Production Mistake

Bad:

```text
Fetch huge entity graph
↓
Serialize entity directly
↓
Lazy loading triggers many queries
↓
Huge JSON
```

Better:

```text
Query required data
↓
DTO projection
↓
Return controlled response
```

---

# 94. Common JPA Interview Trap

Question:

> "If I use JPA, do I no longer need to know SQL?"

Correct answer:

> No. JPA abstracts many database operations, but understanding SQL is still important for joins, indexes, transactions, query performance, debugging generated SQL and database-specific behavior.

---

# 95. Common JPA Interview Trap

Question:

> "Does `save()` immediately execute INSERT?"

Answer:

> Not necessarily. JPA/Hibernate works with a persistence context, and SQL execution can occur during flush/transaction synchronization. The exact timing depends on the operation and flush mode.

---

# 96. Common JPA Interview Trap

Question:

> "Does `@Transactional` mean the method can never fail halfway?"

Answer:

> No. It means database operations participating in that transaction can be committed or rolled back as a unit according to transaction rules. External systems such as email or HTTP calls are not automatically rolled back.

---

# 97. External API Inside Transaction

Avoid unnecessarily long transactions like:

```text
BEGIN
 ↓
Update database
 ↓
Call external payment API
 ↓
Wait 5 seconds
 ↓
Commit
```

This can hold database resources during network latency.

Consider:

```text
Transaction
+
Outbox pattern
+
Asynchronous processing
```

for more complex workflows.

---

# 98. Outbox Pattern

For reliable database + event publishing:

```text
BEGIN
 ↓
Update business data
 ↓
Insert outbox event
 ↓
COMMIT
```

Then:

```text
Outbox processor
 ↓
Publish event
 ↓
Mark event processed
```

This avoids relying on a single transaction spanning unrelated systems.

---

# 99. SQL + JPA Performance Workflow

When an API is slow:

```text
1. Measure endpoint
2. Identify database calls
3. Check query count
4. Check N+1
5. Inspect SQL
6. Run EXPLAIN
7. Check indexes
8. Check transaction duration
9. Check connection pool
10. Optimize
11. Measure again
```

---

# 100. Interview: JPA vs Hibernate

> JPA is a specification for persistence and ORM concepts, while Hibernate is a popular implementation of that specification.

---

# 101. Interview: What Is Spring Data JPA?

> Spring Data JPA provides repository abstractions on top of JPA, reducing boilerplate for common CRUD and query operations.

---

# 102. Interview: What Is Lazy Loading?

> Lazy loading means a relationship is loaded when it is accessed rather than immediately with the parent entity. It can reduce unnecessary data retrieval but requires careful transaction and query design.

---

# 103. Interview: What Is N+1 in Hibernate?

> N+1 occurs when one query loads the parent records and additional queries are executed for each parent's relationship. I address it with appropriate fetch joins, entity graphs, DTO projections or batching depending on the use case.

---

# 104. Interview: What Is Dirty Checking?

> Hibernate tracks managed entity changes in the persistence context and can automatically generate SQL updates during flush without requiring an explicit update statement for every field change.

---

# 105. Interview: Flush vs Commit

> Flush synchronizes pending persistence-context changes with the database, while commit finalizes the transaction. A flushed change can still be rolled back before commit.

---

# 106. Interview: What Is @Transactional?

> `@Transactional` defines a transaction boundary around a method or class so participating database operations can be committed or rolled back according to the transaction configuration.

---

# 107. Interview: Where Would You Put @Transactional?

> I usually place business transaction boundaries in the service layer because a business operation may involve multiple repositories and should be treated as one unit.

---

# 108. Interview: REQUIRED vs REQUIRES_NEW

> `REQUIRED` joins an existing transaction or creates one if necessary. `REQUIRES_NEW` suspends the existing transaction and starts a separate transaction when supported by the transaction manager.

---

# 109. Interview: Optimistic vs Pessimistic Locking

> Optimistic locking detects conflicting updates, commonly using a version field. Pessimistic locking acquires database locks to prevent conflicting access. I choose based on contention and transaction requirements.

---

# 110. Interview: Why Use DTO Projections?

> DTO projections let us retrieve only the fields required by a use case, reducing unnecessary data and entity-loading overhead. They are especially useful for list, reporting and read-heavy APIs.

---

# 111. Interview: JPQL vs Native SQL

> JPQL operates on entities and is more portable across databases, while native SQL gives full database-specific SQL capabilities. I use native SQL when the query complexity or database-specific features justify it.

---

# 112. Interview: How Do You Prevent SQL Injection in Spring Boot?

> I use parameterized queries, JPA named parameters, prepared statements and safe repository APIs. I never concatenate untrusted input directly into SQL.

---

# 113. Interview: How Do You Fix LazyInitializationException?

> I don't solve it by making everything eager. I prefer fetching the required relationship explicitly using a fetch join, entity graph or DTO projection within an appropriate service-layer transaction.

---

# 114. Interview: How Do You Optimize a Slow JPA Query?

> I first inspect the generated SQL and query count, then run `EXPLAIN`, check indexes and joins, look for N+1 queries and excessive data fetching, and finally measure the effect of the optimization.

---

# 115. Interview: How Would You Design Order Placement?

> I'd put the order operation inside a service-level transaction. I'd create the order and items, perform a concurrency-safe inventory update, and commit them together. If any required database operation fails, the transaction should roll back.

---

# 116. Interview: How Do You Prevent Overselling?

> I avoid a simple read-check-update race. I can use an atomic conditional update such as `stock >= quantity`, optimistic locking, or a pessimistic lock depending on contention and business requirements, all within the appropriate transaction boundary.

---

# 117. Interview: Does JPA Remove the Need for SQL Knowledge?

> No. JPA reduces boilerplate but doesn't remove the need to understand SQL. SQL knowledge is essential for joins, indexes, query plans, transactions, debugging generated SQL and performance tuning.

---

# 118. Interview: How Do You Manage Database Schema Changes?

> In production I prefer versioned migrations using Flyway or Liquibase. JPA/Hibernate should generally map to and validate against the controlled schema rather than being responsible for unpredictable production schema changes.

---

# 119. Interview: What Is the Persistence Context?

> The persistence context is the set of managed entity instances associated with an EntityManager. It provides identity management, dirty checking and first-level caching within its scope.

---

# 120. Interview: What Is First-Level Cache?

> The first-level cache is the persistence-context cache. Within the same persistence context, repeated access to the same entity can avoid another database lookup.

---

# 121. Interview: Why Not Use EAGER Everywhere?

> Eager loading can cause unnecessary joins, large result sets, excessive memory use and unexpected database queries. I prefer fetching relationships explicitly based on each use case.

---

# 122. Interview: What Is the Repository Layer Responsible For?

> The repository handles persistence and data-access operations. Business rules and orchestration generally belong in the service layer, while the controller handles HTTP concerns.

---

# 123. Interview: How Do You Handle Large Result Sets?

> I use pagination, projections, streaming or batch processing depending on the use case. I avoid loading millions of entities into memory and avoid returning unbounded API responses.

---

# 124. Interview: How Do You Optimize a List API?

> I first define the exact fields required, use a DTO projection where appropriate, add filtering and stable pagination, check for N+1 queries, inspect indexes and execution plans, and enforce a maximum page size.

---

# 125. Final Integration Mental Model

For a Java backend:

```text
REST Request
     ↓
Controller
     ↓
Service
     ↓
@Transactional
     ↓
Repository
     ↓
Spring Data JPA
     ↓
Hibernate
     ↓
JDBC
     ↓
HikariCP
     ↓
SQL
     ↓
Database
```

Performance:

```text
API slow
 ↓
Query count?
 ↓
N+1?
 ↓
Generated SQL?
 ↓
EXPLAIN?
 ↓
Indexes?
 ↓
Rows fetched?
 ↓
Transaction duration?
 ↓
Connection pool?
 ↓
Cache?
```

Correctness:

```text
Business operation
 ↓
Transaction
 ↓
Constraints
 ↓
Concurrency control
 ↓
Commit / rollback
```

---

# 126. Final Interview Checklist

```text
□ JPA
□ Hibernate
□ Spring Data JPA
□ Entity
□ @Id
□ @GeneratedValue
□ @ManyToOne
□ @OneToMany
□ @OneToOne
□ @ManyToMany
□ mappedBy
□ Owning side
□ LAZY / EAGER
□ N+1
□ JOIN FETCH
□ EntityGraph
□ DTO projection
□ JPQL
□ Native SQL
□ Derived queries
□ @Query
□ Parameter binding
□ SQL injection
□ @Transactional
□ Transaction boundaries
□ readOnly
□ REQUIRED
□ REQUIRES_NEW
□ Isolation levels
□ Dirty reads
□ Non-repeatable reads
□ Phantom reads
□ Optimistic locking
□ Pessimistic locking
□ @Version
□ Flush
□ Commit
□ Persistence context
□ First-level cache
□ Dirty checking
□ Entity lifecycle
□ Cascade
□ orphanRemoval
□ LazyInitializationException
□ ddl-auto
□ Flyway
□ Liquibase
□ JdbcTemplate
□ Pagination
□ Slice vs Page
□ Specifications
□ Bulk updates
□ Connection pool
□ HikariCP
□ Inventory concurrency
□ Outbox pattern
□ Production query optimization
```

---

# 127. Final Rule

For a Java backend interview, connect every JPA answer back to SQL and database behavior.

For example:

> "I use `@ManyToOne(fetch = LAZY)` because the relationship is not always needed. For a list endpoint that needs the related data, I would explicitly fetch the required fields using a DTO projection or fetch join rather than making the relationship globally eager. I'd then verify the generated SQL and execution plan."

That answer demonstrates:

```text
Java
+
Spring
+
JPA
+
SQL
+
Performance
```

rather than only memorizing annotations.
