# Heaps (Priority Queues)

> **Core idea:** Complete binary tree stored as an array. Parent always ≤ children (min-heap) or ≥ children (max-heap). O(log n) insert/delete, O(1) min/max access.

---

## 1. Heap Property & Array Representation

```
Heap stored as array: index 0-based
Parent of i  = (i-1)/2
Left child   = 2*i + 1
Right child  = 2*i + 2
```

Min-heap example:
```
        1
       / \
      3   5
     / \  |
    4   8  7

Array: [1, 3, 5, 4, 8, 7]
```

---

## 2. Heapify Operations

### Sift Down (restore heap after replacing root) — O(log n)
```cpp
void siftDown(vector<int>& h, int i, int n) {
    while (true) {
        int smallest = i;
        int L = 2*i+1, R = 2*i+2;
        if (L < n && h[L] < h[smallest]) smallest = L;
        if (R < n && h[R] < h[smallest]) smallest = R;
        if (smallest == i) break;
        swap(h[i], h[smallest]);
        i = smallest;
    }
}
```

### Build Heap — O(n)
```cpp
void buildHeap(vector<int>& h) {
    int n = h.size();
    for (int i = n/2 - 1; i >= 0; i--) siftDown(h, i, n);
}
// O(n) not O(n log n): lower nodes do less work, harmonic series
```

### Heap Sort — O(n log n), O(1) space
```cpp
void heapSort(vector<int>& arr) {
    buildHeap(arr);  // max-heap
    int n = arr.size();
    for (int i = n-1; i > 0; i--) {
        swap(arr[0], arr[i]);   // move max to end
        siftDown(arr, 0, i);    // restore heap on reduced array
    }
}
```

---

## 3. C++ `priority_queue`

```cpp
// Max-heap (default)
priority_queue<int> maxH;
maxH.push(3); maxH.push(1); maxH.push(5);
maxH.top();   // 5
maxH.pop();   // removes 5

// Min-heap
priority_queue<int, vector<int>, greater<int>> minH;

// Custom comparator (pair, sort by second element ascending)
auto cmp = [](pair<int,int> a, pair<int,int> b){ return a.second > b.second; };
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);
```

### Python `heapq`
```python
import heapq

h = []
heapq.heappush(h, 3)
heapq.heappush(h, 1)
heapq.heappop(h)   # returns 1 (min-heap)

# Max-heap: negate values
heapq.heappush(h, -5)
-heapq.heappop(h)  # returns 5

# heapify in O(n)
arr = [3, 1, 4, 1, 5]
heapq.heapify(arr)
```

---

## 4. Top-K Pattern

### K Largest Elements — min-heap of size K
```cpp
vector<int> topKLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> minH;
    for (int n : nums) {
        minH.push(n);
        if ((int)minH.size() > k) minH.pop();
    }
    vector<int> res;
    while (!minH.empty()) { res.push_back(minH.top()); minH.pop(); }
    return res;
}
// Min-heap of size K: everything smaller than the K-th largest gets evicted.
// Top of heap = K-th largest element.
```

### K Closest Points to Origin
```cpp
vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
    // Max-heap: evict farthest
    auto cmp = [](pair<int,vector<int>> a, pair<int,vector<int>> b){ return a.first < b.first; };
    priority_queue<pair<int,vector<int>>, vector<pair<int,vector<int>>>, decltype(cmp)> pq(cmp);
    for (auto& p : points) {
        int d = p[0]*p[0] + p[1]*p[1];
        pq.push({d, p});
        if ((int)pq.size() > k) pq.pop();
    }
    vector<vector<int>> res;
    while (!pq.empty()) { res.push_back(pq.top().second); pq.pop(); }
    return res;
}
```

---

## 5. Median from Data Stream (Two Heaps)

```cpp
class MedianFinder {
    priority_queue<int> lo;              // max-heap (lower half)
    priority_queue<int, vector<int>, greater<int>> hi; // min-heap (upper half)
public:
    void addNum(int num) {
        lo.push(num);
        hi.push(lo.top()); lo.pop();     // balance: ensure lo.top() <= hi.top()
        if (lo.size() < hi.size()) {     // keep lo.size() >= hi.size()
            lo.push(hi.top()); hi.pop();
        }
    }
    double findMedian() {
        if (lo.size() > hi.size()) return lo.top();
        return (lo.top() + hi.top()) / 2.0;
    }
};
```

---

## 6. Practice Problems

| Problem | Technique | LC # |
|---------|-----------|------|
| Kth Largest in Array | Min-heap size K | 215 |
| K Closest Points to Origin | Max-heap size K | 973 |
| Find Median from Data Stream | Two heaps | 295 |
| Task Scheduler | Max-heap + cooldown | 621 |
| Merge K Sorted Lists | Min-heap of (val, list_idx) | 23 |
| Top K Frequent Elements | Min-heap size K | 347 |
| Reorganize String | Max-heap + previous char | 767 |
| Minimum Cost to Connect Ropes | Min-heap, combine smallest two | — |
