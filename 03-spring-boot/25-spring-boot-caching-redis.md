# Spring Boot — Caching with Redis

This file covers caching in Spring Boot with Redis, including cache-aside, TTL, eviction, serialization, cache invalidation, distributed caching, and interview scenarios.

The goal is to understand when caching improves a backend system and how to avoid the consistency and reliability problems caching introduces.

---

# 1. What Is Caching?

Caching stores frequently accessed data in a faster storage layer so the application does not repeatedly perform expensive operations.

Typical flow:

```text
Client
  ↓
Spring Boot
  ↓
Cache
  |
  +-- hit → response
  |
  +-- miss
       ↓
    Database
       ↓
     Cache
       ↓
    response
```

---

# 2. Why Use a Cache?

Caching can improve:

```text
Response latency
Database load
Throughput
Scalability
```

But caching also introduces:

```text
Stale data
Invalidation complexity
Memory usage
Serialization overhead
Cache failures
```

A cache is useful only when its tradeoffs make sense.

---

# 3. Redis

Redis is an in-memory data store commonly used for:

```text
Caching
Distributed locks
Counters
Rate limiting
Short-lived state
Session storage
Pub/Sub
Streams
```

For Spring Boot applications, Redis is frequently used as a distributed cache.

---

# 4. Local Cache vs Distributed Cache

Local cache:

```text
Application Instance A
       ↓
     Local RAM
```

Each instance has its own cache.

Distributed cache:

```text
Instance A ──┐
Instance B ──┼──> Redis
Instance C ──┘
```

All application instances can access shared cached data.

---

# 5. Why Not Only Use Local Memory?

Suppose:

```text
Instance A → product 100 = cached
Instance B → product 100 = not cached
```

Users can receive different cache behavior depending on which instance receives the request.

Local caching can still be useful for extremely hot, immutable, or low-risk data, but distributed caching provides shared state across instances.

---

# 6. Cache-Aside Pattern

The most common application caching pattern is cache-aside.

Flow:

```text
Read
 ↓
Check cache
 |
 +-- hit → return
 |
 +-- miss
      ↓
   database
      ↓
   cache
      ↓
   return
```

The application explicitly manages cache reads and writes.

---

# 7. Cache-Aside Write

Typical update:

```text
Update database
      ↓
Invalidate cache
```

Then the next read:

```text
Cache miss
 ↓
Database
 ↓
Populate cache
```

This is often safer than blindly updating cache and database in an arbitrary order.

---

# 8. Read-Through Cache

With read-through behavior, the cache layer itself can retrieve data from the backing store when the requested key is missing.

Conceptually:

```text
Application
    ↓
Cache
    ↓ miss
Database
```

The application doesn't explicitly perform every cache-miss lookup.

This is different from the common Spring cache-aside approach where application-level cache abstraction is used around methods.

---

# 9. Write-Through Cache

Write-through means writes go through the cache and the cache updates the backing store.

Conceptually:

```text
Application
   ↓
Cache
   ↓
Database
```

This can keep cache and database updates coordinated, but introduces additional complexity and latency.

---

# 10. Write-Behind / Write-Back

Write-back can acknowledge writes to the cache first and persist them later.

Conceptually:

```text
Application
   ↓
Cache
   ↓
Async persistence
   ↓
Database
```

This can improve write latency but increases durability and consistency risks.

It should be used only when those tradeoffs are acceptable.

---

# 11. Spring Cache Abstraction

Spring provides annotations such as:

```text
@Cacheable
@CachePut
@CacheEvict
@Caching
```

The application code can work with a cache abstraction instead of directly calling Redis for every cache operation.

---

# 12. @Cacheable

Example:

```java
@Cacheable(value = "products", key = "#id")
public ProductResponse getProduct(Long id) {

    return productRepository.findById(id)
        .map(productMapper::toResponse)
        .orElseThrow();
}
```

Flow:

```text
First call
 ↓
Cache miss
 ↓
Database
 ↓
Result stored in cache
```

Next call:

```text
Cache hit
 ↓
Return cached result
```

---

# 13. @CachePut

`@CachePut` executes the method and then updates the cache with the returned value.

Example:

```java
@CachePut(value = "products", key = "#id")
public ProductResponse updateProduct(
        Long id,
        ProductRequest request) {

    Product product = ...;

    return productMapper.toResponse(
        productRepository.save(product)
    );
}
```

Unlike `@Cacheable`, the method is not skipped because of a cache hit.

---

# 14. @CacheEvict

Used to remove cached data.

Example:

```java
@CacheEvict(value = "products", key = "#id")
public void deleteProduct(Long id) {

    productRepository.deleteById(id);
}
```

This prevents deleted data from remaining indefinitely in the cache.

---

# 15. Evict All

Example:

```java
@CacheEvict(
    value = "products",
    allEntries = true
)
public void clearProductCache() {
}
```

Use this carefully.

Evicting a large cache can cause a sudden wave of database traffic.

---

# 16. @Caching

Multiple cache operations can be combined.

Example:

```java
@Caching(
    evict = {
        @CacheEvict(
            value = "products",
            key = "#id"
        )
    }
)
public void update(Long id) {
    ...
}
```

For complex caching rules, keep the configuration understandable.

---

# 17. Enabling Caching

Spring Boot applications can enable caching using:

```java
@EnableCaching
@SpringBootApplication
public class Application {
}
```

Then cache annotations can be used on appropriate methods.

---

# 18. Redis Cache Manager

Spring Boot can configure Redis as the cache provider.

Conceptually:

```text
Spring Cache
     ↓
RedisCacheManager
     ↓
Redis
```

The application interacts primarily with Spring's cache abstraction.

---

# 19. Redis Connection

Typical architecture:

```text
Spring Boot
     ↓
RedisConnectionFactory
     ↓
Redis
```

Depending on the configuration, Spring Boot commonly uses Lettuce as the Redis client.

---

# 20. Basic Dependency

A typical Maven dependency is:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

Spring Boot version determines the exact transitive dependency versions.

---

# 21. Basic Configuration

Example:

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```

For production, credentials, TLS, timeouts, pooling, and topology should be configured appropriately.

Never hardcode production secrets.

---

# 22. Redis Key Design

Example:

```text
product:100
product:101
product:102
```

Good keys are:

```text
Predictable
Consistent
Collision-resistant
Easy to debug
```

A namespace prefix is useful:

```text
product:
order:
user:
```

---

# 23. Cache Key in Spring

Example:

```java
@Cacheable(
    value = "products",
    key = "'product:' + #id"
)
public ProductResponse getProduct(Long id) {
    ...
}
```

Be consistent about key construction.

---

# 24. Composite Cache Key

Suppose pricing depends on:

```text
productId
customerType
currency
```

A key can include all relevant dimensions:

```text
price:100:PREMIUM:INR
```

If a value depends on multiple inputs, the cache key must reflect those inputs.

---

# 25. TTL

TTL means:

```text
Time To Live
```

Example:

```text
product:100
TTL = 10 minutes
```

After expiration, the key becomes unavailable and a future request can repopulate it.

---

# 26. Choosing TTL

Short TTL:

```text
More fresh
More cache misses
Higher database load
```

Long TTL:

```text
Better hit rate
Higher staleness risk
```

Choose based on business tolerance for stale data.

---

# 27. TTL Examples

Product catalog:

```text
5–30 minutes
```

Configuration:

```text
Minutes to hours
```

OTP/session-like short-lived data:

```text
Seconds to minutes
```

These are examples, not universal values.

---

# 28. Cache Hit Ratio

Cache hit ratio:

```text
cache hits
-----------------------------
cache hits + cache misses
```

Example:

```text
900 hits
100 misses
```

Hit ratio:

```text
90%
```

A high hit ratio is useful, but business latency and database load matter more than the number alone.

---

# 29. Cache Miss

A cache miss means requested data isn't available in cache.

Possible causes:

```text
Key never cached
TTL expired
Cache eviction
Cache cleared
Wrong key
Serialization failure
Cache outage
```

---

# 30. Cache Stampede

Suppose a popular key expires:

```text
10,000 requests
      ↓
Cache miss
      ↓
10,000 DB queries
```

This can overload the database.

This is called:

```text
Cache stampede
```

---

# 31. Preventing Cache Stampede

Techniques include:

```text
Jittered TTL
Request coalescing
Distributed lock
Background refresh
Cache warming
```

Use the simplest technique appropriate for the workload.

---

# 32. Jittered TTL

Instead of:

```text
TTL = exactly 10 minutes
```

use a randomized range:

```text
9–11 minutes
```

This prevents many related keys from expiring at exactly the same moment.

---

# 33. Cache Warming

Preload important data before traffic arrives.

Example:

```text
Application startup
      ↓
Load popular products
      ↓
Redis
```

Useful for:

```text
Frequently accessed reference data
Predictable hot content
```

Avoid warming millions of rarely accessed keys.

---

# 34. Cache Penetration

Cache penetration occurs when requests repeatedly query data that doesn't exist.

Example:

```text
product:999999
```

doesn't exist.

Every request:

```text
Cache miss
 ↓
Database query
 ↓
Not found
```

---

# 35. Prevent Cache Penetration

Options:

```text
Cache negative results
Validate input
Bloom filter
Rate limiting
```

Negative caching should have a sensible short TTL.

---

# 36. Cache Avalanche

Cache avalanche can occur when many keys expire or become unavailable around the same time.

Result:

```text
Many cache misses
      ↓
Database overload
```

Mitigations:

```text
TTL jitter
Staggered expiration
High availability
Fallbacks
Cache warming
Rate limiting
```

---

# 37. Cache Consistency

Suppose:

```text
Database = price 600
Redis = price 500
```

The cache is stale.

You must decide:

```text
Can stale data be tolerated?
```

If not, caching may need stronger invalidation/update rules.

---

# 38. Database Update + Cache Invalidation

Common approach:

```text
BEGIN transaction
 ↓
Update database
 ↓
COMMIT
 ↓
Evict cache
```

The exact ordering matters.

Evicting before a transaction commits can expose stale or rolled-back behavior depending on the workflow.

---

# 39. Cache Failure

If Redis fails:

```text
Application
     ↓
Redis unavailable
```

Possible fallback:

```text
Database
```

But if thousands of requests bypass Redis simultaneously:

```text
Database overload
```

So cache failure must be part of capacity and resilience planning.

---

# 40. Circuit Breaker for Redis

If Redis is repeatedly failing, a circuit breaker or carefully designed fallback can prevent every request from repeatedly waiting on an unavailable cache.

However, a fallback must protect the database from a sudden traffic surge.

---

# 41. Redis Serialization

Spring needs to serialize cached objects.

Common choices:

```text
String
JSON
Java serialization
Binary formats
```

JSON is often easier to inspect and evolve, while binary formats can provide different performance and compatibility characteristics.

Avoid blindly using Java native serialization for distributed cache contracts.

---

# 42. Cache DTOs Instead of Entities

Prefer:

```text
ProductResponse
```

over directly caching a JPA entity when possible.

Reasons:

```text
Stable cache contract
Avoid persistence-context issues
Avoid lazy-loading surprises
Smaller payload
Clear API model
```

---

# 43. Redis Data Structures

Redis supports data structures such as:

```text
String
Hash
List
Set
Sorted Set
Streams
```

For ordinary Spring caching, simple key/value storage is often sufficient.

---

# 44. Redis String

Example:

```text
product:100 → JSON
```

Useful for:

```text
Cached objects
Counters
Tokens
Simple values
```

---

# 45. Redis Hash

Conceptually:

```text
user:100
    name → Sudhir
    city → Bangalore
    tier → PREMIUM
```

Useful when individual fields need to be accessed separately.

---

# 46. Redis Set

Useful for unique collections.

Example:

```text
product:100:tags
```

Values:

```text
java
backend
spring
```

---

# 47. Redis Sorted Set

Useful when members have scores.

Example:

```text
leaderboard
```

```text
Sudhir → 1500
Alex   → 1400
John   → 1300
```

The score can determine ranking.

---

# 48. Redis Lists

Lists can represent ordered collections.

Use cases can include:

```text
Queues
Recent items
Simple task lists
```

For robust distributed event processing, use the appropriate messaging technology rather than assuming Redis Lists replace Kafka.

---

# 49. Redis Streams

Redis Streams provide an append-oriented data structure with consumer-group capabilities.

They can support:

```text
Event processing
Consumer groups
Message IDs
Replay-like processing patterns
```

Kafka may still be a better fit for large durable event-streaming workloads.

---

# 50. Redis Pub/Sub

Redis Pub/Sub provides:

```text
Publisher
    ↓
Channel
    ↓
Subscribers
```

It is useful for transient notifications.

Important:

> Redis Pub/Sub does not provide Kafka-style durable message retention and replay.

---

# 51. Redis Distributed Lock

A distributed lock can coordinate work across multiple application instances.

Conceptually:

```text
Instance A
    ↓
Acquire lock
    ↓
Do work
    ↓
Release lock
```

Instance B:

```text
Cannot acquire lock
```

Use distributed locks carefully because failures, expiry, and ownership need to be handled correctly.

---

# 52. Redis for Rate Limiting

Example:

```text
rate:user:100
```

A counter can track requests within a time window.

Conceptually:

```text
Request
 ↓
Redis counter
 ↓
Limit exceeded?
 ├─ yes → 429
 └─ no → continue
```

For precise distributed rate limiting, atomic Redis operations or Lua scripts may be needed.

---

# 53. Redis Atomic Operations

Redis commands such as increments are atomic at the command level.

Example:

```text
INCR rate:user:100
```

This can be useful for counters and rate limiting.

Multi-step logic may require:

```text
Lua script
Transaction
Careful atomic design
```

depending on the requirement.

---

# 54. Cache Eviction Policies

When Redis memory is constrained, eviction policies determine what happens.

Examples include:

```text
noeviction
allkeys-lru
allkeys-lfu
volatile-lru
volatile-ttl
```

The exact policy should match the workload.

---

# 55. LRU

LRU means:

```text
Least Recently Used
```

The system tends to remove keys that have not been accessed recently.

Useful when recent access predicts future access.

---

# 56. LFU

LFU means:

```text
Least Frequently Used
```

The system considers how frequently keys are accessed.

Useful when a small set of keys are consistently hot.

---

# 57. Redis Memory Planning

Consider:

```text
Number of keys
Average value size
Metadata overhead
Replication
Eviction policy
Peak cache usage
Growth
```

Don't size Redis only from the raw application payload size.

---

# 58. Redis High Availability

Possible architectures:

```text
Redis Sentinel
Redis Cluster
Managed Redis
```

They solve different operational and scaling requirements.

---

# 59. Redis Sentinel

Sentinel can provide:

```text
Monitoring
Failover
Primary discovery
```

It is useful for high availability of Redis deployments.

---

# 60. Redis Cluster

Redis Cluster distributes keys across multiple nodes.

Useful for:

```text
Horizontal data scaling
Higher memory capacity
Distributed key space
```

Applications must understand cluster-specific behavior and key distribution.

---

# 61. Redis Replication

Typical model:

```text
Primary
  |
+---+---+
|       |
Replica Replica
```

Replication can improve read scaling and failover capabilities depending on architecture.

---

# 62. Redis Persistence

Redis can support persistence mechanisms such as:

```text
RDB snapshots
AOF
```

For a pure cache, losing the cache may be acceptable because data can be rebuilt from the database.

For Redis used as a source of important state, persistence and recovery requirements become much more important.

---

# 63. Redis Should Not Automatically Become the Source of Truth

Typical architecture:

```text
MySQL → Source of truth
Redis  → Cache
```

If Redis is lost:

```text
Rebuild cache
```

This is usually easier to reason about.

If Redis stores unique business state, design it as a data store rather than treating it as "just a cache."

---

# 64. Cache Key Namespacing

Use prefixes:

```text
product:100
user:100
order:100
```

This helps:

```text
Debugging
Isolation
Bulk operations
Avoiding collisions
```

---

# 65. Versioned Cache Keys

For large schema changes:

```text
product:v1:100
product:v2:100
```

This can help avoid deserialization conflicts during rolling deployments.

Usually, backward-compatible cache serialization plus controlled eviction is simpler.

---

# 66. Cache and Deployment

During deployments, old and new application versions may run simultaneously.

If the cached structure changes:

```text
Old app → old format
New app → new format
```

Potential deserialization failures can occur.

Solutions:

```text
Backward-compatible formats
Versioned keys
Controlled cache eviction
Rolling deployment strategy
```

---

# 67. Spring Cache Key Generator

For complex key rules, Spring allows custom key generation.

Conceptually:

```java
@Bean
public KeyGenerator customKeyGenerator() {
    ...
}
```

Use this only when default key expressions become difficult to maintain.

---

# 68. Conditional Caching

You may not want to cache every result.

Example:

```java
@Cacheable(
    value = "products",
    key = "#id",
    condition = "#id > 0"
)
public ProductResponse getProduct(Long id) {
    ...
}
```

Conditions should reflect meaningful business or technical requirements.

---

# 69. Unless / Unless

Spring cache annotations can prevent caching certain results.

Example:

```java
@Cacheable(
    value = "products",
    key = "#id",
    unless = "#result == null"
)
```

This avoids caching null results when that is not desired.

---

# 70. Cache Null Values

Caching "not found" results can protect the database from repeated invalid lookups.

Example:

```text
product:999999 → NOT_FOUND
```

Use a short TTL.

Otherwise a newly created record might remain invisible until the negative cache expires.

---

# 71. Cache Warm-Up

At application startup:

```text
Application starts
 ↓
Load selected reference data
 ↓
Redis
```

Useful for:

```text
Countries
Currencies
Feature configuration
Frequently accessed catalog data
```

But avoid startup tasks that make application readiness unnecessarily slow.

---

# 72. Cache Refresh

Instead of waiting for expiry:

```text
Current cache
     ↓
Background refresh
     ↓
New value
```

This can reduce latency spikes for hot data.

It requires careful scheduling and failure handling.

---

# 73. Request Coalescing

If many requests miss the same key:

```text
100 requests
     ↓
Same cache key
```

Instead of:

```text
100 database queries
```

coordinate them so:

```text
1 request → database
99 requests → wait for result
```

This is request coalescing.

---

# 74. Distributed Cache Locking

When multiple instances can populate the same hot key, a distributed lock can prevent duplicate expensive work.

Flow:

```text
Cache miss
 ↓
Acquire lock
 ↓
Double-check cache
 ↓
Load DB
 ↓
Populate cache
 ↓
Release lock
```

Always design lock expiry and failure behavior carefully.

---

# 75. Cache Double-Check

After acquiring a lock:

```text
Check cache again
```

Why?

Another instance may have populated the key while the current request was waiting for the lock.

Without a second check, unnecessary database work can still occur.

---

# 76. Cache Serialization Failure

If cached data cannot be deserialized:

```text
Cache read fails
```

Possible response:

```text
Remove bad key
 ↓
Load from database
 ↓
Store corrected value
```

Log the failure and monitor it.

---

# 77. Cache Observability

Monitor:

```text
Hit ratio
Miss rate
Evictions
Memory
Latency
Connection errors
Command rate
Hot keys
Redis CPU
Redis network
```

Also monitor the application impact:

```text
Database load
API latency
Error rate
```

---

# 78. Cache Latency

A cache should be meaningfully faster than the backing store.

If:

```text
Redis = 20 ms
Database = 15 ms
```

caching may not provide the expected benefit for that operation.

Measure actual end-to-end latency.

---

# 79. Cache Only What Helps

Good cache candidates:

```text
Frequently read
Expensive to compute/query
Relatively stable
Safe to be slightly stale
```

Poor candidates:

```text
Rarely accessed
Frequently changing
Huge objects
Highly sensitive data without proper controls
Data requiring strict immediate consistency
```

---

# 80. Sensitive Data in Cache

Be careful caching:

```text
Authentication tokens
Personal data
Payment information
Secrets
```

Consider:

```text
Encryption
Access controls
TTL
Data minimization
Network security
```

Don't cache sensitive data simply because Redis makes it easy.

---

# 81. Cache Security

Production Redis should consider:

```text
Authentication
TLS
Network isolation
ACLs
Secret management
Least privilege
```

Do not expose Redis directly to the public internet.

---

# 82. Redis Connection Pool

Application instances may maintain multiple Redis connections.

Consider:

```text
Number of app instances
Connection pool size
Redis capacity
Command latency
Peak concurrency
```

More connections do not automatically mean better performance.

---

# 83. Redis Timeout

Configure sensible:

```text
Connect timeout
Command timeout
Pool timeout
```

A cache should not cause every request to hang indefinitely when Redis is unavailable.

---

# 84. Cache Failure Strategy

Possible strategies:

```text
Fail open
Fail closed
Fallback to database
Return degraded response
```

For a product catalog:

```text
Redis unavailable
 ↓
Database fallback
```

may be acceptable.

For a security decision, blindly failing open may be dangerous.

---

# 85. Cache-Aside Example

Service:

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    @Cacheable(
        value = "products",
        key = "#id"
    )
    public ProductResponse getProduct(
            Long id) {

        Product product =
            repository.findById(id)
                .orElseThrow();

        return map(product);
    }

    @CacheEvict(
        value = "products",
        key = "#id"
    )
    public void deleteProduct(Long id) {

        repository.deleteById(id);
    }
}
```

This is a simple and common starting point.

---

# 86. Update Strategy

Option 1:

```text
DB update
 ↓
Evict cache
```

Option 2:

```text
DB update
 ↓
Update cache
```

Option 3:

```text
DB update
 ↓
Event
 ↓
Cache invalidation/update
```

Choose based on consistency and architecture.

---

# 87. Cache Invalidation Through Events

In microservices:

```text
Product Service
    ↓
ProductUpdated
    ↓
Kafka
    ↓
Cache Consumer
    ↓
Redis invalidation
```

This can help synchronize distributed read models.

But event delivery is asynchronous, so a short stale period may still exist.

---

# 88. Cache and Transactions

Suppose:

```text
@Transactional
updateProduct()
```

If the cache is updated before the database transaction commits and the transaction later rolls back:

```text
Cache = new value
Database = old value
```

That is inconsistent.

Think carefully about transaction boundaries and cache update timing.

---

# 89. Transactional Event + Cache

One approach:

```text
DB transaction
 ↓
Commit
 ↓
Publish application event
 ↓
Evict cache
```

For cross-service durable propagation, use an appropriate event/outbox architecture.

---

# 90. Redis and Distributed Sessions

Instead of:

```text
Session stored in Instance A
```

use:

```text
Instance A ─┐
Instance B ─┼→ Redis
Instance C ─┘
```

This allows any application instance to access session state.

Spring Session can integrate with Redis for this use case.

---

# 91. Session vs JWT

Session:

```text
Server-side state
```

JWT:

```text
Token carries claims
```

Redis can still be used with JWT for:

```text
Token revocation
Refresh token storage
Session-like state
Rate limiting
```

Don't add Redis simply because JWT exists.

---

# 92. Redis Rate Limiting Example

Conceptual:

```text
Key:
rate:user:100

INCR
EXPIRE
```

If:

```text
count > limit
```

return:

```text
429 Too Many Requests
```

For atomic correctness across multiple operations, use a Lua script or another appropriate atomic approach.

---

# 93. Redis Leaderboard

Use a sorted set:

```text
leaderboard
```

Scores:

```text
Alice → 1000
Bob   → 950
John  → 900
```

Then retrieve rankings efficiently.

---

# 94. Redis Distributed Lock Example

Conceptually:

```text
SET lock:order:100 token NX PX 10000
```

Meaning:

```text
Create only if absent
Set expiration
```

The token identifies the owner.

Release should verify ownership rather than blindly deleting the key.

For production-grade locking, carefully evaluate established algorithms/libraries rather than implementing distributed locking casually.

---

# 95. Redis Pub/Sub vs Kafka

Redis Pub/Sub:

```text
Low latency
Simple
Transient
No durable replay
```

Kafka:

```text
Durable
Replayable
Partitioned
Consumer groups
High-throughput event streaming
```

Choose according to delivery and retention requirements.

---

# 96. Redis Streams vs Kafka

Redis Streams:

```text
Convenient if Redis is already central
Consumer groups
Stream IDs
```

Kafka:

```text
Purpose-built distributed event streaming
Long retention
Large-scale partitioning
Strong ecosystem for event pipelines
```

---

# 97. Cache Consistency Levels

Think in terms of:

```text
Strong
Read-after-write
Eventual
Best effort
```

The correct level depends on the business requirement.

---

# 98. Cache in Ecommerce

Good candidates:

```text
Product details
Category data
Popular products
Reference data
Frequently viewed catalog information
```

Be cautious with:

```text
Inventory
Payment status
Order state
```

if the business requires strongly current values.

---

# 99. Ecommerce Product Cache

Architecture:

```text
Client
  ↓
Product Service
  ↓
Redis
  |
  +-- hit → ProductResponse
  |
  +-- miss
       ↓
     MySQL
       ↓
     Redis
       ↓
   ProductResponse
```

---

# 100. Ecommerce Product Update

```text
Admin updates product
        ↓
Product Service
        ↓
MySQL
        ↓
Commit
        ↓
Evict product cache
```

Next request reloads the new value.

---

# 101. Ecommerce Inventory

Inventory usually requires stronger consistency than product descriptions.

Instead of relying only on a cache:

```text
Request
 ↓
Inventory DB
 ↓
Atomic reservation
```

Redis can still support:

```text
Fast availability hints
Reservation coordination
Counters
```

but the exact design must match overselling requirements.

---

# 102. Cache Key for User Cart

Example:

```text
cart:user:100
```

TTL might be used if abandoned carts should expire.

But the source of truth should be selected based on whether cart durability is required.

---

# 103. Cache Eviction for Cart

When cart changes:

```text
Add item
 ↓
Update cart
 ↓
Invalidate/update cart cache
```

Avoid returning stale cart data after mutations if the business expects immediate read-after-write behavior.

---

# 104. Product List Cache

A product list query:

```text
category=phones
page=0
size=20
sort=price
```

could use:

```text
products:phones:0:20:price
```

But list caches can become difficult to invalidate when individual products change.

Consider whether caching individual product details gives better value.

---

# 105. Cache Invalidation Problem

Classic principle:

> Cache invalidation is one of the hard problems in computer science.

If:

```text
Product changes
```

which caches need invalidation?

```text
product:100
category:phones:page:0
search:phones
popular-products
recommendations
```

The more derived caches you create, the more invalidation complexity you introduce.

---

# 106. Avoid Excessive Caching

Start with:

```text
Product detail cache
```

Measure.

Then consider:

```text
Product list
Search results
Recommendations
```

only if needed.

---

# 107. Cache Stampede Protection Example

Conceptual flow:

```text
Cache miss
    ↓
Acquire lock
    ↓
Check cache again
    ↓
Still missing?
    ↓
Load DB
    ↓
Set cache
    ↓
Release lock
```

The second cache check is important.

---

# 108. Cache Failure and Database Protection

Bad:

```text
Redis down
 ↓
Every request → DB
```

Better:

```text
Redis down
 ↓
Rate limit / fallback controls
 ↓
Database protected
```

The cache should not become a single hidden dependency that can trigger database collapse.

---

# 109. Cache Metrics Example

Dashboard:

```text
Cache Hit Ratio: 94%
Cache Miss Rate: 6%
Redis Latency p95: 2ms
Evictions: low
Memory: 68%
DB CPU: 45%
```

Use these metrics to determine whether caching is actually helping.

---

# 110. Interview: What Is Cache-Aside?

> Cache-aside means the application first checks the cache. On a miss, it reads from the database and then populates the cache. On updates, the application generally updates the database and then invalidates or refreshes the corresponding cache entry.

---

# 111. Interview: Why Redis?

> Redis is an in-memory distributed data store that provides very fast access and supports useful structures such as strings, hashes, sets, sorted sets, and streams. In Spring Boot, I commonly use it for distributed caching, rate limiting, short-lived state, and other low-latency use cases.

---

# 112. Interview: What Is @Cacheable?

> `@Cacheable` tells Spring to check the configured cache before executing the method. If the key is already cached, the method can be skipped and the cached result returned. On a miss, the method executes and the result is cached.

---

# 113. Interview: @Cacheable vs @CachePut

> `@Cacheable` can skip method execution when a value already exists in the cache. `@CachePut` always executes the method and updates the cache with the returned value.

---

# 114. Interview: @CacheEvict?

> `@CacheEvict` removes a cached value. I use it when data is deleted or changed and the existing cache entry is no longer valid.

---

# 115. Interview: What Is TTL?

> TTL means Time To Live. It defines how long a cached entry remains available before it expires. The right TTL depends on how frequently the underlying data changes and how much staleness the business can tolerate.

---

# 116. Interview: How Do You Prevent Cache Stampede?

> I can use TTL jitter, request coalescing, background refresh, or a distributed lock for particularly hot keys. The goal is to prevent many simultaneous cache misses from generating a large burst of database requests.

---

# 117. Interview: What Happens If Redis Goes Down?

> I first decide whether the application can safely fall back to the database. If it can, I use timeouts and fallback controls while protecting the database from a cache-miss storm. For some workloads, degraded responses or fail-fast behavior may be better than overwhelming the primary database.

---

# 118. Interview: How Do You Handle Cache Consistency?

> I first define how much staleness the business allows. For cache-aside, I typically update the database and invalidate the affected cache after the database change succeeds. For distributed workflows, events or an outbox can propagate invalidation, but the system may still be eventually consistent.

---

# 119. Interview: Should You Cache Database Entities?

> I generally prefer caching stable DTOs or explicit cache models rather than JPA entities. This avoids persistence-context and lazy-loading surprises and gives me better control over the cache contract and serialized payload.

---

# 120. Interview: Local Cache vs Redis?

> A local cache is extremely fast but each application instance has separate state. Redis provides a shared distributed cache across instances. I choose based on consistency, scale, latency, and whether sharing cached state across instances is important.

---

# 121. Interview: How Do You Design Redis Keys?

> I use predictable namespaced keys such as `product:100` and include every input that affects the cached result. For composite data, I use a consistent format such as `price:100:PREMIUM:INR`.

---

# 122. Interview: How Do You Handle Cache Invalidation After DB Update?

> A common approach is to commit the database update first and then invalidate the cache. For more distributed systems, I can publish a durable event after the transaction using an Outbox Pattern and let consumers invalidate or update their caches.

---

# 123. Interview: Why Not Cache Everything?

> Caching consumes memory and introduces invalidation and consistency complexity. I cache data that is frequently read, expensive to obtain, reasonably stable, and allowed to be stale for the chosen TTL.

---

# 124. Interview: What Is Cache Penetration?

> Cache penetration happens when repeated requests target data that doesn't exist, causing every request to miss the cache and hit the database. Negative caching, validation, rate limiting, or a Bloom filter can reduce this problem.

---

# 125. Interview: Cache Avalanche vs Stampede?

> A cache stampede usually refers to many requests simultaneously missing the same popular key. A cache avalanche is a broader situation where many cached entries expire or become unavailable together, causing a large load on the backing system.

---

# 126. Interview: Redis Pub/Sub vs Kafka?

> Redis Pub/Sub is lightweight and low latency but messages are transient and don't provide Kafka-style durable replay. Kafka is better when I need durable event streams, retention, consumer groups, replay, and large-scale event processing.

---

# 127. Interview: How Does Spring Cache Work?

> Spring provides a cache abstraction through annotations such as `@Cacheable`, `@CachePut`, and `@CacheEvict`. A cache manager connects that abstraction to a provider such as Redis. Spring intercepts the method invocation and performs the appropriate cache operation.

---

# 128. Interview: What Is Cache-Aside in Ecommerce?

> For product details, the service first checks Redis. On a miss it loads the product from MySQL, maps it to a DTO, stores the DTO in Redis with a suitable TTL, and returns it. When the product changes, I invalidate or refresh the cache after the database update.

---

# 129. Interview: Would You Cache Inventory?

> I would be careful. Inventory is often consistency-sensitive because stale values can cause overselling. I might cache availability hints for performance, but the final reservation should be protected by an authoritative transactional or atomic inventory mechanism.

---

# 130. Interview: How Do You Protect Redis?

> I use network isolation, authentication and authorization, TLS where appropriate, least-privilege access, secure secret management, and monitoring. Redis should not be directly exposed to the public internet.

---

# 131. Interview: What Is Redis Cluster?

> Redis Cluster distributes keys across multiple Redis nodes and provides horizontal data scaling. It is useful when one Redis node cannot provide enough memory or throughput, but it adds topology and key-distribution considerations.

---

# 132. Interview: What Is Redis Sentinel?

> Sentinel provides monitoring and automated failover capabilities for Redis primary/replica deployments. It is mainly focused on availability and failover rather than sharding the dataset across many nodes.

---

# 133. Interview: How Do You Handle Hot Keys?

> I first identify the hot key using metrics. Depending on the workload, I can use local caching, request coalescing, background refresh, CDN caching, or carefully distribute access. The solution must preserve the required consistency.

---

# 134. Interview: How Do You Handle a Database Thundering Herd?

> I protect hot cache misses using request coalescing, locking, TTL jitter, or background refresh. I also make sure Redis failure or mass expiration cannot send uncontrolled traffic to the database.

---

# 135. Interview: Cache vs Database

> The database is usually the source of truth for transactional business data. The cache is an optimization layer that stores data temporarily for faster access. If the cache can be deleted and rebuilt from the database, the architecture is easier to reason about.

---

# 136. Interview: Redis as Database vs Cache?

> Redis can be used as a primary data store for some workloads, but that requires explicit durability, recovery, consistency, and data-model decisions. If I'm using Redis only as a cache, I treat the database as the source of truth and design the cache to be rebuildable.

---

# 137. Practical Spring Boot Architecture

```text
Client
  ↓
Load Balancer
  ↓
Spring Boot
  |
  +--> Redis
  |
  +--> MySQL
  |
  +--> Kafka
```

Typical responsibilities:

```text
Redis → fast reads
MySQL → transactional source of truth
Kafka → asynchronous events
```

---

# 138. Product Read Flow

```text
GET /products/100
        ↓
ProductService
        ↓
Redis?
   |          |
  HIT        MISS
   |          |
   ↓          ↓
Return      MySQL
              ↓
            DTO
              ↓
            Redis
              ↓
            Return
```

---

# 139. Product Update Flow

```text
PUT /products/100
        ↓
ProductService
        ↓
MySQL transaction
        ↓
Commit
        ↓
Evict Redis key
        ↓
Publish ProductUpdated if needed
```

If other services have derived caches, the event can help propagate invalidation.

---

# 140. Complete Cache Strategy

For a product catalog:

```text
Source of truth:
MySQL

Cache:
Redis

Pattern:
Cache-aside

Key:
product:{id}

TTL:
Business-dependent

Invalidation:
After successful DB update

Failure:
Fallback carefully to DB

Monitoring:
Hit ratio + latency + memory + DB load
```

---

# 141. Cache Checklist

```text
□ Cache candidate identified
□ Cache-aside/read-through decision
□ Key design
□ TTL
□ Serialization
□ Invalidation
□ Negative caching
□ Stampede protection
□ Cache penetration protection
□ Cache avalanche protection
□ Memory limits
□ Eviction policy
□ Redis HA
□ Redis security
□ Redis timeout
□ Fallback strategy
□ Observability
□ Deployment compatibility
```

---

# 142. Final Mental Model

```text
                    APPLICATION
                         |
                         v
                      CACHE
                         |
               +---------+---------+
               |                   |
             HIT                  MISS
               |                   |
               v                   v
            Return              DATABASE
                                   |
                                   v
                                 Cache
                                   |
                                   v
                                 Return
```

For writes:

```text
Application
    ↓
Database
    ↓
Commit
    ↓
Invalidate / Refresh Cache
```

For distributed systems:

```text
Database Transaction
       ↓
Outbox
       ↓
Kafka
       ↓
Cache Invalidation / Update
```

---

# 143. Final Rule

> **Caching is an optimization, not a substitute for correct data ownership. Start by identifying expensive and frequently accessed reads, define how much staleness is acceptable, choose a cache strategy, design invalidation carefully, and always plan for cache failure.**
