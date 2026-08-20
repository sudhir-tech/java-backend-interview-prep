# DSA — Two Pointers

The **Two Pointers** technique is one of the most important patterns for Java backend interviews.

Instead of using nested loops, we maintain two indexes/pointers and move them according to the problem's conditions.

Typical pointer names:

```text
left
right
```

The technique is especially useful for:

- Sorted arrays
- Sorted strings
- Pair problems
- Palindromes
- Removing duplicates
- Partitioning
- Merging
- Subarray/range problems
- Container and water problems

---

# 1. What is the Two Pointers Technique?

The basic idea is to maintain two positions in a data structure.

Example:

```java
int left = 0;
int right = arr.length - 1;

while (left < right) {

    // Process arr[left] and arr[right]

    left++;
    right--;
}
```

Instead of checking every pair:

```text
O(n²)
```

we can often solve the problem in:

```text
O(n)
```

---

# 2. When Should You Think About Two Pointers?

Look for these clues:

```text
1. The array is sorted.
2. You need to find a pair.
3. You are comparing both ends.
4. You need to reverse something.
5. You need to remove duplicates in-place.
6. You need to partition an array.
7. You need to merge two sorted arrays.
8. You are checking for a palindrome.
9. You need to shrink or expand a range.
```

---

# 3. Pattern 1 — Opposite Direction Pointers

The pointers start at opposite ends:

```text
left → → → 
         ← ← ← right
```

Example:

```java
int left = 0;
int right = arr.length - 1;

while (left < right) {

    // Process

    left++;
    right--;
}
```

Common problems:

```text
Reverse Array
Palindrome
Two Sum in Sorted Array
Container With Most Water
```

---

# 4. Reverse an Array

Example:

```text
Input:
1 2 3 4 5

Output:
5 4 3 2 1
```

Solution:

```java
static void reverse(int[] arr) {

    int left = 0;
    int right = arr.length - 1;

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

# 5. Palindrome Check

A palindrome reads the same from both directions.

Example:

```text
madam
```

Use two pointers:

```java
static boolean isPalindrome(String s) {

    int left = 0;
    int right = s.length() - 1;

    while (left < right) {

        if (s.charAt(left)
                != s.charAt(right)) {

            return false;
        }

        left++;
        right--;
    }

    return true;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 6. Valid Palindrome with Non-Alphanumeric Characters

Example:

```text
"A man, a plan, a canal: Panama"
```

We can ignore spaces and punctuation.

```java
static boolean isValidPalindrome(String s) {

    int left = 0;
    int right = s.length() - 1;

    while (left < right) {

        while (left < right
                && !Character.isLetterOrDigit(
                    s.charAt(left))) {

            left++;
        }

        while (left < right
                && !Character.isLetterOrDigit(
                    s.charAt(right))) {

            right--;
        }

        if (Character.toLowerCase(
                s.charAt(left))
                != Character.toLowerCase(
                    s.charAt(right))) {

            return false;
        }

        left++;
        right--;
    }

    return true;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 7. Two Sum in a Sorted Array

Given:

```text
numbers = [2, 7, 11, 15]
target = 9
```

Answer:

```text
2 + 7 = 9
```

Because the array is sorted, use two pointers.

```java
static int[] twoSumSorted(
        int[] numbers,
        int target) {

    int left = 0;
    int right = numbers.length - 1;

    while (left < right) {

        int sum =
            numbers[left] + numbers[right];

        if (sum == target) {
            return new int[]{
                left,
                right
            };
        }

        if (sum < target) {
            left++;
        } else {
            right--;
        }
    }

    return new int[]{-1, -1};
}
```

### Why does it work?

If:

```text
sum < target
```

we need a larger value.

Move:

```text
left++
```

If:

```text
sum > target
```

we need a smaller value.

Move:

```text
right--
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 8. Why Sorting Matters

Two pointers are especially powerful when the data is sorted.

Example:

```text
1 2 4 6 8 10
```

If:

```text
1 + 10 > target
```

we know we can safely move the right pointer because every value after `right` would be even larger.

Sorting provides this monotonic behavior.

---

# 9. Two Sum — Unsorted Array

For an unsorted array:

```text
[3, 2, 4]
```

the two-pointer approach does not directly apply unless we sort.

Common approach:

```text
HashMap → O(n)
```

or:

```text
Sort + Two Pointers → O(n log n)
```

If original indexes are required, HashMap is usually preferable.

---

# 10. Remove Duplicates from Sorted Array

Example:

```text
Input:
1 1 2 2 3

Output:
1 2 3
```

Use:

```text
slow pointer
fast pointer
```

```java
static int removeDuplicates(int[] nums) {

    if (nums.length == 0) {
        return 0;
    }

    int slow = 0;

    for (int fast = 1;
         fast < nums.length;
         fast++) {

        if (nums[fast] != nums[slow]) {

            slow++;

            nums[slow] =
                nums[fast];
        }
    }

    return slow + 1;
}
```

### Pointer meaning

```text
slow → position of last unique element
fast → scans the array
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 11. Move Zeroes

Example:

```text
Input:
0 1 0 3 12

Output:
1 3 12 0 0
```

Use a write pointer.

```java
static void moveZeroes(int[] nums) {

    int insert = 0;

    for (int value : nums) {

        if (value != 0) {
            nums[insert++] = value;
        }
    }

    while (insert < nums.length) {
        nums[insert++] = 0;
    }
}
```

This is a variation of the slow/fast pointer pattern.

---

# 12. Partition Array by a Condition

A pointer can track where the next valid element should go.

Example:

```text
Move negative numbers before positive numbers.
```

Conceptually:

```java
int left = 0;

for (int right = 0;
     right < nums.length;
     right++) {

    if (nums[right] < 0) {

        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;

        left++;
    }
}
```

The exact implementation depends on whether the problem requires stable ordering.

---

# 13. Dutch National Flag

Sort:

```text
0, 1, 2
```

using three pointers:

```text
low
mid
high
```

```java
static void sortColors(int[] nums) {

    int low = 0;
    int mid = 0;
    int high = nums.length - 1;

    while (mid <= high) {

        if (nums[mid] == 0) {

            swap(nums, low, mid);

            low++;
            mid++;

        } else if (nums[mid] == 1) {

            mid++;

        } else {

            swap(nums, mid, high);

            high--;
        }
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 14. Merge Two Sorted Arrays

Two pointers can merge two sorted arrays efficiently.

```java
static int[] merge(
        int[] a,
        int[] b) {

    int[] result =
        new int[a.length + b.length];

    int i = 0;
    int j = 0;
    int k = 0;

    while (i < a.length
            && j < b.length) {

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

    return result;
}
```

### Complexity

```text
Time:  O(n + m)
Space: O(n + m)
```

---

# 15. Intersection of Two Sorted Arrays

Example:

```text
A = [1, 2, 2, 3, 4]
B = [2, 2, 4, 6]
```

Use two pointers:

```java
int i = 0;
int j = 0;

while (i < a.length
        && j < b.length) {

    if (a[i] == b[j]) {

        result.add(a[i]);

        i++;
        j++;

    } else if (a[i] < b[j]) {

        i++;

    } else {

        j++;
    }
}
```

### Complexity

```text
Time:  O(n + m)
Space: depends on output
```

---

# 16. Union of Two Sorted Arrays

Again use two pointers.

The idea is:

```text
Compare current elements.
Take the smaller one.
If equal, take once and advance both.
```

Be careful to avoid duplicate output.

---

# 17. Container With Most Water

Given heights:

```text
[1,8,6,2,5,4,8,3,7]
```

Choose two lines that contain the maximum amount of water.

Area:

```text
width × minimum(height[left], height[right])
```

Solution:

```java
static int maxArea(int[] height) {

    int left = 0;
    int right = height.length - 1;

    int maximum = 0;

    while (left < right) {

        int width = right - left;

        int currentHeight =
            Math.min(
                height[left],
                height[right]
            );

        int area =
            width * currentHeight;

        maximum =
            Math.max(maximum, area);

        if (height[left]
                < height[right]) {

            left++;

        } else {

            right--;
        }
    }

    return maximum;
}
```

### Key idea

Move the pointer at the shorter height.

Why?

The area is limited by the shorter side.

Moving the taller side cannot increase the limiting height while width decreases.

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 18. Trapping Rain Water

This is a high-value two-pointer problem.

We maintain:

```text
left
right
leftMax
rightMax
```

```java
static int trap(int[] height) {

    int left = 0;
    int right = height.length - 1;

    int leftMax = 0;
    int rightMax = 0;

    int water = 0;

    while (left < right) {

        if (height[left]
                <= height[right]) {

            if (height[left] >= leftMax) {

                leftMax = height[left];

            } else {

                water +=
                    leftMax - height[left];
            }

            left++;

        } else {

            if (height[right] >= rightMax) {

                rightMax = height[right];

            } else {

                water +=
                    rightMax - height[right];
            }

            right--;
        }
    }

    return water;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 19. Three Sum

Problem:

> Find all unique triplets whose sum is zero.

Example:

```text
[-1, 0, 1, 2, -1, -4]
```

Answer:

```text
[-1, -1, 2]
[-1, 0, 1]
```

First sort the array:

```java
Arrays.sort(nums);
```

Then fix one element and use two pointers.

```java
static List<List<Integer>> threeSum(
        int[] nums) {

    List<List<Integer>> result =
        new ArrayList<>();

    Arrays.sort(nums);

    for (int i = 0;
         i < nums.length - 2;
         i++) {

        if (i > 0
                && nums[i] == nums[i - 1]) {
            continue;
        }

        int left = i + 1;
        int right = nums.length - 1;

        while (left < right) {

            long sum =
                (long) nums[i]
                + nums[left]
                + nums[right];

            if (sum == 0) {

                result.add(
                    Arrays.asList(
                        nums[i],
                        nums[left],
                        nums[right]
                    )
                );

                left++;
                right--;

                while (left < right
                        && nums[left]
                            == nums[left - 1]) {
                    left++;
                }

                while (left < right
                        && nums[right]
                            == nums[right + 1]) {
                    right--;
                }

            } else if (sum < 0) {

                left++;

            } else {

                right--;
            }
        }
    }

    return result;
}
```

### Complexity

```text
Sorting: O(n log n)
Two-pointer scan: O(n²)

Overall: O(n²)
```

Extra space excluding output depends on the sorting implementation.

---

# 20. Four Sum

The same idea can be extended.

Pattern:

```text
Sort
↓
Fix first element
↓
Fix second element
↓
Two pointers for remaining two
```

Typical complexity:

```text
O(n³)
```

This is a useful extension of the Three Sum pattern.

---

# 21. Pair With Given Difference

For a sorted array, two pointers can find whether two values have difference `k`.

```java
int i = 0;
int j = 1;

while (j < nums.length) {

    int difference =
        nums[j] - nums[i];

    if (difference == k) {
        return true;
    }

    if (difference < k) {
        j++;
    } else {
        i++;
    }

    if (i == j) {
        j++;
    }
}

return false;
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 22. Squares of a Sorted Array

Given a sorted array that may contain negative numbers:

```text
[-4, -1, 0, 3, 10]
```

Output:

```text
[0, 1, 9, 16, 100]
```

The largest square must come from one of the ends.

Use two pointers and fill the result from the back.

```java
static int[] sortedSquares(
        int[] nums) {

    int[] result =
        new int[nums.length];

    int left = 0;
    int right = nums.length - 1;

    for (int index = nums.length - 1;
         index >= 0;
         index--) {

        int leftSquare =
            nums[left] * nums[left];

        int rightSquare =
            nums[right] * nums[right];

        if (leftSquare > rightSquare) {

            result[index] = leftSquare;
            left++;

        } else {

            result[index] = rightSquare;
            right--;
        }
    }

    return result;
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 23. Compare Strings with Backspaces

A two-pointer approach can compare two strings while treating:

```text
#
```

as backspace.

Instead of building new strings, process each string from right to left.

This gives:

```text
Time:  O(n + m)
Space: O(1)
```

This is a useful example of using pointers to avoid extra memory.

---

# 24. Remove Element In-Place

Given:

```text
nums = [3, 2, 2, 3]
value = 3
```

Result:

```text
[2, 2]
```

Use a write pointer:

```java
static int removeElement(
        int[] nums,
        int value) {

    int write = 0;

    for (int read = 0;
         read < nums.length;
         read++) {

        if (nums[read] != value) {
            nums[write++] = nums[read];
        }
    }

    return write;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 25. Slow and Fast Pointers

Not all two-pointer problems use opposite ends.

Sometimes both pointers move in the same direction:

```text
slow →
fast → → →
```

This is called the **slow/fast pointer pattern**.

Common uses:

```text
Remove duplicates
Move zeroes
Linked-list cycle detection
Find middle of linked list
Partitioning
```

---

# 26. Fast and Slow Pointer — Linked List

Although this is a linked-list problem, the same pointer concept applies.

Find the middle:

```java
ListNode slow = head;
ListNode fast = head;

while (fast != null
        && fast.next != null) {

    slow = slow.next;
    fast = fast.next.next;
}
```

At the end:

```text
slow → middle
fast → end
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 27. Detect Cycle in Linked List

Floyd's Cycle Detection:

```java
ListNode slow = head;
ListNode fast = head;

while (fast != null
        && fast.next != null) {

    slow = slow.next;
    fast = fast.next.next;

    if (slow == fast) {
        return true;
    }
}

return false;
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

This is also called the **Tortoise and Hare algorithm**.

---

# 28. Find Cycle Entry

After detecting a meeting point:

```java
slow = head;

while (slow != fast) {
    slow = slow.next;
    fast = fast.next;
}

return slow;
```

This returns the cycle's starting node.

---

# 29. Sliding Window vs Two Pointers

These patterns are related but not identical.

### Two pointers

Usually focuses on:

```text
left
right
```

and moving them according to comparisons or structural conditions.

Examples:

```text
Two Sum Sorted
Palindrome
Container With Most Water
Three Sum
```

### Sliding window

Maintains a valid/invalid contiguous range.

Examples:

```text
Longest Substring
Maximum Sum Window
Minimum Window
At Most K Distinct
```

A sliding window is often implemented using two pointers.

---

# 30. Two Pointers vs Hashing

### Two pointers

Best when:

```text
Array is sorted
In-place solution is possible
You need O(1) extra space
```

### Hashing

Best when:

```text
Array is unsorted
You need fast lookup
Original positions matter
```

Example:

```text
Two Sum unsorted → HashMap

Two Sum sorted → Two Pointers
```

---

# 31. Sorting + Two Pointers

A very common interview pattern is:

```text
Unsorted array
      ↓
    Sort
      ↓
Two Pointers
```

Examples:

```text
Three Sum
Four Sum
Pair with difference
Closest pair
```

Be aware that sorting changes the original order.

If original indexes are required, preserve them or use another approach.

---

# 32. Closest Pair to Target

For a sorted array, use:

```text
left
right
```

Calculate:

```text
currentSum
```

and track the closest result.

If:

```text
currentSum < target
```

move:

```text
left++
```

Otherwise:

```text
right--
```

This is another example of exploiting sorted order.

---

# 33. Minimum Difference Pair

After sorting:

```text
[1, 3, 7, 8, 10]
```

the minimum absolute difference must occur between adjacent elements.

```java
Arrays.sort(nums);

int minimum =
    Integer.MAX_VALUE;

for (int i = 1;
     i < nums.length;
     i++) {

    minimum = Math.min(
        minimum,
        nums[i] - nums[i - 1]
    );
}
```

### Complexity

```text
Time: O(n log n)
Space: depends on sorting
```

---

# 34. Two Pointers with Duplicates

When finding unique pairs/triplets, duplicates must be handled carefully.

Example:

```text
[-1, -1, 0, 1, 1]
```

After finding a valid pair, skip duplicates:

```java
while (left < right
        && nums[left]
            == nums[left + 1]) {
    left++;
}

while (left < right
        && nums[right]
            == nums[right - 1]) {
    right--;
}
```

This prevents duplicate results.

---

# 35. Important Pointer Invariants

An **invariant** is something that remains true while the algorithm runs.

For Two Sum on a sorted array:

```text
All pairs outside the current [left, right]
have already been ruled out.
```

For remove duplicates:

```text
[0 ... slow]
contains the processed unique values.
```

For sliding window:

```text
[left ... right]
is maintained according to the window condition.
```

Thinking in invariants makes pointer algorithms easier to design and explain.

---

# 36. Common Two-Pointer Patterns

```text
Pattern 1:
left →       ← right

Pattern 2:
slow →
fast → → →

Pattern 3:
fixed i
   ↓
left →       ← right

Pattern 4:
sort
 ↓
two pointers
```

---

# 37. How to Identify the Correct Pointer Movement

Ask:

### Pair sum

```text
sum < target → left++
sum > target → right--
```

### Palindrome

```text
compare
left++
right--
```

### Container

```text
move shorter side
```

### Three Sum

```text
sum < target → left++
sum > target → right--
```

### Remove duplicates

```text
fast scans
slow writes
```

### Sliding window

```text
right expands
left shrinks when invalid
```

---

# 38. Complexity Benefits

A common brute-force pair solution:

```text
for i
    for j
```

has:

```text
O(n²)
```

Two pointers can reduce it to:

```text
O(n)
```

when the required monotonic property exists.

This is one of the most important optimization patterns in DSA.

---

# 39. When Two Pointers Does NOT Work

Do not force the technique.

It may not work when:

```text
Array is unsorted and cannot be sorted
Pointer movement is not monotonic
You need arbitrary random lookups
The problem requires complex state
```

For example, Two Sum on an unsorted array is usually better with HashMap.

---

# 40. Common Mistakes

### Mistake 1 — Using two pointers without a valid invariant

Moving pointers randomly does not guarantee correctness.

Always know:

```text
Why is it safe to move left?
Why is it safe to move right?
```

---

### Mistake 2 — Forgetting duplicates

Three Sum and Four Sum require careful duplicate handling.

---

### Mistake 3 — Losing original indexes

Sorting makes pointer logic easier but destroys original positions.

If indexes are required, consider:

```text
HashMap
```

or store:

```text
value + original index
```

---

### Mistake 4 — Off-by-one errors

Pay attention to:

```java
left < right
```

vs:

```java
left <= right
```

---

### Mistake 5 — Integer overflow

For large integer values:

```java
long sum =
    (long) a + b + c;
```

may be safer than using `int`.

---

### Mistake 6 — Infinite loops

Make sure every loop iteration moves at least one pointer.

For example:

```java
left++;
```

or:

```java
right--;
```

---

### Mistake 7 — Modifying the input unexpectedly

Some problems allow in-place changes; others do not.

Check the problem statement before:

```java
Arrays.sort(nums);
```

or modifying elements.

---

# 41. Interview Questions — Easy

1. Reverse an array using two pointers.
2. Check whether a string is a palindrome.
3. Check whether an array is a palindrome.
4. Remove duplicates from a sorted array.
5. Move zeroes to the end.
6. Remove a given value from an array.
7. Merge two sorted arrays.
8. Find the intersection of two sorted arrays.
9. Find the minimum difference pair.
10. Check whether two sorted arrays contain a common value.

---

# 42. Interview Questions — Medium

11. Two Sum II — sorted array.
12. Container With Most Water.
13. Three Sum.
14. Four Sum.
15. Sort Colors.
16. Squares of a Sorted Array.
17. Pair with a given difference.
18. Closest pair to a target.
19. Trapping Rain Water.
20. Valid Palindrome.
21. Compare strings with backspaces.
22. Partition an array.
23. Remove duplicates in-place.
24. Merge sorted data.
25. Find duplicate values using pointer techniques.

---

# 43. Interview Questions — Advanced

26. Four Sum.
27. Trapping Rain Water.
28. Three Sum Closest.
29. Minimum window using pointer-based techniques.
30. Linked-list cycle detection.
31. Find cycle entry.
32. Find middle of linked list.
33. Palindrome linked list.
34. Reorder linked list.
35. Partition linked list.
36. Detect intersection of two linked lists.

---

# 44. Two Pointers Problem-Solving Checklist

Before coding:

```text
□ Is the data sorted?
□ Can I use left/right pointers?
□ Can I sort first?
□ Are original indexes required?
□ Is the answer about a pair?
□ Is there a monotonic pointer movement?
□ Can I solve it in-place?
□ Are duplicates present?
□ Do I need to skip duplicates?
□ Could integer overflow occur?
□ What invariant am I maintaining?
□ What is the time complexity?
□ What is the space complexity?
```

---

# 45. Quick Revision

```text
Two Pointers
│
├── Opposite Direction
│   ├── Reverse Array
│   ├── Palindrome
│   ├── Two Sum Sorted
│   ├── Container With Most Water
│   └── Trapping Rain Water
│
├── Same Direction
│   ├── Remove Duplicates
│   ├── Move Zeroes
│   └── Partitioning
│
├── Fixed + Two Pointers
│   ├── Three Sum
│   └── Four Sum
│
├── Multiple Arrays
│   ├── Merge Sorted Arrays
│   ├── Intersection
│   └── Union
│
├── Sorting + Two Pointers
│   ├── Three Sum
│   ├── Four Sum
│   ├── Pair Difference
│   └── Closest Pair
│
└── Fast + Slow
    ├── Linked List Middle
    ├── Cycle Detection
    └── Cycle Entry
```

---

## Interview Rule

> **Two pointers work because pointer movement is justified by an invariant. Before moving a pointer, be able to explain why every option you are skipping can no longer produce a better or valid answer.**
