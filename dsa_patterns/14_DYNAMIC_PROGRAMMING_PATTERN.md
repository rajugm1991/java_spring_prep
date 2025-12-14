# Pattern 14: Dynamic Programming 💡

## Overview

**Dynamic Programming** solves complex problems by breaking them into simpler subproblems and storing results to avoid recomputation.

## When to Use

✅ **Use DP when:**
- Optimal substructure (optimal solution contains optimal subproblems)
- Overlapping subproblems (same subproblems computed multiple times)
- Optimization problems
- Counting problems
- Decision problems

## Pattern Template

```java
public class DynamicProgramming {
    
    // Template 1: Top-Down (Memoization)
    public int topDown(int n) {
        int[] memo = new int[n + 1];
        Arrays.fill(memo, -1);
        return topDownHelper(n, memo);
    }
    
    private int topDownHelper(int n, int[] memo) {
        if (n <= 1) return n;
        if (memo[n] != -1) return memo[n];
        
        memo[n] = topDownHelper(n - 1, memo) + topDownHelper(n - 2, memo);
        return memo[n];
    }
    
    // Template 2: Bottom-Up (Tabulation)
    public int bottomUp(int n) {
        if (n <= 1) return n;
        
        int[] dp = new int[n + 1];
        dp[0] = 0;
        dp[1] = 1;
        
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    // Template 3: Space-Optimized
    public int spaceOptimized(int n) {
        if (n <= 1) return n;
        
        int prev2 = 0;
        int prev1 = 1;
        
        for (int i = 2; i <= n; i++) {
            int current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }
        
        return prev1;
    }
}
```

---

## Core Problems

### Problem 1: Fibonacci Numbers

**LeetCode:** [509. Fibonacci Number](https://leetcode.com/problems/fibonacci-number/)

**Solution (Bottom-Up):**
```java
public int fib(int n) {
    if (n <= 1) return n;
    
    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    
    return dp[n];
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

**Space-Optimized:**
```java
public int fib(int n) {
    if (n <= 1) return n;
    
    int prev2 = 0;
    int prev1 = 1;
    
    for (int i = 2; i <= n; i++) {
        int current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}
```

**Space Complexity:** O(1)

---

### Problem 2: Climbing Stairs

**LeetCode:** [70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)

**Problem:** Ways to climb n stairs (1 or 2 steps at a time).

**Solution:**
```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    
    int prev2 = 1;
    int prev1 = 2;
    
    for (int i = 3; i <= n; i++) {
        int current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}
```

---

### Problem 3: Coin Change

**LeetCode:** [322. Coin Change](https://leetcode.com/problems/coin-change/)

**Problem:** Minimum coins to make amount.

**Solution:**
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    
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

**Time Complexity:** O(amount × coins.length)  
**Space Complexity:** O(amount)

---

### Problem 4: Longest Increasing Subsequence

**LeetCode:** [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

**Solution:**
```java
public int lengthOfLIS(int[] nums) {
    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);
    int maxLength = 1;
    
    for (int i = 1; i < nums.length; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLength = Math.max(maxLength, dp[i]);
    }
    
    return maxLength;
}
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(n)

---

### Problem 5: House Robber

**LeetCode:** [198. House Robber](https://leetcode.com/problems/house-robber/)

**Solution:**
```java
public int rob(int[] nums) {
    if (nums.length == 0) return 0;
    if (nums.length == 1) return nums[0];
    
    int prev2 = 0;
    int prev1 = nums[0];
    
    for (int i = 1; i < nums.length; i++) {
        int current = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}
```

---

## DP Categories

### 1. 1D DP
- Fibonacci, Climbing Stairs, House Robber

### 2. 2D DP
- Unique Paths, Longest Common Subsequence

### 3. Knapsack
- 0/1 Knapsack, Unbounded Knapsack

### 4. String DP
- Edit Distance, Longest Palindromic Substring

---

## Key Takeaways

✅ Identify optimal substructure and overlapping subproblems  
✅ Choose top-down (memoization) or bottom-up (tabulation)  
✅ Optimize space when possible  
✅ Build solution from smaller subproblems

---

## Frequently Asked Interview Questions

### Q1: How to identify DP problems?

**Answer:**
- Optimization or counting problems
- Can be broken into subproblems
- Subproblems overlap
- Optimal substructure property

### Q2: Top-down vs Bottom-up?

**Answer:**
- **Top-down:** Recursive with memoization, easier to think
- **Bottom-up:** Iterative, usually faster, better space optimization

### Q3: How to optimize space?

**Answer:**
- If only previous states needed, use variables instead of array
- Example: Fibonacci only needs prev2 and prev1

---

**🎉 Congratulations! You've completed all 14 DSA Patterns!**

**Review:** [Back to README](./README.md)

