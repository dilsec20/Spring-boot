# 📊 Segment Trees & Fenwick Trees — FAANG Masterclass

> **"You want the sum of elements from index 10,000 to 90,000? And then you want to update index 50,000? And you want it instantly? Meet the Segment Tree."**

---

## 📑 Table of Contents
1. [The Problem Segment Trees Solve](#1-the-problem-segment-trees-solve)
2. [What is a Segment Tree?](#2-what-is-a-segment-tree)
3. [Building the Tree — Full Dry Run](#3-building-the-tree)
4. [Point Updates — Full Dry Run](#4-point-updates)
5. [Range Queries — Full Dry Run](#5-range-queries)
6. [Complete Implementation (Sum Segment Tree)](#6-complete-implementation)
7. [Min/Max Segment Tree Variant](#7-minmax-segment-tree-variant)
8. [Lazy Propagation (Range Updates)](#8-lazy-propagation)
9. [Fenwick Tree (Binary Indexed Tree)](#9-fenwick-tree)
10. [Comparison: Segment Tree vs Fenwick Tree vs Others](#10-comparison)
11. [Problem: Range Sum Query — Mutable (LC 307)](#11-range-sum-query-mutable)
12. [Problem: Count of Smaller Numbers After Self (LC 315)](#12-count-of-smaller-numbers-after-self)
13. [Problem: Range Minimum Query](#13-range-minimum-query)
14. [When to Use What?](#14-when-to-use-what)
15. [Top 10 Must-Do Segment Tree Questions](#15-top-10-must-do-questions)

---

## 1. The Problem Segment Trees Solve

Imagine an array: `arr = [1, 3, 5, 7, 9, 11]`

You have two operations repeated thousands of times:
1.  **Update(index, val):** Update the value at `index`.
2.  **Query(L, R):** Get the sum (or Min/Max) of elements from index `L` to index `R`.

| Approach | Update | Query | Good for? |
|:---|:---:|:---:|:---|
| **Basic Array** | $O(1)$ | $O(N)$ | Few queries, many updates |
| **Prefix Sum** | $O(N)$ | $O(1)$ | Few updates, many queries |
| **Segment Tree** | $O(\log N)$ | $O(\log N)$ | **Both frequent!** ✅ |
| **Fenwick Tree** | $O(\log N)$ | $O(\log N)$ | Sum queries (simpler code) |

---

## 2. What is a Segment Tree?

A Segment Tree is a **binary tree** where:
*   The **Root** represents the entire array range `[0...N-1]`.
*   Each **Leaf Node** represents a single element `[i...i]`.
*   Each **Internal Node** stores the merged result (Sum, Min, Max) of its left and right children.

For array size $N$, the Segment Tree is stored in an array of size **$4N$**.

*(If `node` is at index `i`, its left child is at `2*i + 1`, and right child is at `2*i + 2`)*

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
```

---

## 3. Building the Tree

We use **Divide and Conquer** (Post-order traversal):

1. If `start == end`, we are at a leaf node. `tree[node] = arr[start]`.
2. Find `mid`. Recursively build Left `[start...mid]` and Right `[mid+1...end]`.
3. Backtrack and merge: `tree[node] = tree[left] + tree[right]`.

### 🧠 Full Build Dry Run

```
Array: [2, 1, 5, 3, 4]

build(node=0, start=0, end=4)  → represents [0-4]
├── build(node=1, start=0, end=2)  → represents [0-2]
│   ├── build(node=3, start=0, end=1)  → represents [0-1]
│   │   ├── build(node=7, start=0, end=0)  → LEAF! tree[7] = arr[0] = 2
│   │   └── build(node=8, start=1, end=1)  → LEAF! tree[8] = arr[1] = 1
│   │   tree[3] = tree[7] + tree[8] = 2 + 1 = 3
│   └── build(node=4, start=2, end=2)  → LEAF! tree[4] = arr[2] = 5
│   tree[1] = tree[3] + tree[4] = 3 + 5 = 8
└── build(node=2, start=3, end=4)  → represents [3-4]
    ├── build(node=5, start=3, end=3)  → LEAF! tree[5] = arr[3] = 3
    └── build(node=6, start=4, end=4)  → LEAF! tree[6] = arr[4] = 4
    tree[2] = tree[5] + tree[6] = 3 + 4 = 7
tree[0] = tree[1] + tree[2] = 8 + 7 = 15

Final tree[] array:
Index:  0    1    2    3    4    5    6    7    8
Value: [15,  8,   7,   3,   5,   3,   4,   2,   1]
Range: [0-4][0-2][3-4][0-1][2-2][3-3][4-4][0-0][1-1]
```

---

## 4. Point Updates

When an array element changes, update the leaf and **propagate changes UP to root**.

1. Is the target index in the Left half or Right half?
2. Go down the correct path recursively.
3. Update the leaf.
4. On the way back up, recalculate parents: `tree[node] = tree[left] + tree[right]`.

### 🧠 Update Dry Run

```
Update arr[2] = 5 → 10 (adding 5)

Before:
                     [0-4] = 15
                    /        \
             [0-2] = 8      [3-4] = 7
             /     \          /    \
        [0-1]=3  [2-2]=5  [3-3]=3 [4-4]=4
        /    \
    [0-0]=2 [1-1]=1

Step 1: update(node=0, range=[0-4], idx=2, val=10)
  mid = 2, idx=2 ≤ mid → go LEFT

Step 2: update(node=1, range=[0-2], idx=2, val=10)
  mid = 1, idx=2 > mid → go RIGHT

Step 3: update(node=4, range=[2-2], idx=2, val=10)
  start == end → LEAF! tree[4] = 10 ✅

Backtrack:
  tree[1] = tree[3] + tree[4] = 3 + 10 = 13
  tree[0] = tree[1] + tree[2] = 13 + 7 = 20

After:
                     [0-4] = 20  ← updated!
                    /        \
             [0-2] = 13     [3-4] = 7  ← updated!
             /     \          /    \
        [0-1]=3  [2-2]=10 [3-3]=3 [4-4]=4  ← updated!
        /    \
    [0-0]=2 [1-1]=1

Only 3 nodes changed → O(log N) ✅
```

---

## 5. Range Queries

We want the sum from `L` to `R`.
At any node representing range `[start...end]`, there are **3 cases**:

1.  **Complete Overlap:** `[start...end]` is completely inside `[L...R]`.
    → **Return the node's value immediately!** (This makes it $O(\log N)$)

2.  **No Overlap:** `[start...end]` is completely outside `[L...R]`.
    → **Return 0** (identity for sum).

3.  **Partial Overlap:** Partial intersection.
    → **Go down BOTH children**, add their results.

### 🧠 Query Dry Run

```
Array: [2, 1, 10, 3, 4], query sum(1, 3) → expected: 1+10+3 = 14

Tree:
                     [0-4] = 20
                    /        \
             [0-2] = 13     [3-4] = 7
             /     \          /    \
        [0-1]=3  [2-2]=10 [3-3]=3 [4-4]=4
        /    \
    [0-0]=2 [1-1]=1

query(node=0, range=[0-4], L=1, R=3)
  PARTIAL OVERLAP (0<1, 4>3) → go both children

  query(node=1, range=[0-2], L=1, R=3)
    PARTIAL OVERLAP (0<1) → go both children

    query(node=3, range=[0-1], L=1, R=3)
      PARTIAL OVERLAP (0<1) → go both children

      query(node=7, range=[0-0], L=1, R=3)
        NO OVERLAP (0 < 1) → return 0 ❌

      query(node=8, range=[1-1], L=1, R=3)
        COMPLETE OVERLAP ([1-1] inside [1-3]) → return 1 ✅

    Return: 0 + 1 = 1

    query(node=4, range=[2-2], L=1, R=3)
      COMPLETE OVERLAP ([2-2] inside [1-3]) → return 10 ✅

  Return: 1 + 10 = 11

  query(node=2, range=[3-4], L=1, R=3)
    PARTIAL OVERLAP (4>3) → go both children

    query(node=5, range=[3-3], L=1, R=3)
      COMPLETE OVERLAP ([3-3] inside [1-3]) → return 3 ✅

    query(node=6, range=[4-4], L=1, R=3)
      NO OVERLAP (4 > 3) → return 0 ❌

  Return: 3 + 0 = 3

Total: 11 + 3 = 14 ✅ (matches 1+10+3!)
```

---

## 6. Complete Implementation

```java
class SegmentTree {
    int[] tree;
    int n;

    public SegmentTree(int[] arr) {
        this.n = arr.length;
        this.tree = new int[4 * n];
        build(arr, 0, 0, n - 1);
    }

    // ════════════════════════════════════════
    // 1. BUILD the Tree — O(N)
    // ════════════════════════════════════════
    private void build(int[] arr, int node, int start, int end) {
        if (start == end) {
            tree[node] = arr[start]; // Leaf node
            return;
        }
        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;
        
        build(arr, leftChild, start, mid);       // Build left
        build(arr, rightChild, mid + 1, end);    // Build right
        
        tree[node] = tree[leftChild] + tree[rightChild]; // Merge
    }

    // ════════════════════════════════════════
    // 2. POINT UPDATE — O(log N)
    // ════════════════════════════════════════
    public void update(int index, int val) {
        updateHelper(0, 0, n - 1, index, val);
    }

    private void updateHelper(int node, int start, int end, int idx, int val) {
        if (start == end) {
            tree[node] = val; // Found the leaf, update it
            return;
        }
        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;
        
        if (idx <= mid) {
            updateHelper(leftChild, start, mid, idx, val);     // Go left
        } else {
            updateHelper(rightChild, mid + 1, end, idx, val);  // Go right
        }
        
        // Recalculate on the way back up
        tree[node] = tree[leftChild] + tree[rightChild];
    }

    // ════════════════════════════════════════
    // 3. RANGE QUERY — O(log N)
    // ════════════════════════════════════════
    public int query(int L, int R) {
        return queryHelper(0, 0, n - 1, L, R);
    }

    private int queryHelper(int node, int start, int end, int L, int R) {
        // Case 1: No overlap
        if (R < start || L > end) {
            return 0; // Identity for SUM (use Integer.MAX_VALUE for MIN)
        }
        // Case 2: Complete overlap
        if (L <= start && end <= R) {
            return tree[node]; // Return precomputed value!
        }
        // Case 3: Partial overlap — go both directions
        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;
        
        int leftSum = queryHelper(leftChild, start, mid, L, R);
        int rightSum = queryHelper(rightChild, mid + 1, end, L, R);
        
        return leftSum + rightSum;
    }
}
```

---

## 7. Min/Max Segment Tree Variant

Change the merge function and the identity value:

```java
// ══════ MIN SEGMENT TREE ══════

// Build: merge with MIN instead of SUM
tree[node] = Math.min(tree[leftChild], tree[rightChild]);

// Query: return identity for "no overlap"
// Identity for MIN = Integer.MAX_VALUE (doesn't affect min)
if (R < start || L > end) return Integer.MAX_VALUE;

// Range query returns MIN of range
return Math.min(leftResult, rightResult);

// ══════ MAX SEGMENT TREE ══════

// Build:
tree[node] = Math.max(tree[leftChild], tree[rightChild]);

// Identity:
if (R < start || L > end) return Integer.MIN_VALUE;

// Query:
return Math.max(leftResult, rightResult);
```

### Generalized Segment Tree (Template)
```java
class SegmentTree<T> {
    // Change these for different operations:
    // SUM:  merge(a,b) = a+b,     identity = 0
    // MIN:  merge(a,b) = min(a,b), identity = Integer.MAX_VALUE
    // MAX:  merge(a,b) = max(a,b), identity = Integer.MIN_VALUE
    // GCD:  merge(a,b) = gcd(a,b), identity = 0
    // OR:   merge(a,b) = a|b,      identity = 0
    // AND:  merge(a,b) = a&b,      identity = ~0 (all 1s)
}
```

---

## 8. Lazy Propagation

### The Problem
What if you need to **update a RANGE** of elements?
e.g., "Add 5 to all elements from index 100 to 500."

Point-by-point updates: $O(N \log N)$ → TOO SLOW!

### The Solution
**Lazy Propagation** defers updates to children until they are needed:
1. Mark the parent as "lazy" (has pending updates for children).
2. Only propagate when a child node is actually accessed.
3. This keeps **Range Updates** at $O(\log N)$.

### 🧠 Lazy Propagation Concept

```
Range Update: Add 5 to all elements in [1, 3]

WITHOUT Lazy (update each point):
  update(1, +5) → O(log N)
  update(2, +5) → O(log N)
  update(3, +5) → O(log N)
  Total: O(N log N) for range [0, N-1]

WITH Lazy:
  Mark entire range [1-3] as "lazy += 5"
  Don't go deeper unless needed!
  Total: O(log N) ✅
```

### Full Implementation with Lazy Propagation

```java
class LazySegmentTree {
    int[] tree, lazy;
    int n;

    public LazySegmentTree(int[] arr) {
        n = arr.length;
        tree = new int[4 * n];
        lazy = new int[4 * n];  // Lazy tags (pending additions)
        build(arr, 0, 0, n - 1);
    }

    private void build(int[] arr, int node, int start, int end) {
        if (start == end) {
            tree[node] = arr[start];
            return;
        }
        int mid = start + (end - start) / 2;
        build(arr, 2*node+1, start, mid);
        build(arr, 2*node+2, mid+1, end);
        tree[node] = tree[2*node+1] + tree[2*node+2];
    }

    // ════════════════════════════════════════
    // PUSH DOWN lazy values to children
    // ════════════════════════════════════════
    private void pushDown(int node, int start, int end) {
        if (lazy[node] != 0) {
            int mid = start + (end - start) / 2;
            int leftChild = 2 * node + 1;
            int rightChild = 2 * node + 2;

            // Apply lazy to children's tree values
            tree[leftChild] += lazy[node] * (mid - start + 1);
            tree[rightChild] += lazy[node] * (end - mid);

            // Pass lazy tag to children
            lazy[leftChild] += lazy[node];
            lazy[rightChild] += lazy[node];

            // Clear lazy tag
            lazy[node] = 0;
        }
    }

    // ════════════════════════════════════════
    // RANGE UPDATE: Add val to all elements in [L, R]
    // ════════════════════════════════════════
    public void rangeUpdate(int L, int R, int val) {
        rangeUpdateHelper(0, 0, n - 1, L, R, val);
    }

    private void rangeUpdateHelper(int node, int start, int end, int L, int R, int val) {
        if (R < start || L > end) return; // No overlap

        if (L <= start && end <= R) {
            // Complete overlap — apply update and mark lazy
            tree[node] += val * (end - start + 1);
            lazy[node] += val;
            return;
        }

        // Partial overlap — push down first, then recurse
        pushDown(node, start, end);
        int mid = start + (end - start) / 2;
        rangeUpdateHelper(2*node+1, start, mid, L, R, val);
        rangeUpdateHelper(2*node+2, mid+1, end, L, R, val);
        tree[node] = tree[2*node+1] + tree[2*node+2];
    }

    // ════════════════════════════════════════
    // RANGE QUERY with lazy propagation
    // ════════════════════════════════════════
    public int query(int L, int R) {
        return queryHelper(0, 0, n - 1, L, R);
    }

    private int queryHelper(int node, int start, int end, int L, int R) {
        if (R < start || L > end) return 0;

        if (L <= start && end <= R) return tree[node];

        pushDown(node, start, end);  // Push lazy before going deeper!
        int mid = start + (end - start) / 2;
        return queryHelper(2*node+1, start, mid, L, R)
             + queryHelper(2*node+2, mid+1, end, L, R);
    }
}
```

### 🧠 Lazy Propagation Dry Run

```
Array: [1, 3, 5, 7, 9]

Initial Tree:
              [0-4]=25
             /        \
        [0-2]=9      [3-4]=16
        /    \        /    \
   [0-1]=4 [2]=5   [3]=7  [4]=9
   /    \
  [0]=1 [1]=3


═══ rangeUpdate(1, 3, +10) ═══  Add 10 to indices 1, 2, 3

Step 1: node=0, range=[0-4], L=1, R=3 → PARTIAL OVERLAP

Step 2: Left child, node=1, range=[0-2], L=1, R=3 → PARTIAL OVERLAP
  Step 3: node=3, range=[0-1], L=1, R=3 → PARTIAL
    node=7, range=[0-0] → NO OVERLAP → return
    node=8, range=[1-1] → COMPLETE OVERLAP!
      tree[8] += 10*1 = 13, lazy[8] += 10
  Step 4: node=4, range=[2-2] → COMPLETE OVERLAP!
    tree[4] += 10*1 = 15, lazy[4] += 10
  tree[1] = tree[3] + tree[4] = (1+13) + 15 = 29

Step 5: Right child, node=2, range=[3-4], L=1, R=3 → PARTIAL
  node=5, range=[3-3] → COMPLETE OVERLAP!
    tree[5] += 10*1 = 17, lazy[5] += 10
  node=6, range=[4-4] → NO OVERLAP → return
  tree[2] = tree[5] + tree[6] = 17 + 9 = 26

tree[0] = tree[1] + tree[2] = 29 + 26 = 55

Effective array: [1, 13, 15, 17, 9] → sum = 55 ✅
```

---

## 9. Fenwick Tree (Binary Indexed Tree)

A Fenwick Tree is a **simpler alternative** to Segment Tree for **prefix sum queries** and **point updates**. Uses clever bit manipulation.

### Key Idea
Each index `i` stores the sum of a specific range determined by the **lowest set bit** of `i`.

```
Index (1-indexed): 1    2    3    4    5    6    7    8
Binary:            001  010  011  100  101  110  111  1000
Lowest bit:        1    2    1    4    1    2    1    8
Stores sum of:     [1]  [1-2] [3] [1-4] [5] [5-6] [7] [1-8]
```

### Full Implementation

```java
class FenwickTree {
    int[] bit;
    int n;

    public FenwickTree(int n) {
        this.n = n;
        bit = new int[n + 1]; // 1-indexed!
    }

    // Build from array
    public FenwickTree(int[] arr) {
        this(arr.length);
        for (int i = 0; i < arr.length; i++) {
            update(i + 1, arr[i]);
        }
    }

    // ════════════════════════════════════════
    // POINT UPDATE: Add val to index i
    // Climb UP: i += (i & -i)
    // ════════════════════════════════════════
    public void update(int i, int val) {
        while (i <= n) {
            bit[i] += val;
            i += (i & -i); // Add lowest set bit
        }
    }

    // ════════════════════════════════════════
    // PREFIX SUM: Sum from index 1 to i
    // Climb DOWN: i -= (i & -i)
    // ════════════════════════════════════════
    public int prefixSum(int i) {
        int sum = 0;
        while (i > 0) {
            sum += bit[i];
            i -= (i & -i); // Remove lowest set bit
        }
        return sum;
    }

    // RANGE SUM: Sum from index L to R (1-indexed)
    public int rangeSum(int L, int R) {
        return prefixSum(R) - prefixSum(L - 1);
    }
}
```

### 🧠 Fenwick Tree Dry Run

```
Array (1-indexed): [_, 1, 3, 5, 7, 9, 11]

Building BIT by inserting elements one by one:

After all insertions:
Index:    1    2    3    4    5    6
bit[]:  [ 1,   4,   5,  16,   9,  20]
         [1] [1-2] [3] [1-4] [5] [5-6]

═══ prefixSum(4) ═══  (Sum of indices 1 to 4)
i=4 (binary 100): bit[4]=16, i -= 100 → i=0 → STOP
Sum = 16 = 1+3+5+7 ✅

═══ prefixSum(3) ═══  (Sum of indices 1 to 3)
i=3 (binary 011): bit[3]=5, i -= 001 → i=2
i=2 (binary 010): bit[2]=4, i -= 010 → i=0 → STOP
Sum = 5+4 = 9 = 1+3+5 ✅

═══ rangeSum(2, 4) ═══
= prefixSum(4) - prefixSum(1) = 16 - 1 = 15 = 3+5+7 ✅

═══ update(3, +2) ═══  (Add 2 to index 3)
i=3 (binary 011): bit[3]+=2 → 7, i += 001 → i=4
i=4 (binary 100): bit[4]+=2 → 18, i += 100 → i=8 > n → STOP

After: arr effectively becomes [_, 1, 3, 7, 7, 9, 11]
```

---

## 10. Comparison

| Feature | Segment Tree | Fenwick Tree (BIT) | Sparse Table |
|:---|:---|:---|:---|
| **Build** | O(N) | O(N log N) | O(N log N) |
| **Point Update** | O(log N) | O(log N) | ❌ Not supported |
| **Range Query** | O(log N) | O(log N) | O(1) |
| **Range Update** | O(log N) with lazy | ❌ Complex | ❌ Not supported |
| **Code Complexity** | High | **Low** ✅ | Medium |
| **Space** | 4N | N+1 | N log N |
| **Supports** | Any operation | **Only prefix-decomposable** (sum, XOR) | **Only idempotent** (min, max, GCD) |
| **Best for** | Most general | **Sum queries (interview)** | Static RMQ |

**When to choose:**
- **Segment Tree:** Range updates needed, or non-prefix operations (min/max with lazy)
- **Fenwick Tree:** Only sum/XOR queries + point updates → simpler code, faster constant
- **Sparse Table:** Static array, only RMQ, no updates → O(1) queries

---

## 11. Range Sum Query — Mutable (LC 307)

**Problem:** Implement `update(index, val)` and `sumRange(left, right)`.

```java
class NumArray {
    private SegmentTree segTree;
    
    public NumArray(int[] nums) {
        segTree = new SegmentTree(nums);
    }
    
    public void update(int index, int val) {
        segTree.update(index, val);
    }
    
    public int sumRange(int left, int right) {
        return segTree.query(left, right);
    }
}

// Alternative: Fenwick Tree (simpler for this problem!)
class NumArray {
    int[] bit;
    int[] nums;
    int n;
    
    public NumArray(int[] nums) {
        this.n = nums.length;
        this.nums = nums.clone();
        this.bit = new int[n + 1];
        for (int i = 0; i < n; i++) add(i + 1, nums[i]);
    }
    
    private void add(int i, int val) {
        while (i <= n) { bit[i] += val; i += (i & -i); }
    }
    
    private int prefixSum(int i) {
        int sum = 0;
        while (i > 0) { sum += bit[i]; i -= (i & -i); }
        return sum;
    }
    
    public void update(int index, int val) {
        int diff = val - nums[index];
        nums[index] = val;
        add(index + 1, diff);
    }
    
    public int sumRange(int left, int right) {
        return prefixSum(right + 1) - prefixSum(left);
    }
}
```

---

## 12. Count of Smaller Numbers After Self (LC 315)

**Problem:** For each element, count how many smaller elements are to its right.

Example: `[5, 2, 6, 1]` → `[2, 1, 1, 0]`

### 🧠 How to Think
Process from RIGHT to LEFT. Use a Fenwick Tree indexed by value. For each number, query "how many numbers smaller than me have been inserted?" Then insert the current number.

```java
public List<Integer> countSmaller(int[] nums) {
    // Coordinate compression: map values to 1..n
    int[] sorted = nums.clone();
    Arrays.sort(sorted);
    Map<Integer, Integer> rank = new HashMap<>();
    int r = 1;
    for (int num : sorted) {
        if (!rank.containsKey(num)) rank.put(num, r++);
    }
    
    int maxRank = rank.size();
    int[] bit = new int[maxRank + 1];
    Integer[] result = new Integer[nums.length];
    
    // Process from right to left
    for (int i = nums.length - 1; i >= 0; i--) {
        int rnk = rank.get(nums[i]);
        
        // Query: how many elements with rank < rnk? → prefixSum(rnk - 1)
        result[i] = prefixSum(bit, rnk - 1);
        
        // Update: add current element
        updateBIT(bit, maxRank, rnk, 1);
    }
    
    return Arrays.asList(result);
}

private void updateBIT(int[] bit, int n, int i, int val) {
    while (i <= n) { bit[i] += val; i += (i & -i); }
}

private int prefixSum(int[] bit, int i) {
    int sum = 0;
    while (i > 0) { sum += bit[i]; i -= (i & -i); }
    return sum;
}
```

### 🧠 Dry Run

```
nums = [5, 2, 6, 1]
ranks: 1→1, 2→2, 5→3, 6→4

Process right to left:

i=3, nums[3]=1, rank=1:
  query prefixSum(0) = 0 → result[3] = 0
  insert rank 1 → BIT has {1}

i=2, nums[2]=6, rank=4:
  query prefixSum(3) = 1 (rank 1 is present) → result[2] = 1
  insert rank 4 → BIT has {1, 4}

i=1, nums[1]=2, rank=2:
  query prefixSum(1) = 1 (rank 1 is present) → result[1] = 1
  insert rank 2 → BIT has {1, 2, 4}

i=0, nums[0]=5, rank=3:
  query prefixSum(2) = 2 (ranks 1, 2 are present) → result[0] = 2
  insert rank 3

Result: [2, 1, 1, 0] ✅
```

---

## 13. Range Minimum Query

Static array, many min queries, no updates → Use **Sparse Table** for O(1) queries.
If updates needed → Use **Min Segment Tree**.

### Sparse Table Implementation (O(1) query, no updates)

```java
class SparseTable {
    int[][] table; // table[i][j] = min of range starting at i, length 2^j
    int[] log;
    
    public SparseTable(int[] arr) {
        int n = arr.length;
        int maxLog = (int)(Math.log(n) / Math.log(2)) + 1;
        table = new int[n][maxLog + 1];
        log = new int[n + 1];
        
        // Precompute logs
        for (int i = 2; i <= n; i++) log[i] = log[i / 2] + 1;
        
        // Base case: ranges of length 1
        for (int i = 0; i < n; i++) table[i][0] = arr[i];
        
        // Build: ranges of length 2^j
        for (int j = 1; j <= maxLog; j++) {
            for (int i = 0; i + (1 << j) - 1 < n; i++) {
                table[i][j] = Math.min(table[i][j-1], table[i + (1 << (j-1))][j-1]);
            }
        }
    }
    
    // O(1) Range Minimum Query
    public int query(int L, int R) {
        int j = log[R - L + 1];
        return Math.min(table[L][j], table[R - (1 << j) + 1][j]);
    }
}
```

---

## 14. When to Use What?

```
┌──────────────────────────────────────────────────────────────┐
│           SEGMENT TREE PATTERN RECOGNITION                    │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  "Range sum + point update"         → Fenwick Tree (simpler) │
│  "Range sum + range update"         → Segment Tree + Lazy    │
│  "Range min/max + point update"     → Segment Tree           │
│  "Range min/max + NO updates"       → Sparse Table (O(1))   │
│  "Count elements in range"          → Fenwick Tree           │
│  "Count inversions / smaller after" → Fenwick Tree           │
│  "Range GCD + updates"              → Segment Tree           │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 15. Top 10 Must-Do Questions

1. Range Sum Query — Mutable (LC 307) — Basic Segment Tree / BIT
2. Count of Smaller Numbers After Self (LC 315) — BIT + coordinate compression
3. Range Sum Query 2D — Mutable (LC 308) — 2D BIT
4. My Calendar I/II/III (LC 729, 731, 732) — Segment Tree
5. Count of Range Sum (LC 327) — Merge Sort / BIT
6. Reverse Pairs (LC 493) — Merge Sort / BIT
7. The Skyline Problem (LC 218) — Sweep line + Segment Tree
8. Falling Squares (LC 699) — Segment Tree with lazy
9. Rectangle Area II (LC 850) — Sweep line + Segment Tree
10. Create Sorted Array through Instructions (LC 1649) — BIT
