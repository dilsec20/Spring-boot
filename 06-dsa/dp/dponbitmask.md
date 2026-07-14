# 🧠 Dynamic Programming with Bitmasking — Beginner to Expert Guide

Bitmask DP is used when the subproblems depend on a subset of items that have been visited or chosen. In these problems, $N$ is typically small ($N \le 20$) because the number of possible subsets is $2^N$. We represent the subset using a binary integer (the "mask").

---

## 📑 Table of Contents
1. [Bitwise Operation Foundations](#bitwise-operation-foundations)
2. [Problem 1: The Assignment Problem (Job Allocation)](#problem-1-the-assignment-problem-job-allocation)
3. [Problem 2: Travelling Salesman Problem (TSP)](#problem-2-travelling-salesman-problem-tsp)
4. [Problem 3: Smallest Sufficient Team (LC 1125)](#problem-3-smallest-sufficient-team-lc-1125)
5. [Problem 4: Matchsticks to Square (LC 473)](#problem-4-matchsticks-to-square-lc-473)

---

## Bitwise Operation Foundations

A bitmask is simply an integer where the binary bits represent the inclusion/exclusion of elements:
- `1` at the $i$-th bit: Element $i$ is **included** in the subset.
- `0` at the $i$-th bit: Element $i$ is **excluded** from the subset.

### Essential Bit Operations in Java

| Operation | Java Syntax | Explanation |
| :--- | :--- | :--- |
| **Check if $i$-th bit is set** | `(mask & (1 << i)) != 0` | Returns `true` if element $i$ is in the subset. |
| **Set $i$-th bit (add to subset)** | `mask | (1 << i)` | Sets the $i$-th bit to `1`. |
| **Clear $i$-th bit (remove from subset)** | `mask & ~(1 << i)` | Sets the $i$-th bit to `0`. |
| **Toggle $i$-th bit** | `mask ^ (1 << i)` | Flips the bit from `0` to `1` or `1` to `0`. |
| **Count set bits** | `Integer.bitCount(mask)` | Returns the size of the active subset. |
| **Full Set Mask (Size $N$)** | `(1 << N) - 1` | Generates a binary mask of $N$ ones. |

---

## Problem 1: The Assignment Problem (Job Allocation)

**Problem Statement:** There are $N$ persons and $N$ jobs. Each person has a cost to complete each job, given as a cost matrix `cost[N][N]`. Assign exactly one job to each person to minimize the total cost.
*Input:*
```
cost = [
  [9, 2, 7],
  [6, 4, 3],
  [5, 8, 1]
]
```
*Output:* `13`  
Explanation:
- Person 0 gets Job 1 (cost 2)
- Person 1 gets Job 0 (cost 6)
- Person 2 gets Job 2 (cost 1)
Total = 2 + 6 + 1 = 9. Wait! Let's check:
If Person 0 gets Job 1 (2), Person 1 gets Job 0 (6), Person 2 gets Job 2 (1), sum is 9. That is indeed minimal!

#### 🧠 State Space Mapping
Let `dp[i][mask]` be the minimum cost to assign jobs to persons from index `i` to $N-1$, where `mask` is a bitmask of size $N$ representing which jobs have already been assigned.
Actually, the person index `i` is always equal to the number of set bits in `mask`!
Thus, we can represent our DP array using a single 1D array of size $2^N$:
`dp[mask]` = minimum cost to assign the first `countSetBits(mask)` persons to the jobs represented by `mask`.

#### 1. Recursive Code with Memoization
```java
import java.util.Arrays;

public class AssignmentProblem {
    public int minCost(int[][] cost) {
        int n = cost.length;
        int[] dp = new int[1 << n];
        Arrays.fill(dp, -1);
        return solve(0, 0, cost, dp);
    }
    
    private int solve(int person, int mask, int[][] cost, int[] dp) {
        int n = cost.length;
        if (person == n) return 0; // All persons assigned
        if (dp[mask] != -1) return dp[mask];
        
        int minCost = Integer.MAX_VALUE;
        for (int job = 0; job < n; job++) {
            // Check if job is not yet assigned
            if ((mask & (1 << job)) == 0) {
                int currCost = cost[person][job] + solve(person + 1, mask | (1 << job), cost, dp);
                minCost = Math.min(minCost, currCost);
            }
        }
        
        return dp[mask] = minCost;
    }
}
```
*Time Complexity:* $O(N \cdot 2^N)$  
*Space Complexity:* $O(2^N)$

#### 2. Bottom-Up (Tabulation) Code
```java
import java.util.Arrays;

public class AssignmentTabulation {
    public int minCost(int[][] cost) {
        int n = cost.length;
        int limit = 1 << n;
        int[] dp = new int[limit];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0; // Base case: cost for empty subset is 0
        
        for (int mask = 0; mask < limit; mask++) {
            if (dp[mask] == Integer.MAX_VALUE) continue;
            
            // Person to assign is determined by how many jobs are already taken
            int person = Integer.bitCount(mask);
            if (person == n) break; 
            
            for (int job = 0; job < n; job++) {
                if ((mask & (1 << job)) == 0) { // If job is free
                    int nextMask = mask | (1 << job);
                    dp[nextMask] = Math.min(dp[nextMask], dp[mask] + cost[person][job]);
                }
            }
        }
        
        return dp[limit - 1];
    }
}
```
*Time Complexity:* $O(N \cdot 2^N)$  
*Space Complexity:* $O(2^N)$

#### 🧠 Dry Run (cost matrix of size 3x3)
Let's see how `dp[mask]` is filled from `mask = 000` to `111` in binary.

| Mask (Binary) | person | Job candidate | dp[nextMask] update | Final dp[mask] |
| :--- | :--- | :--- | :--- | :--- |
| **000** | 0 | Job 0, 1, 2 | dp[001]=9, dp[010]=2, dp[100]=7 | **0** |
| **001** (Job 0 taken) | 1 | Job 1, 2 | dp[011] = min(inf, 9+cost[1][1]=13) | **9** |
| **010** (Job 1 taken) | 1 | Job 0, 2 | dp[011] = min(13, 2+cost[1][0]=8), dp[110]=2+3=5 | **2** |
| **100** (Job 2 taken) | 1 | Job 0, 1 | dp[101] = 7+6=13, dp[110] = min(5, 7+4=11) | **7** |
| **011** (Jobs 0,1 taken) | 2 | Job 2 | dp[111] = min(inf, 8+cost[2][2]=9) | **8** |
| **110** (Jobs 1,2 taken) | 2 | Job 0 | dp[111] = min(9, 5+cost[2][0]=10) | **5** |
| **101** (Jobs 0,2 taken) | 2 | Job 1 | dp[111] = min(9, 13+8=21) | **13** |
| **111** (All taken) | 3 | — | Target Reached! | **9** |

*Final Minimum Cost:* `9`.

---

## Problem 2: Travelling Salesman Problem (TSP)

**Problem Statement:** Given a list of cities and the distances between each pair of cities, what is the shortest possible route that visits each city exactly once and returns to the origin city?
*Input:* Distance matrix `dist[N][N]`  
*Output:* Shortest cycle distance.

#### 🧠 State Space Mapping
Let `dp[u][mask]` be the shortest path starting at city `u` that visits all remaining cities in `mask` and then returns to the origin (city 0).
- `u` ranges from `0` to `N-1`.
- `mask` represents the subset of cities already visited.

#### Java Code
```java
import java.util.Arrays;

public class TSP {
    private int[][] dp;
    private int[][] dist;
    private int n;

    public int solveTSP(int[][] dist) {
        this.dist = dist;
        this.n = dist.length;
        this.dp = new int[n][1 << n];
        for (int[] row : dp) {
            Arrays.fill(row, -1);
        }
        // Start at city 0, with only city 0 visited (mask = 1)
        return tspHelper(0, 1);
    }

    private int tspHelper(int u, int mask) {
        // Base case: All cities have been visited (mask contains all 1s)
        if (mask == (1 << n) - 1) {
            return dist[u][0]; // Return to starting city 0
        }

        if (dp[u][mask] != -1) {
            return dp[u][mask];
        }

        int minCost = Integer.MAX_VALUE / 2; // Prevent integer overflow
        for (int v = 0; v < n; v++) {
            // If city v is not yet visited
            if ((mask & (1 << v)) == 0) {
                int cost = dist[u][v] + tspHelper(v, mask | (1 << v));
                minCost = Math.min(minCost, cost);
            }
        }

        return dp[u][mask] = minCost;
    }
}
```
*Time Complexity:* $O(N^2 \cdot 2^N)$  
*Space Complexity:* $O(N \cdot 2^N)$

---

## Problem 3: Smallest Sufficient Team (LC 1125)

**Problem Statement:** In a project, you have a list of required skills `req_skills`, and a list of people. The $i$-th person `people[i]` contains a list of skills that person has.
Return a list of people of the smallest possible size such that the union of their skills contains all required skills in `req_skills`.
*Input:* `req_skills = ["java","nodejs","reactjs"]`, `people = [["java"],["nodejs"],["nodejs","reactjs"]]`  
*Output:* `[0, 2]` (Person 0 and Person 2 cover all required skills).

#### 🧠 Intuition & Approach
We map each skill to a bit index from `0` to `S-1`.
A person's skill set can be represented as an integer `skillsMask`.
Let `dp[mask]` be the list of people indices that covers the subset of skills represented by `mask`. We want to reach `dp[(1 << S) - 1]`.

#### Java Code
```java
import java.util.*;

public class SufficientTeam {
    public int[] smallestSufficientTeam(String[] req_skills, List<List<String>> people) {
        int s = req_skills.length;
        int p = people.size();
        
        // Map skills to bit positions
        Map<String, Integer> skillToBit = new HashMap<>();
        for (int i = 0; i < s; i++) {
            skillToBit.put(req_skills[i], i);
        }
        
        // Map people to their skill mask
        int[] peopleMasks = new int[p];
        for (int i = 0; i < p; i++) {
            int mask = 0;
            for (String skill : people.get(i)) {
                if (skillToBit.containsKey(skill)) {
                    mask |= (1 << skillToBit.get(skill));
                }
            }
            peopleMasks[i] = mask;
        }
        
        // dp[mask] stores the list of person indices that cover the skill mask
        List<Integer>[] dp = new List[1 << s];
        dp[0] = new ArrayList<>(); // Base case: 0 skills covered by 0 people
        
        for (int i = 0; i < p; i++) {
            int pMask = peopleMasks[i];
            if (pMask == 0) continue; // Person has no relevant skills
            
            // Loop through all existing skill sets and check if adding person i improves the team
            for (int mask = 0; mask < (1 << s); mask++) {
                if (dp[mask] == null) continue;
                
                int combinedMask = mask | pMask;
                if (combinedMask == mask) continue; // No new skill gained
                
                if (dp[combinedMask] == null || dp[mask].size() + 1 < dp[combinedMask].size()) {
                    List<Integer> newTeam = new ArrayList<>(dp[mask]);
                    newTeam.add(i);
                    dp[combinedMask] = newTeam;
                }
            }
        }
        
        List<Integer> resultTeam = dp[(1 << s) - 1];
        int[] res = new int[resultTeam.size()];
        for (int i = 0; i < res.length; i++) {
            res[i] = resultTeam.get(i);
        }
        return res;
    }
}
```
*Time Complexity:* $O(\text{People} \cdot 2^{\text{Skills}})$  
*Space Complexity:* $O(2^{\text{Skills}})$

---

## Problem 4: Matchsticks to Square (LC 473)

**Problem Statement:** You are given an integer array `matchsticks` where `matchsticks[i]` is the length of the $i$-th matchstick. You want to use all the matchsticks to form one square. You cannot break any stick, but you can link them up, and each matchstick must be used exactly once.
Return `true` if you can make this square.

#### 🧠 Intuition & Approach
Let the total perimeter be `P`. If `P % 4 != 0`, return `false`. The target side length is `side = P / 4`.
We want to partition the array of size $N$ into 4 subsets, each with sum equal to `side`.
Using bitmasking, let `dp[mask]` be the remaining length of the current side we are building after using the matchsticks represented by `mask`.
- If `dp[mask]` is `-1`, then this mask state is invalid.
- If we successfully place a stick and it fits on the current side, the new remaining space is `(dp[mask] + stick) % side`.

#### Java Code
```java
import java.util.Arrays;

public class MatchsticksSquare {
    public boolean makesquare(int[] matchsticks) {
        if (matchsticks == null || matchsticks.length < 4) return false;
        
        int sum = 0;
        for (int stick : matchsticks) sum += stick;
        if (sum % 4 != 0) return false;
        
        int side = sum / 4;
        int n = matchsticks.length;
        int[] dp = new int[1 << n];
        Arrays.fill(dp, -1);
        dp[0] = 0; // Base Case: 0 sticks used means 0 length filled
        
        for (int mask = 0; mask < (1 << n); mask++) {
            if (dp[mask] == -1) continue;
            
            for (int i = 0; i < n; i++) {
                // If stick i is not yet used
                if ((mask & (1 << i)) == 0) {
                    if (dp[mask] + matchsticks[i] <= side) {
                        int nextMask = mask | (1 << i);
                        // Store the cumulative modular sum
                        dp[nextMask] = (dp[mask] + matchsticks[i]) % side;
                    }
                }
            }
        }
        
        return dp[(1 << n) - 1] == 0;
    }
}
```
*Time Complexity:* $O(N \cdot 2^N)$  
*Space Complexity:* $O(2^N)$
