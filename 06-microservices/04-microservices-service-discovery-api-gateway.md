# Microservices — Service Discovery & API Gateway

This file covers two core microservices infrastructure concepts:

```text
Service Discovery
API Gateway
```

These are extremely common in Spring Boot microservices interviews.

---

# 1. The Problem

Imagine we have:

```text
Order Service
Payment Service
Product Service
User Service
```

Each service may have multiple instances:

```text
Payment-1
Payment-2
Payment-3
```

The Order Service needs to know:

```text
Where is Payment Service?
Which instance should I call?
```

Hardcoding an IP address is fragile.

---

# 2. Why Hardcoded URLs Are a Problem

Bad:

```text
http://10.10.20.15:8080/payments
```

Why?

Because instances can:

```text
Restart
Scale up
Scale down
Move hosts
Change IP addresses
Be replaced
```

Modern platforms are dynamic.

---

# 3. Service Discovery

Service discovery allows services to locate other service instances dynamically.

Conceptually:

```text
                Service Registry
               /       |       \
              /        |        \
        Order       Payment    Product
```

The registry knows which instances are available.

---

# 4. Service Registration

When a service starts:

```text
Payment Service
      |
      | register
      ↓
Service Registry
```

It can register information such as:

```text
Service name
Host
Port
Health status
Metadata
```

Example:

```text
payment-service
  → 10.0.0.10:8080
  → 10.0.0.11:8080
  → 10.0.0.12:8080
```

---

# 5. Service Discovery Flow

A simplified flow:

```text
1. Payment Service starts
2. Registers itself
3. Order Service needs Payment
4. Order Service discovers available instances
5. One instance is selected
6. Request is sent
```

---

# 6. Service Registry

A service registry maintains information about available service instances.

Examples:

```text
Eureka
Consul
Kubernetes service discovery
```

In modern cloud-native environments, platform-native discovery such as Kubernetes DNS is often preferred over introducing a separate registry.

---

# 7. Eureka

Netflix Eureka is a service discovery system that became popular in Spring Cloud architectures.

Conceptually:

```text
Eureka Server
     |
 +---+---+
 |       |
Order  Payment
```

Services register with Eureka.

---

# 8. Eureka Client

A service can act as an Eureka client.

Example concept:

```text
Order Service
   ↓
Eureka
   ↓
Find payment-service
```

The client can then obtain available instances.

---

# 9. Eureka Server

The registry acts as the central service directory.

Conceptually:

```text
Eureka Server
│
├── user-service
│   ├── instance-1
│   └── instance-2
│
├── product-service
│   ├── instance-1
│   └── instance-2
│
└── payment-service
    ├── instance-1
    └── instance-2
```

---

# 10. Heartbeats

Service discovery systems need to know whether instances are still alive.

An instance periodically sends a heartbeat.

Conceptually:

```text
Payment Service
      |
      | heartbeat
      ↓
Registry
```

If heartbeats stop, the registry can eventually remove or mark the instance unhealthy according to its configuration.

---

# 11. Health vs Registration

Registration alone does not guarantee the service is healthy.

You may need:

```text
Registration
+
Health checks
+
Readiness
```

This prevents traffic from being routed to an instance that is registered but not actually ready.

---

# 12. Client-Side Discovery

Conceptually:

```text
Order Service
      |
      ↓
Service Registry
      |
      ↓
Payment instances
      |
      ↓
Order Service chooses instance
```

The caller participates in instance selection.

---

# 13. Server-Side Discovery

Conceptually:

```text
Order Service
      |
      ↓
Load Balancer
      |
   +--+--+
   |     |
Payment-1 Payment-2
```

The load balancer handles instance selection.

This is common in Kubernetes and cloud environments.

---

# 14. Client-Side vs Server-Side

| Client-Side | Server-Side |
|---|---|
| Client discovers instances | Load balancer discovers/routes |
| Client selects instance | Infrastructure selects |
| More logic in client | More logic in platform |
| Common in some Spring Cloud setups | Common in cloud/Kubernetes setups |

---

# 15. Kubernetes Service Discovery

In Kubernetes, a Service provides a stable networking abstraction over changing Pods.

Example:

```text
payment-service
      |
      +---- Pod 1
      +---- Pod 2
      +---- Pod 3
```

The client can use:

```text
http://payment-service
```

instead of individual Pod IP addresses.

---

# 16. Why Kubernetes Discovery Is Powerful

Pods can be replaced:

```text
Pod A dies
↓
Pod D starts
```

The Service continues providing a stable endpoint.

Applications don't need to track individual Pod IPs.

---

# 17. DNS-Based Discovery

Kubernetes commonly provides DNS names for Services.

Conceptually:

```text
payment-service
```

resolves to the appropriate Service endpoint.

This removes the need for hardcoded addresses.

---

# 18. Load Balancing

Suppose:

```text
payment-service
├── Pod A
├── Pod B
└── Pod C
```

Traffic can be distributed across available instances.

This improves:

```text
Capacity
Availability
Horizontal scaling
```

---

# 19. Health Checks and Load Balancing

Suppose:

```text
Pod A → healthy
Pod B → unhealthy
Pod C → healthy
```

The infrastructure should avoid sending normal traffic to:

```text
Pod B
```

This is why readiness checks matter.

---

# 20. Readiness vs Liveness

### Readiness

Answers:

> "Can this instance receive traffic?"

### Liveness

Answers:

> "Should this instance be restarted?"

Example:

```text
Application process running
Database unavailable
```

It might be:

```text
Liveness = healthy
Readiness = unhealthy
```

depending on the configured health checks.

---

# 21. What Is an API Gateway?

An API Gateway is a centralized entry point for external clients.

Example:

```text
Web / Mobile
      |
      ↓
 API Gateway
      |
 +----+----+----+
 |    |    |    |
User Product Order
```

---

# 22. Why Use an API Gateway?

The gateway can centralize cross-cutting concerns:

```text
Routing
Authentication
Authorization
Rate limiting
Request filtering
TLS termination
Observability
API composition
```

---

# 23. Without an API Gateway

Clients might need to know:

```text
user.example.com
product.example.com
order.example.com
payment.example.com
```

This exposes internal architecture.

---

# 24. With an API Gateway

The client sees:

```text
api.example.com
```

The gateway routes internally:

```text
/api/users
→ User Service

/api/products
→ Product Service

/api/orders
→ Order Service
```

This provides a cleaner external boundary.

---

# 25. Spring Cloud Gateway

For Spring-based architectures, Spring Cloud Gateway is a common API Gateway option.

Conceptually:

```text
Client
 ↓
Spring Cloud Gateway
 ↓
Microservices
```

It can route requests based on:

```text
Path
Host
Headers
Query parameters
Other predicates
```

---

# 26. Gateway Route Example

Conceptually:

```text
/api/products/**
        ↓
product-service
```

and:

```text
/api/orders/**
        ↓
order-service
```

The gateway determines the destination.

---

# 27. Gateway Authentication

A gateway can validate authentication before forwarding requests.

Example:

```text
Client
 ↓
JWT
 ↓
Gateway
 ↓
Validate token
 ↓
Route request
```

But authentication architecture should be designed carefully; services may also need to enforce authorization rather than blindly trusting the gateway.

---

# 28. Gateway vs Authentication Service

They are not the same thing.

### Authentication service

Responsible for things such as:

```text
Login
Token issuance
Identity
Credential validation
```

### API Gateway

Responsible for things such as:

```text
Routing
Traffic control
Gateway-level security
Rate limiting
```

They can work together.

---

# 29. Gateway Authorization

Authentication asks:

```text
Who are you?
```

Authorization asks:

```text
What are you allowed to do?
```

Example:

```text
JWT
 ↓
user = 123
roles = CUSTOMER
```

Request:

```text
DELETE /admin/products/10
```

Authorization should reject it if the user lacks the required permission.

---

# 30. Should Only the Gateway Handle Authorization?

Not necessarily.

A common defense-in-depth approach is:

```text
Gateway
 ↓
Coarse access control
 ↓
Service
 ↓
Business authorization
```

The service should not blindly trust client-controlled information.

---

# 31. Rate Limiting at Gateway

Suppose:

```text
Client → 10,000 requests/sec
```

The gateway can limit traffic:

```text
100 requests/sec
```

per:

```text
User
API key
IP
Tenant
```

depending on requirements.

---

# 32. Why Rate Limiting Matters

Without protection:

```text
Client
 ↓
Gateway
 ↓
Services
 ↓
Database
```

can overload the entire backend.

Rate limiting protects downstream resources.

---

# 33. Gateway and Authentication Example

```text
                    Client
                      |
                      ↓
                API Gateway
                      |
               Validate JWT
                      |
             +--------+--------+
             |        |        |
             ↓        ↓        ↓
           User    Product    Order
```

The gateway handles common concerns.

Services still enforce domain-specific authorization.

---

# 34. Request Routing

Example:

```text
GET /api/products/10
```

Gateway:

```text
/api/products/**
        ↓
Product Service
```

Another:

```text
POST /api/orders
```

Gateway:

```text
/api/orders/**
        ↓
Order Service
```

---

# 35. Path Rewriting

A gateway can transform:

```text
/api/products/10
```

into:

```text
/products/10
```

before forwarding to Product Service.

The internal service doesn't necessarily need to know the external URL structure.

---

# 36. Header Forwarding

The gateway may propagate headers such as:

```text
Authorization
X-Correlation-ID
Traceparent
```

This supports:

```text
Authentication
Tracing
Logging
```

Don't blindly forward every client header.

---

# 37. Correlation ID

Example:

```text
Client
 X-Correlation-ID: abc123
      ↓
Gateway
      ↓
Order Service
      ↓
Payment Service
```

All logs can use:

```text
abc123
```

to connect the request across services.

---

# 38. Distributed Tracing

A correlation ID is useful, but distributed tracing provides richer information.

Example:

```text
Trace
├── Gateway span
├── Order span
├── Payment span
└── Database span
```

Tools/ecosystem:

```text
OpenTelemetry
Jaeger
Zipkin
```

---

# 39. Gateway Aggregation

Suppose a mobile screen needs:

```text
User
Orders
Recommendations
```

Without aggregation:

```text
Mobile
 ↓
User
 ↓
Orders
 ↓
Recommendations
```

or three client calls.

A gateway/BFF can sometimes aggregate responses:

```text
Mobile
  ↓
Gateway/BFF
  ├── User
  ├── Orders
  └── Recommendations
```

This is useful when designed carefully.

---

# 40. Backend for Frontend (BFF)

A BFF is a backend tailored to a particular client experience.

Example:

```text
Web BFF
Mobile BFF
```

They can call internal services and return client-specific responses.

---

# 41. API Gateway vs BFF

Gateway:

```text
Common infrastructure boundary
```

BFF:

```text
Client-specific backend aggregation/API
```

They can coexist.

Example:

```text
Mobile
 ↓
Mobile BFF
 ↓
API Gateway
 ↓
Services
```

The exact architecture varies.

---

# 42. Gateway Anti-Pattern: Business Logic

Avoid turning the gateway into:

```text
God Gateway
```

with:

```text
Order rules
Payment rules
Inventory rules
```

Business logic belongs in domain services.

The gateway should primarily handle:

```text
Traffic
Routing
Cross-cutting concerns
```

---

# 43. Gateway Anti-Pattern: Too Much Aggregation

If every request requires:

```text
Gateway
 ↓
10 services
```

the gateway can become:

```text
Bottleneck
Single point of complexity
Latency amplifier
```

Aggregation should be intentional.

---

# 44. Gateway Failure

If the gateway is down:

```text
External traffic
      ↓
     X
```

Therefore gateways should generally be deployed with:

```text
Multiple instances
Load balancing
Health checks
Autoscaling where appropriate
```

Avoid a single gateway instance.

---

# 45. Gateway Scaling

Example:

```text
Load Balancer
      |
 +----+----+
 |         |
Gateway-1 Gateway-2
 |         |
 +----+----+
      |
   Services
```

The gateway itself should normally be horizontally scalable.

---

# 46. Gateway Rate Limiting Strategies

Possible approaches:

```text
Fixed window
Sliding window
Token bucket
Leaky bucket
```

You don't need to implement every algorithm for an interview, but understand the purpose.

---

# 47. Token Bucket

Conceptually:

```text
Bucket
[● ● ● ● ●]
```

Tokens are added over time.

A request consumes a token.

If no token is available:

```text
Request rejected/throttled
```

This can allow controlled bursts.

---

# 48. Gateway Circuit Breaker

A gateway may protect calls to unhealthy downstream services.

Example:

```text
Gateway
 ↓
Payment
```

If Payment repeatedly fails:

```text
Circuit opens
```

The gateway can fail fast or return an appropriate fallback.

However, resilience policies can also live in the calling service or service mesh depending on architecture.

---

# 49. Service Discovery + Gateway

Example:

```text
Client
 ↓
Gateway
 ↓
Service discovery
 ↓
Order instances
```

The gateway doesn't necessarily need hardcoded instance IPs.

In Kubernetes, service DNS can provide the stable destination.

---

# 50. Gateway vs Load Balancer

They can overlap but have different responsibilities.

### Load balancer

Primarily:

```text
Distribute traffic
```

### API Gateway

Can provide:

```text
Routing
Authentication
Rate limiting
Transformation
Aggregation
Observability
```

A gateway can sit behind a load balancer.

---

# 51. Typical Architecture

```text
                   Internet
                      |
                      ↓
                Load Balancer
                      |
                      ↓
                API Gateway
                      |
       +--------------+--------------+
       |              |              |
       ↓              ↓              ↓
    User           Product         Order
   Service         Service        Service
                                      |
                                      ↓
                                  Payment
```

---

# 52. Kubernetes Version

A cloud-native setup may look like:

```text
Internet
   |
Ingress / Load Balancer
   |
API Gateway
   |
Kubernetes Services
   |
Pods
```

Kubernetes handles:

```text
Service discovery
Pod scheduling
Scaling
Health management
```

while the API gateway handles application-level API concerns.

---

# 53. Gateway Security

Typical controls:

```text
TLS
JWT validation
OAuth2
Rate limiting
IP filtering
Request size limits
Security headers
Audit logging
```

Don't treat all of these as mandatory for every application.

---

# 54. Request Size Limits

A gateway can reject extremely large requests before they reach services.

Example:

```text
Maximum request body = 5 MB
```

This protects downstream services from:

```text
Memory pressure
Large payload attacks
Unexpected workloads
```

---

# 55. Timeout at Gateway

Example:

```text
Gateway timeout = 5 seconds
```

If a backend doesn't respond:

```text
Gateway stops waiting
```

But the gateway timeout must align with downstream timeout budgets.

Avoid inconsistent configurations such as:

```text
Gateway = 2 sec
Service = 10 sec
```

for an operation that requires the service's full processing time.

---

# 56. Gateway Observability

Monitor:

```text
Request count
Latency
Error rate
Status codes
Downstream latency
Rate-limit events
Authentication failures
```

Important metrics:

```text
p50
p95
p99 latency
```

---

# 57. Why p95/p99?

Average latency can hide slow requests.

Example:

```text
99 requests = 100 ms
1 request = 10 seconds
```

Average may not tell the whole story.

p95/p99 show tail latency.

---

# 58. Service Discovery Failure

What if the registry is unavailable?

A well-designed system may:

```text
Cache service locations
Use DNS/platform discovery
Fail gracefully
Use health-aware routing
```

The exact behavior depends on the discovery mechanism.

---

# 59. Eureka Failure

If using Eureka, understand that clients can cache registry information.

This can allow continued operation for some period even if the registry temporarily becomes unavailable.

But stale discovery data can still cause failed requests.

---

# 60. Kubernetes Discovery Failure

Kubernetes provides platform-managed discovery.

Applications typically resolve:

```text
service-name
```

through cluster DNS.

If the Kubernetes control plane has issues, already-running data-plane traffic can behave differently depending on the exact failure.

Avoid oversimplifying:

> "If Kubernetes control plane is down, all services immediately stop communicating."

That's not necessarily true.

---

# 61. Service Discovery Metadata

Metadata can be used for routing decisions.

Examples:

```text
version = v2
region = ap-south-1
zone = zone-a
```

This can support:

```text
Canary routing
Zone-aware routing
Region-aware routing
```

---

# 62. Canary Deployment

Suppose:

```text
Product Service v1 → 90%
Product Service v2 → 10%
```

A small percentage of traffic goes to the new version.

Monitor:

```text
Errors
Latency
Business metrics
```

Then gradually increase traffic if healthy.

---

# 63. Blue-Green Deployment

Two environments:

```text
Blue → current
Green → new
```

Traffic switches from:

```text
Blue
```

to:

```text
Green
```

after validation.

This can simplify rollback.

---

# 64. API Gateway Version Routing

The gateway can sometimes route:

```text
/api/v1/**
→ Service v1

/api/v2/**
→ Service v2
```

This can support gradual API evolution.

---

# 65. Service Discovery vs API Gateway

They solve different problems.

### Service Discovery

Answers:

> "Where are the service instances?"

### API Gateway

Answers:

> "Where should this client request go, and what gateway-level policies should apply?"

---

# 66. Service Discovery vs DNS

DNS can itself provide service discovery.

Example:

```text
payment-service.company.internal
```

resolves to the service endpoint.

In Kubernetes, DNS-based discovery is a common approach.

You don't always need a dedicated registry.

---

# 67. API Gateway vs Service Discovery

They can work together:

```text
Client
 ↓
Gateway
 ↓
Service Discovery / DNS
 ↓
Order instances
```

The gateway uses discovery to locate backend services.

---

# 68. Interview Question

### "Why do we need service discovery?"

Strong answer:

> "Because service instances are dynamic in distributed environments. Instances can scale, restart or move, so hardcoded addresses are unreliable. Service discovery provides a stable way to locate healthy service instances."

---

# 69. Interview Question

### "Why do we need an API Gateway?"

Strong answer:

> "It provides a controlled entry point for clients and centralizes concerns such as routing, authentication, rate limiting and observability. It also hides internal service topology from clients."

---

# 70. Interview Question

### "Does API Gateway replace service discovery?"

Answer:

> "No. They solve different problems. The gateway handles client-facing traffic and policies, while service discovery helps locate service instances. In some platforms, the gateway can use DNS or platform-native discovery to find services."

---

# 71. Interview Question

### "Why not let clients call every microservice directly?"

Answer:

> "It exposes internal topology to clients, increases client complexity and makes cross-cutting concerns harder to manage consistently. A gateway provides a stable external boundary."

---

# 72. Interview Question

### "Is API Gateway a single point of failure?"

Answer:

> "A single gateway instance could be, but production gateways are normally deployed redundantly behind load balancing with health checks and autoscaling where appropriate."

---

# 73. Interview Question

### "Should all business logic be in the gateway?"

Answer:

> "No. The gateway should mainly handle routing and cross-cutting concerns. Business rules should remain within the appropriate domain services."

---

# 74. Interview Question

### "Can the gateway call multiple services?"

Answer:

> "Yes. It can aggregate responses, especially for client-specific experiences, but excessive aggregation can create latency and coupling. For client-specific aggregation, a BFF can sometimes be a better boundary."

---

# 75. Interview Question

### "What happens when a service instance dies?"

Typical flow:

```text
Instance fails
 ↓
Health/readiness check detects issue
 ↓
Traffic stops going to it
 ↓
Replacement instance starts
 ↓
Service becomes ready
 ↓
Traffic resumes
```

The exact timing depends on the platform.

---

# 76. Interview Question

### "Why use readiness checks?"

Answer:

> "Readiness prevents traffic from reaching an instance that is running but not ready to serve requests, for example during startup or when a critical dependency makes the service unable to serve traffic."

---

# 77. Interview Question

### "What is a BFF?"

Answer:

> "Backend for Frontend is a backend layer tailored to a specific client, such as mobile or web. It can aggregate and transform data from multiple internal services to provide an API optimized for that client."

---

# 78. Interview Question

### "Gateway vs BFF?"

Answer:

> "A gateway is generally a common infrastructure/API boundary, while a BFF is specifically designed around the needs of a particular client experience. They can coexist."

---

# 79. Interview Question

### "What is a service registry?"

Answer:

> "A service registry maintains information about available service instances so clients or infrastructure can discover where to send requests."

---

# 80. Interview Question

### "What is Eureka?"

Answer:

> "Eureka is a service discovery solution from Netflix that has been integrated into Spring Cloud architectures. Services register with Eureka and clients can discover available instances."

---

# 81. Interview Question

### "What is Kubernetes service discovery?"

Answer:

> "Kubernetes Services provide stable network identities over changing Pods, and Kubernetes DNS allows applications to discover services using stable names instead of Pod IP addresses."

---

# 82. Interview Question

### "Client-side vs server-side discovery?"

Answer:

> "With client-side discovery, the client discovers instances and selects one. With server-side discovery, the client sends the request to a load balancer or routing layer that selects an instance."

---

# 83. Interview Scenario

### "You have 10 Order Service instances. One is unhealthy. How do you prevent traffic from reaching it?"

Answer:

> "I'd use readiness/health checks integrated with the load-balancing or service-discovery layer so unhealthy instances are removed from normal traffic."

---

# 84. Interview Scenario

### "Your gateway is receiving huge traffic."

I'd check:

```text
Gateway CPU
Gateway latency
Connection limits
Rate limits
Downstream latency
Error rates
Traffic distribution
```

Then scale horizontally and investigate the actual bottleneck.

---

# 85. Interview Scenario

### "Gateway latency is 100 ms but backend latency is 2 seconds."

The gateway isn't necessarily the bottleneck.

I'd inspect:

```text
Downstream service latency
Database queries
External dependencies
Network latency
Retries
Connection pools
```

---

# 86. Interview Scenario

### "Gateway is healthy but users receive 503."

Possible causes:

```text
No healthy backend instances
Service discovery issue
Backend unavailable
Connection failures
Circuit breaker open
Deployment problem
```

Check:

```text
Gateway logs
Routing configuration
Service health
Discovery/DNS
Backend metrics
```

---

# 87. Interview Scenario

### "Service discovery says an instance is healthy, but requests fail."

Possible reasons:

```text
Health check is too shallow
Application endpoint is failing
Network policy
Wrong port
Stale discovery
Dependency failure
```

Health checks should represent meaningful readiness, not just "process exists."

---

# 88. Common Mistakes

```text
❌ Hardcoded service IPs
❌ Single gateway instance
❌ No readiness checks
❌ Gateway containing business logic
❌ Gateway calling 20 services for every request
❌ Treating service discovery as authentication
❌ Treating gateway as a database
❌ Trusting every client header
❌ No rate limiting
❌ No observability
```

---

# 89. Practical Architecture

A production-style architecture might look like:

```text
                         Internet
                            |
                            ↓
                     Load Balancer
                            |
                            ↓
                      API Gateway
                            |
              +-------------+-------------+
              |             |             |
              ↓             ↓             ↓
          User Service  Product Service Order Service
              |             |             |
           User DB       Product DB      Order DB
                                            |
                                            ↓
                                      Payment Service

                Kubernetes / Cloud Platform
                         |
              +----------+----------+
              |                     |
         Service Discovery      Health Checks
              |
             DNS
```

---

# 90. Final Mental Model

Remember these four questions:

```text
Service Discovery
→ Where is the service?

Load Balancer
→ Which instance should receive traffic?

API Gateway
→ How should external traffic enter and be controlled?

BFF
→ What API should this specific client experience?
```

---

# 91. Interview-Ready Summary

If asked:

> "Explain API Gateway and service discovery."

Use:

> "Service discovery solves the problem of locating dynamic service instances. Instead of hardcoding IP addresses, services can discover available instances through a registry, DNS or platform-native mechanisms such as Kubernetes Services. An API Gateway is a client-facing entry point that handles routing and cross-cutting concerns such as authentication, rate limiting and observability. They solve different problems and can work together."

---

# 92. Revision Checklist

```text
□ Why hardcoded service URLs are problematic
□ Service discovery
□ Service registration
□ Service registry
□ Eureka
□ Heartbeats
□ Health checks
□ Readiness
□ Liveness
□ Client-side discovery
□ Server-side discovery
□ Load balancing
□ Kubernetes Service
□ Kubernetes DNS
□ API Gateway
□ Spring Cloud Gateway
□ Gateway routing
□ Path rewriting
□ Authentication
□ Authorization
□ Rate limiting
□ Request filtering
□ Header propagation
□ Correlation IDs
□ Distributed tracing
□ Gateway aggregation
□ BFF
□ Gateway scaling
□ Gateway failure
□ Gateway timeout
□ Gateway observability
□ Canary deployment
□ Blue-green deployment
□ Service discovery failure
□ Gateway vs load balancer
□ Gateway vs service discovery
□ Gateway vs BFF
□ Interview scenarios
```

---

# 93. Final Interview Answer

If asked:

> "Design the entry point for an e-commerce microservices system."

A concise answer:

> "I'd expose a load-balanced API Gateway as the external entry point. The gateway would handle TLS, authentication, routing, rate limiting and request-level observability. Backend services would remain independently deployable and discoverable through platform-native service discovery such as Kubernetes Services/DNS. I'd keep business logic inside the domain services rather than the gateway, and I'd use health/readiness checks so traffic only reaches healthy instances."

This demonstrates both **architecture knowledge and practical production thinking**.
