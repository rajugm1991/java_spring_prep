# Pattern 7: Tree BFS/DFS 🌳

## Overview

**Tree BFS/DFS** patterns traverse trees using breadth-first or depth-first approaches. Essential for tree problems.

## When to Use

✅ **Use BFS when:**
- Level-order traversal
- Finding shortest path
- Level-based problems

✅ **Use DFS when:**
- Path problems
- Tree construction
- Recursive problems

## Pattern Template

```java
public class TreeTraversal {
    
    // BFS Template
    public List<List<Integer>> levelOrder(TreeNode root) {
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
    
    // DFS Template (Preorder)
    public void preorder(TreeNode root) {
        if (root == null) return;
        
        // Process root
        process(root);
        
        // Recurse left
        preorder(root.left);
        
        // Recurse right
        preorder(root.right);
    }
    
    // DFS Template (Inorder)
    public void inorder(TreeNode root) {
        if (root == null) return;
        
        inorder(root.left);
        process(root);
        inorder(root.right);
    }
    
    // DFS Template (Postorder)
    public void postorder(TreeNode root) {
        if (root == null) return;
        
        postorder(root.left);
        postorder(root.right);
        process(root);
    }
}
```

---

## Core Problems

### Problem 1: Binary Tree Level Order Traversal

**LeetCode:** [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

**Solution:**
```java
public List<List<Integer>> levelOrder(TreeNode root) {
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

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### Problem 2: Maximum Depth of Binary Tree

**LeetCode:** [104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

**Solution (DFS):**
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    
    int leftDepth = maxDepth(root.left);
    int rightDepth = maxDepth(root.right);
    
    return Math.max(leftDepth, rightDepth) + 1;
}
```

**Solution (BFS):**
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    int depth = 0;
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        depth++;
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    
    return depth;
}
```

---

### Problem 3: Same Tree

**LeetCode:** [100. Same Tree](https://leetcode.com/problems/same-tree/)

**Solution:**
```java
public boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null && q == null) return true;
    if (p == null || q == null) return false;
    if (p.val != q.val) return false;
    
    return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
}
```

---

### Problem 4: Symmetric Tree

**LeetCode:** [101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)

**Solution:**
```java
public boolean isSymmetric(TreeNode root) {
    return isMirror(root, root);
}

private boolean isMirror(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null) return true;
    if (t1 == null || t2 == null) return false;
    
    return (t1.val == t2.val) 
        && isMirror(t1.right, t2.left) 
        && isMirror(t1.left, t2.right);
}
```

---

## Key Takeaways

✅ BFS uses Queue, DFS uses recursion/stack  
✅ BFS: level-order, shortest path  
✅ DFS: path problems, tree construction  
✅ Time: O(n), Space: O(h) for DFS, O(w) for BFS

---

**Next Pattern:** [Subsets Pattern](./08_SUBSETS_PATTERN.md)

