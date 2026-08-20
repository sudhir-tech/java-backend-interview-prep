# DSA — Queue & Deque

Queue and Deque are fundamental data structures for Java backend interviews.

A **Queue** follows:

```text
FIFO
First In, First Out
```

A **Deque** supports insertion and removal from both ends.

They are commonly used in:

- BFS
- Level-order traversal
- Sliding Window Maximum
- Task scheduling
- Producer-consumer systems
- Caching
- Rate limiting
- Monotonic deque problems
- Shortest path in unweighted graphs
- Topological sorting

---

# 1. What is a Queue?

Example:

```text
10 → 20 → 30
↑           ↑
front       rear
```

The first element inserted is the first element removed.

Operations:

```text
enqueue → add
dequeue → remove
peek    → inspect front
```

Typical complexity:

```text
enqueue: O(1)
dequeue: O(1)
peek:    O(1)
```

---

# 2. Queue in Java

Use:

```java
Queue<Integer> queue =
    new ArrayDeque<>();
```

Add:

```java
queue.offer(10);
```

Remove:

```java
queue.poll();
```

Peek:

```java
queue.peek();
```

Check empty:

```java
queue.isEmpty();
```

---

# 3. `offer()` vs `add()`

Both can insert into a queue.

```java
queue.offer(value);
queue.add(value);
```

For queues, `offer()` is generally preferred when you want queue-style semantics.

For bounded queues, `offer()` can indicate failure instead of throwing an exception.

---

# 4. `poll()` vs `remove()`

```java
poll()
```

returns:

```text
null
```

if the queue is empty.

```java
remove()
```

throws an exception if the queue is empty.

For typical DSA code:

```java
queue.poll();
```

is convenient.

---

# 5. `peek()` vs `element()`

```java
peek()
```

returns:

```text
null
```

if empty.

```java
element()
```

throws an exception if empty.

---

# 6. Basic Queue Example

```java
Queue<Integer> queue =
    new ArrayDeque<>();

queue.offer(10);
queue.offer(20);
queue.offer(30);

System.out.println(queue.peek());
// 10

System.out.println(queue.poll());
// 10

System.out.println(queue.poll());
// 20
```

The oldest element leaves first.

---

# 7. Why Use ArrayDeque?

For most single-threaded DSA problems:

```java
ArrayDeque
```

is an excellent choice.

It can act as:

```text
Queue
Stack
Deque
```

For example:

```java
Deque<Integer> deque =
    new ArrayDeque<>();
```

Then:

```java
deque.offerLast(10);
deque.pollFirst();
```

behaves like a queue.

---

# 8. What is a Deque?

Deque means:

```text
Double Ended Queue
```

It supports operations at both ends.

```text
front ← [10, 20, 30] → rear
```

Possible operations:

```text
addFirst
addLast
removeFirst
removeLast
peekFirst
peekLast
```

---

# 9. Deque in Java

```java
Deque<Integer> deque =
    new ArrayDeque<>();
```

Add to front:

```java
deque.offerFirst(10);
```

Add to rear:

```java
deque.offerLast(20);
```

Remove front:

```java
deque.pollFirst();
```

Remove rear:

```java
deque.pollLast();
```

Peek front:

```java
deque.peekFirst();
```

Peek rear:

```java
deque.peekLast();
```

---

# 10. Queue vs Deque

### Queue

```text
Insert → rear
Remove → front
```

### Deque

```text
Insert → front/rear
Remove → front/rear
```

A deque is more flexible.

---

# 11. Circular Queue

A circular queue connects the end of the queue back to the beginning.

Instead of shifting elements after every removal:

```text
front
  ↓
[ ][ ][ ][ ][ ]
```

indexes wrap around using:

```java
(index + 1) % capacity
```

This allows efficient reuse of empty positions.

---

# 12. Circular Queue Design

Maintain:

```text
front
rear
size
capacity
```

When advancing:

```java
front =
    (front + 1) % capacity;
```

and:

```java
rear =
    (rear + 1) % capacity;
```

Typical operations:

```text
enqueue → O(1)
dequeue → O(1)
```

---

# 13. Circular Queue — Java

```java
class MyCircularQueue {

    private final int[] data;
    private int front;
    private int size;

    public MyCircularQueue(
            int capacity) {

        data = new int[capacity];
    }

    public boolean enQueue(int value) {

        if (isFull()) {
            return false;
        }

        int rear =
            (front + size)
            % data.length;

        data[rear] = value;
        size++;

        return true;
    }

    public boolean deQueue() {

        if (isEmpty()) {
            return false;
        }

        front =
            (front + 1)
            % data.length;

        size--;

        return true;
    }

    public int Front() {

        if (isEmpty()) {
            return -1;
        }

        return data[front];
    }

    public int Rear() {

        if (isEmpty()) {
            return -1;
        }

        int rear =
            (front + size - 1)
            % data.length;

        return data[rear];
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public boolean isFull() {
        return size == data.length;
    }
}
```

---

# 14. BFS Using Queue

Breadth First Search uses a queue.

Example graph:

```text
      1
     / \
    2   3
   / \
  4   5
```

BFS order:

```text
1 → 2 → 3 → 4 → 5
```

---

# 15. BFS — Basic Java

```java
static void bfs(
        List<List<Integer>> graph,
        int start) {

    boolean[] visited =
        new boolean[graph.size()];

    Queue<Integer> queue =
        new ArrayDeque<>();

    queue.offer(start);
    visited[start] = true;

    while (!queue.isEmpty()) {

        int node =
            queue.poll();

        System.out.println(node);

        for (int next :
                graph.get(node)) {

            if (!visited[next]) {

                visited[next] = true;
                queue.offer(next);
            }
        }
    }
}
```

### Complexity

For an adjacency-list graph:

```text
Time:  O(V + E)
Space: O(V)
```

---

# 16. Why Mark Visited When Enqueuing?

Prefer:

```java
visited[next] = true;
queue.offer(next);
```

when discovering a node.

If you wait until dequeue time, the same node can potentially be added multiple times.

---

# 17. BFS Level Order

Queue naturally processes nodes level by level.

```java
while (!queue.isEmpty()) {

    int levelSize =
        queue.size();

    for (int i = 0;
         i < levelSize;
         i++) {

        int node =
            queue.poll();

        // Process current level.
    }
}
```

This pattern is extremely important for tree and graph interviews.

---

# 18. Binary Tree Level Order Traversal

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

        int size = queue.size();

        List<Integer> level =
            new ArrayList<>();

        for (int i = 0;
             i < size;
             i++) {

            TreeNode node =
                queue.poll();

            level.add(node.val);

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

# 19. Shortest Path in an Unweighted Graph

BFS finds the shortest number of edges from a source in an unweighted graph.

Why?

Because BFS explores:

```text
distance 0
↓
distance 1
↓
distance 2
↓
distance 3
```

in order.

Therefore the first time a node is reached is through a shortest path.

---

# 20. BFS Distance Array

```java
int[] distance =
    new int[graph.size()];

Arrays.fill(
    distance,
    -1
);

Queue<Integer> queue =
    new ArrayDeque<>();

distance[start] = 0;
queue.offer(start);

while (!queue.isEmpty()) {

    int node =
        queue.poll();

    for (int next :
            graph.get(node)) {

        if (distance[next] == -1) {

            distance[next] =
                distance[node] + 1;

            queue.offer(next);
        }
    }
}
```

---

# 21. Multi-Source BFS

Sometimes multiple starting points exist.

Examples:

```text
Rotting Oranges
Nearest Zero
Fire Spread
Multi-source shortest distance
```

Put all sources into the queue initially:

```java
for (int source : sources) {

    queue.offer(source);
    distance[source] = 0;
}
```

Then run normal BFS.

---

# 22. Rotting Oranges

Classic multi-source BFS.

Initial rotten oranges:

```text
distance = 0
```

are all added to the queue.

Each minute:

```text
current rotten orange
↓
infect neighboring fresh oranges
```

The BFS level represents time.

---

# 23. BFS on a Grid

Typical directions:

```java
int[][] directions = {
    {1, 0},
    {-1, 0},
    {0, 1},
    {0, -1}
};
```

For each cell:

```java
int newRow =
    row + direction[0];

int newCol =
    col + direction[1];
```

Check boundaries before accessing the matrix.

---

# 24. Queue for Task Scheduling

Queues naturally model:

```text
First task arrives
↓
First task processed
```

This is useful for:

```text
Job scheduling
Request processing
Message processing
Work queues
```

---

# 25. Producer-Consumer Model

A producer creates work:

```text
Producer
   ↓
Queue
   ↓
Consumer
```

The queue decouples production from consumption.

In Java applications, concurrency-aware queues such as:

```java
BlockingQueue
```

are commonly used when multiple threads are involved.

---

# 26. `BlockingQueue`

For concurrent producer-consumer scenarios:

```java
BlockingQueue<String> queue =
    new LinkedBlockingQueue<>();
```

Producer:

```java
queue.put(message);
```

Consumer:

```java
String message =
    queue.take();
```

`put()` can wait when the queue is full, and `take()` can wait when it is empty.

This is different from the basic DSA use of `ArrayDeque`.

---

# 27. Queue vs Stack

### Queue

```text
FIFO
```

Used for:

```text
BFS
Scheduling
Processing order
```

### Stack

```text
LIFO
```

Used for:

```text
DFS
Parentheses
Backtracking
Monotonic stack
```

---

# 28. Deque for Sliding Window Maximum

This is one of the most important deque problems.

Example:

```text
nums = [1,3,-1,-3,5,3,6,7]
k = 3
```

Answer:

```text
[3,3,5,5,6,7]
```

Use a **monotonic decreasing deque**.

---

# 29. Why Store Indexes?

Store indexes instead of only values because we need to know whether an element has left the window.

For window:

```text
[left ... right]
```

an index is outside if:

```java
index <= right - k
```

---

# 30. Sliding Window Maximum — Java

```java
static int[] maxSlidingWindow(
        int[] nums,
        int k) {

    if (nums.length == 0 || k == 0) {
        return new int[0];
    }

    int[] result =
        new int[nums.length - k + 1];

    Deque<Integer> deque =
        new ArrayDeque<>();

    int resultIndex = 0;

    for (int right = 0;
         right < nums.length;
         right++) {

        while (!deque.isEmpty()
                && deque.peekFirst()
                    <= right - k) {

            deque.pollFirst();
        }

        while (!deque.isEmpty()
                && nums[deque.peekLast()]
                    <= nums[right]) {

            deque.pollLast();
        }

        deque.offerLast(right);

        if (right >= k - 1) {

            result[resultIndex++] =
                nums[deque.peekFirst()];
        }
    }

    return result;
}
```

### Complexity

```text
Time:  O(n)
Space: O(k)
```

---

# 31. Monotonic Deque

A monotonic deque maintains candidates in a specific order.

For maximum:

```text
decreasing values
```

Example:

```text
9
7
4
2
```

The front is always the maximum candidate.

For minimum:

```text
increasing values
```

Example:

```text
2
4
7
9
```

The front is always the minimum candidate.

---

# 32. Sliding Window Minimum

Use an increasing deque.

For each new value:

```text
Remove expired indexes
Remove larger values from back
Add current index
Front = minimum
```

This is the mirror image of sliding window maximum.

---

# 33. Deque for 0-1 BFS

If graph edges have weights only:

```text
0 or 1
```

use:

```text
Deque
```

instead of a normal queue.

For an edge with weight `0`:

```java
deque.offerFirst(next);
```

For weight `1`:

```java
deque.offerLast(next);
```

This gives an efficient shortest-path algorithm for 0-1 weighted graphs.

---

# 34. 0-1 BFS Concept

Suppose:

```text
current → next
```

has weight:

```text
0
```

It should be processed as soon as possible.

Therefore:

```text
add to front
```

For weight:

```text
1
```

add to:

```text
back
```

This maintains processing order by distance.

---

# 35. Queue for Topological Sort

Kahn's Algorithm uses a queue.

For every node with:

```text
indegree == 0
```

add it to the queue.

Then:

```text
remove node
↓
process node
↓
decrease neighbors' indegree
↓
add newly-zero indegree nodes
```

---

# 36. Topological Sort — Java

```java
static List<Integer> topologicalSort(
        List<List<Integer>> graph,
        int[] indegree) {

    Queue<Integer> queue =
        new ArrayDeque<>();

    for (int i = 0;
         i < indegree.length;
         i++) {

        if (indegree[i] == 0) {
            queue.offer(i);
        }
    }

    List<Integer> result =
        new ArrayList<>();

    while (!queue.isEmpty()) {

        int node =
            queue.poll();

        result.add(node);

        for (int next :
                graph.get(node)) {

            indegree[next]--;

            if (indegree[next] == 0) {
                queue.offer(next);
            }
        }
    }

    if (result.size()
            != graph.size()) {

        throw new IllegalArgumentException(
            "Graph contains a cycle"
        );
    }

    return result;
}
```

---

# 37. Queue and PriorityQueue

Do not confuse:

```text
Queue
```

with:

```text
PriorityQueue
```

### Queue

Processing order:

```text
arrival order
```

### PriorityQueue

Processing order:

```text
priority
```

Example:

```java
PriorityQueue<Integer> pq =
    new PriorityQueue<>();
```

The smallest integer is processed first by default.

---

# 38. Deque vs PriorityQueue

Use a deque when the ordering depends on:

```text
front/rear
```

Use a priority queue when the ordering depends on:

```text
minimum/maximum priority
```

For sliding window maximum:

```text
Deque
```

is typically preferred because it gives:

```text
O(n)
```

instead of:

```text
O(n log k)
```

with a heap.

---

# 39. Queue Using Two Stacks

A classic design problem.

Use:

```text
inStack
outStack
```

```java
class MyQueue {

    private final Deque<Integer> in =
        new ArrayDeque<>();

    private final Deque<Integer> out =
        new ArrayDeque<>();

    public void push(int x) {
        in.push(x);
    }

    public int pop() {

        moveIfNeeded();

        return out.pop();
    }

    public int peek() {

        moveIfNeeded();

        return out.peek();
    }

    public boolean empty() {
        return in.isEmpty()
            && out.isEmpty();
    }

    private void moveIfNeeded() {

        if (out.isEmpty()) {

            while (!in.isEmpty()) {
                out.push(in.pop());
            }
        }
    }
}
```

Amortized complexity:

```text
push: O(1)
pop:  O(1) amortized
peek: O(1) amortized
```

---

# 40. Stack Using Two Queues

The reverse design problem.

One approach:

```text
push → O(n)
pop  → O(1)
```

The newly pushed element is moved to the front of the active queue.

Another approach:

```text
push → O(1)
pop  → O(n)
```

The important interview point is understanding the trade-off.

---

# 41. Deque for Palindrome

A deque can compare characters from both ends.

Example:

```text
racecar
```

Algorithm:

```text
removeFirst
removeLast
compare
```

until one or zero elements remain.

```java
Deque<Character> deque =
    new ArrayDeque<>();

for (char c : s.toCharArray()) {
    deque.offerLast(c);
}

while (deque.size() > 1) {

    if (!deque.pollFirst()
            .equals(deque.pollLast())) {
        return false;
    }
}
```

For production Java, simpler two-pointer code is often preferable for this specific problem.

---

# 42. BFS vs DFS

### BFS

Uses:

```text
Queue
```

Best for:

```text
Shortest path in unweighted graph
Level-order traversal
Minimum number of steps
Multi-source spread
```

### DFS

Uses:

```text
Stack / recursion
```

Best for:

```text
Exploration
Connected components
Cycle detection variants
Backtracking
```

---

# 43. BFS Level Template

```java
Queue<Integer> queue =
    new ArrayDeque<>();

queue.offer(start);

while (!queue.isEmpty()) {

    int levelSize =
        queue.size();

    for (int i = 0;
         i < levelSize;
         i++) {

        int node =
            queue.poll();

        // Process current level.

        // Add next-level nodes.
    }
}
```

Memorize this template.

---

# 44. Multi-Source BFS Template

```java
Queue<Integer> queue =
    new ArrayDeque<>();

for (int source : sources) {

    queue.offer(source);
    distance[source] = 0;
}

while (!queue.isEmpty()) {

    int node =
        queue.poll();

    for (int next :
            graph.get(node)) {

        if (distance[next] == -1) {

            distance[next] =
                distance[node] + 1;

            queue.offer(next);
        }
    }
}
```

---

# 45. Grid BFS Template

```java
Queue<int[]> queue =
    new ArrayDeque<>();

queue.offer(
    new int[]{startRow, startCol}
);

while (!queue.isEmpty()) {

    int[] cell =
        queue.poll();

    int row = cell[0];
    int col = cell[1];

    for (int[] direction :
            directions) {

        int newRow =
            row + direction[0];

        int newCol =
            col + direction[1];

        if (newRow < 0
                || newRow >= rows
                || newCol < 0
                || newCol >= cols) {
            continue;
        }

        // Process neighbor.
    }
}
```

---

# 46. Common Queue/Deque Mistakes

### Mistake 1 — Using a LinkedList unnecessarily

For typical single-threaded DSA:

```java
ArrayDeque
```

is usually a better choice.

---

### Mistake 2 — Removing from the wrong end

Queue:

```text
offerLast
pollFirst
```

Deque depends on the problem.

---

### Mistake 3 — Forgetting visited state in BFS

Without visited tracking, graphs can repeatedly process the same node.

---

### Mistake 4 — Marking visited too late

For standard BFS, mark when discovered/enqueued.

---

### Mistake 5 — Losing BFS levels

If the problem asks for:

```text
minimum number of steps
```

or:

```text
level
```

use:

```java
int size = queue.size();
```

before processing that level.

---

### Mistake 6 — Wrong deque monotonic direction

Maximum:

```text
decreasing
```

Minimum:

```text
increasing
```

---

### Mistake 7 — Storing values instead of indexes

For sliding windows, indexes are usually required to remove expired elements.

---

# 47. Edge Cases

Always test:

```text
Empty queue
One element
Queue becomes empty
Full circular queue
k = 1
k = n
Duplicate values
All values equal
Increasing values
Decreasing values
Disconnected graph
Cycles in graph
Grid with one cell
No path
Multiple BFS sources
```

---

# 48. Interview Questions — Easy

1. Implement a queue.
2. Implement a deque.
3. Implement circular queue.
4. Reverse a queue.
5. Generate binary numbers using a queue.
6. Implement stack using queues.
7. Implement queue using stacks.
8. Palindrome using deque.
9. Basic BFS.
10. Binary tree level-order traversal.

---

# 49. Interview Questions — Medium

11. Rotting Oranges.
12. Number of Islands using BFS.
13. Shortest Path in an Unweighted Graph.
14. Minimum Depth of Binary Tree.
15. Open the Lock.
16. Sliding Window Maximum.
17. Sliding Window Minimum.
18. Walls and Gates.
19. Course Schedule using Kahn's Algorithm.
20. 01 Matrix.
21. Word Ladder.
22. Multi-source BFS.
23. Circular Queue Design.

---

# 50. Interview Questions — Advanced

24. 0-1 BFS.
25. Shortest Path in Binary Matrix.
26. Minimum Cost Grid problems.
27. Advanced multi-source BFS.
28. Design concurrent producer-consumer queues.
29. Monotonic deque optimization.
30. Sliding window optimization with deque.
31. Advanced topological sorting.
32. Shortest path with state tracking.

---

# 51. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Queue Operations | ArrayDeque | O(1) | O(n) |
| Deque Operations | ArrayDeque | O(1) | O(n) |
| BFS | Queue | O(V + E) | O(V) |
| Level Order | Queue | O(n) | O(n) |
| Multi-source BFS | Queue | O(V + E) | O(V) |
| Grid BFS | Queue | O(rows × cols) | O(rows × cols) |
| Sliding Maximum | Monotonic Deque | O(n) | O(k) |
| Sliding Minimum | Monotonic Deque | O(n) | O(k) |
| 0-1 BFS | Deque | O(V + E) | O(V) |
| Topological Sort | Queue | O(V + E) | O(V) |
| Circular Queue | Array | O(1) | O(k) |

---

# 52. Quick Revision

```text
Queue & Deque
│
├── Queue
│   ├── FIFO
│   ├── BFS
│   ├── Scheduling
│   └── Producer-Consumer
│
├── Deque
│   ├── Both Ends
│   ├── Sliding Window
│   ├── Palindrome
│   └── 0-1 BFS
│
├── BFS
│   ├── Graph
│   ├── Tree Levels
│   ├── Shortest Unweighted Path
│   ├── Grid
│   └── Multi-Source BFS
│
├── Monotonic Deque
│   ├── Window Maximum
│   └── Window Minimum
│
└── Queue Algorithms
    ├── Circular Queue
    ├── Kahn's Algorithm
    └── Queue Using Stacks
```

---

## Interview Rule

> **Think Queue when processing must happen in arrival order or level by level. Think Deque when you need efficient access to both ends. For sliding-window maximum/minimum, a monotonic deque is one of the most important O(n) patterns to know.**
