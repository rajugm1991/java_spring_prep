# DSA Patterns - Complete Guide with Examples 🎯

> Master the most frequently asked DSA patterns with visual explanations and interview questions

---

## Table of Contents
1. [Two Pointers](#1-two-pointers)
2. [Sliding Window](#2-sliding-window)
3. [Fast & Slow Pointers](#3-fast--slow-pointers)
4. [Merge Intervals](#4-merge-intervals)
5. [Cyclic Sort](#5-cyclic-sort)
6. [In-place Reversal](#6-in-place-reversal)
7. [Tree BFS/DFS](#7-tree-bfsdfs)
8. [Subsets](#8-subsets)
9. [Modified Binary Search](#9-modified-binary-search)
10. [Top K Elements](#10-top-k-elements)
11. [K-way Merge](#11-k-way-merge)
12. [Topological Sort](#12-topological-sort)
13. [Backtracking](#13-backtracking)
14. [Dynamic Programming Patterns](#14-dynamic-programming-patterns)

---

## 1. Two Pointers

### What is Two Pointers?

**Pattern:** Use two pointers moving from different positions to solve problems efficiently

**When to Use:**
- Sorted arrays
- Palindrome problems
- Pair sum problems
- Removing duplicates

### Visual Pattern

```
Array: [1, 2, 3, 4, 5, 6]
        ↑              ↑
      left          right

Move pointers based on condition:
- If sum < target: left++
- If sum > target: right--
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
   ↑   ↑
  slow fast (same, skip)

Step 2: slow=0, fast=2
  [1, 1, 2, 2, 3, 4, 4, 5]
   ↑      ↑
  slow   fast (different, copy)
  [1, 2, 2, 2, 3, 4, 4, 5]
      ↑   ↑
     slow fast

Step 3: Continue...
  [1, 2, 3, 4, 5, ...]
```

**Code:**
```java
int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    
    int slow = 0;  // Position for unique elements
    
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) {
            slow++;
            nums[slow] = nums[fast];
        }
    }
    
    return slow + 1;  // Length of unique array
}
```

---

### Example 3: Valid Palindrome

**Problem:** Check if string is palindrome (ignoring case and non-alphanumeric)

**Visual:**
```
String: "A man, a plan, a canal: Panama"

Step 1: left=0, right=30
  "A man, a plan, a canal: Panama"
   ↑                            ↑
  'A' == 'a' (case-insensitive) ✅

Step 2: left=1, right=29
  "A man, a plan, a canal: Panama"
    ↑                         ↑
  ' ' (skip) → left++

Step 3: left=2, right=29
  "A man, a plan, a canal: Panama"
     ↑                        ↑
  'm' == 'm' ✅
```

**Code:**
```java
boolean isPalindrome(String s) {
    int left = 0;
    int right = s.length() - 1;
    
    while (left < right) {
        // Skip non-alphanumeric
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) {
            left++;
        }
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) {
            right--;
        }
        
        // Compare
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

---

### Frequently Asked Questions

**Q1: When should I use two pointers?**
- Sorted arrays
- Need to find pairs
- Palindrome problems
- Removing duplicates

**Q2: What's the difference between two pointers and sliding window?**
- **Two Pointers:** Both pointers move based on condition (often opposite directions)
- **Sliding Window:** Fixed or variable window size, both pointers move in same direction

**Q3: Time complexity of two pointers?**
- Usually O(n) - each element visited once

---

## 2. Sliding Window

### What is Sliding Window?

**Pattern:** Maintain a window of elements and slide it to find optimal subarray/substring

**When to Use:**
- Subarray/substring problems
- Fixed or variable window size
- Maximum/minimum in window

### Visual Pattern

```
Array: [1, 3, 2, 6, -1, 4, 1, 8, 2], k=5

Fixed Window (k=5):
[1, 3, 2, 6, -1] 4, 1, 8, 2  → sum = 11
 1, [3, 2, 6, -1, 4] 1, 8, 2  → sum = 14
 1, 3, [2, 6, -1, 4, 1] 8, 2  → sum = 12
 ...
```

### Example 1: Maximum Sum Subarray of Size K

**Problem:** Find maximum sum of subarray with size k

**Visual:**
```
Array: [2, 1, 5, 1, 3, 2], k=3

Window 1: [2, 1, 5] → sum = 8
Window 2: [1, 5, 1] → sum = 7
Window 3: [5, 1, 3] → sum = 9 ← Max
Window 4: [1, 3, 2] → sum = 6

Answer: 9
```

**Code:**
```java
int maxSumSubarray(int[] nums, int k) {
    int windowSum = 0;
    int maxSum = 0;
    
    // Calculate first window
    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }
    maxSum = windowSum;
    
    // Slide window
    for (int i = k; i < nums.length; i++) {
        windowSum = windowSum - nums[i - k] + nums[i];  // Remove left, add right
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
String: "araaci", k=2

Window: [a, r, a, a] → distinct = {a, r} = 2 ✅, length = 4
        [a, r, a, a, c] → distinct = {a, r, c} = 3 ❌
        [r, a, a, c] → distinct = {r, a, c} = 3 ❌
        [a, a, c] → distinct = {a, c} = 2 ✅, length = 3
        [a, a, c, i] → distinct = {a, c, i} = 3 ❌
        [a, c, i] → distinct = {a, c, i} = 3 ❌
        [c, i] → distinct = {c, i} = 2 ✅, length = 2

Answer: 4
```

**Code:**
```java
int longestSubstringKDistinct(String s, int k) {
    Map<Character, Integer> map = new HashMap<>();
    int left = 0;
    int maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        map.put(c, map.getOrDefault(c, 0) + 1);
        
        // Shrink window if distinct > k
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

---

### Example 3: Minimum Window Substring

**Problem:** Find minimum window in s that contains all characters of t

**Visual:**
```
s = "ADOBECODEBANC", t = "ABC"

Need: A, B, C (all characters)

Window 1: [A, D, O, B, E, C] → has A, B, C ✅, length = 6
Window 2: [D, O, B, E, C, O, D, E, B, A, N, C] → has A, B, C ✅, length = 12
Window 3: [B, A, N, C] → has A, B, C ✅, length = 4 ← Minimum

Answer: "BANC"
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
    int valid = 0;  // Number of characters that satisfy condition
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
            // Update result
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

---

### Frequently Asked Questions

**Q1: Fixed vs Variable Window?**
- **Fixed:** Window size is constant (k)
- **Variable:** Window size changes based on condition

**Q2: When to use sliding window?**
- Subarray/substring problems
- Need to find optimal window
- Maximum/minimum in window

**Q3: Time complexity?**
- Usually O(n) - each element visited at most twice

---

## 3. Fast & Slow Pointers

### What is Fast & Slow Pointers?

**Pattern:** Use two pointers moving at different speeds to detect cycles or find middle

**When to Use:**
- Linked list cycle detection
- Find middle of linked list
- Find cycle start
- Palindrome in linked list

### Visual Pattern

```
Linked List: 1 → 2 → 3 → 4 → 5

Step 1:
  1 → 2 → 3 → 4 → 5
  ↑
slow, fast

Step 2:
  1 → 2 → 3 → 4 → 5
     ↑    ↑
   slow  fast

Step 3:
  1 → 2 → 3 → 4 → 5
        ↑       ↑
      slow     fast

When fast reaches end, slow is at middle!
```

### Example 1: Detect Cycle in Linked List

**Problem:** Check if linked list has a cycle

**Visual:**
```
Cycle: 1 → 2 → 3 → 4 → 5
              ↑         ↓
              8 ← 7 ← 6

Step 1: slow=1, fast=1
Step 2: slow=2, fast=3
Step 3: slow=3, fast=5
Step 4: slow=4, fast=7
Step 5: slow=5, fast=3 (fast caught up, cycle detected!)
```

**Code:**
```java
boolean hasCycle(ListNode head) {
    if (head == null) return false;
    
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;        // Move 1 step
        fast = fast.next.next;    // Move 2 steps
        
        if (slow == fast) {
            return true;  // Cycle detected
        }
    }
    
    return false;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Find Middle of Linked List

**Problem:** Find middle node of linked list

**Visual:**
```
List: 1 → 2 → 3 → 4 → 5

Step 1: slow=1, fast=1
Step 2: slow=2, fast=3
Step 3: slow=3, fast=5
Fast reaches end, slow is at middle (3) ✅
```

**Code:**
```java
ListNode findMiddle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    
    return slow;
}
```

---

### Example 3: Find Cycle Start

**Problem:** Find the node where cycle starts

**Visual:**
```
Cycle: 1 → 2 → 3 → 4 → 5
              ↑         ↓
              8 ← 7 ← 6

Step 1: Find meeting point (slow and fast meet at 7)
Step 2: Move one pointer to head, keep other at meeting point
Step 3: Move both one step at a time
Step 4: They meet at cycle start (3)
```

**Code:**
```java
ListNode detectCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    
    // Find meeting point
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        
        if (slow == fast) {
            // Cycle found, find start
            ListNode ptr1 = head;
            ListNode ptr2 = slow;
            
            while (ptr1 != ptr2) {
                ptr1 = ptr1.next;
                ptr2 = ptr2.next;
            }
            
            return ptr1;  // Cycle start
        }
    }
    
    return null;  // No cycle
}
```

**Why it works:** Mathematical proof - distance from head to cycle start = distance from meeting point to cycle start

---

### Frequently Asked Questions

**Q1: Why does fast & slow pointer work for cycle detection?**
- If there's a cycle, fast will eventually catch up to slow
- If no cycle, fast reaches null

**Q2: What if fast moves 3 steps instead of 2?**
- Still works, but 2 steps is optimal and easier to understand

**Q3: Time complexity?**
- O(n) - linear time

---

## 4. Merge Intervals

### What is Merge Intervals?

**Pattern:** Merge overlapping intervals efficiently

**When to Use:**
- Overlapping intervals
- Meeting rooms
- Insert intervals
- Merge time ranges

### Visual Pattern

```
Intervals: [[1,3], [2,6], [8,10], [15,18]]

Step 1: Sort by start time
  [1,3], [2,6], [8,10], [15,18]

Step 2: Merge overlapping
  [1,3] and [2,6] overlap → [1,6]
  [1,6] and [8,10] don't overlap → keep both
  [8,10] and [15,18] don't overlap → keep both

Result: [[1,6], [8,10], [15,18]]
```

### Example 1: Merge Intervals

**Problem:** Merge all overlapping intervals

**Visual:**
```
Input: [[1,3], [2,6], [8,10], [15,18]]

After sorting: [[1,3], [2,6], [8,10], [15,18]]

Merge process:
  Current: [1,3]
  Next: [2,6] → overlap? Yes (2 <= 3) → merge to [1,6]
  
  Current: [1,6]
  Next: [8,10] → overlap? No (8 > 6) → add [1,6], current = [8,10]
  
  Current: [8,10]
  Next: [15,18] → overlap? No (15 > 10) → add [8,10], current = [15,18]
  
  Add [15,18]

Result: [[1,6], [8,10], [15,18]]
```

**Code:**
```java
int[][] merge(int[][] intervals) {
    if (intervals.length <= 1) return intervals;
    
    // Sort by start time
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    
    List<int[]> merged = new ArrayList<>();
    int[] current = intervals[0];
    
    for (int i = 1; i < intervals.length; i++) {
        int[] next = intervals[i];
        
        // Check if overlapping
        if (current[1] >= next[0]) {
            // Merge: update end time
            current[1] = Math.max(current[1], next[1]);
        } else {
            // No overlap, add current and move to next
            merged.add(current);
            current = next;
        }
    }
    
    merged.add(current);
    return merged.toArray(new int[merged.size()][]);
}
```

**Time Complexity:** O(n log n) - sorting  
**Space Complexity:** O(n)

---

### Example 2: Insert Interval

**Problem:** Insert new interval and merge if necessary

**Visual:**
```
Intervals: [[1,3], [6,9]]
New: [2,5]

Step 1: Find position
  [1,3] overlaps with [2,5]? Yes (2 <= 3)
  [6,9] overlaps with [2,5]? No (6 > 5)

Step 2: Merge overlapping
  [1,3] and [2,5] → [1, max(3,5)] = [1,5]

Step 3: Insert
  Result: [[1,5], [6,9]]
```

**Code:**
```java
int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0;
    
    // Add all intervals before newInterval
    while (i < intervals.length && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i]);
        i++;
    }
    
    // Merge overlapping intervals
    while (i < intervals.length && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    
    result.add(newInterval);
    
    // Add remaining intervals
    while (i < intervals.length) {
        result.add(intervals[i]);
        i++;
    }
    
    return result.toArray(new int[result.size()][]);
}
```

---

### Example 3: Meeting Rooms II

**Problem:** Find minimum number of meeting rooms needed

**Visual:**
```
Meetings: [[0,30], [5,10], [15,20]]

Timeline:
  0    5    10   15   20   30
  |----|    |    |----|    |
  [0,30]    [5,10]   [15,20]
     |      |         |
  Room 1  Room 2   Room 1 (reused)

Answer: 2 rooms
```

**Code:**
```java
int minMeetingRooms(int[][] intervals) {
    // Sort start times
    int[] starts = new int[intervals.length];
    int[] ends = new int[intervals.length];
    
    for (int i = 0; i < intervals.length; i++) {
        starts[i] = intervals[i][0];
        ends[i] = intervals[i][1];
    }
    
    Arrays.sort(starts);
    Arrays.sort(ends);
    
    int rooms = 0;
    int endPointer = 0;
    
    for (int start : starts) {
        if (start < ends[endPointer]) {
            rooms++;  // Need new room
        } else {
            endPointer++;  // Reuse room
        }
    }
    
    return rooms;
}
```

---

### Frequently Asked Questions

**Q1: How to check if two intervals overlap?**
- `interval1[1] >= interval2[0]` (end of first >= start of second)

**Q2: Why sort by start time?**
- Makes merging easier - process intervals in order

**Q3: Time complexity?**
- O(n log n) due to sorting

---

## 5. Cyclic Sort

### What is Cyclic Sort?

**Pattern:** Sort array in-place when numbers are in range [1, n]

**When to Use:**
- Array contains numbers 1 to n
- Need O(1) space
- Find missing/duplicate numbers

### Visual Pattern

```
Array: [3, 1, 5, 4, 2]

Idea: Each number should be at index (number - 1)

Step 1: i=0, nums[0]=3
  3 should be at index 2
  Swap: [5, 1, 3, 4, 2]

Step 2: i=0, nums[0]=5
  5 should be at index 4
  Swap: [2, 1, 3, 4, 5]

Step 3: i=0, nums[0]=2
  2 should be at index 1
  Swap: [1, 2, 3, 4, 5] ✅
```

### Example 1: Cyclic Sort

**Problem:** Sort array [1, n] in-place

**Code:**
```java
void cyclicSort(int[] nums) {
    int i = 0;
    
    while (i < nums.length) {
        int correctIndex = nums[i] - 1;  // Where nums[i] should be
        
        if (nums[i] != nums[correctIndex]) {
            // Swap to correct position
            swap(nums, i, correctIndex);
        } else {
            i++;  // Already in correct position
        }
    }
}

void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

**Time Complexity:** O(n) - each element visited at most twice  
**Space Complexity:** O(1)

---

### Example 2: Find Missing Number

**Problem:** Find missing number in array [0, n]

**Visual:**
```
Array: [4, 0, 2, 1]  (missing 3)

After cyclic sort: [0, 1, 2, 4]

Check: nums[3] = 4, but should be 3
Answer: 3
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
```

---

### Example 3: Find All Duplicates

**Problem:** Find all duplicates in array [1, n]

**Visual:**
```
Array: [4, 3, 2, 7, 8, 2, 3, 1]

After cyclic sort: [1, 2, 3, 4, 3, 2, 7, 8]

Check positions:
  nums[4] = 3, but should be 5 → 3 is duplicate
  nums[5] = 2, but should be 6 → 2 is duplicate

Answer: [2, 3]
```

**Code:**
```java
List<Integer> findDuplicates(int[] nums) {
    int i = 0;
    
    // Cyclic sort
    while (i < nums.length) {
        int correctIndex = nums[i] - 1;
        
        if (nums[i] != nums[correctIndex]) {
            swap(nums, i, correctIndex);
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

---

### Frequently Asked Questions

**Q1: When to use cyclic sort?**
- Array contains numbers 1 to n (or 0 to n-1)
- Need O(1) space
- Find missing/duplicate numbers

**Q2: Why is it O(n) time?**
- Each element is placed in correct position at most once
- Total swaps ≤ n

**Q3: What if numbers are not 1 to n?**
- Can't use cyclic sort directly, need to map to range first

---

## 6. In-place Reversal

### What is In-place Reversal?

**Pattern:** Reverse linked list or array without extra space

**When to Use:**
- Reverse linked list
- Reverse subarray
- Rotate array
- Palindrome problems

### Visual Pattern

```
Linked List: 1 → 2 → 3 → 4 → 5

Step 1: prev=null, curr=1
  1 → 2 → 3 → 4 → 5
  ↑
curr

Step 2: Save next, reverse link
  null ← 1  2 → 3 → 4 → 5
         ↑  ↑
      prev curr

Step 3: Move pointers
  null ← 1  2 → 3 → 4 → 5
         ↑  ↑
      prev curr

Continue until curr is null
Result: 5 → 4 → 3 → 2 → 1
```

### Example 1: Reverse Linked List

**Problem:** Reverse linked list in-place

**Code:**
```java
ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    
    while (curr != null) {
        ListNode next = curr.next;  // Save next
        curr.next = prev;            // Reverse link
        prev = curr;                 // Move prev
        curr = next;                 // Move curr
    }
    
    return prev;  // New head
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Reverse Linked List II

**Problem:** Reverse nodes from position left to right

**Visual:**
```
List: 1 → 2 → 3 → 4 → 5, left=2, right=4

Step 1: Find positions
  1 → 2 → 3 → 4 → 5
      ↑         ↑
    left      right

Step 2: Reverse middle part
  1 → 4 → 3 → 2 → 5

Result: 1 → 4 → 3 → 2 → 5
```

**Code:**
```java
ListNode reverseBetween(ListNode head, int left, int right) {
    if (head == null || left == right) return head;
    
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode prev = dummy;
    
    // Move to left position
    for (int i = 0; i < left - 1; i++) {
        prev = prev.next;
    }
    
    // Reverse
    ListNode curr = prev.next;
    for (int i = 0; i < right - left; i++) {
        ListNode next = curr.next;
        curr.next = next.next;
        next.next = prev.next;
        prev.next = next;
    }
    
    return dummy.next;
}
```

---

### Example 3: Rotate Array

**Problem:** Rotate array to right by k steps

**Visual:**
```
Array: [1, 2, 3, 4, 5], k=2

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
    k = k % nums.length;  // Handle k > length
    
    // Reverse entire array
    reverse(nums, 0, nums.length - 1);
    
    // Reverse first k elements
    reverse(nums, 0, k - 1);
    
    // Reverse remaining elements
    reverse(nums, k, nums.length - 1);
}

void reverse(int[] nums, int start, int end) {
    while (start < end) {
        int temp = nums[start];
        nums[start] = nums[end];
        nums[end] = temp;
        start++;
        end--;
    }
}
```

---

### Frequently Asked Questions

**Q1: Why use dummy node?**
- Simplifies edge cases (when reversing from head)

**Q2: Time complexity?**
- O(n) - visit each node once

**Q3: Space complexity?**
- O(1) - only use pointers, no extra space

---

## 7. Tree BFS/DFS

### What is Tree BFS/DFS?

**Pattern:** Traverse tree level-by-level (BFS) or depth-first (DFS)

**When to Use:**
- Tree traversal
- Level order problems
- Path problems
- Tree construction

### BFS Pattern

```
Tree:
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

BFS Order: 10 → 5 → 15 → 3 → 7 → 12 → 20
(Level by level)
```

### Example 1: Level Order Traversal

**Code:**
```java
List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> level = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        
        result.add(level);
    }
    
    return result;
}
```

### DFS Pattern

```
Tree:
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Pre-order:  10 → 5 → 3 → 7 → 15 → 12 → 20
In-order:   3 → 5 → 7 → 10 → 12 → 15 → 20
Post-order: 3 → 7 → 5 → 12 → 20 → 15 → 10
```

### Example 2: Maximum Depth

**Code:**
```java
int maxDepth(TreeNode root) {
    if (root == null) return 0;
    
    int left = maxDepth(root.left);
    int right = maxDepth(root.right);
    
    return 1 + Math.max(left, right);
}
```

### Example 3: Path Sum

**Code:**
```java
boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) return false;
    
    if (root.left == null && root.right == null) {
        return root.val == targetSum;
    }
    
    return hasPathSum(root.left, targetSum - root.val) ||
           hasPathSum(root.right, targetSum - root.val);
}
```

---

## 8. Subsets

### What is Subsets?

**Pattern:** Generate all subsets/combinations using backtracking

**When to Use:**
- Generate all subsets
- Combinations
- Permutations
- Power set

### Visual Pattern

```
Array: [1, 2, 3]

Subsets:
  []
  [1]
  [1, 2]
  [1, 2, 3]
  [1, 3]
  [2]
  [2, 3]
  [3]

Total: 2^3 = 8 subsets
```

### Example 1: Subsets

**Code:**
```java
List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
    // Add current subset
    result.add(new ArrayList<>(current));
    
    // Generate subsets
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);           // Choose
        backtrack(nums, i + 1, current, result);  // Explore
        current.remove(current.size() - 1);  // Unchoose (backtrack)
    }
}
```

**Time Complexity:** O(2^n) - exponential  
**Space Complexity:** O(n) - recursion stack

---

### Example 2: Subsets with Duplicates

**Problem:** Generate subsets when array has duplicates

**Code:**
```java
List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);  // Sort to handle duplicates
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current));
    
    for (int i = start; i < nums.length; i++) {
        // Skip duplicates
        if (i > start && nums[i] == nums[i - 1]) continue;
        
        current.add(nums[i]);
        backtrack(nums, i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

---

### Frequently Asked Questions

**Q1: Why backtrack?**
- Need to explore all possibilities
- Remove choice to try other options

**Q2: Time complexity?**
- O(2^n) - exponential, each element can be included or not

**Q3: How to avoid duplicates?**
- Sort array first
- Skip same elements at same level

---

## 9. Modified Binary Search

### What is Modified Binary Search?

**Pattern:** Binary search with variations (find first/last occurrence, search in rotated array)

**When to Use:**
- Sorted array
- Find target or closest
- Search in rotated array
- Find boundaries

### Visual Pattern

```
Array: [1, 3, 5, 7, 9, 11, 13], target = 7

Step 1: left=0, right=6, mid=3
  [1, 3, 5, 7, 9, 11, 13]
   ↑      ↑        ↑
  left   mid    right
  nums[3] = 7 == target ✅
```

### Example 1: Search in Rotated Array

**Problem:** Search in rotated sorted array

**Visual:**
```
Array: [4, 5, 6, 7, 0, 1, 2], target = 0

Step 1: left=0, right=6, mid=3
  [4, 5, 6, 7, 0, 1, 2]
   ↑      ↑        ↑
  left   mid    right
  nums[3] = 7, target = 0
  
  Left half [4,5,6,7] is sorted
  Is target in left? No (0 < 4 or 0 > 7)
  Search right half

Step 2: left=4, right=6, mid=5
  [4, 5, 6, 7, 0, 1, 2]
              ↑  ↑  ↑
            left mid right
  nums[5] = 1, target = 0
  
  Right half [1,2] is sorted
  Is target in right? No (0 < 1)
  Search left

Step 3: left=4, right=4, mid=4
  nums[4] = 0 == target ✅
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
                right = mid - 1;  // Search left
            } else {
                left = mid + 1;   // Search right
            }
        } else {
            // Right half is sorted
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;   // Search right
            } else {
                right = mid - 1;  // Search left
            }
        }
    }
    
    return -1;
}
```

---

### Example 2: Find First and Last Position

**Problem:** Find first and last occurrence of target

**Code:**
```java
int[] searchRange(int[] nums, int target) {
    int first = findFirst(nums, target);
    int last = findLast(nums, target);
    return new int[]{first, last};
}

int findFirst(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int first = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) {
            first = mid;
            right = mid - 1;  // Continue searching left
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return first;
}

int findLast(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int last = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) {
            last = mid;
            left = mid + 1;  // Continue searching right
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return last;
}
```

---

### Frequently Asked Questions

**Q1: When to use binary search?**
- Sorted array
- Need O(log n) time
- Finding boundaries

**Q2: How to avoid overflow?**
- Use `left + (right - left) / 2` instead of `(left + right) / 2`

**Q3: Time complexity?**
- O(log n) - halves search space each time

---

## 10. Top K Elements

### What is Top K Elements?

**Pattern:** Find K largest/smallest elements using heap

**When to Use:**
- Find top K elements
- Kth largest/smallest
- Frequency problems
- Priority queue problems

### Visual Pattern

```
Array: [3, 1, 5, 12, 2, 11], k=3

Min Heap (size k=3) for top 3 largest:

Step 1: Add 3
  [3]

Step 2: Add 1
  [1, 3]

Step 3: Add 5
  [1, 3, 5]  (heap full)

Step 4: Add 12
  12 > 1 (min), remove 1, add 12
  [3, 5, 12]

Step 5: Add 2
  2 < 3 (min), skip

Step 6: Add 11
  11 > 3 (min), remove 3, add 11
  [5, 11, 12]

Top 3: [12, 11, 5]
```

### Example 1: Kth Largest Element

**Code:**
```java
int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    
    for (int num : nums) {
        minHeap.offer(num);
        
        if (minHeap.size() > k) {
            minHeap.poll();  // Remove smallest
        }
    }
    
    return minHeap.peek();
}
```

**Time Complexity:** O(n log k)  
**Space Complexity:** O(k)

---

### Example 2: Top K Frequent Elements

**Problem:** Find K most frequent elements

**Code:**
```java
int[] topKFrequent(int[] nums, int k) {
    // Count frequencies
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) {
        freq.put(num, freq.getOrDefault(num, 0) + 1);
    }
    
    // Min heap by frequency
    PriorityQueue<Map.Entry<Integer, Integer>> heap = 
        new PriorityQueue<>((a, b) -> a.getValue() - b.getValue());
    
    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        heap.offer(entry);
        if (heap.size() > k) {
            heap.poll();
        }
    }
    
    // Extract results
    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) {
        result[i] = heap.poll().getKey();
    }
    
    return result;
}
```

---

### Frequently Asked Questions

**Q1: Min heap or max heap?**
- **Min heap** for K largest (keep K largest, remove smallest)
- **Max heap** for K smallest (keep K smallest, remove largest)

**Q2: Time complexity?**
- O(n log k) - n elements, heap size k

**Q3: When to use heap?**
- Need top K elements
- Kth largest/smallest
- Priority-based problems

---

## 11. K-way Merge

### What is K-way Merge?

**Pattern:** Merge K sorted lists/arrays efficiently

**When to Use:**
- Merge K sorted lists
- External sorting
- Merge intervals
- Priority queue problems

### Visual Pattern

```
Lists:
  L1: [1, 4, 5]
  L2: [1, 3, 4]
  L3: [2, 6]

Min Heap:
  Step 1: Add heads
    [1(L1), 1(L2), 2(L3)]
  
  Step 2: Pop 1(L1), add next from L1
    [1(L2), 2(L3), 4(L1)]
  
  Step 3: Pop 1(L2), add next from L2
    [2(L3), 3(L2), 4(L1)]
  
  Continue...
  
Result: [1, 1, 2, 3, 4, 4, 5, 6]
```

### Example 1: Merge K Sorted Lists

**Code:**
```java
ListNode mergeKLists(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    
    PriorityQueue<ListNode> minHeap = new PriorityQueue<>(
        (a, b) -> a.val - b.val
    );
    
    // Add all heads
    for (ListNode list : lists) {
        if (list != null) {
            minHeap.offer(list);
        }
    }
    
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (!minHeap.isEmpty()) {
        ListNode node = minHeap.poll();
        current.next = node;
        current = current.next;
        
        if (node.next != null) {
            minHeap.offer(node.next);
        }
    }
    
    return dummy.next;
}
```

**Time Complexity:** O(n log k) where n is total nodes, k is number of lists  
**Space Complexity:** O(k)

---

### Frequently Asked Questions

**Q1: Why use min heap?**
- Always get smallest element from all lists
- Efficient O(log k) insertion

**Q2: Time complexity?**
- O(n log k) - n total elements, k lists

---

## 12. Topological Sort

### What is Topological Sort?

**Pattern:** Order nodes in directed acyclic graph (DAG) such that all dependencies come before

**When to Use:**
- Course prerequisites
- Build order
- Task scheduling
- Dependency resolution

### Visual Pattern

```
Graph:
  1 → 2 → 4
  ↓   ↓
  3   5

Topological Order: [1, 2, 3, 4, 5] or [1, 3, 2, 5, 4]
(All dependencies come before)
```

### Example 1: Course Schedule

**Problem:** Check if can finish all courses given prerequisites

**Code:**
```java
boolean canFinish(int numCourses, int[][] prerequisites) {
    // Build graph and in-degree
    List<List<Integer>> graph = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    
    for (int i = 0; i < numCourses; i++) {
        graph.add(new ArrayList<>());
    }
    
    for (int[] pre : prerequisites) {
        graph.get(pre[1]).add(pre[0]);  // pre[1] → pre[0]
        inDegree[pre[0]]++;
    }
    
    // BFS: Start with courses with no prerequisites
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) {
            queue.offer(i);
        }
    }
    
    int count = 0;
    while (!queue.isEmpty()) {
        int course = queue.poll();
        count++;
        
        for (int next : graph.get(course)) {
            inDegree[next]--;
            if (inDegree[next] == 0) {
                queue.offer(next);
            }
        }
    }
    
    return count == numCourses;  // Can finish if all courses processed
}
```

**Time Complexity:** O(V + E)  
**Space Complexity:** O(V + E)

---

### Frequently Asked Questions

**Q1: What is in-degree?**
- Number of incoming edges to a node

**Q2: Why BFS?**
- Process nodes level by level
- Start with nodes having no dependencies

**Q3: How to detect cycle?**
- If count < numCourses, there's a cycle (can't process all nodes)

---

## 13. Backtracking

### What is Backtracking?

**Pattern:** Try all possibilities, backtrack when solution is invalid

**When to Use:**
- Generate all combinations
- Permutations
- N-Queens
- Sudoku solver
- Path finding

### Visual Pattern

```
Generate permutations of [1, 2, 3]:

Start: []
  Choose 1: [1]
    Choose 2: [1, 2]
      Choose 3: [1, 2, 3] ✅
      Backtrack: [1, 2]
    Backtrack: [1]
    Choose 3: [1, 3]
      Choose 2: [1, 3, 2] ✅
  Backtrack: []
  Choose 2: [2]
    ...
```

### Example 1: Permutations

**Code:**
```java
List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, List<Integer> current, List<List<Integer>> result) {
    // Base case: permutation complete
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    // Try all possibilities
    for (int num : nums) {
        if (current.contains(num)) continue;  // Skip if already used
        
        current.add(num);           // Choose
        backtrack(nums, current, result);  // Explore
        current.remove(current.size() - 1);  // Unchoose (backtrack)
    }
}
```

---

### Example 2: N-Queens

**Problem:** Place N queens on N×N board so no two attack each other

**Code:**
```java
List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    char[][] board = new char[n][n];
    for (char[] row : board) {
        Arrays.fill(row, '.');
    }
    backtrack(board, 0, result);
    return result;
}

void backtrack(char[][] board, int row, List<List<String>> result) {
    if (row == board.length) {
        result.add(construct(board));
        return;
    }
    
    for (int col = 0; col < board.length; col++) {
        if (isValid(board, row, col)) {
            board[row][col] = 'Q';        // Choose
            backtrack(board, row + 1, result);  // Explore
            board[row][col] = '.';        // Unchoose
        }
    }
}

boolean isValid(char[][] board, int row, int col) {
    // Check column
    for (int i = 0; i < row; i++) {
        if (board[i][col] == 'Q') return false;
    }
    
    // Check diagonals
    for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
        if (board[i][j] == 'Q') return false;
    }
    for (int i = row - 1, j = col + 1; i >= 0 && j < board.length; i--, j++) {
        if (board[i][j] == 'Q') return false;
    }
    
    return true;
}
```

---

### Frequently Asked Questions

**Q1: What is backtracking?**
- Try all possibilities
- Backtrack (undo) when solution is invalid
- Find all valid solutions

**Q2: Time complexity?**
- Usually exponential O(2^n) or O(n!)

**Q3: Key steps?**
1. Choose
2. Explore (recurse)
3. Unchoose (backtrack)

---

## 14. Dynamic Programming Patterns

### What is Dynamic Programming?

**Pattern:** Solve problems by breaking into subproblems and storing results

**When to Use:**
- Overlapping subproblems
- Optimal substructure
- Optimization problems
- Counting problems

### Key Patterns:

1. **1D DP** - Fibonacci, Climbing Stairs
2. **2D DP** - Longest Common Subsequence, Edit Distance
3. **Knapsack** - 0/1 Knapsack, Unbounded Knapsack
4. **DP on Strings** - Palindrome, Substring problems
5. **DP on Trees** - Tree DP problems

### Example 1: Fibonacci (1D DP)

**Visual:**
```
fib(5) = fib(4) + fib(3)
  = (fib(3) + fib(2)) + (fib(2) + fib(1))
  = ...

Memoization:
  dp[0] = 0
  dp[1] = 1
  dp[2] = 1
  dp[3] = 2
  dp[4] = 3
  dp[5] = 5
```

**Code:**
```java
int fibonacci(int n) {
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
**Space Complexity:** O(n) - can optimize to O(1)

---

### Example 2: Longest Common Subsequence (2D DP)

**Problem:** Find longest common subsequence between two strings

**Visual:**
```
s1 = "abcde", s2 = "ace"

DP Table:
      ""  a  c  e
  ""  0   0  0  0
  a   0   1  1  1
  b   0   1  1  1
  c   0   1  2  2
  d   0   1  2  2
  e   0   1  2  3

Answer: 3 ("ace")
```

**Code:**
```java
int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length();
    int n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[m][n];
}
```

**Time Complexity:** O(m × n)  
**Space Complexity:** O(m × n) - can optimize to O(min(m, n))

---

### Example 3: 0/1 Knapsack

**Problem:** Maximize value with weight constraint

**Visual:**
```
Items: [(weight, value)]
  Item 1: (1, 1)
  Item 2: (3, 4)
  Item 3: (4, 5)
  Item 4: (5, 7)

Capacity: 7

DP Table (capacity × items):
       0  1  2  3  4  5  6  7
Item1 0  1  1  1  1  1  1  1