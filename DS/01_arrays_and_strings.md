# Arrays and Strings

> **Core idea:** Contiguous memory → O(1) random access, O(n) insert/delete at arbitrary position.

---

## 1. Memory Layout

```
int arr[5] = {10, 20, 30, 40, 50};
Address: base+0  base+4  base+8  base+12  base+16
Value:     10      20      30      40       50
```

- `arr[i]` = `*(arr + i)` = value at address `base + i * sizeof(int)`
- **Cache locality** — elements sit next to each other in cache lines (64 bytes) → cache-friendly iteration

### C++ vs Python Internals
| | C++ `vector<int>` | Python `list` |
|---|---|---|
| Storage | Contiguous int block | Array of *pointers* to objects |
| `arr[i]` type | `int` | `PyObject*` |
| Append | Amortized O(1) (doubling) | Amortized O(1) (over-allocation) |
| Memory overhead | Low | Higher (pointer + object header) |

---

## 2. Prefix Sum

Build a prefix array so any subarray sum is O(1) after O(n) preprocessing.

```
prefix[0] = 0
prefix[i] = prefix[i-1] + arr[i-1]
sum(l, r) = prefix[r+1] - prefix[l]   (0-indexed, inclusive)
```

### C++ Implementation
```cpp
vector<int> buildPrefix(const vector<int>& arr) {
    int n = arr.size();
    vector<int> pre(n + 1, 0);
    for (int i = 0; i < n; i++) pre[i+1] = pre[i] + arr[i];
    return pre;
}
// sum from index l to r (inclusive)
int rangeSum(const vector<int>& pre, int l, int r) {
    return pre[r+1] - pre[l];
}
```

### Python Implementation
```python
def build_prefix(arr):
    pre = [0] * (len(arr) + 1)
    for i, x in enumerate(arr):
        pre[i+1] = pre[i] + x
    return pre

def range_sum(pre, l, r):
    return pre[r+1] - pre[l]
```

### Key Problems
- Subarray Sum Equals K (LC 560) — prefix + hash map
- Product of Array Except Self (LC 238) — prefix product + suffix product
- Range Sum Query 2D (LC 304) — 2D prefix

---

## 3. Sliding Window

Use when the problem asks for a contiguous subarray/substring satisfying some condition.

### Fixed-size Window
```cpp
int windowSum = 0;
for (int i = 0; i < k; i++) windowSum += arr[i];
int maxSum = windowSum;
for (int i = k; i < n; i++) {
    windowSum += arr[i] - arr[i-k];
    maxSum = max(maxSum, windowSum);
}
```

### Variable-size Window (Two Pointers)
```cpp
int left = 0, result = 0;
unordered_map<char,int> freq;
for (int right = 0; right < n; right++) {
    // expand window: include s[right]
    freq[s[right]]++;
    // shrink window while invariant violated
    while (windowIsInvalid(freq)) {
        freq[s[left]]--;
        if (freq[s[left]] == 0) freq.erase(s[left]);
        left++;
    }
    result = max(result, right - left + 1);
}
```

### Template Decision
| Condition | Use |
|-----------|-----|
| Window size fixed | Fixed window |
| "longest subarray with at most K distinct" | Variable window, shrink when violated |
| "minimum window containing all chars" | Variable window, shrink when valid |

---

## 4. Kadane's Algorithm — Maximum Subarray

```
dp[i] = max(nums[i], dp[i-1] + nums[i])
       = "best subarray ENDING at i"
Answer = max over all i
```

```cpp
int maxSubArray(vector<int>& nums) {
    int cur = nums[0], best = nums[0];
    for (int i = 1; i < (int)nums.size(); i++) {
        cur = max(nums[i], cur + nums[i]);
        best = max(best, cur);
    }
    return best;
}
```

**Why it works:** If `cur` becomes negative, starting fresh at `nums[i]` is always better.

---

## 5. String Techniques

### Hashing / Anagram Detection
```python
from collections import Counter

def is_anagram(s: str, t: str) -> bool:
    return Counter(s) == Counter(t)
```

### Two-Pointer on Palindrome
```cpp
bool isPalindrome(const string& s) {
    int l = 0, r = s.size() - 1;
    while (l < r) {
        if (s[l] != s[r]) return false;
        l++; r--;
    }
    return true;
}
```

### KMP — Pattern Matching O(n+m)
```cpp
vector<int> buildLPS(const string& pat) {
    int m = pat.size();
    vector<int> lps(m, 0);
    int len = 0, i = 1;
    while (i < m) {
        if (pat[i] == pat[len]) { lps[i++] = ++len; }
        else if (len) len = lps[len-1];
        else lps[i++] = 0;
    }
    return lps;
}
```

---

## 6. Complexity Cheat Sheet

| Operation | Array | Dynamic Array |
|-----------|-------|---------------|
| Access `arr[i]` | O(1) | O(1) |
| Search (unsorted) | O(n) | O(n) |
| Search (sorted) | O(log n) | O(log n) |
| Insert at end | O(1)* | O(1) amortized |
| Insert at index i | O(n) | O(n) |
| Delete at index i | O(n) | O(n) |

---

## 7. Practice Problems

| Problem | Pattern | LC # |
|---------|---------|------|
| Two Sum | Hash map | 1 |
| Best Time to Buy/Sell | Greedy + one pass | 121 |
| Contains Duplicate | Hash set | 217 |
| Maximum Subarray | Kadane's | 53 |
| Product of Array Except Self | Prefix + suffix | 238 |
| Maximum Product Subarray | DP (track min+max) | 152 |
| Find Minimum in Rotated Sorted Array | Binary search | 153 |
| Container With Most Water | Two pointers | 11 |
| 3Sum | Sort + two pointers | 15 |
| Trapping Rain Water | Prefix max + suffix max | 42 |
