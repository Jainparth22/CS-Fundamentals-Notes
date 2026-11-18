# Tries (Prefix Trees)

> **Core idea:** Tree where each node represents a character. All words sharing a prefix share the same path from root. O(L) operations where L = word length.

---

## 1. When to Use

- Autocomplete / prefix search
- Dictionary lookup with prefix matching
- Word Search II (many words on a board)
- Bitwise XOR tricks (XOR Trie on binary representation)

---

## 2. Full C++ Implementation

```cpp
struct TrieNode {
    TrieNode* children[26];
    bool isEnd;
    int count; // optional: how many words end here

    TrieNode() : isEnd(false), count(0) {
        fill(children, children + 26, nullptr);
    }
};

class Trie {
    TrieNode* root;

public:
    Trie() { root = new TrieNode(); }

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int i = c - 'a';
            if (!node->children[i]) node->children[i] = new TrieNode();
            node = node->children[i];
        }
        node->isEnd = true;
        node->count++;
    }

    bool search(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int i = c - 'a';
            if (!node->children[i]) return false;
            node = node->children[i];
        }
        return node->isEnd;
    }

    bool startsWith(const string& prefix) {
        TrieNode* node = root;
        for (char c : prefix) {
            int i = c - 'a';
            if (!node->children[i]) return false;
            node = node->children[i];
        }
        return true;
    }

    // Delete a word (lazy: just clear isEnd)
    bool remove(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int i = c - 'a';
            if (!node->children[i]) return false;
            node = node->children[i];
        }
        if (!node->isEnd) return false;
        node->isEnd = false;
        return true;
    }
};
```

---

## 3. Full Python Implementation

```python
class TrieNode:
    def __init__(self):
        self.children: dict[str, 'TrieNode'] = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]
        node.is_end = True

    def search(self, word: str) -> bool:
        node = self.root
        for c in word:
            if c not in node.children:
                return False
            node = node.children[c]
        return node.is_end

    def starts_with(self, prefix: str) -> bool:
        node = self.root
        for c in prefix:
            if c not in node.children:
                return False
            node = node.children[c]
        return True
```

---

## 4. Word Search II (Trie + DFS Backtracking)

```cpp
// Build trie from word list, then DFS board
// When we reach a node where isEnd = true, we found a word
// Key optimization: prune trie nodes after finding a word (remove word to avoid duplicates)

void dfs(vector<vector<char>>& board, TrieNode* node, int r, int c,
         string& path, vector<string>& result) {
    if (r<0||r>=(int)board.size()||c<0||c>=(int)board[0].size()) return;
    char ch = board[r][c];
    if (ch == '#' || !node->children[ch-'a']) return;

    node = node->children[ch-'a'];
    path.push_back(ch);
    board[r][c] = '#';  // mark visited

    if (node->isEnd) {
        result.push_back(path);
        node->isEnd = false; // avoid duplicates
    }

    dfs(board, node, r+1, c, path, result);
    dfs(board, node, r-1, c, path, result);
    dfs(board, node, r, c+1, path, result);
    dfs(board, node, r, c-1, path, result);

    board[r][c] = ch;  // restore
    path.pop_back();
}
```

---

## 5. XOR Trie — Maximum XOR

Build a binary trie on 32-bit representations. For each number, greedily pick the opposite bit to maximize XOR.

```cpp
struct XORTrie {
    int children[2];
    XORTrie() { children[0] = children[1] = -1; }
};

vector<XORTrie> trie = {XORTrie()};

void insert(int num) {
    int node = 0;
    for (int bit = 31; bit >= 0; bit--) {
        int b = (num >> bit) & 1;
        if (trie[node].children[b] == -1) {
            trie[node].children[b] = trie.size();
            trie.push_back(XORTrie());
        }
        node = trie[node].children[b];
    }
}

int maxXOR(int num) {
    int node = 0, result = 0;
    for (int bit = 31; bit >= 0; bit--) {
        int b = (num >> bit) & 1;
        int want = 1 - b;  // prefer opposite bit
        if (trie[node].children[want] != -1) {
            result |= (1 << bit);
            node = trie[node].children[want];
        } else {
            node = trie[node].children[b];
        }
    }
    return result;
}
```

---

## 6. Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Insert | O(L) | O(L) per word |
| Search | O(L) | — |
| StartsWith | O(L) | — |
| Total space | — | O(ALPHABET × N × L) |

Where L = average word length, N = number of words.

---

## 7. Practice Problems

| Problem | Technique | LC # |
|---------|-----------|------|
| Implement Trie | Standard trie | 208 |
| Design Add and Search Words | Trie + DFS for '.' wildcard | 211 |
| Word Search II | Trie + board DFS | 212 |
| Replace Words | Trie prefix replacement | 648 |
| Maximum XOR of Two Numbers | XOR Trie | 421 |
