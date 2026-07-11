# 🔙 Backtracking — FAANG Masterclass

> **"Backtracking is just DFS with a twist — if a path doesn't lead to a solution, UNDO the last step and try another. It's controlled brute force."**

---

## 📑 Table of Contents
1. [What is Backtracking?](#1-what-is-backtracking)
2. [The Backtracking Template (MEMORIZE!)](#2-the-backtracking-template)
3. [Subsets (LC 78)](#3-subsets)
4. [Subsets II — With Duplicates (LC 90)](#4-subsets-ii)
5. [Permutations (LC 46)](#5-permutations)
6. [Permutations II — With Duplicates (LC 47)](#6-permutations-ii)
7. [Combinations (LC 77)](#7-combinations)
8. [Combination Sum (LC 39, 40, 216)](#8-combination-sum)
9. [Letter Combinations of Phone Number (LC 17)](#9-letter-combinations-of-phone-number)
10. [Palindrome Partitioning (LC 131)](#10-palindrome-partitioning)
11. [N-Queens (LC 51)](#11-n-queens)
12. [Sudoku Solver (LC 37)](#12-sudoku-solver)
13. [Word Search (LC 79)](#13-word-search)
14. [Generate Parentheses (LC 22)](#14-generate-parentheses)
15. [Backtracking vs DP vs Greedy](#15-backtracking-vs-dp-vs-greedy)
16. [Pattern Recognition Cheat Sheet](#16-pattern-recognition)
17. [Top 15 Must-Do Backtracking Questions](#17-top-15-must-do)

---

## 1. What is Backtracking?

Backtracking = **Recursion + Undo**. You explore all possible paths, and whenever a path doesn't work, you **backtrack** (undo your last choice) and try the next option.

```
Think of it like navigating a maze:
  1. Pick a direction (left, right, straight)
  2. Walk forward
  3. Hit a dead end? → BACKTRACK (walk backward to last fork)
  4. Try the NEXT direction
  5. Repeat until you find the exit

In code:
  1. CHOOSE — Make a decision (add element, place queen, etc.)
  2. EXPLORE — Recurse deeper
  3. UN-CHOOSE — Remove the decision (backtrack!)
```

### When to Use Backtracking?
- "Find **ALL** possible solutions" (subsets, permutations, combinations)
- "Generate all valid configurations" (N-Queens, Sudoku, parentheses)
- "Can you reach from A to B trying all paths?" (Word Search, maze)

---

## 2. The Backtracking Template (MEMORIZE!)

```java
void backtrack(current_state, choices, result) {
    // BASE CASE: Is the current state a valid solution?
    if (isSolution(current_state)) {
        result.add(new ArrayList<>(current_state));  // COPY! Not reference!
        return;
    }
    
    for (choice : choices) {
        // 1. CHOOSE — Make the decision
        current_state.add(choice);
        
        // 2. EXPLORE — Recurse
        backtrack(current_state, remaining_choices, result);
        
        // 3. UN-CHOOSE — Undo the decision (BACKTRACK!)
        current_state.remove(current_state.size() - 1);
    }
}
```

```
The 3 key questions for EVERY backtracking problem:

1. What is the CHOICE at each step?
   → Which element to include? Which cell to fill? Where to place the queen?

2. What is the CONSTRAINT?
   → No duplicates? Sum ≤ target? No two queens in same row/col/diagonal?

3. What is the GOAL (base case)?
   → All positions filled? Target sum reached? All elements used?
```

---

## 3. Subsets (LC 78)

**Problem:** Given array `nums` with unique elements, return ALL possible subsets.

`nums = [1, 2, 3]` → `[[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]`

### 🧠 How to Think
At each index, you have 2 choices: **INCLUDE** the element or **SKIP** it.
Use `startIndex` to avoid going backward (ensures no duplicates).

### 🧠 Dry Run (Decision Tree)

```
                         []
                     /        \
              [1]                []
            /     \           /     \
        [1,2]    [1]      [2]       []
        /  \     /  \     /  \     /  \
   [1,2,3] [1,2] [1,3] [1] [2,3] [2] [3] []

Leaves (all subsets): [], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]
```

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current));  // Every state is a valid subset!
    
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);                          // CHOOSE
        backtrack(nums, i + 1, current, result);       // EXPLORE (i+1: don't reuse)
        current.remove(current.size() - 1);            // UN-CHOOSE (backtrack)
    }
}
```

### 🧠 Execution Trace
```
backtrack(start=0, current=[])
  → add [] to result
  i=0: add 1 → current=[1]
    backtrack(start=1, current=[1])
      → add [1] to result
      i=1: add 2 → current=[1,2]
        backtrack(start=2, current=[1,2])
          → add [1,2] to result
          i=2: add 3 → current=[1,2,3]
            backtrack(start=3) → add [1,2,3] to result
          remove 3 → current=[1,2]
        remove 2 → current=[1]
      i=2: add 3 → current=[1,3]
        backtrack(start=3) → add [1,3] to result
      remove 3 → current=[1]
    remove 1 → current=[]
  i=1: add 2 → current=[2]
    ...adds [2], [2,3]
  i=2: add 3 → current=[3]
    ...adds [3]

Result: [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

---

## 4. Subsets II — With Duplicates (LC 90)

**Problem:** Same as Subsets, but `nums` may have **duplicates**. Return unique subsets.

`nums = [1, 2, 2]` → `[[], [1], [1,2], [1,2,2], [2], [2,2]]` (no duplicate subsets)

### 🧠 The Trick: Sort + Skip Duplicates
1. **Sort** the array first.
2. At same recursion level, if `nums[i] == nums[i-1]`, **SKIP** (duplicate at same position).

```java
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);  // CRUCIAL: Sort first!
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current));
    
    for (int i = start; i < nums.length; i++) {
        // SKIP DUPLICATES at same level
        if (i > start && nums[i] == nums[i - 1]) continue;
        
        current.add(nums[i]);
        backtrack(nums, i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

### 🧠 Why Does This Work?
```
nums = [1, 2, 2] (sorted)

At level where start=1:
  i=1: pick nums[1]=2 → explore [1,2,...] ✅
  i=2: nums[2]=2 == nums[1]=2 AND i > start → SKIP! ❌
  
This prevents [1, first_2] and [1, second_2] from both generating [1,2].
But [1,2,2] is still possible because the second 2 is picked at a DEEPER level.
```

---

## 5. Permutations (LC 46)

**Problem:** Given array with unique elements, return ALL permutations.

`nums = [1, 2, 3]` → `[[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]`

### 🧠 Key Difference from Subsets
- Subsets: use `startIndex` (order doesn't matter, [1,2] == [2,1])
- Permutations: use `boolean[] used` (order matters, [1,2] ≠ [2,1])

### 🧠 Dry Run (Decision Tree)

```
                            []
                    /        |        \
               [1]          [2]         [3]
              /    \       /    \      /    \
          [1,2]  [1,3]  [2,1] [2,3] [3,1] [3,2]
           |       |      |     |     |      |
        [1,2,3] [1,3,2] [2,1,3] [2,3,1] [3,1,2] [3,2,1]

6 permutations = 3! = 6 ✅
```

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> current, List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));  // Found a complete permutation!
        return;
    }
    
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;  // Skip already used elements
        
        used[i] = true;                                 // CHOOSE
        current.add(nums[i]);
        backtrack(nums, used, current, result);          // EXPLORE
        current.remove(current.size() - 1);              // UN-CHOOSE
        used[i] = false;
    }
}
```

---

## 6. Permutations II — With Duplicates (LC 47)

**Problem:** `nums` may have duplicates. Return unique permutations.

`nums = [1, 1, 2]` → `[[1,1,2], [1,2,1], [2,1,1]]` (only 3, not 6)

### 🧠 The Trick: Sort + Skip Rule
Sort the array. Skip a duplicate if the **previous same element was NOT used** at this level.

```java
public List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> current, List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        // Skip duplicate: same value as previous, and previous wasn't used
        if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
        
        used[i] = true;
        current.add(nums[i]);
        backtrack(nums, used, current, result);
        current.remove(current.size() - 1);
        used[i] = false;
    }
}
```

---

## 7. Combinations (LC 77)

**Problem:** Given `n` and `k`, return all combinations of `k` numbers from `[1, n]`.

`n = 4, k = 2` → `[[1,2], [1,3], [1,4], [2,3], [2,4], [3,4]]`

### 🧠 How to Think
Same as Subsets, but only add to result when `current.size() == k`.

```java
public List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(n, k, 1, new ArrayList<>(), result);
    return result;
}

private void backtrack(int n, int k, int start, List<Integer> current, List<List<Integer>> result) {
    if (current.size() == k) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    // Optimization: stop early if not enough elements remaining
    for (int i = start; i <= n - (k - current.size()) + 1; i++) {
        current.add(i);
        backtrack(n, k, i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

---

## 8. Combination Sum (LC 39, 40, 216)

### Combination Sum I (LC 39): Unlimited Use, Unique Candidates
Find all unique combinations that sum to `target`. Same number can be used **unlimited** times.

`candidates = [2,3,6,7], target = 7` → `[[2,2,3], [7]]`

```java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] candidates, int remain, int start, List<Integer> current, List<List<Integer>> result) {
    if (remain == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    if (remain < 0) return;  // Exceeded target
    
    for (int i = start; i < candidates.length; i++) {
        current.add(candidates[i]);
        backtrack(candidates, remain - candidates[i], i, current, result); // i, NOT i+1 (reuse!)
        current.remove(current.size() - 1);
    }
}
```

### Combination Sum II (LC 40): Each Number Used Once, Has Duplicates
`candidates = [10,1,2,7,6,1,5], target = 8` → `[[1,1,6], [1,2,5], [1,7], [2,6]]`

```java
// Sort + skip duplicates (same as Subsets II pattern!)
private void backtrack(int[] candidates, int remain, int start, List<Integer> current, List<List<Integer>> result) {
    if (remain == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = start; i < candidates.length; i++) {
        if (candidates[i] > remain) break;  // Pruning (array is sorted)
        if (i > start && candidates[i] == candidates[i-1]) continue;  // Skip duplicates
        
        current.add(candidates[i]);
        backtrack(candidates, remain - candidates[i], i + 1, current, result); // i+1 (each once)
        current.remove(current.size() - 1);
    }
}
```

### 🧠 Combination Sum Comparison

| Variant | Reuse? | Duplicates in input? | Start index | Skip rule |
|:---|:---:|:---:|:---:|:---|
| **I (LC 39)** | ✅ Unlimited | No | `i` (same) | None |
| **II (LC 40)** | ❌ Each once | Yes | `i + 1` | `i > start && nums[i]==nums[i-1]` |
| **III (LC 216)** | ❌ Each once | No (1-9) | `i + 1` | Fixed k numbers |

---

## 9. Letter Combinations of Phone Number (LC 17)

**Problem:** Given digit string `"23"`, return all letter combinations (phone keypad).

`"23"` → `["ad","ae","af","bd","be","bf","cd","ce","cf"]`

```java
private static final String[] MAPPING = {
    "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
};

public List<String> letterCombinations(String digits) {
    List<String> result = new ArrayList<>();
    if (digits.isEmpty()) return result;
    backtrack(digits, 0, new StringBuilder(), result);
    return result;
}

private void backtrack(String digits, int index, StringBuilder current, List<String> result) {
    if (index == digits.length()) {
        result.add(current.toString());
        return;
    }
    
    String letters = MAPPING[digits.charAt(index) - '0'];
    for (char c : letters.toCharArray()) {
        current.append(c);                                   // CHOOSE
        backtrack(digits, index + 1, current, result);       // EXPLORE
        current.deleteCharAt(current.length() - 1);          // UN-CHOOSE
    }
}
```

### 🧠 Dry Run
```
digits = "23"

backtrack(index=0, current="")
  digit '2' → letters = "abc"
  c='a': current="a"
    backtrack(index=1, current="a")
      digit '3' → letters = "def"
      c='d': current="ad" → ADD "ad" ✅
      c='e': current="ae" → ADD "ae" ✅
      c='f': current="af" → ADD "af" ✅
  c='b': current="b"
    backtrack(index=1, current="b")
      c='d': "bd" ✅, c='e': "be" ✅, c='f': "bf" ✅
  c='c': current="c"
    → "cd", "ce", "cf" ✅

Result: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

---

## 10. Palindrome Partitioning (LC 131)

**Problem:** Partition string `s` such that every substring is a palindrome.

`s = "aab"` → `[["a","a","b"], ["aa","b"]]`

### 🧠 How to Think
At each position, try all possible substrings starting from that position. If the substring is a palindrome, recurse on the remaining string.

```java
public List<List<String>> partition(String s) {
    List<List<String>> result = new ArrayList<>();
    backtrack(s, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(String s, int start, List<String> current, List<List<String>> result) {
    if (start == s.length()) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int end = start; end < s.length(); end++) {
        if (isPalindrome(s, start, end)) {
            current.add(s.substring(start, end + 1));         // CHOOSE
            backtrack(s, end + 1, current, result);            // EXPLORE
            current.remove(current.size() - 1);                // UN-CHOOSE
        }
    }
}

private boolean isPalindrome(String s, int left, int right) {
    while (left < right) {
        if (s.charAt(left++) != s.charAt(right--)) return false;
    }
    return true;
}
```

### 🧠 Dry Run
```
s = "aab"

backtrack(start=0, current=[])
  end=0: "a" palindrome? YES
    backtrack(start=1, current=["a"])
      end=1: "a" palindrome? YES
        backtrack(start=2, current=["a","a"])
          end=2: "b" palindrome? YES
            start==length → ADD ["a","a","b"] ✅
      end=2: "ab" palindrome? NO → skip
  end=1: "aa" palindrome? YES
    backtrack(start=2, current=["aa"])
      end=2: "b" palindrome? YES
        start==length → ADD ["aa","b"] ✅
  end=2: "aab" palindrome? NO → skip

Result: [["a","a","b"], ["aa","b"]]
```

---

## 11. N-Queens (LC 51)

**Problem:** Place N queens on an N×N chessboard so no two queens attack each other (no same row, column, or diagonal).

### 🧠 How to Think
Place queens row by row. For each row, try each column. Check 3 constraints:
1. No queen in same column → `boolean[] cols`
2. No queen in same diagonal (↘) → `boolean[] diag1` (row - col is constant)
3. No queen in same anti-diagonal (↙) → `boolean[] diag2` (row + col is constant)

```java
public List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    char[][] board = new char[n][n];
    for (char[] row : board) Arrays.fill(row, '.');
    
    boolean[] cols = new boolean[n];
    boolean[] diag1 = new boolean[2 * n];  // row - col + n (shift to avoid negative)
    boolean[] diag2 = new boolean[2 * n];  // row + col
    
    backtrack(board, 0, cols, diag1, diag2, result);
    return result;
}

private void backtrack(char[][] board, int row, boolean[] cols, 
                        boolean[] diag1, boolean[] diag2, List<List<String>> result) {
    int n = board.length;
    if (row == n) {
        // All queens placed! Convert board to strings
        List<String> solution = new ArrayList<>();
        for (char[] r : board) solution.add(new String(r));
        result.add(solution);
        return;
    }
    
    for (int col = 0; col < n; col++) {
        if (cols[col] || diag1[row - col + n] || diag2[row + col]) continue; // Conflict!
        
        // CHOOSE: place queen
        board[row][col] = 'Q';
        cols[col] = diag1[row - col + n] = diag2[row + col] = true;
        
        // EXPLORE: next row
        backtrack(board, row + 1, cols, diag1, diag2, result);
        
        // UN-CHOOSE: remove queen
        board[row][col] = '.';
        cols[col] = diag1[row - col + n] = diag2[row + col] = false;
    }
}
```

### 🧠 Dry Run (N=4)
```
Row 0: try col 0
  Row 1: col 0 ❌(col), col 1 ❌(diag), col 2 ✅
    Row 2: col 0 ❌(diag), col 1 ❌(col conflicts)... all fail!
  Backtrack row 1
  
Row 0: try col 1
  Row 1: col 0 ❌(diag), col 1 ❌(col), col 2 ❌(diag), col 3 ✅
    Row 2: col 0 ✅
      Row 3: col 0 ❌, col 1 ❌, col 2 ✅
        All placed! Solution:
        . Q . .
        . . . Q
        Q . . .
        . . Q .     ✅

(4-Queens has exactly 2 solutions)
```

---

## 12. Sudoku Solver (LC 37)

**Problem:** Fill a 9×9 Sudoku board. Each row, column, and 3×3 box must contain digits 1-9.

```java
public void solveSudoku(char[][] board) {
    solve(board);
}

private boolean solve(char[][] board) {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (board[i][j] != '.') continue; // Skip filled cells
            
            for (char c = '1'; c <= '9'; c++) {
                if (isValid(board, i, j, c)) {
                    board[i][j] = c;          // CHOOSE
                    
                    if (solve(board)) return true; // EXPLORE → if solved, done!
                    
                    board[i][j] = '.';        // UN-CHOOSE (backtrack)
                }
            }
            return false; // No valid digit for this cell → backtrack!
        }
    }
    return true; // All cells filled!
}

private boolean isValid(char[][] board, int row, int col, char c) {
    for (int i = 0; i < 9; i++) {
        if (board[row][i] == c) return false;  // Check row
        if (board[i][col] == c) return false;  // Check column
        
        // Check 3×3 box
        int boxRow = 3 * (row / 3) + i / 3;
        int boxCol = 3 * (col / 3) + i % 3;
        if (board[boxRow][boxCol] == c) return false;
    }
    return true;
}
```

---

## 13. Word Search (LC 79)

**Problem:** Given 2D board and a word, check if word exists by connecting adjacent cells (can't reuse).

```java
public boolean exist(char[][] board, String word) {
    int m = board.length, n = board[0].length;
    
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (dfs(board, word, i, j, 0)) return true;
        }
    }
    return false;
}

private boolean dfs(char[][] board, String word, int i, int j, int k) {
    if (k == word.length()) return true; // Found entire word!
    
    if (i < 0 || i >= board.length || j < 0 || j >= board[0].length 
        || board[i][j] != word.charAt(k)) return false;
    
    char temp = board[i][j];
    board[i][j] = '#';  // Mark visited (CHOOSE)
    
    boolean found = dfs(board, word, i+1, j, k+1)   // Down
                 || dfs(board, word, i-1, j, k+1)   // Up
                 || dfs(board, word, i, j+1, k+1)    // Right
                 || dfs(board, word, i, j-1, k+1);   // Left
    
    board[i][j] = temp;  // Restore (UN-CHOOSE / backtrack)
    
    return found;
}
```

---

## 14. Generate Parentheses (LC 22)

**Problem:** Generate all combinations of `n` pairs of well-formed parentheses.

`n = 3` → `["((()))","(()())","(())()","()(())","()()()"]`

### 🧠 How to Think
Two choices at each step: add `(` or add `)`.
Constraints: `open < n` (can add `(`), `close < open` (can add `)`).

```java
public List<String> generateParenthesis(int n) {
    List<String> result = new ArrayList<>();
    backtrack(n, 0, 0, new StringBuilder(), result);
    return result;
}

private void backtrack(int n, int open, int close, StringBuilder current, List<String> result) {
    if (current.length() == 2 * n) {
        result.add(current.toString());
        return;
    }
    
    if (open < n) {
        current.append('(');
        backtrack(n, open + 1, close, current, result);
        current.deleteCharAt(current.length() - 1);
    }
    
    if (close < open) {
        current.append(')');
        backtrack(n, open, close + 1, current, result);
        current.deleteCharAt(current.length() - 1);
    }
}
```

### 🧠 Dry Run (n=2)
```
backtrack(open=0, close=0, curr="")
  open < 2 → add '('
    backtrack(open=1, close=0, curr="(")
      open < 2 → add '('
        backtrack(open=2, close=0, curr="((")
          close < open → add ')'
            backtrack(open=2, close=1, curr="(()")
              close < open → add ')'
                backtrack(open=2, close=2, curr="(())") → ADD "(())" ✅
      close < open → add ')'
        backtrack(open=1, close=1, curr="()")
          open < 2 → add '('
            backtrack(open=2, close=1, curr="()(")
              close < open → add ')'
                backtrack(open=2, close=2, curr="()()") → ADD "()()" ✅

Result: ["(())", "()()"]
```

---

## 15. Backtracking vs DP vs Greedy

| Feature | Backtracking | Dynamic Programming | Greedy |
|:---|:---|:---|:---|
| **Goal** | Find ALL solutions | Find OPTIMAL solution | Find OPTIMAL solution |
| **Approach** | Try everything, undo bad choices | Solve subproblems, store results | Make locally optimal choice |
| **Time** | Exponential O(2ⁿ) or O(n!) | Polynomial O(n²) or O(n×W) | Often O(n log n) or O(n) |
| **When** | "Generate all", "find all valid" | "Count ways", "min/max cost" | "Always take best local option" |
| **Examples** | N-Queens, Permutations, Sudoku | Knapsack, LCS, Coin Change | Activity Selection, Huffman |

```
Decision Flow:
  "Find ALL solutions?"           → Backtracking
  "Find OPTIMAL + overlapping?"   → DP
  "Optimal + greedy choice works?" → Greedy
```

---

## 16. Pattern Recognition

```
┌──────────────────────────────────────────────────────────────┐
│           BACKTRACKING PATTERN RECOGNITION                    │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  "All subsets / power set"        → Subsets pattern          │
│  "All permutations / arrangements"→ Permutations pattern     │
│  "All combinations of size k"     → Combinations pattern    │
│  "Sum to target with candidates"  → Combination Sum         │
│  "Place items with constraints"   → N-Queens / Sudoku       │
│  "Find word in grid"              → Grid backtracking       │
│  "Generate valid strings"         → Parentheses pattern     │
│  "Partition string"               → Palindrome Partition    │
│                                                               │
│  DUPLICATE HANDLING:                                          │
│  • Sort the array first                                       │
│  • Skip: if (i > start && nums[i] == nums[i-1]) continue    │
│                                                               │
│  REUSE vs NO-REUSE:                                           │
│  • Can reuse:   recurse with i     (Combination Sum I)       │
│  • Cannot reuse: recurse with i+1  (Combination Sum II)      │
│                                                               │
│  ORDER MATTERS vs NOT:                                        │
│  • Order matters (permutations): use boolean[] used          │
│  • Order doesn't matter (subsets): use startIndex            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 17. Top 15 Must-Do Backtracking Questions

**Subsets/Combinations:**
1. Subsets (LC 78)
2. Subsets II (LC 90)
3. Combinations (LC 77)
4. Combination Sum I (LC 39)
5. Combination Sum II (LC 40)

**Permutations:**
6. Permutations (LC 46)
7. Permutations II (LC 47)

**String Backtracking:**
8. Letter Combinations of Phone Number (LC 17)
9. Palindrome Partitioning (LC 131)
10. Generate Parentheses (LC 22)

**Grid Backtracking:**
11. Word Search (LC 79)
12. N-Queens (LC 51)
13. Sudoku Solver (LC 37)

**Advanced:**
14. Restore IP Addresses (LC 93)
15. Expression Add Operators (LC 282)
