# REST API — Security & Best Practices

This file covers practical REST API security in Spring Boot, including authentication, authorization, JWT, HTTPS, CORS, CSRF, input validation, secure headers, secrets, rate limiting, error handling, logging, API abuse protection, and interview scenarios.

---

# 1. Why REST API Security Matters

REST APIs are often directly exposed to:

```text
Browsers
Mobile apps
Frontend applications
Other services
Third-party clients
Automated clients
```

A secure API must protect:

```text
Identity
Data
Business operations
Credentials
Sessions/tokens
Infrastructure
```

---

# 2. Authentication vs Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to do?
```

Example:

```text
User logs in
    ↓
Authentication
    ↓
JWT issued
    ↓
Request arrives
    ↓
JWT validated
    ↓
Role checked
    ↓
Authorization
```

---

# 3. HTTPS

REST APIs should use HTTPS in production.

HTTPS provides:

```text
Encryption in transit
Server authentication
Integrity protection
```

Without HTTPS, sensitive information can potentially be intercepted.

Never send credentials or bearer tokens over plain HTTP in production.

---

# 4. TLS Termination

HTTPS may terminate at:

```text
Load Balancer
Reverse Proxy
API Gateway
Application
```

Example:

```text
Client
  ↓ HTTPS
Load Balancer
  ↓
Spring Boot
```

Even if TLS terminates before the application, the internal network should still be appropriately secured for the environment.

---

# 5. Password Storage

Never store passwords as plain text.

Bad:

```text
password = "mypassword"
```

Use a strong password hashing algorithm such as:

```text
BCrypt
Argon2
```

Conceptually:

```text
Password
   ↓
Password encoder
   ↓
Hash
   ↓
Database
```

---

# 6. Password Hashing vs Encryption

Hashing:

```text
One-way transformation
```

Encryption:

```text
Reversible with a key
```

Passwords should generally be hashed rather than encrypted for storage.

---

# 7. Spring Security PasswordEncoder

Example:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

During registration:

```text
Raw password
    ↓
PasswordEncoder
    ↓
Hash
    ↓
Database
```

During login:

```text
Raw password
    ↓
PasswordEncoder.matches(...)
    ↓
Stored hash
```

---

# 8. Never Log Passwords

Avoid:

```text
password
confirmPassword
CVV
secret keys
```

in:

```text
Application logs
Exception messages
Audit logs
Debug output
```

Sensitive data should be minimized everywhere, not only in the database.

---

# 9. JWT

JWT stands for:

```text
JSON Web Token
```

A JWT can carry signed claims such as:

```text
User ID
Subject
Roles
Expiration
Issuer
```

Typical flow:

```text
Login
 ↓
Credentials verified
 ↓
JWT generated
 ↓
Client stores token
 ↓
Client sends:
Authorization: Bearer <token>
 ↓
Spring Security validates token
```

---

# 10. JWT Structure

A JWT commonly contains:

```text
Header
Payload
Signature
```

Represented conceptually as:

```text
header.payload.signature
```

The payload is not automatically encrypted.

Do not put sensitive secrets into JWT claims simply because they are signed.

---

# 11. JWT Signing

A JWT can be signed using:

```text
HMAC secret
RSA keys
EC keys
```

The choice depends on the architecture and key-management requirements.

---

# 12. JWT Signature vs Encryption

Signing provides:

```text
Integrity
Authenticity of issuer
```

Encryption provides:

```text
Confidentiality
```

A normal signed JWT payload can be decoded by the holder.

Therefore:

> Never treat a signed JWT as encrypted storage.

---

# 13. JWT Expiration

JWTs should generally have an expiration time.

Example:

```text
exp = 15 minutes
```

Short-lived access tokens reduce the window of misuse if a token is stolen.

Long-lived tokens increase the risk period.

---

# 14. Refresh Tokens

A common architecture is:

```text
Access Token
    ↓
Short lifetime

Refresh Token
    ↓
Longer lifetime
```

When the access token expires:

```text
Refresh token
    ↓
Authorization server
    ↓
New access token
```

Refresh-token storage and rotation should be designed securely.

---

# 15. JWT Revocation

One challenge with stateless JWTs is revocation.

If a token is valid for:

```text
15 minutes
```

it may remain valid until expiration unless additional controls exist.

Possible strategies:

```text
Short token lifetime
Refresh-token rotation
Token denylist
Session state
Key rotation
```

Redis can be used for short-lived token state or denylist entries when appropriate.

---

# 16. JWT Secret Management

Never hardcode:

```java
private String secret =
    "my-super-secret-key";
```

Production secrets should come from:

```text
Environment variables
Secret managers
Vault
Cloud secret services
Deployment configuration
```

---

# 17. JWT Filter

A typical Spring Security flow:

```text
HTTP Request
     ↓
JWT Filter
     ↓
Extract Authorization header
     ↓
Validate token
     ↓
Create Authentication
     ↓
SecurityContext
     ↓
Controller
```

The filter should reject invalid credentials according to the security configuration.

---

# 18. Authorization Header

Standard format:

```http
Authorization: Bearer eyJ...
```

Avoid custom token formats unless there is a specific reason.

---

# 19. Role-Based Authorization

Example:

```text
ROLE_USER
ROLE_ADMIN
```

An endpoint can require:

```text
ADMIN
```

Example:

```java
@PreAuthorize("hasRole('ADMIN')")
```

The exact method-security configuration must be enabled for this to work.

---

# 20. Permission-Based Authorization

Instead of only roles:

```text
PRODUCT_READ
PRODUCT_WRITE
ORDER_CANCEL
USER_MANAGE
```

Permissions can provide finer-grained access control.

Example:

```text
ADMIN
    ↓
PRODUCT_WRITE
```

---

# 21. Method-Level Security

Spring Security supports annotations such as:

```java
@PreAuthorize("hasRole('ADMIN')")
```

and:

```java
@PreAuthorize("hasAuthority('PRODUCT_WRITE')")
```

This allows authorization close to business operations.

---

# 22. URL-Level Security

Security can also be configured at the HTTP level.

Conceptually:

```java
.requestMatchers(
    "/api/v1/admin/**"
).hasRole("ADMIN")
```

and:

```java
.requestMatchers(
    "/api/v1/products/**"
).authenticated()
```

---

# 23. Defense in Depth

Do not rely on only one authorization check.

For sensitive operations, consider:

```text
Gateway/infrastructure
        ↓
Spring Security
        ↓
Method authorization
        ↓
Business ownership checks
```

Multiple layers can reduce the impact of configuration mistakes.

---

# 24. Object-Level Authorization

A user may be authenticated but still not own the resource.

Example:

```http
GET /api/v1/orders/500
```

User A should not automatically be allowed to access User B's order.

The service should check:

```text
Does this user own order 500?
```

This is an important protection against IDOR/BOLA vulnerabilities.

---

# 25. IDOR / BOLA

Insecure Direct Object Reference or Broken Object Level Authorization can occur when:

```text
GET /users/100/orders
```

lets another user access data simply by changing:

```text
100 → 101
```

Authentication alone does not prevent this.

Always enforce authorization at the object/resource level.

---

# 26. Input Validation

Validate incoming data.

Example:

```java
@NotBlank
private String name;

@Positive
private BigDecimal price;
```

Useful annotations include:

```text
@NotNull
@NotBlank
@Size
@Min
@Max
@Positive
@Email
@Pattern
```

Validation should reflect business rules.

---

# 27. Never Trust Client Input

Clients can send:

```text
Unexpected fields
Invalid IDs
Huge values
Malformed JSON
Unexpected enum values
Malicious strings
```

Validate and constrain input before using it.

---

# 28. SQL Injection

Bad:

```java
String sql =
    "SELECT * FROM users WHERE name = '" + name + "'";
```

An attacker could manipulate the SQL.

Prefer:

```text
Prepared statements
Parameterized queries
JPA repositories
Validated query parameters
```

---

# 29. JPA and SQL Injection

Using JPA does not automatically make every query safe.

Be careful with:

```text
Native SQL
Dynamic JPQL
String concatenation
Dynamic ORDER BY
Dynamic table/column names
```

Parameters should be bound safely.

---

# 30. Dynamic Sorting

Suppose:

```http
GET /products?sort=price
```

Do not blindly insert the client value into SQL.

Instead:

```text
Allowed sort fields:
price
name
createdAt
```

Reject unknown values.

---

# 31. Mass Assignment

Avoid blindly binding all client fields to sensitive entities.

Bad example:

```text
Client sends:
role=ADMIN
```

and the application directly maps it into:

```text
User.role
```

Use request DTOs that expose only fields clients are allowed to change.

---

# 32. DTOs as Security Boundaries

Prefer:

```text
UserUpdateRequest
```

instead of accepting a complete:

```text
User entity
```

The DTO can expose:

```text
name
email
```

while preventing clients from directly modifying:

```text
role
passwordHash
createdAt
internalFlags
```

---

# 33. CORS

CORS stands for:

```text
Cross-Origin Resource Sharing
```

Browsers enforce same-origin restrictions.

CORS controls which browser origins may access the API.

---

# 34. CORS Example

Suppose frontend:

```text
https://app.example.com
```

API:

```text
https://api.example.com
```

The backend can allow the frontend origin.

Avoid blindly configuring:

```text
allowedOrigins = "*"
```

especially when credentials are involved.

---

# 35. CORS and Credentials

When cookies or credentials are involved, CORS must be configured carefully.

Do not combine:

```text
allowCredentials = true
```

with an unrestricted wildcard origin.

Explicitly configure trusted origins.

---

# 36. Preflight Request

Browsers may send:

```http
OPTIONS
```

before certain cross-origin requests.

This is called a preflight request.

The server may respond with headers describing allowed:

```text
Origins
Methods
Headers
Credentials
```

---

# 37. CSRF

CSRF stands for:

```text
Cross-Site Request Forgery
```

It is especially relevant when authentication relies on browser-managed credentials such as cookies.

The browser may automatically attach cookies to a request.

---

# 38. CSRF with Stateless Bearer Tokens

If an API uses:

```http
Authorization: Bearer <token>
```

and the token is not automatically attached by the browser as a cookie, the CSRF threat model is different.

CSRF protection decisions should be based on the authentication mechanism and application architecture.

---

# 39. CSRF with Session Cookies

If authentication uses:

```text
Session cookie
```

CSRF protection is usually important.

Spring Security provides CSRF support for this scenario.

---

# 40. XSS

XSS stands for:

```text
Cross-Site Scripting
```

It occurs when untrusted content is executed as browser code.

For REST APIs:

```text
Validate input
Encode output where appropriate
Use secure frontend rendering
Avoid returning unsafe HTML unnecessarily
```

API security is part of a broader application security model.

---

# 41. Content-Type Validation

Do not assume every request body is safe.

For JSON APIs:

```text
Content-Type: application/json
```

can be required.

Reject unsupported content types where appropriate.

---

# 42. Request Size Limits

Attackers may send extremely large requests.

Configure appropriate limits for:

```text
JSON body size
File uploads
Headers
URL length
Batch requests
```

The goal is to avoid unnecessary memory and processing consumption.

---

# 43. Rate Limiting

Rate limiting protects APIs from:

```text
Brute force
Abuse
Scraping
Traffic spikes
Accidental loops
Resource exhaustion
```

Example:

```text
100 requests/minute/user
```

Redis is commonly used for distributed rate limiting.

---

# 44. Login Rate Limiting

Authentication endpoints are especially sensitive.

For example:

```text
POST /auth/login
```

can be targeted by password-guessing attacks.

Use appropriate controls such as:

```text
Rate limiting
Progressive delays
Account protection
Monitoring
MFA where appropriate
```

Avoid revealing whether a username exists.

---

# 45. Brute Force Protection

Bad:

```text
Unlimited login attempts
```

Better:

```text
Rate limit
        ↓
Detect suspicious activity
        ↓
Apply temporary controls
```

The exact policy should balance security and legitimate user access.

---

# 46. User Enumeration

Avoid responses like:

```text
"Email does not exist"
```

for sensitive authentication flows if they allow attackers to discover registered accounts.

A more generic response can reduce account enumeration risk.

---

# 47. Secure Error Messages

Bad:

```text
SQLSyntaxErrorException:
jdbc:mysql://prod-db...
```

Bad:

```text
NullPointerException at UserService.java:84
```

External clients should receive controlled error responses.

Detailed diagnostic information belongs in secure internal logs.

---

# 48. Global Exception Handling

Use:

```java
@RestControllerAdvice
```

to create consistent error responses.

Example:

```json
{
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "Invalid product request"
}
```

Avoid exposing stack traces.

---

# 49. Logging Security

Log useful security events:

```text
Authentication failures
Authorization failures
Suspicious requests
Rate-limit violations
Important administrative actions
```

Do not log:

```text
Passwords
Full JWTs
API keys
Credit card numbers
Sensitive personal data
```

---

# 50. Correlation IDs

A correlation ID helps trace a request across services.

Example:

```http
X-Correlation-ID: 8f31...
```

Flow:

```text
Gateway
  ↓
Service A
  ↓
Service B
  ↓
Database
```

The same identifier can help connect logs across the request path.

Do not blindly trust client-provided IDs for security decisions.

---

# 51. Audit Logging

Audit logs record important actions.

Examples:

```text
Admin deleted product
User changed password
Order cancelled
Role changed
Payment status changed
```

Audit records should be:

```text
Structured
Protected
Timestamped
Access-controlled
```

---

# 52. Secrets Management

Never commit:

```text
Database passwords
JWT secrets
Cloud credentials
API keys
Private keys
```

to Git.

Use:

```text
Environment variables
Secret managers
Vault
Cloud secret services
```

and ensure secrets are excluded from source control.

---

# 53. .gitignore Is Not a Secret Manager

Adding:

```text
.env
```

to `.gitignore` prevents accidental future tracking, but it does not remove secrets that were already committed.

If a secret was committed:

```text
Rotate the secret
Remove it from repository history when appropriate
```

Treat it as compromised.

---

# 54. API Keys

If an API uses keys:

```http
X-API-Key: ...
```

protect them as credentials.

Consider:

```text
Rotation
Expiration
Scope
Rate limits
Revocation
Monitoring
```

---

# 55. Principle of Least Privilege

Give users/services only the permissions they need.

Example:

```text
Product read service
    ↓
READ_PRODUCTS
```

It should not automatically receive:

```text
DELETE_USERS
MANAGE_PAYMENTS
```

Least privilege reduces blast radius.

---

# 56. Service-to-Service Security

Microservices should not automatically trust every internal request.

Options include:

```text
mTLS
OAuth2 client credentials
Signed tokens
Service identities
Gateway authentication
Network policies
```

The right design depends on the architecture.

---

# 57. API Gateway

A gateway can centralize some cross-cutting concerns:

```text
TLS termination
Authentication
Rate limiting
Routing
Request filtering
Observability
```

But business authorization should still exist where appropriate.

---

# 58. Gateway Is Not Enough

Bad assumption:

```text
Gateway authenticates
↓
Internal services need no security
```

If an internal service can be reached another way, it may become vulnerable.

Use defense in depth.

---

# 59. Security Headers

Depending on architecture, useful HTTP security headers can include:

```text
Content-Security-Policy
Strict-Transport-Security
X-Content-Type-Options
Referrer-Policy
```

Some headers are more relevant to browser-facing applications than pure machine-to-machine APIs.

---

# 60. HSTS

HSTS stands for:

```text
HTTP Strict Transport Security
```

It tells compatible browsers to use HTTPS for the site.

Example:

```text
Strict-Transport-Security
```

Only enable HSTS with a clear understanding of the domain and deployment configuration.

---

# 61. API Versioning

Security changes can require API versioning.

Example:

```text
/api/v1
/api/v2
```

Versioning can help introduce:

```text
New authentication requirements
Changed authorization rules
New response schemas
Removed insecure behavior
```

without unexpectedly breaking all consumers.

---

# 62. Secure API Deprecation

When removing an insecure endpoint:

```text
Announce deprecation
 ↓
Provide replacement
 ↓
Migrate clients
 ↓
Disable old endpoint
```

Do not leave vulnerable legacy endpoints active indefinitely.

---

# 63. Secure File Uploads

If an API accepts files:

```text
POST /documents
```

validate:

```text
File size
Content type
Extension
File signature
Storage location
Filename
```

Do not trust the filename or MIME type supplied by the client alone.

---

# 64. File Upload Storage

Avoid serving uploaded files directly from executable application directories.

Consider:

```text
Object storage
Dedicated file storage
Non-executable directories
Generated filenames
Access controls
```

---

# 65. Path Traversal

Never blindly use client-provided paths.

Bad:

```text
filename = "../../../secret.txt"
```

Normalize and validate file paths and preferably generate server-side filenames.

---

# 66. SSRF

SSRF stands for:

```text
Server-Side Request Forgery
```

It can occur when an API accepts a URL and the server fetches it.

Example:

```http
POST /fetch
{
  "url": "http://..."
}
```

Attackers may attempt to access internal resources.

Use:

```text
Allowlists
URL validation
Network restrictions
DNS/IP validation
Outbound controls
```

---

# 67. Deserialization Security

Avoid unsafe deserialization of untrusted data.

Prefer:

```text
Explicit DTOs
Validated schemas
Known types
Safe serialization formats
```

Do not blindly deserialize arbitrary classes supplied by clients.

---

# 68. HTTP Method Restrictions

Expose only required methods.

Example:

```text
GET
POST
PUT
DELETE
```

Do not accidentally expose unsupported operations through generic routing.

---

# 69. HTTP Status Codes and Security

Use meaningful statuses:

```text
401 → authentication missing/invalid
403 → authenticated but not permitted
404 → resource not found
429 → rate limit exceeded
```

Do not leak sensitive information merely to make errors more descriptive.

---

# 70. Secure Pagination

Limit page size.

Bad:

```http
GET /products?size=10000000
```

Better:

```text
maximum size = 100
```

This protects:

```text
Database
Memory
CPU
Network
Response size
```

---

# 71. Secure Search

Search endpoints can be abused with:

```text
Very long strings
Complex filters
Expensive regex
Wildcard-heavy queries
Unbounded result sets
```

Use:

```text
Length limits
Allowed operators
Timeouts
Pagination
Query optimization
Rate limiting
```

---

# 72. Regular Expression DoS

Some regular expressions can take excessive CPU for malicious input.

Avoid unsafe regex patterns and constrain:

```text
Input length
Pattern complexity
Processing time
```

---

# 73. API Abuse Monitoring

Monitor:

```text
Requests per user
Requests per IP
Authentication failures
403 rates
404 spikes
429 rates
Unusual endpoints
Large payloads
Latency spikes
```

Security and observability should work together.

---

# 74. Dependency Security

Keep dependencies updated.

Monitor for:

```text
Known vulnerabilities
Outdated Spring Security
Vulnerable libraries
Transitive dependency issues
```

Tools may include:

```text
OWASP Dependency-Check
Snyk
GitHub Dependabot
```

Use the security tooling appropriate for the organization.

---

# 75. Security Testing

Security testing can include:

```text
Authentication tests
Authorization tests
Input validation tests
Dependency scanning
SAST
DAST
Penetration testing
API fuzzing
```

No single technique finds every security problem.

---

# 76. OWASP API Security Risks

Important API security areas include:

```text
Broken Object Level Authorization
Broken Authentication
Broken Object Property Level Authorization
Unrestricted Resource Consumption
Broken Function Level Authorization
Unrestricted Access to Sensitive Business Flows
Server-Side Request Forgery
Security Misconfiguration
Improper Inventory Management
Unsafe Consumption of APIs
```

These are useful areas to study for backend interviews.

---

# 77. Broken Object Level Authorization

Example:

```text
GET /orders/100
```

Application checks:

```text
JWT valid
```

but forgets:

```text
Does this user own order 100?
```

This is a classic API authorization problem.

---

# 78. Broken Function Level Authorization

Example:

```text
USER
 ↓
DELETE /admin/users/100
```

If the endpoint only checks authentication and not the required privilege, a normal user may access an administrative function.

---

# 79. Unrestricted Resource Consumption

Examples:

```text
Huge page size
Huge JSON body
Expensive search
Unlimited file upload
Unbounded batch request
```

Controls:

```text
Rate limiting
Request limits
Pagination
Timeouts
Quotas
```

---

# 80. API Inventory

Know what APIs exist.

Maintain:

```text
Endpoints
Versions
Consumers
Authentication requirements
Deprecated APIs
Internal/public classification
```

An unknown old endpoint can become a security liability.

---

# 81. Secure Ecommerce API

Example architecture:

```text
Client
  ↓ HTTPS
API Gateway
  ↓
Spring Security
  ↓
JWT validation
  ↓
Authorization
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
MySQL
```

Additional:

```text
Redis → rate limiting/cache
Kafka  → events
ELK    → logs/observability
```

---

# 82. Ecommerce Login

```text
POST /auth/login
        ↓
Validate request
        ↓
Find user
        ↓
Verify password hash
        ↓
Issue short-lived access token
        ↓
Return authentication response
```

Security controls:

```text
Rate limiting
Generic failure message
Secure logging
HTTPS
Strong password hashing
```

---

# 83. Ecommerce Product Authorization

```text
GET /products
    ↓
Public/authenticated depending on business requirement
```

Admin operation:

```text
POST /products
    ↓
Authenticated
    ↓
ADMIN
```

User should not be able to create or delete products simply by knowing the endpoint.

---

# 84. Ecommerce Order Authorization

```text
GET /orders/100
```

Flow:

```text
JWT valid
    ↓
Find order
    ↓
Check owner / permitted role
    ↓
Return order
```

Never assume the numeric ID itself is authorization.

---

# 85. Ecommerce Payment Security

Payment-related APIs should be especially careful with:

```text
Authentication
Authorization
Idempotency
Sensitive data
Logging
Replay protection
Rate limiting
Webhook verification
```

Never log payment credentials or other highly sensitive secrets.

---

# 86. Webhook Security

When receiving webhooks:

```text
Payment Provider
       ↓
POST /webhooks/payment
```

Verify:

```text
Signature
Timestamp
Event ID
Replay protection
```

Do not trust the webhook payload merely because it came from an apparently correct URL.

---

# 87. Idempotency and Security

Suppose:

```text
POST /orders
```

is retried.

Without idempotency:

```text
Request 1 → order created
Request 2 → duplicate order
```

A secure and reliable API can use:

```text
Idempotency-Key
```

to prevent duplicate business actions.

---

# 88. Replay Attacks

A captured valid request may be replayed.

Controls can include:

```text
Short-lived tokens
Timestamps
Nonces
Idempotency keys
Request signatures
Replay detection
```

The appropriate mechanism depends on the protocol.

---

# 89. Secure Caching

Do not cache sensitive responses accidentally.

Consider:

```text
Cache-Control
Authorization
User-specific data
Shared proxy behavior
TTL
```

A shared cache must never accidentally serve User A's private response to User B.

---

# 90. Cache Key Security

For user-specific data:

```text
profile:user:100
```

not:

```text
profile
```

if the result depends on the authenticated user.

Every input affecting the response must be represented in the cache key or otherwise isolated.

---

# 91. Security and Logging Example

Bad:

```text
User login failed:
username=sudhir
password=123456
token=eyJ...
```

Better:

```text
Authentication failed
userId=<internal-id>
reason=INVALID_CREDENTIALS
correlationId=abc123
```

Sensitive values should be omitted or masked.

---

# 92. Secure Configuration

Production configuration should include appropriate:

```text
HTTPS
Database credentials from secrets
JWT key from secrets
Redis authentication
CORS origins
Request limits
Logging level
Security headers
```

Never copy development security settings blindly into production.

---

# 93. Security Checklist

```text
□ HTTPS
□ Strong password hashing
□ JWT expiration
□ Secure secret management
□ Authentication
□ Authorization
□ Object-level authorization
□ Input validation
□ DTOs
□ SQL injection protection
□ CORS configuration
□ CSRF decision based on auth model
□ Rate limiting
□ Request size limits
□ Secure error handling
□ Sensitive-data log protection
□ Security headers where appropriate
□ Dependency scanning
□ API inventory
□ Audit logging
□ File-upload protection
□ SSRF protection where needed
□ Secure webhook verification
□ Idempotency for retry-sensitive operations
□ Monitoring and alerting
```

---

# 94. Interview: How Do You Secure a REST API?

> I start with HTTPS, strong authentication and authorization, input validation, secure password hashing and proper secret management. Then I add rate limiting, object-level authorization, secure error handling, logging controls, dependency scanning and monitoring. The exact controls depend on the API's threat model.

---

# 95. Interview: JWT vs Session?

> A session keeps authentication state on the server, while a JWT can carry signed claims that the server validates. JWTs can work well for distributed APIs, but revocation and token lifecycle need careful design. Sessions can make server-side invalidation simpler.

---

# 96. Interview: Is JWT Encrypted?

> Not necessarily. A standard JWT is usually signed, not encrypted. Its payload can therefore be decoded by someone who possesses the token. The signature protects integrity and authenticity, not confidentiality.

---

# 97. Interview: What Is CORS?

> CORS is a browser security mechanism that controls which origins can make cross-origin requests to an API. I configure it to allow only the trusted frontend origins and required methods and headers rather than using an unrestricted wildcard.

---

# 98. Interview: What Is CSRF?

> CSRF is an attack where a victim's browser is tricked into making an authenticated request to another site. It is particularly relevant to cookie-based authentication because browsers automatically attach cookies. For bearer tokens sent explicitly in an Authorization header, the threat model is different.

---

# 99. Interview: 401 vs 403?

> 401 means the request lacks valid authentication or credentials. 403 means the client is authenticated but does not have permission to perform the operation.

---

# 100. Interview: How Do You Prevent SQL Injection?

> I use parameterized queries, JPA repositories or prepared statements instead of concatenating user input into SQL. I also validate inputs and carefully handle dynamic query elements such as sorting fields.

---

# 101. Interview: What Is IDOR/BOLA?

> It happens when an API allows a user to access another user's resource simply by changing an object identifier. I prevent it by enforcing object-level authorization, for example checking that the authenticated user owns the requested order.

---

# 102. Interview: Why Use DTOs for Security?

> DTOs let me explicitly control which fields clients can send or receive. This prevents accidental exposure or modification of sensitive entity fields such as roles, password hashes and internal flags.

---

# 103. Interview: How Do You Protect Login?

> I use HTTPS, strong password hashing, rate limiting, generic authentication failure responses, secure logging and appropriate account-protection mechanisms. I also avoid exposing whether a particular username or email exists.

---

# 104. Interview: How Do You Secure Microservices?

> I don't assume the internal network is automatically trusted. Depending on the architecture, I can use service identities, OAuth2 client credentials, mTLS, network policies and authorization at the service level. Gateway controls are useful but shouldn't replace service-level security.

---

# 105. Interview: How Do You Handle Secrets?

> I never hardcode production secrets or commit them to Git. I use environment configuration or a dedicated secret manager, restrict access using least privilege, rotate credentials when needed and treat any committed secret as compromised.

---

# 106. Interview: How Do You Secure File Uploads?

> I enforce file-size limits, validate the file type and content, generate safe filenames, store files outside executable application paths and apply access controls. I don't trust the client-provided filename or MIME type alone.

---

# 107. Interview: How Do You Protect Against API Abuse?

> I use rate limiting, pagination limits, request-size limits, timeouts, validation and monitoring. For distributed applications, Redis can help coordinate rate limits across multiple instances.

---

# 108. Final Mental Model

```text
                    CLIENT
                       |
                     HTTPS
                       |
                       v
                API GATEWAY
                       |
            +----------+----------+
            |                     |
       Rate Limit            Request Rules
            |                     |
            +----------+----------+
                       |
                       v
               SPRING SECURITY
                       |
             +---------+---------+
             |                   |
       Authentication      Authorization
             |                   |
            JWT            Roles / Ownership
             |                   |
             +---------+---------+
                       |
                       v
                   CONTROLLER
                       |
                  Validation
                       |
                       v
                    SERVICE
                       |
               Business Rules
                       |
                       v
                 REPOSITORY
                       |
                       v
                    MYSQL
```

Additional controls:

```text
Redis  → rate limiting / safe caching
ELK    → observability / security events
Kafka  → asynchronous events
Secrets Manager → credentials
CI/CD  → dependency and security checks
```

> **REST API security is layered. HTTPS protects data in transit, authentication establishes identity, authorization controls access, validation protects application boundaries, and rate limiting, logging, monitoring and secure configuration reduce abuse and operational risk.**
