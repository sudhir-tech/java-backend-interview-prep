# System Design — File 08: Database Replication, Partitioning & Sharding

This file goes deeper into three major database scaling techniques:

```text
Replication
Partitioning
Sharding
```

They solve different problems.

A useful mental model is:

```text
Replication
→ More copies of the data

Partitioning
→ Split data into logical pieces

Sharding
→ Distribute partitions across database nodes
```

---

# 1. Why Do We Need These Techniques?

A single database may eventually face:

```text
Too many reads
Too many writes
Too much data
Too many connections
Hardware limits
Availability requirements
```

Before introducing complex distributed architecture, optimize:

```text
Schema
Queries
Indexes
Connection pool
Caching
```

Then choose the scaling technique that addresses the actual bottleneck.

---

# 2. Replication

Replication means maintaining copies of data on multiple database nodes.

Basic architecture:

```text
             Primary
            /       \
           ↓         ↓
       Replica 1   Replica 2
```

The primary commonly handles writes.

Replicas can handle suitable reads.

---

# 3. Why Replicate?

Replication can provide:

```text
Read scaling
Redundancy
Failover capability
Disaster recovery
Geographic copies
```

---

# 4. Primary and Replica

Example:

```text
Application
     |
     +---- Writes ----> Primary
     |
     +---- Reads -----> Replica
```

The primary sends changes to replicas.

Conceptually:

```text
Primary
   ↓
Replication
   ↓
Replica
```

---

# 5. Primary Is Sometimes Called Master

You may see terminology such as:

```text
Primary / Replica
Master / Slave
Leader / Follower
```

Modern systems increasingly prefer:

```text
Primary / Replica
Leader / Follower
```

Use the terminology of the specific database.

---

# 6. Synchronous Replication

The primary waits for replication acknowledgment before considering the operation complete according to the configured replication semantics.

Conceptually:

```text
Client
  ↓
Primary
  ↓
Replica
  ↓
Acknowledgement
  ↓
Client
```

Potential benefit:

```text
Stronger durability/consistency guarantees
```

Potential cost:

```text
Higher write latency
```

---

# 7. Asynchronous Replication

The primary can acknowledge the write before replicas have necessarily received it.

```text
Client
  ↓
Primary
  ↓
Acknowledgement
  ↓
Client

Primary
  ↓
Replica
```

Benefits:

```text
Lower write latency
Better geographic flexibility
```

Risk:

```text
Replica lag
Potential data loss during certain failures
```

---

# 8. Replication Lag

Suppose:

```text
Primary:
order status = PAID
```

but:

```text
Replica:
order status = PENDING
```

for a short period.

This is:

```text
Replication lag
```

---

# 9. Why Replication Lag Matters

Imagine:

```text
POST /orders
```

writes to primary.

Then:

```text
GET /orders/101
```

goes to a replica.

The user may temporarily see:

```text
Order not found
```

or an old state.

This is a read-after-write consistency problem.

---

# 10. Handling Read-After-Write

Possible approaches:

```text
Read from primary
Session-aware routing
Sticky consistency
Wait for replication
Use a consistency-aware data store
```

The appropriate choice depends on the business requirement.

---

# 11. Read Scaling

Suppose:

```text
100K read RPS
10K write RPS
```

One primary may become overloaded by reads.

Architecture:

```text
                 +--- Replica 1
                 |
App → DB Router -+--- Replica 2
                 |
                 +--- Replica 3
                 |
                 +--- Primary
```

Reads can be distributed.

Writes go to the primary.

---

# 12. Replication Does Not Automatically Scale Writes

This is an important interview point.

Adding:

```text
10 read replicas
```

doesn't automatically increase:

```text
Primary write capacity
```

Replication primarily helps read scaling.

Write scaling needs other techniques.

---

# 13. Primary Write Bottleneck

Suppose:

```text
Reads = 90%
Writes = 10%
```

Read replicas can help significantly.

But if:

```text
Writes = 90%
```

the primary may still be the bottleneck.

Possible approaches:

```text
Write optimization
Batching
Partitioning
Sharding
Distributed database
```

---

# 14. Replication Topologies

### Primary → Replicas

```text
Primary
 ├── Replica 1
 ├── Replica 2
 └── Replica 3
```

Simple and common.

---

# 15. Cascading Replication

Instead of all replicas reading directly from the primary:

```text
Primary
   ↓
Replica 1
   ↓
Replica 2
```

This can reduce direct replication load on the primary in some architectures.

But it can increase replication lag and operational complexity.

---

# 16. Multi-Primary Replication

Multiple nodes can accept writes:

```text
Primary A ←→ Primary B
```

Potential benefit:

```text
Write distribution
Geographic write locality
```

But conflicts become much harder.

You need strategies for:

```text
Conflict resolution
Ordering
Consistency
Failover
```

Don't introduce multi-primary without a clear requirement.

---

# 17. Failover

Suppose:

```text
Primary
   X
```

A replica may be promoted:

```text
Replica 1
   ↓
New Primary
```

Applications need a way to discover the new primary.

This can be handled by:

```text
Managed database service
Proxy
Service discovery
DNS
Database cluster manager
```

depending on the environment.

---

# 18. Failover Risk

Failover isn't just:

```text
Primary dies
→ Replica becomes primary
```

You also need to consider:

```text
Replication lag
In-flight transactions
Client connections
Application retries
Split brain
Data loss
```

---

# 19. Split Brain

A dangerous situation occurs when multiple nodes believe they are the primary.

Example:

```text
Node A → thinks it is primary
Node B → thinks it is primary
```

Both accept writes.

This can produce conflicting data.

Distributed coordination mechanisms are needed to avoid this.

---

# 20. Replication and Backups

Replication:

```text
Improves availability/read scaling
```

Backup:

```text
Provides recovery from data loss/corruption
```

Replication does not replace backups.

If bad data is replicated:

```text
Bad data
  ↓
All replicas
```

you still need recovery capability.

---

# 21. Partitioning

Partitioning divides a large logical dataset into smaller pieces.

Example:

```text
Orders
  |
  +--- January
  +--- February
  +--- March
```

The database can manage each partition separately.

---

# 22. Why Partition?

Potential benefits:

```text
Smaller working sets
Partition pruning
Faster maintenance
Archiving
Parallel operations
Manageability
```

Not every partitioned table is automatically faster.

---

# 23. Horizontal Partitioning

Rows are divided across partitions.

Example:

```text
Users 1–1M
→ Partition A

Users 1M–2M
→ Partition B

Users 2M–3M
→ Partition C
```

---

# 24. Vertical Partitioning

Columns are split.

Example:

```text
User Core
---------
id
name
email

User Profile
------------
user_id
bio
preferences
avatar
```

Potentially useful when:

```text
Some fields are rarely accessed
Rows are very wide
Access patterns differ
```

---

# 25. Range Partitioning

Partition based on ranges.

Example:

```text
Orders:
2026-01 → Partition 1
2026-02 → Partition 2
2026-03 → Partition 3
```

Useful for:

```text
Time-series data
Logs
Orders
Events
```

---

# 26. Range Partitioning Problem

Suppose all new writes go to:

```text
Current month
```

Then:

```text
Current partition
```

receives almost all writes.

This can create a hotspot.

---

# 27. Hash Partitioning

Partition based on a hash.

Conceptually:

```text
hash(user_id) → partition
```

Example:

```text
User 101 → Partition 2
User 102 → Partition 4
User 103 → Partition 1
```

Benefits:

```text
Usually more even distribution
```

Trade-off:

```text
Range queries become harder
```

---

# 28. List Partitioning

Partition based on known values.

Example:

```text
Region:
India → Partition A
US → Partition B
Europe → Partition C
```

Potential problem:

```text
Uneven traffic
```

if one region is much larger.

---

# 29. Partition Pruning

Suppose data is partitioned by:

```text
created_at
```

Query:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-01'
  AND created_at < '2026-09-01';
```

The database may only need to inspect the relevant partition(s).

This is:

```text
Partition pruning
```

---

# 30. Partitioning vs Indexing

Index:

```text
Helps locate rows within a table/partition
```

Partitioning:

```text
Splits the table into larger logical pieces
```

They can be used together.

---

# 31. Partitioning vs Sharding

This distinction is important.

### Partitioning

Data is divided logically.

It may still live:

```text
Inside one database system
```

### Sharding

Partitions are distributed across:

```text
Multiple database nodes
```

---

# 32. Sharding

Sharding is horizontal distribution of data across separate database nodes.

Example:

```text
             Application
                  |
            Shard Router
          /       |       \
         ↓        ↓        ↓
      Shard 1  Shard 2  Shard 3
```

Each shard stores part of the data.

---

# 33. Why Shard?

Potential reasons:

```text
Dataset too large for one node
Write throughput too high
Storage limits
CPU limits
Connection limits
Geographic distribution
```

---

# 34. Don't Shard Too Early

Before sharding:

```text
Optimize queries
Optimize indexes
Use cache
Scale vertically
Use read replicas
Partition if useful
```

Sharding introduces major complexity.

---

# 35. Shard Key

The shard key decides:

```text
Which shard stores a record?
```

Example:

```text
user_id
```

Potential mapping:

```text
hash(user_id)
    ↓
Shard
```

---

# 36. Good Shard Key

A good shard key should provide:

```text
Even distribution
High cardinality
Stable value
Good query locality
```

---

# 37. Bad Shard Key

Suppose:

```text
country
```

and:

```text
India = 70% users
```

Then:

```text
India shard → 70%
Other shards → 30%
```

This is poor distribution.

---

# 38. High Cardinality

Example:

```text
country
```

has relatively low cardinality.

```text
user_id
```

has much higher cardinality.

High cardinality often helps distribution.

But:

```text
High cardinality
≠
Automatically good shard key
```

Query patterns still matter.

---

# 39. Query Locality

Suppose the main query is:

```text
Get all orders for user 101
```

If sharding by:

```text
user_id
```

all of that user's orders can be stored on one shard.

Then:

```text
Get user orders
      ↓
One shard
```

This is excellent locality.

---

# 40. Scatter-Gather Query

Suppose data is sharded by:

```text
user_id
```

but the query is:

```text
Find all orders created today.
```

The router may need:

```text
Shard 1 → query
Shard 2 → query
Shard 3 → query
Shard 4 → query
```

Then combine results.

This is:

```text
Scatter-gather
```

---

# 41. Why Scatter-Gather Is Expensive

Potential problems:

```text
More network calls
Higher latency
More database work
Complex aggregation
Failure across multiple shards
```

Good shard keys reduce unnecessary scatter-gather operations.

---

# 42. Shard Rebalancing

Suppose:

```text
Shard 1 → 500 GB
Shard 2 → 500 GB
Shard 3 → 5 TB
```

The distribution is uneven.

You may need:

```text
Rebalancing
```

Moving data between shards can be operationally difficult.

---

# 43. Consistent Hashing

Consistent hashing can reduce data movement when nodes are added or removed.

Conceptually:

```text
Hash ring
---------------------
| A | B | C | D |
---------------------
```

Keys map onto the ring.

Adding a node doesn't necessarily require moving all keys.

---

# 44. Virtual Nodes

Systems often use multiple virtual positions per physical node.

This can improve distribution.

Conceptually:

```text
Physical Node A
→ vNode 1
→ vNode 2
→ vNode 3

Physical Node B
→ vNode 4
→ vNode 5
→ vNode 6
```

This helps balance the hash space.

---

# 45. Hot Shard

Suppose:

```text
Shard 1 → 80K RPS
Shard 2 → 10K RPS
Shard 3 → 10K RPS
```

The cluster's total capacity may look sufficient:

```text
100K RPS
```

but:

```text
Shard 1
→ overloaded
```

This is a hot-shard problem.

---

# 46. Causes of Hot Shards

Common causes:

```text
Poor shard key
Skewed data
Popular tenant
Popular user
Sequential writes
Time-based concentration
```

---

# 47. Hot Key vs Hot Shard

### Hot key

One particular key gets huge traffic.

```text
product:101
```

### Hot shard

A whole shard gets too much traffic.

```text
Shard 1
```

A hot key can cause a hot shard if many hot keys map to the same shard.

---

# 48. Tenant-Based Sharding

For multi-tenant systems:

```text
tenant_id
```

can be a natural shard key.

Example:

```text
Tenant A → Shard 1
Tenant B → Shard 2
Tenant C → Shard 3
```

But a very large tenant can create:

```text
Noisy neighbor
```

problems.

---

# 49. Noisy Neighbor

Suppose:

```text
Tenant A → 90% workload
Tenant B → 5%
Tenant C → 5%
```

If all are on one shard:

```text
Tenant A
```

can affect everyone else.

Potential solutions:

```text
Dedicated shard
Tenant-aware routing
Rate limits
Capacity isolation
```

---

# 50. Geographic Sharding

Data can be distributed by geography:

```text
India → Region A
Europe → Region B
US → Region C
```

Benefits:

```text
Lower latency
Data residency
Regional isolation
```

Challenges:

```text
Cross-region queries
Data movement
Consistency
Failover
```

---

# 51. Sharding and Transactions

Suppose:

```text
Order → Shard 1
Payment → Shard 2
```

A transaction involving both is difficult.

Avoid cross-shard transactions where possible.

Use:

```text
Saga
Events
Outbox
Compensation
```

for distributed workflows.

---

# 52. Sharding and Joins

Traditional join:

```sql
SELECT ...
FROM orders
JOIN users
ON orders.user_id = users.id;
```

If:

```text
orders → Shard 1
users  → Shard 2
```

the join becomes more complicated.

This is why sharding should consider relationships and query patterns.

---

# 53. Co-Location

A useful technique is storing related data on the same shard.

Example:

```text
user_id = 101

User
Orders
Cart
Preferences
```

could be routed to:

```text
Shard 2
```

Then many user-specific queries remain local.

---

# 54. Sharding by User ID

A common strategy:

```text
hash(user_id)
```

Then:

```text
User
 ↓
Shard
```

This works well when most queries are:

```text
By user
```

---

# 55. Sharding by Tenant

For SaaS:

```text
tenant_id
```

can provide strong data ownership boundaries.

But large tenants may need special treatment.

---

# 56. Sharding by Time

Example:

```text
2025 → Shard A
2026 → Shard B
2027 → Shard C
```

Useful for:

```text
Logs
Events
Historical data
```

But current traffic can concentrate on the latest shard.

---

# 57. Compound Shard Key

Sometimes one field isn't enough.

Example:

```text
tenant_id + user_id
```

This can improve distribution while preserving locality.

The exact strategy depends on the database.

---

# 58. Replication + Sharding

Large systems often combine them.

Example:

```text
              Router
            /        \
         Shard 1    Shard 2
          /   \      /   \
         P     R    P     R
```

Where:

```text
P = Primary
R = Replica
```

Each shard can have its own replicas.

---

# 59. Why Combine Them?

Sharding provides:

```text
Horizontal data/write scaling
```

Replication provides:

```text
Read scaling
Redundancy
Failover
```

Together:

```text
Shard
+
Replica
```

can support very large workloads.

---

# 60. Example Architecture

```text
                   Application
                       |
                  Shard Router
                  /     |     \
                 ↓      ↓      ↓
              Shard1 Shard2 Shard3
                |       |       |
             P + R    P + R    P + R
```

This is a more advanced architecture.

Don't use it unless requirements justify it.

---

# 61. Read Routing

With replication:

```text
Write
 ↓
Shard Primary

Read
 ↓
Shard Replica
```

But if:

```text
Read-after-write
```

is required:

```text
Read Primary
```

may be necessary.

---

# 62. Failure Scenario

Suppose:

```text
Shard 2 Primary
```

fails.

If a replica exists:

```text
Shard 2 Replica
       ↓
New Primary
```

Traffic can continue after failover, assuming the failover mechanism is correctly designed.

---

# 63. Failure Scenario Without Replication

If:

```text
Shard 2
```

has no replica and fails:

```text
All data on Shard 2
```

may become unavailable until recovery.

This is why sharding and availability are separate concerns.

---

# 64. Partitioning and Archival

Suppose orders are partitioned monthly:

```text
2025-01
2025-02
...
2026-08
```

Old partitions can potentially be:

```text
Archived
Compressed
Moved
Dropped
```

according to retention policy.

This makes lifecycle management easier.

---

# 65. Time-Based Data Lifecycle

A useful architecture:

```text
Hot
 ↓
Recent data

Warm
 ↓
Older data

Cold
 ↓
Archive/object storage
```

Not every historical record needs to remain in the hottest database tier.

---

# 66. Replication Lag Monitoring

Monitor:

```text
Replica lag
```

because high lag can cause:

```text
Stale reads
Failover risk
Capacity problems
```

---

# 67. Replication Metrics

Useful metrics:

```text
Replication lag
Write throughput
Replay/apply rate
Replica CPU
Replica IO
Network throughput
Replication errors
```

---

# 68. Sharding Metrics

Monitor:

```text
Requests per shard
Storage per shard
CPU per shard
Hot partitions
Latency per shard
Error rate
Rebalancing activity
```

---

# 69. Partitioning Metrics

Monitor:

```text
Partition size
Rows per partition
Query pruning
Hot partitions
Storage growth
Partition maintenance time
```

---

# 70. Replication vs Partitioning vs Sharding

| Technique | Main Purpose |
|---|---|
| Replication | Copies data for read scaling/availability |
| Partitioning | Splits data into logical pieces |
| Sharding | Distributes data across database nodes |
| Caching | Avoids repeated database work |

These techniques can be combined.

---

# 71. Simple Example

Suppose:

```text
100M users
```

and:

```text
90% reads
10% writes
```

Start:

```text
One MySQL
```

Then:

```text
Redis
```

Then:

```text
Primary
+ Read Replicas
```

If data/write scale exceeds one primary:

```text
Shard by user_id
```

Each shard:

```text
Primary
+ Replicas
```

---

# 72. E-commerce Example

Suppose:

```text
Orders = 2 billion
```

Main query:

```text
Get orders for user
```

A possible shard key:

```text
user_id
```

Then:

```text
User 101
 ↓
Shard 3
```

and:

```text
All user orders
 ↓
Shard 3
```

This gives good locality.

---

# 73. E-commerce Search Query

But now query:

```text
Find all orders created today.
```

would likely require:

```text
All shards
```

This is expensive.

Possible solution:

```text
Dedicated reporting/search model
```

rather than forcing the transactional shard layout to serve every query.

---

# 74. Database Router

A shard-aware application may have:

```text
Application
    ↓
Shard Router
    ↓
Determine shard
    ↓
Database
```

Routing can be handled by:

```text
Application
Database middleware
Proxy
Distributed database
```

depending on the architecture.

---

# 75. Shard Mapping

The system needs a mapping:

```text
Shard Key
   ↓
Shard
```

Example:

```text
hash(user_id)
      ↓
Shard 7
```

For dynamic systems, metadata may track:

```text
Shard ranges
Node locations
Rebalancing state
```

---

# 76. Data Migration Between Shards

Moving a user's data from:

```text
Shard 1
```

to:

```text
Shard 2
```

requires careful handling.

Potential process:

```text
Copy data
 ↓
Verify
 ↓
Dual-read/transition
 ↓
Switch routing
 ↓
Stop writes to old location
 ↓
Finalize
```

Exact migration strategy varies.

---

# 77. Online Migration

Large systems should avoid:

```text
Stop everything
 ↓
Move data
 ↓
Restart
```

Instead use:

```text
Incremental migration
+
Continuous synchronization
+
Controlled cutover
```

This reduces downtime.

---

# 78. Replication During Migration

Replication/CDC can help keep:

```text
Old location
New location
```

synchronized during a migration.

But it introduces:

```text
Lag
Failure handling
Duplicate events
Cutover complexity
```

---

# 79. Shard Key Changes

Changing the shard key is difficult.

Why?

Because the existing data is already distributed according to:

```text
Old key
```

Changing it may require:

```text
Large-scale data movement
```

Therefore:

> Choose the shard key carefully before committing to a large-scale sharded architecture.

---

# 80. Choosing Between Range and Hash

### Range

Good when queries are:

```text
By range
```

Example:

```text
created_at between X and Y
```

### Hash

Good when you want:

```text
Even distribution
```

Example:

```text
user_id lookup
```

---

# 81. Interview Question

### What is replication?

Answer:

> "Replication means maintaining copies of data on multiple database nodes. It can provide read scaling, redundancy and failover, but asynchronous replication can introduce replica lag."

---

# 82. Interview Question

### Does replication increase write capacity?

Answer:

> "Not necessarily. Traditional primary-replica replication primarily scales reads. The primary may still handle the write workload."

---

# 83. Interview Question

### What is sharding?

Answer:

> "Sharding distributes different portions of a dataset across multiple database nodes. It can increase storage and throughput capacity but introduces routing, cross-shard query and consistency complexity."

---

# 84. Interview Question

### What is a shard key?

Answer:

> "It's the field or combination of fields used to determine which shard stores a record. A good shard key should distribute both data and traffic well while supporting important query patterns."

---

# 85. Interview Question

### What is a hot shard?

Answer:

> "A hot shard receives disproportionately high traffic or data compared with other shards, creating an uneven bottleneck."

---

# 86. Interview Question

### What is partitioning?

Answer:

> "Partitioning divides a large logical table into smaller pieces based on a partitioning strategy such as range, hash or list. It can improve manageability and sometimes query performance."

---

# 87. Interview Question

### Partitioning vs sharding?

Answer:

> "Partitioning divides data logically, potentially within one database system. Sharding distributes those data partitions across separate database nodes to scale capacity horizontally."

---

# 88. Interview Question

### What is replication lag?

Answer:

> "It's the delay between a change being committed on the primary and that change being applied or visible on a replica."

---

# 89. Interview Question

### How do you handle a read-after-write problem?

Answer:

> "For operations that require immediate visibility, I'd route the read to the primary or use a consistency-aware routing strategy rather than assuming replicas are always up to date."

---

# 90. Interview Question

### When would you shard?

Answer:

> "Only after simpler scaling techniques such as query optimization, indexes, caching, vertical scaling, partitioning and read replicas are no longer sufficient for the required workload."

---

# 91. Interview Question

### What is scatter-gather?

Answer:

> "It's when a query must be sent to multiple shards and the results are gathered and combined. It can increase latency and resource usage."

---

# 92. Interview Question

### Why is shard-key selection important?

Answer:

> "Because the shard key determines data and traffic distribution. A poor key can create hot shards or force important queries to contact every shard."

---

# 93. Interview Question

### How do you handle cross-shard transactions?

Answer:

> "I'd try to avoid them by co-locating related data or redesigning the workflow. If a distributed transaction is unavoidable, I'd evaluate patterns such as Saga, events and compensating actions."

---

# 94. Practical Scenario

### Database reads are too high.

Architecture:

```text
Primary
```

First consider:

```text
Read replicas
```

if reads can tolerate the consistency behavior.

---

# 95. Practical Scenario

### Database writes are too high.

Read replicas won't solve the primary write bottleneck.

Investigate:

```text
Query optimization
Batching
Partitioning
Sharding
Write distribution
```

---

# 96. Practical Scenario

### One shard is overloaded.

Investigate:

```text
Shard key
Traffic distribution
Hot users
Hot tenants
Data skew
```

Then consider:

```text
Rebalancing
Better key
Dedicated shard
Key splitting
Caching
```

depending on the workload.

---

# 97. Practical Scenario

### Replica is 30 seconds behind.

Potential consequences:

```text
Stale reads
Bad user experience
Failover risk
```

Investigate:

```text
Replica CPU
IO
Network
Replication throughput
Long-running queries
```

---

# 98. Practical Scenario

### New database node is added.

If using a distributed/sharded system:

```text
Existing data
```

may need redistribution.

A good system should support:

```text
Controlled rebalancing
```

without unacceptable downtime.

---

# 99. Practical Scenario

### Recent orders are extremely popular.

Time-based partitioning:

```text
2026-08
```

may become hot.

Potential solutions:

```text
Sub-partition
Hash within time range
Spread writes
Use a different partition key
```

---

# 100. Final Architecture

A large-scale database layer may look like:

```text
                    Application
                        |
                   DB Router
                 /     |     \
                ↓      ↓      ↓
             Shard 1 Shard 2 Shard 3
                |       |       |
              P + R   P + R   P + R
```

And above it:

```text
Application
   ↓
Redis
   ↓
DB Router
   ↓
Sharded + Replicated DB
```

This is advanced architecture and should be introduced only when the scale requires it.

---

# 101. Final Mental Model

Remember:

```text
Replication
    ↓
Copies data
    ↓
Read scaling / redundancy

Partitioning
    ↓
Splits data logically
    ↓
Manageability / pruning / lifecycle

Sharding
    ↓
Distributes partitions across nodes
    ↓
Horizontal capacity
```

---

# 102. Scaling Progression

A practical progression:

```text
Single DB
   ↓
Optimize queries
   ↓
Indexes
   ↓
Connection pool tuning
   ↓
Redis
   ↓
Vertical scaling
   ↓
Read replicas
   ↓
Partitioning
   ↓
Sharding
```

Not every system needs every step.

---

# 103. One-Minute Interview Answer

### "How would you scale a database for a large e-commerce system?"

> "I'd first optimize the schema, indexes and queries and add caching for read-heavy workloads. If reads become the bottleneck, I'd add replicas while accounting for replication lag and read-after-write requirements. If the dataset or write throughput eventually exceeds a single database node's capacity, I'd consider partitioning and then sharding based on the main access patterns. I'd choose a shard key that distributes traffic evenly while keeping important queries localized, and I'd replicate each shard for availability and read scaling."

---

# 104. Key Takeaway

> **Replication gives you more copies, partitioning divides data logically, and sharding distributes those pieces across nodes. The hard part is not implementing the mechanism—it is choosing a design that keeps traffic balanced, queries efficient, consistency correct and operations manageable.**

**File 08 complete.**
