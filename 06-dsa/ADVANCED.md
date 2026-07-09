# 🚀 Advanced DSA — Trie, Segment Tree, DSU, Math, Bit Manipulation & CP Tricks

> **"These advanced data structures and techniques separate competitive programmers and senior engineers from the rest. Master these and you'll handle any technical challenge."**

---

## 📑 Table of Contents

1. [Trie (Prefix Tree)](#1-trie-prefix-tree)
2. [Segment Tree](#2-segment-tree)
3. [Disjoint Set Union (DSU / Union-Find)](#3-disjoint-set-union-dsu--union-find)
4. [Advanced Dynamic Programming](#4-advanced-dynamic-programming)
5. [Math for Competitive Programming](#5-math-for-competitive-programming)
6. [Bit Manipulation](#6-bit-manipulation)
7. [CP Tricks & Templates](#7-cp-tricks--templates)
8. [Interview Questions (30+)](#8-interview-questions-30)

---

## 1. Trie (Prefix Tree)

### What is a Trie?

A Trie is a tree-like data structure for **storing strings** where each node represents a **single character**. It's extremely efficient for prefix-based operations.

```
Storing words: "apple", "app", "ape", "bat", "ball"

             root
            /    \
           a      b
          /        \
         p          a
        / \          \
       p   e(✓)      t(✓)
      /               |
     l                l(✓)
    /
   e(✓)

(✓) = marks end of a complete word

Key insight: shared PREFIXES share the same path!
  "apple" and "app" share the path a → p → p
  This saves memory compared to storing each word separately.
```

### Trie Node — Implementation

```java
class TrieNode {
    TrieNode[] children;   // 26 slots for a-z
    boolean isEndOfWord;   // Does a word END at this node?
    
    TrieNode() {
        children = new TrieNode[26];  // children[0] = 'a', children[1] = 'b', ...
        isEndOfWord = false;
    }
}
```

### Trie — Full Implementation with Line-by-Line Explanation

```java
class Trie {
    
    private TrieNode root;
    
    public Trie() {
        root = new TrieNode();  // Root is an empty node (represents "")
    }
    
    // ═══════════════════════════════════════════
    // INSERT a word into the Trie
    // Time: O(L) where L = length of word
    // ═══════════════════════════════════════════
    public void insert(String word) {
        TrieNode current = root;  // Start at the root
        
        for (char c : word.toCharArray()) {
            int index = c - 'a';  // Convert char to index: 'a'→0, 'b'→1, ..., 'z'→25
            
            // Does this child exist?
            if (current.children[index] == null) {
                current.children[index] = new TrieNode();  // Create new node
            }
            
            current = current.children[index];  // Move to child node
        }
        
        current.isEndOfWord = true;  // Mark the last character as end of word
    }
    
    // ═══════════════════════════════════════════
    // SEARCH for an exact word
    // Time: O(L)
    // ═══════════════════════════════════════════
    public boolean search(String word) {
        TrieNode node = findNode(word);  // Traverse to the last character
        return node != null && node.isEndOfWord;  // Must exist AND be end of word
        // "app" in a trie with "apple": node exists but isEndOfWord=false → false
        // Unless "app" was also inserted separately
    }
    
    // ═══════════════════════════════════════════
    // Check if any word STARTS WITH given prefix
    // Time: O(L)
    // ═══════════════════════════════════════════
    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;  // Just needs to exist (no endOfWord check)
    }
    
    // Helper: traverse to the node at end of given string
    private TrieNode findNode(String str) {
        TrieNode current = root;
        
        for (char c : str.toCharArray()) {
            int index = c - 'a';
            
            if (current.children[index] == null) {
                return null;  // Character doesn't exist in trie → not found
            }
            
            current = current.children[index];  // Move to next character
        }
        
        return current;  // Return the node at the end of the string
    }
}
```

### Dry Run: Insert & Search

```
═══ Insert "apple" ═══

Step 1: c='a', index=0
  root.children[0] is null → CREATE new node
  Move to children[0]
  
  root → [a]

Step 2: c='p', index=15
  [a].children[15] is null → CREATE new node
  Move to children[15]
  
  root → [a] → [p]

Step 3: c='p', index=15
  [p].children[15] is null → CREATE new node
  
  root → [a] → [p] → [p]

Step 4: c='l', index=11
  CREATE new node
  
  root → [a] → [p] → [p] → [l]

Step 5: c='e', index=4
  CREATE new node, mark isEndOfWord = true ✓
  
  root → [a] → [p] → [p] → [l] → [e✓]


═══ Insert "app" ═══

Step 1: c='a', index=0
  root.children[0] EXISTS → just move to it

Step 2: c='p', index=15
  [a].children[15] EXISTS → just move to it

Step 3: c='p', index=15
  [p].children[15] EXISTS → just move to it
  Mark isEndOfWord = true ✓

  root → [a] → [p] → [p✓] → [l] → [e✓]
  
  "app" and "apple" now SHARE the path a→p→p !


═══ Search "app" ═══
  Traverse: root → [a] → [p] → [p✓]
  Node exists AND isEndOfWord = true → return true ✅

═══ Search "ap" ═══
  Traverse: root → [a] → [p]
  Node exists BUT isEndOfWord = false → return false ❌

═══ startsWith "ap" ═══
  Traverse: root → [a] → [p]
  Node exists → return true ✅ (don't care about isEndOfWord)
```

### Trie — Advanced: Autocomplete / Find All Words with Prefix

```java
// Return ALL words that start with a given prefix
public List<String> autocomplete(String prefix) {
    List<String> results = new ArrayList<>();
    TrieNode node = findNode(prefix);  // Navigate to end of prefix
    
    if (node == null) return results;  // Prefix doesn't exist
    
    // DFS from this node to find all complete words
    dfs(node, new StringBuilder(prefix), results);
    return results;
}

private void dfs(TrieNode node, StringBuilder path, List<String> results) {
    if (node.isEndOfWord) {
        results.add(path.toString());  // Found a complete word!
    }
    
    // Try all 26 possible children
    for (int i = 0; i < 26; i++) {
        if (node.children[i] != null) {
            path.append((char) ('a' + i));      // Add this character
            dfs(node.children[i], path, results); // Go deeper
            path.deleteCharAt(path.length() - 1); // Backtrack
        }
    }
}
```

```
Dry Run: autocomplete("ap") on Trie with "apple", "app", "ape"

             root
            /
           a
          /
         p
        / \
       p   e(✓)     ← "ape"
      /
     l
    /
   e(✓)              ← "apple"
   
Navigate to "ap" → node at 'p'
DFS from 'p':
  → children[4] = 'e' → isEndOfWord ✓ → add "ape"
  → children[15] = 'p' → isEndOfWord ✓ → add "app"
    → children[11] = 'l'
      → children[4] = 'e' → isEndOfWord ✓ → add "apple"

Result: ["ape", "app", "apple"] ✅
```

### Trie: When to Use

| Use Case | Why Trie? |
|----------|-----------|
| Autocomplete | Find all words with prefix in O(prefix + results) |
| Spell checker | Check if word exists in dictionary |
| IP routing | Longest prefix matching |
| Word search puzzles | Efficiently prune invalid paths |
| Counting unique prefixes | Each node = one unique prefix |

---

## 2. Segment Tree

### What is a Segment Tree?

A Segment Tree is a binary tree for **range queries** (sum, min, max of a range) with efficient **point updates**. Both operations are **O(log n)**.

```
Problem: Given array [2, 1, 5, 3, 4]
  - Query: What is the SUM of elements from index 1 to 3? → 1+5+3 = 9
  - Update: Change index 2 from 5 to 7

Naive approach: Query O(n), Update O(1)
Segment Tree:   Query O(log n), Update O(log n)  ← Much better for many queries!
```

### Segment Tree Structure — Visual

```
Array: [2, 1, 5, 3, 4]

Segment Tree (stores RANGE SUMS):

                    [0-4] = 15          ← Sum of entire array
                   /        \
            [0-2] = 8      [3-4] = 7   ← Sum of left/right halves
            /     \          /    \
       [0-1]=3  [2-2]=5  [3-3]=3 [4-4]=4  ← Smaller ranges
       /    \
   [0-0]=2 [1-1]=1                      ← Individual elements (leaves)

Each node stores the SUM of a range.
Parent = sum of its two children.

Internal array representation (1-indexed):
Index:  1    2    3    4    5    6    7    8    9
Value: [15,  8,   7,   3,   5,   3,   4,   2,   1]
        ↑    ↑    ↑    ↑    ↑    ↑    ↑    ↑    ↑
      [0-4] [0-2] [3-4] [0-1] [2] [3]  [4]  [0]  [1]
```

### Segment Tree — Full Implementation

```java
class SegmentTree {
    
    private int[] tree;  // The segment tree array
    private int n;       // Size of original array
    
    // ═══════════════════════════════════════════
    // BUILD the segment tree from an array
    // Time: O(n)
    // ═══════════════════════════════════════════
    public SegmentTree(int[] arr) {
        n = arr.length;
        tree = new int[4 * n];  // Segment tree needs at most 4n space
        build(arr, 1, 0, n - 1);
    }
    
    // Build recursively: node 'pos' represents range [start, end]
    private void build(int[] arr, int pos, int start, int end) {
        
        if (start == end) {
            // LEAF NODE: stores a single element
            tree[pos] = arr[start];
            return;
        }
        
        int mid = (start + end) / 2;
        
        // Build left child (left half of range)
        build(arr, 2 * pos, start, mid);         // Left child is at 2*pos
        
        // Build right child (right half of range)
        build(arr, 2 * pos + 1, mid + 1, end);   // Right child is at 2*pos+1
        
        // This node = sum of its two children
        tree[pos] = tree[2 * pos] + tree[2 * pos + 1];
    }
    
    // ═══════════════════════════════════════════
    // QUERY: Sum of elements in range [l, r]
    // Time: O(log n)
    // ═══════════════════════════════════════════
    public int query(int l, int r) {
        return query(1, 0, n - 1, l, r);
    }
    
    private int query(int pos, int start, int end, int l, int r) {
        
        // Case 1: Current range is COMPLETELY OUTSIDE query range
        if (r < start || end < l) {
            return 0;  // Contributes nothing to the sum
        }
        
        // Case 2: Current range is COMPLETELY INSIDE query range
        if (l <= start && end <= r) {
            return tree[pos];  // Use the precomputed sum directly!
        }
        
        // Case 3: PARTIAL overlap — query both children
        int mid = (start + end) / 2;
        int leftSum = query(2 * pos, start, mid, l, r);
        int rightSum = query(2 * pos + 1, mid + 1, end, l, r);
        
        return leftSum + rightSum;
    }
    
    // ═══════════════════════════════════════════
    // UPDATE: Change value at index idx to val
    // Time: O(log n)
    // ═══════════════════════════════════════════
    public void update(int idx, int val) {
        update(1, 0, n - 1, idx, val);
    }
    
    private void update(int pos, int start, int end, int idx, int val) {
        
        if (start == end) {
            // LEAF NODE: update the value
            tree[pos] = val;
            return;
        }
        
        int mid = (start + end) / 2;
        
        if (idx <= mid) {
            update(2 * pos, start, mid, idx, val);       // Go left
        } else {
            update(2 * pos + 1, mid + 1, end, idx, val); // Go right
        }
        
        // Recalculate this node's sum after the child was updated
        tree[pos] = tree[2 * pos] + tree[2 * pos + 1];
    }
}
```

### Dry Run: Build

```
Array: [2, 1, 5, 3, 4]

build(pos=1, range=[0,4])
├── build(pos=2, range=[0,2])
│   ├── build(pos=4, range=[0,1])
│   │   ├── build(pos=8, range=[0,0]) → tree[8] = arr[0] = 2  (leaf)
│   │   └── build(pos=9, range=[1,1]) → tree[9] = arr[1] = 1  (leaf)
│   │   → tree[4] = tree[8] + tree[9] = 2 + 1 = 3
│   └── build(pos=5, range=[2,2]) → tree[5] = arr[2] = 5  (leaf)
│   → tree[2] = tree[4] + tree[5] = 3 + 5 = 8
└── build(pos=3, range=[3,4])
    ├── build(pos=6, range=[3,3]) → tree[6] = arr[3] = 3  (leaf)
    └── build(pos=7, range=[4,4]) → tree[7] = arr[4] = 4  (leaf)
    → tree[3] = tree[6] + tree[7] = 3 + 4 = 7
→ tree[1] = tree[2] + tree[3] = 8 + 7 = 15

Final tree array:
Index:  1    2    3    4    5    6    7    8    9
Value: [15,  8,   7,   3,   5,   3,   4,   2,   1]
```

### Dry Run: Query (sum of range [1, 3])

```
query(pos=1, range=[0,4], query=[1,3])
  Partial overlap → check both children

  ├── query(pos=2, range=[0,2], query=[1,3])
  │   Partial overlap → check both children
  │   
  │   ├── query(pos=4, range=[0,1], query=[1,3])
  │   │   Partial overlap
  │   │   ├── query(pos=8, range=[0,0], query=[1,3])
  │   │   │   0 < 1 → OUTSIDE → return 0
  │   │   └── query(pos=9, range=[1,1], query=[1,3])
  │   │       1 >= 1 && 1 <= 3 → COMPLETELY INSIDE → return tree[9] = 1
  │   │   return 0 + 1 = 1
  │   │
  │   └── query(pos=5, range=[2,2], query=[1,3])
  │       2 >= 1 && 2 <= 3 → COMPLETELY INSIDE → return tree[5] = 5
  │   
  │   return 1 + 5 = 6
  │
  └── query(pos=3, range=[3,4], query=[1,3])
      Partial overlap
      ├── query(pos=6, range=[3,3], query=[1,3])
      │   3 >= 1 && 3 <= 3 → COMPLETELY INSIDE → return tree[6] = 3
      └── query(pos=7, range=[4,4], query=[1,3])
          4 > 3 → OUTSIDE → return 0
      return 3 + 0 = 3

Final: 6 + 3 = 9 ✅  (arr[1]+arr[2]+arr[3] = 1+5+3 = 9)
```

### Dry Run: Update (change index 2 from 5 to 7)

```
update(pos=1, range=[0,4], idx=2, val=7)
  idx=2, mid=2 → idx <= mid → go LEFT
  
  ├── update(pos=2, range=[0,2], idx=2, val=7)
  │   idx=2, mid=1 → idx > mid → go RIGHT
  │   
  │   └── update(pos=5, range=[2,2], idx=2, val=7)
  │       start == end → LEAF → tree[5] = 7
  │   
  │   tree[2] = tree[4] + tree[5] = 3 + 7 = 10  (was 8, now 10)
  
  tree[1] = tree[2] + tree[3] = 10 + 7 = 17  (was 15, now 17)

Updated tree:
Before: [15,  8,  7,  3,  5,  3,  4,  2,  1]
After:  [17, 10,  7,  3,  7,  3,  4,  2,  1]
             ↑           ↑
          updated      updated

Only O(log n) = 3 nodes were updated! (Not the entire tree) ✅
```

### Segment Tree Variants

| Variant | Query Type | Example |
|---------|-----------|---------|
| Sum Segment Tree | Range sum | Sum of arr[l..r] |
| Min Segment Tree | Range minimum | Minimum in arr[l..r] |
| Max Segment Tree | Range maximum | Maximum in arr[l..r] |
| Lazy Propagation | Range update + query | Add value to all elements in range |
| Persistent Segment Tree | Version history | Query any past version |

---

## 3. Disjoint Set Union (DSU / Union-Find)

### What is DSU?

DSU (also called Union-Find) tracks a collection of **disjoint (non-overlapping) sets**. It supports two operations efficiently:

1. **Find(x)**: Which set does element x belong to? (returns the "representative")
2. **Union(x, y)**: Merge the sets containing x and y

```
Initial state: Each element is its own set
{0}, {1}, {2}, {3}, {4}

After union(0, 1): {0, 1}, {2}, {3}, {4}
After union(2, 3): {0, 1}, {2, 3}, {4}
After union(1, 3): {0, 1, 2, 3}, {4}

find(0) == find(3)?  → YES (same set)
find(0) == find(4)?  → NO (different sets)
```

### DSU — Visual with Trees

```
Initially (each element is its own root):
  0    1    2    3    4
  ↑    ↑    ↑    ↑    ↑
 root root root root root

parent: [0, 1, 2, 3, 4]  (each element points to itself)


After union(0, 1): Make 1's root point to 0's root
  0    2    3    4
  ↑    ↑    ↑    ↑
  1
  
parent: [0, 0, 2, 3, 4]
         ↑
        1 now points to 0


After union(2, 3): Make 3's root point to 2's root
  0    2    4
  ↑    ↑    ↑
  1    3
  
parent: [0, 0, 2, 2, 4]


After union(1, 3): find(1)=0, find(3)=2, make 2 point to 0
  0         4
 / \        ↑
1   2
    ↑
    3
    
parent: [0, 0, 0, 2, 4]
```

### DSU — Full Implementation with Optimizations

```java
class DSU {
    
    private int[] parent;  // parent[i] = parent of element i
    private int[] rank;    // rank[i] = approximate depth of tree rooted at i
    private int count;     // Number of disjoint sets
    
    // ═══════════════════════════════════════════
    // Initialize: each element is its own set
    // ═══════════════════════════════════════════
    public DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        count = n;  // Initially n separate sets
        
        for (int i = 0; i < n; i++) {
            parent[i] = i;  // Each element is its own parent (root)
            rank[i] = 0;    // Each tree has height 0
        }
    }
    
    // ═══════════════════════════════════════════
    // FIND: Return the ROOT (representative) of x's set
    // With PATH COMPRESSION: make every node point directly to root
    // Time: O(α(n)) ≈ O(1) amortized (α = inverse Ackermann, extremely slow-growing)
    // ═══════════════════════════════════════════
    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);  // PATH COMPRESSION!
            // Instead of: x → a → b → root
            // After:       x → root, a → root, b → root
            // Future find() calls are O(1)!
        }
        return parent[x];
    }
    
    // ═══════════════════════════════════════════
    // UNION: Merge the sets containing x and y
    // With UNION BY RANK: attach smaller tree under larger tree
    // Time: O(α(n)) ≈ O(1) amortized
    // ═══════════════════════════════════════════
    public boolean union(int x, int y) {
        int rootX = find(x);  // Find root of x's set
        int rootY = find(y);  // Find root of y's set
        
        if (rootX == rootY) {
            return false;  // Already in the same set! No merge needed.
        }
        
        // UNION BY RANK: attach smaller tree under root of larger tree
        if (rank[rootX] < rank[rootY]) {
            parent[rootX] = rootY;           // X's tree goes under Y's root
        } else if (rank[rootX] > rank[rootY]) {
            parent[rootY] = rootX;           // Y's tree goes under X's root
        } else {
            parent[rootY] = rootX;           // Same rank: pick one, increment rank
            rank[rootX]++;
        }
        
        count--;  // One fewer disjoint set
        return true;
    }
    
    // Are x and y in the same set?
    public boolean connected(int x, int y) {
        return find(x) == find(y);
    }
    
    // How many disjoint sets exist?
    public int getCount() {
        return count;
    }
}
```

### Dry Run: DSU Operations

```
═══ Initialize DSU(5) ═══
parent: [0, 1, 2, 3, 4]
rank:   [0, 0, 0, 0, 0]
count:  5

═══ union(0, 1) ═══
  find(0) = 0, find(1) = 1
  rootX=0, rootY=1, rank[0]=0 == rank[1]=0
  → parent[1] = 0, rank[0]++
  
parent: [0, 0, 2, 3, 4]
rank:   [1, 0, 0, 0, 0]
count:  4

Tree:  0    2   3   4
       |
       1

═══ union(2, 3) ═══
  find(2) = 2, find(3) = 3
  rank[2]=0 == rank[3]=0
  → parent[3] = 2, rank[2]++

parent: [0, 0, 2, 2, 4]
rank:   [1, 0, 1, 0, 0]
count:  3

Tree:  0    2    4
       |    |
       1    3

═══ union(1, 3) ═══
  find(1): parent[1]=0, parent[0]=0 → root = 0
  find(3): parent[3]=2, parent[2]=2 → root = 2
  rank[0]=1 == rank[2]=1
  → parent[2] = 0, rank[0]++

parent: [0, 0, 0, 2, 4]
rank:   [2, 0, 1, 0, 0]
count:  2

Tree:    0       4
        / \
       1   2
           |
           3

═══ find(3) with PATH COMPRESSION ═══
  parent[3] = 2
  parent[2] = 0
  parent[0] = 0 → root!
  
  PATH COMPRESSION: set parent[3] = 0 directly!
  
parent: [0, 0, 0, 0, 4]  ← 3 now points directly to root!

Tree:      0       4
         / | \
        1  2  3     ← Flattened! Future find(3) is O(1)

═══ connected(1, 3) ═══
  find(1) = 0, find(3) = 0
  0 == 0 → true ✅ (same set)

═══ connected(0, 4) ═══
  find(0) = 0, find(4) = 4
  0 != 4 → false ❌ (different sets)
```

### DSU Applications

```java
// Application 1: COUNT CONNECTED COMPONENTS in a graph
// Given n nodes and edges, how many connected groups are there?

public int countComponents(int n, int[][] edges) {
    DSU dsu = new DSU(n);
    
    for (int[] edge : edges) {
        dsu.union(edge[0], edge[1]);
    }
    
    return dsu.getCount();  // Number of disjoint sets = connected components
}

// Application 2: DETECT CYCLE in undirected graph
// If union(u, v) returns false, u and v are already connected → CYCLE!

public boolean hasCycle(int n, int[][] edges) {
    DSU dsu = new DSU(n);
    
    for (int[] edge : edges) {
        if (!dsu.union(edge[0], edge[1])) {
            return true;  // Already connected → adding this edge creates a cycle!
        }
    }
    return false;
}

// Application 3: KRUSKAL'S MINIMUM SPANNING TREE
// Sort edges by weight, greedily add if they don't create a cycle (using DSU)

public int kruskalMST(int n, int[][] edges) {
    // edges[i] = [u, v, weight]
    Arrays.sort(edges, (a, b) -> a[2] - b[2]);  // Sort by weight
    
    DSU dsu = new DSU(n);
    int totalWeight = 0;
    int edgesUsed = 0;
    
    for (int[] edge : edges) {
        if (dsu.union(edge[0], edge[1])) {
            totalWeight += edge[2];     // Add this edge to MST
            edgesUsed++;
            if (edgesUsed == n - 1) break;  // MST has exactly n-1 edges
        }
        // If union returns false, skip (would create cycle)
    }
    
    return totalWeight;
}
```

---

## 4. Advanced Dynamic Programming

### Pattern 1: LIS (Longest Increasing Subsequence) — O(n log n)

```
PROBLEM: Find length of longest strictly increasing subsequence.
arr = [10, 9, 2, 5, 3, 7, 101, 18]
Answer: 4 ([2, 3, 7, 101] or [2, 5, 7, 101])
```

```java
public int lengthOfLIS(int[] nums) {
    // tails[i] = smallest ending element of all increasing subsequences of length i+1
    // This array is ALWAYS sorted → we can use binary search!
    
    List<Integer> tails = new ArrayList<>();
    
    for (int num : nums) {
        // Binary search for the position where num should go
        int pos = Collections.binarySearch(tails, num);
        
        if (pos < 0) pos = -(pos + 1);  // Convert to insertion point
        
        if (pos == tails.size()) {
            tails.add(num);        // num is larger than all tails → extend LIS
        } else {
            tails.set(pos, num);   // Replace — keep smallest possible tail
        }
    }
    
    return tails.size();
}
```

```
Dry Run: nums = [10, 9, 2, 5, 3, 7, 101, 18]

num=10:  tails = []     → add 10      → tails = [10]
num=9:   tails = [10]   → 9 < 10, replace at pos 0 → tails = [9]
num=2:   tails = [9]    → 2 < 9, replace at pos 0  → tails = [2]
num=5:   tails = [2]    → 5 > 2, add   → tails = [2, 5]
num=3:   tails = [2, 5] → 3 > 2 but < 5, replace at pos 1 → tails = [2, 3]
num=7:   tails = [2, 3] → 7 > 3, add   → tails = [2, 3, 7]
num=101: tails = [2,3,7] → 101 > 7, add → tails = [2, 3, 7, 101]
num=18:  tails = [2,3,7,101] → 18 replaces 101 → tails = [2, 3, 7, 18]

Length = tails.size() = 4 ✅

NOTE: tails = [2, 3, 7, 18] is NOT the actual LIS!
It represents the smallest possible endings for subsequences of each length.
But its LENGTH is correct.
```

### Pattern 2: Edit Distance (Levenshtein Distance)

```
PROBLEM: Minimum operations (insert, delete, replace) to convert word1 to word2.
word1 = "horse", word2 = "ros"
Answer: 3 (horse → rorse → ros → ros... actually: horse→rorse→rose→ros)

STATE:   dp[i][j] = min operations to convert word1[0..i-1] to word2[0..j-1]
FORMULA: If word1[i-1] == word2[j-1]: dp[i][j] = dp[i-1][j-1]  (no operation needed)
         Else: dp[i][j] = 1 + min(
           dp[i-1][j],     // DELETE from word1
           dp[i][j-1],     // INSERT into word1
           dp[i-1][j-1]    // REPLACE in word1
         )
```

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    // Base cases: converting from/to empty string
    for (int i = 0; i <= m; i++) dp[i][0] = i;  // Delete all chars
    for (int j = 0; j <= n; j++) dp[0][j] = j;  // Insert all chars
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];  // Characters match! No operation.
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i - 1][j - 1],  // Replace
                    Math.min(
                        dp[i - 1][j],   // Delete
                        dp[i][j - 1]    // Insert
                    )
                );
            }
        }
    }
    return dp[m][n];
}
```

```
DP Table: word1 = "horse", word2 = "ros"

         ""   r    o    s
    ┌────┬────┬────┬────┐
 "" │  0 │  1 │  2 │  3 │  ← Insert r, o, s (3 operations for empty→"ros")
    ├────┼────┼────┼────┤
  h │  1 │  1 │  2 │  3 │  ← h≠r: 1+min(0,1,1)=1 (replace h with r)
    ├────┼────┼────┼────┤
  o │  2 │  2 │  1 │  2 │  ← o=o: dp[1][1]=1 (match! no operation)
    ├────┼────┼────┼────┤
  r │  3 │  2 │  2 │  2 │  ← r=r at dp[3][1]: dp[2][0]=2 (match!)
    ├────┼────┼────┼────┤
  s │  4 │  3 │  3 │  2 │  ← s=s at dp[4][3]: dp[3][2]=2 (match!)
    ├────┼────┼────┼────┤
  e │  5 │  4 │  4 │  3 │  ← e≠s: 1+min(2,3,2)=3
    └────┴────┴────┴────┘

Answer: dp[5][3] = 3 ✅
Operations: horse → rorse (replace h→r) → rose (delete r) → ros (delete e)
```

### Pattern 3: Matrix Chain Multiplication (Interval DP)

```
PROBLEM: Given matrices A1(10×30), A2(30×5), A3(5×60)
What's the minimum number of multiplications?

Option 1: (A1 × A2) × A3 = 10×30×5 + 10×5×60 = 1500 + 3000 = 4500
Option 2: A1 × (A2 × A3) = 30×5×60 + 10×30×60 = 9000 + 18000 = 27000

Option 1 is MUCH better! Order of multiplication matters!

dp[i][j] = minimum cost to multiply matrices i through j
Formula: dp[i][j] = min over all k of (dp[i][k] + dp[k+1][j] + dims[i]*dims[k+1]*dims[j+1])
```

---

## 5. Math for Competitive Programming

### GCD & LCM

```java
// GCD (Greatest Common Divisor) — Euclidean Algorithm
// Time: O(log(min(a, b)))
public int gcd(int a, int b) {
    // Idea: gcd(a, b) = gcd(b, a % b)
    // Base case: gcd(a, 0) = a
    
    while (b != 0) {
        int temp = b;
        b = a % b;   // Replace a with b, b with remainder
        a = temp;
    }
    return a;
}

// Recursive version (same thing, just recursive)
public int gcdRecursive(int a, int b) {
    if (b == 0) return a;
    return gcdRecursive(b, a % b);
}

// LCM (Least Common Multiple)
// lcm(a, b) = (a * b) / gcd(a, b)
// Use a/gcd first to avoid overflow!
public long lcm(long a, long b) {
    return a / gcd(a, b) * b;  // Divide first, then multiply!
}
```

```
Dry Run: gcd(48, 18)

Step 1: a=48, b=18 → 48 % 18 = 12 → a=18, b=12
Step 2: a=18, b=12 → 18 % 12 = 6  → a=12, b=6
Step 3: a=12, b=6  → 12 % 6 = 0   → a=6, b=0
Step 4: b=0 → return a = 6 ✅

gcd(48, 18) = 6
lcm(48, 18) = 48/6 * 18 = 8 * 18 = 144
```

### Modular Arithmetic

```java
// WHY? Large numbers overflow. We compute answers modulo 10^9 + 7.
static final int MOD = 1_000_000_007;  // 10^9 + 7 (a large prime)

// Rules:
// (a + b) % m = ((a % m) + (b % m)) % m
// (a * b) % m = ((a % m) * (b % m)) % m
// (a - b) % m = ((a % m) - (b % m) + m) % m  ← Add m to handle negatives!
// (a / b) % m = (a * modInverse(b, m)) % m    ← Can't just divide!

// Modular Exponentiation (Fast Power)
// Computes (base^exp) % mod in O(log exp) time
public long power(long base, long exp, long mod) {
    long result = 1;
    base %= mod;
    
    while (exp > 0) {
        if ((exp & 1) == 1) {           // If exp is ODD
            result = (result * base) % mod;  // Multiply result by base
        }
        exp >>= 1;                      // Divide exp by 2
        base = (base * base) % mod;     // Square the base
    }
    return result;
}

// Modular Inverse (using Fermat's little theorem)
// If mod is prime: a^(-1) ≡ a^(mod-2) (mod p)
public long modInverse(long a, long mod) {
    return power(a, mod - 2, mod);
}
```

```
Dry Run: power(2, 10, 1000000007) = 1024

Step 1: exp=10 (even), base=2²=4, exp=5
Step 2: exp=5 (odd), result=1*4=4, base=4²=16, exp=2
Step 3: exp=2 (even), base=16²=256, exp=1
Step 4: exp=1 (odd), result=4*256=1024, base=256², exp=0
Return 1024 ✅

Only 4 steps for 2^10 instead of 10 multiplications!
For 2^1000000, only ~20 steps! That's the power of O(log n).
```

### Sieve of Eratosthenes (Find All Primes up to N)

```java
// Find all prime numbers up to n
// Time: O(n log log n), Space: O(n)
public boolean[] sieve(int n) {
    boolean[] isPrime = new boolean[n + 1];
    Arrays.fill(isPrime, true);
    isPrime[0] = isPrime[1] = false;  // 0 and 1 are not prime
    
    // For each number i from 2 to √n
    for (int i = 2; i * i <= n; i++) {
        if (isPrime[i]) {
            // Mark all multiples of i as NOT prime
            // Start from i*i (smaller multiples already marked)
            for (int j = i * i; j <= n; j += i) {
                isPrime[j] = false;
            }
        }
    }
    
    return isPrime;
}
```

```
Dry Run: Sieve up to 30

Initial: [F, F, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T, T]
               2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30

i=2: Mark multiples of 2: 4,6,8,10,12,14,16,18,20,22,24,26,28,30
i=3: Mark multiples of 3: 9,15,21,27 (6,12,18,24,30 already marked)
i=4: not prime, skip
i=5: Mark multiples of 5: 25 (10,15,20,25,30 most already marked)

Primes: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29 ✅
```

### nCr (Combinations) with Modular Arithmetic

```java
// Precompute factorials for fast nCr calculations
static final int MOD = 1_000_000_007;
long[] fact;

public void precompute(int n) {
    fact = new long[n + 1];
    fact[0] = 1;
    for (int i = 1; i <= n; i++) {
        fact[i] = fact[i - 1] * i % MOD;
    }
}

// nCr = n! / (r! * (n-r)!)
// In modular arithmetic: nCr = n! * modInverse(r!) * modInverse((n-r)!)
public long nCr(int n, int r) {
    if (r > n || r < 0) return 0;
    return fact[n] % MOD
        * power(fact[r], MOD - 2, MOD) % MOD
        * power(fact[n - r], MOD - 2, MOD) % MOD;
}
```

---

## 6. Bit Manipulation

### Bit Basics — Visual

```
Decimal    Binary     
  0        0000
  1        0001
  2        0010
  3        0011
  4        0100
  5        0101
  6        0110
  7        0111
  8        1000
  10       1010
  15       1111
  255      11111111

OPERATORS:
  AND (&):  1 & 1 = 1, otherwise 0    "Both must be 1"
  OR  (|):  0 | 0 = 0, otherwise 1    "Either can be 1"
  XOR (^):  Same = 0, Different = 1   "Different = 1"
  NOT (~):  Flip all bits
  LEFT SHIFT  (<<): Multiply by 2
  RIGHT SHIFT (>>): Divide by 2
```

### Common Bit Tricks — With Explanations

```java
// ═══════════════════════════════════════════
// TRICK 1: Check if number is EVEN or ODD
// ═══════════════════════════════════════════
boolean isOdd = (n & 1) == 1;
// Last bit of odd numbers is always 1
//   5 = 101 → 101 & 001 = 001 = 1 → ODD
//   6 = 110 → 110 & 001 = 000 = 0 → EVEN

// ═══════════════════════════════════════════
// TRICK 2: Check if number is POWER OF 2
// ═══════════════════════════════════════════
boolean isPowerOf2 = (n > 0) && (n & (n - 1)) == 0;
// Powers of 2 have exactly ONE bit set
//   8 = 1000, 7 = 0111 → 1000 & 0111 = 0000 = 0 → YES ✅
//   6 = 0110, 5 = 0101 → 0110 & 0101 = 0100 ≠ 0 → NO ❌
//  16 = 10000, 15 = 01111 → 10000 & 01111 = 0 → YES ✅

// ═══════════════════════════════════════════
// TRICK 3: Get the i-th bit (0-indexed from right)
// ═══════════════════════════════════════════
int getBit = (n >> i) & 1;
// n=13 (1101), i=2: shift right by 2 → 0011, then & 1 → 1 (bit 2 is set)
// n=13 (1101), i=1: shift right by 1 → 0110, then & 1 → 0 (bit 1 is not set)

// ═══════════════════════════════════════════
// TRICK 4: SET the i-th bit to 1
// ═══════════════════════════════════════════
int setBit = n | (1 << i);
// n=9 (1001), i=1: 1001 | 0010 = 1011 = 11 (set bit 1)

// ═══════════════════════════════════════════
// TRICK 5: CLEAR the i-th bit (set to 0)
// ═══════════════════════════════════════════
int clearBit = n & ~(1 << i);
// n=11 (1011), i=1: ~(0010) = 1101, 1011 & 1101 = 1001 = 9

// ═══════════════════════════════════════════
// TRICK 6: TOGGLE the i-th bit (flip 0↔1)
// ═══════════════════════════════════════════
int toggleBit = n ^ (1 << i);
// n=9 (1001), i=1: 1001 ^ 0010 = 1011 = 11 (toggled bit 1)

// ═══════════════════════════════════════════
// TRICK 7: Count number of SET BITS (1s)
// ═══════════════════════════════════════════
int countBits = Integer.bitCount(n);
// Or manually:
int count = 0;
while (n > 0) {
    count += n & 1;   // Check last bit
    n >>= 1;          // Shift right (remove last bit)
}
// Faster: Brian Kernighan's Algorithm
int count = 0;
while (n > 0) {
    n = n & (n - 1);  // Clear the LOWEST set bit
    count++;
}
// n=13 (1101): 1101 & 1100 = 1100, 1100 & 1011 = 1000, 1000 & 0111 = 0000
// count = 3 (13 has 3 ones in binary) ✅

// ═══════════════════════════════════════════
// TRICK 8: Multiply/Divide by powers of 2
// ═══════════════════════════════════════════
int multiplyBy8 = n << 3;   // n * 2^3 = n * 8
int divideBy4 = n >> 2;     // n / 2^2 = n / 4

// ═══════════════════════════════════════════
// TRICK 9: Swap two numbers WITHOUT temp variable
// ═══════════════════════════════════════════
a = a ^ b;   // a now holds a XOR b
b = a ^ b;   // b = (a XOR b) XOR b = a   (b becomes original a)
a = a ^ b;   // a = (a XOR b) XOR a = b   (a becomes original b)

// ═══════════════════════════════════════════
// TRICK 10: Get lowest set bit
// ═══════════════════════════════════════════
int lowestSetBit = n & (-n);
// n=12 (1100): -12 = 0100 (two's complement), 1100 & 0100 = 0100 = 4
// The lowest set bit of 12 is at position 2 (value 4) ✅
```

### Problem: Single Number (XOR Magic)

```java
// PROBLEM: Every element appears TWICE except one. Find the single one.
// nums = [4, 1, 2, 1, 2]
// Answer: 4

public int singleNumber(int[] nums) {
    int result = 0;
    for (int num : nums) {
        result ^= num;  // XOR with every element
    }
    return result;
}
```

```
Dry Run: nums = [4, 1, 2, 1, 2]

WHY XOR WORKS:
  a ^ a = 0  (any number XOR itself = 0)
  a ^ 0 = a  (any number XOR 0 = itself)
  XOR is commutative and associative

result = 0
result ^= 4 → 0 ^ 4 = 4              (binary: 000 ^ 100 = 100)
result ^= 1 → 4 ^ 1 = 5              (binary: 100 ^ 001 = 101)
result ^= 2 → 5 ^ 2 = 7              (binary: 101 ^ 010 = 111)
result ^= 1 → 7 ^ 1 = 6              (binary: 111 ^ 001 = 110)
result ^= 2 → 6 ^ 2 = 4              (binary: 110 ^ 010 = 100)

Result = 4 ✅

The pairs CANCEL OUT: (1^1) = 0, (2^2) = 0, remaining = 4 ^ 0 ^ 0 = 4
Reordered: 4 ^ (1 ^ 1) ^ (2 ^ 2) = 4 ^ 0 ^ 0 = 4
```

### Problem: Subsets using Bitmask

```java
// Generate ALL subsets of nums using bitmask
// For n elements, there are 2^n subsets (each element: include or exclude)

public List<List<Integer>> subsets(int[] nums) {
    int n = nums.length;
    List<List<Integer>> result = new ArrayList<>();
    
    // Iterate through all 2^n possible masks
    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        
        for (int i = 0; i < n; i++) {
            // Is the i-th bit set in mask? If yes, include nums[i]
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        
        result.add(subset);
    }
    return result;
}
```

```
Dry Run: nums = [a, b, c]  (n=3, 2^3 = 8 subsets)

mask=000 (0): no bits set   → []
mask=001 (1): bit 0 set     → [a]
mask=010 (2): bit 1 set     → [b]
mask=011 (3): bits 0,1 set  → [a, b]
mask=100 (4): bit 2 set     → [c]
mask=101 (5): bits 0,2 set  → [a, c]
mask=110 (6): bits 1,2 set  → [b, c]
mask=111 (7): all bits set  → [a, b, c]

Each bit represents a YES/NO decision for including that element!
```

---

## 7. CP Tricks & Templates

### Fast I/O (Java)

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        // BufferedReader is MUCH faster than Scanner for large inputs
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb = new StringBuilder();  // Batch output
        
        int n = Integer.parseInt(br.readLine().trim());
        
        StringTokenizer st = new StringTokenizer(br.readLine());
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = Integer.parseInt(st.nextToken());
        }
        
        // Process...
        
        sb.append(result).append("\n");
        System.out.print(sb);  // Print all at once (fast!)
    }
}
```

### Useful Java Snippets for CP

```java
// ═══════════════════════════════════════════
// ARRAYS
// ═══════════════════════════════════════════
Arrays.sort(arr);                              // Sort ascending
Arrays.sort(arr, Collections.reverseOrder());  // Sort descending (Integer[])
Arrays.fill(arr, -1);                          // Fill with value
int max = Arrays.stream(arr).max().getAsInt(); // Max element
int sum = Arrays.stream(arr).sum();            // Sum
int[] copy = Arrays.copyOf(arr, arr.length);   // Deep copy

// Sort 2D array by first column
int[][] intervals = {{3,4},{1,2},{5,6}};
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);  // → {{1,2},{3,4},{5,6}}

// Sort by second column, then first
Arrays.sort(intervals, (a, b) -> a[1] != b[1] ? a[1] - b[1] : a[0] - b[0]);

// ═══════════════════════════════════════════
// STRINGS
// ═══════════════════════════════════════════
String reversed = new StringBuilder(s).reverse().toString();
char[] chars = s.toCharArray();
Arrays.sort(chars);
String sorted = new String(chars);
boolean isPalin = s.equals(new StringBuilder(s).reverse().toString());

// ═══════════════════════════════════════════
// COLLECTIONS
// ═══════════════════════════════════════════
// Frequency map
Map<Integer, Integer> freq = new HashMap<>();
for (int x : arr) freq.merge(x, 1, Integer::sum);

// Sorted map (TreeMap)
TreeMap<Integer, Integer> tm = new TreeMap<>();
tm.floorKey(x);    // Largest key ≤ x
tm.ceilingKey(x);  // Smallest key ≥ x
tm.lowerKey(x);    // Largest key < x
tm.higherKey(x);   // Smallest key > x

// Priority Queue (custom comparator)
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);  // Min by first
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[0] - a[0]);  // Max by first

// Deque as stack AND queue
Deque<Integer> deque = new ArrayDeque<>();
deque.push(1);       // Stack push
deque.pop();         // Stack pop
deque.offerLast(1);  // Queue enqueue
deque.pollFirst();   // Queue dequeue

// ═══════════════════════════════════════════
// COORDINATE DIRECTION ARRAYS (for grid problems)
// ═══════════════════════════════════════════
// 4 directions: up, down, left, right
int[] dx = {-1, 1, 0, 0};
int[] dy = {0, 0, -1, 1};

for (int d = 0; d < 4; d++) {
    int nx = x + dx[d];
    int ny = y + dy[d];
    if (nx >= 0 && nx < rows && ny >= 0 && ny < cols) {
        // Valid neighbor (nx, ny)
    }
}

// 8 directions (including diagonals)
int[] dx8 = {-1, -1, -1, 0, 0, 1, 1, 1};
int[] dy8 = {-1, 0, 1, -1, 1, -1, 0, 1};
```

### Binary Search on Answer (CP Pattern)

```java
// PROBLEM: Minimum capacity to ship all packages within D days
// packages = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], days = 5
// Answer: 15 (ship [1,2,3,4,5], [6,7], [8], [9], [10] in 5 days)

public int shipWithinDays(int[] weights, int days) {
    // Binary search on the ANSWER (capacity)
    // Minimum capacity = max single package (must fit the heaviest)
    // Maximum capacity = sum of all packages (ship everything in 1 day)
    
    int left = Arrays.stream(weights).max().getAsInt();
    int right = Arrays.stream(weights).sum();
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        if (canShip(weights, days, mid)) {
            right = mid;        // This capacity works! Try smaller
        } else {
            left = mid + 1;     // Too small, need more capacity
        }
    }
    return left;
}

private boolean canShip(int[] weights, int days, int capacity) {
    int daysNeeded = 1;
    int currentLoad = 0;
    
    for (int w : weights) {
        if (currentLoad + w > capacity) {
            daysNeeded++;           // Start a new day
            currentLoad = 0;
        }
        currentLoad += w;
    }
    
    return daysNeeded <= days;
}
```

```
The KEY INSIGHT of "Binary Search on Answer":

Instead of asking "What's the minimum capacity?"
We ask "Can we do it with capacity X?" (YES/NO question)

If the answer forms a pattern like:
  capacity=10: NO
  capacity=11: NO
  ...
  capacity=15: YES ← First YES = answer!
  capacity=16: YES
  ...

This is MONOTONIC (once YES, always YES after).
Binary search finds the transition point in O(log n)!
```

### Monotonic Stack Pattern

```java
// PROBLEM: Next Greater Element
// For each element, find the NEXT element that is GREATER
// nums = [2, 1, 2, 4, 3]
// answer = [4, 2, 4, -1, -1]

public int[] nextGreaterElement(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);     // Default: no greater element
    
    Deque<Integer> stack = new ArrayDeque<>();  // Stack of INDICES
    // Stack maintains DECREASING order of values
    
    for (int i = 0; i < n; i++) {
        // While current element is GREATER than stack top
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            int idx = stack.pop();
            result[idx] = nums[i];  // nums[i] is the next greater for nums[idx]
        }
        stack.push(i);
    }
    
    return result;
}
```

```
Dry Run: nums = [2, 1, 2, 4, 3]

i=0: nums[0]=2, stack empty → push 0     Stack: [0]         (values: [2])
i=1: nums[1]=1, 1 < 2 → push 1          Stack: [0, 1]      (values: [2, 1])
i=2: nums[2]=2, 2 > 1 → pop 1, result[1]=2
                  2 = 2, not > → push 2  Stack: [0, 2]      (values: [2, 2])
i=3: nums[3]=4, 4 > 2 → pop 2, result[2]=4
                  4 > 2 → pop 0, result[0]=4
                  stack empty → push 3    Stack: [3]         (values: [4])
i=4: nums[4]=3, 3 < 4 → push 4          Stack: [3, 4]      (values: [4, 3])

Remaining in stack → no next greater → result stays -1

result = [4, 2, 4, -1, -1] ✅
```

---

## 8. Interview Questions (30+)

### Trie Questions

**Q1: What is a Trie and when would you use it?**
**A:** A tree where each node represents a character. Used for prefix-based lookups in O(L) time where L = word length. Use cases: autocomplete, spell check, IP routing, word games.

**Q2: Time complexity of Trie insert/search?**
**A:** O(L) where L = length of the word. This is independent of the number of words stored!

**Q3: Trie vs HashMap for storing strings?**
**A:** HashMap: O(L) search per word but no prefix operations. Trie: O(L) search AND prefix queries AND autocomplete. Trie uses more memory but supports prefix operations natively.

**Q4: How would you implement wildcard search in a Trie?**
**A:** Use DFS. When encountering '.', explore ALL children instead of a specific one.

**Q5: Space complexity of a Trie?**
**A:** O(N × L × ALPHABET_SIZE) worst case, where N = words, L = avg length. Can be optimized with compressed tries or hash maps instead of arrays.

### Segment Tree Questions

**Q6: What problem does a Segment Tree solve?**
**A:** Range queries (sum/min/max of a range) with point updates, both in O(log n). Array alone gives O(n) query or O(n) update.

**Q7: What is lazy propagation?**
**A:** An optimization for RANGE UPDATES. Instead of updating all leaves, we store pending updates and only apply them when needed. Both range update and range query become O(log n).

**Q8: Segment Tree vs Fenwick Tree (BIT)?**
**A:** Fenwick (BIT): simpler, less memory, O(log n) for prefix sum + point update. Segment Tree: more general (supports min/max, not just sum), supports range updates with lazy propagation.

**Q9: How much memory does a Segment Tree use?**
**A:** Array of size 4n for n elements (to safely fit all levels of the tree).

### DSU Questions

**Q10: What is path compression in Union-Find?**
**A:** During find(), we make every visited node point directly to the root. This flattens the tree, making future find() calls nearly O(1).

**Q11: What is union by rank?**
**A:** When merging two trees, attach the shorter tree under the taller tree's root. This keeps trees balanced and operations fast.

**Q12: Time complexity of DSU with both optimizations?**
**A:** O(α(n)) per operation, where α is the inverse Ackermann function. For all practical purposes, this is O(1).

**Q13: Applications of DSU?**
**A:** Connected components, cycle detection in undirected graphs, Kruskal's MST, dynamic connectivity, network connectivity.

### Bit Manipulation Questions

**Q14: How to check if a number is a power of 2?**
**A:** `n > 0 && (n & (n-1)) == 0`. Powers of 2 have exactly one set bit. Subtracting 1 flips all bits below it. AND gives 0.

**Q15: How to count set bits in an integer?**
**A:** `Integer.bitCount(n)` or Brian Kernighan's: `while(n>0) { n &= n-1; count++; }`. Each `n &= n-1` clears the lowest set bit.

**Q16: How does XOR help find a single non-duplicate element?**
**A:** XOR of a number with itself = 0. XOR of all elements cancels out pairs, leaving the single element.

**Q17: What is a bitmask?**
**A:** An integer where each bit represents a boolean (include/exclude). Used for subset generation, state representation in DP.

### Math Questions

**Q18: What is the Sieve of Eratosthenes?**
**A:** Algorithm to find all primes up to n by iteratively marking multiples as composite. Time: O(n log log n).

**Q19: Why do we use MOD = 10^9+7?**
**A:** It's a large prime that fits in 32-bit int (and its square fits in 64-bit long). Being prime allows modular inverse via Fermat's theorem.

**Q20: How to compute a^b mod m efficiently?**
**A:** Binary exponentiation: if b is even, a^b = (a^(b/2))². If b is odd, a^b = a × a^(b-1). O(log b) time.

### Advanced DP Questions

**Q21: What is the difference between top-down and bottom-up DP?**
**A:** Top-down: recursion + memoization (natural but may stack overflow). Bottom-up: iteration + table (faster, no stack overhead). Both are equivalent in logic.

**Q22: What is Interval DP?**
**A:** DP on subarray ranges [i,j]. Used for problems like matrix chain multiplication, palindrome partitioning. dp[i][j] depends on all splits dp[i][k] + dp[k+1][j].

**Q23: What is DP with bitmask?**
**A:** Using a bitmask to represent a subset of elements as the DP state. Used for TSP, assignment problem. State: dp[mask] where mask is a subset of visited nodes.

**Q24: How to identify if a problem needs DP?**
**A:** Two signs: 1) Optimal substructure (optimal solution uses optimal solutions to subproblems). 2) Overlapping subproblems (same subproblem solved multiple times).

### CP Pattern Questions

**Q25: What is "Binary Search on Answer"?**
**A:** When the answer is monotonic (e.g., if capacity=15 works, then capacity=16 also works), binary search the answer space. Convert "find minimum X" into "can we do it with X?" (YES/NO).

**Q26: What is a Monotonic Stack?**
**A:** A stack maintaining elements in sorted (increasing or decreasing) order. Used for "next greater/smaller element" problems in O(n).

**Q27: When to use BFS vs DFS?**
**A:** BFS: shortest path (unweighted), level-order. DFS: exhaustive search, cycle detection, topological sort, backtracking.

**Q28: What is the two-pointer technique?**
**A:** Two indices moving through sorted data. Opposite directions: 2Sum on sorted array. Same direction: slow/fast for cycle detection, sliding window.

**Q29: How to detect if a graph problem needs Dijkstra vs BFS?**
**A:** BFS: unweighted graph (all edges cost 1). Dijkstra: weighted graph (non-negative weights). Bellman-Ford: negative weights.

**Q30: What is the significance of α(n) in DSU?**
**A:** α(n) is the inverse Ackermann function. It grows SO slowly that α(n) ≤ 4 for any n up to 2^(2^(2^65536)). Effectively O(1) for all practical purposes.

---

## 📚 References

- [CP Handbook (by Antti Laaksonen)](https://cses.fi/book/book.pdf) — Free PDF
- [CSES Problem Set](https://cses.fi/problemset/) — 300 curated problems
- [Competitive Programmer's Handbook](https://cpbook.net/)
- [Algorithms Live! (YouTube)](https://www.youtube.com/c/AlgorithmsLive)
- [AtCoder Educational DP Contest](https://atcoder.jp/contests/dp)
- [Codeforces EDU](https://codeforces.com/edu/courses)

---

> **Main DSA Guide:** [← README.md](./README.md)
