# DSA — Advanced Graph Algorithms

Graphs are one of the most important DSA topics for backend interviews.

A graph contains:

```text
Vertices / Nodes
+
Edges
```

This file focuses on advanced graph patterns beyond basic BFS/DFS.

Topics covered:

- Dijkstra's Algorithm
- Bellman-Ford
- Floyd-Warshall
- Minimum Spanning Tree
- Prim's Algorithm
- Kruskal's Algorithm
- Topological Sort
- Kahn's Algorithm
- Strongly Connected Components
- Kosaraju's Algorithm
- Tarjan's Algorithm
- Bridges
- Articulation Points
- Bipartite Graphs
- 0-1 BFS
- Shortest paths
- Negative weights
- Graph complexity
- Interview patterns

---

# 1. Graph Terminology

A graph can be:

```text
Directed
Undirected
Weighted
Unweighted
Cyclic
Acyclic
Connected
Disconnected
```

Example:

```text
A ---- B
|      |
|      |
C ---- D
```

Vertices:

```text
A, B, C, D
```

Edges:

```text
A-B
A-C
B-D
C-D
```

---

# 2. Directed Graph

Edges have direction.

```text
A → B
```

does not imply:

```text
B → A
```

Example:

```text
A → B → C
```

Common applications:

```text
Dependencies
Course prerequisites
Build systems
Workflow graphs
Directed networks
```

---

# 3. Weighted Graph

Each edge has a cost.

```text
A --5-- B
```

The edge weight is:

```text
5
```

Weighted graphs are common in:

```text
Routing
Transportation
Network optimization
Minimum cost problems
```

---

# 4. Graph Representation

## Adjacency List

```java
List<List<Integer>> graph =
    new ArrayList<>();
```

For weighted graphs:

```java
List<List<int[]>> graph =
    new ArrayList<>();
```

where:

```text
edge[0] = destination
edge[1] = weight
```

---

# 5. Adjacency Matrix

```java
int[][] graph =
    new int[n][n];
```

Access:

```text
graph[u][v]
```

Advantages:

```text
O(1) edge lookup
```

Disadvantages:

```text
O(V²) memory
```

Use matrices when the graph is dense or the problem specifically needs all-pairs relationships.

---

# 6. Dijkstra's Algorithm

Dijkstra finds shortest paths from one source in a graph with:

```text
non-negative edge weights
```

Core idea:

```text
Always process the currently known closest node.
```

It uses:

```text
PriorityQueue
```

---

# 7. Dijkstra Example

Graph:

```text
A --4-- B
|       |
2       1
|       |
C --3-- D
```

Starting from:

```text
A
```

Dijkstra repeatedly chooses the smallest known distance.

---

# 8. Dijkstra — Java

```java
static int[] dijkstra(
        List<List<int[]>> graph,
        int source) {

    int n = graph.size();

    int[] distance =
        new int[n];

    Arrays.fill(
        distance,
        Integer.MAX_VALUE
    );

    PriorityQueue<int[]> pq =
        new PriorityQueue<>(
            Comparator.comparingInt(
                a -> a[1]
            )
        );

    distance[source] = 0;

    pq.offer(
        new int[]{source, 0}
    );

    while (!pq.isEmpty()) {

        int[] current =
            pq.poll();

        int node = current[0];
        int currentDistance =
            current[1];

        if (currentDistance
                != distance[node]) {
            continue;
        }

        for (int[] edge :
                graph.get(node)) {

            int next = edge[0];
            int weight = edge[1];

            if (distance[node]
                    != Integer.MAX_VALUE
                && distance[node]
                    + weight
                    < distance[next]) {

                distance[next] =
                    distance[node]
                    + weight;

                pq.offer(
                    new int[]{
                        next,
                        distance[next]
                    }
                );
            }
        }
    }

    return distance;
}
```

---

# 9. Dijkstra Complexity

Using an adjacency list and binary heap:

```text
Time: O((V + E) log V)
```

Usually simplified to:

```text
O(E log V)
```

Space:

```text
O(V + E)
```

---

# 10. Why Dijkstra Does Not Support Negative Edges

Dijkstra assumes that once the smallest-distance node is selected, its distance will not later become smaller.

A negative edge can violate this assumption.

Therefore:

```text
Dijkstra → no negative edge weights
```

For negative edges:

```text
Bellman-Ford
```

may be appropriate.

---

# 11. Stale PriorityQueue Entries

Java's `PriorityQueue` does not provide a simple decrease-key operation.

Therefore, Dijkstra often inserts a new pair:

```text
(node, newDistance)
```

instead of updating the old entry.

Then skip stale entries:

```java
if (currentDistance
        != distance[node]) {
    continue;
}
```

This is an important Java implementation detail.

---

# 12. Path Reconstruction with Dijkstra

To reconstruct the actual shortest path, maintain:

```java
int[] parent;
```

When relaxing:

```java
parent[next] = node;
```

After reaching the target, walk:

```text
target
↓
parent[target]
↓
parent[parent[target]]
↓
source
```

Then reverse the path.

---

# 13. Dijkstra with Parent

```java
int[] parent =
    new int[n];

Arrays.fill(
    parent,
    -1
);
```

When improving:

```java
distance[next] =
    distance[node] + weight;

parent[next] = node;
```

---

# 14. Bellman-Ford

Bellman-Ford finds shortest paths from one source and supports:

```text
negative edge weights
```

It can also detect:

```text
negative weight cycles
```

---

# 15. Bellman-Ford Idea

Relax every edge repeatedly.

For:

```text
V vertices
```

perform up to:

```text
V - 1
```

full relaxation rounds.

Why?

A shortest simple path can contain at most:

```text
V - 1 edges
```

if there is no negative cycle.

---

# 16. Bellman-Ford — Java

```java
static int[] bellmanFord(
        int n,
        int[][] edges,
        int source) {

    int[] distance =
        new int[n];

    Arrays.fill(
        distance,
        Integer.MAX_VALUE
    );

    distance[source] = 0;

    for (int i = 1;
         i <= n - 1;
         i++) {

        boolean changed = false;

        for (int[] edge : edges) {

            int u = edge[0];
            int v = edge[1];
            int weight = edge[2];

            if (distance[u]
                    == Integer.MAX_VALUE) {
                continue;
            }

            if (distance[u] + weight
                    < distance[v]) {

                distance[v] =
                    distance[u] + weight;

                changed = true;
            }
        }

        if (!changed) {
            break;
        }
    }

    return distance;
}
```

---

# 17. Detect Negative Cycle

After `V - 1` rounds, perform one additional pass.

If any distance can still be improved:

```text
negative cycle exists
```

Pattern:

```java
for (int[] edge : edges) {

    int u = edge[0];
    int v = edge[1];
    int weight = edge[2];

    if (distance[u]
            != Integer.MAX_VALUE
        && distance[u] + weight
            < distance[v]) {

        // Negative cycle.
    }
}
```

---

# 18. Bellman-Ford Complexity

```text
Time: O(VE)
Space: O(V)
```

Compared with Dijkstra:

```text
Dijkstra:
O(E log V)

Bellman-Ford:
O(VE)
```

Bellman-Ford is slower but handles negative edges.

---

# 19. Floyd-Warshall

Floyd-Warshall computes:

```text
shortest paths between every pair of vertices
```

It is an:

```text
all-pairs shortest path
```

algorithm.

---

# 20. Floyd-Warshall Idea

Let:

```text
dist[i][j]
```

be the shortest known distance from:

```text
i → j
```

For every intermediate vertex `k`:

```text
dist[i][j]
=
min(
    dist[i][j],
    dist[i][k] + dist[k][j]
)
```

---

# 21. Floyd-Warshall — Java

```java
static void floydWarshall(
        long[][] dist) {

    int n = dist.length;

    for (int k = 0;
         k < n;
         k++) {

        for (int i = 0;
             i < n;
             i++) {

            if (dist[i][k]
                    == Long.MAX_VALUE) {
                continue;
            }

            for (int j = 0;
                 j < n;
                 j++) {

                if (dist[k][j]
                        == Long.MAX_VALUE) {
                    continue;
                }

                long throughK =
                    dist[i][k]
                    + dist[k][j];

                if (throughK
                        < dist[i][j]) {

                    dist[i][j] =
                        throughK;
                }
            }
        }
    }
}
```

---

# 22. Floyd-Warshall Complexity

```text
Time: O(V³)
Space: O(V²)
```

Use it when:

```text
V is relatively small
```

and:

```text
all-pairs shortest paths
```

are required.

---

# 23. Floyd-Warshall and Negative Cycles

After the algorithm:

```java
dist[i][i] < 0
```

for some `i` indicates a negative cycle reachable from that vertex in the standard all-pairs setup.

---

# 24. Minimum Spanning Tree

A Minimum Spanning Tree (MST):

```text
connects all vertices
```

with:

```text
minimum total edge weight
```

and contains exactly:

```text
V - 1 edges
```

for a connected undirected graph.

Main algorithms:

```text
Kruskal
Prim
```

---

# 25. Kruskal's Algorithm

Kruskal:

```text
Sort all edges by weight
↓
Take the smallest edge
↓
If it does not create a cycle:
    take it
↓
Continue
```

Use:

```text
DSU
```

to detect whether endpoints are already connected.

---

# 26. Kruskal — Java

```java
static int kruskal(
        int n,
        int[][] edges) {

    Arrays.sort(
        edges,
        Comparator.comparingInt(
            e -> e[2]
        )
    );

    DSU dsu =
        new DSU(n);

    int total = 0;
    int used = 0;

    for (int[] edge : edges) {

        int u = edge[0];
        int v = edge[1];
        int weight = edge[2];

        if (dsu.union(u, v)) {

            total += weight;
            used++;

            if (used == n - 1) {
                break;
            }
        }
    }

    return total;
}
```

---

# 27. Kruskal Complexity

Sorting:

```text
O(E log E)
```

DSU:

```text
O(E α(V))
```

Overall:

```text
O(E log E)
```

---

# 28. Prim's Algorithm

Prim builds the MST by growing from a starting vertex.

Steps:

```text
Start with one vertex
↓
Choose the cheapest edge leaving the current tree
↓
Add the new vertex
↓
Repeat
```

It commonly uses:

```text
PriorityQueue
```

---

# 29. Prim — Java

```java
static int prim(
        List<List<int[]>> graph) {

    int n = graph.size();

    boolean[] visited =
        new boolean[n];

    PriorityQueue<int[]> pq =
        new PriorityQueue<>(
            Comparator.comparingInt(
                e -> e[1]
            )
        );

    pq.offer(
        new int[]{0, 0}
    );

    int total = 0;
    int count = 0;

    while (!pq.isEmpty()
            && count < n) {

        int[] current =
            pq.poll();

        int node = current[0];
        int weight = current[1];

        if (visited[node]) {
            continue;
        }

        visited[node] = true;
        total += weight;
        count++;

        for (int[] edge :
                graph.get(node)) {

            int next = edge[0];
            int edgeWeight = edge[1];

            if (!visited[next]) {

                pq.offer(
                    new int[]{
                        next,
                        edgeWeight
                    }
                );
            }
        }
    }

    if (count != n) {
        throw new IllegalArgumentException(
            "Graph is disconnected"
        );
    }

    return total;
}
```

---

# 30. Prim Complexity

With adjacency list + binary heap:

```text
O(E log V)
```

Space:

```text
O(V + E)
```

---

# 31. Kruskal vs Prim

### Kruskal

Uses:

```text
Sort edges
+
DSU
```

Good when:

```text
edge list is readily available
```

### Prim

Uses:

```text
PriorityQueue
+
visited
```

Good when:

```text
graph is represented as adjacency lists
```

Both produce an MST for a connected undirected weighted graph.

---

# 32. Topological Sorting

Topological ordering exists for:

```text
Directed Acyclic Graph (DAG)
```

Example:

```text
A → B → C
```

Valid order:

```text
A, B, C
```

It is used for:

```text
Dependencies
Build systems
Course prerequisites
Task scheduling
```

---

# 33. Kahn's Algorithm

Kahn's algorithm uses:

```text
indegree
+
Queue
```

Steps:

```text
Find all nodes with indegree 0
↓
Add them to queue
↓
Remove one
↓
Reduce indegree of neighbors
↓
Add newly-zero nodes
```

---

# 34. Kahn's Algorithm — Cycle Detection

If:

```text
processedNodes < totalNodes
```

then:

```text
graph contains a cycle
```

because some nodes could never reach indegree zero.

---

# 35. DFS Topological Sort

Another approach:

```text
DFS
+
postorder
```

For every node:

```text
visit dependencies
↓
add node after DFS completes
```

Then reverse the resulting list.

---

# 36. DFS Topological Sort — Java

```java
static void topoDfs(
        int node,
        List<List<Integer>> graph,
        boolean[] visited,
        List<Integer> order) {

    visited[node] = true;

    for (int next :
            graph.get(node)) {

        if (!visited[next]) {

            topoDfs(
                next,
                graph,
                visited,
                order
            );
        }
    }

    order.add(node);
}
```

Then:

```java
Collections.reverse(order);
```

---

# 37. Strongly Connected Components

A Strongly Connected Component (SCC) is a maximal set of vertices in a directed graph where every vertex can reach every other vertex.

Example:

```text
A → B
↑   ↓
D ← C
```

All four can be in one SCC if paths exist both ways between every pair.

---

# 38. SCC Applications

SCCs are useful for:

```text
Dependency analysis
Deadlock analysis
Call graphs
Compiler analysis
Network structure
Cycle condensation
```

Common algorithms:

```text
Kosaraju
Tarjan
```

---

# 39. Kosaraju's Algorithm

Kosaraju uses:

```text
2 DFS passes
+
transpose graph
```

Steps:

```text
1. DFS original graph.
2. Record nodes by finishing time.
3. Reverse every edge.
4. Process nodes in reverse finishing order.
5. Each DFS gives one SCC.
```

---

# 40. Why Transpose Works

If:

```text
A → B
```

the transposed graph contains:

```text
B → A
```

Edges inside an SCC remain mutually reachable after reversal.

The finishing order determines which component should be processed first.

---

# 41. Kosaraju Complexity

```text
Time: O(V + E)
Space: O(V + E)
```

because:

```text
DFS original
+
build transpose
+
DFS transpose
```

are all linear.

---

# 42. Tarjan's SCC Algorithm

Tarjan finds SCCs in:

```text
O(V + E)
```

using one DFS.

It maintains:

```text
discovery time
low-link value
stack
```

For an SCC root:

```text
low[node] == discovery[node]
```

The stack is then popped until the root is reached.

Tarjan is more implementation-heavy than Kosaraju but avoids explicitly building a transpose graph.

---

# 43. Bridges

A bridge is an edge whose removal increases the number of connected components.

Example:

```text
A --- B --- C
```

If:

```text
B-C
```

is removed, `C` becomes disconnected.

Therefore:

```text
B-C
```

is a bridge.

---

# 44. Bridge Detection

Use DFS with:

```text
discovery time
low-link value
```

For an edge:

```text
u → v
```

it is a bridge if:

```text
low[v] > discovery[u]
```

This assumes `v` is a DFS child of `u`.

---

# 45. Bridge Algorithm — Java

```java
static int timer = 0;

static void findBridges(
        int node,
        int parent,
        List<List<Integer>> graph,
        int[] discovery,
        int[] low,
        List<int[]> bridges) {

    discovery[node] =
        low[node] =
            ++timer;

    for (int next :
            graph.get(node)) {

        if (next == parent) {
            continue;
        }

        if (discovery[next] == 0) {

            findBridges(
                next,
                node,
                graph,
                discovery,
                low,
                bridges
            );

            low[node] =
                Math.min(
                    low[node],
                    low[next]
                );

            if (low[next]
                    > discovery[node]) {

                bridges.add(
                    new int[]{
                        node,
                        next
                    }
                );
            }

        } else {

            low[node] =
                Math.min(
                    low[node],
                    discovery[next]
                );
        }
    }
}
```

---

# 46. Articulation Point

An articulation point is a vertex whose removal increases the number of connected components.

Example:

```text
A — B — C
    |
    D
```

Removing:

```text
B
```

disconnects:

```text
A, C, D
```

So `B` is an articulation point.

---

# 47. Articulation Point Rule

For a DFS child:

```text
u → v
```

`u` is an articulation point if:

```text
low[v] >= discovery[u]
```

for a non-root DFS node.

For a DFS root:

```text
root is articulation point
if it has more than one DFS child.
```

---

# 48. Bipartite Graph

A graph is bipartite if its vertices can be divided into two sets such that every edge connects vertices from different sets.

Equivalent condition:

```text
Graph contains no odd cycle.
```

---

# 49. Bipartite Check Using BFS

Use colors:

```text
0
1
```

When visiting an edge:

```text
u → v
```

assign:

```text
color[v] =
1 - color[u]
```

If `v` already has the same color:

```text
not bipartite
```

---

# 50. Bipartite — Java

```java
static boolean isBipartite(
        List<List<Integer>> graph) {

    int n = graph.size();

    int[] color =
        new int[n];

    Arrays.fill(
        color,
        -1
    );

    for (int start = 0;
         start < n;
         start++) {

        if (color[start] != -1) {
            continue;
        }

        Queue<Integer> queue =
            new ArrayDeque<>();

        queue.offer(start);
        color[start] = 0;

        while (!queue.isEmpty()) {

            int node =
                queue.poll();

            for (int next :
                    graph.get(node)) {

                if (color[next] == -1) {

                    color[next] =
                        1 - color[node];

                    queue.offer(next);

                } else if (
                    color[next]
                        == color[node]) {

                    return false;
                }
            }
        }
    }

    return true;
}
```

---

# 51. Bipartite Check Using DFS

The same idea works with DFS.

```java
color[next] =
    1 - color[node];
```

If a neighboring node already has the same color:

```text
false
```

---

# 52. 0-1 BFS

For graph edges with weights only:

```text
0
1
```

use:

```text
Deque
```

instead of Dijkstra's priority queue.

For weight `0`:

```java
deque.offerFirst(next);
```

For weight `1`:

```java
deque.offerLast(next);
```

Complexity:

```text
O(V + E)
```

---

# 53. When to Use Which Shortest Path Algorithm?

| Graph | Algorithm |
|---|---|
| Unweighted | BFS |
| Weights 0/1 | 0-1 BFS |
| Non-negative weights | Dijkstra |
| Negative edges | Bellman-Ford |
| All pairs | Floyd-Warshall |
| DAG | Topological-order relaxation |

This table is extremely important for interviews.

---

# 54. Shortest Path in DAG

A DAG can have negative edge weights and still allow efficient shortest paths.

Steps:

```text
1. Topologically sort graph.
2. Process nodes in topological order.
3. Relax outgoing edges.
```

Complexity:

```text
O(V + E)
```

This is faster than general shortest-path algorithms for DAGs.

---

# 55. DAG Shortest Path

```java
for (int node : topoOrder) {

    if (distance[node]
            == Integer.MAX_VALUE) {
        continue;
    }

    for (int[] edge :
            graph.get(node)) {

        int next = edge[0];
        int weight = edge[1];

        distance[next] =
            Math.min(
                distance[next],
                distance[node]
                    + weight
            );
    }
}
```

---

# 56. DAG Longest Path

For DAGs, longest paths can also be computed efficiently using:

```text
topological order
+
dynamic programming
```

This is possible because DAGs contain no cycles.

The general longest-path problem for arbitrary graphs is much harder.

---

# 57. Graph Condensation

For a directed graph:

```text
Find SCCs
↓
Compress each SCC into one node
```

The resulting graph is:

```text
DAG
```

This is called the:

```text
condensation graph
```

It is useful for advanced graph analysis.

---

# 58. Transitive Closure

Transitive closure answers:

```text
Can i reach j?
```

for every pair of vertices.

One simple solution is:

```text
Floyd-Warshall-style DP
```

with boolean reachability.

Complexity:

```text
O(V³)
```

For sparse graphs, repeated DFS/BFS can sometimes be more appropriate.

---

# 59. Detect Cycle in Directed Graph

Two common approaches:

### DFS

Maintain:

```text
visited
recursionStack
```

If DFS reaches a node currently in the recursion stack:

```text
cycle
```

### Kahn

If:

```text
processedNodes < V
```

then:

```text
cycle
```

---

# 60. Dijkstra vs BFS

If all edges have equal weight:

```text
BFS
```

is simpler and faster.

For example:

```text
every edge weight = 1
```

Use BFS.

Do not use Dijkstra when the problem is simply unweighted.

---

# 61. Dijkstra vs 0-1 BFS

If weights are:

```text
0 or 1
```

use:

```text
0-1 BFS
```

instead of a priority queue when appropriate.

Complexity:

```text
0-1 BFS → O(V + E)
Dijkstra → O((V + E) log V)
```

---

# 62. Dijkstra vs Bellman-Ford

### Dijkstra

```text
Non-negative weights
Fast
PriorityQueue
```

### Bellman-Ford

```text
Negative weights allowed
Can detect negative cycles
Slower
```

---

# 63. Bellman-Ford vs Floyd-Warshall

### Bellman-Ford

```text
Single source
O(VE)
```

### Floyd-Warshall

```text
All pairs
O(V³)
```

Choose based on:

```text
number of sources
number of vertices
graph density
negative edges
```

---

# 64. MST vs Shortest Path

These are different problems.

### MST

Goal:

```text
Connect all vertices
with minimum total edge weight.
```

Algorithms:

```text
Kruskal
Prim
```

### Shortest Path

Goal:

```text
Find minimum-cost route
from source to destination.
```

Algorithms:

```text
BFS
Dijkstra
Bellman-Ford
Floyd-Warshall
```

Do not confuse them.

---

# 65. Common Advanced Graph Mistakes

### Mistake 1 — Using Dijkstra with negative edges

Invalid.

### Mistake 2 — Forgetting stale PriorityQueue entries

Use:

```java
if (currentDistance
        != distance[node]) {
    continue;
}
```

### Mistake 3 — Using BFS for weighted graphs

BFS is correct for:

```text
unweighted
```

or equal-weight edges.

### Mistake 4 — Confusing MST with shortest path

They solve different optimization problems.

### Mistake 5 — Forgetting disconnected components

For DFS/BFS graph algorithms, iterate through every vertex when the graph may be disconnected.

### Mistake 6 — Wrong indegree handling

Kahn's algorithm requires decrementing the indegree of every outgoing neighbor.

### Mistake 7 — Incorrect low-link logic

Bridge and articulation-point problems require careful distinction between:

```text
tree edges
back edges
```

---

# 66. Edge Cases

Always test:

```text
Empty graph
One vertex
Disconnected graph
Single edge
Duplicate edges
Self-loop
Cycle
No cycle
Negative edge
Negative cycle
Zero-weight edges
All nodes connected
Multiple components
Dense graph
Sparse graph
```

---

# 67. Interview Questions — Easy

1. BFS.
2. DFS.
3. Number of connected components.
4. Detect cycle in undirected graph.
5. Check bipartite graph.
6. Topological sorting.
7. Course Schedule.
8. Shortest path in unweighted graph.

---

# 68. Interview Questions — Medium

9. Dijkstra.
10. Network Delay Time.
11. Cheapest Flights Within K Stops.
12. Minimum Spanning Tree.
13. Kruskal.
14. Prim.
15. Number of Provinces.
16. Word Ladder.
17. Rotting Oranges.
18. Course Schedule II.
19. Shortest Path in Binary Matrix.
20. 0-1 BFS.

---

# 69. Interview Questions — Advanced

21. Bellman-Ford.
22. Floyd-Warshall.
23. Strongly Connected Components.
24. Kosaraju.
25. Tarjan.
26. Bridges.
27. Articulation Points.
28. DAG shortest path.
29. Graph condensation.
30. Bitmask graph problems.
31. Advanced state-space BFS.
32. Negative cycle detection.

---

# 70. Algorithm Selection Cheat Sheet

```text
Need connectivity?
        ↓
    DSU / DFS / BFS

Need shortest path?
        ↓
    ┌───────────────┐
    │ Edge weights? │
    └───────┬───────┘
            ↓
       No weights
            ↓
           BFS

       Weight 0/1
            ↓
         0-1 BFS

    Non-negative weights
            ↓
        Dijkstra

       Negative edges
            ↓
      Bellman-Ford

       All pairs
            ↓
      Floyd-Warshall

          DAG
            ↓
 Topological relaxation
```

---

# 71. MST Selection

```text
Need minimum spanning tree?
            ↓
       Connected?
            ↓
      ┌─────┴─────┐
      ↓           ↓
  Edge list    Adjacency
      ↓           ↓
  Kruskal        Prim
      ↓           ↓
     DSU       PriorityQueue
```

---

# 72. SCC Selection

For strongly connected components:

```text
Kosaraju
```

or:

```text
Tarjan
```

Both:

```text
O(V + E)
```

Kosaraju:

```text
2 DFS passes
+
transpose
```

Tarjan:

```text
1 DFS
+
low-link
+
stack
```

---

# 73. Graph Complexity Summary

| Algorithm | Time | Space |
|---|---:|---:|
| BFS | O(V + E) | O(V) |
| DFS | O(V + E) | O(V) |
| Dijkstra | O((V + E) log V) | O(V + E) |
| Bellman-Ford | O(VE) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| 0-1 BFS | O(V + E) | O(V) |
| Topological Sort | O(V + E) | O(V) |
| Kosaraju | O(V + E) | O(V + E) |
| Tarjan SCC | O(V + E) | O(V) |
| Bridges | O(V + E) | O(V) |
| Articulation Points | O(V + E) | O(V) |
| Kruskal | O(E log E) | O(V + E) |
| Prim | O(E log V) | O(V + E) |

---

# 74. Important Graph Patterns

Memorize these patterns:

```text
Unweighted shortest path
→ BFS

Weighted non-negative shortest path
→ Dijkstra

0/1 weights
→ 0-1 BFS

Negative edges
→ Bellman-Ford

All-pairs shortest path
→ Floyd-Warshall

Minimum spanning tree
→ Kruskal / Prim

Dependency ordering
→ Topological Sort

Strong connectivity
→ Kosaraju / Tarjan

Dynamic connectivity
→ DSU

Critical edges
→ Bridges

Critical vertices
→ Articulation Points
```

---

# 75. Backend Interview Connection

Advanced graph algorithms can appear in backend systems involving:

```text
Network routing
Service dependencies
Build pipelines
Workflow scheduling
Dependency resolution
Recommendation graphs
Resource allocation
Distributed systems topology
```

For example:

```text
Service A
   ↓
Service B
   ↓
Service C
```

can be modeled as a directed dependency graph.

A cycle:

```text
A → B → C → A
```

may indicate a dependency problem.

---

# 76. How to Approach a Graph Problem

When you see a graph problem, ask:

```text
1. Directed or undirected?
2. Weighted or unweighted?
3. Positive, zero, or negative weights?
4. Need connectivity or shortest path?
5. One source or all pairs?
6. Is the graph a DAG?
7. Are cycles possible?
8. Is the graph sparse or dense?
9. Static or dynamically changing?
10. Do we need the actual path or only its cost?
```

These questions often reveal the correct algorithm immediately.

---

# 77. Graph Interview Framework

Use this sequence:

```text
Understand graph
↓
Choose representation
↓
Identify objective
↓
Choose algorithm
↓
Define state
↓
Handle visited / distance / parent
↓
Analyze complexity
↓
Test edge cases
```

---

# 78. Quick Revision

```text
Advanced Graph Algorithms
│
├── Shortest Path
│   ├── BFS
│   ├── 0-1 BFS
│   ├── Dijkstra
│   ├── Bellman-Ford
│   ├── Floyd-Warshall
│   └── DAG Shortest Path
│
├── Minimum Spanning Tree
│   ├── Kruskal
│   └── Prim
│
├── Directed Graph
│   ├── Topological Sort
│   ├── Kahn
│   ├── SCC
│   ├── Kosaraju
│   └── Tarjan
│
├── Connectivity
│   ├── DSU
│   ├── Bridges
│   └── Articulation Points
│
└── Graph Properties
    ├── Bipartite
    ├── Cycles
    └── Condensation DAG
```

---

## Most Important Interview Rules

> **BFS for unweighted shortest paths.**

> **Dijkstra for non-negative weighted shortest paths.**

> **Bellman-Ford when negative edges matter.**

> **Floyd-Warshall for all-pairs shortest paths when V is manageable.**

> **Kruskal or Prim for minimum spanning trees.**

> **Topological sorting for dependency ordering in DAGs.**

> **Kosaraju or Tarjan for strongly connected components.**

> **DSU for dynamic connectivity and undirected component merging.**

> **Bridges identify critical edges; articulation points identify critical vertices.**
