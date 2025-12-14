# Pattern 5: Cyclic Sort 🔄

## Overview

**Cyclic Sort** is used when dealing with arrays containing numbers in a given range. Instead of using comparison-based sorting, we place each number at its correct index.

## When to Use

✅ **Use Cyclic Sort when:**
- Array contains numbers in range [1, n] or [0, n-1]
- Need to find missing/duplicate numbers
- Need to sort with O(1) space
- Problems involving cyclic positioning

## Pattern Template

```java
public class CyclicSort {
    
    public void cyclicSort(int[] nums) {
        int i = 0;
        while (i < nums.length) {
            int correctIndex = nums[i] - 1; // For range [1, n]
            // int correctIndex = nums[i]; // For range [0, n-1]
            
            if (nums[i] != nums[correctIndex]) {
                swap(nums, i, correctIndex);
            } else {
                i++;
            }
        }
    }
    
    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

---

## Core Problems

### Problem 1: Cyclic Sort

**Problem:** Sort array containing numbers 1 to n.

**Solution:**
```java
public void cyclicSort(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correctIndex = nums[i] - 1;
        if (nums[i] != nums[correctIndex]) {
            swap(nums, i, correctIndex);
        } else {
            i++;
        }
    }
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 2: Find Missing Number

**LeetCode:** [268. Missing Number](https://leetcode.com/problems/missing-number/)

**Problem:** Find missing number in array containing n distinct numbers in range [0, n].

**Solution:**
```java
public int missingNumber(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        if (nums[i] < nums.length && nums[i] != i) {
            swap(nums, i, nums[i]);
        } else {
            i++;
        }
    }
    
    for (i = 0; i < nums.length; i++) {
        if (nums[i] != i) {
            return i;
        }
    }
    
    return nums.length;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 3: Find All Missing Numbers

**LeetCode:** [448. Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/)

**Problem:** Find all missing numbers in array [1, n].

**Solution:**
```java
public List<Integer> findDisappearedNumbers(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correctIndex = nums[i] - 1;
        if (nums[i] != nums[correctIndex]) {
            swap(nums, i, correctIndex);
        } else {
            i++;
        }
    }
    
    List<Integer> result = new ArrayList<>();
    for (i = 0; i < nums.length; i++) {
        if (nums[i] != i + 1) {
            result.add(i + 1);
        }
    }
    
    return result;
}
```

---

### Problem 4: Find Duplicate Number

**LeetCode:** [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

**Problem:** Find duplicate number in array [1, n] with one duplicate.

**Solution:**
```java
public int findDuplicate(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        if (nums[i] != i + 1) {
            int correctIndex = nums[i] - 1;
            if (nums[i] != nums[correctIndex]) {
                swap(nums, i, correctIndex);
            } else {
                return nums[i]; // Found duplicate
            }
        } else {
            i++;
        }
    }
    
    return -1;
}
```

---

## Key Takeaways

✅ Use when array contains numbers in range [1, n] or [0, n-1]  
✅ O(n) time, O(1) space  
✅ Place each number at its correct index  
✅ After sorting, missing/duplicate numbers are easy to find

---

**Next Pattern:** [In-place Reversal Pattern](./06_INPLACE_REVERSAL_PATTERN.md)

