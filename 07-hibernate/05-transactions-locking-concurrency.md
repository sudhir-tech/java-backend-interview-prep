# Hibernate & JPA — File 05: Transactions, Locking & Concurrency

This file covers one of the most important areas for experienced Java backend interviews:

```text
Transactions
ACID
@Transactional
Transaction Boundaries
Propagation
Isolation
Rollback
readOnly
Flush
Commit
Optimistic Locking
Pessimistic Locking
@Version
Lost Updates
Deadlocks
Isolation Anomalies
Spring Transaction Proxy
Self-Invocation
REQUIRES_NEW
Nested Transactions
Concurrency Scenarios
Distributed Transactions
Interview Questions
Production Scenarios
```

---

# 1. What Is a Transaction?

A transaction is a logical unit of work that should be completed according to defined consistency rules.

Example:

```text
Create Order
+
Reserve Inventory
+
Record Payment
```

If these operations belong to one transactional resource and one operation fails:

```text
Rollback
```

The goal is to avoid an invalid partial state.

---

# 2. ACID

ACID stands for:

```text
Atomicity
Consistency
Isolation
Durability
```

These are fundamental database transaction properties.

---

# 3. Atomicity

Atomicity means:

```text
All operations succeed
OR
all operations are rolled back
```

Example:

```text
Debit ₹100
Credit ₹100
```

If credit fails, the debit should not remain committed when both are part of the same transaction.

---

# 4. Consistency

Consistency means a transaction moves the database from one valid state to another valid state according to its constraints and business rules.

Examples:

```text
Foreign keys
Unique constraints
NOT NULL
Business invariants
```

---

# 5. Isolation

Isolation controls how concurrent transactions interact with one another.

Example:

```text
Transaction A
      ↕
Transaction B
```

The database determines what one transaction can observe from another.

---

# 6. Durability

After a successful commit:

```text
Data survives
```

even if the application later crashes, subject to the database's durability guarantees and configuration.

---

# 7. Spring @Transactional

Spring provides:

```java
@Transactional
```

to define transaction boundaries.

Example:

```java
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order);
}
```

Conceptually:

```text
Begin transaction
      ↓
Execute business logic
      ↓
Flush changes
      ↓
Commit
```

If an appropriate exception causes rollback:

```text
Rollback
```

---

# 8. Where Should @Transactional Usually Go?

A common design is:

```text
Controller
   ↓
Service  ← @Transactional
   ↓
Repository
```

The service layer is often a good place because it represents a business operation.

---

# 9. Why Not Put @Transactional Everywhere?

Adding transactions to every method can create:

```text
Unclear boundaries
Long transactions
Unexpected propagation
Extra overhead
Harder debugging
```

Define transactions around meaningful units of business work.

---

# 10. Transaction Boundary

Example:

```java
@Transactional
public void placeOrder(...) {

    createOrder();

    reserveInventory();

    saveOrderItems();
}
```

The transaction boundary represents the business operation.

---

# 11. Local Transaction

A typical Spring Boot application may have:

```text
Application
 ↓
Service
 ↓
Hibernate
 ↓
JDBC
 ↓
One Database
```

One local database transaction can cover the persistence operations.

---

# 12. Flush and Commit

Remember:

```text
flush
→ synchronize persistence context with database

commit
→ complete database transaction
```

They are not the same thing.

---

# 13. Dirty Checking and Transaction

Example:

```java
@Transactional
public void updateProduct(Long id) {

    Product product =
        productRepository.findById(id)
                         .orElseThrow();

    product.setPrice(newPrice);
}
```

If `product` is managed:

```text
Change detected
 ↓
Dirty checking
 ↓
Flush
 ↓
UPDATE
 ↓
Commit
```

---

# 14. Rollback

Spring can roll back a transaction when configured exception rules are triggered.

By default, Spring's declarative transaction behavior commonly rolls back for:

```text
RuntimeException
Error
```

Checked exceptions do not automatically trigger rollback by default.

---

# 15. rollbackFor

If a checked exception should trigger rollback:

```java
@Transactional(
    rollbackFor = IOException.class
)
public void process() throws IOException {
    ...
}
```

Use rollback rules intentionally.

---

# 16. noRollbackFor

You can also specify exceptions that should not trigger rollback:

```java
@Transactional(
    noRollbackFor = SomeBusinessException.class
)
public void process() {
    ...
}
```

This should be used carefully.

---

# 17. Rollback Does Not Undo External Side Effects

Suppose:

```text
Database update
+
Send email
```

If the database transaction rolls back:

```text
Database update → undone
Email → may already be sent
```

A database transaction cannot automatically undo an external email/API call.

This is why distributed workflows need patterns such as:

```text
Outbox
Saga
Compensation
```

---

# 18. Transaction Propagation

Propagation defines how a method participates in an existing transaction.

Important modes:

```text
REQUIRED
REQUIRES_NEW
SUPPORTS
MANDATORY
NOT_SUPPORTED
NEVER
NESTED
```

---

# 19. REQUIRED

Default/common behavior:

```java
@Transactional(
    propagation = Propagation.REQUIRED
)
```

Meaning:

```text
Existing transaction?
    ↓
   Yes → Join it

   No → Create one
```

---

# 20. Example REQUIRED

```text
Service A
@Transactional
   ↓
Service B
@Transactional(REQUIRED)
```

Service B normally joins Service A's transaction.

Conceptually:

```text
Transaction T1
 ├── Service A
 └── Service B
```

---

# 21. REQUIRES_NEW

```java
@Transactional(
    propagation = Propagation.REQUIRES_NEW
)
```

Meaning:

```text
Existing transaction?
    ↓
   Yes
    ↓
Suspend existing transaction
    ↓
Start new transaction
```

---

# 22. REQUIRES_NEW Example

```text
Main transaction T1
        ↓
Audit operation
        ↓
REQUIRES_NEW
        ↓
Transaction T2
```

T2 commits independently.

If T1 later rolls back:

```text
T1 → rollback
T2 → can remain committed
```

---

# 23. When Would REQUIRES_NEW Be Useful?

Example:

```text
Business transaction
+
Audit record
```

If the audit record must be committed independently:

```text
Audit service → REQUIRES_NEW
```

But this should be designed carefully because it introduces independent commit semantics.

---

# 24. REQUIRES_NEW Risk

Each independent transaction may require another database connection.

Under heavy concurrency:

```text
Outer transactions hold connections
        +
REQUIRES_NEW needs another connection
        ↓
Connection pool pressure
```

Incorrect use can contribute to pool exhaustion or deadlock-like resource starvation.

---

# 25. SUPPORTS

```text
@Transactional(
    propagation = Propagation.SUPPORTS
)
```

Meaning:

```text
Transaction exists?
 → Join it

No transaction?
 → Execute without one
```

Useful when transactional context is optional.

---

# 26. MANDATORY

```text
Propagation.MANDATORY
```

Meaning:

```text
Transaction must already exist.
```

If there isn't one:

```text
Exception
```

---

# 27. NOT_SUPPORTED

Meaning:

```text
Suspend existing transaction
Execute without transaction
```

Useful for operations that should explicitly not run inside the current transaction.

---

# 28. NEVER

Meaning:

```text
If transaction exists → fail
If no transaction → execute
```

Rarely needed, but know it for interviews.

---

# 29. NESTED

`NESTED` can use a database savepoint within an existing transaction when supported by the transaction manager/resource.

Conceptually:

```text
T1
 |
 +-- Savepoint
 |
 +-- Nested operation
```

If nested work fails:

```text
Rollback to savepoint
```

The outer transaction can potentially continue.

Important:

> NESTED is not the same as REQUIRES_NEW.

---

# 30. REQUIRED vs REQUIRES_NEW

```text
REQUIRED
→ Join existing transaction

REQUIRES_NEW
→ Suspend existing transaction
→ Start independent transaction
```

This is a very common interview question.

---

# 31. Transaction Isolation

Common isolation levels:

```text
READ_UNCOMMITTED
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```

The exact behavior can vary by database.

---

# 32. READ_UNCOMMITTED

Transactions may see uncommitted changes from other transactions.

Possible anomaly:

```text
Dirty Read
```

Generally avoided for business-critical operations.

---

# 33. Dirty Read

Transaction A:

```text
Update balance = 500
```

without commit.

Transaction B reads:

```text
500
```

Then A rolls back.

B saw data that never committed.

That is:

```text
Dirty Read
```

---

# 34. READ_COMMITTED

A transaction generally sees only committed data from other transactions.

This prevents:

```text
Dirty Reads
```

but may still allow:

```text
Non-repeatable Reads
```

depending on the database.

---

# 35. Non-Repeatable Read

Transaction A:

```text
Read price = 100
```

Transaction B:

```text
Update price = 120
Commit
```

Transaction A reads again:

```text
120
```

Same transaction:

```text
First read = 100
Second read = 120
```

This is a non-repeatable read.

---

# 36. REPEATABLE_READ

The isolation level aims to make repeated reads of the same row stable within a transaction.

Database implementations can differ in exact behavior and concurrency mechanisms.

For example, MySQL/InnoDB commonly uses MVCC to provide strong repeatable-read semantics.

---

# 37. Phantom Read

Transaction A:

```text
SELECT orders WHERE total > 1000
```

Transaction B inserts another matching order and commits.

Transaction A runs the query again:

```text
Additional row appears
```

This is a phantom phenomenon.

---

# 38. SERIALIZABLE

Highest standard isolation level.

Conceptually:

```text
Transactions behave as if executed serially
```

Advantages:

```text
Strong consistency
```

Disadvantages:

```text
Lower concurrency
More locking/contention
Potentially lower throughput
```

Use only when required by business semantics.

---

# 39. Isolation Summary

Conceptually:

```text
READ_UNCOMMITTED
 ↓
Lowest isolation

READ_COMMITTED
 ↓

REPEATABLE_READ
 ↓

SERIALIZABLE
 ↓
Highest isolation
```

Higher isolation can reduce concurrency depending on implementation.

---

# 40. Spring Isolation

Example:

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
public void process() {
    ...
}
```

But the actual database's transaction/isolation capabilities matter.

---

# 41. Don't Choose Isolation Blindly

Ask:

```text
What anomaly must be prevented?
What concurrency is required?
What does the database support?
What is the workload?
```

Higher isolation is not automatically better.

---

# 42. Optimistic Locking

Optimistic locking assumes conflicts are relatively uncommon.

JPA example:

```java
@Version
private Long version;
```

---

# 43. How @Version Works

Initial:

```text
id = 10
version = 5
```

Transaction A reads:

```text
version = 5
```

Transaction B also reads:

```text
version = 5
```

A updates:

```text
version 5 → 6
```

B tries to update using:

```text
WHERE id = 10
AND version = 5
```

No row matches.

Hibernate detects the conflict.

---

# 44. OptimisticLockException

The losing transaction can receive an:

```text
OptimisticLockException
```

or a provider-specific/concurrent modification exception depending on the operation.

The application can then:

```text
Retry
Reject update
Ask user to refresh
```

depending on business requirements.

---

# 45. Optimistic Locking SQL

Conceptually:

```sql
UPDATE product
SET price = ?, version = ?
WHERE id = ?
  AND version = ?
```

If affected rows:

```text
0
```

the version likely changed.

---

# 46. Why Optimistic Locking Is Useful

Good when:

```text
Concurrent conflicts are uncommon
Reads are frequent
Holding database locks is undesirable
```

Examples:

```text
Product editing
Profile updates
Order status updates
Inventory with controlled contention
```

---

# 47. Pessimistic Locking

Pessimistic locking assumes conflicts may happen and asks the database to lock data.

JPA examples:

```text
PESSIMISTIC_READ
PESSIMISTIC_WRITE
```

---

# 48. Pessimistic Write

Conceptually:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Product> findByIdForUpdate(Long id);
```

The database may issue a row-level lock equivalent to:

```sql
SELECT ...
FOR UPDATE
```

depending on database/provider.

---

# 49. When Use Pessimistic Locking?

Useful when:

```text
Conflicts are frequent
Critical resource must be serialized
Business operation requires strong locking
```

Example:

```text
Inventory = 1
```

Two requests attempt to buy it simultaneously.

A pessimistic lock can serialize access.

---

# 50. Optimistic vs Pessimistic

```text
Optimistic
→ Detect conflict later

Pessimistic
→ Prevent/confine concurrent access through locking
```

---

# 51. Lost Update

Suppose:

```text
Initial stock = 10
```

A reads:

```text
10
```

B reads:

```text
10
```

A writes:

```text
9
```

B writes:

```text
9
```

Expected:

```text
8
```

Actual:

```text
9
```

One update was lost.

---

# 52. Preventing Lost Updates

Options:

```text
Optimistic locking
Pessimistic locking
Atomic database update
Correct transaction isolation
```

Example atomic update:

```sql
UPDATE product
SET stock = stock - 1
WHERE id = ?
  AND stock > 0
```

Then check affected rows.

---

# 53. Atomic Update Advantage

Instead of:

```text
SELECT stock
↓
calculate
↓
UPDATE stock
```

use:

```text
UPDATE stock = stock - 1
WHERE stock > 0
```

This lets the database enforce the condition atomically.

---

# 54. Inventory Example

For an e-commerce system:

```text
Request A → buy last item
Request B → buy last item
```

Possible approaches:

```text
Pessimistic lock
Optimistic @Version
Atomic conditional UPDATE
Queue/serialized processing
```

The right choice depends on throughput and business requirements.

---

# 55. Deadlock

A deadlock occurs when transactions wait for each other indefinitely.

Example:

```text
Transaction A
locks Row 1
waits for Row 2

Transaction B
locks Row 2
waits for Row 1
```

```text
A → waits for B
B → waits for A
```

---

# 56. Preventing Deadlocks

Strategies:

```text
Consistent lock ordering
Short transactions
Avoid unnecessary locks
Good indexes
Avoid user interaction inside transactions
Monitor database deadlocks
Retry safely where appropriate
```

---

# 57. Consistent Lock Ordering

Bad:

```text
Transaction A:
lock Product 1
lock Product 2

Transaction B:
lock Product 2
lock Product 1
```

Better:

```text
Always lock by ascending ID:
1 → 2
```

This reduces circular waits.

---

# 58. Keep Transactions Short

Avoid:

```text
@Transactional
 ↓
Database update
 ↓
Call external API
 ↓
Wait 5 seconds
 ↓
More DB work
 ↓
Commit
```

The transaction holds resources while waiting.

Better architecture often separates external work from database transactions using patterns such as an outbox.

---

# 59. @Transactional and External API

Bad:

```java
@Transactional
public void processOrder() {

    saveOrder();

    paymentClient.charge();

    updateOrder();
}
```

Problems:

```text
Transaction stays open
Connection held
External API may be slow
Failure semantics become complicated
```

Use an appropriate workflow instead.

---

# 60. Self-Invocation Problem

Spring's declarative transactions commonly rely on proxies.

Example:

```java
public void methodA() {
    methodB();
}

@Transactional
public void methodB() {
}
```

If `methodA()` calls `methodB()` directly on the same object:

```text
this.methodB()
```

the proxy may be bypassed.

Therefore the transactional interception may not occur as expected.

---

# 61. Why Self-Invocation Matters

Spring commonly applies:

```text
Proxy
 ↓
@Transactional method
```

But:

```text
same object
 ↓
this.method()
```

doesn't necessarily pass through the proxy.

---

# 62. Better Design for Self-Invocation

Move the transactional operation to another Spring-managed bean when appropriate:

```text
Service A
 ↓
Service B
 ↓
@Transactional
```

Now the call can cross the proxy boundary.

---

# 63. Transactional Visibility

Typical Spring usage:

```java
@Transactional
public void process() {
}
```

Public service methods are the common pattern.

Be aware that proxy-based transaction interception has limitations around:

```text
Self-invocation
Visibility
Proxy type
Final methods/classes depending on proxy mechanism
```

---

# 64. readOnly

Example:

```java
@Transactional(readOnly = true)
public User getUser(Long id) {
    ...
}
```

This communicates that the transaction is intended for reads.

It can influence transaction/provider behavior and may help optimize certain workloads.

But:

> `readOnly=true` is not a guarantee that no database write can ever occur.

It should not replace proper application design.

---

# 65. Transaction Timeout

A transaction can have a timeout:

```java
@Transactional(timeout = 5)
```

This expresses:

```text
Transaction should not run indefinitely.
```

Actual enforcement depends on transaction manager/database behavior.

---

# 66. Transaction Synchronization

Spring can coordinate actions around transaction completion.

Examples:

```text
Before commit
After commit
After rollback
```

This can be useful for:

```text
Publishing events
Clearing caches
Triggering follow-up work
```

For reliable event publication, prefer an outbox-style design when atomicity with database changes matters.

---

# 67. Database Transaction vs Distributed Transaction

Local:

```text
Service
 ↓
Database
```

Easy to manage.

Distributed:

```text
Service A → DB A
Service B → DB B
Service C → DB C
```

One local `@Transactional` cannot automatically make all three databases one atomic transaction.

---

# 68. Why Microservices Don't Share One Transaction

Microservices usually use:

```text
Database per service
```

Therefore:

```text
Order DB
Payment DB
Inventory DB
```

have separate transactions.

Use:

```text
Saga
Outbox
Events
Compensation
```

when cross-service consistency is required.

---

# 69. Transactional Outbox

Example:

```text
Order transaction
 ├── Save order
 └── Save event in outbox table
```

Both happen in one local transaction.

Then:

```text
Outbox publisher
 ↓
Kafka
 ↓
Other services
```

This avoids the classic:

```text
DB commit succeeds
but event publish fails
```

problem.

---

# 70. Saga

A Saga breaks a distributed workflow into local transactions.

Example:

```text
Create Order
 ↓
Reserve Inventory
 ↓
Process Payment
 ↓
Confirm Order
```

If payment fails:

```text
Release Inventory
 ↓
Cancel Order
```

This is compensation rather than one global database rollback.

---

# 71. Isolation vs Locking

Don't confuse:

```text
Isolation level
```

with:

```text
Explicit row locking
```

Isolation controls transaction visibility/concurrency semantics.

Locks can be used by the database/provider to enforce those semantics or explicitly requested by the application.

---

# 72. Transaction Boundary Example

Good:

```text
placeOrder()
 ├── validate
 ├── create order
 ├── reserve local inventory
 └── save state
```

Avoid:

```text
placeOrder()
 ├── DB work
 ├── REST API
 ├── sleep
 ├── file upload
 └── DB work
```

inside one long transaction.

---

# 73. Retry and Transactions

Transient database failures may be retryable.

But don't blindly retry every transaction.

Consider:

```text
Was the operation committed?
Is it idempotent?
Could duplicate side effects occur?
Is the exception transient?
```

Retries must be designed together with idempotency.

---

# 74. Idempotency

Suppose:

```text
POST /payments
```

is retried.

Without idempotency:

```text
Charge ₹100
Charge ₹100 again
```

Use an idempotency key:

```text
request-id = abc123
```

and persist/check it appropriately.

---

# 75. Transaction + Idempotency

A robust payment workflow may use:

```text
Idempotency record
+
Payment state
+
Transaction
```

so retries don't create duplicate business effects.

---

# 76. Optimistic Locking + Retry

If:

```text
OptimisticLockException
```

occurs:

```text
Read latest version
 ↓
Reapply safe operation
 ↓
Retry
```

only if the business operation is safe to retry.

Never blindly retry operations that have external side effects.

---

# 77. Pessimistic Lock + Timeout

Database locks can block.

Configure appropriate:

```text
Lock timeout
Transaction timeout
```

where supported.

Monitor:

```text
Lock wait time
Deadlocks
Connection pool usage
```

---

# 78. Connection Pool Interaction

Transactions usually hold a database connection while active.

Long transactions:

```text
More time holding connections
 ↓
Pool exhaustion
 ↓
Requests wait
 ↓
Latency increases
```

This is why transaction duration matters.

---

# 79. HikariCP

Spring Boot commonly uses:

```text
HikariCP
```

as the JDBC connection pool.

Important metrics:

```text
Active connections
Idle connections
Pending threads
Connection acquisition time
Pool size
```

---

# 80. Transaction and Connection Pool Scenario

Suppose:

```text
Pool = 10 connections
```

10 requests start long transactions.

Now:

```text
Request 11
 ↓
Needs connection
 ↓
Waits
```

If the transactions remain open because of slow external calls:

```text
Pool starvation
```

---

# 81. Transaction Boundary and Performance

Good transaction:

```text
Begin
 ↓
Small amount of DB work
 ↓
Commit
```

Bad transaction:

```text
Begin
 ↓
Heavy computation
 ↓
Network call
 ↓
User interaction
 ↓
More DB work
 ↓
Commit
```

---

# 82. Interview Question

### What is @Transactional?

Answer:

> "`@Transactional` defines a transactional boundary around a method or class. Spring coordinates transaction begin, commit and rollback according to the configured transaction manager and propagation/isolation rules."

---

# 83. Interview Question

### What are ACID properties?

Answer:

> "Atomicity ensures all-or-nothing execution, consistency preserves valid database state, isolation controls concurrent transaction visibility, and durability ensures committed data survives failures according to the database's durability guarantees."

---

# 84. Interview Question

### What is propagation?

Answer:

> "Transaction propagation defines how a method participates in an existing transaction, such as joining it with REQUIRED or creating an independent transaction with REQUIRES_NEW."

---

# 85. Interview Question

### REQUIRED vs REQUIRES_NEW?

Answer:

> "REQUIRED joins an existing transaction or creates one if none exists. REQUIRES_NEW suspends the current transaction and starts an independent one."

---

# 86. Interview Question

### What is transaction isolation?

Answer:

> "Isolation defines how concurrent transactions interact and what changes they can observe. Common levels are READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ and SERIALIZABLE."

---

# 87. Interview Question

### What is optimistic locking?

Answer:

> "Optimistic locking assumes conflicts are relatively uncommon and detects them using a version field, commonly with JPA's `@Version`."

---

# 88. Interview Question

### What is pessimistic locking?

Answer:

> "Pessimistic locking explicitly locks database rows so competing transactions cannot modify them concurrently in the same way. It's useful when contention is high and serialization is required."

---

# 89. Interview Question

### Optimistic vs pessimistic locking?

Answer:

> "Optimistic locking detects conflicts when updating, while pessimistic locking prevents or restricts concurrent access by acquiring database locks. I choose based on contention, throughput and business consistency requirements."

---

# 90. Interview Question

### What is a lost update?

Answer:

> "A lost update happens when concurrent transactions read the same value and later overwrite each other's changes. Optimistic locking, pessimistic locking or atomic database updates can prevent it."

---

# 91. Interview Question

### What is a deadlock?

Answer:

> "A deadlock occurs when transactions hold resources that each other needs and both wait indefinitely. Consistent lock ordering, short transactions and appropriate retry strategies help reduce the risk."

---

# 92. Interview Question

### Why doesn't @Transactional roll back an email?

Answer:

> "A database transaction controls the database resource. An external email service is outside that transaction, so its side effect cannot automatically be rolled back. For reliable workflows, patterns such as the transactional outbox can help."

---

# 93. Interview Question

### Why doesn't @Transactional work during self-invocation?

Answer:

> "Spring's declarative transaction management commonly uses proxies. A direct call from one method to another on the same object can bypass the proxy, so the transactional interceptor may not run."

---

# 94. Interview Scenario

### "Two users edit the same product."

Solution:

```text
@Version
```

Then:

```text
User A → version 5 → update → version 6
User B → version 5 → update fails
```

The application can ask B to refresh or retry safely.

---

# 95. Interview Scenario

### "Two users buy the last item."

Possible solutions:

```text
Pessimistic lock
Optimistic @Version
Atomic conditional UPDATE
Queue/serialization
```

Choose based on contention and required throughput.

---

# 96. Interview Scenario

### "Payment API takes 8 seconds."

Bad:

```text
@Transactional
 ↓
Save order
 ↓
Call payment API
 ↓
Wait 8 sec
 ↓
Commit
```

Why bad:

```text
Connection held
Transaction held
Pool pressure
Long locks
```

Consider asynchronous workflows or state transitions.

---

# 97. Interview Scenario

### "An audit record must remain even if the main transaction rolls back."

Possible:

```text
REQUIRES_NEW
```

But carefully consider:

```text
Connection pool
Failure semantics
Audit reliability
```

For high-reliability event/audit workflows, an outbox or append-only event design may be more appropriate.

---

# 98. Interview Scenario

### "Transaction succeeds but Kafka publish fails."

Use:

```text
Transactional Outbox
```

Flow:

```text
DB transaction
 ├── Business data
 └── Outbox event
        ↓
Committed atomically
        ↓
Publisher
        ↓
Kafka
```

---

# 99. Interview Scenario

### "Database deadlocks appear in production."

Investigate:

```text
Which transactions?
Which rows?
Lock order?
Transaction duration?
Indexes?
Concurrent access pattern?
```

Then:

```text
Standardize lock ordering
Reduce transaction size
Optimize queries
Retry transient deadlocks carefully
```

---

# 100. Production Checklist

```text
□ Transaction boundaries are clear
□ Transactions are short
□ External calls are not unnecessarily inside transactions
□ Isolation level is intentional
□ Rollback rules are understood
□ Optimistic locking used where appropriate
□ Pessimistic locking used only when needed
□ Deadlocks monitored
□ Connection pool monitored
□ Long transactions monitored
□ Bulk operations handle persistence-context state
□ Retries are idempotent
□ Distributed workflows use Saga/Outbox where appropriate
```

---

# 101. Transaction Mental Model

Remember:

```text
@Transactional
      ↓
Begin transaction
      ↓
Load/modify managed entities
      ↓
Hibernate dirty checking
      ↓
Flush
      ↓
Database
      ↓
Commit
```

With concurrency:

```text
Concurrent requests
      ↓
Isolation
+
Locking
+
Versioning
      ↓
Consistent result
```

---

# 102. Final Interview Answer

If asked:

> "How do you handle transactions and concurrency in a Spring Boot application?"

Say:

> "I define transactions around meaningful service-level business operations and keep them short. I choose the isolation level based on the consistency requirements rather than always using the highest level. For concurrent updates, I usually prefer optimistic locking with `@Version` when conflicts are uncommon, and use pessimistic locking or atomic database updates when contention requires stronger serialization. I also avoid keeping transactions open during slow external calls and use patterns like the transactional outbox for reliable cross-service events."

---

# 103. Revision Checklist

```text
□ Transaction
□ ACID
□ Atomicity
□ Consistency
□ Isolation
□ Durability
□ @Transactional
□ Transaction boundary
□ Flush vs commit
□ Rollback
□ rollbackFor
□ noRollbackFor
□ REQUIRED
□ REQUIRES_NEW
□ SUPPORTS
□ MANDATORY
□ NOT_SUPPORTED
□ NEVER
□ NESTED
□ Isolation levels
□ Dirty read
□ Non-repeatable read
□ Phantom read
□ SERIALIZABLE
□ Optimistic locking
□ @Version
□ OptimisticLockException
□ Pessimistic locking
□ PESSIMISTIC_READ
□ PESSIMISTIC_WRITE
□ Lost update
□ Atomic updates
□ Deadlocks
□ Lock ordering
□ Transaction duration
□ Self-invocation
□ Spring proxies
□ readOnly
□ Timeout
□ Connection pools
□ HikariCP
□ Distributed transactions
□ Saga
□ Outbox
□ Idempotency
□ Retry
□ Interview scenarios
```

---

# 104. What Comes Next

```text
File 06 → Hibernate Performance & Optimization
```

Next we will cover:

```text
SQL performance
N+1 optimization
Indexes
Batch operations
JDBC batching
EntityManager batching
Persistence-context size
Flush/clear
Bulk operations
Projections
Connection pool tuning
Hibernate statistics
Second-level cache
Query plan analysis
Memory usage
Large datasets
Production troubleshooting
Performance interview questions
```

The key interview lesson is:

> **Transactions are not just about adding `@Transactional`. A strong backend developer understands transaction boundaries, rollback behavior, isolation, locking, persistence-context behavior, connection-pool impact and how these choices affect concurrency and production reliability.**
