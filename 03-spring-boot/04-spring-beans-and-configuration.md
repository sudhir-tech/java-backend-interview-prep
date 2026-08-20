# Spring Boot — Spring Beans and Configuration

Spring Beans and configuration are central to how Spring Boot applications are assembled.

The key ideas are:

```text
Bean
↓
Bean Definition
↓
IoC Container
↓
Configuration
↓
Bean Lifecycle
↓
Dependency Injection
```

This file covers how Spring creates, configures, scopes, and manages application objects.

---

# 1. What Is a Spring Bean?

A Spring Bean is an object managed by the Spring IoC container.

Spring can:

```text
Create it
Configure it
Inject dependencies
Manage its lifecycle
Apply framework features
```

Example:

```java
@Service
public class ProductService {

}
```

`ProductService` becomes a Spring-managed bean.

---

# 2. Bean Definition

A Bean Definition tells Spring how a bean should be created and managed.

Conceptually, it contains information such as:

```text
Bean class
Scope
Dependencies
Initialization behavior
Destruction behavior
Configuration metadata
```

Spring uses this information when building the application context.

---

# 3. How Beans Are Registered

Common approaches:

```text
@Component
@Service
@Repository
@Controller
@RestController
@Bean
```

Component annotations use scanning.

`@Bean` uses explicit configuration.

---

# 4. @Component

```java
@Component
public class EmailClient {

}
```

Spring discovers the class through component scanning.

This is useful for generic application components.

---

# 5. @Service

```java
@Service
public class OrderService {

}
```

`@Service` is a specialized component stereotype commonly used for business logic.

Conceptually:

```text
@Service
    ↓
@Component
```

The annotation communicates the role of the class.

---

# 6. @Repository

```java
@Repository
public class ProductRepository {

}
```

`@Repository` is intended for persistence/data-access components.

Spring also provides exception translation behavior for suitable repository components.

Spring Data repository interfaces are normally detected and proxied by Spring Data.

---

# 7. @Controller

```java
@Controller
public class HomeController {

}
```

Used for Spring MVC controllers.

---

# 8. @RestController

```java
@RestController
public class ProductController {

}
```

`@RestController` combines the semantics of:

```java
@Controller
@ResponseBody
```

It is commonly used for REST APIs.

---

# 9. @Configuration

`@Configuration` marks a class as a source of bean definitions.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public Clock clock() {
        return Clock.systemUTC();
    }
}
```

The configuration class itself is also managed by Spring.

---

# 10. @Bean

`@Bean` is placed on a method.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentClient paymentClient() {
        return new PaymentClient();
    }
}
```

Spring registers the returned object as a bean.

---

# 11. When to Use @Bean

Use `@Bean` when you need explicit control over object creation.

Common cases:

```text
Third-party libraries
Custom object construction
Custom configuration
Conditional infrastructure
Multiple implementations
```

Example:

```java
@Bean
public ObjectMapper objectMapper() {

    return new ObjectMapper();
}
```

---

# 12. @Component vs @Bean

### @Component

```java
@Component
public class EmailClient {

}
```

Spring discovers the class through component scanning.

### @Bean

```java
@Bean
public EmailClient emailClient() {

    return new EmailClient();
}
```

Spring registers the object returned by the configuration method.

Interview answer:

> I use component scanning for application classes and `@Bean` when I need explicit control over bean creation, especially for third-party or specially configured objects.

---

# 13. Component Scanning

Spring scans configured packages for component classes.

Example:

```text
com.example.app
├── Application.java
├── controller/
├── service/
└── repository/
```

If the application class is located at:

```text
com.example.app
```

Spring can normally discover components in its subpackages.

---

# 14. @ComponentScan

You can configure component scanning explicitly:

```java
@ComponentScan(
    basePackages = "com.example.app"
)
```

Usually, `@SpringBootApplication` already provides component scanning.

Therefore, adding `@ComponentScan` manually is often unnecessary.

---

# 15. @SpringBootApplication

`@SpringBootApplication` combines:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

Therefore:

```java
@SpringBootApplication
public class Application {

}
```

is both:

```text
Configuration source
+
Auto-configuration entry point
+
Component scanning entry point
```

---

# 16. Bean Naming

By default, Spring commonly derives a bean name from the class name.

Example:

```java
@Service
public class OrderService {

}
```

The default bean name is typically:

```text
orderService
```

You can specify a name:

```java
@Service("order")
public class OrderService {

}
```

Then the bean can be referenced using:

```text
order
```

---

# 17. Bean Naming with @Bean

For:

```java
@Bean
public PaymentClient paymentClient() {

    return new PaymentClient();
}
```

the method name commonly becomes the bean name:

```text
paymentClient
```

You can specify names:

```java
@Bean("stripeClient")
public PaymentClient paymentClient() {

    return new PaymentClient();
}
```

---

# 18. Multiple Beans of the Same Type

Suppose:

```java
@Bean
public PaymentClient stripeClient() {
    return new PaymentClient("stripe");
}

@Bean
public PaymentClient paypalClient() {
    return new PaymentClient("paypal");
}
```

Now there are two:

```text
PaymentClient
```

beans.

If you inject only by type:

```java
public OrderService(
        PaymentClient client) {
}
```

Spring cannot determine which one to use.

---

# 19. @Primary

Mark one bean as the default:

```java
@Bean
@Primary
public PaymentClient stripeClient() {

    return new PaymentClient("stripe");
}
```

Now:

```java
PaymentClient client
```

normally resolves to the primary bean unless a more specific qualifier is used.

---

# 20. @Qualifier

Choose a specific bean:

```java
public OrderService(
        @Qualifier("stripeClient")
        PaymentClient client) {

    this.client = client;
}
```

`@Qualifier` is useful when multiple beans of the same type exist.

---

# 21. @Primary vs @Qualifier

```text
@Primary
    ↓
Default choice

@Qualifier
    ↓
Explicit choice
```

Example:

```text
PaymentClient
    ├── Stripe
    └── PayPal
```

Use:

```text
@Primary
```

if Stripe should normally be selected.

Use:

```text
@Qualifier
```

when a specific consumer needs PayPal.

---

# 22. Bean Scope

A bean scope defines how long a bean instance lives and how instances are created.

Common scopes:

```text
singleton
prototype
request
session
application
websocket
```

The most important one is:

```text
singleton
```

---

# 23. Singleton Scope

Default Spring bean scope:

```java
@Service
public class ProductService {

}
```

Normally means:

```text
One ProductService instance
per Spring ApplicationContext
```

This instance may serve many concurrent requests.

---

# 24. Singleton Does Not Mean Thread-Safe

Important interview point:

```text
singleton
≠
thread-safe
```

If a singleton bean contains mutable shared state:

```java
@Service
public class OrderService {

    private String currentUser;
}
```

multiple requests can access the same field concurrently.

This can cause race conditions.

Prefer:

```text
Stateless services
Method-local variables
Immutable state
Thread-safe shared components
```

---

# 25. Prototype Scope

Prototype creates a new bean instance when the container is asked for the bean.

Example:

```java
@Component
@Scope(
    ConfigurableBeanFactory.SCOPE_PROTOTYPE
)
public class ReportGenerator {

}
```

Important:

> Injecting a prototype bean directly into a singleton does not automatically create a new prototype instance for every method call.

This is a common interview trap.

---

# 26. Prototype with ObjectProvider

If a singleton needs a fresh prototype instance repeatedly:

```java
@Component
public class ReportService {

    private final ObjectProvider<
        ReportGenerator> provider;

    public ReportService(
            ObjectProvider<
                ReportGenerator> provider) {

        this.provider = provider;
    }

    public void generate() {

        ReportGenerator generator =
            provider.getObject();

        generator.generate();
    }
}
```

This asks the container for an instance when needed.

---

# 27. Request Scope

A request-scoped bean exists for one HTTP request.

Example:

```java
@Component
@RequestScope
public class RequestContext {

}
```

Useful for request-specific data.

Do not put unnecessary mutable state into request-scoped objects.

---

# 28. Session Scope

A session-scoped bean exists for an HTTP session.

Example:

```java
@Component
@SessionScope
public class UserSession {

}
```

This is less common in stateless REST APIs.

For REST applications, user identity is often represented through authentication tokens rather than server-side HTTP session state.

---

# 29. Application Scope

Application scope generally means one bean instance associated with the web application's `ServletContext`.

Example:

```java
@Component
@ApplicationScope
public class ApplicationState {

}
```

It is different conceptually from Spring's singleton scope, although in a typical simple application they may appear similar.

---

# 30. WebSocket Scope

A WebSocket-scoped bean can have a lifecycle associated with a WebSocket session.

It is useful for applications that maintain WebSocket connections.

This is an advanced scope and is less common in ordinary REST services.

---

# 31. Scope Summary

| Scope | Lifecycle |
|---|---|
| singleton | One per ApplicationContext |
| prototype | New instance when requested |
| request | One per HTTP request |
| session | One per HTTP session |
| application | One per ServletContext |
| websocket | One per WebSocket session |

---

# 32. Bean Lifecycle

A simplified lifecycle:

```text
Bean Definition
      ↓
Instantiate
      ↓
Populate Dependencies
      ↓
Aware Callbacks
      ↓
BeanPostProcessor
      ↓
Initialization
      ↓
Bean Ready
      ↓
Application Running
      ↓
Destruction
```

The real lifecycle has more internal steps.

---

# 33. @PostConstruct

Use `@PostConstruct` for initialization after dependencies are available.

```java
@Component
public class CacheLoader {

    @PostConstruct
    public void initialize() {

        System.out.println(
            "Cache initialized"
        );
    }
}
```

The method runs after dependency injection and before the bean is ready for normal use.

---

# 34. @PreDestroy

Use `@PreDestroy` for cleanup:

```java
@Component
public class ConnectionManager {

    @PreDestroy
    public void shutdown() {

        System.out.println(
            "Closing resources"
        );
    }
}
```

For Spring-managed singleton beans, this is part of the normal destruction lifecycle.

---

# 35. @PostConstruct Package

With modern Jakarta-based Spring applications, the annotation is:

```java
import jakarta.annotation.PostConstruct;
```

and:

```java
import jakarta.annotation.PreDestroy;
```

Do not automatically copy older examples using:

```text
javax.annotation
```

into a modern Spring Boot project.

---

# 36. InitializingBean

Spring also provides lifecycle interfaces such as:

```java
InitializingBean
```

Example:

```java
@Component
public class CacheService
        implements InitializingBean {

    @Override
    public void afterPropertiesSet() {

        // initialization
    }
}
```

`@PostConstruct` is often cleaner for ordinary application code.

---

# 37. DisposableBean

Spring also provides:

```java
DisposableBean
```

Example:

```java
@Component
public class ResourceService
        implements DisposableBean {

    @Override
    public void destroy() {

        // cleanup
    }
}
```

Again, annotation-based lifecycle callbacks are often more convenient.

---

# 38. Custom Initialization and Destruction Methods

A bean can specify lifecycle methods using configuration.

Example:

```java
@Bean(
    initMethod = "start",
    destroyMethod = "stop"
)
public ExternalClient externalClient() {

    return new ExternalClient();
}
```

This is useful when configuring classes whose lifecycle methods already exist.

---

# 39. @Lazy

Singleton beans are normally created eagerly during context startup.

Use:

```java
@Lazy
```

to delay creation.

Example:

```java
@Service
@Lazy
public class ExpensiveReportService {

}
```

The bean is created when it is first needed.

---

# 40. Why Use @Lazy?

Possible reasons:

```text
Expensive initialization
Rarely used component
Startup optimization
Avoid unnecessary resource creation
```

But lazy initialization also means configuration problems may appear later instead of during startup.

Use it intentionally.

---

# 41. BeanFactory

`BeanFactory` is a basic Spring container interface.

It provides core bean management.

Conceptually:

```text
BeanFactory
    ↓
Basic IoC container
```

---

# 42. ApplicationContext

`ApplicationContext` is a richer container.

It adds capabilities such as:

```text
Application events
Internationalization
Resource loading
Environment abstraction
More integration features
```

Spring Boot applications commonly use:

```text
ApplicationContext
```

---

# 43. Environment

Spring's `Environment` abstraction represents configuration and profiles.

Example:

```java
@Component
public class EnvironmentReader {

    private final Environment environment;

    public EnvironmentReader(
            Environment environment) {

        this.environment =
            environment;
    }

    public String getProfile() {

        return Arrays.toString(
            environment.getActiveProfiles()
        );
    }
}
```

Normally, prefer typed configuration with `@ConfigurationProperties` for structured application settings.

---

# 44. @Value

A simple configuration value can be injected using:

```java
@Value("${app.name}")
private String appName;
```

Configuration:

```properties
app.name=ecommerce
```

For a few simple values, this can be convenient.

For larger configuration groups, prefer:

```text
@ConfigurationProperties
```

---

# 45. @ConfigurationProperties

Example:

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

Configuration:

```yaml
payment:
  base-url: https://payment.example.com
  timeout: 5s
```

This provides typed configuration.

---

# 46. Enabling Configuration Properties

Depending on the application's setup, configuration properties can be registered through:

```java
@ConfigurationPropertiesScan
```

or:

```java
@EnableConfigurationProperties(
    PaymentProperties.class
)
```

Example:

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application {

}
```

---

# 47. Why ConfigurationProperties?

Advantages:

```text
Type safety
Centralized configuration
Cleaner code
Validation support
Easy grouping
Better IDE support
```

Instead of:

```java
@Value("${payment.url}")
private String url;

@Value("${payment.timeout}")
private Duration timeout;

@Value("${payment.retry}")
private int retry;
```

you can use one configuration object.

---

# 48. Configuration Validation

Configuration properties can be validated.

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

Invalid configuration can then fail during startup.

This is often preferable to discovering a bad configuration only when a feature is used.

---

# 49. Profiles

Profiles allow environment-specific beans and configuration.

Example:

```java
@Profile("dev")
@Bean
public PaymentClient fakePaymentClient() {

    return new FakePaymentClient();
}
```

Production:

```java
@Profile("prod")
@Bean
public PaymentClient realPaymentClient() {

    return new RealPaymentClient();
}
```

---

# 50. Activating Profiles

Example:

```properties
spring.profiles.active=dev
```

Or externally:

```bash
java -jar app.jar \
  --spring.profiles.active=prod
```

External activation is often preferable for deployment-specific settings.

---

# 51. Profile Files

Common structure:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

Shared configuration:

```text
application.yml
```

Environment-specific overrides:

```text
application-dev.yml
application-prod.yml
```

---

# 52. @Profile

Example:

```java
@Configuration
@Profile("prod")
public class ProductionConfig {

}
```

The configuration is active only when:

```text
prod
```

is active.

---

# 53. Profiles vs Configuration Properties

Profiles answer:

```text
Which environment/configuration set is active?
```

Configuration properties answer:

```text
What are the typed configuration values?
```

They solve different problems and are often used together.

---

# 54. Conditional Beans

Spring Boot supports conditional configuration.

Common annotations include:

```text
@ConditionalOnClass
@ConditionalOnMissingBean
@ConditionalOnBean
@ConditionalOnProperty
@ConditionalOnMissingClass
```

These are heavily used internally by Spring Boot auto-configuration.

---

# 55. @ConditionalOnProperty

Example:

```java
@Bean
@ConditionalOnProperty(
    name = "payment.enabled",
    havingValue = "true"
)
public PaymentClient paymentClient() {

    return new PaymentClient();
}
```

Configuration:

```properties
payment.enabled=true
```

The bean is created when the condition matches.

---

# 56. @ConditionalOnMissingBean

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
Create this default bean
only if a suitable bean is not already present.
```

This is a key idea behind Spring Boot auto-configuration.

---

# 57. Auto Configuration and Back-Off

Spring Boot commonly provides defaults.

But if the developer provides an appropriate custom bean, Boot can often:

```text
Back off
```

from creating its own default bean.

This gives developers:

```text
Convenience
+
Customization
```

---

# 58. BeanPostProcessor

`BeanPostProcessor` allows Spring infrastructure to modify or wrap beans before and/or after initialization.

Conceptually:

```text
Bean created
   ↓
BeanPostProcessor
   ↓
Initialization
   ↓
BeanPostProcessor
   ↓
Ready
```

Many Spring features use post-processing internally.

---

# 59. Why BeanPostProcessor Matters

It helps explain how Spring implements features such as:

```text
Dependency injection
Proxies
AOP
Some annotation-driven behavior
```

You usually do not need to implement it yourself in normal application development.

---

# 60. Spring Proxies

Spring may create proxies around beans to add framework behavior.

Examples:

```text
@Transactional
@Async
@Cacheable
AOP advice
Security method interception
```

Conceptually:

```text
Your Bean
   ↑
Proxy
   ↑
Caller
```

The caller may interact with the proxy rather than the raw target object.

---

# 61. @Transactional and Proxies

Example:

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {

        // database operations
    }
}
```

Spring can create a proxy that starts/commits/rolls back the transaction around the method according to transaction configuration.

This is why proxy behavior matters when discussing:

```text
self-invocation
private methods
final methods/classes
```

depending on the proxy mechanism and Spring configuration.

---

# 62. Bean Lifecycle vs Application Lifecycle

Bean lifecycle:

```text
Create
Inject
Initialize
Destroy
```

Application lifecycle includes broader events:

```text
Application starting
Context refreshed
Application ready
Application stopping
```

Spring Boot provides application events that can be used when broader lifecycle hooks are needed.

---

# 63. Application Events

You can publish application events:

```java
applicationEventPublisher.publishEvent(
    new OrderCreatedEvent(orderId)
);
```

Listeners can react:

```java
@EventListener
public void handleOrderCreated(
        OrderCreatedEvent event) {

    // react to event
}
```

This is useful for decoupling some in-process application actions.

---

# 64. Bean Lifecycle vs Event Listener

Use lifecycle callbacks for:

```text
Bean initialization
Bean cleanup
```

Use application events for:

```text
Communication between application components
```

Do not use lifecycle hooks for normal business events.

---

# 65. FactoryBean

`FactoryBean<T>` is a special Spring extension point for creating objects through a factory abstraction.

It is an advanced topic.

Conceptually:

```text
FactoryBean
     ↓
creates/provides
     ↓
Object
```

You may encounter it in framework or library code.

---

# 66. BeanFactory vs FactoryBean

These names are easy to confuse.

### BeanFactory

The Spring container abstraction.

```text
Manages beans.
```

### FactoryBean

A special bean that creates/provides another object.

```text
Factory for a bean.
```

Remember:

```text
BeanFactory ≠ FactoryBean
```

---

# 67. @DependsOn

You can specify that one bean should be initialized before another:

```java
@Bean
@DependsOn("databaseInitializer")
public OrderService orderService() {

    return new OrderService();
}
```

Use sparingly.

Prefer clear dependency relationships where possible.

---

# 68. Bean Aliases

A bean can have aliases:

```java
@Bean({
    "paymentClient",
    "primaryPaymentClient"
})
public PaymentClient paymentClient() {

    return new PaymentClient();
}
```

Aliases can be useful for compatibility or explicit naming, but avoid unnecessary naming complexity.

---

# 69. Conditional Configuration Pattern

A useful pattern:

```text
Configuration property
       ↓
Condition
       ↓
Bean created or skipped
```

Example:

```properties
feature.payment.enabled=true
```

```java
@Bean
@ConditionalOnProperty(
    name = "feature.payment.enabled",
    havingValue = "true"
)
public PaymentClient paymentClient() {

    return new PaymentClient();
}
```

This pattern is common in configurable applications.

---

# 70. Configuration Class Design

Good:

```text
SecurityConfig
DatabaseConfig
RedisConfig
JacksonConfig
WebConfig
```

Avoid:

```text
EverythingConfig
```

with hundreds of unrelated bean definitions.

Keep configuration cohesive.

---

# 71. Third-Party Bean Configuration

Suppose a library provides:

```java
ThirdPartyClient
```

but it does not have:

```java
@Component
```

You can register it:

```java
@Configuration
public class ClientConfig {

    @Bean
    public ThirdPartyClient client() {

        return new ThirdPartyClient(
            "https://api.example.com"
        );
    }
}
```

This is one of the most common practical uses of `@Bean`.

---

# 72. Multiple Third-Party Clients

```java
@Configuration
public class PaymentConfig {

    @Bean("stripeClient")
    public PaymentClient stripeClient() {

        return new PaymentClient(
            "stripe"
        );
    }

    @Bean("paypalClient")
    public PaymentClient paypalClient() {

        return new PaymentClient(
            "paypal"
        );
    }
}
```

Consumer:

```java
public PaymentService(
        @Qualifier("stripeClient")
        PaymentClient client) {

    this.client = client;
}
```

---

# 73. Configuration Class and Proxy Behavior

Historically, `@Configuration` classes have special semantics around `@Bean` methods.

A full configuration class can preserve inter-bean method calls through Spring's configuration processing.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentClient paymentClient() {
        return new PaymentClient();
    }

    @Bean
    public OrderService orderService() {

        return new OrderService(
            paymentClient()
        );
    }
}
```

Spring's configuration processing ensures the managed bean is used rather than blindly creating a second independent instance in the normal full-configuration scenario.

Avoid relying on this behavior without understanding configuration semantics.

---

# 74. @Configuration(proxyBeanMethods)

Modern Spring allows:

```java
@Configuration(
    proxyBeanMethods = false
)
```

This can avoid configuration-class method interception when inter-bean method calls are not required.

Use it only when the configuration is designed appropriately.

For most beginners:

```java
@Configuration
```

is sufficient.

---

# 75. Bean Definition Overriding

If two configurations define beans with the same name, Spring Boot's behavior depends on the configured bean-definition overriding policy.

Do not solve naming conflicts by blindly enabling overriding.

Prefer:

```text
Unique names
Clear configuration
@Qualifier
@Primary
```

when multiple implementations are intentional.

---

# 76. Dependency Injection with Records

Modern Java makes immutable configuration and DTO types convenient.

Example:

```java
public record PaymentProperties(
    String baseUrl,
    Duration timeout
) {}
```

Records work well for immutable data structures and configuration models.

---

# 77. Bean Validation for Configuration

Example:

```java
@ConfigurationProperties(
    prefix = "payment"
)
@Validated
public record PaymentProperties(

    @NotBlank
    String baseUrl,

    @NotNull
    Duration timeout

) {}
```

This allows invalid configuration to fail early.

---

# 78. Avoid Secrets in Bean Configuration

Do not write:

```java
@Bean
public PaymentClient client() {

    return new PaymentClient(
        "my-secret-key"
    );
}
```

Prefer:

```text
Environment variables
Secret managers
External configuration
Deployment platform secrets
```

The exact secret-management solution depends on the deployment environment.

---

# 79. Configuration Precedence

Spring Boot supports configuration from multiple sources.

Common sources include:

```text
application.properties / YAML
Profile-specific configuration
Environment variables
System properties
Command-line arguments
```

More specific/external sources can override lower-priority configuration according to Spring Boot's property-source ordering.

For interviews, the important idea is:

> Configuration can be externalized and overridden without rebuilding the application.

---

# 80. Externalized Configuration Example

Application:

```properties
server.port=${SERVER_PORT:8080}
```

If:

```text
SERVER_PORT
```

is not provided:

```text
8080
```

is used.

If:

```text
SERVER_PORT=9090
```

is supplied:

```text
9090
```

is used.

---

# 81. Profiles and Deployment

A practical setup:

```text
Local:
application-dev.yml

Testing:
application-test.yml

Production:
application-prod.yml
```

But production secrets should generally not live directly in the repository.

Use:

```text
Environment variables
Secret manager
Deployment configuration
```

---

# 82. Common Bean Problems

### Problem 1 — No qualifying bean

```text
NoSuchBeanDefinitionException
```

Usually means Spring cannot find the required bean.

Check:

```text
Component scanning
Bean registration
Configuration
Profile
Conditional annotations
```

---

# 83. Multiple Beans Found

Example:

```text
NoUniqueBeanDefinitionException
```

Usually means:

```text
Multiple beans match the required type.
```

Solutions:

```text
@Primary
@Qualifier
More specific injection
```

---

# 84. Bean Creation Failure

A bean can fail during startup because:

```text
Dependency missing
Invalid configuration
Initialization failure
Database unavailable
Third-party client misconfigured
```

Read the root cause in the startup stack trace rather than only the final exception message.

---

# 85. Circular Dependency Error

If:

```text
A → B
B → A
```

Spring may fail to create the application context.

Best solution:

```text
Redesign dependency graph.
```

Do not immediately add:

```text
@Lazy
```

just to suppress the symptom.

---

# 86. Bean Lifecycle Interview Question

### What happens during bean creation?

Human-written answer:

> Spring creates the bean, injects its dependencies, runs the relevant initialization callbacks and post-processors, and then makes the bean available for normal use. When the application context shuts down, destruction callbacks can run for applicable beans.

---

# 87. What Is the Default Scope?

Human-written answer:

> Singleton is the default Spring bean scope. It means Spring normally maintains one instance of the bean per application context.

---

# 88. Singleton vs Prototype

Human-written answer:

> Singleton uses one managed instance per application context, while prototype creates a new instance when the container is asked for the bean. If a prototype is injected into a singleton, it does not automatically become a new instance for every method call.

---

# 89. @Bean vs @Component

Human-written answer:

> `@Component` is class-level component scanning, while `@Bean` is method-level explicit bean registration. I prefer `@Bean` when I need custom construction or need to register a third-party class.

---

# 90. @Primary vs @Qualifier

Human-written answer:

> `@Primary` gives Spring a default choice when multiple beans match. `@Qualifier` explicitly identifies which bean should be injected at a particular location.

---

# 91. @Value vs @ConfigurationProperties

Human-written answer:

> `@Value` is convenient for a small number of individual properties. For a group of related configuration values, I prefer `@ConfigurationProperties` because it gives me a typed and organized configuration object.

---

# 92. What Is @Lazy?

Human-written answer:

> `@Lazy` delays bean creation until the bean is actually needed. It can reduce startup work, but it can also move configuration or initialization failures from startup to first use.

---

# 93. What Is @PostConstruct?

Human-written answer:

> `@PostConstruct` marks a method that should run after dependency injection and initialization of the bean. I use it for lightweight initialization that requires the bean's dependencies to already be available.

---

# 94. What Is @PreDestroy?

Human-written answer:

> `@PreDestroy` marks cleanup logic that should run when the Spring-managed bean is being destroyed. It can be useful for releasing resources owned by the bean.

---

# 95. What Is Auto-Configuration Back-Off?

Human-written answer:

> Spring Boot provides default configuration conditionally. If I provide an appropriate custom bean or configuration, Boot can often back off from creating its default implementation. This gives me sensible defaults without preventing customization.

---

# 96. Common Mistakes

### Mistake 1

Thinking singleton means thread-safe.

It does not.

### Mistake 2

Thinking `@Bean` and `@Component` are exactly the same.

They register beans differently.

### Mistake 3

Using `@Primary` everywhere.

Use it only when a sensible default exists.

### Mistake 4

Using `@Qualifier` strings everywhere without clear naming.

Keep bean names consistent.

### Mistake 5

Using `@Lazy` to hide design problems.

### Mistake 6

Hardcoding secrets in configuration classes.

### Mistake 7

Putting all configuration in one huge class.

---

# 97. Best Practices

```text
Use meaningful bean names
Prefer constructor injection
Keep singleton services stateless
Use @Bean for explicit/third-party configuration
Use @ConfigurationProperties for grouped settings
Use profiles for environment-specific behavior
Use @Primary only for sensible defaults
Use @Qualifier for intentional alternatives
Keep configuration classes focused
Validate important configuration
Never commit secrets
Avoid circular dependencies
Use lifecycle hooks for lifecycle work only
```

---

# 98. Bean and Configuration Checklist

```text
□ Spring Bean
□ Bean Definition
□ @Component
□ @Service
□ @Repository
□ @Controller
□ @RestController
□ @Configuration
□ @Bean
□ Component Scanning
□ Bean Naming
□ @Primary
□ @Qualifier
□ Singleton
□ Prototype
□ Request Scope
□ Session Scope
□ Application Scope
□ WebSocket Scope
□ Bean Lifecycle
□ @PostConstruct
□ @PreDestroy
□ @Lazy
□ ApplicationContext
□ BeanFactory
□ @Value
□ @ConfigurationProperties
□ @Profile
□ Conditional Beans
□ @ConditionalOnProperty
□ @ConditionalOnMissingBean
□ BeanPostProcessor
□ Spring Proxies
□ Application Events
```

---

# 99. Quick Revision

```text
Bean
│
├── Registration
│   ├── @Component
│   ├── @Service
│   ├── @Repository
│   ├── @Controller
│   └── @Bean
│
├── Configuration
│   ├── @Configuration
│   ├── @Profile
│   ├── @Value
│   └── @ConfigurationProperties
│
├── Resolution
│   ├── By Type
│   ├── @Primary
│   └── @Qualifier
│
├── Scope
│   ├── Singleton
│   ├── Prototype
│   ├── Request
│   ├── Session
│   ├── Application
│   └── WebSocket
│
└── Lifecycle
    ├── Instantiate
    ├── Inject
    ├── Initialize
    ├── Use
    └── Destroy
```

---

# 100. Final Interview Rule

> **Spring Beans are the managed objects that form the application. Configuration tells Spring how those objects should be created, connected, and customized. Once you understand bean registration, dependency resolution, scope, lifecycle, and externalized configuration, most Spring Boot wiring becomes much easier to reason about.**

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
```
