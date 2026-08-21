# System Design — File 09: Messaging, Queues & Kafka

Messaging is used when one part of a system needs to communicate with another part without requiring everything to happen inside the same synchronous request.

A simple synchronous flow:

```text
Client
  ↓
Order Service
  ↓
Payment Service
  ↓
Notification Service
  ↓
Response
```

This creates tight coupling.

With asynchronous messaging:

```text
Client
  ↓
Order Service
  ↓
Message Broker
  ↓
Consumers
 ├── Payment
 ├── Notification
 └── Analytics
```

The producer can continue without waiting for every consumer to finish.

---

# 1. Why Use Messaging?

Messaging can provide:

```text
Decoupling
Asynchronous processing
Load smoothing
Independent scaling
Retry capability
Event-driven architecture
Better resilience
```

---

# 2. Synchronous vs Asynchronous

### Synchronous

```text
Service A
   ↓
Service B
   ↓
Response
```

Service A waits.

### Asynchronous

```text
Service A
   ↓
Message Broker
   ↓
Service B
```

Service A doesn't necessarily wait for Service B to finish.

---

# 3. Example

An e-commerce order is created.

Synchronous:

```text
Create Order
 ↓
Send Email
 ↓
Update Analytics
 ↓
Notify Warehouse
 ↓
Return response
```

This can make checkout slow.

Asynchronous:

```text
Create Order
 ↓
Publish OrderCreated
 ↓
Return response

OrderCreated
 ├── Email Service
 ├── Inventory Service
 ├── Analytics Service
 └── Shipping Service
```

---

# 4. Message Broker

A message broker sits between producers and consumers.

```text
Producer
   ↓
Broker
   ↓
Consumer
```

Examples:

```text
Kafka
RabbitMQ
Amazon SQS
Azure Service Bus
Google Pub/Sub
```

Different brokers have different semantics.

---

# 5. Producer

The producer creates and sends a message.

Example:

```text
Order Service
```

publishes:

```text
OrderCreated
```

---

# 6. Consumer

A consumer receives and processes messages.

Example:

```text
Notification Service
```

consumes:

```text
OrderCreated
```

and sends an email.

---

# 7. Queue

A queue generally represents work waiting to be processed.

```text
Producer
   ↓
Queue
 ↓ ↓ ↓
Worker Worker Worker
```

Workers consume tasks.

---

# 8. Queue Use Case

Suppose:

```text
10,000 emails
```

need to be sent.

Instead of sending all emails during API requests:

```text
API
 ↓
Queue
 ↓
Email Workers
```

The workers process the jobs at a controlled rate.

---

# 9. Load Smoothing

Suppose traffic spikes:

```text
Normal:
1,000 jobs/min

Peak:
50,000 jobs/min
```

A queue can absorb the temporary spike:

```text
Producer
   ↓
Queue
   ↓
Workers
```

Workers process the backlog as capacity allows.

This is called:

```text
Load smoothing
```

---

# 10. Backpressure

If consumers cannot keep up:

```text
Incoming rate > Processing rate
```

the queue grows.

This gives the system a way to observe and manage overload.

Possible actions:

```text
Scale consumers
Rate-limit producers
Reject low-priority work
Increase processing capacity
```

---

# 11. Queue Depth

Important metric:

```text
Queue depth
```

Example:

```text
10,000 messages waiting
```

If queue depth continuously grows:

```text
Producer rate > Consumer rate
```

The system has a processing-capacity problem.

---

# 12. Consumer Lag

For Kafka, an important metric is:

```text
Consumer lag
```

It represents how far a consumer is behind the available records in its partitions.

High lag can indicate:

```text
Slow consumers
Traffic spikes
Insufficient consumer instances
Downstream bottlenecks
```

---

# 13. Queue vs Event Stream

This distinction matters.

### Work queue

Usually:

```text
Message
 ↓
One worker processes it
```

### Event stream

Often:

```text
Event
 ↓
Multiple independent consumers
```

Kafka is commonly used as an event streaming platform.

---

# 14. RabbitMQ vs Kafka — High Level

### RabbitMQ

Often strong for:

```text
Work queues
Routing
Task distribution
Traditional messaging patterns
```

### Kafka

Strong for:

```text
Event streaming
High-throughput pipelines
Durable event logs
Multiple independent consumers
Replay
```

The exact choice depends on requirements.

---

# 15. Kafka

Kafka is a distributed event streaming platform.

A simplified architecture:

```text
Producer
   ↓
Kafka Topic
   ↓
Consumers
```

Kafka stores records in partitions.

---

# 16. Kafka Topic

A topic is a named stream of records.

Example:

```text
order-events
```

could contain:

```text
OrderCreated
OrderPaid
OrderShipped
OrderCancelled
```

---

# 17. Kafka Partition

A topic is divided into partitions.

```text
order-events
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

# 18. Ordering

Kafka guarantees ordering within a partition.

Example:

```text
Partition 0:

OrderCreated
OrderPaid
OrderShipped
```

The order is preserved within that partition.

Do not assume global ordering across all partitions.

---

# 19. Message Key

A producer can provide a key.

Example:

```text
key = orderId
```

Kafka can use the key to determine a partition according to the producer's partitioning strategy.

This can keep events for the same order in the same partition.

---

# 20. Why Key by Order ID?

Suppose:

```text
Order 101
```

events:

```text
OrderCreated
OrderPaid
OrderShipped
```

If all go to the same partition:

```text
Partition 2

Created
Paid
Shipped
```

their ordering is preserved within that partition.

---

# 21. Consumer Group

A consumer group is a set of consumers working together.

Example:

```text
Topic:
order-events
```

Consumer group:

```text
Payment Service
 ├── Consumer 1
 ├── Consumer 2
 └── Consumer 3
```

Partitions are assigned among consumers in the group.

---

# 22. Consumer Group Parallelism

Suppose:

```text
Topic = 6 partitions
```

and:

```text
Consumer group = 3 consumers
```

Each consumer can process multiple partitions.

Conceptually:

```text
C1 → P0, P1
C2 → P2, P3
C3 → P4, P5
```

---

# 23. Too Many Consumers

Suppose:

```text
3 partitions
10 consumers
```

Only up to the available partitions can be actively assigned within that consumer group.

Some consumers may sit idle.

Therefore:

```text
Maximum active consumer parallelism
≈ number of partitions
```

for a consumer group.

---

# 24. Multiple Consumer Groups

Suppose:

```text
order-events
```

needs to be consumed by:

```text
Payment Service
Analytics Service
Notification Service
```

Use separate consumer groups:

```text
Payment Group
Analytics Group
Notification Group
```

Each group independently processes the topic.

---

# 25. Kafka Retention

Kafka can retain records after a consumer has processed them.

Example:

```text
Retention = 7 days
```

A consumer can potentially replay records from the retained log.

This is a major difference from many traditional work queues.

---

# 26. Replay

Suppose a bug affected:

```text
Analytics Service
```

You can potentially:

```text
Fix consumer
 ↓
Reset/reposition offset
 ↓
Replay events
```

This makes Kafka useful for event-driven architectures.

---

# 27. Offset

A Kafka consumer tracks its position in a partition using an offset.

Conceptually:

```text
Partition:

0 1 2 3 4 5 6 7
          ↑
        offset
```

The consumer can continue from its stored position.

---

# 28. Consumer Commit

A consumer commits its progress.

Conceptually:

```text
Processed message
      ↓
Commit offset
```

If the application crashes before committing:

```text
Message may be processed again
```

This is why consumers should generally be designed for idempotency.

---

# 29. At-Most-Once

Conceptually:

```text
Process
 ↓
Commit
 ↓
Possible failure
```

A message may be lost if the system fails at the wrong point.

Potential benefit:

```text
Lower duplicate-processing risk
```

Trade-off:

```text
Possible message loss
```

---

# 30. At-Least-Once

Conceptually:

```text
Process
 ↓
Failure before commit
 ↓
Message delivered again
```

Potential result:

```text
Duplicate processing
```

Kafka consumers are commonly designed around at-least-once processing patterns.

---

# 31. Exactly-Once

Exactly-once semantics are more complicated than:

```text
"Message runs exactly once."
```

They depend on the complete processing pipeline.

Kafka supports exactly-once processing features for certain Kafka-based workflows, but external side effects still require careful design.

---

# 32. Idempotency

An operation is idempotent if repeating it produces the same intended result.

Example:

```text
Set order status = PAID
```

Running it twice:

```text
PAID
```

is still:

```text
PAID
```

---

# 33. Non-Idempotent Operation

Example:

```text
Add ₹100 to account
```

Processing twice:

```text
+₹200
```

This can be dangerous.

---

# 34. Idempotency Key

A message can include:

```text
eventId
```

or:

```text
idempotencyKey
```

Consumer stores processed IDs:

```text
event-123 → processed
```

If the same message arrives again:

```text
event-123
 ↓
Already processed
 ↓
Skip
```

---

# 35. Consumer Retry

If processing fails:

```text
Message
 ↓
Consumer
 ↓
Failure
```

Possible approaches:

```text
Retry
Delayed retry
Dead-letter queue
Alert
```

---

# 36. Retry Problem

Blind retries can make things worse.

Suppose:

```text
Payment service is down
```

and every consumer retries immediately:

```text
retry
retry
retry
retry
```

This can create:

```text
Retry storm
```

---

# 37. Exponential Backoff

Instead of retrying immediately:

```text
1 sec
2 sec
4 sec
8 sec
16 sec
```

This reduces pressure on a failing dependency.

Often combine with:

```text
Jitter
```

to prevent synchronized retries.

---

# 38. Jitter

Without jitter:

```text
1000 consumers
retry at exactly 10 sec
```

They can create another traffic spike.

With jitter:

```text
10–15 sec
```

retries are spread out.

---

# 39. Dead Letter Queue

A DLQ stores messages that repeatedly fail processing.

Flow:

```text
Message
 ↓
Consumer
 ↓
Failure
 ↓
Retry
 ↓
Failure
 ↓
Retry limit reached
 ↓
DLQ
```

Operators can investigate later.

---

# 40. DLQ Is Not a Trash Can

A DLQ should be monitored.

Track:

```text
DLQ message count
Reason
Age
Consumer
Failure type
```

A growing DLQ indicates a problem.

---

# 41. Poison Message

A poison message is a message that repeatedly causes processing failure.

Example:

```text
Malformed payload
```

Without a DLQ:

```text
Consumer
 ↓
Failure
 ↓
Retry
 ↓
Failure
 ↓
Retry forever
```

A DLQ prevents endless retries.

---

# 42. Ordering vs Parallelism

If you require strict ordering:

```text
Same partition
```

But increasing partitions can improve parallelism:

```text
More partitions
→ More consumers
→ More throughput
```

There is a trade-off.

---

# 43. Partition Count

More partitions can provide:

```text
More parallelism
```

But also:

```text
More metadata
More operational overhead
More complexity
```

Choose based on expected throughput and scaling needs.

---

# 44. Kafka Brokers

Kafka runs as a distributed cluster.

```text
Broker 1
Broker 2
Broker 3
```

Partitions are distributed across brokers.

---

# 45. Kafka Replication

A partition can have replicas.

Example:

```text
Partition 0
 ├── Broker 1 → Leader
 ├── Broker 2 → Replica
 └── Broker 3 → Replica
```

This provides redundancy.

---

# 46. Kafka Leader

Each partition has a leader.

Producers and consumers generally interact with the partition leader for normal operations.

If the leader fails:

```text
Replica
 ↓
New leader
```

can be selected.

---

# 47. Kafka Consumer Lag

Suppose:

```text
Producer:
10,000 events/sec

Consumer:
8,000 events/sec
```

Lag grows:

```text
2,000 events/sec backlog
```

If sustained:

```text
Consumer capacity < producer rate
```

---

# 48. Scaling Consumers

If topic has:

```text
12 partitions
```

you can scale a consumer group to:

```text
12 active consumers
```

assuming workload and assignment allow it.

Adding:

```text
20 consumers
```

doesn't create 20-way partition parallelism for that group.

---

# 49. Scaling Producers

Producers can scale horizontally:

```text
Producer 1
Producer 2
Producer 3
```

They publish to:

```text
Kafka
```

Partitioning distributes records.

---

# 50. Kafka Durability

Kafka stores records on disk and can replicate them.

Durability depends on:

```text
Replication factor
Acknowledgement configuration
Broker health
Storage
```

---

# 51. Producer Acknowledgements

A producer can configure how much acknowledgement it expects.

Conceptually:

```text
acks = 0
→ no broker acknowledgement

acks = 1
→ leader acknowledgement

acks = all
→ strongest configured acknowledgement from in-sync replicas
```

Exact behavior depends on Kafka configuration/version.

---

# 52. `acks=all`

This generally provides stronger durability than:

```text
acks=1
```

but may increase latency.

It should be combined with appropriate replication and in-sync replica settings.

---

# 53. Message Delivery Architecture

Example:

```text
Order Service
      ↓
Kafka
      ↓
order-events
   /    |    \
  ↓     ↓     ↓
Payment Email Analytics
```

Each consumer group independently receives the event stream.

---

# 54. Event vs Command

### Event

Describes something that happened.

```text
OrderCreated
PaymentCompleted
```

### Command

Requests an action.

```text
ProcessPayment
SendEmail
ReserveInventory
```

This distinction can make event-driven architectures easier to reason about.

---

# 55. Event Schema

Example:

```json
{
  "eventId": "evt-123",
  "eventType": "OrderCreated",
  "orderId": 101,
  "userId": 42,
  "timestamp": "2026-08-21T10:00:00Z"
}
```

Good events usually contain enough information for consumers to process them reliably.

---

# 56. Schema Evolution

Events live longer than application deployments.

Suppose version 1:

```json
{
  "orderId": 101,
  "userId": 42
}
```

Version 2:

```json
{
  "orderId": 101,
  "userId": 42,
  "currency": "INR"
}
```

Consumers should handle compatible evolution.

---

# 57. Backward Compatibility

When adding fields:

```text
Prefer optional/additive changes
```

Avoid breaking old consumers unexpectedly.

For larger Kafka systems, schema registries and formats such as:

```text
Avro
Protobuf
JSON Schema
```

can help manage event contracts.

---

# 58. Event Versioning

Possible approaches:

```text
Schema version field
Topic version
Backward-compatible schema evolution
```

Avoid creating dozens of topics without a clear reason.

---

# 59. Message Size

Large messages create:

```text
Network overhead
Memory pressure
Serialization cost
Broker storage pressure
```

Don't use Kafka as a replacement for object storage.

For large files:

```text
Object Storage
```

and put a reference in the event:

```json
{
  "fileKey": "uploads/12345.pdf"
}
```

---

# 60. Ordering Requirements

Ask:

```text
Do we need ordering?
Global ordering?
Per-user ordering?
Per-order ordering?
```

Often:

```text
Per-entity ordering
```

is enough.

Example:

```text
key = orderId
```

---

# 61. Duplicate Messages

Consumers should assume:

```text
Duplicate delivery can happen
```

Use:

```text
Idempotency
Unique constraints
Processed-event table
State checks
```

depending on the workflow.

---

# 62. Transactional Outbox

Classic problem:

```text
DB transaction
+
Kafka publish
```

If DB commits:

```text
Order created
```

but Kafka publish fails:

```text
Event missing
```

---

# 63. Outbox Solution

Store the event in the same DB transaction:

```text
Transaction
 ├── orders
 └── outbox_events
```

Then:

```text
Outbox Publisher
       ↓
Kafka
```

This reduces the risk of:

```text
DB updated
but event missing
```

---

# 64. Outbox Flow

```text
Request
  ↓
Order Service
  ↓
DB Transaction
 ├── Insert Order
 └── Insert Outbox Event
  ↓
Commit
  ↓
Outbox Publisher
  ↓
Kafka
```

---

# 65. Outbox Failure

If Kafka is unavailable:

```text
Outbox Event
```

remains in the database.

The publisher can retry later.

This provides durability for the event publication workflow.

---

# 66. Inbox Pattern

On the consumer side, an inbox/processed-event record can help deduplicate messages.

Example:

```text
eventId = evt-123
```

Consumer stores:

```text
evt-123 → processed
```

A duplicate is ignored.

---

# 67. Outbox + Inbox

A robust workflow can be:

```text
Producer:
DB + Outbox
      ↓
Kafka
      ↓
Consumer
      ↓
Inbox / processed-event record
      ↓
Business DB
```

This is useful for reliable event-driven processing.

---

# 68. Saga

A Saga manages a distributed business workflow using multiple local transactions.

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
 ↓
Cancel Order
```

These are compensating actions.

---

# 69. Why Saga?

Because:

```text
Order DB
Inventory DB
Payment DB
```

may be separate.

A single traditional database transaction cannot cover all of them.

---

# 70. Choreography

Services react to events.

```text
OrderCreated
   ↓
Inventory Service
   ↓
InventoryReserved
   ↓
Payment Service
   ↓
PaymentCompleted
```

No central coordinator.

---

# 71. Orchestration

A central orchestrator coordinates the workflow.

```text
Saga Orchestrator
   ├── Reserve Inventory
   ├── Process Payment
   └── Create Shipment
```

The orchestrator tracks the state.

---

# 72. Choreography vs Orchestration

### Choreography

```text
+ Loosely coupled
+ No central coordinator
- Harder to understand as workflows grow
```

### Orchestration

```text
+ Central visibility
+ Easier complex workflow management
- Orchestrator adds complexity/dependency
```

---

# 73. Queue Backlog

Suppose:

```text
Queue depth:
1K
2K
5K
10K
20K
```

This trend matters more than one isolated number.

A growing backlog indicates:

```text
Processing capacity problem
```

---

# 74. Consumer Autoscaling

Consumers can scale based on:

```text
Queue depth
Kafka lag
CPU
Processing latency
```

Example:

```text
Lag > threshold
 ↓
Add consumers
```

But ensure there are enough partitions to benefit from more Kafka consumers.

---

# 75. Dead Letter Monitoring

Track:

```text
DLQ size
Failure rate
Top error reasons
Oldest message age
```

A DLQ that grows silently is dangerous.

---

# 76. Retry Strategy

A good retry strategy may include:

```text
Limited attempts
Exponential backoff
Jitter
Dead-letter queue
Alerting
```

Avoid:

```text
Infinite immediate retries
```

---

# 77. Messaging and Database Transactions

Don't hold a database transaction while waiting for an external consumer.

Avoid:

```text
BEGIN
 ↓
Publish message
 ↓
Wait for consumer
 ↓
Commit
```

Prefer short local transactions and asynchronous coordination.

---

# 78. Kafka vs Redis Streams

Redis Streams:

```text
Useful for certain lightweight stream/message workloads
```

Kafka:

```text
Designed for high-throughput distributed event streaming
Long retention
Replay
Large-scale partitioning
```

Choose based on requirements.

---

# 79. Kafka vs RabbitMQ

| Requirement | Often consider |
|---|---|
| Traditional work queue | RabbitMQ |
| Complex routing | RabbitMQ |
| Large event stream | Kafka |
| Replay retained events | Kafka |
| Multiple independent consumers | Kafka |
| Task distribution | RabbitMQ |
| High-throughput event pipeline | Kafka |

This is a high-level comparison, not a strict rule.

---

# 80. SQS-Style Queue vs Kafka

A managed queue such as:

```text
Amazon SQS
```

can be excellent when you need:

```text
Simple asynchronous jobs
Managed infrastructure
Automatic scaling
```

Kafka may be more appropriate when you need:

```text
Durable event log
Replay
Partitioned streams
Multiple consumer groups
High-throughput event processing
```

---

# 81. Messaging Failure Scenario

### Consumer crashes after processing but before committing.

Result:

```text
Message delivered again
```

Therefore:

```text
Consumer must tolerate duplicates
```

Use idempotency.

---

# 82. Messaging Failure Scenario

### Kafka broker fails.

With appropriate replication:

```text
Replica
 ↓
New leader
```

The system can continue after failover.

Exact availability depends on:

```text
Replication
In-sync replicas
Cluster health
Producer settings
Consumer behavior
```

---

# 83. Messaging Failure Scenario

### Consumer is slower than producer.

Result:

```text
Consumer lag grows
```

Solutions:

```text
Scale consumers
Optimize processing
Increase partitions if needed
Batch processing
Reduce unnecessary work
```

---

# 84. Messaging Failure Scenario

### Consumer repeatedly fails one message.

Result:

```text
Poison message
```

Solution:

```text
Retry with limits
 ↓
DLQ
```

Then investigate the root cause.

---

# 85. Messaging Failure Scenario

### Kafka event published but database transaction fails.

You may have:

```text
Event says OrderCreated
DB has no order
```

This is a consistency problem.

Possible solution:

```text
Transactional Outbox
```

---

# 86. Messaging Failure Scenario

### Database succeeds but event publication fails.

You may have:

```text
Order exists
Event missing
```

Again:

```text
Outbox
```

can solve this class of problem.

---

# 87. E-commerce Architecture

A scalable order flow:

```text
Client
  ↓
API
  ↓
Order Service
  ↓
MySQL Transaction
 ├── Order
 └── Outbox Event
  ↓
Kafka
  ├── Inventory Service
  ├── Payment Service
  ├── Notification Service
  └── Analytics Service
```

This separates:

```text
Core transaction
```

from:

```text
Asynchronous side effects
```

---

# 88. Order Event Example

```json
{
  "eventId": "evt-1001",
  "type": "OrderCreated",
  "orderId": 5001,
  "userId": 42,
  "timestamp": "2026-08-21T10:30:00Z"
}
```

Partition key:

```text
orderId
```

This can preserve ordering for events belonging to the same order.

---

# 89. Notification Service

The notification service can consume:

```text
OrderCreated
OrderPaid
OrderShipped
```

It doesn't need the Order Service to synchronously call it.

Benefits:

```text
Loose coupling
Independent scaling
Retry
Failure isolation
```

---

# 90. Analytics Service

Analytics can independently consume:

```text
OrderCreated
PaymentCompleted
OrderShipped
```

If analytics is temporarily unavailable:

```text
Kafka retains events
```

and the consumer can catch up later, subject to retention.

---

# 91. Inventory Service

Inventory can consume:

```text
OrderCreated
```

and attempt:

```text
Reserve inventory
```

If the operation fails:

```text
InventoryReservationFailed
```

can trigger compensating workflow.

---

# 92. Payment Service

Payment processing should be carefully designed for:

```text
Idempotency
Retries
Timeouts
Duplicate events
Provider failures
```

Never assume a message is processed only once.

---

# 93. Event-Driven Architecture Benefits

```text
Loose coupling
Independent scaling
Async processing
Failure isolation
Replay
Extensibility
```

---

# 94. Event-Driven Architecture Costs

```text
Eventual consistency
Debugging complexity
Duplicate processing
Ordering challenges
Schema evolution
Operational complexity
Harder local development
```

Do not use Kafka just because:

```text
"Microservices need Kafka."
```

Use it when asynchronous/event-driven behavior is actually useful.

---

# 95. Interview Question

### Why use Kafka?

Answer:

> "I'd use Kafka when I need a durable, high-throughput event stream with multiple independent consumers, partition-based scaling and the ability to replay retained events."

---

# 96. Interview Question

### What is a Kafka topic?

Answer:

> "A topic is a named stream of records. It is divided into partitions, which provide parallelism and ordering within each partition."

---

# 97. Interview Question

### What is a Kafka partition?

Answer:

> "A partition is an ordered log of records within a topic. Kafka distributes partitions across brokers, and consumers process partitions to achieve parallelism."

---

# 98. Interview Question

### What is a consumer group?

Answer:

> "A consumer group is a set of consumers that collectively process a topic. Within the group, a partition is normally assigned to one consumer at a time."

---

# 99. Interview Question

### What is consumer lag?

Answer:

> "Consumer lag represents how far a consumer group is behind the records available in Kafka. Increasing lag usually indicates that consumers aren't keeping up with the incoming workload."

---

# 100. Interview Question

### How do you handle duplicate messages?

Answer:

> "I design consumers to be idempotent using an event ID, unique constraints, processed-event tracking or state checks, depending on the workflow."

---

# 101. Interview Question

### What is a dead-letter queue?

Answer:

> "It's a separate destination for messages that repeatedly fail processing after the allowed retry attempts, so they don't block normal processing and can be investigated separately."

---

# 102. Interview Question

### What is the outbox pattern?

Answer:

> "The outbox pattern stores the business change and the event record in the same database transaction. A separate publisher then sends the event to Kafka, reducing the risk that the database changes but the event is lost."

---

# 103. Interview Question

### Kafka vs RabbitMQ?

Answer:

> "I'd generally consider RabbitMQ for traditional work queues and routing-heavy messaging, while Kafka is a stronger fit for durable high-throughput event streams, replay and multiple independent consumer groups."

---

# 104. Interview Question

### What happens if a Kafka consumer crashes?

Answer:

> "Another consumer in the group can take over its partitions. Depending on where the offset was committed, some messages may be processed again, so consumers should be idempotent."

---

# 105. Interview Question

### How do you preserve order in Kafka?

Answer:

> "Kafka preserves order within a partition. If events for the same entity must remain ordered, I would use a stable key such as orderId so those events are routed to the same partition."

---

# 106. Interview Question

### How do you scale Kafka consumers?

Answer:

> "Increase the number of consumers in the consumer group, but the useful parallelism is bounded by the number of partitions. If necessary, increase partitions while considering ordering and operational implications."

---

# 107. Interview Question

### What is a retry storm?

Answer:

> "It's when many failed requests or messages are retried aggressively at the same time, creating additional load on an already unhealthy dependency. Exponential backoff and jitter help prevent it."

---

# 108. Interview Question

### What is a poison message?

Answer:

> "It's a message that repeatedly fails processing, often because of invalid data or a deterministic consumer bug. Limited retries followed by a DLQ prevent it from blocking the pipeline."

---

# 109. Final Checklist

You should be able to explain:

```text
□ Synchronous vs asynchronous
□ Message broker
□ Producer
□ Consumer
□ Queue
□ Load smoothing
□ Backpressure
□ Queue depth
□ Kafka
□ Topic
□ Partition
□ Ordering
□ Message key
□ Consumer group
□ Offset
□ Consumer lag
□ Retention
□ Replay
□ At-most-once
□ At-least-once
□ Exactly-once concepts
□ Idempotency
□ Retry
□ Exponential backoff
□ Jitter
□ DLQ
□ Poison messages
□ Kafka brokers
□ Replication
□ Producer acknowledgements
□ Event vs command
□ Schema evolution
□ Outbox pattern
□ Inbox pattern
□ Saga
□ Choreography
□ Orchestration
□ Kafka vs RabbitMQ
□ Kafka vs Redis Streams
□ Queue vs event stream
```

---

# 110. One-Minute Interview Answer

### "How would you use Kafka in an e-commerce system?"

> "I'd use Kafka for asynchronous events such as OrderCreated, PaymentCompleted and OrderShipped. The Order Service would first commit the order and an outbox event in one database transaction, and an outbox publisher would send that event to Kafka. Different consumer groups could independently process the event for inventory, notifications and analytics. I'd partition by orderId where per-order ordering matters, make consumers idempotent because duplicate processing can happen, and monitor consumer lag and DLQ depth."

---

# 111. Key Takeaway

> **Messaging separates producers from consumers and lets systems process work asynchronously. Kafka adds durable, partitioned event streams, consumer groups and replay. The real engineering challenges are delivery guarantees, ordering, retries, idempotency, backpressure and consistency—not simply publishing a message.**

**File 09 complete.**
