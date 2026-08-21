# REST API — Security, Authentication and Authorization

This file covers the security concepts that matter when designing and building production REST APIs with Spring Boot.

---

# 1. Why API Security Matters

REST APIs often expose:

```text
User data
Product data
Orders
Payments
Administrative operations
Business workflows
```

An API must protect:

```text
Authentication
Authorization
Confidentiality
Integrity
Availability
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
Login
  ↓
Authentication
  ↓
User identity established
  ↓
Authorization
  ↓
Can this user access /admin/products?
```

---

# 3. Common Authentication Methods

REST APIs can use:

```text
Session-based authentication
JWT
OAuth 2.0
OpenID Connect
API keys
Mutual TLS
```

The correct mechanism depends on the architecture and security requirements.

---

# 4. Session-Based Authentication

With a server-side session:

```text
Client
  ↓
Login
  ↓
Server creates session
  ↓
Session ID returned
  ↓
Client sends session ID
```

The server stores session state.

For multiple instances:

```text
Instance A ─┐
Instance B ─┼→ Shared session store
Instance C ─┘
```

Redis is commonly used for distributed sessions.

---

# 5. JWT Authentication

JWT stands for:

```text
JSON Web Token
```

A JWT can carry signed claims about a user.

Typical flow:

```text
Client
  ↓
Login
  ↓
Authentication service
  ↓
JWT
  ↓
Client
```

Later:

```text
Client
  ↓
Authorization: Bearer <token>
  ↓
API
  ↓
Validate JWT
  ↓
Authorize request
```

---

# 6. JWT Structure

A JWT commonly contains:

```text
Header
Payload
Signature
```

Conceptually:

```text
xxxxx.yyyyy.zzzzz
```

The three parts are Base64URL-encoded.

The payload is not secret simply because it is encoded.

Do not put sensitive information into JWT claims unless the design explicitly protects it.

---

# 7. JWT Claims

Common claims include:

```text
sub
iss
aud
exp
iat
nbf
jti
```

Application-specific claims can also be included.

Example:

```json
{
  "sub": "user123",
  "role": "USER",
  "exp": 1780000000
}
```

Keep tokens reasonably small.

---

# 8. JWT Signature

A signed JWT allows the server to detect whether the token was modified.

Conceptually:

```text
Header + Payload
       ↓
    Signing
       ↓
   Signature
```

The API verifies the signature using the appropriate key.

---

# 9. JWT Is Not Encryption

Important distinction:

```text
Signing → integrity/authenticity
Encryption → confidentiality
```

A normal signed JWT does not hide its payload from someone who can read the token.

Never place:

```text
passwords
secrets
private information
```

into a normal JWT payload.

---

# 10. Access Token

An access token represents permission to access protected resources.

Typical lifetime:

```text
Short
```

For example:

```text
5–30 minutes
```

The exact lifetime should be based on the security requirements.

---

# 11. Refresh Token

A refresh token can be used to obtain a new access token.

Flow:

```text
Access token expires
       ↓
Client sends refresh token
       ↓
Authorization server
       ↓
New access token
```

Refresh tokens generally require stronger protection than short-lived access tokens.

---

# 12. Access Token vs Refresh Token

Access token:

```text
Short-lived
Used frequently
Sent to APIs
```

Refresh token:

```text
Longer-lived
Used to obtain new access tokens
Should be stored more carefully
```

Do not automatically make refresh tokens extremely long-lived.

---

# 13. Bearer Token

A common HTTP header is:

```http
Authorization: Bearer eyJ...
```

The server extracts the token and validates it.

Bearer tokens must be protected because whoever possesses a valid bearer token can generally use it.

---

# 14. HTTPS

Authentication tokens should be transmitted over:

```text
HTTPS
```

Never send credentials or bearer tokens over unencrypted HTTP in production.

HTTPS protects the connection from network-level interception.

---

# 15. Password Storage

Never store passwords as:

```text
Plain text
```

Instead use a password hashing algorithm such as:

```text
Argon2
bcrypt
scrypt
PBKDF2
```

Example with Spring Security:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

The exact algorithm should follow current security guidance and project requirements.

---

# 16. Password Verification

Login flow:

```text
Client
  ↓
email + password
  ↓
Server
  ↓
Load password hash
  ↓
PasswordEncoder.matches(...)
  ↓
Authentication result
```

The server does not decrypt a password hash.

It verifies whether the supplied password matches the stored hash.

---

# 17. Never Log Passwords

Never log:

```text
password
password hash
access token
refresh token
credit card data
```

Be careful with request-body logging because sensitive information can accidentally appear in logs.

---

# 18. Role-Based Authorization

A simple authorization model uses roles:

```text
USER
ADMIN
MANAGER
```

Example:

```text
USER
  → view products

ADMIN
  → create/update/delete products
```

---

# 19. Method-Level Authorization

Spring Security can protect methods.

Example:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteProduct(Long id) {
    ...
}
```

This keeps authorization close to the protected operation.

---

# 20. Request-Level Authorization

Security rules can also protect URL patterns.

Conceptually:

```text
/api/products/** → authenticated
/api/admin/**    → ADMIN
/api/auth/**     → public
```

This provides coarse-grained security at the HTTP boundary.

---

# 21. Authentication Flow

Typical Spring Boot flow:

```text
HTTP Request
     ↓
Security Filter Chain
     ↓
Extract credentials/token
     ↓
Authenticate
     ↓
SecurityContext
     ↓
Authorization
     ↓
Controller
```

A controller should not manually parse JWTs if Spring Security is already responsible for authentication.

---

# 22. SecurityContext

After successful authentication, Spring Security can store the authenticated principal in the security context.

Conceptually:

```text
JWT
 ↓
Authentication
 ↓
SecurityContext
 ↓
Controller / Service
```

Application code can then access the authenticated identity through Spring Security APIs.

---

# 23. Current User

Example:

```java
Authentication authentication =
    SecurityContextHolder
        .getContext()
        .getAuthentication();
```

Then:

```java
String username =
    authentication.getName();
```

In many cases, injecting the principal into a controller is cleaner.

---

# 24. @AuthenticationPrincipal

Example:

```java
@GetMapping("/me")
public UserResponse me(
        @AuthenticationPrincipal
        CustomUserPrincipal user) {

    return userService.getProfile(user.getId());
}
```

This avoids manually retrieving the security context in every controller.

---

# 25. Stateless REST API

A stateless API does not require server-side session state for each request.

JWT-based APIs commonly use:

```text
Client stores token
      ↓
Token sent with each request
      ↓
Server validates token
```

Statelessness can simplify horizontal scaling.

---

# 26. Stateless Does Not Mean No Server State

Even a JWT system may maintain server-side state for:

```text
Refresh tokens
Token revocation
Rate limits
Sessions
Security events
```

Stateless authentication refers primarily to how request authentication is handled.

---

# 27. CSRF

CSRF means:

```text
Cross-Site Request Forgery
```

It is especially relevant when browsers automatically attach authentication credentials such as cookies.

For cookie-based authentication, CSRF protection should be designed explicitly.

---

# 28. JWT and CSRF

A JWT sent manually in:

```http
Authorization: Bearer ...
```

is not automatically attached by the browser in the same way as a session cookie.

That changes the CSRF threat model.

However, browser-based applications still require careful token storage and XSS protection.

---

# 29. XSS

XSS means:

```text
Cross-Site Scripting
```

An attacker attempts to execute malicious JavaScript in a user's browser.

Security considerations include:

```text
Input handling
Output encoding
Content Security Policy
Safe token storage
Avoiding unsafe HTML
```

---

# 30. Token Storage

For browser applications, token storage requires a threat-model decision.

Possible approaches include:

```text
HttpOnly Secure cookies
In-memory storage
Browser storage mechanisms
```

There is no universal answer without considering:

```text
XSS risk
CSRF protection
Application architecture
Token lifetime
Browser behavior
```

Avoid storing highly sensitive long-lived credentials in insecure browser storage.

---

# 31. CORS

CORS means:

```text
Cross-Origin Resource Sharing
```

It controls which browser origins can make cross-origin requests to an API.

Example:

```text
Frontend
https://frontend.example.com

API
https://api.example.com
```

The API can explicitly allow the frontend origin.

---

# 32. Avoid Wildcard CORS in Sensitive APIs

Avoid blindly configuring:

```text
Access-Control-Allow-Origin: *
```

especially when credentials are involved.

Prefer explicit trusted origins.

---

# 33. CORS vs Authentication

CORS does not replace authentication.

CORS answers:

```text
Which browser origins are allowed?
```

Authentication answers:

```text
Who is calling?
```

Authorization answers:

```text
What can they do?
```

These are different controls.

---

# 34. Rate Limiting

Rate limiting protects APIs from:

```text
Abuse
Brute-force attacks
Accidental overload
Resource exhaustion
```

Example:

```text
100 requests / minute / user
```

When exceeded:

```http
429 Too Many Requests
```

Redis is commonly used for distributed rate limiting.

---

# 35. Brute-Force Protection

Login endpoints are common attack targets.

Controls can include:

```text
Rate limiting
Account lockout policies
Progressive delays
CAPTCHA where appropriate
Monitoring
Multi-factor authentication
```

Do not reveal whether a username or email exists through overly specific login errors.

---

# 36. Login Error Messages

Avoid:

```text
"Email exists but password is wrong"
```

Prefer a generic message such as:

```text
"Invalid credentials"
```

This reduces account enumeration risk.

---

# 37. Account Enumeration

An attacker may test:

```text
Does user@example.com exist?
```

Avoid returning noticeably different responses for:

```text
Unknown user
Known user + wrong password
```

where practical.

---

# 38. JWT Expiration

Always validate:

```text
exp
```

A token should not remain valid forever.

Short-lived access tokens reduce the impact of token theft.

---

# 39. JWT Issuer and Audience

For systems with multiple services, validate appropriate claims such as:

```text
iss → trusted issuer
aud → intended audience
```

This prevents accepting a token issued for another application or service.

---

# 40. JWT Algorithm Validation

The server should only accept algorithms explicitly configured by the application.

Do not blindly trust the algorithm specified by an untrusted token header.

Use a well-tested JWT library and framework security configuration.

---

# 41. Key Management

Signing keys must be protected.

Avoid:

```java
String secret = "my-secret";
```

inside source code.

Prefer:

```text
Environment variables
Secret managers
KMS
Vault
Managed identity mechanisms
```

Rotate keys according to the organization's security policy.

---

# 42. Symmetric vs Asymmetric Signing

Symmetric:

```text
HS256
```

Uses one secret:

```text
signing service
      ↓
same secret
      ↓
verification service
```

Asymmetric:

```text
RS256 / ES256
```

Uses:

```text
Private key → signing
Public key  → verification
```

Asymmetric signing can be useful when many services need to verify tokens but only the authorization server should sign them.

---

# 43. OAuth 2.0

OAuth 2.0 is an authorization framework.

It allows a client to obtain access to resources without directly handling the user's password.

Conceptually:

```text
User
 ↓
Authorization Server
 ↓
Access Token
 ↓
Resource Server
```

OAuth is broader than simply "using JWT."

---

# 44. OpenID Connect

OpenID Connect:

```text
OAuth 2.0
+
Identity layer
```

It provides standardized identity information and authentication flows on top of OAuth 2.0.

---

# 45. Resource Server

In a microservices architecture:

```text
Authorization Server
       ↓
Access Token
       ↓
Resource Server
```

The resource server validates the access token and enforces authorization.

Spring Boot applications can be configured as OAuth2 resource servers.

---

# 46. API Gateway Security

A gateway can provide:

```text
Authentication
Rate limiting
TLS termination
Routing
CORS
Request filtering
```

But services should not blindly trust the gateway.

Important security decisions may still need to be enforced at the service level.

---

# 47. Service-to-Service Authentication

Microservices may need authentication between services.

Common mechanisms include:

```text
OAuth2 client credentials
mTLS
Signed service tokens
Managed identity
```

Do not rely only on:

```text
"Requests came from inside the network"
```

Modern systems often use zero-trust principles.

---

# 48. API Keys

API keys can identify clients.

Example:

```http
X-API-Key: abc123
```

They are useful for some:

```text
Server-to-server integrations
Developer APIs
Simple client identification
```

But API keys generally do not provide the same identity and delegated authorization model as OAuth 2.0.

---

# 49. Sensitive Endpoints

Protect endpoints such as:

```text
/admin/**
/payments/**
/users/**
/orders/**
```

using appropriate authentication and authorization.

Never assume a hidden URL is a security control.

---

# 50. IDOR

IDOR means:

```text
Insecure Direct Object Reference
```

Example:

```http
GET /api/orders/100
```

A user changes it to:

```http
GET /api/orders/101
```

If order 101 belongs to another user and the API returns it, there is an authorization flaw.

---

# 51. Object-Level Authorization

Checking:

```text
User is authenticated
```

is not enough.

The service may need to check:

```text
Does this order belong to this user?
```

Example:

```java
if (!order.getUserId().equals(currentUserId)) {
    throw new AccessDeniedException(...);
}
```

Authorization should be enforced server-side.

---

# 52. Mass Assignment

A dangerous pattern is allowing clients to bind arbitrary fields directly to entities.

For example:

```json
{
  "name": "Product",
  "price": 100,
  "role": "ADMIN"
}
```

A client should not be able to modify protected fields simply because they exist on an entity.

Use request DTOs and explicit mapping.

---

# 53. Input Validation Is Not Authorization

Validation checks:

```text
Is this input structurally valid?
```

Authorization checks:

```text
Is this user allowed to perform this action?
```

Both are necessary.

---

# 54. SQL Injection

Never build SQL by concatenating raw user input.

Bad:

```java
String sql =
    "SELECT * FROM users WHERE name = '"
    + input
    + "'";
```

Prefer:

```text
Parameterized queries
JPA
Spring Data
Prepared statements
```

---

# 55. JPA and SQL Injection

Using JPA does not automatically make every query safe.

Be careful with:

```text
Native SQL
JPQL string construction
Dynamic query generation
```

Use parameters rather than concatenating untrusted input.

---

# 56. Sensitive Response Data

Do not expose:

```text
Password hashes
Internal security flags
Database IDs when unnecessary
Secrets
Internal tokens
Private user information
```

Use DTOs to control the response.

---

# 57. Security Headers

Security-related HTTP headers can help protect web applications.

Examples:

```text
Content-Security-Policy
X-Content-Type-Options
Strict-Transport-Security
Referrer-Policy
```

Modern browsers and Spring Security can help configure appropriate headers.

---

# 58. HSTS

HSTS means:

```text
HTTP Strict Transport Security
```

It tells browsers to use HTTPS for the site.

Example:

```http
Strict-Transport-Security: max-age=31536000
```

Use it carefully and only when HTTPS is correctly configured.

---

# 59. Security Logging

Monitor:

```text
Repeated login failures
Invalid tokens
Unexpected authorization failures
Rate-limit violations
Suspicious access patterns
```

Security logs should avoid exposing credentials or raw tokens.

---

# 60. Secret Management

Never commit:

```text
JWT secrets
Database passwords
Cloud credentials
API keys
Private keys
```

into Git.

Use:

```text
Environment variables
Secret managers
Vault
Cloud secret services
```

and ensure secrets are excluded from logs and error responses.

---

# 61. Production Security Checklist

```text
□ HTTPS enabled
□ Passwords hashed
□ Strong password policy
□ JWT expiration configured
□ Signing keys protected
□ Issuer/audience validated where appropriate
□ Roles/permissions enforced
□ Object-level authorization
□ CORS restricted
□ CSRF considered
□ Rate limiting
□ Brute-force protection
□ Input validation
□ DTOs instead of direct entity binding
□ SQL injection protection
□ Security headers
□ Secrets outside source control
□ Sensitive logs prevented
□ Security events monitored
```

---

# 62. Spring Security Request Flow

```text
Client
  ↓
HTTPS
  ↓
Security Filter Chain
  ↓
Authentication
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
  ↓
Service
  ↓
Database
```

---

# 63. Ecommerce Security Flow

Example:

```text
POST /api/auth/login
        ↓
Authenticate credentials
        ↓
Issue access token
        ↓
Client
        ↓
GET /api/orders/100
        ↓
Validate JWT
        ↓
Identify user
        ↓
Check order ownership
        ↓
Return order
```

For:

```text
DELETE /api/products/100
```

the API may additionally require:

```text
ADMIN
```

---

# 64. Interview: Authentication vs Authorization

> Authentication verifies who the user is, while authorization determines what that authenticated user is allowed to access or modify. In Spring Security, authentication establishes the security context and authorization rules then decide whether the request can proceed.

---

# 65. Interview: How Does JWT Authentication Work?

> After successful login, the server issues a signed access token. The client sends it in the Authorization Bearer header on subsequent requests. Spring Security validates the token, creates the authenticated principal, and applies authorization rules before the controller executes.

---

# 66. Interview: Is JWT Encrypted?

> Not normally. A standard signed JWT provides integrity and authenticity, but its payload can generally be decoded. If confidentiality is required, sensitive data should not simply be placed in a signed JWT; an appropriate encryption mechanism is needed.

---

# 67. Interview: Why Use Short-Lived Access Tokens?

> If an access token is stolen, a shorter lifetime limits the attack window. Refresh tokens can be used to obtain new access tokens while applying stronger controls around their storage and rotation.

---

# 68. Interview: What Is 401 vs 403?

> 401 generally means the request does not have valid authentication. 403 means the user is authenticated but does not have sufficient permission for the requested resource.

---

# 69. Interview: How Do You Secure Passwords?

> I never store plaintext passwords. I use a strong password hashing algorithm such as bcrypt or Argon2 through Spring Security's password encoder infrastructure and verify passwords using the encoder's matching function.

---

# 70. Interview: How Do You Prevent IDOR?

> I don't rely only on authentication. I perform object-level authorization and verify that the authenticated user is allowed to access the specific resource. For example, before returning an order, I verify that the order belongs to the current user or that the user has an appropriate administrative role.

---

# 71. Interview: How Do You Secure a REST API?

> I use HTTPS, strong authentication, authorization at both endpoint and resource levels, input validation, secure password hashing, rate limiting, restricted CORS, protected secrets, safe error responses, security logging and dependency updates. I also make sure sensitive data is not exposed through DTOs or logs.

---

# 72. Interview: JWT vs Session?

> Sessions keep authentication state on the server, while JWT-based authentication commonly carries the authentication claims in a signed token sent with each request. JWTs can simplify horizontal scaling, but they introduce token lifetime, revocation and storage considerations. Sessions can provide easier server-side invalidation but require shared session state in a multi-instance deployment.

---

# 73. Interview: What Is CORS?

> CORS is a browser security mechanism that controls which origins can make cross-origin requests to an API. It is not an authentication mechanism. I normally configure explicit trusted origins rather than allowing arbitrary origins.

---

# 74. Interview: What Is CSRF?

> CSRF is an attack where a browser is tricked into sending an authenticated request to a server. It is especially relevant to cookie-based authentication because browsers automatically attach cookies. The mitigation depends on the authentication model and browser architecture.

---

# 75. Interview: How Do You Protect JWT Secrets?

> I keep signing keys outside the source code using environment configuration or a dedicated secret manager. In larger systems I would consider asymmetric signing so only the authorization server needs the private key while services verify tokens using the public key.

---

# 76. Interview Scenario — User Accesses Another User's Order

Question:

```http
GET /api/orders/500
```

The authenticated user does not own order 500. What should happen?

Answer:

> Authentication alone is not enough. The service should perform object-level authorization and verify ownership or an appropriate administrative permission. If the user is authenticated but not allowed to access it, the API should deny the request rather than returning the order.

---

# 77. Interview Scenario — Admin Endpoint

Question:

How would you protect:

```http
DELETE /api/products/{id}
```

Answer:

> I would require authentication and an appropriate admin permission. The authorization rule can be enforced at the HTTP layer and, for sensitive operations, also close to the service method using method-level security such as `@PreAuthorize`.

---

# 78. Interview Scenario — Login Brute Force

Question:

How would you protect a login endpoint?

Answer:

> I would apply rate limiting and monitoring, consider progressive delays or account protection policies, avoid revealing whether an account exists, and use MFA where appropriate. The goal is to slow automated attacks without creating an easy denial-of-service mechanism against legitimate users.

---

# 79. Final Mental Model

```text
                    CLIENT
                       ↓
                     HTTPS
                       ↓
              Security Filter Chain
                       ↓
                Authentication
                       ↓
                 SecurityContext
                       ↓
                Authorization
                       ↓
                  Controller
                       ↓
                    Service
                       ↓
                   Database
```

Security must exist at multiple layers:

```text
Transport
Authentication
Authorization
Input validation
Data access
Secrets
Logging
Monitoring
```

> **A secure REST API does not just authenticate users. It verifies identity, enforces permissions on the actual resource being accessed, protects credentials and secrets, validates input, limits abuse, avoids information leaks, and treats security as part of the architecture rather than a controller-level afterthought.**
