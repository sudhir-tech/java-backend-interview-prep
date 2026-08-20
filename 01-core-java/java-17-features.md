# Java 17 Features — Java Backend Interview Prep

A practical interview-focused guide to the most important Java 17 features and changes for Java Backend Developer interviews.

---

## 1. Why is Java 17 important?

Java 17 is an **LTS (Long-Term Support)** release.

For backend developers, Java 17 is important because it combines modern language features with a stable LTS runtime and is widely used in enterprise Spring Boot applications.

### Key features to know

- Sealed classes
- Pattern matching for `instanceof`
- Records
- Text blocks
- Switch expressions
- Helpful NullPointerExceptions
- Strong encapsulation of JDK internals
- New pseudo-random number generator APIs
- Enhanced macOS/AArch64 support
- Foreign Function & Memory API as an incubator feature

> **Interview note:** Some features were introduced before Java 17 and became permanent by Java 17. For interviews, know both the feature and the Java version in which it became final.

---

## 2. Records

Records were introduced as a preview feature in Java 14 and became a standard feature in Java 16.

```java
public record User(Long id, String name) {
}
```

The compiler automatically provides:

- Private final record components
- A canonical constructor
- Accessor methods such as `id()` and `name()`
- `equals()`
- `hashCode()`
- `toString()`

Example:

```java
User user = new User(1L, "Sudhir");

System.out.println(user.id());
System.out.println(user.name());
```

### Backend use cases

Records are excellent for:

- REST API request/response DTOs
- Configuration/value objects
- Immutable-style data carriers

Example:

```java
public record UserResponse(
    Long id,
    String name,
    String email
) {
}
```

### Interview point

A record is not simply a replacement for every Java class. It is designed primarily for transparent data carriers.

---

## 3. Compact constructor in records

Records can validate constructor input using a compact constructor.

```java
public record User(String name, int age) {

    public User {
        if (age < 0) {
            throw new IllegalArgumentException(
                "Age cannot be negative"
            );
        }
    }
}
```

The compiler handles assignment to the record components.

---

## 4. Pattern matching for `instanceof`

Pattern matching for `instanceof` became a standard feature in Java 16.

Traditional code:

```java
if (obj instanceof String) {
    String value = (String) obj;
    System.out.println(value.length());
}
```

Modern Java:

```java
if (obj instanceof String value) {
    System.out.println(value.length());
}
```

It combines the type check, cast and variable declaration.

### Why is it useful?

It reduces:

- Boilerplate
- Explicit casting
- Opportunities for casting mistakes

---

## 5. Pattern variable scope

The pattern variable is available only where the compiler can prove that the match succeeded.

```java
if (obj instanceof String value
        && !value.isBlank()) {
    System.out.println(value);
}
```

This behavior is called **flow scoping**.

---

## 6. Sealed classes

Sealed classes were finalized in Java 17.

They allow a class or interface to explicitly control which classes can extend or implement it.

```java
public sealed interface Payment
    permits CardPayment, UpiPayment {
}
```

Implementations:

```java
public final class CardPayment
    implements Payment {
}

public final class UpiPayment
    implements Payment {
}
```

### Why use sealed classes?

They are useful when a domain has a known, restricted set of implementations.

Example:

```text
Payment
 ├── CardPayment
 └── UpiPayment
```

This gives the compiler more knowledge about the type hierarchy.

---

## 7. Rules for sealed classes

A sealed type can restrict its direct subclasses using `permits`.

A permitted subclass must explicitly declare one of:

```text
final
sealed
non-sealed
```

Example:

```java
public sealed class Payment
    permits CardPayment, UpiPayment {
}

public final class CardPayment
    extends Payment {
}

public non-sealed class UpiPayment
    extends Payment {
}
```

### Meaning

`final`:

```text
No further inheritance
```

`sealed`:

```text
Inheritance continues but remains restricted
```

`non-sealed`:

```text
Restriction is removed for that branch
```

---

## 8. Text blocks

Text blocks became a standard feature in Java 15.

They make multiline strings easier to write.

Traditional:

```java
String json = "{\n" +
              "  \"name\": \"Sudhir\"\n" +
              "}";
```

Text block:

```java
String json = """
    {
      "name": "Sudhir"
    }
    """;
```

### Backend use cases

Text blocks are useful for:

- JSON examples
- SQL queries
- HTML templates
- Multiline configuration text
- Test data

Example:

```java
String sql = """
    SELECT id, name, email
    FROM users
    WHERE status = 'ACTIVE'
    ORDER BY name
    """;
```

---

## 9. Switch expressions

Switch expressions became a standard feature in Java 14.

Traditional switch:

```java
String result;

switch (status) {
    case "PAID":
        result = "Payment successful";
        break;
    case "FAILED":
        result = "Payment failed";
        break;
    default:
        result = "Unknown";
}
```

Modern switch expression:

```java
String result = switch (status) {
    case "PAID" -> "Payment successful";
    case "FAILED" -> "Payment failed";
    default -> "Unknown";
};
```

### `yield`

For a multi-statement switch branch:

```java
String result = switch (status) {
    case "PAID" -> "Success";
    case "FAILED" -> {
        String message = "Payment failed";
        yield message;
    }
    default -> "Unknown";
};
```

---

## 10. Helpful NullPointerExceptions

Java 14 improved NullPointerException messages.

Instead of a vague message, the JVM can identify the expression that contained the null reference.

Example:

```java
user.getAddress().getCity().getName();
```

If one of the intermediate references is null, modern JVMs can provide more useful diagnostic information.

### Why is this useful?

It improves:

- Production debugging
- Log analysis
- Root cause analysis
- Developer productivity

---

## 11. Strong encapsulation of JDK internals

Java 17 strongly encapsulates internal JDK APIs.

Application code should avoid relying on unsupported internal APIs such as:

```text
sun.*
com.sun.*
```

Use supported Java SE APIs instead.

This is particularly relevant when upgrading older applications to Java 17.

---

## 12. RandomGenerator API

Java 17 introduced a new interface hierarchy for random number generators through `java.util.random`.

Example:

```java
import java.util.random.RandomGenerator;

RandomGenerator random =
    RandomGenerator.getDefault();

int value = random.nextInt(100);
```

This provides a more extensible model for working with different random-number algorithms.

---

## 13. Foreign Function & Memory API

Java 17 included the Foreign Function & Memory API as an **incubator feature**.

It was designed to allow Java programs to interact with:

- Native code
- Native memory

without relying solely on the older JNI approach.

### Interview note

Do not describe this as a finalized Java 17 feature. In Java 17 it was still an incubator API.

---

## 14. macOS AArch64 support

Java 17 added support for macOS/AArch64, which is important for Apple Silicon systems.

This is especially relevant for developers using Macs with Apple Silicon processors.

---

## 15. Java 17 migration considerations

When upgrading an older Java application to Java 17, check for:

```text
Internal JDK APIs
Old libraries
Reflection against JDK internals
Deprecated APIs
Build tool compatibility
Framework compatibility
```

For Spring Boot applications, also verify that the Spring Boot version and dependencies support Java 17.

---

## 16. Java 17 feature timeline

A common interview trap is assuming every modern Java feature was introduced in Java 17.

| Feature | Introduced | Standardized / Finalized |
|---|---:|---:|
| `var` | Java 10 | Java 10 |
| Switch expressions | Java 12 preview | Java 14 |
| Text blocks | Java 13 preview | Java 15 |
| Pattern matching for `instanceof` | Java 14 preview | Java 16 |
| Records | Java 14 preview | Java 16 |
| Sealed classes | Java 15 preview | Java 17 |
| Pattern matching for `switch` | Java 17 preview | Later Java release |
| Virtual threads | Later preview | Java 21 |

This distinction is important in interviews.

---

## 17. Java 17 vs Java 8

| Java 8 | Java 17 |
|---|---|
| Lambdas | Lambdas + modern language features |
| Streams | Streams + newer APIs |
| Default interface methods | Modern interface capabilities |
| Traditional classes | Records and sealed classes |
| Traditional `instanceof` | Pattern matching for `instanceof` |
| Traditional switch | Switch expressions |
| String concatenation | Text blocks |
| Older JVM capabilities | Improved JVM/runtime features |
| Older JDK internal-access assumptions | Stronger encapsulation |

---

## 18. Why should a backend developer know Java 17?

Java 17 is particularly relevant to backend developers because modern enterprise applications increasingly target LTS Java versions.

You should be comfortable with:

```text
Records
Sealed classes
Pattern matching
Switch expressions
Text blocks
Modern collections and streams
Concurrency fundamentals
JVM behavior
Migration considerations
```

---

# Java 17 Interview Questions

## Q1. Is Java 17 an LTS release?

Yes. Java 17 is a Long-Term Support release.

---

## Q2. Which Java version introduced sealed classes as a final feature?

Java 17.

---

## Q3. Are records a Java 17 feature?

Records became a standard feature in Java 16. They are nevertheless commonly used in Java 17 applications.

---

## Q4. Are switch expressions a Java 17 feature?

No. Switch expressions became final in Java 14.

---

## Q5. Are text blocks a Java 17 feature?

No. Text blocks became final in Java 15.

---

## Q6. What is the difference between a sealed class and a final class?

A `final` class cannot be extended at all.

A sealed class can be extended, but only by explicitly permitted subclasses.

---

## Q7. Can a sealed subclass itself be extended?

Yes, depending on whether it is declared:

```text
final
sealed
non-sealed
```

---

## Q8. What problem do records solve?

They reduce boilerplate when creating data-carrier classes by automatically providing components, accessors, constructors, `equals()`, `hashCode()` and `toString()`.

---

## Q9. What problem does pattern matching for `instanceof` solve?

It removes repetitive casting code and makes type checks more concise and readable.

---

## Q10. What Java 17 feature is important for controlled inheritance?

Sealed classes and interfaces.

---

## Q11. What is one important Java 17 migration concern?

Older applications may depend on internal JDK APIs or libraries that are incompatible with Java 17's stronger encapsulation and newer runtime behavior.

---

## Q12. What Java version introduced virtual threads?

Virtual threads became a standard feature in Java 21, not Java 17.

---

# Backend Interview Examples

## Example 1 — DTO with a record

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

This is a clean choice for an API response DTO when record semantics fit the use case.

---

## Example 2 — Controlled domain hierarchy

```java
public sealed interface Payment
    permits CardPayment, UpiPayment {
}
```

This communicates that the payment domain currently has a known set of implementations.

---

## Example 3 — Modern type check

```java
if (request instanceof CreateOrderRequest order) {
    orderService.create(order);
}
```

This avoids a separate cast.

---

## Example 4 — Modern SQL string

```java
String sql = """
    SELECT id, name
    FROM products
    WHERE category = ?
    ORDER BY name
    """;
```

Text blocks make multiline SQL easier to read in examples and tests.

---

# Quick Revision

```text
Java 17
│
├── LTS release
│
├── Sealed Classes → FINAL in Java 17
│
├── Records → FINAL in Java 16
│
├── Pattern Matching instanceof → FINAL in Java 16
│
├── Text Blocks → FINAL in Java 15
│
├── Switch Expressions → FINAL in Java 14
│
├── Helpful NullPointerExceptions
│
├── Strong encapsulation of JDK internals
│
├── RandomGenerator API
│
├── macOS AArch64 support
│
└── Foreign Function & Memory API → INCUBATOR in Java 17
```

## Interview rule to remember

> **Java 17 is an LTS release, but not every modern Java feature was introduced in Java 17. Know the feature timeline and which features were finalized by Java 17.**
