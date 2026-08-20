# DSA — Dynamic Programming

Dynamic Programming (DP) is a technique for solving problems by breaking them into smaller overlapping subproblems and storing their results so the same work is not repeated.

DP is one of the most important DSA topics for software engineering interviews.

Common DP patterns:

- Fibonacci / 1D DP
- Climbing Stairs
- House Robber
- 0/1 Knapsack
- Unbounded Knapsack
- Coin Change
- Subset Sum
- Partition DP
- Longest Common Subsequence
- Longest Increasing Subsequence
- Edit Distance
- Grid DP
- Interval DP
- String DP
- State-machine DP
- Bitmask DP

---

# 1. What Is Dynamic Programming?

DP is useful when a problem has:

```text
1. Overlapping subproblems
2. Optimal substructure
```

### Overlapping Subproblems

The same smaller problem is solved multiple times.

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

`fib(3)` and `fib(2)` are repeated.

DP stores their answers.

---

# 2. Optimal Substructure

A problem has optimal substructure when an optimal solution can be constructed from optimal solutions to smaller subproblems.

Examples:

```text
Shortest Path
Knapsack
LCS
Edit Distance
```

---

# 3. Recursion vs DP

A plain recursive solution may repeatedly solve the same state.

Example:

```java
fib(n - 1)
fib(n - 2)
```

DP changes this from:

```text
Repeated work
```

to:

```text
Solve once
↓
Store answer
↓
Reuse answer
```

---

# 4. Two Main DP Approaches

## Top-Down

```text
Recursion
+
Memoization
```

## Bottom-Up

```text
Iteration
+
DP table
```

Both represent the same underlying recurrence.

---

# 5. Memoization

Memoization stores the answer to each state.

Example:

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

---

# 6. Bottom-Up DP

Instead of recursion:

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

# 7. DP Complexity for Fibonacci

Naive recursion:

```text
Time: O(2^n)
Space: O(n)
```

Memoization:

```text
Time: O(n)
Space: O(n)
```

Bottom-up:

```text
Time: O(n)
Space: O(n)
```

Space can be optimized further.

---

# 8. Space Optimization

Fibonacci only needs the previous two values.

```java
static int fib(int n) {

    if (n <= 1) {
        return n;
    }

    int prev2 = 0;
    int prev1 = 1;

    for (int i = 2;
         i <= n;
         i++) {

        int current =
            prev1 + prev2;

        prev2 = prev1;
        prev1 = current;
    }

    return prev1;
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 9. How to Identify a DP Problem

Ask:

```text
Can the problem be divided into smaller states?
```

Then:

```text
Do the same states appear repeatedly?
```

Then:

```text
Can I store the answer for each state?
```

Common clues:

```text
maximum
minimum
number of ways
whether possible
longest
shortest
count
```

---

# 10. DP State

The state describes the information required to solve a subproblem.

Examples:

```text
dp[i]
```

means:

```text
answer up to index i
```

Or:

```text
dp[i][j]
```

might mean:

```text
answer using first i items
with capacity j
```

Choosing the correct state is often the hardest part of DP.

---

# 11. DP Transition

A transition describes how to calculate the current state from previous states.

Example:

```text
dp[i] =
dp[i - 1]
+
dp[i - 2]
```

The recurrence defines the relationship between states.

---

# 12. Base Cases

Every DP needs valid initial states.

For Fibonacci:

```text
dp[0] = 0
dp[1] = 1
```

For many problems, incorrect base cases cause the entire solution to fail.

---

# 13. Climbing Stairs

You can climb:

```text
1 step
or
2 steps
```

At step `n`, you could have come from:

```text
n - 1
```

or:

```text
n - 2
```

Therefore:

```text
dp[n] =
dp[n - 1]
+
dp[n - 2]
```

---

# 14. Climbing Stairs — Java

```java
static int climbStairs(
        int n) {

    if (n <= 2) {
        return n;
    }

    int prev2 = 1;
    int prev1 = 2;

    for (int i = 3;
         i <= n;
         i++) {

        int current =
            prev1 + prev2;

        prev2 = prev1;
        prev1 = current;
    }

    return prev1;
}
```

---

# 15. House Robber

You cannot rob two adjacent houses.

For each house:

```text
rob it
or
skip it
```

State:

```text
dp[i] = maximum money from first i houses
```

Transition:

```text
dp[i] =
max(
    dp[i - 1],
    dp[i - 2] + money[i]
)
```

---

# 16. House Robber — Java

```java
static int rob(
        int[] nums) {

    int prev2 = 0;
    int prev1 = 0;

    for (int money : nums) {

        int current =
            Math.max(
                prev1,
                prev2 + money
            );

        prev2 = prev1;
        prev1 = current;
    }

    return prev1;
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 17. House Robber II

In House Robber II, houses are arranged in a circle.

Therefore:

```text
first and last
```

are adjacent.

Break the problem into two cases:

```text
1. Rob houses [0 ... n-2]
2. Rob houses [1 ... n-1]
```

Answer:

```text
max(case1, case2)
```

---

# 18. 0/1 Knapsack

Given:

```text
weights
values
capacity
```

Each item can be selected:

```text
0 times
or
1 time
```

State:

```text
dp[i][capacity]
```

means maximum value using the first `i` items.

---

# 19. 0/1 Knapsack Transition

For an item with:

```text
weight = w
value = v
```

Either:

```text
do not take item
```

or:

```text
take item
```

Therefore:

```text
dp[i][c] =
max(
    dp[i-1][c],
    dp[i-1][c-w] + v
)
```

if:

```text
w <= c
```

---

# 20. 0/1 Knapsack — Java

```java
static int knapsack(
        int[] weights,
        int[] values,
        int capacity) {

    int n = weights.length;

    int[][] dp =
        new int[n + 1][capacity + 1];

    for (int i = 1;
         i <= n;
         i++) {

        int weight =
            weights[i - 1];

        int value =
            values[i - 1];

        for (int c = 0;
             c <= capacity;
             c++) {

            dp[i][c] =
                dp[i - 1][c];

            if (weight <= c) {

                dp[i][c] =
                    Math.max(
                        dp[i][c],
                        dp[i - 1][c - weight]
                            + value
                    );
            }
        }
    }

    return dp[n][capacity];
}
```

Complexity:

```text
Time: O(n × capacity)
Space: O(n × capacity)
```

---

# 21. 0/1 Knapsack Space Optimization

Use a 1D array:

```java
int[] dp =
    new int[capacity + 1];
```

Important:

```text
iterate capacity backwards
```

because each item can only be used once.

```java
for (int i = 0;
     i < n;
     i++) {

    for (int c = capacity;
         c >= weights[i];
         c--) {

        dp[c] =
            Math.max(
                dp[c],
                dp[c - weights[i]]
                    + values[i]
            );
    }
}
```

---

# 22. Why Iterate Backward in 0/1 Knapsack?

Backward iteration ensures the current item is not reused during the same iteration.

If you iterate forward:

```text
c = weight → capacity
```

the updated value can be used again.

That changes the problem into an unbounded version.

---

# 23. Unbounded Knapsack

Each item can be selected:

```text
multiple times
```

For a 1D DP solution, iterate capacity:

```text
forward
```

Example:

```java
for (int c = weight;
     c <= capacity;
     c++) {

    dp[c] =
        Math.max(
            dp[c],
            dp[c - weight] + value
        );
}
```

---

# 24. 0/1 vs Unbounded Knapsack

| Type | Item Usage | 1D Capacity Direction |
|---|---|---|
| 0/1 Knapsack | Once | Backward |
| Unbounded Knapsack | Unlimited | Forward |

This distinction is extremely important.

---

# 25. Coin Change

Given coin denominations and a target amount, find the minimum number of coins.

State:

```text
dp[amount]
```

Transition:

```text
dp[x] =
min(
    dp[x - coin] + 1
)
```

for every valid coin.

---

# 26. Coin Change — Java

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
O(amount × numberOfCoins)
```

---

# 27. Coin Change II

Instead of minimum coins, Coin Change II asks:

```text
How many combinations produce the amount?
```

This changes the state and iteration order.

Typical solution:

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

This counts combinations without treating different coin orders as different.

---

# 28. Subset Sum

Given an array, determine whether some subset sums to a target.

State:

```text
dp[s]
```

means:

```text
Can we make sum s?
```

For 0/1 subset selection:

```text
iterate sum backward
```

---

# 29. Subset Sum — Java

```java
static boolean canPartition(
        int[] nums,
        int target) {

    boolean[] dp =
        new boolean[target + 1];

    dp[0] = true;

    for (int num : nums) {

        for (int sum = target;
             sum >= num;
             sum--) {

            dp[sum] =
                dp[sum]
                || dp[sum - num];
        }
    }

    return dp[target];
}
```

---

# 30. Partition Equal Subset Sum

The total sum must be even.

```text
total % 2 == 0
```

Then target:

```text
total / 2
```

The problem becomes:

```text
Can a subset make target?
```

which is a subset-sum DP.

---

# 31. Longest Common Subsequence

Given:

```text
text1
text2
```

find the length of their longest common subsequence.

State:

```text
dp[i][j]
```

means:

```text
LCS of first i characters
and first j characters
```

---

# 32. LCS Transition

If:

```text
text1[i - 1]
==
text2[j - 1]
```

then:

```text
dp[i][j]
=
dp[i-1][j-1] + 1
```

Otherwise:

```text
dp[i][j]
=
max(
    dp[i-1][j],
    dp[i][j-1]
)
```

---

# 33. LCS — Java

```java
static int lcs(
        String a,
        String b) {

    int n = a.length();
    int m = b.length();

    int[][] dp =
        new int[n + 1][m + 1];

    for (int i = 1;
         i <= n;
         i++) {

        for (int j = 1;
             j <= m;
             j++) {

            if (a.charAt(i - 1)
                    == b.charAt(j - 1)) {

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

    return dp[n][m];
}
```

Complexity:

```text
Time: O(nm)
Space: O(nm)
```

---

# 34. Longest Common Substring

Do not confuse:

```text
subsequence
```

with:

```text
substring
```

Substring requires contiguous characters.

For matching characters:

```text
dp[i][j] =
dp[i-1][j-1] + 1
```

For mismatch:

```text
dp[i][j] = 0
```

---

# 35. Edit Distance

Operations:

```text
Insert
Delete
Replace
```

State:

```text
dp[i][j]
```

means minimum operations to transform:

```text
first i characters
```

into:

```text
first j characters
```

---

# 36. Edit Distance Transition

If characters match:

```text
dp[i][j] =
dp[i-1][j-1]
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

Specifically:

```text
insert:
dp[i][j-1]

delete:
dp[i-1][j]

replace:
dp[i-1][j-1]
```

---

# 37. Edit Distance — Java

```java
static int editDistance(
        String a,
        String b) {

    int n = a.length();
    int m = b.length();

    int[][] dp =
        new int[n + 1][m + 1];

    for (int i = 0;
         i <= n;
         i++) {

        dp[i][0] = i;
    }

    for (int j = 0;
         j <= m;
         j++) {

        dp[0][j] = j;
    }

    for (int i = 1;
         i <= n;
         i++) {

        for (int j = 1;
             j <= m;
             j++) {

            if (a.charAt(i - 1)
                    == b.charAt(j - 1)) {

                dp[i][j] =
                    dp[i - 1][j - 1];

            } else {

                dp[i][j] =
                    1 + Math.min(
                        dp[i - 1][j],
                        Math.min(
                            dp[i][j - 1],
                            dp[i - 1][j - 1]
                        )
                    );
            }
        }
    }

    return dp[n][m];
}
```

---

# 38. Longest Increasing Subsequence

Given:

```text
[10,9,2,5,3,7,101,18]
```

LIS length:

```text
4
```

For example:

```text
2,3,7,101
```

A simple DP solution:

```text
dp[i] =
length of LIS ending at i
```

---

# 39. LIS — O(n²) DP

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

            if (nums[j]
                    < nums[i]) {

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
O(n²)
```

---

# 40. LIS — O(n log n)

An optimized approach maintains:

```text
tails
```

Use:

```java
Arrays.binarySearch()
```

or manual lower-bound logic.

Important:

```text
tails is not necessarily the actual LIS.
```

It stores the smallest possible tail for each subsequence length.

Complexity:

```text
O(n log n)
```

---

# 41. Grid DP

Grid problems often define:

```text
dp[row][col]
```

as the answer for reaching or processing that cell.

Common problems:

```text
Unique Paths
Minimum Path Sum
Maximum Path Value
Dungeon Game
Cherry Pickup
```

---

# 42. Unique Paths

You can move:

```text
right
down
```

Number of ways to reach:

```text
(row, col)
```

is:

```text
dp[row][col]
=
dp[row-1][col]
+
dp[row][col-1]
```

---

# 43. Unique Paths — Java

```java
static int uniquePaths(
        int rows,
        int cols) {

    int[][] dp =
        new int[rows][cols];

    for (int r = 0;
         r < rows;
         r++) {

        dp[r][0] = 1;
    }

    for (int c = 0;
         c < cols;
         c++) {

        dp[0][c] = 1;
    }

    for (int r = 1;
         r < rows;
         r++) {

        for (int c = 1;
             c < cols;
             c++) {

            dp[r][c] =
                dp[r - 1][c]
                + dp[r][c - 1];
        }
    }

    return dp[rows - 1][cols - 1];
}
```

---

# 44. Minimum Path Sum

For each cell:

```text
dp[r][c]
=
grid[r][c]
+
min(
    dp[r-1][c],
    dp[r][c-1]
)
```

Initialize the first row and first column carefully.

---

# 45. Grid Obstacles

If a cell is blocked:

```text
dp[r][c] = 0
```

for counting-path problems.

The transition applies only to reachable cells.

---

# 46. State Machine DP

Some problems have states such as:

```text
holding stock
not holding stock
cooldown
transaction count
```

For example, stock trading problems can use:

```text
dp[day][state]
```

where state describes:

```text
holding / not holding
```

---

# 47. Stock Trading Example

For a simple unlimited-transaction stock problem:

```text
hold
cash
```

Transitions:

```text
newHold =
max(
    oldHold,
    oldCash - price
)

newCash =
max(
    oldCash,
    oldHold + price
)
```

This is a classic state-machine DP.

---

# 48. 2D DP State Design

When you see multiple changing variables:

```text
index
capacity
previous choice
transactions
position
remaining operations
```

the DP state may become:

```text
dp[i][j]
```

or:

```text
dp[i][j][k]
```

Always define exactly what each dimension represents.

---

# 49. Interval DP

Interval DP solves problems over ranges:

```text
[l, r]
```

State:

```text
dp[l][r]
```

Typical transition:

```text
choose split point k
```

Then:

```text
dp[l][r]
=
best(
    dp[l][k]
    +
    dp[k+1][r]
    +
    cost
)
```

---

# 50. Interval DP Examples

Common examples:

```text
Matrix Chain Multiplication
Burst Balloons
Palindrome Partitioning
Optimal BST
Minimum Cost to Cut a Stick
```

---

# 51. Matrix Chain Multiplication

Given matrices:

```text
A1 × A2 × A3 ...
```

different parenthesizations can have different costs.

State:

```text
dp[i][j]
```

means minimum multiplication cost for matrices:

```text
i ... j
```

Try every split:

```text
k = i ... j-1
```

---

# 52. Palindrome DP

A common state:

```text
dp[l][r]
```

means:

```text
is substring l...r a palindrome?
```

Transition:

```text
s[l] == s[r]
```

and:

```text
dp[l+1][r-1]
```

must also be true.

Base cases:

```text
length 1 → palindrome
length 2 → palindrome if characters match
```

---

# 53. Partition DP

Partition a sequence into groups and optimize:

```text
minimum cost
maximum score
number of ways
```

State may look like:

```text
dp[i]
```

or:

```text
dp[i][k]
```

where `k` is the number of partitions/groups.

---

# 54. DP with Counting

Sometimes DP stores:

```text
number of ways
```

instead of:

```text
minimum / maximum
```

Example:

```text
dp[i] =
number of ways to reach i
```

Be careful with integer overflow.

For large answers, use:

```java
long
```

or modulo arithmetic.

---

# 55. DP with Modulo

Many problems ask:

```text
answer % MOD
```

Use:

```java
static final long MOD =
    1_000_000_007L;
```

Then:

```java
dp[i] =
(dp[i - 1] + dp[i - 2])
% MOD;
```

Use `long` for intermediate arithmetic when multiplication may overflow `int`.

---

# 56. DP with Reconstruction

Sometimes the problem asks not only for the optimal value but also:

```text
which choices produced it?
```

Maintain:

```text
parent
choice
previous state
```

Then reconstruct the solution from the final state.

Examples:

```text
LCS sequence
Knapsack selected items
Shortest path
LIS sequence
```

---

# 57. DP vs Greedy

DP considers multiple possibilities and stores their best results.

Greedy makes a locally optimal choice.

Greedy works only when the problem has the necessary greedy-choice property.

If choosing locally can prevent a better future solution:

```text
DP
```

may be required.

---

# 58. DP vs Backtracking

Backtracking:

```text
explore choices
undo
```

can be exponential.

DP becomes useful when:

```text
many different paths reach the same state
```

Instead of solving that state repeatedly:

```text
solve once
cache result
```

---

# 59. DP vs Divide and Conquer

Divide and conquer usually creates independent subproblems.

DP is especially useful when subproblems:

```text
overlap
```

Example:

```text
Merge Sort
```

subproblems are largely independent.

```text
Fibonacci
```

subproblems overlap heavily.

---

# 60. Top-Down vs Bottom-Up

### Top-Down

Advantages:

```text
Natural recursive structure
Only computes reachable states
Easy to write from recurrence
```

Disadvantages:

```text
Recursion stack
Possible StackOverflowError
```

### Bottom-Up

Advantages:

```text
No recursion
Often easier to optimize
Good cache locality
```

Disadvantages:

```text
May compute states that are never needed
State ordering must be correct
```

---

# 61. How to Convert Recursion to DP

Start with:

```text
recursive function
```

Identify:

```text
parameters that define unique states
```

Then:

```text
memo[state] = answer
```

Finally, convert the state transitions into loops for bottom-up DP.

---

# 62. DP State Checklist

Before coding, write:

```text
What does dp[i] mean?
```

or:

```text
What does dp[i][j] mean?
```

Then:

```text
What are the base cases?
```

Then:

```text
How do I transition from smaller states?
```

Then:

```text
What order should states be calculated?
```

Then:

```text
What is the final answer?
```

---

# 63. Example DP Thought Process

Problem:

```text
Maximum money without robbing adjacent houses.
```

Ask:

```text
What changes as I move through the array?
```

Answer:

```text
index
```

State:

```text
dp[i] =
maximum money from first i houses
```

Choices:

```text
skip current
rob current
```

Transition:

```text
max(
    dp[i-1],
    dp[i-2] + nums[i]
)
```

Then optimize space.

This is the standard DP thought process.

---

# 64. Common DP Mistakes

### Mistake 1 — Wrong state

If the state does not contain enough information, the transition becomes invalid.

### Mistake 2 — Wrong base case

A wrong initial state corrupts the whole table.

### Mistake 3 — Wrong iteration order

Especially important for:

```text
0/1 Knapsack
Unbounded Knapsack
Coin Change
```

### Mistake 4 — Forgetting impossible states

Use an appropriate sentinel such as:

```text
INF
-1
false
```

depending on the problem.

### Mistake 5 — Integer overflow

Use:

```java
long
```

when required.

### Mistake 6 — Overusing multidimensional DP

First determine whether previous states can be compressed.

---

# 65. Space Optimization

If:

```text
dp[i]
```

depends only on:

```text
dp[i-1]
dp[i-2]
```

you may reduce:

```text
O(n)
```

space to:

```text
O(1)
```

For 2D DP, if row `i` depends only on row `i-1`, space may reduce from:

```text
O(nm)
```

to:

```text
O(m)
```

---

# 66. DP with Rolling Arrays

Example:

```java
int[] previous =
    new int[m];

int[] current =
    new int[m];
```

After each row:

```java
previous = current;
```

or reuse one array carefully.

This is common in:

```text
LCS
Edit Distance
Grid DP
```

---

# 67. DP and Monotonic Queues

Some advanced DP transitions contain:

```text
maximum/minimum over a sliding range
```

A monotonic deque can optimize them.

Pattern:

```text
DP
+
Sliding Window
+
Monotonic Queue
```

This can reduce some solutions from:

```text
O(nk)
```

to:

```text
O(n)
```

---

# 68. DP Optimization Techniques

Common optimizations include:

```text
Space compression
Prefix sums
Binary search
Monotonic queue
Monotonic stack
Deque optimization
Divide and conquer optimization
Knuth optimization
Convex Hull Trick
Bitmasking
```

Advanced optimizations depend heavily on the recurrence.

---

# 69. Bitmask DP

When the number of entities is small:

```text
n <= around 20
```

a bitmask can represent selected states.

Example:

```text
mask
```

represents:

```text
which nodes have been visited
```

State:

```text
dp[mask][last]
```

is common in:

```text
TSP
Hamiltonian path
Assignment problems
```

---

# 70. TSP Bitmask DP

State:

```text
dp[mask][i]
```

means:

```text
minimum cost to visit
all nodes in mask
and finish at i
```

Transition:

```text
choose an unvisited node j
```

```text
newMask =
mask | (1 << j)
```

Then:

```text
dp[newMask][j]
=
min(
    dp[newMask][j],
    dp[mask][i] + cost[i][j]
)
```

Typical complexity:

```text
O(n² × 2^n)
```

---

# 71. Tree DP

DP can also be performed on trees.

State examples:

```text
dp[node][0]
dp[node][1]
```

For example:

```text
0 → do not choose node
1 → choose node
```

The transition combines child states.

Common problems:

```text
House Robber III
Maximum independent set on tree
Tree diameter variants
Subtree optimization
```

---

# 72. Tree DP Example

For maximum sum with no adjacent selected tree nodes:

```text
dp[node][0] =
maximum if node is not selected

dp[node][1] =
maximum if node is selected
```

If node is selected:

```text
children cannot be selected
```

If node is not selected:

```text
each child may be selected or not
```

---

# 73. DAG DP

A DAG naturally supports dynamic programming because:

```text
there are no cycles
```

Topologically order the nodes:

```text
A → B → C
```

Then compute states in dependency order.

This is useful for:

```text
Longest path in DAG
Shortest path in DAG
Number of paths
Dependency scoring
```

---

# 74. Longest Path in DAG

For each node:

```text
dp[node] =
maximum path value ending at node
```

Process nodes in topological order.

For edge:

```text
u → v
```

transition:

```text
dp[v] =
max(
    dp[v],
    dp[u] + weight
)
```

Complexity:

```text
O(V + E)
```

---

# 75. DP Pattern Classification

When you encounter DP, try to classify it.

```text
1D DP
→ index-based

2D DP
→ two changing variables

Knapsack DP
→ item + capacity

String DP
→ two strings / indices

Grid DP
→ row + column

Interval DP
→ left + right

Tree DP
→ node + state

DAG DP
→ node in topological order

Bitmask DP
→ subset + current element

State Machine DP
→ state + time/index
```

---

# 76. DP Complexity

If there are:

```text
S states
```

and each state takes:

```text
T transition time
```

then:

```text
Total Time
≈
S × T
```

This is a useful general way to estimate DP complexity.

---

# 77. DP Interview Strategy

When asked to solve a DP problem:

```text
1. Start with brute force recursion.
2. Identify repeated states.
3. Define the state.
4. Write the recurrence.
5. Add memoization.
6. Analyze complexity.
7. Convert to bottom-up if useful.
8. Optimize space if possible.
```

This gives you a clear explanation during interviews.

---

# 78. DP Interview Questions — Easy

1. Fibonacci.
2. Climbing Stairs.
3. Min Cost Climbing Stairs.
4. House Robber.
5. Unique Paths.
6. Minimum Path Sum.
7. Maximum Subarray.
8. Decode Ways.

---

# 79. DP Interview Questions — Medium

9. House Robber II.
10. Coin Change.
11. Coin Change II.
12. 0/1 Knapsack.
13. Partition Equal Subset Sum.
14. Longest Increasing Subsequence.
15. Longest Common Subsequence.
16. Edit Distance.
17. Word Break.
18. Decode Ways.
19. Target Sum.
20. Longest Palindromic Subsequence.

---

# 80. DP Interview Questions — Advanced

21. Burst Balloons.
22. Matrix Chain Multiplication.
23. Palindrome Partitioning II.
24. Distinct Subsequences.
25. Regular Expression Matching.
26. Wildcard Matching.
27. TSP Bitmask DP.
28. Tree DP.
29. DAG DP.
30. State-machine stock problems.
31. Digit DP.
32. Interval DP.
33. Advanced partition DP.

---

# 81. Digit DP

Digit DP solves counting problems involving the digits of numbers within a range.

Typical state:

```text
position
tight
started
other constraints
```

Example questions:

```text
How many numbers <= N
satisfy a digit property?
```

Digit DP is an advanced topic and usually appears in harder algorithm interviews.

---

# 82. DP with Prefix Sums

If a transition repeatedly calculates:

```text
sum of a range
```

prefix sums can reduce the transition cost.

Instead of:

```text
sum(l...r)
```

in:

```text
O(n)
```

use prefix sums to answer it in:

```text
O(1)
```

This can significantly optimize DP.

---

# 83. DP and Modulo

For counting problems:

```java
static final long MOD =
    1_000_000_007L;
```

Use:

```java
dp[i] =
(dp[i] + value) % MOD;
```

For multiplication:

```java
dp[i] =
(dp[i] * value) % MOD;
```

Use `long` to avoid intermediate `int` overflow.

---

# 84. Impossible States

Suppose we want minimum cost.

Initialize impossible states to:

```text
INF
```

Example:

```java
long INF =
    Long.MAX_VALUE / 4;
```

Do not use:

```text
Long.MAX_VALUE
```

directly if you later add values, because that can overflow.

---

# 85. Maximum DP

For maximum problems, initialize appropriately.

If negative values are possible, avoid blindly using:

```text
0
```

because `0` may represent an invalid choice.

Instead use:

```text
negative infinity
```

or a valid base state.

---

# 86. Minimum DP

Similarly, do not use:

```text
0
```

for impossible minimum states.

Use:

```text
INF
```

and only transition from reachable states.

---

# 87. DP State Invariants

A powerful debugging technique is to write the invariant:

```text
dp[i] means exactly ______.
```

If you cannot clearly explain the meaning of a state, the DP design probably needs refinement.

---

# 88. Bottom-Up Ordering

For each transition, ensure all dependencies have already been calculated.

For example:

```text
dp[i] depends on dp[i-1]
```

then calculate:

```text
i = 1 → n
```

If:

```text
dp[i][j]
```

depends on:

```text
dp[i-1][j]
dp[i][j-1]
```

calculate rows and columns in an order that guarantees both are ready.

---

# 89. DP Table Visualization

For LCS:

```text
        ""  a  b  c
    ""   0  0  0  0
    a    0  1  1  1
    b    0  1  2  2
    c    0  1  2  3
```

Each cell represents the answer for a smaller problem.

Visualizing the table often makes the recurrence easier to understand.

---

# 90. Common DP Patterns to Memorize

```text
dp[i] = best answer up to i

dp[i] = number of ways to reach i

dp[i][j] = answer for first i / j elements

dp[capacity] = best answer for capacity

dp[l][r] = answer for interval l...r

dp[node][state] = tree DP

dp[mask][i] = subset DP

dp[day][state] = state-machine DP
```

---

# 91. DP Decision Tree

```text
Does the problem have repeated subproblems?
        |
       Yes
        ↓
Can the problem be described by a small state?
        |
       Yes
        ↓
Define dp[state]
        ↓
Find base cases
        ↓
Find transition
        ↓
Choose:
Top-Down or Bottom-Up
        ↓
Optimize space if possible
```

---

# 92. DP vs Greedy Interview Rule

If you can prove:

```text
local optimal choice
always leads to global optimum
```

greedy may work.

Otherwise:

```text
consider DP
```

Do not assume every optimization problem is DP.

---

# 93. DP vs Graph Algorithms

Many graph problems are also DP problems when the graph is:

```text
DAG
```

Because topological order provides a natural dependency order.

Example:

```text
Longest path in DAG
```

is a DP problem rather than the general NP-hard longest-path problem.

---

# 94. DP and Backend Engineering

DP is less commonly used directly in CRUD APIs, but the underlying thinking is valuable for:

```text
Optimization
Resource allocation
Scheduling
Caching
Dependency resolution
Recommendation systems
Cost minimization
Capacity planning
```

Understanding state, transitions, and trade-offs also helps with system-design reasoning.

---

# 95. Common Mistakes Checklist

Before submitting a DP solution:

```text
□ Is the state correctly defined?
□ Are base cases correct?
□ Are all transitions considered?
□ Are impossible states handled?
□ Is iteration order correct?
□ Can integer overflow happen?
□ Is modulo required?
□ Can space be optimized?
□ Is the final answer from the correct state?
□ Did I test smallest input?
□ Did I test duplicate values?
□ Did I test edge cases?
```

---

# 96. Complexity Summary

| Problem / Pattern | Typical Time | Typical Space |
|---|---:|---:|
| Fibonacci DP | O(n) | O(1) optimized |
| Climbing Stairs | O(n) | O(1) optimized |
| House Robber | O(n) | O(1) optimized |
| 0/1 Knapsack | O(nC) | O(C) optimized |
| Coin Change | O(amount × coins) | O(amount) |
| Subset Sum | O(n × target) | O(target) |
| LCS | O(nm) | O(nm) |
| Edit Distance | O(nm) | O(nm) |
| LIS | O(n²) / O(n log n) | O(n) |
| Grid DP | O(rows × cols) | O(rows × cols) |
| TSP Bitmask DP | O(n² × 2^n) | O(n × 2^n) |
| DAG DP | O(V + E) | O(V) |
| Tree DP | O(n) | O(n) |

---

# 97. Quick Revision

```text
Dynamic Programming
│
├── Fundamentals
│   ├── Overlapping Subproblems
│   ├── Optimal Substructure
│   ├── State
│   ├── Transition
│   └── Base Case
│
├── 1D DP
│   ├── Fibonacci
│   ├── Climbing Stairs
│   └── House Robber
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
│   ├── Palindrome DP
│   └── Word Break
│
├── Grid DP
│   ├── Unique Paths
│   └── Minimum Path Sum
│
├── Advanced
│   ├── Interval DP
│   ├── Tree DP
│   ├── DAG DP
│   ├── Bitmask DP
│   └── Digit DP
│
└── Optimization
    ├── Space Compression
    ├── Prefix Sums
    ├── Binary Search
    ├── Monotonic Queue
    └── Advanced DP Optimization
```

---

## Most Important DP Formula

For every DP problem, think:

```text
STATE
↓
BASE CASE
↓
TRANSITION
↓
ORDER
↓
ANSWER
```

Then ask:

```text
Can I reduce memory?
```

---

## Interview Rule

> **Don't start by writing a DP table. Start by defining what the state means. Once the state, base case, and transition are clear, the code is usually the easy part.**
