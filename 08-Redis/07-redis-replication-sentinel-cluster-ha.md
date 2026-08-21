# Redis — File 07: Replication, Sentinel, Cluster & High Availability

This file covers how Redis scales and survives failures in production.

Core topics:

```text
Primary / Replica
Replication
Asynchronous replication
Replication lag
Read scaling
Sentinel
Automatic failover
Quorum
Redis Cluster
Hash slots
Hash tags
MOVED / ASK
Resharding
Cluster failover
Multi-key limitations
High availability
Disaster recovery
Spring Boot connection behavior
Production architecture
Interview scenarios
```

---

# 1. Why High Availability Matters

A single Redis instance creates a single point of failure:

```text
Application
    ↓
Redis
    ↓
   FAIL
```

If Redis is unavailable:

```text
Cache unavailable
Locks unavailable
Rate limiting unavailable
Streams unavailable
```

depending on how the application uses Redis.

Production systems therefore often need:

```text
Replication
+
Failover
+
Monitoring
```

---

# 2. Primary / Replica Architecture

Basic Redis replication:

```text
          Primary
         /       \
        ↓         ↓
    Replica A  Replica B
```

The primary handles writes.

Replicas receive copies of the primary's data.

---

# 3. Why Replicas?

Replicas can provide:

```text
High availability
Read scaling
Data redundancy
Backup/recovery support
```

They are not a replacement for a proper backup strategy.

---

# 4. Replication Direction

Typical flow:

```text
Application
    ↓
Primary
    ↓
Replicas
```

The application normally writes to the primary.

Replicas receive replication updates.

---

# 5. Asynchronous Replication

Redis replication is generally asynchronous.

Conceptually:

```text
WRITE
 ↓
Primary acknowledges
 ↓
Replica receives update later
```

Therefore there can be:

```text
Replication lag
```

between primary and replica.

---

# 6. Replication Lag

Suppose:

```text
Primary:
balance = 100

Replica:
balance = 90
```

The replica is temporarily behind.

This matters when:

```text
Application writes
 ↓
Immediately reads from replica
```

The application may see stale data.

---

# 7. Read Scaling

For read-heavy workloads:

```text
Primary
 ↓
Replicas
```

application reads can potentially be distributed across replicas.

Example:

```text
10,000 reads/sec
```

could be spread across:

```text
Replica A
Replica B
Replica C
```

while writes go to:

```text
Primary
```

---

# 8. Read Scaling Trade-Off

Replica reads can improve:

```text
Throughput
```

but introduce:

```text
Staleness
```

Therefore:

> Do not route strongly consistent reads to replicas without understanding replication lag.

---

# 9. Read-After-Write Problem

Application:

```text
SET user:101 status ACTIVE
```

goes to primary.

Then:

```text
GET user:101
```

goes to replica.

The replica may temporarily return:

```text
OLD status
```

This is a read-after-write consistency issue.

---

# 10. Possible Solution

For operations requiring immediate consistency:

```text
Write → Primary
Read → Primary
```

or use an architecture that tracks the required consistency boundary.

Do not blindly use replicas for all reads.

---

# 11. Replica Failure

Suppose:

```text
Primary
 ↓
Replica A
Replica B
```

Replica A fails.

Primary and Replica B can continue.

This provides redundancy.

But if the primary itself fails:

```text
Primary
 ↓
FAIL
```

something must promote a replica.

---

# 12. Sentinel

Redis Sentinel provides monitoring and automatic failover capabilities for Redis primary/replica deployments.

Conceptually:

```text
             Sentinel
          /      |      \
         ↓       ↓       ↓
     Primary   Replica  Replica
```

Sentinels monitor the Redis nodes.

---

# 13. What Sentinel Does

Sentinel can provide:

```text
Monitoring
Failure detection
Automatic failover
Configuration discovery
Notification
```

It does not turn a single Redis server into a sharded cluster.

---

# 14. Sentinel Architecture

Typically multiple Sentinels are deployed:

```text
Sentinel A
Sentinel B
Sentinel C
```

monitoring:

```text
Primary
Replicas
```

Multiple Sentinels reduce the risk of making a failover decision based on one monitoring process.

---

# 15. Sentinel Quorum

Sentinel configuration includes a quorum.

Conceptually:

```text
Several Sentinels
      ↓
Do enough agree primary is unavailable?
      ↓
Failover decision
```

The exact behavior also depends on Sentinel's majority and election rules.

---

# 16. Subjectively Down

A Sentinel may decide:

```text
Primary = subjectively down
```

meaning that Sentinel itself believes the primary is unavailable.

This is different from:

```text
Objectively down
```

where enough Sentinels agree.

---

# 17. Objective Down

When enough Sentinels agree that a primary is unavailable:

```text
SDOWN
+
ODOWN
```

the system can proceed toward failover according to Sentinel rules.

---

# 18. Sentinel Failover

Conceptually:

```text
Primary
   ↓
  FAIL
   ↓
Sentinels detect failure
   ↓
Select replica
   ↓
Promote replica
   ↓
New Primary
```

Applications need to discover the new primary.

---

# 19. Why Multiple Sentinels?

If only one Sentinel exists:

```text
Sentinel fails
```

then:

```text
No monitoring
No reliable failover coordination
```

Multiple Sentinels improve resilience.

---

# 20. Sentinel Does Not Shard

Important:

```text
Sentinel
→ High availability / failover

Cluster
→ Sharding + high availability
```

This is a common interview question.

---

# 21. Redis Cluster

Redis Cluster distributes data across multiple Redis nodes.

Conceptually:

```text
Application
     ↓
Redis Cluster
 ┌────┼────┐
 ↓    ↓    ↓
Node A Node B Node C
```

Each node owns part of the keyspace.

---

# 22. Why Cluster?

Redis Cluster provides:

```text
Horizontal scaling
Data sharding
High availability through replicas
```

It allows the dataset to exceed the practical memory capacity of one Redis primary.

---

# 23. Hash Slots

Redis Cluster divides the keyspace into:

```text
16,384 hash slots
```

Each key maps to one slot.

Conceptually:

```text
key
 ↓
CRC16
 ↓
slot 0..16383
 ↓
cluster node
```

---

# 24. Key to Slot

Conceptually:

```text
user:101
   ↓
hash
   ↓
slot 7421
   ↓
Node B
```

The exact slot number depends on the key.

---

# 25. Slot Ownership

Example:

```text
Node A
slots 0–5000

Node B
slots 5001–10000

Node C
slots 10001–16383
```

Actual cluster layouts can be much more granular.

---

# 26. Cluster Sharding

Instead of:

```text
One Redis
+
100 GB
```

you can distribute:

```text
Node A → 30 GB
Node B → 30 GB
Node C → 40 GB
```

This allows horizontal memory scaling.

---

# 27. Cluster Replicas

A cluster node can have replicas:

```text
Primary A
   ↓
Replica A1

Primary B
   ↓
Replica B1

Primary C
   ↓
Replica C1
```

If a primary fails, its replica can potentially be promoted.

---

# 28. Cluster vs Sentinel

| Feature | Sentinel | Cluster |
|---|---|---|
| Failover | Yes | Yes |
| Sharding | No | Yes |
| Hash slots | No | Yes |
| Horizontal data scaling | No | Yes |
| Primary/replica | Yes | Yes |
| Best for | HA without sharding | HA + sharding |

---

# 29. MOVED Response

Suppose the client sends:

```text
GET user:101
```

to the wrong cluster node.

Redis can respond with:

```text
MOVED
```

telling the client:

```text
This key belongs to another node.
```

A cluster-aware client can update its slot mapping and retry.

---

# 30. Why MOVED Happens

Example:

```text
Client
 ↓
Node A
 ↓
MOVED slot 7421 → Node B
```

The client learns:

```text
slot 7421
→ Node B
```

and routes future requests correctly.

---

# 31. ASK Response

`ASK` can appear during cluster slot migration.

Conceptually:

```text
Slot is being moved
 ↓
Temporary routing
 ↓
ASK
```

The client should send the request to the indicated node for that operation without permanently changing its slot map in the same way as `MOVED`.

---

# 32. MOVED vs ASK

Important interview distinction:

```text
MOVED
→ Permanent/current slot ownership changed

ASK
→ Temporary redirection during slot migration
```

---

# 33. Resharding

Redis Cluster allows hash slots to be moved between nodes.

Example:

```text
Before:

Node A → 7000 slots
Node B → 5000 slots

After:

Node A → 6000
Node B → 6000
```

This can balance data and traffic.

---

# 34. Why Reshard?

Possible reasons:

```text
Uneven memory usage
Uneven traffic
Adding nodes
Removing nodes
Capacity expansion
```

---

# 35. Adding a Node

Suppose:

```text
A
B
C
```

need more capacity.

Add:

```text
D
```

Then move some hash slots:

```text
A/B/C
 ↓
D
```

The cluster becomes more balanced.

---

# 36. Removing a Node

Before removing a cluster node:

```text
Move its slots
```

to other nodes.

Then:

```text
Remove node
```

Otherwise keys owned by that node would become unavailable.

---

# 37. Multi-Key Operations

Redis Cluster creates an important limitation:

```text
Different keys
```

may map to:

```text
Different slots
```

and therefore:

```text
Different nodes
```

A multi-key command may not work across arbitrary slots.

---

# 38. Example

Suppose:

```text
GET user:101
GET user:102
```

map to different nodes.

A command requiring both keys in one atomic operation can become problematic.

This is why key design matters.

---

# 39. Hash Tags

Redis Cluster supports hash tags.

Example:

```text
user:{101}:profile
user:{101}:orders
user:{101}:cart
```

The portion inside:

```text
{101}
```

is used for slot calculation.

Therefore these keys can map to the same slot.

---

# 40. Why Hash Tags?

They enable certain multi-key operations to work within one slot.

Example:

```text
user:{101}:balance
user:{101}:transactions
```

can be colocated.

---

# 41. Hash Tag Trade-Off

Hash tags are powerful, but overusing the same tag can create:

```text
Hot slot
```

Example:

```text
{global}
```

on millions of keys.

Everything maps to the same slot.

This defeats the purpose of sharding.

---

# 42. Cluster-Aware Client

Application should use a Redis Cluster-aware client.

It understands:

```text
Slot mapping
MOVED
ASK
Cluster topology
```

Modern Redis client libraries can handle these details.

---

# 43. Spring Boot and Cluster

Spring Data Redis can be configured to connect to a Redis Cluster.

Conceptually:

```text
Spring Boot
 ↓
Cluster-aware Redis client
 ↓
Redis Cluster
```

The client routes keys based on cluster topology.

Exact configuration depends on the Spring Boot/Spring Data Redis version and Redis client.

---

# 44. Sentinel Client

With Sentinel:

```text
Spring Boot
 ↓
Sentinel-aware client
 ↓
Sentinels
 ↓
Current primary
```

The client can discover the current primary after failover.

---

# 45. Why Client Awareness Matters

If the application always connects to:

```text
10.0.0.1:6379
```

and that node fails over:

```text
Primary changes
```

the application needs a way to discover:

```text
New primary
```

Sentinel/Cluster-aware clients solve this at the topology level.

---

# 46. Connection Pooling

Spring Data Redis clients can use connection pooling/configuration appropriate to the chosen Redis client.

Consider:

```text
Pool size
Timeout
Connection limits
Network latency
Command workload
```

Don't blindly make the pool enormous.

---

# 47. Redis Timeout

A Redis timeout should be bounded.

Bad:

```text
Wait forever
```

because Redis may be unavailable and application threads can pile up.

Use:

```text
Reasonable connection timeout
Command timeout
Retry policy
```

based on latency requirements.

---

# 48. Retry Storm

Suppose Redis fails.

1,000 application threads retry immediately.

Result:

```text
Redis recovers
 ↓
1,000 simultaneous requests
```

This can cause another failure.

Use:

```text
Backoff
+
Jitter
+
Circuit breaking
```

when appropriate.

---

# 49. Failover Window

During failover:

```text
Primary fails
 ↓
Sentinels/Cluster detect
 ↓
Replica promoted
 ↓
Clients discover new primary
```

There can be a short period where:

```text
Writes fail
Reads fail
Connections reconnect
```

Application design should tolerate transient failures.

---

# 50. Replication Lag During Failover

Because replication is generally asynchronous:

```text
Primary
 ↓
Replica
```

may not be perfectly synchronized at the instant the primary fails.

Therefore some recently acknowledged writes may not exist on the promoted replica depending on the exact failure timing and durability/replication state.

This is an important trade-off.

---

# 51. Durability vs Availability

There is a fundamental trade-off:

```text
Fast acknowledgement
+
asynchronous replication
```

can improve:

```text
Latency
Availability
```

but may increase:

```text
Potential data loss during catastrophic failure
```

The correct architecture depends on business requirements.

---

# 52. WAIT

Redis provides commands such as:

```text
WAIT
```

that can make a client wait for replication acknowledgements from replicas.

This can improve confidence that writes have reached replicas.

But:

> `WAIT` does not turn Redis replication into a fully synchronous distributed transaction or guarantee disk durability.

---

# 53. Important Interview Answer

Question:

> "Does Redis replication guarantee zero data loss?"

Answer:

> "No. Redis replication is generally asynchronous, so there can be replication lag. During a primary failure, recently acknowledged writes may potentially not have reached the promoted replica. Durability and consistency requirements should determine the architecture."

---

# 54. Persistence Still Matters

Replication:

```text
Primary
 ↓
Replica
```

is not the same as persistence:

```text
Memory
 ↓
RDB/AOF
 ↓
Disk
```

A production architecture may use:

```text
Replication
+
Persistence
+
Backups
```

---

# 55. RDB

RDB creates snapshots of Redis data.

Conceptually:

```text
Memory
 ↓
Snapshot
 ↓
RDB file
```

Useful for:

```text
Backups
Faster restart in some scenarios
Point-in-time-ish snapshots
```

---

# 56. AOF

AOF records write operations in an append-only log.

Conceptually:

```text
SET
INCR
DEL
...
 ↓
AOF
```

It can provide stronger durability characteristics depending on configuration.

---

# 57. RDB vs AOF

```text
RDB
→ Snapshot-oriented

AOF
→ Operation-log-oriented
```

You can also use both depending on requirements.

---

# 58. Disaster Recovery

High availability is not the same as disaster recovery.

HA handles:

```text
Node failure
```

DR handles larger failures:

```text
Region failure
Data-center failure
Corruption
Accidental deletion
Major outage
```

---

# 59. Backup Strategy

Consider:

```text
RDB backups
AOF where appropriate
Off-host storage
Off-region copies
Backup verification
Restore testing
Retention
```

A backup that has never been restored is not a proven backup.

---

# 60. Redis Cluster Is Not a Backup

Cluster provides:

```text
Sharding
Availability
Replication
```

It does not protect you from every scenario such as:

```text
Application deletes all keys
Bad deployment
Logical corruption
Bad data replicated everywhere
```

Backups are still important.

---

# 61. Failover vs Backup

```text
Failover
→ Keep service running

Backup
→ Recover data
```

You need to understand both.

---

# 62. Sentinel Architecture Example

```text
             Sentinel A
                |
       +--------+--------+
       |                 |
   Primary           Replica
       |
   Replica

             Sentinel B
             Sentinel C
```

If primary fails:

```text
Sentinels coordinate
 ↓
Replica promoted
```

---

# 63. Cluster Architecture Example

```text
              Redis Cluster
        +---------+---------+
        |         |         |
     Primary A Primary B Primary C
        |         |         |
     Replica A Replica B Replica C
```

Data is sharded:

```text
Slots → A/B/C
```

---

# 64. Sentinel vs Cluster Architecture

Use Sentinel when:

```text
Dataset fits on one primary
Need HA
Need automatic failover
Don't need sharding
```

Use Cluster when:

```text
Dataset needs horizontal sharding
Need more write/memory capacity
Need HA across multiple shard primaries
```

---

# 65. Cluster Limitation

Cluster complicates:

```text
Multi-key operations
Transactions
Lua scripts
Key design
Distributed coordination
```

because related keys may live on different nodes.

Use hash tags when appropriate.

---

# 66. Cluster and Lua

A Lua script involving multiple keys generally needs those keys to belong to the same hash slot in Redis Cluster.

Therefore:

```text
Keys must be designed carefully
```

for atomic multi-key scripts.

---

# 67. Cluster and Transactions

A transaction involving keys across different hash slots cannot be treated like a single-node Redis transaction.

Design keys so related transactional data is colocated when cluster semantics require it.

---

# 68. Cluster and Distributed Locks

A simple:

```text
SET lock NX EX
```

against a single cluster key is straightforward.

But advanced distributed locking semantics require understanding:

```text
Failover
Replication
Quorum
Stale owners
Fencing
```

Do not assume cluster automatically makes locking safe.

---

# 69. Hot Slot

Even with a cluster:

```text
One hot key
 ↓
One slot
 ↓
One primary
```

can become a bottleneck.

Cluster does not automatically solve hot-key problems.

---

# 70. Hot Slot Mitigation

Potential approaches:

```text
Key spreading
Application-local caching
Request coalescing
Read replicas where appropriate
Workload redesign
```

Be careful with key spreading because it can complicate consistency and aggregation.

---

# 71. Cluster Capacity

Adding nodes increases:

```text
Memory capacity
Potential throughput
```

but not infinitely.

Bottlenecks can move to:

```text
Network
CPU
Hot slots
Client
Downstream database
```

---

# 72. Monitoring Redis Cluster

Track:

```text
Node health
Memory
CPU
Network
Replication lag
Slot distribution
Hot keys
Hot slots
Failovers
Connection count
Command latency
```

---

# 73. Monitoring Sentinel

Track:

```text
Sentinel health
Primary status
Replica status
Failovers
Quorum
Connection errors
Promotion events
```

---

# 74. Failover Testing

Don't wait for production to discover failover problems.

Test:

```text
Primary failure
Replica promotion
Client reconnect
Write recovery
Read behavior
Application retry
Connection pool recovery
```

This is part of production readiness.

---

# 75. Chaos Testing

A controlled test:

```text
Kill primary
 ↓
Observe failover
 ↓
Measure outage
 ↓
Check data
 ↓
Check application recovery
```

Important metrics:

```text
Recovery time
Error rate
Lost writes
Client reconnection time
```

---

# 76. Recovery Time Objective

RTO:

```text
How quickly must service recover?
```

Example:

```text
RTO = 30 seconds
```

Architecture must meet that requirement.

---

# 77. Recovery Point Objective

RPO:

```text
How much data loss can be tolerated?
```

Example:

```text
RPO = 0
```

is far stricter than:

```text
RPO = 5 minutes
```

Redis replication/persistence design should match RPO.

---

# 78. HA Design Question

Ask:

```text
What happens if primary dies?
What happens if replica dies?
What happens if Redis cluster loses a node?
What happens if the region dies?
What data can be lost?
How quickly must we recover?
```

These questions turn "Redis HA" into an actual architecture.

---

# 79. Interview Question

### Sentinel vs Redis Cluster?

Answer:

> "Sentinel provides monitoring, primary discovery and automatic failover for primary-replica deployments, but it doesn't shard the dataset. Redis Cluster provides sharding across 16,384 hash slots and can also provide high availability through replicas and failover."

---

# 80. Interview Question

### What is a Redis hash slot?

Answer:

> "Redis Cluster divides the keyspace into 16,384 hash slots. A key is mapped to one slot, and the cluster assigns slots to primary nodes. This is how Redis distributes data across the cluster."

---

# 81. Interview Question

### MOVED vs ASK?

Answer:

> "`MOVED` indicates that the slot belongs to another node and the client should update its topology mapping. `ASK` is a temporary redirection used during slot migration."

---

# 82. Interview Question

### Why use hash tags?

Answer:

> "Hash tags let related keys use the same hash-tag portion, such as `{101}`, so they map to the same Redis Cluster slot. This enables certain multi-key operations to work together. I would use them carefully because excessive concentration can create hot slots."

---

# 83. Interview Question

### Why can replicas return stale data?

Answer:

> "Redis replication is generally asynchronous, so a replica can temporarily lag behind the primary. Reads routed to replicas can therefore observe older state."

---

# 84. Interview Question

### Does Sentinel shard data?

Answer:

> "No. Sentinel is primarily for monitoring, discovery and failover. Redis Cluster is the mechanism for sharding data across multiple primary nodes."

---

# 85. Interview Question

### Is Redis Cluster a backup?

Answer:

> "No. Cluster provides sharding and availability, but it doesn't replace backups. Logical mistakes or corrupted data can be replicated across the cluster, so a separate backup and restore strategy is still necessary."

---

# 86. Interview Question

### Can Redis lose acknowledged writes during failover?

Answer:

> "Potentially yes, because replication is asynchronous. If the primary fails before a recent write reaches the replica that gets promoted, that write may be lost depending on the exact failure and persistence state."

---

# 87. Interview Question

### How would you design Redis for high availability?

Answer:

> "I'd start with the business RTO and RPO. For a dataset that fits on one primary, I'd consider primary-replica with multiple Sentinels for automatic failover. If the dataset or write workload needs horizontal scaling, I'd use Redis Cluster with replicas. I'd also configure persistence and backups appropriate to the recovery requirements and test failover regularly."

---

# 88. Interview Scenario

### Primary Redis fails.

Expected flow:

```text
Primary
 ↓
Failure
 ↓
Sentinels detect
 ↓
Replica promoted
 ↓
Clients discover new primary
 ↓
Application reconnects
```

Then verify:

```text
Writes
Reads
Replication
Error rate
```

---

# 89. Interview Scenario

### Application still writes to the old primary after failover.

Investigate:

```text
Client topology awareness
Sentinel configuration
Connection pooling
DNS/static connection configuration
Retry behavior
```

The application should discover the current primary rather than assuming a permanent endpoint.

---

# 90. Interview Scenario

### One cluster node has much more memory than others.

Possible causes:

```text
Uneven slot distribution
Hot keys
Large keys
Poor hash-tag strategy
Recent resharding
```

Investigate:

```text
Slot ownership
Key sizes
Traffic
```

---

# 91. Interview Scenario

### Multi-key transaction fails in Redis Cluster.

Likely:

```text
Keys map to different hash slots
```

Solution:

```text
Use a hash tag
```

when logically appropriate:

```text
user:{101}:profile
user:{101}:cart
```

---

# 92. Interview Scenario

### Redis cluster has enough total capacity, but one node is overloaded.

Possible:

```text
Hot slot
Hot key
Uneven workload
```

Adding more nodes may not immediately fix a single hot key.

---

# 93. Interview Scenario

### Replica reads return old data.

Expected:

```text
Replication lag
```

For consistency-sensitive reads:

```text
Read from primary
```

or use an explicit consistency strategy.

---

# 94. Interview Scenario

### Redis data disappears after catastrophic failure.

Investigate:

```text
Persistence
Backups
Replication
Restore process
RPO
```

Do not assume:

```text
Replica = backup
```

---

# 95. Production Architecture

A common HA architecture:

```text
                Application
                     |
              Cluster-aware
                  client
                     |
        +------------+------------+
        |            |            |
     Primary A    Primary B    Primary C
        |            |            |
     Replica A    Replica B    Replica C
```

For sharded high availability:

```text
Redis Cluster
+
replicas
+
persistence
+
backups
+
monitoring
```

---

# 96. Simpler HA Architecture

For a smaller dataset:

```text
Application
     |
Sentinel-aware client
     |
   Primary
   /     \
Replica A Replica B

Sentinel A
Sentinel B
Sentinel C
```

This avoids sharding complexity when sharding isn't needed.

---

# 97. Redis as a Cache

If Redis is only a cache:

```text
Primary failure
 ↓
Replica promotion
```

is still useful, but the application's fallback may be:

```text
Database
```

The key concern becomes:

```text
Protect database during Redis failure
```

---

# 98. Redis as Primary State

If Redis holds:

```text
Critical durable business state
```

then:

```text
Replication
Persistence
Backup
Recovery testing
RPO/RTO
```

become much more important.

The architecture should not casually treat Redis as disposable cache memory.

---

# 99. HA Checklist

```text
□ Primary
□ Replicas
□ Replication lag
□ Sentinel
□ Multiple Sentinels
□ Quorum
□ Automatic failover
□ Redis Cluster
□ 16,384 hash slots
□ Hash tags
□ MOVED
□ ASK
□ Resharding
□ Cluster replicas
□ Multi-key limitations
□ Persistence
□ RDB
□ AOF
□ Backups
□ Disaster recovery
□ RTO
□ RPO
□ Monitoring
□ Failover testing
□ Client topology awareness
```

---

# 100. Final Mental Model

Think of Redis production architecture as four layers:

```text
1. Scaling
   ↓
Redis Cluster / sharding

2. Availability
   ↓
Replicas + Sentinel/Cluster failover

3. Durability
   ↓
RDB/AOF + backups

4. Recovery
   ↓
Monitoring + tested restore/failover
```

Each solves a different problem.

---

# 101. Final Interview Answer

If asked:

> "How would you design a highly available Redis architecture for a Spring Boot microservices application?"

Say:

> "First I'd determine whether we need sharding. If the dataset fits on one primary, I'd use primary-replica with multiple Sentinels for automatic failover. If we need horizontal memory or write scaling, I'd use Redis Cluster with replicas. I'd configure persistence and backups based on the RPO, use topology-aware Redis clients in Spring Boot, monitor replication lag and failovers, and regularly test failure and recovery scenarios."

---

# 102. What Comes Next

```text
File 08 → Redis Security, Persistence, Monitoring & Production Best Practices
```

Next we will consolidate the remaining production topics:

```text
Redis authentication
ACLs
TLS
Network security
RDB
AOF
Persistence trade-offs
Backup & restore
Memory monitoring
Latency monitoring
Slowlog
Keyspace notifications
Security mistakes
Production configuration
Observability
Capacity planning
Failure handling
Spring Boot production configuration
Interview scenarios
```

Key takeaway:

> **Redis high availability is not one feature. Replication provides redundancy, Sentinel provides failover coordination, Cluster provides sharding plus HA, persistence provides restart durability, and backups provide disaster recovery. A production design needs all the pieces that match its RPO, RTO, scale and consistency requirements.**
