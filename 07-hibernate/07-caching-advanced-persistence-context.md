# Hibernate & JPA — File 07: Caching & Advanced Persistence Context

This file covers advanced Hibernate concepts that commonly appear in experienced Java backend interviews.

Core topics:

```text
Persistence Context
First-Level Cache
Second-Level Cache
Query Cache
Cache Providers
Cache Regions
Cache Eviction
Cache Invalidation
Cache Concurrency
Entity States
Managed / Detached / Removed / Transient
persist()
merge()
detach()
clear()
refresh()
flush()
Flush Modes
Dirty Checking
Snapshots
Identity Map
Hibernate Session
EntityManager
Cache vs Redis
Production Cache Design
Interview Questions
Production Scenarios
```

---

# 1. Persistence Context

The persistence context is the set of entity instances currently managed by the JPA `EntityManager`.

Conceptually:

```text
EntityManager
      ↓
Persistence Context
      ↓
Managed Entities
```

Hibernate uses it to:

```text
Track entities
Perform dirty checking
Maintain identity
Coordinate persistence
Avoid duplicate managed instances
```

---

# 2. First-Level Cache

The first-level cache belongs to the persistence context.

It is enabled by default.

Example:

```java
@Transactional
public void process(Long id) {

    Product p1 =
        entityManager.find(Product.class, id);

    Product p2 =
        entityManager.find(Product.class, id);
}
```

Within the same persistence context, Hibernate can return the same managed entity instance rather than issuing another database query.

---

# 3. First-Level Cache Scope

Typically:

```text
EntityManager / persistence context
```

In a normal Spring transaction:

```text
Transaction
   ↓
Persistence Context
```

When the persistence context ends, its first-level cache goes away.

---

# 4. Identity Guarantee

A key concept is:

```text
One persistence context
+
One entity identity
=
One managed object instance
```

Conceptually:

```java
Product p1 = find(Product.class, 10);
Product p2 = find(Product.class, 10);

p1 == p2
```

For the same managed entity identity within the same persistence context, this should be true.

This is sometimes described as the persistence-context identity map.

---

# 5. Why First-Level Cache Exists

It helps with:

```text
Repeated reads
Identity management
Dirty checking
Consistency inside a persistence context
```

Example:

```text
Load Product 10
      ↓
Modify Product 10
      ↓
Load Product 10 again
      ↓
Same managed instance
```

---

# 6. First-Level Cache Is Not Redis

First-level cache:

```text
Local
Per persistence context
Automatic
Short-lived
```

Redis:

```text
External
Shared
Distributed
Application-controlled
```

They solve different problems.

---

# 7. Second-Level Cache

Second-level cache lives beyond an individual persistence context.

Conceptually:

```text
Transaction A
   ↓
Persistence Context
   ↓
Second-Level Cache
   ↓
Database
```

A later persistence context may be able to reuse cached entity data.

---

# 8. Why Use Second-Level Cache?

Potential benefits:

```text
Fewer database reads
Lower latency
Reduced database load
```

Best candidates are often:

```text
Frequently-read
Rarely-changing
Shared reference data
```

Examples:

```text
Country
Currency
Product category
Application configuration
```

---

# 9. Second-Level Cache Is Optional

Unlike first-level cache:

```text
Second-level cache
```

is not automatically enabled as a universal JPA requirement.

It requires:

```text
Hibernate configuration
+
Cache provider
+
Cache strategy
```

---

# 10. Cache Provider

Hibernate can integrate with caching providers.

Modern Hibernate applications commonly use a provider such as:

```text
JCache-compatible provider
```

The exact provider and configuration depend on the Hibernate version and application architecture.

---

# 11. Cache Region

A cache region is a logical area where cached data is stored.

Examples:

```text
Product region
Category region
Query region
Collection region
```

This allows different cache policies to be configured for different data.

---

# 12. Entity Cache

An entity can be marked cacheable.

Example:

```java
@Entity
@Cacheable
public class Category {

    @Id
    private Long id;

    private String name;
}
```

Hibernate can then use the configured second-level cache for the entity.

---

# 13. Collection Cache

Associations can also be cached in Hibernate-specific configurations.

Example concept:

```text
Category
  ↓
Products
```

Caching the entity does not automatically mean every collection associated with it is cached.

The collection cache has its own considerations.

---

# 14. Cache vs Database

Think:

```text
Application
    ↓
First-Level Cache
    ↓
Second-Level Cache
    ↓
Database
```

The higher the cache hit rate:

```text
Less database traffic
```

But caching introduces:

```text
Staleness
Invalidation
Memory
Complexity
```

---

# 15. Cache Invalidation

One of the hardest cache problems is:

```text
When should cached data be removed or updated?
```

Example:

```text
Product price = 100
```

Cache:

```text
price = 100
```

Database:

```text
price = 120
```

If the cache still returns:

```text
100
```

the application sees stale data.

---

# 16. Cache Invalidation Strategies

Common approaches include:

```text
Invalidate on update
Expire after TTL
Update cache on write
Evict manually
Version data
```

The correct strategy depends on consistency requirements.

---

# 17. Cache Eviction

Eviction means removing data from the cache.

Reasons:

```text
Data changed
TTL expired
Memory pressure
Manual invalidation
Deployment
Administrative action
```

---

# 18. TTL

TTL means:

```text
Time To Live
```

Example:

```text
Cache entry created
 ↓
TTL = 10 minutes
 ↓
Entry expires
```

TTL is useful for data where slightly stale values are acceptable.

---

# 19. Cache Hit

A cache hit means:

```text
Requested data
 ↓
Found in cache
```

No database query may be required.

---

# 20. Cache Miss

A cache miss means:

```text
Requested data
 ↓
Not in cache
 ↓
Database
 ↓
Potentially populate cache
```

---

# 21. Cache Hit Rate

A useful metric:

```text
Hit rate =
cache hits / total cache requests
```

Example:

```text
900 hits
100 misses
```

Hit rate:

```text
90%
```

A cache with a very low hit rate may not provide enough benefit to justify its complexity.

---

# 22. Cache Stampede

Suppose:

```text
Popular cache entry expires
```

At the same moment:

```text
1,000 requests
```

all miss the cache.

They may all query the database:

```text
1,000 requests
 ↓
1,000 DB queries
```

This is a cache stampede.

---

# 23. Preventing Cache Stampede

Possible techniques:

```text
Request coalescing
Locking
Jittered expiration
Refresh-ahead
Stale-while-revalidate
Prewarming
```

The appropriate approach depends on the cache architecture.

---

# 24. Cache Penetration

Cache penetration occurs when requests repeatedly ask for data that doesn't exist.

Example:

```text
GET product/999999999
```

If every request:

```text
Cache miss
 ↓
Database
 ↓
No record
```

the database can receive unnecessary load.

---

# 25. Preventing Cache Penetration

Possible approach:

```text
Negative caching
```

For example:

```text
product/999999999
→ NOT_FOUND
```

with a short TTL.

Be careful not to cache temporary failures as permanent absence.

---

# 26. Cache Avalanche

A cache avalanche can happen when many entries expire or become unavailable around the same time.

Possible result:

```text
Huge cache miss
 ↓
Database overload
```

Mitigation:

```text
Jitter TTLs
Stagger expiration
Warm cache
Use resilient fallback strategies
```

---

# 27. Hibernate Query Cache

Hibernate can also cache query results.

Conceptually:

```text
Query
 ↓
Query Cache
 ↓
IDs/results
 ↓
Entity Cache / Database
```

Query caching is more complicated than simply caching an entity.

---

# 28. Why Query Cache Can Be Tricky

Suppose:

```text
Query:
Products WHERE status = ACTIVE
```

If product status changes:

```text
Old query result
```

may no longer be valid.

Hibernate must manage invalidation based on affected data regions.

---

# 29. Query Cache Is Not Always Necessary

Before enabling it, ask:

```text
Is the query repeated frequently?
Is the result relatively stable?
Is cache invalidation manageable?
Does measurement show database pressure?
```

Do not enable query cache just because it exists.

---

# 30. Cache Concurrency

Hibernate supports different cache concurrency strategies/provider behaviors.

The conceptual goals are:

```text
Prevent stale/incorrect cache behavior
Handle concurrent reads/writes
Balance consistency and performance
```

The exact available strategy names and guarantees depend on Hibernate/provider configuration.

---

# 31. Entity States

JPA entities commonly move through these states:

```text
Transient
   ↓
Managed
   ↓
Detached
   ↓
Removed
```

Understanding entity state is essential for Hibernate interviews.

---

# 32. Transient

A newly created object that is not associated with a persistence context:

```java
Product product = new Product();
product.setName("Laptop");
```

At this point:

```text
Transient
```

It has not been persisted.

---

# 33. Managed

After:

```java
entityManager.persist(product);
```

the entity becomes managed.

Conceptually:

```text
Transient
   ↓ persist()
Managed
```

Hibernate tracks changes to it.

---

# 34. Dirty Checking

Suppose:

```java
Product product =
    entityManager.find(Product.class, id);

product.setPrice(120);
```

No explicit:

```java
save(product);
```

may be necessary.

If the entity is managed:

```text
Hibernate detects change
 ↓
Dirty checking
 ↓
UPDATE
```

during flush.

---

# 35. How Dirty Checking Works Conceptually

Hibernate maintains state information for managed entities.

Conceptually:

```text
Original state
      ↓
Current state
      ↓
Compare
      ↓
Changes detected
      ↓
SQL UPDATE
```

The exact implementation is more sophisticated than a simple Java object comparison.

---

# 36. Dirty Checking Cost

If the persistence context contains:

```text
10 entities
```

tracking is easy.

If it contains:

```text
500,000 entities
```

dirty checking and memory overhead can become significant.

This is another reason to keep persistence contexts bounded.

---

# 37. Detached

An entity becomes detached when it is no longer managed by the persistence context.

Example:

```java
entityManager.detach(product);
```

Or when:

```text
Persistence context ends
```

the entity is no longer managed by that context.

---

# 38. Detached Entity

Example:

```text
Managed entity
   ↓
Transaction ends
   ↓
Detached entity
```

Changing it:

```java
product.setPrice(150);
```

does not automatically cause Hibernate to update the database.

---

# 39. merge()

`merge()` copies the state of a detached entity into a managed instance.

Example:

```java
Product managed =
    entityManager.merge(detachedProduct);
```

Important:

> `merge()` does not simply "reattach the same object."

It returns a managed instance.

---

# 40. merge() Example

Conceptually:

```text
Detached Product A
       ↓ merge()
Managed Product B
```

The object returned by `merge()` is the one you should generally continue working with as the managed instance.

---

# 41. Common merge Mistake

Bad:

```java
entityManager.merge(product);

product.setPrice(200);
```

The original `product` may still be detached.

Better:

```java
product =
    entityManager.merge(product);

product.setPrice(200);
```

when that behavior is appropriate.

---

# 42. persist() vs merge()

Remember:

```text
persist()
→ makes a new/transient entity managed

merge()
→ copies state into a managed entity
```

They are not interchangeable.

---

# 43. detach()

Example:

```java
entityManager.detach(product);
```

Now:

```text
product
→ detached
```

Changes are no longer automatically tracked.

---

# 44. clear()

Example:

```java
entityManager.clear();
```

This detaches all currently managed entities from the persistence context.

Conceptually:

```text
Persistence Context
 ↓ clear()
Empty Persistence Context
```

---

# 45. refresh()

Example:

```java
entityManager.refresh(product);
```

This reloads the entity state from the database.

Useful when:

```text
Database changed externally
Need latest DB state
Trigger/generated values need to be refreshed
```

Use carefully because local in-memory changes can be overwritten.

---

# 46. flush()

Example:

```java
entityManager.flush();
```

This synchronizes pending changes with the database.

It does not necessarily mean:

```text
Transaction committed
```

Remember:

```text
flush ≠ commit
```

---

# 47. Flush Modes

JPA/Hibernate supports different flush strategies.

Common concepts include:

```text
AUTO
COMMIT
MANUAL / provider-specific behavior
```

Exact options and behavior depend on JPA/Hibernate version and API being used.

---

# 48. AUTO Flush

With typical `AUTO` behavior, Hibernate may flush pending changes before operations where synchronization is required.

Example:

```text
Update Product
 ↓
Execute query affected by Product
 ↓
Hibernate may flush
 ↓
Execute query
```

---

# 49. Why Flush Before Query?

Suppose:

```java
product.setPrice(500);

queryProductsWithPriceGreaterThan(400);
```

If the query depends on the changed state, Hibernate may need to synchronize pending changes before executing it.

This helps keep query results consistent with the current persistence context semantics.

---

# 50. Flush Is Not Save

Another interview trap:

```text
flush()
```

doesn't mean:

```text
commit()
```

A later rollback can still undo flushed database changes.

---

# 51. Persistence Context and Transactions

Typical Spring flow:

```text
@Transactional
      ↓
EntityManager
      ↓
Persistence Context
      ↓
Managed entities
      ↓
Flush
      ↓
Commit
```

---

# 52. First-Level Cache vs Second-Level Cache

| Feature | First-Level | Second-Level |
|---|---|---|
| Scope | Persistence context | SessionFactory/entity-manager factory |
| Default | Yes | Optional |
| Lifetime | Short | Longer |
| Shared across transactions | No | Potentially |
| Main purpose | Identity + tracking | Reduce DB reads |

---

# 53. Hibernate Cache vs Redis

Hibernate second-level cache:

```text
Entity/ORM aware
```

Redis:

```text
Application-controlled
Distributed
General-purpose
```

Use Redis when you need:

```text
Cross-service/shared cache
Custom keys
Counters
Rate limiting
Session data
Distributed coordination
```

Use Hibernate cache when:

```text
Entity-level caching naturally fits the ORM workload
```

---

# 54. Cache Consistency

Ask:

```text
How stale can data be?
```

Examples:

```text
Country list
→ minutes/hours may be acceptable

Inventory
→ stale data may be dangerous

Payment status
→ strong correctness requirements
```

Caching strategy depends on business semantics.

---

# 55. Don't Cache Everything

Caching increases:

```text
Complexity
Memory usage
Operational overhead
Invalidation requirements
```

Only cache data that benefits from it.

---

# 56. Cache Aside

A common application-level pattern:

```text
Read
 ↓
Check cache
 ↓
Hit → return
Miss
 ↓
Read DB
 ↓
Put in cache
 ↓
Return
```

Redis commonly uses this pattern.

---

# 57. Write-Through

Conceptually:

```text
Application
 ↓
Cache
 ↓
Database
```

The cache is updated as part of the write path.

This can simplify read freshness in some designs but introduces additional infrastructure/consistency considerations.

---

# 58. Write-Behind

Conceptually:

```text
Application
 ↓
Cache
 ↓
Later DB write
```

This can improve write performance but is much more complex and can risk data loss if not carefully designed.

Use only when the architecture explicitly supports the required durability semantics.

---

# 59. Persistence Context and Serialization

Avoid returning managed entities directly from controllers.

Why?

```text
Lazy loading
Circular relationships
Unexpected SQL
Huge JSON graphs
```

Prefer:

```text
Entity
 ↓
DTO
 ↓
JSON
```

---

# 60. Persistence Context and Threads

A persistence context is not something you should casually share across application threads.

Avoid:

```text
Thread A
   ↓
EntityManager

Thread B
   ↓
Same EntityManager
```

Use transaction/request-scoped persistence appropriately.

---

# 61. Async Processing

If an entity is loaded in one transaction and passed to an asynchronous task:

```text
Transaction ends
 ↓
Entity detached
 ↓
Async task accesses lazy association
 ↓
Potential LazyInitializationException
```

Instead pass:

```text
IDs
DTOs
immutable data
```

and load required data inside the async transaction.

---

# 62. Entity State Mental Model

Remember:

```text
new Entity()
     ↓
Transient
     ↓ persist()
Managed
     ↓ detach()/clear()/transaction end
Detached
     ↓ merge()
Managed copy
```

Removal:

```text
Managed
   ↓ remove()
Removed
```

---

# 63. remove()

Example:

```java
entityManager.remove(product);
```

The entity becomes marked for deletion.

The actual SQL DELETE is typically executed during flush.

---

# 64. remove() and Detached Entities

`remove()` generally operates on a managed entity.

If you have a detached object:

```text
Detached
 ↓
merge
 ↓
Managed
 ↓
remove
```

may be necessary depending on the use case.

---

# 65. EntityManager vs Hibernate Session

JPA standard API:

```text
EntityManager
```

Hibernate native API:

```text
Session
```

Spring Data JPA typically works through JPA abstractions while Hibernate is commonly the underlying provider.

---

# 66. Why Prefer EntityManager?

Advantages:

```text
Standard JPA API
Portability
Clear abstraction
```

Use Hibernate-specific APIs only when a real provider-specific feature is needed.

---

# 67. Hibernate-Specific Features

Hibernate provides additional functionality beyond standard JPA.

Examples may include:

```text
Advanced caching
Provider-specific fetch modes
StatelessSession
Hibernate-specific annotations
Provider-specific query features
```

Know when you're leaving the JPA standard.

---

# 68. StatelessSession

Hibernate provides:

```text
StatelessSession
```

which has a much simpler persistence model and does not maintain the normal first-level persistence context/dirty checking behavior.

It can be useful for specialized high-volume operations.

Trade-offs include:

```text
No normal persistence-context behavior
No normal dirty checking
Limited lifecycle semantics
```

Use only when the workload justifies it.

---

# 69. Advanced Batch Processing

For very large jobs:

```text
Read chunk
 ↓
Process
 ↓
Write chunk
 ↓
flush
 ↓
clear
 ↓
Next chunk
```

This is usually safer than:

```text
Load entire dataset
 ↓
Process everything
 ↓
Commit
```

---

# 70. Read-Only Processing

If you only need report data:

```text
DTO projection
```

is often preferable to:

```text
Load 500,000 managed entities
```

This reduces:

```text
Memory
Dirty checking
Persistence-context overhead
```

---

# 71. Cache Invalidation and Transactions

A common design question:

```text
Update DB
+
Invalidate cache
```

What if:

```text
DB commit succeeds
cache invalidation fails
```

Now:

```text
Cache stale
```

Reliable cache invalidation requires careful sequencing, retries, events or an outbox depending on the architecture.

---

# 72. Transaction + Cache Pattern

A safer conceptual flow can be:

```text
DB transaction
 ↓
Commit
 ↓
Publish invalidation event
 ↓
Cache invalidated
```

This avoids invalidating before a transaction that may eventually roll back.

---

# 73. Cache Failure

Never assume cache is always available.

A resilient application should decide:

```text
Cache unavailable
 ↓
Can we read DB?
```

For many caching use cases:

```text
Cache failure
→ fallback to database
```

But this must be designed carefully to avoid turning a cache outage into a database overload event.

---

# 74. Cache Outage Scenario

Suppose:

```text
Redis unavailable
```

and:

```text
10,000 requests/sec
```

all fall back to the database.

Result:

```text
Database overload
```

This is why resilient caching needs:

```text
Rate limiting
Circuit breakers
Load shedding
Request coalescing
Capacity planning
```

where appropriate.

---

# 75. Interview Question

### What is the first-level cache?

Answer:

> "The first-level cache is the persistence context associated with an EntityManager or Hibernate Session. It is enabled by default and ensures managed entities are tracked and reused within that persistence context."

---

# 76. Interview Question

### What is the second-level cache?

Answer:

> "The second-level cache is an optional Hibernate cache shared beyond an individual persistence context. It can reduce database reads across transactions when the data and consistency requirements make caching appropriate."

---

# 77. Interview Question

### First-level vs second-level cache?

Answer:

> "First-level cache is scoped to a persistence context and is always present in normal JPA usage. Second-level cache is optional and can be shared across persistence contexts through a configured cache provider."

---

# 78. Interview Question

### What is dirty checking?

Answer:

> "Dirty checking is Hibernate's mechanism for detecting changes made to managed entities and generating the required SQL during flush."

---

# 79. Interview Question

### persist vs merge?

Answer:

> "`persist()` makes a new entity managed, while `merge()` copies the state of a detached entity into a managed instance and returns that managed instance."

---

# 80. Interview Question

### detach vs clear?

Answer:

> "`detach()` removes one entity from the persistence context, while `clear()` removes all currently managed entities from the persistence context."

---

# 81. Interview Question

### What does refresh() do?

Answer:

> "`refresh()` reloads the entity state from the database, potentially overwriting local in-memory changes."

---

# 82. Interview Question

### Does flush mean commit?

Answer:

> "No. Flush synchronizes pending changes with the database, while commit completes the transaction. A flushed change can still be rolled back before commit."

---

# 83. Interview Question

### Should every entity be cached?

Answer:

> "No. I would cache frequently-read, relatively stable data where the performance benefit justifies the memory and invalidation complexity. Highly volatile data such as inventory often needs different consistency considerations."

---

# 84. Interview Question

### Hibernate cache vs Redis?

Answer:

> "Hibernate's second-level cache is ORM/entity-oriented, while Redis is an application-level distributed cache. I use Hibernate caching when entity caching fits naturally, and Redis when I need shared application caching or patterns such as counters, rate limiting or custom keys."

---

# 85. Interview Scenario

### "The same entity is loaded twice in one transaction."

Think:

```text
First-level cache
```

If it is the same managed identity:

```text
Hibernate can reuse the managed instance.
```

---

# 86. Interview Scenario

### "A detached entity was changed but the database did not update."

Likely:

```text
Entity is detached.
```

Possible solution:

```text
Load managed entity
or
merge detached state
```

depending on the use case.

---

# 87. Interview Scenario

### "After a bulk update, an entity still shows its old value."

Likely:

```text
Persistence context contains stale state.
```

Consider:

```text
flush
+
clear
```

or refreshing/reloading the entity.

---

# 88. Interview Scenario

### "Calling clear() made unsaved changes disappear."

Reason:

```text
clear()
→ detaches managed entities.
```

If changes were not flushed before clearing, they may no longer be tracked.

Remember:

```text
flush()
 ↓
clear()
```

for appropriate batch-processing workflows.

---

# 89. Interview Scenario

### "Cache returns old product price."

Investigate:

```text
Cache invalidation
TTL
Update flow
Multiple cache layers
Replication
Stale entries
```

Don't immediately blame Hibernate.

---

# 90. Interview Scenario

### "Redis goes down and database CPU spikes."

Likely:

```text
Cache fallback storm
```

Consider:

```text
Circuit breaker
Rate limiting
Request coalescing
Load shedding
Cache recovery/warm-up
```

---

# 91. Production Best Practices

```text
Keep persistence contexts bounded
Use DTOs for large read workloads
Understand entity state
Use flush/clear for large batch processing
Avoid accidental lazy loading
Cache only suitable data
Monitor cache hit rates
Design invalidation explicitly
Don't cache highly volatile data blindly
Monitor DB and cache together
Use transactions with clear boundaries
Don't share persistence contexts across threads
```

---

# 92. Advanced Mental Model

Think of Hibernate as:

```text
Persistence Context
        ↓
Identity + Dirty Checking
        ↓
First-Level Cache
        ↓
Optional Second-Level Cache
        ↓
JDBC
        ↓
Database
```

For every operation ask:

```text
Is this entity managed?
Is it cached?
Will SQL execute?
When will SQL execute?
What happens at flush?
What happens at commit?
```

---

# 93. Entity State Decision Tree

```text
Do I have a new object?
       |
      Yes
       ↓
    persist()

Do I have a detached object?
       |
      Yes
       ↓
     merge()

Need to remove one?
       |
      Yes
       ↓
     remove()

Need to detach one?
       |
      Yes
       ↓
    detach()

Need to detach all?
       |
      Yes
       ↓
     clear()

Need latest DB state?
       |
      Yes
       ↓
    refresh()
```

---

# 94. Cache Decision Tree

```text
Is data frequently read?
       |
      No
       ↓
Don't cache

      Yes
       ↓
Does stale data matter?
       |
      Yes
       ↓
Design invalidation carefully

      No
       ↓
TTL/cache-aside may work
```

---

# 95. Final Interview Answer

If asked:

> "Explain Hibernate caching and persistence context."

Say:

> "Hibernate maintains a first-level cache inside the persistence context, which tracks managed entities and supports identity and dirty checking. Hibernate can also use an optional second-level cache shared across persistence contexts. I don't cache everything; I choose relatively stable, frequently-read data where the performance benefit justifies invalidation and consistency complexity. I also understand entity states such as transient, managed and detached, because operations like persist, merge, clear and flush behave differently depending on that state."

---

# 96. Revision Checklist

```text
□ Persistence context
□ First-level cache
□ Identity map
□ Second-level cache
□ Entity cache
□ Collection cache
□ Cache regions
□ Cache provider
□ Query cache
□ Cache hit
□ Cache miss
□ Cache invalidation
□ Cache eviction
□ TTL
□ Cache stampede
□ Cache penetration
□ Cache avalanche
□ Cache concurrency
□ Entity states
□ Transient
□ Managed
□ Detached
□ Removed
□ persist()
□ merge()
□ detach()
□ clear()
□ refresh()
□ flush()
□ Dirty checking
□ Snapshots
□ Flush modes
□ EntityManager
□ Hibernate Session
□ StatelessSession
□ Hibernate vs Redis
□ Cache consistency
□ Cache fallback
□ Cache outage
□ Transaction + cache
□ Interview scenarios
```

---

# 97. What Comes Next

```text
File 08 → Hibernate Advanced Mappings & Real-World Patterns
```

Next we will cover:

```text
Inheritance
SINGLE_TABLE
JOINED
TABLE_PER_CLASS
@MappedSuperclass
Embeddables
Composite Keys
@MapsId
Auditing
@CreatedDate
@LastModifiedDate
Soft Delete
@SQLRestriction / Hibernate-specific filtering
Entity Listeners
Enums
Custom Types
AttributeConverter
JSON columns
UUIDs
Natural IDs
Database constraints
Schema design
Real-world e-commerce mappings
Interview scenarios
```

The key interview lesson is:

> **A strong Hibernate developer understands not only queries, but also the persistence context and entity lifecycle. Once you understand managed state, dirty checking, flushing and caching, Hibernate's behavior becomes much easier to predict and troubleshoot.**
