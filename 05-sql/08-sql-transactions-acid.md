# SQL — Transactions & ACID Properties

Transactions are one of the most important SQL and backend interview topics.

They connect database theory directly to real Java/Spring Boot development.

You should understand:

```text
Transactions
ACID
COMMIT
ROLLBACK
Atomicity
Consistency
Isolation
Durability
Isolation levels
Dirty reads
Non-repeatable reads
Phantom reads
Locks
Deadlocks
Spring @Transactional
Propagation
Rollback
```

---

# 1. What Is a Transaction?

A transaction is a group of database operations treated as one logical unit of work.

Example:

```text
Create order
    ↓
Create order items
    ↓
Reduce inventory
    ↓
Commit
```

If an important operation fails:

```text
Rollback
```

so the database does not end up in an invalid intermediate state.

---

# 2. Simple Transaction Example

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

Both updates belong to the same transaction.

---

# 3. ROLLBACK

If something goes wrong:

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

ROLLBACK;
```

The transaction's changes are undone according to the database's transaction semantics.

---

# 4. COMMIT

`COMMIT` makes the transaction's changes permanent.

```sql
START TRANSACTION;

UPDATE products
SET stock = stock - 1
WHERE id = 10;

COMMIT;
```

After commit, the changes are no longer part of the active transaction.

---

# 5. ACID

ACID stands for:

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

These properties describe important guarantees of database transactions.

---

# 6. Atomicity

Atomicity means:

```text
All operations succeed
OR
the transaction is rolled back
```

Example:

```text
Transfer ₹1000

Debit account A
Credit account B
```

You don't want:

```text
Debit succeeds
Credit fails
```

while keeping the debit.

Atomicity treats the transaction as one unit.

---

# 7. Consistency

Consistency means a committed transaction must leave the database satisfying its defined integrity rules and constraints.

Example:

```text
balance >= 0
```

if the application/database enforces that invariant.

Another example:

```text
Foreign key must reference a valid parent
```

A successful transaction should move the database from one valid state to another valid state.

---

# 8. Isolation

Isolation controls how concurrent transactions interact with each other's changes.

Suppose:

```text
Transaction A
Transaction B
```

are running at the same time.

Isolation determines what one transaction can observe from another.

---

# 9. Durability

Durability means that once a transaction is successfully committed, its committed changes are expected to survive failures according to the database's durability guarantees.

Conceptually:

```text
COMMIT
  ↓
Database acknowledges success
  ↓
System crashes
  ↓
Committed data survives recovery
```

The exact durability mechanisms are database and configuration dependent.

---

# 10. ACID Interview Answer

> ACID stands for Atomicity, Consistency, Isolation and Durability. Atomicity ensures a transaction is treated as one unit, consistency maintains database rules, isolation controls concurrent transaction visibility, and durability ensures committed changes survive failures according to the database's guarantees.

---

# 11. Transaction Lifecycle

A simplified lifecycle:

```text
BEGIN
  ↓
ACTIVE
  ↓
Operations
  ↓
COMMIT
  ↓
COMPLETED
```

Or:

```text
BEGIN
  ↓
Operations
  ↓
ERROR
  ↓
ROLLBACK
```

---

# 12. Transaction Boundary

A transaction boundary defines which operations belong to the same transaction.

Example:

```text
Place Order
   |
   +-- Create Order
   |
   +-- Create Order Items
   |
   +-- Update Inventory
   |
   +-- Save Payment Record
```

These may need to be treated as one transaction depending on the business requirements.

---

# 13. Spring Boot Transaction

In Spring Boot, you commonly use:

```java
@Transactional
public void placeOrder() {
    // database operations
}
```

Spring manages the transaction boundary around the method.

---

# 14. Example Service

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order order) {

        orderRepository.save(order);

        inventoryService.reduceStock(order);

        paymentRepository.save(payment);
    }
}
```

The exact design depends on whether payment is local database work or an external service call.

---

# 15. What Happens with @Transactional?

Conceptually:

```text
Method called
    ↓
Spring starts transaction
    ↓
Method executes
    ↓
Success
    ↓
Commit
```

If the transaction is configured to roll back for an encountered exception:

```text
Method throws exception
    ↓
Rollback
```

The exact rollback rules matter.

---

# 16. Runtime Exceptions and Rollback

Spring's default rollback behavior generally rolls back on unchecked exceptions such as:

```text
RuntimeException
Error
```

Checked exceptions do not automatically trigger rollback under the default rule.

You can customize this.

---

# 17. rollbackFor

Example:

```java
@Transactional(rollbackFor = Exception.class)
public void processOrder() throws Exception {
    // operations
}
```

This tells Spring to roll back for matching exceptions, including checked exceptions.

Use explicit rollback rules when the business behavior requires them.

---

# 18. noRollbackFor

You can also specify exceptions that should not trigger rollback.

Example:

```java
@Transactional(
    rollbackFor = Exception.class,
    noRollbackFor = BusinessWarningException.class
)
public void process() {
    // ...
}
```

Use this carefully because rollback behavior should match the business transaction semantics.

---

# 19. Isolation Levels

Common isolation levels are:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

They represent different trade-offs between:

```text
Concurrency
Consistency
Locking
Performance
```

The exact implementation and default level depend on the database.

---

# 20. READ UNCOMMITTED

Transactions may be able to observe changes that have not yet been committed by another transaction.

This permits:

```text
Dirty reads
```

It provides weaker isolation.

---

# 21. Dirty Read

Transaction A:

```text
UPDATE balance = 500
```

but hasn't committed.

Transaction B reads:

```text
balance = 500
```

Then A rolls back.

B observed data that was never committed.

That is a:

```text
Dirty Read
```

---

# 22. READ COMMITTED

A transaction generally sees only committed data from other transactions.

This prevents dirty reads.

However, the same query executed twice can potentially return different committed values if another transaction commits a change between the two reads.

This is associated with:

```text
Non-repeatable reads
```

---

# 23. Non-Repeatable Read

Transaction A:

```text
SELECT balance
```

gets:

```text
1000
```

Transaction B changes balance:

```text
1000 → 500
```

and commits.

Transaction A runs the same query again:

```text
500
```

The same row returned different committed values during the transaction.

---

# 24. REPEATABLE READ

A transaction generally provides stronger repeatability for rows it has already read than READ COMMITTED.

Depending on the database implementation, snapshot/MVCC behavior and locking can also affect what is observed.

This level is stronger than READ COMMITTED but does not universally mean that every possible concurrent phenomenon is prevented in exactly the same way across databases.

---

# 25. Phantom Read

A phantom read occurs when a repeated range query observes a different set of rows because another transaction inserted or removed rows matching the condition.

Example:

Transaction A:

```sql
SELECT *
FROM orders
WHERE amount > 10000;
```

Suppose it sees:

```text
5 rows
```

Transaction B inserts another matching order and commits.

Transaction A repeats the query and sees:

```text
6 rows
```

The additional row is a phantom.

---

# 26. SERIALIZABLE

`SERIALIZABLE` provides the strongest standard isolation level.

It aims to make concurrent transactions behave as if they were executed serially.

Trade-offs can include:

```text
More blocking
More contention
Lower concurrency
Potential transaction retries/errors depending on implementation
```

Use it when the consistency requirement justifies the cost.

---

# 27. Isolation Summary

Conceptually:

```text
READ UNCOMMITTED
↓
weakest

READ COMMITTED
↓
prevents dirty reads

REPEATABLE READ
↓
stronger repeatability

SERIALIZABLE
↓
strongest standard isolation
```

But exact behavior varies by database.

---

# 28. Read Phenomena

Common interview table:

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| READ UNCOMMITTED | Possible | Possible | Possible |
| READ COMMITTED | Prevented | Possible | Possible |
| REPEATABLE READ | Prevented | Prevented for normal repeated row reads | Database-dependent behavior |
| SERIALIZABLE | Prevented | Prevented | Prevented |

Important:

> Do not memorize this table without knowing the database. MVCC and implementation details can change exact behavior.

---

# 29. MVCC

Many modern relational databases use:

```text
MVCC
```

which means:

```text
Multi-Version Concurrency Control
```

The database can maintain multiple row versions so readers and writers can often operate concurrently with less blocking.

PostgreSQL and InnoDB are common examples with MVCC-based behavior.

---

# 30. Why MVCC Helps

Conceptually:

```text
Transaction A reads
        ↓
Transaction B updates
        ↓
A can continue reading an appropriate visible version
```

This can reduce reader-writer blocking compared with a purely lock-based approach.

The exact visibility rules depend on the database and isolation level.

---

# 31. Locks

Databases can use locks to coordinate concurrent operations.

Common conceptual categories:

```text
Shared lock
Exclusive lock
```

The exact lock implementation is database-specific.

---

# 32. Shared Lock

A shared lock generally allows compatible readers but prevents conflicting modifications while the lock is held.

Conceptually:

```text
Transaction A → reads row
Transaction B → may also read
Transaction C → cannot perform conflicting update
```

Exact behavior depends on the database and locking mode.

---

# 33. Exclusive Lock

An exclusive lock is used for operations that modify data.

Conceptually:

```text
Transaction A → updates row
Transaction B → conflicting access waits/fails depending on conditions
```

Again, actual behavior depends on the database engine.

---

# 34. Pessimistic Locking

Pessimistic locking assumes:

```text
Conflict is likely
```

So the application/database acquires a lock before proceeding.

In JPA:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Product> findById(Long id);
```

This can be useful for inventory-like scenarios where concurrent updates must be carefully coordinated.

---

# 35. Optimistic Locking

Optimistic locking assumes:

```text
Conflicts are relatively uncommon
```

A version column is commonly used.

Example:

```java
@Version
private Long version;
```

When an update occurs:

```text
Read version = 5
        ↓
Modify entity
        ↓
Update WHERE id = ? AND version = 5
        ↓
If successful → version becomes 6
```

If another transaction already changed the row:

```text
version != 5
```

and the update can fail with an optimistic-locking conflict.

---

# 36. Pessimistic vs Optimistic Locking

Pessimistic:

```text
Lock first
↓
Perform work
↓
Release lock
```

Optimistic:

```text
Read version
↓
Perform work
↓
Check version during update
↓
Detect conflict
```

Use based on workload and business requirements.

---

# 37. Deadlock

A deadlock occurs when transactions wait for each other indefinitely.

Example:

```text
Transaction A locks Row 1
Transaction B locks Row 2

A waits for Row 2
B waits for Row 1
```

Result:

```text
A waits for B
B waits for A
```

---

# 38. Deadlock Diagram

```text
Transaction A
     |
 locks Row 1
     |
 waits for Row 2
     ↑
     |
Transaction B
     |
 locks Row 2
     |
 waits for Row 1
```

Neither can continue.

The database detects the cycle and typically aborts one transaction.

---

# 39. Preventing Deadlocks

Common strategies:

```text
Acquire locks in a consistent order
Keep transactions short
Avoid unnecessary locks
Avoid user/network calls inside DB transactions
Use appropriate indexes
Reduce contention
Retry transient deadlock failures where appropriate
```

---

# 40. Transaction Duration

Long transactions are dangerous because they can hold:

```text
Locks
Connections
Resources
```

for longer.

Bad pattern:

```text
BEGIN TRANSACTION

database update

call external API
(wait 5 seconds)

another database update

COMMIT
```

The transaction may hold resources while waiting on the external service.

---

# 41. External APIs and Transactions

Avoid assuming:

```java
@Transactional
public void process() {
    databaseUpdate();
    paymentClient.callExternalService();
}
```

makes the external API call part of the same database transaction.

It does not.

The database transaction and external service have separate consistency boundaries.

---

# 42. Distributed Transactions

If multiple independent systems must participate in one logical operation, you may need patterns such as:

```text
Saga
Outbox pattern
Event-driven workflows
Compensating actions
```

Distributed transactions are significantly more complex than a single database transaction.

---

# 43. Transaction + Order Placement

Imagine:

```text
Create Order
Create Order Items
Reduce Stock
```

These database operations may belong in one transaction:

```java
@Transactional
public void placeOrder() {
    createOrder();
    createOrderItems();
    reduceStock();
}
```

If stock reduction fails:

```text
Rollback order
Rollback order items
```

assuming all operations participate in the same database transaction.

---

# 44. Transaction Propagation

Spring supports transaction propagation.

Common modes:

```text
REQUIRED
REQUIRES_NEW
SUPPORTS
MANDATORY
NOT_SUPPORTED
NEVER
NESTED
```

For interviews, understand `REQUIRED` and `REQUIRES_NEW` especially well.

---

# 45. REQUIRED

`REQUIRED` is the default propagation behavior in Spring.

Conceptually:

```text
Existing transaction?
    ↓
Yes → join it

No → create a new one
```

Example:

```java
@Transactional
public void placeOrder() {
    orderService.saveOrder();
}
```

If `saveOrder()` uses `REQUIRED`, it normally participates in the existing transaction.

---

# 46. REQUIRES_NEW

`REQUIRES_NEW` suspends an existing transaction and starts a separate transaction.

Example:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAudit() {
    auditRepository.save(...);
}
```

Conceptually:

```text
Main transaction
      ↓
suspend
      ↓
Audit transaction
      ↓
commit
      ↓
resume main transaction
```

This can be useful when audit/log persistence should commit independently.

It also requires careful thought about connection usage and failure behavior.

---

# 47. REQUIRED vs REQUIRES_NEW

`REQUIRED`:

```text
Join existing transaction
```

`REQUIRES_NEW`:

```text
Suspend existing transaction
Start independent transaction
```

Interview answer:

> `REQUIRED` participates in the existing transaction if one exists, while `REQUIRES_NEW` suspends the current transaction and starts an independent one.

---

# 48. Self-Invocation Problem

A common Spring interview question.

Example:

```java
public void methodA() {
    methodB();
}

@Transactional
public void methodB() {
}
```

Calling:

```text
this.methodB()
```

may bypass the Spring proxy.

Therefore the transaction behavior you expect may not be applied.

This is because Spring's common declarative transaction model uses proxies.

---

# 49. Spring Transaction Proxy

Conceptually:

```text
Caller
  ↓
Spring Proxy
  ↓
@Transactional method
  ↓
Transaction
```

But:

```text
methodA()
  ↓
this.methodB()
```

does not necessarily go through the proxy again.

This is why self-invocation is a common `@Transactional` pitfall.

---

# 50. Transactional Service Layer

A common architecture:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Transaction boundaries are often placed at the service/use-case layer because one business operation may involve multiple repository calls.

Example:

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(...) {
        ...
    }
}
```

---

# 51. Read-Only Transactions

Spring supports:

```java
@Transactional(readOnly = true)
```

Example:

```java
@Transactional(readOnly = true)
public Order getOrder(Long id) {
    return orderRepository.findById(id)
        .orElseThrow();
}
```

It communicates that the transaction is intended for reads.

Depending on the database/JPA configuration, it may provide optimization hints.

It does not universally mean:

```text
"writes are physically impossible"
```

---

# 52. Transaction Isolation in Spring

You can specify isolation:

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
public void process() {
}
```

But don't set isolation arbitrarily.

The right level depends on:

```text
Database
Concurrency
Business invariants
Performance
Locking behavior
```

---

# 53. Propagation in Spring

Example:

```java
@Transactional(
    propagation = Propagation.REQUIRED
)
public void placeOrder() {
}
```

For independent work:

```java
@Transactional(
    propagation = Propagation.REQUIRES_NEW
)
public void saveAudit() {
}
```

Know the behavior before using these settings.

---

# 54. Rollback Example

Suppose:

```java
@Transactional
public void transfer() {

    debitAccount();

    creditAccount();

    throw new RuntimeException("Something failed");
}
```

If the exception reaches the transaction boundary and matches the rollback rules:

```text
debit
↓
credit
↓
exception
↓
rollback
```

The database transaction is rolled back.

---

# 55. Catching Exceptions Inside Transactions

Potential problem:

```java
@Transactional
public void process() {
    try {
        repository.save(...);
        throw new RuntimeException();
    } catch (RuntimeException e) {
        log.error("failed", e);
    }
}
```

If the exception is caught and the method completes normally, Spring may not roll back automatically based on that exception.

If the operation should fail the transaction, you need to propagate the exception or explicitly mark rollback according to the design.

---

# 56. Transaction Boundary and Exception Handling

Think:

```text
Exception escapes transaction method
        ↓
Spring sees it
        ↓
Rollback rules apply
```

If you swallow the exception:

```text
Exception caught
        ↓
Method returns normally
        ↓
Commit may occur
```

This is a common production bug.

---

# 57. Transactional Testing

A transaction test may look like:

```java
@SpringBootTest
@Transactional
class OrderServiceTest {
}
```

But test transaction behavior depends on the test framework and configuration.

Don't assume a test transaction behaves exactly like a production service transaction.

---

# 58. Transaction and Connection Pool

A database transaction generally uses a database connection.

If transactions are held for too long:

```text
Connection remains occupied
↓
Pool has fewer available connections
↓
Requests wait
↓
Latency increases
```

This is why long transactions can contribute to connection pool exhaustion.

---

# 59. HikariCP and Transactions

In a Spring Boot application using HikariCP:

```text
Request
 ↓
@Transactional
 ↓
Connection acquired
 ↓
SQL operations
 ↓
Commit/Rollback
 ↓
Connection returned to pool
```

The connection pool manages connections; the transaction manager manages transaction boundaries.

They are related but different responsibilities.

---

# 60. Transaction Isolation vs Locking

These concepts are related but not identical.

Isolation:

```text
What concurrent changes can a transaction observe?
```

Locking:

```text
How does the database coordinate conflicting access?
```

MVCC can provide snapshot visibility while locks coordinate writes and other operations.

The exact behavior is database-specific.

---

# 61. Lost Update

A lost update can occur when two transactions read the same value and then overwrite each other's changes.

Example:

```text
Initial stock = 10

Transaction A reads 10
Transaction B reads 10

A writes 9
B writes 9
```

Expected after two purchases:

```text
8
```

But final value may become:

```text
9
```

depending on the access pattern.

---

# 62. Preventing Lost Updates

Possible approaches:

```text
Atomic SQL update
Pessimistic locking
Optimistic locking
Appropriate transaction isolation
```

Atomic update:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = ?
AND stock > 0;
```

Then check affected rows.

This can be a powerful inventory pattern.

---

# 63. Optimistic Locking for Inventory

Entity:

```java
@Version
private Long version;
```

Two transactions:

```text
A reads version 5
B reads version 5

A updates → version 6
B attempts update with version 5
        ↓
conflict
```

The application can detect the conflict and retry/reject the operation.

---

# 64. Atomic Database Update

Instead of:

```text
SELECT stock
↓
Java subtracts 1
↓
UPDATE stock
```

you can sometimes use:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = ?
AND stock > 0;
```

Then:

```text
affected rows = 1
→ stock reduced

affected rows = 0
→ product unavailable / condition failed
```

This reduces the race window.

---

# 65. Transaction + Inventory

A robust inventory workflow may be:

```text
BEGIN
  ↓
Reserve/reduce inventory
  ↓
Create order
  ↓
Create order items
  ↓
COMMIT
```

The exact design depends on whether inventory is in the same database and whether distributed services are involved.

---

# 66. Deadlock Retry

Deadlocks are often transient.

A production application may:

```text
Detect deadlock
↓
Rollback failed transaction
↓
Retry operation
```

Retries should be:

```text
Bounded
Carefully designed
Idempotent where appropriate
```

Do not blindly retry every database error.

---

# 67. Idempotency

Idempotency means repeating the same request does not create unintended additional effects.

Important for APIs such as:

```text
POST payment
POST order
```

A transaction alone does not automatically make an API idempotent.

A common approach is an:

```text
Idempotency key
```

stored with a unique constraint.

---

# 68. Transaction vs Idempotency

Transaction:

```text
Protects a database unit of work
```

Idempotency:

```text
Protects against repeated requests producing duplicate effects
```

They solve different problems.

---

# 69. Transaction vs Lock

Transaction:

```text
Unit of database work
```

Lock:

```text
Concurrency-control mechanism
```

A transaction can involve locks, MVCC snapshots or both depending on the database and operations.

---

# 70. Transaction vs Connection

Connection:

```text
Communication channel to database
```

Transaction:

```text
Logical unit of database work
```

One connection can execute transaction work, but these are not the same concept.

---

# 71. Transaction vs Batch

Batching:

```text
Send multiple operations efficiently
```

Transaction:

```text
Define atomicity/consistency boundary
```

You can use:

```text
Batch + Transaction
```

together.

---

# 72. ACID in E-Commerce

Imagine:

```text
Place Order
```

Operations:

```text
1. Validate order
2. Reduce inventory
3. Create order
4. Create order items
5. Commit
```

Atomicity:

```text
All local database changes succeed or roll back.
```

Consistency:

```text
Constraints and business invariants remain valid.
```

Isolation:

```text
Concurrent orders don't incorrectly interfere.
```

Durability:

```text
Committed order survives database recovery according to configured guarantees.
```

---

# 73. Interview: What Is a Transaction?

> A transaction is a logical unit of database work. Multiple operations are executed together so the database can commit them as a unit or roll them back according to the transaction rules.

---

# 74. Interview: Explain ACID

> ACID stands for Atomicity, Consistency, Isolation and Durability. Atomicity handles all-or-nothing changes, consistency maintains database rules, isolation controls concurrent transaction visibility, and durability protects committed changes from loss after failures according to the database's guarantees.

---

# 75. Interview: What Is Dirty Read?

> A dirty read happens when one transaction reads data written by another transaction before that transaction commits, and the other transaction later rolls back.

---

# 76. Interview: What Is Non-Repeatable Read?

> It happens when a transaction reads the same row twice and gets different committed values because another transaction modified and committed that row between the reads.

---

# 77. Interview: What Is Phantom Read?

> A phantom read occurs when a transaction repeats a range query and sees a different set of matching rows because another transaction inserted or removed rows that match the condition.

---

# 78. Interview: What Is Isolation Level?

> An isolation level defines how much one transaction is isolated from concurrent transactions and controls phenomena such as dirty reads, non-repeatable reads and phantom reads. The exact behavior depends on the database implementation.

---

# 79. Interview: What Is @Transactional?

> `@Transactional` tells Spring to apply transaction management around a method. Typically, Spring starts or joins a transaction, executes the method, and commits or rolls back based on the configured transaction and rollback rules.

---

# 80. Interview: REQUIRED vs REQUIRES_NEW

> `REQUIRED` joins an existing transaction or creates one if none exists. `REQUIRES_NEW` suspends the existing transaction and starts an independent transaction.

---

# 81. Interview: Why Put @Transactional on Service Layer?

> A service method often represents one business operation and may call multiple repositories. Putting the transaction boundary around that operation lets those database changes succeed or fail together.

---

# 82. Interview: What Is a Deadlock?

> A deadlock occurs when two or more transactions hold resources that the others need and each waits for the other to release them. The database usually detects the cycle and aborts one transaction.

---

# 83. Interview: How Do You Prevent Deadlocks?

> I keep transactions short, acquire resources in a consistent order, avoid unnecessary locks, use appropriate indexes and retry transient deadlock failures when the operation is safe to retry.

---

# 84. Interview: Optimistic vs Pessimistic Locking

> Optimistic locking detects conflicts using a version or similar mechanism and is useful when conflicts are relatively uncommon. Pessimistic locking acquires database locks upfront and is useful when concurrent modification must be prevented during the operation.

---

# 85. Interview: Does @Transactional Make External API Calls Transactional?

> No. `@Transactional` manages the configured transaction resource, typically the database. An external HTTP service has its own consistency boundary. For distributed workflows, patterns such as Saga or the Outbox pattern may be more appropriate.

---

# 86. Interview: What Happens if I Catch an Exception Inside @Transactional?

> If I catch an exception and don't rethrow it, the transaction may complete normally and commit depending on the transaction state. If the business operation must roll back, I need to propagate the exception or explicitly mark the transaction for rollback.

---

# 87. Interview: What Is the Self-Invocation Problem?

> Spring's declarative transaction support commonly works through proxies. If one method calls another `@Transactional` method through `this`, the call may bypass the proxy, so the expected transactional interception may not occur.

---

# 88. Interview: Why Avoid Long Transactions?

> Long transactions can hold database resources and connections for too long, increase lock contention and reduce connection-pool availability. I try to keep transaction boundaries focused on the actual business operation.

---

# 89. Transaction Checklist

```text
□ Transaction
□ BEGIN
□ COMMIT
□ ROLLBACK
□ ACID
□ Atomicity
□ Consistency
□ Isolation
□ Durability
□ READ UNCOMMITTED
□ READ COMMITTED
□ REPEATABLE READ
□ SERIALIZABLE
□ Dirty read
□ Non-repeatable read
□ Phantom read
□ MVCC
□ Shared lock
□ Exclusive lock
□ Optimistic locking
□ Pessimistic locking
□ Deadlock
□ Lost update
□ @Transactional
□ rollback rules
□ rollbackFor
□ readOnly
□ REQUIRED
□ REQUIRES_NEW
□ Self-invocation
□ Transaction boundary
□ Connection pool interaction
□ Distributed transactions
□ Saga
□ Outbox
□ Idempotency
```

---

# 90. Final Mental Model

Think about transactions as:

```text
Business Operation
       ↓
Transaction Boundary
       ↓
Multiple DB Operations
       ↓
Success → COMMIT
       ↓
Failure → ROLLBACK
```

Then ask four ACID questions:

```text
Atomicity
→ Did everything succeed or fail together?

Consistency
→ Did we preserve database rules and invariants?

Isolation
→ What can concurrent transactions see?

Durability
→ Does committed data survive failures?
```

For Spring Boot:

```text
Controller
   ↓
Service
   ↓
@Transactional
   ↓
Repository
   ↓
Database
```

For concurrency:

```text
Multiple requests
       ↓
Transactions
       ↓
Locks / MVCC
       ↓
Isolation
       ↓
Consistent result
```

> **Interview shortcut:** If you can explain ACID with a money-transfer or order-placement example, then explain isolation levels, deadlocks, optimistic locking and `@Transactional`, you have covered most of the transaction questions asked in Java backend interviews.
