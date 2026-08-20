# Spring Boot — Security and JWT

This file covers Spring Security and JWT topics that are important for Java backend interviews and real-world REST APIs.

The goal is to understand not only authentication configuration, but also how to design secure APIs.

---

# 1. What Is Spring Security?

Spring Security is a framework for securing Spring applications.

It provides:

```text
Authentication
Authorization
Password hashing
Security filters
Method security
OAuth2
JWT resource-server support
CSRF protection
Session management
```

---

# 2. Authentication vs Authorization

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
Login
↓
Authentication

Delete product
↓
Authorization
```

---

# 3. Security Filter Chain

For a REST API, requests pass through Spring Security's filter chain before reaching the controller.

Conceptually:

```text
HTTP Request
     ↓
Security Filters
     ↓
Authentication
     ↓
Authorization
     ↓
Controller
```

Security filters can:

```text
Extract credentials
Validate authentication
Apply authorization rules
Handle security exceptions
```

---

# 4. What Is SecurityFilterChain?

Modern Spring Security configuration commonly defines a `SecurityFilterChain` bean.

Example:

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http) throws Exception {

    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers(
                "/api/auth/**"
            ).permitAll()
            .anyRequest()
            .authenticated()
        );

    return http.build();
}
```

---

# 5. Why Not Use WebSecurityConfigurerAdapter?

Modern Spring Security applications generally configure security using beans such as:

```java
SecurityFilterChain
```

instead of extending the older:

```text
WebSecurityConfigurerAdapter
```

This provides a more component-oriented configuration style.

---

# 6. Permit Public Endpoints

Example:

```java
.requestMatchers(
    "/api/auth/login",
    "/api/auth/register"
).permitAll()
```

These endpoints don't require an authenticated user.

Protect everything else:

```java
.anyRequest().authenticated()
```

---

# 7. Role-Based Authorization

Example:

```java
.requestMatchers(
    "/api/admin/**"
).hasRole("ADMIN")
```

Then:

```text
USER → denied
ADMIN → allowed
```

---

# 8. hasRole vs hasAuthority

Example:

```java
.hasRole("ADMIN")
```

Spring commonly treats this as:

```text
ROLE_ADMIN
```

Whereas:

```java
.hasAuthority("PRODUCT_DELETE")
```

checks the exact authority string.

---

# 9. Method-Level Security

Enable method security:

```java
@EnableMethodSecurity
```

Then:

```java
@PreAuthorize(
    "hasRole('ADMIN')"
)
public void deleteProduct(Long id) {
    ...
}
```

This places authorization close to the business operation.

---

# 10. Why Use Method Security?

URL-level security:

```text
/api/admin/**
```

is useful for broad access rules.

Method-level security is useful for:

```text
Specific business operations
Fine-grained permissions
Resource-level checks
```

Use the appropriate layer rather than relying on only one mechanism.

---

# 11. Password Storage

Never store:

```text
Plaintext password
```

Store:

```text
Password hash
```

Example:

```java
PasswordEncoder encoder =
    new BCryptPasswordEncoder();

String hash =
    encoder.encode(password);
```

---

# 12. PasswordEncoder

A typical configuration:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

Then:

```java
String encoded =
    passwordEncoder.encode(rawPassword);
```

For login:

```java
passwordEncoder.matches(
    rawPassword,
    encodedPassword
);
```

---

# 13. Why Hash Passwords?

If the database is compromised, plaintext passwords would immediately be exposed.

Password hashing provides a one-way transformation designed specifically for password storage.

Use a password hashing algorithm such as:

```text
bcrypt
Argon2
scrypt
```

according to project requirements.

---

# 14. What Is JWT?

JWT means:

```text
JSON Web Token
```

It is commonly used for bearer-token authentication.

Typical structure:

```text
Header.Payload.Signature
```

Example concept:

```text
xxxxx.yyyyy.zzzzz
```

---

# 15. JWT Header

The header can contain information such as:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

The exact algorithm depends on the application's configuration.

---

# 16. JWT Payload

The payload contains claims.

Example:

```json
{
  "sub": "user-101",
  "roles": ["USER"],
  "exp": 1780000000
}
```

Do not put secrets in the payload.

JWT payloads are generally readable by whoever possesses the token.

---

# 17. JWT Signature

The signature protects the token against unauthorized modification.

Conceptually:

```text
Header
+
Payload
+
Signing key
↓
Signature
```

If the payload is modified:

```text
Signature validation fails
```

---

# 18. JWT Is Not Encryption

Important:

```text
JWT signing
≠
Encryption
```

A signed JWT provides integrity/authenticity when properly validated.

The payload should not contain sensitive information simply because it is encoded.

---

# 19. JWT Authentication Flow

Typical flow:

```text
Client
  ↓
POST /login
  ↓
Credentials validated
  ↓
JWT generated
  ↓
Client stores token
  ↓
Client sends:
Authorization: Bearer <token>
  ↓
Spring Security validates token
  ↓
Authenticated request
```

---

# 20. Authorization Header

Typical request:

```http
Authorization: Bearer eyJ...
```

The backend extracts the bearer token and validates it.

---

# 21. Stateless Authentication

A bearer-token REST API can be stateless because the server does not need a traditional HTTP session for every request.

Conceptually:

```text
Request
↓
JWT
↓
Validate
↓
Authentication
```

This can make horizontal scaling easier.

---

# 22. Stateless Session Configuration

Example:

```java
http
    .sessionManagement(session ->
        session.sessionCreationPolicy(
            SessionCreationPolicy.STATELESS
        )
    );
```

Use this when the application is designed around stateless bearer-token authentication.

---

# 23. JWT Expiration

A JWT should generally have an expiration time.

Example claim:

```json
{
  "exp": 1780000000
}
```

When expired:

```text
Request
↓
JWT validation
↓
Expired
↓
401 Unauthorized
```

---

# 24. Access Token vs Refresh Token

Access token:

```text
Short-lived
Used for API requests
```

Refresh token:

```text
Longer-lived
Used to obtain a new access token
```

This can reduce the lifetime of bearer access credentials.

The exact storage and rotation strategy depends on the application.

---

# 25. Refresh Token Rotation

A stronger refresh-token design can rotate refresh tokens:

```text
Refresh Token A
      ↓
New Access Token
      +
Refresh Token B
      ↓
Invalidate A
```

This helps detect and limit replay of stolen refresh tokens.

---

# 26. JWT Secret Key

For symmetric signing:

```text
HMAC
```

the same secret is used for signing and verification.

Example algorithms:

```text
HS256
HS384
HS512
```

Keep secrets outside source code.

---

# 27. RSA JWT

Asymmetric signing uses:

```text
Private key → sign
Public key  → verify
```

Example:

```text
RS256
```

This can be useful when multiple services need to verify tokens without receiving the private signing key.

---

# 28. JWT Key Rotation

Production systems should support key rotation.

Conceptually:

```text
Old key → verify existing tokens
New key → sign new tokens
```

During a transition, multiple verification keys may need to remain available.

---

# 29. JWT Claims

Common claims include:

```text
iss → issuer
sub → subject
aud → audience
exp → expiration
iat → issued-at
nbf → not-before
jti → token ID
```

Don't trust arbitrary claims without proper token validation.

---

# 30. What Should Be in a JWT?

Usually keep claims minimal:

```text
User identifier
Authorities/roles where appropriate
Issuer
Audience
Expiration
Issued-at
Token ID where useful
```

Avoid putting:

```text
Password
Credit card details
Secrets
Large user profiles
Sensitive personal data
```

---

# 31. AuthenticationManager

In traditional username/password authentication, an `AuthenticationManager` coordinates authentication.

Conceptually:

```text
Username + Password
        ↓
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
UserDetailsService / Identity Store
        ↓
Password verification
```

---

# 32. UserDetailsService

`UserDetailsService` loads user information for username/password authentication.

Example:

```java
@Service
public class CustomUserDetailsService
        implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(
            String username) {

        ...
    }
}
```

---

# 33. UserDetails

Spring Security's `UserDetails` represents authenticated user information.

It can contain:

```text
Username
Password
Authorities
Account status
```

---

# 34. AuthenticationProvider

An `AuthenticationProvider` knows how to authenticate a particular type of credentials.

Example:

```text
UsernamePasswordAuthenticationToken
        ↓
DaoAuthenticationProvider
        ↓
UserDetailsService
        ↓
PasswordEncoder
```

---

# 35. BCrypt Password Verification

Do not do:

```java
rawPassword.equals(
    storedPassword
);
```

Instead:

```java
passwordEncoder.matches(
    rawPassword,
    storedHash
);
```

---

# 36. Custom JWT Filter

A custom JWT filter can:

```text
Read Authorization header
↓
Extract Bearer token
↓
Validate token
↓
Extract identity/authorities
↓
Create Authentication
↓
Set SecurityContext
```

However, don't create custom security infrastructure unnecessarily when Spring Security's built-in resource-server support meets the requirement.

---

# 37. SecurityContext

The `SecurityContext` contains the current authentication information.

Conceptually:

```text
Request
 ↓
Authentication
 ↓
SecurityContext
 ↓
Controller / Service
```

Code can access:

```java
SecurityContextHolder
```

when appropriate.

---

# 38. Authentication Object

After successful authentication, Spring Security can provide:

```text
Principal
Authorities
Authentication state
```

Example:

```java
Authentication authentication =
    SecurityContextHolder
        .getContext()
        .getAuthentication();
```

Avoid coupling business logic unnecessarily to `SecurityContextHolder`; pass explicit user identifiers where that improves design and testability.

---

# 39. JWT Resource Server

For applications that receive and validate JWTs issued by an identity provider, Spring Security can be configured as an OAuth2 Resource Server.

Conceptually:

```text
Identity Provider
       ↓
JWT
       ↓
Spring Boot Resource Server
       ↓
Validate signature/claims
       ↓
Authorization
```

---

# 40. JWT Validation

Validation should include appropriate checks such as:

```text
Signature
Expiration
Issuer
Audience where required
Not-before where applicable
Algorithm
```

Do not validate only the token structure.

---

# 41. Token Expiration vs Revocation

Expiration:

```text
Token becomes invalid after time T
```

Revocation:

```text
Token becomes invalid before expiration
```

JWTs are often easier to validate statelessly than to revoke immediately.

Possible strategies:

```text
Short access-token lifetime
Refresh-token rotation
Token denylist where necessary
Session/version checks
Identity-provider revocation
```

---

# 42. JWT Logout

With purely stateless access tokens, logout is not simply:

```text
Delete server session
```

because there may be no session.

A practical design can use:

```text
Short-lived access token
+
Refresh-token revocation
```

and additional controls where required.

---

# 43. CSRF

CSRF is mainly a concern when browsers automatically attach authentication credentials such as cookies.

For a stateless API using bearer tokens sent explicitly in the `Authorization` header, the CSRF threat model is different.

Don't blindly disable or enable CSRF without understanding how authentication credentials are transported.

---

# 44. CORS

CORS controls which browser origins may make cross-origin requests.

Example concept:

```text
Frontend
https://app.example.com
        ↓
Backend
https://api.example.com
```

Configure only trusted origins.

---

# 45. CORS vs CSRF

CORS:

```text
Browser cross-origin access policy
```

CSRF:

```text
Protection against unwanted authenticated actions
```

They solve different problems.

---

# 46. Security Headers

Useful security headers can include:

```text
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
Strict-Transport-Security
```

Exact headers depend on application architecture.

Spring Security can help configure several of them.

---

# 47. HTTPS

Never send credentials over plain HTTP in production.

Use:

```text
HTTPS
TLS
```

TLS protects credentials and tokens while in transit.

---

# 48. Secrets Management

Do not put:

```text
JWT secret
Database password
API key
Private key
```

directly in Git.

Use:

```text
Environment variables
Secret manager
Vault
Cloud secret services
Kubernetes Secrets
CI/CD secret storage
```

with appropriate access controls.

---

# 49. Least Privilege

A service should have only the permissions it needs.

Example:

```text
Order Service
→ order DB permissions

Not:
→ unrestricted access to every database
```

This limits blast radius.

---

# 50. Resource-Level Authorization

Role checks alone may not be enough.

Example:

```text
USER
GET /orders/100
```

The user may have the `USER` role but should only access:

```text
Their own order
```

Authorization should check ownership when required.

---

# 51. IDOR / BOLA

Insecure Direct Object Reference / Broken Object Level Authorization.

Bad:

```text
GET /users/101/orders
```

and simply returning the data because the user is authenticated.

Good:

```text
Authenticated user
+
Resource ownership/permission check
```

---

# 52. Mass Assignment

Avoid binding untrusted JSON directly to sensitive entities.

Bad:

```java
@PostMapping
public User create(
    @RequestBody User user
) {
    ...
}
```

The request might attempt to set:

```text
isAdmin
enabled
roles
```

Use:

```text
CreateUserRequest DTO
```

instead.

---

# 53. Input Validation

Validate:

```text
Required fields
Length
Format
Range
Business constraints
```

Example:

```java
@NotBlank
@Size(max = 100)
private String name;
```

Validation is not a replacement for authorization.

---

# 54. SQL Injection

Use:

```text
JPA parameters
Prepared statements
Parameterized queries
```

Avoid:

```java
"SELECT * FROM users WHERE name='"
    + name
    + "'"
```

---

# 55. Password Reset

A secure reset flow typically uses:

```text
Random single-use token
Expiration
HTTPS
Minimal user disclosure
Password hashing
Token invalidation after use
```

Don't return:

```text
"Email does not exist"
```

if that would allow account enumeration.

---

# 56. Account Enumeration

Bad:

```text
user@example.com → account exists
unknown@example.com → account doesn't exist
```

This reveals which accounts exist.

Prefer generic responses where appropriate:

```text
"If the account exists, a reset link has been sent."
```

---

# 57. Brute Force Protection

Protect login endpoints with mechanisms such as:

```text
Rate limiting
Progressive delays
Monitoring
MFA
Account protection
```

Avoid permanently locking accounts based only on simplistic thresholds because that can enable denial-of-service against users.

---

# 58. Rate Limiting

Example:

```text
Login
→ 5 attempts/minute/IP
```

or a policy based on:

```text
User
IP
API key
Client
Endpoint
```

Return:

```text
429 Too Many Requests
```

when appropriate.

---

# 59. Security Logging

Log useful security events:

```text
Login failure
Privilege change
Password reset
Suspicious access
Authentication failure
```

Do not log:

```text
Passwords
JWT tokens
API secrets
Private keys
Sensitive personal data
```

---

# 60. Audit Logging

For important actions:

```text
Who
What
When
Resource
Result
Correlation ID
```

Example:

```text
admin-101
DELETE_PRODUCT
product-500
2026-08-20T10:00:00Z
SUCCESS
```

Keep audit logs protected from unauthorized modification.

---

# 61. Security Headers and APIs

For REST APIs, security should include more than authentication:

```text
TLS
Authentication
Authorization
Input validation
Output control
Rate limiting
Security headers where relevant
Secret management
Audit logging
Dependency updates
```

---

# 62. Dependency Security

Keep dependencies updated.

Check for:

```text
Known CVEs
Outdated libraries
Transitive vulnerabilities
Unmaintained dependencies
```

Tools can include:

```text
OWASP Dependency-Check
Snyk
GitHub Dependabot
Trivy
```

Use the tools supported by your organization.

---

# 63. Secure Error Responses

Bad:

```json
{
  "error": "SQLSyntaxErrorException at database table users..."
}
```

Better:

```json
{
  "status": 500,
  "message": "Internal server error"
}
```

Detailed stack traces belong in controlled logs, not public API responses.

---

# 64. Authentication Failure Response

Avoid revealing too much detail.

Bad:

```text
"User exists but password is wrong."
```

Better:

```text
"Invalid username or password."
```

This reduces account enumeration.

---

# 65. Authorization Failure

If the user is authenticated but lacks permission:

```text
403 Forbidden
```

The API should not expose unnecessary internal authorization details.

---

# 66. Token Theft

If an access token is stolen, an attacker may use it until it expires or is otherwise invalidated.

Reduce risk with:

```text
Short token lifetime
Secure transport
Secure storage
Refresh-token rotation
Audience validation
Issuer validation
Monitoring
```

---

# 67. Browser Token Storage

There is no universal storage answer.

Common considerations:

```text
HttpOnly cookies
Secure cookies
SameSite policy
In-memory storage
Browser storage
```

The correct design depends on the application's architecture and threat model.

Do not casually store long-lived sensitive tokens in browser-accessible storage.

---

# 68. Secure Cookie

For cookie-based authentication, common flags include:

```text
Secure
HttpOnly
SameSite
```

These reduce certain browser-side attack risks when configured appropriately.

---

# 69. SameSite

Cookie `SameSite` controls when browsers send cookies in cross-site contexts.

Common values:

```text
Strict
Lax
None
```

`None` requires `Secure`.

The correct setting depends on application requirements.

---

# 70. OAuth2

OAuth 2.0 is an authorization framework.

Common actors:

```text
Resource Owner
Client
Authorization Server
Resource Server
```

Example:

```text
User
 ↓
Identity Provider
 ↓
Access Token
 ↓
Spring Boot API
```

---

# 71. OpenID Connect

OIDC builds an identity layer on top of OAuth 2.0.

OAuth 2.0:

```text
Authorization
```

OIDC:

```text
Authentication + identity
```

Commonly used with:

```text
Keycloak
Okta
Auth0
Microsoft Entra ID
Other identity providers
```

---

# 72. Authorization Code Flow

A browser-based application can use:

```text
User
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Client
 ↓
Token endpoint
 ↓
Access token
```

PKCE is commonly used for public clients.

---

# 73. PKCE

PKCE protects the authorization code flow against certain authorization-code interception attacks.

Conceptually:

```text
Code Verifier
     ↓
Code Challenge
     ↓
Authorization Request
```

Then the verifier is presented when exchanging the authorization code.

---

# 74. OAuth2 Resource Server

A Spring Boot API can act as a resource server:

```text
Client
 ↓
Access Token
 ↓
Spring Boot API
 ↓
Validate JWT
 ↓
Authorize
 ↓
Resource
```

This is often cleaner than implementing an entire custom authentication system.

---

# 75. Identity Provider

An identity provider can handle:

```text
User authentication
Token issuance
Key management
Password policies
MFA
Federation
```

Examples:

```text
Keycloak
Okta
Auth0
Microsoft Entra ID
```

---

# 76. When Should You Build Authentication Yourself?

For many production systems:

```text
Use established identity provider
```

rather than implementing:

```text
Password reset
MFA
Token issuance
Key rotation
Account recovery
Brute-force protection
```

yourself.

Custom authentication has a large security maintenance burden.

---

# 77. Spring Security Request Flow

A simplified flow:

```text
HTTP Request
      ↓
SecurityFilterChain
      ↓
Extract credentials
      ↓
Authenticate
      ↓
SecurityContext
      ↓
Authorization
      ↓
Controller
      ↓
Service
      ↓
Repository
```

---

# 78. JWT Resource Server Flow

```text
Client
  |
  | Authorization: Bearer JWT
  v
Spring Security
  |
  +--> Validate signature
  |
  +--> Validate claims
  |
  +--> Build Authentication
  |
  v
Authorization
  |
  v
Controller
```

---

# 79. Security Scenario: User Accesses Another User's Order

Request:

```text
GET /api/orders/500
```

Authentication:

```text
Valid
```

Authorization:

```text
Order belongs to user 101
Current user = 202
```

Result:

```text
403 Forbidden
```

or another intentionally chosen response depending on whether the API wants to reveal resource existence.

---

# 80. Security Scenario: Admin Endpoint

Endpoint:

```text
DELETE /api/products/100
```

Rules:

```text
Unauthenticated → 401
USER → 403
ADMIN → allowed
```

---

# 81. Security Scenario: Expired Token

```text
Request
 ↓
Bearer token
 ↓
exp check
 ↓
Expired
 ↓
401
```

Client can obtain a fresh access token through the configured authentication flow.

---

# 82. Security Scenario: Invalid Signature

```text
Token payload changed
↓
Signature mismatch
↓
Authentication fails
↓
401
```

Never accept a token just because its payload can be decoded.

---

# 83. Security Scenario: Wrong Audience

Token:

```text
aud = service-A
```

Request:

```text
service-B
```

If the resource server requires:

```text
aud = service-B
```

validation should fail.

---

# 84. Security Scenario: Wrong Issuer

If the API trusts:

```text
https://identity.example.com
```

but the token was issued by:

```text
https://attacker.example.com
```

issuer validation should reject it.

---

# 85. Security Scenario: Secret Leaked

If a JWT signing secret is exposed:

```text
Rotate secret/key
Invalidate affected credentials where possible
Investigate exposure
Review logs
Deploy updated configuration
```

Do not just hide the secret again and assume the incident is over.

---

# 86. Security Scenario: Database Password Leaked

Actions:

```text
Rotate credential
↓
Update secret manager
↓
Restart/redeploy services
↓
Review database access logs
↓
Investigate source of leak
```

---

# 87. Security Scenario: Vulnerable Dependency

Process:

```text
Detect CVE
↓
Assess exploitability
↓
Upgrade dependency
↓
Run tests
↓
Security scan
↓
Deploy
↓
Monitor
```

If no fixed version exists:

```text
Mitigate
+
Track remediation
```

---

# 88. Security Testing Checklist

```text
□ Authentication
□ Authorization
□ Role checks
□ Resource ownership
□ JWT signature
□ JWT expiration
□ Issuer
□ Audience
□ Password hashing
□ Input validation
□ SQL injection
□ CORS
□ CSRF threat model
□ Rate limiting
□ Sensitive data exposure
□ Secret management
□ Dependency vulnerabilities
□ Secure error responses
□ Audit logging
```

---

# 89. Interview: Explain Spring Security

> Spring Security provides authentication and authorization for Spring applications. In a REST API, requests pass through the security filter chain, credentials are validated, an authenticated principal is established, and authorization rules determine whether the request can access the endpoint.

---

# 90. Interview: Explain JWT

> JWT is a signed token commonly used for bearer authentication. The client sends it with each request, and the resource server validates the signature and important claims such as expiration, issuer, and audience before establishing authentication.

---

# 91. Interview: Is JWT Encrypted?

> Not by default. A normal signed JWT provides integrity and authenticity, but its payload can generally be decoded. I never put passwords or sensitive secrets in JWT claims unless an appropriate encryption mechanism is deliberately used.

---

# 92. Interview: Why Use JWT?

> JWT can support stateless API authentication, which makes horizontal scaling easier because the server doesn't need to maintain a traditional session for every request. The tradeoff is that token expiration, revocation, refresh, and key management need to be designed carefully.

---

# 93. Interview: 401 vs 403

> 401 means the request isn't successfully authenticated. 403 means the user is authenticated but doesn't have permission to perform the operation.

---

# 94. Interview: How Do You Secure Passwords?

> I never store plaintext passwords. I use a strong password hashing algorithm through Spring Security's `PasswordEncoder`, and I compare passwords using the encoder's verification method.

---

# 95. Interview: How Do You Secure JWTs?

> I keep access tokens short-lived, validate signature and important claims, use HTTPS, protect signing keys, implement refresh-token controls where needed, and avoid putting sensitive information into the token.

---

# 96. Interview: How Do You Handle Authorization?

> I use broad endpoint rules for common access control and method-level or resource-level authorization for sensitive business operations. For example, an authenticated user may access an order only if that order belongs to them.

---

# 97. Interview: How Do You Prevent SQL Injection?

> I use parameterized queries, Spring Data JPA query parameters, or prepared statements. I never concatenate untrusted user input directly into SQL.

---

# 98. Interview: How Do You Secure an Ecommerce Backend?

> I would use HTTPS, Spring Security, strong password hashing, JWT or an established identity provider, role and resource-level authorization, DTOs, input validation, secure error responses, rate limiting for sensitive endpoints, secret management, audit logging, and dependency security scanning.

---

# 99. Interview: How Would You Secure Admin APIs?

> I would require authentication and explicit admin authority, and for sensitive operations I would also validate resource-level permissions and audit the action. I wouldn't rely only on hiding the endpoint from the UI.

---

# 100. Interview: How Do You Protect Against Duplicate Payments?

> I would use an idempotency key for the payment operation and enforce uniqueness at the appropriate persistence layer. The payment workflow should also have controlled retries and a clear reconciliation strategy.

---

# 101. Interview: How Would You Handle a Stolen JWT?

> I would limit the access token lifetime, use secure transport, rotate and revoke refresh credentials where applicable, monitor suspicious activity, and rotate signing keys if the signing key itself has been compromised.

---

# 102. Interview: Should JWT Contain User Roles?

> Roles can be included when they are appropriate and trustworthy, but I still validate the token properly and design authorization around the application's security model. I keep claims minimal and avoid putting sensitive information in the token.

---

# 103. Interview: JWT vs Session

JWT:

```text
Stateless bearer credential
Easy horizontal scaling
Revocation requires additional design
```

Session:

```text
Server-side state
Straightforward revocation
Requires session management
```

The right choice depends on the application architecture.

---

# 104. Interview: What Is OAuth2?

> OAuth 2.0 is an authorization framework that allows a client to obtain access to protected resources using access tokens. For user identity and authentication, OpenID Connect builds an identity layer on top of OAuth 2.0.

---

# 105. Interview: What Is an OAuth2 Resource Server?

> A resource server hosts protected APIs and validates access tokens presented by clients. In Spring Boot, Spring Security can validate JWT bearer tokens and then apply authorization rules.

---

# 106. Interview: Why Use Keycloak or Another Identity Provider?

> An identity provider can handle difficult security capabilities such as login, token issuance, MFA, password policies, key management, and account recovery. Using a mature identity provider can reduce the amount of security-sensitive code the application has to maintain.

---

# 107. Interview: How Do You Handle Security in Microservices?

> I would normally centralize identity through an identity provider or authorization service, validate bearer tokens at the appropriate service boundary, apply service-level and resource-level authorization, use HTTPS internally where required, manage secrets securely, and add audit logs and observability.

---

# 108. Final Security Mental Model

```text
                 SECURITY
                    |
       +------------+------------+
       |            |            |
 Authentication Authorization   Data
       |            |            |
 Password       Roles          Encryption
 JWT            Permissions    TLS
 OAuth2         Ownership      Hashing
       |
       +------------------------+
                    |
              Secure APIs
                    |
       Validation + Logging
       Rate Limiting + Secrets
```

---

# 109. Final Security Rule

> **Authentication answers "who are you?" Authorization answers "what can you do?" A secure backend needs both, plus safe data handling, strong credential management, validation, monitoring, and a clear threat model.**
