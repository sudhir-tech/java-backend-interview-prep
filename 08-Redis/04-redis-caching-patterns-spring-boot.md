# Redis — File 04: Caching Patterns & Spring Boot Integration

This file connects Redis concepts to real Java backend development with Spring Boot.

Core topics:

```text
Spring Boot + Redis
RedisTemplate
StringRedisTemplate
Spring Cache
@Cacheable
@CachePut
@CacheEvict
CacheManager
Serialization
JSON
Cache-aside
Write-through
Write-behind
Read-through
TTL
Cache invalidation
E-commerce examples
Production patterns
```

---

# 1. Redis in a Spring Boot Application

A common architecture is:

```text
Client
  ↓
Controller
  ↓
Service
  ↓
Redis Cache
  ↓ cache miss
Repository
  ↓
MySQL
```

Redis is normally introduced at the service/read layer rather than directly exposing Redis to controllers.

---

# 2. Why Put Caching in the Service Layer?

The service layer understands:

```text
Business operation
Data ownership
Cacheability
Invalidation
Transactions
```

Example:

```text
ProductService
    ↓
getProduct()
    ↓
Redis
    ↓
ProductRepository
```

This keeps caching concerns away from HTTP/controller code.

---

# 3. Spring Boot Redis Dependency

For a Spring Boot application using Spring Data Redis, a typical Maven dependency is:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

The exact dependency version is managed by the Spring Boot dependency management.

---

# 4. Redis Connection

Spring Boot can configure Redis using application properties.

Example:

```properties
spring.data.redis.host=localhost
spring.data.redis.port=6379
```

For a secured remote Redis instance, additional settings may include:

```text
Username
Password
TLS/SSL
Connection configuration
```

Use environment variables/secrets rather than hardcoding credentials.

---

# 5. Typical Local Setup

Local architecture:

```text
Spring Boot
localhost
   ↓
Redis
localhost:6379
```

The application sends Redis commands through a Spring Data Redis client.

---

# 6. RedisTemplate

`RedisTemplate` provides a high-level API for interacting with Redis.

Example:

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;
```

Common operations are exposed through:

```text
opsForValue()
opsForHash()
opsForList()
opsForSet()
opsForZSet()
```

---

# 7. StringRedisTemplate

For string-based Redis operations:

```java
private final StringRedisTemplate redisTemplate;
```

Example:

```java
redisTemplate.opsForValue()
             .set("user:101", "Sudhir");
```

Then:

```java
String value =
    redisTemplate.opsForValue()
                .get("user:101");
```

---

# 8. RedisTemplate Data Structure APIs

Typical mapping:

```text
opsForValue()
→ Strings

opsForHash()
→ Hashes

opsForList()
→ Lists

opsForSet()
→ Sets

opsForZSet()
→ Sorted Sets
```

Streams have dedicated Redis APIs as well.

---

# 9. Basic String Example

```java
redisTemplate.opsForValue()
             .set("product:101", "Laptop");
```

Read:

```java
String product =
    redisTemplate.opsForValue()
                .get("product:101");
```

---

# 10. Set With Expiration

Spring Data Redis can set expiration when writing:

```java
redisTemplate.opsForValue()
             .set(
                 "session:abc",
                 "user-data",
                 Duration.ofMinutes(30)
             );
```

This is useful for:

```text
Sessions
Temporary state
Cached values
Tokens
```

---

# 11. Hash Example

```java
redisTemplate.opsForHash()
             .put("user:101", "name", "Sudhir");

redisTemplate.opsForHash()
             .put("user:101", "role", "USER");
```

Read:

```java
Object role =
    redisTemplate.opsForHash()
                .get("user:101", "role");
```

---

# 12. Set Example

```java
redisTemplate.opsForSet()
             .add(
                 "user:101:roles",
                 "USER",
                 "ADMIN"
             );
```

Membership:

```java
redisTemplate.opsForSet()
             .isMember(
                 "user:101:roles",
                 "ADMIN"
             );
```

---

# 13. Sorted Set Example

```java
redisTemplate.opsForZSet()
             .add(
                 "leaderboard",
                 "user:101",
                 100
             );
```

This maps naturally to:

```text
Sorted Set
member + score
```

---

# 14. CacheManager

Spring's cache abstraction provides:

```text
CacheManager
```

It lets application code work with:

```text
@Cacheable
@CachePut
@CacheEvict
```

without directly writing Redis commands for every cache operation.

---

# 15. Enable Spring Caching

Use:

```java
@EnableCaching
@SpringBootApplication
public class Application {
}
```

This enables Spring's cache abstraction.

---

# 16. @Cacheable

Example:

```java
@Cacheable("products")
public Product getProduct(Long id) {
    return repository.findById(id)
                     .orElseThrow();
}
```

Conceptually:

```text
getProduct(101)
      ↓
Cache lookup
      ↓
Hit?
 /        \
Yes        No
 |          |
Return     DB
            ↓
          Cache
            ↓
          Return
```

---

# 17. Why @Cacheable Is Powerful

Without cache abstraction:

```java
if (redis.contains(key)) {
    return redis.get(key);
}

Product p = repository.findById(id);
redis.set(key, p);
return p;
```

With:

```java
@Cacheable
```

Spring handles much of the cache lookup/population behavior.

---

# 18. @Cacheable Key

By default, Spring generates a cache key based on method arguments.

For clarity, explicitly define it when needed:

```java
@Cacheable(
    value = "products",
    key = "#id"
)
public Product getProduct(Long id) {
    ...
}
```

---

# 19. Custom Key

Example:

```java
@Cacheable(
    value = "products",
    key = "'product:' + #id"
)
```

This creates a predictable logical key.

However, remember that Spring's cache infrastructure may apply its own cache-name/key serialization or prefixing depending on configuration.

---

# 20. Key Naming Strategy

A good logical design is:

```text
products
+
id
```

or:

```text
product:{id}
```

For distributed systems, consistent naming helps:

```text
product:101
product:102
```

Avoid putting unnecessary information into keys.

---

# 21. @CachePut

`@CachePut` always executes the method and updates the cache with the result.

Example:

```java
@CachePut(
    value = "products",
    key = "#product.id"
)
public Product updateProduct(Product product) {
    return repository.save(product);
}
```

Conceptually:

```text
Execute method
 ↓
Update database
 ↓
Put returned value into cache
```

---

# 22. @Cacheable vs @CachePut

```text
@Cacheable
→ Cache hit can skip method execution

@CachePut
→ Method always executes
→ Result updates cache
```

This distinction is a common interview question.

---

# 23. @CacheEvict

Used to remove cached data.

Example:

```java
@CacheEvict(
    value = "products",
    key = "#id"
)
public void deleteProduct(Long id) {
    repository.deleteById(id);
}
```

Conceptually:

```text
Delete DB record
+
Remove cache entry
```

---

# 24. allEntries

You can evict an entire cache:

```java
@CacheEvict(
    value = "products",
    allEntries = true
)
```

Be careful.

If the cache contains:

```text
1 million entries
```

clearing all entries can create a large cache miss wave.

---

# 25. beforeInvocation

`@CacheEvict` supports:

```java
beforeInvocation = true
```

Normally eviction occurs after successful method invocation.

With:

```text
beforeInvocation = true
```

the eviction occurs before the method executes.

Use this only when the semantics genuinely require it.

---

# 26. @Caching

Multiple cache operations can be grouped:

```java
@Caching(
    evict = {
        @CacheEvict(...),
        @CacheEvict(...)
    }
)
```

Useful when one business operation affects multiple cache entries.

---

# 27. Conditional Caching

You can conditionally cache:

```java
@Cacheable(
    value = "products",
    condition = "#id > 0"
)
```

You can also use:

```text
unless
```

to avoid caching certain results.

---

# 28. Example: Don't Cache Null Results

Depending on configuration, null values may be cached by Spring's cache abstraction.

You can define conditions such as:

```java
unless = "#result == null"
```

if null results should not be cached.

This should be chosen intentionally.

---

# 29. Negative Caching

Sometimes caching a "not found" result is useful.

Example:

```text
product:999
→ NOT_FOUND
```

with a short TTL.

This protects the database from repeated requests for nonexistent records.

But if the product can be created shortly afterward, use a short TTL.

---

# 30. Cache-Aside Pattern

The application manages:

```text
Cache read
Cache miss
Database read
Cache population
Cache invalidation
```

Spring's `@Cacheable` often implements the read side conveniently.

Conceptually:

```text
Application
 ↓
Cache
 ↓ miss
Database
 ↓
Cache
```

---

# 31. Write-Through

Conceptually:

```text
Application
 ↓
Cache
 ↓
Database
```

The cache layer participates in the write path.

The important idea is:

```text
Write cache
+
write backing store
```

The exact implementation depends on the cache technology and architecture.

---

# 32. Write-Behind

Conceptually:

```text
Application
 ↓
Cache
 ↓
Later asynchronous write
 ↓
Database
```

Advantages:

```text
Fast writes
Batching opportunities
```

Risks:

```text
Data loss
Complex consistency
Recovery complexity
Ordering problems
```

Use only when the business can tolerate the semantics.

---

# 33. Read-Through

Conceptually:

```text
Application
 ↓
Cache
 ↓ miss
Cache loads data from source
 ↓
Return
```

The cache abstraction/provider handles the backing-store lookup.

This is different from cache-aside because the application may not directly perform the database fallback.

---

# 34. Pattern Comparison

```text
Cache-aside
→ Application manages cache + DB

Read-through
→ Cache layer loads source on miss

Write-through
→ Cache participates in writes

Write-behind
→ Cache accepts writes and persists later
```

For typical Spring Boot + Redis applications:

```text
Cache-aside
```

is one of the most common patterns.

---

# 35. Cache Invalidation

For update operations:

```text
Update DB
 ↓
Invalidate cache
```

Then:

```text
Next read
 ↓
Cache miss
 ↓
Fresh DB value
 ↓
Repopulate cache
```

This is often easier to reason about than maintaining two independent write paths.

---

# 36. Database-First Update

A common strategy:

```text
@Transactional
    ↓
Update DB
    ↓
Commit
    ↓
Evict cache
```

The exact ordering and transaction integration need careful consideration.

---

# 37. Why Invalidation Is Hard

Suppose:

```text
Thread A
updates DB

Thread B
reads cache
```

Depending on timing:

```text
B may read old cached data
```

Therefore cache consistency is a distributed-systems problem, not merely an annotation problem.

---

# 38. Race Example

Initial:

```text
DB = 100
Cache = 100
```

Thread A:

```text
Update DB → 120
```

Thread B:

```text
Read cache → 100
```

Thread A:

```text
Evict cache
```

The exact sequence can produce temporary stale reads.

If strong consistency is required, caching strategy must be designed around the business requirement.

---

# 39. Transaction + Cache Trap

A dangerous assumption:

```text
@Transactional
+
@CacheEvict
=
perfect cache consistency
```

Not necessarily.

Database transaction and Redis are different resources.

Without a distributed transaction protocol:

```text
DB commit
+
Redis operation
```

are not one atomic transaction.

---

# 40. Failure Scenario

Suppose:

```text
DB update succeeds
Redis eviction fails
```

Then:

```text
DB = new value
Redis = old value
```

TTL can eventually correct this, but there may be stale reads.

This is why:

```text
TTL
+
invalidation
+
retry/outbox/event strategy
```

may be considered for important workloads.

---

# 41. Event-Based Invalidation

Architecture:

```text
Service
 ↓
DB transaction
 ↓
Outbox/event
 ↓
Cache invalidation consumer
 ↓
Redis DEL
```

This can make invalidation more reliable in distributed systems.

The Outbox Pattern will be covered more deeply in the microservices/system-design section.

---

# 42. Cache Serialization

Spring needs to convert Java objects into Redis-compatible data.

Conceptually:

```text
Java Object
 ↓
Serializer
 ↓
Redis bytes
```

Read:

```text
Redis bytes
 ↓
Deserializer
 ↓
Java Object
```

---

# 43. Why Serialization Matters

Suppose:

```text
Application v1
```

stores:

```text
Product object
```

Then:

```text
Application v2
```

changes the class structure.

Old cache data may become:

```text
Incompatible
```

Possible result:

```text
DeserializationException
```

---

# 44. JSON Serialization

JSON is commonly used because it is:

```text
Readable
Interoperable
Easy to inspect
Less coupled to Java internals
```

But JSON has:

```text
More bytes
Serialization/deserialization cost
```

compared with some binary formats.

---

# 45. Generic JSON Serialization

A Redis cache may store:

```json
{
  "id": 101,
  "name": "Laptop",
  "price": 50000
}
```

This is easier to inspect than opaque Java-specific binary data.

Security and type metadata must still be handled carefully.

---

# 46. Cache Versioning

Use versioned keys:

```text
product:v1:101
```

Then deploy:

```text
product:v2:101
```

This can avoid incompatible old cache entries during rolling deployments.

---

# 47. Redis Cache Configuration

Spring Boot can configure a Redis-backed `CacheManager`.

A typical application can define:

```text
CacheManager
 ↓
Redis
```

and configure:

```text
TTL
Key prefix
Serialization
Null caching
```

The exact configuration API varies somewhat by Spring Boot/Spring Data Redis version.

---

# 48. Cache TTL Configuration

Conceptually:

```text
products → 10 minutes
sessions → 30 minutes
reference-data → 1 hour
```

Different caches can have different TTLs.

Don't use one global TTL for unrelated business data unless the requirement genuinely supports it.

---

# 49. Example Cache Configuration

Conceptually:

```java
RedisCacheConfiguration config =
    RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(10));
```

Then configure a Redis-backed cache manager.

The exact bean configuration should follow the Spring Boot/Spring Data Redis version used by the project.

---

# 50. Cache Key Prefixes

A useful design:

```text
products::101
users::101
orders::101
```

The exact delimiter is not important.

The goal is:

```text
Namespace isolation
```

Spring's cache abstraction can also add cache-name prefixes automatically.

---

# 51. Cache Manager

Conceptually:

```text
CacheManager
 ├── products
 ├── users
 └── orders
```

Each cache can have its own:

```text
TTL
Serialization
Naming
```

depending on configuration.

---

# 52. Spring Cache vs RedisTemplate

### Spring Cache

Best when:

```text
Simple caching
Method-level caching
Cache-aside
```

Example:

```java
@Cacheable
```

### RedisTemplate

Best when:

```text
Custom Redis data structures
Counters
Sets
Sorted sets
Streams
Distributed locks
Custom operations
```

---

# 53. Interview Comparison

Question:

> RedisTemplate vs Spring Cache?

Answer:

> "Spring Cache is an abstraction for method-level caching and is convenient for common cache-aside use cases. RedisTemplate gives lower-level access to Redis data structures and commands, so I would use it when I need Redis-specific operations such as sets, sorted sets, counters, streams or custom atomic logic."

---

# 54. Cacheable Example

```java
@Service
public class ProductService {

    @Cacheable(
        value = "products",
        key = "#id"
    )
    public Product getProduct(Long id) {

        return repository.findById(id)
                .orElseThrow();
    }
}
```

First call:

```text
Cache miss
 ↓
Database
 ↓
Cache
```

Second call:

```text
Cache hit
 ↓
Return
```

---

# 55. Update Example

```java
@CachePut(
    value = "products",
    key = "#result.id"
)
public Product updateProduct(Product product) {

    return repository.save(product);
}
```

The method executes.

Then:

```text
Returned Product
 ↓
Cache updated
```

---

# 56. Delete Example

```java
@CacheEvict(
    value = "products",
    key = "#id"
)
public void deleteProduct(Long id) {

    repository.deleteById(id);
}
```

Conceptually:

```text
Delete DB
+
Evict cache
```

---

# 57. Cache Multiple Entries

Suppose:

```text
Product detail
Product list
Category product list
```

An update to one product may affect:

```text
product:101
products:category:5
products:featured
```

A single update can therefore require multiple invalidations.

This is where:

```text
@Caching
```

or explicit cache-management logic can be useful.

---

# 58. Cache Lists Carefully

Caching:

```text
GET /products
```

can be more complicated than caching:

```text
GET /products/101
```

Why?

Because one product update may invalidate:

```text
Product detail
+
Many list queries
```

For highly dynamic lists, caching individual entities may be easier to maintain.

---

# 59. Cache Key for Search

Suppose:

```text
GET /products?category=5&page=2&size=20
```

A cache key could conceptually contain:

```text
products:category=5:page=2:size=20
```

But:

```text
Too many combinations
```

can create huge cache cardinality.

Cache only search patterns with sufficient reuse.

---

# 60. Cache Cardinality

If every request creates a unique key:

```text
user-specific timestamp
random query
unique UUID
```

then:

```text
Hit rate ↓
Memory usage ↑
```

A cache is useful when multiple requests can reuse entries.

---

# 61. Cache Warming in Spring Boot

After application startup:

```text
Application starts
 ↓
Load popular reference data
 ↓
Redis
```

Possible for:

```text
Countries
Categories
Popular products
Configuration
```

Don't warm millions of entries blindly.

---

# 62. Cache Stampede in Spring

`@Cacheable` alone does not automatically solve every stampede scenario.

Suppose:

```text
Popular key expires
```

Many threads can encounter:

```text
cache miss
```

and attempt to load the same database record.

For critical hot keys, consider:

```text
Request coalescing
Locking
Refresh ahead
Jittered TTL
```

---

# 63. sync Attribute

Spring's caching abstraction can support synchronized cache loading in appropriate configurations using:

```java
@Cacheable(
    value = "products",
    key = "#id",
    sync = true
)
```

This is useful for reducing duplicate concurrent loads for the same key within the cache abstraction.

But it is not a universal distributed lock and should not be treated as one.

---

# 64. Distributed Lock vs sync=true

```text
sync=true
→ Cache abstraction coordination for concurrent loading

Redis distributed lock
→ Cross-instance distributed coordination
```

Don't confuse them.

---

# 65. Null Caching

If:

```text
Product does not exist
```

caching null can prevent repeated database queries.

But:

```text
Product gets created
```

soon afterward.

Therefore negative cache TTL should generally be short.

---

# 66. Cache Security

Do not blindly cache:

```text
Passwords
Authentication secrets
Sensitive tokens
Highly confidential data
```

If sensitive information must be cached:

```text
Access control
Encryption requirements
TTL
Network security
Audit
```

should be considered.

---

# 67. Redis Authentication

Production Redis should generally be protected using:

```text
Private networking
Authentication/ACL
TLS where required
Firewall/security groups
Least privilege
```

Never expose an unsecured Redis instance publicly.

---

# 68. Redis Failure

What should happen if Redis is unavailable?

Possible strategy:

```text
Try Redis
 ↓ failure
Fallback to DB
```

But this can create:

```text
Database overload
```

Therefore use:

```text
Timeouts
Circuit breaker
Rate limiting
Load shedding
Fallback policy
```

based on the application's needs.

---

# 69. Cache Failure Strategy

Ask:

```text
Can we serve stale data?
Can we serve partial data?
Can we return an error?
Can we fall back to DB?
Is the endpoint critical?
```

Cache failure should have a deliberate behavior.

---

# 70. Cache Observability

Track:

```text
Hit count
Miss count
Hit rate
Latency
Evictions
Memory
Errors
Serialization failures
DB fallback rate
```

A useful derived metric:

```text
DB fallback rate
```

can reveal when Redis is failing to protect the database.

---

# 71. Cache Hit Rate Is Not Everything

Suppose:

```text
Hit rate = 99%
```

but:

```text
Redis latency = 200 ms
```

The cache may still be hurting performance.

Always monitor:

```text
Hit rate
+
latency
+
memory
+
database load
```

---

# 72. Cache Invalidation Metrics

Track:

```text
Number of evictions
Number of explicit invalidations
Cache misses after writes
Stale-read incidents
```

This can help detect inconsistent cache behavior.

---

# 73. Common Mistake

Don't put:

```java
@Cacheable
```

on every method.

Ask:

```text
Is data reused?
Does it change frequently?
Can stale data be tolerated?
Is serialization expensive?
How large is it?
```

---

# 74. Common Mistake

Don't cache highly volatile data simply because:

```text
Redis is fast.
```

Caching can actually increase complexity when:

```text
Data changes constantly
Invalidation is difficult
Hit rate is low
```

---

# 75. Common Mistake

Don't cache database queries that are:

```text
Almost always unique
```

Example:

```text
GET /orders/{random-new-order-id}
```

if each order is requested once.

The cache may provide little value.

---

# 76. Common Mistake

Don't cache huge responses without considering:

```text
Memory
Serialization
Network
Eviction
```

---

# 77. Common Mistake

Don't assume:

```text
@CacheEvict
```

solves distributed consistency automatically.

There can still be:

```text
Race conditions
Failed Redis operations
Multiple cache copies
Replication delays
Concurrent requests
```

---

# 78. E-commerce Example

Product detail:

```text
GET /products/101
```

Use:

```text
@Cacheable
```

with:

```text
products
key = 101
TTL = 10 minutes
```

Update:

```text
DB update
 ↓
cache invalidation/update
```

---

# 79. E-commerce Example

Product catalog:

```text
GET /products?category=5
```

Potential cache:

```text
products:category:5
```

But invalidation is harder.

If one product changes:

```text
Many category/list caches
```

may become stale.

Consider whether caching the entire list is worth the complexity.

---

# 80. E-commerce Example

Shopping cart:

```text
cart:user:101
```

Cart is user-specific and frequently updated.

Possible Redis structure:

```text
Hash
```

or:

```text
JSON/value
```

depending on operations.

TTL may be appropriate for abandoned carts, but business rules should determine the expiration.

---

# 81. E-commerce Example

Session:

```text
session:abc123
```

Use:

```text
Redis
+
TTL
```

because sessions naturally expire.

---

# 82. E-commerce Example

Inventory:

```text
stock:product:101
```

Redis can help with:

```text
Fast counters
Atomic operations
Short-lived reservation state
```

But the final inventory correctness model must be designed carefully.

Do not assume:

```text
Redis counter
=
database inventory truth
```

without considering persistence, failures and concurrency.

---

# 83. E-commerce Example

Rate limiting:

```text
rate:user:101
```

Use:

```text
INCR
+
TTL
```

or an atomic Lua implementation.

---

# 84. Interview Question

### What does @Cacheable do?

Answer:

> "`@Cacheable` checks the configured cache before executing the method. If a matching value exists, the method can be skipped and the cached result returned. On a miss, the method executes and its result is stored in the cache."

---

# 85. Interview Question

### @CachePut vs @Cacheable?

Answer:

> "`@Cacheable` can skip method execution on a cache hit. `@CachePut` always executes the method and stores its result in the cache."

---

# 86. Interview Question

### @CacheEvict?

Answer:

> "`@CacheEvict` removes an entry or entries from a cache. It is commonly used after updates or deletes so stale data is not served."

---

# 87. Interview Question

### RedisTemplate vs Spring Cache?

Answer:

> "Spring Cache is a higher-level abstraction for method-level caching, while RedisTemplate gives direct access to Redis operations and data structures. I use Spring Cache for straightforward caching and RedisTemplate when I need Redis-specific behavior."

---

# 88. Interview Question

### Cache-aside vs write-through?

Answer:

> "With cache-aside, the application explicitly reads the cache and falls back to the database on a miss. With write-through, the cache participates in the write path and updates the backing store as part of that process."

---

# 89. Interview Question

### Why use TTL if you have @CacheEvict?

Answer:

> "Explicit eviction handles normal updates and deletes, while TTL protects against stale or orphaned entries if an invalidation is missed or a failure occurs."

---

# 90. Interview Question

### What happens if Redis goes down?

Answer:

> "If Redis is only a cache, the application can potentially fall back to the database, but I'd protect the database with timeouts, circuit breaking, rate limiting or load shedding so a Redis outage doesn't create a cache-miss storm."

---

# 91. Interview Question

### How do you serialize objects in Redis?

Answer:

> "The application uses a serializer to convert objects into Redis-compatible data. JSON is often attractive for interoperability and debugging, while binary formats can reduce size or improve performance in some cases. I also consider compatibility across deployments."

---

# 92. Interview Scenario

### Product update sometimes returns old data.

Investigate:

```text
DB commit
 ↓
Cache invalidation
 ↓
Concurrent reads
 ↓
Serialization
 ↓
TTL
```

Possible issue:

```text
DB updated
but old Redis value remains
```

---

# 93. Interview Scenario

### Redis hit rate is only 20%.

Investigate:

```text
Key cardinality
TTL
Request patterns
Cache invalidation
Unique queries
Evictions
Key construction
```

Maybe the data simply isn't reusable enough to justify caching.

---

# 94. Interview Scenario

### Redis hit rate is 99%, but database is still overloaded.

Possible:

```text
Cached data isn't the expensive DB workload
```

or:

```text
Cache misses are extremely expensive
```

or:

```text
Some endpoints bypass cache
```

Look at:

```text
DB query distribution
```

rather than assuming Redis is enough.

---

# 95. Interview Scenario

### Deployment causes deserialization errors.

Likely:

```text
Cache contains old schema
```

Solutions:

```text
Version keys
Flush compatible caches
Backward-compatible serialization
Rolling migration strategy
```

---

# 96. Interview Scenario

### One application instance sees cached data, another sees different data.

Investigate:

```text
Different Redis instances
Key prefix
Serialization
Local cache
Redis topology
Replication
Configuration
```

If there is an application-level local cache in addition to Redis:

```text
L1 cache
+
L2 Redis
```

consistency becomes more complex.

---

# 97. Two-Level Cache

Possible architecture:

```text
Application
 ↓
Local cache (L1)
 ↓ miss
Redis (L2)
 ↓ miss
Database
```

Benefits:

```text
Very low latency for hot values
Less Redis traffic
```

Costs:

```text
More complexity
Local stale data
Invalidation challenges
```

Use only when justified.

---

# 98. Cache Stampede Protection

For a very hot product:

```text
product:101
```

Possible flow:

```text
Cache expired
 ↓
One request obtains refresh lock
 ↓
Loads DB
 ↓
Updates Redis
 ↓
Other requests use refreshed value
```

This can dramatically reduce duplicate database work.

But distributed locking needs careful ownership and expiration semantics.

---

# 99. Cache Warming Strategy

After Redis restart:

```text
Load only high-value data
```

For example:

```text
Top 100 products
Popular categories
Reference data
```

Don't blindly preload every row from a huge database.

---

# 100. Cache Design Checklist

Before adding Redis caching:

```text
□ Is the data reused?
□ Is it read-heavy?
□ Can it be stale?
□ What is the source of truth?
□ What is the key?
□ What is the TTL?
□ How is invalidation handled?
□ What happens on Redis failure?
□ How large is the cached value?
□ What is expected hit rate?
□ How will serialization work?
□ How will deployment compatibility work?
□ How will it be monitored?
```

---

# 101. Final Mental Model

For Spring Boot:

```text
Controller
   ↓
Service
   ↓
Spring Cache / RedisTemplate
   ↓
Redis
   ↓ miss
Repository
   ↓
MySQL
```

For writes:

```text
Service
   ↓
DB transaction
   ↓
Cache update/invalidation
```

For failures:

```text
Redis failure
   ↓
Fallback strategy
   ↓
Protect DB
```

---

# 102. Final Interview Answer

If asked:

> "How would you implement Redis caching in your Spring Boot e-commerce application?"

Say:

> "I'd start with read-heavy data such as product details. I'd use Spring Cache with `@Cacheable` for straightforward cache-aside behavior and define explicit keys and TTLs. On updates and deletes I'd invalidate or update affected cache entries. For Redis-specific operations such as counters, sets, leaderboards or distributed locks I'd use `RedisTemplate`. I'd also monitor hit rate, latency, memory and database fallback, and design a safe fallback strategy for Redis failures."

---

# 103. Revision Checklist

```text
□ Spring Data Redis
□ RedisTemplate
□ StringRedisTemplate
□ CacheManager
□ @EnableCaching
□ @Cacheable
□ @CachePut
□ @CacheEvict
□ @Caching
□ condition
□ unless
□ sync
□ Cache-aside
□ Read-through
□ Write-through
□ Write-behind
□ Cache invalidation
□ TTL
□ Serialization
□ JSON
□ Cache key design
□ Cache versioning
□ Null caching
□ Negative caching
□ Cache stampede
□ Cache warming
□ Refresh-ahead
□ Stale-while-revalidate
□ Redis failure
□ Database protection
□ Cache hit rate
□ Cache observability
□ Two-level cache
□ E-commerce examples
□ Interview questions
```

---

# 104. What Comes Next

```text
File 05 → Redis Distributed Locking, Rate Limiting & Atomic Operations
```

Next we will focus deeply on:

```text
Distributed locks
SET NX EX
Safe lock release
Lock ownership
Lease expiration
Redlock concepts
Fencing tokens
Race conditions
Rate limiting
Fixed window
Sliding window
Token bucket
Lua scripts
Atomic inventory
Idempotency
Distributed job coordination
Real-world Spring Boot examples
Interview scenarios
```

The key takeaway:

> **Spring Cache makes common caching easy, but production Redis requires you to understand what happens around the abstraction: key design, serialization, TTL, invalidation, concurrency, failure handling and database protection.**
