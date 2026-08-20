# DSA — Prefix Sum

Prefix Sum is one of the most useful techniques for solving array and subarray problems efficiently.

The main idea is simple:

> Store cumulative information so that a range can be calculated without repeatedly traversing it.

Prefix sums are especially important for:

- Range sum queries
- Subarray sums
- Zero-sum subarrays
- Subarray Sum Equals K
- Longest subarray with a target sum
- 2D matrix range queries
- Combining with HashMap
- Combining with binary search
- Combining with difference arrays

---

# 1. What is Prefix Sum?

Given:

```text
arr = [2, 4, 1, 3]
```

The prefix sums are:

```text
[2, 6, 7, 10]
```

Because:

```text
2
2 + 4 = 6
2 + 4 + 1 = 7
2 + 4 + 1 + 3 = 10
```

---

# 2. Basic Prefix Sum

```java
int[] prefix =
    new int[arr.length];

prefix[0] = arr[0];

for (int i = 1;
     i < arr.length;
     i++) {

    prefix[i] =
        prefix[i - 1] + arr[i];
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 3. Prefix Sum with an Extra Zero

A cleaner implementation uses an array of size `n + 1`.

```java
int[] prefix =
    new int[arr.length + 1];

for (int i = 0;
     i < arr.length;
     i++) {

    prefix[i + 1] =
        prefix[i] + arr[i];
}
```

For:

```text
arr = [2, 4, 1, 3]
```

we get:

```text
prefix = [0, 2, 6, 7, 10]
```

This version makes range calculations easier.

---

# 4. Range Sum Query

Suppose:

```text
arr = [2, 4, 1, 3, 5]
```

Find the sum from index:

```text
left = 1
right = 3
```

The answer is:

```text
4 + 1 + 3 = 8
```

Using the extra-zero prefix array:

```java
int sum =
    prefix[right + 1] - prefix[left];
```

So:

```java
int sum =
    prefix[4] - prefix[1];
```

```text
10 - 2 = 8
```

---

# 5. Why Prefix Sum Is Useful

Without prefix sum:

```text
Every query → O(n)
```

With prefix sum:

```text
Build → O(n)

Each range query → O(1)
```

So for many range queries:

```text
O(nq)
```

can become approximately:

```text
O(n + q)
```

where:

```text
n = array size
q = number of queries
```

---

# 6. Prefix Sum Formula

Using the extra-zero array:

```text
prefix[i + 1]
=
prefix[i] + arr[i]
```

Range:

```text
sum(left, right)
=
prefix[right + 1] - prefix[left]
```

Memorize this formula.

---

# 7. Subarray Sum

A subarray is contiguous.

Example:

```text
[2, 4, 1, 3]
```

Possible subarrays:

```text
[2]
[4]
[1]
[3]
[2,4]
[4,1]
[1,3]
[2,4,1]
...
```

Prefix sums let us calculate any subarray sum efficiently.

---

# 8. Prefix Sum Identity

If:

```text
prefix[j]
```

is the sum from the beginning through `j`, then:

```text
sum(i ... j)
=
prefix[j] - prefix[i - 1]
```

Using the extra-zero version:

```text
sum(i ... j)
=
prefix[j + 1] - prefix[i]
```

This identity is the foundation of many subarray problems.

---

# 9. Count Subarrays with Sum K

This is one of the most important prefix-sum interview problems.

Example:

```text
nums = [1, 1, 1]
k = 2
```

Answer:

```text
2
```

Use:

```text
Prefix Sum + HashMap
```

---

# 10. Subarray Sum Equals K — Java

```java
static int subarraySum(
        int[] nums,
        int k) {

    Map<Integer, Integer> frequency =
        new HashMap<>();

    frequency.put(0, 1);

    int prefix = 0;
    int count = 0;

    for (int value : nums) {

        prefix += value;

        int required =
            prefix - k;

        count +=
            frequency.getOrDefault(
                required,
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

    return count;
}
```

### Complexity

```text
Time:  O(n) average
Space: O(n)
```

---

# 11. Why Prefix Sum + HashMap Works

Suppose the current prefix sum is:

```text
current
```

We want a previous prefix:

```text
previous
```

such that:

```text
current - previous = k
```

Therefore:

```text
previous = current - k
```

So we ask the HashMap:

```java
frequency.get(current - k)
```

This tells us how many previous prefixes can form a subarray with sum `k`.

---

# 12. Why `frequency.put(0, 1)` Is Important

Consider:

```text
nums = [5]
k = 5
```

The first prefix is:

```text
5
```

We need:

```text
5 - 5 = 0
```

The prefix sum `0` represents the empty prefix before the array starts.

Therefore:

```java
frequency.put(0, 1);
```

is essential.

---

# 13. Longest Subarray with Sum K

For arbitrary integers, store the earliest index of each prefix sum.

```java
static int longestSubarray(
        int[] nums,
        int k) {

    Map<Integer, Integer> firstIndex =
        new HashMap<>();

    firstIndex.put(0, -1);

    int prefix = 0;
    int maximum = 0;

    for (int i = 0;
         i < nums.length;
         i++) {

        prefix += nums[i];

        int required =
            prefix - k;

        if (firstIndex.containsKey(required)) {

            maximum = Math.max(
                maximum,
                i - firstIndex.get(required)
            );
        }

        firstIndex.putIfAbsent(
            prefix,
            i
        );
    }

    return maximum;
}
```

### Why `putIfAbsent()`?

We want the earliest occurrence.

Earlier index:

```text
→ longer possible subarray
```

---

# 14. Zero Sum Subarray

A subarray has sum zero when:

```text
prefix[j] == prefix[i]
```

for two different prefix positions.

Example:

```text
[1, -1]
```

Prefix:

```text
1
0
```

The prefix `0` returns to the initial prefix.

---

# 15. Check if a Zero-Sum Subarray Exists

```java
static boolean hasZeroSumSubarray(
        int[] nums) {

    Set<Integer> seen =
        new HashSet<>();

    int prefix = 0;

    seen.add(0);

    for (int value : nums) {

        prefix += value;

        if (!seen.add(prefix)) {
            return true;
        }
    }

    return false;
}
```

### Complexity

```text
Time:  O(n) average
Space: O(n)
```

---

# 16. Count Zero-Sum Subarrays

Use frequency rather than just a Set.

```java
static int countZeroSumSubarrays(
        int[] nums) {

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

    return count;
}
```

---

# 17. Maximum Length Zero-Sum Subarray

Store the earliest index for each prefix sum.

```java
static int maxZeroSumLength(
        int[] nums) {

    Map<Integer, Integer> firstIndex =
        new HashMap<>();

    firstIndex.put(0, -1);

    int prefix = 0;
    int maximum = 0;

    for (int i = 0;
         i < nums.length;
         i++) {

        prefix += nums[i];

        if (firstIndex.containsKey(prefix)) {

            maximum = Math.max(
                maximum,
                i - firstIndex.get(prefix)
            );

        } else {

            firstIndex.put(
                prefix,
                i
            );
        }
    }

    return maximum;
}
```

---

# 18. Equal Number of 0s and 1s

A useful transformation is:

```text
0 → -1
1 → +1
```

Then:

```text
equal number of 0s and 1s
```

becomes:

```text
subarray sum = 0
```

Example:

```text
[0, 1, 0, 1]
```

Transform:

```text
[-1, 1, -1, 1]
```

Now use the longest zero-sum subarray technique.

---

# 19. Count Subarrays with Equal 0s and 1s

Transform:

```text
0 → -1
1 → +1
```

Then count zero-sum subarrays using:

```text
Prefix Sum + HashMap
```

This is a very common interview transformation.

---

# 20. Equal 0s, 1s and 2s

For more advanced problems, track differences between counts.

For example:

```text
count0 - count1
count1 - count2
```

Store the pair of differences as the HashMap key.

If the same pair occurs twice, the elements between those positions have equal counts of the relevant values.

This extends the prefix-state idea.

---

# 21. Running Sum

Prefix sum can also be used to maintain a running cumulative value.

```java
int sum = 0;

for (int value : nums) {

    sum += value;

    System.out.println(sum);
}
```

If you only need the running total once, an extra prefix array is unnecessary.

---

# 22. Prefix Maximum / Minimum

Prefix techniques are not limited to sums.

You can maintain:

```text
prefix maximum
prefix minimum
prefix XOR
prefix product
```

Example:

```java
int maximum = Integer.MIN_VALUE;

for (int value : nums) {

    maximum =
        Math.max(maximum, value);

    // maximum is the largest
    // value seen so far.
}
```

---

# 23. Prefix XOR

XOR has a property similar to prefix sums:

```text
a ^ a = 0
a ^ 0 = a
```

Build:

```java
int[] prefixXor =
    new int[nums.length + 1];

for (int i = 0;
     i < nums.length;
     i++) {

    prefixXor[i + 1] =
        prefixXor[i] ^ nums[i];
}
```

XOR of range `[left, right]`:

```java
int result =
    prefixXor[right + 1]
    ^ prefixXor[left];
```

---

# 24. Prefix Sum for 2D Matrix

Prefix sums can also work on matrices.

For:

```text
matrix
```

build:

```text
prefix[row][col]
```

so rectangular region sums can be calculated quickly.

---

# 25. 2D Prefix Sum Formula

For a matrix prefix:

```text
P[i][j]
```

represents the sum of the rectangle from the origin to `(i, j)`.

The inclusion-exclusion formula is:

```text
P[i][j]
=
matrix[i][j]
+ P[i-1][j]
+ P[i][j-1]
- P[i-1][j-1]
```

The subtraction prevents the top-left overlapping region from being counted twice.

---

# 26. 2D Prefix Sum with Padding

A clean implementation uses:

```java
int[][] prefix =
    new int[rows + 1][cols + 1];

for (int r = 0;
     r < rows;
     r++) {

    for (int c = 0;
         c < cols;
         c++) {

        prefix[r + 1][c + 1] =
            matrix[r][c]
            + prefix[r][c + 1]
            + prefix[r + 1][c]
            - prefix[r][c];
    }
}
```

The extra row and column simplify boundary conditions.

---

# 27. 2D Range Sum Query

For rectangle:

```text
(row1, col1)
to
(row2, col2)
```

the sum is:

```java
int sum =
    prefix[row2 + 1][col2 + 1]
    - prefix[row1][col2 + 1]
    - prefix[row2 + 1][col1]
    + prefix[row1][col1];
```

This is classic inclusion-exclusion.

---

# 28. Difference Array

A difference array is closely related to prefix sums.

Instead of storing cumulative values, store changes between consecutive positions.

Given:

```text
arr = [2, 5, 7, 7]
```

difference:

```text
[2, 3, 2, 0]
```

because:

```text
2
5 - 2 = 3
7 - 5 = 2
7 - 7 = 0
```

---

# 29. Why Use a Difference Array?

It is useful when you have many range updates.

Example:

```text
Add +5 to every element from index 2 to 6.
```

Instead of updating every element:

```text
O(n)
```

mark the boundaries.

```java
diff[left] += value;

if (right + 1 < diff.length) {
    diff[right + 1] -= value;
}
```

Then take a prefix sum to reconstruct the final array.

---

# 30. Range Update Example

Suppose:

```text
arr = [0, 0, 0, 0, 0]
```

Add `5` to indexes:

```text
1 through 3
```

Difference update:

```java
diff[1] += 5;
diff[4] -= 5;
```

Then prefix sum gives:

```text
[0, 5, 5, 5, 0]
```

---

# 31. Multiple Range Updates

If there are many range updates:

```text
q operations
```

naively:

```text
O(nq)
```

Difference array:

```text
O(q + n)
```

This is a major optimization.

---

# 32. Prefix Sum + Binary Search

Suppose all numbers are positive.

Prefix sums are strictly increasing:

```text
[2, 5, 9, 14, 20]
```

This allows binary search on the prefix array.

For example, find the first prefix sum greater than or equal to a target.

Use:

```java
Arrays.binarySearch(...)
```

or implement lower bound manually.

---

# 33. Prefix Sum + Sliding Window

These techniques can sometimes solve related problems, but they have different assumptions.

### Prefix Sum

Works with:

```text
positive
negative
zero
```

for exact sum calculations.

### Sliding Window

Often requires:

```text
positive/non-negative values
```

for monotonic sum conditions.

Do not confuse the two.

---

# 34. Prefix Sum + HashMap Pattern

Memorize this:

```java
Map<Integer, Integer> map =
    new HashMap<>();

map.put(0, 1);

int prefix = 0;

for (int value : nums) {

    prefix += value;

    int required =
        prefix - k;

    // Use map.get(required)

    map.put(
        prefix,
        map.getOrDefault(
            prefix,
            0
        ) + 1
    );
}
```

This pattern solves many subarray-count problems.

---

# 35. Prefix Sum + Earliest Index Pattern

For longest subarray:

```java
Map<Integer, Integer> firstIndex =
    new HashMap<>();

firstIndex.put(0, -1);

int prefix = 0;

for (int i = 0;
     i < nums.length;
     i++) {

    prefix += nums[i];

    int required =
        prefix - k;

    if (firstIndex.containsKey(required)) {

        // Calculate length
    }

    firstIndex.putIfAbsent(
        prefix,
        i
    );
}
```

The key difference is:

```text
Count problem → frequency
Longest problem → earliest index
```

---

# 36. Prefix Sum + HashSet Pattern

For existence:

```java
Set<Integer> seen =
    new HashSet<>();

seen.add(0);

int prefix = 0;

for (int value : nums) {

    prefix += value;

    if (!seen.add(prefix)) {
        return true;
    }
}
```

Useful for:

```text
Zero-sum subarray exists?
Repeated prefix state?
```

---

# 37. Important Transformations

Many prefix-sum problems become easier after transforming the input.

Examples:

```text
0/1 balance
0 → -1
1 → +1
```

Then:

```text
Equal 0s and 1s
→
Zero-sum subarray
```

Another example:

```text
Even/odd
even → 0
odd → 1
```

Then count odd numbers using prefix sums.

---

# 38. Prefix Sum for Even/Odd Problems

Suppose:

```text
nums = [2, 4, 3, 7]
```

Convert:

```text
even → 0
odd → 1
```

Result:

```text
[0, 0, 1, 1]
```

Prefix sum tells us:

```text
number of odd values
```

in any range.

---

# 39. Prefix Sum for Frequency Queries

If values are small and bounded, create separate prefix counts.

For example:

```text
prefix[i]
=
number of even numbers
from 0 to i - 1
```

Then a range query can be answered in:

```text
O(1)
```

after:

```text
O(n)
```

preprocessing.

---

# 40. Prefix Sum vs Brute Force

Suppose there are:

```text
n = 100,000
```

and:

```text
q = 100,000
```

range queries.

Brute force can approach:

```text
O(nq)
```

which is too slow.

Prefix preprocessing:

```text
O(n)
```

and each query:

```text
O(1)
```

gives:

```text
O(n + q)
```

This is why prefix sums are so powerful.

---

# 41. Overflow Considerations

If values can be large, an `int` prefix sum may overflow.

Use:

```java
long prefix = 0;
```

and:

```java
Map<Long, Integer> frequency =
    new HashMap<>();
```

when constraints require it.

Example:

```java
long[] prefix =
    new long[nums.length + 1];
```

Always check the numeric constraints.

---

# 42. Common Prefix Sum Mistakes

### Mistake 1 — Wrong range boundaries

For padded prefix:

```text
sum(left, right)
=
prefix[right + 1] - prefix[left]
```

---

### Mistake 2 — Forgetting the initial zero

For subarray-count problems:

```java
map.put(0, 1);
```

is usually required.

---

### Mistake 3 — Using the latest index for longest subarray

For longest length, keep:

```java
putIfAbsent()
```

not:

```java
put()
```

---

### Mistake 4 — Using sliding window with negative values

Exact-sum problems with negative numbers often need prefix sums instead.

---

### Mistake 5 — Integer overflow

Use `long` when required.

---

### Mistake 6 — Confusing subarray and subsequence

Prefix sum applies naturally to contiguous ranges.

---

# 43. Prefix Sum Problem Recognition

When you see:

```text
Range sum
```

think:

```text
Prefix Sum
```

When you see:

```text
Many range queries
```

think:

```text
Prefix Sum
```

When you see:

```text
Subarray Sum = K
```

think:

```text
Prefix Sum + HashMap
```

When you see:

```text
Longest Subarray Sum = K
```

think:

```text
Prefix Sum + Earliest Index
```

When you see:

```text
Zero Sum
```

think:

```text
Repeated Prefix Sum
```

When you see:

```text
Many range updates
```

think:

```text
Difference Array
```

When you see:

```text
2D rectangle sum
```

think:

```text
2D Prefix Sum
```

---

# 44. Interview Questions — Easy

1. Build a prefix sum array.
2. Find the sum of a range.
3. Answer multiple range sum queries.
4. Find the running sum.
5. Find prefix maximum.
6. Find prefix minimum.
7. Find range XOR using prefix XOR.
8. Find the sum of every fixed range.

---

# 45. Interview Questions — Medium

9. Subarray Sum Equals K.
10. Check for zero-sum subarray.
11. Count zero-sum subarrays.
12. Longest zero-sum subarray.
13. Longest subarray with sum K.
14. Equal number of 0s and 1s.
15. Count subarrays with equal 0s and 1s.
16. Range update using difference array.
17. 2D range sum query.
18. Count subarrays with a given property using prefix state.
19. Find pivot index.
20. Find equilibrium index.

---

# 46. Interview Questions — Advanced

21. Count subarrays divisible by K.
22. Longest subarray divisible by K.
23. Continuous subarray sum.
24. Number of submatrices with target sum.
25. 2D prefix sum problems.
26. Range updates using difference arrays.
27. Difference array + prefix reconstruction.
28. Prefix sum + binary search.
29. Prefix state compression problems.
30. Equal frequency/count prefix-state problems.

---

# 47. Subarray Divisible by K

A powerful variation uses:

```text
prefixSum % k
```

If two prefix sums have the same remainder:

```text
prefix[i] % k
==
prefix[j] % k
```

then the subarray between them is divisible by `k`.

Important with negative numbers:

```java
int remainder =
    ((prefix % k) + k) % k;
```

This normalizes the remainder to a non-negative value.

---

# 48. Count Subarrays Divisible by K

Concept:

```java
Map<Integer, Integer> frequency =
    new HashMap<>();

frequency.put(0, 1);

int prefix = 0;
int count = 0;

for (int value : nums) {

    prefix += value;

    int remainder =
        ((prefix % k) + k) % k;

    count +=
        frequency.getOrDefault(
            remainder,
            0
        );

    frequency.put(
        remainder,
        frequency.getOrDefault(
            remainder,
            0
        ) + 1
    );
}
```

### Complexity

```text
Time:  O(n) average
Space: O(k)
```

---

# 49. Pivot Index

The pivot index is an index where:

```text
sum of left elements
=
sum of right elements
```

One approach:

```java
int total = 0;

for (int value : nums) {
    total += value;
}

int leftSum = 0;

for (int i = 0;
     i < nums.length;
     i++) {

    int rightSum =
        total - leftSum - nums[i];

    if (leftSum == rightSum) {
        return i;
    }

    leftSum += nums[i];
}

return -1;
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 50. Equilibrium Index

Same fundamental idea as pivot index.

At each index:

```text
leftSum
=
totalSum - leftSum - current
```

If equal:

```text
equilibrium found
```

---

# 51. Prefix Sum in 2D Problems

For matrix problems, always consider:

```text
2D Prefix Sum
```

when the question asks for:

```text
Rectangle sum
Multiple submatrix queries
Sum of a region
```

Build once:

```text
O(rows × cols)
```

Then each rectangle query can be:

```text
O(1)
```

---

# 52. Difference Array vs Prefix Sum

### Prefix Sum

Use when:

```text
You need range queries.
```

Example:

```text
What is the sum from L to R?
```

### Difference Array

Use when:

```text
You need many range updates.
```

Example:

```text
Add X to every element from L to R.
```

Often the workflow is:

```text
Difference Array
        ↓
Apply range updates
        ↓
Prefix Sum
        ↓
Final Array
```

---

# 53. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Build Prefix Sum | Prefix | O(n) | O(n) |
| Range Sum | Prefix | O(1) query | O(n) |
| Many Range Queries | Prefix | O(n + q) | O(n) |
| Subarray Sum K | Prefix + HashMap | O(n) avg | O(n) |
| Longest Subarray K | Prefix + Map | O(n) avg | O(n) |
| Zero Sum Exists | Prefix + Set | O(n) avg | O(n) |
| Count Zero Sum | Prefix + Map | O(n) avg | O(n) |
| Divisible by K | Prefix Mod + Map | O(n) avg | O(k) |
| 2D Range Sum | 2D Prefix | O(1) query | O(rc) |
| Range Updates | Difference Array | O(n + q) | O(n) |

---

# 54. Quick Revision

```text
Prefix Sum
│
├── 1D Prefix
│   ├── Range Sum
│   ├── Multiple Queries
│   └── Running Sum
│
├── Prefix + HashMap
│   ├── Subarray Sum K
│   ├── Zero Sum
│   ├── Longest Subarray K
│   └── Equal 0s and 1s
│
├── Prefix + HashSet
│   └── Zero Sum Exists
│
├── Prefix Modulo
│   └── Subarrays Divisible by K
│
├── Prefix XOR
│   └── Range XOR
│
├── 2D Prefix
│   └── Rectangle Queries
│
└── Difference Array
    ├── Range Updates
    └── Prefix Reconstruction
```

---

## Interview Rule

> **When you see a range-query or subarray-sum problem, think Prefix Sum. When you see `Subarray Sum = K`, immediately think Prefix Sum + HashMap. When you see many range updates, think Difference Array.**
