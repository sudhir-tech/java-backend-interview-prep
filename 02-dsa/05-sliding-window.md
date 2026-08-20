# DSA — Sliding Window

Sliding Window is one of the most important DSA patterns for Java backend interviews.

It is mainly used for problems involving a **contiguous range** such as:

- Subarrays
- Substrings
- Fixed-size windows
- Longest valid ranges
- Shortest valid ranges
- Frequency-based windows
- At-most / exactly `K` conditions

The core idea is to avoid recalculating every possible range from scratch.

---

# 1. What is Sliding Window?

A sliding window maintains a range:

```text
[left ... right]
```

Instead of starting over for every subarray, we:

```text
Expand → move right
Shrink  → move left
```

Basic pattern:

```java
int left = 0;

for (int right = 0;
     right < nums.length;
     right++) {

    // Add nums[right]

    while (/* window is invalid */) {

        // Remove nums[left]
        left++;
    }

    // Process current window
}
```

---

# 2. Why Sliding Window?

Consider finding the maximum sum of every subarray of size `k`.

Brute force:

```text
For every starting position:
    Calculate the next k elements
```

This can take:

```text
O(n * k)
```

Sliding window:

```text
Add new element
Remove old element
```

takes:

```text
O(n)
```

---

# 3. When Should You Think About Sliding Window?

Look for words such as:

```text
Subarray
Substring
Contiguous
Consecutive
Window
Longest
Shortest
Maximum
Minimum
At most K
Exactly K
```

Ask:

> Can I maintain a contiguous range while moving through the input?

If yes, sliding window may be appropriate.

---

# 4. Fixed-Size Sliding Window

The window size is exactly:

```text
k
```

Example:

```text
[1, 2, 3, 4, 5]
k = 3
```

Windows:

```text
[1, 2, 3]
[2, 3, 4]
[3, 4, 5]
```

---

# 5. Maximum Sum Subarray of Size K

```java
static int maxSum(int[] nums, int k) {

    int windowSum = 0;

    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }

    int maximum = windowSum;

    for (int right = k;
         right < nums.length;
         right++) {

        windowSum += nums[right];

        windowSum -=
            nums[right - k];

        maximum = Math.max(
            maximum,
            windowSum
        );
    }

    return maximum;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 6. Minimum Sum Subarray of Size K

The same technique works for minimum sum.

```java
static int minSum(int[] nums, int k) {

    int windowSum = 0;

    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }

    int minimum = windowSum;

    for (int right = k;
         right < nums.length;
         right++) {

        windowSum += nums[right];
        windowSum -= nums[right - k];

        minimum = Math.min(
            minimum,
            windowSum
        );
    }

    return minimum;
}
```

---

# 7. Average of Every Window

For a fixed window:

```text
sum / k
```

Instead of recalculating the sum:

```java
windowSum += nums[right];
windowSum -= nums[right - k];
```

Then:

```java
double average =
    (double) windowSum / k;
```

---

# 8. First Negative Number in Every Window

Suppose:

```text
nums = [12, -1, -7, 8, -15, 30, 16, 28]
k = 3
```

For each window, find the first negative number.

A useful structure is:

```java
Deque<Integer> negatives =
    new ArrayDeque<>();
```

Store indexes of negative numbers.

For every `right`:

```text
1. Add negative index.
2. Remove indexes outside the window.
3. The front is the first negative.
```

---

# 9. Variable-Size Sliding Window

The window size changes.

Basic structure:

```java
int left = 0;

for (int right = 0;
     right < nums.length;
     right++) {

    // Add nums[right]

    while (/* window invalid */) {

        // Remove nums[left]
        left++;
    }

    // Process window
}
```

This is one of the most important patterns to recognize.

---

# 10. Longest Substring Without Repeating Characters

Example:

```text
abcabcbb
```

Answer:

```text
3
```

because:

```text
abc
```

is the longest substring without repeating characters.

Using HashSet:

```java
static int lengthOfLongestSubstring(
        String s) {

    Set<Character> set =
        new HashSet<>();

    int left = 0;
    int maximum = 0;

    for (int right = 0;
         right < s.length();
         right++) {

        char current =
            s.charAt(right);

        while (set.contains(current)) {

            set.remove(
                s.charAt(left)
            );

            left++;
        }

        set.add(current);

        maximum = Math.max(
            maximum,
            right - left + 1
        );
    }

    return maximum;
}
```

### Complexity

```text
Time:  O(n)
Space: O(k)
```

---

# 11. Optimized Longest Unique Substring

Store the latest index instead of removing characters one by one.

```java
static int lengthOfLongestSubstring(
        String s) {

    Map<Character, Integer> lastSeen =
        new HashMap<>();

    int left = 0;
    int maximum = 0;

    for (int right = 0;
         right < s.length();
         right++) {

        char c = s.charAt(right);

        if (lastSeen.containsKey(c)) {
            left = Math.max(
                left,
                lastSeen.get(c) + 1
            );
        }

        lastSeen.put(c, right);

        maximum = Math.max(
            maximum,
            right - left + 1
        );
    }

    return maximum;
}
```

### Important

Do not simply write:

```java
left = lastSeen.get(c) + 1;
```

Use:

```java
left = Math.max(
    left,
    lastSeen.get(c) + 1
);
```

Otherwise `left` could move backwards.

---

# 12. Longest Subarray with Sum at Most K

For positive numbers, a variable-size window can be used.

```java
int left = 0;
int sum = 0;
int maximum = 0;

for (int right = 0;
     right < nums.length;
     right++) {

    sum += nums[right];

    while (sum > k) {
        sum -= nums[left++];
    }

    maximum = Math.max(
        maximum,
        right - left + 1
    );
}
```

### Important

This direct sliding-window approach relies on the values being non-negative/positive.

If negative values are allowed, the monotonic behavior may disappear.

---

# 13. Longest Subarray with Sum Exactly K

Be careful.

For arrays containing arbitrary positive and negative values, a simple sliding window is generally **not** reliable.

Instead, use:

```text
Prefix Sum + HashMap
```

Example:

```java
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
```

### Interview lesson

> Sliding window often needs a monotonic condition. Prefix sum + hashing is safer for arbitrary integer arrays.

---

# 14. Minimum Size Subarray Sum

Given positive integers, find the smallest contiguous subarray whose sum is at least `target`.

```java
static int minSubArrayLen(
        int target,
        int[] nums) {

    int left = 0;
    int sum = 0;
    int minimum = Integer.MAX_VALUE;

    for (int right = 0;
         right < nums.length;
         right++) {

        sum += nums[right];

        while (sum >= target) {

            minimum = Math.min(
                minimum,
                right - left + 1
            );

            sum -= nums[left++];
        }
    }

    return minimum == Integer.MAX_VALUE
        ? 0
        : minimum;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 15. Longest Substring with At Most K Distinct Characters

Maintain a frequency map.

```java
static int longestAtMostKDistinct(
        String s,
        int k) {

    Map<Character, Integer> frequency =
        new HashMap<>();

    int left = 0;
    int maximum = 0;

    for (int right = 0;
         right < s.length();
         right++) {

        char c = s.charAt(right);

        frequency.merge(
            c,
            1,
            Integer::sum
        );

        while (frequency.size() > k) {

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

        maximum = Math.max(
            maximum,
            right - left + 1
        );
    }

    return maximum;
}
```

---

# 16. Longest Substring with Exactly K Distinct Characters

A useful trick:

```text
Exactly K
=
At most K
-
At most K - 1
```

For counting substrings:

```java
countExactlyK(k)
=
countAtMostK(k)
-
countAtMostK(k - 1)
```

This is a very important interview pattern.

---

# 17. Count Subarrays with Exactly K Distinct Integers

Same principle:

```text
exactly(K)
=
atMost(K)
-
atMost(K - 1)
```

This is often easier than trying to maintain an exact-K window directly.

---

# 18. Fruit Into Baskets

This is essentially:

```text
Longest subarray with at most 2 distinct values
```

Maintain:

```java
Map<Integer, Integer> frequency;
```

and shrink when:

```text
frequency.size() > 2
```

This is a classic sliding-window recognition problem.

---

# 19. Longest Repeating Character Replacement

Given:

```text
AABABBA
k = 1
```

The goal is the longest substring that can be made entirely of one character after at most `k` replacements.

Key formula:

```text
window length - maximum character frequency <= k
```

If the condition becomes invalid:

```text
left++
```

---

# 20. Longest Repeating Character Replacement — Java

```java
static int characterReplacement(
        String s,
        int k) {

    int[] frequency =
        new int[26];

    int left = 0;
    int maxFrequency = 0;
    int maximum = 0;

    for (int right = 0;
         right < s.length();
         right++) {

        int index =
            s.charAt(right) - 'A';

        frequency[index]++;

        maxFrequency = Math.max(
            maxFrequency,
            frequency[index]
        );

        while ((right - left + 1)
                - maxFrequency > k) {

            frequency[
                s.charAt(left) - 'A'
            ]--;

            left++;
        }

        maximum = Math.max(
            maximum,
            right - left + 1
        );
    }

    return maximum;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 21. Permutation in String

Determine whether one string's permutation occurs as a substring of another.

Example:

```text
s1 = "ab"
s2 = "eidbaooo"
```

Answer:

```text
true
```

because:

```text
ba
```

is a permutation of:

```text
ab
```

Use:

```text
Fixed-size sliding window
+
Character frequency
```

---

# 22. Find All Anagrams in a String

Example:

```text
s = "cbaebabacd"
p = "abc"
```

Output:

```text
[0, 6]
```

The windows:

```text
cba
bac
```

are anagrams of `abc`.

For lowercase English letters, maintain two frequency arrays or a single difference/count structure.

---

# 23. Minimum Window Substring

One of the most important advanced sliding-window problems.

Given:

```text
s = "ADOBECODEBANC"
t = "ABC"
```

Answer:

```text
"BANC"
```

Approach:

```text
1. Count required characters.
2. Expand right.
3. Track characters in the current window.
4. Once all required characters are present,
   shrink from the left.
5. Record the smallest valid window.
```

---

# 24. Minimum Window Substring — Core Template

```java
Map<Character, Integer> need =
    new HashMap<>();

for (char c : t.toCharArray()) {
    need.merge(
        c,
        1,
        Integer::sum
    );
}

Map<Character, Integer> window =
    new HashMap<>();

int left = 0;
int formed = 0;
int required = need.size();

int bestLength =
    Integer.MAX_VALUE;

int bestLeft = 0;

for (int right = 0;
     right < s.length();
     right++) {

    char c = s.charAt(right);

    window.merge(
        c,
        1,
        Integer::sum
    );

    if (need.containsKey(c)
            && window.get(c)
                .intValue()
                == need.get(c).intValue()) {

        formed++;
    }

    while (formed == required) {

        if (right - left + 1
                < bestLength) {

            bestLength =
                right - left + 1;

            bestLeft = left;
        }

        char leftChar =
            s.charAt(left++);

        window.merge(
            leftChar,
            -1,
            Integer::sum
        );

        if (need.containsKey(leftChar)
                && window.get(leftChar)
                    < need.get(leftChar)) {

            formed--;
        }
    }
}

return bestLength == Integer.MAX_VALUE
    ? ""
    : s.substring(
        bestLeft,
        bestLeft + bestLength
    );
```

### Complexity

```text
Time:  O(n)
Space: O(k)
```

---

# 25. Sliding Window Maximum

Given:

```text
nums = [1,3,-1,-3,5,3,6,7]
k = 3
```

Output:

```text
[3,3,5,5,6,7]
```

A naive solution scans every window:

```text
O(nk)
```

The optimized solution uses:

```text
Deque
```

---

# 26. Sliding Window Maximum — Deque

Maintain indexes in decreasing order of values.

```java
static int[] maxSlidingWindow(
        int[] nums,
        int k) {

    if (nums.length == 0 || k == 0) {
        return new int[0];
    }

    int[] result =
        new int[nums.length - k + 1];

    Deque<Integer> deque =
        new ArrayDeque<>();

    int index = 0;

    for (int right = 0;
         right < nums.length;
         right++) {

        while (!deque.isEmpty()
                && deque.peekFirst()
                    <= right - k) {

            deque.pollFirst();
        }

        while (!deque.isEmpty()
                && nums[deque.peekLast()]
                    <= nums[right]) {

            deque.pollLast();
        }

        deque.offerLast(right);

        if (right >= k - 1) {
            result[index++] =
                nums[deque.peekFirst()];
        }
    }

    return result;
}
```

### Complexity

```text
Time:  O(n)
Space: O(k)
```

---

# 27. Why Deque Works for Sliding Maximum

The deque stores indexes whose values are useful candidates.

Maintain decreasing values:

```text
largest
↓
smaller
↓
smaller
```

If a new value is larger than elements at the back, those smaller elements can never become the maximum while the new value remains in the window.

So remove them.

---

# 28. Count Subarrays with Sum at Most K

For positive numbers, a sliding window can sometimes maintain:

```text
sum <= k
```

However, if negative values are allowed, standard sliding-window assumptions may fail.

Always check constraints before choosing the technique.

---

# 29. Maximum Number of Vowels in a Substring of Size K

For a fixed-size window:

```java
static int maxVowels(
        String s,
        int k) {

    int count = 0;

    for (int i = 0; i < k; i++) {
        if (isVowel(s.charAt(i))) {
            count++;
        }
    }

    int maximum = count;

    for (int right = k;
         right < s.length();
         right++) {

        if (isVowel(s.charAt(right))) {
            count++;
        }

        if (isVowel(
                s.charAt(right - k))) {
            count--;
        }

        maximum = Math.max(
            maximum,
            count
        );
    }

    return maximum;
}
```

---

# 30. Binary Subarrays With Sum

For binary arrays, counting exact sums can be approached using:

```text
Prefix Sum
```

or:

```text
AtMost(goal)
-
AtMost(goal - 1)
```

The second approach is another important connection between sliding windows and exact-count problems.

---

# 31. Subarrays with Exactly K Odd Numbers

Treat odd numbers as:

```text
1
```

and even numbers as:

```text
0
```

Then the problem becomes:

```text
Count subarrays with sum exactly K
```

Possible approaches:

```text
Prefix Sum + HashMap
```

or:

```text
atMost(K) - atMost(K - 1)
```

---

# 32. Count Number of Nice Subarrays

This is the same pattern as exactly `K` odd numbers.

Recognition:

```text
Exactly K
```

Think:

```text
atMost(K)
-
atMost(K - 1)
```

---

# 33. Fixed vs Variable Window

### Fixed window

Window size is known:

```text
k
```

Example:

```text
Maximum sum of size k
```

Typical structure:

```java
for (int right = 0;
     right < n;
     right++) {

    // Add right

    if (right >= k) {
        // Remove right - k
    }

    if (right >= k - 1) {
        // Process window
    }
}
```

### Variable window

Window expands and shrinks based on a condition:

```text
while invalid:
    left++
```

Example:

```text
Minimum size subarray sum
Longest substring without duplicates
```

---

# 34. Sliding Window + HashMap

Common combination:

```text
String
   ↓
Sliding Window
   ↓
HashMap frequency
```

Useful for:

```text
K distinct
Anagrams
Permutation
Minimum Window
Character Replacement
```

---

# 35. Sliding Window + HashSet

Common for:

```text
Unique elements
No duplicates
Nearby duplicates
Longest unique substring
```

Example:

```java
Set<Character> set =
    new HashSet<>();
```

---

# 36. Sliding Window + Deque

Useful when you need:

```text
Maximum in every window
Minimum in every window
Monotonic window data
```

Examples:

```text
Sliding Window Maximum
Sliding Window Minimum
```

---

# 37. Monotonic Deque

A deque can maintain values in:

```text
Increasing order
```

or:

```text
Decreasing order
```

For maximum:

```text
decreasing deque
```

For minimum:

```text
increasing deque
```

This gives efficient access to the current extreme.

---

# 38. Why Sliding Window Is Usually O(n)

At first glance:

```java
for (right ...)
    while (left ...)
```

looks like:

```text
O(n²)
```

But in a proper sliding-window algorithm:

```text
right moves at most n times
left moves at most n times
```

Therefore total pointer movement is:

```text
O(n + n)
=
O(n)
```

This is an important interview explanation.

---

# 39. When Sliding Window Fails

Do not use sliding window automatically for every subarray problem.

It may fail when:

```text
Negative numbers destroy monotonicity
The condition cannot be maintained incrementally
The window is not contiguous
The problem requires arbitrary ranges
```

For example:

```text
Longest subarray with sum K
```

with arbitrary positive and negative integers is generally better handled with prefix sums and hashing.

---

# 40. Sliding Window vs Prefix Sum

### Sliding Window

Best when:

```text
The window condition is monotonic.
```

Examples:

```text
Positive numbers
At most K
Minimum size satisfying condition
Longest valid range
```

### Prefix Sum

Best when:

```text
You need exact range sums
Negative numbers exist
You need many range queries
```

Examples:

```text
Subarray Sum Equals K
Zero Sum Subarrays
Longest Subarray Sum K
```

---

# 41. Sliding Window vs Two Pointers

Sliding window is often implemented using two pointers:

```text
left
right
```

But the purpose is different.

### Two pointers

Usually focuses on pointer relationships.

Examples:

```text
Two Sum Sorted
Palindrome
Three Sum
Container With Most Water
```

### Sliding window

Maintains a contiguous valid range.

Examples:

```text
Longest Substring
Minimum Window
Maximum Sum Window
K Distinct
```

---

# 42. Common Sliding Window Template

```java
int left = 0;

for (int right = 0;
     right < n;
     right++) {

    // Add right element.

    while (/* invalid */) {

        // Remove left element.
        left++;
    }

    // Current window:
    // [left ... right]
}
```

Memorize the structure, but understand the condition.

---

# 43. Fixed Window Template

```java
int windowValue = 0;

for (int right = 0;
     right < n;
     right++) {

    windowValue += nums[right];

    if (right >= k) {
        windowValue -=
            nums[right - k];
    }

    if (right >= k - 1) {
        // Process current window.
    }
}
```

---

# 44. Variable Window Template

```java
int left = 0;

for (int right = 0;
     right < n;
     right++) {

    // Add nums[right]

    while (/* window invalid */) {

        // Remove nums[left]
        left++;
    }

    // Process valid window.
}
```

---

# 45. Exact K Template

A powerful pattern:

```text
exactly(K)
=
atMost(K)
-
atMost(K - 1)
```

Common applications:

```text
Exactly K distinct
Exactly K odd numbers
Binary subarrays with sum K
Nice subarrays
```

---

# 46. Interview Questions — Easy

1. Maximum sum subarray of size K.
2. Minimum sum subarray of size K.
3. Average of every window of size K.
4. Maximum number of vowels in a substring of size K.
5. First negative number in every window.
6. Find a fixed-size window with maximum/minimum value.
7. Count elements satisfying a condition in every fixed window.

---

# 47. Interview Questions — Medium

8. Longest substring without repeating characters.
9. Longest substring with at most K distinct characters.
10. Longest repeating character replacement.
11. Minimum size subarray sum.
12. Fruit Into Baskets.
13. Permutation in String.
14. Find All Anagrams in a String.
15. Binary Subarrays With Sum.
16. Count Nice Subarrays.
17. Subarrays with K distinct integers.
18. Nearby Duplicate.
19. Maximum consecutive ones with K replacements.

---

# 48. Interview Questions — Advanced

20. Minimum Window Substring.
21. Sliding Window Maximum.
22. Sliding Window Minimum.
23. Longest substring with exactly K distinct characters.
24. Count subarrays with exactly K distinct values.
25. Maximum frequency stack/window combinations.
26. Shortest subarray with sum at least K.
27. Subarray problems with monotonic deque.
28. Advanced substring frequency problems.

---

# 49. Common Mistakes

### Mistake 1 — Forgetting to remove the left element

When shrinking:

```java
left++;
```

you usually must also remove:

```text
nums[left]
```

from the window's state.

---

### Mistake 2 — Moving left without checking validity

Do not shrink randomly.

Know exactly:

```text
Why is the window invalid?
```

---

### Mistake 3 — Using sliding window with negative numbers incorrectly

For sum-based problems, negative values can break monotonicity.

---

### Mistake 4 — Moving `left` backwards

In optimized substring problems:

```java
left = Math.max(
    left,
    lastSeen + 1
);
```

---

### Mistake 5 — Wrong window size

For inclusive indexes:

```text
window size =
right - left + 1
```

---

### Mistake 6 — Forgetting the first valid window

For fixed-size windows, process only when:

```java
right >= k - 1
```

---

### Mistake 7 — Not removing zero-frequency keys

If using:

```java
Map<Character, Integer>
```

remove entries when their count reaches zero if your condition depends on:

```java
map.size()
```

---

### Mistake 8 — Overcomplicating fixed windows

If the window size is fixed, often all you need is:

```text
add new
remove old
```

---

# 50. Edge Cases

Always test:

```text
Empty input
One element
k = 1
k = n
k > n
All elements equal
All elements different
All negative values
All positive values
Duplicate values
No valid window
Entire array is the answer
Answer is one element
```

---

# 51. Sliding Window Problem-Solving Checklist

Before coding:

```text
□ Is the range contiguous?
□ Is the window fixed-size or variable-size?
□ What makes the window valid?
□ What makes the window invalid?
□ What state must I maintain?
□ Do I need a HashMap?
□ Do I need a HashSet?
□ Do I need a Deque?
□ Can negative numbers occur?
□ Is the condition monotonic?
□ When should I move right?
□ When should I move left?
□ What happens when the window becomes valid?
□ What happens when it becomes invalid?
□ What is the time complexity?
□ What is the space complexity?
```

---

# 52. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Max Sum Size K | Fixed Window | O(n) | O(1) |
| Min Sum Size K | Fixed Window | O(n) | O(1) |
| Longest Unique Substring | Window + Set/Map | O(n) | O(k) |
| At Most K Distinct | Window + Map | O(n) | O(k) |
| Min Size Subarray Sum | Variable Window | O(n) | O(1) |
| Character Replacement | Window + Frequency | O(n) | O(1) |
| Permutation in String | Fixed Window + Frequency | O(n) | O(1) |
| Find All Anagrams | Fixed Window + Frequency | O(n) | O(1) |
| Minimum Window | Window + Map | O(n) | O(k) |
| Sliding Window Maximum | Window + Deque | O(n) | O(k) |
| Nearby Duplicate | Window + Set | O(n) avg | O(k) |

---

# 53. Quick Revision

```text
Sliding Window
│
├── Fixed Window
│   ├── Max Sum Size K
│   ├── Min Sum Size K
│   ├── Average Window
│   ├── Max Vowels
│   └── First Negative
│
├── Variable Window
│   ├── Longest Unique Substring
│   ├── At Most K Distinct
│   ├── Min Size Subarray
│   ├── Fruit Into Baskets
│   └── Character Replacement
│
├── Frequency Window
│   ├── Anagrams
│   ├── Permutation
│   └── Minimum Window
│
├── Hashing + Window
│   ├── K Distinct
│   ├── Nearby Duplicate
│   └── Exact K
│
├── Deque + Window
│   ├── Maximum
│   └── Minimum
│
└── Exact K
    └── atMost(K) - atMost(K - 1)
```

---

## Interview Rule

> **When you see a contiguous subarray or substring problem, first ask whether a sliding window can maintain the required condition incrementally. The most important skill is knowing exactly when to expand, when to shrink, and what state the window must maintain.**
