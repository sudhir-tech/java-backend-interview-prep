# DSA — Monotonic Stack

A Monotonic Stack is a stack whose elements are maintained in a specific order.

It is one of the most useful patterns for problems involving:

- Next Greater Element
- Next Smaller Element
- Previous Greater Element
- Previous Smaller Element
- Daily Temperatures
- Stock Span
- Largest Rectangle in Histogram
- Maximal Rectangle
- Trapping Rain Water
- Sum of Subarray Minimums
- Sum of Subarray Ranges
- Remove K Digits
- Lexicographically smallest subsequences
- Circular arrays
- Contribution technique

The key idea is:

```text
Process each element once
while removing elements that can no longer be useful.
```

With a monotonic stack, many O(n²) problems become:

```text
O(n)
```

---

# 1. What Is a Monotonic Stack?

A normal stack follows:

```text
LIFO
```

A monotonic stack additionally maintains:

```text
increasing order
```

or:

```text
decreasing order
```

Example increasing stack:

```text
1
3
5
8
```

Example decreasing stack:

```text
8
5
3
1
```

The exact direction depends on the problem.

---

# 2. Why Use a Monotonic Stack?

Suppose for every element we need to find:

```text
the next larger element
```

A brute-force approach checks elements to the right:

```text
O(n²)
```

A monotonic stack lets us discard elements that can never become the answer.

Complexity becomes:

```text
O(n)
```

---

# 3. The Most Important Pattern

For each element:

```text
while stack is not empty
and current element makes stack top useless:

    pop stack

process stack top

push current element
```

The `while` loop may look like it makes the algorithm O(n²), but every element is:

```text
pushed once
+
popped at most once
```

Therefore total stack operations are:

```text
O(n)
```

---

# 4. Next Greater Element

Given:

```text
[2, 1, 2, 4, 3]
```

The next greater element for each position is:

```text
[4, 2, 4, -1, -1]
```

For each element, find the first element to its right that is greater.

---

# 5. Brute Force

```java
static int[] nextGreaterBrute(
        int[] nums) {

    int n = nums.length;
    int[] result =
        new int[n];

    Arrays.fill(result, -1);

    for (int i = 0;
         i < n;
         i++) {

        for (int j = i + 1;
             j < n;
             j++) {

            if (nums[j] > nums[i]) {

                result[i] = nums[j];
                break;
            }
        }
    }

    return result;
}
```

Complexity:

```text
O(n²)
```

---

# 6. Next Greater Element — Monotonic Stack

Use a:

```text
decreasing stack of values
```

When the current value is greater than the stack top:

```text
current value is the next greater element
for the stack top.
```

---

# 7. Next Greater Element — Java

```java
static int[] nextGreater(
        int[] nums) {

    int n = nums.length;

    int[] result =
        new int[n];

    Arrays.fill(result, -1);

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = n - 1;
         i >= 0;
         i--) {

        while (!stack.isEmpty()
                && stack.peek() <= nums[i]) {

            stack.pop();
        }

        if (!stack.isEmpty()) {
            result[i] =
                stack.peek();
        }

        stack.push(nums[i]);
    }

    return result;
}
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

---

# 8. Why We Iterate Right to Left

The answer for:

```text
nums[i]
```

is somewhere to the right.

Therefore, by iterating:

```text
right → left
```

the stack already contains useful candidates from the right side.

---

# 9. Next Greater Element Using Indices

Sometimes values are not enough.

We may need:

```text
index
```

instead.

Store indexes in the stack:

```java
Deque<Integer> stack =
    new ArrayDeque<>();
```

Then access:

```java
nums[stack.peek()]
```

This is especially useful for:

```text
distance
width
contribution
```

problems.

---

# 10. Next Greater Element — Index Version

```java
static int[] nextGreaterIndex(
        int[] nums) {

    int n = nums.length;

    int[] result =
        new int[n];

    Arrays.fill(result, -1);

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = n - 1;
         i >= 0;
         i--) {

        while (!stack.isEmpty()
                && nums[stack.peek()]
                    <= nums[i]) {

            stack.pop();
        }

        if (!stack.isEmpty()) {

            result[i] =
                nums[stack.peek()];
        }

        stack.push(i);
    }

    return result;
}
```

---

# 11. Next Greater Element — Left to Right

Another common approach is:

```text
iterate left → right
```

When the current element is greater than the stack top:

```text
current element solves the stack top.
```

This is often more intuitive.

---

# 12. Left-to-Right Template

```java
Deque<Integer> stack =
    new ArrayDeque<>();

for (int i = 0;
     i < nums.length;
     i++) {

    while (!stack.isEmpty()
            && nums[i]
                > nums[stack.peek()]) {

        int index =
            stack.pop();

        result[index] =
            nums[i];
    }

    stack.push(i);
}
```

The stack contains indexes whose answer has not yet been found.

---

# 13. Next Smaller Element

For next smaller:

```text
pop while current <= stack top
```

Template:

```java
while (!stack.isEmpty()
        && nums[stack.peek()]
            >= nums[i]) {

    stack.pop();
}
```

Then:

```text
stack.peek()
```

is the next smaller candidate.

---

# 14. Four Core Monotonic Patterns

| Problem | Stack Type |
|---|---|
| Next Greater | Decreasing |
| Next Smaller | Increasing |
| Previous Greater | Decreasing |
| Previous Smaller | Increasing |

Remember:

```text
Greater → remove smaller/equal candidates
Smaller → remove greater/equal candidates
```

---

# 15. Previous Greater Element

For each element, find the closest greater element to its left.

Example:

```text
nums = [2, 1, 4, 3]
```

Answer:

```text
[-1, 2, -1, 4]
```

---

# 16. Previous Greater — Java

```java
static int[] previousGreater(
        int[] nums) {

    int n = nums.length;

    int[] result =
        new int[n];

    Arrays.fill(result, -1);

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = 0;
         i < n;
         i++) {

        while (!stack.isEmpty()
                && nums[stack.peek()]
                    <= nums[i]) {

            stack.pop();
        }

        if (!stack.isEmpty()) {

            result[i] =
                nums[stack.peek()];
        }

        stack.push(i);
    }

    return result;
}
```

---

# 17. Previous Smaller Element

For each element, find the closest smaller element to the left.

Use an increasing stack.

```java
static int[] previousSmaller(
        int[] nums) {

    int n = nums.length;

    int[] result =
        new int[n];

    Arrays.fill(result, -1);

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = 0;
         i < n;
         i++) {

        while (!stack.isEmpty()
                && nums[stack.peek()]
                    >= nums[i]) {

            stack.pop();
        }

        if (!stack.isEmpty()) {

            result[i] =
                nums[stack.peek()];
        }

        stack.push(i);
    }

    return result;
}
```

---

# 18. Daily Temperatures

Given daily temperatures, find how many days you must wait until a warmer temperature.

Example:

```text
[73,74,75,71,69,72,76,73]
```

Answer:

```text
[1,1,4,2,1,1,0,0]
```

---

# 19. Daily Temperatures — Idea

Store indexes of temperatures that are waiting for a warmer day.

When:

```text
temperature[i] > temperature[stack.peek()]
```

the current day solves the previous day.

Distance:

```text
i - previousIndex
```

---

# 20. Daily Temperatures — Java

```java
static int[] dailyTemperatures(
        int[] temperatures) {

    int n =
        temperatures.length;

    int[] result =
        new int[n];

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = 0;
         i < n;
         i++) {

        while (!stack.isEmpty()
                && temperatures[i]
                    > temperatures[stack.peek()]) {

            int previous =
                stack.pop();

            result[previous] =
                i - previous;
        }

        stack.push(i);
    }

    return result;
}
```

Complexity:

```text
O(n)
```

---

# 21. Stock Span

For each day, determine how many consecutive previous days had a price less than or equal to today's price.

Example:

```text
[100,80,60,70,60,75,85]
```

Answer:

```text
[1,1,1,2,1,4,6]
```

---

# 22. Stock Span — Java

```java
static int[] stockSpan(
        int[] prices) {

    int n = prices.length;

    int[] result =
        new int[n];

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = 0;
         i < n;
         i++) {

        while (!stack.isEmpty()
                && prices[stack.peek()]
                    <= prices[i]) {

            stack.pop();
        }

        if (stack.isEmpty()) {

            result[i] = i + 1;

        } else {

            result[i] =
                i - stack.peek();
        }

        stack.push(i);
    }

    return result;
}
```

---

# 23. Largest Rectangle in Histogram

Given:

```text
heights = [2,1,5,6,2,3]
```

find the largest rectangular area.

Answer:

```text
10
```

The rectangle:

```text
5 × 2
```

uses heights:

```text
5, 6
```

---

# 24. Why Histogram Is Difficult

For every bar, we need to know:

```text
nearest smaller element on left
nearest smaller element on right
```

Then:

```text
width =
rightSmaller - leftSmaller - 1
```

Area:

```text
height × width
```

A monotonic increasing stack solves this in O(n).

---

# 25. Largest Rectangle — Java

```java
static int largestRectangleArea(
        int[] heights) {

    int n = heights.length;

    Deque<Integer> stack =
        new ArrayDeque<>();

    int maxArea = 0;

    for (int i = 0;
         i <= n;
         i++) {

        int current =
            i == n
                ? 0
                : heights[i];

        while (!stack.isEmpty()
                && current
                    < heights[stack.peek()]) {

            int height =
                heights[stack.pop()];

            int leftBoundary =
                stack.isEmpty()
                    ? -1
                    : stack.peek();

            int width =
                i - leftBoundary - 1;

            maxArea =
                Math.max(
                    maxArea,
                    height * width
                );
        }

        stack.push(i);
    }

    return maxArea;
}
```

---

# 26. Histogram Stack Type

For largest rectangle:

```text
increasing stack
```

We pop when:

```text
current < stack top
```

The popped height has just discovered its:

```text
first smaller element on the right.
```

The remaining stack top gives:

```text
nearest smaller element on the left.
```

---

# 27. Sentinel Technique

A useful trick is to process one extra element:

```text
height = 0
```

after the array.

This forces all remaining stack elements to be popped.

Instead of writing a separate cleanup loop:

```java
for (int i = 0;
     i <= n;
     i++)
```

use:

```java
int current =
    i == n ? 0 : heights[i];
```

---

# 28. Maximal Rectangle

Given a binary matrix, find the largest rectangle containing only `1`s.

Convert every row into a histogram.

For each row:

```text
height[j] += 1
```

if the cell is `1`.

Otherwise:

```text
height[j] = 0
```

Then apply:

```text
Largest Rectangle in Histogram
```

---

# 29. Maximal Rectangle — Java

```java
static int maximalRectangle(
        char[][] matrix) {

    if (matrix.length == 0) {
        return 0;
    }

    int cols =
        matrix[0].length;

    int[] heights =
        new int[cols];

    int maxArea = 0;

    for (char[] row : matrix) {

        for (int j = 0;
             j < cols;
             j++) {

            if (row[j] == '1') {

                heights[j]++;

            } else {

                heights[j] = 0;
            }
        }

        maxArea =
            Math.max(
                maxArea,
                largestRectangleArea(
                    heights
                )
            );
    }

    return maxArea;
}
```

Complexity:

```text
O(rows × cols)
```

---

# 30. Trapping Rain Water

Given:

```text
[0,1,0,2,1,0,1,3,2,1,2,1]
```

find how much water can be trapped.

A monotonic stack can solve this in:

```text
O(n)
```

---

# 31. Trapping Water — Stack Idea

Maintain a:

```text
decreasing stack of indexes
```

When the current height is greater than the stack top:

```text
a valley has been closed.
```

Pop the valley.

Then determine:

```text
left boundary
right boundary
width
bounded height
```

---

# 32. Trapping Rain Water — Java

```java
static int trap(
        int[] height) {

    Deque<Integer> stack =
        new ArrayDeque<>();

    int water = 0;

    for (int i = 0;
         i < height.length;
         i++) {

        while (!stack.isEmpty()
                && height[i]
                    > height[stack.peek()]) {

            int bottom =
                stack.pop();

            if (stack.isEmpty()) {
                break;
            }

            int left =
                stack.peek();

            int width =
                i - left - 1;

            int boundedHeight =
                Math.min(
                    height[left],
                    height[i]
                ) - height[bottom];

            water +=
                width * boundedHeight;
        }

        stack.push(i);
    }

    return water;
}
```

---

# 33. Sum of Subarray Minimums

For every subarray, consider its minimum.

Find:

```text
sum of all subarray minimums.
```

A brute-force solution is too slow.

Monotonic stacks allow each element's contribution to be calculated.

---

# 34. Contribution Technique

For an element:

```text
nums[i]
```

find:

```text
number of choices on the left
×
number of choices on the right
```

for which it is the minimum.

If:

```text
leftCount
rightCount
```

then contribution is:

```text
nums[i]
× leftCount
× rightCount
```

---

# 35. Sum of Subarray Minimums — Java

```java
static int sumSubarrayMins(
        int[] arr) {

    final long MOD =
        1_000_000_007L;

    int n = arr.length;

    int[] left =
        new int[n];

    int[] right =
        new int[n];

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = 0;
         i < n;
         i++) {

        while (!stack.isEmpty()
                && arr[stack.peek()]
                    > arr[i]) {

            stack.pop();
        }

        left[i] =
            stack.isEmpty()
                ? i + 1
                : i - stack.peek();

        stack.push(i);
    }

    stack.clear();

    for (int i = n - 1;
         i >= 0;
         i--) {

        while (!stack.isEmpty()
                && arr[stack.peek()]
                    >= arr[i]) {

            stack.pop();
        }

        right[i] =
            stack.isEmpty()
                ? n - i
                : stack.peek() - i;

        stack.push(i);
    }

    long answer = 0;

    for (int i = 0;
         i < n;
         i++) {

        answer =
            (answer
                + (long) arr[i]
                * left[i]
                % MOD
                * right[i]
                % MOD)
            % MOD;
    }

    return (int) answer;
}
```

Notice the asymmetric comparisons:

```text
left  → >
right → >=
```

This avoids double-counting equal values.

---

# 36. Why Handle Duplicates Carefully?

Suppose:

```text
[2, 2]
```

Both elements can potentially be the minimum of the same subarray.

We need a consistent ownership rule.

For example:

```text
left uses >
right uses >=
```

assigns equal-value ranges consistently.

This is a very important monotonic-stack detail.

---

# 37. Sum of Subarray Maximums

The same contribution technique works for maximums.

Reverse the comparisons.

For example:

```text
previous greater
+
next greater
```

Then:

```text
contribution =
value
× left choices
× right choices
```

---

# 38. Sum of Subarray Ranges

For every subarray:

```text
range =
maximum - minimum
```

Therefore:

```text
sum of ranges
=
sum of subarray maximums
-
sum of subarray minimums
```

Use:

```text
monotonic stack
```

for both parts.

---

# 39. Remove K Digits

Given a number represented as a string, remove exactly `k` digits to produce the smallest possible number.

Greedy idea:

```text
If the current digit is smaller than
the previous digit, remove the previous digit.
```

This uses an increasing monotonic stack.

---

# 40. Remove K Digits — Java

```java
static String removeKdigits(
        String num,
        int k) {

    Deque<Character> stack =
        new ArrayDeque<>();

    for (char digit :
            num.toCharArray()) {

        while (!stack.isEmpty()
                && k > 0
                && stack.peekLast()
                    > digit) {

            stack.removeLast();
            k--;
        }

        stack.addLast(digit);
    }

    while (k > 0
            && !stack.isEmpty()) {

        stack.removeLast();
        k--;
    }

    StringBuilder result =
        new StringBuilder();

    boolean leadingZero = true;

    for (char digit : stack) {

        if (leadingZero
                && digit == '0') {
            continue;
        }

        leadingZero = false;
        result.append(digit);
    }

    return result.length() == 0
        ? "0"
        : result.toString();
}
```

---

# 41. Lexicographically Smallest Subsequence

Monotonic stacks can also maintain a smallest subsequence.

Typical ingredients:

```text
stack
+
remaining frequency
+
visited/used state
```

When the current character is smaller than the stack top, remove the top if that character appears again later.

This pattern is useful for:

```text
Remove Duplicate Letters
Smallest Subsequence of Distinct Characters
```

---

# 42. Remove Duplicate Letters

Goal:

```text
Return the lexicographically smallest string
containing every distinct character exactly once.
```

Use:

```text
frequency count
+
used array
+
monotonic stack
```

---

# 43. Remove Duplicate Letters — Java

```java
static String removeDuplicateLetters(
        String s) {

    int[] count =
        new int[26];

    for (char c : s.toCharArray()) {
        count[c - 'a']++;
    }

    boolean[] used =
        new boolean[26];

    Deque<Character> stack =
        new ArrayDeque<>();

    for (char c : s.toCharArray()) {

        int index =
            c - 'a';

        count[index]--;

        if (used[index]) {
            continue;
        }

        while (!stack.isEmpty()
                && stack.peekLast() > c
                && count[
                    stack.peekLast() - 'a'
                ] > 0) {

            char removed =
                stack.removeLast();

            used[
                removed - 'a'
            ] = false;
        }

        stack.addLast(c);
        used[index] = true;
    }

    StringBuilder result =
        new StringBuilder();

    for (char c : stack) {
        result.append(c);
    }

    return result.toString();
}
```

---

# 44. Circular Array

For circular arrays, an element can see elements after the end.

Example:

```text
[1, 2, 1]
```

The next greater element for the last `1` is:

```text
2
```

even though `2` appears before it physically.

---

# 45. Circular Next Greater Element

A common trick:

```text
process the array twice.
```

Instead of actually duplicating it:

```java
i % n
```

gives the circular index.

---

# 46. Circular Next Greater — Java

```java
static int[] nextGreaterElements(
        int[] nums) {

    int n = nums.length;

    int[] result =
        new int[n];

    Arrays.fill(result, -1);

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = 2 * n - 1;
         i >= 0;
         i--) {

        int index = i % n;

        while (!stack.isEmpty()
                && nums[stack.peek()]
                    <= nums[index]) {

            stack.pop();
        }

        if (i < n
                && !stack.isEmpty()) {

            result[index] =
                nums[stack.peek()];
        }

        stack.push(index);
    }

    return result;
}
```

---

# 47. Monotonic Deque vs Monotonic Stack

A monotonic stack is useful when:

```text
we process elements once
and need nearest greater/smaller relationships.
```

A monotonic deque is useful when:

```text
we need the best value inside
a sliding window.
```

Example:

```text
Sliding Window Maximum
```

usually uses a monotonic deque.

---

# 48. Sliding Window Maximum

For every window of size `k`, find the maximum.

Maintain a:

```text
decreasing deque of indexes.
```

The front always contains the maximum for the current window.

---

# 49. Sliding Window Maximum — Java

```java
static int[] maxSlidingWindow(
        int[] nums,
        int k) {

    int n = nums.length;

    int[] result =
        new int[n - k + 1];

    Deque<Integer> deque =
        new ArrayDeque<>();

    for (int i = 0;
         i < n;
         i++) {

        while (!deque.isEmpty()
                && deque.peekFirst()
                    <= i - k) {

            deque.pollFirst();
        }

        while (!deque.isEmpty()
                && nums[deque.peekLast()]
                    <= nums[i]) {

            deque.pollLast();
        }

        deque.offerLast(i);

        if (i >= k - 1) {

            result[i - k + 1] =
                nums[deque.peekFirst()];
        }
    }

    return result;
}
```

---

# 50. Stack of Values vs Stack of Indexes

Use values when you only need:

```text
the next greater/smaller value.
```

Use indexes when you need:

```text
distance
width
boundaries
contribution
position
```

In advanced problems, indexes are usually more flexible.

---

# 51. Choosing Stack Direction

### Increasing Stack

Values from bottom to top:

```text
small → large
```

Useful for:

```text
next smaller
previous smaller
largest rectangle
```

### Decreasing Stack

Values from bottom to top:

```text
large → small
```

Useful for:

```text
next greater
previous greater
```

---

# 52. Monotonic Stack Template — Next Greater

```java
Deque<Integer> stack =
    new ArrayDeque<>();

for (int i = n - 1;
     i >= 0;
     i--) {

    while (!stack.isEmpty()
            && nums[stack.peek()]
                <= nums[i]) {

        stack.pop();
    }

    // stack.peek() is the answer

    stack.push(i);
}
```

---

# 53. Monotonic Stack Template — Next Smaller

```java
Deque<Integer> stack =
    new ArrayDeque<>();

for (int i = n - 1;
     i >= 0;
     i--) {

    while (!stack.isEmpty()
            && nums[stack.peek()]
                >= nums[i]) {

        stack.pop();
    }

    // stack.peek() is the answer

    stack.push(i);
}
```

---

# 54. Monotonic Stack Template — Resolve on Current Element

Sometimes iterate left to right:

```java
for (int i = 0;
     i < n;
     i++) {

    while (!stack.isEmpty()
            && nums[i]
                > nums[stack.peek()]) {

        int index =
            stack.pop();

        result[index] =
            nums[i];
    }

    stack.push(i);
}
```

Think:

```text
Current element resolves
previous unresolved elements.
```

---

# 55. Largest Rectangle Pattern

When you see:

```text
histogram
rectangle
nearest smaller
maximum area
```

think:

```text
Monotonic Increasing Stack
```

Formula:

```text
area =
height
×
(right smaller - left smaller - 1)
```

---

# 56. Contribution Pattern

When you see:

```text
sum of subarray minimums
sum of subarray maximums
sum of ranges
```

think:

```text
How many subarrays use this element
as the minimum/maximum?
```

Then:

```text
contribution =
value
× left choices
× right choices
```

---

# 57. Common Monotonic Stack Mistakes

### Mistake 1 — Wrong comparison

For duplicates, decide carefully between:

```text
>
>=
<
<=
```

### Mistake 2 — Storing values when indexes are required

Distances and widths require indexes.

### Mistake 3 — Forgetting circular behavior

Use:

```text
2 * n
```

or another circular strategy.

### Mistake 4 — Forgetting remaining stack elements

Use:

```text
sentinel
```

or a final cleanup loop.

### Mistake 5 — Assuming every stack problem is monotonic

The stack must maintain a meaningful invariant.

---

# 58. Complexity Proof

For a monotonic stack:

```text
Each element is pushed at most once.
Each element is popped at most once.
```

Therefore:

```text
Total pushes = O(n)
Total pops   = O(n)
```

So:

```text
Total time = O(n)
```

This amortized-analysis idea is important in interviews.

---

# 59. Edge Cases

Test:

```text
Empty array
One element
Strictly increasing
Strictly decreasing
All equal
Duplicate values
Negative values
Large values
Circular arrays
No greater element
No smaller element
```

---

# 60. Interview Questions — Easy

1. Next Greater Element.
2. Next Smaller Element.
3. Previous Greater Element.
4. Previous Smaller Element.
5. Daily Temperatures.
6. Stock Span.

---

# 61. Interview Questions — Medium

7. Circular Next Greater Element.
8. Largest Rectangle in Histogram.
9. Trapping Rain Water.
10. Remove K Digits.
11. Remove Duplicate Letters.
12. Sliding Window Maximum.
13. Sum of Subarray Minimums.

---

# 62. Interview Questions — Advanced

14. Maximal Rectangle.
15. Sum of Subarray Ranges.
16. Constrained Subsequence Sum.
17. Maximum of Minimum for Every Window Size.
18. Online Stock Span.
19. Lexicographically smallest subsequence.
20. Advanced contribution problems.

---

# 63. Pattern Recognition

When you see:

```text
next greater
```

think:

```text
decreasing stack
```

When you see:

```text
next smaller
```

think:

```text
increasing stack
```

When you see:

```text
nearest greater/smaller
```

think:

```text
monotonic stack
```

When you see:

```text
largest rectangle
```

think:

```text
increasing stack
```

When you see:

```text
subarray minimum/maximum contribution
```

think:

```text
monotonic stack + contribution
```

---

# 64. Monotonic Stack Decision Tree

```text
Need nearest element?
        |
       Yes
        ↓
Greater or smaller?
        |
   +----+----+
   |         |
Greater    Smaller
   |         |
Decreasing Increasing
 Stack       Stack

Need subarray min/max contribution?
        |
       Yes
        ↓
Monotonic Stack

Need maximum/minimum in a sliding window?
        |
       Yes
        ↓
Monotonic Deque
```

---

# 65. Quick Revision

```text
Monotonic Stack
│
├── Next
│   ├── Greater
│   └── Smaller
│
├── Previous
│   ├── Greater
│   └── Smaller
│
├── Classic Problems
│   ├── Daily Temperatures
│   ├── Stock Span
│   ├── Histogram
│   ├── Trapping Rain Water
│   └── Remove K Digits
│
├── Contribution
│   ├── Subarray Minimums
│   ├── Subarray Maximums
│   └── Subarray Ranges
│
└── Advanced
    ├── Circular Arrays
    ├── Maximal Rectangle
    └── Monotonic Deque
```

---

# 66. Most Important Templates

### Next Greater

```java
while (!stack.isEmpty()
        && nums[stack.peek()]
            <= nums[i]) {

    stack.pop();
}
```

### Next Smaller

```java
while (!stack.isEmpty()
        && nums[stack.peek()]
            >= nums[i]) {

    stack.pop();
}
```

### Resolve Previous Elements

```java
while (!stack.isEmpty()
        && current
            > nums[stack.peek()]) {

    int index = stack.pop();

    result[index] = current;
}
```

---

# 67. Interview Explanation Template

For a monotonic-stack problem:

```text
A brute-force solution would compare each element
with many elements to its left/right, giving O(n²).

Instead, I maintain a monotonic stack containing
only candidates that can still be useful.

Whenever the current element makes the top of the
stack impossible as a future candidate, I remove it.

Each element is pushed once and popped at most once,
so the total complexity is O(n).
```

---

# 68. Final Interview Rule

> **When a problem asks for the nearest greater/smaller element, or repeatedly asks for boundaries around an element, immediately consider a monotonic stack. The key is to maintain the correct increasing or decreasing invariant.**
