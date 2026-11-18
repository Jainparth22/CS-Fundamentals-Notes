# Dynamic Programming

> **Core idea:** Break problem into overlapping subproblems. Memoize (top-down) or build a table (bottom-up). Every DP solution starts with identifying the recurrence.

**3-step process:**
1. Define state: `dp[i]` means what?
2. Write recurrence: `dp[i] = f(dp[i-1], dp[i-2], ...)`
3. Identify base cases

---

## 1. 1D DP — Linear

### Fibonacci / Climbing Stairs
```
dp[i] = dp[i-1] + dp[i-2]
dp[0] = 1, dp[1] = 1
```

### House Robber
```
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
State: max money robbing up to house i
```
```cpp
int rob(vector<int>& nums) {
    int prev2 = 0, prev1 = 0;
    for (int n : nums) {
        int cur = max(prev1, prev2 + n);
        prev2 = prev1; prev1 = cur;
    }
    return prev1;
}
```

---

## 2. Knapsack Patterns

### 0/1 Knapsack — take or leave each item
```cpp
// Can we make sum = target using subset of nums?
// dp[j] = true if subset sum j is achievable
vector<bool> dp(target+1, false);
dp[0] = true;
for (int n : nums)
    for (int j = target; j >= n; j--)  // iterate backwards (each item once)
        dp[j] = dp[j] || dp[j-n];
```

### Unbounded Knapsack — can use each item multiple times (Coin Change)
```cpp
// dp[j] = min coins to make amount j
vector<int> dp(amount+1, INT_MAX);
dp[0] = 0;
for (int j = 1; j <= amount; j++)
    for (int coin : coins)
        if (coin <= j && dp[j-coin] != INT_MAX)
            dp[j] = min(dp[j], dp[j-coin] + 1);
```

**Key difference:** 0/1 iterates coins in outer loop, Unbounded iterates amounts in outer loop (or iterates backwards vs forwards).

---

## 3. LCS Family — 2D DP

### Longest Common Subsequence
```
dp[i][j] = LCS of text1[0..i-1] and text2[0..j-1]
If text1[i-1] == text2[j-1]: dp[i][j] = dp[i-1][j-1] + 1
Else: dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```

```cpp
int lcs(string& a, string& b) {
    int m = a.size(), n = b.size();
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = (a[i-1] == b[j-1]) ? dp[i-1][j-1]+1
                                            : max(dp[i-1][j], dp[i][j-1]);
    return dp[m][n];
}
```

### Edit Distance (Levenshtein)
```
dp[i][j] = min edits to convert word1[0..i-1] to word2[0..j-1]
If word1[i-1] == word2[j-1]: dp[i][j] = dp[i-1][j-1]
Else: dp[i][j] = 1 + min(dp[i-1][j],    // delete
                          dp[i][j-1],    // insert
                          dp[i-1][j-1])  // replace
```

---

## 4. LIS — Longest Increasing Subsequence

### O(n²) DP
```cpp
// dp[i] = length of LIS ending at index i
vector<int> dp(n, 1);
for (int i = 1; i < n; i++)
    for (int j = 0; j < i; j++)
        if (nums[j] < nums[i]) dp[i] = max(dp[i], dp[j]+1);
return *max_element(dp.begin(), dp.end());
```

### O(n log n) with Patience Sorting
```cpp
vector<int> tails;  // tails[i] = smallest tail of IS of length i+1
for (int n : nums) {
    auto it = lower_bound(tails.begin(), tails.end(), n);
    if (it == tails.end()) tails.push_back(n);
    else *it = n;
}
return tails.size();
```

---

## 5. Interval DP — Burst Balloons

```
dp[l][r] = max coins from bursting all balloons in (l, r) exclusive
For each k as the LAST balloon to burst in [l+1, r-1]:
  dp[l][r] = max(dp[l][r], dp[l][k] + nums[l]*nums[k]*nums[r] + dp[k][r])
```

```cpp
int maxCoins(vector<int>& nums) {
    nums.insert(nums.begin(), 1); nums.push_back(1); // sentinels
    int n = nums.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int len = 2; len < n; len++) {
        for (int l = 0; l < n-len; l++) {
            int r = l + len;
            for (int k = l+1; k < r; k++) {
                dp[l][r] = max(dp[l][r],
                    dp[l][k] + nums[l]*nums[k]*nums[r] + dp[k][r]);
            }
        }
    }
    return dp[0][n-1];
}
```

---

## 6. State Machine DP — Stock with Cooldown

```
hold[i]  = max profit when holding stock after day i
sold[i]  = max profit right after selling on day i
rest[i]  = max profit when in cooldown (day after selling)

hold[i] = max(hold[i-1], rest[i-1] - prices[i])  // buy today or hold
sold[i] = hold[i-1] + prices[i]                    // sell today
rest[i] = max(rest[i-1], sold[i-1])                // skip or was sold yesterday
```

---

## 7. Practice Problems

| Problem | Pattern | LC # |
|---------|---------|------|
| Climbing Stairs | 1D Fib | 70 |
| House Robber | 1D linear DP | 198 |
| Coin Change | Unbounded knapsack | 322 |
| Partition Equal Subset Sum | 0/1 knapsack | 416 |
| Longest Common Subsequence | 2D LCS | 1143 |
| Edit Distance | 2D edit | 72 |
| Longest Increasing Subsequence | LIS O(n log n) | 300 |
| Burst Balloons | Interval DP | 312 |
| Best Time to Buy/Sell with Cooldown | State machine | 309 |
| Regular Expression Matching | 2D DP | 10 |
