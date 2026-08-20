# DSA — Backtracking

Backtracking is a systematic way of exploring all possible choices while abandoning a path as soon as it becomes invalid or cannot produce a useful solution.

It is especially important for interview problems involving:

- Permutations
- Combinations
- Subsets
- Decision trees
- Constraint satisfaction
- Grid exploration
- Word search
- Sudoku
- N-Queens
- Partitioning
- Generate-and-test problems

---

# 1. What is Backtracking?

The basic idea is:

```text
Choose
↓
Explore
↓
Undo the choice
↓
Try another choice
```

Example:

```text
        start
       /     \
      A       B
     / \     / \
   AB  AC   BA  BC
```

We explore one possibility, then return and try another.

---

# 2. Backtracking vs Brute Force

Both can explore many possibilities.

The important difference is:

```text
Brute Force:
generate everything and check later
```

Backtracking:

```text
build the solution incrementally
↓
stop immediately when the partial solution is invalid
```

This is called:

```text
pruning
```

---

# 3. General Backtracking Template

```java
static void backtrack(
        List<Integer> path) {

    if (isComplete(path)) {
        result.add(
            new ArrayList<>(path)
        );
        return;
    }

    for (int choice : choices) {

        if (!isValid(choice, path)) {
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

The most important line is:

```java
path.remove(path.size() - 1);
```

This is the:

```text
undo
```

step.

---

# 4. Decision Tree

Backtracking problems can usually be visualized as a decision tree.

For:

```text
[1,2,3]
```

subsets:

```text
                 []
             /         \
           [1]          []
          /   \        /   \
      [1,2] [1]      [2]    []
       / \             ...
```

Each level represents a decision.

---

# 5. State in Backtracking

A backtracking state commonly contains:

```text
current path
current index
used elements
remaining capacity
current board
```

The state should contain everything required to continue the search.

---

# 6. Subsets

Given:

```text
[1,2,3]
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

# 7. Subsets — Java

```java
static List<List<Integer>> subsets(
        int[] nums) {

    List<List<Integer>> result =
        new ArrayList<>();

    backtrack(
        nums,
        0,
        new ArrayList<>(),
        result
    );

    return result;
}

static void backtrack(
        int[] nums,
        int start,
        List<Integer> path,
        List<List<Integer>> result) {

    result.add(
        new ArrayList<>(path)
    );

    for (int i = start;
         i < nums.length;
         i++) {

        path.add(nums[i]);

        backtrack(
            nums,
            i + 1,
            path,
            result
        );

        path.remove(
            path.size() - 1
        );
    }
}
```

Complexity:

```text
Time: O(n × 2^n)
Space: O(n) recursion/path
```

excluding the output.

---

# 8. Subsets — Include/Exclude Pattern

Another way to think about subsets:

```text
For every element:

Include it
or
Exclude it
```

Example:

```text
        1
       / \
    take  skip
     /      \
    2        2
   / \      / \
 take skip take skip
```

This creates:

```text
2^n
```

possibilities.

---

# 9. Subsets With Duplicates

If:

```text
nums = [1,2,2]
```

we must avoid duplicate subsets.

First sort:

```java
Arrays.sort(nums);
```

Then skip duplicate choices at the same recursion level:

```java
if (i > start
        && nums[i] == nums[i - 1]) {
    continue;
}
```

---

# 10. Subsets With Duplicates — Java

```java
static List<List<Integer>>
subsetsWithDup(int[] nums) {

    Arrays.sort(nums);

    List<List<Integer>> result =
        new ArrayList<>();

    backtrackDup(
        nums,
        0,
        new ArrayList<>(),
        result
    );

    return result;
}

static void backtrackDup(
        int[] nums,
        int start,
        List<Integer> path,
        List<List<Integer>> result) {

    result.add(
        new ArrayList<>(path)
    );

    for (int i = start;
         i < nums.length;
         i++) {

        if (i > start
                && nums[i] == nums[i - 1]) {
            continue;
        }

        path.add(nums[i]);

        backtrackDup(
            nums,
            i + 1,
            path,
            result
        );

        path.remove(
            path.size() - 1
        );
    }
}
```

---

# 11. Permutations

Given:

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

# 12. Permutations — Used Array

```java
static List<List<Integer>> permute(
        int[] nums) {

    List<List<Integer>> result =
        new ArrayList<>();

    boolean[] used =
        new boolean[nums.length];

    backtrackPermute(
        nums,
        used,
        new ArrayList<>(),
        result
    );

    return result;
}

static void backtrackPermute(
        int[] nums,
        boolean[] used,
        List<Integer> path,
        List<List<Integer>> result) {

    if (path.size()
            == nums.length) {

        result.add(
            new ArrayList<>(path)
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
        path.add(nums[i]);

        backtrackPermute(
            nums,
            used,
            path,
            result
        );

        path.remove(
            path.size() - 1
        );

        used[i] = false;
    }
}
```

---

# 13. Permutations With Duplicates

For:

```text
[1,1,2]
```

sorting plus a duplicate check prevents duplicate permutations.

Important condition:

```java
if (i > 0
        && nums[i] == nums[i - 1]
        && !used[i - 1]) {
    continue;
}
```

The `!used[i - 1]` part means:

```text
skip duplicate choices
at the same recursion level
```

but still allows equal values in different positions.

---

# 14. Combinations

Choose exactly:

```text
k
```

numbers from:

```text
1...n
```

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
```

---

# 15. Combinations — Java

```java
static List<List<Integer>>
combine(int n, int k) {

    List<List<Integer>> result =
        new ArrayList<>();

    combineBacktrack(
        1,
        n,
        k,
        new ArrayList<>(),
        result
    );

    return result;
}

static void combineBacktrack(
        int start,
        int n,
        int k,
        List<Integer> path,
        List<List<Integer>> result) {

    if (path.size() == k) {

        result.add(
            new ArrayList<>(path)
        );

        return;
    }

    for (int i = start;
         i <= n;
         i++) {

        path.add(i);

        combineBacktrack(
            i + 1,
            n,
            k,
            path,
            result
        );

        path.remove(
            path.size() - 1
        );
    }
}
```

---

# 16. Combination Sum

Given candidates and a target, find combinations whose sum equals the target.

Example:

```text
candidates = [2,3,6,7]
target = 7
```

Result:

```text
[2,2,3]
[7]
```

Elements can be reused.

---

# 17. Combination Sum — Java

```java
static List<List<Integer>>
combinationSum(
        int[] candidates,
        int target) {

    List<List<Integer>> result =
        new ArrayList<>();

    backtrackCombination(
        candidates,
        target,
        0,
        new ArrayList<>(),
        result
    );

    return result;
}

static void backtrackCombination(
        int[] candidates,
        int remaining,
        int start,
        List<Integer> path,
        List<List<Integer>> result) {

    if (remaining == 0) {

        result.add(
            new ArrayList<>(path)
        );

        return;
    }

    if (remaining < 0) {
        return;
    }

    for (int i = start;
         i < candidates.length;
         i++) {

        if (candidates[i]
                > remaining) {
            continue;
        }

        path.add(candidates[i]);

        backtrackCombination(
            candidates,
            remaining - candidates[i],
            i,
            path,
            result
        );

        path.remove(
            path.size() - 1
        );
    }
}
```

---

# 18. Combination Sum II

Difference:

```text
Each candidate can be used once.
```

Sort the input.

Skip duplicates:

```java
if (i > start
        && candidates[i]
            == candidates[i - 1]) {
    continue;
}
```

Then recurse with:

```java
i + 1
```

instead of:

```java
i
```

---

# 19. Combination Sum — Key Difference

### Combination Sum

```java
backtrack(..., i, ...)
```

Can reuse the same element.

### Combination Sum II

```java
backtrack(..., i + 1, ...)
```

Each element is used once.

This small index difference is extremely important.

---

# 20. Letter Combinations of a Phone Number

Mapping:

```text
2 → abc
3 → def
4 → ghi
...
```

For:

```text
23
```

generate:

```text
ad
ae
af
bd
be
bf
cd
ce
cf
```

At each position:

```text
choose one character
```

and recurse to the next digit.

---

# 21. Phone Combinations — Java

```java
static List<String>
letterCombinations(String digits) {

    List<String> result =
        new ArrayList<>();

    if (digits.isEmpty()) {
        return result;
    }

    String[] mapping = {
        "",
        "",
        "abc",
        "def",
        "ghi",
        "jkl",
        "mno",
        "pqrs",
        "tuv",
        "wxyz"
    };

    backtrackPhone(
        digits,
        0,
        new StringBuilder(),
        mapping,
        result
    );

    return result;
}

static void backtrackPhone(
        String digits,
        int index,
        StringBuilder path,
        String[] mapping,
        List<String> result) {

    if (index == digits.length()) {

        result.add(
            path.toString()
        );

        return;
    }

    String letters =
        mapping[
            digits.charAt(index) - '0'
        ];

    for (char letter :
            letters.toCharArray()) {

        path.append(letter);

        backtrackPhone(
            digits,
            index + 1,
            path,
            mapping,
            result
        );

        path.deleteCharAt(
            path.length() - 1
        );
    }
}
```

---

# 22. Generate Parentheses

Given:

```text
n = 3
```

generate valid combinations:

```text
((()))
(()())
(())()
()(())
()()()
```

There are:

```text
Catalan(n)
```

valid combinations.

---

# 23. Generate Parentheses — Key Rule

At any point:

```text
open < n
```

means we can add:

```text
(
```

And:

```text
close < open
```

means we can add:

```text
)
```

This prevents invalid strings from ever being generated.

That is an example of:

```text
pruning
```

---

# 24. Generate Parentheses — Java

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
        StringBuilder path,
        List<String> result) {

    if (path.length()
            == 2 * n) {

        result.add(
            path.toString()
        );

        return;
    }

    if (open < n) {

        path.append('(');

        generate(
            n,
            open + 1,
            close,
            path,
            result
        );

        path.deleteCharAt(
            path.length() - 1
        );
    }

    if (close < open) {

        path.append(')');

        generate(
            n,
            open,
            close + 1,
            path,
            result
        );

        path.deleteCharAt(
            path.length() - 1
        );
    }
}
```

---

# 25. Word Search

Given a grid of characters, determine whether a word exists by moving:

```text
up
down
left
right
```

A cell cannot be reused in the same path.

Use:

```text
DFS
+
backtracking
```

---

# 26. Word Search — Java

```java
static boolean exist(
        char[][] board,
        String word) {

    for (int row = 0;
         row < board.length;
         row++) {

        for (int col = 0;
             col < board[0].length;
             col++) {

            if (dfsWord(
                    board,
                    word,
                    row,
                    col,
                    0)) {

                return true;
            }
        }
    }

    return false;
}

static boolean dfsWord(
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
        dfsWord(
            board,
            word,
            row + 1,
            col,
            index + 1
        )
        || dfsWord(
            board,
            word,
            row - 1,
            col,
            index + 1
        )
        || dfsWord(
            board,
            word,
            row,
            col + 1,
            index + 1
        )
        || dfsWord(
            board,
            word,
            row,
            col - 1,
            index + 1
        );

    board[row][col] = original;

    return found;
}
```

The restoration:

```java
board[row][col] = original;
```

is the backtracking step.

---

# 27. N-Queens

Place:

```text
N queens
```

on an:

```text
N × N
```

chessboard so that no two queens attack each other.

Queens cannot share:

```text
row
column
diagonal
```

---

# 28. N-Queens Strategy

Place one queen per row.

For each row:

```text
Try every column.
```

Check:

```text
column available?
diagonal available?
```

If valid:

```text
place queen
↓
recurse
↓
remove queen
```

---

# 29. N-Queens — Java

```java
static List<List<String>>
solveNQueens(int n) {

    List<List<String>> result =
        new ArrayList<>();

    char[][] board =
        new char[n][n];

    for (char[] row : board) {
        Arrays.fill(row, '.');
    }

    boolean[] columns =
        new boolean[n];

    boolean[] diagonals =
        new boolean[2 * n - 1];

    boolean[] antiDiagonals =
        new boolean[2 * n - 1];

    solveQueens(
        0,
        board,
        columns,
        diagonals,
        antiDiagonals,
        result
    );

    return result;
}

static void solveQueens(
        int row,
        char[][] board,
        boolean[] columns,
        boolean[] diagonals,
        boolean[] antiDiagonals,
        List<List<String>> result) {

    int n = board.length;

    if (row == n) {

        List<String> solution =
            new ArrayList<>();

        for (char[] current :
                board) {

            solution.add(
                new String(current)
            );
        }

        result.add(solution);
        return;
    }

    for (int col = 0;
         col < n;
         col++) {

        int diagonal =
            row - col + n - 1;

        int antiDiagonal =
            row + col;

        if (columns[col]
                || diagonals[diagonal]
                || antiDiagonals[antiDiagonal]) {
            continue;
        }

        board[row][col] = 'Q';

        columns[col] = true;
        diagonals[diagonal] = true;
        antiDiagonals[antiDiagonal] = true;

        solveQueens(
            row + 1,
            board,
            columns,
            diagonals,
            antiDiagonals,
            result
        );

        board[row][col] = '.';

        columns[col] = false;
        diagonals[diagonal] = false;
        antiDiagonals[antiDiagonal] = false;
    }
}
```

---

# 30. Sudoku Solver

Sudoku is a classic constraint-satisfaction problem.

For each empty cell:

```text
Try 1
Try 2
...
Try 9
```

If valid:

```text
place number
↓
recurse
↓
undo if necessary
```

---

# 31. Sudoku Constraints

A number cannot already exist in:

```text
same row
same column
same 3 × 3 box
```

---

# 32. Sudoku Solver — Java

```java
static boolean solveSudoku(
        char[][] board) {

    for (int row = 0;
         row < 9;
         row++) {

        for (int col = 0;
             col < 9;
             col++) {

            if (board[row][col] != '.') {
                continue;
            }

            for (char digit = '1';
                 digit <= '9';
                 digit++) {

                if (!isValid(
                        board,
                        row,
                        col,
                        digit)) {
                    continue;
                }

                board[row][col] =
                    digit;

                if (solveSudoku(board)) {
                    return true;
                }

                board[row][col] =
                    '.';
            }

            return false;
        }
    }

    return true;
}

static boolean isValid(
        char[][] board,
        int row,
        int col,
        char digit) {

    for (int i = 0; i < 9; i++) {

        if (board[row][i] == digit) {
            return false;
        }

        if (board[i][col] == digit) {
            return false;
        }

        int boxRow =
            3 * (row / 3) + i / 3;

        int boxCol =
            3 * (col / 3) + i % 3;

        if (board[boxRow][boxCol]
                == digit) {

            return false;
        }
    }

    return true;
}
```

---

# 33. Combination vs Permutation

This is a very common interview confusion.

### Combination

Order does not matter.

```text
[1,2]
```

and:

```text
[2,1]
```

are the same.

Use:

```text
start index
```

to avoid revisiting earlier elements.

### Permutation

Order matters.

```text
[1,2]
```

and:

```text
[2,1]
```

are different.

Usually use:

```text
used[]
```

---

# 34. Subset vs Subsequence

### Subset

Usually refers to selecting elements without caring about original order.

### Subsequence

Must preserve original relative order.

Example:

```text
[1,2,3]
```

`[1,3]` is a subsequence.

Backtracking can generate subsequences using:

```text
include
or
exclude
```

---

# 35. Pruning

Pruning means stopping a branch early.

Example:

```java
if (remaining < 0) {
    return;
}
```

There is no reason to continue because the target cannot be reached.

Other pruning conditions:

```text
duplicate choice
invalid constraint
exceeded capacity
impossible remaining sum
already found better solution
```

---

# 36. Sorting for Pruning

Sorting can make pruning more powerful.

Example:

```java
Arrays.sort(candidates);
```

Then:

```java
if (candidates[i] > remaining) {
    break;
}
```

If the current candidate is too large, all later candidates are also too large.

This can significantly reduce the search tree.

---

# 37. Backtracking Complexity

Backtracking often has exponential complexity.

Examples:

```text
Subsets → O(2^n)
Permutations → O(n!)
```

But actual complexity depends on:

```text
number of states
branching factor
depth
pruning
output size
```

---

# 38. Backtracking Space

Typical auxiliary space:

```text
O(depth)
```

for the recursion stack and current path.

But if storing all results:

```text
output space
```

can be exponential or factorial.

Always distinguish:

```text
auxiliary space
```

from:

```text
output space
```

---

# 39. Backtracking vs Dynamic Programming

Use backtracking when:

```text
you need to enumerate solutions
```

or:

```text
find one valid configuration
```

Use DP when:

```text
many different paths reach the same state
```

and you only need the optimal/count/boolean result rather than every solution.

---

# 40. Backtracking vs Graph DFS

They look similar but have different goals.

### Graph DFS

Usually:

```text
visit each node once
```

using:

```text
visited[]
```

### Backtracking

May intentionally revisit the same element/state in different decision paths.

The goal is:

```text
explore combinations of choices
```

---

# 41. Backtracking with Bitmask

For small sets, a bitmask can represent used elements.

Example:

```text
0000
```

nothing used.

```text
0101
```

elements 0 and 2 are used.

Check:

```java
(mask & (1 << i)) != 0
```

Set:

```java
mask | (1 << i)
```

Unset:

```java
mask & ~(1 << i)
```

This is useful for advanced permutation and subset problems.

---

# 42. Backtracking with StringBuilder

For string generation, prefer:

```java
StringBuilder
```

instead of repeatedly creating strings.

Add:

```java
path.append(ch);
```

Undo:

```java
path.deleteCharAt(
    path.length() - 1
);
```

This reduces unnecessary object creation.

---

# 43. Backtracking Template — Index Based

```java
void backtrack(
        int start,
        List<Integer> path) {

    result.add(
        new ArrayList<>(path)
    );

    for (int i = start;
         i < nums.length;
         i++) {

        path.add(nums[i]);

        backtrack(
            i + 1,
            path
        );

        path.remove(
            path.size() - 1
        );
    }
}
```

Use this for:

```text
Subsets
Combinations
Combination Sum variants
```

---

# 44. Backtracking Template — Used Array

```java
void backtrack(
        List<Integer> path,
        boolean[] used) {

    if (path.size()
            == nums.length) {

        result.add(
            new ArrayList<>(path)
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
        path.add(nums[i]);

        backtrack(
            path,
            used
        );

        path.remove(
            path.size() - 1
        );

        used[i] = false;
    }
}
```

Use this for:

```text
Permutations
```

---

# 45. Backtracking Template — Constraint

```java
boolean solve(State state) {

    if (isComplete(state)) {
        return true;
    }

    for (Choice choice :
            choices(state)) {

        if (!isValid(
                state,
                choice)) {
            continue;
        }

        apply(
            state,
            choice
        );

        if (solve(state)) {
            return true;
        }

        undo(
            state,
            choice
        );
    }

    return false;
}
```

Use this for:

```text
Sudoku
N-Queens
Word Search
Maze problems
Constraint problems
```

---

# 46. Common Backtracking Mistakes

### Mistake 1 — Forgetting to undo

Always reverse the choice:

```java
path.add(choice);

backtrack(...);

path.remove(
    path.size() - 1
);
```

---

### Mistake 2 — Adding the same list reference

Wrong:

```java
result.add(path);
```

Usually use:

```java
result.add(
    new ArrayList<>(path)
);
```

---

### Mistake 3 — Duplicate solutions

For duplicate inputs:

```text
sort
+
skip duplicates at the same level
```

---

### Mistake 4 — Wrong recursion index

Combination reuse:

```text
i
```

Use once:

```text
i + 1
```

---

### Mistake 5 — Missing pruning

Ask:

```text
Can I prove this branch cannot produce a solution?
```

If yes:

```text
return
```

or:

```text
continue
```

---

# 47. Edge Cases

Always test:

```text
Empty input
One element
All elements identical
Duplicate values
k = 0
k = n
Target = 0
Impossible target
Very small board
Already solved board
No valid solution
Multiple valid solutions
```

---

# 48. Interview Questions — Easy

1. Subsets.
2. Combinations.
3. Letter Combinations of a Phone Number.
4. Generate Parentheses.
5. Binary Watch.
6. Find All Numbers Disappeared-style search problems.
7. Simple maze traversal.

---

# 49. Interview Questions — Medium

10. Permutations.
11. Subsets II.
12. Permutations II.
13. Combination Sum.
14. Combination Sum II.
15. Combination Sum III.
16. Word Search.
17. Palindrome Partitioning.
18. Restore IP Addresses.
19. Letter Tile Possibilities.
20. Matchsticks to Square.

---

# 50. Interview Questions — Advanced

21. N-Queens.
22. Sudoku Solver.
23. Word Search II.
24. Expression Add Operators.
25. Remove Invalid Parentheses.
26. Partition to K Equal Sum Subsets.
27. Unique Paths III.
28. Cryptarithmetic.
29. Advanced constraint satisfaction.
30. Backtracking + bitmask optimization.

---

# 51. Complexity Summary

| Problem | Typical Complexity |
|---|---:|
| Subsets | O(n × 2^n) |
| Permutations | O(n × n!) |
| Combinations | O(k × C(n,k)) |
| Generate Parentheses | O(n × Catalan(n)) |
| Combination Sum | Exponential |
| Word Search | O(rows × cols × 4^L) worst case |
| N-Queens | O(n!) upper-bound style analysis |
| Sudoku | Exponential worst case |
| Palindrome Partitioning | Exponential |
| Subsets with duplicates | Depends on unique states |

`L` in Word Search represents the word length.

---

# 52. Quick Revision

```text
Backtracking
│
├── Core Pattern
│   ├── Choose
│   ├── Explore
│   ├── Undo
│   └── Prune
│
├── Selection
│   ├── Subsets
│   ├── Combinations
│   └── Combination Sum
│
├── Ordering
│   ├── Permutations
│   └── Permutations II
│
├── Strings
│   ├── Phone Combinations
│   ├── Generate Parentheses
│   └── Palindrome Partitioning
│
├── Grid
│   ├── Word Search
│   └── Unique Paths III
│
└── Constraint Problems
    ├── N-Queens
    ├── Sudoku
    └── Matchsticks to Square
```

---

## Interview Rule

> **Backtracking = choose → explore → undo. The biggest interview skill is knowing when to prune. Before exploring a branch, ask whether the current partial solution can still possibly become valid.**
