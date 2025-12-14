# Pattern 8: Subsets 🔢

## Overview

**Subsets** pattern generates all possible subsets/combinations of a set. Uses backtracking to explore all possibilities.

## When to Use

✅ **Use Subsets when:**
- Generating all subsets/combinations
- Permutations
- Power set problems
- Combination problems

## Pattern Template

```java
public class Subsets {
    
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }
    
    private void backtrack(int[] nums, int start, 
                          List<Integer> current, 
                          List<List<Integer>> result) {
        // Add current subset
        result.add(new ArrayList<>(current));
        
        // Explore all possibilities
        for (int i = start; i < nums.length; i++) {
            current.add(nums[i]);        // Choose
            backtrack(nums, i + 1, current, result); // Explore
            current.remove(current.size() - 1); // Unchoose
        }
    }
}
```

---

## Core Problems

### Problem 1: Subsets

**LeetCode:** [78. Subsets](https://leetcode.com/problems/subsets/)

**Solution:**
```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, 
                      List<Integer> current, 
                      List<List<Integer>> result) {
    result.add(new ArrayList<>(current));
    
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);
        backtrack(nums, i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

**Time Complexity:** O(2^n)  
**Space Complexity:** O(2^n)

---

### Problem 2: Subsets II (With Duplicates)

**LeetCode:** [90. Subsets II](https://leetcode.com/problems/subsets-ii/)

**Solution:**
```java
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, 
                      List<Integer> current, 
                      List<List<Integer>> result) {
    result.add(new ArrayList<>(current));
    
    for (int i = start; i < nums.length; i++) {
        // Skip duplicates
        if (i > start && nums[i] == nums[i-1]) continue;
        
        current.add(nums[i]);
        backtrack(nums, i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

---

### Problem 3: Combinations

**LeetCode:** [77. Combinations](https://leetcode.com/problems/combinations/)

**Problem:** Generate all combinations of k numbers from 1 to n.

**Solution:**
```java
public List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(1, n, k, new ArrayList<>(), result);
    return result;
}

private void backtrack(int start, int n, int k,
                      List<Integer> current,
                      List<List<Integer>> result) {
    if (current.size() == k) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = start; i <= n; i++) {
        current.add(i);
        backtrack(i + 1, n, k, current, result);
        current.remove(current.size() - 1);
    }
}
```

---

## Key Takeaways

✅ Use backtracking for subset/combination problems  
✅ Time: O(2^n) for subsets  
✅ Skip duplicates when array has duplicates  
✅ Add current state to result at each step

---

**Next Pattern:** [Modified Binary Search Pattern](./09_MODIFIED_BINARY_SEARCH_PATTERN.md)

