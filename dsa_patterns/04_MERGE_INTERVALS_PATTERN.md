# Pattern 4: Merge Intervals 📅

## Overview

The **Merge Intervals** pattern is used to merge overlapping intervals or find gaps between intervals. It's essential for scheduling, calendar, and time-based problems.

## When to Use

✅ **Use Merge Intervals when:**
- Merging overlapping intervals
- Finding gaps between intervals
- Scheduling problems
- Calendar problems
- Time-based conflicts

## Pattern Template

```java
public class MergeIntervals {
    
    public int[][] merge(int[][] intervals) {
        if (intervals.length <= 1) return intervals;
        
        // Sort by start time
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        
        List<int[]> merged = new ArrayList<>();
        int[] current = intervals[0];
        
        for (int i = 1; i < intervals.length; i++) {
            int[] next = intervals[i];
            
            if (current[1] >= next[0]) {
                // Overlapping - merge
                current[1] = Math.max(current[1], next[1]);
            } else {
                // No overlap - add current and move to next
                merged.add(current);
                current = next;
            }
        }
        
        merged.add(current);
        return merged.toArray(new int[merged.size()][]);
    }
}
```

---

## Core Problems

### Problem 1: Merge Intervals

**LeetCode:** [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)

**Problem:** Merge all overlapping intervals.

**Example:**
```
Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

**Solution:**
```java
public int[][] merge(int[][] intervals) {
    if (intervals.length <= 1) return intervals;
    
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    
    List<int[]> merged = new ArrayList<>();
    int[] current = intervals[0];
    
    for (int i = 1; i < intervals.length; i++) {
        if (current[1] >= intervals[i][0]) {
            current[1] = Math.max(current[1], intervals[i][1]);
        } else {
            merged.add(current);
            current = intervals[i];
        }
    }
    
    merged.add(current);
    return merged.toArray(new int[merged.size()][]);
}
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(n)

---

### Problem 2: Insert Interval

**LeetCode:** [57. Insert Interval](https://leetcode.com/problems/insert-interval/)

**Problem:** Insert new interval into sorted non-overlapping intervals.

**Solution:**
```java
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0;
    int n = intervals.length;
    
    // Add all intervals before newInterval
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i]);
        i++;
    }
    
    // Merge overlapping intervals
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);
    
    // Add remaining intervals
    while (i < n) {
        result.add(intervals[i]);
        i++;
    }
    
    return result.toArray(new int[result.size()][]);
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### Problem 3: Non-overlapping Intervals

**LeetCode:** [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)

**Problem:** Find minimum intervals to remove to make rest non-overlapping.

**Solution:**
```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length == 0) return 0;
    
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));
    
    int count = 0;
    int end = intervals[0][1];
    
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < end) {
            count++; // Remove this interval
        } else {
            end = intervals[i][1];
        }
    }
    
    return count;
}
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(1)

**Key Insight:** Sort by end time, keep intervals with earliest end times

---

### Problem 4: Meeting Rooms

**LeetCode:** [252. Meeting Rooms](https://leetcode.com/problems/meeting-rooms/)

**Problem:** Check if person can attend all meetings.

**Solution:**
```java
public boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i-1][1]) {
            return false;
        }
    }
    
    return true;
}
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(1)

---

### Problem 5: Meeting Rooms II

**LeetCode:** [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)

**Problem:** Find minimum rooms needed for all meetings.

**Solution:**
```java
public int minMeetingRooms(int[][] intervals) {
    int n = intervals.length;
    int[] starts = new int[n];
    int[] ends = new int[n];
    
    for (int i = 0; i < n; i++) {
        starts[i] = intervals[i][0];
        ends[i] = intervals[i][1];
    }
    
    Arrays.sort(starts);
    Arrays.sort(ends);
    
    int rooms = 0;
    int endIndex = 0;
    
    for (int start : starts) {
        if (start < ends[endIndex]) {
            rooms++; // Need new room
        } else {
            endIndex++; // Reuse room
        }
    }
    
    return rooms;
}
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(n)

---

## Advanced Problems

### Problem 6: Employee Free Time

**LeetCode:** [759. Employee Free Time](https://leetcode.com/problems/employee-free-time/)

**Problem:** Find common free time for all employees.

**Solution:**
```java
public List<Interval> employeeFreeTime(List<List<Interval>> schedule) {
    List<Interval> allIntervals = new ArrayList<>();
    for (List<Interval> employee : schedule) {
        allIntervals.addAll(employee);
    }
    
    Collections.sort(allIntervals, (a, b) -> a.start - b.start);
    
    List<Interval> merged = new ArrayList<>();
    Interval current = allIntervals.get(0);
    
    for (int i = 1; i < allIntervals.size(); i++) {
        Interval next = allIntervals.get(i);
        if (current.end >= next.start) {
            current.end = Math.max(current.end, next.end);
        } else {
            merged.add(current);
            current = next;
        }
    }
    merged.add(current);
    
    List<Interval> freeTime = new ArrayList<>();
    for (int i = 1; i < merged.size(); i++) {
        freeTime.add(new Interval(merged.get(i-1).end, merged.get(i).start));
    }
    
    return freeTime;
}
```

---

## Key Takeaways

✅ Always sort intervals first  
✅ Sort by start time for merging  
✅ Sort by end time for scheduling  
✅ Check overlap: interval1.end >= interval2.start

---

**Next Pattern:** [Cyclic Sort Pattern](./05_CYCLIC_SORT_PATTERN.md)

