# HashMap & HashTable - Complete Guide 🗺️

> Master hash-based data structures with patterns, tricks, and problem-solving strategies

---

## Table of Contents
1. [What is HashMap/HashTable?](#what-is-hashmaphashtable)
2. [Key Differences](#key-differences)
3. [How Hashing Works](#how-hashing-works)
4. [Common Operations](#common-operations)
5. [Essential Patterns](#essential-patterns)
6. [Problem-Solving Patterns](#problem-solving-patterns)
7. [Tricks and Optimizations](#tricks-and-optimizations)
8. [Frequently Asked Questions](#frequently-asked-questions)

---

## What is HashMap/HashTable?

**HashMap/HashTable** is a data structure that stores key-value pairs with **O(1) average time complexity** for operations.

### Key Concepts:

- **Key**: Unique identifier
- **Value**: Data associated with key
- **Hash Function**: Converts key to index
- **Collision**: When two keys map to same index

### Visual Representation:

```
HashMap Structure:

Keys → Hash Function → Index → Bucket
                              ↓
                          [Key, Value]

Example:
"apple" → hash("apple") → 3 → [("apple", 5)]
"banana" → hash("banana") → 7 → [("banana", 3)]
"cherry" → hash("cherry") → 3 → [("cherry", 8)]  (Collision!)
```

---

## Key Differences

| Feature | HashMap | HashTable |
|---------|---------|-----------|
| **Synchronization** | Not synchronized | Synchronized (thread-safe) |
| **Null Keys** | Allows one null key | No null keys |
| **Null Values** | Allows null values | No null values |
| **Performance** | Faster (no locking) | Slower (synchronized) |
| **Iteration** | Fail-fast iterator | Enumerator |
| **Use Case** | Single-threaded | Multi-threaded |

**In Interviews:** Use **HashMap** (unless specifically asked for thread-safety)

---

## How Hashing Works

### Hash Function Process:

```
Step 1: Key → Hash Code
"apple" → hashCode("apple") → 123456789

Step 2: Hash Code → Index
123456789 % bucketSize → 3

Step 3: Store at Index
Bucket[3] = [("apple", 5)]
```

### Collision Handling:

**1. Chaining (Linked List)**
```
Index 3: [("apple", 5)] → [("cherry", 8)] → null
```

**2. Open Addressing (Linear Probing)**
```
Index 3: ("apple", 5)
Index 4: ("cherry", 8)  (moved to next available)
```

---

## Common Operations

### Basic Operations:

```java
// Create HashMap
Map<String, Integer> map = new HashMap<>();

// Put (Insert/Update)
map.put("apple", 5);        // O(1) average
map.put("banana", 3);
map.put("apple", 7);        // Updates existing

// Get
int count = map.get("apple");  // O(1) average, returns 7
int count = map.getOrDefault("orange", 0);  // Returns 0 if not found

// Contains
boolean exists = map.containsKey("apple");  // O(1) average
boolean hasValue = map.containsValue(7);    // O(n) - slow!

// Remove
map.remove("apple");       // O(1) average

// Size
int size = map.size();     // O(1)

// Clear
map.clear();               // O(n)
```

### Iteration Patterns:

```java
// 1. Iterate over keys
for (String key : map.keySet()) {
    System.out.println(key + ": " + map.get(key));
}

// 2. Iterate over values
for (Integer value : map.values()) {
    System.out.println(value);
}

// 3. Iterate over entries (BEST - most efficient)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// 4. Java 8+ Streams
map.forEach((key, value) -> 
    System.out.println(key + ": " + value)
);
```

---

## Essential Patterns

### Pattern 1: Frequency Counting

**When to Use:** Count occurrences of elements

**Example:** Count character frequency

```java
String s = "hello";
Map<Character, Integer> freq = new HashMap<>();

for (char c : s.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}

// Result: {h=1, e=1, l=2, o=1}
```

**Visual:**
```
"hello"
  ↓
h → freq.getOrDefault('h', 0) + 1 = 0 + 1 = 1
e → freq.getOrDefault('e', 0) + 1 = 0 + 1 = 1
l → freq.getOrDefault('l', 0) + 1 = 0 + 1 = 1
l → freq.getOrDefault('l', 0) + 1 = 1 + 1 = 2
o → freq.getOrDefault('o', 0) + 1 = 0 + 1 = 1

HashMap: {h=1, e=1, l=2, o=1}
```

**One-liner with Java 8:**
```java
Map<Character, Long> freq = s.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

---

### Pattern 2: Two Sum / Complement Pattern

**When to Use:** Find pairs that sum to target

**Problem:** Find two numbers that add up to target

```java
int[] nums = {2, 7, 11, 15};
int target = 9;

Map<Integer, Integer> map = new HashMap<>();  // value -> index

for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    
    if (map.containsKey(complement)) {
        return new int[]{map.get(complement), i};
    }
    
    map.put(nums[i], i);
}
```

**Visual:**
```
nums = [2, 7, 11, 15], target = 9

i=0: nums[0]=2
     complement = 9 - 2 = 7
     map contains 7? No
     map.put(2, 0) → {2: 0}

i=1: nums[1]=7
     complement = 9 - 7 = 2
     map contains 2? Yes! → Found pair [0, 1]
     Return [0, 1]
```

**Key Insight:** Store `complement` or `value` as key, index as value

---

### Pattern 3: Grouping Elements

**When to Use:** Group elements by some property

**Example:** Group strings by length

```java
String[] words = {"cat", "dog", "bat", "rat", "car", "bar"};

Map<Integer, List<String>> groups = new HashMap<>();

for (String word : words) {
    int length = word.length();
    groups.computeIfAbsent(length, k -> new ArrayList<>()).add(word);
}

// Result: {3=[cat, dog, bat, rat, car, bar]}
```

**Visual:**
```
words = ["cat", "dog", "bat", "rat", "car", "bar"]

"cat" → length=3 → groups.computeIfAbsent(3, ...) → [cat]
"dog" → length=3 → groups.get(3).add("dog") → [cat, dog]
"bat" → length=3 → groups.get(3).add("bat") → [cat, dog, bat]
...

HashMap: {3=[cat, dog, bat, rat, car, bar]}
```

**Java 8 Streams:**
```java
Map<Integer, List<String>> groups = Arrays.stream(words)
    .collect(Collectors.groupingBy(String::length));
```

---

### Pattern 4: Index Mapping

**When to Use:** Store element → index mapping

**Example:** Find indices of all occurrences

```java
int[] nums = {1, 2, 3, 2, 4, 2};
Map<Integer, List<Integer>> indices = new HashMap<>();

for (int i = 0; i < nums.length; i++) {
    indices.computeIfAbsent(nums[i], k -> new ArrayList<>()).add(i);
}

// Result: {1=[0], 2=[1, 3, 5], 3=[2], 4=[4]}
```

**Visual:**
```
nums = [1, 2, 3, 2, 4, 2]

i=0: nums[0]=1 → {1: [0]}
i=1: nums[1]=2 → {1: [0], 2: [1]}
i=2: nums[2]=3 → {1: [0], 2: [1], 3: [2]}
i=3: nums[3]=2 → {1: [0], 2: [1, 3], 3: [2]}
i=4: nums[4]=4 → {1: [0], 2: [1, 3], 3: [2], 4: [4]}
i=5: nums[5]=2 → {1: [0], 2: [1, 3, 5], 3: [2], 4: [4]}
```

---

### Pattern 5: Sliding Window with HashMap

**When to Use:** Find subarray/substring with specific property

**Example:** Longest substring with at most K distinct characters

```java
String s = "eceba";
int k = 2;

Map<Character, Integer> window = new HashMap<>();
int left = 0, maxLen = 0;

for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);
    window.put(c, window.getOrDefault(c, 0) + 1);
    
    // Shrink window if distinct chars > k
    while (window.size() > k) {
        char leftChar = s.charAt(left);
        window.put(leftChar, window.get(leftChar) - 1);
        if (window.get(leftChar) == 0) {
            window.remove(leftChar);
        }
        left++;
    }
    
    maxLen = Math.max(maxLen, right - left + 1);
}
```

**Visual:**
```
s = "eceba", k = 2

right=0: window={e:1}, size=1 ≤ 2, maxLen=1
right=1: window={e:1, c:1}, size=2 ≤ 2, maxLen=2
right=2: window={e:2, c:1}, size=2 ≤ 2, maxLen=3
right=3: window={e:2, c:1, b:1}, size=3 > 2
         Shrink: remove 'c', window={e:2, b:1}, left=2
         maxLen=3
right=4: window={e:2, b:1, a:1}, size=3 > 2
         Shrink: remove 'e', window={b:1, a:1}, left=3
         maxLen=3
```

---

## Problem-Solving Patterns

### Pattern 6: Anagram Detection

**Problem:** Check if two strings are anagrams

**Approach 1: Frequency Map**

```java
boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    
    Map<Character, Integer> freq = new HashMap<>();
    
    // Count characters in s
    for (char c : s.toCharArray()) {
        freq.put(c, freq.getOrDefault(c, 0) + 1);
    }
    
    // Decrement for characters in t
    for (char c : t.toCharArray()) {
        if (!freq.containsKey(c) || freq.get(c) == 0) {
            return false;
        }
        freq.put(c, freq.get(c) - 1);
    }
    
    return true;
}
```

**Visual:**
```
s = "listen", t = "silent"

Step 1: Count s
{l=1, i=1, s=1, t=1, e=1, n=1}

Step 2: Decrement for t
s → l=1, i=1, s=0, t=1, e=1, n=1
i → l=1, i=0, s=0, t=1, e=1, n=1
l → l=0, i=0, s=0, t=1, e=1, n=1
e → l=0, i=0, s=0, t=1, e=0, n=1
n → l=0, i=0, s=0, t=1, e=0, n=0
t → l=0, i=0, s=0, t=0, e=0, n=0

All counts = 0 → Anagram ✅
```

**Approach 2: Sort and Compare (Alternative)**
```java
char[] sArr = s.toCharArray();
char[] tArr = t.toCharArray();
Arrays.sort(sArr);
Arrays.sort(tArr);
return Arrays.equals(sArr, tArr);
```

---

### Pattern 7: First Unique Character

**Problem:** Find first non-repeating character

```java
String s = "leetcode";

// Step 1: Count frequency
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}

// Step 2: Find first with count = 1
for (int i = 0; i < s.length(); i++) {
    if (freq.get(s.charAt(i)) == 1) {
        return i;  // Return index of 'l'
    }
}
```

**Visual:**
```
s = "leetcode"

Step 1: Frequency
{l=1, e=3, t=1, c=1, o=1, d=1}

Step 2: Check in order
i=0: 'l' → count=1 ✅ Return 0
```

---

### Pattern 8: Subarray Sum Equals K

**Problem:** Count subarrays with sum = k

**Key Insight:** Use prefix sum with HashMap

```java
int[] nums = {1, 1, 1};
int k = 2;

Map<Integer, Integer> prefixSum = new HashMap<>();
prefixSum.put(0, 1);  // Important: sum 0 appears once
int sum = 0, count = 0;

for (int num : nums) {
    sum += num;
    
    // If (sum - k) exists, we found a subarray
    if (prefixSum.containsKey(sum - k)) {
        count += prefixSum.get(sum - k);
    }
    
    prefixSum.put(sum, prefixSum.getOrDefault(sum, 0) + 1);
}

return count;  // 2
```

**Visual:**
```
nums = [1, 1, 1], k = 2

Initial: prefixSum = {0: 1}, sum = 0, count = 0

i=0: num=1
     sum = 0 + 1 = 1
     sum - k = 1 - 2 = -1 (not in map)
     prefixSum = {0: 1, 1: 1}

i=1: num=1
     sum = 1 + 1 = 2
     sum - k = 2 - 2 = 0 (in map, count=1)
     count = 0 + 1 = 1
     prefixSum = {0: 1, 1: 1, 2: 1}

i=2: num=1
     sum = 2 + 1 = 3
     sum - k = 3 - 2 = 1 (in map, count=1)
     count = 1 + 1 = 2
     prefixSum = {0: 1, 1: 1, 2: 1, 3: 1}

Result: 2 subarrays
[1,1] (indices 0-1) and [1,1] (indices 1-2)
```

**Why prefixSum.put(0, 1)?**
- Handles case when subarray starts from index 0
- If sum = k, then sum - k = 0 should exist

---

### Pattern 9: Longest Consecutive Sequence

**Problem:** Find longest consecutive sequence length

**Approach:** Use HashSet to check existence, then expand

```java
int[] nums = {100, 4, 200, 1, 3, 2};

Set<Integer> set = new HashSet<>();
for (int num : nums) {
    set.add(num);
}

int maxLen = 0;

for (int num : set) {
    // Only start from beginning of sequence
    if (!set.contains(num - 1)) {
        int currentNum = num;
        int currentLen = 1;
        
        // Expand sequence
        while (set.contains(currentNum + 1)) {
            currentNum++;
            currentLen++;
        }
        
        maxLen = Math.max(maxLen, currentLen);
    }
}

return maxLen;  // 4 (sequence: 1,2,3,4)
```

**Visual:**
```
nums = [100, 4, 200, 1, 3, 2]
set = {100, 4, 200, 1, 3, 2}

Check 100: 99 not in set → start sequence
          101 not in set → len=1

Check 4: 3 in set → skip (not start)

Check 200: 199 not in set → start sequence
           201 not in set → len=1

Check 1: 0 not in set → start sequence
         2 in set → len=2
         3 in set → len=3
         4 in set → len=4
         5 not in set → stop
         maxLen = 4 ✅
```

**Key Insight:** Only start from beginning of sequence (num-1 not in set)

---

### Pattern 10: Top K Frequent Elements

**Problem:** Find K most frequent elements

**Approach:** Frequency map + Priority Queue

```java
int[] nums = {1, 1, 1, 2, 2, 3};
int k = 2;

// Step 1: Count frequency
Map<Integer, Integer> freq = new HashMap<>();
for (int num : nums) {
    freq.put(num, freq.getOrDefault(num, 0) + 1);
}

// Step 2: Min heap of size k
PriorityQueue<Map.Entry<Integer, Integer>> pq = new PriorityQueue<>(
    (a, b) -> a.getValue() - b.getValue()
);

for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
    pq.offer(entry);
    if (pq.size() > k) {
        pq.poll();  // Remove least frequent
    }
}

// Step 3: Extract results
int[] result = new int[k];
for (int i = k - 1; i >= 0; i--) {
    result[i] = pq.poll().getKey();
}

return result;  // [1, 2]
```

**Visual:**
```
nums = [1, 1, 1, 2, 2, 3], k = 2

Step 1: Frequency
{1: 3, 2: 2, 3: 1}

Step 2: Min Heap (size k=2)
Add (3,1): [(3,1)]
Add (2,2): [(3,1), (2,2)]
Add (1,3): [(2,2), (1,3)] → remove (3,1)

Step 3: Extract
[1, 2]
```

**Alternative: Bucket Sort (O(n))**
```java
// Group by frequency
Map<Integer, List<Integer>> buckets = new HashMap<>();
for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
    int frequency = entry.getValue();
    buckets.computeIfAbsent(frequency, k -> new ArrayList<>())
           .add(entry.getKey());
}

// Extract top k from highest frequency
List<Integer> result = new ArrayList<>();
for (int freq = nums.length; freq >= 1 && result.size() < k; freq--) {
    if (buckets.containsKey(freq)) {
        result.addAll(buckets.get(freq));
    }
}
```

---

### Pattern 11: Word Pattern

**Problem:** Check if string follows pattern

```java
String pattern = "abba";
String s = "dog cat cat dog";

String[] words = s.split(" ");
if (pattern.length() != words.length) return false;

Map<Character, String> charToWord = new HashMap<>();
Map<String, Character> wordToChar = new HashMap<>();

for (int i = 0; i < pattern.length(); i++) {
    char c = pattern.charAt(i);
    String word = words[i];
    
    // Check both mappings
    if (charToWord.containsKey(c) && !charToWord.get(c).equals(word)) {
        return false;
    }
    if (wordToChar.containsKey(word) && wordToChar.get(word) != c) {
        return false;
    }
    
    charToWord.put(c, word);
    wordToChar.put(word, c);
}

return true;
```

**Visual:**
```
pattern = "abba", s = "dog cat cat dog"

i=0: c='a', word="dog"
     charToWord = {a: dog}
     wordToChar = {dog: a}

i=1: c='b', word="cat"
     charToWord = {a: dog, b: cat}
     wordToChar = {dog: a, cat: b}

i=2: c='b', word="cat"
     charToWord.get('b') = "cat" == "cat" ✅
     wordToChar.get("cat") = 'b' == 'b' ✅

i=3: c='a', word="dog"
     charToWord.get('a') = "dog" == "dog" ✅
     wordToChar.get("dog") = 'a' == 'a' ✅

Result: true ✅
```

**Key Insight:** Need bidirectional mapping (char↔word)

---

## Tricks and Optimizations

### Trick 1: getOrDefault() Pattern

**Instead of:**
```java
if (map.containsKey(key)) {
    map.put(key, map.get(key) + 1);
} else {
    map.put(key, 1);
}
```

**Use:**
```java
map.put(key, map.getOrDefault(key, 0) + 1);
```

---

### Trick 2: computeIfAbsent() Pattern

**Instead of:**
```java
if (!map.containsKey(key)) {
    map.put(key, new ArrayList<>());
}
map.get(key).add(value);
```

**Use:**
```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

---

### Trick 3: compute() Pattern

**Update value based on existing value:**
```java
// Increment or set to 1
map.compute(key, (k, v) -> v == null ? 1 : v + 1);

// Multiply by 2
map.compute(key, (k, v) -> v == null ? 1 : v * 2);
```

---

### Trick 4: merge() Pattern

**Merge values:**
```java
// Sum values
map.merge(key, value, Integer::sum);

// Concatenate strings
map.merge(key, value, (old, newVal) -> old + ", " + newVal);
```

---

### Trick 5: Initialize with Default Capacity

**If you know approximate size:**
```java
// Avoid resizing
Map<String, Integer> map = new HashMap<>(1000);
```

---

### Trick 6: Use Entry Set for Iteration

**More efficient:**
```java
// ✅ Good - O(n)
for (Map.Entry<K, V> entry : map.entrySet()) {
    K key = entry.getKey();
    V value = entry.getValue();
}

// ❌ Bad - O(n) but slower (two lookups)
for (K key : map.keySet()) {
    V value = map.get(key);  // Extra lookup
}
```

---

### Trick 7: Check Before get() to Avoid Null

**Option 1: getOrDefault()**
```java
int value = map.getOrDefault(key, 0);
```

**Option 2: containsKey()**
```java
if (map.containsKey(key)) {
    int value = map.get(key);
}
```

**Option 3: computeIfAbsent()**
```java
int value = map.computeIfAbsent(key, k -> 0);
```

---

### Trick 8: Custom Objects as Keys

**Must override equals() and hashCode():**
```java
class Person {
    String name;
    int age;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

**Rule:** If two objects are equal, they must have same hashCode

---

### Trick 9: LinkedHashMap for Order

**Maintains insertion order:**
```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("first", 1);
map.put("second", 2);
map.put("third", 3);

// Iteration order: first → second → third
```

**LRU Cache Pattern:**
```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);  // accessOrder = true
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
```

---

### Trick 10: TreeMap for Sorted Keys

**Keys automatically sorted:**
```java
Map<String, Integer> map = new TreeMap<>();
map.put("zebra", 1);
map.put("apple", 2);
map.put("banana", 3);

// Iteration order: apple → banana → zebra
```

**Custom Comparator:**
```java
Map<String, Integer> map = new TreeMap<>((a, b) -> b.compareTo(a));
// Descending order
```

---

## Frequently Asked Questions

### Q1: What is the time complexity of HashMap operations?

**Answer:**
- **Average Case:** O(1) for get, put, remove, containsKey
- **Worst Case:** O(n) when all keys hash to same bucket (rare)
- **containsValue():** O(n) - must check all values

**Why O(1) average?**
- Hash function distributes keys evenly
- Collisions handled efficiently (chaining or probing)
- Load factor kept low (typically 0.75)

---

### Q2: How does HashMap handle collisions?

**Answer:**
Two main strategies:

**1. Chaining (Java HashMap uses this):**
```
Index 3: [("apple", 5)] → [("cherry", 8)] → null
```
- Each bucket is a linked list
- Colliding keys stored in same bucket
- O(1) average, O(n) worst case

**2. Open Addressing:**
```
Index 3: ("apple", 5)
Index 4: ("cherry", 8)  // moved to next available
```
- Store in next available slot
- Linear probing, quadratic probing, or double hashing

---

### Q3: What is load factor and why is it 0.75?

**Answer:**
**Load Factor = Number of elements / Number of buckets**

**Default: 0.75 (75% full)**

**Why 0.75?**
- **Too low (0.5):** More memory, fewer collisions, but frequent resizing
- **Too high (0.9):** Less memory, but more collisions
- **0.75:** Good balance between memory and performance

**When load factor exceeded:**
- HashMap resizes (doubles capacity)
- Rehashes all elements
- O(n) operation, but amortized O(1)

---

### Q4: When should I use HashMap vs HashSet?

**Answer:**

**Use HashMap when:**
- Need key-value pairs
- Need to store/retrieve values by key
- Need frequency counting
- Need index mapping

**Use HashSet when:**
- Only need to check existence
- Need unique elements
- Don't need associated values

**Example:**
```java
// HashMap: Count frequency
Map<Character, Integer> freq = new HashMap<>();

// HashSet: Check existence
Set<Integer> seen = new HashSet<>();
if (seen.contains(num)) { ... }
```

---

### Q5: How to iterate HashMap in sorted order?

**Answer:**

**Option 1: Sort keys**
```java
List<String> sortedKeys = new ArrayList<>(map.keySet());
Collections.sort(sortedKeys);
for (String key : sortedKeys) {
    System.out.println(key + ": " + map.get(key));
}
```

**Option 2: Use TreeMap**
```java
Map<String, Integer> sortedMap = new TreeMap<>(map);
for (Map.Entry<String, Integer> entry : sortedMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

**Option 3: Sort by value**
```java
List<Map.Entry<String, Integer>> entries = new ArrayList<>(map.entrySet());
entries.sort((a, b) -> b.getValue() - a.getValue());  // Descending
```

---

### Q6: How to find duplicate elements using HashMap?

**Answer:**

**Approach 1: Frequency Map**
```java
int[] nums = {1, 2, 3, 2, 4, 2};
Map<Integer, Integer> freq = new HashMap<>();

for (int num : nums) {
    freq.put(num, freq.getOrDefault(num, 0) + 1);
}

// Find duplicates (count > 1)
for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
    if (entry.getValue() > 1) {
        System.out.println("Duplicate: " + entry.getKey());
    }
}
```

**Approach 2: HashSet (if only need to detect)**
```java
Set<Integer> seen = new HashSet<>();
for (int num : nums) {
    if (seen.contains(num)) {
        System.out.println("Duplicate: " + num);
    }
    seen.add(num);
}
```

---

### Q7: How to implement LRU Cache using HashMap?

**Answer:**

**Using LinkedHashMap:**
```java
class LRUCache {
    private LinkedHashMap<Integer, Integer> cache;
    private int capacity;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new LinkedHashMap<>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
                return size() > capacity;
            }
        };
    }
    
    public int get(int key) {
        return cache.getOrDefault(key, -1);
    }
    
    public void put(int key, int value) {
        cache.put(key, value);
    }
}
```

**Using HashMap + Doubly Linked List:**
```java
class LRUCache {
    class Node {
        int key, value;
        Node prev, next;
    }
    
    private Map<Integer, Node> cache = new HashMap<>();
    private int capacity;
    private Node head, tail;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        head = new Node();
        tail = new Node();
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        Node node = cache.get(key);
        if (node == null) return -1;
        
        moveToHead(node);
        return node.value;
    }
    
    public void put(int key, int value) {
        Node node = cache.get(key);
        
        if (node == null) {
            Node newNode = new Node();
            newNode.key = key;
            newNode.value = value;
            
            cache.put(key, newNode);
            addNode(newNode);
            
            if (cache.size() > capacity) {
                Node tail = popTail();
                cache.remove(tail.key);
            }
        } else {
            node.value = value;
            moveToHead(node);
        }
    }
    
    private void addNode(Node node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }
    
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private void moveToHead(Node node) {
        removeNode(node);
        addNode(node);
    }
    
    private Node popTail() {
        Node last = tail.prev;
        removeNode(last);
        return last;
    }
}
```

---

### Q8: How to find intersection of two arrays?

**Answer:**

**Using HashMap:**
```java
int[] nums1 = {1, 2, 2, 1};
int[] nums2 = {2, 2};

Map<Integer, Integer> freq = new HashMap<>();
for (int num : nums1) {
    freq.put(num, freq.getOrDefault(num, 0) + 1);
}

List<Integer> result = new ArrayList<>();
for (int num : nums2) {
    if (freq.containsKey(num) && freq.get(num) > 0) {
        result.add(num);
        freq.put(num, freq.get(num) - 1);
    }
}

return result.stream().mapToInt(i -> i).toArray();
```

**Visual:**
```
nums1 = [1, 2, 2, 1]
freq = {1: 2, 2: 2}

nums2 = [2, 2]
num=2: in map, count=2 > 0 → add, decrement → {1: 2, 2: 1}
num=2: in map, count=1 > 0 → add, decrement → {1: 2, 2: 0}

Result: [2, 2]
```

---

### Q9: How to group anagrams together?

**Answer:**

**Key Insight:** Sort each string, use sorted string as key

```java
String[] strs = {"eat", "tea", "tan", "ate", "nat", "bat"};

Map<String, List<String>> groups = new HashMap<>();

for (String str : strs) {
    char[] chars = str.toCharArray();
    Arrays.sort(chars);
    String key = new String(chars);
    
    groups.computeIfAbsent(key, k -> new ArrayList<>()).add(str);
}

return new ArrayList<>(groups.values());
```

**Visual:**
```
strs = ["eat", "tea", "tan", "ate", "nat", "bat"]

"eat" → sort → "aet" → {aet: [eat]}
"tea" → sort → "aet" → {aet: [eat, tea]}
"tan" → sort → "ant" → {aet: [eat, tea], ant: [tan]}
"ate" → sort → "aet" → {aet: [eat, tea, ate], ant: [tan]}
"nat" → sort → "ant" → {aet: [eat, tea, ate], ant: [tan, nat]}
"bat" → sort → "abt" → {aet: [eat, tea, ate], ant: [tan, nat], abt: [bat]}

Result: [[eat, tea, ate], [tan, nat], [bat]]
```

---

### Q10: How to find all pairs with given sum?

**Answer:**

**Two Sum (find indices):**
```java
int[] nums = {2, 7, 11, 15};
int target = 9;

Map<Integer, Integer> map = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (map.containsKey(complement)) {
        return new int[]{map.get(complement), i};
    }
    map.put(nums[i], i);
}
```

**All Pairs (find all pairs):**
```java
int[] nums = {1, 2, 3, 4, 5};
int target = 6;

Map<Integer, List<Integer>> map = new HashMap<>();
List<int[]> result = new ArrayList<>();

for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (map.containsKey(complement)) {
        for (int idx : map.get(complement)) {
            result.add(new int[]{idx, i});
        }
    }
    map.computeIfAbsent(nums[i], k -> new ArrayList<>()).add(i);
}

return result;
```

---

## Quick Reference

### Common Patterns Summary

| Pattern | When to Use | Example |
|---------|-------------|---------|
| **Frequency Counting** | Count occurrences | Character/word frequency |
| **Two Sum** | Find pairs | Complement pattern |
| **Grouping** | Group by property | Group strings by length |
| **Index Mapping** | Store indices | Find all indices of element |
| **Sliding Window** | Subarray problems | Longest substring with K distinct |
| **Prefix Sum** | Range queries | Subarray sum equals K |
| **Anagram** | Character matching | Group anagrams |
| **LRU Cache** | Cache eviction | Least recently used |

### Time Complexity Cheat Sheet

| Operation | HashMap | TreeMap | LinkedHashMap |
|-----------|---------|---------|---------------|
| **get()** | O(1) avg | O(log n) | O(1) avg |
| **put()** | O(1) avg | O(log n) | O(1) avg |
| **remove()** | O(1) avg | O(log n) | O(1) avg |
| **containsKey()** | O(1) avg | O(log n) | O(1) avg |
| **Iteration** | O(n) | O(n) | O(n) |
| **Order** | No order | Sorted | Insertion order |

---

## Memory Tips 🧠

### HashMap
- **H**ash **M**ap = **H**ash + **M**ap (key-value)
- **O(1)** average, **O(n)** worst case
- **Load factor 0.75** = 75% full before resize

### HashSet
- **H**ash **S**et = **H**ash + **S**et (unique elements)
- Same as HashMap but no values

### TreeMap
- **T**ree **M**ap = **T**ree + **M**ap (sorted keys)
- **O(log n)** operations, sorted order

### LinkedHashMap
- **L**inked **H**ash **M**ap = **L**inked + **H**ash + **M**ap
- Maintains insertion order

---

## Daily Revision Checklist ✅

- [ ] HashMap basic operations (put, get, remove)
- [ ] Frequency counting pattern
- [ ] Two sum / complement pattern
- [ ] Grouping elements pattern
- [ ] Sliding window with HashMap
- [ ] Prefix sum with HashMap
- [ ] Anagram detection
- [ ] LRU Cache implementation
- [ ] Time/space complexity
- [ ] Common tricks (getOrDefault, computeIfAbsent)

---

**Happy Coding! 🗺️**

