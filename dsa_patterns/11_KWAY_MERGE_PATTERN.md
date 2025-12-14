# Pattern 11: K-way Merge 🔀

## Overview

**K-way Merge** merges K sorted arrays/lists into one sorted array/list using heaps.

## When to Use

✅ **Use K-way Merge when:**
- Merging K sorted arrays
- Merging K sorted linked lists
- External sorting
- Merge multiple sorted streams

## Pattern Template

```java
public class KWayMerge {
    
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;
        
        PriorityQueue<ListNode> minHeap = new PriorityQueue<>(
            (a, b) -> a.val - b.val
        );
        
        // Add first node of each list
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
}
```

---

## Core Problems

### Problem 1: Merge K Sorted Lists

**LeetCode:** [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

**Solution:**
```java
public ListNode mergeKLists(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    
    PriorityQueue<ListNode> minHeap = new PriorityQueue<>(
        (a, b) -> a.val - b.val
    );
    
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

### Problem 2: Merge K Sorted Arrays

**Problem:** Merge K sorted arrays into one sorted array.

**Solution:**
```java
public int[] mergeKSortedArrays(int[][] arrays) {
    PriorityQueue<int[]> minHeap = new PriorityQueue<>(
        (a, b) -> a[0] - b[0]
    );
    
    int totalSize = 0;
    for (int[] array : arrays) {
        if (array.length > 0) {
            minHeap.offer(new int[]{array[0], 0, array.length});
            totalSize += array.length;
        }
    }
    
    int[] result = new int[totalSize];
    int index = 0;
    
    while (!minHeap.isEmpty()) {
        int[] current = minHeap.poll();
        int value = current[0];
        int arrayIndex = current[1];
        int arrayLength = current[2];
        
        result[index++] = value;
        
        if (arrayIndex + 1 < arrayLength) {
            minHeap.offer(new int[]{
                arrays[arrayIndex][arrayIndex + 1],
                arrayIndex + 1,
                arrayLength
            });
        }
    }
    
    return result;
}
```

---

## Key Takeaways

✅ Use min heap to always get smallest element  
✅ Time: O(n log k)  
✅ Space: O(k)  
✅ Add next element from same list after polling

---

**Next Pattern:** [Topological Sort Pattern](./12_TOPOLOGICAL_SORT_PATTERN.md)

