# REST API — Security, Authentication & Authorization

This file covers the core security concepts needed to build secure Spring Boot REST APIs, including authentication, authorization, JWT, password hashing, CORS, CSRF, security filters, roles, permissions, and common interview scenarios.

---

# 1. Why REST API Security Matters

A REST API may expose:

```text
User data
Products
Orders
Payments
Admin operations
Internal business logic
```

Security must protect:

```text
Confidentiality
Integrity
Availability
Authentication
Authorization
```

A secure API should assume that every client request can be manipulated.

---

# 2. Authentication vs Authorization

These are different concepts.

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
User logs in
      ↓
Authentication
      ↓
JWT issued
      ↓
Request /api/admin/products
      ↓
Authorization
      ↓
ADMIN?
```

---

# 3. Authentication

Authentication verifies the identity of a client.

Common approaches:

```text
Username/password
Session
JWT
OAuth 2.0
OpenID Connect
API keys
mTLS
```

The correct approach depends on the application and trust model.

---

# 4. Authorization

Authorization determines whether an authenticated identity can perform an operation.

Example:

```text
USER
 ├── GET /products
 ├── GET /orders
 └── POST /orders

ADMIN
 ├── POST /products
 ├── PUT /products/{id}
 └── DELETE /products/{id}
```

---

# 5. Spring Security

Spring Security provides a framework for:

```text
Authentication
Authorization
Security filters
Password encoding
Session management
OAuth2
JWT resource server support
CSRF
CORS integration
Security headers
```

A Spring Boot application can configure these rules centrally.

---

# 6. Security Filter Chain

A simplified request flow:

```text
Client
  ↓
HTTP Request
  ↓
Security Filter Chain
  ↓
Authentication
  ↓
Authorization
  ↓
Controller
  ↓
Service
```

Security filters can inspect the request before it reaches the controller.

---

# 7. SecurityFilterChain

Modern Spring Security configuration commonly uses a `SecurityFilterChain` bean.

Example:

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http) throws Exception {

    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**")
            .permitAll()
            .anyRequest()
            .authenticated()
        );

    return http.build();
}
```

The exact configuration depends on the application's security model.

---

# 8. Public vs Protected Endpoints

Typical configuration:

```text
/api/auth/login       → public
/api/auth/register    → public
/api/products         → authenticated
/api/orders           → authenticated
/api/admin/**         → ADMIN
```

The principle is:

```text
Allow only what should be public.
Protect everything else.
```

---

# 9. HTTP Status Codes

Security-related responses commonly include:

```text
401 Unauthorized
403 Forbidden
```

Important distinction:

```text
401 → authentication is missing or invalid
403 → authentication exists, but access is denied
```

---

# 10. 401 Unauthorized

Example:

```text
GET /api/orders
Authorization: missing
```

Response:

```http
401 Unauthorized
```

The client needs valid authentication.

---

# 11. 403 Forbidden

Example:

```text
Authenticated USER
        ↓
DELETE /api/products/100
        ↓
Requires ADMIN
        ↓
403 Forbidden
```

The identity is known but lacks sufficient permission.

---

# 12. Password Storage

Never store passwords as plain text.

Bad:

```text
password = "mypassword123"
```

Instead store a secure password hash.

Example:

```text
$2a$...
```

with a strong password hashing algorithm.

---

# 13. BCrypt

BCrypt is commonly used for password hashing.

Example:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

Password creation:

```java
String hash =
    passwordEncoder.encode(password);
```

Verification:

```java
passwordEncoder.matches(
    rawPassword,
    storedHash
);
```

---

# 14. Why Hash Instead of Encrypt?

Encryption is reversible:

```text
Plaintext
   ↓
Encryption
   ↓
Ciphertext
   ↓
Decryption
   ↓
Plaintext
```

Password hashing is designed to be one-way:

```text
Password
   ↓
Hash
```

You verify by hashing/checking the supplied password against the stored hash rather than decrypting the stored value.

---

# 15. Salt

Password hashing should use a unique salt.

A salt helps prevent attackers from efficiently using precomputed hashes against many accounts.

Modern password-hashing algorithms such as BCrypt handle salting as part of their design.

---

# 16. Password Hashing Principle

The application should do:

```text
User enters password
        ↓
PasswordEncoder
        ↓
Compare with stored hash
```

Never:

```text
Decrypt stored password
```

because passwords should not be stored as reversible encrypted values.

---

# 17. Login Flow

Typical username/password login:

```text
Client
  ↓
POST /api/auth/login
  ↓
Validate credentials
  ↓
Load user
  ↓
Compare password hash
  ↓
Generate access token
  ↓
Return token
```

---

# 18. JWT

JWT means:

```text
JSON Web Token
```

A JWT can carry claims about an authenticated identity.

Typical structure:

```text
header.payload.signature
```

Example:

```text
xxxxx.yyyyy.zzzzz
```

---

# 19. JWT Header

The header commonly contains:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

The exact algorithm depends on the security architecture.

---

# 20. JWT Payload

The payload contains claims.

Example:

```json
{
  "sub": "user123",
  "roles": ["USER"],
  "iat": 1724230000,
  "exp": 1724233600
}
```

Do not put sensitive secrets into the payload.

JWT payloads are normally encoded, not encrypted.

---

# 21. JWT Signature

The signature protects the token from undetected modification.

Conceptually:

```text
Header
+
Payload
+
Secret/private key
      ↓
Signature
```

The server verifies the signature before trusting the claims.

---

# 22. JWT Is Not Encryption

Important:

```text
JWT ≠ encrypted data
```

A signed JWT provides integrity/authenticity of the token when verified correctly.

If confidential information must be protected from token readers, use appropriate encryption mechanisms rather than assuming signing hides the payload.

---

# 23. JWT Claims

Common registered claims include:

```text
iss → issuer
sub → subject
aud → audience
exp → expiration
iat → issued-at
nbf → not-before
jti → token identifier
```

Custom claims can also be added when appropriate.

---

# 24. JWT Authentication Request

The client commonly sends:

```http
Authorization: Bearer <token>
```

Example:

```http
GET /api/orders
Authorization: Bearer eyJ...
```

---

# 25. Stateless Authentication

A JWT-based API can be stateless when the server does not need to maintain a login session for every request.

Flow:

```text
Request
 ↓
JWT
 ↓
Validate token
 ↓
Build Authentication
 ↓
Authorize
 ↓
Controller
```

However, supporting refresh tokens, revocation, or other server-side state can introduce state into parts of the overall authentication system.

---

# 26. JWT Filter

A custom JWT filter may:

```text
Read Authorization header
        ↓
Extract token
        ↓
Validate token
        ↓
Extract identity/authorities
        ↓
Create Authentication
        ↓
Store it in SecurityContext
```

This is the basic idea behind many custom JWT implementations.

---

# 27. SecurityContext

Spring Security stores the current authenticated identity in the security context.

Conceptually:

```text
JWT
 ↓
Authentication
 ↓
SecurityContext
 ↓
Controller
```

A controller can then access the authenticated principal.

---

# 28. Authentication Object

The authentication object can contain:

```text
Principal
Authorities
Authentication status
Details
```

Example:

```java
Authentication authentication =
    SecurityContextHolder
        .getContext()
        .getAuthentication();
```

---

# 29. Roles and Authorities

Spring Security distinguishes between roles and authorities.

Example authority:

```text
PRODUCT_READ
```

Example role:

```text
ROLE_ADMIN
```

Role-based checks often use:

```java
hasRole("ADMIN")
```

which corresponds to the conventional `ROLE_ADMIN` authority prefix.

---

# 30. hasRole vs hasAuthority

Example:

```java
.hasRole("ADMIN")
```

typically checks:

```text
ROLE_ADMIN
```

Whereas:

```java
.hasAuthority("PRODUCT_DELETE")
```

checks that exact authority.

Use roles for broad categories and authorities/permissions for fine-grained access where appropriate.

---

# 31. Method-Level Security

Spring Security can protect individual methods.

Enable it with:

```java
@EnableMethodSecurity
```

Then:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteProduct(Long id) {
    ...
}
```

---

# 32. @PreAuthorize

Example:

```java
@PreAuthorize(
    "hasAuthority('PRODUCT_DELETE')"
)
public void deleteProduct(Long id) {
    ...
}
```

The expression is evaluated before the method executes.

---

# 33. @PostAuthorize

`@PostAuthorize` evaluates authorization after a method returns.

Example:

```java
@PostAuthorize(
    "returnObject.ownerId == authentication.name"
)
public OrderResponse getOrder(Long id) {
    ...
}
```

Use carefully because the method has already executed.

---

# 34. Resource Ownership

Role checks are not always enough.

Example:

```text
USER A
 ↓
GET /api/orders/100
```

The system should verify:

```text
Order 100 belongs to USER A
```

This is object-level authorization.

---

# 35. IDOR

IDOR means:

```text
Insecure Direct Object Reference
```

Example vulnerability:

```http
GET /api/orders/100
```

works for any authenticated user simply by changing:

```text
100 → 101
```

The API must check ownership or permission before returning the resource.

---

# 36. CORS

CORS means:

```text
Cross-Origin Resource Sharing
```

Browsers enforce same-origin restrictions.

Example:

```text
Frontend:
https://shop.example.com

API:
https://api.example.com
```

The browser may require appropriate CORS headers.

---

# 37. CORS Configuration

Conceptually:

```java
@Bean
CorsConfigurationSource corsConfigurationSource() {

    CorsConfiguration configuration =
        new CorsConfiguration();

    configuration.setAllowedOrigins(
        List.of("https://shop.example.com")
    );

    configuration.setAllowedMethods(
        List.of("GET", "POST", "PUT", "DELETE")
    );

    configuration.setAllowedHeaders(
        List.of("Authorization", "Content-Type")
    );

    return ...
}
```

Avoid using `*` casually in production.

---

# 38. Preflight Request

Browsers may send:

```http
OPTIONS /api/orders
```

before the actual request.

This is called a preflight request.

The server needs to respond with appropriate CORS headers when the request requires preflight handling.

---

# 39. CORS vs CSRF

These are different.

CORS:

```text
Browser cross-origin access policy
```

CSRF:

```text
Attack where a victim's browser is tricked into sending an authenticated request
```

Do not treat CORS as a replacement for CSRF protection.

---

# 40. CSRF

CSRF means:

```text
Cross-Site Request Forgery
```

It is especially relevant to browser applications using automatically attached credentials such as cookies.

---

# 41. CSRF and Stateless APIs

For APIs using:

```text
Authorization: Bearer JWT
```

with tokens not automatically attached by the browser, CSRF risk can differ from traditional cookie-based session authentication.

Whether CSRF protection should be enabled or disabled depends on how credentials are transported.

Do not disable CSRF merely because the application is called a REST API.

---

# 42. Session-Based Authentication

Traditional session flow:

```text
Login
 ↓
Server creates session
 ↓
Session ID cookie
 ↓
Browser sends cookie
 ↓
Server loads session
```

The server maintains session state.

---

# 43. JWT vs Session

Session:

```text
Server-side state
Cookie
Easy revocation
Needs session management
```

JWT:

```text
Token-based
Can be stateless
Claims carried by token
Revocation requires additional design
```

Neither is universally better.

---

# 44. Access Token and Refresh Token

A common architecture uses:

```text
Short-lived access token
+
Longer-lived refresh token
```

Example:

```text
Access token → 15 minutes
Refresh token → days/weeks
```

Exact lifetimes should be based on security requirements.

---

# 45. Refresh Flow

```text
Access token expires
        ↓
POST /auth/refresh
        ↓
Validate refresh token
        ↓
Issue new access token
```

The refresh token should be protected carefully.

---

# 46. Refresh Token Storage

Depending on architecture, refresh tokens may be:

```text
Stored server-side
Stored as hashed values
Stored in secure cookies
Associated with a device/session
```

Avoid treating long-lived bearer credentials casually.

---

# 47. Token Expiration

Short-lived access tokens reduce the impact of token theft.

Example:

```text
Access token
    ↓
15 minutes
    ↓
Expires
```

But shorter lifetimes increase refresh frequency.

Security and usability must be balanced.

---

# 48. Token Revocation

JWTs are often difficult to revoke immediately because they are self-contained.

Possible strategies:

```text
Short expiration
Refresh token rotation
Token denylist
jti tracking
Session/version checks
```

Choose according to risk and operational needs.

---

# 49. JWT Key Management

Never hardcode production secrets directly in source code.

Bad:

```java
private static final String SECRET =
    "my-production-secret";
```

Better:

```text
Secret manager
Environment configuration
Key management service
Secure deployment configuration
```

---

# 50. HMAC vs RSA/EC

Symmetric signing:

```text
HS256
```

uses one shared secret.

Asymmetric signing:

```text
RS256
ES256
```

uses:

```text
Private key → signing
Public key  → verification
```

Asymmetric keys can be useful when multiple services need to verify tokens without receiving the private signing key.

---

# 51. JWT Audience and Issuer

A resource server should not blindly accept any correctly signed JWT.

Where appropriate, validate:

```text
issuer
audience
expiration
not-before
signature
algorithm
```

This helps ensure the token was issued for the expected security context.

---

# 52. Algorithm Confusion

The server should explicitly configure acceptable algorithms.

Do not trust an algorithm value from an untrusted token without enforcing an allowed configuration.

---

# 53. Bearer Token Security

Bearer token means:

```text
Whoever possesses the token
may be able to use it.
```

Therefore:

```text
Use HTTPS
Short expiration
Secure storage
Avoid logging tokens
Rotate/refresh safely
```

---

# 54. HTTPS

Authentication credentials and tokens should be transmitted over:

```text
HTTPS
```

not plain HTTP.

HTTPS protects data in transit from network-level interception.

---

# 55. Security Headers

Useful HTTP security headers can include:

```text
Content-Security-Policy
X-Content-Type-Options
Strict-Transport-Security
Referrer-Policy
```

The exact headers depend on the application and clients.

Spring Security can help configure security headers.

---

# 56. SQL Injection

Never build SQL using raw user input.

Bad:

```java
String sql =
    "SELECT * FROM users WHERE name = '"
    + username
    + "'";
```

Use:

```text
Prepared statements
JPA
Spring Data repositories
Parameterized queries
```

---

# 57. JPA and SQL Injection

Spring Data JPA reduces many SQL injection risks when using parameterized repository methods.

For custom queries:

```java
@Query(
    "select p from Product p where p.name = :name"
)
List<Product> findByName(
    @Param("name") String name
);
```

Use parameters rather than string concatenation.

---

# 58. Input Validation

Validate incoming requests.

Example:

```java
public record ProductRequest(

    @NotBlank
    String name,

    @Positive
    BigDecimal price

) {}
```

Validation protects both:

```text
Business rules
Application robustness
```

It is not a replacement for authorization.

---

# 59. Mass Assignment

Do not blindly bind client JSON to sensitive domain entities.

Bad:

```text
Client sends:
{
  "name": "Laptop",
  "price": 100,
  "role": "ADMIN"
}
```

If the entity is bound directly, the client may attempt to modify fields it should not control.

Prefer:

```text
Request DTO
   ↓
Service
   ↓
Entity
```

---

# 60. DTOs and Security

DTOs help control the API boundary.

Example:

```text
ProductRequest
 ├── name
 ├── price
 └── category
```

The entity may contain:

```text
id
createdAt
internalStatus
owner
```

Only explicitly allowed fields are mapped.

---

# 61. Sensitive Data in Responses

Do not return fields such as:

```text
Password hash
Internal tokens
Secret keys
Unnecessary personal information
Internal database identifiers when inappropriate
```

Use response DTOs.

---

# 62. Logging Security

Never log:

```text
Passwords
JWTs
Refresh tokens
API secrets
Credit card information
Sensitive personal data
```

Instead log safe identifiers and structured metadata.

---

# 63. Rate Limiting

Rate limiting protects APIs from:

```text
Abuse
Brute-force login
Excessive traffic
Resource exhaustion
```

Example:

```text
POST /auth/login
5 attempts/minute
```

A distributed implementation can use Redis.

---

# 64. Brute-Force Protection

For login endpoints consider:

```text
Rate limiting
Account lockout policies
Progressive delays
CAPTCHA where appropriate
Monitoring
Alerting
```

Avoid making account lockout itself an easy denial-of-service mechanism.

---

# 65. API Gateway Security

In microservices:

```text
Client
  ↓
API Gateway
  ↓
Services
```

The gateway may handle:

```text
Authentication
Rate limiting
Routing
TLS termination
Request filtering
```

Services should still enforce authorization for sensitive operations.

---

# 66. Service-to-Service Security

Internal services can use:

```text
OAuth2 access tokens
mTLS
Service identities
Signed requests
Network policies
```

Do not assume "internal network" automatically means trusted.

---

# 67. Least Privilege

Give users and services only the permissions they need.

Example:

```text
Product read service
    ↓
PRODUCT_READ
```

rather than:

```text
ADMIN
```

for every service.

---

# 68. Defense in Depth

Security should exist at multiple layers:

```text
TLS
 ↓
Gateway
 ↓
Authentication
 ↓
Authorization
 ↓
Input validation
 ↓
Database permissions
 ↓
Audit logging
```

No single security control should be treated as perfect.

---

# 69. Security Configuration Example

Conceptually:

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http) throws Exception {

    http
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            .requestMatchers(
                "/api/auth/**"
            ).permitAll()
            .requestMatchers(
                "/api/admin/**"
            ).hasRole("ADMIN")
            .anyRequest()
            .authenticated()
        );

    return http.build();
}
```

Important:

> Whether `csrf.disable()` is appropriate depends on how authentication credentials are transported and how the application is used.

---

# 70. JWT Request Flow

```text
Client
  ↓
Authorization: Bearer JWT
  ↓
Security Filter Chain
  ↓
JWT validation
  ↓
Authentication
  ↓
Authorities
  ↓
Authorization
  ↓
Controller
```

This is the key mental model for JWT-protected Spring APIs.

---

# 71. Login + JWT Ecommerce Example

```text
POST /api/auth/login
        ↓
Validate username/password
        ↓
BCrypt password verification
        ↓
Generate access token
        ↓
Return token
```

Then:

```text
GET /api/orders
Authorization: Bearer JWT
        ↓
Validate JWT
        ↓
USER authority
        ↓
OrderController
```

---

# 72. Admin Ecommerce Example

```text
DELETE /api/products/100
Authorization: Bearer JWT
        ↓
JWT valid?
        ↓
ADMIN authority?
   |             |
  NO            YES
   ↓             ↓
403           Controller
```

---

# 73. Object-Level Authorization Example

Suppose:

```text
GET /api/orders/100
```

The server should verify:

```text
authenticatedUser == order.owner
```

or:

```text
authenticatedUser has ORDER_READ_ALL
```

Do not rely only on:

```text
"User is logged in"
```

---

# 74. Authentication Architecture

A production system may look like:

```text
              Identity Provider
                     |
                     v
Client ───────> Access Token
                     |
                     v
                API Gateway
                     |
                     v
              Spring Boot API
                     |
              JWT validation
                     |
                     v
               Authorization
```

For simpler systems, Spring Boot can perform the authentication flow itself.

---

# 75. Security Testing

Security should be tested.

Examples:

```text
No token → 401
Invalid token → 401
Expired token → 401
USER → admin endpoint → 403
ADMIN → admin endpoint → allowed
User A → User B resource → 403/404 as designed
```

Also test:

```text
CORS
CSRF where relevant
Validation
Rate limiting
Token expiration
```

---

# 76. Common Security Mistakes

Avoid:

```text
Plaintext passwords
Hardcoded secrets
Long-lived tokens without controls
Logging JWTs
Returning sensitive fields
Trusting client roles
Missing object-level authorization
SQL string concatenation
Overly permissive CORS
Public Redis/database access
Disabling security controls without understanding them
```

---

# 77. Security Checklist

```text
□ HTTPS
□ Password hashing
□ Authentication
□ Authorization
□ Role/permission model
□ Object-level authorization
□ JWT validation
□ Token expiration
□ Refresh-token strategy
□ Secure key management
□ Input validation
□ DTO boundary
□ SQL injection protection
□ CORS configuration
□ CSRF decision
□ Rate limiting
□ Security headers
□ Secret management
□ Sensitive-data protection
□ Secure logging
□ Security testing
□ Least privilege
□ Monitoring and alerting
```

---

# 78. Interview: Authentication vs Authorization

> Authentication verifies who the user is, while authorization determines what that authenticated user is allowed to access. For example, a user can authenticate successfully but still receive 403 when trying to access an admin-only endpoint.

---

# 79. Interview: How Does JWT Authentication Work?

> During login, the server validates the credentials and issues a signed access token. The client sends that token in the Authorization Bearer header on subsequent requests. Spring Security validates the token, creates an authenticated principal with the appropriate authorities, and then authorization rules determine whether the request can proceed.

---

# 80. Interview: Why BCrypt?

> BCrypt is designed for password hashing rather than reversible encryption. It incorporates salting and is intentionally computationally expensive, which makes large-scale password cracking more difficult than using a fast general-purpose hash.

---

# 81. Interview: JWT vs Session?

> Session authentication keeps authentication state on the server, usually referenced by a cookie. JWT authentication can carry the authentication claims in the token and allow stateless request validation. JWTs can simplify horizontal scaling, but revocation and token lifecycle management require additional design.

---

# 82. Interview: 401 vs 403?

> 401 means the request does not have valid authentication, while 403 means the request is authenticated but the identity does not have sufficient permission for the resource.

---

# 83. Interview: How Do You Secure an Ecommerce API?

> I use HTTPS, secure password hashing, authentication, role and permission-based authorization, DTOs for the API boundary, input validation, object-level access checks, parameterized database queries, rate limiting for sensitive endpoints, secure secret management and security-focused tests.

---

# 84. Interview: How Do You Prevent Users Accessing Another User's Orders?

> I don't rely only on authentication. After identifying the user, I check that the requested order belongs to that user or that the user has an explicit permission to access other users' orders. This is object-level authorization and helps prevent IDOR vulnerabilities.

---

# 85. Interview: Why Is JWT Not Encrypted?

> A standard signed JWT is encoded and signed, not encrypted. Its signature provides integrity and authenticity when verified correctly, but the payload can generally be decoded by anyone who has the token. Therefore I never put secrets or sensitive information into normal JWT claims.

---

# 86. Interview: What Happens in the Spring Security Filter Chain?

> The request passes through security filters before reaching the controller. Authentication mechanisms such as JWT processing can establish the authenticated principal and authorities in the SecurityContext, after which authorization rules determine whether the request is allowed.

---

# 87. Interview: Why Use DTOs for Security?

> DTOs create an explicit API boundary. They prevent clients from directly controlling fields that should be server-managed, such as roles, ownership, IDs or internal status. I map validated request DTOs to domain entities inside the application.

---

# 88. Interview: How Do You Handle JWT Revocation?

> Because a self-contained JWT can remain valid until expiration, I prefer short-lived access tokens and a secure refresh-token strategy. For higher-risk scenarios I can also use token identifiers, server-side session state or a denylist, depending on the revocation requirement.

---

# 89. Interview: What Is Least Privilege?

> Least privilege means giving a user or service only the permissions required to perform its job. For example, a service that only reads products should receive product-read permission rather than an administrator role.

---

# 90. Final Mental Model

```text
                         REQUEST
                            |
                            v
                    HTTPS / Network
                            |
                            v
                  Security Filter Chain
                            |
              +-------------+-------------+
              |                           |
        Authentication               Invalid
              |                           |
              v                           v
       SecurityContext                 401
              |
              v
        Authorization
         /         \
       Allow       Deny
        |            |
        v            v
   Controller       403
        |
        v
      Service
        |
        v
     Database
```

For JWT:

```text
Login
  ↓
Validate password
  ↓
Issue JWT
  ↓
Client sends Bearer token
  ↓
Validate JWT
  ↓
Build Authentication
  ↓
Authorize
  ↓
Execute endpoint
```

> **Security is not just adding JWT. A secure REST API combines strong authentication, explicit authorization, safe password handling, input validation, object-level access checks, secure secrets, transport security, rate limiting, and continuous testing.**
