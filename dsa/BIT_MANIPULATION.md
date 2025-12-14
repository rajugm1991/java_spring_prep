# Bit Manipulation - Complete Guide 🔢

> Master bitwise operators, tricks, and patterns for solving problems efficiently

---

## Table of Contents
1. [Basic Bit Operators](#basic-bit-operators)
2. [Essential Bit Tricks](#essential-bit-tricks)
3. [Common Bit Patterns](#common-bit-patterns)
4. [Problem-Solving Patterns](#problem-solving-patterns)
5. [Visual Examples](#visual-examples)
6. [Frequently Asked Questions](#frequently-asked-questions)

---

## Basic Bit Operators

### 1. AND (&)

**Operation:** Returns 1 only if both bits are 1

```
A & B = Result
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1
```

**Example:**
```
5 & 3 = 1

5 = 101
3 = 011
---
1 = 001
```

**Use Cases:**
- Check if bit is set: `num & (1 << i)`
- Clear bits: `num & ~mask`
- Extract specific bits

### 2. OR (|)

**Operation:** Returns 1 if at least one bit is 1

```
A | B = Result
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1
```

**Example:**
```
5 | 3 = 7

5 = 101
3 = 011
---
7 = 111
```

**Use Cases:**
- Set bits: `num | (1 << i)`
- Combine flags
- Merge bit patterns

### 3. XOR (^)

**Operation:** Returns 1 if bits are different

```
A ^ B = Result
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0
```

**Example:**
```
5 ^ 3 = 6

5 = 101
3 = 011
---
6 = 110
```

**Key Properties:**
- `A ^ A = 0`
- `A ^ 0 = A`
- `A ^ B ^ B = A` (XOR twice cancels out)
- Commutative and Associative

**Use Cases:**
- Find unique element
- Swap without temp: `a ^= b; b ^= a; a ^= b;`
- Toggle bits: `num ^ mask`

### 4. NOT (~)

**Operation:** Flips all bits (1's complement)

```
~A = Result
~0 = 1
~1 = 0
```

**Example:**
```
~5 = -6 (in 32-bit)

5  = 0000...0101
~5 = 1111...1010 = -6
```

**Use Cases:**
- Invert all bits
- Create masks: `~0` = all 1s
- Clear bits with AND: `num & ~mask`

### 5. Left Shift (<<)

**Operation:** Shifts bits left, fills with 0s

```
A << n = A * 2^n
```

**Example:**
```
5 << 2 = 20

5  = 101
20 = 10100 (shifted left by 2)
```

**Use Cases:**
- Multiply by powers of 2
- Set bit at position i: `1 << i`
- Create masks

### 6. Right Shift (>>)

**Operation:** Shifts bits right

**Signed Right Shift (>>):** Fills with sign bit (arithmetic shift)
**Unsigned Right Shift (>>>):** Fills with 0s (logical shift)

```
A >> n = A / 2^n (integer division)
```

**Example:**
```
20 >> 2 = 5

20 = 10100
5  = 101 (shifted right by 2)
```

**Use Cases:**
- Divide by powers of 2
- Extract bits
- Check bit at position i: `(num >> i) & 1`

---

## Essential Bit Tricks

### Trick 1: Check if Number is Power of 2

**Pattern:** `n & (n - 1) == 0`

```
Power of 2: Only one bit is set

8 = 1000
7 = 0111
8 & 7 = 0000 ✅

6 = 0110
5 = 0101
6 & 5 = 0100 ≠ 0 ❌
```

**Code:**
```java
boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

### Trick 2: Get Rightmost Set Bit

**Pattern:** `n & -n` or `n & (~n + 1)`

```
12 = 1100
-12 = 0100 (two's complement)
12 & -12 = 0100 = 4

The rightmost set bit is at position 2 (value 4)
```

**Code:**
```java
int getRightmostSetBit(int n) {
    return n & -n;
}

// Or get the position
int getRightmostSetBitPosition(int n) {
    return (int)(Math.log(n & -n) / Math.log(2));
}
```

### Trick 3: Clear Rightmost Set Bit

**Pattern:** `n & (n - 1)`

```
12 = 1100
11 = 1011
12 & 11 = 1000 = 8

Rightmost set bit cleared!
```

**Code:**
```java
int clearRightmostSetBit(int n) {
    return n & (n - 1);
}

// Count set bits using this trick
int countSetBits(int n) {
    int count = 0;
    while (n > 0) {
        n = n & (n - 1);  // Clear rightmost set bit
        count++;
    }
    return count;
}
```

### Trick 4: Set Bit at Position i

**Pattern:** `n | (1 << i)`

```
5 = 101
Set bit at position 1:
1 << 1 = 010
5 | 2 = 111 = 7
```

**Code:**
```java
int setBit(int n, int i) {
    return n | (1 << i);
}
```

### Trick 5: Clear Bit at Position i

**Pattern:** `n & ~(1 << i)`

```
7 = 111
Clear bit at position 1:
1 << 1 = 010
~(1 << 1) = 101
7 & 5 = 101 = 5
```

**Code:**
```java
int clearBit(int n, int i) {
    return n & ~(1 << i);
}
```

### Trick 6: Toggle Bit at Position i

**Pattern:** `n ^ (1 << i)`

```
5 = 101
Toggle bit at position 1:
1 << 1 = 010
5 ^ 2 = 111 = 7

Toggle again:
7 ^ 2 = 101 = 5
```

**Code:**
```java
int toggleBit(int n, int i) {
    return n ^ (1 << i);
}
```

### Trick 7: Check if Bit is Set

**Pattern:** `(n >> i) & 1` or `n & (1 << i)`

```
5 = 101
Check bit at position 0:
(5 >> 0) & 1 = 1 ✅

Check bit at position 1:
(5 >> 1) & 1 = 0 ❌
```

**Code:**
```java
boolean isBitSet(int n, int i) {
    return (n >> i) & 1 == 1;
    // Or: return (n & (1 << i)) != 0;
}
```

### Trick 8: Swap Two Numbers

**Pattern:** XOR swap (no temporary variable)

```
a = 5 (101)
b = 3 (011)

a = a ^ b  // a = 110 (6)
b = a ^ b  // b = 101 (5) - original a
a = a ^ b  // a = 011 (3) - original b
```

**Code:**
```java
void swap(int a, int b) {
    a = a ^ b;
    b = a ^ b;
    a = a ^ b;
}
```

### Trick 9: Get All Subsets

**Pattern:** Use bitmask to represent subsets

```
Array: [1, 2, 3]
n = 3, so 2^3 = 8 subsets

Bitmask  Binary  Subset
0        000     []
1        001     [1]
2        010     [2]
3        011     [1,2]
4        100     [3]
5        101     [1,3]
6        110     [2,3]
7        111     [1,2,3]
```

**Code:**
```java
List<List<Integer>> getAllSubsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    int n = nums.length;
    int total = 1 << n;  // 2^n
    
    for (int mask = 0; mask < total; mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        result.add(subset);
    }
    return result;
}
```

### Trick 10: Find Single Number (XOR Property)

**Pattern:** XOR all numbers, duplicates cancel out

```
Array: [2, 3, 2, 4, 4]
Find the single number:

2 ^ 3 ^ 2 ^ 4 ^ 4
= (2 ^ 2) ^ (4 ^ 4) ^ 3
= 0 ^ 0 ^ 3
= 3 ✅
```

**Code:**
```java
int findSingleNumber(int[] nums) {
    int result = 0;
    for (int num : nums) {
        result ^= num;
    }
    return result;
}
```

---

## Common Bit Patterns

### Pattern 1: Counting Set Bits (Brian Kernighan's Algorithm)

**Efficient way to count 1s in binary representation**

```
12 = 1100

Step 1: 12 & 11 = 1000 (8), count = 1
Step 2: 8 & 7 = 0000 (0), count = 2
Done! 2 set bits
```

**Code:**
```java
int countSetBits(int n) {
    int count = 0;
    while (n > 0) {
        n = n & (n - 1);  // Clear rightmost set bit
        count++;
    }
    return count;
}

// Built-in method
int countSetBitsBuiltIn(int n) {
    return Integer.bitCount(n);
}
```

### Pattern 2: Reverse Bits

**Reverse the binary representation**

```
Input:  0000...1010 (10)
Output: 0101...0000 (1342177280)
```

**Code:**
```java
int reverseBits(int n) {
    int result = 0;
    for (int i = 0; i < 32; i++) {
        result <<= 1;           // Shift result left
        result |= (n & 1);      // Add rightmost bit of n
        n >>= 1;                 // Shift n right
    }
    return result;
}
```

### Pattern 3: Find Missing Number

**Given array [0, n] with one missing, find it**

**Approach 1: XOR**
```
Array: [0, 1, 3, 4] (missing 2)
n = 4

XOR all numbers: 0 ^ 1 ^ 3 ^ 4 = 6
XOR with range: 6 ^ 0 ^ 1 ^ 2 ^ 3 ^ 4 = 2 ✅
```

**Code:**
```java
int findMissingNumber(int[] nums) {
    int n = nums.length;
    int xor = 0;
    
    // XOR all numbers in array
    for (int num : nums) {
        xor ^= num;
    }
    
    // XOR with range [0, n]
    for (int i = 0; i <= n; i++) {
        xor ^= i;
    }
    
    return xor;
}
```

**Approach 2: Sum**
```java
int findMissingNumberSum(int[] nums) {
    int n = nums.length;
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) {
        actualSum += num;
    }
    return expectedSum - actualSum;
}
```

### Pattern 4: Find Two Single Numbers

**Array has two unique numbers, rest appear twice**

```
Array: [1, 2, 1, 3, 2, 5]
Unique: 3 and 5

Step 1: XOR all → 3 ^ 5 = 6 (0110)
Step 2: Find rightmost set bit → 2 (0010)
Step 3: Group by this bit:
  Group 0 (bit not set): 1, 1, 2, 2, 3 → XOR = 3
  Group 1 (bit set): 5 → XOR = 5
```

**Code:**
```java
int[] findTwoSingleNumbers(int[] nums) {
    int xor = 0;
    for (int num : nums) {
        xor ^= num;
    }
    
    // Find rightmost set bit
    int rightmostSetBit = xor & -xor;
    
    int num1 = 0, num2 = 0;
    
    // Group by rightmost set bit
    for (int num : nums) {
        if ((num & rightmostSetBit) == 0) {
            num1 ^= num;
        } else {
            num2 ^= num;
        }
    }
    
    return new int[]{num1, num2};
}
```

### Pattern 5: Power Set Using Bitmask

**Generate all subsets using bit manipulation**

```
Array: [1, 2, 3]

For each bitmask from 0 to 7:
- Check which bits are set
- Add corresponding elements

Bitmask  Binary  Elements
0        000     []
1        001     [1]
2        010     [2]
3        011     [1,2]
4        100     [3]
5        101     [1,3]
6        110     [2,3]
7        111     [1,2,3]
```

**Code:**
```java
List<List<Integer>> powerSet(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    int n = nums.length;
    int total = 1 << n;  // 2^n subsets
    
    for (int mask = 0; mask < total; mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        result.add(subset);
    }
    return result;
}
```

### Pattern 6: Check if All Bits are Set

**Check if number has all bits set (n+1 is power of 2)**

```
7 = 111 (all bits set)
7 + 1 = 8 = 1000 (power of 2) ✅

5 = 101 (not all bits set)
5 + 1 = 6 = 110 (not power of 2) ❌
```

**Code:**
```java
boolean allBitsSet(int n) {
    return (n & (n + 1)) == 0;
    // Or: return (n + 1) is power of 2
}
```

### Pattern 7: Flip Bits to Make All Same

**Minimum flips to make all bits same**

```
10110 → 11111 (flip 2 bits) or 00000 (flip 3 bits)
Minimum: 2 flips
```

**Code:**
```java
int minFlipsToMakeSame(int n) {
    // Count set bits and unset bits
    int setBits = countSetBits(n);
    int totalBits = 32;  // or count bits in n
    int unsetBits = totalBits - setBits;
    
    return Math.min(setBits, unsetBits);
}
```

### Pattern 8: Maximum XOR of Two Numbers

**Find maximum XOR value from array**

```
Array: [3, 10, 5, 25, 2, 8]

3 ^ 10 = 9
3 ^ 5 = 6
...
25 ^ 5 = 28 (maximum)
```

**Code (Brute Force):**
```java
int findMaximumXOR(int[] nums) {
    int max = 0;
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            max = Math.max(max, nums[i] ^ nums[j]);
        }
    }
    return max;
}
```

**Code (Trie - Optimized):**
```java
class TrieNode {
    TrieNode[] children = new TrieNode[2];
}

int findMaximumXORTrie(int[] nums) {
    TrieNode root = new TrieNode();
    
    // Build trie
    for (int num : nums) {
        TrieNode node = root;
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (node.children[bit] == null) {
                node.children[bit] = new TrieNode();
            }
            node = node.children[bit];
        }
    }
    
    // Find maximum XOR
    int max = 0;
    for (int num : nums) {
        TrieNode node = root;
        int currentMax = 0;
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            int flipBit = 1 - bit;  // Try opposite bit
            
            if (node.children[flipBit] != null) {
                currentMax |= (1 << i);
                node = node.children[flipBit];
            } else {
                node = node.children[bit];
            }
        }
        max = Math.max(max, currentMax);
    }
    return max;
}
```

---

## Problem-Solving Patterns

### Pattern 1: Single Number Problems

#### Problem: Find Single Number
**Array has one unique number, rest appear twice**

**Solution:** XOR all numbers
```java
int singleNumber(int[] nums) {
    int result = 0;
    for (int num : nums) {
        result ^= num;
    }
    return result;
}
```

#### Problem: Find Single Number II
**Array has one unique number, rest appear three times**

**Solution:** Count bits modulo 3
```java
int singleNumberII(int[] nums) {
    int result = 0;
    for (int i = 0; i < 32; i++) {
        int sum = 0;
        for (int num : nums) {
            if ((num >> i) & 1 == 1) {
                sum++;
            }
        }
        if (sum % 3 != 0) {
            result |= (1 << i);
        }
    }
    return result;
}
```

### Pattern 2: Bitmask DP

**Use bitmask to represent state in dynamic programming**

#### Problem: Traveling Salesman Problem (TSP)
**Visit all cities exactly once, return to start**

```java
int tsp(int[][] graph) {
    int n = graph.length;
    int[][] dp = new int[1 << n][n];
    
    // Initialize: visit single city
    for (int i = 0; i < n; i++) {
        dp[1 << i][i] = graph[0][i];  // Start from city 0
    }
    
    // Fill DP table
    for (int mask = 1; mask < (1 << n); mask++) {
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) == 0) continue;
            
            for (int j = 0; j < n; j++) {
                if ((mask & (1 << j)) != 0) continue;
                
                int newMask = mask | (1 << j);
                dp[newMask][j] = Math.min(
                    dp[newMask][j],
                    dp[mask][i] + graph[i][j]
                );
            }
        }
    }
    
    // Return minimum cost to visit all and return
    return dp[(1 << n) - 1][0];
}
```

### Pattern 3: Subset Generation

**Generate all subsets efficiently using bitmask**

```java
List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    int n = nums.length;
    
    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        result.add(subset);
    }
    return result;
}
```

### Pattern 4: Gray Code

**Generate Gray code sequence**

```
n = 2:
00 → 01 → 11 → 10

n = 3:
000 → 001 → 011 → 010 → 110 → 111 → 101 → 100
```

**Code:**
```java
List<Integer> grayCode(int n) {
    List<Integer> result = new ArrayList<>();
    int total = 1 << n;
    
    for (int i = 0; i < total; i++) {
        result.add(i ^ (i >> 1));  // Gray code formula
    }
    
    return result;
}
```

### Pattern 5: Bit Manipulation in Arrays

#### Problem: Find Duplicate and Missing
**Array [1, n] has one duplicate and one missing**

```java
int[] findErrorNums(int[] nums) {
    int xor = 0;
    int n = nums.length;
    
    // XOR all numbers and range
    for (int i = 0; i < n; i++) {
        xor ^= nums[i];
        xor ^= (i + 1);
    }
    
    // Find rightmost set bit
    int rightmost = xor & -xor;
    
    int num1 = 0, num2 = 0;
    
    // Group by rightmost bit
    for (int i = 0; i < n; i++) {
        if ((nums[i] & rightmost) != 0) {
            num1 ^= nums[i];
        } else {
            num2 ^= nums[i];
        }
        if (((i + 1) & rightmost) != 0) {
            num1 ^= (i + 1);
        } else {
            num2 ^= (i + 1);
        }
    }
    
    // Determine which is duplicate and which is missing
    for (int num : nums) {
        if (num == num1) {
            return new int[]{num1, num2};
        }
    }
    return new int[]{num2, num1};
}
```

---

## Visual Examples

### Example 1: XOR Swap Visualization

```
Initial:
a = 5 (101)
b = 3 (011)

Step 1: a = a ^ b
a = 101 ^ 011 = 110 (6)
b = 011 (3)

Step 2: b = a ^ b
a = 110 (6)
b = 110 ^ 011 = 101 (5) ← original a

Step 3: a = a ^ b
a = 110 ^ 101 = 011 (3) ← original b
b = 101 (5)

Final:
a = 3 (011)
b = 5 (101)
✅ Swapped!
```

### Example 2: Counting Set Bits

```
Number: 12 (1100)

Step 1: 12 & 11
  1100 (12)
& 1011 (11)
------
  1000 (8)  count = 1

Step 2: 8 & 7
  1000 (8)
& 0111 (7)
------
  0000 (0)  count = 2

Done! 12 has 2 set bits
```

### Example 3: Power Set Generation

```
Array: [1, 2, 3]
n = 3, total = 8 subsets

Mask  Binary  Check Bits        Subset
0     000     None              []
1     001     Bit 0 set         [1]
2     010     Bit 1 set         [2]
3     011     Bits 0,1 set      [1,2]
4     100     Bit 2 set         [3]
5     101     Bits 0,2 set      [1,3]
6     110     Bits 1,2 set      [2,3]
7     111     All bits set       [1,2,3]
```

### Example 4: Finding Single Number

```
Array: [4, 1, 2, 1, 2]

XOR all:
4 ^ 1 ^ 2 ^ 1 ^ 2

Group by pairs:
(1 ^ 1) ^ (2 ^ 2) ^ 4
= 0 ^ 0 ^ 4
= 4 ✅

Single number is 4
```

### Example 5: Check Power of 2

```
Check if 8 is power of 2:

8 = 1000
7 = 0111
8 & 7 = 0000 = 0 ✅

Check if 6 is power of 2:

6 = 0110
5 = 0101
6 & 5 = 0100 ≠ 0 ❌
```

---

## Frequently Asked Questions

### Q1: How to check if a number is even or odd using bits?

**Answer:**
```java
boolean isEven(int n) {
    return (n & 1) == 0;
}

// n & 1 checks the rightmost bit
// If 0 → even, if 1 → odd
```

**Example:**
```
5 & 1 = 1 → odd
6 & 1 = 0 → even
```

### Q2: How to multiply/divide by powers of 2?

**Answer:**
```java
// Multiply by 2^n
int multiply(int n, int power) {
    return n << power;
}

// Divide by 2^n
int divide(int n, int power) {
    return n >> power;
}

// Examples:
5 << 2 = 20  (5 * 4)
20 >> 2 = 5  (20 / 4)
```

### Q3: How to find the position of the rightmost set bit?

**Answer:**
```java
int getRightmostSetBitPosition(int n) {
    if (n == 0) return -1;
    
    // Method 1: Using log
    return (int)(Math.log(n & -n) / Math.log(2));
    
    // Method 2: Count trailing zeros
    return Integer.numberOfTrailingZeros(n);
}
```

### Q4: How to set/clear/toggle the nth bit?

**Answer:**
```java
// Set bit
int setBit(int num, int n) {
    return num | (1 << n);
}

// Clear bit
int clearBit(int num, int n) {
    return num & ~(1 << n);
}

// Toggle bit
int toggleBit(int num, int n) {
    return num ^ (1 << n);
}
```

### Q5: How to count the number of set bits efficiently?

**Answer:**
```java
// Method 1: Brian Kernighan's Algorithm (O(k) where k is number of set bits)
int countSetBits(int n) {
    int count = 0;
    while (n > 0) {
        n = n & (n - 1);
        count++;
    }
    return count;
}

// Method 2: Built-in (optimized)
int countSetBitsBuiltIn(int n) {
    return Integer.bitCount(n);
}
```

### Q6: How to reverse bits of a number?

**Answer:**
```java
int reverseBits(int n) {
    int result = 0;
    for (int i = 0; i < 32; i++) {
        result <<= 1;
        result |= (n & 1);
        n >>= 1;
    }
    return result;
}
```

### Q7: How to find if two numbers have opposite signs?

**Answer:**
```java
boolean oppositeSigns(int a, int b) {
    return (a ^ b) < 0;
}

// XOR of opposite signs gives negative number
// Example: 5 ^ -3 = negative
```

### Q8: How to swap two numbers without temporary variable?

**Answer:**
```java
void swap(int a, int b) {
    a = a ^ b;
    b = a ^ b;  // b = original a
    a = a ^ b;  // a = original b
}
```

### Q9: How to find the maximum of two numbers without comparison?

**Answer:**
```java
int max(int a, int b) {
    // If a - b is negative, b is larger
    int diff = a - b;
    int sign = (diff >> 31) & 1;  // Sign bit
    return a - sign * diff;
}

// Or using absolute value
int max(int a, int b) {
    return (a + b + Math.abs(a - b)) / 2;
}
```

### Q10: How to check if a number is a power of 4?

**Answer:**
```java
boolean isPowerOfFour(int n) {
    // Must be power of 2
    if (n <= 0 || (n & (n - 1)) != 0) {
        return false;
    }
    
    // Set bit must be at even position (0, 2, 4, ...)
    // Check: n & 0x55555555 != 0
    return (n & 0x55555555) != 0;
}
```

### Q11: How to find the complement of a number?

**Answer:**
```java
int findComplement(int num) {
    // Find number of bits
    int bits = (int)(Math.log(num) / Math.log(2)) + 1;
    
    // Create mask with all bits set
    int mask = (1 << bits) - 1;
    
    // XOR with mask to flip bits
    return num ^ mask;
}
```

### Q12: How to add two numbers using bit manipulation?

**Answer:**
```java
int add(int a, int b) {
    while (b != 0) {
        int carry = a & b;      // Find carry bits
        a = a ^ b;              // Sum without carry
        b = carry << 1;         // Shift carry left
    }
    return a;
}
```

### Q13: How to find the missing number in array [0, n]?

**Answer:**
```java
int missingNumber(int[] nums) {
    int n = nums.length;
    int xor = 0;
    
    // XOR all numbers in array
    for (int num : nums) {
        xor ^= num;
    }
    
    // XOR with range [0, n]
    for (int i = 0; i <= n; i++) {
        xor ^= i;
    }
    
    return xor;
}
```

### Q14: How to check if bits are alternating?

**Answer:**
```java
boolean hasAlternatingBits(int n) {
    // XOR with right-shifted version
    // If alternating, result should be all 1s
    int x = n ^ (n >> 1);
    return (x & (x + 1)) == 0;
}
```

### Q15: How to find the number of 1-bit differences (Hamming Distance)?

**Answer:**
```java
int hammingDistance(int x, int y) {
    int xor = x ^ y;
    return Integer.bitCount(xor);
    // Or count manually
    // int count = 0;
    // while (xor > 0) {
    //     xor = xor & (xor - 1);
    //     count++;
    // }
    // return count;
}
```

---

## Quick Reference Table

| Operation | Code | Time Complexity |
|-----------|------|----------------|
| **Check if power of 2** | `n > 0 && (n & (n-1)) == 0` | O(1) |
| **Get rightmost set bit** | `n & -n` | O(1) |
| **Clear rightmost set bit** | `n & (n-1)` | O(1) |
| **Set bit at position i** | `n \| (1 << i)` | O(1) |
| **Clear bit at position i** | `n & ~(1 << i)` | O(1) |
| **Toggle bit at position i** | `n ^ (1 << i)` | O(1) |
| **Check if bit is set** | `(n >> i) & 1` | O(1) |
| **Count set bits** | `Integer.bitCount(n)` | O(1) |
| **Reverse bits** | Loop with shift | O(32) |
| **Find single number** | XOR all | O(n) |
| **Power set** | Bitmask 0 to 2^n | O(2^n * n) |

---

## Memory Tips 🧠

### Bit Operators
- **& (AND)**: Both must be 1 → "All true"
- **\| (OR)**: At least one 1 → "Any true"
- **^ (XOR)**: Different → "Exclusive or"
- **~ (NOT)**: Flip all bits
- **<< (Left Shift)**: Multiply by 2^n
- **>> (Right Shift)**: Divide by 2^n

### Common Tricks
- **Power of 2**: `n & (n-1) == 0`
- **Rightmost set bit**: `n & -n`
- **Clear rightmost**: `n & (n-1)`
- **Swap**: `a^=b; b^=a; a^=b;`
- **Single number**: XOR all

### Patterns
- **Subsets**: Use bitmask 0 to 2^n
- **DP with state**: Use bitmask to represent visited/selected
- **XOR properties**: A^A=0, A^0=A, A^B^B=A

---

## Daily Revision Checklist ✅

- [ ] Basic bit operators: &, |, ^, ~, <<, >>
- [ ] Essential tricks: Power of 2, rightmost bit, clear bit
- [ ] Common patterns: Single number, subsets, bitmask DP
- [ ] Problem-solving: XOR properties, counting bits, reversing bits
- [ ] Practice: LeetCode bit manipulation problems

---

**Happy Coding! 🔢**

