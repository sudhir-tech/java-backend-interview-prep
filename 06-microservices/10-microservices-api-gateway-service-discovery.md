# Microservices — API Gateway & Service Discovery

This file covers API Gateway and Service Discovery from a Java/Spring Boot microservices interview perspective.

Core topics:

```text
API Gateway
Reverse Proxy
Routing
Authentication
Authorization
Rate Limiting
Load Balancing
Retries
Timeouts
Circuit Breakers
CORS
Request Aggregation
Service Discovery
Client-side Discovery
Server-side Discovery
Eureka
Consul
Kubernetes Service Discovery
DNS
Health Checks
Gateway Security
Production Scenarios
Interview Questions
```

---

# 1. What Is an API Gateway?

An API Gateway is a single entry point through which clients access multiple backend services.

Without a gateway:

```text
Mobile App ──→ Order Service
            ──→ Product Service
            ──→ Payment Service
            ──→ User Service
```

With a gateway:

```text
                 ┌→ Order Service
Client → Gateway ├→ Product Service
                 ├→ Payment Service
                 └→ User Service
```

---

# 2. Why Use an API Gateway?

A gateway can centralize cross-cutting concerns such as:

```text
Routing
Authentication
Authorization
Rate limiting
TLS termination
Request/response filtering
Observability
CORS
Traffic policies
```

This avoids implementing the same infrastructure concern independently in every client.

---

# 3. Gateway Is Not a Business Service

The gateway should generally handle:

```text
Traffic management
Security enforcement
Routing
Cross-cutting concerns
```

Avoid putting core business logic such as:

```text
Calculate order price
Reserve inventory
Process payment
```

into the gateway.

---

# 4. Reverse Proxy

An API Gateway often acts as a reverse proxy.

Client sends:

```text
GET /api/products/101
```

Gateway forwards internally:

```text
GET /products/101
```

to:

```text
Product Service
```

The client doesn't need to know the internal service location.

---

# 5. Gateway Routing

Example:

```text
/api/products/** → Product Service
/api/orders/**   → Order Service
/api/users/**    → User Service
/api/payments/** → Payment Service
```

The gateway determines where the request should go.

---

# 6. Spring Cloud Gateway

For Spring Boot applications, Spring Cloud Gateway is a common gateway option.

Conceptually:

```text
Client
 ↓
Spring Cloud Gateway
 ↓
Spring Boot Services
```

Gateway routes can use:

```text
Path
Method
Header
Query parameter
Host
Other predicates
```

---

# 7. Gateway Filter

A gateway filter can modify or inspect a request/response.

Examples:

```text
Add headers
Remove headers
Validate tokens
Add correlation ID
Rate limit
Rewrite paths
Log requests
```

---

# 8. Global vs Route Filters

A global filter can apply across routes.

A route-specific filter applies only to selected routes.

Example:

```text
Global:
  correlation ID

Order route:
  order-specific policy
```

---

# 9. Authentication at Gateway

A gateway can validate authentication tokens before forwarding requests.

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
Service
```

This can reduce duplicate authentication work.

However, downstream services should still enforce authorization and trust boundaries appropriately rather than assuming the gateway is the only security boundary.

---

# 10. Authentication vs Authorization

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
JWT valid
```

means authentication may be successful.

But:

```text
USER trying to DELETE /admin/products/101
```

may still fail authorization.

---

# 11. JWT at Gateway

Typical flow:

```text
Client
 ↓
Authorization: Bearer <token>
 ↓
Gateway
 ↓
Validate JWT
 ↓
Forward request
```

The gateway may pass selected claims/context downstream.

Never blindly trust arbitrary client-provided identity headers.

---

# 12. Gateway Security Risk

A dangerous design:

```text
Client
 ↓
X-User-Role: ADMIN
 ↓
Gateway trusts header
```

A malicious client can forge it.

Identity information should come from trusted authentication mechanisms.

---

# 13. TLS Termination

The gateway can terminate TLS:

```text
HTTPS
 ↓
Gateway
 ↓
Internal network
 ↓
Services
```

Depending on security requirements, internal service-to-service traffic may also need encryption.

---

# 14. Rate Limiting

The gateway can limit traffic.

Example:

```text
100 requests/minute/user
```

If exceeded:

```text
HTTP 429 Too Many Requests
```

---

# 15. Why Rate Limit?

Protect services from:

```text
Abuse
Accidental traffic spikes
Bots
Brute-force attacks
Noisy clients
```

---

# 16. Rate Limiting Algorithms

Common approaches include:

```text
Fixed Window
Sliding Window
Token Bucket
Leaky Bucket
```

---

# 17. Token Bucket

Conceptually:

```text
Bucket capacity = 100
Refill = 10 tokens/sec
```

Each request consumes a token.

If no token is available:

```text
Reject or delay request
```

This allows controlled bursts while maintaining an average rate.

---

# 18. Gateway and Redis Rate Limiting

In a distributed gateway:

```text
Gateway instance 1 ─┐
Gateway instance 2 ─┼→ Redis
Gateway instance 3 ─┘
```

Redis can hold shared rate-limit state.

This avoids each gateway instance maintaining an isolated counter.

---

# 19. Gateway Load Balancing

A gateway may distribute traffic across service instances.

Example:

```text
Product Service

Instance 1
Instance 2
Instance 3
```

Traffic:

```text
Request 1 → Instance 1
Request 2 → Instance 2
Request 3 → Instance 3
```

---

# 20. Service Discovery

Service discovery allows applications to find service instances dynamically.

Without discovery:

```text
Order Service
 ↓
http://10.0.0.15:8081
```

But the IP may change.

With discovery:

```text
Order Service
 ↓
product-service
 ↓
Discovery mechanism
 ↓
Available instances
```

---

# 21. Why Service Discovery?

Microservice instances can:

```text
Scale up
Scale down
Restart
Move hosts
Change IPs
```

Hardcoding addresses doesn't scale well.

---

# 22. Service Registry

A service registry keeps track of service instances.

Conceptually:

```text
Service Registry
-----------------------------
product-service
  10.0.0.10:8081
  10.0.0.11:8081

order-service
  10.0.0.20:8082
```

---

# 23. Service Registration

When a service starts:

```text
Product Service
 ↓
Register with registry
 ↓
product-service
10.0.0.10:8081
```

It may periodically send heartbeats or health information depending on the discovery system.

---

# 24. Service Deregistration

When an instance stops gracefully:

```text
Product Service
 ↓
Deregister
```

If it crashes, the registry needs a way to detect that it is no longer healthy.

---

# 25. Heartbeat

A service can periodically tell the registry:

```text
"I'm alive."
```

Example:

```text
heartbeat every 30 seconds
```

If heartbeats stop, the registry can eventually remove or mark the instance unavailable according to its configuration.

---

# 26. Eureka

Netflix Eureka is a well-known service discovery technology historically used with Spring Cloud.

Conceptually:

```text
Eureka Server
      |
      +-- Product Service
      +-- Order Service
      +-- Payment Service
```

Services register with Eureka and discover other services through it.

---

# 27. Eureka Server

The Eureka Server acts as a service registry.

It maintains information such as:

```text
Service name
Host
Port
Instance metadata
Health/lease information
```

---

# 28. Eureka Client

A service can act as a Eureka client.

Example:

```text
Order Service
 ↓
Eureka
 ↓
Find product-service
 ↓
Receive instances
```

---

# 29. Client-Side Service Discovery

In client-side discovery:

```text
Client
 ↓
Service Registry
 ↓
Get instances
 ↓
Choose instance
 ↓
Call service
```

The client or client-side load balancer chooses the destination.

---

# 30. Server-Side Service Discovery

In server-side discovery:

```text
Client
 ↓
Load Balancer / Gateway
 ↓
Service Registry / platform
 ↓
Choose instance
 ↓
Service
```

The client doesn't need to know how service instances are selected.

---

# 31. Client-Side vs Server-Side Discovery

| Client-side | Server-side |
|---|---|
| Client selects instance | Gateway/LB selects instance |
| More logic in client | Simpler client |
| Registry visible to client logic | Registry hidden behind LB |
| Example: client-side load balancing | Example: Kubernetes Service / LB |

---

# 32. Load Balancing

Load balancing distributes traffic across healthy instances.

Common algorithms:

```text
Round Robin
Random
Weighted
Least Connections
Latency-aware
```

The exact algorithm depends on the platform/tool.

---

# 33. Round Robin

Example:

```text
Instance A
Instance B
Instance C
```

Requests:

```text
1 → A
2 → B
3 → C
4 → A
5 → B
```

Simple and common.

---

# 34. Weighted Load Balancing

Suppose:

```text
Instance A = weight 2
Instance B = weight 1
```

A can receive roughly twice as much traffic as B under the chosen algorithm.

Useful when instances have different capacities.

---

# 35. Health-Aware Routing

Don't route traffic to an unhealthy instance.

Example:

```text
A = healthy
B = unhealthy
C = healthy
```

Traffic should generally go to:

```text
A
C
```

rather than:

```text
B
```

---

# 36. Gateway + Discovery

A common architecture:

```text
                 ┌→ Product Service
                 │
Client → Gateway ├→ Order Service
                 │
                 └→ Payment Service
                        ↑
                 Service Discovery
```

Gateway can discover available instances dynamically.

---

# 37. DNS-Based Discovery

Many modern environments use DNS.

Example:

```text
order-service.default.svc.cluster.local
```

A service can resolve the DNS name to available endpoints through the platform.

This is common in Kubernetes.

---

# 38. Kubernetes Service Discovery

Kubernetes provides service discovery through Services and DNS.

Conceptually:

```text
Order Service
 ↓
http://product-service
 ↓
Kubernetes Service
 ↓
Product Pods
```

The application doesn't need to know individual Pod IP addresses.

---

# 39. Kubernetes Service

A Kubernetes Service provides a stable network identity in front of changing Pods.

Example:

```text
product-service
      |
      +--> Pod 1
      +--> Pod 2
      +--> Pod 3
```

Pods can be replaced without changing the service's stable identity.

---

# 40. Eureka vs Kubernetes Discovery

### Eureka

```text
Application-level service registry
```

### Kubernetes

```text
Platform-level service discovery
```

In Kubernetes deployments, native Services/DNS often make a separate Eureka layer unnecessary.

---

# 41. Don't Add Eureka Automatically

If your application already runs in Kubernetes:

```text
Kubernetes Service
+
DNS
```

may be sufficient.

Adding Eureka can introduce:

```text
Another infrastructure component
Another failure mode
More operational complexity
```

Choose based on architecture.

---

# 42. Gateway Routing with Discovery

Conceptually:

```text
/api/products/**
        ↓
lb://product-service
        ↓
Discovery/load balancer
        ↓
Product instance
```

The exact URI/configuration depends on the gateway implementation.

---

# 43. Gateway Timeouts

Every downstream call should have a sensible timeout.

Example:

```text
Gateway
 ↓
Payment Service
 ↓
timeout = 3 seconds
```

Don't allow requests to wait indefinitely.

---

# 44. Why Timeouts Matter

Without timeout:

```text
Payment hangs
 ↓
Gateway thread/connection waits
 ↓
More requests arrive
 ↓
Resources exhausted
```

Eventually the gateway itself may become unavailable.

---

# 45. Retry at Gateway

Retries can help with transient failures.

Example:

```text
Request
 ↓
Service unavailable
 ↓
Retry once
 ↓
Success
```

But retries can also make outages worse.

---

# 46. Retry Storm

Suppose:

```text
1000 requests
```

all fail and each retries 3 times.

Potential attempts:

```text
1000 × 4 = 4000
```

This can overload an already failing service.

Use:

```text
Bounded retries
Backoff
Jitter
Idempotency
Retry only safe/transient failures
```

---

# 47. Retry and Idempotency

Be careful retrying:

```text
POST /payments
```

If the operation isn't idempotent, a retry may create a duplicate payment.

Use an idempotency key or business-level mechanism where appropriate.

---

# 48. Exponential Backoff

Instead of:

```text
Retry immediately
Retry immediately
Retry immediately
```

use increasing delays:

```text
100ms
200ms
400ms
800ms
```

Exact values depend on the system.

---

# 49. Jitter

Jitter adds randomness to retry delays.

Without jitter:

```text
Many clients retry at exactly the same time
```

With jitter:

```text
Retries spread out
```

This reduces synchronized retry spikes.

---

# 50. Circuit Breaker

A circuit breaker prevents repeated calls to an unhealthy dependency.

States commonly include:

```text
CLOSED
OPEN
HALF_OPEN
```

---

# 51. CLOSED

Normal operation:

```text
Gateway
 ↓
Service
```

Failures are monitored.

---

# 52. OPEN

Too many failures:

```text
Gateway
 ↓
Circuit OPEN
 ↓
Fail fast
```

The gateway doesn't keep sending requests to the failing dependency.

---

# 53. HALF_OPEN

After a waiting period:

```text
Circuit OPEN
 ↓
HALF_OPEN
 ↓
Allow limited test calls
```

If successful:

```text
CLOSED
```

If failures continue:

```text
OPEN
```

---

# 54. Gateway + Circuit Breaker

Example:

```text
Client
 ↓
Gateway
 ↓
Payment Service
```

Payment is failing.

Circuit breaker:

```text
OPEN
```

Gateway can return:

```text
503 Service Unavailable
```

or a business-appropriate fallback.

---

# 55. Circuit Breaker vs Retry

Retry:

```text
Try again
```

Circuit breaker:

```text
Stop trying for a while
```

They can work together, but should be configured carefully.

---

# 56. Bulkhead

Bulkhead isolation limits the impact of one dependency consuming all resources.

Example:

```text
Payment calls → pool A
Inventory calls → pool B
```

If Payment becomes slow:

```text
Payment pool exhausted
```

Inventory can still have resources.

---

# 57. Gateway Request Aggregation

Sometimes the client needs data from multiple services.

Without aggregation:

```text
Client
 ├→ Product
 ├→ Inventory
 └→ Reviews
```

Gateway/BFF can potentially aggregate:

```text
Client
 ↓
Gateway/BFF
 ├→ Product
 ├→ Inventory
 └→ Reviews
 ↓
Combined response
```

---

# 58. BFF

BFF means:

```text
Backend For Frontend
```

Different clients can have different backend aggregation layers.

Example:

```text
Web BFF
Mobile BFF
```

Each can optimize responses for its client.

---

# 59. Gateway vs BFF

Gateway:

```text
Generic traffic entry point
```

BFF:

```text
Client-specific backend composition
```

They can coexist.

---

# 60. Gateway Shouldn't Become a Monolith

A common anti-pattern:

```text
Gateway
 ↓
All business logic
 ↓
Everything happens here
```

This effectively creates a distributed monolith with a giant central component.

Keep gateway responsibilities focused.

---

# 61. CORS

CORS controls which browser origins can make cross-origin requests.

Example:

```text
Frontend:
https://shop.example.com

API:
https://api.example.com
```

The gateway can enforce CORS policies.

---

# 62. CORS vs Authentication

CORS is a browser security mechanism.

It is not authentication.

```text
CORS
→ Can this browser origin access the resource?

Authentication
→ Who is the caller?
```

---

# 63. Path Rewriting

Gateway can transform:

```text
/api/v1/products/101
```

into:

```text
/products/101
```

before forwarding.

Useful for hiding internal URL structures.

---

# 64. Header Manipulation

Gateway can:

```text
Add headers
Remove headers
Rewrite headers
```

Example:

```text
X-Correlation-ID
```

can be added for observability.

Be careful not to overwrite trusted security headers with untrusted client values.

---

# 65. Request Size Limits

Gateway can reject oversized requests.

Example:

```text
Maximum body = 10 MB
```

This helps protect services from accidental or malicious large payloads.

---

# 66. Authentication Centralization Trade-Off

Centralizing authentication can simplify services.

But:

```text
Gateway becomes security-critical
```

Therefore:

```text
High availability
Strong security
Monitoring
```

are essential.

---

# 67. Gateway High Availability

Never make one gateway instance the only entry point.

Use:

```text
Load Balancer
 ↓
Gateway 1
Gateway 2
Gateway 3
```

---

# 68. Gateway Failure

If:

```text
Gateway DOWN
```

all external traffic may fail.

Therefore gateway should be:

```text
Horizontally scalable
Stateless where possible
Highly available
Observable
```

---

# 69. Stateless Gateway

Avoid storing important per-user session state only in gateway memory.

If:

```text
Gateway 1
```

holds session state and request goes to:

```text
Gateway 2
```

the user may lose state.

Prefer:

```text
JWT
or
Shared session store
```

depending on architecture.

---

# 70. Service Discovery Failure

What happens if discovery becomes unavailable?

Depends on implementation.

Possible strategies:

```text
Cached service locations
Platform-native discovery
Existing connections
Fail fast
Retry with backoff
```

Don't create an infinite retry loop.

---

# 71. Discovery and Health

Discovery should avoid routing traffic to instances that are not ready.

Conceptually:

```text
Instance starts
 ↓
Not ready
 ↓
No traffic
 ↓
Ready
 ↓
Traffic allowed
```

---

# 72. Registration vs Readiness

A service may technically be running:

```text
Process = alive
```

but not ready:

```text
DB migration incomplete
Cache initialization incomplete
Required dependency unavailable
```

Discovery/load balancing should use appropriate health/readiness information.

---

# 73. Service Discovery Metadata

Registry entries can contain metadata such as:

```text
Version
Zone
Region
Environment
Instance ID
```

This can support routing policies.

---

# 74. Zone-Aware Routing

Suppose:

```text
Region A
  Service 1
  Service 2

Region B
  Service 3
  Service 4
```

Prefer local instances when possible:

```text
Client in Region A
 ↓
Service 1/2
```

This can reduce:

```text
Latency
Cross-zone traffic
Cost
```

---

# 75. Blue-Green Deployment

Two environments:

```text
Blue = current
Green = new
```

Gateway/load balancer switches traffic:

```text
Blue
 ↓
Green
```

Observability is important before fully switching traffic.

---

# 76. Canary Routing

Route a small percentage to a new version:

```text
95% → v1
5%  → v2
```

Then increase:

```text
10%
25%
50%
100%
```

if metrics remain healthy.

---

# 77. Version Routing

Gateway may route based on:

```text
Path
Header
Cookie
Tenant
Weight
```

Example:

```text
/api/v1/orders → v1
/api/v2/orders → v2
```

---

# 78. Gateway and API Versioning

Don't use the gateway as an excuse for uncontrolled version proliferation.

Define:

```text
Deprecation policy
Compatibility period
Migration strategy
```

---

# 79. Gateway Logging

Log useful metadata:

```text
Route
HTTP method
Status
Latency
Trace ID
Correlation ID
Upstream service
```

Avoid logging:

```text
Passwords
Authorization tokens
Sensitive request bodies
```

---

# 80. Gateway Metrics

Monitor:

```text
Requests/sec
4xx
5xx
Latency
Route-level latency
Upstream failures
Timeouts
Retries
Circuit breaker state
Rate-limit rejections
```

---

# 81. Gateway Rate Limit Dimensions

Rate limits can be based on:

```text
IP
User
API key
Tenant
Endpoint
Combination
```

Choose based on the threat model and business rules.

---

# 82. Rate Limiting at Multiple Layers

Possible:

```text
CDN/WAF
 ↓
API Gateway
 ↓
Service
```

Each layer can protect against different types of traffic.

---

# 83. Gateway vs Load Balancer

Load balancer:

```text
Distributes traffic
```

API Gateway:

```text
Routing
Authentication
Rate limiting
Transformation
Observability
Policy enforcement
```

Modern infrastructure can overlap, so exact responsibilities depend on platform design.

---

# 84. Gateway vs Service Mesh

API Gateway:

```text
Primarily north-south traffic
Client → services
```

Service mesh:

```text
Primarily east-west traffic
Service → service
```

They can coexist.

---

# 85. North-South vs East-West

North-south:

```text
Internet
 ↓
Gateway
 ↓
Internal services
```

East-west:

```text
Order → Inventory
Order → Payment
```

---

# 86. Service Mesh

A service mesh can provide infrastructure-level capabilities such as:

```text
mTLS
Traffic routing
Retries
Timeouts
Telemetry
Service-to-service policy
```

Examples include:

```text
Istio
Linkerd
```

The exact capabilities vary.

---

# 87. Should Every Project Use a Service Mesh?

No.

A service mesh adds:

```text
Operational complexity
Infrastructure overhead
Learning curve
```

Use it when the organization/system actually benefits from those capabilities.

---

# 88. API Gateway Failure Scenario

### "Gateway returns 503 for all requests."

Check:

```text
Gateway health
Service discovery
DNS
Upstream health
Network
Certificates
Configuration
Recent deployment
Rate limits
Circuit breaker state
```

---

# 89. Service Discovery Failure Scenario

### "Order Service cannot find Product Service."

Check:

```text
Registry health
Service registration
Service name
DNS
Port
Network policy
Health status
Discovery client configuration
Recent deployment
```

---

# 90. Gateway Timeout Scenario

### "Only payment requests timeout."

Check:

```text
Gateway route
Payment service health
Payment dependency
Timeout settings
Connection pool
Thread pool
Trace
Payment service logs
```

---

# 91. Gateway Retry Scenario

### "After enabling retries, the system became slower."

Likely:

```text
Downstream service already overloaded
 ↓
Retries multiply traffic
 ↓
More overload
```

Fix:

```text
Reduce retries
Use exponential backoff
Add jitter
Retry only transient failures
Use circuit breaker
```

---

# 92. Service Discovery Interview Question

### "Why do we need service discovery?"

Answer:

> "Microservice instances are dynamic because they can scale, restart or move to different hosts. Service discovery lets clients or infrastructure find healthy instances without hardcoding IP addresses and ports."

---

# 93. Interview Question

### "What is an API Gateway?"

Answer:

> "An API Gateway is a centralized entry point for client traffic. It can handle routing and cross-cutting concerns such as authentication, rate limiting, observability, CORS and traffic policies before forwarding requests to backend services."

---

# 94. Interview Question

### "Gateway vs Load Balancer?"

Answer:

> "A load balancer primarily distributes traffic across instances, while an API Gateway usually provides richer application-level capabilities such as routing, authentication, rate limiting, filtering and request transformation. They can be deployed together."

---

# 95. Interview Question

### "What is service discovery?"

Answer:

> "Service discovery allows services to find dynamically changing service instances using a registry or platform mechanism rather than hardcoded addresses."

---

# 96. Interview Question

### "Client-side vs server-side discovery?"

Answer:

> "With client-side discovery, the client obtains service instances and chooses one. With server-side discovery, a gateway or load balancer receives the request and selects the service instance. Server-side discovery keeps discovery logic out of the client."

---

# 97. Interview Question

### "Eureka vs Kubernetes Service Discovery?"

Answer:

> "Eureka is an application-level service registry commonly associated with Spring Cloud architectures. Kubernetes provides native service discovery through Services and DNS. If the system is already running in Kubernetes, native discovery may remove the need for Eureka."

---

# 98. Interview Question

### "Why are retries dangerous?"

Answer:

> "Retries can multiply traffic against an already unhealthy dependency and create retry storms. I'd use bounded retries, exponential backoff, jitter and idempotency, and only retry failures that are likely to be transient."

---

# 99. Interview Question

### "What is a circuit breaker?"

Answer:

> "A circuit breaker prevents repeated calls to a failing dependency. It can open after repeated failures, fail fast for a period, and then enter half-open mode to test whether the dependency has recovered."

---

# 100. Interview Question

### "Why shouldn't business logic live in the gateway?"

Answer:

> "The gateway should focus on traffic and cross-cutting concerns. Putting business logic there creates a central bottleneck and makes the gateway difficult to scale and evolve independently."

---

# 101. Interview Scenario

### "Design an e-commerce API entry point."

A reasonable architecture:

```text
                    Internet
                       |
                       ↓
                 Load Balancer
                       |
                       ↓
                API Gateway
                 /    |    \
                /     |     \
               ↓      ↓      ↓
          Product   Order   User
            Service Service Service
                       |
                 +-----+------+
                 |            |
                 ↓            ↓
             Inventory      Payment
```

The gateway handles:

```text
Authentication
Routing
Rate limiting
Correlation ID
CORS
Timeouts
Circuit breakers
Observability
```

Services handle:

```text
Business logic
```

---

# 102. Interview Scenario

### "Product Service has 5 instances. How does Gateway choose one?"

Answer:

> "The gateway can use service discovery or a platform load balancer to obtain healthy instances and then apply a load-balancing strategy such as round robin or another configured policy."

---

# 103. Interview Scenario

### "Product Service is down. What should Gateway do?"

Possible flow:

```text
Gateway
 ↓
Product Service unavailable
 ↓
Circuit breaker / timeout
 ↓
Fail fast or business-appropriate fallback
 ↓
503 or fallback response
```

Don't wait indefinitely.

---

# 104. Interview Scenario

### "How do you protect the gateway?"

Use:

```text
TLS
Authentication
Rate limiting
Request size limits
WAF/CDN where appropriate
Security headers
Input validation
Monitoring
High availability
```

---

# 105. Interview Scenario

### "Gateway is a single point of failure. How do you solve it?"

Use:

```text
Multiple gateway instances
        ↓
Load Balancer
        ↓
Gateway 1
Gateway 2
Gateway 3
```

Keep the gateway stateless where practical.

---

# 106. Common Mistakes

```text
❌ Hardcoding service IPs
❌ Putting all business logic in gateway
❌ Infinite retries
❌ Retrying non-idempotent operations blindly
❌ No timeout
❌ No circuit breaker for fragile dependencies
❌ Trusting arbitrary identity headers
❌ One gateway instance
❌ Treating CORS as authentication
❌ Adding Eureka unnecessarily in Kubernetes
❌ Ignoring readiness
❌ No gateway observability
❌ Logging JWTs/passwords
```

---

# 107. Practical Gateway Checklist

Before deploying an API Gateway, ask:

```text
□ What routes exist?
□ How is authentication handled?
□ Where is authorization enforced?
□ What are the timeout values?
□ Which calls can be retried?
□ Are retries bounded?
□ Is idempotency handled?
□ Is rate limiting enabled?
□ Is CORS configured correctly?
□ Are request sizes limited?
□ Is service discovery available?
□ Are unhealthy instances excluded?
□ Is circuit breaking needed?
□ Is the gateway highly available?
□ Are logs structured?
□ Are trace IDs propagated?
□ Are gateway metrics monitored?
```

---

# 108. Practical Service Discovery Checklist

```text
□ How do services register?
□ How are unhealthy instances detected?
□ How are instances removed?
□ Is discovery highly available?
□ Is DNS available?
□ Is service identity stable?
□ How does load balancing work?
□ What happens if discovery fails?
□ Is platform-native discovery sufficient?
□ Is zone/region awareness needed?
```

---

# 109. Final Mental Model

Remember:

```text
API Gateway
→ Front door for external/client traffic

Reverse Proxy
→ Gateway forwards requests to internal services

Service Discovery
→ Finds service instances dynamically

Registry
→ Stores service instance information

Load Balancer
→ Chooses an instance

Timeout
→ Don't wait forever

Retry
→ Try again carefully

Circuit Breaker
→ Stop calling a failing dependency

Rate Limiter
→ Control traffic

BFF
→ Client-specific backend composition

Kubernetes Service
→ Stable service identity over changing Pods

DNS
→ Common service discovery mechanism in Kubernetes
```

---

# 110. Final Interview Answer

If asked:

> "How would you design API Gateway and service discovery for a Spring Boot microservices system?"

Use:

> "I'd place a highly available API Gateway behind a load balancer as the external entry point. The gateway would handle routing, authentication, rate limiting, CORS, correlation IDs, timeouts and appropriate circuit-breaking policies, while keeping business logic inside the services. For service discovery, I'd use the platform's native discovery mechanism where available, such as Kubernetes Services and DNS. In a Spring Cloud environment outside Kubernetes, Eureka or another discovery mechanism could be appropriate. I'd also make the gateway observable and carefully configure retries because uncontrolled retries can amplify downstream failures."

---

# 111. Revision Checklist

```text
□ API Gateway
□ Reverse proxy
□ Gateway routing
□ Spring Cloud Gateway
□ Gateway filters
□ Global filters
□ Route filters
□ Authentication
□ Authorization
□ JWT
□ TLS termination
□ Rate limiting
□ Token bucket
□ Load balancing
□ Service discovery
□ Service registry
□ Registration
□ Deregistration
□ Heartbeats
□ Eureka
□ Eureka Server
□ Eureka Client
□ Client-side discovery
□ Server-side discovery
□ Round robin
□ Weighted routing
□ Health-aware routing
□ DNS discovery
□ Kubernetes Service
□ Kubernetes DNS
□ Eureka vs Kubernetes
□ Gateway timeout
□ Retry
□ Retry storm
□ Exponential backoff
□ Jitter
□ Circuit breaker
□ Closed/Open/Half-open
□ Bulkhead
□ Request aggregation
□ BFF
□ CORS
□ Path rewriting
□ Header manipulation
□ Request size limits
□ Gateway HA
□ Stateless gateway
□ Discovery failure
□ Zone-aware routing
□ Canary
□ Blue-green
□ API versioning
□ Gateway metrics
□ Service mesh
□ North-south traffic
□ East-west traffic
□ Production scenarios
```

---

# 112. The Interviewer's Real Test

If asked:

> "Payment Service is down. What should happen when 5,000 checkout requests arrive?"

Don't say:

```text
Gateway retries them.
```

A stronger answer is:

```text
5,000 requests
      ↓
Gateway
      ↓
Payment calls start failing
      ↓
Timeouts + bounded retries
      ↓
Circuit breaker opens
      ↓
Fail fast
      ↓
Don't overload Payment further
      ↓
Return appropriate response
      ↓
Monitor + alert
      ↓
When Payment recovers
      ↓
Half-open test calls
      ↓
Circuit closes
```

The key interview lesson is:

> **A gateway isn't just a router. It is a critical resilience and policy boundary, so routing, security, retries, timeouts, discovery and observability must be designed together.**
