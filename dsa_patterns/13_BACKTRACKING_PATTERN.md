# Pattern 13: Backtracking 🔙

## Overview

**Backtracking** explores all possible solutions by building candidates incrementally and abandoning ("backtracking") when it determines the candidate cannot lead to a valid solution.

## When to Use

✅ **Use Backtracking when:**
- Generating all permutations/combinations
- Constraint satisfaction problems
- Path finding with constraints
- N-Queens, Sudoku
- Subset/combination problems

## Pattern Template

```java
public class Backtracking {
    
    public List<List<Integer>> solve(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }
    
    private void backtrack(int[] nums, int start,
                           List<Integer> current,
                           List<List<Integer>> result) {
        // Base case: solution found
        if (isValidSolution(current)) {
            result.add(new ArrayList<>(current));
            return;
        }
        
        // Pruning: skip invalid paths
        if (!isValid(current)) {
            return;
        }
        
        // Explore all possibilities
        for (int i = start; i < nums.length; i++) {
            // Choose
            current.add(nums[i]);
            
            // Explore
            backtrack(nums, i + 1, current, result);
            
            // Unchoose (backtrack)
            current.remove(current.size() - 1);
        }
    }
    
    private boolean isValid(List<Integer> current) {
        // Check if current path is valid
        return true;
    }
    
    private boolean isValidSolution(List<Integer> current) {
        // Check if current is a solution
        return false;
    }
}
```

---

## Core Problems

### Problem 1: Permutations

**LeetCode:** [46. Permutations](https://leetcode.com/problems/permutations/)

**Solution:**
```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, new ArrayList<>(), new boolean[nums.length], result);
    return result;
}

private void backtrack(int[] nums, List<Integer> current,
                      boolean[] used,
                      List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        
        current.add(nums[i]);
        used[i] = true;
        backtrack(nums, current, used, result);
        used[i] = false;
        current.remove(current.size() - 1);
    }
}
```

**Time Complexity:** O(n! × n)  
**Space Complexity:** O(n)

---

### Problem 2: N-Queens

**LeetCode:** [51. N-Queens](https://leetcode.com/problems/n-queens/)

**Solution:**
```java
public List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    char[][] board = new char[n][n];
    for (char[] row : board) {
        Arrays.fill(row, '.');
    }
    backtrack(board, 0, result);
    return result;
}

private void backtrack(char[][] board, int row,
                       List<List<String>> result) {
    if (row == board.length) {
        result.add(construct(board));
        return;
    }
    
    for (int col = 0; col < board.length; col++) {
        if (isValid(board, row, col)) {
            board[row][col] = 'Q';
            backtrack(board, row + 1, result);
            board[row][col] = '.';
        }
    }
}

private boolean isValid(char[][] board, int row, int col) {
    // Check column
    for (int i = 0; i < row; i++) {
        if (board[i][col] == 'Q') return false;
    }
    
    // Check diagonal \
    for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
        if (board[i][j] == 'Q') return false;
    }
    
    // Check diagonal /
    for (int i = row - 1, j = col + 1; i >= 0 && j < board.length; i--, j++) {
        if (board[i][j] == 'Q') return false;
    }
    
    return true;
}

private List<String> construct(char[][] board) {
    List<String> result = new ArrayList<>();
    for (char[] row : board) {
        result.add(new String(row));
    }
    return result;
}
```

---

### Problem 3: Word Search

**LeetCode:** [79. Word Search](https://leetcode.com/problems/word-search/)

**Solution:**
```java
public boolean exist(char[][] board, String word) {
    int m = board.length;
    int n = board[0].length;
    
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (backtrack(board, i, j, word, 0)) {
                return true;
            }
        }
    }
    
    return false;
}

private boolean backtrack(char[][] board, int i, int j,
                         String word, int index) {
    if (index == word.length()) {
        return true;
    }
    
    if (i < 0 || i >= board.length || 
        j < 0 || j >= board[0].length ||
        board[i][j] != word.charAt(index)) {
        return false;
    }
    
    char temp = board[i][j];
    board[i][j] = '#'; // Mark as visited
    
    boolean found = backtrack(board, i + 1, j, word, index + 1) ||
                   backtrack(board, i - 1, j, word, index + 1) ||
                   backtrack(board, i, j + 1, word, index + 1) ||
                   backtrack(board, i, j - 1, word, index + 1);
    
    board[i][j] = temp; // Restore
    return found;
}
```

---

## Key Takeaways

✅ Three steps: Choose, Explore, Unchoose  
✅ Prune invalid paths early  
✅ Restore state after backtracking  
✅ Time: Usually exponential

---

**Next Pattern:** [Dynamic Programming Pattern](./14_DYNAMIC_PROGRAMMING_PATTERN.md)

