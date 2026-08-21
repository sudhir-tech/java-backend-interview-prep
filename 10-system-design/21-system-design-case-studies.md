# System Design — File 21: Interview Case Studies

This is the final system-design theory/practice file.

The goal is not to memorize one architecture.

The goal is to learn a repeatable interview process:

```text
Requirements
    |
    v
Estimation
    |
    v
APIs
    |
    v
Data Model
    |
    v
High-Level Design
    |
    v
Scaling
    |
    v
Reliability
    |
    v
Trade-offs
```

---

# 1. System Design Interview Framework

When an interviewer gives you a problem, don't immediately start drawing microservices.

Use this order.

## Step 1 — Clarify Requirements

Ask:

```text
Who are the users?
What are the main features?
What is the expected scale?
What latency is required?
What consistency is required?
What data must be durable?
```

Separate:

```text
Functional requirements
Non-functional requirements
```

---

# 2. Functional Requirements

Functional requirements describe what the system does.

Example for URL shortener:

```text
Create short URL
Redirect short URL
Optional expiration
View basic statistics
```

Don't design features the interviewer didn't ask for.

---

# 3. Non-Functional Requirements

Examples:

```text
High availability
Low latency
Scalability
Durability
Security
Observability
```

Ask which ones are most important.

---

# 4. Estimate Scale

You don't need perfect numbers.

Example:

```text
100M users
10M daily active users
100M requests/day
```

Then estimate:

```text
Requests/sec
Storage
Bandwidth
```

A rough estimate is enough to guide architecture.

---

# 5. Back-of-the-Envelope Calculation

Example:

```text
100M requests/day
```

Approximately:

```text
100,000,000 / 86,400
≈ 1,157 requests/sec
```

Peak traffic may be several times higher.

If peak = 5x:

```text
≈ 5,800 requests/sec
```

The goal is architectural direction, not mathematical perfection.

---

# 6. API Design

Define important APIs.

Example:

```http
POST /urls
GET /{shortCode}
```

For a backend interview, mention:

```text
Request
Response
Authentication
Idempotency where required
Error handling
```

---

# 7. Data Model

Identify the core entities.

Example URL shortener:

```text
Url
----
id
shortCode
originalUrl
createdAt
expiresAt
```

Don't over-design the schema before understanding the traffic pattern.

---

# 8. High-Level Architecture

Typical building blocks:

```text
Client
  |
  v
Load Balancer
  |
  v
Application Servers
  |
  +--> Cache
  |
  +--> Database
  |
  +--> Message Broker
  |
  +--> Object Storage
```

Add components only when they solve a real problem.

---

# 9. Case Study 1 — URL Shortener

## Requirements

Functional:

```text
Create short URL
Redirect short URL
Optional expiration
Basic analytics
```

Non-functional:

```text
High availability
Low redirect latency
Large scale
```

---

## Architecture

```text
Client
  |
  v
Load Balancer
  |
  v
URL Service
  |
  +--> Redis
  |
  +--> MySQL
  |
  +--> Kafka
```

Flow:

```text
Create URL
   |
   v
Generate short code
   |
   v
MySQL
   |
   v
Return short URL
```

Redirect:

```text
Short code
   |
   v
Redis
   |
   +--> Cache hit -> URL
   |
   +--> Miss -> MySQL -> Redis
```

Analytics can be sent asynchronously:

```text
Redirect
   |
   v
Kafka
   |
   v
Analytics Consumer
```

---

## Short Code Generation

Options:

```text
Random string
Hash
Database-generated ID + Base62
```

Base62 uses:

```text
0-9
a-z
A-Z
```

A numeric ID can be converted into a compact Base62 string.

---

## Collision Handling

If generating random codes:

```text
Generate
   |
   v
Check uniqueness
   |
   +--> Collision -> retry
```

A database unique constraint should protect correctness.

---

## Interview Answer

> "I'd use MySQL as the source of truth and Redis for hot redirect lookups. The create API generates a unique short code, persists the mapping, and returns the shortened URL. Redirects check Redis first and fall back to MySQL. Analytics should be asynchronous through Kafka so tracking doesn't increase redirect latency."

---

# 10. Case Study 2 — Rate Limiter

## Requirements

Limit requests by:

```text
User
IP
API key
Endpoint
```

Example:

```text
100 requests/minute
```

---

## Architecture

```text
Client
  |
  v
API Gateway
  |
  v
Rate Limiter
  |
  v
Redis
```

Redis is useful because rate limiting requires:

```text
Low latency
Atomic counters
TTL
Shared state
```

---

## Algorithms

Common algorithms:

```text
Fixed Window
Sliding Window
Token Bucket
Leaky Bucket
```

---

## Token Bucket

Concept:

```text
Bucket
Capacity = 100
Refill = 10/sec
```

Each request consumes a token.

If no tokens remain:

```text
429 Too Many Requests
```

---

## Interview Answer

> "I'd place rate limiting at the gateway and use Redis for shared counters or token-bucket state. The exact algorithm depends on the requirement, but token bucket is a good general choice because it allows controlled bursts while maintaining an average rate."

---

# 11. Case Study 3 — Notification System

## Requirements

Send:

```text
Email
SMS
Push
In-app notifications
```

---

## Architecture

```text
Business Service
      |
      v
Kafka
      |
      v
Notification Service
   /    |       v     v      v
Email  SMS    Push
```

In-app notifications:

```text
Notification DB
      |
      v
WebSocket / SSE
      |
      v
Client
```

---

## Why Asynchronous?

If checkout triggers:

```text
Email
SMS
Push
```

don't make checkout wait for all three.

Instead:

```text
Checkout
   |
   v
Event
   |
   v
Kafka
```

The notification service processes them independently.

---

## Interview Answer

> "I'd make notifications asynchronous. Business services publish events to Kafka, and separate consumers handle email, SMS, push and in-app delivery. I'd persist important notifications and use retries with a dead-letter queue for failures."

---

# 12. Case Study 4 — Chat System

## Requirements

```text
1-to-1 chat
Group chat
Message history
Online presence
Read receipts
```

---

## Architecture

```text
Client
  |
  v
Load Balancer
  |
  +--> WebSocket Server
  +--> WebSocket Server
  +--> WebSocket Server
            |
            v
       Chat Service
        /               v          v
   Database     Kafka/Redis
```

---

## Message Flow

```text
User A
  |
  v
WebSocket
  |
  v
Chat Service
  |
  +--> Persist message
  |
  +--> Publish event
  |
  v
User B WebSocket
```

---

## Offline User

If B is offline:

```text
Message
  |
  v
Database
```

When B reconnects:

```text
Fetch missed messages
```

The WebSocket is not the source of truth.

---

## Ordering

Use:

```text
Conversation ID
Sequence number
Message ID
```

Example:

```text
seq 101
seq 102
seq 103
```

---

## Interview Answer

> "I'd use WebSockets for bidirectional low-latency communication, but persist messages in durable storage. A shared event layer lets users connected to different WebSocket servers communicate. Sequence numbers can provide conversation-level ordering, and offline users can fetch missed messages after reconnecting."

---

# 13. Case Study 5 — E-Commerce Backend

This is especially relevant for a Java backend interview.

## Requirements

```text
User registration/login
Product catalog
Search
Cart
Order placement
Payment
Inventory
Notifications
Admin operations
```

---

## Architecture

```text
                    Client
                      |
                      v
                Load Balancer
                      |
                      v
                API Gateway
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
    User           Product         Order
   Service         Service        Service
       |              |              |
       v              v              v
    MySQL          MySQL          MySQL
                      |
                      v
                Elasticsearch

Order Service
      |
      +--> Inventory
      +--> Payment
      +--> Kafka
      +--> Notification
```

---

## Cart

For high-frequency cart operations:

```text
Client
  |
  v
Cart Service
  |
  v
Redis
```

Persistent cart data can also be stored in a database depending on requirements.

---

## Product Search

Use:

```text
MySQL -> source of truth
Kafka -> product events
Elasticsearch -> search index
```

---

## Order Workflow

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
Confirm Order
     |
     v
Create Shipment
```

This is a good candidate for Saga-style coordination.

---

## Payment Failure

```text
Payment fails
     |
     v
Release inventory
     |
     v
Cancel order
```

---

## Interview Answer

> "I'd separate high-volume concerns such as product search, cart access and order processing. MySQL would hold transactional business data, Redis would handle hot cart/cache data, Elasticsearch would handle product search, and Kafka would decouple asynchronous workflows. Order, inventory and payment would use explicit state transitions and compensation for partial failures."

---

# 14. Case Study 6 — Food Delivery System

## Requirements

```text
Restaurant discovery
Menu
Order
Payment
Driver assignment
Live tracking
Notifications
```

---

## Architecture

```text
Client
  |
  v
API Gateway
  |
  +--> Restaurant Service
  +--> Order Service
  +--> Payment Service
  +--> Delivery Service
  +--> Notification Service
  +--> Location Service
```

Real-time tracking:

```text
Driver
  |
  v
Location Service
  |
  v
Event Stream
  |
  v
WebSocket
  |
  v
Customer
```

---

## Driver Assignment

Possible approach:

```text
New delivery request
        |
        v
Find nearby drivers
        |
        v
Filter available drivers
        |
        v
Rank by distance
        |
        v
Offer job
```

For geographic search, specialized geo indexes or databases may be used.

---

# 15. Case Study 7 — Ride Booking

## Requirements

```text
Request ride
Find nearby drivers
Assign driver
Track ride
Calculate fare
Payment
Trip history
```

---

## Critical Challenge

Finding nearby available drivers quickly.

Architecture:

```text
Passenger
    |
    v
Ride Service
    |
    v
Geo/Location Service
    |
    v
Available Drivers
```

Driver location updates:

```text
Driver
  |
  v
WebSocket / Location API
  |
  v
Location Store
```

---

## Race Condition

Two passengers may attempt to get the same driver.

Use:

```text
Atomic reservation
Optimistic locking
Distributed coordination
```

The driver must transition:

```text
AVAILABLE
    |
    v
RESERVED
    |
    v
ON_TRIP
```

atomically.

---

# 16. Case Study 8 — Instagram-Like Feed

## Requirements

```text
Create post
Follow users
View feed
Like/comment
Media upload
```

---

## Media

```text
Client
  |
  v
Pre-signed upload URL
  |
  v
Object Storage
  |
  v
CDN
```

---

## Feed

Two major strategies:

### Fan-out on write

```text
User posts
   |
   v
Push post to followers' feeds
```

Fast reads, expensive writes.

### Fan-out on read

```text
User opens feed
   |
   v
Read followed users
   |
   v
Build feed
```

Less write work, more read work.

---

## Hybrid Approach

Normal users:

```text
Fan-out on write
```

Celebrity/high-follower users:

```text
Fan-out on read
```

This avoids massive write amplification.

---

# 17. Case Study 9 — YouTube-Like Video Platform

## Upload

```text
Client
  |
  v
Pre-signed URL
  |
  v
Object Storage
```

## Processing

```text
Object Storage
      |
      v
Queue
      |
      v
Transcoding Workers
      |
      +--> 360p
      +--> 720p
      +--> 1080p
```

## Delivery

```text
Object Storage
      |
      v
CDN
      |
      v
Users
```

The application server should not stream every video byte.

---

# 18. Case Study 10 — Payment System

Payments require strong correctness.

## Requirements

```text
Create payment
Prevent duplicate charge
Track status
Handle retries
Reconcile with provider
```

---

## Payment State Machine

```text
INITIATED
    |
    v
PROCESSING
   /   v   v
SUCCESS  FAILED
```

Possible additional state:

```text
UNKNOWN / PENDING
```

A timeout does not necessarily mean failure.

---

## Idempotency

Client sends:

```http
Idempotency-Key: abc123
```

Server stores:

```text
abc123 -> payment result
```

Retrying the same key should not create another payment.

---

## Provider Timeout

Do not blindly retry.

```text
Timeout
   |
   v
Check provider status
   |
   +--> Success -> update payment
   |
   +--> Failed -> safely retry if allowed
   |
   +--> Unknown -> reconcile
```

---

# 19. Case Study 11 — Distributed Rate Limiter

A more detailed architecture:

```text
Client
  |
  v
Load Balancer
  |
  v
API Gateway
  |
  v
Redis
```

Key:

```text
rate:{userId}:{endpoint}
```

Use atomic operations where needed.

For multiple gateway instances:

```text
Gateway 1 ---Gateway 2 ----+--> Redis
Gateway 3 ---/
```

This prevents each instance from maintaining an independent limit.

---

# 20. Case Study 12 — Notification + Real-Time Combination

```text
Business Event
      |
      v
Kafka
      |
      v
Notification Service
      |
      +--> DB
      |
      +--> Email
      +--> Push
      +--> SMS
      |
      +--> WebSocket/SSE
```

Important notifications are persisted.

Real-time delivery is treated as a fast delivery channel.

---

# 21. Case Study 13 — File Storage System

## Requirements

```text
Upload
Download
Share
Delete
Large files
Private files
```

Architecture:

```text
Client
  |
  v
Backend
  |
  +--> Auth
  +--> Metadata DB
  |
  +--> Pre-signed URL
           |
           v
      Object Storage
           |
           v
          CDN
```

Large files:

```text
Multipart upload
Resumable upload
```

---

# 22. Case Study 14 — Search System

```text
MySQL
  |
  v
Product Event
  |
  v
Kafka
  |
  v
Search Indexer
  |
  v
Elasticsearch
```

Search API:

```text
Client
  |
  v
Search Service
  |
  v
Elasticsearch
```

Use:

```text
Filters
Ranking
Autocomplete
Aggregations
Pagination
```

---

# 23. Case Study 15 — Notification Feed

For an in-app notification feed:

```text
Event
 |
 v
Notification Service
 |
 +--> MySQL
 |
 +--> Redis cache
 |
 +--> WebSocket/SSE
```

Database provides durability.

Redis can speed up hot notification reads.

WebSocket/SSE provides low-latency delivery.

---

# 24. How to Approach Any New System Design Problem

Use this checklist:

```text
1. Clarify requirements
2. Identify scale
3. Estimate traffic
4. Define APIs
5. Define data model
6. Draw high-level architecture
7. Explain request flow
8. Identify bottlenecks
9. Add caching
10. Add asynchronous processing
11. Add database scaling
12. Add reliability
13. Add security
14. Add observability
15. Explain trade-offs
```

---

# 25. Database Selection

Ask:

```text
Relational or NoSQL?
```

Use relational databases when you need:

```text
Transactions
Strong relationships
Complex queries
Strong consistency
```

NoSQL can be useful when you need:

```text
Massive scale
Flexible schema
Specific access patterns
High write/read throughput
```

Don't choose NoSQL just because the system is large.

---

# 26. Caching Decision

Ask:

```text
Is the data read frequently?
Is it expensive to calculate?
Can stale data be tolerated?
```

If yes:

```text
Redis
```

Possible patterns:

```text
Cache-aside
Write-through
Write-behind
```

---

# 27. Async Processing Decision

Use queues/events when:

```text
Task is slow
Task doesn't need immediate completion
Work can be retried
Traffic needs smoothing
Services should be decoupled
```

Examples:

```text
Email
Video transcoding
Search indexing
Analytics
Notifications
```

---

# 28. Scaling Decision

### Vertical scaling

```text
Bigger machine
```

Simple but limited.

### Horizontal scaling

```text
More instances
```

Better for large-scale stateless services.

---

# 29. Stateless Application Servers

Prefer:

```text
Instance 1
Instance 2
Instance 3
```

without important local business state.

Store shared state in:

```text
Database
Redis
Object Storage
Message Broker
```

This makes horizontal scaling easier.

---

# 30. Database Scaling

Common approaches:

```text
Indexes
Read replicas
Partitioning
Sharding
Caching
Archiving
Connection pooling
```

Start with the simplest approach that solves the problem.

---

# 31. Read Replicas

```text
Primary
  |
  +--> Replica 1
  +--> Replica 2
```

Writes:

```text
Primary
```

Reads:

```text
Replicas
```

But replication lag must be considered.

---

# 32. Sharding

Split data across database partitions:

```text
Shard 1 -> Users 1-1M
Shard 2 -> Users 1M-2M
Shard 3 -> Users 2M-3M
```

A good shard key distributes load evenly.

Bad shard keys can create hotspots.

---

# 33. Reliability

For every major component ask:

```text
What if it crashes?
What if the network fails?
What if the database is unavailable?
What if a message is duplicated?
What if a request times out?
```

This is often where strong system-design answers stand out.

---

# 34. Availability

Improve availability through:

```text
Multiple instances
Load balancing
Replication
Failover
Health checks
Timeouts
Retries
Circuit breakers
```

But retries must be bounded.

---

# 35. Circuit Breaker

If a dependency repeatedly fails:

```text
Service A
   |
   v
Service B X
```

Instead of continuously calling B:

```text
CLOSED
  |
  v
OPEN
  |
  v
Fail fast
```

After a recovery period:

```text
HALF-OPEN
```

Test a limited request.

---

# 36. Observability

Monitor:

```text
Latency
Traffic
Errors
Saturation
Logs
Metrics
Traces
```

Use:

```text
Correlation IDs
Distributed tracing
Dashboards
Alerts
```

---

# 37. Security

Every design should consider:

```text
Authentication
Authorization
TLS
Input validation
Rate limiting
Secrets management
Encryption
Audit logging
```

Don't add security only at the end.

---

# 38. Trade-Offs

A strong system-design answer explains trade-offs.

Example:

```text
Redis improves latency
but introduces another dependency.
```

```text
Kafka improves decoupling
but introduces eventual consistency.
```

```text
Read replicas improve read scale
but can return stale data.
```

```text
Sharding improves scale
but increases operational complexity.
```

---

# 39. Common Interview Mistakes

Avoid:

```text
Jumping directly to microservices
Adding Kafka everywhere
Adding Redis everywhere
Using NoSQL without justification
Ignoring failure scenarios
Ignoring consistency
Ignoring security
Ignoring monitoring
Over-designing the database
```

Most importantly:

> Explain why each component exists.

---

# 40. Five-Minute System Design Structure

A practical interview flow:

```text
Minute 1:
Requirements + scale

Minute 2:
APIs + data model

Minute 3:
High-level architecture

Minute 4:
Scaling + reliability

Minute 5:
Trade-offs + bottlenecks
```

Then go deeper into whatever the interviewer asks.

---

# 41. What Interviewers Usually Look For

They are usually evaluating:

```text
Requirement clarification
Architecture thinking
Scalability
Data modeling
API design
Caching
Distributed systems
Reliability
Consistency
Trade-offs
Communication
```

They are not simply checking whether you can draw 20 boxes.

---

# 42. How to Explain a Diagram

Don't say:

> "Here is Kafka, here is Redis, here is MySQL."

Instead say:

> "The request first reaches the load balancer, which distributes traffic across stateless application instances. Frequently accessed data is served from Redis. Transactional data is stored in MySQL. Slow or asynchronous work is published to Kafka so the request path remains fast."

Explain the **flow**, not just the components.

---

# 43. Final System Design Template

Use this template during interviews:

```text
1. Requirements
   - Functional
   - Non-functional

2. Scale
   - Users
   - Requests/sec
   - Storage
   - Peak traffic

3. APIs
   - Endpoints
   - Request/response
   - Auth/idempotency

4. Data Model
   - Entities
   - Relationships
   - Indexes

5. Architecture
   - Load balancer
   - Services
   - Cache
   - Database
   - Queue
   - Object storage

6. Request Flow
   - Read path
   - Write path

7. Scaling
   - Horizontal scaling
   - Cache
   - Read replicas
   - Sharding

8. Reliability
   - Retry
   - Timeout
   - Circuit breaker
   - Failover

9. Consistency
   - Strong/eventual
   - Idempotency
   - Ordering

10. Security
    - Auth
    - Authorization
    - Encryption
    - Rate limits

11. Observability
    - Logs
    - Metrics
    - Traces

12. Trade-offs
```

---

# 44. Final Case Study Practice List

After completing the theory, practice these without looking at the solution:

```text
□ URL Shortener
□ Rate Limiter
□ Notification System
□ Chat System
□ E-commerce
□ Food Delivery
□ Ride Booking
□ Instagram Feed
□ Video Platform
□ Payment System
□ File Storage
□ Search System
```

For each one, practice explaining:

```text
Requirements
Scale
APIs
Data model
Architecture
Request flow
Bottlenecks
Caching
Async processing
Database scaling
Failure handling
Consistency
Security
Observability
Trade-offs
```

---

# 45. Final Interview Rule

When you don't know something, don't panic.

Use:

```text
"I'd clarify the requirement first."
```

Then reason from:

```text
Traffic
Data
Latency
Consistency
Failure
Cost
```

Good system design is not about knowing every technology.

It is about making justified engineering decisions.

---

# 46. One-Minute Master Answer

If the interviewer asks:

### "How do you approach system design?"

> "I first clarify the functional and non-functional requirements and estimate the scale. Then I define the main APIs and data model before drawing the high-level architecture. I usually start with stateless application servers behind a load balancer, a suitable database, and caching where it provides value. For slow or independent work I'd use asynchronous messaging. Then I'd discuss scaling, consistency, failure handling, security and observability. Finally, I'd explain the main bottlenecks and trade-offs rather than adding components without a reason."

---

# 47. System Design Completion Checklist

```text
□ Requirements gathering
□ Capacity estimation
□ API design
□ Data modeling
□ High-level architecture
□ Caching
□ Database scaling
□ Read replicas
□ Sharding
□ Message queues
□ Event-driven architecture
□ Distributed transactions
□ Saga
□ Outbox
□ Idempotency
□ Consistency models
□ Object storage
□ CDN
□ Search
□ Elasticsearch
□ Real-time systems
□ WebSockets
□ SSE
□ Rate limiting
□ Notifications
□ Chat
□ Payments
□ E-commerce
□ Food delivery
□ Ride booking
□ Social feeds
□ Video platforms
□ Reliability
□ Circuit breakers
□ Retries
□ Timeouts
□ Security
□ Observability
□ Case-study practice
```

---

# 48. Final Takeaway

> **System design interviews are not about producing the biggest architecture. They are about understanding requirements, identifying bottlenecks, choosing appropriate building blocks, handling failures and explaining trade-offs clearly.**

Your core mental model should be:

```text
Requirements
     |
     v
Scale
     |
     v
APIs + Data
     |
     v
Simple Architecture
     |
     v
Identify Bottlenecks
     |
     v
Scale Only Where Needed
     |
     v
Reliability + Consistency
     |
     v
Security + Observability
     |
     v
Trade-offs
```

**File 21 complete — System Design section complete.**
