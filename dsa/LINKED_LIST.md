# DSA Quick Cheat Sheet - Linked List 🔗

> Daily revision guide for linked list data structures and algorithms

---

## Table of Contents
1. [What is Linked List?](#what-is-linked-list)
2. [Types of Linked Lists](#types-of-linked-lists)
3. [Basic Operations](#basic-operations)
4. [Common Problems](#common-problems)
5. [Two Pointer Technique](#two-pointer-technique)
6. [Fast and Slow Pointers](#fast-and-slow-pointers)
7. [Frequently Asked Questions](#frequently-asked-questions)

---

## What is Linked List?

A **Linked List** is a linear data structure where elements are stored in nodes, and each node points to the next node.

### Key Characteristics:
- **Dynamic Size**: Grows and shrinks at runtime
- **Non-contiguous Memory**: Nodes can be anywhere in memory
- **Sequential Access**: Must traverse from head to find elements
- **No Index**: Can't access by index like arrays

### Node Structure:

```java
class ListNode {
    int val;
    ListNode next;
    
    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}
```

### Visual Representation:

```
Singly Linked List:

Head → [10] → [20] → [30] → [40] → null
        ↑      ↑      ↑      ↑
       Node1  Node2  Node3  Node4
```

**Key Points:**
- **Head**: First node (entry point)
- **Tail**: Last node (points to null)
- **Next Pointer**: Points to next node
- **Null**: End of list

---

## Types of Linked Lists

### 1. Singly Linked List

**Each node points to the next node only**

```
Head → [10] → [20] → [30] → null
```

**Traversal:** Forward only (head to tail)

### 2. Doubly Linked List

**Each node points to both next and previous nodes**

```java
class DoublyListNode {
    int val;
    DoublyListNode next;
    DoublyListNode prev;
}
```

**Visual:**
```
null ← [10] ⇄ [20] ⇄ [30] → null
        ↑      ↑      ↑
      Head   Node2  Tail
```

**Traversal:** Forward and backward

### 3. Circular Linked List

**Last node points back to head (forms a circle)**

```
Head → [10] → [20] → [30] ─┐
        ↑                  │
        └───────────────────┘
```

**Traversal:** Can loop infinitely (need to track visited nodes)

---

## Basic Operations

### 1. Create Linked List

**From Array:**
```java
ListNode createLinkedList(int[] arr) {
    if (arr.length == 0) return null;
    
    ListNode head = new ListNode(arr[0]);
    ListNode current = head;
    
    for (int i = 1; i < arr.length; i++) {
        current.next = new ListNode(arr[i]);
        current = current.next;
    }
    
    return head;
}
```

**Visual:**
```
Array: [10, 20, 30, 40]

Step 1: head = [10]
Step 2: head → [10] → [20]
Step 3: head → [10] → [20] → [30]
Step 4: head → [10] → [20] → [30] → [40] → null
```

### 2. Traverse Linked List

**Print all nodes:**
```java
void traverse(ListNode head) {
    ListNode current = head;
    
    while (current != null) {
        System.out.print(current.val + " → ");
        current = current.next;
    }
    System.out.println("null");
}
```

**Visual:**
```
Head → [10] → [20] → [30] → null
        ↑
      current

Step 1: Print 10, move to next
Step 2: Print 20, move to next
Step 3: Print 30, move to next
Step 4: current = null, stop
```

### 3. Insert at Head

**Add node at the beginning**

```java
ListNode insertAtHead(ListNode head, int val) {
    ListNode newNode = new ListNode(val);
    newNode.next = head;
    return newNode;  // New head
}
```

**Visual:**
```
Before:
Head → [10] → [20] → null

After inserting 5:
Head → [5] → [10] → [20] → null
        ↑
      newNode
```

### 4. Insert at Tail

**Add node at the end**

```java
ListNode insertAtTail(ListNode head, int val) {
    ListNode newNode = new ListNode(val);
    
    if (head == null) {
        return newNode;
    }
    
    ListNode current = head;
    while (current.next != null) {
        current = current.next;
    }
    
    current.next = newNode;
    return head;
}
```

**Visual:**
```
Before:
Head → [10] → [20] → null
                    ↑
                  current

After inserting 30:
Head → [10] → [20] → [30] → null
```

### 5. Delete Node

**Delete node with given value**

```java
ListNode deleteNode(ListNode head, int val) {
    if (head == null) return null;
    
    // If head needs to be deleted
    if (head.val == val) {
        return head.next;
    }
    
    ListNode current = head;
    while (current.next != null) {
        if (current.next.val == val) {
            current.next = current.next.next;
            return head;
        }
        current = current.next;
    }
    
    return head;
}
```

**Visual:**
```
Before (delete 20):
Head → [10] → [20] → [30] → null
        ↑      ↑
      current  toDelete

After:
Head → [10] → [30] → null
        ↑
      current.next = current.next.next
```

### 6. Find Length

**Count number of nodes**

```java
int length(ListNode head) {
    int count = 0;
    ListNode current = head;
    
    while (current != null) {
        count++;
        current = current.next;
    }
    
    return count;
}
```

**Recursive:**
```java
int length(ListNode head) {
    if (head == null) return 0;
    return 1 + length(head.next);
}
```

---

## Common Problems

### 1. Reverse Linked List

**Problem:** Reverse a linked list

**Approach 1: Iterative (Three Pointers)**

```java
ListNode reverse(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    ListNode next = null;
    
    while (current != null) {
        next = current.next;      // Save next
        current.next = prev;      // Reverse link
        prev = current;           // Move prev
        current = next;           // Move current
    }
    
    return prev;  // New head
}
```

**Visual Step-by-Step:**
```
Original: Head → [10] → [20] → [30] → null

Step 1: prev=null, current=[10], next=[20]
        null ← [10]    [20] → [30] → null
        ↑      ↑       ↑
       prev  current  next

Step 2: prev=[10], current=[20], next=[30]
        null ← [10] ← [20]    [30] → null
               ↑      ↑       ↑
              prev  current  next

Step 3: prev=[20], current=[30], next=null
        null ← [10] ← [20] ← [30]    null
                      ↑      ↑
                     prev  current

Step 4: prev=[30], current=null
        null ← [10] ← [20] ← [30] ← null
                            ↑
                           prev (new head)
```

**Approach 2: Recursive**

```java
ListNode reverse(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    
    ListNode newHead = reverse(head.next);
    head.next.next = head;  // Reverse link
    head.next = null;        // Break old link
    
    return newHead;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) iterative, O(n) recursive

---

### 2. Detect Cycle in Linked List

**Problem:** Check if linked list has a cycle

**Approach: Floyd's Cycle Detection (Fast & Slow Pointers)**

```java
boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) {
        return false;
    }
    
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;        // Move 1 step
        fast = fast.next.next;   // Move 2 steps
        
        if (slow == fast) {
            return true;  // Cycle detected
        }
    }
    
    return false;
}
```

**Visual:**
```
Cycle: Head → [10] → [20] → [30] ─┐
                  ↑                │
                  └────────────────┘

Step 1: slow=[10], fast=[10]
Step 2: slow=[20], fast=[30]
Step 3: slow=[30], fast=[20] (fast loops back)
Step 4: slow=[10], fast=[10] → slow == fast ✅ Cycle!
```

**Why it works:**
- If no cycle: fast reaches null
- If cycle: fast and slow will meet (fast catches up)

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 3. Find Middle Node

**Problem:** Find middle node of linked list

**Approach: Fast & Slow Pointers**

```java
ListNode findMiddle(ListNode head) {
    if (head == null) return null;
    
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;        // Move 1 step
        fast = fast.next.next;   // Move 2 steps
    }
    
    return slow;
}
```

**Visual:**
```
List: Head → [10] → [20] → [30] → [40] → [50] → null

Step 1: slow=[10], fast=[10]
Step 2: slow=[20], fast=[30]
Step 3: slow=[30], fast=[50]
Step 4: fast.next=null, stop
        Return slow=[30] ✅
```

**For even length (return second middle):**
```
List: Head → [10] → [20] → [30] → [40] → null

Step 1: slow=[10], fast=[10]
Step 2: slow=[20], fast=[30]
Step 3: fast.next=null, stop
        Return slow=[20] ✅
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 4. Merge Two Sorted Lists

**Problem:** Merge two sorted linked lists into one sorted list

**Approach: Two Pointers**

```java
ListNode mergeTwoLists(ListNode list1, ListNode list2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (list1 != null && list2 != null) {
        if (list1.val <= list2.val) {
            current.next = list1;
            list1 = list1.next;
        } else {
            current.next = list2;
            list2 = list2.next;
        }
        current = current.next;
    }
    
    // Append remaining nodes
    current.next = (list1 != null) ? list1 : list2;
    
    return dummy.next;
}
```

**Visual:**
```
List1: [1] → [3] → [5] → null
List2: [2] → [4] → [6] → null

Step 1: Compare 1 and 2 → Add 1
        dummy → [1]
        
Step 2: Compare 3 and 2 → Add 2
        dummy → [1] → [2]
        
Step 3: Compare 3 and 4 → Add 3
        dummy → [1] → [2] → [3]
        
... Continue until both lists are merged

Result: [1] → [2] → [3] → [4] → [5] → [6] → null
```

**Time Complexity:** O(n + m)  
**Space Complexity:** O(1)

---

### 5. Remove Nth Node from End

**Problem:** Remove the nth node from the end of list

**Approach: Two Pointers (Fast & Slow)**

```java
ListNode removeNthFromEnd(ListNode head, int n) {
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
        fast = fast.next;
        slow = slow.next;
    }
    
    // Remove node
    slow.next = slow.next.next;
    
    return dummy.next;
}
```

**Visual:**
```
List: Head → [1] → [2] → [3] → [4] → [5] → null
Remove: 2nd from end (node with value 4)

Step 1: Move fast 3 steps ahead
        slow=[0], fast=[3]
        
Step 2: Move both until fast=null
        slow=[3], fast=null
        
Step 3: slow.next = slow.next.next
        Remove [4]

Result: [1] → [2] → [3] → [5] → null
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 6. Intersection of Two Linked Lists

**Problem:** Find intersection node of two linked lists

**Approach: Two Pointers**

```java
ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;
    
    ListNode a = headA;
    ListNode b = headB;
    
    // When one pointer reaches end, switch to other list
    // They will meet at intersection (or both null)
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    
    return a;
}
```

**Visual:**
```
List A: [1] → [2] → [3] → [8] → [9] → null
List B:        [4] → [5] → [8] → [9] → null
                      ↑
                  Intersection

Pointer a: [1] → [2] → [3] → [8] → [9] → null → [4] → [5] → [8]
Pointer b: [4] → [5] → [8] → [9] → null → [1] → [2] → [3] → [8]

They meet at [8] ✅
```

**Time Complexity:** O(m + n)  
**Space Complexity:** O(1)

---

### 7. Palindrome Linked List

**Problem:** Check if linked list is palindrome

**Approach:**
1. Find middle
2. Reverse second half
3. Compare both halves

```java
boolean isPalindrome(ListNode head) {
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

ListNode reverse(ListNode head) {
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

**Visual:**
```
Original: [1] → [2] → [2] → [1] → null

Step 1: Find middle (slow at second [2])
Step 2: Reverse second half: [1] → [2] → null
Step 3: Compare:
        [1] == [1] ✅
        [2] == [2] ✅
        Palindrome! ✅
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 8. Add Two Numbers

**Problem:** Add two numbers represented as linked lists

**Example:**
```
List1: [2] → [4] → [3]  (represents 342)
List2: [5] → [6] → [4]  (represents 465)
Result: [7] → [0] → [8]  (represents 807)
```

**Code:**
```java
ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    int carry = 0;
    
    while (l1 != null || l2 != null || carry != 0) {
        int sum = carry;
        
        if (l1 != null) {
            sum += l1.val;
            l1 = l1.next;
        }
        
        if (l2 != null) {
            sum += l2.val;
            l2 = l2.next;
        }
        
        carry = sum / 10;
        current.next = new ListNode(sum % 10);
        current = current.next;
    }
    
    return dummy.next;
}
```

**Visual:**
```
Step 1: 2 + 5 = 7, carry = 0 → [7]
Step 2: 4 + 6 = 10, carry = 1 → [0]
Step 3: 3 + 4 + 1 = 8, carry = 0 → [8]
Result: [7] → [0] → [8]
```

---

## Two Pointer Technique

### When to Use:
- Finding middle node
- Detecting cycles
- Finding nth node from end
- Palindrome checking
- Intersection problems

### Pattern:
```java
ListNode slow = head;
ListNode fast = head;

while (fast != null && fast.next != null) {
    slow = slow.next;      // Move 1 step
    fast = fast.next.next; // Move 2 steps
}
```

---

## Fast and Slow Pointers

### Common Use Cases:

#### 1. Find Middle
```java
// Slow will be at middle when fast reaches end
```

#### 2. Detect Cycle
```java
// If fast catches slow, there's a cycle
```

#### 3. Find Cycle Start
```java
ListNode detectCycleStart(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    
    // Find meeting point
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }
    
    if (fast == null || fast.next == null) return null;
    
    // Move one pointer to head, move both 1 step
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    
    return slow;  // Cycle start
}
```

---

## Frequently Asked Questions

### Q1: How do you reverse a linked list?

**Answer:**
"I use the iterative approach with three pointers: prev, current, and next. I traverse the list, reversing each link as I go. The key is to save the next node before breaking the link, then reverse the current link, and move all pointers forward. Time complexity is O(n) and space is O(1)."

**Code:**
```java
ListNode reverse(ListNode head) {
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

---

### Q2: How do you detect a cycle in a linked list?

**Answer:**
"I use Floyd's cycle detection algorithm with fast and slow pointers. The slow pointer moves one step at a time, while the fast pointer moves two steps. If there's a cycle, the fast pointer will eventually catch up to the slow pointer. If there's no cycle, the fast pointer will reach null. This is O(n) time and O(1) space."

**Code:**
```java
boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    
    return false;
}
```

---

### Q3: How do you find the middle node of a linked list?

**Answer:**
"I use the fast and slow pointer technique. The slow pointer moves one step while the fast pointer moves two steps. When the fast pointer reaches the end, the slow pointer will be at the middle. This works in a single pass with O(n) time and O(1) space."

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

### Q4: How do you merge two sorted linked lists?

**Answer:**
"I use a dummy node to simplify the code and two pointers to traverse both lists. I compare values at each step and add the smaller one to the result. After one list is exhausted, I append the remaining nodes from the other list. This is O(n + m) time and O(1) space."

**Code:**
```java
ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            current.next = l1;
            l1 = l1.next;
        } else {
            current.next = l2;
            l2 = l2.next;
        }
        current = current.next;
    }
    
    current.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

---

### Q5: How do you remove the nth node from the end?

**Answer:**
"I use two pointers with a gap of n nodes between them. I first move the fast pointer n+1 steps ahead, then move both pointers until the fast pointer reaches the end. At this point, the slow pointer will be just before the node to remove. I then update slow.next to skip the target node. I use a dummy node to handle edge cases like removing the head."

**Code:**
```java
ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy;
    ListNode slow = dummy;
    
    for (int i = 0; i <= n; i++) {
        fast = fast.next;
    }
    
    while (fast != null) {
        fast = fast.next;
        slow = slow.next;
    }
    
    slow.next = slow.next.next;
    return dummy.next;
}
```

---

### Q6: What's the difference between array and linked list?

**Answer:**
"Arrays store elements in contiguous memory with fixed size, allowing O(1) random access by index. Linked lists store elements in non-contiguous memory with dynamic size, requiring O(n) sequential access. Arrays are better for random access and cache efficiency, while linked lists are better for frequent insertions/deletions and dynamic sizing."

| Aspect | Array | Linked List |
|--------|-------|-------------|
| **Memory** | Contiguous | Non-contiguous |
| **Size** | Fixed | Dynamic |
| **Access** | O(1) by index | O(n) sequential |
| **Insert/Delete** | O(n) | O(1) at known position |
| **Cache** | Better | Worse |

---

### Q7: How do you check if a linked list is palindrome?

**Answer:**
"I use a three-step approach: First, find the middle using fast and slow pointers. Second, reverse the second half. Third, compare both halves node by node. If all nodes match, it's a palindrome. I restore the list afterward if needed. This is O(n) time and O(1) space."

**Code:**
```java
boolean isPalindrome(ListNode head) {
    // Find middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    
    // Reverse second half
    ListNode secondHalf = reverse(slow.next);
    ListNode firstHalf = head;
    
    // Compare
    while (secondHalf != null) {
        if (firstHalf.val != secondHalf.val) return false;
        firstHalf = firstHalf.next;
        secondHalf = secondHalf.next;
    }
    
    return true;
}
```

---

### Q8: How do you find the intersection of two linked lists?

**Answer:**
"I use a clever two-pointer technique. I traverse both lists, and when one pointer reaches the end, I switch it to the other list. This way, both pointers will traverse the same total distance and meet at the intersection point (if it exists). If they both reach null, there's no intersection. This is O(m + n) time and O(1) space."

**Code:**
```java
ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA;
    ListNode b = headB;
    
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    
    return a;
}
```

---

### Q9: How do you add two numbers represented as linked lists?

**Answer:**
"I traverse both lists simultaneously, adding corresponding digits along with any carry from the previous addition. I create a new node for each digit of the result. I handle cases where lists have different lengths and ensure I process any remaining carry. This is O(max(m, n)) time and O(max(m, n)) space for the result."

**Code:**
```java
ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    int carry = 0;
    
    while (l1 != null || l2 != null || carry != 0) {
        int sum = carry;
        if (l1 != null) { sum += l1.val; l1 = l1.next; }
        if (l2 != null) { sum += l2.val; l2 = l2.next; }
        
        carry = sum / 10;
        current.next = new ListNode(sum % 10);
        current = current.next;
    }
    
    return dummy.next;
}
```

---

### Q10: What's the time complexity of inserting at the end of a linked list?

**Answer:**
"Inserting at the end of a linked list is O(n) because I need to traverse from the head to the tail to find the last node. However, if I maintain a tail pointer, it becomes O(1). For inserting at the beginning, it's always O(1) since I just update the head pointer."

**Without tail pointer:**
```java
// O(n) - need to traverse to end
ListNode insertAtEnd(ListNode head, int val) {
    ListNode newNode = new ListNode(val);
    if (head == null) return newNode;
    
    ListNode current = head;
    while (current.next != null) {
        current = current.next;  // O(n) traversal
    }
    current.next = newNode;
    return head;
}
```

**With tail pointer:**
```java
// O(1) - direct access to tail
class LinkedList {
    ListNode head;
    ListNode tail;
    
    void insertAtEnd(int val) {
        ListNode newNode = new ListNode(val);
        if (tail == null) {
            head = tail = newNode;
        } else {
            tail.next = newNode;
            tail = newNode;  // O(1)
        }
    }
}
```

---

## Quick Reference Table

| Operation | Time Complexity | Space Complexity | Notes |
|-----------|----------------|-------------------|-------|
| **Access by Index** | O(n) | O(1) | Must traverse |
| **Insert at Head** | O(1) | O(1) | Update head |
| **Insert at Tail** | O(n) | O(1) | O(1) with tail pointer |
| **Delete Node** | O(n) | O(1) | Find then delete |
| **Reverse** | O(n) | O(1) | Iterative approach |
| **Find Middle** | O(n) | O(1) | Fast & slow pointers |
| **Detect Cycle** | O(n) | O(1) | Floyd's algorithm |
| **Merge Two Lists** | O(n + m) | O(1) | Two pointers |
| **Palindrome Check** | O(n) | O(1) | Reverse half |

---

## Memory Tips 🧠

### Linked List Basics
- **L**inked **L**ist = **L**inear + **L**inks
- **H**ead = **H**ome (starting point)
- **N**ull = **N**o more (end marker)

### Two Pointers
- **F**ast & **S**low = **F**ind **S**tuff (middle, cycle)
- **G**ap of **N** = **G**et **N**th from end

### Common Patterns
- **D**ummy node = **D**on't worry about head edge cases
- **P**rev, **C**urrent, **N**ext = **P**erfect for **R**everse

---

## Daily Revision Checklist ✅

- [ ] Create and traverse linked list
- [ ] Insert at head and tail
- [ ] Delete node
- [ ] Reverse linked list (iterative & recursive)
- [ ] Find middle node
- [ ] Detect cycle
- [ ] Merge two sorted lists
- [ ] Remove nth from end
- [ ] Check palindrome
- [ ] Find intersection
- [ ] Add two numbers

---

## Common Mistakes to Avoid ❌

1. **Forgetting to check null**: Always check `head == null` and `current.next == null`
2. **Losing reference**: Save `next` before modifying links
3. **Not updating head**: When inserting/deleting at head, remember to return new head
4. **Infinite loops**: Ensure loop condition properly checks for null
5. **Memory leaks**: In languages without GC, properly free deleted nodes

---

## Practice Problems

### Easy:
- [ ] Reverse Linked List
- [ ] Merge Two Sorted Lists
- [ ] Delete Node in Linked List
- [ ] Remove Duplicates from Sorted List

### Medium:
- [ ] Add Two Numbers
- [ ] Remove Nth Node From End
- [ ] Swap Nodes in Pairs
- [ ] Rotate List
- [ ] Partition List

### Hard:
- [ ] Reverse Nodes in k-Group
- [ ] Merge k Sorted Lists
- [ ] Copy List with Random Pointer
- [ ] LRU Cache (uses doubly linked list)

---

**Happy Coding! 🔗**

