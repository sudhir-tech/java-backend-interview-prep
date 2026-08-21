# System Design — File 05: Caching & Redis

Caching is one of the most important tools for improving backend performance.

The basic idea:

```text
Without cache:

Client
  ↓
Application
  ↓
Database


With cache:

Client
  ↓
Application
  ↓
Redis
  ↓ cache miss
Database
```

The goal is not to cache everything.

The goal is to reduce expensive work where caching provides a meaningful benefit.

---

# 1. What Is a Cache?

A cache stores frequently accessed data so future requests can retrieve it faster.

Example:

```text
Product ID = 101

First request:
App → DB → Product
          ↓
        Redis

Next request:
App → Redis → Product
```

The second request can avoid the database.

---

# 2. Why Use Caching?

Main benefits:

```text
Lower latency
Reduced database load
Higher throughput
Better scalability
Reduced repeated computation
```

---

# 3. Why Is Redis Commonly Used?

Redis is an in-memory data store.

It is commonly used for:

```text
Caching
Sessions
Rate limiting
Counters
Distributed locks
Pub/Sub
Short-lived data
```

Redis also supports richer data structures than a simple key/value string cache.

---

# 4. Cache vs Database

A database is generally:

```text
System of record
```

A cache is generally:

```text
Performance layer
```

Example:

```text
MySQL
→ authoritative product data

Redis
→ frequently accessed copy
```

If Redis disappears:

```text
Application
   ↓
MySQL
```

can potentially rebuild the cache.

This depends on the application's design.

---

# 5. Cache Hit

A cache hit means:

```text
Requested data exists in cache.
```

Flow:

```text
Request
  ↓
Redis
  ↓
Found
  ↓
Return
```

---

# 6. Cache Miss

A cache miss means:

```text
Requested data isn't in cache.
```

Flow:

```text
Request
  ↓
Redis
  ↓
Miss
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
Cache Hits
-------------------------
Cache Hits + Cache Misses
```

Example:

```text
900 hits
100 misses
```

Then:

```text
900 / 1000
=
90%
```

A higher hit ratio generally means more requests are served by the cache.

But a high hit ratio isn't automatically good if the cached data isn't useful or is too stale.

---

# 8. Cache Miss Rate

```text
Miss Rate = 1 - Hit Ratio
```

If:

```text
Hit ratio = 90%
```

then:

```text
Miss rate = 10%
```

---

# 9. Cache-Aside Pattern

One of the most common application caching patterns.

Flow:

```text
Application
    ↓
Check Redis
    ↓
Hit? ── Yes → Return
 |
 No
 ↓
Database
 ↓
Store in Redis
 ↓
Return
```

The application controls when the cache is read and populated.

---

# 10. Cache-Aside Example

Suppose:

```java
Product getProduct(Long id)
```

Conceptually:

```text
1. Check Redis
2. If found → return
3. Query MySQL
4. Put result in Redis
5. Return
```

This is very common in Spring Boot applications.

---

# 11. Read-Through Cache

The application asks the caching layer for data.

Conceptually:

```text
Application
    ↓
Cache
    ↓ miss
Data Store
```

The cache layer itself handles loading missing data.

This differs from cache-aside because the application doesn't directly manage the database lookup in the same way.

---

# 12. Write-Through Cache

Write goes through the cache and is synchronously propagated to the underlying store.

Conceptually:

```text
Application
    ↓
Cache
    ↓
Database
```

This can keep cache and database more closely synchronized.

Trade-off:

```text
More write latency
```

---

# 13. Write-Back / Write-Behind

The application writes to the cache first.

The underlying database is updated asynchronously.

```text
Application
    ↓
Cache
    ↓ async
Database
```

Potential benefit:

```text
Lower write latency
```

Major risk:

```text
Data loss if cache fails before persistence
```

Use only when the consistency/durability requirements support it.

---

# 14. Cache Strategies Summary

```text
Cache-Aside
→ Application manages cache and DB

Read-Through
→ Cache loads data on miss

Write-Through
→ Cache writes through to DB

Write-Back
→ Cache writes first, DB later
```

For many Spring Boot applications:

```text
Cache-Aside
```

is a practical starting point.

---

# 15. What Should You Cache?

Good candidates often have:

```text
High read frequency
Relatively stable data
Expensive database queries
Expensive computation
Frequently repeated requests
```

Examples:

```text
Product details
Configuration
Reference data
Popular articles
Permissions
Exchange rates
```

depending on freshness requirements.

---

# 16. What Should You Avoid Caching?

Be careful with:

```text
Highly volatile data
Large objects
Sensitive data
Data requiring strict freshness
Rarely accessed data
```

Caching should be driven by access patterns and requirements.

---

# 17. Cache Invalidation

One of the classic engineering problems:

> "How do we know when cached data is no longer valid?"

Suppose:

```text
MySQL:
Product price = ₹500

Redis:
Product price = ₹500
```

Then product price changes:

```text
MySQL:
₹600

Redis:
₹500
```

The cache is stale.

---

# 18. TTL

TTL:

```text
Time To Live
```

Example:

```text
TTL = 10 minutes
```

After 10 minutes, the cached entry expires.

This is one simple way to limit staleness.

---

# 19. TTL Trade-off

Short TTL:

```text
Fresher data
More cache misses
More DB traffic
```

Long TTL:

```text
Better cache hit ratio
Potentially stale data
```

Choose based on business requirements.

---

# 20. Explicit Invalidation

When data changes:

```text
Update DB
   ↓
Delete/update cache
```

Example:

```text
PUT /products/101
       ↓
Update MySQL
       ↓
Invalidate Redis key
```

This can reduce stale data.

---

# 21. Update vs Delete Cache

Two common strategies:

```text
Update cache with new value
```

or:

```text
Delete cache entry
```

Then the next read:

```text
Cache miss
 ↓
Database
 ↓
Populate cache
```

Deleting is often simpler when there are many representations of the same data.

---

# 22. Race Conditions During Cache Updates

Imagine:

```text
Request A updates DB
Request B reads DB
Request C updates cache
```

Incorrect ordering can cause:

```text
Old value
```

to be written into the cache after a newer value.

This is why cache consistency needs careful thought in concurrent systems.

---

# 23. Cache Stampede

Suppose:

```text
Popular key
```

expires.

Thousands of requests arrive simultaneously:

```text
Redis → miss
Redis → miss
Redis → miss
Redis → miss
...
```

All hit:

```text
Database
```

The database gets overloaded.

This is called:

```text
Cache stampede
```

---

# 24. Preventing Cache Stampede

Possible techniques:

```text
Request coalescing
Locking
Early refresh
Randomized TTL
Prewarming
Stale-while-revalidate
```

---

# 25. Request Coalescing

Instead of:

```text
1,000 requests
 ↓
1,000 DB queries
```

do:

```text
1,000 requests
      ↓
Single cache rebuild
      ↓
Shared result
```

This prevents duplicate expensive work.

---

# 26. Randomized TTL

Instead of:

```text
Every key:
TTL = exactly 10 minutes
```

use a small random variation:

```text
10m ± random value
```

This prevents many keys from expiring at exactly the same moment.

---

# 27. Cache Penetration

Suppose clients repeatedly request IDs that don't exist:

```text
Product 999999999
Product 999999999
Product 999999999
```

Every request:

```text
Redis miss
 ↓
DB query
 ↓
Not found
```

The DB receives repeated unnecessary queries.

---

# 28. Preventing Cache Penetration

Possible techniques:

```text
Negative caching
Bloom filters
Input validation
Rate limiting
```

Negative caching means temporarily caching:

```text
"Not found"
```

for appropriate keys.

---

# 29. Cache Avalanche

An avalanche can occur when many cache entries expire or become unavailable around the same time.

Result:

```text
Large traffic
 ↓
Cache misses
 ↓
Database overload
```

Potential protections:

```text
Random TTL
Staggered expiration
Warm-up
Multiple cache layers
Rate limiting
Graceful degradation
```

---

# 30. Cache Breakdown

The term is used in different ways, but commonly refers to a very hot key expiring and causing many requests to hit the underlying database simultaneously.

Typical protection:

```text
Locking
Request coalescing
Early refresh
```

---

# 31. Hot Key

A hot key is a cache entry receiving extremely high traffic.

Example:

```text
Product ID 1
```

gets:

```text
50,000 requests/sec
```

Potential problems:

```text
Redis CPU
Network
Single-key concentration
```

---

# 32. Handling Hot Keys

Possible approaches:

```text
Local in-process cache
Replication
Key replication
Request coalescing
CDN
```

The right choice depends on the data and consistency requirements.

---

# 33. Local Cache vs Distributed Cache

### Local cache

Stored inside the application:

```text
App1 → local cache
App2 → local cache
App3 → local cache
```

Advantages:

```text
Very low latency
No network hop
```

Problems:

```text
Each instance has its own copy
Invalidation becomes harder
Memory is limited
```

---

# 34. Distributed Cache

Example:

```text
App1 ─┐
App2 ─┼→ Redis
App3 ─┘
```

Advantages:

```text
Shared cache
Centralized management
Consistent access across instances
```

Trade-offs:

```text
Network latency
Redis becomes infrastructure to operate
Potential cache availability issue
```

---

# 35. Two-Level Cache

Some systems use:

```text
L1 → Local cache
L2 → Redis
L3 → Database
```

Flow:

```text
Request
 ↓
Local cache
 ↓ miss
Redis
 ↓ miss
Database
```

This can reduce Redis traffic but makes invalidation more complex.

---

# 36. Cache Key Design

Good keys should be:

```text
Unique
Predictable
Stable
Easy to invalidate
```

Example:

```text
product:101
user:42
order:9001
```

Avoid ambiguous keys:

```text
101
42
9001
```

Prefixes improve clarity.

---

# 37. Key Namespaces

Example:

```text
product:101
product:102

user:101

order:101
```

This prevents collisions and makes operational debugging easier.

---

# 38. Cache Serialization

Objects must be represented in a format Redis can store.

Possible formats:

```text
JSON
String
Hash
Binary serialization
```

Consider:

```text
Size
Performance
Compatibility
Schema evolution
Security
```

---

# 39. Redis Data Structures

Redis supports several useful structures.

```text
String
Hash
List
Set
Sorted Set
Stream
```

Use the structure that matches the workload.

---

# 40. Redis String

Example:

```text
product:101
→ "{...}"
```

Useful for:

```text
Simple values
JSON blobs
Counters
Tokens
```

---

# 41. Redis Hash

Conceptually:

```text
user:42
  name = Sudhir
  role = USER
```

Useful when fields need to be accessed individually.

---

# 42. Redis Set

A Set stores unique values.

Example:

```text
online-users
```

Could contain:

```text
user1
user2
user3
```

No duplicates.

---

# 43. Redis Sorted Set

Stores members with scores.

Useful for:

```text
Leaderboards
Ranking
Priority-like ordering
Time-based indexes
```

Example:

```text
user1 → 1500
user2 → 1200
user3 → 1800
```

---

# 44. Redis List

Useful for:

```text
Queues
Ordered collections
Simple task lists
```

For large-scale event streaming, specialized systems such as Kafka may be more appropriate depending on requirements.

---

# 45. Redis Streams

Redis Streams provide an append-oriented data structure for event-like data and consumer processing.

Useful for certain:

```text
Event processing
Consumer groups
Message workflows
```

Don't automatically choose Streams over Kafka; their operational and delivery characteristics differ.

---

# 46. Redis Pub/Sub

Pub/Sub allows publishers to send messages to subscribers.

```text
Publisher
   ↓
Redis Channel
  / \
 ↓   ↓
S1   S2
```

Messages are generally transient.

If durable event history is required:

```text
Kafka
```

or another durable messaging system may be more appropriate.

---

# 47. Redis for Rate Limiting

Example:

```text
User:
42
```

Counter:

```text
rate:user:42
```

Conceptually:

```text
INCR
EXPIRE
```

Then:

```text
100 requests
→ allow

101st
→ reject
```

Production rate limiting needs careful handling of atomicity, windows, distributed instances and edge cases.

---

# 48. Redis for Sessions

Instead of:

```text
App memory
```

use:

```text
Redis
```

Example:

```text
session:abc123
```

Multiple app instances can access it.

---

# 49. Redis Distributed Lock

A distributed lock can coordinate work across multiple application instances.

Conceptually:

```text
App1 → acquire lock
App2 → lock unavailable
```

Use carefully.

Distributed locks have tricky failure modes.

Don't use them as a replacement for proper database transactions or idempotency when those are more appropriate.

---

# 50. Cache Consistency

Important question:

> "How consistent must the cache be with the database?"

Possible models:

```text
Best effort
Eventual consistency
Near-real-time
Strong coordination
```

Choose based on business requirements.

---

# 51. Cache and Transactions

Suppose:

```text
DB update
Cache update
```

These are two different systems.

A transaction in MySQL does not automatically make Redis transactional with it.

This creates consistency challenges.

---

# 52. Common Safe Pattern

For many read-heavy applications:

```text
1. Update DB
2. Invalidate cache
```

Then:

```text
Next read
 ↓
Cache miss
 ↓
DB
 ↓
Repopulate cache
```

This is simple, but race conditions and concurrent updates still need consideration.

---

# 53. Why "DB First" Is Common

If the database is the source of truth:

```text
DB first
```

helps avoid a situation where:

```text
Cache says new value
DB still says old value
```

The exact ordering should depend on the consistency model.

---

# 54. Cache Failure

What if Redis goes down?

A resilient application might:

```text
Redis unavailable
 ↓
Fallback to DB
```

But this can create a sudden database load spike.

So consider:

```text
Timeout
Circuit breaker
Rate limiting
Local cache
Load shedding
```

---

# 55. Cache Failure Scenario

Normal:

```text
100K requests/sec
90% cache hit
```

DB receives:

```text
10K/sec
```

Redis fails.

Now potentially:

```text
100K/sec → DB
```

This can overload MySQL.

This is why cache failure is a system-design problem, not merely a Redis problem.

---

# 56. Cache Warming

Before serving traffic:

```text
Load frequently accessed data
```

into the cache.

Example:

```text
Top 10,000 products
```

This can reduce cold-start cache misses.

---

# 57. Cache Prewarming

Useful after:

```text
Redis restart
Deployment
Failover
Cache flush
```

But don't blindly preload massive datasets.

---

# 58. Eviction Policies

Redis has limited memory.

When memory is full, entries may be evicted according to the configured policy.

Conceptually:

```text
Memory limit reached
      ↓
Evict entries
```

Common policies include:

```text
allkeys-lru
allkeys-lfu
volatile-lru
noeviction
```

The right policy depends on workload.

---

# 59. LRU vs LFU

### LRU

Least Recently Used.

Evicts data that hasn't been accessed recently.

### LFU

Least Frequently Used.

Evicts data that is accessed less often.

For workloads with very hot keys, LFU can sometimes be useful.

---

# 60. TTL + Eviction

These solve different problems.

```text
TTL
→ Time-based expiration

Eviction
→ Memory-pressure behavior
```

You often need both.

---

# 61. Redis Persistence

Redis can support persistence mechanisms such as:

```text
RDB
AOF
```

But if Redis is being used purely as a cache:

```text
Loss of cached data
```

may be acceptable.

If Redis stores important state:

```text
Persistence
Backup
Recovery
Replication
```

become much more important.

---

# 62. Redis High Availability

A production Redis architecture may use:

```text
Replication
Sentinel
Cluster
Managed Redis
```

These solve different problems.

---

# 63. Redis Cluster

Redis Cluster distributes data across multiple nodes.

Useful for:

```text
Horizontal data scaling
Higher capacity
Partitioning
```

But it introduces:

```text
Operational complexity
Cluster-aware clients
Key distribution considerations
```

---

# 64. Redis Replication

Conceptually:

```text
Primary
  |
  +--- Replica
  +--- Replica
```

Useful for:

```text
Read scaling
Redundancy
Failover
```

Exact consistency behavior depends on the Redis architecture and client configuration.

---

# 65. Cache vs Redis Interview Trap

Redis is:

```text
A technology
```

Caching is:

```text
An architectural pattern/use case
```

You can cache with:

```text
Local memory
Redis
Memcached
CDN
Database buffers
```

Redis is not synonymous with caching.

---

# 66. Redis vs Memcached

High-level comparison:

```text
Redis
→ Rich data structures
→ Persistence options
→ Replication/cluster capabilities
→ Pub/Sub/Streams

Memcached
→ Simple distributed cache
→ Simple key/value model
→ Often chosen for straightforward caching
```

The best choice depends on requirements and managed-service options.

---

# 67. When Not to Use Redis

Don't introduce Redis when:

```text
Database already meets latency requirements
Data is rarely reused
Cache invalidation is too complex
Dataset is too large for the available memory
Operational complexity isn't justified
```

Start simple.

---

# 68. Cache Security

Be careful with:

```text
Authentication tokens
PII
Sensitive customer data
Passwords
Payment information
```

If sensitive data must be cached:

```text
Encryption
Access control
Network isolation
TTL
Data minimization
```

should be considered.

Never store plaintext passwords.

---

# 69. Cache Stampede Scenario

Suppose:

```text
Popular product
TTL = 10 minutes
```

At exactly:

```text
12:00
```

the entry expires.

At:

```text
12:00:00
```

10,000 requests arrive.

All miss.

Result:

```text
10,000 DB queries
```

Potential solution:

```text
Lock
+
single rebuild
```

---

# 70. Stale-While-Revalidate

Return stale data temporarily while refreshing in the background.

Conceptually:

```text
Request
 ↓
Stale cache exists
 ↓
Return stale value
 +
Refresh asynchronously
```

This can reduce latency spikes.

Use only when slightly stale data is acceptable.

---

# 71. Cache Invalidation Strategies

Common choices:

```text
TTL
Explicit delete
Explicit update
Event-driven invalidation
Versioned keys
```

Example:

```text
Product updated
 ↓
ProductUpdated event
 ↓
Consumers invalidate product cache
```

This becomes useful in distributed systems.

---

# 72. Versioned Cache Keys

Instead of:

```text
product:101
```

you can use:

```text
product:v2:101
```

This can help during schema changes or bulk invalidation.

---

# 73. Cache Metrics

Monitor:

```text
Hit ratio
Miss ratio
Latency
Memory usage
Evictions
Expired keys
Commands/sec
Connections
CPU
Network
Hot keys
```

Without metrics, cache tuning becomes guesswork.

---

# 74. Cache Observability

For a request:

```text
GET /products/101
```

use metrics/logging to understand:

```text
Cache hit?
Cache miss?
Redis latency?
DB latency?
Total latency?
```

This helps identify whether caching is actually improving the system.

---

# 75. Spring Boot + Redis

Spring Boot can integrate with Redis using Spring Data Redis.

Conceptually:

```text
Controller
   ↓
Service
   ↓
Redis
```

Possible approaches include:

```text
RedisTemplate
Spring Cache
@Cacheable
@CacheEvict
@CachePut
```

---

# 76. `@Cacheable`

Conceptually:

```java
@Cacheable("products")
public Product getProduct(Long id) {
    return repository.findById(id).orElseThrow();
}
```

The exact key/configuration should be designed carefully.

---

# 77. `@CacheEvict`

When product changes:

```java
@CacheEvict(value = "products", key = "#id")
```

This can invalidate the corresponding cache entry.

Again, real applications need to consider transaction ordering and concurrent updates.

---

# 78. `@CachePut`

`@CachePut` executes the method and updates the cache with the returned result.

It is different from `@Cacheable`, which can skip method execution on a cache hit.

---

# 79. Spring Cache Abstraction

Spring's caching abstraction can let application code use caching without tightly coupling every method to Redis-specific APIs.

Conceptually:

```text
Service
 ↓
Spring Cache
 ↓
Redis
```

This can improve flexibility.

---

# 80. Cache Key Example

For:

```text
Product ID = 101
```

a key might be:

```text
products::101
```

The exact key prefix depends on your cache configuration.

---

# 81. Interview Question

### Why use Redis?

Answer:

> "Redis provides very low-latency in-memory access and supports useful data structures. In backend systems I would commonly use it for caching, sessions, rate limiting and some coordination use cases."

---

# 82. Interview Question

### What is cache-aside?

Answer:

> "The application first checks the cache. On a miss it reads from the database, stores the result in the cache and returns it."

---

# 83. Interview Question

### What is cache invalidation?

Answer:

> "It's the process of removing or updating cached data when the underlying source changes, so stale values aren't served beyond the allowed consistency window."

---

# 84. Interview Question

### What is cache stampede?

Answer:

> "It's when many requests miss or simultaneously expire the same cache entry and all hit the underlying database, causing a sudden load spike."

---

# 85. Interview Question

### How would you prevent a cache stampede?

Answer:

> "I could use request coalescing or a lock so only one request rebuilds the value, while other requests wait or use stale data. Randomized TTL and early refresh can also reduce synchronized expiration."

---

# 86. Interview Question

### What happens if Redis goes down?

Answer:

> "It depends on the system. If Redis is only a cache, the application can potentially fall back to the database, but I would protect the database with timeouts, rate limiting and possibly graceful degradation because a cache outage can create a sudden database load spike."

---

# 87. Interview Question

### Redis vs MySQL?

Answer:

> "MySQL is generally the durable source of truth for relational business data, while Redis is commonly used as a fast in-memory layer for frequently accessed or temporary data."

---

# 88. Interview Question

### Why not cache everything?

Answer:

> "Caching consumes memory and introduces invalidation and consistency complexity. I cache data where the performance or database-load benefit justifies that complexity."

---

# 89. Interview Question

### What is TTL?

Answer:

> "TTL is the time-to-live of a cache entry. After the configured period, the entry expires and usually needs to be loaded again."

---

# 90. Interview Question

### What is a cache hit ratio?

Answer:

> "It's the percentage of cache requests that are served directly from the cache instead of falling back to the underlying data store."

---

# 91. Interview Question

### Local cache vs Redis?

Answer:

> "A local cache has extremely low latency but each application instance has its own copy. Redis provides a shared distributed cache, but introduces a network hop and its own availability and operational considerations."

---

# 92. Interview Question

### Can Redis replace a database?

Answer:

> "It can store persistent data in some architectures, but I wouldn't treat Redis as a replacement for a relational database by default. The choice depends on durability, query requirements, consistency and access patterns."

---

# 93. Practical Scenario

### Product API is slow.

Metrics:

```text
DB latency = 200 ms
Redis = not used
```

Potential improvement:

```text
Cache frequently requested product data.
```

But first verify:

```text
Query plan
Indexes
Access pattern
Freshness requirement
```

---

# 94. Practical Scenario

### Redis hit ratio is 95%, but DB is still overloaded.

Investigate:

```text
Which endpoints miss?
Are misses expensive?
Are writes overwhelming DB?
Are some queries uncached?
Is cache traffic concentrated?
Are replicas needed?
```

A high overall hit ratio doesn't guarantee every expensive workload is protected.

---

# 95. Practical Scenario

### Redis fails and DB CPU reaches 100%.

This is a classic cache dependency failure.

Potential protections:

```text
Circuit breaker
Request limiting
Local fallback cache
Stale data
Autoscaling DB where appropriate
Graceful degradation
```

---

# 96. Practical Scenario

### Product price changed but users still see the old price.

Possible causes:

```text
TTL too long
Cache invalidation failed
Update ordering issue
Multiple cache layers
Replica lag
```

Investigate the entire read path.

---

# 97. Practical Scenario

### Redis memory is full.

Check:

```text
Cache size
TTL
Eviction policy
Large keys
Hot keys
Unbounded data
```

Then decide:

```text
Evict
Increase capacity
Reduce TTL
Reduce object size
Remove unnecessary cached data
```

---

# 98. Practical Scenario

### One Redis key gets enormous traffic.

This is:

```text
Hot key
```

Potential solutions:

```text
Local cache
Replication
Request coalescing
CDN
Key replication
```

Choose based on consistency and workload.

---

# 99. E-commerce Example

For your e-commerce backend:

```text
Client
  ↓
Load Balancer
  ↓
Spring Boot
  ↓
Redis
  ↓ cache miss
MySQL
```

Good candidates:

```text
Product details
Product categories
Frequently accessed reference data
```

Potentially avoid caching:

```text
Highly sensitive transactional state
Data requiring strict freshness
```

unless the consistency model is explicitly designed.

---

# 100. E-commerce Product Update

Possible flow:

```text
Admin updates product
        ↓
Update MySQL
        ↓
Invalidate Redis
        ↓
Next customer request
        ↓
Redis miss
        ↓
MySQL
        ↓
Redis populated
```

This is simple cache-aside behavior.

---

# 101. E-commerce Order Data

Be more careful with:

```text
Order status
Payment status
Inventory
```

because stale data can have business consequences.

Caching may still be useful, but the consistency requirements must be explicit.

---

# 102. Redis in Your Backend Interview

You should be able to explain:

```text
Why Redis?
What do you cache?
What is TTL?
What happens on cache miss?
How do you invalidate?
What if Redis fails?
What is cache stampede?
What is a hot key?
How do you monitor Redis?
```

---

# 103. Cache Design Checklist

Before adding a cache, ask:

```text
□ Is this data read frequently?
□ Is it expensive to retrieve?
□ Can it tolerate some staleness?
□ What is the TTL?
□ How is invalidation handled?
□ What happens on a cache miss?
□ What happens if Redis fails?
□ How large is the cache?
□ What is the eviction policy?
□ Could a hot key occur?
□ Could a stampede occur?
□ How will we monitor it?
```

---

# 104. Final Architecture

A common read-heavy backend:

```text
                 Client
                   ↓
             Load Balancer
                   ↓
             Spring Boot
                   ↓
                Redis
               /     \
          Cache Hit   Miss
              ↓        ↓
           Response   MySQL
                         ↓
                    Populate Redis
                         ↓
                      Response
```

---

# 105. Final Mental Model

Remember:

```text
CACHE
  ↓
Fast
  ↓
But temporary
  ↓
Needs invalidation
  ↓
Needs memory management
  ↓
Can fail
  ↓
Can overload the DB when it fails
```

Redis is powerful, but every cache introduces another system you need to operate and reason about.

---

# 106. One-Minute Interview Answer

### "How would you use Redis in an e-commerce system?"

> "I'd use Redis primarily for frequently accessed, relatively stable data such as product details or reference data. I'd use a cache-aside pattern: check Redis first, query MySQL on a miss, then populate Redis with an appropriate TTL. For updates I'd invalidate or update the relevant cache entry. I'd also monitor hit ratio, latency and memory, and protect MySQL against a Redis outage because a cache failure can suddenly increase database traffic."

---

# 107. Key Takeaway

> **Caching is a performance optimization, not a replacement for the source of truth. The hard part isn't putting data into Redis; it's deciding what to cache, how long to keep it, how to invalidate it, and what happens when the cache is stale, overloaded or unavailable.**

**File 05 complete.**
