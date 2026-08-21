# Hibernate & JPA — File 03: Fetching, Lazy/Eager Loading & N+1

This file focuses on one of the most important Hibernate interview areas:

```text
Lazy Loading
Eager Loading
Fetch Strategies
N+1 Query Problem
JOIN FETCH
EntityGraph
Batch Fetching
DTO Projections
LazyInitializationException
Open Session in View
Pagination
Collection Fetching
MultipleBagFetchException
Query Design
Performance Debugging
Production Scenarios
Interview Questions
```

---

# 1. What Is Fetching?

Fetching determines when and how associated entity data is loaded from the database.

Example:

```text
Order
 |
 +---- Customer
 |
 +---- OrderItems
```

When loading an Order, Hibernate must decide:

```text
Load Customer now?
Load OrderItems now?
Load them later?
Load them using another query?
Load them with a join?
```

This is the purpose of fetch planning.

---

# 2. Lazy Loading

Lazy loading means associated data is loaded only when needed.

Example:

```java
@OneToMany(
    mappedBy = "order",
    fetch = FetchType.LAZY
)
private List<OrderItem> items;
```

Conceptually:

```text
Load Order
    ↓
Order loaded

Order.items
    ↓
Not necessarily loaded

order.getItems()
    ↓
Hibernate loads items
```

---

# 3. Why Use Lazy Loading?

Suppose:

```text
Order
 ├── Customer
 ├── 50 Items
 ├── Payments
 ├── Audit Logs
 └── Shipping Details
```

An API may only need:

```text
Order ID
Total
Status
```

Loading everything immediately wastes:

```text
Database work
Network bandwidth
Memory
CPU
```

Lazy loading helps avoid unnecessary work.

---

# 4. Eager Loading

Eager loading means the association is fetched as part of loading the entity.

Conceptually:

```text
Load Order
    ↓
Order + association data
```

Eager loading can be useful for relationships that are always required, but careless eager loading can create performance problems.

---

# 5. JPA Default Fetch Types

Important interview fact:

```text
@ManyToOne → EAGER
@OneToOne  → EAGER

@OneToMany → LAZY
@ManyToMany → LAZY
```

These are JPA defaults.

However:

> Know the defaults for interviews, but design production fetch plans intentionally.

---

# 6. Setting FetchType

Example:

```java
@ManyToOne(fetch = FetchType.LAZY)
private Customer customer;
```

Collection:

```java
@OneToMany(
    mappedBy = "order",
    fetch = FetchType.LAZY
)
private List<OrderItem> items;
```

---

# 7. Lazy Is Not the Same as "No Query"

Lazy means:

> Don't load the association until it is required.

If code accesses:

```java
order.getItems()
```

Hibernate may execute a query.

So:

```text
LAZY
```

doesn't mean:

```text
Never query
```

It means:

```text
Delay query until needed
```

---

# 8. Lazy Loading and Proxies

Hibernate can use proxies or bytecode enhancement to support lazy loading.

Conceptually:

```text
Order
 ↓
Proxy/reference
 ↓
Actual data requested
 ↓
Database query
```

The exact implementation depends on the association, Hibernate version and enhancement/proxy configuration.

---

# 9. LazyInitializationException

Classic problem:

```text
Transaction
   ↓
Load Order
   ↓
Transaction ends
   ↓
Persistence Context unavailable
   ↓
Access lazy items
   ↓
LazyInitializationException
```

Example:

```java
Order order = service.getOrder(id);

// Persistence context may now be closed

order.getItems().size();
```

If the collection was not initialized, Hibernate may fail.

---

# 10. Why Does LazyInitializationException Happen?

Usually:

```text
Lazy association
+
No active persistence context
+
Association accessed
```

It is usually a sign that the fetch plan or transaction boundary doesn't match the use case.

---

# 11. Bad Fix #1 — Make Everything EAGER

A common reaction:

```java
@ManyToOne(fetch = FetchType.EAGER)
```

everywhere.

This can cause:

```text
Too many joins
Too many queries
Large object graphs
High memory usage
Slow APIs
```

Don't use EAGER as a universal fix.

---

# 12. Better Approach

Ask:

```text
What data does this use case need?
```

Then explicitly fetch that data.

Options:

```text
JOIN FETCH
EntityGraph
DTO projection
Batch fetching
Purpose-built query
```

---

# 13. N+1 Query Problem

This is one of the most frequently asked Hibernate interview topics.

Suppose:

```java
List<Order> orders = orderRepository.findAll();
```

This executes:

```text
1 query
```

Then:

```java
for (Order order : orders) {
    order.getCustomer().getName();
}
```

could cause:

```text
1 query for orders
+
N queries for customers
```

Total:

```text
N + 1
```

---

# 14. Example N+1

Suppose there are:

```text
100 orders
```

Possible queries:

```text
1 → SELECT orders
100 → SELECT customer
```

Total:

```text
101 queries
```

The application may appear correct but become slow under production load.

---

# 15. Why N+1 Is Dangerous

If:

```text
N = 10
```

then:

```text
11 queries
```

Maybe acceptable.

But:

```text
N = 10,000
```

becomes:

```text
10,001 queries
```

This can create:

```text
Database overload
Connection pool pressure
High latency
CPU overhead
Network overhead
```

---

# 16. Detecting N+1

Useful techniques:

```text
Enable SQL logging in development
Use Hibernate statistics
Use APM tools
Use distributed tracing
Inspect database query metrics
Use integration tests that assert query counts
```

Don't rely only on code inspection.

---

# 17. JOIN FETCH

One common solution is JPQL `JOIN FETCH`.

Example:

```java
@Query("""
    select o
    from Order o
    join fetch o.customer
    where o.id = :id
""")
Optional<Order> findOrderWithCustomer(Long id);
```

Conceptually:

```text
Order
  +
Customer
```

can be loaded through one SQL join.

---

# 18. JOIN vs JOIN FETCH

Normal join:

```sql
JOIN
```

can affect query filtering.

`JOIN FETCH` additionally tells JPA:

> Fetch the associated entity as part of this query result.

This distinction is important.

---

# 19. JOIN FETCH Example

Without fetch join:

```text
SELECT orders
SELECT customer 1
SELECT customer 2
SELECT customer 3
...
```

With fetch join:

```text
SELECT orders
JOIN customers
```

The exact SQL depends on mappings and query shape.

---

# 20. JOIN FETCH and Collections

Example:

```java
@Query("""
    select o
    from Order o
    join fetch o.items
    where o.id = :id
""")
Optional<Order> findOrderWithItems(Long id);
```

Useful when the API specifically needs:

```text
Order + Items
```

---

# 21. JOIN FETCH Can Create Duplicate Parent Rows

Suppose:

```text
Order 1
 ├── Item A
 ├── Item B
 └── Item C
```

SQL join produces multiple rows for the same order.

Conceptually:

```text
Order 1 | Item A
Order 1 | Item B
Order 1 | Item C
```

Hibernate can assemble the object graph, but query result semantics need careful handling.

---

# 22. SELECT DISTINCT with JOIN FETCH

JPQL can use:

```java
select distinct o
from Order o
join fetch o.items
```

This can help eliminate duplicate root entities in the result.

But understand what `DISTINCT` means in the ORM/query context and verify the generated SQL and behavior for your Hibernate version.

---

# 23. EntityGraph

Spring Data JPA supports EntityGraph.

Example:

```java
@EntityGraph(attributePaths = {"items"})
Optional<Order> findById(Long id);
```

This lets you define a fetch plan without writing a fetch-join query.

---

# 24. EntityGraph Example

Entity:

```java
@Entity
class Order {

    @OneToMany(mappedBy = "order")
    private List<OrderItem> items;
}
```

Repository:

```java
@EntityGraph(attributePaths = {"items"})
Optional<Order> findById(Long id);
```

Now this repository operation requests:

```text
Order
+
Items
```

---

# 25. JOIN FETCH vs EntityGraph

JOIN FETCH:

```text
Explicit query
Precise query control
Useful for complex query conditions
```

EntityGraph:

```text
Reusable fetch plan
Works well with repository methods
Separates fetch plan from query in many cases
```

Choose based on readability and query requirements.

---

# 26. DTO Projection

Instead of loading entities and relationships, fetch only the data needed by the API.

Example:

```java
public record OrderSummary(
    Long id,
    BigDecimal total,
    String customerName
) {}
```

Query:

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
List<OrderSummary> findOrderSummaries();
```

This can be highly efficient for read-heavy use cases.

---

# 27. Why DTO Projection Helps

Instead of:

```text
Entity
 ↓
Customer
 ↓
Orders
 ↓
Items
 ↓
...
```

load only:

```text
Order ID
Order Total
Customer Name
```

Benefits:

```text
Less data
Less memory
Fewer accidental lazy loads
Clear API intent
Potentially simpler SQL
```

---

# 28. DTO Projection vs Entity

Entity query:

```text
Good for:
Business logic
Updates
Managed state
Domain operations
```

DTO projection:

```text
Good for:
Read-only responses
Reports
Lists
Dashboards
Large datasets
```

---

# 29. Batch Fetching

Hibernate can batch lazy association loading.

Instead of:

```text
Customer 1
Customer 2
Customer 3
...
```

one query at a time, Hibernate may load multiple IDs together.

Conceptually:

```sql
SELECT *
FROM customer
WHERE id IN (?, ?, ?, ?)
```

This can reduce N+1 severity.

---

# 30. @BatchSize

Example:

```java
@BatchSize(size = 20)
@ManyToOne(fetch = FetchType.LAZY)
private Customer customer;
```

Hibernate can batch-load related entities.

Exact behavior depends on the query/use case and Hibernate configuration.

---

# 31. default_batch_fetch_size

Hibernate can also configure a default batch fetch size.

Example:

```properties
spring.jpa.properties.hibernate.default_batch_fetch_size=20
```

This can be useful for reducing repeated lazy-loading queries.

Tune based on workload rather than choosing an arbitrary large value.

---

# 32. Batch Fetching Does Not Mean JOIN FETCH

JOIN FETCH:

```text
One query with join
```

Batch fetching:

```text
Multiple IDs loaded together
```

Example:

```text
JOIN FETCH:
Order + Customer

Batch:
Orders loaded
 ↓
Customers loaded in groups
```

---

# 33. FetchMode

Hibernate has provider-specific fetching options.

Concepts can include:

```text
SELECT
JOIN
SUBSELECT
```

These are Hibernate-specific and should be used only when their behavior is understood.

---

# 34. SUBSELECT Fetching

Suppose:

```text
Load 100 orders
```

and then access their items.

A subselect strategy can conceptually load all relevant items using a query based on the original parent selection.

This can reduce one-query-per-parent behavior.

---

# 35. MultipleBagFetchException

A classic Hibernate-specific problem.

Suppose an entity has:

```java
@OneToMany
List<Item> items;

@OneToMany
List<Payment> payments;
```

and you try to fetch-join both collections in one query.

Hibernate may throw:

```text
MultipleBagFetchException
```

when both are represented as bags.

---

# 36. Why MultipleBagFetchException Happens

Fetching multiple bag collections simultaneously can create ambiguous/huge cartesian-style result sets.

Example:

```text
Order
 ├── 10 Items
 └── 5 Payments
```

A join can produce:

```text
10 × 5 = 50 rows
```

for one order.

For large collections this becomes expensive.

---

# 37. Better Solutions

Depending on the use case:

```text
Fetch one collection at a time
Use Set where semantically appropriate
Use DTO queries
Use batch fetching
Use separate queries
Redesign the read model
```

Don't simply change `List` to `Set` without considering whether set semantics are correct.

---

# 38. Cartesian Explosion

Suppose:

```text
Order
  |
  +-- 100 Items
  |
  +-- 20 Payments
```

Joining both collections can produce roughly:

```text
100 × 20 = 2,000 rows
```

for one root entity before ORM reconstruction.

This is why fetching multiple collections requires care.

---

# 39. Pagination

Suppose:

```text
10 million orders
```

Never load:

```java
findAll();
```

and return everything.

Use pagination:

```text
Page
Slice
Keyset/seek pagination
```

depending on requirements.

---

# 40. Offset Pagination

Example:

```sql
LIMIT 20 OFFSET 100000
```

Simple but can become expensive at large offsets because the database may need to scan/skip many rows.

---

# 41. Keyset Pagination

Instead of:

```text
OFFSET 100000
```

use a stable cursor:

```sql
WHERE id < :lastSeenId
ORDER BY id DESC
LIMIT 20
```

This can be much more efficient for deep pagination when the ordering/index supports it.

---

# 42. Fetch Join + Pagination Warning

Pagination over a collection fetch join is problematic because the SQL result is multiplied by child rows.

Example:

```text
Order
 ↓
Items
```

A database-level page of joined rows may not correspond to a page of distinct orders.

Hibernate may warn, reject, or fall back to in-memory handling depending on configuration/version.

---

# 43. Better Pattern for Collection Pagination

Two-step approach:

```text
Step 1:
Fetch page of Order IDs

Step 2:
Fetch Orders + Items
using those IDs
```

This can preserve correct parent pagination.

---

# 44. EntityGraph + Pagination

EntityGraph can be useful with pagination, but collection fetching still needs careful consideration.

Always inspect:

```text
Generated SQL
Result size
Duplicate rows
Database execution plan
```

---

# 45. Open Session in View

OSIV means the persistence context can remain available through the web request.

This may allow:

```text
Controller
 ↓
DTO conversion
 ↓
Lazy association
 ↓
Database query
```

after the service transaction has ended.

---

# 46. OSIV Advantage

It can make development simpler because lazy associations may still be accessible during request processing.

---

# 47. OSIV Risk

It can hide poor fetch planning.

Example:

```java
for (Order order : orders) {
    order.getCustomer().getName();
}
```

The controller may trigger:

```text
1 + N
```

queries.

The transaction boundary no longer makes the problem obvious.

---

# 48. Better Approach

For important APIs:

```text
Controller
 ↓
Service
 ↓
Purpose-built query/fetch plan
 ↓
DTO
 ↓
Response
```

Make data requirements explicit.

---

# 49. Transaction Boundary and Fetching

Example:

```java
@Transactional(readOnly = true)
public OrderResponse getOrder(Long id) {
    Order order = repository.findOrderWithItems(id);
    return mapper.toResponse(order);
}
```

The service knows:

```text
This use case requires items.
```

So it fetches them intentionally.

---

# 50. Avoid Lazy Loading in Controllers

Avoid:

```java
@GetMapping("/{id}")
public OrderResponse getOrder(@PathVariable Long id) {

    Order order = service.getOrder(id);

    order.getItems(); // hidden DB access
}
```

Better:

```text
Service fetches required data
 ↓
Maps to DTO
 ↓
Controller returns DTO
```

---

# 51. Query Count Matters

For an endpoint:

```text
GET /orders/100
```

You should understand whether it executes:

```text
1 query
5 queries
100 queries
```

Correctness alone is not enough.

---

# 52. Performance Investigation

If an endpoint becomes slow:

```text
Check latency
 ↓
Check traces
 ↓
Check SQL count
 ↓
Check slow queries
 ↓
Check execution plan
 ↓
Check indexes
 ↓
Check fetch strategy
 ↓
Optimize
```

---

# 53. Hibernate Statistics

Hibernate can expose statistics useful for debugging:

```text
Entity load count
Query count
Cache hits
Flush count
```

Use in development/testing and controlled diagnostics.

---

# 54. Query Logging

During development:

```properties
spring.jpa.show-sql=true
```

For production-grade diagnostics, use structured logging configuration and appropriate Hibernate SQL/bind-parameter logging carefully.

Avoid exposing sensitive values.

---

# 55. N+1 Test Strategy

A useful integration test can verify query behavior.

Conceptually:

```text
Load 20 orders
 ↓
Access required data
 ↓
Assert query count stays within expected range
```

This prevents performance regressions.

---

# 56. Fetch Plan

A fetch plan answers:

```text
What entities/fields does this use case need?
```

Example:

```text
Order Details API:
Order
Customer
Items
Product summary
```

Instead of:

```text
Order
Customer
Customer Orders
Payments
Audit Logs
Shipping History
```

Fetch only what the use case needs.

---

# 57. Entity Graph vs DTO Projection

Use EntityGraph when:

```text
You still want entities
Business logic needs managed state
The fetch plan is the main concern
```

Use DTO projection when:

```text
Read-only response
Large result set
Only a few fields required
API/reporting use case
```

---

# 58. Fetch Strategy Decision Tree

```text
Need entity for update?
        |
       Yes
        ↓
Entity query
        |
Need relationship?
        |
       Yes
        ↓
JOIN FETCH / EntityGraph / batch
```

For read-only:

```text
Need only selected fields?
        |
       Yes
        ↓
DTO projection
```

---

# 59. N+1 Solution Decision Tree

If N+1 appears:

```text
Do you need all related data?
        |
       No
        ↓
DTO projection

       Yes
        |
        ↓
Can one join safely fetch it?
        |
       Yes
        ↓
JOIN FETCH / EntityGraph

       No
        |
        ↓
Batch fetching / separate queries
```

---

# 60. Don't Optimize Blindly

Before changing:

```text
LAZY
EAGER
JOIN FETCH
Batch size
Cache
```

measure:

```text
Query count
Latency
Rows returned
Memory
Database CPU
```

---

# 61. Common Mistake

Bad:

```text
@OneToMany(fetch = EAGER)
```

everywhere.

Why?

```text
Unexpected joins
Large object graphs
Memory overhead
Slow endpoints
```

---

# 62. Common Mistake

Bad:

```text
Use JOIN FETCH for every relationship
```

Why?

```text
Huge result sets
Duplicate rows
Cartesian multiplication
Pagination problems
```

---

# 63. Common Mistake

Bad:

```text
Use batch size = 1000
```

without measuring.

Large batches can cause:

```text
Large IN clauses
More memory
Database pressure
```

Tune based on workload.

---

# 64. Common Mistake

Bad:

```text
Fix N+1 by enabling OSIV
```

OSIV may hide the issue rather than solve it.

---

# 65. Common Mistake

Bad:

```text
Fix LazyInitializationException by EAGER
```

Instead:

```text
Define the correct transaction boundary
+
Fetch the required data explicitly
```

---

# 66. Interview Question

### What is lazy loading?

Answer:

> "Lazy loading delays loading an association until it is accessed. It can reduce unnecessary database work, but the association must be accessed while an appropriate persistence context is available or explicitly fetched."

---

# 67. Interview Question

### What is eager loading?

Answer:

> "Eager loading requests that the association be loaded as part of fetching the entity. It can be useful for always-required data but can cause unnecessary database work when overused."

---

# 68. Interview Question

### What is N+1?

Answer:

> "N+1 occurs when one query loads the parent entities and then an additional query is executed for each parent to load related data. For N parents, this can result in N+1 database queries."

---

# 69. Interview Question

### How do you solve N+1?

Answer:

> "Depending on the use case, I can use JOIN FETCH, EntityGraph, batch fetching, DTO projections or separate optimized queries. I choose based on the data required and the size/cardinality of the relationships."

---

# 70. Interview Question

### JOIN FETCH vs normal JOIN?

Answer:

> "A normal join can be used for filtering or query logic, while JOIN FETCH additionally tells JPA to fetch the associated entity as part of the query result."

---

# 71. Interview Question

### What is EntityGraph?

Answer:

> "EntityGraph defines which associations should be fetched for a particular repository operation. It lets us customize the fetch plan without necessarily writing a fetch-join query."

---

# 72. Interview Question

### What is LazyInitializationException?

Answer:

> "It typically occurs when a lazy association is accessed after the entity is no longer attached to an active persistence context. I prefer fixing the fetch plan and transaction boundary instead of making the association globally eager."

---

# 73. Interview Question

### What is batch fetching?

Answer:

> "Batch fetching allows Hibernate to load multiple related entities together instead of executing one query per related entity, which can reduce the impact of N+1."

---

# 74. Interview Question

### Why can JOIN FETCH be dangerous?

Answer:

> "Fetching large collections with joins can multiply result rows, consume memory and cause pagination problems. Fetch joins should be designed around the actual use case."

---

# 75. Interview Question

### Why can multiple collection fetches be problematic?

Answer:

> "Joining multiple collections can produce a cartesian multiplication of rows and, with Hibernate bag mappings, can result in MultipleBagFetchException. I would consider separate queries, batch fetching, DTO projections or redesigning the read model."

---

# 76. Interview Scenario

### "GET /orders is suddenly slow."

Answer approach:

```text
1. Check latency metrics
2. Check trace
3. Count SQL queries
4. Look for N+1
5. Inspect query execution plan
6. Check indexes
7. Check returned row count
8. Review fetch strategy
9. Optimize
10. Add regression test
```

---

# 77. Interview Scenario

### "100 orders cause 101 queries."

Answer:

> "That's a classic N+1 problem. I'd first identify which association is being loaded repeatedly, then choose a solution such as a fetch join, EntityGraph, batch fetching or DTO projection based on the API's actual data requirements."

---

# 78. Interview Scenario

### "The API throws LazyInitializationException."

Answer:

> "I'd check whether the lazy association is accessed outside the persistence context. Then I'd move the required fetch into the service transaction and use an explicit fetch plan or DTO rather than making the association globally eager."

---

# 79. Interview Scenario

### "Fetching Order and Items returns too many rows."

Think:

```text
Join multiplication
```

Check:

```text
Collection size
Duplicate root rows
DISTINCT
Pagination
Separate query
DTO
```

---

# 80. Interview Scenario

### "You need a page of Orders and each Order's Items."

Better design:

```text
Step 1
Get page of Order IDs

Step 2
Fetch Orders + Items
WHERE order.id IN (...)
```

Then assemble the response.

This avoids applying pagination directly to a multiplied collection join.

---

# 81. Interview Scenario

### "You need a dashboard with 20 fields from 5 tables."

Don't automatically load:

```text
5 full entities
```

Consider:

```text
DTO projection
```

that selects only:

```text
20 required fields
```

---

# 82. Interview Scenario

### "Customer has millions of Orders."

Never:

```text
customer.getOrders()
```

and assume the whole collection should be loaded.

Use:

```text
Pagination
Filtering
Projection
Aggregations
```

---

# 83. Production Best Practices

```text
Keep large collections LAZY
Design explicit fetch plans
Monitor SQL query count
Detect N+1 early
Use DTO projections for read-heavy endpoints
Use JOIN FETCH selectively
Use EntityGraph where appropriate
Use batch fetching for suitable access patterns
Avoid multiple collection fetch joins
Paginate large datasets
Prefer keyset pagination for suitable deep-pagination workloads
Keep transaction boundaries clear
Avoid relying on OSIV to hide fetch problems
Inspect execution plans
Index appropriately
Measure before tuning
```

---

# 84. Practical Mental Model

Think:

```text
Entity
  ↓
What data do I actually need?
  ↓
How many rows?
  ↓
How many SQL queries?
  ↓
Can a join safely fetch it?
  ↓
Would a DTO be better?
  ↓
What happens at 10x traffic?
```

---

# 85. Final Interview Answer

If asked:

> "How do you handle Hibernate fetching and avoid performance problems?"

Say:

> "I generally keep large associations lazy and define the fetch plan based on each use case. For N+1 problems, I use JOIN FETCH, EntityGraph, batch fetching or DTO projections depending on the query and relationship cardinality. I avoid making everything eager because that can create large object graphs and unnecessary database work. For large collections I use pagination, and I monitor generated SQL, query counts, execution plans and endpoint latency."

---

# 86. Revision Checklist

```text
□ Lazy loading
□ Eager loading
□ JPA default fetch types
□ Hibernate proxies
□ LazyInitializationException
□ N+1
□ Detecting N+1
□ JOIN FETCH
□ JOIN vs JOIN FETCH
□ DISTINCT
□ EntityGraph
□ DTO projection
□ Batch fetching
□ @BatchSize
□ default_batch_fetch_size
□ SUBSELECT
□ MultipleBagFetchException
□ Cartesian explosion
□ Pagination
□ Offset pagination
□ Keyset pagination
□ Collection fetch + pagination
□ Open Session in View
□ Transaction boundaries
□ Fetch plans
□ Query count
□ Hibernate statistics
□ SQL logging
□ Performance debugging
□ Interview scenarios
```

---

# 87. What Comes Next

```text
File 04 → JPQL, Native SQL & Projections
```

Next we will cover:

```text
JPQL
Named Queries
@Query
Parameters
JOIN
JOIN FETCH
GROUP BY
HAVING
Subqueries
Native SQL
Interface Projections
DTO Projections
Dynamic Queries
Specifications
Criteria API
Query Performance
```

The key interview lesson is:

> **Fetching is a query-design problem, not simply a LAZY-vs-EAGER setting. A strong Hibernate developer chooses what data to fetch, when to fetch it, how many rows/queries it produces, and how that behavior scales under real traffic.**
