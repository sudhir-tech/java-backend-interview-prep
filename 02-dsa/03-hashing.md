# DSA — Hashing

Hashing is one of the most important DSA patterns for Java backend interviews.

It is commonly used to reduce lookup time from `O(n)` to average `O(1)` and appears in problems involving:

- Frequency counting
- Duplicate detection
- Two Sum
- Anagrams
- Subarrays
- Prefix sums
- Caching
- Grouping
- Fast membership checks

---

## 1. What is Hashing?

Hashing converts a key into a hash value that helps determine where the key-value entry should be stored.

In Java, the most commonly used hashing structures are:

```java
HashMap<K, V>
HashSet<E>
```

Example:

```java
Map<String, Integer> map = new HashMap<>();

map.put("Java", 10);
map.put("Spring", 20);

System.out.println(map.get("Java"));
```

Output:

```text
10
```

---

## 2. HashMap

`HashMap` stores key-value pairs.

```java
Map<String, Integer> map = new HashMap<>();
```

Example:

```java
map.put("Alice", 90);
map.put("Bob", 85);
map.put("Charlie", 95);
```

Retrieve:

```java
int score = map.get("Alice");
```

---

## 3. HashMap Complexity

Average-case complexity:

| Operation | Average |
|---|---:|
| `put()` | O(1) |
| `get()` | O(1) |
| `containsKey()` | O(1) |
| `remove()` | O(1) |

Worst-case behavior depends on collisions and implementation details.

### Interview answer

> HashMap provides average O(1) insertion, lookup and deletion by using hashing.

---

# 4. HashSet

`HashSet` stores unique values.

```java
Set<Integer> set = new HashSet<>();

set.add(10);
set.add(20);
set.add(10);
```

The second `10` is ignored.

```java
System.out.println(set.size());
```

Output:

```text
2
```

---

# 5. HashSet Complexity

Average:

```text
add()       → O(1)
remove()    → O(1)
contains()  → O(1)
```

This makes HashSet useful for fast membership checking.

---

# 6. HashMap vs HashSet

| HashMap | HashSet |
|---|---|
| Stores key-value pairs | Stores values |
| `put(key, value)` | `add(value)` |
| `get(key)` | `contains(value)` |
| Useful for mappings | Useful for uniqueness/membership |

Example:

```text
HashMap:
userId → userName

HashSet:
unique userIds
```

---

# 7. Frequency Counting

One of the most common hashing patterns.

Example:

```text
Input:
banana

Output:
b → 1
a → 3
n → 2
```

Java:

```java
Map<Character, Integer> frequency =
    new HashMap<>();

for (char c : s.toCharArray()) {
    frequency.merge(
        c,
        1,
        Integer::sum
    );
}
```

Alternative:

```java
frequency.put(
    c,
    frequency.getOrDefault(c, 0) + 1
);
```

---

# 8. Frequency Counting for Integers

```java
Map<Integer, Integer> frequency =
    new HashMap<>();

for (int value : arr) {
    frequency.put(
        value,
        frequency.getOrDefault(value, 0) + 1
    );
}
```

This is useful for:

- Finding duplicates
- Finding most frequent values
- Counting occurrences
- Frequency-based sorting

---

# 9. Find Duplicates

```java
Set<Integer> seen = new HashSet<>();

for (int value : arr) {

    if (!seen.add(value)) {
        System.out.println(
            "Duplicate: " + value
        );
    }
}
```

Why does this work?

`Set.add()` returns:

```text
true  → value was not already present
false → value already existed
```

---

# 10. Contains Duplicate

```java
Set<Integer> set = new HashSet<>();

for (int value : nums) {

    if (!set.add(value)) {
        return true;
    }
}

return false;
```

### Complexity

```text
Time:  O(n) average
Space: O(n)
```

---

# 11. Two Sum Using HashMap

Given an array and target:

```text
nums = [2, 7, 11, 15]
target = 9
```

Answer:

```text
[0, 1]
```

Optimized solution:

```java
Map<Integer, Integer> map =
    new HashMap<>();

for (int i = 0; i < nums.length; i++) {

    int complement =
        target - nums[i];

    if (map.containsKey(complement)) {
        return new int[]{
            map.get(complement),
            i
        };
    }

    map.put(nums[i], i);
}
```

### Complexity

```text
Time:  O(n) average
Space: O(n)
```

---

# 12. Why HashMap Makes Two Sum Faster

Brute force checks every pair:

```text
O(n²)
```

HashMap allows us to ask:

```text
Have I already seen target - current?
```

in average:

```text
O(1)
```

Therefore:

```text
O(n²) → O(n)
```

This is a classic example of using extra space to improve time complexity.

---

# 13. Group Anagrams

Example:

```text
["eat", "tea", "tan", "ate", "nat", "bat"]
```

Group:

```text
["eat", "tea", "ate"]
["tan", "nat"]
["bat"]
```

A simple approach is to sort each word and use the sorted word as the key.

```java
Map<String, List<String>> groups =
    new HashMap<>();

for (String word : words) {

    char[] chars =
        word.toCharArray();

    Arrays.sort(chars);

    String key =
        new String(chars);

    groups
        .computeIfAbsent(
            key,
            k -> new ArrayList<>()
        )
        .add(word);
}
```

### Complexity

If there are `n` words and average word length is `m`:

```text
Time: O(n * m log m)
```

A frequency-based key can be faster when the alphabet is fixed.

---

# 14. First Non-Repeating Character

Approach:

```text
1. Count every character.
2. Traverse the string again.
3. Return the first character with count 1.
```

```java
Map<Character, Integer> frequency =
    new HashMap<>();

for (char c : s.toCharArray()) {
    frequency.merge(
        c,
        1,
        Integer::sum
    );
}

for (char c : s.toCharArray()) {
    if (frequency.get(c) == 1) {
        return c;
    }
}

return '\0';
```

### Complexity

```text
Time:  O(n)
Space: O(k)
```

where `k` is the number of distinct characters.

---

# 15. Intersection of Two Arrays

Example:

```text
A = [1, 2, 2, 1]
B = [2, 2]
```

If unique intersection is required:

```java
Set<Integer> first =
    new HashSet<>();

for (int value : A) {
    first.add(value);
}

Set<Integer> result =
    new HashSet<>();

for (int value : B) {
    if (first.contains(value)) {
        result.add(value);
    }
}
```

### Complexity

```text
Time:  O(n + m) average
Space: O(n + m)
```

---

# 16. Intersection with Duplicates

If duplicates matter, use frequencies.

```java
Map<Integer, Integer> frequency =
    new HashMap<>();

for (int value : nums1) {
    frequency.merge(
        value,
        1,
        Integer::sum
    );
}

List<Integer> result =
    new ArrayList<>();

for (int value : nums2) {

    int count =
        frequency.getOrDefault(value, 0);

    if (count > 0) {

        result.add(value);

        frequency.put(
            value,
            count - 1
        );
    }
}
```

---

# 17. Longest Consecutive Sequence

Example:

```text
[100, 4, 200, 1, 3, 2]
```

Answer:

```text
4
```

because:

```text
1, 2, 3, 4
```

Use a HashSet:

```java
Set<Integer> set =
    new HashSet<>();

for (int value : nums) {
    set.add(value);
}

int longest = 0;

for (int value : set) {

    if (!set.contains(value - 1)) {

        int current = value;
        int length = 1;

        while (set.contains(current + 1)) {
            current++;
            length++;
        }

        longest =
            Math.max(longest, length);
    }
}
```

### Complexity

```text
Time:  O(n) average
Space: O(n)
```

### Key insight

Only start counting when:

```text
value - 1
```

does not exist.

This prevents repeatedly scanning the same sequence.

---

# 18. Prefix Sum + HashMap

This is one of the most powerful DSA combinations.

Problem:

> Find the number of subarrays whose sum equals `k`.

Maintain:

```text
prefixSum
```

and store how many times each prefix sum has occurred.

```java
Map<Integer, Integer> prefixCount =
    new HashMap<>();

prefixCount.put(0, 1);

int prefixSum = 0;
int count = 0;

for (int value : nums) {

    prefixSum += value;

    int required =
        prefixSum - k;

    count +=
        prefixCount.getOrDefault(
            required,
            0
        );

    prefixCount.put(
        prefixSum,
        prefixCount.getOrDefault(
            prefixSum,
            0
        ) + 1
    );
}

return count;
```

### Why does it work?

If:

```text
prefixSum[j] - prefixSum[i] = k
```

then:

```text
prefixSum[i] = prefixSum[j] - k
```

So while processing the current prefix sum, we look for:

```text
currentPrefix - k
```

---

# 19. Subarray Sum Equals K

This problem is extremely important.

Example:

```text
nums = [1, 1, 1]
k = 2
```

Answer:

```text
2
```

Subarrays:

```text
[1, 1]
[1, 1]
```

Use:

```text
Prefix Sum + HashMap
```

Typical complexity:

```text
Time:  O(n)
Space: O(n)
```

---

# 20. Longest Subarray with Sum K

Store the earliest index at which each prefix sum occurs.

```java
Map<Integer, Integer> firstIndex =
    new HashMap<>();

firstIndex.put(0, -1);

int prefixSum = 0;
int maximum = 0;

for (int i = 0;
     i < nums.length;
     i++) {

    prefixSum += nums[i];

    int required =
        prefixSum - k;

    if (firstIndex.containsKey(required)) {
        maximum = Math.max(
            maximum,
            i - firstIndex.get(required)
        );
    }

    firstIndex.putIfAbsent(
        prefixSum,
        i
    );
}
```

### Important

Use `putIfAbsent()` because we want the **earliest** index.

The earliest index gives the longest possible subarray.

---

# 21. Count Zero-Sum Subarrays

Use prefix sums.

```java
Map<Integer, Integer> frequency =
    new HashMap<>();

frequency.put(0, 1);

int prefix = 0;
int count = 0;

for (int value : nums) {

    prefix += value;

    count +=
        frequency.getOrDefault(
            prefix,
            0
        );

    frequency.put(
        prefix,
        frequency.getOrDefault(
            prefix,
            0
        ) + 1
    );
}
```

Why?

If the same prefix sum occurs twice:

```text
prefix[i] == prefix[j]
```

then:

```text
sum(i + 1 ... j) = 0
```

---

# 22. Find Pair with Given Difference

Given:

```text
arr
difference = k
```

Use a HashSet.

```java
Set<Integer> set =
    new HashSet<>();

for (int value : arr) {

    if (set.contains(value - k)
            || set.contains(value + k)) {
        return true;
    }

    set.add(value);
}

return false;
```

The exact condition should be adjusted depending on whether `k` can be negative and the problem definition.

---

# 23. Find Pair with Given Sum

Same core idea as Two Sum.

```java
Set<Integer> seen =
    new HashSet<>();

for (int value : nums) {

    if (seen.contains(target - value)) {
        return true;
    }

    seen.add(value);
}

return false;
```

If indexes are required, use a HashMap instead.

---

# 24. Frequency of Elements in a Range

For many frequency queries, hashing can provide fast lookup.

```java
Map<Integer, Integer> frequency =
    new HashMap<>();

for (int value : arr) {
    frequency.merge(
        value,
        1,
        Integer::sum
    );
}

int count =
    frequency.getOrDefault(
        target,
        0
    );
```

---

# 25. Top K Frequent Elements

Approach:

```text
1. Count frequencies using HashMap.
2. Use a heap or bucket structure.
3. Extract the K most frequent elements.
```

Frequency map:

```java
Map<Integer, Integer> frequency =
    new HashMap<>();

for (int value : nums) {
    frequency.merge(
        value,
        1,
        Integer::sum
    );
}
```

Then use:

```text
PriorityQueue
```

or bucket sort depending on the constraints.

---

# 26. HashMap + PriorityQueue Pattern

For Top K problems:

```java
PriorityQueue<Map.Entry<Integer, Integer>> heap =
    new PriorityQueue<>(
        Comparator.comparingInt(
            Map.Entry::getValue
        )
    );
```

Maintain only `k` elements in the heap.

### Typical complexity

```text
Frequency counting: O(n)
Heap: O(m log k)
```

where `m` is the number of distinct values.

---

# 27. Sort Characters by Frequency

Approach:

```text
1. Count characters.
2. Put characters into a max heap.
3. Build the result from highest frequency to lowest.
```

This combines:

```text
HashMap + PriorityQueue
```

---

# 28. Nearby Duplicate

Problem:

> Determine whether the same value occurs within distance `k`.

One approach uses a sliding window HashSet.

```java
Set<Integer> window =
    new HashSet<>();

for (int i = 0;
     i < nums.length;
     i++) {

    if (window.contains(nums[i])) {
        return true;
    }

    window.add(nums[i]);

    if (window.size() > k) {
        window.remove(
            nums[i - k]
        );
    }
}

return false;
```

### Complexity

```text
Time:  O(n) average
Space: O(k)
```

This is a great example of combining:

```text
HashSet + Sliding Window
```

---

# 29. Isomorphic Strings

Two strings are isomorphic if every character from one string maps consistently to exactly one character in the other.

Example:

```text
egg
add
```

Valid.

Use two maps:

```java
Map<Character, Character> sToT =
    new HashMap<>();

Map<Character, Character> tToS =
    new HashMap<>();
```

For each pair:

```java
char a = s.charAt(i);
char b = t.charAt(i);

if (sToT.containsKey(a)
        && sToT.get(a) != b) {
    return false;
}

if (tToS.containsKey(b)
        && tToS.get(b) != a) {
    return false;
}

sToT.put(a, b);
tToS.put(b, a);
```

### Why two maps?

To guarantee a one-to-one mapping.

---

# 30. Word Pattern

Example:

```text
pattern = "abba"
sentence = "dog cat cat dog"
```

Use:

```text
character → word
word → character
```

The same two-way mapping principle from isomorphic strings applies.

---

# 31. LRU Cache Concept

LRU means:

```text
Least Recently Used
```

A common Java implementation uses:

```text
HashMap + Doubly Linked List
```

The HashMap provides:

```text
O(1) lookup
```

The linked list provides:

```text
O(1) removal and insertion
```

Java also provides:

```java
LinkedHashMap
```

which can be configured for access order.

Example:

```java
Map<Integer, String> cache =
    new LinkedHashMap<>(
        16,
        0.75f,
        true
    );
```

The third argument:

```java
true
```

enables access-order behavior.

---

# 32. Hashing in Backend Development

Hashing is not only for coding interviews.

Backend systems use hashing for:

```text
Caching
Session lookup
Database indexing concepts
Deduplication
Rate limiting
Request tracking
Load distribution
Fast membership checks
Hash-based data structures
```

Example:

```text
userId
   ↓
HashMap
   ↓
User object
```

---

# 33. HashMap Internals — Interview Level

A simplified mental model:

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
key-value entry
```

When two keys map to the same bucket, a collision occurs.

Java's modern HashMap implementation can use a tree structure for heavily collided buckets under certain conditions.

### Important interview point

Do not say:

> HashMap is always O(1).

Better:

> HashMap operations are O(1) on average, with worst-case behavior depending on collisions and implementation details.

---

# 34. What is a Hash Collision?

A collision occurs when multiple keys map to the same bucket.

Example conceptually:

```text
Key A ──┐
        ├──> Bucket 5
Key B ──┘
```

The HashMap must resolve the collision while still distinguishing the keys.

---

# 35. `hashCode()` and `equals()`

Hash-based collections depend on both.

Important contract:

> If two objects are equal according to `equals()`, they must return the same `hashCode()`.

Example:

```java
@Override
public boolean equals(Object obj) {
    ...
}

@Override
public int hashCode() {
    ...
}
```

If the contract is broken, `HashMap` and `HashSet` can behave unexpectedly.

---

# 36. Why Mutable Keys Are Dangerous

Avoid changing fields that participate in `equals()` or `hashCode()` while an object is being used as a HashMap key.

Bad scenario:

```text
1. Put object into HashMap.
2. Modify its hash-relevant field.
3. Try to retrieve it.
4. Lookup may fail.
```

Prefer immutable key objects.

Good examples:

```text
String
Integer
Long
Records with appropriate semantics
Immutable value objects
```

---

# 37. HashMap Null Handling

`HashMap` allows:

```text
one null key
multiple null values
```

Example:

```java
Map<String, Integer> map =
    new HashMap<>();

map.put(null, 100);
map.put("Java", null);
```

---

# 38. HashSet and Null

`HashSet` allows one `null` element.

```java
Set<String> set =
    new HashSet<>();

set.add(null);
set.add(null);
```

Only one null is stored.

---

# 39. LinkedHashMap

`LinkedHashMap` maintains predictable iteration order.

Common use cases:

```text
Insertion-order maps
Access-order maps
LRU cache implementations
```

Example:

```java
Map<String, Integer> map =
    new LinkedHashMap<>();

map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
```

Iteration follows insertion order.

---

# 40. TreeMap vs HashMap

| HashMap | TreeMap |
|---|---|
| Hash-based | Tree-based |
| Average O(1) lookup | O(log n) lookup |
| No sorted ordering guarantee | Sorted by key |
| Allows one null key in typical usage | Null keys generally not allowed with natural ordering |

Use:

```text
HashMap → fast lookup
TreeMap → sorted keys
```

---

# 41. HashSet vs TreeSet

| HashSet | TreeSet |
|---|---|
| Hash-based | Tree-based |
| Average O(1) operations | O(log n) operations |
| No sorted order | Sorted order |
| Allows one null | Natural ordering generally does not support null |

---

# 42. Choosing the Right Hashing Structure

```text
Need key → value?
    → HashMap

Need unique values?
    → HashSet

Need insertion order?
    → LinkedHashMap / LinkedHashSet

Need sorted keys?
    → TreeMap

Need sorted unique values?
    → TreeSet
```

---

# 43. Common Hashing Patterns

When you see a problem, ask:

```text
1. Need frequency?
   → HashMap

2. Need to know whether something exists?
   → HashSet

3. Need target - current?
   → HashMap / HashSet

4. Need duplicate detection?
   → HashSet

5. Need subarray sum?
   → Prefix Sum + HashMap

6. Need longest subarray?
   → Prefix Sum + HashMap

7. Need grouping?
   → HashMap<List>

8. Need Top K?
   → HashMap + Heap

9. Need recent/nearby duplicates?
   → HashSet + Sliding Window

10. Need sorted keys?
    → TreeMap
```

---

# 44. Hashing + Prefix Sum Template

Memorize the general idea:

```java
Map<Integer, Integer> map =
    new HashMap<>();

map.put(0, 1);

int prefix = 0;

for (int value : nums) {

    prefix += value;

    int required =
        prefix - k;

    // Use required prefix.

    map.put(
        prefix,
        map.getOrDefault(prefix, 0) + 1
    );
}
```

This template appears in many subarray problems.

---

# 45. Hashing + Sliding Window Template

For fixed/limited windows:

```java
Map<Character, Integer> frequency =
    new HashMap<>();

int left = 0;

for (int right = 0;
     right < s.length();
     right++) {

    char c = s.charAt(right);

    frequency.merge(
        c,
        1,
        Integer::sum
    );

    while (/* window invalid */) {

        char leftChar =
            s.charAt(left++);

        frequency.merge(
            leftChar,
            -1,
            Integer::sum
        );

        if (frequency.get(leftChar) == 0) {
            frequency.remove(leftChar);
        }
    }

    // Process valid window.
}
```

---

# 46. Interview Questions — Easy

1. What is hashing?
2. What is a HashMap?
3. What is a HashSet?
4. What is the average complexity of HashMap lookup?
5. Find duplicates in an array.
6. Find the frequency of each element.
7. Find the first non-repeating character.
8. Check whether an array contains duplicates.
9. Check whether two strings are anagrams.
10. Find the intersection of two arrays.

---

# 47. Interview Questions — Medium

11. Two Sum.
12. Group Anagrams.
13. Longest Consecutive Sequence.
14. Subarray Sum Equals K.
15. Longest Subarray with Sum K.
16. Count Zero-Sum Subarrays.
17. Top K Frequent Elements.
18. Sort Characters by Frequency.
19. Isomorphic Strings.
20. Word Pattern.
21. Nearby Duplicate.
22. Pair with Given Difference.
23. Pair with Given Sum.
24. Longest Substring with K Distinct Characters.
25. Find All Anagrams in a String.

---

# 48. Interview Questions — Advanced

26. Design an LRU Cache.
27. Implement a custom HashMap.
28. Explain HashMap internals.
29. Explain hash collisions.
30. Explain `equals()` and `hashCode()`.
31. Explain why mutable HashMap keys are dangerous.
32. Design a frequency-based cache.
33. Design a deduplication system.
34. Solve subarray problems using prefix sum + hashing.
35. Combine hashing with heap/sliding-window techniques.

---

# 49. Common Mistakes

### Mistake 1 — Assuming HashMap is always O(1)

Say:

```text
Average O(1)
```

not:

```text
Always O(1)
```

---

### Mistake 2 — Forgetting `equals()` and `hashCode()`

Custom objects used as HashMap keys should correctly implement both.

---

### Mistake 3 — Using mutable keys

Changing hash-relevant fields after insertion can make lookup unreliable.

---

### Mistake 4 — Using HashMap when HashSet is enough

If you only need:

```text
Does this exist?
```

use:

```java
HashSet
```

---

### Mistake 5 — Storing every prefix sum incorrectly

For subarray-count problems:

```java
map.put(0, 1);
```

is usually essential.

It represents the empty prefix.

---

### Mistake 6 — Overwriting the earliest index

For longest-subarray problems, use:

```java
putIfAbsent()
```

because the earliest occurrence produces the longest range.

---

# 50. Hashing Problem-Solving Checklist

Before coding:

```text
□ Do I need fast lookup?
□ Do I need frequency?
□ Do I need uniqueness?
□ Do I need key → value mapping?
□ Can I trade O(n) space for O(n) time?
□ Is this a prefix-sum problem?
□ Is this a sliding-window problem?
□ Do I need the earliest index?
□ Do I need the latest index?
□ Are keys immutable?
□ Do I need sorted order instead?
```

---

# 51. Complexity Summary

| Problem | Technique | Typical Time | Space |
|---|---|---:|---:|
| Contains Duplicate | HashSet | O(n) | O(n) |
| Two Sum | HashMap | O(n) | O(n) |
| Frequency Count | HashMap | O(n) | O(k) |
| Anagram | HashMap/Array | O(n) | O(k) |
| First Unique | HashMap | O(n) | O(k) |
| Longest Consecutive | HashSet | O(n) avg | O(n) |
| Subarray Sum K | Prefix + HashMap | O(n) | O(n) |
| Longest Subarray K | Prefix + HashMap | O(n) | O(n) |
| Top K Frequent | HashMap + Heap | O(n log k) approx. | O(n) |
| Nearby Duplicate | HashSet + Window | O(n) avg | O(k) |
| Group Anagrams | HashMap | O(n·m log m)* | O(n·m) |

`*` When sorting each word to create the key.

---

# 52. Quick Revision

```text
Hashing
│
├── HashMap
│   ├── Key → Value
│   ├── Frequency
│   ├── Two Sum
│   └── Grouping
│
├── HashSet
│   ├── Uniqueness
│   ├── Membership
│   └── Duplicate Detection
│
├── Prefix Sum + HashMap
│   ├── Subarray Sum K
│   ├── Zero Sum
│   └── Longest Subarray
│
├── Sliding Window + HashSet
│   └── Nearby Duplicate
│
├── HashMap + Heap
│   └── Top K
│
├── HashMap + Two-way Mapping
│   ├── Isomorphic Strings
│   └── Word Pattern
│
└── Java Collections
    ├── HashMap
    ├── HashSet
    ├── LinkedHashMap
    ├── TreeMap
    └── TreeSet
```

---

## Interview Rule

> **When a problem asks for fast lookup, frequency, duplicates, grouping, or “have I seen this before?”, immediately consider hashing. The most powerful combinations to recognize are HashMap + Prefix Sum, HashSet + Sliding Window, and HashMap + Heap.**
