# Redis — File 01: Fundamentals

This is the first Redis interview-preparation file.

The goal is to understand Redis from the ground up before moving into Spring Boot integration, caching patterns, distributed locking, messaging, clustering and production architecture.

---

# 1. What Is Redis?

Redis is an in-memory data store commonly used for:

```text
Caching
Session storage
Counters
Rate limiting
Distributed locking
Pub/Sub
Streams
Queues
Real-time data
```

Redis is often described as an:

> In-memory data structure server.

The important idea is that Redis stores data primarily in memory, which allows very fast reads and writes.

---

# 2. Why Is Redis Fast?

Traditional relational database access often involves:

```text
Application
 ↓
Network
 ↓
Database
 ↓
Disk / Buffer Pool
 ↓
Query execution
```

Redis is designed around in-memory operations:

```text
Application
 ↓
Network
 ↓
Redis
 ↓
Memory
```

Redis can therefore provide very low-latency operations for suitable workloads.

But:

> Redis is not automatically faster for every workload. Network latency, command complexity, serialization, contention and architecture still matter.

---

# 3. Redis Is More Than a Cache

A common beginner misconception is:

```text
Redis = cache
```

Redis can be used as a cache, but it also supports:

```text
Key-value storage
Lists
Sets
Sorted Sets
Hashes
Streams
Pub/Sub
Counters
Distributed coordination
```

So:

```text
Redis
→ Data structure server

Caching
→ One major use case
```

---

# 4. Redis vs MySQL

Think of them differently.

### MySQL

```text
Relational database
Persistent system of record
SQL
Tables
Relationships
Transactions
Constraints
```

### Redis

```text
In-memory data store
Key-based access
Rich data structures
Very low latency
Often used as cache/coordination layer
```

A common architecture is:

```text
Application
   ↓
Redis
   ↓ cache miss
MySQL
```

---

# 5. Cache-Aside Architecture

A common pattern:

```text
Application
     ↓
Check Redis
     ↓
   Hit?
  /    \
Yes     No
 |       |
Return   MySQL
         ↓
       Redis
         ↓
       Return
```

Example:

```text
GET product:101
```

If Redis has it:

```text
Return cached product
```

If not:

```text
Read MySQL
 ↓
Store in Redis
 ↓
Return
```

---

# 6. Why Use Redis in Front of a Database?

Suppose:

```text
Product catalog
```

has:

```text
10,000 requests/sec
```

but the data changes relatively rarely.

Without cache:

```text
10,000 requests/sec
 ↓
MySQL
```

With Redis:

```text
10,000 requests/sec
 ↓
Redis
 ↓
Only cache misses → MySQL
```

This can significantly reduce database load.

---

# 7. Redis Key-Value Model

The basic Redis model is:

```text
KEY → VALUE
```

Example:

```text
user:101 → "Sudhir"
```

But Redis values can be structured data.

Examples:

```text
String
Hash
List
Set
Sorted Set
Stream
```

---

# 8. Redis Keys

Good Redis keys are:

```text
Descriptive
Consistent
Namespaced
Predictable
```

Example:

```text
user:101
product:500
order:10001
session:abc123
```

---

# 9. Key Namespacing

Instead of:

```text
101
```

use:

```text
user:101
```

Instead of:

```text
500
```

use:

```text
product:500
```

This avoids collisions and makes keys easier to understand.

---

# 10. Hierarchical Key Names

A common convention:

```text
entity:id
```

Examples:

```text
user:100
product:500
order:10001
```

More detailed:

```text
product:500:details
product:500:reviews
user:100:cart
```

The exact naming convention should be consistent across the application.

---

# 11. Redis Commands

The basic Redis CLI looks like:

```text
redis-cli
```

Set:

```text
SET user:101 "Sudhir"
```

Get:

```text
GET user:101
```

Delete:

```text
DEL user:101
```

---

# 12. SET and GET

Example:

```text
SET product:101 "Laptop"
```

Then:

```text
GET product:101
```

returns:

```text
Laptop
```

These are basic string operations.

---

# 13. Checking Key Existence

```text
EXISTS user:101
```

Returns whether the key exists.

You can also inspect:

```text
TYPE user:101
```

to determine the Redis data type associated with the key.

---

# 14. Delete

```text
DEL user:101
```

removes the key.

Multiple keys can also be deleted in one command:

```text
DEL user:101 user:102
```

Be careful with large-scale deletion patterns because deleting huge structures can create latency.

---

# 15. Key Expiration

Redis supports expiration.

Example:

```text
SET session:abc "data" EX 3600
```

This means approximately:

```text
Expire after 3600 seconds
```

Another approach:

```text
SET session:abc "data"
EXPIRE session:abc 3600
```

---

# 16. TTL

Check remaining lifetime:

```text
TTL session:abc
```

Example:

```text
3500
```

means roughly:

```text
3500 seconds remaining
```

Redis also supports millisecond-oriented expiration commands.

---

# 17. Why TTL Is Important

For cache data:

```text
Product
 ↓
Cache
 ↓
TTL
 ↓
Eventually expires
```

This prevents stale data from remaining forever.

TTL is especially useful when:

```text
Perfect invalidation is difficult
Some staleness is acceptable
Data naturally becomes less useful over time
```

---

# 18. Redis Persistence

Redis is primarily memory-oriented, but it can persist data.

Two major persistence mechanisms are:

```text
RDB
AOF
```

---

# 19. RDB

RDB creates snapshots of Redis data.

Conceptually:

```text
Redis memory
 ↓
Snapshot
 ↓
RDB file
```

Advantages:

```text
Compact
Fast restart for many workloads
Good for backups
```

Trade-off:

```text
Potential data loss since the last snapshot
```

depending on configuration.

---

# 20. AOF

AOF means:

```text
Append Only File
```

Redis records write operations so they can be replayed during recovery.

Conceptually:

```text
SET
INCR
HSET
...
 ↓
AOF
```

Advantages:

```text
Potentially lower data-loss window
More detailed persistence history
```

Trade-offs:

```text
More I/O
Larger persistence data
Rewrite/maintenance overhead
```

---

# 21. RDB vs AOF

Think:

```text
RDB
→ Snapshot-based

AOF
→ Operation-log based
```

Depending on requirements, Redis can use:

```text
RDB
AOF
Both
```

Persistence configuration should match the application's durability requirements.

---

# 22. Redis as a Cache

When Redis is only a cache:

```text
MySQL
→ source of truth

Redis
→ derived/cache copy
```

If Redis data disappears:

```text
Application
 ↓
MySQL
 ↓
Repopulate Redis
```

This is fundamentally different from using Redis as the primary durable data store.

---

# 23. Cache Data vs Source of Truth

Ask:

```text
Can this data be reconstructed?
```

If yes:

```text
Redis cache
```

may be appropriate.

If Redis contains the only copy of important data:

```text
Persistence
Replication
Failover
Backups
Recovery
```

become much more important.

---

# 24. Redis Strings

Strings are the simplest Redis data type.

Example:

```text
SET user:name "Sudhir"
GET user:name
```

Strings can also represent:

```text
Numbers
JSON
Serialized values
Tokens
Flags
```

---

# 25. Counters

Redis is excellent for atomic counters.

Example:

```text
INCR page:views
```

Every command increments the value.

This is useful for:

```text
Page views
Likes
Request counters
Rate limiting
Metrics
```

---

# 26. Atomic INCR

Suppose:

```text
counter = 100
```

Run:

```text
INCR counter
```

Result:

```text
101
```

The operation is atomic at the Redis command level.

This makes Redis useful for concurrent counters.

---

# 27. Decrement

```text
DECR inventory:101
```

decreases a numeric value.

Other increment/decrement variants exist for integer and floating-point use cases.

---

# 28. Redis Hashes

Hashes store fields under one key.

Example:

```text
HSET user:101 name "Sudhir"
HSET user:101 city "Bangalore"
HSET user:101 role "Developer"
```

Conceptually:

```text
user:101
 ├── name → Sudhir
 ├── city → Bangalore
 └── role → Developer
```

---

# 29. Hash vs JSON String

You could store:

```text
user:101 → JSON
```

or:

```text
user:101 → Hash
```

Hash can be useful when you need:

```text
Individual field operations
HGET
HSET
HINCRBY
```

JSON can be convenient when the application naturally serializes/deserializes an entire object.

Choose based on access patterns.

---

# 30. Redis Lists

Lists are ordered collections.

Example:

```text
LPUSH queue task1
LPUSH queue task2
```

Read/remove:

```text
LPOP queue
```

Lists can be used for:

```text
Simple queues
Recent items
Activity feeds
Task processing
```

For advanced event processing, Redis Streams may be more appropriate.

---

# 31. Redis Sets

Sets contain unique members.

Example:

```text
SADD users:online 101
SADD users:online 102
```

Adding the same member again does not create a duplicate.

Useful for:

```text
Unique tags
Membership
Online users
Feature flags
Deduplication
```

---

# 32. Set Operations

Redis supports operations such as:

```text
SADD
SREM
SISMEMBER
SMEMBERS
SCARD
```

and set algebra such as:

```text
SINTER
SUNION
SDIFF
```

Useful for:

```text
Intersections
Membership
Recommendations
Permissions
```

---

# 33. Sorted Sets

Sorted Sets contain:

```text
member
+
score
```

Example:

```text
ZADD leaderboard 100 user:101
ZADD leaderboard 250 user:102
```

Redis keeps members ordered by score.

---

# 34. Sorted Set Use Cases

Excellent for:

```text
Leaderboards
Ranking
Priority queues
Time-based indexes
Top-N results
```

Example:

```text
Leaderboard
250 → user:102
100 → user:101
```

---

# 35. Redis Data Structure Selection

Think:

```text
Single value?
→ String

Object fields?
→ Hash

Ordered sequence?
→ List

Unique membership?
→ Set

Ranking by score?
→ Sorted Set

Event log?
→ Stream
```

---

# 36. Redis Atomicity

Redis processes individual commands sequentially in its main execution model.

This means a single command such as:

```text
INCR counter
```

can be atomic with respect to other Redis commands.

But:

> A multi-step business operation is not automatically atomic just because Redis is fast.

Example:

```text
GET stock
 ↓
calculate
 ↓
SET stock
```

can have race conditions.

---

# 37. Multi-Step Race Condition

Suppose:

```text
stock = 1
```

Two clients:

```text
A reads 1
B reads 1

A writes 0
B writes 0
```

One logical update is lost.

For complex atomic operations consider:

```text
Lua scripts
Transactions
WATCH
Atomic commands
```

depending on the requirement.

---

# 38. Redis Transactions

Redis provides:

```text
MULTI
EXEC
DISCARD
WATCH
```

A transaction queues commands and executes them as a group.

Example concept:

```text
MULTI
INCR counter
INCR counter
EXEC
```

---

# 39. Redis Transactions vs Database Transactions

Don't assume Redis transactions behave exactly like relational database transactions.

Redis transactions:

```text
Queue commands
Execute them sequentially
```

They do not provide the same rollback model as a relational database transaction.

If one queued command fails during execution, previously executed commands are not automatically rolled back.

---

# 40. WATCH

`WATCH` provides optimistic concurrency control.

Conceptually:

```text
WATCH stock:101
 ↓
GET stock:101
 ↓
Calculate
 ↓
MULTI
SET stock:101 ...
EXEC
```

If another client modifies the watched key before `EXEC`:

```text
EXEC
 ↓
Aborted
```

The application can retry.

---

# 41. Lua Scripts

Lua scripts can execute multiple Redis operations atomically from Redis's command execution perspective.

Conceptually:

```text
Application
 ↓
Lua script
 ↓
Multiple Redis operations
```

Useful when:

```text
Several related operations must happen atomically
```

---

# 42. Example Atomic Stock Check

Conceptually:

```text
if stock > 0:
    decrement stock
    return success
else:
    return failure
```

Doing this in one server-side script avoids the race between:

```text
GET
+
SET
```

---

# 43. Redis Key Expiration

Expiration can be used for:

```text
Sessions
OTP state
Cache entries
Temporary locks
Rate-limit windows
Temporary tokens
```

Always choose TTL based on business semantics.

---

# 44. Expiration Is Not Exact Timing

A key with:

```text
TTL = 60 seconds
```

should not be interpreted as a guarantee that the key disappears at exactly:

```text
60.000 seconds
```

Redis manages expiration internally.

The important guarantee is that expired keys are not intended to remain indefinitely accessible as valid data.

---

# 45. Keyspace Notifications

Redis can optionally emit notifications for certain key events.

Potential uses:

```text
Cache-related events
Expiration notifications
Monitoring
Application integrations
```

But they should not be treated as a universal reliable event-delivery mechanism.

For durable event processing, Redis Streams or a dedicated messaging system may be more appropriate.

---

# 46. Redis Pub/Sub

Pub/Sub allows publishers to send messages to channels.

Conceptually:

```text
Publisher
   ↓
Channel
   ↓
Subscribers
```

Example:

```text
PUBLISH notifications "hello"
```

Subscriber:

```text
SUBSCRIBE notifications
```

---

# 47. Pub/Sub Limitation

Pub/Sub messages are generally ephemeral.

If a subscriber is disconnected:

```text
Message can be missed
```

For durable message processing, consider:

```text
Redis Streams
Kafka
RabbitMQ
```

depending on requirements.

---

# 48. Redis Streams

Streams are append-oriented data structures for event/message processing.

Conceptually:

```text
Stream
--------------------------------
event1
event2
event3
event4
```

Useful for:

```text
Event processing
Message queues
Consumer groups
Audit/event streams
```

---

# 49. Pub/Sub vs Streams

```text
Pub/Sub
→ Real-time fan-out
→ Ephemeral
→ Subscriber can miss messages

Streams
→ Persistent stream entries
→ Consumer groups
→ Acknowledgement/processing model
```

---

# 50. Redis and Sessions

Redis is commonly used for distributed sessions.

Architecture:

```text
Client
 ↓
Load Balancer
 ↓
Instance A / B / C
 ↓
Redis
```

Any application instance can retrieve:

```text
session:user:101
```

This avoids storing session state only in one application server.

---

# 51. Session Key

Example:

```text
session:abc123
```

Value:

```text
userId
roles
preferences
expiration
```

Set an appropriate TTL.

---

# 52. Redis for Rate Limiting

Suppose:

```text
Maximum 100 requests/minute
```

Redis can maintain:

```text
rate:user:101
```

with an expiration window.

Atomic operations such as:

```text
INCR
EXPIRE
```

or Lua scripts can help implement rate limiting.

---

# 53. Rate Limiting Race

Naively doing:

```text
INCR
EXPIRE
```

as separate operations can have edge cases around expiration setup and concurrent requests.

A Lua script or carefully designed atomic pattern can make the operation more reliable.

---

# 54. Redis Distributed Lock

A distributed lock can coordinate work across multiple application instances.

Conceptually:

```text
Instance A
 ↓
Redis lock

Instance B
 ↓
Cannot acquire lock
```

A common primitive is:

```text
SET lock:key token NX EX 30
```

Meaning:

```text
NX → only if key doesn't exist
EX → expiration
```

The token should uniquely identify the lock owner.

---

# 55. Why Lock TTL Matters

Suppose:

```text
Instance A acquires lock
```

Then crashes.

Without expiration:

```text
Lock remains forever
```

With TTL:

```text
Lock eventually expires
```

But lock correctness requires more than simply adding a TTL.

---

# 56. Distributed Lock Safety

A robust design should consider:

```text
Unique owner token
Lock expiration
Safe release
Clock/network failures
Long-running work
Retry
Fencing where necessary
```

Never blindly do:

```text
DEL lock:key
```

without verifying ownership, because another process may have acquired the lock after expiration.

---

# 57. Redis Architecture

A simple deployment:

```text
Application
    |
    v
  Redis
```

Production may use:

```text
Application
    |
Load Balancer
    |
Redis topology
 ├── Primary
 ├── Replicas
 └── Sentinel/Cluster
```

---

# 58. Redis Replication

Redis can replicate data from:

```text
Primary
 ↓
Replica
```

Replicas can help with:

```text
Read scaling
High availability architecture
Failover support
```

But replication semantics and read consistency must be understood.

---

# 59. Redis Sentinel

Sentinel can help with:

```text
Monitoring
Failure detection
Automatic failover
Primary discovery
```

It is primarily a high-availability mechanism.

---

# 60. Redis Cluster

Redis Cluster distributes keyspace across multiple nodes.

Conceptually:

```text
Keyspace
 ↓
Hash slots
 ↓
Multiple Redis nodes
```

Useful for:

```text
Horizontal scaling
Large datasets
Higher aggregate throughput
```

---

# 61. Cluster vs Sentinel

```text
Sentinel
→ High availability/failover

Cluster
→ Data sharding + horizontal scaling
```

They solve different primary problems.

---

# 62. Redis Hash Slots

Redis Cluster divides the keyspace into:

```text
16384 hash slots
```

Keys are mapped to slots.

The slot determines which cluster node owns the key.

---

# 63. Hash Tags

Redis Cluster supports hash tags.

Example:

```text
user:{101}:profile
user:{101}:cart
```

The portion inside:

```text
{101}
```

can ensure related keys map to the same hash slot.

This is useful when an operation needs multiple related keys to be co-located.

---

# 64. Cluster Multi-Key Operations

Some multi-key commands require keys to belong to the same hash slot.

Therefore:

```text
user:101
user:102
```

may be on different nodes.

Hash tags can help when the data model permits:

```text
user:{101}:profile
user:{101}:cart
```

---

# 65. Redis Memory

Redis is memory-first.

Therefore monitor:

```text
Used memory
Peak memory
Fragmentation
Evictions
Key count
Large keys
```

Memory planning is critical.

---

# 66. Big Keys

A big key is an unusually large Redis key/value structure.

Examples:

```text
Huge List
Huge Hash
Huge Set
Large serialized JSON
```

Problems:

```text
Memory spikes
Slow operations
Network overhead
Deletion latency
Replication impact
```

---

# 67. Hot Keys

A hot key receives a very high number of requests.

Example:

```text
product:1
```

receives:

```text
500,000 requests/sec
```

Potential problems:

```text
Single-node CPU pressure
Network bottleneck
Lock/contention patterns
Cluster imbalance
```

Possible strategies:

```text
Local caching
Key replication patterns
Request coalescing
Sharding
Traffic distribution
```

---

# 68. Cache Stampede

Suppose:

```text
Popular key expires
```

Then:

```text
10,000 requests
 ↓
Redis miss
 ↓
10,000 DB queries
```

Solutions:

```text
Locking
Request coalescing
Jittered TTL
Refresh ahead
Stale-while-revalidate
```

---

# 69. Cache Penetration

Requests repeatedly ask for nonexistent data:

```text
product:999999
```

Every request:

```text
Redis miss
 ↓
DB miss
```

Potential solution:

```text
Negative caching
```

with careful TTL selection.

---

# 70. Cache Avalanche

Many cache entries expire simultaneously:

```text
Huge cache miss
 ↓
Database overload
```

Mitigate with:

```text
TTL jitter
Staggered expiration
Warm-up
Load shedding
Fallback strategies
```

---

# 71. Redis Security

Don't expose Redis directly to the public internet.

Use:

```text
Private network
Authentication/ACLs
TLS where appropriate
Firewall rules
Least privilege
```

Also avoid putting secrets or sensitive data into Redis without appropriate protection.

---

# 72. Redis Serialization

Spring applications often serialize Java objects before storing them.

Possible formats:

```text
JSON
String
Binary serialization
```

JSON is often easier to inspect and interoperable.

Binary formats can be more compact/faster in some scenarios but introduce compatibility considerations.

---

# 73. Serialization Versioning

Suppose:

```text
Application v1
stores User object
```

Then deploy:

```text
Application v2
```

If the serialization format changed:

```text
Old Redis data
 ↓
New application
 ↓
Deserialization failure
```

Plan serialization compatibility during deployments.

---

# 74. Redis and Cache Versioning

A useful technique:

```text
user:v1:101
```

Later:

```text
user:v2:101
```

This can help during schema/serialization migrations.

Another approach is controlled cache eviction.

---

# 75. Redis Monitoring

Monitor:

```text
Memory
CPU
Commands/sec
Latency
Hit rate
Evictions
Connected clients
Blocked clients
Replication lag
Persistence status
Key distribution
Hot keys
Big keys
```

---

# 76. Redis Latency

A Redis operation can still be slow because of:

```text
Large value
Large collection
Network latency
CPU pressure
Blocking command
Hot key
Persistence workload
Server overload
```

"Redis is in-memory" does not mean every command is automatically cheap.

---

# 77. Avoid Dangerous Large Operations

Be careful with commands that can scan huge datasets or return huge results.

Examples:

```text
KEYS *
SMEMBERS huge-set
LRANGE huge-list 0 -1
```

For production key discovery, prefer:

```text
SCAN
```

where appropriate.

---

# 78. KEYS vs SCAN

```text
KEYS *
```

can block the server for large keyspaces.

Prefer:

```text
SCAN
```

for incremental iteration.

Similarly:

```text
HSCAN
SSCAN
ZSCAN
```

exist for appropriate structures.

---

# 79. Redis Command Complexity

Understand complexity.

Examples:

```text
GET
→ approximately O(1)

SET
→ approximately O(1)

HGET
→ approximately O(1)

SISMEMBER
→ approximately O(1)
```

But some operations depend on collection size.

Always check the command's complexity before using it on large structures.

---

# 80. Why Complexity Matters

Example:

```text
LRANGE huge-list 0 -1
```

returns the entire list.

Even if Redis is fast:

```text
Huge response
+
serialization
+
network
+
client processing
```

can be expensive.

---

# 81. Redis Best Practices

```text
Use predictable key naming
Set TTLs for temporary/cache data
Avoid huge values
Avoid huge collections when unnecessary
Avoid KEYS in production
Monitor memory
Monitor latency
Use appropriate data structures
Design invalidation explicitly
Secure Redis
Test failover
```

---

# 82. Redis Interview Question

### What is Redis?

Answer:

> "Redis is an in-memory data structure server commonly used for caching, counters, sessions, rate limiting, distributed coordination and messaging. It supports data structures such as strings, hashes, lists, sets, sorted sets and streams."

---

# 83. Redis Interview Question

### Why is Redis fast?

Answer:

> "Redis is designed around in-memory data access and efficient data structures with low-overhead commands. Performance still depends on network latency, command complexity, value size, serialization and server load."

---

# 84. Redis Interview Question

### Is Redis a database or cache?

Answer:

> "It can be used as both. In many backend systems Redis acts as a cache in front of a relational database, but Redis also supports persistence and use cases such as sessions, counters, streams and coordination."

---

# 85. Redis Interview Question

### Redis vs MySQL?

Answer:

> "MySQL is a relational system of record with SQL, constraints and durable relational storage. Redis is an in-memory data structure store optimized for low-latency key/data-structure operations. They often complement each other rather than replace each other."

---

# 86. Redis Interview Question

### What is TTL?

Answer:

> "TTL is the remaining lifetime of a Redis key before it expires. It is commonly used for cache entries, sessions, temporary tokens and rate-limit state."

---

# 87. Redis Interview Question

### RDB vs AOF?

Answer:

> "RDB stores snapshots of Redis data, while AOF records write operations. RDB is compact and useful for snapshots, while AOF can provide a smaller potential data-loss window depending on its configuration. The choice depends on durability and performance requirements."

---

# 88. Redis Interview Question

### What is cache-aside?

Answer:

> "The application first checks Redis. On a cache hit it returns the cached data; on a miss it reads from the database, stores the result in Redis with an appropriate TTL and returns it."

---

# 89. Redis Interview Question

### What is cache stampede?

Answer:

> "It occurs when many requests miss the same cache entry at the same time, often after expiration, causing a large number of requests to hit the database simultaneously."

---

# 90. Redis Interview Question

### How do you prevent cache stampede?

Answer:

> "Depending on the workload, I can use request coalescing, a short-lived lock, jittered TTLs, refresh-ahead or stale-while-revalidate strategies. The goal is to avoid all requests rebuilding the same cache entry simultaneously."

---

# 91. Redis Interview Question

### What is a hot key?

Answer:

> "A hot key is a Redis key receiving disproportionately high traffic. It can overload the node responsible for that key even when the overall cluster has spare capacity."

---

# 92. Redis Interview Question

### What is a big key?

Answer:

> "A big key is an unusually large value or collection stored under one Redis key. It can cause memory, network, latency and deletion problems, so large structures should be monitored and bounded."

---

# 93. Redis Interview Question

### KEYS vs SCAN?

Answer:

> "`KEYS` can scan the entire keyspace in a blocking operation and is risky on large production datasets. `SCAN` iterates incrementally and is generally safer for production keyspace inspection."

---

# 94. Redis Interview Question

### Pub/Sub vs Streams?

Answer:

> "Pub/Sub is useful for ephemeral real-time fan-out where missed messages are acceptable. Streams provide persisted entries and consumer-group features, making them more suitable for durable event or message processing."

---

# 95. Redis Interview Question

### Sentinel vs Cluster?

Answer:

> "Sentinel primarily provides monitoring and automatic failover for high availability. Redis Cluster provides sharding across nodes as well as high-availability capabilities, making it suitable for horizontal data and throughput scaling."

---

# 96. Redis Interview Question

### How would you use Redis for rate limiting?

Answer:

> "I would maintain a counter for the relevant user, IP or API key and use expiration to define the window. For operations requiring multiple steps to be atomic, I'd use a Lua script or another atomic Redis pattern."

---

# 97. Redis Interview Question

### How would you implement a distributed lock?

Answer:

> "I'd use an atomic lock acquisition such as `SET key token NX EX ttl`, where the token identifies the owner. On release I'd verify the token before deleting the lock, usually through an atomic script, so one process cannot accidentally release another process's lock."

---

# 98. Production Scenario

### Product API is slow.

Architecture:

```text
GET /products/101
```

Current:

```text
API
 ↓
MySQL
```

Improve:

```text
API
 ↓
Redis
 ↓ miss
MySQL
 ↓
Redis
```

Add:

```text
TTL
+
invalidation strategy
+
monitoring
```

---

# 99. Production Scenario

### Redis goes down.

If Redis is only a cache:

```text
Redis failure
 ↓
Fallback to MySQL
```

But be careful:

```text
All traffic moves to MySQL
 ↓
Database overload
```

Therefore design:

```text
Rate limiting
Circuit breaker
Load shedding
Fallback limits
```

where appropriate.

---

# 100. Production Scenario

### Cache hit rate suddenly drops.

Investigate:

```text
TTL changes
Key format changes
Cache invalidation
Deployment
Serialization changes
Memory eviction
Redis restart
Traffic pattern
```

Don't immediately increase Redis capacity.

---

# 101. Production Scenario

### Redis memory is full.

Investigate:

```text
Memory usage
Key count
Big keys
TTL coverage
Evictions
Duplicate data
Unbounded collections
```

Then:

```text
Delete unnecessary data
Set appropriate TTL
Bound collections
Optimize values
Increase capacity if justified
```

---

# 102. Production Scenario

### Redis CPU is high.

Investigate:

```text
Commands/sec
Expensive commands
Large collections
Hot keys
Lua scripts
Client behavior
Network traffic
```

Use monitoring and slow-command analysis rather than guessing.

---

# 103. Production Scenario

### One Redis Cluster node is overloaded.

Possible:

```text
Hot key
Uneven key distribution
Large key
Traffic imbalance
```

Cluster scaling alone may not solve a hot-key problem.

---

# 104. Production Scenario

### Two application instances process the same scheduled job.

Possible approach:

```text
Distributed lock
```

But also ask:

```text
What if lock expires?
What if instance crashes?
Can the job be retried?
Is the job idempotent?
```

Distributed locks are not a substitute for idempotent job design.

---

# 105. Redis + Spring Boot Mental Model

Later, we'll build:

```text
Controller
   ↓
Service
   ↓
Cache / RedisTemplate
   ↓
Redis
```

For cache-aside:

```text
Service
 ↓
Redis GET
 ↓ miss
Repository
 ↓
MySQL
 ↓
Redis SETEX
```

Spring Cache can abstract much of this.

---

# 106. Final Interview Answer

If asked:

> "How would you introduce Redis into an existing Spring Boot application?"

Say:

> "I'd first identify read-heavy or latency-sensitive data that is suitable for caching. I'd use a cache-aside approach where the application checks Redis and falls back to the database on a miss. I'd define predictable keys, appropriate TTLs and an explicit invalidation strategy. Then I'd monitor hit rate, Redis latency, memory, evictions and database load. I'd also design the fallback carefully so a Redis outage doesn't overwhelm the database."

---

# 107. Revision Checklist

```text
□ What is Redis?
□ Why Redis is fast
□ Redis vs MySQL
□ Redis as cache
□ Redis as data store
□ Key-value model
□ Key naming
□ SET
□ GET
□ DEL
□ EXISTS
□ TYPE
□ TTL
□ EXPIRE
□ Strings
□ Counters
□ Hashes
□ Lists
□ Sets
□ Sorted Sets
□ Streams
□ Atomic operations
□ MULTI
□ EXEC
□ WATCH
□ Lua scripts
□ RDB
□ AOF
□ Cache-aside
□ Sessions
□ Rate limiting
□ Pub/Sub
□ Distributed locks
□ Replication
□ Sentinel
□ Cluster
□ Hash slots
□ Hash tags
□ Big keys
□ Hot keys
□ Cache stampede
□ Cache penetration
□ Cache avalanche
□ KEYS vs SCAN
□ Memory monitoring
□ Security
□ Serialization
□ Production failure scenarios
```

---

# 108. What Comes Next

```text
File 02 → Redis Data Structures & Commands
```

The next file will go much deeper into:

```text
Strings
Hashes
Lists
Sets
Sorted Sets
Streams
Bitmaps
HyperLogLog
Geospatial
TTL commands
Atomic operations
Command complexity
Data modeling
Real-world examples
E-commerce use cases
Interview coding questions
```

The key takeaway:

> **Redis is not simply "a fast cache." It is an in-memory data structure system. The important backend skill is choosing the right Redis data structure, designing safe key patterns, understanding atomicity and expiration, and knowing when Redis should complement rather than replace the primary database.**
