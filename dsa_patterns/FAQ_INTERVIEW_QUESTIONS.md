# DSA Interview Questions - Frequently Asked Questions 🎯

> Complete collection of frequently asked interview questions for all 14 DSA patterns

---

## Table of Contents

1. [Two Pointers FAQ](#1-two-pointers-faq)
2. [Sliding Window FAQ](#2-sliding-window-faq)
3. [Fast & Slow Pointers FAQ](#3-fast--slow-pointers-faq)
4. [Merge Intervals FAQ](#4-merge-intervals-faq)
5. [Cyclic Sort FAQ](#5-cyclic-sort-faq)
6. [In-place Reversal FAQ](#6-in-place-reversal-faq)
7. [Tree BFS/DFS FAQ](#7-tree-bfsdfs-faq)
8. [Subsets FAQ](#8-subsets-faq)
9. [Modified Binary Search FAQ](#9-modified-binary-search-faq)
10. [Top K Elements FAQ](#10-top-k-elements-faq)
11. [K-way Merge FAQ](#11-k-way-merge-faq)
12. [Topological Sort FAQ](#12-topological-sort-faq)
13. [Backtracking FAQ](#13-backtracking-faq)
14. [Dynamic Programming FAQ](#14-dynamic-programming-faq)

---

## 1. Two Pointers FAQ

### Q1: When should I use two pointers vs sliding window?

**Answer:**
- **Two Pointers:** Pointers move independently based on condition (sum, comparison)
- **Sliding Window:** Maintains a window, both pointers move together

**Example:**
- Two Pointers: Two Sum, Three Sum
- Sliding Window: Maximum Sum Subarray, Longest Substring

### Q2: Can two pointers work on unsorted arrays?

**Answer:**
- **Opposite ends pattern:** Requires sorted array
- **Same direction pattern:** Works on unsorted (removing duplicates, partitioning)
- Can sort first if needed (adds O(n log n))

### Q3: How to handle duplicates in two sum problems?

**Answer:**
```java
// After finding a match, skip duplicates
while (left < right && nums[left] == nums[left + 1]) left++;
while (left < right && nums[right] == nums[right - 1]) right--;
```

### Q4: What's the space complexity of two pointers?

**Answer:**
- Usually O(1) - only using a few variables
- O(n) if storing results (like in 3Sum returning all triplets)

### Q5: How do you decide which pointer to move?

**Answer:**
- Move pointer that helps achieve target condition
- If sum < target → move left (need larger value)
- If sum > target → move right (need smaller value)

---

## 2. Sliding Window FAQ

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

## 3. Fast & Slow Pointers FAQ

### Q1: Why does fast and slow pointer detect cycles?

**Answer:**
- If cycle exists, fast will eventually enter cycle
- Fast catches up to slow (relative speed = 1 step per iteration)
- They meet inside the cycle

### Q2: How to find cycle length?

**Answer:**
```java
// After finding meeting point, count steps to meet again
int cycleLength = 0;
ListNode current = slow;
do {
    current = current.next;
    cycleLength++;
} while (current != slow);
```

### Q3: What if fast pointer moves 3 steps?

**Answer:**
- Still works, but less efficient
- Fast pointer might skip over slow pointer
- Standard is 2 steps for simplicity

### Q4: How to find start of cycle?

**Answer:**
- After finding meeting point, move one pointer to head
- Move both one step at a time
- They meet at cycle start

---

## 4. Merge Intervals FAQ

### Q1: How to sort intervals?

**Answer:**
- **For merging:** Sort by start time
- **For scheduling:** Sort by end time (greedy)

### Q2: How to check if two intervals overlap?

**Answer:**
```java
boolean overlaps(int[] a, int[] b) {
    return a[1] >= b[0]; // a ends after b starts
}
```

### Q3: How to merge overlapping intervals?

**Answer:**
```java
if (current[1] >= next[0]) {
    // Overlapping - merge
    current[1] = Math.max(current[1], next[1]);
}
```

### Q4: How to find minimum rooms for meetings?

**Answer:**
- Sort start and end times separately
- Use two pointers, increment rooms when start < end
- Maximum rooms needed = maximum concurrent meetings

---

## 5. Cyclic Sort FAQ

### Q1: When to use cyclic sort?

**Answer:**
- Array contains numbers in range [1, n] or [0, n-1]
- Need to find missing/duplicate numbers
- Need to sort with O(1) space

### Q2: How does cyclic sort work?

**Answer:**
- Place each number at its correct index
- If number is already at correct index, move to next
- Otherwise, swap with number at correct index

### Q3: What's the time complexity?

**Answer:**
- O(n) - each element is placed at most once
- Space: O(1)

---

## 6. In-place Reversal FAQ

### Q1: How to reverse a linked list?

**Answer:**
```java
ListNode prev = null;
ListNode current = head;
while (current != null) {
    ListNode next = current.next;
    current.next = prev;
    prev = current;
    current = next;
}
return prev;
```

### Q2: How to reverse part of linked list?

**Answer:**
- Find start and end positions
- Reverse the segment
- Connect back to original list

### Q3: What's the space complexity?

**Answer:**
- O(1) - only using a few pointers

---

## 7. Tree BFS/DFS FAQ

### Q1: When to use BFS vs DFS?

**Answer:**
- **BFS:** Level-order, shortest path, level-based problems
- **DFS:** Path problems, tree construction, recursive problems

### Q2: What's the space complexity?

**Answer:**
- **BFS:** O(w) where w is maximum width
- **DFS:** O(h) where h is height (recursion stack)

### Q3: How to do iterative DFS?

**Answer:**
- Use stack instead of recursion
- Push children in reverse order

### Q4: How to find maximum depth?

**Answer:**
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
}
```

---

## 8. Subsets FAQ

### Q1: How to generate all subsets?

**Answer:**
- Use backtracking
- For each element: include or exclude
- Time: O(2^n), Space: O(n)

### Q2: How to handle duplicates?

**Answer:**
- Sort array first
- Skip duplicates: `if (i > start && nums[i] == nums[i-1]) continue;`

### Q3: What's the difference between subsets and combinations?

**Answer:**
- **Subsets:** All possible subsets (2^n)
- **Combinations:** Subsets of specific size (C(n,k))

---

## 9. Modified Binary Search FAQ

### Q1: How to find first occurrence?

**Answer:**
- When found, continue searching left
- Store result and update when found

### Q2: How to search in rotated array?

**Answer:**
- Check which half is sorted
- Determine if target is in sorted half
- Adjust search accordingly

### Q3: How to find peak element?

**Answer:**
- Compare mid with mid+1
- If mid > mid+1, peak is in left half
- Otherwise, peak is in right half

---

## 10. Top K Elements FAQ

### Q1: Min heap vs Max heap?

**Answer:**
- **Min heap:** For K largest (keep K smallest, remove when size > K)
- **Max heap:** For K smallest (keep K largest, remove when size > K)

### Q2: What's the time complexity?

**Answer:**
- O(n log k) - n elements, heap size k
- Space: O(k)

### Q3: When to use heap vs sorting?

**Answer:**
- **Heap:** When K << n (K is much smaller than n)
- **Sorting:** When K is close to n

---

## 11. K-way Merge FAQ

### Q1: How to merge K sorted lists?

**Answer:**
- Use min heap
- Add first node of each list
- Poll smallest, add its next node

### Q2: What's the time complexity?

**Answer:**
- O(n log k) where n is total nodes, k is number of lists

### Q3: Alternative approach?

**Answer:**
- Divide and conquer: merge pairs recursively
- Time: O(n log k), Space: O(1) for linked lists

---

## 12. Topological Sort FAQ

### Q1: How to detect cycle using topological sort?

**Answer:**
- If count of processed nodes != total nodes, cycle exists

### Q2: Kahn's algorithm vs DFS?

**Answer:**
- **Kahn's (BFS):** Use in-degree, queue
- **DFS:** Use recursion, stack

### Q3: When is topological sort possible?

**Answer:**
- Only for Directed Acyclic Graph (DAG)
- If cycle exists, topological sort not possible

---

## 13. Backtracking FAQ

### Q1: What's the three-step process?

**Answer:**
1. **Choose:** Add current choice
2. **Explore:** Recurse with choice
3. **Unchoose:** Remove choice (backtrack)

### Q2: How to prune invalid paths?

**Answer:**
- Check validity before exploring
- Return early if path is invalid

### Q3: What's the time complexity?

**Answer:**
- Usually exponential: O(2^n) or O(n!)
- Depends on branching factor

---

## 14. Dynamic Programming FAQ

### Q1: How to identify DP problems?

**Answer:**
- Optimization or counting problems
- Can be broken into subproblems
- Subproblems overlap
- Optimal substructure property

### Q2: Top-down vs Bottom-up?

**Answer:**
- **Top-down (Memoization):** Recursive, easier to think
- **Bottom-up (Tabulation):** Iterative, usually faster

### Q3: How to optimize space?

**Answer:**
- If only previous states needed, use variables
- Example: Fibonacci only needs prev2 and prev1

### Q4: How to find state transition?

**Answer:**
- Identify what changes between states
- Define state variables
- Find relationship between states

### Q5: 1D vs 2D DP?

**Answer:**
- **1D:** Single variable changes (position, amount)
- **2D:** Two variables change (two strings, grid position)

---

## General Interview Tips

### 1. Problem-Solving Approach

1. **Understand:** Read problem carefully, ask questions
2. **Examples:** Work through examples
3. **Approach:** Discuss approach before coding
4. **Code:** Write clean, readable code
5. **Test:** Walk through with examples
6. **Optimize:** Discuss time/space complexity

### 2. Common Mistakes to Avoid

- ❌ Jumping to code without understanding
- ❌ Not considering edge cases
- ❌ Not optimizing time/space
- ❌ Not explaining thought process
- ❌ Not testing with examples

### 3. Time Complexity Cheat Sheet

- O(1): Constant
- O(log n): Binary search
- O(n): Single loop
- O(n log n): Sorting
- O(n²): Nested loops
- O(2^n): Subsets, backtracking
- O(n!): Permutations

### 4. Space Complexity Cheat Sheet

- O(1): Variables
- O(n): Array, string
- O(h): Recursion stack (tree height)
- O(w): Queue (tree width)

---

## Practice Problems by Pattern

### Two Pointers
- [ ] Two Sum
- [ ] Three Sum
- [ ] Container With Most Water
- [ ] Trapping Rain Water

### Sliding Window
- [ ] Longest Substring Without Repeating Characters
- [ ] Minimum Window Substring
- [ ] Permutation in String
- [ ] Subarray Product Less Than K

### Fast & Slow Pointers
- [ ] Linked List Cycle
- [ ] Middle of Linked List
- [ ] Palindrome Linked List
- [ ] Reorder List

### Merge Intervals
- [ ] Merge Intervals
- [ ] Insert Interval
- [ ] Meeting Rooms
- [ ] Meeting Rooms II

### Cyclic Sort
- [ ] Missing Number
- [ ] Find All Missing Numbers
- [ ] Find Duplicate Number

### In-place Reversal
- [ ] Reverse Linked List
- [ ] Reverse Linked List II
- [ ] Reverse Nodes in k-Group

### Tree BFS/DFS
- [ ] Level Order Traversal
- [ ] Maximum Depth
- [ ] Same Tree
- [ ] Symmetric Tree

### Subsets
- [ ] Subsets
- [ ] Subsets II
- [ ] Combinations
- [ ] Permutations

### Modified Binary Search
- [ ] Search in Rotated Sorted Array
- [ ] Find First and Last Position
- [ ] Find Peak Element

### Top K Elements
- [ ] Kth Largest Element
- [ ] Top K Frequent Elements
- [ ] K Closest Points

### K-way Merge
- [ ] Merge K Sorted Lists
- [ ] Merge K Sorted Arrays

### Topological Sort
- [ ] Course Schedule
- [ ] Course Schedule II

### Backtracking
- [ ] N-Queens
- [ ] Word Search
- [ ] Sudoku Solver

### Dynamic Programming
- [ ] Fibonacci
- [ ] Climbing Stairs
- [ ] Coin Change
- [ ] Longest Increasing Subsequence
- [ ] House Robber

---

**Good luck with your interviews! 🚀**

