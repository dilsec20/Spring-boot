# 🧠 Dynamic Programming on Trees & Rerooting — Beginner to Expert Guide

Tree DP is a design pattern used to solve optimization and counting problems on hierarchical structures (trees). Unlike grid or array DP, trees have no simple index sequence; instead, states are computed recursively using depth-first search (DFS) traversals, propagating children's sub-answers back to parent nodes in post-order.

---

## 📑 Table of Contents
1. [Core Foundations of Tree DP](#core-foundations-of-tree-dp)
2. [Problem 1: Diameter of Binary Tree (LC 543)](#problem-1-diameter-of-binary-tree-lc-543)
3. [Problem 2: House Robber III (LC 337)](#problem-2-house-robber-iii-lc-337)
4. [Problem 3: Binary Tree Maximum Path Sum (LC 124)](#problem-3-binary-tree-maximum-path-sum-lc-124)
5. [Problem 4: CSES Tree Distances I (Tree Rerooting Pattern)](#problem-4-cses-tree-distances-i-tree-rerooting-pattern)

---

## Core Foundations of Tree DP

In Tree DP:
1.  **State Definition:** `dp[u]` represents the solution for the subtree rooted at node `u`.
2.  **Order of Evaluation:** We must resolve the subproblems for all children `v` of node `u` *before* we can compute `dp[u]`. This naturally maps to a **post-order traversal** (`DFS`).
3.  **Adjacency Representation:**
    ```java
    import java.util.List;
    import java.util.ArrayList;

    class Node {
        int val;
        List<Node> children = new ArrayList<>();
    }
    ```

---

## Problem 1: Diameter of Binary Tree (LC 543)

**Problem Statement:** Given the `root` of a binary tree, return the length of the diameter of the tree. The diameter of a binary tree is the length of the longest path between any two nodes in a tree. This path may or may not pass through the root.
The length of a path between two nodes is represented by the number of edges between them.

#### 🧠 State Space Mapping
For each node `u`, we want to find:
- The longest path passing *through* `u` (this is `height(u.left) + height(u.right)`).
- We maintain a global variable `diameter` that keeps track of the maximum path found across all nodes.
- Our recursive DFS function returns `height(u)` to its parent: `1 + max(height(u.left), height(u.right))`.

#### Java Code
```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

public class TreeDiameter {
    private int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        maxDiameter = 0;
        calculateHeight(root);
        return maxDiameter;
    }

    private int calculateHeight(TreeNode node) {
        if (node == null) return 0;

        // Post-Order DFS
        int leftHeight = calculateHeight(node.left);
        int rightHeight = calculateHeight(node.right);

        // Path passing through current node
        maxDiameter = Math.max(maxDiameter, leftHeight + rightHeight);

        // Return height of subtree to parent
        return 1 + Math.max(leftHeight, rightHeight);
    }
}
```
*Time Complexity:* $O(N)$ — we visit each node exactly once.  
*Space Complexity:* $O(H)$ auxiliary stack space, where $H$ is the tree height.

---

## Problem 2: House Robber III (LC 337)

**Problem Statement:** The thief has found himself a new place for his thievery. There is only one entrance to this area, called `root`. Besides the `root`, each house has one and only one parent house. After a tour, the smart thief realized that all houses in this place form a binary tree. It will automatically contact the police if two directly-linked houses were broken into on the same night.
Given the `root` of the binary tree, return the maximum amount of money the thief can rob without alerting the police.

#### 🧠 State Space Configuration
For each node `u`, we return two values in a 2-element array `result`:
1.  `result[0]` = The maximum money collected if we **do NOT rob** node `u`.
    - If we do not rob `u`, we can choose to either rob or not rob its children.
    `result[0] = max(left[0], left[1]) + max(right[0], right[1])`
2.  `result[1]` = The maximum money collected if we **rob** node `u`.
    - If we rob `u`, we cannot rob its children.
    `result[1] = u.val + left[0] + right[0]`

#### Java Code
```java
public class HouseRobberIII {
    public int rob(TreeNode root) {
        int[] result = robHelper(root);
        return Math.max(result[0], result[1]);
    }

    private int[] robHelper(TreeNode node) {
        if (node == null) return new int[]{0, 0};

        // Post-order DFS
        int[] left = robHelper(node.left);
        int[] right = robHelper(node.right);

        int[] res = new int[2];
        
        // State 0: Do NOT rob current node
        res[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
        
        // State 1: Rob current node
        res[1] = node.val + left[0] + right[0];

        return res;
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(H)$

#### 🧠 Dry Run Simulation
Consider the following tree:
```
     3 (A)
    / \
   2 (B) 3 (C)
    \     \
     3(D)  1(E)
```

We evaluate from bottom to top:
- **Node D (3):**
  - Left child = null (0, 0), Right child = null (0, 0)
  - `res[0]` (skip D) = $\max(0,0) + \max(0,0) = 0$
  - `res[1]` (rob D) = $3 + 0 + 0 = 3$
  - Returns `[0, 3]`
- **Node E (1):**
  - Returns `[0, 1]`
- **Node B (2):**
  - Left = null `[0, 0]`, Right = Node D `[0, 3]`
  - `res[0]` (skip B) = $\max(0,0) + \max(0,3) = 3$
  - `res[1]` (rob B) = $2 + 0 + 0 = 2$
  - Returns `[3, 2]`
- **Node C (3):**
  - Left = null `[0, 0]`, Right = Node E `[0, 1]`
  - `res[0]` (skip C) = $\max(0,0) + \max(0,1) = 1$
  - `res[1]` (rob C) = $3 + 0 + 0 = 3$
  - Returns `[1, 3]`
- **Node A (3 - Root):**
  - Left = Node B `[3, 2]`, Right = Node C `[1, 3]`
  - `res[0]` (skip A) = $\max(3,2) + \max(1,3) = 3 + 3 = 6$
  - `res[1]` (rob A) = $3 + B_{\text{skip}}(3) + C_{\text{skip}}(1) = 3 + 3 + 1 = 7$

*Final Answer:* $\max(6, 7) = 7$.

---

## Problem 3: Binary Tree Maximum Path Sum (LC 124)

**Problem Statement:** A path in a binary tree is a sequence of nodes where each pair of adjacent nodes in the sequence has an edge connecting them. A node can only appear in the sequence at most once. Note that the path does not need to pass through the root.
The path sum of a path is the sum of the node's values in the path.
Given the `root` of a binary tree, return the maximum path sum of any non-empty path.

#### 🧠 State Space Mapping
For each node `u`, we calculate:
- The maximum single path extending down into its left subtree: `max(0, dfs(u.left))`.
- The maximum single path extending down into its right subtree: `max(0, dfs(u.right))`.
- We maintain a global `maxPathSum` that tests the combined path passing through `u`: `u.val + leftMax + rightMax`.
- The function returns the single path sum to its parent: `u.val + max(leftMax, rightMax)`.

#### Java Code
```java
public class MaxPathSum {
    private int globalMax = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        globalMax = Integer.MIN_VALUE;
        solve(root);
        return globalMax;
    }

    private int solve(TreeNode node) {
        if (node == null) return 0;

        // Post-Order DFS (Ignore paths that contribute negative values)
        int leftGain = Math.max(0, solve(node.left));
        int rightGain = Math.max(0, solve(node.right));

        // Calculate value of path through current node
        int currentPathSum = node.val + leftGain + rightGain;
        globalMax = Math.max(globalMax, currentPathSum);

        // Return path sum of single branch
        return node.val + Math.max(leftGain, rightGain);
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(H)$

---

## Problem 4: CSES Tree Distances I (Tree Rerooting Pattern)

**Problem Statement:** You are given a tree consisting of $N$ nodes. Your task is to determine for each node the maximum distance to another node.
*Input:* $N = 5$, Edges = `[[1, 2], [1, 3], [2, 4], [2, 5]]`  
*Output:* `[2, 2, 3, 3, 3]` (Distances for nodes 1, 2, 3, 4, 5).

#### 🧠 The Rerooting DP Concept
If we run standard DFS from a single root node, we only know distances going down. Running DFS from *every* node as root takes $O(N^2)$ time, which is too slow for $N = 2 \cdot 10^5$.
Instead, we use **Tree Rerooting** in two passes ($O(N)$):
1.  **First Pass (Bottom-Up):** We root the tree arbitrarily (e.g. at node 1). We compute `depth[u]` (height of node `u`) and track the top two largest depths from its children (`max1[u]` and `max2[u]`).
2.  **Second Pass (Top-Down):** We propagate information down from parents. For a child `v` of node `u`, the maximum distance can either be in its own subtree (`max1[v]`) OR in the rest of the tree via its parent `u`. The distance via parent is `1 + (if max1[u] goes through v, use max2[u], else use max1[u])`.

#### Java Code
```java
import java.util.ArrayList;
import java.util.List;

public class TreeDistancesI {
    private List<List<Integer>> adj;
    private int[] max1; // Largest height in subtree
    private int[] max2; // Second largest height in subtree
    private int[] c1;   // Child node yielding the largest height
    private int[] ans;  // Final max distance array

    public int[] solveTreeDistances(int n, int[][] edges) {
        adj = new ArrayList<>();
        for (int i = 0; i <= n; i++) {
            adj.add(new ArrayList<>());
        }
        for (int[] edge : edges) {
            adj.get(edge[0]).add(edge[1]);
            adj.get(edge[1]).add(edge[0]);
        }

        max1 = new int[n + 1];
        max2 = new int[n + 1];
        c1 = new int[n + 1];
        ans = new int[n + 1];

        // Pass 1: Bottom-up DFS to compute heights in subtree
        dfs1(1, 0);

        // Pass 2: Top-down DFS to compute rerooting distances
        dfs2(1, 0, 0);

        int[] finalAns = new int[n];
        for (int i = 0; i < n; i++) {
            finalAns[i] = ans[i + 1];
        }
        return finalAns;
    }

    private void dfs1(int u, int p) {
        for (int v : adj.get(u)) {
            if (v == p) continue;
            dfs1(v, u);

            int dist = max1[v] + 1;
            if (dist > max1[u]) {
                max2[u] = max1[u];
                max1[u] = dist;
                c1[u] = v;
            } else if (dist > max2[u]) {
                max2[u] = dist;
            }
        }
    }

    private void dfs2(int u, int p, int parentDist) {
        ans[u] = Math.max(max1[u], parentDist);

        for (int v : adj.get(u)) {
            if (v == p) continue;

            // If the max path of parent u passes through current child v,
            // we must use parent's second best path. Otherwise, use parent's best path.
            int bestPathFromParent = (c1[u] == v) ? max2[u] : max1[u];
            int nextParentDist = 1 + Math.max(parentDist, bestPathFromParent);

            dfs2(v, u, nextParentDist);
        }
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(N)$
