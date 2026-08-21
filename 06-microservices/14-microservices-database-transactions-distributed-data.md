# Microservices — Database per Service, Transactions & Distributed Data

This file covers how microservices manage databases, transactions, consistency and data ownership.

Core topics:

```text
Database per Service
Data Ownership
Database-per-Service Patterns
Shared Database Anti-Pattern
Local Transactions
ACID
Distributed Transactions
Two-Phase Commit
Saga
Eventual Consistency
CQRS
Read Models
Event Sourcing
Outbox Pattern
CDC
Idempotency
Optimistic Locking
Pessimistic Locking
Distributed Locks
Caching
Data Duplication
Denormalization
Cross-Service Queries
Reporting
Production Scenarios
Interview Questions
```

---

# 1. Database Ownership

A core microservices principle is:

> Each service should own the data required for its business domain.

Example:

```text
Order Service
    ↓
Order DB

Product Service
    ↓
Product DB

Payment Service
    ↓
Payment DB
```

---

# 2. Database per Service

This pattern gives each service control over its persistence model.

```text
Order Service
     ↓
   MySQL

Inventory Service
     ↓
  PostgreSQL

Payment Service
     ↓
   PostgreSQL
```

Services can even use different database technologies when justified.

---

# 3. Why Database per Service?

Benefits:

```text
Loose coupling
Independent schema changes
Independent deployment
Independent scaling
Clear ownership
Technology freedom
Failure isolation
```

---

# 4. Shared Database

A shared database looks like:

```text
Order Service ──┐
Product Service ├──→ Same DB
Payment Service ┘
```

This may look simple initially, but creates strong coupling.

---

# 5. Why Shared Database Is Risky

Problems include:

```text
Shared schema
Cross-service queries
Deployment coupling
Security coupling
Lock contention
Migration coordination
Unclear ownership
```

One service can accidentally break another service's assumptions.

---

# 6. Direct Cross-Service DB Access

Avoid:

```text
Order Service
     ↓
Payment DB
```

Prefer:

```text
Order Service
     ↓
Payment API/Event
     ↓
Payment Service
```

The owning service controls its data.

---

# 7. Local Transaction

A local transaction operates within one service's database.

Example:

```text
BEGIN
 ↓
Create Order
 ↓
Create Order Items
 ↓
COMMIT
```

Spring example:

```java
@Transactional
public Order createOrder(...) {
    ...
}
```

---

# 8. ACID

Traditional database transactions commonly provide:

```text
Atomicity
Consistency
Isolation
Durability
```

---

# 9. Atomicity

All operations succeed or the transaction rolls back.

```text
Create Order
Create Order Items
      ↓
Failure
      ↓
Rollback
```

---

# 10. Consistency

A transaction should preserve database integrity constraints and valid state according to the application's rules.

Examples:

```text
Foreign key constraints
Unique constraints
Check constraints
Business invariants
```

---

# 11. Isolation

Concurrent transactions should not incorrectly interfere with each other.

Common isolation levels include:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Exact behavior varies by database.

---

# 12. Durability

After a successful commit, the database should preserve the committed data despite normal failures.

---

# 13. Transaction Boundary

In Spring:

```java
@Transactional
```

normally defines a local transaction boundary around the method invocation.

A useful rule:

> Keep transaction boundaries around one service's local data operations.

Don't assume `@Transactional` automatically creates a transaction across multiple microservices.

---

# 14. The Distributed Transaction Problem

Suppose:

```text
Order DB
Inventory DB
Payment DB
```

A single business operation needs to update all three.

You cannot simply do:

```text
@Transactional
Order DB
Inventory DB
Payment DB
```

and expect three independent databases to participate automatically.

---

# 15. Why Distributed Transactions Are Difficult

Failures can happen between steps:

```text
Order committed
 ↓
Inventory committed
 ↓
Payment failed
```

Now the system is partially complete.

---

# 16. Two-Phase Commit

2PC is a distributed transaction protocol.

High-level:

```text
Coordinator
    |
Prepare
    |
+---+---+
|       |
DB A   DB B
|       |
+---+---+
    |
Commit
```

---

# 17. Phase 1 — Prepare

Coordinator asks participants:

```text
Can you commit?
```

Participants prepare the transaction.

---

# 18. Phase 2 — Commit

If all participants are ready:

```text
Coordinator
 ↓
COMMIT
```

Otherwise:

```text
ROLLBACK
```

---

# 19. Why 2PC Is Often Avoided in Microservices

Problems:

```text
Coordination overhead
Latency
Resource locking
Failure complexity
Availability impact
Operational complexity
```

Microservices commonly prefer local transactions plus asynchronous coordination.

---

# 20. Saga

Saga breaks a distributed business transaction into local transactions.

Example:

```text
Create Order
 ↓
Reserve Inventory
 ↓
Process Payment
```

Each service commits locally.

---

# 21. Compensation

If a later operation fails:

```text
Payment fails
```

perform a compensating action:

```text
Release Inventory
 ↓
Cancel Order
```

Compensation isn't the same as a database rollback.

It is a new business operation that offsets a previous action.

---

# 22. Saga Example

```text
Order Created
      ↓
Inventory Reserved
      ↓
Payment Failed
      ↓
Inventory Released
      ↓
Order Cancelled
```

---

# 23. Saga Orchestration

A central coordinator controls the workflow.

```text
Saga Orchestrator
       |
       +→ Order
       |
       +→ Inventory
       |
       +→ Payment
```

Advantages:

```text
Central workflow visibility
Explicit process
Easier to understand complex workflows
```

Disadvantages:

```text
Orchestrator complexity
Potential central dependency
```

---

# 24. Saga Choreography

Services react to events.

```text
OrderCreated
    ↓
Inventory
    ↓
InventoryReserved
    ↓
Payment
    ↓
PaymentCompleted
```

Advantages:

```text
Loose coupling
No central workflow controller
Natural event-driven architecture
```

Disadvantages:

```text
Complex event chains
Harder debugging
Workflow visibility can decrease
```

---

# 25. Eventual Consistency

With asynchronous workflows, all services may not reflect the latest state immediately.

Example:

```text
Order = CREATED
Inventory = not yet reserved
Payment = not yet processed
```

A short period of inconsistency is expected.

Eventually:

```text
Order = CONFIRMED
Inventory = RESERVED
Payment = COMPLETED
```

---

# 26. Why Eventual Consistency?

It allows:

```text
Independent services
Asynchronous processing
Better availability
Loose coupling
Independent scaling
```

But business workflows must be designed to tolerate intermediate states.

---

# 27. Pending States

Instead of pretending an operation is complete:

```text
Payment = SUCCESS
```

when payment is still being processed, use:

```text
Payment = PENDING
```

Then update later.

This is especially important for distributed workflows.

---

# 28. State Machine

A business process can be represented as states:

```text
CREATED
   ↓
PAYMENT_PENDING
   ↓
PAID
   ↓
CONFIRMED
```

Failure:

```text
PAYMENT_PENDING
   ↓
PAYMENT_FAILED
```

Explicit states make asynchronous systems easier to reason about.

---

# 29. Outbox Pattern

The dual-write problem:

```text
Update DB
+
Publish Event
```

What if:

```text
DB commit succeeds
Event publish fails
```

The database says:

```text
Order Created
```

but consumers never receive:

```text
OrderCreated
```

---

# 30. Outbox Solution

Write both business data and event record in the same local transaction:

```text
Transaction
   |
   +→ Orders table
   |
   +→ Outbox table
   |
 COMMIT
```

Then publish the outbox event asynchronously.

---

# 31. Outbox Flow

```text
Order Service
     |
     +→ Orders DB
     |
     +→ Outbox DB row
             ↓
       Outbox Publisher
             ↓
           Kafka
             ↓
        Consumers
```

---

# 32. Outbox Guarantees

The Outbox Pattern helps ensure:

```text
If business transaction commits,
the event record also exists.
```

It does not automatically guarantee that every downstream side effect is exactly once.

Consumers still need appropriate idempotency.

---

# 33. Change Data Capture

CDC means:

```text
Change Data Capture
```

It captures database changes and publishes them to downstream systems.

Conceptually:

```text
Database
   ↓
CDC
   ↓
Kafka
   ↓
Consumers
```

A common ecosystem example is Debezium.

---

# 34. Outbox vs CDC

Outbox:

```text
Application intentionally writes event
```

CDC:

```text
Infrastructure captures database changes
```

They can also be combined:

```text
Application
 ↓
Outbox table
 ↓
CDC
 ↓
Kafka
```

---

# 35. CQRS

CQRS means:

```text
Command Query Responsibility Segregation
```

It separates:

```text
Write model
```

from:

```text
Read model
```

---

# 36. CQRS Example

```text
Commands
   ↓
Write Model
   ↓
Events
   ↓
Read Model
   ↓
Queries
```

---

# 37. Why CQRS?

Useful when:

```text
Read and write workloads differ
Read model needs different structure
Complex queries need optimization
Different scaling requirements
```

Don't use CQRS automatically for every microservice.

---

# 38. Read Model

A read model is optimized for queries.

Example:

```text
Order DB
```

may normalize data for writes.

A read model might contain:

```text
Order ID
Customer Name
Product Name
Total
Payment Status
```

in a single query-friendly structure.

---

# 39. Denormalization

Instead of joining many tables:

```text
Order
Customer
Product
Payment
```

a read model can store frequently accessed fields together.

This improves read performance but introduces synchronization complexity.

---

# 40. CQRS Event Flow

```text
Command
 ↓
Order Service
 ↓
Order DB
 ↓
OrderCreated
 ↓
Kafka
 ↓
Read Model Consumer
 ↓
Read DB
```

Queries read:

```text
Read DB
```

instead of the transactional write database.

---

# 41. Event Sourcing

Event sourcing stores state changes as events.

Instead of only storing:

```text
Order status = PAID
```

store:

```text
OrderCreated
PaymentInitiated
PaymentCompleted
```

Current state can be reconstructed from events.

---

# 42. Event Sourcing vs Event-Driven Architecture

These are not the same.

Event-driven:

```text
Events are used for communication.
```

Event sourcing:

```text
Events are the primary source of persisted state/history.
```

You can have event-driven systems without event sourcing.

---

# 43. Event Sourcing Example

```text
OrderCreated
PaymentStarted
PaymentCompleted
OrderConfirmed
```

Replay:

```text
Event 1
 ↓
State 1

Event 2
 ↓
State 2

Event 3
 ↓
State 3
```

---

# 44. Event Sourcing Benefits

```text
Complete event history
Auditability
Replay
Temporal debugging
Natural event publication
```

---

# 45. Event Sourcing Costs

```text
More complexity
Event schema evolution
Storage growth
Replay complexity
Debugging complexity
Operational overhead
```

Use it when the business actually benefits from event history.

---

# 46. Snapshots

If there are thousands of events:

```text
Event 1
Event 2
...
Event 10,000
```

replaying everything can be expensive.

A snapshot can store:

```text
State after Event 9,000
```

Then replay only:

```text
Event 9,001
...
Event 10,000
```

---

# 47. Optimistic Locking

Optimistic locking assumes conflicts are relatively uncommon.

Example:

```text
Product
version = 10
```

User A reads:

```text
version 10
```

User B reads:

```text
version 10
```

A updates:

```text
version 11
```

B tries to update version 10:

```text
Conflict
```

---

# 48. JPA Optimistic Locking

Common approach:

```java
@Version
private Long version;
```

The persistence provider uses the version to detect concurrent updates.

---

# 49. Why Optimistic Locking?

Useful when:

```text
Concurrent conflicts are relatively rare
Long-running reads should not hold DB locks
```

---

# 50. Pessimistic Locking

Pessimistic locking assumes conflicts are likely.

Conceptually:

```text
SELECT ... FOR UPDATE
```

A database lock prevents competing transactions from modifying the locked row until the transaction completes, subject to database behavior.

---

# 51. Optimistic vs Pessimistic

| Optimistic | Pessimistic |
|---|---|
| Detect conflict later | Prevent conflict with lock |
| Version-based | DB locking |
| Good for low conflict | Useful for high conflict |
| Less blocking | More blocking |
| Retry/update after conflict | Wait for lock |

---

# 52. Distributed Lock

Sometimes coordination is needed across application instances.

Example:

```text
Instance A
Instance B
Instance C
```

Only one should execute:

```text
Scheduled job
```

A distributed lock can coordinate this.

Possible technologies:

```text
Redis
Database
ZooKeeper
```

depending on architecture.

---

# 53. Distributed Lock Risks

Distributed locks can introduce:

```text
Deadlocks
Lease expiration
Clock assumptions
Network partitions
Lock ownership problems
Operational complexity
```

Prefer simpler coordination when possible.

---

# 54. Caching

Caching reduces database load.

Example:

```text
Product Service
 ↓
Redis
 ↓
MySQL
```

---

# 55. Cache-Aside

Common pattern:

```text
Read
 ↓
Check cache
 ↓
Hit → return
Miss
 ↓
Read DB
 ↓
Write cache
 ↓
Return
```

---

# 56. Cache Invalidation

When data changes:

```text
Update DB
 ↓
Invalidate cache
```

or use an appropriate update strategy.

Remember:

> Cache invalidation is one of the hardest parts of distributed systems.

---

# 57. Stale Data

Caching can create:

```text
DB = price 100
Cache = price 90
```

The application must define whether temporary stale data is acceptable.

For critical values, choose consistency requirements carefully.

---

# 58. Distributed Cache Failure

Suppose:

```text
Redis DOWN
```

If Redis is only a cache:

```text
Fallback to DB
```

But protect the DB from a cache stampede.

---

# 59. Cache Stampede

Suppose a popular cache entry expires:

```text
10,000 requests
 ↓
Cache miss
 ↓
10,000 DB queries
```

The database can become overloaded.

Possible techniques:

```text
Request coalescing
Jittered TTLs
Prewarming
Locks
Stale-while-revalidate
```

---

# 60. Data Duplication

Microservices may intentionally duplicate data.

Example:

```text
Customer Service
      ↓
Customer data

Order Read Model
      ↓
customerName
```

This is acceptable when ownership is clear.

---

# 61. Why Duplicate Data?

Benefits:

```text
Fast reads
Reduced remote calls
Independent availability
Optimized read models
```

Cost:

```text
Synchronization complexity
Eventual consistency
Storage duplication
```

---

# 62. Cross-Service Queries

Avoid:

```sql
SELECT ...
FROM order_db.orders
JOIN customer_db.customers ...
```

across service-owned databases.

Instead:

```text
API composition
Read model
Event-driven projection
Data warehouse
```

---

# 63. Reporting

Operational databases are usually not ideal for complex enterprise analytics.

Architecture can use:

```text
Microservices
 ↓
Events/CDC
 ↓
Data Platform
 ↓
Analytics / Warehouse
```

This avoids putting heavy reporting queries on transactional databases.

---

# 64. Database Migration

Each service should manage its own schema migrations.

Common tools:

```text
Flyway
Liquibase
```

Migration example:

```text
V1__create_orders.sql
V2__add_status.sql
V3__add_created_at.sql
```

---

# 65. Backward-Compatible Migration

For zero/minimal downtime deployments:

Bad:

```text
Rename column immediately
```

Safer expand-and-contract:

```text
1. Add new column
2. Deploy code that supports both
3. Backfill data
4. Switch reads/writes
5. Remove old column later
```

---

# 66. Expand-and-Contract

Example:

```text
Old:
customer_name

New:
full_name
```

Phase 1:

```text
Add full_name
```

Phase 2:

```text
Write both
```

Phase 3:

```text
Read full_name
```

Phase 4:

```text
Stop old column
```

Phase 5:

```text
Remove old column
```

---

# 67. Database Scaling

Possible techniques:

```text
Indexes
Query optimization
Connection pooling
Read replicas
Caching
Partitioning
Sharding
Vertical scaling
Horizontal scaling
```

---

# 68. Read Replicas

For read-heavy workloads:

```text
Primary
  ↓
Writes

Replica 1
  ↓
Reads

Replica 2
  ↓
Reads
```

Remember replicas can have replication lag.

---

# 69. Sharding

Sharding distributes data across multiple database nodes.

Example:

```text
User ID 1-1M
 → Shard A

User ID 1M-2M
 → Shard B
```

Sharding introduces additional complexity:

```text
Routing
Rebalancing
Cross-shard queries
Transactions
```

---

# 70. Database-per-Service Does Not Mean Database Server per Service

You can have:

```text
One DB server
multiple isolated databases
```

or:

```text
Separate DB servers
```

The important principle is:

```text
Logical ownership
+
Controlled access
```

---

# 71. Transactional Outbox + Local Transaction

A powerful pattern:

```text
@Transactional
   |
   +→ Business state
   |
   +→ Outbox event
   |
 COMMIT
```

Both become part of one local transaction.

---

# 72. Idempotency in Distributed Workflows

Suppose:

```text
PaymentCompleted
```

is delivered twice.

Consumer should ensure:

```text
First → process
Second → ignore
```

Use:

```text
event ID
business key
unique constraint
processed-event table
```

---

# 73. Unique Constraint as Deduplication

Example:

```sql
UNIQUE(event_id)
```

Consumer attempts:

```text
INSERT event_id
```

First:

```text
Success
```

Duplicate:

```text
Constraint violation
```

Handle it as already processed where appropriate.

---

# 74. Exactly-Once Business Effect

Don't confuse:

```text
Exactly-once message delivery
```

with:

```text
Exactly-once business effect
```

Business effects often require:

```text
Idempotency
Transactions
Unique constraints
Reconciliation
```

---

# 75. Reconciliation

Distributed systems can reach ambiguous states.

Example:

```text
Local payment = PENDING
Provider = SUCCESS
```

A reconciliation job can compare:

```text
Local state
vs
External state
```

and repair mismatches.

---

# 76. Distributed Data Principles

Remember:

```text
Each service owns its data
Avoid shared DB coupling
Use local transactions
Use Saga for distributed workflows
Expect eventual consistency
Use Outbox for reliable event creation
Use idempotency
Use read models when needed
```

---

# 77. Production Scenario

### "Order DB commit succeeds but Kafka publish fails."

Answer:

```text
Use Outbox Pattern.
```

Flow:

```text
Order transaction
 ↓
Orders table + Outbox table
 ↓
Commit
 ↓
Publisher sends event
 ↓
Kafka
```

The event isn't lost just because the immediate Kafka call failed.

---

# 78. Production Scenario

### "Payment succeeds but Order Service doesn't receive the response."

Potential state:

```text
Payment Provider = SUCCESS
Order = PENDING
```

Don't charge again blindly.

Use:

```text
Idempotency
Provider status query
Reconciliation
```

---

# 79. Production Scenario

### "Two users update the same product stock."

Use an appropriate concurrency strategy:

```text
Optimistic locking
```

or:

```text
Pessimistic locking
```

depending on conflict rate and business behavior.

---

# 80. Production Scenario

### "Product data is read millions of times."

Consider:

```text
Redis cache
Read replicas
Indexes
CDN where appropriate
```

But define:

```text
How stale can product data be?
```

before choosing the cache strategy.

---

# 81. Production Scenario

### "Order page needs Order + Customer + Product."

Options:

```text
API composition
```

or:

```text
CQRS/read model
```

If the page is extremely read-heavy, a dedicated read model may reduce repeated remote calls.

---

# 82. Production Scenario

### "A schema change breaks another service."

Likely problem:

```text
Shared database/schema coupling
```

Better:

```text
Service-owned schema
API/event contract
Backward-compatible migration
```

---

# 83. Production Scenario

### "Reporting query is slowing the Order DB."

Move reporting workload toward:

```text
Read replica
Data warehouse
Analytics store
CDC/event pipeline
```

Don't let heavy reporting compete with critical transactional traffic.

---

# 84. Interview Question

### "What does database per service mean?"

Answer:

> "It means each microservice owns its persistence and other services don't directly access that database. This gives the service control over its schema and reduces coupling."

---

# 85. Interview Question

### "Can two microservices share a database?"

Answer:

> "They can technically, but it creates strong schema and deployment coupling. In a true microservice architecture I'd prefer service-owned data and communication through APIs or events unless there is a deliberate architectural reason to share storage."

---

# 86. Interview Question

### "How do you handle transactions across microservices?"

Answer:

> "I avoid treating multiple service databases as one local transaction. I'd normally use local transactions within each service and coordinate the business workflow using a Saga, with compensating actions when necessary."

---

# 87. Interview Question

### "What is 2PC?"

Answer:

> "Two-phase commit is a distributed transaction protocol with prepare and commit phases. It can provide stronger transaction coordination but introduces latency, locking and operational complexity, so many microservice architectures prefer Saga-based workflows."

---

# 88. Interview Question

### "What is Saga?"

Answer:

> "A Saga breaks a distributed transaction into local transactions. If a later step fails, compensating actions are performed for earlier steps. It can be coordinated through orchestration or choreography."

---

# 89. Interview Question

### "What is eventual consistency?"

Answer:

> "It means different services may temporarily have different states, especially when updates are propagated asynchronously. After events are processed, the services converge to a consistent state."

---

# 90. Interview Question

### "What is CQRS?"

Answer:

> "CQRS separates the write model from the read model. It is useful when read and write workloads have different requirements or when queries need a specialized projection. It adds complexity, so I would use it when the business needs justify it."

---

# 91. Interview Question

### "What is Event Sourcing?"

Answer:

> "Event sourcing stores state changes as an append-only sequence of events and reconstructs current state from those events. It provides a strong history and replay capability but adds significant complexity around event schemas and operations."

---

# 92. Interview Question

### "Outbox vs Event Sourcing?"

Answer:

> "Outbox solves reliable publication of events alongside a local database transaction. Event sourcing uses events as the primary persisted representation of state changes. An application can use an Outbox without using Event Sourcing."

---

# 93. Interview Question

### "What is optimistic locking?"

Answer:

> "Optimistic locking detects concurrent updates using a version or similar mechanism. If the version has changed since the entity was read, the update fails and the application can retry or report a conflict."

---

# 94. Interview Question

### "Optimistic vs pessimistic locking?"

Answer:

> "Optimistic locking assumes conflicts are relatively rare and detects them at update time. Pessimistic locking acquires database locks to prevent competing updates. I'd choose based on contention, transaction duration and database behavior."

---

# 95. Interview Question

### "Why duplicate data in microservices?"

Answer:

> "Sometimes duplicating a small amount of data is useful for performance and availability, especially in read models. The trade-off is synchronization complexity and eventual consistency."

---

# 96. Interview Question

### "How do you prevent direct database coupling?"

Answer:

> "I define clear data ownership. Other services access the owner's API or consume its events instead of querying its database directly. For reporting, I'd use dedicated read models or an analytics platform."

---

# 97. Interview Question

### "What is the dual-write problem?"

Answer:

> "It happens when an operation must update a database and publish a message separately. One operation can succeed while the other fails, creating inconsistent state. The Outbox Pattern is a common solution."

---

# 98. Final Architecture

```text
                    +----------------+
                    | Order Service  |
                    +-------+--------+
                            |
                    +-------+-------+
                    |               |
                    ↓               ↓
                Order DB       Outbox Table
                                    |
                                    ↓
                                  Kafka
                                    |
                +-------------------+-------------------+
                |                   |                   |
                ↓                   ↓                   ↓
           Inventory            Payment           Notification
            Service             Service              Service
                |                   |                   |
                ↓                   ↓                   ↓
          Inventory DB         Payment DB        Notification DB
```

Core rules:

```text
Service owns data
       ↓
Local transaction
       ↓
Outbox event
       ↓
Message broker
       ↓
Other services
       ↓
Eventual consistency
       ↓
Idempotent consumers
```

---

# 99. Final Mental Model

```text
Database per Service
→ Clear data ownership.

Local Transaction
→ Strong consistency within one service.

2PC
→ Distributed transaction coordination, but costly.

Saga
→ Distributed workflow using local transactions + compensation.

Eventual Consistency
→ Services converge over time.

Outbox
→ Reliable event creation with local state changes.

CDC
→ Capture database changes.

CQRS
→ Separate write and read models.

Event Sourcing
→ Store state changes as events.

Optimistic Locking
→ Detect concurrent conflicts.

Pessimistic Locking
→ Lock resources to prevent concurrent modification.

Cache
→ Reduce read load.

Read Model
→ Optimize complex/high-volume reads.

Reconciliation
→ Repair ambiguous distributed states.
```

---

# 100. Final Interview Answer

If asked:

> "How would you design data management for an e-commerce microservices system?"

Use:

> "I'd give each service ownership of its own data and avoid direct cross-service database access. Each service would use local ACID transactions for its own changes. For workflows spanning multiple services, I'd use a Saga with explicit states and compensating actions rather than relying on a single distributed transaction. For reliable event publication, I'd use the Outbox Pattern. Consumers would be idempotent because duplicate delivery is possible. For read-heavy use cases, I'd consider caching or CQRS read models, and for analytics I'd use a separate data platform rather than putting heavy queries on transactional databases."

---

# 101. Revision Checklist

```text
□ Database per service
□ Data ownership
□ Shared database
□ Direct DB access
□ Local transactions
□ ACID
□ Atomicity
□ Consistency
□ Isolation
□ Durability
□ Transaction boundaries
□ Distributed transactions
□ 2PC
□ Saga
□ Compensation
□ Orchestration
□ Choreography
□ Eventual consistency
□ Pending states
□ State machines
□ Outbox Pattern
□ CDC
□ CQRS
□ Read models
□ Denormalization
□ Event Sourcing
□ Snapshots
□ Optimistic locking
□ Pessimistic locking
□ Distributed locks
□ Caching
□ Cache-aside
□ Cache invalidation
□ Cache stampede
□ Data duplication
□ Cross-service queries
□ Reporting
□ Database migrations
□ Expand-and-contract
□ Read replicas
□ Sharding
□ Idempotency
□ Deduplication
□ Exactly-once caveats
□ Reconciliation
□ Production scenarios
```

---

# 102. The Interviewer's Real Test

If asked:

> "Order DB committed successfully, but the event was not published. How do you prevent this?"

Think:

```text
Business transaction
        ↓
+-----------------------+
| Order DB              |
| Outbox event          |
+-----------------------+
        ↓
      COMMIT
        ↓
Outbox Publisher
        ↓
      Kafka
        ↓
    Consumers
```

Then remember:

```text
Outbox
+
Idempotent consumers
+
Retry
+
DLQ
+
Monitoring
```

The key interview lesson is:

> **In microservices, don't try to pretend independent databases are one database. Keep transactions local, coordinate business workflows explicitly, and design for eventual consistency, duplicate messages and partial failure.**
