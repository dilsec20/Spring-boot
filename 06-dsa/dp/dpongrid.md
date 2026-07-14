# 🧠 Dynamic Programming on Grids — Beginner to Expert Guide

Grid-based Dynamic Programming represents a major subset of 2D/3D DP problems. In these problems, you typically navigate a grid (matrix) from a starting cell (e.g., top-left) to a destination cell (e.g., bottom-right) under constraint vectors (e.g., you can only move Down or Right).

---

## 📑 Table of Contents
1. [Core Grid DP Mechanics](#core-grid-dp-mechanics)
2. [Problem 1: Unique Paths I & II (Standard Grid Traversal)](#problem-1-unique-paths-i--ii-standard-grid-traversal)
3. [Problem 2: Minimum Path Sum (Path Weight Minimization)](#problem-2-minimum-path-sum-path-weight-minimization)
4. [Problem 3: Triangle (Non-Rectangular Grid DP)](#problem-3-triangle-non-rectangular-grid-dp)
5. [Problem 4: Maximal Square (Submatrix Area Search)](#problem-4-maximal-square-submatrix-area-search)
6. [Problem 5: Dungeon Game (Reverse Transition DP)](#problem-5-dungeon-game-reverse-transition-dp)
7. [Problem 6: Cherry Pickup II (3D State Space Dual Traversal)](#problem-6-cherry-pickup-ii-3d-state-space-dual-traversal)

---

## Core Grid DP Mechanics

When solving grid DP problems, we define the state `dp[r][c]` representing the optimal value (cost, path count, area, etc.) ending at row `r` and column `c`.

### Directional Vectors
If you can move:
*   **Right or Down:** To arrive at cell `(r, c)`, you must have come from `(r-1, c)` (Down) or `(r, c-1)` (Right).
    `dp[r][c] = f(dp[r-1][c], dp[r][c-1])`
*   **Diagonally (Down-Right):** You can also come from `(r-1, c-1)`.
    `dp[r][c] = f(dp[r-1][c], dp[r][c-1], dp[r-1][c-1])`

### Boundary Checking
To prevent `IndexOutOfBoundsException` in Java:
- Always handle the row index `r < 0` or `r >= rows`.
- Always handle the col index `c < 0` or `c >= cols`.

---

## Problem 1: Unique Paths I & II (Standard Grid Traversal)

### Unique Paths I (LC 62)
**Problem Statement:** There is a robot on an $m \times n$ grid. The robot is initially located at the top-left corner (`grid[0][0]`). The robot tries to move to the bottom-right corner (`grid[m-1][n-1]`). The robot can only move either down or right at any point in time.
Given the two integers $m$ and $n$, return the number of possible unique paths that the robot can take to reach the bottom-right corner.

#### 1. Recursive Code (Brute Force)
```java
public class UniquePaths {
    public int uniquePaths(int m, int n) {
        return countPaths(m - 1, n - 1);
    }
    
    private int countPaths(int r, int c) {
        // Base case: If we reach the starting cell, we found 1 valid path
        if (r == 0 && c == 0) return 1;
        // Out of bounds
        if (r < 0 || c < 0) return 0;
        
        // Sum paths from top and left neighbors
        int up = countPaths(r - 1, c);
        int left = countPaths(r, c - 1);
        
        return up + left;
    }
}
```
*Time Complexity:* $O(2^{M+N})$  
*Space Complexity:* $O(M + N)$ recursion stack.

#### 2. Top-Down (Memoization) Code
```java
import java.util.Arrays;

public class UniquePathsMemo {
    public int uniquePaths(int m, int n) {
        int[][] memo = new int[m][n];
        for (int[] row : memo) Arrays.fill(row, -1);
        return countPaths(m - 1, n - 1, memo);
    }
    
    private int countPaths(int r, int c, int[][] memo) {
        if (r == 0 && c == 0) return 1;
        if (r < 0 || c < 0) return 0;
        if (memo[r][c] != -1) return memo[r][c];
        
        int up = countPaths(r - 1, c, memo);
        int left = countPaths(r, c - 1, memo);
        
        return memo[r][c] = up + left;
    }
}
```
*Time Complexity:* $O(M \cdot N)$  
*Space Complexity:* $O(M \cdot N)$ memo table + $O(M+N)$ stack space.

#### 3. Bottom-Up (Tabulation) Code
```java
public class UniquePathsTabulation {
    public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        
        // Base Case: start is 1
        dp[0][0] = 1;
        
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (r == 0 && c == 0) continue;
                int up = (r > 0) ? dp[r-1][c] : 0;
                int left = (c > 0) ? dp[r][c-1] : 0;
                dp[r][c] = up + left;
            }
        }
        
        return dp[m-1][n-1];
    }
}
```
*Time Complexity:* $O(M \cdot N)$  
*Space Complexity:* $O(M \cdot N)$

#### 🧠 Dry Run Table (m = 3, n = 3)

| Row `r` | Col `c` | Transition: `dp[r-1][c] + dp[r][c-1]` | dp[r][c] | Explanation |
| :--- | :--- | :--- | :--- | :--- |
| **0** | **0** | Base Case | **1** | Start cell. |
| **0** | **1** | $dp[-1][1] (\text{out}) + dp[0][0] = 0 + 1$ | **1** | Only 1 way to go right along the edge. |
| **0** | **2** | $dp[-1][2] + dp[0][1] = 0 + 1$ | **1** | Only 1 way. |
| **1** | **0** | $dp[0][0] + dp[1][-1] = 1 + 0$ | **1** | Only 1 way down. |
| **1** | **1** | $dp[0][1] + dp[1][0] = 1 + 1$ | **2** | Can reach via `(0,1)` or `(1,0)`. |
| **1** | **2** | $dp[0][2] + dp[1][1] = 1 + 2$ | **3** | Reach via `(0,2)` or `(1,1)`. |
| **2** | **0** | $dp[1][0] + dp[2][-1] = 1 + 0$ | **1** | Only 1 way down. |
| **2** | **1** | $dp[1][1] + dp[2][0] = 2 + 1$ | **3** | Reach via `(1,1)` or `(2,0)`. |
| **2** | **2** | $dp[1][2] + dp[2][1] = 3 + 3$ | **6** | **Answer: 6 unique paths.** |

#### 4. Space Optimized Code
Since `dp[r][c]` only requires the current row's left element `dp[r][c-1]` and the previous row's element `dp[r-1][c]`, we can compress the space to a single 1D array of size `n` representing the active row.
```java
import java.util.Arrays;

public class UniquePathsOptimized {
    public int uniquePaths(int m, int n) {
        int[] dp = new int[n];
        Arrays.fill(dp, 1); // Row 0 is all 1s
        
        for (int r = 1; r < m; r++) {
            for (int c = 1; c < n; c++) {
                dp[c] = dp[c] + dp[c-1]; // dp[c] is dp[r-1][c], dp[c-1] is dp[r][c-1]
            }
        }
        return dp[n-1];
    }
}
```
*Time Complexity:* $O(M \cdot N)$  
*Space Complexity:* $O(N)$

---

### Unique Paths II (With Obstacles) (LC 63)
**Problem Statement:** Same, but there are obstacles in the grid, represented by `1`. You cannot pass through obstacles.
*Input:* `obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]`  
*Output:* `2`

```java
public class UniquePathsII {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int m = obstacleGrid.length;
        int n = obstacleGrid[0].length;
        
        if (obstacleGrid[0][0] == 1 || obstacleGrid[m-1][n-1] == 1) return 0;
        
        int[] dp = new int[n];
        dp[0] = 1; // Base Case
        
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (obstacleGrid[r][c] == 1) {
                    dp[c] = 0; // Obstacle blocks all paths
                } else if (c > 0) {
                    dp[c] = dp[c] + dp[c-1];
                }
            }
        }
        
        return dp[n-1];
    }
}
```
*Time Complexity:* $O(M \cdot N)$  
*Space Complexity:* $O(N)$

---

## Problem 2: Minimum Path Sum (Path Weight Minimization)

### Minimum Path Sum (LC 64)
**Problem Statement:** Given a $m \times n$ grid filled with non-negative numbers, find a path from top-left to bottom-right, which minimizes the sum of all numbers along its path.
*Input:* `grid = [[1,3,1],[1,5,1],[4,2,1]]`  
*Output:* `7` (Path: 1 → 3 → 1 → 1 → 1. Sum = 1+3+1+1+1 = 7).

#### 🧠 State Space & Transition
`dp[r][c] = grid[r][c] + min(dp[r-1][c], dp[r][c-1])`

#### 1. Java Tabulation Code
```java
public class MinPathSum {
    public int minPathSum(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        int[][] dp = new int[m][n];
        
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (r == 0 && c == 0) {
                    dp[0][0] = grid[0][0];
                } else {
                    int up = (r > 0) ? dp[r-1][c] : Integer.MAX_VALUE;
                    int left = (c > 0) ? dp[r][c-1] : Integer.MAX_VALUE;
                    dp[r][c] = grid[r][c] + Math.min(up, left);
                }
            }
        }
        
        return dp[m-1][n-1];
    }
}
```

#### 2. Space Optimized Code
```java
public class MinPathSumOptimized {
    public int minPathSum(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        int[] dp = new int[n];
        
        dp[0] = grid[0][0];
        // Populate first row
        for (int c = 1; c < n; c++) {
            dp[c] = dp[c-1] + grid[0][c];
        }
        
        for (int r = 1; r < m; r++) {
            dp[0] = dp[0] + grid[r][0]; // First element of current row
            for (int c = 1; c < n; c++) {
                dp[c] = grid[r][c] + Math.min(dp[c], dp[c-1]);
            }
        }
        
        return dp[n-1];
    }
}
```
*Time Complexity:* $O(M \cdot N)$  
*Space Complexity:* $O(N)$

---

## Problem 3: Triangle (Non-Rectangular Grid DP)

### Triangle (LC 120)
**Problem Statement:** Given a `triangle` array, return the minimum path sum from top to bottom.
For each step, you may move to an adjacent number of the row below. More formally, if you are on index `i` on the current row, you may move to either index `i` or index `i + 1` on the next row.
*Input:* `triangle = [[2],[3,4],[6,5,7],[4,1,8,3]]`  
*Output:* `11` (Path: 2 → 3 → 5 → 1. Sum = 2+3+5+1 = 11).

#### 🧠 Bottom-Up (Tabulation) without Grid Copy
Since we start from the bottom row and move upwards, we can represent our state as:
`dp[i] = triangle[row][i] + min(dp[i], dp[i+1])`
This matches moving to adjacent children on the row below!

```java
import java.util.List;

public class TriangleDP {
    public int minimumTotal(List<List<Integer>> triangle) {
        int n = triangle.size();
        // dp array stores min path sum starting from row `r` at col `c`
        int[] dp = new int[n + 1];
        
        // Traverse backwards from bottom row to top row
        for (int r = n - 1; r >= 0; r--) {
            List<Integer> row = triangle.get(r);
            for (int c = 0; c < row.size(); c++) {
                dp[c] = row.get(c) + Math.min(dp[c], dp[c+1]);
            }
        }
        
        return dp[0];
    }
}
```
*Time Complexity:* $O(N^2)$ where $N$ is the number of rows.  
*Space Complexity:* $O(N)$

---

## Problem 4: Maximal Square (Submatrix Area Search)

### Maximal Square (LC 221)
**Problem Statement:** Given an $m \times n$ binary `matrix` filled with `'0'`s and `'1'`s, find the largest square containing only `'1'`s and return its area.
*Input:*
```
matrix = [
  ["1","0","1","0","0"],
  ["1","0","1","1","1"],
  ["1","1","1","1","1"],
  ["1","0","0","1","0"]
]
```
*Output:* `4` (2x2 square of ones).

#### 🧠 Intuition & Approach
Let `dp[r][c]` be the side length of the largest square of `'1'`s whose bottom-right corner is at cell `(r, c)`.
If `matrix[r][c] == '1'`, then a square can be formed if and only if its three adjacent predecessor squares (top, left, top-left) also contain `'1'`.
The limiting factor is the minimum of these three.
`dp[r][c] = 1 + min(dp[r-1][c], dp[r][c-1], dp[r-1][c-1])`

#### Complete Java Code
```java
public class MaximalSquare {
    public int maximalSquare(char[][] matrix) {
        if (matrix == null || matrix.length == 0) return 0;
        int m = matrix.length;
        int n = matrix[0].length;
        
        int[][] dp = new int[m + 1][n + 1];
        int maxSide = 0;
        
        for (int r = 1; r <= m; r++) {
            for (int c = 1; c <= n; c++) {
                if (matrix[r-1][c-1] == '1') {
                    dp[r][c] = 1 + Math.min(dp[r-1][c], 
                                   Math.min(dp[r][c-1], dp[r-1][c-1]));
                    maxSide = Math.max(maxSide, dp[r][c]);
                }
            }
        }
        
        return maxSide * maxSide; // Return area
    }
}
```
*Time Complexity:* $O(M \cdot N)$  
*Space Complexity:* $O(M \cdot N)$

---

## Problem 5: Dungeon Game (Reverse Transition DP)

### Dungeon Game (LC 174)
**Problem Statement:** The demons had captured the princess and imprisoned her in the bottom-right corner of a dungeon. The dungeon consists of $m \times n$ rooms. Our knight was initially positioned in the top-left room and must fight his way through dungeon to rescue the princess.
The knight has an initial health point represented by a positive integer. If at any point his health drops to 0 or below, he dies immediately.
Some of the rooms are guarded by demons (negative integers representing health loss), others contain magic orbs (positive integers representing health gain).
In order to reach the princess, the knight can only move rightward or downward.
Determine the knight's minimum initial health so that he can rescue the princess.

#### 🧠 Why Reverse DP?
If we do standard top-down (left-to-right) DP, we don't know how much health we need later. A path that starts with high health might hit a dead end, whereas a path with low health could find health potions later.
Instead, we start at the **Princess's room (bottom-right)** and work backwards to the **Knight's room (top-left)**.
`dp[r][c]` represents the minimum health required *before* entering room `(r, c)` to complete the journey alive.

#### 1. Java Tabulation Code
```java
import java.util.Arrays;

public class DungeonGame {
    public int calculateMinimumHP(int[][] dungeon) {
        int m = dungeon.length;
        int n = dungeon[0].length;
        
        int[][] dp = new int[m][n];
        
        // Base case: Minimum health needed at princess room
        dp[m-1][n-1] = Math.max(1, 1 - dungeon[m-1][n-1]);
        
        // Last Column (can only move Down)
        for (int r = m - 2; r >= 0; r--) {
            dp[r][n-1] = Math.max(1, dp[r+1][n-1] - dungeon[r][n-1]);
        }
        
        // Last Row (can only move Right)
        for (int c = n - 2; c >= 0; c--) {
            dp[m-1][c] = Math.max(1, dp[m-1][c+1] - dungeon[m-1][c]);
        }
        
        // Fill rest of the grid from bottom-right up
        for (int r = m - 2; r >= 0; r--) {
            for (int c = n - 2; c >= 0; c--) {
                int nextHealth = Math.min(dp[r+1][c], dp[r][c+1]);
                dp[r][c] = Math.max(1, nextHealth - dungeon[r][c]);
            }
        }
        
        return dp[0][0];
    }
}
```
*Time Complexity:* $O(M \cdot N)$  
*Space Complexity:* $O(M \cdot N)$

---

## Problem 6: Cherry Pickup II (3D State Space Dual Traversal)

### Cherry Pickup II (LC 1463)
**Problem Statement:** You are given a $rows \times cols$ matrix `grid` representing a field of cherries. You have two robots:
*   Robot 1 starts at the top-left corner `(0, 0)`.
*   Robot 2 starts at the top-right corner `(0, cols-1)`.

Both robots must reach the bottom row of the grid. In each step:
*   From `(r, c)`, a robot can move to `(r+1, c-1)`, `(r+1, c)`, or `(r+1, c+1)`.
*   If both robots land on the same cell, only one collects the cherries.
Return the maximum cherries both robots can collect.

#### 🧠 State Space Mapping
Since both robots move down one row per step, they will always be on the **same row `r`**.
Thus, our state is `dp[r][c1][c2]`, representing:
- Row index `r` (ranges from $0$ to $\text{rows}-1$).
- Column of Robot 1 `c1` (ranges from $0$ to $\text{cols}-1$).
- Column of Robot 2 `c2` (ranges from $0$ to $\text{cols}-1$).

For each step, both robots have 3 possible column choices: `dc1 ∈ {-1, 0, 1}` and `dc2 ∈ {-1, 0, 1}`. This leads to $3 \times 3 = 9$ possible transitions for `(r+1, c1+dc1, c2+dc2)`.

#### 1. Complete Java Code (Memoization)
```java
import java.util.Arrays;

public class CherryPickupII {
    public int cherryPickup(int[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;
        int[][][] memo = new int[rows][cols][cols];
        for (int[][] m2 : memo) {
            for (int[] m1 : m2) {
                Arrays.fill(m1, -1);
            }
        }
        return solve(0, 0, cols - 1, grid, memo);
    }
    
    private int solve(int r, int c1, int c2, int[][] grid, int[][][] memo) {
        int rows = grid.length;
        int cols = grid[0].length;
        
        // Out of bounds checks
        if (c1 < 0 || c1 >= cols || c2 < 0 || c2 >= cols) return 0;
        
        if (r == rows) return 0;
        
        if (memo[r][c1][c2] != -1) return memo[r][c1][c2];
        
        // Calculate cherries at current cells
        int cherries = grid[r][c1];
        if (c1 != c2) {
            cherries += grid[r][c2]; // Add Robot 2 cherries only if they are on different cells
        }
        
        // Explore 9 next combinations
        int maxNext = 0;
        for (int dc1 = -1; dc1 <= 1; dc1++) {
            for (int dc2 = -1; dc2 <= 1; dc2++) {
                maxNext = Math.max(maxNext, solve(r + 1, c1 + dc1, c2 + dc2, grid, memo));
            }
        }
        
        return memo[r][c1][c2] = cherries + maxNext;
    }
}
```
*Time Complexity:* $O(\text{Rows} \cdot \text{Cols}^2)$  
*Space Complexity:* $O(\text{Rows} \cdot \text{Cols}^2)$
