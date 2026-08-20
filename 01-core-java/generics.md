# Generics in Java

Generics allow classes, interfaces and methods to work with different types while providing **compile-time type safety**.

They help us avoid unnecessary type casting and make code more reusable.

---

# 1. Why Do We Need Generics?

Without generics:

```java
List names = new ArrayList();

names.add("Sudhir");
names.add(100);

String name = (String) names.get(0);
```

Problems:

- No compile-time type safety
- Manual casting is required
- Runtime `ClassCastException` can occur

With generics:

```java
List<String> names = new ArrayList<>();

names.add("Sudhir");
names.add("Alex");
```

Now Java knows that the list contains only `String` values.

This will fail at compile time:

```java
names.add(100);
```

### Interview Answer

> Generics provide compile-time type safety and allow us to write reusable code that works with different types without requiring explicit casting.

---

# 2. Generic Class

A class can define a type parameter.

```java
class Box<T> {

    private T value;

    public void setValue(T value) {
        this.value = value;
    }

    public T getValue() {
        return value;
    }
}
```

Usage:

```java
Box<String> stringBox = new Box<>();

stringBox.setValue("Java");

String value = stringBox.getValue();
```

Another type:

```java
Box<Integer> numberBox = new Box<>();

numberBox.setValue(100);

Integer number = numberBox.getValue();
```

The same class works with different types.

---

# 3. Generic Method

Methods can also define their own type parameters.

```java
public static <T> void printValue(T value) {

    System.out.println(value);
}
```

Usage:

```java
printValue("Java");
printValue(100);
printValue(10.5);
```

### Syntax

```text
<T>
```

before the return type indicates that the method declares a generic type parameter.

---

# 4. Multiple Type Parameters

A class can have multiple type parameters.

```java
class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() {
        return key;
    }

    public V getValue() {
        return value;
    }
}
```

Usage:

```java
Pair<Integer, String> user =
        new Pair<>(101, "Sudhir");
```

Here:

```text
K → Integer
V → String
```

Common naming conventions:

| Type Parameter | Meaning |
|---|---|
| T | Type |
| E | Element |
| K | Key |
| V | Value |
| N | Number |
| S | Second type |

---

# 5. Generic Interface

Interfaces can also use generics.

```java
interface Repository<T> {

    void save(T entity);

    T findById(Long id);
}
```

Implementation:

```java
class UserRepository implements Repository<User> {

    @Override
    public void save(User entity) {
        System.out.println("Saving user");
    }

    @Override
    public User findById(Long id) {
        return new User();
    }
}
```

This pattern is very common in backend development.

---

# 6. Generic Bounds

Sometimes we want a generic type to be restricted to a specific class or interface.

Example:

```java
public static <T extends Number> double square(T number) {

    return number.doubleValue() * number.doubleValue();
}
```

Allowed:

```java
square(10);
square(10.5);
```

Because `Integer` and `Double` extend `Number`.

A `String` would not be allowed.

---

# 7. Upper Bounded Type

The syntax:

```java
<T extends Number>
```

means:

> T must be `Number` or a subclass of `Number`.

Example:

```java
public static <T extends Number>
void printNumber(T number) {

    System.out.println(number);
}
```

---

# 8. Multiple Bounds

A type parameter can have multiple bounds.

```java
<T extends Number & Comparable<T>>
```

The class bound, if present, must come first.

Example:

```java
public static <T extends Number & Comparable<T>>
void process(T value) {

    System.out.println(value);
}
```

The type must satisfy both bounds.

---

# 9. Wildcards

A wildcard is represented using:

```java
?
```

It means an unknown type.

Example:

```java
List<?> list;
```

This means:

> A list of some unknown type.

It could be:

```java
List<String>
```

or:

```java
List<Integer>
```

or:

```java
List<Double>
```

---

# 10. Unbounded Wildcard

Example:

```java
public static void printList(List<?> list) {

    for (Object value : list) {
        System.out.println(value);
    }
}
```

This method can accept:

```java
List<String>
List<Integer>
List<Double>
```

### Interview Answer

> An unbounded wildcard represents a collection of an unknown type and is useful when the method does not need to know the specific element type.

---

# 11. Upper Bounded Wildcard

Syntax:

```java
? extends Number
```

Example:

```java
public static double sum(List<? extends Number> numbers) {

    double total = 0;

    for (Number number : numbers) {
        total += number.doubleValue();
    }

    return total;
}
```

This accepts:

```java
List<Integer>
List<Double>
List<Float>
```

because all of them extend `Number`.

---

# 12. Lower Bounded Wildcard

Syntax:

```java
? super Integer
```

Example:

```java
public static void addNumbers(
        List<? super Integer> numbers) {

    numbers.add(10);
    numbers.add(20);
}
```

This can accept:

```java
List<Integer>
List<Number>
List<Object>
```

---

# 13. PECS

One of the most important generic interview rules is:

> **PECS = Producer Extends, Consumer Super**

### Producer

If a collection produces values for you:

```java
? extends T
```

Example:

```java
List<? extends Number>
```

You can safely read values as `Number`.

### Consumer

If a collection consumes values from you:

```java
? super T
```

Example:

```java
List<? super Integer>
```

You can safely add `Integer` values.

### Easy Rule

```text
Producer → extends

Consumer → super
```

---

# 14. extends vs super

| `? extends T` | `? super T` |
|---|---|
| Producer | Consumer |
| Mainly used for reading | Mainly used for writing |
| Accepts T and subclasses | Accepts T and superclasses |
| Safe to read as T | Safe to add T |

Example:

```java
List<? extends Number> numbers;
```

You can read:

```java
Number number = numbers.get(0);
```

But you generally cannot add a specific `Number` safely.

With:

```java
List<? super Integer> numbers;
```

you can:

```java
numbers.add(10);
```

---

# 15. Generic Type Erasure

Java implements generics using **type erasure**.

Generic type information is primarily used by the compiler for type checking and is erased from most runtime generic operations.

Example:

```java
List<String> names = new ArrayList<>();
```

At runtime, the JVM generally sees the underlying `List` implementation rather than retaining `String` as the collection's generic type parameter.

### Why does Java use type erasure?

It allowed Java to introduce generics while maintaining compatibility with older code written before generics existed.

### Interview Answer

> Type erasure means generic type parameters are primarily enforced at compile time and are erased from most runtime type information.

---

# 16. Type Erasure Example

Consider:

```java
List<String> names = new ArrayList<>();
List<Integer> numbers = new ArrayList<>();
```

At runtime, both use the same raw collection implementation.

This is why you cannot normally do:

```java
if (value instanceof List<String>) {
}
```

Instead, you can check:

```java
if (value instanceof List<?>) {
}
```

---

# 17. Why Can't We Use Primitive Types?

Generics work with reference types, not primitive types.

This is invalid:

```java
List<int> numbers;
```

Use the wrapper type:

```java
List<Integer> numbers;
```

Java handles conversion between primitives and wrappers through **autoboxing and unboxing**.

---

# 18. Autoboxing and Unboxing

### Autoboxing

Primitive → Wrapper

```java
Integer number = 10;
```

Java automatically converts:

```text
int → Integer
```

### Unboxing

Wrapper → Primitive

```java
Integer number = 10;

int value = number;
```

Java converts:

```text
Integer → int
```

---

# 19. Generic Arrays

Creating generic arrays directly is not normally allowed.

This is invalid:

```java
T[] values = new T[10];
```

Because generic type information is erased at runtime.

A common workaround involves creating an array of a known runtime type and using a cast carefully, but generic array creation should be approached cautiously.

---

# 20. Raw Types

A raw type is a generic type used without specifying its type parameter.

Example:

```java
List names = new ArrayList();
```

This is allowed for backward compatibility but should generally be avoided in new code.

Prefer:

```java
List<String> names = new ArrayList<>();
```

### Why avoid raw types?

- Loses compile-time type safety
- Requires casting
- Can produce runtime errors
- Generates compiler warnings

---

# 21. Diamond Operator

Java allows type inference using:

```java
<>
```

Instead of:

```java
List<String> names =
        new ArrayList<String>();
```

Use:

```java
List<String> names =
        new ArrayList<>();
```

The compiler infers the generic type from the variable declaration.

---

# 22. Generics and Inheritance

A common misconception is:

```text
Integer extends Number
```

therefore:

```text
List<Integer> is a List<Number>
```

This is **not true**.

Java generics are invariant.

So this is invalid:

```java
List<Integer> integers =
        new ArrayList<>();

List<Number> numbers = integers;
```

Instead, use a wildcard:

```java
List<? extends Number> numbers = integers;
```

---

# 23. Invariance

If:

```text
Dog extends Animal
```

that does not mean:

```text
List<Dog> extends List<Animal>
```

This prevents unsafe operations.

Imagine Java allowed:

```java
List<Dog> dogs = new ArrayList<>();

List<Animal> animals = dogs;

animals.add(new Cat());
```

Now the `List<Dog>` would contain a `Cat`.

Generics prevent this type-safety problem.

---

# 24. Generic Repository Example

Generics are commonly used in backend architectures.

```java
public interface Repository<T> {

    T findById(Long id);

    List<T> findAll();

    void save(T entity);

    void delete(T entity);
}
```

A specific repository can use it:

```java
public interface UserRepository
        extends Repository<User> {
}
```

Another:

```java
public interface ProductRepository
        extends Repository<Product> {
}
```

The common generic abstraction can be reused across entity types.

---

# 25. Generic Utility Method

A useful example:

```java
public static <T> T getFirst(List<T> values) {

    if (values == null || values.isEmpty()) {
        return null;
    }

    return values.get(0);
}
```

Usage:

```java
List<String> names =
        List.of("Sudhir", "Alex");

String firstName = getFirst(names);
```

The compiler infers:

```text
T = String
```

---

# 26. Generic Method with Two Types

```java
public static <K, V> void printPair(
        K key,
        V value) {

    System.out.println(key + " = " + value);
}
```

Usage:

```java
printPair(101, "Sudhir");
printPair("language", "Java");
```

---

# 27. Generic Class with a Bound

```java
class NumericBox<T extends Number> {

    private T value;

    public NumericBox(T value) {
        this.value = value;
    }

    public double getDoubleValue() {
        return value.doubleValue();
    }
}
```

Usage:

```java
NumericBox<Integer> integerBox =
        new NumericBox<>(100);

NumericBox<Double> doubleBox =
        new NumericBox<>(10.5);
```

---

# 28. Generic Constructor

A constructor can also use a generic type parameter.

```java
class Example {

    <T> Example(T value) {
        System.out.println(value);
    }
}
```

Usage:

```java
Example first = new Example("Java");
Example second = new Example(100);
```

---

# 29. Static Members and Class Type Parameters

A static field cannot use the class's type parameter.

This is invalid:

```java
class Box<T> {

    private static T value;
}
```

Why?

Because static members belong to the class itself, while `T` belongs to a particular instance type.

You can, however, define a generic static method:

```java
class Utility {

    public static <T> T identity(T value) {
        return value;
    }
}
```

---

# 30. Common Interview Questions

1. What are generics in Java?
2. Why do we need generics?
3. What is type safety?
4. What is type erasure?
5. What is a raw type?
6. What is a wildcard?
7. What is an unbounded wildcard?
8. What is `? extends`?
9. What is `? super`?
10. What is PECS?
11. Why can't we use primitive types with generics?
12. What is autoboxing?
13. What is unboxing?
14. Can we create generic arrays?
15. Why is `List<Integer>` not a subtype of `List<Number>`?
16. What is invariance?
17. What is the diamond operator?
18. Can a generic class have static members?
19. Can a generic method exist in a non-generic class?
20. Can an interface be generic?
21. Can a generic type have multiple bounds?
22. What is a bounded type parameter?
23. What is the difference between `T` and `?`?
24. `List<T>` vs `List<?>`?
25. How are generics used in backend applications?

---

# 31. T vs ?

This is a common interview question.

### T

`T` represents a type parameter that can be referred to within the class or method.

Example:

```java
public static <T> T identity(T value) {
    return value;
}
```

The method knows that the input and return value have the same type.

### ?

`?` represents an unknown type.

Example:

```java
public static void print(List<?> values) {

    for (Object value : values) {
        System.out.println(value);
    }
}
```

The method does not need to know the exact type.

### Quick Answer

> `T` is a named type parameter that can be used consistently within a generic declaration, while `?` represents an unknown type.

---

# 32. T vs ? extends T

Consider:

```java
List<T>
```

versus:

```java
List<? extends T>
```

`List<T>` means the exact type parameter is `T`.

`List<? extends T>` means the list contains some unknown type that is `T` or a subclass of `T`.

Example:

```java
List<Integer> integers = new ArrayList<>();

List<? extends Number> numbers = integers;
```

This is valid because `Integer` extends `Number`.

---

# 33. Why Use Generics in Backend Development?

Generics are heavily used in real Java backend code.

Examples include:

```java
List<User>
```

```java
Optional<User>
```

```java
ResponseEntity<User>
```

```java
Page<Product>
```

```java
Map<Long, Order>
```

```java
Repository<User>
```

They provide:

- Type safety
- Reusability
- Cleaner APIs
- Less casting
- Better maintainability

---

# 34. Spring Boot Example

Spring applications use generics extensively.

Example:

```java
public ResponseEntity<User> getUser(Long id) {

    User user = userService.findById(id);

    return ResponseEntity.ok(user);
}
```

Here:

```text
ResponseEntity<User>
```

clearly communicates that the response contains a `User`.

Another example:

```java
public ResponseEntity<List<Product>> getProducts() {

    List<Product> products =
            productService.findAll();

    return ResponseEntity.ok(products);
}
```

The generic type makes the API contract clearer.

---

# 35. Generic Best Practices

### Prefer parameterized types

Good:

```java
List<String> names = new ArrayList<>();
```

Avoid:

```java
List names = new ArrayList();
```

---

### Use meaningful type parameters

Good:

```java
class Repository<T> {
}
```

For multiple types:

```java
class Pair<K, V> {
}
```

---

### Use bounded wildcards when appropriate

Producer:

```java
List<? extends Number>
```

Consumer:

```java
List<? super Integer>
```

---

### Avoid unnecessary wildcards

Don't use:

```java
List<?> 
```

when the method actually needs to preserve a relationship between multiple types.

For example:

```java
public static <T> T first(List<T> values) {
    return values.get(0);
}
```

Here `T` is more useful than `?`.

---

# 36. Quick Revision

```text
Generics
    ↓
Compile-time type safety

<T>
    ↓
Named type parameter

?
    ↓
Unknown type

? extends T
    ↓
Producer

? super T
    ↓
Consumer

PECS
    ↓
Producer Extends
Consumer Super

Type Erasure
    ↓
Generic type information is primarily
enforced at compile time

Raw Type
    ↓
Generic type without type parameters

Diamond Operator
    ↓
<>
Compiler infers the generic type
```

---

# 37. Interview Mindset

Don't memorize:

> "Generics provide type safety."

Be ready to explain:

```text
Why do we need generics?
        ↓
What problem existed before generics?
        ↓
What is type erasure?
        ↓
What is a wildcard?
        ↓
When do we use extends?
        ↓
When do we use super?
        ↓
What is PECS?
        ↓
Why is List<Integer> not List<Number>?
        ↓
Where do generics appear in Spring Boot?
```

A strong Java developer should be comfortable reading and designing APIs such as:

```java
List<User>
Map<Long, Order>
Optional<Product>
ResponseEntity<User>
Page<Product>
Repository<User>
```

Generics are not just an interview topic — they are a fundamental part of writing type-safe and reusable Java backend code.
