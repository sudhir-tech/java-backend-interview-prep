# Java 8+ Features

Java 8 introduced major language and API improvements that are still heavily used in Java backend development.

For backend interviews, focus on:

- Lambda expressions
- Functional interfaces
- Stream API
- Optional
- Method references
- Default/static interface methods
- Date and Time API
- Collectors
- `map()` vs `flatMap()`
- `filter()`
- `reduce()`
- `groupingBy()`
- Parallel streams
- Modern Java features after Java 8

---

# 1. Why Java 8 Is Important

Before Java 8, Java code often required verbose anonymous classes.

Java 8 introduced functional programming features such as:

```text
Lambda expressions
Functional interfaces
Streams
Method references
Optional
```

These features are common in Spring Boot and backend code.

---

# 2. Lambda Expression

A lambda is a concise way to represent behavior.

Traditional:

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

Lambda:

```java
Runnable task = () ->
    System.out.println("Running");
```

---

# 3. Lambda Syntax

General form:

```java
(parameters) -> expression
```

or:

```java
(parameters) -> {
    // statements
}
```

Example:

```java
(int a, int b) -> a + b
```

Type inference allows:

```java
(a, b) -> a + b
```

when the target functional interface provides the parameter types.

---

# 4. Functional Interface

A functional interface has exactly one abstract method.

Example:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);
}
```

Use:

```java
Calculator add =
    (a, b) -> a + b;
```

---

# 5. @FunctionalInterface

This annotation tells the compiler that the interface is intended to have one abstract method.

Example:

```java
@FunctionalInterface
interface Printer {

    void print(String value);
}
```

If another abstract method is added, compilation fails.

---

# 6. Common Functional Interfaces

Java provides common functional interfaces in:

```java
java.util.function
```

Important ones:

```text
Predicate<T>
Function<T, R>
Consumer<T>
Supplier<T>
UnaryOperator<T>
BinaryOperator<T>
```

---

# 7. Predicate

`Predicate<T>` takes a value and returns:

```text
boolean
```

Example:

```java
Predicate<Integer> even =
    n -> n % 2 == 0;
```

Usage:

```java
even.test(10);
```

Result:

```text
true
```

Commonly used with:

```java
filter()
```

---

# 8. Function

`Function<T, R>` converts one value into another.

Example:

```java
Function<String, Integer> length =
    value -> value.length();
```

Usage:

```java
int length =
    length.apply("Java");
```

Result:

```text
4
```

Commonly used with:

```java
map()
```

---

# 9. Consumer

`Consumer<T>` accepts a value and returns nothing.

Example:

```java
Consumer<String> printer =
    value -> System.out.println(value);
```

Usage:

```java
printer.accept("Java");
```

Commonly used for:

```java
forEach()
```

---

# 10. Supplier

`Supplier<T>` takes no input and returns a value.

Example:

```java
Supplier<Double> random =
    () -> Math.random();
```

Usage:

```java
Double value =
    random.get();
```

---

# 11. UnaryOperator

`UnaryOperator<T>` takes and returns the same type.

Example:

```java
UnaryOperator<Integer> square =
    n -> n * n;
```

Equivalent conceptually to:

```text
Function<Integer, Integer>
```

---

# 12. BinaryOperator

`BinaryOperator<T>` takes two values of the same type and returns the same type.

Example:

```java
BinaryOperator<Integer> sum =
    (a, b) -> a + b;
```

---

# 13. Method Reference

Method references provide a shorter syntax for certain lambdas.

Lambda:

```java
names.forEach(
    name -> System.out.println(name)
);
```

Method reference:

```java
names.forEach(
    System.out::println
);
```

---

# 14. Types of Method References

Common forms:

```text
Static method
instance method of a particular object
instance method of an arbitrary object of a type
constructor
```

Examples:

```java
Math::abs
System.out::println
String::toUpperCase
ArrayList::new
```

---

# 15. Constructor Reference

Instead of:

```java
Supplier<List<String>> supplier =
    () -> new ArrayList<>();
```

you can write:

```java
Supplier<List<String>> supplier =
    ArrayList::new;
```

---

# 16. Stream API

Streams provide a declarative way to process collections and other data sources.

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);

List<Integer> result =
    numbers.stream()
        .filter(n -> n % 2 == 0)
        .toList();
```

Result:

```text
[2, 4]
```

---

# 17. Stream Pipeline

A stream pipeline usually contains:

```text
Source
  ↓
Intermediate operations
  ↓
Terminal operation
```

Example:

```java
numbers.stream()
    .filter(n -> n > 10)
    .map(n -> n * 2)
    .toList();
```

---

# 18. Stream Source

A stream can come from:

```java
list.stream();
```

or:

```java
Arrays.stream(array);
```

or:

```java
Stream.of("Java", "Spring");
```

---

# 19. Intermediate Operations

Intermediate operations return another stream.

Examples:

```text
filter()
map()
flatMap()
distinct()
sorted()
limit()
skip()
peek()
```

They are generally lazy.

---

# 20. Terminal Operations

Terminal operations produce a result or side effect and trigger stream processing.

Examples:

```text
collect()
toList()
forEach()
reduce()
count()
min()
max()
findFirst()
findAny()
anyMatch()
allMatch()
noneMatch()
```

After a terminal operation, the stream cannot be reused.

---

# 21. filter()

`filter()` keeps elements matching a condition.

Example:

```java
List<Integer> result =
    numbers.stream()
        .filter(n -> n > 10)
        .toList();
```

If:

```text
[5, 10, 15, 20]
```

result:

```text
[15, 20]
```

---

# 22. map()

`map()` transforms each element.

Example:

```java
List<String> names =
    List.of("java", "spring");

List<String> result =
    names.stream()
        .map(String::toUpperCase)
        .toList();
```

Result:

```text
[JAVA, SPRING]
```

---

# 23. filter() vs map()

### filter

Changes the number of elements by selecting some.

```text
[1,2,3,4]
   ↓ filter even
[2,4]
```

### map

Transforms elements.

```text
[1,2,3]
   ↓ map n * 2
[2,4,6]
```

---

# 24. flatMap()

`flatMap()` is used when each input element produces multiple values and you want one flattened stream.

Example:

```java
List<List<Integer>> values =
    List.of(
        List.of(1, 2),
        List.of(3, 4)
    );

List<Integer> result =
    values.stream()
        .flatMap(List::stream)
        .toList();
```

Result:

```text
[1, 2, 3, 4]
```

---

# 25. map() vs flatMap()

`map()`:

```text
List<List<Integer>>
       ↓
map
       ↓
Stream<List<Integer>>
```

`flatMap()`:

```text
List<List<Integer>>
       ↓
flatMap
       ↓
Stream<Integer>
```

### Interview Answer

> `map()` transforms one element into one result, while `flatMap()` is useful when each element produces a stream or collection and we want to flatten the results into a single stream.

---

# 26. distinct()

Removes duplicates based on equality semantics.

Example:

```java
List<Integer> result =
    List.of(1, 2, 2, 3, 3)
        .stream()
        .distinct()
        .toList();
```

Result:

```text
[1, 2, 3]
```

---

# 27. sorted()

Sorts stream elements.

Example:

```java
List<Integer> result =
    numbers.stream()
        .sorted()
        .toList();
```

Custom comparator:

```java
numbers.stream()
    .sorted(Comparator.reverseOrder())
    .toList();
```

---

# 28. limit()

Limits the number of elements.

```java
numbers.stream()
    .limit(3)
    .toList();
```

---

# 29. skip()

Skips the first N elements.

```java
numbers.stream()
    .skip(2)
    .toList();
```

Useful in some pagination-like processing, although database-level pagination is usually preferable for large datasets.

---

# 30. forEach()

Example:

```java
names.stream()
    .forEach(System.out::println);
```

For simple collection iteration, a normal enhanced `for` loop may sometimes be clearer.

Avoid using `forEach()` as a replacement for every kind of loop.

---

# 31. reduce()

`reduce()` combines stream elements into one result.

Example:

```java
int sum =
    numbers.stream()
        .reduce(0, Integer::sum);
```

For:

```text
[1, 2, 3, 4]
```

result:

```text
10
```

---

# 32. reduce() Without Identity

Example:

```java
Optional<Integer> result =
    numbers.stream()
        .reduce(Integer::sum);
```

The result is `Optional<Integer>` because the stream might be empty.

---

# 33. count()

Example:

```java
long count =
    numbers.stream()
        .filter(n -> n > 10)
        .count();
```

---

# 34. min() and max()

Example:

```java
Optional<Integer> min =
    numbers.stream()
        .min(Integer::compareTo);

Optional<Integer> max =
    numbers.stream()
        .max(Integer::compareTo);
```

Because there may be no element, the result is optional.

---

# 35. findFirst()

Example:

```java
Optional<Integer> first =
    numbers.stream()
        .filter(n -> n > 10)
        .findFirst();
```

---

# 36. findAny()

Example:

```java
Optional<Integer> value =
    numbers.parallelStream()
        .filter(n -> n > 10)
        .findAny();
```

`findAny()` does not promise the first matching element, which allows more flexibility in parallel execution.

---

# 37. anyMatch()

Returns true if any element matches.

```java
boolean exists =
    numbers.stream()
        .anyMatch(n -> n > 100);
```

---

# 38. allMatch()

Returns true if all elements match.

```java
boolean valid =
    numbers.stream()
        .allMatch(n -> n > 0);
```

---

# 39. noneMatch()

Returns true if no element matches.

```java
boolean valid =
    numbers.stream()
        .noneMatch(n -> n < 0);
```

---

# 40. Collectors

`Collectors` provides useful reduction operations.

Example:

```java
List<String> result =
    names.stream()
        .filter(...)
        .collect(Collectors.toList());
```

In modern Java, you can often use:

```java
.toList()
```

when an unmodifiable result is acceptable.

---

# 41. Collecting to Set

```java
Set<String> result =
    names.stream()
        .collect(
            Collectors.toSet()
        );
```

---

# 42. Joining

Example:

```java
String result =
    names.stream()
        .collect(
            Collectors.joining(", ")
        );
```

If:

```text
Java
Spring
SQL
```

result:

```text
Java, Spring, SQL
```

---

# 43. groupingBy()

One of the most important collector operations for interviews.

Example:

```java
Map<String, List<Employee>> byDepartment =
    employees.stream()
        .collect(
            Collectors.groupingBy(
                Employee::getDepartment
            )
        );
```

Conceptually:

```text
Employees
   ↓
group by department
   ↓
Map<Department, List<Employee>>
```

---

# 44. groupingBy() with Counting

Example:

```java
Map<String, Long> countByDepartment =
    employees.stream()
        .collect(
            Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
            )
        );
```

Result conceptually:

```text
Engineering → 10
HR          → 5
Finance     → 7
```

---

# 45. groupingBy() with Summing

Example:

```java
Map<String, Integer> salaryByDepartment =
    employees.stream()
        .collect(
            Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summingInt(
                    Employee::getSalary
                )
            )
        );
```

---

# 46. partitioningBy()

`partitioningBy()` divides elements into two groups based on a boolean predicate.

Example:

```java
Map<Boolean, List<Integer>> result =
    numbers.stream()
        .collect(
            Collectors.partitioningBy(
                n -> n % 2 == 0
            )
        );
```

Conceptually:

```text
true  → even numbers
false → odd numbers
```

---

# 47. toMap()

Example:

```java
Map<Long, Employee> employeesById =
    employees.stream()
        .collect(
            Collectors.toMap(
                Employee::getId,
                employee -> employee
            )
        );
```

Important:

If duplicate keys are possible, provide a merge function.

```java
Collectors.toMap(
    Employee::getId,
    employee -> employee,
    (existing, replacement) ->
        existing
)
```

---

# 48. Optional

`Optional<T>` represents a value that may or may not be present.

Example:

```java
Optional<String> name =
    Optional.of("Sudhir");
```

Empty:

```java
Optional<String> name =
    Optional.empty();
```

---

# 49. Why Optional?

Without Optional:

```java
String name =
    findName();

if (name != null) {
    ...
}
```

Optional can make the absence explicit:

```java
Optional<String> name =
    findName();
```

It can reduce some null-related errors when used appropriately.

---

# 50. Optional.of()

Use when the value is known to be non-null.

```java
Optional<String> value =
    Optional.of("Java");
```

If the value is null:

```java
Optional.of(null);
```

throws:

```text
NullPointerException
```

---

# 51. Optional.ofNullable()

Use when a value may be null.

```java
Optional<String> value =
    Optional.ofNullable(name);
```

If `name` is null:

```text
Optional.empty()
```

---

# 52. isPresent()

Example:

```java
if (value.isPresent()) {
    System.out.println(
        value.get()
    );
}
```

However, repeatedly using:

```java
isPresent()
get()
```

often recreates traditional null-checking.

Prefer operations such as:

```java
map()
orElse()
orElseGet()
ifPresent()
orElseThrow()
```

when appropriate.

---

# 53. orElse()

Example:

```java
String result =
    value.orElse("Unknown");
```

If the Optional is empty:

```text
Unknown
```

---

# 54. orElseGet()

Example:

```java
String result =
    value.orElseGet(
        () -> loadDefaultName()
    );
```

The supplier is evaluated only when the Optional is empty.

---

# 55. orElse() vs orElseGet()

This is a common interview question.

```java
value.orElse(expensiveOperation());
```

The argument can be evaluated even if the Optional already contains a value.

With:

```java
value.orElseGet(
    () -> expensiveOperation()
);
```

the supplier is invoked only when the value is absent.

### Interview Answer

> `orElse()` receives an already evaluated fallback value, while `orElseGet()` takes a supplier and evaluates it only when the Optional is empty.

---

# 56. orElseThrow()

Example:

```java
User user =
    repository.findById(id)
        .orElseThrow(
            () -> new UserNotFoundException(
                "User not found"
            )
        );
```

This is very common in Spring Boot services.

---

# 57. Optional map()

Example:

```java
Optional<String> upper =
    Optional.of("java")
        .map(String::toUpperCase);
```

Result:

```text
Optional[JAVA]
```

---

# 58. Optional flatMap()

Use `flatMap()` when the mapping function already returns an Optional.

Example:

```java
Optional<Address> address =
    userOptional.flatMap(
        User::getAddress
    );
```

If `getAddress()` returns:

```java
Optional<Address>
```

`flatMap()` avoids:

```text
Optional<Optional<Address>>
```

---

# 59. Optional Best Practices

Good:

```java
Optional<User> findById(
    Long id
);
```

Useful when absence is a valid result.

Avoid using Optional as every field type:

```java
class User {
    Optional<String> name;
}
```

It is generally not the preferred Java domain-model pattern.

Also avoid:

```java
Optional<String> optional =
    Optional.of(...);

optional.get();
```

without checking presence or using a safer terminal operation.

---

# 60. Date and Time API

Java 8 introduced:

```text
java.time
```

Important classes:

```text
LocalDate
LocalTime
LocalDateTime
Instant
ZonedDateTime
Duration
Period
DateTimeFormatter
```

---

# 61. LocalDate

Represents a date without time or timezone.

```java
LocalDate today =
    LocalDate.now();
```

Example:

```java
LocalDate date =
    LocalDate.of(
        2026,
        8,
        20
    );
```

---

# 62. LocalTime

Represents a time without date or timezone.

```java
LocalTime now =
    LocalTime.now();
```

---

# 63. LocalDateTime

Represents date + time without timezone.

```java
LocalDateTime now =
    LocalDateTime.now();
```

Important:

> LocalDateTime does not identify a unique instant globally because it has no timezone/offset.

---

# 64. Instant

`Instant` represents a point on the UTC timeline.

Example:

```java
Instant now =
    Instant.now();
```

This is often useful for timestamps stored or compared across distributed systems.

---

# 65. ZonedDateTime

Represents date and time with a timezone.

Example:

```java
ZonedDateTime now =
    ZonedDateTime.now(
        ZoneId.of("Asia/Kolkata")
    );
```

Useful when business logic depends on local timezone rules.

---

# 66. Duration

Used for time-based amounts.

Example:

```java
Duration duration =
    Duration.between(
        start,
        end
    );
```

Common for:

```text
Seconds
Minutes
Hours
```

---

# 67. Period

Used for date-based amounts.

Example:

```java
Period period =
    Period.between(
        startDate,
        endDate
    );
```

Common for:

```text
Years
Months
Days
```

---

# 68. DateTimeFormatter

Example:

```java
DateTimeFormatter formatter =
    DateTimeFormatter.ofPattern(
        "yyyy-MM-dd"
    );

String formatted =
    LocalDate.now()
        .format(formatter);
```

---

# 69. Why java.time Is Better

The modern Date/Time API is generally preferred over the old:

```text
java.util.Date
java.util.Calendar
```

because it provides:

- Better API design
- Immutable types
- Thread-safe formatters
- Explicit timezone concepts
- Clearer date/time semantics

---

# 70. Interface Default Methods

Java 8 allows interfaces to contain default methods.

Example:

```java
interface PaymentService {

    default void logPayment() {
        System.out.println(
            "Payment started"
        );
    }
}
```

A class implementing the interface can inherit the default implementation.

---

# 71. Why Default Methods?

They allow interfaces to evolve without forcing every existing implementation to implement a newly added method.

This was important for evolving Java's collection APIs.

---

# 72. Static Methods in Interfaces

Java 8 also allows static methods in interfaces.

Example:

```java
interface MathUtil {

    static int add(int a, int b) {
        return a + b;
    }
}
```

Call:

```java
MathUtil.add(10, 20);
```

---

# 73. Interface Static vs Default Method

### Default

Called through an implementing object:

```java
object.defaultMethod();
```

### Static

Called through the interface:

```java
InterfaceName.staticMethod();
```

---

# 74. Stream Laziness

Intermediate stream operations are lazy.

Example:

```java
Stream<Integer> stream =
    numbers.stream()
        .filter(n -> {
            System.out.println(n);
            return n > 2;
        });
```

At this point, the filter may not execute.

Execution starts when a terminal operation is called:

```java
stream.toList();
```

---

# 75. Why Streams Are Lazy

Laziness allows the stream pipeline to:

- Avoid unnecessary work
- Process operations as needed
- Support short-circuiting
- Compose multiple operations efficiently

---

# 76. Short-Circuiting Operations

Examples:

```text
findFirst()
findAny()
anyMatch()
allMatch()
noneMatch()
limit()
```

Example:

```java
boolean exists =
    numbers.stream()
        .anyMatch(n -> n > 100);
```

The stream can stop once a matching value is found.

---

# 77. Stream Cannot Be Reused

Example:

```java
Stream<String> stream =
    names.stream();

stream.count();

stream.forEach(
    System.out::println
);
```

The second terminal operation fails because a stream is consumable.

Create a new stream when needed:

```java
names.stream()
    .forEach(...);
```

---

# 78. Collection vs Stream

### Collection

Stores data.

```text
List
Set
Map
```

### Stream

Processes data.

```text
filter
map
reduce
collect
```

A stream does not normally store its own collection of elements.

---

# 79. Parallel Stream

Example:

```java
numbers.parallelStream()
    .map(...)
    .toList();
```

Parallel streams use the common ForkJoinPool by default for parallel stream processing.

---

# 80. Should We Always Use Parallel Streams?

No.

Parallel streams can hurt performance when:

- Dataset is small
- Operations are cheap
- Work is I/O-bound
- Shared state causes contention
- Ordering requirements reduce benefits
- The common pool is already heavily used

Measure before using them.

---

# 81. Stream Side Effects

Avoid mutable shared state inside streams.

Bad:

```java
List<Integer> result =
    new ArrayList<>();

numbers.parallelStream()
    .forEach(
        n -> result.add(n)
    );
```

This can cause concurrency problems.

Prefer collectors:

```java
List<Integer> result =
    numbers.parallelStream()
        .filter(...)
        .toList();
```

---

# 82. Stream Example — Employee Salaries

Suppose:

```java
List<Employee> employees;
```

Find employees earning more than 100000:

```java
List<Employee> result =
    employees.stream()
        .filter(
            employee ->
                employee.getSalary() > 100000
        )
        .toList();
```

---

# 83. Stream Example — Employee Names

```java
List<String> names =
    employees.stream()
        .map(Employee::getName)
        .toList();
```

---

# 84. Stream Example — Highest Salary

```java
Optional<Employee> employee =
    employees.stream()
        .max(
            Comparator.comparing(
                Employee::getSalary
            )
        );
```

---

# 85. Stream Example — Group Employees

```java
Map<String, List<Employee>> grouped =
    employees.stream()
        .collect(
            Collectors.groupingBy(
                Employee::getDepartment
            )
        );
```

---

# 86. Stream Example — Count by Department

```java
Map<String, Long> counts =
    employees.stream()
        .collect(
            Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
            )
        );
```

---

# 87. Stream Example — Average Salary

```java
double average =
    employees.stream()
        .collect(
            Collectors.averagingInt(
                Employee::getSalary
            )
        );
```

---

# 88. Stream Example — Duplicate Values

Find duplicates:

```java
Set<Integer> seen =
    new HashSet<>();

Set<Integer> duplicates =
    numbers.stream()
        .filter(
            n -> !seen.add(n)
        )
        .collect(
            Collectors.toSet()
        );
```

This uses mutable state, so it should be used carefully and is not a good pattern for parallel streams.

---

# 89. Stream Example — Sort Employees

```java
List<Employee> result =
    employees.stream()
        .sorted(
            Comparator.comparing(
                Employee::getSalary
            ).reversed()
        )
        .toList();
```

---

# 90. Stream Example — Top 3 Salaries

```java
List<Integer> top3 =
    employees.stream()
        .map(Employee::getSalary)
        .sorted(Comparator.reverseOrder())
        .limit(3)
        .toList();
```

If you need distinct salaries:

```java
List<Integer> top3 =
    employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .limit(3)
        .toList();
```

---

# 91. Stream Example — Flatten Nested Lists

```java
List<List<String>> categories =
    ...;

List<String> products =
    categories.stream()
        .flatMap(List::stream)
        .toList();
```

---

# 92. Java 9 — Useful Additions

After Java 8, Java continued adding useful APIs.

Java 9 introduced features such as:

```text
List.of()
Set.of()
Map.of()
Stream.takeWhile()
Stream.dropWhile()
Stream.iterate() enhancements
Optional.ifPresentOrElse()
```

Example:

```java
List<String> values =
    List.of("Java", "Spring");
```

---

# 93. Java 10 — var

Java 10 introduced local variable type inference:

```java
var name = "Sudhir";
```

The compiler infers:

```text
String
```

This does not make Java dynamically typed.

The variable still has a compile-time type.

---

# 94. var Example

Instead of:

```java
Map<String, List<Employee>> employees =
    new HashMap<>();
```

you can write:

```java
var employees =
    new HashMap<String, List<Employee>>();
```

Use `var` when it improves readability.

Avoid it when the inferred type is unclear.

---

# 95. Java 11 — Useful APIs

Java 11 added useful String methods such as:

```java
" ".isBlank();

"Java\n".strip();

"Java".repeat(3);

"a\nb".lines();
```

Java 11 also added the standard HTTP Client API.

---

# 96. Java 14/16 — switch Expressions

Modern Java supports switch expressions.

Example:

```java
String result =
    switch (status) {
        case "SUCCESS" -> "Completed";
        case "FAILED" -> "Failed";
        default -> "Unknown";
    };
```

This can be cleaner than older switch statements.

---

# 97. Java 15+ — Text Blocks

Text blocks make multiline strings easier to write.

Example:

```java
String json = """
    {
      "name": "Sudhir",
      "role": "Java Backend Developer"
    }
    """;
```

Useful for:

- JSON
- SQL
- Multiline text

---

# 98. Java 16 — Records

Records provide a concise way to model data carriers.

Example:

```java
public record UserResponse(
    Long id,
    String name
) {}
```

The compiler provides methods such as:

```text
accessors
equals()
hashCode()
toString()
```

Records are especially useful for DTO-like data.

---

# 99. Java 17 — Sealed Classes

Sealed classes restrict which classes can extend or implement a type.

Example:

```java
public sealed interface Payment
    permits CardPayment,
            CashPayment {
}
```

This can make domain hierarchies more explicit.

---

# 100. Java 21 — Pattern Matching and Modern Concurrency

Java 21 includes important modern features such as:

```text
Pattern matching for switch
Record patterns
Virtual threads
Sequenced collections
```

Example virtual thread:

```java
Thread.startVirtualThread(
    () -> process()
);
```

For backend interviews, Java 17 and Java 21 features are particularly worth knowing because they are common long-term support releases.

---

# 101. Java 8 Interview Question — What Is a Functional Interface?

### Answer

> A functional interface is an interface with exactly one abstract method. It can be used as the target type of a lambda expression. Examples include Runnable, Predicate, Function and Consumer.

---

# 102. Java 8 Interview Question — What Is a Lambda?

### Answer

> A lambda is a concise way to represent behavior that can be passed around as a value. It is commonly used with functional interfaces and the Stream API.

---

# 103. Java 8 Interview Question — What Is Stream API?

### Answer

> Stream API provides a declarative way to process data from collections and other sources using operations such as filter, map, flatMap, reduce and collect. Streams are lazy for intermediate operations and are executed when a terminal operation is invoked.

---

# 104. Java 8 Interview Question — map vs flatMap?

### Answer

> `map()` transforms each element into another value, while `flatMap()` is useful when each element produces multiple values or another stream and we need to flatten the result into one stream.

---

# 105. Java 8 Interview Question — Intermediate vs Terminal?

### Answer

> Intermediate operations return another stream and are generally lazy, such as `filter()` and `map()`. Terminal operations produce the final result or side effect, such as `collect()`, `count()`, `reduce()` and `forEach()`.

---

# 106. Java 8 Interview Question — Why Optional?

### Answer

> Optional represents the presence or absence of a value explicitly. It can make APIs clearer and reduce some null-related errors, especially for methods where no result is a valid outcome.

---

# 107. Java 8 Interview Question — orElse vs orElseGet?

### Answer

> `orElse()` receives a fallback value that can be evaluated immediately, while `orElseGet()` receives a supplier and evaluates it only when the Optional is empty. `orElseGet()` is preferable when the fallback operation is expensive.

---

# 108. Java 8 Interview Question — Why Are Streams Lazy?

### Answer

> Laziness allows the JVM to process the pipeline only when needed, avoid unnecessary work and take advantage of short-circuiting operations such as `findFirst()` and `anyMatch()`.

---

# 109. Java 8 Interview Question — Can a Stream Be Reused?

### Answer

> No. A stream is consumed by a terminal operation. After that, attempting another terminal operation on the same stream results in an IllegalStateException. We should create a new stream from the source.

---

# 110. Java 8 Interview Question — Are Streams Faster Than Loops?

### Answer

Not automatically.

Streams can improve readability and expressiveness, but performance depends on:

- Dataset size
- Operation complexity
- Allocation
- Pipeline design
- Parallelism
- JVM optimizations

Use the approach that provides clear, correct code and measure performance when it matters.

---

# 111. Java 8 Interview Question — Why Should Parallel Streams Be Used Carefully?

### Answer

> Parallel streams use shared execution resources and can introduce overhead, contention and unexpected interactions with other work. They are not automatically faster, especially for small datasets or I/O-bound operations. I would benchmark before using them for performance-sensitive code.

---

# 112. Java 8 Interview Question — What Is a Method Reference?

### Answer

> A method reference is a shorter syntax for a lambda when the lambda simply calls an existing method. For example, `name -> System.out.println(name)` can become `System.out::println`.

---

# 113. Java 8 Interview Question — What Is a Default Method?

### Answer

> A default method allows an interface to provide an implementation. Java 8 introduced it so interfaces could evolve without requiring every existing implementation to immediately implement a newly added method.

---

# 114. Java 8 Interview Question — What Is a Functional Interface Example?

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);
}
```

Usage:

```java
Calculator calculator =
    (a, b) -> a + b;
```

---

# 115. Backend Example — Stream API

Suppose an API returns:

```java
List<Product> products;
```

You need active products:

```java
List<Product> activeProducts =
    products.stream()
        .filter(Product::isActive)
        .toList();
```

This is concise and readable.

---

# 116. Backend Example — DTO Mapping

Suppose:

```java
List<Product> products;
```

You need response DTOs:

```java
List<ProductResponse> response =
    products.stream()
        .map(product ->
            new ProductResponse(
                product.getId(),
                product.getName(),
                product.getPrice()
            )
        )
        .toList();
```

This pattern is common in Spring Boot service layers.

---

# 117. Backend Example — Optional Repository

Spring Data repositories commonly return:

```java
Optional<Product>
```

Example:

```java
Product product =
    productRepository.findById(id)
        .orElseThrow(
            () -> new ProductNotFoundException(
                "Product not found"
            )
        );
```

This is cleaner than:

```java
Product product =
    productRepository.findById(id);

if (product == null) {
    ...
}
```

---

# 118. Backend Example — Grouping

Suppose an order contains items.

You may need:

```text
Product category
      ↓
Order items
```

Streams and collectors can group data:

```java
Map<String, List<OrderItem>> grouped =
    items.stream()
        .collect(
            Collectors.groupingBy(
                OrderItem::getCategory
            )
        );
```

---

# 119. Backend Example — Async Operations

Suppose two independent services are called:

```text
Product Service
Payment Service
```

They may potentially be executed concurrently:

```java
CompletableFuture<Product> product =
    getProduct();

CompletableFuture<Payment> payment =
    getPayment();

return product.thenCombine(
    payment,
    (p, pay) ->
        createResponse(p, pay)
);
```

But concurrency should be used only when the operations are actually independent and downstream capacity can handle it.

---

# 120. Common Mistakes

Avoid:

```text
Using streams everywhere
Using parallelStream blindly
Calling Optional.get() without checking
Using Optional as every field type
Creating unnecessary intermediate collections
Using side effects in stream pipelines
Using complex nested streams that hurt readability
Ignoring nullability and API contracts
Using LocalDateTime when a timezone-aware instant is required
```

---

# 121. Java 8+ Quick Revision

```text
Lambda
    ↓
Concise behavior

Functional Interface
    ↓
One abstract method

Predicate
    ↓
T → boolean

Function
    ↓
T → R

Consumer
    ↓
T → void

Supplier
    ↓
() → T

Stream
    ↓
Declarative data processing

filter
    ↓
Select

map
    ↓
Transform

flatMap
    ↓
Flatten

reduce
    ↓
Combine

collect
    ↓
Build result

Optional
    ↓
Represent possible absence

Method Reference
    ↓
Shorter lambda syntax

Default Method
    ↓
Implementation in interface

java.time
    ↓
Modern date/time API

var
    ↓
Local variable type inference

record
    ↓
Concise data carrier

sealed
    ↓
Restricted hierarchy

virtual thread
    ↓
Lightweight high-concurrency execution
```

---

# 122. Interview Strategy

For Java backend interviews, don't just memorize syntax.

Be able to explain:

```text
Why use streams?
Why use Optional?
When is flatMap needed?
When should parallel streams be avoided?
Why is orElseGet different?
How does groupingBy work?
How does CompletableFuture help?
When should you use records?
When is LocalDateTime inappropriate?
When are virtual threads useful?
```

The strongest answers connect the language feature to a real backend problem.

For example:

> I use streams when they make collection transformations clearer, Optional when absence is part of an API contract, CompletableFuture when independent asynchronous operations can be composed, and records for immutable data-carrier types such as API DTOs when their semantics fit the model.
