# DSA — Advanced Dynamic Programming

Advanced Dynamic Programming builds on the basic DP patterns from `24-dynamic-programming.md`.

The main goal is to recognize more complex state spaces and optimize transitions.

Topics covered:

- State-machine DP
- Interval DP
- Partition DP
- Tree DP
- DAG DP
- Bitmask DP
- Digit DP
- Palindrome DP
- Stock DP
- Advanced knapsack
- String matching DP
- DP optimization
- Monotonic queue optimization
- Prefix-sum optimization
- Reconstruction
- Advanced interview patterns

---

# 1. Advanced DP Mindset

For advanced DP, always define:

```text
State
↓
Base Case
↓
Transition
↓
Iteration Order
↓
Answer
```

Then ask:

```text
Can the state be reduced?
Can the transition be optimized?
Can memory be compressed?
```

---

# 2. State-Machine DP

State-machine DP is useful when the problem has a small number of conditions or modes.

Examples:

```text
Stock trading
Cooldown
Transaction limits
Holding / not holding
Buy / sell states
```

A state can represent:

```text
what happened previously
```

---

# 3. Stock Trading DP

For stock problems, a common state is:

```text
hold
cash
```

Meaning:

```text
hold = maximum profit while holding a stock
cash = maximum profit while not holding a stock
```

For price `p`:

```text
newHold =
max(oldHold, oldCash - p)

newCash =
max(oldCash, oldHold + p)
```

---

# 4. Stock Trading — Java

```java
static int maxProfit(int[] prices) {

    int hold = Integer.MIN_VALUE;
    int cash = 0;

    for (int price : prices) {

        int oldHold = hold;
        int oldCash = cash;

        hold =
            Math.max(
                oldHold,
                oldCash - price
            );

        cash =
            Math.max(
                oldCash,
                oldHold + price
            );
    }

    return cash;
}
```

Time:

```text
O(n)
```

Space:

```text
O(1)
```

---

# 5. Stock Trading with Cooldown

After selling a stock, you may need to wait one day before buying again.

Useful states:

```text
hold
sold
rest
```

Conceptually:

```text
hold → sold → rest → hold
```

This is a classic state-machine DP problem.

---

# 6. Stock Trading with Transaction Limit

If at most `k` transactions are allowed:

```text
dp[transaction][state]
```

can represent the current maximum profit.

A common state:

```text
dp[t][0] = not holding
dp[t][1] = holding
```

The transaction dimension tracks how many completed transactions are available.

---

# 7. Stock DP Complexity

For:

```text
n days
k transactions
```

a typical DP solution is:

```text
O(nk)
```

space can often be reduced to:

```text
O(k)
```

---

# 8. Interval DP

Interval DP works on ranges:

```text
[l, r]
```

State:

```text
dp[l][r]
```

means:

```text
best answer for interval l...r
```

Transitions often choose a split:

```text
k
```

between `l` and `r`.

---

# 9. Generic Interval DP

Typical structure:

```java
for (int length = 2;
     length <= n;
     length++) {

    for (int left = 0;
         left + length <= n;
         left++) {

        int right =
            left + length - 1;

        for (int k = left;
             k < right;
             k++) {

            // Transition
        }
    }
}
```

The important point is:

```text
shorter intervals must be solved first.
```

---

# 10. Matrix Chain Multiplication

Given matrices:

```text
A1 × A2 × A3 × ... × An
```

different parenthesizations can have different multiplication costs.

State:

```text
dp[i][j]
```

means minimum cost to multiply:

```text
Ai ... Aj
```

---

# 11. Matrix Chain Transition

Try every split:

```text
i ... k
k+1 ... j
```

Transition:

```text
dp[i][j] =
min(
    dp[i][k]
    + dp[k+1][j]
    + multiplicationCost
)
```

Complexity:

```text
O(n³)
```

---

# 12. Burst Balloons

Bursting a balloon changes the neighbors of the remaining balloons.

Instead of choosing:

```text
which balloon to burst first
```

choose:

```text
which balloon is burst last
```

This removes dependency on changing neighbors.

State:

```text
dp[l][r]
```

means maximum coins from interval `l...r`.

---

# 13. Burst Balloons Transition

For each possible last balloon `k`:

```text
dp[l][r] =
max(
    dp[l][k-1]
    + dp[k+1][r]
    + nums[l-1] * nums[k] * nums[r+1]
)
```

This is a classic interval-DP transformation.

---

# 14. Palindrome DP

A common state:

```text
dp[l][r]
```

means:

```text
whether s[l...r] is a palindrome
```

Transition:

```text
s[l] == s[r]
```

and:

```text
dp[l + 1][r - 1]
```

must be true.

---

# 15. Palindrome DP — Java

```java
static boolean[][] palindromeTable(
        String s) {

    int n = s.length();

    boolean[][] dp =
        new boolean[n][n];

    for (int length = 1;
         length <= n;
         length++) {

        for (int left = 0;
             left + length <= n;
             left++) {

            int right =
                left + length - 1;

            if (length <= 2) {

                dp[left][right] =
                    s.charAt(left)
                    == s.charAt(right);

            } else {

                dp[left][right] =
                    s.charAt(left)
                    == s.charAt(right)
                    && dp[left + 1][right - 1];
            }
        }
    }

    return dp;
}
```

---

# 16. Longest Palindromic Subsequence

State:

```text
dp[l][r]
```

means longest palindromic subsequence inside:

```text
l...r
```

If:

```text
s[l] == s[r]
```

then:

```text
dp[l][r]
=
dp[l+1][r-1] + 2
```

Otherwise:

```text
dp[l][r]
=
max(
    dp[l+1][r],
    dp[l][r-1]
)
```

---

# 17. Palindromic Substring vs Subsequence

### Substring

Characters must be contiguous.

### Subsequence

Characters do not have to be contiguous.

Example:

```text
abcba
```

Both are possible.

But:

```text
abcde
```

can have a subsequence:

```text
ace
```

while `ace` is not a substring.

---

# 18. Partition DP

Partition DP divides a sequence into several parts.

Typical state:

```text
dp[i][k]
```

meaning:

```text
best answer for first i elements
using k partitions
```

Transition:

```text
try previous partition point j
```

---

# 19. Generic Partition DP

Conceptually:

```java
dp[i][k] =
    best(
        dp[j][k - 1]
        + cost(j, i)
    );
```

Try:

```text
j = 0 ... i - 1
```

Naive complexity can become:

```text
O(k n²)
```

or worse depending on the cost calculation.

---

# 20. Minimum Cost to Cut a Stick

Suppose a stick has cuts at certain positions.

Choosing the order of cuts affects total cost.

This can be modeled as interval DP.

State:

```text
dp[l][r]
```

means minimum cost to perform all cuts between:

```text
l and r
```

Try each cut as the first or last cut.

---

# 21. Tree DP

Trees are naturally recursive, making them excellent candidates for DP.

State:

```text
dp[node][state]
```

where `state` describes a condition on the node.

Examples:

```text
selected
not selected
parent selected
parent not selected
```

---

# 22. Tree DP Example — Independent Set

Suppose adjacent nodes cannot both be selected.

For each node:

```text
dp[node][0] =
maximum value if node is not selected

dp[node][1] =
maximum value if node is selected
```

If node is selected:

```text
children cannot be selected
```

---

# 23. Tree DP Transition

For each child:

```text
dp[node][0] +=
    max(
        dp[child][0],
        dp[child][1]
    );
```

If node is selected:

```text
dp[node][1] +=
    dp[child][0];
```

Then:

```text
answer =
max(
    dp[root][0],
    dp[root][1]
)
```

---

# 24. Tree DP — Java

```java
static int[] dfs(
        int node,
        int parent,
        List<List<Integer>> graph,
        int[] value) {

    int notTake = 0;
    int take = value[node];

    for (int child :
            graph.get(node)) {

        if (child == parent) {
            continue;
        }

        int[] childDp =
            dfs(
                child,
                node,
                graph,
                value
            );

        notTake +=
            Math.max(
                childDp[0],
                childDp[1]
            );

        take +=
            childDp[0];
    }

    return new int[]{
        notTake,
        take
    };
}
```

---

# 25. DAG DP

A Directed Acyclic Graph naturally supports DP because there are no cycles.

First compute:

```text
topological order
```

Then process nodes in that order.

---

# 26. Longest Path in DAG

For weighted edges:

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

Because the graph is acyclic, this can be solved in:

```text
O(V + E)
```

---

# 27. Number of Paths in DAG

Set:

```text
dp[source] = 1
```

For each edge:

```text
u → v
```

update:

```text
dp[v] += dp[u]
```

after processing nodes in topological order.

This counts the number of paths from the source.

---

# 28. Bitmask DP

Bitmask DP is useful when the number of entities is small.

A bitmask represents a subset.

For:

```text
n = 4
```

mask:

```text
0101
```

means:

```text
elements 0 and 2 are selected.
```

---

# 29. Bitmask DP State

A common state:

```text
dp[mask][last]
```

means:

```text
best result after visiting
the nodes in mask
and ending at last.
```

This is common in:

```text
TSP
Hamiltonian path
Assignment problems
Subset optimization
```

---

# 30. TSP Bitmask DP

For Traveling Salesman Problem:

```text
dp[mask][last]
```

represents the minimum cost to:

```text
visit all nodes in mask
and finish at last.
```

Transition:

```text
for every unvisited next node:
    newMask = mask | (1 << next)
```

---

# 31. TSP Transition

```java
for (int mask = 0;
     mask < (1 << n);
     mask++) {

    for (int last = 0;
         last < n;
         last++) {

        if ((mask & (1 << last)) == 0) {
            continue;
        }

        for (int next = 0;
             next < n;
             next++) {

            if ((mask & (1 << next)) != 0) {
                continue;
            }

            int newMask =
                mask | (1 << next);

            dp[newMask][next] =
                Math.min(
                    dp[newMask][next],
                    dp[mask][last]
                        + cost[last][next]
                );
        }
    }
}
```

Typical complexity:

```text
O(n² × 2^n)
```

---

# 32. Digit DP

Digit DP solves counting problems over integer ranges based on digit properties.

Typical question:

```text
How many numbers from 0 to N
satisfy a property?
```

Instead of iterating over every number, process:

```text
one digit at a time.
```

---

# 33. Digit DP State

A common state contains:

```text
position
tight
started
additional constraint
```

For example:

```text
dp[pos][tight][sum]
```

---

# 34. Tight State

`tight` tells us whether the digits chosen so far are exactly equal to the prefix of `N`.

If:

```text
tight = true
```

the next digit cannot exceed the corresponding digit of `N`.

If:

```text
tight = false
```

the next digit can usually range from:

```text
0 ... 9
```

---

# 35. Started State

`started` handles leading zeros.

For example, when representing:

```text
7
```

using the same number of digits as:

```text
123
```

we may represent it as:

```text
007
```

The leading zeros should often not count as actual digits.

---

# 36. Digit DP Example State

```java
long solve(
        int pos,
        boolean tight,
        boolean started,
        int sum) {

    // Memoize state.
}
```

The exact state depends on the problem.

---

# 37. Advanced Knapsack

Knapsack has many variations:

```text
0/1
Unbounded
Bounded
Multiple-choice
Group knapsack
Exact-fill
At-most capacity
Minimum cost
Maximum value
```

The most important skill is recognizing:

```text
item dimension
+
capacity dimension
```

---

# 38. Bounded Knapsack

Each item has a limited count.

Example:

```text
item A → maximum 3 copies
item B → maximum 2 copies
```

A straightforward solution loops over:

```text
how many copies to take
```

but advanced optimizations can reduce the complexity.

---

# 39. Multiple-Choice Knapsack

Items are divided into groups.

You can select:

```text
at most one item from each group.
```

State:

```text
dp[group][capacity]
```

For every group:

```text
try each candidate item.
```

---

# 40. Exact-Fill Knapsack

Sometimes the capacity must be filled exactly.

In this case, initialize unreachable states carefully.

For example:

```java
Arrays.fill(
    dp,
    Integer.MIN_VALUE
);

dp[0] = 0;
```

Then only reachable capacities should transition.

---

# 41. String DP

String problems often use:

```text
dp[i][j]
```

where:

```text
i = position in first string
j = position in second string
```

Examples:

```text
LCS
Edit Distance
Wildcard Matching
Regex Matching
Distinct Subsequences
Interleaving String
```

---

# 42. Distinct Subsequences

Question:

```text
How many ways can string t
be formed as a subsequence of s?
```

State:

```text
dp[i][j]
```

means number of ways to form first `j` characters of `t` using first `i` characters of `s`.

If characters match:

```text
dp[i][j] =
dp[i-1][j]
+
dp[i-1][j-1]
```

Otherwise:

```text
dp[i][j] =
dp[i-1][j]
```

---

# 43. Interleaving String

Determine whether:

```text
s3
```

can be formed by interleaving:

```text
s1
s2
```

State:

```text
dp[i][j]
```

means whether the first:

```text
i characters of s1
j characters of s2
```

can form the corresponding prefix of `s3`.

---

# 44. Wildcard Matching

Typical symbols:

```text
?
*
```

where:

```text
? → matches one character
* → matches zero or more characters
```

State:

```text
dp[i][j]
```

represents whether prefixes match.

For `*`, transitions may include:

```text
match zero characters
or
consume one character
```

---

# 45. Regular Expression Matching

Typical symbols:

```text
.
*
```

The state depends on:

```text
string index
pattern index
```

The `*` operator creates multiple possible transitions.

This is a classic advanced string DP problem.

---

# 46. DP with Reconstruction

Sometimes we need the actual choices, not just the optimal value.

Maintain:

```text
parent
choice
previous state
```

Example:

```text
Knapsack
```

After computing the optimal value, walk backward through the DP table to determine which items were selected.

---

# 47. LCS Reconstruction

After computing:

```text
dp[i][j]
```

start from:

```text
i = n
j = m
```

If:

```text
a[i-1] == b[j-1]
```

add the character and move:

```text
i--
j--
```

Otherwise move toward the larger DP value:

```text
dp[i-1][j]
```

or:

```text
dp[i][j-1]
```

Finally reverse the result.

---

# 48. LIS Reconstruction

For LIS, maintain:

```text
parent[i]
```

When:

```text
dp[i] =
dp[j] + 1
```

set:

```java
parent[i] = j;
```

Then start from the index with the maximum LIS length and follow the parent pointers backward.

---

# 49. Prefix Sum Optimization

Suppose a transition requires:

```text
dp[i] =
sum(dp[j])
```

over a range.

Naively:

```text
O(n)
```

per state.

Using prefix sums:

```text
prefix[i] =
prefix[i-1] + dp[i]
```

the range sum can become:

```text
O(1)
```

This can reduce:

```text
O(n²)
```

to:

```text
O(n)
```

in appropriate problems.

---

# 50. Monotonic Queue Optimization

Suppose:

```text
dp[i]
```

depends on:

```text
maximum/minimum of dp[j]
```

within a sliding window.

A monotonic deque can maintain the best candidate.

Typical improvement:

```text
O(nk)
```

→

```text
O(n)
```

for a window of size `k`.

---

# 51. DP + Binary Search

Some DP problems can be optimized with binary search.

Classic example:

```text
Longest Increasing Subsequence
```

The `tails` method maintains the smallest possible ending value for each subsequence length.

Each new value uses:

```text
binary search
```

to find its position.

Complexity:

```text
O(n log n)
```

---

# 52. Convex Hull Trick

The Convex Hull Trick optimizes DP transitions involving lines.

A common form is:

```text
dp[i] =
min(
    dp[j]
    + m[j] * x[i]
    + b[j]
)
```

Each previous state creates a line:

```text
y = mx + b
```

The data structure maintains only useful lines.

This is an advanced competitive-programming optimization.

---

# 53. Divide and Conquer DP Optimization

Some partition DP problems have special structure allowing:

```text
O(kn²)
```

to be reduced.

The optimization relies on properties of the optimal split positions.

Do not apply it unless the problem satisfies the required monotonicity conditions.

---

# 54. Knuth Optimization

Knuth optimization can reduce certain interval DP problems.

It relies on specific properties of the cost function and optimal split points.

Typical context:

```text
interval DP
optimal binary search tree
related partition problems
```

It is an advanced topic and should be learned after mastering ordinary interval DP.

---

# 55. State Compression

Sometimes a DP state contains information that can be represented more compactly.

Examples:

```text
boolean[] used
```

can become:

```text
bitmask
```

A large 2D table may become:

```text
1D rolling array
```

State compression can dramatically reduce memory.

---

# 56. Sparse DP

Not every possible state may be reachable.

Instead of allocating a huge multidimensional array, use:

```text
HashMap<State, Value>
```

or another sparse representation.

This can be useful when:

```text
state space is large
but reachable states are few.
```

---

# 57. Memoization with Multiple Parameters

For recursion:

```java
solve(index, capacity, state)
```

the complete state consists of all parameters that affect future decisions.

If:

```text
index
capacity
state
```

can change independently, all may need to be part of the memoization key.

Never memoize only part of the state.

---

# 58. DP with HashMap

For irregular state spaces:

```java
Map<String, Integer> memo =
    new HashMap<>();
```

or preferably a structured key.

For performance-sensitive problems, arrays are usually faster when dimensions are bounded.

---

# 59. Advanced DP Debugging

When a DP answer is wrong, check in this order:

```text
1. State definition
2. Base cases
3. Transition
4. Iteration order
5. Impossible-state initialization
6. Integer overflow
7. Final answer state
```

Do not immediately rewrite the entire solution.

---

# 60. Advanced DP Complexity

If:

```text
number of states = S
```

and:

```text
transitions per state = T
```

then:

```text
Time ≈ O(S × T)
```

This is the most useful general rule for estimating DP complexity.

---

# 61. Advanced DP Interview Questions

### State Machine

1. Best Time to Buy and Sell Stock.
2. Stock with Cooldown.
3. Stock with Transaction Fee.
4. Stock with K Transactions.

### Interval DP

5. Matrix Chain Multiplication.
6. Burst Balloons.
7. Minimum Cost to Cut a Stick.
8. Palindrome Partitioning.

### Tree DP

9. House Robber III.
10. Maximum Independent Set on Tree.
11. Tree matching variants.

### Bitmask DP

12. Traveling Salesman Problem.
13. Hamiltonian Path.
14. Assignment Problem.

### Digit DP

15. Count numbers with digit constraints.
16. Count numbers without repeated digits.
17. Digit sum constraints.

---

# 62. More Advanced Interview Questions

18. Distinct Subsequences.
19. Interleaving String.
20. Wildcard Matching.
21. Regular Expression Matching.
22. Boolean Parenthesization.
23. Optimal Binary Search Tree.
24. Palindrome Partitioning II.
25. Advanced partition DP.
26. DP with monotonic queue.
27. DP with convex hull trick.

---

# 63. DP Pattern Recognition

When you see:

```text
maximum/minimum over a sequence
```

consider:

```text
1D DP
```

When you see:

```text
two strings
```

consider:

```text
2D string DP
```

When you see:

```text
capacity
```

consider:

```text
Knapsack DP
```

When you see:

```text
range [l, r]
```

consider:

```text
Interval DP
```

When you see:

```text
tree + choices
```

consider:

```text
Tree DP
```

When you see:

```text
small n + subsets
```

consider:

```text
Bitmask DP
```

When you see:

```text
count numbers <= N
```

consider:

```text
Digit DP
```

---

# 64. Advanced DP Decision Tree

```text
Is there a small changing state?
        |
       Yes
        ↓
Can previous states be reused?
        |
       Yes
        ↓
Define DP state
        |
        +-----------------------------+
        |                             |
     Sequence                      Range
        |                             |
      1D DP                       Interval DP
        |
        +------------+
        |            |
     Capacity      Strings
        |            |
    Knapsack      String DP

Tree
 ↓
Tree DP

DAG
 ↓
DAG DP

Small subset
 ↓
Bitmask DP

Numbers up to N
 ↓
Digit DP
```

---

# 65. Advanced DP Best Practices

Prefer:

```text
clear state definitions
```

over clever code.

Prefer:

```text
long
```

when counts or costs can exceed `int`.

Use:

```text
INF = Long.MAX_VALUE / 4
```

instead of `Long.MAX_VALUE` when addition is involved.

Use:

```text
1L
```

for long arithmetic.

Keep transitions explicit.

---

# 66. DP Space Optimization Checklist

Ask:

```text
Does dp[i] depend only on dp[i-1]?
```

If yes:

```text
O(n) → O(1)
```

Ask:

```text
Does row i depend only on row i-1?
```

If yes:

```text
O(nm) → O(m)
```

Ask:

```text
Can the state be represented as a bitmask?
```

If yes:

```text
boolean array → integer/long mask
```

---

# 67. Common Advanced DP Mistakes

### Mistake 1 — Overcomplicating the state

Store only information that affects future decisions.

### Mistake 2 — Using recursion without memoization

This often causes exponential time.

### Mistake 3 — Wrong interval order

For `dp[l][r]`, shorter intervals usually need to be computed first.

### Mistake 4 — Wrong knapsack iteration direction

```text
0/1 → backward
unbounded → forward
```

### Mistake 5 — Ignoring overflow

Use `long` when needed.

### Mistake 6 — Applying advanced optimization without proof

Do not use:

```text
Knuth
Convex Hull Trick
Divide-and-Conquer Optimization
```

unless the required mathematical conditions hold.

---

# 68. Complexity Summary

| Pattern | Typical Complexity |
|---|---:|
| Stock state DP | O(n) |
| Stock with K transactions | O(nk) |
| Interval DP | O(n³) |
| Matrix Chain Multiplication | O(n³) |
| Tree DP | O(n) |
| DAG DP | O(V + E) |
| Bitmask DP | O(n² × 2^n) |
| Digit DP | O(digits × states × 10) |
| LCS | O(nm) |
| Edit Distance | O(nm) |
| LIS optimized | O(n log n) |
| Prefix-sum DP optimization | Often reduces O(n²) to O(n) |
| Monotonic queue DP | Often O(n) |
| Convex Hull Trick DP | Often O(n log n) or O(n) depending on implementation |

---

# 69. Quick Revision

```text
Advanced Dynamic Programming
│
├── State Machine
│   ├── Stock
│   ├── Cooldown
│   └── Transactions
│
├── Interval DP
│   ├── Matrix Chain
│   ├── Burst Balloons
│   ├── Cutting Problems
│   └── Palindrome Partitioning
│
├── Tree DP
│   ├── Node + State
│   └── Independent Set
│
├── DAG DP
│   ├── Longest Path
│   ├── Shortest Path
│   └── Path Counting
│
├── Bitmask DP
│   ├── TSP
│   ├── Hamiltonian Path
│   └── Assignment
│
├── Digit DP
│   ├── Position
│   ├── Tight
│   ├── Started
│   └── Constraint
│
├── String DP
│   ├── LCS
│   ├── Edit Distance
│   ├── Distinct Subsequences
│   └── Matching
│
└── Optimization
    ├── Prefix Sums
    ├── Binary Search
    ├── Monotonic Queue
    ├── Convex Hull Trick
    ├── Divide & Conquer
    └── Knuth Optimization
```

---

# 70. Most Important Advanced DP Rules

```text
Sequence → 1D DP

Two strings → 2D DP

Capacity → Knapsack

Range → Interval DP

Tree → Tree DP

DAG → Topological DP

Small N + subset → Bitmask DP

Numbers <= N → Digit DP

State changes over time → State-machine DP
```

---

## Interview Rule

> **Advanced DP is mostly about recognizing the state. Once you can clearly say what `dp[state]` means, identify the base cases, and explain how one state transitions into another, the implementation becomes much easier.**
