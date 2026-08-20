# DSA — Advanced Binary Search

Binary Search is not limited to searching for a value in a sorted array.

Advanced Binary Search is mainly about recognizing when a problem has a **monotonic condition** or a **searchable answer space**.

Core patterns:

- Search in sorted arrays
- Lower Bound
- Upper Bound
- First and last occurrence
- Search on answer
- Minimum feasible value
- Maximum feasible value
- Rotated sorted arrays
- Minimum in rotated array
- Peak finding
- Binary search on floating-point values
- Binary search with duplicates
- Binary search over an implicit range
- Capacity / allocation problems
- Aggressive placement problems
- Koko Eating Bananas
- Ship Packages Within D Days
- Split Array Largest Sum
- Minimum Days to Make Bouquets
- Median of Two Sorted Arrays

---

# 1. Binary Search Fundamentals

Classic binary search works when the search space is ordered.

For:

```text
[1, 3, 5, 7, 9]
```

searching for:

```text
7
```

we repeatedly eliminate half of the remaining search space.

Complexity:

```text
Time: O(log n)
Space: O(1)
```

---

# 2. Basic Binary Search

```java
static int binarySearch(
        int[] nums,
        int target) {

    int left = 0;
    int right = nums.length - 1;

    while (left <= right) {

        int mid =
            left + (right - left) / 2;

        if (nums[mid] == target) {
            return mid;
        }

        if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;
}
```

Use:

```java
left + (right - left) / 2
```

instead of:

```java
(left + right) / 2
```

to avoid integer overflow.

---

# 3. Lower Bound

Lower Bound finds the first position where:

```text
nums[index] >= target
```

Example:

```text
nums = [1, 2, 2, 2, 4, 6]
target = 2
```

Answer:

```text
index 1
```

---

# 4. Lower Bound — Java

```java
static int lowerBound(
        int[] nums,
        int target) {

    int left = 0;
    int right = nums.length;

    while (left < right) {

        int mid =
            left + (right - left) / 2;

        if (nums[mid] < target) {

            left = mid + 1;

        } else {

            right = mid;
        }
    }

    return left;
}
```

Notice:

```text
right = nums.length
```

because the answer may be:

```text
n
```

when every element is smaller than the target.

---

# 5. Upper Bound

Upper Bound finds the first position where:

```text
nums[index] > target
```

Example:

```text
[1, 2, 2, 2, 4]
target = 2
```

Answer:

```text
index 4
```

---

# 6. Upper Bound — Java

```java
static int upperBound(
        int[] nums,
        int target) {

    int left = 0;
    int right = nums.length;

    while (left < right) {

        int mid =
            left + (right - left) / 2;

        if (nums[mid] <= target) {

            left = mid + 1;

        } else {

            right = mid;
        }
    }

    return left;
}
```

---

# 7. First Occurrence

To find the first occurrence of a target:

```text
Find lower bound.
```

Then verify:

```text
index < n
and
nums[index] == target
```

---

# 8. First Occurrence — Java

```java
static int firstOccurrence(
        int[] nums,
        int target) {

    int index =
        lowerBound(nums, target);

    if (index < nums.length
            && nums[index] == target) {

        return index;
    }

    return -1;
}
```

---

# 9. Last Occurrence

Use:

```text
upperBound(target) - 1
```

Then verify the value.

```java
static int lastOccurrence(
        int[] nums,
        int target) {

    int index =
        upperBound(nums, target) - 1;

    if (index >= 0
            && nums[index] == target) {

        return index;
    }

    return -1;
}
```

---

# 10. Count Occurrences

For a sorted array:

```text
count =
upperBound(target)
-
lowerBound(target)
```

Example:

```text
[1,2,2,2,4]
```

For target `2`:

```text
upper = 4
lower = 1

count = 3
```

---

# 11. Binary Search on Answer

This is one of the most important advanced binary-search patterns.

Instead of searching for:

```text
an array element
```

search for:

```text
the answer itself.
```

The key requirement is a monotonic feasibility condition.

---

# 12. Monotonic Predicate

Suppose:

```text
x = candidate answer
```

and:

```text
isPossible(x)
```

has this pattern:

```text
false false false true true true
```

Then we can binary search for:

```text
first true
```

Alternatively:

```text
true true true false false
```

lets us search for:

```text
last true
```

---

# 13. Search on Answer Template

```java
int left = minimumPossible;
int right = maximumPossible;

while (left < right) {

    int mid =
        left + (right - left) / 2;

    if (isPossible(mid)) {

        right = mid;

    } else {

        left = mid + 1;
    }
}

return left;
```

This finds the:

```text
smallest feasible answer.
```

---

# 14. How to Recognize Search on Answer

Look for questions such as:

```text
What is the minimum possible maximum?
What is the maximum possible minimum?
What is the minimum capacity?
What is the minimum speed?
What is the minimum time?
What is the maximum distance?
```

These often indicate:

```text
Binary Search on Answer
```

---

# 15. Koko Eating Bananas

Koko eats bananas at speed:

```text
k bananas/hour
```

Given piles and a time limit:

```text
h
```

find the minimum speed that allows all bananas to be eaten.

Possible speeds:

```text
1 ... max(piles)
```

---

# 16. Koko Feasibility

For speed `k`:

```text
hours =
ceil(pile / k)
```

Total hours:

```text
sum(ceil(pile / k))
```

If:

```text
hours <= h
```

the speed is feasible.

---

# 17. Koko — Java

```java
static int minEatingSpeed(
        int[] piles,
        int h) {

    int left = 1;
    int right = 0;

    for (int pile : piles) {
        right =
            Math.max(right, pile);
    }

    while (left < right) {

        int mid =
            left + (right - left) / 2;

        if (canFinish(
                piles,
                h,
                mid)) {

            right = mid;

        } else {

            left = mid + 1;
        }
    }

    return left;
}

static boolean canFinish(
        int[] piles,
        int h,
        int speed) {

    long hours = 0;

    for (int pile : piles) {

        hours +=
            (pile + speed - 1)
            / speed;

        if (hours > h) {
            return false;
        }
    }

    return true;
}
```

Complexity:

```text
O(n log M)
```

where `M` is the maximum pile size.

---

# 18. Ship Packages Within D Days

Given package weights in order, find the minimum ship capacity required to ship all packages within `days`.

Capacity must be at least:

```text
max(weights)
```

and at most:

```text
sum(weights)
```

---

# 19. Feasibility for Shipping

For a candidate capacity:

```text
capacity
```

simulate the shipment.

When adding the next package exceeds capacity:

```text
start another day.
```

If required days:

```text
<= allowed days
```

the capacity works.

---

# 20. Ship Packages — Java

```java
static int shipWithinDays(
        int[] weights,
        int days) {

    int left = 0;
    int right = 0;

    for (int weight : weights) {

        left =
            Math.max(left, weight);

        right += weight;
    }

    while (left < right) {

        int capacity =
            left
            + (right - left) / 2;

        if (canShip(
                weights,
                days,
                capacity)) {

            right = capacity;

        } else {

            left = capacity + 1;
        }
    }

    return left;
}

static boolean canShip(
        int[] weights,
        int days,
        int capacity) {

    int usedDays = 1;
    int current = 0;

    for (int weight : weights) {

        if (current + weight
                > capacity) {

            usedDays++;
            current = 0;
        }

        current += weight;

        if (usedDays > days) {
            return false;
        }
    }

    return true;
}
```

---

# 21. Split Array Largest Sum

Split an array into:

```text
k non-empty subarrays
```

Minimize the largest subarray sum.

This sounds like a difficult DP problem, but it can be solved using:

```text
Binary Search on Answer
```

---

# 22. Split Array Search Space

Minimum possible answer:

```text
max(nums)
```

Maximum possible answer:

```text
sum(nums)
```

For a candidate maximum sum:

```text
limit
```

check whether the array can be split into at most `k` parts where each part has sum:

```text
<= limit
```

---

# 23. Minimum Days to Make Bouquets

Given flower bloom days, determine the minimum day on which we can make:

```text
m bouquets
```

where each bouquet requires:

```text
k adjacent flowers
```

Search:

```text
minimum day
```

Feasibility:

```text
Can make at least m bouquets
by day X?
```

---

# 24. Impossible Bouquet Case

If:

```text
m * k > number of flowers
```

then the answer is immediately:

```text
-1
```

Use `long` when multiplying potentially large values:

```java
(long) m * k
```

---

# 25. Aggressive Cows / Maximum Minimum Distance

Given stall positions, place `k` cows so that the minimum distance between any two cows is maximized.

This is:

```text
maximize the minimum distance.
```

Search:

```text
distance
```

---

# 26. Aggressive Cows Greedy Check

For a candidate distance:

```text
d
```

place the first cow at the first stall.

Then greedily place each next cow at the earliest stall satisfying:

```text
position - lastPosition >= d
```

If at least `k` cows can be placed:

```text
d is feasible.
```

---

# 27. Aggressive Cows — Java

```java
static int maxMinDistance(
        int[] stalls,
        int k) {

    Arrays.sort(stalls);

    int left = 0;
    int right =
        stalls[stalls.length - 1]
        - stalls[0];

    while (left < right) {

        int mid =
            left
            + (right - left + 1) / 2;

        if (canPlace(
                stalls,
                k,
                mid)) {

            left = mid;

        } else {

            right = mid - 1;
        }
    }

    return left;
}

static boolean canPlace(
        int[] stalls,
        int k,
        int distance) {

    int count = 1;
    int last = stalls[0];

    for (int i = 1;
         i < stalls.length;
         i++) {

        if (stalls[i] - last
                >= distance) {

            count++;
            last = stalls[i];

            if (count >= k) {
                return true;
            }
        }
    }

    return false;
}
```

Notice the midpoint:

```java
(left + (right - left + 1) / 2)
```

This is useful when searching for the:

```text
maximum feasible value.
```

---

# 28. Minimum vs Maximum Feasible Answer

### Find minimum feasible

Use:

```java
if (isPossible(mid)) {
    right = mid;
} else {
    left = mid + 1;
}
```

### Find maximum feasible

Use:

```java
if (isPossible(mid)) {
    left = mid;
} else {
    right = mid - 1;
}
```

For maximum search, use an upper midpoint:

```java
mid =
    left + (right - left + 1) / 2;
```

---

# 29. Rotated Sorted Array

Original:

```text
[1,2,3,4,5,6,7]
```

rotated:

```text
[4,5,6,7,1,2,3]
```

The array is no longer globally sorted, but at least one half of the current search range is sorted.

This allows modified binary search.

---

# 30. Search in Rotated Sorted Array

At each step:

```text
mid
```

Determine which half is sorted.

If:

```text
nums[left] <= nums[mid]
```

then:

```text
left half is sorted.
```

Otherwise:

```text
right half is sorted.
```

Then determine whether the target lies inside the sorted half.

---

# 31. Search Rotated Array — Java

```java
static int search(
        int[] nums,
        int target) {

    int left = 0;
    int right =
        nums.length - 1;

    while (left <= right) {

        int mid =
            left
            + (right - left) / 2;

        if (nums[mid] == target) {
            return mid;
        }

        if (nums[left]
                <= nums[mid]) {

            if (nums[left] <= target
                    && target < nums[mid]) {

                right = mid - 1;

            } else {

                left = mid + 1;
            }

        } else {

            if (nums[mid] < target
                    && target <= nums[right]) {

                left = mid + 1;

            } else {

                right = mid - 1;
            }
        }
    }

    return -1;
}
```

Typical complexity:

```text
O(log n)
```

when there are no problematic duplicates.

---

# 32. Rotated Array with Duplicates

Duplicates can make it impossible to determine which half is sorted.

Example:

```text
[2,2,2,3,2]
```

If:

```text
nums[left]
==
nums[mid]
==
nums[right]
```

we may shrink:

```text
left++
right--
```

This can degrade the worst-case complexity to:

```text
O(n)
```

---

# 33. Find Minimum in Rotated Sorted Array

Example:

```text
[4,5,6,7,0,1,2]
```

Minimum:

```text
0
```

Compare:

```text
nums[mid]
```

with:

```text
nums[right]
```

If:

```text
nums[mid] > nums[right]
```

the minimum is to the right.

Otherwise:

```text
minimum is at mid or left.
```

---

# 34. Find Minimum — Java

```java
static int findMin(
        int[] nums) {

    int left = 0;
    int right =
        nums.length - 1;

    while (left < right) {

        int mid =
            left
            + (right - left) / 2;

        if (nums[mid]
                > nums[right]) {

            left = mid + 1;

        } else {

            right = mid;
        }
    }

    return nums[left];
}
```

---

# 35. Find Peak Element

A peak is an element greater than its neighbors.

You do not necessarily need to inspect every element.

Compare:

```text
nums[mid]
```

with:

```text
nums[mid + 1]
```

If:

```text
nums[mid] < nums[mid + 1]
```

a peak exists to the right.

Otherwise:

```text
a peak exists at mid or to the left.
```

---

# 36. Peak Element — Java

```java
static int findPeakElement(
        int[] nums) {

    int left = 0;
    int right =
        nums.length - 1;

    while (left < right) {

        int mid =
            left
            + (right - left) / 2;

        if (nums[mid]
                < nums[mid + 1]) {

            left = mid + 1;

        } else {

            right = mid;
        }
    }

    return left;
}
```

Complexity:

```text
O(log n)
```

---

# 37. Binary Search on Floating Point

Binary search can also work on continuous ranges.

Example:

```text
Find square root of x.
```

Search:

```text
[0, x]
```

until the interval is sufficiently small.

---

# 38. Square Root — Double

```java
static double sqrt(
        double x) {

    double left = 0;
    double right =
        Math.max(1.0, x);

    for (int i = 0;
         i < 100;
         i++) {

        double mid =
            left
            + (right - left) / 2;

        if (mid * mid <= x) {

            left = mid;

        } else {

            right = mid;
        }
    }

    return left;
}
```

A fixed number of iterations is often simpler than choosing an epsilon.

---

# 39. Binary Search with Epsilon

Another approach:

```java
while (right - left > 1e-9) {
    ...
}
```

The exact precision should depend on the problem's requirements.

Do not use unnecessary precision if the problem only asks for a few decimal places.

---

# 40. Median of Two Sorted Arrays

This is an advanced binary-search problem.

Given two sorted arrays:

```text
A
B
```

find their combined median without merging them.

The key idea is to binary search the partition in the smaller array.

---

# 41. Median Partition

Choose:

```text
i elements from A
j elements from B
```

such that:

```text
i + j
```

is half the combined size.

A valid partition satisfies:

```text
leftA <= rightB
```

and:

```text
leftB <= rightA
```

Then the median can be determined from the boundary values.

---

# 42. Median of Two Sorted Arrays — Java

```java
static double findMedianSortedArrays(
        int[] a,
        int[] b) {

    if (a.length > b.length) {
        return findMedianSortedArrays(b, a);
    }

    int m = a.length;
    int n = b.length;

    int left = 0;
    int right = m;

    while (left <= right) {

        int partitionA =
            left
            + (right - left) / 2;

        int partitionB =
            (m + n + 1) / 2
            - partitionA;

        int leftA =
            partitionA == 0
                ? Integer.MIN_VALUE
                : a[partitionA - 1];

        int rightA =
            partitionA == m
                ? Integer.MAX_VALUE
                : a[partitionA];

        int leftB =
            partitionB == 0
                ? Integer.MIN_VALUE
                : b[partitionB - 1];

        int rightB =
            partitionB == n
                ? Integer.MAX_VALUE
                : b[partitionB];

        if (leftA <= rightB
                && leftB <= rightA) {

            if ((m + n) % 2 == 1) {

                return Math.max(
                    leftA,
                    leftB
                );

            }

            return (
                (double)
                Math.max(
                    leftA,
                    leftB
                )
                +
                Math.min(
                    rightA,
                    rightB
                )
            ) / 2.0;
        }

        if (leftA > rightB) {

            right =
                partitionA - 1;

        } else {

            left =
                partitionA + 1;
        }
    }

    throw new IllegalArgumentException(
        "Input arrays are invalid."
    );
}
```

Complexity:

```text
O(log(min(m, n)))
```

---

# 43. Binary Search in an Infinite Sorted Array

If the array size is conceptually unknown, first find a range.

Start:

```text
left = 0
right = 1
```

While:

```text
target > nums[right]
```

double:

```text
right *= 2
```

Then perform normal binary search.

This is called:

```text
Exponential Search
```

---

# 44. Exponential Search

Pattern:

```text
right = 1

while (right < n
        && nums[right] < target) {

    right *= 2;
}
```

Then:

```text
binary search
```

within:

```text
right / 2 ... right
```

Complexity:

```text
O(log p)
```

where `p` is the target position.

---

# 45. Binary Search on Implicit Search Space

The values being searched do not need to be stored in an array.

Example:

```text
minimum capacity
minimum speed
minimum time
maximum distance
```

The search space can simply be:

```text
[left, right]
```

and a function determines whether a candidate is feasible.

---

# 46. Feasibility Function

The most important part of answer-based binary search is:

```text
isPossible(mid)
```

It must be:

```text
monotonic
```

Example:

```text
capacity:

1 2 3 4 5 6 7 8

possible:

F F F F T T T T
```

Once it becomes true, it stays true.

---

# 47. Finding First True

Template:

```java
static int firstTrue(
        int left,
        int right) {

    while (left < right) {

        int mid =
            left
            + (right - left) / 2;

        if (isPossible(mid)) {

            right = mid;

        } else {

            left = mid + 1;
        }
    }

    return left;
}
```

---

# 48. Finding Last True

Template:

```java
static int lastTrue(
        int left,
        int right) {

    while (left < right) {

        int mid =
            left
            + (right - left + 1) / 2;

        if (isPossible(mid)) {

            left = mid;

        } else {

            right = mid - 1;
        }
    }

    return left;
}
```

Memorize the difference:

```text
First true → lower midpoint
Last true  → upper midpoint
```

---

# 49. Binary Search Invariant

A good binary-search solution maintains an invariant.

For example:

```text
answer is always inside [left, right].
```

Every iteration must preserve this property.

This makes binary search easier to reason about and debug.

---

# 50. Common Binary Search Bugs

### Bug 1 — Infinite loop

Usually caused by:

```text
left = mid
```

when using a lower midpoint.

### Bug 2 — Wrong boundary

Mixing:

```text
left <= right
```

with:

```text
right = nums.length
```

without understanding the interval convention.

### Bug 3 — Overflow

Use:

```java
left + (right - left) / 2
```

### Bug 4 — Wrong monotonic direction

Determine whether you need:

```text
first true
```

or:

```text
last true
```

---

# 51. Closed vs Half-Open Intervals

Two common styles:

### Closed

```text
[left, right]
```

Condition:

```java
while (left <= right)
```

### Half-open

```text
[left, right)
```

Condition:

```java
while (left < right)
```

Both are valid.

The important thing is to remain consistent.

---

# 52. Advanced Binary Search Checklist

Before coding:

```text
□ Is the search space ordered?
□ Or is the answer space monotonic?
□ What is the minimum possible answer?
□ What is the maximum possible answer?
□ What does isPossible(x) mean?
□ Is isPossible monotonic?
□ Do I need first true or last true?
□ Can mid overflow?
□ Are values large enough to require long?
```

---

# 53. Binary Search + Greedy

Many answer-search problems use:

```text
Binary Search
+
Greedy feasibility check
```

Examples:

```text
Aggressive Cows
Ship Packages
Koko Eating Bananas
Split Array
Bouquets
```

This combination is extremely important for interviews.

---

# 54. Binary Search + Sorting

Some problems first require:

```text
sort input
```

then use:

```text
binary search
```

Examples:

```text
Search rotated array
Aggressive Cows
Closest elements
Pair counting
Coordinate placement
```

Overall complexity may be:

```text
O(n log n)
```

because sorting dominates.

---

# 55. Binary Search + Prefix Sum

For some problems, feasibility checks repeatedly calculate ranges.

Use:

```text
prefix sums
```

to make each range calculation faster.

This can turn:

```text
O(n² log M)
```

into:

```text
O(n log M)
```

in suitable problems.

---

# 56. Binary Search Complexity

If the answer range contains:

```text
M
```

possible values and each feasibility check costs:

```text
O(n)
```

then:

```text
O(n log M)
```

is the typical complexity.

---

# 57. Common Advanced Binary Search Problems

### Medium

1. Search in Rotated Sorted Array.
2. Find Minimum in Rotated Sorted Array.
3. Find Peak Element.
4. Koko Eating Bananas.
5. Capacity to Ship Packages.
6. Minimum Days to Make Bouquets.
7. Aggressive Cows.
8. Split Array Largest Sum.

### Advanced

9. Median of Two Sorted Arrays.
10. Binary Search on Floating Point.
11. Exponential Search.
12. Advanced allocation problems.
13. Binary search with complex feasibility checks.

---

# 58. Binary Search Pattern Table

| Pattern | Search |
|---|---|
| Exact value | Target |
| First occurrence | First equal |
| Last occurrence | Last equal |
| Lower bound | First `>= target` |
| Upper bound | First `> target` |
| Minimum feasible | First true |
| Maximum feasible | Last true |
| Rotated array | Sorted half |
| Peak | Slope |
| Median of two arrays | Partition |
| Continuous answer | Floating-point range |

---

# 59. Quick Revision

```text
Advanced Binary Search
│
├── Basic
│   ├── Exact Search
│   ├── Lower Bound
│   ├── Upper Bound
│   └── First / Last Occurrence
│
├── Search on Answer
│   ├── Minimum Feasible
│   ├── Maximum Feasible
│   └── Monotonic Predicate
│
├── Rotated Arrays
│   ├── Search
│   └── Minimum
│
├── Special
│   ├── Peak
│   ├── Floating Point
│   ├── Infinite Array
│   └── Median of Two Arrays
│
└── Combinations
    ├── Binary Search + Greedy
    ├── Binary Search + Sorting
    └── Binary Search + Prefix Sum
```

---

# 60. Most Important Templates

### Lower Bound

```java
while (left < right) {

    int mid =
        left + (right - left) / 2;

    if (nums[mid] < target) {
        left = mid + 1;
    } else {
        right = mid;
    }
}
```

### First Feasible

```java
while (left < right) {

    int mid =
        left + (right - left) / 2;

    if (isPossible(mid)) {
        right = mid;
    } else {
        left = mid + 1;
    }
}
```

### Last Feasible

```java
while (left < right) {

    int mid =
        left + (right - left + 1) / 2;

    if (isPossible(mid)) {
        left = mid;
    } else {
        right = mid - 1;
    }
}
```

---

# 61. Interview Explanation Template

For an advanced binary-search problem, say:

```text
The answer lies between [low] and [high].

For a candidate value mid, I can check whether it is feasible
using [greedy/checking logic].

The feasibility condition is monotonic:
once a value becomes feasible, all larger values are feasible
(or vice versa).

Therefore I can binary search the answer.

The complexity is O(checkCost × log searchSpace).
```

---

# 62. Final Interview Rule

> **When you see "minimum possible maximum", "maximum possible minimum", "minimum speed", "minimum capacity", or "minimum time", immediately ask whether the answer space is monotonic. If it is, Binary Search on Answer is likely the intended pattern.**
