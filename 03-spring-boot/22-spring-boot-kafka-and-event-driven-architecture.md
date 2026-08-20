# Spring Boot — Kafka and Event-Driven Architecture

This file covers Kafka and event-driven architecture concepts that are important for Java/Spring Boot backend interviews and real-world distributed systems.

The goal is to understand how to design reliable asynchronous workflows rather than simply knowing Kafka annotations.

---

# 1. What Is Event-Driven Architecture?

In an event-driven architecture, services communicate by publishing and consuming events.

Example:

```text
Order Service
     |
     | OrderCreated
     v
   Kafka
     |
 +---+---+
 |       |
 v       v
Inventory Notification
```

The producer does not need to directly call every consumer.

---

# 2. What Is an Event?

An event represents something that already happened.

Examples:

```text
OrderCreated
PaymentCompleted
ProductUpdated
UserRegistered
OrderCancelled
```

Good event names are usually past tense because they represent completed facts.

---

# 3. Command vs Event

Command:

```text
ProcessPayment
ReserveInventory
```

It asks another component to perform an action.

Event:

```text
PaymentCompleted
InventoryReserved
```

It announces that something happened.

---

# 4. Event Producer

A producer publishes messages/events.

Example:

```text
Order Service
     |
     v
OrderCreated
     |
     v
Kafka
```

The producer generally does not need to know every consumer.

---

# 5. Event Consumer

A consumer reads events and performs processing.

Example:

```text
Kafka
  |
  v
Inventory Service
  |
  v
Reserve stock
```

A consumer should be designed to handle retries and duplicate delivery safely.

---

# 6. Why Use Kafka?

Kafka is useful for:

```text
Asynchronous processing
High-throughput event streaming
Service decoupling
Event-driven workflows
Log/event retention
Data pipelines
```

It is not simply a faster REST API.

---

# 7. Kafka Core Concepts

Know these terms:

```text
Broker
Topic
Partition
Producer
Consumer
Consumer Group
Offset
Record
Key
Replication
```

---

# 8. Kafka Broker

A broker is a Kafka server that stores and serves records.

A Kafka cluster can contain:

```text
Broker 1
Broker 2
Broker 3
```

Together they provide storage and distributed processing.

---

# 9. Kafka Topic

A topic is a named stream of records.

Example:

```text
order-events
payment-events
inventory-events
```

A topic is divided into partitions.

---

# 10. Kafka Partition

Example:

```text
order-events

Partition 0
Partition 1
Partition 2
```

Partitions provide:

```text
Parallelism
Scalability
Ordering within a partition
```

---

# 11. Kafka Ordering

Kafka guarantees ordering within a partition.

It does not provide global ordering across all partitions.

If events for the same order must remain ordered, use a suitable key:

```text
key = orderId
```

This can route related records to the same partition.

---

# 12. Kafka Record

A Kafka record commonly contains:

```text
Topic
Partition
Offset
Key
Value
Timestamp
Headers
```

Example:

```json
{
  "orderId": 1001,
  "status": "CREATED"
}
```

---

# 13. Producer

A producer sends records to Kafka.

Conceptually:

```text
Spring Boot
    |
Kafka Producer
    |
    v
Kafka Topic
```

Spring Kafka provides abstractions such as:

```text
KafkaTemplate
```

---

# 14. KafkaTemplate

Example:

```java
@Service
public class OrderEventPublisher {

    private final KafkaTemplate<String, OrderCreatedEvent>
        kafkaTemplate;

    public OrderEventPublisher(
            KafkaTemplate<String, OrderCreatedEvent>
                kafkaTemplate) {

        this.kafkaTemplate = kafkaTemplate;
    }

    public void publish(
            OrderCreatedEvent event) {

        kafkaTemplate.send(
            "order-events",
            event.orderId().toString(),
            event
        );
    }
}
```

The order ID is used as the key so events for the same order can be routed consistently.

---

# 15. Consumer

Spring Kafka can consume records using:

```java
@KafkaListener(
    topics = "order-events",
    groupId = "inventory-service"
)
public void consume(
        OrderCreatedEvent event) {

    ...
}
```

The consumer processes the event.

---

# 16. Consumer Group

A consumer group allows multiple consumers to share work.

Example:

```text
Topic
P0 P1 P2 P3

Consumer A → P0 P1
Consumer B → P2 P3
```

Each partition is assigned to one consumer within a group at a time.

---

# 17. Consumer Groups Are Independent

Suppose:

```text
inventory-service
notification-service
analytics-service
```

Each uses a different group.

Then each group receives the event independently.

```text
order-events
      |
 +----+----+----+
 |         |    |
Inventory Notification Analytics
group      group       group
```

---

# 18. Same Consumer Group

If two instances use:

```text
groupId = inventory-service
```

they share the workload.

```text
Inventory Instance A
Inventory Instance B
        |
        v
order-events
```

This is useful for scaling consumers.

---

# 19. Number of Consumers vs Partitions

Suppose:

```text
4 partitions
```

and:

```text
6 consumers
```

Only up to 4 consumers can actively consume partitions in that group at the same time.

Two consumers will have no partition assigned until the assignment changes.

---

# 20. Kafka Offset

An offset identifies a record's position within a partition.

Example:

```text
0 1 2 3 4 5 6
        ^
      offset
```

Consumers use offsets to track progress.

---

# 21. Offset Commit

After processing a message, the consumer can commit its offset.

Conceptually:

```text
Read
 ↓
Process
 ↓
Commit offset
```

If the application fails before committing:

```text
Message may be processed again
```

This is one reason consumers should be idempotent.

---

# 22. At-Least-Once Processing

A common practical model is:

```text
Message may be delivered/processed more than once
```

Therefore:

```text
Consumer must tolerate duplicates
```

This is often preferable to silently losing messages.

---

# 23. At-Most-Once Processing

Conceptually:

```text
Commit
 ↓
Process
```

If the application crashes after committing but before processing:

```text
Message may be lost
```

Use this model only when that tradeoff is acceptable.

---

# 24. Exactly-Once Semantics

Exactly-once behavior is more complicated than setting one Kafka property.

You must consider:

```text
Kafka producer
Kafka transactions
Consumer offsets
Database transaction
External side effects
```

A Kafka exactly-once guarantee does not automatically make an entire distributed business workflow exactly once.

---

# 25. Idempotent Consumer

Suppose:

```text
eventId = EVT-100
```

arrives twice.

Consumer:

```text
Check EVT-100
 ↓
Already processed?
 ├─ yes → ignore/reuse result
 └─ no → process + record
```

This prevents duplicate business effects.

---

# 26. Idempotency Table

Example:

```text
processed_events

event_id
consumer_name
processed_at
```

Unique constraint:

```text
(event_id, consumer_name)
```

This can prevent the same event from being processed twice by the same consumer.

---

# 27. Kafka Message Key

The key can determine partition placement.

Example:

```text
key = orderId
```

Then:

```text
Order 100
Order 100
Order 100
```

can remain in the same partition and preserve ordering for that key.

---

# 28. Choosing a Kafka Key

Choose a key based on the entity for which ordering matters.

Examples:

```text
orderId
customerId
accountId
productId
```

Do not choose a key randomly if ordering or partition distribution matters.

---

# 29. Partitioning Tradeoff

More partitions provide:

```text
More parallelism
```

but also increase:

```text
Operational complexity
Resource usage
Consumer coordination
```

Choose partition count based on expected workload and growth.

---

# 30. Consumer Rebalancing

When consumers join or leave a consumer group, Kafka can rebalance partition assignments.

Example:

```text
Consumer A
Consumer B
```

then:

```text
Consumer C joins
```

Partitions may be reassigned.

Frequent rebalances can hurt throughput.

---

# 31. Consumer Lag

Consumer lag represents how far a consumer is behind the latest available records.

Example:

```text
Latest offset = 10,000
Consumer offset = 9,500

Lag = 500
```

High or continuously growing lag can indicate a processing bottleneck.

---

# 32. What Causes Consumer Lag?

Possible causes:

```text
Slow processing
Insufficient consumers
Too few partitions
Database bottleneck
External API latency
Large messages
CPU saturation
Downstream dependency failure
```

---

# 33. How Do You Reduce Consumer Lag?

Options:

```text
Optimize consumer processing
Increase consumer instances
Increase partitions where appropriate
Batch processing
Optimize database operations
Remove unnecessary external calls
Scale downstream dependencies
```

Don't blindly increase consumer count beyond available partition parallelism.

---

# 34. Kafka Retention

Kafka can retain records for a configured period or according to size policies.

Retention allows:

```text
Reprocessing
Replay
Recovery
Analytics
```

Kafka is therefore different from a traditional transient queue.

---

# 35. Log Compaction

Kafka log compaction keeps the latest value for a key under its compaction rules.

Useful for:

```text
Current state
Configuration
Entity snapshots
```

Example:

```text
product-100 → price=500
product-100 → price=550
product-100 → price=600
```

The compacted log can eventually retain the latest record for the key.

---

# 36. Kafka Replication

Partitions can have replicas across brokers.

Example:

```text
Partition 0
Leader → Broker 1
Replica → Broker 2
Replica → Broker 3
```

Replication improves fault tolerance.

---

# 37. Leader and Followers

For a partition:

```text
Leader
Followers
```

The leader handles writes and reads according to Kafka configuration, while replicas replicate the partition data.

If the leader fails, another suitable replica can become leader.

---

# 38. Producer Acknowledgments

Producer acknowledgments control durability tradeoffs.

Common setting:

```text
acks=0
acks=1
acks=all
```

`acks=all` generally provides stronger durability guarantees than `acks=1`, assuming appropriate replication configuration.

---

# 39. Producer Retries

Temporary broker/network failures can cause producer retries.

But retries should be combined with:

```text
Idempotence
Appropriate delivery timeout
Reasonable retry configuration
```

to avoid unintended duplicate effects.

---

# 40. Idempotent Producer

Kafka supports idempotent producer behavior.

Conceptually:

```text
Producer
 ↓
Retry
 ↓
Broker
```

The broker can prevent certain duplicate writes caused by producer retries when idempotence is configured correctly.

---

# 41. Serialization

Kafka values need serialization.

Common formats:

```text
JSON
Avro
Protobuf
String
Byte array
```

For enterprise event systems, schema-managed formats can help with compatibility and payload evolution.

---

# 42. JSON Events

Example:

```json
{
  "eventId": "evt-1",
  "orderId": 1001,
  "type": "ORDER_CREATED"
}
```

Advantages:

```text
Easy to inspect
Easy to integrate
Human-readable
```

Tradeoffs:

```text
Larger payloads
Less strict schema
Potential compatibility problems
```

---

# 43. Avro / Protobuf

Binary schema-based formats can provide:

```text
Compact messages
Explicit schemas
Schema evolution
Strong contracts
```

They are useful in large event-driven systems.

---

# 44. Schema Evolution

Suppose version 1:

```json
{
  "orderId": 100
}
```

Version 2 adds:

```json
{
  "orderId": 100,
  "customerId": 200
}
```

Adding optional fields is generally easier to make backward compatible than removing or changing existing fields.

---

# 45. Schema Registry

A schema registry can manage:

```text
Event schemas
Versions
Compatibility rules
```

This reduces accidental breaking changes between producers and consumers.

---

# 46. Kafka Headers

Headers can carry metadata.

Example:

```text
traceId
eventType
source
correlationId
```

Don't use headers as a replacement for clearly defined business event fields.

---

# 47. Spring Kafka

Spring Kafka integrates Kafka with Spring applications.

Important abstractions include:

```text
KafkaTemplate
@KafkaListener
ConsumerFactory
ProducerFactory
KafkaListenerContainerFactory
Error handlers
Transactions
```

---

# 48. @KafkaListener

Example:

```java
@KafkaListener(
    topics = "order-events",
    groupId = "inventory-service"
)
public void handle(
        OrderCreatedEvent event) {

    inventoryService.reserve(
        event.orderId()
    );
}
```

---

# 49. Listener Concurrency

Spring Kafka can run multiple listener threads.

Conceptually:

```text
Kafka Topic
P0 P1 P2

Listener Thread 1 → P0
Listener Thread 2 → P1
Listener Thread 3 → P2
```

The actual parallelism is constrained by partition count.

---

# 50. Error Handling

Consumer processing can fail because of:

```text
Invalid payload
Database failure
Network failure
Business rule
External service failure
```

A robust design needs a defined failure strategy.

---

# 51. Retry

Possible flow:

```text
Main topic
    ↓
Consumer
    ↓
Failure
    ↓
Retry
    ↓
Failure
    ↓
Retry
```

Retries should be bounded.

---

# 52. Dead Letter Topic

After repeated failures:

```text
Main Topic
    ↓
Consumer
    ↓
Retry
    ↓
Retry exhausted
    ↓
DLT
```

Operators can inspect and resolve failed messages.

---

# 53. Poison Message

A poison message repeatedly fails.

Example:

```text
Malformed event
```

If retried forever:

```text
Consumer keeps failing
```

Better:

```text
Bounded retries
↓
Dead letter handling
↓
Alert
↓
Manual investigation/replay
```

---

# 54. Retryable vs Non-Retryable Errors

Retryable:

```text
Temporary network failure
Temporary database outage
Transient service unavailable
```

Usually non-retryable:

```text
Invalid payload
Schema violation
Permanent business validation failure
```

Don't waste resources retrying permanent failures.

---

# 55. Exponential Backoff

Instead of:

```text
retry immediately
retry immediately
retry immediately
```

use:

```text
100ms
200ms
400ms
800ms
```

The exact values should be based on the dependency and SLA.

---

# 56. Jitter

If many consumers retry at exactly the same time:

```text
Service fails
 ↓
10,000 consumers retry together
 ↓
Traffic spike
```

Jitter randomizes retry timing and reduces synchronized bursts.

---

# 57. Kafka Consumer Transaction

Kafka supports transactional processing.

Conceptually:

```text
Consume
 ↓
Process
 ↓
Produce
 ↓
Commit transaction
```

This can help with Kafka-to-Kafka processing.

It does not automatically create an atomic transaction between Kafka and an arbitrary external database.

---

# 58. Kafka + Database Transaction

A common problem:

```text
Consume event
 ↓
Update MySQL
 ↓
Publish event
```

If MySQL succeeds but Kafka publishing fails:

```text
Database changed
Event missing
```

Patterns such as Outbox can solve this more reliably.

---

# 59. Outbox Pattern with Kafka

Flow:

```text
Business transaction
      |
      +--> orders
      |
      +--> outbox
              |
              v
       Outbox Publisher
              |
              v
             Kafka
```

The order and outbox record are committed together.

---

# 60. CDC and Outbox

Change Data Capture tools can publish outbox records to Kafka.

Conceptually:

```text
Application
   ↓
Outbox Table
   ↓
CDC
   ↓
Kafka
```

Debezium is a commonly used CDC technology.

---

# 61. Inbox Pattern

Consumer-side reliability can use an inbox table.

```text
Kafka
  ↓
Inbox
  ↓
Business processing
```

The event ID is stored so duplicate delivery can be detected.

---

# 62. Kafka and Idempotency

A reliable consumer often combines:

```text
Kafka at-least-once delivery
+
Idempotent business processing
+
Database uniqueness
```

This provides practical protection against duplicate events.

---

# 63. Kafka Consumer Database Example

Suppose:

```text
OrderCreated
```

Consumer:

```text
Check order already processed?
   |
   +-- yes → skip
   |
   +-- no
        ↓
    reserve inventory
        ↓
    mark event processed
```

The database constraints and transaction boundaries must be designed carefully to avoid races.

---

# 64. Kafka Event Ordering

Suppose:

```text
OrderCreated
OrderCancelled
```

If both events use:

```text
orderId
```

as the key, they can be routed to the same partition and preserve their order within that partition.

Without an appropriate key, they may land on different partitions and lose per-order ordering.

---

# 65. Kafka Duplicate Events

Possible causes:

```text
Consumer crash before offset commit
Retry
Rebalance
Producer retry
Network uncertainty
```

Therefore:

```text
Don't assume exactly-once business processing
```

Design idempotently.

---

# 66. Kafka Consumer Failure

Example:

```text
Consumer reads message
 ↓
Database update
 ↓
Application crashes
 ↓
Offset not committed
```

After restart:

```text
Message read again
```

The database operation must tolerate this scenario.

---

# 67. Kafka Consumer Rebalance During Slow Processing

If processing takes too long relative to consumer settings, the consumer can be considered unhealthy or leave the group.

Possible improvements:

```text
Optimize processing
Batch work
Tune consumer settings
Move long-running work appropriately
```

Do not simply increase timeout values without understanding the root cause.

---

# 68. Large Kafka Messages

Avoid unnecessarily large events.

Large messages can cause:

```text
Higher network usage
Memory pressure
Serialization overhead
Consumer latency
Broker storage cost
```

Prefer references to large objects when appropriate.

---

# 69. Event Payload Design

Bad:

```text
Entire customer profile
Entire order history
Huge product catalog
```

Better:

```json
{
  "eventId": "evt-100",
  "orderId": 500,
  "customerId": 101
}
```

Consumers can fetch additional data if necessary, but avoid turning every event into a chain of synchronous calls.

---

# 70. Event Should Represent a Business Fact

Good:

```text
OrderCreated
PaymentCompleted
InventoryReserved
```

Less useful:

```text
OrderServiceMethodExecuted
```

Events should be meaningful to the domain.

---

# 71. Event Naming

Use consistent names:

```text
OrderCreated
OrderPaid
OrderCancelled
InventoryReserved
InventoryReleased
```

Avoid ambiguous names such as:

```text
OrderEvent
DataChanged
UpdateMessage
```

---

# 72. Event Metadata

Useful metadata:

```text
eventId
eventType
aggregateId
occurredAt
producer
schemaVersion
correlationId
traceId
```

This helps debugging and replay.

---

# 73. Correlation ID

Example:

```text
Checkout Request
correlationId = ABC

OrderCreated
correlationId = ABC

PaymentRequested
correlationId = ABC
```

This lets operators connect a business workflow across services.

---

# 74. Distributed Tracing with Kafka

Tracing can follow:

```text
HTTP Request
 ↓
Order Service
 ↓
Kafka publish
 ↓
Inventory Consumer
 ↓
Database
```

Propagate trace context through supported messaging instrumentation.

---

# 75. Kafka Monitoring

Important metrics:

```text
Consumer lag
Producer error rate
Consumer error rate
Request latency
Bytes in/out
Under-replicated partitions
Broker health
Disk usage
```

---

# 76. Consumer Lag Alert

An alert can trigger when:

```text
Lag > threshold
```

for a sustained period.

Don't alert on a tiny temporary lag spike if the system naturally processes bursts.

Use business impact and trends.

---

# 77. Kafka Partition Hotspot

Suppose:

```text
key = one popular customer
```

and many events use the same key.

One partition may become overloaded.

Symptoms:

```text
One partition lagging
Others mostly idle
```

Key design should balance ordering requirements with distribution.

---

# 78. Consumer Scaling

If:

```text
8 partitions
```

you can scale a consumer group up to roughly:

```text
8 active consumers
```

for that topic/group.

Beyond that, additional consumers don't provide additional partition-level parallelism.

---

# 79. Kafka vs RabbitMQ

Kafka:

```text
Event streaming
High throughput
Durable retention
Replay
Partition-based scaling
```

RabbitMQ:

```text
Message broker
Routing
Queues
Traditional work distribution
```

The right choice depends on the workload.

---

# 80. Kafka vs REST

REST:

```text
Immediate request/response
Simple synchronous interaction
```

Kafka:

```text
Asynchronous
Decoupled
Durable event stream
Replayable
```

They are complementary rather than direct replacements.

---

# 81. When Should You NOT Use Kafka?

Don't introduce Kafka just because:

```text
Microservices use Kafka
```

Avoid it when:

```text
Simple synchronous CRUD is enough
Very low system complexity is required
No asynchronous workflow exists
Operational overhead isn't justified
```

---

# 82. Kafka in Ecommerce

Useful events:

```text
OrderCreated
PaymentCompleted
PaymentFailed
InventoryReserved
InventoryReleased
OrderCancelled
UserRegistered
```

Potential consumers:

```text
Inventory
Payment
Notification
Analytics
Search
```

---

# 83. Ecommerce Order Flow

Example:

```text
Client
  ↓
Order Service
  ↓
Create PENDING order
  ↓
OrderCreated
  ↓
Kafka
  |
  +--> Inventory
  |
  +--> Payment
  |
  +--> Notification
```

Then:

```text
InventoryReserved
PaymentCompleted
        ↓
Order Service
        ↓
CONFIRMED
```

---

# 84. Ecommerce Payment Failure

Flow:

```text
OrderCreated
      ↓
InventoryReserved
      ↓
PaymentFailed
      ↓
Order Service
      ↓
OrderCancelled
      ↓
InventoryReleased
```

The compensating operations must be idempotent.

---

# 85. Ecommerce Notification

Notification is usually a good candidate for asynchronous processing.

Example:

```text
OrderCreated
    ↓
Kafka
    ↓
Notification Service
    ↓
Email/SMS
```

The order API doesn't necessarily need to wait for email delivery.

---

# 86. Ecommerce Analytics

Analytics can consume events independently:

```text
OrderCreated
PaymentCompleted
ProductViewed
```

Analytics does not need to be directly called by the Order Service.

This is a major benefit of event-driven architecture.

---

# 87. Ecommerce Search Indexing

A product update can publish:

```text
ProductUpdated
```

Then:

```text
Kafka
 ↓
Search Indexer
 ↓
Search engine
```

The search index becomes eventually consistent with the primary database.

---

# 88. Eventual Consistency Example

Product updated:

```text
MySQL
 ↓
ProductUpdated
 ↓
Kafka
 ↓
Search Index
```

For a short period:

```text
MySQL = new product
Search = old product
```

This is acceptable if the business requirement allows eventual search consistency.

---

# 89. Kafka Security

Production Kafka should consider:

```text
Authentication
Authorization
TLS
Encryption
ACLs
Network restrictions
Secret management
```

Don't expose Kafka brokers publicly without strong security controls.

---

# 90. Kafka ACLs

Access can be restricted so a service has only required permissions.

Example:

```text
Inventory Service
→ consume order-events
→ produce inventory-events
```

It should not automatically have permission to every topic.

---

# 91. Kafka Authentication

Common mechanisms can include:

```text
SASL
TLS certificates
OAuth-based authentication
```

The exact configuration depends on the Kafka deployment.

---

# 92. Kafka Disaster Recovery

Consider:

```text
Replication
Backup strategy
Cross-region architecture
Topic retention
Consumer offset recovery
Replay strategy
```

Don't assume replication alone is a complete disaster recovery plan.

---

# 93. Kafka Schema Compatibility

Before changing an event:

```text
Check consumers
Check schema compatibility
Deploy compatible producer
Deploy consumers
```

A breaking event change can impact many services.

---

# 94. Kafka Event Versioning

Possible approach:

```text
schemaVersion = 1
schemaVersion = 2
```

Prefer backward-compatible changes where possible.

Avoid creating a new topic for every tiny field change unless there is a strong reason.

---

# 95. Kafka Testing

Test:

```text
Producer serialization
Consumer deserialization
Successful processing
Duplicate events
Retry
DLT
Ordering
Database transaction
Failure recovery
```

For realistic integration tests, Testcontainers can run Kafka during tests.

---

# 96. Kafka Integration Test

Conceptual flow:

```text
Start Kafka container
       ↓
Start Spring context
       ↓
Publish OrderCreated
       ↓
Consumer processes event
       ↓
Verify database state
```

This catches issues that mocks cannot.

---

# 97. Kafka Consumer Test

Verify:

```text
Given OrderCreated
When consumer receives it
Then inventory is reserved
```

Also:

```text
Given duplicate event
Then inventory is not reserved twice
```

---

# 98. Kafka DLT Test

Test:

```text
Invalid event
 ↓
Consumer fails
 ↓
Retries
 ↓
Retries exhausted
 ↓
DLT
```

Verify the message reaches the expected dead-letter destination and useful metadata is preserved.

---

# 99. Kafka Retry Test

Simulate:

```text
Attempt 1 → temporary failure
Attempt 2 → temporary failure
Attempt 3 → success
```

Verify:

```text
Bounded retry count
Expected backoff
One successful business effect
```

---

# 100. Kafka Performance

Measure:

```text
Records/sec
Consumer lag
Producer latency
Batch size
Compression
Partition distribution
CPU
Network
Disk
```

Optimize based on workload.

---

# 101. Batch Consumption

Processing messages individually:

```text
Message 1 → DB
Message 2 → DB
Message 3 → DB
```

may be slower than batching where the application supports it:

```text
Messages 1-100
      ↓
Batch DB operation
```

But batching increases complexity and can increase failure/retry scope.

---

# 102. Kafka Compression

Compression can reduce:

```text
Network usage
Disk usage
```

but increases:

```text
CPU usage
```

Choose an appropriate compression strategy based on workload.

---

# 103. Kafka Consumer Concurrency

Increasing concurrency can improve throughput when:

```text
Enough partitions exist
Processing can run safely in parallel
Downstream systems can handle the load
```

Otherwise it can simply move the bottleneck downstream.

---

# 104. Kafka Backpressure

If:

```text
Producer rate > Consumer processing rate
```

lag grows.

Solutions:

```text
Scale consumers
Increase partitions when appropriate
Optimize processing
Batch
Reduce unnecessary downstream calls
Control producer rate
```

---

# 105. Event-Driven Architecture Benefits

```text
Loose coupling
Asynchronous processing
Independent consumers
Replay
Scalability
Failure isolation
Easy integration with analytics
```

---

# 106. Event-Driven Architecture Challenges

```text
Eventual consistency
Duplicate messages
Ordering
Schema evolution
Debugging
Retries
Poison messages
Operational complexity
Distributed transactions
```

---

# 107. Interview: What Is Kafka?

> Kafka is a distributed event-streaming platform used for high-throughput, durable, asynchronous communication. It organizes records into topics and partitions and allows multiple consumer groups to independently process the same events.

---

# 108. Interview: Topic vs Partition

> A topic is a logical stream of events. A partition is a subdivision of that topic that provides scalability and ordering within the partition. Multiple consumers can process different partitions in parallel.

---

# 109. Interview: What Is a Consumer Group?

> A consumer group is a set of consumers that share the work of consuming a topic. Each partition is assigned to one consumer within a group at a time, while different consumer groups can independently consume the same topic.

---

# 110. Interview: What Is Consumer Lag?

> Consumer lag is the difference between the latest available offset and the consumer's current processing position. Growing lag usually indicates that consumers are processing slower than producers are publishing.

---

# 111. Interview: How Do You Handle Duplicate Kafka Messages?

> I design consumers to be idempotent. I usually use a unique event ID or business idempotency key and enforce uniqueness at the persistence layer where appropriate, so processing the same message again doesn't create a duplicate business effect.

---

# 112. Interview: How Do You Handle Failed Messages?

> I classify failures into retryable and non-retryable categories. Temporary failures can use bounded retries with exponential backoff and jitter. Permanent failures or exhausted retries can be routed to a dead-letter topic for investigation and controlled replay.

---

# 113. Interview: Explain the Outbox Pattern

> The Outbox Pattern solves the database-plus-message publishing problem. I store the business change and an outbox event in the same local database transaction. A publisher then reads the outbox and sends the event to Kafka, so a successful database transaction doesn't silently lose its event.

---

# 114. Interview: Kafka Exactly Once?

> Kafka supports transactional and idempotent processing mechanisms, but exactly-once behavior for an entire business workflow is more complicated. If the workflow also changes an external database or calls another service, I still need idempotency and appropriate transaction or outbox patterns.

---

# 115. Interview: How Do You Maintain Event Ordering?

> Kafka guarantees ordering within a partition. If events for the same business entity must remain ordered, I use a stable key such as orderId so those events are routed to the same partition.

---

# 116. Interview: Why Use Kafka Instead of REST?

> REST is useful for synchronous request-response operations. Kafka is useful when I want asynchronous processing, decoupling, durable event retention, multiple independent consumers, or replay. I would choose based on the workflow rather than replacing every REST call with Kafka.

---

# 117. Interview: How Do You Handle Consumer Failure?

> I use appropriate offset management, retries for transient failures, dead-letter handling for messages that cannot be processed, and idempotent consumers. I also monitor consumer lag and processing errors so failures are visible operationally.

---

# 118. Interview: How Would You Design an Order Event Flow?

> I would create the order in a local transaction and publish an OrderCreated event, preferably using an Outbox Pattern. Inventory, payment, notification, and analytics services can consume the event independently. The order state can then be updated based on subsequent events such as InventoryReserved or PaymentCompleted.

---

# 119. Interview: How Do You Handle Payment Retry?

> Payment operations must be idempotent. I would use an idempotency key, bounded retries with backoff for transient failures, and a clear payment state such as PENDING or FAILED so a retry cannot accidentally create a second charge.

---

# 120. Interview: How Do You Monitor Kafka?

> I monitor consumer lag, producer and consumer errors, processing latency, throughput, partition distribution, broker health, disk usage, and under-replicated partitions. I also connect Kafka traces and logs with correlation or trace IDs where possible.

---

# 121. Interview: How Do You Test Kafka?

> For unit tests I can mock the producer or consumer dependencies, but for important integration behavior I prefer a real Kafka instance using Testcontainers. I test serialization, successful processing, duplicate messages, retries, dead-letter handling, and database effects.

---

# 122. Interview: What Happens If Database Update Succeeds but Kafka Publish Fails?

> That creates an inconsistency because the database change exists but the event doesn't. A common solution is the Outbox Pattern, where the business change and event record are committed in the same local database transaction and a separate publisher sends the event later.

---

# 123. Interview: What Happens If Kafka Message Is Processed but Offset Commit Fails?

> The message can be delivered again. That's why I design the consumer operation to be idempotent, so the repeated message doesn't create a duplicate business effect.

---

# 124. Interview: What Happens If a Consumer Is Slower Than the Producer?

> Consumer lag increases. I would first identify the bottleneck, then optimize processing, database operations, external calls, batching, or consumer concurrency. If the workload supports it, I can scale consumers and partitions while making sure downstream dependencies can handle the additional load.

---

# 125. Interview: How Do You Handle Poison Messages?

> I don't retry them indefinitely. I classify the failure, apply a bounded retry policy for transient problems, and route permanently failing messages to a dead-letter topic. The failed event should remain available for investigation and controlled replay.

---

# 126. Interview: Choreography vs Orchestration

> In choreography, services coordinate through events without a central workflow controller. In orchestration, a central component coordinates the steps. Choreography can be loosely coupled, while orchestration can make complex workflows easier to understand and monitor.

---

# 127. Interview: Kafka vs RabbitMQ

> Kafka is particularly strong for durable event streams, high throughput, partition-based scaling, and replay. RabbitMQ is a traditional message broker with strong queueing and routing capabilities. I choose based on whether the problem is primarily event streaming or message/work distribution.

---

# 128. Interview: What Is Eventual Consistency?

> Eventual consistency means different services or read models can temporarily have different states, but they converge after the relevant events are processed. It's common in event-driven microservices because each service owns its own data.

---

# 129. Ecommerce Kafka Architecture

A practical design:

```text
                     API Gateway
                          |
                          v
                    Order Service
                          |
                     MySQL + Outbox
                          |
                          v
                        Kafka
                          |
        +-----------------+------------------+
        |                 |                  |
        v                 v                  v
   Inventory           Payment         Notification
    Service             Service            Service
        |                 |                  |
        v                 v                  v
   Inventory DB       Payment DB         Email/SMS
```

Additional consumers can include:

```text
Analytics
Search indexing
Fraud detection
Audit
```

---

# 130. Complete Ecommerce Event Flow

```text
1. Client creates order
            ↓
2. Order Service creates PENDING order
            ↓
3. Outbox record created
            ↓
4. Publisher sends OrderCreated
            ↓
5. Inventory reserves stock
            ↓
6. Payment processes payment
            ↓
7. InventoryReserved
            ↓
8. PaymentCompleted
            ↓
9. Order Service marks order CONFIRMED
            ↓
10. Notification Service sends confirmation
```

Failure:

```text
PaymentFailed
      ↓
OrderCancelled
      ↓
InventoryReleased
```

Every consumer should be designed for retries and duplicate events.

---

# 131. Final Kafka Checklist

```text
□ Broker
□ Topic
□ Partition
□ Producer
□ Consumer
□ Consumer group
□ Offset
□ Key
□ Ordering
□ Replication
□ Retention
□ Consumer lag
□ Rebalancing
□ Serialization
□ Schema evolution
□ Retry
□ Backoff
□ Jitter
□ Dead-letter topic
□ Idempotency
□ Outbox
□ Inbox
□ Transactions
□ Security
□ Monitoring
□ Testing
□ Replay
```

---

# 132. Final Event-Driven Mental Model

```text
                 EVENT-DRIVEN SYSTEM
                         |
             +-----------+-----------+
             |                       |
          Producer                Consumer
             |                       |
             v                       v
           Kafka ----------------> Service
             |                       |
          Topic                   Database
             |
        Partitions
             |
      Consumer Groups
             |
      Offset + Lag
             |
     Retry + DLT + Replay
             |
      Idempotent Processing
```

---

# 133. Final Rule

> **Kafka gives you durable, distributed event streaming, but Kafka alone doesn't solve distributed-system reliability. A production design still needs idempotency, retries, dead-letter handling, schema evolution, observability, security, and patterns such as Outbox when database and event consistency matters.**
