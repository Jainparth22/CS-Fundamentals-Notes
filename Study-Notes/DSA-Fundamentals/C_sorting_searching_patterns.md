# DSA Notes C: Sorting, searching, array patterns (concepts 8 to 10)

Compiled notes, not yet assessed (M0). **Est.** = sourced, **Ex.** = my example.

---
## 8. Sorting
**ELI15.** Sorting is arranging a messy shelf. Smart methods split the shelf, sort the halves, and merge, instead of comparing every pair.

**Foundations.**
| Algorithm | Best | Average | Worst | Extra space | Stable | Idea |
|---|---|---|---|---|---|---|
| Bubble / insertion / selection | O(n) for insertion on sorted, else O(n^2) | O(n^2) | O(n^2) | O(1) | bubble, insertion yes; selection no | simple; insertion sort is fast on small or nearly sorted data |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n) | yes | divide, sort halves, merge |
| Quicksort | O(n log n) | O(n log n) | O(n^2) | O(log n) stack | no | partition around a pivot |
| Heap sort | O(n log n) | O(n log n) | O(n log n) | O(1) | no | build heap, pop max repeatedly |
| Counting sort | O(n+k) | O(n+k) | O(n+k) | O(n+k) | yes | count keys in range `k` |
| Radix sort | O(d(n+k)) | | | O(n+k) | yes | sort digit by digit with a stable sort |
**Stable** = equal keys keep their original order.

**Lower bound.** Any comparison-based sort needs Omega(n log n) comparisons: a decision tree must distinguish `n!` orderings, so depth >= `log2(n!)` = Theta(n log n) (standard result, **Ex.**).

**Library sorts (Est.).**
- Python `list.sort`/`sorted`: **Timsort, stable** (Python Sorting HOWTO), O(n log n) worst case and O(n) on already-sorted data (time-complexity page). `key=` function; `reverse=`.
- C++ `std::sort`: O(N log N) comparisons, **not stable** (cppreference; the worst-case bound was made mandatory by a defect report, older libraries may differ); `std::stable_sort` keeps order.

```python
def merge_sort(a):
    if len(a) <= 1: return a
    m = len(a)//2; L, R = merge_sort(a[:m]), merge_sort(a[m:])
    i = j = 0; out = []
    while i < len(L) and j < len(R):
        if L[i] <= R[j]: out.append(L[i]); i += 1      # <= keeps it stable
        else: out.append(R[j]); j += 1
    return out + L[i:] + R[j:]
```
```cpp
int partition(vector<int>& a,int lo,int hi){ int p=a[hi],i=lo; for(int j=lo;j<hi;++j) if(a[j]<p) swap(a[i++],a[j]); swap(a[i],a[hi]); return i; }
void quick(vector<int>& a,int lo,int hi){ if(lo>=hi) return; int p=partition(a,lo,hi); quick(a,lo,p-1); quick(a,p+1,hi); }
```
Quicksort fixes: random or median-of-three pivot, three-way partition for many duplicates, recurse on the smaller side first (bounds stack at O(log n)). Introsort (used by many `std::sort` implementations) switches to heap sort when depth is excessive (**Ex.**, common implementation).

**Interview / expert.** Google's prep PDF expects knowledge of at least one, ideally two, O(n log n) sorts (quicksort and merge sort named), when each is preferable, and that insertion or radix sort can beat them in special cases (**Est.**). Merge sort suits linked lists and external sorting; quicksort has better cache behaviour on arrays. Quickselect finds the k-th element in O(n) average (O(n^2) worst). Know counting sort limits (small integer range). Sorting by multiple keys: sort by the least significant key first with a stable sort, or use a tuple key.

**Pitfalls.** Assuming quicksort is always O(n log n); sorting in place when you needed to keep the input; comparator that is not a strict weak ordering (C++ undefined behaviour); forgetting that `sorted` returns a new list.

---
## 9. Searching: binary search and invariants
**ELI15.** Guess a number from 1 to 100: ask "higher or lower?" and always guess the middle. You find it in about 7 guesses.

**Foundations.** Binary search needs **sorted** (or at least monotonic) data and random access. Each step halves the range: O(log n) time, O(1) space iteratively.
**Invariant**: the target, if present, lies within `[lo, hi]`. Every iteration must keep this true and shrink the range, which proves correctness and termination.

```python
def bsearch(a, x):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if a[mid] == x: return mid
        if a[mid] < x: lo = mid + 1
        else: hi = mid - 1
    return -1
```
```cpp
int bsearch(const vector<int>& a,int x){ int lo=0,hi=(int)a.size()-1; while(lo<=hi){ int mid=lo+(hi-lo)/2; if(a[mid]==x) return mid; if(a[mid]<x) lo=mid+1; else hi=mid-1; } return -1; }
```
`lo + (hi - lo)/2` avoids signed overflow of `lo + hi` in C++ (overflow of signed int is undefined behaviour); Python ints do not overflow, but the habit is kept (**Ex.**).

**Working knowledge. Variants.**
- **Lower/upper bound**: first index with `a[i] >= x` (Python `bisect_left`, C++ `lower_bound`); first index `> x` (`bisect_right`, `upper_bound`).
- **Binary search on the answer**: when feasibility is monotonic (if speed `k` works, `k+1` works), search the answer range, e.g. minimum capacity to ship in D days, smallest `x` with `x*x >= n`.
- Rotated sorted array, peak element, search in a matrix with sorted rows and columns (O(m+n) staircase).
- Floating-point: loop a fixed number of times, not until equality.

**Interview / expert.** Decide the template once (`lo < hi` with `hi = mid` for lower bound) and reuse it; most bugs are off-by-one and infinite loops when `mid` does not progress. Complexity of `bisect` on a list insertion is still O(n) because insertion shifts.

**Pitfalls.** Applying it to unsorted data; `mid` rounding causing an infinite loop; wrong boundary updates (`lo = mid` instead of `mid + 1`); returning `lo` without checking bounds.

---
## 10. Two pointers, sliding window, prefix sums
**ELI15.** Two pointers: two fingers moving along a list so you never re-scan. Sliding window: a frame you slide along a row, adding the new item and dropping the old one. Prefix sums: keep a running total so any range sum is "end total minus start total".

**Foundations and patterns.**
- **Opposite-end pointers** on a sorted array: two-sum II, container with most water, 3-sum (sort, fix one, two pointers: O(n^2)). Move the pointer that can improve the answer.
- **Same-direction (fast/slow)**: remove duplicates in place, partition.
- **Sliding window**: maintain a window `[l, r]` and an invariant. Fixed size: update by adding `a[r]` and removing `a[r-k]`. Variable size: expand `r`; while the invariant is broken, shrink `l`. Each index enters and leaves once: O(n).
```python
def longest_unique(s):
    last, l, best = {}, 0, 0
    for r, ch in enumerate(s):
        if ch in last and last[ch] >= l: l = last[ch] + 1
        last[ch] = r; best = max(best, r - l + 1)
    return best
```
- **Prefix sums**: `P[i] = a[0] + ... + a[i-1]`; sum of `a[l..r]` is `P[r+1] - P[l]` in O(1) after O(n) build. With a hash map of prefix counts, "number of subarrays summing to k" is O(n). 2D prefix sums for rectangle sums; difference arrays for range updates.

**Interview / expert.** Sliding window needs a property that stays valid when shrinking (positive numbers for sum-at-least problems; with negatives use prefix sums or deque). Monotonic deque gives sliding window maximum in O(n). Kadane's algorithm (maximum subarray) is a one-pass DP/greedy: `cur = max(x, cur + x)`.

**Pitfalls.** Using sliding window when negatives break monotonicity; off-by-one on window length (`r - l + 1`); forgetting to remove the left element's contribution; not handling an empty window.
