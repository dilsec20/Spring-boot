# 🏗️ Design Problems — FAANG Masterclass

> **"LRU Cache is asked at every single Big Tech company. If you can't code it in 15 minutes, you're not ready."**

---

## 📑 Table of Contents
1. [Why Design Problems?](#1-why-design-problems)
2. [LRU Cache (LC 146) — Full Dry Run](#2-lru-cache)
3. [LFU Cache (LC 460)](#3-lfu-cache)
4. [Min Stack (LC 155)](#4-min-stack)
5. [Design HashMap (LC 706)](#5-design-hashmap)
6. [Find Median from Data Stream (LC 295)](#6-find-median-from-data-stream)
7. [Merge K Sorted Lists (LC 23)](#7-merge-k-sorted-lists)
8. [Top K Frequent Elements (LC 347)](#8-top-k-frequent-elements)
9. [Kth Largest Element in Stream (LC 703)](#9-kth-largest-element-in-stream)
10. [Design Twitter (LC 355)](#10-design-twitter)
11. [Insert Delete GetRandom O(1) (LC 380)](#11-insert-delete-getrandom)
12. [Design Browser History (LC 1472)](#12-design-browser-history)
13. [Time Based Key-Value Store (LC 981)](#13-time-based-key-value-store)
14. [Heap / Priority Queue Internals](#14-heap-internals)
15. [Pattern Recognition](#15-pattern-recognition)
16. [Top 15 Must-Do Questions](#16-top-15-must-do)

---

## 1. Why Design Problems?

Design problems test your ability to:
- **Combine multiple data structures** into one cohesive system
- Achieve specific **time complexity** guarantees
- Handle **edge cases** (empty cache, duplicate inserts, etc.)
- Write **clean, organized** code under pressure

**Most common pattern:** HashMap + Some ordered structure (LinkedList, Heap, TreeMap)

---

## 2. LRU Cache (LC 146)

**Problem:** Design a cache with `get(key)` and `put(key, value)` in O(1). When capacity is exceeded, evict the **Least Recently Used** item.

### 🧠 How to Think
We need:
- **O(1) lookup by key** → HashMap
- **O(1) insert/delete + track order** → Doubly Linked List

HashMap maps `key → node`. Doubly Linked List maintains usage order (most recent at head, least recent at tail).

```
HashMap: {1: Node1, 2: Node2, 3: Node3}

Doubly Linked List (most recent → least recent):
  HEAD ↔ Node3 ↔ Node1 ↔ Node2 ↔ TAIL
  (most recent)              (least recent = evict first)
```

### Complete Implementation

```java
class LRUCache {
    
    // Doubly Linked List Node
    class Node {
        int key, value;
        Node prev, next;
        
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private int capacity;
    private Map<Integer, Node> map;
    private Node head, tail; // Dummy head and tail
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        
        // Initialize dummy head and tail
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }
    
    // ═══ GET: O(1) ═══
    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        
        Node node = map.get(key);
        removeNode(node);     // Remove from current position
        addToHead(node);      // Move to head (most recently used)
        return node.value;
    }
    
    // ═══ PUT: O(1) ═══
    public void put(int key, int value) {
        if (map.containsKey(key)) {
            // Key exists → update value and move to head
            Node node = map.get(key);
            node.value = value;
            removeNode(node);
            addToHead(node);
        } else {
            // New key
            if (map.size() == capacity) {
                // Evict LRU (node before tail)
                Node lru = tail.prev;
                removeNode(lru);
                map.remove(lru.key);  // MUST remove from map too!
            }
            Node newNode = new Node(key, value);
            map.put(key, newNode);
            addToHead(newNode);
        }
    }
    
    // ═══ Helper: Add node right after HEAD ═══
    private void addToHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
    
    // ═══ Helper: Remove a node from its current position ═══
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
}
```

### 🧠 Full Dry Run

```
LRUCache cache = new LRUCache(2);  // capacity = 2

═══ put(1, 1) ═══
  Map empty, capacity not reached.
  Create Node(1,1), add to head.
  Map: {1: Node(1,1)}
  List: HEAD ↔ [1:1] ↔ TAIL

═══ put(2, 2) ═══
  Create Node(2,2), add to head.
  Map: {1: Node(1,1), 2: Node(2,2)}
  List: HEAD ↔ [2:2] ↔ [1:1] ↔ TAIL

═══ get(1) ═══  → returns 1
  Found! Remove [1:1] from current position, add to head.
  List: HEAD ↔ [1:1] ↔ [2:2] ↔ TAIL
  (Node 1 is now most recently used)

═══ put(3, 3) ═══  → capacity exceeded!
  Map.size() == 2 == capacity → EVICT LRU!
  LRU = tail.prev = Node(2,2) → remove from list AND map.
  Map: {1: Node(1,1)}
  
  Create Node(3,3), add to head.
  Map: {1: Node(1,1), 3: Node(3,3)}
  List: HEAD ↔ [3:3] ↔ [1:1] ↔ TAIL

═══ get(2) ═══  → returns -1 (evicted!)

═══ put(4, 4) ═══  → capacity exceeded!
  Evict LRU = tail.prev = Node(1,1)
  Map: {3: Node(3,3), 4: Node(4,4)}
  List: HEAD ↔ [4:4] ↔ [3:3] ↔ TAIL

═══ get(1) ═══  → returns -1 (evicted!)
═══ get(3) ═══  → returns 3 ✅
═══ get(4) ═══  → returns 4 ✅
```

---

## 3. LFU Cache (LC 460)

**Problem:** Like LRU, but evict the **Least Frequently Used** item. If tie in frequency, evict the **least recently used** among them.

### 🧠 How to Think
We need to track:
1. **Key → Value** (and frequency): `HashMap<Key, Node>`
2. **Frequency → List of keys**: `HashMap<Freq, LinkedHashSet<Key>>` (insertion order = LRU)
3. **Minimum frequency**: `int minFreq`

```java
class LFUCache {
    int capacity, minFreq;
    Map<Integer, int[]> keyMap;  // key → [value, freq]
    Map<Integer, LinkedHashSet<Integer>> freqMap; // freq → set of keys (ordered)
    
    public LFUCache(int capacity) {
        this.capacity = capacity;
        this.minFreq = 0;
        this.keyMap = new HashMap<>();
        this.freqMap = new HashMap<>();
    }
    
    public int get(int key) {
        if (!keyMap.containsKey(key)) return -1;
        
        int[] val = keyMap.get(key);
        int value = val[0], freq = val[1];
        
        // Update frequency
        updateFreq(key, freq);
        val[1] = freq + 1;
        
        return value;
    }
    
    public void put(int key, int value) {
        if (capacity == 0) return;
        
        if (keyMap.containsKey(key)) {
            int[] val = keyMap.get(key);
            val[0] = value;
            get(key); // Update frequency (reuse get logic)
            val[0] = value; // Ensure value is set
            return;
        }
        
        if (keyMap.size() == capacity) {
            // Evict LFU (and LRU among ties)
            LinkedHashSet<Integer> minSet = freqMap.get(minFreq);
            int evictKey = minSet.iterator().next(); // First = LRU
            minSet.remove(evictKey);
            if (minSet.isEmpty()) freqMap.remove(minFreq);
            keyMap.remove(evictKey);
        }
        
        // Insert new key with freq 1
        keyMap.put(key, new int[]{value, 1});
        freqMap.computeIfAbsent(1, k -> new LinkedHashSet<>()).add(key);
        minFreq = 1; // New key always has freq 1
    }
    
    private void updateFreq(int key, int freq) {
        LinkedHashSet<Integer> set = freqMap.get(freq);
        set.remove(key);
        if (set.isEmpty()) {
            freqMap.remove(freq);
            if (minFreq == freq) minFreq++; // Update min!
        }
        freqMap.computeIfAbsent(freq + 1, k -> new LinkedHashSet<>()).add(key);
    }
}
```

---

## 4. Min Stack (LC 155)

**Problem:** Design a stack that supports `push`, `pop`, `top`, and `getMin` in **O(1) time**.

### 🧠 How to Think
Maintain two stacks: one for values and one for the current minimum.

```java
class MinStack {
    Stack<Integer> stack;
    Stack<Integer> minStack; // Tracks the minimum at each level
    
    public MinStack() {
        stack = new Stack<>();
        minStack = new Stack<>();
    }
    
    public void push(int val) {
        stack.push(val);
        int currentMin = minStack.isEmpty() ? val : Math.min(val, minStack.peek());
        minStack.push(currentMin);
    }
    
    public void pop() {
        stack.pop();
        minStack.pop();
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int getMin() {
        return minStack.peek();
    }
}
```

### 🧠 Dry Run
```
push(5):  stack=[5],    minStack=[5]    → min=5
push(3):  stack=[5,3],  minStack=[5,3]  → min=3
push(7):  stack=[5,3,7],minStack=[5,3,3]→ min=3
getMin(): return minStack.peek() = 3 ✅
pop():    stack=[5,3],  minStack=[5,3]  → min=3
pop():    stack=[5],    minStack=[5]    → min=5
getMin(): return 5 ✅
```

### Single Stack Approach (Space Optimized)
```java
class MinStack {
    Stack<long[]> stack; // {value, currentMin}
    
    public MinStack() { stack = new Stack<>(); }
    
    public void push(int val) {
        long min = stack.isEmpty() ? val : Math.min(val, stack.peek()[1]);
        stack.push(new long[]{val, min});
    }
    
    public void pop() { stack.pop(); }
    public int top() { return (int) stack.peek()[0]; }
    public int getMin() { return (int) stack.peek()[1]; }
}
```

---

## 5. Design HashMap (LC 706)

**Problem:** Implement a HashMap without using any built-in hash library.

### 🧠 How to Think
Use **Array of Linked Lists** (Separate Chaining). Hash function maps key to a bucket index. Collisions are handled by chaining.

```java
class MyHashMap {
    private static final int SIZE = 10007; // Prime number for better distribution
    private LinkedList<int[]>[] buckets;
    
    public MyHashMap() {
        buckets = new LinkedList[SIZE];
    }
    
    private int hash(int key) {
        return key % SIZE;
    }
    
    public void put(int key, int value) {
        int idx = hash(key);
        if (buckets[idx] == null) buckets[idx] = new LinkedList<>();
        
        for (int[] pair : buckets[idx]) {
            if (pair[0] == key) {
                pair[1] = value; // Update existing
                return;
            }
        }
        buckets[idx].add(new int[]{key, value}); // Insert new
    }
    
    public int get(int key) {
        int idx = hash(key);
        if (buckets[idx] == null) return -1;
        
        for (int[] pair : buckets[idx]) {
            if (pair[0] == key) return pair[1];
        }
        return -1;
    }
    
    public void remove(int key) {
        int idx = hash(key);
        if (buckets[idx] == null) return;
        
        buckets[idx].removeIf(pair -> pair[0] == key);
    }
}
```

---

## 6. Find Median from Data Stream (LC 295)

**Problem:** Design a class that continuously receives numbers and returns the median at any time.

### 🧠 How to Think (Two Heaps!)
- **Max-Heap** for the smaller half (gives us the max of the smaller half)
- **Min-Heap** for the larger half (gives us the min of the larger half)
- Balance: `|maxHeap.size - minHeap.size| ≤ 1`
- Median = top of max-heap (if odd) or average of both tops (if even)

```java
class MedianFinder {
    PriorityQueue<Integer> maxHeap; // Smaller half (max on top)
    PriorityQueue<Integer> minHeap; // Larger half (min on top)
    
    public MedianFinder() {
        maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        minHeap = new PriorityQueue<>();
    }
    
    public void addNum(int num) {
        maxHeap.add(num); // Always add to max-heap first
        
        // Balance: ensure max-heap top ≤ min-heap top
        minHeap.add(maxHeap.poll()); // Move max's top to min
        
        // Ensure max-heap has equal or one more element
        if (minHeap.size() > maxHeap.size()) {
            maxHeap.add(minHeap.poll());
        }
    }
    
    public double findMedian() {
        if (maxHeap.size() > minHeap.size()) {
            return maxHeap.peek();
        }
        return (maxHeap.peek() + minHeap.peek()) / 2.0;
    }
}
```

### 🧠 Dry Run
```
addNum(1):
  maxHeap=[1] → move 1 to minHeap → minHeap=[1]
  minHeap.size > maxHeap.size → move 1 back → maxHeap=[1], minHeap=[]
  
addNum(2):
  maxHeap=[2,1] → move 2 to minHeap → maxHeap=[1], minHeap=[2]
  sizes equal → OK
  median = (1+2)/2 = 1.5

addNum(3):
  maxHeap=[3,1] → move 3 to minHeap → maxHeap=[1], minHeap=[2,3]
  minHeap.size > maxHeap.size → move 2 to maxHeap
  maxHeap=[2,1], minHeap=[3]
  median = 2 (maxHeap.peek()) ✅

State:
  maxHeap (smaller half): [2, 1] → peek = 2
  minHeap (larger half):  [3]    → peek = 3
  Median = 2 ✅
```

---

## 7. Merge K Sorted Lists (LC 23)

**Problem:** Merge `k` sorted linked lists into one sorted list.

### 🧠 How to Think
Use a **Min-Heap** of size K. Always pop the smallest, then push its next node.

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    
    // Add the head of each list to the heap
    for (ListNode head : lists) {
        if (head != null) pq.add(head);
    }
    
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (!pq.isEmpty()) {
        ListNode smallest = pq.poll();
        current.next = smallest;
        current = current.next;
        
        if (smallest.next != null) {
            pq.add(smallest.next); // Add next node from the same list
        }
    }
    
    return dummy.next;
}
```

### 🧠 Dry Run
```
lists = [1→4→5, 1→3→4, 2→6]

PQ (min-heap): [1(list1), 1(list2), 2(list3)]

Poll 1(list1) → result: 1 → push 4(list1). PQ: [1(list2), 2(list3), 4(list1)]
Poll 1(list2) → result: 1→1 → push 3(list2). PQ: [2(list3), 3(list2), 4(list1)]
Poll 2(list3) → result: 1→1→2 → push 6(list3). PQ: [3(list2), 4(list1), 6(list3)]
Poll 3(list2) → result: 1→1→2→3 → push 4(list2). PQ: [4(list1), 4(list2), 6(list3)]
Poll 4(list1) → result: ...→3→4 → push 5(list1)
Poll 4(list2) → result: ...→4→4 → null next
Poll 5(list1) → result: ...→4→5 → null next
Poll 6(list3) → result: ...→5→6 → null next

Result: 1→1→2→3→4→4→5→6 ✅
Time: O(N log K) where N = total nodes, K = number of lists
```

### Divide & Conquer Approach (O(N log K), no extra space)
```java
public ListNode mergeKLists(ListNode[] lists) {
    if (lists.length == 0) return null;
    return mergeRange(lists, 0, lists.length - 1);
}

private ListNode mergeRange(ListNode[] lists, int lo, int hi) {
    if (lo == hi) return lists[lo];
    int mid = lo + (hi - lo) / 2;
    ListNode left = mergeRange(lists, lo, mid);
    ListNode right = mergeRange(lists, mid + 1, hi);
    return mergeTwoLists(left, right);
}

private ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

---

## 8. Top K Frequent Elements (LC 347)

**Problem:** Given array, return the `k` most frequent elements.

### Approach 1: Min-Heap of Size K — O(N log K)
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);
    
    // Min-heap of size k (keeps the k largest frequencies)
    PriorityQueue<Integer> pq = new PriorityQueue<>((a, b) -> freq.get(a) - freq.get(b));
    
    for (int num : freq.keySet()) {
        pq.add(num);
        if (pq.size() > k) pq.poll(); // Remove least frequent
    }
    
    int[] result = new int[k];
    for (int i = 0; i < k; i++) result[i] = pq.poll();
    return result;
}
```

### Approach 2: Bucket Sort — O(N) 🏆
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);
    
    // Bucket: index = frequency, value = list of numbers with that frequency
    List<Integer>[] buckets = new List[nums.length + 1];
    for (int num : freq.keySet()) {
        int f = freq.get(num);
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(num);
    }
    
    // Collect from highest frequency
    int[] result = new int[k];
    int idx = 0;
    for (int i = buckets.length - 1; i >= 0 && idx < k; i--) {
        if (buckets[i] != null) {
            for (int num : buckets[i]) {
                result[idx++] = num;
                if (idx == k) break;
            }
        }
    }
    return result;
}
```

---

## 9. Kth Largest Element in Stream (LC 703)

```java
class KthLargest {
    PriorityQueue<Integer> minHeap;
    int k;
    
    public KthLargest(int k, int[] nums) {
        this.k = k;
        this.minHeap = new PriorityQueue<>();
        for (int num : nums) add(num);
    }
    
    public int add(int val) {
        minHeap.add(val);
        if (minHeap.size() > k) minHeap.poll(); // Keep only top k
        return minHeap.peek(); // Smallest of top k = kth largest
    }
}
```

---

## 10. Design Twitter (LC 355)

**Problem:** Design a simplified Twitter with `postTweet`, `getNewsFeed` (10 most recent from followed users), `follow`, `unfollow`.

### 🧠 How to Think
- Each user has a **list of tweets** (timestamped)
- Each user has a **set of followed users**
- `getNewsFeed` = **Merge K sorted lists** (each user's tweet list) — use min-heap!

```java
class Twitter {
    private int timestamp = 0;
    private Map<Integer, List<int[]>> tweets;     // userId → [{time, tweetId}]
    private Map<Integer, Set<Integer>> following;  // userId → set of followed users
    
    public Twitter() {
        tweets = new HashMap<>();
        following = new HashMap<>();
    }
    
    public void postTweet(int userId, int tweetId) {
        tweets.computeIfAbsent(userId, k -> new ArrayList<>())
              .add(new int[]{timestamp++, tweetId});
    }
    
    public List<Integer> getNewsFeed(int userId) {
        // Max-heap to get 10 most recent
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[0] - a[0]);
        
        // Add own tweets
        if (tweets.containsKey(userId)) {
            for (int[] tweet : tweets.get(userId)) pq.add(tweet);
        }
        
        // Add followed users' tweets
        if (following.containsKey(userId)) {
            for (int followee : following.get(userId)) {
                if (tweets.containsKey(followee)) {
                    for (int[] tweet : tweets.get(followee)) pq.add(tweet);
                }
            }
        }
        
        // Get top 10
        List<Integer> feed = new ArrayList<>();
        int count = 0;
        while (!pq.isEmpty() && count < 10) {
            feed.add(pq.poll()[1]);
            count++;
        }
        return feed;
    }
    
    public void follow(int followerId, int followeeId) {
        if (followerId == followeeId) return;
        following.computeIfAbsent(followerId, k -> new HashSet<>()).add(followeeId);
    }
    
    public void unfollow(int followerId, int followeeId) {
        if (following.containsKey(followerId)) {
            following.get(followerId).remove(followeeId);
        }
    }
}
```

---

## 11. Insert Delete GetRandom O(1) (LC 380)

**Problem:** Design a data structure with `insert`, `remove`, and `getRandom` all in O(1).

### 🧠 How to Think
- **ArrayList** for O(1) random access → `getRandom`
- **HashMap** for O(1) lookup → `insert`/`remove`
- **Trick for O(1) remove:** Swap element to delete with the LAST element, then remove last.

```java
class RandomizedSet {
    List<Integer> list;
    Map<Integer, Integer> map; // value → index in list
    Random rand;
    
    public RandomizedSet() {
        list = new ArrayList<>();
        map = new HashMap<>();
        rand = new Random();
    }
    
    public boolean insert(int val) {
        if (map.containsKey(val)) return false;
        map.put(val, list.size());
        list.add(val);
        return true;
    }
    
    public boolean remove(int val) {
        if (!map.containsKey(val)) return false;
        
        int idx = map.get(val);
        int lastVal = list.get(list.size() - 1);
        
        // Swap with last element
        list.set(idx, lastVal);
        map.put(lastVal, idx);
        
        // Remove last element
        list.remove(list.size() - 1);
        map.remove(val);
        
        return true;
    }
    
    public int getRandom() {
        return list.get(rand.nextInt(list.size()));
    }
}
```

### 🧠 Dry Run
```
insert(1): list=[1], map={1:0} ✅
insert(2): list=[1,2], map={1:0, 2:1} ✅
insert(3): list=[1,2,3], map={1:0, 2:1, 3:2} ✅

remove(2):
  idx=1, lastVal=3
  Swap: list[1]=3 → list=[1,3,3]
  Update map: map[3]=1
  Remove last: list=[1,3], map={1:0, 3:1}
  Remove map[2]
  ✅ O(1)!

getRandom(): return list[random(0,1)] → 1 or 3 with equal probability ✅
```

---

## 12. Design Browser History (LC 1472)

```java
class BrowserHistory {
    List<String> history;
    int current;
    
    public BrowserHistory(String homepage) {
        history = new ArrayList<>();
        history.add(homepage);
        current = 0;
    }
    
    public void visit(String url) {
        // Clear forward history
        while (history.size() > current + 1) {
            history.remove(history.size() - 1);
        }
        history.add(url);
        current++;
    }
    
    public String back(int steps) {
        current = Math.max(0, current - steps);
        return history.get(current);
    }
    
    public String forward(int steps) {
        current = Math.min(history.size() - 1, current + steps);
        return history.get(current);
    }
}
```

---

## 13. Time Based Key-Value Store (LC 981)

**Problem:** Store key-value pairs with timestamps. `get(key, timestamp)` returns the value with the largest timestamp ≤ given timestamp.

### 🧠 How to Think
Store values with timestamps (auto-sorted). Use **binary search** on timestamps.

```java
class TimeMap {
    Map<String, List<int[]>> map; // key → [{timestamp, valueIndex}]
    Map<String, List<String>> values;
    
    // Cleaner approach using TreeMap
    Map<String, TreeMap<Integer, String>> store;
    
    public TimeMap() {
        store = new HashMap<>();
    }
    
    public void set(String key, String value, int timestamp) {
        store.computeIfAbsent(key, k -> new TreeMap<>()).put(timestamp, value);
    }
    
    public String get(String key, int timestamp) {
        if (!store.containsKey(key)) return "";
        
        TreeMap<Integer, String> tree = store.get(key);
        Integer floor = tree.floorKey(timestamp); // Largest key ≤ timestamp
        
        return floor == null ? "" : tree.get(floor);
    }
}
```

---

## 14. Heap / Priority Queue Internals

Understanding the heap is critical for design problems.

### Min-Heap Properties
```
Parent: index i
Left child: 2*i + 1
Right child: 2*i + 2
Parent of i: (i-1) / 2

Min-Heap: parent ≤ both children

Array: [1, 3, 5, 7, 9, 8, 6]

        1
       / \
      3   5
     / \ / \
    7  9 8  6
```

### Heap Operations Dry Run
```
═══ INSERT 2 into [1, 3, 5, 7, 9, 8, 6] ═══

Step 1: Add at end → [1, 3, 5, 7, 9, 8, 6, 2]
Step 2: Bubble UP (sift up):
  idx=7, parent=(7-1)/2=3, arr[3]=7 > 2 → swap → [1, 3, 5, 2, 9, 8, 6, 7]
  idx=3, parent=(3-1)/2=1, arr[1]=3 > 2 → swap → [1, 2, 5, 3, 9, 8, 6, 7]
  idx=1, parent=0, arr[0]=1 ≤ 2 → STOP
  
Result: [1, 2, 5, 3, 9, 8, 6, 7]

═══ EXTRACT MIN from [1, 2, 5, 3, 9, 8, 6, 7] ═══

Step 1: Save min = arr[0] = 1
Step 2: Move last to root → [7, 2, 5, 3, 9, 8, 6]
Step 3: Bubble DOWN (sift down):
  idx=0, children: left=2, right=5 → min child=2 at idx=1
  7 > 2 → swap → [2, 7, 5, 3, 9, 8, 6]
  idx=1, children: left=3, right=9 → min child=3 at idx=3
  7 > 3 → swap → [2, 3, 5, 7, 9, 8, 6]
  idx=3, no children → STOP

Result: [2, 3, 5, 7, 9, 8, 6], returned 1

Time: O(log N) for both insert and extract
```

### When to use Max-Heap vs Min-Heap
```
Min-Heap (default PriorityQueue):
  - Kth LARGEST → keep K elements, min on top = Kth largest
  - Merge K sorted lists → always get the minimum

Max-Heap (Collections.reverseOrder()):
  - Kth SMALLEST → keep K elements, max on top = Kth smallest
  - Median finder → smaller half uses max-heap
```

---

## 15. Pattern Recognition

```
┌──────────────────────────────────────────────────────────────┐
│           DESIGN PATTERN RECOGNITION                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  "O(1) get + O(1) put + eviction"    → HashMap + DLL        │
│    LRU: evict least recently used    → Order by access time  │
│    LFU: evict least frequently used  → Track frequencies     │
│                                                               │
│  "O(1) push/pop + O(1) getMin"       → Two stacks           │
│  "O(1) insert + O(1) delete + O(1) random"                   │
│    → ArrayList + HashMap (swap trick)                        │
│                                                               │
│  "Find median continuously"          → Two heaps            │
│  "Merge K sorted things"             → Min-heap of size K   │
│  "Top K / Kth largest"               → Min-heap of size K   │
│  "Kth smallest"                      → Max-heap of size K   │
│                                                               │
│  COMMON DATA STRUCTURE COMBOS:                                │
│  HashMap + Doubly Linked List  → LRU Cache                   │
│  HashMap + ArrayList           → Insert/Delete/Random O(1)  │
│  HashMap + TreeMap             → Time-based key-value        │
│  Two Heaps (max + min)         → Running median             │
│  HashMap + LinkedHashSet       → LFU Cache                   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 16. Top 15 Must-Do Questions

**Cache Design:**
1. LRU Cache (LC 146) — HashMap + DLL
2. LFU Cache (LC 460) — HashMap + Freq map

**Stack/Queue Design:**
3. Min Stack (LC 155) — Two stacks
4. Implement Queue using Stacks (LC 232)
5. Design Circular Queue (LC 622)

**Heap Problems:**
6. Find Median from Data Stream (LC 295) — Two heaps
7. Merge K Sorted Lists (LC 23) — Min-heap
8. Top K Frequent Elements (LC 347) — Heap or bucket sort
9. Kth Largest Element (LC 215) — Quickselect or heap
10. Kth Largest in Stream (LC 703) — Min-heap size K

**Data Structure Design:**
11. Insert Delete GetRandom O(1) (LC 380) — ArrayList + Map
12. Design HashMap (LC 706) — Chaining
13. Design Twitter (LC 355) — Heap + graph
14. Time Based Key-Value Store (LC 981) — TreeMap
15. Design Browser History (LC 1472) — List + pointer
