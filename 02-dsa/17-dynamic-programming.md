# DSA — Dynamic Programming

Dynamic Programming (DP) is a technique for solving problems by breaking them into smaller overlapping subproblems and storing their results so that the same work is not repeated.

DP is one of the most important topics for software engineering interviews.

It commonly appears in:

- Arrays
- Strings
- Subsequence problems
- Knapsack
- Grid problems
- Scheduling
- Stock trading
- Partitioning
- Counting
- Optimization

---

# 1. What is Dynamic Programming?

DP is usually useful when a problem has:

```text
1. Overlapping subproblems
2. Optimal substructure
```

### Overlapping Subproblems

The same smaller problem appears multiple times.

Example:

```text
fib(5)
├── fib(4)
│   ├── fib(3)
│   └── fib(2)
└── fib(3)
    ├── fib(2)
    └── fib(1)
```

`fib(3)` and `fib(2)` are calculated repeatedly.

DP stores their answers.

---

# 2. Optimal Substructure

A problem has optimal substructure when an optimal solution can be built from optimal solutions to smaller subproblems.

Example:

```text
Shortest path A → C
```

through:

```text
A → B → C
```

requires an optimal solution for:

```text
A → B
```

and:

```text
B → C
```

---

# 3. Two Main DP Approaches

There are two common approaches:

```text
1. Top-Down
2. Bottom-Up
```

---

# 4. Top-Down DP

Top-down uses:

```text
Recursion
+
Memoization
```

Start with the original problem and recursively solve smaller problems.

Store each result.

---

# 5. Bottom-Up DP

Bottom-up uses:

```text
Iteration
+
Table
```

Start from the smallest subproblems and build toward the final answer.

---

# 6. Fibonacci — Recursive

Naive recursion:

```java
static int fib(int n) {

    if (n <= 1) {
        return n;
    }

    return fib(n - 1)
        + fib(n - 2);
}
```

Complexity:

```text
Time: O(2^n)
Space: O(n)
```

This repeats the same work many times.

---

# 7. Fibonacci — Memoization

```java
static int fib(
        int n,
        int[] memo) {

    if (n <= 1) {
        return n;
    }

    if (memo[n] != -1) {
        return memo[n];
    }

    memo[n] =
        fib(n - 1, memo)
        + fib(n - 2, memo);

    return memo[n];
}
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

---

# 8. Fibonacci — Bottom-Up

```java
static int fib(int n) {

    if (n <= 1) {
        return n;
    }

    int[] dp =
        new int[n + 1];

    dp[0] = 0;
    dp[1] = 1;

    for (int i = 2;
         i <= n;
         i++) {

        dp[i] =
            dp[i - 1]
            + dp[i - 2];
    }

    return dp[n];
}
```

---

# 9. Space Optimization

Notice:

```text
dp[i]
depends only on:
dp[i - 1]
dp[i - 2]
```

We do not need the entire array.

```java
static int fibOptimized(
        int n) {

    if (n <= 1) {
        return n;
    }

    int previous2 = 0;
    int previous1 = 1;

    for (int i = 2;
         i <= n;
         i++) {

        int current =
            previous1 + previous2;

        previous2 = previous1;
        previous1 = current;
    }

    return previous1;
}
```

Space:

```text
O(1)
```

---

# 10. How to Identify DP

Ask:

```text
1. Is there a choice at every step?
2. Does the problem contain smaller repeated states?
3. Can I define a state?
4. Does the answer depend on previous states?
5. Am I maximizing, minimizing, or counting?
```

Common phrases:

```text
maximum
minimum
number of ways
can you achieve
longest
shortest
minimum cost
maximum profit
```

---

# 11. The 5-Step DP Framework

For most DP problems:

```text
1. Define the state.
2. Define the transition.
3. Define base cases.
4. Determine computation order.
5. Return the required state.
```

Example:

```text
dp[i] =
best answer using first i elements
```

Then determine:

```text
dp[i] from previous states
```

---

# 12. Climbing Stairs

You can climb:

```text
1 step
or
2 steps
```

How many distinct ways to reach step `n`?

For the last move:

```text
come from n - 1
or
come from n - 2
```

Therefore:

```text
dp[n] =
dp[n - 1]
+
dp[n - 2]
```

---

# 13. Climbing Stairs — Java

```java
static int climbStairs(
        int n) {

    if (n <= 2) {
        return n;
    }

    int previous2 = 1;
    int previous1 = 2;

    for (int i = 3;
         i <= n;
         i++) {

        int current =
            previous1 + previous2;

        previous2 = previous1;
        previous1 = current;
    }

    return previous1;
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 14. Min Cost Climbing Stairs

Each step has a cost.

You can climb:

```text
1 or 2 steps
```

Goal:

```text
minimum cost to reach the top
```

State:

```text
dp[i] =
minimum cost to reach step i
```

Transition:

```text
dp[i] =
cost[i]
+
min(
    dp[i - 1],
    dp[i - 2]
)
```

---

# 15. House Robber

You cannot rob two adjacent houses.

Example:

```text
[2,7,9,3,1]
```

Answer:

```text
12
```

Choose:

```text
2 + 9 + 1
```

---

# 16. House Robber — DP

At house `i`:

```text
Skip house i
```

or:

```text
Rob house i
```

Therefore:

```text
dp[i] =
max(
    dp[i - 1],
    dp[i - 2] + nums[i]
)
```

---

# 17. House Robber — Java

```java
static int rob(
        int[] nums) {

    int previous2 = 0;
    int previous1 = 0;

    for (int money : nums) {

        int current =
            Math.max(
                previous1,
                previous2 + money
            );

        previous2 = previous1;
        previous1 = current;
    }

    return previous1;
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 18. House Robber II

Houses are arranged in a circle.

Therefore:

```text
first and last
```

cannot both be robbed.

Break into two cases:

```text
1. Rob from house 0 to n-2
2. Rob from house 1 to n-1
```

Answer:

```text
max(case1, case2)
```

This is a common DP decomposition technique.

---

# 19. Maximum Subarray

Given:

```text
[-2,1,-3,4,-1,2,1,-5,4]
```

maximum subarray sum:

```text
6
```

from:

```text
[4,-1,2,1]
```

---

# 20. Kadane's Algorithm

State:

```text
current =
best subarray ending at current position
```

Transition:

```text
current =
max(
    nums[i],
    current + nums[i]
)
```

Global answer:

```text
maximum current value
```

---

# 21. Kadane — Java

```java
static int maxSubArray(
        int[] nums) {

    int current = nums[0];
    int best = nums[0];

    for (int i = 1;
         i < nums.length;
         i++) {

        current =
            Math.max(
                nums[i],
                current + nums[i]
            );

        best =
            Math.max(
                best,
                current
            );
    }

    return best;
}
```

Complexity:

```text
O(n)
```

---

# 22. Maximum Product Subarray

Unlike maximum sum, a negative number can turn a small negative product into a large positive product.

Maintain:

```text
maximum product ending here
minimum product ending here
```

Because:

```text
negative × negative = positive
```

---

# 23. Longest Increasing Subsequence

Given:

```text
[10,9,2,5,3,7,101,18]
```

LIS length:

```text
4
```

Example:

```text
[2,3,7,101]
```

---

# 24. LIS — O(n²) DP

State:

```text
dp[i] =
length of LIS ending at i
```

Transition:

```text
if nums[j] < nums[i]

dp[i] =
max(
    dp[i],
    dp[j] + 1
)
```

---

# 25. LIS — Java

```java
static int lengthOfLIS(
        int[] nums) {

    int n = nums.length;

    int[] dp =
        new int[n];

    Arrays.fill(
        dp,
        1
    );

    int answer = 0;

    for (int i = 0;
         i < n;
         i++) {

        for (int j = 0;
             j < i;
             j++) {

            if (nums[j] < nums[i]) {

                dp[i] =
                    Math.max(
                        dp[i],
                        dp[j] + 1
                    );
            }
        }

        answer =
            Math.max(
                answer,
                dp[i]
            );
    }

    return answer;
}
```

Complexity:

```text
Time: O(n²)
Space: O(n)
```

---

# 26. LIS — O(n log n)

A more optimized LIS solution uses:

```text
tails array
+
binary search
```

Important:

> The `tails` array does not necessarily represent the actual LIS. It stores the smallest possible tail value for increasing subsequences of different lengths.

Complexity:

```text
O(n log n)
```

This is a common advanced interview optimization.

---

# 27. Longest Common Subsequence

Given:

```text
text1 = "abcde"
text2 = "ace"
```

LCS:

```text
"ace"
```

Length:

```text
3
```

---

# 28. LCS State

Define:

```text
dp[i][j]
```

as the LCS length using:

```text
first i characters of text1
first j characters of text2
```

If:

```text
text1[i - 1] == text2[j - 1]
```

then:

```text
dp[i][j] =
dp[i - 1][j - 1] + 1
```

Otherwise:

```text
dp[i][j] =
max(
    dp[i - 1][j],
    dp[i][j - 1]
)
```

---

# 29. LCS — Java

```java
static int longestCommonSubsequence(
        String first,
        String second) {

    int m = first.length();
    int n = second.length();

    int[][] dp =
        new int[m + 1][n + 1];

    for (int i = 1;
         i <= m;
         i++) {

        for (int j = 1;
             j <= n;
             j++) {

            if (first.charAt(i - 1)
                    == second.charAt(j - 1)) {

                dp[i][j] =
                    dp[i - 1][j - 1] + 1;

            } else {

                dp[i][j] =
                    Math.max(
                        dp[i - 1][j],
                        dp[i][j - 1]
                    );
            }
        }
    }

    return dp[m][n];
}
```

Complexity:

```text
Time: O(mn)
Space: O(mn)
```

---

# 30. Longest Common Substring

Do not confuse:

```text
Subsequence
```

with:

```text
Substring
```

### Subsequence

Characters do not have to be adjacent.

### Substring

Characters must be contiguous.

The DP transition for longest common substring resets to:

```text
0
```

when characters do not match.

---

# 31. Edit Distance

Operations:

```text
Insert
Delete
Replace
```

Convert:

```text
word1
```

into:

```text
word2
```

State:

```text
dp[i][j] =
minimum operations to convert
first i chars to first j chars
```

---

# 32. Edit Distance Transition

If characters match:

```text
dp[i][j] =
dp[i - 1][j - 1]
```

Otherwise:

```text
dp[i][j] =
1 + min(
    insert,
    delete,
    replace
)
```

---

# 33. 0/1 Knapsack

Each item has:

```text
weight
value
```

You can either:

```text
take
or
skip
```

Each item can be used at most once.

---

# 34. Knapsack State

```text
dp[i][capacity]
```

represents the maximum value using the first `i` items with the given capacity.

For item `i`:

```text
skip:
dp[i - 1][capacity]
```

take:

```text
value[i]
+
dp[i - 1][capacity - weight[i]]
```

Therefore:

```text
dp[i][capacity] =
max(
    skip,
    take
)
```

---

# 35. 0/1 Knapsack — 1D DP

Can optimize space to:

```text
O(capacity)
```

Important:

```text
iterate capacity backwards
```

because each item can be used only once.

```java
static int knapsack(
        int[] weights,
        int[] values,
        int capacity) {

    int[] dp =
        new int[capacity + 1];

    for (int i = 0;
         i < weights.length;
         i++) {

        for (int current =
                capacity;
             current >= weights[i];
             current--) {

            dp[current] =
                Math.max(
                    dp[current],
                    values[i]
                    + dp[
                        current - weights[i]
                    ]
                );
        }
    }

    return dp[capacity];
}
```

---

# 36. Unbounded Knapsack

Unlike 0/1 Knapsack:

```text
an item can be used multiple times
```

The capacity loop usually goes:

```text
forward
```

instead of backward.

This difference is very important.

---

# 37. Coin Change

Given coins:

```text
[1,2,5]
```

find the minimum number of coins needed for:

```text
amount = 11
```

Answer:

```text
3
```

because:

```text
5 + 5 + 1
```

---

# 38. Coin Change — DP

State:

```text
dp[amount] =
minimum coins needed
```

Transition:

```text
dp[current] =
min(
    dp[current],
    dp[current - coin] + 1
)
```

---

# 39. Coin Change — Java

```java
static int coinChange(
        int[] coins,
        int amount) {

    int[] dp =
        new int[amount + 1];

    Arrays.fill(
        dp,
        amount + 1
    );

    dp[0] = 0;

    for (int current = 1;
         current <= amount;
         current++) {

        for (int coin : coins) {

            if (coin <= current) {

                dp[current] =
                    Math.min(
                        dp[current],
                        dp[current - coin] + 1
                    );
            }
        }
    }

    return dp[amount]
        > amount
        ? -1
        : dp[amount];
}
```

Complexity:

```text
O(amount × number of coins)
```

---

# 40. Coin Change II

Instead of minimum number of coins, count:

```text
number of combinations
```

The order of coins does not matter.

A common pattern is:

```java
for (int coin : coins) {
    for (int amount = coin;
         amount <= target;
         amount++) {

        dp[amount] +=
            dp[amount - coin];
    }
}
```

The loop order prevents counting different permutations as separate combinations.

---

# 41. Partition Equal Subset Sum

Determine whether an array can be divided into two subsets with equal sum.

If total sum is:

```text
S
```

we need a subset with:

```text
S / 2
```

If `S` is odd:

```text
impossible
```

This becomes:

```text
0/1 Knapsack
```

---

# 42. Target Sum

Assign:

```text
+
or
-
```

before each number to reach a target.

This can often be transformed into a subset-sum DP.

This is a good example of:

```text
problem transformation
```

rather than immediately writing a DP table.

---

# 43. Unique Paths

Grid:

```text
m × n
```

You can move:

```text
right
down
```

How many paths from top-left to bottom-right?

State:

```text
dp[row][col]
```

Transition:

```text
dp[row][col] =
dp[row - 1][col]
+
dp[row][col - 1]
```

---

# 44. Unique Paths — Java

```java
static int uniquePaths(
        int m,
        int n) {

    int[] dp =
        new int[n];

    Arrays.fill(
        dp,
        1
    );

    for (int row = 1;
         row < m;
         row++) {

        for (int col = 1;
             col < n;
             col++) {

            dp[col] +=
                dp[col - 1];
        }
    }

    return dp[n - 1];
}
```

Complexity:

```text
Time: O(mn)
Space: O(n)
```

---

# 45. Unique Paths II

Now some cells contain obstacles.

If a cell is blocked:

```text
dp[col] = 0
```

Otherwise:

```text
dp[col] += dp[col - 1]
```

This is a common grid DP pattern.

---

# 46. Minimum Path Sum

Each grid cell has a cost.

Move:

```text
right
down
```

Find minimum total path sum.

Transition:

```text
dp[row][col] =
grid[row][col]
+
min(
    top,
    left
)
```

---

# 47. Triangle Minimum Path

Given a triangle:

```text
    2
   3 4
  6 5 7
 4 1 8 3
```

Find the minimum top-to-bottom path.

A bottom-up DP is often clean:

```text
dp[j] =
triangle[row][j]
+
min(
    dp[j],
    dp[j + 1]
)
```

---

# 48. Dungeon Game

A more advanced grid DP problem.

You need to determine the minimum initial health required to reach the destination.

Unlike normal path-sum problems, the state represents:

```text
minimum health required before entering a cell
```

This is an important reminder:

> The DP state should represent exactly what the problem asks you to preserve.

---

# 49. Stock DP

Stock problems are classic state-machine DP.

Typical state:

```text
hold
cash
```

For each day:

```text
hold =
max(
    previousHold,
    previousCash - price
)

cash =
max(
    previousCash,
    previousHold + price
)
```

---

# 50. Best Time to Buy and Sell Stock

With one transaction, a simple greedy/DP-like approach works.

Maintain:

```text
minimum price seen
maximum profit
```

```java
static int maxProfit(
        int[] prices) {

    int minimum =
        Integer.MAX_VALUE;

    int profit = 0;

    for (int price : prices) {

        minimum =
            Math.min(
                minimum,
                price
            );

        profit =
            Math.max(
                profit,
                price - minimum
            );
    }

    return profit;
}
```

Complexity:

```text
O(n)
```

---

# 51. Stock with Unlimited Transactions

If you can buy and sell multiple times:

```text
take every positive price increase
```

or use a two-state DP:

```text
hold
cash
```

The state-machine approach becomes especially useful when restrictions are added.

---

# 52. Stock with Cooldown

After selling:

```text
must wait one day
```

Now we need additional state.

Example states:

```text
hold
sold
rest
```

This is a classic state-machine DP problem.

---

# 53. Stock with Transaction Fee

Every sale has a fee.

State:

```text
hold
cash
```

Transition:

```text
cash =
max(
    cash,
    hold + price - fee
)
```

---

# 54. House Robber Pattern

A useful general pattern is:

```text
At each position:
take
or
skip
```

Transition:

```text
dp[i] =
max(
    dp[i - 1],
    dp[i - 2] + value
)
```

This pattern appears in many sequence-selection problems.

---

# 55. Interval DP

Some problems require choosing a split point.

State:

```text
dp[l][r]
```

means:

```text
answer for interval [l, r]
```

Then try every possible split:

```text
l ≤ k < r
```

Examples:

```text
Matrix Chain Multiplication
Burst Balloons
Optimal BST
Palindrome Partitioning
```

---

# 56. Matrix Chain Multiplication

Given matrices:

```text
A × B × C
```

different parenthesizations can have different multiplication costs.

State:

```text
dp[i][j]
```

= minimum cost to multiply matrices `i...j`.

Try every split:

```text
k
```

between `i` and `j`.

This is classic interval DP.

---

# 57. Burst Balloons

Choose an order to burst balloons to maximize coins.

The difficult part is that removing a balloon changes its neighbors.

A useful transformation is:

```text
Instead of asking which balloon is burst first,
ask which balloon is burst last.
```

This creates a stable interval.

State:

```text
dp[left][right]
```

This is a classic advanced interval DP problem.

---

# 58. Palindrome Partitioning

Partition a string into the minimum number of palindromic pieces.

Possible DP states:

```text
dp[i] =
minimum cuts for prefix i
```

and:

```text
palindrome[i][j]
```

to determine whether substring `[i,j]` is a palindrome.

This is a good example of combining two DP tables.

---

# 59. Word Break

Given a string and a dictionary, determine whether the string can be segmented into dictionary words.

State:

```text
dp[i] =
whether prefix of length i
can be formed
```

For every possible previous split:

```text
if dp[j]
and substring(j, i) is in dictionary
```

then:

```text
dp[i] = true
```

---

# 60. Word Break — Java

```java
static boolean wordBreak(
        String s,
        Set<String> dictionary) {

    boolean[] dp =
        new boolean[s.length() + 1];

    dp[0] = true;

    for (int i = 1;
         i <= s.length();
         i++) {

        for (int j = 0;
             j < i;
             j++) {

            if (dp[j]
                    && dictionary.contains(
                        s.substring(j, i)
                    )) {

                dp[i] = true;
                break;
            }
        }
    }

    return dp[s.length()];
}
```

Complexity depends on substring/hash-set behavior, but the DP state structure is:

```text
O(n²)
```

candidate transitions.

---

# 61. Partition DP

Partition problems often ask:

```text
Can we split?
Minimum number of partitions?
Maximum score after partitioning?
```

Common state:

```text
dp[i]
```

or:

```text
dp[i][j]
```

depending on whether the partition is one-dimensional or interval-based.

---

# 62. Counting DP

Some DP problems ask:

```text
How many ways?
```

Examples:

```text
Climbing Stairs
Coin Change II
Decode Ways
Unique Paths
Target Sum
```

Typical transition:

```text
dp[state] += dp[previousState]
```

Be careful with:

```text
integer overflow
```

when counts become large.

---

# 63. Decode Ways

Given:

```text
"226"
```

Possible decodings:

```text
2 2 6
22 6
2 26
```

Answer:

```text
3
```

State:

```text
dp[i] =
number of ways to decode prefix i
```

At each position, consider:

```text
one-digit code
two-digit code
```

if valid.

---

# 64. Boolean DP

Some DP problems ask:

```text
Can this be achieved?
```

Use:

```text
boolean[]
```

or:

```text
boolean[][]
```

Examples:

```text
Subset Sum
Word Break
Partition Equal Subset Sum
Wildcard Matching
Regular Expression Matching
```

---

# 65. Min/Max DP

Optimization problems commonly use:

```text
min
```

or:

```text
max
```

Example:

```text
dp[i] =
minimum cost to reach i
```

or:

```text
dp[i] =
maximum profit up to i
```

---

# 66. DP with Strings

Common string DP problems:

```text
Longest Common Subsequence
Edit Distance
Longest Palindromic Subsequence
Longest Palindromic Substring
Word Break
Decode Ways
Wildcard Matching
Regex Matching
```

Most use:

```text
dp[i][j]
```

to represent two positions or prefixes.

---

# 67. Longest Palindromic Subsequence

A subsequence does not need to be contiguous.

State:

```text
dp[l][r]
```

If:

```text
s[l] == s[r]
```

then:

```text
dp[l][r] =
dp[l + 1][r - 1] + 2
```

Otherwise:

```text
max(
    dp[l + 1][r],
    dp[l][r - 1]
)
```

---

# 68. Longest Palindromic Substring

Here we need a contiguous substring.

A common DP state:

```text
dp[l][r] =
whether s[l...r] is palindrome
```

Transition:

```text
s[l] == s[r]
and
inner substring is palindrome
```

---

# 69. DP State Compression

Before optimizing memory, ask:

```text
What previous states are actually needed?
```

Examples:

```text
dp[i] depends on dp[i-1]
→ O(1) possible

dp[i] depends on dp[i-1] and dp[i-2]
→ O(1)

dp[i][j] depends only on previous row
→ O(n)
```

Do not optimize blindly.

First make the recurrence correct.

---

# 70. Top-Down vs Bottom-Up

### Top-Down

Advantages:

```text
Natural recursive structure
Only computes reachable states
Often easier to write initially
```

Disadvantages:

```text
Recursion stack
Potential stack overflow
```

### Bottom-Up

Advantages:

```text
No recursion
Usually efficient
Easy space optimization
```

Disadvantages:

```text
May compute states that are never needed
Requires correct iteration order
```

---

# 71. Memoization vs Tabulation

### Memoization

```text
Recursive
+
cache
```

### Tabulation

```text
Iterative
+
table
```

Both can have the same asymptotic complexity.

Choose whichever makes the state transition clearer.

---

# 72. How to Build a DP Solution in an Interview

Use this sequence:

```text
1. Explain the brute-force idea.
2. Identify repeated subproblems.
3. Define the state.
4. Write the recurrence.
5. Explain base cases.
6. Explain iteration/order.
7. Implement.
8. Analyze time and space.
9. Optimize memory if useful.
```

This makes your reasoning much easier to follow.

---

# 73. Common DP Mistakes

### Mistake 1 — Starting with code

First define:

```text
What does dp[i] mean?
```

---

### Mistake 2 — Wrong state

If the state does not contain enough information to make the next decision, the recurrence will fail.

---

### Mistake 3 — Missing base cases

Always define:

```text
dp[0]
```

and other smallest valid states.

---

### Mistake 4 — Wrong loop direction

Especially important in:

```text
0/1 Knapsack
Unbounded Knapsack
Coin Change
```

---

### Mistake 5 — Confusing subsequence and substring

```text
Subsequence → gaps allowed
Substring → contiguous
```

---

### Mistake 6 — Over-optimizing too early

First get:

```text
correct recurrence
```

then optimize memory/time.

---

# 74. DP Complexity

If there are:

```text
n states
```

and each state has:

```text
k transitions
```

complexity is usually:

```text
O(nk)
```

For a 2D DP:

```text
O(mn)
```

If each state tries all split points:

```text
O(n³)
```

may occur.

---

# 75. DP Problem Categories

```text
1. Linear DP
2. Grid DP
3. Knapsack DP
4. String DP
5. Subsequence DP
6. Interval DP
7. Tree DP
8. Bitmask DP
9. Digit DP
10. State-machine DP
11. Probability DP
12. DP on DAGs
```

---

# 76. Linear DP

State depends on previous positions.

Examples:

```text
Fibonacci
Climbing Stairs
House Robber
Decode Ways
Maximum Subarray
```

Typical:

```text
dp[i]
```

---

# 77. Grid DP

State depends on neighboring cells.

Examples:

```text
Unique Paths
Minimum Path Sum
Dungeon Game
Cherry Pickup
```

Typical:

```text
dp[row][col]
```

---

# 78. Knapsack DP

Typical state:

```text
dp[capacity]
```

or:

```text
dp[item][capacity]
```

Examples:

```text
0/1 Knapsack
Unbounded Knapsack
Coin Change
Subset Sum
Partition Equal Subset Sum
```

---

# 79. Subsequence DP

Examples:

```text
LIS
LCS
Longest Palindromic Subsequence
```

Typical states:

```text
dp[i]
```

or:

```text
dp[i][j]
```

---

# 80. Interval DP

Typical:

```text
dp[left][right]
```

Examples:

```text
Matrix Chain Multiplication
Burst Balloons
Palindrome Partitioning
Optimal BST
```

---

# 81. Tree DP

DP can also be performed on trees.

Example:

```text
House Robber III
```

For each node:

```text
rob this node
or
skip this node
```

State can be:

```text
[rob, skip]
```

This combines:

```text
Tree traversal
+
DP
```

---

# 82. DP on DAG

A DAG provides a natural dependency order.

You can:

```text
Topologically sort
↓
process nodes
↓
relax DP transitions
```

This is useful for:

```text
Longest path in DAG
Shortest path in DAG
Counting paths
Scheduling
```

---

# 83. Bitmask DP

Used when the state involves a subset of a small number of items.

Example:

```text
Traveling Salesman Problem
```

State:

```text
dp[mask][i]
```

where:

```text
mask = visited nodes
i = current node
```

Typical complexity:

```text
O(2^n × n)
```

or higher depending on transitions.

---

# 84. Digit DP

Used for counting numbers satisfying digit-related constraints over a range.

Typical state contains:

```text
position
tight
leadingZero
additional problem-specific state
```

This is an advanced topic.

---

# 85. State-Machine DP

Used when the problem has a small number of states.

Examples:

```text
Stock Trading
Cooldown
Transaction Fee
Buy/Sell
```

State examples:

```text
hold
cash
cooldown
```

These can often be optimized to:

```text
O(1) space
```

---

# 86. DP + Greedy

Some problems have multiple possible approaches.

Example:

```text
Jump Game
```

can be solved greedily.

Other variations may require:

```text
DP
```

The key is understanding the constraints and whether a greedy proof exists.

---

# 87. DP + Binary Search

Some problems combine:

```text
DP
+
binary search
```

Example:

```text
Longest Increasing Subsequence
```

The optimized LIS algorithm maintains candidate tails and uses binary search.

Other examples include:

```text
Weighted Interval Scheduling
```

where binary search finds the previous compatible interval.

---

# 88. Weighted Interval Scheduling

Each interval has:

```text
start
end
profit
```

Choose non-overlapping intervals with maximum total profit.

Sort by end time.

For each interval:

```text
take it
+
best compatible previous interval
```

or:

```text
skip it
```

State:

```text
dp[i]
```

and binary search finds the previous compatible interval.

---

# 89. DP + Prefix Sums

Some DP transitions require range sums.

Instead of:

```text
sum from l to r
```

in O(n), use prefix sums to make it:

```text
O(1)
```

This can reduce:

```text
O(n³)
```

to:

```text
O(n²)
```

in some problems.

---

# 90. How to Recognize the State

Ask:

> What information about the past is necessary to make the best decision for the future?

That information becomes the state.

Examples:

```text
House Robber:
current index

Knapsack:
index + capacity

LCS:
index in string A + index in string B

Stock:
day + holding state

Grid:
row + column

Bitmask DP:
visited set + current node
```

---

# 91. DP Recurrence Example

For House Robber:

```text
dp[i] =
max(
    skip current house,
    rob current house
)
```

Expanded:

```text
dp[i] =
max(
    dp[i - 1],
    dp[i - 2] + nums[i]
)
```

This is the essence of DP:

```text
state
+
transition
+
base case
```

---

# 92. DP Interview Checklist

Before coding:

```text
□ What is the state?
□ What does dp[i] mean?
□ What does dp[i][j] mean?
□ What are the choices?
□ What is the transition?
□ What are the base cases?
□ What is the answer state?
□ What is the computation order?
□ Can space be optimized?
□ What are the constraints?
```

---

# 93. Important DP Problems — Easy

1. Fibonacci.
2. Climbing Stairs.
3. Min Cost Climbing Stairs.
4. House Robber.
5. Maximum Subarray.
6. Unique Paths.
7. Minimum Path Sum.
8. Best Time to Buy and Sell Stock.
9. Pascal's Triangle.
10. Tribonacci.

---

# 94. Important DP Problems — Medium

11. House Robber II.
12. Coin Change.
13. Coin Change II.
14. Partition Equal Subset Sum.
15. Decode Ways.
16. Word Break.
17. Longest Increasing Subsequence.
18. Longest Common Subsequence.
19. Edit Distance.
20. Target Sum.
21. Combination Sum IV.
22. Longest Palindromic Subsequence.
23. Unique Paths II.
24. Triangle.
25. Interleaving String.

---

# 95. Important DP Problems — Advanced

26. 0/1 Knapsack.
27. Unbounded Knapsack.
28. Burst Balloons.
29. Matrix Chain Multiplication.
30. Palindrome Partitioning.
31. Weighted Interval Scheduling.
32. House Robber III.
33. Stock with Cooldown.
34. Stock with Transaction Fee.
35. Regular Expression Matching.
36. Wildcard Matching.
37. Distinct Subsequences.
38. Bitmask DP.
39. Digit DP.
40. DP on DAGs.

---

# 96. Complexity Summary

| Problem | Typical Time | Typical Space |
|---|---:|---:|
| Fibonacci | O(n) | O(1) optimized |
| Climbing Stairs | O(n) | O(1) |
| House Robber | O(n) | O(1) |
| Maximum Subarray | O(n) | O(1) |
| Unique Paths | O(mn) | O(n) optimized |
| Coin Change | O(amount × coins) | O(amount) |
| 0/1 Knapsack | O(n × capacity) | O(capacity) |
| LIS — DP | O(n²) | O(n) |
| LIS — optimized | O(n log n) | O(n) |
| LCS | O(mn) | O(mn) |
| Edit Distance | O(mn) | O(mn) |
| Word Break | O(n²) candidate transitions | O(n) |
| Interval DP | Often O(n³) | O(n²) |
| Bitmask DP | Often O(2ⁿ × n) | O(2ⁿ × n) |

---

# 97. Quick Revision

```text
Dynamic Programming
│
├── Approaches
│   ├── Top-Down
│   │   └── Memoization
│   └── Bottom-Up
│       └── Tabulation
│
├── Linear DP
│   ├── Fibonacci
│   ├── Climbing Stairs
│   ├── House Robber
│   └── Decode Ways
│
├── Grid DP
│   ├── Unique Paths
│   ├── Minimum Path Sum
│   └── Dungeon Game
│
├── Knapsack
│   ├── 0/1
│   ├── Unbounded
│   ├── Coin Change
│   └── Subset Sum
│
├── String DP
│   ├── LCS
│   ├── Edit Distance
│   ├── Word Break
│   └── Palindromes
│
├── Subsequence
│   ├── LIS
│   └── LCS
│
├── Interval DP
│   ├── Matrix Chain
│   └── Burst Balloons
│
├── State Machine
│   └── Stock Problems
│
└── Advanced
    ├── Tree DP
    ├── DAG DP
    ├── Bitmask DP
    └── Digit DP
```

---

## Interview Rule

> **The hardest part of DP is usually not writing the table — it is defining the correct state. Start by asking what information from the past is necessary to make the next decision. Then write the transition, base cases, and computation order.**

---

## DP Formula to Remember

```text
DP =
State
+
Transition
+
Base Case
+
Computation Order
```

When stuck, write these four things down before writing code.
