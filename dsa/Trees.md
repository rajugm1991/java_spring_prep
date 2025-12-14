# DSA Quick Cheat Sheet - Trees 🌳

> Daily revision guide for tree data structures and algorithms

---

## Table of Contents
1. [BST (Binary Search Tree)](#bst-binary-search-tree)
2. [AVL Tree](#avl-tree)
3. [Segment Tree](#segment-tree)
4. [BFS (Breadth-First Search)](#bfs-breadth-first-search)
5. [DFS (Depth-First Search)](#dfs-depth-first-search)

---

## BST (Binary Search Tree)

### What is BST?

A tree where:
- **Left child** < Parent
- **Right child** > Parent
- All nodes follow this rule

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Left < Parent < Right ✅
```

### Key Concepts

#### 1. Insert Left and Right Values

**Rule:** 
- If value < node → Go LEFT
- If value > node → Go RIGHT

```
Insert 7 into BST:

Start at root (10)
├─ 7 < 10? YES → Go LEFT
├─ 7 > 5? YES → Go RIGHT
└─ Insert 7 as right child of 5

       10
      /  \
     5    15
    / \
   3   7  ← Inserted here
```

**Code:**
```java
Node insert(Node root, int value) {
    if (root == null) {
        return new Node(value);
    }
    
    if (value < root.val) {
        root.left = insert(root.left, value);  // Go LEFT
    } else {
        root.right = insert(root.right, value); // Go RIGHT
    }
    
    return root;
}
```

#### 2. Find Height of Node

**Height:** Longest path from node to leaf

```
       10 (height: 3)
      /  \
     5    15 (height: 1)
    / \
   3   7 (height: 0)

Height of 10 = 3 (path: 10 → 5 → 3)
Height of 15 = 1 (path: 15 → null)
Height of 7 = 0 (leaf node)
```

**Code:**
```java
int height(Node node) {
    if (node == null) {
        return -1;  // or 0, depending on definition
    }
    
    int leftHeight = height(node.left);
    int rightHeight = height(node.right);
    
    return 1 + Math.max(leftHeight, rightHeight);
}
```

#### 3. Balanced Tree

**Balanced:** Height difference between left and right ≤ 1

```
Balanced ✅:
       10
      /  \
     5    15
    / \
   3   7
   
   Height diff: |1 - 1| = 0 ≤ 1 ✅

Not Balanced ❌:
       10
      /
     5
    /
   3
   
   Height diff: |2 - 0| = 2 > 1 ❌
```

**Code:**
```java
boolean isBalanced(Node root) {
    if (root == null) return true;
    
    int leftHeight = height(root.left);
    int rightHeight = height(root.right);
    
    if (Math.abs(leftHeight - rightHeight) > 1) {
        return false;
    }
    
    return isBalanced(root.left) && isBalanced(root.right);
}
```

#### 4. Siblings

**Siblings:** Nodes with same parent

```
       10
      /  \
     5    15  ← Siblings (same parent: 10)
    / \   / \
   3   7 12  20
   
   3 and 7 are siblings (parent: 5)
   12 and 20 are siblings (parent: 15)
```

**Code:**
```java
boolean areSiblings(Node root, int a, int b) {
    if (root == null) return false;
    
    // Check if both are children of current node
    boolean leftHasA = hasNode(root.left, a);
    boolean leftHasB = hasNode(root.left, b);
    boolean rightHasA = hasNode(root.right, a);
    boolean rightHasB = hasNode(root.right, b);
    
    // Both in left or both in right = siblings
    return (leftHasA && leftHasB) || (rightHasA && rightHasB);
}
```

#### 5. Level

**Level:** Distance from root (root is level 0)

```
Level 0:        10
              /  \
Level 1:      5    15
            / \   / \
Level 2:   3   7 12  20
```

**Code:**
```java
int getLevel(Node root, int target, int level) {
    if (root == null) return -1;
    
    if (root.val == target) {
        return level;
    }
    
    int left = getLevel(root.left, target, level + 1);
    if (left != -1) return left;
    
    return getLevel(root.right, target, level + 1);
}
```

---

## AVL Tree

### What is AVL?

**Self-balancing BST** - Automatically maintains balance

**Balance Factor:** `height(left) - height(right)`
- Must be: -1, 0, or 1

```
AVL Tree ✅:
       10 (BF: 0)
      /  \
     5    15 (BF: 0)
    / \
   3   7 (BF: 0)

Not AVL ❌:
       10 (BF: 2)
      /
     5 (BF: 1)
    /
   3 (BF: 0)
```

### Rotations

#### Left Rotate

**When:** Right subtree is taller (BF < -1)

```
Before (Right Heavy):
   10
     \
      15
        \
         20

After Left Rotate:
      15
     /  \
   10    20
```

**Code:**
```java
Node leftRotate(Node x) {
    Node y = x.right;
    Node T2 = y.left;
    
    // Rotate
    y.left = x;
    x.right = T2;
    
    return y;  // New root
}
```

#### Right Rotate

**When:** Left subtree is taller (BF > 1)

```
Before (Left Heavy):
       10
      /
     5
    /
   3

After Right Rotate:
      5
     / \
    3   10
```

**Code:**
```java
Node rightRotate(Node y) {
    Node x = y.left;
    Node T2 = x.right;
    
    // Rotate
    x.right = y;
    y.left = T2;
    
    return x;  // New root
}
```

### Insert with Balancing

```java
Node insert(Node node, int value) {
    // 1. Normal BST insert
    if (node == null) {
        return new Node(value);
    }
    
    if (value < node.val) {
        node.left = insert(node.left, value);
    } else {
        node.right = insert(node.right, value);
    }
    
    // 2. Get balance factor
    int balance = getBalance(node);
    
    // 3. Left Left Case (Right Rotate)
    if (balance > 1 && value < node.left.val) {
        return rightRotate(node);
    }
    
    // 4. Right Right Case (Left Rotate)
    if (balance < -1 && value > node.right.val) {
        return leftRotate(node);
    }
    
    // 5. Left Right Case (Left Rotate, then Right Rotate)
    if (balance > 1 && value > node.left.val) {
        node.left = leftRotate(node.left);
        return rightRotate(node);
    }
    
    // 6. Right Left Case (Right Rotate, then Left Rotate)
    if (balance < -1 && value < node.right.val) {
        node.right = rightRotate(node.right);
        return leftRotate(node);
    }
    
    return node;
}
```

---

## Segment Tree

### What is Segment Tree?

Tree for **range queries** (sum, min, max, avg)

**Use Case:** Fast range queries on arrays

```
Array: [1, 3, 5, 7, 9, 11]

Segment Tree:
             36 (0-5)
           /        \
       9 (0-2)      27 (3-5)
      /    \       /    \
   4 (0-1) 5(2)  16(3-4) 11(5)
  /   \
1(0)  3(1)
```

### Range Sum Query

**Query:** Sum of elements from index `l` to `r`

```
Array: [1, 3, 5, 7, 9, 11]
Query: sum(2, 4) = 5 + 7 + 9 = 21

Segment Tree helps find this in O(log n) time
```

**Code:**
```java
class SegmentTree {
    int[] tree;
    int n;
    
    SegmentTree(int[] arr) {
        n = arr.length;
        tree = new int[4 * n];
        build(arr, 0, 0, n - 1);
    }
    
    void build(int[] arr, int node, int start, int end) {
        if (start == end) {
            tree[node] = arr[start];
        } else {
            int mid = (start + end) / 2;
            build(arr, 2*node + 1, start, mid);
            build(arr, 2*node + 2, mid + 1, end);
            tree[node] = tree[2*node + 1] + tree[2*node + 2];
        }
    }
    
    int query(int l, int r) {
        return query(0, 0, n - 1, l, r);
    }
    
    int query(int node, int start, int end, int l, int r) {
        if (r < start || end < l) {
            return 0;  // Outside range
        }
        if (l <= start && end <= r) {
            return tree[node];  // Completely inside
        }
        
        int mid = (start + end) / 2;
        int left = query(2*node + 1, start, mid, l, r);
        int right = query(2*node + 2, mid + 1, end, l, r);
        return left + right;
    }
}
```

### Range Average Query

**Average = Sum / Count**

```java
double rangeAverage(int l, int r) {
    int sum = query(l, r);
    int count = r - l + 1;
    return (double) sum / count;
}
```

---

## BFS (Breadth-First Search)

### What is BFS?

**Level-by-level** traversal using **Queue**

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

BFS Order: 10 → 5 → 15 → 3 → 7 → 12 → 20
(Level by level)
```

### How BFS Works

```
1. Start with root in queue
2. While queue not empty:
   - Dequeue node
   - Print node
   - Enqueue left child
   - Enqueue right child
```

**Visual:**
```
Queue: [10]
Print: 10
Queue: [5, 15]

Queue: [5, 15]
Print: 5
Queue: [15, 3, 7]

Queue: [15, 3, 7]
Print: 15
Queue: [3, 7, 12, 20]

... and so on
```

**Code:**
```java
void bfs(Node root) {
    if (root == null) return;
    
    Queue<Node> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        Node node = queue.poll();
        System.out.print(node.val + " ");
        
        if (node.left != null) {
            queue.offer(node.left);
        }
        if (node.right != null) {
            queue.offer(node.right);
        }
    }
}
```

### Level Order Traversal

**Print each level on separate line**

```java
void levelOrder(Node root) {
    if (root == null) return;
    
    Queue<Node> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        
        for (int i = 0; i < levelSize; i++) {
            Node node = queue.poll();
            System.out.print(node.val + " ");
            
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        
        System.out.println();  // New line for each level
    }
}

Output:
10
5 15
3 7 12 20
```

### Is Cousin

**Cousins:** Same level, different parents

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

3 and 12 are cousins (level 2, different parents)
5 and 15 are NOT cousins (same parent)
```

**Code:**
```java
boolean isCousin(Node root, int x, int y) {
    if (root == null) return false;
    
    // Find level and parent of both nodes
    int[] xInfo = findLevelAndParent(root, x, 0, -1);
    int[] yInfo = findLevelAndParent(root, y, 0, -1);
    
    // Same level but different parents
    return xInfo[0] == yInfo[0] && xInfo[1] != yInfo[1];
}

int[] findLevelAndParent(Node root, int target, int level, int parent) {
    if (root == null) return new int[]{-1, -1};
    
    if (root.val == target) {
        return new int[]{level, parent};
    }
    
    int[] left = findLevelAndParent(root.left, target, level + 1, root.val);
    if (left[0] != -1) return left;
    
    return findLevelAndParent(root.right, target, level + 1, root.val);
}
```

### Symmetric Tree

**Symmetric:** Mirror image of itself

```
Symmetric ✅:
       1
      / \
     2   2
    / \ / \
   3  4 4  3

Not Symmetric ❌:
       1
      / \
     2   2
      \   \
       3   3
```

**Code:**
```java
boolean isSymmetric(Node root) {
    if (root == null) return true;
    return isMirror(root.left, root.right);
}

boolean isMirror(Node left, Node right) {
    if (left == null && right == null) return true;
    if (left == null || right == null) return false;
    
    return left.val == right.val &&
           isMirror(left.left, right.right) &&
           isMirror(left.right, right.left);
}
```

---

## DFS (Depth-First Search)

### What is DFS?

**Go deep** before going wide

**Three Types:**
- **Pre-order:** Root → Left → Right
- **In-order:** Left → Root → Right
- **Post-order:** Left → Right → Root

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20
```

### Pre-order (Root → Left → Right)

**Order:** 10 → 5 → 3 → 7 → 15 → 12 → 20

**Code:**
```java
void preOrder(Node root) {
    if (root == null) return;
    
    System.out.print(root.val + " ");  // Root FIRST
    preOrder(root.left);                // Then Left
    preOrder(root.right);                // Then Right
}
```

### In-order (Left → Root → Right)

**Order:** 3 → 5 → 7 → 10 → 12 → 15 → 20

**Note:** For BST, in-order gives **sorted order**!

**Code:**
```java
void inOrder(Node root) {
    if (root == null) return;
    
    inOrder(root.left);                 // Left FIRST
    System.out.print(root.val + " ");   // Then Root
    inOrder(root.right);                 // Then Right
}
```

### Post-order (Left → Right → Root)

**Order:** 3 → 7 → 5 → 12 → 20 → 15 → 10

**Code:**
```java
void postOrder(Node root) {
    if (root == null) return;
    
    postOrder(root.left);                // Left FIRST
    postOrder(root.right);                // Then Right
    System.out.print(root.val + " ");    // Root LAST
}
```

### Find Height (Hint: Post-order)

**Why Post-order?** Need children's height before calculating parent's height

```java
int height(Node root) {
    if (root == null) return -1;
    
    int leftHeight = height(root.left);   // Get left height
    int rightHeight = height(root.right); // Get right height
    
    return 1 + Math.max(leftHeight, rightHeight);  // Calculate parent
}
```

**Visual:**
```
       10 (height: 3)
      /  \
     5    15 (height: 1)
    / \
   3   7 (height: 0)

Post-order traversal:
- Process 3 first (height: 0)
- Process 7 (height: 0)
- Process 5 (height: 1 + max(0, 0) = 1)
- Process 15 (height: 1)
- Process 10 (height: 1 + max(1, 1) = 3)
```

### Diameter of Tree

**Diameter:** Longest path between any two nodes

**Hint:** Use post-order to find height, then calculate diameter

```java
int diameter = 0;

int diameterOfBinaryTree(Node root) {
    height(root);
    return diameter;
}

int height(Node root) {
    if (root == null) return -1;
    
    int leftHeight = height(root.left);
    int rightHeight = height(root.right);
    
    // Diameter through current node
    diameter = Math.max(diameter, leftHeight + rightHeight + 2);
    
    return 1 + Math.max(leftHeight, rightHeight);
}
```

### Invert Binary Tree (Post-order)

**Invert:** Swap left and right children

**Why Post-order?** Swap after processing children

```
Before:
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

After:
       10
      /  \
     15   5
    / \   / \
   20  12 7  3
```

**Code:**
```java
Node invertTree(Node root) {
    if (root == null) return null;
    
    // Post-order: Process children first
    invertTree(root.left);
    invertTree(root.right);
    
    // Then swap
    Node temp = root.left;
    root.left = root.right;
    root.right = temp;
    
    return root;
}
```

### Convert Sorted Array to BST

**Key:** Middle element is root, recursively build left and right

```java
Node sortedArrayToBST(int[] nums) {
    return buildBST(nums, 0, nums.length - 1);
}

Node buildBST(int[] nums, int left, int right) {
    if (left > right) return null;
    
    int mid = (left + right) / 2;
    Node root = new Node(nums[mid]);
    
    root.left = buildBST(nums, left, mid - 1);
    root.right = buildBST(nums, mid + 1, right);
    
    return root;
}
```

**Example:**
```
Array: [1, 3, 5, 7, 9, 11, 13]

Step 1: Mid = 7 → Root
       7
      / \
     
Step 2: Left: [1,3,5], Right: [9,11,13]
       7
      / \
     3   11
    / \  / \
   1  5 9  13
```

### Flatten Binary Tree to Linked List

**Flatten:** Convert tree to linked list (right pointers)

```
Before:
       1
      / \
     2   5
    / \   \
   3   4   6

After:
1 → 2 → 3 → 4 → 5 → 6
```

**Code:**
```java
Node prev = null;

void flatten(Node root) {
    if (root == null) return;
    
    // Post-order: Process right first, then left
    flatten(root.right);
    flatten(root.left);
    
    // Connect
    root.right = prev;
    root.left = null;
    prev = root;
}
```

### Valid BST

**Valid:** All nodes follow BST property

**Key:** Pass min and max bounds while traversing

```java
boolean isValidBST(Node root) {
    return isValid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

boolean isValid(Node root, long min, long max) {
    if (root == null) return true;
    
    // Check current node
    if (root.val <= min || root.val >= max) {
        return false;
    }
    
    // Check left (max becomes current value)
    // Check right (min becomes current value)
    return isValid(root.left, min, root.val) &&
           isValid(root.right, root.val, max);
}
```

**Visual:**
```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Check 10: (-∞, +∞) ✅
Check 5: (-∞, 10) ✅
Check 15: (10, +∞) ✅
Check 3: (-∞, 5) ✅
Check 7: (5, 10) ✅
Check 12: (10, 15) ✅
Check 20: (15, +∞) ✅
```

### Lowest Common Ancestor (LCA)

**LCA:** The deepest node that has both nodes as descendants

**Key Insight:** Use DFS to find both nodes, then backtrack to find common ancestor

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

LCA(3, 7) = 5  (Both in left subtree of 5)
LCA(12, 20) = 15  (Both in right subtree of 15)
LCA(3, 12) = 10  (One in left, one in right)
LCA(5, 7) = 5  (One is ancestor of other)
```

**Approach 1: Recursive DFS (Post-order)**

**Idea:** 
- If we find both nodes in left and right subtrees → current node is LCA
- If we find one node → return that node
- If we find nothing → return null

**Code:**
```java
Node lowestCommonAncestor(Node root, Node p, Node q) {
    // Base case: null or found one of the nodes
    if (root == null || root == p || root == q) {
        return root;
    }
    
    // Search in left and right subtrees
    Node left = lowestCommonAncestor(root.left, p, q);
    Node right = lowestCommonAncestor(root.right, p, q);
    
    // If both found in different subtrees → current node is LCA
    if (left != null && right != null) {
        return root;
    }
    
    // If found in one subtree → return that result
    return left != null ? left : right;
}
```

**Visual Explanation:**
```
Find LCA(3, 7):

       10
      /  \
     5    15
    / \   / \
   3   7 12  20

DFS Traversal:
1. Start at 10: Search left (5) and right (15)
2. At 5: Search left (3) and right (7)
   - Left returns 3 (found!)
   - Right returns 7 (found!)
   - Both found → 5 is LCA ✅
3. Return 5 to 10
4. Result: 5
```

**Approach 2: Path-based (Find paths, then compare)**

**Idea:** Find path to both nodes, then find last common node

**Code:**
```java
Node lowestCommonAncestor(Node root, Node p, Node q) {
    List<Node> pathP = new ArrayList<>();
    List<Node> pathQ = new ArrayList<>();
    
    // Find paths to both nodes
    findPath(root, p, pathP);
    findPath(root, q, pathQ);
    
    // Find last common node in paths
    int i = 0;
    while (i < pathP.size() && i < pathQ.size() && 
           pathP.get(i) == pathQ.get(i)) {
        i++;
    }
    
    return pathP.get(i - 1);
}

boolean findPath(Node root, Node target, List<Node> path) {
    if (root == null) return false;
    
    path.add(root);
    
    if (root == target) return true;
    
    if (findPath(root.left, target, path) || 
        findPath(root.right, target, path)) {
        return true;
    }
    
    path.remove(path.size() - 1);  // Backtrack
    return false;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(h) where h is height

---

### Kth Smallest Element in BST

**Problem:** Find the kth smallest element in a BST

**Key Insight:** In-order traversal of BST gives sorted order!

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

In-order: 3 → 5 → 7 → 10 → 12 → 15 → 20
1st smallest: 3
2nd smallest: 5
3rd smallest: 7
4th smallest: 10
```

**Approach 1: In-order Traversal with Counter**

**Idea:** Do in-order traversal, count nodes, return kth node

**Code:**
```java
int count = 0;
int result = -1;

int kthSmallest(Node root, int k) {
    count = 0;
    result = -1;
    inOrder(root, k);
    return result;
}

void inOrder(Node root, int k) {
    if (root == null) return;
    
    // Left first (smallest values)
    inOrder(root.left, k);
    
    // Process current node
    count++;
    if (count == k) {
        result = root.val;
        return;  // Found it!
    }
    
    // Right last (larger values)
    inOrder(root.right, k);
}
```

**Visual Explanation:**
```
Find 3rd smallest:

       10
      /  \
     5    15
    / \   / \
   3   7 12  20

In-order traversal:
1. Go to 3 (leftmost) → count = 1, not 3rd
2. Back to 5 → count = 2, not 3rd
3. Go to 7 → count = 3, found! ✅
   Return 7
```

**Approach 2: Iterative In-order (Stack)**

**Idea:** Use stack to simulate in-order traversal

**Code:**
```java
int kthSmallest(Node root, int k) {
    Stack<Node> stack = new Stack<>();
    Node current = root;
    int count = 0;
    
    while (current != null || !stack.isEmpty()) {
        // Go to leftmost node
        while (current != null) {
            stack.push(current);
            current = current.left;
        }
        
        // Process node
        current = stack.pop();
        count++;
        
        if (count == k) {
            return current.val;
        }
        
        // Go to right
        current = current.right;
    }
    
    return -1;
}
```

**Time Complexity:** O(h + k) where h is height  
**Space Complexity:** O(h) for recursion stack

**Optimization for Frequent Queries:** Use augmented BST with size of left subtree

---

### Path Sum

**Problem:** Check if there exists a path from root to leaf with given sum

**Key Insight:** Use DFS (pre-order) and subtract from sum as we go down

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Path sum = 18?
Path: 10 → 5 → 3 = 10 + 5 + 3 = 18 ✅
```

**Code:**
```java
boolean hasPathSum(Node root, int targetSum) {
    if (root == null) return false;
    
    // Leaf node: check if remaining sum equals node value
    if (root.left == null && root.right == null) {
        return root.val == targetSum;
    }
    
    // Check left and right with reduced sum
    return hasPathSum(root.left, targetSum - root.val) ||
           hasPathSum(root.right, targetSum - root.val);
}
```

**Visual Explanation:**
```
Check path sum = 18:

       10 (target: 18)
      /  \
     5    15
    / \   / \
   3   7 12  20

DFS:
1. At 10: target = 18, check left(5, 8) and right(15, 8)
2. At 5: target = 8, check left(3, 3) and right(7, 1)
3. At 3: target = 3, leaf node, 3 == 3 ✅
   Return true
```

**Variation: Find All Paths with Sum**

```java
List<List<Integer>> result = new ArrayList<>();

List<List<Integer>> pathSum(Node root, int targetSum) {
    result = new ArrayList<>();
    dfs(root, targetSum, new ArrayList<>());
    return result;
}

void dfs(Node root, int targetSum, List<Integer> path) {
    if (root == null) return;
    
    path.add(root.val);
    
    // Leaf node: check if sum matches
    if (root.left == null && root.right == null && root.val == targetSum) {
        result.add(new ArrayList<>(path));
    }
    
    // Recurse with reduced sum
    dfs(root.left, targetSum - root.val, path);
    dfs(root.right, targetSum - root.val, path);
    
    path.remove(path.size() - 1);  // Backtrack
}
```

---

### Maximum Path Sum

**Problem:** Find maximum path sum (path can start and end anywhere)

**Key Insight:** For each node, calculate:
1. Path through node (left + node + right)
2. Path ending at node (max of left/right + node)
3. Return path ending at node to parent

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Max path: 3 + 5 + 10 + 15 + 20 = 53
```

**Code:**
```java
int maxSum = Integer.MIN_VALUE;

int maxPathSum(Node root) {
    maxSum = Integer.MIN_VALUE;
    maxPathDown(root);
    return maxSum;
}

int maxPathDown(Node root) {
    if (root == null) return 0;
    
    // Get max path from left and right (ignore negative)
    int left = Math.max(0, maxPathDown(root.left));
    int right = Math.max(0, maxPathDown(root.right));
    
    // Path through current node
    maxSum = Math.max(maxSum, left + root.val + right);
    
    // Return max path ending at current node
    return root.val + Math.max(left, right);
}
```

**Visual Explanation:**
```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Post-order traversal:
1. At 3: left=0, right=0, sum=3, return 3
2. At 7: left=0, right=0, sum=7, return 7
3. At 5: left=3, right=7, sum=3+5+7=15, return 5+7=12
4. At 12: left=0, right=0, sum=12, return 12
5. At 20: left=0, right=0, sum=20, return 20
6. At 15: left=12, right=20, sum=12+15+20=47, return 15+20=35
7. At 10: left=12, right=35, sum=12+10+35=57, return 10+35=45
   Max sum = 57 ✅
```

---

### Count Complete Tree Nodes

**Problem:** Count nodes in a complete binary tree efficiently

**Complete Tree:** All levels filled except possibly last, filled left to right

**Key Insight:** Use binary search on last level

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Complete tree with 7 nodes
```

**Code:**
```java
int countNodes(Node root) {
    if (root == null) return 0;
    
    int leftHeight = getLeftHeight(root);
    int rightHeight = getRightHeight(root);
    
    // If heights equal → perfect tree
    if (leftHeight == rightHeight) {
        return (1 << leftHeight) - 1;  // 2^h - 1
    }
    
    // Otherwise, recurse
    return 1 + countNodes(root.left) + countNodes(root.right);
}

int getLeftHeight(Node root) {
    int height = 0;
    while (root != null) {
        height++;
        root = root.left;
    }
    return height;
}

int getRightHeight(Node root) {
    int height = 0;
    while (root != null) {
        height++;
        root = root.right;
    }
    return height;
}
```

**Time Complexity:** O(log² n) for complete tree

---

### Serialize and Deserialize Binary Tree

**Problem:** Convert tree to string and back

**Key Insight:** Use pre-order traversal with null markers

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Serialized: "10,5,3,#,#,7,#,#,15,12,#,#,20,#,#"
```

**Code:**
```java
// Serialize (Pre-order)
String serialize(Node root) {
    StringBuilder sb = new StringBuilder();
    buildString(root, sb);
    return sb.toString();
}

void buildString(Node root, StringBuilder sb) {
    if (root == null) {
        sb.append("#,");
        return;
    }
    
    sb.append(root.val).append(",");
    buildString(root.left, sb);
    buildString(root.right, sb);
}

// Deserialize
Node deserialize(String data) {
    Queue<String> queue = new LinkedList<>(Arrays.asList(data.split(",")));
    return buildTree(queue);
}

Node buildTree(Queue<String> queue) {
    String val = queue.poll();
    
    if (val.equals("#")) {
        return null;
    }
    
    Node root = new Node(Integer.parseInt(val));
    root.left = buildTree(queue);
    root.right = buildTree(queue);
    
    return root;
}
```

**Why Pre-order?** Root comes first, so we can build tree from left to right

---

### Binary Tree Right Side View

**Problem:** Return values visible from right side (top to bottom)

**Key Insight:** Use DFS, track level, keep only rightmost node at each level

```
       10
      /  \
     5    15
    / \   / \
   3   7 12  20

Right view: [10, 15, 20]
```

**Code:**
```java
List<Integer> rightSideView(Node root) {
    List<Integer> result = new ArrayList<>();
    rightView(root, result, 0);
    return result;
}

void rightView(Node root, List<Integer> result, int level) {
    if (root == null) return;
    
    // First node at this level (rightmost)
    if (level == result.size()) {
        result.add(root.val);
    }
    
    // Right first (to get rightmost)
    rightView(root.right, result, level + 1);
    rightView(root.left, result, level + 1);
}
```

**Visual:**
```
Level 0: 10 (first at level 0) → add 10
Level 1: 15 (first at level 1) → add 15
Level 2: 20 (first at level 2) → add 20
Result: [10, 15, 20]
```

---

## Quick Reference Table

## Quick Reference Table

| Concept | Key Point | Time Complexity |
|---------|-----------|----------------|
| **BST Insert** | Left < Root < Right | O(log n) |
| **BST Height** | Longest path to leaf | O(n) |
| **AVL Rotate** | Balance when |BF| > 1 | O(1) |
| **Segment Tree** | Range queries | O(log n) |
| **BFS** | Queue, level-by-level | O(n) |
| **DFS Pre-order** | Root → Left → Right | O(n) |
| **DFS In-order** | Left → Root → Right | O(n) |
| **DFS Post-order** | Left → Right → Root | O(n) |
| **LCA** | Lowest common ancestor | O(n) |
| **Kth Smallest BST** | In-order traversal | O(h + k) |
| **Path Sum** | DFS with sum reduction | O(n) |
| **Max Path Sum** | Post-order with max tracking | O(n) |
| **Right Side View** | DFS with level tracking | O(n) |

---

## Memory Tips 🧠

### BST
- **B**inary **S**earch: **S**maller **L**eft, **L**arger **R**ight
- **H**eight: **H**ow **H**igh from leaf

### AVL
- **A**lways **V**ery **L**evel (balanced)
- Rotate when |BF| > 1

### BFS
- **B**readth = **B**road = **Q**ueue
- Level by level

### DFS
- **D**epth = **D**own = **S**tack (recursion)
- **P**re: **P**arent first
- **I**n: **I**n between
- **P**ost: **P**arent last

---

## Daily Revision Checklist ✅

- [ ] BST: Insert, Height, Balanced
- [ ] AVL: Rotations, Balance Factor
- [ ] Segment Tree: Range queries
- [ ] BFS: Queue, Level order
- [ ] DFS: Pre, In, Post order
- [ ] Tree Problems: Diameter, Invert, Valid BST
- [ ] DFS Problems: LCA, Kth Smallest, Path Sum, Max Path Sum

---

**Happy Coding! 🌳**

