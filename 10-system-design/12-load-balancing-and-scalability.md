# System Design — File 12: Load Balancing & Scalability

Load balancing distributes incoming traffic across multiple healthy servers or service instances.

Basic architecture:

```text
Clients
   |
   v
Load Balancer
 /    |    \
v     v     v
S1    S2    S3
```

The main goals are:

- High availability
- Horizontal scaling
- Traffic distribution
- Health-aware routing
- Better fault tolerance

---

## 1. Why Do We Need Load Balancing?

Without a load balancer:

```text
Clients
   |
   v
One Server
```

Problems:

- Single point of failure
- Limited capacity
- Difficult scaling
- Uneven workload

With a load balancer:

```text
Clients
   |
   v
Load Balancer
 /    |    \
S1    S2    S3
```

Traffic can be distributed across multiple instances.

---

## 2. Horizontal vs Vertical Scaling

### Vertical Scaling

Increase resources on one machine:

```text
4 CPU + 8 GB RAM
        |
        v
16 CPU + 64 GB RAM
```

Advantages:

- Simple
- Minimal application changes

Limitations:

- Hardware limits
- Expensive at larger sizes
- Often still a single failure domain

### Horizontal Scaling

Add more instances:

```text
Server 1
Server 2
Server 3
Server 4
```

Advantages:

- Better capacity
- Better availability
- Easier replacement of failed instances

For highly scalable web services, horizontal scaling is usually preferred.

---

## 3. Layer 4 vs Layer 7 Load Balancing

### Layer 4

Works around transport-level information:

```text
TCP
UDP
IP
Port
```

It doesn't need to understand HTTP semantics.

Advantages:

- High performance
- Lower overhead
- Protocol independent

### Layer 7

Understands application protocols such as HTTP.

It can route using:

```text
Host
Path
Headers
Cookies
HTTP method
```

Example:

```text
/api/orders
    |
    v
Order Service

/api/products
    |
    v
Product Service
```

### Interview Answer

> "L4 load balancing operates at the transport layer, while L7 understands application-level protocols such as HTTP and can make routing decisions based on paths, headers or hosts."

---

## 4. Common Load-Balancing Algorithms

Important algorithms:

```text
Round Robin
Weighted Round Robin
Least Connections
IP Hash
Consistent Hashing
Random
```

---

## 5. Round Robin

Requests are distributed sequentially:

```text
Request 1 -> S1
Request 2 -> S2
Request 3 -> S3
Request 4 -> S1
Request 5 -> S2
```

Good when:

- Servers have similar capacity
- Request costs are reasonably similar
- Services are stateless

---

## 6. Weighted Round Robin

Servers receive different weights.

```text
S1 -> weight 5
S2 -> weight 3
S3 -> weight 2
```

S1 receives more traffic.

Useful when servers have different capacities.

---

## 7. Least Connections

New traffic goes to the server with the fewest active connections.

Example:

```text
S1 -> 50 connections
S2 -> 20 connections
S3 -> 30 connections
```

New request:

```text
        |
        v
       S2
```

Useful when request duration varies significantly.

---

## 8. IP Hash

The client's IP contributes to server selection.

```text
hash(clientIP)
      |
      v
   Server
```

It can provide a form of session affinity.

However, many users can share one public IP, so distribution may become uneven.

---

## 9. Consistent Hashing

Keys are mapped onto a hash ring.

```text
          S1
       /      \
     S4        S2
       \      /
          S3
```

Useful for:

- Distributed caches
- Partition routing
- Affinity/locality

Compared with simple modulo hashing, consistent hashing can reduce the amount of remapping when nodes are added or removed.

---

## 10. Health Checks

A load balancer should avoid sending traffic to unhealthy instances.

Example:

```text
S1 -> Healthy
S2 -> Unhealthy
S3 -> Healthy
```

Traffic should go to:

```text
S1
S3
```

not S2.

---

## 11. Liveness vs Readiness

### Liveness

Answers:

> "Is the application process alive?"

### Readiness

Answers:

> "Can this instance safely receive traffic?"

An application can be alive but not ready.

Example:

```text
Application process -> running
Database dependency -> unavailable
```

Possible state:

```text
Liveness  = healthy
Readiness = unhealthy
```

Readiness is especially useful during startup, shutdown and deployments.

---

## 12. Graceful Shutdown

When an instance is being terminated:

```text
Stop accepting new traffic
        |
        v
Finish active requests
        |
        v
Close resources
        |
        v
Shutdown
```

This reduces failed requests during:

- Deployments
- Autoscaling
- Maintenance

---

## 13. Connection Draining

A load balancer can stop sending new requests to an instance while allowing existing connections to finish.

```text
New traffic
     |
     +------> Other instances

Existing connections
     |
     +------> Draining instance
```

This is useful when removing instances from service.

---

## 14. Sticky Sessions

Sticky sessions, also called session affinity, try to keep a client on the same server.

```text
User A -> Server 1
User A -> Server 1
User A -> Server 1
```

Potential problem:

```text
Server 1 -> overloaded
Server 2 -> lightly loaded
Server 3 -> lightly loaded
```

Sticky sessions can also make failover harder.

Prefer stateless services where practical.

---

## 15. Externalizing Session State

Instead of:

```text
Server
  |
Local session
```

use:

```text
Server
  |
  v
Redis
```

Now any application instance can handle the request.

This makes horizontal scaling easier.

---

## 16. Stateless Services

A stateless service does not rely on local instance memory for critical user state.

State can be stored in:

```text
Database
Redis
Object storage
Token
```

Benefits:

- Easier load balancing
- Easier autoscaling
- Easier failover
- Easier deployments

---

## 17. Autoscaling

Autoscaling adds or removes instances according to demand.

Example:

```text
Low traffic
    |
    v
2 instances

High traffic
    |
    v
10 instances
```

Possible scaling signals:

```text
CPU
Memory
Requests/sec
Latency
Queue depth
Kafka consumer lag
Custom business metrics
```

CPU is useful, but it is not always the best signal.

---

## 18. Scale-Out vs Scale-In

### Scale-Out

Add instances:

```text
2 -> 5 -> 10
```

### Scale-In

Remove instances:

```text
10 -> 5 -> 2
```

Scale-in should be graceful so active work isn't unnecessarily interrupted.

---

## 19. Reactive vs Scheduled Scaling

### Reactive

Scale after a metric crosses a threshold:

```text
CPU > 70%
    |
    v
Add instances
```

### Scheduled

Scale based on known traffic patterns:

```text
09:00
  |
  v
Increase capacity
```

Scheduled scaling is useful for predictable traffic.

---

## 20. Predictive Scaling

Predictive scaling attempts to increase capacity before demand arrives.

Example:

```text
Expected sale at 10:00
        |
        v
Increase capacity at 09:45
```

This can reduce scaling delay during predictable spikes.

---

## 21. Don't Scale Blindly

Adding application instances doesn't fix every bottleneck.

Example:

```text
20 API instances
       |
       v
One overloaded database
```

The database is the bottleneck.

Always identify the actual saturated resource.

---

## 22. End-to-End Scaling

A production system may contain:

```text
Client
  |
  v
CDN
  |
  v
Load Balancer
  |
  v
API Servers
  |
  +----> Redis
  |
  +----> Database
  |
  +----> Kafka
           |
           v
        Workers
```

Every layer can become a bottleneck.

---

## 23. Load Balancer High Availability

Avoid:

```text
Clients
   |
   v
One Load Balancer
   X
```

If the load balancer fails, traffic may be unable to reach the application.

Use:

- Multiple instances
- Managed highly available load-balancing services
- Redundant infrastructure

---

## 24. DNS Load Balancing

DNS can return multiple addresses:

```text
api.example.com
     |
     +--> IP1
     +--> IP2
     +--> IP3
```

It can distribute traffic between regions or endpoints.

However:

```text
DNS routing
```

and:

```text
Request-level load balancing
```

are different things.

DNS is not a full replacement for an application-aware load balancer.

---

## 25. Global / Geographic Load Balancing

Users can be routed to nearby regions.

Example:

```text
India -> Mumbai
Europe -> Frankfurt
US -> Virginia
```

Benefits:

- Lower latency
- Regional isolation
- Better user experience

Challenges:

- Cross-region data
- Failover
- Consistency
- Operational complexity

---

## 26. Active-Active

Multiple regions actively serve traffic.

```text
Global Router
   /       \
  v         v
Region A  Region B
Active     Active
```

Benefits:

- High availability
- Regional traffic distribution
- Better geographic latency

Challenges:

- Data consistency
- Cross-region writes
- Failover
- Cost

---

## 27. Active-Passive

One region is active and another is on standby.

```text
Region A -> Active
Region B -> Standby
```

If Region A fails:

```text
Traffic
   |
   v
Region B
```

This is often simpler than active-active but may leave standby capacity underutilized.

---

## 28. CDN vs Load Balancer

### CDN

Primarily handles:

```text
Edge caching
Static content delivery
```

### Load Balancer

Primarily handles:

```text
Traffic distribution
Health-aware routing
```

They commonly work together:

```text
Client
  |
  v
CDN
  |
  v
Load Balancer
  |
  v
Application
```

---

## 29. Queue-Based Scaling

For asynchronous work:

```text
Producer
   |
   v
Queue
   |
   v
Workers
```

Workers can scale based on:

```text
Queue depth
Message age
Consumer lag
```

This is often more meaningful than CPU alone.

---

## 30. Backpressure

If:

```text
Incoming work > Processing capacity
```

the system needs a strategy.

Possible mechanisms:

```text
Queue
Rate limiting
Throttling
Load shedding
Autoscaling
```

Without backpressure:

```text
Memory grows
Threads grow
Latency grows
System becomes unstable
```

---

## 31. Rate Limiting vs Load Balancing

These solve different problems.

### Rate Limiting

Controls how much traffic is accepted.

```text
100 requests/sec/client
```

### Load Balancing

Distributes accepted traffic.

```text
S1
S2
S3
```

A production system may need both.

---

## 32. Load Shedding

During extreme overload, protect critical functionality by rejecting or degrading lower-priority work.

Example:

```text
Keep:
Checkout
Payment
Order creation

Degrade:
Recommendations
Analytics
Personalization
```

This prevents one overloaded feature from taking down the entire system.

---

## 33. Priority Queues

Not all work has equal importance.

Example:

```text
Payment jobs    -> High priority
Analytics jobs  -> Low priority
```

Workers can process important work first.

---

## 34. Connection Pool Scaling Trap

Suppose:

```text
10 API instances
20 DB connections/instance
```

Total possible connections:

```text
10 × 20 = 200
```

If the database safely supports only:

```text
150 connections
```

scaling the API to more instances can make the database problem worse.

Always consider downstream capacity.

---

## 35. Thread Pools

Application servers also have limited worker threads.

If every request waits on a slow dependency:

```text
Thread Pool
████████████████
```

threads can become exhausted.

Useful protections include:

```text
Timeouts
Bulkheads
Async processing
Connection-pool tuning
Circuit breakers
```

---

## 36. Request Amplification

One incoming request can create many downstream calls.

Example:

```text
Gateway
   |
   v
Order Service
   |
   +--> User
   +--> Product
   +--> Inventory
   +--> Pricing
   +--> Recommendation
```

One request becomes five downstream calls.

At:

```text
10,000 RPS
```

that could create roughly:

```text
50,000 downstream calls/sec
```

This is request amplification.

---

## 37. Caching to Reduce Amplification

If product information changes infrequently:

```text
Order Service
     |
     v
Redis
```

can avoid calling Product Service for every request.

Benefits:

- Lower latency
- Less network traffic
- Less database load

Trade-off:

- Stale data
- Invalidation complexity

---

## 38. Local vs Distributed Cache

### Local Cache

```text
API instance
    |
Local memory
```

Very fast, but every instance has its own cache.

### Distributed Cache

```text
API 1 --API 2 ----> Redis
API 3 --/
```

Shared across instances.

Redis is a common choice.

---

## 39. Capacity Planning

Estimate:

```text
Peak RPS
Request latency
Capacity per instance
Database capacity
Cache hit rate
Failure requirements
```

Example:

```text
Peak traffic = 10,000 RPS
One instance = 500 RPS
```

Initial baseline:

```text
10,000 / 500 = 20 instances
```

Then add headroom for:

```text
Traffic spikes
Instance failures
Deployments
Capacity variation
```

---

## 40. Little's Law

A useful relationship:

```text
L = λW
```

Where:

```text
L = average number of items in the system
λ = throughput
W = average time in the system
```

Example:

```text
λ = 100 requests/sec
W = 0.2 sec
```

Then:

```text
L = 100 × 0.2
  = 20 concurrent requests
```

This helps reason about concurrency.

---

## 41. Latency vs Throughput

### Latency

How long one request takes:

```text
200 ms
```

### Throughput

How much work the system completes:

```text
5,000 requests/sec
```

A system can have:

```text
High throughput
but high latency
```

or:

```text
Low latency
but low throughput
```

Optimize according to requirements.

---

## 42. Tail Latency

Monitor:

```text
p50
p95
p99
```

Example:

```text
p50 = 80 ms
p95 = 200 ms
p99 = 800 ms
```

Average latency can hide slow requests.

Tail latency may reveal:

- Slow queries
- GC pauses
- Network issues
- Contention
- Cache misses

---

## 43. Load Testing

Useful tests include:

```text
Load test
Stress test
Spike test
Soak test
Failure test
```

---

## 44. Stress Testing

Gradually increase traffic until performance becomes unacceptable.

Goal:

```text
Find system capacity limits
```

---

## 45. Spike Testing

Suddenly increase traffic.

Example:

```text
1K RPS
  |
  v
20K RPS
```

Tests:

- Autoscaling
- Queues
- Rate limiting
- Load balancing
- Recovery

---

## 46. Soak Testing

Run a sustained workload for a long time.

Useful for detecting:

```text
Memory leaks
Connection leaks
Resource exhaustion
Gradual degradation
```

---

## 47. Failure Testing

Test:

```text
Server failure
Database failure
Network failure
Dependency timeout
Region failure
```

The goal is to validate resilience, not only performance.

---

## 48. Zero-Downtime Deployment

A common process:

```text
Old instances
      |
      v
Start new instances
      |
      v
Health checks pass
      |
      v
Send traffic to new instances
      |
      v
Drain old instances
      |
      v
Terminate old instances
```

This requires:

```text
Readiness checks
Graceful shutdown
Connection draining
Backward-compatible changes
```

---

## 49. Canary Deployment

Send a small percentage of traffic to the new version.

```text
95% -> v1
5%  -> v2
```

Monitor:

```text
Errors
Latency
Resource usage
Business metrics
```

Then gradually increase:

```text
5% -> 25% -> 50% -> 100%
```

if healthy.

---

## 50. Blue-Green Deployment

Maintain two environments:

```text
Blue  -> Current
Green -> New
```

Switch traffic:

```text
Blue
  |
  v
Green
```

Rollback can be fast because the previous environment remains available.

---

## 51. WebSocket Scaling

WebSockets are long-lived connections.

Example:

```text
Client A -> Server 1
Client B -> Server 2
```

If Server 1 needs to send an event to Client B, it may need shared communication:

```text
Server 1
   |
   v
Pub/Sub
   |
   v
Server 2
   |
   v
Client B
```

Redis Pub/Sub or another messaging mechanism can help, depending on reliability requirements.

---

## 52. WAF

A Web Application Firewall can inspect HTTP traffic and block common malicious patterns.

Examples include:

```text
SQL injection patterns
Cross-site scripting patterns
Malicious HTTP requests
```

WAF complements application-level security.

---

## 53. DDoS Protection

Large traffic attacks can overwhelm infrastructure.

Defense can include:

```text
CDN
DDoS protection
WAF
Rate limiting
Load balancer
Autoscaling
```

Do not rely only on autoscaling to handle an attack.

---

## 54. Availability vs Scalability

### Availability

The system remains usable despite failures.

### Scalability

The system can handle increasing workload.

They are related but different.

A system can scale well but have poor availability, or be highly available but unable to handle large traffic.

---

## 55. Fault Tolerance

A fault-tolerant system continues operating despite certain failures.

Example:

```text
Server 1 fails
     |
     v
Load Balancer
     |
     v
Server 2
```

The user may not notice the failure.

---

## 56. Redundancy

Avoid single points of failure where the availability requirement does not permit them.

Potential redundancy:

```text
Multiple API instances
Multiple load balancer instances
Database replicas
Multiple availability zones
Multiple regions
```

---

## 57. RTO and RPO

### RTO

Recovery Time Objective:

> How quickly must the system recover?

### RPO

Recovery Point Objective:

> How much data loss is acceptable?

Example:

```text
RTO = 5 minutes
RPO = 1 minute
```

These requirements influence:

```text
Replication
Backup
Failover
Multi-region design
```

---

## 58. Interview Question — What Is Load Balancing?

Answer:

> "Load balancing distributes traffic across multiple healthy instances so the application can scale horizontally, improve availability and avoid overloading one server."

---

## 59. Interview Question — L4 vs L7?

Answer:

> "L4 operates using transport-level information such as TCP connections, while L7 understands application-level protocols like HTTP and can route based on paths, headers or hosts."

---

## 60. Interview Question — Round Robin vs Least Connections?

Answer:

> "Round Robin distributes requests sequentially and works well when instances and request costs are similar. Least Connections considers active connections and can work better when request duration varies."

---

## 61. Interview Question — Why Avoid Sticky Sessions?

Answer:

> "Sticky sessions can create uneven load and make failover harder. I'd generally prefer stateless services with state stored externally when possible."

---

## 62. Interview Question — How Would You Scale a Backend?

Answer:

> "I'd first identify the bottleneck using metrics and tracing. Depending on the bottleneck, I'd use horizontal scaling, caching, read replicas, asynchronous processing, partitioning or sharding rather than blindly adding application instances."

---

## 63. Interview Question — What Happens If One Server Fails?

Answer:

> "Health checks should remove the failed instance from rotation and the load balancer should route new traffic to healthy instances. Graceful shutdown and externalized state help minimize user-visible impact."

---

## 64. Interview Question — How Do You Handle Traffic Spikes?

Answer:

> "I'd use CDN and caching where applicable, load balancing, autoscaling and rate limiting. For asynchronous workloads I'd use a queue or Kafka to absorb bursts and scale workers based on queue depth or consumer lag."

---

## 65. Interview Question — What Is Horizontal Scaling?

Answer:

> "Horizontal scaling means adding more instances instead of making one machine larger. It increases capacity and can improve availability when instances are independently replaceable."

---

## 66. Interview Question — What Is Vertical Scaling?

Answer:

> "Vertical scaling increases CPU, memory or other resources on an existing machine. It's simple but has hardware and cost limits."

---

## 67. Interview Question — How Do You Prevent Database Overload During Autoscaling?

Answer:

> "I'd consider downstream capacity before increasing application instances. I'd control connection-pool sizes, use caching and replicas where appropriate, and make sure total database connections and query throughput stay within the database's capacity."

---

## 68. Interview Question — What Is Graceful Shutdown?

Answer:

> "The instance stops receiving new traffic, allows active requests to complete, closes resources cleanly and then terminates. This helps prevent failed requests during deployments and scaling."

---

## 69. Interview Question — What Is Connection Draining?

Answer:

> "The load balancer stops sending new traffic to an instance while allowing existing connections to finish before the instance is removed."

---

## 70. Interview Question — What Is Load Shedding?

Answer:

> "When the system is overloaded, it intentionally rejects or degrades lower-priority work to protect critical functionality and prevent total failure."

---

## 71. Practical Scenario — Traffic Suddenly Becomes 10× Higher

Think through:

```text
CDN
  |
Rate limiting
  |
Load Balancer
  |
Autoscaling
  |
Cache
  |
Database
```

Then ask:

> Which layer is actually saturated?

Don't assume the API servers are automatically the bottleneck.

---

## 72. Practical Scenario — CPU Is Low but Latency Is High

Don't immediately add more servers.

Investigate:

```text
Database latency
Connection pool waits
Downstream latency
Lock contention
Queueing
Network
```

Distributed tracing can identify where time is being spent.

---

## 73. Practical Scenario — API Scales From 5 to 50 Instances and Database Crashes

Likely areas to investigate:

```text
Database connections
Queries/sec
Connection pool size
Cache hit rate
Database CPU
Database IO
```

More application instances can increase pressure on the database.

---

## 74. Practical Scenario — One Server Receives Much More Traffic

Check:

```text
Load-balancing algorithm
Sticky sessions
Health status
Weights
Hash distribution
```

A traffic imbalance doesn't automatically mean the load balancer is broken.

---

## 75. Practical Scenario — Deployment Causes 500 Errors

Check:

```text
Readiness checks
Connection draining
Graceful shutdown
Load-balancer health checks
Backward compatibility
Database migration compatibility
```

---

## 76. Practical Scenario — One Region Fails

A multi-region architecture needs:

```text
Global health-aware routing
Capacity in another region
Data replication/failover strategy
Recovery procedures
```

The database strategy is often the hardest part of multi-region failover.

---

## 77. Final Architecture

A scalable web application can look like:

```text
                         Users
                           |
                           v
                          CDN
                           |
                           v
                    Global Routing
                           |
                           v
                    Load Balancer
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
            API           API           API
             |             |             |
             +-------------+-------------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
           Redis                       Kafka
             |                           |
             v                           v
         Database                     Workers
             |
        +----+----+
        |         |
        v         v
     Primary   Replicas
```

Each layer has a purpose:

```text
CDN            -> Edge caching
Global routing -> Regional traffic distribution
Load balancer  -> Instance distribution
API servers    -> Business logic
Redis          -> Low-latency caching
Kafka          -> Asynchronous processing
Database       -> Durable state
Workers        -> Background processing
```

---

## 78. Scaling Checklist

When designing a scalable system, ask:

```text
□ Expected RPS?
□ Peak RPS?
□ Average latency?
□ p95/p99 latency?
□ Is the service stateless?
□ Can it scale horizontally?
□ Where is the bottleneck?
□ Do we need a load balancer?
□ What health checks are required?
□ Do we need sticky sessions?
□ Can state be externalized?
□ Do we need caching?
□ Can traffic be queued?
□ Can work be asynchronous?
□ What is database capacity?
□ What is connection capacity?
□ What happens when one instance fails?
□ What happens during deployment?
□ What happens during a traffic spike?
□ What happens if a region fails?
```

---

## 79. One-Minute Interview Answer

### "How would you scale a Spring Boot backend?"

> "I'd first measure the bottleneck rather than immediately adding servers. I'd keep the application stateless and run multiple instances behind a load balancer with health checks. I'd use Redis for suitable read-heavy data and database replicas if database reads become the bottleneck. For background work I'd use Kafka or a queue so traffic spikes don't block synchronous requests. I'd configure autoscaling based on meaningful metrics, control database connection pools so scaling doesn't overload the database, and use timeouts, circuit breakers and graceful shutdown for resilience."

---

## 80. Key Takeaway

> **Scalability is not simply adding more servers. A scalable system distributes traffic, externalizes state, protects downstream dependencies, scales each bottleneck independently and remains healthy during spikes, failures and deployments.**

**File 12 complete.**
