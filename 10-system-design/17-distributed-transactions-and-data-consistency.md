# System Design — File 17: Distributed Transactions & Data Consistency

Distributed applications often need one business operation to update multiple services or databases.

```text
Order Service
   |
   +--> Order DB
   +--> Inventory
   +--> Payment
   +--> Shipping
```

The challenge is handling **partial success, retries, duplicates, failures and consistency** without assuming that all services share one database transaction.

---

## 1. Local vs Distributed Transactions

A local database transaction can provide:

```text
BEGIN
  update A
  update B
COMMIT
```

A distributed workflow may span:

```text
Order DB
Payment DB
Inventory DB
```

A normal database transaction cannot automatically make all three atomic.

---

## 2. Why Distributed Transactions Are Difficult

Example:

```text
Create Order       ✓
Reserve Inventory  ✓
Process Payment    X
```

Now the system must decide:

```text
Release inventory?
Cancel order?
Retry payment?
Keep order pending?
```

Failure handling must be explicitly designed.

---

## 3. ACID

Traditional database transactions provide:

```text
Atomicity
Consistency
Isolation
Durability
```

Microservice workflows often combine local ACID transactions with distributed coordination patterns.

---

## 4. Saga Pattern

A Saga breaks a distributed workflow into local transactions.

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

If a later step fails, use a **compensating action**.

Example:

```text
Payment fails
     |
     v
Release Inventory
     |
     v
Cancel Order
```

Compensation is a new business operation, not a database rollback.

---

## 5. Saga Choreography

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
     |
PaymentCompleted
     |
     v
Shipping
```

Advantages:

```text
Loose coupling
Event-driven
No central coordinator
```

Disadvantages:

```text
Complex workflows become difficult to understand
Debugging is harder
Event dependencies can become complicated
```

---

## 6. Saga Orchestration

A central orchestrator coordinates the workflow.

```text
             Saga Orchestrator
              /      |                    v       v        v
        Inventory Payment  Shipping
```

Advantages:

```text
Clear workflow
Centralized failure handling
Easier visibility
```

Trade-off:

```text
Orchestrator becomes important infrastructure
```

---

## 7. Choreography vs Orchestration

| Choreography | Orchestration |
|---|---|
| Events drive workflow | Coordinator drives workflow |
| No central controller | Central workflow component |
| Good for simpler flows | Good for complex flows |
| Can become hard to trace | Easier workflow visibility |

---

## 8. Dual-Write Problem

Suppose a service needs to:

```text
Update database
+
Publish Kafka event
```

Bad:

```text
DB update
   |
   v
Kafka publish
```

If Kafka fails:

```text
DB updated
Event missing
```

This is the **dual-write problem**.

---

## 9. Transactional Outbox

Write the business change and event record in the same local database transaction.

```text
Database Transaction
      |
      +--> Business Data
      |
      +--> Outbox Event
```

Then publish asynchronously:

```text
Outbox
   |
   v
Publisher
   |
   v
Kafka
```

If Kafka is unavailable, the outbox event remains and can be retried.

---

## 10. Outbox Example

```text
BEGIN

INSERT order

INSERT outbox_event

COMMIT
```

Later:

```text
Outbox Publisher -> Kafka
```

The business update and event record are committed atomically in the same database.

Outbox does **not** magically provide end-to-end exactly-once delivery.

---

## 11. Idempotency

An operation is idempotent when repeating it produces the same business result.

Example:

```text
PUT /users/42
name = "Sudhir"
```

Repeated requests leave the resource in the same state.

This is especially important for retryable side effects such as payments.

---

## 12. Idempotency Keys

Example:

```http
POST /payments
Idempotency-Key: abc123
```

The server stores the result for that key.

If the client retries:

```text
Same key
   |
   v
Return existing result
```

instead of charging again.

---

## 13. At-Most-Once vs At-Least-Once

### At-most-once

```text
0 or 1 delivery
```

Possible problem:

```text
Consumer crashes
   |
   v
Message may be lost
```

### At-least-once

```text
1 or more deliveries
```

Duplicates are possible.

Therefore consumers should often be idempotent.

---

## 14. Exactly-Once

Exactly-once behavior is difficult to guarantee across independent systems.

A practical architecture often uses:

```text
At-least-once delivery
+
Idempotent processing
+
Durable state
```

This can provide effectively-once business behavior.

---

## 15. Idempotent Consumer

Consumer receives:

```text
eventId = 123
```

Check:

```text
Already processed?
```

If yes:

```text
Ignore duplicate
```

If no:

```text
Process
Record event ID
```

A unique constraint on the event ID can help enforce deduplication.

---

## 16. Inbox Pattern

The Inbox pattern records received events.

```text
Message
   |
   v
Inbox / Processed Events
   |
   v
Business Transaction
```

It is mainly used for reliable and deduplicated consumption.

Remember:

```text
Outbox -> reliable publishing
Inbox  -> reliable/deduplicated consumption
```

---

## 17. Optimistic Locking

Optimistic locking assumes conflicts are relatively uncommon.

Example:

```text
Product
stock = 10
version = 5
```

Update:

```sql
UPDATE product
SET stock = 9, version = 6
WHERE id = 101
  AND version = 5;
```

If zero rows are updated:

```text
Someone else changed the record.
```

The application can retry or return a conflict.

---

## 18. Pessimistic Locking

Pessimistic locking blocks competing updates.

Conceptually:

```sql
SELECT ...
FROM product
WHERE id = 101
FOR UPDATE;
```

Useful when:

```text
Contention is high
Critical shared state is involved
Transactions are short
```

Trade-offs:

```text
Blocking
Lower concurrency
Deadlocks
```

---

## 19. Optimistic vs Pessimistic

| Optimistic | Pessimistic |
|---|---|
| Detects conflict | Prevents competing updates |
| Good for lower contention | Good for higher contention |
| Higher concurrency | More blocking |
| Requires conflict handling | Can cause deadlocks |

---

## 20. Lost Update

Two users read:

```text
stock = 10
```

A writes:

```text
9
```

B writes:

```text
8
```

One update can overwrite the other.

Solutions:

```text
Optimistic locking
Pessimistic locking
Atomic database updates
```

---

## 21. Atomic Update

Instead of:

```text
Read stock
Calculate stock - 1
Write stock
```

use an atomic database operation where appropriate:

```sql
UPDATE product
SET stock = stock - 1
WHERE id = 101
  AND stock > 0;
```

Then check the affected-row count.

---

## 22. Distributed Locks

A distributed lock coordinates multiple application instances.

```text
Instance A ----Instance B -----+--> Distributed Lock
Instance C ----/
```

Possible technologies:

```text
Redis
Database
ZooKeeper
etcd
```

Use a lock only when the problem actually requires distributed coordination.

---

## 23. Distributed Lock Risks

Potential problems:

```text
Lock holder crashes
Lease expires
Network partition
Stale owner continues working
```

For critical workflows, fencing tokens can protect downstream resources from stale owners.

---

## 24. Fencing Tokens

Example:

```text
Lock acquisition #1 -> token 10
Lock acquisition #2 -> token 11
```

If an old owner with token 10 continues after token 11 has been issued:

```text
Request(token=10)
       |
       v
Rejected
```

This prevents stale owners from performing protected operations.

---

## 25. Consistency Models

Common models:

```text
Strong consistency
Eventual consistency
Causal consistency
Read-after-write consistency
```

Choose based on business requirements.

---

## 26. Strong Consistency

After a successful write, reads reflect the latest state according to the chosen consistency model.

Useful for:

```text
Critical financial state
Some inventory constraints
Authorization decisions
```

Trade-offs can include:

```text
Coordination
Latency
Availability during partitions
```

---

## 27. Eventual Consistency

Replicas may temporarily disagree:

```text
Primary = 100
Replica = 95
```

Later:

```text
Replica = 100
```

Useful when scalability, availability and low latency are more important than immediate consistency.

---

## 28. Read-After-Write

A user:

```text
Update profile
     |
     v
Immediately read profile
```

A lagging replica may return old data.

Possible solutions:

```text
Read from primary
Session-aware routing
Synchronous replication
Causal/version mechanisms
```

---

## 29. Monotonic Reads

A reader should not observe data going backward.

Bad:

```text
Read 1 -> version 10
Read 2 -> version 8
```

A consistency model with monotonic reads prevents this type of regression for a given reader/session.

---

## 30. Causal Consistency

If:

```text
A causes B
```

observers should not see:

```text
B before A
```

Example:

```text
Post created
    |
    v
Comment created
```

Seeing the comment before the post violates the expected causal relationship.

---

## 31. Versioning

A version number can detect stale updates.

```text
Record version = 10
```

Client sends:

```text
version = 10
```

Server accepts only if the current version is still 10.

After success:

```text
version = 11
```

This is the basis of optimistic concurrency control.

---

## 32. Compare-and-Set

Update only if the current value equals the expected value.

```text
if version == 10:
    update to version 11
else:
    conflict
```

Useful for:

```text
Concurrency control
Atomic state changes
Distributed coordination
```

---

## 33. Quorum

For replicated systems:

```text
N = total replicas
W = write acknowledgements
R = read responses
```

A common relationship is:

```text
R + W > N
```

which can create read/write overlap under appropriate assumptions.

Real systems have additional details, so don't present this equation as a universal guarantee.

---

## 34. Conflict Resolution

Eventually consistent systems can have concurrent conflicting updates.

Possible strategies:

```text
Last-write-wins
Version vectors
Application-specific merge
CRDTs
```

The correct approach depends on the data model.

---

## 35. Last-Write-Wins

A simple policy:

```text
Latest timestamp wins
```

It is easy to implement but can lose meaningful concurrent updates.

Use it only when that behavior is acceptable.

---

## 36. Saga State Machine

Complex workflows can use explicit states:

```text
ORDER_CREATED
      |
      v
INVENTORY_RESERVED
      |
      v
PAYMENT_COMPLETED
      |
      v
SHIPPING_CREATED
```

Failure states:

```text
PAYMENT_FAILED
ORDER_CANCELLED
```

Explicit state makes recovery and observability easier.

---

## 37. Retry vs Compensation

Retry when:

```text
Failure is likely transient
```

Compensate when:

```text
A previous successful business action must be reversed
```

Example:

```text
Payment timeout
```

Do not blindly charge again. First determine whether the original operation succeeded.

---

## 38. Distributed Transaction Checklist

For every cross-service workflow ask:

```text
What data must change?
Which service owns each piece?
Can the operation remain local?
What happens if a later step fails?
Can operations be retried?
Are operations idempotent?
What compensates successful previous steps?
Do we need Outbox?
Do consumers need deduplication?
What consistency level is required?
How do we recover after partial failure?
```

---

## 39. Interview — What Is a Saga?

> "A Saga breaks a distributed transaction into a sequence of local transactions. If a later step fails, compensating actions undo the business effects of earlier successful steps."

---

## 40. Interview — What Is the Outbox Pattern?

> "The service writes its business change and an outbox event in the same local database transaction. A separate publisher then sends the event to the broker and retries if publishing fails. This solves the common database-plus-message dual-write problem."

---

## 41. Interview — Exactly Once vs At Least Once?

> "At-least-once delivery allows duplicates, so consumers should be idempotent. Exactly-once behavior is difficult to guarantee end-to-end, so I focus on effectively-once business effects using idempotency, deduplication and durable state."

---

## 42. Interview — Optimistic vs Pessimistic Locking?

> "Optimistic locking detects conflicts using a version or similar mechanism and works well when contention is low. Pessimistic locking blocks competing transactions and can be useful when conflicts are frequent, but it reduces concurrency and can introduce deadlocks."

---

## 43. Interview — How Would You Prevent Double Payment?

> "I'd use an idempotency key, persist payment state durably and make retries check the existing transaction before creating another charge. A timeout should not automatically be treated as payment failure."

---

## 44. Interview — How Would You Keep DB and Kafka Consistent?

> "I'd use the transactional outbox pattern: update the business data and outbox record in one database transaction, then publish the outbox event asynchronously with retries."

---

## 45. Practical Scenario — Order Created but Event Was Lost

Problem:

```text
Order DB -> success
Kafka    -> failure
```

Solution:

```text
DB transaction
   |
   +--> Order
   +--> Outbox event
```

The publisher retries the event later.

---

## 46. Practical Scenario — Kafka Delivers an Event Twice

Use:

```text
eventId
processed-event storage
unique constraint
idempotent business operation
```

Second delivery should not create another side effect.

---

## 47. Practical Scenario — Payment Times Out

Do not blindly issue another payment.

```text
Check payment status
       |
       v
Already successful?
   |            |
  Yes           No
   |            |
Use result   Retry safely
             with same
             idempotency key
```

---

## 48. Practical Scenario — Inventory Reserved but Payment Fails

Saga:

```text
Order created
      |
      v
Inventory reserved
      |
      v
Payment fails
      |
      v
Release inventory
      |
      v
Cancel / fail order
```

---

## 49. Practical Scenario — Two Users Buy the Last Product

One option is an atomic stock update:

```sql
UPDATE product
SET stock = stock - 1
WHERE id = 101
  AND stock > 0;
```

If affected rows = 0:

```text
Out of stock / conflict
```

For more complex workflows, use suitable locking or reservation mechanisms.

---

## 50. One-Minute Interview Answer

### "How would you handle a distributed order workflow?"

> "I'd keep each service responsible for its own local transaction and coordinate the workflow with a Saga. For reliable events, I'd use the transactional outbox pattern. I'd make important operations idempotent so retries don't create duplicate effects, and use compensation for successful steps that need to be reversed. For example, if inventory is reserved but payment fails, I'd release the inventory and move the order to a failed or cancelled state. I'd also model the workflow with explicit states so partial failures and recovery are observable."

---

## 51. Final Checklist

```text
□ ACID
□ Local vs distributed transactions
□ Distributed transaction challenges
□ Saga
□ Saga compensation
□ Choreography
□ Orchestration
□ Dual-write problem
□ Transactional Outbox
□ Idempotency
□ Idempotency keys
□ At-most-once
□ At-least-once
□ Exactly-once challenges
□ Idempotent consumers
□ Inbox pattern
□ Optimistic locking
□ Pessimistic locking
□ Lost updates
□ Atomic updates
□ Distributed locks
□ Lock leases
□ Fencing tokens
□ Strong consistency
□ Eventual consistency
□ Read-after-write
□ Monotonic reads
□ Causal consistency
□ Versioning
□ Compare-and-set
□ Quorum
□ Conflict resolution
□ Last-write-wins
□ Saga state machines
□ Retry vs compensation
□ Distributed transaction interview scenarios
```

---

## 52. Key Takeaway

> **Distributed consistency is mostly about handling partial success safely. Keep transactions local where possible, use Saga and compensation for multi-service workflows, use Outbox for reliable event publication, and design important operations to tolerate retries and duplicates.**

**File 17 complete.**
