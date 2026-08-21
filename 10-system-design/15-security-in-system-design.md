# System Design — File 15: Security in System Design

Security should be designed into the architecture rather than added at the end.

A typical backend request:

```text
Client
  |
  v
CDN / WAF
  |
  v
API Gateway / Load Balancer
  |
  v
Spring Boot Service
  |
  +----> Redis
  |
  +----> Database
  |
  +----> Other Services
```

## 1. Authentication vs Authorization

Authentication answers:

> Who are you?

Authorization answers:

> What are you allowed to do?

Example:

```text
USER  -> view products
ADMIN -> create/delete products
```

Interview answer:

> "Authentication verifies identity, while authorization determines what that identity is allowed to access."

## 2. Principle of Least Privilege

Give users and services only the permissions they need.

```text
Application
    |
    v
Dedicated DB user
    |
    v
Only required permissions
```

Apply least privilege to:

```text
Users
Services
Databases
Cloud IAM
Queues
Files
APIs
```

## 3. RBAC

Role-Based Access Control assigns permissions through roles.

```text
USER
  -> read products
  -> create orders

ADMIN
  -> read products
  -> create products
  -> delete products
```

For larger systems, fine-grained permissions can be useful:

```text
PRODUCT_READ
PRODUCT_CREATE
PRODUCT_DELETE
ORDER_REFUND
```

## 4. Password Security

Never store plaintext passwords.

Use password-specific hashing algorithms such as:

```text
Argon2
bcrypt
scrypt
```

Conceptually:

```text
Password + unique salt
        |
        v
      Hash
        |
        v
     Database
```

Do not use ordinary SHA-256 alone as a password-storage scheme.

## 5. Hashing vs Encryption

### Hashing

```text
Password -> Hash
```

One-way transformation used for password verification.

### Encryption

```text
Plaintext
   |
   v
Encrypted data
   |
   v
Decrypted data
```

Use encryption when the original value must later be recovered.

## 6. JWT

JWT means JSON Web Token.

A JWT commonly contains:

```text
Header
Payload
Signature
```

Typical flow:

```text
Login
  |
  v
Auth Server
  |
  v
JWT
  |
  v
Client
  |
  | Authorization: Bearer <JWT>
  v
API
```

JWT claims can include:

```text
userId
roles
issuer
expiration
```

## 7. JWT Is Not Automatically Encrypted

A normal JWT is encoded and signed.

The signature provides:

```text
Integrity
Authenticity
```

It does not automatically provide:

```text
Confidentiality
```

Therefore don't put sensitive secrets in ordinary JWT claims.

## 8. Access Tokens and Refresh Tokens

Common model:

```text
Access Token  -> short-lived
Refresh Token -> longer-lived
```

When the access token expires:

```text
Client
  |
  v
Refresh Token
  |
  v
Auth Server
  |
  v
New Access Token
```

Use secure refresh-token storage and rotation/revocation where appropriate.

## 9. OAuth 2.0 vs OpenID Connect

```text
OAuth 2.0 -> Authorization
OIDC      -> Authentication / Identity
```

OAuth allows controlled access to resources.

OIDC adds an identity layer on top of OAuth 2.0.

## 10. API Gateway Security

An API Gateway can provide:

```text
TLS termination
Authentication
Rate limiting
IP filtering
Request validation
Routing
Logging
```

But business-level authorization should still be enforced by the service when necessary.

## 11. HTTPS / TLS

TLS protects communication with:

```text
Confidentiality
Integrity
Server authentication
```

Use HTTPS for APIs and sensitive traffic.

For sensitive service-to-service communication, consider:

```text
mTLS
```

## 12. mTLS

Mutual TLS authenticates both sides:

```text
Client authenticates Server
Server authenticates Client
```

Useful for:

```text
Service-to-service security
Zero-trust architectures
Internal APIs
```

## 13. Zero Trust

Traditional assumption:

```text
Inside network = trusted
```

Zero Trust moves toward:

```text
Never automatically trust
Always verify
```

Verify:

```text
Identity
Service identity
Authorization
Context
```

## 14. Object-Level Authorization

Consider:

```text
GET /orders/1001
```

A user changes it to:

```text
GET /orders/1002
```

If order 1002 belongs to another user and the API returns it, authorization is broken.

Always check:

```text
Who is requesting?
What resource?
What operation?
Does the caller have permission?
```

## 15. SQL Injection

Unsafe dynamic SQL can allow user input to alter query semantics.

Better:

```text
Prepared statements
Parameterized queries
ORM parameter binding
```

Spring Data JPA and JDBC parameter binding help when used correctly.

## 16. Input Validation

Validate:

```text
Type
Length
Range
Format
Allowed values
```

Example:

```text
age = -500
```

should be rejected when outside the business domain.

## 17. XSS

Cross-Site Scripting occurs when attacker-controlled content executes in a browser.

Defenses include:

```text
Output encoding
Content Security Policy
Safe HTML handling
HttpOnly cookies for session cookies
```

## 18. CSRF

Cross-Site Request Forgery can cause a browser to send an unwanted authenticated request.

Relevant defenses include:

```text
CSRF tokens
SameSite cookies
Origin/Referer validation where appropriate
```

The exact threat model depends on how authentication is implemented.

## 19. CORS

CORS controls which browser origins can make cross-origin requests.

Example:

```text
Frontend:
https://app.example.com

API:
https://api.example.com
```

Important:

> CORS is a browser security mechanism, not an authentication mechanism.

## 20. Secure Cookies

Important attributes:

```text
Secure
HttpOnly
SameSite
```

- `Secure` -> send over HTTPS
- `HttpOnly` -> JavaScript cannot directly read the cookie
- `SameSite` -> controls cross-site cookie behavior

## 21. Session Authentication

Traditional flow:

```text
Login
  |
  v
Server creates session
  |
  v
Session ID -> Client cookie
```

For multiple application instances:

```text
API 1
API 2
API 3
   |
   v
Shared Session Store
```

Redis is a common choice.

## 22. JWT vs Session

### JWT

```text
Self-contained
Less server-side session state
Easy propagation
Revocation can be harder
```

### Session

```text
Server-side state
Easy central revocation
Requires shared state when scaled horizontally
```

Neither is universally better.

## 23. Secrets Management

Never hardcode:

```text
Database passwords
API keys
JWT signing secrets
Cloud credentials
Private keys
```

Use:

```text
Secret managers
Environment/config management
Vault-style systems
Cloud secret stores
```

## 24. Secret Rotation

Good systems support:

```text
Rotation
Expiration
Access auditing
Emergency revocation
```

Secrets should be replaceable without major code changes.

## 25. Encryption at Rest vs in Transit

### At Rest

Protect:

```text
Database
Object storage
Backups
Disks
```

### In Transit

Protect:

```text
Client -> API
Service -> Service
Application -> Database
```

Use TLS where supported.

## 26. Data Minimization

Don't collect or store information you don't need.

Prefer:

```text
Collect required data
Store only as long as necessary
Delete when appropriate
```

Less stored data means less security exposure.

## 27. PII

Personally identifiable information can include:

```text
Email
Phone number
Address
Government identifiers
```

Protect sensitive personal data with:

```text
Access controls
Encryption
Minimization
Retention policies
Auditing
```

## 28. Audit Logging

Security-sensitive actions should be auditable:

```text
Login
Password change
Role change
Admin action
Payment refund
Data export
```

An audit event may include:

```text
Actor
Action
Resource
Timestamp
Result
Correlation ID
```

Avoid logging secrets.

## 29. Rate Limiting

Rate limiting protects against:

```text
Brute-force attempts
API abuse
Traffic spikes
Resource exhaustion
```

Example:

```text
Login -> 5 attempts/minute/IP
```

Choose limits based on actual requirements.

## 30. MFA

Multi-factor authentication combines factors such as:

```text
Something you know
Something you have
Something you are
```

Example:

```text
Password
+
Authenticator code
```

## 31. Dependency Security

Monitor security of:

```text
Spring
Spring Security
Hibernate
Jackson
Database drivers
Third-party libraries
Container images
```

Useful practices:

```text
Dependency scanning
Patch management
SBOM where appropriate
```

## 32. Container Security

For Docker:

```text
Use minimal base images
Run as non-root
Keep images patched
Scan dependencies
Don't put secrets in images
Use read-only filesystems where practical
```

A container is not automatically a security boundary.

## 33. Network Segmentation

A common architecture:

```text
Internet
   |
   v
Public API
   |
   v
Private Application
   |
   v
Private Database
```

The database should normally not be directly reachable from the public internet.

## 34. API Gateway vs Service Security

A gateway can provide:

```text
Authentication
Rate limiting
TLS
Routing
```

But services should still enforce important authorization.

Do not assume:

```text
Internal network = trusted
```

## 35. SSRF

Server-Side Request Forgery tricks a server into making a request to an unintended destination.

Potential targets:

```text
Internal services
Cloud metadata endpoints
Private network resources
```

Defenses:

```text
Allowlist destinations
Validate URLs
Restrict outbound network access
Block private/internal ranges where appropriate
```

## 36. Secure File Uploads

Validate:

```text
File size
Content
Type
Filename
```

Also consider:

```text
Malware scanning
Safe filenames
Non-executable storage
Restricted access
```

Never trust a file extension alone.

## 37. Pre-Signed URLs

For object storage:

```text
Client
  |
  v
API -> temporary upload URL
  |
  v
Client -> Object Storage
```

Benefits:

```text
Less API bandwidth
Temporary access
Better scalability
```

## 38. Payment Security

Payment workflows need:

```text
Idempotency
Audit logs
Encryption
Least privilege
Strong authorization
Tokenization where appropriate
```

Never assume a timeout means a payment failed.

## 39. Microservice Security

A common architecture:

```text
                API Gateway
                     |
             Authentication
                     |
        +------------+------------+
        |            |            |
        v            v            v
      Order       Payment     Inventory
```

Each service should enforce its own important authorization boundaries.

## 40. Threat Modeling

Ask:

```text
What are we protecting?
Who can attack it?
What can an attacker access?
What happens if credentials are stolen?
What happens if a service is compromised?
```

A common framework is:

```text
STRIDE
```

## 41. STRIDE

```text
S -> Spoofing
T -> Tampering
R -> Repudiation
I -> Information Disclosure
D -> Denial of Service
E -> Elevation of Privilege
```

Use it as a structured threat-modeling checklist.

## 42. Security Checklist

```text
□ Authentication
□ Authorization
□ Least privilege
□ RBAC / permissions
□ Password hashing
□ JWT / OAuth2 / OIDC
□ Token expiration
□ Refresh-token security
□ HTTPS / TLS
□ mTLS where needed
□ WAF
□ Rate limiting
□ Input validation
□ SQL injection protection
□ XSS protection
□ CSRF protection where applicable
□ CORS configuration
□ Secure cookies
□ Secrets management
□ Secret rotation
□ Encryption at rest
□ Encryption in transit
□ PII protection
□ Audit logging
□ MFA
□ Dependency security
□ Container security
□ Network segmentation
□ Private databases
□ SSRF protection
□ Secure file uploads
□ Threat modeling
```

## 43. Interview — Authentication vs Authorization?

> "Authentication verifies who the user or service is. Authorization determines what that identity is allowed to do."

## 44. Interview — Why Hash Passwords?

> "Passwords should be stored using a password-specific hashing algorithm such as bcrypt, scrypt or Argon2. Hashing is one-way, so the application verifies a password by comparing the hash of the supplied password with the stored hash."

## 45. Interview — JWT vs Session?

> "JWTs are self-contained and can reduce server-side session state, but revocation is more complex. Sessions keep state on the server and make central revocation straightforward, but horizontally scaled applications need shared session storage."

## 46. Interview — Is JWT Encrypted?

> "Not by default. A normal JWT is encoded and signed. The signature provides integrity and authenticity, not confidentiality."

## 47. Interview — OAuth2 vs OIDC?

> "OAuth 2.0 is an authorization framework. OpenID Connect adds authentication and identity on top of OAuth 2.0."

## 48. Interview — How Would You Secure a REST API?

> "I'd use HTTPS, strong authentication, resource-level authorization, input validation, rate limiting, secure secret management and appropriate logging. I'd also protect against SQL injection, broken access control, SSRF and other common application threats."

## 49. Interview — How Do You Secure Microservices?

> "I'd use service identities, least privilege, TLS or mTLS where appropriate, authorization policies, private networking and short-lived credentials. Each service should enforce its own important authorization."

## 50. Practical Scenario — JWT Is Stolen

Consider:

```text
Short access-token TTL
Refresh-token rotation/revocation
TLS
Secure client storage
Re-authentication for sensitive actions
```

There is no universal single solution; choose according to the threat model.

## 51. Practical Scenario — User Changes /users/42 to /users/43

If the caller shouldn't access user 43, this is a broken authorization problem.

Fix:

```text
Authenticate caller
+
Check authorization for resource 43
```

Authentication alone is not enough.

## 52. Practical Scenario — Database Is Publicly Reachable

Redesign:

```text
Internet
   |
   v
API
   |
   v
Private Database
```

Use:

```text
Firewalls/security groups
Private subnets
Least-privilege credentials
TLS
```

## 53. Practical Scenario — Malicious File Upload

Use:

```text
Size limits
Content validation
Safe filenames
Non-executable storage
Malware scanning where appropriate
Restricted access
```

## 54. Practical Scenario — Internal Service Is Compromised

Limit blast radius with:

```text
Least privilege
Service identity
Network segmentation
Authorization
Short-lived credentials
Audit logs
```

## 55. One-Minute Interview Answer

### "How would you design security for a Spring Boot backend?"

> "I'd start with HTTPS, authentication and resource-level authorization. I'd use OAuth2/OIDC or JWT depending on the architecture, and keep access tokens short-lived. I'd enforce least privilege for users, services and database accounts, keep databases private, protect secrets through a secret manager, validate inputs and use parameterized queries. I'd also add rate limiting, WAF protection where appropriate, audit logging, dependency scanning and monitoring. For microservices, I'd use service identities and TLS or mTLS where the security requirements justify it."

## 56. Key Takeaway

> **Good security architecture assumes that credentials, services and networks can be compromised. Reduce the blast radius with strong authentication, explicit authorization, least privilege, encryption, secret management, network isolation, monitoring and defense in depth.**

**File 15 complete.**
