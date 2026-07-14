# 🧠 Dynamic Programming on Arrays — Beginner to Expert Guide

Dynamic Programming on arrays is the most common category of DP questions in technical interviews. It ranges from simple 1D state transitions to complex multi-state transitions, subarray optimizations, and subsequence matching.

---

## 📑 Table of Contents
1. [Core Concepts: Subarray vs. Subsequence vs. Subset](#core-concepts-subarray-vs-subsequence-vs-subset)
2. [Problem 1: House Robber I & II (Linear vs. Circular DP)](#problem-1-house-robber-i--ii-linear-vs-circular-dp)
3. [Problem 2: Partition Equal Subset Sum (Subset Sum Pattern)](#problem-2-partition-equal-subset-sum-subset-sum-pattern)
4. [Problem 3: Coin Change I & II (Unbounded Knapsack vs. 0/1 Knapsack)](#problem-3-coin-change-i--ii-unbounded-knapsack-vs-01-knapsack)
5. [Problem 4: Longest Increasing Subsequence (LIS Pattern)](#problem-4-longest-increasing-subsequence-lis-pattern)
6. [Problem 5: Best Time to Buy and Sell Stock with Cooldown & Transaction Fee](#problem-5-best-time-to-buy-and-sell-stock-with-cooldown--transaction-fee)
7. [Problem 6: Maximum Product Subarray (Min/Max Tracking Pattern)](#problem-6-maximum-product-subarray-minmax-tracking-pattern)

---

## Core Concepts: Subarray vs. Subsequence vs. Subset

When working with array DP, it is critical to understand the distinction between these three terms:

| Term | Continuity | Order | Example for `[1, 2, 3]` |
| :--- | :--- | :--- | :--- |
| **Subarray** | Must be contiguous | Must maintain relative order | `[1, 2]`, `[2, 3]` (but NOT `[1, 3]`) |
| **Subsequence** | Can be non-contiguous | Must maintain relative order | `[1, 3]`, `[1, 2, 3]`, `[2]` |
| **Subset** | Can be non-contiguous | Order does not matter | `[3, 1]`, `[1, 3]`, `[2, 3]` |

---

## Problem 1: House Robber I & II (Linear vs. Circular DP)

### House Robber I (LC 198)
**Problem Statement:** You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed. The only constraint stopping you from robbing each of them is that adjacent houses have security systems connected and it will automatically contact the police if two adjacent houses were broken into on the same night.
*Input:* `nums = [2, 7, 9, 3, 1]`  
*Output:* `12` (Rob house 1 (2), house 3 (9), and house 5 (1). Total = 2 + 9 + 1 = 12)

#### 🧠 Intuition & Approach
For each house `i`, we have two choices:
1. **Rob house `i`:** We collect `nums[i]` and must skip house `i-1`. Thus, we add the maximum loot from houses up to `i-2`.
2. **Skip house `i`:** We collect nothing from house `i`. The maximum loot remains the same as the maximum loot up to house `i-1`.

#### 1. Recursive Code (Brute Force)
```java
public class HouseRobber {
    public int robRecursive(int[] nums) {
        return robHelper(nums, nums.length - 1);
    }
    
    private int robHelper(int[] nums, int i) {
        if (i < 0) return 0;
        if (i == 0) return nums[0];
        
        // Choice 1: Rob current house, skip adjacent
        int rob = nums[i] + robHelper(nums, i - 2);
        // Choice 2: Skip current house
        int skip = robHelper(nums, i - 1);
        
        return Math.max(rob, skip);
    }
}
```
*Time Complexity:* $O(2^N)$  
*Space Complexity:* $O(N)$ auxiliary stack space.

#### 2. Top-Down (Memoization) Code
```java
import java.util.Arrays;

public class HouseRobberMemo {
    public int robMemo(int[] nums) {
        int[] memo = new int[nums.length];
        Arrays.fill(memo, -1);
        return robHelper(nums, nums.length - 1, memo);
    }
    
    private int robHelper(int[] nums, int i, int[] memo) {
        if (i < 0) return 0;
        if (i == 0) return nums[0];
        if (memo[i] != -1) return memo[i];
        
        int rob = nums[i] + robHelper(nums, i - 2, memo);
        int skip = robHelper(nums, i - 1, memo);
        
        return memo[i] = Math.max(rob, skip);
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(N)$ memo array + $O(N)$ recursion stack.

#### 3. Bottom-Up (Tabulation) Code
```java
public class HouseRobberTabulation {
    public int robTab(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        if (nums.length == 1) return nums[0];
        
        int n = nums.length;
        int[] dp = new int[n];
        
        // Base Cases
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);
        
        for (int i = 2; i < n; i++) {
            dp[i] = Math.max(nums[i] + dp[i-2], dp[i-1]);
        }
        
        return dp[n-1];
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(N)$ for the `dp` array.

#### 🧠 Dry Run Table (nums = [2, 7, 9, 3, 1])

| Index `i` | nums[i] | Rob Choice (`nums[i] + dp[i-2]`) | Skip Choice (`dp[i-1]`) | dp[i] (Max) | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | 2 | 2 (base case) | - | **2** | Only house 0 is available to rob. |
| **1** | 7 | 7 (rob house 1 only) | 2 (skip house 1, rob house 0) | **7** | Max of house 0 and house 1. |
| **2** | 9 | $9 + dp[0] = 9 + 2 = 11$ | $dp[1] = 7$ | **11** | Robbing house 2 + 0 gives 11, skipping gives 7. Max is 11. |
| **3** | 3 | $3 + dp[1] = 3 + 7 = 10$ | $dp[2] = 11$ | **11** | Robbing house 3 + 1 gives 10. Skipping house 3 is better (11). |
| **4** | 1 | $1 + dp[2] = 1 + 11 = 12$ | $dp[3] = 11$ | **12** | Robbing house 4 + 2 gives 12, skipping gives 11. Max is 12. |

#### 4. Space Optimized Code
Since `dp[i]` only depends on `dp[i-1]` (skip) and `dp[i-2]` (rob), we can optimize space to $O(1)$.
```java
public class HouseRobberOptimized {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        if (nums.length == 1) return nums[0];
        
        int prev2 = nums[0];
        int prev1 = Math.max(nums[0], nums[1]);
        
        for (int i = 2; i < nums.length; i++) {
            int curr = Math.max(nums[i] + prev2, prev1);
            prev2 = prev1;
            prev1 = curr;
        }
        
        return prev1;
    }
}
```

---

### House Robber II (LC 213)
**Problem Statement:** Same as House Robber I, but houses are arranged in a circle. This means the first house is the neighbor of the last one.
*Input:* `nums = [2, 3, 2]`  
*Output:* `3` (You cannot rob house 0 and house 2 together because they are adjacent).

#### 🧠 Intuition & Approach
Since house 0 and house `n-1` are adjacent, we cannot rob both. This splits the problem into two subproblems:
1. Rob houses from index `0` to `n-2`.
2. Rob houses from index `1` to `n-1`.

We run our linear `rob` helper on both ranges and return the maximum of the two.

```java
public class HouseRobberII {
    public int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) return nums[0];
        if (n == 2) return Math.max(nums[0], nums[1]);
        
        return Math.max(
            robLinear(nums, 0, n - 2),
            robLinear(nums, 1, n - 1)
        );
    }
    
    private int robLinear(int[] nums, int start, int end) {
        int prev2 = 0;
        int prev1 = 0;
        
        for (int i = start; i <= end; i++) {
            int curr = Math.max(nums[i] + prev2, prev1);
            prev2 = prev1;
            prev1 = curr;
        }
        
        return prev1;
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(1)$

---

## Problem 2: Partition Equal Subset Sum (Subset Sum Pattern)

### Partition Equal Subset Sum (LC 416)
**Problem Statement:** Given a non-empty array `nums` containing only positive integers, find if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.
*Input:* `nums = [1, 5, 11, 5]`  
*Output:* `true` (The array can be partitioned as `[1, 5, 5]` and `[11]`).

#### 🧠 Intuition & Approach
Let the total sum of the array be `S`.
If `S` is odd, we cannot partition it into two equal integer subsets. Return `false`.
If `S` is even, our target is to find if there exists a subset with a sum equal to `target = S / 2`.
This is a variation of the **0/1 Knapsack** problem. At each element `i`, we can either:
1. **Take the element:** The target becomes `target - nums[i]`.
2. **Exclude the element:** The target remains `target`.

#### 1. Recursive Code
```java
public class PartitionSubsetSum {
    public boolean canPartition(int[] nums) {
        int totalSum = 0;
        for (int num : nums) totalSum += num;
        if (totalSum % 2 != 0) return false;
        
        return subsetSumHelper(nums, nums.length - 1, totalSum / 2);
    }
    
    private boolean subsetSumHelper(int[] nums, int i, int target) {
        if (target == 0) return true;
        if (i < 0 || target < 0) return false;
        
        // Option 1: Take element
        boolean take = subsetSumHelper(nums, i - 1, target - nums[i]);
        // Option 2: Exclude element
        boolean exclude = subsetSumHelper(nums, i - 1, target);
        
        return take || exclude;
    }
}
```
*Time Complexity:* $O(2^N)$  
*Space Complexity:* $O(N)$ recursion stack.

#### 2. Top-Down (Memoization) Code
```java
public class PartitionSubsetSumMemo {
    public boolean canPartition(int[] nums) {
        int totalSum = 0;
        for (int num : nums) totalSum += num;
        if (totalSum % 2 != 0) return false;
        
        int target = totalSum / 2;
        // Boolean objects allow 3 states: null (uncalculated), true, false
        Boolean[][] memo = new Boolean[nums.length][target + 1];
        return subsetSumHelper(nums, nums.length - 1, target, memo);
    }
    
    private boolean subsetSumHelper(int[] nums, int i, int target, Boolean[][] memo) {
        if (target == 0) return true;
        if (i < 0 || target < 0) return false;
        if (memo[i][target] != null) return memo[i][target];
        
        boolean take = false;
        if (target >= nums[i]) {
            take = subsetSumHelper(nums, i - 1, target - nums[i], memo);
        }
        boolean exclude = subsetSumHelper(nums, i - 1, target, memo);
        
        return memo[i][target] = take || exclude;
    }
}
```
*Time Complexity:* $O(N \cdot \text{Target})$ where $\text{Target} = S/2$  
*Space Complexity:* $O(N \cdot \text{Target})$ memo table + $O(N)$ recursion stack.

#### 3. Bottom-Up (Tabulation) Code
```java
public class PartitionSubsetSumTabulation {
    public boolean canPartition(int[] nums) {
        int totalSum = 0;
        for (int num : nums) totalSum += num;
        if (totalSum % 2 != 0) return false;
        
        int target = totalSum / 2;
        int n = nums.length;
        boolean[][] dp = new boolean[n][target + 1];
        
        // Base case: Target 0 is always achievable by choosing an empty subset
        for (int i = 0; i < n; i++) {
            dp[i][0] = true;
        }
        
        // Base case: First element only
        if (nums[0] <= target) {
            dp[0][nums[0]] = true;
        }
        
        for (int i = 1; i < n; i++) {
            for (int t = 1; t <= target; t++) {
                boolean exclude = dp[i-1][t];
                boolean take = false;
                if (nums[i] <= t) {
                    take = dp[i-1][t - nums[i]];
                }
                dp[i][t] = take || exclude;
            }
        }
        
        return dp[n-1][target];
    }
}
```
*Time Complexity:* $O(N \cdot \text{Target})$  
*Space Complexity:* $O(N \cdot \text{Target})$

#### 🧠 Dry Run Table (nums = [1, 5, 11, 5], Target = 11)
Let's see if we can achieve target sums from 0 to 11. Shown for indices 0 to 3:

| Index `i` | nums[i] | Target 0 | Target 1 | Target 2 | Target 5 | Target 6 | Target 11 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | 1 | T | T | F | F | F | F |
| **1** | 5 | T | T (exclude) | F | T (take) | T (take 5 + dp[0][1]) | F |
| **2** | 11 | T | T | F | T | T | T (take 11 + dp[1][0]) |
| **3** | 5 | T | T | F | T | T | T (exclude or take 5 + dp[2][6]) |

*Result:* `dp[3][11] = true`.

#### 4. Space Optimized Code
Since each row `dp[i]` only depends on `dp[i-1]`, we can optimize this to a single 1D array of size `target + 1`. We must traverse the target values **backwards** to avoid overwriting values from the same iteration.
```java
public class PartitionSubsetSumOptimized {
    public boolean canPartition(int[] nums) {
        int totalSum = 0;
        for (int num : nums) totalSum += num;
        if (totalSum % 2 != 0) return false;
        
        int target = totalSum / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true; // Base case
        
        for (int num : nums) {
            for (int t = target; t >= num; t--) {
                dp[t] = dp[t] || dp[t - num];
            }
        }
        
        return dp[target];
    }
}
```
*Time Complexity:* $O(N \cdot \text{Target})$  
*Space Complexity:* $O(\text{Target})$

---

## Problem 3: Coin Change I & II (Unbounded Knapsack vs. 0/1 Knapsack)

### Coin Change I - Min Coins (LC 322)
**Problem Statement:** Given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money, return the fewest number of coins that you need to make up that amount. If that amount cannot be made up by any combination of the coins, return `-1`.
You may assume that you have an infinite number of each kind of coin.
*Input:* `coins = [1, 2, 5], amount = 11`  
*Output:* `3` (5 + 5 + 1 = 11)

#### 🧠 Intuition & Approach
This is an **Unbounded Knapsack** pattern because we can reuse coins infinitely.
Let `dp[i]` be the minimum coins required to make `amount = i`.
To find `dp[i]`, we look at all coins `c`. If `i - c >= 0`, we can formulate `dp[i]` as:
`dp[i] = min(dp[i], 1 + dp[i - c])`

#### 1. Recursive Code
```java
public class CoinChangeMin {
    public int coinChange(int[] coins, int amount) {
        int result = helper(coins, amount);
        return result == Integer.MAX_VALUE ? -1 : result;
    }
    
    private int helper(int[] coins, int amount) {
        if (amount == 0) return 0;
        if (amount < 0) return Integer.MAX_VALUE;
        
        int minCoins = Integer.MAX_VALUE;
        for (int coin : coins) {
            int res = helper(coins, amount - coin);
            if (res != Integer.MAX_VALUE) {
                minCoins = Math.min(minCoins, 1 + res);
            }
        }
        return minCoins;
    }
}
```
*Time Complexity:* $O(\text{Coins}^{\text{Amount}})$  
*Space Complexity:* $O(\text{Amount})$

#### 2. Top-Down (Memoization) Code
```java
import java.util.Arrays;

public class CoinChangeMinMemo {
    public int coinChange(int[] coins, int amount) {
        int[] memo = new int[amount + 1];
        Arrays.fill(memo, -2); // Unvisited flag
        int result = helper(coins, amount, memo);
        return result == Integer.MAX_VALUE ? -1 : result;
    }
    
    private int helper(int[] coins, int amount, int[] memo) {
        if (amount == 0) return 0;
        if (amount < 0) return Integer.MAX_VALUE;
        if (memo[amount] != -2) return memo[amount];
        
        int minCoins = Integer.MAX_VALUE;
        for (int coin : coins) {
            int res = helper(coins, amount - coin, memo);
            if (res != Integer.MAX_VALUE) {
                minCoins = Math.min(minCoins, 1 + res);
            }
        }
        return memo[amount] = minCoins;
    }
}
```
*Time Complexity:* $O(\text{Coins} \cdot \text{Amount})$  
*Space Complexity:* $O(\text{Amount})$

#### 3. Bottom-Up (Tabulation) Code
```java
import java.util.Arrays;

public class CoinChangeMinTabulation {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1); // Large value representing infinity
        dp[0] = 0;
        
        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (i - coin >= 0) {
                    dp[i] = Math.min(dp[i], 1 + dp[i - coin]);
                }
            }
        }
        
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```
*Time Complexity:* $O(\text{Coins} \cdot \text{Amount})$  
*Space Complexity:* $O(\text{Amount})$

#### 🧠 Dry Run Table (coins = [1, 2, 5], amount = 5)

| Amount `i` | dp[i] Formulation | Value | Explanation |
| :--- | :--- | :--- | :--- |
| **0** | Base Case | 0 | 0 coins needed to make amount 0. |
| **1** | $\min(\infty, 1+dp[0]) = 1+0$ | 1 | Take coin 1. |
| **2** | $\min(\infty, 1+dp[1], 1+dp[0]) = \min(2, 1)$ | 1 | Take coin 2. |
| **3** | $\min(\infty, 1+dp[2], 1+dp[1]) = \min(2, 2)$ | 2 | Take coin 2 + coin 1, or coin 1 + coin 1 + coin 1. |
| **4** | $\min(\infty, 1+dp[3], 1+dp[2]) = \min(3, 2)$ | 2 | Take coin 2 + coin 2. |
| **5** | $\min(\infty, 1+dp[4], 1+dp[3], 1+dp[0]) = \min(3, 3, 1)$ | 1 | Take coin 5 directly. |

---

### Coin Change II - Total Ways (LC 518)
**Problem Statement:** Return the number of combinations that make up that amount.
*Input:* `coins = [1, 2, 5], amount = 5`  
*Output:* `4`  
Combinations:
1. `5`
2. `2 + 2 + 1`
3. `2 + 1 + 1 + 1`
4. `1 + 1 + 1 + 1 + 1`

#### 🧠 Tabulation Code
Note: To avoid duplicate permutations (like `1+2` and `2+1` counted separately), we iterate through **coins first**, then amount.
```java
public class CoinChangeWays {
    public int change(int amount, int[] coins) {
        int[] dp = new int[amount + 1];
        dp[0] = 1; // 1 way to make amount 0 (choose empty set)
        
        for (int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
        }
        
        return dp[amount];
    }
}
```
*Time Complexity:* $O(\text{Coins} \cdot \text{Amount})$  
*Space Complexity:* $O(\text{Amount})$

---

## Problem 4: Longest Increasing Subsequence (LIS Pattern)

### Longest Increasing Subsequence (LC 300)
**Problem Statement:** Given an integer array `nums`, return the length of the longest strictly increasing subsequence.
*Input:* `nums = [10, 9, 2, 5, 3, 7, 101, 18]`  
*Output:* `4` (The LIS is `[2, 3, 7, 101]` or `[2, 3, 7, 18]`).

#### 🧠 Intuition & Approach
For every index `i`, we want to find the LIS ending at `i`.
`dp[i]` is defined as the length of the LIS ending at index `i`.
To compute `dp[i]`, we scan all previous indices `j` (where `0 <= j < i`). If `nums[i] > nums[j]`, then we can extend the LIS ending at `j`.
`dp[i] = max(dp[i], 1 + dp[j])`

#### 1. Recursive Code
At each element, we decide to either include it in our LIS (if it's greater than the previously chosen element) or exclude it.
```java
public class LISRecursive {
    public int lengthOfLIS(int[] nums) {
        return helper(nums, 0, -1);
    }
    
    private int helper(int[] nums, int idx, int prevIdx) {
        if (idx == nums.length) return 0;
        
        // Choice 1: Exclude current element
        int exclude = helper(nums, idx + 1, prevIdx);
        
        // Choice 2: Include current element (if valid)
        int include = 0;
        if (prevIdx == -1 || nums[idx] > nums[prevIdx]) {
            include = 1 + helper(nums, idx + 1, idx);
        }
        
        return Math.max(include, exclude);
    }
}
```
*Time Complexity:* $O(2^N)$  
*Space Complexity:* $O(N)$ recursion stack.

#### 2. Top-Down (Memoization) Code
We use a 2D memo table of size `N * (N+1)` because the previous index ranges from `-1` to `N-1`. We offset it by `1` to avoid negative index errors.
```java
import java.util.Arrays;

public class LISMemo {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[][] memo = new int[n][n + 1];
        for (int[] row : memo) Arrays.fill(row, -1);
        return helper(nums, 0, -1, memo);
    }
    
    private int helper(int[] nums, int idx, int prevIdx, int[][] memo) {
        if (idx == nums.length) return 0;
        if (memo[idx][prevIdx + 1] != -1) return memo[idx][prevIdx + 1];
        
        int exclude = helper(nums, idx + 1, prevIdx, memo);
        int include = 0;
        if (prevIdx == -1 || nums[idx] > nums[prevIdx]) {
            include = 1 + helper(nums, idx + 1, idx, memo);
        }
        
        return memo[idx][prevIdx + 1] = Math.max(include, exclude);
    }
}
```
*Time Complexity:* $O(N^2)$  
*Space Complexity:* $O(N^2)$

#### 3. Bottom-Up (Tabulation) Code
```java
import java.util.Arrays;

public class LISTabulation {
    public int lengthOfLIS(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1); // Each element itself is an LIS of length 1
        
        int maxLIS = 1;
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[i], 1 + dp[j]);
                }
            }
            maxLIS = Math.max(maxLIS, dp[i]);
        }
        
        return maxLIS;
    }
}
```
*Time Complexity:* $O(N^2)$  
*Space Complexity:* $O(N)$

#### 🧠 Dry Run Table (nums = [10, 9, 2, 5, 3, 7])

| Index `i` | nums[i] | Checking `j` from 0 to `i-1` | dp[i] updates | Max LIS Ending here |
| :--- | :--- | :--- | :--- | :--- |
| **0** | 10 | — | Base Case | 1 (`[10]`) |
| **1** | 9 | $nums[1] > nums[0]$? No ($9 > 10$ is False) | dp[1] remains 1 | 1 (`[9]`) |
| **2** | 2 | No smaller element before it | dp[2] remains 1 | 1 (`[2]`) |
| **3** | 5 | $5 > 10$ (F), $5 > 9$ (F), $5 > 2$ (T) | $\max(1, 1 + dp[2]) = 2$ | 2 (`[2, 5]`) |
| **4** | 3 | $3 > 2$ (T), others false | $\max(1, 1 + dp[2]) = 2$ | 2 (`[2, 3]`) |
| **5** | 7 | $7 > 2$ (T), $7 > 5$ (T), $7 > 3$ (T) | $\max(1, 1+dp[2], 1+dp[3], 1+dp[4]) = \max(1, 2, 3, 3) = 3$ | 3 (`[2, 5, 7]` or `[2, 3, 7]`) |

#### 4. Expert Level: Binary Search ($O(N \log N)$)
If the interviewer demands $O(N \log N)$, we can maintain an active list of elements that represent the smallest end elements of all active increasing subsequences of various lengths.
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class LISBinarySearch {
    public int lengthOfLIS(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        List<Integer> tails = new ArrayList<>();
        for (int num : nums) {
            int idx = Collections.binarySearch(tails, num);
            if (idx < 0) {
                idx = -(idx + 1); // Get insertion point
            }
            if (idx == tails.size()) {
                tails.add(num);
            } else {
                tails.set(idx, num);
            }
        }
        return tails.size();
    }
}
```
*Time Complexity:* $O(N \log N)$  
*Space Complexity:* $O(N)$

---

## Problem 5: Best Time to Buy and Sell Stock with Cooldown & Transaction Fee

### Stock with Cooldown (LC 309)
**Problem Statement:** You are given an array `prices` where `prices[i]` is the price of a given stock on the $i$-th day.
Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following restrictions:
*   After you sell your stock, you cannot buy stock on the next day (i.e., cooldown 1 day).
*   You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

#### 🧠 State Space Analysis
We define 3 states for each day `i`:
1.  `hold[i]`: Maximum profit on day `i` if we hold a stock.
    - We either bought a stock today (meaning we were in a `cooldown` state yesterday: `cooldown[i-1] - prices[i]`).
    - Or we already held stock from yesterday: `hold[i-1]`.
    `hold[i] = max(hold[i-1], cooldown[i-1] - prices[i])`
2.  `sell[i]`: Maximum profit on day `i` if we sold a stock today.
    - We must have held stock yesterday and sold it today: `hold[i-1] + prices[i]`.
    `sell[i] = hold[i-1] + prices[i]`
3.  `cooldown[i]`: Maximum profit on day `i` if we are in a cooldown (unheld state).
    - We either sold stock yesterday: `sell[i-1]`.
    - Or we were already in cooldown state yesterday: `cooldown[i-1]`.
    `cooldown[i] = max(cooldown[i-1], sell[i-1])`

#### 1. Java Tabulation Code
```java
public class StockCooldown {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length < 2) return 0;
        
        int n = prices.length;
        int[] hold = new int[n];
        int[] sell = new int[n];
        int[] cooldown = new int[n];
        
        hold[0] = -prices[0];
        sell[0] = 0;
        cooldown[0] = 0;
        
        for (int i = 1; i < n; i++) {
            hold[i] = Math.max(hold[i-1], cooldown[i-1] - prices[i]);
            sell[i] = hold[i-1] + prices[i];
            cooldown[i] = Math.max(cooldown[i-1], sell[i-1]);
        }
        
        return Math.max(sell[n-1], cooldown[n-1]);
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(N)$

#### 🧠 Dry Run Table (prices = [1, 2, 3, 0, 2])

| Day `i` | Price | hold[i] | sell[i] | cooldown[i] | Explanation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | 1 | -1 | 0 | 0 | Base cases |
| **1** | 2 | $\max(-1, 0-2) = -1$ | $-1+2 = 1$ | $\max(0, 0) = 0$ | Buy at 1 or do nothing. |
| **2** | 3 | $\max(-1, 0-3) = -1$ | $-1+3 = 2$ | $\max(0, 1) = 1$ | Hold stock or sell at 3. |
| **3** | 0 | $\max(-1, 1-0) = 1$ | $-1+0 = -1$ | $\max(1, 2) = 2$ | Buy at 0 because cooldown[2] was 1. |
| **4** | 2 | $\max(1, 2-2) = 1$ | $1+2 = 3$ | $\max(2, -1) = 2$ | Sell bought stock at 2. Total profit = 3. |

#### 2. Space Optimized Code
Since we only depend on day `i-1` states, we can optimize space to $O(1)$.
```java
public class StockCooldownOptimized {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length < 2) return 0;
        
        int hold = -prices[0];
        int sell = 0;
        int cooldown = 0;
        
        for (int i = 1; i < prices.length; i++) {
            int nextHold = Math.max(hold, cooldown - prices[i]);
            int nextSell = hold + prices[i];
            int nextCooldown = Math.max(cooldown, sell);
            
            hold = nextHold;
            sell = nextSell;
            cooldown = nextCooldown;
        }
        
        return Math.max(sell, cooldown);
    }
}
```

---

### Stock with Transaction Fee (LC 714)
**Problem Statement:** Same as stock trading, but you pay a flat transaction `fee` for each transaction (applied on either buy or sell).
`hold[i] = max(hold[i-1], free[i-1] - prices[i])`  
`free[i] = max(free[i-1], hold[i-1] + prices[i] - fee)`

```java
public class StockTransactionFee {
    public int maxProfit(int[] prices, int fee) {
        if (prices == null || prices.length < 2) return 0;
        
        int hold = -prices[0];
        int free = 0;
        
        for (int i = 1; i < prices.length; i++) {
            int nextHold = Math.max(hold, free - prices[i]);
            int nextFree = Math.max(free, hold + prices[i] - fee);
            hold = nextHold;
            free = nextFree;
        }
        
        return free;
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(1)$

---

## Problem 6: Maximum Product Subarray (Min/Max Tracking Pattern)

### Maximum Product Subarray (LC 152)
**Problem Statement:** Given an integer array `nums`, find a contiguous non-empty subarray within the array that has the largest product, and return the product.
*Input:* `nums = [2, 3, -2, 4]`  
*Output:* `6` (Subarray `[2, 3]` has product 6).

#### 🧠 Intuition & Approach
Unlike Maximum Sum Subarray (Kadane's algorithm), products can turn from extremely negative to extremely positive when multiplied by a negative number.
Therefore, at each index `i`, we must track:
1.  `maxProduct[i]`: Maximum product of a subarray ending at `i`.
2.  `minProduct[i]`: Minimum product of a subarray ending at `i`.

If `nums[i]` is negative, multiplying it by `minProduct[i-1]` could yield the new maximum.
Transitions:
`currMax = max(nums[i], nums[i] * prevMax, nums[i] * prevMin)`  
`currMin = min(nums[i], nums[i] * prevMax, nums[i] * prevMin)`

#### 1. Complete Java Code
```java
public class MaxProductSubarray {
    public int maxProduct(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        int maxSoFar = nums[0];
        int minSoFar = nums[0];
        int globalMax = nums[0];
        
        for (int i = 1; i < nums.length; i++) {
            int curr = nums[i];
            
            // Temporary store to prevent overwriting maxSoFar before calculating minSoFar
            int tempMax = Math.max(curr, Math.max(curr * maxSoFar, curr * minSoFar));
            minSoFar = Math.min(curr, Math.min(curr * maxSoFar, curr * minSoFar));
            maxSoFar = tempMax;
            
            globalMax = Math.max(globalMax, maxSoFar);
        }
        
        return globalMax;
    }
}
```
*Time Complexity:* $O(N)$  
*Space Complexity:* $O(1)$

#### 🧠 Dry Run Table (nums = [2, 3, -2, 4, -1])

| Index `i` | nums[i] | Candidate 1 (`curr`) | Candidate 2 (`curr * maxSoFar`) | Candidate 3 (`curr * minSoFar`) | maxSoFar | minSoFar | globalMax |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | 2 | — | — | — | **2** | **2** | **2** |
| **1** | 3 | 3 | $3 \times 2 = 6$ | $3 \times 2 = 6$ | **6** | **3** | **6** |
| **2** | -2 | -2 | $-2 \times 6 = -12$ | $-2 \times 3 = -6$ | **-2** | **-12** | **6** |
| **3** | 4 | 4 | $4 \times (-2) = -8$ | $4 \times (-12) = -48$ | **4** | **-48** | **6** |
| **4** | -1 | -1 | $-1 \times 4 = -4$ | $-1 \times (-48) = 48$ | **48** | **-4** | **48** |

*Final Maximum Product:* `48` (for subarray `[2, 3, -2, 4, -1]`).
