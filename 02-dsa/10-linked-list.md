# DSA — Linked List

A **Linked List** is a linear data structure made of nodes.

Each node typically contains:

```text
data
+
reference to the next node
```

Unlike arrays, linked-list elements are not required to be stored next to each other in memory.

Linked Lists are important for Java backend interviews because they test:

- References
- Pointer manipulation
- Two-pointer techniques
- Recursion
- Data structure design
- Memory concepts
- Edge-case handling

---

# 1. Basic Linked List

A singly linked list looks like:

```text
10 → 20 → 30 → null
```

Each node stores:

```text
value
next
```

---

# 2. Node Class

```java
static class ListNode {

    int value;
    ListNode next;

    ListNode(int value) {
        this.value = value;
    }
}
```

A list:

```java
ListNode head =
    new ListNode(10);

head.next =
    new ListNode(20);

head.next.next =
    new ListNode(30);
```

---

# 3. Linked List vs Array

### Array

```text
Fast random access
nums[i]
```

Typical:

```text
Access: O(1)
Search: O(n)
```

### Linked List

```text
Sequential access
```

Typical:

```text
Access: O(n)
Search: O(n)
```

Insertion/deletion can be O(1) when you already have the correct node/reference.

---

# 4. Singly Linked List

Each node points only to the next node:

```text
10 → 20 → 30 → null
```

Advantages:

```text
Simple
Less memory than doubly linked list
Easy insertion/deletion at known positions
```

---

# 5. Doubly Linked List

Each node has:

```text
previous
value
next
```

Example:

```text
null ← 10 ⇄ 20 ⇄ 30 → null
```

Node:

```java
static class DoublyNode {

    int value;

    DoublyNode previous;
    DoublyNode next;

    DoublyNode(int value) {
        this.value = value;
    }
}
```

---

# 6. Circular Linked List

The last node points back to the first:

```text
10 → 20 → 30
↑         ↓
└─────────┘
```

Useful for:

```text
Round-robin scheduling
Circular buffers
Repeated traversal
```

---

# 7. Traversing a Linked List

```java
static void printList(
        ListNode head) {

    ListNode current = head;

    while (current != null) {

        System.out.print(
            current.value + " "
        );

        current =
            current.next;
    }
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 8. Find Length

```java
static int length(
        ListNode head) {

    int count = 0;

    ListNode current = head;

    while (current != null) {

        count++;
        current =
            current.next;
    }

    return count;
}
```

---

# 9. Search in Linked List

```java
static boolean contains(
        ListNode head,
        int target) {

    ListNode current = head;

    while (current != null) {

        if (current.value == target) {
            return true;
        }

        current =
            current.next;
    }

    return false;
}
```

---

# 10. Insert at Head

```java
static ListNode insertAtHead(
        ListNode head,
        int value) {

    ListNode newNode =
        new ListNode(value);

    newNode.next = head;

    return newNode;
}
```

### Complexity

```text
Time: O(1)
Space: O(1)
```

---

# 11. Insert at Tail

```java
static ListNode insertAtTail(
        ListNode head,
        int value) {

    ListNode newNode =
        new ListNode(value);

    if (head == null) {
        return newNode;
    }

    ListNode current = head;

    while (current.next != null) {
        current =
            current.next;
    }

    current.next = newNode;

    return head;
}
```

Without a tail pointer:

```text
Time: O(n)
```

With a tail pointer:

```text
Time: O(1)
```

---

# 12. Delete First Occurrence

```java
static ListNode deleteValue(
        ListNode head,
        int target) {

    if (head == null) {
        return null;
    }

    if (head.value == target) {
        return head.next;
    }

    ListNode current = head;

    while (current.next != null) {

        if (current.next.value
                == target) {

            current.next =
                current.next.next;

            return head;
        }

        current =
            current.next;
    }

    return head;
}
```

---

# 13. Reverse a Linked List

This is one of the most important linked-list interview problems.

Input:

```text
1 → 2 → 3 → 4 → null
```

Output:

```text
4 → 3 → 2 → 1 → null
```

Use three references:

```text
previous
current
next
```

---

# 14. Reverse Linked List — Java

```java
static ListNode reverse(
        ListNode head) {

    ListNode previous = null;
    ListNode current = head;

    while (current != null) {

        ListNode next =
            current.next;

        current.next =
            previous;

        previous =
            current;

        current =
            next;
    }

    return previous;
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 15. Reverse Linked List — Pointer Visualization

Before:

```text
null    1 → 2 → 3 → null
 ↑      ↑
prev   curr
```

After processing `1`:

```text
null ← 1    2 → 3 → null
       ↑    ↑
      prev curr
```

After processing `2`:

```text
null ← 1 ← 2    3 → null
            ↑   ↑
           prev curr
```

Continue until:

```text
null ← 1 ← 2 ← 3
```

Then:

```text
previous
```

is the new head.

---

# 16. Recursive Reverse

```java
static ListNode reverseRecursive(
        ListNode head) {

    if (head == null
            || head.next == null) {

        return head;
    }

    ListNode newHead =
        reverseRecursive(
            head.next
        );

    head.next.next =
        head;

    head.next = null;

    return newHead;
}
```

### Complexity

```text
Time: O(n)
Space: O(n)
```

because of the recursion stack.

---

# 17. Find Middle of Linked List

Use:

```text
slow pointer
fast pointer
```

Slow moves:

```text
1 step
```

Fast moves:

```text
2 steps
```

When fast reaches the end:

```text
slow = middle
```

---

# 18. Middle of Linked List — Java

```java
static ListNode middleNode(
        ListNode head) {

    ListNode slow = head;
    ListNode fast = head;

    while (fast != null
            && fast.next != null) {

        slow =
            slow.next;

        fast =
            fast.next.next;
    }

    return slow;
}
```

For an even-length list, this returns the **second middle**.

---

# 19. Detect Cycle

Use Floyd's Cycle Detection Algorithm.

Two pointers:

```text
slow → 1 step
fast → 2 steps
```

If there is a cycle:

```text
slow == fast
```

at some point.

---

# 20. Cycle Detection — Java

```java
static boolean hasCycle(
        ListNode head) {

    ListNode slow = head;
    ListNode fast = head;

    while (fast != null
            && fast.next != null) {

        slow =
            slow.next;

        fast =
            fast.next.next;

        if (slow == fast) {
            return true;
        }
    }

    return false;
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 21. Why Cycle Detection Works

If both pointers enter a cycle:

```text
fast
→
→
→

slow
→
```

Fast gains one node per iteration relative to slow.

Because the cycle is finite:

```text
fast eventually catches slow
```

This is similar to two runners moving around a circular track.

---

# 22. Find Cycle Start

After:

```text
slow == fast
```

reset one pointer to:

```text
head
```

Then move both one step at a time.

Their meeting point is the cycle start.

```java
static ListNode detectCycle(
        ListNode head) {

    ListNode slow = head;
    ListNode fast = head;

    while (fast != null
            && fast.next != null) {

        slow =
            slow.next;

        fast =
            fast.next.next;

        if (slow == fast) {

            ListNode pointer =
                head;

            while (pointer != slow) {

                pointer =
                    pointer.next;

                slow =
                    slow.next;
            }

            return pointer;
        }
    }

    return null;
}
```

---

# 23. Remove Nth Node from End

Use two pointers.

First move:

```text
fast
```

ahead by `n`.

Then move:

```text
slow
fast
```

together.

When `fast` reaches the end:

```text
slow
```

is positioned before the node to delete.

---

# 24. Remove Nth from End — Java

Using a dummy node simplifies the head-removal case.

```java
static ListNode removeNthFromEnd(
        ListNode head,
        int n) {

    ListNode dummy =
        new ListNode(0);

    dummy.next = head;

    ListNode slow = dummy;
    ListNode fast = dummy;

    for (int i = 0;
         i <= n;
         i++) {

        fast =
            fast.next;
    }

    while (fast != null) {

        slow =
            slow.next;

        fast =
            fast.next;
    }

    slow.next =
        slow.next.next;

    return dummy.next;
}
```

---

# 25. Why Use a Dummy Node?

A dummy node makes the list look like:

```text
dummy → head → ...
```

Now deleting the first actual node becomes the same pointer operation as deleting any other node.

Dummy nodes are extremely useful in linked-list problems.

---

# 26. Merge Two Sorted Lists

Example:

```text
1 → 3 → 5
2 → 4 → 6
```

Result:

```text
1 → 2 → 3 → 4 → 5 → 6
```

Use two pointers.

---

# 27. Merge Two Sorted Lists — Java

```java
static ListNode mergeTwoLists(
        ListNode first,
        ListNode second) {

    ListNode dummy =
        new ListNode(0);

    ListNode current = dummy;

    while (first != null
            && second != null) {

        if (first.value
                <= second.value) {

            current.next = first;
            first = first.next;

        } else {

            current.next = second;
            second = second.next;
        }

        current =
            current.next;
    }

    current.next =
        first != null
            ? first
            : second;

    return dummy.next;
}
```

### Complexity

```text
Time: O(n + m)
Space: O(1)
```

---

# 28. Merge K Sorted Lists

If there are `k` sorted linked lists, a common approach is:

```text
PriorityQueue
```

Store the smallest current node from each list.

Process:

```text
remove smallest
↓
add its next node
↓
repeat
```

Complexity:

```text
O(N log k)
```

where `N` is the total number of nodes.

---

# 29. Palindrome Linked List

Example:

```text
1 → 2 → 2 → 1
```

is a palindrome.

Approach:

```text
1. Find middle.
2. Reverse second half.
3. Compare both halves.
```

---

# 30. Palindrome Linked List — Java

```java
static boolean isPalindrome(
        ListNode head) {

    if (head == null
            || head.next == null) {
        return true;
    }

    ListNode slow = head;
    ListNode fast = head;

    while (fast != null
            && fast.next != null) {

        slow =
            slow.next;

        fast =
            fast.next.next;
    }

    ListNode secondHalf =
        reverse(slow);

    ListNode firstHalf =
        head;

    while (secondHalf != null) {

        if (firstHalf.value
                != secondHalf.value) {

            return false;
        }

        firstHalf =
            firstHalf.next;

        secondHalf =
            secondHalf.next;
    }

    return true;
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 31. Intersection of Two Linked Lists

Two lists may merge:

```text
A: 1 → 2
         ↘
           8 → 9
         ↗
B:   4 → 5
```

The key is to align the effective paths.

A neat two-pointer trick:

```text
pointer A traverses A then B
pointer B traverses B then A
```

If an intersection exists, they meet there.

---

# 32. Intersection — Java

```java
static ListNode getIntersectionNode(
        ListNode headA,
        ListNode headB) {

    ListNode a = headA;
    ListNode b = headB;

    while (a != b) {

        a =
            a == null
                ? headB
                : a.next;

        b =
            b == null
                ? headA
                : b.next;
    }

    return a;
}
```

### Complexity

```text
Time: O(n + m)
Space: O(1)
```

---

# 33. Why the Intersection Trick Works

Pointer A travels:

```text
lengthA + lengthB
```

Pointer B travels:

```text
lengthB + lengthA
```

Therefore both have traversed the same total distance.

This automatically aligns them at the intersection.

---

# 34. Add Two Numbers

Numbers are represented in reverse order.

Example:

```text
2 → 4 → 3
```

represents:

```text
342
```

and:

```text
5 → 6 → 4
```

represents:

```text
465
```

Result:

```text
7 → 0 → 8
```

represents:

```text
807
```

Use:

```text
carry
```

while traversing both lists.

---

# 35. Add Two Numbers — Java

```java
static ListNode addTwoNumbers(
        ListNode first,
        ListNode second) {

    ListNode dummy =
        new ListNode(0);

    ListNode current = dummy;

    int carry = 0;

    while (first != null
            || second != null
            || carry != 0) {

        int sum = carry;

        if (first != null) {
            sum += first.value;
            first =
                first.next;
        }

        if (second != null) {
            sum += second.value;
            second =
                second.next;
        }

        current.next =
            new ListNode(sum % 10);

        carry =
            sum / 10;

        current =
            current.next;
    }

    return dummy.next;
}
```

---

# 36. Reorder List

Example:

```text
1 → 2 → 3 → 4
```

becomes:

```text
1 → 4 → 2 → 3
```

For:

```text
1 → 2 → 3 → 4 → 5
```

becomes:

```text
1 → 5 → 2 → 4 → 3
```

Approach:

```text
1. Find middle.
2. Reverse second half.
3. Merge alternating nodes.
```

This combines several linked-list patterns.

---

# 37. Reorder List — Core Structure

```text
Find middle
     ↓
Split
     ↓
Reverse second half
     ↓
Merge:
first → second → first → second
```

This is an excellent interview problem because it combines:

```text
Fast/slow pointers
+
Reverse list
+
Pointer manipulation
```

---

# 38. Reverse Nodes in K-Group

Given:

```text
1 → 2 → 3 → 4 → 5
k = 2
```

result:

```text
2 → 1 → 4 → 3 → 5
```

For:

```text
k = 3
```

result:

```text
3 → 2 → 1 → 4 → 5
```

The final incomplete group usually remains unchanged.

---

# 39. Reverse K-Group — Strategy

For each group:

```text
1. Check whether k nodes exist.
2. Reverse exactly k nodes.
3. Connect the previous group.
4. Continue.
```

This is a hard pointer-manipulation problem.

---

# 40. Swap Nodes in Pairs

Example:

```text
1 → 2 → 3 → 4
```

becomes:

```text
2 → 1 → 4 → 3
```

Use a dummy node and repeatedly swap two nodes.

```java
static ListNode swapPairs(
        ListNode head) {

    ListNode dummy =
        new ListNode(0);

    dummy.next = head;

    ListNode current = dummy;

    while (current.next != null
            && current.next.next != null) {

        ListNode first =
            current.next;

        ListNode second =
            first.next;

        first.next =
            second.next;

        second.next =
            first;

        current.next =
            second;

        current =
            first;
    }

    return dummy.next;
}
```

---

# 41. Partition List

Given a value `x`, arrange nodes so that:

```text
nodes < x
```

come before:

```text
nodes >= x
```

while preserving relative order.

Use two lists:

```text
before
after
```

Then connect them.

Dummy nodes make the implementation easier.

---

# 42. Delete Node Without Head

A classic trick:

```text
Copy next node's value
into current node
```

Then:

```text
current.next =
    current.next.next;
```

This works when the node is guaranteed not to be the tail.

---

# 43. Sort a Linked List

Merge Sort is ideal for linked lists.

Why?

Linked lists do not support efficient random access, so array-based sorting approaches are less natural.

Merge Sort gives:

```text
Time: O(n log n)
```

and can be implemented with linked-list pointer operations.

---

# 44. Merge Sort on Linked List

Steps:

```text
1. Find middle.
2. Split into two lists.
3. Recursively sort each half.
4. Merge sorted halves.
```

This uses:

```text
Fast/slow pointers
+
Recursion
+
Merge
```

---

# 45. Detect and Remove Cycle

First:

```text
detect cycle
```

using Floyd's algorithm.

Then:

```text
find cycle start
```

Then find the node whose:

```text
next == cycleStart
```

and set:

```java
node.next = null;
```

---

# 46. Circular Linked List Detection

If the list is circular:

```text
fast/slow
```

will eventually meet.

The same Floyd algorithm works.

---

# 47. Linked List with Random Pointer

Some linked-list nodes contain:

```text
next
random
```

where `random` can point to any node.

A common approach uses:

```text
HashMap<OriginalNode, CopyNode>
```

Another advanced O(1)-extra-space approach interleaves copied nodes with original nodes.

---

# 48. Copy List with Random Pointer — HashMap Idea

First pass:

```text
original node
→ copied node
```

Store:

```java
Map<Node, Node> map =
    new HashMap<>();
```

Second pass:

```text
copy.next =
    map.get(original.next)

copy.random =
    map.get(original.random)
```

### Complexity

```text
Time: O(n)
Space: O(n)
```

---

# 49. LRU Cache

A very important Java/backend design problem.

An LRU cache needs:

```text
O(1) get
O(1) put
```

A classic design combines:

```text
HashMap
+
Doubly Linked List
```

HashMap gives:

```text
key → node
```

Doubly linked list maintains:

```text
most recently used
        ↓
...
        ↓
least recently used
```

---

# 50. Why Doubly Linked List for LRU?

We need to remove a node from the middle in:

```text
O(1)
```

With a doubly linked list, we have:

```text
previous
next
```

so removal is straightforward.

The HashMap gives direct access to the node.

---

# 51. LRU Cache Structure

```text
HashMap
   ↓
key → Node

Doubly Linked List:

HEAD
 ↓
Most Recent
 ↓
...
 ↓
Least Recent
 ↓
TAIL
```

When an item is accessed:

```text
remove from current position
↓
move to front
```

When capacity is exceeded:

```text
remove tail
↓
remove key from map
```

---

# 52. Java's LinkedHashMap

Java already provides an implementation pattern useful for LRU caches:

```java
LinkedHashMap
```

with:

```java
accessOrder = true
```

Example:

```java
LinkedHashMap<Integer, Integer> cache =
    new LinkedHashMap<>(
        16,
        0.75f,
        true
    );
```

The third argument:

```text
true
```

enables access-order iteration.

---

# 53. Linked List Problem Recognition

Think linked list when you see:

```text
Node
next
previous
Pointer
Reverse
Cycle
Middle
Intersection
Merge
Reorder
Remove nth from end
```

---

# 54. Two-Pointer Linked List Patterns

### Slow/Fast

Used for:

```text
Middle
Cycle detection
Cycle start
Palindrome
```

### Gap pointers

Used for:

```text
Nth node from end
```

### Two list pointers

Used for:

```text
Merge
Intersection
```

---

# 55. Dummy Node Pattern

Use:

```java
ListNode dummy =
    new ListNode(0);

dummy.next = head;
```

Then:

```java
return dummy.next;
```

Dummy nodes simplify:

```text
Insertion at head
Deletion at head
Merging lists
Swapping nodes
Partitioning
```

---

# 56. Pointer Safety

Before doing:

```java
current.next.next
```

make sure:

```java
current != null
current.next != null
```

Pointer bugs often come from dereferencing:

```text
null
```

---

# 57. Common Linked List Mistakes

### Mistake 1 — Losing the next node

Before changing:

```java
current.next
```

save it:

```java
ListNode next =
    current.next;
```

This is essential during reversal.

---

### Mistake 2 — Forgetting to update head

Operations affecting the first node may return a new head.

---

### Mistake 3 — Creating accidental cycles

Incorrect pointer assignments can cause:

```text
1 → 2 → 3
    ↑   ↓
    └───┘
```

Always reason about every changed pointer.

---

### Mistake 4 — Not using a dummy node

Many head-removal problems become unnecessarily complicated without one.

---

### Mistake 5 — Incorrect fast/slow initialization

For:

```text
middle
cycle
```

small differences in initialization can change whether you get the first or second middle.

---

### Mistake 6 — Forgetting the tail

After reversing or rearranging, make sure the final node points to:

```text
null
```

when appropriate.

---

# 58. Edge Cases

Always test:

```text
Empty list
One node
Two nodes
Even length
Odd length
Duplicate values
All values equal
Cycle
No cycle
Cycle at head
Cycle near tail
Remove head
Remove tail
Remove only node
n = 1
n = list length
Two lists do not intersect
Two lists intersect at head
```

---

# 59. Interview Questions — Easy

1. Traverse a linked list.
2. Find length.
3. Search for a value.
4. Insert at head.
5. Insert at tail.
6. Delete a node.
7. Reverse a linked list.
8. Find middle of linked list.
9. Detect cycle.
10. Merge two sorted lists.

---

# 60. Interview Questions — Medium

11. Remove Nth Node From End.
12. Palindrome Linked List.
13. Intersection of Two Linked Lists.
14. Add Two Numbers.
15. Reorder List.
16. Swap Nodes in Pairs.
17. Partition List.
18. Find Cycle Start.
19. Remove Cycle.
20. Sort Linked List.
21. Copy List with Random Pointer.
22. Rotate List.
23. Odd-Even Linked List.

---

# 61. Interview Questions — Advanced

24. Reverse Nodes in K-Group.
25. Merge K Sorted Lists.
26. LRU Cache.
27. Flatten a Multilevel Doubly Linked List.
28. Design a linked-list based cache.
29. Clone complex linked structures.
30. Advanced pointer manipulation.

---

# 62. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Traverse | Iteration | O(n) | O(1) |
| Search | Iteration | O(n) | O(1) |
| Insert Head | Pointer | O(1) | O(1) |
| Insert Tail | Traversal | O(n) | O(1) |
| Reverse | Three Pointers | O(n) | O(1) |
| Find Middle | Slow/Fast | O(n) | O(1) |
| Detect Cycle | Floyd | O(n) | O(1) |
| Cycle Start | Floyd | O(n) | O(1) |
| Remove Nth End | Two Pointers | O(n) | O(1) |
| Merge Two Lists | Two Pointers | O(n + m) | O(1) |
| Palindrome | Reverse Half | O(n) | O(1) |
| Intersection | Two Pointers | O(n + m) | O(1) |
| Add Two Numbers | Carry | O(max(n,m)) | O(max(n,m)) for result |
| Merge K Lists | Heap | O(N log k) | O(k) |
| Sort List | Merge Sort | O(n log n) | O(log n) recursion |
| Reverse K Group | Pointer Manipulation | O(n) | O(1) |
| LRU Cache | HashMap + DLL | O(1) avg | O(capacity) |

---

# 63. Quick Revision

```text
Linked List
│
├── Basics
│   ├── Traversal
│   ├── Insert
│   ├── Delete
│   └── Search
│
├── Pointer Patterns
│   ├── Reverse
│   ├── Slow/Fast
│   ├── Gap Pointers
│   └── Two List Pointers
│
├── Fast/Slow
│   ├── Middle
│   ├── Cycle
│   ├── Cycle Start
│   └── Palindrome
│
├── Merge
│   ├── Two Sorted Lists
│   └── K Sorted Lists
│
├── Rearrangement
│   ├── Reorder
│   ├── Swap Pairs
│   ├── Reverse K Group
│   └── Partition
│
├── Advanced
│   ├── Random Pointer
│   ├── Sort List
│   └── LRU Cache
│
└── Design
    ├── Singly Linked List
    ├── Doubly Linked List
    └── Circular Linked List
```

---

## Interview Rule

> **Linked-list problems are mostly about pointer control. Before changing a pointer, know exactly where the next node is. The highest-value patterns are dummy nodes, slow/fast pointers, gap pointers, and the three-pointer reversal technique.**
