# Sorting Algorithms

> Know time/space complexity of all standard sorts, their stability, and when to prefer each. Interviewers love asking about merge sort vs quick sort trade-offs.

---

## 1. Complexity Table

| Algorithm | Best | Average | Worst | Space | Stable? | Notes |
|-----------|------|---------|-------|-------|---------|-------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | Early-exit if no swaps |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ | Minimum swaps |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | Best for nearly-sorted |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | Preferred for linked lists |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | Cache-friendly, in-place |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | Not cache-friendly |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ | Only for integer ranges |
| Radix Sort | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | ✅ | d = digits, k = base |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | ✅ | Python/Java default |
| Intro Sort | O(n log n) | O(n log n) | O(n log n) | O(log n) | ❌ | C++ `std::sort` default |

---

## 2. Merge Sort — Full Implementation

```cpp
void merge(vector<int>& arr, int l, int m, int r) {
    vector<int> left(arr.begin()+l, arr.begin()+m+1);
    vector<int> right(arr.begin()+m+1, arr.begin()+r+1);
    int i = 0, j = 0, k = l;
    while (i < (int)left.size() && j < (int)right.size())
        arr[k++] = (left[i] <= right[j]) ? left[i++] : right[j++];
    while (i < (int)left.size())  arr[k++] = left[i++];
    while (j < (int)right.size()) arr[k++] = right[j++];
}

void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int m = l + (r - l) / 2;
    mergeSort(arr, l, m);
    mergeSort(arr, m+1, r);
    merge(arr, l, m, r);
}
```

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left  = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    # merge
    result, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    return result + left[i:] + right[j:]
```

**Use for:** Counting inversions, stable sort on linked list, external sort (data doesn't fit in RAM).

---

## 3. Quick Sort — Full Implementation

```cpp
int partition(vector<int>& arr, int lo, int hi) {
    int pivot = arr[hi]; // last element as pivot
    int i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i+1], arr[hi]);
    return i + 1;
}

void quickSort(vector<int>& arr, int lo, int hi) {
    if (lo < hi) {
        int pi = partition(arr, lo, hi);
        quickSort(arr, lo, pi - 1);
        quickSort(arr, pi + 1, hi);
    }
}
```

**Randomized pivot** (avoids O(n²) on sorted input):
```cpp
int randomPartition(vector<int>& arr, int lo, int hi) {
    int r = lo + rand() % (hi - lo + 1);
    swap(arr[r], arr[hi]);
    return partition(arr, lo, hi);
}
```

---

## 4. Counting Sort

```cpp
void countingSort(vector<int>& arr, int maxVal) {
    vector<int> count(maxVal + 1, 0);
    for (int x : arr) count[x]++;
    // cumulative (for stable sort)
    for (int i = 1; i <= maxVal; i++) count[i] += count[i-1];
    vector<int> output(arr.size());
    for (int i = arr.size()-1; i >= 0; i--)
        output[--count[arr[i]]] = arr[i];
    arr = output;
}
```

**Use when:** k is small (e.g., age, grades, letters). Pointless if k >> n.

---

## 5. Key Interview Points

| Question | Answer |
|----------|--------|
| Why is merge sort preferred for linked lists? | No random access needed; merge is O(1) space if done in-place with pointer manipulation |
| When does quick sort degrade to O(n²)? | Already sorted input with last/first element as pivot |
| What sort does C++ `std::sort` use? | Intro Sort (quick + heap + insertion) |
| What sort does Python use? | Tim Sort (merge + insertion, exploits natural runs) |
| Why isn't heap sort used much in practice? | Poor cache locality (accesses distant array elements) |
| Is quick sort stable? | No. Use merge sort or `std::stable_sort` when stability matters |
