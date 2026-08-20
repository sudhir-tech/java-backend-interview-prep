# DSA — Disjoint Set Union (Union-Find)

Disjoint Set Union, commonly called **DSU** or **Union-Find**, is a data structure used to efficiently maintain a collection of disjoint sets.

It is especially useful for:

- Connected components
- Dynamic connectivity
- Cycle detection in undirected graphs
- Kruskal's Minimum Spanning Tree
- Network connectivity
- Grouping related elements
- Redundant connections
- Account merging
- Grid connectivity problems

The two core operations are:

```text
find
union
```

With:

```text
Path Compression
+
Union by Rank / Size
```

both operations are effectively:

```text
O(α(n))
```

amortized, where `α(n)` is the inverse Ackermann function and grows extremely slowly.

---

# 1. What is DSU?

Suppose we have:

```text
1  2  3  4  5
```

Initially, every element belongs to its own set:

```text
{1}
{2}
{3}
{4}
{5}
```

If we connect:

```text
1 with 2
```

we get:

```text
{1,2}
{3}
{4}
{5}
```

Then:

```text
2 with 3
```

gives:

```text
{1,2,3}
{4}
{5}
```

DSU efficiently answers:

```text
Are 1 and 3 connected?
```

and supports:

```text
Merge the set containing 1
with the set containing 3.
```

---

# 2. Core Operations

### Find

Determine the representative/root of an element's set.

```java
find(x)
```

### Union

Merge the sets containing two elements.

```java
union(a, b)
```

---

# 3. Parent Array

A basic DSU uses:

```java
int[] parent;
```

Initially:

```text
parent[i] = i
```

because every element is its own parent.

Example:

```text
parent:

0 → 0
1 → 1
2 → 2
3 → 3
4 → 4
```

---

# 4. Basic Find

Without optimizations:

```java
int find(int x) {

    while (x != parent[x]) {
        x = parent[x];
    }

    return x;
}
```

The root satisfies:

```java
parent[root] == root
```

---

# 5. Basic Union

```java
void union(
        int a,
        int b) {

    int rootA = find(a);
    int rootB = find(b);

    if (rootA == rootB) {
        return;
    }

    parent[rootB] = rootA;
}
```

This merges the two sets.

---

# 6. Why Basic DSU Can Become Slow

Suppose we repeatedly connect:

```text
1 → 2
2 → 3
3 → 4
4 → 5
```

A poor union strategy can create:

```text
1
↓
2
↓
3
↓
4
↓
5
```

Now:

```text
find(1)
```

requires walking through many nodes.

For large inputs, this can become inefficient.

---

# 7. Path Compression

Path compression makes `find()` faster.

When finding the root:

```text
x
↓
parent[x]
↓
parent[parent[x]]
↓
root
```

we make every visited node point directly to the root.

---

# 8. Path Compression — Java

```java
int find(int x) {

    if (parent[x] != x) {

        parent[x] =
            find(parent[x]);
    }

    return parent[x];
}
```

The important line is:

```java
parent[x] = find(parent[x]);
```

After the operation:

```text
x → root
```

---

# 9. Before and After Path Compression

Before:

```text
1
↓
2
↓
3
↓
4
```

After:

```text
1 ─┐
2 ─┤
3 ─┤→ 4
4 ─┘
```

Future `find()` operations become much faster.

---

# 10. Union by Rank

Path compression alone is powerful, but we can also keep trees shallow using:

```text
rank
```

Rank roughly represents tree height.

Maintain:

```java
int[] rank;
```

Initially:

```text
rank[i] = 0
```

When merging:

```text
smaller rank
→
larger rank
```

If ranks are equal:

```text
choose one root
increase its rank
```

---

# 11. Union by Rank — Java

```java
void union(
        int a,
        int b) {

    int rootA = find(a);
    int rootB = find(b);

    if (rootA == rootB) {
        return;
    }

    if (rank[rootA]
            < rank[rootB]) {

        parent[rootA] = rootB;

    } else if (rank[rootA]
            > rank[rootB]) {

        parent[rootB] = rootA;

    } else {

        parent[rootB] = rootA;
        rank[rootA]++;
    }
}
```

---

# 12. Union by Size

Instead of rank, store:

```java
int[] size;
```

Initially:

```text
size[i] = 1
```

Always attach the smaller set under the larger set.

---

# 13. Union by Size — Java

```java
void union(
        int a,
        int b) {

    int rootA = find(a);
    int rootB = find(b);

    if (rootA == rootB) {
        return;
    }

    if (size[rootA]
            < size[rootB]) {

        parent[rootA] = rootB;
        size[rootB] += size[rootA];

    } else {

        parent[rootB] = rootA;
        size[rootA] += size[rootB];
    }
}
```

Union by size is often easier to reason about than rank.

---

# 14. Complete DSU — Union by Size

```java
class DSU {

    private final int[] parent;
    private final int[] size;

    public DSU(int n) {

        parent =
            new int[n];

        size =
            new int[n];

        for (int i = 0;
             i < n;
             i++) {

            parent[i] = i;
            size[i] = 1;
        }
    }

    public int find(int x) {

        if (parent[x] != x) {

            parent[x] =
                find(parent[x]);
        }

        return parent[x];
    }

    public boolean union(
            int a,
            int b) {

        int rootA = find(a);
        int rootB = find(b);

        if (rootA == rootB) {
            return false;
        }

        if (size[rootA]
                < size[rootB]) {

            parent[rootA] = rootB;
            size[rootB] +=
                size[rootA];

        } else {

            parent[rootB] = rootA;
            size[rootA] +=
                size[rootB];
        }

        return true;
    }
}
```

Returning `false` when both elements are already connected is useful for cycle detection.

---

# 15. Check If Two Nodes Are Connected

```java
if (dsu.find(a)
        == dsu.find(b)) {

    // Same component
}
```

This answers:

```text
Are a and b in the same set?
```

---

# 16. DSU Complexity

With:

```text
Path Compression
+
Union by Rank
```

or:

```text
Path Compression
+
Union by Size
```

the amortized complexity is:

```text
find  → O(α(n))
union → O(α(n))
```

For practical input sizes:

```text
α(n) is effectively a very small constant.
```

---

# 17. Count Connected Components

Suppose:

```text
n = 5
```

and edges:

```text
0 - 1
1 - 2
3 - 4
```

Components:

```text
{0,1,2}
{3,4}
```

Answer:

```text
2
```

Start with:

```text
components = n
```

Every successful union reduces the component count by one.

---

# 18. Count Components — Java

```java
static int countComponents(
        int n,
        int[][] edges) {

    DSU dsu =
        new DSU(n);

    int components = n;

    for (int[] edge : edges) {

        if (dsu.union(
                edge[0],
                edge[1])) {

            components--;
        }
    }

    return components;
}
```

---

# 19. Why Successful Union Matters

Suppose:

```text
1 and 2
```

are already connected.

Calling:

```java
union(1, 2)
```

does not create a new connection between components.

Therefore:

```text
components
```

should not decrease.

This is why `union()` returning a boolean is useful.

---

# 20. Cycle Detection in an Undirected Graph

For every edge:

```text
u — v
```

check:

```java
if (find(u) == find(v))
```

If they already belong to the same component:

```text
adding this edge creates a cycle
```

Otherwise:

```java
union(u, v);
```

---

# 21. Cycle Detection — Java

```java
static boolean hasCycle(
        int n,
        int[][] edges) {

    DSU dsu =
        new DSU(n);

    for (int[] edge : edges) {

        int u = edge[0];
        int v = edge[1];

        if (dsu.find(u)
                == dsu.find(v)) {

            return true;
        }

        dsu.union(u, v);
    }

    return false;
}
```

This approach is specifically for:

```text
undirected graphs
```

---

# 22. Redundant Connection

Given edges forming a tree plus one extra edge, find the edge that creates a cycle.

Example:

```text
1 - 2
2 - 3
3 - 1
```

The redundant edge is:

```text
3 - 1
```

Algorithm:

```text
For each edge:
    if endpoints already connected:
        return edge
    union endpoints
```

---

# 23. Redundant Connection — Java

```java
static int[] findRedundantConnection(
        int[][] edges) {

    DSU dsu =
        new DSU(edges.length + 1);

    for (int[] edge : edges) {

        if (!dsu.union(
                edge[0],
                edge[1])) {

            return edge;
        }
    }

    return new int[0];
}
```

---

# 24. Kruskal's Algorithm

DSU is one of the key data structures behind:

```text
Kruskal's Minimum Spanning Tree
```

Steps:

```text
1. Sort edges by weight.
2. Start with every vertex separate.
3. Process edges from smallest weight.
4. If endpoints are in different components:
       add edge
       union components
5. Stop after V - 1 edges.
```

---

# 25. Kruskal Example

Edges:

```text
A-B = 1
B-C = 2
A-C = 3
C-D = 4
```

Process:

```text
A-B → take
B-C → take
A-C → skip because cycle
C-D → take
```

MST:

```text
A-B
B-C
C-D
```

Total:

```text
1 + 2 + 4 = 7
```

---

# 26. Kruskal — Java

```java
static int kruskal(
        int n,
        int[][] edges) {

    Arrays.sort(
        edges,
        Comparator.comparingInt(
            edge -> edge[2]
        )
    );

    DSU dsu =
        new DSU(n);

    int totalWeight = 0;
    int edgesUsed = 0;

    for (int[] edge : edges) {

        int u = edge[0];
        int v = edge[1];
        int weight = edge[2];

        if (dsu.union(u, v)) {

            totalWeight += weight;
            edgesUsed++;

            if (edgesUsed == n - 1) {
                break;
            }
        }
    }

    return totalWeight;
}
```

---

# 27. Kruskal Complexity

Sorting dominates:

```text
O(E log E)
```

DSU operations are approximately:

```text
O(E α(V))
```

Overall:

```text
O(E log E)
```

---

# 28. DSU vs DFS/BFS for Connectivity

Both can solve connectivity problems, but they work differently.

### DFS/BFS

Good when:

```text
You need to traverse the graph.
```

### DSU

Excellent when:

```text
Edges are added dynamically
and you repeatedly ask whether two nodes are connected.
```

DSU avoids rebuilding a full traversal for every connectivity query.

---

# 29. Dynamic Connectivity

Suppose operations are:

```text
connect(1,2)
connect(2,3)
isConnected(1,3)
connect(4,5)
isConnected(1,5)
```

DSU handles these efficiently.

```java
dsu.union(1, 2);
dsu.union(2, 3);

boolean connected =
    dsu.find(1)
        == dsu.find(3);
```

---

# 30. Number of Islands — DSU Approach

An island problem can be solved using:

```text
DFS/BFS
```

or:

```text
DSU
```

For DSU:

```text
Each land cell = node
```

Union neighboring land cells.

The number of connected components among land cells is the number of islands.

---

# 31. Mapping a Grid Cell to DSU Index

For:

```text
rows × cols
```

map:

```text
(row, col)
```

to:

```java
int id =
    row * cols + col;
```

Example:

```text
row = 2
col = 3
cols = 5

id = 2 * 5 + 3
   = 13
```

---

# 32. Dynamic Number of Islands

If land cells are added one at a time:

```text
addLand(row, col)
```

DSU is particularly useful.

For each newly added land:

```text
components++
```

Then union with neighboring land cells.

Each successful union:

```text
components--
```

This produces the island count after every addition.

---

# 33. Accounts Merge

Suppose accounts share email addresses.

If:

```text
Account A
email: x@example.com
```

and:

```text
Account B
email: x@example.com
```

they belong to the same person.

DSU can union:

```text
emails belonging to the same account
```

Then group emails by their root.

---

# 34. DSU with Strings

DSU normally works with integer indexes.

For strings:

```text
Map<String, Integer>
```

can assign an integer ID.

Example:

```java
Map<String, Integer> ids =
    new HashMap<>();

int id =
    ids.computeIfAbsent(
        email,
        key -> ids.size()
    );
```

Then DSU operates on the integer IDs.

---

# 35. DSU with Generic Objects

Conceptually, DSU can be built around objects, but for interviews:

```text
Map<T, T> parent
```

is possible.

However, integer-indexed arrays are usually:

```text
simpler
+
faster
+
more memory efficient
```

when the universe of elements is known.

---

# 36. DSU Component Size

With union by size:

```java
size[root]
```

contains the number of elements in the component.

To get the size containing `x`:

```java
int root =
    dsu.find(x);

int componentSize =
    dsu.getSize(root);
```

---

# 37. DSU with Component Count

A useful production/interview implementation can maintain:

```java
int components;
```

Initialize:

```text
components = n
```

Every successful union:

```text
components--
```

Then:

```java
getComponentCount()
```

is:

```text
O(1)
```

---

# 38. Complete DSU with Component Count

```java
class DSU {

    private final int[] parent;
    private final int[] size;
    private int components;

    public DSU(int n) {

        parent =
            new int[n];

        size =
            new int[n];

        components = n;

        for (int i = 0;
             i < n;
             i++) {

            parent[i] = i;
            size[i] = 1;
        }
    }

    public int find(int x) {

        if (parent[x] != x) {

            parent[x] =
                find(parent[x]);
        }

        return parent[x];
    }

    public boolean union(
            int a,
            int b) {

        int rootA = find(a);
        int rootB = find(b);

        if (rootA == rootB) {
            return false;
        }

        if (size[rootA]
                < size[rootB]) {

            parent[rootA] = rootB;
            size[rootB] +=
                size[rootA];

        } else {

            parent[rootB] = rootA;
            size[rootA] +=
                size[rootB];
        }

        components--;

        return true;
    }

    public int getComponentCount() {
        return components;
    }

    public int getComponentSize(int x) {

        return size[find(x)];
    }
}
```

---

# 39. Iterative Find

Recursive path compression is simple, but an iterative version can also be used.

First find the root:

```java
int root = x;

while (root != parent[root]) {
    root = parent[root];
}
```

Then compress the path.

For normal interview problems, the recursive version is usually easier to explain.

---

# 40. DSU Initialization

Always initialize:

```java
for (int i = 0;
     i < n;
     i++) {

    parent[i] = i;
    size[i] = 1;
}
```

Each node begins as its own component.

---

# 41. DSU Indexing

Be careful about whether nodes are:

```text
0-based
```

or:

```text
1-based
```

For nodes:

```text
1...n
```

it is common to allocate:

```java
new DSU(n + 1)
```

For:

```text
0...n-1
```

use:

```java
new DSU(n)
```

---

# 42. Union Return Value

A useful design:

```java
boolean union(a, b)
```

returns:

```text
true
```

if two different components were merged.

Returns:

```text
false
```

if they were already connected.

This makes these problems easy:

```text
Cycle detection
Component counting
Kruskal
Redundant connection
```

---

# 43. DSU and Directed Graphs

Basic DSU cycle detection is naturally suited to:

```text
undirected graphs
```

For directed graphs, cycle detection usually requires:

```text
DFS recursion stack
```

or:

```text
Kahn's topological sort
```

Do not blindly apply DSU to directed cycle problems.

---

# 44. DSU and Minimum Spanning Tree

DSU is not itself an MST algorithm.

It is a supporting data structure used by:

```text
Kruskal's algorithm
```

The overall process is:

```text
Sort edges
+
DSU connectivity checks
```

---

# 45. Kruskal vs Prim

Both find an MST.

### Kruskal

Uses:

```text
Sorting
+
DSU
```

Often convenient when edges are given explicitly.

### Prim

Uses:

```text
PriorityQueue
```

and grows the MST from a starting vertex.

Understanding both is useful for graph interviews.

---

# 46. Common DSU Mistakes

### Mistake 1 — Unioning nodes directly

Wrong:

```java
parent[b] = a;
```

without first finding roots.

Correct:

```java
int rootA = find(a);
int rootB = find(b);
```

then union the roots.

---

### Mistake 2 — Forgetting path compression

A basic DSU can become unnecessarily slow.

---

### Mistake 3 — Forgetting union by size/rank

Poorly shaped trees can increase `find()` cost.

---

### Mistake 4 — Decreasing component count for every edge

Only successful unions reduce the number of components.

---

### Mistake 5 — Using DSU for directed cycle detection

Basic DSU is intended for undirected connectivity.

---

### Mistake 6 — Off-by-one indexing

Check whether nodes start from:

```text
0
```

or:

```text
1
```

---

# 47. Edge Cases

Always test:

```text
n = 0
n = 1
No edges
Duplicate edges
Self-loop
Already connected nodes
Disconnected graph
One giant component
Multiple components
1-based node labels
0-based node labels
```

---

# 48. Interview Questions — Easy

1. Implement DSU.
2. Implement `find`.
3. Implement `union`.
4. Check if two nodes are connected.
5. Count connected components.
6. Detect a cycle in an undirected graph.

---

# 49. Interview Questions — Medium

7. Redundant Connection.
8. Number of Provinces.
9. Number of Islands using DSU.
10. Accounts Merge.
11. Dynamic Number of Islands.
12. Graph connectivity queries.
13. Largest component size.
14. Similar String Groups.

---

# 50. Interview Questions — Advanced

15. Kruskal's MST.
16. Minimum Cost to Connect All Points.
17. Offline dynamic connectivity.
18. DSU on grids.
19. DSU with component metadata.
20. DSU with rollback.
21. Weighted Union-Find.
22. Bipartite / parity DSU.
23. Advanced connectivity queries.

---

# 51. DSU with Rollback

A more advanced variation supports:

```text
union
snapshot
rollback
```

It is useful in:

```text
Offline dynamic connectivity
Divide-and-conquer over time
Competitive programming
```

The implementation is more complex because normal path compression can interfere with rollback.

For standard backend interviews, understand the concept rather than memorizing the implementation.

---

# 52. Weighted Union-Find

A weighted DSU stores relationships between nodes in addition to connectivity.

Examples:

```text
a / b = ratio
relative positions
distance differences
equivalence relationships
```

This is an advanced extension of the basic DSU concept.

---

# 53. Parity DSU

A DSU can store parity information:

```text
same group
different group
```

This can help solve:

```text
Bipartite constraints
Odd/even relationships
XOR constraints
```

This is an advanced interview topic.

---

# 54. DSU Mental Model

Think of each component as a tree:

```text
       root
      /    \
     A      B
    /
   C
```

Every node eventually points toward:

```text
root
```

`find(x)`:

```text
Find the root.
Compress the path.
```

`union(a,b)`:

```text
Find both roots.
Attach one root under the other.
```

---

# 55. DSU Pattern

The standard pattern is:

```text
Initialize
↓
For every connection:
    rootA = find(a)
    rootB = find(b)

    if rootA != rootB:
        union roots
```

This should become automatic when you see:

```text
dynamic connectivity
```

or:

```text
merge groups
```

---

# 56. DSU vs HashSet of Components

You could theoretically maintain groups manually:

```text
Set<Set<Integer>>
```

but merging sets can be expensive.

DSU avoids physically moving all elements during every merge.

It stores a compact parent relationship and efficiently finds component representatives.

---

# 57. DSU vs Adjacency List

Adjacency lists are better when you need to:

```text
Traverse neighbors
Run DFS/BFS
Find paths
Explore graph structure
```

DSU is better when you mainly need:

```text
Connectivity
Merging
Component membership
```

---

# 58. Practical Backend Connection

DSU is less common in everyday CRUD backend development than:

```text
HashMap
Queue
Tree
Graph traversal
```

but it demonstrates an important algorithmic concept:

```text
efficient dynamic grouping
```

It can appear in systems involving:

```text
network clusters
entity merging
relationship grouping
connectivity analysis
```

---

# 59. Complexity Summary

| Operation / Algorithm | Complexity |
|---|---:|
| Initialize DSU | O(n) |
| Find with path compression | O(α(n)) amortized |
| Union by size/rank | O(α(n)) amortized |
| Count components | O(1) if maintained |
| Undirected cycle detection | O(E α(V)) |
| Kruskal MST | O(E log E) |
| Dynamic connectivity | O(α(n)) per operation amortized |
| Grid connectivity | Depends on number of cells/edges |

---

# 60. Quick Revision

```text
Disjoint Set Union
│
├── Core
│   ├── parent[]
│   ├── find()
│   └── union()
│
├── Optimizations
│   ├── Path Compression
│   ├── Union by Rank
│   └── Union by Size
│
├── Connectivity
│   ├── Connected Components
│   ├── Dynamic Connectivity
│   └── Cycle Detection
│
├── Graph Algorithms
│   ├── Kruskal
│   └── Redundant Connection
│
├── Grid Problems
│   ├── Number of Islands
│   └── Dynamic Islands
│
└── Advanced
    ├── Rollback DSU
    ├── Weighted DSU
    └── Parity DSU
```

---

## Most Important Code to Memorize

```java
class DSU {

    int[] parent;
    int[] size;

    DSU(int n) {

        parent =
            new int[n];

        size =
            new int[n];

        for (int i = 0;
             i < n;
             i++) {

            parent[i] = i;
            size[i] = 1;
        }
    }

    int find(int x) {

        if (parent[x] != x) {

            parent[x] =
                find(parent[x]);
        }

        return parent[x];
    }

    boolean union(
            int a,
            int b) {

        int rootA = find(a);
        int rootB = find(b);

        if (rootA == rootB) {
            return false;
        }

        if (size[rootA]
                < size[rootB]) {

            parent[rootA] = rootB;
            size[rootB] +=
                size[rootA];

        } else {

            parent[rootB] = rootA;
            size[rootA] +=
                size[rootB];
        }

        return true;
    }
}
```

---

## Interview Rule

> **When you see repeated operations that merge groups and ask whether two elements belong to the same group, think DSU. The two optimizations to remember are path compression and union by size/rank.**
