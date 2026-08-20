# DSA — Binary Search Tree (BST)

A **Binary Search Tree (BST)** is a binary tree with an ordering property:

```text
left subtree values < node value < right subtree values
```

For a BST with unique values:

```text
        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4   7 13
```

BSTs are important in Java interviews because they combine:

- Binary tree traversal
- Binary search
- Recursion
- Iteration
- Ordering
- Tree construction
- Lowest Common Ancestor
- Predecessor/successor
- Validation

---

# 1. BST Property

For every node:

```text
all values in left subtree < node
all values in right subtree > node
```

Example:

```text
        8
       / \
      3   10
```

Because:

```text
3 < 8
10 > 8
```

The property must hold recursively for every subtree.

---

# 2. BST Node

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

# 3. Search in BST

Unlike a normal binary tree, we do not need to search both subtrees.

If:

```text
target < root.value
```

search left.

If:

```text
target > root.value
```

search right.

Otherwise:

```text
found
```

---

# 4. Search — Recursive

```java
static TreeNode search(
        TreeNode root,
        int target) {

    if (root == null
            || root.value == target) {

        return root;
    }

    if (target < root.value) {

        return search(
            root.left,
            target
        );
    }

    return search(
        root.right,
        target
    );
}
```

---

# 5. Search — Iterative

```java
static TreeNode searchIterative(
        TreeNode root,
        int target) {

    TreeNode current = root;

    while (current != null) {

        if (current.value == target) {
            return current;
        }

        if (target < current.value) {
            current = current.left;
        } else {
            current = current.right;
        }
    }

    return null;
}
```

Iterative search avoids recursion-stack usage.

---

# 6. Search Complexity

For a balanced BST:

```text
Time: O(log n)
```

For a skewed BST:

```text
Time: O(n)
```

Space:

```text
Iterative: O(1)
Recursive: O(h)
```

where `h` is tree height.

---

# 7. Insert into BST

To insert a value:

```text
Start at root
↓
Compare value
↓
Go left or right
↓
Find null position
↓
Insert
```

---

# 8. Insert — Recursive

```java
static TreeNode insert(
        TreeNode root,
        int value) {

    if (root == null) {
        return new TreeNode(value);
    }

    if (value < root.value) {

        root.left =
            insert(
                root.left,
                value
            );

    } else if (value > root.value) {

        root.right =
            insert(
                root.right,
                value
            );
    }

    return root;
}
```

This version ignores duplicate values.

---

# 9. Insert — Iterative

```java
static TreeNode insertIterative(
        TreeNode root,
        int value) {

    if (root == null) {
        return new TreeNode(value);
    }

    TreeNode current = root;

    while (true) {

        if (value < current.value) {

            if (current.left == null) {

                current.left =
                    new TreeNode(value);

                break;
            }

            current =
                current.left;

        } else if (value > current.value) {

            if (current.right == null) {

                current.right =
                    new TreeNode(value);

                break;
            }

            current =
                current.right;

        } else {

            break;
        }
    }

    return root;
}
```

---

# 10. Inorder Traversal of BST

The most important BST property:

> **Inorder traversal of a BST produces values in sorted order.**

Example:

```text
        5
       / \
      3   7
     / \ / \
    1  4 6  8
```

Inorder:

```text
1 3 4 5 6 7 8
```

---

# 11. Validate BST

A common mistake is checking only immediate children.

Incorrect:

```java
root.left.value < root.value
root.right.value > root.value
```

This is not enough.

Example:

```text
        5
       / \
      3   7
         /
        4
```

`4 < 7`, but `4` cannot be inside the right subtree of `5`.

We need valid bounds.

---

# 12. Validate BST — Bounds

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
        TreeNode root,
        long min,
        long max) {

    if (root == null) {
        return true;
    }

    if (root.value <= min
            || root.value >= max) {

        return false;
    }

    return validate(
            root.left,
            min,
            root.value
        )
        && validate(
            root.right,
            root.value,
            max
        );
}
```

Using `long` bounds avoids problems when node values can be `Integer.MIN_VALUE` or `Integer.MAX_VALUE`.

---

# 13. Validate BST — Inorder

Because inorder traversal must be strictly increasing for a BST with unique values:

```text
previous < current
```

We can validate while traversing.

```java
static boolean isValidBSTInorder(
        TreeNode root) {

    Deque<TreeNode> stack =
        new ArrayDeque<>();

    TreeNode current = root;

    long previous =
        Long.MIN_VALUE;

    while (current != null
            || !stack.isEmpty()) {

        while (current != null) {

            stack.push(current);

            current =
                current.left;
        }

        current =
            stack.pop();

        if (current.value <= previous) {
            return false;
        }

        previous =
            current.value;

        current =
            current.right;
    }

    return true;
}
```

---

# 14. Find Minimum in BST

The minimum value is the:

```text
leftmost node
```

```java
static TreeNode findMin(
        TreeNode root) {

    if (root == null) {
        return null;
    }

    TreeNode current = root;

    while (current.left != null) {
        current =
            current.left;
    }

    return current;
}
```

---

# 15. Find Maximum in BST

The maximum value is the:

```text
rightmost node
```

```java
static TreeNode findMax(
        TreeNode root) {

    if (root == null) {
        return null;
    }

    TreeNode current = root;

    while (current.right != null) {
        current =
            current.right;
    }

    return current;
}
```

---

# 16. Delete from BST

Deletion has three cases.

### Case 1

Node is a leaf:

```text
      5
     /
    3
```

Delete `3`.

Simply remove it.

---

### Case 2

Node has one child:

```text
      5
     /
    3
     \
      4
```

Delete `3`.

Connect:

```text
5 → 4
```

---

### Case 3

Node has two children:

```text
      5
     / \
    3   7
```

Replace it with:

```text
inorder successor
```

or:

```text
inorder predecessor
```

---

# 17. Inorder Successor

The inorder successor is:

```text
smallest value greater than the current node
```

If the node has a right subtree:

```text
successor =
leftmost node of right subtree
```

---

# 18. BST Delete — Java

```java
static TreeNode delete(
        TreeNode root,
        int value) {

    if (root == null) {
        return null;
    }

    if (value < root.value) {

        root.left =
            delete(
                root.left,
                value
            );

    } else if (value > root.value) {

        root.right =
            delete(
                root.right,
                value
            );

    } else {

        if (root.left == null) {
            return root.right;
        }

        if (root.right == null) {
            return root.left;
        }

        TreeNode successor =
            findMin(root.right);

        root.value =
            successor.value;

        root.right =
            delete(
                root.right,
                successor.value
            );
    }

    return root;
}
```

---

# 19. Delete Complexity

Balanced BST:

```text
O(log n)
```

Worst-case skewed BST:

```text
O(n)
```

---

# 20. Find Floor

The floor of `target` is:

```text
largest value <= target
```

Example:

```text
BST values:
1 3 5 7 9

target = 6

floor = 5
```

---

# 21. Floor — Java

```java
static Integer floor(
        TreeNode root,
        int target) {

    Integer answer = null;

    TreeNode current = root;

    while (current != null) {

        if (current.value == target) {
            return current.value;
        }

        if (current.value < target) {

            answer =
                current.value;

            current =
                current.right;

        } else {

            current =
                current.left;
        }
    }

    return answer;
}
```

---

# 22. Find Ceiling

The ceiling of `target` is:

```text
smallest value >= target
```

Example:

```text
BST:
1 3 5 7 9

target = 6

ceiling = 7
```

---

# 23. Ceiling — Java

```java
static Integer ceiling(
        TreeNode root,
        int target) {

    Integer answer = null;

    TreeNode current = root;

    while (current != null) {

        if (current.value == target) {
            return current.value;
        }

        if (current.value > target) {

            answer =
                current.value;

            current =
                current.left;

        } else {

            current =
                current.right;
        }
    }

    return answer;
}
```

---

# 24. Lowest Common Ancestor in BST

BST ordering makes LCA easier.

For nodes:

```text
p
q
```

If both are smaller than root:

```text
go left
```

If both are larger:

```text
go right
```

Otherwise:

```text
current root = LCA
```

---

# 25. LCA in BST — Java

```java
static TreeNode lowestCommonAncestor(
        TreeNode root,
        TreeNode first,
        TreeNode second) {

    TreeNode current = root;

    while (current != null) {

        if (first.value < current.value
                && second.value < current.value) {

            current =
                current.left;

        } else if (
            first.value > current.value
                && second.value > current.value) {

            current =
                current.right;

        } else {

            return current;
        }
    }

    return null;
}
```

---

# 26. Kth Smallest Element

Inorder traversal of a BST is sorted.

Therefore:

```text
1st inorder node = smallest
2nd inorder node = 2nd smallest
...
kth inorder node = kth smallest
```

---

# 27. Kth Smallest — Iterative

```java
static int kthSmallest(
        TreeNode root,
        int k) {

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

        k--;

        if (k == 0) {
            return current.value;
        }

        current =
            current.right;
    }

    throw new IllegalArgumentException(
        "k is larger than the number of nodes"
    );
}
```

---

# 28. Kth Largest Element

Reverse the inorder traversal:

```text
Right
Root
Left
```

The kth visited node is the kth largest.

---

# 29. Range Sum in BST

Given:

```text
low
high
```

calculate the sum of values within the range.

BST pruning helps.

If:

```text
root.value < low
```

we only need the right subtree.

If:

```text
root.value > high
```

we only need the left subtree.

---

# 30. Range Sum — Java

```java
static int rangeSumBST(
        TreeNode root,
        int low,
        int high) {

    if (root == null) {
        return 0;
    }

    if (root.value < low) {

        return rangeSumBST(
            root.right,
            low,
            high
        );
    }

    if (root.value > high) {

        return rangeSumBST(
            root.left,
            low,
            high
        );
    }

    return root.value
        + rangeSumBST(
            root.left,
            low,
            high
        )
        + rangeSumBST(
            root.right,
            low,
            high
        );
}
```

---

# 31. Trim a BST

Remove all nodes outside:

```text
[low, high]
```

If:

```text
root.value < low
```

the root and its left subtree are invalid.

Return the trimmed right subtree.

If:

```text
root.value > high
```

return the trimmed left subtree.

---

# 32. Trim BST — Java

```java
static TreeNode trimBST(
        TreeNode root,
        int low,
        int high) {

    if (root == null) {
        return null;
    }

    if (root.value < low) {

        return trimBST(
            root.right,
            low,
            high
        );
    }

    if (root.value > high) {

        return trimBST(
            root.left,
            low,
            high
        );
    }

    root.left =
        trimBST(
            root.left,
            low,
            high
        );

    root.right =
        trimBST(
            root.right,
            low,
            high
        );

    return root;
}
```

---

# 33. Convert Sorted Array to BST

Given a sorted array:

```text
[1,2,3,4,5,6,7]
```

choose the middle element as root:

```text
        4
       / \
      2   6
     / \ / \
    1  3 5  7
```

This creates a balanced BST.

---

# 34. Sorted Array to BST — Java

```java
static TreeNode sortedArrayToBST(
        int[] nums) {

    return build(
        nums,
        0,
        nums.length - 1
    );
}

static TreeNode build(
        int[] nums,
        int left,
        int right) {

    if (left > right) {
        return null;
    }

    int middle =
        left + (right - left) / 2;

    TreeNode root =
        new TreeNode(nums[middle]);

    root.left =
        build(
            nums,
            left,
            middle - 1
        );

    root.right =
        build(
            nums,
            middle + 1,
            right
        );

    return root;
}
```

---

# 35. Convert BST to Sorted Doubly Linked List

An inorder traversal visits nodes in sorted order.

We can connect:

```text
previous ⇄ current
```

during inorder traversal.

This is a useful advanced tree transformation problem.

---

# 36. Recover a Corrupted BST

Sometimes two BST nodes are swapped accidentally.

Example:

```text
Expected:
1 2 3 4

Actual:
1 3 2 4
```

Inorder traversal should be sorted.

Find the nodes where:

```text
previous.value > current.value
```

Then swap the misplaced values.

---

# 37. Construct BST from Preorder

Example:

```text
Preorder:
8 5 1 7 10 12
```

Use BST bounds.

At each value:

```text
if value is inside current bounds
→ create node
```

Then recursively build:

```text
left subtree
right subtree
```

This can be solved in:

```text
O(n)
```

using a preorder index and bounds.

---

# 38. Construct BST from Sorted Data

Sorted data can produce a balanced BST if we repeatedly choose the middle element.

Pattern:

```text
middle → root
left half → left subtree
right half → right subtree
```

This is also useful when converting:

```text
sorted array
```

into an efficient search tree.

---

# 39. BST Iterator

A BST iterator can return values in ascending order.

Use a stack containing the path to the next smallest node.

Initialization:

```text
push all left nodes
```

`next()`:

```text
pop node
push left path of its right subtree
return node
```

This provides:

```text
next() → O(1) amortized
```

with:

```text
O(h)
```

space.

---

# 40. BST Iterator — Core Idea

```java
private void pushLeft(
        TreeNode node) {

    while (node != null) {

        stack.push(node);

        node =
            node.left;
    }
}
```

Then:

```java
public int next() {

    TreeNode node =
        stack.pop();

    pushLeft(node.right);

    return node.value;
}
```

---

# 41. Predecessor and Successor

For a target:

```text
predecessor =
largest value < target

successor =
smallest value > target
```

BST ordering allows both to be found efficiently.

Typical approach:

```text
Search while tracking the best candidate.
```

---

# 42. BST vs Binary Tree

### Binary Tree

Only rule:

```text
At most two children.
```

No ordering guarantee.

Search:

```text
O(n)
```

### BST

Rule:

```text
left < root < right
```

Balanced search:

```text
O(log n)
```

Worst-case:

```text
O(n)
```

---

# 43. BST vs Heap

### BST

Optimized for:

```text
Ordered searching
Predecessor/successor
Range queries
Sorted traversal
```

### Heap

Optimized for:

```text
Minimum/maximum
Priority Queue
Top K
Scheduling
```

A heap is not globally sorted.

---

# 44. Balanced BST

Examples include self-balancing trees such as:

```text
AVL Tree
Red-Black Tree
```

They maintain tree height around:

```text
O(log n)
```

This prevents the worst-case linked-list-like shape.

Java's `TreeMap` and `TreeSet` are based on a red-black tree implementation.

---

# 45. TreeMap

Java:

```java
TreeMap<Integer, String> map =
    new TreeMap<>();
```

It maintains keys in sorted order.

Common operations:

```java
map.firstKey();
map.lastKey();

map.floorKey(key);
map.ceilingKey(key);

map.lowerKey(key);
map.higherKey(key);
```

These are extremely useful when a problem asks for predecessor/successor behavior.

---

# 46. TreeSet

```java
TreeSet<Integer> set =
    new TreeSet<>();
```

Useful methods:

```java
set.first();
set.last();

set.floor(value);
set.ceiling(value);

set.lower(value);
set.higher(value);
```

Typical complexity:

```text
O(log n)
```

for search/insertion/removal.

---

# 47. BST Range Query

Suppose we need:

```text
all values between L and R
```

A BST allows pruning.

If:

```text
root.value < L
```

ignore the left subtree.

If:

```text
root.value > R
```

ignore the right subtree.

This can be much more efficient than scanning every node in a balanced tree.

---

# 48. BST Complexity

For height `h`:

```text
Search: O(h)
Insert: O(h)
Delete: O(h)
Min: O(h)
Max: O(h)
LCA: O(h)
```

Balanced:

```text
h = O(log n)
```

Skewed:

```text
h = O(n)
```

---

# 49. Common BST Mistakes

### Mistake 1 — Assuming every binary tree is a BST

A binary tree has no ordering guarantee.

---

### Mistake 2 — Validating only children

You need global subtree bounds.

---

### Mistake 3 — Forgetting duplicates

Decide whether duplicates are:

```text
not allowed
allowed on left
allowed on right
```

before implementing.

---

### Mistake 4 — Forgetting skewed trees

A BST can degrade to:

```text
O(n)
```

height.

---

### Mistake 5 — Using the wrong successor

For a node with a right subtree:

```text
successor = leftmost node of right subtree
```

---

### Mistake 6 — Confusing inorder with preorder

BST sorted order is:

```text
INORDER
```

not preorder.

---

# 50. Edge Cases

Always test:

```text
Empty BST
One node
Two nodes
Search root
Search missing value
Insert smallest
Insert largest
Delete leaf
Delete node with one child
Delete node with two children
Delete root
Duplicate values
Skewed BST
Balanced BST
Integer.MIN_VALUE
Integer.MAX_VALUE
```

---

# 51. Interview Questions — Easy

1. Search in BST.
2. Insert into BST.
3. Find minimum.
4. Find maximum.
5. Validate BST.
6. Inorder traversal.
7. Lowest Common Ancestor in BST.
8. Kth Smallest Element.
9. Kth Largest Element.
10. Range Sum of BST.

---

# 52. Interview Questions — Medium

11. Delete Node in BST.
12. Convert Sorted Array to BST.
13. Trim a BST.
14. BST Iterator.
15. Find Floor and Ceiling.
16. Find Inorder Successor.
17. Find Inorder Predecessor.
18. Convert BST to Sorted Doubly Linked List.
19. Construct BST from Preorder.
20. Recover Binary Search Tree.

---

# 53. Interview Questions — Advanced

21. Design an ordered set.
22. Design an ordered map.
23. Implement AVL Tree.
24. Implement Red-Black Tree concepts.
25. Augmented BST for range queries.
26. Count nodes in a range.
27. Rank queries using an augmented BST.
28. Dynamic order-statistics tree.
29. Serialize and deserialize BST efficiently.
30. Build a balanced BST from streaming data.

---

# 54. Complexity Summary

| Operation | Balanced BST | Skewed BST |
|---|---:|---:|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Minimum | O(log n) | O(n) |
| Maximum | O(log n) | O(n) |
| LCA | O(log n) | O(n) |
| Floor | O(log n) | O(n) |
| Ceiling | O(log n) | O(n) |
| Inorder Traversal | O(n) | O(n) |

---

# 55. Quick Revision

```text
Binary Search Tree
│
├── Property
│   ├── Left < Root
│   └── Right > Root
│
├── Operations
│   ├── Search
│   ├── Insert
│   └── Delete
│
├── Ordered Queries
│   ├── Min
│   ├── Max
│   ├── Floor
│   ├── Ceiling
│   ├── Predecessor
│   └── Successor
│
├── Traversal
│   └── Inorder = Sorted Order
│
├── Important Problems
│   ├── Validate BST
│   ├── Kth Smallest
│   ├── LCA
│   ├── Range Sum
│   └── Trim BST
│
└── Java
    ├── TreeMap
    └── TreeSet
```

---

## Interview Rule

> **The key to BST problems is the ordering property: left values are smaller and right values are larger. Whenever you see a BST, immediately think “Can I eliminate half the tree?” Also remember that inorder traversal of a valid BST gives sorted values.**
