# 🔍 Binary Search on Answer — FAANG Masterclass

> **"If the answer lies in a range, and you can check if a value is feasible in O(N), binary search the answer space."**

---

## 📑 Table of Contents
1. [Standard Binary Search Recap](#1-standard-binary-search)
2. [Binary Search on Answer — The Pattern](#2-binary-search-on-answer)
3. [Koko Eating Bananas (LC 875)](#3-koko-eating-bananas)
4. [Capacity to Ship Packages (LC 1011)](#4-capacity-to-ship-packages)
5. [Split Array Largest Sum (LC 410)](#5-split-array-largest-sum)
6. [Aggressive Cows / Magnetic Balls (LC 2517)](#6-aggressive-cows)
7. [Minimize Max Distance to Gas Station (LC 774)](#7-minimize-max-distance-to-gas-station)
8. [Find Peak Element (LC 162)](#8-find-peak-element)
9. [Search in Rotated Sorted Array (LC 33)](#9-search-in-rotated-sorted-array)
10. [Find Minimum in Rotated Sorted Array (LC 153)](#10-find-minimum-in-rotated-sorted-array)
11. [Median of Two Sorted Arrays (LC 4)](#11-median-of-two-sorted-arrays)
12. [Binary Search Patterns Cheat Sheet](#12-patterns-cheat-sheet)
13. [Top 15 Must-Do Questions](#13-top-15-must-do)

---

## 1. Standard Binary Search

```java
// Standard: Find target in sorted array
public int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;  // Avoids overflow!
        
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;  // Not found
}

// Lower Bound: First index where arr[i] >= target
public int lowerBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;  // First position >= target
}

// Upper Bound: First index where arr[i] > target
public int upperBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= target) lo = mid + 1;
        else hi = mid;
    }
    return lo;  // First position > target
}
```

---

## 2. Binary Search on Answer — The Pattern

### 🧠 When to Use This
The problem asks for a **minimum/maximum value** that satisfies a condition, and:
1. The answer lies in a **range** `[low, high]`
2. You can write a **feasibility check** function: `canDo(mid)` → boolean
3. The feasibility is **monotonic**: if `mid` works, then `mid+1` also works (or vice versa)

### The Template
```java
// Find MINIMUM value that satisfies the condition
int lo = MIN_POSSIBLE_ANSWER;
int hi = MAX_POSSIBLE_ANSWER;

while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    
    if (canDo(mid)) {
        hi = mid;       // mid works, try smaller
    } else {
        lo = mid + 1;   // mid doesn't work, need bigger
    }
}
return lo; // Minimum feasible answer

// Find MAXIMUM value that satisfies the condition
while (lo < hi) {
    int mid = lo + (hi - lo + 1) / 2;  // Note: +1 to avoid infinite loop!
    
    if (canDo(mid)) {
        lo = mid;       // mid works, try bigger
    } else {
        hi = mid - 1;   // mid doesn't work, need smaller
    }
}
return lo; // Maximum feasible answer
```

```
The 3 key questions:

1. What is the SEARCH SPACE?
   → [min possible answer, max possible answer]

2. What is the FEASIBILITY CHECK?
   → Can we achieve the goal with this value of mid?

3. Is the answer MINIMUM or MAXIMUM satisfying the condition?
   → Minimum: hi = mid. Maximum: lo = mid.
```

---

## 3. Koko Eating Bananas (LC 875)

**Problem:** Koko has `n` piles of bananas. She can eat at speed `k` bananas/hour (one pile per hour, even if < k). She has `h` hours. Find minimum `k` to finish all bananas.

`piles = [3,6,7,11], h = 8` → `k = 4`

### 🧠 How to Think
- **Search space:** `k` ranges from `1` to `max(piles)`
- **Feasibility:** With speed `k`, can she finish in ≤ `h` hours?
- **Goal:** Find MINIMUM `k`

### 🧠 Dry Run

```
piles = [3, 6, 7, 11], h = 8

lo=1, hi=11

mid=6: hours = ceil(3/6)+ceil(6/6)+ceil(7/6)+ceil(11/6) = 1+1+2+2 = 6 ≤ 8 → works! hi=6
mid=3: hours = 1+2+3+4 = 10 > 8 → too slow! lo=4
mid=5: hours = 1+2+2+3 = 8 ≤ 8 → works! hi=5
mid=4: hours = 1+2+2+3 = 8 ≤ 8 → works! hi=4
lo==hi==4 → Answer: 4 ✅
```

```java
public int minEatingSpeed(int[] piles, int h) {
    int lo = 1, hi = 0;
    for (int p : piles) hi = Math.max(hi, p);
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (canFinish(piles, mid, h)) {
            hi = mid;       // Can finish, try slower speed
        } else {
            lo = mid + 1;   // Can't finish, need faster
        }
    }
    return lo;
}

private boolean canFinish(int[] piles, int speed, int h) {
    int hours = 0;
    for (int pile : piles) {
        hours += (pile + speed - 1) / speed; // Ceiling division
    }
    return hours <= h;
}
```

---

## 4. Capacity to Ship Packages (LC 1011)

**Problem:** Packages must be shipped in order in `days` days. Find minimum ship capacity.

`weights = [1,2,3,4,5,6,7,8,9,10], days = 5` → **15**

### 🧠 How to Think
- **Search space:** `[max(weights), sum(weights)]`
- **Feasibility:** With capacity `mid`, can we ship in ≤ `days` days?

```java
public int shipWithinDays(int[] weights, int days) {
    int lo = 0, hi = 0;
    for (int w : weights) {
        lo = Math.max(lo, w);  // At least the heaviest package
        hi += w;                // At most all in one day
    }
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (canShip(weights, mid, days)) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return lo;
}

private boolean canShip(int[] weights, int capacity, int days) {
    int daysNeeded = 1, currentLoad = 0;
    
    for (int w : weights) {
        if (currentLoad + w > capacity) {
            daysNeeded++;
            currentLoad = 0;
        }
        currentLoad += w;
    }
    return daysNeeded <= days;
}
```

### 🧠 Dry Run
```
weights = [1,2,3,4,5,6,7,8,9,10], days = 5

lo=10, hi=55

mid=32: days=2 ≤ 5 → hi=32
mid=21: days=3 ≤ 5 → hi=21
mid=15: days=5 ≤ 5 → hi=15
mid=12: days=6 > 5 → lo=13
mid=14: days=6 > 5 → lo=15
lo==hi==15 → Answer: 15 ✅

With capacity 15:
Day 1: [1,2,3,4,5] = 15
Day 2: [6,7] = 13
Day 3: [8] = 8
Day 4: [9] = 9
Day 5: [10] = 10
```

---

## 5. Split Array Largest Sum (LC 410)

**Problem:** Split array into `k` subarrays to minimize the largest subarray sum.

`nums = [7,2,5,10,8], k = 2` → **18** (split: [7,2,5] and [10,8])

### 🧠 How to Think
Same pattern as ship capacity!
- **Search space:** `[max(nums), sum(nums)]`
- **Feasibility:** With max sum `mid`, can we split into ≤ `k` parts?

```java
public int splitArray(int[] nums, int k) {
    int lo = 0, hi = 0;
    for (int num : nums) {
        lo = Math.max(lo, num);
        hi += num;
    }
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (canSplit(nums, mid, k)) {
            hi = mid;       // Possible, try smaller max sum
        } else {
            lo = mid + 1;   // Not possible, need larger max sum
        }
    }
    return lo;
}

private boolean canSplit(int[] nums, int maxSum, int k) {
    int splits = 1, currentSum = 0;
    for (int num : nums) {
        if (currentSum + num > maxSum) {
            splits++;
            currentSum = 0;
        }
        currentSum += num;
    }
    return splits <= k;
}
```

---

## 6. Aggressive Cows / Magnetic Balls (LC 2517)

**Problem:** Place `k` cows in `n` stalls (positions given). Maximize the **minimum distance** between any two cows.

`positions = [1, 2, 4, 8, 9], k = 3` → **3** (place at 1, 4, 8 → min gap = 3)

### 🧠 How to Think
- **Search space:** `[1, max_position - min_position]`
- **Feasibility:** With minimum gap `mid`, can we place `k` cows?
- **Goal:** Find MAXIMUM gap → use `lo = mid` template

```java
public int maxMinDistance(int[] positions, int k) {
    Arrays.sort(positions);
    int lo = 1;
    int hi = positions[positions.length - 1] - positions[0];
    
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2;  // +1 for max search!
        
        if (canPlace(positions, mid, k)) {
            lo = mid;       // Can place, try larger gap
        } else {
            hi = mid - 1;   // Can't place, need smaller gap
        }
    }
    return lo;
}

private boolean canPlace(int[] positions, int minDist, int k) {
    int count = 1;  // Place first cow at positions[0]
    int lastPos = positions[0];
    
    for (int i = 1; i < positions.length; i++) {
        if (positions[i] - lastPos >= minDist) {
            count++;
            lastPos = positions[i];
            if (count >= k) return true;
        }
    }
    return false;
}
```

### 🧠 Dry Run
```
positions = [1, 2, 4, 8, 9], k = 3

lo=1, hi=8

mid=5: canPlace with gap≥5?
  Place at 1, next≥6 → place at 8, next≥13 → no more. count=2 < 3 → NO
  hi=4

mid=3: canPlace with gap≥3?
  Place at 1, next≥4 → place at 4, next≥7 → place at 8. count=3 ≥ 3 → YES
  lo=3

mid=4: canPlace with gap≥4?
  Place at 1, next≥5 → place at 8, next≥12 → no. count=2 < 3 → NO
  hi=3

lo==hi==3 → Answer: 3 ✅ (place at 1, 4, 8)
```

---

## 7. Minimize Max Distance to Gas Station (LC 774)

**Problem:** Insert `k` gas stations on a number line to minimize the maximum gap between any two adjacent stations.

### 🧠 How to Think
Binary search on the answer (the maximum gap distance). For a given gap `d`, count how many stations are needed.

```java
public double minmaxGasDist(int[] stations, int k) {
    double lo = 0, hi = 0;
    for (int i = 1; i < stations.length; i++) {
        hi = Math.max(hi, stations[i] - stations[i - 1]);
    }
    
    while (hi - lo > 1e-6) { // Precision for doubles
        double mid = lo + (hi - lo) / 2;
        
        if (canAchieve(stations, mid, k)) {
            hi = mid;
        } else {
            lo = mid;
        }
    }
    return lo;
}

private boolean canAchieve(int[] stations, double maxGap, int k) {
    int needed = 0;
    for (int i = 1; i < stations.length; i++) {
        double gap = stations[i] - stations[i - 1];
        needed += (int)(gap / maxGap); // How many stations to fill this gap
    }
    return needed <= k;
}
```

---

## 8. Find Peak Element (LC 162)

**Problem:** Find a peak element (greater than neighbors). Array may have multiple peaks. Return any.

### 🧠 How to Think
If `nums[mid] < nums[mid+1]`, peak is to the RIGHT. Else, peak is to the LEFT (or at mid).

```java
public int findPeakElement(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (nums[mid] < nums[mid + 1]) {
            lo = mid + 1;  // Peak is to the right
        } else {
            hi = mid;      // Peak is at mid or to the left
        }
    }
    return lo;
}
```

---

## 9. Search in Rotated Sorted Array (LC 33)

**Problem:** Array was sorted then rotated. Find target in O(log n).

`nums = [4,5,6,7,0,1,2], target = 0` → index **4**

### 🧠 How to Think
One half is always sorted. Determine which half, then check if target is in that half.

```java
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (nums[mid] == target) return mid;
        
        // Left half is sorted
        if (nums[lo] <= nums[mid]) {
            if (target >= nums[lo] && target < nums[mid]) {
                hi = mid - 1;  // Target in left sorted half
            } else {
                lo = mid + 1;  // Target in right half
            }
        }
        // Right half is sorted
        else {
            if (target > nums[mid] && target <= nums[hi]) {
                lo = mid + 1;  // Target in right sorted half
            } else {
                hi = mid - 1;  // Target in left half
            }
        }
    }
    return -1;
}
```

### 🧠 Dry Run
```
nums = [4, 5, 6, 7, 0, 1, 2], target = 0

lo=0, hi=6, mid=3: nums[3]=7 ≠ 0
  Left [4,5,6,7] sorted (nums[0]=4 ≤ nums[3]=7)
  Is 0 in [4,7)? NO → lo=4

lo=4, hi=6, mid=5: nums[5]=1 ≠ 0
  Left [0,1] → nums[4]=0 ≤ nums[5]=1 → sorted
  Is 0 in [0,1)? YES → hi=4

lo=4, hi=4, mid=4: nums[4]=0 == target → return 4 ✅
```

---

## 10. Find Minimum in Rotated Sorted Array (LC 153)

```java
public int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (nums[mid] > nums[hi]) {
            lo = mid + 1;  // Minimum is in right half
        } else {
            hi = mid;      // Minimum is at mid or left
        }
    }
    return nums[lo];
}
```

---

## 11. Median of Two Sorted Arrays (LC 4)

**Problem:** Find median of two sorted arrays in O(log(min(m,n))).

### 🧠 How to Think
Binary search on the **partition point** of the smaller array. Ensure left halves of both arrays form the left half of the merged array.

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    // Ensure nums1 is the smaller array
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);
    
    int m = nums1.length, n = nums2.length;
    int lo = 0, hi = m;
    
    while (lo <= hi) {
        int i = lo + (hi - lo) / 2;     // Partition point in nums1
        int j = (m + n + 1) / 2 - i;    // Partition point in nums2
        
        int maxLeft1 = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
        int minRight1 = (i == m) ? Integer.MAX_VALUE : nums1[i];
        int maxLeft2 = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
        int minRight2 = (j == n) ? Integer.MAX_VALUE : nums2[j];
        
        if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
            // Perfect partition found!
            if ((m + n) % 2 == 0) {
                return (Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2.0;
            } else {
                return Math.max(maxLeft1, maxLeft2);
            }
        } else if (maxLeft1 > minRight2) {
            hi = i - 1;  // Too far right in nums1
        } else {
            lo = i + 1;  // Too far left in nums1
        }
    }
    return -1;
}
```

### 🧠 Dry Run
```
nums1 = [1, 3], nums2 = [2]

m=2, n=1, total=3 (odd → median is middle)

lo=0, hi=2, i=1, j=(3+1)/2-1=1
  maxLeft1=nums1[0]=1, minRight1=nums1[1]=3
  maxLeft2=nums2[0]=2, minRight2=∞
  
  1 ≤ ∞ ✅ AND 2 ≤ 3 ✅ → Perfect partition!
  Odd → median = max(1, 2) = 2.0 ✅
```

---

## 12. Patterns Cheat Sheet

```
┌──────────────────────────────────────────────────────────────┐
│           BINARY SEARCH PATTERN RECOGNITION                   │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  STANDARD BINARY SEARCH:                                      │
│  "Find target in sorted array"      → Classic BS             │
│  "Find insert position"             → Lower/Upper bound      │
│  "Search in rotated array"          → Check which half sorted│
│  "Find peak element"                → Compare mid vs mid+1   │
│                                                               │
│  BINARY SEARCH ON ANSWER:                                     │
│  "Minimum speed/capacity/rate"      → BS + feasibility check │
│  "Maximum gap/distance"             → BS + placement check   │
│  "Minimize the maximum X"           → BS + greedy partition  │
│  "Maximize the minimum X"           → BS + greedy placement  │
│                                                               │
│  HOW TO IDENTIFY:                                             │
│  1. "Minimize the max" or "Maximize the min"  → BS on answer │
│  2. Answer is in a RANGE [lo, hi]              → BS on answer │
│  3. Can verify feasibility in O(N)             → BS on answer │
│  4. Feasibility is MONOTONIC                   → BS on answer │
│                                                               │
│  TEMPLATE:                                                    │
│  Find MIN satisfying: hi = mid, lo = mid+1                   │
│  Find MAX satisfying: lo = mid, hi = mid-1 (+1 in mid calc)  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 13. Top 15 Must-Do Questions

**Standard Binary Search:**
1. Binary Search (LC 704)
2. Search in Rotated Sorted Array (LC 33)
3. Find Minimum in Rotated Sorted Array (LC 153)
4. Find Peak Element (LC 162)
5. Search a 2D Matrix (LC 74)
6. Find First and Last Position (LC 34)

**Binary Search on Answer:**
7. Koko Eating Bananas (LC 875)
8. Capacity to Ship Packages (LC 1011)
9. Split Array Largest Sum (LC 410)
10. Aggressive Cows / Max Min Distance (LC 2517)
11. Minimize Max Distance to Gas Station (LC 774)
12. Magnetic Force Between Two Balls (LC 1552)

**Advanced:**
13. Median of Two Sorted Arrays (LC 4)
14. Kth Smallest Element in Sorted Matrix (LC 378)
15. Find K-th Smallest Pair Distance (LC 719)
