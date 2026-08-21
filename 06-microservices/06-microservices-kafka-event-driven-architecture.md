# Microservices — Kafka & Event-Driven Architecture

This file covers Kafka and event-driven microservices from an interview perspective.

The goal is not to memorize every Kafka configuration.

You should be able to explain:

```text
Why Kafka?
Topic
Partition
Producer
Consumer
Consumer Group
Offset
Broker
Replication
Ordering
Delivery semantics
Retries
Dead-letter handling
Idempotency
Event-driven architecture
Transactional Outbox
```

---

# 1. What Is Event-Driven Architecture?

In an event-driven architecture, services communicate by publishing and consuming events.

Example:

```text
Order Service
      |
      | OrderCreated
      ↓
   Message Broker
      |
      +----------+----------+
      ↓          ↓          ↓
Notification  Analytics  Inventory
```

The producer doesn't necessarily need to know every consumer.

---

# 2. What Is an Event?

An event represents something that happened.

Examples:

```text
OrderCreated
PaymentCompleted
InventoryReserved
OrderShipped
UserRegistered
```

An event is generally a fact.

Example:

```text
PaymentCompleted
```

means:

> Payment has completed.

---

# 3. Event vs Command

Very common interview question.

### Event

```text
Something happened.
```

Example:

```text
OrderCreated
```

### Command

```text
Please perform this action.
```

Example:

```text
ProcessPayment
```

A command expresses intent.

An event represents a fact.

---

# 4. Why Event-Driven Architecture?

Benefits include:

```text
Loose runtime coupling
Asynchronous processing
Scalability
Independent consumers
Fault isolation
Event history
Integration between services
```

---

# 5. Example Without Events

Suppose Order Service does:

```text
Order
 ↓
Notification
 ↓
Analytics
 ↓
Inventory
```

All synchronously.

Order Service may become tightly coupled to all three services.

---

# 6. Example With Events

Instead:

```text
Order Service
      |
      ↓
 OrderCreated
      |
      ↓
    Kafka
   /  |  \
  ↓   ↓   ↓
Inventory
Notification
Analytics
```

Order Service publishes one event.

Consumers process it independently.

---

# 7. What Is Kafka?

Apache Kafka is a distributed event streaming platform.

It is commonly used for:

```text
Event streaming
Asynchronous communication
High-throughput data pipelines
Service integration
Log/event processing
Analytics
```

---

# 8. Kafka Is More Than a Queue

A traditional queue often emphasizes:

```text
Message
 ↓
Consumer
 ↓
Message removed
```

Kafka stores records durably and allows consumers to track their own positions.

This enables:

```text
Replay
Multiple consumer groups
Event history
Independent consumption
```

---

# 9. Kafka Architecture

Basic architecture:

```text
Producer
   |
   ↓
Kafka Cluster
   |
   ↓
Consumer
```

A Kafka cluster contains multiple brokers.

---

# 10. Broker

A broker is a Kafka server.

Example:

```text
Kafka Cluster
├── Broker 1
├── Broker 2
└── Broker 3
```

Brokers store and serve Kafka records.

---

# 11. Kafka Cluster

A cluster is a group of Kafka brokers working together.

Example:

```text
           Kafka Cluster
       +-------------------+
       |                   |
   Broker 1            Broker 2
       |                   |
       +---------+---------+
                 |
             Broker 3
```

A production cluster typically uses multiple brokers for availability and scalability.

---

# 12. Topic

A topic is a named stream/category of records.

Examples:

```text
orders
payments
inventory-events
user-events
```

Producer:

```text
Order Service
      ↓
orders topic
```

Consumers can subscribe to the topic.

---

# 13. Topic Example

```text
Topic: orders

OrderCreated
OrderCreated
OrderCancelled
OrderShipped
OrderCreated
```

The topic can contain many records.

---

# 14. Partition

A topic is divided into partitions.

Example:

```text
orders
├── Partition 0
├── Partition 1
└── Partition 2
```

Partitions provide:

```text
Parallelism
Scalability
Ordering within a partition
```

---

# 15. Why Partitions?

Suppose:

```text
1 partition
```

limits parallel processing.

With:

```text
10 partitions
```

multiple consumers can process records concurrently.

This allows Kafka to scale workloads horizontally.

---

# 16. Partition Ordering

Kafka guarantees ordering within a partition.

Example:

```text
Partition 0

OrderCreated
PaymentCompleted
OrderShipped
```

The records are read in partition order.

Kafka does not provide one global ordering guarantee across all partitions.

---

# 17. Ordering by Key

Suppose we publish events with:

```text
key = orderId
```

Kafka can consistently route records with the same key to the same partition under normal partitioning behavior.

Then:

```text
orderId = 101
```

events can remain ordered relative to each other.

Example:

```text
OrderCreated
PaymentCompleted
OrderShipped
```

for order 101.

---

# 18. Why Key by Business Entity?

If order events are keyed by:

```text
orderId
```

all events for one order can be processed in partition order.

Similarly:

```text
customerId
accountId
productId
```

can be useful keys depending on the business workflow.

---

# 19. Producer

A producer publishes records to Kafka.

Example:

```text
Order Service
     |
     | OrderCreated
     ↓
orders topic
```

The producer decides:

```text
Topic
Key
Value
Headers
```

---

# 20. Consumer

A consumer reads records from Kafka.

Example:

```text
orders topic
     ↓
Order Notification Consumer
```

The consumer processes events.

---

# 21. Consumer Group

A consumer group allows multiple consumer instances to share work.

Example:

```text
orders topic
  |
  +-- Partition 0
  +-- Partition 1
  +-- Partition 2

Consumer Group: notification
  |
  +-- Consumer A
  +-- Consumer B
  +-- Consumer C
```

Each partition is assigned to one consumer within the group at a time.

---

# 22. Why Consumer Groups?

They provide:

```text
Parallel processing
Horizontal scalability
Work distribution
Fault recovery
```

---

# 23. One Consumer Group vs Multiple Groups

Suppose:

```text
OrderCreated
```

must be consumed by:

```text
Notification
Analytics
Inventory
```

Use separate consumer groups:

```text
orders topic
   |
   +---- notification-group
   |
   +---- analytics-group
   |
   +---- inventory-group
```

Each group gets its own logical consumption position.

---

# 24. Consumer Group Example

Within:

```text
notification-group
```

you can have:

```text
Consumer 1
Consumer 2
Consumer 3
```

They divide partitions among themselves.

But:

```text
analytics-group
```

can independently consume the same events.

---

# 25. Maximum Parallelism

If a topic has:

```text
3 partitions
```

a consumer group can have at most roughly:

```text
3 actively assigned consumers
```

for that topic at a time.

Additional consumers may remain idle for that partition assignment.

---

# 26. Offset

An offset identifies a record's position within a partition.

Conceptually:

```text
Partition 0

offset 0 → Event A
offset 1 → Event B
offset 2 → Event C
offset 3 → Event D
```

Consumers track their progress using offsets.

---

# 27. Why Offsets Matter

Suppose consumer processed:

```text
offset 100
```

and then crashes.

After recovery it can continue based on its committed position rather than starting from the beginning.

---

# 28. Consumer Offset

Conceptually:

```text
Processed:
0
1
2
3
4
```

Current committed position:

```text
5
```

The consumer can resume from its stored position according to Kafka's offset semantics.

---

# 29. Offset Commit

A consumer can commit offsets after processing records.

Conceptually:

```text
Read event
 ↓
Process event
 ↓
Commit offset
```

This helps avoid losing knowledge of processing progress.

---

# 30. Commit Too Early

Bad pattern:

```text
Read
 ↓
Commit
 ↓
Process
 ↓
Crash
```

The record may be considered consumed even though business processing didn't complete.

This can cause message loss from the consumer's perspective.

---

# 31. Commit Too Late

Opposite:

```text
Read
 ↓
Process successfully
 ↓
Crash before commit
```

After restart, the same event may be processed again.

This can lead to duplicate processing.

Therefore consumers should generally be designed to be idempotent.

---

# 32. At-Most-Once

Conceptually:

```text
Commit before processing
```

Potential result:

```text
0 or 1 processing
```

A failure can cause a record to be skipped.

---

# 33. At-Least-Once

Conceptually:

```text
Process
 ↓
Commit
```

If the consumer crashes between processing and commit:

```text
Same event may be processed again.
```

This is why idempotency matters.

---

# 34. Exactly-Once

Exactly-once is more nuanced than:

> "The message is delivered exactly once."

It refers to achieving exactly-once processing semantics under specific Kafka/application guarantees.

It requires careful architecture.

Don't casually claim that Kafka automatically makes every business operation exactly once.

---

# 35. Idempotent Consumer

Suppose:

```text
OrderCreated eventId = abc123
```

Consumer receives it twice.

Store processed IDs:

```text
abc123
```

Second time:

```text
Already processed
→ skip duplicate side effect
```

---

# 36. Idempotency Storage

Possible approaches:

```text
Database table
Redis
Unique business key
Processed-event table
```

Example:

```text
processed_events
----------------
event_id
processed_at
```

Add a unique constraint on:

```text
event_id
```

to help prevent duplicate processing.

---

# 37. Event ID

An event should generally have a unique identifier.

Example:

```json
{
  "eventId": "evt-123",
  "eventType": "OrderCreated",
  "orderId": 101
}
```

The event ID can support:

```text
Idempotency
Tracing
Debugging
Audit
```

---

# 38. Event Schema

Example:

```json
{
  "eventId": "evt-123",
  "eventType": "OrderCreated",
  "occurredAt": "2026-08-21T10:30:00Z",
  "orderId": 101,
  "customerId": 20
}
```

A consistent event envelope makes event processing easier.

---

# 39. Event Versioning

Events evolve.

Version 1:

```json
{
  "orderId": 101,
  "customerId": 20
}
```

Later:

```json
{
  "orderId": 101,
  "customerId": 20,
  "currency": "INR"
}
```

Adding optional fields is generally safer than removing or changing existing fields.

---

# 40. Schema Registry

A schema registry can manage event schemas.

Benefits:

```text
Schema validation
Version management
Compatibility checks
```

Commonly used in Kafka ecosystems.

---

# 41. Schema Evolution

Avoid breaking old consumers.

Safer:

```text
Add optional field
```

Riskier:

```text
Remove field
Rename field
Change field type
Change field meaning
```

---

# 42. Kafka Replication

Kafka partitions can have replicas.

Example:

```text
Partition 0
├── Broker 1 → Leader
├── Broker 2 → Replica
└── Broker 3 → Replica
```

Replication improves fault tolerance.

---

# 43. Leader

A partition has a leader replica that handles normal reads/writes for that partition.

Other replicas maintain copies.

If the leader fails, Kafka can elect an eligible replica according to its replication configuration.

---

# 44. Replication Factor

Example:

```text
Replication Factor = 3
```

means each partition has three replicas.

It improves resilience but consumes additional storage/network resources.

---

# 45. ISR

ISR means:

```text
In-Sync Replicas
```

These are replicas sufficiently caught up with the leader according to Kafka's replication rules.

If a broker falls behind significantly, it may temporarily leave the ISR.

---

# 46. Producer Acknowledgments

Kafka producers can configure acknowledgments.

Common levels:

```text
acks=0
acks=1
acks=all
```

---

# 47. acks=0

Producer doesn't wait for broker acknowledgment.

Advantages:

```text
Low latency
```

Risk:

```text
Possible message loss
```

---

# 48. acks=1

Leader acknowledges the record after accepting it.

Provides more durability than:

```text
acks=0
```

but replication guarantees depend on configuration and timing.

---

# 49. acks=all

The leader waits for acknowledgment from the required in-sync replicas according to the cluster configuration.

This generally provides stronger durability.

It can increase latency.

---

# 50. Producer Idempotence

Kafka producers can be configured for idempotent production.

This helps prevent certain duplicate records caused by producer retries.

Important:

> Producer idempotence does not automatically make your entire business workflow exactly once.

---

# 51. Kafka Retention

Kafka does not necessarily delete a record immediately after a consumer reads it.

Records are retained according to topic policies.

Examples:

```text
Time-based retention
Size-based retention
```

This enables replay.

---

# 52. Replay

Suppose:

```text
Analytics consumer
```

has a bug.

After fixing it, you may be able to replay historical events from earlier offsets, depending on retention and available data.

This is a major Kafka advantage.

---

# 53. Consumer Lag

Consumer lag measures how far a consumer group is behind the latest available records.

Example:

```text
Latest offset = 1000
Consumer offset = 900

Lag ≈ 100 records
```

Lag is an important operational metric.

---

# 54. Why Consumer Lag Matters

High lag can indicate:

```text
Consumer too slow
Traffic spike
Downstream dependency slow
Insufficient consumers
Partition imbalance
Processing errors
```

---

# 55. Consumer Scaling

Suppose:

```text
Topic = 10 partitions
```

You can scale a consumer group across multiple instances.

Example:

```text
Consumer 1 → partitions 0,1
Consumer 2 → partitions 2,3
Consumer 3 → partitions 4,5
Consumer 4 → partitions 6,7
Consumer 5 → partitions 8,9
```

The exact assignment depends on the partition assignment strategy.

---

# 56. Rebalancing

When consumers join or leave a consumer group:

```text
Partitions
   ↓
Reassigned
```

This is called rebalancing.

During rebalancing, processing behavior can temporarily change.

---

# 57. Consumer Failure

Suppose:

```text
Consumer A
```

dies.

Kafka detects the group membership change and reassigns its partitions to another consumer in the same group.

This supports fault recovery.

---

# 58. Poison Message

A poison message repeatedly causes processing failure.

Example:

```text
Malformed event
```

Consumer:

```text
Read
 ↓
Fail
 ↓
Retry
 ↓
Fail
 ↓
Retry
```

This can block progress depending on the processing model.

---

# 59. Dead Letter Topic

A failed event can eventually be moved to a dead-letter topic.

Example:

```text
orders
 ↓
Consumer
 ↓
failure
 ↓
retry
 ↓
retry
 ↓
orders.DLT
```

The exact DLT naming/configuration depends on the framework.

---

# 60. Retry Topic

Another approach is:

```text
orders
 ↓
retry topic
 ↓
consumer
```

with delayed/repeated processing.

This can separate retry traffic from the main topic.

---

# 61. Why Not Retry Forever?

Because:

```text
One bad event
 ↓
blocks processing
 ↓
consumer lag increases
 ↓
system becomes unhealthy
```

Retries should be bounded.

---

# 62. Event Ordering vs Parallelism

More partitions:

```text
More parallelism
```

but global ordering becomes harder.

One partition:

```text
Simple ordering
```

but limited parallelism.

Kafka design is therefore a trade-off.

---

# 63. Partition Count Is an Architectural Decision

Changing partition count can affect key-based distribution and ordering behavior.

Don't choose:

```text
1000 partitions
```

just because "more partitions = faster."

Consider:

```text
Traffic
Consumer parallelism
Broker capacity
Ordering
Operational overhead
```

---

# 64. Kafka Consumer Groups and Broadcast

If you want multiple independent applications to receive every event:

```text
Topic
 |
 +-- Group A
 |
 +-- Group B
 |
 +-- Group C
```

Each group receives the event independently.

If you put all consumers in one group:

```text
Group A
 ├── Consumer 1
 ├── Consumer 2
 └── Consumer 3
```

they share the workload.

---

# 65. Kafka vs RabbitMQ

Simple interview comparison:

### Kafka

```text
Event streaming
High throughput
Durable log
Replay
Multiple consumer groups
```

### RabbitMQ

```text
Traditional messaging
Queues
Routing
Task distribution
Acknowledgment-driven workflows
```

Neither is universally better.

---

# 66. Kafka vs REST

REST:

```text
Synchronous
Request/response
Immediate result
```

Kafka:

```text
Asynchronous
Event-driven
Decoupled consumers
Replayable stream
```

They often coexist.

---

# 67. Example E-Commerce Architecture

```text
                   Client
                     |
                     ↓
                API Gateway
                     |
                     ↓
                Order Service
                     |
                     | OrderCreated
                     ↓
                    Kafka
          +----------+----------+
          |          |          |
          ↓          ↓          ↓
     Inventory   Notification  Analytics
       Service      Service      Service
          |
          ↓
     Inventory DB
```

Order Service doesn't need to synchronously call all three consumers.

---

# 68. Order Workflow

Example:

```text
1. Client creates order
2. Order Service validates request
3. Order is persisted
4. OrderCreated event is published
5. Inventory consumes event
6. Notification consumes event
7. Analytics consumes event
```

The exact transaction/event reliability mechanism is important.

---

# 69. The Dual-Write Problem

Dangerous:

```text
Database
   ↓
commit

Kafka
   ↓
publish
```

What if:

```text
DB succeeds
Kafka publish fails
```

Now:

```text
Order exists
but OrderCreated event is missing
```

---

# 70. Reverse Dual-Write Problem

What if:

```text
Kafka publish succeeds
DB commit fails
```

Now consumers see:

```text
OrderCreated
```

even though the order doesn't exist.

This is a consistency problem.

---

# 71. Transactional Outbox

A common solution is the transactional outbox pattern.

Instead of writing:

```text
Order DB
Kafka
```

separately, write:

```text
Order
+
Outbox Event
```

in the same database transaction.

---

# 72. Outbox Flow

```text
Order Service
      |
      ↓
Database Transaction
  ├── orders
  └── outbox
      |
      ↓
Transaction commits
      |
      ↓
Outbox Publisher
      |
      ↓
Kafka
```

If the transaction commits, the event is durably recorded for publishing.

---

# 73. Outbox Publisher

A separate process/thread can read:

```text
outbox
```

and publish events to Kafka.

After successful publication, it can mark the outbox record as published according to the chosen implementation.

---

# 74. Why Outbox Helps

It solves the atomicity problem between:

```text
Business database write
```

and:

```text
Event creation
```

It does not magically eliminate every delivery problem.

Consumers still need idempotency.

---

# 75. Outbox and Duplicate Events

Suppose:

```text
Publish succeeds
 ↓
Publisher crashes before marking outbox row complete
 ↓
Publisher retries
 ↓
Same event published again
```

Therefore:

```text
Consumers should be idempotent.
```

This is a key interview point.

---

# 76. Kafka Transactions

Kafka supports transactions for certain Kafka-to-Kafka processing scenarios.

But don't confuse:

```text
Kafka transaction
```

with:

```text
Database + Kafka atomic transaction
```

A Kafka transaction does not automatically make an arbitrary external database transaction atomic with Kafka.

---

# 77. Exactly-Once Business Effect

Even if Kafka provides exactly-once processing guarantees in a particular setup, your external side effects still require careful design.

Example:

```text
Kafka
 ↓
Payment Provider
```

Kafka exactly-once semantics do not automatically make an external payment API exactly once.

You still need:

```text
Idempotency
```

where appropriate.

---

# 78. Eventual Consistency

Event-driven systems commonly introduce eventual consistency.

Example:

```text
Order created
 ↓
Order DB updated immediately
 ↓
Inventory event processed later
```

There can be a temporary period where:

```text
Order = CREATED
Inventory = not yet updated
```

---

# 79. Is Eventual Consistency Bad?

Not necessarily.

It depends on the business requirement.

Good fit:

```text
Analytics
Notifications
Search indexing
Recommendations
```

Potentially problematic:

```text
Payment authorization
Inventory reservation
```

unless the workflow is carefully designed.

---

# 80. Saga Pattern

For distributed business workflows, a Saga coordinates multiple local transactions.

Example:

```text
Create Order
   ↓
Reserve Inventory
   ↓
Process Payment
   ↓
Create Shipment
```

If payment fails:

```text
Release Inventory
Cancel Order
```

These are compensating actions.

---

# 81. Saga Orchestration

An orchestrator controls the workflow.

```text
Saga Orchestrator
      |
      +--> Order
      |
      +--> Inventory
      |
      +--> Payment
      |
      +--> Shipping
```

The orchestrator knows the sequence.

---

# 82. Saga Choreography

Services react to events.

```text
OrderCreated
   ↓
InventoryReserved
   ↓
PaymentCompleted
   ↓
ShipmentCreated
```

Each service publishes events that trigger the next step.

---

# 83. Orchestration vs Choreography

### Orchestration

```text
Central coordinator
```

Pros:

```text
Explicit workflow
Easier to visualize
Central control
```

Cons:

```text
Orchestrator can become complex
```

### Choreography

```text
Services react to events
```

Pros:

```text
Less central coordination
Looser coupling
```

Cons:

```text
Workflow can become difficult to understand
```

---

# 84. Kafka and Saga

Kafka can transport events used by Saga workflows.

Example:

```text
OrderCreated
      ↓
Kafka
      ↓
Inventory Service
      ↓
InventoryReserved
      ↓
Kafka
      ↓
Payment Service
```

Kafka is a communication mechanism, not the Saga itself.

---

# 85. Event Naming

Prefer names representing facts:

```text
OrderCreated
PaymentCompleted
InventoryReserved
```

Avoid vague names:

```text
OrderEvent
DataUpdated
ProcessThing
```

Clear event names improve understanding.

---

# 86. Event Payload

A useful event might contain:

```json
{
  "eventId": "evt-123",
  "eventType": "OrderCreated",
  "occurredAt": "2026-08-21T10:30:00Z",
  "orderId": 101,
  "customerId": 20,
  "totalAmount": 5999
}
```

Don't expose unnecessary internal database details.

---

# 87. Event Ownership

The service owning the domain should publish its domain events.

Example:

```text
Order Service
→ OrderCreated
```

not:

```text
Payment Service
→ OrderCreated
```

---

# 88. Consumer Independence

A consumer should ideally depend on:

```text
Event contract
```

rather than:

```text
Producer's database
```

Good:

```text
Kafka event
```

Bad:

```text
Consumer reads Producer DB directly
```

---

# 89. Event-Driven API Design

Don't turn every API call into an event.

Use events when:

```text
Asynchronous processing
Decoupling
Integration
Event history
Independent consumers
```

are useful.

Use REST/gRPC when:

```text
Immediate response
Request-specific query
Synchronous validation
```

is required.

---

# 90. Event-Driven Architecture Isn't Free

Costs include:

```text
Operational complexity
Eventual consistency
Debugging complexity
Schema management
Duplicate handling
Ordering
Replay management
Monitoring
```

Use it because it solves a real problem.

---

# 91. Event Observability

Important metadata:

```text
eventId
correlationId
traceId
eventType
occurredAt
producer
version
```

This helps answer:

```text
Where did this event come from?
Which request created it?
Which consumers processed it?
```

---

# 92. Correlation ID

Example:

```text
Client request
correlationId = abc123
        ↓
OrderCreated
correlationId = abc123
        ↓
InventoryReserved
correlationId = abc123
```

This makes distributed workflows easier to trace.

---

# 93. Consumer Metrics

Monitor:

```text
Consumer lag
Processing latency
Error rate
Retry count
DLT count
Throughput
Rebalances
```

---

# 94. Producer Metrics

Monitor:

```text
Publish rate
Publish latency
Error rate
Retry rate
Record size
Broker response
```

---

# 95. Kafka Capacity

Important dimensions:

```text
Partitions
Brokers
Replication factor
Throughput
Message size
Consumer speed
Network
Disk
```

Don't evaluate Kafka capacity using CPU alone.

---

# 96. Large Messages

Avoid unnecessarily huge Kafka messages.

Large messages can cause:

```text
Network overhead
Memory pressure
Consumer latency
Broker storage pressure
```

If payloads are large, consider storing the large object elsewhere and publishing a reference where appropriate.

---

# 97. Event Contract vs Database Schema

Don't make event schema identical to the database schema by default.

Database:

```text
Internal storage model
```

Event:

```text
Integration contract
```

They serve different purposes.

---

# 98. Event Immutability

An event should represent what happened at a point in time.

Example:

```text
OrderCreated
```

should not later be mutated into:

```text
OrderCancelled
```

Instead publish:

```text
OrderCancelled
```

as a new event.

---

# 99. Event Replay Consideration

If consumers can replay historical events:

```text
Old event
 ↓
New consumer
```

the event contract needs to remain understandable over time.

This makes schema evolution particularly important.

---

# 100. Interview Question

### "What is Kafka?"

Answer:

> "Kafka is a distributed event streaming platform that stores records in partitioned topics. Producers publish records and consumers read them, typically through consumer groups. Kafka provides durable storage, horizontal scalability, replay capabilities and multiple independent consumers."

---

# 101. Interview Question

### "What is a topic?"

Answer:

> "A topic is a named stream or category of Kafka records. Topics are divided into partitions for scalability and parallel processing."

---

# 102. Interview Question

### "What is a partition?"

Answer:

> "A partition is an ordered sequence of records within a Kafka topic. Partitions provide scalability and parallelism, while ordering is guaranteed within each partition."

---

# 103. Interview Question

### "What is a consumer group?"

Answer:

> "A consumer group is a set of consumers that cooperate to process a topic's partitions. Within a group, a partition is assigned to one consumer at a time, allowing the workload to scale horizontally."

---

# 104. Interview Question

### "Can two consumer groups consume the same topic?"

Answer:

> "Yes. Each consumer group maintains its own offsets, so different applications can independently consume the same topic."

---

# 105. Interview Question

### "What is an offset?"

Answer:

> "An offset identifies a record's position within a Kafka partition. Consumers use offsets to track their processing progress."

---

# 106. Interview Question

### "How does Kafka guarantee ordering?"

Answer:

> "Kafka guarantees ordering within a partition, not globally across all partitions. If events for the same business entity need ordering, I would normally use a stable key such as orderId so those records are routed consistently to the same partition."

---

# 107. Interview Question

### "What is consumer lag?"

Answer:

> "Consumer lag represents how far a consumer group is behind the latest available records. High lag can indicate insufficient consumer capacity, slow processing, downstream problems or traffic spikes."

---

# 108. Interview Question

### "How do you handle duplicate Kafka messages?"

Answer:

> "I'd design consumers to be idempotent. For example, I can persist a unique event ID with a database constraint and ignore an event if it has already been successfully processed."

---

# 109. Interview Question

### "What is a dead-letter topic?"

Answer:

> "It's a separate topic where messages that repeatedly fail processing can be routed after bounded retries, allowing the main consumer flow to continue while the problematic messages are investigated or reprocessed."

---

# 110. Interview Question

### "Why use the transactional outbox?"

Answer:

> "It prevents the dual-write problem between a business database and an event broker. The service writes its business change and an outbox record in the same database transaction. A separate publisher then sends the outbox event to Kafka. Consumers still need idempotency because publishing can be retried."

---

# 111. Interview Question

### "Why not directly publish to Kafka after saving to the database?"

Answer:

> "Because the database write and Kafka publish are separate operations. If the database succeeds and Kafka fails, the business event can be lost. The outbox pattern records the event transactionally with the business data."

---

# 112. Interview Question

### "What if Kafka publish succeeds but the service crashes before marking the outbox row processed?"

Answer:

> "The event may be published again after recovery. Therefore the consumer should be idempotent. This is a common trade-off in reliable event publishing."

---

# 113. Interview Question

### "Kafka or REST for OrderCreated?"

Answer:

> "If other services need to react asynchronously to the order being created, I'd publish an OrderCreated event to Kafka. The initial client request itself can still use REST. REST and Kafka are complementary rather than mutually exclusive."

---

# 114. Interview Question

### "Kafka or RabbitMQ?"

Answer:

> "I'd choose based on the messaging requirements. Kafka is particularly strong for high-throughput event streaming, durable logs, replay and multiple consumer groups. RabbitMQ is often a good fit for traditional queues, routing and task distribution."

---

# 115. Interview Scenario

### "Inventory consumer is slower than Order Service."

Possible architecture:

```text
Order Service
 ↓
Kafka
 ↓
Inventory Consumer
```

Kafka buffers events.

Monitor:

```text
Consumer lag
```

Then:

```text
Scale consumers
Optimize processing
Increase partition parallelism if needed
Investigate downstream bottlenecks
```

---

# 116. Interview Scenario

### "Consumer keeps failing on one event."

Answer:

> "I'd avoid retrying indefinitely. I'd use bounded retries and then move the problematic event to a dead-letter topic or retry topic. I'd monitor the failure and investigate whether the issue is bad data, a schema problem or a temporary dependency failure."

---

# 117. Interview Scenario

### "Order events must remain ordered."

Answer:

> "I'd partition the topic using orderId as the key so events for a given order are consistently routed to the same partition. I'd also make sure the consumer processing model preserves the required ordering."

---

# 118. Interview Scenario

### "You need every service to receive OrderCreated."

Answer:

> "I'd create separate consumer groups for each independent application. For example, notification-group, analytics-group and inventory-group. Each group receives the topic independently while consumers within a group share the workload."

---

# 119. Interview Scenario

### "Kafka is temporarily unavailable when an order is created."

Answer:

> "For a business-critical event, I'd use a transactional outbox so the event is stored durably with the order transaction. The publisher can retry sending it to Kafka when Kafka becomes available."

---

# 120. Interview Scenario

### "Can we guarantee exactly-once payment with Kafka?"

Answer:

> "Kafka's exactly-once processing capabilities don't automatically make an external payment operation exactly once. The payment side effect still needs appropriate idempotency and provider-level guarantees."

---

# 121. Common Kafka Mistakes

```text
❌ Treating Kafka as just a simple queue
❌ Assuming global ordering
❌ Ignoring consumer lag
❌ No idempotent consumers
❌ Infinite retries
❌ No dead-letter strategy
❌ Huge messages
❌ Breaking event schemas
❌ Direct database sharing between services
❌ Assuming Kafka automatically gives exactly-once business effects
❌ Publishing DB changes without considering dual-write failure
```

---

# 122. Practical Event-Driven Checklist

When designing a Kafka workflow, ask:

```text
1. What event happened?
2. Who owns the event?
3. What topic should contain it?
4. What key determines ordering?
5. How many partitions?
6. Which consumers need it?
7. What consumer groups are required?
8. What delivery semantics are acceptable?
9. How is duplicate processing handled?
10. What happens after repeated failure?
11. How is the event schema versioned?
12. Is the DB/event dual-write problem handled?
13. How will lag be monitored?
14. How will events be traced?
15. Can the event be replayed safely?
```

---

# 123. Final Mental Model

Remember:

```text
Producer
→ Publishes records

Topic
→ Named stream

Partition
→ Scalable ordered sequence

Broker
→ Kafka server

Consumer
→ Reads records

Consumer Group
→ Shares work

Offset
→ Consumer position

Key
→ Helps determine partitioning/order

Replication
→ Fault tolerance

Lag
→ Consumer behind producer

DLT
→ Failed records

Outbox
→ Reliable DB + event publishing

Idempotency
→ Safe duplicate processing
```

---

# 124. Final Interview Answer

If asked:

> "Explain how you would use Kafka in an e-commerce microservices architecture."

Use:

> "I'd use Kafka for asynchronous domain events such as OrderCreated, PaymentCompleted and InventoryReserved. The owning service publishes the event to a partitioned topic, and independent services consume it using separate consumer groups. I'd choose a key such as orderId when ordering per order is required. Consumers would be idempotent because at-least-once processing can produce duplicates, and I'd use bounded retries with a dead-letter topic for repeatedly failing events. For reliable publishing together with database changes, I'd consider the transactional outbox pattern. I'd also monitor consumer lag, processing latency and failures."

---

# 125. Revision Checklist

```text
□ Event-driven architecture
□ Event
□ Command
□ Kafka
□ Broker
□ Cluster
□ Topic
□ Partition
□ Producer
□ Consumer
□ Consumer group
□ Offset
□ Offset commit
□ At-most-once
□ At-least-once
□ Exactly-once nuance
□ Idempotent consumer
□ Event ID
□ Event schema
□ Schema Registry
□ Schema evolution
□ Replication
□ Leader
□ ISR
□ Replication factor
□ Producer acknowledgments
□ Producer idempotence
□ Retention
□ Replay
□ Consumer lag
□ Consumer scaling
□ Rebalancing
□ Poison messages
□ Dead-letter topic
□ Retry topic
□ Ordering
□ Partition keys
□ Kafka vs RabbitMQ
□ Kafka vs REST
□ Dual-write problem
□ Transactional outbox
□ Eventual consistency
□ Saga
□ Orchestration
□ Choreography
□ Event ownership
□ Event immutability
□ Observability
□ Interview scenarios
```

---

# 126. The Interviewer's Real Test

If asked:

> "What happens when Order Service creates an order?"

Don't just say:

```text
Publish to Kafka.
```

Walk through the architecture:

```text
Client
 ↓
Order Service
 ↓
DB transaction
 ├── Order
 └── Outbox event
 ↓
Commit
 ↓
Outbox publisher
 ↓
Kafka
 ↓
OrderCreated
 ├── Inventory group
 ├── Notification group
 └── Analytics group
 ↓
Consumers process independently
 ↓
Duplicate protection
 ↓
Retries / DLT on failures
 ↓
Lag + tracing + metrics
```

That is the level of reasoning expected from a Java backend developer working with microservices.
