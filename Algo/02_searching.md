# Binary Search

> **Core idea:** On a sorted (or monotone-property) space, eliminate half the candidates each step. O(log n).

**The key to binary search:** Define the invariant. What is `lo`? What is `hi`? What does each mid check?

---

## 1. Standard Binary Search

```cpp
// Returns index of target, or -1
int binarySearch(vector<int>& arr, int target) {
    int lo = 0, hi = arr.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;  // avoid overflow vs (lo+hi)/2
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

---

## 2. Lower Bound & Upper Bound

```cpp
// lower_bound: first index where arr[i] >= target
int lowerBound(vector<int>& arr, int target) {
    int lo = 0, hi = arr.size();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;  // lo == hi, arr[lo] >= target (or lo = n if all < target)
}

// upper_bound: first index where arr[i] > target
int upperBound(vector<int>& arr, int target) {
    int lo = 0, hi = arr.size();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

// Count occurrences of target:
int count = upperBound(arr, target) - lowerBound(arr, target);
```

### C++ STL (use these in interviews)
```cpp
#include <algorithm>
auto it = lower_bound(arr.begin(), arr.end(), target);
auto it2 = upper_bound(arr.begin(), arr.end(), target);
int idx = it - arr.begin();     // index
bool found = (it != arr.end() && *it == target);
```

---

## 3. Binary Search on the Answer ("On Answer" Template)

Use when: "find the minimum/maximum X such that condition(X) is true."

```cpp
// Find minimum X in [lo, hi] where feasible(X) is true
// feasible must be monotone: once true, stays true as X increases
int bsOnAnswer(int lo, int hi) {
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (feasible(mid)) hi = mid;    // mid might be answer, search left
        else lo = mid + 1;              // mid too small, search right
    }
    return lo;  // lo == hi == minimum feasible X
}
```

**Examples:**
- Koko Eating Bananas: feasible(speed) = "can eat all piles in H hours"
- Capacity to Ship Packages: feasible(capacity) = "can ship all packages in D days"
- Aggressive Cows: feasible(dist) = "can place K cows with min distance >= dist"

---

## 4. Rotated Sorted Array

```cpp
int searchRotated(vector<int>& nums, int target) {
    int lo = 0, hi = nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        // Left half is sorted
        if (nums[lo] <= nums[mid]) {
            if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
            else lo = mid + 1;
        } else { // Right half is sorted
            if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
            else hi = mid - 1;
        }
    }
    return -1;
}
```

---

## 5. Find Peak Element

```cpp
int findPeakElement(vector<int>& nums) {
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[mid+1]) hi = mid;    // peak in left half (including mid)
        else lo = mid + 1;                          // ascending, peak in right half
    }
    return lo;
}
```

---

## 6. Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Infinite loop when `lo = mid` | Use `lo = mid + 1` or `hi = mid - 1` correctly |
| Integer overflow in `(lo + hi) / 2` | Use `lo + (hi - lo) / 2` |
| Off-by-one on `hi` | Decide: is `hi` inclusive or exclusive? Be consistent |
| Wrong condition for "first true" | Use `hi = mid` (not `mid-1`) when `mid` could be the answer |

---

## 7. Practice Problems

| Problem | Type | LC # |
|---------|------|------|
| Binary Search | Standard | 704 |
| Search a 2D Matrix | Flatten index | 74 |
| Koko Eating Bananas | On answer | 875 |
| Find Min in Rotated Sorted | Rotated | 153 |
| Search in Rotated Sorted | Rotated with target | 33 |
| Median of Two Sorted Arrays | Binary search on partition | 4 |
| Capacity to Ship Packages | On answer | 1011 |
| Split Array Largest Sum | On answer (minimize max) | 410 |
