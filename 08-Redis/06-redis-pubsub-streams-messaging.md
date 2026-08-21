# Redis — File 06: Pub/Sub, Streams & Messaging

This file covers Redis messaging capabilities and how they fit into backend and microservices architecture.

Core topics:

```text
Pub/Sub
Channels
Publishers
Subscribers
Pattern subscriptions
Redis Streams
XADD
XREAD
Consumer groups
XREADGROUP
XACK
Pending entries
Retries
Dead-letter patterns
Ordering
At-least-once delivery
Idempotent consumers
Spring Boot messaging
Microservices event examples
```

---

# 1. Redis Messaging

Redis can support several messaging patterns:

```text
Pub/Sub
Lists
Streams
```

They are not interchangeable.

The most important interview skill is understanding:

```text
Delivery guarantees
Persistence
Consumer behavior
Acknowledgement
Ordering
Replay
Failure recovery
```

---

# 2. Pub/Sub

Redis Pub/Sub follows:

```text
Publisher
    ↓
Channel
    ↓
Subscribers
```

Example:

```text
PUBLISH notifications "Order created"
```

Subscribers listening to:

```text
notifications
```

receive the message.

---

# 3. Basic Pub/Sub

Subscriber:

```text
SUBSCRIBE notifications
```

Publisher:

```text
PUBLISH notifications "hello"
```

The subscriber receives the message while actively subscribed.

---

# 4. Pub/Sub Is Real-Time

Pub/Sub is useful for:

```text
Live notifications
WebSocket fan-out
Cache invalidation hints
Real-time dashboards
Presence updates
Transient events
```

The key characteristic is:

```text
Low-latency fan-out
```

---

# 5. Pub/Sub Is Ephemeral

Pub/Sub messages are not intended to provide durable message storage.

If:

```text
Subscriber disconnected
```

and a message is published:

```text
Message may be missed
```

There is no normal replay mechanism for missed Pub/Sub messages.

---

# 6. Pub/Sub Mental Model

Think:

```text
Publisher
   ↓
Channel
   ↓
All currently connected subscribers
```

It is similar to:

```text
Broadcast
```

rather than:

```text
Durable queue
```

---

# 7. Multiple Subscribers

Suppose:

```text
Subscriber A
Subscriber B
Subscriber C
```

all subscribe to:

```text
orders
```

When:

```text
PUBLISH orders "Order 101"
```

all active subscribers receive the message.

This is fan-out.

---

# 8. Pub/Sub vs Queue

A queue usually means:

```text
One message
 ↓
One consumer processes it
```

Pub/Sub:

```text
One message
 ↓
Many subscribers receive it
```

Streams can support:

```text
Many consumers
+
consumer groups
+
durable processing
```

---

# 9. Pattern Subscription

Redis supports pattern subscriptions.

Example:

```text
PSUBSCRIBE order:*
```

This can receive messages published to channels matching:

```text
order:created
order:cancelled
order:shipped
```

Use patterns carefully because broad patterns can increase processing overhead.

---

# 10. PUBLISH Return Value

When using:

```text
PUBLISH channel message
```

Redis returns the number of subscribers that received the message.

Important:

> This does not mean the message was durably processed.

It only indicates delivery to matching active subscribers.

---

# 11. Pub/Sub Failure

Suppose:

```text
Service B
```

disconnects.

During the outage:

```text
Service A
 ↓
PUBLISH event
```

Service B misses the message.

This is why Pub/Sub should not normally be used when:

```text
Every event must be processed
```

---

# 12. Pub/Sub Use Case

Example:

```text
Order service
 ↓
PUBLISH order-events
 ↓
WebSocket notification service
```

If the notification service misses a transient event:

```text
Potentially acceptable
```

depending on product requirements.

---

# 13. Pub/Sub Cache Invalidation

Possible architecture:

```text
Service A
 ↓
Update data
 ↓
PUBLISH cache-invalidation
 ↓
Service B/C
```

This can notify local caches.

But:

> If missing an invalidation would cause unacceptable stale data, Pub/Sub alone may not be reliable enough.

---

# 14. Redis Streams

Redis Streams provide a persistent, append-oriented event structure.

Conceptually:

```text
Stream
--------------------------------
event 1
event 2
event 3
event 4
```

Unlike Pub/Sub:

```text
Stream entries remain available
```

until they are trimmed/deleted according to the stream's retention strategy.

---

# 15. XADD

Add an event:

```text
XADD orders * orderId 101 status CREATED
```

Conceptually:

```text
orders
  ↓
ID
  |
  +-- orderId = 101
  +-- status = CREATED
```

The `*` asks Redis to generate the stream entry ID.

---

# 16. Stream Entry ID

A Redis Stream entry ID has a form similar to:

```text
timestamp-sequence
```

Example:

```text
1730000000000-0
```

Redis uses IDs to order entries within the stream.

Do not confuse a stream entry ID with a business event ID.

---

# 17. Business Event ID

You may have:

```text
Redis stream ID
→ 1730000000000-0

Business order ID
→ 101
```

These serve different purposes.

Your event payload should include identifiers needed for business processing.

---

# 18. XREAD

Read stream entries:

```text
XREAD STREAMS orders 0
```

This reads entries from a specified position.

The application can maintain its progress using stream IDs.

---

# 19. Reading New Entries

Conceptually:

```text
XREAD STREAMS orders $
```

means:

```text
Read entries arriving after the current end
```

For long-running consumers, blocking reads can be used so the consumer waits for new events instead of constantly polling.

---

# 20. Blocking XREAD

Conceptually:

```text
XREAD BLOCK 5000 STREAMS orders $
```

The consumer can wait for new events for a period.

This is useful for:

```text
Worker processes
Event consumers
Low-latency processing
```

---

# 21. Streams and Consumer Groups

Consumer groups allow multiple consumers to share processing.

Architecture:

```text
orders stream
       ↓
consumer group
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
 C1    C2    C3
```

Messages are distributed among group consumers.

---

# 22. XGROUP CREATE

A consumer group can be created with:

```text
XGROUP CREATE orders order-workers 0
```

Conceptually:

```text
stream
+
group name
+
starting position
```

The starting ID determines where the group begins processing.

---

# 23. Consumer Group

Example:

```text
orders
 ↓
order-workers
 ├── consumer-1
 ├── consumer-2
 └── consumer-3
```

The group coordinates which entries have been delivered to which consumers.

---

# 24. XREADGROUP

A consumer reads as part of the group:

```text
XREADGROUP GROUP order-workers consumer-1 ...
```

Redis tracks:

```text
Which consumer received the message
```

This is a major difference from simple Pub/Sub.

---

# 25. Consumer Groups and Scaling

Suppose:

```text
1000 events
```

and:

```text
5 consumers
```

A consumer group can distribute work across them.

Conceptually:

```text
1000 events
 ↓
C1 → 200
C2 → 200
C3 → 200
C4 → 200
C5 → 200
```

Actual distribution depends on processing and consumer availability.

---

# 26. Acknowledgement

After successful processing:

```text
XACK orders order-workers message-id
```

This acknowledges the entry.

The consumer should acknowledge only after:

```text
Business processing succeeds
```

not immediately after receiving it.

---

# 27. Why Acknowledgement Matters

Suppose:

```text
Consumer receives event
 ↓
Application crashes
```

before:

```text
XACK
```

The event remains pending.

This allows recovery.

---

# 28. Pending Entries

A consumer group tracks entries that have been delivered but not acknowledged.

Conceptually:

```text
Stream
 ↓
Consumer group
 ↓
Pending entries
```

These can indicate:

```text
In-progress work
Failed processing
Dead consumers
Stuck consumers
```

---

# 29. XPENDING

Redis provides:

```text
XPENDING
```

to inspect pending entries.

This is important during:

```text
Consumer failure
Backlog investigation
Recovery
```

---

# 30. Consumer Crash

Suppose:

```text
Consumer A
```

receives:

```text
event 101
```

Then crashes before acknowledgement.

Now:

```text
event 101
```

may remain pending under A.

Another consumer can eventually claim/reprocess it according to the recovery design.

---

# 31. Reclaiming Pending Messages

Redis Streams provide commands/mechanisms such as:

```text
XCLAIM
XAUTOCLAIM
```

for transferring ownership of pending entries.

Conceptually:

```text
Consumer A crashed
       ↓
Pending event
       ↓
Consumer B claims it
       ↓
Process
       ↓
XACK
```

---

# 32. Why Reprocessing Happens

Suppose:

```text
Process event
 ↓
Database commit succeeds
 ↓
Application crashes
 ↓
XACK never happens
```

The event can be processed again.

This leads to:

```text
At-least-once processing
```

unless the architecture explicitly provides stronger semantics.

---

# 33. At-Least-Once Delivery

At-least-once means:

```text
Event should not be silently lost
```

but:

```text
Duplicate processing can occur
```

Therefore consumers should generally be:

```text
Idempotent
```

---

# 34. Idempotent Consumer

Suppose event:

```text
order:101:CREATED
```

is delivered twice.

Bad consumer:

```text
Create shipment twice
```

Good consumer:

```text
Check whether event/order was already processed
 ↓
If yes → ignore/reuse result
```

---

# 35. Idempotency Store

Possible:

```text
processed:event:101
```

in Redis.

But for critical business correctness, a durable database record or unique constraint may be a stronger final safeguard.

---

# 36. Stream Event Processing

Architecture:

```text
Producer
   ↓
Redis Stream
   ↓
Consumer Group
   ↓
Consumer
   ↓
Business Logic
   ↓
Database
   ↓
XACK
```

Important order:

```text
Process successfully
 ↓
Commit durable side effect
 ↓
Acknowledge message
```

---

# 37. What If XACK Happens First?

Bad:

```text
Receive
 ↓
XACK
 ↓
Database update
 ↓
Crash
```

If the application crashes after:

```text
XACK
```

but before:

```text
DB commit
```

the event may be considered processed even though the business side effect failed.

---

# 38. Correct Acknowledgement Principle

Prefer:

```text
Receive
 ↓
Process
 ↓
Persist side effect
 ↓
Acknowledge
```

This can still produce duplicate processing if the application crashes between:

```text
DB commit
```

and:

```text
XACK
```

Therefore:

```text
Idempotency
```

is essential.

---

# 39. Exactly-Once Processing

Be careful when an interviewer asks:

> "Does Redis Streams provide exactly-once processing?"

A strong answer:

> "Redis Streams can support reliable processing patterns, but an end-to-end exactly-once business effect is not automatically guaranteed. Consumers can be redelivered after failures, so I would design idempotent processing and durable side-effect handling."

---

# 40. Stream Ordering

Redis Streams preserve ordering within a stream.

However:

```text
Consumer group
+
multiple consumers
```

means different entries may be processed concurrently.

Therefore:

```text
Stream order
≠
Global business completion order
```

---

# 41. Per-Key Ordering

Suppose:

```text
Order 101
```

has events:

```text
CREATED
PAID
SHIPPED
```

You may require:

```text
CREATED
 ↓
PAID
 ↓
SHIPPED
```

If strict ordering matters, partition/routing and consumer design must preserve it.

---

# 42. Ordering vs Parallelism

More consumers:

```text
Throughput ↑
```

but potentially:

```text
Parallel processing ↑
Ordering complexity ↑
```

This is a common distributed-systems trade-off.

---

# 43. Stream Consumer Lag

Important operational metric:

```text
How far behind is the consumer?
```

If producers generate:

```text
10,000 events/sec
```

but consumers process:

```text
5,000/sec
```

backlog grows.

---

# 44. Stream Backlog

Conceptually:

```text
Producer
  ↓
1000 events/sec
  ↓
Stream
  ↓
500 events/sec consumer
```

Backlog:

```text
+500/sec
```

This eventually becomes a capacity problem.

---

# 45. Consumer Scaling

If processing is independent:

```text
1 consumer
 ↓
2 consumers
 ↓
5 consumers
```

can improve throughput.

But consider:

```text
CPU
DB capacity
Redis capacity
Ordering
Downstream limits
```

Don't scale consumers blindly.

---

# 46. Poison Message

A poison message is an event that repeatedly fails processing.

Example:

```text
Invalid payload
```

Consumer:

```text
Receive
 ↓
Fail
 ↓
Retry
 ↓
Fail
 ↓
Retry
```

This can create an infinite retry loop.

---

# 47. Retry Strategy

Use:

```text
Limited retries
+
backoff
+
dead-letter handling
```

Example:

```text
Attempt 1
 ↓
fail
 ↓
1 sec
 ↓
Attempt 2
 ↓
fail
 ↓
5 sec
 ↓
Attempt 3
 ↓
fail
 ↓
dead-letter
```

---

# 48. Dead-Letter Pattern

Redis Streams don't magically create a universal dead-letter queue.

You can implement a pattern:

```text
orders
 ↓
consumer
 ↓
retries exceeded
 ↓
orders:dead-letter
```

Store:

```text
Original event
Error
Attempt count
Timestamp
```

---

# 49. Retry Metadata

An event may carry:

```text
retryCount
lastError
failedAt
```

Alternatively, retry state can be tracked separately.

Keep the event contract clear.

---

# 50. Dead-Letter Processing

A dead-letter stream should be monitored.

Possible workflow:

```text
Dead-letter
 ↓
Investigate
 ↓
Fix issue
 ↓
Replay
```

Don't treat a dead-letter stream as:

```text
Trash
```

It is an operational recovery mechanism.

---

# 51. Event Schema

A useful event may contain:

```text
eventId
eventType
timestamp
aggregateId
version
payload
```

Example:

```json
{
  "eventId": "evt-123",
  "eventType": "ORDER_CREATED",
  "aggregateId": "101",
  "version": 1,
  "timestamp": "2026-08-21T10:00:00Z"
}
```

---

# 52. Why Event ID Matters

Use:

```text
eventId
```

for idempotency.

Example:

```text
processed:evt-123
```

If event arrives again:

```text
Already processed
```

---

# 53. Event Version

Include:

```text
version
```

when event schema may evolve.

Example:

```text
ORDER_CREATED v1
ORDER_CREATED v2
```

Consumers can then support:

```text
Backward compatibility
Migration
```

---

# 54. Redis Stream Retention

Streams can grow indefinitely if not managed.

Use trimming strategies when appropriate.

Example concept:

```text
XADD ... MAXLEN ...
```

or other stream trimming mechanisms.

---

# 55. Why Retention Matters

Without retention:

```text
Events
 ↓
Events
 ↓
Events
 ↓
Memory growth
```

You need a policy based on:

```text
Replay requirements
Compliance
Storage limits
Event age
Consumer recovery window
```

---

# 56. MAXLEN

A stream can be trimmed based on length.

Conceptually:

```text
Keep latest N entries
```

This is useful when only recent events are needed.

---

# 57. MINID

Redis also supports trimming based on stream IDs.

This can be useful when retention is tied to:

```text
Event age/time
```

rather than only entry count.

---

# 58. Stream Retention Trade-Off

More retention:

```text
Replay capability ↑
Memory usage ↑
```

Less retention:

```text
Memory ↓
Replay window ↓
```

Choose based on business needs.

---

# 59. Pub/Sub vs Streams

| Feature | Pub/Sub | Streams |
|---|---|---|
| Persistent entries | No | Yes |
| Replay | No normal replay | Yes |
| Consumer groups | No | Yes |
| Acknowledgement | No | Yes |
| Pending messages | No | Yes |
| Fan-out | Excellent | Supported through groups/consumers |
| Missed messages | Can be lost | Can be recovered |
| Best for | Real-time transient events | Durable event processing |

---

# 60. Lists vs Streams

| Feature | Lists | Streams |
|---|---|---|
| Simple queue | Excellent | Yes |
| Consumer groups | No | Yes |
| Acknowledgement | Limited | Yes |
| Pending tracking | No | Yes |
| Event IDs | No dedicated stream IDs | Yes |
| Replay | Limited/manual | Yes |
| Complex processing | Less suitable | Better |

---

# 61. Pub/Sub Use Case

Use Pub/Sub for:

```text
WebSocket notifications
Live UI updates
Transient events
Presence
Simple fan-out
```

When:

```text
Missing an event is acceptable
```

---

# 62. Streams Use Case

Use Streams for:

```text
Order events
Notification processing
Audit/event pipelines
Background workers
Microservice integration
```

when you need:

```text
Durability
Acknowledgement
Consumer groups
Recovery
Replay
```

---

# 63. Redis Streams vs Kafka

Redis Streams:

```text
Simple
Already have Redis
Low operational complexity
Good for moderate event workloads
Useful for Redis-centric architectures
```

Kafka:

```text
Large-scale event streaming
Long retention
High throughput
Partitioned event architecture
Rich ecosystem
```

The correct choice depends on scale and requirements.

---

# 64. Redis Streams vs RabbitMQ

RabbitMQ:

```text
Message broker
Routing
Acknowledgement
Queues
Rich messaging semantics
```

Redis Streams:

```text
Data structure
Stream entries
Consumer groups
Fast Redis-centric architecture
```

Don't choose based only on speed.

---

# 65. Spring Boot Pub/Sub

Spring applications can integrate Redis messaging through Spring Data Redis.

Typical components include:

```text
RedisMessageListenerContainer
MessageListener
ChannelTopic
PatternTopic
```

Conceptually:

```text
Redis
 ↓
MessageListenerContainer
 ↓
Java listener
```

---

# 66. Pub/Sub Listener Concept

Example:

```java
@Bean
RedisMessageListenerContainer container(
        RedisConnectionFactory connectionFactory,
        MessageListener listener) {

    RedisMessageListenerContainer container =
        new RedisMessageListenerContainer();

    container.setConnectionFactory(connectionFactory);

    return container;
}
```

The exact bean configuration depends on the application.

---

# 67. Pub/Sub Topic

A topic/channel can be represented conceptually as:

```java
new ChannelTopic("orders");
```

Then register:

```text
listener
+
topic
```

---

# 68. Spring Boot Pub/Sub Flow

```text
OrderService
 ↓
PUBLISH orders
 ↓
Redis
 ↓
RedisMessageListenerContainer
 ↓
NotificationListener
```

---

# 69. Spring Boot Streams

Spring Data Redis also provides stream abstractions.

Conceptually:

```text
StreamOperations
StreamMessageListenerContainer
Consumer
ReadOffset
```

The exact APIs vary with Spring Data Redis version.

---

# 70. Stream Consumer Architecture

```text
Redis Stream
     ↓
Consumer Group
     ↓
Spring consumer
     ↓
Business service
     ↓
Database
     ↓
ACK
```

---

# 71. Consumer Error Handling

A consumer should distinguish:

```text
Transient failure
Permanent failure
Poison message
```

Transient:

```text
Retry
```

Permanent:

```text
Dead-letter
```

Unknown:

```text
Controlled retry + monitoring
```

---

# 72. Database Failure During Event Processing

Suppose:

```text
Stream event
 ↓
Consumer
 ↓
MySQL unavailable
```

Don't acknowledge.

Instead:

```text
Retry later
```

with backoff.

Otherwise the event can be lost from the consumer's pending workload.

---

# 73. Redis Failure During Processing

Suppose:

```text
Consumer processing
 ↓
Redis unavailable
```

Recovery depends on:

```text
What Redis is being used for
```

If Redis is the stream itself:

```text
Stream availability is affected
```

If Redis is only an idempotency cache:

```text
Fallback strategy may differ
```

---

# 74. Event Processing and Transactions

Suppose:

```text
Redis Stream
 ↓
Update MySQL
 ↓
XACK Redis
```

MySQL and Redis are separate systems.

You cannot simply assume:

```text
DB commit + XACK
```

is one atomic transaction.

This is why:

```text
Idempotency
+
retry
+
durable state
```

are important.

---

# 75. Transactional Outbox

A common microservices solution:

```text
Service
 ↓
DB transaction
 ├── business data
 └── outbox event
        ↓
    publisher
        ↓
 Redis Stream
```

The DB transaction ensures:

```text
Business update
+
event record
```

are committed together.

The publisher later sends the event.

---

# 76. Why Outbox Helps

Without outbox:

```text
Update DB
 ↓
Publish Redis event
```

Failure between them:

```text
DB updated
event not published
```

Outbox avoids that gap by recording the event transactionally.

---

# 77. Event Consumer + Idempotency

Architecture:

```text
Redis Stream
 ↓
Consumer
 ↓
Check processed event
 ↓
DB transaction
 ↓
Mark processed
 ↓
XACK
```

The durable processed-event record can be protected with:

```text
Unique constraint
```

for strong duplicate prevention.

---

# 78. At-Least-Once + Idempotency

This is one of the most important backend patterns:

```text
Delivery:
At least once

Processing:
Idempotent

Result:
Effectively one logical business effect
```

This is often more realistic than claiming true exactly-once distributed processing.

---

# 79. Event Replay

One advantage of Streams is:

```text
Events remain available
```

for a configured retention window.

This allows:

```text
Consumer restart
Replay
Recovery
New consumer
```

Replay strategy must account for:

```text
Duplicate side effects
```

Therefore consumers should be idempotent.

---

# 80. Consumer Group Restart

Suppose:

```text
consumer-1
```

crashes.

Pending events can be:

```text
Inspected
Claimed
Reprocessed
Acknowledged
```

This is much more robust than ephemeral Pub/Sub.

---

# 81. Consumer Group Scaling

Increase:

```text
Consumers
```

when:

```text
Backlog grows
```

But monitor:

```text
DB capacity
Redis CPU
Network
Processing time
Downstream APIs
```

---

# 82. Backpressure

If producer speed is:

```text
10,000/sec
```

and consumer capacity:

```text
5,000/sec
```

backlog grows.

Solutions:

```text
Increase consumers
Batch processing
Optimize consumer
Slow producers
Rate limit
Scale downstream systems
```

---

# 83. Batch Processing

A consumer may process multiple events per batch.

Benefits:

```text
Fewer DB round trips
Higher throughput
Better network efficiency
```

Trade-offs:

```text
Larger failure unit
More latency before acknowledgement
More retry complexity
```

---

# 84. Event Ordering

If strict order is required for:

```text
same order
same account
same aggregate
```

design the event routing around that requirement.

Do not assume:

```text
Multiple consumers
=
global order
```

---

# 85. Duplicate Event Example

Event:

```text
evt-123
ORDER_CREATED
```

Delivered:

```text
A
```

A commits DB.

Before `XACK`:

```text
A crashes
```

Event delivered again:

```text
B
```

B must detect:

```text
evt-123 already processed
```

and avoid duplicating the side effect.

---

# 86. Poison Message Recovery

Architecture:

```text
Stream
 ↓
Consumer
 ↓
Failure
 ↓
Retry count
 ↓
Exceeded
 ↓
Dead-letter stream
```

Monitor:

```text
Dead-letter count
```

and alert when it grows.

---

# 87. Event Contract Best Practices

Include:

```text
eventId
eventType
aggregateId
timestamp
version
payload
```

Keep event contracts:

```text
Stable
Backward compatible
Documented
```

---

# 88. Event Payload Size

Avoid huge event payloads.

Instead of:

```text
Entire 5 MB order
```

consider:

```text
eventId
orderId
eventType
small relevant fields
```

Consumers can retrieve additional data if needed.

But this creates another trade-off:

```text
Smaller event
+
extra DB/API lookup
```

Choose based on workload.

---

# 89. Pub/Sub Security

Protect Redis messaging with:

```text
Authentication
ACLs
Private networking
TLS
Least privilege
```

Don't expose production Redis channels publicly.

---

# 90. Monitoring Pub/Sub

Track:

```text
Connected subscribers
Publish rate
Subscriber errors
Message delivery latency
```

Remember:

```text
Pub/Sub has no durable backlog
```

so operational visibility is different from Streams.

---

# 91. Monitoring Streams

Important metrics:

```text
Stream length
Consumer lag
Pending entries
Processing rate
Failure rate
Retry count
Dead-letter count
Oldest pending message
```

---

# 92. Consumer Lag

If:

```text
Producer = 10k/sec
Consumer = 9k/sec
```

backlog grows by:

```text
1k/sec
```

Even if Redis itself is healthy, the system can eventually fail because consumers cannot keep up.

---

# 93. Stream Trimming

A stream needs a retention strategy.

Options conceptually:

```text
Keep latest N entries
Keep entries newer than a time boundary
Trim after successful processing
```

Choose based on replay requirements.

---

# 94. Trim Carefully

If consumers need:

```text
7 days of replay
```

don't trim events after:

```text
1 hour
```

Retention is part of the event-processing design.

---

# 95. Common Mistake

Don't say:

> "Redis Pub/Sub guarantees message delivery."

Better:

> "Pub/Sub is ephemeral and active subscribers receive published messages; disconnected subscribers can miss them."

---

# 96. Common Mistake

Don't say:

> "Redis Streams are exactly once."

Better:

> "Streams support reliable processing patterns, but consumers can receive messages again after failures, so idempotent processing is important."

---

# 97. Common Mistake

Don't acknowledge before processing.

Bad:

```text
XREADGROUP
 ↓
XACK
 ↓
DB update
```

Better:

```text
XREADGROUP
 ↓
Process
 ↓
DB commit
 ↓
XACK
```

with idempotent handling.

---

# 98. Common Mistake

Don't let a poison message retry forever.

Use:

```text
Retry limit
Backoff
Dead-letter handling
Monitoring
```

---

# 99. Common Mistake

Don't let Streams grow without bound.

Use:

```text
Retention
Trimming
Capacity planning
```

---

# 100. Interview Question

### Pub/Sub vs Streams?

Answer:

> "Pub/Sub is mainly for low-latency ephemeral fan-out, where disconnected subscribers can miss messages. Streams provide persistent entries, consumer groups, acknowledgements and pending-message tracking, so they are more suitable for reliable event processing."

---

# 101. Interview Question

### What happens if a Redis Stream consumer crashes?

Answer:

> "If the message was delivered but not acknowledged, it can remain pending in the consumer group. Another consumer can inspect and reclaim it using mechanisms such as XCLAIM or XAUTOCLAIM, then process and acknowledge it."

---

# 102. Interview Question

### What is XACK?

Answer:

> "`XACK` acknowledges that a consumer successfully processed a stream entry. It removes the entry from the consumer group's pending work for that consumer."

---

# 103. Interview Question

### Why do we need idempotent consumers?

Answer:

> "Because a consumer can crash after committing the business side effect but before acknowledging the Redis message. The message can then be delivered again, so processing must safely tolerate duplicates."

---

# 104. Interview Question

### How do you handle poison messages?

Answer:

> "I'd use bounded retries with backoff. If processing continues to fail after the retry limit, I'd move the event to a dead-letter stream with enough metadata for investigation and replay."

---

# 105. Interview Question

### How do you scale Redis Stream consumers?

Answer:

> "I'd add consumers to the same consumer group so work can be distributed, but I'd also monitor consumer lag and downstream capacity. Scaling consumers is useful only if Redis and the dependent systems can handle the additional throughput."

---

# 106. Interview Question

### How do you guarantee event processing order?

Answer:

> "Redis Streams preserve entry order within a stream, but multiple consumers can process events concurrently. If business ordering is required, I'd partition or route events by aggregate key so events for the same entity are processed in the required order."

---

# 107. Interview Question

### What is a consumer group?

Answer:

> "A consumer group lets multiple consumers share processing of a stream. Redis tracks which consumer received each entry and maintains pending entries until they are acknowledged."

---

# 108. Interview Question

### What is at-least-once delivery?

Answer:

> "It means the system prioritizes not losing an event, but the same event may be delivered more than once after failures. Consumers therefore need idempotent processing."

---

# 109. Interview Question

### Redis Streams vs Kafka?

Answer:

> "Redis Streams are convenient when Redis is already part of the architecture and the event workload is moderate. Kafka is generally better suited to large-scale distributed event streaming, long retention and a broader streaming ecosystem. I'd choose based on throughput, retention, replay, operational requirements and architecture."

---

# 110. Interview Question

### Redis Streams vs RabbitMQ?

Answer:

> "RabbitMQ is a dedicated message broker with rich routing and messaging semantics. Redis Streams provide stream storage and consumer groups inside Redis. The right choice depends on delivery, routing, retention, throughput and operational requirements."

---

# 111. Interview Scenario

### Order event is processed twice.

Investigate:

```text
Consumer crash
XACK timing
Retry
Pending entries
Duplicate producer event
```

Then add:

```text
eventId
+
idempotency
+
durable uniqueness
```

---

# 112. Interview Scenario

### Consumer lag is increasing.

Investigate:

```text
Producer rate
Consumer rate
Processing time
DB latency
Downstream APIs
Consumer count
Redis CPU
```

Then:

```text
Scale consumers
Optimize processing
Batch work
Reduce producer rate
Scale downstream
```

---

# 113. Interview Scenario

### A consumer keeps failing one message.

Likely:

```text
Poison message
```

Use:

```text
Retry limit
Dead-letter stream
Alert
Manual/replay workflow
```

---

# 114. Interview Scenario

### Redis Stream memory keeps growing.

Investigate:

```text
Stream retention
Trimming
Consumer backlog
Producer rate
Consumer rate
Dead-letter growth
```

Fix:

```text
Retention policy
Consumer scaling
Backpressure
```

---

# 115. Interview Scenario

### Notification service can tolerate missed events.

Use:

```text
Pub/Sub
```

Potential architecture:

```text
Order Service
 ↓
PUBLISH notification
 ↓
Notification Service
```

If missed notifications are unacceptable:

```text
Use Streams or another durable messaging system
```

---

# 116. Interview Scenario

### Payment events must not be silently lost.

Don't choose:

```text
Pub/Sub only
```

Consider:

```text
Redis Streams
+
idempotent consumer
+
durable payment state
+
reconciliation
```

For very high-value payment workflows, a dedicated durable event platform may be preferable.

---

# 117. Interview Scenario

### Multiple services need every event.

Pub/Sub:

```text
Publisher
 ↓
Channel
 ├── Service A
 ├── Service B
 └── Service C
```

Streams can also support separate consumer groups when each logical service needs its own processing position.

---

# 118. Consumer Groups Per Service

Example:

```text
orders stream
 ├── inventory-group
 ├── notification-group
 └── analytics-group
```

Each group maintains its own consumption state.

This allows different services to process the same stream independently.

---

# 119. Consumer Group Semantics

Within one group:

```text
Consumers share work
```

Across different groups:

```text
Each group gets its own logical stream position
```

This distinction is very important.

---

# 120. Stream Architecture

Example:

```text
                    Redis Stream
                         |
        +----------------+----------------+
        |                |                |
 inventory-group   notification-group   analytics-group
        |                |                |
    C1  C2           C1  C2             C1
```

This provides:

```text
Service-level fan-out
+
consumer-level load balancing
```

---

# 121. Pub/Sub Architecture

```text
                 Channel
                    |
       +------------+------------+
       |            |            |
      A             B            C
```

All active subscribers receive the message.

---

# 122. Stream Architecture Difference

```text
                 Stream
                    |
       +------------+------------+
       |            |            |
    Group A       Group B      Group C
       |
    C1 C2 C3
```

Within each group:

```text
Work is distributed
```

Across groups:

```text
Each group processes its own copy logically
```

---

# 123. Event-Driven E-commerce

Example:

```text
Order Service
      ↓
ORDER_CREATED
      ↓
Redis Stream
      ↓
 ┌────┼────────────┐
 ↓    ↓            ↓
Inventory Notification Analytics
```

Each service can use its own consumer group.

---

# 124. Order Processing Flow

```text
Client
 ↓
Order API
 ↓
MySQL transaction
 ↓
Outbox event
 ↓
Redis Stream
 ↓
Consumer groups
```

This provides a much stronger architecture than:

```text
DB update
+
best-effort Pub/Sub
```

for important events.

---

# 125. Idempotency at Consumer

Example:

```text
eventId = evt-123
```

Consumer checks:

```text
processed_event
```

If exists:

```text
Skip duplicate
XACK
```

If not:

```text
Process
 ↓
Record processed event
 ↓
XACK
```

For strong correctness, make the processing and duplicate protection part of the same durable transaction where possible.

---

# 126. Redis for Lightweight Event Processing

Redis Streams are a good fit when:

```text
Redis already exists
Moderate event volume
Low latency
Simple consumer groups
Short/moderate retention
```

For:

```text
Huge event history
Very long retention
Massive streaming ecosystem
Complex stream processing
```

consider Kafka or another specialized platform.

---

# 127. Final Messaging Checklist

```text
□ Pub/Sub
□ SUBSCRIBE
□ PUBLISH
□ PSUBSCRIBE
□ Ephemeral delivery
□ Pub/Sub fan-out
□ Streams
□ XADD
□ XREAD
□ XGROUP
□ XREADGROUP
□ XACK
□ XPENDING
□ XCLAIM
□ XAUTOCLAIM
□ Consumer groups
□ Pending messages
□ At-least-once
□ Idempotent consumers
□ Event ordering
□ Consumer lag
□ Backpressure
□ Poison messages
□ Retry
□ Dead-letter
□ Stream retention
□ Stream trimming
□ Spring Boot integration
□ Transactional outbox
□ Redis Streams vs Kafka
□ Redis Streams vs RabbitMQ
```

---

# 128. Final Interview Answer

If asked:

> "How would you design event processing with Redis in a Spring Boot microservices system?"

Say:

> "For important events I'd use Redis Streams rather than Pub/Sub because Streams provide persistence, consumer groups and acknowledgement tracking. I'd create a consumer group per logical service and scale consumers within each group. The consumer would process the event, commit the durable business change, and then acknowledge it. Because failures can cause redelivery, I'd make consumers idempotent using an event ID and a durable uniqueness mechanism. I'd also monitor lag, pending messages, retries and dead-letter events."

---

# 129. What Comes Next

```text
File 07 → Redis Replication, Sentinel, Cluster & High Availability
```

Next we will cover:

```text
Primary/Replica
Replication
Read scaling
Replication lag
Sentinel
Failover
Quorum
Redis Cluster
Hash slots
Hash tags
Resharding
Cluster failover
MOVED / ASK
Multi-key operations
Cluster limitations
High availability
Disaster recovery
Spring Boot connection behavior
Production architecture
Interview scenarios
```

Key takeaway:

> **Pub/Sub is excellent for ephemeral real-time fan-out, while Redis Streams are much better suited to reliable event processing. The real backend skill is designing the consumer side correctly: acknowledgement after successful processing, idempotency for redelivery, bounded retries, dead-letter handling, ordering where required, and monitoring consumer lag.**
