# Array Patterns - Complete Guide 🎯

> Master array problem-solving patterns with visual explanations and recognition strategies

---

## Table of Contents
1. [How to Recognize Patterns](#how-to-recognize-patterns)
2. [Two Pointers Pattern](#two-pointers-pattern)
3. [Sliding Window Pattern](#sliding-window-pattern)
4. [Prefix Sum Pattern](#prefix-sum-pattern)
5. [Cyclic Sort Pattern](#cyclic-sort-pattern)
6. [In-place Reversal Pattern](#in-place-reversal-pattern)
7. [Fast & Slow Pointers Pattern](#fast--slow-pointers-pattern)
8. [Merge Intervals Pattern](#merge-intervals-pattern)
9. [Modified Binary Search Pattern](#modified-binary-search-pattern)
10. [Subarray Problems Pattern](#subarray-problems-pattern)
11. [Frequently Asked Questions](#frequently-asked-questions)

---

## How to Recognize Patterns

### Pattern Recognition Checklist

When you see an array problem, ask these questions:

#### 1. **Is the array sorted?**
- ✅ **Yes** → Consider **Two Pointers** or **Binary Search**
- ❌ **No** → May need sorting first, or use other patterns

#### 2. **Do you need to find pairs/triplets?**
- ✅ **Yes** → **Two Pointers** (if sorted) or **Hash Set/Map**
- ❌ **No** → Continue to next question

#### 3. **Do you need a subarray/substring?**
- ✅ **Yes** → **Sliding Window** (fixed or variable size)
- ❌ **No** → Continue to next question

#### 4. **Do you need range queries (sum, min, max)?**
- ✅ **Yes** → **Prefix Sum** or **Segment Tree**
- ❌ **No** → Continue to next question

#### 5. **Are elements in range [1, n] or [0, n-1]?**
- ✅ **Yes** → **Cyclic Sort** (find missing/duplicate)
- ❌ **No** → Continue to next question

#### 6. **Do you need to reverse or rotate?**
- ✅ **Yes** → **In-place Reversal**
- ❌ **No** → Continue to next question

#### 7. **Is there a cycle or loop?**
- ✅ **Yes** → **Fast & Slow Pointers** (Floyd's algorithm)
- ❌ **No** → Continue to next question

#### 8. **Do you need to merge overlapping intervals?**
- ✅ **Yes** → **Merge Intervals**
- ❌ **No** → Continue to next question

### Quick Pattern Decision Tree

```
Array Problem
    │
    ├─ Sorted? ──Yes──→ Two Pointers / Binary Search
    │
    ├─ Subarray? ──Yes──→ Sliding Window
    │
    ├─ Range Queries? ──Yes──→ Prefix Sum
    │
    ├─ Elements [1,n]? ──Yes──→ Cyclic Sort
    │
    ├─ Reverse/Rotate? ──Yes──→ In-place Reversal
    │
    ├─ Cycle/Loop? ──Yes──→ Fast & Slow Pointers
    │
    └─ Intervals? ──Yes──→ Merge Intervals
```

---

## Two Pointers Pattern

### When to Use

**Recognize:**
- ✅ Sorted array
- ✅ Finding pairs/triplets
- ✅ Palindrome problems
- ✅ Removing duplicates
- ✅ Partitioning array

**Key Insight:** Use two pointers moving from different positions

### Visual Pattern

```
Array: [1, 2, 3, 4, 5, 6]
        ↑              ↑
      left          right

Move based on condition:
- If condition met: move both
- If need smaller: right--
- If need larger: left++
```

### Example 1: Two Sum (Sorted Array)

**Problem:** Find two numbers that add up to target

**Visual:**
```
Array: [2, 7, 11, 15], target = 9

Step 1: left=0, right=3
  [2, 7, 11, 15]
   ↑         ↑
  2 + 15 = 17 > 9 → right--

Step 2: left=0, right=2
  [2, 7, 11, 15]
   ↑      ↑
  2 + 11 = 13 > 9 → right--

Step 3: left=0, right=1
  [2, 7, 11, 15]
   ↑  ↑
  2 + 7 = 9 ✅ Found!
```

**Code:**
```java
int[] twoSum(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    
    while (left < right) {
        int sum = nums[left] + nums[right];
        
        if (sum == target) {
            return new int[]{left, right};
        } else if (sum < target) {
            left++;  // Need larger sum
        } else {
            right--; // Need smaller sum
        }
    }
    
    return new int[]{-1, -1};
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Remove Duplicates

**Problem:** Remove duplicates from sorted array in-place

**Visual:**
```
Array: [1, 1, 2, 2, 3, 4, 4, 5]

Step 1: slow=0, fast=1
  [1, 1, 2, 2, 3, 4, 4, 5]
   ↑  ↑
  1 == 1 → fast++

Step 2: slow=0, fast=2
  [1, 1, 2, 2, 3, 4, 4, 5]
   ↑     ↑
  1 != 2 → nums[++slow] = 2

Step 3: slow=1, fast=3
  [1, 2, 2, 2, 3, 4, 4, 5]
      ↑  ↑
  2 == 2 → fast++

... continue until fast reaches end

Result: [1, 2, 3, 4, 5]
```

**Code:**
```java
int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    
    int slow = 0;  // Position for next unique element
    
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) {
            slow++;
            nums[slow] = nums[fast];
        }
    }
    
    return slow + 1;  // Length of unique array
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 3: Three Sum

**Problem:** Find all unique triplets that sum to zero

**Visual:**
```
Array: [-4, -1, -1, 0, 1, 2]

For each i:
  i=0: [-4, -1, -1, 0, 1, 2]
        ↑   ↑              ↑
       -4 + (-1) + 2 = -3 < 0 → left++
       
  i=0: [-4, -1, -1, 0, 1, 2]
        ↑      ↑           ↑
       -4 + (-1) + 2 = -3 < 0 → left++
       
  i=0: [-4, -1, -1, 0, 1, 2]
        ↑         ↑        ↑
       -4 + 0 + 2 = -2 < 0 → left++
       
  i=0: [-4, -1, -1, 0, 1, 2]
        ↑            ↑     ↑
       -4 + 1 + 2 = -1 < 0 → left++
       
  i=1: [-4, -1, -1, 0, 1, 2]
            ↑   ↑        ↑
       -1 + (-1) + 2 = 0 ✅ Found: [-1, -1, 2]
```

**Code:**
```java
List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    
    for (int i = 0; i < nums.length - 2; i++) {
        // Skip duplicates
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        
        int left = i + 1;
        int right = nums.length - 1;
        
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                
                // Skip duplicates
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                
                left++;
                right--;
            } else if (sum < 0) {
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

## Sliding Window Pattern

### When to Use

**Recognize:**
- ✅ Subarray/substring problems
- ✅ Fixed size window
- ✅ Variable size window (longest/shortest)
- ✅ "Contiguous" keyword
- ✅ "Subarray" keyword

**Key Insight:** Maintain a window and slide it based on condition

### Visual Pattern

```
Fixed Window (size k=3):
[1, 2, 3, 4, 5, 6]
 ↑     ↑
window

Slide:
[1, 2, 3, 4, 5, 6]
    ↑     ↑
   window

Variable Window:
[1, 2, 3, 4, 5, 6]
 ↑  ↑
window (expand/contract)
```

### Example 1: Maximum Sum Subarray of Size K

**Problem:** Find maximum sum of subarray of size k

**Visual:**
```
Array: [2, 1, 5, 1, 3, 2], k = 3

Step 1: Window [2, 1, 5]
  [2, 1, 5, 1, 3, 2]
   ↑     ↑
  sum = 2 + 1 + 5 = 8

Step 2: Slide window [1, 5, 1]
  [2, 1, 5, 1, 3, 2]
      ↑     ↑
  sum = 1 + 5 + 1 = 7

Step 3: Slide window [5, 1, 3]
  [2, 1, 5, 1, 3, 2]
         ↑     ↑
  sum = 5 + 1 + 3 = 9 ✅ Max

Step 4: Slide window [1, 3, 2]
  [2, 1, 5, 1, 3, 2]
            ↑     ↑
  sum = 1 + 3 + 2 = 6

Max sum = 9
```

**Code:**
```java
int maxSumSubarray(int[] nums, int k) {
    int windowSum = 0;
    int maxSum = 0;
    
    // Calculate sum of first window
    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }
    maxSum = windowSum;
    
    // Slide window
    for (int i = k; i < nums.length; i++) {
        windowSum = windowSum - nums[i - k] + nums[i];
        maxSum = Math.max(maxSum, windowSum);
    }
    
    return maxSum;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Longest Substring with K Distinct Characters

**Problem:** Find longest substring with at most k distinct characters

**Visual:**
```
String: "araaci", k = 2

Step 1: Expand window
  "araaci"
   ↑
  window = "a" (distinct: 1)

Step 2: Expand
  "araaci"
   ↑↑
  window = "ar" (distinct: 2)

Step 3: Expand
  "araaci"
   ↑↑↑
  window = "ara" (distinct: 2) ✅

Step 4: Expand
  "araaci"
   ↑↑↑↑
  window = "araa" (distinct: 2) ✅

Step 5: Expand
  "araaci"
   ↑↑↑↑↑
  window = "araac" (distinct: 3) ❌
  
  Shrink from left:
  "araaci"
    ↑↑↑↑
  window = "raac" (distinct: 3) ❌
  
  Shrink more:
  "araaci"
     ↑↑↑
  window = "aac" (distinct: 2) ✅

Longest = "araa" (length 4)
```

**Code:**
```java
int longestSubstring(String s, int k) {
    Map<Character, Integer> map = new HashMap<>();
    int left = 0;
    int maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char rightChar = s.charAt(right);
        map.put(rightChar, map.getOrDefault(rightChar, 0) + 1);
        
        // Shrink window if distinct chars > k
        while (map.size() > k) {
            char leftChar = s.charAt(left);
            map.put(leftChar, map.get(leftChar) - 1);
            if (map.get(leftChar) == 0) {
                map.remove(leftChar);
            }
            left++;
        }
        
        maxLen = Math.max(maxLen, right - left + 1);
    }
    
    return maxLen;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(k)

---

### Example 3: Minimum Window Substring

**Problem:** Find minimum window in S that contains all characters of T

**Visual:**
```
S = "ADOBECODEBANC", T = "ABC"

Step 1: Expand until all chars found
  "ADOBECODEBANC"
   ↑         ↑
  window = "ADOBEC" (contains A, B, C) ✅

Step 2: Shrink from left
  "ADOBECODEBANC"
    ↑        ↑
  window = "DOBECODEBA" (contains A, B, C) ✅

Step 3: Shrink more
  "ADOBECODEBANC"
     ↑       ↑
  window = "OBECODEBA" (contains A, B, C) ✅

Step 4: Shrink more
  "ADOBECODEBANC"
      ↑      ↑
  window = "BECODEBA" (missing A) ❌
  
  Expand:
  "ADOBECODEBANC"
      ↑       ↑
  window = "BECODEBAN" (contains A, B, C) ✅

Step 5: Shrink
  "ADOBECODEBANC"
       ↑      ↑
  window = "ECODEBAN" (missing A) ❌
  
  Expand:
  "ADOBECODEBANC"
       ↑       ↑
  window = "ECODEBANC" (contains A, B, C) ✅

Minimum window = "BANC"
```

**Code:**
```java
String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    
    // Count characters needed
    for (char c : t.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }
    
    int left = 0, right = 0;
    int valid = 0;  // Number of characters that meet requirement
    int start = 0, len = Integer.MAX_VALUE;
    
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
            // Update minimum window
            if (right - left < len) {
                start = left;
                len = right - left;
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
    
    return len == Integer.MAX_VALUE ? "" : s.substring(start, start + len);
}
```

**Time Complexity:** O(|s| + |t|)  
**Space Complexity:** O(|s| + |t|)

---

## Prefix Sum Pattern

### When to Use

**Recognize:**
- ✅ Range sum queries
- ✅ Subarray sum problems
- ✅ "Sum from index i to j"
- ✅ Multiple range queries

**Key Insight:** Precompute cumulative sums for O(1) range queries

### Visual Pattern

```
Array: [1, 2, 3, 4, 5]

Prefix Sum: [0, 1, 3, 6, 10, 15]
            ↑  ↑  ↑  ↑  ↑   ↑
           0  1  3  6  10  15

Sum from index 1 to 3:
prefix[4] - prefix[1] = 10 - 1 = 9
Which is: 2 + 3 + 4 = 9 ✅
```

### Example 1: Range Sum Query

**Problem:** Answer multiple range sum queries efficiently

**Visual:**
```
Array: [1, 2, 3, 4, 5]

Build prefix sum:
prefix[0] = 0
prefix[1] = 1
prefix[2] = 1 + 2 = 3
prefix[3] = 3 + 3 = 6
prefix[4] = 6 + 4 = 10
prefix[5] = 10 + 5 = 15

Query: sum(1, 3)
prefix[4] - prefix[1] = 10 - 1 = 9 ✅

Query: sum(0, 2)
prefix[3] - prefix[0] = 6 - 0 = 6 ✅
```

**Code:**
```java
class RangeSumQuery {
    private int[] prefix;
    
    RangeSumQuery(int[] nums) {
        prefix = new int[nums.length + 1];
        for (int i = 0; i < nums.length; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }
    }
    
    int sumRange(int left, int right) {
        return prefix[right + 1] - prefix[left];
    }
}
```

**Time Complexity:** O(1) per query, O(n) preprocessing  
**Space Complexity:** O(n)

---

### Example 2: Subarray Sum Equals K

**Problem:** Count subarrays with sum equal to k

**Visual:**
```
Array: [1, 1, 1], k = 2

Prefix Sum: [0, 1, 2, 3]

We need: prefix[j] - prefix[i] = k
Which means: prefix[j] - k = prefix[i]

Check each j:
j=1: prefix[1]=1, need prefix[i]=1-2=-1 → not found
j=2: prefix[2]=2, need prefix[i]=2-2=0 → found! (i=0)
     Subarray: [1, 1] ✅
j=3: prefix[3]=3, need prefix[i]=3-2=1 → found! (i=1)
     Subarray: [1, 1] ✅

Count = 2
```

**Code:**
```java
int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> map = new HashMap<>();
    map.put(0, 1);  // prefix[0] = 0
    
    int prefixSum = 0;
    int count = 0;
    
    for (int num : nums) {
        prefixSum += num;
        
        // Check if prefixSum - k exists
        if (map.containsKey(prefixSum - k)) {
            count += map.get(prefixSum - k);
        }
        
        // Add current prefix sum
        map.put(prefixSum, map.getOrDefault(prefixSum, 0) + 1);
    }
    
    return count;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## Cyclic Sort Pattern

### When to Use

**Recognize:**
- ✅ Array contains numbers in range [1, n] or [0, n-1]
- ✅ Find missing number
- ✅ Find duplicate number
- ✅ Find all missing/duplicate numbers

**Key Insight:** Each number should be at index = number - 1

### Visual Pattern

```
Array: [3, 1, 5, 4, 2] (range [1, 5])

Ideal: [1, 2, 3, 4, 5]
        ↑  ↑  ↑  ↑  ↑
        0  1  2  3  4 (indices)

For each position i:
  If nums[i] != i + 1:
    Swap nums[i] with nums[nums[i] - 1]
```

### Example 1: Find Missing Number

**Problem:** Find missing number in array [0, n] with n+1 numbers

**Visual:**
```
Array: [4, 0, 3, 1] (n=4, should have [0,1,2,3,4])

Step 1: i=0, nums[0]=4
  [4, 0, 3, 1]
   ↑
  4 should be at index 4, but array size is 4
  Skip (4 >= n)

Step 2: i=1, nums[1]=0
  [4, 0, 3, 1]
      ↑
  0 should be at index 0
  Swap: [0, 4, 3, 1]

Step 3: i=1, nums[1]=4
  [0, 4, 3, 1]
      ↑
  4 >= n, skip

Step 4: i=2, nums[2]=3
  [0, 4, 3, 1]
         ↑
  3 should be at index 3
  Swap: [0, 4, 1, 3]

Step 5: i=2, nums[2]=1
  [0, 4, 1, 3]
         ↑
  1 should be at index 1
  Swap: [0, 1, 4, 3]

Step 6: i=2, nums[2]=4
  [0, 1, 4, 3]
            ↑
  4 >= n, skip

Step 7: i=3, nums[3]=3
  [0, 1, 4, 3]
               ↑
  3 is at index 3 ✅

Final: [0, 1, 4, 3]
Missing: 2 (index 2 doesn't have value 2)
```

**Code:**
```java
int findMissingNumber(int[] nums) {
    int i = 0;
    
    // Cyclic sort
    while (i < nums.length) {
        if (nums[i] < nums.length && nums[i] != nums[nums[i]]) {
            swap(nums, i, nums[i]);
        } else {
            i++;
        }
    }
    
    // Find missing
    for (i = 0; i < nums.length; i++) {
        if (nums[i] != i) {
            return i;
        }
    }
    
    return nums.length;
}

void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Find All Duplicates

**Problem:** Find all duplicates in array [1, n]

**Visual:**
```
Array: [4, 3, 2, 7, 8, 2, 3, 1]

Cyclic sort:
[1, 2, 3, 4, 3, 2, 7, 8]
         ↑  ↑
    Duplicates at wrong positions

After sort:
[1, 2, 3, 4, 3, 2, 7, 8]
            ↑  ↑
    At index 4: value 3, but should be 5 → duplicate
    At index 5: value 2, but should be 6 → duplicate

Duplicates: [2, 3]
```

**Code:**
```java
List<Integer> findDuplicates(int[] nums) {
    int i = 0;
    
    // Cyclic sort
    while (i < nums.length) {
        if (nums[i] != nums[nums[i] - 1]) {
            swap(nums, i, nums[i] - 1);
        } else {
            i++;
        }
    }
    
    // Find duplicates
    List<Integer> duplicates = new ArrayList<>();
    for (i = 0; i < nums.length; i++) {
        if (nums[i] != i + 1) {
            duplicates.add(nums[i]);
        }
    }
    
    return duplicates;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## In-place Reversal Pattern

### When to Use

**Recognize:**
- ✅ Reverse array/string
- ✅ Rotate array
- ✅ Reverse subarray
- ✅ "In-place" requirement

**Key Insight:** Use two pointers swapping from both ends

### Visual Pattern

```
Reverse: [1, 2, 3, 4, 5]

Step 1: left=0, right=4
  [1, 2, 3, 4, 5]
   ↑           ↑
  swap → [5, 2, 3, 4, 1]

Step 2: left=1, right=3
  [5, 2, 3, 4, 1]
      ↑     ↑
  swap → [5, 4, 3, 2, 1]

Step 3: left=2, right=2
  Done!

Result: [5, 4, 3, 2, 1]
```

### Example 1: Reverse Array

**Code:**
```java
void reverse(int[] nums) {
    int left = 0;
    int right = nums.length - 1;
    
    while (left < right) {
        swap(nums, left, right);
        left++;
        right--;
    }
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Rotate Array

**Problem:** Rotate array to right by k steps

**Visual:**
```
Array: [1, 2, 3, 4, 5], k = 2

Step 1: Reverse entire array
  [5, 4, 3, 2, 1]

Step 2: Reverse first k elements
  [4, 5, 3, 2, 1]

Step 3: Reverse remaining elements
  [4, 5, 1, 2, 3] ✅
```

**Code:**
```java
void rotate(int[] nums, int k) {
    k = k % nums.length;  // Handle k > n
    
    // Reverse entire array
    reverse(nums, 0, nums.length - 1);
    
    // Reverse first k elements
    reverse(nums, 0, k - 1);
    
    // Reverse remaining elements
    reverse(nums, k, nums.length - 1);
}

void reverse(int[] nums, int start, int end) {
    while (start < end) {
        swap(nums, start, end);
        start++;
        end--;
    }
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Fast & Slow Pointers Pattern

### When to Use

**Recognize:**
- ✅ Linked list cycle detection
- ✅ Find middle of linked list
- ✅ Find cycle start
- ✅ Palindrome in linked list

**Key Insight:** Use two pointers moving at different speeds

### Visual Pattern

```
Array/Linked List: [1, 2, 3, 4, 5, 6]

Fast & Slow:
[1, 2, 3, 4, 5, 6]
 ↑     ↑
slow  fast

After one step:
[1, 2, 3, 4, 5, 6]
    ↑        ↑
  slow      fast

Fast moves 2x speed of slow
```

### Example 1: Find Middle Element

**Visual:**
```
Array: [1, 2, 3, 4, 5]

Step 1: slow=0, fast=0
  [1, 2, 3, 4, 5]
   ↑
  slow, fast

Step 2: slow=1, fast=2
  [1, 2, 3, 4, 5]
      ↑     ↑
    slow   fast

Step 3: slow=2, fast=4
  [1, 2, 3, 4, 5]
         ↑     ↑
       slow   fast

Fast reached end, slow is at middle: 3
```

**Code:**
```java
int findMiddle(int[] nums) {
    int slow = 0;
    int fast = 0;
    
    while (fast < nums.length - 1) {
        slow++;
        fast += 2;
    }
    
    return nums[slow];
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Detect Cycle (Floyd's Algorithm)

**Visual:**
```
Array with cycle: [1, 2, 3, 4, 5, 3, 4, 5, ...]
                  ↑  ↑  ↑  ↑  ↑  ↑
                  0  1  2  3  4  5 (indices)

Step 1: slow=0, fast=0
  [1, 2, 3, 4, 5, 3, ...]
   ↑
  slow, fast

Step 2: slow=1, fast=2
  [1, 2, 3, 4, 5, 3, ...]
      ↑     ↑
    slow   fast

Step 3: slow=2, fast=4
  [1, 2, 3, 4, 5, 3, ...]
         ↑     ↑
       slow   fast

Step 4: slow=3, fast=1 (cycle: 5→3)
  [1, 2, 3, 4, 5, 3, ...]
            ↑  ↑
          slow fast

Step 5: slow=4, fast=3
  [1, 2, 3, 4, 5, 3, ...]
               ↑ ↑
             slow fast

Step 6: slow=3, fast=3 (meet!) ✅ Cycle detected
```

**Code:**
```java
boolean hasCycle(int[] nums) {
    int slow = 0;
    int fast = 0;
    
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
        
        if (slow == fast) {
            return true;  // Cycle detected
        }
    } while (slow != fast && fast < nums.length);
    
    return false;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Merge Intervals Pattern

### When to Use

**Recognize:**
- ✅ Overlapping intervals
- ✅ Merge intervals
- ✅ Insert interval
- ✅ Meeting rooms problems

**Key Insight:** Sort by start time, then merge overlapping

### Visual Pattern

```
Intervals: [[1,3], [2,6], [8,10], [15,18]]

Step 1: Sort by start
  [[1,3], [2,6], [8,10], [15,18]]

Step 2: Check overlap
  [1,3] and [2,6]: overlap (2 < 3)
  Merge: [1,6]

Step 3: Check next
  [1,6] and [8,10]: no overlap (8 > 6)
  Keep both

Step 4: Check next
  [8,10] and [15,18]: no overlap
  Keep both

Result: [[1,6], [8,10], [15,18]]
```

### Example: Merge Intervals

**Code:**
```java
int[][] merge(int[][] intervals) {
    if (intervals.length <= 1) return intervals;
    
    // Sort by start time
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    
    List<int[]> merged = new ArrayList<>();
    int[] current = intervals[0];
    
    for (int i = 1; i < intervals.length; i++) {
        int[] next = intervals[i];
        
        // Check overlap: next.start <= current.end
        if (next[0] <= current[1]) {
            // Merge: extend current end
            current[1] = Math.max(current[1], next[1]);
        } else {
            // No overlap: add current, start new
            merged.add(current);
            current = next;
        }
    }
    
    merged.add(current);
    return merged.toArray(new int[merged.size()][]);
}
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(n)

---

## Modified Binary Search Pattern

### When to Use

**Recognize:**
- ✅ Sorted array
- ✅ Find element in rotated sorted array
- ✅ Find peak element
- ✅ Search in 2D matrix

**Key Insight:** Modify binary search condition based on problem

### Example: Search in Rotated Sorted Array

**Visual:**
```
Array: [4, 5, 6, 7, 0, 1, 2], target = 0

Step 1: left=0, right=6, mid=3
  [4, 5, 6, 7, 0, 1, 2]
   ↑        ↑        ↑
  left    mid      right
  nums[mid] = 7 > nums[left] → left half sorted
  
  target = 0 not in [4, 5, 6, 7]
  left = mid + 1 = 4

Step 2: left=4, right=6, mid=5
  [4, 5, 6, 7, 0, 1, 2]
               ↑  ↑  ↑
             left mid right
  nums[mid] = 1 < nums[left] → right half sorted
  
  target = 0 in [0, 1, 2]
  right = mid - 1 = 4

Step 3: left=4, right=4, mid=4
  [4, 5, 6, 7, 0, 1, 2]
                  ↑
                found!
```

**Code:**
```java
int search(int[] nums, int target) {
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
        } else {  // Right half is sorted
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

## Subarray Problems Pattern

### When to Use

**Recognize:**
- ✅ Maximum/minimum subarray
- ✅ Subarray with given sum
- ✅ Subarray with given property

**Key Insight:** Use sliding window or Kadane's algorithm

### Example: Maximum Subarray (Kadane's Algorithm)

**Visual:**
```
Array: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

Kadane's: Keep track of max ending at current position

i=0: maxEnding = -2, maxSoFar = -2
i=1: maxEnding = max(-2+1, 1) = 1, maxSoFar = 1
i=2: maxEnding = max(1-3, -3) = -2, maxSoFar = 1
i=3: maxEnding = max(-2+4, 4) = 4, maxSoFar = 4
i=4: maxEnding = max(4-1, -1) = 3, maxSoFar = 4
i=5: maxEnding = max(3+2, 2) = 5, maxSoFar = 5
i=6: maxEnding = max(5+1, 1) = 6, maxSoFar = 6 ✅
i=7: maxEnding = max(6-5, -5) = 1, maxSoFar = 6
i=8: maxEnding = max(1+4, 4) = 5, maxSoFar = 6

Max subarray sum = 6 ([4, -1, 2, 1])
```

**Code:**
```java
int maxSubarray(int[] nums) {
    int maxEnding = nums[0];
    int maxSoFar = nums[0];
    
    for (int i = 1; i < nums.length; i++) {
        // Either extend previous subarray or start new
        maxEnding = Math.max(nums[i], maxEnding + nums[i]);
        maxSoFar = Math.max(maxSoFar, maxEnding);
    }
    
    return maxSoFar;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Frequently Asked Questions

### Q1: How do I know which pattern to use?

**Answer:**
"Look for keywords and characteristics:
- **Sorted array + pairs** → Two Pointers
- **Subarray/substring** → Sliding Window
- **Range queries** → Prefix Sum
- **Elements [1,n]** → Cyclic Sort
- **Reverse/rotate** → In-place Reversal
- **Cycle detection** → Fast & Slow Pointers
- **Overlapping intervals** → Merge Intervals
- **Sorted + search** → Binary Search

I start by identifying the problem type, then match it to the pattern."

---

### Q2: What's the difference between Two Pointers and Sliding Window?

**Answer:**
"**Two Pointers:**
- Usually moves from both ends toward center
- Used for pairs, palindromes, removing duplicates
- Both pointers move based on condition

**Sliding Window:**
- Maintains a window that slides
- Used for subarrays/substrings
- Window expands/contracts based on condition
- Can be fixed size or variable size

Example: Two Sum uses two pointers, Longest Substring uses sliding window."

---

### Q3: When should I use Prefix Sum vs Sliding Window?

**Answer:**
"**Prefix Sum:**
- Multiple range queries (preprocess once, query many times)
- Need O(1) range sum queries
- Subarray sum equals k problems

**Sliding Window:**
- Single pass through array
- Need to find optimal window
- Real-time processing

If you need to answer many queries, use prefix sum. If you need one optimal solution, use sliding window."

---

### Q4: How does Cyclic Sort work?

**Answer:**
"Cyclic sort places each number at its correct position (index = value - 1 for [1,n]).

Algorithm:
1. For each position i, if nums[i] != i+1, swap it to correct position
2. Continue until all numbers are in correct positions
3. Then scan to find missing/duplicate numbers

Time: O(n) because each element is placed at most once."

---

### Q5: What's the intuition behind Fast & Slow Pointers?

**Answer:**
"Fast pointer moves 2x speed of slow pointer.

**Key insight:** If there's a cycle, fast will eventually catch slow.

**Applications:**
- Cycle detection: If fast == slow, cycle exists
- Find middle: When fast reaches end, slow is at middle
- Find cycle start: After detecting cycle, reset one pointer to start

This is Floyd's cycle detection algorithm."

---

### Q6: How to handle edge cases in array problems?

**Answer:**
"Always check:
1. **Empty array**: `if (nums.length == 0) return ...`
2. **Single element**: `if (nums.length == 1) return ...`
3. **All same elements**: Test with [1,1,1,1]
4. **Negative numbers**: Check if algorithm handles negatives
5. **Integer overflow**: Use long if sum might overflow
6. **Null checks**: `if (nums == null) return ...`

I write these checks first before implementing the main logic."

---

### Q7: What's the time complexity of these patterns?

**Answer:**
"**O(n):**
- Two Pointers (single pass)
- Sliding Window (single pass)
- Fast & Slow Pointers (single pass)
- Cyclic Sort (each element placed once)
- In-place Reversal (single pass)

**O(n log n):**
- Merge Intervals (sorting)

**O(log n):**
- Modified Binary Search

**O(1) per query:**
- Prefix Sum (after O(n) preprocessing)

Most array patterns are O(n) because we traverse the array once."

---

### Q8: How to optimize space complexity?

**Answer:**
"**In-place operations:**
- Use same array, swap elements
- Two pointers instead of extra array
- Modify array instead of creating new one

**Examples:**
- Remove duplicates: Use slow pointer in same array
- Reverse array: Swap in-place
- Cyclic sort: Swap to correct positions

**Trade-off:** Sometimes O(n) space is needed (like prefix sum for O(1) queries)."

---

## Quick Reference Table

| Pattern | When to Use | Time | Space | Key Insight |
|---------|-------------|------|-------|-------------|
| **Two Pointers** | Sorted array, pairs | O(n) | O(1) | Move from both ends |
| **Sliding Window** | Subarray/substring | O(n) | O(k) | Maintain window |
| **Prefix Sum** | Range queries | O(1) query | O(n) | Precompute sums |
| **Cyclic Sort** | Elements [1,n] | O(n) | O(1) | Place at index = value-1 |
| **In-place Reversal** | Reverse/rotate | O(n) | O(1) | Swap from ends |
| **Fast & Slow** | Cycle detection | O(n) | O(1) | 2x speed difference |
| **Merge Intervals** | Overlapping intervals | O(n log n) | O(n) | Sort then merge |
| **Binary Search** | Sorted array search | O(log n) | O(1) | Divide and conquer |
| **Kadane's** | Max subarray | O(n) | O(1) | Track max ending here |

---

## Pattern Recognition Cheat Sheet

```
Problem Type → Pattern

"Find two numbers" + Sorted → Two Pointers
"Subarray" / "Substring" → Sliding Window
"Sum from i to j" → Prefix Sum
"Missing number" + [1,n] → Cyclic Sort
"Reverse" / "Rotate" → In-place Reversal
"Cycle" / "Loop" → Fast & Slow Pointers
"Overlapping" + Intervals → Merge Intervals
"Search" + Sorted → Binary Search
"Maximum subarray" → Kadane's Algorithm
```

---

## Daily Revision Checklist ✅

- [ ] Two Pointers: Two Sum, Remove Duplicates
- [ ] Sliding Window: Max Sum, Longest Substring
- [ ] Prefix Sum: Range Query, Subarray Sum
- [ ] Cyclic Sort: Missing Number, Duplicates
- [ ] In-place Reversal: Reverse, Rotate
- [ ] Fast & Slow: Middle, Cycle Detection
- [ ] Merge Intervals: Overlapping Intervals
- [ ] Binary Search: Rotated Array, Peak
- [ ] Subarray: Maximum Subarray (Kadane's)

---

**Happy Coding! 🎯**

