# System Design — File 02: Requirements & Back-of-the-Envelope Estimation

This file covers the first thing you should do in a system-design interview:

```text
Understand the problem
        ↓
Estimate the scale
        ↓
Then design
```

A lot of weak system-design answers fail because they jump straight into:

```text
Kafka
Redis
Microservices
Kubernetes
```

without first establishing what the system actually needs.

---

# 1. Why Requirements Come First

Imagine the interviewer says:

> "Design a food delivery application."

That statement is not enough.

You need to understand:

```text
Who uses it?
What can they do?
How many users?
How much traffic?
How much data?
How quickly must it respond?
What happens during failures?
```

The answers directly affect the architecture.

---

# 2. Two Types of Requirements

Start with:

```text
Functional Requirements
Non-Functional Requirements
```

---

# 3. Functional Requirements

Functional requirements describe what the system does.

For a food delivery application:

```text
Customer can:
- Search restaurants
- View menus
- Add items to cart
- Place order
- Pay
- Track order
- Cancel order
```

Restaurant:

```text
- Accept order
- Update order status
- Manage menu
```

Delivery partner:

```text
- Accept delivery
- Update location
- Mark order delivered
```

---

# 4. Non-Functional Requirements

These describe system qualities.

Examples:

```text
Availability
Scalability
Latency
Reliability
Security
Consistency
Durability
Maintainability
```

Example:

```text
Restaurant search should respond within 200 ms for most requests.
```

---

# 5. Functional vs Non-Functional Example

### Functional

```text
User can place an order.
```

### Non-functional

```text
The order API should remain available during peak traffic.
```

Both are important.

---

# 6. What Should You Clarify?

You don't need to ask 30 questions.

Ask questions that change the design.

Useful categories:

```text
Users
Traffic
Features
Data
Latency
Availability
Consistency
Geography
Security
Real-time requirements
```

---

# 7. Question 1 — Who Are the Users?

Example:

```text
Customers
Restaurants
Delivery partners
Admins
```

Different users may have different APIs and workloads.

---

# 8. Question 2 — What Is the Core Use Case?

Don't try to design the entire company.

For example:

> "Let's focus on restaurant search and order placement."

This keeps the interview manageable.

---

# 9. Question 3 — How Many Users?

Ask:

```text
How many registered users?
How many daily active users?
How many concurrent users?
```

These are different measurements.

---

# 10. Registered vs Active Users

Suppose:

```text
50 million registered users
```

but only:

```text
5 million DAU
```

The system doesn't receive traffic from all 50 million users every day.

So:

```text
DAU
```

is often more useful for traffic estimation.

---

# 11. DAU

DAU means:

```text
Daily Active Users
```

Example:

```text
5 million DAU
```

means approximately 5 million users are active on a typical day according to the system's defined activity metric.

---

# 12. MAU

MAU means:

```text
Monthly Active Users
```

Useful for understanding overall user base and product engagement.

For request-volume estimation, DAU is often more directly useful.

---

# 13. Concurrent Users

Concurrent users are users interacting with the system at roughly the same time.

Example:

```text
5 million DAU
```

doesn't mean:

```text
5 million simultaneous users
```

Peak concurrency could be much smaller.

---

# 14. Question 4 — How Many Requests?

Ask:

```text
Average requests per user/day?
```

For example:

```text
5 million DAU
×
20 requests/day
=
100 million requests/day
```

Now estimate RPS.

---

# 15. RPS

RPS:

```text
Requests Per Second
```

Formula:

```text
Daily Requests / 86,400
```

Example:

```text
100,000,000 / 86,400
≈ 1,157 RPS
```

So average traffic is approximately:

```text
1.2K RPS
```

---

# 16. Average vs Peak

Never stop at average RPS.

Suppose:

```text
Average = 1,200 RPS
```

If peak traffic is 5× average:

```text
Peak = 6,000 RPS
```

You can state:

> "I'll design for roughly 6K peak RPS based on a 5× peak factor."

The exact factor is an assumption.

---

# 17. Peak Traffic Factor

You can assume something like:

```text
Peak = 3× to 10× average
```

depending on the product.

For a real-time event or flash sale:

```text
10×+
```

may be reasonable.

Always state the assumption.

---

# 18. Why State Assumptions?

Don't say:

> "The system handles 10,000 RPS."

Say:

> "Assuming 5 million DAU and 20 requests per user per day, we get around 1,200 average RPS. If peak traffic is 5×, we should plan for around 6,000 peak RPS."

This demonstrates engineering reasoning.

---

# 19. Read/Write Ratio

Ask:

```text
How much is read traffic?
How much is write traffic?
```

Example:

```text
90% reads
10% writes
```

At:

```text
6,000 peak RPS
```

approximately:

```text
5,400 reads
600 writes
```

---

# 20. Why Read/Write Ratio Matters

Read-heavy:

```text
Caching
Read replicas
Search indexes
```

may be useful.

Write-heavy:

```text
Database write capacity
Partitioning
Queues
Batching
```

may become more important.

---

# 21. Storage Estimation

Estimate:

```text
New records/day
Record size
Retention period
Replication
Indexes
Backups
```

Example:

```text
1 million orders/day
```

Average record:

```text
2 KB
```

Raw daily storage:

```text
1,000,000 × 2 KB
=
2 GB/day
```

---

# 22. Annual Storage

```text
2 GB/day × 365
≈ 730 GB/year
```

But that's raw logical data.

Real storage may be much higher because of:

```text
Indexes
Replication
Metadata
Backups
Temporary data
Historical records
```

---

# 23. Storage Growth

Don't only calculate today's storage.

Think:

```text
Year 1 → 730 GB
Year 2 → 1.46 TB
Year 3 → 2.19 TB
```

assuming the same growth rate.

Then ask:

```text
Can the chosen database handle this?
```

---

# 24. Image/File Storage

Suppose:

```text
10 million product images
```

Average:

```text
500 KB
```

Raw storage:

```text
10,000,000 × 500 KB
≈ 5 TB
```

This is usually a better fit for:

```text
Object storage
```

than a relational database.

---

# 25. Bandwidth Estimation

Suppose:

```text
1,000 requests/sec
```

and average response:

```text
20 KB
```

Then:

```text
1,000 × 20 KB
=
20,000 KB/sec
≈ 20 MB/sec
```

This gives a rough bandwidth requirement.

---

# 26. Request vs Response Size

Be careful.

A request may be:

```text
2 KB
```

while a response might be:

```text
100 KB
```

For API bandwidth, estimate both if relevant.

---

# 27. Latency Requirements

Ask:

```text
What's an acceptable response time?
```

Example:

```text
Search → <200 ms
Product details → <150 ms
Checkout → <500 ms
```

These are illustrative requirements, not universal standards.

---

# 28. p50, p95, p99

Latency isn't just an average.

Common percentiles:

```text
p50
p95
p99
```

Meaning:

### p50

50% of requests are faster than this value.

### p95

95% are faster.

### p99

99% are faster.

---

# 29. Why p95/p99 Matter

Suppose:

```text
Average = 100 ms
```

but:

```text
p99 = 5 seconds
```

Most users may be fine, but some users experience terrible latency.

For production systems:

```text
Tail latency matters.
```

---

# 30. Availability Requirement

Ask:

```text
How available should the system be?
```

Examples:

```text
99%
99.9%
99.99%
99.999%
```

The higher the availability target:

```text
More redundancy
More operational complexity
Higher cost
```

usually follow.

---

# 31. Availability Numbers

Approximate yearly downtime:

```text
99%
   ≈ 3.65 days

99.9%
   ≈ 8.76 hours

99.99%
   ≈ 52.6 minutes

99.999%
   ≈ 5.26 minutes
```

These are rough 365-day calculations.

---

# 32. Availability Trade-off

Going from:

```text
99.9%
```

to:

```text
99.99%
```

is not just:

```text
0.09% better
```

It significantly reduces allowed downtime.

Higher availability usually requires:

```text
Redundancy
Failover
Monitoring
Automation
Multi-zone deployment
Careful deployments
```

---

# 33. Consistency Requirement

Ask:

```text
Does every read need the latest data immediately?
```

Examples where stronger consistency may matter:

```text
Bank balance
Payment status
Inventory reservation
```

Examples where eventual consistency may be acceptable:

```text
Product view count
Analytics dashboard
Recommendation data
Some search indexes
```

---

# 34. Real-Time Requirement

Ask:

```text
Does the user need updates immediately?
```

Examples:

```text
Driver location
Chat
Live sports score
Order tracking
```

Possible technologies:

```text
WebSocket
Server-Sent Events
Polling
Message streaming
```

---

# 35. Geographic Scope

Ask:

```text
One city?
One country?
Global?
```

This changes:

```text
Latency
Data residency
CDN
Multi-region architecture
Disaster recovery
```

---

# 36. Multi-Region

A global application might use:

```text
Users
 / \
Region A   Region B
   \         /
    Global services
```

But multi-region introduces:

```text
Data replication
Consistency challenges
Failover complexity
Higher cost
```

Don't add it unless requirements justify it.

---

# 37. Back-of-the-Envelope Estimation

The goal is:

```text
Fast
Reasonable
Transparent
```

You are not expected to produce production-grade capacity planning in an interview.

---

# 38. Useful Numbers to Memorize

These are rough mental-math values:

```text
1 minute      ≈ 60 seconds
1 hour        ≈ 3,600 seconds
1 day         ≈ 86,400 seconds
1 year        ≈ 31.5 million seconds
1 KB          ≈ 1,000 bytes
1 MB          ≈ 1,000 KB
1 GB          ≈ 1,000 MB
```

For interview estimation, decimal approximations are usually enough.

---

# 39. Quick RPS Formula

```text
RPS
=
DAU × requests/user/day
÷ 86,400
```

Example:

```text
5M × 20
---------
 86,400
```

≈

```text
1,157 RPS
```

---

# 40. Peak RPS Formula

```text
Peak RPS
=
Average RPS × Peak Factor
```

Example:

```text
1,157 × 5
≈ 5,785 RPS
```

Round it:

```text
≈ 6K peak RPS
```

---

# 41. Write RPS

```text
Write RPS
=
Total RPS × Write %
```

Example:

```text
6,000 × 10%
=
600 writes/sec
```

---

# 42. Read RPS

```text
Read RPS
=
Total RPS × Read %
```

Example:

```text
6,000 × 90%
=
5,400 reads/sec
```

---

# 43. Daily Data Growth

Formula:

```text
Records/day × average record size
```

Example:

```text
1M × 2 KB
=
2 GB/day
```

---

# 44. Annual Data Growth

```text
Daily growth × 365
```

Example:

```text
2 GB × 365
≈ 730 GB/year
```

Then add a reasonable allowance for:

```text
Indexes
Replication
Backups
```

---

# 45. Bandwidth Formula

Approximate:

```text
RPS × average payload size
```

Example:

```text
5,000 RPS × 20 KB
=
100 MB/sec
```

Again, this is a rough estimate.

---

# 46. Concurrent Requests

A useful approximation is Little's Law:

```text
Concurrency ≈ Throughput × Latency
```

Example:

```text
5,000 RPS
×
0.2 seconds
=
1,000 concurrent requests
```

This is useful for reasoning about:

```text
Thread pools
Connection pools
Memory
```

---

# 47. Database Connection Estimation

Suppose:

```text
20 application instances
```

and each has:

```text
20 DB connections
```

Then:

```text
20 × 20
=
400 DB connections
```

This can become a database bottleneck even if application CPU looks fine.

---

# 48. Cache Size Estimation

Suppose you cache:

```text
5 million products
```

Average cached object:

```text
2 KB
```

Raw cache data:

```text
5M × 2 KB
≈ 10 GB
```

Real memory requirements will be higher due to:

```text
Keys
Metadata
Redis overhead
Replication
Eviction headroom
```

---

# 49. Queue Throughput

Suppose:

```text
10,000 events/sec
```

and each consumer handles:

```text
500 events/sec
```

Then roughly:

```text
10,000 / 500
=
20 consumers
```

You also need to consider:

```text
Processing variance
Retries
Failures
Partitioning
Headroom
```

---

# 50. Capacity Planning

Don't design at:

```text
100% capacity
```

If peak load is:

```text
10,000 RPS
```

you may want capacity above that:

```text
15,000 RPS
```

or more depending on the service's SLOs and scaling model.

This provides headroom for:

```text
Traffic spikes
Failures
Deployments
Unexpected workloads
```

---

# 51. Example — URL Shortener

Suppose:

```text
100 million users
```

But only:

```text
10 million URLs created/day
```

and:

```text
100 million redirects/day
```

Then:

```text
Writes:
10M / 86,400
≈ 116 writes/sec
```

Reads:

```text
100M / 86,400
≈ 1,157 reads/sec
```

If peak factor is 5:

```text
≈ 580 writes/sec peak
≈ 5,785 reads/sec peak
```

This is read-heavy.

Possible architecture:

```text
Client
  ↓
Load Balancer
  ↓
App
  ↓
Redis
  ↓
Database
```

---

# 52. Example — E-commerce

Assume:

```text
10M DAU
20 requests/user/day
```

Total:

```text
200M requests/day
```

Average:

```text
200M / 86,400
≈ 2,315 RPS
```

Peak at 5×:

```text
≈ 11.6K RPS
```

If:

```text
90% reads
10% writes
```

then:

```text
Read ≈ 10.4K RPS
Write ≈ 1.16K RPS
```

Now the architecture should focus heavily on:

```text
Read scaling
Caching
Database capacity
```

---

# 53. Example — Chat Application

Suppose:

```text
1M DAU
```

and each user sends:

```text
20 messages/day
```

Messages/day:

```text
20M
```

Average message writes:

```text
20M / 86,400
≈ 231 messages/sec
```

But chat also has:

```text
Concurrent connections
Real-time delivery
Presence
Ordering
Offline delivery
```

So message RPS alone isn't enough.

---

# 54. Estimation Doesn't Give the Architecture

This is important.

Suppose you calculate:

```text
10K RPS
```

That doesn't automatically mean:

```text
Kafka
Kubernetes
20 microservices
Sharding
```

Instead ask:

```text
What is the bottleneck?
```

Then choose the component that solves it.

---

# 55. Turning Estimates Into Architecture

Example:

```text
10K read RPS
```

If MySQL handles the workload comfortably:

```text
Don't shard yet.
```

If repeated reads overload MySQL:

```text
Add Redis.
```

If read traffic still exceeds one database:

```text
Consider read replicas.
```

If writes become the bottleneck:

```text
Investigate indexing,
partitioning,
sharding,
batching,
or architecture changes.
```

---

# 56. Interview Assumptions

Say assumptions clearly.

Example:

> "I'll assume 10 million DAU, 20 requests per user per day, a 90/10 read-write ratio, and a 5× peak factor."

Then calculate.

This is much better than silently inventing numbers.

---

# 57. Don't Get Stuck on Exact Numbers

If your estimate is:

```text
5,800 RPS
```

and someone else says:

```text
6,500 RPS
```

that doesn't automatically make one answer wrong.

The important thing is:

```text
Reasonable assumptions
Correct formulas
Clear reasoning
```

---

# 58. Requirement Prioritization

When time is limited, prioritize:

```text
1. Core functional requirements
2. Scale
3. Availability
4. Latency
5. Consistency
6. Security
7. Operational requirements
```

Then go deeper where the interviewer asks.

---

# 59. Questions That Change Architecture

### "Is the system read-heavy?"

Could lead to:

```text
Redis
Read replicas
Search
```

### "Does it need real-time updates?"

Could lead to:

```text
WebSocket
SSE
Streaming
```

### "Can data be eventually consistent?"

Could enable:

```text
Async processing
Replication
Event-driven architecture
```

### "Must it be globally available?"

Could lead to:

```text
Multi-region
CDN
Geo-routing
```

---

# 60. Requirement Checklist

Before designing, write:

```text
Users:
DAU:
Peak concurrency:
Requests/day:
Average RPS:
Peak RPS:
Read/write ratio:
Data created/day:
Data retention:
Latency target:
Availability target:
Consistency requirement:
Geographic scope:
Real-time requirement:
```

You don't always need every field.

Use the ones relevant to the problem.

---

# 61. Interview Template

When interviewer gives you a problem, say:

> "Before I design it, I'd like to clarify a few requirements."

Then:

```text
1. What are the core use cases?
2. What's the expected user scale?
3. What's the expected traffic?
4. Is the workload read or write heavy?
5. What are the latency and availability requirements?
6. How strong does consistency need to be?
7. Is the system global or regional?
8. Are there real-time requirements?
```

Then summarize your assumptions.

---

# 62. Estimation Template

Use:

```text
Users
 ↓
DAU
 ↓
Requests/user/day
 ↓
Requests/day
 ↓
Average RPS
 ↓
Peak factor
 ↓
Peak RPS
```

For data:

```text
Records/day
 ↓
Record size
 ↓
Daily storage
 ↓
Annual storage
 ↓
Replication/index overhead
```

For bandwidth:

```text
RPS
 ↓
Payload size
 ↓
MB/sec
```

---

# 63. Common Mistakes

### Mistake 1

Starting with technology.

```text
"We'll use Kafka."
```

Ask:

```text
Why?
```

---

### Mistake 2

No assumptions.

```text
"System needs 50 servers."
```

Why?

---

### Mistake 3

Only calculating average traffic.

Always think:

```text
Peak
```

---

### Mistake 4

Ignoring writes.

Read-heavy systems still need to handle writes.

---

### Mistake 5

Ignoring data growth.

A database that works today may not work at 10× scale.

---

### Mistake 6

Ignoring latency.

A system can handle huge throughput and still feel slow.

---

### Mistake 7

Ignoring failure scenarios.

Ask:

```text
What if DB fails?
What if Redis fails?
What if a service times out?
```

---

# 64. Final Interview Example

Interviewer:

> "Design a product catalog."

You:

> "I'll first clarify the requirements. I'll assume 10 million DAU, around 20 requests per user per day, 90% reads and 10% writes, and a 5× peak factor. That gives roughly 2.3K average RPS and around 11.5K peak RPS. Since it's read-heavy, I'd initially consider a Spring Boot service behind a load balancer, Redis for frequently accessed product data, and MySQL for durable product information. I'd then look at read replicas if database reads become the bottleneck."

That's already a strong beginning.

---

# 65. What We Will Do Next

Now that requirements and estimation are clear, the next files will build the architecture piece by piece:

```text
03 → Scalability & Stateless Architecture
04 → Load Balancing
05 → Caching
06 → Database Design
07 → SQL vs NoSQL
08 → Replication & Sharding
09 → Messaging & Kafka
...
```

---

# 66. Final Mental Model

Remember:

```text
Don't design first.

Ask:
"What does the system need?"

Then:
"How big is it?"

Then:
"What constraints matter?"

Then:
"What architecture solves those constraints?"
```

**Key takeaway:**

> **Good system design starts with assumptions, not technologies. Estimate the load, identify the important constraints, and let those constraints drive the architecture.**
