# DSA Quick Cheat Sheet - Recursion 🔄

> Complete guide to recursion with visual explanations and interview questions

---

## Table of Contents
1. [What is Recursion?](#what-is-recursion)
2. [Recursion Components](#recursion-components)
3. [How Recursion Works](#how-recursion-works)
4. [Common Recursion Patterns](#common-recursion-patterns)
5. [Classic Recursion Problems](#classic-recursion-problems)
6. [Tree Problems with Recursion](#tree-problems-with-recursion)
7. [Backtracking](#backtracking)
8. [Dynamic Programming with Recursion](#dynamic-programming-with-recursion)
9. [Common Mistakes](#common-mistakes)
10. [Optimization Techniques](#optimization-techniques)
11. [Interview Questions](#interview-questions)

---

## What is Recursion?

**Recursion:** A function that calls itself

**Key Concept:** Solve a problem by solving smaller instances of the same problem

### Simple Example: Countdown

```java
void countdown(int n) {
    if (n <= 0) {  // Base case
        System.out.println("Blastoff!");
        return;
    }
    
    System.out.println(n);
    countdown(n - 1);  // Recursive call
}
```

**Visual:**
```
countdown(5)
  │
  ├─ Print: 5
  ├─ countdown(4)
  │    │
  │    ├─ Print: 4
  │    ├─ countdown(3)
  │    │    │
  │    │    ├─ Print: 3
  │    │    ├─ countdown(2)
  │    │    │    │
  │    │    │    ├─ Print: 2
  │    │    │    ├─ countdown(1)
  │    │    │    │    │
  │    │    │    │    ├─ Print: 1
  │    │    │    │    ├─ countdown(0)
  │    │    │    │    │    │
  │    │    │    │    │    └─ Print: "Blastoff!" ✅
```

**Output:** 5 → 4 → 3 → 2 → 1 → Blastoff!

---

## Recursion Components

### 1. Base Case

**Base Case:** Condition that stops recursion (prevents infinite loop)

**Rule:** Always have a base case!

```java
int factorial(int n) {
    // Base case: n = 0 or 1
    if (n <= 1) {
        return 1;  // Stop recursion
    }
    
    // Recursive case
    return n * factorial(n - 1);
}
```

### 2. Recursive Case

**Recursive Case:** Function calls itself with smaller input

**Rule:** Input must get closer to base case!

```java
int sum(int n) {
    // Base case
    if (n == 0) {
        return 0;
    }
    
    // Recursive case: n gets smaller (n-1)
    return n + sum(n - 1);
}
```

### 3. Recursive Call Stack

**Call Stack:** Memory structure that tracks function calls

**Visual:**
```
sum(5)
  │
  ├─ Call Stack: [sum(5)]
  ├─ sum(5) = 5 + sum(4)
  │    │
  │    ├─ Call Stack: [sum(5), sum(4)]
  │    ├─ sum(4) = 4 + sum(3)
  │    │    │
  │    │    ├─ Call Stack: [sum(5), sum(4), sum(3)]
  │    │    ├─ sum(3) = 3 + sum(2)
  │    │    │    │
  │    │    │    ├─ Call Stack: [sum(5), sum(4), sum(3), sum(2)]
  │    │    │    ├─ sum(2) = 2 + sum(1)
  │    │    │    │    │
  │    │    │    │    ├─ Call Stack: [sum(5), sum(4), sum(3), sum(2), sum(1)]
  │    │    │    │    ├─ sum(1) = 1 + sum(0)
  │    │    │    │    │    │
  │    │    │    │    │    ├─ Call Stack: [sum(5), sum(4), sum(3), sum(2), sum(1), sum(0)]
  │    │    │    │    │    └─ sum(0) = 0 ✅ (Base case)
  │    │    │    │    │
  │    │    │    │    └─ Return: 1 + 0 = 1
  │    │    │    │
  │    │    │    └─ Return: 2 + 1 = 3
  │    │    │
  │    │    └─ Return: 3 + 3 = 6
  │    │
  │    └─ Return: 4 + 6 = 10
  │
  └─ Return: 5 + 10 = 15 ✅
```

**Result:** 15

---

## How Recursion Works

### Step-by-Step: Factorial

**Problem:** Calculate 5! = 5 × 4 × 3 × 2 × 1

```java
int factorial(int n) {
    // Base case
    if (n <= 1) {
        return 1;
    }
    
    // Recursive case
    return n * factorial(n - 1);
}
```

**Visual Execution:**
```
factorial(5)
  │
  ├─ n = 5, not base case
  ├─ return 5 * factorial(4)
  │    │
  │    ├─ n = 4, not base case
  │    ├─ return 4 * factorial(3)
  │    │    │
  │    │    ├─ n = 3, not base case
  │    │    ├─ return 3 * factorial(2)
  │    │    │    │
  │    │    │    ├─ n = 2, not base case
  │    │    │    ├─ return 2 * factorial(1)
  │    │    │    │    │
  │    │    │    │    ├─ n = 1, base case ✅
  │    │    │    │    └─ return 1
  │    │    │    │
  │    │    │    └─ return 2 * 1 = 2
  │    │    │
  │    │    └─ return 3 * 2 = 6
  │    │
  │    └─ return 4 * 6 = 24
  │
  └─ return 5 * 24 = 120 ✅
```

**Result:** 120

---

## Common Recursion Patterns

### Pattern 1: Linear Recursion

**One recursive call per function**

```java
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1);  // Single recursive call
}
```

**Time:** O(n)  
**Space:** O(n) (call stack)

### Pattern 2: Binary Recursion

**Two recursive calls per function**

```java
int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);  // Two recursive calls
}
```

**Time:** O(2^n)  
**Space:** O(n) (call stack)

**Visual:**
```
fibonacci(5)
  │
  ├─ fib(4) + fib(3)
  │    │
  │    ├─ fib(3) + fib(2)
  │    │    │
  │    │    ├─ fib(2) + fib(1)
  │    │    │    │
  │    │    │    ├─ fib(1) + fib(0) = 1 + 0 = 1
  │    │    │    └─ fib(1) = 1
  │    │    │
  │    │    └─ fib(1) + fib(0) = 1 + 0 = 1
  │    │
  │    └─ fib(2) + fib(1) = 1 + 1 = 2
  │
  └─ fib(3) + fib(2) = 2 + 1 = 3
```

### Pattern 3: Tail Recursion

**Recursive call is the last operation**

```java
int sumTail(int n, int acc) {
    if (n == 0) return acc;
    return sumTail(n - 1, acc + n);  // Tail call
}
```

**Optimization:** Can be optimized to iteration (no stack growth)

### Pattern 4: Multiple Recursion

**More than two recursive calls**

```java
int tribonacci(int n) {
    if (n <= 1) return n;
    if (n == 2) return 1;
    return tribonacci(n - 1) + tribonacci(n - 2) + tribonacci(n - 3);
}
```

---

## Classic Recursion Problems

### 1. Factorial

**Problem:** Calculate n! = n × (n-1) × ... × 1

```java
int factorial(int n) {
    // Base case
    if (n <= 1) {
        return 1;
    }
    
    // Recursive case
    return n * factorial(n - 1);
}
```

**Visual:**
```
factorial(4) = 4 × 3 × 2 × 1 = 24

4 × factorial(3)
  3 × factorial(2)
    2 × factorial(1)
      1 ✅
```

**Time:** O(n)  
**Space:** O(n)

---

### 2. Fibonacci

**Problem:** Calculate nth Fibonacci number

**Sequence:** 0, 1, 1, 2, 3, 5, 8, 13, 21, ...

```java
int fibonacci(int n) {
    // Base cases
    if (n == 0) return 0;
    if (n == 1) return 1;
    
    // Recursive case
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

**Visual:**
```
fibonacci(5)
  │
  ├─ fib(4) + fib(3)
  │    │
  │    ├─ fib(3) + fib(2) = 2 + 1 = 3
  │    │    │
  │    │    ├─ fib(2) + fib(1) = 1 + 1 = 2
  │    │    └─ fib(1) + fib(0) = 1 + 0 = 1
  │    │
  │    └─ fib(2) + fib(1) = 1 + 1 = 2
  │
  └─ fib(3) + fib(2) = 2 + 1 = 3

Result: 3 + 2 = 5 ✅
```

**Time:** O(2^n) (inefficient!)  
**Space:** O(n)

**Optimized Version (Memoization):**
```java
Map<Integer, Integer> memo = new HashMap<>();

int fibonacciMemo(int n) {
    if (n <= 1) return n;
    
    if (memo.containsKey(n)) {
        return memo.get(n);
    }
    
    int result = fibonacciMemo(n - 1) + fibonacciMemo(n - 2);
    memo.put(n, result);
    return result;
}
```

**Time:** O(n)  
**Space:** O(n)

---

### 3. Power (Exponentiation)

**Problem:** Calculate x^n

**Naive Approach:**
```java
double power(double x, int n) {
    if (n == 0) return 1;
    if (n == 1) return x;
    return x * power(x, n - 1);
}
```

**Time:** O(n)

**Optimized Approach (Divide and Conquer):**
```java
double power(double x, int n) {
    if (n == 0) return 1;
    if (n == 1) return x;
    
    double half = power(x, n / 2);
    
    if (n % 2 == 0) {
        return half * half;  // Even: x^n = (x^(n/2))^2
    } else {
        return x * half * half;  // Odd: x^n = x × (x^(n/2))^2
    }
}
```

**Visual:**
```
power(2, 10)
  │
  ├─ n = 10 (even)
  ├─ half = power(2, 5)
  │    │
  │    ├─ n = 5 (odd)
  │    ├─ half = power(2, 2)
  │    │    │
  │    │    ├─ n = 2 (even)
  │    │    ├─ half = power(2, 1) = 2
  │    │    └─ return 2 × 2 = 4
  │    │
  │    └─ return 2 × 4 × 4 = 32
  │
  └─ return 32 × 32 = 1024 ✅
```

**Time:** O(log n)  
**Space:** O(log n)

---

### 4. Tower of Hanoi

**Problem:** Move n disks from source to destination using auxiliary rod

**Rules:**
1. Only move one disk at a time
2. Only move top disk
3. Larger disk cannot be on smaller disk

```java
void towerOfHanoi(int n, char source, char destination, char auxiliary) {
    // Base case: only one disk
    if (n == 1) {
        System.out.println("Move disk 1 from " + source + " to " + destination);
        return;
    }
    
    // Step 1: Move n-1 disks from source to auxiliary
    towerOfHanoi(n - 1, source, auxiliary, destination);
    
    // Step 2: Move largest disk from source to destination
    System.out.println("Move disk " + n + " from " + source + " to " + destination);
    
    // Step 3: Move n-1 disks from auxiliary to destination
    towerOfHanoi(n - 1, auxiliary, destination, source);
}
```

**Visual (n=3):**
```
Initial:        Step 1:         Step 2:         Step 3:
  A  B  C        A  B  C        A  B  C        A  B  C
  |  |  |        |  |  |        |  |  |        |  |  |
  |  |  |        |  |  |        |  |  |        |  |  |
  |  |  |        |  |  |        |  |  |        |  |  |
 [3] |  |       [3] |  |        |  | [3]       |  | [3]
 [2] |  |       [2] |  |        |  | [2]       |  | [2]
 [1] |  |        | [1]|        | [1]|        |  | [1]

Move 1: A → C
Move 2: A → B
Move 3: C → B
Move 4: A → C
Move 5: B → A
Move 6: B → C
Move 7: A → C
```

**Time:** O(2^n)  
**Space:** O(n)

---

### 5. String Reversal

**Problem:** Reverse a string using recursion

```java
String reverse(String str) {
    // Base case: empty or single character
    if (str.length() <= 1) {
        return str;
    }
    
    // Recursive case: last char + reverse of rest
    return str.charAt(str.length() - 1) + 
           reverse(str.substring(0, str.length() - 1));
}
```

**Visual:**
```
reverse("hello")
  │
  ├─ "o" + reverse("hell")
  │    │
  │    ├─ "l" + reverse("hel")
  │    │    │
  │    │    ├─ "l" + reverse("he")
  │    │    │    │
  │    │    │    ├─ "e" + reverse("h")
  │    │    │    │    │
  │    │    │    │    └─ "h" ✅
  │    │    │    │
  │    │    │    └─ "e" + "h" = "eh"
  │    │    │
  │    │    └─ "l" + "eh" = "leh"
  │    │
  │    └─ "l" + "leh" = "lleh"
  │
  └─ "o" + "lleh" = "olleh" ✅
```

**Time:** O(n)  
**Space:** O(n)

---

### 6. Palindrome Check

**Problem:** Check if string is palindrome

```java
boolean isPalindrome(String str) {
    return isPalindrome(str, 0, str.length() - 1);
}

boolean isPalindrome(String str, int left, int right) {
    // Base case: checked all characters
    if (left >= right) {
        return true;
    }
    
    // Check if characters match
    if (str.charAt(left) != str.charAt(right)) {
        return false;
    }
    
    // Recursive case: check inner substring
    return isPalindrome(str, left + 1, right - 1);
}
```

**Visual:**
```
isPalindrome("racecar")
  │
  ├─ left=0, right=6: 'r' == 'r' ✅
  ├─ isPalindrome("racecar", 1, 5)
  │    │
  │    ├─ left=1, right=5: 'a' == 'a' ✅
  │    ├─ isPalindrome("racecar", 2, 4)
  │    │    │
  │    │    ├─ left=2, right=4: 'c' == 'c' ✅
  │    │    ├─ isPalindrome("racecar", 3, 3)
  │    │    │    │
  │    │    │    └─ left >= right ✅
  │    │    │
  │    │    └─ return true
  │    │
  │    └─ return true
  │
  └─ return true ✅
```

**Time:** O(n)  
**Space:** O(n)

---

### 7. GCD (Greatest Common Divisor)

**Problem:** Find GCD using Euclidean algorithm

```java
int gcd(int a, int b) {
    // Base case
    if (b == 0) {
        return a;
    }
    
    // Recursive case: GCD(a, b) = GCD(b, a % b)
    return gcd(b, a % b);
}
```

**Visual:**
```
gcd(48, 18)
  │
  ├─ gcd(18, 48 % 18) = gcd(18, 12)
  │    │
  │    ├─ gcd(12, 18 % 12) = gcd(12, 6)
  │    │    │
  │    │    ├─ gcd(6, 12 % 6) = gcd(6, 0)
  │    │    │    │
  │    │    │    └─ b == 0, return 6 ✅
  │    │    │
  │    │    └─ return 6
  │    │
  │    └─ return 6
  │
  └─ return 6 ✅
```

**Time:** O(log(min(a, b)))  
**Space:** O(log(min(a, b)))

---

## Tree Problems with Recursion

### 1. Tree Traversal

**Pre-order (Root → Left → Right):**
```java
void preOrder(Node root) {
    if (root == null) return;
    
    System.out.print(root.val + " ");  // Process root
    preOrder(root.left);                // Left
    preOrder(root.right);               // Right
}
```

**In-order (Left → Root → Right):**
```java
void inOrder(Node root) {
    if (root == null) return;
    
    inOrder(root.left);                 // Left
    System.out.print(root.val + " ");   // Process root
    inOrder(root.right);                 // Right
}
```

**Post-order (Left → Right → Root):**
```java
void postOrder(Node root) {
    if (root == null) return;
    
    postOrder(root.left);                // Left
    postOrder(root.right);               // Right
    System.out.print(root.val + " ");    // Process root
}
```

---

### 2. Tree Height

```java
int height(Node root) {
    if (root == null) return -1;
    
    int leftHeight = height(root.left);
    int rightHeight = height(root.right);
    
    return 1 + Math.max(leftHeight, rightHeight);
}
```

---

### 3. Count Nodes

```java
int countNodes(Node root) {
    if (root == null) return 0;
    
    return 1 + countNodes(root.left) + countNodes(root.right);
}
```

---

### 4. Search in BST

```java
Node searchBST(Node root, int val) {
    if (root == null || root.val == val) {
        return root;
    }
    
    if (val < root.val) {
        return searchBST(root.left, val);
    } else {
        return searchBST(root.right, val);
    }
}
```

---

## Subset Problems

**Subset Problems:** Generate all possible subsets of a set

### Pattern: Include/Exclude Each Element

**Key Insight:** For each element, decide whether to include it or exclude it

### 1. All Subsets (Power Set)

**Problem:** Generate all subsets of an array

**Approach:** For each element, include it or exclude it

```java
List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    subsetsHelper(nums, 0, current, result);
    return result;
}

void subsetsHelper(int[] nums, int index, List<Integer> current, List<List<Integer>> result) {
    // Base case: processed all elements
    if (index == nums.length) {
        result.add(new ArrayList<>(current));  // Add current subset
        return;
    }
    
    // Choice 1: Exclude current element
    subsetsHelper(nums, index + 1, current, result);
    
    // Choice 2: Include current element
    current.add(nums[index]);
    subsetsHelper(nums, index + 1, current, result);
    current.remove(current.size() - 1);  // Backtrack
}
```

**Visual (nums = [1, 2]):**
```
subsets([1, 2])
  │
  ├─ Exclude 1: subsets([2])
  │    │
  │    ├─ Exclude 2: [] ✅
  │    └─ Include 2: [2] ✅
  │
  └─ Include 1: [1]
       │
       ├─ Exclude 2: [1] ✅
       └─ Include 2: [1, 2] ✅

Result: [[], [2], [1], [1, 2]]
```

**Time Complexity:** O(2^n × n) - 2^n subsets, each takes O(n) to copy  
**Space Complexity:** O(n) - recursion depth

### 2. Subsets with Duplicates

**Problem:** Generate all subsets, handle duplicates

**Approach:** Skip duplicates when excluding

```java
List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);  // Sort to group duplicates
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    subsetsWithDupHelper(nums, 0, current, result);
    return result;
}

void subsetsWithDupHelper(int[] nums, int index, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current));
    
    for (int i = index; i < nums.length; i++) {
        // Skip duplicates: if same as previous and not first in this level
        if (i > index && nums[i] == nums[i - 1]) {
            continue;
        }
        
        current.add(nums[i]);
        subsetsWithDupHelper(nums, i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

**Example:**
```
Input: [1, 2, 2]
Output: [[], [1], [1,2], [1,2,2], [2], [2,2]]
```

### 3. Subset Sum

**Problem:** Find all subsets that sum to target

```java
List<List<Integer>> subsetSum(int[] nums, int target) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    subsetSumHelper(nums, 0, 0, target, current, result);
    return result;
}

void subsetSumHelper(int[] nums, int index, int currentSum, int target, 
                     List<Integer> current, List<List<Integer>> result) {
    // Base case: processed all elements
    if (index == nums.length) {
        if (currentSum == target) {
            result.add(new ArrayList<>(current));
        }
        return;
    }
    
    // Choice 1: Exclude current element
    subsetSumHelper(nums, index + 1, currentSum, target, current, result);
    
    // Choice 2: Include current element
    current.add(nums[index]);
    subsetSumHelper(nums, index + 1, currentSum + nums[index], target, current, result);
    current.remove(current.size() - 1);  // Backtrack
}
```

**Visual (nums = [2, 3, 5], target = 5):**
```
subsetSum([2,3,5], 5)
  │
  ├─ Exclude 2: subsetSum([3,5], 5, sum=0)
  │    │
  │    ├─ Exclude 3: subsetSum([5], 5, sum=0)
  │    │    │
  │    │    ├─ Exclude 5: sum=0 ≠ 5 ❌
  │    │    └─ Include 5: sum=5 = 5 ✅ [5]
  │    │
  │    └─ Include 3: subsetSum([5], 5, sum=3)
  │         │
  │         ├─ Exclude 5: sum=3 ≠ 5 ❌
  │         └─ Include 5: sum=8 ≠ 5 ❌
  │
  └─ Include 2: subsetSum([3,5], 5, sum=2)
       │
       ├─ Exclude 3: subsetSum([5], 5, sum=2)
       │    │
       │    ├─ Exclude 5: sum=2 ≠ 5 ❌
       │    └─ Include 5: sum=7 ≠ 5 ❌
       │
       └─ Include 3: subsetSum([5], 5, sum=5)
            │
            ├─ Exclude 5: sum=5 = 5 ✅ [2,3]
            └─ Include 5: sum=10 ≠ 5 ❌

Result: [[5], [2,3]]
```

### 4. Combination Sum (Can Reuse Elements)

**Problem:** Find all combinations that sum to target (can reuse elements)

```java
List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    combinationSumHelper(candidates, 0, target, current, result);
    return result;
}

void combinationSumHelper(int[] candidates, int index, int remaining, 
                         List<Integer> current, List<List<Integer>> result) {
    // Base case: found valid combination
    if (remaining == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    // Base case: remaining < 0 or processed all
    if (remaining < 0 || index == candidates.length) {
        return;
    }
    
    // Try including current candidate (can reuse)
    current.add(candidates[index]);
    combinationSumHelper(candidates, index, remaining - candidates[index], current, result);
    current.remove(current.size() - 1);  // Backtrack
    
    // Try excluding current candidate (move to next)
    combinationSumHelper(candidates, index + 1, remaining, current, result);
}
```

**Visual (candidates = [2, 3, 6, 7], target = 7):**
```
combinationSum([2,3,6,7], 7)
  │
  ├─ Include 2: [2], remaining=5
  │    │
  │    ├─ Include 2: [2,2], remaining=3
  │    │    │
  │    │    ├─ Include 2: [2,2,2], remaining=1
  │    │    │    └─ Include 2: [2,2,2,2], remaining=-1 ❌
  │    │    │    └─ Exclude 2: remaining=1 ≠ 0 ❌
  │    │    │
  │    │    └─ Include 3: [2,2,3], remaining=0 ✅
  │    │
  │    └─ Include 3: [2,3], remaining=2
  │         └─ Include 2: [2,3,2], remaining=0 ✅
  │
  └─ Include 3: [3], remaining=4
       └─ Include 3: [3,3], remaining=1 ❌

Result: [[2,2,3], [7]]
```

### 5. Combination Sum II (No Duplicates)

**Problem:** Find combinations that sum to target (each number used once)

```java
List<List<Integer>> combinationSum2(int[] candidates, int target) {
    Arrays.sort(candidates);
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    combinationSum2Helper(candidates, 0, target, current, result);
    return result;
}

void combinationSum2Helper(int[] candidates, int index, int remaining,
                          List<Integer> current, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    if (remaining < 0 || index == candidates.length) {
        return;
    }
    
    for (int i = index; i < candidates.length; i++) {
        // Skip duplicates
        if (i > index && candidates[i] == candidates[i - 1]) {
            continue;
        }
        
        current.add(candidates[i]);
        combinationSum2Helper(candidates, i + 1, remaining - candidates[i], current, result);
        current.remove(current.size() - 1);
    }
}
```

---

## Backtracking

**Backtracking:** Try a path, if it doesn't work, undo and try another

### Backtracking Template

```java
void backtrack(parameters) {
    // Base case: found solution
    if (isSolution()) {
        addToResult();
        return;
    }
    
    // Try all choices
    for (each choice) {
        // Make choice
        makeChoice();
        
        // Recurse
        backtrack(updatedParameters);
        
        // Undo choice (backtrack)
        undoChoice();
    }
}
```

### 1. N-Queens Problem

**Problem:** Place n queens on n×n board so no two attack each other

```java
List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    char[][] board = new char[n][n];
    
    // Initialize board
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            board[i][j] = '.';
        }
    }
    
    solveNQueensHelper(board, 0, result);
    return result;
}

void solveNQueensHelper(char[][] board, int row, List<List<String>> result) {
    // Base case: placed all queens
    if (row == board.length) {
        result.add(constructSolution(board));
        return;
    }
    
    // Try placing queen in each column of current row
    for (int col = 0; col < board.length; col++) {
        if (isValid(board, row, col)) {
            board[row][col] = 'Q';  // Place queen
            
            // Recurse to next row
            solveNQueensHelper(board, row + 1, result);
            
            board[row][col] = '.';  // Backtrack (undo)
        }
    }
}

boolean isValid(char[][] board, int row, int col) {
    // Check column
    for (int i = 0; i < row; i++) {
        if (board[i][col] == 'Q') return false;
    }
    
    // Check diagonal (top-left to bottom-right)
    for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
        if (board[i][j] == 'Q') return false;
    }
    
    // Check diagonal (top-right to bottom-left)
    for (int i = row - 1, j = col + 1; i >= 0 && j < board.length; i--, j++) {
        if (board[i][j] == 'Q') return false;
    }
    
    return true;
}
```

**Visual (4-Queens):**
```
Try row 0:
  Place Q at (0,0) ✅
  Try row 1:
    Place Q at (1,2) ✅
    Try row 2:
      Place Q at (2,1) ✅
      Try row 3:
        No valid position ❌
        Backtrack: Remove Q from (2,1)
      Try row 2:
        Place Q at (2,3) ✅
        Try row 3:
          Place Q at (3,1) ✅
          Solution found! ✅
```

---

### 2. Generate Parentheses

**Problem:** Generate all valid combinations of n pairs of parentheses

```java
List<String> generateParenthesis(int n) {
    List<String> result = new ArrayList<>();
    generateParenthesisHelper("", 0, 0, n, result);
    return result;
}

void generateParenthesisHelper(String current, int open, int close, int n, List<String> result) {
    // Base case: generated n pairs
    if (current.length() == 2 * n) {
        result.add(current);
        return;
    }
    
    // Add opening parenthesis if we haven't used all
    if (open < n) {
        generateParenthesisHelper(current + "(", open + 1, close, n, result);
    }
    
    // Add closing parenthesis if we have more open than close
    if (close < open) {
        generateParenthesisHelper(current + ")", open, close + 1, n, result);
    }
}
```

**Visual (n=2):**
```
generateParenthesis(2)
  │
  ├─ "(" (open=1, close=0)
  │    │
  │    ├─ "((" (open=2, close=0)
  │    │    │
  │    │    ├─ "(()" (open=2, close=1)
  │    │    │    │
  │    │    │    └─ "(())" ✅ (open=2, close=2)
  │    │    │
  │    │    └─ "(())" ✅
  │    │
  │    └─ "()" (open=1, close=1)
  │         │
  │         ├─ "()(" (open=2, close=1)
  │         │    │
  │         │    └─ "()()" ✅
  │         │
  │         └─ "()()" ✅
  │
  Result: ["(())", "()()"]
```

### 3. Permutations

**Problem:** Generate all permutations of an array

**Approach:** For each position, try each remaining element

```java
List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    permuteHelper(nums, used, current, result);
    return result;
}

void permuteHelper(int[] nums, boolean[] used, List<Integer> current, List<List<Integer>> result) {
    // Base case: all elements used
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    // Try each unused element
    for (int i = 0; i < nums.length; i++) {
        if (!used[i]) {
            // Make choice
            used[i] = true;
            current.add(nums[i]);
            
            // Recurse
            permuteHelper(nums, used, current, result);
            
            // Backtrack
            current.remove(current.size() - 1);
            used[i] = false;
        }
    }
}
```

**Visual (nums = [1, 2, 3]):**
```
permute([1,2,3])
  │
  ├─ Position 0: 1
  │    │
  │    ├─ Position 1: 2
  │    │    │
  │    │    └─ Position 2: 3 → [1,2,3] ✅
  │    │
  │    └─ Position 1: 3
  │         │
  │         └─ Position 2: 2 → [1,3,2] ✅
  │
  ├─ Position 0: 2
  │    │
  │    ├─ Position 1: 1
  │    │    └─ Position 2: 3 → [2,1,3] ✅
  │    │
  │    └─ Position 1: 3
  │         └─ Position 2: 1 → [2,3,1] ✅
  │
  └─ Position 0: 3
       │
       ├─ Position 1: 1
       │    └─ Position 2: 2 → [3,1,2] ✅
       │
       └─ Position 1: 2
            └─ Position 2: 1 → [3,2,1] ✅

Result: [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
```

**Time Complexity:** O(n! × n) - n! permutations, each takes O(n) to copy  
**Space Complexity:** O(n) - recursion depth

### 4. Permutations with Duplicates

**Problem:** Generate all unique permutations (handle duplicates)

```java
List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    permuteUniqueHelper(nums, used, current, result);
    return result;
}

void permuteUniqueHelper(int[] nums, boolean[] used, List<Integer> current, List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = 0; i < nums.length; i++) {
        // Skip if used
        if (used[i]) continue;
        
        // Skip duplicates: if same as previous and previous not used
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
            continue;
        }
        
        used[i] = true;
        current.add(nums[i]);
        permuteUniqueHelper(nums, used, current, result);
        current.remove(current.size() - 1);
        used[i] = false;
    }
}
```

### 5. Combinations

**Problem:** Generate all combinations of k elements from n elements

```java
List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> current = new ArrayList<>();
    combineHelper(n, k, 1, current, result);
    return result;
}

void combineHelper(int n, int k, int start, List<Integer> current, List<List<Integer>> result) {
    // Base case: found k elements
    if (current.size() == k) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    // Try each number from start to n
    for (int i = start; i <= n; i++) {
        current.add(i);
        combineHelper(n, k, i + 1, current, result);  // Next start is i+1
        current.remove(current.size() - 1);  // Backtrack
    }
}
```

**Visual (n=4, k=2):**
```
combine(4, 2)
  │
  ├─ Include 1: [1]
  │    │
  │    ├─ Include 2: [1,2] ✅
  │    ├─ Include 3: [1,3] ✅
  │    └─ Include 4: [1,4] ✅
  │
  ├─ Include 2: [2]
  │    │
  │    ├─ Include 3: [2,3] ✅
  │    └─ Include 4: [2,4] ✅
  │
  └─ Include 3: [3]
       │
       └─ Include 4: [3,4] ✅

Result: [[1,2], [1,3], [1,4], [2,3], [2,4], [3,4]]
```

### 6. Word Search (Backtracking on 2D Grid)

**Problem:** Find if word exists in 2D board

```java
boolean exist(char[][] board, String word) {
    int m = board.length;
    int n = board[0].length;
    
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (existHelper(board, word, i, j, 0)) {
                return true;
            }
        }
    }
    return false;
}

boolean existHelper(char[][] board, String word, int row, int col, int index) {
    // Base case: found entire word
    if (index == word.length()) {
        return true;
    }
    
    // Base case: out of bounds or character doesn't match
    if (row < 0 || row >= board.length || 
        col < 0 || col >= board[0].length ||
        board[row][col] != word.charAt(index)) {
        return false;
    }
    
    // Mark as visited
    char temp = board[row][col];
    board[row][col] = '#';
    
    // Try all 4 directions
    boolean found = existHelper(board, word, row + 1, col, index + 1) ||
                   existHelper(board, word, row - 1, col, index + 1) ||
                   existHelper(board, word, row, col + 1, index + 1) ||
                   existHelper(board, word, row, col - 1, index + 1);
    
    // Backtrack: restore character
    board[row][col] = temp;
    
    return found;
}
```

**Visual:**
```
Board:
A B C E
S F C S
A D E E

Word: "ABCCED"

Start at (0,0): A matches
  Try (0,1): B matches
    Try (0,2): C matches
      Try (1,2): C matches
        Try (2,2): E matches
          Try (2,1): D matches ✅
```

### 7. Sudoku Solver

**Problem:** Solve 9×9 Sudoku puzzle

```java
boolean solveSudoku(char[][] board) {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (board[i][j] == '.') {
                // Try each digit 1-9
                for (char c = '1'; c <= '9'; c++) {
                    if (isValid(board, i, j, c)) {
                        board[i][j] = c;  // Make choice
                        
                        if (solveSudoku(board)) {  // Recurse
                            return true;
                        }
                        
                        board[i][j] = '.';  // Backtrack
                    }
                }
                return false;  // No valid digit
            }
        }
    }
    return true;  // All cells filled
}

boolean isValid(char[][] board, int row, int col, char c) {
    for (int i = 0; i < 9; i++) {
        // Check row
        if (board[row][i] == c) return false;
        // Check column
        if (board[i][col] == c) return false;
        // Check 3×3 box
        if (board[3 * (row / 3) + i / 3][3 * (col / 3) + i % 3] == c) {
            return false;
        }
    }
    return true;
}
```

### 8. Letter Combinations of Phone Number

**Problem:** Generate all letter combinations from phone number digits

```java
List<String> letterCombinations(String digits) {
    if (digits.isEmpty()) return new ArrayList<>();
    
    String[] mapping = {
        "", "", "abc", "def", "ghi", "jkl",
        "mno", "pqrs", "tuv", "wxyz"
    };
    
    List<String> result = new ArrayList<>();
    letterCombinationsHelper(digits, 0, "", mapping, result);
    return result;
}

void letterCombinationsHelper(String digits, int index, String current,
                              String[] mapping, List<String> result) {
    // Base case: processed all digits
    if (index == digits.length()) {
        result.add(current);
        return;
    }
    
    // Get letters for current digit
    String letters = mapping[digits.charAt(index) - '0'];
    
    // Try each letter
    for (char c : letters.toCharArray()) {
        letterCombinationsHelper(digits, index + 1, current + c, mapping, result);
    }
}
```

**Visual (digits = "23"):**
```
letterCombinations("23")
  │
  ├─ Digit 2: "abc"
  │    │
  │    ├─ "a" + combinations("3")
  │    │    │
  │    │    ├─ "ad" ✅
  │    │    ├─ "ae" ✅
  │    │    └─ "af" ✅
  │    │
  │    ├─ "b" + combinations("3")
  │    │    ├─ "bd" ✅
  │    │    ├─ "be" ✅
  │    │    └─ "bf" ✅
  │    │
  │    └─ "c" + combinations("3")
  │         ├─ "cd" ✅
  │         ├─ "ce" ✅
  │         └─ "cf" ✅

Result: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

### 9. Restore IP Addresses

**Problem:** Generate all valid IP addresses from string

```java
List<String> restoreIpAddresses(String s) {
    List<String> result = new ArrayList<>();
    List<String> current = new ArrayList<>();
    restoreIpHelper(s, 0, current, result);
    return result;
}

void restoreIpHelper(String s, int index, List<String> current, List<String> result) {
    // Base case: 4 segments and used all characters
    if (current.size() == 4 && index == s.length()) {
        result.add(String.join(".", current));
        return;
    }
    
    // Base case: 4 segments but not all characters used, or vice versa
    if (current.size() == 4 || index == s.length()) {
        return;
    }
    
    // Try 1, 2, or 3 digit segments
    for (int len = 1; len <= 3 && index + len <= s.length(); len++) {
        String segment = s.substring(index, index + len);
        
        // Validate segment
        if (isValidSegment(segment)) {
            current.add(segment);
            restoreIpHelper(s, index + len, current, result);
            current.remove(current.size() - 1);  // Backtrack
        }
    }
}

boolean isValidSegment(String segment) {
    if (segment.length() > 1 && segment.charAt(0) == '0') {
        return false;  // No leading zeros
    }
    int num = Integer.parseInt(segment);
    return num >= 0 && num <= 255;
}
```

---

## Dynamic Programming with Recursion

### Memoization (Top-Down)

**Store results to avoid recomputation**

```java
Map<Integer, Integer> memo = new HashMap<>();

int fibonacciMemo(int n) {
    // Base cases
    if (n <= 1) return n;
    
    // Check if already computed
    if (memo.containsKey(n)) {
        return memo.get(n);
    }
    
    // Compute and store
    int result = fibonacciMemo(n - 1) + fibonacciMemo(n - 2);
    memo.put(n, result);
    return result;
}
```

**Visual:**
```
fibonacciMemo(5)
  │
  ├─ fib(4) + fib(3)
  │    │
  │    ├─ fib(3) + fib(2) → Store memo[3]=2, memo[2]=1
  │    │    │
  │    │    ├─ fib(2) + fib(1) → Store memo[2]=1
  │    │    └─ fib(1) + fib(0) = 1 + 0 = 1
  │    │
  │    └─ Use memo[2]=1 (already computed!)
  │
  └─ Use memo[3]=2 (already computed!)

Result: 3 + 2 = 5 ✅
```

---

### Climbing Stairs

**Problem:** Ways to climb n stairs (1 or 2 steps at a time)

```java
Map<Integer, Integer> memo = new HashMap<>();

int climbStairs(int n) {
    // Base cases
    if (n <= 2) return n;
    
    // Check memo
    if (memo.containsKey(n)) {
        return memo.get(n);
    }
    
    // Recurrence: ways(n) = ways(n-1) + ways(n-2)
    int result = climbStairs(n - 1) + climbStairs(n - 2);
    memo.put(n, result);
    return result;
}
```

**Visual:**
```
climbStairs(4)
  │
  ├─ climbStairs(3) + climbStairs(2)
  │    │
  │    ├─ climbStairs(2) + climbStairs(1)
  │    │    │
  │    │    ├─ climbStairs(2) = 2 ✅
  │    │    └─ climbStairs(1) = 1 ✅
  │    │    │
  │    │    └─ 2 + 1 = 3
  │    │
  │    └─ climbStairs(2) = 2 ✅
  │
  └─ 3 + 2 = 5 ✅

Ways to climb 4 stairs: 5
```

---

## Common Mistakes

### Mistake 1: Missing Base Case

**❌ WRONG:**
```java
int factorial(int n) {
    return n * factorial(n - 1);  // Infinite recursion!
}
```

**✅ CORRECT:**
```java
int factorial(int n) {
    if (n <= 1) return 1;  // Base case
    return n * factorial(n - 1);
}
```

### Mistake 2: Not Approaching Base Case

**❌ WRONG:**
```java
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n);  // n never changes!
}
```

**✅ CORRECT:**
```java
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1);  // n decreases
}
```

### Mistake 3: Stack Overflow

**Problem:** Deep recursion causes stack overflow

**Solution:** Use iteration or tail recursion optimization

```java
// ❌ Deep recursion
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1);
}

// ✅ Iterative (no stack overflow)
int sumIterative(int n) {
    int result = 0;
    for (int i = 1; i <= n; i++) {
        result += i;
    }
    return result;
}
```

---

## Optimization Techniques

### 1. Memoization

**Cache results to avoid recomputation**

```java
Map<String, Integer> memo = new HashMap<>();

int expensiveOperation(String input) {
    if (memo.containsKey(input)) {
        return memo.get(input);
    }
    
    int result = /* expensive computation */;
    memo.put(input, result);
    return result;
}
```

### 2. Tail Recursion

**Make recursive call the last operation**

```java
// ❌ Not tail recursive
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);  // Multiplication after call
}

// ✅ Tail recursive
int factorialTail(int n, int acc) {
    if (n <= 1) return acc;
    return factorialTail(n - 1, n * acc);  // Call is last operation
}
```

### 3. Convert to Iteration

**Eliminate recursion for better performance**

```java
// Recursive
int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// Iterative
int fibonacciIterative(int n) {
    if (n <= 1) return n;
    
    int prev = 0, curr = 1;
    for (int i = 2; i <= n; i++) {
        int next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}
```

---

## Problem Identification Patterns

### When to Use Recursion?

**Pattern 1: Tree/Graph Traversal**
- Problem involves traversing nodes
- Natural recursive structure
- Examples: Tree traversal, Graph DFS, Path finding

**Pattern 2: Divide and Conquer**
- Problem can be split into smaller subproblems
- Subproblems are similar to original
- Examples: Merge sort, Quick sort, Binary search

**Pattern 3: Backtracking**
- Need to try all possibilities
- Can undo choices if they don't work
- Examples: N-Queens, Sudoku, Permutations

**Pattern 4: Subset/Combination Problems**
- Generate all subsets/combinations
- For each element: include or exclude
- Examples: Power set, Subset sum, Combinations

**Pattern 5: String/Array Manipulation**
- Process character/element by character/element
- Build solution incrementally
- Examples: String reversal, Palindrome, Permutations

### Decision Tree: Recursion vs Iteration

```
Is problem naturally recursive?
├─ YES → Use Recursion
│   ├─ Tree/Graph? → Recursion
│   ├─ Divide & Conquer? → Recursion
│   ├─ Backtracking? → Recursion
│   └─ Subset/Combination? → Recursion
│
└─ NO → Consider Iteration
    ├─ Simple loop? → Iteration
    ├─ Performance critical? → Iteration
    └─ Deep recursion risk? → Iteration
```

### Common Recursion Patterns

#### Pattern 1: Include/Exclude Pattern
**Use for:** Subsets, Combinations, Subset Sum

```java
void includeExclude(int[] nums, int index, List<Integer> current) {
    if (index == nums.length) {
        // Process current subset
        return;
    }
    
    // Exclude
    includeExclude(nums, index + 1, current);
    
    // Include
    current.add(nums[index]);
    includeExclude(nums, index + 1, current);
    current.remove(current.size() - 1);  // Backtrack
}
```

#### Pattern 2: Try All Choices Pattern
**Use for:** Permutations, N-Queens, Sudoku

```java
void tryAllChoices(int[] choices, boolean[] used, List<Integer> current) {
    if (current.size() == choices.length) {
        // Process current solution
        return;
    }
    
    for (int i = 0; i < choices.length; i++) {
        if (!used[i]) {
            used[i] = true;
            current.add(choices[i]);
            tryAllChoices(choices, used, current);
            current.remove(current.size() - 1);
            used[i] = false;
        }
    }
}
```

#### Pattern 3: Two Choices Pattern
**Use for:** Generate Parentheses, Binary Strings

```java
void twoChoices(String current, int open, int close, int n) {
    if (current.length() == 2 * n) {
        // Process solution
        return;
    }
    
    // Choice 1
    if (open < n) {
        twoChoices(current + "(", open + 1, close, n);
    }
    
    // Choice 2
    if (close < open) {
        twoChoices(current + ")", open, close + 1, n);
    }
}
```

#### Pattern 4: DFS on Grid Pattern
**Use for:** Word Search, Island Problems, Maze

```java
boolean dfsGrid(char[][] grid, int row, int col, int index, String word) {
    // Base cases
    if (index == word.length()) return true;
    if (invalidPosition(row, col, grid)) return false;
    if (grid[row][col] != word.charAt(index)) return false;
    
    // Mark visited
    char temp = grid[row][col];
    grid[row][col] = '#';
    
    // Try 4 directions
    boolean found = dfsGrid(grid, row+1, col, index+1, word) ||
                   dfsGrid(grid, row-1, col, index+1, word) ||
                   dfsGrid(grid, row, col+1, index+1, word) ||
                   dfsGrid(grid, row, col-1, index+1, word);
    
    // Backtrack
    grid[row][col] = temp;
    return found;
}
```

### Problem Classification

| Problem Type | Pattern | Example |
|-------------|---------|---------|
| **Subsets** | Include/Exclude | All subsets, Subset sum |
| **Permutations** | Try all choices | All permutations, Permutations with duplicates |
| **Combinations** | Include/Exclude with constraint | nCk, Combination sum |
| **Backtracking** | Try and undo | N-Queens, Sudoku, Word search |
| **Tree Problems** | DFS traversal | Tree height, Count nodes, Path sum |
| **Divide & Conquer** | Split and merge | Merge sort, Quick sort, Power |
| **String Problems** | Character by character | Palindrome, String reversal, Permutations |

---

## Interview Questions

### Q1: What is Recursion?

**Answer:**
"Recursion is a programming technique where a function calls itself to solve a problem. It consists of:
1. **Base case**: Condition that stops recursion
2. **Recursive case**: Function calls itself with smaller input

For example, to calculate factorial:
- Base case: factorial(0) = 1
- Recursive case: factorial(n) = n × factorial(n-1)

The key is that the input must get closer to the base case with each recursive call, otherwise we get infinite recursion."

### Q2: What is the Difference Between Recursion and Iteration?

**Answer:**
"Recursion and iteration both repeat operations, but differently:

**Recursion:**
- Function calls itself
- Uses call stack (memory overhead)
- More intuitive for tree/graph problems
- Can cause stack overflow for deep recursion

**Iteration:**
- Uses loops (for/while)
- No call stack overhead
- More memory efficient
- Better for simple repetitive tasks

I choose recursion for:
- Tree traversal
- Divide and conquer problems
- Backtracking

I choose iteration for:
- Simple loops
- Performance-critical code
- When recursion depth is unknown"

### Q3: How Does the Call Stack Work in Recursion?

**Answer:**
"The call stack is a data structure that tracks function calls. When a function calls itself:
1. Each recursive call pushes a new frame onto the stack
2. The frame stores local variables and return address
3. When base case is reached, frames are popped in reverse order
4. Each frame returns its result to the caller

For example, factorial(4):
- Stack: [factorial(4)] → [factorial(4), factorial(3)] → [factorial(4), factorial(3), factorial(2)] → ...
- When factorial(1) returns 1, stack unwinds
- Each frame multiplies its n with the returned value

The maximum stack depth equals the recursion depth, which can cause stack overflow for deep recursion."

### Q4: What is Tail Recursion?

**Answer:**
"Tail recursion is when the recursive call is the last operation in the function. For example:

```java
// Tail recursive
int sumTail(int n, int acc) {
    if (n == 0) return acc;
    return sumTail(n - 1, acc + n);  // Last operation
}

// Not tail recursive
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1);  // Addition after call
}
```

The benefit of tail recursion is that compilers can optimize it to iteration, eliminating stack growth. This prevents stack overflow and improves performance."

### Q5: How Do You Avoid Stack Overflow in Recursion?

**Answer:**
"I avoid stack overflow by:
1. **Ensure base case is reached**: Input must approach base case
2. **Use iteration for deep recursion**: Convert recursive solution to iterative
3. **Use tail recursion**: Allows compiler optimization
4. **Increase stack size**: JVM option -Xss (not recommended)
5. **Use memoization**: Reduces redundant calls

For example, for calculating large Fibonacci numbers, I use iterative approach or memoization instead of naive recursion to avoid exponential time and stack overflow."

### Q6: What is Memoization?

**Answer:**
"Memoization is an optimization technique that stores results of expensive function calls and returns cached result when same inputs occur again.

For example, in Fibonacci:
- Without memoization: O(2^n) time, many redundant calculations
- With memoization: O(n) time, each value calculated once

I use memoization when:
- Function has overlapping subproblems
- Same inputs are computed multiple times
- Time complexity can be improved significantly

I implement it using HashMap to store computed results."

### Q7: Explain Backtracking

**Answer:**
"Backtracking is a recursive algorithm that tries to build a solution incrementally, and abandons a path (backtracks) as soon as it determines the path cannot lead to a valid solution.

Key steps:
1. **Try**: Make a choice
2. **Recurse**: Solve subproblem with choice
3. **Backtrack**: Undo choice if it doesn't lead to solution

Example: N-Queens problem
- Try placing queen in a position
- If valid, recurse to place next queen
- If no valid positions, backtrack and try different position

I use backtracking for:
- Constraint satisfaction problems
- Generating all combinations/permutations
- Path finding problems"

### Q8: What is the Time Complexity of Recursive Fibonacci?

**Answer:"
"The naive recursive Fibonacci has O(2^n) time complexity because:
- Each call makes two recursive calls
- Height of recursion tree is n
- Total nodes ≈ 2^n

However, with memoization, it becomes O(n) because:
- Each value is calculated only once
- We make n recursive calls total

I always mention both complexities and explain when to use memoization for optimization."

### Q9: How Do You Debug Recursion?

**Answer:**
"I debug recursion by:
1. **Add print statements**: Print function parameters and return values
2. **Track recursion depth**: Print depth to see call stack
3. **Check base case**: Ensure it's reached
4. **Visualize call tree**: Draw the recursion tree
5. **Use debugger**: Step through recursive calls
6. **Add assertions**: Verify preconditions and postconditions

For example:
```java
int factorial(int n, int depth) {
    System.out.println("Depth: " + depth + ", n: " + n);
    if (n <= 1) return 1;
    return n * factorial(n - 1, depth + 1);
}
```"

### Q10: When Should You Use Recursion vs Iteration?

**Answer:**
"I use recursion when:
- Problem has natural recursive structure (trees, graphs)
- Solution is more intuitive with recursion
- Divide and conquer approach
- Backtracking problems

I use iteration when:
- Simple repetitive tasks
- Performance is critical
- Recursion depth is unknown/large
- Memory is constrained

For example:
- Tree traversal: Recursion (natural fit)
- Factorial: Either works, but iteration is more efficient
- Fibonacci: Iteration or memoized recursion (naive recursion is too slow)"

### Q11: How do you generate all subsets of an array?

**Answer:**
"I use the include/exclude pattern. For each element, I make two choices: include it or exclude it. I use backtracking to explore all possibilities.

```java
List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    subsetsHelper(nums, 0, new ArrayList<>(), result);
    return result;
}

void subsetsHelper(int[] nums, int index, List<Integer> current, List<List<Integer>> result) {
    if (index == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    // Exclude
    subsetsHelper(nums, index + 1, current, result);
    
    // Include
    current.add(nums[index]);
    subsetsHelper(nums, index + 1, current, result);
    current.remove(current.size() - 1);  // Backtrack
}
```

Time complexity is O(2^n × n) because there are 2^n subsets and each takes O(n) to copy. Space is O(n) for recursion depth."

### Q12: Explain the difference between permutations and combinations.

**Answer:**
"Permutations consider order, combinations don't.

**Permutations:** Arrangements where order matters
- Example: [1,2,3] and [2,1,3] are different
- Count: n! / (n-k)!
- Use when: Order matters (passwords, arrangements)

**Combinations:** Selections where order doesn't matter
- Example: {1,2,3} and {2,1,3} are same
- Count: n! / (k! × (n-k)!)
- Use when: Order doesn't matter (teams, groups)

**Code difference:**
- Permutations: Try all positions, use boolean array to track used elements
- Combinations: Only try elements after current index (no need to track used)

For permutations, I iterate through all indices. For combinations, I only consider elements after the current index to avoid duplicates."

### Q13: How do you solve the subset sum problem?

**Answer:**
"I use backtracking with the include/exclude pattern. For each element, I decide whether to include it in the current subset. I track the current sum and compare it with the target.

```java
List<List<Integer>> subsetSum(int[] nums, int target) {
    List<List<Integer>> result = new ArrayList<>();
    subsetSumHelper(nums, 0, 0, target, new ArrayList<>(), result);
    return result;
}

void subsetSumHelper(int[] nums, int index, int sum, int target,
                     List<Integer> current, List<List<Integer>> result) {
    if (index == nums.length) {
        if (sum == target) {
            result.add(new ArrayList<>(current));
        }
        return;
    }
    
    // Exclude
    subsetSumHelper(nums, index + 1, sum, target, current, result);
    
    // Include
    current.add(nums[index]);
    subsetSumHelper(nums, index + 1, sum + nums[index], target, current, result);
    current.remove(current.size() - 1);
}
```

I can optimize by pruning: if current sum exceeds target, I can skip further exploration. Time complexity is O(2^n) in worst case."

### Q14: How do you generate all permutations?

**Answer:**
"I use backtracking with a boolean array to track which elements have been used. For each position, I try all unused elements.

```java
List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    permuteHelper(nums, used, new ArrayList<>(), result);
    return result;
}

void permuteHelper(int[] nums, boolean[] used, List<Integer> current, List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = 0; i < nums.length; i++) {
        if (!used[i]) {
            used[i] = true;
            current.add(nums[i]);
            permuteHelper(nums, used, current, result);
            current.remove(current.size() - 1);
            used[i] = false;  // Backtrack
        }
    }
}
```

Time complexity is O(n! × n) because there are n! permutations and each takes O(n) to copy. For duplicates, I sort first and skip when same as previous and previous not used."

### Q15: Explain the backtracking algorithm for N-Queens.

**Answer:**
"N-Queens uses backtracking to place queens row by row. For each row, I try placing a queen in each column. If valid, I recurse to the next row. If no valid position, I backtrack.

```java
void solveNQueens(char[][] board, int row, List<List<String>> result) {
    if (row == board.length) {
        result.add(constructSolution(board));
        return;
    }
    
    for (int col = 0; col < board.length; col++) {
        if (isValid(board, row, col)) {
            board[row][col] = 'Q';  // Place queen
            solveNQueens(board, row + 1, result);  // Recurse
            board[row][col] = '.';  // Backtrack
        }
    }
}
```

The key is the isValid function that checks:
1. No queen in same column
2. No queen in same diagonal (both directions)

Time complexity is O(n!) in worst case, but pruning makes it much better in practice."

### Q16: How do you solve the word search problem?

**Answer:**
"I use DFS backtracking on a 2D grid. For each cell, I check if it matches the current character of the word. If yes, I mark it as visited and recursively search in all 4 directions. After recursion, I unmark it (backtrack).

```java
boolean exist(char[][] board, String word) {
    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[0].length; j++) {
            if (existHelper(board, word, i, j, 0)) {
                return true;
            }
        }
    }
    return false;
}

boolean existHelper(char[][] board, String word, int row, int col, int index) {
    if (index == word.length()) return true;
    if (invalidPosition(row, col, board)) return false;
    if (board[row][col] != word.charAt(index)) return false;
    
    char temp = board[row][col];
    board[row][col] = '#';  // Mark visited
    
    boolean found = existHelper(board, word, row+1, col, index+1) ||
                   existHelper(board, word, row-1, col, index+1) ||
                   existHelper(board, word, row, col+1, index+1) ||
                   existHelper(board, word, row, col-1, index+1);
    
    board[row][col] = temp;  // Backtrack
    return found;
}
```

Time complexity is O(m × n × 4^L) where L is word length. Space is O(L) for recursion depth."

### Q17: What is the difference between combination sum and subset sum?

**Answer:**
"Both find subsets that sum to target, but with different constraints:

**Subset Sum:**
- Each element used at most once
- Find all subsets that sum to target
- Example: [2,3,5], target=5 → [[2,3], [5]]

**Combination Sum:**
- Elements can be reused
- Find all combinations that sum to target
- Example: [2,3,6,7], target=7 → [[2,2,3], [7]]

**Code difference:**
- Subset Sum: Move to next index after including
- Combination Sum: Can stay at same index to reuse element

```java
// Subset Sum: index + 1 (no reuse)
subsetSumHelper(nums, index + 1, sum + nums[index], ...);

// Combination Sum: index (can reuse)
combinationSumHelper(candidates, index, remaining - candidates[index], ...);
```"

### Q18: How do you generate all combinations of k elements from n?

**Answer:**
"I use backtracking where I only consider elements after the current index to avoid duplicates and maintain order.

```java
List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> result = new ArrayList<>();
    combineHelper(n, k, 1, new ArrayList<>(), result);
    return result;
}

void combineHelper(int n, int k, int start, List<Integer> current, List<List<Integer>> result) {
    if (current.size() == k) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = start; i <= n; i++) {
        current.add(i);
        combineHelper(n, k, i + 1, current, result);  // Next start is i+1
        current.remove(current.size() - 1);
    }
}
```

Key insight: Start from `start` parameter, not 0, to avoid duplicates. Time complexity is O(C(n,k) × k) where C(n,k) is binomial coefficient."

### Q19: Explain how to solve Sudoku using backtracking.

**Answer:**
"I use backtracking to fill empty cells one by one. For each empty cell, I try digits 1-9. If a digit is valid, I place it and recurse. If recursion returns false, I backtrack and try next digit.

```java
boolean solveSudoku(char[][] board) {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (board[i][j] == '.') {
                for (char c = '1'; c <= '9'; c++) {
                    if (isValid(board, i, j, c)) {
                        board[i][j] = c;
                        if (solveSudoku(board)) {
                            return true;
                        }
                        board[i][j] = '.';  // Backtrack
                    }
                }
                return false;  // No valid digit
            }
        }
    }
    return true;  // All filled
}
```

The isValid function checks row, column, and 3×3 box. Time complexity is exponential but pruning makes it practical."

### Q20: How do you handle duplicates in permutation/combination problems?

**Answer:**
"I handle duplicates by:
1. **Sorting the array first** - Groups duplicates together
2. **Skipping duplicates** - When same as previous and previous not used (for permutations) or when same as previous at same level (for combinations)

**For Permutations:**
```java
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) {
    continue;  // Skip duplicate
}
```

**For Combinations:**
```java
if (i > index && candidates[i] == candidates[i-1]) {
    continue;  // Skip duplicate at same level
}
```

The key difference: Permutations check if previous was used, combinations check if we're at the same level (index)."

### Q21: What is the time complexity of generating all subsets?

**Answer:**
"The time complexity is O(2^n × n) where:
- 2^n is the number of subsets (each element: include or exclude)
- n is the time to copy each subset to result

Space complexity is O(n) for:
- Recursion stack depth: O(n)
- Current subset: O(n)

For example, array of size 3 has 2^3 = 8 subsets:
- [] (exclude all)
- [1], [2], [3] (include one)
- [1,2], [1,3], [2,3] (include two)
- [1,2,3] (include all)

I can optimize by using bit manipulation for O(2^n) subsets, but the copy operation still takes O(n) per subset."

### Q22: How do you optimize backtracking problems?

**Answer:**
"I optimize backtracking using:

1. **Pruning**: Stop early if current path can't lead to solution
   ```java
   if (currentSum > target) return;  // Prune
   ```

2. **Memoization**: Cache results for overlapping subproblems
   ```java
   if (memo.containsKey(state)) return memo.get(state);
   ```

3. **Constraint Propagation**: Reduce search space early
   ```java
   if (!isValid(board, row, col)) continue;  // Skip invalid
   ```

4. **Heuristics**: Try most promising choices first
   ```java
   // Sort candidates to try smaller values first
   Arrays.sort(candidates);
   ```

5. **Early Termination**: Return immediately when solution found
   ```java
   if (found) return true;  // Don't explore further
   ```

These optimizations can reduce time from exponential to polynomial in many cases."

### Q23: Explain the include/exclude pattern in recursion.

**Answer:**
"The include/exclude pattern is used for subset and combination problems. For each element, I make two choices:

1. **Exclude**: Don't add to current solution, move to next
2. **Include**: Add to current solution, recurse, then backtrack

```java
void includeExclude(int[] nums, int index, List<Integer> current) {
    if (index == nums.length) {
        process(current);  // Base case
        return;
    }
    
    // Choice 1: Exclude
    includeExclude(nums, index + 1, current);
    
    // Choice 2: Include
    current.add(nums[index]);
    includeExclude(nums, index + 1, current);
    current.remove(current.size() - 1);  // Backtrack
}
```

This pattern generates all 2^n subsets. I use it for:
- All subsets
- Subset sum
- Combinations
- Any problem where each element has two choices"

### Q24: How do you solve the letter combinations of phone number problem?

**Answer:**
"I use recursion to build combinations character by character. For each digit, I get its corresponding letters and try each letter.

```java
List<String> letterCombinations(String digits) {
    String[] mapping = {"", "", "abc", "def", "ghi", "jkl",
                       "mno", "pqrs", "tuv", "wxyz"};
    List<String> result = new ArrayList<>();
    letterCombinationsHelper(digits, 0, "", mapping, result);
    return result;
}

void letterCombinationsHelper(String digits, int index, String current,
                              String[] mapping, List<String> result) {
    if (index == digits.length()) {
        result.add(current);
        return;
    }
    
    String letters = mapping[digits.charAt(index) - '0'];
    for (char c : letters.toCharArray()) {
        letterCombinationsHelper(digits, index + 1, current + c, mapping, result);
    }
}
```

Time complexity is O(4^n × n) where 4 is average letters per digit. Space is O(n) for recursion depth."

### Q25: What are the key differences between backtracking and dynamic programming?

**Answer:**
"Backtracking and DP both solve problems recursively, but differently:

**Backtracking:**
- Explores all possibilities
- Backtracks when path doesn't work
- Used for: Permutations, Combinations, Constraint satisfaction
- Time: Often exponential
- Space: O(depth) for recursion

**Dynamic Programming:**
- Stores results of subproblems
- Avoids recomputation
- Used for: Optimization problems, Overlapping subproblems
- Time: Polynomial (after memoization)
- Space: O(n) or O(n²) for memo table

**Key difference:**
- Backtracking: Tries all paths, discards invalid ones
- DP: Computes each subproblem once, reuses results

**When to use:**
- Backtracking: Need all solutions, constraint satisfaction
- DP: Need optimal solution, overlapping subproblems"

### Q26: How do you identify if a problem requires backtracking?

**Answer:**
"I identify backtracking problems by these characteristics:

1. **Need to try all possibilities**: Generate all permutations, combinations, or solutions
2. **Can undo choices**: If a choice doesn't work, can backtrack and try another
3. **Constraint satisfaction**: Need to satisfy multiple constraints (N-Queens, Sudoku)
4. **Decision tree structure**: Problem can be visualized as a decision tree
5. **Incremental building**: Build solution step by step, can abandon partial solutions

**Common problem types:**
- Generate all X (permutations, combinations, subsets)
- Find if path exists (word search, maze)
- Constraint satisfaction (N-Queens, Sudoku)
- Partition problems (restore IP addresses)

**Red flags for backtracking:**
- 'Generate all' in problem statement
- 'Find all possible' solutions
- Constraint satisfaction problems
- Problems where you can 'try and undo'"

### Q27: Explain the time complexity of permutation generation.

**Answer:**
"The time complexity of generating all permutations is O(n! × n) where:
- n! is the number of permutations of n elements
- n is the time to copy each permutation to result

**Why n!?**
- First position: n choices
- Second position: n-1 choices
- Third position: n-2 choices
- ...
- Total: n × (n-1) × (n-2) × ... × 1 = n!

**Space complexity:**
- Recursion stack: O(n)
- Boolean array: O(n)
- Current permutation: O(n)
- Total: O(n)

For example, 3 elements have 3! = 6 permutations:
[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]

I can optimize by using swapping instead of boolean array, but time complexity remains O(n! × n)."

### Q28: How do you solve combination sum problems?

**Answer:**
"Combination sum problems use backtracking with the ability to reuse elements. I track the remaining target and try including each candidate.

```java
List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    combinationSumHelper(candidates, 0, target, new ArrayList<>(), result);
    return result;
}

void combinationSumHelper(int[] candidates, int index, int remaining,
                         List<Integer> current, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    if (remaining < 0 || index == candidates.length) {
        return;
    }
    
    // Include current candidate (can reuse)
    current.add(candidates[index]);
    combinationSumHelper(candidates, index, remaining - candidates[index], current, result);
    current.remove(current.size() - 1);
    
    // Exclude current candidate (move to next)
    combinationSumHelper(candidates, index + 1, remaining, current, result);
}
```

**Key points:**
- Can reuse: Pass same `index` when including
- Pruning: Return early if `remaining < 0`
- Sort candidates first for better pruning

Time complexity is exponential but pruning helps significantly."

### Q29: What is the difference between subset and combination?

**Answer:**
"Subset and combination are related but used in different contexts:

**Subset:**
- Selection of elements from a set
- Order doesn't matter
- Can be any size (0 to n)
- Example: All subsets of [1,2,3] → [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]
- Count: 2^n

**Combination:**
- Selection of k elements from n elements
- Order doesn't matter
- Fixed size k
- Example: Combinations of 2 from [1,2,3] → [[1,2], [1,3], [2,3]]
- Count: C(n,k) = n! / (k! × (n-k)!)

**Relationship:**
- All combinations of size k are subsets of size k
- Subsets include all possible sizes

**Code difference:**
- Subset: Base case when `index == nums.length`
- Combination: Base case when `current.size() == k`

I use subsets when I need all possible selections, combinations when I need selections of specific size."

### Q30: How do you optimize recursive solutions?

**Answer:**
"I optimize recursive solutions using:

1. **Memoization**: Cache results to avoid recomputation
   ```java
   Map<String, Integer> memo = new HashMap<>();
   if (memo.containsKey(key)) return memo.get(key);
   ```

2. **Tail Recursion**: Make recursive call the last operation
   ```java
   return factorialTail(n - 1, acc * n);  // Tail recursive
   ```

3. **Pruning**: Stop early if path can't lead to solution
   ```java
   if (currentSum > target) return;  // Prune
   ```

4. **Convert to Iteration**: Eliminate recursion overhead
   ```java
   // Recursive → Iterative
   while (condition) { /* loop body */ }
   ```

5. **Bottom-up DP**: Instead of top-down recursion
   ```java
   // Memoization (top-down) → Tabulation (bottom-up)
   int[] dp = new int[n+1];
   for (int i = 0; i <= n; i++) { /* fill dp */ }
   ```

6. **Early Termination**: Return immediately when solution found
   ```java
   if (found) return true;  // Don't explore further
   ```

These optimizations can improve time from exponential to polynomial and reduce space overhead."

---

## Quick Reference Table

| Concept | Key Point | Time Complexity | Space Complexity |
|---------|-----------|----------------|------------------|
| **Factorial** | Base case: n≤1 | O(n) | O(n) |
| **Fibonacci (naive)** | Two recursive calls | O(2^n) | O(n) |
| **Fibonacci (memoized)** | Store computed values | O(n) | O(n) |
| **Power (optimized)** | Divide and conquer | O(log n) | O(log n) |
| **Tower of Hanoi** | Three recursive calls | O(2^n) | O(n) |
| **Tree Traversal** | Visit all nodes | O(n) | O(h) |
| **Backtracking** | Try and undo | O(b^d) | O(d) |
| **Memoization** | Cache results | Varies | O(n) |
| **Subsets** | Include/Exclude | O(2^n × n) | O(n) |
| **Permutations** | Try all choices | O(n! × n) | O(n) |
| **Combinations** | Include/Exclude | O(C(n,k) × k) | O(k) |
| **Word Search** | DFS on grid | O(m×n×4^L) | O(L) |
| **N-Queens** | Backtracking | O(n!) | O(n) |

---

## Memory Tips 🧠

### Recursion Components
- **B**ase case: **B**reak recursion
- **R**ecursive case: **R**educe input

### When to Use
- **T**rees → Recursion
- **G**raphs → Recursion
- **D**ivide and **C**onquer → Recursion
- **B**acktracking → Recursion

### Optimization
- **M**emoization → **M**emorize results
- **T**ail recursion → **T**ransform to iteration

---

## Daily Revision Checklist ✅

- [ ] Understand base case and recursive case
- [ ] Practice factorial and Fibonacci
- [ ] Master tree traversal (pre, in, post)
- [ ] Understand backtracking pattern
- [ ] Know when to use memoization
- [ ] Convert recursion to iteration
- [ ] Debug recursive functions
- [ ] Solve classic recursion problems
- [ ] Master subset problems (include/exclude pattern)
- [ ] Master permutation and combination problems
- [ ] Understand problem identification patterns
- [ ] Practice backtracking on 2D grids
- [ ] Know time/space complexity for each pattern

---

## Practice Problems

### Easy
1. Factorial
2. Fibonacci
3. Sum of digits
4. String reversal
5. Palindrome check

### Medium
1. Power (x^n)
2. Tower of Hanoi
3. Generate parentheses
4. Permutations
5. Combinations
6. Subsets
7. Subset sum
8. Combination sum
9. Letter combinations of phone number
10. Restore IP addresses

### Hard
1. N-Queens
2. Sudoku solver
3. Word search
4. Maximum path sum
5. Serialize/deserialize tree
6. Permutations with duplicates
7. Combination sum II
8. Subsets with duplicates
9. Word search II (multiple words)
10. Expression add operators

---

**Happy Coding! 🔄**

