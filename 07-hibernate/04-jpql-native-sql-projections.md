# Hibernate & JPA — File 04: JPQL, Native SQL & Projections

This file covers how to write and optimize queries with Spring Data JPA and Hibernate.

Core topics:

```text
JPQL
@Query
Parameters
Named Parameters
Positional Parameters
JOIN
JOIN FETCH
LEFT JOIN
INNER JOIN
GROUP BY
HAVING
ORDER BY
Subqueries
Native SQL
@NativeQuery / nativeQuery
Interface Projections
DTO Projections
Constructor Expressions
Scalar Results
Pagination
Sorting
Specifications
Criteria API
Dynamic Queries
Query Design
Common Mistakes
Interview Questions
Production Scenarios
```

---

# 1. Why Querying Matters

Hibernate can generate SQL automatically, but real applications need custom queries.

Examples:

```text
Find orders by status
Find products within a price range
Find customers with recent orders
Generate reports
Fetch an order with its items
Calculate totals
Search dynamically
```

For these cases, we use:

```text
Derived queries
JPQL
Native SQL
Specifications
Criteria API
```

---

# 2. Query Options in Spring Data JPA

A repository can use:

```text
Method-name queries
@Query with JPQL
@Query with native SQL
Specifications
Criteria API
QueryDSL
```

Choose the simplest approach that remains readable and maintainable.

---

# 3. Derived Query

Simple query:

```java
Optional<User> findByEmail(String email);
```

Spring Data derives the query from the method name.

Good for:

```text
Simple filters
Simple combinations
Common lookups
```

---

# 4. When Derived Queries Become Difficult

Avoid giant method names such as:

```text
findByStatusAndCreatedAtBetweenAndCustomerNameContainingAndTotalGreaterThan...
```

For complex logic, use:

```text
JPQL
Specification
QueryDSL
Native SQL
```

depending on the use case.

---

# 5. What Is JPQL?

JPQL means:

```text
Java Persistence Query Language
```

It is an object-oriented query language defined by JPA.

Important:

> JPQL queries entities and their fields, not database tables and columns directly.

---

# 6. JPQL Example

Entity:

```java
@Entity
class User {

    private Long id;
    private String name;
    private String email;
}
```

JPQL:

```java
@Query("""
    select u
    from User u
    where u.email = :email
""")
Optional<User> findByEmail(@Param("email") String email);
```

Notice:

```text
User
email
```

are Java entity/property names.

---

# 7. JPQL vs SQL

SQL:

```sql
SELECT *
FROM users
WHERE email = ?;
```

JPQL:

```sql
SELECT u
FROM User u
WHERE u.email = :email
```

JPQL works with the entity model.

---

# 8. JPQL Does Not Use Table Names

Suppose:

```java
@Entity
@Table(name = "customer_accounts")
class Customer {
}
```

JPQL still uses:

```text
Customer
```

not:

```text
customer_accounts
```

unless you are writing native SQL.

---

# 9. JPQL Uses Entity Property Names

Suppose:

```java
@Column(name = "email_address")
private String email;
```

JPQL uses:

```text
u.email
```

not:

```text
u.email_address
```

---

# 10. @Query

Spring Data JPA allows custom queries:

```java
@Query("""
    select u
    from User u
    where u.name = :name
""")
List<User> findUsersByName(@Param("name") String name);
```

---

# 11. Named Parameters

Recommended style:

```java
@Query("""
    select u
    from User u
    where u.email = :email
""")
Optional<User> findByEmail(
    @Param("email") String email
);
```

The query parameter:

```text
:email
```

maps to:

```java
@Param("email")
```

---

# 12. Positional Parameters

You can also use:

```java
@Query("""
    select u
    from User u
    where u.email = ?1
""")
Optional<User> findByEmail(String email);
```

Meaning:

```text
?1 → first method argument
```

Named parameters are generally easier to maintain.

---

# 13. Why Prefer Named Parameters?

Compare:

```text
?1
?2
?3
```

with:

```text
:name
:status
:minTotal
```

Named parameters are:

```text
More readable
Less error-prone
Easier to maintain
```

---

# 14. JPQL SELECT

Example:

```java
@Query("""
    select u
    from User u
""")
List<User> findAllUsers();
```

This returns:

```text
List<User>
```

---

# 15. Filtering

```java
@Query("""
    select p
    from Product p
    where p.price > :price
""")
List<Product> findExpensiveProducts(
    @Param("price") BigDecimal price
);
```

---

# 16. AND

```java
@Query("""
    select p
    from Product p
    where p.status = :status
      and p.price > :price
""")
List<Product> findProducts(
    @Param("status") ProductStatus status,
    @Param("price") BigDecimal price
);
```

---

# 17. OR

```java
@Query("""
    select p
    from Product p
    where p.status = :status
       or p.featured = true
""")
List<Product> findAvailableProducts(
    @Param("status") ProductStatus status
);
```

Use parentheses when combining complex `AND`/`OR` expressions so the intended precedence is explicit.

---

# 18. BETWEEN

```java
where p.price between :min and :max
```

Useful for:

```text
Price ranges
Dates
Numbers
```

Always confirm the boundary semantics required by the business rule.

---

# 19. IN

```java
where p.status in :statuses
```

Repository:

```java
List<Product> findByStatusIn(
    @Param("statuses") Collection<ProductStatus> statuses
);
```

Useful when filtering against multiple values.

---

# 20. LIKE

```java
where p.name like :pattern
```

Example:

```text
%phone%
```

can match:

```text
Smart Phone
Phone Case
Gaming Phone
```

Leading wildcards can prevent efficient use of normal B-tree indexes, so search requirements should be considered.

---

# 21. LOWER

Case-insensitive comparison can be implemented with:

```java
where lower(u.email) = lower(:email)
```

Database collation/index behavior should also be considered for performance.

---

# 22. ORDER BY

```java
@Query("""
    select o
    from Order o
    order by o.createdAt desc
""")
List<Order> findLatestOrders();
```

---

# 23. Sorting with Spring Data

Spring Data can also accept:

```java
Sort sort
```

Example:

```java
List<Product> findByStatus(
    ProductStatus status,
    Sort sort
);
```

This can be useful when sorting is allowed to vary without changing the query.

---

# 24. Pagination

Repository:

```java
Page<Order> findByStatus(
    OrderStatus status,
    Pageable pageable
);
```

Usage:

```java
Pageable pageable =
    PageRequest.of(0, 20);
```

This gives:

```text
Page 0
20 records
```

---

# 25. JPQL with Pagination

```java
@Query("""
    select o
    from Order o
    where o.status = :status
    order by o.createdAt desc
""")
Page<Order> findOrders(
    @Param("status") OrderStatus status,
    Pageable pageable
);
```

Spring Data handles pagination around the query.

For complex queries, always inspect the generated count query and SQL.

---

# 26. Page vs Slice

`Page` provides:

```text
Content
Total elements
Total pages
```

It may require a count query.

`Slice` provides:

```text
Current content
Whether another slice exists
```

It can avoid the need for a total-count query in some cases.

---

# 27. When to Use Slice

If the UI only needs:

```text
Next page?
```

rather than:

```text
There are exactly 4,821 results.
```

a `Slice` can be more efficient.

---

# 28. JOIN

Suppose:

```text
Order → Customer
```

JPQL:

```java
select o
from Order o
join o.customer c
where c.email = :email
```

This lets the query use the related entity.

---

# 29. INNER JOIN

Regular:

```text
JOIN
```

is effectively an inner join in JPQL.

Only matching relationships are returned.

---

# 30. LEFT JOIN

Example:

```java
select c
from Customer c
left join c.orders o
```

This can return customers even when they have no matching orders.

Useful when the parent should remain in the result even if the association is empty.

---

# 31. JOIN FETCH

As covered in the previous file:

```java
select o
from Order o
join fetch o.customer
```

This means:

```text
Join
+
Fetch the relationship into the entity graph
```

---

# 32. JOIN vs JOIN FETCH

Normal JOIN:

```text
Useful for filtering/query logic
```

JOIN FETCH:

```text
Useful when the associated entity should be initialized as part of the query
```

This is a frequent interview question.

---

# 33. LEFT JOIN FETCH

Example:

```java
select o
from Order o
left join fetch o.items
where o.id = :id
```

Use `LEFT JOIN FETCH` when the association may be empty but the root entity should still be returned.

---

# 34. JOIN FETCH and N+1

Without fetch join:

```text
Order query
+
Customer query
+
Customer query
+
Customer query
```

With fetch join:

```text
Order
 +
Customer
```

can often be loaded through one query.

---

# 35. GROUP BY

Example:

```java
select o.status, count(o)
from Order o
group by o.status
```

Result conceptually:

```text
PENDING   120
PAID      950
CANCELLED 35
```

---

# 36. HAVING

`HAVING` filters groups after aggregation.

Example:

```java
select o.status, count(o)
from Order o
group by o.status
having count(o) > 100
```

This returns only statuses with more than 100 orders.

---

# 37. WHERE vs HAVING

Remember:

```text
WHERE
→ filters rows before grouping

HAVING
→ filters groups after grouping
```

---

# 38. Aggregate Functions

JPQL supports common aggregates such as:

```text
COUNT
SUM
AVG
MIN
MAX
```

Example:

```java
select sum(o.total)
from Order o
where o.status = :status
```

---

# 39. Scalar Results

A query does not always return an entity.

Example:

```java
@Query("""
    select count(o)
    from Order o
""")
long countOrders();
```

Return type:

```text
long
```

---

# 40. Object Arrays / Tuples

A query can select multiple scalar values:

```java
select o.status, count(o)
from Order o
group by o.status
```

Depending on the API, results can be represented as:

```text
Object[]
Tuple
DTO
```

DTO projections are usually clearer for application code.

---

# 41. DTO Projection

Example DTO:

```java
public record OrderSummary(
    Long id,
    BigDecimal total,
    String customerName
) {}
```

JPQL:

```java
@Query("""
    select new com.example.OrderSummary(
        o.id,
        o.total,
        c.name
    )
    from Order o
    join o.customer c
""")
List<OrderSummary> findSummaries();
```

---

# 42. Constructor Expression

This part:

```text
new com.example.OrderSummary(...)
```

is a JPQL constructor expression.

Hibernate creates the DTO directly from the query result.

---

# 43. Why DTO Projection Is Useful

It avoids loading:

```text
Entire Order
Entire Customer
All unnecessary fields
```

when the API only needs:

```text
Order ID
Total
Customer Name
```

---

# 44. Interface Projection

Spring Data JPA supports interface-based projections.

Example:

```java
public interface OrderSummaryView {

    Long getId();

    BigDecimal getTotal();

    String getCustomerName();
}
```

Repository:

```java
@Query("""
    select
        o.id as id,
        o.total as total,
        c.name as customerName
    from Order o
    join o.customer c
""")
List<OrderSummaryView> findSummaries();
```

Aliases should match the projection getter/property names.

---

# 45. Interface vs DTO Projection

Interface:

```text
Simple read views
Spring Data convenience
```

DTO:

```text
Explicit type
More control
Clear constructor contract
Useful for service/API boundaries
```

Choose based on the use case.

---

# 46. Native SQL

Native queries use actual database SQL.

Example:

```java
@Query(
    value = """
        SELECT id, name, price
        FROM products
        WHERE price > :price
    """,
    nativeQuery = true
)
List<Product> findProducts(
    @Param("price") BigDecimal price
);
```

Now:

```text
products
price
```

are database table/column names.

---

# 47. JPQL vs Native SQL

JPQL:

```text
Portable
Entity-oriented
Database-independent at the query-language level
```

Native SQL:

```text
Database-specific
Full SQL features
Useful for specialized queries
```

---

# 48. When to Use Native SQL

Good candidates:

```text
Database-specific features
Complex SQL
Window functions
Vendor-specific syntax
CTEs where supported/needed
Performance-critical SQL that is clearer in native form
Existing legacy queries
```

Don't use native SQL simply because JPQL feels unfamiliar.

---

# 49. Native SQL Trade-Offs

Pros:

```text
Full database capabilities
Precise SQL control
Potentially easier for complex SQL
```

Cons:

```text
Database coupling
Less portability
More schema coupling
Mapping complexity
Harder migrations across databases
```

---

# 50. Native Query Column Names

Native SQL uses:

```text
Database columns
```

not:

```text
Java entity properties
```

Example:

```sql
customer_id
```

rather than:

```text
customer
```

if the actual database column is `customer_id`.

---

# 51. Native Query and Database Functions

Native SQL can use database-specific functions.

Example:

```sql
DATE_TRUNC(...)
```

or:

```sql
JSON_EXTRACT(...)
```

depending on the database.

This is powerful but reduces portability.

---

# 52. JPQL Subquery

JPQL supports subqueries in appropriate contexts, commonly within:

```text
WHERE
HAVING
```

Example:

```java
select o
from Order o
where o.total >
    (select avg(o2.total) from Order o2)
```

This finds orders above average total.

---

# 53. EXISTS

Example:

```java
select c
from Customer c
where exists (
    select o
    from Order o
    where o.customer = c
)
```

This can find customers who have at least one order.

---

# 54. NOT EXISTS

Example:

```java
select c
from Customer c
where not exists (
    select o
    from Order o
    where o.customer = c
)
```

This can find customers with no orders.

---

# 55. Query Parameters and Injection

Never build JPQL using string concatenation:

```java
"where u.email = '" + email + "'"
```

Use:

```java
@Param
```

or parameter binding.

This improves safety and query handling.

---

# 56. Dynamic Search

Suppose an API supports:

```text
name
status
minPrice
maxPrice
category
createdAfter
```

Avoid building huge strings manually.

Consider:

```text
Specification
Criteria API
QueryDSL
```

depending on project requirements.

---

# 57. JPA Specification

Spring Data JPA supports:

```java
Specification<Product>
```

This allows reusable dynamic predicates.

Conceptually:

```text
Filter 1
+
Filter 2
+
Filter 3
→
Dynamic query
```

---

# 58. Specification Example

Conceptually:

```java
Specification<Product> spec =
    hasStatus(status)
    .and(priceGreaterThan(minPrice))
    .and(nameContains(name));
```

Then:

```java
productRepository.findAll(spec);
```

This is useful for complex search screens.

---

# 59. Criteria API

JPA Criteria API builds queries programmatically.

Conceptually:

```text
Root
 ↓
Predicate
 ↓
CriteriaQuery
 ↓
EntityManager
```

It is type-aware compared with raw query strings, but can be verbose.

---

# 60. Criteria API vs Specification

Spring Data Specifications commonly use the JPA Criteria API underneath.

Think:

```text
Specification
→ convenient Spring Data abstraction

Criteria API
→ standard JPA programmatic query API
```

---

# 61. QueryDSL

QueryDSL is an external library that provides a fluent, strongly typed query API.

Example style:

```text
product.price.gt(...)
product.status.eq(...)
```

It can be useful for complex dynamic queries, though it adds project dependencies and build configuration.

---

# 62. Query Design

Before writing a query, ask:

```text
What data is actually needed?
How many rows can this return?
Which tables/relationships are involved?
Can indexes support the filters?
Is pagination required?
Do I need entities or DTOs?
Could this create N+1?
```

---

# 63. Avoid SELECT *

JPQL:

```java
select u
```

is normal for loading an entity.

For read-only endpoints, consider:

```text
DTO projection
```

rather than loading an entire entity if only a few fields are required.

---

# 64. Index Awareness

Suppose:

```java
where p.status = :status
and p.createdAt > :date
```

Consider whether the database has useful indexes for the actual workload.

Hibernate doesn't automatically make every query fast.

---

# 65. Query Execution Plan

For slow native SQL or generated SQL, inspect:

```text
EXPLAIN
EXPLAIN ANALYZE
```

depending on database.

Look for:

```text
Full table scans
Bad join order
Missing indexes
Large row counts
Sorts
Temporary tables
```

---

# 66. JPQL Does Not Guarantee Efficient SQL

This:

```java
select o
from Order o
where o.status = :status
```

can still be slow if:

```text
orders.status
```

is poorly indexed for the workload.

ORM abstraction doesn't eliminate database performance concerns.

---

# 67. Query Count vs Query Complexity

One query is not automatically better than five.

Example:

```text
One enormous join
```

may be worse than:

```text
Two small targeted queries
```

Measure:

```text
Rows
Latency
Memory
DB CPU
Network
```

---

# 68. Count Query

When using:

```java
Page<T>
```

Spring Data may execute a count query to determine:

```text
Total elements
Total pages
```

For expensive queries, count operations can become a bottleneck.

Consider:

```text
Slice
Custom countQuery
Keyset pagination
```

when appropriate.

---

# 69. Custom Count Query

For complex pageable queries, you can define:

```java
@Query(
    value = "...",
    countQuery = "..."
)
Page<Order> findOrders(
    Pageable pageable
);
```

The count query should be designed specifically for counting and need not duplicate unnecessary fetches.

---

# 70. Fetch Join and Count Queries

A common mistake is using:

```text
JOIN FETCH
```

in a count query.

A count query generally should count root records without fetching collections.

---

# 71. Query Method Return Types

Possible return types include:

```text
Entity
Optional<Entity>
List<Entity>
Page<Entity>
Slice<Entity>
DTO
Projection
long
boolean
```

Choose the type that matches the semantics of the query.

---

# 72. Optional

Good for a query expected to return:

```text
0 or 1
```

Example:

```java
Optional<User> findByEmail(String email);
```

---

# 73. List

Good for:

```text
Multiple results
```

Example:

```java
List<Product> findByCategory(String category);
```

Be careful with unbounded result sets.

---

# 74. Page

Good when the client needs:

```text
Data
+
Total count/page information
```

---

# 75. Slice

Good when the client needs:

```text
Data
+
Whether more results exist
```

without necessarily needing a full count.

---

# 76. Bulk Update

JPQL can update multiple rows:

```java
@Modifying
@Query("""
    update Product p
    set p.status = :status
    where p.category.id = :categoryId
""")
int updateStatus(
    @Param("categoryId") Long categoryId,
    @Param("status") ProductStatus status
);
```

---

# 77. @Modifying

For JPQL update/delete queries in Spring Data:

```java
@Modifying
```

is commonly required.

Example:

```java
@Modifying
@Query("delete from User u where u.id = :id")
int deleteUser(@Param("id") Long id);
```

---

# 78. Bulk Update Warning

Bulk JPQL updates operate directly against database rows and can bypass normal per-entity lifecycle behavior.

The persistence context may contain stale entity state afterward.

Consider:

```text
clearAutomatically = true
```

or explicit context management when appropriate.

---

# 79. Bulk Delete Warning

Similarly, bulk deletes do not behave exactly like:

```java
entityManager.remove(entity)
```

for every managed entity.

They execute a bulk database operation.

Understand the persistence-context consequences.

---

# 80. flushAutomatically

For modifying queries, Spring Data can provide options such as:

```java
@Modifying(
    flushAutomatically = true,
    clearAutomatically = true
)
```

Use them when the operation requires synchronization/clearing behavior.

Don't enable them blindly.

---

# 81. Query Naming

Prefer descriptive repository methods:

```text
findRecentOrdersForCustomer
findActiveProductsByCategory
findOrderSummaryById
```

rather than:

```text
query1
getData
findStuff
```

---

# 82. Repository Responsibility

Repositories should focus on persistence/query operations.

Business rules belong in:

```text
Service/domain layer
```

Avoid turning repositories into huge business-logic classes.

---

# 83. Example Repository

```java
public interface OrderRepository
        extends JpaRepository<Order, Long> {

    @Query("""
        select o
        from Order o
        join fetch o.customer
        where o.id = :id
    """)
    Optional<Order> findDetailsById(
        @Param("id") Long id
    );
}
```

The repository describes how the data is retrieved.

---

# 84. Service Layer

```java
@Transactional(readOnly = true)
public OrderResponse getOrder(Long id) {

    Order order = repository.findDetailsById(id)
        .orElseThrow();

    return mapper.toResponse(order);
}
```

This keeps:

```text
Controller
 ↓
Service
 ↓
Repository
```

clear.

---

# 85. Common Query Mistake

Bad:

```java
@Query("select o from Order o")
List<Order> findEverything();
```

for an endpoint with millions of orders.

Problems:

```text
Memory
Latency
Database load
Network
```

Use pagination or a targeted query.

---

# 86. Common Query Mistake

Bad:

```text
SELECT full entity
```

when only:

```text
id
name
price
```

are required.

Consider DTO projection.

---

# 87. Common Query Mistake

Bad:

```text
Use native SQL everywhere
```

Why?

```text
Database coupling
Schema coupling
Reduced portability
More maintenance
```

Use it when it provides real value.

---

# 88. Common Query Mistake

Bad:

```text
Use JPQL for every complex database-specific operation
```

Sometimes native SQL is clearer and more appropriate.

Choose the right abstraction.

---

# 89. Common Query Mistake

Bad:

```text
Ignore SQL generated by Hibernate
```

Always understand what your ORM is doing for important queries.

---

# 90. Interview Question

### What is JPQL?

Answer:

> "JPQL is the object-oriented query language defined by JPA. It queries entities and their properties rather than directly querying database tables and columns."

---

# 91. Interview Question

### JPQL vs SQL?

Answer:

> "JPQL operates on the entity model and is designed to be database-independent, while SQL operates directly on database tables and columns. Hibernate translates JPQL into SQL."

---

# 92. Interview Question

### When would you use native SQL?

Answer:

> "I'd use native SQL when I need database-specific features, complex SQL that is clearer natively, specialized performance tuning, or functionality that is difficult to express through JPQL."

---

# 93. Interview Question

### What is DTO projection?

Answer:

> "DTO projection retrieves only the fields needed by a use case and maps them into a dedicated DTO rather than loading complete entity graphs."

---

# 94. Interview Question

### Interface projection vs DTO projection?

Answer:

> "Interface projections are a convenient Spring Data mechanism for read views, while DTO projections provide an explicit concrete type and are often useful at service or API boundaries."

---

# 95. Interview Question

### What is a Specification?

Answer:

> "A Spring Data JPA Specification provides a reusable way to build dynamic predicates using the JPA Criteria API, which is useful for complex search/filter APIs."

---

# 96. Interview Question

### What is @Modifying?

Answer:

> "`@Modifying` tells Spring Data that a repository query performs an update or delete rather than a normal select query."

---

# 97. Interview Question

### Why can bulk updates be dangerous?

Answer:

> "Bulk updates operate directly on database rows and can leave managed entities in the persistence context stale. I would manage flushing and clearing appropriately after bulk operations."

---

# 98. Interview Question

### Page vs Slice?

Answer:

> "`Page` includes total-result information and can require a count query. `Slice` focuses on the current chunk and whether another chunk exists, so it can avoid an expensive total count."

---

# 99. Interview Scenario

### "A search API has 10 optional filters."

Approach:

```text
Don't create dozens of repository methods.
        ↓
Use Specification / Criteria / QueryDSL
        ↓
Add predicates dynamically
        ↓
Paginate
        ↓
Index important filters
```

---

# 100. Interview Scenario

### "The query works but is slow."

Answer:

```text
1. Capture generated SQL
2. Run EXPLAIN/EXPLAIN ANALYZE
3. Check indexes
4. Check joins
5. Check returned rows
6. Check pagination
7. Consider projection
8. Check N+1
9. Compare query alternatives
10. Measure again
```

---

# 101. Interview Scenario

### "The API only needs three columns from a huge table."

Answer:

> "I'd consider a DTO or interface projection so the database returns only the required columns rather than loading the complete entity."

---

# 102. Interview Scenario

### "You need 1 million search results."

Answer:

> "I would not return one million entities. I'd use pagination or cursor/keyset pagination, apply appropriate indexes, and return only the fields required by the client."

---

# 103. Interview Scenario

### "Why does Page make the endpoint slow?"

Possible reason:

```text
Expensive count query
```

Investigate:

```text
Generated count SQL
Indexes
Query complexity
Result size
```

Consider:

```text
Slice
Keyset pagination
Custom count query
```

---

# 104. Interview Scenario

### "Bulk update succeeded but existing entity objects still show old values."

Likely:

```text
Persistence context contains stale state.
```

After bulk update:

```text
Clear persistence context
or
Refresh affected entities
```

depending on the use case.

---

# 105. Query Design Checklist

Before shipping a query:

```text
□ What data is required?
□ Entity or DTO?
□ Expected row count?
□ Pagination?
□ Index support?
□ N+1 risk?
□ Join cardinality?
□ Fetch join needed?
□ Count query needed?
□ Database-specific feature?
□ Native SQL justified?
□ Generated SQL reviewed?
□ Execution plan reviewed?
□ Transaction semantics understood?
```

---

# 106. Practical Query Flow

```text
API requirement
      ↓
Define exact data needed
      ↓
Choose entity vs DTO
      ↓
Choose derived query / JPQL / native SQL
      ↓
Add filters
      ↓
Add fetch plan
      ↓
Add pagination
      ↓
Check indexes
      ↓
Inspect generated SQL
      ↓
Inspect execution plan
      ↓
Measure
```

---

# 107. Final Interview Answer

If asked:

> "How do you choose between JPQL, native SQL and projections?"

Say:

> "For straightforward entity-based queries I prefer derived queries or JPQL. If the use case is read-heavy and only needs selected fields, I prefer DTO or interface projections. I use native SQL when I need database-specific functionality or complex SQL that is clearer or more efficient natively. Regardless of the approach, I check generated SQL, indexes, query cardinality and execution plans for important queries."

---

# 108. Revision Checklist

```text
□ JPQL
□ JPQL vs SQL
□ @Query
□ Named parameters
□ Positional parameters
□ WHERE
□ AND
□ OR
□ BETWEEN
□ IN
□ LIKE
□ LOWER
□ ORDER BY
□ Sort
□ Pagination
□ Page
□ Slice
□ JOIN
□ INNER JOIN
□ LEFT JOIN
□ JOIN FETCH
□ LEFT JOIN FETCH
□ GROUP BY
□ HAVING
□ COUNT
□ SUM
□ AVG
□ MIN
□ MAX
□ Subqueries
□ EXISTS
□ NOT EXISTS
□ DTO projection
□ Constructor expression
□ Interface projection
□ Native SQL
□ Native query trade-offs
□ Dynamic queries
□ Specification
□ Criteria API
□ QueryDSL
□ Bulk update
□ Bulk delete
□ @Modifying
□ Persistence-context staleness
□ Count queries
□ Query optimization
□ EXPLAIN
□ Index awareness
□ Query scenarios
□ Interview questions
```

---

# 109. What Comes Next

```text
File 05 → Transactions, Locking & Concurrency
```

Next we will cover:

```text
@Transactional
ACID
Propagation
Isolation
Rollback
readOnly
Transaction boundaries
Optimistic locking
Pessimistic locking
@Version
Lost updates
Deadlocks
Isolation anomalies
Flush behavior
Spring transaction proxy
Self-invocation
REQUIRES_NEW
Nested transactions
Distributed transaction considerations
Interview scenarios
```

The key interview lesson is:

> **Hibernate querying is not just about writing JPQL. A strong backend developer understands the entity model, the generated SQL, fetch behavior, pagination, indexes, transaction boundaries and the performance characteristics of the database underneath the ORM.**
