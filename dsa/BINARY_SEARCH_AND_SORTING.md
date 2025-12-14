# DSA Quick Cheat Sheet - Binary Search & Sorting 🔍📊

> Daily revision guide for binary search and sorting algorithms

---

## Table of Contents
1. [Binary Search](#binary-search)
2. [Sorting Algorithms](#sorting-algorithms)
3. [Frequently Asked Questions](#frequently-asked-questions)
4. [Quick Reference](#quick-reference)

---

## Binary Search

### What is Binary Search?

**Binary Search** is a search algorithm that finds the position of a target value in a **sorted array** by repeatedly dividing the search interval in half.

**Key Requirement:** Array must be **sorted**

**Time Complexity:** O(log n)  
**Space Complexity:** O(1) iterative, O(log n) recursive

### How Binary Search Works

**Principle:** Compare target with middle element, eliminate half of the array

```
Array: [1, 3, 5, 7, 9, 11, 13, 15]
Target: 7

Step 1: left=0, right=7, mid=3
        [1, 3, 5, 7, 9, 11, 13, 15]
         ↑        ↑              ↑
        left     mid           right
        arr[3] = 7 == target ✅ Found!
```

### Basic Binary Search Implementation

**Code:**
```java
int binarySearch(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;  // Avoid overflow
        
        if (arr[mid] == target) {
            return mid;  // Found!
        } else if (arr[mid] < target) {
            left = mid + 1;  // Search right half
        } else {
            right = mid - 1;  // Search left half
        }
    }
    
    return -1;  // Not found
}
```

**Visual Example:**
```
Search for 7 in [1, 3, 5, 7, 9, 11, 13, 15]

Iteration 1:
left=0, right=7, mid=3
[1, 3, 5, 7, 9, 11, 13, 15]
 ↑        ↑              ↑
arr[3] = 7 == 7 ✅ Found at index 3!
```

### Why `mid = left + (right - left) / 2`?

**Avoid Integer Overflow:**
```java
// ❌ BAD: Can overflow
int mid = (left + right) / 2;
// If left = 2,000,000,000 and right = 2,000,000,000
// left + right = 4,000,000,000 (overflow!)

// ✅ GOOD: No overflow
int mid = left + (right - left) / 2;
// left + (right - left) / 2 = left + 0 = left (safe)
```

---

## Binary Search Variations

### 1. Find First Occurrence

**Problem:** Find the first index of target in sorted array with duplicates

```
Array: [1, 2, 2, 2, 3, 4, 5]
Target: 2
Result: Index 1 (first occurrence)
```

**Code:**
```java
int findFirst(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;  // Found, but continue searching left
            right = mid - 1;  // Search left for earlier occurrence
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}
```

**Visual:**
```
Array: [1, 2, 2, 2, 3, 4, 5], target = 2

Iteration 1: mid=3, arr[3]=2 == target
             result=3, search left (right=2)

Iteration 2: mid=1, arr[1]=2 == target
             result=1, search left (right=0)

Iteration 3: mid=0, arr[0]=1 < target
             left=1

Iteration 4: left=1, right=0 → break
             Return result=1 ✅
```

### 2. Find Last Occurrence

**Problem:** Find the last index of target

**Code:**
```java
int findLast(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;  // Found, but continue searching right
            left = mid + 1;  // Search right for later occurrence
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}
```

### 3. Find Insertion Position

**Problem:** Find index where target should be inserted to maintain sorted order

```
Array: [1, 3, 5, 7, 9]
Target: 6
Result: Index 3 (insert between 5 and 7)
```

**Code:**
```java
int searchInsert(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return left;  // Insertion position
}
```

### 4. Search in Rotated Sorted Array

**Problem:** Array is rotated, find target

```
Array: [4, 5, 6, 7, 0, 1, 2]
Target: 0
Result: Index 4
```

**Code:**
```java
int searchRotated(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            return mid;
        }
        
        // Left half is sorted
        if (arr[left] <= arr[mid]) {
            if (target >= arr[left] && target < arr[mid]) {
                right = mid - 1;  // Target in left half
            } else {
                left = mid + 1;  // Target in right half
            }
        } 
        // Right half is sorted
        else {
            if (target > arr[mid] && target <= arr[right]) {
                left = mid + 1;  // Target in right half
            } else {
                right = mid - 1;  // Target in left half
            }
        }
    }
    
    return -1;
}
```

**Visual:**
```
Array: [4, 5, 6, 7, 0, 1, 2], target = 0

Iteration 1: mid=3, arr[3]=7
             arr[0]=4 <= arr[3]=7 (left sorted)
             target=0 not in [4, 7) → search right
             left=4

Iteration 2: mid=5, arr[5]=1
             arr[4]=0 <= arr[5]=1 (left sorted)
             target=0 in [0, 1) → search left
             right=4

Iteration 3: mid=4, arr[4]=0 == target ✅
```

### 5. Find Peak Element

**Problem:** Find any peak element (greater than neighbors)

```
Array: [1, 2, 3, 1]
Peak: Index 2 (value 3)
```

**Code:**
```java
int findPeak(int[] arr) {
    int left = 0;
    int right = arr.length - 1;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        // If mid < mid+1, peak is on right
        if (arr[mid] < arr[mid + 1]) {
            left = mid + 1;
        } 
        // If mid > mid+1, peak is on left (or mid is peak)
        else {
            right = mid;
        }
    }
    
    return left;  // Peak index
}
```

---

## Sorting Algorithms

### Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable | Notes |
|-----------|------|---------|-------|-------|--------|-------|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes | Simple, inefficient |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | No | Find min, swap |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes | Good for small arrays |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Divide & conquer |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | No | Fast average case |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Uses heap |
| **Counting Sort** | O(n + k) | O(n + k) | O(n + k) | O(k) | Yes | For small range |
| **Radix Sort** | O(d(n + k)) | O(d(n + k)) | O(d(n + k)) | O(n + k) | Yes | For integers |

---

## 1. Bubble Sort

### How It Works

**Principle:** Compare adjacent elements, swap if wrong order, repeat

**Visual:**
```
Array: [64, 34, 25, 12, 22, 11, 90]

Pass 1:
[64, 34, 25, 12, 22, 11, 90]
 ↑   ↑
34 < 64? Yes → Swap
[34, 64, 25, 12, 22, 11, 90]
     ↑   ↑
25 < 64? Yes → Swap
[34, 25, 64, 12, 22, 11, 90]
        ↑   ↑
12 < 64? Yes → Swap
... continue

After Pass 1: [34, 25, 12, 22, 11, 64, 90]
                              ↑   ↑
                              Largest at end

Pass 2: [25, 12, 22, 11, 34, 64, 90]
                    ↑   ↑
                    Second largest

... Continue until sorted
```

**Code:**
```java
void bubbleSort(int[] arr) {
    int n = arr.length;
    
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                // Swap
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        
        // Optimization: If no swap, array is sorted
        if (!swapped) break;
    }
}
```

**Time Complexity:**
- Best: O(n) - already sorted
- Average: O(n²)
- Worst: O(n²) - reverse sorted

---

## 2. Selection Sort

### How It Works

**Principle:** Find minimum element, swap with first position, repeat

**Visual:**
```
Array: [64, 25, 12, 22, 11]

Pass 1: Find min in [64, 25, 12, 22, 11]
        Min = 11 at index 4
        Swap with arr[0]
        [11, 25, 12, 22, 64]

Pass 2: Find min in [25, 12, 22, 64]
        Min = 12 at index 2
        Swap with arr[1]
        [11, 12, 25, 22, 64]

Pass 3: Find min in [25, 22, 64]
        Min = 22 at index 3
        Swap with arr[2]
        [11, 12, 22, 25, 64]

Pass 4: Find min in [25, 64]
        Min = 25 at index 3
        Already in place
        [11, 12, 22, 25, 64] ✅
```

**Code:**
```java
void selectionSort(int[] arr) {
    int n = arr.length;
    
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        
        // Find minimum element
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }
        
        // Swap
        int temp = arr[minIdx];
        arr[minIdx] = arr[i];
        arr[i] = temp;
    }
}
```

**Time Complexity:** O(n²) always

---

## 3. Insertion Sort

### How It Works

**Principle:** Build sorted array one element at a time, insert each element in correct position

**Visual:**
```
Array: [64, 25, 12, 22, 11]

Pass 1: [64] | [25, 12, 22, 11]
         Insert 25 before 64
         [25, 64] | [12, 22, 11]

Pass 2: [25, 64] | [12, 22, 11]
         Insert 12 before 25
         [12, 25, 64] | [22, 11]

Pass 3: [12, 25, 64] | [22, 11]
         Insert 22 between 12 and 25
         [12, 22, 25, 64] | [11]

Pass 4: [12, 22, 25, 64] | [11]
         Insert 11 at beginning
         [11, 12, 22, 25, 64] ✅
```

**Code:**
```java
void insertionSort(int[] arr) {
    int n = arr.length;
    
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        
        // Shift elements greater than key to right
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        
        arr[j + 1] = key;
    }
}
```

**Time Complexity:**
- Best: O(n) - already sorted
- Average: O(n²)
- Worst: O(n²) - reverse sorted

**Why Good for Small Arrays?**
- Simple implementation
- Low overhead
- Efficient for nearly sorted data

---

## 4. Merge Sort

### How It Works

**Principle:** Divide array into halves, sort each half, merge sorted halves

**Visual:**
```
Array: [38, 27, 43, 3, 9, 82, 10]

Divide:
                    [38, 27, 43, 3, 9, 82, 10]
                   /                           \
        [38, 27, 43, 3]                [9, 82, 10]
        /           \                   /         \
   [38, 27]      [43, 3]            [9, 82]    [10]
   /      \      /     \            /     \      |
 [38]   [27]   [43]   [3]         [9]   [82]  [10]

Merge:
   [38]   [27]   [43]   [3]         [9]   [82]  [10]
    \      /      \     /            \     /     |
   [27, 38]      [3, 43]            [9, 82]  [10]
        \           /                    \      /
        [3, 27, 38, 43]              [9, 10, 82]
                   \                    /
              [3, 9, 10, 27, 38, 43, 82] ✅
```

**Code:**
```java
void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        
        // Sort left half
        mergeSort(arr, left, mid);
        
        // Sort right half
        mergeSort(arr, mid + 1, right);
        
        // Merge sorted halves
        merge(arr, left, mid, right);
    }
}

void merge(int[] arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    
    // Create temp arrays
    int[] leftArr = new int[n1];
    int[] rightArr = new int[n2];
    
    // Copy data
    for (int i = 0; i < n1; i++) {
        leftArr[i] = arr[left + i];
    }
    for (int j = 0; j < n2; j++) {
        rightArr[j] = arr[mid + 1 + j];
    }
    
    // Merge temp arrays
    int i = 0, j = 0, k = left;
    
    while (i < n1 && j < n2) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k++] = leftArr[i++];
        } else {
            arr[k++] = rightArr[j++];
        }
    }
    
    // Copy remaining elements
    while (i < n1) arr[k++] = leftArr[i++];
    while (j < n2) arr[k++] = rightArr[j++];
}
```

**Time Complexity:** O(n log n) always  
**Space Complexity:** O(n)

**Why Stable?** Preserves relative order of equal elements

---

## 5. Quick Sort

### How It Works

**Principle:** Choose pivot, partition array around pivot, recursively sort partitions

**Visual:**
```
Array: [38, 27, 43, 3, 9, 82, 10]

Step 1: Choose pivot = 38 (last element)
        Partition: [27, 3, 9, 10] | 38 | [43, 82]
                   (smaller)      (pivot) (larger)

Step 2: Sort [27, 3, 9, 10]
        Pivot = 10
        Partition: [3, 9] | 10 | [27]
        Sort [3, 9]: Already sorted
        Result: [3, 9, 10, 27]

Step 3: Sort [43, 82]
        Already sorted

Final: [3, 9, 10, 27, 38, 43, 82] ✅
```

**Code:**
```java
void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        // Partition and get pivot index
        int pi = partition(arr, low, high);
        
        // Sort elements before and after partition
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

int partition(int[] arr, int low, int high) {
    int pivot = arr[high];  // Choose last element as pivot
    int i = low - 1;  // Index of smaller element
    
    for (int j = low; j < high; j++) {
        // If current element <= pivot
        if (arr[j] <= pivot) {
            i++;
            // Swap arr[i] and arr[j]
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    
    // Swap pivot with element at i+1
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    
    return i + 1;  // Return pivot index
}
```

**Time Complexity:**
- Best: O(n log n) - balanced partitions
- Average: O(n log n)
- Worst: O(n²) - unbalanced partitions (already sorted)

**Pivot Selection Strategies:**
1. **Last element** (simple, but worst case for sorted array)
2. **First element** (same issue)
3. **Middle element** (better)
4. **Random element** (good average case)
5. **Median of three** (best for worst case)

**Optimized Quick Sort:**
```java
int partition(int[] arr, int low, int high) {
    // Use median of three
    int mid = low + (high - low) / 2;
    if (arr[mid] < arr[low]) swap(arr, low, mid);
    if (arr[high] < arr[low]) swap(arr, low, high);
    if (arr[high] < arr[mid]) swap(arr, mid, high);
    
    int pivot = arr[mid];
    // ... rest of partition logic
}
```

---

## 6. Heap Sort

### How It Works

**Principle:** Build max heap, repeatedly extract maximum element

**Visual:**
```
Array: [4, 10, 3, 5, 1]

Step 1: Build Max Heap
        10
       /  \
      5    3
     / \
    4   1

Step 2: Extract max (10), swap with last
        [1, 5, 3, 4, 10]
        Heapify: [5, 4, 3, 1, 10]

Step 3: Extract max (5)
        [1, 4, 3, 5, 10]
        Heapify: [4, 1, 3, 5, 10]

Step 4: Extract max (4)
        [3, 1, 4, 5, 10]
        Heapify: [3, 1, 4, 5, 10]

Step 5: Extract max (3)
        [1, 3, 4, 5, 10] ✅
```

**Code:**
```java
void heapSort(int[] arr) {
    int n = arr.length;
    
    // Build max heap
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }
    
    // Extract elements from heap
    for (int i = n - 1; i > 0; i--) {
        // Move root to end
        int temp = arr[0];
        arr[0] = arr[i];
        arr[i] = temp;
        
        // Heapify reduced heap
        heapify(arr, i, 0);
    }
}

void heapify(int[] arr, int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    
    if (left < n && arr[left] > arr[largest]) {
        largest = left;
    }
    
    if (right < n && arr[right] > arr[largest]) {
        largest = right;
    }
    
    if (largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        
        heapify(arr, n, largest);
    }
}
```

**Time Complexity:** O(n log n) always  
**Space Complexity:** O(1)

---

## 7. Counting Sort

### How It Works

**Principle:** Count frequency of each element, place elements in sorted order

**Visual:**
```
Array: [4, 2, 2, 8, 3, 3, 1]
Range: 1 to 8

Step 1: Count frequencies
Index:  0  1  2  3  4  5  6  7  8
Count: [0, 1, 2, 2, 1, 0, 0, 0, 1]

Step 2: Cumulative counts
Index:  0  1  2  3  4  5  6  7  8
Count: [0, 1, 3, 5, 6, 6, 6, 6, 7]

Step 3: Place elements
1 at position 0 → [1, _, _, _, _, _, _]
2 at position 1-2 → [1, 2, 2, _, _, _, _]
3 at position 3-4 → [1, 2, 2, 3, 3, _, _]
4 at position 5 → [1, 2, 2, 3, 3, 4, _]
8 at position 6 → [1, 2, 2, 3, 3, 4, 8] ✅
```

**Code:**
```java
void countingSort(int[] arr) {
    int n = arr.length;
    int max = Arrays.stream(arr).max().getAsInt();
    int min = Arrays.stream(arr).min().getAsInt();
    int range = max - min + 1;
    
    // Count array
    int[] count = new int[range];
    int[] output = new int[n];
    
    // Count frequencies
    for (int i = 0; i < n; i++) {
        count[arr[i] - min]++;
    }
    
    // Cumulative counts
    for (int i = 1; i < range; i++) {
        count[i] += count[i - 1];
    }
    
    // Build output array
    for (int i = n - 1; i >= 0; i--) {
        output[count[arr[i] - min] - 1] = arr[i];
        count[arr[i] - min]--;
    }
    
    // Copy to original array
    System.arraycopy(output, 0, arr, 0, n);
}
```

**Time Complexity:** O(n + k) where k is range  
**Space Complexity:** O(k)

**When to Use:** Small range of integers (0-100, 0-1000)

---

## Frequently Asked Questions

### Binary Search Questions

#### Q1: Why is binary search O(log n)?

**Answer:**
"Binary search eliminates half of the array in each iteration. If we start with n elements:
- After 1 iteration: n/2 elements
- After 2 iterations: n/4 elements
- After k iterations: n/2^k elements

When n/2^k = 1, we have k = log₂(n) iterations.

Example: For array of 1024 elements:
- Iteration 1: 512 elements
- Iteration 2: 256 elements
- Iteration 3: 128 elements
- ...
- Iteration 10: 1 element

Total: log₂(1024) = 10 iterations"

#### Q2: When should you use binary search?

**Answer:**
"I use binary search when:
1. **Array is sorted** (required condition)
2. **Need O(log n) search** (faster than linear O(n))
3. **Searching multiple times** (amortize sorting cost)
4. **Large dataset** (binary search scales better)

I don't use it for:
- Unsorted arrays (sort first or use linear search)
- Small arrays (overhead not worth it)
- One-time search (sorting cost > search benefit)"

#### Q3: What's the difference between `left <= right` and `left < right`?

**Answer:**
"`left <= right`: Standard binary search, checks all elements
- Returns exact index if found
- Returns -1 if not found
- Example: `while (left <= right)`

`left < right`: Used for boundary problems
- Finds insertion point
- Used in 'find first/last occurrence'
- Example: `while (left < right)`

**Key Difference:**
- `<=` processes all elements including when left == right
- `<` stops when left == right, leaving one element unchecked"

#### Q4: How do you handle duplicates in binary search?

**Answer:**
"For duplicates, I modify binary search to find first or last occurrence:

**Find First:**
- When found, continue searching left
- `right = mid - 1` (don't stop)

**Find Last:**
- When found, continue searching right
- `left = mid + 1` (don't stop)

**Code Pattern:**
```java
if (arr[mid] == target) {
    result = mid;  // Remember position
    // Continue searching in one direction
    right = mid - 1;  // For first
    // OR
    left = mid + 1;   // For last
}
```"

#### Q5: How do you search in a rotated sorted array?

**Answer:**
"I identify which half is sorted, then decide where to search:

1. **Check if left half is sorted**: `arr[left] <= arr[mid]`
   - If target in [left, mid) → search left
   - Otherwise → search right

2. **Otherwise, right half is sorted**
   - If target in (mid, right] → search right
   - Otherwise → search left

**Key Insight:** At least one half is always sorted in rotated array"

### Sorting Questions

#### Q1: Which sorting algorithm is best?

**Answer:**
"It depends on the use case:

**General Purpose:**
- **Quick Sort**: Fast average case O(n log n), in-place
- **Merge Sort**: Guaranteed O(n log n), stable, good for linked lists

**Small Arrays (< 50 elements):**
- **Insertion Sort**: Simple, efficient, low overhead

**Nearly Sorted:**
- **Insertion Sort**: O(n) best case

**Large Range of Integers:**
- **Counting Sort**: O(n + k) where k is range
- **Radix Sort**: O(d(n + k)) for integers

**External Sorting (disk):**
- **Merge Sort**: Good sequential access pattern

**In Practice:** Most languages use hybrid algorithms (Timsort in Python, Introsort in C++)"

#### Q2: What is a stable sort?

**Answer:**
"Stable sort preserves the relative order of equal elements.

**Example:**
```
Array: [(3, 'a'), (2, 'b'), (3, 'c'), (1, 'd')]

Sort by first element:

Stable Sort (Merge Sort):
[(1, 'd'), (2, 'b'), (3, 'a'), (3, 'c')]
                    ↑        ↑
                    Original order preserved

Unstable Sort (Quick Sort):
[(1, 'd'), (2, 'b'), (3, 'c'), (3, 'a')]
                    ↑        ↑
                    Order may change
```

**Stable Algorithms:** Merge Sort, Insertion Sort, Bubble Sort, Counting Sort, Radix Sort

**Unstable Algorithms:** Quick Sort, Heap Sort, Selection Sort"

#### Q3: Why is Quick Sort preferred over Merge Sort for arrays?

**Answer:**
"Quick Sort is often preferred because:

1. **In-place**: O(log n) space vs O(n) for Merge Sort
2. **Cache friendly**: Better locality of reference
3. **Faster average case**: Constants are smaller
4. **Tail recursion**: Can be optimized

**However, Merge Sort is better when:**
- Stability is required
- Worst-case O(n log n) is needed
- Sorting linked lists (no random access needed)
- External sorting (sequential access)"

#### Q4: What is the worst case for Quick Sort?

**Answer:**
"Worst case is O(n²) when:
1. **Already sorted array**: Pivot always at end
2. **Reverse sorted array**: Pivot always at beginning
3. **All elements equal**: Poor pivot selection

**Example:**
```
Array: [1, 2, 3, 4, 5]
Pivot = 5 (last element)
Partition: [1, 2, 3, 4] | 5
Recurse on [1, 2, 3, 4]
Pivot = 4
Partition: [1, 2, 3] | 4
... O(n²) comparisons
```

**Solutions:**
1. **Random pivot**: Choose random element
2. **Median of three**: Choose median of first, middle, last
3. **Introsort**: Switch to Heap Sort if depth > 2 log n"

#### Q5: When would you use Counting Sort?

**Answer:**
"I use Counting Sort when:
1. **Small range**: Elements in range [0, k] where k is small
2. **Integer sorting**: Works only with integers
3. **Stability needed**: Preserves order of equal elements
4. **Linear time**: O(n + k) is better than O(n log n) when k < n log n

**Example Use Cases:**
- Sorting ages (0-150)
- Sorting grades (0-100)
- Sorting test scores
- Histogram generation

**Not suitable for:**
- Large range (k >> n)
- Floating point numbers
- Strings (use Radix Sort instead)"

#### Q6: What's the difference between Merge Sort and Quick Sort?

**Answer:**
"Key differences:

| Aspect | Merge Sort | Quick Sort |
|--------|------------|------------|
| **Time (Worst)** | O(n log n) | O(n²) |
| **Time (Average)** | O(n log n) | O(n log n) |
| **Space** | O(n) | O(log n) |
| **Stable** | Yes | No |
| **Divide** | Always in middle | Pivot-based |
| **Combine** | Merge step | No combine |
| **Best For** | Linked lists, stability | Arrays, average case |

**Merge Sort:** Predictable, stable, good worst case
**Quick Sort:** Fast average, in-place, cache-friendly"

#### Q7: How does Heap Sort work?

**Answer:**
"Heap Sort works in two phases:

**Phase 1: Build Max Heap**
- Convert array to max heap
- Parent >= children at every level
- Time: O(n)

**Phase 2: Extract Maximum**
- Swap root (max) with last element
- Reduce heap size
- Heapify to restore max heap property
- Repeat until sorted
- Time: O(n log n)

**Total:** O(n log n)

**Advantages:**
- Guaranteed O(n log n)
- In-place (O(1) space)
- No worst-case scenario

**Disadvantages:**
- Not stable
- Slower than Quick Sort in practice
- Poor cache performance"

#### Q8: Can you sort in O(n) time?

**Answer:**
"Yes, but only in special cases:

**Counting Sort:** O(n + k) where k is range
- Works when k is small (e.g., 0-100)
- Only for integers

**Radix Sort:** O(d(n + k)) where d is digits
- For integers with fixed number of digits
- Uses Counting Sort as subroutine

**Bucket Sort:** O(n) average case
- When data is uniformly distributed
- Divides into buckets, sorts each

**General Comparison Sort:** No
- Lower bound: Ω(n log n)
- Cannot beat this for arbitrary data

**In Practice:** Most real-world sorts are O(n log n) or use special cases for O(n)"

---

## Quick Reference

### Binary Search Template

```java
int binarySearch(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
    return -1;
}
```

### Sorting Algorithm Selection

```
Is array small (< 50)? → Insertion Sort
Is array nearly sorted? → Insertion Sort
Need stability? → Merge Sort
Need guaranteed O(n log n)? → Merge Sort or Heap Sort
General purpose? → Quick Sort
Integer with small range? → Counting Sort
Integer with many digits? → Radix Sort
```

### Time Complexity Cheat Sheet

| Operation | Linear Search | Binary Search |
|-----------|---------------|---------------|
| **Sorted Array** | O(n) | O(log n) |
| **Unsorted Array** | O(n) | O(n log n) to sort first |

| Algorithm | Best | Average | Worst |
|-----------|------|---------|-------|
| **Bubble** | O(n) | O(n²) | O(n²) |
| **Selection** | O(n²) | O(n²) | O(n²) |
| **Insertion** | O(n) | O(n²) | O(n²) |
| **Merge** | O(n log n) | O(n log n) | O(n log n) |
| **Quick** | O(n log n) | O(n log n) | O(n²) |
| **Heap** | O(n log n) | O(n log n) | O(n log n) |
| **Counting** | O(n + k) | O(n + k) | O(n + k) |

---

## Memory Tips 🧠

### Binary Search
- **B**inary = **B**isect = **B**reak in half
- **L**og n = **L**ookups needed
- Always check: Is array sorted?

### Sorting
- **B**ubble = **B**iggest **B**ubbles up
- **S**election = **S**elect **S**mallest
- **I**nsertion = **I**nsert in order
- **M**erge = **M**erge sorted halves
- **Q**uick = **Q**uick average case

---

## Daily Revision Checklist ✅

- [ ] Binary Search: Basic, first/last occurrence, rotated array
- [ ] Bubble Sort: Adjacent comparison, O(n²)
- [ ] Selection Sort: Find min, swap, O(n²)
- [ ] Insertion Sort: Build sorted array, O(n) best case
- [ ] Merge Sort: Divide & conquer, O(n log n), stable
- [ ] Quick Sort: Pivot partition, O(n log n) average
- [ ] Heap Sort: Build heap, extract max, O(n log n)
- [ ] Counting Sort: Count frequencies, O(n + k)
- [ ] When to use which algorithm
- [ ] Time/Space complexity of each

---

**Happy Coding! 🔍📊**

