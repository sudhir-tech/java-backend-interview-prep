# Redis — File 02: Data Structures & Commands

This file goes deeper into Redis data structures, commands, complexity and practical backend use cases.

---

# 1. Redis Data Structures

The core structures to know for backend interviews are:

```text
String
Hash
List
Set
Sorted Set
Stream
```

Also know:

```text
Bitmap
HyperLogLog
Geospatial indexes
```

The most important interview skill is not memorizing commands.

It is:

> Choosing the correct data structure for the access pattern.

---

# 2. Strings

Strings are the simplest Redis data type.

Example:

```text
SET user:101 "Sudhir"
GET user:101
```

Possible uses:

```text
Simple values
JSON payloads
Tokens
Counters
Flags
Cached responses
```

---

# 3. SET

Basic command:

```text
SET key value
```

Example:

```text
SET product:101 "Laptop"
```

Retrieve:

```text
GET product:101
```

---

# 4. SET With TTL

You can set a value and expiration together:

```text
SET session:abc123 "user-data" EX 3600
```

This is preferable to relying on a separate:

```text
SET
EXPIRE
```

sequence when atomic setup of value + expiration is desired.

---

# 5. NX

`NX` means:

```text
Set only if the key does not already exist.
```

Example:

```text
SET lock:order:101 token NX EX 30
```

Useful for:

```text
Lock acquisition
One-time initialization
Idempotency coordination
```

---

# 6. XX

`XX` means:

```text
Set only if the key already exists.
```

Useful when you want to avoid accidentally creating a missing key.

---

# 7. GET

```text
GET product:101
```

Returns the string value.

If the key doesn't exist:

```text
(nil)
```

---

# 8. MGET

Retrieve multiple string keys:

```text
MGET user:101 user:102 user:103
```

This can reduce application/network round trips compared with issuing separate requests.

---

# 9. MSET

Set multiple string keys:

```text
MSET user:101 "A" user:102 "B"
```

Useful for batching simple string writes.

---

# 10. GETDEL

Modern Redis versions provide:

```text
GETDEL key
```

It retrieves the current string value and deletes the key as one Redis operation.

Useful for one-time values such as:

```text
Temporary token
One-time state
```

---

# 11. GETEX

`GETEX` retrieves a string and can also modify its expiration.

Example:

```text
GETEX session:abc EX 3600
```

Useful for sliding-session patterns where reading can extend the lifetime.

---

# 12. Numeric Strings

Redis strings can hold numeric values.

Example:

```text
SET views 100
INCR views
```

Result:

```text
101
```

---

# 13. INCR

```text
INCR counter
```

Equivalent conceptually to:

```text
counter = counter + 1
```

Useful for:

```text
Views
Likes
Counters
Rate limiting
Sequence-like values
```

---

# 14. INCRBY

```text
INCRBY stock:101 5
```

Increases by 5.

---

# 15. DECR

```text
DECR stock:101
```

Decreases by 1.

---

# 16. DECRBY

```text
DECRBY stock:101 5
```

Decreases by 5.

---

# 17. INCRBYFLOAT

For floating-point counters:

```text
INCRBYFLOAT balance 10.5
```

Use carefully for financial calculations.

For monetary correctness, don't rely on binary floating-point semantics. Prefer integer minor units or a suitable exact representation at the application/domain level.

---

# 18. String Length

```text
STRLEN key
```

returns the length of the string value.

---

# 19. APPEND

```text
APPEND message " world"
```

Appends to an existing string.

Useful for specialized use cases, but avoid building huge strings this way.

---

# 20. Hashes

Hashes store field-value pairs under one Redis key.

Example:

```text
HSET user:101 name "Sudhir"
HSET user:101 role "Developer"
HSET user:101 city "Bangalore"
```

Conceptually:

```text
user:101
 ├── name → Sudhir
 ├── role → Developer
 └── city → Bangalore
```

---

# 21. HGET

```text
HGET user:101 name
```

Returns:

```text
Sudhir
```

---

# 22. HGETALL

```text
HGETALL user:101
```

Returns all fields and values.

Be careful with very large hashes because returning everything creates:

```text
Server work
Network traffic
Client memory
```

---

# 23. HMGET

Retrieve selected fields:

```text
HMGET user:101 name role
```

Prefer this when you don't need the complete hash.

---

# 24. HSET Multiple Fields

You can set multiple fields:

```text
HSET user:101 name "Sudhir" role "Developer" city "Bangalore"
```

---

# 25. HDEL

Delete fields:

```text
HDEL user:101 city
```

This removes the `city` field without deleting the entire hash.

---

# 26. HEXISTS

Check whether a field exists:

```text
HEXISTS user:101 email
```

---

# 27. HINCRBY

Increment a numeric hash field:

```text
HINCRBY user:101 loginCount 1
```

Useful for:

```text
Counters associated with an object
```

---

# 28. Hash Use Case

A user profile:

```text
user:101
 ├── name
 ├── email
 ├── role
 └── loginCount
```

A hash can be more convenient than maintaining separate keys for every field.

---

# 29. Hash vs JSON

Hash:

```text
Field-level access
HGET/HSET
```

JSON string:

```text
Whole-object serialization
```

Choose based on:

```text
Read pattern
Update pattern
Serialization requirements
Interoperability
```

---

# 30. Lists

Lists are ordered collections.

Commands:

```text
LPUSH
RPUSH
LPOP
RPOP
LRANGE
LLEN
```

---

# 31. LPUSH

```text
LPUSH tasks task1
LPUSH tasks task2
```

Because insertion happens on the left:

```text
task2
task1
```

---

# 32. RPUSH

```text
RPUSH tasks task1
RPUSH tasks task2
```

Produces:

```text
task1
task2
```

---

# 33. LPOP

```text
LPOP tasks
```

Removes the first item from the left.

---

# 34. RPOP

```text
RPOP tasks
```

Removes the last item.

---

# 35. LRANGE

```text
LRANGE tasks 0 -1
```

returns the list contents.

For a huge list, avoid repeatedly requesting the entire list.

---

# 36. LLEN

```text
LLEN tasks
```

returns the list length.

---

# 37. List as Queue

Producer:

```text
RPUSH jobs job1
RPUSH jobs job2
```

Consumer:

```text
LPOP jobs
```

Conceptually:

```text
Producer
   ↓
Redis List
   ↓
Consumer
```

For more advanced delivery semantics, Redis Streams are generally a better fit.

---

# 38. Blocking List Commands

Redis provides blocking operations such as:

```text
BLPOP
BRPOP
```

A consumer can wait for data rather than polling continuously.

---

# 39. Blocking Command Consideration

Blocking commands wait for data for the client connection.

This can be useful for worker patterns, but clients and connection pools must be configured appropriately.

---

# 40. Sets

Sets store unique members.

Example:

```text
SADD online-users 101
SADD online-users 102
SADD online-users 101
```

The last operation doesn't create a duplicate.

---

# 41. SADD

```text
SADD tags:product:101 java redis backend
```

Adds members.

---

# 42. SREM

```text
SREM online-users 101
```

Removes a member.

---

# 43. SISMEMBER

```text
SISMEMBER online-users 101
```

Checks membership.

Excellent for:

```text
Permission checks
Feature membership
Deduplication
```

---

# 44. SMEMBERS

```text
SMEMBERS online-users
```

Returns all members.

Again, be careful with very large sets.

---

# 45. SCARD

```text
SCARD online-users
```

returns the number of members.

---

# 46. Set Intersection

```text
SINTER users:premium users:active
```

Returns members present in both sets.

Useful for:

```text
Audience segmentation
Recommendations
Permissions
```

---

# 47. Set Union

```text
SUNION team:java team:redis
```

Returns members present in either set.

---

# 48. Set Difference

```text
SDIFF users:all users:blocked
```

Returns members in the first set but not the second.

---

# 49. Sorted Sets

Sorted Sets are:

```text
Unique member
+
Numeric score
```

Example:

```text
ZADD leaderboard 100 user:101
ZADD leaderboard 250 user:102
```

---

# 50. ZADD

```text
ZADD leaderboard 100 user:101
```

If the member already exists, its score is updated.

---

# 51. ZSCORE

```text
ZSCORE leaderboard user:101
```

Returns the score.

---

# 52. ZRANK

```text
ZRANK leaderboard user:101
```

Returns the ascending rank.

For descending rank:

```text
ZREVRANK leaderboard user:101
```

---

# 53. ZRANGE

```text
ZRANGE leaderboard 0 9
```

returns the first ten members in ascending score order.

Modern Redis also supports options such as:

```text
WITHSCORES
```

---

# 54. ZREVRANGE

```text
ZREVRANGE leaderboard 0 9 WITHSCORES
```

Useful for:

```text
Top 10 leaderboard
```

---

# 55. ZINCRBY

```text
ZINCRBY leaderboard 10 user:101
```

Adds 10 to the user's score.

Excellent for:

```text
Game scores
Points
Ranking systems
```

---

# 56. Score-Based Queries

Sorted Sets can query ranges by score.

Conceptually:

```text
ZRANGEBYSCORE leaderboard 100 500
```

Useful for:

```text
Users with score in range
Time windows
Priority ranges
```

Exact syntax varies across Redis versions as newer range-query commands have been introduced.

---

# 57. Sorted Set as Time Index

Store:

```text
score = Unix timestamp
member = event ID
```

Then query:

```text
Events between time A and time B
```

This is a useful Redis modeling technique.

---

# 58. Streams

Redis Streams are append-oriented structures.

Example:

```text
XADD orders * orderId 1001 status CREATED
```

Redis creates a stream entry with an ID.

---

# 59. Stream Entry

Conceptually:

```text
1730000000000-0
    |
    +-- orderId = 1001
    +-- status = CREATED
```

The exact generated ID depends on Redis.

---

# 60. XREAD

Read stream entries:

```text
XREAD STREAMS orders 0
```

Streams allow applications to process ordered event data.

---

# 61. Consumer Groups

Consumer groups allow multiple consumers to share stream processing.

Conceptually:

```text
orders stream
      ↓
consumer group
 ├── consumer A
 ├── consumer B
 └── consumer C
```

Messages can be distributed among consumers.

---

# 62. XGROUP

Consumer groups are created with:

```text
XGROUP CREATE
```

The exact command includes:

```text
stream
group name
starting ID
```

---

# 63. XREADGROUP

Consumers read as part of a group:

```text
XREADGROUP GROUP group1 consumer1 ...
```

This supports distributed event processing.

---

# 64. Acknowledgement

After processing a stream message:

```text
XACK
```

acknowledges it.

This allows Redis to track messages that have been delivered to a consumer but not yet acknowledged.

---

# 65. Pending Entries

If a consumer receives:

```text
event123
```

but crashes before acknowledgement:

```text
event123
```

can remain pending.

This enables recovery workflows.

---

# 66. Streams vs Lists

Lists:

```text
Simple queue
Basic producer/consumer
```

Streams:

```text
Durable stream entries
Consumer groups
Acknowledgements
Pending messages
Event-processing semantics
```

For serious event processing, Streams are usually more capable.

---

# 67. Bitmaps

Redis strings can be used as bitmaps.

Useful for:

```text
Daily active users
Feature flags
Boolean state
Compact membership tracking
```

Example:

```text
SETBIT users:active:2026-08-21 101 1
```

The exact key design should depend on the ID range and workload.

---

# 68. GETBIT

```text
GETBIT users:active:2026-08-21 101
```

Returns:

```text
0
or
1
```

---

# 69. BITCOUNT

```text
BITCOUNT users:active:2026-08-21
```

Can count set bits.

Useful for:

```text
Approximate/compact boolean activity tracking
```

---

# 70. HyperLogLog

HyperLogLog is useful for approximate cardinality estimation.

Example:

```text
PFADD visitors user1 user2 user3
PFCOUNT visitors
```

It is useful when:

```text
Exact uniqueness is not required
Very large cardinality needs to be estimated
Memory efficiency matters
```

---

# 71. HyperLogLog Trade-Off

You get:

```text
Very small memory footprint
Approximate count
```

You do not get:

```text
Exact unique set membership
```

Use it for analytics where approximation is acceptable.

---

# 72. Geospatial

Redis supports geospatial operations.

Example:

```text
GEOADD stores 77.5946 12.9716 store:101
```

Then you can query locations around a coordinate.

Useful for:

```text
Nearby stores
Drivers
Restaurants
Delivery locations
```

---

# 73. GEOPOS

```text
GEOPOS stores store:101
```

returns the stored coordinates.

---

# 74. GEOSEARCH

Modern Redis provides:

```text
GEOSEARCH
```

for geographic searches.

Example concept:

```text
Find stores within 5 km
```

This is useful for location-aware applications.

---

# 75. Data Structure Decision Table

| Requirement | Redis Structure |
|---|---|
| Simple cached value | String |
| Object fields | Hash |
| Queue | List |
| Unique membership | Set |
| Ranking | Sorted Set |
| Event processing | Stream |
| Compact boolean state | Bitmap |
| Approximate unique count | HyperLogLog |
| Nearby locations | Geospatial |

---

# 76. Command Complexity

You should understand that Redis performance depends on command complexity.

Typical simple commands:

```text
GET → O(1)
SET → O(1)
HGET → O(1)
SISMEMBER → O(1)
```

But some commands depend on collection size.

Always check complexity before applying an operation to a large data structure.

---

# 77. O(N) Warning

Commands that process large collections can become expensive.

Examples:

```text
SMEMBERS
HGETALL
LRANGE large-range
```

The issue is not that these commands are always bad.

The issue is:

```text
Large collection
+
request for everything
=
large work
```

---

# 78. KEYS

Avoid:

```text
KEYS *
```

on a large production Redis instance.

It can scan the keyspace in one blocking operation.

---

# 79. SCAN

Prefer:

```text
SCAN
```

for incremental keyspace iteration.

Similarly:

```text
HSCAN
SSCAN
ZSCAN
```

can iterate through large structures incrementally.

---

# 80. DEL vs UNLINK

For large values, deleting synchronously can create latency.

Redis also provides:

```text
UNLINK
```

which can make deletion more asynchronous/background-oriented.

Use it when appropriate for large keys.

---

# 81. EXISTS

```text
EXISTS user:101
```

checks whether a key exists.

But avoid unnecessary:

```text
EXISTS
+
GET
```

when a single command can provide the required information.

---

# 82. TYPE

```text
TYPE user:101
```

returns the Redis type.

Useful for troubleshooting incorrect key usage.

---

# 83. OBJECT

Redis provides:

```text
OBJECT
```

subcommands for inspecting certain internal properties.

This can be useful for advanced diagnostics.

Use carefully in production.

---

# 84. TTL Commands

Common commands:

```text
EXPIRE
PEXPIRE
TTL
PTTL
PERSIST
EXPIREAT
```

---

# 85. PERSIST

```text
PERSIST session:abc
```

removes the expiration from a key.

The key becomes persistent unless removed by another operation.

---

# 86. EXPIREAT

Expiration can be based on an absolute Unix timestamp:

```text
EXPIREAT key timestamp
```

This can be useful when expiration is tied to a known deadline.

---

# 87. PTTL

```text
PTTL key
```

returns remaining lifetime in milliseconds.

---

# 88. Expiration Return Values

When checking TTL, special values can indicate:

```text
Key doesn't exist
Key exists without expiration
```

Know these cases when debugging cache behavior.

---

# 89. Atomicity Rule

Single Redis commands are designed to execute atomically with respect to other Redis commands.

But this:

```text
GET
+
application calculation
+
SET
```

is not one atomic operation.

For atomic business logic use:

```text
Atomic command
Lua script
WATCH/MULTI/EXEC
```

as appropriate.

---

# 90. Example: Safe Counter

Good:

```text
INCR login-count
```

Instead of:

```text
GET login-count
application +1
SET login-count
```

The latter can lose updates under concurrency.

---

# 91. Example: Safe Stock Decrement

Bad:

```text
GET stock
if stock > 0:
    SET stock stock - 1
```

Potential race:

```text
A reads 1
B reads 1
A writes 0
B writes 0
```

Better:

```text
Atomic conditional update
```

using an appropriate Lua script or another atomic Redis pattern.

---

# 92. Redis Data Modeling

Don't think only:

```text
Java object → JSON → Redis
```

Think:

```text
What queries do I need?
What updates do I need?
What should expire?
What should be unique?
What needs atomicity?
How large can it become?
```

Then choose the structure.

---

# 93. Example: User Permissions

Requirement:

```text
Check whether user 101 has permission "ORDER_READ"
```

Good structure:

```text
user:101:permissions
```

as a Set:

```text
SADD user:101:permissions ORDER_READ
```

Check:

```text
SISMEMBER user:101:permissions ORDER_READ
```

This is more natural than storing a large serialized permission object if the dominant operation is membership checking.

---

# 94. Example: Product Leaderboard

Requirement:

```text
Top 100 products by sales
```

Use:

```text
Sorted Set
```

Example:

```text
ZINCRBY product:sales 1 product:101
```

Top products:

```text
ZREVRANGE product:sales 0 99 WITHSCORES
```

---

# 95. Example: Recently Viewed Products

Requirement:

```text
Store user's recent products
```

Possible structure:

```text
List
```

But if you need:

```text
No duplicates
+
ordering
```

you may need a different model, such as a sorted set using timestamp as score.

---

# 96. Example: Online Users

Requirement:

```text
Is user online?
```

Set:

```text
SADD online-users 101
```

Check:

```text
SISMEMBER online-users 101
```

Remove:

```text
SREM online-users 101
```

---

# 97. Example: Session

Requirement:

```text
sessionId → user/session data
```

Possible:

```text
Hash
```

Example:

```text
session:abc
 ├── userId
 ├── role
 └── createdAt
```

with TTL:

```text
EXPIRE session:abc 1800
```

---

# 98. Example: Rate Limiting

Requirement:

```text
100 requests/minute
```

Possible:

```text
rate:user:101
```

Use:

```text
INCR
+
expiration
```

or a Lua implementation for an atomic fixed-window operation.

---

# 99. Example: Event Processing

Requirement:

```text
Order events need multiple consumers
```

Use:

```text
Redis Stream
```

Conceptually:

```text
orders
 ↓
consumer group
 ├── inventory-service
 ├── notification-service
 └── analytics-service
```

---

# 100. Choosing the Data Structure

Interview thought process:

```text
What is the primary operation?
       ↓
Read one value?
       → String

Read/update fields?
       → Hash

Queue?
       → List

Unique membership?
       → Set

Ranking?
       → Sorted Set

Durable event processing?
       → Stream

Approximate cardinality?
       → HyperLogLog

Nearby search?
       → Geospatial
```

---

# 101. Common Mistake

Don't store everything as:

```text
JSON String
```

just because it is convenient.

If your main operation is:

```text
membership
```

a Set may be better.

If:

```text
ranking
```

a Sorted Set is better.

If:

```text
field update
```

a Hash may be better.

---

# 102. Common Mistake

Don't use a Redis List as a full message broker without understanding its limitations.

For:

```text
Acknowledgements
Consumer groups
Pending messages
Recovery
```

Redis Streams are more suitable.

---

# 103. Common Mistake

Don't use a Set when ordering matters.

Sets provide:

```text
Uniqueness
```

but not the same score-based ordering semantics as Sorted Sets.

---

# 104. Common Mistake

Don't use Sorted Sets when you don't need scoring.

They add unnecessary complexity if the requirement is simply:

```text
Is this member present?
```

Use a Set.

---

# 105. Interview Question

### What Redis data structure would you use for a leaderboard?

Answer:

> "A Sorted Set, because each member can have a numeric score and Redis can efficiently retrieve members by rank or score."

---

# 106. Interview Question

### What would you use for unique membership?

Answer:

> "A Set. It gives uniqueness and efficient membership checks."

---

# 107. Interview Question

### What would you use for user profile fields?

Answer:

> "A Hash can be a good fit when the application frequently reads or updates individual fields. If the application always reads the whole object and uses JSON naturally, a serialized string can also be appropriate."

---

# 108. Interview Question

### What would you use for event processing?

Answer:

> "Redis Streams when I need persisted stream entries, consumer groups and acknowledgement-style processing. Pub/Sub is better for ephemeral real-time fan-out where missed messages are acceptable."

---

# 109. Interview Question

### Why not use KEYS?

Answer:

> "`KEYS` can scan the entire keyspace in a blocking operation. On a large production dataset that can cause latency. I'd generally use `SCAN` for incremental iteration."

---

# 110. Interview Question

### Is INCR atomic?

Answer:

> "Yes, an individual Redis INCR command is atomic from the perspective of Redis command execution. However, a read-modify-write sequence implemented as separate commands is not automatically atomic."

---

# 111. Interview Question

### How do you make multiple Redis operations atomic?

Answer:

> "Depending on the requirement, I can use a single atomic command, MULTI/EXEC with WATCH, or a Lua script. Lua is useful when several related operations need to execute as one Redis-side operation."

---

# 112. Interview Question

### List vs Stream?

Answer:

> "Lists are useful for simple queue-like workloads. Streams provide a richer event-processing model with persisted entries, consumer groups, acknowledgements and pending-message tracking."

---

# 113. Interview Question

### Set vs Sorted Set?

Answer:

> "A Set is for unique membership. A Sorted Set adds a numeric score and ordering, so it is useful for rankings, priorities and time-based indexes."

---

# 114. Interview Question

### String vs Hash?

Answer:

> "A String is useful for a single value or serialized object. A Hash is useful when I want field-level operations on an object without replacing the entire value."

---

# 115. Coding Scenario

### Build a leaderboard

Data:

```text
userId
score
```

Use:

```text
ZADD leaderboard score userId
```

Update:

```text
ZINCRBY leaderboard points userId
```

Top 10:

```text
ZREVRANGE leaderboard 0 9 WITHSCORES
```

---

# 116. Coding Scenario

### Build online-user tracking

Use:

```text
SADD online-users userId
```

Check:

```text
SISMEMBER online-users userId
```

Remove:

```text
SREM online-users userId
```

---

# 117. Coding Scenario

### Build a simple counter

```text
INCR page:views
```

Avoid:

```text
GET
+
application increment
+
SET
```

because concurrent requests can lose updates.

---

# 118. Coding Scenario

### Build a session

```text
HSET session:abc userId 101 role USER
EXPIRE session:abc 1800
```

If value + expiration must be coordinated carefully, use an appropriate atomic/pipeline pattern.

---

# 119. Coding Scenario

### Find users active in both groups

Use:

```text
SINTER group:a group:b
```

---

# 120. Coding Scenario

### Find top 10 users

Use:

```text
ZREVRANGE leaderboard 0 9 WITHSCORES
```

---

# 121. Coding Scenario

### Find events in a time range

Possible approach:

```text
Sorted Set
```

where:

```text
score = timestamp
member = eventId
```

Or use:

```text
Redis Stream
```

if you need event-stream semantics.

Choose based on whether you need:

```text
Index/range query
```

or:

```text
Event processing
```

---

# 122. Performance Checklist

Before using a Redis command in production:

```text
What is its time complexity?
How large can the key become?
How many results are returned?
How often is it called?
Can it block the server?
Does it create network overhead?
Is the data bounded?
```

---

# 123. Final Mental Model

Redis is:

```text
Key
 ↓
Data Structure
 ↓
Command
 ↓
Access Pattern
 ↓
Complexity
 ↓
Memory
 ↓
Network
```

The correct Redis design begins with the access pattern.

---

# 124. Revision Checklist

```text
□ Strings
□ SET
□ GET
□ MGET
□ MSET
□ GETDEL
□ GETEX
□ INCR
□ DECR
□ Hashes
□ HSET
□ HGET
□ HGETALL
□ HMGET
□ HDEL
□ HEXISTS
□ HINCRBY
□ Lists
□ LPUSH
□ RPUSH
□ LPOP
□ RPOP
□ LRANGE
□ LLEN
□ BLPOP
□ Sets
□ SADD
□ SREM
□ SISMEMBER
□ SMEMBERS
□ SCARD
□ SINTER
□ SUNION
□ SDIFF
□ Sorted Sets
□ ZADD
□ ZSCORE
□ ZRANK
□ ZRANGE
□ ZREVRANGE
□ ZINCRBY
□ Score ranges
□ Streams
□ XADD
□ XREAD
□ Consumer groups
□ XGROUP
□ XREADGROUP
□ XACK
□ Pending entries
□ Bitmaps
□ HyperLogLog
□ Geospatial
□ TTL commands
□ Atomicity
□ MULTI
□ EXEC
□ WATCH
□ Lua
□ Command complexity
□ KEYS vs SCAN
□ DEL vs UNLINK
□ Data modeling
□ E-commerce examples
```

---

# 125. What Comes Next

```text
File 03 → Redis Commands, Expiration & Memory Management
```

The next file will focus more deeply on:

```text
TTL behavior
Expiration internals
Eviction policies
maxmemory
LRU/LFU
Memory fragmentation
Big keys
Hot keys
SCAN patterns
Pipelining
Transactions
Lua scripts
Command complexity
Production-safe Redis operations
```

The key takeaway:

> **The best Redis developers don't just know commands. They understand the relationship between data structure, access pattern, command complexity, memory usage and concurrency.**
