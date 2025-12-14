# Pattern 2: Sliding Window 🪟

## Overview

The **Sliding Window** pattern maintains a window of elements and slides it through the array/string to solve problems efficiently. It's perfect for subarray/substring problems where you need to find optimal windows.

## When to Use

✅ **Use Sliding Window when:**
- Finding subarrays/substrings with specific properties
- Maximum/minimum in a window
- Fixed or variable window size
- String problems (anagrams, permutations)
- Array problems (sum, average, product)

## Pattern Template

```java
public class SlidingWindow {
    
    // Template 1: Fixed Window Size
    public int fixedWindow(int[] arr, int k) {
        int windowSum = 0;
        int maxSum = 0;
        
        // Calculate sum of first window
        for (int i = 0; i < k; i++) {
            windowSum += arr[i];
        }
        maxSum = windowSum;
        
        // Slide the window
        for (int i = k; i < arr.length; i++) {
            windowSum = windowSum - arr[i - k] + arr[i]; // Remove left, add right
            maxSum = Math.max(maxSum, windowSum);
        }
        
        return maxSum;
    }
    
    // Template 2: Variable Window Size
    public int variableWindow(int[] arr, int target) {
        int left = 0;
        int windowSum = 0;
        int minLength = Integer.MAX_VALUE;
        
        for (int right = 0; right < arr.length; right++) {
            windowSum += arr[right]; // Expand window
            
            while (windowSum >= target) { // Shrink window
                minLength = Math.min(minLength, right - left + 1);
                windowSum -= arr[left];
                left++;
            }
        }
        
        return minLength == Integer.MAX_VALUE ? 0 : minLength;
    }
}
```

---

## Core Problems

### Problem 1: Maximum Sum Subarray of Size K

**Problem:** Find maximum sum of any contiguous subarray of size k.

**Example:**
```
Input: arr = [2, 1, 5, 1, 3, 2], k = 3
Output: 9
Explanation: Subarray [5, 1, 3] has maximum sum = 9
```

**Solution:**
```java
public int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0;
    int maxSum = 0;
    
    // Calculate sum of first window
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    maxSum = windowSum;
    
    // Slide the window
    for (int i = k; i < arr.length; i++) {
        windowSum = windowSum - arr[i - k] + arr[i];
        maxSum = Math.max(maxSum, windowSum);
    }
    
    return maxSum;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 2: Longest Substring with K Distinct Characters

**LeetCode:** [340. Longest Substring with At Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/)

**Problem:** Find longest substring with at most k distinct characters.

**Example:**
```
Input: s = "araaci", k = 2
Output: 4
Explanation: "araa" has 2 distinct characters
```

**Solution:**
```java
public int longestSubstringKDistinct(String s, int k) {
    if (s == null || s.length() == 0 || k == 0) return 0;
    
    Map<Character, Integer> charCount = new HashMap<>();
    int left = 0;
    int maxLength = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char rightChar = s.charAt(right);
        charCount.put(rightChar, charCount.getOrDefault(rightChar, 0) + 1);
        
        // Shrink window if distinct chars > k
        while (charCount.size() > k) {
            char leftChar = s.charAt(left);
            charCount.put(leftChar, charCount.get(leftChar) - 1);
            if (charCount.get(leftChar) == 0) {
                charCount.remove(leftChar);
            }
            left++;
        }
        
        maxLength = Math.max(maxLength, right - left + 1);
    }
    
    return maxLength;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(k)

---

### Problem 3: Minimum Window Substring

**LeetCode:** [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)

**Problem:** Find minimum window in s that contains all characters of t.

**Example:**
```
Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
```

**Solution:**
```java
public String minWindow(String s, String t) {
    if (s.length() < t.length()) return "";
    
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }
    
    Map<Character, Integer> window = new HashMap<>();
    int left = 0, right = 0;
    int valid = 0; // Number of characters that meet requirement
    int start = 0, minLen = Integer.MAX_VALUE;
    
    while (right < s.length()) {
        char c = s.charAt(right);
        right++;
        
        // Update window
        if (need.containsKey(c)) {
            window.put(c, window.getOrDefault(c, 0) + 1);
            if (window.get(c).equals(need.get(c))) {
                valid++;
            }
        }
        
        // Shrink window
        while (valid == need.size()) {
            // Update result
            if (right - left < minLen) {
                start = left;
                minLen = right - left;
            }
            
            char d = s.charAt(left);
            left++;
            
            if (need.containsKey(d)) {
                if (window.get(d).equals(need.get(d))) {
                    valid--;
                }
                window.put(d, window.get(d) - 1);
            }
        }
    }
    
    return minLen == Integer.MAX_VALUE ? "" : s.substring(start, start + minLen);
}
```

**Time Complexity:** O(|s| + |t|)  
**Space Complexity:** O(|s| + |t|)

---

### Problem 4: Longest Substring Without Repeating Characters

**LeetCode:** [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

**Problem:** Find length of longest substring without repeating characters.

**Example:**
```
Input: s = "abcabcbb"
Output: 3
Explanation: "abc" is longest substring
```

**Solution:**
```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> charIndex = new HashMap<>();
    int left = 0;
    int maxLength = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        
        // If character seen before and within current window
        if (charIndex.containsKey(c) && charIndex.get(c) >= left) {
            left = charIndex.get(c) + 1; // Move left past duplicate
        }
        
        charIndex.put(c, right);
        maxLength = Math.max(maxLength, right - left + 1);
    }
    
    return maxLength;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(min(n, m)) where m is charset size

---

### Problem 5: Permutation in String

**LeetCode:** [567. Permutation in String](https://leetcode.com/problems/permutation-in-string/)

**Problem:** Check if s2 contains a permutation of s1.

**Example:**
```
Input: s1 = "ab", s2 = "eidbaooo"
Output: true
Explanation: s2 contains "ba" which is permutation of "ab"
```

**Solution:**
```java
public boolean checkInclusion(String s1, String s2) {
    if (s1.length() > s2.length()) return false;
    
    int[] count1 = new int[26];
    int[] count2 = new int[26];
    
    // Count characters in s1 and first window of s2
    for (int i = 0; i < s1.length(); i++) {
        count1[s1.charAt(i) - 'a']++;
        count2[s2.charAt(i) - 'a']++;
    }
    
    // Check first window
    if (Arrays.equals(count1, count2)) return true;
    
    // Slide window
    for (int i = s1.length(); i < s2.length(); i++) {
        count2[s2.charAt(i) - 'a']++; // Add right
        count2[s2.charAt(i - s1.length()) - 'a']--; // Remove left
        
        if (Arrays.equals(count1, count2)) return true;
    }
    
    return false;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Advanced Problems (Staff Engineer Level)

### Problem 6: Subarray Product Less Than K

**LeetCode:** [713. Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/)

**Problem:** Count subarrays with product less than k.

**Example:**
```
Input: nums = [10,5,2,6], k = 100
Output: 8
Explanation: [10], [5], [2], [6], [10,5], [5,2], [2,6], [5,2,6]
```

**Solution:**
```java
public int numSubarrayProductLessThanK(int[] nums, int k) {
    if (k <= 1) return 0;
    
    int left = 0;
    int product = 1;
    int count = 0;
    
    for (int right = 0; right < nums.length; right++) {
        product *= nums[right];
        
        // Shrink window while product >= k
        while (product >= k) {
            product /= nums[left];
            left++;
        }
        
        // All subarrays ending at right with product < k
        count += right - left + 1;
    }
    
    return count;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

**Key Insight:** For window [left, right], number of valid subarrays = right - left + 1

---

### Problem 7: Maximum Points You Can Obtain from Cards

**LeetCode:** [1423. Maximum Points You Can Obtain from Cards](https://leetcode.com/problems/maximum-points-you-can-obtain-from-cards/)

**Problem:** Pick k cards from either end, maximize sum.

**Example:**
```
Input: cardPoints = [1,2,3,4,5,6,1], k = 3
Output: 12
Explanation: Pick [1,6,5] from end
```

**Solution:**
```java
public int maxScore(int[] cardPoints, int k) {
    int n = cardPoints.length;
    int windowSize = n - k;
    int windowSum = 0;
    int totalSum = 0;
    
    // Calculate sum of first window and total sum
    for (int i = 0; i < windowSize; i++) {
        windowSum += cardPoints[i];
        totalSum += cardPoints[i];
    }
    for (int i = windowSize; i < n; i++) {
        totalSum += cardPoints[i];
    }
    
    int minWindowSum = windowSum;
    
    // Slide window to find minimum window sum
    for (int i = windowSize; i < n; i++) {
        windowSum = windowSum - cardPoints[i - windowSize] + cardPoints[i];
        minWindowSum = Math.min(minWindowSum, windowSum);
    }
    
    return totalSum - minWindowSum;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

**Key Insight:** Maximize sum from ends = Minimize sum of middle window

---

## Frequently Asked Interview Questions

### Q1: What's the difference between fixed and variable window?

**Answer:**
- **Fixed Window:** Window size is constant (k)
- **Variable Window:** Window size changes based on condition (expand until condition met, then shrink)

### Q2: How to handle negative numbers in sliding window?

**Answer:**
- Use HashMap to track counts (works with negatives)
- For sum problems, use prefix sum technique
- Consider absolute values if applicable

### Q3: When to use sliding window vs two pointers?

**Answer:**
- **Sliding Window:** Subarray/substring problems, maintaining window state
- **Two Pointers:** Pair finding, palindrome, when pointers move independently

### Q4: How to optimize sliding window for large inputs?

**Answer:**
- Use array instead of HashMap if charset is small (26 for lowercase)
- Pre-calculate window sum/product
- Avoid recalculating window state

### Q5: What's the space complexity of sliding window?

**Answer:**
- Usually O(1) for fixed window
- O(k) or O(n) for variable window (depends on what we're tracking)

---

## Real-World Applications

1. **Network Traffic Analysis:** Analyzing packets in time windows
2. **Stock Trading:** Moving averages, maximum profit in window
3. **Text Processing:** Finding patterns, anagrams
4. **Video Streaming:** Buffer management, quality adaptation
5. **System Monitoring:** Rate limiting, anomaly detection

---

## Optimization Tips

1. **Use Arrays for Small Charsets:** Faster than HashMap
2. **Pre-calculate Window State:** Avoid recalculating
3. **Early Termination:** Stop when optimal found
4. **Deque for Min/Max:** Use Deque for sliding window min/max
5. **Prefix Sum:** For sum-related problems

---

## Practice Problems

### Easy
- [ ] Maximum Sum Subarray of Size K
- [ ] Average of Subarrays of Size K
- [ ] Maximum of All Subarrays of Size K

### Medium
- [ ] Longest Substring with K Distinct Characters
- [ ] Minimum Window Substring
- [ ] Permutation in String
- [ ] Longest Substring Without Repeating Characters

### Hard
- [ ] Subarray Product Less Than K
- [ ] Sliding Window Maximum
- [ ] Minimum Window Subsequence
- [ ] Count Subarrays with Given XOR

---

## Key Takeaways

✅ Sliding window reduces O(n²) to O(n) for subarray problems  
✅ Fixed window: maintain size k  
✅ Variable window: expand until condition, then shrink  
✅ Use HashMap/Array to track window state  
✅ Space complexity usually O(k) or O(1)

---

**Next Pattern:** [Fast & Slow Pointers Pattern](./03_FAST_SLOW_POINTERS_PATTERN.md)

