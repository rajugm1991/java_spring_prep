# Pattern 3: Fast & Slow Pointers 🐢🐰

## Overview

The **Fast & Slow Pointers** pattern uses two pointers moving at different speeds to solve problems efficiently. The slow pointer moves one step at a time, while the fast pointer moves two steps. This pattern is perfect for detecting cycles and finding middle elements.

## When to Use

✅ **Use Fast & Slow Pointers when:**
- Detecting cycles in linked lists
- Finding middle element
- Finding kth element from end
- Palindrome checking in linked lists
- Cycle detection in arrays

## Pattern Template

```java
public class FastSlowPointers {
    
    // Template: Cycle Detection
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;
        
        ListNode slow = head;
        ListNode fast = head;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;        // Move 1 step
            fast = fast.next.next;   // Move 2 steps
            
            if (slow == fast) {
                return true; // Cycle detected
            }
        }
        
        return false;
    }
}
```

---

## Core Problems

### Problem 1: Linked List Cycle

**LeetCode:** [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

**Problem:** Detect if linked list has a cycle.

**Solution:**
```java
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) return false;
    
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        
        if (slow == fast) {
            return true;
        }
    }
    
    return false;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

**Why it works:** If there's a cycle, fast pointer will eventually catch slow pointer.

---

### Problem 2: Middle of Linked List

**LeetCode:** [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)

**Problem:** Find middle node of linked list.

**Example:**
```
Input: [1,2,3,4,5]
Output: [3,4,5]
```

**Solution:**
```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    
    return slow;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 3: Remove Nth Node From End

**LeetCode:** [19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

**Problem:** Remove nth node from end.

**Example:**
```
Input: head = [1,2,3,4,5], n = 2
Output: [1,2,3,5]
```

**Solution:**
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    
    ListNode fast = dummy;
    ListNode slow = dummy;
    
    // Move fast n+1 steps ahead
    for (int i = 0; i <= n; i++) {
        fast = fast.next;
    }
    
    // Move both until fast reaches end
    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    
    // Remove node
    slow.next = slow.next.next;
    
    return dummy.next;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Problem 4: Linked List Cycle II (Find Start of Cycle)

**LeetCode:** [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)

**Problem:** Find node where cycle begins.

**Solution:**
```java
public ListNode detectCycle(ListNode head) {
    if (head == null || head.next == null) return null;
    
    ListNode slow = head;
    ListNode fast = head;
    
    // Find meeting point
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        
        if (slow == fast) {
            break; // Cycle found
        }
    }
    
    if (fast == null || fast.next == null) {
        return null; // No cycle
    }
    
    // Find start of cycle
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    
    return slow;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

**Key Insight:** Distance from head to cycle start = Distance from meeting point to cycle start

---

### Problem 5: Palindrome Linked List

**LeetCode:** [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

**Problem:** Check if linked list is palindrome.

**Solution:**
```java
public boolean isPalindrome(ListNode head) {
    if (head == null || head.next == null) return true;
    
    // Find middle
    ListNode slow = head;
    ListNode fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    
    // Reverse second half
    ListNode secondHalf = reverse(slow.next);
    ListNode firstHalf = head;
    
    // Compare
    while (secondHalf != null) {
        if (firstHalf.val != secondHalf.val) {
            return false;
        }
        firstHalf = firstHalf.next;
        secondHalf = secondHalf.next;
    }
    
    return true;
}

private ListNode reverse(ListNode head) {
    ListNode prev = null;
    while (head != null) {
        ListNode next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Advanced Problems

### Problem 6: Reorder List

**LeetCode:** [143. Reorder List](https://leetcode.com/problems/reorder-list/)

**Problem:** Reorder list: L0 → L1 → ... → Ln-1 → Ln to L0 → Ln → L1 → Ln-1 → ...

**Solution:**
```java
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;
    
    // Find middle
    ListNode slow = head;
    ListNode fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    
    // Reverse second half
    ListNode secondHalf = reverse(slow.next);
    slow.next = null;
    
    // Merge
    ListNode first = head;
    ListNode second = secondHalf;
    while (second != null) {
        ListNode temp1 = first.next;
        ListNode temp2 = second.next;
        first.next = second;
        second.next = temp1;
        first = temp1;
        second = temp2;
    }
}

private ListNode reverse(ListNode head) {
    ListNode prev = null;
    while (head != null) {
        ListNode next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Frequently Asked Interview Questions

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

---

## Key Takeaways

✅ Fast pointer moves 2x speed of slow pointer  
✅ Detects cycles in O(n) time, O(1) space  
✅ Can find middle element efficiently  
✅ Useful for palindrome and reordering problems

---

**Next Pattern:** [Merge Intervals Pattern](./04_MERGE_INTERVALS_PATTERN.md)

