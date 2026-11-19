# DSA Notes B: Linked lists, stacks and queues, hashing (concepts 5 to 7)

Compiled notes, not yet assessed (M0). **Est.** = sourced, **Ex.** = my example.

---
## 5. Linked lists and pointers/references
**ELI15.** A linked list is a scavenger hunt: each clue (node) holds a value and tells you where the next clue is. You cannot jump to clue 50; you must follow the trail.

**Foundations.** A node holds `value` and `next` (singly) plus `prev` (doubly). Access by index O(n); insert/delete after a known node O(1); search O(n). Memory is not contiguous, so it is cache-unfriendly and uses extra per-node space.

**Working knowledge.**
```python
class Node:
    def __init__(self, v, nxt=None): self.v, self.next = v, nxt
def reverse(head):
    prev = None
    while head:
        nxt = head.next      # save
        head.next = prev     # flip
        prev, head = head, nxt
    return prev              # O(n) time, O(1) space
```
```cpp
struct Node{ int v; Node* next; };
Node* reverse(Node* h){ Node* prev=nullptr; while(h){ Node* n=h->next; h->next=prev; prev=h; h=n; } return prev; }
```
Key patterns:
- **Dummy (sentinel) head** removes special cases for inserting/deleting the first node.
- **Fast/slow pointers**: find the middle (slow moves 1, fast moves 2); detect a cycle (Floyd): if they meet there is a cycle; find the cycle start by resetting one pointer to the head and moving both at speed 1.
- **Merge two sorted lists**, **remove nth from end** (two pointers n apart), **reverse a sublist**.

**Interview / expert.** In C++ you own memory: free nodes, avoid dangling pointers, prefer `std::unique_ptr` or ownership clarity. `std::list` (doubly) and `std::forward_list` (singly): inserting does not invalidate other iterators (cppreference iterator-invalidation table, **Est.**). In practice `vector` usually beats a list even for middle inserts because of cache effects (**Ex.**, widely observed). LRU cache = hash map + doubly linked list (O(1) get/put).

**Pitfalls.** Losing the rest of the list by overwriting `next` before saving it; null dereference; forgetting to update `tail`; infinite loops in cyclic lists.

---
## 6. Stacks, queues, deques
**ELI15.** Stack: a pile of plates, take the top one (last in, first out). Queue: a line at a shop (first in, first out).

**Foundations.** Stack: `push`, `pop`, `peek` all O(1). Queue: `enqueue`, `dequeue` O(1). Deque supports both ends O(1).
- Python: stack with `list` (`append`/`pop`); queue/deque with `collections.deque` (`list.pop(0)` is O(n), **Est.**).
- C++: `std::stack`, `std::queue`, `std::deque` (container adaptors over a sequence container, cppreference, **Est.**).

**Working knowledge.**
- Stack uses: balanced brackets, expression evaluation, undo, DFS (explicit stack), call stack.
- **Monotonic stack**: keep a stack of increasing/decreasing values to answer "next greater element" in O(n): each element is pushed and popped at most once (amortised).
```python
def next_greater(a):
    res, st = [-1]*len(a), []           # st holds indices with decreasing values
    for i, x in enumerate(a):
        while st and a[st[-1]] < x: res[st.pop()] = x
        st.append(i)
    return res
```
- Queue uses: BFS, scheduling, sliding-window maximum with a monotonic deque (O(n)).
- A queue from two stacks (amortised O(1)); a stack from two queues.

**Interview / expert.** Know circular-buffer queues (fixed-size array with head/tail modulo). `priority_queue` is not FIFO (concept 12). Amortised argument for monotonic structures: total pushes <= n so total pops <= n.

**Pitfalls.** Popping from an empty stack; using `list.pop(0)` in BFS (O(n) each, O(n^2) total); confusing queue and priority queue.

---
## 7. Hashing: hash tables, maps and sets
**ELI15.** A hash table is a coat-check: your ticket number tells staff exactly which hook to look at, so no searching. The "ticket" is computed from the item by a hash function.

**Foundations.** A hash function maps a key to an integer; `index = hash(key) mod m`. Two keys can land on the same index (**collision**). Strategies:
- **Chaining**: each bucket holds a list. C++ `unordered_map` is organised into buckets (cppreference, **Est.**).
- **Open addressing**: probe other slots (linear, quadratic, double hashing). CPython dicts use open addressing (**Ex.**, standard knowledge; not verified in this research).
**Load factor** `alpha = n/m`; resize (rehash) when it gets high to keep operations O(1) average.

**Costs (Est.).** Python dict/set: get, set, delete, membership O(1) **average**; O(n) worst case if every key collides; resizing makes insert amortised (Python docs). C++ `unordered_map`: average O(1), worst O(n); `std::map`/`std::set` are ordered, O(log n) (cppreference).
| Need | Python | C++ |
|---|---|---|
| key to value | `dict` | `unordered_map` |
| unique items | `set` | `unordered_set` |
| count items | `collections.Counter` | `unordered_map<T,int>` |
| ordered keys | sort keys / `sortedcontainers` (3rd party) | `std::map` |

**Working knowledge. Patterns.**
- Frequency counting; two-sum via complement lookup (O(n) time, O(n) space); group anagrams (key = sorted word or count tuple); longest substring without repeats; subarray sum equals k using prefix-sum counts; detect duplicates.
```python
def two_sum(nums, target):
    seen = {}
    for i, x in enumerate(nums):
        if target - x in seen: return seen[target - x], i
        seen[x] = i
```
```cpp
vector<int> twoSum(vector<int>& a,int t){ unordered_map<int,int> m; for(int i=0;i<a.size();++i){ auto it=m.find(t-a[i]); if(it!=m.end()) return {it->second,i}; m[a[i]]=i; } return {}; }
```

**Interview / expert.** Keys must be hashable and their hash must not change: in Python lists cannot be keys, tuples of hashables can. Equal keys must have equal hashes (cppreference unordered_map, **Est.**). Python's string hashing is randomised per process, so dict order is by insertion, not by hash (since 3.7). Know how to implement a hash table with arrays (Google's PDF says you should be able to, **Est.**). Adversarial inputs can force collisions (hash-flooding); languages use randomised or keyed hashes. Bloom filters and consistent hashing extend the idea.

**Pitfalls.** Assuming O(1) is guaranteed; mutating a key object; using floats as keys; iterating and modifying a dict at the same time; unordered iteration order in C++.
