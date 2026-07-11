# 🌳 Trees (Binary Tree & BST) — FAANG Masterclass

> **"A tree is just a linked list that couldn't decide which way to go."** 

---

## 📑 Table of Contents
1. [Tree Terminology & Representation](#1-tree-terminology--representation)
2. [Tree Traversals (DFS & BFS) — With Dry Runs](#2-tree-traversals-dfs--bfs)
3. [Binary Search Tree (BST) Properties](#3-binary-search-tree-bst-properties)
4. [BST Insert & Delete](#4-bst-insert--delete)
5. [Mastering the Recursion (Height & Diameter)](#5-mastering-the-recursion-height--diameter)
6. [Balanced Binary Tree](#6-balanced-binary-tree)
7. [Lowest Common Ancestor (LCA)](#7-lowest-common-ancestor-lca)
8. [Validate BST](#8-validate-bst)
9. [Kth Smallest Element in BST](#9-kth-smallest-element-in-bst)
10. [Binary Tree Maximum Path Sum](#10-binary-tree-maximum-path-sum)
11. [Construct Tree from Preorder & Inorder](#11-construct-tree-from-preorder--inorder)
12. [Serialize & Deserialize Binary Tree](#12-serialize--deserialize-binary-tree)
13. [Tree Views (Left, Right, Top, Bottom)](#13-tree-views)
14. [Vertical Order Traversal](#14-vertical-order-traversal)
15. [Binary Tree Zigzag Level Order](#15-binary-tree-zigzag-level-order)
16. [Flatten Binary Tree to Linked List](#16-flatten-binary-tree-to-linked-list)
17. [Morris Traversal (O(1) Space Inorder)](#17-morris-traversal)
18. [Tricks to Remember & Patterns](#18-tricks-to-remember--patterns)
19. [Top 20 Must-Do Tree Questions](#19-top-20-must-do-tree-questions)

---

## 1. Tree Terminology & Representation

*   **Node:** Holds a value and pointers to children.
*   **Root:** The top node (no parent).
*   **Leaf:** A node with NO children (Left = null, Right = null).
*   **Height:** Number of edges on the longest path from the node to a leaf.
*   **Depth:** Number of edges from the root to this node.

```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    
    TreeNode(int val) {
        this.val = val;
    }
}
```

```
Example Tree:
         1
        / \
       2   3
      / \   \
     4   5   6

Nodes: 6
Height: 2 (root to leaf)
Leaves: 4, 5, 6
```

---

## 2. Tree Traversals (DFS & BFS)

### Depth-First Search (DFS)
Traversing deep down the tree using Recursion (or a Stack).

1.  **In-Order (Left, Root, Right):** Gives **sorted output** in a BST!
2.  **Pre-Order (Root, Left, Right):** Used to create a copy / serialize.
3.  **Post-Order (Left, Right, Root):** Used to delete the tree / calculate height.

### 🧠 Dry Run (All 3 DFS Traversals)

```
         1
        / \
       2   3
      / \   \
     4   5   6

In-Order (L, Root, R):
  go left(2) → go left(4) → 4 is leaf → print 4
  → back to 2 → print 2
  → go right(5) → 5 is leaf → print 5
  → back to 1 → print 1
  → go right(3) → left is null → print 3
  → go right(6) → print 6
  Result: [4, 2, 5, 1, 3, 6]

Pre-Order (Root, L, R):
  print 1 → go left → print 2 → go left → print 4
  → back → go right → print 5
  → back to 1 → go right → print 3 → go right → print 6
  Result: [1, 2, 4, 5, 3, 6]

Post-Order (L, R, Root):
  go left(4) → print 4 → go right(5) → print 5 → print 2
  → go right(6) → print 6 → print 3 → print 1
  Result: [4, 5, 2, 6, 3, 1]
```

```java
// In-Order (sorted for BST)
public void inorder(TreeNode root) {
    if (root == null) return;
    inorder(root.left);
    System.out.print(root.val + " ");
    inorder(root.right);
}

// Pre-Order
public void preorder(TreeNode root) {
    if (root == null) return;
    System.out.print(root.val + " ");
    preorder(root.left);
    preorder(root.right);
}

// Post-Order
public void postorder(TreeNode root) {
    if (root == null) return;
    postorder(root.left);
    postorder(root.right);
    System.out.print(root.val + " ");
}
```

### Iterative In-Order (Using Stack)
```java
public List<Integer> inorderIterative(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    Stack<TreeNode> stack = new Stack<>();
    TreeNode curr = root;
    
    while (curr != null || !stack.isEmpty()) {
        // Go as far left as possible
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        result.add(curr.val);  // Process node
        curr = curr.right;      // Go right
    }
    return result;
}
```

### Breadth-First Search (BFS) / Level Order Traversal
Traversing level by level. **Crucial Pattern:** Always use a `Queue`.

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();  // KEY: snapshot the level size!
        List<Integer> level = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode current = queue.poll();
            level.add(current.val);
            
            if (current.left != null) queue.offer(current.left);
            if (current.right != null) queue.offer(current.right);
        }
        result.add(level);
    }
    return result;
}
```

### 🧠 BFS Dry Run

```
         1
        / \
       2   3
      / \   \
     4   5   6

Step | Queue            | Poll | Level
-----|------------------|------|---------
  1  | [1]              | 1    | [1]
  2  | [2, 3]           | 2, 3 | [2, 3]
  3  | [4, 5, 6]        | 4,5,6| [4, 5, 6]

Result: [[1], [2, 3], [4, 5, 6]]
```

---

## 3. Binary Search Tree (BST) Properties

A BST is a binary tree with a strict rule:
*   Everything in the **Left Subtree** is `< Root`.
*   Everything in the **Right Subtree** is `> Root`.

**Key Property:** In-Order traversal of a BST gives **sorted output**.

**Search in a BST:** O(log N) average, O(N) worst case
```java
public TreeNode searchBST(TreeNode root, int val) {
    if (root == null || root.val == val) return root;
    if (val < root.val) return searchBST(root.left, val);
    else return searchBST(root.right, val);
}
```

---

## 4. BST Insert & Delete

### Insert into BST (LC 701)
```java
public TreeNode insertIntoBST(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);
    
    if (val < root.val) {
        root.left = insertIntoBST(root.left, val);
    } else {
        root.right = insertIntoBST(root.right, val);
    }
    return root;
}
```

### Delete from BST (LC 450)
3 cases:
1. **Leaf node:** Just delete.
2. **One child:** Replace with child.
3. **Two children:** Replace with **inorder successor** (smallest in right subtree), then delete the successor.

```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;
    
    if (key < root.val) {
        root.left = deleteNode(root.left, key);
    } else if (key > root.val) {
        root.right = deleteNode(root.right, key);
    } else {
        // Found the node to delete
        
        // Case 1 & 2: No left or no right child
        if (root.left == null) return root.right;
        if (root.right == null) return root.left;
        
        // Case 3: Two children — find inorder successor
        TreeNode successor = findMin(root.right);
        root.val = successor.val;  // Replace value
        root.right = deleteNode(root.right, successor.val);  // Delete successor
    }
    return root;
}

private TreeNode findMin(TreeNode node) {
    while (node.left != null) node = node.left;
    return node;
}
```

### 🧠 Delete Dry Run

```
Delete 3 from:
         5
        / \
       3   6
      / \   \
     2   4   7

Node 3 has TWO children (2, 4).
→ Find inorder successor: smallest in right subtree = 4
→ Replace 3 with 4, then delete the original 4 (leaf → simple)

Result:
         5
        / \
       4   6
      /     \
     2       7
```

---

## 5. Mastering the Recursion (Height & Diameter)

Most tree problems are solved by asking: *"What does my left child return? What does my right child return? What do I do with it?"*

### Maximum Depth / Height (LC 104)
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    
    int leftHeight = maxDepth(root.left);
    int rightHeight = maxDepth(root.right);
    
    return Math.max(leftHeight, rightHeight) + 1; // +1 for the current node
}
```

### 🧠 Height Dry Run
```
         1
        / \
       2   3
      /     
     4       

maxDepth(4): left=0, right=0 → max(0,0)+1 = 1
maxDepth(2): left=1(from 4), right=0 → max(1,0)+1 = 2
maxDepth(3): left=0, right=0 → max(0,0)+1 = 1
maxDepth(1): left=2(from 2), right=1(from 3) → max(2,1)+1 = 3

Height = 3 ✅
```

### Diameter of Binary Tree (LC 543) — O(N) trick
The diameter is the longest path between any two nodes. It may or may not pass through the root.
*Trick:* We calculate the height, but we also update a global `maxDiameter` at every node.

```java
int maxDiameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
    calculateHeight(root);
    return maxDiameter;
}

private int calculateHeight(TreeNode root) {
    if (root == null) return 0;
    
    int leftH = calculateHeight(root.left);
    int rightH = calculateHeight(root.right);
    
    // Diameter passing through CURRENT node is leftH + rightH
    maxDiameter = Math.max(maxDiameter, leftH + rightH);
    
    return Math.max(leftH, rightH) + 1;
}
```

### 🧠 Diameter Dry Run
```
         1
        / \
       2   3
      / \
     4   5

calculateHeight(4): L=0, R=0, diameter=0+0=0, return 1
calculateHeight(5): L=0, R=0, diameter=0+0=0, return 1
calculateHeight(2): L=1, R=1, diameter=1+1=2, maxDiameter=2, return 2
calculateHeight(3): L=0, R=0, diameter=0+0=0, return 1
calculateHeight(1): L=2, R=1, diameter=2+1=3, maxDiameter=3, return 3

Diameter = 3 (path: 4→2→1→3 or 5→2→1→3) ✅
```

### Symmetric Tree (LC 101)
```java
public boolean isSymmetric(TreeNode root) {
    return isMirror(root.left, root.right);
}

private boolean isMirror(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null) return true;
    if (t1 == null || t2 == null) return false;
    
    return t1.val == t2.val 
        && isMirror(t1.left, t2.right)    // Left's left vs Right's right
        && isMirror(t1.right, t2.left);   // Left's right vs Right's left
}
```

### Invert Binary Tree (LC 226)
```java
public TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    
    TreeNode temp = root.left;
    root.left = invertTree(root.right);
    root.right = invertTree(temp);
    
    return root;
}
```

---

## 6. Balanced Binary Tree

**Problem (LC 110):** Check if a binary tree is height-balanced (every node's left and right subtree heights differ by at most 1).

### 🧠 O(N) Trick
Instead of checking height at every node (O(N²)), return `-1` from height calculation if unbalanced.

```java
public boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}

private int checkHeight(TreeNode root) {
    if (root == null) return 0;
    
    int left = checkHeight(root.left);
    if (left == -1) return -1;  // Left subtree is unbalanced
    
    int right = checkHeight(root.right);
    if (right == -1) return -1; // Right subtree is unbalanced
    
    if (Math.abs(left - right) > 1) return -1;  // Current node is unbalanced
    
    return Math.max(left, right) + 1;
}
```

---

## 7. Lowest Common Ancestor (LCA)

### LCA in Binary Tree (LC 236)
**Logic:**
1. If I am `p` or `q`, return myself.
2. Search Left. Search Right.
3. If Left is NOT null and Right is NOT null → `p` is on my left and `q` is on my right → **I AM THE LCA!**
4. If only one side returns something, pass it up.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    
    if (left != null && right != null) return root; // Root is LCA
    
    return left != null ? left : right; // Pass the non-null child up
}
```

### 🧠 LCA Dry Run

```
         3
        / \
       5   1
      / \ / \
     6  2 0  8
       / \
      7   4

LCA(5, 1):
  lca(3, 5, 1):
    left = lca(5, 5, 1) → root==p → return 5 ← FOUND p on left
    right = lca(1, 5, 1) → root==q → return 1 ← FOUND q on right
    left != null AND right != null → return 3 (I AM LCA!) ✅

LCA(5, 4):
  lca(3, 5, 4):
    left = lca(5, 5, 4) → root==p → return 5
    right = lca(1, 5, 4) → eventually returns null (4 not found)
    left=5, right=null → return 5 (pass up)
  
  LCA = 5 ✅ (because 4 is a descendant of 5)
```

### LCA in BST (LC 235) — Simpler!
Use BST property: if both values < root, go left. If both > root, go right. Otherwise, root is LCA.

```java
public TreeNode lcaBST(TreeNode root, TreeNode p, TreeNode q) {
    if (p.val < root.val && q.val < root.val) {
        return lcaBST(root.left, p, q);   // Both on left
    } else if (p.val > root.val && q.val > root.val) {
        return lcaBST(root.right, p, q);  // Both on right
    } else {
        return root;  // Split point = LCA
    }
}
```

---

## 8. Validate BST

**Problem (LC 98):** Check if a binary tree is a valid BST.

### 🧠 The Pitfall
`if (left.val < root.val)` is NOT enough! The left child's rightmost grandchild could violate the rule.

**Correct approach:** Pass `MIN` and `MAX` bounds down the tree.

```java
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    
    if (node.val <= min || node.val >= max) return false;
    
    return validate(node.left, min, node.val)        // Left: max = current
        && validate(node.right, node.val, max);      // Right: min = current
}
```

### 🧠 Dry Run

```
         5
        / \
       1   7
          / \
         4   8    ← 4 < 5, so this is INVALID!

validate(5, -∞, +∞): val=5, OK
  validate(1, -∞, 5): val=1, OK ✅
  validate(7, 5, +∞): val=7, OK
    validate(4, 5, 7): val=4, 4 <= min(5) → FALSE ❌

Result: Not a valid BST ✅
(Without min/max bounds, 4 < 7 would pass incorrectly!)
```

### Alternative: In-Order Traversal Check
```java
// In a valid BST, inorder traversal is strictly increasing
TreeNode prev = null;

public boolean isValidBST(TreeNode root) {
    if (root == null) return true;
    
    if (!isValidBST(root.left)) return false;
    
    if (prev != null && root.val <= prev.val) return false;
    prev = root;
    
    return isValidBST(root.right);
}
```

---

## 9. Kth Smallest Element in BST

**Problem (LC 230):** Find the kth smallest element in a BST.

### 🧠 How to Think
In-order traversal of BST = sorted order. Just do inorder and stop at the kth element!

```java
int count = 0;
int result = 0;

public int kthSmallest(TreeNode root, int k) {
    inorder(root, k);
    return result;
}

private void inorder(TreeNode root, int k) {
    if (root == null) return;
    
    inorder(root.left, k);
    
    count++;
    if (count == k) {
        result = root.val;
        return;  // Early stop!
    }
    
    inorder(root.right, k);
}
```

### Iterative Version (Cleaner)
```java
public int kthSmallest(TreeNode root, int k) {
    Stack<TreeNode> stack = new Stack<>();
    TreeNode curr = root;
    
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        k--;
        if (k == 0) return curr.val;  // Found kth smallest!
        curr = curr.right;
    }
    return -1;  // Should never reach
}
```

---

## 10. Binary Tree Maximum Path Sum

**Problem (LC 124):** Find the maximum path sum. A path can start and end at ANY node.

### 🧠 How to Think (Global Variable Trick)
At each node, calculate the max path sum that **includes this node as the turning point** (left + node + right). But when returning to the parent, you can only choose ONE direction (left OR right, not both — you can't fork).

```java
int maxSum = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    maxGain(root);
    return maxSum;
}

private int maxGain(TreeNode node) {
    if (node == null) return 0;
    
    // Get max gain from left and right, ignore negative paths (take 0)
    int leftGain = Math.max(maxGain(node.left), 0);
    int rightGain = Math.max(maxGain(node.right), 0);
    
    // Path sum through this node as turning point
    int pathThroughNode = node.val + leftGain + rightGain;
    maxSum = Math.max(maxSum, pathThroughNode);
    
    // Return max gain for parent (can only take ONE direction)
    return node.val + Math.max(leftGain, rightGain);
}
```

### 🧠 Dry Run

```
        -10
        / \
       9   20
          / \
         15  7

maxGain(9):  L=0, R=0, path=9, return 9
maxGain(15): L=0, R=0, path=15, return 15
maxGain(7):  L=0, R=0, path=7, return 7
maxGain(20): L=15, R=7, path=20+15+7=42, maxSum=42, return 20+15=35
maxGain(-10): L=max(9,0)=9, R=max(35,0)=35, path=-10+9+35=34, maxSum=42
              return -10+35=25

Answer: 42 (path: 15→20→7) ✅
```

---

## 11. Construct Tree from Preorder & Inorder

**Problem (LC 105):** Given preorder and inorder traversal arrays, construct the binary tree.

### 🧠 How to Think
- **Preorder first element = ROOT** (always!)
- Find root in inorder → elements left of it = left subtree, right of it = right subtree
- Recursively build left and right subtrees

```java
int preIndex = 0;
Map<Integer, Integer> inorderMap = new HashMap<>();

public TreeNode buildTree(int[] preorder, int[] inorder) {
    // Build map for O(1) lookup of root position in inorder
    for (int i = 0; i < inorder.length; i++) {
        inorderMap.put(inorder[i], i);
    }
    return build(preorder, 0, inorder.length - 1);
}

private TreeNode build(int[] preorder, int inStart, int inEnd) {
    if (inStart > inEnd) return null;
    
    int rootVal = preorder[preIndex++];  // Next element in preorder = root
    TreeNode root = new TreeNode(rootVal);
    
    int inIndex = inorderMap.get(rootVal);  // Find root in inorder
    
    root.left = build(preorder, inStart, inIndex - 1);    // Left subtree
    root.right = build(preorder, inIndex + 1, inEnd);     // Right subtree
    
    return root;
}
```

### 🧠 Dry Run

```
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]

Step 1: root = preorder[0] = 3
  Find 3 in inorder at index 1
  Left subtree inorder:  [9]         (indices 0..0)
  Right subtree inorder: [15, 20, 7] (indices 2..4)

Step 2: Build left → root = preorder[1] = 9
  Find 9 in inorder at index 0
  No left or right children

Step 3: Build right → root = preorder[2] = 20
  Find 20 in inorder at index 3
  Left: [15], Right: [7]

Step 4: Build 20's left → root = preorder[3] = 15 (leaf)
Step 5: Build 20's right → root = preorder[4] = 7 (leaf)

Result:
       3
      / \
     9  20
        / \
       15  7
```

---

## 12. Serialize & Deserialize Binary Tree

**Problem (LC 297):** Design an algorithm to serialize a binary tree to a string, and deserialize it back.

### 🧠 How to Think
Use Pre-order traversal. Mark `null` nodes with a special character (e.g., `"#"` or `"null"`).

```java
public class Codec {
    
    // Serialize: preorder traversal, null = "#"
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        serializeHelper(root, sb);
        return sb.toString();
    }
    
    private void serializeHelper(TreeNode root, StringBuilder sb) {
        if (root == null) {
            sb.append("#,");
            return;
        }
        sb.append(root.val).append(",");
        serializeHelper(root.left, sb);
        serializeHelper(root.right, sb);
    }
    
    // Deserialize: rebuild from preorder
    public TreeNode deserialize(String data) {
        Queue<String> queue = new LinkedList<>(Arrays.asList(data.split(",")));
        return deserializeHelper(queue);
    }
    
    private TreeNode deserializeHelper(Queue<String> queue) {
        String val = queue.poll();
        if (val.equals("#")) return null;
        
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left = deserializeHelper(queue);
        node.right = deserializeHelper(queue);
        return node;
    }
}
```

### 🧠 Dry Run
```
Tree:   1
       / \
      2   3
         / \
        4   5

Serialize (preorder): "1,2,#,#,3,4,#,#,5,#,#,"

Deserialize:
  queue = [1, 2, #, #, 3, 4, #, #, 5, #, #]
  poll 1 → create node(1)
    left: poll 2 → create node(2)
      left: poll # → null
      right: poll # → null
    right: poll 3 → create node(3)
      left: poll 4 → create node(4)
        left: poll # → null
        right: poll # → null
      right: poll 5 → create node(5)
        left: poll # → null
        right: poll # → null

Rebuilt tree: ✅ matches original
```

---

## 13. Tree Views

### Right Side View (LC 199)
Imagine looking at the tree from the right. You see the LAST node at every level.

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode current = q.poll();
            if (i == size - 1) result.add(current.val);  // Last in level
            
            if (current.left != null) q.offer(current.left);
            if (current.right != null) q.offer(current.right);
        }
    }
    return result;
}
```

### Left Side View
Same as above but take `i == 0` (first in each level).

### 🧠 Dry Run (Right Side View)
```
         1
        / \
       2   3
      /     \
     5       4

Level 0: [1]       → last = 1
Level 1: [2, 3]    → last = 3
Level 2: [5, 4]    → last = 4

Right Side View: [1, 3, 4]
```

### DFS Approach (Space efficient)
```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    dfs(root, 0, result);
    return result;
}

private void dfs(TreeNode root, int depth, List<Integer> result) {
    if (root == null) return;
    
    if (depth == result.size()) {
        result.add(root.val);  // First node at this depth (going right first)
    }
    
    dfs(root.right, depth + 1, result);  // RIGHT first for right view
    dfs(root.left, depth + 1, result);
}
```

---

## 14. Vertical Order Traversal

**Problem (LC 987):** Group nodes by their column (vertical line). At same position, sort by value.

### 🧠 How to Think
Assign coordinates: root = (row=0, col=0). Left child = (row+1, col-1). Right child = (row+1, col+1).
Use a `TreeMap<col, TreeMap<row, PriorityQueue<val>>>` to auto-sort.

```java
public List<List<Integer>> verticalTraversal(TreeNode root) {
    TreeMap<Integer, TreeMap<Integer, PriorityQueue<Integer>>> map = new TreeMap<>();
    dfs(root, 0, 0, map);
    
    List<List<Integer>> result = new ArrayList<>();
    for (var colMap : map.values()) {
        List<Integer> column = new ArrayList<>();
        for (var pq : colMap.values()) {
            while (!pq.isEmpty()) column.add(pq.poll());
        }
        result.add(column);
    }
    return result;
}

private void dfs(TreeNode node, int row, int col, 
                 TreeMap<Integer, TreeMap<Integer, PriorityQueue<Integer>>> map) {
    if (node == null) return;
    
    map.computeIfAbsent(col, k -> new TreeMap<>())
       .computeIfAbsent(row, k -> new PriorityQueue<>())
       .offer(node.val);
    
    dfs(node.left, row + 1, col - 1, map);
    dfs(node.right, row + 1, col + 1, map);
}
```

---

## 15. Binary Tree Zigzag Level Order

**Problem (LC 103):** Level order traversal, but alternate direction each level (left→right, then right→left, etc.).

```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    boolean leftToRight = true;
    
    while (!queue.isEmpty()) {
        int size = queue.size();
        LinkedList<Integer> level = new LinkedList<>();
        
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            
            if (leftToRight) {
                level.addLast(node.val);   // Normal: add to end
            } else {
                level.addFirst(node.val);  // Reverse: add to front
            }
            
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        
        result.add(level);
        leftToRight = !leftToRight;  // Toggle direction
    }
    return result;
}
```

---

## 16. Flatten Binary Tree to Linked List

**Problem (LC 114):** Flatten tree to a linked list in-place (preorder). Use right pointers as next, left pointers = null.

```java
// Reverse post-order approach (right, left, root)
TreeNode prev = null;

public void flatten(TreeNode root) {
    if (root == null) return;
    
    flatten(root.right);   // Process right first
    flatten(root.left);    // Then left
    
    root.right = prev;     // Point right to previously processed node
    root.left = null;      // Clear left
    prev = root;           // Update prev
}
```

### 🧠 Dry Run
```
         1
        / \
       2   5
      / \   \
     3   4   6

flatten(6): prev = 6
flatten(5): right=prev=6, left=null, prev = 5 → 5→6
flatten(4): right=prev=5, left=null, prev = 4 → 4→5→6
flatten(3): right=prev=4, left=null, prev = 3 → 3→4→5→6
flatten(2): right=prev=3, left=null, prev = 2 → 2→3→4→5→6
flatten(1): right=prev=2, left=null, prev = 1 → 1→2→3→4→5→6 ✅
```

---

## 17. Morris Traversal (O(1) Space Inorder)

Regular inorder uses O(H) stack space. Morris Traversal achieves **O(1) space** by temporarily modifying tree pointers (threading).

### 🧠 How It Works
For each node:
1. If no left child → visit node, go right.
2. If left child exists → find **inorder predecessor** (rightmost node in left subtree).
   - If predecessor's right is null → set it to current node (create thread), go left.
   - If predecessor's right is current node → remove thread, visit node, go right.

```java
public List<Integer> morrisInorder(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    TreeNode curr = root;
    
    while (curr != null) {
        if (curr.left == null) {
            result.add(curr.val);  // Visit
            curr = curr.right;
        } else {
            // Find inorder predecessor
            TreeNode predecessor = curr.left;
            while (predecessor.right != null && predecessor.right != curr) {
                predecessor = predecessor.right;
            }
            
            if (predecessor.right == null) {
                // Create thread
                predecessor.right = curr;
                curr = curr.left;
            } else {
                // Remove thread (already visited left subtree)
                predecessor.right = null;
                result.add(curr.val);  // Visit
                curr = curr.right;
            }
        }
    }
    return result;
}
```

---

## 18. Tricks to Remember & Patterns

1.  **Level Tracking:** When doing BFS, always extract `size = q.size()` and do a `for` loop. This isolates each level perfectly.
2.  **Global Variable Trick:** When a function needs to return height, but you also need to find Diameter / Max Path Sum, use a global variable to track the max while the function returns height.
3.  **Validate BST Pitfall:** Pass `MIN` and `MAX` bounds down the tree, not just check immediate children.
4.  **Serialize/Deserialize:** Use Pre-order traversal with `"#"` for null nodes.
5.  **BST + Inorder = Sorted:** Many BST problems reduce to "do inorder traversal and apply array logic."
6.  **Height -1 Trick:** For balanced tree check, return `-1` as a sentinel for unbalanced instead of checking height separately.
7.  **Right/Left View:** BFS with first/last element, OR DFS with depth tracking.
8.  **Construct Tree:** Preorder first element = root. Find in inorder to split left/right subtrees.

```
┌───────────────────────────────────────────────────────┐
│              TREE PATTERN RECOGNITION                  │
├───────────────────────────────────────────────────────┤
│                                                        │
│  "Height / Depth"             → Recursion: max+1      │
│  "Diameter / Max Path Sum"    → Global var + height   │
│  "Level by level"             → BFS with Queue        │
│  "Validate BST"               → min/max bounds DFS   │
│  "Kth smallest in BST"        → Inorder + counter    │
│  "LCA"                        → Find p/q from both   │
│  "Serialize"                  → Preorder + "#"        │
│  "Right/Left view"            → BFS last/first       │
│  "Construct tree"             → Preorder root + map  │
│  "Flatten"                    → Reverse post-order   │
│  "O(1) space traversal"       → Morris Traversal     │
│                                                        │
└───────────────────────────────────────────────────────┘
```

---

## 19. Top 20 Must-Do Tree Questions

**Basic Recursion:**
1. Invert Binary Tree (LC 226)
2. Maximum Depth of Binary Tree (LC 104)
3. Diameter of Binary Tree (LC 543)
4. Balanced Binary Tree (LC 110)
5. Same Tree / Symmetric Tree (LC 100, 101)
6. Subtree of Another Tree (LC 572)

**BST Operations:**
7. Validate BST (LC 98)
8. Kth Smallest Element in BST (LC 230)
9. Insert / Delete in BST (LC 701, 450)

**LCA:**
10. LCA of Binary Tree (LC 236)
11. LCA of BST (LC 235)

**BFS / Level Order:**
12. Level Order Traversal (LC 102)
13. Binary Tree Right Side View (LC 199)
14. Binary Tree Zigzag Level Order (LC 103)

**Construction / Serialization:**
15. Construct from Preorder & Inorder (LC 105)
16. Serialize & Deserialize Binary Tree (LC 297)

**Hard:**
17. Binary Tree Maximum Path Sum (LC 124)
18. Vertical Order Traversal (LC 987)
19. Flatten Binary Tree to Linked List (LC 114)
20. Count Good Nodes in Binary Tree (LC 1448)
