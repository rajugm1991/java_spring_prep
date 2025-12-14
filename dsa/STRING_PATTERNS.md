# String Patterns - Complete Guide 🎯

> Master string problem-solving patterns with visual explanations and recognition strategies

---

## Table of Contents
1. [How to Recognize String Patterns](#how-to-recognize-string-patterns)
2. [Two Pointers Pattern](#two-pointers-pattern)
3. [Sliding Window Pattern](#sliding-window-pattern)
4. [Hash Map/Set Pattern](#hash-mapset-pattern)
5. [Trie (Prefix Tree) Pattern](#trie-prefix-tree-pattern)
6. [KMP Algorithm Pattern](#kmp-algorithm-pattern)
7. [String Manipulation Pattern](#string-manipulation-pattern)
8. [Backtracking with Strings](#backtracking-with-strings)
9. [Dynamic Programming with Strings](#dynamic-programming-with-strings)
10. [Frequently Asked Questions](#frequently-asked-questions)

---

## How to Recognize String Patterns

### Pattern Recognition Checklist

When you see a string problem, ask these questions:

#### 1. **Do you need to find a substring?**
- ✅ **Yes** → Consider **Sliding Window** or **KMP Algorithm**
- ❌ **No** → Continue to next question

#### 2. **Do you need to check characters/counts?**
- ✅ **Yes** → **Hash Map/Set** for frequency counting
- ❌ **No** → Continue to next question

#### 3. **Is the string sorted or can be sorted?**
- ✅ **Yes** → **Two Pointers** for pairs/palindromes
- ❌ **No** → Continue to next question

#### 4. **Do you need prefix/suffix matching?**
- ✅ **Yes** → **Trie** or **KMP Algorithm**
- ❌ **No** → Continue to next question

#### 5. **Do you need to generate combinations/permutations?**
- ✅ **Yes** → **Backtracking**
- ❌ **No** → Continue to next question

#### 6. **Is it a pattern matching problem?**
- ✅ **Yes** → **KMP** or **Rabin-Karp**
- ❌ **No** → Continue to next question

#### 7. **Do you need optimal substructure?**
- ✅ **Yes** → **Dynamic Programming**
- ❌ **No** → Continue to next question

### Quick Pattern Decision Tree

```
String Problem
    │
    ├─ Substring? ──Yes──→ Sliding Window / KMP
    │
    ├─ Character Counts? ──Yes──→ Hash Map/Set
    │
    ├─ Sorted/Palindrome? ──Yes──→ Two Pointers
    │
    ├─ Prefix/Suffix? ──Yes──→ Trie / KMP
    │
    ├─ Combinations? ──Yes──→ Backtracking
    │
    ├─ Pattern Match? ──Yes──→ KMP / Rabin-Karp
    │
    └─ Optimal Substructure? ──Yes──→ Dynamic Programming
```

---

## Two Pointers Pattern

### When to Use

**Recognize:**
- ✅ Palindrome problems
- ✅ Valid parentheses
- ✅ Reverse string
- ✅ Remove duplicates
- ✅ String compression

**Key Insight:** Use two pointers moving from different positions

### Visual Pattern

```
String: "hello"
         ↑    ↑
        left right

Move pointers based on condition:
- If palindrome check: left++, right--
- If valid parentheses: track balance
```

### Example 1: Valid Palindrome

**Problem:** Check if string is palindrome (ignoring case and non-alphanumeric)

**Visual:**
```
String: "A man, a plan, a canal: Panama"

Step 1: Clean and normalize
  "amanaplanacanalpanama"

Step 2: Two pointers
  "amanaplanacanalpanama"
   ↑                    ↑
  left                right
  'a' == 'a' ✅ → left++, right--

Step 3:
  "amanaplanacanalpanama"
    ↑                  ↑
  'm' == 'm' ✅ → continue

... continue until left >= right
All match → Palindrome ✅
```

**Code:**
```java
boolean isPalindrome(String s) {
    int left = 0;
    int right = s.length() - 1;
    
    while (left < right) {
        // Skip non-alphanumeric
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) {
            left++;
        }
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) {
            right--;
        }
        
        // Compare (case-insensitive)
        if (Character.toLowerCase(s.charAt(left)) != 
            Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        
        left++;
        right--;
    }
    
    return true;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 2: Reverse String

**Problem:** Reverse string in-place

**Visual:**
```
String: "hello"

Step 1:
  "hello"
   ↑   ↑
  i=0 j=4
  swap('h', 'o') → "oellh"

Step 2:
  "oellh"
    ↑ ↑
  i=1 j=3
  swap('e', 'l') → "olleh"

Step 3:
  "olleh"
     ↑↑
  i=2 j=2
  i >= j → Stop ✅
```

**Code:**
```java
void reverseString(char[] s) {
    int left = 0;
    int right = s.length - 1;
    
    while (left < right) {
        char temp = s[left];
        s[left] = s[right];
        s[right] = temp;
        
        left++;
        right--;
    }
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### Example 3: Valid Parentheses

**Problem:** Check if parentheses are valid

**Visual:**
```
String: "()[]{}"

Use Stack + Two Pointers approach:

Step 1: '(' → push
Stack: ['(']

Step 2: ')' → pop, matches ✅
Stack: []

Step 3: '[' → push
Stack: ['[']

Step 4: ']' → pop, matches ✅
Stack: []

Step 5: '{' → push
Stack: ['{']

Step 6: '}' → pop, matches ✅
Stack: []

Stack empty → Valid ✅
```

**Code:**
```java
boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            
            char top = stack.pop();
            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{')) {
                return false;
            }
        }
    }
    
    return stack.isEmpty();
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## Sliding Window Pattern

### When to Use

**Recognize:**
- ✅ Substring problems
- ✅ Longest/shortest substring with condition
- ✅ Anagrams
- ✅ Character frequency in window
- ✅ Minimum window substring

**Key Insight:** Maintain a window and slide it based on condition

### Visual Pattern

```
String: "abcabcbb"
Window: [abc]abcbb  (size 3)
        a[bca]bcbb  (slide right)
        ab[cab]cbb  (slide right)
```

### Example 1: Longest Substring Without Repeating Characters

**Problem:** Find longest substring with unique characters

**Visual:**
```
String: "abcabcbb"

Step 1: Window [a]
  "abcabcbb"
   ↑
  start=0, end=0
  map: {a:0}
  maxLen = 1

Step 2: Window [ab]
  "abcabcbb"
   ↑↑
  start=0, end=1
  map: {a:0, b:1}
  maxLen = 2

Step 3: Window [abc]
  "abcabcbb"
   ↑ ↑
  start=0, end=2
  map: {a:0, b:1, c:2}
  maxLen = 3

Step 4: Window [bca] (duplicate 'a' found)
  "abcabcbb"
    ↑  ↑
  start=1, end=3
  map: {a:3, b:1, c:2}
  maxLen = 3

Step 5: Window [cab]
  "abcabcbb"
     ↑  ↑
  start=2, end=4
  maxLen = 3

... continue
Result: "abc" (length 3) ✅
```

**Code:**
```java
int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int maxLen = 0;
    int start = 0;
    
    for (int end = 0; end < s.length(); end++) {
        char c = s.charAt(end);
        
        // If character seen before, move start
        if (map.containsKey(c)) {
            start = Math.max(start, map.get(c) + 1);
        }
        
        map.put(c, end);
        maxLen = Math.max(maxLen, end - start + 1);
    }
    
    return maxLen;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(min(n, m)) where m is charset size

---

### Example 2: Minimum Window Substring

**Problem:** Find minimum window in S that contains all characters of T

**Visual:**
```
S = "ADOBECODEBANC", T = "ABC"

Step 1: Expand window
  "ADOBECODEBANC"
   ↑      ↑
  start=0, end=5
  Window: "ADOBEC" contains A, B, C ✅
  minLen = 6

Step 2: Shrink window
  "ADOBECODEBANC"
    ↑     ↑
  start=1, end=5
  Window: "DOBEC" doesn't contain A ❌

Step 3: Expand again
  "ADOBECODEBANC"
    ↑          ↑
  start=1, end=10
  Window: "DOBECODEBA" contains A, B, C ✅
  minLen = 10

Step 4: Shrink
  "ADOBECODEBANC"
         ↑    ↑
  start=5, end=10
  Window: "CODEBA" contains A, B, C ✅
  minLen = 6

Step 5: Continue...
  Final: "BANC" (length 4) ✅
```

**Code:**
```java
String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    
    // Count characters in t
    for (char c : t.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }
    
    int left = 0, right = 0;
    int valid = 0;  // Number of characters that satisfy condition
    int start = 0, len = Integer.MAX_VALUE;
    
    while (right < s.length()) {
        char c = s.charAt(right);
        right++;
        
        // Update window
        if (need.containsKey(c)) {
            window.put(c, window.getOrDefault(c, 0) + 1);
            if (window.get(c).equals(need.get(c))) {
                valid++;
            }
        }
        
        // Try to shrink window
        while (valid == need.size()) {
            // Update result
            if (right - left < len) {
                start = left;
                len = right - left;
            }
            
            char d = s.charAt(left);
            left++;
            
            if (need.containsKey(d)) {
                if (window.get(d).equals(need.get(d))) {
                    valid--;
                }
                window.put(d, window.get(d) - 1);
            }
        }
    }
    
    return len == Integer.MAX_VALUE ? "" : s.substring(start, start + len);
}
```

**Time Complexity:** O(|s| + |t|)  
**Space Complexity:** O(|s| + |t|)

---

### Example 3: Find All Anagrams

**Problem:** Find all anagram substrings of pattern in string

**Visual:**
```
s = "cbaebabacd", p = "abc"

Step 1: Window size = 3
  "cbaebabacd"
   ↑  ↑
  Window: "cba" → count: {a:1, b:1, c:1}
  Pattern: "abc" → count: {a:1, b:1, c:1}
  Match ✅ → Add index 0

Step 2: Slide window
  "cbaebabacd"
    ↑  ↑
  Window: "bae" → count: {a:1, b:1, e:1}
  Pattern: "abc" → count: {a:1, b:1, c:1}
  Not match ❌

Step 3: Continue...
  Window: "eba" → Not match
  Window: "bab" → Not match
  Window: "aba" → Not match
  Window: "bac" → Match ✅ → Add index 6

Result: [0, 6]
```

**Code:**
```java
List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    
    if (s.length() < p.length()) return result;
    
    Map<Character, Integer> pCount = new HashMap<>();
    Map<Character, Integer> sCount = new HashMap<>();
    
    // Count characters in pattern
    for (char c : p.toCharArray()) {
        pCount.put(c, pCount.getOrDefault(c, 0) + 1);
    }
    
    // Sliding window
    for (int i = 0; i < s.length(); i++) {
        // Add right character
        char c = s.charAt(i);
        sCount.put(c, sCount.getOrDefault(c, 0) + 1);
        
        // Remove left character if window size > p.length()
        if (i >= p.length()) {
            char leftChar = s.charAt(i - p.length());
            sCount.put(leftChar, sCount.get(leftChar) - 1);
            if (sCount.get(leftChar) == 0) {
                sCount.remove(leftChar);
            }
        }
        
        // Check if window matches pattern
        if (sCount.equals(pCount)) {
            result.add(i - p.length() + 1);
        }
    }
    
    return result;
}
```

**Time Complexity:** O(|s|)  
**Space Complexity:** O(|p|)

---

## Hash Map/Set Pattern

### When to Use

**Recognize:**
- ✅ Character frequency counting
- ✅ Anagram problems
- ✅ First unique/non-repeating character
- ✅ Group anagrams
- ✅ Word frequency

**Key Insight:** Use hash map to count characters/words

### Visual Pattern

```
String: "hello"
Map: {
  'h': 1,
  'e': 1,
  'l': 2,
  'o': 1
}
```

### Example 1: First Unique Character

**Problem:** Find first non-repeating character

**Visual:**
```
String: "leetcode"

Step 1: Count frequencies
  'l': 1
  'e': 3
  't': 1
  'c': 1
  'o': 1
  'd': 1

Step 2: Find first with count = 1
  Index 0: 'l' → count = 1 ✅
  Return index 0
```

**Code:**
```java
int firstUniqChar(String s) {
    Map<Character, Integer> count = new HashMap<>();
    
    // Count frequencies
    for (char c : s.toCharArray()) {
        count.put(c, count.getOrDefault(c, 0) + 1);
    }
    
    // Find first unique
    for (int i = 0; i < s.length(); i++) {
        if (count.get(s.charAt(i)) == 1) {
            return i;
        }
    }
    
    return -1;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) - at most 26 characters

---

### Example 2: Group Anagrams

**Problem:** Group strings that are anagrams

**Visual:**
```
Input: ["eat", "tea", "tan", "ate", "nat", "bat"]

Step 1: Sort each string
  "eat" → "aet"
  "tea" → "aet"
  "tan" → "ant"
  "ate" → "aet"
  "nat" → "ant"
  "bat" → "abt"

Step 2: Group by sorted string
  "aet" → ["eat", "tea", "ate"]
  "ant" → ["tan", "nat"]
  "abt" → ["bat"]

Result: [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

**Code:**
```java
List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    
    for (String str : strs) {
        // Sort string to get key
        char[] chars = str.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        
        // Add to map
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(str);
    }
    
    return new ArrayList<>(map.values());
}
```

**Time Complexity:** O(n * k log k) where k is max string length  
**Space Complexity:** O(n * k)

---

### Example 3: Word Pattern

**Problem:** Check if string follows pattern

**Visual:**
```
pattern = "abba", str = "dog cat cat dog"

Step 1: Split string
  words = ["dog", "cat", "cat", "dog"]

Step 2: Map pattern to words
  'a' → "dog"
  'b' → "cat"
  'a' → "dog" (matches ✅)
  'b' → "cat" (matches ✅)

Result: true ✅
```

**Code:**
```java
boolean wordPattern(String pattern, String str) {
    String[] words = str.split(" ");
    
    if (pattern.length() != words.length) return false;
    
    Map<Character, String> charToWord = new HashMap<>();
    Map<String, Character> wordToChar = new HashMap<>();
    
    for (int i = 0; i < pattern.length(); i++) {
        char c = pattern.charAt(i);
        String word = words[i];
        
        if (charToWord.containsKey(c)) {
            if (!charToWord.get(c).equals(word)) {
                return false;
            }
        } else {
            if (wordToChar.containsKey(word)) {
                return false;
            }
            charToWord.put(c, word);
            wordToChar.put(word, c);
        }
    }
    
    return true;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## Trie (Prefix Tree) Pattern

### When to Use

**Recognize:**
- ✅ Prefix matching
- ✅ Word search
- ✅ Autocomplete
- ✅ Longest common prefix
- ✅ Word dictionary problems

**Key Insight:** Build tree structure for efficient prefix search

### Visual Pattern

```
Trie for words: ["cat", "car", "dog"]

        root
       /    \
      c      d
     /        \
    a          o
   / \          \
  t   r          g
```

### Example 1: Implement Trie

**Problem:** Implement Trie data structure

**Visual:**
```
Insert "cat":
  root → c → a → t (end)

Insert "car":
  root → c → a → r (end)
              \
               t (end)

Search "cat": Traverse c → a → t → found ✅
Search "ca": Traverse c → a → not end ❌
```

**Code:**
```java
class TrieNode {
    TrieNode[] children;
    boolean isEnd;
    
    TrieNode() {
        children = new TrieNode[26];
        isEnd = false;
    }
}

class Trie {
    private TrieNode root;
    
    public Trie() {
        root = new TrieNode();
    }
    
    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                node.children[index] = new TrieNode();
            }
            node = node.children[index];
        }
        node.isEnd = true;
    }
    
    public boolean search(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                return false;
            }
            node = node.children[index];
        }
        return node.isEnd;
    }
    
    public boolean startsWith(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                return false;
            }
            node = node.children[index];
        }
        return true;
    }
}
```

**Time Complexity:**  
- Insert: O(m) where m is word length
- Search: O(m)
- startsWith: O(m)

**Space Complexity:** O(ALPHABET_SIZE * N * M) where N is number of words

---

### Example 2: Longest Common Prefix

**Problem:** Find longest common prefix using Trie

**Visual:**
```
Words: ["flower", "flow", "flight"]

Build Trie:
        root
         |
         f
         |
         l
        / \
       o   i
      /     \
     w       g
    /         \
   e           h
  /             \
 r               t

Find common prefix:
  root → f → l → (branch at o/i)
  Common prefix: "fl"
```

**Code:**
```java
String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    
    // Build Trie
    Trie trie = new Trie();
    for (String str : strs) {
        trie.insert(str);
    }
    
    // Find common prefix
    StringBuilder prefix = new StringBuilder();
    TrieNode node = trie.root;
    
    while (node != null) {
        int count = 0;
        int index = -1;
        
        // Count children
        for (int i = 0; i < 26; i++) {
            if (node.children[i] != null) {
                count++;
                index = i;
            }
        }
        
        // If more than one child, stop
        if (count != 1) break;
        
        prefix.append((char)('a' + index));
        node = node.children[index];
        
        // If end of word, stop
        if (node.isEnd) break;
    }
    
    return prefix.toString();
}
```

**Time Complexity:** O(S) where S is sum of all characters  
**Space Complexity:** O(S)

---

## KMP Algorithm Pattern

### When to Use

**Recognize:**
- ✅ Pattern matching in string
- ✅ Find substring occurrences
- ✅ String search optimization
- ✅ Repeated substring problems

**Key Insight:** Use failure function (LPS array) to avoid re-checking

### Visual Pattern

```
Text: "ABABDABACDABABCABCABAB"
Pattern: "ABABCABAB"

Build LPS (Longest Proper Prefix which is also Suffix):
Pattern: A B A B C A B A B
LPS:     0 0 1 2 0 1 2 3 4

Search using LPS to skip characters
```

### Example 1: Implement KMP

**Problem:** Find all occurrences of pattern in text

**Visual:**
```
Text: "ABABDABACDABABCABCABAB"
Pattern: "ABABCABAB"

Step 1: Build LPS array
  Pattern: A B A B C A B A B
  Index:   0 1 2 3 4 5 6 7 8
  LPS:     0 0 1 2 0 1 2 3 4

Step 2: Search
  Text:    ABABDABACDABABCABCABAB
  Pattern: ABABCABAB
           ↑
  Match until index 4, mismatch at 'D' vs 'C'
  
  Use LPS: Skip 2 characters (LPS[3] = 2)
  Text:    ABABDABACDABABCABCABAB
  Pattern:   ABABCABAB
             ↑
  Continue...
```

**Code:**
```java
List<Integer> kmpSearch(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    int[] lps = buildLPS(pattern);
    
    int i = 0;  // text index
    int j = 0;  // pattern index
    
    while (i < text.length()) {
        if (text.charAt(i) == pattern.charAt(j)) {
            i++;
            j++;
        }
        
        if (j == pattern.length()) {
            result.add(i - j);  // Found pattern
            j = lps[j - 1];
        } else if (i < text.length() && text.charAt(i) != pattern.charAt(j)) {
            if (j != 0) {
                j = lps[j - 1];
            } else {
                i++;
            }
        }
    }
    
    return result;
}

int[] buildLPS(String pattern) {
    int[] lps = new int[pattern.length()];
    int len = 0;
    int i = 1;
    
    while (i < pattern.length()) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len != 0) {
                len = lps[len - 1];
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
    
    return lps;
}
```

**Time Complexity:** O(n + m) where n is text length, m is pattern length  
**Space Complexity:** O(m)

---

### Example 2: Repeated Substring Pattern

**Problem:** Check if string can be formed by repeating a substring

**Visual:**
```
String: "ababab"

Use KMP to find if string is repetition:
  Build LPS: [0, 0, 1, 2, 3, 4]
  
  If LPS[n-1] > 0 and n % (n - LPS[n-1]) == 0:
    String is repetition ✅
  
  n = 6, LPS[5] = 4
  6 % (6 - 4) = 6 % 2 = 0 ✅
  
  Repeated substring: "ab" (length 2)
```

**Code:**
```java
boolean repeatedSubstringPattern(String s) {
    int n = s.length();
    int[] lps = buildLPS(s);
    
    int len = lps[n - 1];
    return len > 0 && n % (n - len) == 0;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## String Manipulation Pattern

### When to Use

**Recognize:**
- ✅ String reversal
- ✅ Character replacement
- ✅ String splitting/joining
- ✅ Case conversion
- ✅ String formatting

### Example 1: Reverse Words in String

**Problem:** Reverse order of words

**Visual:**
```
Input: "the sky is blue"
Output: "blue is sky the"

Step 1: Trim and split
  ["the", "sky", "is", "blue"]

Step 2: Reverse array
  ["blue", "is", "sky", "the"]

Step 3: Join
  "blue is sky the"
```

**Code:**
```java
String reverseWords(String s) {
    // Trim and split
    String[] words = s.trim().split("\\s+");
    
    // Reverse
    int left = 0, right = words.length - 1;
    while (left < right) {
        String temp = words[left];
        words[left] = words[right];
        words[right] = temp;
        left++;
        right--;
    }
    
    // Join
    return String.join(" ", words);
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### Example 2: String Compression

**Problem:** Compress string (aaabb → a3b2)

**Visual:**
```
Input: "aaabbcc"
Output: "a3b2c2"

Step 1: Count consecutive
  'a': 3 → "a3"
  'b': 2 → "b2"
  'c': 2 → "c2"

Step 2: Build result
  "a3b2c2"
```

**Code:**
```java
String compress(String chars) {
    StringBuilder result = new StringBuilder();
    int count = 1;
    
    for (int i = 1; i < chars.length(); i++) {
        if (chars.charAt(i) == chars.charAt(i - 1)) {
            count++;
        } else {
            result.append(chars.charAt(i - 1));
            if (count > 1) {
                result.append(count);
            }
            count = 1;
        }
    }
    
    // Add last character
    result.append(chars.charAt(chars.length() - 1));
    if (count > 1) {
        result.append(count);
    }
    
    return result.toString();
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## Backtracking with Strings

### When to Use

**Recognize:**
- ✅ Generate all combinations/permutations
- ✅ Phone number combinations
- ✅ Word search
- ✅ Palindrome partitioning

### Example 1: Letter Combinations of Phone Number

**Problem:** Generate all letter combinations from phone number

**Visual:**
```
Input: "23"

Mapping:
  2: "abc"
  3: "def"

Backtracking:
  Start: ""
  Level 1: "a", "b", "c"
  Level 2: "ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"

Result: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

**Code:**
```java
List<String> letterCombinations(String digits) {
    List<String> result = new ArrayList<>();
    if (digits.isEmpty()) return result;
    
    String[] mapping = {
        "", "", "abc", "def", "ghi", "jkl",
        "mno", "pqrs", "tuv", "wxyz"
    };
    
    backtrack(digits, 0, new StringBuilder(), result, mapping);
    return result;
}

void backtrack(String digits, int index, StringBuilder current,
               List<String> result, String[] mapping) {
    if (index == digits.length()) {
        result.add(current.toString());
        return;
    }
    
    String letters = mapping[digits.charAt(index) - '0'];
    for (char c : letters.toCharArray()) {
        current.append(c);
        backtrack(digits, index + 1, current, result, mapping);
        current.deleteCharAt(current.length() - 1);  // Backtrack
    }
}
```

**Time Complexity:** O(4^n) where n is digits length  
**Space Complexity:** O(n) for recursion stack

---

### Example 2: Word Search

**Problem:** Find if word exists in 2D grid

**Visual:**
```
Grid:
  A B C E
  S F C S
  A D E E

Word: "ABCCED"

DFS Backtracking:
  Start at (0,0): 'A' ✅
  → (0,1): 'B' ✅
  → (0,2): 'C' ✅
  → (1,2): 'C' ✅
  → (2,2): 'E' ✅
  → (2,1): 'D' ✅
  Found! ✅
```

**Code:**
```java
boolean exist(char[][] board, String word) {
    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[0].length; j++) {
            if (dfs(board, i, j, word, 0)) {
                return true;
            }
        }
    }
    return false;
}

boolean dfs(char[][] board, int i, int j, String word, int index) {
    if (index == word.length()) return true;
    
    if (i < 0 || i >= board.length || j < 0 || j >= board[0].length ||
        board[i][j] != word.charAt(index)) {
        return false;
    }
    
    char temp = board[i][j];
    board[i][j] = '#';  // Mark as visited
    
    boolean found = dfs(board, i + 1, j, word, index + 1) ||
                   dfs(board, i - 1, j, word, index + 1) ||
                   dfs(board, i, j + 1, word, index + 1) ||
                   dfs(board, i, j - 1, word, index + 1);
    
    board[i][j] = temp;  // Backtrack
    return found;
}
```

**Time Complexity:** O(m * n * 4^L) where L is word length  
**Space Complexity:** O(L) for recursion stack

---

## Dynamic Programming with Strings

### When to Use

**Recognize:**
- ✅ Longest common subsequence
- ✅ Edit distance
- ✅ Palindrome problems
- ✅ String matching with wildcards
- ✅ Optimal substructure

### Example 1: Longest Common Subsequence

**Problem:** Find longest common subsequence between two strings

**Visual:**
```
s1 = "abcde", s2 = "ace"

DP Table:
      ""  a  c  e
  ""   0  0  0  0
  a    0  1  1  1
  b    0  1  1  1
  c    0  1  2  2
  d    0  1  2  2
  e    0  1  2  3

LCS: "ace" (length 3)
```

**Code:**
```java
int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length();
    int n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[m][n];
}
```

**Time Complexity:** O(m * n)  
**Space Complexity:** O(m * n)

---

### Example 2: Edit Distance

**Problem:** Minimum operations to convert word1 to word2

**Visual:**
```
word1 = "horse", word2 = "ros"

DP Table:
      ""  r  o  s
  ""   0  1  2  3
  h    1  1  2  3
  o    2  2  1  2
  r    3  2  2  2
  s    4  3  3  2
  e    5  4  4  3

Operations:
  horse → rorse (replace h with r)
  rorse → rose (remove r)
  rose → ros (remove e)

Edit distance: 3
```

**Code:**
```java
int minDistance(String word1, String word2) {
    int m = word1.length();
    int n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    // Base cases
    for (int i = 0; i <= m; i++) {
        dp[i][0] = i;  // Delete all characters
    }
    for (int j = 0; j <= n; j++) {
        dp[0][j] = j;  // Insert all characters
    }
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];  // No operation
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i - 1][j],      // Delete
                    Math.min(
                        dp[i][j - 1],  // Insert
                        dp[i - 1][j - 1]  // Replace
                    )
                );
            }
        }
    }
    
    return dp[m][n];
}
```

**Time Complexity:** O(m * n)  
**Space Complexity:** O(m * n)

---

## Frequently Asked Questions

### Q1: How do I recognize when to use Two Pointers for strings?

**Answer:**
"Use Two Pointers when:
1. **Palindrome problems** - Check if string is palindrome
2. **Valid parentheses** - Match opening and closing
3. **Reverse string** - Swap characters from both ends
4. **String is sorted** - Can use two pointers for pairs

Example: Valid Palindrome - use two pointers from start and end, moving towards center."

---

### Q2: When should I use Sliding Window for strings?

**Answer:**
"Use Sliding Window when:
1. **Substring problems** - Need to find substring with condition
2. **Longest/shortest substring** - With specific property
3. **Anagram problems** - Find anagram substrings
4. **Character frequency** - Count characters in window

Example: Longest Substring Without Repeating Characters - maintain window with unique characters, expand/shrink based on duplicates."

---

### Q3: How do I know when to use Hash Map for strings?

**Answer:**
"Use Hash Map when:
1. **Character counting** - Count frequency of characters
2. **Anagram problems** - Compare character counts
3. **First unique character** - Track character occurrences
4. **Pattern matching** - Map pattern to words

Example: Group Anagrams - use sorted string as key, group words with same character counts."

---

### Q4: When should I use Trie?

**Answer:**
"Use Trie when:
1. **Prefix matching** - Check if prefix exists
2. **Autocomplete** - Find all words with prefix
3. **Word dictionary** - Efficient word lookup
4. **Longest common prefix** - Find common prefix of multiple strings

Example: Implement Autocomplete - build Trie, search by prefix, return all words with that prefix."

---

### Q5: How do I recognize KMP algorithm problems?

**Answer:**
"Use KMP when:
1. **Pattern matching** - Find pattern in text efficiently
2. **Repeated substring** - Check if string is repetition
3. **String search optimization** - Avoid re-checking characters
4. **Multiple pattern searches** - Search multiple patterns

Example: Find All Occurrences - use KMP to find all positions where pattern appears in text, using LPS array to skip characters."

---

### Q6: What's the difference between substring and subsequence?

**Answer:**
"**Substring**: Contiguous characters from string
- Example: "abc" is substring of "abcdef"
- Must be consecutive

**Subsequence**: Characters in order but not necessarily contiguous
- Example: "ace" is subsequence of "abcdef"
- Can skip characters

For substring → Use Sliding Window
For subsequence → Use Dynamic Programming"

---

### Q7: How do I solve palindrome problems?

**Answer:**
"Palindrome problems can use:
1. **Two Pointers** - Check from both ends
2. **Dynamic Programming** - Build palindrome table
3. **Expand Around Centers** - Check all possible centers

Example: Longest Palindromic Substring
- Use expand around centers
- For each position, expand left and right
- Track longest palindrome found"

---

### Q8: When should I use Backtracking for strings?

**Answer:**
"Use Backtracking when:
1. **Generate combinations** - All possible combinations
2. **Generate permutations** - All possible arrangements
3. **Word search** - Find word in grid
4. **Partition problems** - Partition string into valid parts

Example: Letter Combinations of Phone Number
- Generate all combinations
- Backtrack when invalid
- Add to result when complete"

---

### Q9: How do I optimize string operations?

**Answer:**
"Optimize by:
1. **Use StringBuilder** - For string concatenation (O(n) vs O(n²))
2. **Pre-allocate** - If you know size, pre-allocate array
3. **Avoid substring** - Use indices instead of creating new strings
4. **Character array** - Convert to char[] for in-place operations
5. **Hash Map caching** - Cache computed results

Example: String compression - use StringBuilder instead of string concatenation to avoid O(n²) complexity."

---

### Q10: What's the time complexity of common string operations?

**Answer:**
"Common complexities:
- **Substring**: O(n) where n is substring length
- **String concatenation**: O(n + m) for two strings
- **String comparison**: O(min(n, m))
- **String search (naive)**: O(n * m)
- **String search (KMP)**: O(n + m)
- **Trie search**: O(m) where m is pattern length
- **Hash Map lookup**: O(1) average, O(n) worst case"

---

## Quick Reference Table

| Pattern | When to Use | Time Complexity | Space Complexity |
|---------|-------------|----------------|------------------|
| **Two Pointers** | Palindrome, reverse | O(n) | O(1) |
| **Sliding Window** | Substring problems | O(n) | O(k) |
| **Hash Map/Set** | Character counting | O(n) | O(k) |
| **Trie** | Prefix matching | O(m) | O(ALPHABET * N * M) |
| **KMP** | Pattern matching | O(n + m) | O(m) |
| **Backtracking** | Combinations/permutations | O(2^n) | O(n) |
| **Dynamic Programming** | Optimal substructure | O(m * n) | O(m * n) |

---

## Memory Tips 🧠

### Two Pointers
- **P**alindrome = **P**ointers from both ends
- **R**everse = **R**ight and left swap

### Sliding Window
- **S**ubstring = **S**liding window
- **W**indow = **W**iden and shrink

### Hash Map
- **C**ount = **C**haracter map
- **A**nagram = **A**lphabet count

### Trie
- **P**refix = **P**refix tree
- **T**ree = **T**rie structure

---

## Daily Revision Checklist ✅

- [ ] Two Pointers: Palindrome, reverse
- [ ] Sliding Window: Longest substring, anagrams
- [ ] Hash Map: Character counting, anagrams
- [ ] Trie: Prefix matching, autocomplete
- [ ] KMP: Pattern matching, repeated substring
- [ ] Backtracking: Combinations, word search
- [ ] Dynamic Programming: LCS, edit distance

---

**Happy Coding! 🎯**

