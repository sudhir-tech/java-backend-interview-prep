# System Design — File 01: Fundamentals

This is the starting point for the System Design section.

The goal is to understand how to think about a backend system before jumping into:

```text
Microservices
Kafka
Redis
Sharding
Kubernetes
```

System design is mainly about making good engineering trade-offs.

---

# 1. What Is System Design?

System design is the process of deciding:

```text
What components do we need?
How do they communicate?
Where is data stored?
How does the system scale?
What happens when something fails?
How do we keep it secure and observable?
```

A simple backend:

```text
Client
  ↓
Spring Boot
  ↓
MySQL
```

A larger system might become:

```text
Clients
   ↓
Load Balancer
   ↓
API Gateway
   ↓
Services
 /   |   \
DB  Redis Kafka
```

---

# 2. HLD vs LLD

This distinction is important for interviews.

## HLD — High-Level Design

Focuses on the overall system:

```text
Services
Databases
Cache
Queues
Load balancers
Networks
Scaling
Availability
```

Example:

```text
User
 ↓
API Gateway
 ↓
Order Service
 ↓
MySQL
```

## LLD — Low-Level Design

Focuses on the internal design of a component:

```text
Classes
Interfaces
Methods
Objects
Relationships
Design patterns
SOLID
```

Example:

```text
OrderService
     |
OrderRepository
     |
Order
```

---

# 3. Simple Rule

Remember:

```text
HLD
→ How the system is structured

LLD
→ How the code inside the system is structured
```

Both are part of broader system-design preparation.

---

# 4. What Does an Interviewer Want?

They usually don't expect one "perfect architecture."

They want to see whether you can:

```text
Understand requirements
Estimate scale
Identify bottlenecks
Choose appropriate components
Explain trade-offs
Handle failures
Think about security
Think about observability
```

---

# 5. Start With Requirements

Don't immediately draw:

```text
Kafka
Redis
Kubernetes
Microservices
```

First ask:

```text
What does the system need to do?
```

For example, an e-commerce system might need:

```text
User registration
Login
Product browsing
Cart
Order placement
Payment
Order tracking
```

---

# 6. Functional Requirements

Functional requirements describe:

```text
What the system does
```

Example:

```text
User can register.
User can log in.
User can search products.
User can add products to cart.
User can place an order.
User can view order history.
```

---

# 7. Non-Functional Requirements

Non-functional requirements describe:

```text
How well the system should work
```

Examples:

```text
Latency
Availability
Scalability
Reliability
Security
Consistency
Durability
Maintainability
```

Example:

```text
API should respond within 200 ms for most requests.
```

---

# 8. Functional vs Non-Functional

Easy interview example:

### Functional

> "User can place an order."

### Non-functional

> "The order API should remain available during high traffic."

---

# 9. Requirements Clarification

Suppose interviewer says:

> "Design an e-commerce platform."

Don't start drawing immediately.

Ask:

```text
How many users?
How many requests?
Read/write ratio?
Expected availability?
Geographic scope?
Payment required?
Real-time requirements?
Search required?
```

You don't need to ask every possible question.

Ask the questions that materially affect the architecture.

---

# 10. Scale Matters

Architecture depends heavily on scale.

Example:

```text
1,000 users
```

and:

```text
100 million users
```

shouldn't necessarily use the same architecture.

For a small system:

```text
Spring Boot
   ↓
MySQL
```

may be enough.

For a very large system:

```text
Load Balancer
      ↓
Multiple application instances
      ↓
Cache / DB / Queue / Services
```

may be required.

---

# 11. Back-of-the-Envelope Estimation

Before designing, estimate:

```text
Users
Requests per second
Storage
Bandwidth
```

You don't need perfect numbers.

You need reasonable assumptions.

---

# 12. Example Estimation

Suppose:

```text
10 million users
```

Assume:

```text
10% active daily
```

Then:

```text
1 million daily active users
```

If each makes:

```text
10 requests/day
```

then:

```text
10 million requests/day
```

---

# 13. Convert Daily Requests to RPS

Approximate:

```text
10,000,000 requests/day
```

A day has:

```text
86,400 seconds
```

So:

```text
10,000,000 / 86,400
≈ 116 requests/second
```

Average:

```text
≈ 116 RPS
```

Peak traffic could be several times higher.

---

# 14. Why Estimate Peak Traffic?

Average traffic isn't enough.

Example:

```text
Average → 100 RPS
Peak    → 1,000 RPS
```

If your architecture handles only 100 RPS:

```text
Peak traffic
   ↓
System overload
```

Design around reasonable peak assumptions.

---

# 15. Read vs Write Ratio

Suppose:

```text
90% reads
10% writes
```

Then:

```text
1000 requests/sec
```

roughly becomes:

```text
900 reads/sec
100 writes/sec
```

This affects architecture.

A read-heavy system may benefit significantly from:

```text
Caching
Read replicas
Search indexes
```

---

# 16. Storage Estimation

Suppose:

```text
1 million new orders/year
```

Average order record:

```text
2 KB
```

Then:

```text
1,000,000 × 2 KB
≈ 2 GB/year
```

Real storage needs will be higher after considering:

```text
Indexes
Metadata
Replication
Logs
Backups
Growth
```

---

# 17. Traffic Estimation

For an API:

```text
RPS × average response size
```

gives a rough bandwidth estimate.

Example:

```text
1,000 RPS
×
10 KB
=
10 MB/sec
```

This helps determine:

```text
Network capacity
Load balancer capacity
Caching
Compression
```

---

# 18. Latency

Latency means:

```text
How long a request takes
```

Example:

```text
Client
 ↓
API
 ↓
DB
```

Total latency includes:

```text
Network
Application processing
Database
Cache
External APIs
```

---

# 19. Throughput

Throughput means:

```text
How much work the system handles per unit time
```

Examples:

```text
Requests/sec
Orders/sec
Messages/sec
Transactions/sec
```

---

# 20. Latency vs Throughput

Easy distinction:

```text
Latency
→ Time for one request

Throughput
→ Amount of work processed over time
```

A system can have:

```text
High throughput
but
high latency
```

These are different metrics.

---

# 21. Availability

Availability describes:

```text
How often the system is operational
```

For example:

```text
99.9%
```

means approximately:

```text
0.1% downtime
```

over the measurement period.

---

# 22. Availability Calculation

For a rough yearly calculation:

```text
1 year ≈ 525,600 minutes
```

At:

```text
99.9%
```

allowed downtime is approximately:

```text
525.6 minutes
```

At:

```text
99.99%
```

approximately:

```text
52.56 minutes
```

The actual SLA measurement window and exclusions depend on the service agreement.

---

# 23. Reliability

Reliability is about:

```text
The system performing correctly over time.
```

Availability:

```text
Is it available?
```

Reliability:

```text
Does it keep working correctly?
```

A system can be available but return incorrect results.

---

# 24. Scalability

Scalability means:

```text
Ability to handle increasing load.
```

Two major approaches:

```text
Vertical scaling
Horizontal scaling
```

---

# 25. Vertical Scaling

Increase resources of one machine:

```text
2 CPU
8 GB RAM
```

to:

```text
8 CPU
32 GB RAM
```

Simple, but has limits.

---

# 26. Horizontal Scaling

Add more instances:

```text
App 1
App 2
App 3
App 4
```

Traffic can be distributed across them.

This is generally the preferred scaling approach for stateless application servers.

---

# 27. Stateless Application

A stateless application doesn't rely on local instance memory for important session state.

Example:

```text
Request 1 → App 1
Request 2 → App 3
Request 3 → App 2
```

Any instance can handle the request.

---

# 28. Stateful Application

A stateful application keeps important state locally.

Example:

```text
User session
   ↓
App 1 memory
```

If the next request goes to:

```text
App 2
```

the state may be unavailable.

This complicates scaling.

---

# 29. Externalizing State

Instead of:

```text
App memory
```

use:

```text
Redis
Database
```

depending on the state.

Example:

```text
App 1
App 2
App 3
   |
   ↓
Redis
```

Now all instances can access shared session/cache state.

---

# 30. Load Balancer

When multiple application instances exist:

```text
Client
  ↓
Load Balancer
 /    |    \
App1 App2 App3
```

The load balancer distributes traffic.

---

# 31. Why Use a Load Balancer?

It can provide:

```text
Traffic distribution
Health checks
Failover
TLS termination
High availability
```

depending on the product/configuration.

---

# 32. Health Checks

Suppose:

```text
App1 → healthy
App2 → unhealthy
App3 → healthy
```

A load balancer can stop sending traffic to:

```text
App2
```

if health checks indicate it should not receive traffic.

---

# 33. Reverse Proxy

A reverse proxy sits in front of backend services.

Example:

```text
Client
  ↓
Reverse Proxy
  ↓
Backend
```

It can handle:

```text
Routing
TLS
Compression
Caching
Security policies
```

depending on configuration.

---

# 34. API Gateway

An API gateway is a common entry point for multiple backend services.

```text
Client
  ↓
API Gateway
 /    |    \
User Order Product
```

Potential responsibilities:

```text
Routing
Authentication
Rate limiting
Request transformation
Logging
```

Don't put every piece of business logic in the gateway.

---

# 35. Monolith

A monolith is deployed as one application unit.

Example:

```text
Spring Boot
│
├── User
├── Product
├── Cart
└── Order
```

One deployment artifact.

---

# 36. Advantages of Monolith

```text
Simple deployment
Simple local development
Easy debugging
Simple transactions
Less network communication
Lower operational complexity
```

For a small team/system, this can be an excellent choice.

---

# 37. Microservices

Break the application into independently deployable services.

Example:

```text
User Service
Product Service
Cart Service
Order Service
Payment Service
```

Each service can have:

```text
Own code
Own deployment
Possibly own database
```

---

# 38. Microservices Trade-off

You gain:

```text
Independent scaling
Independent deployment
Team ownership
Fault isolation
```

But introduce:

```text
Network calls
Distributed transactions
Observability complexity
Deployment complexity
Data consistency problems
Operational overhead
```

Microservices aren't automatically better.

---

# 39. Service Communication

Services can communicate using:

```text
REST
gRPC
Messaging
Events
```

Example:

```text
Order Service
      ↓
Payment Service
```

or asynchronously:

```text
Order Service
      ↓
Kafka
      ↓
Notification Service
```

---

# 40. Synchronous Communication

Example:

```text
Order Service
      ↓
Payment Service
      ↓
Response
```

The caller waits for the response.

Advantages:

```text
Simple request/response model
Immediate result
```

Disadvantages:

```text
Higher coupling
Failure propagation
Latency
```

---

# 41. Asynchronous Communication

Example:

```text
Order Service
      ↓
Message Queue
      ↓
Notification Service
```

The producer doesn't need the consumer to complete immediately.

Advantages:

```text
Decoupling
Better resilience
Traffic smoothing
Independent processing
```

Trade-offs:

```text
Eventual consistency
More complex debugging
Duplicate messages
Ordering concerns
```

---

# 42. Database Choice

A system might use:

```text
Relational DB
NoSQL DB
Search engine
Cache
Object storage
```

Choose based on requirements.

Don't start with:

```text
"We need MongoDB."
```

Start with:

```text
"What data and access patterns do we have?"
```

---

# 43. Relational Database

Examples:

```text
MySQL
PostgreSQL
Oracle
```

Good for:

```text
Structured data
Transactions
Relationships
Strong consistency requirements
Complex queries
```

---

# 44. NoSQL

Examples:

```text
MongoDB
DynamoDB
Cassandra
```

Useful for particular access patterns and scale requirements.

NoSQL isn't one single database model.

---

# 45. Caching

A cache stores frequently accessed data closer to the application.

```text
App
 ↓
Cache
 ↓ cache miss
Database
```

Example:

```text
Product details
```

---

# 46. Cache Hit

```text
Request
 ↓
Redis
 ↓
Data found
 ↓
Return
```

The database isn't needed.

---

# 47. Cache Miss

```text
Request
 ↓
Redis
 ↓
Not found
 ↓
MySQL
 ↓
Store in Redis
 ↓
Return
```

This can reduce database load.

---

# 48. Cache Trade-offs

Benefits:

```text
Lower latency
Reduced DB load
Higher throughput
```

Problems:

```text
Stale data
Cache invalidation
Memory limits
Eviction
Consistency
```

---

# 49. Database Replication

Instead of one database:

```text
Primary
  |
  +--- Replica
  |
  +--- Replica
```

Writes usually go to the primary.

Reads may be distributed to replicas depending on the architecture.

---

# 50. Why Replicate?

Potential benefits:

```text
Read scaling
High availability
Disaster recovery
```

But replication introduces:

```text
Replication lag
Consistency considerations
Failover complexity
```

---

# 51. Database Sharding

Split data across multiple database nodes.

Example:

```text
Users 1–10M
   ↓
Shard 1

Users 10M–20M
   ↓
Shard 2
```

Or shard using a key:

```text
hash(userId)
```

Sharding can improve scale but adds significant complexity.

---

# 52. Partitioning vs Sharding

A useful interview distinction:

```text
Partitioning
→ Splitting data logically

Sharding
→ Distributing partitions across separate database nodes
```

Terminology can vary by database technology.

---

# 53. CAP Theorem

In a distributed system, CAP describes trade-offs involving:

```text
Consistency
Availability
Partition tolerance
```

The important interview idea:

> When a network partition occurs, a distributed system has to trade off between consistency and availability.

---

# 54. Don't Say "Choose Two" Too Literally

The popular phrase:

```text
Pick 2 of 3
```

is an oversimplification.

In practical distributed systems, partition tolerance is generally a reality you must account for.

The meaningful question becomes:

```text
During a partition,
do we favor consistency or availability?
```

---

# 55. Consistency

Consistency means clients observe data according to the system's defined consistency guarantees.

Example:

```text
Order placed
```

Should another service immediately see:

```text
Order = PAID
```

or can it temporarily see an older state?

The answer depends on the business requirement.

---

# 56. Eventual Consistency

With eventual consistency:

```text
Update
 ↓
Propagation
 ↓
Other nodes eventually see update
```

Temporary differences can exist.

Useful when:

```text
Immediate consistency isn't required
```

---

# 57. Strong Consistency

A read after a successful write can be guaranteed to see the latest value according to the system's consistency model.

Useful for:

```text
Critical financial state
Inventory constraints
Some transactional workflows
```

But it can have availability/latency trade-offs.

---

# 58. Message Queue

A queue decouples producers and consumers.

```text
Producer
   ↓
Queue
   ↓
Consumer
```

Examples:

```text
Kafka
RabbitMQ
AWS SQS
```

They have different semantics and should not be treated as interchangeable.

---

# 59. Why Use a Queue?

```text
Traffic smoothing
Asynchronous processing
Decoupling
Retry
Buffering
```

Example:

```text
Order placed
   ↓
Queue
   ↓
Email service
```

The user doesn't need to wait for email delivery.

---

# 60. Rate Limiting

Rate limiting protects services from excessive traffic.

Example:

```text
100 requests/minute/user
```

Potential implementation:

```text
API Gateway
Redis
Application
```

---

# 61. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

Example:

```text
JWT authentication
+
ROLE_ADMIN authorization
```

---

# 62. Observability

A production system needs:

```text
Logs
Metrics
Traces
```

This is often called:

```text
Three pillars of observability
```

---

# 63. Logs

Logs answer:

```text
What happened?
```

Example:

```text
Order 123 failed payment
```

---

# 64. Metrics

Metrics answer:

```text
How much?
How often?
How long?
```

Examples:

```text
RPS
Latency
Error rate
CPU
Memory
DB connections
```

---

# 65. Traces

Distributed tracing answers:

```text
Where did this request spend time?
```

Example:

```text
API Gateway
   ↓ 10ms
Order Service
   ↓ 40ms
Payment Service
   ↓ 200ms
Database
```

---

# 66. Fault Tolerance

Systems should expect failures.

Examples:

```text
Database unavailable
Redis unavailable
Service timeout
Network failure
Message duplication
Instance crash
```

Possible techniques:

```text
Retry
Timeout
Circuit breaker
Fallback
Queue
Replication
```

---

# 67. Timeout

Never allow a network call to wait forever.

Example:

```text
Order Service
    ↓
Payment Service
```

Configure an appropriate timeout.

Otherwise:

```text
Payment slow
 ↓
Order threads blocked
 ↓
Thread pool exhausted
 ↓
API becomes unavailable
```

---

# 68. Retry

Retry can help with temporary failures.

But don't retry blindly.

Bad:

```text
Retry forever
```

Better:

```text
Limited retries
+
Backoff
+
Jitter
```

---

# 69. Circuit Breaker

Concept:

```text
Service A
   ↓
Service B
```

If B keeps failing:

```text
Circuit opens
```

A stops sending normal requests temporarily.

This prevents cascading failure.

---

# 70. Backpressure

Backpressure means:

```text
Slow consumer tells producer:
"Don't send more than I can handle."
```

Useful for:

```text
Queues
Streaming
Reactive systems
High-throughput processing
```

---

# 71. Single Point of Failure

A component is a single point of failure if:

```text
Its failure can take down the system.
```

Example:

```text
One application server
```

Better:

```text
App1
App2
App3
```

with appropriate load balancing.

---

# 72. Redundancy

Redundancy means having multiple components so one failure doesn't necessarily stop the system.

Examples:

```text
Multiple app instances
DB replicas
Multiple availability zones
Multiple queue consumers
```

Redundancy usually costs more and adds operational complexity.

---

# 73. Availability Zone

Cloud systems often provide multiple isolated locations within a region.

Conceptually:

```text
Region
 ├── AZ1
 ├── AZ2
 └── AZ3
```

Deploying across multiple zones can reduce the impact of a zone-level failure.

---

# 74. Disaster Recovery

Think about:

```text
Backup
Replication
Recovery Point Objective
Recovery Time Objective
```

---

# 75. RPO

Recovery Point Objective:

```text
How much data loss is acceptable?
```

Example:

```text
RPO = 5 minutes
```

means the recovery process should aim to lose no more than about five minutes of data, depending on the exact architecture.

---

# 76. RTO

Recovery Time Objective:

```text
How quickly must the service recover?
```

Example:

```text
RTO = 30 minutes
```

means the target recovery time is 30 minutes.

---

# 77. Security in System Design

Think about:

```text
Authentication
Authorization
Encryption
Secrets
Network security
Input validation
Rate limiting
Audit logs
```

Don't treat security as something added at the end.

---

# 78. API Design

A system design interview may require API definitions.

Example:

```http
POST /orders
GET /orders/{id}
GET /users/{id}/orders
```

Think about:

```text
Request
Response
Status code
Authentication
Idempotency
Pagination
Validation
```

---

# 79. Idempotency

An operation is idempotent if repeating the same operation produces the same intended result.

This is extremely important for:

```text
Payments
Orders
Retries
Message processing
```

Example:

```text
POST /payments
Idempotency-Key: abc123
```

If the client retries:

```text
abc123
```

the server can recognize the previous operation.

---

# 80. Why Is Idempotency Important?

Imagine:

```text
Client
 ↓
Payment API
 ↓
Payment succeeds
 ↓
Response lost
```

Client retries.

Without idempotency:

```text
Payment charged twice
```

With idempotency:

```text
Same key
 ↓
Return previous result
```

---

# 81. Pagination

Don't return:

```text
10 million products
```

in one response.

Use pagination:

```http
GET /products?page=1&size=20
```

For large datasets, cursor/keyset pagination may be more efficient than deep offset pagination.

---

# 82. API Versioning

Example:

```text
/api/v1/orders
/api/v2/orders
```

Versioning can help evolve APIs without breaking existing clients.

---

# 83. Rate Limiting Example

```text
User
 ↓
API Gateway
 ↓
Redis
 ↓
Allow / Reject
```

Example:

```text
100 requests/minute
```

If exceeded:

```http
429 Too Many Requests
```

---

# 84. Search

Don't automatically use MySQL for every search requirement.

For advanced search:

```text
Application
   ↓
Search engine
```

Examples:

```text
Elasticsearch
OpenSearch
```

depending on requirements.

---

# 85. Object Storage

Large files such as:

```text
Product images
Videos
Documents
```

usually shouldn't be stored directly inside the application container.

Use object storage:

```text
Spring Boot
   ↓
Object Storage
```

Examples:

```text
Amazon S3
Azure Blob Storage
Google Cloud Storage
```

---

# 86. CDN

A CDN caches content closer to users.

```text
User
 ↓
CDN
 ↓ cache miss
Origin
```

Useful for:

```text
Images
CSS
JavaScript
Videos
Static content
```

---

# 87. Database Connection Pool

A Spring Boot backend typically uses:

```text
HikariCP
```

A connection pool avoids opening a new database connection for every request.

System design should consider:

```text
Number of app instances
×
Connections per instance
```

---

# 88. Example Connection Calculation

Suppose:

```text
10 app instances
```

and:

```text
20 DB connections/instance
```

Then approximately:

```text
10 × 20 = 200
```

potential DB connections.

The database must be able to handle this.

---

# 89. Bottleneck Thinking

Every system has a bottleneck.

Possible bottlenecks:

```text
CPU
Memory
Database
Network
Disk
External API
Thread pool
Connection pool
Queue
Cache
```

When traffic increases, ask:

```text
What fails first?
```

---

# 90. Avoid Overengineering

If the system has:

```text
100 users
```

you probably don't need:

```text
20 microservices
Kafka
Sharding
Multi-region active-active
```

A simple:

```text
Spring Boot
MySQL
Redis
```

may be better.

---

# 91. Start Simple

A good interview approach:

```text
1. Simple architecture
2. Identify bottleneck
3. Scale the bottleneck
4. Add complexity only when justified
```

This is much better than drawing a huge architecture immediately.

---

# 92. Evolution of an Architecture

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

Reads grow:

```text
Apps
 ↓
Redis
 ↓
MySQL
```

Async work grows:

```text
Apps
 ↓
Kafka
 ↓
Workers
```

Database grows:

```text
Primary
 ↓
Read replicas
```

Eventually:

```text
Services
Cache
Queues
Replicas
Search
Object storage
```

Only add these when the requirements justify them.

---

# 93. Trade-offs

System design is mostly trade-offs.

Examples:

```text
Consistency vs availability
Latency vs throughput
Cost vs performance
Simplicity vs flexibility
Strong consistency vs scalability
Caching vs freshness
Microservices independence vs operational complexity
```

---

# 94. Cost

Don't forget cost.

A design with:

```text
20 services
10 DB replicas
3 regions
multiple queues
```

may be technically impressive but unnecessarily expensive.

Ask:

```text
What problem does this component solve?
```

If there is no clear answer:

```text
Don't add it.
```

---

# 95. Interview Framework

Use this structure in system-design interviews:

```text
1. Clarify requirements
2. Define scale
3. Define APIs
4. Draw high-level architecture
5. Design data storage
6. Discuss scaling
7. Discuss caching
8. Discuss async processing
9. Discuss reliability
10. Discuss security
11. Discuss observability
12. Identify bottlenecks
13. Explain trade-offs
```

---

# 96. Example: E-commerce High-Level Design

Start simple:

```text
             Client
                |
                ↓
          Load Balancer
                |
        +-------+-------+
        |       |       |
      App1    App2    App3
        |       |       |
        +-------+-------+
                |
              MySQL
```

Then add cache:

```text
Apps
 ↓
Redis
 ↓
MySQL
```

Then asynchronous events:

```text
Order Service
      ↓
    Kafka
   /     \
Email   Inventory
```

---

# 97. E-commerce Services

A possible microservice decomposition:

```text
User Service
Product Service
Cart Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Don't assume all of these are needed from day one.

---

# 98. E-commerce Order Flow

Possible flow:

```text
Client
  ↓
Order API
  ↓
Order Service
  ↓
Create pending order
  ↓
Payment
  ↓
Inventory reservation
  ↓
Order confirmed
  ↓
Event
  ↓
Notification
```

This immediately raises system-design questions around:

```text
Transactions
Idempotency
Retries
Consistency
Failures
```

Those are the interesting parts.

---

# 99. Failure Scenario

Payment succeeds:

```text
Payment = SUCCESS
```

but:

```text
Order service crashes
```

Now what?

Possible approaches:

```text
Event-driven workflow
Idempotency
Outbox pattern
Retry
Reconciliation
```

We'll study these later.

---

# 100. System Design Topics Coming Next

The next files will go deeper into:

```text
02 → Requirements & Estimation
03 → Scalability
04 → Load Balancing
05 → Caching
06 → Database Design
07 → SQL vs NoSQL
08 → Replication & Sharding
09 → Messaging & Kafka
10 → Microservices Architecture
11 → API Gateway & Service Discovery
12 → Consistency & CAP
13 → Distributed Systems
14 → Security
15 → Observability
16 → HLD vs LLD
17+ → Real System Design Problems
```

---

# 101. Final Revision

Remember this mental model:

```text
REQUIREMENTS
     ↓
    SCALE
     ↓
ARCHITECTURE
     ↓
 DATA + APIs
     ↓
SCALABILITY
     ↓
RELIABILITY
     ↓
 SECURITY
     ↓
OBSERVABILITY
     ↓
TRADE-OFFS
```

Don't start with technology.

Start with the problem.

---

# 102. One-Minute Interview Answer

### "How do you approach a system design problem?"

> "I first clarify the functional and non-functional requirements and estimate the expected scale. Then I define the main APIs and propose a simple high-level architecture. After that I look at data storage, caching, scaling, asynchronous processing and failure handling. Finally, I discuss security, observability, bottlenecks and the trade-offs behind the design. I try not to overengineer until the requirements justify the additional complexity."

---

# 103. Key Takeaway

> **System design is not about drawing the most complicated architecture. It's about understanding requirements, identifying constraints, choosing appropriate components, and explaining why your design makes sense.**

For a Java backend developer, you should be comfortable moving from:

```text
Spring Boot
    ↓
MySQL
```

to:

```text
Load Balancer
    ↓
Multiple Spring Boot instances
    ↓
Redis
    ↓
MySQL / replicas
    ↓
Kafka
    ↓
Other services
```

only when the system's requirements justify each step.

**File 01 complete.**
