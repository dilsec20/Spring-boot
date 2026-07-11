# 🧠 Dynamic Programming (DP) — FAANG Masterclass

> **"Those who cannot remember the past are condemned to repeat it."** — George Santayana (and Dynamic Programming).

Dynamic Programming is simply **Recursion + Memorization**. If you are calculating the same sub-problem twice, you are doing it wrong. Store the result, and reuse it.

---

## 📑 Table of Contents
1. [How to Identify a DP Problem? (The Pattern)](#1-how-to-identify-a-dp-problem)
2. [Top-Down (Memoization) vs Bottom-Up (Tabulation)](#2-top-down-vs-bottom-up)
3. [The 1D DP: Climbing Stairs](#3-the-1d-dp-climbing-stairs)
4. [House Robber (I & II)](#4-house-robber-i--ii)
5. [Coin Change (Min Coins)](#5-coin-change-min-coins)
6. [Decode Ways](#6-decode-ways)
7. [The 2D DP: 0/1 Knapsack (The Mother of All DP)](#7-the-2d-dp-01-knapsack)
8. [Partition Equal Subset Sum](#8-partition-equal-subset-sum)
9. [Target Sum](#9-target-sum)
10. [String DP: Longest Common Subsequence (LCS)](#10-string-dp-longest-common-subsequence)
11. [Edit Distance (Levenshtein)](#11-edit-distance)
12. [Longest Palindromic Subsequence](#12-longest-palindromic-subsequence)
13. [Word Break](#13-word-break)
14. [DP on Grids: Unique Paths & Min Path Sum](#14-dp-on-grids-unique-paths--min-path-sum)
15. [Longest Increasing Subsequence (LIS)](#15-longest-increasing-subsequence)
16. [Best Time to Buy & Sell Stock (All Variations)](#16-best-time-to-buy--sell-stock)
17. [Interval DP: Burst Balloons (MCM Pattern)](#17-interval-dp-burst-balloons)
18. [DP Patterns Cheat Sheet](#18-dp-patterns-cheat-sheet)
19. [Top 25 Must-Do DP Questions](#19-top-25-must-do-dp-questions)

---

## 1. How to Identify a DP Problem?

Stop guessing. Look for these **two specific triggers** in the interview question:

1.  **"Maximize", "Minimize", "Longest", "Shortest", "Total number of ways"**: The problem is asking for an optimal solution or counting total combinations.
2.  **Choices**: At every step, you have to make a choice (e.g., "Include this item or exclude it", "Take 1 step or 2 steps").

**The Ultimate DP Blueprint (MEMORIZE THIS!):**
1. Express the problem in terms of indexes (e.g., `f(i, j)`)
2. Explore all choices at that index
3. Return the `max/min/sum` of those choices
4. Base case (What happens when the array ends/capacity becomes 0?)

```
HOW TO THINK about any DP problem:

Step 1: Can I write a recursive brute-force solution?
        → YES → Go to Step 2

Step 2: Are there OVERLAPPING SUBPROBLEMS?
        (Same function called with same arguments multiple times?)
        → YES → This is a DP problem!

Step 3: Write the recursion FIRST.
Step 4: Add memoization (2 lines of code).
Step 5: Convert to tabulation (bottom-up loops).
Step 6: Optimize space if possible.
```

---

## 2. Top-Down vs Bottom-Up

| Feature | Top-Down (Memoization) | Bottom-Up (Tabulation) | Space Optimization |
| :--- | :--- | :--- | :--- |
| **Approach** | Start from $N$, go down to $0$. | Start from $0$, build up to $N$. | Start from $0$, but only keep previous 1-2 states. |
| **Code Style** | Recursive. Easy to write. | Iterative (Loops). Harder to think. | Iterative. Hardest to think. |
| **Time Complexity** | $O(N)$ | $O(N)$ | $O(N)$ |
| **Space Complexity** | $O(N)$ + $O(N)$ Stack Space | $O(N)$ (Array) | $O(1)$ (Variables) |
| **Risk** | Stack Overflow for huge $N$. | Safe. | Extremely Safe & Fast. |

*Golden Rule:* ALWAYS write the recursive solution first. Once recursion works, adding a memoization table takes exactly 2 lines of code.

---

## 3. The 1D DP: Climbing Stairs

**Problem (LC 70):** You are climbing a staircase. It takes `n` steps to reach the top. Each time you can either climb `1` or `2` steps. In how many distinct ways can you climb to the top?

### 🧠 How to Think
At step `n`, you could have come from `n-1` (took a 1-step) OR `n-2` (took a 2-step).
`f(n) = f(n-1) + f(n-2)` → *(Wait, this is the Fibonacci sequence!)*

### 🧠 What is stored in the DP Table?
*   `dp[i]` = The total number of distinct ways to reach step `i`.

### 🧠 Dry Run (n = 5)

| Step `i` | dp[i] | How? |
|:---:|:---:|:---|
| 0 | 1 | Base: 1 way to be at ground (do nothing) |
| 1 | 1 | Only: 1 step |
| 2 | 2 | dp[1] + dp[0] = 1 + 1 = 2 → (1+1) or (2) |
| 3 | 3 | dp[2] + dp[1] = 2 + 1 = 3 → (1+1+1), (1+2), (2+1) |
| 4 | 5 | dp[3] + dp[2] = 3 + 2 = 5 |
| 5 | **8** | dp[4] + dp[3] = 5 + 3 = **8** |

### 🧠 Tabulation (Bottom-Up) Code

```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    
    int[] dp = new int[n + 1]; // Step 0 to n
    
    // Base Cases
    dp[1] = 1; // 1 way to reach step 1 (1)
    dp[2] = 2; // 2 ways to reach step 2 (1+1, 2)
    
    // Build from bottom to top
    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2]; 
    }
    
    return dp[n];
}
```

### 🧠 Space Optimization (Expert Level)
Since `dp[i]` only relies on `dp[i-1]` and `dp[i-2]`, we don't need the whole array!

```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1;
    int prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1; // O(1) Space!
}
```

### 🧠 Variation: Min Cost Climbing Stairs (LC 746)
Each step has a cost. Find minimum cost to reach the top.
```java
// dp[i] = min cost to reach step i
// dp[i] = cost[i] + min(dp[i-1], dp[i-2])
public int minCostClimbingStairs(int[] cost) {
    int n = cost.length;
    int[] dp = new int[n];
    dp[0] = cost[0];
    dp[1] = cost[1];
    for (int i = 2; i < n; i++) {
        dp[i] = cost[i] + Math.min(dp[i-1], dp[i-2]);
    }
    return Math.min(dp[n-1], dp[n-2]); // Can start from last or second-last
}
```

---

## 4. House Robber (I & II)

### House Robber I (LC 198)
**Problem:** You are a robber. Houses are in a row. Each has money. You CANNOT rob two adjacent houses (alarm goes off). Maximize total money.

### 🧠 How to Think
At house `i`, you have 2 choices:
1. **Rob it:** Take `nums[i]` + best from `dp[i-2]` (skip adjacent)
2. **Skip it:** Take `dp[i-1]` (best without this house)

`dp[i] = max(nums[i] + dp[i-2], dp[i-1])`

### 🧠 Dry Run

`nums = [2, 7, 9, 3, 1]`

| i | nums[i] | Rob it: nums[i]+dp[i-2] | Skip it: dp[i-1] | dp[i] = max |
|:---:|:---:|:---:|:---:|:---:|
| 0 | 2 | 2 (base) | — | **2** |
| 1 | 7 | 7 | 2 | **7** |
| 2 | 9 | 9 + dp[0] = 9+2 = 11 | dp[1] = 7 | **11** |
| 3 | 3 | 3 + dp[1] = 3+7 = 10 | dp[2] = 11 | **11** |
| 4 | 1 | 1 + dp[2] = 1+11 = 12 | dp[3] = 11 | **12** |

**Answer: 12** (Rob houses 0, 2, 4 → 2 + 9 + 1 = 12)

```java
public int rob(int[] nums) {
    int n = nums.length;
    if (n == 1) return nums[0];
    
    int[] dp = new int[n];
    dp[0] = nums[0];
    dp[1] = Math.max(nums[0], nums[1]);
    
    for (int i = 2; i < n; i++) {
        dp[i] = Math.max(nums[i] + dp[i-2], dp[i-1]);
    }
    return dp[n-1];
}

// Space Optimized O(1):
public int rob(int[] nums) {
    int prev2 = 0, prev1 = 0;
    for (int num : nums) {
        int curr = Math.max(num + prev2, prev1);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

### House Robber II (LC 213)
**Problem:** Same as above, but houses are in a **CIRCLE** (first and last house are adjacent).

### 🧠 How to Think
If we rob house 0, we CANNOT rob house n-1.
If we rob house n-1, we CANNOT rob house 0.

**Trick:** Run House Robber I twice:
1. On `nums[0...n-2]` (exclude last house)
2. On `nums[1...n-1]` (exclude first house)
Answer = `max(result1, result2)`

```java
public int rob(int[] nums) {
    int n = nums.length;
    if (n == 1) return nums[0];
    // Run on [0, n-2] and [1, n-1], take max
    return Math.max(
        robLinear(nums, 0, n - 2),
        robLinear(nums, 1, n - 1)
    );
}

private int robLinear(int[] nums, int start, int end) {
    int prev2 = 0, prev1 = 0;
    for (int i = start; i <= end; i++) {
        int curr = Math.max(nums[i] + prev2, prev1);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

---

## 5. Coin Change (Min Coins)

**Problem (LC 322):** Given coins `[1, 3, 4]` and amount `6`. Find the **minimum number of coins** to make the amount. (Unbounded Knapsack — can use same coin infinite times)

### 🧠 How to Think
`dp[i]` = minimum coins needed to make amount `i`.
For each amount, try every coin: `dp[i] = min(dp[i], dp[i - coin] + 1)`

### 🧠 Dry Run

`coins = [1, 3, 4]`, `amount = 6`

| Amount | Try coin=1 | Try coin=3 | Try coin=4 | dp[amt] | Coins used |
|:---:|:---:|:---:|:---:|:---:|:---|
| 0 | — | — | — | **0** | none |
| 1 | dp[0]+1 = 1 | ✗ (1<3) | ✗ (1<4) | **1** | [1] |
| 2 | dp[1]+1 = 2 | ✗ | ✗ | **2** | [1,1] |
| 3 | dp[2]+1 = 3 | dp[0]+1 = 1 | ✗ | **1** | [3] |
| 4 | dp[3]+1 = 2 | dp[1]+1 = 2 | dp[0]+1 = 1 | **1** | [4] |
| 5 | dp[4]+1 = 2 | dp[2]+1 = 3 | dp[1]+1 = 2 | **2** | [1,4] |
| 6 | dp[5]+1 = 3 | dp[3]+1 = 2 | dp[2]+1 = 3 | **2** | [3,3] |

**Answer: 2** (use two 3-coins: 3+3=6)

```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);  // Fill with impossible large value
    dp[0] = 0;                     // 0 coins needed for amount 0
    
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount];
}
```

### Variation: Coin Change II — Count Ways (LC 518)
**Problem:** Count the **number of combinations** to make the amount.

```java
// CAREFUL: Loop coins OUTER, amount INNER → avoids counting permutations
public int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;  // 1 way to make amount 0: use nothing
    
    for (int coin : coins) {           // Outer loop: each coin
        for (int i = coin; i <= amount; i++) {  // Inner loop: amounts
            dp[i] += dp[i - coin];
        }
    }
    return dp[amount];
}

// WHY coins outer?
// If amount outer: [1,3] and [3,1] counted as different (permutations)
// If coins outer:  [1,3] counted only once (combinations)
```

---

## 6. Decode Ways

**Problem (LC 91):** A message containing letters 'A'-'Z' is encoded: 'A'→1, 'B'→2, ..., 'Z'→26. Given an encoded string `s`, return the number of ways to decode it.

Example: `"226"` → "BZ" (2,26), "VF" (22,6), "BBF" (2,2,6) → **3 ways**

### 🧠 How to Think
At index `i`:
- If `s[i]` is '1'-'9', it forms a valid single digit → add `dp[i-1]`
- If `s[i-1..i]` forms 10-26, it forms a valid two-digit → add `dp[i-2]`

### 🧠 Dry Run

`s = "226"`

| i | s[i] | Single digit valid? | Two digit (s[i-1..i]) valid? | dp[i] |
|:---:|:---:|:---:|:---:|:---:|
| 0 | '2' | Yes → base case | — | **1** |
| 1 | '2' | Yes ('2') → +dp[0]=1 | "22" → valid (10-26) → +dp[-1]=1 | **2** |
| 2 | '6' | Yes ('6') → +dp[1]=2 | "26" → valid (10-26) → +dp[0]=1 | **3** |

**Answer: 3**

```java
public int numDecodings(String s) {
    int n = s.length();
    if (s.charAt(0) == '0') return 0;
    
    int[] dp = new int[n + 1];
    dp[0] = 1;  // Empty string: 1 way
    dp[1] = 1;  // First char (already checked not '0')
    
    for (int i = 2; i <= n; i++) {
        int oneDigit = Integer.parseInt(s.substring(i-1, i));   // s[i-1]
        int twoDigit = Integer.parseInt(s.substring(i-2, i));   // s[i-2..i-1]
        
        if (oneDigit >= 1 && oneDigit <= 9) {
            dp[i] += dp[i-1];
        }
        if (twoDigit >= 10 && twoDigit <= 26) {
            dp[i] += dp[i-2];
        }
    }
    return dp[n];
}
```

---

## 7. The 2D DP: 0/1 Knapsack (The Mother of all DP)

**Problem:** You are a thief with a bag of capacity `W`. You have `N` items, each with a `weight` and a `value`. Maximize the total value you can steal without exceeding `W`. (You can only take an item once → 0/1).

### 🧠 How to Think
Since there are TWO changing variables (which item we are at `i`, and the remaining capacity `w`), we need a 2D table.
*   `dp[i][w]` = The maximum value achievable using the first `i` items, with a knapsack capacity of `w`.

### 🧠 The Recursive Choice
For every item, you have 2 choices:
1. **Take it:** `value[i-1] + dp[i-1][w - weight[i-1]]` (Only if `weight <= w`)
2. **Leave it:** `0 + dp[i-1][w]`
*Take the `Math.max` of both.*

### 🧠 Dry Run & DP Table Visualization

Items: `val = [60, 100, 120]`, `wt = [10, 20, 30]`, Bag Capacity `W = 50`.

**Full DP Table (every cell filled step by step):**

| `i` / `w` | 0 | 10 | 20 | 30 | 40 | 50 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **0** (No items) | 0 | 0 | 0 | 0 | 0 | 0 |
| **1** (wt=10, val=60) | 0 | **60** | 60 | 60 | 60 | 60 |
| **2** (wt=20, val=100) | 0 | 60 | **100** | 160 | 160 | 160 |
| **3** (wt=30, val=120) | 0 | 60 | 100 | 120 | 180 | **220** |

**How dp[3][50] = 220 was computed:**
```
Item 3: wt=30, val=120, capacity w=50
  Take it:  val=120 + dp[2][50-30] = 120 + dp[2][20] = 120 + 100 = 220
  Leave it: dp[2][50] = 160
  dp[3][50] = max(220, 160) = 220 ✅

Backtrack: Items 2 (val=100) + Item 3 (val=120) = 220, weight = 20+30 = 50
```

### 🧠 The Code
```java
public int knapsack(int[] wt, int[] val, int W, int n) {
    int[][] dp = new int[n + 1][W + 1];

    for (int i = 1; i <= n; i++) {
        for (int w = 1; w <= W; w++) {
            if (wt[i - 1] <= w) {
                // Max of (Taking Item, Leaving Item)
                dp[i][w] = Math.max(
                    val[i - 1] + dp[i - 1][w - wt[i - 1]], 
                    dp[i - 1][w]
                );
            } else {
                // Cannot take item, too heavy
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    return dp[n][W]; // Bottom-right corner is the answer
}
```

### 🧠 Space Optimization (1D Array)
Since each row only depends on the previous row, we can use a single 1D array. **Trick:** Iterate `w` from RIGHT to LEFT to avoid overwriting values we still need.

```java
public int knapsack(int[] wt, int[] val, int W, int n) {
    int[] dp = new int[W + 1];
    
    for (int i = 0; i < n; i++) {
        for (int w = W; w >= wt[i]; w--) {  // RIGHT to LEFT!
            dp[w] = Math.max(dp[w], val[i] + dp[w - wt[i]]);
        }
    }
    return dp[W];
}
// Why right-to-left? 
// Left-to-right would use UPDATED dp[w-wt[i]] from current row = UNBOUNDED knapsack
// Right-to-left uses dp[w-wt[i]] from previous row = 0/1 knapsack
```

---

## 8. Partition Equal Subset Sum

**Problem (LC 416):** Given array `nums`, can you partition it into two subsets such that the sum of both subsets is equal?

### 🧠 How to Think
If total sum is odd → impossible.
If even → find a subset with sum = `totalSum / 2` → **This is 0/1 Knapsack!**

`dp[j]` = Can we make sum `j` using elements seen so far?

### 🧠 Dry Run

`nums = [1, 5, 11, 5]`, totalSum = 22, target = 11

| After processing | dp[0] | dp[1] | dp[5] | dp[6] | dp[10] | dp[11] |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| Init | T | F | F | F | F | F |
| nums[0]=1 | T | **T** | F | F | F | F |
| nums[1]=5 | T | T | **T** | **T** | F | F |
| nums[2]=11 | T | T | T | T | F | **T** |
| nums[3]=5 | T | T | T | T | **T** | **T** ✅ |

**dp[11] = true → Answer: YES, can partition!** (subset {1,5,5} and {11})

```java
public boolean canPartition(int[] nums) {
    int sum = 0;
    for (int num : nums) sum += num;
    if (sum % 2 != 0) return false;  // Odd sum → impossible
    
    int target = sum / 2;
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;  // Sum 0 is always possible (empty subset)
    
    for (int num : nums) {
        for (int j = target; j >= num; j--) {  // Right to left (0/1 knapsack)
            dp[j] = dp[j] || dp[j - num];
        }
    }
    return dp[target];
}
```

---

## 9. Target Sum

**Problem (LC 494):** Given array `nums` and target. Assign `+` or `-` to each element to make total equal to target. Count the number of ways.

Example: `nums = [1,1,1,1,1]`, target = 3 → 5 ways

### 🧠 How to Think
Let P = sum of elements with `+`, N = sum of elements with `-`.
- P + N = totalSum
- P - N = target
- **P = (totalSum + target) / 2**

This reduces to: **Count subsets with sum = P** → 0/1 Knapsack counting!

```java
public int findTargetSumWays(int[] nums, int target) {
    int sum = 0;
    for (int num : nums) sum += num;
    
    // Edge cases
    if ((sum + target) % 2 != 0 || Math.abs(target) > sum) return 0;
    
    int subsetSum = (sum + target) / 2;
    int[] dp = new int[subsetSum + 1];
    dp[0] = 1;  // 1 way to make sum 0
    
    for (int num : nums) {
        for (int j = subsetSum; j >= num; j--) {
            dp[j] += dp[j - num];
        }
    }
    return dp[subsetSum];
}
```

---

## 10. String DP: Longest Common Subsequence (LCS)

**Problem (LC 1143):** Given two strings `text1` and `text2`, return the length of their longest common subsequence. 
Example: "abcde" and "ace" → "ace" (Length 3).

### 🧠 How to Think
*   `dp[i][j]` = Length of LCS of `text1[0...i-1]` and `text2[0...j-1]`.

### 🧠 The Recursive Choice
If `text1[i-1] == text2[j-1]`:
   * Match found! `1 + dp[i-1][j-1]` (diagonal)
Else:
   * No match: `max(dp[i-1][j], dp[i][j-1])` (max of top or left)

### 🧠 Dry Run

`text1 = "abcde"`, `text2 = "ace"`

| | "" | a | c | e |
|:---:|:---:|:---:|:---:|:---:|
| **""** | 0 | 0 | 0 | 0 |
| **a** | 0 | **1**↖ | 1 | 1 |
| **b** | 0 | 1 | 1 | 1 |
| **c** | 0 | 1 | **2**↖ | 2 |
| **d** | 0 | 1 | 2 | 2 |
| **e** | 0 | 1 | 2 | **3**↖ |

↖ = came from diagonal (match found!)

**How dp[5][3] = 3:**
```
text1[4]='e' == text2[2]='e' → MATCH! → 1 + dp[4][2] = 1 + 2 = 3
LCS = "ace"
```

### 🧠 The Code
```java
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length();
    int n = text2.length();
    int[][] dp = new int[m + 1][n + 1]; // Auto-initialized to 0
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = 1 + dp[i - 1][j - 1]; // Diagonal + 1
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]); // Max of Top or Left
            }
        }
    }
    return dp[m][n];
}
```

### 🧠 How to PRINT the actual LCS
```java
// After building dp table, backtrack from dp[m][n]:
public String printLCS(String text1, String text2, int[][] dp) {
    StringBuilder sb = new StringBuilder();
    int i = text1.length(), j = text2.length();
    
    while (i > 0 && j > 0) {
        if (text1.charAt(i-1) == text2.charAt(j-1)) {
            sb.append(text1.charAt(i-1));  // This char is part of LCS
            i--; j--;                       // Move diagonally
        } else if (dp[i-1][j] > dp[i][j-1]) {
            i--;   // Move up
        } else {
            j--;   // Move left
        }
    }
    return sb.reverse().toString();
}
```

### LCS Variations (FAANG Favorites!)

| Problem | Reduction to LCS |
|:---|:---|
| Longest Common Substring | Same DP, but reset to 0 when chars don't match |
| Shortest Common Supersequence | `m + n - LCS(text1, text2)` |
| Min Insertions to Make Palindrome | `n - LPS(s)` where LPS = LCS(s, reverse(s)) |
| Min Deletions to Make Strings Equal | `m + n - 2 * LCS(text1, text2)` |

---

## 11. Edit Distance

**Problem (LC 72):** Given two strings `word1` and `word2`, return the minimum number of operations (insert, delete, replace) to convert `word1` to `word2`.

### 🧠 How to Think
`dp[i][j]` = min operations to convert `word1[0..i-1]` to `word2[0..j-1]`

3 choices when chars don't match:
- **Insert:** `dp[i][j-1] + 1` (insert char from word2)
- **Delete:** `dp[i-1][j] + 1` (delete char from word1)
- **Replace:** `dp[i-1][j-1] + 1` (replace char)

### 🧠 Dry Run

`word1 = "horse"`, `word2 = "ros"`

| | "" | r | o | s |
|:---:|:---:|:---:|:---:|:---:|
| **""** | 0 | 1 | 2 | 3 |
| **h** | 1 | 1(replace h→r) | 2 | 3 |
| **o** | 2 | 2 | 1(match!) | 2 |
| **r** | 3 | 2(match!) | 2 | 2 |
| **s** | 4 | 3 | 3 | 2(match!) |
| **e** | 5 | 4 | 4 | **3** |

**How dp[5][3] = 3:**
```
word1="horse" → word2="ros"
Step 1: Replace 'h' with 'r' → "rorse"
Step 2: Delete 'r'          → "rose"
Step 3: Delete 'e'          → "ros" ✅
```

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    // Base cases: converting empty string
    for (int i = 0; i <= m; i++) dp[i][0] = i;  // Delete all chars
    for (int j = 0; j <= n; j++) dp[0][j] = j;  // Insert all chars
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i-1) == word2.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1];  // Match! No operation needed
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i-1][j-1],   // Replace
                    Math.min(
                        dp[i-1][j],  // Delete
                        dp[i][j-1]   // Insert
                    )
                );
            }
        }
    }
    return dp[m][n];
}
```

---

## 12. Longest Palindromic Subsequence

**Problem (LC 516):** Given string `s`, find the length of the longest palindromic subsequence.

### 🧠 The Trick (Reduce to LCS!)
The LPS of a string `s` = LCS of `s` and `reverse(s)`.

Example: `s = "bbbab"`, `reverse(s) = "babbb"`
LCS("bbbab", "babbb") = "bbbb" → Length **4** ✅

```java
public int longestPalindromeSubseq(String s) {
    String reversed = new StringBuilder(s).reverse().toString();
    return longestCommonSubsequence(s, reversed);  // Reuse LCS!
}
```

### Variation: Min Insertions to Make Palindrome (LC 1312)
```java
// Answer = s.length() - LPS(s)
// We need to insert characters to "fill the gaps" in the palindrome
public int minInsertions(String s) {
    return s.length() - longestPalindromeSubseq(s);
}
```

---

## 13. Word Break

**Problem (LC 139):** Given string `s` and dictionary `wordDict`, can `s` be segmented into space-separated dictionary words?

Example: `s = "leetcode"`, `wordDict = ["leet", "code"]` → true ("leet code")

### 🧠 How to Think
`dp[i]` = Can `s[0..i-1]` be broken into dictionary words?
For each position `i`, check all possible last words ending at `i`.

### 🧠 Dry Run

`s = "leetcode"`, dict = {"leet", "code"}

| i | Substring check | dp[i] |
|:---:|:---|:---:|
| 0 | "" (empty) | **T** (base) |
| 1 | "l" → not in dict | **F** |
| 2 | "le" → not in dict | **F** |
| 3 | "lee" → not in dict | **F** |
| 4 | "leet" → in dict AND dp[0]=T ✅ | **T** |
| 5 | "c","ec","etc" → none in dict with dp[j]=T | **F** |
| 6 | "co","ecode" → no | **F** |
| 7 | "cod" → no | **F** |
| 8 | "code" → in dict AND dp[4]=T ✅ | **T** ✅ |

```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    int n = s.length();
    boolean[] dp = new boolean[n + 1];
    dp[0] = true;  // Empty string can be segmented
    
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            // If s[0..j-1] can be broken AND s[j..i-1] is a word
            if (dp[j] && dict.contains(s.substring(j, i))) {
                dp[i] = true;
                break;  // No need to check further
            }
        }
    }
    return dp[n];
}
```

---

## 14. DP on Grids: Unique Paths & Min Path Sum

### Unique Paths (LC 62)

**Problem:** Robot at top-left of `m x n` grid. Can only move DOWN or RIGHT. Count unique paths to bottom-right.

### 🧠 Dry Run (3×3 grid)

| | col 0 | col 1 | col 2 |
|:---:|:---:|:---:|:---:|
| **row 0** | 1 | 1 | 1 |
| **row 1** | 1 | 2 | 3 |
| **row 2** | 1 | 3 | **6** |

`dp[i][j] = dp[i-1][j] + dp[i][j-1]` (top + left)

```java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    
    // Base Case: Top row and Left column = 1 (only one straight path)
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;
    
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1]; // Top + Left
        }
    }
    
    return dp[m - 1][n - 1];
}
```

### Unique Paths II (LC 63) — With Obstacles
```java
// Same logic, but if grid[i][j] == 1 (obstacle), dp[i][j] = 0
public int uniquePathsWithObstacles(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    if (grid[0][0] == 1) return 0;
    
    int[][] dp = new int[m][n];
    dp[0][0] = 1;
    
    for (int i = 1; i < m; i++) dp[i][0] = (grid[i][0] == 1) ? 0 : dp[i-1][0];
    for (int j = 1; j < n; j++) dp[0][j] = (grid[0][j] == 1) ? 0 : dp[0][j-1];
    
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = (grid[i][j] == 1) ? 0 : dp[i-1][j] + dp[i][j-1];
        }
    }
    return dp[m-1][n-1];
}
```

### Minimum Path Sum (LC 64)

**Problem:** Given `m x n` grid with non-negative numbers, find path from top-left to bottom-right minimizing the sum.

### 🧠 Dry Run

```
grid = [[1, 3, 1],
        [1, 5, 1],
        [4, 2, 1]]
```

| | col 0 | col 1 | col 2 |
|:---:|:---:|:---:|:---:|
| **row 0** | 1 | 1+3=4 | 4+1=5 |
| **row 1** | 1+1=2 | min(4,2)+5=7 | min(5,7)+1=6 |
| **row 2** | 2+4=6 | min(7,6)+2=8 | min(6,8)+1=**7** |

**Answer: 7** (path: 1→3→1→1→1)

```java
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    dp[0][0] = grid[0][0];
    
    for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];
    
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
        }
    }
    return dp[m-1][n-1];
}
```

---

## 15. Longest Increasing Subsequence (LIS)

**Problem (LC 300):** Given integer array `nums`, return the length of the longest strictly increasing subsequence.

Example: `nums = [10, 9, 2, 5, 3, 7, 101, 18]` → LIS = [2, 3, 7, 101] → Length **4**

### Approach 1: O(n²) DP

`dp[i]` = Length of LIS ending at index `i`.
For each `i`, check all `j < i`: if `nums[j] < nums[i]`, then `dp[i] = max(dp[i], dp[j] + 1)`.

### 🧠 Dry Run

`nums = [10, 9, 2, 5, 3, 7, 101, 18]`

| i | nums[i] | Check all j < i where nums[j] < nums[i] | dp[i] | LIS ending here |
|:---:|:---:|:---|:---:|:---|
| 0 | 10 | — | **1** | [10] |
| 1 | 9 | no j satisfies | **1** | [9] |
| 2 | 2 | no j satisfies | **1** | [2] |
| 3 | 5 | j=2: nums[2]=2 < 5 → dp[2]+1=2 | **2** | [2,5] |
| 4 | 3 | j=2: nums[2]=2 < 3 → dp[2]+1=2 | **2** | [2,3] |
| 5 | 7 | j=2: 2<7→2, j=3: 5<7→3, j=4: 3<7→3 | **3** | [2,3,7] or [2,5,7] |
| 6 | 101 | j=5: 7<101→4 (best) | **4** | [2,3,7,101] |
| 7 | 18 | j=5: 7<18→4 (best) | **4** | [2,3,7,18] |

**Answer: max(dp) = 4**

```java
// O(n²) solution
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);  // Every element is a LIS of length 1
    int maxLen = 1;
    
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.max(maxLen, dp[i]);
    }
    return maxLen;
}
```

### Approach 2: O(n log n) — Binary Search (Patience Sorting)

Maintain a `tails[]` array where `tails[i]` = smallest tail element of all increasing subsequences of length `i+1`.

```java
// O(n log n) — OPTIMAL
public int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();
    
    for (int num : nums) {
        int pos = Collections.binarySearch(tails, num);
        if (pos < 0) pos = -(pos + 1);  // Insertion point
        
        if (pos == tails.size()) {
            tails.add(num);       // Extend the longest LIS
        } else {
            tails.set(pos, num);  // Replace to keep smallest possible tail
        }
    }
    return tails.size();
}
```

### 🧠 Dry Run of O(n log n)

`nums = [10, 9, 2, 5, 3, 7, 101, 18]`

| Step | num | tails[] before | Action | tails[] after |
|:---:|:---:|:---|:---|:---|
| 1 | 10 | [] | Append | [10] |
| 2 | 9 | [10] | Replace at pos 0 (9<10) | [9] |
| 3 | 2 | [9] | Replace at pos 0 (2<9) | [2] |
| 4 | 5 | [2] | Append (5>2) | [2, 5] |
| 5 | 3 | [2, 5] | Replace at pos 1 (3<5) | [2, 3] |
| 6 | 7 | [2, 3] | Append (7>3) | [2, 3, 7] |
| 7 | 101 | [2, 3, 7] | Append (101>7) | [2, 3, 7, 101] |
| 8 | 18 | [2, 3, 7, 101] | Replace at pos 3 (18<101) | [2, 3, 7, 18] |

**tails.size() = 4** ✅

---

## 16. Best Time to Buy & Sell Stock

### Variation I (LC 121): One Transaction Only
Just track the minimum price so far and max profit.

```java
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

### Variation II (LC 122): Unlimited Transactions
Buy and sell as many times as you want.

```java
// Greedy: take every uphill
public int maxProfit(int[] prices) {
    int profit = 0;
    for (int i = 1; i < prices.length; i++) {
        if (prices[i] > prices[i-1]) {
            profit += prices[i] - prices[i-1];
        }
    }
    return profit;
}
```

### Variation III (LC 123): At Most 2 Transactions — State Machine DP

### 🧠 How to Think (State Machine)
Maintain states for each day:
- `buy1` = max profit after 1st buy
- `sell1` = max profit after 1st sell
- `buy2` = max profit after 2nd buy
- `sell2` = max profit after 2nd sell

```java
public int maxProfit(int[] prices) {
    int buy1 = Integer.MIN_VALUE, sell1 = 0;
    int buy2 = Integer.MIN_VALUE, sell2 = 0;
    
    for (int price : prices) {
        buy1 = Math.max(buy1, -price);             // Best price to buy 1st
        sell1 = Math.max(sell1, buy1 + price);      // Best profit after sell 1st
        buy2 = Math.max(buy2, sell1 - price);       // Best after buy 2nd (using profit from 1st)
        sell2 = Math.max(sell2, buy2 + price);      // Best profit after sell 2nd
    }
    return sell2;
}
```

### 🧠 Dry Run

`prices = [3, 3, 5, 0, 0, 3, 1, 4]`

| Day | price | buy1 | sell1 | buy2 | sell2 |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 0 | 3 | -3 | 0 | 0 | 0 |
| 1 | 3 | -3 | 0 | 0 | 0 |
| 2 | 5 | -3 | 2 | 2 | 2 |
| 3 | 0 | **0** | 2 | 2 | 2 |
| 4 | 0 | 0 | 2 | 2 | 2 |
| 5 | 3 | 0 | 3 | 2 | 5 |
| 6 | 1 | 0 | 3 | 2 | 5 |
| 7 | 4 | 0 | 4 | 2 | **6** |

**Answer: 6** (Buy at 0, sell at 3, buy at 1, sell at 4 → 3 + 3 = 6)

### Variation IV (LC 188): At Most K Transactions
Generalize Variation III to k transactions.

```java
public int maxProfit(int k, int[] prices) {
    int n = prices.length;
    if (k >= n / 2) {  // Unlimited transactions
        int profit = 0;
        for (int i = 1; i < n; i++) {
            if (prices[i] > prices[i-1]) profit += prices[i] - prices[i-1];
        }
        return profit;
    }
    
    int[] buy = new int[k + 1];
    int[] sell = new int[k + 1];
    Arrays.fill(buy, Integer.MIN_VALUE);
    
    for (int price : prices) {
        for (int j = 1; j <= k; j++) {
            buy[j] = Math.max(buy[j], sell[j-1] - price);
            sell[j] = Math.max(sell[j], buy[j] + price);
        }
    }
    return sell[k];
}
```

### With Cooldown (LC 309)
After selling, you must wait 1 day before buying again.

```java
// States: hold (have stock), sold (just sold today), rest (cooldown/idle)
public int maxProfit(int[] prices) {
    int hold = Integer.MIN_VALUE; // Max profit while holding stock
    int sold = 0;                 // Max profit after selling today
    int rest = 0;                 // Max profit while in cooldown/idle
    
    for (int price : prices) {
        int prevSold = sold;
        sold = hold + price;             // Sell today
        hold = Math.max(hold, rest - price); // Buy today (must be from rest, not sold)
        rest = Math.max(rest, prevSold);     // Cooldown or continue idle
    }
    return Math.max(sold, rest);
}
```

---

## 17. Interval DP: Burst Balloons (MCM Pattern)

**Problem (LC 312):** Given `n` balloons with values `nums[i]`. Burst all balloons. When you burst balloon `i`, you get `nums[i-1] * nums[i] * nums[i+1]` coins. Find max coins.

### 🧠 How to Think (Matrix Chain Multiplication Pattern)
Instead of thinking "which balloon to burst first", think "which balloon to burst **LAST** in a range".

If balloon `k` is the last to burst in range `[i, j]`:
- Left subproblem: `dp[i][k-1]` (all balloons left of k are already burst)
- Right subproblem: `dp[k+1][j]` (all balloons right of k are already burst)
- Coins from bursting k last: `nums[i-1] * nums[k] * nums[j+1]` (neighbors are i-1 and j+1 because everything else is burst)

`dp[i][j]` = max coins from bursting all balloons in range `[i, j]`

```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    // Add boundary balloons with value 1
    int[] arr = new int[n + 2];
    arr[0] = arr[n + 1] = 1;
    for (int i = 0; i < n; i++) arr[i + 1] = nums[i];
    
    int[][] dp = new int[n + 2][n + 2];
    
    // Length of range: from 1 to n
    for (int len = 1; len <= n; len++) {
        for (int i = 1; i <= n - len + 1; i++) {
            int j = i + len - 1;
            
            for (int k = i; k <= j; k++) { // k = last balloon to burst in [i,j]
                int coins = arr[i-1] * arr[k] * arr[j+1]  // Coins from k
                          + dp[i][k-1]                      // Left subproblem
                          + dp[k+1][j];                     // Right subproblem
                
                dp[i][j] = Math.max(dp[i][j], coins);
            }
        }
    }
    return dp[1][n];
}
```

### 🧠 Dry Run

`nums = [3, 1, 5, 8]` → `arr = [1, 3, 1, 5, 8, 1]`

**Building dp table (length by length):**

```
Length 1 (single balloons):
dp[1][1]: burst 3 last → 1*3*1 = 3
dp[2][2]: burst 1 last → 3*1*5 = 15
dp[3][3]: burst 5 last → 1*5*8 = 40
dp[4][4]: burst 8 last → 5*8*1 = 40

Length 2:
dp[1][2]: 
  k=1 (3 last): 1*3*5 + dp[2][2] = 15+15 = 30
  k=2 (1 last): dp[1][1] + 1*1*5 = 3+5 = 8
  dp[1][2] = 30

dp[2][3]:
  k=2 (1 last): 3*1*8 + dp[3][3] = 8+40 = 48
  k=3 (5 last): dp[2][2] + 3*5*8 = 15+120 = 135
  dp[2][3] = 135

Length 4 (final answer):
dp[1][4] = 167
```

**Answer: 167**

---

## 18. DP Patterns Cheat Sheet

```
┌──────────────────────────────────────────────────────────────────┐
│                    DP PATTERN RECOGNITION                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Ways to reach / climb / decode"     →  1D DP (Fibonacci)      │
│  "Rob houses / pick elements"         →  1D DP (Include/Exclude)│
│  "Min coins / ways to make amount"    →  Unbounded Knapsack     │
│  "Select items with weight limit"     →  0/1 Knapsack           │
│  "Subset sum / partition"             →  0/1 Knapsack (boolean) │
│  "Two strings: common / edit / diff"  →  2D String DP (LCS)     │
│  "Palindrome subsequence"             →  LCS(s, reverse(s))     │
│  "Grid: paths / min cost"             →  2D Grid DP             │
│  "Longest increasing"                 →  LIS                    │
│  "Buy/sell stock with conditions"     →  State Machine DP       │
│  "Burst / merge / split ranges"       →  Interval DP (MCM)     │
│  "Break string into words"            →  1D DP + HashSet        │
│                                                                  │
│  SPACE OPTIMIZATION RULES:                                       │
│  • dp[i] depends on dp[i-1], dp[i-2]  →  Use 2 variables       │
│  • dp[i][j] depends on dp[i-1][...]   →  Use 1D array          │
│  • 0/1 knapsack 1D                    →  Iterate RIGHT to LEFT  │
│  • Unbounded knapsack 1D              →  Iterate LEFT to RIGHT  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 19. Top 25 Must-Do DP Questions

**Category 1: 1D DP (Fibonacci Pattern)**
1. Climbing Stairs (LC 70)
2. Min Cost Climbing Stairs (LC 746)
3. House Robber I (LC 198)
4. House Robber II (LC 213)
5. Decode Ways (LC 91)

**Category 2: 0/1 Knapsack Pattern**
6. Partition Equal Subset Sum (LC 416)
7. Target Sum (LC 494)
8. Last Stone Weight II (LC 1049)

**Category 3: Unbounded Knapsack**
9. Coin Change — Min Coins (LC 322)
10. Coin Change II — Count Ways (LC 518)
11. Perfect Squares (LC 279)

**Category 4: String DP (LCS Pattern)**
12. Longest Common Subsequence (LC 1143)
13. Edit Distance (LC 72)
14. Longest Palindromic Subsequence (LC 516)
15. Min Insertions to Make Palindrome (LC 1312)
16. Distinct Subsequences (LC 115)

**Category 5: Grid DP**
17. Unique Paths I & II (LC 62, 63)
18. Minimum Path Sum (LC 64)
19. Triangle (LC 120)

**Category 6: LIS Pattern**
20. Longest Increasing Subsequence (LC 300)
21. Russian Doll Envelopes (LC 354)

**Category 7: Buy & Sell Stock (State Machine)**
22. Best Time to Buy & Sell Stock I, II, III, IV (LC 121, 122, 123, 188)
23. With Cooldown (LC 309)
24. With Transaction Fee (LC 714)

**Category 8: Interval DP (MCM Pattern)**
25. Burst Balloons (LC 312)

---

## Tricks to Remember & Common Pitfalls

1. **Array Out of Bounds:** Always initialize your DP table with `N+1` or `M+1` to handle empty strings, 0 weights, or 0 items seamlessly without `index -1` exceptions.
2. **Space Optimization Tip:** If your equation looks like `dp[i][j] = ... dp[i-1]...`, you only need the *PREVIOUS ROW* in memory, not the whole matrix. You can reduce $O(M \times N)$ space to $O(N)$ space!
3. **Memoization Array Size:** The size of the memoization array is exactly the maximum values the changing parameters can take. E.g., if `f(int index, int target)` is called, and `index` goes up to 100, `target` up to 1000, memo array is `memo[101][1001]`.
4. **Initialization:** Initialize memo arrays with `-1` (not `0`), because `0` could be a valid mathematical answer to the subproblem.
5. **0/1 vs Unbounded in 1D:** Right-to-left loop = 0/1 (each item once). Left-to-right loop = Unbounded (item reuse).
6. **String DP Base Case:** Always handle empty string (`i=0` or `j=0`) in the first row/column.
7. **Print the answer:** After building the DP table, backtrack from `dp[m][n]` by checking which direction (diagonal, up, left) gave the optimal value.
