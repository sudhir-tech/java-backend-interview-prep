# Spring Boot — Configuration Properties and Profiles

Spring Boot applications should keep environment-specific and operational settings outside business logic.

Typical configuration includes:

```text
Database URL
Database credentials
Server port
External API URLs
Timeouts
Feature flags
Logging levels
Cache settings
Message broker settings
Security configuration
```

The main concepts are:

```text
application.properties
application.yml
Externalized Configuration
Environment Variables
Profiles
@ConfigurationProperties
@Value
Validation
Configuration Precedence
Secrets
```

---

# 1. Why Externalized Configuration?

Bad:

```java
String paymentUrl =
    "https://payment.example.com";
```

The URL is now hardcoded into application code.

Better:

```properties
payment.base-url=https://payment.example.com
```

Then the application reads it from configuration.

Benefits:

```text
No code changes between environments
Easier deployment
Better security
Centralized configuration
Easier testing
```

---

# 2. application.properties

Default configuration file:

```text
src/main/resources/application.properties
```

Example:

```properties
spring.application.name=ecommerce-service

server.port=8080

spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

---

# 3. application.yml

Spring Boot also supports YAML:

```text
src/main/resources/application.yml
```

Example:

```yaml
spring:
  application:
    name: ecommerce-service

  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

server:
  port: 8080
```

Choose either format based on project/team conventions.

---

# 4. Properties vs YAML

Properties:

```properties
server.port=8080
spring.application.name=ecommerce
```

YAML:

```yaml
server:
  port: 8080

spring:
  application:
    name: ecommerce
```

YAML is often easier to read for deeply nested configuration.

Both represent Spring Boot configuration properties.

---

# 5. Externalized Configuration

Spring Boot can obtain configuration from multiple sources.

Common sources include:

```text
application.properties
application.yml
Profile-specific files
Environment variables
Java system properties
Command-line arguments
External configuration files
```

This allows the same application artifact to run with different configuration.

---

# 6. Environment Variables

Example:

```bash
export DB_URL="jdbc:mysql://localhost:3306/ecommerce"
```

Configuration:

```properties
spring.datasource.url=${DB_URL}
```

The application gets the value from the environment.

---

# 7. Environment Variable for Port

```properties
server.port=${SERVER_PORT:8080}
```

Meaning:

```text
Use SERVER_PORT if available.
Otherwise use 8080.
```

If:

```text
SERVER_PORT=9090
```

then:

```text
server.port = 9090
```

---

# 8. Property Placeholders

Spring supports placeholders:

```properties
payment.url=${PAYMENT_URL}
```

You can provide a default:

```properties
payment.url=${PAYMENT_URL:https://localhost:9000}
```

Format:

```text
${PROPERTY:DEFAULT_VALUE}
```

---

# 9. @Value

A single configuration value can be injected using:

```java
@Value("${payment.timeout}")
private Duration timeout;
```

Configuration:

```properties
payment.timeout=5s
```

This is convenient for small, isolated values.

---

# 10. Problems with Too Much @Value

Imagine:

```java
@Value("${payment.url}")
private String url;

@Value("${payment.timeout}")
private Duration timeout;

@Value("${payment.retry}")
private int retry;

@Value("${payment.enabled}")
private boolean enabled;
```

This spreads configuration knowledge throughout the application.

For related settings, prefer:

```text
@ConfigurationProperties
```

---

# 11. @ConfigurationProperties

Example:

```java
@ConfigurationProperties(
    prefix = "payment"
)
public record PaymentProperties(
    String baseUrl,
    Duration timeout,
    int retry,
    boolean enabled
) {
}
```

Configuration:

```yaml
payment:
  base-url: https://payment.example.com
  timeout: 5s
  retry: 3
  enabled: true
```

Now related settings are grouped into one typed object.

---

# 12. Why @ConfigurationProperties?

Advantages:

```text
Type safety
Grouped configuration
Cleaner code
Better IDE support
Validation support
Easy testing
```

It is generally a better choice for larger configuration groups.

---

# 13. Enabling Configuration Properties

Modern Spring Boot applications can scan configuration property classes:

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class EcommerceApplication {

}
```

Then:

```java
@ConfigurationProperties(
    prefix = "payment"
)
public record PaymentProperties(
    String baseUrl,
    Duration timeout
) {
}
```

---

# 14. @EnableConfigurationProperties

You can explicitly register a configuration properties class:

```java
@Configuration
@EnableConfigurationProperties(
    PaymentProperties.class
)
public class PaymentConfig {

}
```

This is useful when you want explicit registration.

---

# 15. Configuration Properties with a Record

Modern Java makes records convenient for immutable configuration.

```java
@ConfigurationProperties(
    prefix = "payment"
)
public record PaymentProperties(
    String baseUrl,
    Duration timeout
) {
}
```

This makes configuration values:

```text
Explicit
Immutable
Type-safe
```

---

# 16. Configuration Properties with a Class

Traditional style:

```java
@ConfigurationProperties(
    prefix = "payment"
)
public class PaymentProperties {

    private String baseUrl;

    private Duration timeout;

    public String getBaseUrl() {
        return baseUrl;
    }

    public void setBaseUrl(
            String baseUrl) {

        this.baseUrl = baseUrl;
    }

    public Duration getTimeout() {
        return timeout;
    }

    public void setTimeout(
            Duration timeout) {

        this.timeout = timeout;
    }
}
```

Records are often more concise when immutability is appropriate.

---

# 17. Configuration Validation

Important configuration should be validated.

Example:

```java
@ConfigurationProperties(
    prefix = "payment"
)
@Validated
public class PaymentProperties {

    @NotBlank
    private String baseUrl;

    @NotNull
    private Duration timeout;

    // getters/setters
}
```

If configuration is invalid, application startup can fail early.

---

# 18. Common Validation Annotations

For configuration:

```text
@NotNull
@NotBlank
@Min
@Max
@Positive
@PositiveOrZero
@Size
@Email
```

Use constraints appropriate to the property type.

---

# 19. Nested Configuration

Example:

```yaml
payment:
  provider:
    name: stripe
    region: ap-south-1
  timeout: 5s
```

Can be represented with nested objects:

```java
@ConfigurationProperties(
    prefix = "payment"
)
public record PaymentProperties(
    Provider provider,
    Duration timeout
) {

    public record Provider(
        String name,
        String region
    ) {}
}
```

---

# 20. Relaxed Binding

Spring Boot supports relaxed property binding.

For example, a Java property:

```java
baseUrl
```

can commonly map from configuration such as:

```text
base-url
baseUrl
BASE_URL
```

depending on the property source and binding rules.

For environment variables, uppercase underscore-separated names are commonly used.

---

# 21. Configuration Prefix

Example:

```java
@ConfigurationProperties(
    prefix = "payment"
)
```

maps:

```yaml
payment:
  base-url: ...
  timeout: ...
```

The prefix provides a logical namespace for related settings.

Good prefixes are:

```text
payment
database
app
storage
external-api
```

Avoid overly generic names such as:

```text
config
settings
values
```

when a domain-specific prefix is clearer.

---

# 22. Profiles

Profiles allow environment-specific beans and configuration.

Typical profiles:

```text
dev
test
prod
```

Example:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

---

# 23. application.yml

Shared configuration:

```yaml
spring:
  application:
    name: ecommerce-service

server:
  port: 8080
```

This can contain values shared across environments.

---

# 24. application-dev.yml

Development-specific configuration:

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce_dev

logging:
  level:
    com.example.ecommerce: DEBUG
```

---

# 25. application-test.yml

Testing-specific configuration:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb

logging:
  level:
    root: WARN
```

The exact test database setup depends on the project's testing strategy.

---

# 26. application-prod.yml

Production-specific configuration:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

logging:
  level:
    root: INFO
```

Avoid committing actual production secrets.

---

# 27. Activating a Profile

One option:

```properties
spring.profiles.active=dev
```

But for deployment, it is often better to activate externally:

```bash
java -jar app.jar \
  --spring.profiles.active=prod
```

or through an environment variable:

```text
SPRING_PROFILES_ACTIVE=prod
```

---

# 28. @Profile

A bean can be active only for a specific profile.

```java
@Bean
@Profile("dev")
public PaymentClient fakePaymentClient() {

    return new FakePaymentClient();
}
```

Production:

```java
@Bean
@Profile("prod")
public PaymentClient realPaymentClient() {

    return new RealPaymentClient();
}
```

---

# 29. Multiple Active Profiles

Spring can activate multiple profiles.

Example:

```text
dev
local
```

The exact combination depends on how the application is configured.

Be careful when multiple profiles define the same property because property precedence matters.

---

# 30. Profile Groups

Spring Boot supports profile groups.

Example:

```properties
spring.profiles.group.production[0]=prod-db
spring.profiles.group.production[1]=prod-security
```

Activating:

```text
production
```

can activate:

```text
prod-db
prod-security
```

This can simplify complex environment configurations.

---

# 31. Default Profile

Spring supports a default profile when no other profile is active.

Example:

```properties
spring.profiles.default=dev
```

Be careful using development defaults in production environments.

Deployment systems should explicitly define the intended environment.

---

# 32. Profile-Specific Beans

Example:

```java
@Service
@Profile("dev")
public class MockEmailService
        implements EmailService {

}
```

Production:

```java
@Service
@Profile("prod")
public class RealEmailService
        implements EmailService {

}
```

Only the matching implementation is registered.

---

# 33. Profile vs Conditional Property

Profile:

```java
@Profile("prod")
```

means:

```text
Only active in prod profile.
```

Conditional property:

```java
@ConditionalOnProperty(
    name = "payment.enabled",
    havingValue = "true"
)
```

means:

```text
Only active when this property condition matches.
```

Profiles are environment-oriented.

Properties are feature/configuration-oriented.

---

# 34. Conditional Configuration

Spring Boot provides conditional annotations.

Important examples:

```text
@ConditionalOnProperty
@ConditionalOnClass
@ConditionalOnMissingBean
@ConditionalOnBean
@ConditionalOnMissingClass
```

These are heavily used by Spring Boot's auto-configuration system.

---

# 35. @ConditionalOnProperty

Example:

```java
@Bean
@ConditionalOnProperty(
    name = "feature.recommendations.enabled",
    havingValue = "true"
)
public RecommendationService
recommendationService() {

    return new RecommendationService();
}
```

Configuration:

```properties
feature.recommendations.enabled=true
```

---

# 36. Match If Missing

`@ConditionalOnProperty` has an important option:

```java
matchIfMissing = true
```

Example:

```java
@Bean
@ConditionalOnProperty(
    name = "feature.cache.enabled",
    havingValue = "true",
    matchIfMissing = true
)
public CacheService cacheService() {

    return new CacheService();
}
```

This means the bean can be created when the property is absent.

Use this carefully because defaults should be intentional.

---

# 37. @ConditionalOnMissingBean

Example:

```java
@Bean
@ConditionalOnMissingBean
public Clock clock() {

    return Clock.systemUTC();
}
```

Meaning:

```text
Create the default bean
unless an appropriate bean already exists.
```

This is central to Boot's customization model.

---

# 38. Auto-Configuration Back-Off

Spring Boot may provide:

```text
Default Bean
```

but if you define:

```text
Custom Bean
```

Boot can often back off.

Conceptually:

```text
Boot Default
     ↓
Is custom bean present?
     ↓
Yes → use custom configuration
No  → create default
```

---

# 39. Environment Abstraction

Spring provides:

```java
Environment
```

for accessing properties and profiles.

Example:

```java
@Component
public class AppEnvironment {

    private final Environment environment;

    public AppEnvironment(
            Environment environment) {

        this.environment =
            environment;
    }

    public boolean isProd() {

        return Arrays.asList(
            environment.getActiveProfiles()
        ).contains("prod");
    }
}
```

Prefer configuration binding for application settings instead of repeatedly querying `Environment`.

---

# 40. System Properties

Java system properties can provide configuration:

```bash
java \
  -Dserver.port=9090 \
  -jar app.jar
```

Spring Boot can use system properties as a configuration source.

---

# 41. Command-Line Arguments

You can provide:

```bash
java -jar app.jar \
  --server.port=9090
```

Spring Boot can treat command-line arguments as configuration properties.

This is useful for deployment overrides.

---

# 42. Environment Variable Naming

Example Spring property:

```text
spring.datasource.url
```

Common environment-variable form:

```text
SPRING_DATASOURCE_URL
```

Example:

```bash
export SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/ecommerce
```

---

# 43. Secrets

Never hardcode:

```properties
spring.datasource.password=myPassword123
```

in a committed production configuration file.

Prefer:

```properties
spring.datasource.password=${DB_PASSWORD}
```

and provide:

```text
DB_PASSWORD
```

through the deployment environment or a secret manager.

---

# 44. Secrets Management

Depending on infrastructure, secrets may come from:

```text
Environment variables
Cloud secret managers
Kubernetes Secrets
Vault
CI/CD secret stores
Container orchestration secret systems
```

The important principle:

> Keep secrets outside source code and version control.

---

# 45. Configuration Repository

For a simple Spring Boot application:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

may be enough.

For larger distributed systems, configuration may be managed centrally.

Possible approaches include:

```text
Spring Cloud Config
Kubernetes configuration
Cloud configuration services
Secret managers
```

The appropriate choice depends on architecture and deployment platform.

---

# 46. Configuration Precedence

Spring Boot combines multiple property sources.

A simplified mental model:

```text
Default configuration
      ↓
Application configuration
      ↓
Profile-specific configuration
      ↓
External configuration
      ↓
Environment variables
      ↓
System properties
      ↓
Command-line arguments
```

The exact ordering contains more sources and details, so do not treat this simplified diagram as a complete precedence table.

The important interview concept is:

```text
External configuration can override packaged defaults.
```

---

# 47. External Configuration Files

Spring Boot can load configuration from locations outside the packaged JAR.

This allows:

```text
Same JAR
+
Different external configuration
=
Different environment
```

This is useful in deployment environments.

---

# 48. Configuration Import

Modern Spring Boot supports configuration imports.

Example:

```properties
spring.config.import=optional:file:./config/
```

The exact import mechanism depends on the configuration source.

This allows applications to load configuration from additional locations.

---

# 49. Optional Configuration

The keyword:

```text
optional:
```

can allow an imported configuration location to be absent without automatically failing startup.

Example:

```properties
spring.config.import=optional:file:./config/application.properties
```

Use optional imports intentionally. A required configuration should generally fail fast when missing.

---

# 50. Configuration Naming

Prefer clear names:

```yaml
payment:
  base-url: ...
  timeout: ...
  retry-count: 3
```

Avoid:

```yaml
x:
  a: ...
  b: ...
```

Good configuration should be understandable without reading implementation code.

---

# 51. Configuration Defaults

Good:

```properties
payment.timeout=${PAYMENT_TIMEOUT:5s}
```

This provides a safe default.

But do not provide defaults for values that must be explicitly configured, such as:

```text
Production credentials
Security keys
Critical external service endpoints
```

Those should generally fail fast when missing.

---

# 52. Fail Fast Configuration

For required configuration:

```java
@NotBlank
private String apiKey;
```

or appropriate validation can make the application fail during startup.

This is preferable to:

```text
Application starts
↓
User sends request
↓
Missing API key discovered
↓
Runtime failure
```

---

# 53. Type-Safe Duration

Instead of:

```properties
payment.timeout=5000
```

prefer a clear unit:

```properties
payment.timeout=5s
```

and bind to:

```java
Duration timeout
```

This makes configuration easier to understand.

---

# 54. Type-Safe Data Size

Spring Boot configuration binding can also work with data-size types.

Example:

```java
DataSize maxUploadSize
```

Configuration:

```yaml
storage:
  max-upload-size: 20MB
```

This is safer and more readable than manually interpreting integer byte counts.

---

# 55. Configuration Metadata

Spring Boot can provide IDE metadata for configuration properties.

For example, with:

```java
@ConfigurationProperties(
    prefix = "payment"
)
```

IDE tooling can provide:

```text
Autocomplete
Documentation
Type information
```

This improves developer experience.

---

# 56. Naming Convention

Prefer kebab-case in external configuration:

```yaml
payment:
  base-url: ...
  timeout-seconds: ...
```

rather than inventing inconsistent naming styles.

Spring's relaxed binding helps map configuration names to Java properties.

---

# 57. Configuration for Your Ecommerce Backend

A practical configuration might be:

```yaml
spring:
  application:
    name: ecommerce-backend

  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

server:
  port: ${SERVER_PORT:8080}

jwt:
  expiration: ${JWT_EXPIRATION:3600s}

payment:
  base-url: ${PAYMENT_URL}
  timeout: ${PAYMENT_TIMEOUT:5s}
```

This keeps environment-specific values outside Java code.

---

# 58. Development Configuration

For local development:

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce
    username: root
    password: ${LOCAL_DB_PASSWORD}

server:
  port: 8080

logging:
  level:
    com.example.ecommerce: DEBUG
```

Even local passwords should preferably not be committed as real secrets.

---

# 59. Production Configuration

Production should look more like:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

server:
  port: ${SERVER_PORT:8080}

logging:
  level:
    root: INFO
```

Secrets come from the deployment environment.

---

# 60. Common Configuration Mistakes

### Mistake 1

Hardcoding credentials:

```properties
spring.datasource.password=password123
```

### Mistake 2

Putting production secrets in Git.

### Mistake 3

Using `@Value` for dozens of related settings.

### Mistake 4

Having unclear profile naming.

### Mistake 5

Using development defaults in production.

### Mistake 6

Not validating required configuration.

### Mistake 7

Assuming a simplified property-precedence diagram is the complete Spring Boot ordering.

---

# 61. @Value vs @ConfigurationProperties

| Feature | @Value | @ConfigurationProperties |
|---|---|---|
| Single value | Good | Possible |
| Grouped values | Poor | Excellent |
| Type-safe grouping | Limited | Excellent |
| Validation | Less convenient | Excellent |
| IDE metadata | Limited | Better |
| Large configuration | Not ideal | Recommended |

Interview rule:

```text
Few isolated values → @Value
Related configuration → @ConfigurationProperties
```

---

# 62. Profiles vs Environment Variables

Profiles:

```text
dev
test
prod
```

Environment variables:

```text
DB_URL
DB_PASSWORD
JWT_SECRET
```

Use profiles to select environment-specific behavior/configuration.

Use environment variables or secret management for deployment-specific values and secrets.

They are complementary.

---

# 63. Profile Example

```java
public interface PaymentGateway {

    void pay();
}
```

Development:

```java
@Service
@Profile("dev")
public class FakePaymentGateway
        implements PaymentGateway {

    public void pay() {

        System.out.println(
            "Fake payment"
        );
    }
}
```

Production:

```java
@Service
@Profile("prod")
public class StripePaymentGateway
        implements PaymentGateway {

    public void pay() {

        // real payment
    }
}
```

Now:

```text
dev → FakePaymentGateway
prod → StripePaymentGateway
```

---

# 64. Feature Flags

Some configuration is better represented as a feature flag:

```yaml
features:
  recommendations:
    enabled: true
```

Then:

```java
@Bean
@ConditionalOnProperty(
    prefix = "features.recommendations",
    name = "enabled",
    havingValue = "true"
)
public RecommendationService
recommendationService() {

    return new RecommendationService();
}
```

This separates:

```text
Environment
```

from:

```text
Feature activation
```

---

# 65. Configuration and Testing

Tests may use:

```text
application-test.yml
```

or test-specific properties.

Example:

```java
@SpringBootTest
@ActiveProfiles("test")
class OrderServiceTest {

}
```

Then Spring loads test-specific configuration.

---

# 66. @ActiveProfiles

Used in tests to activate a profile:

```java
@ActiveProfiles("test")
```

Example:

```java
@SpringBootTest
@ActiveProfiles("test")
class ProductRepositoryTest {

}
```

This helps isolate test configuration from development/production settings.

---

# 67. @TestPropertySource

Can add test-specific properties:

```java
@TestPropertySource(
    properties = {
        "payment.enabled=false"
    }
)
class PaymentServiceTest {

}
```

This is useful for focused test configuration.

---

# 68. Configuration Testing

A configuration properties class can be tested independently.

Example:

```java
@SpringBootTest
@ActiveProfiles("test")
class PaymentPropertiesTest {

}
```

For complex configuration, verify:

```text
Binding
Validation
Defaults
Profile overrides
```

---

# 69. Production Best Practices

```text
Use environment-specific profiles carefully
Keep secrets outside Git
Prefer typed configuration
Validate critical configuration
Use meaningful prefixes
Provide safe defaults only where appropriate
Use explicit units such as 5s and 20MB
Keep configuration organized by domain
Do not duplicate the same property unnecessarily
Fail fast for required settings
```

---

# 70. Configuration Interview Question

### What is externalized configuration?

Human-written answer:

> Externalized configuration means keeping environment-specific settings outside application code so the same application artifact can run in different environments. Spring Boot supports properties, YAML, environment variables, system properties, command-line arguments, and other configuration sources.

---

# 71. Why Use @ConfigurationProperties?

Human-written answer:

> I use `@ConfigurationProperties` when I have a group of related configuration values. It gives me a typed configuration object, cleaner code, and easier validation compared with scattering many `@Value` fields throughout the application.

---

# 72. @Value vs @ConfigurationProperties

Human-written answer:

> `@Value` is convenient for a few individual properties. For a larger group of related settings, I prefer `@ConfigurationProperties` because the configuration becomes strongly structured and easier to validate and maintain.

---

# 73. What Are Spring Profiles?

Human-written answer:

> Profiles allow us to activate different beans or configuration for different environments, such as development, testing, and production.

---

# 74. How Do You Activate a Profile?

Human-written answer:

> It can be activated using configuration such as `spring.profiles.active`, or externally through command-line arguments or environment variables. In deployment environments, I generally prefer external activation.

---

# 75. How Do You Store Secrets?

Human-written answer:

> I don't hardcode production secrets in source code. I keep them outside the repository using environment variables, deployment secret stores, or a dedicated secrets manager.

---

# 76. What Is @ConditionalOnProperty?

Human-written answer:

> `@ConditionalOnProperty` creates a bean or enables configuration only when a specified property matches a condition. It is useful for feature flags and conditional infrastructure.

---

# 77. What Is Auto-Configuration Back-Off?

Human-written answer:

> Spring Boot provides default configuration conditionally. If I provide my own suitable bean or configuration, Boot can often back off from creating its default implementation. This allows customization without losing the convenience of auto-configuration.

---

# 78. What Is Configuration Validation?

Human-written answer:

> Configuration validation checks important settings during application startup. For example, I can require a non-empty API URL or a valid timeout so the application fails early instead of discovering invalid configuration during a request.

---

# 79. Common Interview Scenario

### The application works locally but fails in production. What would you check?

A good answer:

> I would first compare the active profile and effective configuration between the environments. Then I would check environment variables, database connectivity, external service URLs, credentials, required secrets, and startup logs. I would also verify that production is using the expected configuration file and application version.

---

# 80. Common Interview Scenario

### How would you manage database credentials across environments?

Human-written answer:

> I would keep the datasource configuration structure in Spring Boot configuration but inject the actual credentials through environment variables or a secrets-management system. That way the same application artifact can run across environments without committing credentials to Git.

---

# 81. Common Interview Scenario

### How would you enable a feature only in production?

Option 1:

```java
@Profile("prod")
```

Option 2:

```java
@ConditionalOnProperty(
    name = "feature.enabled",
    havingValue = "true"
)
```

Which one to choose depends on whether the behavior is:

```text
Environment-specific
```

or:

```text
Explicitly feature-controlled
```

---

# 82. Configuration Checklist

```text
□ application.properties
□ application.yml
□ Externalized configuration
□ Environment variables
□ Property placeholders
□ Default values
□ @Value
□ @ConfigurationProperties
□ @ConfigurationPropertiesScan
□ @EnableConfigurationProperties
□ Configuration validation
□ Profiles
□ application-dev.yml
□ application-test.yml
□ application-prod.yml
□ @Profile
□ Profile activation
□ Profile groups
□ Conditional configuration
□ @ConditionalOnProperty
□ @ConditionalOnMissingBean
□ Configuration precedence
□ Secrets management
□ Feature flags
□ Test configuration
□ @ActiveProfiles
```

---

# 83. Quick Revision

```text
Configuration
│
├── Files
│   ├── application.yml
│   ├── application.properties
│   └── Profile-specific files
│
├── External Sources
│   ├── Environment Variables
│   ├── System Properties
│   └── Command-line Arguments
│
├── Binding
│   ├── @Value
│   └── @ConfigurationProperties
│
├── Environments
│   ├── dev
│   ├── test
│   └── prod
│
├── Conditional
│   ├── @Profile
│   ├── @ConditionalOnProperty
│   └── @ConditionalOnMissingBean
│
└── Security
    ├── No hardcoded secrets
    ├── Environment/Secret Store
    └── Fail fast for required values
```

---

# 84. Final Interview Rule

> **Treat configuration as part of the application's architecture. Keep code environment-independent, bind related settings into typed configuration objects, use profiles for environment-specific behavior, and keep secrets outside source control.**

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
```
