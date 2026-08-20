# DSA — Arrays

Arrays are one of the most important topics for Java backend interviews. Many advanced DSA problems are built from array techniques such as hashing, two pointers, sliding window, prefix sums, binary search, and greedy algorithms.

---

## 1. What is an Array?

An array stores elements of the same type in contiguous memory locations.

```java
int[] numbers = {10, 20, 30, 40};
```

Access by index:

```java
System.out.println(numbers[2]); // 30
```

Array indexing starts from `0`.

---

## 2. Time Complexity

| Operation | Complexity |
|---|---:|
| Access by index | O(1) |
| Update by index | O(1) |
| Linear search | O(n) |
| Binary search on sorted array | O(log n) |
| Insert at beginning | O(n) |
| Insert at end* | O(1) |
| Delete from beginning | O(n) |
| Delete from middle | O(n) |

`*` Assuming enough capacity / an appropriate dynamic array structure.

---

## 3. Basic Traversal

```java
int[] arr = {10, 20, 30, 40};

for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}
```

Enhanced for loop:

```java
for (int value : arr) {
    System.out.println(value);
}
```

Use the normal `for` loop when you need the index.

---

# 4. Find Maximum Element

```java
int max = arr[0];

for (int value : arr) {
    if (value > max) {
        max = value;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 5. Find Minimum Element

```java
int min = arr[0];

for (int value : arr) {
    if (value < min) {
        min = value;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 6. Reverse an Array

Use two pointers.

```java
int left = 0;
int right = arr.length - 1;

while (left < right) {
    int temp = arr[left];
    arr[left] = arr[right];
    arr[right] = temp;

    left++;
    right--;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 7. Check if Array is Sorted

```java
boolean sorted = true;

for (int i = 1; i < arr.length; i++) {
    if (arr[i] < arr[i - 1]) {
        sorted = false;
        break;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 8. Linear Search

```java
int target = 30;
int index = -1;

for (int i = 0; i < arr.length; i++) {
    if (arr[i] == target) {
        index = i;
        break;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 9. Second Largest Element

A common interview problem.

```java
int largest = Integer.MIN_VALUE;
int secondLargest = Integer.MIN_VALUE;

for (int value : arr) {
    if (value > largest) {
        secondLargest = largest;
        largest = value;
    } else if (value > secondLargest && value != largest) {
        secondLargest = value;
    }
}
```

### Important

If the array contains fewer than two distinct values, there may be no valid second-largest element.

---

# 10. Remove Duplicates from Sorted Array

Use the two-pointer technique.

```java
int j = 0;

for (int i = 1; i < arr.length; i++) {
    if (arr[i] != arr[j]) {
        arr[++j] = arr[i];
    }
}

int uniqueLength = j + 1;
```

Example:

```text
Input:
1 1 2 2 3

Output:
1 2 3
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 11. Move Zeroes to the End

```java
int insert = 0;

for (int value : arr) {
    if (value != 0) {
        arr[insert++] = value;
    }
}

while (insert < arr.length) {
    arr[insert++] = 0;
}
```

Example:

```text
Input:
0 1 0 3 12

Output:
1 3 12 0 0
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 12. Two Sum

Given an array and a target, find two numbers whose sum equals the target.

Brute force:

```java
for (int i = 0; i < arr.length; i++) {
    for (int j = i + 1; j < arr.length; j++) {
        if (arr[i] + arr[j] == target) {
            return new int[]{i, j};
        }
    }
}
```

Complexity:

```text
Time:  O(n²)
Space: O(1)
```

Optimized using HashMap:

```java
Map<Integer, Integer> map = new HashMap<>();

for (int i = 0; i < arr.length; i++) {
    int complement = target - arr[i];

    if (map.containsKey(complement)) {
        return new int[]{map.get(complement), i};
    }

    map.put(arr[i], i);
}
```

Complexity:

```text
Time:  O(n)
Space: O(n)
```

### Interview pattern

This is an important example of trading memory for faster lookup.

---

# 13. Maximum Subarray — Kadane's Algorithm

Find the contiguous subarray with the largest sum.

```java
int current = arr[0];
int maximum = arr[0];

for (int i = 1; i < arr.length; i++) {
    current = Math.max(arr[i], current + arr[i]);
    maximum = Math.max(maximum, current);
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

### Key idea

At every position:

```text
Should I:
1. Start a new subarray?
2. Extend the previous subarray?
```

---

# 14. Prefix Sum

Prefix sums allow repeated range-sum queries to be answered efficiently.

Build:

```java
int[] prefix = new int[arr.length];

prefix[0] = arr[0];

for (int i = 1; i < arr.length; i++) {
    prefix[i] = prefix[i - 1] + arr[i];
}
```

Range sum from `left` to `right`:

```java
int sum = prefix[right];

if (left > 0) {
    sum -= prefix[left - 1];
}
```

### Complexity

Building:

```text
O(n)
```

Each range query:

```text
O(1)
```

---

# 15. Prefix Sum with an Extra Zero

A cleaner implementation is:

```java
int[] prefix = new int[arr.length + 1];

for (int i = 0; i < arr.length; i++) {
    prefix[i + 1] = prefix[i] + arr[i];
}
```

Range sum:

```java
int sum = prefix[right + 1] - prefix[left];
```

This version reduces boundary conditions.

---

# 16. Subarray vs Subsequence vs Subset

### Subarray

Elements must be contiguous.

```text
[2, 3, 4]
```

### Subsequence

Elements maintain relative order but do not need to be contiguous.

```text
[2, 4]
```

### Subset

Order generally does not matter.

```text
{2, 4}
```

This distinction appears frequently in interviews.

---

# 17. Rotate Array

Rotate an array to the right by `k` positions.

Efficient approach:

1. Reverse the entire array.
2. Reverse the first `k` elements.
3. Reverse the remaining elements.

Example:

```text
Input:
1 2 3 4 5 6 7
k = 3

Output:
5 6 7 1 2 3 4
```

Implementation:

```java
static void rotate(int[] arr, int k) {
    int n = arr.length;

    k %= n;

    reverse(arr, 0, n - 1);
    reverse(arr, 0, k - 1);
    reverse(arr, k, n - 1);
}

static void reverse(int[] arr, int left, int right) {
    while (left < right) {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;

        left++;
        right--;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 18. Dutch National Flag Problem

Sort an array containing only:

```text
0, 1, 2
```

Use three pointers:

```text
low
mid
high
```

```java
int low = 0;
int mid = 0;
int high = arr.length - 1;

while (mid <= high) {
    if (arr[mid] == 0) {
        swap(arr, low, mid);
        low++;
        mid++;
    } else if (arr[mid] == 1) {
        mid++;
    } else {
        swap(arr, mid, high);
        high--;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 19. Merge Two Sorted Arrays

Two-pointer approach:

```java
int i = 0;
int j = 0;
int k = 0;

while (i < a.length && j < b.length) {
    if (a[i] <= b[j]) {
        result[k++] = a[i++];
    } else {
        result[k++] = b[j++];
    }
}

while (i < a.length) {
    result[k++] = a[i++];
}

while (j < b.length) {
    result[k++] = b[j++];
}
```

### Complexity

```text
Time:  O(n + m)
Space: O(n + m)
```

---

# 20. Best Time to Buy and Sell Stock

Track the minimum price seen so far.

```java
int minPrice = Integer.MAX_VALUE;
int maxProfit = 0;

for (int price : prices) {
    minPrice = Math.min(minPrice, price);
    maxProfit = Math.max(
        maxProfit,
        price - minPrice
    );
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 21. Majority Element

The Boyer-Moore Voting Algorithm finds an element appearing more than `n / 2` times.

```java
int candidate = 0;
int count = 0;

for (int value : arr) {
    if (count == 0) {
        candidate = value;
    }

    count += (value == candidate) ? 1 : -1;
}
```

If the problem guarantees a majority element, `candidate` is the answer.

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 22. Missing Number

For numbers from `0` to `n`, one number is missing.

Using XOR:

```java
int result = arr.length;

for (int i = 0; i < arr.length; i++) {
    result ^= i;
    result ^= arr[i];
}
```

### Why XOR?

Important properties:

```text
x ^ x = 0
x ^ 0 = x
```

Therefore matching values cancel out.

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 23. Find Duplicate Number

A common optimized approach uses Floyd's Cycle Detection algorithm.

The idea is to treat array values as pointers.

```java
int slow = nums[0];
int fast = nums[0];

do {
    slow = nums[slow];
    fast = nums[nums[fast]];
} while (slow != fast);

slow = nums[0];

while (slow != fast) {
    slow = nums[slow];
    fast = nums[fast];
}

return slow;
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

This approach depends on the problem's constraints.

---

# 24. Product of Array Except Self

Calculate the product of every element except the current element without using division.

```java
int n = nums.length;
int[] result = new int[n];

int prefix = 1;

for (int i = 0; i < n; i++) {
    result[i] = prefix;
    prefix *= nums[i];
}

int suffix = 1;

for (int i = n - 1; i >= 0; i--) {
    result[i] *= suffix;
    suffix *= nums[i];
}
```

### Complexity

```text
Time:  O(n)
Space: O(1) extra
```

The returned `result` array itself uses O(n) space.

---

# 25. Trapping Rain Water

This problem tests two pointers and prefix/suffix concepts.

Two-pointer idea:

```text
left
right
leftMax
rightMax
```

At each step, process the side with the smaller maximum boundary.

Typical complexity:

```text
Time:  O(n)
Space: O(1)
```

This is a high-value interview problem.

---

# 26. Sliding Window on Arrays

Sliding window is useful when dealing with contiguous subarrays.

Example: maximum sum of a subarray of size `k`.

```java
int windowSum = 0;

for (int i = 0; i < k; i++) {
    windowSum += arr[i];
}

int maximum = windowSum;

for (int i = k; i < arr.length; i++) {
    windowSum += arr[i];
    windowSum -= arr[i - k];

    maximum = Math.max(maximum, windowSum);
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 27. Binary Search

Binary search works on sorted data.

```java
int left = 0;
int right = arr.length - 1;

while (left <= right) {
    int mid = left + (right - left) / 2;

    if (arr[mid] == target) {
        return mid;
    }

    if (arr[mid] < target) {
        left = mid + 1;
    } else {
        right = mid - 1;
    }
}

return -1;
```

### Complexity

```text
Time:  O(log n)
Space: O(1)
```

### Important

Always use:

```java
left + (right - left) / 2
```

instead of:

```java
(left + right) / 2
```

to avoid integer overflow in extreme cases.

---

# 28. Binary Search on Answer

Some problems do not directly ask you to search an array.

Instead, you binary-search the answer space.

Typical structure:

```java
while (low <= high) {
    int mid = low + (high - low) / 2;

    if (isPossible(mid)) {
        high = mid - 1;
    } else {
        low = mid + 1;
    }
}
```

Recognize this pattern when:

- The answer lies within a numeric range.
- You can efficiently check whether a candidate answer is possible.
- The feasibility condition is monotonic.

---

# 29. Frequency Counting

For small bounded integer values, a frequency array can be faster and simpler than a HashMap.

```java
int[] frequency = new int[101];

for (int value : arr) {
    frequency[value]++;
}
```

Use this when the value range is known and reasonably small.

---

# 30. When to Use HashMap vs Array

### Use an array/frequency array when:

```text
Values are bounded
Indexes can represent values
Memory usage is predictable
```

### Use HashMap when:

```text
Values are large
Values may be negative
Value range is sparse
You need key-value mapping
```

---

# 31. Common Array Patterns

When you see an array problem, ask:

```text
1. Is the array sorted?
   → Binary Search / Two Pointers

2. Do I need fast lookup?
   → HashMap / HashSet

3. Is it about a contiguous subarray?
   → Sliding Window / Prefix Sum / Kadane

4. Are there two ends involved?
   → Two Pointers

5. Is the answer a numeric range?
   → Binary Search on Answer

6. Is there a frequency requirement?
   → HashMap / Frequency Array

7. Is the problem about maximum/minimum subarray?
   → Kadane / Sliding Window

8. Is the array rotated?
   → Two Pointers / Binary Search

9. Are values only 0, 1, 2?
   → Dutch National Flag

10. Is there a repeated/missing value?
    → XOR / Hashing / Cycle Detection
```

---

# 32. Interview Questions

## Easy

1. Find the maximum element in an array.
2. Find the minimum element.
3. Reverse an array.
4. Check whether an array is sorted.
5. Find the sum of all elements.
6. Find the second-largest element.
7. Search for an element.
8. Count even and odd numbers.
9. Move zeroes to the end.
10. Remove duplicates from a sorted array.

## Medium

11. Two Sum.
12. Maximum Subarray.
13. Best Time to Buy and Sell Stock.
14. Rotate Array.
15. Merge Sorted Arrays.
16. Majority Element.
17. Product of Array Except Self.
18. Find Missing Number.
19. Find Duplicate Number.
20. Sort an array containing 0, 1 and 2.
21. Longest consecutive sequence.
22. Subarray with a given sum.
23. Maximum sum subarray of size `k`.
24. Find leaders in an array.
25. Merge overlapping intervals.

## Hard / Advanced

26. Trapping Rain Water.
27. Maximum product subarray.
28. Median of two sorted arrays.
29. First missing positive.
30. Sliding window maximum.
31. Count inversions.
32. Search in a rotated sorted array.
33. Find the peak element.
34. Allocate minimum pages.
35. Split array largest sum.

---

# 33. Java Array Utilities

Useful methods from `java.util.Arrays`:

```java
Arrays.sort(arr);
```

```java
Arrays.binarySearch(arr, target);
```

```java
Arrays.fill(arr, 0);
```

```java
Arrays.copyOf(arr, newLength);
```

```java
Arrays.equals(a, b);
```

```java
Arrays.toString(arr);
```

For 2D arrays:

```java
Arrays.deepToString(matrix);
```

---

# 34. Important Interview Mistakes

### Mistake 1 — Off-by-one errors

Be careful with:

```java
i < arr.length
```

rather than:

```java
i <= arr.length
```

---

### Mistake 2 — Integer overflow

Instead of:

```java
int mid = (left + right) / 2;
```

prefer:

```java
int mid = left + (right - left) / 2;
```

---

### Mistake 3 — Forgetting negative values

Do not initialize a maximum using:

```java
int max = 0;
```

if the array can contain only negative numbers.

Use:

```java
int max = Integer.MIN_VALUE;
```

or initialize from the first element when appropriate.

---

### Mistake 4 — Using extra memory unnecessarily

Before creating another array, ask whether the problem can be solved using:

```text
Two pointers
In-place modification
Prefix/suffix variables
Hashing
```

---

### Mistake 5 — Ignoring constraints

Always check:

```text
n
Value range
Sorted or unsorted
Duplicates
Negative numbers
Required time complexity
Required space complexity
```

Constraints often reveal the intended approach.

---

# 35. Array Problem-Solving Checklist

Before coding:

```text
□ Understand the input
□ Understand the expected output
□ Check constraints
□ Identify whether the array is sorted
□ Look for duplicates
□ Check whether negative values are possible
□ Decide brute force complexity
□ Look for a better pattern
□ Think about edge cases
□ State time and space complexity
```

### Edge cases to test

```text
Empty array
One element
Two elements
All elements equal
Already sorted
Reverse sorted
All negative
Contains zero
Duplicates
Very large values
Very large input
```

---

# 36. Quick Revision

```text
Arrays
│
├── Traversal
├── Searching
│   ├── Linear Search
│   └── Binary Search
│
├── Two Pointers
│   ├── Reverse Array
│   ├── Remove Duplicates
│   ├── Move Zeroes
│   └── Container / Rain Water patterns
│
├── Hashing
│   ├── Two Sum
│   └── Frequency Counting
│
├── Prefix Sum
│   └── Range Queries
│
├── Sliding Window
│   └── Contiguous Subarrays
│
├── Kadane's Algorithm
│   └── Maximum Subarray
│
├── Greedy
│   └── Buy/Sell and interval patterns
│
├── XOR
│   └── Missing Number
│
├── Cycle Detection
│   └── Duplicate Number
│
└── Binary Search on Answer
    └── Optimization Problems
```

---

## Interview Rule

> **Do not jump directly into coding. First identify the pattern. For array problems, the most important patterns are hashing, two pointers, sliding window, prefix sum, binary search and Kadane's algorithm.**
