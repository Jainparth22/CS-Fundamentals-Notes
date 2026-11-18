# Stacks and Queues

> **Stack:** LIFO. **Queue:** FIFO. Both are O(1) push/pop. Know monotonic stack cold — it appears in 30+ medium/hard problems.

---

## 1. Stack

### C++ (using `std::stack`)
```cpp
stack<int> st;
st.push(5);         // push
st.top();           // peek (no removal)
st.pop();           // remove top (void)
st.empty();         // true/false
st.size();          // number of elements
```

### Python (using list)
```python
stack = []
stack.append(5)     # push
stack[-1]           # peek
stack.pop()         # pop and return
len(stack) == 0     # empty check
```

---

## 2. Queue

### C++ (using `std::queue`)
```cpp
queue<int> q;
q.push(5);     // enqueue (at back)
q.front();     // peek front
q.pop();       // dequeue from front
```

### Python (using `collections.deque`)
```python
from collections import deque
q = deque()
q.append(5)        # enqueue right
q.popleft()        # dequeue left
q[0]               # peek front
```

---

## 3. Monotonic Stack

A stack that maintains elements in increasing or decreasing order. Use when you need the **next greater/smaller** element.

### Next Greater Element (NGE) — O(n)
```cpp
vector<int> nextGreater(vector<int>& arr) {
    int n = arr.size();
    vector<int> result(n, -1);
    stack<int> st;  // indices, elements in stack are DECREASING
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] < arr[i]) {
            result[st.top()] = arr[i];
            st.pop();
        }
        st.push(i);
    }
    return result;
}
```

**Trace:** arr = [2, 1, 2, 4, 3]
- i=0: push 0 → stack: [0]
- i=1: 1 < 2, push 1 → stack: [0, 1]
- i=2: 2 > 1, pop 1 (result[1]=2), 2 >= 2, push 2 → stack: [0, 2]
- i=3: 4 > 2, pop 2 (result[2]=4), 4 > 2, pop 0 (result[0]=4), push 3 → stack: [3]
- i=4: 3 < 4, push 4 → stack: [3, 4]
- Remaining in stack: result stays -1

Result: [4, 2, 4, -1, -1]

### Previous Smaller Element — O(n)
```cpp
vector<int> prevSmaller(vector<int>& arr) {
    int n = arr.size();
    vector<int> result(n, -1);
    stack<int> st;  // indices, elements INCREASING
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
        if (!st.empty()) result[i] = arr[st.top()];
        st.push(i);
    }
    return result;
}
```

---

## 4. Largest Rectangle in Histogram (Stack)

```cpp
int largestRectangle(vector<int>& heights) {
    int n = heights.size();
    stack<int> st;
    int maxArea = 0;
    heights.push_back(0);  // sentinel to flush stack at end
    for (int i = 0; i <= n; i++) {
        while (!st.empty() && heights[st.top()] > heights[i]) {
            int h = heights[st.top()]; st.pop();
            int w = st.empty() ? i : i - st.top() - 1;
            maxArea = max(maxArea, h * w);
        }
        st.push(i);
    }
    return maxArea;
}
```

---

## 5. Implement Queue Using Two Stacks

```cpp
class MyQueue {
    stack<int> in, out;
    void transfer() {
        while (!in.empty()) { out.push(in.top()); in.pop(); }
    }
public:
    void push(int x) { in.push(x); }
    int pop()  { if (out.empty()) transfer(); int t=out.top(); out.pop(); return t; }
    int peek() { if (out.empty()) transfer(); return out.top(); }
    bool empty() { return in.empty() && out.empty(); }
};
```

Amortized O(1) per operation — each element is transferred at most once.

---

## 6. Deque (Double-Ended Queue)

```cpp
deque<int> dq;
dq.push_front(x);   dq.push_back(x);
dq.pop_front();     dq.pop_back();
dq.front();         dq.back();
```

Used in: Sliding Window Maximum (monotonic deque), 0-1 BFS.

---

## 7. Practice Problems

| Problem | Key Idea | LC # |
|---------|----------|------|
| Valid Parentheses | Stack match | 20 |
| Min Stack | Stack + auxiliary min-stack | 155 |
| Daily Temperatures | Monotonic stack (NGE days) | 739 |
| Largest Rectangle in Histogram | Monotonic stack | 84 |
| Sliding Window Maximum | Monotonic deque | 239 |
| Implement Queue with Stacks | Two stacks | 232 |
| Car Fleet | Sort + stack | 853 |
| Evaluate Reverse Polish Notation | Stack for operands | 150 |
| Decode String | Stack (k and current string) | 394 |
