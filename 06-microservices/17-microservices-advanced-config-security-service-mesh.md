# Microservices — Advanced Architecture: Config, Secrets, Service Mesh & Security

This file covers advanced infrastructure and architecture topics commonly asked in Java/Spring Boot microservices interviews.

Core topics:

```text
Externalized Configuration
Spring Cloud Config
Config Server
Configuration Refresh
Secrets Management
Vault
Cloud Secret Managers
Service Discovery
Client-Side Load Balancing
API Gateway
Service Mesh
Sidecar
mTLS
Traffic Management
Retries
Circuit Breakers
Rate Limiting
Bulkheads
Authentication
Authorization
OAuth2
JWT
Service-to-Service Security
RBAC
Zero Trust
Network Policies
Distributed Configuration
Feature Flags
Resilience
Fault Isolation
Security Best Practices
Production Scenarios
Interview Questions
```

---

# 1. Why Advanced Infrastructure?

As the number of services grows:

```text
3 services
 ↓
10 services
 ↓
50 services
 ↓
100+ services
```

Managing:

```text
Configuration
Security
Networking
Retries
Authentication
Traffic
Certificates
```

individually becomes difficult.

Platform-level solutions help standardize these concerns.

---

# 2. Externalized Configuration

Application configuration should normally be outside the application binary.

Instead of:

```java
String url = "jdbc:mysql://prod-db:3306/orders";
```

use external configuration.

Conceptually:

```text
Application
    ↓
External Configuration
```

---

# 3. Why Externalize Configuration?

The same application artifact can run in:

```text
Development
Testing
Staging
Production
```

with different configuration.

Benefits:

```text
No rebuild for environment changes
Cleaner deployments
Centralized management
Reduced configuration duplication
```

---

# 4. Spring Boot Configuration

Common configuration sources include:

```text
application.properties
application.yml
Environment variables
Command-line arguments
External config files
Configuration services
```

Spring Boot combines configuration sources according to its property-resolution rules.

---

# 5. Spring Cloud Config

Spring Cloud Config provides centralized configuration management for distributed applications.

Conceptually:

```text
Config Repository
       ↓
Config Server
       ↓
+------+------+------+
|      |      |      |
Order Inventory Payment
```

---

# 6. Config Server

The Config Server provides configuration to applications.

Example:

```text
GET /order-service/default
```

The service receives configuration appropriate to its application/environment.

---

# 7. Config Repository

Configuration can be stored in a version-controlled repository.

Example:

```text
config-repo/
 ├── order-service.yml
 ├── inventory-service.yml
 ├── payment-service.yml
 └── application.yml
```

Git provides:

```text
Version history
Code review
Rollback
Auditability
```

---

# 8. Configuration Profiles

Spring supports profiles such as:

```text
dev
test
staging
prod
```

Example:

```text
application-dev.yml
application-prod.yml
```

Different environments can load different values.

---

# 9. Configuration Hierarchy

A common design:

```text
Global configuration
       ↓
Service-specific configuration
       ↓
Environment-specific configuration
```

Keep common settings centralized, while service-specific settings remain with the service.

---

# 10. Configuration Refresh

Some configuration can be refreshed without restarting an application.

But:

> Not every configuration property is safe to change at runtime.

Changes involving:

```text
Database connection
Thread pools
Security
Networking
```

may require controlled restart or reinitialization.

---

# 11. Dynamic Configuration

Dynamic configuration can be useful for:

```text
Feature flags
Timeout values
Non-critical application settings
Rate limits
```

Always consider:

```text
Consistency
Validation
Rollback
Auditability
```

---

# 12. Secrets vs Configuration

Configuration:

```text
Service URL
Port
Feature flag
Timeout
```

Secret:

```text
Password
API key
Private key
OAuth client secret
Database credential
```

Do not treat secrets like ordinary configuration.

---

# 13. Why Secrets Must Be Protected

If a secret leaks:

```text
Database access
API access
Cloud resources
Customer data
```

could be compromised.

Never commit production credentials to Git.

---

# 14. Secret Management

Common solutions:

```text
HashiCorp Vault
AWS Secrets Manager
Azure Key Vault
Google Secret Manager
Kubernetes Secrets
```

Use a dedicated secret manager when the platform and security requirements justify it.

---

# 15. Secret Rotation

Secrets should ideally be rotatable.

Example:

```text
Old DB password
      ↓
New DB password
```

A mature system should support:

```text
Rotation
Expiration
Audit
Revocation
```

---

# 16. Secret Rotation Challenge

If a service has:

```text
Database password = old
```

and the database changes to:

```text
password = new
```

the application must receive the new credential safely.

Plan rotation so old and new credentials can overlap when necessary.

---

# 17. Service Discovery

Microservices need to find each other.

Bad:

```text
http://10.42.3.18:8080
```

Pod/container addresses can change.

Better:

```text
inventory-service
```

through a service discovery mechanism.

---

# 18. Service Discovery Options

Depending on the platform:

```text
Kubernetes DNS
Eureka
Consul
Cloud-native service discovery
```

Kubernetes already provides service discovery through Services and DNS, so an additional registry may not be necessary.

---

# 19. Client-Side Load Balancing

Suppose:

```text
inventory-service
 ↓
Pod A
Pod B
Pod C
```

A client can distribute requests among available instances.

---

# 20. Server-Side Load Balancing

Another model:

```text
Client
 ↓
Load Balancer
 ↓
Pod A/B/C
```

The load balancer selects the destination.

---

# 21. Client-Side vs Server-Side

Client-side:

```text
Client chooses instance
```

Server-side:

```text
Load balancer chooses instance
```

Modern platforms can abstract these details.

---

# 22. API Gateway

An API Gateway provides a controlled entry point into backend services.

```text
Clients
   ↓
API Gateway
   |
   +→ Order
   +→ Product
   +→ User
```

---

# 23. Gateway Responsibilities

Common responsibilities:

```text
Routing
Authentication
Authorization
Rate limiting
TLS termination
Request filtering
Observability
API policies
```

Avoid putting large amounts of business logic into the gateway.

---

# 24. Gateway Authentication

Example:

```text
Client
 ↓
JWT
 ↓
API Gateway
 ↓
Validate token
 ↓
Backend Service
```

Depending on architecture, backend services may also validate tokens rather than blindly trusting gateway headers.

---

# 25. Gateway Rate Limiting

Example:

```text
Client
 ↓
Gateway
 ↓
100 requests/minute
```

Requests above the limit may receive:

```text
429 Too Many Requests
```

---

# 26. Why Rate Limit?

Protect:

```text
Backend services
Databases
External providers
```

from excessive traffic.

It can also reduce abuse.

---

# 27. Service Mesh

A service mesh provides infrastructure for service-to-service communication.

Conceptually:

```text
Order Service
   ↕
Sidecar / Proxy
   ↕
Network
   ↕
Sidecar / Proxy
   ↕
Inventory Service
```

---

# 28. Sidecar

A sidecar is a helper container/process deployed alongside the application.

Example:

```text
Pod
 ├── Spring Boot Container
 └── Proxy Container
```

The proxy can handle networking concerns.

---

# 29. What Can a Service Mesh Provide?

Depending on implementation:

```text
mTLS
Traffic routing
Retries
Timeouts
Circuit breaking
Observability
Service identity
Traffic policies
```

---

# 30. Why Use a Service Mesh?

Without a mesh:

```text
Every service
 ↓
implements networking concerns
```

With a mesh:

```text
Application
 ↓
Proxy
 ↓
Network policy
```

Common communication behavior can be standardized.

---

# 31. Service Mesh Examples

Popular technologies include:

```text
Istio
Linkerd
```

The exact choice depends on operational requirements.

---

# 32. Service Mesh Trade-Off

Benefits:

```text
Centralized traffic policies
mTLS
Observability
Consistent networking
```

Costs:

```text
Operational complexity
Extra infrastructure
Latency/overhead
Debugging complexity
Learning curve
```

Don't add a service mesh just because it is popular.

---

# 33. mTLS

mTLS means:

```text
Mutual TLS
```

Both sides authenticate each other.

Normal TLS:

```text
Client authenticates server
```

mTLS:

```text
Client ↔ Server
both authenticate
```

---

# 34. Why mTLS?

It can provide:

```text
Service identity
Encryption
Mutual authentication
Protection against unauthorized service calls
```

---

# 35. Service Identity

Instead of trusting:

```text
IP address
```

identify a workload using:

```text
Certificate
Workload identity
Cloud identity
Service account
```

This supports zero-trust principles.

---

# 36. Zero Trust

Zero Trust means:

> Don't automatically trust a request simply because it comes from an internal network.

Validate:

```text
Identity
Authentication
Authorization
Context
```

---

# 37. Zero Trust Example

Bad assumption:

```text
Inside cluster = trusted
```

Better:

```text
Order → Inventory
       ↓
Authenticate
       ↓
Authorize
       ↓
Allow
```

---

# 38. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

---

# 39. JWT

JWT is commonly used to carry claims about an authenticated identity.

Example conceptual claims:

```json
{
  "sub": "user123",
  "roles": ["USER"],
  "exp": 1780000000
}
```

A JWT commonly contains:

```text
Header
Payload
Signature
```

---

# 40. JWT Is Signed, Not Necessarily Encrypted

A standard signed JWT generally allows its payload to be decoded by anyone holding the token.

Therefore:

```text
Don't put passwords
Don't put secrets
Don't put sensitive data unnecessarily
```

A signature provides integrity/authenticity, not confidentiality.

---

# 41. OAuth2

OAuth 2.0 is an authorization framework.

Common use cases:

```text
User authorization
Service-to-service authorization
Delegated access
```

---

# 42. Client Credentials Flow

For machine-to-machine communication:

```text
Service A
 ↓
Authorization Server
 ↓
Access Token
 ↓
Service B
```

This is commonly called the client credentials flow.

---

# 43. Access Token

A service uses the access token when calling another protected service.

```http
Authorization: Bearer <token>
```

The receiving service validates the token according to the configured security model.

---

# 44. JWT Validation

Validation can include:

```text
Signature
Issuer
Audience
Expiration
Not-before
Required scopes/claims
```

Don't only check:

```text
Signature
```

---

# 45. Scope

Scopes represent permissions.

Example:

```text
orders:read
orders:write
payments:read
```

A service can require:

```text
orders:write
```

for a particular endpoint.

---

# 46. Role vs Scope

Role:

```text
ADMIN
USER
```

Scope:

```text
orders:read
orders:write
```

The exact authorization model depends on the application.

---

# 47. Service-to-Service Authentication

Options include:

```text
OAuth2 client credentials
mTLS
Cloud workload identity
Signed service credentials
```

Choose based on the platform and security requirements.

---

# 48. API Gateway Does Not Replace Service Security

A common mistake:

```text
Gateway validates user
 ↓
Backend trusts everything
```

If a backend can be reached through another route, the assumption may fail.

Services should enforce appropriate authorization themselves.

---

# 49. Defense in Depth

Security should exist at multiple layers:

```text
Internet
 ↓
WAF/Gateway
 ↓
Authentication
 ↓
Authorization
 ↓
Network policy
 ↓
Service authentication
 ↓
Database permissions
```

No single layer should be the only defense.

---

# 50. Least Privilege

Give each service only the permissions it needs.

Example:

```text
Order Service
→ Read inventory API

Order Service
→ Cannot access payment database
```

---

# 51. RBAC

Role-Based Access Control maps permissions to roles.

Example:

```text
ADMIN
 ├── create product
 ├── update product
 └── delete product

USER
 ├── view product
 └── create order
```

---

# 52. Network Policies

Restrict network communication.

Example:

```text
Order → Inventory = allowed
Order → Payment DB = denied
Notification → Order DB = denied
```

This reduces blast radius.

---

# 53. Traffic Management

Advanced traffic controls include:

```text
Timeouts
Retries
Circuit breakers
Load balancing
Traffic splitting
Rate limiting
Fault injection
```

---

# 54. Retry Policy

Retries should be:

```text
Bounded
Targeted
Backoff-based
Jittered where appropriate
```

Avoid:

```text
Retry forever
```

---

# 55. Exponential Backoff

Example:

```text
Attempt 1 → immediate
Attempt 2 → 100ms
Attempt 3 → 200ms
Attempt 4 → 400ms
```

Actual values depend on the system.

---

# 56. Jitter

Without jitter:

```text
1000 clients
 ↓
all retry at exactly 1 second
```

This can create a retry storm.

Jitter adds randomness to retry timing.

---

# 57. Retry Storm

Imagine:

```text
Payment Service
 ↓
temporarily unavailable
```

1000 clients retry simultaneously.

Traffic can become:

```text
Normal: 1000 req/sec
Retry storm: 5000 req/sec
```

The service becomes even less healthy.

---

# 58. Circuit Breaker

Circuit breaker states:

```text
CLOSED
OPEN
HALF_OPEN
```

---

# 59. CLOSED

Requests flow normally:

```text
Client
 ↓
Dependency
```

Failures are monitored.

---

# 60. OPEN

Too many failures:

```text
Circuit OPEN
```

Calls fail fast instead of repeatedly hitting the unhealthy dependency.

---

# 61. HALF_OPEN

After a recovery period:

```text
Circuit HALF_OPEN
```

A limited number of test requests are allowed.

If successful:

```text
→ CLOSED
```

If failures continue:

```text
→ OPEN
```

---

# 62. Bulkhead

Bulkhead isolates resources.

Example:

```text
Payment calls
→ Thread pool A

Inventory calls
→ Thread pool B
```

If Payment becomes slow, it should not consume all resources required by Inventory.

---

# 63. Timeout

Every remote call should have a bounded timeout.

Example:

```text
Inventory timeout = 500ms
```

Don't allow requests to wait indefinitely.

---

# 64. Timeout Budget

Suppose:

```text
API deadline = 2 sec
```

and:

```text
Inventory = 500ms
Payment = 800ms
```

The service should consider the overall request budget.

Nested services shouldn't independently wait several seconds each.

---

# 65. Rate Limiting

Rate limiting controls request frequency.

Algorithms can include:

```text
Token bucket
Leaky bucket
Fixed window
Sliding window
```

---

# 66. Token Bucket

Conceptually:

```text
Bucket
Capacity = 100
Refill = 10 tokens/sec
```

Each request consumes tokens.

No token:

```text
Reject / delay
```

---

# 67. Feature Flags

Feature flags allow controlled behavior changes without necessarily redeploying.

Example:

```text
new-checkout = false
```

Then:

```text
false → old flow
true  → new flow
```

---

# 68. Feature Flag Benefits

```text
Gradual rollout
Kill switch
A/B testing
Risk reduction
Decoupled release from launch
```

---

# 69. Feature Flag Risks

Too many flags create:

```text
Configuration complexity
Dead code
Unexpected combinations
Testing complexity
```

Flags should have ownership and cleanup plans.

---

# 70. Fault Isolation

A healthy architecture prevents one dependency from taking down everything.

Example:

```text
Notification Service DOWN
        ↓
Order creation should still work
```

if notification is not part of the critical synchronous transaction.

---

# 71. Graceful Degradation

If a recommendation service fails:

```text
Checkout
 ↓
still works
```

but:

```text
Recommendations unavailable
```

This is graceful degradation.

---

# 72. Fallback

A fallback provides an alternative response.

Example:

```text
Recommendation Service
 ↓
timeout
 ↓
Return cached recommendations
```

Be careful:

> A fallback should be meaningful and safe, not simply hide failures.

---

# 73. Resilience4j

Resilience4j is a popular Java library for resilience patterns.

It supports patterns such as:

```text
Circuit Breaker
Retry
Rate Limiter
Bulkhead
Time Limiter
```

---

# 74. Circuit Breaker Example

Conceptually:

```java
@CircuitBreaker(
    name = "inventory",
    fallbackMethod = "fallback"
)
public InventoryResponse getInventory(...) {
    ...
}
```

Exact configuration should be externalized.

---

# 75. Retry Example

Conceptually:

```java
@Retry(name = "inventory")
public InventoryResponse getInventory(...) {
    ...
}
```

Only retry errors that are actually transient and safe to retry.

---

# 76. Circuit Breaker + Retry

Be careful combining:

```text
Retry
+
Circuit Breaker
```

Poor configuration can multiply traffic.

Example:

```text
1 original request
× 3 retries
× 100 callers
```

can create substantial load.

---

# 77. Resilience Ordering

The exact ordering depends on the framework/configuration, but conceptually you want:

```text
Timeout
+
Retry only when appropriate
+
Circuit breaker
+
Bulkhead
```

with a clear understanding of how the chosen library composes these decorators.

---

# 78. Service Mesh vs Application Resilience

Application library:

```text
Resilience4j
```

can implement resilience inside Java code.

Service mesh:

```text
Proxy
```

can implement networking policies outside application code.

---

# 79. When Application-Level Resilience Is Useful

Use application code when the behavior depends on:

```text
Business semantics
Exception type
Fallback logic
Idempotency
Domain state
```

Example:

```text
Payment timeout
→ Put payment in PENDING
```

The service needs business knowledge.

---

# 80. When Infrastructure-Level Resilience Is Useful

Service mesh/network layer can handle generic concerns:

```text
mTLS
Traffic routing
Basic retries
Timeout policies
Load balancing
Telemetry
```

---

# 81. Don't Duplicate Everything

If both:

```text
Application
+
Service mesh
```

retry the same request multiple times, traffic can multiply unexpectedly.

Define ownership for each resilience policy.

---

# 82. Configuration Ownership

Decide:

```text
What belongs to application?
What belongs to platform?
What belongs to security team?
```

Avoid having the same setting controlled by multiple layers without clear precedence.

---

# 83. Certificate Management

mTLS requires certificates.

Production systems need:

```text
Issuance
Rotation
Expiration monitoring
Revocation strategy
Trust management
```

Manual certificate management does not scale well.

---

# 84. Service Mesh Certificate Rotation

A mesh can automate certificate lifecycle depending on implementation.

Conceptually:

```text
Service identity
 ↓
Certificate
 ↓
Automatic rotation
```

This reduces manual operational work.

---

# 85. Security Headers

For APIs, depending on exposure and architecture, consider appropriate HTTP security controls such as:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
```

Not every header applies to every backend service.

---

# 86. TLS Termination

TLS can terminate at:

```text
Load balancer
Ingress
Gateway
Application
```

For internal service-to-service security, mTLS may extend encryption/authentication further into the network.

---

# 87. Encryption in Transit

Protect communication:

```text
Client → Gateway
Gateway → Service
Service → Service
Service → Database
```

according to security requirements.

---

# 88. Encryption at Rest

Protect stored data:

```text
Database
Object storage
Backups
Secrets
Logs
```

Use platform/database encryption capabilities as appropriate.

---

# 89. Audit Logging

Security-sensitive actions should be auditable.

Examples:

```text
Login
Permission changes
Admin operations
Payment changes
Secret access
Configuration changes
```

Audit logs should be protected against unauthorized modification.

---

# 90. Security vs Observability

Observability data itself can be sensitive.

For example:

```text
Trace
 ↓
Request headers
 ↓
Authorization token
```

Do not automatically record all headers.

Redact sensitive values.

---

# 91. Production Scenario

### "One service is calling another service without authentication."

Fix:

```text
Service identity
+
Authentication
+
Authorization
```

Possible mechanisms:

```text
mTLS
OAuth2 client credentials
Workload identity
```

---

# 92. Production Scenario

### "Inventory service is failing and Order Service becomes overloaded."

Use:

```text
Timeout
Circuit breaker
Bulkhead
Bounded retries
```

Then ask whether Inventory really needs to be synchronous.

---

# 93. Production Scenario

### "1000 clients retry at the same time."

Problem:

```text
Retry storm
```

Mitigate with:

```text
Exponential backoff
Jitter
Retry limits
Circuit breaker
Rate limiting
```

---

# 94. Production Scenario

### "We want to release a new checkout flow to 5% of users."

Use:

```text
Feature flag
```

or:

```text
Canary deployment
```

depending on whether the change is application behavior or deployment-level traffic control.

---

# 95. Production Scenario

### "Production database password needs rotation."

Use:

```text
Secret manager
 ↓
New credential
 ↓
Controlled application refresh/restart
```

Design the database/application transition so there is no unnecessary outage.

---

# 96. Production Scenario

### "A service should only communicate with Inventory and User services."

Use:

```text
Network policies
+
Service authorization
```

Do not rely only on application code.

---

# 97. Production Scenario

### "We need encrypted and authenticated service-to-service communication."

Consider:

```text
mTLS
```

with managed certificate lifecycle.

---

# 98. Production Scenario

### "Different services implement different retry behavior."

Problem:

```text
Inconsistent resilience
```

Standardize generic policies through:

```text
Platform defaults
Service mesh
Shared libraries
```

while keeping business-specific behavior inside services.

---

# 99. Interview Question

### "What is Spring Cloud Config?"

Answer:

> "Spring Cloud Config provides centralized external configuration for distributed applications. A Config Server can serve version-controlled configuration to multiple services, reducing duplicated environment-specific configuration."

---

# 100. Interview Question

### "How should secrets be managed?"

Answer:

> "Secrets should never be hardcoded or committed to Git. I'd use a dedicated secret-management solution such as Vault or a cloud secret manager, with controlled access, encryption, auditing and rotation."

---

# 101. Interview Question

### "What is service discovery?"

Answer:

> "Service discovery allows a service to locate healthy instances of another service without hardcoding instance IP addresses. In Kubernetes, Services and DNS commonly provide this capability."

---

# 102. Interview Question

### "What is an API Gateway?"

Answer:

> "An API Gateway provides a controlled entry point for clients and can handle routing, authentication, rate limiting, TLS termination and other cross-cutting policies. Business logic should generally remain in backend services."

---

# 103. Interview Question

### "What is a service mesh?"

Answer:

> "A service mesh provides infrastructure-level control over service-to-service communication, often through proxies. It can provide mTLS, traffic management, observability and resilience policies without putting all of that networking logic into each application."

---

# 104. Interview Question

### "What is a sidecar?"

Answer:

> "A sidecar is a helper container deployed alongside an application container in the same Pod. In a service mesh, the sidecar proxy can handle networking concerns such as mTLS and traffic policies."

---

# 105. Interview Question

### "What is mTLS?"

Answer:

> "mTLS is mutual TLS, where both client and server authenticate each other using certificates. It provides encrypted communication and strong workload identity for service-to-service traffic."

---

# 106. Interview Question

### "What is Zero Trust?"

Answer:

> "Zero Trust means we don't automatically trust traffic just because it originates inside the network. Requests should be authenticated and authorized based on identity and policy."

---

# 107. Interview Question

### "JWT vs OAuth2?"

Answer:

> "JWT is a token format that can carry claims and be signed. OAuth2 is an authorization framework that defines how access can be delegated and tokens obtained. OAuth2 can use JWT access tokens, but OAuth2 and JWT are not the same thing."

---

# 108. Interview Question

### "What is client credentials flow?"

Answer:

> "It is an OAuth2 flow commonly used for machine-to-machine authorization. A service authenticates to an authorization server using its client credentials and receives an access token to call a protected service."

---

# 109. Interview Question

### "What is circuit breaker?"

Answer:

> "A circuit breaker prevents repeated calls to an unhealthy dependency after failures cross a threshold. It opens the circuit and fails fast, then periodically tests recovery in a half-open state."

---

# 110. Interview Question

### "What is bulkhead?"

Answer:

> "Bulkhead isolation prevents one dependency or workload from consuming all available resources. For example, separate connection or thread pools can stop a slow Payment service from exhausting resources needed by Inventory."

---

# 111. Interview Question

### "Why use exponential backoff and jitter?"

Answer:

> "Exponential backoff spaces retries further apart, while jitter adds randomness. Together they reduce retry storms when many clients experience the same temporary failure."

---

# 112. Interview Question

### "Service mesh or Resilience4j?"

Answer:

> "A service mesh is useful for infrastructure-level networking concerns such as mTLS, traffic routing and generic resilience policies. Resilience4j is useful when the application needs business-aware behavior such as deciding whether a specific exception is retryable or returning a domain-specific fallback."

---

# 113. Interview Question

### "Why isn't a gateway enough for security?"

Answer:

> "Because a gateway should not be the only trust boundary. Backend services may have other access paths, so services should still enforce appropriate authentication and authorization. This provides defense in depth."

---

# 114. Final Advanced Architecture

```text
                         Internet
                            |
                          HTTPS
                            |
                     +------+------+
                     | API Gateway |
                     +------+------+
                            |
                   Authentication
                   Rate Limiting
                   Routing
                            |
              +-------------+-------------+
              |             |             |
              ↓             ↓             ↓
           Order         Inventory      Payment
           Service        Service       Service
              |             |             |
           Proxy           Proxy         Proxy
              |             |             |
              +-------------+-------------+
                            |
                     Service Network
                            |
                   mTLS / Policies
                            |
       +--------------------+--------------------+
       |                    |                    |
       ↓                    ↓                    ↓
   Config Server       Secret Manager       Observability
       |                    |                    |
       ↓                    ↓                    ↓
   Git Config          Vault/Cloud        Metrics/Logs/Traces
```

---

# 115. Final Mental Model

```text
Configuration
→ Externalize it.

Secrets
→ Store securely and rotate.

Service Discovery
→ Don't hardcode instance addresses.

Gateway
→ Control external API traffic.

Service Mesh
→ Standardize service-to-service networking where justified.

mTLS
→ Authenticate and encrypt workloads.

OAuth2
→ Authorization framework.

JWT
→ Token format.

Zero Trust
→ Verify instead of assuming trust.

Circuit Breaker
→ Fail fast when dependencies are unhealthy.

Retry
→ Only transient/safe operations.

Backoff + Jitter
→ Avoid retry storms.

Bulkhead
→ Isolate resources.

Rate Limiting
→ Control traffic.

Feature Flags
→ Control application behavior.

Network Policies
→ Restrict communication.

Least Privilege
→ Give only required access.
```

---

# 116. Final Interview Answer

If asked:

> "How would you secure and manage communication between microservices?"

Use:

> "I'd use service identities and authenticated service-to-service communication, with mTLS or OAuth2 client credentials depending on the platform and requirements. I'd enforce authorization at the service level and use network policies for additional isolation. For external clients, I'd use an API Gateway for routing, authentication and rate limiting. For resilience, I'd configure bounded timeouts, targeted retries with backoff and jitter, circuit breakers and bulkheads. Configuration would be externalized and secrets managed through a dedicated secret manager with rotation and auditing."

---

# 117. Revision Checklist

```text
□ Externalized configuration
□ Spring Cloud Config
□ Config Server
□ Profiles
□ Dynamic configuration
□ Secrets
□ Secret rotation
□ Vault
□ Cloud secret managers
□ Service discovery
□ Kubernetes DNS
□ Client-side load balancing
□ Server-side load balancing
□ API Gateway
□ Gateway authentication
□ Gateway rate limiting
□ Service mesh
□ Sidecar
□ Istio
□ Linkerd
□ mTLS
□ Service identity
□ Zero Trust
□ Authentication
□ Authorization
□ JWT
□ OAuth2
□ Client credentials
□ Access tokens
□ Token validation
□ Scopes
□ RBAC
□ Least privilege
□ Network policies
□ Defense in depth
□ Traffic management
□ Retry
□ Exponential backoff
□ Jitter
□ Retry storm
□ Circuit breaker
□ Bulkhead
□ Timeout
□ Timeout budget
□ Rate limiting
□ Token bucket
□ Feature flags
□ Graceful degradation
□ Fallbacks
□ Resilience4j
□ Application vs infrastructure resilience
□ Certificate rotation
□ TLS
□ Encryption at rest
□ Audit logging
□ Production scenarios
```

---

# 118. The Interviewer's Real Test

If asked:

> "Payment Service is intermittently failing. Order Service retries three times, the service mesh also retries twice, and traffic has increased dramatically. What is wrong?"

Think:

```text
Application retry
      ×
Mesh retry
      ↓
Retry multiplication
      ↓
Retry storm
      ↓
Payment becomes even less healthy
```

Fix:

```text
Define one clear retry ownership model
        ↓
Bound retries
        ↓
Use exponential backoff
        ↓
Add jitter
        ↓
Use circuit breaker
        ↓
Use timeout
        ↓
Monitor dependency health
```

The key interview lesson is:

> **Advanced microservices architecture is about making distributed systems safer and easier to operate. Externalize configuration, protect secrets, establish service identity, control traffic, isolate failures and avoid duplicating the same resilience behavior across multiple layers.**
