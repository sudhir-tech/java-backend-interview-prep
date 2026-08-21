# System Design — File 04: Load Balancing & Traffic Distribution

This file focuses on how traffic reaches backend services once you have more than one application instance.

The basic idea:

```text
Clients
   ↓
Load Balancer
   ↓
+-------+-------+
|       |       |
App 1  App 2  App 3
```

The load balancer is one of the most important building blocks in scalable backend architecture.

---

# 1. What Is a Load Balancer?

A load balancer distributes incoming traffic across multiple backend servers or service instances.

Instead of:

```text
Client
  ↓
One Server
```

we have:

```text
Client
  ↓
Load Balancer
  ↓
Multiple Servers
```

---

# 2. Why Do We Need One?

Without a load balancer:

```text
Client
  ↓
App Server
```

That creates:

```text
Single point of failure
Limited capacity
Difficult scaling
```

With multiple instances:

```text
             Load Balancer
             /     |     \
            ↓      ↓      ↓
          App1    App2    App3
```

Traffic can be distributed.

---

# 3. Main Responsibilities

A load balancer may provide:

```text
Traffic distribution
Health checking
Failover
TLS termination
Connection management
Routing
```

The exact capabilities depend on the load balancer.

---

# 4. Layer 4 vs Layer 7

Two common categories:

```text
Layer 4
Layer 7
```

---

# 5. Layer 4 Load Balancing

Layer 4 operates using transport-level information such as:

```text
IP
TCP
UDP
Port
```

Example:

```text
Client
  ↓
TCP Load Balancer
  ↓
App1 / App2 / App3
```

It generally doesn't need to understand HTTP request details.

---

# 6. Layer 7 Load Balancing

Layer 7 understands application protocols such as HTTP.

It can route based on:

```text
URL path
Host
HTTP headers
Cookies
Methods
```

Example:

```text
/api/users
    ↓
User Service

/api/orders
    ↓
Order Service
```

---

# 7. Layer 4 vs Layer 7

Simple interview answer:

> "Layer 4 load balancing operates at the transport level using information such as IP and port, while Layer 7 understands application-level protocols such as HTTP and can make routing decisions based on paths, headers or hosts."

---

# 8. Reverse Proxy vs Load Balancer

They overlap in many modern systems.

A reverse proxy:

```text
Client
  ↓
Reverse Proxy
  ↓
Backend
```

A load balancer:

```text
Client
  ↓
Load Balancer
  ↓
Backend Pool
```

A modern Layer 7 proxy can effectively perform both roles.

---

# 9. Round Robin

The simplest strategy:

```text
Request 1 → App1
Request 2 → App2
Request 3 → App3
Request 4 → App1
Request 5 → App2
```

Useful when:

```text
Instances have similar capacity
Requests have relatively similar cost
```

---

# 10. Weighted Round Robin

Assign different weights.

Example:

```text
App1 → weight 1
App2 → weight 2
App3 → weight 1
```

Approximate distribution:

```text
App1 → 25%
App2 → 50%
App3 → 25%
```

Useful when:

```text
Instances have different capacities
```

---

# 11. Least Connections

Send traffic toward the instance with fewer active connections.

Example:

```text
App1 → 100 connections
App2 → 30 connections
App3 → 60 connections
```

A new connection may go to:

```text
App2
```

This can help when connection durations vary.

---

# 12. IP Hash

A hash of client information determines the backend.

Conceptually:

```text
Client A → App1
Client B → App3
Client C → App2
```

The same client may tend to reach the same backend.

This can be useful in specific scenarios but can create uneven distribution.

---

# 13. Consistent Hashing

Consistent hashing is commonly discussed for distributed systems where keys need to be mapped across changing nodes.

Example:

```text
Key
 ↓
Hash
 ↓
Node
```

When a node is added or removed, ideally only a portion of keys need to move.

This is particularly useful for:

```text
Distributed caches
Sharded systems
Partitioned workloads
```

It is different from simply choosing a backend using ordinary modulo hashing.

---

# 14. Health Checks

A load balancer should know whether a backend is healthy.

Example:

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

App2 is temporarily removed from normal routing.

---

# 15. Liveness vs Readiness

This distinction matters in containerized applications.

### Liveness

Answers:

> "Is this process alive?"

If not:

```text
Restart it
```

### Readiness

Answers:

> "Is this instance ready to receive traffic?"

If not:

```text
Don't send normal traffic
```

An application can be:

```text
Alive
but
Not ready
```

---

# 16. Spring Boot Health

Spring Boot Actuator can expose health endpoints.

Common endpoint:

```text
/actuator/health
```

A production setup may expose separate health information depending on configuration.

Don't blindly make detailed internal health information public.

---

# 17. Health Check Example

Imagine:

```text
App1:
Process running
DB unavailable

App2:
Healthy

App3:
Healthy
```

If the health check correctly detects that App1 isn't ready:

```text
Load Balancer
   /       \
 App2     App3
```

App1 doesn't receive new traffic.

---

# 18. What Makes a Good Health Check?

It should answer:

```text
Can this instance safely serve traffic?
```

Avoid checks that are:

```text
Too expensive
Too slow
Too broad
Dependent on every optional component
```

Otherwise a minor dependency issue could remove every instance from service.

---

# 19. Health Check Failure

Suppose:

```text
App1 → unhealthy
```

The load balancer can:

```text
Stop routing traffic
```

The orchestration platform may separately:

```text
Restart
Replace
Reschedule
```

These are related but separate responsibilities.

---

# 20. Connection Draining

Suppose App2 is being removed.

Existing requests:

```text
Client → App2
```

should ideally be allowed to finish.

New requests:

```text
Client → App2
```

should stop.

This is often called:

```text
Connection draining
Graceful draining
```

---

# 21. Why Draining Matters

Without draining:

```text
Deployment
 ↓
App instance terminated
 ↓
Active requests interrupted
 ↓
Errors
```

With draining:

```text
Stop new traffic
 ↓
Finish existing requests
 ↓
Terminate instance
```

---

# 22. Graceful Shutdown

A Spring Boot application should handle shutdown cleanly.

Think:

```text
Stop receiving new traffic
        ↓
Finish active work
        ↓
Close resources
        ↓
Exit
```

This works especially well with:

```text
Readiness
+
Load balancer
+
Orchestrator
```

---

# 23. TLS Termination

A load balancer can terminate HTTPS:

```text
Client
  |
 HTTPS
  ↓
Load Balancer
  |
 HTTP/HTTPS
  ↓
Application
```

This is called:

```text
TLS termination
```

---

# 24. Why Terminate TLS at the Load Balancer?

Potential benefits:

```text
Centralized certificate management
Reduced application overhead
Simpler application configuration
```

But security requirements may require encryption between internal components too.

---

# 25. TLS Re-encryption

A more secure internal design can be:

```text
Client
  ↓ HTTPS
Load Balancer
  ↓ HTTPS
Application
```

The load balancer decrypts the external connection and establishes another encrypted connection to the backend.

---

# 26. Routing by Host

Layer 7 routing can use the hostname.

Example:

```text
api.example.com
      ↓
API Service

admin.example.com
      ↓
Admin Service
```

---

# 27. Routing by Path

Example:

```text
/api/users
    ↓
User Service

/api/orders
    ↓
Order Service

/api/products
    ↓
Product Service
```

This is commonly used in API gateways and reverse proxies.

---

# 28. Routing by Header

A proxy can route based on HTTP headers.

For example:

```text
X-API-Version: v2
```

could route to:

```text
Version 2
```

Use this carefully because routing rules can become difficult to maintain.

---

# 29. Canary Routing

A small percentage of traffic goes to a new version.

Example:

```text
95% → v1
5%  → v2
```

If v2 performs well:

```text
90% → v1
10% → v2
```

Then:

```text
50% → v1
50% → v2
```

Eventually:

```text
100% → v2
```

---

# 30. Blue-Green Deployment

Two environments:

```text
Blue → Current version
Green → New version
```

Traffic initially:

```text
100% → Blue
```

After validation:

```text
100% → Green
```

Rollback:

```text
100% → Blue
```

This can make rollback fast but requires additional infrastructure/resources.

---

# 31. Canary vs Blue-Green

### Canary

```text
Small traffic
→ New version
```

Gradual rollout.

### Blue-Green

```text
Complete environment
→ Switch traffic
```

Fast cutover and rollback.

---

# 32. Load Balancer and Autoscaling

They often work together:

```text
Traffic increases
      ↓
Autoscaler creates App4
      ↓
Health check passes
      ↓
Load Balancer adds App4
```

When traffic falls:

```text
App4
 ↓
Drain
 ↓
Remove
```

---

# 33. Load Balancer and Stateless Services

Stateless applications make this simple.

```text
Request
  ↓
Any healthy instance
```

The load balancer doesn't need to remember which instance owns the user's state.

---

# 34. Sticky Sessions

With sticky sessions:

```text
User A → App1
User A → App1
User A → App1
```

The load balancer tries to maintain affinity.

Potential problems:

```text
Uneven traffic
Harder failover
Instance dependency
Scaling complexity
```

---

# 35. When Sticky Sessions May Be Useful

They can be reasonable when:

```text
Legacy application
Local session state
Migration constraints
Specific workload affinity
```

But don't use them just because they're easy.

---

# 36. Global Traffic Management

For globally distributed systems:

```text
Users
 /   \
Region A   Region B
```

A global routing layer may direct users toward an appropriate region.

Potential factors:

```text
Latency
Health
Geography
Capacity
Failover
```

---

# 37. DNS-Based Routing

DNS can direct clients toward different endpoints.

Example:

```text
api.example.com
       ↓
DNS
   /       \
Region A  Region B
```

DNS-based approaches have caching and TTL considerations, so failover isn't necessarily instantaneous.

---

# 38. Anycast

Some global networking architectures use:

```text
Same IP
 ↓
Nearest/appropriate network location
```

Routing infrastructure determines where traffic goes.

This is more advanced and typically managed by cloud/network providers.

---

# 39. Load Balancer as a Single Point of Failure

A badly designed architecture might have:

```text
Client
  ↓
ONE load balancer
  ↓
Apps
```

If that load balancer fails:

```text
System unavailable
```

Managed/load-balancing platforms usually provide redundancy.

---

# 40. Redundant Load Balancers

Conceptually:

```text
             Global/External Layer
                    ↓
             +------+------+
             |             |
            LB1           LB2
             |             |
          App pool      App pool
```

The exact implementation depends on the infrastructure.

---

# 41. Layered Traffic Architecture

A larger architecture may look like:

```text
Internet
   ↓
CDN
   ↓
WAF
   ↓
Load Balancer
   ↓
API Gateway
   ↓
Services
```

Each layer has a purpose.

---

# 42. CDN

A CDN is useful for:

```text
Static assets
Images
Videos
Cacheable content
```

It reduces traffic reaching the origin.

---

# 43. WAF

A Web Application Firewall can inspect HTTP traffic and help protect against common web attacks.

Conceptually:

```text
Internet
   ↓
WAF
   ↓
Load Balancer
```

WAF is not a replacement for secure application code.

---

# 44. API Gateway vs Load Balancer

A load balancer mainly focuses on:

```text
Traffic distribution
Health
Routing
```

An API gateway may additionally provide:

```text
Authentication
Rate limiting
Request transformation
API policies
Quotas
```

They can overlap depending on the platform.

---

# 45. Reverse Proxy vs API Gateway

A reverse proxy:

```text
General traffic proxy
```

An API gateway:

```text
API-focused policy and routing layer
```

In real systems, the boundaries can overlap.

---

# 46. Request Flow

A typical modern backend might be:

```text
Client
  ↓
DNS
  ↓
CDN / WAF
  ↓
Load Balancer
  ↓
API Gateway
  ↓
Spring Boot instances
```

The exact layers depend on the system.

---

# 47. What If One Application Instance Fails?

Suppose:

```text
App1 → crashed
App2 → healthy
App3 → healthy
```

Health checks detect App1.

Traffic:

```text
Load Balancer
    /      \
  App2    App3
```

If enough capacity remains:

```text
Users may not notice.
```

---

# 48. What If All Instances Fail?

Then the load balancer cannot solve the problem.

You need:

```text
Application redundancy
Capacity
Autoscaling
Multi-zone deployment
Recovery procedures
```

A load balancer is not a magic availability solution.

---

# 49. Availability Zones

Deploy instances across multiple zones:

```text
Region
│
├── AZ1
│    └── App1
│
├── AZ2
│    └── App2
│
└── AZ3
     └── App3
```

If one zone fails:

```text
Other zones
   ↓
Continue serving
```

assuming dependencies are also resilient.

---

# 50. Cross-Zone Load Balancing

A load balancer may distribute traffic across instances in multiple zones.

This helps avoid:

```text
Traffic concentrated in one zone
```

But cross-zone traffic can have:

```text
Cost
Latency
Network considerations
```

depending on the provider.

---

# 51. Request Affinity

Some workloads need requests for the same key to reach the same backend.

Examples:

```text
WebSocket connection
Stateful legacy session
Cache locality
Partition ownership
```

Possible techniques:

```text
Sticky sessions
Consistent hashing
Partition-aware routing
```

Use them only when justified.

---

# 52. WebSockets

WebSockets create long-lived connections.

Load balancing becomes more interesting because:

```text
Connection
   ↓
Specific instance
```

The connection remains there until closed.

Scaling requires:

```text
Connection distribution
Shared state/pub-sub where needed
Connection draining
```

---

# 53. WebSocket Example

Chat:

```text
User A → App1
User B → App2
```

If messages must reach both users:

```text
App1
  ↓
Redis Pub/Sub / Kafka / messaging
  ↓
App2
```

The exact choice depends on requirements.

---

# 54. Long-Running Requests

Suppose an API request takes:

```text
30 seconds
```

Load balancer and proxy timeouts matter.

If configured for:

```text
10 seconds
```

the request may fail even if the backend eventually finishes.

For long work, consider:

```text
Async job
Queue
Polling
Webhook
```

instead of keeping HTTP connections open unnecessarily.

---

# 55. Retry at the Load Balancer

Some infrastructure can retry requests to another backend.

Be careful with:

```text
POST
Payment
Order creation
```

Blind retries can duplicate operations.

Idempotency is important.

---

# 56. Idempotency + Load Balancing

Suppose:

```text
Client
 ↓
LB
 ↓
App1
```

App1 processes the payment but response is lost.

LB/client retries:

```text
Client
 ↓
LB
 ↓
App2
```

Without idempotency:

```text
Payment twice
```

With:

```text
Idempotency-Key
```

App2 can detect the existing operation.

---

# 57. Load Balancer Metrics

Monitor:

```text
Requests/sec
Latency
5xx errors
4xx errors
Healthy instances
Unhealthy instances
Connection count
Traffic distribution
```

These metrics help detect:

```text
Overload
Bad deployments
Uneven routing
Instance failures
```

---

# 58. Uneven Traffic

Suppose:

```text
App1 → 80%
App2 → 10%
App3 → 10%
```

Possible causes:

```text
Sticky sessions
Unequal weights
Long-lived connections
Hashing
Unhealthy instances
```

Don't assume the load balancer algorithm is broken.

---

# 59. Connection Draining During Deployment

Deployment:

```text
v1
 ↓
Remove instance
```

Better:

```text
Mark not ready
 ↓
Stop new traffic
 ↓
Drain active requests
 ↓
Terminate
```

Then:

```text
Start v2
 ↓
Health check
 ↓
Receive traffic
```

---

# 60. Load Balancing Interview Question

### Why do we need a load balancer?

Answer:

> "When we have multiple application instances, the load balancer distributes traffic among them, performs health checks and helps us remove unhealthy instances from rotation. It also supports horizontal scaling and improves availability."

---

# 61. Interview Question

### Layer 4 vs Layer 7?

Answer:

> "Layer 4 works at the transport level using information such as IP, TCP and port. Layer 7 understands HTTP and can route based on paths, hosts, headers or other application-level information."

---

# 62. Interview Question

### What happens if one server goes down?

Answer:

> "The load balancer's health check should detect the unhealthy instance and stop sending new traffic to it. If enough healthy instances remain, the service can continue operating."

---

# 63. Interview Question

### What is sticky session?

Answer:

> "Sticky session means the load balancer tries to keep a client associated with the same backend instance. It can help legacy stateful applications but makes scaling and failover more complicated."

---

# 64. Interview Question

### What is connection draining?

Answer:

> "During instance removal, the load balancer stops sending new requests to the instance while allowing existing requests to finish before terminating it."

---

# 65. Interview Question

### What is a health check?

Answer:

> "A health check determines whether a backend instance is healthy or ready to serve traffic. Unhealthy instances can be removed from the load-balancing pool."

---

# 66. Interview Question

### How would you deploy a new version safely?

Answer:

> "I'd use a rolling, canary or blue-green deployment depending on the requirements. The new instances should pass readiness checks before receiving traffic, and existing instances should be drained gracefully."

---

# 67. Interview Question

### Why isn't a load balancer enough for high availability?

Answer:

> "The load balancer only distributes traffic. The application instances, database, cache and other dependencies also need redundancy and appropriate failure handling. Otherwise the actual bottleneck or single point of failure simply moves elsewhere."

---

# 68. Interview Question

### Why should application instances be stateless behind a load balancer?

Answer:

> "Stateless instances allow any healthy server to handle a request. This makes traffic distribution, autoscaling and failover much easier."

---

# 69. Practical Scenario

### Scenario:

```text
App1 → 1,000 RPS
App2 → 100 RPS
App3 → 100 RPS
```

What would you investigate?

```text
Sticky sessions
Weights
Hashing
Long-lived connections
Health status
Instance capacity
```

---

# 70. Practical Scenario

### Scenario:

```text
Deployment starts
 ↓
5xx errors increase
```

Check:

```text
Readiness checks
Health checks
Connection draining
New version logs
Dependency compatibility
Traffic percentage
```

If canary:

```text
Reduce/stop traffic to new version
```

---

# 71. Practical Scenario

### Scenario:

```text
One availability zone fails.
```

A resilient architecture should have:

```text
Instances in other zones
Load balancer
Healthy capacity
Resilient database/cache dependencies
```

---

# 72. Practical Scenario

### Scenario:

```text
Payment POST
```

Load balancer retries it after a timeout.

What protects against duplicate payment?

```text
Idempotency key
```

---

# 73. Practical Scenario

### Scenario:

```text
HTTP request takes 2 minutes.
```

Should the system keep the connection open?

Not necessarily.

Consider:

```text
Submit job
 ↓
Return job ID
 ↓
Process asynchronously
 ↓
Client polls/status callback
```

This can make the system more resilient.

---

# 74. Recommended Traffic Architecture

For a typical scalable Java backend:

```text
                    Internet
                       |
                       ↓
                     DNS
                       |
                       ↓
                  CDN / WAF
                       |
                       ↓
                Load Balancer
                       |
                       ↓
                 API Gateway
                       |
              +--------+--------+
              |        |        |
             App1     App2     App3
              |        |        |
              +--------+--------+
                       |
                Cache / Database
```

This is an example, not a mandatory architecture.

---

# 75. Final Revision Checklist

You should be able to explain:

```text
□ What a load balancer does
□ Why horizontal scaling needs traffic distribution
□ Layer 4 vs Layer 7
□ Round robin
□ Weighted routing
□ Least connections
□ Hash-based routing
□ Health checks
□ Liveness vs readiness
□ Connection draining
□ Graceful shutdown
□ TLS termination
□ Host/path routing
□ Canary deployments
□ Blue-green deployments
□ Sticky sessions
□ Global routing
□ DNS-based routing
□ Availability zones
□ WAF/CDN relationship
□ API gateway vs load balancer
□ WebSocket considerations
□ Retry + idempotency
□ Load-balancer metrics
□ Failure scenarios
```

---

# 76. Final Mental Model

Think:

```text
Clients
   ↓
Traffic Management
   ↓
Load Balancer
   ↓
Healthy Instances
   ↓
Stateless Application
   ↓
Shared Dependencies
```

During deployment:

```text
New instance
    ↓
Readiness check
    ↓
Healthy
    ↓
Receive traffic
```

During removal:

```text
Not ready
    ↓
Stop new traffic
    ↓
Drain
    ↓
Terminate
```

---

# 77. Key Takeaway

> **A load balancer is not simply a traffic distributor. In a scalable backend, it becomes part of the system's availability, routing, deployment and failure-handling strategy.**

The most important mental model is:

```text
Multiple instances
       ↓
Load Balancer
       ↓
Health checks
       ↓
Only healthy instances receive traffic
       ↓
Graceful draining during changes
       ↓
Stateless services make scaling easier
```

**File 04 complete.**
