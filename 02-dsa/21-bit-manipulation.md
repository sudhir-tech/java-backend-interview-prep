# DSA — Bit Manipulation

Bit manipulation means working directly with the binary representation of integers.

It is useful for:

- Competitive programming
- DSA interviews
- XOR problems
- Subsets
- Bitmasking
- Counting set bits
- Power-of-two checks
- State compression
- Binary Trie
- Low-level optimization

Java provides bitwise operators that work directly on integer bits.

---

# 1. Binary Representation

An integer is represented using bits:

```text
Decimal: 13

Binary:
1101
```

Because:

```text
1101
=
8 + 4 + 0 + 1
=
13
```

For an integer, each position represents a power of two.

```text
Bit position:

3  2  1  0
↓  ↓  ↓  ↓
8  4  2  1
```

---

# 2. Bitwise Operators

Java provides:

```text
&   AND
|   OR
^   XOR
~   NOT
<<  Left Shift
>>  Signed Right Shift
>>> Unsigned Right Shift
```

These are different from logical operators:

```text
&&
||
!
```

---

# 3. AND `&`

Truth table:

```text
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1
```

Example:

```text
  1101
& 1011
------
  1001
```

Result:

```text
9
```

---

# 4. OR `|`

Truth table:

```text
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1
```

Example:

```text
  1101
| 1011
------
  1111
```

Result:

```text
15
```

---

# 5. XOR `^`

Truth table:

```text
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0
```

XOR returns:

```text
1 when the bits are different
```

Example:

```text
  1101
^ 1011
------
  0110
```

Result:

```text
6
```

XOR is one of the most important operators in DSA.

---

# 6. NOT `~`

NOT flips every bit:

```text
0 → 1
1 → 0
```

Example for a simplified 4-bit representation:

```text
  0101
~ 0101
------
  1010
```

In Java, integers use signed two's-complement representation, so:

```java
~5
```

returns:

```text
-6
```

Important identity:

```text
~x = -x - 1
```

---

# 7. Left Shift `<<`

Left shift moves bits to the left.

```java
x << n
```

means:

```text
shift x left by n positions
```

For values where overflow is not involved:

```text
x << 1 ≈ x × 2
x << 2 ≈ x × 4
x << 3 ≈ x × 8
```

Example:

```text
5 = 0101

5 << 1

= 1010

= 10
```

---

# 8. Right Shift `>>`

Signed right shift moves bits right and preserves the sign bit.

Example:

```text
20 >> 2
```

gives:

```text
5
```

For positive integers:

```text
x >> 1 ≈ x / 2
```

using integer arithmetic.

---

# 9. Unsigned Right Shift `>>>`

Java also provides:

```java
>>>
```

It shifts bits right and fills the left side with:

```text
0
```

This matters for negative numbers.

Example:

```java
int x = -8;

x >> 1;
x >>> 1;
```

These produce different results.

---

# 10. Signed vs Unsigned Right Shift

### `>>`

Preserves the sign:

```text
positive → fills with 0
negative → fills with 1
```

### `>>>`

Always fills with:

```text
0
```

This is particularly useful when treating an integer as an unsigned bit pattern.

---

# 11. Check Whether a Bit Is Set

To check bit position `i`:

```java
(num & (1 << i)) != 0
```

Example:

```java
int num = 10;
// binary: 1010

boolean set =
    (num & (1 << 3)) != 0;
```

Bit 3 is:

```text
1
```

so:

```text
true
```

---

# 12. Set a Bit

Set bit `i` to `1`:

```java
num =
    num | (1 << i);
```

Example:

```text
num = 1000
```

Set bit 1:

```text
1000
OR
0010
----
1010
```

---

# 13. Clear a Bit

Clear bit `i`:

```java
num =
    num & ~(1 << i);
```

Example:

```text
1010
```

Clear bit 3:

```text
1010
AND
0111
----
0010
```

---

# 14. Toggle a Bit

Toggle bit `i`:

```java
num =
    num ^ (1 << i);
```

If the bit is:

```text
0 → 1
```

If the bit is:

```text
1 → 0
```

---

# 15. Extract the Lowest Set Bit

Very important identity:

```java
x & -x
```

returns the value of the lowest set bit.

Example:

```text
x = 12

binary:
1100
```

Then:

```text
x & -x
```

gives:

```text
0100
```

which is:

```text
4
```

---

# 16. Remove the Lowest Set Bit

Another extremely important identity:

```java
x & (x - 1)
```

removes the lowest set bit.

Example:

```text
x = 12

1100
```

Then:

```text
x - 1 = 1011
```

Therefore:

```text
  1100
& 1011
------
  1000
```

Result:

```text
8
```

---

# 17. Count Set Bits — Brian Kernighan

Use:

```java
static int countSetBits(int n) {

    int count = 0;

    while (n != 0) {

        n = n & (n - 1);

        count++;
    }

    return count;
}
```

Each iteration removes one set bit.

Therefore complexity is:

```text
O(number of set bits)
```

---

# 18. Java Built-in Bit Counting

For an `int`:

```java
Integer.bitCount(n);
```

For a `long`:

```java
Long.bitCount(n);
```

In interview code, the built-in method is often perfectly acceptable unless the interviewer specifically asks you to implement it.

---

# 19. Check Power of Two

A positive number is a power of two if it has exactly one set bit.

Examples:

```text
1  = 0001
2  = 0010
4  = 0100
8  = 1000
16 = 10000
```

Therefore:

```java
static boolean isPowerOfTwo(
        int n) {

    return n > 0
        && (n & (n - 1)) == 0;
}
```

---

# 20. Check Power of Four

A power of four is also a power of two, but its single set bit appears at an even bit position.

One approach:

```java
static boolean isPowerOfFour(
        int n) {

    return n > 0
        && (n & (n - 1)) == 0
        && (n & 0x55555555) != 0;
}
```

The mask:

```text
0x55555555
```

has bits set at even positions.

---

# 21. XOR Properties

Memorize these.

### Identity

```text
x ^ 0 = x
```

### Self cancellation

```text
x ^ x = 0
```

### Commutative

```text
a ^ b = b ^ a
```

### Associative

```text
(a ^ b) ^ c
=
a ^ (b ^ c)
```

These properties make XOR extremely useful for finding unique elements.

---

# 22. Find the Single Number

Every element appears twice except one.

Example:

```text
[4,1,2,1,2]
```

Answer:

```text
4
```

Use XOR:

```java
static int singleNumber(
        int[] nums) {

    int result = 0;

    for (int num : nums) {
        result ^= num;
    }

    return result;
}
```

Why?

Pairs cancel:

```text
4 ^ 1 ^ 2 ^ 1 ^ 2

= 4
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 23. Find Two Unique Numbers

Every number appears twice except two numbers.

Example:

```text
[1,2,1,3,2,5]
```

Unique numbers:

```text
3, 5
```

First XOR everything:

```text
xor = 3 ^ 5
```

Since:

```text
3 ^ 5 != 0
```

there is at least one bit where they differ.

---

# 24. Two Unique Numbers — Java

```java
static int[] singleNumbers(
        int[] nums) {

    int xor = 0;

    for (int num : nums) {
        xor ^= num;
    }

    int differentBit =
        xor & -xor;

    int first = 0;
    int second = 0;

    for (int num : nums) {

        if ((num & differentBit) == 0) {
            first ^= num;
        } else {
            second ^= num;
        }
    }

    return new int[]{
        first,
        second
    };
}
```

The differing bit divides the numbers into two groups.

Each duplicate pair stays in the same group and cancels.

---

# 25. Missing Number

Given:

```text
[3,0,1]
```

numbers should be:

```text
0,1,2,3
```

Missing:

```text
2
```

Use XOR:

```java
static int missingNumber(
        int[] nums) {

    int result =
        nums.length;

    for (int i = 0;
         i < nums.length;
         i++) {

        result ^= i;
        result ^= nums[i];
    }

    return result;
}
```

Everything appearing twice cancels.

---

# 26. XOR From 1 to N

XOR from:

```text
1 ^ 2 ^ 3 ^ ... ^ n
```

follows a pattern based on:

```text
n % 4
```

### If `n % 4 == 0`

```text
answer = n
```

### If `n % 4 == 1`

```text
answer = 1
```

### If `n % 4 == 2`

```text
answer = n + 1
```

### If `n % 4 == 3`

```text
answer = 0
```

---

# 27. XOR Swap

Mathematically:

```text
a = a ^ b
b = a ^ b
a = a ^ b
```

This swaps values without a temporary variable.

However, in normal Java code:

```java
int temp = a;
a = b;
b = temp;
```

is clearer and preferable.

Know XOR swap for interview concepts, but do not use it unnecessarily in production code.

---

# 28. Odd or Even

The least significant bit determines parity.

```java
(n & 1) == 0
```

means:

```text
even
```

and:

```java
(n & 1) != 0
```

means:

```text
odd
```

Example:

```text
6 = 110
LSB = 0 → even

7 = 111
LSB = 1 → odd
```

---

# 29. Clear Lowest Set Bit

Use:

```java
n = n & (n - 1);
```

This is useful for:

```text
Counting bits
Power-of-two checks
Bitmask algorithms
Subset algorithms
```

---

# 30. Isolate Lowest Set Bit

Use:

```java
int lowest =
    n & -n;
```

This isolates the lowest `1` bit.

For:

```text
n = 40
```

binary:

```text
101000
```

lowest set bit:

```text
001000
```

which is:

```text
8
```

---

# 31. Get Bit at Position

```java
static int getBit(
        int num,
        int position) {

    return (num >>> position) & 1;
}
```

Using `>>>` avoids sign extension when extracting bits.

---

# 32. Set Bit

```java
static int setBit(
        int num,
        int position) {

    return num
        | (1 << position);
}
```

---

# 33. Clear Bit

```java
static int clearBit(
        int num,
        int position) {

    return num
        & ~(1 << position);
}
```

---

# 34. Toggle Bit

```java
static int toggleBit(
        int num,
        int position) {

    return num
        ^ (1 << position);
}
```

---

# 35. Update Bit

To set a bit to a specific value:

```java
static int updateBit(
        int num,
        int position,
        int value) {

    num =
        num & ~(1 << position);

    return num
        | (value << position);
}
```

Assumes:

```text
value = 0 or 1
```

---

# 36. Bitmask

A bitmask uses bits to represent a set of boolean states.

Example:

```text
0000
```

means no states selected.

```text
0101
```

means:

```text
state 0 selected
state 2 selected
```

Bitmasks are very useful when the number of states is small.

---

# 37. Subsets Using Bitmask

For:

```text
nums = [a,b,c]
```

there are:

```text
2^3 = 8
```

subsets.

Use masks:

```text
000
001
010
011
100
101
110
111
```

Each bit tells whether to include an element.

---

# 38. Generate Subsets Using Bitmask

```java
static List<List<Integer>>
subsets(int[] nums) {

    List<List<Integer>> result =
        new ArrayList<>();

    int total =
        1 << nums.length;

    for (int mask = 0;
         mask < total;
         mask++) {

        List<Integer> subset =
            new ArrayList<>();

        for (int i = 0;
             i < nums.length;
             i++) {

            if ((mask & (1 << i))
                    != 0) {

                subset.add(nums[i]);
            }
        }

        result.add(subset);
    }

    return result;
}
```

Complexity:

```text
O(n × 2^n)
```

---

# 39. Bitmask DP

Bitmasks can represent which items have already been selected.

For example:

```text
mask = 01011
```

means:

```text
items 0, 1, 3 selected
```

This is common in:

```text
Traveling Salesman Problem
Assignment problems
Subset DP
Scheduling DP
State compression
```

---

# 40. Check Whether a Bitmask Contains an Item

```java
boolean used =
    (mask & (1 << i)) != 0;
```

Add item:

```java
mask =
    mask | (1 << i);
```

Remove item:

```java
mask =
    mask & ~(1 << i);
```

Toggle item:

```java
mask =
    mask ^ (1 << i);
```

---

# 41. Iterate Through All Masks

For `n` boolean states:

```java
for (int mask = 0;
     mask < (1 << n);
     mask++) {

    // process mask
}
```

There are:

```text
2^n
```

masks.

This is a fundamental subset-enumeration pattern.

---

# 42. Enumerate Set Bits

Efficiently iterate through set bits:

```java
int mask = n;

while (mask != 0) {

    int bit =
        mask & -mask;

    // Process bit.

    mask &=
        mask - 1;
}
```

This runs once per set bit.

---

# 43. Power of Two Using Bitmask

Remember:

```java
n > 0
&& (n & (n - 1)) == 0
```

Why?

A power of two has:

```text
exactly one set bit
```

Removing that bit leaves:

```text
0
```

---

# 44. Hamming Distance

Hamming distance between two integers is the number of differing bits.

Use XOR:

```java
a ^ b
```

Then count set bits.

```java
static int hammingDistance(
        int a,
        int b) {

    return Integer.bitCount(
        a ^ b
    );
}
```

---

# 45. Hamming Weight

Hamming weight means:

```text
number of 1 bits
```

Example:

```text
101101
```

contains:

```text
4
```

set bits.

Java:

```java
Integer.bitCount(n);
```

---

# 46. Reverse Bits

A classic bit manipulation problem.

Process each bit:

```java
static int reverseBits(
        int n) {

    int result = 0;

    for (int i = 0;
         i < 32;
         i++) {

        result <<= 1;

        result |=
            (n & 1);

        n >>>= 1;
    }

    return result;
}
```

Important:

```text
Use >>> for unsigned shifting.
```

---

# 47. Count Trailing Zeros

Java provides:

```java
Integer.numberOfTrailingZeros(n);
```

For `long`:

```java
Long.numberOfTrailingZeros(n);
```

For example, if:

```text
n = 40
binary = 101000
```

there are:

```text
3
```

trailing zeros.

---

# 48. Count Leading Zeros

Java:

```java
Integer.numberOfLeadingZeros(n);
```

For `long`:

```java
Long.numberOfLeadingZeros(n);
```

This can be useful when determining the highest set bit.

---

# 49. Highest Set Bit

For a positive integer:

```java
Integer.highestOneBit(n);
```

returns the value of the highest set bit.

Example:

```text
n = 13
binary = 1101
```

highest set bit:

```text
1000
```

which is:

```text
8
```

---

# 50. Lowest Set Bit

Java provides:

```java
Integer.lowestOneBit(n);
```

Equivalent conceptually to:

```java
n & -n
```

Example:

```text
12 = 1100
```

lowest set bit:

```text
0100
```

---

# 51. Rotate Bits

Java provides:

```java
Integer.rotateLeft(n, distance);

Integer.rotateRight(n, distance);
```

These differ from normal shifts because bits that leave one side are reintroduced on the other side.

Bit rotations are common in:

```text
Hashing
Cryptographic algorithms
Checksums
Low-level algorithms
```

---

# 52. Signed Integer Representation

Java `int` uses:

```text
32 bits
```

and:

```text
two's complement
```

The highest bit is the sign bit.

```text
0 → non-negative
1 → negative
```

For `long`:

```text
64 bits
```

---

# 53. Negative Numbers

Negative numbers can look surprising in bit operations.

Example:

```java
int x = -1;
```

binary representation:

```text
11111111111111111111111111111111
```

Therefore:

```java
x >>> 1
```

produces:

```text
01111111111111111111111111111111
```

while:

```java
x >> 1
```

keeps the sign:

```text
11111111111111111111111111111111
```

---

# 54. Overflow with Shifts

Be careful with:

```java
1 << 31
```

because Java's `int` is signed.

The result is:

```text
Integer.MIN_VALUE
```

For masks involving large bit positions, understand whether you need:

```java
int
```

or:

```java
long
```

---

# 55. Use `1L` for Long Masks

For a 64-bit mask:

```java
1L << position
```

instead of:

```java
1 << position
```

Example:

```java
long mask =
    1L << 40;
```

Using `1` would perform the shift as an `int`.

---

# 56. XOR and Missing Values

Bit manipulation is especially useful when values cancel in pairs.

Pattern:

```text
a ^ b ^ a
=
b
```

This can solve:

```text
Single Number
Missing Number
Duplicate cancellation
Two unique values
```

---

# 57. XOR and Array Problems

If every element appears twice:

```text
a ^ b ^ c ^ a ^ b
```

the pairs cancel:

```text
c
```

This gives:

```text
O(n)
```

time and:

```text
O(1)
```

extra space.

---

# 58. XOR and Range Queries

Prefix XOR can be used similarly to prefix sums.

Define:

```text
prefix[i]
=
a[0] ^ a[1] ^ ... ^ a[i]
```

Then XOR of a range can be obtained using XOR cancellation.

Conceptually:

```text
range XOR
=
prefix[right]
^
prefix[left - 1]
```

when `left > 0`.

---

# 59. Bitwise AND of a Range

For finding:

```text
AND of all numbers from left to right
```

a useful observation is that common high-order bits survive while differing lower bits become zero.

One approach is to repeatedly shift both boundaries right until they become equal, then shift back.

---

# 60. Bitwise AND Range — Java

```java
static int rangeBitwiseAnd(
        int left,
        int right) {

    int shift = 0;

    while (left < right) {

        left >>= 1;
        right >>= 1;

        shift++;
    }

    return left << shift;
}
```

The idea is to find the common binary prefix.

---

# 61. XOR Trie Connection

A binary Trie stores:

```text
0 / 1
```

instead of characters.

For maximum XOR:

```text
At each bit:
prefer the opposite bit.
```

This is a direct application of bit manipulation + Trie.

---

# 62. Bit Manipulation + Backtracking

Bitmasks can replace:

```text
boolean[] used
```

when the number of elements is small.

Instead of:

```java
boolean[] used;
```

use:

```java
int mask;
```

Then:

```java
if ((mask & (1 << i)) != 0) {
    continue;
}
```

This can make state representation compact.

---

# 63. Bitmask + DFS

For a small set of nodes:

```text
visited = mask
```

At node `i`:

```java
int newMask =
    mask | (1 << i);
```

This is useful in:

```text
Subset DP
Hamiltonian path
TSP
State-space search
```

---

# 64. Bitmask + BFS

A state can combine:

```text
current node
+
visited mask
```

Example:

```text
(node, mask)
```

Two states are considered different if their masks differ.

This appears in problems such as:

```text
Shortest path visiting all nodes
```

---

# 65. Common Bit Manipulation Mistakes

### Mistake 1 — Confusing `&` with `&&`

```text
&  → bitwise AND
&& → logical AND
```

---

### Mistake 2 — Confusing `|` with `||`

```text
|  → bitwise OR
|| → logical OR
```

---

### Mistake 3 — Forgetting signed shifts

```text
>>>
```

and:

```text
>>
```

behave differently for negative numbers.

---

### Mistake 4 — Using `1 << i` for long masks

Use:

```java
1L << i
```

when working with `long`.

---

### Mistake 5 — Forgetting operator precedence

Use parentheses:

```java
if ((num & (1 << i)) != 0)
```

instead of relying on precedence.

---

### Mistake 6 — Ignoring overflow

Shifts can overflow the fixed-width integer type.

---

# 66. Edge Cases

Always test:

```text
0
1
-1
Integer.MAX_VALUE
Integer.MIN_VALUE
Power of two
Power of four
All bits set
No bits set
Duplicate values
Large bit positions
int vs long
```

---

# 67. Java Bit Utility Methods

Useful methods to remember:

```java
Integer.bitCount(n);

Integer.numberOfLeadingZeros(n);

Integer.numberOfTrailingZeros(n);

Integer.highestOneBit(n);

Integer.lowestOneBit(n);

Integer.reverse(n);

Integer.reverseBytes(n);

Integer.rotateLeft(n, distance);

Integer.rotateRight(n, distance);
```

For `long`:

```java
Long.bitCount(n);

Long.numberOfLeadingZeros(n);

Long.numberOfTrailingZeros(n);

Long.highestOneBit(n);

Long.lowestOneBit(n);
```

---

# 68. Interview Questions — Easy

1. Check odd/even.
2. Check power of two.
3. Count set bits.
4. Find single number.
5. Find missing number.
6. Hamming distance.
7. Reverse bits.
8. Get/set/clear/toggle a bit.
9. Find highest set bit.
10. Find lowest set bit.

---

# 69. Interview Questions — Medium

11. Two unique numbers.
12. Power of four.
13. Bitwise AND of a range.
14. Subsets using bitmask.
15. XOR prefix queries.
16. Sum of two integers without `+`.
17. Maximum XOR of two numbers.
18. Counting bits for all numbers.
19. Binary Trie.
20. Bitmask DFS.

---

# 70. Interview Questions — Advanced

21. Maximum XOR with Binary Trie.
22. Shortest path with visited-state bitmask.
23. Traveling Salesman Problem with bitmask DP.
24. Hamiltonian path using bitmask.
25. Assignment problem with bitmask DP.
26. State compression DP.
27. Bitmask + BFS.
28. Advanced XOR range problems.
29. Trie + bit manipulation.
30. Subset DP.

---

# 71. Complexity Summary

| Problem | Typical Complexity | Extra Space |
|---|---:|---:|
| Check Bit | O(1) | O(1) |
| Set/Clear/Toggle Bit | O(1) | O(1) |
| Count Set Bits | O(number of set bits) | O(1) |
| Single Number | O(n) | O(1) |
| Missing Number | O(n) | O(1) |
| Hamming Distance | O(1) for fixed int | O(1) |
| Generate Subsets | O(n × 2^n) | O(n) excluding output |
| Maximum XOR Trie | O(32n) | O(32n) |
| Bitmask DP | Often O(n × 2^n) | Often O(2^n) |
| TSP Bitmask DP | O(n² × 2^n) | O(n × 2^n) |

---

# 72. Quick Revision

```text
Bit Manipulation
│
├── Operators
│   ├── &
│   ├── |
│   ├── ^
│   ├── ~
│   ├── <<
│   ├── >>
│   └── >>>
│
├── Core Tricks
│   ├── x & (x - 1)
│   ├── x & -x
│   ├── x ^ x = 0
│   └── x ^ 0 = x
│
├── Bit Operations
│   ├── Get
│   ├── Set
│   ├── Clear
│   └── Toggle
│
├── XOR Problems
│   ├── Single Number
│   ├── Missing Number
│   ├── Two Unique Numbers
│   └── Maximum XOR
│
├── Bitmask
│   ├── Subsets
│   ├── State Compression
│   ├── DFS
│   ├── BFS
│   └── DP
│
└── Advanced
    ├── Binary Trie
    ├── XOR Trie
    └── Bitmask DP
```

---

## Most Important Tricks to Memorize

```java
// Check bit i
(num & (1 << i)) != 0

// Set bit i
num | (1 << i)

// Clear bit i
num & ~(1 << i)

// Toggle bit i
num ^ (1 << i)

// Remove lowest set bit
num & (num - 1)

// Isolate lowest set bit
num & -num

// Power of two
n > 0 && (n & (n - 1)) == 0

// Count set bits
Integer.bitCount(n)
```

---

## Interview Rule

> **Bit manipulation becomes much easier when you recognize patterns. The five worth memorizing first are `x & (x - 1)`, `x & -x`, XOR cancellation, bit masks with `1 << i`, and the difference between `>>` and `>>>`.**
