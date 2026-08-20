# Strings in Java

Strings are one of the most frequently used types in Java and are extremely common in Java backend interviews.

Important topics include:

- String immutability
- String pool
- `==` vs `equals()`
- `String`, `StringBuilder`, `StringBuffer`
- String concatenation
- `intern()`
- Common String methods
- Performance
- Thread safety
- String interview problems

---

# 1. What is String?

`String` is a class in Java:

```java
String name = "Sudhir";
```

It represents a sequence of characters.

Strings are objects, even though Java provides convenient literal syntax.

```java
String name = "Java";
```

is conceptually creating/using a `String` object.

---

# 2. String Is Immutable

The most important String concept is:

> **String objects are immutable.**

Once a String object is created, its contents cannot be changed.

Example:

```java
String name = "Java";

name.concat(" Backend");

System.out.println(name);
```

Output:

```text
Java
```

The `concat()` operation creates a new String.

To use the new value:

```java
name = name.concat(" Backend");
```

Now:

```text
Java Backend
```

---

# 3. Why Is String Immutable?

String immutability provides several benefits:

- Security
- Thread safety
- String pooling
- Stable hash codes
- Easier caching
- Predictable behavior

Strings are frequently used for:

- Usernames
- URLs
- File paths
- Database connection strings
- Configuration
- Authentication-related values

Immutability prevents accidental modification of a shared String object.

---

# 4. String Pool

Java maintains a special pool of String literals.

Example:

```java
String a = "Java";
String b = "Java";
```

Both references can point to the same pooled String object.

Conceptually:

```text
        String Pool

       +---------+
a ---> |  "Java"  | <--- b
       +---------+
```

This saves memory when the same String literal is used repeatedly.

---

# 5. `==` vs `equals()`

This is one of the most common Java interview questions.

### `==`

For object references, `==` compares whether two references point to the same object.

### `equals()`

`equals()` is used to compare logical/content equality according to the class's implementation.

Example:

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);
System.out.println(a.equals(b));
```

Output:

```text
false
true
```

Because:

```text
a == b
    ↓
Different objects

a.equals(b)
    ↓
Same String content
```

### Interview Answer

> For objects, `==` compares references, while `equals()` compares logical equality as defined by the class.

---

# 6. String Literals and `==`

Consider:

```java
String a = "Java";
String b = "Java";

System.out.println(a == b);
```

Output:

```text
true
```

Why?

Because both literals can refer to the same pooled String object.

But do not use `==` to compare String content.

Always prefer:

```java
a.equals(b)
```

or, when null safety is important:

```java
Objects.equals(a, b)
```

---

# 7. `new String()`

Consider:

```java
String a = "Java";
String b = new String("Java");
```

The literal uses the String pool.

`new String("Java")` explicitly creates a new String object.

Therefore:

```java
a == b
```

is:

```text
false
```

while:

```java
a.equals(b)
```

is:

```text
true
```

---

# 8. `intern()`

`intern()` returns the canonical representation of a String.

Example:

```java
String a = new String("Java");

String b = a.intern();

String c = "Java";
```

Now:

```java
b == c
```

is:

```text
true
```

because `intern()` returns the pooled representation.

### Interview Answer

> `intern()` returns a canonical pooled representation of the String, allowing equal strings to share the same pooled reference.

Use it carefully; manually interning large numbers of dynamically generated strings can increase memory pressure.

---

# 9. String Concatenation

Example:

```java
String result = "Hello " + "Java";
```

The compiler can optimize constant expressions.

For runtime concatenation:

```java
String result = name + " Backend";
```

Java uses string-concatenation machinery rather than repeatedly mutating the original String.

For modern Java, the compiler/JVM may optimize concatenation using `invokedynamic` and related mechanisms.

---

# 10. String vs StringBuilder

`String` is immutable.

`StringBuilder` is mutable.

Example:

```java
StringBuilder builder =
        new StringBuilder("Java");

builder.append(" Backend");

System.out.println(builder);
```

Output:

```text
Java Backend
```

The builder modifies its internal character storage rather than creating a new String object for every append.

---

# 11. StringBuilder

Use `StringBuilder` when building strings through multiple modifications.

Example:

```java
StringBuilder builder =
        new StringBuilder();

for (int i = 1; i <= 5; i++) {
    builder.append(i);
}

String result = builder.toString();
```

Result:

```text
12345
```

### Interview Answer

> StringBuilder is mutable and is generally preferred for repeated string modifications in single-threaded code.

---

# 12. StringBuffer

`StringBuffer` is also mutable but its methods are synchronized.

```java
StringBuffer buffer =
        new StringBuffer();

buffer.append("Java");
buffer.append(" Backend");
```

Because of synchronization, it can have more overhead than `StringBuilder`.

### Interview Answer

> StringBuffer is a synchronized mutable character sequence, while StringBuilder is not synchronized and is generally preferred when thread safety is not required.

---

# 13. String vs StringBuilder vs StringBuffer

| String | StringBuilder | StringBuffer |
|---|---|---|
| Immutable | Mutable | Mutable |
| Thread-safe due to immutability | Not synchronized | Synchronized methods |
| Good for fixed text | Good for repeated modifications | Useful when synchronized mutable sequence is specifically needed |
| Concatenation creates new String results | Modifies builder | Modifies buffer |
| Usually preferred for simple values | Preferred for local string building | Less commonly needed |

---

# 14. `charAt()`

Returns the character at an index.

```java
String value = "Java";

char ch = value.charAt(0);

System.out.println(ch);
```

Output:

```text
J
```

Indexes are zero-based.

---

# 15. `length()`

Returns the number of characters represented by the String.

```java
String value = "Java";

System.out.println(value.length());
```

Output:

```text
4
```

---

# 16. `substring()`

Extracts part of a String.

```java
String value = "Java Backend";

String result = value.substring(5);

System.out.println(result);
```

Output:

```text
Backend
```

Another form:

```java
value.substring(0, 4);
```

The ending index is exclusive.

---

# 17. `contains()`

```java
String value = "Spring Boot";

boolean result =
        value.contains("Boot");
```

Result:

```text
true
```

---

# 18. `startsWith()` and `endsWith()`

```java
String value = "SpringBoot";

value.startsWith("Spring");
value.endsWith("Boot");
```

Both return:

```text
true
```

---

# 19. `indexOf()`

Finds the index of a character or substring.

```java
String value = "Java Backend";

int index = value.indexOf("Backend");
```

Result:

```text
5
```

If the value is not found:

```text
-1
```

---

# 20. `lastIndexOf()`

Returns the last occurrence.

```java
String value = "banana";

int index = value.lastIndexOf("a");
```

Result:

```text
5
```

---

# 21. `toUpperCase()` and `toLowerCase()`

```java
String value = "Java";

System.out.println(value.toUpperCase());
System.out.println(value.toLowerCase());
```

Output:

```text
JAVA
java
```

Remember that String is immutable, so these methods return new String values.

---

# 22. `trim()` vs `strip()`

`trim()` removes certain leading and trailing characters based on older ASCII-oriented behavior.

```java
String value = "  Java  ";

String result = value.trim();
```

Java 11 introduced:

```java
strip()
stripLeading()
stripTrailing()
```

These are Unicode-aware and generally preferable when working with modern text.

---

# 23. `isEmpty()` vs `isBlank()`

### isEmpty()

Checks whether length is zero.

```java
"".isEmpty();
```

returns:

```text
true
```

But:

```java
" ".isEmpty();
```

returns:

```text
false
```

### isBlank()

Java 11 introduced `isBlank()`.

```java
"   ".isBlank();
```

returns:

```text
true
```

It considers whitespace.

---

# 24. `replace()`

Replaces characters or literal sequences.

```java
String value = "Java Java";

String result =
        value.replace("Java", "Spring");
```

Result:

```text
Spring Spring
```

---

# 25. `replaceAll()`

`replaceAll()` uses a regular expression.

```java
String value = "Java123";

String result =
        value.replaceAll("\\d", "");
```

Result:

```text
Java
```

For simple literal replacement, `replace()` is usually preferable because it does not interpret the target as a regex.

---

# 26. `split()`

Splits a String using a regular expression.

```java
String value =
        "Java,Spring,SQL";

String[] parts =
        value.split(",");
```

Result:

```text
Java
Spring
SQL
```

Remember that `split()` accepts a regex, so special regex characters need escaping.

---

# 27. `join()`

```java
String result =
        String.join(
            ", ",
            "Java",
            "Spring",
            "SQL"
        );
```

Result:

```text
Java, Spring, SQL
```

---

# 28. `equalsIgnoreCase()`

Compares content without considering case differences.

```java
"java".equalsIgnoreCase("JAVA");
```

Result:

```text
true
```

---

# 29. `compareTo()`

Used for lexicographical comparison.

```java
String a = "Apple";
String b = "Banana";

int result = a.compareTo(b);
```

Conceptually:

```text
negative → a comes before b
zero     → equal ordering
positive → a comes after b
```

The exact positive/negative value should generally not be relied on; the sign is what matters.

---

# 30. `String.valueOf()`

Converts values to their String representation.

```java
int number = 100;

String value =
        String.valueOf(number);
```

It is useful when converting primitives or objects to String form.

---

# 31. Null and String Conversion

Be careful with:

```java
String.valueOf((Object) null);
```

which returns:

```text
"null"
```

But:

```java
String.valueOf((char[]) null);
```

has different overload behavior and can throw `NullPointerException`.

The general lesson is to understand overload selection when working with null values.

---

# 32. StringBuilder Methods

Common methods:

```java
append()
insert()
delete()
deleteCharAt()
replace()
reverse()
length()
capacity()
charAt()
setCharAt()
toString()
```

Example:

```java
StringBuilder builder =
        new StringBuilder("Java");

builder.append(" Backend");
builder.reverse();
```

---

# 33. StringBuilder Capacity

A StringBuilder maintains an internal capacity.

Example:

```java
StringBuilder builder =
        new StringBuilder();
```

As characters are appended, its capacity can grow.

You can specify an initial capacity:

```java
StringBuilder builder =
        new StringBuilder(100);
```

This can reduce repeated resizing when the approximate output size is known.

---

# 34. Why Is StringBuilder Faster for Repeated Concatenation?

Consider:

```java
String result = "";

for (int i = 0; i < 1000; i++) {
    result += i;
}
```

Because Strings are immutable, repeated concatenation can create many intermediate String results.

Prefer:

```java
StringBuilder builder =
        new StringBuilder();

for (int i = 0; i < 1000; i++) {
    builder.append(i);
}

String result = builder.toString();
```

This avoids repeatedly creating a new immutable String for each append operation.

---

# 35. StringBuilder vs StringBuffer

Use:

```java
StringBuilder
```

for most local string-building operations.

Use:

```java
StringBuffer
```

when you specifically need its synchronized API.

Do not assume `StringBuffer` is automatically the best solution simply because multiple threads exist. Often, designing thread ownership correctly is better than sharing a mutable buffer.

---

# 36. String Thread Safety

Because String is immutable, the same String object can safely be shared between threads without synchronization for its state.

Example:

```java
String message = "Hello";

Thread t1 = ...;
Thread t2 = ...;
```

Both threads can read the same String safely.

This does not mean every operation involving Strings is automatically thread-safe; shared mutable state around the String can still require synchronization.

---

# 37. Strings as HashMap Keys

Strings are commonly used as keys:

```java
Map<String, User> users =
        new HashMap<>();
```

Immutability is important here.

If a key's equality-relevant state could change after insertion, hash-based collections could become difficult to use correctly.

String's stable content and hash code make it a good key type.

---

# 38. `hashCode()` and String

Two equal Strings have the same hash code:

```java
String a = "Java";
String b = new String("Java");

System.out.println(a.equals(b));
System.out.println(a.hashCode() == b.hashCode());
```

Output:

```text
true
true
```

This follows the `equals()`/`hashCode()` contract.

---

# 39. String Pool vs Heap

Conceptually:

```text
String a = "Java";
```

uses a pooled String literal.

While:

```java
String b = new String("Java");
```

creates a distinct String object.

The exact JVM memory implementation is more nuanced than simply saying "pool is in stack" or "pool is in heap." Avoid those oversimplified interview statements.

### Interview Answer

> String literals are managed through the JVM's String pool, while explicitly creating a String with `new` creates a distinct object. The String pool itself is managed as part of JVM heap memory in modern JVMs.

---

# 40. String Interning

Example:

```java
String a = new String("Java");
String b = a.intern();
String c = "Java";
```

Then:

```java
b == c
```

is:

```text
true
```

while:

```java
a == c
```

is:

```text
false
```

---

# 41. String Constant Expressions

The compiler can evaluate certain constant String expressions.

Example:

```java
String a = "Ja" + "va";
String b = "Java";

System.out.println(a == b);
```

This can be:

```text
true
```

because the concatenation consists entirely of compile-time constants.

But:

```java
String x = "Ja";
String a = x + "va";
String b = "Java";
```

is different because the concatenation involves a variable.

Do not rely on `==` for String content comparison.

---

# 42. String Concatenation in Loops

Bad for repeated concatenation:

```java
String result = "";

for (String value : values) {
    result += value;
}
```

Better:

```java
StringBuilder result =
        new StringBuilder();

for (String value : values) {
    result.append(value);
}
```

Then:

```java
String finalResult =
        result.toString();
```

---

# 43. Common String Interview Questions

1. Why is String immutable?
2. What is the String pool?
3. What is the difference between `==` and `equals()`?
4. Why does `"Java" == "Java"` return true?
5. What happens with `new String("Java")`?
6. What is `intern()`?
7. String vs StringBuilder?
8. StringBuilder vs StringBuffer?
9. Why is StringBuilder faster for repeated modifications?
10. What is the difference between `isEmpty()` and `isBlank()`?
11. `trim()` vs `strip()`?
12. `replace()` vs `replaceAll()`?
13. What does `substring()` do?
14. What does `compareTo()` return?
15. Why are Strings good HashMap keys?
16. Is String thread-safe?
17. Can String be extended?
18. Why is String final?
19. What is String interning?
20. Where is the String pool stored?
21. What happens when Strings are concatenated?
22. Why should we avoid repeated `+` inside loops?
23. What is the difference between `StringBuilder` capacity and length?
24. How would you reverse a String?
25. How would you check whether two Strings are anagrams?
26. How would you find duplicate characters?
27. How would you find the first non-repeating character?

---

# 44. Coding Problem — Reverse a String

Using StringBuilder:

```java
public static String reverse(String value) {

    return new StringBuilder(value)
            .reverse()
            .toString();
}
```

Without StringBuilder:

```java
public static String reverse(String value) {

    char[] chars = value.toCharArray();

    int left = 0;
    int right = chars.length - 1;

    while (left < right) {

        char temp = chars[left];

        chars[left] = chars[right];
        chars[right] = temp;

        left++;
        right--;
    }

    return new String(chars);
}
```

---

# 45. Coding Problem — Check Palindrome

```java
public static boolean isPalindrome(String value) {

    int left = 0;
    int right = value.length() - 1;

    while (left < right) {

        if (value.charAt(left) !=
            value.charAt(right)) {

            return false;
        }

        left++;
        right--;
    }

    return true;
}
```

Time complexity:

```text
O(n)
```

Space complexity:

```text
O(1)
```

assuming character access and comparison are constant-time.

---

# 46. Coding Problem — Count Characters

```java
public static Map<Character, Integer>
countCharacters(String value) {

    Map<Character, Integer> frequency =
            new HashMap<>();

    for (char ch : value.toCharArray()) {

        frequency.put(
            ch,
            frequency.getOrDefault(ch, 0) + 1
        );
    }

    return frequency;
}
```

This pattern is useful for:

- Frequency counting
- Anagrams
- Duplicate detection
- Character statistics

---

# 47. Coding Problem — First Non-Repeating Character

```java
public static Character firstUnique(
        String value) {

    Map<Character, Integer> frequency =
            new LinkedHashMap<>();

    for (char ch : value.toCharArray()) {

        frequency.put(
            ch,
            frequency.getOrDefault(ch, 0) + 1
        );
    }

    for (Map.Entry<Character, Integer> entry :
            frequency.entrySet()) {

        if (entry.getValue() == 1) {
            return entry.getKey();
        }
    }

    return null;
}
```

Why `LinkedHashMap`?

Because it preserves insertion order.

---

# 48. Coding Problem — Anagram Check

A simple approach:

```java
public static boolean isAnagram(
        String first,
        String second) {

    if (first.length() != second.length()) {
        return false;
    }

    char[] a = first.toCharArray();
    char[] b = second.toCharArray();

    Arrays.sort(a);
    Arrays.sort(b);

    return Arrays.equals(a, b);
}
```

Time complexity:

```text
O(n log n)
```

A frequency-map approach can achieve:

```text
O(n)
```

average time with additional space.

---

# 49. Coding Problem — Remove Duplicate Characters

Example:

```text
Input:
programming

Output:
progamin
```

Using `LinkedHashSet`:

```java
public static String removeDuplicates(
        String value) {

    Set<Character> chars =
            new LinkedHashSet<>();

    for (char ch : value.toCharArray()) {
        chars.add(ch);
    }

    StringBuilder result =
            new StringBuilder();

    for (char ch : chars) {
        result.append(ch);
    }

    return result.toString();
}
```

`LinkedHashSet` preserves insertion order.

---

# 50. Backend Example — Build an API Message

Suppose a backend needs to construct a response message from multiple values.

Prefer:

```java
StringBuilder message =
        new StringBuilder();

message.append("Order ID: ")
       .append(orderId)
       .append(", Status: ")
       .append(status)
       .append(", Amount: ")
       .append(amount);

return message.toString();
```

For a small number of simple concatenations, normal `+` is perfectly readable:

```java
String message =
        "Order ID: " + orderId +
        ", Status: " + status;
```

Don't replace every `+` with StringBuilder unnecessarily.

---

# 51. Java String Interview Traps

### Trap 1

```java
String a = "Java";
String b = "Java";

a == b
```

Likely:

```text
true
```

because of pooling.

But use:

```java
a.equals(b)
```

for content comparison.

---

### Trap 2

```java
String a = new String("Java");
String b = new String("Java");

a == b
```

Result:

```text
false
```

---

### Trap 3

```java
String a = "Java";

a.concat("8");

System.out.println(a);
```

Result:

```text
Java
```

Because String is immutable.

---

### Trap 4

```java
StringBuilder builder =
        new StringBuilder("Java");

builder.append("8");

System.out.println(builder);
```

Result:

```text
Java8
```

Because StringBuilder is mutable.

---

# 52. Quick Revision

```text
String
    ↓
Immutable

String Pool
    ↓
Stores/reuses canonical String literals

==
    ↓
Reference identity for objects

equals()
    ↓
Logical/content equality

StringBuilder
    ↓
Mutable, generally preferred for local repeated modifications

StringBuffer
    ↓
Mutable + synchronized API

intern()
    ↓
Canonical pooled String

isEmpty()
    ↓
Length == 0

isBlank()
    ↓
Empty or whitespace-only

trim()
    ↓
Legacy-style whitespace removal

strip()
    ↓
Unicode-aware whitespace removal

replace()
    ↓
Literal replacement

replaceAll()
    ↓
Regex replacement

split()
    ↓
Regex-based splitting
```

---

# 53. Strong Interview Answer

### Question: "Why is String immutable in Java?"

A good answer:

> String is immutable so that its value cannot change after creation. This provides benefits such as security, thread safety, stable hash codes and efficient reuse through the String pool. It also makes Strings reliable as keys in hash-based collections. When we need repeated modifications, we can use StringBuilder instead.

---

# 54. Interview Mindset

Don't stop at:

> "String is immutable."

Be ready for the follow-up chain:

```text
Why immutable?
      ↓
String pool
      ↓
Why pooling is useful
      ↓
Why == sometimes returns true
      ↓
Why equals() should be used
      ↓
Why StringBuilder exists
      ↓
StringBuilder vs StringBuffer
      ↓
String as HashMap key
      ↓
Performance in loops
```

These follow-ups are extremely common in Java backend interviews.
