# 🔗 Disjoint Set Union (DSU) & Minimum Spanning Tree (MST) — FAANG Masterclass

> **"If you want to know if two people are in the same friend group, don't ask all their friends. Just ask the leader of the group."**

---

## 📑 Table of Contents
1. [What is Disjoint Set Union (DSU)?](#1-what-is-disjoint-set-union-dsu)
2. [The Two Magic Functions: Find & Union](#2-the-two-magic-functions-find--union)
3. [Optimization: Path Compression & Union by Rank](#3-optimization-path-compression--union-by-rank)
4. [DSU Dry Run (Step by Step)](#4-dsu-dry-run)
5. [Minimum Spanning Tree (MST)](#5-minimum-spanning-tree-mst)
6. [Kruskal's Algorithm (Using DSU) — Full Dry Run](#6-kruskals-algorithm)
7. [Prim's Algorithm (Using Min-Heap) — Full Code & Dry Run](#7-prims-algorithm)
8. [Kruskal vs Prim Comparison](#8-kruskal-vs-prim-comparison)
9. [Problem: Number of Provinces (LC 547)](#9-number-of-provinces)
10. [Problem: Redundant Connection (LC 684)](#10-redundant-connection)
11. [Problem: Min Cost to Connect All Points (LC 1584)](#11-min-cost-to-connect-all-points)
12. [Problem: Accounts Merge (LC 721)](#12-accounts-merge)
13. [Problem: Swim in Rising Water (LC 778)](#13-swim-in-rising-water)
14. [DSU with Size vs Rank](#14-dsu-with-size-vs-rank)
15. [When to Use What?](#15-when-to-use-what)
16. [Top 10 Must-Do DSU/MST Questions](#16-top-10-must-do-dsumst-questions)

---

## 1. What is Disjoint Set Union (DSU)?

Also known as the **Union-Find** data structure.
It solves a very specific problem extremely fast: **Dynamic Connectivity**.

Imagine a graph where edges are constantly being added. You need to answer: *"Are Node A and Node B connected (in the same component)?"*
*   Doing a BFS/DFS every time takes $O(V + E)$. Too slow!
*   DSU answers this in **$O(\alpha(V))$**, which is effectively **$O(1)$** (Constant time!).

---

## 2. The Two Magic Functions: Find & Union

DSU maintains a collection of disjoint sets. Every set has exactly one **Ultimate Parent (Leader/Root)**.

1.  **`Find(x)`:** Returns the Ultimate Parent of `x`. If `Find(A) == Find(B)`, they are in the same set/component.
2.  **`Union(x, y)`:** Connects the set containing `x` with the set containing `y`. (Makes the leader of one point to the leader of the other).

### The Basic Implementation (Slow — Understand First)
```java
class DSU {
    int[] parent;

    public DSU(int size) {
        parent = new int[size];
        for (int i = 0; i < size; i++) parent[i] = i; // Everyone is their own leader
    }

    public int find(int i) {
        if (parent[i] == i) return i; // I am the leader!
        return find(parent[i]); // Keep climbing up
    }

    public void union(int i, int j) {
        int rootI = find(i);
        int rootJ = find(j);
        if (rootI != rootJ) {
            parent[rootI] = rootJ; // Connect them
        }
    }
}
```
*Problem:* If the tree becomes a long chain (like a linked list), `Find` takes $O(N)$.

---

## 3. Optimization: Path Compression & Union by Rank

We can optimize DSU to run in near $O(1)$ time with two tricks:

**Path Compression:** While finding the ultimate parent, make every node on the path point DIRECTLY to the ultimate parent. This flattens the tree.

**Union by Rank:** Always attach the smaller tree under the root of the larger tree. This prevents the tree from getting too tall.

### The Master DSU Implementation (MEMORIZE THIS!)
```java
class DSU {
    int[] parent;
    int[] rank;

    public DSU(int size) {
        parent = new int[size];
        rank = new int[size];
        for (int i = 0; i < size; i++) {
            parent[i] = i;
            rank[i] = 0;
        }
    }

    // Path Compression: make every node point directly to root
    public int find(int i) {
        if (parent[i] == i) return i;
        return parent[i] = find(parent[i]); // ← This is the magic line!
    }

    // Union by Rank: attach smaller tree under larger tree
    public boolean union(int i, int j) {
        int rootI = find(i);
        int rootJ = find(j);
        
        if (rootI == rootJ) return false; // Already connected (CYCLE!)

        if (rank[rootI] > rank[rootJ]) {
            parent[rootJ] = rootI;
        } else if (rank[rootI] < rank[rootJ]) {
            parent[rootI] = rootJ;
        } else {
            parent[rootJ] = rootI;
            rank[rootI]++;
        }
        return true;
    }
}
```

---

## 4. DSU Dry Run

### 🧠 Step-by-step walkthrough

```
Initial State (5 nodes):
parent[] = [0, 1, 2, 3, 4]   ← Everyone is their own parent
rank[]   = [0, 0, 0, 0, 0]

Visual: {0} {1} {2} {3} {4}  ← 5 separate sets


═══ UNION(0, 1) ═══
find(0) = 0, find(1) = 1 → Different roots → MERGE
rank[0]==rank[1]==0 → Attach 1 under 0, rank[0]++
parent[] = [0, 0, 2, 3, 4]
rank[]   = [1, 0, 0, 0, 0]

Visual: {0←1} {2} {3} {4}


═══ UNION(2, 3) ═══
find(2) = 2, find(3) = 3 → Different roots → MERGE
parent[] = [0, 0, 2, 2, 4]
rank[]   = [1, 0, 1, 0, 0]

Visual: {0←1} {2←3} {4}


═══ UNION(1, 3) ═══
find(1): parent[1]=0, parent[0]=0 → root = 0
find(3): parent[3]=2, parent[2]=2 → root = 2
0 ≠ 2 → Different roots → MERGE
rank[0]==rank[2]==1 → Attach 2 under 0, rank[0]++
parent[] = [0, 0, 0, 2, 4]    ← parent[2] = 0
rank[]   = [2, 0, 1, 0, 0]

Visual:
    0
   / \
  1   2
      |
      3


═══ FIND(3) with Path Compression ═══
find(3): parent[3]=2, not root
  find(2): parent[2]=0, not root
    find(0): parent[0]=0, IS root → return 0
  parent[2] = 0 (already 0)
parent[3] = 0 ← PATH COMPRESSION! Direct link!

After: parent[] = [0, 0, 0, 0, 4]

Visual (after path compression):
    0
   /|\
  1 2 3  ← 3 now points directly to 0!


═══ UNION(0, 4) ═══
find(0) = 0, find(4) = 4 → MERGE
rank[0]=2 > rank[4]=0 → Attach 4 under 0
parent[] = [0, 0, 0, 0, 0]

Visual: All connected!
      0
    / |\ \
   1  2 3  4


═══ FIND(1) == FIND(4)? ═══
find(1) = 0, find(4) = 0 → YES, connected! ✅
```

---

## 5. Minimum Spanning Tree (MST)

**Scenario:** You have 5 cities, and you want to lay fiber-optic cables to connect all of them so that any city can reach any other city. You want to spend the **minimum amount of money**.

A Spanning Tree is a subgraph that:
1. Contains all $V$ vertices.
2. Has exactly $V - 1$ edges.
3. Has NO cycles.

The **Minimum Spanning Tree (MST)** is the spanning tree with the minimum possible total edge weight.

```
Graph:
    0 ---4--- 1
    |  \      |
    8   8     11
    |    \    |
    2 ---7--- 3
    |  \
    2    6
    |     \
    4 --9-- 5

MST (pick cheapest edges without cycles):
    0 ---4--- 1
    |         
    8         
    |         
    2 ---7--- 3
    |  
    2    
    |     
    4 

Total cost = 4 + 8 + 7 + 2 = 21
```

---

## 6. Kruskal's Algorithm

Kruskal's is a **Greedy Algorithm** that uses DSU.

**Logic:**
1. Sort all edges from smallest weight to largest.
2. Iterate through sorted edges.
3. For each edge `(u, v, weight)`:
   * Use DSU: if `Find(u) != Find(v)`, take the edge (`Union(u, v)`).
   * If `Find(u) == Find(v)`, skip (would create a cycle).
4. Stop when we have $V - 1$ edges.

```java
public int kruskalMST(int V, int[][] edges) {
    // 1. Sort edges by weight
    Arrays.sort(edges, (a, b) -> a[2] - b[2]);
    
    DSU dsu = new DSU(V);
    int totalCost = 0;
    int edgesTaken = 0;
    
    // 2. Iterate and greedily pick
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], weight = edge[2];
        
        if (dsu.union(u, v)) { // Returns true if no cycle
            totalCost += weight;
            edgesTaken++;
            if (edgesTaken == V - 1) break; // MST complete!
        }
    }
    return totalCost;
}
```

### 🧠 Kruskal's Dry Run

```
Edges (sorted by weight):
  (2,4,2), (0,1,4), (2,5,6), (2,3,7), (0,2,8), (0,3,8), (4,5,9), (1,3,11)

V = 6, need V-1 = 5 edges.

Step | Edge    | Weight | find(u)==find(v)? | Action        | DSU Components        | Cost
-----|---------|--------|-------------------|---------------|----------------------|-----
  1  | (2,4)   | 2      | find(2)≠find(4)   | TAKE ✅ Union | {0}{1}{2,4}{3}{5}     | 2
  2  | (0,1)   | 4      | find(0)≠find(1)   | TAKE ✅ Union | {0,1}{2,4}{3}{5}      | 6
  3  | (2,5)   | 6      | find(2)≠find(5)   | TAKE ✅ Union | {0,1}{2,4,5}{3}       | 12
  4  | (2,3)   | 7      | find(2)≠find(3)   | TAKE ✅ Union | {0,1}{2,3,4,5}        | 19
  5  | (0,2)   | 8      | find(0)≠find(2)   | TAKE ✅ Union | {0,1,2,3,4,5}         | 27
  6  | (0,3)   | 8      | find(0)==find(3)   | SKIP ❌ Cycle |                       | —
  
5 edges taken → MST complete!
MST edges: (2,4), (0,1), (2,5), (2,3), (0,2)
Total cost: 27 ✅
```

---

## 7. Prim's Algorithm

Prim's builds the MST organically from a starting node using a **Min-Heap**.

**Logic:**
1. Start at any node (e.g., node 0). Mark as visited.
2. Add all its edges to a `PriorityQueue` (Min-Heap).
3. While PQ is not empty:
   * Pop the smallest weight edge.
   * If destination already visited → skip.
   * Otherwise, mark visited, add weight to total, push all edges of new node.

```java
public int primMST(int V, List<List<int[]>> adj) {
    boolean[] visited = new boolean[V];
    // Min-heap: {weight, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.add(new int[]{0, 0}); // Start from node 0 with weight 0
    
    int totalCost = 0;
    int edgesTaken = 0;
    
    while (!pq.isEmpty() && edgesTaken < V) {
        int[] curr = pq.poll();
        int weight = curr[0], u = curr[1];
        
        if (visited[u]) continue; // Skip if already in MST
        
        visited[u] = true;
        totalCost += weight;
        edgesTaken++;
        
        // Add all edges of u to PQ
        for (int[] edge : adj.get(u)) {
            int v = edge[0], w = edge[1];
            if (!visited[v]) {
                pq.add(new int[]{w, v});
            }
        }
    }
    
    return totalCost;
}
```

### 🧠 Prim's Dry Run

```
Graph (same as Kruskal example):
Adjacency List:
  0: [(1,4), (2,8), (3,8)]
  1: [(0,4), (3,11)]
  2: [(0,8), (3,7), (4,2), (5,6)]
  3: [(0,8), (1,11), (2,7)]
  4: [(2,2), (5,9)]
  5: [(2,6), (4,9)]

Start from node 0.

Step | PQ (weight,node)              | Poll    | Visited? | Action       | MST Cost
-----|-------------------------------|---------|----------|--------------|--------
Init | [(0,0)]                       |         |          |              | 0
  1  | [(0,0)]                       | (0,0)   | No→Visit | Add 0's edges| 0
     | PQ: [(4,1),(8,2),(8,3)]       |         |          |              |
  2  | [(4,1),(8,2),(8,3)]            | (4,1)   | No→Visit | Add 1's edges| 4
     | PQ: [(8,2),(8,3),(11,3)]      |         |          |              |
  3  | [(8,2),(8,3),(11,3)]           | (8,2)   | No→Visit | Add 2's edges| 12
     | PQ: [(2,4),(6,5),(7,3),(8,3),(11,3)] |   |          |              |
  4  | [(2,4),(6,5),(7,3)...]         | (2,4)   | No→Visit | Add 4's edges| 14
     | PQ: [(6,5),(7,3),(8,3),(9,5),(11,3)] |   |          |              |
  5  | [(6,5),(7,3)...]               | (6,5)   | No→Visit | Add 5's edges| 20
     | PQ: [(7,3),(8,3),(9,5),(11,3)] |         |          |              |
  6  | [(7,3),(8,3)...]               | (7,3)   | No→Visit | Done!        | 27

Total MST Cost: 27 ✅ (same as Kruskal!)
```

---

## 8. Kruskal vs Prim Comparison

| Feature | Kruskal's | Prim's |
|:---|:---|:---|
| **Approach** | Edge-centric (sort edges) | Vertex-centric (grow from node) |
| **Data Structure** | DSU (Union-Find) | Min-Heap (PriorityQueue) |
| **Time Complexity** | $O(E \log E)$ | $O(E \log V)$ |
| **Best for** | **Sparse graphs** (few edges) | **Dense graphs** (many edges) |
| **Input format** | Edge list | Adjacency list |
| **Easier to code?** | ✅ Yes (in interviews) | Slightly more complex |
| **Handles disconnected?** | Naturally (just won't get V-1 edges) | Need to check all components |

**Interview tip:** Kruskal's + DSU is usually easier and faster to code in interviews.

---

## 9. Number of Provinces (LC 547)

**Problem:** Given `n` cities and an adjacency matrix `isConnected`, find the number of provinces (connected components).

### 🧠 How to Think
This is just "Count Connected Components" → Use DSU!

```java
public int findCircleNum(int[][] isConnected) {
    int n = isConnected.length;
    DSU dsu = new DSU(n);
    int provinces = n;  // Start with n separate provinces
    
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (isConnected[i][j] == 1) {
                if (dsu.union(i, j)) {
                    provinces--;  // Merged two components
                }
            }
        }
    }
    return provinces;
}
```

### 🧠 Dry Run
```
isConnected = [[1,1,0],
               [1,1,0],
               [0,0,1]]

Start: provinces = 3, DSU = {0} {1} {2}

i=0, j=1: connected → union(0,1) → success → provinces = 2
i=0, j=2: not connected → skip
i=1, j=2: not connected → skip

Answer: 2 provinces → {0,1} and {2} ✅
```

---

## 10. Redundant Connection (LC 684)

**Problem:** A tree has one extra edge added, creating a cycle. Find and return that edge.

### 🧠 How to Think
Process edges one by one with DSU. The first edge where `Find(u) == Find(v)` (already connected) is the redundant edge creating the cycle!

```java
public int[] findRedundantConnection(int[][] edges) {
    int n = edges.length;
    DSU dsu = new DSU(n + 1); // 1-indexed
    
    for (int[] edge : edges) {
        if (!dsu.union(edge[0], edge[1])) {
            return edge; // This edge creates a cycle!
        }
    }
    return new int[0]; // Should never reach
}
```

### 🧠 Dry Run
```
edges = [[1,2], [1,3], [2,3]]

Process [1,2]: find(1)≠find(2) → union → DSU: {1,2} {3}
Process [1,3]: find(1)≠find(3) → union → DSU: {1,2,3}
Process [2,3]: find(2)==find(3)==1 → CYCLE! Return [2,3] ✅
```

---

## 11. Min Cost to Connect All Points (LC 1584)

**Problem:** Given `points`, connect all with minimum cost. Cost between two points = Manhattan distance.

### 🧠 How to Think
Generate all possible edges with their Manhattan distance → Run Kruskal's MST!

```java
public int minCostConnectPoints(int[][] points) {
    int n = points.length;
    
    // Generate all edges
    List<int[]> edges = new ArrayList<>();
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            int dist = Math.abs(points[i][0] - points[j][0]) 
                     + Math.abs(points[i][1] - points[j][1]);
            edges.add(new int[]{i, j, dist});
        }
    }
    
    // Kruskal's
    edges.sort((a, b) -> a[2] - b[2]);
    DSU dsu = new DSU(n);
    int totalCost = 0, edgesTaken = 0;
    
    for (int[] edge : edges) {
        if (dsu.union(edge[0], edge[1])) {
            totalCost += edge[2];
            if (++edgesTaken == n - 1) break;
        }
    }
    return totalCost;
}
```

---

## 12. Accounts Merge (LC 721)

**Problem:** Given accounts where each account has a name and emails, merge accounts that share any email. Return merged accounts.

### 🧠 How to Think
Two accounts belong to the same person if they share any email → DSU!
- Map each email to the first account index that owns it.
- If an email already has an owner, union the current account with that owner.

```java
public List<List<String>> accountsMerge(List<List<String>> accounts) {
    int n = accounts.size();
    DSU dsu = new DSU(n);
    Map<String, Integer> emailToId = new HashMap<>(); // email → account index
    
    // Step 1: Union accounts that share emails
    for (int i = 0; i < n; i++) {
        for (int j = 1; j < accounts.get(i).size(); j++) {
            String email = accounts.get(i).get(j);
            
            if (emailToId.containsKey(email)) {
                dsu.union(i, emailToId.get(email)); // Same email → merge accounts
            } else {
                emailToId.put(email, i);
            }
        }
    }
    
    // Step 2: Group emails by root account
    Map<Integer, TreeSet<String>> rootToEmails = new HashMap<>();
    for (Map.Entry<String, Integer> entry : emailToId.entrySet()) {
        int root = dsu.find(entry.getValue());
        rootToEmails.computeIfAbsent(root, k -> new TreeSet<>()).add(entry.getKey());
    }
    
    // Step 3: Build result
    List<List<String>> result = new ArrayList<>();
    for (Map.Entry<Integer, TreeSet<String>> entry : rootToEmails.entrySet()) {
        List<String> merged = new ArrayList<>();
        merged.add(accounts.get(entry.getKey()).get(0)); // Name
        merged.addAll(entry.getValue()); // Sorted emails
        result.add(merged);
    }
    return result;
}
```

### 🧠 Dry Run
```
accounts = [
  ["John", "john1@mail.com", "john_shared@mail.com"],   // 0
  ["John", "john2@mail.com", "john_shared@mail.com"],   // 1
  ["Mary", "mary@mail.com"]                             // 2
]

Processing account 0:
  "john1@mail.com" → new, emailToId = {john1→0}
  "john_shared@mail.com" → new, emailToId = {john1→0, john_shared→0}

Processing account 1:
  "john2@mail.com" → new, emailToId = {..., john2→1}
  "john_shared@mail.com" → ALREADY EXISTS (owner=0) → union(1, 0)!
  DSU: {0,1} {2}

Processing account 2:
  "mary@mail.com" → new, emailToId = {..., mary→2}

Group by root:
  root 0: ["john1@mail.com", "john2@mail.com", "john_shared@mail.com"]
  root 2: ["mary@mail.com"]

Result: [
  ["John", "john1@mail.com", "john2@mail.com", "john_shared@mail.com"],
  ["Mary", "mary@mail.com"]
] ✅
```

---

## 13. Swim in Rising Water (LC 778)

**Problem:** Grid of elevations. At time `t`, you can swim to any cell with elevation ≤ t. Find minimum time to swim from (0,0) to (n-1, n-1).

### 🧠 How to Think (Binary Search + DSU)
Binary search on the answer `t`. For each `t`, union all cells with elevation ≤ t. Check if (0,0) and (n-1,n-1) are connected.

**Alternative: Kruskal's approach** — Sort all cell pairs by max elevation, process in order.

```java
public int swimInWater(int[][] grid) {
    int n = grid.length;
    
    // Create list of all cells sorted by elevation
    int[][] cells = new int[n * n][3]; // {elevation, row, col}
    int idx = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            cells[idx++] = new int[]{grid[i][j], i, j};
        }
    }
    Arrays.sort(cells, (a, b) -> a[0] - b[0]);
    
    DSU dsu = new DSU(n * n);
    boolean[][] added = new boolean[n][n];
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    
    for (int[] cell : cells) {
        int elev = cell[0], r = cell[1], c = cell[2];
        added[r][c] = true;
        
        // Union with already-added neighbors
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < n && nc >= 0 && nc < n && added[nr][nc]) {
                dsu.union(r * n + c, nr * n + nc);
            }
        }
        
        // Check if start and end are connected
        if (dsu.find(0) == dsu.find(n * n - 1)) {
            return elev;
        }
    }
    return -1;
}
```

---

## 14. DSU with Size vs Rank

### Union by Rank (Track tree height)
```java
// rank = approximate height of tree
// Attach shorter tree under taller tree
if (rank[rootI] > rank[rootJ]) {
    parent[rootJ] = rootI;
} else if (rank[rootI] < rank[rootJ]) {
    parent[rootI] = rootJ;
} else {
    parent[rootJ] = rootI;
    rank[rootI]++;     // Only increase when equal
}
```

### Union by Size (Track component size)
```java
class DSU {
    int[] parent, size;
    
    public DSU(int n) {
        parent = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;    // Each component starts with size 1
        }
    }
    
    public int find(int i) {
        if (parent[i] == i) return i;
        return parent[i] = find(parent[i]);
    }
    
    public boolean union(int i, int j) {
        int rootI = find(i), rootJ = find(j);
        if (rootI == rootJ) return false;
        
        // Attach smaller component under larger
        if (size[rootI] < size[rootJ]) {
            parent[rootI] = rootJ;
            size[rootJ] += size[rootI];    // Update size!
        } else {
            parent[rootJ] = rootI;
            size[rootI] += size[rootJ];
        }
        return true;
    }
    
    public int getSize(int i) {
        return size[find(i)];   // Size of component containing i
    }
}
```

| Feature | Union by Rank | Union by Size |
|:---|:---|:---|
| Tracks | Tree height (approximate) | Component size (exact) |
| Use when | Don't need component sizes | Need to know "how many nodes in this group?" |
| Example | Cycle detection, MST | Social networks, group counting |

---

## 15. When to Use What?

**Use DSU when:**
1. Dynamic connectivity ("add edges over time, check if connected")
2. Cycle detection in undirected graph (Redundant Connection)
3. Connected components count
4. Kruskal's MST algorithm
5. Grouping/merging (Accounts Merge)

**Kruskal vs Prim:**
*   **Kruskal:** Better for **sparse** graphs. Edge list input. Easier to code.
*   **Prim:** Better for **dense** graphs. Adjacency list input.

**DSU vs BFS/DFS for components:**
*   **DSU:** Dynamic (edges added incrementally). O(α(N)) per query.
*   **BFS/DFS:** Static (graph fully known). O(V+E) per traversal.

```
┌────────────────────────────────────────────────────────────┐
│              DSU/MST PATTERN RECOGNITION                    │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  "Connect cities with minimum cost"   → MST (Kruskal/Prim)│
│  "Redundant edge / find cycle"        → DSU               │
│  "Number of components"               → DSU (or BFS/DFS)  │
│  "Merge groups sharing common item"   → DSU               │
│  "Dynamic: edges added, check path?"  → DSU               │
│  "Min time/threshold to connect"      → Sort + DSU        │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 16. Top 10 Must-Do DSU/MST Questions

1. Redundant Connection (LC 684) — Find cycle edge with DSU
2. Number of Provinces (LC 547) — Count connected components
3. Min Cost to Connect All Points (LC 1584) — Kruskal's MST
4. Accounts Merge (LC 721) — DSU with strings
5. Swim in Rising Water (LC 778) — Sort + DSU
6. Number of Connected Components (LC 323) — Basic DSU
7. Longest Consecutive Sequence (LC 128) — DSU (or HashSet)
8. Satisfiability of Equality Equations (LC 990) — DSU
9. Making a Large Island (LC 827) — DSU + flip one cell
10. Graph Valid Tree (LC 261) — DSU: V-1 edges + no cycle + connected
