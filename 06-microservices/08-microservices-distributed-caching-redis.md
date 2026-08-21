# Microservices — Distributed Caching & Redis

This file covers caching and Redis from a Java/Spring Boot microservices interview perspective.

Core topics:

```text
Why caching?
Cache-aside
Read-through
Write-through
Write-behind
Cache invalidation
TTL
Redis
Eviction
Serialization
Distributed cache
Cache stampede
Cache penetration
Cache avalanche
Consistency
Distributed locking
Caching in Spring Boot
Redis interview scenarios
```

---

# 1. What Is Caching?

Caching stores frequently accessed data in a faster storage layer so future requests can be served more quickly.

Without cache:

```text
Client
 ↓
Service
 ↓
Database
 ↓
Response
```

With cache:

```text
Client
 ↓
Service
 ↓
Cache
 ↓
Response
```

If the data is not cached:

```text
Cache MISS
 ↓
Database
 ↓
Cache
 ↓
Response
```

---

# 2. Why Use Caching?

Caching can provide:

```text
Lower latency
Reduced database load
Higher throughput
Better scalability
```

Example:

```text
Product page
 ↓
Redis
 ↓
5 ms
```

instead of:

```text
Product page
 ↓
Database
 ↓
50 ms
```

The exact latency depends on infrastructure and workload.

---

# 3. What Should We Cache?

Good candidates often include:

```text
Frequently read data
Slow-to-compute data
Data that doesn't change every second
Reference data
Product catalog
Configuration
Popular queries
Session-related data where appropriate
```

---

# 4. What Should We Avoid Caching?

Be careful with:

```text
Highly sensitive data
Frequently changing data
Data requiring strict real-time consistency
Very large objects
Low-read data
Data with expensive invalidation requirements
```

Caching is a trade-off, not a default requirement.

---

# 5. Cache Hit

A cache hit occurs when requested data exists in the cache.

```text
Request
 ↓
Redis
 ↓
FOUND
 ↓
Return data
```

This avoids a database call.

---

# 6. Cache Miss

A cache miss occurs when requested data isn't available.

```text
Request
 ↓
Redis
 ↓
NOT FOUND
 ↓
Database
 ↓
Store in Redis
 ↓
Return
```

---

# 7. Cache Hit Ratio

A useful metric:

```text
Cache Hit Ratio
=
Cache Hits / Total Cache Requests
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

A low hit ratio may indicate poor cache selection, short TTLs, high key churn or incorrect invalidation.

---

# 8. Redis

Redis is an in-memory data store commonly used for:

```text
Caching
Sessions
Distributed locks
Counters
Rate limiting
Queues/streams in some architectures
Pub/Sub
```

Redis also supports persistence options, so describing it simply as "only an in-memory cache" is incomplete.

---

# 9. Why Redis Is Fast

Redis keeps active data in memory and uses efficient data structures.

Common structures include:

```text
String
Hash
List
Set
Sorted Set
Stream
```

---

# 10. Redis String

Example:

```text
product:101
→ "{...product JSON...}"
```

Strings can store:

```text
Text
Numbers
Serialized objects
JSON strings
```

depending on the application.

---

# 11. Redis Hash

Useful for object-like data.

Example:

```text
user:101

name = Sudhir
email = example@example.com
role = USER
```

Hashes can avoid storing every field as a separate top-level key.

---

# 12. Redis List

A list is an ordered collection.

Potential uses:

```text
Simple queues
Recent items
Task lists
```

---

# 13. Redis Set

A set contains unique values.

Useful for:

```text
Tags
Unique IDs
Membership checks
```

---

# 14. Redis Sorted Set

Values are associated with scores.

Useful for:

```text
Leaderboards
Ranking
Priority queues
Time-based ordering
```

---

# 15. Redis Streams

Redis Streams support append-only event-like data structures with consumer-group capabilities.

They can be useful for:

```text
Event processing
Activity feeds
Lightweight streaming workflows
```

For large-scale durable event streaming, Kafka may still be a better fit depending on requirements.

---

# 16. Cache-Aside Pattern

The most common application caching pattern.

Flow:

```text
Application
 ↓
Check Cache
 ↓
Hit? ──YES──> Return
 |
 NO
 ↓
Database
 ↓
Put in Cache
 ↓
Return
```

---

# 17. Cache-Aside Example

```java
Product product = redis.get(key);

if (product == null) {
    product = productRepository.findById(id);
    redis.set(key, product);
}

return product;
```

This is conceptual; production code should also handle serialization, nulls, TTL and failures.

---

# 18. Why Cache-Aside Is Popular

Benefits:

```text
Application controls caching
Simple to understand
Cache failure doesn't necessarily make data unavailable
Only requested data gets cached
```

---

# 19. Cache-Aside Update

Suppose product price changes.

Database:

```text
price = 999
```

Cache may still contain:

```text
price = 799
```

You need an invalidation/update strategy.

---

# 20. Cache Invalidation

One of the hardest caching problems.

A common approach:

```text
Update DB
 ↓
Delete cache
```

Example:

```text
UPDATE product
DELETE product:101
```

Next read:

```text
Cache miss
 ↓
DB
 ↓
New value cached
```

---

# 21. Why Delete Instead of Update?

Deleting after a successful database update is often simpler because the next read loads the authoritative current value.

But there are race conditions, so the exact strategy matters.

---

# 22. Write-Through Cache

Application writes through the cache.

Conceptually:

```text
Application
 ↓
Cache
 ↓
Database
```

The cache layer coordinates persistence.

Benefits:

```text
Cache stays synchronized through the write path
```

Costs:

```text
More complexity
Every write passes through cache
```

---

# 23. Write-Behind

Application writes:

```text
Application
 ↓
Cache
```

and the cache later persists to the database.

Benefits:

```text
Fast writes
```

Risks:

```text
Data loss if durability isn't designed correctly
More complex consistency
Delayed persistence
```

Use carefully.

---

# 24. Read-Through Cache

Application asks the cache for data.

If missing:

```text
Cache
 ↓
Data source
```

The cache itself loads the data.

Compared with cache-aside:

```text
Cache-aside → application handles DB lookup
Read-through → cache layer handles loading
```

---

# 25. Cache-Aside vs Read-Through

| Cache-Aside | Read-Through |
|---|---|
| App checks cache | App asks cache |
| App loads DB on miss | Cache loads source |
| Simple and common | More cache-layer abstraction |
| Application controls logic | Cache controls loading |

---

# 26. TTL

TTL means:

```text
Time To Live
```

Example:

```text
product:101
TTL = 10 minutes
```

After expiration:

```text
Key expires
```

Next request can reload the data.

---

# 27. Why TTL Matters

Without expiration:

```text
Old data
 ↓
stays forever
```

TTL limits how long stale data can remain cached.

But TTL alone does not guarantee perfect consistency.

---

# 28. Choosing TTL

Short TTL:

```text
More fresh
More database/cache misses
```

Long TTL:

```text
Better cache hit ratio
Potentially staler data
```

Choose based on business requirements.

---

# 29. TTL Example

Product catalog:

```text
TTL = 10 minutes
```

Product stock:

```text
Potentially much shorter or not cached
```

depending on the consistency requirement.

---

# 30. Cache Invalidation Strategies

Common approaches:

```text
Delete on update
Update cache on write
TTL expiration
Event-driven invalidation
Versioned keys
```

---

# 31. Event-Driven Cache Invalidation

Example:

```text
Product Service
 ↓
DB update
 ↓
ProductUpdated event
 ↓
Kafka
 ↓
Cache consumer
 ↓
Invalidate product:101
```

Useful when multiple cache-owning components need to react to domain changes.

---

# 32. Cache Consistency

A cache is usually a copy of authoritative data.

Therefore:

```text
Database = source of truth
Cache = performance layer
```

unless your architecture explicitly makes the cache authoritative.

---

# 33. Cache Stampede

A cache stampede happens when a popular key expires and many requests simultaneously miss the cache.

Example:

```text
Popular product
 ↓
Cache expires
 ↓
10,000 requests
 ↓
10,000 DB queries
```

The database gets overloaded.

---

# 34. Preventing Cache Stampede

Possible approaches:

```text
Randomized TTL
Cache warming
Request coalescing
Distributed locking
Early refresh
Stale-while-revalidate
```

---

# 35. Randomized TTL

Instead of:

```text
TTL = 10 minutes
```

use a small random variation:

```text
9m 41s
10m 13s
9m 56s
```

This reduces synchronized expiration across keys.

---

# 36. Cache Warming

Before traffic arrives:

```text
Load popular data
 ↓
Cache
```

Example:

```text
Popular products
Popular categories
```

This reduces cold-cache pressure.

---

# 37. Request Coalescing

If many requests miss the same key:

```text
Request A ─┐
Request B ─┼→ one DB load
Request C ─┘
```

The result is then shared with waiting requests.

This prevents duplicate backend work.

---

# 38. Distributed Lock

Another approach:

```text
Request A
 ↓
Acquire lock
 ↓
Load DB
 ↓
Populate cache
 ↓
Release lock
```

Other requests:

```text
Wait
```

or:

```text
Return stale data
```

depending on the strategy.

---

# 39. Cache Penetration

Cache penetration happens when requests repeatedly query data that does not exist.

Example:

```text
GET /products/999999999
```

Database:

```text
Not found
```

If nothing is cached:

```text
Every request
 ↓
Database
```

---

# 40. Preventing Cache Penetration

Options:

```text
Negative caching
Bloom filters
Input validation
Rate limiting
```

---

# 41. Negative Caching

Cache the fact that a value doesn't exist.

Example:

```text
product:999999
→ NOT_FOUND
TTL = 30 seconds
```

This prevents repeated database queries for the same invalid key.

Use a short TTL and ensure it fits the data's consistency needs.

---

# 42. Bloom Filter

A Bloom filter can quickly indicate:

```text
Definitely not present
```

or:

```text
Possibly present
```

It can reduce unnecessary database queries.

Important:

> Bloom filters can have false positives but should not have false negatives under normal operation.

---

# 43. Cache Avalanche

Cache avalanche refers to a large number of cached keys expiring or becoming unavailable around the same time.

Example:

```text
10,000 keys
 ↓
expire together
 ↓
massive DB traffic
```

---

# 44. Preventing Cache Avalanche

Use:

```text
Randomized TTLs
Cache warming
Staggered expiration
High availability
Request limiting
Graceful degradation
```

---

# 45. Stampede vs Penetration vs Avalanche

Very common interview question.

### Stampede

```text
Popular key expires
→ many requests miss
→ backend overloaded
```

### Penetration

```text
Requests target nonexistent data
→ repeated backend lookups
```

### Avalanche

```text
Many cached keys expire/fail together
→ large backend traffic spike
```

---

# 46. Redis Eviction

Redis can remove keys when memory limits are reached according to its configured eviction policy.

Policies can include strategies based on:

```text
LRU
LFU
TTL
No eviction
```

The exact available policies depend on Redis version/configuration.

---

# 47. LRU

Least Recently Used.

Idea:

```text
Remove keys that haven't been used recently.
```

Useful when recent access is a good predictor of future access.

---

# 48. LFU

Least Frequently Used.

Idea:

```text
Remove keys accessed least often.
```

Useful when frequency is more meaningful than recency.

---

# 49. TTL-Based Eviction

Redis can prefer keys based on expiration metadata.

This can be useful when some cached data has explicit freshness requirements.

---

# 50. Cache Key Design

Good:

```text
product:101
user:20
order:1001
```

Bad:

```text
data1
object2
cache123
```

A clear naming convention improves:

```text
Debugging
Operations
Invalidation
Monitoring
```

---

# 51. Cache Key Versioning

Instead of:

```text
product:101
```

you might use:

```text
product:v2:101
```

When the representation changes:

```text
v3
```

can be introduced.

This can simplify schema changes and avoid incompatible cached values.

---

# 52. Serialization

Java objects must be serialized before being stored in Redis.

Common formats:

```text
JSON
String
Binary formats
```

Choose based on:

```text
Compatibility
Performance
Size
Debuggability
```

---

# 53. JSON vs Java Serialization

Java native serialization can tightly couple cached data to Java implementation details.

JSON is often easier to inspect and more interoperable.

However, JSON can be larger/slower than binary formats.

There is no universal winner.

---

# 54. Cache Data Structure

Don't blindly cache entire entities.

Maybe only cache:

```text
ProductSummary
```

instead of:

```text
Entire Product aggregate
```

Cache what the read path actually needs.

---

# 55. Redis Availability

A single Redis instance can become a single point of failure.

Production deployments may use:

```text
Replication
Sentinel
Cluster
Managed Redis
```

depending on requirements.

---

# 56. Redis Replication

A primary can replicate data to replicas.

Conceptually:

```text
Primary
  |
  +--> Replica 1
  +--> Replica 2
```

Replication can improve availability and read scaling depending on architecture.

---

# 57. Redis Sentinel

Sentinel provides monitoring and automatic failover capabilities for Redis deployments using primary/replica architecture.

Conceptually:

```text
Sentinels
   |
Primary
 |
Replicas
```

---

# 58. Redis Cluster

Redis Cluster distributes data across multiple nodes.

Benefits include:

```text
Horizontal scaling
Data partitioning
Higher capacity
```

It introduces operational considerations such as:

```text
Hash slots
Cross-slot operations
Cluster topology
```

---

# 59. Redis Cluster Hash Slots

Redis Cluster partitions the keyspace into hash slots.

Keys are mapped to slots.

Conceptually:

```text
Key
 ↓
Hash slot
 ↓
Cluster node
```

Keys with hash tags can be intentionally placed in the same slot when needed.

Example:

```text
{order:101}:items
{order:101}:summary
```

can share the same hash tag.

---

# 60. Distributed Lock

Redis can be used to implement distributed coordination.

Conceptually:

```text
SET lock:product:101 unique-token NX EX 10
```

The idea is:

```text
Only one worker obtains the lock
```

The exact implementation must handle expiration, ownership and safe release.

---

# 61. Safe Lock Release

Don't blindly do:

```text
DEL lock
```

A process might accidentally delete another process's lock after its own lock expired and was acquired by someone else.

Use an ownership token and verify it before releasing.

---

# 62. Distributed Lock Use Cases

Potential uses:

```text
Cache stampede protection
Scheduled job coordination
Single-resource processing
Short critical sections
```

Avoid using distributed locks as a substitute for proper transactional design.

---

# 63. Redlock

Redlock is a Redis distributed-locking algorithm designed for multiple independent Redis instances.

It has been debated in the distributed systems community.

For interviews, understand the broader point:

> Distributed locking is difficult, and correctness requirements matter.

Don't claim Redis locking is automatically safe for every distributed coordination problem.

---

# 64. Spring Boot + Redis

Spring Data Redis provides abstractions for Redis.

Common components:

```text
RedisTemplate
StringRedisTemplate
Spring Cache
```

---

# 65. Spring Cache

Spring's caching abstraction can simplify cache usage.

Conceptually:

```java
@Cacheable("products")
public Product getProduct(Long id) {
    return productRepository.findById(id)
        .orElseThrow();
}
```

The first call loads the data.

Later calls can use the cache.

---

# 66. @Cacheable

Conceptually:

```java
@Cacheable(
    value = "products",
    key = "#id"
)
public Product getProduct(Long id) {
    ...
}
```

Meaning:

```text
Cache lookup
 ↓
Hit → return cached value
Miss → execute method
       ↓
     cache result
```

---

# 67. @CachePut

`@CachePut` executes the method and updates the cache with the returned value.

Conceptually:

```java
@CachePut(
    value = "products",
    key = "#product.id"
)
public Product updateProduct(Product product) {
    return repository.save(product);
}
```

Unlike `@Cacheable`, the method still executes.

---

# 68. @CacheEvict

Used to remove cache entries.

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

---

# 69. Cacheable Flow

```text
GET product 101
      |
      ↓
@Cacheable
      |
   +--+--+
   |     |
 HIT    MISS
   |      |
 Return   DB
          |
          ↓
        Cache
          |
          ↓
        Return
```

---

# 70. Cache Eviction on Update

A simple pattern:

```text
Update DB
 ↓
Evict cache
```

Then:

```text
Next GET
 ↓
Cache miss
 ↓
DB
 ↓
Cache new value
```

The exact annotation/transaction ordering should be tested carefully.

---

# 71. Cache and Transactions

A subtle issue:

```text
DB transaction
+
cache update
```

If the DB transaction later rolls back after the cache has already been updated, the cache can contain incorrect data.

Think carefully about:

```text
When cache is updated
When transaction commits
```

---

# 72. Cache Failure

What if Redis is unavailable?

A resilient application may:

```text
Log the cache failure
Fall back to database
Continue serving requests
```

if database load and business requirements permit.

For some systems, cache availability may be critical.

Don't assume cache failures can always be ignored.

---

# 73. Cache Failure Storm

If Redis fails and every request suddenly hits the database:

```text
Redis DOWN
 ↓
Cache MISS for everything
 ↓
Database overload
 ↓
Database DOWN
```

This is why cache failures need:

```text
Rate limiting
Load shedding
Connection limits
Fallback strategy
Monitoring
```

---

# 74. Cache-aside Failure Handling

Suppose:

```text
Cache GET fails
```

Possible approach:

```text
Catch cache failure
 ↓
Read DB
 ↓
Return data
```

Then optionally attempt cache population asynchronously or safely.

Don't let a cache outage automatically take down the business operation unless the architecture requires it.

---

# 75. Cache Consistency Example

Timeline:

```text
T1: DB price = 100
T2: Cache price = 100

T3: Update DB → 200
T4: Cache still = 100
```

There is a stale window.

Possible strategy:

```text
Update DB
 ↓
Evict cache
```

Then:

```text
Next read → DB = 200
```

---

# 76. Race Condition During Cache Invalidation

Consider:

```text
Request A reads old DB value
Request B updates DB
Request B deletes cache
Request A writes old value to cache
```

Now stale data can reappear.

This is why cache consistency can be surprisingly difficult.

Solutions may include:

```text
Versioning
Write ordering
Locks
Delayed double delete
Event-driven invalidation
Careful transaction/cache coordination
```

Choose based on requirements.

---

# 77. Cache-Aside Double Delete

One practical strategy sometimes discussed:

```text
Delete cache
Update DB
Delete cache again
```

The second deletion helps handle certain race conditions.

It is not a universal solution and should be evaluated carefully.

---

# 78. Versioned Cache Values

Store:

```text
value
version
```

Example:

```text
Product 101
version = 8
price = 999
```

A stale writer with:

```text
version = 7
```

can be rejected.

Versioning can help with concurrent updates.

---

# 79. Hot Key

A hot key is a key receiving extremely high traffic.

Example:

```text
product:1
```

receives:

```text
100,000 requests/sec
```

Potential problems:

```text
Single-node pressure
Network bottleneck
Lock contention
```

---

# 80. Hot Key Mitigation

Possible strategies:

```text
Local in-memory cache
Key replication
Request coalescing
Precomputation
Traffic distribution
```

Use carefully because replicating keys complicates invalidation.

---

# 81. Local Cache + Redis

Two-level caching:

```text
Application
 ↓
Local Cache
 ↓ miss
Redis
 ↓ miss
Database
```

Example local cache:

```text
Caffeine
```

Benefits:

```text
Very low latency
Less Redis traffic
```

Costs:

```text
Multiple copies
More invalidation complexity
Potential staleness
```

---

# 82. L1 and L2 Cache

Common model:

```text
L1 = local application cache
L2 = distributed Redis cache
L3 = database
```

Flow:

```text
L1
 ↓ miss
L2
 ↓ miss
DB
```

---

# 83. Cache Warming After Deployment

After deployment:

```text
Cache = empty
```

Traffic causes many misses.

Possible strategy:

```text
Warm popular keys
```

before full traffic arrives.

---

# 84. Cache Invalidation Through Events

Example:

```text
Product Service
 ↓
ProductUpdated
 ↓
Kafka
 ↓
Cache consumers
 ↓
Evict product cache
```

This can be useful when multiple service instances maintain local caches.

---

# 85. Redis Pub/Sub

Redis Pub/Sub can broadcast messages to subscribers.

Example:

```text
Product Service
 ↓
Redis Pub/Sub
 ↓
Application instances
```

Potential use:

```text
Cache invalidation notifications
```

But Pub/Sub is not a durable event log; disconnected subscribers can miss messages.

For durable replayable events, Kafka is often more appropriate.

---

# 86. Redis Streams vs Pub/Sub

### Pub/Sub

```text
Real-time broadcast
No durable replay for disconnected subscribers
```

### Streams

```text
Persistent stream
Consumer groups
Replay/offset-style processing
```

Choose based on delivery requirements.

---

# 87. Redis vs Database

Redis:

```text
Fast
In-memory
Good for cache/state
```

Database:

```text
Durable system of record
Rich queries
Transactions
Long-term storage
```

Do not replace a relational database with Redis simply because Redis is faster.

---

# 88. Redis vs Kafka

Redis:

```text
Fast key/value access
Caching
Counters
Locks
Short-lived state
```

Kafka:

```text
Durable event streaming
Replay
High-throughput event pipelines
Multiple independent consumer groups
```

They solve different problems.

---

# 89. Cache Security

Avoid putting sensitive data into cache without considering:

```text
Encryption
Access controls
Network security
TTL
PII exposure
Logging
```

Also avoid accidentally logging:

```text
JWTs
Passwords
Payment details
Secrets
```

---

# 90. Cache Serialization Compatibility

Suppose version 1 stores:

```text
ProductV1
```

Version 2 expects:

```text
ProductV2
```

Old cache entries may be incompatible.

Solutions:

```text
Versioned keys
Cache flush during controlled deployment
Backward-compatible serialization
```

---

# 91. Cache Observability

Monitor:

```text
Hit ratio
Miss ratio
Latency
Memory
Evictions
Key count
Hot keys
Redis errors
Connection pool
Command latency
```

---

# 92. Cache Metrics Example

Suppose:

```text
Requests = 1,000,000
Hits = 950,000
Misses = 50,000
```

Hit ratio:

```text
95%
```

But don't celebrate blindly.

Also inspect:

```text
DB load
Latency
Stale data
Evictions
```

---

# 93. Cache TTL and Business Requirements

Example:

```text
Product description
TTL = 30 minutes
```

might be fine.

But:

```text
Inventory available quantity
```

may require much stronger consistency.

Never choose TTL only because:

> "10 minutes is standard."

There is no universal TTL.

---

# 94. Cache Consistency Models

Possible approaches:

```text
Best effort
Eventual consistency
Strong invalidation discipline
Versioned reads
Read-through/write-through
```

The right choice depends on business requirements.

---

# 95. Interview Question

### "What is cache-aside?"

Answer:

> "The application first checks the cache. On a miss it reads from the database, returns the result and populates the cache. On updates, the application typically updates the database and invalidates or refreshes the corresponding cache entry."

---

# 96. Interview Question

### "Why use Redis?"

Answer:

> "Redis provides very fast in-memory data access and useful data structures. In microservices it's commonly used for caching, distributed counters, rate limiting, sessions and some coordination use cases."

---

# 97. Interview Question

### "What happens when the cache expires?"

Answer:

> "The next request gets a cache miss and can load the current value from the source of truth, then repopulate the cache. For high-traffic keys, I'd consider stampede protection such as request coalescing, locking or early refresh."

---

# 98. Interview Question

### "What is cache stampede?"

Answer:

> "It happens when a popular cache entry expires and many requests simultaneously miss the cache, causing a sudden burst of requests to the database or backend."

---

# 99. Interview Question

### "Cache penetration vs avalanche?"

Answer:

> "Penetration is repeated requests for data that doesn't exist, so the cache can't help. Avalanche is when many cached entries expire or become unavailable around the same time, causing a large backend traffic spike."

---

# 100. Interview Question

### "How do you prevent stale data?"

Answer:

> "I'd choose an invalidation strategy based on the consistency requirement. Common options are deleting the cache after a successful database update, refreshing the cache, using TTLs, or using domain events for invalidation. For highly consistent data, I would avoid relying on a stale cache."

---

# 101. Interview Question

### "What if Redis goes down?"

Answer:

> "If Redis is a performance layer, I'd consider falling back to the database with appropriate protection so a cache outage doesn't overload it. I'd also monitor Redis failures and database load. If Redis is part of a critical state-management path, the architecture needs stronger availability guarantees."

---

# 102. Interview Question

### "What is cache eviction?"

Answer:

> "Eviction is the removal of keys from the cache, often because the configured memory limit is reached or according to an expiration/eviction policy such as LRU or LFU."

---

# 103. Interview Question

### "LRU vs LFU?"

Answer:

> "LRU removes data that hasn't been accessed recently, while LFU removes data that is accessed least frequently. The better choice depends on the application's access pattern."

---

# 104. Interview Question

### "Can Redis replace MySQL?"

Answer:

> "Usually not. Redis and MySQL solve different problems. MySQL is typically the durable source of truth with relational queries and transactions, while Redis is commonly used as a fast cache or specialized in-memory data store."

---

# 105. Interview Question

### "How do you prevent duplicate cache loading?"

Answer:

> "For a hot key, I'd consider request coalescing or a short-lived distributed lock so only one request loads the missing value while others wait or use stale data if the business allows it."

---

# 106. Interview Question

### "How do you cache a product in Spring Boot?"

Answer:

> "I'd use Spring's caching abstraction with Redis as the cache provider. For example, @Cacheable can cache product reads, @CacheEvict can remove stale entries after updates/deletes, and TTLs can control freshness. I'd also define a clear cache key strategy."

---

# 107. Interview Scenario

### "10,000 users request the same product after its cache expires."

Answer:

> "That's a cache stampede. I'd prevent all requests from hitting the database by using request coalescing, a short-lived distributed lock, early refresh or stale-while-revalidate. I'd also monitor the database and cache load."

---

# 108. Interview Scenario

### "Product price was updated but users still see the old price."

I'd check:

```text
Was cache invalidated?
Did DB transaction commit?
Is there another cache layer?
Is the TTL too long?
Are multiple application instances using local caches?
Is event-based invalidation delayed?
```

Then verify:

```text
DB value
Redis value
L1 cache value
```

---

# 109. Interview Scenario

### "Redis is down and database CPU reaches 100%."

Likely flow:

```text
Redis DOWN
 ↓
Everything becomes cache miss
 ↓
DB receives huge traffic
 ↓
DB overload
```

Mitigation:

```text
Rate limiting
Request coalescing
Load shedding
Connection limits
Graceful degradation
Redis high availability
```

---

# 110. Interview Scenario

### "How would you cache product catalog?"

Possible design:

```text
GET /products/{id}
 ↓
Redis
 ↓ miss
MySQL
 ↓
Redis TTL
 ↓
Response
```

On update:

```text
Update MySQL
 ↓
Evict product key
```

For a high-traffic catalog:

```text
Stampede protection
+
Monitoring
```

---

# 111. Interview Scenario

### "Would you cache inventory?"

Answer:

> "I'd be careful. Inventory is highly dynamic and can affect business correctness. I might cache supporting read information with a very short TTL, but I wouldn't rely on a stale cache for the final inventory reservation decision."

---

# 112. Common Mistakes

```text
❌ Caching everything
❌ No TTL
❌ No invalidation strategy
❌ Treating Redis as the source of truth by default
❌ Ignoring stale data
❌ Ignoring cache stampede
❌ No Redis monitoring
❌ Infinite cache growth
❌ Blindly using distributed locks
❌ Assuming Pub/Sub is durable
❌ Assuming cache failure is harmless
❌ Caching sensitive data carelessly
```

---

# 113. Practical Cache Design Checklist

Before adding Redis, ask:

```text
1. What problem does caching solve?
2. What is the source of truth?
3. What data is cacheable?
4. What is the cache key?
5. What is the TTL?
6. What happens on a cache miss?
7. How is the cache invalidated?
8. What happens if Redis fails?
9. Can stale data be tolerated?
10. Can the key become hot?
11. Can stampede happen?
12. How will duplicate loading be prevented?
13. What eviction policy is appropriate?
14. How will Redis scale?
15. What metrics will be monitored?
```

---

# 114. Final Mental Model

Remember:

```text
Cache
→ Performance layer

Redis
→ Fast distributed in-memory data store

Cache-aside
→ App checks cache, then source

TTL
→ Controls freshness/lifetime

Invalidation
→ Keeps cached state from becoming unnecessarily stale

Stampede
→ Many requests miss the same hot key

Penetration
→ Requests repeatedly ask for nonexistent data

Avalanche
→ Many keys expire/fail together

L1
→ Local cache

L2
→ Distributed cache

Database
→ Usually the source of truth
```

---

# 115. Final Interview Answer

If asked:

> "How would you use Redis in an e-commerce microservices application?"

Use:

> "I'd primarily use Redis as a distributed cache for high-read, relatively stable data such as product details. I'd use a cache-aside approach with clear keys and TTLs, and invalidate or refresh entries when product data changes. For hot keys I'd consider stampede protection, and I'd monitor hit ratio, latency, memory and evictions. I'd treat MySQL as the source of truth and design the application so a Redis failure doesn't automatically overload the database. I could also use Redis for use cases such as rate limiting or short-lived coordination where appropriate."

---

# 116. Revision Checklist

```text
□ Caching
□ Cache hit
□ Cache miss
□ Hit ratio
□ Redis
□ Redis data structures
□ Cache-aside
□ Read-through
□ Write-through
□ Write-behind
□ TTL
□ Cache invalidation
□ Event-driven invalidation
□ Cache consistency
□ Cache stampede
□ Cache penetration
□ Cache avalanche
□ Negative caching
□ Bloom filter
□ Cache warming
□ Request coalescing
□ Distributed lock
□ LRU
□ LFU
□ Redis eviction
□ Redis replication
□ Redis Sentinel
□ Redis Cluster
□ Hash slots
□ Serialization
□ Cache key design
□ Versioned keys
□ Hot keys
□ L1/L2 caching
□ Redis Pub/Sub
□ Redis Streams
□ Redis vs MySQL
□ Redis vs Kafka
□ Spring Cache
□ @Cacheable
□ @CachePut
□ @CacheEvict
□ Cache failure
□ Cache observability
□ Stale data
□ Interview scenarios
```

---

# 117. The Interviewer's Real Test

If asked:

> "Your Redis cache expires during a flash sale and the database suddenly gets 50,000 requests. What would you do?"

Don't just say:

```text
Increase Redis memory.
```

Walk through the failure:

```text
Hot key expires
      ↓
Many simultaneous cache misses
      ↓
Database traffic spikes
      ↓
Protect database
      |
      +--> Request coalescing
      +--> Distributed lock where appropriate
      +--> Rate limiting
      +--> Load shedding
      +--> Stale-while-revalidate
      +--> Cache warming
      ↓
Reload cache safely
      ↓
Monitor DB + Redis
```

The key interview lesson is:

> **Caching is not just about making reads faster. You also need a plan for expiration, invalidation, failures, and sudden traffic.**
