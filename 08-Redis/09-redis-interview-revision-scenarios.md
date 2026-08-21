# Redis — File 09: Interview Revision & Scenario-Based Questions

This is the final Redis interview revision file.

Use it after completing Files 01–08.

---

# 1. Redis in One Minute

Redis is an in-memory data store commonly used for:

```text
Caching
Session storage
Counters
Rate limiting
Distributed locks
Idempotency
Pub/Sub
Streams
Queues
Fast temporary state
```

Its main strengths are:

```text
Low latency
High throughput
Rich data structures
Atomic operations
TTL
Distributed coordination
```

---

# 2. Redis Mental Model

Remember:

```text
Data structures
    ↓
Atomic operations
    ↓
TTL / eviction
    ↓
Caching
    ↓
Distributed coordination
    ↓
Messaging
    ↓
Replication / HA
    ↓
Persistence
    ↓
Production operations
```

---

# 3. Top Redis Interview Topics

Be comfortable with:

```text
1. Redis vs database
2. Redis data structures
3. TTL
4. Eviction
5. Cache-aside
6. Cache invalidation
7. Cache stampede
8. RedisTemplate
9. @Cacheable
10. Distributed locks
11. SET NX EX
12. Lua
13. Rate limiting
14. Pub/Sub
15. Streams
16. Consumer groups
17. XACK
18. Idempotency
19. Replication
20. Sentinel
21. Cluster
22. Hash slots
23. MOVED / ASK
24. RDB / AOF
25. Monitoring
```

---

# 4. Question: What Is Redis?

Answer:

> "Redis is an in-memory data store that supports multiple data structures such as strings, hashes, lists, sets and sorted sets. It's commonly used for caching, counters, distributed coordination, rate limiting and messaging."

---

# 5. Question: Why Is Redis Fast?

Answer:

> "Redis primarily operates in memory, uses efficient data structures and has a lightweight request-processing model. This allows very low-latency operations compared with disk-based databases for many workloads."

---

# 6. Question: Redis vs MySQL?

Answer:

> "Redis is optimized for fast in-memory access and temporary or distributed state, while MySQL provides durable relational storage, transactions and strong data integrity. I would typically use Redis alongside MySQL rather than treating them as direct replacements."

---

# 7. Question: Why Use Redis as a Cache?

Answer:

> "Redis can reduce database load and improve response latency by storing frequently accessed data closer to the application."

---

# 8. Question: What Is Cache-Aside?

Answer:

> "The application first checks Redis. On a cache miss, it reads from the database, stores the result in Redis with an appropriate TTL, and returns it."

Flow:

```text
Request
 ↓
Redis
 ↓ miss
Database
 ↓
Redis
 ↓
Response
```

---

# 9. Question: What Happens When Cache Expires?

Answer:

```text
Cache miss
 ↓
Read database
 ↓
Repopulate cache
 ↓
Return response
```

But for a hot key, many requests can miss simultaneously.

That can create a cache stampede.

---

# 10. Question: What Is Cache Stampede?

Answer:

> "A cache stampede happens when many requests miss the same cache entry at roughly the same time and all hit the database."

Solutions include:

```text
Request coalescing
Distributed lock
TTL jitter
Refresh ahead
Stale-while-revalidate
```

---

# 11. Question: What Is TTL?

Answer:

> "TTL is the amount of time a Redis key is allowed to remain before expiration."

Example:

```text
SET user:101 value EX 300
```

means:

```text
Expire after 300 seconds
```

---

# 12. Question: What Is Eviction?

Answer:

> "Eviction is Redis removing keys when memory reaches the configured limit according to the selected eviction policy."

Examples:

```text
allkeys-lru
allkeys-lfu
volatile-ttl
noeviction
```

---

# 13. Question: LRU vs LFU?

Answer:

> "LRU favors keys that haven't been used recently, while LFU favors keys that are accessed frequently. LFU can be useful when popularity is more important than recency."

---

# 14. Question: What Is RedisTemplate?

Answer:

> "`RedisTemplate` is a Spring Data Redis abstraction that provides Java operations for interacting with Redis data structures."

Examples:

```java
redisTemplate.opsForValue()
redisTemplate.opsForHash()
redisTemplate.opsForList()
redisTemplate.opsForSet()
redisTemplate.opsForZSet()
```

---

# 15. Question: What Is @Cacheable?

Answer:

> "`@Cacheable` tells Spring to check the cache before executing the method. If the value exists, the method can be skipped; otherwise the method executes and the result is cached."

---

# 16. Question: @CachePut vs @CacheEvict?

Answer:

```text
@CachePut
→ Executes method and updates cache

@CacheEvict
→ Removes cached entry
```

---

# 17. Question: How Do You Design a Redis Cache Key?

Answer:

> "I use predictable namespaces and include the relevant identifier, for example `product:101` or `user:101:profile`. For versioned data I may include a schema or cache version."

---

# 18. Question: What Is Cache Invalidation?

Answer:

> "Cache invalidation means removing or updating cached data when the underlying source of truth changes."

Common approaches:

```text
TTL
Explicit eviction
Write-through
Event-based invalidation
Versioned keys
```

---

# 19. Question: How Do You Prevent Stale Data?

Answer:

> "I'd choose the consistency strategy based on business requirements. Options include shorter TTLs, explicit invalidation, updating the cache during writes, versioned keys, or reading critical data directly from the source of truth."

---

# 20. Question: What Is a Redis Distributed Lock?

Answer:

> "It's a coordination mechanism that allows multiple application instances to agree that only one instance should execute a critical section at a time."

---

# 21. Question: How Do You Acquire a Redis Lock?

Answer:

> "I'd use an atomic command such as `SET lockKey uniqueToken NX EX ttl`. `NX` prevents overwriting an existing lock and `EX` provides a lease expiration."

---

# 22. Question: Why Use a Unique Token?

Answer:

> "The token identifies the lock owner. When releasing the lock, I can verify that the stored token matches mine before deleting it."

---

# 23. Question: Why Can't You Just DEL the Lock?

Answer:

> "The lock may have expired and another instance may have acquired it. The old instance could then delete the new owner's lock. Ownership verification and deletion should therefore be atomic."

---

# 24. Question: What Is a Fencing Token?

Answer:

> "A fencing token is a monotonically increasing ownership value that the protected resource can use to reject operations from stale lock holders."

---

# 25. Question: Does a Redis Lock Guarantee Correctness?

Answer:

> "Not by itself. Processes can pause, locks can expire and networks can fail. For critical operations I'd combine locking with idempotency, durable constraints and, where needed, fencing tokens."

---

# 26. Question: What Is Redis Rate Limiting?

Answer:

> "Redis can maintain counters or timestamps to limit how many requests a user, API key or IP can make within a defined period."

---

# 27. Fixed Window Rate Limiting

Example:

```text
100 requests / minute
```

Maintain a Redis counter:

```text
rate:user:101:2026082113
```

Increment it atomically.

If:

```text
count > 100
```

return:

```text
HTTP 429
```

---

# 28. Fixed Window Problem

The boundary problem:

```text
12:00:59
100 requests

12:01:00
100 requests
```

Potentially:

```text
200 requests
```

in approximately one second.

---

# 29. Sliding Window

A sliding window considers requests over the most recent period rather than fixed calendar boundaries.

Redis Sorted Sets can store:

```text
timestamp
+
request ID
```

Then old entries can be removed and recent requests counted.

---

# 30. Token Bucket

Token bucket maintains:

```text
Capacity
Refill rate
Available tokens
```

Each request consumes a token.

It supports:

```text
Controlled sustained traffic
+
bounded bursts
```

---

# 31. Question: Why Use Lua?

Answer:

> "Lua lets me execute multiple Redis operations as one server-side atomic script. It's useful when a business rule involves multiple reads and writes that must not interleave with concurrent operations."

---

# 32. Example Lua Use Cases

```text
Conditional inventory decrement
Atomic rate limiting
Safe lock release
Multi-step state transition
```

---

# 33. Question: What Is Redis Pub/Sub?

Answer:

> "Pub/Sub is a real-time messaging mechanism where publishers send messages to channels and active subscribers receive them."

---

# 34. Pub/Sub Limitation

Answer:

> "Pub/Sub is ephemeral. A subscriber that is disconnected when the message is published can miss the message."

Therefore:

```text
Pub/Sub
→ Real-time transient communication

Streams
→ Durable event processing
```

---

# 35. Question: What Are Redis Streams?

Answer:

> "Redis Streams are an append-oriented data structure for event processing. They support persistent entries, consumer groups, acknowledgements and pending-message tracking."

---

# 36. Question: What Is XADD?

Answer:

> "`XADD` appends an entry to a Redis Stream."

Example:

```text
XADD orders * orderId 101 status CREATED
```

---

# 37. Question: What Is a Consumer Group?

Answer:

> "A consumer group allows multiple consumers to share processing of a stream. Redis tracks which consumer received entries and maintains pending entries until they're acknowledged."

---

# 38. Question: What Is XACK?

Answer:

> "`XACK` acknowledges successful processing of a stream entry."

Important:

```text
Process
 ↓
Durable side effect
 ↓
XACK
```

---

# 39. Why Not XACK First?

Bad:

```text
Receive
 ↓
XACK
 ↓
Database update
 ↓
Crash
```

The event may be considered processed even though the business operation failed.

---

# 40. Question: Why Do Stream Consumers Need Idempotency?

Answer:

> "A consumer can successfully commit a database change and then crash before XACK. Redis can redeliver the message, so the consumer must safely handle duplicates."

---

# 41. Question: What Is At-Least-Once Delivery?

Answer:

> "The system prioritizes not losing messages, but a message can be delivered more than once after failures."

Therefore:

```text
At-least-once
+
Idempotent consumer
```

is a common production pattern.

---

# 42. Question: What Is a Poison Message?

Answer:

> "A poison message is an event that repeatedly fails processing, potentially causing an endless retry loop."

Solution:

```text
Bounded retries
+
Backoff
+
Dead-letter stream
```

---

# 43. Question: What Is Redis Replication?

Answer:

> "Redis replication maintains copies of a primary's data on replica nodes. It can improve availability and support read scaling."

---

# 44. Is Replication Synchronous?

Answer:

> "Redis replication is generally asynchronous, so replicas can temporarily lag behind the primary."

---

# 45. Question: Can Replica Reads Be Stale?

Answer:

> "Yes. Because replication is asynchronous, a replica may not have received the latest write yet."

---

# 46. Question: What Is Sentinel?

Answer:

> "Redis Sentinel provides monitoring, primary discovery and automatic failover for Redis primary-replica deployments."

---

# 47. Question: Does Sentinel Shard Data?

Answer:

> "No. Sentinel provides high availability and failover but does not shard the dataset."

---

# 48. Question: What Is Redis Cluster?

Answer:

> "Redis Cluster distributes keys across multiple primary nodes using 16,384 hash slots and can provide high availability through replicas."

---

# 49. Question: Sentinel vs Cluster?

Answer:

```text
Sentinel
→ HA / failover
→ No sharding

Cluster
→ Sharding
→ HA
→ Horizontal scaling
```

---

# 50. Question: What Is a Hash Slot?

Answer:

> "Redis Cluster divides the keyspace into 16,384 hash slots. Each key maps to one slot, and slots are assigned to cluster primary nodes."

---

# 51. Question: What Is a Hash Tag?

Answer:

> "A hash tag lets related keys use the same portion of the key for slot calculation, allowing them to be colocated on the same cluster slot."

Example:

```text
user:{101}:profile
user:{101}:cart
```

---

# 52. Question: MOVED vs ASK?

Answer:

```text
MOVED
→ Slot ownership is at another node

ASK
→ Temporary redirection during slot migration
```

---

# 53. Question: Why Can Multi-Key Commands Be Difficult in Cluster?

Answer:

> "Different keys can map to different hash slots and therefore different nodes. Redis Cluster can't treat arbitrary cross-slot keys as one local atomic operation."

Use hash tags when the data model logically requires co-location.

---

# 54. Question: RDB vs AOF?

Answer:

> "RDB is snapshot-based persistence. AOF records write operations. RDB is convenient for snapshots and backups, while AOF can provide more frequent durability depending on configuration."

---

# 55. Question: Do Replicas Replace Backups?

Answer:

> "No. Replicas improve availability, but logical mistakes and accidental deletions can propagate to replicas. Separate backups are required for disaster recovery."

---

# 56. Question: How Do You Secure Redis?

Answer:

> "I'd keep Redis on a private network, use authentication and ACLs, apply least privilege, use TLS where required, protect administrative commands, and store credentials in a secret manager."

---

# 57. Question: Why Avoid KEYS *?

Answer:

> "`KEYS *` can scan the entire keyspace and cause significant blocking on large production datasets. For incremental keyspace iteration I'd use `SCAN`."

---

# 58. Question: What Is a Big Key?

Answer:

> "A big key is a Redis key whose value or collection contains a large amount of data. Big keys can increase memory usage, latency and operational risk."

---

# 59. Question: What Is a Hot Key?

Answer:

> "A hot key receives disproportionately high traffic and can become a bottleneck even when the overall Redis cluster has enough capacity."

---

# 60. Question: What Should You Monitor?

Answer:

```text
Memory
CPU
Network
Latency
P95/P99
Connections
Evictions
Expirations
Hit rate
Replication lag
Persistence
Streams
Pending entries
Consumer lag
Errors
Failovers
```

---

# 61. Scenario: Redis Goes Down

### If Redis is only a cache

```text
Redis unavailable
 ↓
Controlled database fallback
```

But protect MySQL with:

```text
Circuit breaker
Rate limiting
Load shedding
Request prioritization
```

### If Redis holds critical state

Use:

```text
HA
Persistence
Backups
Recovery plan
```

---

# 62. Scenario: Database CPU Suddenly Reaches 100%

Redis may have failed.

Check:

```text
Cache hit rate
Redis availability
Redis latency
Application fallback traffic
Database query volume
```

Potential cause:

```text
Cache failure
→ Massive cache misses
→ Database overload
```

---

# 63. Scenario: Redis Latency Increased

Check:

```text
CPU
Memory
Network
Slow commands
Big keys
Hot keys
Lua scripts
Persistence
Connections
Application pool
```

Don't assume Redis itself is the only cause.

---

# 64. Scenario: Cache Hit Rate Dropped

Check:

```text
TTL
Evictions
Key format
Deployment
Invalidation
Redis restart
Traffic pattern
Serialization
```

---

# 65. Scenario: One Redis Cluster Node Is Hot

Check:

```text
Slot distribution
Hot keys
Hot slots
Hash tags
Large values
Traffic distribution
```

Adding another node may not fix a single hot key.

---

# 66. Scenario: Duplicate Stream Processing

Possible flow:

```text
Process DB
 ↓
Crash
 ↓
No XACK
 ↓
Redelivery
```

Solution:

```text
Idempotent consumer
+
event ID
+
durable uniqueness
```

---

# 67. Scenario: Cache Stampede

Problem:

```text
Popular key expires
 ↓
1000 requests miss
 ↓
1000 DB queries
```

Solutions:

```text
TTL jitter
Request coalescing
Distributed lock
Refresh ahead
Stale-while-revalidate
```

---

# 68. Scenario: Overselling Inventory

Bad:

```text
GET stock
 ↓
if stock > 0
 ↓
DECR
```

Concurrent requests can race.

Better:

```text
Atomic Redis operation
or
Lua script
```

Then use the durable database transaction as the final business correctness boundary.

---

# 69. Scenario: Duplicate Payment

Use:

```text
Idempotency-Key
```

Example:

```text
payment:request:abc123
```

Store:

```text
PROCESSING
COMPLETED
FAILED
```

For critical payments, also rely on the payment provider's idempotency support or reconciliation strategy.

---

# 70. Scenario: Scheduled Job Runs Twice

Possible solution:

```text
Redis lease
+
idempotent job
```

Acquire:

```text
SET job:daily-report token NX EX 60
```

But don't assume the lock alone guarantees exactly one execution.

---

# 71. Scenario: Lock Holder Crashes

With:

```text
EX 30
```

the lock eventually expires.

Another instance can acquire it.

But the operation should be safe if the old process resumes unexpectedly.

Use:

```text
Idempotency
Fencing tokens
Durable constraints
```

where required.

---

# 72. Scenario: Redis Memory Keeps Increasing

Investigate:

```text
Key count
Big keys
TTL
Unbounded collections
Value size
Fragmentation
Recent deployments
Stream retention
```

Then fix the root cause.

---

# 73. Scenario: Redis Failover

Expected:

```text
Primary fails
 ↓
Sentinel/Cluster detects
 ↓
Replica promoted
 ↓
Client discovers new primary
 ↓
Application reconnects
```

Test:

```text
Read
Write
Connection recovery
Error rate
Data consistency
```

---

# 74. Scenario: Multi-Key Lua Script Fails in Cluster

Likely:

```text
Keys are in different hash slots
```

Use:

```text
Hash tags
```

when the data model permits.

Example:

```text
cart:{101}:items
cart:{101}:total
```

---

# 75. Scenario: Consumer Lag Keeps Growing

Check:

```text
Producer rate
Consumer throughput
DB latency
Downstream API latency
Consumer count
Redis capacity
```

Possible actions:

```text
Scale consumers
Batch work
Optimize processing
Apply backpressure
Scale downstream systems
```

---

# 76. Scenario: Poison Message

Architecture:

```text
Receive
 ↓
Retry 1
 ↓
Retry 2
 ↓
Retry 3
 ↓
Dead-letter
```

Store enough information to:

```text
Investigate
Fix
Replay
```

---

# 77. Scenario: Need Every Event Processed

Avoid:

```text
Pub/Sub only
```

Prefer:

```text
Redis Streams
+
Consumer groups
+
Acknowledgement
+
Idempotent consumers
```

For larger event-streaming requirements, evaluate Kafka or another dedicated platform.

---

# 78. Scenario: Notifications Can Be Lost

If acceptable:

```text
Pub/Sub
```

is simple and low latency.

Example:

```text
Order Service
 ↓
PUBLISH notification
 ↓
Notification Service
```

---

# 79. Scenario: Need Each Service to Process the Same Event

Streams can use separate consumer groups:

```text
orders
 ├── inventory-group
 ├── notification-group
 └── analytics-group
```

Each group has its own processing position.

---

# 80. Scenario: Need Horizontal Redis Scaling

If one Redis primary cannot handle:

```text
Memory
or
Write throughput
```

consider:

```text
Redis Cluster
```

with:

```text
Hash slots
+
multiple primaries
+
replicas
```

---

# 81. Scenario: Dataset Fits on One Redis Node

If:

```text
Memory fits
Workload fits
```

and you primarily need:

```text
High availability
```

consider:

```text
Primary
+
Replicas
+
Sentinel
```

instead of introducing cluster sharding unnecessarily.

---

# 82. Scenario: Redis Region Fails

This is:

```text
Disaster Recovery
```

not just:

```text
Node failover
```

Consider:

```text
Cross-region backups
DR Redis architecture
Infrastructure automation
DNS/failover
RPO
RTO
Restore testing
```

---

# 83. Rapid-Fire Round

### What is Redis?

```text
In-memory data store.
```

### Why is it fast?

```text
Memory + efficient data structures + low-latency operations.
```

### What is TTL?

```text
Key expiration time.
```

### What is eviction?

```text
Removing keys under memory pressure.
```

### What is cache-aside?

```text
Read cache → miss → DB → populate cache.
```

---

# 84. Rapid-Fire Round

### What is SET NX EX?

```text
Set only if absent + expiration.
```

### What is Lua used for?

```text
Atomic multi-step Redis logic.
```

### What is Pub/Sub?

```text
Ephemeral real-time messaging.
```

### What are Streams?

```text
Persistent event streams with consumer groups.
```

### What is XACK?

```text
Acknowledge processed stream entry.
```

---

# 85. Rapid-Fire Round

### What is Sentinel?

```text
HA, monitoring and failover.
```

### What is Cluster?

```text
Sharding + HA.
```

### How many cluster hash slots?

```text
16,384.
```

### MOVED?

```text
Slot belongs elsewhere.
```

### ASK?

```text
Temporary migration redirection.
```

---

# 86. Rapid-Fire Round

### RDB?

```text
Snapshot persistence.
```

### AOF?

```text
Append-only write log.
```

### Replica stale?

```text
Possible due to asynchronous replication.
```

### Why not KEYS *?

```text
Can block large production instances.
```

### What should replace it?

```text
SCAN.
```

---

# 87. Top 10 Redis Mistakes

Avoid these interview mistakes:

```text
1. Saying Redis replaces MySQL
2. Saying Pub/Sub guarantees delivery
3. Saying Streams guarantee exactly once
4. Using DEL without lock ownership verification
5. Ignoring lock expiration
6. Using GET + SET for concurrent counters
7. Ignoring cache stampede
8. Saying replicas are backups
9. Saying Sentinel provides sharding
10. Saying Cluster automatically solves hot keys
```

---

# 88. Strong Interview Thinking

When answering Redis questions, mention the trade-off.

Example:

> "Replica reads improve scalability, but they can be stale because replication is asynchronous."

Or:

> "Redis locking can coordinate concurrent work, but I'd still make the business operation idempotent because leases can expire."

Or:

> "Pub/Sub gives low-latency fan-out, but disconnected consumers can miss messages, so I'd use Streams for durable processing."

This makes the answer sound like production experience rather than memorization.

---

# 89. Redis + Spring Boot Architecture

A strong backend architecture might look like:

```text
                 Spring Boot
                      |
          +-----------+-----------+
          |           |           |
        Cache       Lock       Rate Limit
          |           |           |
          +-----------+-----------+
                      |
                    Redis
                      |
          +-----------+-----------+
          |                       |
       Streams                 Cluster
          |
    Consumer Groups
          |
      Microservices
          |
        MySQL
```

---

# 90. E-commerce Example

```text
Client
 ↓
Spring Boot API
 ↓
Redis Cache
 ↓
MySQL
```

For concurrency:

```text
Redis
 ↓
Inventory atomic operation
```

For rate limiting:

```text
Redis counter/token bucket
```

For events:

```text
Redis Streams
```

For HA:

```text
Redis Cluster / Sentinel
```

For durable business correctness:

```text
MySQL transaction + constraints
```

---

# 91. Final System Design Answer

If asked:

> "Design an e-commerce backend using Redis."

Say:

> "I'd use Redis as a cache for frequently accessed products and other read-heavy data, with TTL and explicit invalidation where required. I'd use atomic Redis operations or Lua for concurrency-sensitive operations such as inventory reservation and Redis-based rate limiting. For important asynchronous events I'd use Redis Streams with consumer groups and idempotent consumers. For high availability I'd choose Sentinel if the dataset fits on one primary, or Redis Cluster if sharding is needed. MySQL would remain the durable source of truth for critical business data."

---

# 92. Final Redis Revision

You should now be able to explain:

```text
Redis
 ↓
Why it is fast
 ↓
Data structures
 ↓
Caching
 ↓
TTL
 ↓
Eviction
 ↓
Distributed locks
 ↓
Rate limiting
 ↓
Pub/Sub
 ↓
Streams
 ↓
Consumer groups
 ↓
Idempotency
 ↓
Replication
 ↓
Sentinel
 ↓
Cluster
 ↓
Persistence
 ↓
Security
 ↓
Monitoring
 ↓
Production architecture
```

---

# 93. Final 30-Second Answer

If an interviewer asks:

> "Tell me about your Redis knowledge."

Use:

> "I've worked with Redis concepts around caching, TTL, eviction and Spring Boot integration using RedisTemplate and Spring Cache. I understand atomic operations, distributed locks using SET NX EX, rate limiting and Lua scripts. I've also studied Pub/Sub and Redis Streams with consumer groups, acknowledgements and idempotent consumers. On the infrastructure side, I understand replication, Sentinel, Redis Cluster, hash slots, persistence, security and monitoring. I also focus on the failure scenarios rather than treating Redis as just a cache."

---

# 94. Redis Preparation Complete

The Redis folder now contains:

```text
01 → Redis Fundamentals
02 → Data Structures & Commands
03 → Expiration, Eviction & Memory
04 → Caching Patterns & Spring Boot
05 → Distributed Locks, Rate Limiting & Atomicity
06 → Pub/Sub, Streams & Messaging
07 → Replication, Sentinel, Cluster & HA
08 → Security, Persistence, Monitoring & Production
09 → Interview Revision & Scenarios
```

Use File 09 for:

```text
Interview revision
Mock interviews
Scenario practice
System design preparation
Quick Redis refresh
```

---

# 95. Final Takeaway

> **The most important Redis interview skill is not memorizing commands. It is knowing why you would use Redis, what consistency and failure guarantees you actually get, and what you would do when Redis, the network, the application or the downstream database fails.**

Next backend topic:

```text
Docker
```

Recommended Docker sequence:

```text
Docker Fundamentals
Images & Containers
Dockerfile
Volumes & Networks
Docker Compose
Spring Boot + Docker
MySQL + Redis + Spring Boot
Production Docker
Docker Interview Questions
```
