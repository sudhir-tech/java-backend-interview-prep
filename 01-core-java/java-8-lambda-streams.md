# Java 8 — Lambda Expressions, Functional Interfaces and Streams

Java 8 introduced several features that changed how Java applications are written.

The most important Java 8 topics for backend interviews are:

- Lambda expressions
- Functional interfaces
- Method references
- Stream API
- Optional
- Default and static methods in interfaces
- `forEach()`
- Collectors
- `map()`, `filter()`, `reduce()`
- `flatMap()`
- Sorting with lambdas
- Parallel streams

---

# 1. Lambda Expressions

A lambda expression is a concise way to represent a function that can be passed around as a value.

Before Java 8:

```java
Runnable runnable = new Runnable() {

    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

With a lambda:

```java
Runnable runnable = () -> {
    System.out.println("Running");
};
```

For a single statement:

```java
Runnable runnable = () ->
        System.out.println("Running");
```

### Basic Syntax

```text
(parameters) -> expression
```

or:

```text
(parameters) -> {
    statements;
}
```

Example:

```java
(a, b) -> a + b
```

---

# 2. Lambda with Parameters

```java
(a, b) -> a + b
```

Example:

```java
BiFunction<Integer, Integer, Integer> add =
        (a, b) -> a + b;

System.out.println(add.apply(10, 20));
```

Output:

```text
30
```

---

# 3. Lambda with One Parameter

Parentheses can usually be omitted for a single inferred parameter.

```java
name -> name.toUpperCase()
```

Equivalent to:

```java
(name) -> name.toUpperCase()
```

---

# 4. Lambda with Multiple Statements

```java
(a, b) -> {

    int result = a + b;

    return result;
}
```

When braces are used, a `return` statement is required when the lambda returns a value.

---

# 5. Why Were Lambdas Introduced?

Lambdas reduce boilerplate and make it easier to pass behavior as an argument.

They are heavily used with:

- Collections
- Streams
- Functional interfaces
- Event processing
- Sorting
- Filtering
- Mapping

### Interview Answer

> Lambda expressions provide a concise way to represent behavior and pass functions as values, especially when working with functional interfaces and the Stream API.

---

# 6. Functional Interface

A functional interface is an interface with **exactly one abstract method**.

It can still have:

- Default methods
- Static methods
- Methods inherited from `Object`

Example:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);
}
```

Usage:

```java
Calculator addition =
        (a, b) -> a + b;

System.out.println(
        addition.calculate(10, 20)
);
```

Output:

```text
30
```

### Interview Answer

> A functional interface is an interface with exactly one abstract method and can be used as the target type of a lambda expression.

---

# 7. @FunctionalInterface

The annotation:

```java
@FunctionalInterface
```

tells the compiler that the interface is intended to have one abstract method.

Example:

```java
@FunctionalInterface
interface Greeting {

    void greet(String name);
}
```

If another abstract method is added, the compiler reports an error.

The annotation is not required for an interface to be functional, but it provides compile-time verification of the intent.

---

# 8. Built-in Functional Interfaces

Java provides common functional interfaces in:

```java
java.util.function
```

Important ones include:

- Predicate
- Function
- Consumer
- Supplier
- UnaryOperator
- BinaryOperator
- BiFunction
- BiConsumer
- BiPredicate

---

# 9. Predicate

`Predicate<T>` accepts a value and returns a boolean.

```java
Predicate<Integer> isEven =
        number -> number % 2 == 0;

System.out.println(isEven.test(10));
```

Output:

```text
true
```

### Signature

```java
boolean test(T value)
```

### Use Case

Useful for:

- Filtering
- Validation
- Conditions

---

# 10. Function

`Function<T, R>` accepts a value of type `T` and returns a value of type `R`.

```java
Function<String, Integer> length =
        text -> text.length();

System.out.println(length.apply("Java"));
```

Output:

```text
4
```

### Signature

```java
R apply(T value)
```

---

# 11. Consumer

`Consumer<T>` accepts a value and returns nothing.

```java
Consumer<String> printer =
        value -> System.out.println(value);

printer.accept("Java");
```

### Signature

```java
void accept(T value)
```

### Use Case

Useful when performing an action such as:

- Logging
- Printing
- Updating state

---

# 12. Supplier

`Supplier<T>` takes no input and returns a value.

```java
Supplier<String> supplier =
        () -> "Java";

System.out.println(supplier.get());
```

### Signature

```java
T get()
```

### Use Case

Useful for:

- Lazy value creation
- Generating values
- Deferred operations

---

# 13. Functional Interface Quick Revision

| Interface | Input | Output | Main Method |
|---|---|---|---|
| Predicate<T> | T | boolean | test() |
| Function<T,R> | T | R | apply() |
| Consumer<T> | T | void | accept() |
| Supplier<T> | none | T | get() |
| BiFunction<T,U,R> | T,U | R | apply() |
| BiConsumer<T,U> | T,U | void | accept() |
| UnaryOperator<T> | T | T | apply() |
| BinaryOperator<T> | T,T | T | apply() |

Easy memory trick:

```text
Predicate → condition

Function → transform

Consumer → use

Supplier → provide
```

---

# 14. Method References

Method references provide a shorter syntax for certain lambdas.

Example:

```java
List<String> names =
        List.of("Sudhir", "Alex", "John");

names.forEach(name ->
        System.out.println(name));
```

Can become:

```java
names.forEach(System.out::println);
```

### Common Forms

```text
ClassName::staticMethod

object::instanceMethod

ClassName::instanceMethod

ClassName::new
```

---

# 15. Constructor Reference

```java
Supplier<User> supplier =
        User::new;
```

This is equivalent to:

```java
Supplier<User> supplier =
        () -> new User();
```

---

# 16. Stream API

The Stream API allows us to process collections and other data sources using a declarative pipeline.

Example:

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5, 6);

List<Integer> evenNumbers =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .toList();
```

Result:

```text
[2, 4, 6]
```

A stream pipeline commonly consists of:

```text
Source
  ↓
Intermediate Operations
  ↓
Terminal Operation
```

---

# 17. Creating a Stream

From a collection:

```java
List<String> names =
        List.of("Java", "Spring");

Stream<String> stream =
        names.stream();
```

From an array:

```java
int[] numbers = {1, 2, 3, 4};

IntStream stream =
        Arrays.stream(numbers);
```

From values:

```java
Stream<String> stream =
        Stream.of("Java", "Spring", "SQL");
```

---

# 18. Intermediate vs Terminal Operations

### Intermediate Operations

Examples:

- filter()
- map()
- flatMap()
- sorted()
- distinct()
- limit()
- skip()
- peek()

They return another stream and are generally lazy.

### Terminal Operations

Examples:

- collect()
- toList()
- forEach()
- reduce()
- count()
- min()
- max()
- findFirst()
- anyMatch()
- allMatch()
- noneMatch()

They produce a final result or side effect and trigger stream processing.

---

# 19. filter()

`filter()` keeps elements that satisfy a condition.

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5, 6);

List<Integer> even =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .toList();
```

Result:

```text
[2, 4, 6]
```

### Interview Answer

> `filter()` is an intermediate stream operation used to retain elements that satisfy a given predicate.

---

# 20. map()

`map()` transforms each element into another value.

```java
List<String> names =
        List.of("java", "spring", "sql");

List<String> upperCase =
        names.stream()
             .map(String::toUpperCase)
             .toList();
```

Result:

```text
[JAVA, SPRING, SQL]
```

### Interview Answer

> `map()` transforms each element of a stream into another value.

---

# 21. filter() + map()

This is a common interview pattern.

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5, 6);

List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .map(n -> n * n)
               .toList();
```

Result:

```text
[4, 16, 36]
```

Pipeline:

```text
1 2 3 4 5 6
      ↓
filter even
      ↓
2 4 6
      ↓
square
      ↓
4 16 36
```

---

# 22. distinct()

Removes duplicate elements.

```java
List<Integer> numbers =
        List.of(1, 2, 2, 3, 3, 4);

List<Integer> result =
        numbers.stream()
               .distinct()
               .toList();
```

Result:

```text
[1, 2, 3, 4]
```

`distinct()` relies on equality semantics of the elements.

---

# 23. sorted()

Sorts stream elements.

```java
List<Integer> numbers =
        List.of(5, 2, 8, 1, 3);

List<Integer> sorted =
        numbers.stream()
               .sorted()
               .toList();
```

Result:

```text
[1, 2, 3, 5, 8]
```

Custom sorting:

```java
List<String> names =
        List.of("John", "Alex", "Sudhir");

List<String> sorted =
        names.stream()
             .sorted(Comparator.comparing(String::length))
             .toList();
```

---

# 24. limit()

Limits the number of elements.

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5);

List<Integer> result =
        numbers.stream()
               .limit(3)
               .toList();
```

Result:

```text
[1, 2, 3]
```

---

# 25. skip()

Skips the first N elements.

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5);

List<Integer> result =
        numbers.stream()
               .skip(2)
               .toList();
```

Result:

```text
[3, 4, 5]
```

---

# 26. count()

Counts elements.

```java
long count =
        List.of(1, 2, 3, 4, 5)
            .stream()
            .count();
```

Result:

```text
5
```

---

# 27. reduce()

`reduce()` combines stream elements into a single result.

Example:

```java
int sum =
        List.of(1, 2, 3, 4, 5)
            .stream()
            .reduce(0, Integer::sum);
```

Result:

```text
15
```

Conceptually:

```text
0 + 1
  ↓
1 + 2
  ↓
3 + 3
  ↓
6 + 4
  ↓
10 + 5
  ↓
15
```

### Interview Answer

> `reduce()` combines stream elements into a single result using an accumulation operation.

---

# 28. findFirst()

Returns the first element as an Optional.

```java
Optional<Integer> first =
        List.of(10, 20, 30)
            .stream()
            .findFirst();
```

Use:

```java
first.ifPresent(System.out::println);
```

---

# 29. anyMatch()

Checks whether at least one element matches a condition.

```java
boolean result =
        List.of(1, 2, 3, 4)
            .stream()
            .anyMatch(n -> n > 3);
```

Result:

```text
true
```

---

# 30. allMatch()

Checks whether every element matches.

```java
boolean result =
        List.of(2, 4, 6)
            .stream()
            .allMatch(n -> n % 2 == 0);
```

Result:

```text
true
```

---

# 31. noneMatch()

Checks whether no element matches.

```java
boolean result =
        List.of(1, 3, 5)
            .stream()
            .noneMatch(n -> n % 2 == 0);
```

Result:

```text
true
```

---

# 32. flatMap()

`flatMap()` is used when each element produces multiple elements and we want one flattened stream.

Example:

```java
List<List<Integer>> numbers =
        List.of(
            List.of(1, 2),
            List.of(3, 4),
            List.of(5, 6)
        );

List<Integer> result =
        numbers.stream()
               .flatMap(List::stream)
               .toList();
```

Result:

```text
[1, 2, 3, 4, 5, 6]
```

### map() vs flatMap()

`map()`:

```text
one element → one result
```

`flatMap()`:

```text
one element → multiple results
then flatten
```

### Interview Answer

> `flatMap()` transforms each element into a stream and then flattens all resulting streams into a single stream.

---

# 33. Collectors

`Collectors` provides utilities for collecting stream results.

Example:

```java
List<String> names =
        List.of("Java", "Spring", "SQL");

List<String> result =
        names.stream()
             .filter(name -> name.length() > 3)
             .collect(Collectors.toList());
```

In modern Java, this can also be written:

```java
List<String> result =
        names.stream()
             .filter(name -> name.length() > 3)
             .toList();
```

---

# 34. Collecting to a Set

```java
Set<String> result =
        names.stream()
             .collect(Collectors.toSet());
```

---

# 35. Joining Strings

```java
String result =
        List.of("Java", "Spring", "SQL")
            .stream()
            .collect(Collectors.joining(", "));
```

Result:

```text
Java, Spring, SQL
```

---

# 36. Grouping By

One of the most useful `Collectors` operations for backend interviews is `groupingBy()`.

Suppose:

```java
class Employee {

    String name;
    String department;

    Employee(String name, String department) {
        this.name = name;
        this.department = department;
    }

    public String getDepartment() {
        return department;
    }
}
```

Group employees by department:

```java
Map<String, List<Employee>> employeesByDepartment =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment
                     )
                 );
```

Result conceptually:

```text
IT → [Employee1, Employee2]
HR → [Employee3]
Sales → [Employee4, Employee5]
```

---

# 37. Partitioning By

`partitioningBy()` divides elements into two groups based on a boolean condition.

```java
Map<Boolean, List<Integer>> result =
        List.of(1, 2, 3, 4, 5, 6)
            .stream()
            .collect(
                Collectors.partitioningBy(
                    n -> n % 2 == 0
                )
            );
```

Conceptually:

```text
true  → [2, 4, 6]
false → [1, 3, 5]
```

---

# 38. toMap()

Convert stream elements into a Map.

```java
Map<Integer, String> users =
        employees.stream()
                 .collect(
                     Collectors.toMap(
                         Employee::getId,
                         Employee::getName
                     )
                 );
```

Be careful with duplicate keys. If duplicate keys are possible, provide a merge function.

Example:

```java
Collectors.toMap(
    Employee::getDepartment,
    Employee::getName,
    (existing, replacement) -> existing
)
```

---

# 39. Optional

`Optional<T>` represents a value that may or may not be present.

Instead of returning `null`:

```java
public User findUser(Long id) {
    // may return null
}
```

we can use:

```java
public Optional<User> findUser(Long id) {
    // may or may not contain User
}
```

Usage:

```java
Optional<User> user =
        userRepository.findById(id);
```

---

# 40. Creating Optional

```java
Optional<String> value =
        Optional.of("Java");
```

For a possibly null value:

```java
Optional<String> value =
        Optional.ofNullable(name);
```

For an empty Optional:

```java
Optional<String> value =
        Optional.empty();
```

### Important

Do not use:

```java
Optional.of(null);
```

because it throws `NullPointerException`.

Use:

```java
Optional.ofNullable(null);
```

---

# 41. isPresent()

```java
Optional<String> value =
        Optional.of("Java");

if (value.isPresent()) {
    System.out.println(value.get());
}
```

However, blindly using `isPresent()` followed by `get()` can make code similar to explicit null checking.

Often prefer methods such as:

```java
ifPresent()
orElse()
orElseGet()
orElseThrow()
map()
```

when they make the intent clearer.

---

# 42. orElse()

Provides a fallback value.

```java
String name =
        Optional.ofNullable(userName)
                .orElse("Unknown");
```

---

# 43. orElseGet()

Uses a Supplier to create the fallback lazily.

```java
String name =
        Optional.ofNullable(userName)
                .orElseGet(() -> getDefaultName());
```

### Important Interview Difference

`orElse()` evaluates its argument even when the Optional contains a value.

`orElseGet()` invokes the supplier only when the Optional is empty.

Example:

```java
String value =
        optional.orElse(expensiveOperation());
```

The method may execute even when the value exists.

With:

```java
String value =
        optional.orElseGet(
            () -> expensiveOperation()
        );
```

the fallback operation is deferred until needed.

---

# 44. orElseThrow()

```java
User user =
        userRepository.findById(id)
                      .orElseThrow(
                          () -> new UserNotFoundException(
                              "User not found"
                          )
                      );
```

This is common in Spring Boot services.

---

# 45. Optional map()

Optional can be transformed using `map()`.

```java
Optional<String> name =
        Optional.of("sudhir");

Optional<String> upper =
        name.map(String::toUpperCase);
```

Result:

```text
Optional[SUDHIR]
```

---

# 46. Optional flatMap()

`flatMap()` is useful when the mapping function itself returns an Optional.

```java
Optional<String> result =
        userService.findUser(id)
                   .flatMap(User::getEmail);
```

This avoids creating:

```text
Optional<Optional<String>>
```

---

# 47. Optional Best Practices

Use Optional mainly when a value may legitimately be absent and the API benefits from making that possibility explicit.

Avoid:

```java
Optional<String> name = null;
```

An Optional variable itself should generally not be null.

Avoid using Optional as a field or parameter simply because "Optional is better than null." Its most common use is as a return type where absence is part of the API contract.

Avoid:

```java
optional.get();
```

without considering the empty case.

Prefer:

```java
orElse()
orElseGet()
orElseThrow()
ifPresent()
map()
flatMap()
```

when appropriate.

---

# 48. Default Methods in Interfaces

Java 8 allows interfaces to contain default methods.

```java
interface Vehicle {

    void start();

    default void stop() {
        System.out.println("Vehicle stopped");
    }
}
```

A class implementing the interface automatically gets the default implementation unless it overrides it.

### Why were default methods introduced?

They allow interfaces to evolve by adding behavior without forcing every existing implementation to immediately implement a new method.

---

# 49. Static Methods in Interfaces

Interfaces can also contain static methods.

```java
interface MathUtil {

    static int square(int value) {
        return value * value;
    }
}
```

Call it using the interface name:

```java
int result =
        MathUtil.square(5);
```

Static interface methods are not inherited by implementing classes in the same way instance methods are.

---

# 50. Stream Laziness

Intermediate stream operations are lazy.

Example:

```java
Stream<Integer> stream =
        numbers.stream()
               .filter(n -> {
                   System.out.println(n);
                   return n % 2 == 0;
               });
```

Nothing happens yet.

The stream starts processing when a terminal operation is called:

```java
stream.toList();
```

### Interview Answer

> Stream intermediate operations are lazy and are generally executed only when a terminal operation triggers the pipeline.

---

# 51. Streams Do Not Store Data

A Stream is not a data structure that stores elements.

A collection stores data:

```text
List → stores elements
```

A stream represents a pipeline for processing data:

```text
Collection
    ↓
Stream
    ↓
filter
    ↓
map
    ↓
collect
```

---

# 52. Streams vs Collections

| Collections | Streams |
|---|---|
| Store data | Process data |
| Can be iterated multiple times | Generally consumed once |
| External iteration is common | Internal/declarative iteration |
| Data structure | Processing abstraction |
| Can add/remove elements depending on type | Does not itself store elements |

---

# 53. Can a Stream Be Reused?

No.

Once a terminal operation has been executed, the stream is consumed.

This is invalid:

```java
Stream<String> stream =
        names.stream();

stream.count();

stream.toList();
```

The second operation throws:

```text
IllegalStateException
```

Create a new stream instead:

```java
names.stream().count();

names.stream().toList();
```

---

# 54. forEach()

```java
List<String> names =
        List.of("Java", "Spring", "SQL");

names.forEach(
        name -> System.out.println(name)
);
```

Or:

```java
names.forEach(System.out::println);
```

`forEach()` is a terminal operation on streams.

Use it mainly for side effects rather than building transformed results.

---

# 55. Avoid Side Effects in Stream Pipelines

Avoid code such as:

```java
List<String> result = new ArrayList<>();

names.stream()
     .filter(name -> name.length() > 3)
     .forEach(result::add);
```

Prefer:

```java
List<String> result =
        names.stream()
             .filter(name -> name.length() > 3)
             .toList();
```

The second version is clearer and better aligned with the stream model.

---

# 56. Parallel Streams

Java can process streams in parallel:

```java
numbers.parallelStream()
       .map(this::process)
       .toList();
```

Parallel streams use the common ForkJoinPool by default.

### Important Interview Point

Parallel streams are not automatically faster.

They can add:

- Thread coordination overhead
- Splitting/merging overhead
- Contention
- Ordering complexity
- Problems with shared mutable state

Use them only when the workload is appropriate and performance has been measured.

---

# 57. Sequential vs Parallel Stream

```java
numbers.stream()
```

Sequential processing.

```java
numbers.parallelStream()
```

Parallel processing.

### Interview Answer

> A parallel stream can improve throughput for suitable CPU-bound workloads with sufficient data and independent operations, but it should not be used blindly because parallelism introduces overhead and concurrency concerns.

---

# 58. Common Java 8 Interview Questions

1. What are the major features introduced in Java 8?
2. What is a lambda expression?
3. What is a functional interface?
4. What is `@FunctionalInterface`?
5. Predicate vs Function?
6. Consumer vs Supplier?
7. What is a method reference?
8. What is the Stream API?
9. Intermediate vs terminal operations?
10. What is lazy evaluation in streams?
11. What is `map()`?
12. What is `filter()`?
13. What is `flatMap()`?
14. `map()` vs `flatMap()`?
15. What is `reduce()`?
16. What is `collect()`?
17. What is `groupingBy()`?
18. What is `partitioningBy()`?
19. What is Optional?
20. `orElse()` vs `orElseGet()`?
21. What is `orElseThrow()`?
22. What is the difference between `of()` and `ofNullable()`?
23. Can streams be reused?
24. Can streams store data?
25. Sequential stream vs parallel stream?
26. What are default methods?
27. Why were default methods introduced?
28. Can interfaces have static methods?
29. What is a method reference?
30. Why should side effects generally be avoided in streams?

---

# 59. Common Coding Interview Patterns

## Find Even Numbers

```java
List<Integer> even =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .toList();
```

---

## Find Squares

```java
List<Integer> squares =
        numbers.stream()
               .map(n -> n * n)
               .toList();
```

---

## Find Maximum

```java
Optional<Integer> max =
        numbers.stream()
               .max(Integer::compareTo);
```

---

## Find Minimum

```java
Optional<Integer> min =
        numbers.stream()
               .min(Integer::compareTo);
```

---

## Sum Numbers

```java
int sum =
        numbers.stream()
               .mapToInt(Integer::intValue)
               .sum();
```

---

## Count Elements

```java
long count =
        numbers.stream()
               .filter(n -> n > 10)
               .count();
```

---

## Remove Duplicates

```java
List<Integer> unique =
        numbers.stream()
               .distinct()
               .toList();
```

---

## Sort Descending

```java
List<Integer> descending =
        numbers.stream()
               .sorted(Comparator.reverseOrder())
               .toList();
```

---

## Find First Matching Element

```java
Optional<Integer> result =
        numbers.stream()
               .filter(n -> n > 50)
               .findFirst();
```

---

# 60. Backend Example

Suppose an e-commerce application has:

```java
List<Order> orders;
```

Find completed orders:

```java
List<Order> completed =
        orders.stream()
              .filter(order ->
                  order.getStatus() == OrderStatus.COMPLETED
              )
              .toList();
```

Calculate total order amount:

```java
double total =
        orders.stream()
              .mapToDouble(Order::getAmount)
              .sum();
```

Find the most expensive order:

```java
Optional<Order> expensive =
        orders.stream()
              .max(
                  Comparator.comparingDouble(
                      Order::getAmount
                  )
              );
```

Group orders by status:

```java
Map<OrderStatus, List<Order>> ordersByStatus =
        orders.stream()
              .collect(
                  Collectors.groupingBy(
                      Order::getStatus
                  )
              );
```

These patterns are commonly useful in backend services.

---

# 61. Quick Revision

```text
Lambda
    ↓
Concise function-like behavior

Functional Interface
    ↓
Exactly one abstract method

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
Data processing pipeline

filter()
    ↓
Select elements

map()
    ↓
Transform elements

flatMap()
    ↓
Transform + flatten

reduce()
    ↓
Combine into one result

collect()
    ↓
Build a result

Optional
    ↓
Represent possible absence

orElse()
    ↓
Fallback value

orElseGet()
    ↓
Lazy fallback

orElseThrow()
    ↓
Throw when absent
```

---

# 62. Interview Mindset

Don't just memorize:

> "Streams are used to process collections."

Be ready to explain:

```text
What is a stream?
        ↓
How is it different from a collection?
        ↓
What are intermediate operations?
        ↓
What are terminal operations?
        ↓
Why are intermediate operations lazy?
        ↓
What is map vs flatMap?
        ↓
What is reduce?
        ↓
What is Optional?
        ↓
What is orElse vs orElseGet?
        ↓
When should parallel streams be avoided?
```

A strong Java backend developer should be able to use Java 8 features naturally in service-layer code while still writing code that is readable, testable and maintainable.
