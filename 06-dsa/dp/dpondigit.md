# 🧠 Digit Dynamic Programming (Digit DP) — Beginner to Expert Guide

Digit DP is a powerful dynamic programming pattern used to solve counting problems involving digits of numbers in a range $[L, R]$. Typically, the range is extremely large ($R \le 10^{18}$), making brute-force enumeration impossible.

---

## 📑 Table of Contents
1. [Core Concepts of Digit DP](#core-concepts-of-digit-dp)
2. [The Standard Java Template](#the-standard-java-template)
3. [Problem 1: Count of Numbers with Given Digit Sum in Range](#problem-1-count-of-numbers-with-given-digit-sum-in-range)
4. [Problem 2: Non-negative Integers without Consecutive Ones (LC 600)](#problem-2-non-negative-integers-without-consecutive-ones-lc-600)
5. [Problem 3: Numbers with Repeated Digits (LC 1012)](#problem-3-numbers-with-repeated-digits-lc-1012)

---

## Core Concepts of Digit DP

When counting numbers in a range $[L, R]$ matching a certain property (e.g. digit sum is prime), we solve the problem for $[0, R]$ and $[0, L-1]$ separately. The final answer is:
$$\text{Solve}(R) - \text{Solve}(L - 1)$$

### The `tight` Constraint Flag
When building a number digit-by-digit from left to right, we must ensure the generated number does not exceed $R$.
Suppose $R = 345$.
1.  **If we choose the first digit to be `3`:** The second digit cannot exceed `4`. The search remains **restricted** (`tight = 1`).
2.  **If we choose the first digit to be `1` or `2`:** The second digit can be anything from `0` to `9`. The search becomes **unrestricted** (`tight = 0`).

Once the constraint is broken (`tight = 0`), it remains `0` for all subsequent positions.

### The `leading_zero` Flag
Some problems require tracking if we have placed any non-zero digit yet. For example, counting unique digits: leading zeros should not be counted as repeated digits (e.g., `005` represents the number `5` and has only 1 distinct digit).
- `leading_zero = 1` if no non-zero digit has been placed yet.
- `leading_zero = 0` as soon as we place a non-zero digit.

---

## The Standard Java Template

Here is the standard recursive method layout for Digit DP:

```java
import java.util.Arrays;

public class DigitDPTemplate {
    // 4D Memo array: dp[index][tight][leading_zero][state_variable]
    private int[][][][] dp;
    private String numStr;

    public int solve(int R) {
        this.numStr = String.valueOf(R);
        int n = numStr.length();
        
        // Define DP dimensions based on your state needs
        dp = new int[n][2][2][100]; 
        for (int[][][] d3 : dp) {
            for (int[][] d2 : d3) {
                for (int[] d1 : d2) {
                    Arrays.fill(d1, -1);
                }
            }
        }
        return solveRecursive(0, 1, 1, 0);
    }

    private int solveRecursive(int idx, int tight, int leadingZero, int stateVar) {
        // Base case: Reached the end of the number
        if (idx == numStr.length()) {
            return isValid(stateVar) ? 1 : 0; 
        }

        if (dp[idx][tight][leadingZero][stateVar] != -1) {
            return dp[idx][tight][leadingZero][stateVar];
        }

        int limit = (tight == 1) ? (numStr.charAt(idx) - '0') : 9;
        int count = 0;

        for (int digit = 0; digit <= limit; digit++) {
            int newTight = (tight == 1 && digit == limit) ? 1 : 0;
            int newLeadingZero = (leadingZero == 1 && digit == 0) ? 1 : 0;
            int newStateVar = updateState(stateVar, digit, newLeadingZero);

            count += solveRecursive(idx + 1, newTight, newLeadingZero, newStateVar);
        }

        return dp[idx][tight][leadingZero][stateVar] = count;
    }

    private boolean isValid(int stateVar) {
        // Define condition
        return true; 
    }

    private int updateState(int stateVar, int digit, int newLeadingZero) {
        // Update condition state
        return stateVar + digit;
    }
}
```

---

## Problem 1: Count of Numbers with Given Digit Sum in Range

**Problem Statement:** Given two integers $L$ and $R$, find the number of integers in the range $[L, R]$ whose sum of digits is equal to a given sum $S$.
*Input:* $L = 10, R = 30, S = 3$  
*Output:* $3$ (The numbers are 12, 21, 30).

#### 🧠 State Space Configuration
`dp[idx][tight][sum]` represents the count of valid suffixes starting at index `idx` given the current `tight` state and remaining `sum` needed.

#### Java Code
```java
import java.util.Arrays;

public class DigitSumRange {
    private int[][][] dp;
    private String numStr;

    public int countWithSum(int L, int R, int S) {
        return solve(R, S) - solve(L - 1, S);
    }

    private int solve(int N, int S) {
        if (N < 0) return 0;
        this.numStr = String.valueOf(N);
        int len = numStr.length();
        
        // Sum can be at most 9 * 18 = 162 for long integers, we size up to S
        dp = new int[len][2][S + 1];
        for (int[][] d2 : dp) {
            for (int[] d1 : d2) {
                Arrays.fill(d1, -1);
            }
        }
        return countPaths(0, 1, S);
    }

    private int countPaths(int idx, int tight, int remainingSum) {
        if (remainingSum < 0) return 0;
        if (idx == numStr.length()) {
            return (remainingSum == 0) ? 1 : 0;
        }

        if (dp[idx][tight][remainingSum] != -1) {
            return dp[idx][tight][remainingSum];
        }

        int limit = (tight == 1) ? (numStr.charAt(idx) - '0') : 9;
        int ans = 0;

        for (int digit = 0; digit <= limit; digit++) {
            int newTight = (tight == 1 && digit == limit) ? 1 : 0;
            ans += countPaths(idx + 1, newTight, remainingSum - digit);
        }

        return dp[idx][tight][remainingSum] = ans;
    }
}
```
*Time Complexity:* $O(\text{Digits} \cdot 2 \cdot S) = O(\log_{10}(R) \cdot S)$  
*Space Complexity:* $O(\log_{10}(R) \cdot S)$

#### 🧠 Dry Run Simulation (N = 25, S = 3)
Let's see how the recursions expand starting from `(0, tight=1, sum=3)`:
- Choose digit `0`: `(1, tight=0, sum=3)`
  - Choose digit `0`: `(2, tight=0, sum=3)` -> Sum 3 remaining at index 2 (Not possible, returns 0)
  - Choose digit `1`: `(2, tight=0, sum=2)` -> Digit 2 (Returns 1 because remaining sum becomes 0, i.e., number `012` = 12)
  - Choose digit `2`: `(2, tight=0, sum=1)` -> Digit 1 (Returns 1, i.e., number `021` = 21)
  - Choose digit `3`: `(2, tight=0, sum=0)` -> Digit 0 (Returns 1, i.e., number `030` = 30 - but wait, 30 > 25, is this pruned? No, `tight` became 0 when we chose `0` at index 0. However, the number we generated is `030`, wait. Let's trace it.
    If we chose digit `0` at index 0 (which is less than limit `2`), `tight` became `0`.
    At index 1, we can choose `0` to `9`. We chose `3`.
    At index 2, we must choose `0` to complete the length. The generated number is `030` which is indeed 30.
    Wait, why did we generate 30 if $N=25$?
    Ah, the number length of 25 is 2. So the indices are 0 and 1.
    Let's re-trace with index length = 2:
    - At index 0: Limit is `2` (since tight=1).
      - Option A: Digit `0` (tight becomes 0).
        - Index 1: Limit is `9`. Choose digit `3`. Reached end index 2 with remainingSum = 0. Valid number `03` = 3.
      - Option B: Digit `1` (tight becomes 0).
        - Index 1: Limit is `9`. Choose digit `2`. Reached end index 2 with remainingSum = 0. Valid number `12`.
      - Option C: Digit `2` (tight remains 1).
        - Index 1: Limit is `5`. We can choose digits `0` to `5`.
          - Choose `1`: Reached end with remainingSum = 0. Valid number `21`.
          - All other choices fail sum.

Total count: `3` (3, 12, 21). This is correct!

---

## Problem 2: Non-negative Integers without Consecutive Ones (LC 600)

**Problem Statement:** Given a positive integer $n$, return the number of the non-negative integers less than or equal to $n$, whose binary representations do not contain consecutive ones.
*Input:* $n = 5$ (Binary: `101`)  
*Output:* $5$  
Explanation:
- `0` (binary `0`)
- `1` (binary `1`)
- `2` (binary `10`)
- `3` (binary `11`) - Invalid
- `4` (binary `100`)
- `5` (binary `101`)
Count is 5.

#### 🧠 Intuition & Approach
Since this is binary, we convert $N$ to a binary string.
The state requires:
1.  `idx`: Current bit index.
2.  `tight`: Constraint flag.
3.  `prevBit`: The value of the bit we placed in the previous step (must not place a `1` if `prevBit == 1`).

#### Complete Java Code
```java
import java.util.Arrays;

public class ConsecutiveOnes {
    public int findIntegers(int n) {
        String binaryStr = Integer.toBinaryString(n);
        int len = binaryStr.length();
        int[][][] dp = new int[len][2][2];
        for (int[][] row : dp) {
            for (int[] cell : row) {
                Arrays.fill(cell, -1);
            }
        }
        return solve(0, 1, 0, binaryStr, dp);
    }

    private int solve(int idx, int tight, int prevBit, String binaryStr, int[][][] dp) {
        if (idx == binaryStr.length()) {
            return 1; // Found a valid binary number
        }

        if (dp[idx][tight][prevBit] != -1) {
            return dp[idx][tight][prevBit];
        }

        int limit = (tight == 1) ? (binaryStr.charAt(idx) - '0') : 1;
        int count = 0;

        for (int bit = 0; bit <= limit; bit++) {
            // Constraint: No consecutive ones
            if (bit == 1 && prevBit == 1) continue;

            int newTight = (tight == 1 && bit == limit) ? 1 : 0;
            count += solve(idx + 1, newTight, bit, binaryStr, dp);
        }

        return dp[idx][tight][prevBit] = count;
    }
}
```
*Time Complexity:* $O(\log_2(N))$ — since binary representation has $\log_2(N)$ bits, and states are small ($N_{\text{states}} = \log_2(N) \times 2 \times 2$).  
*Space Complexity:* $O(\log_2(N))$

---

## Problem 3: Numbers with Repeated Digits (LC 1012)

**Problem Statement:** Given an integer $n$, return the number of positive integers in the range $[1, n]$ that have at least one repeated digit.
*Input:* $n = 20$  
*Output:* $1$ (Only 11 has a repeated digit).

#### 🧠 Intuition & Approach
It is easier to calculate the **complement**: the count of numbers in $[1, n]$ that have **no repeated digits**.
Then, the final answer is:
$$\text{Repeated} = n - \text{NoRepeated}$$

To track repeating digits, we use a **bitmask** of size 10 (since there are 10 unique digits, 0-9).
- If digit $d$ is used, set the $d$-th bit in the mask.
- If the $d$-th bit is already set, we cannot use digit $d$ because that would cause a repetition.

#### Complete Java Code
```java
import java.util.Arrays;

public class RepeatedDigits {
    private int[][] dp;
    private String numStr;

    public int numDupDigitsAtMostN(int n) {
        this.numStr = String.valueOf(n);
        int len = numStr.length();
        
        // dp[idx][mask] stores count of suffix strings with distinct digits
        // We handle tight and leadingZero explicitly in transitions or separate dimensions.
        // To be safe, we dimension dp as dp[len][1 << 10]
        dp = new int[len][1 << 10];
        for (int[] row : dp) {
            Arrays.fill(row, -1);
        }

        int noRepeated = solve(0, 1, 1, 0);
        return n - (noRepeated - 1); // Subtract 1 to exclude the number 0
    }

    private int solve(int idx, int tight, int leadingZero, int mask) {
        if (idx == numStr.length()) {
            return 1; // Valid sequence completed
        }

        // We only memoize if the state is unrestricted and contains no leading zeroes.
        // This ensures the mask is universally caching valid digit choices.
        if (tight == 0 && leadingZero == 0 && dp[idx][mask] != -1) {
            return dp[idx][mask];
        }

        int limit = (tight == 1) ? (numStr.charAt(idx) - '0') : 9;
        int count = 0;

        for (int digit = 0; digit <= limit; digit++) {
            int newTight = (tight == 1 && digit == limit) ? 1 : 0;
            int newLeadingZero = (leadingZero == 1 && digit == 0) ? 1 : 0;

            if (newLeadingZero == 1) {
                // If it is still a leading zero, we don't set the bit in the mask
                count += solve(idx + 1, newTight, 1, mask);
            } else {
                // Check if digit has already been used
                if ((mask & (1 << digit)) != 0) continue; // Skip to prevent repetition
                
                count += solve(idx + 1, newTight, 0, mask | (1 << digit));
            }
        }

        if (tight == 0 && leadingZero == 0) {
            dp[idx][mask] = count;
        }

        return count;
    }
}
```
*Time Complexity:* $O(\log_{10}(N) \cdot 2^{10} \cdot 10)$  
*Space Complexity:* $O(\log_{10}(N) \cdot 2^{10})$
