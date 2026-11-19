# Python Core Language: Interview Notes

## 1. Data model basics
- **Everything is an object.** Names are references to objects; assignment binds a name, it never copies.
- **Mutable:** list, dict, set, bytearray, user classes. **Immutable:** int, float, bool, str, tuple, frozenset, bytes.
- `is` compares identity (`id()`); `==` compares value (`__eq__`). Use `is` only for `None`, `True`, `False`.
- Small ints (-5..256) and some strings are cached, so `a is b` can be True by accident. Never rely on it.
- A tuple is immutable, but a mutable object inside it can change: `t = ([1], 2); t[0].append(9)` works.

### Mutable default argument (classic trap)
```python
def add(x, bucket=[]):      # default evaluated ONCE at definition time
    bucket.append(x)
    return bucket
add(1); add(2)              # [1, 2] - shared list
# fix
def add(x, bucket=None):
    bucket = [] if bucket is None else bucket
    bucket.append(x); return bucket
```

### Shallow vs deep copy
```python
import copy
a = [[1, 2], [3]]
b = a.copy()           # shallow: inner lists shared
c = copy.deepcopy(a)   # fully independent
a[0].append(9)         # b sees it, c does not
```

## 2. Scope: LEGB
Local, Enclosing, Global, Built-in. `global x` rebinds a module-level name; `nonlocal x` rebinds an enclosing function's name.
```python
def outer():
    n = 0
    def inc():
        nonlocal n
        n += 1
        return n
    return inc
```
**Late binding in closures:** `[lambda: i for i in range(3)]` all return 2. Fix with a default arg: `lambda i=i: i`.

## 3. Built-in collections and their cost
| Type | Ordered | Mutable | Lookup | Notes |
|---|---|---|---|---|
| list | yes | yes | O(n) search, O(1) index | dynamic array, amortised O(1) append, O(n) insert at front |
| tuple | yes | no | O(n) | hashable if items hashable |
| dict | insertion order (3.7+) | yes | O(1) avg | keys must be hashable |
| set | no | yes | O(1) avg | unique, hashable items |
| deque | yes | yes | O(1) both ends | `collections.deque` |

- `dict` is a hash table with open addressing; resizing is amortised O(1).
- Hashable means: has `__hash__`, equal objects have equal hashes, and the hash never changes during the object's life.
- `defaultdict(list)`, `Counter`, `OrderedDict`, `namedtuple`, `heapq` (min-heap), `bisect` are the useful ones.

## 4. Strings
- Immutable; building in a loop with `+=` is O(n^2) in theory (CPython sometimes optimises). Use `''.join(parts)`.
- f-strings (3.6+) are fastest and clearest: `f"{name!r:>10}"`.
- `str` is Unicode; `bytes` is raw. `s.encode('utf-8')` and `b.decode('utf-8')`.

## 5. Comprehensions and generators
```python
squares = [x*x for x in range(10) if x % 2 == 0]   # list comprehension
unique = {x % 3 for x in range(10)}                 # set
index = {w: i for i, w in enumerate(words)}         # dict
gen = (x*x for x in range(10**9))                   # generator: lazy, O(1) memory
```
- A **generator function** uses `yield`; it keeps its frame between calls. `yield from` delegates to a sub-generator.
- A generator is consumed once. Iterators implement `__iter__` and `__next__`.
- `sum(x for x in data)` is memory-light; `sum([x for x in data])` builds a list first.

## 6. Functions
- First-class objects: pass, return, store them.
- `*args` collects extra positional args (tuple); `**kwargs` collects keyword args (dict).
- Keyword-only args after `*`: `def f(a, *, b)`. Positional-only before `/`: `def f(a, /, b)`.
- `lambda` is a single-expression anonymous function.
- **Closures** capture variables, not values.
- `functools`: `lru_cache` (memoise), `partial`, `wraps`, `reduce`, `cached_property`.

### Decorators
```python
import functools, time
def timed(fn):
    @functools.wraps(fn)          # keeps name and docstring
    def wrapper(*a, **kw):
        t = time.perf_counter()
        try:
            return fn(*a, **kw)
        finally:
            print(fn.__name__, time.perf_counter() - t)
    return wrapper

@timed
def work(): ...

# decorator with arguments = a function that returns a decorator
def retry(times):
    def deco(fn):
        @functools.wraps(fn)
        def wrapper(*a, **kw):
            for i in range(times):
                try: return fn(*a, **kw)
                except Exception:
                    if i == times - 1: raise
        return wrapper
    return deco
```

## 7. Context managers
```python
class Timer:
    def __enter__(self): self.t = time.time(); return self
    def __exit__(self, exc_type, exc, tb): print(time.time() - self.t); return False  # False = do not swallow errors

from contextlib import contextmanager
@contextmanager
def opened(path):
    f = open(path); 
    try: yield f
    finally: f.close()
```

## 8. Exceptions
```python
try:
    risky()
except (ValueError, KeyError) as e:
    handle(e)
else:           # runs only if no exception
    ok()
finally:        # always runs
    cleanup()
```
- Catch specific exceptions; avoid bare `except:` (it also catches `KeyboardInterrupt`).
- `raise ... from e` chains the cause. Custom errors subclass `Exception`.
- EAFP ("easier to ask forgiveness") is Pythonic; LBYL ("look before you leap") is the alternative.

## 9. Memory and the GIL
- **Reference counting** frees objects immediately at zero; a **cyclic garbage collector** handles reference cycles (`gc` module). `weakref` avoids keeping objects alive.
- **GIL (Global Interpreter Lock):** only one thread executes Python bytecode at a time in CPython. It protects reference counts.
  - CPU-bound work: threads do not speed it up; use `multiprocessing`, `concurrent.futures.ProcessPoolExecutor`, or C extensions (NumPy releases the GIL).
  - I/O-bound work: threads and `asyncio` work well because the GIL is released while waiting.
- Python 3.13 adds an optional free-threaded build (experimental).
- `__slots__` removes the per-instance `__dict__` and saves memory.

## 10. Useful idioms
```python
a, b = b, a                                    # swap
first, *rest = [1, 2, 3, 4]                    # unpacking
for i, x in enumerate(items, start=1): ...
for a, b in zip(xs, ys): ...                   # stops at the shortest
sorted(data, key=lambda d: (d['age'], d['name']), reverse=True)
any(x > 3 for x in xs); all(...)
import itertools  # chain, groupby, product, permutations, combinations, accumulate
x if cond else y
if (n := len(s)) > 10: ...                     # walrus
```

## 11. Modules and packaging
- `import` caches modules in `sys.modules`; circular imports are fixed by moving imports inside functions or restructuring.
- `if __name__ == "__main__":` runs code only when executed directly.
- `venv` or `uv` for isolation; `pyproject.toml` for metadata; `pip freeze > requirements.txt`.

## 12. Typing
```python
from typing import Optional, Union, Callable, TypeVar, Protocol
def f(x: int | None, items: list[str]) -> dict[str, int]: ...
```
Type hints are not enforced at runtime; use `mypy`/`pyright` or Pydantic for validation.

## Quick interview Q&A
- **Difference between list and tuple?** Mutability, hashability, small speed/memory advantage for tuples; semantics (record vs collection).
- **How is a dict implemented?** Hash table with open addressing and a compact entries array that preserves insertion order.
- **What is `*args` vs `**kwargs`?** Variable positional vs keyword arguments.
- **What is a generator and why use one?** Lazy iterator; constant memory for large or infinite sequences.
- **What does the GIL do?** Serialises bytecode execution per process; hurts CPU-bound threads, not I/O-bound ones.
- **`@staticmethod` vs `@classmethod` vs instance method?** No implicit arg vs class (`cls`) vs instance (`self`).
- **Why is `x = []` default arg bad?** Evaluated once; shared across calls.
- **How do you make a class hashable?** Define `__eq__` and `__hash__` consistently (or use `@dataclass(frozen=True)`).
- **What is `__init__` vs `__new__`?** `__new__` creates the instance (used for singletons, immutables); `__init__` initialises it.
- **`is` vs `==`?** Identity vs equality.
- **How does Python manage memory?** Private heap, reference counting plus generational cyclic GC.
- **What is monkey patching?** Changing a class or module at runtime; useful in tests, risky in production.
