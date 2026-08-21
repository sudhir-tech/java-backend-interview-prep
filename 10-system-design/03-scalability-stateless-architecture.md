# System Design — File 03: Scalability & Stateless Architecture

This file focuses on one of the most common system-design questions:

> "How would you scale this system?"

The basic progression is:

```text
Single instance
      ↓
Vertical scaling
      ↓
Horizontal scaling
      ↓
Load balancer
      ↓
Stateless application
      ↓
Cache / database scaling
      ↓
Async processing
```

The important skill is knowing **why** each step is needed.

---

# 1. What Is Scalability?

Scalability is the ability of a system to handle increasing workload while maintaining acceptable performance.

Workload can increase through:

```text
More users
More requests
Larger data
More concurrent connections
More background jobs
```

---

# 2. Two Main Types

```text
Vertical scaling
Horizontal scaling
```

---

# 3. Vertical Scaling

Increase the resources of one machine.

Example:

```text
Before:

4 CPU
8 GB RAM

        ↓

After:

16 CPU
64 GB RAM
```

This is also called:

```text
Scale up
```

---

# 4. Advantages of Vertical Scaling

```text
Simple
Less architectural change
Easy to operate
No distributed coordination
```

For a small application, this can be completely reasonable.

---

# 5. Limitations of Vertical Scaling

Eventually:

```text
Hardware has limits
Costs increase
One machine can still fail
```

So:

```text
Vertical scaling
≠
Infinite scaling
```

---

# 6. Horizontal Scaling

Instead of making one server bigger:

```text
App 1
App 2
App 3
App 4
```

This is:

```text
Scale out
```

Traffic is distributed across instances.

---

# 7. Horizontal Scaling Example

Before:

```text
Client
  ↓
App
  ↓
MySQL
```

After:

```text
             Client
                ↓
         Load Balancer
          /     |     \
         ↓      ↓      ↓
       App1   App2   App3
          \     |     /
               ↓
             MySQL
```

The application tier can now handle more requests.

---

# 8. Why Horizontal Scaling Works Well for APIs

HTTP APIs are often naturally suitable for horizontal scaling if application instances are:

```text
Stateless
```

Any instance can handle a request.

---

# 9. Stateless Architecture

A stateless application does not depend on local instance memory for important persistent client state.

Example:

```text
Request 1 → App1
Request 2 → App3
Request 3 → App2
```

All instances can process the requests.

---

# 10. Stateful Architecture

Suppose:

```text
User session
     ↓
App1 memory
```

Then:

```text
Request 1 → App1
Request 2 → App2
```

App2 may not know the user's session state.

This makes horizontal scaling more complicated.

---

# 11. Externalize State

Instead of:

```text
App memory
```

use:

```text
Redis
Database
```

Example:

```text
          +------ App1
          |
Client → Load Balancer
          |
          +------ App2
          |
          +------ App3
                  |
                  ↓
                Redis
```

Now application instances can share state.

---

# 12. Stateless Doesn't Mean "No State Exists"

This is a common interview trap.

A stateless service can still use:

```text
Database
Redis
Object storage
Message queue
```

It simply doesn't depend on **local instance memory or local filesystem** for state that must survive or be shared.

---

# 13. Authentication and Stateless APIs

JWT authentication is often used with stateless APIs.

Request:

```http
Authorization: Bearer <token>
```

The application can validate the token without storing a server-side session for every request.

But JWT doesn't automatically solve every authentication problem.

You still need to consider:

```text
Expiration
Revocation
Refresh tokens
Key rotation
Authorization
```

---

# 14. Session-Based Authentication

Traditional approach:

```text
Client
  ↓
Session ID
  ↓
Server-side session
```

If session data is stored only in App1:

```text
App1
 ↓
Session
```

then another instance may not have it.

---

# 15. Session Sharing

Possible solution:

```text
App1
App2
App3
 |
 ↓
Redis
 |
 ↓
Sessions
```

Now all instances can access shared session data.

---

# 16. Sticky Sessions

Another approach is:

```text
User A → App1
User A → App1
User A → App1
```

The load balancer keeps sending the user to the same instance.

This is called:

```text
Sticky sessions
```

---

# 17. Why Sticky Sessions Are Often Less Attractive

They can make scaling and failure handling harder.

Example:

```text
User → App1
         ↓
       crash
         ↓
User moved to App2
         ↓
Session missing
```

Externalizing session state is often cleaner.

---

# 18. Stateless API Benefits

```text
Easy horizontal scaling
Better load distribution
Simpler failover
Easier deployments
Better elasticity
```

---

# 19. Load Balancer

With multiple instances:

```text
Client
  ↓
Load Balancer
 /    |    \
App1 App2 App3
```

The load balancer distributes traffic.

---

# 20. Load Balancing Algorithms

Common strategies include:

```text
Round robin
Least connections
Weighted routing
Hash-based routing
```

The appropriate strategy depends on the workload.

---

# 21. Round Robin

Requests are distributed sequentially:

```text
Request 1 → App1
Request 2 → App2
Request 3 → App3
Request 4 → App1
```

Simple and common.

---

# 22. Least Connections

Send new requests toward the instance with fewer active connections.

Conceptually:

```text
App1 → 50 connections
App2 → 10 connections
App3 → 30 connections
```

Next request:

```text
→ App2
```

Useful when requests have significantly different durations.

---

# 23. Weighted Load Balancing

Suppose:

```text
App1 → weight 1
App2 → weight 2
App3 → weight 1
```

App2 receives roughly twice as much traffic as each of the others.

Useful when instances have different capacities.

---

# 24. Health Checks

The load balancer should avoid unhealthy instances.

```text
App1 → healthy
App2 → unhealthy
App3 → healthy
```

Traffic:

```text
App1
App3
```

not App2.

---

# 25. Health Check Types

Possible checks:

```text
TCP
HTTP
Application health endpoint
```

For Spring Boot, Actuator can expose health information.

Example:

```text
/actuator/health
```

The exact endpoint configuration depends on the application.

---

# 26. Scaling Application Instances

Suppose:

```text
1 instance
→ 500 RPS
```

and we need:

```text
2,000 RPS
```

A rough first estimate:

```text
2,000 / 500
=
4 instances
```

In practice, add headroom.

Maybe:

```text
5 instances
```

depending on:

```text
CPU
Memory
Latency
Traffic variation
Failure tolerance
```

---

# 27. Don't Assume Perfect Linear Scaling

If:

```text
1 instance → 500 RPS
```

then:

```text
4 instances ≠ guaranteed 2,000 RPS
```

Reasons include:

```text
Database bottleneck
Network
Connection pools
Lock contention
Cache
External services
Load-balancer overhead
```

---

# 28. The Database Often Becomes the Bottleneck

You may scale:

```text
App1
App2
App3
App4
App5
```

but all connect to:

```text
One MySQL
```

Then:

```text
Application scales
        ↓
Database overloaded
        ↓
System bottleneck
```

This is a classic system-design problem.

---

# 29. Scaling the Database

Potential approaches:

```text
Better indexes
Query optimization
Caching
Read replicas
Partitioning
Sharding
Database scaling
```

Use the simplest solution that solves the actual bottleneck.

---

# 30. Connection Pool Scaling

Suppose:

```text
10 app instances
```

and each has:

```text
20 DB connections
```

Potential total:

```text
200 connections
```

If you scale to:

```text
50 instances
```

then:

```text
50 × 20
=
1,000 DB connections
```

The database may become overwhelmed.

---

# 31. Important Lesson

Horizontal scaling the application can **increase pressure on dependencies**.

Think about:

```text
App scaling
 ↓
DB connections
 ↓
Redis connections
 ↓
External API calls
 ↓
Queue throughput
```

Always check the downstream systems.

---

# 32. Autoscaling

Autoscaling changes the number of instances based on demand.

Example:

```text
Low traffic
→ 2 instances

High traffic
→ 8 instances

Traffic falls
→ 2 instances
```

---

# 33. Autoscaling Metrics

Common signals:

```text
CPU
Memory
Requests/sec
Queue depth
Latency
Custom business metrics
```

CPU alone isn't always the best metric.

---

# 34. Queue-Based Autoscaling

Suppose:

```text
Queue depth = 100
```

Normal workers:

```text
2
```

Traffic spikes:

```text
Queue depth = 100,000
```

Scale workers:

```text
2 → 20
```

The queue becomes a useful signal.

---

# 35. Scaling Based on Latency

Suppose:

```text
p95 latency
```

rises significantly as traffic increases.

You can use latency as a signal for scaling depending on the platform and architecture.

The important idea:

```text
Scale based on the actual bottleneck.
```

---

# 36. Stateless Services and Autoscaling

Stateless services are especially suitable for autoscaling:

```text
Request
  ↓
Any healthy instance
```

Instances can be created or destroyed without moving important local state.

---

# 37. Containerized Scaling

Docker gives you:

```text
Container
```

but production orchestration handles:

```text
Scheduling
Scaling
Health
Restart
Rolling deployment
```

Examples:

```text
Kubernetes
ECS
Cloud Run
Container Apps
```

The exact platform depends on the environment.

---

# 38. Scaling Background Workers

Not everything is an HTTP API.

Example:

```text
Order events
    ↓
Queue
    ↓
Worker
```

If queue depth increases:

```text
Worker1
Worker2
Worker3
...
Worker20
```

can process messages concurrently.

---

# 39. Scaling Consumers

If each worker processes:

```text
500 messages/sec
```

and traffic is:

```text
5,000 messages/sec
```

roughly:

```text
5,000 / 500
=
10 workers
```

Then add headroom and account for:

```text
Retries
Failures
Uneven workload
Ordering
```

---

# 40. Database Read Scaling

If reads dominate:

```text
Primary
  |
  +--- Replica1
  +--- Replica2
  +--- Replica3
```

Applications can route suitable reads to replicas.

---

# 41. Read Replica Trade-off

Replicas can have:

```text
Replication lag
```

So:

```text
Write → Primary
Immediate read → Replica
```

may return stale data.

For data requiring fresh reads:

```text
Read from primary
```

may be necessary.

---

# 42. Caching for Scale

Suppose:

```text
10,000 product requests/sec
```

but most users request the same popular products.

Without cache:

```text
10,000 → DB
```

With cache:

```text
10,000 → Redis
```

Only cache misses reach the database.

---

# 43. Cache Reduces More Than Latency

It can reduce:

```text
Database CPU
Database connections
Database I/O
Application processing
Network traffic
```

---

# 44. Cache Isn't Free

Redis also has limits:

```text
Memory
CPU
Network
Connections
```

And cached data needs:

```text
Expiration
Eviction
Invalidation
```

---

# 45. CDN Scaling

For static content:

```text
Images
JavaScript
CSS
Videos
```

a CDN can move traffic away from the origin.

```text
Users
 ↓
CDN
 ↓ cache miss
Origin
```

This improves:

```text
Latency
Origin load
Bandwidth efficiency
```

---

# 46. Database Write Scaling

Writes are harder to scale than reads.

Possible techniques:

```text
Batching
Asynchronous processing
Partitioning
Sharding
Write optimization
Queueing
```

But each introduces trade-offs.

---

# 47. Queue as a Traffic Buffer

Suppose:

```text
Incoming:
10,000 jobs/sec

Workers:
5,000 jobs/sec
```

Without a queue:

```text
Workers overloaded
```

With a queue:

```text
10K incoming
 ↓
Queue
 ↓
5K processing
```

The queue absorbs temporary bursts.

But if the difference persists:

```text
Queue keeps growing
```

So you need more consumers or reduced workload.

---

# 48. Backpressure

A system should not allow producers to overwhelm consumers indefinitely.

Possible controls:

```text
Rate limiting
Queue limits
Load shedding
Backpressure
Autoscaling
```

---

# 49. Load Shedding

When a system is overloaded, it may intentionally reject lower-priority work.

Example:

```text
Critical:
Checkout → process

Non-critical:
Recommendation refresh → delay/reject
```

This protects the core system.

---

# 50. Graceful Degradation

When dependencies fail, don't always take down the entire application.

Example:

```text
Redis unavailable
```

Product details might still come from:

```text
MySQL
```

but perhaps with:

```text
Higher latency
```

---

# 51. Another Example

Recommendation service fails:

```text
Recommendation unavailable
```

The product page can still work:

```text
Product details → available
Recommendations → temporarily unavailable
```

This is graceful degradation.

---

# 52. Fail Fast

If a dependency is unavailable:

```text
Don't wait 30 seconds
```

Use:

```text
Timeout
Circuit breaker
Fallback
```

This protects application resources.

---

# 53. Cascading Failure

Example:

```text
Payment Service slows
        ↓
Order Service waits
        ↓
Threads blocked
        ↓
Requests queue up
        ↓
CPU/memory increases
        ↓
Order Service becomes unhealthy
        ↓
Entire checkout fails
```

This is a cascading failure.

---

# 54. Preventing Cascading Failure

Use:

```text
Timeouts
Circuit breakers
Bulkheads
Rate limiting
Async processing
Load shedding
```

---

# 55. Bulkhead Pattern

Separate resources for different workloads.

Example:

```text
Checkout thread pool
Search thread pool
Notification thread pool
```

If search becomes slow:

```text
Search pool exhausted
```

Checkout can still have resources available.

---

# 56. Scaling by Service

In microservices:

```text
Product Service → 10 instances
Order Service   → 8 instances
User Service    → 3 instances
```

Each service can scale according to its workload.

This is one benefit of service decomposition.

---

# 57. But Microservices Add Complexity

More services means:

```text
More deployments
More networks
More logs
More metrics
More failures
More configurations
More operational work
```

Don't use microservices only because scaling is possible.

---

# 58. Vertical + Horizontal Scaling

You can use both.

Example:

```text
Each app instance:
8 CPU
16 GB RAM

And:

10 app instances
```

This is common.

---

# 59. Scaling Strategy

A useful progression:

```text
Start simple
   ↓
Optimize code
   ↓
Optimize queries
   ↓
Vertical scale
   ↓
Cache
   ↓
Horizontal scale
   ↓
Read replicas
   ↓
Async processing
   ↓
Partition/shard if required
```

This is not a strict sequence.

The correct step depends on the bottleneck.

---

# 60. Capacity Headroom

Don't operate permanently at:

```text
95–100% CPU
```

A traffic spike can immediately cause:

```text
Latency
Errors
Queue growth
```

Maintain reasonable headroom.

---

# 61. Overprovisioning

Having too much capacity also has a cost.

Example:

```text
Need:
5 instances

Run:
50 instances
```

This wastes:

```text
Money
Resources
Operational capacity
```

Autoscaling can help balance cost and performance.

---

# 62. Cost-Aware Scaling

Think:

```text
Performance
+
Reliability
+
Cost
```

System design isn't just:

```text
Maximum performance
```

---

# 63. Example — E-commerce Scaling

Start:

```text
Client
 ↓
Spring Boot
 ↓
MySQL
```

Traffic grows:

```text
Client
 ↓
Load Balancer
 ↓
App1 App2 App3
 ↓
MySQL
```

Reads become heavy:

```text
Apps
 ↓
Redis
 ↓
MySQL
```

Database reads grow:

```text
                 → Replica1
Apps → DB layer → Replica2
                 → Replica3
```

Async work grows:

```text
Order Service
      ↓
Kafka
      ↓
Notification workers
```

---

# 64. Scaling Doesn't Fix Bad Code

Suppose one query takes:

```text
5 seconds
```

Adding:

```text
10 application instances
```

doesn't necessarily solve the database problem.

First investigate:

```text
Index
Query plan
N+1 query
Connection pool
Database load
```

---

# 65. N+1 Example

Suppose:

```text
1 query → fetch 100 orders
```

Then application performs:

```text
100 additional queries
```

Total:

```text
101 queries
```

Scaling the application may hide the problem temporarily.

Fix the query behavior first.

---

# 66. Connection Pool as a Bottleneck

Suppose:

```text
100 requests/sec
```

but the app has only:

```text
10 DB connections
```

Long-running queries can cause:

```text
Connection wait
Latency
Timeouts
```

Increasing the pool isn't always the answer.

You need to check:

```text
Database capacity
Query duration
Connection usage
```

---

# 67. Thread Pool Bottleneck

If:

```text
200 requests/sec
```

but request processing is slow and threads are exhausted:

```text
Requests wait
```

You may need:

```text
Faster operations
Async processing
Better connection management
Appropriate thread-pool tuning
```

not simply more servers.

---

# 68. Scaling Checklist

When asked:

> "How do you scale this?"

Think:

```text
1. What is growing?
2. What is the bottleneck?
3. Can we optimize it?
4. Can we cache?
5. Can we scale horizontally?
6. Can we scale the database?
7. Can we move work async?
8. Can we isolate workloads?
9. What happens during failure?
10. What does it cost?
```

---

# 69. Interview Question

### How do you scale a Spring Boot application?

Answer:

> "I'd first make the application stateless so multiple instances can run behind a load balancer. Then I'd monitor the bottlenecks and scale horizontally based on traffic. I'd use caching for read-heavy workloads and scale the database separately using query optimization, replicas or partitioning when required."

---

# 70. Interview Question

### Why is stateless architecture useful?

Answer:

> "Because any healthy application instance can handle a request. That makes horizontal scaling, load balancing and failover much simpler."

---

# 71. Interview Question

### Why shouldn't you rely on sticky sessions?

Answer:

> "Sticky sessions can create uneven traffic distribution and make failover harder because session state is tied to a particular instance. Externalizing shared session state makes the application easier to scale."

---

# 72. Interview Question

### Does adding more servers always improve performance?

Answer:

> "No. If the bottleneck is the database, connection pool, external API or another shared dependency, adding application instances can actually increase pressure on that bottleneck."

---

# 73. Interview Question

### How do you scale a read-heavy application?

Answer:

> "I'd first consider caching frequently accessed data, then database read replicas if the database is still a bottleneck. I'd also optimize queries and indexes before introducing more complex techniques."

---

# 74. Interview Question

### How do you scale a write-heavy system?

Answer:

> "I'd optimize the write path first, then consider asynchronous processing, batching, partitioning or sharding depending on the workload. I'd also look at whether some writes can be delayed or processed asynchronously."

---

# 75. Interview Question

### What happens when traffic suddenly doubles?

Answer:

> "I'd want the application tier to scale horizontally, ideally through autoscaling. I'd also check whether the database, cache, connection pools and downstream services have enough capacity because scaling only the application tier can move the bottleneck downstream."

---

# 76. Interview Question

### What is graceful degradation?

Answer:

> "It's allowing core functionality to continue when a non-critical dependency fails. For example, if recommendations are unavailable, the product page can still return the product details."

---

# 77. Interview Question

### What is cascading failure?

Answer:

> "It's when a failure or slowdown in one component causes dependent components to become overloaded and fail as well. Timeouts, circuit breakers, bulkheads and asynchronous processing can help prevent it."

---

# 78. Interview Question

### What is a bottleneck?

Answer:

> "A bottleneck is the component that limits the system's overall capacity, such as a database, network, CPU, connection pool or external service."

---

# 79. Practical Scenario

### Scenario:

```text
Traffic:
10K RPS

Application:
10 instances

CPU:
30%

Database:
95% CPU
```

What would you do?

Don't immediately add more application instances.

The likely bottleneck is:

```text
Database
```

Investigate:

```text
Queries
Indexes
Connection pool
Caching
Read replicas
Database capacity
```

---

# 80. Practical Scenario

### Scenario:

```text
API CPU = 90%
DB CPU = 30%
```

Likely bottleneck:

```text
Application tier
```

Possible action:

```text
Horizontal scaling
```

Also investigate:

```text
CPU-heavy code
Serialization
Algorithms
Garbage collection
```

---

# 81. Practical Scenario

### Scenario:

```text
API CPU = 30%
DB CPU = 40%

External Payment API:
p99 = 8 seconds
```

The bottleneck may be:

```text
External dependency
```

Use:

```text
Timeout
Circuit breaker
Async processing where appropriate
```

Don't blindly add app servers.

---

# 82. Practical Scenario

### Scenario:

```text
Product API:
10K RPS

90% requests are for the same 1,000 products.
```

A strong candidate is:

```text
Redis cache
```

because the workload is highly cacheable.

---

# 83. Practical Scenario

### Scenario:

```text
Order processing:
5K requests/sec

Database writes:
too slow
```

Possible investigation:

```text
Can some work be asynchronous?
Can writes be batched?
Are indexes excessive?
Can data be partitioned?
Is the schema optimized?
```

---

# 84. Practical Scenario

### Scenario:

```text
Notification system
```

User doesn't need the email/SMS to finish before receiving:

```text
Order placed
```

Use:

```text
Order Service
    ↓
Queue
    ↓
Notification Worker
```

This removes notification latency from the critical request path.

---

# 85. Final Mental Model

Scaling is not:

```text
"Add more servers."
```

Scaling is:

```text
Measure
  ↓
Find bottleneck
  ↓
Optimize
  ↓
Scale the bottleneck
  ↓
Protect dependencies
  ↓
Monitor
```

---

# 86. Final Revision

Remember these concepts:

```text
Vertical scaling
Horizontal scaling
Stateless architecture
Load balancing
Health checks
Autoscaling
Caching
Read replicas
Database bottlenecks
Connection pool limits
Queue-based scaling
Backpressure
Load shedding
Graceful degradation
Circuit breakers
Bulkheads
Capacity headroom
Cost-aware scaling
```

---

# 87. One-Minute Interview Answer

### "How would you scale a backend system?"

> "I'd first identify the bottleneck instead of immediately adding infrastructure. For the application tier, I'd keep services stateless and run multiple instances behind a load balancer, with autoscaling based on appropriate metrics. For read-heavy workloads I'd consider caching and read replicas. For background work I'd use asynchronous processing and queues. I'd also monitor database connections, downstream dependencies and queue depth, because scaling the application can simply move the bottleneck elsewhere."

---

# 88. Key Takeaway

> **Scalability is about removing bottlenecks, not blindly adding servers. Make the application stateless, scale horizontally where appropriate, protect shared dependencies, and let measurements drive the next architectural decision.**

**File 03 complete.**
