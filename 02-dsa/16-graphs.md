# DSA — Graphs

A **Graph** is a data structure used to represent relationships between entities.

Graphs are extremely important in Java backend interviews because they appear in:

- Networks
- Social connections
- Maps
- Dependencies
- Microservice communication
- Recommendation systems
- Routing
- Scheduling
- Shortest paths
- Connectivity
- Topological sorting

---

# 1. What is a Graph?

A graph consists of:

```text
Vertices (Nodes)
+
Edges (Connections)
```

Example:

```text
1 ---- 2
|      |
|      |
3 ---- 4
```

Nodes:

```text
1, 2, 3, 4
```

Edges:

```text
1-2
1-3
2-4
3-4
```

---

# 2. Types of Graphs

Important classifications:

```text
Directed
Undirected

Weighted
Unweighted

Connected
Disconnected

Cyclic
Acyclic

Simple
Multigraph
```

---

# 3. Directed Graph

Edges have a direction.

```text
A → B
```

This means:

```text
A can reach B
```

but not necessarily:

```text
B can reach A
```

Example:

```text
A → B → C
```

---

# 4. Undirected Graph

Edges have no direction.

```text
A — B
```

means:

```text
A connected to B
B connected to A
```

---

# 5. Weighted Graph

Edges contain a cost or weight.

```text
A --5-- B
```

The edge weight is:

```text
5
```

Weights can represent:

```text
Distance
Cost
Time
Latency
Risk
```

---

# 6. Unweighted Graph

Edges do not have explicit weights.

```text
A — B — C
```

For shortest path problems, every edge can effectively be treated as:

```text
weight = 1
```

---

# 7. Connected Graph

An undirected graph is connected if every vertex can reach every other vertex.

Example:

```text
1 — 2 — 3
    |
    4
```

All nodes are connected.

---

# 8. Disconnected Graph

Example:

```text
1 — 2

3 — 4
```

There are multiple connected components.

---

# 9. Cyclic Graph

A graph contains a cycle if you can start at a vertex and return to it without repeating edges/vertices according to the problem's definition.

Example:

```text
1
| \
|  \
2 — 3
```

Cycle:

```text
1 → 2 → 3 → 1
```

---

# 10. Acyclic Graph

An acyclic graph contains no cycles.

A directed acyclic graph is called:

```text
DAG
```

DAGs are extremely important for:

```text
Task dependencies
Build systems
Course prerequisites
Scheduling
Workflow systems
```

---

# 11. Graph Representation

The three most common representations are:

```text
1. Adjacency Matrix
2. Adjacency List
3. Edge List
```

---

# 12. Adjacency Matrix

For `n` vertices:

```java
int[][] graph =
    new int[n][n];
```

If:

```text
graph[i][j] = 1
```

there is an edge from:

```text
i → j
```

For weighted graphs:

```text
graph[i][j] = weight
```

---

# 13. Adjacency Matrix Example

Graph:

```text
1 — 2
|
3
```

Matrix:

```text
    1 2 3
1   0 1 1
2   1 0 0
3   1 0 0
```

For an undirected graph:

```text
matrix[i][j]
=
matrix[j][i]
```

---

# 14. Adjacency Matrix Complexity

Space:

```text
O(V²)
```

Check whether an edge exists:

```text
O(1)
```

Iterate neighbors:

```text
O(V)
```

Good for:

```text
Dense graphs
Small graphs
Fast edge lookup
```

---

# 15. Adjacency List

Store neighbors for each vertex.

Java:

```java
List<List<Integer>> graph =
    new ArrayList<>();
```

Initialize:

```java
for (int i = 0;
     i < n;
     i++) {

    graph.add(
        new ArrayList<>()
    );
}
```

---

# 16. Add Undirected Edge

```java
static void addEdge(
        List<List<Integer>> graph,
        int u,
        int v) {

    graph.get(u).add(v);
    graph.get(v).add(u);
}
```

For:

```text
1 — 2
```

store:

```text
1 → 2
2 → 1
```

---

# 17. Add Directed Edge

```java
static void addDirectedEdge(
        List<List<Integer>> graph,
        int u,
        int v) {

    graph.get(u).add(v);
}
```

For:

```text
1 → 2
```

only:

```text
1 → 2
```

is stored.

---

# 18. Adjacency List Complexity

Space:

```text
O(V + E)
```

Check a particular edge:

```text
O(degree)
```

Iterate neighbors:

```text
O(degree)
```

Adjacency lists are usually preferred for sparse graphs.

---

# 19. Edge List

Store edges directly:

```java
int[][] edges = {
    {0, 1},
    {1, 2},
    {2, 3}
};
```

For weighted edges:

```java
int[][] edges = {
    {0, 1, 5},
    {1, 2, 3}
};
```

This representation is useful for:

```text
Kruskal
Sorting edges
Input/output
```

---

# 20. BFS

Breadth First Search explores a graph level by level.

Use:

```text
Queue
+
Visited
```

---

# 21. BFS — Java

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

        System.out.print(
            node + " "
        );

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

Complexity:

```text
Time: O(V + E)
Space: O(V)
```

---

# 22. DFS

Depth First Search explores one path as deeply as possible before backtracking.

Can use:

```text
Recursion
```

or:

```text
Stack
```

---

# 23. DFS — Recursive

```java
static void dfs(
        List<List<Integer>> graph,
        int node,
        boolean[] visited) {

    visited[node] = true;

    System.out.print(
        node + " "
    );

    for (int next :
            graph.get(node)) {

        if (!visited[next]) {

            dfs(
                graph,
                next,
                visited
            );
        }
    }
}
```

Complexity:

```text
Time: O(V + E)
Space: O(V)
```

including recursion/visited state.

---

# 24. DFS — Iterative

```java
static void dfsIterative(
        List<List<Integer>> graph,
        int start) {

    boolean[] visited =
        new boolean[graph.size()];

    Deque<Integer> stack =
        new ArrayDeque<>();

    stack.push(start);

    while (!stack.isEmpty()) {

        int node =
            stack.pop();

        if (visited[node]) {
            continue;
        }

        visited[node] = true;

        System.out.print(
            node + " "
        );

        for (int next :
                graph.get(node)) {

            if (!visited[next]) {
                stack.push(next);
            }
        }
    }
}
```

---

# 25. BFS vs DFS

### BFS

Uses:

```text
Queue
```

Best for:

```text
Shortest path in unweighted graph
Level-by-level exploration
Minimum number of edges
Multi-source problems
```

### DFS

Uses:

```text
Stack / recursion
```

Best for:

```text
Connected components
Cycle detection
Backtracking
Topological DFS
Graph exploration
```

---

# 26. Connected Components

For an undirected graph:

```text
1 — 2

3 — 4

5
```

there are:

```text
3 connected components
```

Run DFS/BFS from every unvisited node.

---

# 27. Connected Components — Java

```java
static int countComponents(
        int n,
        List<List<Integer>> graph) {

    boolean[] visited =
        new boolean[n];

    int components = 0;

    for (int i = 0;
         i < n;
         i++) {

        if (!visited[i]) {

            components++;

            dfs(
                graph,
                i,
                visited
            );
        }
    }

    return components;
}
```

Complexity:

```text
O(V + E)
```

---

# 28. Number of Islands

Given a grid:

```text
1 = land
0 = water
```

count connected land components.

Each island is a connected component.

Use:

```text
DFS
```

or:

```text
BFS
```

---

# 29. Grid DFS

```java
static void dfs(
        char[][] grid,
        int row,
        int col) {

    if (row < 0
            || row >= grid.length
            || col < 0
            || col >= grid[0].length
            || grid[row][col] != '1') {

        return;
    }

    grid[row][col] = '0';

    dfs(
        grid,
        row + 1,
        col
    );

    dfs(
        grid,
        row - 1,
        col
    );

    dfs(
        grid,
        row,
        col + 1
    );

    dfs(
        grid,
        row,
        col - 1
    );
}
```

---

# 30. Cycle Detection — Undirected Graph

For an undirected graph, DFS can track:

```text
parent
```

If we encounter a visited neighbor that is not the parent:

```text
cycle exists
```

---

# 31. Undirected Cycle Detection

```java
static boolean hasCycle(
        List<List<Integer>> graph) {

    boolean[] visited =
        new boolean[graph.size()];

    for (int i = 0;
         i < graph.size();
         i++) {

        if (!visited[i]
                && dfsCycle(
                    graph,
                    i,
                    -1,
                    visited)) {

            return true;
        }
    }

    return false;
}

static boolean dfsCycle(
        List<List<Integer>> graph,
        int node,
        int parent,
        boolean[] visited) {

    visited[node] = true;

    for (int next :
            graph.get(node)) {

        if (!visited[next]) {

            if (dfsCycle(
                    graph,
                    next,
                    node,
                    visited)) {

                return true;
            }

        } else if (next != parent) {

            return true;
        }
    }

    return false;
}
```

---

# 32. Cycle Detection — Directed Graph

For directed graphs, a visited node alone is not enough.

Use three states:

```text
0 = unvisited
1 = currently in recursion stack
2 = completely processed
```

If we reach a node with state:

```text
1
```

we found a directed cycle.

---

# 33. Directed Cycle Detection

```java
static boolean hasDirectedCycle(
        List<List<Integer>> graph) {

    int[] state =
        new int[graph.size()];

    for (int i = 0;
         i < graph.size();
         i++) {

        if (state[i] == 0
                && dfsDirectedCycle(
                    graph,
                    i,
                    state)) {

            return true;
        }
    }

    return false;
}

static boolean dfsDirectedCycle(
        List<List<Integer>> graph,
        int node,
        int[] state) {

    state[node] = 1;

    for (int next :
            graph.get(node)) {

        if (state[next] == 1) {
            return true;
        }

        if (state[next] == 0
                && dfsDirectedCycle(
                    graph,
                    next,
                    state)) {

            return true;
        }
    }

    state[node] = 2;

    return false;
}
```

---

# 34. Topological Sort

Topological sorting orders vertices of a DAG so that:

```text
u → v
```

means:

```text
u appears before v
```

Example:

```text
Learn Java
    ↓
Spring Boot
    ↓
Microservices
```

Possible ordering:

```text
Java → Spring Boot → Microservices
```

---

# 35. Topological Sort — Kahn's Algorithm

Uses:

```text
Indegree
+
Queue
```

Steps:

```text
1. Calculate indegree.
2. Add all indegree-0 nodes.
3. Remove one node.
4. Decrease neighbors' indegree.
5. Add newly-zero nodes.
6. Repeat.
```

---

# 36. Kahn's Algorithm — Java

```java
static List<Integer> topologicalSort(
        List<List<Integer>> graph) {

    int n = graph.size();

    int[] indegree =
        new int[n];

    for (int node = 0;
         node < n;
         node++) {

        for (int next :
                graph.get(node)) {

            indegree[next]++;
        }
    }

    Queue<Integer> queue =
        new ArrayDeque<>();

    for (int i = 0;
         i < n;
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

    if (result.size() != n) {
        throw new IllegalArgumentException(
            "Graph contains a cycle"
        );
    }

    return result;
}
```

Complexity:

```text
O(V + E)
```

---

# 37. Topological Sort Using DFS

Alternative approach:

```text
DFS
+
postorder
+
stack
```

Add a node after processing all its neighbors.

Reverse the finishing order.

This produces a topological ordering for a DAG.

---

# 38. Shortest Path in Unweighted Graph

If every edge has equal cost:

```text
BFS
```

is usually the right algorithm.

Maintain:

```java
int[] distance;
```

Initialize:

```java
Arrays.fill(distance, -1);
```

Then:

```text
distance[source] = 0
```

---

# 39. Unweighted Shortest Path — Java

```java
static int[] shortestPath(
        List<List<Integer>> graph,
        int source) {

    int[] distance =
        new int[graph.size()];

    Arrays.fill(
        distance,
        -1
    );

    Queue<Integer> queue =
        new ArrayDeque<>();

    distance[source] = 0;
    queue.offer(source);

    while (!queue.isEmpty()) {

        int node =
            queue.poll();

        for (int next :
                graph.get(node)) {

            if (distance[next] != -1) {
                continue;
            }

            distance[next] =
                distance[node] + 1;

            queue.offer(next);
        }
    }

    return distance;
}
```

---

# 40. Dijkstra's Algorithm

Use Dijkstra when:

```text
Edge weights are non-negative.
```

It finds shortest paths from one source.

Use:

```text
PriorityQueue
```

to process the node with the smallest tentative distance.

---

# 41. Dijkstra — Core Java

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
                pair -> pair[1]
            )
        );

    distance[source] = 0;

    pq.offer(
        new int[]{source, 0}
    );

    while (!pq.isEmpty()) {

        int[] current =
            pq.poll();

        int node =
            current[0];

        int currentDistance =
            current[1];

        if (currentDistance
                != distance[node]) {

            continue;
        }

        for (int[] edge :
                graph.get(node)) {

            int next =
                edge[0];

            int weight =
                edge[1];

            int newDistance =
                currentDistance + weight;

            if (newDistance
                    < distance[next]) {

                distance[next] =
                    newDistance;

                pq.offer(
                    new int[]{
                        next,
                        newDistance
                    }
                );
            }
        }
    }

    return distance;
}
```

---

# 42. Dijkstra Important Rule

Dijkstra does **not** work correctly with negative edge weights in its standard form.

Use other algorithms when negative weights are possible.

---

# 43. Bellman-Ford

Bellman-Ford can handle:

```text
negative edge weights
```

and can detect:

```text
negative cycles
```

Basic idea:

```text
Relax every edge
V - 1 times
```

Complexity:

```text
O(VE)
```

It is slower than Dijkstra but more flexible with negative weights.

---

# 44. Floyd-Warshall

Floyd-Warshall finds:

```text
all-pairs shortest paths
```

using dynamic programming.

Complexity:

```text
Time: O(V³)
Space: O(V²)
```

Useful when:

```text
number of vertices is relatively small
```

and we need distances between every pair.

---

# 45. Minimum Spanning Tree

A Minimum Spanning Tree (MST) is a set of edges that:

```text
connects all vertices
has no cycles
has minimum total weight
```

For a connected undirected weighted graph.

Two major algorithms:

```text
Kruskal
Prim
```

---

# 46. Kruskal's Algorithm

Kruskal:

```text
Sort all edges by weight.
↓
Take smallest edge
if it does not create a cycle.
```

Use:

```text
Disjoint Set Union
```

to detect whether two vertices are already connected.

---

# 47. Kruskal Complexity

Sorting:

```text
O(E log E)
```

DSU operations are almost constant amortized:

```text
O(α(V))
```

Overall:

```text
O(E log E)
```

---

# 48. Disjoint Set Union

Also called:

```text
Union-Find
```

Supports:

```text
find(x)
union(a, b)
```

Used for:

```text
Cycle detection
Kruskal
Connected components
Dynamic connectivity
```

---

# 49. DSU with Path Compression

```java
class DSU {

    private final int[] parent;
    private final int[] rank;

    DSU(int n) {

        parent =
            new int[n];

        rank =
            new int[n];

        for (int i = 0;
             i < n;
             i++) {

            parent[i] = i;
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
            int first,
            int second) {

        int rootFirst =
            find(first);

        int rootSecond =
            find(second);

        if (rootFirst == rootSecond) {
            return false;
        }

        if (rank[rootFirst]
                < rank[rootSecond]) {

            parent[rootFirst] =
                rootSecond;

        } else if (
            rank[rootFirst]
                > rank[rootSecond]) {

            parent[rootSecond] =
                rootFirst;

        } else {

            parent[rootSecond] =
                rootFirst;

            rank[rootFirst]++;
        }

        return true;
    }
}
```

---

# 50. Prim's Algorithm

Prim grows the MST from a starting vertex.

At each step:

```text
Choose the minimum-weight edge
that connects the current tree
to a new vertex.
```

Use:

```text
PriorityQueue
```

---

# 51. Prim vs Kruskal

### Prim

```text
Vertex-focused
PriorityQueue
Good with adjacency lists
```

### Kruskal

```text
Edge-focused
Sort edges
DSU
```

Both find an MST for a connected, undirected, weighted graph.

---

# 52. Bipartite Graph

A graph is bipartite if its vertices can be divided into two sets such that:

```text
No edge connects vertices
within the same set.
```

Example:

```text
Set A: 1, 3
Set B: 2, 4
```

Edges go:

```text
A → B
```

only.

---

# 53. Check Bipartite Graph

Use:

```text
BFS/DFS
+
2-coloring
```

Assign:

```text
color 0
color 1
```

For every edge:

```text
neighbors must have opposite colors.
```

---

# 54. Bipartite Check — Java

```java
static boolean isBipartite(
        List<List<Integer>> graph) {

    int[] color =
        new int[graph.size()];

    Arrays.fill(
        color,
        -1
    );

    for (int start = 0;
         start < graph.size();
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

# 55. Bridges

A bridge is an edge whose removal increases the number of connected components.

Example:

```text
1 — 2 — 3
    |
    4
```

An edge that is the only connection to a component may be a bridge.

Finding bridges commonly uses:

```text
DFS
discovery time
low-link values
```

This is an advanced graph topic.

---

# 56. Articulation Points

An articulation point is a vertex whose removal increases the number of connected components.

Finding articulation points also uses:

```text
DFS
discovery time
low-link values
```

These concepts are common in advanced graph interviews.

---

# 57. Strongly Connected Components

In a directed graph, a strongly connected component (SCC) is a group where:

```text
every vertex can reach every other vertex
```

Popular algorithms:

```text
Kosaraju
Tarjan
```

---

# 58. Kosaraju's Algorithm

High-level steps:

```text
1. DFS on original graph.
2. Store nodes by finishing time.
3. Reverse all edges.
4. Process nodes in reverse finishing order.
5. Each DFS gives one SCC.
```

Complexity:

```text
O(V + E)
```

---

# 59. Tarjan's Algorithm

Tarjan finds SCCs using:

```text
DFS
discovery index
low-link value
stack
```

It can find all SCCs in:

```text
O(V + E)
```

---

# 60. Graph Coloring

Graph coloring assigns colors so that:

```text
Adjacent vertices have different colors.
```

Bipartite checking is essentially:

```text
2-coloring
```

General graph coloring is much harder and is a classic optimization/complexity problem.

---

# 61. Course Schedule

Course prerequisites form a directed graph.

Example:

```text
0 → 1
```

means:

```text
course 0 must be completed before course 1
```

If the graph contains a cycle:

```text
0 → 1 → 2 → 0
```

the courses cannot all be completed.

Use:

```text
Topological Sort
```

---

# 62. Word Ladder

Given words, change one character at a time.

Each valid word is a graph node.

An edge exists when two words differ by one character.

Because every transformation has equal cost:

```text
BFS
```

finds the shortest transformation sequence.

---

# 63. Clone Graph

To clone a graph:

```text
DFS/BFS
+
Map original → clone
```

The map prevents:

```text
duplicate cloning
```

and handles cycles.

Core idea:

```java
Map<Node, Node> clones =
    new HashMap<>();
```

---

# 64. Clone Graph — Core Pattern

```java
static Node cloneGraph(Node node) {

    if (node == null) {
        return null;
    }

    Map<Node, Node> clones =
        new HashMap<>();

    return clone(
        node,
        clones
    );
}

static Node clone(
        Node node,
        Map<Node, Node> clones) {

    if (clones.containsKey(node)) {
        return clones.get(node);
    }

    Node copy =
        new Node(node.val);

    clones.put(
        node,
        copy
    );

    for (Node neighbor :
            node.neighbors) {

        copy.neighbors.add(
            clone(
                neighbor,
                clones
            )
        );
    }

    return copy;
}
```

---

# 65. Graph Traversal Template

```java
void dfs(
        int node,
        boolean[] visited,
        List<List<Integer>> graph) {

    visited[node] = true;

    for (int next :
            graph.get(node)) {

        if (!visited[next]) {
            dfs(
                next,
                visited,
                graph
            );
        }
    }
}
```

---

# 66. BFS Template

```java
Queue<Integer> queue =
    new ArrayDeque<>();

queue.offer(source);
visited[source] = true;

while (!queue.isEmpty()) {

    int node =
        queue.poll();

    for (int next :
            graph.get(node)) {

        if (!visited[next]) {

            visited[next] = true;
            queue.offer(next);
        }
    }
}
```

---

# 67. Shortest Path Decision Guide

Use:

```text
Unweighted graph
→ BFS

0/1 edge weights
→ 0-1 BFS

Non-negative weights
→ Dijkstra

Negative weights
→ Bellman-Ford

All pairs
→ Floyd-Warshall

DAG shortest path
→ Topological order + relaxation
```

This decision tree is very useful in interviews.

---

# 68. Graph Algorithm Decision Guide

### Need to explore?

```text
DFS / BFS
```

### Need shortest unweighted path?

```text
BFS
```

### Need shortest weighted path?

```text
Dijkstra / Bellman-Ford
```

### Need dependencies?

```text
Topological Sort
```

### Need minimum spanning tree?

```text
Kruskal / Prim
```

### Need dynamic connectivity?

```text
DSU
```

### Need cycle detection?

```text
DFS / BFS / DSU
```

depending on graph type.

---

# 69. Common Graph Mistakes

### Mistake 1 — Forgetting visited

Can cause:

```text
infinite traversal
```

in cyclic graphs.

---

### Mistake 2 — Marking visited too late

For BFS, mark nodes when discovered/enqueued.

---

### Mistake 3 — Treating directed as undirected

An edge:

```text
A → B
```

does not imply:

```text
B → A
```

---

### Mistake 4 — Using Dijkstra with negative weights

Standard Dijkstra requires:

```text
non-negative edge weights
```

---

### Mistake 5 — Forgetting disconnected components

Starting DFS/BFS from one node does not necessarily visit the entire graph.

Use:

```java
for every unvisited node
```

when the whole graph matters.

---

### Mistake 6 — Wrong cycle detection

Undirected and directed graphs require different logic.

---

### Mistake 7 — Confusing MST with shortest path

MST minimizes:

```text
total weight of a tree connecting all vertices.
```

Shortest path minimizes:

```text
distance from one source to another.
```

They are different problems.

---

# 70. Edge Cases

Always test:

```text
Empty graph
One vertex
No edges
Disconnected graph
Single edge
Self-loop
Duplicate edges
Directed graph
Undirected graph
Cycle
No cycle
Negative edge
Zero-weight edge
Multiple shortest paths
Multiple MSTs
```

---

# 71. Interview Questions — Easy

1. BFS traversal.
2. DFS traversal.
3. Number of connected components.
4. Number of Islands.
5. Find if Path Exists.
6. Flood Fill.
7. Clone Graph.
8. Find Center of Star Graph.
9. Find Town Judge.
10. Find if Graph is Bipartite.

---

# 72. Interview Questions — Medium

11. Course Schedule.
12. Course Schedule II.
13. Rotting Oranges.
14. Word Ladder.
15. Shortest Path in Binary Matrix.
16. Network Delay Time.
17. Number of Provinces.
18. Pacific Atlantic Water Flow.
19. Surrounded Regions.
20. Keys and Rooms.
21. Evaluate Division.
22. Graph Valid Tree.
23. Redundant Connection.
24. Cheapest Flights Within K Stops.

---

# 73. Interview Questions — Advanced

25. Dijkstra's Algorithm.
26. Bellman-Ford.
27. Floyd-Warshall.
28. Kruskal's MST.
29. Prim's MST.
30. Bridges.
31. Articulation Points.
32. Strongly Connected Components.
33. Tarjan's Algorithm.
34. Kosaraju's Algorithm.
35. 0-1 BFS.
36. Shortest path in DAG.
37. Advanced graph state-space problems.

---

# 74. Complexity Summary

| Problem / Algorithm | Time | Space |
|---|---:|---:|
| BFS | O(V + E) | O(V) |
| DFS | O(V + E) | O(V) |
| Connected Components | O(V + E) | O(V) |
| Cycle Detection | O(V + E) | O(V) |
| Topological Sort | O(V + E) | O(V) |
| Bipartite Check | O(V + E) | O(V) |
| Unweighted Shortest Path | O(V + E) | O(V) |
| Dijkstra with Binary Heap | O((V + E) log V) | O(V) |
| Bellman-Ford | O(VE) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| Kruskal | O(E log E) | O(V) |
| Prim with Binary Heap | O((V + E) log V) | O(V) |
| DSU Operation | O(α(V)) amortized | O(V) |
| SCC | O(V + E) | O(V) |
| 0-1 BFS | O(V + E) | O(V) |

---

# 75. Quick Revision

```text
Graphs
│
├── Representation
│   ├── Adjacency Matrix
│   ├── Adjacency List
│   └── Edge List
│
├── Traversal
│   ├── BFS
│   └── DFS
│
├── Connectivity
│   ├── Components
│   ├── DSU
│   └── Bipartite
│
├── Cycles
│   ├── Undirected
│   └── Directed
│
├── Ordering
│   └── Topological Sort
│
├── Shortest Path
│   ├── BFS
│   ├── 0-1 BFS
│   ├── Dijkstra
│   ├── Bellman-Ford
│   └── Floyd-Warshall
│
├── MST
│   ├── Kruskal
│   └── Prim
│
└── Advanced
    ├── Bridges
    ├── Articulation Points
    ├── SCC
    ├── Tarjan
    └── Kosaraju
```

---

## Interview Rule

> **First identify the graph type: directed or undirected, weighted or unweighted, cyclic or acyclic. Then choose the algorithm. For interviews, master BFS, DFS, cycle detection, topological sort, Dijkstra, DSU, Kruskal, and Prim before moving to advanced graph algorithms.**
