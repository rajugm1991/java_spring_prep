# Pattern 10: Top K Elements 🔝

## Overview

**Top K Elements** pattern finds K largest/smallest/frequent elements using heaps (PriorityQueue in Java).

## When to Use

✅ **Use Top K Elements when:**
- Finding K largest/smallest
- Finding K most frequent
- Finding K closest points
- Heap-based problems

## Pattern Template

```java
public class TopKElements {
    
    // Find K largest elements
    public int[] findKLargest(int[] nums, int k) {
        // Min heap of size k
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        
        for (int num : nums) {
            minHeap.offer(num);
            if (minHeap.size() > k) {
                minHeap.poll(); // Remove smallest
            }
        }
        
        int[] result = new int[k];
        for (int i = k - 1; i >= 0; i--) {
            result[i] = minHeap.poll();
        }
        
        return result;
    }
    
    // Find K most frequent elements
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> frequency = new HashMap<>();
        for (int num : nums) {
            frequency.put(num, frequency.getOrDefault(num, 0) + 1);
        }
        
        // Min heap by frequency
        PriorityQueue<Map.Entry<Integer, Integer>> heap = 
            new PriorityQueue<>((a, b) -> a.getValue() - b.getValue());
        
        for (Map.Entry<Integer, Integer> entry : frequency.entrySet()) {
            heap.offer(entry);
            if (heap.size() > k) {
                heap.poll();
            }
        }
        
        int[] result = new int[k];
        for (int i = k - 1; i >= 0; i--) {
            result[i] = heap.poll().getKey();
        }
        
        return result;
    }
}
```

---

## Core Problems

### Problem 1: Kth Largest Element

**LeetCode:** [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

**Solution:**
```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll();
        }
    }
    
    return minHeap.peek();
}
```

**Time Complexity:** O(n log k)  
**Space Complexity:** O(k)

---

### Problem 2: Top K Frequent Elements

**LeetCode:** [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

**Solution:**
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> frequency = new HashMap<>();
    for (int num : nums) {
        frequency.put(num, frequency.getOrDefault(num, 0) + 1);
    }
    
    PriorityQueue<Map.Entry<Integer, Integer>> heap = 
        new PriorityQueue<>((a, b) -> a.getValue() - b.getValue());
    
    for (Map.Entry<Integer, Integer> entry : frequency.entrySet()) {
        heap.offer(entry);
        if (heap.size() > k) {
            heap.poll();
        }
    }
    
    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) {
        result[i] = heap.poll().getKey();
    }
    
    return result;
}
```

---

### Problem 3: K Closest Points to Origin

**LeetCode:** [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

**Solution:**
```java
public int[][] kClosest(int[][] points, int k) {
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(
        (a, b) -> Integer.compare(
            b[0]*b[0] + b[1]*b[1],
            a[0]*a[0] + a[1]*a[1]
        )
    );
    
    for (int[] point : points) {
        maxHeap.offer(point);
        if (maxHeap.size() > k) {
            maxHeap.poll();
        }
    }
    
    int[][] result = new int[k][2];
    for (int i = 0; i < k; i++) {
        result[i] = maxHeap.poll();
    }
    
    return result;
}
```

---

## Key Takeaways

✅ Use min heap for K largest, max heap for K smallest  
✅ Time: O(n log k)  
✅ Space: O(k)  
✅ Heap size should be K

---

**Next Pattern:** [K-way Merge Pattern](./11_KWAY_MERGE_PATTERN.md)

