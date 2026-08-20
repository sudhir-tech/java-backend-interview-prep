# Reflection & Annotations — Java Backend Interview Preparation

Reflection and annotations are important Java topics for understanding how frameworks such as Spring, Spring Boot, JPA and testing libraries work internally.

---

# 1. What Is Reflection?

Reflection is the ability of a Java program to inspect and interact with classes, methods, fields, constructors and other metadata at runtime.

Reflection APIs are mainly available under:

```java
java.lang.reflect
```

Example:

```java
Class<?> clazz = User.class;

System.out.println(clazz.getName());
```

---

# 2. What Is the Class Object?

Every loaded Java type has a corresponding `Class` object representing its runtime metadata.

You can obtain it in several ways.

### Using `.class`

```java
Class<User> clazz = User.class;
```

### Using an object

```java
User user = new User();

Class<?> clazz = user.getClass();
```

### Using the class name

```java
Class<?> clazz =
    Class.forName("com.example.User");
```

---

# 3. What Is Class.forName()?

`Class.forName()` loads a class by its fully qualified name and returns its `Class` object.

Example:

```java
Class<?> clazz =
    Class.forName(
        "com.example.service.UserService"
    );
```

Historically, `Class.forName()` was also commonly associated with triggering class initialization.

Modern frameworks may use class-loading mechanisms in different ways, so don't assume every class-loading operation behaves identically.

---

# 4. Reflection Example

Suppose:

```java
class User {

    private String name;

    public void printName() {
        System.out.println(name);
    }
}
```

Reflection can inspect:

```java
Class<?> clazz = User.class;

for (Field field : clazz.getDeclaredFields()) {
    System.out.println(field.getName());
}
```

Output:

```text
name
```

---

# 5. Getting Fields

```java
Class<?> clazz = User.class;

Field[] fields =
    clazz.getDeclaredFields();
```

This includes fields declared directly by the class, including non-public fields.

---

# 6. getFields() vs getDeclaredFields()

### getFields()

Returns accessible public fields, including inherited public fields.

```java
clazz.getFields();
```

### getDeclaredFields()

Returns fields declared directly by the class, regardless of visibility.

```java
clazz.getDeclaredFields();
```

Interview answer:

> `getFields()` focuses on public fields, including inherited public fields, while `getDeclaredFields()` returns fields declared directly in the class regardless of access modifier.

---

# 7. Getting Methods

```java
Method[] methods =
    User.class.getDeclaredMethods();
```

You can inspect:

```java
for (Method method : methods) {
    System.out.println(
        method.getName()
    );
}
```

---

# 8. getMethods() vs getDeclaredMethods()

### getMethods()

Returns public methods including inherited public methods.

```java
clazz.getMethods();
```

### getDeclaredMethods()

Returns methods declared directly by the class regardless of visibility.

```java
clazz.getDeclaredMethods();
```

---

# 9. Getting Constructors

```java
Constructor<?>[] constructors =
    User.class.getDeclaredConstructors();
```

You can inspect constructor parameters:

```java
for (Constructor<?> constructor :
        constructors) {

    System.out.println(
        constructor.getParameterCount()
    );
}
```

---

# 10. Creating an Object with Reflection

Modern Java code can use a reflected constructor:

```java
Constructor<User> constructor =
    User.class.getDeclaredConstructor();

User user =
    constructor.newInstance();
```

The old:

```java
Class.newInstance()
```

approach is deprecated and should generally be avoided.

---

# 11. Invoking a Method with Reflection

Example:

```java
Method method =
    User.class.getDeclaredMethod(
        "printName"
    );

User user = new User();

method.invoke(user);
```

`invoke()` calls the method on the supplied target object.

---

# 12. Accessing a Private Field

Reflection can inspect private fields.

Example:

```java
Field field =
    User.class.getDeclaredField("name");
```

Older reflection-based code may use:

```java
field.setAccessible(true);
```

to bypass normal Java access checks.

Modern Java's module system can restrict such access, so `setAccessible(true)` is not a universal bypass.

---

# 13. Setting a Field

Example:

```java
Field field =
    User.class.getDeclaredField("name");

field.setAccessible(true);

field.set(user, "Sudhir");
```

Then:

```java
field.get(user);
```

can retrieve the value, subject to Java access and module restrictions.

---

# 14. Why Reflection Is Powerful

Reflection allows frameworks to build generic mechanisms.

Examples:

```text
Dependency Injection
ORM
Serialization
Testing
Configuration binding
Annotations
Plugin systems
```

Frameworks can inspect classes and determine:

```text
What class is this?
What annotations exist?
What methods are available?
What dependencies exist?
What fields should be mapped?
```

---

# 15. Reflection in Spring

Spring heavily relies on metadata and reflection-related mechanisms.

For example:

```java
@Service
class UserService {
}
```

Spring can discover the class during component scanning and manage it as a bean.

Dependency injection:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(
        PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

Spring determines the required dependency and constructs the object through its bean-management infrastructure.

---

# 16. Reflection in JPA

JPA/Hibernate needs to inspect entity metadata.

Example:

```java
@Entity
class Product {

    @Id
    private Long id;

    private String name;
}
```

The persistence framework uses metadata to understand:

```text
Entity
Fields
Identifiers
Relationships
Column mappings
```

Modern frameworks may use reflection, generated accessors, proxies and bytecode enhancement depending on configuration.

---

# 17. Reflection in Testing

Testing frameworks can inspect methods and annotations.

Example:

```java
@Test
void shouldCreateUser() {
}
```

The test framework discovers methods marked with:

```java
@Test
```

and executes them according to its test engine.

---

# 18. What Are Annotations?

Annotations are metadata attached to Java program elements.

Examples:

```java
@Override
@Deprecated
@SuppressWarnings
```

Framework examples:

```java
@Component
@Service
@RestController
@Autowired
@Entity
@Id
@Transactional
```

Annotations themselves generally do not automatically change behavior. Code, the compiler, or frameworks can interpret them.

---

# 19. @Override

Example:

```java
@Override
public void pay() {
}
```

`@Override` tells the compiler that the method is intended to override an inherited method.

If it doesn't actually override one, compilation fails.

This helps catch mistakes.

---

# 20. @Deprecated

Marks an API as discouraged for new code.

Example:

```java
@Deprecated
void oldMethod() {
}
```

IDEs and compilers can warn developers about usage.

A deprecated API may still work; deprecation is not the same as immediate removal.

---

# 21. @SuppressWarnings

Suppresses selected compiler warnings.

Example:

```java
@SuppressWarnings("unchecked")
```

Use it carefully and as narrowly as possible.

Don't suppress warnings blindly.

---

# 22. Built-In vs Custom Annotations

Java provides built-in annotations such as:

```text
@Override
@Deprecated
@SuppressWarnings
```

Developers can also create custom annotations.

Example:

```java
@interface Audit {
}
```

Then:

```java
@Audit
class OrderService {
}
```

---

# 23. Creating a Custom Annotation

Example:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Audit {
}
```

Now:

```java
@Audit
public void createOrder() {
}
```

The annotation can be inspected at runtime.

---

# 24. @Target

`@Target` defines where an annotation can be applied.

Example:

```java
@Target(ElementType.METHOD)
@interface Audit {
}
```

This means the annotation can be applied to methods.

Common target values include:

```text
TYPE
METHOD
FIELD
PARAMETER
CONSTRUCTOR
ANNOTATION_TYPE
PACKAGE
MODULE
TYPE_USE
```

---

# 25. @Retention

`@Retention` determines how long annotation metadata is retained.

Three main policies:

```text
SOURCE
CLASS
RUNTIME
```

---

# 26. SOURCE Retention

```java
@Retention(
    RetentionPolicy.SOURCE
)
```

Available only in source code.

The annotation is discarded during compilation and is not available in the generated class file.

Useful for:

```text
Compile-time tooling
Source processors
Developer tooling
```

---

# 27. CLASS Retention

```java
@Retention(
    RetentionPolicy.CLASS
)
```

The annotation is stored in the compiled class file but is not necessarily available through normal runtime reflection.

This is the default retention policy if no retention annotation is specified.

---

# 28. RUNTIME Retention

```java
@Retention(
    RetentionPolicy.RUNTIME
)
```

The annotation is retained at runtime and can be inspected using reflection.

Example:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Audit {
}
```

Then:

```java
Method method =
    OrderService.class
        .getDeclaredMethod("createOrder");

if (method.isAnnotationPresent(Audit.class)) {
    System.out.println("Audited");
}
```

---

# 29. @Documented

`@Documented` indicates that an annotation should be included in generated API documentation when appropriate.

Example:

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface PublicApi {
}
```

---

# 30. @Inherited

`@Inherited` affects certain class-level annotations.

If an annotation is marked `@Inherited`, subclasses can inherit the annotation when queried through APIs such as `Class.getAnnotation()`.

Important:

> `@Inherited` applies to class inheritance, not to methods or fields.

---

# 31. Annotation with a Value

Custom annotations can define elements.

Example:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface ServiceInfo {

    String name();
}
```

Usage:

```java
@ServiceInfo(name = "payment")
class PaymentService {
}
```

---

# 32. Annotation Default Value

Annotation elements can have defaults.

```java
@interface Retry {

    int attempts() default 3;
}
```

Then:

```java
@Retry
class PaymentService {
}
```

uses:

```text
attempts = 3
```

Or:

```java
@Retry(attempts = 5)
class PaymentService {
}
```

overrides the default.

---

# 33. Multiple Annotation Values

```java
@interface Endpoint {

    String path();

    String method() default "GET";

    boolean authenticated() default true;
}
```

Usage:

```java
@Endpoint(
    path = "/users",
    method = "POST",
    authenticated = true
)
```

---

# 34. Special `value()` Element

If an annotation has an element named:

```java
value
```

you can often omit the element name when supplying that value.

Example:

```java
@interface Role {

    String value();
}
```

Then:

```java
@Role("ADMIN")
```

instead of:

```java
@Role(value = "ADMIN")
```

---

# 35. Annotation Types

An annotation declaration:

```java
@interface Audit {
}
```

defines an annotation type.

Annotations are not ordinary classes that you instantiate with:

```java
new Audit()
```

They are metadata constructs with special language and reflection support.

---

# 36. Reading an Annotation

Example:

```java
Audit audit =
    method.getAnnotation(
        Audit.class
    );
```

If present:

```java
if (audit != null) {
    // process metadata
}
```

You can also use:

```java
method.isAnnotationPresent(
    Audit.class
);
```

---

# 37. Getting All Annotations

```java
Annotation[] annotations =
    clazz.getDeclaredAnnotations();
```

Or for a method:

```java
Annotation[] annotations =
    method.getDeclaredAnnotations();
```

---

# 38. getAnnotations() vs getDeclaredAnnotations()

Similar to fields and methods:

### getAnnotations()

Returns annotations visible through the element's inheritance rules where applicable.

### getDeclaredAnnotations()

Returns annotations declared directly on that element.

For method and field annotations, don't assume inheritance works the same way as class-level `@Inherited`.

---

# 39. Meta-Annotations

Annotations can themselves be annotated.

For example:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ServiceInfo {
}
```

Here:

```text
@Target
@Retention
```

are annotations applied to another annotation.

They are called meta-annotations.

---

# 40. Spring Stereotype Annotations

Common Spring annotations include:

```text
@Component
@Service
@Repository
@Controller
@RestController
```

They help describe the role of components in a Spring application.

For example:

```java
@Service
public class OrderService {
}
```

---

# 41. @Component vs @Service

`@Service` is a specialization of Spring's component model intended to express a service-layer component.

Conceptually:

```text
@Component
    ↑
  @Service
```

The distinction is primarily semantic and helps communicate application architecture.

---

# 42. @Repository

Example:

```java
@Repository
public class UserRepository {
}
```

It indicates a persistence-related component.

Spring's repository abstraction and exception translation mechanisms can also use repository metadata depending on how the component is configured.

---

# 43. @RestController

```java
@RestController
class UserController {
}
```

It is commonly used for REST controllers.

Conceptually it combines controller semantics with response-body behavior.

---

# 44. @Autowired

Example:

```java
@Autowired
private PaymentService paymentService;
```

Spring can inject a matching bean.

However, constructor injection is generally preferred:

```java
private final PaymentService paymentService;

@Autowired
OrderService(
    PaymentService paymentService
) {
    this.paymentService = paymentService;
}
```

With a single constructor, modern Spring can generally infer constructor injection without requiring `@Autowired`.

---

# 45. Reflection and Dependency Injection

A simplified conceptual flow:

```text
Spring scans classes
        ↓
Finds component metadata
        ↓
Creates bean definitions
        ↓
Creates objects
        ↓
Resolves dependencies
        ↓
Injects dependencies
        ↓
Manages bean lifecycle
```

The actual Spring internals are more sophisticated than this simplified model.

---

# 46. Reflection and Proxies

Frameworks may use proxies to add behavior around objects.

Examples:

```text
@Transactional
@Cacheable
@Async
Spring Security
AOP
```

Conceptually:

```text
Caller
  ↓
Proxy
  ↓
Target object
```

The proxy can perform additional work before or after the target method.

---

# 47. Reflection and AOP

Aspect-Oriented Programming allows cross-cutting concerns such as:

```text
Logging
Transactions
Security
Metrics
Caching
```

to be applied around business operations.

Annotations can identify where behavior should apply.

For example:

```java
@Transactional
public void createOrder() {
}
```

The framework can use metadata to apply transactional behavior.

---

# 48. Reflection Performance

Reflection can be slower and less type-safe than direct method or field access.

Direct:

```java
user.getName();
```

Reflection:

```java
method.invoke(user);
```

Reflection can involve:

```text
Metadata lookup
Access checks
Dynamic invocation
Additional framework overhead
```

For most business applications, framework-level reflection overhead is often acceptable, but it should not be used unnecessarily in performance-critical hot paths.

---

# 49. Reflection and Type Safety

Normal Java code:

```java
User user = new User();
user.getName();
```

is strongly checked by the compiler.

Reflection:

```java
Method method =
    clazz.getMethod("getName");

method.invoke(user);
```

moves more errors to runtime.

Possible failures include:

```text
NoSuchMethodException
IllegalAccessException
InvocationTargetException
```

---

# 50. Reflection Exceptions

Common reflection-related checked exceptions include:

```text
ClassNotFoundException
NoSuchMethodException
NoSuchFieldException
IllegalAccessException
InstantiationException
InvocationTargetException
```

Modern constructor-based reflective creation commonly uses:

```java
Constructor.newInstance()
```

rather than deprecated `Class.newInstance()`.

---

# 51. InvocationTargetException

When a reflected method or constructor throws an exception, `Method.invoke()` or reflective construction can wrap the underlying exception in:

```java
InvocationTargetException
```

The underlying exception can be obtained using:

```java
exception.getCause();
```

---

# 52. Reflection Security

Reflection can expose internal implementation details if used carelessly.

Potential concerns include:

```text
Breaking encapsulation
Unexpected access
Security vulnerabilities
Framework complexity
```

Modern Java modules can also restrict reflective access across module boundaries.

---

# 53. Modules and Reflection

The Java Platform Module System introduced stronger encapsulation boundaries.

An application may need to explicitly open packages for certain deep reflection scenarios.

Conceptually:

```text
exports
    ↓
normal API visibility

opens
    ↓
deep reflection access
```

This distinction can matter when working with frameworks and modular applications.

---

# 54. Annotation Processing

Annotations can also be processed at compile time rather than runtime.

Examples of technologies that use annotation processing include tools that generate:

```text
Source code
Metadata
Adapters
Builders
Mappers
```

An annotation processor implements:

```java
javax.annotation.processing.Processor
```

or uses the modern processor APIs.

---

# 55. Runtime vs Compile-Time Annotation Processing

### Runtime

```text
Annotation
   ↓
Class file
   ↓
Reflection
   ↓
Framework
```

### Compile-time

```text
Source code
   ↓
Annotation processor
   ↓
Generated code / metadata
   ↓
Compilation
```

Compile-time processing can reduce runtime reflection for some use cases.

---

# 56. Why Frameworks Use Annotations

Annotations provide declarative metadata.

Instead of:

```java
registerService(
    UserService.class
);
```

a framework can allow:

```java
@Service
class UserService {
}
```

The framework interprets the metadata and handles registration.

This can make application code more declarative.

---

# 57. Custom Annotation Example — Audit

Define:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Audit {
}
```

Use:

```java
@Audit
public void updateOrder() {
}
```

A framework or custom interceptor can inspect the method and apply audit behavior.

---

# 58. Custom Annotation with Parameters

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Audit {

    String action();

    boolean includeArguments()
        default false;
}
```

Usage:

```java
@Audit(
    action = "CREATE_ORDER",
    includeArguments = true
)
public void createOrder() {
}
```

---

# 59. Practical Backend Example

Suppose you want to mark methods that require audit logging.

```java
@Audit(action = "DELETE_USER")
public void deleteUser(Long userId) {
}
```

An interceptor/aspect could inspect the annotation and produce:

```text
User ID
Action
Timestamp
Method
Result
```

This is a common conceptual use of custom annotations.

---

# 60. Annotation Design Guidelines

When creating custom annotations:

```text
Choose meaningful names
Define appropriate targets
Choose retention deliberately
Keep metadata focused
Use defaults where useful
Avoid unnecessary annotation complexity
```

---

# 61. Reflection Best Practices

Use reflection when it provides genuine flexibility.

Good use cases:

```text
Framework infrastructure
Plugin discovery
Generic tooling
Testing utilities
Serialization infrastructure
Metadata inspection
```

Avoid reflection when direct Java APIs provide a simpler and safer solution.

---

# 62. Reflection vs Direct Access

### Direct

```java
user.getName();
```

Advantages:

```text
Fast
Type-safe
Readable
Compiler checked
```

### Reflection

```java
method.invoke(user);
```

Advantages:

```text
Dynamic
Generic
Framework-friendly
Metadata-driven
```

Trade-off:

```text
More runtime complexity
Less compile-time safety
Potential performance cost
```

---

# 63. Interview Question

### What is reflection?

### Answer

> Reflection is a Java mechanism that allows an application to inspect and interact with classes, methods, fields and constructors at runtime. Frameworks such as Spring and Hibernate use reflection and related metadata mechanisms to build dynamic behavior.

---

# 64. Interview Question

### Where is reflection used in Spring?

### Answer

> Spring uses reflection and related infrastructure for tasks such as component discovery, dependency injection, invoking lifecycle methods and working with framework metadata. Spring also uses proxies and other mechanisms for features such as transactions and AOP.

---

# 65. Interview Question

### What is an annotation?

### Answer

> An annotation is metadata attached to a Java program element. The compiler, tools or frameworks can inspect that metadata and use it to influence processing or runtime behavior.

---

# 66. Interview Question

### What is @Retention?

### Answer

> `@Retention` defines how long an annotation is retained: source level, class-file level or runtime. Runtime retention is required when the annotation needs to be inspected through reflection at runtime.

---

# 67. Interview Question

### What is @Target?

### Answer

> `@Target` specifies the program elements where an annotation can be used, such as a class, method, field, parameter or constructor.

---

# 68. Interview Question

### Difference between SOURCE, CLASS and RUNTIME?

### Answer

> SOURCE annotations are discarded during compilation, CLASS annotations are stored in the class file but are not normally available through runtime reflection, and RUNTIME annotations remain available for runtime inspection.

---

# 69. Interview Question

### Can we access private fields using reflection?

### Answer

> Reflection can inspect private fields, and older or permitted scenarios can use accessibility APIs to access them. However, Java's module system and strong encapsulation can restrict deep reflective access, so it should not be treated as an unrestricted bypass of access control.

---

# 70. Interview Question

### Why is reflection considered less type-safe?

### Answer

> With normal Java calls, the compiler verifies method names, parameters and types. Reflection resolves these dynamically, so errors such as missing methods or incompatible arguments can appear at runtime instead of compile time.

---

# 71. Interview Question

### Why can reflection be slower?

### Answer

> Reflection performs dynamic metadata lookup and invocation rather than normal statically linked method calls, so it can introduce additional overhead. Frameworks usually manage this overhead, but direct calls are preferable in performance-critical code when dynamic behavior isn't required.

---

# 72. Interview Question

### What is a custom annotation?

### Answer

> A custom annotation is an annotation type defined by the application to attach domain-specific metadata to classes, methods, fields or other program elements.

Example:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Audit {
}
```

---

# 73. Interview Question

### What is @Inherited?

### Answer

> `@Inherited` allows a class-level annotation to be inherited by subclasses when queried through the relevant class APIs. It does not make method or field annotations automatically inherited.

---

# 74. Interview Question

### What is annotation processing?

### Answer

> Annotation processing analyzes annotations during compilation and can generate source code or other artifacts. This allows some framework and tooling behavior to happen at build time instead of runtime.

---

# 75. Reflection + Annotation Example

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Audit {
    String action();
}
```

Class:

```java
class OrderService {

    @Audit(action = "CREATE")
    public void createOrder() {
        System.out.println(
            "Creating order"
        );
    }
}
```

Reflection:

```java
Method method =
    OrderService.class.getDeclaredMethod(
        "createOrder"
    );

Audit audit =
    method.getAnnotation(Audit.class);

if (audit != null) {
    System.out.println(
        audit.action()
    );
}
```

Output:

```text
CREATE
```

---

# 76. How Spring Uses the Same Idea

Conceptually:

```text
@Service
    ↓
Spring discovers metadata
    ↓
Bean definition
    ↓
Object creation
    ↓
Dependency injection
    ↓
Bean lifecycle
```

And:

```text
@Transactional
    ↓
Metadata
    ↓
Proxy/interceptor infrastructure
    ↓
Transaction behavior
```

This is why understanding annotations and reflection helps when learning Spring Boot internals.

---

# 77. Reflection Quick Revision

```text
Class
  ↓
Runtime metadata

Field
  ↓
Inspect/access fields

Method
  ↓
Inspect/invoke methods

Constructor
  ↓
Inspect/create objects

Reflection
  ↓
Dynamic runtime inspection

Annotation
  ↓
Metadata

@Target
  ↓
Where annotation can be used

@Retention
  ↓
How long annotation is retained

SOURCE
  ↓
Source only

CLASS
  ↓
Class file

RUNTIME
  ↓
Runtime reflection

Annotation Processor
  ↓
Compile-time processing
```

---

# 78. Most Important Interview Questions

Be comfortable answering:

1. What is reflection?
2. What is a `Class` object?
3. `.class` vs `getClass()` vs `Class.forName()`?
4. `getFields()` vs `getDeclaredFields()`?
5. `getMethods()` vs `getDeclaredMethods()`?
6. How do you invoke a method using reflection?
7. How do you create an object using reflection?
8. What is `InvocationTargetException`?
9. What is an annotation?
10. What is a custom annotation?
11. What is `@Target`?
12. What is `@Retention`?
13. SOURCE vs CLASS vs RUNTIME?
14. What is `@Inherited`?
15. What is `@Documented`?
16. How does Spring use annotations?
17. How does Spring use reflection?
18. Why is constructor injection preferred?
19. What is annotation processing?
20. Runtime reflection vs compile-time annotation processing?
21. Why can reflection be slower?
22. Why is reflection less type-safe?
23. Can reflection access private fields?
24. What are Java module restrictions on reflection?
25. Where would you use a custom annotation in a backend system?

---

# 79. Final Interview Answer

If asked:

### "Why should a Java backend developer understand reflection and annotations?"

A natural answer:

> I think it's important because modern Java frameworks rely heavily on metadata and runtime infrastructure. Spring uses annotations to describe components and configuration, while reflection and related mechanisms help frameworks inspect classes and manage objects. Understanding these concepts makes it easier to understand what is happening behind annotations such as `@Service`, `@Autowired` and `@Transactional`, instead of treating them as magic.

---

# 80. Practical Rule

Don't try to become a reflection expert just to pass an interview.

Know:

```text
What reflection is
Why frameworks use it
How Class/Field/Method work
What annotations are
@Target
@Retention
Runtime vs compile-time processing
Basic Spring connection
Reflection trade-offs
Security/module restrictions
```

For a Java Backend Developer, understanding **why Spring needs these mechanisms** is more valuable than memorizing every reflection API.
