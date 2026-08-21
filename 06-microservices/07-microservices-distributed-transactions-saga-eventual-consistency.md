# Microservices — Distributed Transactions, Saga & Eventual Consistency

This file covers one of the most important advanced microservices interview areas:

```text
Distributed Transactions
Eventual Consistency
Saga Pattern
Orchestration
Choreography
Compensating Transactions
Transactional Outbox
Idempotency
Failure Handling
```

The key idea:

> In microservices, one business operation may span multiple independently managed services and databases.

---

# 1. The Distributed Transaction Problem

Imagine an e-commerce checkout:

```text
Order Service
    ↓
Inventory Service
    ↓
Payment Service
    ↓
Shipping Service
```

Each service may own a separate database.

Example:

```text
Order DB
Inventory DB
Payment DB
Shipping DB
```

A single checkout operation can therefore involve multiple transactions.

---

# 2. Why One Database Transaction Doesn't Work

In a monolith:

```text
BEGIN TRANSACTION

Create Order
Update Inventory
Create Payment

COMMIT
```

If something fails:

```text
ROLLBACK
```

Everything can be rolled back together.

In microservices:

```text
Order DB
Inventory DB
Payment DB
```

There is no simple local transaction covering all of them.

---

# 3. Distributed Transaction

A distributed transaction spans multiple resources or services.

Example:

```text
Order transaction
+
Inventory transaction
+
Payment transaction
```

The system needs to maintain business consistency despite failures between these operations.

---

# 4. Why Distributed Transactions Are Difficult

Imagine:

```text
Order created
      ↓
Inventory reserved
      ↓
Payment failed
```

Now we need to decide:

```text
What happens to the reserved inventory?
What happens to the order?
```

We may need:

```text
Release inventory
Cancel order
```

These are compensating actions.

---

# 5. Two-Phase Commit

A traditional distributed transaction technique is:

```text
Two-Phase Commit (2PC)
```

A coordinator manages multiple participants.

---

# 6. 2PC Phase 1 — Prepare

Conceptually:

```text
Coordinator
    |
    +--> Order DB: Prepare?
    |
    +--> Inventory DB: Prepare?
    |
    +--> Payment DB: Prepare?
```

Participants respond:

```text
YES
```

or:

```text
NO
```

---

# 7. 2PC Phase 2 — Commit

If everyone says:

```text
YES
```

the coordinator sends:

```text
COMMIT
```

to all participants.

If someone says:

```text
NO
```

the coordinator can request rollback/abort.

---

# 8. Why 2PC Is Often Avoided in Microservices

Problems include:

```text
Coordination overhead
Latency
Lock/resource holding
Failure complexity
Reduced availability
Tight coupling
Operational complexity
```

Modern microservice architectures often prefer local transactions plus asynchronous workflows.

---

# 9. Local Transaction

Each service should normally own its own database transaction.

Example:

```text
Order Service
    ↓
Order DB
    ↓
COMMIT
```

Then communicate the result through:

```text
Event
```

or another explicit integration mechanism.

---

# 10. Eventual Consistency

Eventual consistency means different parts of the system may temporarily have different views of the state.

Example:

```text
Order created
 ↓
Order DB = CREATED

Inventory event not processed yet
 ↓
Inventory still unchanged
```

After processing:

```text
Inventory updated
```

The system eventually converges to the desired state.

---

# 11. Is Eventual Consistency a Bug?

Not automatically.

It is an architectural trade-off.

For example:

```text
Product recommendations
Analytics
Notifications
Search indexing
```

can often tolerate small delays.

---

# 12. Strong Consistency vs Eventual Consistency

### Strong consistency

Readers see the latest committed state according to the consistency model.

### Eventual consistency

Readers may temporarily see stale state, but replicas/workflows converge over time if failures are resolved.

---

# 13. Example

Customer places order:

```text
10:00:00
Order = CREATED
```

Inventory event processed:

```text
10:00:01
Inventory = RESERVED
```

There was a temporary inconsistency window:

```text
Order created
Inventory not yet reserved
```

The business workflow determines whether that is acceptable.

---

# 14. Saga Pattern

The Saga pattern manages a long-running business transaction using a sequence of local transactions.

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

Each step commits independently.

If a later step fails, compensating actions can undo the business effect of earlier steps.

---

# 15. Saga Example

Successful flow:

```text
Create Order
      ↓
Reserve Inventory
      ↓
Payment Success
      ↓
Shipment Created
```

Failure:

```text
Create Order
      ↓
Reserve Inventory
      ↓
Payment Failed
```

Compensation:

```text
Release Inventory
      ↓
Cancel Order
```

---

# 16. Compensation Is Not Database Rollback

Very important.

A database rollback:

```text
ROLLBACK
```

can undo changes inside a transaction.

A Saga compensation:

```text
ReleaseInventory()
CancelOrder()
```

is a new business operation.

It may itself fail and require retry/recovery.

---

# 17. Compensation Is Not Always a Perfect Undo

Example:

```text
Send email
```

If the email was already delivered:

```text
"Undo email"
```

may not be possible.

Therefore Saga design must understand real business side effects.

---

# 18. Saga Types

Two common approaches:

```text
Orchestration
Choreography
```

---

# 19. Saga Orchestration

A central orchestrator coordinates the workflow.

```text
             Saga Orchestrator
              /      |      \
             ↓       ↓       ↓
          Order   Inventory Payment
```

The orchestrator tells services what to do.

---

# 20. Orchestration Example

```text
Orchestrator
    |
    | Create Order
    ↓
Order Service
    |
    | success
    ↓
Orchestrator
    |
    | Reserve Inventory
    ↓
Inventory Service
    |
    | success
    ↓
Orchestrator
    |
    | Process Payment
    ↓
Payment Service
```

---

# 21. Orchestration Failure

Suppose:

```text
Payment fails
```

The orchestrator can issue:

```text
Release Inventory
Cancel Order
```

The workflow is explicit.

---

# 22. Advantages of Orchestration

```text
Central workflow visibility
Explicit business process
Easier to understand complex workflows
Centralized compensation logic
```

---

# 23. Disadvantages of Orchestration

```text
Orchestrator complexity
Potential central coupling
More workflow logic in one component
```

A badly designed orchestrator can become a "god service."

---

# 24. Saga Choreography

There is no central coordinator.

Services react to events.

Example:

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
    ↓
Shipping Service
```

Each service decides what to do based on events.

---

# 25. Choreography Example

```text
Order Service
    |
    | OrderCreated
    ↓
Kafka
    |
    ↓
Inventory Service
    |
    | InventoryReserved
    ↓
Kafka
    |
    ↓
Payment Service
    |
    | PaymentCompleted
    ↓
Kafka
    |
    ↓
Shipping Service
```

---

# 26. Advantages of Choreography

```text
Loose coupling
No central orchestrator
Natural event-driven communication
Services can evolve independently
```

---

# 27. Disadvantages of Choreography

Large workflows can become difficult to understand.

Example:

```text
OrderCreated
 ↓
InventoryReserved
 ↓
PaymentStarted
 ↓
PaymentFailed
 ↓
InventoryReleased
 ↓
OrderCancelled
 ↓
NotificationSent
```

It can become difficult to answer:

> "Who is responsible for the overall workflow?"

---

# 28. Orchestration vs Choreography

| Orchestration | Choreography |
|---|---|
| Central coordinator | No central coordinator |
| Explicit workflow | Event-driven workflow |
| Easier to visualize | Can be harder to trace |
| Central compensation logic | Compensation distributed |
| Orchestrator can become complex | Event chains can become complex |

---

# 29. When to Prefer Orchestration

Consider orchestration when:

```text
Workflow is complex
Many steps exist
Compensation is complicated
Business process needs explicit visibility
```

---

# 30. When to Prefer Choreography

Consider choreography when:

```text
Workflow is relatively simple
Services naturally react to events
Loose coupling is important
There is no need for a central workflow controller
```

---

# 31. Saga State

A Saga may need to track:

```text
Saga ID
Current step
Completed steps
Failed step
Compensation status
Retry count
```

Example:

```text
sagaId = saga-123
status = PAYMENT_FAILED
```

---

# 32. Saga State Machine

Conceptually:

```text
ORDER_CREATED
      ↓
INVENTORY_RESERVED
      ↓
PAYMENT_COMPLETED
      ↓
SHIPMENT_CREATED
      ↓
COMPLETED
```

Failure:

```text
PAYMENT_FAILED
      ↓
RELEASE_INVENTORY
      ↓
CANCEL_ORDER
      ↓
COMPENSATED
```

---

# 33. Idempotency in Saga

Saga operations can be retried.

Example:

```text
ReserveInventory(orderId=101)
```

may be called twice.

The Inventory Service should safely handle duplicate requests.

Possible approach:

```text
Unique reservation ID
+
Database constraint
```

---

# 34. Why Idempotency Is Critical

Distributed systems have:

```text
Retries
Timeouts
Duplicate messages
Consumer restarts
Network failures
```

Therefore:

> "The same request may arrive more than once."

Services should be designed accordingly.

---

# 35. Outbox Pattern

The Transactional Outbox is commonly used with Saga/event-driven systems.

Example:

```text
Order DB Transaction
 ├── Order record
 └── Outbox event
```

Both commit together.

---

# 36. Outbox Flow

```text
Order Service
     |
     ↓
BEGIN TRANSACTION
     |
     +--> INSERT order
     |
     +--> INSERT OrderCreated event into outbox
     |
     ↓
COMMIT
     |
     ↓
Outbox Publisher
     |
     ↓
Kafka
```

---

# 37. Why Outbox Helps Saga

It ensures:

```text
Business state changed
+
Event describing that change exists
```

inside the same local transaction.

This avoids the most dangerous part of the dual-write problem.

---

# 38. Outbox Does Not Guarantee Exactly Once

Important interview point:

```text
Outbox publish succeeds
 ↓
Publisher crashes
 ↓
Outbox row still appears unpublished
 ↓
Publisher retries
```

The event can be published twice.

Therefore:

```text
Consumers must be idempotent.
```

---

# 39. Inbox Pattern

An Inbox pattern can help consumers handle duplicate messages.

Conceptually:

```text
Kafka Event
     ↓
Inbox table
     ↓
Check eventId
     ↓
Process business change
```

The event ID can have a unique constraint.

---

# 40. Inbox + Outbox

For robust workflows:

```text
Producer
   |
Outbox
   |
 Kafka
   |
Inbox
   |
Consumer DB
```

This provides stronger control over:

```text
Reliable publishing
Duplicate detection
Business processing
```

---

# 41. Saga Failure Scenarios

Consider:

```text
Order Created
Inventory Reserved
Payment Failed
```

Now:

```text
Release Inventory
```

fails.

What happens?

You need:

```text
Retry
Alerting
Manual recovery
Compensation state
```

A Saga must handle compensation failure too.

---

# 42. Compensation Retry

Example:

```text
Payment Failed
 ↓
Release Inventory
 ↓
Network timeout
 ↓
Retry
 ↓
Success
```

Use bounded retries and appropriate idempotency.

---

# 43. Permanent Compensation Failure

Suppose:

```text
Inventory Service
```

remains unavailable.

The Saga may enter:

```text
COMPENSATION_PENDING
```

rather than pretending everything is complete.

This state can trigger:

```text
Retry worker
Alert
Operations intervention
```

---

# 44. Saga Status

Useful statuses:

```text
STARTED
IN_PROGRESS
COMPLETED
FAILED
COMPENSATING
COMPENSATED
COMPENSATION_FAILED
```

Exact states depend on the workflow.

---

# 45. Business State vs Technical State

Don't mix everything into one field.

Example:

```text
Order Status:
CREATED
PAID
SHIPPED
CANCELLED
```

Saga state might separately track:

```text
Saga:
PAYMENT_FAILED
COMPENSATION_PENDING
```

This separation can make business logic clearer.

---

# 46. Eventual Consistency and User Experience

Users may see:

```text
Order placed
```

while payment processing is still:

```text
PROCESSING
```

Instead of pretending the operation is already complete.

A good API can expose meaningful states:

```text
PENDING
CONFIRMED
FAILED
```

---

# 47. Asynchronous Checkout

Possible architecture:

```text
POST /orders
      ↓
Order Service
      ↓
Order = PENDING
      ↓
OrderCreated
      ↓
Kafka
      ↓
Payment workflow
```

Client can later query:

```text
GET /orders/{id}
```

or receive a notification.

---

# 48. Synchronous vs Asynchronous Saga

A Saga can be implemented using:

```text
Synchronous service calls
```

or:

```text
Asynchronous events
```

Asynchronous workflows often improve decoupling but introduce more eventual consistency and operational complexity.

---

# 49. Saga and API Response

Suppose:

```text
POST /checkout
```

starts a long-running Saga.

Don't necessarily block for 30 seconds waiting for every step.

Instead return something like:

```http
202 Accepted
```

with a workflow/order identifier if the business process is asynchronous.

The client can query status.

---

# 50. 200 vs 202

### 200 OK

Usually means:

```text
Request completed successfully.
```

### 202 Accepted

Means:

```text
Request accepted for processing.
```

It doesn't necessarily mean the business operation has completed.

---

# 51. Saga Timeout

A Saga can have an overall timeout.

Example:

```text
Checkout Saga
maximum duration = 5 minutes
```

If it remains stuck:

```text
Mark failed/pending
Trigger compensation or manual recovery
```

The correct action depends on business semantics.

---

# 52. Saga Monitoring

Track:

```text
Saga started
Saga completed
Saga failed
Saga duration
Current step
Compensation count
Compensation failures
Stuck workflows
```

---

# 53. Correlation

Every Saga should have a traceable identifier.

Example:

```text
sagaId = saga-123
correlationId = req-456
```

Use them across:

```text
Logs
Events
Database
Traces
```

---

# 54. Distributed Tracing

A workflow might look like:

```text
Trace
 |
 +-- Order
 |
 +-- Inventory
 |
 +-- Payment
 |
 +-- Shipping
```

OpenTelemetry can help trace these interactions.

---

# 55. Saga vs 2PC

Very common interview question.

### 2PC

```text
Global transaction
Coordinator
Prepare
Commit
```

### Saga

```text
Local transactions
Business events
Compensating actions
Eventual consistency
```

Saga generally provides more flexibility for independently deployed microservices.

---

# 56. Why Saga Is More Microservice-Friendly

It allows:

```text
Independent databases
Independent transactions
Asynchronous communication
Service autonomy
```

But the price is:

```text
More complex failure handling
Eventual consistency
Compensation logic
```

---

# 57. Saga vs Database Transaction

Database transaction:

```text
Atomic
Immediate
Local
```

Saga:

```text
Distributed workflow
Multiple local commits
Compensations
Eventually consistent
```

Don't describe a Saga as:

> "A distributed database transaction."

It is a business-process coordination pattern.

---

# 58. Compensation Examples

### Inventory

```text
Reserve
→ Release
```

### Payment

```text
Authorize
→ Void authorization
```

or:

```text
Capture
→ Refund
```

depending on the payment lifecycle.

### Order

```text
Create
→ Cancel
```

---

# 59. Compensation Is Business-Specific

A generic:

```text
ROLLBACK()
```

doesn't exist across independent services.

You need explicit business operations:

```text
releaseInventory()
refundPayment()
cancelOrder()
```

---

# 60. Compensation Can Create New Events

Example:

```text
PaymentFailed
      ↓
InventoryReleaseRequested
      ↓
InventoryReleased
      ↓
OrderCancelled
```

The compensation process can itself be event-driven.

---

# 61. Saga Choreography Failure

Suppose:

```text
OrderCreated
```

is published.

Inventory receives it.

But:

```text
InventoryReserved
```

event never gets published.

The workflow may become stuck.

This is why:

```text
Outbox
Monitoring
Timeouts
Retries
DLT
```

are important.

---

# 62. Orchestrator Failure

Suppose the Saga orchestrator crashes.

A durable orchestrator should persist its state so it can resume.

Avoid storing workflow state only in memory.

---

# 63. Durable Saga State

Example:

```text
saga_instance
----------------
saga_id
order_id
current_step
status
created_at
updated_at
```

This allows recovery after process restart.

---

# 64. Duplicate Command

Suppose orchestrator sends:

```text
ReserveInventory
```

twice.

Inventory should use:

```text
reservationId
```

or another idempotency key.

This prevents duplicate reservations.

---

# 65. Duplicate Event

Suppose:

```text
InventoryReserved
```

is delivered twice.

Payment Service should not:

```text
charge twice
```

It should deduplicate or make the operation idempotent.

---

# 66. Out-of-Order Events

Distributed systems can encounter unexpected timing.

Example:

```text
OrderCancelled
```

arrives before:

```text
OrderCreated
```

depending on architecture and multiple partitions/flows.

The consumer should define how such cases are handled.

---

# 67. Ordering Requirements

Ask:

```text
Do all events need global ordering?
```

Usually:

```text
No.
```

Instead ask:

```text
Which entity requires ordering?
```

Example:

```text
orderId
```

Then partition by that key.

---

# 68. Saga and Ordering

For a single order:

```text
OrderCreated
PaymentCompleted
OrderShipped
```

may need ordering.

Use:

```text
orderId
```

as the partition key where Kafka is involved.

---

# 69. Saga and Dead Letter Topics

If a compensation event fails repeatedly:

```text
InventoryRelease
 ↓
Retry
 ↓
Retry
 ↓
DLT
```

Operations can investigate.

But putting a critical compensation into a DLT should trigger alerting; otherwise the workflow may remain inconsistent.

---

# 70. Manual Recovery

Some distributed workflows eventually require human intervention.

Example:

```text
Payment captured
Inventory release failed
```

If automation cannot safely recover:

```text
Alert operations
 ↓
Investigate
 ↓
Run approved recovery action
```

A mature system plans for this.

---

# 71. Saga Security

Events may contain sensitive business data.

Consider:

```text
Authentication
Authorization
Encryption
Access control
Minimal payloads
```

Don't publish secrets unnecessarily.

---

# 72. Don't Put Everything in Events

Avoid huge events containing:

```text
Entire customer profile
Entire order history
Internal database state
Secrets
```

Publish what consumers actually need.

---

# 73. Event Contract

An event is an integration contract.

Example:

```json
{
  "eventId": "evt-123",
  "eventType": "InventoryReserved",
  "version": 1,
  "orderId": 101,
  "reservationId": "res-789"
}
```

Consumers should depend on stable semantics.

---

# 74. Versioning Saga Events

When changing:

```text
PaymentCompleted
```

don't break existing consumers.

Possible approaches:

```text
Backward-compatible fields
Event version
New event type
Schema compatibility rules
```

---

# 75. Long-Running Sagas

A Saga can run for:

```text
seconds
minutes
hours
```

depending on the business process.

Examples:

```text
Travel booking
Insurance claim
Order fulfillment
Payment settlement
```

Long-running workflows need durable state.

---

# 76. Temporal Workflows

Workflow orchestration platforms such as Temporal can manage durable long-running workflows.

The important interview concept is:

```text
Durable workflow state
+
Retries
+
Timers
+
Failure recovery
```

You don't need to use Temporal in every Spring Boot project.

---

# 77. Saga in an E-Commerce System

Example:

```text
POST /checkout
        |
        ↓
Order Service
        |
        ↓
Create PENDING Order
        |
        ↓
OrderCreated
        |
        ↓
Inventory
        |
        ↓
InventoryReserved
        |
        ↓
Payment
        |
        ↓
PaymentCompleted
        |
        ↓
Shipping
        |
        ↓
Order CONFIRMED
```

---

# 78. Failure Path

```text
PaymentFailed
      |
      ↓
ReleaseInventory
      |
      ↓
InventoryReleased
      |
      ↓
CancelOrder
      |
      ↓
Order CANCELLED
```

---

# 79. What If Inventory Release Fails?

```text
PaymentFailed
      ↓
ReleaseInventory
      ↓
FAILED
      ↓
Retry
      ↓
Retry
      ↓
Still failed
      ↓
COMPENSATION_PENDING
      ↓
Alert
```

Don't silently mark the order as fully cancelled if inventory is still reserved.

---

# 80. Saga State Example

```text
sagaId: saga-123
orderId: 101

Steps:
✓ Order created
✓ Inventory reserved
✗ Payment failed
✓ Inventory released
✓ Order cancelled

Status:
COMPENSATED
```

---

# 81. Distributed Transaction Interview Answer

If asked:

> "How do you handle transactions across microservices?"

Answer:

> "I avoid trying to wrap multiple service databases in one local transaction. Each service owns its local transaction, and for multi-step business workflows I'd consider the Saga pattern with compensating actions. For reliable event publication, I'd use a transactional outbox. Consumers should be idempotent because retries and duplicate delivery are possible."

---

# 82. Interview Question

### "What is eventual consistency?"

Answer:

> "It's a consistency model where different parts of a distributed system can temporarily have different states, but they converge to a consistent state after successful propagation and processing."

---

# 83. Interview Question

### "What is a Saga?"

Answer:

> "A Saga is a distributed business workflow composed of local transactions. If a later step fails, compensating transactions are executed to undo the business effects of previously completed steps."

---

# 84. Interview Question

### "Saga vs 2PC?"

Answer:

> "2PC coordinates a global transaction across participants, while Saga uses independent local transactions and compensating actions. Saga is often more suitable for independently deployed microservices, but it introduces eventual consistency and more application-level failure handling."

---

# 85. Interview Question

### "Orchestration vs choreography?"

Answer:

> "Orchestration uses a central coordinator that explicitly controls the workflow. Choreography uses events where services react to each other's results without a central coordinator. Orchestration is often easier for complex workflows, while choreography can work well for simpler event-driven flows."

---

# 86. Interview Question

### "What is a compensating transaction?"

Answer:

> "It's a business operation that reverses or offsets the effect of a previously completed local transaction. For example, if inventory was reserved and payment later fails, releasing the inventory is a compensating action."

---

# 87. Interview Question

### "Why can't we simply rollback?"

Answer:

> "Because each microservice owns its own transaction and database. A local database rollback cannot automatically undo a committed transaction in another service. We need explicit compensating operations."

---

# 88. Interview Question

### "Does Saga guarantee atomicity?"

Answer:

> "No. Saga provides a way to manage distributed business consistency through local transactions and compensations. There can be temporary inconsistency and compensation itself can fail."

---

# 89. Interview Scenario

### "Payment succeeded but Order Service crashed."

Possible approach:

```text
PaymentCompleted event
```

must be durably published.

If using:

```text
Outbox
```

the event can be published after the local transaction.

The Order Service can recover its state from durable data/events.

The exact solution depends on which service owns the source of truth.

---

# 90. Interview Scenario

### "Payment succeeded but client received a timeout."

Important:

```text
Client timeout ≠ payment failure
```

The system should use:

```text
Idempotency key
Payment status
Provider transaction ID
```

so the client can safely retry/query status without creating a duplicate payment.

---

# 91. Interview Scenario

### "Order created but Kafka was unavailable."

Answer:

> "I'd persist the order and an OrderCreated outbox event in the same database transaction. Kafka publication can happen asynchronously when the broker is available."

---

# 92. Interview Scenario

### "Kafka event is delivered twice."

Answer:

> "The consumer should be idempotent. I'd use an event ID or business idempotency key and persist processed state with a unique constraint so duplicate side effects are prevented."

---

# 93. Interview Scenario

### "Saga is stuck for 30 minutes."

I'd inspect:

```text
Saga state
Current step
Correlation ID
Event history
Consumer lag
Retries
DLT
Downstream health
```

Then either:

```text
Resume
Retry
Compensate
or perform approved manual recovery
```

depending on the failure.

---

# 94. Common Mistakes

```text
❌ Calling Saga a database transaction
❌ Assuming compensation is automatic
❌ Assuming Saga guarantees strong consistency
❌ Assuming Kafka makes business operations exactly once
❌ No idempotency
❌ No durable Saga state
❌ No compensation failure handling
❌ No timeout for stuck workflows
❌ Ignoring manual recovery
❌ Using choreography for an extremely complex workflow without considering operational visibility
❌ Using 2PC everywhere
```

---

# 95. Practical Design Checklist

For a distributed business workflow, ask:

```text
□ Which service owns each piece of data?
□ What are the local transactions?
□ Which steps are synchronous?
□ Which steps are asynchronous?
□ What happens if step 2 fails?
□ What compensates step 1?
□ What if compensation fails?
□ How are retries bounded?
□ Are commands/events idempotent?
□ How is state persisted?
□ How are events published reliably?
□ Do we need an outbox?
□ Do we need an inbox?
□ What are the user-visible states?
□ How is the workflow monitored?
□ How can operators recover it?
```

---

# 96. Final Mental Model

Remember:

```text
Local transaction
→ Protects one service's database change.

Saga
→ Coordinates a multi-service business workflow.

Compensation
→ Offsets an earlier business action.

Outbox
→ Reliably connects DB changes to event publication.

Idempotency
→ Makes retries/duplicates safe.

Eventual consistency
→ Accepts temporary differences while the workflow progresses.

Durable state
→ Allows recovery after crashes.
```

---

# 97. Final Interview Answer

If asked:

> "Design a reliable checkout workflow using microservices."

Use:

> "I'd keep each service's database transaction local. The Order Service would create a pending order and an outbox event in the same transaction. An event-driven Saga could then coordinate inventory, payment and shipping. Each step would be idempotent, and failures would trigger explicit compensating actions such as releasing inventory or cancelling the order. I'd persist Saga state so the workflow can recover after crashes, use bounded retries and timeouts, and monitor stuck or failed compensations. This gives us eventual consistency without requiring a global database transaction."

---

# 98. Revision Checklist

```text
□ Distributed transaction
□ Local transaction
□ 2PC
□ 2PC prepare
□ 2PC commit
□ 2PC drawbacks
□ Eventual consistency
□ Strong vs eventual consistency
□ Saga
□ Compensation
□ Compensation failure
□ Saga orchestration
□ Saga choreography
□ Orchestration vs choreography
□ Saga state
□ Durable workflow state
□ Idempotency
□ Outbox
□ Inbox
□ Outbox + Inbox
□ Dual-write problem
□ Duplicate events
□ Duplicate commands
□ Out-of-order events
□ Ordering requirements
□ Kafka partition key
□ Retry
□ Timeout
□ Dead-letter topic
□ Manual recovery
□ User-visible workflow states
□ 200 vs 202
□ Long-running workflow
□ Distributed tracing
□ Correlation ID
□ Saga vs 2PC
□ Saga vs local transaction
□ Checkout design
□ Failure scenarios
```

---

# 99. The Interviewer's Real Test

If asked:

> "Payment failed after inventory was reserved. What happens?"

Don't answer:

```text
Rollback.
```

Instead:

```text
Payment fails
      ↓
Saga detects failure
      ↓
Release inventory
      ↓
If release succeeds
      ↓
Cancel order
      ↓
Publish final state
```

And if release fails:

```text
Release inventory
      ↓
failure
      ↓
bounded retry
      ↓
still failing
      ↓
COMPENSATION_PENDING
      ↓
alert + durable recovery
```

That distinction — **rollback vs compensation** — is one of the most important things to understand in distributed systems interviews.
