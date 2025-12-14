# Pattern 1: Two Pointers 🎯

## Overview

The **Two Pointers** pattern uses two pointers moving through a data structure (usually an array or linked list) to solve problems efficiently. This pattern is particularly useful for sorted arrays and can reduce time complexity from O(n²) to O(n).

## When to Use

✅ **Use Two Pointers when:**
- Array or linked list is sorted
- Need to find pairs/triplets that meet certain criteria
- Need to remove duplicates
- Palindrome checking
- Reversing or partitioning arrays
- Finding subarrays with specific properties

## Pattern Template

```java
public class TwoPointers {
    
    // Template 1: Opposite Ends (Most Common)
    public int[] twoPointersOppositeEnds(int[] arr, int target) {
        int left = 0;
        int right = arr.length - 1;
        
        while (left < right) {
            // Process current pair
            int sum = arr[left] + arr[right];
            
            if (sum == target) {
                return new int[]{left, right};
            } else if (sum < target) {
                left++;  // Need larger value
            } else {
                right--; // Need smaller value
            }
        }
        
        return new int[]{-1, -1};
    }
    
    // Template 2: Same Direction (Fast & Slow)
    public int twoPointersSameDirection(int[] arr) {
        int slow = 0;
        int fast = 0;
        
        while (fast < arr.length) {
            if (condition(arr[slow], arr[fast])) {
                arr[++slow] = arr[fast];
            }
            fast++;
        }
        
        return slow + 1;
    }
}
```

---

## Core Problems

### Problem 1: Two Sum (Sorted Array)

**LeetCode:** [167. Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

**Problem:** Given a sorted array, find two numbers that add up to target. Return indices (1-indexed).

**Example:**
```
Input: numbers = [2,7,11,15], target = 9
Output: [1,2]
Explanation: numbers[0] + numbers[1] = 2 + 7 = 9
```

**Solution:**
```java
public int[] twoSum(int[] numbers, int target) {
    int left = 0;
    int right = numbers.length - 1;
    
    while (left < right) {
        int sum = numbers[left] + numbers[right];
        
        if (sum == target) {
            return new int[]{left + 1, right + 1}; // 1-indexed
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    
    return new int[]{-1, -1};
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 2: Remove Duplicates from Sorted Array

**LeetCode:** [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

**Problem:** Remove duplicates in-place, return new length.

**Example:**
```
Input: nums = [1,1,2]
Output: 2, nums = [1,2,_]
```

**Solution:**
```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    
    int slow = 0;
    
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) {
            nums[++slow] = nums[fast];
        }
    }
    
    return slow + 1;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 3: Valid Palindrome

**LeetCode:** [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)

**Problem:** Check if string is palindrome (ignoring non-alphanumeric, case-insensitive).

**Example:**
```
Input: s = "A man, a plan, a canal: Panama"
Output: true
```

**Solution:**
```java
public boolean isPalindrome(String s) {
    int left = 0;
    int right = s.length() - 1;
    
    while (left < right) {
        // Skip non-alphanumeric from left
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) {
            left++;
        }
        
        // Skip non-alphanumeric from right
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) {
            right--;
        }
        
        // Compare characters (case-insensitive)
        if (Character.toLowerCase(s.charAt(left)) != 
            Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        
        left++;
        right--;
    }
    
    return true;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 4: Container With Most Water

**LeetCode:** [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)

**Problem:** Find two lines that form container with most water.

**Example:**
```
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49
Explanation: Lines at index 1 and 8 form container with area = 49
```

**Solution:**
```java
public int maxArea(int[] height) {
    int left = 0;
    int right = height.length - 1;
    int maxArea = 0;
    
    while (left < right) {
        int width = right - left;
        int minHeight = Math.min(height[left], height[right]);
        int area = width * minHeight;
        maxArea = Math.max(maxArea, area);
        
        // Move pointer with smaller height
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    
    return maxArea;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

**Why this works:** We always move the pointer with smaller height because:
- Area is limited by smaller height
- Moving larger height pointer won't increase area (width decreases, height stays same or decreases)

---

### Problem 5: Three Sum

**LeetCode:** [15. 3Sum](https://leetcode.com/problems/3sum/)

**Problem:** Find all unique triplets that sum to zero.

**Example:**
```
Input: nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]
```

**Solution:**
```java
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);
    
    for (int i = 0; i < nums.length - 2; i++) {
        // Skip duplicates for first number
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue;
        }
        
        int left = i + 1;
        int right = nums.length - 1;
        int target = -nums[i];
        
        while (left < right) {
            int sum = nums[left] + nums[right];
            
            if (sum == target) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                
                // Skip duplicates
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                
                left++;
                right--;
            } else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }
    }
    
    return result;
}
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(1) excluding result array

---

## Advanced Problems (Staff Engineer Level)

### Problem 6: Trapping Rain Water

**LeetCode:** [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

**Problem:** Calculate trapped rainwater.

**Example:**
```
Input: height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

**Solution (Two Pointers):**
```java
public int trap(int[] height) {
    if (height.length == 0) return 0;
    
    int left = 0;
    int right = height.length - 1;
    int leftMax = 0;
    int rightMax = 0;
    int water = 0;
    
    while (left < right) {
        if (height[left] < height[right]) {
            if (height[left] >= leftMax) {
                leftMax = height[left];
            } else {
                water += leftMax - height[left];
            }
            left++;
        } else {
            if (height[right] >= rightMax) {
                rightMax = height[right];
            } else {
                water += rightMax - height[right];
            }
            right--;
        }
    }
    
    return water;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

**Key Insight:** At each position, trapped water = min(leftMax, rightMax) - currentHeight

---

### Problem 7: Sort Colors (Dutch National Flag)

**LeetCode:** [75. Sort Colors](https://leetcode.com/problems/sort-colors/)

**Problem:** Sort array of 0s, 1s, and 2s in-place.

**Example:**
```
Input: nums = [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]
```

**Solution (Three Pointers):**
```java
public void sortColors(int[] nums) {
    int left = 0;        // Points to next position for 0
    int current = 0;     // Current element
    int right = nums.length - 1; // Points to next position for 2
    
    while (current <= right) {
        if (nums[current] == 0) {
            swap(nums, left, current);
            left++;
            current++;
        } else if (nums[current] == 1) {
            current++;
        } else { // nums[current] == 2
            swap(nums, current, right);
            right--;
            // Don't increment current - need to check swapped element
        }
    }
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Frequently Asked Interview Questions

### Q1: How do you decide which pointer to move?

**Answer:**
- **Opposite ends pattern:** Move the pointer that helps achieve the target condition
  - If sum < target → move left (need larger value)
  - If sum > target → move right (need smaller value)
- **Same direction pattern:** Move fast pointer, conditionally move slow pointer

### Q2: What's the difference between two pointers and sliding window?

**Answer:**
- **Two Pointers:** Pointers move independently based on condition
- **Sliding Window:** Maintains a window of fixed/variable size, both pointers move together

### Q3: Can two pointers work on unsorted arrays?

**Answer:**
- Generally no for opposite ends pattern (requires sorted array)
- Yes for same direction pattern (removing duplicates, partitioning)
- Can sort first if needed (adds O(n log n) time)

### Q4: How to handle duplicates in two sum problems?

**Answer:**
```java
// Skip duplicates after finding a match
while (left < right && nums[left] == nums[left + 1]) left++;
while (left < right && nums[right] == nums[right - 1]) right--;
```

### Q5: What's the space complexity of two pointers?

**Answer:**
- Usually O(1) - only using a few variables
- O(n) if storing results (like in 3Sum)

---

## Real-World Applications

1. **Database Query Optimization:** Finding pairs in sorted indexes
2. **Merge Operations:** Merging two sorted arrays/lists
3. **String Processing:** Palindrome checking, string matching
4. **Image Processing:** Finding boundaries, edge detection
5. **Game Development:** Collision detection, pathfinding

---

## Optimization Tips

1. **Early Termination:** Stop when condition is met
2. **Skip Duplicates:** Avoid processing same elements multiple times
3. **Use While Loops:** More efficient than for loops for pointer movement
4. **Avoid Unnecessary Swaps:** Only swap when necessary
5. **Cache Calculations:** Store repeated calculations (like sum, product)

---

## Practice Problems

### Easy
- [ ] Two Sum (Sorted)
- [ ] Remove Duplicates
- [ ] Valid Palindrome
- [ ] Reverse String

### Medium
- [ ] Container With Most Water
- [ ] Three Sum
- [ ] Four Sum
- [ ] Sort Colors

### Hard
- [ ] Trapping Rain Water
- [ ] Longest Substring Without Repeating Characters (with two pointers)
- [ ] Minimum Window Substring
- [ ] Subarray Product Less Than K

---

## Key Takeaways

✅ Two pointers reduce O(n²) to O(n) for sorted arrays  
✅ Use opposite ends for pair finding, same direction for filtering  
✅ Always consider edge cases (empty array, single element)  
✅ Handle duplicates explicitly  
✅ Space complexity is usually O(1)

---

**Next Pattern:** [Sliding Window Pattern](./02_SLIDING_WINDOW_PATTERN.md)

