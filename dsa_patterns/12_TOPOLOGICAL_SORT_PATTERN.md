# Pattern 12: Topological Sort 📊

## Overview

**Topological Sort** orders nodes in a directed acyclic graph (DAG) such that for every edge (u, v), u comes before v.

## When to Use

✅ **Use Topological Sort when:**
- Course prerequisites
- Task scheduling
- Build systems
- Dependency resolution
- DAG ordering

## Pattern Template

```java
public class TopologicalSort {
    
    // Kahn's Algorithm (BFS)
    public List<Integer> topologicalSort(int numCourses, int[][] prerequisites) {
        // Build graph and in-degree
        List<List<Integer>> graph = new ArrayList<>();
        int[] inDegree = new int[numCourses];
        
        for (int i = 0; i < numCourses; i++) {
            graph.add(new ArrayList<>());
        }
        
        for (int[] edge : prerequisites) {
            graph.get(edge[1]).add(edge[0]);
            inDegree[edge[0]]++;
        }
        
        // BFS: Start with nodes having in-degree 0
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) {
                queue.offer(i);
            }
        }
        
        List<Integer> result = new ArrayList<>();
        while (!queue.isEmpty()) {
            int node = queue.poll();
            result.add(node);
            
            for (int neighbor : graph.get(node)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }
        
        // Check if all nodes processed (no cycle)
        return result.size() == numCourses ? result : new ArrayList<>();
    }
}
```

---

## Core Problems

### Problem 1: Course Schedule

**LeetCode:** [207. Course Schedule](https://leetcode.com/problems/course-schedule/)

**Problem:** Check if can finish all courses given prerequisites.

**Solution:**
```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> graph = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    
    for (int i = 0; i < numCourses; i++) {
        graph.add(new ArrayList<>());
    }
    
    for (int[] edge : prerequisites) {
        graph.get(edge[1]).add(edge[0]);
        inDegree[edge[0]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) {
            queue.offer(i);
        }
    }
    
    int count = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        count++;
        
        for (int neighbor : graph.get(node)) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] == 0) {
                queue.offer(neighbor);
            }
        }
    }
    
    return count == numCourses;
}
```

**Time Complexity:** O(V + E)  
**Space Complexity:** O(V + E)

---

### Problem 2: Course Schedule II

**LeetCode:** [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

**Problem:** Return course order if possible.

**Solution:**
```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> graph = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    
    for (int i = 0; i < numCourses; i++) {
        graph.add(new ArrayList<>());
    }
    
    for (int[] edge : prerequisites) {
        graph.get(edge[1]).add(edge[0]);
        inDegree[edge[0]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) {
            queue.offer(i);
        }
    }
    
    int[] result = new int[numCourses];
    int index = 0;
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        result[index++] = node;
        
        for (int neighbor : graph.get(node)) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] == 0) {
                queue.offer(neighbor);
            }
        }
    }
    
    return index == numCourses ? result : new int[0];
}
```

---

## Key Takeaways

✅ Use Kahn's algorithm (BFS) or DFS  
✅ Start with nodes having in-degree 0  
✅ Detect cycles: if count != total nodes  
✅ Time: O(V + E)

---

**Next Pattern:** [Backtracking Pattern](./13_BACKTRACKING_PATTERN.md)

