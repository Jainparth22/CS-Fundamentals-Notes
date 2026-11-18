# Backtracking

> **Core idea:** Incrementally build a solution; abandon ("backtrack") as soon as you detect a constraint violation. Almost always exponential time, but pruning makes it practical.

---

## 1. The Template

```python
def backtrack(state, choices):
    if is_complete(state):
        record(state)
        return
    for choice in choices:
        if is_valid(state, choice):
            make_choice(state, choice)      # modify state
            backtrack(state, next_choices)  # recurse
            undo_choice(state, choice)      # restore state
```

**Three questions to answer before coding:**
1. What is the **base case** (complete solution)?
2. What **choices** do I have at each step?
3. What makes a choice **invalid** (pruning)?

---

## 2. Subsets — O(2^n)

```cpp
vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> result;
    vector<int> current;

    function<void(int)> bt = [&](int start) {
        result.push_back(current);  // every prefix is a valid subset
        for (int i = start; i < (int)nums.size(); i++) {
            current.push_back(nums[i]);
            bt(i + 1);
            current.pop_back();
        }
    };

    bt(0);
    return result;
}
```

### Subsets II (with duplicates)
```cpp
sort(nums.begin(), nums.end());
function<void(int)> bt = [&](int start) {
    result.push_back(current);
    for (int i = start; i < (int)nums.size(); i++) {
        if (i > start && nums[i] == nums[i-1]) continue; // skip duplicate
        current.push_back(nums[i]);
        bt(i + 1);
        current.pop_back();
    }
};
```

---

## 3. Combinations / Combination Sum — O(2^n)

```cpp
// Combination Sum: pick any element any number of times (unbounded)
function<void(int, int)> bt = [&](int start, int remaining) {
    if (remaining == 0) { result.push_back(current); return; }
    if (remaining < 0) return;  // pruning
    for (int i = start; i < (int)candidates.size(); i++) {
        current.push_back(candidates[i]);
        bt(i, remaining - candidates[i]);  // i not i+1: allow reuse
        current.pop_back();
    }
};
```

---

## 4. Permutations — O(n!)

```cpp
// Permutations without duplicates
vector<bool> used(nums.size(), false);
function<void()> bt = [&]() {
    if ((int)current.size() == (int)nums.size()) {
        result.push_back(current); return;
    }
    for (int i = 0; i < (int)nums.size(); i++) {
        if (used[i]) continue;
        used[i] = true;
        current.push_back(nums[i]);
        bt();
        current.pop_back();
        used[i] = false;
    }
};

// Permutations II (with duplicates)
sort(nums.begin(), nums.end());
// Add: if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
```

---

## 5. N-Queens — O(n!)

```cpp
int totalNQueens(int n) {
    int count = 0;
    vector<bool> cols(n,false), diag1(2*n,false), diag2(2*n,false);

    function<void(int)> bt = [&](int row) {
        if (row == n) { count++; return; }
        for (int col = 0; col < n; col++) {
            if (cols[col] || diag1[row-col+n] || diag2[row+col]) continue;
            cols[col] = diag1[row-col+n] = diag2[row+col] = true;
            bt(row + 1);
            cols[col] = diag1[row-col+n] = diag2[row+col] = false;
        }
    };

    bt(0);
    return count;
}
```

**Conflict checks:** `cols[col]`, `diag1[row-col]` (top-left to bottom-right), `diag2[row+col]` (top-right to bottom-left)

---

## 6. Word Search (Grid DFS + Backtracking)

```cpp
bool dfs(vector<vector<char>>& board, string& word, int r, int c, int idx) {
    if (idx == (int)word.size()) return true;
    if (r<0||r>=(int)board.size()||c<0||c>=(int)board[0].size()) return false;
    if (board[r][c] != word[idx]) return false;

    char tmp = board[r][c];
    board[r][c] = '#';  // mark visited

    bool found = dfs(board,word,r+1,c,idx+1) || dfs(board,word,r-1,c,idx+1)
              || dfs(board,word,r,c+1,idx+1) || dfs(board,word,r,c-1,idx+1);

    board[r][c] = tmp;  // restore
    return found;
}
```

---

## 7. Pruning Strategies

| Strategy | Example |
|----------|---------|
| Sort first, skip duplicates | Subsets II, Combination Sum II |
| Bound check (remaining < 0) | Combination Sum |
| Constraint propagation | Sudoku (pre-compute invalid cells) |
| Count early termination | N-Queens stop early if column count exceeded |

---

## 8. Practice Problems

| Problem | Key Idea | LC # |
|---------|----------|------|
| Subsets | Pick/skip each element | 78 |
| Subsets II | Skip duplicates | 90 |
| Combination Sum | Allow reuse (i not i+1) | 39 |
| Permutations | Used[] array | 46 |
| N-Queens | Column + diagonal bitmasks | 51 |
| Word Search | Grid DFS + in-place marking | 79 |
| Palindrome Partitioning | Cut + check palindrome | 131 |
| Sudoku Solver | Propagate constraints | 37 |
| Generate Parentheses | Count open and close | 22 |
