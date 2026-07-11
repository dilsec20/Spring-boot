# 🕸️ Graphs — FAANG Masterclass

> **"If a tree is a linked list that couldn't decide which way to go, a graph is a tree that got completely lost and started looping back on itself."**

Graphs are the foundation of social networks, Google Maps routing, and the internet itself.

---

## 📑 Table of Contents
1. [Representation (Matrix vs List)](#1-representation-matrix-vs-list)
2. [Graph Traversals (BFS & DFS) — With Dry Runs](#2-graph-traversals-bfs--dfs)
3. [Cycle Detection (Undirected & Directed)](#3-cycle-detection)
4. [Number of Islands (Matrix DFS)](#4-number-of-islands)
5. [Rotting Oranges (Multi-Source BFS)](#5-rotting-oranges)
6. [Topological Sorting (Kahn's BFS + DFS)](#6-topological-sorting)
7. [Course Schedule I & II](#7-course-schedule-i--ii)
8. [Shortest Path Algorithms](#8-shortest-path-algorithms)
    - [Dijkstra's Algorithm](#dijkstras-algorithm)
    - [Bellman-Ford](#bellman-ford)
    - [Floyd-Warshall](#floyd-warshall)
9. [Word Ladder (BFS Shortest Transformation)](#9-word-ladder)
10. [Clone Graph](#10-clone-graph)
11. [Pacific Atlantic Water Flow](#11-pacific-atlantic-water-flow)
12. [Is Graph Bipartite?](#12-is-graph-bipartite)
13. [Cheapest Flights Within K Stops](#13-cheapest-flights-within-k-stops)
14. [Alien Dictionary](#14-alien-dictionary)
15. [How to Identify Graph Patterns](#15-how-to-identify-graph-patterns)
16. [Top 20 Must-Do Graph Questions](#16-top-20-must-do-graph-questions)

---

## 1. Representation (Matrix vs List)

A Graph consists of Vertices (Nodes) and Edges.

### Adjacency Matrix (2D Array)
*   `graph[i][j] = 1` if there is an edge between `i` and `j`.
*   *Pros:* Extremely fast $O(1)$ to check if an edge exists.
*   *Cons:* Wastes massive amounts of memory $O(V^2)$ if the graph is sparse (few edges).

### Adjacency List (Array of Lists) — **Use this in interviews!**
*   `List<List<Integer>> adj = new ArrayList<>();`
*   `adj.get(i)` returns a list of all neighbors of node `i`.
*   *Pros:* Memory efficient $O(V + E)$. Best for almost all interview problems.

```java
// Building an adjacency list
int V = 5; // Number of vertices
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < V; i++) adj.add(new ArrayList<>());

// Add undirected edge between u and v
adj.get(0).add(1);  // 0 → 1
adj.get(1).add(0);  // 1 → 0 (undirected = both directions)

// For weighted graphs, use List<List<int[]>> where int[] = {neighbor, weight}
```

```
Example Graph:
    0 --- 1
    |     |
    3 --- 2
    |
    4

Adjacency List:
  0: [1, 3]
  1: [0, 2]
  2: [1, 3]
  3: [0, 2, 4]
  4: [3]
```

---

## 2. Graph Traversals (BFS & DFS)

Unlike Trees, Graphs have **Cycles**. If you don't keep track of where you've been, you will loop infinitely.
**Rule #1 of Graphs:** ALWAYS use a `boolean[] visited` array!

### Breadth-First Search (BFS)
Used to find the **Shortest Path in an UNWEIGHTED graph**.
Uses a `Queue`. Explores level-by-level (closest nodes first).

```java
public List<Integer> bfs(int start, List<List<Integer>> adj, int V) {
    List<Integer> result = new ArrayList<>();
    boolean[] visited = new boolean[V];
    Queue<Integer> q = new LinkedList<>();
    
    q.add(start);
    visited[start] = true;
    
    while (!q.isEmpty()) {
        int node = q.poll();
        result.add(node);
        
        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                q.add(neighbor);
            }
        }
    }
    return result;
}
```

### 🧠 BFS Dry Run

```
Graph:
    0 --- 1
    |     |
    3 --- 2
    |
    4

Starting BFS from node 0:

Step | Queue (front→back) | Poll | Visit | Add neighbors
-----|---------------------|------|-------|---------------
  1  | [0]                 | 0    | ✓     | 1, 3
  2  | [1, 3]              | 1    | ✓     | 2 (0 already visited)
  3  | [3, 2]              | 3    | ✓     | 4 (0,2 → 0 visited, 2 added)
  4  | [2, 4]              | 2    | ✓     | (1,3 already visited)
  5  | [4]                 | 4    | ✓     | (3 already visited)
  6  | []                  | DONE |       |

BFS Order: 0 → 1 → 3 → 2 → 4  (level by level!)
```

### Depth-First Search (DFS)
Used to explore all paths, detect cycles, and topological sorting.
Uses Recursion (Stack). Goes as deep as possible before backtracking.

```java
public void dfs(int node, List<List<Integer>> adj, boolean[] visited, List<Integer> result) {
    visited[node] = true;
    result.add(node);
    
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            dfs(neighbor, adj, visited, result);
        }
    }
}
```

### 🧠 DFS Dry Run

```
Same Graph, starting DFS from node 0:

Call Stack                    | Action          | Result so far
------------------------------|-----------------|---------------
dfs(0)                        | Visit 0         | [0]
  → dfs(1)                   | Visit 1         | [0, 1]
    → dfs(2)                 | Visit 2         | [0, 1, 2]
      → dfs(3)               | Visit 3         | [0, 1, 2, 3]
        → dfs(4)             | Visit 4         | [0, 1, 2, 3, 4]
        ← return to 3        | No more unvisited|
      ← return to 2          |                  |
    ← return to 1            |                  |
  ← return to 0              |                  |

DFS Order: 0 → 1 → 2 → 3 → 4  (goes deep first!)
```

### Connected Components (Multiple Islands)
If the graph is disconnected, BFS/DFS from one node won't visit all nodes. Run BFS/DFS from every unvisited node.

```java
public int countComponents(int V, List<List<Integer>> adj) {
    boolean[] visited = new boolean[V];
    int count = 0;
    
    for (int i = 0; i < V; i++) {
        if (!visited[i]) {
            dfs(i, adj, visited, new ArrayList<>());
            count++;  // Each DFS call = one connected component
        }
    }
    return count;
}
```

---

## 3. Cycle Detection

### Undirected Graph (Using DFS)
If you visit a node that is *already visited*, and it is NOT the node you just came from (the parent), you found a cycle!

```java
public boolean hasCycle(int node, int parent, List<List<Integer>> adj, boolean[] visited) {
    visited[node] = true;
    
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            if (hasCycle(neighbor, node, adj, visited)) return true;
        } else if (neighbor != parent) {
            return true; // Visited AND not parent → CYCLE!
        }
    }
    return false;
}
```

### 🧠 Dry Run (Undirected Cycle Detection)
```
Graph with cycle:
    0 --- 1
    |     |
    3 --- 2

dfs(0, parent=-1):
  visit 0, check neighbor 1 → not visited
    dfs(1, parent=0):
      visit 1, check neighbor 0 → visited BUT parent → skip
      check neighbor 2 → not visited
        dfs(2, parent=1):
          visit 2, check neighbor 1 → visited BUT parent → skip
          check neighbor 3 → not visited
            dfs(3, parent=2):
              visit 3, check neighbor 0 → visited AND NOT parent (parent=2)
              → CYCLE DETECTED! ✅
```

### Directed Graph (Using DFS + Path Visited)
In directed graphs, a visited node is NOT necessarily a cycle. You need to track if a node is in the **current DFS path** (recursion stack).

```java
public boolean hasCycleDirected(int node, List<List<Integer>> adj, 
                                 boolean[] visited, boolean[] pathVisited) {
    visited[node] = true;
    pathVisited[node] = true;  // Mark as part of current DFS path
    
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            if (hasCycleDirected(neighbor, adj, visited, pathVisited)) return true;
        } else if (pathVisited[neighbor]) {
            return true; // In current path → CYCLE!
        }
    }
    
    pathVisited[node] = false;  // Backtrack: remove from current path
    return false;
}
```

### 🧠 Why two arrays for directed?
```
Graph: 1 → 2 → 3, and 4 → 3 (no cycle!)

If we only use visited[]:
  DFS from 1: visit 1→2→3 (all marked visited)
  DFS from 4: visit 4, check neighbor 3 → ALREADY VISITED!
  False positive! This is NOT a cycle. Node 3 was visited from a different path.

With pathVisited[]:
  DFS from 1: pathVisited[1,2,3]=true → backtrack → pathVisited[1,2,3]=false
  DFS from 4: pathVisited[4]=true, check 3 → visited but NOT in current path → No cycle ✅
```

---

## 4. Number of Islands

**Problem (LC 200):** Given a 2D grid of `'1'`s (land) and `'0'`s (water), count the number of islands.

### 🧠 How to Think
Each cell is a node. Adjacent cells (up/down/left/right) are edges. → **Implicit graph!**
Run DFS/BFS from every unvisited `'1'`. Each run = one island.

### 🧠 Dry Run

```
Grid:
  1 1 0 0 0
  1 1 0 0 0
  0 0 1 0 0
  0 0 0 1 1

Scan row by row:
  (0,0) = '1', not visited → DFS! Mark all connected '1's as visited.
    DFS visits: (0,0), (0,1), (1,0), (1,1) → Island 1 ✅
  (0,2) = '0' → skip
  (2,2) = '1', not visited → DFS! → Island 2 ✅
  (3,3) = '1', not visited → DFS visits (3,3), (3,4) → Island 3 ✅

Answer: 3 islands
```

```java
public int numIslands(char[][] grid) {
    int count = 0;
    int m = grid.length, n = grid[0].length;
    
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == '1') {
                dfs(grid, i, j);
                count++;
            }
        }
    }
    return count;
}

private void dfs(char[][] grid, int i, int j) {
    // Boundary check + water check + already visited check
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == '0') {
        return;
    }
    
    grid[i][j] = '0'; // Mark as visited (sink the island)
    
    dfs(grid, i + 1, j); // Down
    dfs(grid, i - 1, j); // Up
    dfs(grid, i, j + 1); // Right
    dfs(grid, i, j - 1); // Left
}
```

### Variation: Max Area of Island (LC 695)
```java
// Same DFS but return the area count
private int dfs(int[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == 0)
        return 0;
    
    grid[i][j] = 0; // Mark visited
    return 1 + dfs(grid, i+1, j) + dfs(grid, i-1, j) 
             + dfs(grid, i, j+1) + dfs(grid, i, j-1);
}
```

---

## 5. Rotting Oranges

**Problem (LC 994):** In a grid, `0` = empty, `1` = fresh orange, `2` = rotten orange. Every minute, rotten oranges rot adjacent (up/down/left/right) fresh oranges. Return minimum minutes until no fresh orange remains, or `-1` if impossible.

### 🧠 How to Think
This is **Multi-Source BFS**! Start BFS from ALL rotten oranges simultaneously (add all to queue at start). Each BFS level = 1 minute.

### 🧠 Dry Run

```
Grid:
  2 1 1
  1 1 0
  0 1 1

Step 0: Queue = [(0,0)], fresh = 6
  After level: rot (0,1), (1,0) → Queue = [(0,1),(1,0)], fresh = 4

Step 1 (minute 1): Process (0,1) and (1,0)
  (0,1) rots → (0,2), (1,1)
  (1,0) rots → (nothing new)
  Queue = [(0,2),(1,1)], fresh = 2

Step 2 (minute 2): Process (0,2) and (1,1)
  (1,1) rots → (2,1)
  Queue = [(2,1)], fresh = 1

Step 3 (minute 3): Process (2,1)
  (2,1) rots → (2,2)
  Queue = [(2,2)], fresh = 0

Step 4 (minute 4): Process (2,2) → no fresh neighbors
  Queue empty, fresh = 0 → Answer: 4 minutes ✅
```

```java
public int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    Queue<int[]> queue = new LinkedList<>();
    int fresh = 0;
    
    // Step 1: Add ALL rotten oranges to queue + count fresh
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) queue.add(new int[]{i, j});
            else if (grid[i][j] == 1) fresh++;
        }
    }
    
    if (fresh == 0) return 0;  // No fresh oranges
    
    int minutes = 0;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    
    // Step 2: BFS level by level
    while (!queue.isEmpty()) {
        int size = queue.size();
        boolean rotted = false;
        
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            
            for (int[] d : dirs) {
                int r = cell[0] + d[0], c = cell[1] + d[1];
                
                if (r >= 0 && r < m && c >= 0 && c < n && grid[r][c] == 1) {
                    grid[r][c] = 2;  // Rot it!
                    queue.add(new int[]{r, c});
                    fresh--;
                    rotted = true;
                }
            }
        }
        if (rotted) minutes++;
    }
    
    return fresh == 0 ? minutes : -1;  // -1 if unreachable fresh oranges
}
```

---

## 6. Topological Sorting

**What is it?** A linear ordering of vertices in a DAG (Directed Acyclic Graph) such that for every edge `u → v`, `u` comes before `v`.

**When to use?** Prerequisites, dependencies, build order, task scheduling.

### Kahn's Algorithm (BFS based) — **MEMORIZE THIS!**
1. Calculate `in-degree` for all nodes.
2. Add all nodes with `in-degree == 0` to queue.
3. Pop a node, add to result, decrease in-degree of neighbors.
4. If neighbor's in-degree becomes 0, add to queue.
5. **Cycle check:** If `result.size() != V`, there's a cycle!

```java
public List<Integer> topologicalSort(int V, List<List<Integer>> adj) {
    int[] inDegree = new int[V];
    
    // Step 1: Calculate in-degrees
    for (int u = 0; u < V; u++) {
        for (int v : adj.get(u)) {
            inDegree[v]++;
        }
    }
    
    // Step 2: Add all nodes with in-degree 0 to queue
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < V; i++) {
        if (inDegree[i] == 0) queue.add(i);
    }
    
    // Step 3: BFS
    List<Integer> result = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        result.add(node);
        
        for (int neighbor : adj.get(node)) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] == 0) {
                queue.add(neighbor);
            }
        }
    }
    
    // Step 4: Cycle check
    if (result.size() != V) {
        throw new RuntimeException("Cycle detected! Topological sort not possible.");
    }
    
    return result;
}
```

### 🧠 Dry Run (Kahn's Algorithm)

```
Graph (course prerequisites):
  5 → 0, 5 → 2
  4 → 0, 4 → 1
  2 → 3
  3 → 1

In-degrees: [2, 2, 1, 1, 0, 0]
             0  1  2  3  4  5

Step | Queue      | Poll | Reduce in-degree of | Result
-----|------------|------|---------------------|-------
  1  | [4, 5]     | 4    | 0(2→1), 1(2→1)     | [4]
  2  | [5]        | 5    | 0(1→0), 2(1→0)     | [4, 5]
  3  | [0, 2]     | 0    | (none)               | [4, 5, 0]
  4  | [2]        | 2    | 3(1→0)              | [4, 5, 0, 2]
  5  | [3]        | 3    | 1(1→0)              | [4, 5, 0, 2, 3]
  6  | [1]        | 1    | (none)               | [4, 5, 0, 2, 3, 1]

result.size() = 6 = V → No cycle ✅
Topological Order: [4, 5, 0, 2, 3, 1]
```

### DFS-based Topological Sort
```java
// Add to stack AFTER processing all neighbors (post-order)
public List<Integer> topoSortDFS(int V, List<List<Integer>> adj) {
    boolean[] visited = new boolean[V];
    Stack<Integer> stack = new Stack<>();
    
    for (int i = 0; i < V; i++) {
        if (!visited[i]) {
            dfsTopo(i, adj, visited, stack);
        }
    }
    
    List<Integer> result = new ArrayList<>();
    while (!stack.isEmpty()) result.add(stack.pop());
    return result;
}

private void dfsTopo(int node, List<List<Integer>> adj, boolean[] visited, Stack<Integer> stack) {
    visited[node] = true;
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            dfsTopo(neighbor, adj, visited, stack);
        }
    }
    stack.push(node);  // Push AFTER all descendants are processed
}
```

---

## 7. Course Schedule I & II

### Course Schedule I (LC 207): Can you finish all courses?
Just check if topological sort is possible (no cycle).

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    
    for (int[] pre : prerequisites) {
        adj.get(pre[1]).add(pre[0]);  // pre[1] → pre[0]
        inDegree[pre[0]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) queue.add(i);
    }
    
    int count = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        count++;
        for (int neighbor : adj.get(node)) {
            if (--inDegree[neighbor] == 0) queue.add(neighbor);
        }
    }
    
    return count == numCourses;  // If all courses processed → no cycle
}
```

### Course Schedule II (LC 210): Return the order
Same as above, but collect the order in a result array.

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    // Same Kahn's algorithm, but return result array
    // If count != numCourses, return empty array (cycle exists)
    List<List<Integer>> adj = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    for (int[] pre : prerequisites) {
        adj.get(pre[1]).add(pre[0]);
        inDegree[pre[0]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) queue.add(i);
    }
    
    int[] result = new int[numCourses];
    int idx = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        result[idx++] = node;
        for (int neighbor : adj.get(node)) {
            if (--inDegree[neighbor] == 0) queue.add(neighbor);
        }
    }
    
    return idx == numCourses ? result : new int[0];
}
```

---

## 8. Shortest Path Algorithms

### Dijkstra's Algorithm
Finds the shortest path from Source to all other nodes in a **Weighted Graph (non-negative weights)**.
*   **Data Structure:** `PriorityQueue` (Min-Heap).
*   **Time:** $O((V + E) \log V)$

```java
public int[] dijkstra(int V, List<List<int[]>> adj, int source) {
    int[] dist = new int[V];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;
    
    // Min-heap: {distance, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.add(new int[]{0, source});
    
    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int d = curr[0], u = curr[1];
        
        if (d > dist[u]) continue;  // Skip outdated entries
        
        for (int[] edge : adj.get(u)) {
            int v = edge[0], weight = edge[1];
            
            // Relaxation
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.add(new int[]{dist[v], v});
            }
        }
    }
    return dist;
}
```

### 🧠 Dijkstra Dry Run

```
Graph:
  0 --1-- 1 --3-- 3
  |       |       |
  4       2       1
  |       |       |
  2 --5-- 4 ------+

Edges: 0→1(1), 0→2(4), 1→3(3), 1→4(2), 2→4(5), 3→4(1)
Source: 0

Step | PQ (dist,node)     | Poll     | Relax neighbors        | dist[]
-----|---------------------|----------|------------------------|-----------
Init | [(0,0)]             |          |                        | [0,∞,∞,∞,∞]
  1  | [(0,0)]             | (0,0)    | 1: 0+1=1, 2: 0+4=4    | [0,1,4,∞,∞]
  2  | [(1,1),(4,2)]       | (1,1)    | 3: 1+3=4, 4: 1+2=3    | [0,1,4,4,3]
  3  | [(3,4),(4,2),(4,3)] | (3,4)    | 3: 3+1=4 (no better)  | [0,1,4,4,3]
  4  | [(4,2),(4,3)]       | (4,2)    | 4: 4+5=9 (no better)  | [0,1,4,4,3]
  5  | [(4,3)]             | (4,3)    | (no unprocessed)       | [0,1,4,4,3]

Final dist[] = [0, 1, 4, 4, 3]
Shortest path to node 4: 0→1→4 (cost: 3) ✅
```

### Bellman-Ford
Finds shortest path from Source to all nodes. **Handles negative weights! Detects negative cycles!**

Logic: Relax ALL edges exactly `V - 1` times. If we can still relax on the V-th iteration, there's a negative cycle.

```java
public int[] bellmanFord(int V, int[][] edges, int source) {
    int[] dist = new int[V];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;
    
    // Relax all edges V-1 times
    for (int i = 0; i < V - 1; i++) {
        for (int[] edge : edges) {
            int u = edge[0], v = edge[1], w = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }
    
    // Check for negative weight cycle (V-th iteration)
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
            throw new RuntimeException("Negative weight cycle detected!");
        }
    }
    
    return dist;
}
```

### 🧠 Bellman-Ford Dry Run

```
Edges: (0→1, 4), (0→2, 5), (1→2, -3), (2→3, 4)
V=4, Source=0

Iteration | Edge      | dist[] before         | Relax?                  | dist[] after
----------|-----------|----------------------|-------------------------|-------------
    1     | (0→1, 4) | [0, ∞, ∞, ∞]         | 0+4 < ∞ → YES          | [0, 4, ∞, ∞]
    1     | (0→2, 5) | [0, 4, ∞, ∞]         | 0+5 < ∞ → YES          | [0, 4, 5, ∞]
    1     | (1→2,-3) | [0, 4, 5, ∞]         | 4+(-3)=1 < 5 → YES     | [0, 4, 1, ∞]
    1     | (2→3, 4) | [0, 4, 1, ∞]         | 1+4=5 < ∞ → YES        | [0, 4, 1, 5]
    2     | All edges | [0, 4, 1, 5]         | No further relaxation   | [0, 4, 1, 5]
    3     | All edges | Same                  | No change               | Same

V-th iteration: No relaxation possible → No negative cycle ✅
Final: [0, 4, 1, 5]
```

### Floyd-Warshall
Finds shortest path from **EVERY node to EVERY OTHER node**.
Logic: 3 nested loops — try inserting every possible intermediate node `k` between `i` and `j`.

```java
public int[][] floydWarshall(int V, int[][] graph) {
    int[][] dist = new int[V][V];
    int INF = (int) 1e9;
    
    // Initialize
    for (int i = 0; i < V; i++) {
        for (int j = 0; j < V; j++) {
            if (i == j) dist[i][j] = 0;
            else if (graph[i][j] != 0) dist[i][j] = graph[i][j];
            else dist[i][j] = INF;
        }
    }
    
    // Try every node k as intermediate
    for (int k = 0; k < V; k++) {
        for (int i = 0; i < V; i++) {
            for (int j = 0; j < V; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    return dist;
}
```

### Shortest Path Algorithm Comparison

| Algorithm | Handles Negatives | All-Pairs | Time | Use When |
|:---|:---:|:---:|:---:|:---|
| **BFS** | No weights | No | O(V+E) | Unweighted graph |
| **Dijkstra** | No ❌ | No | O((V+E)logV) | Non-negative weighted |
| **Bellman-Ford** | Yes ✅ | No | O(V×E) | Negative weights / K stops |
| **Floyd-Warshall** | Yes ✅ | Yes ✅ | O(V³) | Small graph, all-pairs |

---

## 9. Word Ladder

**Problem (LC 127):** Transform `beginWord` to `endWord`, changing one letter at a time. Each intermediate word must be in `wordList`. Find minimum transformations.

Example: "hit" → "hot" → "dot" → "dog" → "cog" = **5 words (4 transformations)**

### 🧠 How to Think
Each word = a node. Two words are connected if they differ by exactly 1 letter. Find **shortest path** = BFS!

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> dict = new HashSet<>(wordList);
    if (!dict.contains(endWord)) return 0;
    
    Queue<String> queue = new LinkedList<>();
    queue.add(beginWord);
    dict.remove(beginWord);
    int level = 1;
    
    while (!queue.isEmpty()) {
        int size = queue.size();
        
        for (int i = 0; i < size; i++) {
            String word = queue.poll();
            char[] chars = word.toCharArray();
            
            // Try changing each position to every letter a-z
            for (int j = 0; j < chars.length; j++) {
                char original = chars[j];
                
                for (char c = 'a'; c <= 'z'; c++) {
                    if (c == original) continue;
                    chars[j] = c;
                    String newWord = new String(chars);
                    
                    if (newWord.equals(endWord)) return level + 1;
                    
                    if (dict.contains(newWord)) {
                        queue.add(newWord);
                        dict.remove(newWord);  // Remove to avoid revisiting
                    }
                }
                chars[j] = original;  // Restore
            }
        }
        level++;
    }
    return 0;  // No transformation possible
}
```

---

## 10. Clone Graph

**Problem (LC 133):** Deep copy a graph. Each node has `val` and `List<Node> neighbors`.

### 🧠 How to Think
Use a `HashMap<Node, Node>` to map original → clone. DFS/BFS and clone as you go.

```java
public Node cloneGraph(Node node) {
    if (node == null) return null;
    
    Map<Node, Node> map = new HashMap<>();
    return dfsClone(node, map);
}

private Node dfsClone(Node node, Map<Node, Node> map) {
    if (map.containsKey(node)) return map.get(node);  // Already cloned
    
    Node clone = new Node(node.val);
    map.put(node, clone);
    
    for (Node neighbor : node.neighbors) {
        clone.neighbors.add(dfsClone(neighbor, map));
    }
    return clone;
}
```

---

## 11. Pacific Atlantic Water Flow

**Problem (LC 417):** Given island heightmap. Water flows from higher to lower or equal height. Pacific ocean is on top/left edges. Atlantic ocean is on bottom/right edges. Find cells where water can flow to BOTH oceans.

### 🧠 How to Think (Reverse Thinking!)
Instead of checking "can this cell reach both oceans" (hard), think in reverse:
- Start DFS/BFS from Pacific border cells → mark all cells reachable by Pacific
- Start DFS/BFS from Atlantic border cells → mark all cells reachable by Atlantic
- Answer = intersection of both sets

```java
public List<List<Integer>> pacificAtlantic(int[][] heights) {
    int m = heights.length, n = heights[0].length;
    boolean[][] pacific = new boolean[m][n];
    boolean[][] atlantic = new boolean[m][n];
    
    // DFS from Pacific border (top row + left column)
    for (int j = 0; j < n; j++) dfs(heights, pacific, 0, j, 0);
    for (int i = 0; i < m; i++) dfs(heights, pacific, i, 0, 0);
    
    // DFS from Atlantic border (bottom row + right column)
    for (int j = 0; j < n; j++) dfs(heights, atlantic, m-1, j, 0);
    for (int i = 0; i < m; i++) dfs(heights, atlantic, i, n-1, 0);
    
    // Find intersection
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (pacific[i][j] && atlantic[i][j]) {
                result.add(Arrays.asList(i, j));
            }
        }
    }
    return result;
}

private void dfs(int[][] heights, boolean[][] visited, int i, int j, int prevHeight) {
    if (i < 0 || i >= heights.length || j < 0 || j >= heights[0].length 
        || visited[i][j] || heights[i][j] < prevHeight) return;
    
    visited[i][j] = true;
    dfs(heights, visited, i+1, j, heights[i][j]);
    dfs(heights, visited, i-1, j, heights[i][j]);
    dfs(heights, visited, i, j+1, heights[i][j]);
    dfs(heights, visited, i, j-1, heights[i][j]);
}
```

---

## 12. Is Graph Bipartite?

**Problem (LC 785):** Can you color the graph with 2 colors such that no two adjacent nodes have the same color?

### 🧠 How to Think
BFS/DFS with 2-coloring. Assign color 0 to start node. All neighbors get color 1. Their neighbors get color 0. If you find a neighbor already colored with the SAME color → NOT bipartite.

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n];
    Arrays.fill(color, -1);  // -1 = uncolored
    
    for (int i = 0; i < n; i++) {
        if (color[i] == -1) {
            // BFS from this node
            Queue<Integer> queue = new LinkedList<>();
            queue.add(i);
            color[i] = 0;
            
            while (!queue.isEmpty()) {
                int node = queue.poll();
                for (int neighbor : graph[node]) {
                    if (color[neighbor] == -1) {
                        color[neighbor] = 1 - color[node];  // Opposite color
                        queue.add(neighbor);
                    } else if (color[neighbor] == color[node]) {
                        return false;  // Same color → NOT bipartite!
                    }
                }
            }
        }
    }
    return true;
}
```

---

## 13. Cheapest Flights Within K Stops

**Problem (LC 787):** Find cheapest flight from `src` to `dst` with at most `k` stops. 

### 🧠 How to Think
Use **modified Bellman-Ford** with exactly `k+1` iterations (k stops = k+1 edges).

```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    
    // Relax exactly k+1 times (k stops = k+1 edges)
    for (int i = 0; i <= k; i++) {
        int[] temp = dist.clone();  // Use COPY to avoid cascading updates
        
        for (int[] flight : flights) {
            int u = flight[0], v = flight[1], w = flight[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < temp[v]) {
                temp[v] = dist[u] + w;
            }
        }
        dist = temp;
    }
    
    return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
}
```

---

## 14. Alien Dictionary

**Problem (LC 269):** Given a sorted list of words from an alien language, derive the order of characters.

### 🧠 How to Think
Compare adjacent words to find ordering rules → Build a directed graph → Topological Sort!

```java
public String alienOrder(String[] words) {
    Map<Character, Set<Character>> adj = new HashMap<>();
    Map<Character, Integer> inDegree = new HashMap<>();
    
    // Initialize all characters
    for (String word : words) {
        for (char c : word.toCharArray()) {
            adj.putIfAbsent(c, new HashSet<>());
            inDegree.putIfAbsent(c, 0);
        }
    }
    
    // Compare adjacent words to find edges
    for (int i = 0; i < words.length - 1; i++) {
        String w1 = words[i], w2 = words[i + 1];
        
        // Edge case: "abc" before "ab" → invalid
        if (w1.length() > w2.length() && w1.startsWith(w2)) return "";
        
        for (int j = 0; j < Math.min(w1.length(), w2.length()); j++) {
            if (w1.charAt(j) != w2.charAt(j)) {
                // w1[j] comes BEFORE w2[j]
                if (adj.get(w1.charAt(j)).add(w2.charAt(j))) {
                    inDegree.merge(w2.charAt(j), 1, Integer::sum);
                }
                break;  // Only first difference matters!
            }
        }
    }
    
    // Kahn's topological sort
    Queue<Character> queue = new LinkedList<>();
    for (char c : inDegree.keySet()) {
        if (inDegree.get(c) == 0) queue.add(c);
    }
    
    StringBuilder result = new StringBuilder();
    while (!queue.isEmpty()) {
        char c = queue.poll();
        result.append(c);
        for (char neighbor : adj.get(c)) {
            inDegree.merge(neighbor, -1, Integer::sum);
            if (inDegree.get(neighbor) == 0) queue.add(neighbor);
        }
    }
    
    return result.length() == inDegree.size() ? result.toString() : "";
}
```

---

## 15. How to Identify Graph Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│                  GRAPH PATTERN RECOGNITION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Shortest path / min steps" (Unweighted) → BFS                │
│  "Shortest path" (Weighted, no neg)       → Dijkstra           │
│  "Shortest path" (Negative weights)       → Bellman-Ford       │
│  "Shortest path within K stops"           → Modified B-Ford    │
│  "All-pairs shortest path"                → Floyd-Warshall     │
│                                                                  │
│  "2D matrix: islands, mazes, grids"       → Implicit Graph     │
│     Each cell = node, Up/Down/Left/Right = edges                │
│                                                                  │
│  "Prerequisites / Dependencies / Order"   → Topological Sort   │
│  "Can we finish? / Deadlock?"             → Cycle Detection    │
│  "Connect all with minimum cost"          → MST (Kruskal/Prim) │
│  "Groups / Is A connected to B?"          → DSU or BFS/DFS     │
│  "2-colorable / bipartite"                → BFS 2-coloring     │
│  "Transform X to Y (min steps)"          → BFS (Word Ladder)  │
│  "Copy/Clone a structure"                 → DFS + HashMap      │
│                                                                  │
│  MULTI-SOURCE BFS:                                               │
│  "Rotten oranges / fire spreading"        → Start BFS from ALL │
│                                              sources at once     │
│                                                                  │
│  REVERSE DFS/BFS:                                                │
│  "Can water reach ocean?"                 → Start from border   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 16. Top 20 Must-Do Graph Questions

**BFS/DFS on Grids:**
1. Number of Islands (LC 200)
2. Max Area of Island (LC 695)
3. Rotting Oranges (LC 994)
4. Surrounded Regions (LC 130)
5. Pacific Atlantic Water Flow (LC 417)
6. Flood Fill (LC 733)

**Graph Traversal & Components:**
7. Clone Graph (LC 133)
8. Number of Connected Components (LC 323)
9. Is Graph Bipartite? (LC 785)

**Cycle Detection & Topological Sort:**
10. Course Schedule I (LC 207)
11. Course Schedule II (LC 210)
12. Alien Dictionary (LC 269)

**Shortest Path:**
13. Network Delay Time — Dijkstra (LC 743)
14. Cheapest Flights Within K Stops — Bellman-Ford (LC 787)
15. Word Ladder (LC 127)
16. Shortest Path in Binary Matrix (LC 1091)

**DSU (Union-Find):**
17. Redundant Connection (LC 684)
18. Accounts Merge (LC 721)
19. Min Cost to Connect All Points — Kruskal (LC 1584)

**Advanced:**
20. Swim in Rising Water (LC 778)
