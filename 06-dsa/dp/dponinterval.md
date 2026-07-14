# 🧠 Interval Dynamic Programming — Beginner to Expert Guide

Interval DP is a pattern where the state `dp[i][j]` represents the optimal solution for a subsegment or substring spanning from index `i` to index `j`. The transition usually involves dividing the interval `[i, j]` into smaller sub-intervals `[i, k]` and `[k+1, j]` at a partition point `k` (where $i \le k < j$), and combining their results.

---

## 📑 Table of Contents
1. [Core Mechanics of Interval DP](#core-mechanics-of-interval-dp)
2. [Problem 1: Matrix Chain Multiplication (MCM) (Baseline)](#problem-1-matrix-chain-multiplication-mcm-baseline)
3. [Problem 2: Longest Palindromic Subsequence (LC 516)](#problem-2-longest-palindromic-subsequence-lc-516)
4. [Problem 3: Burst Balloons (LC 312)](#problem-3-burst-balloons-lc-312)
5. [Problem 4: Minimum Cost to Cut a Stick (LC 1547)](#problem-4-minimum-cost-to-cut-a-stick-lc-1547)
6. [Problem 5: Strange Printer (LC 664)](#problem-5-strange-printer-lc-664)

---

## Core Mechanics of Interval DP

In standard 1D/2D DP, we fill the table linearly row-by-row or col-by-col.
In Interval DP, we compute the table **diagonal-by-diagonal** based on the **length of the interval** (difference between `j` and `i`).
1.  **Base Cases:** Intervals of length 1 (`i == j`) or length 2 (`j - i == 1`).
2.  **Iterative Loop:**
    ```java
    for (int len = 1; len <= n; len++) { // Length of interval
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            // Solve for interval [i, j]
        }
    }
    ```

---

## Problem 1: Matrix Chain Multiplication (MCM) (Baseline)

**Problem Statement:** Given a sequence of matrices, find the most efficient way to multiply these matrices together. The problem is not actually to perform the multiplications, but merely to decide in which order to perform the multiplications.
You are given an array `arr[]` which represents the chain of matrices such that the $i$-th matrix $A_i$ has dimensions `arr[i-1] x arr[i]`.
*Input:* `arr = [40, 20, 30, 10, 30]`  
*Output:* `26000`  
Explanation:
- There are 4 matrices of dimensions: $40 \times 20, 20 \times 30, 30 \times 10, 10 \times 30$.
- Let them be $A, B, C, D$.
- Optimal multiplication order: $(A(BC))D$ or $A((BC)D)$, etc.
- Minimum operations = 26,000.

#### 🧠 State Space Mapping
Let `dp[i][j]` be the minimum multiplication operations to multiply matrices from index `i` to `j`.
To calculate `dp[i][j]`, we select partition point `k` ($i \le k < j$):
`dp[i][j] = min(dp[i][k] + dp[k+1][j] + arr[i-1] * arr[k] * arr[j])`

#### 1. Recursive Code with Memoization
```java
import java.util.Arrays;

public class MCM {
    public int matrixMultiplication(int[] arr) {
        int n = arr.length;
        int[][] dp = new int[n][n];
        for (int[] row : dp) Arrays.fill(row, -1);
        
        // Matrix index starts at 1, goes up to n-1
        return solve(1, n - 1, arr, dp);
    }
    
    private int solve(int i, int j, int[] arr, int[][] dp) {
        // Base case: Single matrix has 0 multiplication cost
        if (i >= j) return 0;
        
        if (dp[i][j] != -1) return dp[i][j];
        
        int minOps = Integer.MAX_VALUE;
        for (int k = i; k < j; k++) {
            int ops = solve(i, k, arr, dp) 
                     + solve(k + 1, j, arr, dp) 
                     + arr[i-1] * arr[k] * arr[j];
            minOps = Math.min(minOps, ops);
        }
        
        return dp[i][j] = minOps;
    }
}
```
*Time Complexity:* $O(N^3)$  
*Space Complexity:* $O(N^2)$

#### 2. Bottom-Up (Tabulation) Code
```java
public class MCMTabulation {
    public int matrixMultiplication(int[] arr) {
        int n = arr.length;
        int[][] dp = new int[n][n];
        
        // Base case: dp[i][i] is already initialized to 0
        
        // Fill table diagonal by diagonal (increasing interval length)
        for (int len = 2; len < n; len++) {
            for (int i = 1; i < n - len + 1; i++) {
                int j = i + len - 1;
                dp[i][j] = Integer.MAX_VALUE;
                
                for (int k = i; k < j; k++) {
                    int cost = dp[i][k] + dp[k+1][j] + arr[i-1] * arr[k] * arr[j];
                    dp[i][j] = Math.min(dp[i][j], cost);
                }
            }
        }
        
        return dp[1][n-1];
    }
}
```
*Time Complexity:* $O(N^3)$  
*Space Complexity:* $O(N^2)$

---

## Problem 2: Longest Palindromic Subsequence (LC 516)

**Problem Statement:** Given a string `s`, find the longest palindromic subsequence's length in `s`.
*Input:* `s = "bbbab"`  
*Output:* `4` (The LPS is `"bbbb"`).

#### 🧠 Interval State Space
Let `dp[i][j]` be the length of the LPS in substring `s[i...j]`.
- If `s.charAt(i) == s.charAt(j)`: These two characters can match, adding `2` to the LPS of the inner substring.
  `dp[i][j] = 2 + dp[i+1][j-1]`
- If `s.charAt(i) != s.charAt(j)`: We take the maximum LPS by either skipping `i` or skipping `j`.
  `dp[i][j] = max(dp[i+1][j], dp[i][j-1])`

#### Bottom-Up (Tabulation) Code
```java
public class LPSIntervalDP {
    public int longestPalindromeSubseq(String s) {
        int n = s.length();
        int[][] dp = new int[n][n];
        
        // Base Cases: single characters are palindromes of length 1
        for (int i = 0; i < n; i++) {
            dp[i][i] = 1;
        }
        
        for (int len = 2; len <= n; len++) {
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                
                if (s.charAt(i) == s.charAt(j)) {
                    dp[i][j] = 2 + (len == 2 ? 0 : dp[i+1][j-1]);
                } else {
                    dp[i][j] = Math.max(dp[i+1][j], dp[i][j-1]);
                }
            }
        }
        
        return dp[0][n-1];
    }
}
```
*Time Complexity:* $O(N^2)$  
*Space Complexity:* $O(N^2)$

#### 🧠 Dry Run Matrix (s = "bbbab")
The table is filled diagonally starting from length 1 up to 5:

| Index `i \ j` | 0 (b) | 1 (b) | 2 (b) | 3 (a) | 4 (b) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **0 (b)** | 1 | 2 | 3 | 3 | **4** |
| **1 (b)** | — | 1 | 2 | 2 | 3 |
| **2 (b)** | — | — | 1 | 1 | 2 |
| **3 (a)** | — | — | — | 1 | 1 |
| **4 (b)** | — | — | — | — | 1 |

*Final Result:* `dp[0][4] = 4`.

---

## Problem 3: Burst Balloons (LC 312)

**Problem Statement:** You are given $n$ balloons, indexed from $0$ to $n-1$. Each balloon is painted with a number on it represented by an array `nums`. You are asked to burst all the balloons.
If you burst the $i$-th balloon, you will get `nums[i-1] * nums[i] * nums[i+1]` coins. After it is burst, the $i$-th and $(i+1)$-th balloons become adjacent.
Find the maximum coins you can collect by bursting the balloons wisely.
Assume `nums[-1] = nums[n] = 1`.
*Input:* `nums = [3, 1, 5, 8]`  
*Output:* `167`

#### 🧠 Trick: Reverse the Thinking
Instead of asking "which balloon to burst first", which changes the adjacency of all remaining balloons, we ask:
**"Which balloon is the LAST one to burst in the interval `[i, j]`?"**
If balloon `k` is the last to burst in `[i, j]`, then the balloons adjacent to `k` at the time of its burst are `i-1` and `j+1`.
`dp[i][j] = max(dp[i][k-1] + dp[k+1][j] + nums[i-1] * nums[k] * nums[j+1])` for $i \le k \le j$.

#### Java Code
```java
public class BurstBalloons {
    public int maxCoins(int[] nums) {
        int n = nums.length;
        
        // Pad the array with 1 at both boundaries
        int[] padded = new int[n + 2];
        padded[0] = 1;
        padded[n + 1] = 1;
        System.arraycopy(nums, 0, padded, 1, n);
        
        int[][] dp = new int[n + 2][n + 2];
        
        for (int len = 1; len <= n; len++) {
            for (int i = 1; i <= n - len + 1; i++) {
                int j = i + len - 1;
                
                for (int k = i; k <= j; k++) {
                    int coins = dp[i][k-1] 
                              + dp[k+1][j] 
                              + padded[i-1] * padded[k] * padded[j+1];
                    dp[i][j] = Math.max(dp[i][j], coins);
                }
            }
        }
        
        return dp[1][n];
    }
}
```
*Time Complexity:* $O(N^3)$  
*Space Complexity:* $O(N^2)$

---

## Problem 4: Minimum Cost to Cut a Stick (LC 1547)

**Problem Statement:** Given a wooden stick of length `n`, and an integer array `cuts` where `cuts[i]` denotes a position you should perform a cut at.
You should perform the cuts in order, you can change the order of the cuts as you wish.
The cost of one cut is the length of the stick to be cut.
Return the minimum total cost of the cuts.

#### 🧠 Interval Configuration
Add `0` and `n` to the `cuts` array, and sort it. Now, cuts are at `cuts[0], cuts[1], ..., cuts[m-1]`.
Let `dp[i][j]` be the minimum cost to make all cuts between `cuts[i]` and `cuts[j]`.
For a cut at index `k` ($i < k < j$):
`dp[i][j] = min(dp[i][k] + dp[k][j] + cuts[j] - cuts[i])`

#### Java Code
```java
import java.util.Arrays;

public class CutStick {
    public int minCost(int n, int[] cuts) {
        int m = cuts.length;
        int[] newCuts = new int[m + 2];
        newCuts[0] = 0;
        newCuts[m + 1] = n;
        System.arraycopy(cuts, 0, newCuts, 1, m);
        Arrays.sort(newCuts);
        
        int numCuts = newCuts.length;
        int[][] dp = new int[numCuts][numCuts];
        
        for (int len = 2; len < numCuts; len++) {
            for (int i = 0; i < numCuts - len; i++) {
                int j = i + len;
                dp[i][j] = Integer.MAX_VALUE;
                
                for (int k = i + 1; k < j; k++) {
                    int cost = dp[i][k] + dp[k][j] + newCuts[j] - newCuts[i];
                    dp[i][j] = Math.min(dp[i][j], cost);
                }
            }
        }
        
        return dp[0][numCuts-1];
    }
}
```
*Time Complexity:* $O(M^3)$ where $M$ is the number of cuts.  
*Space Complexity:* $O(M^2)$

---

## Problem 5: Strange Printer (LC 664)

**Problem Statement:** There is a strange printer with the following two special properties:
1.  The printer can only print a sequence of the same character each time.
2.  At each turn, the printer can print new characters starting from any position and ending at any position, and will cover the original existing characters.

Given a string `s`, return the minimum number of turns the printer needed to print it.
*Input:* `s = "aba"`  
*Output:* `2` (Print "aaa" first, then paint 'b' at index 1 to make "aba").

#### 🧠 Intuition & State Transitions
Let `dp[i][j]` be the minimum prints needed for substring `s[i...j]`.
- **Default Option:** We can print `s[i]` separately, and then print `s[i+1...j]`.
  `dp[i][j] = 1 + dp[i+1][j]`
- **Merge Option:** If there is some character `s[k] == s[i]` for $i+1 \le k \le j$, we can print `s[i]` and `s[k]` in the *same* print stroke, merging the cost.
  `dp[i][j] = min(dp[i][j], dp[i][k-1] + dp[k+1][j])`

#### Java Code
```java
public class StrangePrinter {
    public int strangePrinter(String s) {
        if (s == null || s.length() == 0) return 0;
        int n = s.length();
        int[][] dp = new int[n][n];
        
        for (int i = 0; i < n; i++) {
            dp[i][i] = 1; // 1 print for a single character
        }
        
        for (int len = 2; len <= n; len++) {
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                
                // Base option
                dp[i][j] = 1 + dp[i+1][j];
                
                // Check if any character matches the starting character
                for (int k = i + 1; k <= j; k++) {
                    if (s.charAt(k) == s.charAt(i)) {
                        int cost = dp[i][k-1] + (k == j ? 0 : dp[k+1][j]);
                        dp[i][j] = Math.min(dp[i][j], cost);
                    }
                }
            }
        }
        
        return dp[0][n-1];
    }
}
```
*Time Complexity:* $O(N^3)$  
*Space Complexity:* $O(N^2)$
