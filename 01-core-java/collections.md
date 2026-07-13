# ☕ Java Collections Framework — C++ STL → Java Cheat Sheet

> **"You know `vector`, `map`, `set`, `sort()`, `lower_bound()` in C++. Here's EXACTLY how to do the same in Java."**

This guide is written for **C++ competitive programmers switching to Java**. Every section shows the C++ STL equivalent and Java code side by side.

---

## 📑 Table of Contents
1. [Quick STL → Java Mapping Table](#1-quick-stl--java-mapping)
2. [ArrayList (vector)](#2-arraylist--vector)
3. [LinkedList (list)](#3-linkedlist--list)
4. [Stack (stack)](#4-stack)
5. [Queue & Deque (queue / deque)](#5-queue--deque)
6. [PriorityQueue (priority_queue)](#6-priorityqueue--priority_queue)
7. [HashMap (unordered_map)](#7-hashmap--unordered_map)
8. [TreeMap (map / ordered map)](#8-treemap--map)
9. [HashSet (unordered_set)](#9-hashset--unordered_set)
10. [TreeSet (set / ordered set)](#10-treeset--set)
11. [Pair — Java Doesn't Have It! (Workarounds)](#11-pair--workarounds)
12. [int[] array vs vector](#12-arrays)
13. [Sorting (sort / custom comparators)](#13-sorting)
14. [Binary Search (lower_bound / upper_bound)](#14-binary-search)
15. [Collections Utility Methods](#15-collections-utility)
16. [Iterators (begin/end → Iterator/for-each)](#16-iterators)
17. [Streams (C++ algorithms → Java Streams)](#17-streams)
18. [String Operations (C++ string vs Java String)](#18-string-operations)
19. [Common Patterns: C++ → Java Translation](#19-common-patterns)
20. [Pitfalls for C++ Programmers](#20-pitfalls)

---

## 1. Quick STL → Java Mapping

| C++ STL | Java Equivalent | Notes |
|:---|:---|:---|
| `vector<int>` | `ArrayList<Integer>` | No primitives in generics! Use `Integer` |
| `array<int, N>` / `int[]` | `int[]` | Same concept |
| `list<int>` | `LinkedList<Integer>` | Doubly linked |
| `stack<int>` | `Stack<Integer>` or `Deque<Integer>` | `Deque` preferred |
| `queue<int>` | `Queue<Integer>` = `LinkedList<>()` | Interface + implementation |
| `deque<int>` | `ArrayDeque<Integer>` | Faster than LinkedList |
| `priority_queue<int>` | `PriorityQueue<Integer>` | **MIN-HEAP by default!** (C++ is MAX) |
| `unordered_map<K,V>` | `HashMap<K,V>` | O(1) average |
| `map<K,V>` | `TreeMap<K,V>` | O(log N), sorted keys |
| `unordered_set<T>` | `HashSet<T>` | O(1) average |
| `set<T>` | `TreeSet<T>` | O(log N), sorted |
| `multiset<T>` | `TreeMap<T, Integer>` (freq map) | No direct equivalent |
| `pair<A,B>` | `int[]`, `Map.Entry`, or custom class | **No built-in Pair!** |
| `tuple` | No equivalent | Use `int[]` or custom class |
| `bitset<N>` | `BitSet` | Dynamic size |
| `sort()` | `Arrays.sort()` / `Collections.sort()` | |
| `lower_bound()` | `TreeSet.ceiling()` / `Arrays.binarySearch()` | |
| `upper_bound()` | `TreeSet.higher()` | |
| `reverse()` | `Collections.reverse()` | |
| `next_permutation()` | No built-in | Must implement manually |
| `__builtin_popcount()` | `Integer.bitCount()` | |
| `INT_MAX` | `Integer.MAX_VALUE` | |
| `INT_MIN` | `Integer.MIN_VALUE` | |
| `1e18` / `LLONG_MAX` | `Long.MAX_VALUE` | |

---

## 2. ArrayList (= `vector<int>`)

```cpp
// C++
vector<int> v;
v.push_back(10);          // Add to end
v.pop_back();              // Remove last
v[0];                      // Access by index
v.size();                  // Size
v.empty();                 // Is empty?
v.clear();                 // Clear all
v.begin(), v.end();        // Iterators
vector<int> v(n, 0);       // n elements, all 0
```

```java
// Java
ArrayList<Integer> v = new ArrayList<>();
v.add(10);                 // push_back
v.remove(v.size() - 1);   // pop_back (remove last — by INDEX)
v.get(0);                 // v[0] — NO bracket access!
v.size();                  // size
v.isEmpty();               // empty
v.clear();                 // clear
// No begin/end — use for-each or Iterator
List<Integer> v = new ArrayList<>(Collections.nCopies(n, 0)); // n elements, all 0
```

### Common Operations
```java
ArrayList<Integer> list = new ArrayList<>();

// Add
list.add(5);                    // [5]
list.add(10);                   // [5, 10]
list.add(0, 3);                 // Insert at index 0 → [3, 5, 10]

// Access
int val = list.get(1);          // 5 (use get(), NOT list[1])
list.set(1, 99);                // list[1] = 99 → [3, 99, 10]

// Remove
list.remove(0);                 // Remove by INDEX → [99, 10]
list.remove(Integer.valueOf(10)); // Remove by VALUE → [99]

// Search
boolean found = list.contains(99);  // true
int idx = list.indexOf(99);         // 0

// Convert
int[] arr = list.stream().mapToInt(i -> i).toArray();  // ArrayList → int[]
List<Integer> list2 = Arrays.stream(arr).boxed().collect(Collectors.toList()); // int[] → List

// Sort
Collections.sort(list);                        // Ascending
Collections.sort(list, Collections.reverseOrder()); // Descending

// Sublist (like v.begin()+2, v.begin()+5)
List<Integer> sub = list.subList(2, 5);  // [index 2, 3, 4]

// Initialize with values
List<Integer> list3 = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
List<Integer> list4 = List.of(1, 2, 3); // Immutable! Can't add/remove
```

### ⚠️ C++ `v[i]` vs Java `v.get(i)`
```java
// C++ allows: v[i] = 5;
// Java: v.set(i, 5);   and   int x = v.get(i);
// Java ArrayList does NOT support [] bracket notation!
```

---

## 3. LinkedList (= `list<int>`)

```java
LinkedList<Integer> ll = new LinkedList<>();

ll.addFirst(1);           // push_front → [1]
ll.addLast(2);             // push_back → [1, 2]
ll.add(1, 5);              // Insert at index 1 → [1, 5, 2]

ll.removeFirst();          // pop_front → [5, 2]
ll.removeLast();           // pop_back → [5]

ll.getFirst();             // front()
ll.getLast();               // back()

// LinkedList also implements Queue and Deque!
// You can use it as a queue:
Queue<Integer> q = new LinkedList<>();
```

---

## 4. Stack (= `stack<int>`)

```cpp
// C++
stack<int> st;
st.push(5);    st.top();    st.pop();    st.empty();    st.size();
```

```java
// Java (Option 1: Stack class)
Stack<Integer> st = new Stack<>();
st.push(5);        // Push
st.peek();          // top() — doesn't remove
st.pop();           // Pop — returns AND removes
st.isEmpty();       // empty
st.size();          // size

// Java (Option 2: ArrayDeque as Stack — PREFERRED, faster!)
Deque<Integer> st = new ArrayDeque<>();
st.push(5);         // Push
st.peek();           // Top
st.pop();            // Pop
st.isEmpty();
```

---

## 5. Queue & Deque

### Queue (= `queue<int>`)
```cpp
// C++
queue<int> q;
q.push(5);    q.front();    q.pop();    q.empty();
```

```java
// Java — Queue is an INTERFACE, use LinkedList or ArrayDeque
Queue<Integer> q = new LinkedList<>();
q.add(5);           // push (throws exception if full)
q.offer(5);         // push (returns false if full — safer)
q.peek();           // front() — doesn't remove
q.poll();           // pop() — returns AND removes (null if empty)
q.remove();         // pop() — throws exception if empty
q.isEmpty();
q.size();
```

### Deque (= `deque<int>`) — Double-ended queue
```java
Deque<Integer> dq = new ArrayDeque<>();

dq.addFirst(1);     // push_front
dq.addLast(2);      // push_back
dq.peekFirst();     // front()
dq.peekLast();      // back()
dq.pollFirst();     // pop_front (returns and removes)
dq.pollLast();      // pop_back

// ArrayDeque is FASTER than LinkedList for both Stack and Queue!
```

### ⚠️ C++ `q.pop()` vs Java `q.poll()`
```
C++: q.front() gives value, q.pop() removes (returns void)
Java: q.poll() does BOTH (returns the value AND removes it)
      q.peek() just looks without removing
```

---

## 6. PriorityQueue (= `priority_queue<int>`)

### ⚠️ BIGGEST GOTCHA: Java PQ is MIN-HEAP, C++ is MAX-HEAP!

```cpp
// C++ — MAX heap by default
priority_queue<int> pq;              // Max-heap
priority_queue<int, vector<int>, greater<int>> pq; // Min-heap
```

```java
// Java — MIN heap by default!
PriorityQueue<Integer> minPQ = new PriorityQueue<>();             // Min-heap
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(Collections.reverseOrder()); // Max-heap

minPQ.add(5);        // push
minPQ.peek();        // top (smallest element)
minPQ.poll();        // pop (removes and returns smallest)
minPQ.size();
minPQ.isEmpty();

// Custom comparator (like C++ custom compare)
// Sort by second element of int[]
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
pq.add(new int[]{1, 5});
pq.add(new int[]{2, 3});
pq.poll();  // Returns [2, 3] (smaller second element)
```

### Common PQ Patterns
```java
// Min-heap of pairs (distance, node) — for Dijkstra
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
// OR
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

// Heap of strings by length
PriorityQueue<String> pq = new PriorityQueue<>((a, b) -> a.length() - b.length());
```

---

## 7. HashMap (= `unordered_map<K,V>`)

```cpp
// C++
unordered_map<string, int> mp;
mp["hello"] = 5;          // Insert/update
mp["hello"];               // Access (creates default if missing!)
mp.count("hello");         // 1 if exists, 0 if not
mp.erase("hello");         // Remove
for (auto& [k, v] : mp)   // Iterate
```

```java
// Java
HashMap<String, Integer> map = new HashMap<>();
map.put("hello", 5);                // Insert/update
map.get("hello");                   // Access (returns null if missing, NOT 0!)
map.getOrDefault("hello", 0);      // Access with default (like C++ mp["hello"])
map.containsKey("hello");           // count() — returns boolean
map.remove("hello");                // erase
map.size();
map.isEmpty();

// Iterate
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    int value = entry.getValue();
}
// OR simpler:
map.forEach((key, value) -> System.out.println(key + ": " + value));
```

### Frequency Counting (SUPER common in DSA)
```cpp
// C++
for (int x : arr) mp[x]++;
```
```java
// Java — 3 ways (pick ONE and master it)

// Way 1: getOrDefault (most readable)
for (int x : arr) map.put(x, map.getOrDefault(x, 0) + 1);

// Way 2: merge (most concise)
for (int x : arr) map.merge(x, 1, Integer::sum);

// Way 3: compute
for (int x : arr) map.compute(x, (k, v) -> v == null ? 1 : v + 1);
```

### Useful HashMap Methods
```java
map.putIfAbsent("key", 0);         // Only insert if key doesn't exist
map.merge("key", 1, Integer::sum); // If key exists, apply function; else insert
map.computeIfAbsent("key", k -> new ArrayList<>()); // Create value lazily

// Check and update atomically
map.compute("key", (k, v) -> v == null ? 1 : v + 1);

// Get all keys / values
Set<String> keys = map.keySet();
Collection<Integer> values = map.values();
```

---

## 8. TreeMap (= `map<K,V>` — Ordered Map)

TreeMap keeps keys **sorted** (like C++ `map`). Supports log(N) floor/ceiling operations.

```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(5, "five");
tm.put(1, "one");
tm.put(10, "ten");

// Sorted order: {1=one, 5=five, 10=ten}

tm.firstKey();              // 1 (smallest key) — like mp.begin()->first
tm.lastKey();               // 10 (largest key) — like mp.rbegin()->first

// ═══ THIS IS THE KILLER FEATURE (replaces lower_bound/upper_bound!) ═══
tm.floorKey(7);             // 5  — largest key ≤ 7 (like --upper_bound)
tm.ceilingKey(7);           // 10 — smallest key ≥ 7 (like lower_bound)
tm.lowerKey(5);             // 1  — largest key STRICTLY < 5
tm.higherKey(5);            // 10 — smallest key STRICTLY > 5

tm.headMap(5);              // {1=one} — keys < 5
tm.tailMap(5);              // {5=five, 10=ten} — keys ≥ 5
tm.subMap(1, 10);           // {1=one, 5=five} — keys in [1, 10)

// Navigation
Map.Entry<Integer, String> e = tm.firstEntry();  // 1=one
Map.Entry<Integer, String> e = tm.pollFirstEntry(); // Remove & return smallest
```

### C++ lower_bound/upper_bound → Java TreeMap
```cpp
// C++
auto it = mp.lower_bound(x);  // First key >= x
auto it = mp.upper_bound(x);  // First key > x
```
```java
// Java
Integer key = tm.ceilingKey(x);  // First key >= x (lower_bound)
Integer key = tm.higherKey(x);   // First key > x  (upper_bound)
Integer key = tm.floorKey(x);    // Largest key <= x
Integer key = tm.lowerKey(x);    // Largest key < x

// Returns null if no such key exists!
```

---

## 9. HashSet (= `unordered_set<T>`)

```java
HashSet<Integer> set = new HashSet<>();
set.add(5);                 // insert
set.remove(5);              // erase
set.contains(5);            // count/find
set.size();
set.isEmpty();

// Initialize from array
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));

// Set operations
set1.retainAll(set2);       // Intersection (modifies set1)
set1.addAll(set2);          // Union
set1.removeAll(set2);       // Difference
```

---

## 10. TreeSet (= `set<T>` — Ordered Set)

TreeSet is a **sorted set** with O(log N) operations. This is the Java equivalent of C++ `set`.

```java
TreeSet<Integer> ts = new TreeSet<>();
ts.add(5); ts.add(1); ts.add(10); ts.add(3);
// Sorted: [1, 3, 5, 10]

ts.first();                 // 1 — *s.begin()
ts.last();                  // 10 — *s.rbegin()
ts.pollFirst();             // Remove & return smallest
ts.pollLast();              // Remove & return largest

// ═══ lower_bound / upper_bound equivalents ═══
ts.ceiling(4);              // 5  — smallest element ≥ 4 (lower_bound)
ts.higher(5);               // 10 — smallest element > 5  (upper_bound)
ts.floor(4);                // 3  — largest element ≤ 4
ts.lower(5);                // 3  — largest element < 5

ts.headSet(5);              // [1, 3] — elements < 5
ts.tailSet(5);              // [5, 10] — elements ≥ 5
ts.subSet(3, 10);           // [3, 5] — elements in [3, 10)

// Returns null if no such element exists!
```

### ⚠️ Java Has No `multiset`!
```java
// C++ multiset allows duplicates. Java TreeSet does NOT.
// Workaround 1: TreeMap<Integer, Integer> (value → count)
TreeMap<Integer, Integer> multiset = new TreeMap<>();
// Add:
multiset.merge(val, 1, Integer::sum);
// Remove one occurrence:
if (multiset.merge(val, -1, Integer::sum) == 0) multiset.remove(val);

// Workaround 2: TreeMap with custom key to break ties
// Store as TreeMap<int[], Boolean> with custom comparator
```

---

## 11. Pair — Java Doesn't Have It!

### ⚠️ Java Has No `pair<int,int>`!

```cpp
// C++ — so easy!
pair<int,int> p = {5, 10};
p.first;  p.second;
vector<pair<int,int>> v;
sort(v.begin(), v.end()); // Sorts by first, then second
```

### Java Workarounds

```java
// ═══ Option 1: int[] array (MOST COMMON in competitive/interviews) ═══
int[] pair = {5, 10};     // pair[0] = first, pair[1] = second
List<int[]> list = new ArrayList<>();
list.add(new int[]{5, 10});
list.add(new int[]{3, 7});
list.sort((a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]); // Sort by first, then second

// ═══ Option 2: Map.Entry (for single pairs) ═══
Map.Entry<Integer, Integer> pair = Map.entry(5, 10);
pair.getKey();    // 5
pair.getValue();  // 10

// ═══ Option 3: Custom class (cleanest for complex code) ═══
record Pair(int first, int second) implements Comparable<Pair> {
    public int compareTo(Pair other) {
        return this.first != other.first 
            ? Integer.compare(this.first, other.first)
            : Integer.compare(this.second, other.second);
    }
}
Pair p = new Pair(5, 10);
p.first();  p.second();

// ═══ Option 4: Two separate arrays (fastest for CP) ═══
int[] first = new int[n];
int[] second = new int[n];
```

### Triple / Tuple
```java
// C++ tuple<int,int,int> → Java int[3]
int[] triple = {1, 2, 3};

// For PriorityQueue with 3 values:
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
pq.add(new int[]{distance, row, col});
```

---

## 12. Arrays

### Primitive Arrays (= C++ `int arr[N]`)
```java
int[] arr = new int[5];              // [0, 0, 0, 0, 0] — auto-initialized to 0!
int[] arr = {1, 2, 3, 4, 5};        // Initialize with values
int[][] grid = new int[m][n];       // 2D array (auto 0)
int[][] grid = {{1,2},{3,4}};       // 2D initialize

arr.length;                          // Size (NOT size(), it's a FIELD)
Arrays.sort(arr);                    // Sort ascending
Arrays.sort(arr, 0, 5);             // Sort range [0, 5) — like sort(a+0, a+5)
Arrays.fill(arr, -1);               // Fill all with -1 (like memset)
Arrays.fill(arr, 2, 5, -1);         // Fill range [2, 5) with -1
Arrays.copyOf(arr, arr.length);     // Copy (like = in C++)
Arrays.equals(arr1, arr2);          // Compare arrays
Arrays.toString(arr);               // Print: "[1, 2, 3, 4, 5]"
Arrays.deepToString(grid);          // Print 2D array

// Binary search (array MUST be sorted)
int idx = Arrays.binarySearch(arr, target); // Returns index or -(insertionPoint)-1
```

### ⚠️ C++ memset vs Java Arrays.fill
```cpp
// C++ memset only works for 0 and -1 on int arrays
memset(arr, 0, sizeof(arr));
memset(arr, -1, sizeof(arr));
```
```java
// Java Arrays.fill works for ANY value
Arrays.fill(arr, 0);
Arrays.fill(arr, -1);
Arrays.fill(arr, Integer.MAX_VALUE);  // Works!

// For 2D arrays, fill row by row:
for (int[] row : grid) Arrays.fill(row, -1);
```

---

## 13. Sorting

### Basic Sort
```cpp
// C++
sort(v.begin(), v.end());                    // Ascending
sort(v.begin(), v.end(), greater<int>());    // Descending
sort(v.begin() + 2, v.begin() + 5);         // Partial sort [2, 5)
```

```java
// Java — Primitive array
Arrays.sort(arr);                            // Ascending
// NO built-in descending for primitives!
// Workaround: sort ascending, then reverse manually
// OR use Integer[] instead of int[]

Integer[] arr2 = {5, 3, 1, 4, 2};
Arrays.sort(arr2, Collections.reverseOrder()); // Descending ✅

// Partial sort
Arrays.sort(arr, 2, 5);                     // Sort indices [2, 5)

// Java — ArrayList
Collections.sort(list);                      // Ascending
Collections.sort(list, Collections.reverseOrder()); // Descending
list.sort(Comparator.naturalOrder());         // Same as above
list.sort(Comparator.reverseOrder());
```

### Custom Comparator (= C++ lambda/functor)
```cpp
// C++
sort(v.begin(), v.end(), [](auto& a, auto& b) {
    return a.second < b.second;  // Sort by second element
});
```

```java
// Java — Lambda comparator
// Sort int[][] by second element
Arrays.sort(intervals, (a, b) -> a[1] - b[1]);

// Sort by first element, then by second
Arrays.sort(arr, (a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);

// Sort strings by length
Arrays.sort(words, (a, b) -> a.length() - b.length());
// OR cleaner:
Arrays.sort(words, Comparator.comparingInt(String::length));

// Sort objects by multiple fields
list.sort(Comparator.comparingInt(Person::getAge)
                     .thenComparing(Person::getName));

// ⚠️ USE Integer.compare() to avoid overflow!
Arrays.sort(arr, (a, b) -> Integer.compare(a[0], b[0])); // SAFE
// NOT: (a, b) -> a[0] - b[0]  ← can overflow if a[0] is close to Integer.MAX_VALUE
```

### ⚠️ You CANNOT sort `int[]` in descending order directly!
```java
// This does NOT work:
// Arrays.sort(intArray, Collections.reverseOrder()); ← COMPILE ERROR!

// Workaround 1: Sort ascending, then reverse
Arrays.sort(arr);
// Reverse manually:
for (int i = 0, j = arr.length - 1; i < j; i++, j--) {
    int tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp;
}

// Workaround 2: Use Integer[] instead of int[]
Integer[] arr = Arrays.stream(intArr).boxed().toArray(Integer[]::new);
Arrays.sort(arr, Collections.reverseOrder());
```

---

## 14. Binary Search (lower_bound / upper_bound)

### ⚠️ Java's `Arrays.binarySearch` is NOT like `lower_bound`!

```cpp
// C++
lower_bound(v.begin(), v.end(), x);  // First element >= x
upper_bound(v.begin(), v.end(), x);  // First element > x
```

```java
// Java Arrays.binarySearch:
// - Returns index if found
// - Returns -(insertionPoint) - 1 if NOT found
int idx = Arrays.binarySearch(arr, target);
// If idx >= 0: found at index idx
// If idx < 0: not found, insertion point = -(idx + 1)

// ═══ Implementing lower_bound (first index >= target) ═══
public static int lowerBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

// ═══ Implementing upper_bound (first index > target) ═══
public static int upperBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

// ═══ Using TreeSet (easiest replacement) ═══
TreeSet<Integer> ts = new TreeSet<>(Arrays.asList(1, 3, 5, 7, 9));
ts.ceiling(4);   // 5  → lower_bound(4)  — first >= 4
ts.higher(5);    // 7  → upper_bound(5)  — first > 5
ts.floor(4);     // 3  → largest <= 4
ts.lower(5);     // 3  → largest < 5
```

---

## 15. Collections Utility Methods

```java
// ═══ Collections class (static methods for Lists) ═══
Collections.sort(list);                     // Sort
Collections.reverse(list);                  // Reverse
Collections.swap(list, i, j);              // Swap elements
Collections.min(list);                      // Min element
Collections.max(list);                      // Max element
Collections.frequency(list, element);      // Count occurrences
Collections.shuffle(list);                  // Random shuffle
Collections.fill(list, value);             // Fill all with value
Collections.nCopies(n, value);             // Create n copies
Collections.unmodifiableList(list);        // Make immutable
Collections.singletonList(x);             // List with one element
Collections.emptyList();                    // Empty immutable list
Collections.binarySearch(list, target);    // Binary search (list must be sorted)

// ═══ Arrays class (static methods for arrays) ═══
Arrays.sort(arr);
Arrays.sort(arr, fromIndex, toIndex);
Arrays.binarySearch(arr, target);
Arrays.fill(arr, value);
Arrays.copyOf(arr, newLength);
Arrays.copyOfRange(arr, from, to);
Arrays.equals(arr1, arr2);
Arrays.toString(arr);                       // Print 1D
Arrays.deepToString(arr2D);                // Print 2D
Arrays.stream(arr);                         // Convert to stream

// ═══ Math class ═══
Math.max(a, b);    Math.min(a, b);
Math.abs(x);       Math.pow(base, exp);    // Returns double!
Math.sqrt(x);      Math.log(x);
Math.ceil(x);      Math.floor(x);          // Returns double!
(int) Math.ceil((double) a / b);            // Ceiling division
// OR: (a + b - 1) / b                     // Integer ceiling division trick

// ═══ Integer class ═══
Integer.MAX_VALUE;                          // 2147483647
Integer.MIN_VALUE;                          // -2147483648
Integer.parseInt("123");                    // String → int
Integer.toString(123);                      // int → String
Integer.bitCount(n);                        // __builtin_popcount
Integer.numberOfLeadingZeros(n);            // __builtin_clz
Integer.numberOfTrailingZeros(n);           // __builtin_ctz
Integer.highestOneBit(n);                   // Highest set bit
Integer.toBinaryString(n);                  // "10110"
```

---

## 16. Iterators

### C++ iterators vs Java
```cpp
// C++ — Iterator based
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << endl;
}
```

```java
// Java — For-each (PREFERRED, use 99% of the time)
for (int x : list) {
    System.out.println(x);
}

// Java — Iterator (use when you need to REMOVE during iteration)
Iterator<Integer> it = list.iterator();
while (it.hasNext()) {
    int val = it.next();
    if (val == 5) it.remove();  // Safe removal during iteration!
}

// ⚠️ You CANNOT remove elements during for-each! It throws ConcurrentModificationException.
// for (int x : list) list.remove(x);  ← CRASH!

// Java — ListIterator (bidirectional, like C++ bidirectional iterator)
ListIterator<Integer> lit = list.listIterator();
while (lit.hasNext()) {
    int val = lit.next();
    lit.set(val * 2);  // Modify current element
}
```

### Iterate over Map
```java
// Key-Value pairs
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    entry.getKey();
    entry.getValue();
}

// Keys only
for (String key : map.keySet()) { ... }

// Values only
for (int value : map.values()) { ... }

// Lambda
map.forEach((k, v) -> System.out.println(k + "=" + v));
```

---

## 17. Streams (C++ algorithms → Java Streams)

Java Streams replace many C++ `<algorithm>` functions:

```java
int[] arr = {3, 1, 4, 1, 5, 9};

// Sum of array
int sum = Arrays.stream(arr).sum();                    // 23

// Max / Min
int max = Arrays.stream(arr).max().getAsInt();         // 9
int min = Arrays.stream(arr).min().getAsInt();         // 1

// Filter + Collect
List<Integer> evens = Arrays.stream(arr)
    .filter(x -> x % 2 == 0)
    .boxed()
    .collect(Collectors.toList());                      // [4]

// Map (transform each element)
int[] doubled = Arrays.stream(arr).map(x -> x * 2).toArray(); // [6,2,8,2,10,18]

// Count
long count = Arrays.stream(arr).filter(x -> x > 3).count();  // 3

// Distinct (unique elements)
int[] unique = Arrays.stream(arr).distinct().toArray();        // [3,1,4,5,9]

// Reduce (accumulate)
int product = Arrays.stream(arr).reduce(1, (a, b) -> a * b);

// int[] → ArrayList<Integer>
List<Integer> list = Arrays.stream(arr).boxed().collect(Collectors.toList());

// ArrayList<Integer> → int[]
int[] arr2 = list.stream().mapToInt(i -> i).toArray();

// String joining
String s = list.stream().map(String::valueOf).collect(Collectors.joining(", "));
```

---

## 18. String Operations

### ⚠️ Java String is IMMUTABLE (unlike C++ `string`)!

```java
// ═══ IMMUTABLE String ═══
String s = "hello";
s.length();                     // 5 (NOT size())
s.charAt(0);                    // 'h' (NOT s[0])
s.substring(1, 3);              // "el" [start, end)
s.indexOf("ll");                // 2
s.contains("ell");              // true
s.equals("hello");              // true — ALWAYS use .equals(), NEVER ==
s.compareTo("world");           // Lexicographic comparison
s.toCharArray();                // char[] for iteration
s.split(" ");                   // Split by space
s.trim();                       // Remove leading/trailing whitespace
s.toLowerCase();
s.toUpperCase();
s.replace('l', 'r');            // "herro"
s.startsWith("hel");            // true
s.isEmpty();                    // false

// ═══ MUTABLE StringBuilder (like C++ string) ═══
StringBuilder sb = new StringBuilder();
sb.append("hello");             // += "hello"
sb.append('!');                 // += '!'
sb.insert(5, " world");        // Insert at position
sb.deleteCharAt(sb.length()-1); // Remove last char (like pop_back)
sb.delete(0, 5);                // Remove range [0, 5)
sb.reverse();                   // Reverse in-place
sb.charAt(0);                   // Access
sb.setCharAt(0, 'H');          // Modify
sb.length();
sb.toString();                  // Convert to String

// ⚠️ NEVER do string concatenation in a loop! Use StringBuilder!
// BAD (O(N²)):
String result = "";
for (int i = 0; i < n; i++) result += "x";  // Creates new String every time!

// GOOD (O(N)):
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) sb.append("x");
String result = sb.toString();
```

### C++ string vs Java String
```
C++: s[i] = 'x';          Java: sb.setCharAt(i, 'x'); // Only StringBuilder
C++: s += "abc";           Java: sb.append("abc");
C++: s.substr(2, 3);       Java: s.substring(2, 5);    // Java: [start, end) not (start, length)!
C++: s.find("abc");        Java: s.indexOf("abc");
C++: s.size();             Java: s.length();
C++: reverse(s.begin(), s.end());   Java: new StringBuilder(s).reverse().toString();
C++: to_string(123);       Java: String.valueOf(123);  or  Integer.toString(123);
C++: stoi("123");          Java: Integer.parseInt("123");
```

---

## 19. Common Patterns: C++ → Java Translation

### Reading Input (CP style)
```java
import java.util.*;
import java.io.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        // OR for faster I/O:
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) arr[i] = sc.nextInt();
        
        String s = sc.next();       // Read one word
        String line = sc.nextLine(); // Read entire line
    }
}
```

### Common DSA Snippets

```java
// ═══ Swap (no built-in swap for primitives) ═══
// C++: swap(a, b);
int temp = a; a = b; b = temp;
// For arrays: just swap by index
int t = arr[i]; arr[i] = arr[j]; arr[j] = t;

// ═══ Reverse array (no std::reverse for int[]) ═══
// C++: reverse(v.begin(), v.end());
for (int i = 0, j = arr.length - 1; i < j; i++, j--) {
    int t = arr[i]; arr[i] = arr[j]; arr[j] = t;
}
// For ArrayList:
Collections.reverse(list);

// ═══ Max element in array ═══
// C++: *max_element(v.begin(), v.end());
int max = Arrays.stream(arr).max().getAsInt();
// OR manual loop (faster):
int max = arr[0]; for (int x : arr) max = Math.max(max, x);

// ═══ GCD ═══
// C++: __gcd(a, b);
// Java (no built-in before Java 9):
int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }

// ═══ Fast Power (modular exponentiation) ═══
long power(long base, long exp, long mod) {
    long result = 1;
    base %= mod;
    while (exp > 0) {
        if ((exp & 1) == 1) result = result * base % mod;
        exp >>= 1;
        base = base * base % mod;
    }
    return result;
}

// ═══ next_permutation (NO built-in!) ═══
public boolean nextPermutation(int[] arr) {
    int i = arr.length - 2;
    while (i >= 0 && arr[i] >= arr[i + 1]) i--;
    if (i < 0) return false;
    int j = arr.length - 1;
    while (arr[j] <= arr[i]) j--;
    int tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp;
    // Reverse from i+1 to end
    for (int l = i + 1, r = arr.length - 1; l < r; l++, r--) {
        tmp = arr[l]; arr[l] = arr[r]; arr[r] = tmp;
    }
    return true;
}
```

### Adjacency List (Graph)
```cpp
// C++
vector<vector<int>> adj(n);
adj[0].push_back(1);
```
```java
// Java
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
adj.get(0).add(1);

// Weighted graph
List<List<int[]>> adj = new ArrayList<>();  // int[] = {neighbor, weight}
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
adj.get(0).add(new int[]{1, 5}); // Edge 0→1 with weight 5
```

---

## 20. Pitfalls for C++ Programmers

### 1. `==` vs `.equals()` for Strings (and all objects)
```java
String a = "hello";
String b = new String("hello");
a == b;        // false! (compares REFERENCES, not values)
a.equals(b);   // true!  (compares VALUES)

// For Integer: == works for -128 to 127 (cached), fails beyond!
Integer x = 200, y = 200;
x == y;        // false! (different objects)
x.equals(y);   // true!
```

### 2. No Primitives in Generics
```java
// This does NOT work:
// ArrayList<int> list;     ← COMPILE ERROR!
// HashMap<int, int> map;   ← COMPILE ERROR!

// Use wrapper types:
ArrayList<Integer> list;
HashMap<Integer, Integer> map;

// Autoboxing handles conversion automatically:
list.add(5);        // int 5 → Integer.valueOf(5) automatically
int x = list.get(0); // Integer → int automatically
```

### 3. ArrayList.remove() by Index vs by Value
```java
ArrayList<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 2));

list.remove(2);                  // Removes element at INDEX 2 → [1, 2, 2]
list.remove(Integer.valueOf(2)); // Removes first occurrence of VALUE 2 → [1, 3, 2]
```

### 4. Array Size: `.length` vs `.length()` vs `.size()`
```java
int[] arr;        arr.length;      // FIELD (no parentheses!)
String s;         s.length();      // METHOD (with parentheses!)
ArrayList list;   list.size();     // METHOD (different name!)
```

### 5. Integer Overflow in Comparators
```java
// ⚠️ DANGEROUS:
Arrays.sort(arr, (a, b) -> a[0] - b[0]);  // Can overflow if a[0] = MAX_VALUE!

// ✅ SAFE:
Arrays.sort(arr, (a, b) -> Integer.compare(a[0], b[0]));
```

### 6. Java `char` Tricks
```java
char c = 'a';
int idx = c - 'a';              // 0 (same as C++)
char next = (char)(c + 1);      // 'b' (need explicit cast!)
boolean isDigit = Character.isDigit(c);
boolean isLetter = Character.isLetter(c);
```

### 7. 2D Array Gotcha
```java
// C++ vector<vector<int>> v(m, vector<int>(n, 0)); 
// Java:
int[][] grid = new int[m][n]; // Auto-initialized to 0 ✅

// But for ArrayList of ArrayLists:
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>()); // MUST initialize each!
// adj.get(0) returns an ArrayList, NOT null
```

### 8. HashMap `get()` Returns `null`, Not `0`
```java
// C++ map: mp[key] returns 0 if key doesn't exist (creates it)
// Java HashMap: map.get(key) returns NULL if key doesn't exist!

map.get("missing");                    // null (NOT 0!)
map.getOrDefault("missing", 0);       // 0 (explicitly set default)
```

### 9. No `auto` Keyword (until Java 10 `var`)
```java
// C++: auto it = mp.begin();
// Java 10+: var it = map.entrySet().iterator();
// Java 8:   Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();
```

### 10. Copy vs Reference
```java
// Arrays: assignment copies REFERENCE (not values!)
int[] a = {1, 2, 3};
int[] b = a;          // b points to SAME array as a!
b[0] = 99;            // a[0] is also 99 now!

// To copy:
int[] b = a.clone();                    // Shallow copy
int[] b = Arrays.copyOf(a, a.length);   // Shallow copy

// ArrayList: same issue
List<Integer> b = new ArrayList<>(a);   // Copy constructor
```
