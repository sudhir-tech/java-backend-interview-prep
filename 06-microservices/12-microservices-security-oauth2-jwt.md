# Microservices — Security, OAuth2 & JWT

This file covers security concepts commonly asked in Java/Spring Boot microservices interviews.

Core topics:

```text
Authentication
Authorization
OAuth 2.0
OpenID Connect
JWT
Access Token
Refresh Token
Scopes
Roles
Claims
Resource Server
Authorization Server
Client
Bearer Token
Spring Security
Password Hashing
BCrypt
HTTPS/TLS
CORS
CSRF
API Gateway Security
Service-to-Service Security
mTLS
Secret Management
Token Validation
Token Expiration
Revocation
Security Best Practices
Production Scenarios
Interview Questions
```

---

# 1. Authentication

Authentication answers:

> "Who are you?"

Example:

```text
Username + Password
        ↓
Authentication
        ↓
User identified
```

---

# 2. Authorization

Authorization answers:

> "What are you allowed to do?"

Example:

```text
Authenticated user
        ↓
Role = USER
        ↓
GET /products     → allowed
DELETE /products  → denied
```

Remember:

```text
Authentication → identity
Authorization  → permissions
```

---

# 3. Microservices Security

A typical architecture:

```text
Client
  ↓
API Gateway
  ↓
Authentication
  ↓
Service
  ↓
Database
```

For service-to-service communication:

```text
Order Service
      ↓
Inventory Service
      ↓
Payment Service
```

Those internal calls also need appropriate security controls.

---

# 4. OAuth 2.0

OAuth 2.0 is an authorization framework for delegated access.

It allows a client to obtain access to protected resources without sharing the resource owner's password with the client.

Think:

```text
Client
  ↓
Authorization Server
  ↓
Access Token
  ↓
Resource Server
```

---

# 5. OAuth Roles

The OAuth model commonly describes:

```text
Resource Owner
Client
Authorization Server
Resource Server
```

---

# 6. Resource Owner

The resource owner is the entity that can grant access to a protected resource.

In many applications:

```text
User
```

---

# 7. Client

The client is the application requesting access.

Examples:

```text
Web application
Mobile application
Backend application
```

---

# 8. Authorization Server

The authorization server issues tokens.

Conceptually:

```text
Client
 ↓
Authorization Server
 ↓
Access Token
```

Examples can include identity providers such as:

```text
Keycloak
Auth0
Okta
Microsoft Entra ID
```

The exact provider depends on the organization.

---

# 9. Resource Server

The resource server hosts protected APIs.

Example:

```text
Product Service
Order Service
Payment Service
```

It validates access tokens before allowing access to protected resources.

---

# 10. Access Token

An access token represents authorization to access protected resources.

Example request:

```http
GET /api/orders
Authorization: Bearer <access-token>
```

The API validates the token.

---

# 11. Bearer Token

A bearer token is generally usable by whoever possesses it.

Therefore:

```text
Bearer token
=
Sensitive credential
```

Protect it carefully.

Never log access tokens casually.

---

# 12. JWT

JWT means:

```text
JSON Web Token
```

A JWT commonly contains:

```text
Header
Payload
Signature
```

Format:

```text
xxxxx.yyyyy.zzzzz
```

---

# 13. JWT Header

The header contains metadata.

Example:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

---

# 14. JWT Payload

The payload contains claims.

Example:

```json
{
  "sub": "101",
  "scope": "orders:read",
  "role": "USER",
  "exp": 1788000000
}
```

Don't put secrets in the payload.

---

# 15. JWT Signature

The signature protects token integrity.

Conceptually:

```text
Header + Payload
      ↓
Signing algorithm + key
      ↓
Signature
```

The resource server verifies the signature.

---

# 16. JWT Is Not Encryption

Very important interview point.

A normal signed JWT is:

```text
Encoded + signed
```

not:

```text
Encrypted
```

The payload can generally be decoded by anyone who has the token.

Therefore don't put:

```text
Password
Credit card number
Secret information
```

inside a normal JWT.

---

# 17. JWT Integrity

If someone changes:

```json
"role": "USER"
```

to:

```json
"role": "ADMIN"
```

the signature no longer matches.

The resource server rejects the modified token.

---

# 18. Symmetric Signing

Example:

```text
HS256
```

Same secret is used to sign and verify.

```text
Authorization Server
      |
   shared secret
      |
Resource Server
```

---

# 19. Asymmetric Signing

Examples:

```text
RS256
ES256
```

A private key signs the token.

A public key verifies it.

```text
Private key
    ↓
Sign

Public key
    ↓
Verify
```

This is often convenient in distributed systems because resource servers only need the public key.

---

# 20. Why Asymmetric Signing Is Useful

Suppose:

```text
Authorization Server
Order Service
Product Service
Payment Service
```

With asymmetric signing:

```text
Authorization Server
  → private key

Services
  → public key
```

Services don't need the private signing key.

---

# 21. Token Claims

Common JWT claims:

```text
iss → issuer
sub → subject
aud → audience
exp → expiration
iat → issued-at
nbf → not-before
jti → token identifier
```

Custom claims can also exist.

---

# 22. Issuer

`iss` identifies who issued the token.

Example:

```text
iss = https://auth.example.com
```

The resource server should validate the expected issuer.

---

# 23. Subject

`sub` identifies the subject of the token.

Often:

```text
user ID
client ID
principal identifier
```

depending on the system.

---

# 24. Audience

`aud` identifies the intended audience.

Example:

```text
aud = order-service
```

The service should verify that the token is intended for it when audience validation is part of the security design.

---

# 25. Expiration

`exp` determines when the token expires.

Example:

```text
Issued: 10:00
Expires: 10:15
```

After expiration:

```text
Token rejected
```

---

# 26. Issued At

`iat` indicates when the token was issued.

Useful for:

```text
Token age
Auditing
Security policies
```

---

# 27. JWT ID

`jti` is a unique token identifier.

It can support:

```text
Token tracking
Revocation strategies
Replay detection
```

depending on architecture.

---

# 28. OAuth2 Scopes

Scopes represent permissions granted to a client.

Example:

```text
orders:read
orders:write
products:read
```

Token:

```json
{
  "scope": "orders:read orders:write"
}
```

---

# 29. Roles

Roles group permissions.

Example:

```text
USER
ADMIN
MANAGER
```

A role might grant:

```text
ADMIN
→ product:create
→ product:update
→ product:delete
```

---

# 30. Scope vs Role

Scope:

```text
What access was granted?
```

Role:

```text
What category of permissions does the principal have?
```

They can overlap conceptually, but they serve different authorization models.

---

# 31. Fine-Grained Authorization

Don't only check:

```text
ROLE_ADMIN
```

Sometimes you need:

```text
Can this user update this particular order?
```

This is resource-level authorization.

Example:

```text
User 101
 ↓
GET /orders/500
```

The system must verify whether user 101 owns or is allowed to access order 500.

---

# 32. RBAC

RBAC means:

```text
Role-Based Access Control
```

Example:

```text
ADMIN → all product operations
USER  → read products
```

---

# 33. ABAC

ABAC means:

```text
Attribute-Based Access Control
```

Authorization can depend on attributes such as:

```text
User
Resource
Action
Location
Department
Time
```

Example:

```text
Manager
+
Same department
+
Business hours
→
Allowed
```

---

# 34. Authentication Flow

Simple login-based flow:

```text
User
 ↓
Login
 ↓
Authorization Server
 ↓
Authenticate user
 ↓
Issue tokens
 ↓
Client
```

Then:

```text
Client
 ↓
Access Token
 ↓
API
 ↓
Token Validation
 ↓
Response
```

---

# 35. Authorization Code Flow

For user-facing applications, Authorization Code is a common OAuth flow.

High-level:

```text
Browser
 ↓
Authorization Server
 ↓
User authenticates
 ↓
Authorization Code
 ↓
Client
 ↓
Token Endpoint
 ↓
Access Token
```

---

# 36. PKCE

PKCE means:

```text
Proof Key for Code Exchange
```

It protects the authorization code flow against certain code interception attacks.

Conceptually:

```text
Client generates:
code_verifier

Transforms it into:
code_challenge
```

The authorization server associates the challenge with the authorization request.

Later the client presents the verifier when exchanging the code.

---

# 37. Why PKCE?

If an attacker steals the authorization code:

```text
Stolen code
```

the attacker still cannot redeem it without the correct:

```text
code_verifier
```

This is especially important for public clients.

---

# 38. Client Credentials Flow

Used for machine-to-machine authentication when there is no end-user acting directly in the request.

Example:

```text
Order Service
 ↓
Authorization Server
 ↓
Access Token
 ↓
Inventory Service
```

---

# 39. Client Credentials Example

```text
Service A
 ↓
client_id + client_secret
 ↓
Authorization Server
 ↓
access token
 ↓
Service B
```

The token represents the client/application rather than a human user.

---

# 40. Refresh Token

A refresh token can be used to obtain a new access token without requiring the user to authenticate again, according to the authorization server's policy.

Conceptually:

```text
Access token expires
        ↓
Refresh token
        ↓
Authorization Server
        ↓
New access token
```

---

# 41. Access Token vs Refresh Token

| Access Token | Refresh Token |
|---|---|
| Used to access APIs | Used to obtain new access tokens |
| Usually shorter-lived | Often longer-lived |
| Sent to resource server | Normally sent only to authorization server |
| High exposure risk | Highly sensitive |

---

# 42. Don't Send Refresh Tokens to APIs

Generally:

```text
Client
 ↓
Refresh Token
 ↓
Authorization Server
```

not:

```text
Client
 ↓
Refresh Token
 ↓
Every microservice
```

---

# 43. Token Expiration

Short-lived access tokens reduce the impact of token theft.

Example:

```text
Access token = 10 minutes
```

If stolen:

```text
Limited lifetime
```

Still, expiration alone does not eliminate all risk.

---

# 44. Refresh Token Rotation

Some systems rotate refresh tokens.

Conceptually:

```text
Refresh token A
 ↓
New access token
New refresh token B
 ↓
Token A becomes invalid
```

This can reduce replay risk.

---

# 45. Token Revocation

JWTs are often self-contained.

Once issued, a resource server can validate the signature without calling the authorization server every time.

This makes immediate revocation harder.

---

# 46. JWT Revocation Strategies

Possible approaches:

```text
Short token lifetime
Refresh-token revocation
Token denylist
Introspection
Key rotation
Session management
```

Choose based on requirements.

---

# 47. JWT vs Session

Session-based:

```text
Client
 ↓
Session ID
 ↓
Server-side session store
```

JWT:

```text
Client
 ↓
JWT
 ↓
Server validates token
```

JWT can reduce server-side session state, but introduces token lifecycle and revocation considerations.

---

# 48. Stateless Authentication

A stateless resource server can validate a JWT without storing a user session.

Example:

```text
Request
 ↓
JWT
 ↓
Signature validation
 ↓
Claims validation
 ↓
Authorization
```

---

# 49. JWT Validation

Don't only verify the signature.

Validate appropriate claims such as:

```text
Signature
Issuer
Audience
Expiration
Not-before
Algorithm
```

and apply application-specific authorization checks.

---

# 50. Algorithm Confusion

Never blindly trust the algorithm supplied by an untrusted token.

Configure allowed algorithms.

For example:

```text
Expected:
RS256
```

Do not dynamically accept arbitrary algorithms.

---

# 51. Key Rotation

Signing keys should be rotated periodically according to security requirements.

Architecture:

```text
Authorization Server
 ↓
Private signing key

JWKS endpoint
 ↓
Public keys
 ↓
Resource Services
```

---

# 52. JWKS

JWKS means:

```text
JSON Web Key Set
```

It provides public keys used to verify signed tokens.

A resource server can obtain the appropriate public key based on the token's key identifier.

---

# 53. Key ID

JWT header can contain:

```json
{
  "alg": "RS256",
  "kid": "key-2026-01"
}
```

`kid` helps the verifier select the correct public key.

---

# 54. Spring Security Resource Server

A Spring Boot API can be configured as an OAuth2 Resource Server.

Conceptually:

```text
Client
 ↓
Bearer Token
 ↓
Spring Security
 ↓
JWT validation
 ↓
Controller
```

---

# 55. Spring Security JWT Flow

```text
HTTP Request
 ↓
Security Filter Chain
 ↓
Bearer Token extraction
 ↓
JWT Decoder
 ↓
Signature/claim validation
 ↓
Authentication object
 ↓
Authorization
 ↓
Controller
```

---

# 56. Security Filter Chain

Spring Security processes requests through a filter chain.

Security filters can handle:

```text
Authentication
Authorization
CSRF
Session handling
Security context
Exception handling
```

The exact filter chain depends on configuration.

---

# 57. SecurityContext

Spring Security stores the authenticated principal in the security context for the current request.

Conceptually:

```text
JWT
 ↓
Authentication
 ↓
SecurityContext
 ↓
Controller/service
```

---

# 58. Accessing Authenticated User

In Spring applications, the authenticated principal can be accessed through mechanisms such as:

```java
Authentication authentication
```

or:

```java
@AuthenticationPrincipal
```

Use the mechanism appropriate to the application.

---

# 59. Method-Level Authorization

Authorization can be applied at method level.

Conceptually:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteProduct(Long id) {
    ...
}
```

The exact expression depends on the configured authority mapping.

---

# 60. Role Prefix

Spring Security commonly represents roles with a `ROLE_` prefix internally.

For example:

```text
ROLE_ADMIN
```

while an expression may use:

```text
hasRole("ADMIN")
```

Understand the mapping instead of assuming role strings are identical everywhere.

---

# 61. Authority vs Role

Authority:

```text
PRODUCT_READ
```

Role:

```text
ADMIN
```

A role can be treated as a higher-level grouping of permissions.

Spring Security supports both concepts.

---

# 62. Endpoint Authorization

Example:

```text
GET /products
→ public

POST /products
→ ADMIN

DELETE /products/{id}
→ ADMIN
```

Authorization should be explicit.

---

# 63. Authentication at Gateway vs Service

Gateway:

```text
Authenticate/validate token
```

Service:

```text
Enforce authorization
```

Don't blindly rely on gateway-only checks.

Each protected service should have an appropriate trust boundary.

---

# 64. Token Forwarding

Sometimes the gateway forwards the access token to downstream services.

Example:

```text
Client
 ↓
JWT
 ↓
Gateway
 ↓
JWT
 ↓
Order Service
```

This allows downstream services to perform authorization based on the token.

---

# 65. Token Propagation Risk

Forwarding tokens increases the number of components that handle sensitive credentials.

Therefore:

```text
Protect tokens
Avoid logging
Use TLS
Limit scopes
```

---

# 66. Service-to-Service Authentication

Options include:

```text
OAuth2 client credentials
mTLS
Signed service credentials
Platform identity
```

Use the organization's security architecture.

---

# 67. OAuth2 Client Credentials

Example:

```text
Order Service
 ↓
Authorization Server
 ↓
Access Token
 ↓
Inventory Service
```

Inventory validates:

```text
Issuer
Audience
Scopes
Expiration
Signature
```

---

# 68. mTLS

mTLS means:

```text
Mutual TLS
```

Both sides authenticate using certificates.

Normal TLS:

```text
Client verifies server
```

mTLS:

```text
Client verifies server
Server verifies client
```

---

# 69. When mTLS Helps

Useful for:

```text
Service-to-service identity
Zero-trust architectures
Internal APIs
Strong workload authentication
```

---

# 70. OAuth2 vs mTLS

OAuth2:

```text
Token-based authorization
```

mTLS:

```text
Certificate-based mutual authentication
```

They can be used together.

---

# 71. Zero Trust

Zero Trust generally means:

> Don't automatically trust a network location or internal service just because it is inside the network.

Instead use:

```text
Identity
Authentication
Authorization
Encryption
Least privilege
Continuous verification
```

---

# 72. HTTPS

Always protect production API traffic with HTTPS.

```text
Client
 ↓
HTTPS
 ↓
Gateway
 ↓
HTTPS/internal encryption as required
 ↓
Service
```

---

# 73. Why HTTPS?

It provides:

```text
Confidentiality
Integrity
Server authentication
```

Without HTTPS, bearer tokens can be exposed on the network.

---

# 74. Password Storage

Never store passwords as plaintext.

Bad:

```text
password = "Sudhir123"
```

Good:

```text
password hash
```

---

# 75. BCrypt

BCrypt is a password hashing function commonly used in Java applications.

Conceptually:

```text
Password
 ↓
BCrypt
 ↓
Hash
 ↓
Database
```

When logging in:

```text
Password
 ↓
BCrypt verification
 ↓
Match?
```

---

# 76. Password Hashing vs Encryption

Hashing:

```text
One-way transformation
```

Encryption:

```text
Reversible with key
```

Passwords should generally be hashed with a password hashing algorithm rather than encrypted for later recovery.

---

# 77. Salt

Password hashing uses a salt so identical passwords don't necessarily produce identical hashes.

Modern password hashing libraries handle salt generation as part of their design.

---

# 78. CSRF

CSRF means:

```text
Cross-Site Request Forgery
```

It tricks a user's browser into sending an unwanted authenticated request.

---

# 79. CSRF and JWT

If authentication uses:

```text
Authorization: Bearer <token>
```

and the token is not automatically attached by the browser, CSRF exposure differs from cookie-based authentication.

But if authentication uses cookies:

```text
session cookie
```

CSRF protections become particularly important.

Don't simply say:

> "JWT means CSRF is impossible."

The authentication transport mechanism matters.

---

# 80. CORS

CORS controls browser cross-origin access.

Example:

```text
Frontend
https://shop.example.com

API
https://api.example.com
```

The API can specify which browser origins are allowed.

---

# 81. CORS Is Not Authentication

Remember:

```text
CORS
→ Browser cross-origin policy

Authentication
→ Identity

Authorization
→ Permissions
```

---

# 82. Security Headers

Depending on architecture, security headers can include:

```text
Content-Security-Policy
X-Content-Type-Options
Strict-Transport-Security
```

Configure them appropriately for the application.

---

# 83. Secrets

Never hardcode:

```text
JWT secret
Database password
API key
OAuth client secret
```

inside source code.

---

# 84. Secret Management

Use appropriate secret-management systems.

Examples:

```text
HashiCorp Vault
AWS Secrets Manager
Azure Key Vault
Google Secret Manager
Kubernetes Secrets
```

Kubernetes Secrets should still be handled carefully and are not automatically equivalent to a dedicated secrets manager.

---

# 85. Environment Variables

Environment variables can keep secrets out of source code:

```text
DB_PASSWORD
CLIENT_SECRET
```

But environment variables are not a complete secrets-management strategy.

Protect:

```text
Runtime environment
Logs
CI/CD
Access permissions
```

---

# 86. Least Privilege

Give services only the permissions they need.

Example:

```text
Order Service
→ read/write orders

Not:
→ full database administrator access
```

---

# 87. Scope-Based Least Privilege

Instead of:

```text
orders:*
```

use:

```text
orders:read
```

when write access isn't required.

---

# 88. Audience Restriction

A token intended for:

```text
order-service
```

should not automatically be accepted by:

```text
payment-service
```

Validate audience where appropriate.

---

# 89. Token Leakage

Potential leakage points:

```text
Logs
Browser storage
URLs
Error messages
Monitoring systems
Source code
Git history
```

Never put access tokens in URLs when an Authorization header is appropriate.

---

# 90. Token Storage

For browser applications, token storage requires careful security design.

Risks include:

```text
XSS
Token theft
Browser persistence
```

Use an architecture appropriate to the application's threat model rather than blindly choosing localStorage or cookies.

---

# 91. Refresh Token Security

Refresh tokens are highly sensitive.

Use:

```text
Secure transport
Limited exposure
Rotation where appropriate
Revocation
Secure storage
```

---

# 92. OAuth2 Authorization Code + PKCE

Typical modern browser/mobile flow:

```text
User
 ↓
Client
 ↓
Authorization Server
 ↓
Login
 ↓
Authorization Code
 ↓
Client
 ↓
Code + PKCE verifier
 ↓
Token Endpoint
 ↓
Access Token
```

---

# 93. Authorization Code vs Client Credentials

Authorization Code:

```text
User involved
```

Client Credentials:

```text
Machine-to-machine
No end-user authorization
```

---

# 94. OpenID Connect

OpenID Connect builds an identity layer on top of OAuth 2.0.

OAuth:

```text
Authorization
```

OIDC:

```text
Authentication + identity
```

OIDC introduces concepts such as:

```text
ID Token
UserInfo endpoint
```

---

# 95. ID Token

An ID token is intended for the client to learn information about the authenticated user.

It is not the same thing as an API access token.

Remember:

```text
ID Token
→ identity information for client

Access Token
→ authorization to access resource server
```

---

# 96. Don't Send ID Token to API as Access Token

A common mistake is:

```text
ID Token
 ↓
Resource API
```

The API should normally receive an access token intended for that API.

---

# 97. OAuth Authorization Server vs Resource Server

Authorization Server:

```text
Authenticates/authorizes
Issues tokens
```

Resource Server:

```text
Protects APIs
Validates access tokens
```

They are separate roles, even if one product performs both.

---

# 98. Token Introspection

Some systems use an introspection endpoint.

Conceptually:

```text
Resource Server
 ↓
Authorization Server
 ↓
Is token active?
 ↓
Response
```

This can support centralized token state/revocation but introduces network dependency and latency.

---

# 99. JWT Validation vs Introspection

JWT local validation:

```text
Fast
No network call for each request
Harder immediate revocation
```

Introspection:

```text
Centralized token state
Can support immediate invalidation
Adds network dependency
```

Choose based on requirements.

---

# 100. Security Architecture Example

```text
                         Authorization Server
                              |
                              | tokens
                              ↓
Client → Load Balancer → API Gateway
                              |
                    +---------+---------+
                    |         |         |
                    ↓         ↓         ↓
                 Order     Product   Payment
                 Service   Service   Service
                    |
                    ↓
                  MySQL
```

Security:

```text
HTTPS
JWT/OAuth2
Scopes
Roles
Audience validation
Least privilege
Secrets management
Service-to-service authentication
Observability
```

---

# 101. Production Security Checklist

```text
□ HTTPS everywhere required
□ Strong authentication
□ Explicit authorization
□ Short-lived access tokens
□ Secure refresh tokens
□ JWT issuer validation
□ JWT audience validation
□ JWT expiration validation
□ Algorithm allowlist
□ Key rotation
□ Least privilege
□ Narrow scopes
□ Secrets outside source code
□ Password hashing
□ Rate limiting
□ Input validation
□ CORS configured correctly
□ CSRF protection where applicable
□ Tokens not logged
□ Sensitive data redacted
□ Security monitoring
□ Dependency updates
```

---

# 102. Security Incident Scenario

### "A JWT was accidentally logged."

Response:

```text
1. Treat token as compromised
2. Determine token scope and lifetime
3. Revoke/disable where possible
4. Rotate affected credentials if necessary
5. Remove sensitive logs according to retention procedures
6. Investigate access
7. Fix logging configuration
8. Add preventive controls
```

Do not assume the token is harmless because it expires later.

---

# 103. Security Incident Scenario

### "A service accepts any JWT with a valid signature."

Problem:

```text
Signature valid
```

but perhaps:

```text
Wrong issuer
Wrong audience
Expired
Wrong scope
```

Fix:

```text
Validate signature
+
issuer
+
audience
+
expiration
+
authorization
```

---

# 104. Security Incident Scenario

### "User can access another user's order."

Authentication succeeded:

```text
User = 101
```

Request:

```text
GET /orders/500
```

But authorization failed to verify ownership.

Fix:

```text
Authenticate
 ↓
Authorize action
 ↓
Check resource ownership
```

---

# 105. Security Incident Scenario

### "Internal service trusts X-User-Id header."

Problem:

```text
Client
 ↓
X-User-Id: admin
```

Client can forge it unless the header is protected by a trusted boundary.

Use:

```text
Validated token claims
Trusted identity propagation
mTLS/service identity
```

as appropriate.

---

# 106. Security Incident Scenario

### "Access token is valid but user is not allowed to delete a product."

Answer:

```text
Authentication = successful
Authorization = failed
```

Return an appropriate authorization error such as:

```text
403 Forbidden
```

---

# 107. 401 vs 403

Generally:

```text
401 Unauthorized
→ Authentication is missing/invalid

403 Forbidden
→ Request is understood/authenticated, but access is not permitted
```

The exact API behavior should be consistent with the security architecture.

---

# 108. Security + Gateway

Gateway can provide:

```text
Token validation
Rate limiting
TLS
CORS
Basic policy enforcement
```

But services should still enforce:

```text
Business authorization
Resource ownership
Service-specific permissions
```

---

# 109. Security + Microservices

Every service should know:

```text
Who is calling?
What are they allowed to do?
Is this request intended for me?
```

Do not rely purely on:

```text
"it's internal traffic, so it's trusted."
```

---

# 110. Security Mental Model

Remember:

```text
Authentication
→ Who are you?

Authorization
→ What can you do?

OAuth2
→ Delegated authorization framework

OIDC
→ Identity layer on OAuth2

JWT
→ Token format

Access Token
→ API authorization credential

Refresh Token
→ Obtain new access tokens

Scope
→ Granted permission

Role
→ Permission grouping

Resource Server
→ Protects APIs

Authorization Server
→ Issues tokens

PKCE
→ Protects authorization-code exchange

mTLS
→ Mutual service identity

HTTPS
→ Encrypts/authenticates transport

Least Privilege
→ Give only required access
```

---

# 111. Interview Question

### "What is OAuth 2.0?"

Answer:

> "OAuth 2.0 is an authorization framework that allows a client to obtain delegated access to protected resources using access tokens without sharing the resource owner's password with the client."

---

# 112. Interview Question

### "What is JWT?"

Answer:

> "JWT is a compact token format containing claims that can be signed to provide integrity. A signed JWT is not encrypted by default, so sensitive information shouldn't be placed in its payload."

---

# 113. Interview Question

### "JWT vs OAuth2?"

Answer:

> "OAuth2 is an authorization framework, while JWT is a token format. OAuth2 can use JWT access tokens, but OAuth2 itself is not synonymous with JWT."

---

# 114. Interview Question

### "What is OpenID Connect?"

Answer:

> "OpenID Connect is an identity layer built on OAuth2. It provides authentication and identity information, including the ID token concept, whereas OAuth2 primarily addresses authorization."

---

# 115. Interview Question

### "Access token vs refresh token?"

Answer:

> "The access token is used to access protected APIs and is usually short-lived. A refresh token is used with the authorization server to obtain new access tokens and is normally kept away from resource APIs."

---

# 116. Interview Question

### "Why use asymmetric JWT signing?"

Answer:

> "The authorization server signs tokens with a private key while resource servers verify them using a public key. This means resource servers don't need access to the private signing key."

---

# 117. Interview Question

### "What is PKCE?"

Answer:

> "PKCE adds a proof key to the authorization code flow. The client creates a code verifier and corresponding challenge, and later proves possession of the verifier when exchanging the authorization code. This helps protect against authorization-code interception."

---

# 118. Interview Question

### "How does Spring Security validate a JWT?"

Answer:

> "The request enters Spring Security's filter chain, the bearer token is extracted, and the configured JWT decoder validates the signature and relevant claims such as issuer and expiration. Spring then creates an authenticated security context, after which authorization rules are applied."

---

# 119. Interview Question

### "How do you secure service-to-service communication?"

Answer:

> "Depending on the architecture, I'd use OAuth2 client credentials, mTLS, platform workload identity or another trusted mechanism. I'd use least-privilege scopes, validate issuer and audience, encrypt traffic and monitor authentication failures."

---

# 120. Interview Question

### "How do you store passwords?"

Answer:

> "Never in plaintext. I'd use a dedicated password hashing algorithm such as BCrypt or another modern password hashing approach, with proper salt handling and an appropriate work factor."

---

# 121. Interview Question

### "Why is JWT considered stateless?"

Answer:

> "A resource server can validate a self-contained JWT using its signature and claims without maintaining a server-side session for each user. However, token revocation and lifecycle management still need to be designed."

---

# 122. Interview Question

### "How do you revoke JWTs?"

Answer:

> "Because self-contained JWTs can remain valid until expiration, I'd normally use short-lived access tokens and control refresh tokens. For stronger immediate revocation requirements, options include token introspection, denylisting or centralized session/token state."

---

# 123. Interview Scenario

### "A user steals another user's JWT. What happens?"

Answer:

> "If the token is valid, an attacker may be able to use it until it expires or is otherwise revoked. That's why access tokens should be short-lived, transported over HTTPS, protected from logging and storage vulnerabilities, scoped narrowly and revoked when compromise is detected."

---

# 124. Interview Scenario

### "Can a JWT payload contain a password?"

Answer:

> "No. A normal signed JWT is not encrypted. Its payload can be decoded, so passwords and other secrets should never be placed there."

---

# 125. Interview Scenario

### "Can an API trust the role claim from a JWT?"

Answer:

> "Only after the token has been properly validated and the issuer and audience are trusted. The API should then map the validated claims to authorities and enforce authorization. It should never trust a client-provided role header independently."

---

# 126. Interview Scenario

### "Gateway validates JWT. Should services validate it too?"

Answer:

> "For protected services, I'd normally maintain an appropriate service-level trust boundary rather than relying blindly on the gateway. Services can validate the token or receive trusted identity from a strongly secured architecture. The exact design depends on the deployment and security model."

---

# 127. Interview Scenario

### "Why not use one JWT secret across all microservices?"

Answer:

> "Sharing one symmetric secret across many services increases the blast radius. A compromised service could potentially forge tokens for the whole system. Asymmetric signing can reduce this risk because services only need the public verification key."

---

# 128. Final Security Architecture

```text
                    Authorization Server
                         |
                    Access Tokens
                         |
                         ↓
Client
  |
 HTTPS
  |
  ↓
Load Balancer
  |
  ↓
API Gateway
  |
  | JWT validation / rate limiting
  |
  +-----------------------------+
  |              |              |
  ↓              ↓              ↓
Order         Product        Payment
Service       Service        Service
  |              |              |
  +------ OAuth2/mTLS ----------+
                 |
                 ↓
            Protected APIs
```

Core principles:

```text
Authenticate
Authorize
Encrypt
Validate
Limit
Monitor
Rotate
Revoke
```

---

# 129. The Interviewer's Real Test

If asked:

> "Design security for an e-commerce microservices system."

Think:

```text
Client
  ↓
HTTPS
  ↓
Gateway
  ↓
OAuth2/OIDC
  ↓
Access Token
  ↓
Gateway validation
  ↓
Microservice
  ↓
Issuer + Audience + Expiration validation
  ↓
Scope/Role authorization
  ↓
Resource ownership check
  ↓
Database
```

For service-to-service calls:

```text
Order
  ↓
Client Credentials / mTLS
  ↓
Inventory
```

For sensitive operations:

```text
Least privilege
+
Short-lived tokens
+
Idempotency
+
Audit logs
+
Secrets management
```

The key interview lesson is:

> **Security is not just "add JWT." Authentication, authorization, token lifecycle, service identity, transport security, secrets, and least privilege all have to work together.**
