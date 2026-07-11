# 🔤 Tries (Prefix Trees) — FAANG Masterclass

> **"How does Google Search know what you're going to type after you enter just 3 letters? Welcome to the Trie."**

---

## 📑 Table of Contents
1. [What is a Trie?](#1-what-is-a-trie)
2. [TrieNode Structure (Array vs Map)](#2-trienode-structure)
3. [Insert Operation — Full Dry Run](#3-insert-operation)
4. [Search & Prefix Search — Full Dry Run](#4-search--prefix-search)
5. [Delete Operation — Full Code & Dry Run](#5-delete-operation)
6. [Autocomplete / Find All Words with Prefix](#6-autocomplete)
7. [Count Words & Count Prefix](#7-count-words--count-prefix)
8. [Trie vs Hash Map](#8-trie-vs-hash-map)
9. [Problem: Implement Trie (LC 208) — Complete Class](#9-implement-trie)
10. [Problem: Design Add and Search Words (LC 211)](#10-add-and-search-words)
11. [Problem: Word Search II (LC 212) — Full Dry Run](#11-word-search-ii)
12. [Problem: Replace Words (LC 648)](#12-replace-words)
13. [Problem: Longest Word in Dictionary (LC 720)](#13-longest-word-in-dictionary)
14. [XOR Trie — Maximum XOR of Two Numbers](#14-xor-trie)
15. [When to Use a Trie](#15-when-to-use-a-trie)
16. [Top 10 Must-Do Trie Questions](#16-top-10-must-do-trie-questions)

---

## 1. What is a Trie?

A Trie (pronounced "Try") is a special tree used to store strings where each node represents a **single character**. The **path** from root to a node spells out a string.

```
Storing words: "apple", "app", "ape", "bat", "ball"

             root
            /    \
           a      b
          /        \
         p          a
        / \          \
       p   e(✓)      t(✓)
      /               |
     l                l(✓)
    /
   e(✓)

(✓) = marks end of a complete word

Key insight: shared PREFIXES share the same path!
  "apple" and "app" share: a → p → p
  This saves memory compared to storing each word separately.
```

### Trie Properties
- **Time: O(L)** for insert, search, delete (where L = word length)
- **Space: O(TOTAL_CHARS)** across all words
- Each node has up to **26 children** (for a-z)
- A boolean `isEndOfWord` marks complete words

---

## 2. TrieNode Structure

### Array-based (Fast — use in interviews!)
```java
class TrieNode {
    TrieNode[] children;   // 26 slots for a-z
    boolean isEndOfWord;   // Does a word END at this node?
    
    public TrieNode() {
        children = new TrieNode[26];  // children[0]='a', children[1]='b', ...
        isEndOfWord = false;
    }
}
```

### Map-based (Flexible — supports any characters)
```java
class TrieNode {
    Map<Character, TrieNode> children;
    boolean isEndOfWord;
    
    public TrieNode() {
        children = new HashMap<>();
        isEndOfWord = false;
    }
}
```

| Feature | Array-based | Map-based |
|:---|:---|:---|
| Lookup speed | O(1) direct index | O(1) HashMap (higher constant) |
| Memory | 26 pointers per node (many null) | Only stores existing children |
| Character set | Fixed (a-z only) | Any character (unicode, etc.) |
| Use in interviews? | **Yes ✅ (preferred)** | When charset is large/unknown |

---

## 3. Insert Operation

To insert a word like `"apple"`:
1. Start at the Root.
2. For each character, check if the child exists. If not, create it.
3. Move to the child node.
4. After the last character, set `isEndOfWord = true`.

```java
public void insert(String word) {
    TrieNode curr = root;
    for (char c : word.toCharArray()) {
        int index = c - 'a'; // Convert 'a'-'z' to 0-25
        if (curr.children[index] == null) {
            curr.children[index] = new TrieNode(); // Create new node
        }
        curr = curr.children[index]; // Move down
    }
    curr.isEndOfWord = true; // Mark the end
}
```

### 🧠 Insert Dry Run

```
═══ Insert "apple" ═══

Step 1: c='a', index=0
  root.children[0] is null → CREATE new node
  Move to children[0]
  root → [a]

Step 2: c='p', index=15
  [a].children[15] is null → CREATE new node
  root → [a] → [p]

Step 3: c='p', index=15
  [p].children[15] is null → CREATE
  root → [a] → [p] → [p]

Step 4: c='l', index=11
  CREATE
  root → [a] → [p] → [p] → [l]

Step 5: c='e', index=4
  CREATE, mark isEndOfWord = true ✓
  root → [a] → [p] → [p] → [l] → [e✓]


═══ Insert "app" ═══

Step 1: c='a' → root.children[0] EXISTS → just move
Step 2: c='p' → [a].children[15] EXISTS → just move
Step 3: c='p' → [p].children[15] EXISTS → just move
  Mark isEndOfWord = true ✓

  root → [a] → [p] → [p✓] → [l] → [e✓]
  
  "app" and "apple" now SHARE the path a→p→p! 🎯


═══ Insert "ape" ═══

Step 1: c='a' → EXISTS → move
Step 2: c='p' → EXISTS → move  
Step 3: c='e' → [p].children[4] is null → CREATE, mark end ✓

  Final Trie:
       root
        |
       [a]
        |
       [p]
      /   \
   [p✓]   [e✓]    ← "ape"
    |
   [l]
    |
   [e✓]            ← "apple"
```

---

## 4. Search & Prefix Search

### Exact Word Search
Traverse the path. If any character is missing → `false`. If path exists, check `isEndOfWord`.

### Prefix Search (startsWith)
Same traversal, but DON'T check `isEndOfWord`. Just need the path to exist.

```java
// Exact word search
public boolean search(String word) {
    TrieNode node = findNode(word);
    return node != null && node.isEndOfWord;  // Must be end of word!
}

// Prefix search
public boolean startsWith(String prefix) {
    return findNode(prefix) != null;  // Just needs to exist
}

// Helper: traverse to end of string
private TrieNode findNode(String str) {
    TrieNode curr = root;
    for (char c : str.toCharArray()) {
        int index = c - 'a';
        if (curr.children[index] == null) return null; // Path broke!
        curr = curr.children[index];
    }
    return curr;
}
```

### 🧠 Search Dry Run

```
Trie contains: "apple", "app", "ape"

═══ search("app") ═══
  root → [a](exists) → [p](exists) → [p✓](exists)
  Node exists AND isEndOfWord = true → return true ✅

═══ search("ap") ═══
  root → [a](exists) → [p](exists)
  Node exists BUT isEndOfWord = false → return false ❌
  ("ap" is a prefix but not a complete word)

═══ startsWith("ap") ═══
  root → [a](exists) → [p](exists)
  Node exists → return true ✅
  (Don't care about isEndOfWord for prefix search)

═══ search("application") ═══
  root → [a] → [p] → [p] → [l] → ... 'i' → null!
  Path broke at 'i' → return false ❌
```

---

## 5. Delete Operation

Deleting from a Trie is trickier — we need to clean up nodes that are no longer needed.

**Rules:**
1. If the word doesn't exist → do nothing.
2. Unmark `isEndOfWord`.
3. If the node has no children and is no longer end of any word → delete it (going bottom-up).

```java
public boolean delete(String word) {
    return deleteHelper(root, word, 0);
}

private boolean deleteHelper(TrieNode node, String word, int depth) {
    if (node == null) return false;
    
    // Reached end of word
    if (depth == word.length()) {
        if (!node.isEndOfWord) return false; // Word doesn't exist
        node.isEndOfWord = false; // Unmark
        return isEmpty(node); // Should this node be deleted?
    }
    
    int index = word.charAt(depth) - 'a';
    boolean shouldDeleteChild = deleteHelper(node.children[index], word, depth + 1);
    
    if (shouldDeleteChild) {
        node.children[index] = null; // Delete the child
        // Should THIS node also be deleted?
        return !node.isEndOfWord && isEmpty(node);
    }
    return false;
}

private boolean isEmpty(TrieNode node) {
    for (TrieNode child : node.children) {
        if (child != null) return false;
    }
    return true;
}
```

### 🧠 Delete Dry Run

```
Trie: "apple", "app"

═══ Delete "apple" ═══

Before:
  root → [a] → [p] → [p✓] → [l] → [e✓]

deleteHelper(root, "apple", depth=0):
  → deleteHelper([a], "apple", 1):
    → deleteHelper([p], "apple", 2):
      → deleteHelper([p✓], "apple", 3):
        → deleteHelper([l], "apple", 4):
          → deleteHelper([e✓], "apple", 5):  depth == length!
            Unmark isEndOfWord → [e] (no longer end)
            isEmpty([e])? Yes (no children) → return true (delete me)
          Delete [e], [l].children[4] = null
          isEmpty([l])? Yes → return true (delete me)
        Delete [l], [p✓].children[11] = null
        isEmpty([p✓])? Yes, BUT [p✓].isEndOfWord = true → return false (keep!)
      return false → keep [p✓] and everything above

After:
  root → [a] → [p] → [p✓]
  
  "apple" is deleted, "app" remains ✅
  Nodes [l] and [e] were cleaned up!
```

---

## 6. Autocomplete

Find **all words** that start with a given prefix.

```java
public List<String> autocomplete(String prefix) {
    List<String> results = new ArrayList<>();
    TrieNode node = findNode(prefix); // Navigate to end of prefix
    
    if (node == null) return results; // Prefix doesn't exist
    
    // DFS from this node to find all complete words
    dfs(node, new StringBuilder(prefix), results);
    return results;
}

private void dfs(TrieNode node, StringBuilder path, List<String> results) {
    if (node.isEndOfWord) {
        results.add(path.toString()); // Found a complete word!
    }
    
    // Try all 26 possible children
    for (int i = 0; i < 26; i++) {
        if (node.children[i] != null) {
            path.append((char) ('a' + i));       // Add character
            dfs(node.children[i], path, results); // Go deeper
            path.deleteCharAt(path.length() - 1); // Backtrack
        }
    }
}
```

### 🧠 Autocomplete Dry Run

```
Trie contains: "apple", "app", "ape", "application"

═══ autocomplete("ap") ═══

Navigate to "ap": root → [a] → [p]

DFS from [p]:
  → children[4] = 'e' → [e✓] → isEndOfWord → add "ape" ✅
  → children[15] = 'p' → [p✓] → isEndOfWord → add "app" ✅
    → children[11] = 'l' → [l]
      → children[4] = 'e' → [e✓] → add "apple" ✅
      → children[8] = 'i' → ...
        → ... → "application" → add "application" ✅

Result: ["ape", "app", "apple", "application"]
(Naturally in lexicographic order because we iterate a-z!)
```

---

## 7. Count Words & Count Prefix

Enhanced Trie that counts:
- How many words end at each node
- How many words pass through each node (prefix count)

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    int countWord = 0;    // How many words END here
    int countPrefix = 0;  // How many words PASS through here
}

class Trie {
    TrieNode root = new TrieNode();
    
    public void insert(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) {
                curr.children[idx] = new TrieNode();
            }
            curr = curr.children[idx];
            curr.countPrefix++;  // Every insert passes through here
        }
        curr.countWord++;  // Word ends here
    }
    
    public int countWordsEqualTo(String word) {
        TrieNode node = findNode(word);
        return node == null ? 0 : node.countWord;
    }
    
    public int countWordsStartingWith(String prefix) {
        TrieNode node = findNode(prefix);
        return node == null ? 0 : node.countPrefix;
    }
    
    public void delete(String word) {
        // Only delete if word exists
        if (countWordsEqualTo(word) == 0) return;
        
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            curr = curr.children[idx];
            curr.countPrefix--;
        }
        curr.countWord--;
    }
}
```

### 🧠 Dry Run
```
insert("apple"), insert("apple"), insert("app")

After all inserts:
  root → [a](prefix=3) → [p](prefix=3) → [p](prefix=3, word=1) → [l](prefix=2) → [e](prefix=2, word=2)
  
  "app": countWord = 1
  "apple": countWord = 2 (inserted twice!)

countWordsEqualTo("apple") → 2
countWordsStartingWith("app") → 3 (app, apple, apple)
countWordsStartingWith("b") → 0
```

---

## 8. Trie vs Hash Map

Why use a Trie when a `HashSet` can do `O(1)` word lookups?

| Feature | HashSet/HashMap | Trie |
|:---|:---|:---|
| Exact word search | O(L) hash computation | O(L) traversal |
| **Prefix search** | ❌ Cannot | ✅ O(L) natively |
| **Autocomplete** | ❌ Cannot | ✅ O(L + results) |
| Memory for similar words | Stores duplicates ("apple", "app", "application") | **Shares prefixes** (saves memory) |
| Lexicographic order | ❌ Need sort | ✅ Natural (DFS gives sorted order) |
| Wildcard search (`.`) | ❌ Cannot | ✅ DFS with branching |

**Use Trie when:** Prefix operations, autocomplete, wildcard, XOR queries.
**Use HashMap when:** Only exact match needed (simpler).

---

## 9. Implement Trie (LC 208) — Complete Class

```java
class Trie {
    private TrieNode root;
    
    private class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEndOfWord = false;
    }
    
    public Trie() {
        root = new TrieNode();
    }
    
    // O(L) — Insert word
    public void insert(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) {
                curr.children[idx] = new TrieNode();
            }
            curr = curr.children[idx];
        }
        curr.isEndOfWord = true;
    }
    
    // O(L) — Search exact word
    public boolean search(String word) {
        TrieNode node = findNode(word);
        return node != null && node.isEndOfWord;
    }
    
    // O(L) — Check if any word starts with prefix
    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }
    
    private TrieNode findNode(String str) {
        TrieNode curr = root;
        for (char c : str.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) return null;
            curr = curr.children[idx];
        }
        return curr;
    }
}
```

---

## 10. Add and Search Words (LC 211)

**Problem:** Add words and search with `.` as wildcard (matches any letter).

### 🧠 How to Think
When we hit a `.`, we **branch out** and try ALL 26 children (DFS/backtracking).

```java
class WordDictionary {
    private TrieNode root;
    
    private class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEndOfWord = false;
    }
    
    public WordDictionary() {
        root = new TrieNode();
    }
    
    public void addWord(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) {
                curr.children[idx] = new TrieNode();
            }
            curr = curr.children[idx];
        }
        curr.isEndOfWord = true;
    }
    
    public boolean search(String word) {
        return searchHelper(root, word, 0);
    }
    
    private boolean searchHelper(TrieNode node, String word, int depth) {
        if (node == null) return false;
        if (depth == word.length()) return node.isEndOfWord;
        
        char c = word.charAt(depth);
        
        if (c == '.') {
            // Wildcard: try ALL 26 children
            for (int i = 0; i < 26; i++) {
                if (searchHelper(node.children[i], word, depth + 1)) {
                    return true; // Found a match in any branch!
                }
            }
            return false;
        } else {
            // Normal character
            return searchHelper(node.children[c - 'a'], word, depth + 1);
        }
    }
}
```

### 🧠 Dry Run
```
addWord("bad"), addWord("dad"), addWord("mad")

Trie:
    root
   / | \
  b  d  m
  |  |  |
  a  a  a
  |  |  |
  d✓ d✓ d✓

search(".ad"):
  depth=0, c='.': try all children
    → b: searchHelper([b], ".ad", 1)
      depth=1, c='a': [b].children['a'] = [a] → go deeper
        depth=2, c='d': [a].children['d'] = [d✓] → isEndOfWord = true!
        return true ✅

search("b.."):
  depth=0, c='b': go to [b]
    depth=1, c='.': try all children of [b]
      → [a]: depth=2, c='.': try all children of [a]
        → [d✓]: depth=3 == length, isEndOfWord = true!
        return true ✅
```

---

## 11. Word Search II (LC 212) — Full Dry Run

**Problem:** Given a 2D board and a list of words, find ALL words that can be formed by sequentially adjacent cells (up/down/left/right). Each cell can only be used once per word.

### 🧠 How to Think
Brute force: DFS for each word → TLE!
**Master Trick:** Build a Trie of ALL target words. DFS on the grid while walking the Trie simultaneously. If Trie path is null → prune immediately!

```java
class Solution {
    public List<String> findWords(char[][] board, String[] words) {
        // Step 1: Build Trie from all words
        TrieNode root = new TrieNode();
        for (String word : words) {
            TrieNode curr = root;
            for (char c : word.toCharArray()) {
                int idx = c - 'a';
                if (curr.children[idx] == null) {
                    curr.children[idx] = new TrieNode();
                }
                curr = curr.children[idx];
            }
            curr.word = word; // Store complete word at leaf
        }
        
        // Step 2: DFS from every cell
        List<String> result = new ArrayList<>();
        int m = board.length, n = board[0].length;
        
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                dfs(board, i, j, root, result);
            }
        }
        return result;
    }
    
    private void dfs(char[][] board, int i, int j, TrieNode node, List<String> result) {
        // Boundary check
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length) return;
        
        char c = board[i][j];
        if (c == '#' || node.children[c - 'a'] == null) return; // Visited or no Trie path
        
        node = node.children[c - 'a']; // Move down Trie
        
        if (node.word != null) {
            result.add(node.word); // Found a word!
            node.word = null;     // De-duplicate (don't add same word twice)
        }
        
        board[i][j] = '#'; // Mark visited
        
        dfs(board, i + 1, j, node, result); // Down
        dfs(board, i - 1, j, node, result); // Up
        dfs(board, i, j + 1, node, result); // Right
        dfs(board, i, j - 1, node, result); // Left
        
        board[i][j] = c; // Unmark (backtrack)
    }
}

class TrieNode {
    TrieNode[] children = new TrieNode[26];
    String word = null; // Complete word stored at end nodes
}
```

### 🧠 Dry Run

```
board = [['o','a','a','n'],
         ['e','t','a','e'],
         ['i','h','k','r'],
         ['i','f','l','v']]

words = ["oath","pea","eat","rain"]

Step 1: Build Trie
    root
   / | \
  o  p  e  r
  |  |  |  |
  a  e  a  a
  |  |  |  |
  t  a✓ t✓ i
  |         |
  h✓        n✓

Step 2: DFS from each cell

Starting at (0,0)='o':
  Trie: root → 'o' → exists!
  Move to Trie node for 'o'. Check neighbors:
    (1,0)='e': Trie 'o'.children['e'] = null → PRUNE! ✂️
    (0,1)='a': Trie 'o'.children['a'] = exists!
      Move to Trie node for 'a'. Check neighbors:
        (1,1)='t': Trie 'a'.children['t'] = exists!
          Move to Trie node for 't'. Check neighbors:
            (1,0)='e': nope
            (2,1)='h': Trie 't'.children['h'] = exists!
              word = "oath" → FOUND! Add to result ✅
              
Starting at (1,2)='a':
  Eventually finds: e→a→t → "eat" ✅

"pea" — 'p' doesn't exist on board → never found ❌
"rain" — can't form with adjacent cells → not found ❌

Result: ["oath", "eat"] ✅
```

---

## 12. Replace Words (LC 648)

**Problem:** Given dictionary of roots and a sentence, replace words with their shortest root.
Example: roots = ["cat","bat","rat"], sentence = "the cattle was rattled by the battery"
→ "the cat was rat by the bat"

### 🧠 How to Think
Build Trie from roots. For each word in sentence, traverse Trie. If we hit `isEndOfWord`, that's the shortest root — use it!

```java
public String replaceWords(List<String> dictionary, String sentence) {
    // Build Trie from dictionary
    TrieNode root = new TrieNode();
    for (String word : dictionary) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) curr.children[idx] = new TrieNode();
            curr = curr.children[idx];
        }
        curr.isEndOfWord = true;
    }
    
    // Process each word in sentence
    StringBuilder result = new StringBuilder();
    for (String word : sentence.split(" ")) {
        if (result.length() > 0) result.append(" ");
        
        TrieNode curr = root;
        StringBuilder prefix = new StringBuilder();
        boolean found = false;
        
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) break; // No root matches
            curr = curr.children[idx];
            prefix.append(c);
            
            if (curr.isEndOfWord) { // Found shortest root!
                found = true;
                break;
            }
        }
        
        result.append(found ? prefix.toString() : word);
    }
    return result.toString();
}
```

---

## 13. Longest Word in Dictionary (LC 720)

**Problem:** Find the longest word where every prefix is also in the dictionary.

### 🧠 How to Think
Build Trie from all words. BFS/DFS to find the longest word where every node along the path is `isEndOfWord`.

```java
public String longestWord(String[] words) {
    // Build Trie
    TrieNode root = new TrieNode();
    for (String word : words) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) curr.children[idx] = new TrieNode();
            curr = curr.children[idx];
        }
        curr.isEndOfWord = true;
        curr.word = word;
    }
    
    // BFS: only follow paths where every node is isEndOfWord
    String result = "";
    Queue<TrieNode> queue = new LinkedList<>();
    queue.add(root);
    
    while (!queue.isEmpty()) {
        TrieNode node = queue.poll();
        for (int i = 25; i >= 0; i--) { // Reverse order for lexicographic smallest
            TrieNode child = node.children[i];
            if (child != null && child.isEndOfWord) {
                if (child.word.length() > result.length()) {
                    result = child.word;
                }
                queue.add(child);
            }
        }
    }
    return result;
}
```

---

## 14. XOR Trie — Maximum XOR of Two Numbers

**Problem (LC 421):** Given an array, find two numbers whose XOR is maximum.

### 🧠 How to Think
Build a Trie of binary representations (32 bits). For each number, greedily pick the opposite bit at each level to maximize XOR.

```java
class XORTrie {
    int[][] children;  // children[node][0/1]
    int cnt = 0;       // Node counter
    
    public XORTrie() {
        children = new int[3200001][2]; // Max nodes
        for (int[] row : children) Arrays.fill(row, -1);
    }
    
    public void insert(int num) {
        int node = 0;
        for (int i = 31; i >= 0; i--) { // Process bit by bit, MSB first
            int bit = (num >> i) & 1;
            if (children[node][bit] == -1) {
                children[node][bit] = ++cnt;
            }
            node = children[node][bit];
        }
    }
    
    public int maxXor(int num) {
        int node = 0;
        int xor = 0;
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            int wanted = 1 - bit; // We WANT the opposite bit (maximizes XOR)
            
            if (children[node][wanted] != -1) {
                xor |= (1 << i); // This bit contributes to XOR
                node = children[node][wanted];
            } else {
                node = children[node][bit]; // Take same bit (no choice)
            }
        }
        return xor;
    }
}

public int findMaximumXOR(int[] nums) {
    XORTrie trie = new XORTrie();
    int maxXor = 0;
    
    trie.insert(nums[0]);
    for (int i = 1; i < nums.length; i++) {
        maxXor = Math.max(maxXor, trie.maxXor(nums[i]));
        trie.insert(nums[i]);
    }
    return maxXor;
}
```

### 🧠 Dry Run (simplified to 4 bits)
```
nums = [3, 10, 5, 25, 2, 8]
Binary: 3=0011, 10=1010, 5=0101, 25=11001, 2=0010, 8=1000

Insert 3 (0011) into Trie:
  root → 0 → 0 → 1 → 1

maxXor(10 = 1010):
  bit=1, want 0 → exists → take 0, xor += 8
  bit=0, want 1 → nope → take 0, xor += 0
  bit=1, want 0 → nope → take 1, xor += 0
  bit=0, want 1 → exists → take 1, xor += 1
  XOR = 8+0+0+1 = 9  (3 XOR 10 = 9)

Continue for all numbers...
Maximum XOR = 28 (5 XOR 25 = 0101 XOR 11001)
```

---

## 15. When to Use a Trie

```
┌──────────────────────────────────────────────────────────────┐
│                TRIE PATTERN RECOGNITION                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  "Autocomplete / search suggestions"  → Trie + DFS           │
│  "Check if prefix exists"             → Trie.startsWith()    │
│  "Wildcard search (.at → bat, cat)"   → Trie + DFS branching│
│  "Find words in 2D grid (Boggle)"     → Trie + Grid DFS     │
│  "Replace words with shortest root"   → Trie prefix match   │
│  "Count words with prefix"            → Trie with counters  │
│  "Maximum XOR of two numbers"         → Binary Trie (XOR)   │
│  "Spell checker / dictionary"         → Trie                │
│  "Lexicographic ordering"             → Trie DFS = sorted   │
│                                                               │
│  DON'T use Trie when:                                        │
│  "Only exact match needed"            → Use HashSet instead  │
│  "Single string pattern matching"     → Use KMP/Rabin-Karp  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 16. Top 10 Must-Do Trie Questions

1. Implement Trie (LC 208) — Basic insert/search/prefix
2. Design Add and Search Words (LC 211) — Wildcard with `.`
3. Word Search II (LC 212) — Trie + DFS on grid (HARD)
4. Replace Words (LC 648) — Prefix replacement
5. Longest Word in Dictionary (LC 720) — Every prefix exists
6. Maximum XOR of Two Numbers (LC 421) — Binary Trie
7. Map Sum Pairs (LC 677) — Trie with sum values
8. Implement Trie II (LC 1804) — With count functions
9. Stream of Characters (LC 1032) — Trie + suffix matching
10. Palindrome Pairs (LC 336) — Trie (HARD)
