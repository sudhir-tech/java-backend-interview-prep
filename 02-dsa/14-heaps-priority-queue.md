# DSA — Heaps & Priority Queue

A **Heap** is a specialized tree-based data structure that is commonly used to implement a **Priority Queue**.

Heaps are extremely important in Java backend interviews for:

- Top K problems
- Kth largest/smallest
- Scheduling
- Merging sorted data
- Running median
- Dijkstra's algorithm
- Task prioritization
- Streaming data
- Greedy algorithms

---

# 1. What is a Heap?

A binary heap is a **complete binary tree** that satisfies a heap property.

Two common types:

```text
Min Heap
Max Heap
```

---

# 2. Min Heap

In a min heap:

```text
parent <= children
```

Example:

```text
        1
       / \
      3   2
     / \
    7   5
```

The minimum value is always at the root.

```text
min = 1
```

---

# 3. Max Heap

In a max heap:

```text
parent >= children
```

Example:

```text
        9
       / \
      7   8
     / \
    3   5
```

The maximum value is always at the root.

```text
max = 9
```

---

# 4. Complete Binary Tree

A heap is a complete binary tree.

That means:

```text
Every level is full
except possibly the last,
which is filled from left to right.
```

This structure allows efficient array representation.

---

# 5. Heap Array Representation

For a zero-based array:

```text
parent = (i - 1) / 2
left   = 2 * i + 1
right  = 2 * i + 2
```

Example:

```text
        1
       / \
      3   2
     / \
    7   5
```

Array:

```text
[1, 3, 2, 7, 5]
```

Indexes:

```text
        0
       / \
      1   2
     / \
    3   4
```

---

# 6. Java PriorityQueue

Java provides:

```java
PriorityQueue
```

which implements a priority queue using a heap.

By default:

```text
Min Heap
```

Example:

```java
PriorityQueue<Integer> pq =
    new PriorityQueue<>();
```

---

# 7. Basic PriorityQueue

```java
PriorityQueue<Integer> pq =
    new PriorityQueue<>();

pq.offer(5);
pq.offer(1);
pq.offer(3);

System.out.println(
    pq.peek()
);
// 1

System.out.println(
    pq.poll()
);
// 1
```

The smallest element has the highest priority.

---

# 8. Max Heap in Java

Use a reverse comparator:

```java
PriorityQueue<Integer> maxHeap =
    new PriorityQueue<>(
        Collections.reverseOrder()
    );
```

Or:

```java
PriorityQueue<Integer> maxHeap =
    new PriorityQueue<>(
        Comparator.reverseOrder()
    );
```

Example:

```java
maxHeap.offer(5);
maxHeap.offer(1);
maxHeap.offer(3);

System.out.println(
    maxHeap.peek()
);
// 5
```

---

# 9. PriorityQueue Operations

Common methods:

```java
offer()
add()
peek()
poll()
remove()
isEmpty()
size()
```

Typical complexity:

```text
offer: O(log n)
poll:  O(log n)
peek:  O(1)
```

---

# 10. Why `peek()` is O(1)

The highest-priority element is always stored at:

```text
root
```

For a min heap:

```text
minimum
```

For a max heap:

```text
maximum
```

Therefore:

```java
pq.peek();
```

does not need to search the heap.

---

# 11. Why Insert is O(log n)

When inserting:

```text
1. Add element at the end.
2. Compare with parent.
3. Swap if heap property is violated.
4. Continue upward.
```

This is called:

```text
Heapify Up
```

The maximum number of levels is:

```text
O(log n)
```

---

# 12. Heapify Up

Example min heap:

```text
        2
       / \
      4   5
```

Insert:

```text
1
```

Initially:

```text
        2
       / \
      4   5
     /
    1
```

Compare:

```text
1 < 4
```

Swap:

```text
        2
       / \
      1   5
     /
    4
```

Again:

```text
1 < 2
```

Swap:

```text
        1
       / \
      2   5
     /
    4
```

---

# 13. Heapify Down

Used after removing the root.

Steps:

```text
1. Move last element to root.
2. Compare with children.
3. Swap with appropriate child.
4. Continue downward.
```

This is:

```text
Heapify Down
```

---

# 14. Removing from a Min Heap

Suppose:

```text
        1
       / \
      3   2
     / \
    7   5
```

Remove `1`.

Move last element:

```text
        5
       / \
      3   2
     /
    7
```

Then heapify down.

Compare:

```text
5
```

with:

```text
3, 2
```

Choose smaller child:

```text
2
```

Swap:

```text
        2
       / \
      3   5
     /
    7
```

---

# 15. Kth Largest Element

One of the most important heap problems.

Example:

```text
nums = [3,2,1,5,6,4]
k = 2
```

Answer:

```text
5
```

Use a min heap of size:

```text
k
```

---

# 16. Kth Largest — Java

```java
static int kthLargest(
        int[] nums,
        int k) {

    PriorityQueue<Integer> heap =
        new PriorityQueue<>();

    for (int value : nums) {

        heap.offer(value);

        if (heap.size() > k) {
            heap.poll();
        }
    }

    return heap.peek();
}
```

Complexity:

```text
Time: O(n log k)
Space: O(k)
```

---

# 17. Why Min Heap for Kth Largest?

Keep only the:

```text
largest k elements
```

The smallest among those `k` elements is:

```text
kth largest
```

Therefore the root of the min heap gives the answer.

---

# 18. Kth Smallest

Use a max heap of size `k`.

```java
static int kthSmallest(
        int[] nums,
        int k) {

    PriorityQueue<Integer> heap =
        new PriorityQueue<>(
            Comparator.reverseOrder()
        );

    for (int value : nums) {

        heap.offer(value);

        if (heap.size() > k) {
            heap.poll();
        }
    }

    return heap.peek();
}
```

Complexity:

```text
O(n log k)
```

---

# 19. Top K Largest Elements

Maintain a min heap of size `k`.

```java
PriorityQueue<Integer> heap =
    new PriorityQueue<>();

for (int value : nums) {

    heap.offer(value);

    if (heap.size() > k) {
        heap.poll();
    }
}
```

The heap contains the top `k` values.

---

# 20. Top K Frequent Elements

Suppose:

```text
nums = [1,1,1,2,2,3]
k = 2
```

Answer:

```text
[1,2]
```

First count frequencies:

```java
Map<Integer, Integer> frequency =
    new HashMap<>();
```

Then use a min heap ordered by frequency.

---

# 21. Top K Frequent — Java

```java
static int[] topKFrequent(
        int[] nums,
        int k) {

    Map<Integer, Integer> frequency =
        new HashMap<>();

    for (int value : nums) {

        frequency.merge(
            value,
            1,
            Integer::sum
        );
    }

    PriorityQueue<Integer> heap =
        new PriorityQueue<>(
            Comparator.comparingInt(
                frequency::get
            )
        );

    for (int value :
            frequency.keySet()) {

        heap.offer(value);

        if (heap.size() > k) {
            heap.poll();
        }
    }

    int[] result =
        new int[k];

    for (int i = k - 1;
         i >= 0;
         i--) {

        result[i] =
            heap.poll();
    }

    return result;
}
```

---

# 22. Custom Objects in PriorityQueue

Suppose:

```java
record Task(
    String name,
    int priority
) {}
```

Create:

```java
PriorityQueue<Task> queue =
    new PriorityQueue<>(
        Comparator.comparingInt(
            Task::priority
        )
    );
```

Lowest priority number comes first.

---

# 23. Max Priority for Custom Objects

```java
PriorityQueue<Task> queue =
    new PriorityQueue<>(
        Comparator.comparingInt(
            Task::priority
        ).reversed()
    );
```

Now:

```text
highest priority first
```

---

# 24. Comparator with Multiple Fields

Suppose tasks have:

```text
priority
timestamp
```

Sort by priority first, then timestamp:

```java
PriorityQueue<Task> queue =
    new PriorityQueue<>(
        Comparator
            .comparingInt(Task::priority)
            .thenComparingLong(Task::timestamp)
    );
```

This is useful in scheduling problems.

---

# 25. Merge K Sorted Lists

Given:

```text
1 → 4 → 7
2 → 5 → 8
3 → 6 → 9
```

Use a min heap containing the current smallest node from each list.

Algorithm:

```text
Add first node of each list
↓
Remove smallest
↓
Add its next node
↓
Repeat
```

Complexity:

```text
O(N log k)
```

where:

```text
N = total nodes
k = number of lists
```

---

# 26. Merge K Sorted Arrays

Same idea.

Put the first element from every array into a min heap.

Each heap entry stores:

```text
value
array index
element index
```

When an element is removed:

```text
add next element from same array
```

---

# 27. Find Median from Data Stream

A classic two-heap problem.

Use:

```text
max heap → smaller half
min heap → larger half
```

Maintain:

```text
size difference <= 1
```

Example:

```text
smaller half:
[1,2,3]

larger half:
[4,5,6]
```

The median is determined from the two roots.

---

# 28. Median — Two Heap Structure

```text
       Max Heap
      smaller half
           ↓
          max

          median

           ↑
       Min Heap
      larger half
```

For odd number of elements:

```text
larger heap root
```

or:

```text
smaller heap root
```

can be chosen depending on implementation.

For even number:

```text
average of both roots
```

---

# 29. MedianFinder — Java

```java
class MedianFinder {

    private final PriorityQueue<Integer>
        lower =
            new PriorityQueue<>(
                Comparator.reverseOrder()
            );

    private final PriorityQueue<Integer>
        upper =
            new PriorityQueue<>();

    public void addNum(int num) {

        if (lower.isEmpty()
                || num <= lower.peek()) {

            lower.offer(num);

        } else {

            upper.offer(num);
        }

        if (lower.size()
                > upper.size() + 1) {

            upper.offer(
                lower.poll()
            );
        }

        if (upper.size()
                > lower.size()) {

            lower.offer(
                upper.poll()
            );
        }
    }

    public double findMedian() {

        if (lower.size()
                > upper.size()) {

            return lower.peek();
        }

        return (
            lower.peek()
            + (double) upper.peek()
        ) / 2.0;
    }
}
```

Complexity:

```text
addNum: O(log n)
findMedian: O(1)
```

---

# 30. Sliding Window Median

For every window:

```text
k elements
```

we need the median.

The basic two-heap idea can be extended with:

```text
lazy deletion
```

because arbitrary removal from a Java `PriorityQueue` is not efficient.

This is an advanced heap problem.

---

# 31. PriorityQueue Does Not Support Sorted Iteration

Important Java detail:

```java
PriorityQueue<Integer> pq =
    new PriorityQueue<>();
```

does **not** guarantee that iterating through it gives sorted order.

This:

```java
for (int value : pq) {
    ...
}
```

does not produce sorted values.

Only:

```java
peek()
poll()
```

follow priority ordering.

---

# 32. Removing Arbitrary Elements

This:

```java
pq.remove(value);
```

is generally:

```text
O(n)
```

not:

```text
O(log n)
```

because the heap may need to search for the value.

This matters in advanced sliding-window problems.

---

# 33. Heap Sort

Heap Sort uses a heap to sort elements.

For a max heap:

```text
largest element → root
```

Repeatedly:

```text
swap root with last
reduce heap
heapify
```

Complexity:

```text
Time: O(n log n)
Space: O(1) auxiliary
```

for an in-place array implementation.

---

# 34. Build Heap

Building a heap from an array can be done in:

```text
O(n)
```

using bottom-up heapify.

This surprises many candidates because inserting all elements individually costs:

```text
O(n log n)
```

but bottom-up construction is:

```text
O(n)
```

---

# 35. Bottom-Up Heapify

Start from:

```text
last non-leaf node
```

and heapify down each node.

For zero-based array:

```java
int lastParent =
    (nums.length - 2) / 2;
```

Then move backward toward index `0`.

---

# 36. Why Build Heap is O(n)

Nodes near the bottom have very small heights.

Most nodes require little or no work.

Only a few nodes near the root have large heights.

The total work sums to:

```text
O(n)
```

not:

```text
O(n log n)
```

---

# 37. K Closest Elements

Given sorted numbers and target `x`, find `k` closest values.

One approach:

```text
max heap of size k
```

Keep the closest `k` elements.

Comparator can compare:

```text
absolute difference
```

---

# 38. K Closest Points to Origin

Given:

```text
(x, y)
```

distance:

```text
x² + y²
```

Use a max heap of size `k`.

Why max heap?

```text
largest distance among selected k
```

stays at the root and can be removed when a closer point arrives.

Complexity:

```text
O(n log k)
```

---

# 39. Task Scheduling with PriorityQueue

Suppose tasks have priorities.

```java
PriorityQueue<Task> queue =
    new PriorityQueue<>(
        Comparator.comparingInt(
            Task::priority
        )
    );
```

Then:

```java
Task next =
    queue.poll();
```

returns the highest-priority task according to the comparator.

This pattern appears in:

```text
Schedulers
Job systems
Event processing
Resource allocation
```

---

# 40. Dijkstra's Algorithm

Dijkstra's shortest-path algorithm commonly uses:

```text
PriorityQueue
```

Each queue entry contains:

```text
node
distance
```

The queue always processes the currently smallest known distance.

Typical complexity with adjacency lists and a binary heap:

```text
O((V + E) log V)
```

depending on implementation.

---

# 41. Dijkstra — Core Pattern

```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        Comparator.comparingInt(
            pair -> pair[1]
        )
    );

pq.offer(
    new int[]{source, 0}
);

while (!pq.isEmpty()) {

    int[] current =
        pq.poll();

    int node =
        current[0];

    int distance =
        current[1];

    // Relax neighboring edges.
}
```

---

# 42. Greedy + Heap

Many greedy algorithms use a heap to repeatedly choose:

```text
smallest
```

or:

```text
largest
```

available option.

Examples:

```text
Meeting Rooms
Task Scheduling
Minimum Cost to Connect Sticks
Huffman Coding
IPO
Kth Largest
Merge K Sorted Lists
```

---

# 43. Minimum Cost to Connect Sticks

Given stick lengths:

```text
2, 4, 3
```

Always combine the two smallest:

```text
2 + 3 = 5
```

Then:

```text
4 + 5 = 9
```

Total:

```text
14
```

Use a min heap.

---

# 44. Connect Sticks — Java

```java
static int connectSticks(
        int[] sticks) {

    PriorityQueue<Integer> heap =
        new PriorityQueue<>();

    for (int stick : sticks) {
        heap.offer(stick);
    }

    int cost = 0;

    while (heap.size() > 1) {

        int first =
            heap.poll();

        int second =
            heap.poll();

        int combined =
            first + second;

        cost += combined;

        heap.offer(combined);
    }

    return cost;
}
```

---

# 45. Huffman Coding

Huffman coding repeatedly combines the two smallest frequencies.

This is another classic:

```text
Min Heap
+
Greedy
```

pattern.

Algorithm:

```text
Add all frequencies
↓
Remove two smallest
↓
Combine
↓
Add combined frequency
↓
Repeat
```

---

# 46. Heap vs Sorting

Suppose we need:

```text
top k
```

Sorting:

```text
O(n log n)
```

Heap:

```text
O(n log k)
```

when maintaining only `k` elements.

If:

```text
k << n
```

the heap can be much more efficient.

---

# 47. Heap vs Quickselect

For kth largest:

### Heap

```text
O(n log k)
```

### Quickselect

Average:

```text
O(n)
```

Worst case:

```text
O(n²)
```

Heap is often easier to implement and provides predictable logarithmic behavior for the maintained heap.

---

# 48. Common Heap Mistakes

### Mistake 1 — Using the wrong heap

Kth largest:

```text
min heap of size k
```

Kth smallest:

```text
max heap of size k
```

---

### Mistake 2 — Forgetting heap size

For top K:

```java
if (heap.size() > k) {
    heap.poll();
}
```

is the key idea.

---

### Mistake 3 — Assuming PriorityQueue is a max heap

Java's default:

```java
PriorityQueue
```

is a min heap.

---

### Mistake 4 — Assuming iteration is sorted

Only priority operations guarantee ordering.

---

### Mistake 5 — Removing arbitrary elements

```java
pq.remove(value)
```

is generally O(n).

---

### Mistake 6 — Incorrect comparator

Always verify whether the root should represent:

```text
minimum
maximum
closest
farthest
highest frequency
lowest frequency
```

---

# 49. Edge Cases

Always test:

```text
Empty heap
One element
k = 1
k = n
Duplicate values
All values equal
Negative numbers
Large values
Multiple elements with same priority
Empty PriorityQueue
Custom comparator
```

---

# 50. Interview Questions — Easy

1. Implement a min heap.
2. Implement a max heap.
3. Basic PriorityQueue operations.
4. Kth largest element.
5. Kth smallest element.
6. Top K largest elements.
7. Top K smallest elements.
8. Last stone weight.
9. Minimum cost to connect sticks.
10. K closest points.

---

# 51. Interview Questions — Medium

11. Top K Frequent Elements.
12. Merge K Sorted Lists.
13. Merge K Sorted Arrays.
14. K Closest Elements.
15. Task Scheduler.
16. Find Median from Data Stream.
17. Meeting Rooms II.
18. Reorganize String.
19. Sort Characters by Frequency.
20. Dijkstra's Algorithm.
21. Smallest Range Covering Elements from K Lists.

---

# 52. Interview Questions — Advanced

23. Sliding Window Median.
24. Implement Heap from Scratch.
25. Heap Sort.
26. Build Heap in O(n).
27. Huffman Coding.
28. Median of Multiple Data Streams.
29. Advanced scheduling with multiple priority rules.
30. Indexed priority queue concepts.
31. Dijkstra with custom state.
32. Heap-based greedy optimization.

---

# 53. Complexity Summary

| Operation / Problem | Time | Space |
|---|---:|---:|
| Peek | O(1) | O(1) |
| Insert | O(log n) | O(1) auxiliary |
| Poll | O(log n) | O(1) auxiliary |
| Remove arbitrary value | O(n) | O(1) auxiliary |
| Build Heap | O(n) | O(1) auxiliary |
| Heap Sort | O(n log n) | O(1) auxiliary |
| Kth Largest | O(n log k) | O(k) |
| Kth Smallest | O(n log k) | O(k) |
| Top K Frequent | O(n log k) | O(n) |
| Merge K Lists | O(N log k) | O(k) |
| K Closest Points | O(n log k) | O(k) |
| Median Stream | O(log n) per insert | O(n) |
| Dijkstra | O((V+E) log V) typical | O(V) |

---

# 54. Quick Revision

```text
Heaps & Priority Queue
│
├── Heap
│   ├── Min Heap
│   ├── Max Heap
│   └── Complete Binary Tree
│
├── Java
│   └── PriorityQueue
│
├── Top K
│   ├── Kth Largest
│   ├── Kth Smallest
│   ├── Top K Frequent
│   └── K Closest
│
├── Multiple Heaps
│   └── Running Median
│
├── Merge
│   ├── K Sorted Lists
│   └── K Sorted Arrays
│
├── Algorithms
│   ├── Dijkstra
│   ├── Heap Sort
│   └── Huffman Coding
│
└── Greedy
    ├── Scheduling
    ├── Connect Sticks
    └── Resource Allocation
```

---

## Interview Rule

> **When a problem repeatedly asks for the smallest, largest, highest-priority, or lowest-priority item, think PriorityQueue. For Top K problems, the key trick is usually maintaining a heap of size K.**
