# System Design — File 13: Distributed Systems, CAP, Consistency & Consensus

When a system runs across multiple machines, new problems appear:

```text
Network failures
Partial failures
Replication lag
Concurrent updates
Data consistency
Node failures
Clock differences
```

This file covers the distributed-systems concepts that commonly appear in backend and system-design interviews.

---

# 1. What Is a Distributed System?

A distributed system consists of multiple independent machines that work together as one logical system.

Example:

```text
                    Client
                       |
                       v
                 Load Balancer
                  /    |    \
                 v     v     v
                S1    S2    S3
                 |     |     |
                 +-----+-----+
                       |
                    Database
```

The machines communicate over a network.

The key difficulty is:

> The network can fail even when individual machines are healthy.

---

# 2. Partial Failure

In a single process:

```text
Process crashes
→ System knows it failed
```

In a distributed system:

```text
Service A
    |
    X
    |
Service B
```

A may not know whether B:

```text
Crashed
Is slow
Is disconnected
Processed the request
```

This uncertainty is a fundamental distributed-systems problem.

---

# 3. Network Failure

Possible failures:

```text
Packet loss
Connection reset
Timeout
Network partition
High latency
DNS failure
```

Never assume:

```text
"Internal network = reliable"
```

Design every network dependency with failure in mind.

---

# 4. CAP Theorem

CAP refers to:

```text
C = Consistency
A = Availability
P = Partition tolerance
```

During a network partition, a distributed data system cannot simultaneously guarantee both perfect consistency and availability.

In practice:

```text
Partition happens
       |
       +--> Prefer consistency
       |
       +--> Prefer availability
```

The exact trade-off depends on the system.

---

# 5. Consistency

A strongly consistent system attempts to make reads reflect the latest successful write according to its consistency model.

Example:

```text
Write:
balance = 100

Immediately read:
balance = 100
```

Strong consistency is useful for data where stale reads are unacceptable.

Examples may include:

```text
Critical financial state
Inventory constraints
Certain authorization decisions
```

---

# 6. Availability

Availability means the system continues responding to requests according to its availability guarantees.

During a failure:

```text
Request
  |
  v
Healthy replica
  |
  v
Response
```

A highly available system may continue serving requests even if some components fail.

---

# 7. Partition Tolerance

A partition occurs when nodes cannot reliably communicate.

```text
Node A  X  Node B
```

The system must continue operating under the assumption that such network failures can happen.

This is why distributed systems have to make a consistency/availability trade-off during partitions.

---

# 8. CAP Interview Answer

Question:

> What is CAP theorem?

Answer:

> "CAP says that when a distributed system experiences a network partition, it cannot guarantee both strong consistency and availability at the same time. Partition tolerance is required in a distributed environment, so the practical choice is about how the system behaves during the partition."

---

# 9. CAP Is Not "Pick Any Two"

A common interview mistake is:

```text
Choose any 2 of C, A and P
```

That's an oversimplification.

The important point is:

```text
When a partition occurs,
you must trade off consistency and availability.
```

Without a partition, a system can often provide both strong consistency and availability.

---

# 10. Eventual Consistency

With eventual consistency:

```text
Write
  |
  v
Primary
  |
  +--> Replica 1
  |
  +--> Replica 2
```

Replicas may temporarily contain older data.

Eventually:

```text
Replica 1 → latest
Replica 2 → latest
```

This is common in highly distributed systems.

---

# 11. Strong vs Eventual Consistency

| Strong Consistency | Eventual Consistency |
|---|---|
| Reads reflect latest state according to the model | Reads may temporarily be stale |
| More coordination | Less coordination |
| Often higher latency | Often lower latency |
| Useful for critical state | Useful for scalable read-heavy systems |

Choose based on business requirements.

---

# 12. Read-After-Write Consistency

A user updates:

```text
Name = "Sudhir"
```

and immediately reads:

```text
Name
```

If the read goes to a lagging replica:

```text
Old name
```

may be returned.

If the application requires immediate visibility, it may need:

```text
Read from primary
Session/causal routing
Synchronous replication
```

depending on the architecture.

---

# 13. Replication Lag

Example:

```text
Primary
  |
  +----> Replica 1
  |
  +----> Replica 2
```

If replicas process changes asynchronously:

```text
Primary = version 100
Replica = version 95
```

The difference is replication lag.

Monitor:

```text
Lag
Replication throughput
Replica health
```

---

# 14. Quorum

A quorum means enough nodes must participate for an operation to be considered successful.

For:

```text
N = 5 nodes
```

a majority quorum might be:

```text
3 nodes
```

because:

```text
floor(N/2) + 1 = 3
```

Quorum systems can help balance:

```text
Availability
Consistency
Fault tolerance
```

depending on the protocol.

---

# 15. Read and Write Quorums

Conceptually:

```text
N = total replicas
W = write acknowledgements
R = read responses
```

A common quorum relationship is:

```text
R + W > N
```

which can ensure read/write overlap under appropriate assumptions.

Real systems have additional details, so don't treat this equation as a universal guarantee.

---

# 16. Leader-Based Replication

One node acts as the leader:

```text
          Leader
         /      \
        v        v
    Replica 1  Replica 2
```

Writes go to:

```text
Leader
```

Leader replicates changes to followers.

Benefits:

```text
Simple write ordering
Centralized write coordination
```

---

# 17. Leader Election

If the leader fails:

```text
Leader
   X
   |
   v
Election
   |
   v
New Leader
```

The system needs a consensus or election protocol to avoid having multiple conflicting leaders.

---

# 18. Split Brain

Split brain occurs when multiple nodes believe they are the leader.

Example:

```text
Node A → "I am leader"
Node B → "I am leader"
```

Both may accept writes.

This can cause:

```text
Conflicting data
Duplicate work
Corruption
```

Quorum-based coordination and consensus algorithms help prevent this.

---

# 19. Consensus

Consensus allows distributed nodes to agree on a value or sequence despite certain failures.

Common consensus algorithms:

```text
Raft
Paxos
```

Consensus is used in systems that need coordinated decisions such as:

```text
Leader election
Replicated logs
Cluster membership/state
```

---

# 20. Raft

Raft is a consensus algorithm designed to be easier to understand and implement than Paxos.

Basic roles:

```text
Leader
Follower
Candidate
```

Normal operation:

```text
Leader
  |
  +--> Follower
  +--> Follower
```

The leader manages replicated log entries.

---

# 21. Raft Leader Election

If followers stop receiving heartbeats:

```text
Follower
   |
   v
Candidate
   |
   v
Request votes
   |
   v
Majority
   |
   v
Leader
```

Only a candidate receiving enough votes becomes leader.

---

# 22. Majority Quorum

For:

```text
5 nodes
```

majority:

```text
3
```

If one node fails:

```text
5 → 4 → still have majority
```

If two fail:

```text
5 → 3 → still have majority
```

If three fail:

```text
5 → 2 → no majority
```

The cluster cannot safely make quorum-dependent decisions.

---

# 23. Why Odd Number of Nodes?

Consider:

```text
3 nodes → majority = 2
5 nodes → majority = 3
```

Adding a fourth node:

```text
4 nodes → majority = 3
```

doesn't improve failure tolerance compared with three nodes.

This is why consensus clusters commonly use odd numbers of voting nodes.

---

# 24. Idempotency in Distributed Systems

Network retries are normal.

Example:

```text
Client
  |
  v
Payment Service
  |
  X timeout
```

Client retries:

```text
Payment Service
```

The first request may already have succeeded.

Use:

```text
Idempotency key
```

to prevent duplicate side effects.

---

# 25. Exactly-Once Is Difficult

Distributed systems commonly use:

```text
At-most-once
At-least-once
```

Exactly-once behavior is difficult to achieve end-to-end.

A practical design often combines:

```text
At-least-once delivery
+
Idempotent processing
```

to achieve effectively-once business behavior.

---

# 26. Clock Problems

Different machines have different clocks.

```text
Server A → 10:00:00.100
Server B → 10:00:00.250
```

Clock differences can cause problems with:

```text
Ordering
Timeouts
Distributed transactions
Event timestamps
```

Don't assume wall-clock timestamps perfectly establish causal ordering.

---

# 27. Logical Clocks

Distributed systems can use logical clocks to reason about event ordering.

A famous concept is:

```text
Lamport Clock
```

The goal is not to know exact real-world time.

The goal is to reason about:

```text
Which event happened before another?
```

---

# 28. Causal Ordering

Suppose:

```text
Event A
  |
  v
Event B
```

B depends on A.

The system should understand:

```text
A happened before B
```

Causal consistency attempts to preserve such relationships.

---

# 29. UUIDs and Distributed IDs

Multiple services may need to generate IDs independently.

Avoid relying only on:

```text
Database auto-increment
```

when global ID generation is required.

Options include:

```text
UUID
ULID
Snowflake-style IDs
```

---

# 30. Snowflake-Style IDs

A distributed ID can encode information such as:

```text
Timestamp
Worker/node ID
Sequence
```

This allows many machines to generate IDs without a central database counter.

Benefits:

```text
Distributed generation
Rough time ordering
Compact numeric representation
```

Exact format depends on the implementation.

---

# 31. Distributed Lock

Multiple instances may need exclusive access:

```text
Instance A
Instance B
Instance C
```

Only one should execute:

```text
Critical task
```

A distributed lock can coordinate this.

Possible technologies:

```text
Redis
Database
ZooKeeper
etcd
```

Use locks carefully because distributed locks have failure and lease-expiration complexities.

---

# 32. Lock Lease

A distributed lock should generally have an expiration/lease.

Example:

```text
Lock acquired
TTL = 30 sec
```

If the holder crashes:

```text
Lock eventually expires
```

Without expiration:

```text
Crash
  |
  v
Permanent lock
```

which can block the system.

---

# 33. Fencing Tokens

For important distributed locks, a simple lease may not be enough.

Example:

```text
Old owner
  |
Lock expires
  |
New owner gets lock
```

If the old owner continues working because of a delayed network response, both may perform operations.

Fencing tokens provide monotonically increasing ownership values so downstream systems can reject stale owners.

---

# 34. Distributed Transactions

A transaction spanning multiple independent services is difficult.

Example:

```text
Order DB
Payment DB
Inventory DB
```

A single normal database transaction cannot safely cover all of them.

Prefer:

```text
Saga
Outbox
Events
Compensating actions
```

when appropriate.

---

# 35. Saga

A Saga breaks a distributed workflow into local transactions.

Example:

```text
Create Order
    |
    v
Reserve Inventory
    |
    v
Process Payment
    |
    v
Create Shipment
```

If payment fails:

```text
Release Inventory
    |
    v
Cancel Order
```

---

# 36. Choreography vs Orchestration

### Choreography

Services react to events:

```text
OrderCreated
     |
     v
Inventory
     |
InventoryReserved
     |
     v
Payment
```

No central coordinator.

### Orchestration

A central coordinator manages the workflow:

```text
Saga Orchestrator
   |
   +--> Inventory
   |
   +--> Payment
   |
   +--> Shipping
```

---

# 37. Outbox Pattern

A service needs to update its database and publish an event.

Bad:

```text
Update DB
   |
   X
Publish event
```

If publishing fails:

```text
DB updated
Event missing
```

Outbox pattern:

```text
Database Transaction
   |
   +--> Business Data
   |
   +--> Outbox Event
```

A separate publisher reads the outbox and sends the event.

---

# 38. Why Outbox Helps

The business update and outbox record are committed atomically.

Then:

```text
Outbox
  |
  v
Publisher
  |
  v
Kafka
```

If publishing fails:

```text
Retry later
```

This reduces the dual-write problem.

---

# 39. Exactly-Once Business Processing

A robust pattern:

```text
Event
  |
  v
Consumer
  |
  +--> Check processed event ID
  |
  +--> Perform idempotent operation
  |
  +--> Mark processed
```

The exact transaction boundary depends on the storage and messaging system.

---

# 40. Distributed System Observability

A request may travel:

```text
Gateway
 ↓
Order
 ↓
Payment
 ↓
Inventory
 ↓
Kafka
 ↓
Worker
```

Use:

```text
Trace ID
Correlation ID
Structured logs
Metrics
Distributed tracing
```

to reconstruct the request path.

---

# 41. Timeout Design

Never wait forever for another service.

Bad:

```text
Service A
   |
   v
Service B
   |
  wait...
```

Good:

```text
Service A
   |
   v
Service B
   |
 timeout
   |
   v
Fallback/error
```

Timeouts protect resources.

---

# 42. Retry With Backoff

A basic retry strategy:

```text
Attempt 1
   |
   wait
Attempt 2
   |
   wait longer
Attempt 3
```

Use:

```text
Exponential backoff
Jitter
Retry limits
```

to avoid synchronized retry storms.

---

# 43. Retry Storm

Suppose 10,000 clients retry at exactly the same time:

```text
Service fails
   |
   v
10,000 retries
   |
   v
Service overloaded
   |
   v
More failures
```

This creates a feedback loop.

Use:

```text
Backoff
Jitter
Circuit breakers
Rate limits
```

---

# 44. Bulkhead

Separate resources for different workloads:

```text
Payment → Pool A
Search  → Pool B
```

If payment becomes slow:

```text
Pool A exhausted
```

Search can continue using:

```text
Pool B
```

---

# 45. Graceful Degradation

If a non-critical service fails:

```text
Recommendation Service
        X
```

the product page can still work:

```text
Product page
+ no recommendations
```

Protect critical business functions first.

---

# 46. Backpressure

If consumers process slower than producers:

```text
Producer rate = 10,000 msg/s
Consumer rate = 2,000 msg/s
```

Backlog grows.

Possible responses:

```text
Scale consumers
Throttle producers
Queue
Drop low-priority work
Apply load shedding
```

---

# 47. Distributed System Failure Checklist

For every remote dependency ask:

```text
What if it is slow?
What if it is unavailable?
What if the network fails?
What if the request succeeds but response is lost?
What if it returns duplicates?
What if retries happen?
What if data is stale?
What if two nodes act simultaneously?
```

This mindset is extremely useful in system-design interviews.

---

# 48. CAP vs PACELC

A more advanced model is:

```text
CAP:
During Partition → Consistency vs Availability
```

PACELC adds:

```text
Else
→ Latency vs Consistency
```

Meaning:

```text
If Partition:
    choose Availability or Consistency

Else:
    choose Latency or Consistency
```

This is an advanced interview topic. Understand the idea; don't overcomplicate a basic interview.

---

# 49. Availability Zones

Cloud regions often contain multiple availability zones.

Example:

```text
Region
 ├── AZ-A
 ├── AZ-B
 └── AZ-C
```

Deploying across multiple zones reduces the impact of a single-zone failure.

---

# 50. Multi-AZ Architecture

```text
             Load Balancer
              /    |    \
             v     v     v
           AZ-A  AZ-B  AZ-C
            API   API   API
             \    |    /
              Database
```

For databases, the exact replication/failover model depends on the technology.

---

# 51. Region vs Availability Zone

### Availability Zone

A separate failure domain within a region.

### Region

A larger geographic location containing one or more availability zones.

Example conceptually:

```text
Region
  |
  +-- AZ 1
  +-- AZ 2
  +-- AZ 3
```

---

# 52. Fault Domains

A fault domain is a group of infrastructure that can fail together.

Good architecture spreads replicas across independent failure domains.

Avoid:

```text
Primary + Replica
both in same failure domain
```

if that failure domain can take them both down.

---

# 53. Data Locality

Keeping data close to compute can reduce:

```text
Network latency
Cross-region traffic
Cost
```

But moving data for locality can introduce:

```text
Replication complexity
Consistency challenges
```

---

# 54. Distributed System Trade-Offs

There is rarely a perfect design.

Typical trade-offs:

```text
Consistency ↔ Availability
Latency ↔ Consistency
Cost ↔ Redundancy
Simplicity ↔ Scalability
Freshness ↔ Cache performance
Durability ↔ Write latency
```

Good system design explains these trade-offs instead of pretending everything can be maximized simultaneously.

---

# 55. Interview Question — What Is Eventual Consistency?

Answer:

> "Eventual consistency means replicas may temporarily return older data, but assuming updates continue to propagate and failures are resolved, the replicas eventually converge to the same state."

---

# 56. Interview Question — What Is Split Brain?

Answer:

> "Split brain happens when multiple nodes independently believe they are the leader or active owner. It can cause conflicting writes or duplicate work, so systems use quorum and consensus mechanisms to prevent it."

---

# 57. Interview Question — Why Is a Network Timeout Difficult?

Answer:

> "A timeout doesn't tell us whether the remote operation failed or actually succeeded but the response was lost. That's why retries can create duplicate side effects, and operations such as payments need idempotency."

---

# 58. Interview Question — What Is Consensus?

Answer:

> "Consensus is a mechanism that allows distributed nodes to agree on a value or ordered state despite certain failures. Raft and Paxos are common consensus algorithms."

---

# 59. Interview Question — Why Use a Quorum?

Answer:

> "A quorum requires enough replicas to agree before an operation is accepted. A majority helps prevent conflicting decisions and allows the system to tolerate some node failures."

---

# 60. Interview Question — How Do You Handle Distributed Transactions?

Answer:

> "I prefer local transactions within each service and use Saga, events, the outbox pattern and compensating actions for multi-service workflows rather than trying to use one global database transaction."

---

# 61. Interview Question — How Do You Prevent Duplicate Processing?

Answer:

> "I'd assume messages or requests can be retried and design the consumer to be idempotent. I can also persist processed event IDs or use an idempotency key where appropriate."

---

# 62. Interview Question — What Happens If Kafka Is Down?

Answer:

> "I'd determine whether the operation can safely continue without publishing. For critical events, I'd persist them in an outbox and retry publication later rather than losing the event."

---

# 63. Interview Question — How Do You Handle a Slow Dependency?

Answer:

> "I'd set a timeout, use a limited retry policy with exponential backoff and jitter, and consider a circuit breaker and bulkhead. If the dependency is non-critical, I'd provide a graceful fallback."

---

# 64. Practical Scenario — Two Services Update the Same Data

Ask:

```text
Who owns the data?
Can writes be serialized?
Do we need optimistic locking?
Do we need a distributed lock?
Can the workflow be redesigned?
```

Avoid adding distributed locks before understanding the ownership problem.

---

# 65. Practical Scenario — User Sees Old Data After an Update

Possible causes:

```text
Replica lag
Cache staleness
Eventual consistency
Read routed to another replica
```

Solutions depend on the required consistency model.

---

# 66. Practical Scenario — Leader Fails

Typical process:

```text
Leader fails
    |
    v
Followers detect missing heartbeats
    |
    v
Election
    |
    v
New leader
    |
    v
Replication continues
```

Consensus/quorum rules determine whether the election is safe.

---

# 67. Practical Scenario — Request Times Out but Payment Succeeded

The client cannot know whether the payment completed.

Use:

```text
Idempotency key
Payment status lookup
Durable transaction record
```

A retry should not create a second payment.

---

# 68. Practical Scenario — Event Published but Consumer Fails

With at-least-once delivery:

```text
Event
 |
 v
Consumer
 |
 X failure
```

the message can be delivered again.

Consumer should be:

```text
Idempotent
```

and use:

```text
Retry
DLQ
Processed-event tracking
```

where appropriate.

---

# 69. Practical Scenario — Database and Kafka Must Both Be Updated

Avoid:

```text
DB update
+
Kafka publish
```

as two unrelated operations.

Prefer:

```text
DB transaction
 |
 +--> Business data
 +--> Outbox event
```

then:

```text
Outbox
  |
  v
Publisher
  |
  v
Kafka
```

---

# 70. Final Interview Checklist

You should be able to explain:

```text
□ Distributed systems
□ Partial failure
□ Network partitions
□ CAP theorem
□ Consistency
□ Availability
□ Eventual consistency
□ Read-after-write
□ Replication lag
□ Quorum
□ Leader/follower
□ Leader election
□ Split brain
□ Consensus
□ Raft basics
□ Majority
□ Idempotency
□ Exactly-once challenges
□ Clock problems
□ Logical clocks
□ Causal ordering
□ Distributed IDs
□ Distributed locks
□ Lock leases
□ Fencing tokens
□ Distributed transactions
□ Saga
□ Outbox
□ Choreography
□ Orchestration
□ Observability
□ Timeout
□ Retry/backoff/jitter
□ Retry storms
□ Bulkhead
□ Graceful degradation
□ Backpressure
□ PACELC basics
□ Availability Zones
□ Multi-AZ
□ Fault domains
□ Data locality
□ System-design trade-offs
```

---

# 71. One-Minute Interview Answer

### "What are the main challenges in distributed systems?"

> "The biggest challenge is that failures become partial and the network is unreliable. A service can be slow, unavailable or process a request even when the caller doesn't receive the response. That leads to problems around consistency, retries, duplicate processing, leader election and distributed transactions. I'd handle these using timeouts, controlled retries, idempotency, appropriate consistency models, replication, quorum or consensus where required, and patterns such as Saga and Outbox for distributed workflows."

---

# 72. Key Takeaway

> **Distributed system design is mostly about handling uncertainty: machines fail, networks fail, messages are delayed or duplicated, and different nodes can temporarily disagree. Strong designs make these failure modes explicit and choose the right consistency, availability and coordination strategy for the business requirement.**

**File 13 complete.**
