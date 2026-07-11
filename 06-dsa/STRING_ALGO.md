# 🔤 String Algorithms — FAANG Masterclass

> **"String pattern matching is where brute force O(N×M) becomes O(N+M) — and that's exactly what interviewers want to see."**

---

## 📑 Table of Contents
1. [String Basics & Common Operations](#1-string-basics)
2. [Rabin-Karp (Rolling Hash)](#2-rabin-karp)
3. [KMP Algorithm (Knuth-Morris-Pratt)](#3-kmp-algorithm)
4. [Z-Algorithm](#4-z-algorithm)
5. [Manacher's Algorithm (Longest Palindromic Substring)](#5-manachers-algorithm)
6. [Longest Palindromic Substring (LC 5) — Expand Around Center](#6-longest-palindromic-substring)
7. [Longest Common Prefix (LC 14)](#7-longest-common-prefix)
8. [Group Anagrams (LC 49)](#8-group-anagrams)
9. [Valid Anagram (LC 242)](#9-valid-anagram)
10. [Minimum Window Substring (LC 76)](#10-minimum-window-substring)
11. [Longest Substring Without Repeating (LC 3)](#11-longest-substring-without-repeating)
12. [String to Integer (atoi) (LC 8)](#12-string-to-integer)
13. [Pattern Recognition Cheat Sheet](#13-pattern-recognition)
14. [Top 15 Must-Do Questions](#14-top-15-must-do)

---

## 1. String Basics

### Java String Essentials for Interviews
```java
// String is IMMUTABLE in Java!
String s = "hello";

// Common operations
s.length();                    // 5
s.charAt(0);                   // 'h'
s.substring(1, 3);             // "el" (start inclusive, end exclusive)
s.indexOf("ll");               // 2
s.contains("ell");             // true
s.equals("hello");             // true (USE THIS, not ==)
s.compareTo("world");          // Negative (lexicographic comparison)
s.toCharArray();               // char[] {'h','e','l','l','o'}

// StringBuilder for mutable strings (MUCH faster than string concatenation)
StringBuilder sb = new StringBuilder();
sb.append("hello");
sb.insert(0, "say ");          // "say hello"
sb.deleteCharAt(sb.length()-1);
sb.reverse();
sb.toString();                 // Convert back to String

// Character utilities
Character.isLetter('a');       // true
Character.isDigit('5');        // true
Character.toLowerCase('A');    // 'a'
```

### Frequency Counting (CRITICAL for anagram/window problems)
```java
// Method 1: int[26] array (only lowercase letters — fastest!)
int[] freq = new int[26];
for (char c : s.toCharArray()) freq[c - 'a']++;

// Method 2: HashMap (any characters)
Map<Character, Integer> map = new HashMap<>();
for (char c : s.toCharArray()) map.merge(c, 1, Integer::sum);
```

---

## 2. Rabin-Karp (Rolling Hash)

**What:** Pattern matching using **hashing**. Instead of comparing characters one by one, compare hash values. If hashes match, verify with actual comparison.

**When:** Multiple pattern search, plagiarism detection, find repeated substrings.

**Time:** O(N+M) average, O(N×M) worst case (hash collisions).

```java
public int rabinKarp(String text, String pattern) {
    int n = text.length(), m = pattern.length();
    if (m > n) return -1;
    
    int BASE = 26, MOD = 1_000_000_007;
    long patHash = 0, txtHash = 0, power = 1;
    
    // Calculate hash of pattern and first window
    for (int i = 0; i < m; i++) {
        patHash = (patHash * BASE + pattern.charAt(i)) % MOD;
        txtHash = (txtHash * BASE + text.charAt(i)) % MOD;
        if (i > 0) power = (power * BASE) % MOD;
    }
    
    // Slide the window
    for (int i = 0; i <= n - m; i++) {
        if (patHash == txtHash) {
            // Hash match → verify characters (avoid false positive)
            if (text.substring(i, i + m).equals(pattern)) return i;
        }
        
        // Roll the hash: remove first char, add next char
        if (i < n - m) {
            txtHash = (txtHash - text.charAt(i) * power % MOD + MOD) % MOD;
            txtHash = (txtHash * BASE + text.charAt(i + m)) % MOD;
        }
    }
    return -1; // Not found
}
```

### 🧠 Dry Run
```
text = "AABCAAB", pattern = "CAA"
BASE = 26, window size = 3

Pattern hash: C*26² + A*26 + A = 2*676 + 0 + 0 = 1352

Window "AAB": hash = 0*676 + 0*26 + 1 = 1         ≠ 1352
Window "ABC": Remove A, add C → rolling hash       ≠ 1352
Window "BCA": rolling hash                          ≠ 1352
Window "CAA": rolling hash = 1352                   == 1352 → VERIFY → MATCH! ✅

Return index 3
```

---

## 3. KMP Algorithm (Knuth-Morris-Pratt)

**What:** Pattern matching in O(N+M) guaranteed. Never goes backward in the text!

**Key Idea:** Precompute a **failure function (LPS array)** that tells us: "If there's a mismatch at position `j`, where should we try next in the pattern?"

### Step 1: Build the LPS (Longest Prefix Suffix) Array
```
LPS[i] = length of the longest PROPER prefix of pattern[0..i] 
         that is also a suffix of pattern[0..i].

Pattern: A B C A B D
LPS:     [0, 0, 0, 1, 2, 0]

  "A"       → no proper prefix=suffix → 0
  "AB"      → no match → 0
  "ABC"     → no match → 0
  "ABCA"    → "A" is both prefix and suffix → 1
  "ABCAB"   → "AB" is both prefix and suffix → 2
  "ABCABD"  → no match → 0
```

### Step 2: Search using LPS
```java
public int kmpSearch(String text, String pattern) {
    int n = text.length(), m = pattern.length();
    int[] lps = buildLPS(pattern);
    
    int i = 0, j = 0; // i = text pointer, j = pattern pointer
    
    while (i < n) {
        if (text.charAt(i) == pattern.charAt(j)) {
            i++;
            j++;
            if (j == m) return i - m; // Found match!
        } else {
            if (j > 0) {
                j = lps[j - 1]; // Don't go back in text! Jump in pattern.
            } else {
                i++;
            }
        }
    }
    return -1;
}

private int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    int len = 0; // Length of previous longest prefix suffix
    int i = 1;
    
    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len > 0) {
                len = lps[len - 1]; // Fall back (don't increment i!)
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
    return lps;
}
```

### 🧠 KMP Full Dry Run

```
Text:    A B A B C A B C A B A B D
Pattern: A B A B D
LPS:    [0, 0, 1, 2, 0]

i=0,j=0: A==A → match, i=1,j=1
i=1,j=1: B==B → match, i=2,j=2
i=2,j=2: A==A → match, i=3,j=3
i=3,j=3: B==B → match, i=4,j=4
i=4,j=4: C≠D → MISMATCH!
  j>0, so j = lps[4-1] = lps[3] = 2
  (We know first 2 chars of pattern already match! Skip them!)

i=4,j=2: C≠A → MISMATCH!
  j>0, so j = lps[2-1] = lps[1] = 0

i=4,j=0: C≠A → j==0, so i=5

i=5,j=0: A==A → match, i=6,j=1
i=6,j=1: B==B → match, i=7,j=2
i=7,j=2: C≠A → j = lps[1] = 0
i=7,j=0: C≠A → i=8

i=8,j=0: A==A → match, i=9,j=1
i=9,j=1: B==B → match, i=10,j=2
i=10,j=2: A==A → match, i=11,j=3
i=11,j=3: B==B → match, i=12,j=4
i=12,j=4: D==D → match, j=5=m → FOUND at index 12-5=8!

Pattern "ABABD" found at index 8 ✅
```

---

## 4. Z-Algorithm

**What:** For string `S`, Z[i] = length of the longest substring starting at `i` that matches a prefix of `S`.

**Use:** Pattern matching by concatenating `pattern + "$" + text`, then finding where Z[i] = pattern.length.

```java
public int[] zFunction(String s) {
    int n = s.length();
    int[] z = new int[n];
    int l = 0, r = 0;
    
    for (int i = 1; i < n; i++) {
        if (i < r) {
            z[i] = Math.min(r - i, z[i - l]);
        }
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) {
            z[i]++;
        }
        if (i + z[i] > r) {
            l = i;
            r = i + z[i];
        }
    }
    return z;
}

// Pattern matching using Z-algorithm
public int zSearch(String text, String pattern) {
    String combined = pattern + "$" + text;
    int[] z = zFunction(combined);
    int m = pattern.length();
    
    for (int i = m + 1; i < combined.length(); i++) {
        if (z[i] == m) return i - m - 1; // Found at index (i - m - 1) in text
    }
    return -1;
}
```

### 🧠 Z-Array Dry Run
```
S = "aabxaab"

Z[0] = undefined (convention: 0 or length)
Z[1]: compare s[1]='a' with s[0]='a' → match!
      compare s[2]='b' with s[1]='a' → no → Z[1] = 1
Z[2]: compare s[2]='b' with s[0]='a' → no → Z[2] = 0
Z[3]: compare s[3]='x' with s[0]='a' → no → Z[3] = 0
Z[4]: compare s[4]='a' with s[0]='a' → match
      compare s[5]='a' with s[1]='a' → match
      compare s[6]='b' with s[2]='b' → match → Z[4] = 3
Z[5]: inside Z-box, Z[5] = min(r-5, Z[5-4]) = min(2, Z[1]) = min(2,1) = 1
Z[6]: inside Z-box, Z[6] = min(r-6, Z[6-4]) = min(1, Z[2]) = 0

Z = [0, 1, 0, 0, 3, 1, 0]
```

---

## 5. Manacher's Algorithm (Longest Palindromic Substring)

**What:** Finds the longest palindromic substring in O(N). Uses the trick of expanding from centers while leveraging previously computed palindromes.

```java
public String longestPalindrome(String s) {
    // Transform: "abc" → "^#a#b#c#$"
    StringBuilder t = new StringBuilder("^");
    for (char c : s.toCharArray()) {
        t.append('#').append(c);
    }
    t.append("#$");
    
    int n = t.length();
    int[] p = new int[n]; // p[i] = radius of palindrome centered at i
    int c = 0, r = 0;     // Center and right boundary of rightmost palindrome
    
    for (int i = 1; i < n - 1; i++) {
        int mirror = 2 * c - i;
        
        if (i < r) {
            p[i] = Math.min(r - i, p[mirror]);
        }
        
        // Try to expand
        while (t.charAt(i + p[i] + 1) == t.charAt(i - p[i] - 1)) {
            p[i]++;
        }
        
        // Update center if expanded past right boundary
        if (i + p[i] > r) {
            c = i;
            r = i + p[i];
        }
    }
    
    // Find max in p
    int maxLen = 0, centerIdx = 0;
    for (int i = 1; i < n - 1; i++) {
        if (p[i] > maxLen) {
            maxLen = p[i];
            centerIdx = i;
        }
    }
    
    int start = (centerIdx - maxLen) / 2;
    return s.substring(start, start + maxLen);
}
```

---

## 6. Longest Palindromic Substring (LC 5) — Expand Around Center

For interviews, this O(N²) approach is usually sufficient and much easier to code:

```java
public String longestPalindrome(String s) {
    int start = 0, maxLen = 0;
    
    for (int i = 0; i < s.length(); i++) {
        // Odd length palindromes (center = single char)
        int len1 = expand(s, i, i);
        // Even length palindromes (center = between two chars)
        int len2 = expand(s, i, i + 1);
        
        int len = Math.max(len1, len2);
        if (len > maxLen) {
            maxLen = len;
            start = i - (len - 1) / 2;
        }
    }
    return s.substring(start, start + maxLen);
}

private int expand(String s, int left, int right) {
    while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
        left--;
        right++;
    }
    return right - left - 1;
}
```

### 🧠 Dry Run
```
s = "babad"

i=0 ('b'): odd expand(0,0) → "b" len=1. even expand(0,1) → 'b'≠'a' len=0. max=1
i=1 ('a'): odd expand(1,1) → 'b'=='b'? NO → len=1. even expand(1,2) → 'a'≠'b' len=0.
  Wait: expand(1,1) → check s[0]='b'==s[2]='b' → YES → len=3 → "bab"!
  maxLen=3, start=0

i=2 ('b'): odd expand(2,2) → s[1]='a'==s[3]='a' → YES → s[0]='b'==s[4]='d'? NO → len=3 → "aba"
  Same length, keep first.

i=3 ('a'): odd len=1, even len=0
i=4 ('d'): odd len=1, even len=0

Answer: "bab" (or "aba") ✅
```

---

## 7. Longest Common Prefix (LC 14)

```java
public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        while (strs[i].indexOf(prefix) != 0) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.isEmpty()) return "";
        }
    }
    return prefix;
}
```

---

## 8. Group Anagrams (LC 49)

**Problem:** Group strings that are anagrams of each other.

`["eat","tea","tan","ate","nat","bat"]` → `[["eat","tea","ate"], ["tan","nat"], ["bat"]]`

### 🧠 How to Think
Two strings are anagrams if they have the same character frequency. Sort each string → use as key.

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars); // Sorted = canonical form
        
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    
    return new ArrayList<>(map.values());
}

// Optimized: Use frequency array as key (avoids O(k log k) sort)
public List<List<String>> groupAnagramsOptimized(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    
    for (String s : strs) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;
        String key = Arrays.toString(freq); // "[1,0,0,...,1,0]"
        
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
```

---

## 9. Valid Anagram (LC 242)

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    
    int[] freq = new int[26];
    for (int i = 0; i < s.length(); i++) {
        freq[s.charAt(i) - 'a']++;
        freq[t.charAt(i) - 'a']--;
    }
    
    for (int f : freq) {
        if (f != 0) return false;
    }
    return true;
}
```

---

## 10. Minimum Window Substring (LC 76)

**Problem:** Find the smallest substring of `s` that contains all characters of `t`.

`s = "ADOBECODEBANC", t = "ABC"` → `"BANC"`

### 🧠 How to Think
**Sliding Window** with two pointers. Expand right to include characters, shrink left to minimize window.

```java
public String minWindow(String s, String t) {
    int[] need = new int[128]; // Frequency of characters needed
    for (char c : t.toCharArray()) need[c]++;
    
    int left = 0, minLen = Integer.MAX_VALUE, minStart = 0;
    int count = t.length(); // Characters remaining to match
    
    for (int right = 0; right < s.length(); right++) {
        // Expand: add s[right] to window
        if (need[s.charAt(right)] > 0) count--; // Found a needed character
        need[s.charAt(right)]--;
        
        // Shrink: while window contains all of t
        while (count == 0) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minStart = left;
            }
            
            need[s.charAt(left)]++;
            if (need[s.charAt(left)] > 0) count++; // Lost a needed character
            left++;
        }
    }
    
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minStart, minStart + minLen);
}
```

### 🧠 Dry Run
```
s = "ADOBECODEBANC", t = "ABC"
need: {A:1, B:1, C:1}, count=3

right=0 'A': need[A]-- → {A:0}, count=2. Window: "A"
right=1 'D': need[D]-- → {D:-1}. count=2
right=2 'O': count=2
right=3 'B': need[B]-- → {B:0}, count=1. Window: "ADOB"
right=4 'E': count=1
right=5 'C': need[C]-- → {C:0}, count=0!
  → Window "ADOBEC" contains all! len=6, minStart=0
  Shrink: left=0 'A': need[A]++ → {A:1}, count=1. left=1
  
right=6..9: count stays ≥1

right=10 'A': need[A]-- → {A:0}, count=0!
  → Window "CODEBA" contains all! (wait, let me track better)
  
... (continuing the shrink/expand)

right=12 'C': count=0 → window "BANC" len=4. 
  This is shorter! minLen=4, minStart=9

Answer: "BANC" ✅
```

---

## 11. Longest Substring Without Repeating (LC 3)

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1; // Jump left past the duplicate
        }
        
        lastSeen.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

---

## 12. String to Integer (atoi) (LC 8)

OA favorite — tests edge case handling!

```java
public int myAtoi(String s) {
    int i = 0, n = s.length();
    
    // 1. Skip whitespace
    while (i < n && s.charAt(i) == ' ') i++;
    if (i == n) return 0;
    
    // 2. Handle sign
    int sign = 1;
    if (s.charAt(i) == '-' || s.charAt(i) == '+') {
        sign = s.charAt(i) == '-' ? -1 : 1;
        i++;
    }
    
    // 3. Convert digits (handle overflow!)
    int result = 0;
    while (i < n && Character.isDigit(s.charAt(i))) {
        int digit = s.charAt(i) - '0';
        
        // Check overflow BEFORE adding
        if (result > (Integer.MAX_VALUE - digit) / 10) {
            return sign == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
        }
        
        result = result * 10 + digit;
        i++;
    }
    
    return result * sign;
}
```

---

## 13. Pattern Recognition

```
┌──────────────────────────────────────────────────────────────┐
│           STRING PATTERN RECOGNITION                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  "Find pattern in text"             → KMP (O(N+M) exact)    │
│  "Multiple pattern search"          → Rabin-Karp (hashing)  │
│  "Longest palindrome"               → Expand from center    │
│  "All palindromic substrings"       → Manacher's O(N)       │
│  "Prefix matching"                  → Z-Algorithm or Trie   │
│                                                               │
│  "Anagram check / grouping"         → Frequency array [26]  │
│  "Minimum window containing X"      → Sliding window        │
│  "Longest substring w/o repeating"  → Sliding window + map  │
│  "Longest common prefix"            → Vertical scan / Trie  │
│                                                               │
│  SLIDING WINDOW (2 pointers):                                 │
│  1. Expand RIGHT to satisfy condition                        │
│  2. Shrink LEFT to minimize/optimize                         │
│  3. Track window state with HashMap/array                    │
│                                                               │
│  COMMON TRICKS:                                               │
│  • Sorted string = anagram key                               │
│  • int[26] freq array beats HashMap for a-z                  │
│  • StringBuilder beats string concatenation                  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 14. Top 15 Must-Do Questions

**Pattern Matching:**
1. Implement strStr() / KMP (LC 28)
2. Repeated Substring Pattern (LC 459) — KMP/Z-algo
3. Shortest Palindrome (LC 214) — KMP

**Palindromes:**
4. Longest Palindromic Substring (LC 5)
5. Palindromic Substrings (LC 647)
6. Valid Palindrome (LC 125)

**Sliding Window:**
7. Longest Substring Without Repeating (LC 3)
8. Minimum Window Substring (LC 76)
9. Find All Anagrams in String (LC 438)
10. Longest Repeating Character Replacement (LC 424)

**Anagrams & Frequency:**
11. Valid Anagram (LC 242)
12. Group Anagrams (LC 49)

**Parsing & Manipulation:**
13. String to Integer (LC 8)
14. Decode String (LC 394) — Stack-based
15. Longest Common Prefix (LC 14)
