# 🧮 Data Structures & Algorithms (DSA) — Complete In-Depth Guide

> **"Data structures and algorithms are the building blocks of efficient software. They separate good developers from great ones, and are essential for cracking technical interviews."**

---

## 📑 Table of Contents

1. [Introduction & Big O Notation](#1-introduction--big-o-notation)
2. [Arrays & Strings](#2-arrays--strings)
3. [Linked Lists](#3-linked-lists)
4. [Stacks & Queues](#4-stacks--queues)
5. [Hash Maps & Hash Sets](#5-hash-maps--hash-sets)
6. [Trees — Visual Deep Dive](#6-trees--visual-deep-dive)
7. [Binary Search Trees (BST)](#7-binary-search-trees-bst)
8. [Heaps & Priority Queues](#8-heaps--priority-queues)
9. [Graphs](#9-graphs)
10. [Sorting Algorithms](#10-sorting-algorithms)
11. [Searching Algorithms](#11-searching-algorithms)
12. [Recursion — Visual Deep Dive](#12-recursion--visual-deep-dive)
13. [Dynamic Programming — Visual Deep Dive](#13-dynamic-programming--visual-deep-dive)
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

### Big O Notation — Think of It Like This

Imagine you have a phone book with **1000 names**:

| Algorithm | What You Do | How Many Steps? | Big O |
|-----------|-------------|-----------------|-------|
| **Linear Search** | Read every name from page 1 | 1000 steps (worst case) | O(n) |
| **Binary Search** | Open middle, go left/right | ~10 steps | O(log n) |
| **Direct Lookup** | You know the page number | 1 step | O(1) |
| **Compare every pair** | Check each name with every other | 1,000,000 steps | O(n²) |

```
Common complexities (fastest to slowest):

O(1)        — Constant      — HashMap get/put, array access by index
O(log n)    — Logarithmic   — Binary search, balanced BST operations
O(n)        — Linear        — Single loop, linear search
O(n log n)  — Linearithmic  — Merge sort, efficient sorts
O(n²)       — Quadratic     — Nested loops, bubble sort
O(2ⁿ)       — Exponential   — Recursive fibonacci (naive), subsets
O(n!)       — Factorial     — Permutations, traveling salesman brute force
```

### How to Calculate Big O — Step by Step

```java
// ═══════════════════════════════════════════════════════
// EXAMPLE 1: O(1) — Constant Time
// No matter how big the array, this ALWAYS takes 1 step
// ═══════════════════════════════════════════════════════
int getFirst(int[] arr) {
    return arr[0];  // Just one operation, always
    // Array has 10 elements? → 1 step
    // Array has 10,000,000 elements? → Still 1 step!
}

// ═══════════════════════════════════════════════════════
// EXAMPLE 2: O(n) — Linear Time
// If array has n elements, we do n iterations
// ═══════════════════════════════════════════════════════
int sum(int[] arr) {
    int total = 0;              // 1 operation
    for (int num : arr) {       // This loop runs n times (n = arr.length)
        total += num;           // 1 operation, but repeated n times
    }
    return total;               // 1 operation
    // Total: 1 + n + 1 = n + 2 → We drop constants → O(n)
}

// ═══════════════════════════════════════════════════════
// EXAMPLE 3: O(n²) — Quadratic Time
// Loop inside loop = multiply iterations
// ═══════════════════════════════════════════════════════
void printPairs(int[] arr) {
    // Outer loop: runs n times
    for (int i = 0; i < arr.length; i++) {
        // Inner loop: for EACH i, runs n times
        for (int j = 0; j < arr.length; j++) {
            System.out.println(arr[i] + ", " + arr[j]);
            // This line executes n × n = n² times total
        }
    }
}
// If arr = [1, 2, 3] (n=3): prints 3×3 = 9 pairs
// If arr has 100 elements: prints 100×100 = 10,000 pairs
// If arr has 1000 elements: prints 1000×1000 = 1,000,000 pairs!

// ═══════════════════════════════════════════════════════
// EXAMPLE 4: O(log n) — Logarithmic Time
// We cut the problem in HALF each time
// ═══════════════════════════════════════════════════════
int binarySearch(int[] arr, int target) {
    int left = 0;                        // Start of search range
    int right = arr.length - 1;          // End of search range
    
    while (left <= right) {              // Keep searching while range is valid
        int mid = left + (right - left) / 2;  // Find the middle index
        
        if (arr[mid] == target) {
            return mid;                  // Found it!
        }
        if (arr[mid] < target) {
            left = mid + 1;             // Target is in RIGHT half → discard left
        } else {
            right = mid - 1;            // Target is in LEFT half → discard right
        }
    }
    return -1;                           // Not found
}
// Array of 1024 elements: at most 10 steps (2^10 = 1024)
// Array of 1,000,000 elements: at most 20 steps (2^20 ≈ 1M)
// That's the POWER of O(log n)!
```

### Space Complexity — How Much Extra Memory?

```java
// O(1) space — uses CONSTANT extra memory (just a few variables)
void swap(int[] arr, int i, int j) {
    int temp = arr[i];  // Only ONE extra variable, no matter the array size
    arr[i] = arr[j];
    arr[j] = temp;
}

// O(n) space — creates memory PROPORTIONAL to input
int[] duplicate(int[] arr) {
    int[] result = new int[arr.length]; // New array same size as input
    // If input has 1000 elements, we use 1000 extra slots = O(n)
    System.arraycopy(arr, 0, result, 0, arr.length);
    return result;
}

// O(n) space — recursive call stack
int factorial(int n) {
    if (n <= 1) return 1;            // Base case: stop here
    return n * factorial(n - 1);     // Each call adds a FRAME to the call stack
}
// factorial(5) creates 5 stack frames:
//   factorial(5) → factorial(4) → factorial(3) → factorial(2) → factorial(1)
//   That's n frames on the stack = O(n) space
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

### Problem 1: Two Sum — Line by Line

```
PROBLEM: Given an array and a target, find two numbers that add up to target.
         Return their INDICES.

Example: nums = [2, 7, 11, 15], target = 9
Answer:  [0, 1] because nums[0] + nums[1] = 2 + 7 = 9
```

```java
public int[] twoSum(int[] nums, int target) {
    
    // IDEA: For each number, we need (target - number) to exist.
    //       We use a HashMap to remember numbers we've already seen.
    //       HashMap gives O(1) lookup, so overall is O(n).
    
    // Step 1: Create a map to store {number → its index}
    Map<Integer, Integer> map = new HashMap<>();
    // map will look like: {2→0, 7→1, 11→2, ...}
    
    // Step 2: Walk through the array
    for (int i = 0; i < nums.length; i++) {
        
        // Step 3: What number do we NEED to reach the target?
        int complement = target - nums[i];
        // When i=0: nums[0]=2, complement = 9-2 = 7 (we need a 7!)
        // When i=1: nums[1]=7, complement = 9-7 = 2 (we need a 2!)
        
        // Step 4: Have we already SEEN this complement?
        if (map.containsKey(complement)) {
            // YES! We found it! Return both indices.
            return new int[]{map.get(complement), i};
            // When i=1: map contains {2→0}, complement=2, found!
            // Return [0, 1] ✅
        }
        
        // Step 5: We haven't found a pair yet. Remember this number.
        map.put(nums[i], i);
        // When i=0: map becomes {2→0}
    }
    
    return new int[]{}; // No solution found
}
```

```
TRACE through: nums = [2, 7, 11, 15], target = 9

i=0: nums[0]=2, complement=9-2=7, map={} → 7 not in map → add 2→0 → map={2:0}
i=1: nums[1]=7, complement=9-7=2, map={2:0} → 2 IS in map! → return [0, 1] ✅

Why is this O(n)? We visit each element at most once. HashMap lookup is O(1).
Why not use two nested loops? That would be O(n²) — much slower!
```

### Problem 2: Maximum Subarray (Kadane's Algorithm) — Line by Line

```
PROBLEM: Find the contiguous subarray with the LARGEST sum.

Example: nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Answer:  6 (the subarray [4, -1, 2, 1] has the largest sum)
```

```java
public int maxSubArray(int[] nums) {
    
    // IDEA (Kadane's Algorithm):
    // Walk through the array. At each position, decide:
    //   "Should I EXTEND the previous subarray, or START FRESH here?"
    // If previous sum is negative, starting fresh is better.
    
    int maxSum = nums[0];      // Best sum found so far (start with first element)
    int currentSum = nums[0];  // Current subarray sum
    
    // Start from index 1 (we already used index 0)
    for (int i = 1; i < nums.length; i++) {
        
        // KEY DECISION: Is it better to...
        //   (A) Add nums[i] to the current subarray (extend)?
        //   (B) Start a new subarray from nums[i] (start fresh)?
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        // If currentSum was negative, nums[i] alone is better → start fresh
        // If currentSum was positive, adding nums[i] extends the winning streak
        
        // Update the global maximum
        maxSum = Math.max(maxSum, currentSum);
    }
    
    return maxSum;
}
```

```
TRACE through: nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

Start: maxSum = -2, currentSum = -2

i=1: nums[1]=1
     currentSum = max(1, -2+1) = max(1, -1) = 1  ← Start fresh! (previous was negative)
     maxSum = max(-2, 1) = 1

i=2: nums[2]=-3
     currentSum = max(-3, 1+(-3)) = max(-3, -2) = -2  ← Extend (less bad)
     maxSum = max(1, -2) = 1

i=3: nums[3]=4
     currentSum = max(4, -2+4) = max(4, 2) = 4  ← Start fresh!
     maxSum = max(1, 4) = 4

i=4: nums[4]=-1
     currentSum = max(-1, 4+(-1)) = max(-1, 3) = 3  ← Extend
     maxSum = max(4, 3) = 4

i=5: nums[5]=2
     currentSum = max(2, 3+2) = max(2, 5) = 5  ← Extend
     maxSum = max(4, 5) = 5

i=6: nums[6]=1
     currentSum = max(1, 5+1) = max(1, 6) = 6  ← Extend
     maxSum = max(5, 6) = 6 ✅

i=7: nums[7]=-5
     currentSum = max(-5, 6+(-5)) = max(-5, 1) = 1  ← Extend
     maxSum = max(6, 1) = 6

i=8: nums[8]=4
     currentSum = max(4, 1+4) = max(4, 5) = 5  ← Extend
     maxSum = max(6, 5) = 6

ANSWER: 6 (subarray [4, -1, 2, 1])
```

### Problem 3: Remove Duplicates from Sorted Array — Line by Line

```
PROBLEM: Remove duplicates IN-PLACE from a sorted array. Return new length.

Example: nums = [1, 1, 2, 2, 3]
After:   nums = [1, 2, 3, _, _], return 3
```

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    
    // IDEA: Two-pointer technique
    // 'slow' points to the last UNIQUE element
    // 'fast' scans ahead looking for new unique elements
    
    int slow = 0;  // Position of last unique element
    
    for (int fast = 1; fast < nums.length; fast++) {
        //  fast scans every element starting from index 1
        
        if (nums[fast] != nums[slow]) {
            // Found a NEW unique element!
            slow++;                    // Move slow pointer forward
            nums[slow] = nums[fast];   // Place new unique element there
        }
        // If nums[fast] == nums[slow], it's a duplicate → skip it
    }
    
    return slow + 1;  // Number of unique elements
}
```

```
TRACE: nums = [1, 1, 2, 2, 3]
       slow=0

fast=1: nums[1]=1, nums[slow]=nums[0]=1 → SAME (duplicate) → skip
        Array: [1, 1, 2, 2, 3], slow=0

fast=2: nums[2]=2, nums[slow]=nums[0]=1 → DIFFERENT! → slow=1, nums[1]=2
        Array: [1, 2, 2, 2, 3], slow=1

fast=3: nums[3]=2, nums[slow]=nums[1]=2 → SAME (duplicate) → skip
        Array: [1, 2, 2, 2, 3], slow=1

fast=4: nums[4]=3, nums[slow]=nums[1]=2 → DIFFERENT! → slow=2, nums[2]=3
        Array: [1, 2, 3, 2, 3], slow=2

Return slow+1 = 3 → First 3 elements [1, 2, 3] are the unique ones ✅
```

### Problem 4: Valid Anagram — Line by Line

```java
// PROBLEM: Are two strings anagrams? (same letters, different order)
// Example: "listen" and "silent" → true
// Example: "hello" and "world" → false

public boolean isAnagram(String s, String t) {
    // If lengths differ, IMPOSSIBLE to be anagrams
    if (s.length() != t.length()) return false;
    
    // IDEA: Count occurrences of each letter.
    // If both strings have the SAME counts, they're anagrams.
    // We use an array of size 26 (one slot per letter a-z)
    
    int[] count = new int[26];
    // count[0] = count of 'a', count[1] = count of 'b', ... count[25] = count of 'z'
    
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;  // INCREMENT for each letter in s
        count[t.charAt(i) - 'a']--;  // DECREMENT for each letter in t
        // If s and t have the same letters, increments and decrements cancel out!
    }
    
    // Check if ALL counts are zero
    for (int c : count) {
        if (c != 0) return false;  // Some letter has different count → not anagram
    }
    return true;
}
```

```
TRACE: s = "listen", t = "silent"

After processing all characters:
count['l'-'a'] = count[11] = 1-1 = 0  (l appears 1 time in both)
count['i'-'a'] = count[8]  = 1-1 = 0  (i appears 1 time in both)
count['s'-'a'] = count[18] = 1-1 = 0  (s appears 1 time in both)
count['t'-'a'] = count[19] = 1-1 = 0  (t appears 1 time in both)
count['e'-'a'] = count[4]  = 1-1 = 0  (e appears 1 time in both)
count['n'-'a'] = count[13] = 1-1 = 0  (n appears 1 time in both)

All zeros → return true ✅
```

---

## 3. Linked Lists

### What is a Linked List? — Visual

```
Array (contiguous memory):
┌───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │  ← Elements stored side by side
└───┴───┴───┴───┴───┘
  0   1   2   3   4    ← Indices (instant access!)

Linked List (scattered memory):
┌───┬───┐     ┌───┬───┐     ┌───┬──────┐
│ 1 │ ──┼────►│ 2 │ ──┼────►│ 3 │ null │
└───┴───┘     └───┴───┘     └───┴──────┘
 data next     data next     data  next
 (head)                      (tail)

Each node has: DATA + POINTER TO NEXT NODE
The last node points to null (end of list)
```

### Implementation — Line by Line

```java
// A single node in the linked list
public class ListNode {
    int val;        // The data this node holds
    ListNode next;  // Pointer/reference to the NEXT node (or null if last)
    
    ListNode(int val) {
        this.val = val;
        this.next = null;  // By default, next is null (no next node yet)
    }
}
```

### Problem 1: Reverse a Linked List — Visual Step by Step

```
PROBLEM: Reverse the direction of all pointers.

Before: 1 → 2 → 3 → 4 → null
After:  null ← 1 ← 2 ← 3 ← 4  (which is: 4 → 3 → 2 → 1 → null)
```

```java
public ListNode reverseList(ListNode head) {
    
    // We need THREE pointers:
    ListNode prev = null;      // The node BEHIND the current node
    ListNode current = head;   // The node we're currently processing
    // (We'll also need 'next' to save the next node before we break the link)
    
    // Walk through the entire list
    while (current != null) {
        
        ListNode next = current.next;  // SAVE the next node (before we lose it!)
        current.next = prev;           // REVERSE the pointer (point backward!)
        prev = current;               // Move prev forward
        current = next;               // Move current forward
    }
    
    return prev;  // prev is now the new head (last node of original list)
}
```

```
VISUAL TRACE: 1 → 2 → 3 → null

═══ INITIAL STATE ═══
prev = null
current = [1] → [2] → [3] → null

═══ ITERATION 1 (current = 1) ═══
next = current.next = [2]         Save next: we'll need it!
current.next = prev = null        1 now points to null (reversed!)
prev = current = [1]              Move prev to 1
current = next = [2]              Move current to 2

State: null ← [1]    [2] → [3] → null
       prev          current

═══ ITERATION 2 (current = 2) ═══
next = current.next = [3]         Save next
current.next = prev = [1]         2 now points to 1 (reversed!)
prev = current = [2]              Move prev to 2
current = next = [3]              Move current to 3

State: null ← [1] ← [2]    [3] → null
                     prev   current

═══ ITERATION 3 (current = 3) ═══
next = current.next = null        Save next (it's null, end of list)
current.next = prev = [2]         3 now points to 2 (reversed!)
prev = current = [3]              Move prev to 3
current = next = null             current is null → STOP the loop

State: null ← [1] ← [2] ← [3]
                            prev   current=null

═══ RESULT ═══
Return prev = [3], which is: 3 → 2 → 1 → null ✅
```

### Problem 2: Detect Cycle (Floyd's Algorithm) — Visual

```
PROBLEM: Does the linked list have a cycle (loop)?

No cycle:    1 → 2 → 3 → 4 → null
Has cycle:   1 → 2 → 3 → 4
                  ↑           │
                  └───────────┘  (4 points back to 2!)
```

```java
public boolean hasCycle(ListNode head) {
    
    // IDEA: Two runners on a circular track.
    //   Slow runner: moves 1 step at a time
    //   Fast runner: moves 2 steps at a time
    //   If there's a cycle, fast will eventually LAP slow (they'll meet)
    //   If there's no cycle, fast will reach the end (null)
    
    ListNode slow = head;  // Tortoise 🐢
    ListNode fast = head;  // Hare 🐇
    
    while (fast != null && fast.next != null) {
        // Why check fast.next? Because fast moves 2 steps,
        // so we need to make sure BOTH steps are valid
        
        slow = slow.next;          // Tortoise moves 1 step 🐢
        fast = fast.next.next;     // Hare moves 2 steps 🐇
        
        if (slow == fast) {
            return true;   // They met! There's a cycle! 🔄
        }
    }
    
    return false;  // Fast reached null → no cycle
}
```

```
VISUAL TRACE — List with cycle: 1 → 2 → 3 → 4 → back to 2

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3   (slow +1, fast +2)
Step 2: slow=3, fast=2   (fast went 4→2, wrapped around!)
Step 3: slow=4, fast=4   (slow=fast → CYCLE DETECTED! ✅)

VISUAL TRACE — List without cycle: 1 → 2 → 3 → 4 → null

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=null  (fast reached end → NO CYCLE ✅)
```

### Problem 3: Find Middle Node — Visual

```java
public ListNode middleNode(ListNode head) {
    
    // IDEA: Same fast/slow pointer trick!
    //   When FAST reaches the end, SLOW is at the MIDDLE.
    //   Because fast moves 2x speed, slow covers half the distance.
    
    ListNode slow = head;  // 🐢 moves 1 step
    ListNode fast = head;  // 🐇 moves 2 steps
    
    while (fast != null && fast.next != null) {
        slow = slow.next;          // 🐢: 1 step
        fast = fast.next.next;     // 🐇: 2 steps
    }
    
    return slow;  // 🐢 is at the middle!
}
```

```
TRACE: 1 → 2 → 3 → 4 → 5 → null

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=5
Step 3: fast.next=null → STOP

Return slow = 3 (middle!) ✅

Why does this work?
  fast travels 2x the speed of slow.
  When fast reaches the END (distance = n), 
  slow has traveled n/2 = the MIDDLE.
```

---

## 4. Stacks & Queues

### Stack — Visual (LIFO: Last In, First Out)

```
Think of a STACK OF PLATES 🍽️

Push 1:  |1|        Push 2:  |2|       Push 3:  |3|
         └─┘                 |1|                |2|
                             └─┘                |1|
                                                └─┘
Pop: removes 3 (the last one added) = LIFO!
```

```java
Deque<Integer> stack = new ArrayDeque<>(); // Use ArrayDeque, NOT Stack class

stack.push(1);   // Stack: [1]        (1 is on top)
stack.push(2);   // Stack: [2, 1]     (2 is on top)
stack.push(3);   // Stack: [3, 2, 1]  (3 is on top)

stack.peek();    // → 3 (look at top without removing)
stack.pop();     // → 3 (remove from top)  Stack: [2, 1]
stack.pop();     // → 2                    Stack: [1]
stack.isEmpty(); // → false
stack.size();    // → 1
```

### Problem: Valid Parentheses — Line by Line

```
PROBLEM: Given a string with brackets, check if they're properly matched.
"()"     → true
"()[]{}" → true
"(]"     → false
"([)]"   → false
"{[]}"   → true
```

```java
public boolean isValid(String s) {
    
    // IDEA: When we see an OPENING bracket, push it onto the stack.
    //       When we see a CLOSING bracket, the top of stack must be its match.
    //       If not → invalid. If stack is empty at the end → valid.
    
    Deque<Character> stack = new ArrayDeque<>();
    
    // Map each closing bracket to its matching opening bracket
    Map<Character, Character> map = Map.of(
        ')', '(',    // ) matches (
        '}', '{',    // } matches {
        ']', '['     // ] matches [
    );
    
    for (char c : s.toCharArray()) {
        
        if (c == '(' || c == '{' || c == '[') {
            // It's an OPENING bracket → push onto stack
            stack.push(c);
        } else {
            // It's a CLOSING bracket → check if it matches the top
            if (stack.isEmpty()) {
                return false;  // No opening bracket to match!
            }
            char top = stack.pop();  // Get the most recent opening bracket
            if (top != map.get(c)) {
                return false;  // Mismatch! e.g., ( doesn't match ]
            }
        }
    }
    
    return stack.isEmpty();  // Stack should be empty if all brackets matched
}
```

```
TRACE: s = "{[()]}"

c='{' → opening → push → stack: [{]
c='[' → opening → push → stack: [[, {]
c='(' → opening → push → stack: [(, [, {]
c=')' → closing → pop '(' → matches ')' ✅ → stack: [[, {]
c=']' → closing → pop '[' → matches ']' ✅ → stack: [{]
c='}' → closing → pop '{' → matches '}' ✅ → stack: []

Stack is empty → return true ✅

TRACE: s = "([)]"

c='(' → push → stack: [(]
c='[' → push → stack: [[, (]
c=')' → closing → pop '[' → '[' should match ')' → NO! '[' ≠ '(' → return false ❌
```

### Queue — Visual (FIFO: First In, First Out)

```
Think of a LINE at a store 🏪

Enqueue 1:  →|1|→        Enqueue 2:  →|2|1|→      Enqueue 3:  →|3|2|1|→
              (front)                   (front=1)                  (front=1)

Dequeue: removes 1 (the FIRST one added) = FIFO!
```

```java
Queue<Integer> queue = new LinkedList<>();

queue.offer(1);  // Queue: [1]        (1 is at front)
queue.offer(2);  // Queue: [1, 2]     (1 is still at front)
queue.offer(3);  // Queue: [1, 2, 3]  (1 is still at front)

queue.peek();    // → 1 (look at front without removing)
queue.poll();    // → 1 (remove from front)  Queue: [2, 3]
queue.poll();    // → 2                      Queue: [3]
```

---

## 5. Hash Maps & Hash Sets

### How HashMap Works — Visual

```
HashMap uses an ARRAY OF BUCKETS.

Step 1: key.hashCode() → some integer
Step 2: integer % arraySize → bucket index
Step 3: Store (key, value) in that bucket

Example: map.put("Dilip", 25)

"Dilip".hashCode() → 66847384
66847384 % 16 → 8  (bucket index)

Buckets:
Index: [0] [1] [2] [3] [4] [5] [6] [7] [8]          [9] ...
       null                               ↓
                                    ("Dilip", 25)
                                          ↓
                                    ("Alice", 30)  ← COLLISION! Same bucket!
                                          ↓
                                         null

Collision handling: Linked list in each bucket (becomes tree at 8+ entries)
```

### Problem: First Non-Repeating Character — Line by Line

```java
// PROBLEM: Find the first character that appears only ONCE.
// "leetcode" → 'l' (first non-repeating)
// "aabb" → -1 (no non-repeating)

public int firstUniqChar(String s) {
    
    // Step 1: Count how many times each character appears
    Map<Character, Integer> count = new HashMap<>();
    
    for (char c : s.toCharArray()) {
        // merge: if key exists, add 1. If not, set to 1.
        count.merge(c, 1, Integer::sum);
    }
    // For "leetcode": {l:1, e:3, t:1, c:1, o:1, d:1}
    //                  ^--- l appears once!
    
    // Step 2: Find the FIRST character with count = 1
    for (int i = 0; i < s.length(); i++) {
        if (count.get(s.charAt(i)) == 1) {
            return i;  // Return the INDEX of the first unique char
        }
    }
    
    return -1;  // No unique character found
}
```

```
TRACE: s = "leetcode"

Step 1 — Count:
  l: 1
  e: 3 (appears at index 1, 2, 8 — wait, let me recount)
  
  Actually: "l-e-e-t-c-o-d-e"
  l:1, e:3, t:1, c:1, o:1, d:1

Step 2 — First with count=1:
  i=0: s[0]='l', count['l']=1 → FOUND! Return 0 ✅
```

---

## 6. Trees — Visual Deep Dive

### What is a Binary Tree?

```
A tree where each node has AT MOST 2 children (left and right).

        1          ← ROOT (the topmost node)
       / \
      2   3        ← CHILDREN of 1
     / \   \
    4   5   6      ← LEAVES (no children)

Terminology:
- Root: node 1 (top of tree)
- Parent of 4: node 2
- Children of 2: nodes 4 and 5
- Leaves: nodes 4, 5, 6 (no children)
- Depth of node 4: 2 (edges from root to node)
- Height of tree: 2 (edges from root to deepest leaf)
```

### Tree Traversals — The 4 Ways to Visit Every Node

```
        1
       / \
      2   3
     / \
    4   5

Traversal results:
  Inorder   (Left → Root → Right): 4, 2, 5, 1, 3
  Preorder  (Root → Left → Right): 1, 2, 4, 5, 3
  Postorder (Left → Right → Root): 4, 5, 2, 3, 1
  Level Order (BFS):               1, 2, 3, 4, 5

MEMORY AID:
  In-order:   "Root is IN the middle"      L-Root-R
  Pre-order:  "Root comes FIRST (PRE)"     Root-L-R
  Post-order: "Root comes LAST (POST)"     L-R-Root
```

### Inorder Traversal — Visual Recursion

```java
//         1
//        / \
//       2   3
//      / \
//     4   5

public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    
    if (root == null) return result;  // Base case: empty tree
    
    // Step 1: Visit LEFT subtree first (go as deep left as possible)
    result.addAll(inorderTraversal(root.left));
    
    // Step 2: Visit ROOT (current node)
    result.add(root.val);
    
    // Step 3: Visit RIGHT subtree
    result.addAll(inorderTraversal(root.right));
    
    return result;
}
```

```
VISUAL RECURSION CALL STACK:

inorder(1)
├── inorder(1.left = 2)
│   ├── inorder(2.left = 4)
│   │   ├── inorder(4.left = null) → return []     ← BASE CASE
│   │   ├── add 4                                    ← VISIT NODE
│   │   └── inorder(4.right = null) → return []     ← BASE CASE
│   │   → returns [4]
│   ├── add 2                                        ← VISIT NODE
│   └── inorder(2.right = 5)
│       ├── inorder(5.left = null) → return []
│       ├── add 5
│       └── inorder(5.right = null) → return []
│       → returns [5]
│   → returns [4, 2, 5]
├── add 1                                            ← VISIT NODE
└── inorder(1.right = 3)
    ├── inorder(3.left = null) → return []
    ├── add 3
    └── inorder(3.right = null) → return []
    → returns [3]
→ returns [4, 2, 5, 1, 3] ✅
```

### Level Order Traversal (BFS) — Visual

```java
//         1
//        / \
//       2   3
//      / \   \
//     4   5   6

public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    // BFS uses a QUEUE (FIFO)
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);  // Start with the root
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();  // How many nodes at THIS level?
        List<Integer> level = new ArrayList<>();
        
        // Process ALL nodes at current level
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();    // Take node from front of queue
            level.add(node.val);             // Record its value
            
            // Add its children to queue (they're the NEXT level)
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        
        result.add(level);  // Finished this level
    }
    
    return result;
}
```

```
VISUAL TRACE:

Queue: [1]                    Level 1: processing...
  poll 1 → add to level      Level 1: [1]
  add children 2, 3 to queue
  Queue: [2, 3]

Queue: [2, 3]                Level 2: processing (2 nodes)...
  poll 2 → add to level      Level 2: [2]
  add children 4, 5
  poll 3 → add to level      Level 2: [2, 3]
  add child 6
  Queue: [4, 5, 6]

Queue: [4, 5, 6]             Level 3: processing (3 nodes)...
  poll 4 → add to level      Level 3: [4]
  poll 5 → add to level      Level 3: [4, 5]
  poll 6 → add to level      Level 3: [4, 5, 6]
  Queue: []

Queue empty → DONE!
Result: [[1], [2, 3], [4, 5, 6]] ✅
```

### Problem: Maximum Depth of Binary Tree — Visual

```java
//         3           depth = 0 (root level)
//        / \
//       9  20          depth = 1
//         /  \
//        15   7        depth = 2
//
// Answer: 3 (there are 3 levels)

public int maxDepth(TreeNode root) {
    // BASE CASE: If the tree is empty, depth is 0
    if (root == null) return 0;
    
    // RECURSIVE CASE:
    // The depth of a tree = 1 + the maximum depth of its subtrees
    // "How deep is this tree? = 1 (for me) + how deep is my deepest child"
    
    int leftDepth = maxDepth(root.left);    // How deep is the left side?
    int rightDepth = maxDepth(root.right);  // How deep is the right side?
    
    return 1 + Math.max(leftDepth, rightDepth);  // 1 (for me) + the deeper side
}
```

```
VISUAL RECURSION:

maxDepth(3)
├── leftDepth = maxDepth(9)
│   ├── maxDepth(null) → 0       (9 has no left child)
│   └── maxDepth(null) → 0       (9 has no right child)
│   → 1 + max(0, 0) = 1
│
└── rightDepth = maxDepth(20)
    ├── maxDepth(15)
    │   ├── maxDepth(null) → 0
    │   └── maxDepth(null) → 0
    │   → 1 + max(0, 0) = 1
    │
    └── maxDepth(7)
        ├── maxDepth(null) → 0
        └── maxDepth(null) → 0
        → 1 + max(0, 0) = 1
    
    → 1 + max(1, 1) = 2

→ 1 + max(1, 2) = 3 ✅

HOW TO THINK ABOUT TREE PROBLEMS:
1. What's the BASE CASE? → Usually: if root == null, return something
2. What do I need from my LEFT child?
3. What do I need from my RIGHT child?
4. How do I COMBINE them with my current node?
```

### Problem: Invert Binary Tree — Visual

```java
// BEFORE:          AFTER:
//      4              4
//    /   \          /   \
//   2     7        7     2
//  / \   / \      / \   / \
// 1   3 6   9    9   6 3   1
//
// Just SWAP left and right children at EVERY node!

public TreeNode invertTree(TreeNode root) {
    // BASE CASE: empty tree or leaf
    if (root == null) return null;
    
    // SWAP the left and right children
    TreeNode temp = root.left;      // Save left
    root.left = root.right;         // Left becomes right
    root.right = temp;              // Right becomes left
    
    // Recursively invert the subtrees
    invertTree(root.left);          // Invert what's now on the left
    invertTree(root.right);         // Invert what's now on the right
    
    return root;
}
```

```
VISUAL STEP BY STEP:

Step 1: At node 4 → swap children
      4                    4
    /   \       →        /   \
   2     7              7     2
  / \   / \            / \   / \
 1   3 6   9          6   9 1   3

Step 2: At node 7 (now left child) → swap children
      4                    4
    /   \       →        /   \
   7     2              7     2
  / \   / \            / \   / \
 6   9 1   3          9   6 1   3

Step 3: At node 2 (now right child) → swap children
      4                    4
    /   \       →        /   \
   7     2              7     2
  / \   / \            / \   / \
 9   6 1   3          9   6 3   1

DONE! ✅
```

---

## 7. Binary Search Trees (BST)

### BST Property

```
For EVERY node in the tree:
  - ALL values in LEFT subtree < node's value
  - ALL values in RIGHT subtree > node's value

Valid BST:          Invalid BST:
      8                  8
     / \                / \
    3   10              3  10
   / \    \            / \   \
  1   6   14          1   9  14    ← 9 > 8 but it's in LEFT subtree!
```

### BST Search — Visual (O(log n) on balanced tree)

```java
public TreeNode search(TreeNode root, int val) {
    // BASE CASE: not found or found
    if (root == null || root.val == val) return root;
    
    // If target is SMALLER, go LEFT (smaller values are on the left)
    if (val < root.val) return search(root.left, val);
    
    // If target is BIGGER, go RIGHT
    return search(root.right, val);
}
```

```
Search for 6 in BST:
      8          → 6 < 8, go LEFT
     / \
    3   10       → 6 > 3, go RIGHT
   / \
  1   6          → 6 == 6, FOUND! ✅

Only visited 3 nodes out of 6 = O(log n) 🚀
(Compare to O(n) if we had to check every node)
```

### Validate BST — Why Simple Comparison Fails

```java
// ❌ WRONG: Only checking parent-child relationship
// This MISSES cases like: 8 → 3 → 9 (9 > 3 but 9 > 8, invalid!)
boolean isValidBSTWrong(TreeNode root) {
    if (root == null) return true;
    if (root.left != null && root.left.val >= root.val) return false;
    if (root.right != null && root.right.val <= root.val) return false;
    return isValidBSTWrong(root.left) && isValidBSTWrong(root.right);
}

// ✅ CORRECT: Pass down min/max boundaries
public boolean isValidBST(TreeNode root) {
    return isValid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean isValid(TreeNode node, long min, long max) {
    if (node == null) return true;  // Empty tree is valid
    
    // This node's value must be WITHIN (min, max) range
    if (node.val <= min || node.val >= max) return false;
    
    // Left child must be in range (min, node.val)
    // Right child must be in range (node.val, max)
    return isValid(node.left, min, node.val)
        && isValid(node.right, node.val, max);
}
```

```
TRACE on valid BST:
      8
     / \
    3   10

isValid(8, -∞, +∞)  → 8 is in range (-∞, +∞) ✅
├── isValid(3, -∞, 8)  → 3 is in range (-∞, 8) ✅
└── isValid(10, 8, +∞) → 10 is in range (8, +∞) ✅

TRACE on INVALID BST:
      8
     / \
    3   10
   / \
  1   9     ← 9 is in LEFT subtree of 8 but 9 > 8!

isValid(8, -∞, +∞)  ✅
├── isValid(3, -∞, 8)  ✅
│   ├── isValid(1, -∞, 3)  ✅
│   └── isValid(9, 3, 8)   → 9 is NOT in range (3, 8) ❌ INVALID!
```

---

## 8. Heaps & Priority Queues

### Min-Heap Visual

```
A Min-Heap: parent is ALWAYS smaller than children

        1            PriorityQueue (Min-Heap)
       / \           peek() → 1 (smallest, always at root)
      3   2
     / \
    7   4

After poll() (remove smallest = 1):
        2            Heap re-organizes!
       / \           New smallest is at root
      3   4
     /
    7
```

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

minHeap.offer(7);   // Heap: [7]
minHeap.offer(3);   // Heap: [3, 7]       (3 < 7, bubbles up)
minHeap.offer(1);   // Heap: [1, 7, 3]    (1 < 3, bubbles up to root)
minHeap.offer(4);   // Heap: [1, 4, 3, 7]

minHeap.peek();     // → 1 (smallest, always at top)
minHeap.poll();     // → 1, Heap becomes [3, 4, 7]
minHeap.poll();     // → 3, Heap becomes [4, 7]
```

### Problem: Kth Largest Element — Line by Line

```java
// PROBLEM: Find the k-th LARGEST element in array.
// nums = [3,2,1,5,6,4], k = 2 → Answer: 5 (sorted: [1,2,3,4,5,6], 2nd largest = 5)

public int findKthLargest(int[] nums, int k) {
    
    // IDEA: Use a MIN-HEAP of size k.
    // Keep only the k LARGEST elements in the heap.
    // The ROOT of the heap = the SMALLEST among the k largest = kth largest!
    
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();  // Min-heap
    
    for (int num : nums) {
        minHeap.offer(num);             // Add element to heap
        
        if (minHeap.size() > k) {
            minHeap.poll();             // Remove the smallest!
            // This ensures we keep only the k LARGEST values
        }
    }
    
    return minHeap.peek();  // Root = smallest of the k largest = kth largest
}
```

```
TRACE: nums = [3, 2, 1, 5, 6, 4], k = 2

num=3: heap=[3]          size=1 ≤ 2, keep
num=2: heap=[2,3]        size=2 ≤ 2, keep
num=1: heap=[1,3,2]      size=3 > 2, poll smallest(1) → heap=[2,3]
num=5: heap=[2,5,3]      size=3 > 2, poll smallest(2) → heap=[3,5]
num=6: heap=[3,5,6]      size=3 > 2, poll smallest(3) → heap=[5,6]
num=4: heap=[4,6,5]      size=3 > 2, poll smallest(4) → heap=[5,6]

Result: heap.peek() = 5 ✅ (5 is the 2nd largest!)

Heap always contains the 2 largest values: [5, 6]
The ROOT (smallest of these) = 5 = 2nd largest. Clever!
```

---

## 9. Graphs

### Graph Representations — Visual

```
Graph with 5 nodes and edges:

    0 --- 1
    |     |
    |     |
    3 --- 2
          |
          4

Adjacency List representation:
  0: [1, 3]
  1: [0, 2]
  2: [1, 3, 4]
  3: [0, 2]
  4: [2]
```

### BFS (Breadth-First Search) — Visual

```java
// BFS explores LEVEL BY LEVEL (like tree level order traversal)
// Uses a QUEUE

public List<Integer> bfs(Map<Integer, List<Integer>> graph, int start) {
    List<Integer> result = new ArrayList<>();
    Set<Integer> visited = new HashSet<>();  // Track visited nodes (avoid cycles!)
    Queue<Integer> queue = new LinkedList<>();
    
    visited.add(start);   // Mark start as visited
    queue.offer(start);   // Add start to queue
    
    while (!queue.isEmpty()) {
        int node = queue.poll();         // Take from front of queue
        result.add(node);               // Process this node
        
        // Add all UNVISITED neighbors to the queue
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);   // Mark as visited
                queue.offer(neighbor);   // Add to queue for later processing
            }
        }
    }
    return result;
}
```

```
VISUAL TRACE: Start from node 0

    0 --- 1
    |     |
    3 --- 2
          |
          4

Step 0: Queue=[0], Visited={0}, Result=[]
Step 1: Process 0 → neighbors: 1, 3
        Queue=[1, 3], Visited={0, 1, 3}, Result=[0]
Step 2: Process 1 → neighbors: 0(visited), 2
        Queue=[3, 2], Visited={0, 1, 3, 2}, Result=[0, 1]
Step 3: Process 3 → neighbors: 0(visited), 2(visited)
        Queue=[2], Visited={0, 1, 3, 2}, Result=[0, 1, 3]
Step 4: Process 2 → neighbors: 1(visited), 3(visited), 4
        Queue=[4], Visited={0, 1, 2, 3, 4}, Result=[0, 1, 3, 2]
Step 5: Process 4 → neighbors: 2(visited)
        Queue=[], Result=[0, 1, 3, 2, 4]

BFS Order: 0 → 1 → 3 → 2 → 4 (level by level!) ✅
```

### DFS (Depth-First Search) — Visual

```java
// DFS goes as DEEP as possible before backtracking
// Uses RECURSION (or explicit Stack)

public void dfs(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
    visited.add(node);             // Mark as visited
    System.out.print(node + " "); // Process this node
    
    // Visit ALL unvisited neighbors (go DEEP first)
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        if (!visited.contains(neighbor)) {
            dfs(graph, neighbor, visited);  // Recursive call — go deeper!
        }
    }
    // When all neighbors are visited, BACKTRACK (return from recursion)
}
```

```
VISUAL TRACE: Start from node 0

    0 --- 1
    |     |
    3 --- 2
          |
          4

dfs(0)           Visit 0
├── dfs(1)       Visit 1 (first unvisited neighbor of 0)
│   ├── dfs(2)   Visit 2 (first unvisited neighbor of 1, 0 is visited)
│   │   ├── dfs(3)  Visit 3 (1 is visited, try 3)
│   │   │   └── all neighbors visited → BACKTRACK
│   │   └── dfs(4)  Visit 4
│   │       └── all neighbors visited → BACKTRACK
│   └── BACKTRACK
└── BACKTRACK

DFS Order: 0 → 1 → 2 → 3 → 4 (goes deep first!) ✅
```

### Problem: Number of Islands — Visual

```
PROBLEM: Grid of '1' (land) and '0' (water). Count connected land masses.

Grid:
  1 1 0 0 0
  1 1 0 0 0
  0 0 1 0 0
  0 0 0 1 1

Answer: 3 islands (marked as A, B, C below)
  A A 0 0 0
  A A 0 0 0
  0 0 B 0 0
  0 0 0 C C
```

```java
public int numIslands(char[][] grid) {
    int count = 0;
    
    // Scan every cell in the grid
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            
            if (grid[i][j] == '1') {
                count++;                // Found a NEW island!
                dfsGrid(grid, i, j);    // "Sink" the entire island (mark as visited)
            }
        }
    }
    return count;
}

private void dfsGrid(char[][] grid, int i, int j) {
    // BOUNDARY CHECK: out of bounds or water?
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == '0') {
        return;  // Stop: either out of bounds or water
    }
    
    grid[i][j] = '0';         // SINK this land cell (mark as visited by changing to '0')
    
    // Explore all 4 directions (up, down, left, right)
    dfsGrid(grid, i + 1, j);  // DOWN
    dfsGrid(grid, i - 1, j);  // UP
    dfsGrid(grid, i, j + 1);  // RIGHT
    dfsGrid(grid, i, j - 1);  // LEFT
}
```

---

## 10. Sorting Algorithms

### Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ |

### Merge Sort — Visual Step by Step

```
IDEA: Divide the array in half, sort each half, then MERGE them together.

Array: [38, 27, 43, 3, 9, 82, 10]

Step 1: DIVIDE (split into halves until single elements)

         [38, 27, 43, 3, 9, 82, 10]
              /                \
     [38, 27, 43]         [3, 9, 82, 10]
       /      \              /        \
   [38, 27]  [43]       [3, 9]    [82, 10]
    /   \                /   \      /    \
  [38] [27]            [3]  [9]  [82]  [10]
  
Step 2: MERGE (combine sorted halves)

  [38] [27]  →  [27, 38]        (compare: 27 < 38)
  [3]  [9]   →  [3, 9]          (compare: 3 < 9)
  [82] [10]  →  [10, 82]        (compare: 10 < 82)
  
  [27, 38] [43]    →  [27, 38, 43]
  [3, 9] [10, 82]  →  [3, 9, 10, 82]
  
  [27, 38, 43] [3, 9, 10, 82]  →  [3, 9, 10, 27, 38, 43, 82] ✅
```

```java
public void mergeSort(int[] arr, int left, int right) {
    if (left < right) {                              // At least 2 elements
        int mid = left + (right - left) / 2;         // Find middle
        
        mergeSort(arr, left, mid);                   // Sort LEFT half
        mergeSort(arr, mid + 1, right);              // Sort RIGHT half
        merge(arr, left, mid, right);                // MERGE the sorted halves
    }
}

private void merge(int[] arr, int left, int mid, int right) {
    // Create temporary arrays for left and right halves
    int[] leftArr = Arrays.copyOfRange(arr, left, mid + 1);
    int[] rightArr = Arrays.copyOfRange(arr, mid + 1, right + 1);
    
    int i = 0, j = 0, k = left;  // Pointers for leftArr, rightArr, and result
    
    // Compare elements from both halves, pick the SMALLER one
    while (i < leftArr.length && j < rightArr.length) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k++] = leftArr[i++];   // Left element is smaller → take it
        } else {
            arr[k++] = rightArr[j++];  // Right element is smaller → take it
        }
    }
    
    // Copy remaining elements (one array may have leftovers)
    while (i < leftArr.length) arr[k++] = leftArr[i++];
    while (j < rightArr.length) arr[k++] = rightArr[j++];
}
```

---

## 11. Searching Algorithms

### Binary Search — Visual Step by Step

```
PROBLEM: Find target=7 in sorted array [1, 3, 5, 7, 9, 11, 13]

Step 1: left=0, right=6, mid=3
        [1, 3, 5, 7, 9, 11, 13]
         L        M           R
        arr[3]=7 == target=7 → FOUND at index 3! ✅

PROBLEM: Find target=11 in [1, 3, 5, 7, 9, 11, 13]

Step 1: left=0, right=6, mid=3
        [1, 3, 5, 7, 9, 11, 13]
         L        M           R
        arr[3]=7 < 11 → target is in RIGHT half → left = mid+1 = 4

Step 2: left=4, right=6, mid=5
        [1, 3, 5, 7, 9, 11, 13]
                      L   M   R
        arr[5]=11 == target=11 → FOUND at index 5! ✅

Each step cuts the search space in HALF → O(log n)
```

---

## 12. Recursion — Visual Deep Dive

### How Recursion Works — The Call Stack

```
Think of recursion like RUSSIAN NESTING DOLLS 🪆
Each function call is a "doll" placed inside the previous one.
When the smallest doll is reached (BASE CASE), we start unwinding.
```

### Example: Factorial — Visual Call Stack

```java
int factorial(int n) {
    if (n <= 1) return 1;              // BASE CASE: stop here!
    return n * factorial(n - 1);       // RECURSIVE CASE: call myself with smaller input
}
```

```
factorial(5) = 5 * factorial(4)
                     4 * factorial(3)
                           3 * factorial(2)
                                 2 * factorial(1)
                                       return 1    ← BASE CASE HIT!

Now UNWIND (substitute values back up):
                                 2 * 1 = 2
                           3 * 2 = 6
                     4 * 6 = 24
               5 * 24 = 120 ✅

CALL STACK VISUALIZATION:
┌──────────────────┐
│ factorial(1) = 1 │ ← Returns first (base case)
├──────────────────┤
│ factorial(2) = 2×1 = 2 │
├──────────────────┤
│ factorial(3) = 3×2 = 6 │
├──────────────────┤
│ factorial(4) = 4×6 = 24 │
├──────────────────┤
│ factorial(5) = 5×24 = 120 │ ← Returns last (final answer)
└──────────────────┘
```

### Example: Fibonacci — Why Naive Recursion is SLOW

```java
int fib(int n) {
    if (n <= 1) return n;              // Base cases: fib(0)=0, fib(1)=1
    return fib(n - 1) + fib(n - 2);   // Two recursive calls!
}
```

```
fib(5) call tree:
                         fib(5)
                       /        \
                   fib(4)       fib(3)
                  /     \       /     \
              fib(3)  fib(2)  fib(2)  fib(1)
              /   \    / \     / \      |
          fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)   1
          / \     |      |      |      |      |
      fib(1) fib(0)  1      1      0      1      0
        |      |
        1      0

Notice: fib(3) is computed TWICE! fib(2) is computed THREE times!
This is O(2ⁿ) — EXPONENTIAL! fib(50) takes FOREVER.

SOLUTION: Dynamic Programming (store results, never recompute)
```

### Backtracking — Visual (Subsets Problem)

```java
// PROBLEM: Generate all subsets of [1, 2, 3]
// Answer: [], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]

public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(result, new ArrayList<>(), nums, 0);
    return result;
}

private void backtrack(List<List<Integer>> result, List<Integer> current, int[] nums, int start) {
    // Every state is a valid subset → add it
    result.add(new ArrayList<>(current));
    
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);                           // CHOOSE: include nums[i]
        backtrack(result, current, nums, i + 1);        // EXPLORE: try adding more
        current.remove(current.size() - 1);             // UN-CHOOSE: backtrack!
    }
}
```

```
VISUAL — Decision Tree for subsets([1, 2, 3]):

                          []
                 /         |          \
              [1]         [2]        [3]
            /    \         |
         [1,2]  [1,3]   [2,3]
           |
        [1,2,3]

Each node is a valid subset. We explore by adding elements,
then BACKTRACK by removing the last element and trying the next.

Detailed trace:
backtrack([], start=0)
  ADD [] to result
  i=0: add 1 → current=[1]
    backtrack([1], start=1)
      ADD [1] to result
      i=1: add 2 → current=[1,2]
        backtrack([1,2], start=2)
          ADD [1,2] to result
          i=2: add 3 → current=[1,2,3]
            backtrack([1,2,3], start=3)
              ADD [1,2,3] to result
              loop ends (start=3 >= length)
            remove 3 → current=[1,2]  ← BACKTRACK!
        remove 2 → current=[1]        ← BACKTRACK!
      i=2: add 3 → current=[1,3]
        backtrack([1,3], start=3)
          ADD [1,3] to result
        remove 3 → current=[1]        ← BACKTRACK!
    remove 1 → current=[]             ← BACKTRACK!
  i=1: add 2 → current=[2]
    ...continues similarly...

Result: [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]] ✅
```

---

## 13. Dynamic Programming — Visual Deep Dive

### What is DP? — Think of It Like This

```
DP = Recursion + Memoization (remembering past answers)

WITHOUT DP (Naive Recursion):
  "Hey, what's fib(5)? I need fib(4) and fib(3)..."
  "Hey, what's fib(4)? I need fib(3) and fib(2)..."
  "Hey, what's fib(3)? I need fib(2) and fib(1)..."
  "Hey, what's fib(3) AGAIN? I need fib(2) and fib(1)..."  ← DUPLICATE WORK!

WITH DP (Memoization):
  "Hey, what's fib(5)? I need fib(4) and fib(3)..."
  "Hey, what's fib(4)? I need fib(3) and fib(2)..."
  "Hey, what's fib(3)? I need fib(2) and fib(1)..."
  "Hey, what's fib(3) AGAIN? I ALREADY KNOW IT'S 2!"  ← INSTANT! ⚡
```

### DP Approach — 4 Steps

```
1. DEFINE THE STATE
   → What do I store? What does dp[i] represent?

2. FIND THE RECURRENCE RELATION (formula)
   → How does dp[i] relate to previous values?

3. DEFINE BASE CASES
   → What are the starting values?

4. DECIDE DIRECTION
   → Top-down (recursion + memo) or Bottom-up (loop + table)?
```

### Problem 1: Fibonacci — DP Table

```java
// dp[i] = the i-th Fibonacci number
// Formula: dp[i] = dp[i-1] + dp[i-2]
// Base cases: dp[0] = 0, dp[1] = 1

public int fib(int n) {
    if (n <= 1) return n;
    
    int[] dp = new int[n + 1];   // Table to store results
    dp[0] = 0;                   // Base case 1
    dp[1] = 1;                   // Base case 2
    
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];  // Fill table using formula
    }
    
    return dp[n];
}
```

```
DP TABLE for fib(7):

┌───────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬──────┐
│ Index │  0  │  1  │  2  │  3  │  4  │  5  │  6  │  7   │
├───────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼──────┤
│ dp[i] │  0  │  1  │  1  │  2  │  3  │  5  │  8  │  13  │
├───────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼──────┤
│Formula│base │base │0+1  │1+1  │1+2  │2+3  │3+5  │5+8   │
└───────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴──────┘

How dp[5] was calculated:
  dp[5] = dp[4] + dp[3] = 3 + 2 = 5 ✅
```

### Space Optimized (We only need the LAST TWO values!)

```java
public int fibOptimized(int n) {
    if (n <= 1) return n;
    
    int prev2 = 0;  // dp[i-2], starts as dp[0]
    int prev1 = 1;  // dp[i-1], starts as dp[1]
    
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;  // dp[i] = dp[i-1] + dp[i-2]
        prev2 = prev1;             // Shift: old dp[i-1] becomes new dp[i-2]
        prev1 = curr;              // Shift: dp[i] becomes new dp[i-1]
    }
    
    return prev1;  // This is dp[n]
}
// Space: O(1) instead of O(n)! We don't need the whole table.
```

### Problem 2: Climbing Stairs — DP Table

```
PROBLEM: You're climbing a staircase with n steps.
         Each time you can climb 1 or 2 steps.
         How many DISTINCT WAYS can you reach the top?

Example: n=4
  Ways: [1+1+1+1], [1+1+2], [1+2+1], [2+1+1], [2+2] = 5 ways

THINKING PROCESS:
  To reach step i, I could have come from:
    - Step (i-1) by taking 1 step, OR
    - Step (i-2) by taking 2 steps
  So: dp[i] = dp[i-1] + dp[i-2]  ← Same as Fibonacci!

  Base cases:
    dp[1] = 1 (only one way: take 1 step)
    dp[2] = 2 (two ways: 1+1 or 2)
```

```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    
    int[] dp = new int[n + 1];
    dp[1] = 1;  // 1 way to climb 1 step
    dp[2] = 2;  // 2 ways to climb 2 steps (1+1 or 2)
    
    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
        // "Ways to reach step i = ways to reach (i-1) + ways to reach (i-2)"
    }
    
    return dp[n];
}
```

```
DP TABLE for climbStairs(5):

┌──────────┬──────┬──────┬──────┬──────┬──────┐
│  Steps   │  1   │  2   │  3   │  4   │  5   │
├──────────┼──────┼──────┼──────┼──────┼──────┤
│ dp[i]    │  1   │  2   │  3   │  5   │  8   │
├──────────┼──────┼──────┼──────┼──────┼──────┤
│ Formula  │ base │ base │ 1+2  │ 2+3  │ 3+5  │
├──────────┼──────┼──────┼──────┼──────┼──────┤
│ The ways │  1   │ 1+1  │1+1+1 │1111  │11111 │
│          │      │  2   │ 1+2  │112   │1112  │
│          │      │      │ 2+1  │121   │1121  │
│          │      │      │      │211   │1211  │
│          │      │      │      │ 22   │2111  │
│          │      │      │      │      │122   │
│          │      │      │      │      │212   │
│          │      │      │      │      │221   │
└──────────┴──────┴──────┴──────┴──────┴──────┘

dp[5] = dp[4] + dp[3] = 5 + 3 = 8 ways ✅
```

### Problem 3: Coin Change — DP Table (2D Thinking)

```
PROBLEM: Given coins [1, 3, 4] and amount = 6
         What's the MINIMUM number of coins to make 6?

Answer: 2 coins (3 + 3 = 6)

STATE:   dp[i] = minimum coins to make amount i
FORMULA: dp[i] = min(dp[i - coin] + 1) for each coin
BASE:    dp[0] = 0 (zero coins needed for amount 0)
```

```java
public int coinChange(int[] coins, int amount) {
    // dp[i] = minimum number of coins to make amount i
    int[] dp = new int[amount + 1];
    
    // Initialize with a large value (impossible to make)
    Arrays.fill(dp, amount + 1);  // Use amount+1 as "infinity"
    
    dp[0] = 0;  // Base case: 0 coins needed for amount 0
    
    for (int i = 1; i <= amount; i++) {
        // Try every coin
        for (int coin : coins) {
            if (coin <= i) {
                // If I use this coin, I need dp[i - coin] more coins + this one
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount];
}
```

```
DP TABLE: coins = [1, 3, 4], amount = 6

Building dp[] step by step:

Amount:  0    1    2    3    4    5    6
dp[]:   [0]  [∞]  [∞]  [∞]  [∞]  [∞]  [∞]

─── i=1: Try each coin ───
  coin=1: 1 ≤ 1 → dp[1] = min(∞, dp[1-1]+1) = min(∞, dp[0]+1) = min(∞, 1) = 1
  coin=3: 3 > 1 → skip
  coin=4: 4 > 1 → skip
  dp[1] = 1

Amount:  0    1    2    3    4    5    6
dp[]:   [0]  [1]  [∞]  [∞]  [∞]  [∞]  [∞]

─── i=2: Try each coin ───
  coin=1: dp[2] = min(∞, dp[1]+1) = min(∞, 2) = 2
  dp[2] = 2

Amount:  0    1    2    3    4    5    6
dp[]:   [0]  [1]  [2]  [∞]  [∞]  [∞]  [∞]

─── i=3: Try each coin ───
  coin=1: dp[3] = min(∞, dp[2]+1) = min(∞, 3) = 3
  coin=3: dp[3] = min(3, dp[0]+1) = min(3, 1) = 1  ← Better! Use coin 3 directly!
  dp[3] = 1

Amount:  0    1    2    3    4    5    6
dp[]:   [0]  [1]  [2]  [1]  [∞]  [∞]  [∞]

─── i=4: Try each coin ───
  coin=1: dp[4] = min(∞, dp[3]+1) = min(∞, 2) = 2
  coin=3: dp[4] = min(2, dp[1]+1) = min(2, 2) = 2
  coin=4: dp[4] = min(2, dp[0]+1) = min(2, 1) = 1  ← Better! Use coin 4 directly!
  dp[4] = 1

Amount:  0    1    2    3    4    5    6
dp[]:   [0]  [1]  [2]  [1]  [1]  [∞]  [∞]

─── i=5: Try each coin ───
  coin=1: dp[5] = min(∞, dp[4]+1) = min(∞, 2) = 2
  coin=3: dp[5] = min(2, dp[2]+1) = min(2, 3) = 2
  coin=4: dp[5] = min(2, dp[1]+1) = min(2, 2) = 2
  dp[5] = 2

─── i=6: Try each coin ───
  coin=1: dp[6] = min(∞, dp[5]+1) = min(∞, 3) = 3
  coin=3: dp[6] = min(3, dp[3]+1) = min(3, 2) = 2  ← Use two 3's!
  coin=4: dp[6] = min(2, dp[2]+1) = min(2, 3) = 2
  dp[6] = 2

FINAL TABLE:
┌────────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ Amount │  0  │  1  │  2  │  3  │  4  │  5  │  6  │
├────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│ dp[i]  │  0  │  1  │  2  │  1  │  1  │  2  │  2  │
├────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│ Coins  │ --  │ {1} │{1,1}│ {3} │ {4} │{1,4}│{3,3}│
│ Used   │     │     │     │     │     │     │     │
└────────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Answer: dp[6] = 2 (use two coins of value 3: 3+3=6) ✅
```

### Problem 4: Longest Common Subsequence — 2D DP Table

```
PROBLEM: Find the longest common subsequence of two strings.
         (Subsequence = not necessarily contiguous)

text1 = "abcde"
text2 = "ace"

Common subsequences: "a", "c", "e", "ac", "ae", "ce", "ace"
Longest: "ace" (length 3)

STATE:   dp[i][j] = LCS length of text1[0..i-1] and text2[0..j-1]
FORMULA: If text1[i-1] == text2[j-1]: dp[i][j] = dp[i-1][j-1] + 1
         Else: dp[i][j] = max(dp[i-1][j], dp[i][j-1])
BASE:    dp[0][j] = 0, dp[i][0] = 0 (empty string has LCS 0)
```

```java
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    // dp[0][anything] = 0, dp[anything][0] = 0 (default in Java)
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                // Characters MATCH! LCS extends by 1
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                // Characters DON'T match. Take the best we had without one of them.
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[m][n];
}
```

```
2D DP TABLE: text1 = "abcde", text2 = "ace"

         ""   a    c    e
    ┌────┬────┬────┬────┐
 "" │  0 │  0 │  0 │  0 │  ← Base cases (empty string)
    ├────┼────┼────┼────┤
  a │  0 │  1 │  1 │  1 │  ← 'a' matches 'a'! dp[1][1]=dp[0][0]+1=1
    ├────┼────┼────┼────┤
  b │  0 │  1 │  1 │  1 │  ← 'b' doesn't match anything, carry best=1
    ├────┼────┼────┼────┤
  c │  0 │  1 │  2 │  2 │  ← 'c' matches 'c'! dp[3][2]=dp[2][1]+1=2
    ├────┼────┼────┼────┤
  d │  0 │  1 │  2 │  2 │  ← 'd' doesn't match, carry best=2
    ├────┼────┼────┼────┤
  e │  0 │  1 │  2 │  3 │  ← 'e' matches 'e'! dp[5][3]=dp[4][2]+1=3 ✅
    └────┴────┴────┴────┘

Answer: dp[5][3] = 3 (LCS = "ace") ✅

HOW TO READ THE TABLE:
  dp[3][2] = 2 means: LCS of "abc" and "ac" has length 2 ("ac")
  dp[5][3] = 3 means: LCS of "abcde" and "ace" has length 3 ("ace")
```

### Problem 5: 0/1 Knapsack — 2D DP Table

```
PROBLEM: You have a bag that holds weight W=7.
         You have items with weights and values:
           Item 1: weight=1, value=1
           Item 2: weight=3, value=4
           Item 3: weight=4, value=5
           Item 4: weight=5, value=7
         Maximize the total value! (Can't use item more than once)

STATE:   dp[i][w] = max value using items 1..i with bag capacity w
FORMULA: dp[i][w] = max(
           dp[i-1][w],                          (DON'T take item i)
           dp[i-1][w-weight[i]] + value[i]       (TAKE item i, if it fits)
         )
```

```java
public int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[][] dp = new int[n + 1][W + 1];
    
    for (int i = 1; i <= n; i++) {
        for (int w = 1; w <= W; w++) {
            
            // Option 1: DON'T take item i (value same as without item i)
            dp[i][w] = dp[i - 1][w];
            
            // Option 2: TAKE item i (if it fits in remaining capacity)
            if (weights[i - 1] <= w) {
                int valueIfTaken = dp[i - 1][w - weights[i - 1]] + values[i - 1];
                dp[i][w] = Math.max(dp[i][w], valueIfTaken);
            }
        }
    }
    return dp[n][W];
}
```

```
2D DP TABLE: W=7, items: (w=1,v=1), (w=3,v=4), (w=4,v=5), (w=5,v=7)

Capacity→   0    1    2    3    4    5    6    7
         ┌────┬────┬────┬────┬────┬────┬────┬────┐
No items │  0 │  0 │  0 │  0 │  0 │  0 │  0 │  0 │
         ├────┼────┼────┼────┼────┼────┼────┼────┤
Item 1   │  0 │  1 │  1 │  1 │  1 │  1 │  1 │  1 │ w=1,v=1
(w=1,v=1)│    │take│    │    │    │    │    │    │
         ├────┼────┼────┼────┼────┼────┼────┼────┤
Item 2   │  0 │  1 │  1 │  4 │  5 │  5 │  5 │  5 │ w=3,v=4
(w=3,v=4)│    │    │    │take│1+4 │    │    │    │
         ├────┼────┼────┼────┼────┼────┼────┼────┤
Item 3   │  0 │  1 │  1 │  4 │  5 │  6 │  6 │  9 │ w=4,v=5
(w=4,v=5)│    │    │    │    │take│1+5 │    │4+5 │
         ├────┼────┼────┼────┼────┼────┼────┼────┤
Item 4   │  0 │  1 │  1 │  4 │  5 │  7 │  8 │  9 │ w=5,v=7
(w=5,v=7)│    │    │    │    │    │take│1+7 │    │
         └────┴────┴────┴────┴────┴────┴────┴────┘

Answer: dp[4][7] = 9

How? Take Item 2 (w=3, v=4) + Item 3 (w=4, v=5) = weight 7, value 9 ✅

READING dp[4][7] = 9:
  "Using items 1-4, with bag capacity 7, max value = 9"
  
READING dp[3][7] = 9:
  "Using items 1-3, with bag capacity 7, max value = 9"
  (Same! Item 4 wasn't needed for the optimal solution)
```

---

## 14. Greedy Algorithms

### Greedy vs DP

```
GREEDY: Make the LOCALLY BEST choice at each step.
        Faster but doesn't always give optimal answer.

DP:     Consider ALL possibilities.
        Slower but ALWAYS gives optimal answer.

When to use Greedy? When the locally optimal choice leads to globally optimal.
  Example: Coin change with coins [1, 5, 10, 25] → Greedy works!
  Example: Coin change with coins [1, 3, 4] → Greedy FAILS! (for amount 6)
    Greedy picks: 4+1+1 = 3 coins
    Optimal:      3+3   = 2 coins ← DP finds this!
```

### Problem: Best Time to Buy and Sell Stock — Line by Line

```java
// PROBLEM: Given stock prices for each day, find max profit (buy then sell)
// prices = [7, 1, 5, 3, 6, 4]
// Answer: 5 (buy at 1, sell at 6)

public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE;  // Track the LOWEST price seen so far
    int maxProfit = 0;                 // Track the BEST profit seen so far
    
    for (int price : prices) {
        // Is today's price the new LOWEST?
        minPrice = Math.min(minPrice, price);
        
        // If I sell today, what's my profit? (today's price - lowest price before today)
        int profit = price - minPrice;
        
        // Is this the BEST profit we've seen?
        maxProfit = Math.max(maxProfit, profit);
    }
    
    return maxProfit;
}
```

```
TRACE: prices = [7, 1, 5, 3, 6, 4]

Day 0: price=7, minPrice=min(MAX, 7)=7, profit=7-7=0, maxProfit=0
Day 1: price=1, minPrice=min(7, 1)=1, profit=1-1=0, maxProfit=0     ← New low!
Day 2: price=5, minPrice=min(1, 5)=1, profit=5-1=4, maxProfit=4     ← Profit!
Day 3: price=3, minPrice=min(1, 3)=1, profit=3-1=2, maxProfit=4
Day 4: price=6, minPrice=min(1, 6)=1, profit=6-1=5, maxProfit=5 ✅  ← Best!
Day 5: price=4, minPrice=min(1, 4)=1, profit=4-1=3, maxProfit=5

Answer: 5 (buy at day 1 for $1, sell at day 4 for $6) ✅
```

---

## 15. Two Pointers & Sliding Window

### Two Pointers — Visual

```
TWO POINTERS: Use two indices that move through the data structure.

Type 1: Opposite directions (converge toward each other)
  [1, 2, 3, 4, 5, 6, 7]
   L→                ←R

Type 2: Same direction (fast and slow)
  [1, 2, 3, 4, 5, 6, 7]
   S→ F→→
```

### Sliding Window — Visual

```
SLIDING WINDOW: Maintain a "window" that slides through the array.

Find max sum of 3 consecutive elements:
arr = [2, 1, 5, 1, 3, 2]

Window size k=3:

Step 1: [2, 1, 5] 1, 3, 2  → sum = 8
Step 2:  2 [1, 5, 1] 3, 2  → sum = 7
Step 3:  2, 1 [5, 1, 3] 2  → sum = 9 ← MAX!
Step 4:  2, 1, 5 [1, 3, 2] → sum = 6

Instead of recalculating sum each time:
  New sum = old sum - element going out + element coming in
  Step 2: 8 - 2 + 1 = 7
  Step 3: 7 - 1 + 3 = 9
  Step 4: 9 - 5 + 2 = 6

This makes it O(n) instead of O(n×k)!
```

### Problem: Longest Substring Without Repeating Characters — Line by Line

```java
// PROBLEM: Find length of longest substring without repeating characters
// "abcabcbb" → 3 ("abc")
// "bbbbb" → 1 ("b")
// "pwwkew" → 3 ("wke")

public int lengthOfLongestSubstring(String s) {
    // Map: character → its most recent index
    Map<Character, Integer> map = new HashMap<>();
    
    int maxLen = 0;    // Best answer found so far
    int left = 0;      // Left boundary of our window
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);  // Current character (expanding window right)
        
        if (map.containsKey(c)) {
            // We've seen this character before!
            // Move left boundary PAST the previous occurrence
            left = Math.max(left, map.get(c) + 1);
            // Why Math.max? Because left should never move BACKWARD
        }
        
        map.put(c, right);  // Record/update this character's latest position
        
        // Window is [left, right], all unique characters
        maxLen = Math.max(maxLen, right - left + 1);
    }
    
    return maxLen;
}
```

```
TRACE: s = "abcabcbb"

right=0: c='a', not in map → map={a:0}, window=[0,0]="a", maxLen=1
right=1: c='b', not in map → map={a:0,b:1}, window=[0,1]="ab", maxLen=2
right=2: c='c', not in map → map={a:0,b:1,c:2}, window=[0,2]="abc", maxLen=3
right=3: c='a', IN map at 0! → left=max(0, 0+1)=1
         map={a:3,b:1,c:2}, window=[1,3]="bca", maxLen=3
right=4: c='b', IN map at 1! → left=max(1, 1+1)=2
         map={a:3,b:4,c:2}, window=[2,4]="cab", maxLen=3
right=5: c='c', IN map at 2! → left=max(2, 2+1)=3
         map={a:3,b:4,c:5}, window=[3,5]="abc", maxLen=3
right=6: c='b', IN map at 4! → left=max(3, 4+1)=5
         map={a:3,b:6,c:5}, window=[5,6]="cb", maxLen=3
right=7: c='b', IN map at 6! → left=max(5, 6+1)=7
         map={a:3,b:7,c:5}, window=[7,7]="b", maxLen=3

Answer: 3 ("abc") ✅
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
| Dynamic Programming | Overlapping subproblems | Knapsack, LCS, coin change |
| Backtracking | Constraint satisfaction | N-Queens, Sudoku solver |
| Greedy | Local optimal = global optimal | Activity selection, Huffman |
| Heap/Priority Queue | Top K, scheduling | K largest, merge K lists |
| Stack | Matching, nesting, monotonic | Valid parentheses, next greater |

### Problem-Solving Steps

```
1. UNDERSTAND the problem (read 2-3 times, identify inputs/outputs)
2. EXAMPLES — walk through examples, edge cases
3. BRUTE FORCE — think of the simplest solution first
4. OPTIMIZE — can you use a pattern? Better data structure?
5. CODE — write clean, readable code
6. TEST — trace through examples, test edge cases
```

### How to Think About Tree Problems

```
TEMPLATE for most tree problems:

int solve(TreeNode root) {
    // 1. BASE CASE: What to return for empty tree?
    if (root == null) return BASE_VALUE;
    
    // 2. ASK LEFT: What do I need from my left subtree?
    int leftResult = solve(root.left);
    
    // 3. ASK RIGHT: What do I need from my right subtree?
    int rightResult = solve(root.right);
    
    // 4. COMBINE: How do I use leftResult, rightResult, and root.val?
    return COMBINE(leftResult, rightResult, root.val);
}

Examples:
- maxDepth:   return 1 + max(leftDepth, rightDepth)
- sum:        return leftSum + rightSum + root.val
- isBalanced: return abs(leftHeight - rightHeight) <= 1
```

### How to Think About DP Problems

```
TEMPLATE for most DP problems:

1. Can I DEFINE dp[i]? What does it represent?
   "dp[i] is the answer to the subproblem for input of size i"

2. Can I find a FORMULA relating dp[i] to smaller subproblems?
   dp[i] = some function of dp[i-1], dp[i-2], etc.

3. What are the BASE CASES?
   dp[0] = ?, dp[1] = ?

4. Which DIRECTION do I fill the table?
   Usually left to right, top to bottom

5. Where is the ANSWER?
   Usually dp[n] or dp[m][n]
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
**A:** O(log n) — each step halves the search space. Requires sorted input.

**Q6: What is a Hash Map?**
**A:** A data structure mapping keys to values using a hash function. Average O(1) for get/put/remove. Handles collisions with chaining (linked list → tree at 8+ entries in Java).

**Q7: What is recursion?**
**A:** A function calling itself with smaller input until a base case is reached. Every recursion needs: 1) Base case (when to stop), 2) Recursive case (how to make the problem smaller).

**Q8: What is the difference between BFS and DFS?**
**A:** BFS explores level by level (Queue, finds shortest path). DFS goes as deep as possible first (Stack/recursion, explores all paths).

---

### Intermediate Level

**Q9: What is Dynamic Programming?**
**A:** Solving complex problems by breaking them into overlapping subproblems and storing results to avoid recomputation. Two approaches: top-down (recursion + memoization) and bottom-up (tabulation with loops).

**Q10: Explain merge sort vs quick sort.**
**A:** Merge sort: always O(n log n), stable, O(n) space. Quick sort: O(n log n) average, O(n²) worst, not stable, O(log n) space. Quick sort is faster in practice due to cache locality.

**Q11: What is a Binary Search Tree?**
**A:** A binary tree where left subtree values < node < right subtree values. O(log n) operations when balanced. Inorder traversal gives sorted order.

**Q12: What is a heap?**
**A:** A complete binary tree where parent is always greater (max-heap) or smaller (min-heap) than children. Used for priority queues and top-K problems.

**Q13: What is backtracking?**
**A:** Building solutions incrementally and abandoning (backtracking) when a candidate can't lead to a valid solution. Used for permutations, combinations, N-Queens, sudoku.

**Q14: What is the sliding window technique?**
**A:** Maintain a window [left, right] that slides through data. Expand right to include, shrink left to exclude. Reduces O(n²) brute force to O(n).

**Q15: What is the two-pointer technique?**
**A:** Using two indices moving through a data structure. Types: opposite directions (converge) or same direction (fast/slow). Used for pair finding, cycle detection, partitioning.

---

### Advanced Level

**Q16: Explain Dijkstra's algorithm.**
**A:** Finds shortest path from source to all vertices in a weighted graph with non-negative edges. Uses min-heap to greedily process nearest unvisited vertex. Time: O((V+E) log V).

**Q17: What is a Trie?**
**A:** Tree for string prefix operations. Each node represents a character. O(L) search/insert where L = word length. Used for autocomplete, spell check.

**Q18: What is topological sorting?**
**A:** Linear ordering of vertices in a DAG where for every edge (u,v), u comes before v. Used in build systems and dependency resolution.

**Q19: What is Union-Find?**
**A:** Tracks elements in disjoint sets. find(x) returns set identifier, union(x,y) merges sets. With path compression + union by rank: nearly O(1). Used for Kruskal's MST and cycle detection.

**Q20: Compare top-down vs bottom-up DP.**
**A:** Top-down: recursion + cache (memoization), natural to write, may have stack overflow. Bottom-up: iterative + table (tabulation), faster (no recursion overhead), harder to think about.

---

### Rapid-Fire (Q21–Q50)

**Q21: What is O(1) space?** Constant extra memory regardless of input size.

**Q22: How does HashMap handle collisions?** Chaining (linked list per bucket, tree at 8+ entries in Java 8+).

**Q23: What is a balanced BST?** Height is O(log n). Examples: AVL tree, Red-Black tree.

**Q24: What is stable sorting?** Preserves relative order of equal elements. Merge sort: stable. Quick sort: not stable.

**Q25: What is an in-place algorithm?** Uses O(1) extra space. Examples: quick sort, selection sort.

**Q26: What is memoization?** Caching results of expensive function calls to reuse when same inputs occur.

**Q27: What is Floyd's cycle detection?** Slow pointer (1 step) and fast pointer (2 steps). If they meet, cycle exists. O(n) time, O(1) space.

**Q28: What is a monotonic stack?** Stack maintaining monotonically increasing/decreasing order. Used for "next greater element" problems.

**Q29: What is amortized analysis?** Average cost over a sequence. ArrayList.add() is O(1) amortized despite occasional O(n) resizing.

**Q30: What is tail recursion?** Recursive call is the last operation. Can be optimized to avoid stack overflow (Java doesn't optimize this).

**Q31: Array vs LinkedList?** Array: fast random access O(1). LinkedList: fast insert/delete at ends O(1).

**Q32: What is a circular buffer?** Fixed-size buffer wrapping around with head/tail pointers. Used in producer-consumer.

**Q33: HashMap time complexity?** O(1) average, O(n) worst case for all operations.

**Q34: What is counting sort?** Non-comparison sort counting value occurrences. O(n+k) where k = range.

**Q35: Tree vs Graph?** Tree is connected, acyclic graph with n-1 edges. Graphs can have cycles.

**Q36: What is a complete binary tree?** All levels full except possibly last (filled left to right). Used for heaps.

**Q37: What is Kadane's algorithm?** Finds max subarray sum in O(n). At each position: extend previous sum or start fresh.

**Q38: Greedy vs DP?** Greedy: locally optimal (faster, not always correct). DP: considers all possibilities (slower, always optimal).

**Q39: What is a segment tree?** Tree for range queries (sum/min/max) with O(log n) query and update.

**Q40: What is radix sort?** Non-comparison sort by digits. O(d × (n+k)).

**Q41: What is the Master Theorem?** Solves recurrences T(n) = aT(n/b) + O(n^d) for divide-and-conquer.

**Q42: BFS vs Dijkstra?** BFS: unweighted shortest path O(V+E). Dijkstra: weighted O((V+E)log V).

**Q43: What is a spanning tree?** Subgraph with all vertices, minimum edges (n-1), no cycles.

**Q44: What is Kruskal's algorithm?** MST by sorting edges, greedily adding non-cycle-forming edges (Union-Find).

**Q45: What is an AVL tree?** Self-balancing BST with height difference ≤ 1 between subtrees.

**Q46: What is a deque?** Double-Ended Queue: O(1) insert/delete at both ends.

**Q47: What is consistent hashing?** Distributes data on a ring. Adding/removing nodes affects only K/n keys.

**Q48: What is bit manipulation?** Operating on individual bits: AND, OR, XOR, shifts. Power of 2: `n & (n-1) == 0`.

**Q49: How does Arrays.sort() work?** Dual-pivot quicksort for primitives. TimSort (merge+insertion) for objects.

**Q50: What makes a good hash function?** Uniform distribution, fast computation, deterministic, minimizes collisions.

---

## 📚 References & Further Reading

- [LeetCode](https://leetcode.com/) — Practice platform
- [NeetCode Roadmap](https://neetcode.io/roadmap) — Structured problem list
- [Blind 75 / NeetCode 150](https://neetcode.io/) — Must-do problems
- [GeeksforGeeks DSA](https://www.geeksforgeeks.org/data-structures/)
- [Visualgo](https://visualgo.net/) — Algorithm visualization
- [Introduction to Algorithms (CLRS)](https://mitpress.mit.edu/books/introduction-algorithms)

---

> **Previous Topic:** [← 05 - Git](../05-git/README.md)  
> **Next Topic:** [07 - JDBC →](../07-jdbc/README.md)
