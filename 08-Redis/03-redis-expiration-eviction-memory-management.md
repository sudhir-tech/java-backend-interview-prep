# Redis — File 03: Expiration, Eviction & Memory Management

This file focuses on one of the most important Redis production areas:

```text
TTL
Expiration
Eviction
maxmemory
LRU
LFU
Memory fragmentation
Big keys
Hot keys
SCAN
Pipelining
Transactions
Lua scripts
Production-safe operations
```

The interview goal is to understand not only how Redis stores data, but what happens when the dataset grows and memory becomes constrained.

---

# 1. Why Memory Management Matters

Redis is memory-first.

Therefore:

```text
More data
 ↓
More RAM
 ↓
Higher memory pressure
```

If memory reaches the configured limit, Redis may:

```text
Reject writes
Evict keys
Increase latency
Create operational pressure
```

depending on configuration.

---

# 2. maxmemory

Redis can be configured with a maximum memory limit:

```text
maxmemory
```

Conceptually:

```text
Redis
 ↓
Memory usage
 ↓
maxmemory
```

Once the limit is reached, the configured eviction policy determines what happens to eligible keys.

---

# 3. maxmemory Policy

Important policies include:

```text
noeviction
allkeys-lru
volatile-lru
allkeys-lfu
volatile-lfu
allkeys-random
volatile-random
volatile-ttl
```

The exact available policies depend on the Redis version.

---

# 4. noeviction

With:

```text
noeviction
```

Redis does not automatically evict keys to make room for writes.

When memory is exhausted:

```text
Write
 ↓
Rejected
```

This can be appropriate when losing cached data is unacceptable or when the application wants explicit capacity management.

---

# 5. allkeys-lru

Redis considers keys across the dataset and attempts to evict less recently used keys.

Conceptually:

```text
Old/less recently used
        ↓
Evict
```

Useful for:

```text
General-purpose cache
```

where all cached keys are candidates.

---

# 6. volatile-lru

Only keys with an expiration are eligible for LRU eviction.

This is useful when:

```text
Some keys are persistent
Some keys are temporary/cache entries
```

The expiration metadata becomes part of the eligibility.

---

# 7. allkeys-lfu

Uses a frequency-oriented policy across eligible keys.

Conceptually:

```text
Rarely accessed
 ↓
Eviction candidate
```

Useful when:

```text
Access frequency is more meaningful than recency
```

for example, workloads with very popular keys that should remain cached.

---

# 8. volatile-lfu

Only keys with expiration are considered, using frequency-oriented eviction.

Useful when:

```text
Persistent data
+
expiring cache data
```

share the same Redis instance.

---

# 9. allkeys-random

Redis can evict random keys from the eligible dataset.

This is generally less useful for a normal cache when access patterns contain meaningful locality.

---

# 10. volatile-random

Random eviction among keys that have an expiration.

This can be useful for specialized workloads but is less common than LRU/LFU cache policies.

---

# 11. volatile-ttl

Prioritizes keys based on their remaining TTL.

Conceptually:

```text
Shorter remaining lifetime
 ↓
More likely eviction candidate
```

This can make sense when expiration time itself reflects data importance.

---

# 12. LRU Meaning

LRU means:

```text
Least Recently Used
```

Example:

```text
A → accessed recently
B → accessed recently
C → not accessed for a long time
```

C becomes a stronger eviction candidate.

---

# 13. LFU Meaning

LFU means:

```text
Least Frequently Used
```

Example:

```text
A → accessed 10,000 times
B → accessed 2 times
C → accessed 1 time
```

C is a stronger eviction candidate.

---

# 14. LRU vs LFU

```text
LRU
→ How recently was it used?

LFU
→ How frequently is it used?
```

Choose based on workload.

---

# 15. Important Redis LRU Detail

Redis does not necessarily maintain a perfect global LRU ordering for every key.

It uses an approximate algorithm to keep memory and CPU overhead reasonable.

Interview answer:

> "Redis uses an approximation of LRU rather than maintaining an exact global ordering, which gives a good balance between eviction quality and performance."

---

# 16. LFU Detail

Redis's LFU implementation also uses approximations rather than maintaining an exact access counter with unlimited precision.

The goal is:

```text
Good eviction decisions
+
Low metadata overhead
```

---

# 17. TTL

TTL means:

```text
Time To Live
```

Example:

```text
SET product:101 "data" EX 300
```

The key has approximately:

```text
300 seconds
```

of lifetime.

---

# 18. TTL Commands

Important commands:

```text
EXPIRE
PEXPIRE
TTL
PTTL
PERSIST
EXPIREAT
PEXPIREAT
```

---

# 19. TTL

```text
TTL product:101
```

returns remaining lifetime in seconds.

---

# 20. PTTL

```text
PTTL product:101
```

returns remaining lifetime in milliseconds.

Useful when:

```text
Fine-grained expiration
```

matters.

---

# 21. PERSIST

```text
PERSIST product:101
```

removes the expiration.

The key remains until another operation deletes it.

---

# 22. EXPIREAT

```text
EXPIREAT key timestamp
```

sets an expiration using an absolute Unix timestamp.

Useful when the application has:

```text
Known deadline
```

rather than:

```text
Duration
```

---

# 23. TTL Special Cases

When querying TTL, Redis can return special negative values indicating situations such as:

```text
Key does not exist
Key exists but has no expiration
```

Always account for these states when debugging cache behavior.

---

# 24. Does Expired Data Disappear Exactly at TTL?

Not necessarily at the exact millisecond.

Redis manages expiration through internal expiration mechanisms.

Conceptually:

```text
TTL reaches zero
 ↓
Key becomes expired
 ↓
Redis removes/ignores it according to expiration processing
```

The important application-level rule is:

> Do not depend on expiration as a precise scheduling mechanism.

---

# 25. Active vs Lazy Expiration

Redis can encounter expired keys in different ways.

### Lazy expiration

A key can be discovered as expired when it is accessed.

### Active expiration

Redis also periodically samples keys with expiration and removes expired keys proactively.

This prevents expired keys from accumulating indefinitely.

---

# 26. Why Both?

If Redis only used lazy expiration:

```text
Expired but never-read keys
 ↓
Could remain in memory
```

Active expiration helps clean them up.

If Redis only performed continuous full scans:

```text
CPU cost
```

could become too high.

The combination provides a practical balance.

---

# 27. TTL and Cache Freshness

TTL is not the same thing as data correctness.

Example:

```text
Product price
```

with:

```text
TTL = 1 hour
```

means:

```text
Cached value may remain for up to roughly that period
```

unless invalidated earlier.

If business correctness requires immediate updates, use:

```text
Explicit invalidation
```

instead of relying only on TTL.

---

# 28. TTL + Invalidation

A common strategy:

```text
Read
 ↓
Redis cache
 ↓
TTL

Write
 ↓
Database
 ↓
Invalidate Redis key
```

Then:

```text
Next read
 ↓
Cache miss
 ↓
Database
 ↓
Fresh cache
```

TTL becomes a safety net rather than the only consistency mechanism.

---

# 29. Cache Invalidation Problem

Suppose:

```text
MySQL:
price = 100

Redis:
price = 100
```

Application updates MySQL:

```text
price = 120
```

but Redis remains:

```text
price = 100
```

Now the cache is stale.

Solution:

```text
Update DB
 ↓
Invalidate cache
```

or use another carefully designed cache update strategy.

---

# 30. Delete vs Update Cache

Two common approaches after a database write:

### Invalidate

```text
DB update
 ↓
DEL cache:key
```

Next read repopulates it.

### Update cache

```text
DB update
 ↓
SET cache:key new-value
```

Invalidation is often simpler because it avoids maintaining a second write path for cached data.

---

# 31. Cache Consistency

The difficult question is:

```text
Which copy is authoritative?
```

Usually:

```text
MySQL
 ↓
Source of truth

Redis
 ↓
Derived cache
```

This simplifies recovery:

```text
Redis lost
 ↓
Rebuild from MySQL
```

---

# 32. Cache Stampede

Suppose:

```text
Popular key expires
```

Then:

```text
10,000 requests
 ↓
Cache miss
 ↓
10,000 DB requests
```

This is a cache stampede.

---

# 33. Preventing Stampede

Possible strategies:

```text
Request coalescing
Distributed lock
Jittered TTL
Refresh ahead
Stale-while-revalidate
Prewarming
```

The best solution depends on workload.

---

# 34. Jittered TTL

Instead of:

```text
TTL = 300 seconds
```

for every key:

```text
TTL = 300 + random(0..60)
```

This spreads expirations over time.

Conceptually:

```text
Same expiration
 ↓
Stampede risk

Randomized expiration
 ↓
More distributed misses
```

---

# 35. Cache Penetration

A request repeatedly asks for data that doesn't exist:

```text
product:999999
```

Flow:

```text
Redis miss
 ↓
MySQL miss
 ↓
No cache entry
 ↓
Next request repeats
```

---

# 36. Negative Caching

Store a short-lived marker:

```text
product:999999 → NOT_FOUND
```

with a short TTL.

Then:

```text
Request
 ↓
Redis
 ↓
NOT_FOUND
 ↓
Return 404
```

Be careful with the TTL because the object may be created later.

---

# 37. Cache Avalanche

A cache avalanche occurs when many cache entries become unavailable at approximately the same time.

Causes:

```text
Same TTL
Redis restart
Mass invalidation
Network failure
```

Result:

```text
Database receives huge traffic
```

---

# 38. Avalanche Prevention

Use:

```text
TTL jitter
Staggered expiration
Cache warming
Fallback
Rate limiting
Load shedding
Circuit breakers
```

Also avoid designing a cache whose entire contents expire at one exact moment.

---

# 39. Big Keys

A big key can be:

```text
Huge String
Huge Hash
Huge List
Huge Set
Huge Sorted Set
```

Example:

```text
user:all-data
```

containing millions of members.

---

# 40. Why Big Keys Are Dangerous

They can cause:

```text
High memory usage
Large network responses
Long command execution
Slow deletion
Replication pressure
Backup overhead
```

---

# 41. Avoid Huge Values

Instead of:

```text
user:101 → 50 MB JSON
```

consider:

```text
user:101:profile
user:101:orders
user:101:preferences
```

depending on access patterns.

Bound data structures where possible.

---

# 42. Large Collection Trap

This is dangerous on a huge collection:

```text
SMEMBERS huge-set
```

because Redis may need to return every member.

Prefer:

```text
SSCAN
```

when incremental iteration is appropriate.

---

# 43. Hot Keys

A hot key receives extremely high traffic.

Example:

```text
product:1
```

receives:

```text
500,000 requests/sec
```

Even with a Redis Cluster:

```text
One key
 ↓
One hash slot
 ↓
One primary
```

can become a bottleneck.

---

# 44. Hot Key Mitigation

Potential approaches:

```text
Application-local cache
Replication patterns
Request coalescing
Key spreading/sharding
Traffic shaping
```

The correct approach depends on whether the data is:

```text
Read-only
Mutable
Consistent
Frequently invalidated
```

---

# 45. Memory Fragmentation

Redis can have:

```text
Used memory
```

different from:

```text
Memory allocated by the operating system
```

due to allocator behavior and fragmentation.

A process can therefore appear to use more system memory than the logical dataset size suggests.

---

# 46. Fragmentation Ratio

Redis exposes memory statistics including a fragmentation ratio.

Conceptually:

```text
RSS / logical memory
```

A high ratio can indicate memory fragmentation or allocator overhead.

Interpret metrics in context rather than treating one ratio as an automatic failure.

---

# 47. Why Fragmentation Happens

Memory can become fragmented because of:

```text
Different object sizes
Allocations
Deallocations
Allocator behavior
Changing key sizes
```

Long-running workloads can develop fragmentation patterns.

---

# 48. Memory Optimization

Strategies:

```text
Avoid unnecessarily large values
Use compact structures
Set TTLs where appropriate
Bound collections
Remove stale data
Choose efficient serialization
Monitor fragmentation
```

---

# 49. maxmemory + Cache Policy

For a pure cache:

```text
maxmemory
+
eviction policy
```

is often intentional.

Example:

```text
allkeys-lru
```

means:

```text
When memory is full
 ↓
Evict less recently used cache entries
```

---

# 50. Cache vs Persistent Redis

If Redis is only:

```text
Cache
```

losing keys may be acceptable.

If Redis contains:

```text
Primary business data
```

automatic eviction can be dangerous.

Therefore:

> Eviction policy must match the role Redis plays in the architecture.

---

# 51. Pipelining

Pipelining allows multiple commands to be sent without waiting for each response individually.

Without pipeline:

```text
Request
 ↓
Response
 ↓
Request
 ↓
Response
```

With pipeline:

```text
Request 1
Request 2
Request 3
Request 4
 ↓
Redis
 ↓
Responses
```

This reduces network round trips.

---

# 52. Pipeline Does Not Mean Transaction

Important:

```text
Pipeline
≠
Transaction
```

Pipeline is primarily about:

```text
Reducing round trips
```

It does not by itself provide transactional semantics.

---

# 53. Pipeline Example

Suppose you need:

```text
GET user:101
GET user:102
GET user:103
GET user:104
```

A pipeline can send these together.

This is useful when:

```text
Many independent commands
```

need to be executed.

---

# 54. Pipeline Trade-Off

Pipelining can improve throughput, but:

```text
Huge pipeline
```

can create:

```text
Large client memory
Large response
Long processing burst
```

Use reasonable batch sizes.

---

# 55. Transactions

Redis transaction commands:

```text
MULTI
EXEC
DISCARD
WATCH
```

A transaction queues commands and executes them sequentially.

---

# 56. MULTI

```text
MULTI
```

starts transaction mode.

Commands after it are queued.

---

# 57. EXEC

```text
EXEC
```

executes queued commands.

---

# 58. DISCARD

```text
DISCARD
```

cancels the queued transaction.

---

# 59. WATCH

`WATCH` provides optimistic concurrency control.

Conceptually:

```text
WATCH stock:101
GET stock:101

calculate

MULTI
SET stock:101 ...
EXEC
```

If another client modifies the watched key:

```text
EXEC
 ↓
transaction aborts
```

The application can retry.

---

# 60. Redis Transactions and Rollback

Redis transactions are not equivalent to SQL transactions with rollback.

Important:

```text
Commands execute sequentially
```

If a runtime command error occurs:

```text
Previously executed commands are not automatically undone
```

Therefore design carefully.

---

# 61. Lua Scripts

Lua scripts run server-side.

Useful for:

```text
Atomic multi-step operations
Rate limiting
Conditional updates
Lock release
Complex state transitions
```

---

# 62. Why Lua Helps

Without Lua:

```text
GET
 ↓
Application
 ↓
calculate
 ↓
SET
```

has a race window.

With Lua:

```text
Application
 ↓
Lua script
 ↓
Redis executes logic as one operation
```

This can eliminate the application-side race between the steps.

---

# 63. Lua Trade-Off

Don't put huge workloads into Lua.

A long-running script can block Redis command processing.

Keep scripts:

```text
Small
Deterministic
Efficient
```

---

# 64. Blocking Operations

Redis is designed for low latency, so expensive/blocking commands should be avoided.

Examples to be careful with:

```text
KEYS *
Huge SMEMBERS
Huge LRANGE
Large Lua scripts
Very large deletes
```

---

# 65. SCAN

Use:

```text
SCAN cursor
```

to iterate through the keyspace incrementally.

Example concept:

```text
SCAN 0 MATCH product:* COUNT 100
```

The result includes:

```text
next cursor
keys
```

Continue until:

```text
cursor = 0
```

---

# 66. SCAN Is Not a Snapshot

Important interview detail:

`SCAN` is incremental iteration, not a consistent point-in-time snapshot.

During iteration:

```text
Keys can be added
Keys can be removed
```

Therefore application logic must tolerate iteration semantics.

---

# 67. HSCAN

For a large hash:

```text
HSCAN user:101 0
```

This can iterate fields incrementally.

---

# 68. SSCAN

For a large set:

```text
SSCAN online-users 0
```

Useful when:

```text
Set is large
```

and you don't want to retrieve all members at once.

---

# 69. ZSCAN

For a large sorted set:

```text
ZSCAN leaderboard 0
```

iterates incrementally.

---

# 70. Monitoring Memory

Useful Redis information commands include:

```text
INFO memory
MEMORY USAGE key
MEMORY STATS
```

depending on Redis version and operational needs.

---

# 71. MEMORY USAGE

Conceptually:

```text
MEMORY USAGE product:101
```

helps identify large keys.

This is useful during:

```text
Big-key investigation
Memory incident
Capacity planning
```

---

# 72. Slow Commands

If Redis latency is high, investigate:

```text
Slow commands
Large payloads
Hot keys
CPU
Network
Persistence
Client behavior
```

Do not assume the database is the cause just because Redis is involved.

---

# 73. Redis Slow Log

Redis can record commands that exceed a configured execution threshold.

This helps identify:

```text
Unexpectedly expensive commands
```

Remember:

> Slowlog measures command execution time on Redis and doesn't represent all network/client latency.

---

# 74. Latency Monitoring

Monitor:

```text
Command latency
Network latency
Application-to-Redis latency
P99/P95 latency
```

Averages can hide tail-latency problems.

---

# 75. Eviction Monitoring

Monitor:

```text
evicted_keys
```

If evictions suddenly increase:

```text
Memory pressure
```

may be occurring.

But whether eviction is bad depends on the cache design.

---

# 76. Cache Hit Rate

A basic cache metric:

```text
hits / (hits + misses)
```

Higher is usually better, but:

> A high hit rate does not automatically mean a healthy cache.

You also need to consider:

```text
Latency
Memory
Staleness
Database load
Cost
```

---

# 77. Cache Miss

A miss can happen because:

```text
Key never existed
TTL expired
Evicted
Explicitly deleted
Redis restarted
Wrong key
Serialization mismatch
```

Always investigate the actual reason when debugging.

---

# 78. Key Naming and Memory

Good:

```text
product:101
```

Bad:

```text
this_is_a_very_long_key_name_for_a_product_object_that_contains...
```

Key names are metadata too.

Avoid unnecessary key-name bloat at massive scale.

---

# 79. Cache Key Versioning

Use version prefixes when schema changes:

```text
product:v1:101
```

Later:

```text
product:v2:101
```

This can prevent new code from attempting to deserialize incompatible old values.

---

# 80. Serialization and Memory

Suppose JSON contains:

```json
{
  "id": 101,
  "name": "Laptop",
  "description": "..."
}
```

Large repeated field names increase memory and network usage.

Consider:

```text
Compact representation
Appropriate fields
Compression where justified
```

But don't optimize serialization prematurely.

---

# 81. Redis Memory Incident

Suppose memory suddenly reaches:

```text
95%
```

Investigation:

```text
1. INFO memory
2. Check evictions
3. Find big keys
4. Check key growth
5. Check TTL coverage
6. Check recent deployment
7. Check serialization changes
8. Check traffic pattern
```

Then fix root cause.

---

# 82. Redis Incident: Evictions Suddenly Increase

Possible causes:

```text
Traffic increase
Larger values
More keys
TTL changes
New cache feature
Wrong eviction policy
Memory limit reduction
```

Don't simply increase RAM without understanding the change.

---

# 83. Redis Incident: Database Suddenly Overloaded

Redis metrics:

```text
Hit rate ↓
Misses ↑
```

Potential causes:

```text
Mass expiration
Cache flush
Serialization failure
Key namespace change
Redis outage
Eviction spike
```

Result:

```text
Cache miss
 ↓
Database overload
```

This is why Redis outages can indirectly become database outages.

---

# 84. Cache Failure Protection

If Redis becomes unavailable:

```text
Application
 ↓
Redis failure
 ↓
Fallback DB
```

Potentially dangerous.

Use:

```text
Circuit breaker
Rate limiting
Load shedding
Request prioritization
Stale cache where appropriate
Database protection
```

to prevent a thundering herd against the database.

---

# 85. Cache Warming

After:

```text
Redis restart
```

the cache may be empty.

Cache warming means:

```text
Load popular data
 ↓
Redis
```

before traffic fully ramps up.

Useful for:

```text
Hot product catalogs
Configuration
Reference data
```

---

# 86. Refresh Ahead

Instead of waiting for expiration:

```text
TTL nearly expired
 ↓
Refresh in background
```

This can reduce cache misses for very popular data.

Trade-off:

```text
Extra refresh traffic
```

---

# 87. Stale-While-Revalidate

A strategy can allow:

```text
Serve slightly stale value
+
refresh cache asynchronously
```

This is useful when:

```text
Low latency
```

is more important than:

```text
Perfect freshness
```

and the business accepts bounded staleness.

---

# 88. Redis Persistence and Eviction Are Different

Persistence answers:

```text
How do we recover data after restart?
```

Eviction answers:

```text
What happens when memory is full?
```

They solve different problems.

---

# 89. Redis Replication and Persistence Are Different

Replication:

```text
Primary
 ↓
Replica
```

helps availability/read scaling.

Persistence:

```text
Memory
 ↓
RDB/AOF
```

helps recovery/durability.

You may use both.

---

# 90. Redis Sentinel and Cluster

Sentinel:

```text
Failure detection
Failover
Primary discovery
```

Cluster:

```text
Sharding
Horizontal scaling
High availability features
```

Again:

```text
Sentinel ≠ Cluster
```

---

# 91. Production-Safe Redis Rule

Before running a command, ask:

```text
How many keys?
How large are they?
How many clients?
What is the complexity?
Can it block?
What happens under failure?
```

---

# 92. Dangerous Pattern

Avoid:

```text
KEYS *
```

during normal production traffic.

Prefer:

```text
SCAN
```

---

# 93. Dangerous Pattern

Avoid:

```text
HGETALL huge-hash
```

if you only need:

```text
2 fields
```

Use:

```text
HMGET
```

---

# 94. Dangerous Pattern

Avoid:

```text
SMEMBERS huge-set
```

if you need:

```text
one membership check
```

Use:

```text
SISMEMBER
```

---

# 95. Dangerous Pattern

Avoid:

```text
LRANGE huge-list 0 -1
```

if you only need:

```text
latest 20
```

Use:

```text
LRANGE list 0 19
```

---

# 96. Dangerous Pattern

Avoid:

```text
DEL huge-key
```

without considering deletion latency.

For very large structures, investigate:

```text
UNLINK
```

where appropriate.

---

# 97. Interview Question

### What happens when Redis reaches maxmemory?

Answer:

> "It depends on the configured maxmemory policy. With noeviction, writes that need additional memory can fail. With an eviction policy such as allkeys-LRU or allkeys-LFU, Redis can remove eligible keys to make room."

---

# 98. Interview Question

### LRU vs LFU?

Answer:

> "LRU considers recency of access, while LFU considers access frequency. LRU can work well when recent usage predicts future usage; LFU can be useful when a small set of keys stays consistently popular."

---

# 99. Interview Question

### What is cache stampede?

Answer:

> "It's when many requests miss the same cache entry at roughly the same time, often after expiration, causing a sudden load spike on the backing database."

---

# 100. Interview Question

### How do you prevent cache avalanche?

Answer:

> "I can stagger expiration with TTL jitter, avoid synchronized invalidation, warm important keys and use fallback/load-shedding strategies so a cache failure doesn't overwhelm the database."

---

# 101. Interview Question

### Why is KEYS dangerous?

Answer:

> "`KEYS` can perform a large keyspace scan in a blocking operation. On a large production Redis instance that can create significant latency. I would normally use `SCAN` for incremental iteration."

---

# 102. Interview Question

### Is pipeline a transaction?

Answer:

> "No. Pipelining primarily reduces network round trips. It doesn't by itself provide transaction semantics."

---

# 103. Interview Question

### What is a big key?

Answer:

> "A big key is an unusually large Redis value or collection. It can cause memory pressure, high network traffic, long command execution and replication or deletion problems."

---

# 104. Interview Question

### What is a hot key?

Answer:

> "A hot key receives disproportionately high traffic and can overload the Redis node responsible for that key, even when the overall cluster has available capacity."

---

# 105. Interview Question

### Why use TTL if you already invalidate cache entries?

Answer:

> "TTL acts as a safety mechanism against stale or orphaned entries. Explicit invalidation handles normal writes, while TTL prevents a bug or missed invalidation from keeping stale data forever."

---

# 106. Interview Question

### What is the difference between expiration and eviction?

Answer:

> "Expiration removes data because its configured lifetime has ended. Eviction removes eligible data because Redis needs memory according to its configured memory policy."

---

# 107. Interview Question

### Why can Redis use more memory than the data size?

Answer:

> "Memory usage also includes object overhead, allocator overhead and fragmentation. Therefore logical dataset size and process RSS are not necessarily the same."

---

# 108. Interview Question

### How would you investigate high Redis memory?

Answer:

> "I'd inspect INFO memory, eviction metrics and key growth, identify big keys, check TTL coverage, review recent deployments and serialization changes, and then determine whether the issue is expected growth, fragmentation or an application-level memory leak."

---

# 109. Interview Question

### How would you investigate Redis latency?

Answer:

> "I'd check command latency and slow commands, look for large values or collections, hot keys, expensive scripts, CPU and network pressure, persistence activity and client behavior. I'd focus on P95/P99 latency rather than only averages."

---

# 110. Interview Scenario

### Cache hit rate dropped after deployment.

Investigate:

```text
Key format
Serialization
TTL
Cache invalidation
Cache namespace
Application version
Evictions
```

Potential cause:

```text
v1 key
 ↓
deployment starts writing v2 key
 ↓
old cache ignored
 ↓
miss rate increases
```

---

# 111. Interview Scenario

### Redis memory reaches 100%.

First:

```text
Protect availability
```

Then investigate:

```text
Big keys
Unbounded collections
Missing TTL
Unexpected key growth
Serialization changes
Traffic changes
```

Don't immediately flush Redis.

---

# 112. Interview Scenario

### Database overload occurs every hour.

Possible Redis clue:

```text
Many keys expire simultaneously
```

Check:

```text
TTL distribution
```

If synchronized:

```text
Add TTL jitter
```

and/or use refresh-ahead for important hot data.

---

# 113. Interview Scenario

### One product receives enormous traffic.

Possible:

```text
Hot key
```

Mitigation:

```text
Local application cache
Request coalescing
Replication/sharding patterns
```

depending on consistency requirements.

---

# 114. Interview Scenario

### You need to iterate millions of keys.

Don't use:

```text
KEYS *
```

Use:

```text
SCAN
```

and process in bounded batches.

---

# 115. Interview Scenario

### You need to retrieve only two fields from a huge hash.

Don't use:

```text
HGETALL
```

Use:

```text
HMGET
```

This reduces:

```text
Response size
Network traffic
Client processing
```

---

# 116. Interview Scenario

### You need one item from a huge set.

Don't use:

```text
SMEMBERS
```

Use:

```text
SISMEMBER
```

if the requirement is only membership.

---

# 117. Interview Scenario

### Redis is used as a cache and restarts.

Expected:

```text
Some/all cache data may be unavailable
```

Application should:

```text
Fallback
 ↓
Database
 ↓
Repopulate cache
```

But protect the database against a cache-miss storm.

---

# 118. Redis Production Checklist

```text
□ Define maxmemory
□ Choose eviction policy
□ Understand TTL
□ Avoid synchronized expiration
□ Monitor memory
□ Monitor fragmentation
□ Identify big keys
□ Identify hot keys
□ Monitor hit rate
□ Monitor evictions
□ Avoid KEYS
□ Use SCAN
□ Bound collections
□ Use pipelines for many independent commands
□ Keep pipeline sizes reasonable
□ Use transactions intentionally
□ Keep Lua scripts small
□ Monitor slow commands
□ Protect database fallback
□ Plan cache warming
□ Design invalidation
□ Version serialized cache values
□ Test Redis failure
□ Test recovery
```

---

# 119. Redis Memory Mental Model

Think:

```text
Application
     ↓
Redis
     ↓
Data structures
     ↓
Memory
     ↓
maxmemory
     ↓
Eviction policy
```

At the same time:

```text
TTL
 ↓
Expiration
```

and:

```text
Persistence
 ↓
RDB / AOF
```

and:

```text
Replication
 ↓
Replica
```

These are separate mechanisms.

---

# 120. Final Interview Answer

If asked:

> "How would you manage Redis memory in production?"

Say:

> "I'd first define a realistic maxmemory limit and choose an eviction policy based on whether Redis is purely a cache or also holds persistent application data. I'd use TTLs for temporary data, monitor memory, evictions, fragmentation and big keys, and avoid unbounded collections. For large-scale inspection I'd use SCAN rather than KEYS. I'd also monitor hot keys and cache hit rate and protect the backing database from a cache-miss storm."

---

# 121. What Comes Next

```text
File 04 → Redis Caching Patterns & Spring Boot Integration
```

The next file will connect Redis to real Java backend development:

```text
Spring Boot
RedisTemplate
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
Cache invalidation
TTL configuration
E-commerce examples
```

The key takeaway:

> **Redis performance is not just about RAM and fast commands. Production Redis requires careful control of expiration, eviction, memory growth, command complexity, cache consistency and failure behavior.**
