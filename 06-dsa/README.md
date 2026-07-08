# 🧮 Data Structures & Algorithms (DSA) — Complete In-Depth Guide

> **"Data structures and algorithms are the building blocks of efficient software. They separate good developers from great ones, and are essential for cracking technical interviews."**

---

## 📑 Table of Contents

1. [Introduction & Big O Notation](#1-introduction--big-o-notation)
2. [Arrays & Strings](#2-arrays--strings)
3. [Linked Lists](#3-linked-lists)
4. [Stacks & Queues](#4-stacks--queues)
5. [Hash Maps & Hash Sets](#5-hash-maps--hash-sets)
6. [Trees](#6-trees)
7. [Binary Search Trees (BST)](#7-binary-search-trees-bst)
8. [Heaps & Priority Queues](#8-heaps--priority-queues)
9. [Graphs](#9-graphs)
10. [Sorting Algorithms](#10-sorting-algorithms)
11. [Searching Algorithms](#11-searching-algorithms)
12. [Recursion & Backtracking](#12-recursion--backtracking)
13. [Dynamic Programming](#13-dynamic-programming)
14. [Greedy Algorithms](#14-greedy-algorithms)
15. [Two Pointers & Sliding Window](#15-two-pointers--sliding-window)
16. [Java Collections for DSA](#16-java-collections-for-dsa)
17. [Best Practices & Problem-Solving Patterns](#17-best-practices--problem-solving-patterns)
18. [Interview Questions & Answers (50+)](#18-interview-questions--answers-50)

---

## 1. Introduction & Big O Notation

### Why DSA Matters for Spring Boot Developers?

- **Interview essential** — Most tech interviews require DSA knowledge
- **Performance** — Choose the right data structure for your Spring services
- **Database queries** — Understanding indexing, hashing, and trees helps optimize queries
- **Caching strategies** — LRU caches use LinkedHashMap (doubly linked list + hashmap)
- **API design** — Pagination, sorting, filtering all involve algorithmic thinking
- **System design** — Load balancing, consistent hashing, rate limiting

### Big O Notation

Big O describes the **worst-case** time/space complexity as input grows.

```
Common complexities (fastest to slowest):

O(1)        — Constant      — HashMap get/put, array access
O(log n)    — Logarithmic   — Binary search, balanced BST operations
O(n)        — Linear        — Single loop, linear search
O(n log n)  — Linearithmic  — Merge sort, efficient sorts
O(n²)       — Quadratic     — Nested loops, bubble sort
O(2ⁿ)       — Exponential   — Recursive fibonacci (naive)
O(n!)       — Factorial     — Permutations, traveling salesman (brute force)
```

### Complexity Comparison Chart

```
Time (operations)
│
│  O(n!)
│  │
│  │    O(2ⁿ)
│  │    │
│  │    │     O(n²)
│  │    │     │
│  │    │     │       O(n log n)
│  │    │     │       │
│  │    │     │       │     O(n)
│  │    │     │       │     │
│  │    │     │       │     │    O(log n)
│  │    │     │       │     │    │   O(1)
│  │    │     │       │     │    │   │
└──┴────┴─────┴───────┴─────┴────┴───┴────── n (input size)
```

### How to Calculate Big O

```java
// O(1) — Constant
int getFirst(int[] arr) {
    return arr[0]; // Always one operation regardless of array size
}

// O(n) — Linear
int sum(int[] arr) {
    int total = 0;
    for (int num : arr) {   // n iterations
        total += num;
    }
    return total;
}

// O(n²) — Quadratic
void printPairs(int[] arr) {
    for (int i = 0; i < arr.length; i++) {      // n
        for (int j = 0; j < arr.length; j++) {  // n
            System.out.println(arr[i] + ", " + arr[j]);
        }
    }
} // n × n = O(n²)

// O(log n) — Logarithmic
int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
} // Halves the search space each iteration
```

### Space Complexity

```java
// O(1) space — uses constant extra space
void swap(int[] arr, int i, int j) {
    int temp = arr[i]; // Only one extra variable
    arr[i] = arr[j];
    arr[j] = temp;
}

// O(n) space — creates array proportional to input
int[] duplicate(int[] arr) {
    int[] result = new int[arr.length]; // Same size as input
    System.arraycopy(arr, 0, result, 0, arr.length);
    return result;
}

// O(n) space — recursive call stack
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1); // n stack frames
}
```

---

## 2. Arrays & Strings

### Array Operations & Complexity

| Operation | Array | ArrayList |
|-----------|-------|-----------|
| Access by index | O(1) | O(1) |
| Search (unsorted) | O(n) | O(n) |
| Search (sorted) | O(log n) | O(log n) |
| Insert at end | O(1)* | O(1) amortized |
| Insert at beginning | O(n) | O(n) |
| Delete at end | O(1) | O(1) |
| Delete at beginning | O(n) | O(n) |

### Common Array Problems

```java
// 1. Two Sum — Find two numbers that add up to target
// O(n) time, O(n) space — HashMap approach
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[]{};
}

// 2. Maximum Subarray (Kadane's Algorithm)
// O(n) time, O(1) space
public int maxSubArray(int[] nums) {
    int maxSum = nums[0];
    int currentSum = nums[0];
    
    for (int i = 1; i < nums.length; i++) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}

// 3. Rotate Array by K positions
// O(n) time, O(1) space — Reverse approach
public void rotate(int[] nums, int k) {
    k = k % nums.length;
    reverse(nums, 0, nums.length - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, nums.length - 1);
}

private void reverse(int[] nums, int start, int end) {
    while (start < end) {
        int temp = nums[start];
        nums[start] = nums[end];
        nums[end] = temp;
        start++;
        end--;
    }
}

// 4. Remove Duplicates from Sorted Array (In-Place)
// O(n) time, O(1) space
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int slow = 0;
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) {
            slow++;
            nums[slow] = nums[fast];
        }
    }
    return slow + 1;
}

// 5. Reverse a String In-Place
public void reverseString(char[] s) {
    int left = 0, right = s.length - 1;
    while (left < right) {
        char temp = s[left];
        s[left] = s[right];
        s[right] = temp;
        left++;
        right--;
    }
}

// 6. Valid Anagram
// O(n) time, O(1) space (26 characters)
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        count[t.charAt(i) - 'a']--;
    }
    for (int c : count) {
        if (c != 0) return false;
    }
    return true;
}
```

---

## 3. Linked Lists

### Implementation

```java
// Singly Linked List
public class ListNode {
    int val;
    ListNode next;
    
    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}

public class LinkedList {
    private ListNode head;
    private int size;
    
    // Add at beginning — O(1)
    public void addFirst(int val) {
        ListNode newNode = new ListNode(val);
        newNode.next = head;
        head = newNode;
        size++;
    }
    
    // Add at end — O(n)
    public void addLast(int val) {
        ListNode newNode = new ListNode(val);
        if (head == null) {
            head = newNode;
        } else {
            ListNode current = head;
            while (current.next != null) {
                current = current.next;
            }
            current.next = newNode;
        }
        size++;
    }
    
    // Delete by value — O(n)
    public void delete(int val) {
        if (head == null) return;
        if (head.val == val) {
            head = head.next;
            size--;
            return;
        }
        ListNode current = head;
        while (current.next != null && current.next.val != val) {
            current = current.next;
        }
        if (current.next != null) {
            current.next = current.next.next;
            size--;
        }
    }
    
    // Search — O(n)
    public boolean contains(int val) {
        ListNode current = head;
        while (current != null) {
            if (current.val == val) return true;
            current = current.next;
        }
        return false;
    }
}
```

### Common Linked List Problems

```java
// 1. Reverse a Linked List — O(n) time, O(1) space
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    while (current != null) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    return prev;
}

// 2. Detect Cycle (Floyd's Algorithm) — O(n) time, O(1) space
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;          // Move 1 step
        fast = fast.next.next;     // Move 2 steps
        if (slow == fast) return true;  // They meet = cycle exists
    }
    return false;
}

// 3. Find Middle Node — O(n) time, O(1) space
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow; // slow is at middle when fast reaches end
}

// 4. Merge Two Sorted Lists — O(n+m) time, O(1) space
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            current.next = l1;
            l1 = l1.next;
        } else {
            current.next = l2;
            l2 = l2.next;
        }
        current = current.next;
    }
    current.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}

// 5. Remove Nth Node From End — O(n) time, O(1) space
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy, slow = dummy;
    
    for (int i = 0; i <= n; i++) fast = fast.next; // Move fast n+1 ahead
    
    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    slow.next = slow.next.next; // Skip the nth node
    return dummy.next;
}
```

---

## 4. Stacks & Queues

### Stack (LIFO — Last In, First Out)

```java
// Using Deque (preferred over Stack class)
Deque<Integer> stack = new ArrayDeque<>();

stack.push(1);      // Push: O(1)
stack.push(2);
stack.push(3);

stack.peek();       // Peek (top element without removing): O(1) → 3
stack.pop();        // Pop (remove top): O(1) → 3
stack.isEmpty();    // Check empty: O(1)
stack.size();       // Size: O(1)
```

### Common Stack Problems

```java
// 1. Valid Parentheses
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> map = Map.of(')', '(', '}', '{', ']', '[');
    
    for (char c : s.toCharArray()) {
        if (map.containsValue(c)) {
            stack.push(c);
        } else if (map.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != map.get(c)) {
                return false;
            }
        }
    }
    return stack.isEmpty();
}

// 2. Min Stack — O(1) for all operations
class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();
    
    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek()) {
            minStack.push(val);
        }
    }
    
    public void pop() {
        if (stack.pop().equals(minStack.peek())) {
            minStack.pop();
        }
    }
    
    public int top() { return stack.peek(); }
    public int getMin() { return minStack.peek(); }
}
```

### Queue (FIFO — First In, First Out)

```java
// Using LinkedList or ArrayDeque
Queue<Integer> queue = new LinkedList<>();

queue.offer(1);     // Enqueue: O(1)
queue.offer(2);
queue.offer(3);

queue.peek();       // Peek (front element): O(1) → 1
queue.poll();       // Dequeue (remove front): O(1) → 1
queue.isEmpty();    // Check empty: O(1)

// Deque (Double-Ended Queue)
Deque<Integer> deque = new ArrayDeque<>();
deque.offerFirst(1);   // Add to front
deque.offerLast(2);    // Add to back
deque.pollFirst();     // Remove from front
deque.pollLast();      // Remove from back
```

---

## 5. Hash Maps & Hash Sets

### HashMap Internals

```
HashMap uses an array of buckets.

1. key.hashCode() → bucket index (hash & (capacity - 1))
2. Store key-value pair in that bucket
3. Collision → linked list (or Red-Black tree when >= 8 entries)

Default capacity: 16
Default load factor: 0.75
Resize: doubles capacity when size > capacity × loadFactor
```

### Common Hash Map Problems

```java
// 1. First Non-Repeating Character — O(n)
public int firstUniqChar(String s) {
    Map<Character, Integer> count = new LinkedHashMap<>();
    for (char c : s.toCharArray()) {
        count.merge(c, 1, Integer::sum);
    }
    for (int i = 0; i < s.length(); i++) {
        if (count.get(s.charAt(i)) == 1) return i;
    }
    return -1;
}

// 2. Group Anagrams — O(n × k log k)
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}

// 3. LRU Cache — O(1) for get and put
class LRUCache {
    private final int capacity;
    private final LinkedHashMap<Integer, Integer> cache;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new LinkedHashMap<>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > capacity;
            }
        };
    }
    
    public int get(int key) {
        return cache.getOrDefault(key, -1);
    }
    
    public void put(int key, int value) {
        cache.put(key, value);
    }
}
```

---

## 6. Trees

### Binary Tree

```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    
    TreeNode(int val) {
        this.val = val;
    }
}
```

### Tree Traversals

```java
// 1. Inorder (Left → Root → Right) — gives sorted order for BST
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    result.addAll(inorderTraversal(root.left));
    result.add(root.val);
    result.addAll(inorderTraversal(root.right));
    return result;
}

// 2. Preorder (Root → Left → Right)
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    result.add(root.val);
    result.addAll(preorderTraversal(root.left));
    result.addAll(preorderTraversal(root.right));
    return result;
}

// 3. Postorder (Left → Right → Root)
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    result.addAll(postorderTraversal(root.left));
    result.addAll(postorderTraversal(root.right));
    result.add(root.val);
    return result;
}

// 4. Level Order (BFS)
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> level = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

### Common Tree Problems

```java
// 1. Maximum Depth — O(n)
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// 2. Invert Binary Tree — O(n)
public TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    TreeNode temp = root.left;
    root.left = invertTree(root.right);
    root.right = invertTree(temp);
    return root;
}

// 3. Is Symmetric Tree — O(n)
public boolean isSymmetric(TreeNode root) {
    return isMirror(root, root);
}

private boolean isMirror(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null) return true;
    if (t1 == null || t2 == null) return false;
    return t1.val == t2.val
        && isMirror(t1.left, t2.right)
        && isMirror(t1.right, t2.left);
}

// 4. Path Sum — O(n)
public boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) return false;
    if (root.left == null && root.right == null) return targetSum == root.val;
    return hasPathSum(root.left, targetSum - root.val) 
        || hasPathSum(root.right, targetSum - root.val);
}

// 5. Lowest Common Ancestor (LCA) — O(n)
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root;
    return left != null ? left : right;
}
```

---

## 7. Binary Search Trees (BST)

### BST Property

For every node: **left subtree values < node value < right subtree values**

```java
// BST Operations
// Search — O(log n) average, O(n) worst
public TreeNode search(TreeNode root, int val) {
    if (root == null || root.val == val) return root;
    if (val < root.val) return search(root.left, val);
    return search(root.right, val);
}

// Insert — O(log n) average
public TreeNode insert(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);
    if (val < root.val) root.left = insert(root.left, val);
    else if (val > root.val) root.right = insert(root.right, val);
    return root;
}

// Validate BST — O(n)
public boolean isValidBST(TreeNode root) {
    return isValid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean isValid(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return isValid(node.left, min, node.val) 
        && isValid(node.right, node.val, max);
}

// Kth Smallest Element — O(n)
public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;
    int count = 0;
    
    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }
        current = stack.pop();
        count++;
        if (count == k) return current.val;
        current = current.right;
    }
    return -1;
}
```

---

## 8. Heaps & Priority Queues

### Min-Heap / Max-Heap

```java
// Min-Heap (default in Java)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);    // O(log n)
minHeap.offer(3);
minHeap.offer(7);
minHeap.peek();      // O(1) → 3 (smallest)
minHeap.poll();      // O(log n) → 3

// Max-Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(5);
maxHeap.offer(3);
maxHeap.offer(7);
maxHeap.peek();      // → 7 (largest)

// Custom comparator
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
```

### Common Heap Problems

```java
// 1. Kth Largest Element — O(n log k)
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll(); // Remove smallest — keep k largest
        }
    }
    return minHeap.peek(); // k-th largest
}

// 2. Top K Frequent Elements — O(n log k)
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    for (int n : nums) count.merge(n, 1, Integer::sum);
    
    PriorityQueue<Integer> heap = new PriorityQueue<>(
        Comparator.comparingInt(count::get)
    );
    
    for (int num : count.keySet()) {
        heap.offer(num);
        if (heap.size() > k) heap.poll();
    }
    
    return heap.stream().mapToInt(Integer::intValue).toArray();
}

// 3. Merge K Sorted Lists — O(n log k) where n = total elements, k = lists
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(
        Comparator.comparingInt(a -> a.val)
    );
    
    for (ListNode list : lists) {
        if (list != null) pq.offer(list);
    }
    
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (!pq.isEmpty()) {
        ListNode node = pq.poll();
        current.next = node;
        current = current.next;
        if (node.next != null) pq.offer(node.next);
    }
    return dummy.next;
}
```

---

## 9. Graphs

### Graph Representations

```java
// 1. Adjacency List (most common, space-efficient for sparse graphs)
Map<Integer, List<Integer>> graph = new HashMap<>();
graph.computeIfAbsent(0, k -> new ArrayList<>()).add(1);
graph.computeIfAbsent(0, k -> new ArrayList<>()).add(2);
graph.computeIfAbsent(1, k -> new ArrayList<>()).add(3);

// 2. Adjacency Matrix (good for dense graphs)
int[][] matrix = new int[n][n];
matrix[0][1] = 1; // Edge from 0 to 1
matrix[0][2] = 1; // Edge from 0 to 2
```

### BFS and DFS

```java
// BFS — Breadth-First Search (uses Queue)
// O(V + E) time, O(V) space
public List<Integer> bfs(Map<Integer, List<Integer>> graph, int start) {
    List<Integer> result = new ArrayList<>();
    Set<Integer> visited = new HashSet<>();
    Queue<Integer> queue = new LinkedList<>();
    
    visited.add(start);
    queue.offer(start);
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        result.add(node);
        
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.offer(neighbor);
            }
        }
    }
    return result;
}

// DFS — Depth-First Search (uses Stack/Recursion)
// O(V + E) time, O(V) space
public void dfs(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
    visited.add(node);
    System.out.println(node);
    
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        if (!visited.contains(neighbor)) {
            dfs(graph, neighbor, visited);
        }
    }
}

// Number of Islands — BFS/DFS on grid
public int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == '1') {
                count++;
                dfsGrid(grid, i, j);
            }
        }
    }
    return count;
}

private void dfsGrid(char[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == '0') return;
    grid[i][j] = '0'; // Mark as visited
    dfsGrid(grid, i + 1, j);
    dfsGrid(grid, i - 1, j);
    dfsGrid(grid, i, j + 1);
    dfsGrid(grid, i, j - 1);
}

// Detect Cycle in Directed Graph (using DFS with coloring)
public boolean hasCycle(int n, List<List<Integer>> adj) {
    int[] color = new int[n]; // 0=white, 1=gray, 2=black
    for (int i = 0; i < n; i++) {
        if (color[i] == 0 && dfsHasCycle(i, adj, color)) return true;
    }
    return false;
}

private boolean dfsHasCycle(int u, List<List<Integer>> adj, int[] color) {
    color[u] = 1; // Gray (in progress)
    for (int v : adj.get(u)) {
        if (color[v] == 1) return true; // Back edge = cycle
        if (color[v] == 0 && dfsHasCycle(v, adj, color)) return true;
    }
    color[u] = 2; // Black (done)
    return false;
}
```

### Shortest Path — Dijkstra's Algorithm

```java
// O((V + E) log V) with priority queue
public int[] dijkstra(Map<Integer, List<int[]>> graph, int src, int n) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    pq.offer(new int[]{src, 0});
    
    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int u = curr[0], d = curr[1];
        
        if (d > dist[u]) continue; // Already found shorter path
        
        for (int[] edge : graph.getOrDefault(u, List.of())) {
            int v = edge[0], weight = edge[1];
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.offer(new int[]{v, dist[v]});
            }
        }
    }
    return dist;
}
```

---

## 10. Sorting Algorithms

### Comparison of Sorting Algorithms

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ |

### Implementations

```java
// Merge Sort — O(n log n) time, O(n) space, STABLE
public void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

private void merge(int[] arr, int left, int mid, int right) {
    int[] leftArr = Arrays.copyOfRange(arr, left, mid + 1);
    int[] rightArr = Arrays.copyOfRange(arr, mid + 1, right + 1);
    
    int i = 0, j = 0, k = left;
    while (i < leftArr.length && j < rightArr.length) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k++] = leftArr[i++];
        } else {
            arr[k++] = rightArr[j++];
        }
    }
    while (i < leftArr.length) arr[k++] = leftArr[i++];
    while (j < rightArr.length) arr[k++] = rightArr[j++];
}

// Quick Sort — O(n log n) average, O(n²) worst
public void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

private int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    return i + 1;
}
```

---

## 11. Searching Algorithms

### Binary Search

```java
// Iterative Binary Search — O(log n)
public int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2; // Avoid overflow
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// Find First and Last Position of Element
public int[] searchRange(int[] nums, int target) {
    return new int[]{findBound(nums, target, true), findBound(nums, target, false)};
}

private int findBound(int[] nums, int target, boolean isFirst) {
    int left = 0, right = nums.length - 1, result = -1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            result = mid;
            if (isFirst) right = mid - 1; else left = mid + 1;
        } else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return result;
}

// Search in Rotated Sorted Array
public int searchRotated(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        
        if (nums[left] <= nums[mid]) { // Left half is sorted
            if (target >= nums[left] && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else { // Right half is sorted
            if (target > nums[mid] && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return -1;
}
```

---

## 12. Recursion & Backtracking

### Backtracking Template

```java
// General backtracking template:
void backtrack(result, currentState, choices) {
    if (isGoalReached(currentState)) {
        result.add(copy(currentState));
        return;
    }
    for (choice : choices) {
        if (isValid(choice)) {
            make(choice);                    // Choose
            backtrack(result, currentState, remainingChoices);  // Explore
            undo(choice);                    // Unchoose (backtrack)
        }
    }
}

// Subsets — O(2^n)
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackSubsets(result, new ArrayList<>(), nums, 0);
    return result;
}

private void backtrackSubsets(List<List<Integer>> result, List<Integer> current, int[] nums, int start) {
    result.add(new ArrayList<>(current)); // Add every state as a valid subset
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);           // Choose
        backtrackSubsets(result, current, nums, i + 1); // Explore
        current.remove(current.size() - 1); // Unchoose
    }
}

// Permutations — O(n!)
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackPermute(result, new ArrayList<>(), nums, new boolean[nums.length]);
    return result;
}

private void backtrackPermute(List<List<Integer>> result, List<Integer> current, int[] nums, boolean[] used) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        current.add(nums[i]);
        backtrackPermute(result, current, nums, used);
        current.remove(current.size() - 1);
        used[i] = false;
    }
}

// N-Queens — O(n!)
public List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    char[][] board = new char[n][n];
    for (char[] row : board) Arrays.fill(row, '.');
    backtrackQueens(result, board, 0);
    return result;
}

private void backtrackQueens(List<List<String>> result, char[][] board, int row) {
    if (row == board.length) {
        result.add(construct(board));
        return;
    }
    for (int col = 0; col < board.length; col++) {
        if (isSafe(board, row, col)) {
            board[row][col] = 'Q';
            backtrackQueens(result, board, row + 1);
            board[row][col] = '.';
        }
    }
}
```

---

## 13. Dynamic Programming

### DP Approach

```
1. Define the STATE (what changes between subproblems)
2. Define the RECURRENCE RELATION (how states relate)
3. Define BASE CASES
4. Build the solution (top-down with memoization or bottom-up with tabulation)
```

### Classic DP Problems

```java
// 1. Fibonacci — O(n) time, O(n) space (memoization)
public int fib(int n) {
    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}

// Space-optimized: O(1) space
public int fibOptimized(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}

// 2. Climbing Stairs — O(n)
// How many distinct ways to climb n stairs (1 or 2 steps at a time)?
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}

// 3. Coin Change — O(amount × coins.length)
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}

// 4. Longest Common Subsequence — O(m × n)
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}

// 5. 0/1 Knapsack — O(n × W)
public int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[][] dp = new int[n + 1][W + 1];
    
    for (int i = 1; i <= n; i++) {
        for (int w = 1; w <= W; w++) {
            dp[i][w] = dp[i - 1][w]; // Don't take item i
            if (weights[i - 1] <= w) {
                dp[i][w] = Math.max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
            }
        }
    }
    return dp[n][W];
}

// 6. Longest Increasing Subsequence — O(n log n)
public int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();
    for (int num : nums) {
        int pos = Collections.binarySearch(tails, num);
        if (pos < 0) pos = -(pos + 1);
        if (pos == tails.size()) tails.add(num);
        else tails.set(pos, num);
    }
    return tails.size();
}
```

---

## 14. Greedy Algorithms

```java
// 1. Activity Selection — O(n log n)
public int maxActivities(int[][] activities) {
    Arrays.sort(activities, (a, b) -> a[1] - b[1]); // Sort by end time
    int count = 1;
    int lastEnd = activities[0][1];
    for (int i = 1; i < activities.length; i++) {
        if (activities[i][0] >= lastEnd) {
            count++;
            lastEnd = activities[i][1];
        }
    }
    return count;
}

// 2. Jump Game — O(n)
public boolean canJump(int[] nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > maxReach) return false;
        maxReach = Math.max(maxReach, i + nums[i]);
    }
    return true;
}

// 3. Best Time to Buy and Sell Stock — O(n)
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE;
    int maxProfit = 0;
    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }
    return maxProfit;
}
```

---

## 15. Two Pointers & Sliding Window

### Two Pointers

```java
// Container With Most Water — O(n)
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxWater = 0;
    while (left < right) {
        int water = Math.min(height[left], height[right]) * (right - left);
        maxWater = Math.max(maxWater, water);
        if (height[left] < height[right]) left++;
        else right--;
    }
    return maxWater;
}

// 3Sum — O(n²)
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // Skip duplicates
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(List.of(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                left++;
                right--;
            } else if (sum < 0) left++;
            else right--;
        }
    }
    return result;
}
```

### Sliding Window

```java
// Longest Substring Without Repeating Characters — O(n)
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int maxLen = 0, left = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (map.containsKey(c)) {
            left = Math.max(left, map.get(c) + 1);
        }
        map.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}

// Maximum Sum Subarray of Size K — O(n)
public int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0, maxSum = 0;
    for (int i = 0; i < arr.length; i++) {
        windowSum += arr[i];
        if (i >= k - 1) {
            maxSum = Math.max(maxSum, windowSum);
            windowSum -= arr[i - k + 1];
        }
    }
    return maxSum;
}
```

---

## 16. Java Collections for DSA

### Quick Reference

| Need | Use | Operations |
|------|-----|------------|
| Dynamic array | `ArrayList` | O(1) access, O(1) amortized add |
| Linked list | `LinkedList` | O(1) add/remove at ends |
| Stack | `ArrayDeque` | push, pop, peek: O(1) |
| Queue | `LinkedList` or `ArrayDeque` | offer, poll, peek: O(1) |
| Set (unordered) | `HashSet` | O(1) add, remove, contains |
| Set (sorted) | `TreeSet` | O(log n) add, remove, contains |
| Map (unordered) | `HashMap` | O(1) get, put |
| Map (sorted) | `TreeMap` | O(log n) get, put |
| Map (insertion order) | `LinkedHashMap` | O(1) get, put |
| Priority Queue | `PriorityQueue` | O(log n) offer, O(1) peek |
| Bit manipulation | `BitSet` | Compact boolean array |

---

## 17. Best Practices & Problem-Solving Patterns

### Pattern Recognition

| Pattern | Use When | Examples |
|---------|----------|---------|
| Two Pointers | Sorted array, pair finding | 2Sum, 3Sum, Container With Most Water |
| Sliding Window | Subarray/substring problems | Max sum subarray, longest substring |
| Binary Search | Sorted data, optimization | Search rotated array, peak finding |
| BFS | Shortest path, level order | Level order traversal, shortest path |
| DFS | Exhaustive search, paths | Permutations, island counting |
| Dynamic Programming | Overlapping subproblems, optimal | Knapsack, LCS, coin change |
| Backtracking | Constraint satisfaction | N-Queens, Sudoku solver |
| Greedy | Locally optimal = globally optimal | Activity selection, Huffman |
| Heap/Priority Queue | Top K, scheduling | K largest, merge K lists |
| Stack | Matching, nesting, monotonic | Valid parentheses, next greater element |

### Problem-Solving Steps

```
1. UNDERSTAND the problem (read 2-3 times, identify inputs/outputs)
2. EXAMPLES — walk through examples, edge cases
3. BRUTE FORCE — think of the simplest solution first
4. OPTIMIZE — can you use a pattern? Better data structure?
5. CODE — write clean, readable code
6. TEST — trace through examples, test edge cases
```

---

## 18. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is the difference between Array and ArrayList?**
**A:** Array has fixed size, stores primitives and objects. ArrayList is dynamically resizable, stores only objects (auto-boxing for primitives). ArrayList provides methods like add, remove, contains. Array uses `[]` syntax.

**Q2: What is Big O notation?**
**A:** Big O describes the upper bound (worst case) of an algorithm's time or space complexity as input grows. It ignores constants and lower-order terms. O(n) means time grows linearly with input size.

**Q3: What is the difference between Stack and Queue?**
**A:** Stack is LIFO (Last In, First Out) — push/pop. Queue is FIFO (First In, First Out) — enqueue/dequeue. Stack: undo operations, recursion. Queue: BFS, task scheduling.

**Q4: What is a Linked List?**
**A:** A linear data structure where elements (nodes) are not stored contiguously. Each node contains data and a reference to the next node. Types: singly linked, doubly linked, circular.

**Q5: What is the time complexity of binary search?**
**A:** O(log n) — each step halves the search space. Requires sorted input. Much faster than linear search O(n) for large datasets.

**Q6: What is a Hash Map?**
**A:** A data structure that maps keys to values using a hash function. Average O(1) for get/put/remove. Java's HashMap handles collisions with chaining (linked list → tree at 8+ entries).

**Q7: What is recursion?**
**A:** A function that calls itself to solve smaller instances of the same problem. Every recursion needs a base case (stopping condition) and a recursive case (reduces the problem size).

**Q8: What is the difference between BFS and DFS?**
**A:** BFS explores level by level (uses Queue). DFS explores as deep as possible first (uses Stack/recursion). BFS finds shortest path in unweighted graphs. DFS is good for exploring all paths.

---

### Intermediate Level

**Q9: What is Dynamic Programming?**
**A:** An optimization technique that solves complex problems by breaking them into overlapping subproblems and storing solutions (memoization/tabulation) to avoid redundant computation. Two approaches: top-down (recursion + memo) and bottom-up (tabulation).

**Q10: Explain merge sort vs quick sort.**
**A:** Merge sort: O(n log n) always, stable, O(n) space. Quick sort: O(n log n) average, O(n²) worst, not stable, O(log n) space. Quick sort is faster in practice due to cache locality. Java uses TimSort (hybrid merge + insertion sort) for Arrays.sort().

**Q11: What is a Binary Search Tree?**
**A:** A binary tree where left subtree values < node < right subtree values. Enables O(log n) search, insert, delete in balanced BSTs. Inorder traversal gives sorted order.

**Q12: What is a heap?**
**A:** A complete binary tree where parent is always greater (max-heap) or smaller (min-heap) than children. Used for priority queues, top-K problems, heap sort.

**Q13: What is a Graph?**
**A:** A non-linear data structure consisting of vertices (nodes) and edges (connections). Types: directed/undirected, weighted/unweighted, cyclic/acyclic. Representations: adjacency list, adjacency matrix.

**Q14: What is the sliding window technique?**
**A:** A technique for solving problems involving subarrays/substrings. Maintain a window [left, right] that slides through the data. Expand right to include, shrink left to exclude. Reduces O(n²) brute force to O(n).

**Q15: What is backtracking?**
**A:** An algorithmic technique that incrementally builds candidates for a solution and abandons (backtracks) when a candidate cannot lead to a valid solution. Used for permutations, combinations, sudoku, N-Queens.

---

### Advanced Level

**Q16: Explain Dijkstra's algorithm.**
**A:** Finds shortest path from source to all other vertices in a weighted graph with non-negative edges. Uses a priority queue (min-heap). Greedily selects the nearest unvisited vertex. Time: O((V+E) log V).

**Q17: What is a Trie?**
**A:** A tree-like data structure for efficient string prefix operations. Each node represents a character. O(L) search/insert where L is word length. Used for autocomplete, spell checking, IP routing.

**Q18: What is topological sorting?**
**A:** Linear ordering of vertices in a DAG (Directed Acyclic Graph) such that for every edge (u,v), u comes before v. Uses: build systems, task scheduling, dependency resolution. Implemented with DFS or Kahn's algorithm (BFS).

**Q19: What is the Union-Find (Disjoint Set Union) data structure?**
**A:** Tracks elements partitioned into disjoint sets. Supports: `find(x)` — which set contains x; `union(x,y)` — merge two sets. With path compression + union by rank: nearly O(1) amortized. Used for: Kruskal's MST, cycle detection, connected components.

**Q20: Explain the two approaches to Dynamic Programming.**
**A:** **Top-Down (Memoization)**: Recursive approach with a cache. Start from the final state, break into subproblems. **Bottom-Up (Tabulation)**: Iterative approach building a table. Start from base cases, build up. Bottom-up is usually faster (no recursion overhead) and uses less stack space.

---

### Rapid-Fire (Q21–Q50)

**Q21: What is O(1) space complexity?** Uses constant extra memory regardless of input size.

**Q22: How does HashMap handle collisions?** Chaining (linked list at each bucket). Java 8+: when bucket has 8+ entries, list → Red-Black tree.

**Q23: What is a balanced BST?** A BST where the height is O(log n). Examples: AVL tree, Red-Black tree, B-tree.

**Q24: What is stable sorting?** Maintains the relative order of elements with equal keys. Merge sort is stable; quick sort is not.

**Q25: What is an in-place algorithm?** Uses O(1) extra space (modifies input directly). Examples: quick sort, selection sort.

**Q26: What is memoization?** Caching results of expensive function calls to return cached result when the same inputs occur again.

**Q27: What is the Floyd cycle detection algorithm?** Uses slow and fast pointers. Slow moves 1 step, fast moves 2 steps. If they meet, there's a cycle. O(n) time, O(1) space.

**Q28: What is a monotonic stack?** A stack that maintains elements in monotonically increasing or decreasing order. Used for "next greater element" problems.

**Q29: What is amortized analysis?** The average cost of operations over a sequence. ArrayList.add() is O(1) amortized even though occasional resizing is O(n).

**Q30: What is tail recursion?** A recursive call that is the last operation in the function. Can be optimized by compilers to avoid stack overflow. Java does NOT optimize tail calls.

**Q31: Array vs LinkedList: when to use each?** Array: frequent random access, known size. LinkedList: frequent insert/delete at ends, no random access needed.

**Q32: What is a circular buffer?** A fixed-size buffer that wraps around. Uses two pointers (head, tail). Used in producer-consumer scenarios, buffered I/O.

**Q33: What is the time complexity of HashMap operations?** O(1) average for get/put/remove. O(n) worst case (all keys hash to same bucket — extremely rare with good hash function).

**Q34: What is counting sort?** A non-comparison sort that counts occurrences of each value. O(n+k) where k is the range. Only works for integers with limited range.

**Q35: What is the difference between tree and graph?** A tree is a special graph that is connected and acyclic with exactly n-1 edges for n nodes. Graphs can have cycles, can be disconnected, and can have any number of edges.

**Q36: What is a complete binary tree?** All levels are fully filled except possibly the last, which is filled from left to right. Used for heaps.

**Q37: What is Kadane's algorithm?** Finds maximum subarray sum in O(n). Maintains current sum and max sum. Reset current sum when it goes negative.

**Q38: What is the difference between greedy and DP?** Greedy makes locally optimal choices (faster, doesn't always work). DP considers all possibilities (slower, always optimal when applicable).

**Q39: What is a segment tree?** A tree for range queries (sum, min, max) with O(log n) query and update. Used in competitive programming and databases.

**Q40: What is radix sort?** Non-comparison sort that sorts by digits (LSD or MSD). O(d × (n+k)) where d is digits, k is the base. Very fast for integers.

**Q41: What is the Master Theorem?** Solves recurrences of the form T(n) = aT(n/b) + O(n^d). Used to analyze divide-and-conquer algorithms.

**Q42: What is the difference between BFS and Dijkstra?** BFS: unweighted graphs, O(V+E). Dijkstra: weighted graphs (non-negative), O((V+E)log V). BFS uses queue; Dijkstra uses priority queue.

**Q43: What is a spanning tree?** A subgraph that includes all vertices with minimum edges (n-1) and no cycles. MST (Minimum Spanning Tree) minimizes total edge weight.

**Q44: What is Kruskal's algorithm?** Finds MST by sorting edges by weight and greedily adding them if they don't form a cycle (using Union-Find). O(E log E).

**Q45: What is an AVL tree?** A self-balancing BST where the height difference between left and right subtrees is at most 1. Uses rotations to maintain balance after insertions/deletions.

**Q46: What is a deque?** Double-Ended Queue that supports insertion and deletion at both ends in O(1). Java: ArrayDeque.

**Q47: What is consistent hashing?** Distributes data across nodes on a ring. When a node is added/removed, only K/n keys need redistribution. Used in distributed systems.

**Q48: What is the two-pointer technique?** Using two pointers that move through data structure simultaneously. Types: same direction (fast/slow), opposite directions (left/right).

**Q49: What is bit manipulation and when is it useful?** Operating on individual bits using AND, OR, XOR, NOT, shifts. O(1) operations. Useful for: checking power of 2 (`n & (n-1) == 0`), counting bits, XOR for finding missing numbers.

**Q50: How does Java's Arrays.sort() work internally?** Uses dual-pivot quicksort for primitives (O(n log n) average). Uses TimSort (merge sort + insertion sort hybrid) for objects (O(n log n), stable). Switches to insertion sort for small arrays (< 47 elements).

---

## 📚 References & Further Reading

- [LeetCode](https://leetcode.com/) — Practice platform
- [NeetCode Roadmap](https://neetcode.io/roadmap) — Structured problem list
- [Blind 75 / NeetCode 150](https://neetcode.io/) — Must-do problems
- [GeeksforGeeks DSA](https://www.geeksforgeeks.org/data-structures/) — Theory + problems
- [Algorithms by Robert Sedgewick](https://algs4.cs.princeton.edu/) — Textbook
- [Introduction to Algorithms (CLRS)](https://mitpress.mit.edu/books/introduction-algorithms) — Bible of algorithms
- [Visualgo](https://visualgo.net/) — Algorithm visualization

---

> **Previous Topic:** [← 05 - Git](../05-git/README.md)  
> **Next Topic:** [07 - JDBC →](../07-jdbc/README.md)
