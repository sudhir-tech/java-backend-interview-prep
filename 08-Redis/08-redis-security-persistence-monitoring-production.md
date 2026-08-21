# Redis — File 08: Security, Persistence, Monitoring & Production Best Practices

This file brings Redis production knowledge together.

Core topics:

```text
Authentication
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

---

# 1. Redis Security

Redis is extremely fast, but that also makes an exposed Redis server dangerous.

A production deployment should consider:

```text
Network isolation
Authentication
ACLs
TLS
Least privilege
Secret management
Monitoring
```

---

# 2. Never Expose Redis Publicly

Avoid:

```text
Internet
   ↓
Redis:6379
```

Prefer:

```text
Application private network
        ↓
      Redis
```

Use:

```text
VPC/private subnet
Firewall
Security groups
Network policies
```

depending on the infrastructure.

---

# 3. Authentication

Redis supports authentication mechanisms that can restrict access.

Applications should not rely on:

```text
No authentication
```

for production systems.

Credentials should be stored in:

```text
Environment variables
Secret manager
Vault
Cloud secret service
```

rather than committed to Git.

---

# 4. ACLs

Redis Access Control Lists allow more granular permissions.

Conceptually:

```text
Application user
    ↓
Allowed commands
    +
Allowed key patterns
```

For example, an application may be allowed to:

```text
GET
SET
DEL
```

but not administrative commands.

---

# 5. Least Privilege

Don't give every application:

```text
Full Redis administrative access
```

Instead:

```text
Service A
 ↓
Only commands it needs

Service B
 ↓
Different permissions
```

This reduces blast radius.

---

# 6. Dangerous Commands

Administrative commands can be dangerous if exposed to untrusted applications.

Examples to carefully restrict include:

```text
CONFIG
FLUSHALL
FLUSHDB
DEBUG
MODULE
```

The exact command restrictions should follow the Redis version and deployment requirements.

---

# 7. FLUSHALL

```text
FLUSHALL
```

removes all keys from all databases in the Redis instance.

This is extremely destructive.

Never run it casually in production.

---

# 8. FLUSHDB

```text
FLUSHDB
```

removes all keys from the selected logical database.

Again:

```text
Potentially destructive
```

Protect it with ACLs and operational controls.

---

# 9. TLS

Redis deployments can use TLS to protect traffic.

Without encryption:

```text
Application
   ↓
Redis
```

traffic may be visible to an attacker who can observe the network.

With TLS:

```text
Application
   ↓ encrypted connection
Redis
```

---

# 10. What TLS Protects

TLS can provide:

```text
Encryption in transit
Server authentication
Potential client authentication
```

depending on configuration.

---

# 11. TLS Does Not Solve Everything

TLS does not protect against:

```text
Compromised application
Stolen credentials
Bad ACLs
Malicious commands
Application bugs
Exposed Redis endpoint
```

Security needs multiple layers.

---

# 12. Secret Management

Bad:

```properties
spring.data.redis.password=mySecret123
```

committed into Git.

Better:

```text
Environment
 ↓
Secret manager
 ↓
Application
 ↓
Redis
```

Never commit production credentials.

---

# 13. Network Security

A strong deployment can use:

```text
Internet
   X
   |
Firewall
   |
Private subnet
   |
Redis
```

Only authorized application services can reach Redis.

---

# 14. Authentication + Network Security

Use defense in depth:

```text
Private network
+
Authentication
+
ACL
+
TLS
```

rather than relying on one security layer.

---

# 15. Redis Persistence

Redis stores data primarily in memory.

Persistence determines how Redis can recover data after restart.

Two major mechanisms:

```text
RDB
AOF
```

---

# 16. RDB

RDB is snapshot-based persistence.

Conceptually:

```text
Redis memory
     ↓
Snapshot
     ↓
RDB file
```

The snapshot represents the dataset at a particular point.

---

# 17. RDB Advantages

RDB can provide:

```text
Compact backup files
Efficient restore
Convenient snapshots
Lower ongoing write-log overhead
```

It can be useful for:

```text
Backups
Disaster recovery
Faster dataset transfer
```

---

# 18. RDB Disadvantage

If the last snapshot was:

```text
10 minutes ago
```

and Redis crashes:

```text
Potentially recent data
```

may not be represented in the latest snapshot.

Therefore RDB alone may not satisfy strict RPO requirements.

---

# 19. AOF

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

On restart, Redis can reconstruct state by replaying the log according to its persistence mechanism.

---

# 20. AOF Advantages

AOF can provide:

```text
More frequent durability
Detailed write history
Configurable fsync behavior
```

depending on configuration.

---

# 21. AOF Disadvantages

Compared with snapshots, AOF can involve:

```text
More disk activity
Larger log management
Rewrite operations
Additional operational complexity
```

---

# 22. AOF Fsync Policies

Conceptually, common durability choices include:

```text
Always
Every second
OS-controlled
```

The trade-off is:

```text
Durability
vs
Performance
```

A stricter fsync policy generally has more I/O overhead.

---

# 23. RDB vs AOF

| Feature | RDB | AOF |
|---|---|---|
| Model | Snapshot | Write log |
| File size | Often compact | Can be larger |
| Recovery | Snapshot load | Log replay |
| Recent write durability | Depends on snapshot interval | Depends on fsync policy |
| Backup use | Excellent | Also possible |
| Operational complexity | Lower | Higher |

---

# 24. Use Both?

Depending on requirements, Redis can use:

```text
RDB
+
AOF
```

This can provide:

```text
Snapshots
+
more detailed write persistence
```

But more persistence is not automatically better.

Choose based on:

```text
RPO
RTO
Performance
Storage
Recovery requirements
```

---

# 25. Persistence for a Cache

If Redis is only:

```text
Cache
```

you may decide:

```text
Persistence is less important
```

because:

```text
Database
=
source of truth
```

After restart:

```text
Cache can be rebuilt
```

---

# 26. Persistence for Primary State

If Redis contains:

```text
Critical application state
```

then persistence becomes much more important.

Consider:

```text
RDB
AOF
Replication
Backups
Restore testing
```

---

# 27. Persistence vs Replication

Remember:

```text
Persistence
→ Recover from storage

Replication
→ Maintain copies across Redis nodes
```

They solve different failure scenarios.

---

# 28. Backup Strategy

A production backup strategy should define:

```text
What is backed up?
How often?
Where?
How long retained?
Encrypted?
Cross-region?
How restored?
```

---

# 29. Backup Is Not Enough

You need:

```text
Backup
+
Restore testing
```

If:

```text
backup exists
```

but:

```text
restore fails
```

you don't really have a reliable recovery plan.

---

# 30. Disaster Recovery

For major failures:

```text
Region A
   ↓
Failure
   ↓
Region B
```

A DR strategy may use:

```text
Backups
Replication
Infrastructure automation
DNS/failover
Application redeployment
```

Redis architecture should match the required RTO/RPO.

---

# 31. RPO

Recovery Point Objective asks:

```text
How much data can we afford to lose?
```

Example:

```text
RPO = 1 minute
```

means losing more than approximately one minute of recent data may be unacceptable.

---

# 32. RTO

Recovery Time Objective asks:

```text
How quickly must service recover?
```

Example:

```text
RTO = 30 seconds
```

requires much faster recovery than:

```text
RTO = 2 hours
```

---

# 33. Monitoring Redis

Production Redis should be monitored continuously.

Important areas:

```text
Memory
CPU
Network
Latency
Connections
Commands
Evictions
Replication
Persistence
Errors
Streams
```

---

# 34. Memory Metrics

Monitor:

```text
Used memory
Max memory
RSS
Fragmentation
Evicted keys
Expired keys
Dataset size
```

This helps identify:

```text
Memory pressure
Growth
Fragmentation
Eviction problems
```

---

# 35. CPU Metrics

High CPU can indicate:

```text
High request rate
Expensive commands
Lua scripts
Large data structures
Serialization
Hot keys
Persistence work
```

Investigate command-level behavior before simply adding CPU.

---

# 36. Network Metrics

Monitor:

```text
Inbound traffic
Outbound traffic
Packets
Bandwidth
Connection count
```

Huge values can cause:

```text
Network saturation
Latency
Client timeouts
```

---

# 37. Latency

Monitor:

```text
Average latency
P95
P99
P99.9
```

Tail latency is important.

Example:

```text
Average = 1 ms
P99 = 100 ms
```

The system may still feel slow for a significant subset of requests.

---

# 38. Redis Slowlog

Redis can record commands whose execution time exceeds the configured threshold.

Useful for finding:

```text
Expensive commands
Unexpectedly large operations
Slow scripts
```

---

# 39. Slowlog Limitation

Slowlog focuses on command execution inside Redis.

It doesn't automatically explain:

```text
Network latency
Client queueing
Connection pool waits
Application serialization
```

Investigate the full request path.

---

# 40. Command Complexity

Understand complexity before using commands.

Examples:

```text
GET
→ O(1)

SET
→ O(1)

INCR
→ O(1)

HGET
→ O(1) average

SMEMBERS
→ O(N)

HGETALL
→ O(N)

LRANGE
→ Depends on requested range
```

Large `O(N)` operations can be dangerous at scale.

---

# 41. KEYS

Avoid:

```text
KEYS *
```

on large production instances.

It can block Redis while scanning the keyspace.

Prefer:

```text
SCAN
```

for incremental iteration.

---

# 42. Large Deletes

Deleting huge data structures can be expensive.

Where appropriate:

```text
UNLINK
```

can move deletion work away from the main command execution path.

It still consumes resources, so it is not a free operation.

---

# 43. Big Key Monitoring

Identify:

```text
Huge strings
Huge hashes
Huge lists
Huge sets
Huge sorted sets
```

Use tools/commands such as:

```text
MEMORY USAGE
```

and appropriate key-analysis tooling.

---

# 44. Hot Key Monitoring

A hot key can create:

```text
High command rate
Single-node bottleneck
Latency
CPU pressure
```

Even in a Redis Cluster.

---

# 45. Keyspace Notifications

Redis can publish notifications about key events.

Examples conceptually:

```text
Key expired
Key deleted
Key modified
```

This can be useful for:

```text
Lightweight event reactions
Expiration notifications
Local cache coordination
```

---

# 46. Keyspace Notification Limitation

Keyspace notifications are not a durable event system.

If a consumer is disconnected:

```text
Notification can be missed
```

Therefore:

```text
Keyspace notifications
≠
Reliable event stream
```

---

# 47. Monitoring Expiration

Track:

```text
Expired keys
Evicted keys
TTL distribution
Cache misses
```

If expiration spikes suddenly:

```text
Database load may increase
```

because many cache entries disappear at once.

---

# 48. Cache Hit Rate

Track:

```text
hits
misses
```

A common derived metric:

```text
hit rate =
hits / (hits + misses)
```

But also track:

```text
Latency
Database load
Memory
Staleness
```

---

# 49. Eviction Monitoring

Track:

```text
evicted_keys
```

A sudden increase can indicate:

```text
Memory pressure
Unexpected dataset growth
Too-small maxmemory
Changed workload
```

---

# 50. Replication Monitoring

Monitor:

```text
Replica health
Replication offset
Replication lag
Disconnected replicas
Failover events
```

Replica lag matters especially when replicas serve reads.

---

# 51. Persistence Monitoring

Monitor:

```text
RDB save status
AOF status
Rewrite activity
Disk usage
Disk latency
Persistence failures
```

A healthy Redis process with broken persistence is still a production problem if durability matters.

---

# 52. Stream Monitoring

For Redis Streams:

```text
Stream length
Consumer groups
Pending entries
Consumer lag
Retry count
Dead-letter events
```

A growing pending list can indicate:

```text
Consumer failure
Slow processing
Downstream outage
```

---

# 53. Connection Monitoring

Track:

```text
Connected clients
Rejected connections
Connection churn
Blocked clients
```

Too many connections can create:

```text
CPU overhead
Memory overhead
Network pressure
```

---

# 54. Connection Pooling

For Spring Boot applications:

```text
Application
 ↓
Redis connection pool
 ↓
Redis
```

Pool size should match:

```text
Concurrency
Command latency
Redis capacity
Application thread model
```

Too large:

```text
Connection explosion
```

Too small:

```text
Pool wait
Latency
```

---

# 55. Timeouts

Configure reasonable:

```text
Connect timeout
Command timeout
Read timeout
```

Exact settings depend on the Redis client.

Avoid indefinite waits.

---

# 56. Retry Policy

Retries should be:

```text
Bounded
Backoff-based
Jittered
Failure-aware
```

Avoid:

```text
Infinite immediate retries
```

---

# 57. Circuit Breaker

If Redis is unavailable:

```text
Application
 ↓
Redis errors
 ↓
Circuit opens
 ↓
Fallback behavior
```

This prevents every request from repeatedly waiting on a failing Redis connection.

---

# 58. Fallback

If Redis is a cache:

```text
Redis unavailable
 ↓
Database fallback
```

But protect the database:

```text
Rate limiting
Load shedding
Circuit breaker
Request prioritization
```

---

# 59. Cache Stampede Protection

When a hot key expires:

```text
Many requests
 ↓
Cache miss
 ↓
Database
```

Potential solutions:

```text
TTL jitter
Request coalescing
Locks
Refresh ahead
Stale-while-revalidate
```

---

# 60. Capacity Planning

Before production, estimate:

```text
Keys
Average value size
Peak QPS
Read/write ratio
Memory overhead
Replication factor
Growth rate
Retention
```

---

# 61. Memory Estimation

A simplistic estimate:

```text
Total memory
≈
number of keys
×
average key/value footprint
+
Redis overhead
+
fragmentation
+
operational headroom
```

Don't use raw payload size as the exact Redis memory requirement.

Redis objects and allocator overhead matter.

---

# 62. Headroom

Don't operate permanently at:

```text
99% memory
```

Leave capacity for:

```text
Traffic spikes
Replication
Persistence
Fragmentation
Temporary operations
Growth
```

The exact headroom depends on workload and operational policy.

---

# 63. Capacity Example

Suppose:

```text
Average object footprint ≈ 2 KB
Keys = 10 million
```

Raw payload approximation:

```text
~20 GB
```

But production memory must also account for:

```text
Key metadata
Redis object overhead
Allocator overhead
Fragmentation
Replication
Operational headroom
```

Therefore don't provision exactly 20 GB.

---

# 64. Memory Growth Investigation

If memory grows unexpectedly:

```text
1. Check key count
2. Check big keys
3. Check TTL coverage
4. Check unbounded collections
5. Check value sizes
6. Check recent deployment
7. Check fragmentation
8. Check replication/persistence effects
```

---

# 65. Production Key Design

Good:

```text
product:101
user:101:cart
order:101
```

Keep keys:

```text
Consistent
Predictable
Reasonably short
Namespace-aware
```

---

# 66. Avoid Unbounded Keys

Dangerous:

```text
events:all
```

with millions of entries forever.

Better:

```text
events:2026-08
```

or use:

```text
Streams
+
retention
```

depending on requirements.

---

# 67. Serialization Strategy

Consider:

```text
JSON
Binary
Hash fields
String values
```

Evaluate:

```text
Memory
CPU
Compatibility
Debuggability
Interoperability
```

---

# 68. Production Configuration

Common production considerations:

```text
maxmemory
eviction policy
persistence
authentication
ACL
TLS
timeouts
connection limits
monitoring
backup
```

Don't copy a configuration blindly from another workload.

---

# 69. Cache-Only Redis

If Redis is purely a cache:

```text
Database
   ↓
Source of truth

Redis
   ↓
Derived data
```

Potential configuration:

```text
maxmemory
+
eviction policy
+
TTL
```

and possibly limited persistence depending on recovery needs.

---

# 70. Stateful Redis

If Redis stores:

```text
Sessions
Queues
Streams
Counters
Business state
```

the durability and recovery requirements may be much stronger.

Define:

```text
RPO
RTO
Persistence
Backup
Failover
```

before production.

---

# 71. Security Checklist

```text
□ Private network
□ Authentication
□ ACLs
□ Least privilege
□ TLS where required
□ Secret manager
□ Restricted admin commands
□ No public Redis
□ Firewall
□ Monitoring
□ Audit access
```

---

# 72. Observability Checklist

```text
□ Memory
□ CPU
□ Network
□ Latency
□ P95/P99
□ Slowlog
□ Connections
□ Evictions
□ Expirations
□ Hit rate
□ Replication lag
□ Persistence
□ Stream lag
□ Pending entries
□ Errors
□ Failovers
```

---

# 73. Reliability Checklist

```text
□ Replication
□ Failover
□ Persistence
□ Backups
□ Restore testing
□ RPO
□ RTO
□ Retry strategy
□ Circuit breaker
□ Cache fallback
□ Stampede protection
□ Capacity planning
```

---

# 74. Common Security Mistake

Bad:

```text
Redis exposed to the internet
```

Better:

```text
Private network
+
authentication
+
ACL
+
TLS
```

---

# 75. Common Operations Mistake

Bad:

```text
KEYS *
```

Better:

```text
SCAN
```

---

# 76. Common Reliability Mistake

Bad:

```text
Redis failure
 ↓
Every request hits MySQL
```

This can cause:

```text
Database meltdown
```

Better:

```text
Circuit breaker
+
load shedding
+
controlled fallback
```

---

# 77. Common Persistence Mistake

Bad:

> "We have replicas, so we don't need backups."

Better:

> "Replicas improve availability, while backups provide recovery from logical corruption, accidental deletion and larger failures."

---

# 78. Common Monitoring Mistake

Bad:

```text
CPU = 30%
```

therefore:

```text
Redis is healthy
```

Not enough.

Also check:

```text
Latency
Memory
Evictions
Replication
Connections
Errors
```

---

# 79. Common Cache Mistake

Bad:

```text
99% hit rate
```

therefore:

```text
Cache is perfect
```

Maybe:

```text
P99 latency is high
```

or:

```text
Cached data is stale
```

or:

```text
Memory is almost full
```

Always monitor multiple dimensions.

---

# 80. Interview Question

### How would you secure Redis in production?

Answer:

> "I'd keep Redis on a private network, require authentication and appropriate ACLs, use TLS when required by the environment, restrict administrative commands, store credentials in a secret manager and monitor access. I would also avoid exposing Redis directly to the public internet."

---

# 81. Interview Question

### RDB vs AOF?

Answer:

> "RDB is snapshot-based persistence, while AOF records write operations. RDB is convenient for compact backups and snapshots, while AOF can provide more frequent durability depending on fsync settings. The choice depends on RPO, RTO, performance and recovery requirements."

---

# 82. Interview Question

### Do replicas replace backups?

Answer:

> "No. Replicas help availability and read scaling, but logical mistakes or accidental deletions can propagate to replicas. Backups provide a separate recovery mechanism."

---

# 83. Interview Question

### How do you monitor Redis?

Answer:

> "I'd monitor memory, CPU, network, command latency and P95/P99 latency, connections, evictions, expirations, cache hit rate, replication lag, persistence health and, for Streams, consumer lag and pending entries. I'd also inspect slow commands when latency increases."

---

# 84. Interview Question

### What would you check if Redis memory keeps increasing?

Answer:

> "I'd check key count and growth, big keys, unbounded collections, TTL coverage, value sizes, recent deployments, fragmentation and replication/persistence behavior. Then I'd determine whether the growth is expected or caused by an application issue."

---

# 85. Interview Question

### What happens if Redis goes down?

Answer:

> "It depends on the role Redis plays. If it is only a cache, I can potentially fall back to the database, but I would protect the database from a cache-miss storm using timeouts, circuit breaking, rate limiting and controlled fallback. If Redis holds critical state, I'd need an HA and recovery strategy rather than simply falling back."

---

# 86. Interview Question

### Why are P99 metrics important?

Answer:

> "Averages can hide slow requests. P99 tells me how the slowest one percent of requests behave, which is important for user experience and distributed-system tail latency."

---

# 87. Interview Scenario

### Redis latency suddenly increases.

Investigate:

```text
CPU
Memory pressure
Network
Slow commands
Large values
Big collections
Lua scripts
Hot keys
Persistence activity
Connection pool
```

Then correlate with:

```text
Application deployment
Traffic spike
Infrastructure change
```

---

# 88. Interview Scenario

### Redis is healthy but application latency increased.

Investigate:

```text
Connection pool
Serialization
Network
Thread pool
Client retries
Application CPU
DB fallback
```

Redis server latency alone does not explain total application latency.

---

# 89. Interview Scenario

### Redis hit rate suddenly drops.

Investigate:

```text
TTL changes
Evictions
Key format
Deployment
Serialization
Cache invalidation
Traffic pattern
Redis restart
```

---

# 90. Interview Scenario

### Evictions suddenly increase.

Investigate:

```text
maxmemory
Dataset growth
Value sizes
Traffic
TTL coverage
Recent features
Serialization
```

Then decide:

```text
Increase capacity
Fix data growth
Change TTL
Change eviction policy
```

based on the root cause.

---

# 91. Interview Scenario

### Redis needs to survive an entire availability-zone failure.

Consider:

```text
Replicas across failure domains
Automatic failover
Topology-aware deployment
Cross-zone networking
Persistence
Backups
RTO/RPO testing
```

---

# 92. Interview Scenario

### Redis region is completely unavailable.

This is a disaster-recovery problem.

Consider:

```text
Cross-region backup
Replica/DR architecture
Infrastructure automation
DNS/failover
Data recovery
Application recovery
```

Do not confuse this with ordinary node failover.

---

# 93. Production Readiness Checklist

Before going live:

```text
□ Security configured
□ Authentication configured
□ ACLs reviewed
□ Network isolated
□ TLS requirements reviewed
□ maxmemory configured
□ Eviction policy chosen
□ TTL strategy defined
□ Persistence strategy defined
□ Backup strategy defined
□ Restore tested
□ Monitoring configured
□ Alerts configured
□ Failover tested
□ RPO documented
□ RTO documented
□ Capacity estimated
□ Hot keys reviewed
□ Big keys reviewed
□ Timeouts configured
□ Retry strategy configured
□ Circuit breaker considered
□ Cache stampede strategy
□ Incident runbook
```

---

# 94. Redis Production Mental Model

Think in this order:

```text
1. Security
   ↓
Can only authorized services access Redis?

2. Correctness
   ↓
What data can be stale/lost?

3. Availability
   ↓
What happens when a node fails?

4. Durability
   ↓
How do we recover data?

5. Performance
   ↓
Can Redis handle peak load?

6. Observability
   ↓
How will we know something is wrong?

7. Recovery
   ↓
Can we actually restore/fail over?
```

---

# 95. Final Backend Interview Answer

If asked:

> "How would you make Redis production-ready?"

Say:

> "I'd start by isolating Redis on a private network and configuring authentication, ACLs and TLS where required. Then I'd define maxmemory, TTL and eviction policies based on the workload. For availability I'd use replicas with Sentinel or Redis Cluster depending on whether sharding is needed. For durability I'd configure RDB/AOF and separate backups based on the RPO/RTO. Finally I'd monitor memory, latency, evictions, replication, persistence and errors, configure bounded retries and circuit breaking, and regularly test failover and restore procedures."

---

# 96. Redis Complete Revision Map

At this point, the Redis preparation folder covers:

```text
01 → Redis Fundamentals
02 → Data Structures & Commands
03 → Expiration, Eviction & Memory
04 → Caching Patterns & Spring Boot
05 → Distributed Locks, Rate Limiting & Atomicity
06 → Pub/Sub, Streams & Messaging
07 → Replication, Sentinel, Cluster & HA
08 → Security, Persistence, Monitoring & Production
```

This gives you a strong interview-level Redis foundation for:

```text
Java Backend
Spring Boot
Microservices
System Design
Production troubleshooting
```

---

# 97. What Comes Next

Redis core preparation is now substantially covered.

The next useful step is:

```text
Redis Interview Revision / Scenarios
```

before moving to the next backend technology.

Recommended sequence:

```text
Redis
 ↓
Quick revision
 ↓
Interview questions
 ↓
Scenario-based questions
 ↓
Hibernate
 ↓
Docker
```

Key takeaway:

> **Production Redis is more than an in-memory cache. You need to understand security, persistence, replication, failover, memory, latency, observability, backups and recovery. The best interview answers connect each Redis feature to a concrete business requirement such as availability, consistency, performance or durability.**
