# DSA — Recursion & Backtracking

Recursion is a technique where a function solves a problem by calling itself on a smaller version of the same problem.

Backtracking is an extension of recursion where we:

```text
Choose
↓
Explore
↓
Undo
↓
Choose another option
```

These patterns are extremely important for Java interviews and are commonly used in:

- Trees
- Graphs
- Permutations
- Combinations
- Subsets
- N-Queens
- Sudoku
- Maze problems
- Generate Parentheses
- Word Search
- Constraint satisfaction

---

# 1. What is Recursion?

A recursive function calls itself.

Example:

```java
static void printNumbers(int n) {

    if (n == 0) {
        return;
    }

    System.out.println(n);

    printNumbers(n - 1);
}
```

Calling:

```java
printNumbers(5);
```

produces:

```text
5
4
3
2
1
```

---

# 2. Two Essential Parts of Recursion

Every recursive solution needs:

```text
Base Case
+
Recursive Case
```

### Base Case

Stops recursion.

```java
if (n == 0) {
    return;
}
```

### Recursive Case

Moves toward the base case.

```java
printNumbers(n - 1);
```

Without a valid base case, recursion can continue until:

```text
StackOverflowError
```

---

# 3. Recursion Call Stack

For:

```java
factorial(4)
```

the calls build up:

```text
factorial(4)
factorial(3)
factorial(2)
factorial(1)
```

Then results return:

```text
factorial(1)
↓
factorial(2)
↓
factorial(3)
↓
factorial(4)
```

The JVM uses the call stack to keep track of active method calls.

---

# 4. Factorial

Mathematically:

```text
n! = n × (n - 1)!
```

Base case:

```text
0! = 1
```

Java:

```java
static long factorial(int n) {

    if (n <= 1) {
        return 1;
    }

    return n *
        factorial(n - 1);
}
```

Complexity:

```text
Time:  O(n)
Space: O(n)
```

because of recursion depth.

---

# 5. Sum of First N Numbers

```java
static int sum(int n) {

    if (n == 0) {
        return 0;
    }

    return n + sum(n - 1);
}
```

For:

```text
sum(5)
```

the result is:

```text
5 + 4 + 3 + 2 + 1 = 15
```

---

# 6. Power

Calculate:

```text
x^n
```

Basic recursion:

```java
static long power(
        long x,
        int n) {

    if (n == 0) {
        return 1;
    }

    return x *
        power(x, n - 1);
}
```

Complexity:

```text
O(n)
```

---

# 7. Fast Power

We can reduce the complexity using:

```text
x^n
```

If `n` is even:

```text
x^n =
(x^(n/2))²
```

If `n` is odd:

```text
x^n =
x × x^(n-1)
```

Java:

```java
static long fastPower(
        long x,
        long n) {

    if (n == 0) {
        return 1;
    }

    long half =
        fastPower(x, n / 2);

    long result =
        half * half;

    if (n % 2 != 0) {
        result *= x;
    }

    return result;
}
```

Complexity:

```text
Time: O(log n)
Space: O(log n)
```

---

# 8. Fibonacci

Definition:

```text
F(n) =
F(n - 1) + F(n - 2)
```

Base cases:

```text
F(0) = 0
F(1) = 1
```

Naive recursive solution:

```java
static int fibonacci(int n) {

    if (n <= 1) {
        return n;
    }

    return fibonacci(n - 1)
        + fibonacci(n - 2);
}
```

---

# 9. Why Naive Fibonacci Is Slow

The same values are calculated repeatedly.

For example:

```text
F(5)
├── F(4)
│   ├── F(3)
│   └── F(2)
└── F(3)
    ├── F(2)
    └── F(1)
```

Complexity is approximately:

```text
O(2^n)
```

This is a classic introduction to:

```text
Dynamic Programming
```

---

# 10. Recursion with Memoization

Store already-computed results.

```java
static int fibonacci(
        int n,
        int[] memo) {

    if (n <= 1) {
        return n;
    }

    if (memo[n] != -1) {
        return memo[n];
    }

    memo[n] =
        fibonacci(n - 1, memo)
        + fibonacci(n - 2, memo);

    return memo[n];
}
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

---

# 11. Recursion vs Iteration

### Recursion

Advantages:

```text
Natural for trees
Natural for divide-and-conquer
Natural for backtracking
Readable for recursive definitions
```

Disadvantages:

```text
Uses call stack
Can cause StackOverflowError
May use more memory
```

### Iteration

Advantages:

```text
Usually constant auxiliary stack space
Often safer for very deep inputs
```

Choose based on the problem rather than using recursion automatically.

---

# 12. What is Backtracking?

Backtracking explores possible choices.

General structure:

```text
Choose
↓
Explore
↓
Undo
```

Example:

```text
Choose A
Explore A
Undo A

Choose B
Explore B
Undo B
```

This allows us to systematically explore a search space.

---

# 13. Generic Backtracking Template

```java
static void backtrack(
        List<Integer> path) {

    if (isComplete(path)) {

        process(path);
        return;
    }

    for (int choice : choices) {

        if (!isValid(choice)) {
            continue;
        }

        path.add(choice);

        backtrack(path);

        path.remove(
            path.size() - 1
        );
    }
}
```

The last line is the crucial:

```text
UNDO
```

operation.

---

# 14. Subsets

Given:

```text
[1, 2, 3]
```

generate:

```text
[]
[1]
[2]
[3]
[1,2]
[1,3]
[2,3]
[1,2,3]
```

There are:

```text
2^n
```

subsets.

---

# 15. Subsets — Java

```java
static List<List<Integer>> subsets(
        int[] nums) {

    List<List<Integer>> result =
        new ArrayList<>();

    backtrackSubsets(
        nums,
        0,
        new ArrayList<>(),
        result
    );

    return result;
}

static void backtrackSubsets(
        int[] nums,
        int index,
        List<Integer> current,
        List<List<Integer>> result) {

    result.add(
        new ArrayList<>(current)
    );

    for (int i = index;
         i < nums.length;
         i++) {

        current.add(nums[i]);

        backtrackSubsets(
            nums,
            i + 1,
            current,
            result
        );

        current.remove(
            current.size() - 1
        );
    }
}
```

---

# 16. Why Copy the Current List?

This is important:

```java
result.add(
    new ArrayList<>(current)
);
```

Do not simply do:

```java
result.add(current);
```

because `current` continues changing during backtracking.

You need to store a snapshot.

---

# 17. Subsets with Duplicates

Input:

```text
[1,2,2]
```

may produce duplicate subsets.

Sort first:

```java
Arrays.sort(nums);
```

Then skip duplicates at the same recursion level:

```java
if (i > index
        && nums[i] == nums[i - 1]) {
    continue;
}
```

---

# 18. Permutations

For:

```text
[1,2,3]
```

generate:

```text
[1,2,3]
[1,3,2]
[2,1,3]
[2,3,1]
[3,1,2]
[3,2,1]
```

Number of permutations:

```text
n!
```

---

# 19. Permutations — Java

```java
static List<List<Integer>> permute(
        int[] nums) {

    List<List<Integer>> result =
        new ArrayList<>();

    boolean[] used =
        new boolean[nums.length];

    backtrackPermutations(
        nums,
        used,
        new ArrayList<>(),
        result
    );

    return result;
}

static void backtrackPermutations(
        int[] nums,
        boolean[] used,
        List<Integer> current,
        List<List<Integer>> result) {

    if (current.size()
            == nums.length) {

        result.add(
            new ArrayList<>(current)
        );

        return;
    }

    for (int i = 0;
         i < nums.length;
         i++) {

        if (used[i]) {
            continue;
        }

        used[i] = true;
        current.add(nums[i]);

        backtrackPermutations(
            nums,
            used,
            current,
            result
        );

        current.remove(
            current.size() - 1
        );

        used[i] = false;
    }
}
```

---

# 20. Permutations with Duplicates

Sort:

```java
Arrays.sort(nums);
```

Then:

```java
if (i > 0
        && nums[i] == nums[i - 1]
        && !used[i - 1]) {

    continue;
}
```

This prevents duplicate permutations.

---

# 21. Combinations

Choose `k` elements from `n`.

Example:

```text
n = 4
k = 2
```

Results:

```text
[1,2]
[1,3]
[1,4]
[2,3]
[2,4]
[3,4]
```

Number of combinations:

```text
C(n,k)
=
n! / (k!(n-k)!)
```

---

# 22. Combinations — Java

```java
static List<List<Integer>> combine(
        int n,
        int k) {

    List<List<Integer>> result =
        new ArrayList<>();

    backtrackCombinations(
        1,
        n,
        k,
        new ArrayList<>(),
        result
    );

    return result;
}

static void backtrackCombinations(
        int start,
        int n,
        int k,
        List<Integer> current,
        List<List<Integer>> result) {

    if (current.size() == k) {

        result.add(
            new ArrayList<>(current)
        );

        return;
    }

    for (int i = start;
         i <= n;
         i++) {

        current.add(i);

        backtrackCombinations(
            i + 1,
            n,
            k,
            current,
            result
        );

        current.remove(
            current.size() - 1
        );
    }
}
```

---

# 23. Combination Sum

Given:

```text
candidates = [2,3,6,7]
target = 7
```

Possible results:

```text
[2,2,3]
[7]
```

The same candidate may be reused.

The recursive call uses:

```java
i
```

instead of:

```java
i + 1
```

to allow reuse.

---

# 24. Combination Sum — Core Pattern

```java
for (int i = start;
     i < candidates.length;
     i++) {

    if (candidates[i] > remaining) {
        break;
    }

    current.add(
        candidates[i]
    );

    backtrack(
        i,
        remaining - candidates[i],
        current,
        result
    );

    current.remove(
        current.size() - 1
    );
}
```

Sorting candidates can enable early termination.

---

# 25. Generate Parentheses

Given:

```text
n = 3
```

generate:

```text
((()))
(()())
(())()
()(())
()()()
```

Rules:

```text
open < n
close < open
```

---

# 26. Generate Parentheses — Java

```java
static List<String>
generateParenthesis(int n) {

    List<String> result =
        new ArrayList<>();

    generate(
        n,
        0,
        0,
        new StringBuilder(),
        result
    );

    return result;
}

static void generate(
        int n,
        int open,
        int close,
        StringBuilder current,
        List<String> result) {

    if (current.length()
            == 2 * n) {

        result.add(
            current.toString()
        );

        return;
    }

    if (open < n) {

        current.append('(');

        generate(
            n,
            open + 1,
            close,
            current,
            result
        );

        current.deleteCharAt(
            current.length() - 1
        );
    }

    if (close < open) {

        current.append(')');

        generate(
            n,
            open,
            close + 1,
            current,
            result
        );

        current.deleteCharAt(
            current.length() - 1
        );
    }
}
```

---

# 27. N-Queens

Place `n` queens on an `n × n` chessboard so that no two queens attack each other.

Queens cannot share:

```text
row
column
diagonal
```

A backtracking solution places:

```text
one queen per row
```

---

# 28. N-Queens Constraints

For a position:

```text
(row, col)
```

the diagonals can be represented using:

```text
row - col
```

and:

```text
row + col
```

Use sets:

```java
Set<Integer> columns =
    new HashSet<>();

Set<Integer> diagonal1 =
    new HashSet<>();

Set<Integer> diagonal2 =
    new HashSet<>();
```

---

# 29. N-Queens — Core Java

```java
static void solveNQueens(
        int row,
        int n,
        char[][] board,
        Set<Integer> columns,
        Set<Integer> diagonal1,
        Set<Integer> diagonal2,
        List<List<String>> result) {

    if (row == n) {

        List<String> solution =
            new ArrayList<>();

        for (char[] currentRow :
                board) {

            solution.add(
                new String(currentRow)
            );
        }

        result.add(solution);
        return;
    }

    for (int col = 0;
         col < n;
         col++) {

        int d1 = row - col;
        int d2 = row + col;

        if (columns.contains(col)
                || diagonal1.contains(d1)
                || diagonal2.contains(d2)) {

            continue;
        }

        board[row][col] = 'Q';

        columns.add(col);
        diagonal1.add(d1);
        diagonal2.add(d2);

        solveNQueens(
            row + 1,
            n,
            board,
            columns,
            diagonal1,
            diagonal2,
            result
        );

        board[row][col] = '.';

        columns.remove(col);
        diagonal1.remove(d1);
        diagonal2.remove(d2);
    }
}
```

---

# 30. Sudoku Solver

Backtracking can solve Sudoku.

At each empty cell:

```text
Try 1
Try 2
...
Try 9
```

If a number is valid:

```text
place it
recurse
```

If the recursive solution fails:

```text
remove it
try next number
```

This is classic:

```text
Choose → Explore → Undo
```

---

# 31. Sudoku Validity

A number is valid if it does not already exist in:

```text
row
column
3 × 3 box
```

For cell:

```text
(row, col)
```

box index:

```text
(row / 3) * 3
+
(col / 3)
```

---

# 32. Word Search

Given a grid and a word, determine whether the word can be formed by moving:

```text
up
down
left
right
```

without reusing the same cell.

Backtracking:

```text
match current character
↓
mark cell visited
↓
explore neighbors
↓
unmark cell
```

---

# 33. Word Search — Core Pattern

```java
static boolean dfs(
        char[][] board,
        String word,
        int row,
        int col,
        int index) {

    if (index == word.length()) {
        return true;
    }

    if (row < 0
            || row >= board.length
            || col < 0
            || col >= board[0].length
            || board[row][col]
                != word.charAt(index)) {

        return false;
    }

    char original =
        board[row][col];

    board[row][col] = '#';

    boolean found =
        dfs(board, word,
            row + 1, col, index + 1)
        || dfs(board, word,
            row - 1, col, index + 1)
        || dfs(board, word,
            row, col + 1, index + 1)
        || dfs(board, word,
            row, col - 1, index + 1);

    board[row][col] = original;

    return found;
}
```

---

# 34. Maze Problems

A maze can be solved using:

```text
DFS
+
Backtracking
```

At each position:

```text
Try direction
↓
mark visited
↓
recurse
↓
unmark if necessary
```

If the goal is simply reachability, visited cells generally should not be revisited.

---

# 35. Rat in a Maze

A classic problem.

From:

```text
source
```

find paths to:

```text
destination
```

using allowed cells.

Backtracking explores possible paths.

---

# 36. Partition a String

Example:

```text
"aab"
```

Possible palindrome partitions:

```text
["a","a","b"]
["aa","b"]
```

At each index:

```text
choose a substring
check palindrome
recurse on remaining suffix
undo
```

---

# 37. Palindrome Partitioning — Pattern

```java
for (int end = start;
     end < s.length();
     end++) {

    if (!isPalindrome(
            s,
            start,
            end)) {
        continue;
    }

    current.add(
        s.substring(start, end + 1)
    );

    backtrack(
        end + 1,
        ...
    );

    current.remove(
        current.size() - 1
    );
}
```

---

# 38. Letter Combinations of Phone Number

Mapping:

```text
2 → abc
3 → def
4 → ghi
...
```

For each digit:

```text
try each possible character
```

This creates a tree of choices.

If there are `n` digits, the number of combinations is up to:

```text
4^n
```

depending on digits such as `7` and `9`.

---

# 39. Letter Combinations — Core Pattern

```java
static void backtrack(
        String digits,
        int index,
        StringBuilder current,
        List<String> result,
        String[] mapping) {

    if (index == digits.length()) {

        result.add(
            current.toString()
        );

        return;
    }

    String letters =
        mapping[
            digits.charAt(index) - '0'
        ];

    for (char c :
            letters.toCharArray()) {

        current.append(c);

        backtrack(
            digits,
            index + 1,
            current,
            result,
            mapping
        );

        current.deleteCharAt(
            current.length() - 1
        );
    }
}
```

---

# 40. Combination Sum II

Difference from Combination Sum:

```text
Each candidate can be used once.
```

Sort:

```java
Arrays.sort(candidates);
```

Skip duplicate choices at the same level:

```java
if (i > start
        && candidates[i]
            == candidates[i - 1]) {

    continue;
}
```

Recursive call:

```java
i + 1
```

because the same element cannot be reused.

---

# 41. Subset Decision Tree

For every element, there are usually two choices:

```text
Take
or
Don't Take
```

Example:

```text
          []
        /    \
      [1]     []
     /  \    /  \
 [1,2] [1] [2]  []
```

This is why subsets contain:

```text
2^n
```

possibilities.

---

# 42. Permutation Decision Tree

For:

```text
[1,2,3]
```

first choose:

```text
1
```

then:

```text
2 or 3
```

then the remaining value.

Every position has a choice among unused values.

This creates:

```text
n!
```

permutations.

---

# 43. Backtracking State

A backtracking function usually needs some combination of:

```text
current path
start/index
remaining target
used[]
visited
board
sets
```

Ask:

> What information uniquely describes the current state of my search?

That becomes the recursion state.

---

# 44. Backtracking and Mutable State

Common mutable objects:

```java
List<Integer> current
StringBuilder current
char[][] board
boolean[] used
Set<Integer> columns
```

When changing shared state:

```text
make change
↓
recursive call
↓
undo change
```

This is essential.

---

# 45. Backtracking Template

```java
void backtrack(State state) {

    if (isComplete(state)) {
        save(state);
        return;
    }

    for (Choice choice :
            choices(state)) {

        if (!valid(choice, state)) {
            continue;
        }

        apply(choice, state);

        backtrack(state);

        undo(choice, state);
    }
}
```

Memorize this structure.

---

# 46. Pruning

Pruning means eliminating branches that cannot produce a valid answer.

Example:

```text
target = 10
current sum = 15
```

If all future values are non-negative:

```text
stop exploring
```

There is no reason to continue.

Pruning can dramatically reduce search time.

---

# 47. Sorting for Pruning

In combination problems, sort candidates:

```java
Arrays.sort(nums);
```

Then:

```java
if (nums[i] > remaining) {
    break;
}
```

Because all later values are even larger.

This is a common optimization.

---

# 48. Backtracking vs Brute Force

Both explore possibilities, but backtracking:

```text
stops invalid branches early
```

Brute force might generate everything and validate later.

Example:

```text
Brute force:
generate all combinations
→ check validity

Backtracking:
check validity while building
→ stop invalid branch
```

---

# 49. Backtracking vs Dynamic Programming

### Backtracking

Usually asks:

```text
Generate all valid possibilities
```

Examples:

```text
Permutations
Subsets
N-Queens
Sudoku
```

### Dynamic Programming

Usually asks:

```text
Find optimal/count result
with overlapping subproblems
```

Examples:

```text
Knapsack
Fibonacci
Longest Common Subsequence
Coin Change
```

Some problems can use either depending on the required output.

---

# 50. Recursion Tree

Before coding a recursive problem, draw a small recursion tree.

For:

```text
subsets of [1,2]
```

```text
              []
             /  \
           [1]   []
          /  \   / \
       [1,2] [1][2] []
```

This helps identify:

```text
choices
base case
state
```

---

# 51. Recursion Complexity

Always ask:

```text
How many recursive calls?
How much work per call?
What is recursion depth?
```

For permutations:

```text
O(n!)
```

For subsets:

```text
O(2^n)
```

For a binary recursion with two branches and depth n:

```text
O(2^n)
```

But exact complexity depends on the work performed at each node and output size.

---

# 52. Output-Sensitive Complexity

If a problem requires generating all:

```text
2^n subsets
```

you cannot do better than the size of the output itself.

If each subset can contain up to `n` elements, the total amount of output can be:

```text
O(n × 2^n)
```

This is important when discussing interview complexity.

---

# 53. Tail Recursion

A recursive call is tail-recursive when it is the final operation.

Example:

```java
static int sum(
        int n,
        int result) {

    if (n == 0) {
        return result;
    }

    return sum(
        n - 1,
        result + n
    );
}
```

Java does not generally perform guaranteed tail-call optimization, so deep recursion can still consume stack space.

---

# 54. Divide and Conquer

Recursion is also used for divide-and-conquer.

Pattern:

```text
Divide
↓
Solve subproblems
↓
Combine
```

Examples:

```text
Merge Sort
Quick Sort
Binary Search
Fast Power
```

This is related to recursion but is different from backtracking.

---

# 55. Backtracking vs Divide and Conquer

### Divide and Conquer

Subproblems are usually independent.

```text
Problem
 /    \
A      B
 \    /
 combine
```

### Backtracking

Choices form a search tree.

```text
Choose
 / | \
A  B  C
```

Then each choice is explored and undone.

---

# 56. Common Recursion Mistakes

### Mistake 1 — Missing base case

Leads to infinite recursion.

---

### Mistake 2 — Base case never reached

Example:

```java
solve(n);
```

instead of:

```java
solve(n - 1);
```

---

### Mistake 3 — Modifying shared state without undo

This causes one branch's choices to leak into another.

---

### Mistake 4 — Forgetting to copy results

Use:

```java
new ArrayList<>(current)
```

when storing mutable paths.

---

### Mistake 5 — Wrong recursion index

For combinations:

```text
i + 1
```

often means each element is used once.

For reusable candidates:

```text
i
```

may be required.

---

### Mistake 6 — Missing duplicate handling

For duplicate input, sort and skip duplicates when appropriate.

---

### Mistake 7 — No pruning

The algorithm may explore enormous unnecessary branches.

---

# 57. Edge Cases

Always test:

```text
n = 0
n = 1
Empty array
One element
Duplicate elements
All identical elements
Target = 0
No valid solution
Already complete state
Very deep recursion
```

---

# 58. Interview Questions — Easy

1. Factorial using recursion.
2. Sum of first N numbers.
3. Reverse a string recursively.
4. Power of a number.
5. Fibonacci.
6. Check palindrome recursively.
7. Generate binary strings.
8. Generate subsets.
9. Generate combinations.
10. Recursive binary search.

---

# 59. Interview Questions — Medium

11. Permutations.
12. Combination Sum.
13. Combination Sum II.
14. Generate Parentheses.
15. Letter Combinations of Phone Number.
16. Palindrome Partitioning.
17. Word Search.
18. Maze problems.
19. Subsets with duplicates.
20. Permutations with duplicates.
21. Rat in a Maze.
22. Flood Fill using DFS.

---

# 60. Interview Questions — Advanced

23. N-Queens.
24. Sudoku Solver.
25. Word Search II.
26. Expression Add Operators.
27. Restore IP Addresses.
28. Matchsticks to Square.
29. Partition to K Equal Sum Subsets.
30. Remove Invalid Parentheses.
31. Advanced constraint satisfaction.
32. Backtracking with pruning and memoization.

---

# 61. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Factorial | Recursion | O(n) | O(n) |
| Fast Power | Divide & Conquer | O(log n) | O(log n) |
| Naive Fibonacci | Recursion | O(2^n) | O(n) |
| Memoized Fibonacci | DP + Recursion | O(n) | O(n) |
| Subsets | Backtracking | O(n × 2^n) output-sensitive | O(n) excluding output |
| Permutations | Backtracking | O(n × n!) output-sensitive | O(n) excluding output |
| Combinations | Backtracking | O(C(n,k) × k) output-sensitive | O(k) excluding output |
| Generate Parentheses | Backtracking | Output-sensitive | O(n) recursion |
| Word Search | Backtracking | Exponential worst case | O(L) recursion |
| N-Queens | Backtracking | Exponential worst case | O(n) auxiliary sets/recursion |
| Sudoku | Backtracking | Exponential worst case | O(1) board + recursion |
| Fast Power | Divide & Conquer | O(log n) | O(log n) |

---

# 62. Quick Revision

```text
Recursion & Backtracking
│
├── Recursion
│   ├── Base Case
│   ├── Recursive Case
│   ├── Call Stack
│   └── Divide & Conquer
│
├── Basic Problems
│   ├── Factorial
│   ├── Sum
│   ├── Power
│   └── Fibonacci
│
├── Backtracking
│   ├── Choose
│   ├── Explore
│   └── Undo
│
├── Generate
│   ├── Subsets
│   ├── Permutations
│   ├── Combinations
│   └── Parentheses
│
├── Constraint Problems
│   ├── N-Queens
│   ├── Sudoku
│   └── Word Search
│
└── Optimization
    ├── Pruning
    ├── Memoization
    └── Sorting for pruning
```

---

## Interview Rule

> **For recursion, always identify the base case and make sure every recursive call moves toward it. For backtracking, memorize the core pattern: Choose → Explore → Undo. Most interview problems become much easier once you can clearly define the state, choices, base case, and pruning conditions.**
