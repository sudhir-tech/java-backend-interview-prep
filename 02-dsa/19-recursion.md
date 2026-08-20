# DSA — Recursion

Recursion is a technique where a function calls itself to solve a smaller version of the same problem.

Recursion is foundational for:

- Trees
- Graphs
- Backtracking
- Divide and conquer
- Binary search
- Merge sort
- Quick sort
- Dynamic programming
- Mathematical problems

---

# 1. What is Recursion?

A recursive function has two essential parts:

```text
1. Base case
2. Recursive case
```

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

The function keeps calling itself until:

```text
n == 0
```

---

# 2. Base Case

The base case tells recursion when to stop.

Without a valid base case:

```text
function
↓
function
↓
function
↓
...
```

Eventually:

```text
StackOverflowError
```

Example:

```java
if (n == 0) {
    return;
}
```

---

# 3. Recursive Case

The recursive case moves the problem toward the base case.

```java
printNumbers(n - 1);
```

Here:

```text
n
↓
n - 1
↓
n - 2
↓
...
↓
0
```

A recursive call should generally make measurable progress toward termination.

---

# 4. Call Stack

Every recursive call creates a stack frame.

Example:

```java
factorial(3)
```

creates:

```text
factorial(3)
    ↓
factorial(2)
    ↓
factorial(1)
    ↓
factorial(0)
```

Then calls return in reverse order:

```text
factorial(0)
↑
factorial(1)
↑
factorial(2)
↑
factorial(3)
```

---

# 5. Factorial

Mathematically:

```text
n! = n × (n - 1)!
```

Base case:

```text
0! = 1
```

---

# 6. Factorial — Java

```java
static long factorial(int n) {

    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
}
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

The space is due to the recursion stack.

---

# 7. Fibonacci

Definition:

```text
F(0) = 0
F(1) = 1

F(n) =
F(n - 1)
+
F(n - 2)
```

Recursive implementation:

```java
static int fibonacci(int n) {

    if (n <= 1) {
        return n;
    }

    return fibonacci(n - 1)
        + fibonacci(n - 2);
}
```

Naive complexity:

```text
Time: O(2^n)
Space: O(n)
```

This is inefficient because the same subproblems are repeatedly calculated.

---

# 8. Recursion Tree

For:

```text
fibonacci(5)
```

the calls look roughly like:

```text
                 fib(5)
              /         \
          fib(4)        fib(3)
          /   \         /   \
      fib(3) fib(2) fib(2) fib(1)
```

Notice that:

```text
fib(3)
fib(2)
```

appear multiple times.

This is why memoization can improve Fibonacci to:

```text
O(n)
```

---

# 9. Sum of First N Numbers

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
sum(4)
```

the calculation becomes:

```text
4 + 3 + 2 + 1 + 0
```

Result:

```text
10
```

---

# 10. Power

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

    return x * power(
        x,
        n - 1
    );
}
```

Complexity:

```text
O(n)
```

---

# 11. Fast Power

We can improve power calculation using:

```text
x^n =
(x^(n/2))²
```

for even `n`.

For odd `n`:

```text
x^n =
x × x^(n-1)
```

More efficiently:

```java
static long fastPower(
        long x,
        int n) {

    if (n == 0) {
        return 1;
    }

    long half =
        fastPower(x, n / 2);

    long result =
        half * half;

    if (n % 2 == 1) {
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

# 12. Binary Search Recursively

```java
static int binarySearch(
        int[] nums,
        int target,
        int left,
        int right) {

    if (left > right) {
        return -1;
    }

    int mid =
        left + (right - left) / 2;

    if (nums[mid] == target) {
        return mid;
    }

    if (target < nums[mid]) {

        return binarySearch(
            nums,
            target,
            left,
            mid - 1
        );
    }

    return binarySearch(
        nums,
        target,
        mid + 1,
        right
    );
}
```

Complexity:

```text
Time: O(log n)
Space: O(log n)
```

Iterative binary search uses:

```text
O(1)
```

auxiliary space.

---

# 13. Reverse a String Recursively

```java
static String reverse(
        String s) {

    if (s.length() <= 1) {
        return s;
    }

    return reverse(
        s.substring(1)
    ) + s.charAt(0);
}
```

This is easy to understand but can create many intermediate strings.

For production code, prefer:

```text
StringBuilder
```

or iterative two-pointer logic when appropriate.

---

# 14. Reverse an Array Recursively

```java
static void reverse(
        int[] nums,
        int left,
        int right) {

    if (left >= right) {
        return;
    }

    int temp =
        nums[left];

    nums[left] =
        nums[right];

    nums[right] =
        temp;

    reverse(
        nums,
        left + 1,
        right - 1
    );
}
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

because of recursion depth.

---

# 15. Palindrome Check

```java
static boolean isPalindrome(
        String s,
        int left,
        int right) {

    if (left >= right) {
        return true;
    }

    if (s.charAt(left)
            != s.charAt(right)) {

        return false;
    }

    return isPalindrome(
        s,
        left + 1,
        right - 1
    );
}
```

---

# 16. Greatest Common Divisor

Euclid's algorithm:

```text
gcd(a,b)
=
gcd(b, a % b)
```

Base case:

```text
b == 0
```

---

# 17. GCD — Java

```java
static int gcd(
        int a,
        int b) {

    if (b == 0) {
        return Math.abs(a);
    }

    return gcd(
        b,
        a % b
    );
}
```

Complexity:

```text
O(log(min(a,b)))
```

---

# 18. Recursive Tree Traversal

Recursion is naturally suited for trees.

Three classic traversals:

```text
Preorder
Inorder
Postorder
```

---

# 19. Preorder

Order:

```text
Root
Left
Right
```

```java
static void preorder(
        TreeNode root) {

    if (root == null) {
        return;
    }

    System.out.println(
        root.val
    );

    preorder(root.left);

    preorder(root.right);
}
```

---

# 20. Inorder

Order:

```text
Left
Root
Right
```

```java
static void inorder(
        TreeNode root) {

    if (root == null) {
        return;
    }

    inorder(root.left);

    System.out.println(
        root.val
    );

    inorder(root.right);
}
```

For a Binary Search Tree, inorder traversal gives:

```text
sorted order
```

---

# 21. Postorder

Order:

```text
Left
Right
Root
```

```java
static void postorder(
        TreeNode root) {

    if (root == null) {
        return;
    }

    postorder(root.left);

    postorder(root.right);

    System.out.println(
        root.val
    );
}
```

---

# 22. Tree Height

```java
static int height(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    return 1 + Math.max(
        height(root.left),
        height(root.right)
    );
}
```

For a balanced tree:

```text
O(log n)
```

recursion depth.

For a skewed tree:

```text
O(n)
```

recursion depth.

---

# 23. Count Nodes in a Tree

```java
static int countNodes(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    return 1
        + countNodes(root.left)
        + countNodes(root.right);
}
```

Complexity:

```text
Time: O(n)
Space: O(h)
```

where `h` is tree height.

---

# 24. Sum of Tree Nodes

```java
static int sumNodes(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    return root.val
        + sumNodes(root.left)
        + sumNodes(root.right);
}
```

---

# 25. Maximum Depth of Binary Tree

```java
static int maxDepth(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    return 1 + Math.max(
        maxDepth(root.left),
        maxDepth(root.right)
    );
}
```

This is one of the most common recursive interview questions.

---

# 26. Validate Binary Search Tree

For a BST:

```text
left subtree < node
right subtree > node
```

A recursive solution can carry:

```text
lower bound
upper bound
```

---

# 27. Validate BST — Java

```java
static boolean isValidBST(
        TreeNode root) {

    return validate(
        root,
        Long.MIN_VALUE,
        Long.MAX_VALUE
    );
}

static boolean validate(
        TreeNode node,
        long lower,
        long upper) {

    if (node == null) {
        return true;
    }

    if (node.val <= lower
            || node.val >= upper) {

        return false;
    }

    return validate(
        node.left,
        lower,
        node.val
    )
    && validate(
        node.right,
        node.val,
        upper
    );
}
```

---

# 28. Divide and Conquer

Divide and conquer follows:

```text
Divide
↓
Solve smaller problems
↓
Combine
```

Classic examples:

```text
Merge Sort
Quick Sort
Binary Search
Fast Power
```

Recursion is commonly used to implement divide-and-conquer algorithms.

---

# 29. Merge Sort

Steps:

```text
Divide array into two halves
↓
Sort left half
↓
Sort right half
↓
Merge sorted halves
```

Recurrence:

```text
T(n) =
2T(n/2)
+
O(n)
```

Therefore:

```text
O(n log n)
```

---

# 30. Merge Sort — Java

```java
static void mergeSort(
        int[] nums,
        int left,
        int right) {

    if (left >= right) {
        return;
    }

    int mid =
        left + (right - left) / 2;

    mergeSort(
        nums,
        left,
        mid
    );

    mergeSort(
        nums,
        mid + 1,
        right
    );

    merge(
        nums,
        left,
        mid,
        right
    );
}
```

---

# 31. Quick Sort

Quick Sort:

```text
Choose pivot
↓
Partition
↓
Recursively sort left
↓
Recursively sort right
```

Average:

```text
O(n log n)
```

Worst case:

```text
O(n²)
```

depending on pivot selection and input.

---

# 32. Recursion Tree

For divide-and-conquer problems, visualize:

```text
                  n
             /         \
          n/2           n/2
         /   \         /   \
       n/4   n/4     n/4   n/4
```

This helps reason about complexity.

---

# 33. Tail Recursion

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

Some languages optimize tail recursion, but Java does not generally perform tail-call optimization.

Therefore deep tail recursion can still cause:

```text
StackOverflowError
```

---

# 34. Recursion Stack Space

If maximum recursion depth is:

```text
h
```

then recursion stack space is typically:

```text
O(h)
```

Examples:

```text
Balanced binary tree → O(log n)
Skewed binary tree → O(n)
```

---

# 35. Recursion vs Iteration

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
Stack memory
Potential StackOverflowError
Function-call overhead
```

### Iteration

Advantages:

```text
O(1) stack in many cases
Often faster
No recursion depth issue
```

Disadvantages:

```text
Can be more verbose
Tree/backtracking logic may be less natural
```

---

# 36. When to Use Recursion

Good candidates:

```text
Tree traversal
Graph DFS
Backtracking
Divide and conquer
Recursive mathematical definitions
Nested structures
```

---

# 37. When to Prefer Iteration

Prefer iteration when:

```text
recursion depth can be very large
```

or:

```text
the iterative solution is significantly simpler
```

Examples:

```text
Simple array traversal
Large linked list traversal
Production code with unknown depth
```

---

# 38. Recursion + Memoization

If recursive calls repeat the same state:

```text
cache the answer
```

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

This converts:

```text
exponential recursion
```

into:

```text
O(n) DP
```

---

# 39. Recursion + Backtracking

Backtracking is essentially recursion plus:

```text
choose
explore
undo
```

Example:

```java
path.add(choice);

backtrack(...);

path.remove(
    path.size() - 1
);
```

Common problems:

```text
Subsets
Permutations
N-Queens
Sudoku
Combination Sum
Word Search
```

---

# 40. Recursion + Binary Trees

Tree recursion often follows:

```text
solve left
solve right
combine results
```

Example:

```java
int left =
    solve(root.left);

int right =
    solve(root.right);

return combine(
    left,
    right
);
```

This pattern appears repeatedly in tree interviews.

---

# 41. Recursion + Linked Lists

Reverse a linked list recursively:

```java
static ListNode reverse(
        ListNode head) {

    if (head == null
            || head.next == null) {

        return head;
    }

    ListNode newHead =
        reverse(head.next);

    head.next.next = head;
    head.next = null;

    return newHead;
}
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

because of recursion depth.

---

# 42. Reverse Linked List — Iterative Alternative

The iterative version uses:

```text
prev
current
next
```

and has:

```text
O(1)
```

auxiliary space.

This is a good interview comparison:

```text
recursive → elegant
iterative → memory efficient
```

---

# 43. Recursive Digit Sum

```java
static int digitSum(
        int n) {

    n = Math.abs(n);

    if (n < 10) {
        return n;
    }

    return n % 10
        + digitSum(n / 10);
}
```

Example:

```text
12345

5
+
4
+
3
+
2
+
1

= 15
```

---

# 44. Generate Binary Strings

Generate all binary strings of length `n`.

At each position:

```text
choose 0
or
choose 1
```

Number of strings:

```text
2^n
```

This is a simple example of a recursion decision tree.

---

# 45. Tower of Hanoi

Classic recursion problem.

Rules:

```text
Move one disk at a time.
Never place a larger disk on a smaller disk.
```

For `n` disks:

```text
move n - 1 disks
move largest disk
move n - 1 disks
```

Minimum moves:

```text
2^n - 1
```

---

# 46. Tower of Hanoi — Java

```java
static void towerOfHanoi(
        int n,
        char source,
        char auxiliary,
        char destination) {

    if (n == 0) {
        return;
    }

    towerOfHanoi(
        n - 1,
        source,
        destination,
        auxiliary
    );

    System.out.println(
        "Move disk " + n
        + " from " + source
        + " to " + destination
    );

    towerOfHanoi(
        n - 1,
        auxiliary,
        source,
        destination
    );
}
```

Complexity:

```text
Time: O(2^n)
Space: O(n)
```

---

# 47. Recursion and StackOverflowError

Java's call stack has a finite size.

Very deep recursion can produce:

```text
java.lang.StackOverflowError
```

Example risk:

```java
void recurse() {
    recurse();
}
```

There is no terminating condition.

Even with a base case, very large input can exceed the stack.

---

# 48. How to Debug Recursion

When debugging, write down:

```text
Current parameters
↓
Base case
↓
Recursive call
↓
What happens after recursive call?
```

Example:

```text
factorial(4)
→ 4 * factorial(3)
→ 4 * 3 * factorial(2)
→ 4 * 3 * 2 * factorial(1)
→ 24
```

Tracing calls manually is one of the best ways to understand recursion.

---

# 49. Common Recursion Mistakes

### Mistake 1 — Missing base case

Leads to infinite recursion.

### Mistake 2 — Base case never reached

The recursive arguments do not move toward termination.

### Mistake 3 — Wrong return value

For example:

```java
recursiveCall(...);
```

when the result should have been:

```java
return recursiveCall(...);
```

### Mistake 4 — Modifying shared state incorrectly

Common in backtracking.

### Mistake 5 — Forgetting to undo

Use:

```text
choose
recurse
undo
```

### Mistake 6 — Ignoring stack depth

A theoretically correct recursive solution can still fail on very large input.

---

# 50. Recursion Complexity Checklist

Ask:

```text
1. How many recursive calls are made?
2. What happens to the input size?
3. What is the recursion depth?
4. Is there repeated work?
5. Is there memoization?
6. Is there additional data stored per call?
```

For:

```text
T(n) = T(n - 1) + O(1)
```

usually:

```text
O(n)
```

For:

```text
T(n) = 2T(n - 1) + O(1)
```

usually:

```text
O(2^n)
```

For:

```text
T(n) = 2T(n/2) + O(n)
```

usually:

```text
O(n log n)
```

---

# 51. Master Theorem

For divide-and-conquer recurrences:

```text
T(n) =
aT(n/b)
+
f(n)
```

The Master Theorem can help determine complexity.

Classic example:

```text
Merge Sort:

T(n) =
2T(n/2)
+
O(n)

= O(n log n)
```

You should understand the intuition even if you do not memorize every theorem case.

---

# 52. Recursion Problem-Solving Framework

When you see a recursive problem:

```text
1. Define the smallest valid input.
2. Write the base case.
3. Assume the smaller problem is already solved.
4. Determine how to build the current answer.
5. Make the recursive call.
6. Verify that the input moves toward the base case.
```

This is often called:

```text
recursive leap of faith
```

You assume the recursive call correctly solves the smaller problem, then focus on combining it with the current step.

---

# 53. Interview Questions — Easy

1. Factorial.
2. Fibonacci.
3. Sum of numbers.
4. Reverse a string.
5. Palindrome check.
6. GCD.
7. Power calculation.
8. Binary search recursively.
9. Tree traversals.
10. Maximum depth of binary tree.

---

# 54. Interview Questions — Medium

11. Reverse linked list recursively.
12. Merge sort.
13. Quick sort.
14. Generate binary strings.
15. Generate subsets.
16. Generate permutations.
17. Combination Sum.
18. Word Search.
19. Tree path problems.
20. Validate BST.
21. Lowest Common Ancestor.
22. Recursive tree diameter.

---

# 55. Interview Questions — Advanced

23. N-Queens.
24. Sudoku Solver.
25. Tower of Hanoi.
26. Expression Add Operators.
27. Advanced tree DP.
28. Divide-and-conquer optimization.
29. Recursion + memoization.
30. Recursion + bitmask.
31. Advanced backtracking.
32. Recursive parsing problems.

---

# 56. Complexity Summary

| Problem | Time | Auxiliary Space |
|---|---:|---:|
| Factorial | O(n) | O(n) |
| Fibonacci naive | O(2^n) | O(n) |
| Fibonacci memoized | O(n) | O(n) |
| Fast Power | O(log n) | O(log n) |
| Binary Search | O(log n) | O(log n) |
| GCD | O(log n) | O(log n) |
| Tree Traversal | O(n) | O(h) |
| Tree Height | O(n) | O(h) |
| Merge Sort | O(n log n) | O(n + log n) |
| Quick Sort average | O(n log n) | O(log n) average |
| Tower of Hanoi | O(2^n) | O(n) |

`h` = tree height.

---

# 57. Quick Revision

```text
Recursion
│
├── Fundamentals
│   ├── Base Case
│   ├── Recursive Case
│   └── Call Stack
│
├── Mathematics
│   ├── Factorial
│   ├── Fibonacci
│   ├── Power
│   └── GCD
│
├── Arrays / Strings
│   ├── Reverse
│   ├── Palindrome
│   └── Binary Search
│
├── Trees
│   ├── Preorder
│   ├── Inorder
│   ├── Postorder
│   ├── Height
│   └── Tree Problems
│
├── Divide & Conquer
│   ├── Merge Sort
│   ├── Quick Sort
│   └── Binary Search
│
├── Backtracking
│   ├── Subsets
│   ├── Permutations
│   ├── N-Queens
│   └── Sudoku
│
└── Optimization
    └── Memoization / DP
```

---

## Interview Rule

> **Before writing recursive code, identify the base case and make sure every recursive call moves toward it. Then ask whether the same state is being solved repeatedly — if it is, memoization or dynamic programming may be the right next step.**

---

## Recursion Formula to Remember

```text
Recursion =
Base Case
+
Smaller Problem
+
Current Work
```

For backtracking:

```text
Backtracking =
Choose
+
Explore
+
Undo
+
Prune
```
