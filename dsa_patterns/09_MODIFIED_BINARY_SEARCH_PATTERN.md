# Pattern 9: Modified Binary Search 🔍

## Overview

**Modified Binary Search** adapts standard binary search for variations like finding boundaries, rotated arrays, or search spaces.

## When to Use

✅ **Use Modified Binary Search when:**
- Finding first/last occurrence
- Search in rotated sorted array
- Finding peak elements
- Search space problems
- Finding boundaries

## Pattern Template

```java
public class ModifiedBinarySearch {
    
    // Template: Find target in sorted array
    public int binarySearch(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return -1;
    }
    
    // Template: Find first occurrence
    public int findFirst(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        int result = -1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) {
                result = mid;
                right = mid - 1; // Continue searching left
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return result;
    }
}
```

---

## Core Problems

### Problem 1: Search in Rotated Sorted Array

**LeetCode:** [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)

**Solution:**
```java
public int search(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) {
            return mid;
        }
        
        // Left half is sorted
        if (nums[left] <= nums[mid]) {
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        } else { // Right half is sorted
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    
    return -1;
}
```

**Time Complexity:** O(log n)  
**Space Complexity:** O(1)

---

### Problem 2: Find First and Last Position

**LeetCode:** [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

**Solution:**
```java
public int[] searchRange(int[] nums, int target) {
    int first = findFirst(nums, target);
    if (first == -1) return new int[]{-1, -1};
    
    int last = findLast(nums, target);
    return new int[]{first, last};
}

private int findFirst(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            result = mid;
            right = mid - 1;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}

private int findLast(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            result = mid;
            left = mid + 1;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}
```

---

### Problem 3: Find Peak Element

**LeetCode:** [162. Find Peak Element](https://leetcode.com/problems/find-peak-element/)

**Solution:**
```java
public int findPeakElement(int[] nums) {
    int left = 0;
    int right = nums.length - 1;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] > nums[mid + 1]) {
            right = mid; // Peak is in left half
        } else {
            left = mid + 1; // Peak is in right half
        }
    }
    
    return left;
}
```

---

## Key Takeaways

✅ Adapt binary search for variations  
✅ Find boundaries by continuing search  
✅ Handle rotated arrays by checking sorted halves  
✅ Time: O(log n)

---

**Next Pattern:** [Top K Elements Pattern](./10_TOP_K_ELEMENTS_PATTERN.md)

