# DSA — Prefix Sum / Difference Array

Prefix Sum and Difference Array are powerful techniques for converting repeated range operations from O(n) per operation into much faster solutions.

Core patterns:

- Prefix Sum
- Prefix Sum on arrays
- Prefix Sum on strings
- Prefix Sum with HashMap
- Subarray Sum
- Range Sum Queries
- 2D Prefix Sum
- Difference Array
- Range Increment
- Range Decrement
- Sweep Line
- Prefix XOR
- Prefix Product
- Running Prefix/Suffix
- Coordinate compression with difference arrays
- Prefix Sum + Binary Search
- Prefix Sum + Sliding Window
- Prefix Sum + HashMap

---

# 1. Prefix Sum

Given:

```text
nums = [2, 4, 1, 5, 3]
```

Build:

```text
prefix = [0, 2, 6, 7, 12, 15]
```

where:

```text
prefix[i]
=
sum of first i elements
```

Therefore:

```text
sum(l...r)
=
prefix[r + 1] - prefix[l]
```

---

# 2. Why Prefix Sum?

Without prefix sums, calculating every range sum can cost:

```text
O(n)
```

With prefix sums:

```text
O(1)
```

per range query after:

```text
O(n)
```

preprocessing.

---

# 3. Prefix Sum — Java

```java
static long[] buildPrefix(
        int[] nums) {

    int n = nums.length;

    long[] prefix =
        new long[n + 1];

    for (int i = 0;
         i < n;
         i++) {

        prefix[i + 1] =
            prefix[i] + nums[i];
    }

    return prefix;
}
```

Use `long` when the total sum may exceed `int`.

---

# 4. Range Sum Query

```java
static long rangeSum(
        long[] prefix,
        int left,
        int right) {

    return prefix[right + 1]
        - prefix[left];
}
```

For:

```text
nums = [2,4,1,5,3]
```

range:

```text
1...3
```

sum:

```text
4 + 1 + 5 = 10
```

Using prefix:

```text
prefix[4] - prefix[1]
= 12 - 2
= 10
```

---

# 5. Prefix Sum with Multiple Queries

If there are `q` range-sum queries:

```text
Build prefix: O(n)
Each query: O(1)
Total: O(n + q)
```

This is much faster than:

```text
O(nq)
```

for large numbers of queries.

---

# 6. Prefix Sum and Subarrays

Prefix sums provide an important identity:

```text
sum(l...r)
=
prefix[r] - prefix[l - 1]
```

Therefore:

```text
sum(l...r) = target
```

can be rewritten as:

```text
prefix[r] - prefix[l - 1] = target
```

which gives:

```text
prefix[l - 1]
=
prefix[r] - target
```

This leads directly to the:

```text
Prefix Sum + HashMap
```

pattern.

---

# 7. Subarray Sum Equals K

Given an array, count the number of subarrays whose sum equals:

```text
k
```

Maintain:

```text
currentPrefix
```

and a map:

```text
prefixSum → frequency
```

For current prefix:

```text
current
```

we need an earlier prefix:

```text
current - k
```

---

# 8. Subarray Sum Equals K — Java

```java
static int subarraySum(
        int[] nums,
        int k) {

    Map<Long, Integer> map =
        new HashMap<>();

    map.put(0L, 1);

    long prefix = 0;
    int count = 0;

    for (int num : nums) {

        prefix += num;

        count +=
            map.getOrDefault(
                prefix - k,
                0
            );

        map.put(
            prefix,
            map.getOrDefault(
                prefix,
                0
            ) + 1
        );
    }

    return count;
}
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

---

# 9. Why Initialize Prefix 0?

This line is essential:

```java
map.put(0L, 1);
```

It represents:

```text
empty prefix
```

If:

```text
prefix == k
```

then:

```text
prefix - k == 0
```

and the subarray from index `0` to the current index is counted.

---

# 10. Longest Subarray with Sum K

Instead of storing the frequency of every prefix sum, store the:

```text
first index
```

where each prefix sum appeared.

For:

```text
prefix[r] - prefix[l - 1] = k
```

we want the earliest possible previous prefix index to maximize the length.

---

# 11. Longest Subarray Sum K — Java

```java
static int longestSubarraySumK(
        int[] nums,
        long k) {

    Map<Long, Integer> first =
        new HashMap<>();

    first.put(0L, -1);

    long prefix = 0;
    int maxLength = 0;

    for (int i = 0;
         i < nums.length;
         i++) {

        prefix += nums[i];

        if (first.containsKey(
                prefix - k)) {

            int previous =
                first.get(prefix - k);

            maxLength =
                Math.max(
                    maxLength,
                    i - previous
                );
        }

        first.putIfAbsent(
            prefix,
            i
        );
    }

    return maxLength;
}
```

---

# 12. Prefix Sum + HashMap Pattern

When you see:

```text
count subarrays
sum equals k
longest subarray
number of subarrays with a property
```

consider:

```text
Prefix Sum
+
HashMap
```

---

# 13. Binary Array — Equal 0s and 1s

Convert:

```text
0 → -1
1 → +1
```

Then:

```text
equal number of 0s and 1s
```

means:

```text
subarray sum = 0
```

Now use prefix sum.

---

# 14. Equal 0s and 1s — Java

```java
static int findMaxLength(
        int[] nums) {

    Map<Integer, Integer> first =
        new HashMap<>();

    first.put(0, -1);

    int prefix = 0;
    int answer = 0;

    for (int i = 0;
         i < nums.length;
         i++) {

        prefix +=
            nums[i] == 0 ? -1 : 1;

        if (first.containsKey(prefix)) {

            answer =
                Math.max(
                    answer,
                    i - first.get(prefix)
                );

        } else {

            first.put(prefix, i);
        }
    }

    return answer;
}
```

---

# 15. Prefix XOR

Prefix techniques are not limited to addition.

Define:

```text
prefixXor[i]
=
xor of elements before i
```

Then:

```text
xor(l...r)
=
prefixXor[r + 1]
^
prefixXor[l]
```

because:

```text
x ^ x = 0
```

---

# 16. Prefix XOR Example

```text
nums = [1, 2, 3]
```

Prefix XOR:

```text
[0, 1, 3, 0]
```

Range:

```text
1...2
```

is:

```text
prefixXor[3]
^
prefixXor[1]

= 0 ^ 1
= 1
```

And:

```text
2 ^ 3 = 1
```

---

# 17. Prefix XOR — Java

```java
static int[] buildPrefixXor(
        int[] nums) {

    int[] prefix =
        new int[nums.length + 1];

    for (int i = 0;
         i < nums.length;
         i++) {

        prefix[i + 1] =
            prefix[i] ^ nums[i];
    }

    return prefix;
}
```

---

# 18. Subarray XOR Equals K

Same idea as subarray sum.

If:

```text
prefixXor[r]
^
prefixXor[l]
=
k
```

then:

```text
prefixXor[l]
=
prefixXor[r] ^ k
```

Use a frequency map.

---

# 19. Subarray XOR Equals K — Java

```java
static int countSubarraysXorK(
        int[] nums,
        int k) {

    Map<Integer, Integer> map =
        new HashMap<>();

    map.put(0, 1);

    int prefix = 0;
    int count = 0;

    for (int num : nums) {

        prefix ^= num;

        count +=
            map.getOrDefault(
                prefix ^ k,
                0
            );

        map.put(
            prefix,
            map.getOrDefault(
                prefix,
                0
            ) + 1
        );
    }

    return count;
}
```

---

# 20. Prefix Sum on Strings

Prefix arrays can store character counts.

For example:

```text
prefix[i][c]
```

can represent how many times character `c` occurs in:

```text
s[0...i-1]
```

This is useful for:

```text
anagram queries
frequency queries
substring character counts
```

---

# 21. Character Frequency Prefix

```java
static int[][] buildCharPrefix(
        String s) {

    int n = s.length();

    int[][] prefix =
        new int[n + 1][26];

    for (int i = 0;
         i < n;
         i++) {

        for (int c = 0;
             c < 26;
             c++) {

            prefix[i + 1][c] =
                prefix[i][c];
        }

        prefix[i + 1]
            [s.charAt(i) - 'a']++;
    }

    return prefix;
}
```

Substring frequency for character `c`:

```text
prefix[right + 1][c]
-
prefix[left][c]
```

---

# 22. Two-Dimensional Prefix Sum

For a matrix:

```text
A × B
```

a 2D prefix sum allows rectangular range queries in:

```text
O(1)
```

after:

```text
O(A × B)
```

preprocessing.

---

# 23. 2D Prefix Sum Definition

Define:

```text
prefix[i][j]
```

as the sum of the rectangle:

```text
rows 0...i-1
columns 0...j-1
```

Then:

```text
prefix[i][j]
=
matrix[i-1][j-1]
+
prefix[i-1][j]
+
prefix[i][j-1]
-
prefix[i-1][j-1]
```

---

# 24. 2D Prefix Sum — Java

```java
static long[][] build2DPrefix(
        int[][] matrix) {

    int rows =
        matrix.length;

    int cols =
        matrix[0].length;

    long[][] prefix =
        new long[rows + 1]
               [cols + 1];

    for (int i = 1;
         i <= rows;
         i++) {

        for (int j = 1;
             j <= cols;
             j++) {

            prefix[i][j] =
                matrix[i - 1][j - 1]
                + prefix[i - 1][j]
                + prefix[i][j - 1]
                - prefix[i - 1][j - 1];
        }
    }

    return prefix;
}
```

---

# 25. Rectangle Sum Query

For rectangle:

```text
(row1, col1)
to
(row2, col2)
```

use:

```text
sum =
prefix[row2 + 1][col2 + 1]
- prefix[row1][col2 + 1]
- prefix[row2 + 1][col1]
+ prefix[row1][col1]
```

---

# 26. 2D Range Query — Java

```java
static long rectangleSum(
        long[][] prefix,
        int row1,
        int col1,
        int row2,
        int col2) {

    return prefix[row2 + 1][col2 + 1]
        - prefix[row1][col2 + 1]
        - prefix[row2 + 1][col1]
        + prefix[row1][col1];
}
```

The final addition removes the double subtraction of the top-left region.

---

# 27. Difference Array

Prefix sum answers:

```text
range queries
```

Difference arrays efficiently perform:

```text
range updates
```

Suppose we want to add:

```text
x
```

to every element in:

```text
[l, r]
```

Instead of updating all elements:

```text
diff[l] += x
diff[r + 1] -= x
```

Then take a prefix sum of `diff`.

---

# 28. Difference Array Example

Initial:

```text
[0,0,0,0,0]
```

Add `5` to:

```text
[1,3]
```

Difference changes:

```text
diff[1] += 5
diff[4] -= 5
```

Then prefix sum produces:

```text
[0,5,5,5,0]
```

---

# 29. Difference Array — Java

```java
static int[] rangeAdd(
        int n,
        int[][] updates) {

    int[] diff =
        new int[n + 1];

    for (int[] update : updates) {

        int left = update[0];
        int right = update[1];
        int value = update[2];

        diff[left] += value;

        if (right + 1 < diff.length) {
            diff[right + 1] -= value;
        }
    }

    int[] result =
        new int[n];

    int current = 0;

    for (int i = 0;
         i < n;
         i++) {

        current += diff[i];
        result[i] = current;
    }

    return result;
}
```

---

# 30. Why Difference Arrays Work

Suppose:

```text
diff[l] += x
```

This starts adding:

```text
x
```

from position `l`.

Then:

```text
diff[r + 1] -= x
```

stops the effect after `r`.

Prefix accumulation converts these boundary changes into the actual range update.

---

# 31. Multiple Range Updates

If there are:

```text
q
```

range additions, direct updates can cost:

```text
O(nq)
```

Difference array reduces the updates to:

```text
O(q)
```

and the final reconstruction costs:

```text
O(n)
```

Total:

```text
O(n + q)
```

---

# 32. Difference Array with Long

When update values can be large:

```java
long[] diff =
    new long[n + 1];
```

is safer.

Always consider:

```text
number of updates
×
maximum update value
```

when selecting the numeric type.

---

# 33. Range Increment and Decrement

Difference arrays work with both:

```text
add x
```

and:

```text
subtract x
```

For subtraction:

```java
diff[left] -= x;
diff[right + 1] += x;
```

---

# 34. Range Assignment Is Different

Difference arrays naturally support:

```text
range addition
```

but not arbitrary:

```text
range assignment
```

because assignment overwrites previous values.

For complex range assignments, consider:

```text
Segment Tree
Lazy Propagation
```

or another appropriate data structure.

---

# 35. Range Update + Final Array

Typical pattern:

```text
Initial array
+
many range updates
=
final array
```

Use:

```text
difference array
```

when updates are additive and the final result is needed.

---

# 36. Car Pooling

Given trips:

```text
[number of passengers, start, end]
```

determine whether vehicle capacity is ever exceeded.

Use a difference array:

```text
diff[start] += passengers
diff[end] -= passengers
```

Then scan locations.

---

# 37. Car Pooling — Java

```java
static boolean carPooling(
        int[][] trips,
        int capacity) {

    int[] diff =
        new int[1001];

    for (int[] trip : trips) {

        int passengers = trip[0];
        int start = trip[1];
        int end = trip[2];

        diff[start] += passengers;
        diff[end] -= passengers;
    }

    int current = 0;

    for (int value : diff) {

        current += value;

        if (current > capacity) {
            return false;
        }
    }

    return true;
}
```

This is also a:

```text
Sweep Line
```

technique.

---

# 38. Sweep Line

Sweep Line processes events in sorted order.

For intervals:

```text
[start, end)
```

create events:

```text
start → +value
end   → -value
```

Then scan from left to right.

This is conceptually similar to a difference array.

---

# 39. Difference Array vs Sweep Line

### Difference Array

Best when:

```text
coordinates are small and bounded.
```

You can directly index:

```text
diff[position]
```

### Sweep Line

Best when:

```text
coordinates are huge
or sparse.
```

Store events and sort them.

---

# 40. Coordinate Compression

Suppose positions are:

```text
1
1,000,000,000
2,000,000,000
```

A difference array indexed by the actual coordinate is impossible.

Coordinate compression maps important coordinates to smaller indexes.

Example:

```text
1 → 0
1,000,000,000 → 1
2,000,000,000 → 2
```

Then process the compressed coordinates.

---

# 41. Prefix Sum + Binary Search

Suppose prefix sums are:

```text
[0, 3, 7, 10, 15, 20]
```

and we need the first position where the prefix reaches:

```text
12
```

If all values are non-negative, prefix sums are monotonic.

Therefore:

```text
Prefix Sum
+
Binary Search
```

can answer such queries efficiently.

---

# 42. Prefix Sum + Sliding Window

When all numbers are non-negative, prefix sums are often monotonic.

This can allow:

```text
binary search
```

or:

```text
two pointers
```

for subarray conditions.

For example:

```text
minimum length subarray
with sum >= target
```

can be solved using a sliding window.

---

# 43. Minimum Size Subarray Sum

Given positive integers, find the minimum length subarray with sum at least:

```text
target
```

Two-pointer solution:

```java
static int minSubArrayLen(
        int target,
        int[] nums) {

    int left = 0;
    long sum = 0;

    int answer =
        Integer.MAX_VALUE;

    for (int right = 0;
         right < nums.length;
         right++) {

        sum += nums[right];

        while (sum >= target) {

            answer =
                Math.min(
                    answer,
                    right - left + 1
                );

            sum -= nums[left++];
        }
    }

    return answer ==
            Integer.MAX_VALUE
        ? 0
        : answer;
}
```

This is often preferable to explicit prefix sums.

---

# 44. Prefix Sum with Negative Numbers

Important:

If the array contains negative values:

```text
prefix sums are not necessarily monotonic.
```

Therefore:

```text
binary search on prefix sums
```

may not work.

But:

```text
prefix sum + HashMap
```

still works for many exact-sum problems.

---

# 45. Prefix Sum for Range Average

If you need:

```text
average(l, r)
```

use:

```text
sum(l, r)
/
(r - l + 1)
```

with prefix sums.

For floating-point answers:

```java
double average =
    (double) sum
    / (right - left + 1);
```

---

# 46. Prefix Sum for Frequency

For a small fixed alphabet or value range, maintain:

```text
prefix[value][index]
```

This allows queries like:

```text
How many values equal x
between l and r?
```

in:

```text
O(1)
```

per query.

---

# 47. Prefix Sum on 3 Dimensions

The same idea extends to:

```text
3D prefix sums
```

for volumetric data.

State:

```text
prefix[x][y][z]
```

The formula uses inclusion-exclusion.

This is less common in interviews but follows the same concept.

---

# 48. Prefix Product

For multiplication problems, a prefix product can be useful.

Example:

```text
prefixProduct[i]
=
product of elements before i
```

But unlike addition:

```text
zero
```

complicates the calculation.

For product problems, always handle zero carefully.

---

# 49. Product Except Self

A classic example uses:

```text
prefix product
+
suffix product
```

without division.

```java
static int[] productExceptSelf(
        int[] nums) {

    int n = nums.length;

    int[] result =
        new int[n];

    result[0] = 1;

    for (int i = 1;
         i < n;
         i++) {

        result[i] =
            result[i - 1]
            * nums[i - 1];
    }

    int suffix = 1;

    for (int i = n - 1;
         i >= 0;
         i--) {

        result[i] *= suffix;
        suffix *= nums[i];
    }

    return result;
}
```

Complexity:

```text
O(n)
```

extra space:

```text
O(1)
```

excluding the output array.

---

# 50. Prefix/Suffix Technique

Many problems can be solved by combining:

```text
prefix information
+
suffix information
```

Examples:

```text
Product Except Self
Trapping Rain Water
Maximum Subarray variants
Left/right maxima
Array partition problems
```

---

# 51. Prefix Minimum / Maximum

Sometimes the prefix operation is:

```text
minimum
```

or:

```text
maximum
```

Example:

```java
prefixMax[i] =
Math.max(
    prefixMax[i - 1],
    nums[i]
);
```

This helps answer:

```text
maximum value from 0 to i
```

in O(1).

---

# 52. Difference Array for 2D

Difference arrays also extend to matrices.

For rectangle update:

```text
add x to rectangle
(r1,c1) ... (r2,c2)
```

update four corners:

```text
diff[r1][c1] += x
diff[r1][c2 + 1] -= x
diff[r2 + 1][c1] -= x
diff[r2 + 1][c2 + 1] += x
```

Then reconstruct using a 2D prefix sum.

---

# 53. 2D Difference Array Formula

For:

```text
rectangle:
(r1,c1)
to
(r2,c2)
```

apply:

```text
diff[r1][c1] += x

diff[r1][c2 + 1] -= x

diff[r2 + 1][c1] -= x

diff[r2 + 1][c2 + 1] += x
```

Then perform 2D prefix accumulation.

---

# 54. Prefix Sum + Modulo

For problems involving:

```text
subarray sum divisible by k
```

store:

```text
prefix % k
```

in a frequency map.

If two prefix sums have the same remainder:

```text
(prefix[j] - prefix[i]) % k = 0
```

therefore the subarray is divisible by `k`.

---

# 55. Subarrays Divisible by K — Java

```java
static int subarraysDivByK(
        int[] nums,
        int k) {

    int[] frequency =
        new int[k];

    frequency[0] = 1;

    int prefix = 0;
    int answer = 0;

    for (int num : nums) {

        prefix =
            ((prefix + num) % k + k)
            % k;

        answer +=
            frequency[prefix];

        frequency[prefix]++;
    }

    return answer;
}
```

The double modulo handles negative values correctly.

---

# 56. Continuous Subarray Sum

For a subarray whose sum is a multiple of `k`, track:

```text
prefix % k
```

Store the earliest index where each remainder occurred.

If the same remainder appears again far enough apart, the intervening subarray is divisible by `k`.

---

# 57. Prefix Sum + HashMap Decision Rule

Use a frequency map when the question asks:

```text
How many?
```

Use the first-index map when the question asks:

```text
Longest?
```

This is a very useful distinction.

---

# 58. Prefix Sum + HashMap Template

### Count

```java
Map<Long, Integer> map =
    new HashMap<>();

map.put(0L, 1);

long prefix = 0;

for (int x : nums) {

    prefix += x;

    answer +=
        map.getOrDefault(
            prefix - k,
            0
        );

    map.put(
        prefix,
        map.getOrDefault(
            prefix,
            0
        ) + 1
    );
}
```

### Longest

```java
Map<Long, Integer> first =
    new HashMap<>();

first.put(0L, -1);

long prefix = 0;

for (int i = 0;
     i < nums.length;
     i++) {

    prefix += nums[i];

    if (first.containsKey(
            prefix - k)) {

        answer =
            Math.max(
                answer,
                i - first.get(
                    prefix - k
                )
            );
    }

    first.putIfAbsent(
        prefix,
        i
    );
}
```

---

# 59. Common Prefix Sum Mistakes

### Mistake 1 — Off-by-one

Use:

```text
prefix[n + 1]
```

when convenient.

### Mistake 2 — Forgetting empty prefix

Initialize:

```text
prefix 0 → index -1
```

or:

```text
frequency[0] = 1
```

### Mistake 3 — Using int for large sums

Use:

```text
long
```

when required.

### Mistake 4 — Assuming prefix sums are monotonic

Negative numbers break monotonicity.

---

# 60. Common Difference Array Mistakes

### Mistake 1

Forgetting:

```text
diff[r + 1] -= value
```

### Mistake 2

Updating every element directly.

### Mistake 3

Forgetting the final prefix reconstruction.

### Mistake 4

Using a coordinate array that is too large.

Use:

```text
coordinate compression
```

or:

```text
sweep line
```

for huge coordinates.

---

# 61. Prefix Sum vs Difference Array

| Technique | Best For |
|---|---|
| Prefix Sum | Range queries |
| Difference Array | Range updates |
| Prefix XOR | Range XOR queries |
| Prefix + HashMap | Subarray conditions |
| 2D Prefix Sum | Rectangle queries |
| 2D Difference | Rectangle updates |
| Sweep Line | Sparse interval events |

---

# 62. Prefix Sum Complexity

For:

```text
n elements
q queries
```

normal range sum:

```text
O(nq)
```

Prefix sum:

```text
O(n + q)
```

if all queries are range sums.

---

# 63. Difference Array Complexity

For:

```text
n elements
q range updates
```

naive:

```text
O(nq)
```

difference array:

```text
O(n + q)
```

when coordinates are directly indexable.

---

# 64. Interview Questions — Easy

1. Range Sum Query.
2. Running Sum.
3. Find Pivot Index.
4. Left/Right Sum Difference.
5. Product Except Self.
6. Prefix XOR.

---

# 65. Interview Questions — Medium

7. Subarray Sum Equals K.
8. Longest Subarray with Sum K.
9. Subarray Sums Divisible by K.
10. Binary Array Equal 0s and 1s.
11. Car Pooling.
12. Range Addition.
13. 2D Range Sum Query.
14. Minimum Size Subarray Sum.

---

# 66. Interview Questions — Advanced

15. 2D Difference Array.
16. Coordinate Compression + Difference Array.
17. Prefix Sum + Binary Search.
18. Prefix Sum + Monotonic Structure.
19. Complex subarray counting.
20. Prefix XOR counting.
21. Multi-dimensional prefix techniques.

---

# 67. Pattern Recognition

When you see:

```text
range sum
```

think:

```text
Prefix Sum
```

When you see:

```text
many range additions
```

think:

```text
Difference Array
```

When you see:

```text
subarray sum equals K
```

think:

```text
Prefix Sum + HashMap
```

When you see:

```text
rectangle sum
```

think:

```text
2D Prefix Sum
```

When you see:

```text
rectangle updates
```

think:

```text
2D Difference Array
```

When you see:

```text
sum divisible by K
```

think:

```text
Prefix Sum + Modulo
```

---

# 68. Decision Tree

```text
Range problem
     |
     +--------------------+
     |                    |
 Query                  Update
     |                    |
     ↓                    ↓
Prefix Sum          Difference Array
     |
     +----------------------+
     |                      |
Exact subarray         2D rectangle
condition                  |
     |                      |
Prefix + HashMap       2D Prefix Sum
     |
     +----------------------+
     |
Modulo condition
     |
Prefix % K + HashMap
```

---

# 69. Quick Revision

```text
Prefix Sum / Difference Array
│
├── Prefix Sum
│   ├── Range Sum
│   ├── Subarray Sum
│   ├── HashMap
│   └── Modulo
│
├── Prefix XOR
│   └── XOR Range Queries
│
├── 2D Prefix Sum
│   └── Rectangle Queries
│
├── Difference Array
│   ├── Range Addition
│   ├── Range Updates
│   └── Sweep Line
│
├── 2D Difference Array
│   └── Rectangle Updates
│
└── Prefix/Suffix
    ├── Product Except Self
    ├── Prefix Max/Min
    └── Combined Array Problems
```

---

# 70. Most Important Formulas

### 1D Range Sum

```text
sum(l,r)
=
prefix[r+1] - prefix[l]
```

### Subarray Sum K

```text
currentPrefix - oldPrefix = K
```

therefore:

```text
oldPrefix =
currentPrefix - K
```

### Range Update

```text
diff[l] += x
diff[r+1] -= x
```

### 2D Prefix

```text
prefix[i][j]
=
matrix[i-1][j-1]
+
prefix[i-1][j]
+
prefix[i][j-1]
-
prefix[i-1][j-1]
```

### 2D Difference

```text
diff[r1][c1] += x
diff[r1][c2+1] -= x
diff[r2+1][c1] -= x
diff[r2+1][c2+1] += x
```

---

# 71. Interview Explanation Template

For a prefix-sum problem:

```text
I can preprocess prefix sums so that each range sum
can be calculated in O(1).

For a subarray condition, I use the relationship between
two prefix sums and store previous prefix values in a HashMap.

This reduces the repeated range calculation from O(n)
to O(1), giving an overall O(n) solution.
```

For a difference-array problem:

```text
Instead of updating every element in each range,
I record only the boundaries of each update.

I add the value at the left boundary and subtract it
after the right boundary.

Finally, I take a prefix sum to reconstruct the final array.

This reduces the total complexity to O(n + q).
```

---

# 72. Final Interview Rule

> **Prefix Sum is for efficiently answering range information. Difference Array is for efficiently applying range additions. When you see repeated subarray or range operations, look for a way to move the work to the boundaries or to prefix relationships.**
