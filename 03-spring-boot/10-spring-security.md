# Spring Boot — Spring Security

Spring Security is the standard security framework used with Spring applications.

It provides support for:

```text
Authentication
Authorization
Password encoding
Security filters
Session management
CSRF protection
CORS integration
Method security
OAuth2
JWT-based APIs
Security headers
```

For a typical REST backend:

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
  ↓
Repository
```

---

# 1. Authentication vs Authorization

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
Authentication
→ JWT is valid

Authorization
→ User has ADMIN role
```

---

# 2. Spring Security

Add the Spring Security starter:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Once Spring Security is included, web endpoints are secured by default unless you configure otherwise.

---

# 3. Security Filter Chain

Spring Security processes requests through a chain of filters.

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

Security filters can handle:

```text
Authentication
Authorization
CSRF
CORS-related integration
Security context
Exception handling
Session management
```

---

# 4. SecurityFilterChain

Modern Spring Security configuration commonly uses:

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http)
        throws Exception {

    return http
        .authorizeHttpRequests(auth ->
            auth
                .requestMatchers(
                    "/api/auth/**"
                ).permitAll()
                .anyRequest()
                .authenticated()
        )
        .build();
}
```

This configures the authorization rules for HTTP requests.

---

# 5. AuthenticationManager

`AuthenticationManager` is responsible for processing authentication requests.

Conceptually:

```text
Credentials
    ↓
AuthenticationManager
    ↓
AuthenticationProvider
    ↓
User details / credentials
    ↓
Authenticated result
```

The exact providers involved depend on the application's authentication setup.

---

# 6. AuthenticationProvider

An `AuthenticationProvider` knows how to authenticate a particular type of authentication request.

For username/password authentication, a common provider uses:

```text
UserDetailsService
PasswordEncoder
```

---

# 7. UserDetails

Spring Security represents authenticated user information through:

```java
UserDetails
```

It contains information such as:

```text
Username
Password
Authorities
Account enabled status
Account expiration
Credentials expiration
```

---

# 8. UserDetailsService

Spring Security uses:

```java
UserDetailsService
```

to load a user.

Example:

```java
@Service
public class CustomUserDetailsService
        implements UserDetailsService {

    private final UserRepository repository;

    public CustomUserDetailsService(
            UserRepository repository) {

        this.repository = repository;
    }

    @Override
    public UserDetails loadUserByUsername(
            String username)
            throws UsernameNotFoundException {

        User user =
            repository
                .findByEmail(username)
                .orElseThrow(
                    () ->
                        new UsernameNotFoundException(
                            "User not found"
                        )
                );

        return User.withUsername(
                user.getEmail()
            )
            .password(user.getPassword())
            .authorities(
                user.getRole().name()
            )
            .build();
    }
}
```

---

# 9. PasswordEncoder

Never store passwords as plain text.

Use:

```java
PasswordEncoder
```

A common implementation is:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

---

# 10. Password Hashing

Registration:

```text
Plain password
      ↓
PasswordEncoder
      ↓
Hash
      ↓
Database
```

Login:

```text
Entered password
      ↓
PasswordEncoder.matches(...)
      ↓
Stored hash
```

You do not decrypt a password hash.

---

# 11. BCrypt

BCrypt is a password hashing algorithm commonly used for password storage.

Example:

```java
String encoded =
    passwordEncoder.encode(
        rawPassword
    );
```

Verify:

```java
boolean matches =
    passwordEncoder.matches(
        rawPassword,
        encodedPassword
    );
```

---

# 12. Why Not Encrypt Passwords?

Password storage normally uses hashing rather than reversible encryption.

Encryption:

```text
Plaintext
   ↕
Encrypted data
```

Hashing:

```text
Password
   ↓
One-way hash
```

Authentication checks whether the entered password matches the stored hash.

---

# 13. Password Registration Flow

```text
POST /api/auth/register
        ↓
Validate request
        ↓
Check duplicate email
        ↓
Encode password
        ↓
Save user
        ↓
Return safe response
```

Never return:

```text
Password
Password hash
```

to the client.

---

# 14. Login Flow

Traditional username/password login:

```text
POST /api/auth/login
        ↓
Username + password
        ↓
AuthenticationManager
        ↓
UserDetailsService
        ↓
PasswordEncoder
        ↓
Authentication successful
        ↓
Generate token/session
        ↓
Return authentication result
```

---

# 15. Stateless REST Authentication

For REST APIs, a common architecture is:

```text
Login
 ↓
JWT
 ↓
Client stores token
 ↓
Authorization header
 ↓
Server validates token
```

Example:

```http
Authorization: Bearer eyJ...
```

---

# 16. JWT

JWT stands for:

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
header.payload.signature
```

---

# 17. JWT Header

Example conceptually:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

The header describes token metadata such as the signing algorithm.

---

# 18. JWT Payload

Example:

```json
{
  "sub": "user@example.com",
  "role": "USER",
  "iat": 1724150000,
  "exp": 1724153600
}
```

The payload contains claims.

Do not put sensitive secrets into the JWT payload because it is generally encoded, not encrypted.

---

# 19. JWT Signature

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

The signature helps verify that the token has not been modified.

---

# 20. JWT Is Not Encryption

Important interview point:

> A standard JWT is usually signed, not encrypted. Its payload can generally be decoded, so sensitive information should not be placed inside it.

---

# 21. JWT Authentication Flow

```text
Client
  ↓
POST /api/auth/login
  ↓
Credentials validated
  ↓
JWT generated
  ↓
Client receives JWT
  ↓
Client sends JWT with requests
  ↓
JWT filter validates token
  ↓
SecurityContext populated
  ↓
Authorization
  ↓
Controller
```

---

# 22. Authorization Header

Common format:

```http
Authorization: Bearer <JWT>
```

Example:

```http
Authorization: Bearer eyJhbGciOi...
```

---

# 23. JWT Filter

A custom JWT filter often:

```text
Read Authorization header
        ↓
Check Bearer token
        ↓
Extract JWT
        ↓
Validate signature/claims
        ↓
Load user/authorities if required
        ↓
Create Authentication
        ↓
Set SecurityContext
```

Avoid performing unnecessary database calls on every request if the architecture can safely validate the required information from the token itself.

---

# 24. OncePerRequestFilter

A common base class for a custom JWT filter is:

```java
OncePerRequestFilter
```

Example structure:

```java
@Component
public class JwtAuthenticationFilter
        extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        // Read and validate token

        filterChain.doFilter(
            request,
            response
        );
    }
}
```

---

# 25. SecurityContext

Spring Security stores the current authentication in:

```text
SecurityContext
```

Conceptually:

```text
JWT
 ↓
Authentication
 ↓
SecurityContext
 ↓
Authorization checks
```

You can access the current authentication through:

```java
SecurityContextHolder
```

---

# 26. SecurityContextHolder

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

Use carefully in application code; prefer method parameters such as `@AuthenticationPrincipal` where they make the controller/service contract clearer.

---

# 27. @AuthenticationPrincipal

Example:

```java
@GetMapping("/me")
public UserResponse currentUser(
        @AuthenticationPrincipal
        UserDetails user) {

    return service.getUser(
        user.getUsername()
    );
}
```

This is often cleaner than directly accessing `SecurityContextHolder` in a controller.

---

# 28. Authorities

Spring Security represents permissions through:

```text
GrantedAuthority
```

Example:

```text
PRODUCT_READ
PRODUCT_WRITE
ORDER_READ
ORDER_CANCEL
```

---

# 29. Roles

A role is a common higher-level authorization concept.

Example:

```text
USER
ADMIN
MANAGER
```

Spring Security traditionally represents roles using the:

```text
ROLE_
```

prefix internally.

For example:

```text
ROLE_ADMIN
```

---

# 30. hasRole

Example:

```java
.requestMatchers("/api/admin/**")
.hasRole("ADMIN")
```

Spring Security typically checks for:

```text
ROLE_ADMIN
```

when you write:

```java
hasRole("ADMIN")
```

Do not accidentally write:

```java
hasRole("ROLE_ADMIN")
```

because the role prefix is normally added by the framework.

---

# 31. hasAuthority

For explicit authorities:

```java
.hasAuthority("PRODUCT_WRITE")
```

This checks the exact authority string.

---

# 32. Role vs Authority

Example:

```text
Role:
ADMIN

Authorities:
PRODUCT_READ
PRODUCT_WRITE
USER_DELETE
```

A role can represent a broad group of permissions.

Authorities can represent finer-grained permissions.

---

# 33. requestMatchers

Example:

```java
.authorizeHttpRequests(auth ->
    auth
        .requestMatchers(
            "/api/auth/**"
        ).permitAll()
        .requestMatchers(
            "/api/admin/**"
        ).hasRole("ADMIN")
        .anyRequest()
        .authenticated()
)
```

---

# 34. permitAll

Example:

```java
.requestMatchers(
    "/api/auth/login",
    "/api/auth/register"
).permitAll()
```

These endpoints can be accessed without authentication.

---

# 35. authenticated

Example:

```java
.anyRequest()
.authenticated()
```

Any request not matched by earlier rules must have a valid authenticated principal.

---

# 36. denyAll

Example:

```java
.requestMatchers(
    "/internal/**"
).denyAll()
```

Useful when an endpoint should not be accessible through the current HTTP security configuration.

---

# 37. Order of Authorization Rules

Authorization rules are evaluated according to their matching order.

Prefer:

```java
.requestMatchers("/api/auth/**")
    .permitAll()

.requestMatchers("/api/admin/**")
    .hasRole("ADMIN")

.anyRequest()
    .authenticated()
```

Keep specific rules before the catch-all rule.

---

# 38. Method Security

Enable method-level authorization:

```java
@EnableMethodSecurity
```

Then:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteProduct(
        Long id) {

}
```

---

# 39. @PreAuthorize

Example:

```java
@PreAuthorize(
    "hasAuthority('PRODUCT_WRITE')"
)
public ProductResponse updateProduct(
        Long id,
        ProductRequest request) {

    ...
}
```

This checks authorization before method execution.

---

# 40. @PostAuthorize

Example:

```java
@PostAuthorize(
    "returnObject.owner == authentication.name"
)
public OrderResponse getOrder(
        Long id) {

    ...
}
```

Authorization is evaluated after method execution.

Use carefully because the method executes before the authorization decision.

---

# 41. @Secured

Spring Security also supports:

```java
@Secured("ROLE_ADMIN")
```

when method security is configured appropriately.

For new applications, `@PreAuthorize` is often more expressive because it supports Spring Expression Language.

---

# 42. Security Configuration

Modern configuration:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http)
            throws Exception {

        return http
            .authorizeHttpRequests(auth ->
                auth
                    .requestMatchers(
                        "/api/auth/**"
                    ).permitAll()
                    .requestMatchers(
                        "/api/admin/**"
                    ).hasRole("ADMIN")
                    .anyRequest()
                    .authenticated()
            )
            .build();
    }
}
```

---

# 43. CSRF

CSRF stands for:

```text
Cross-Site Request Forgery
```

It is primarily relevant to browser-based applications using credentials automatically attached by the browser, such as cookies.

---

# 44. CSRF and Stateless APIs

For a stateless API using:

```text
Authorization: Bearer <JWT>
```

rather than browser-managed authentication cookies, CSRF risk is different.

Many APIs disable CSRF:

```java
http.csrf(
    csrf -> csrf.disable()
);
```

But do not blindly disable it.

The correct configuration depends on how authentication credentials are transported.

---

# 45. Cookie-Based Authentication

If authentication uses:

```text
Session cookie
```

the browser automatically sends the cookie.

A malicious site may attempt to cause requests using the victim's browser credentials.

CSRF protection is therefore important.

---

# 46. JWT in Authorization Header

If the client explicitly sends:

```http
Authorization: Bearer <token>
```

the browser does not automatically attach that header to arbitrary cross-site requests in the same way it automatically attaches cookies.

The CSRF threat model is therefore different.

Still consider:

```text
XSS
Token storage
CORS
Refresh tokens
Cookie usage
```

---

# 47. CORS

CORS stands for:

```text
Cross-Origin Resource Sharing
```

Example:

```text
Angular
http://localhost:4200

Spring Boot
http://localhost:8080
```

These are different origins.

The backend must allow the required browser origin.

---

# 48. CORS Configuration

Example:

```java
@Bean
CorsConfigurationSource corsConfigurationSource() {

    CorsConfiguration configuration =
        new CorsConfiguration();

    configuration.setAllowedOrigins(
        List.of("http://localhost:4200")
    );

    configuration.setAllowedMethods(
        List.of(
            "GET",
            "POST",
            "PUT",
            "PATCH",
            "DELETE"
        )
    );

    configuration.setAllowedHeaders(
        List.of("*")
    );

    UrlBasedCorsConfigurationSource source =
        new UrlBasedCorsConfigurationSource();

    source.registerCorsConfiguration(
        "/**",
        configuration
    );

    return source;
}
```

Do not use unrestricted origins in production without understanding the security implications.

---

# 49. HTTPS

Authentication should use:

```text
HTTPS
```

not plain HTTP in production.

HTTPS protects data in transit from network interception.

It does not protect against:

```text
Compromised server
Application vulnerabilities
Stolen tokens
XSS
Poor authorization
```

---

# 50. Security Headers

Spring Security can provide security-related response headers.

Examples include:

```text
X-Content-Type-Options
Content-Security-Policy
Strict-Transport-Security
Cache-Control
```

The exact headers and policies should match the application's requirements.

---

# 51. Session Management

For a stateless JWT API:

```java
.sessionManagement(session ->
    session.sessionCreationPolicy(
        SessionCreationPolicy.STATELESS
    )
)
```

This tells Spring Security not to use the traditional HTTP session for storing authentication state.

---

# 52. Stateful vs Stateless

Stateful:

```text
Server
 ↓
Session
 ↓
Authentication state
```

Stateless:

```text
Client
 ↓
Token on every request
 ↓
Server validates token
```

Stateless architecture can simplify horizontal scaling, but token lifecycle and revocation still need careful design.

---

# 53. JWT Expiration

A JWT should generally have an expiration claim:

```text
exp
```

Example concept:

```text
Access token
→ short lifetime
```

If stolen, a shorter lifetime limits the exposure window.

---

# 54. Refresh Tokens

A common architecture:

```text
Access Token
→ short-lived

Refresh Token
→ longer-lived
```

When the access token expires:

```text
Refresh token
      ↓
Authentication server
      ↓
New access token
```

Refresh-token storage and rotation require careful security design.

---

# 55. JWT Logout

JWTs are stateless, so logout is not always equivalent to deleting a server-side session.

Possible strategies:

```text
Short-lived access tokens
Refresh-token revocation
Refresh-token rotation
Token denylist for special cases
Server-side session/token state
```

Choose based on security requirements.

---

# 56. JWT Claims

Common claims:

```text
sub → subject
iss → issuer
aud → audience
iat → issued at
exp → expiration
jti → token ID
```

Custom claims can represent:

```text
role
permissions
tenant
```

Do not put unnecessary information into tokens.

---

# 57. JWT Secret

For HMAC signing:

```text
Secret key
```

must be protected.

Never hardcode production secrets:

```java
String secret =
    "my-secret-key";
```

Use:

```text
Environment variables
Secret manager
Vault
Cloud secret storage
```

---

# 58. RSA / EC Signing

JWTs can also use asymmetric cryptography.

Conceptually:

```text
Private key
→ signs token

Public key
→ verifies token
```

This can be useful in distributed systems where multiple services need to verify tokens without receiving the private signing key.

---

# 59. Password Storage Rules

Never store:

```text
password
plainPassword
```

directly.

Use:

```text
PasswordEncoder
```

Store:

```text
encoded password hash
```

Also:

```text
Never log passwords
Never return passwords
Never put passwords in JWT claims
```

---

# 60. Authentication Failure

Spring Security can handle authentication failures.

Typical result:

```text
401 Unauthorized
```

For example:

```text
Missing token
Invalid token
Expired token
Invalid credentials
```

The exact response depends on configured authentication entry points.

---

# 61. Access Denied

When authentication succeeds but authorization fails:

```text
403 Forbidden
```

Spring Security uses an access-denied mechanism for this.

Conceptually:

```text
Authentication
      ↓
Valid
      ↓
Authorization
      ↓
Denied
      ↓
403
```

---

# 62. AuthenticationEntryPoint

An `AuthenticationEntryPoint` handles unauthenticated access attempts.

Conceptually:

```java
http.exceptionHandling(
    exception ->
        exception.authenticationEntryPoint(
            customEntryPoint
        )
);
```

For REST APIs, configure a consistent JSON response if required.

---

# 63. AccessDeniedHandler

An `AccessDeniedHandler` handles authorization failures.

Conceptually:

```java
http.exceptionHandling(
    exception ->
        exception.accessDeniedHandler(
            customAccessDeniedHandler
        )
);
```

Typical response:

```text
403 Forbidden
```

---

# 64. Security Error Response

A REST API may return:

```json
{
  "code": "ACCESS_DENIED",
  "message": "You do not have permission to access this resource"
}
```

Keep authentication and authorization errors consistent with the rest of the API.

---

# 65. Role-Based Access Control

RBAC means:

```text
Role-Based Access Control
```

Example:

```text
ADMIN
USER
MANAGER
```

Authorization:

```text
ADMIN
→ create/update/delete products

USER
→ view products

MANAGER
→ view reports
```

---

# 66. Permission-Based Authorization

Instead of only roles:

```text
PRODUCT_READ
PRODUCT_WRITE
ORDER_READ
ORDER_CANCEL
```

Then:

```java
@PreAuthorize(
    "hasAuthority('PRODUCT_WRITE')"
)
```

This is more granular.

---

# 67. Role Hierarchy

Sometimes:

```text
ADMIN
  ↓
MANAGER
  ↓
USER
```

A role hierarchy can allow higher-level roles to inherit lower-level permissions.

Do not add role hierarchies unless the business model actually requires them.

---

# 68. Multi-Tenant Security

For a multi-tenant application:

```text
Tenant A
Tenant B
Tenant C
```

Authorization must ensure:

```text
User from Tenant A
        ↓
Can only access Tenant A data
```

This is not solved merely by checking:

```text
ROLE_USER
```

Tenant isolation must be enforced at appropriate service/query/data boundaries.

---

# 69. Object-Level Authorization

Example:

```text
User can update only their own order.
```

Checking:

```text
hasRole("USER")
```

is not enough.

You also need:

```text
order.userId == authenticatedUserId
```

This is called object/resource-level authorization.

---

# 70. Method-Level Example

```java
@PreAuthorize(
    "#userId == authentication.principal.id"
)
public UserResponse getUser(
        Long userId) {

    ...
}
```

The exact expression depends on the application's principal model.

---

# 71. Security and Service Layer

Do not rely only on controller authorization.

For critical business operations, authorization should be enforced at a layer that cannot easily be bypassed by another entry point.

Example:

```text
Controller
    ↓
Service
    ↓
Business authorization
    ↓
Repository
```

---

# 72. API Gateway

In microservices:

```text
Client
   ↓
API Gateway
   ↓
Service A
Service B
Service C
```

The gateway may handle:

```text
Authentication
Token validation
Rate limiting
Routing
CORS
```

But individual services should still enforce authorization and trust boundaries appropriately.

---

# 73. SecurityContext in Async Processing

Security context propagation can become important when using:

```text
@Async
Thread pools
Reactive pipelines
Message consumers
```

Do not assume authentication automatically behaves the same across every asynchronous boundary.

---

# 74. Method Security Best Practice

Use:

```java
@EnableMethodSecurity
```

then:

```java
@PreAuthorize(
    "hasAuthority('ORDER_CANCEL')"
)
public void cancelOrder(
        Long orderId) {

}
```

This keeps important authorization close to the business operation.

---

# 75. Spring Security and Password Authentication

Typical flow:

```text
Login Request
     ↓
AuthenticationManager
     ↓
DaoAuthenticationProvider
     ↓
UserDetailsService
     ↓
User from database
     ↓
PasswordEncoder.matches()
     ↓
Authenticated
```

The exact provider configuration may differ depending on the application.

---

# 76. AuthenticationManager Example

Conceptually:

```java
Authentication authentication =
    authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(
            request.email(),
            request.password()
        )
    );
```

If successful:

```text
authentication.isAuthenticated()
```

can be true.

---

# 77. Token Generation

After authentication:

```java
String token =
    jwtService.generateToken(
        authentication
    );
```

Return:

```json
{
  "accessToken": "eyJ..."
}
```

Do not include the user's password.

---

# 78. JWT Validation

On each protected request:

```text
Authorization header
        ↓
Extract token
        ↓
Validate signature
        ↓
Validate expiration
        ↓
Validate issuer/audience where required
        ↓
Create Authentication
        ↓
SecurityContext
```

---

# 79. Token Validation Security

Do not only check:

```text
JWT is structurally valid
```

Depending on the system, validate:

```text
Signature
Expiration
Issuer
Audience
Algorithm
Required claims
Token type
```

---

# 80. Algorithm Confusion

JWT verification should use a controlled, expected algorithm configuration.

Do not trust an algorithm value from an untrusted token without enforcing what algorithms the application accepts.

---

# 81. Token Storage

For browser applications, token storage has security tradeoffs.

Consider:

```text
XSS risk
CSRF risk
HttpOnly cookies
SameSite
Secure flag
Memory storage
Refresh-token rotation
```

There is no universally correct storage choice for every architecture.

---

# 82. HttpOnly Cookie

An `HttpOnly` cookie cannot be accessed by JavaScript through normal browser APIs.

Example concept:

```text
Set-Cookie:
accessToken=...;
HttpOnly;
Secure;
SameSite=...
```

This can reduce token exposure to JavaScript-based theft, but cookie-based authentication introduces CSRF considerations that must be handled.

---

# 83. Secure Cookie

Production authentication cookies should generally use:

```text
Secure
```

so browsers send them only over HTTPS.

---

# 84. SameSite

Cookie:

```text
SameSite
```

helps control cross-site cookie sending.

Common values:

```text
Strict
Lax
None
```

`SameSite=None` requires `Secure` in modern browsers.

---

# 85. CORS vs CSRF

They are different.

CORS:

```text
Controls browser cross-origin access to responses/requests
```

CSRF:

```text
Protects against unwanted authenticated state-changing requests
```

Configuring CORS does not automatically solve CSRF.

---

# 86. XSS

XSS means:

```text
Cross-Site Scripting
```

An attacker injects malicious script into a web page.

XSS can be especially dangerous when tokens are accessible to JavaScript.

Defenses include:

```text
Output encoding
Content Security Policy
Input handling
Secure frameworks
HttpOnly cookies where appropriate
Avoiding unsafe HTML injection
```

---

# 87. SQL Injection

Never build SQL by string concatenation:

```java
"SELECT * FROM users WHERE email = '"
    + email
    + "'";
```

Use:

```text
Prepared statements
JPA parameters
Repository query parameters
```

Example:

```java
@Query("""
    SELECT u
    FROM User u
    WHERE u.email = :email
""")
Optional<User> findByEmail(
    @Param("email")
    String email
);
```

---

# 88. Mass Assignment

Do not blindly bind client JSON directly to entities containing sensitive fields.

Bad concept:

```text
Client sends:
role=ADMIN
```

and application directly updates the entity.

Use DTOs:

```text
UpdateUserRequest
```

with only fields the client is allowed to change.

---

# 89. Sensitive Data Exposure

Do not return:

```text
passwordHash
internal tokens
secret keys
payment secrets
internal database details
```

Use response DTOs to control exposed fields.

---

# 90. Rate Limiting

Authentication endpoints are common targets for:

```text
Brute-force attacks
Credential stuffing
Abuse
```

Consider:

```text
Rate limiting
Account lockout policies
Progressive delays
Monitoring
CAPTCHA where appropriate
```

Rate limiting can be implemented at:

```text
API gateway
Reverse proxy
Application
Distributed cache
```

---

# 91. Login Brute Force

Example:

```text
1000 failed logins
from same IP/account
```

Possible defenses:

```text
Rate limit
Temporary lockout
Progressive delay
Alerting
IP/device analysis
```

Do not reveal whether an account exists through overly specific login error messages when that creates an account-enumeration risk.

---

# 92. Security Logging

Log security events such as:

```text
Authentication failures
Authorization failures
Password changes
Account lockouts
Administrative actions
Suspicious activity
```

Do not log:

```text
Passwords
Full access tokens
Secret keys
Sensitive personal data
```

---

# 93. Security Testing

Important tests:

```text
Unauthenticated request → 401
Authenticated USER → allowed user endpoint
USER → ADMIN endpoint → 403
ADMIN → ADMIN endpoint → success
Invalid JWT → 401
Expired JWT → 401
Missing permission → 403
```

---

# 94. Controller Security Test

Example:

```java
mockMvc.perform(
        get("/api/admin/products")
    )
    .andExpect(
        status().isUnauthorized()
    );
```

Authenticated user:

```text
USER
```

should receive:

```text
403
```

when authenticated but unauthorized.

---

# 95. Security Configuration Example

A typical stateless API:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http)
            throws Exception {

        return http
            .csrf(csrf ->
                csrf.disable()
            )
            .sessionManagement(session ->
                session.sessionCreationPolicy(
                    SessionCreationPolicy.STATELESS
                )
            )
            .authorizeHttpRequests(auth ->
                auth
                    .requestMatchers(
                        "/api/auth/**"
                    )
                    .permitAll()
                    .requestMatchers(
                        "/api/admin/**"
                    )
                    .hasRole("ADMIN")
                    .anyRequest()
                    .authenticated()
            )
            .build();
    }
}
```

Important:

> Disabling CSRF here is appropriate only when the authentication architecture and threat model justify it, such as a properly designed stateless bearer-token API.

---

# 96. Adding a JWT Filter

Conceptually:

```java
http.addFilterBefore(
    jwtAuthenticationFilter,
    UsernamePasswordAuthenticationFilter.class
);
```

This places the JWT authentication filter before the standard username/password authentication filter.

---

# 97. Complete JWT Flow

```text
                    LOGIN
                      │
                      ▼
              /api/auth/login
                      │
                      ▼
             AuthenticationManager
                      │
                      ▼
             UserDetailsService
                      │
                      ▼
              PasswordEncoder
                      │
                      ▼
               Authentication
                      │
                      ▼
                  JWT
                      │
                      ▼
                  Client
                      │
          Authorization: Bearer JWT
                      │
                      ▼
             Security Filter Chain
                      │
                      ▼
               JWT Validation
                      │
                      ▼
               SecurityContext
                      │
                      ▼
                Authorization
                      │
                      ▼
                 Controller
```

---

# 98. Spring Security Interview Questions

## What is Spring Security?

> Spring Security is a framework for authentication, authorization, password handling, security filters, and other application security concerns.

---

## Authentication vs Authorization?

> Authentication verifies who the user is, while authorization determines what the authenticated user is allowed to access.

---

## What is SecurityFilterChain?

> It defines the security filters and HTTP authorization rules that process incoming requests before they reach the controller.

---

## What is UserDetailsService?

> It loads user information, typically from a database, during username/password authentication.

---

## What is PasswordEncoder?

> It securely hashes passwords and verifies a raw password against the stored hash. We should never store passwords as plain text.

---

## What is JWT?

> JWT is a signed token format commonly used for stateless authentication. It contains claims and a signature that can be validated by the server.

---

## Is JWT encrypted?

> Usually no. A standard JWT is signed, not encrypted, so its payload should not contain sensitive information.

---

## How does JWT authentication work?

> The user logs in and receives a token. The client sends the token in the Authorization header on subsequent requests. A security filter validates the token and establishes the authenticated principal in the SecurityContext.

---

## Why use stateless sessions?

> In a stateless API, the server doesn't need to maintain traditional session authentication state for every client. Each request carries its authentication information, which can simplify horizontal scaling.

---

## What is @PreAuthorize?

> It provides method-level authorization using expressions, for example checking whether the current user has a specific role or authority.

---

## What is the difference between hasRole and hasAuthority?

> `hasRole("ADMIN")` normally checks for the `ROLE_ADMIN` authority, while `hasAuthority("PRODUCT_WRITE")` checks the exact authority string.

---

## What is 401 vs 403?

> 401 means the request isn't successfully authenticated. 403 means the user is authenticated but doesn't have permission to perform the operation.

---

## Why use BCrypt?

> BCrypt is a password hashing algorithm designed for password storage and includes salting and configurable computational cost.

---

## What is CSRF?

> CSRF is an attack where a victim's browser is tricked into making an unwanted authenticated request. It is especially relevant to cookie-based authentication.

---

## Should CSRF always be disabled for REST APIs?

> No. It depends on the authentication mechanism. A stateless bearer-token API has a different CSRF threat model from a browser application using automatically attached cookies.

---

## What is CORS?

> CORS is a browser security mechanism that controls whether a web application from one origin can make cross-origin requests to another origin.

---

## Where should authorization happen?

> HTTP-level rules can be configured in the SecurityFilterChain, while important business/resource-level authorization can also be enforced at the service or method level so it cannot be bypassed through another entry point.

---

# 99. Common Security Mistakes

Avoid:

```text
□ Plain-text passwords
□ Hardcoded JWT secrets
□ Long-lived access tokens without reason
□ Sensitive data in JWT payload
□ Logging passwords/tokens
□ Returning entities with sensitive fields
□ Using "*" CORS without understanding it
□ Disabling CSRF blindly
□ Trusting client-provided roles
□ Relying only on controller authorization
□ Ignoring database-level authorization boundaries
□ Returning different login messages that reveal account existence
□ Skipping rate limiting on sensitive endpoints
```

---

# 100. Production Security Checklist

```text
□ HTTPS
□ Password hashing
□ Secure password policy
□ JWT validation
□ Token expiration
□ Refresh-token strategy
□ Authentication rules
□ Authorization rules
□ Method security
□ Role/authority design
□ Resource-level authorization
□ CORS configuration
□ CSRF configuration
□ Security headers
□ Secure cookies where applicable
□ Rate limiting
□ Brute-force protection
□ Input validation
□ DTOs
□ SQL injection prevention
□ Sensitive-data protection
□ Security logging
□ Monitoring
□ Security tests
□ Secret management
```

---

# 101. Final Security Mental Model

```text
                 HTTP Request
                       │
                       ▼
             SecurityFilterChain
                       │
                       ▼
                Authentication
                       │
              ┌────────┴────────┐
              │                 │
           Invalid             Valid
              │                 │
             401                ▼
                         Authorization
                               │
                         ┌─────┴─────┐
                         │           │
                       Denied      Allowed
                         │           │
                        403          ▼
                               Controller
                                   │
                                 Service
                                   │
                               Database
```

---

# 102. Final Interview Rule

> **Spring Security should authenticate the caller, authorize access based on roles/authorities and resource ownership, protect credentials and tokens, and enforce security before business operations execute. For REST APIs, I typically use a stateless security model with carefully validated bearer tokens, while keeping authorization rules explicit and avoiding sensitive data exposure.**

Next:

```text
01 Fundamentals
      ↓
02 Project Structure
      ↓
03 Dependency Injection & IoC
      ↓
04 Spring Beans & Configuration
      ↓
05 Spring Boot Annotations
      ↓
06 Configuration Properties & Profiles
      ↓
07 REST API Development
      ↓
08 Spring Data JPA
      ↓
09 Exception Handling & Validation
      ↓
10 Spring Security
      ↓
11 Spring Boot Testing
```
