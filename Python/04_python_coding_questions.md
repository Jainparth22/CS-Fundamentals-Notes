# Python Coding and Output Questions (with answers)

## A. "What is the output?" questions
```python
# 1
a = [1, 2, 3]; b = a; b.append(4); print(a)          # [1, 2, 3, 4]  same object
# 2
def f(x, l=[]): l.append(x); return l
print(f(1), f(2))                                     # [1, 2] [1, 2]  shared default
# 3
print(0.1 + 0.2 == 0.3)                               # False  floating point; use math.isclose
# 4
x = 256; y = 256; print(x is y)                       # True (small int cache); 257 may be False
# 5
print([i for i in range(3)] == list(range(3)))        # True
# 6
fs = [lambda: i for i in range(3)]; print([f() for f in fs])   # [2, 2, 2] late binding
# 7
t = (1, [2, 3]); t[1].append(4); print(t)             # (1, [2, 3, 4])
# 8
print(bool([]), bool(""), bool(0), bool("0"))         # False False False True
# 9
d = {}; d[[1]] = 2                                    # TypeError: unhashable type: 'list'
# 10
print("a" * 3, [0] * 3, [[0]] * 2)                    # aaa [0, 0, 0] [[0], [0]] (same inner list!)
# 11
s = "hello"; print(s[::-1], s[1:3], s[-1])            # olleh el o
# 12
print(type(1/2), 7//2, -7//2, 7 % -3)                 # float 3 -4 -2
# 13
try: 1/0
except ZeroDivisionError: print("a")
else: print("b")
finally: print("c")                                   # a c
# 14
def g():
    try: return "try"
    finally: print("finally")                          # prints finally, then returns "try"
# 15
print(sorted({3: 'a', 1: 'b'}))                       # [1, 3]  sorts keys
```

## B. Classic coding problems
```python
# Reverse a string / check palindrome
def is_pal(s): s = ''.join(c.lower() for c in s if c.isalnum()); return s == s[::-1]

# Count word frequency
from collections import Counter
Counter(text.split()).most_common(3)

# Two sum: O(n) with a dict
def two_sum(nums, target):
    seen = {}
    for i, n in enumerate(nums):
        if target - n in seen: return seen[target - n], i
        seen[n] = i

# Fibonacci with memoization
from functools import lru_cache
@lru_cache(None)
def fib(n): return n if n < 2 else fib(n-1) + fib(n-2)

# Flatten nested list
def flatten(x):
    for i in x:
        if isinstance(i, list): yield from flatten(i)
        else: yield i

# First non-repeating character
def first_unique(s):
    c = Counter(s)
    return next((ch for ch in s if c[ch] == 1), None)

# Anagram groups
from collections import defaultdict
def group_anagrams(words):
    g = defaultdict(list)
    for w in words: g[''.join(sorted(w))].append(w)
    return list(g.values())

# Remove duplicates, keep order
list(dict.fromkeys(items))

# Merge two dicts
{**a, **b}        # or a | b in 3.9+

# Check balanced brackets
def balanced(s):
    pairs = {')': '(', ']': '[', '}': '{'}; st = []
    for ch in s:
        if ch in '([{': st.append(ch)
        elif ch in pairs:
            if not st or st.pop() != pairs[ch]: return False
    return not st

# LRU cache from scratch
from collections import OrderedDict
class LRU:
    def __init__(self, cap): self.cap, self.d = cap, OrderedDict()
    def get(self, k):
        if k not in self.d: return -1
        self.d.move_to_end(k); return self.d[k]
    def put(self, k, v):
        self.d[k] = v; self.d.move_to_end(k)
        if len(self.d) > self.cap: self.d.popitem(last=False)

# Read a huge file lazily
def lines(path):
    with open(path) as f:
        for line in f: yield line.rstrip("\n")

# Thread-safe counter
import threading
class Counter_:
    def __init__(self): self.n, self.lock = 0, threading.Lock()
    def inc(self):
        with self.lock: self.n += 1

# Simple retry decorator (see core notes) and rate limiter (token bucket)
import time
class TokenBucket:
    def __init__(self, rate, cap): self.rate, self.cap, self.tokens, self.t = rate, cap, cap, time.monotonic()
    def allow(self):
        now = time.monotonic()
        self.tokens = min(self.cap, self.tokens + (now - self.t) * self.rate); self.t = now
        if self.tokens >= 1: self.tokens -= 1; return True
        return False
```

## C. Short conceptual drills
1. Explain `deepcopy` vs `copy` with a nested list.
2. Why does `sorted(d.items(), key=lambda kv: kv[1])` sort by value?
3. Write a decorator that caches results with a TTL.
4. Implement `__iter__` and `__next__` for a countdown class.
5. Write a context manager that suppresses a specific exception (`contextlib.suppress` exists).
6. Explain MRO for `class D(B, C)` with a diamond and print it.
7. Convert a list of dicts to a dict of lists.
8. Group a list of orders by customer and compute totals (`defaultdict(float)`).
9. Implement a generator that yields Fibonacci numbers forever.
10. Explain why `x += y` can differ from `x = x + y` for lists (`+=` mutates in place).

## D. Pandas and NumPy one-liners that come up
```python
import pandas as pd, numpy as np
df.groupby('dept')['salary'].agg(['mean', 'max']).reset_index()
df.sort_values('salary', ascending=False).head(3)
df.isna().sum(); df.fillna(0); df.dropna(subset=['x'])
df.merge(other, on='id', how='left')
df['x'].value_counts(normalize=True)
df.pivot_table(index='a', columns='b', values='c', aggfunc='sum')
np.arange(10).reshape(2, 5); a[a > 3]; a.mean(axis=0)
```
Know: vectorisation vs `apply`, broadcasting, `loc` vs `iloc`, `copy` warnings (SettingWithCopy).
