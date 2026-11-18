# Hash Tables

> **Core idea:** Map keys to buckets using a hash function. O(1) average-case insert/search/delete. The devil is in collision handling.

---

## 1. Hash Function Requirements

1. **Deterministic** — same key always maps to same bucket
2. **Uniform distribution** — minimize collisions
3. **Fast to compute** — O(1) for fixed-size keys

### Polynomial Rolling Hash (for strings)
```
h = (s[0]*p^0 + s[1]*p^1 + ... + s[n-1]*p^(n-1)) % MOD
where p = 31 (or 37 for lowercase+digits), MOD = 1e9 + 9
```

---

## 2. Collision Resolution

### Separate Chaining (C++ `unordered_map` uses this)
```
bucket[0] → [key1, val1] → [key4, val4] → null
bucket[1] → [key2, val2] → null
bucket[2] → [key3, val3] → null
```
- Worst case O(n) if all keys hash to same bucket
- Load factor λ = n/m; rehash when λ > threshold (~0.75)

### Open Addressing (Python `dict` uses this, with probing)
- Linear probing: try (h+1) % m, (h+2) % m, ...
- Quadratic probing: try h + 1², h + 2², ...
- Double hashing: use second hash function for step size

**Tombstone** — when deleting, place a "deleted" marker so searches don't stop prematurely.

---

## 3. C++ `unordered_map` / `unordered_set`

```cpp
unordered_map<string, int> freq;
freq["hello"]++;
freq.count("hello");  // 0 or 1
freq.find("hello");   // iterator, .end() if not found
freq.erase("hello");
freq.size();

// Iterate
for (auto& [key, val] : freq) { /* ... */ }

// Reserve to avoid rehashing
freq.reserve(1024);
```

**Pitfall:** `unordered_map` can degrade to O(n) with adversarial inputs. Use `map` (O(log n)) when guaranteed order or worst-case matters.

---

## 4. Python `dict` / `set`

```python
d = {}
d['hello'] = 1
'hello' in d          # O(1)
d.get('hello', 0)     # default 0 if missing
del d['hello']

from collections import defaultdict, Counter
freq = Counter("abracadabra")  # {'a':5, 'b':2, 'r':2, 'c':1, 'd':1}
dd = defaultdict(list)         # avoids KeyError for missing keys
```

---

## 5. Classic Patterns

### Frequency Count
```python
freq = Counter(nums)
for val, count in freq.items():
    # ...
```

### Prefix Sum + Hash (Subarray Sum = K)
```python
def subarraySum(nums, k):
    count = 0
    prefix = 0
    seen = {0: 1}  # prefix -> count
    for n in nums:
        prefix += n
        count += seen.get(prefix - k, 0)
        seen[prefix] = seen.get(prefix, 0) + 1
    return count
```

### Complement Lookup (Two Sum)
```python
seen = {}
for i, n in enumerate(nums):
    if target - n in seen:
        return [seen[target - n], i]
    seen[n] = i
```

### Grouping with Hash Key
```python
# Group anagrams
from collections import defaultdict
groups = defaultdict(list)
for s in strs:
    groups[tuple(sorted(s))].append(s)
return list(groups.values())
```

---

## 6. LRU Cache Implementation

**Data structures:** `unordered_map` + doubly linked list (DLL)

```cpp
class LRUCache {
    struct Node { int key, val; Node *prev, *next; };
    unordered_map<int, Node*> cache;
    Node *head, *tail;  // sentinels
    int cap;

    void remove(Node* n) {
        n->prev->next = n->next;
        n->next->prev = n->prev;
    }
    void insertFront(Node* n) {  // insert after head (MRU position)
        n->next = head->next;
        n->prev = head;
        head->next->prev = n;
        head->next = n;
    }

public:
    LRUCache(int capacity) : cap(capacity) {
        head = new Node(); tail = new Node();
        head->next = tail; tail->prev = head;
    }

    int get(int key) {
        if (!cache.count(key)) return -1;
        Node* n = cache[key];
        remove(n); insertFront(n);
        return n->val;
    }

    void put(int key, int value) {
        if (cache.count(key)) { remove(cache[key]); delete cache[key]; }
        Node* n = new Node{key, value};
        cache[key] = n;
        insertFront(n);
        if ((int)cache.size() > cap) {
            Node* lru = tail->prev;
            remove(lru);
            cache.erase(lru->key);
            delete lru;
        }
    }
};
```

**Python:** Use `collections.OrderedDict` which maintains insertion order.

---

## 7. Practice Problems

| Problem | Pattern | LC # |
|---------|---------|------|
| Two Sum | Complement lookup | 1 |
| Valid Anagram | Frequency count | 242 |
| Group Anagrams | Sorted string as key | 49 |
| Longest Consecutive Sequence | Hash set + extend | 128 |
| Subarray Sum Equals K | Prefix + hash | 560 |
| LRU Cache | HashMap + DLL | 146 |
| Insert Delete GetRandom O(1) | HashMap + array swap | 380 |
| Top K Frequent Elements | HashMap + heap | 347 |
