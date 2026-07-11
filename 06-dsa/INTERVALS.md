# 📐 Interval Problems — FAANG Masterclass

> **"Sort by start time, then greedily process — that's 80% of interval problems."**

---

## 📑 Table of Contents
1. [The Interval Pattern](#1-the-interval-pattern)
2. [Merge Intervals (LC 56) — Full Dry Run](#2-merge-intervals)
3. [Insert Interval (LC 57)](#3-insert-interval)
4. [Non-overlapping Intervals (LC 435)](#4-non-overlapping-intervals)
5. [Meeting Rooms I (LC 252)](#5-meeting-rooms-i)
6. [Meeting Rooms II (LC 253) — Min Rooms Needed](#6-meeting-rooms-ii)
7. [Interval List Intersections (LC 986)](#7-interval-list-intersections)
8. [Minimum Number of Arrows (LC 452)](#8-minimum-number-of-arrows)
9. [Employee Free Time (LC 759)](#9-employee-free-time)
10. [Pattern Recognition Cheat Sheet](#10-pattern-recognition)
11. [Top 10 Must-Do Questions](#11-top-10-must-do)

---

## 1. The Interval Pattern

Most interval problems follow this approach:
1. **Sort** by start time (or sometimes end time).
2. **Iterate** and compare current interval with the previous/result.
3. **Merge, count, or skip** based on overlap.

### How to Check Overlap?
```
Two intervals [a, b] and [c, d] overlap if:
  a <= d AND c <= b

  |---A---|
      |---B---|     → Overlap ✅

  |---A---|
              |---B---|  → No overlap ❌ (a > d or c > b)
```

---

## 2. Merge Intervals (LC 56)

**Problem:** Given a collection of intervals, merge all overlapping intervals.

`[[1,3],[2,6],[8,10],[15,18]]` → `[[1,6],[8,10],[15,18]]`

### 🧠 How to Think
1. Sort by start time.
2. For each interval, check if it overlaps with the last merged interval.
3. If overlap → extend the end. If no overlap → add as new interval.

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // Sort by start
    
    List<int[]> merged = new ArrayList<>();
    merged.add(intervals[0]);
    
    for (int i = 1; i < intervals.length; i++) {
        int[] last = merged.get(merged.size() - 1);
        int[] curr = intervals[i];
        
        if (curr[0] <= last[1]) {
            // Overlap → extend end
            last[1] = Math.max(last[1], curr[1]);
        } else {
            // No overlap → add new interval
            merged.add(curr);
        }
    }
    
    return merged.toArray(new int[merged.size()][]);
}
```

### 🧠 Dry Run
```
Input: [[1,3], [2,6], [8,10], [15,18]]
After sort: [[1,3], [2,6], [8,10], [15,18]] (already sorted)

Step | Current    | Last in merged | Overlap?           | Action       | Merged
-----|------------|---------------|--------------------|--------------|---------
Init |            |               |                    | Add [1,3]    | [[1,3]]
  1  | [2,6]      | [1,3]         | 2 ≤ 3 → YES       | Extend to [1,6] | [[1,6]]
  2  | [8,10]     | [1,6]         | 8 > 6 → NO        | Add [8,10]   | [[1,6],[8,10]]
  3  | [15,18]    | [8,10]        | 15 > 10 → NO      | Add [15,18]  | [[1,6],[8,10],[15,18]]

Result: [[1,6], [8,10], [15,18]] ✅
```

---

## 3. Insert Interval (LC 57)

**Problem:** Insert a new interval into a sorted, non-overlapping list. Merge if necessary.

```java
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;
    
    // 1. Add all intervals BEFORE the new one (no overlap)
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i++]);
    }
    
    // 2. Merge overlapping intervals with newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);
    
    // 3. Add all intervals AFTER the new one
    while (i < n) {
        result.add(intervals[i++]);
    }
    
    return result.toArray(new int[result.size()][]);
}
```

### 🧠 Dry Run
```
intervals = [[1,3], [6,9]], newInterval = [2,5]

Phase 1 (before): [1,3] → end 3 ≥ 2 → NOT before → stop. result=[]
Phase 2 (merge):  [1,3] overlaps [2,5] → merge to [1,5]
                  [6,9] → start 6 > 5 → NOT overlapping → stop. result=[[1,5]]
Phase 3 (after):  Add [6,9]. result=[[1,5],[6,9]]

Result: [[1,5], [6,9]] ✅
```

---

## 4. Non-overlapping Intervals (LC 435)

**Problem:** Find minimum number of intervals to REMOVE so remaining intervals don't overlap.

`[[1,2],[2,3],[3,4],[1,3]]` → Remove **1** (remove [1,3])

### 🧠 How to Think (Greedy)
Sort by **end time**. Always keep the interval that ends earliest (greedy: leaves more room for future intervals). Count conflicts.

```java
public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // Sort by END time!
    
    int removals = 0;
    int prevEnd = Integer.MIN_VALUE;
    
    for (int[] interval : intervals) {
        if (interval[0] >= prevEnd) {
            prevEnd = interval[1]; // No overlap, keep this interval
        } else {
            removals++;            // Overlap! Remove this one (it ends later)
        }
    }
    return removals;
}
```

### 🧠 Dry Run
```
Sorted by end: [[1,2], [2,3], [1,3], [3,4]]

prevEnd = -∞

[1,2]: 1 ≥ -∞ → keep. prevEnd = 2
[2,3]: 2 ≥ 2  → keep. prevEnd = 3
[1,3]: 1 < 3  → OVERLAP! removals=1 (remove [1,3])
[3,4]: 3 ≥ 3  → keep. prevEnd = 4

Removals = 1 ✅
```

---

## 5. Meeting Rooms I (LC 252)

**Problem:** Given meeting time intervals, determine if a person can attend all meetings.

```java
public boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i - 1][1]) {
            return false; // Overlap!
        }
    }
    return true;
}
```

---

## 6. Meeting Rooms II (LC 253) — Min Rooms Needed

**Problem:** Find the minimum number of conference rooms needed.

`[[0,30],[5,10],[15,20]]` → **2** rooms

### 🧠 How to Think (3 approaches)

### Approach 1: Min-Heap (Most Intuitive)
Sort by start time. Use a min-heap to track the earliest ending meeting. If current meeting starts after earliest end → reuse room (poll). Otherwise → need new room (push).

```java
public int minMeetingRooms(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // Sort by start
    
    PriorityQueue<Integer> pq = new PriorityQueue<>(); // Min-heap of END times
    
    for (int[] meeting : intervals) {
        if (!pq.isEmpty() && meeting[0] >= pq.peek()) {
            pq.poll(); // Reuse room (earliest ending room is free)
        }
        pq.add(meeting[1]); // Assign room with this end time
    }
    
    return pq.size(); // Number of rooms in use
}
```

### 🧠 Dry Run (Min-Heap)
```
Sorted: [[0,30], [5,10], [15,20]]

Meeting [0,30]: PQ empty → add 30. PQ: [30]. Rooms: 1
Meeting [5,10]: peek=30, 5 < 30 → OVERLAP! Add 10. PQ: [10,30]. Rooms: 2
Meeting [15,20]: peek=10, 15 ≥ 10 → REUSE! Poll 10, add 20. PQ: [20,30]. Rooms: 2

Answer: 2 rooms ✅
```

### Approach 2: Event Line (Sweep Line) — Elegant
Create events: +1 for start, -1 for end. Sort events. Track running count.

```java
public int minMeetingRooms(int[][] intervals) {
    int n = intervals.length;
    int[] starts = new int[n];
    int[] ends = new int[n];
    
    for (int i = 0; i < n; i++) {
        starts[i] = intervals[i][0];
        ends[i] = intervals[i][1];
    }
    
    Arrays.sort(starts);
    Arrays.sort(ends);
    
    int rooms = 0, maxRooms = 0, endPtr = 0;
    
    for (int i = 0; i < n; i++) {
        if (starts[i] < ends[endPtr]) {
            rooms++;        // New meeting started before earliest end
        } else {
            endPtr++;       // A meeting ended, reuse room
        }
        maxRooms = Math.max(maxRooms, rooms);
    }
    return maxRooms;
}
```

---

## 7. Interval List Intersections (LC 986)

**Problem:** Given two sorted interval lists, find all intersections.

```java
public int[][] intervalIntersection(int[][] A, int[][] B) {
    List<int[]> result = new ArrayList<>();
    int i = 0, j = 0;
    
    while (i < A.length && j < B.length) {
        int start = Math.max(A[i][0], B[j][0]);
        int end = Math.min(A[i][1], B[j][1]);
        
        if (start <= end) {
            result.add(new int[]{start, end}); // Intersection exists!
        }
        
        // Move the one that ends first
        if (A[i][1] < B[j][1]) {
            i++;
        } else {
            j++;
        }
    }
    
    return result.toArray(new int[result.size()][]);
}
```

### 🧠 Dry Run
```
A = [[0,2], [5,10], [13,23], [24,25]]
B = [[1,5], [8,12], [15,24], [25,26]]

i=0, j=0: A=[0,2], B=[1,5] → start=max(0,1)=1, end=min(2,5)=2 → [1,2] ✅
  A[0] ends first (2<5) → i++

i=1, j=0: A=[5,10], B=[1,5] → start=5, end=5 → [5,5] ✅
  B[0] ends first (5<10) → j++

i=1, j=1: A=[5,10], B=[8,12] → start=8, end=10 → [8,10] ✅
  A[1] ends first → i++

i=2, j=1: A=[13,23], B=[8,12] → start=13, end=12 → 13>12 → NO intersection
  B[1] ends first → j++

i=2, j=2: A=[13,23], B=[15,24] → start=15, end=23 → [15,23] ✅
  A[2] ends first → i++

i=3, j=2: A=[24,25], B=[15,24] → start=24, end=24 → [24,24] ✅
  B[2] ends first → j++

i=3, j=3: A=[24,25], B=[25,26] → start=25, end=25 → [25,25] ✅

Result: [[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]] ✅
```

---

## 8. Minimum Number of Arrows (LC 452)

**Problem:** Balloons represented as intervals on x-axis. An arrow at x bursts all balloons where `x_start ≤ x ≤ x_end`. Find minimum arrows.

### 🧠 How to Think
Same as "maximum non-overlapping intervals" → Sort by end, greedily shoot at each end.

```java
public int findMinArrowShots(int[][] points) {
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1])); // Sort by END
    
    int arrows = 1;
    int arrowPos = points[0][1]; // Shoot at the end of first balloon
    
    for (int i = 1; i < points.length; i++) {
        if (points[i][0] > arrowPos) {
            // This balloon is NOT hit by current arrow
            arrows++;
            arrowPos = points[i][1];
        }
        // else: balloon is hit (overlaps with arrow position)
    }
    return arrows;
}
```

---

## 9. Employee Free Time (LC 759)

**Problem:** Given multiple employees' schedules (sorted intervals), find common free time slots.

### 🧠 How to Think
Flatten all intervals → Merge → Gaps between merged intervals = free time.

```java
public List<Interval> employeeFreeTime(List<List<Interval>> schedule) {
    // Flatten all intervals
    List<Interval> all = new ArrayList<>();
    for (List<Interval> emp : schedule) {
        all.addAll(emp);
    }
    
    // Sort by start
    all.sort((a, b) -> a.start - b.start);
    
    // Merge and find gaps
    List<Interval> result = new ArrayList<>();
    int prevEnd = all.get(0).end;
    
    for (int i = 1; i < all.size(); i++) {
        if (all.get(i).start > prevEnd) {
            // Gap found → free time!
            result.add(new Interval(prevEnd, all.get(i).start));
        }
        prevEnd = Math.max(prevEnd, all.get(i).end);
    }
    return result;
}
```

---

## 10. Pattern Recognition

```
┌──────────────────────────────────────────────────────────────┐
│           INTERVAL PATTERN RECOGNITION                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  "Merge overlapping intervals"    → Sort by start, merge     │
│  "Insert interval"               → 3 phases (before/merge/  │
│                                      after)                   │
│  "Min removals for no overlap"    → Sort by END, greedy      │
│  "Can attend all meetings?"       → Sort, check overlap      │
│  "Min rooms / max concurrent"     → Min-heap OR sweep line   │
│  "Intersection of two lists"      → Two pointers             │
│  "Min arrows / max non-overlap"   → Sort by END, greedy      │
│  "Free time / gaps"               → Merge, find gaps         │
│                                                               │
│  SORTING RULE:                                                │
│  • Sort by START: merge, insert, meeting rooms               │
│  • Sort by END: max non-overlapping, min removals, arrows    │
│                                                               │
│  OVERLAP CHECK:                                               │
│  • [a,b] and [c,d] overlap ↔ a ≤ d AND c ≤ b               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 11. Top 10 Must-Do Questions

1. Merge Intervals (LC 56) — Foundation
2. Insert Interval (LC 57) — 3-phase approach
3. Non-overlapping Intervals (LC 435) — Sort by end, greedy
4. Meeting Rooms I (LC 252) — Overlap check
5. Meeting Rooms II (LC 253) — Min-heap / sweep line
6. Interval List Intersections (LC 986) — Two pointers
7. Minimum Number of Arrows (LC 452) — Sort by end
8. Employee Free Time (LC 759) — Merge + gaps
9. My Calendar I / II / III (LC 729, 731, 732) — Sweep line
10. Car Pooling (LC 1094) — Sweep line / events
