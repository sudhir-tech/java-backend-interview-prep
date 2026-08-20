# Java Collections Framework

The Java Collections Framework provides interfaces, implementations and utility methods for storing and manipulating groups of objects.

The most important collection types for backend interviews are:

- List
- Set
- Map
- Queue
- Deque

---

# 1. Collection Hierarchy

A simplified view:

```text
                    Iterable
                       |
                   Collection
              _________|_________
             |         |         |
            List       Set      Queue
             |         |         |
        ArrayList   HashSet   PriorityQueue
        LinkedList  LinkedHashSet
                    TreeSet

                    Map
                     |
          ______________________
         |          |           |
      HashMap   LinkedHashMap  TreeMap
         |
 ConcurrentHashMap
```

> `Map` is part of the Java Collections Framework, but it does not extend the `Collection` interface.

---

# 2. List

A `List` is an ordered collection that:

- Maintains insertion order
- Allows duplicate elements
- Supports index-based access

Common implementations:

- ArrayList
- LinkedList
- Vector

---

## ArrayList

`ArrayList` is backed by a dynamically resizable array.

### Example

```java
import java.util.ArrayList;
import java.util.List;

public class Example {

    public static void main(String[] args) {

        List<String> names = new ArrayList<>();

        names.add("Sudhir");
        names.add("Alex");
        names.add("John");

        System.out.println(names.get(0));
    }
}
```

### Advantages

- Fast random access
- Simple to use
- Good cache locality
- Excellent for read-heavy operations

### Time Complexity

| Operation | Average |
|---|---:|
| get(index) | O(1) |
| set(index) | O(1) |
| add(element) | O(1) amortized |
| add(index, element) | O(n) |
| remove(index) | O(n) |
| contains(element) | O(n) |

### Interview Answer

> ArrayList is a resizable array implementation of List. It provides O(1) random access but insertion or deletion in the middle is generally O(n).

---

# 3. LinkedList

`LinkedList` is implemented as a doubly linked list.

Each node contains:

```text
previous | data | next
```

### Example

```java
import java.util.LinkedList;
import java.util.List;

List<String> names = new LinkedList<>();

names.add("Sudhir");
names.add("Alex");
names.addFirst("John");
names.addLast("Mike");
```

### Advantages

- Efficient insertion/removal at known ends
- Can be used as a List
- Can also be used as a Queue or Deque

### Disadvantages

- Slow random access
- Higher memory overhead
- Poorer cache locality than ArrayList

### Time Complexity

| Operation | Complexity |
|---|---:|
| get(index) | O(n) |
| addFirst | O(1) |
| addLast | O(1) |
| removeFirst | O(1) |
| removeLast | O(1) |
| contains | O(n) |

### Interview Tip

For most normal application code, **ArrayList is usually the better default** when you need a List and frequent random access.

---

# 4. ArrayList vs LinkedList

| ArrayList | LinkedList |
|---|---|
| Dynamic array | Doubly linked list |
| Fast random access | Slow random access |
| `get()` is O(1) | `get()` is O(n) |
| Better cache locality | Poorer cache locality |
| Less memory overhead | More memory overhead |
| Usually preferred for general List usage | Useful when frequent insertion/removal at ends is required |

### Interview Answer

> I would generally choose ArrayList unless the access pattern specifically benefits from LinkedList. ArrayList provides fast random access and usually has better cache locality.

---

# 5. Set

A `Set` is a collection that does **not allow duplicate elements**.

Common implementations:

- HashSet
- LinkedHashSet
- TreeSet

---

# 6. HashSet

`HashSet` stores unique elements using hashing.

### Example

```java
import java.util.HashSet;
import java.util.Set;

Set<String> languages = new HashSet<>();

languages.add("Java");
languages.add("Python");
languages.add("Java");

System.out.println(languages);
```

The second `"Java"` is not added because a Set does not allow duplicates.

### Important Properties

- No duplicate elements
- No guaranteed iteration order
- Average O(1) add, remove and contains
- Allows one `null` element

### Time Complexity

| Operation | Average |
|---|---:|
| add | O(1) |
| remove | O(1) |
| contains | O(1) |

### Interview Answer

> HashSet uses hashing to store unique elements and provides average O(1) insertion, deletion and lookup.

---

# 7. LinkedHashSet

`LinkedHashSet` maintains insertion order while still preventing duplicates.

```java
Set<String> names = new LinkedHashSet<>();

names.add("John");
names.add("Alex");
names.add("Sudhir");

System.out.println(names);
```

Output preserves insertion order:

```text
[John, Alex, Sudhir]
```

### Use Case

Use it when you need:

> **Uniqueness + insertion order**

---

# 8. TreeSet

`TreeSet` stores unique elements in sorted order.

```java
Set<Integer> numbers = new TreeSet<>();

numbers.add(50);
numbers.add(10);
numbers.add(30);

System.out.println(numbers);
```

Output:

```text
[10, 30, 50]
```

### Time Complexity

| Operation | Complexity |
|---|---:|
| add | O(log n) |
| remove | O(log n) |
| contains | O(log n) |

### Use Case

Use `TreeSet` when you need:

> **Uniqueness + sorted order**

---

# 9. HashSet vs LinkedHashSet vs TreeSet

| HashSet | LinkedHashSet | TreeSet |
|---|---|---|
| No guaranteed order | Insertion order | Sorted order |
| Average O(1) | Average O(1) | O(log n) |
| Hash table based | Hash table + linked structure | Tree-based |
| Fast lookup | Fast lookup + order | Sorted data |

---

# 10. Map

A `Map` stores data as **key-value pairs**.

Example:

```text
Key       Value
----------------
101       Sudhir
102       Alex
103       John
```

Important implementations:

- HashMap
- LinkedHashMap
- TreeMap
- ConcurrentHashMap

---

# 11. HashMap

`HashMap` stores key-value pairs using hashing.

### Example

```java
import java.util.HashMap;
import java.util.Map;

Map<Integer, String> users = new HashMap<>();

users.put(101, "Sudhir");
users.put(102, "Alex");
users.put(103, "John");

System.out.println(users.get(101));
```

Output:

```text
Sudhir
```

### Important Properties

- Average O(1) put/get/remove
- Does not guarantee iteration order
- Allows one `null` key
- Allows multiple `null` values
- Not thread-safe
- Keys should have a consistent `equals()` and `hashCode()` implementation

---

# 12. How HashMap Works Internally

This is one of the **most important Java interview questions**.

Conceptually:

```text
Key
 ↓
hashCode()
 ↓
hash spreading
 ↓
bucket index
 ↓
bucket
 ↓
key comparison using equals()
```

When you execute:

```java
map.put("Java", 100);
```

HashMap roughly performs:

```text
"Java"
   ↓
hashCode()
   ↓
calculate bucket index
   ↓
store key-value entry
```

When retrieving:

```java
map.get("Java");
```

HashMap calculates the appropriate bucket and then checks keys using `equals()` to find the matching entry.

---

# 13. What Happens When Two Keys Have the Same Hash?

This is called a **hash collision**.

Example:

```text
Key A → hash → bucket 5
Key B → hash → bucket 5
```

Both keys map to the same bucket.

HashMap handles collisions by storing multiple entries in the same bucket.

In modern Java, heavily populated buckets can be converted from a linked structure into a balanced tree under appropriate conditions, improving worst-case lookup behavior.

### Interview Answer

> Hash collisions occur when multiple keys map to the same bucket. HashMap handles collisions within the bucket and compares keys using equals() to identify the correct entry.

---

# 14. Why Are equals() and hashCode() Important?

Hash-based collections depend on both methods.

The contract is:

> If two objects are equal according to `equals()`, they must return the same `hashCode()`.

Example:

```java
class Employee {

    private int id;

    Employee(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Employee)) {
            return false;
        }

        Employee other = (Employee) obj;

        return this.id == other.id;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

If you override `equals()` without correctly overriding `hashCode()`, hash-based collections can behave unexpectedly.

---

# 15. HashMap Capacity and Load Factor

HashMap has a capacity and a load factor.

The default load factor is commonly **0.75**.

The resize threshold is approximately:

```text
capacity × load factor
```

For example:

```text
capacity = 16
load factor = 0.75

threshold = 16 × 0.75
          = 12
```

When the number of entries exceeds the threshold, HashMap resizes and redistributes entries.

### Interview Question

**Why is the load factor 0.75?**

### Answer

> It provides a practical balance between memory usage and lookup performance. A lower load factor means more empty space, while a higher load factor reduces memory overhead but increases the likelihood of collisions.

---

# 16. LinkedHashMap

`LinkedHashMap` maintains a predictable iteration order.

By default, it maintains **insertion order**.

```java
Map<Integer, String> users = new LinkedHashMap<>();

users.put(1, "A");
users.put(2, "B");
users.put(3, "C");
```

Iteration follows:

```text
1 → 2 → 3
```

It can also be configured for access order, which makes it useful for implementing LRU-style caches.

### Use Case

> HashMap + predictable ordering

---

# 17. TreeMap

`TreeMap` stores entries sorted by key.

```java
Map<Integer, String> users = new TreeMap<>();

users.put(30, "John");
users.put(10, "Alex");
users.put(20, "Sudhir");

System.out.println(users);
```

Output:

```text
{10=Alex, 20=Sudhir, 30=John}
```

### Time Complexity

| Operation | Complexity |
|---|---:|
| put | O(log n) |
| get | O(log n) |
| remove | O(log n) |

### Use Case

Use `TreeMap` when you need:

> **Key-value storage + sorted keys**

---

# 18. HashMap vs LinkedHashMap vs TreeMap

| HashMap | LinkedHashMap | TreeMap |
|---|---|---|
| No guaranteed order | Insertion/access order | Sorted by key |
| Average O(1) | Average O(1) | O(log n) |
| Hash table | Hash table + linked structure | Tree-based |
| Best for general lookup | Lookup + predictable order | Sorted/range-based operations |

---

# 19. ConcurrentHashMap

`ConcurrentHashMap` is designed for concurrent access from multiple threads.

```java
Map<String, Integer> counts = new ConcurrentHashMap<>();

counts.put("Java", 1);
counts.put("Spring", 2);
```

It is useful when multiple threads need to read and update a shared map safely.

### Important

`ConcurrentHashMap` does **not** allow `null` keys or `null` values.

### Interview Answer

> ConcurrentHashMap provides thread-safe access to a map while allowing much better concurrency than synchronizing an entire HashMap.

---

# 20. HashMap vs ConcurrentHashMap

| HashMap | ConcurrentHashMap |
|---|---|
| Not thread-safe | Thread-safe for concurrent access |
| Allows null key/value | Does not allow null keys or values |
| Suitable for single-threaded use | Suitable for concurrent applications |
| Better when no synchronization is required | Designed for concurrent access |

---

# 21. Queue

A Queue is generally used when elements are processed in an ordered sequence.

Typical FIFO behavior:

```text
First In → First Out
```

Example:

```java
Queue<String> queue = new LinkedList<>();

queue.offer("A");
queue.offer("B");
queue.offer("C");

System.out.println(queue.poll());
```

Output:

```text
A
```

Important methods:

| Method | Purpose |
|---|---|
| offer() | Add element |
| poll() | Remove and return head |
| peek() | View head without removing |

---

# 22. PriorityQueue

`PriorityQueue` processes elements according to priority rather than insertion order.

By default, the smallest element has the highest priority.

```java
PriorityQueue<Integer> queue = new PriorityQueue<>();

queue.offer(30);
queue.offer(10);
queue.offer(20);

System.out.println(queue.poll());
```

Output:

```text
10
```

### Common Use Cases

- Top K problems
- Scheduling
- Finding minimum/maximum elements
- Dijkstra's algorithm

---

# 23. Deque

`Deque` means **Double Ended Queue**.

Elements can be added or removed from both ends.

```java
Deque<Integer> deque = new ArrayDeque<>();

deque.addFirst(10);
deque.addLast(20);

System.out.println(deque.removeFirst());
```

`ArrayDeque` is usually preferred over the legacy `Stack` class for stack behavior.

---

# 24. Stack

Java has the legacy `Stack` class, but modern Java code commonly uses `Deque` / `ArrayDeque` for stack operations.

Example:

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(10);
stack.push(20);

System.out.println(stack.pop());
```

Output:

```text
20
```

This follows:

```text
LIFO
Last In → First Out
```

---

# 25. Comparable vs Comparator

This is another very common interview topic.

## Comparable

Comparable defines the **natural ordering** of a class.

```java
class Employee implements Comparable<Employee> {

    int salary;

    Employee(int salary) {
        this.salary = salary;
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}
```

Then:

```java
Collections.sort(employees);
```

### Interview Answer

> Comparable is used when a class has a natural ordering and defines that ordering inside the class using compareTo().

---

## Comparator

Comparator allows you to define different sorting strategies externally.

```java
Comparator<Employee> bySalary =
        Comparator.comparingInt(employee -> employee.salary);
```

Then:

```java
employees.sort(bySalary);
```

### Interview Answer

> Comparator is useful when you need different sorting strategies without modifying the class itself.

---

# 26. Comparable vs Comparator

| Comparable | Comparator |
|---|---|
| Defines natural ordering | Defines custom ordering |
| Uses `compareTo()` | Uses `compare()` |
| Implemented by the class | Usually defined separately |
| Usually one natural ordering | Can have multiple strategies |

---

# 27. Iterator

An `Iterator` is used to traverse a collection.

```java
List<String> names = new ArrayList<>();

names.add("Java");
names.add("Spring");

Iterator<String> iterator = names.iterator();

while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

You can safely remove elements during iteration using the iterator's `remove()` method where supported.

---

# 28. Fail-Fast Behavior

Many standard collection iterators are **fail-fast**.

Example:

```java
List<String> names = new ArrayList<>();

names.add("A");
names.add("B");

for (String name : names) {

    if (name.equals("A")) {
        names.remove(name);
    }
}
```

This can result in a `ConcurrentModificationException`.

A safer approach is:

```java
Iterator<String> iterator = names.iterator();

while (iterator.hasNext()) {

    String name = iterator.next();

    if (name.equals("A")) {
        iterator.remove();
    }
}
```

> Fail-fast behavior is a best-effort mechanism to detect structural modification during iteration. It should not be treated as a synchronization mechanism.

---

# 29. Collections Utility Class

`Collections` provides utility methods for working with collections.

Examples:

```java
Collections.sort(list);

Collections.reverse(list);

Collections.shuffle(list);

Collections.max(list);

Collections.min(list);
```

Example:

```java
List<Integer> numbers =
        new ArrayList<>(List.of(5, 2, 8, 1));

Collections.sort(numbers);

System.out.println(numbers);
```

Output:

```text
[1, 2, 5, 8]
```

---

# 30. Collection vs Collections

This is a common interview question.

### Collection

`Collection` is an **interface**.

```java
Collection<String> names = new ArrayList<>();
```

### Collections

`Collections` is a **utility class** containing static methods.

```java
Collections.sort(names);
```

### Quick Answer

> Collection is an interface representing a group of objects, while Collections is a utility class providing helper methods for working with collections.

---

# 31. Collection vs Map

| Collection | Map |
|---|---|
| Stores individual elements | Stores key-value pairs |
| Root interface for List, Set, Queue | Separate hierarchy |
| Examples: ArrayList, HashSet | Examples: HashMap, TreeMap |
| `add()` commonly used | `put()` commonly used |

---

# 32. List vs Set

| List | Set |
|---|---|
| Allows duplicates | Does not allow duplicates |
| Maintains order depending on implementation | Order depends on implementation |
| Supports index access | No index-based access |
| ArrayList, LinkedList | HashSet, LinkedHashSet, TreeSet |

---

# 33. HashMap Key Best Practices

Keys used in a HashMap should ideally be:

- Immutable
- Consistent with `equals()`
- Consistent with `hashCode()`

Good examples:

```java
String
Integer
Long
UUID
```

A mutable object used as a key can cause lookup problems if fields involved in `equals()` / `hashCode()` are changed after insertion.

---

# 34. Why Are Strings Good HashMap Keys?

`String` is commonly used as a HashMap key because it is immutable and has a well-defined `equals()` and `hashCode()` implementation.

Example:

```java
Map<String, Integer> scores = new HashMap<>();

scores.put("Sudhir", 95);

System.out.println(scores.get("Sudhir"));
```

---

# 35. Choosing the Right Collection

A useful decision guide:

```text
Need ordered elements?
        |
       Yes
        |
Need duplicates?
   /             \
 Yes              No
  |                |
List              Set
  |                |
ArrayList       HashSet
LinkedList      LinkedHashSet
                TreeSet

Need key-value pairs?
        |
       Yes
        |
Need sorted keys?
   /          \
 Yes           No
  |             |
TreeMap       HashMap
                |
        Need predictable order?
             |
        LinkedHashMap

Need thread-safe map?
        |
ConcurrentHashMap
```

---

# 36. Common Interview Questions

## Core Questions

1. What is the Java Collections Framework?
2. What is the difference between Collection and Collections?
3. What is the difference between Collection and Map?
4. List vs Set?
5. ArrayList vs LinkedList?
6. HashSet vs TreeSet?
7. HashSet vs LinkedHashSet?
8. HashMap vs Hashtable?
9. HashMap vs ConcurrentHashMap?
10. HashMap vs LinkedHashMap?
11. HashMap vs TreeMap?
12. Comparable vs Comparator?
13. Iterator vs ListIterator?
14. What is fail-fast behavior?
15. Why does HashMap allow one null key?
16. Why doesn't ConcurrentHashMap allow null?
17. How does HashMap work internally?
18. What is a hash collision?
19. Why are equals() and hashCode() important?
20. What is the load factor?
21. When does HashMap resize?
22. What happens when two keys have the same hash?
23. Why should HashMap keys be immutable?
24. ArrayList vs Vector?
25. Why is ArrayDeque preferred over Stack?

---

# 37. Interview Follow-Up: HashMap

If you say:

> "HashMap provides O(1) lookup."

Be prepared for:

- Average or worst case?
- How is the bucket calculated?
- What happens during a collision?
- What is hashCode()?
- What is equals()?
- What happens during resizing?
- What is load factor?
- What happens when a bucket becomes heavily populated?
- Why are immutable keys preferred?
- Is HashMap thread-safe?

A strong answer should mention that O(1) is **average expected complexity**, not an unconditional guarantee.

---

# 38. Interview Follow-Up: ArrayList

If you say:

> "ArrayList is fast."

Expect:

- Why is get() O(1)?
- What happens when capacity is exceeded?
- What is resizing?
- Why is insertion in the middle O(n)?
- ArrayList vs LinkedList?
- Is ArrayList thread-safe?
- How would you make a collection thread-safe?

---

# 39. Interview Follow-Up: ConcurrentHashMap

Expect:

- Why use ConcurrentHashMap?
- Why doesn't it allow null?
- HashMap vs ConcurrentHashMap?
- Is the entire map locked?
- What happens when multiple threads update it?
- What is atomic compound operation?
- When would you use `computeIfAbsent()`?

Example:

```java
ConcurrentHashMap<String, Integer> counts =
        new ConcurrentHashMap<>();

counts.computeIfAbsent("Java", key -> 0);
```

---

# 40. Quick Revision

```text
ArrayList
    ↓
Dynamic array
Fast random access
Allows duplicates

LinkedList
    ↓
Doubly linked list
Useful for operations at ends
Slow random access

HashSet
    ↓
Unique elements
Average O(1)
No guaranteed order

LinkedHashSet
    ↓
Unique + insertion order

TreeSet
    ↓
Unique + sorted order
O(log n)

HashMap
    ↓
Key-value pairs
Average O(1)
No guaranteed order

LinkedHashMap
    ↓
Key-value pairs + predictable order

TreeMap
    ↓
Key-value pairs + sorted keys
O(log n)

ConcurrentHashMap
    ↓
Concurrent key-value access

PriorityQueue
    ↓
Priority-based processing

ArrayDeque
    ↓
Efficient stack/queue operations
```

---

# 🎯 Interview Mindset

Don't memorize collection names only.

For every collection, know:

```text
What data structure is underneath?
        ↓
Does it allow duplicates?
        ↓
Does it maintain order?
        ↓
Does it sort?
        ↓
What is the average time complexity?
        ↓
Is it thread-safe?
        ↓
When would I use it in a real application?
```

The most important collections to master first are:

1. `ArrayList`
2. `HashSet`
3. `HashMap`
4. `LinkedHashMap`
5. `TreeMap`
6. `ConcurrentHashMap`
7. `PriorityQueue`
8. `ArrayDeque`

---

# 🚀 Real Backend Examples

### HashMap

Useful for fast lookup:

```java
Map<Long, User> usersById = new HashMap<>();
```

### LinkedHashMap

Useful when predictable iteration order matters:

```java
Map<String, String> headers = new LinkedHashMap<>();
```

### TreeMap

Useful when keys need to remain sorted:

```java
Map<Integer, Order> ordersById = new TreeMap<>();
```

### HashSet

Useful for tracking unique values:

```java
Set<Long> processedOrderIds = new HashSet<>();
```

### ConcurrentHashMap

Useful when multiple threads access shared state:

```java
ConcurrentHashMap<String, Integer> requestCounts =
        new ConcurrentHashMap<>();
```

### PriorityQueue

Useful for priority-based processing:

```java
PriorityQueue<Task> tasks = new PriorityQueue<>();
```

---

# ⭐ Final Takeaway

For Java Backend interviews, **HashMap, ArrayList, HashSet, ConcurrentHashMap, Comparable/Comparator and collection time complexities are must-know topics**.

Don't just remember:

> "HashMap is O(1)."

Be ready to explain:

> **Why** it is approximately O(1), how hashing works, what happens during collisions, how `equals()` and `hashCode()` work together, and what changes when multiple threads access the map.

That level of understanding is what turns a memorized answer into a strong interview answer.
