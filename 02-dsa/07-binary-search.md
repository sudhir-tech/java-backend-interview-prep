# DSA — Binary Search

Binary Search is one of the most important DSA patterns for Java backend interviews.

The basic idea is to repeatedly eliminate half of the search space.

For a sorted array:

```text
O(n) linear search
        ↓
O(log n) binary search
```

Binary Search is useful for:

- Searching sorted arrays
- Finding first/last occurrence
- Finding insertion position
- Rotated sorted arrays
- Finding peaks
- Binary Search on Answer
- Minimum/maximum feasible values
- Capacity and allocation problems

---

# 1. What is Binary Search?

Given a sorted array:

```text
[1, 3, 5, 7, 9, 11]
```

To find:

```text
7
```

instead of checking every element, inspect the middle.

```text
middle = 5
```

Since:

```text
7 > 5
```

ignore the left half.

Then search:

```text
[7, 9, 11]
```

This repeatedly cuts the search space in half.

---

# 2. Complexity

```text
Time:  O(log n)
Space: O(1)
```

For recursive binary search:

```text
Space: O(log n)
```

because of the recursion stack.

---

# 3. Basic Binary Search

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

---

# 4. Why Use `left + (right - left) / 2`?

Avoid:

```java
int mid = (left + right) / 2;
```

because `left + right` can overflow for very large integer indexes.

Prefer:

```java
int mid =
    left + (right - left) / 2;
```

This is a common interview detail.

---

# 5. Binary Search Invariant

During the search:

```text
The target, if it exists, is inside [left, right].
```

Every iteration must preserve this invariant.

If:

```text
nums[mid] < target
```

then everything from:

```text
left ... mid
```

can be eliminated.

Set:

```java
left = mid + 1;
```

If:

```text
nums[mid] > target
```

eliminate:

```text
mid ... right
```

Set:

```java
right = mid - 1;
```

---

# 6. Recursive Binary Search

```java
static int binarySearch(
        int[] nums,
        int left,
        int right,
        int target) {

    if (left > right) {
        return -1;
    }

    int mid =
        left + (right - left) / 2;

    if (nums[mid] == target) {
        return mid;
    }

    if (nums[mid] < target) {

        return binarySearch(
            nums,
            mid + 1,
            right,
            target
        );

    }

    return binarySearch(
        nums,
        left,
        mid - 1,
        target
    );
}
```

---

# 7. Search Insert Position

Find the position where `target` should be inserted into a sorted array.

Example:

```text
nums = [1, 3, 5, 6]
target = 5

answer = 2
```

If:

```text
target = 2
```

answer:

```text
1
```

Implementation:

```java
static int searchInsert(
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

This is a **lower bound**.

---

# 8. Lower Bound

Lower bound means:

> Find the first index where `nums[index] >= target`.

Template:

```java
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
```

Notice:

```text
right = n
```

instead of:

```text
right = n - 1
```

This allows the answer to be:

```text
n
```

when every element is smaller than the target.

---

# 9. Upper Bound

Upper bound means:

> Find the first index where `nums[index] > target`.

Template:

```java
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
```

---

# 10. Lower Bound vs Upper Bound

For:

```text
nums = [1, 2, 2, 2, 4]
target = 2
```

Lower bound:

```text
index 1
```

Upper bound:

```text
index 4
```

Therefore:

```text
count of target
=
upperBound - lowerBound
```

```text
4 - 1 = 3
```

---

# 11. First Occurrence

If duplicates exist, normal binary search may return any occurrence.

To find the first:

```java
static int firstOccurrence(
        int[] nums,
        int target) {

    int left = 0;
    int right = nums.length - 1;

    int answer = -1;

    while (left <= right) {

        int mid =
            left + (right - left) / 2;

        if (nums[mid] == target) {

            answer = mid;
            right = mid - 1;

        } else if (nums[mid] < target) {

            left = mid + 1;

        } else {

            right = mid - 1;
        }
    }

    return answer;
}
```

Key idea:

```text
Found target
→ save answer
→ continue searching left
```

---

# 12. Last Occurrence

```java
static int lastOccurrence(
        int[] nums,
        int target) {

    int left = 0;
    int right = nums.length - 1;

    int answer = -1;

    while (left <= right) {

        int mid =
            left + (right - left) / 2;

        if (nums[mid] == target) {

            answer = mid;
            left = mid + 1;

        } else if (nums[mid] < target) {

            left = mid + 1;

        } else {

            right = mid - 1;
        }
    }

    return answer;
}
```

Key idea:

```text
Found target
→ save answer
→ continue searching right
```

---

# 13. Count Occurrences

Once you have:

```text
first occurrence
last occurrence
```

count:

```java
last - first + 1
```

Or:

```text
upperBound - lowerBound
```

depending on the implementation.

---

# 14. Find Floor

Floor of `target`:

> Largest value less than or equal to target.

Example:

```text
nums = [1, 3, 5, 7]
target = 6
```

Floor:

```text
5
```

Binary search can track the best candidate.

```java
int left = 0;
int right = nums.length - 1;
int answer = -1;

while (left <= right) {

    int mid =
        left + (right - left) / 2;

    if (nums[mid] <= target) {

        answer = nums[mid];
        left = mid + 1;

    } else {

        right = mid - 1;
    }
}
```

---

# 15. Find Ceiling

Ceiling means:

> Smallest value greater than or equal to target.

Example:

```text
nums = [1, 3, 5, 7]
target = 6
```

Ceiling:

```text
7
```

```java
int left = 0;
int right = nums.length - 1;
int answer = -1;

while (left <= right) {

    int mid =
        left + (right - left) / 2;

    if (nums[mid] >= target) {

        answer = nums[mid];
        right = mid - 1;

    } else {

        left = mid + 1;
    }
}
```

---

# 16. Search in Rotated Sorted Array

Example:

```text
[4, 5, 6, 7, 0, 1, 2]
```

The array was originally sorted but rotated.

At least one half of the current range is always sorted.

```java
static int searchRotated(
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

        if (nums[left] <= nums[mid]) {

            // Left half is sorted.

            if (nums[left] <= target
                    && target < nums[mid]) {

                right = mid - 1;

            } else {

                left = mid + 1;
            }

        } else {

            // Right half is sorted.

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

### Complexity

```text
Time:  O(log n)
Space: O(1)
```

---

# 17. Rotated Array with Duplicates

Duplicates can make it impossible to immediately determine which side is sorted.

Example:

```text
[2, 2, 2, 3, 2]
```

If:

```java
nums[left] == nums[mid]
    &&
nums[mid] == nums[right]
```

we may need to shrink both ends:

```java
left++;
right--;
```

This can degrade worst-case complexity to:

```text
O(n)
```

---

# 18. Find Minimum in Rotated Sorted Array

Example:

```text
[4, 5, 6, 7, 0, 1, 2]
```

Answer:

```text
0
```

```java
static int findMin(
        int[] nums) {

    int left = 0;
    int right = nums.length - 1;

    while (left < right) {

        int mid =
            left + (right - left) / 2;

        if (nums[mid] > nums[right]) {

            left = mid + 1;

        } else {

            right = mid;
        }
    }

    return nums[left];
}
```

### Complexity

```text
Time: O(log n)
Space: O(1)
```

---

# 19. Find Rotation Count

For a rotated sorted array without duplicates:

```text
rotation count
=
index of minimum element
```

Example:

```text
[4, 5, 6, 7, 0, 1, 2]
```

minimum index:

```text
4
```

rotation count:

```text
4
```

---

# 20. Find Peak Element

A peak is an element greater than its neighbors.

Example:

```text
[1, 2, 3, 1]
```

Peak:

```text
index 2
```

Binary search can compare:

```text
nums[mid]
nums[mid + 1]
```

```java
static int findPeak(
        int[] nums) {

    int left = 0;
    int right = nums.length - 1;

    while (left < right) {

        int mid =
            left + (right - left) / 2;

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

### Complexity

```text
Time:  O(log n)
Space: O(1)
```

---

# 21. Binary Search on Answer

This is one of the most important advanced patterns.

Instead of searching an array:

```text
Search the possible answer range.
```

Example:

```text
Minimum capacity
Minimum speed
Minimum time
Maximum minimum distance
Minimum maximum workload
```

---

# 22. How to Recognize Binary Search on Answer

Look for:

```text
1. Answer lies in a numeric range.
2. You can check if a candidate is feasible.
3. Feasibility changes monotonically.
```

For example:

```text
capacity too small → impossible
capacity large enough → possible
```

Once it becomes possible, larger capacities remain possible.

That monotonicity enables binary search.

---

# 23. Generic Binary Search on Answer

```java
long low = minimumPossible;
long high = maximumPossible;

while (low < high) {

    long mid =
        low + (high - low) / 2;

    if (isPossible(mid)) {
        high = mid;
    } else {
        low = mid + 1;
    }
}

return low;
```

This is a **first feasible value** pattern.

---

# 24. Koko Eating Bananas

Classic binary-search-on-answer problem.

Given piles of bananas and `h` hours, find the minimum eating speed.

Search:

```text
speed = 1
to
max(piles)
```

Feasibility:

```text
Can all bananas be eaten within h hours?
```

---

# 25. Koko Feasibility Check

```java
static boolean canFinish(
        int[] piles,
        int speed,
        int hours) {

    long required = 0;

    for (int pile : piles) {

        required +=
            (pile + (long) speed - 1)
            / speed;

        if (required > hours) {
            return false;
        }
    }

    return true;
}
```

Binary search:

```java
int low = 1;
int high =
    Arrays.stream(piles).max().getAsInt();

while (low < high) {

    int mid =
        low + (high - low) / 2;

    if (canFinish(
            piles,
            mid,
            h)) {

        high = mid;

    } else {

        low = mid + 1;
    }
}

return low;
```

---

# 26. Capacity to Ship Packages Within D Days

Search:

```text
low = maximum package weight
high = total weight
```

For each candidate capacity:

```text
Can all packages be shipped within D days?
```

The feasibility function is monotonic.

Therefore:

```text
Binary Search on Answer
```

---

# 27. Split Array Largest Sum

Goal:

> Split an array into `k` parts while minimizing the largest subarray sum.

Search the answer:

```text
low = maximum element
high = total sum
```

Check whether a candidate maximum sum allows the array to be split into at most `k` parts.

---

# 28. Allocate Minimum Pages

Given books and students:

```text
Assign books in order
Minimize the maximum pages assigned
```

Search:

```text
low = maximum pages in one book
high = total pages
```

Feasibility:

```text
Can we allocate all books to at most k students
if no student gets more than candidate pages?
```

---

# 29. Aggressive Cows

Goal:

> Place cows in stalls so that the minimum distance between any two cows is maximized.

Search the answer:

```text
minimum distance
```

Feasibility:

```text
Can we place all cows
with at least this distance?
```

This is a classic:

```text
Maximize minimum
```

binary-search pattern.

---

# 30. Minimize Maximum vs Maximize Minimum

### Minimize maximum

Examples:

```text
Shipping capacity
Book allocation
Split array
Workload distribution
```

Pattern:

```text
Find first feasible answer
```

### Maximize minimum

Examples:

```text
Aggressive cows
Minimum distance
Router placement
```

Pattern:

```text
Find last feasible answer
```

---

# 31. First Feasible vs Last Feasible

### First feasible

Search for minimum valid value:

```java
if (isPossible(mid)) {
    high = mid;
} else {
    low = mid + 1;
}
```

### Last feasible

Search for maximum valid value:

```java
if (isPossible(mid)) {
    low = mid + 1;
} else {
    high = mid - 1;
}
```

Recognizing this distinction makes binary-search-on-answer problems much easier.

---

# 32. Binary Search with a Monotonic Predicate

Think of feasibility as:

```text
false false false false true true true
                         ↑
                    first true
```

Binary search finds the transition.

Or:

```text
true true true true false false
                ↑
           last true
```

Binary search can find that transition too.

---

# 33. Lower Bound as First True

A useful mental model:

```text
nums[mid] >= target
```

is a predicate.

For:

```text
[1, 2, 2, 4, 7]
target = 2
```

predicate:

```text
false false true true true
```

Lower bound finds the first `true`.

This is a powerful way to understand boundary binary search.

---

# 34. Upper Bound as First True

Predicate:

```text
nums[mid] > target
```

For:

```text
[1, 2, 2, 4, 7]
target = 2
```

becomes:

```text
false false false true true
```

Upper bound finds the first `true`.

---

# 35. Binary Search on a Function

Binary search does not require an array.

If a function is monotonic:

```text
f(x)
```

and you can determine whether:

```text
f(x) >= target
```

you may be able to binary search over `x`.

Examples:

```text
Minimum speed
Minimum capacity
Minimum time
Maximum distance
```

---

# 36. Integer Square Root

Find:

```text
floor(sqrt(x))
```

without using `Math.sqrt()`.

Search:

```text
1 ... x
```

For large values, use `long` when multiplying:

```java
long square =
    (long) mid * mid;
```

Example:

```java
static int sqrt(int x) {

    if (x < 2) {
        return x;
    }

    int left = 1;
    int right = x / 2;

    while (left <= right) {

        int mid =
            left + (right - left) / 2;

        long square =
            (long) mid * mid;

        if (square == x) {
            return mid;
        }

        if (square < x) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return right;
}
```

---

# 37. Find Exact Square Root

If the input may not be a perfect square, distinguish between:

```text
floor square root
```

and:

```text
exact square root
```

For exact:

```text
if mid * mid == x
    answer found
```

Otherwise no integer square root exists.

---

# 38. Search in a 2D Matrix

If the matrix is globally sorted as if flattened:

```text
1  3  5
7  9  11
13 15 17
```

we can treat it as a one-dimensional sorted array.

For:

```text
rows = m
cols = n
```

map:

```text
index
```

to:

```text
row = index / cols
col = index % cols
```

---

# 39. Search 2D Matrix — Java

```java
static boolean searchMatrix(
        int[][] matrix,
        int target) {

    if (matrix.length == 0
            || matrix[0].length == 0) {
        return false;
    }

    int rows = matrix.length;
    int cols = matrix[0].length;

    int left = 0;
    int right = rows * cols - 1;

    while (left <= right) {

        int mid =
            left + (right - left) / 2;

        int row = mid / cols;
        int col = mid % cols;

        if (matrix[row][col] == target) {
            return true;
        }

        if (matrix[row][col] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return false;
}
```

### Complexity

```text
Time:  O(log(mn))
Space: O(1)
```

---

# 40. Search in Row/Column Sorted Matrix

If rows and columns are sorted but the entire matrix is not one globally sorted sequence, a different approach is used.

Start from:

```text
top-right
```

Then:

```text
if current > target:
    move left

if current < target:
    move down
```

Complexity:

```text
O(rows + cols)
```

This is not ordinary binary search.

---

# 41. Median of Two Sorted Arrays

This is a famous hard binary-search problem.

The idea is to binary search a partition in the smaller array.

You want:

```text
max(left partition)
<=
min(right partition)
```

for both arrays.

Typical complexity:

```text
O(log(min(m, n)))
```

This is a high-value advanced interview topic.

---

# 42. Binary Search with Duplicates

Duplicates change some boundary behavior.

For:

```text
[2, 2, 2, 3, 4]
```

normal search is still possible, but if you need:

```text
first
last
count
range
```

use explicit boundary logic.

Do not assume a normal binary search returns the first occurrence.

---

# 43. Java Built-in Binary Search

Java provides:

```java
Arrays.binarySearch(
    nums,
    target
);
```

Example:

```java
int index =
    Arrays.binarySearch(
        nums,
        target
    );
```

The array must be sorted for the expected binary-search semantics.

---

# 44. Important `Arrays.binarySearch()` Detail

If the target is not found, Java returns:

```text
-(insertion point) - 1
```

Example conceptually:

```text
insertion point = 2

return = -3
```

Recover insertion point:

```java
int insertionPoint =
    -result - 1;
```

---

# 45. Binary Search on Strings

Binary search can be used on sorted strings.

```java
String[] names = {
    "Alice",
    "Bob",
    "Charlie",
    "David"
};

int left = 0;
int right = names.length - 1;

while (left <= right) {

    int mid =
        left + (right - left) / 2;

    int comparison =
        names[mid].compareTo(target);

    if (comparison == 0) {
        return mid;
    }

    if (comparison < 0) {
        left = mid + 1;
    } else {
        right = mid - 1;
    }
}
```

---

# 46. Common Binary Search Templates

## Exact Search

```java
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
```

---

## Lower Bound

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

---

## Upper Bound

```java
while (left < right) {

    int mid =
        left + (right - left) / 2;

    if (nums[mid] <= target) {
        left = mid + 1;
    } else {
        right = mid;
    }
}
```

---

## First Feasible

```java
while (low < high) {

    long mid =
        low + (high - low) / 2;

    if (isPossible(mid)) {
        high = mid;
    } else {
        low = mid + 1;
    }
}
```

---

## Last Feasible

```java
while (low <= high) {

    long mid =
        low + (high - low) / 2;

    if (isPossible(mid)) {
        low = mid + 1;
    } else {
        high = mid - 1;
    }
}

return high;
```

---

# 47. Choosing the Correct Template

Ask:

```text
Need exact value?
→ Exact Search

Need first >= target?
→ Lower Bound

Need first > target?
→ Upper Bound

Need first valid answer?
→ First Feasible

Need largest valid answer?
→ Last Feasible
```

---

# 48. Common Binary Search Mistakes

### Mistake 1 — Searching unsorted data

Normal binary search requires a valid ordering.

---

### Mistake 2 — Wrong boundary

Mixing:

```text
left <= right
```

with:

```text
left < right
```

without understanding the invariant causes bugs.

---

### Mistake 3 — Infinite loops

Always make sure the search range shrinks.

For example:

```java
left = mid + 1;
```

not:

```java
left = mid;
```

when using a closed interval and the current midpoint has been ruled out.

---

### Mistake 4 — Incorrect midpoint

Prefer:

```java
left + (right - left) / 2
```

---

### Mistake 5 — Forgetting duplicates

Normal binary search does not guarantee first/last occurrence.

---

### Mistake 6 — Overflow in feasibility calculations

Use:

```java
long
```

when sums, products, capacities or time calculations can exceed `int`.

---

### Mistake 7 — Not proving monotonicity

Binary Search on Answer only works when feasibility is monotonic.

Ask:

> If this value is feasible, are all larger values also feasible?

or:

> If this value is infeasible, are all smaller values also infeasible?

---

# 49. Binary Search Problem Recognition

When you see:

```text
Sorted array
```

think:

```text
Binary Search
```

When you see:

```text
First / Last occurrence
```

think:

```text
Boundary Binary Search
```

When you see:

```text
Rotated sorted array
```

think:

```text
Modified Binary Search
```

When you see:

```text
Minimum possible maximum
```

think:

```text
Binary Search on Answer
```

When you see:

```text
Maximum possible minimum
```

think:

```text
Binary Search on Answer
```

When you see:

```text
Can we do it with X?
```

and feasibility is monotonic:

```text
Binary Search on Answer
```

---

# 50. Interview Questions — Easy

1. Binary Search.
2. Search Insert Position.
3. Find first occurrence.
4. Find last occurrence.
5. Count occurrences in a sorted array.
6. Find floor.
7. Find ceiling.
8. Integer square root.
9. Search in a sorted string array.
10. Find insertion position.

---

# 51. Interview Questions — Medium

11. Search in Rotated Sorted Array.
12. Find Minimum in Rotated Sorted Array.
13. Find Peak Element.
14. Find rotation count.
15. Search in a 2D matrix.
16. Find first and last position of an element.
17. Find a single element in a sorted array.
18. Find minimum speed to finish work.
19. Capacity to ship packages.
20. Allocate minimum pages.
21. Split array largest sum.
22. Koko Eating Bananas.
23. Aggressive Cows.
24. Find square root.
25. Find smallest divisor under a threshold.

---

# 52. Interview Questions — Advanced

26. Median of Two Sorted Arrays.
27. Binary Search on Answer problems.
28. Minimum time to complete trips.
29. Minimum days to make bouquets.
30. Kth smallest value using binary search.
31. Kth smallest pair distance.
32. Maximum minimum distance.
33. Minimize maximum workload.
34. Search in rotated array with duplicates.
35. Advanced partition-based binary search.

---

# 53. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Exact Binary Search | Binary Search | O(log n) | O(1) |
| First Occurrence | Boundary Search | O(log n) | O(1) |
| Last Occurrence | Boundary Search | O(log n) | O(1) |
| Lower Bound | Binary Search | O(log n) | O(1) |
| Upper Bound | Binary Search | O(log n) | O(1) |
| Rotated Search | Modified Binary Search | O(log n) | O(1) |
| Find Minimum Rotated | Binary Search | O(log n) | O(1) |
| Peak Element | Binary Search | O(log n) | O(1) |
| Search 2D Matrix | Binary Search | O(log(mn)) | O(1) |
| Binary Search on Answer | Search × Check | O(log R × check) | depends |
| Rotated with Duplicates | Modified Search | O(n) worst case | O(1) |
| Median Two Sorted Arrays | Partition Search | O(log min(m,n)) | O(1) |

`R` represents the size of the answer/search range.

---

# 54. Quick Revision

```text
Binary Search
│
├── Exact Search
│   └── Target exists?
│
├── Boundary Search
│   ├── Lower Bound
│   ├── Upper Bound
│   ├── First Occurrence
│   └── Last Occurrence
│
├── Rotated Arrays
│   ├── Search
│   ├── Minimum
│   └── Rotation Count
│
├── Peak Problems
│   └── Find Peak
│
├── Search Space
│   ├── Square Root
│   └── Numeric ranges
│
├── Binary Search on Answer
│   ├── Koko
│   ├── Shipping Capacity
│   ├── Book Allocation
│   ├── Split Array
│   └── Aggressive Cows
│
└── Advanced
    ├── 2D Matrix
    └── Median of Two Sorted Arrays
```

---

## Interview Rule

> **Binary Search is not just “searching a sorted array.” The real skill is recognizing a monotonic search space. If you can define a clear feasibility check and prove that the result changes only once from false to true (or true to false), binary search may be the right optimization.**
