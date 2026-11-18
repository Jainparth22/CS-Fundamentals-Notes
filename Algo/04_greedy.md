# Greedy Algorithms

> **Core idea:** At each step, make the locally optimal choice and never look back. Correct only when the greedy choice property holds — prove it with an exchange argument.

---

## 1. The Exchange Argument (How to Prove Greedy Correct)

**Template:** "Suppose an optimal solution differs from our greedy solution. We can swap one greedy choice in and not make things worse. By induction, the greedy solution is optimal."

**Example — Activity Selection:**
- Greedy: pick activity with earliest END time
- Proof: If optimal picks activity A and greedy picks B (earlier end), swapping B for A leaves at least as many slots → greedy no worse

---

## 2. Interval Scheduling / Greedy on Intervals

### Maximum Non-Overlapping Intervals (Activity Selection)
```cpp
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(),
         [](auto& a, auto& b){ return a[1] < b[1]; }); // sort by END
    int count = 0, lastEnd = INT_MIN;
    for (auto& iv : intervals) {
        if (iv[0] >= lastEnd) lastEnd = iv[1]; // select this interval
        else count++;  // remove this overlapping interval
    }
    return count;
}
```

### Merge Intervals
```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    vector<vector<int>> res = {intervals[0]};
    for (int i = 1; i < (int)intervals.size(); i++) {
        if (intervals[i][0] <= res.back()[1])
            res.back()[1] = max(res.back()[1], intervals[i][1]);
        else res.push_back(intervals[i]);
    }
    return res;
}
```

---

## 3. Jump Game

```cpp
// Can you reach the last index?
bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i < (int)nums.size(); i++) {
        if (i > maxReach) return false;      // stuck
        maxReach = max(maxReach, i + nums[i]);
    }
    return true;
}

// Minimum jumps to reach end
int jump(vector<int>& nums) {
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < (int)nums.size()-1; i++) {
        farthest = max(farthest, i + nums[i]);
        if (i == curEnd) {   // must jump here
            jumps++;
            curEnd = farthest;
        }
    }
    return jumps;
}
```

---

## 4. Gas Station

```cpp
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int total = 0, tank = 0, start = 0;
    for (int i = 0; i < (int)gas.size(); i++) {
        int diff = gas[i] - cost[i];
        total += diff;
        tank  += diff;
        if (tank < 0) {    // can't reach i+1 from start
            start = i + 1; // reset start
            tank = 0;
        }
    }
    return total >= 0 ? start : -1;
}
```

**Why it works:** If total gas >= total cost, a solution exists. The greedy start point is valid because any prefix where tank goes negative cannot be part of the solution.

---

## 5. Partition Labels

```cpp
vector<int> partitionLabels(string s) {
    // Last occurrence of each character
    unordered_map<char, int> last;
    for (int i = 0; i < (int)s.size(); i++) last[s[i]] = i;

    vector<int> result;
    int start = 0, end = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        end = max(end, last[s[i]]);    // extend partition to include all occurrences
        if (i == end) {                 // reached end of current partition
            result.push_back(end - start + 1);
            start = i + 1;
        }
    }
    return result;
}
```

---

## 6. Fractional Knapsack

```cpp
// Maximize value with capacity W
// Sort items by value/weight ratio descending
double fractionalKnapsack(vector<pair<int,int>>& items, int W) {
    sort(items.begin(), items.end(), [](auto& a, auto& b){
        return (double)a.first/a.second > (double)b.first/b.second;
    }); // {value, weight}

    double total = 0;
    for (auto [v, w] : items) {
        if (W >= w) { total += v; W -= w; }
        else { total += (double)v * W / w; break; }
    }
    return total;
}
```

---

## 7. When NOT to Use Greedy

| Problem | Why Greedy Fails |
|---------|-----------------|
| 0/1 Knapsack | Taking highest ratio item may prevent optimal subset |
| Coin Change (non-canonical denominations) | e.g., coins {1,3,4}, target=6: greedy takes 4+1+1=3 coins, but 3+3=2 |
| Shortest Path with negative edges | Dijkstra (greedy) fails; use Bellman-Ford |

---

## 8. Practice Problems

| Problem | Greedy Choice | LC # |
|---------|---------------|------|
| Jump Game | Track maxReach | 55 |
| Jump Game II | Greedy level by level | 45 |
| Gas Station | Reset start on negative tank | 134 |
| Partition Labels | Extend to last occurrence | 763 |
| Non-overlapping Intervals | Earliest end time | 435 |
| Minimum Arrows for Balloons | Sort by end, one arrow at start of overlap | 452 |
| Task Scheduler | Max freq task determines cooldown | 621 |
| Valid Parenthesis String | Track lo/hi range of open counts | 678 |
