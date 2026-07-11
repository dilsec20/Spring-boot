# 📚 Monotonic Stack & Queue — FAANG Masterclass

> **"When you need the NEXT greater, PREVIOUS smaller, or a SLIDING WINDOW maximum, the monotonic stack/queue is your secret weapon."**

---

## 📑 Table of Contents
1. [What is a Monotonic Stack?](#1-what-is-a-monotonic-stack)
2. [Next Greater Element (LC 496, 503)](#2-next-greater-element)
3. [Daily Temperatures (LC 739)](#3-daily-temperatures)
4. [Largest Rectangle in Histogram (LC 84)](#4-largest-rectangle-in-histogram)
5. [Trapping Rain Water (LC 42)](#5-trapping-rain-water)
6. [Maximal Rectangle (LC 85)](#6-maximal-rectangle)
7. [Sum of Subarray Minimums (LC 907)](#7-sum-of-subarray-minimums)
8. [Monotonic Queue: Sliding Window Maximum (LC 239)](#8-sliding-window-maximum)
9. [Shortest Subarray with Sum ≥ K (LC 862)](#9-shortest-subarray-with-sum-k)
10. [Online Stock Span (LC 901)](#10-online-stock-span)
11. [Pattern Recognition Cheat Sheet](#11-pattern-recognition)
12. [Top 12 Must-Do Questions](#12-top-12-must-do)

---

## 1. What is a Monotonic Stack?

A **Monotonic Stack** is a stack that maintains elements in strictly increasing or decreasing order. When a new element violates the order, we **pop** elements until the order is restored.

```
Monotonic DECREASING stack:    [5, 3, 2, 1]  ← top is smallest
  New element 4 arrives:
  Pop 1 (1 < 4), Pop 2 (2 < 4), Pop 3 (3 < 4)
  Push 4: [5, 4]              ← still decreasing ✅

Monotonic INCREASING stack:    [1, 3, 5, 7]  ← top is largest
  New element 4 arrives:
  Pop 7 (7 > 4), Pop 5 (5 > 4)
  Push 4: [1, 3, 4]           ← still increasing ✅
```

### Why is it O(N)?
Each element is **pushed once** and **popped at most once** → Total operations = 2N → **O(N)**.

### When to Use?
- "**Next greater/smaller** element"
- "**Previous greater/smaller** element"
- "**Largest rectangle** in histogram"
- "**Trapping rain water**"
- Any problem where you need to find the nearest boundary on left/right

---

## 2. Next Greater Element

**Problem (LC 496):** For each element, find the first element to its RIGHT that is greater.

`nums = [2, 1, 2, 4, 3]` → `[4, 2, 4, -1, -1]`

### 🧠 How to Think
Traverse RIGHT to LEFT. Maintain a **decreasing** stack. For each element, pop all smaller elements from stack (they can't be the answer for anyone). The top of stack is the next greater.

### 🧠 Dry Run

```
nums = [2, 1, 2, 4, 3]

Process right to left, stack stores VALUES:

i=4, nums[4]=3: stack=[] → no greater → result[4]=-1, push 3. Stack: [3]
i=3, nums[3]=4: stack=[3] → pop 3 (3<4) → stack=[] → no greater → result[3]=-1, push 4. Stack: [4]
i=2, nums[2]=2: stack=[4] → top 4>2 → result[2]=4, push 2. Stack: [4, 2]
i=1, nums[1]=1: stack=[4,2] → top 2>1 → result[1]=2, push 1. Stack: [4, 2, 1]
i=0, nums[0]=2: stack=[4,2,1] → pop 1 (1<2), top 2≤2 → pop 2, top 4>2 → result[0]=4, push 2. Stack: [4, 2]

Result: [4, 2, 4, -1, -1] ✅
```

```java
public int[] nextGreaterElement(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Stack<Integer> stack = new Stack<>(); // Stores values
    
    for (int i = n - 1; i >= 0; i--) {
        // Pop all elements smaller than or equal to current
        while (!stack.isEmpty() && stack.peek() <= nums[i]) {
            stack.pop();
        }
        result[i] = stack.isEmpty() ? -1 : stack.peek();
        stack.push(nums[i]);
    }
    return result;
}
```

### Next Greater Element II (LC 503) — Circular Array
Trick: process the array TWICE (imagine it doubled). Use `i % n`.

```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Stack<Integer> stack = new Stack<>(); // Stores INDICES
    
    for (int i = 2 * n - 1; i >= 0; i--) {
        while (!stack.isEmpty() && nums[stack.peek()] <= nums[i % n]) {
            stack.pop();
        }
        if (!stack.isEmpty()) result[i % n] = nums[stack.peek()];
        stack.push(i % n);
    }
    return result;
}
```

---

## 3. Daily Temperatures (LC 739)

**Problem:** Given daily temperatures, for each day find how many days until a warmer day.

`temps = [73,74,75,71,69,72,76,73]` → `[1,1,4,2,1,1,0,0]`

### 🧠 How to Think
Store INDICES on the stack (not values). When you find a warmer day, pop and calculate the gap.

```java
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] result = new int[n];
    Stack<Integer> stack = new Stack<>(); // Stores INDICES
    
    for (int i = 0; i < n; i++) {
        // Pop all days that are colder than today
        while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
            int prevDay = stack.pop();
            result[prevDay] = i - prevDay; // Days to wait
        }
        stack.push(i);
    }
    return result; // Remaining in stack = no warmer day = 0
}
```

### 🧠 Dry Run
```
temps = [73, 74, 75, 71, 69, 72, 76, 73]

i=0 (73): stack=[] → push 0. Stack: [0]
i=1 (74): 74>73 → pop 0, result[0]=1-0=1. Push 1. Stack: [1]
i=2 (75): 75>74 → pop 1, result[1]=2-1=1. Push 2. Stack: [2]
i=3 (71): 71<75 → push 3. Stack: [2, 3]
i=4 (69): 69<71 → push 4. Stack: [2, 3, 4]
i=5 (72): 72>69 → pop 4, result[4]=5-4=1. 72>71 → pop 3, result[3]=5-3=2. 72<75 → push 5. Stack: [2, 5]
i=6 (76): 76>72 → pop 5, result[5]=6-5=1. 76>75 → pop 2, result[2]=6-2=4. Push 6. Stack: [6]
i=7 (73): 73<76 → push 7. Stack: [6, 7]

Result: [1, 1, 4, 2, 1, 1, 0, 0] ✅
```

---

## 4. Largest Rectangle in Histogram (LC 84)

**Problem:** Given heights of bars in a histogram, find the area of the largest rectangle.

`heights = [2,1,5,6,2,3]` → **10** (the 5,6 bars form a 5×2 rectangle)

### 🧠 How to Think
For each bar, find how far it can extend LEFT and RIGHT (until a shorter bar). Width × height = area.
Use a **monotonically increasing** stack of indices.

When we encounter a bar shorter than the stack top, the stack top bar can no longer extend right → pop and calculate its area.

```java
public int largestRectangleArea(int[] heights) {
    int n = heights.length;
    Stack<Integer> stack = new Stack<>();
    int maxArea = 0;
    
    for (int i = 0; i <= n; i++) {
        int currHeight = (i == n) ? 0 : heights[i]; // Append 0 to flush stack
        
        while (!stack.isEmpty() && currHeight < heights[stack.peek()]) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```

### 🧠 Dry Run
```
heights = [2, 1, 5, 6, 2, 3]

i=0 (h=2): stack=[]. Push 0. Stack: [0]
i=1 (h=1): 1 < 2 → pop 0, height=2, width=1 (stack empty, so width=i=1), area=2. Push 1. Stack: [1]
i=2 (h=5): 5>1 → push 2. Stack: [1, 2]
i=3 (h=6): 6>5 → push 3. Stack: [1, 2, 3]
i=4 (h=2): 2 < 6 → pop 3, height=6, width=4-2-1=1, area=6
           2 < 5 → pop 2, height=5, width=4-1-1=2, area=10 ← MAX!
           2 > 1 → push 4. Stack: [1, 4]
i=5 (h=3): 3>2 → push 5. Stack: [1, 4, 5]
i=6 (h=0, sentinel): 
           0<3 → pop 5, height=3, width=6-4-1=1, area=3
           0<2 → pop 4, height=2, width=6-1-1=4, area=8
           0<1 → pop 1, height=1, width=6 (stack empty), area=6

maxArea = 10 ✅ (bars at index 2,3 with height 5, width 2)
```

---

## 5. Trapping Rain Water (LC 42)

**Problem:** Given elevation map, compute how much water it can trap.

`height = [0,1,0,2,1,0,1,3,2,1,2,1]` → **6**

### Approach 1: Two Pointers (O(1) Space — Preferred)

```java
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int leftMax = 0, rightMax = 0;
    int water = 0;
    
    while (left < right) {
        if (height[left] < height[right]) {
            leftMax = Math.max(leftMax, height[left]);
            water += leftMax - height[left]; // Water above current bar
            left++;
        } else {
            rightMax = Math.max(rightMax, height[right]);
            water += rightMax - height[right];
            right--;
        }
    }
    return water;
}
```

### Approach 2: Monotonic Stack

```java
public int trap(int[] height) {
    Stack<Integer> stack = new Stack<>();
    int water = 0;
    
    for (int i = 0; i < height.length; i++) {
        while (!stack.isEmpty() && height[i] > height[stack.peek()]) {
            int bottom = stack.pop();
            if (stack.isEmpty()) break;
            
            int width = i - stack.peek() - 1;
            int boundedHeight = Math.min(height[i], height[stack.peek()]) - height[bottom];
            water += width * boundedHeight;
        }
        stack.push(i);
    }
    return water;
}
```

### 🧠 Two Pointers Dry Run
```
height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]

Step | left | right | leftMax | rightMax | water | Action
-----|------|-------|---------|----------|-------|-------
  1  | 0    | 11    | 0       | 1        | 0     | h[L]=0 < h[R]=1, leftMax=0, water+=0
  2  | 1    | 11    | 1       | 1        | 0     | h[L]=1 ≥ h[R]=1, rightMax=1, water+=0
  3  | 1    | 10    | 1       | 2        | 0     | h[L]=1 < h[R]=2, leftMax=1, water+=0
  4  | 2    | 10    | 1       | 2        | 1     | h[L]=0, leftMax=1, water+=1-0=1
  5  | 3    | 10    | 2       | 2        | 1     | h[L]=2 ≥ h[R]=2, rightMax=2, water+=0
  6  | 3    | 9     | 2       | 2        | 2     | rightMax=2, water+=2-1=1
  7  | 3    | 8     | 2       | 2        | 2     | rightMax=2, water+=2-2=0
  ... (continues)

Total water = 6 ✅
```

---

## 6. Maximal Rectangle (LC 85)

**Problem:** Given a binary matrix of 0s and 1s, find the largest rectangle containing only 1s.

### 🧠 How to Think
Build a histogram for each row (height of consecutive 1s above). Then apply **Largest Rectangle in Histogram** for each row!

```java
public int maximalRectangle(char[][] matrix) {
    if (matrix.length == 0) return 0;
    int n = matrix[0].length;
    int[] heights = new int[n];
    int maxArea = 0;
    
    for (char[] row : matrix) {
        // Build histogram heights
        for (int j = 0; j < n; j++) {
            heights[j] = (row[j] == '1') ? heights[j] + 1 : 0;
        }
        // Apply largest rectangle in histogram
        maxArea = Math.max(maxArea, largestRectangleArea(heights));
    }
    return maxArea;
}
```

### 🧠 Dry Run
```
Matrix:
  1 0 1 0 0
  1 0 1 1 1
  1 1 1 1 1
  1 0 0 1 0

Row 0 heights: [1, 0, 1, 0, 0] → max rectangle = 1
Row 1 heights: [2, 0, 2, 1, 1] → max rectangle = 3
Row 2 heights: [3, 1, 3, 2, 2] → max rectangle = 6 ← ANSWER!
Row 3 heights: [4, 0, 0, 3, 0] → max rectangle = 4

Answer: 6 ✅
```

---

## 7. Sum of Subarray Minimums (LC 907)

**Problem:** Find the sum of `min(subarray)` for all subarrays. Return modulo 10⁹+7.

### 🧠 How to Think
For each element, find how many subarrays have IT as the minimum. Use monotonic stack to find **Previous Less Element (PLE)** and **Next Less Element (NLE)**.

Count of subarrays where `arr[i]` is min = `left[i] * right[i]`, where `left[i]` = distance to PLE, `right[i]` = distance to NLE.

```java
public int sumSubarrayMins(int[] arr) {
    int n = arr.length;
    long MOD = 1_000_000_007;
    int[] left = new int[n];   // Distance to Previous Less Element
    int[] right = new int[n];  // Distance to Next Less Element
    
    Stack<Integer> stack = new Stack<>();
    
    // Find PLE distances
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && arr[stack.peek()] >= arr[i]) stack.pop();
        left[i] = stack.isEmpty() ? i + 1 : i - stack.peek();
        stack.push(i);
    }
    
    stack.clear();
    
    // Find NLE distances
    for (int i = n - 1; i >= 0; i--) {
        while (!stack.isEmpty() && arr[stack.peek()] > arr[i]) stack.pop();
        right[i] = stack.isEmpty() ? n - i : stack.peek() - i;
        stack.push(i);
    }
    
    // Calculate sum
    long sum = 0;
    for (int i = 0; i < n; i++) {
        sum = (sum + (long) arr[i] * left[i] * right[i]) % MOD;
    }
    return (int) sum;
}
```

---

## 8. Sliding Window Maximum (LC 239) — Monotonic Deque

**Problem:** Given array and window size `k`, find the maximum in each sliding window.

`nums = [1,3,-1,-3,5,3,6,7], k = 3` → `[3,3,5,5,6,7]`

### 🧠 How to Think
Use a **Monotonic Decreasing Deque** (front = maximum). 
- Remove from front if index is out of window.
- Remove from back if value ≤ current (they'll never be the max).
- Front of deque = window maximum.

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>(); // Stores INDICES
    
    for (int i = 0; i < n; i++) {
        // Remove indices outside the window
        while (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
            deque.pollFirst();
        }
        
        // Remove smaller elements from back (they're useless)
        while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
            deque.pollLast();
        }
        
        deque.addLast(i);
        
        // Window is fully formed (i >= k-1)
        if (i >= k - 1) {
            result[i - k + 1] = nums[deque.peekFirst()]; // Front = max
        }
    }
    return result;
}
```

### 🧠 Dry Run
```
nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3

i=0 (1): deque=[0]. Window not full yet.
i=1 (3): 3>1 → remove 0 from back. deque=[1]. Not full.
i=2 (-1): -1<3 → keep. deque=[1, 2]. Window [1,3,-1] → max=nums[1]=3 ✅
i=3 (-3): -3<-1 → keep. deque=[1, 2, 3]. Window [3,-1,-3] → max=nums[1]=3 ✅
i=4 (5): Remove back: -3<5, -1<5, 3<5 → all removed. deque=[4]. 
         Remove front: index 1 < 4-3+1=2 → already gone.
         Window [-1,-3,5] → max=nums[4]=5 ✅
i=5 (3): 3<5 → keep. deque=[4, 5]. Window [-3,5,3] → max=5 ✅
i=6 (6): 3<6 → remove 5. 5<6 → remove 4. deque=[6]. Window [5,3,6] → max=6 ✅
i=7 (7): 6<7 → remove 6. deque=[7]. Window [3,6,7] → max=7 ✅

Result: [3, 3, 5, 5, 6, 7] ✅
```

---

## 9. Shortest Subarray with Sum ≥ K (LC 862)

**Problem:** Find the shortest subarray with sum ≥ K (can have **negative numbers**).

### 🧠 How to Think
Use prefix sums + monotonic deque. For each `prefix[i]`, find the largest `j < i` such that `prefix[i] - prefix[j] >= K` and `i - j` is minimized.

```java
public int shortestSubarray(int[] nums, int K) {
    int n = nums.length;
    long[] prefix = new long[n + 1];
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + nums[i];
    
    Deque<Integer> deque = new ArrayDeque<>();
    int minLen = n + 1;
    
    for (int i = 0; i <= n; i++) {
        // Check if we can satisfy the condition
        while (!deque.isEmpty() && prefix[i] - prefix[deque.peekFirst()] >= K) {
            minLen = Math.min(minLen, i - deque.pollFirst());
        }
        // Maintain increasing deque of prefix sums
        while (!deque.isEmpty() && prefix[deque.peekLast()] >= prefix[i]) {
            deque.pollLast();
        }
        deque.addLast(i);
    }
    
    return minLen <= n ? minLen : -1;
}
```

---

## 10. Online Stock Span (LC 901)

**Problem:** For each day's stock price, find how many consecutive previous days the price was ≤ today.

```java
class StockSpanner {
    Stack<int[]> stack; // {price, span}
    
    public StockSpanner() {
        stack = new Stack<>();
    }
    
    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1]; // Absorb their spans
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
```

### 🧠 Dry Run
```
Prices: [100, 80, 60, 70, 60, 75, 85]

next(100): stack=[], span=1. Push {100,1}. Return 1
next(80):  80<100, span=1. Push {80,1}. Return 1  
next(60):  60<80, span=1. Push {60,1}. Return 1
next(70):  70>60 → pop {60,1}, span=1+1=2. 70<80 → stop. Push {70,2}. Return 2
next(60):  60<70, span=1. Push {60,1}. Return 1
next(75):  75>60 → pop {60,1}, span=2. 75>70 → pop {70,2}, span=4. 75<80 → stop. Push {75,4}. Return 4
next(85):  85>75 → pop {75,4}, span=5. 85>80 → pop {80,1}, span=6. 85<100 → stop. Push {85,6}. Return 6
```

---

## 11. Pattern Recognition

```
┌──────────────────────────────────────────────────────────────────┐
│         MONOTONIC STACK/QUEUE PATTERN RECOGNITION                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  "Next greater element"              → Decreasing stack, R→L    │
│  "Next smaller element"              → Increasing stack, R→L    │
│  "Previous greater element"          → Decreasing stack, L→R    │
│  "Previous smaller element"          → Increasing stack, L→R    │
│  "Days until warmer/higher"          → Indices on stack, L→R    │
│                                                                   │
│  "Largest rectangle in histogram"    → Increasing stack (idx)   │
│  "Trapping rain water"               → Two pointers OR stack   │
│  "Maximal rectangle in matrix"       → Histogram per row       │
│                                                                   │
│  "Sliding window max/min"            → Monotonic DEQUE          │
│  "Stock span"                        → Decreasing stack         │
│  "Sum of subarray min/max"           → PLE + NLE with stack    │
│                                                                   │
│  STACK vs DEQUE:                                                  │
│  Stack: one-directional (next/prev greater/smaller)              │
│  Deque: sliding window (need to remove expired elements)         │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 12. Top 12 Must-Do Questions

1. Next Greater Element I (LC 496) — Basic monotonic stack
2. Next Greater Element II (LC 503) — Circular array
3. Daily Temperatures (LC 739) — Days until warmer
4. Largest Rectangle in Histogram (LC 84) — Classic hard
5. Trapping Rain Water (LC 42) — Two pointers / stack
6. Maximal Rectangle (LC 85) — Histogram per row
7. Sliding Window Maximum (LC 239) — Monotonic deque
8. Sum of Subarray Minimums (LC 907) — PLE + NLE
9. Online Stock Span (LC 901) — Running span
10. Remove K Digits (LC 402) — Greedy + monotonic stack
11. 132 Pattern (LC 456) — Decreasing stack from right
12. Shortest Subarray with Sum ≥ K (LC 862) — Prefix + deque
