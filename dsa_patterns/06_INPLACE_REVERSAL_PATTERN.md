# Pattern 6: In-place Reversal 🔄

## Overview

**In-place Reversal** reverses linked lists or parts of arrays without using extra space. Essential for space-optimized solutions.

## When to Use

✅ **Use In-place Reversal when:**
- Reversing linked lists
- Reversing parts of arrays
- Palindrome problems
- Rotating arrays
- Space optimization needed

## Pattern Template

```java
public class InPlaceReversal {
    
    // Reverse entire linked list
    public ListNode reverse(ListNode head) {
        ListNode prev = null;
        ListNode current = head;
        
        while (current != null) {
            ListNode next = current.next;
            current.next = prev;
            prev = current;
            current = next;
        }
        
        return prev;
    }
    
    // Reverse array in-place
    public void reverseArray(int[] arr, int start, int end) {
        while (start < end) {
            int temp = arr[start];
            arr[start] = arr[end];
            arr[end] = temp;
            start++;
            end--;
        }
    }
}
```

---

## Core Problems

### Problem 1: Reverse Linked List

**LeetCode:** [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

**Solution:**
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    
    while (current != null) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    
    return prev;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 2: Reverse Linked List II

**LeetCode:** [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)

**Problem:** Reverse nodes from position left to right.

**Solution:**
```java
public ListNode reverseBetween(ListNode head, int left, int right) {
    if (head == null || left == right) return head;
    
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode prev = dummy;
    
    // Move to left position
    for (int i = 0; i < left - 1; i++) {
        prev = prev.next;
    }
    
    ListNode current = prev.next;
    ListNode next = current.next;
    
    // Reverse
    for (int i = 0; i < right - left; i++) {
        current.next = next.next;
        next.next = prev.next;
        prev.next = next;
        next = current.next;
    }
    
    return dummy.next;
}
```

---

### Problem 3: Reverse Nodes in k-Group

**LeetCode:** [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

**Solution:**
```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode current = head;
    int count = 0;
    
    // Check if k nodes exist
    while (current != null && count < k) {
        current = current.next;
        count++;
    }
    
    if (count == k) {
        current = reverseKGroup(current, k);
        
        // Reverse current k-group
        while (count > 0) {
            ListNode next = head.next;
            head.next = current;
            current = head;
            head = next;
            count--;
        }
        head = current;
    }
    
    return head;
}
```

---

## Key Takeaways

✅ Use three pointers: prev, current, next  
✅ Reverse in O(n) time, O(1) space  
✅ Handle edge cases: empty list, single node

---

**Next Pattern:** [Tree BFS/DFS Pattern](./07_TREE_BFS_DFS_PATTERN.md)

