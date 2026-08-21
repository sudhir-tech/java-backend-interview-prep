# Redis — File 05: Distributed Locking, Rate Limiting & Atomic Operations

This file focuses on Redis features that commonly appear in backend interviews and production systems:

```text
Atomic operations
Race conditions
MULTI / EXEC
WATCH
Lua scripts
Distributed locks
SET NX EX
Lock ownership
Safe release
Lease expiration
Fencing tokens
Redlock concepts
Rate limiting
Fixed window
Sliding window
Token bucket
Idempotency
Inventory reservation
Distributed job coordination
Spring Boot examples
Production failure scenarios
```

---

# 1. Why Atomicity Matters

Suppose two application instances update the same value:

```text
Instance A
Instance B
     ↓
   Redis
```

If the operation is not atomic:

```text
A reads 10
B reads 10
A writes 9
B writes 9
```

One update is lost.

This is a race condition.

---

# 2. Atomic Redis Commands

Individual Redis commands are designed to execute atomically with respect to other Redis commands.

Examples:

```text
INCR
DECR
HINCRBY
ZINCRBY
SET NX
```

Whenever possible, prefer one atomic command over:

```text
GET
+
application calculation
+
SET
```

---

# 3. Read-Modify-Write Problem

Bad:

```text
GET stock:101
```

Application:

```text
stock = stock - 1
```

Then:

```text
SET stock:101 9
```

Concurrent requests can overwrite one another.

---

# 4. Atomic Counter

Instead:

```text
DECR stock:101
```

is one Redis operation.

However, this alone does not guarantee:

```text
stock never becomes negative
```

For conditional business rules, you need stronger logic.

---

# 5. Conditional Atomic Update

Requirement:

```text
Only decrement stock if stock > 0
```

This is not simply:

```text
DECR
```

because:

```text
stock = 0
```

could become:

```text
-1
```

Use an atomic conditional operation, commonly a Lua script.

---

# 6. Lua Script for Inventory

Conceptually:

```lua
local stock = redis.call('GET', KEYS[1])

if not stock then
    return -1
end

if tonumber(stock) <= 0 then
    return 0
end

redis.call('DECR', KEYS[1])
return 1
```

Meaning:

```text
-1 → key missing
 0 → out of stock
 1 → successfully reserved
```

The exact script should be adapted to the application's data model.

---

# 7. Why Lua Is Atomic

Redis executes a Lua script as a server-side operation.

Conceptually:

```text
Application
   ↓
Lua script
   ↓
Redis
   ↓
multiple commands
```

Other Redis commands don't interleave inside the script's execution in the same way as separate client round trips.

---

# 8. Lua Best Practices

Keep scripts:

```text
Short
Deterministic
Efficient
Bounded
```

Avoid:

```text
Huge loops
Large data scans
Long-running computation
```

A long script can delay other Redis commands.

---

# 9. MULTI / EXEC

Redis transactions provide:

```text
MULTI
EXEC
DISCARD
WATCH
```

Example:

```text
MULTI
INCR counter
INCR counter
EXEC
```

The commands are queued and then executed sequentially.

---

# 10. Transaction Limitation

Redis transactions do not provide the same rollback semantics as relational database transactions.

If commands execute and a later runtime command fails:

```text
Earlier successful commands
```

are not automatically rolled back.

---

# 11. WATCH

`WATCH` provides optimistic concurrency control.

Conceptually:

```text
WATCH stock:101
GET stock:101

calculate

MULTI
SET stock:101 9
EXEC
```

If another client changes:

```text
stock:101
```

before `EXEC`:

```text
EXEC
 ↓
aborted
```

The application can retry.

---

# 12. Optimistic vs Pessimistic Coordination

```text
WATCH
→ Optimistic

Distributed lock
→ Pessimistic-style coordination
```

Optimistic control assumes conflicts are relatively uncommon.

Locks intentionally prevent competing work from entering a critical section.

---

# 13. Distributed Lock

A distributed lock coordinates multiple application instances.

Example:

```text
Application A
Application B
Application C
       ↓
     Redis
       ↓
    Lock key
```

Only one instance should own the lock at a time.

---

# 14. Why Distributed Locks?

Use cases include:

```text
Scheduled jobs
Inventory reservation
Duplicate processing prevention
Leader-like coordination
Resource initialization
Single-flight work
```

But don't use locks automatically.

Ask:

```text
Can the operation be idempotent instead?
Can optimistic concurrency solve it?
Does the critical section really need exclusive access?
```

---

# 15. Basic Lock Acquisition

A common Redis primitive is:

```text
SET lock:order:101 unique-token NX EX 30
```

Meaning:

```text
SET
→ Store value

NX
→ Only if key doesn't exist

EX 30
→ Expire after 30 seconds
```

If successful:

```text
Lock acquired
```

If not:

```text
Lock already exists
```

---

# 16. Why Use a Unique Token?

Suppose:

```text
Instance A
```

owns:

```text
lock:order:101
```

with token:

```text
token-A
```

The token identifies the owner.

This matters when releasing the lock safely.

---

# 17. Unsafe Lock Release

Don't blindly do:

```text
DEL lock:order:101
```

because:

```text
A acquires lock
 ↓
A pauses
 ↓
Lock expires
 ↓
B acquires lock
 ↓
A resumes
 ↓
A executes DEL
```

Now:

```text
B's lock
```

has been deleted by A.

---

# 18. Safe Lock Release

The application should verify:

```text
lock value == my unique token
```

and only then delete the key.

This comparison + delete must itself be atomic.

A Lua script is a common solution.

---

# 19. Safe Unlock Script

Conceptually:

```lua
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
```

Where:

```text
KEYS[1]
→ lock key

ARGV[1]
→ owner token
```

This prevents one owner from deleting another owner's lock.

---

# 20. Lock TTL

Always consider:

```text
Lock expiration
```

Why?

If the owner crashes:

```text
Application A
 ↓
Lock acquired
 ↓
Crash
```

Without expiration:

```text
Lock remains forever
```

TTL provides eventual recovery.

---

# 21. Lock TTL Problem

Suppose:

```text
TTL = 30 seconds
```

but the critical operation takes:

```text
60 seconds
```

Then:

```text
30 sec
 ↓
lock expires
 ↓
another instance acquires it
```

while A is still running.

Now two instances can execute the critical section.

---

# 22. Lease Extension

For long-running work, a lock may need a lease-renewal mechanism.

Conceptually:

```text
Acquire lock
 ↓
Work
 ↓
Renew TTL
 ↓
Work
 ↓
Release
```

Renewal must verify ownership.

---

# 23. Lock Renewal Risk

A renewal mechanism introduces additional failure cases:

```text
Network partition
Application pause
GC pause
Redis unavailable
Renewal thread failure
```

Therefore:

> A distributed lock is not automatically a correctness guarantee.

The business operation should ideally also be safe under retries or duplicate execution.

---

# 24. Fencing Tokens

For high-safety distributed locking, fencing tokens can help.

Conceptually:

```text
Acquire lock
 ↓
Get increasing token
 ↓
Perform operation with token
```

Downstream resource accepts only increasingly valid tokens.

Example:

```text
Token 41
Token 42
```

If stale process A has:

```text
Token 41
```

and B has:

```text
Token 42
```

the downstream system can reject A's stale operation.

---

# 25. Why Fencing Tokens Matter

They protect against:

```text
Lock expires
+
old process continues running
```

A Redis lock alone cannot always prevent a paused process from continuing after its lease expires.

Fencing moves part of the correctness check to the resource being protected.

---

# 26. Redlock

Redlock is a Redis distributed-lock algorithm designed around multiple independent Redis instances.

Conceptually:

```text
Client
 ↓
Redis A
Redis B
Redis C
Redis D
Redis E
```

The client attempts to acquire the lock across multiple nodes and uses timing/quorum rules.

---

# 27. Redlock Interview Position

You should know:

```text
Redlock exists
```

but don't simply say:

> "Redlock guarantees distributed locking perfectly."

Distributed locking involves:

```text
Network failures
Clock/timing assumptions
Process pauses
Partitions
Resource semantics
```

For high-value correctness, fencing tokens and idempotency can be more important than the lock algorithm alone.

---

# 28. Lock vs Idempotency

Suppose:

```text
Process A
Process B
```

both attempt:

```text
Create order
```

Instead of relying only on a lock, use:

```text
Idempotency key
```

Example:

```text
payment:request:abc123
```

Then repeated requests produce the same logical result.

---

# 29. Why Idempotency Is Powerful

A distributed lock says:

```text
Only one should run now.
```

Idempotency says:

```text
Repeated execution produces one logical effect.
```

In distributed systems:

```text
Idempotency
```

is often a stronger safety property.

---

# 30. Idempotency with Redis

Example:

```text
SET payment:req:abc123 PROCESSING NX EX 300
```

If the key already exists:

```text
Request already seen
```

The application can then:

```text
Return existing result
or
check processing state
```

depending on the workflow.

---

# 31. Idempotency State

Instead of only storing:

```text
true
```

store state such as:

```text
PROCESSING
COMPLETED
FAILED
```

and potentially:

```text
response
timestamp
```

This allows better recovery.

---

# 32. Rate Limiting

Redis is commonly used for API rate limiting.

Requirement:

```text
100 requests/minute per user
```

Redis can maintain:

```text
rate:user:101
```

and count requests.

---

# 33. Fixed Window Rate Limiter

Example:

```text
Key:
rate:user:101:2026082113
```

where the suffix represents the current time window.

Then:

```text
INCR key
```

and:

```text
EXPIRE key 60
```

If count:

```text
> 100
```

reject the request.

---

# 34. Fixed Window Problem

Suppose limit:

```text
100/minute
```

At:

```text
12:00:59
```

client sends 100 requests.

Then at:

```text
12:01:00
```

another 100 requests.

Potentially:

```text
200 requests
```

arrive in about one second across the boundary.

This is called the boundary burst problem.

---

# 35. Sliding Window

A sliding window considers recent requests continuously.

Possible Redis structure:

```text
Sorted Set
```

Store:

```text
timestamp
+
request ID
```

Then:

```text
Remove old timestamps
Count recent timestamps
Add current timestamp
```

This can provide more accurate rate limiting.

---

# 36. Sliding Window Race

The sequence:

```text
remove old
count
add new
```

contains multiple operations.

Concurrent requests can race.

Use:

```text
Lua script
```

when these operations must be atomic.

---

# 37. Token Bucket

Token bucket maintains:

```text
Available tokens
Refill rate
Maximum capacity
```

Each request consumes:

```text
1 token
```

If no token is available:

```text
Reject
```

This allows controlled bursts up to bucket capacity.

---

# 38. Token Bucket Example

Configuration:

```text
Capacity = 100
Refill = 10 tokens/sec
```

A user can:

```text
Burst up to 100
```

then continue around:

```text
10 requests/sec
```

as tokens refill.

---

# 39. Fixed Window vs Sliding Window vs Token Bucket

```text
Fixed Window
→ Simple
→ Cheap
→ Boundary bursts

Sliding Window
→ More accurate
→ More state/work

Token Bucket
→ Controls sustained rate
→ Allows bounded bursts
```

Choose based on API requirements.

---

# 40. Rate Limiting Key

Possible dimensions:

```text
rate:user:101
rate:ip:192.0.2.1
rate:api-key:abc
rate:endpoint:orders
```

You can also combine dimensions:

```text
rate:user:101:orders
```

---

# 41. Rate Limiting Response

When rejected:

```text
HTTP 429 Too Many Requests
```

Useful response information can include:

```text
Retry-After
```

if appropriate.

---

# 42. Atomic Fixed-Window Rate Limiting

A simple safe approach is to perform:

```text
increment
+
set expiration when needed
```

inside a Lua script.

Conceptually:

```lua
local count = redis.call("INCR", KEYS[1])

if count == 1 then
    redis.call("EXPIRE", KEYS[1], ARGV[1])
end

return count
```

Then application checks:

```text
count <= limit
```

---

# 43. Why Lua?

Without atomic execution:

```text
Request A:
INCR

Request B:
INCR

A:
EXPIRE

B:
EXPIRE
```

The final TTL can become dependent on race timing.

A script can define the operation as one Redis-side unit.

---

# 44. Distributed Job Lock

Suppose:

```text
3 application instances
```

all run:

```text
DailyReportJob
```

Without coordination:

```text
Job executes 3 times
```

A lock can coordinate:

```text
job:daily-report
```

Only one instance proceeds.

---

# 45. Scheduled Job Pattern

Conceptually:

```text
tryAcquireLock()
       ↓
    success?
     /    \
   No      Yes
   |        |
return     run job
            ↓
        release lock
```

Use a short but appropriate lease.

---

# 46. Job Lock Failure

Suppose:

```text
A acquires lock
A crashes
```

TTL allows:

```text
Lock expires
B can acquire
```

But if the job is not idempotent:

```text
A may have partially completed
B may repeat side effects
```

Therefore:

```text
Lock + idempotency
```

is often safer.

---

# 47. Inventory Reservation

Example:

```text
stock = 5
```

Request:

```text
reserve 1
```

A robust Redis operation should atomically:

```text
Check stock
 ↓
Decrement
 ↓
Create reservation state
```

If these must be one logical atomic operation, a Lua script can coordinate Redis-side state.

---

# 48. Inventory Caveat

Do not assume:

```text
Redis stock
=
final inventory truth
```

If MySQL is the system of record:

```text
Redis
→ fast coordination/cache

MySQL
→ durable source of truth
```

The complete consistency model must be designed.

---

# 49. Atomic Reservation

Conceptually:

```text
if stock >= quantity:
    decrement stock
    create reservation
    return success
else:
    return failure
```

This should avoid:

```text
GET
+
application calculation
+
SET
```

when concurrent requests can oversell inventory.

---

# 50. Distributed Lock in Spring Boot

For simple custom coordination, the application can use:

```text
StringRedisTemplate
```

to perform:

```text
SET key token NX EX ttl
```

However, production distributed-lock libraries can provide additional handling.

The important interview skill is understanding the underlying Redis mechanism.

---

# 51. Lock Acquisition Concept

Conceptual Java:

```java
Boolean acquired =
    redisTemplate.opsForValue()
        .setIfAbsent(
            lockKey,
            token,
            Duration.ofSeconds(30)
        );
```

If:

```text
acquired == true
```

the caller owns the lock.

---

# 52. Safe Unlock

Do not:

```java
redisTemplate.delete(lockKey);
```

unless ownership has been safely verified.

Use an atomic ownership-check-and-delete operation, commonly implemented with Lua.

---

# 53. Lock Token

Generate a unique token:

```java
String token = UUID.randomUUID().toString();
```

Store:

```text
lock:key → token
```

Only the owner holding that token should release it.

---

# 54. Lock Acquisition Failure

If:

```text
setIfAbsent() == false
```

possible behavior:

```text
Return
Retry after delay
Queue work
Fail fast
```

Do not blindly spin in a tight loop.

---

# 55. Lock Retry

If retrying:

```text
Retry with backoff
```

For example:

```text
50ms
100ms
200ms
```

with jitter.

Avoid:

```text
while(true) {
    tryLock();
}
```

because it can overload Redis.

---

# 56. Lock TTL Selection

TTL should consider:

```text
Typical operation duration
Maximum expected duration
Failure recovery
Network latency
Application pauses
```

Too short:

```text
Lock expires while work continues
```

Too long:

```text
Crash causes long wait
```

---

# 57. Lock Extension

For unpredictable long-running operations:

```text
Lease renewal
```

may be required.

But renewal itself must be:

```text
Ownership-aware
Failure-aware
Bounded
```

---

# 58. Fencing Token Example

Imagine:

```text
A gets token 10
```

A pauses.

Lock expires.

```text
B gets token 11
```

B writes to the protected database with:

```text
fencingToken = 11
```

Later A resumes with:

```text
fencingToken = 10
```

The database/resource rejects:

```text
10 < 11
```

This protects against stale owners.

---

# 59. Idempotency Key vs Lock

### Lock

Prevents concurrent execution.

### Idempotency key

Prevents duplicate logical effects.

Often:

```text
Lock
+
Idempotency
```

provides stronger protection than either alone.

---

# 60. Payment Example

Client sends:

```text
POST /payments
Idempotency-Key: abc123
```

Store:

```text
payment:idempotency:abc123
```

with state:

```text
PROCESSING
```

If the same request arrives again:

```text
Already processing
```

After completion:

```text
COMPLETED
+
response
```

The exact state machine depends on business requirements.

---

# 61. Payment Failure

Suppose:

```text
Payment succeeds
```

but application crashes before storing:

```text
COMPLETED
```

A retry may happen.

This shows why:

```text
Redis idempotency
```

alone may not be sufficient for external payment providers.

The payment provider itself should support:

```text
Idempotency
```

or the system needs a reliable reconciliation strategy.

---

# 62. Distributed Systems Principle

Redis can coordinate:

```text
Fast state
Locks
Counters
Rate limits
Idempotency
```

But external side effects require:

```text
Durability
Retry safety
Idempotency
Reconciliation
```

Redis does not magically make distributed operations transactional.

---

# 63. Common Lock Mistake

Bad answer:

> "I'll use Redis lock so only one server can process the request."

Better:

> "I'll use a Redis lease with an owner token and safe release, but I'll also make the operation idempotent because the lock can expire or the process can pause."

This is a much stronger interview answer.

---

# 64. Common Rate-Limit Mistake

Bad:

```text
GET counter
if counter < limit
    INCR
```

Race:

```text
A reads 99
B reads 99
A increments
B increments
```

Two requests may pass incorrectly.

Use:

```text
Atomic increment
+
atomic limit logic
```

when required.

---

# 65. Common Inventory Mistake

Bad:

```text
if stock > 0
    DECR stock
```

The condition and decrement must be evaluated atomically.

Use:

```text
Lua
```

or another atomic server-side pattern.

---

# 66. Common Lock Mistake

Bad:

```text
SET lock
```

without:

```text
NX
```

Two instances can overwrite the lock.

Use:

```text
SET lock token NX EX ttl
```

---

# 67. Common Lock Mistake

Bad:

```text
SET lock token NX
```

without expiration.

If the owner crashes:

```text
Lock remains
```

Use a lease/TTL where appropriate.

---

# 68. Common Lock Mistake

Bad:

```text
DEL lock
```

without ownership verification.

A stale process could delete another owner's lock.

---

# 69. Common Lock Mistake

Bad:

```text
TTL = 5 seconds
Job = 30 seconds
```

The lock may expire while the job is still running.

Design the lease around realistic execution and failure semantics.

---

# 70. Common Distributed Systems Mistake

Bad assumption:

```text
Lock acquired
=
operation guaranteed safe
```

Not necessarily.

The process can:

```text
Pause
Crash
Lose network
Continue after lease expiry
```

Use:

```text
Idempotency
Fencing
Transactional source of truth
```

where required.

---

# 71. Interview Question

### How do you implement a Redis distributed lock?

Answer:

> "I'd use an atomic `SET key uniqueToken NX EX ttl` to acquire the lock. The unique token identifies the owner. On release, I'd atomically check that the stored token matches my token before deleting the key. I'd also consider lease renewal, idempotency and fencing tokens for long-running or high-value operations."

---

# 72. Interview Question

### Why NX?

Answer:

> "`NX` ensures the lock key is created only when it doesn't already exist, preventing another owner from overwriting an existing lock."

---

# 73. Interview Question

### Why EX?

Answer:

> "`EX` gives the lock a lease duration, allowing recovery if the process holding the lock crashes."

---

# 74. Interview Question

### Why use a unique token?

Answer:

> "It identifies the lock owner so that a process can safely release only the lock it actually owns."

---

# 75. Interview Question

### Why can't I simply DEL the lock?

Answer:

> "Because the lock might have expired and another process may have acquired the same key. The old process could then delete the new owner's lock. Ownership verification must happen atomically with deletion."

---

# 76. Interview Question

### What is a fencing token?

Answer:

> "A fencing token is a monotonically increasing value associated with lock ownership. The protected resource can reject operations from older tokens, preventing a stale process whose lease expired from performing unsafe writes."

---

# 77. Interview Question

### What is Redlock?

Answer:

> "Redlock is an algorithm for acquiring Redis-based locks across multiple independent Redis instances using quorum and timing rules. I would still consider network failures, process pauses and the need for fencing or idempotency rather than treating the lock as an absolute correctness guarantee."

---

# 78. Interview Question

### How would you implement API rate limiting?

Answer:

> "For a simple fixed window I'd maintain a Redis counter with an expiration. For stronger concurrency guarantees I'd perform the increment and expiration setup atomically, often with Lua. For smoother traffic control I'd consider sliding-window or token-bucket algorithms."

---

# 79. Interview Question

### Fixed window vs sliding window?

Answer:

> "Fixed window is simpler and cheaper but can allow boundary bursts. Sliding window gives more accurate control over recent requests but requires more state and processing."

---

# 80. Interview Question

### What is token bucket?

Answer:

> "Token bucket maintains a bucket of tokens that refill at a configured rate. Each request consumes a token, allowing bounded bursts up to the bucket capacity while controlling the long-term request rate."

---

# 81. Interview Question

### Redis lock or database lock?

Answer:

> "It depends on what is being protected. If the resource of record is in a relational database, database transactions or optimistic/pessimistic locking may be more appropriate for correctness. Redis locks are useful for distributed coordination where low latency and cross-instance coordination are needed."

---

# 82. Interview Question

### Can Redis provide distributed transactions with MySQL?

Answer:

> "Not by simply using Redis transactions. Redis and MySQL are separate resources. Coordinating atomic changes across both requires a distributed transaction strategy or, more commonly, patterns such as transactional outbox, idempotency and eventual consistency."

---

# 83. E-commerce Scenario

### Prevent overselling

Architecture:

```text
Request
  ↓
Redis atomic stock check
  ↓
Reserve
  ↓
Order service
  ↓
MySQL transaction
```

But the final architecture must define:

```text
Reservation expiry
Payment failure
Order failure
Redis failure
MySQL failure
Retry
Reconciliation
```

---

# 84. E-commerce Scenario

### Prevent duplicate payment request

Use:

```text
Idempotency-Key
```

Store:

```text
payment:idempotency:<key>
```

State:

```text
PROCESSING
COMPLETED
FAILED
```

with an appropriate retention period.

---

# 85. E-commerce Scenario

### Prevent duplicate order processing

Possible:

```text
order:process:<orderId>
```

Use:

```text
SET NX EX
```

But also make the processing operation:

```text
Idempotent
```

because distributed locks can fail or expire.

---

# 86. Microservices Scenario

Three instances:

```text
Order Service A
Order Service B
Order Service C
```

All receive:

```text
same event
```

Possible strategy:

```text
Idempotency key
+
database unique constraint
+
optional Redis coordination
```

The database constraint can provide the durable correctness boundary.

---

# 87. Redis vs Database for Idempotency

Redis:

```text
Fast
TTL
Distributed
```

Database:

```text
Durable
Transactional
Strong source of truth
```

For critical business operations, a database unique constraint can be an important final safeguard.

---

# 88. Rate Limiter Architecture

```text
Client
 ↓
API Gateway
 ↓
Redis
 ↓
Allowed?
 /      \
Yes      No
 |        |
API      429
```

Rate limiting is often best placed at:

```text
Gateway
```

or:

```text
edge/service boundary
```

depending on architecture.

---

# 89. Rate Limiter Dimensions

Can limit by:

```text
IP
User
API key
Tenant
Endpoint
Combination
```

Example:

```text
tenant:101:/orders
```

This is useful in multi-tenant systems.

---

# 90. Distributed Lock Architecture

```text
Instance A ─┐
Instance B ─┼── Redis lock
Instance C ─┘
```

Only the owner proceeds.

But:

```text
Redis lock
+
business idempotency
+
source-of-truth constraints
```

is often a stronger design.

---

# 91. Backoff Strategy

When lock acquisition fails:

```text
Don't retry continuously.
```

Use:

```text
Exponential backoff
+
jitter
```

Example:

```text
50 ms
100 ms
200 ms
400 ms
```

with a maximum retry limit.

---

# 92. Why Jitter?

If 1,000 instances all retry at exactly:

```text
100 ms
```

they can create another synchronized traffic spike.

Jitter randomizes retry timing.

---

# 93. Lock Acquisition Timeout

Don't wait forever.

Define:

```text
Maximum wait
Maximum retries
```

Then:

```text
Fail
Queue
Fallback
```

depending on business semantics.

---

# 94. Distributed Lock Observability

Monitor:

```text
Lock acquisition success rate
Lock wait time
Lock hold duration
Lock expiration
Failed releases
Contention
Retry count
```

High contention can indicate:

```text
Too coarse lock
Slow critical section
Incorrect architecture
```

---

# 95. Lock Granularity

Bad:

```text
global:lock
```

if it protects unrelated resources.

Better:

```text
order:101
order:102
order:103
```

when independent resources can be processed concurrently.

---

# 96. Rate Limiter Observability

Monitor:

```text
Requests allowed
Requests rejected
Limit utilization
Redis latency
Redis errors
Hot rate-limit keys
```

A rate limiter itself can become a bottleneck if badly designed.

---

# 97. Atomic Operations Checklist

```text
□ INCR
□ DECR
□ HINCRBY
□ ZINCRBY
□ SET NX
□ GET + SET race
□ MULTI
□ EXEC
□ DISCARD
□ WATCH
□ Lua scripts
□ Conditional updates
```

---

# 98. Distributed Lock Checklist

```text
□ SET NX EX
□ Unique owner token
□ Safe unlock
□ Lock TTL
□ Lease renewal
□ Lock contention
□ Retry/backoff
□ Fencing tokens
□ Idempotency
□ Process pauses
□ Network failures
□ Crash recovery
```

---

# 99. Rate Limiting Checklist

```text
□ Fixed window
□ Sliding window
□ Token bucket
□ Atomic counter
□ TTL
□ Lua
□ 429
□ Retry-After
□ User/IP/API key
□ Burst control
□ Distributed consistency
```

---

# 100. Final Interview Answer

If asked:

> "How would you use Redis for distributed coordination in a Spring Boot microservices application?"

Say:

> "I'd use Redis for low-latency coordination such as distributed locks, idempotency keys and rate limiting. For a lock I'd use an atomic `SET NX EX` with a unique owner token and safe token-verified release. For multi-step operations like conditional inventory updates or rate limiting, I'd use Lua scripts when atomicity is required. I wouldn't rely on the Redis lock alone for business correctness; I'd also use idempotency and durable database constraints where appropriate."

---

# 101. Final Mental Model

```text
Atomic command
     ↓
Simple concurrency problem

Lua / WATCH
     ↓
Multi-step Redis state transition

Distributed lock
     ↓
Exclusive coordination

Idempotency
     ↓
Safe repeated execution

Fencing
     ↓
Protect against stale owners

Rate limiter
     ↓
Control traffic

Database constraint
     ↓
Durable correctness
```

The strongest backend designs combine these mechanisms rather than expecting Redis to solve every distributed-systems problem.

---

# 102. What Comes Next

```text
File 06 → Redis Pub/Sub, Streams & Messaging
```

Next we will cover:

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
Event processing
Message ordering
At-least-once delivery
Idempotent consumers
Spring Boot messaging
Microservices event examples
```

Key takeaway:

> **Redis becomes especially powerful in backend systems when you combine atomic primitives with good distributed-systems design. Locks coordinate work, rate limiting protects services, idempotency prevents duplicate effects, and durable database constraints provide the final correctness boundary.**
