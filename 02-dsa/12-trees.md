# DSA — Trees

A **Tree** is a hierarchical data structure made of nodes connected by edges.

Unlike a linked list, a tree can branch into multiple children.

Trees are extremely important in Java backend interviews because they test:

- Recursion
- DFS
- BFS
- Binary trees
- Binary Search Trees
- Tree traversal
- Height/depth
- Lowest Common Ancestor
- Path problems
- Serialization
- Tree construction

---

# 1. Basic Tree Structure

Example:

```text
        1
       / \
      2   3
     / \
    4   5
```

Here:

```text
1 → root
2, 3 → children of 1
4, 5 → children of 2
```

---

# 2. Important Tree Terms

### Root

The topmost node.

```text
        1
```

### Parent

A node directly above another node.

```text
1
|
2
```

`1` is the parent of `2`.

### Child

A node directly below another node.

### Leaf

A node with no children.

```text
4, 5, 3
```

are leaves in the example tree.

### Edge

Connection between two nodes.

### Height

Number of edges on the longest path from a node to a leaf.

### Depth

Number of edges from the root to a node.

---

# 3. Binary Tree

A binary tree is a tree where each node has at most:

```text
2 children
```

Usually:

```text
left
right
```

Node:

```java
static class TreeNode {

    int value;

    TreeNode left;
    TreeNode right;

    TreeNode(int value) {
        this.value = value;
    }
}
```

---

# 4. Creating a Binary Tree

```java
TreeNode root =
    new TreeNode(1);

root.left =
    new TreeNode(2);

root.right =
    new TreeNode(3);

root.left.left =
    new TreeNode(4);

root.left.right =
    new TreeNode(5);
```

Result:

```text
        1
       / \
      2   3
     / \
    4   5
```

---

# 5. Types of Binary Trees

### Full Binary Tree

Every node has either:

```text
0 children
or
2 children
```

### Complete Binary Tree

Every level is completely filled except possibly the last, and the last level is filled from left to right.

### Perfect Binary Tree

All internal nodes have two children and all leaves are at the same level.

### Balanced Binary Tree

Height remains approximately:

```text
O(log n)
```

### Skewed Tree

Each node has only one child.

Example:

```text
1
 \
  2
   \
    3
     \
      4
```

This behaves similarly to a linked list.

---

# 6. Tree Traversal

Traversal means visiting every node.

The most important traversals are:

```text
Preorder
Inorder
Postorder
Level Order
```

---

# 7. Preorder Traversal

Order:

```text
Root
Left
Right
```

For:

```text
        1
       / \
      2   3
     / \
    4   5
```

Preorder:

```text
1 2 4 5 3
```

---

# 8. Preorder — Recursive

```java
static void preorder(
        TreeNode root) {

    if (root == null) {
        return;
    }

    System.out.print(
        root.value + " "
    );

    preorder(root.left);

    preorder(root.right);
}
```

Complexity:

```text
Time: O(n)
Space: O(h)
```

where `h` is tree height.

---

# 9. Inorder Traversal

Order:

```text
Left
Root
Right
```

Example:

```text
        1
       / \
      2   3
     / \
    4   5
```

Inorder:

```text
4 2 5 1 3
```

In a Binary Search Tree, inorder traversal produces values in sorted order.

---

# 10. Inorder — Recursive

```java
static void inorder(
        TreeNode root) {

    if (root == null) {
        return;
    }

    inorder(root.left);

    System.out.print(
        root.value + " "
    );

    inorder(root.right);
}
```

---

# 11. Postorder Traversal

Order:

```text
Left
Right
Root
```

Example:

```text
4 5 2 3 1
```

Useful when children must be processed before the parent.

---

# 12. Postorder — Recursive

```java
static void postorder(
        TreeNode root) {

    if (root == null) {
        return;
    }

    postorder(root.left);

    postorder(root.right);

    System.out.print(
        root.value + " "
    );
}
```

---

# 13. Level Order Traversal

Level Order uses:

```text
Queue
```

Example:

```text
        1
       / \
      2   3
     / \
    4   5
```

Output:

```text
1
2 3
4 5
```

---

# 14. Level Order — Java

```java
static List<List<Integer>>
levelOrder(TreeNode root) {

    List<List<Integer>> result =
        new ArrayList<>();

    if (root == null) {
        return result;
    }

    Queue<TreeNode> queue =
        new ArrayDeque<>();

    queue.offer(root);

    while (!queue.isEmpty()) {

        int size =
            queue.size();

        List<Integer> level =
            new ArrayList<>();

        for (int i = 0;
             i < size;
             i++) {

            TreeNode node =
                queue.poll();

            level.add(node.value);

            if (node.left != null) {
                queue.offer(node.left);
            }

            if (node.right != null) {
                queue.offer(node.right);
            }
        }

        result.add(level);
    }

    return result;
}
```

---

# 15. Traversal Summary

```text
Preorder:
Root → Left → Right

Inorder:
Left → Root → Right

Postorder:
Left → Right → Root

Level Order:
Level by Level
```

Memory trick:

```text
PRE
Root comes PRE-viously

IN
Root comes IN between children

POST
Root comes POST after children
```

---

# 16. Iterative Preorder

Use a stack.

```java
static List<Integer>
preorderIterative(TreeNode root) {

    List<Integer> result =
        new ArrayList<>();

    if (root == null) {
        return result;
    }

    Deque<TreeNode> stack =
        new ArrayDeque<>();

    stack.push(root);

    while (!stack.isEmpty()) {

        TreeNode node =
            stack.pop();

        result.add(node.value);

        if (node.right != null) {
            stack.push(node.right);
        }

        if (node.left != null) {
            stack.push(node.left);
        }
    }

    return result;
}
```

Push right first so that left is processed first.

---

# 17. Iterative Inorder

Use a stack and move left as far as possible.

```java
static List<Integer>
inorderIterative(TreeNode root) {

    List<Integer> result =
        new ArrayList<>();

    Deque<TreeNode> stack =
        new ArrayDeque<>();

    TreeNode current = root;

    while (current != null
            || !stack.isEmpty()) {

        while (current != null) {

            stack.push(current);

            current =
                current.left;
        }

        current =
            stack.pop();

        result.add(
            current.value
        );

        current =
            current.right;
    }

    return result;
}
```

---

# 18. Iterative Postorder

Postorder is slightly more complicated.

One approach uses two stacks.

```java
static List<Integer>
postorderIterative(TreeNode root) {

    List<Integer> result =
        new ArrayList<>();

    if (root == null) {
        return result;
    }

    Deque<TreeNode> first =
        new ArrayDeque<>();

    Deque<TreeNode> second =
        new ArrayDeque<>();

    first.push(root);

    while (!first.isEmpty()) {

        TreeNode node =
            first.pop();

        second.push(node);

        if (node.left != null) {
            first.push(node.left);
        }

        if (node.right != null) {
            first.push(node.right);
        }
    }

    while (!second.isEmpty()) {

        result.add(
            second.pop().value
        );
    }

    return result;
}
```

---

# 19. Maximum Depth

Maximum depth is the number of nodes on the longest root-to-leaf path in this definition.

Recursive solution:

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

Complexity:

```text
Time: O(n)
Space: O(h)
```

---

# 20. Minimum Depth

Minimum depth is the number of nodes on the shortest root-to-leaf path.

Be careful:

```text
A node with only one child is not a leaf.
```

Recursive approach:

```java
static int minDepth(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    if (root.left == null) {
        return 1 + minDepth(root.right);
    }

    if (root.right == null) {
        return 1 + minDepth(root.left);
    }

    return 1 + Math.min(
        minDepth(root.left),
        minDepth(root.right)
    );
}
```

---

# 21. Maximum Depth Using BFS

Level order can also calculate depth.

```java
static int maxDepthBfs(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    Queue<TreeNode> queue =
        new ArrayDeque<>();

    queue.offer(root);

    int depth = 0;

    while (!queue.isEmpty()) {

        int size =
            queue.size();

        depth++;

        for (int i = 0;
             i < size;
             i++) {

            TreeNode node =
                queue.poll();

            if (node.left != null) {
                queue.offer(node.left);
            }

            if (node.right != null) {
                queue.offer(node.right);
            }
        }
    }

    return depth;
}
```

---

# 22. Count Nodes

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

---

# 23. Sum of Nodes

```java
static int sumNodes(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    return root.value
        + sumNodes(root.left)
        + sumNodes(root.right);
}
```

---

# 24. Search in Binary Tree

A normal binary tree is not necessarily sorted.

Therefore:

```text
DFS/BFS
```

may be required.

```java
static boolean contains(
        TreeNode root,
        int target) {

    if (root == null) {
        return false;
    }

    if (root.value == target) {
        return true;
    }

    return contains(
            root.left,
            target
        )
        || contains(
            root.right,
            target
        );
}
```

Complexity:

```text
O(n)
```

---

# 25. Same Tree

Two trees are identical if:

```text
values match
+
left subtrees match
+
right subtrees match
```

```java
static boolean isSameTree(
        TreeNode first,
        TreeNode second) {

    if (first == null
            && second == null) {
        return true;
    }

    if (first == null
            || second == null) {
        return false;
    }

    return first.value == second.value
        && isSameTree(
            first.left,
            second.left
        )
        && isSameTree(
            first.right,
            second.right
        );
}
```

---

# 26. Symmetric Tree

A tree is symmetric if its left and right subtrees are mirrors.

Example:

```text
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

Compare:

```text
left.left
with
right.right
```

and:

```text
left.right
with
right.left
```

---

# 27. Symmetric Tree — Java

```java
static boolean isSymmetric(
        TreeNode root) {

    if (root == null) {
        return true;
    }

    return mirror(
        root.left,
        root.right
    );
}

static boolean mirror(
        TreeNode first,
        TreeNode second) {

    if (first == null
            && second == null) {
        return true;
    }

    if (first == null
            || second == null) {
        return false;
    }

    return first.value == second.value
        && mirror(
            first.left,
            second.right
        )
        && mirror(
            first.right,
            second.left
        );
}
```

---

# 28. Invert Binary Tree

Swap:

```text
left
right
```

at every node.

```java
static TreeNode invert(
        TreeNode root) {

    if (root == null) {
        return null;
    }

    TreeNode temp =
        root.left;

    root.left =
        root.right;

    root.right =
        temp;

    invert(root.left);
    invert(root.right);

    return root;
}
```

---

# 29. Balanced Binary Tree

A binary tree is balanced if the height difference between left and right subtrees is at most:

```text
1
```

at every node.

A naive approach recalculates height repeatedly.

An optimized approach calculates height once.

---

# 30. Balanced Tree — O(n)

Return:

```text
height
```

or:

```text
-1
```

if unbalanced.

```java
static boolean isBalanced(
        TreeNode root) {

    return height(root) != -1;
}

static int height(
        TreeNode root) {

    if (root == null) {
        return 0;
    }

    int left =
        height(root.left);

    if (left == -1) {
        return -1;
    }

    int right =
        height(root.right);

    if (right == -1) {
        return -1;
    }

    if (Math.abs(left - right) > 1) {
        return -1;
    }

    return 1 +
        Math.max(left, right);
}
```

Complexity:

```text
O(n)
```

---

# 31. Diameter of Binary Tree

Diameter is the longest path between two nodes.

At a node:

```text
left height
+
right height
```

can form a path through that node.

---

# 32. Diameter — Java

```java
static int diameter(
        TreeNode root) {

    int[] maximum = {0};

    diameterHeight(
        root,
        maximum
    );

    return maximum[0];
}

static int diameterHeight(
        TreeNode root,
        int[] maximum) {

    if (root == null) {
        return 0;
    }

    int left =
        diameterHeight(
            root.left,
            maximum
        );

    int right =
        diameterHeight(
            root.right,
            maximum
        );

    maximum[0] =
        Math.max(
            maximum[0],
            left + right
        );

    return 1 +
        Math.max(left, right);
}
```

---

# 33. Maximum Path Sum

Find the maximum sum of a path between any two nodes.

At each node:

```text
left contribution
+
node value
+
right contribution
```

can form the path through that node.

But when returning upward, only one branch can be used:

```text
node + max(left, right)
```

---

# 34. Lowest Common Ancestor

The Lowest Common Ancestor (LCA) of two nodes is the deepest node that is an ancestor of both.

Example:

```text
        3
       / \
      5   1
     / \
    6   2
```

LCA of:

```text
6 and 2
```

is:

```text
5
```

---

# 35. LCA in a Binary Tree

```java
static TreeNode lowestCommonAncestor(
        TreeNode root,
        TreeNode first,
        TreeNode second) {

    if (root == null
            || root == first
            || root == second) {
        return root;
    }

    TreeNode left =
        lowestCommonAncestor(
            root.left,
            first,
            second
        );

    TreeNode right =
        lowestCommonAncestor(
            root.right,
            first,
            second
        );

    if (left != null
            && right != null) {
        return root;
    }

    return left != null
        ? left
        : right;
}
```

This assumes the target nodes exist in the tree.

---

# 36. Path Sum

Determine whether a root-to-leaf path has a target sum.

```java
static boolean hasPathSum(
        TreeNode root,
        int target) {

    if (root == null) {
        return false;
    }

    if (root.left == null
            && root.right == null) {

        return target == root.value;
    }

    int remaining =
        target - root.value;

    return hasPathSum(
            root.left,
            remaining
        )
        || hasPathSum(
            root.right,
            remaining
        );
}
```

---

# 37. Root-to-Leaf Paths

To generate all root-to-leaf paths:

```text
Add current node
↓
Recurse
↓
Remove current node
```

This combines:

```text
Tree DFS
+
Backtracking
```

---

# 38. Right Side View

Imagine looking at a tree from the right side.

For each level:

```text
rightmost node
```

is visible.

BFS can solve this by storing the final node of each level.

```java
if (i == size - 1) {
    result.add(node.value);
}
```

---

# 39. Left Side View

Similarly:

```text
leftmost node
```

of each level is visible.

With BFS:

```java
if (i == 0) {
    result.add(node.value);
}
```

---

# 40. Zigzag Level Order

Alternate direction:

```text
left → right
right → left
left → right
```

Use BFS.

For every level:

```text
normal insertion
or
insert at front
```

A `Deque<Integer>` can help build each level.

---

# 41. Boundary Traversal

Boundary traversal usually includes:

```text
Root
Left boundary
Leaves
Right boundary in reverse
```

Care must be taken not to duplicate leaf nodes.

This is an advanced traversal problem.

---

# 42. Vertical Order Traversal

Assign each node a column:

```text
root → column 0
left → column -1
right → column +1
```

Then group nodes by column.

Depending on the exact problem, you may also need:

```text
row
value
```

ordering.

---

# 43. Top View

The top view contains the first visible node in each vertical column.

A common approach:

```text
BFS
+
column index
+
HashMap
```

Because BFS visits nodes by increasing depth, the first node recorded for a column is the topmost node.

---

# 44. Bottom View

The bottom view contains the lowest visible node in each vertical column.

With BFS:

```text
overwrite the value for each column
```

so the last encountered node becomes the bottom view.

---

# 45. Serialize a Binary Tree

Serialization converts a tree into a string/sequence.

Example:

```text
1,2,null,null,3,null,null
```

A common approach uses:

```text
Preorder
+
null markers
```

Without null markers, the structure may become ambiguous.

---

# 46. Deserialize a Binary Tree

Read the serialized sequence in the same order:

```text
value
↓
create node
↓
build left
↓
build right
```

The serialization and deserialization rules must match exactly.

---

# 47. Complete Binary Tree

A complete binary tree is useful for:

```text
Heap
```

because its nodes can be stored efficiently in an array.

For zero-based indexing:

```text
left child  = 2 * i + 1
right child = 2 * i + 2
parent      = (i - 1) / 2
```

for valid child/parent positions.

---

# 48. Heap and Tree Connection

A binary heap is a:

```text
Complete Binary Tree
```

with an ordering property.

### Min Heap

```text
parent <= children
```

### Max Heap

```text
parent >= children
```

Java provides:

```java
PriorityQueue
```

for heap functionality.

Detailed heap concepts are covered separately.

---

# 49. Tree Recursion Template

Most binary-tree recursion follows:

```java
static Result solve(
        TreeNode root) {

    if (root == null) {
        return baseValue;
    }

    Result left =
        solve(root.left);

    Result right =
        solve(root.right);

    return combine(
        root,
        left,
        right
    );
}
```

This is one of the most useful templates to memorize.

---

# 50. Tree BFS Template

```java
Queue<TreeNode> queue =
    new ArrayDeque<>();

queue.offer(root);

while (!queue.isEmpty()) {

    int size =
        queue.size();

    for (int i = 0;
         i < size;
         i++) {

        TreeNode node =
            queue.poll();

        // Process node.

        if (node.left != null) {
            queue.offer(node.left);
        }

        if (node.right != null) {
            queue.offer(node.right);
        }
    }
}
```

---

# 51. Tree DFS Template

```java
void dfs(TreeNode root) {

    if (root == null) {
        return;
    }

    // Process root.

    dfs(root.left);

    dfs(root.right);
}
```

Change where the processing occurs to get:

```text
Preorder
Inorder
Postorder
```

---

# 52. Tree Problem Recognition

Think **Tree DFS** when you see:

```text
Height
Depth
Path
Diameter
LCA
Subtree
Maximum path
```

Think **Tree BFS** when you see:

```text
Level
Minimum depth
Right side view
Left side view
Nearest node
Level-by-level processing
```

---

# 53. Common Tree Mistakes

### Mistake 1 — Confusing depth and height

Depth:

```text
root → node
```

Height:

```text
node → deepest leaf
```

---

### Mistake 2 — Incorrect minimum depth

A node with only one child is not a leaf.

---

### Mistake 3 — Forgetting null nodes

Most recursive tree functions need:

```java
if (root == null)
```

---

### Mistake 4 — Using BST logic on a normal binary tree

A normal binary tree has no ordering guarantee.

---

### Mistake 5 — Wrong traversal

Remember:

```text
Pre:
Root Left Right

In:
Left Root Right

Post:
Left Right Root
```

---

### Mistake 6 — Recursing repeatedly for height

This can turn a solution from:

```text
O(n)
```

into:

```text
O(n²)
```

on skewed trees.

---

# 54. Edge Cases

Always test:

```text
Empty tree
One node
Only left children
Only right children
Balanced tree
Skewed tree
Duplicate values
Negative values
Very deep tree
Two identical trees
Symmetric tree
Non-symmetric tree
```

---

# 55. Interview Questions — Easy

1. Maximum Depth.
2. Minimum Depth.
3. Invert Binary Tree.
4. Same Tree.
5. Symmetric Tree.
6. Count Nodes.
7. Sum of Nodes.
8. Preorder Traversal.
9. Inorder Traversal.
10. Postorder Traversal.
11. Level Order Traversal.
12. Search in Binary Tree.

---

# 56. Interview Questions — Medium

13. Balanced Binary Tree.
14. Diameter of Binary Tree.
15. Path Sum.
16. Binary Tree Right Side View.
17. Binary Tree Left Side View.
18. Zigzag Level Order.
19. Lowest Common Ancestor.
20. Construct Tree from Traversals.
21. Serialize and Deserialize.
22. Vertical Order Traversal.
23. Boundary Traversal.
24. Root-to-Leaf Paths.
25. Maximum Path Sum.

---

# 57. Interview Questions — Advanced

26. Binary Tree Maximum Path Sum.
27. Serialize/Deserialize optimized designs.
28. Morris Traversal.
29. Recover a corrupted tree.
30. Flatten Binary Tree to Linked List.
31. Vertical Traversal with ordering constraints.
32. Count Complete Tree Nodes efficiently.
33. Tree DP problems.
34. Advanced LCA problems.
35. Distance between two tree nodes.

---

# 58. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Preorder | DFS | O(n) | O(h) |
| Inorder | DFS | O(n) | O(h) |
| Postorder | DFS | O(n) | O(h) |
| Level Order | BFS | O(n) | O(w) |
| Max Depth | DFS | O(n) | O(h) |
| Min Depth | DFS/BFS | O(n) | O(h) / O(w) |
| Same Tree | DFS | O(n) | O(h) |
| Symmetric Tree | DFS | O(n) | O(h) |
| Invert Tree | DFS | O(n) | O(h) |
| Balanced Tree | DFS | O(n) | O(h) |
| Diameter | DFS | O(n) | O(h) |
| LCA | DFS | O(n) | O(h) |
| Path Sum | DFS | O(n) | O(h) |
| Right Side View | BFS | O(n) | O(w) |
| Serialize | DFS/BFS | O(n) | O(n) |
| Vertical Order | BFS + Map | O(n log n) typically | O(n) |

Where:

```text
n = number of nodes
h = tree height
w = maximum width
```

---

# 59. Quick Revision

```text
Trees
│
├── Traversals
│   ├── Preorder
│   ├── Inorder
│   ├── Postorder
│   └── Level Order
│
├── DFS Problems
│   ├── Height
│   ├── Depth
│   ├── Diameter
│   ├── Path Sum
│   ├── LCA
│   └── Maximum Path Sum
│
├── BFS Problems
│   ├── Level Order
│   ├── Minimum Depth
│   ├── Right View
│   ├── Left View
│   └── Zigzag
│
├── Structure
│   ├── Full
│   ├── Complete
│   ├── Perfect
│   ├── Balanced
│   └── Skewed
│
└── Advanced
    ├── Serialization
    ├── Vertical Traversal
    ├── Boundary Traversal
    ├── Tree Construction
    └── Tree DP
```

---

## Interview Rule

> **For binary trees, master the four traversals first. Then learn to recognize whether a problem is naturally DFS or BFS. Most tree recursion follows the same pattern: solve the left subtree, solve the right subtree, then combine their results at the current node.**
