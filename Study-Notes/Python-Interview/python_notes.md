# Python Interview Notes (ELI15 to expert)

Compiled reference notes, unassessed (M0). **Est.** = checked against official Python documentation (sources in `SOURCES.md`); **Ex.** = standard knowledge or my example, not verified in this pass. Version-sensitive items are flagged.

---
## 1. Data model: objects, names, mutability
**ELI15.** A variable is a sticky label on a box, not the box. `b = a` puts a second label on the same box. If the box can be changed (a list), both labels see the change.

**Foundations (Est.).** Everything is an object. Mutability is set by type: numbers, strings, tuples are immutable; lists, dicts are mutable. An immutable container such as a tuple can still "change" if it holds a mutable object that changes (the data-model reference says this). `is` compares identity, `==` compares value.

**Working knowledge.**
- Assignment never copies. Copy: `list(a)`, `a[:]`, `copy.copy` (shallow), `copy.deepcopy` (recursive).
- **Mutable default argument**: `def f(x, acc=[])` creates the list once at definition time (**Ex.**); use `None` and create inside.
- `a += b` on a list mutates in place; on a tuple/str it rebinds a new object.
- Small-int and string caching (CPython, -5 to 256 for ints) is an implementation detail: never use `is` to compare values (**Ex.**).
- Pass by object reference: a function gets a reference to the same object; mutating it affects the caller, rebinding the parameter does not.

**Interview / expert.** Hashability: a key must have a hash that never changes while it is in a dict/set, and **equal objects must have equal hashes** (**Est.**). A class that defines `__eq__` without `__hash__` becomes unhashable (the page's text was cut off; this rule is general knowledge, **Ex.**). Mutable classes with `__eq__` should not define `__hash__` (**Est.**). Build hashes from a tuple of the compared fields (**Est.**). `dict` keeps insertion order as a language guarantee from 3.7 (**Ex.**).

**Pitfalls.** `[[0]*3]*3` aliases rows; modifying a list while iterating it; `x == None` instead of `x is None`; assuming `tuple` is deeply immutable.

---
## 2. Functions, scope, closures, decorators
**ELI15.** A function is a mini machine. A closure is a machine that remembers the values around it when it was built. A decorator is a gift wrapper that adds behaviour to a machine without opening it.

**Foundations.** Functions are first-class objects. Scope lookup is **LEGB**: Local, Enclosing, Global, Built-in; `global`/`nonlocal` rebind outer names. Arguments: positional, keyword, defaults, `*args` (tuple), `**kwargs` (dict), keyword-only after `*`, positional-only before `/`.

**Decorators (Est.).** A decorator is a function that returns another function; `@wrapper` is shorthand for `f = wrapper(f)` (glossary, with `@staticmethod` as the same pattern).
```python
import functools, time
def timed(fn):
    @functools.wraps(fn)                 # keep __name__ and docstring
    def wrapper(*a, **kw):
        t = time.perf_counter()
        try: return fn(*a, **kw)
        finally: print(fn.__name__, time.perf_counter() - t)
    return wrapper
def retry(n):                            # decorator with arguments = factory returning a decorator
    def deco(fn):
        @functools.wraps(fn)
        def w(*a, **kw):
            for i in range(n):
                try: return fn(*a, **kw)
                except Exception:
                    if i == n - 1: raise
        return w
    return deco
```
**Interview / expert.** Closures capture **variables, not values**: `[lambda: i for i in range(3)]` all return 2; fix with `lambda i=i: i` (**Ex.**). Stacked decorators apply bottom-up. `functools`: `lru_cache` (needs hashable args; default `maxsize=128`), `cache`, `partial`, `cached_property`, `singledispatch`. Class decorators and decorating methods (the wrapper receives `self` as the first argument).

**Pitfalls.** Forgetting `functools.wraps`; decorator that swallows the return value; unbounded `lru_cache(maxsize=None)` memory growth; caching functions with side effects.

---
## 3. Iterators, generators, comprehensions
**ELI15.** A list is a printed book. A generator is a storyteller who tells the next page only when you ask, so you never need the whole book in memory.

**Foundations.** An **iterable** has `__iter__`; an **iterator** has `__next__` and raises `StopIteration` at the end; `for` calls `iter()` then `next()`. A **generator function** contains `yield` and returns a generator iterator; each `yield` suspends and keeps local variables, and it also supports `send()` and `throw()` (**Est.**, glossary). A generator is single-use.
```python
squares = [x*x for x in range(10) if x % 2 == 0]     # eager list
lazy = (x*x for x in range(10**9))                    # lazy, O(1) memory
def fib():
    a, b = 0, 1
    while True: yield a; a, b = b, a + b
def flatten(x):
    for i in x:
        yield from flatten(i) if isinstance(i, list) else [i]
```
**Interview / expert.** `sum(x for x in data)` avoids building a list. `itertools`: `chain`, `islice`, `groupby` (needs sorted input), `product`, `permutations`, `accumulate`. Write your own iterator with `__iter__`/`__next__`. `range` is a lazy sequence, not a list. Generators implement pipelines and are the basis of `async` coroutines historically.

**Pitfalls.** Re-iterating an exhausted generator; `groupby` on unsorted data; list comprehension variable scope (it has its own scope in Python 3).

---
## 4. OOP, inheritance, MRO and the diamond problem
**ELI15.** A class is a blueprint, an object is a house built from it. Inheritance lets a new blueprint start from an old one. The diamond problem asks: if two parent blueprints share a grandparent, whose instructions win, and does the grandparent do its work twice?

**Foundations.** Instance attributes live in `self.__dict__`; class attributes are shared. `@classmethod` gets `cls` (alternative constructors); `@staticmethod` gets nothing. `@property` gives controlled attribute access. `_x` is convention for internal; `__x` is name-mangled to `_Class__x` (avoids clashes, not privacy). Abstract classes: `abc.ABC` + `@abstractmethod` (cannot instantiate until implemented); `typing.Protocol` for structural typing.

**MRO and the diamond (Est.).** Python uses the **C3 linearization** for new-style classes (the official MRO howto): the order merges the parents' linearizations with the list of parents, preserves local precedence and monotonicity, and raises `TypeError` when no consistent order exists. `Class.__mro__` shows it.
```python
class A:
    def hi(self): print("A")
class B(A):
    def hi(self): print("B"); super().hi()
class C(A):
    def hi(self): print("C"); super().hi()
class D(B, C):
    def hi(self): print("D"); super().hi()
D().hi()          # D B C A   (A once)
D.__mro__         # (D, B, C, A, object)
```
`L[D] = D + merge(L[B], L[C], [B, C])` with `L[B]=[B,A,object]`, `L[C]=[C,A,object]` gives `[D, B, C, A, object]` (**Ex.**, computed by hand). `super()` means "next class in the MRO", not "the parent". Inconsistent orders such as `class Z(X, Y)` where `X(A, B)` and `Y(B, A)` raise `TypeError`.

**Interview / expert.** Cooperative multiple inheritance: every class calls `super()` and passes `**kwargs` along. Prefer composition or small mixins over deep hierarchies. Don't call `A.hi(self)` directly in a diamond (can run `A` twice). `__init__` vs `__new__` (`__new__` creates, used for singletons/immutables). Metaclasses create classes (`type`); `__init_subclass__` is often enough. Descriptors implement `property`, methods, `classmethod`.

**Pitfalls.** Treating `super()` as the direct parent; mutable class attributes shared by all instances; overriding `__eq__` and forgetting `__hash__`.

---
## 5. Dunder methods, dataclasses, properties
**Foundations.** `__init__`, `__repr__` (unambiguous, developer) vs `__str__` (user), `__eq__`/`__lt__`/`__hash__`, `__len__`, `__getitem__`, `__iter__`, `__contains__`, `__call__`, `__enter__`/`__exit__`, operator methods (`__add__`, `__radd__`). `functools.total_ordering` fills comparisons.
```python
from dataclasses import dataclass, field
@dataclass(frozen=True, order=True)            # immutable + hashable + ordering
class Point:
    x: int
    y: int = 0
    tags: list = field(default_factory=list)   # never a bare mutable default
```
Choosing: `namedtuple`/`NamedTuple` (immutable record), `dataclass` (boilerplate, optional validation absent), **Pydantic** (runtime validation and serialisation, used by FastAPI), plain class with `__slots__` (memory savings, no `__dict__`) (**Ex.**).

**Pitfalls.** `@dataclass` with a mutable default raises an error unless `default_factory`; `frozen=True` still allows mutating a list field's contents.

---
## 6. Exceptions and context managers
**Foundations.** `try / except / else / finally`: `else` runs only if no exception, `finally` always runs. Catch specific exceptions; bare `except:` also catches `KeyboardInterrupt`. `raise ... from e` chains causes; custom exceptions subclass `Exception`. EAFP ("ask forgiveness") is idiomatic; LBYL ("look before you leap") is the alternative.
**Context managers.** `with` calls `__enter__` and guarantees `__exit__`, even on errors (files, locks, transactions).
```python
from contextlib import contextmanager
@contextmanager
def opened(p):
    f = open(p)
    try: yield f
    finally: f.close()
```
**Pitfalls.** `return` inside `finally` swallows exceptions; catching `Exception` and passing silently; `__exit__` returning True suppresses the error.

---
## 7. Memory, the GIL, and concurrency
**Foundations.** Memory: private heap, **reference counting** frees objects at zero, plus a cyclic garbage collector for reference cycles (**Ex.**). `weakref` references do not keep objects alive.
**GIL (Est.).** The global interpreter lock lets only one thread run Python bytecode at a time in CPython; it simplifies the object model but limits multi-core parallelism for threads; it is released during I/O and by some extension modules during heavy computation. Since Python 3.13 an optional **free-threaded build** (`--disable-gil`) exists (glossary). Treat the 3.13 build as new and experimental until verified for your version.

| Tool | Model | Best for |
|---|---|---|
| `threading` | OS threads, GIL | I/O-bound work with blocking libraries |
| `multiprocessing`, `ProcessPoolExecutor` | separate processes | CPU-bound work (arguments must be picklable) |
| `asyncio` | one thread, cooperative tasks | many concurrent I/O connections |
```python
import asyncio, httpx
async def fetch(c, u): return (await c.get(u)).status_code
async def main(urls):
    async with httpx.AsyncClient() as c:
        return await asyncio.gather(*(fetch(c, u) for u in urls))
asyncio.run(main(["https://example.com"]))
```
**Interview / expert.** Never block the event loop (`time.sleep`, sync DB driver, heavy CPU); use `await asyncio.sleep`, `asyncio.to_thread`, or a worker. `x += 1` on a shared int is not atomic; protect with `threading.Lock`. Deadlock: inconsistent lock order. `asyncio.TaskGroup` (3.11+) gives structured concurrency (**Ex.**). Use `queue.Queue` for producer-consumer between threads.

**Pitfalls.** Using threads to speed up CPU loops; forgetting to `await`; sharing mutable globals across workers (they run in separate processes in web servers).

---
## 8. Performance and complexity of built-ins
**Est. (Python time-complexity page).** list: index O(1), append amortised O(1), pop end O(1), insert/pop at index `k` O(n-k), `x in l` O(n), sort O(n log n). dict/set: get/set/delete/membership O(1) **average** (O(n) worst if all keys collide), set item amortised. `collections.deque` for both ends. Sort is adaptive: already-sorted input takes O(n) comparisons.
**Tips (Ex.).** Membership tests on a `set`, not a list; `''.join(parts)` instead of `+=` in loops; local variable lookups are faster than globals; `functools.lru_cache`; generators for streams; measure with `timeit`, `cProfile`, `py-spy`, `tracemalloc`. Slicing copies. Stability: Python's sort is **stable** and uses Timsort (Sorting HOWTO, **Est.**).

---
## 9. Typing, packaging, testing
- **Type hints** are not enforced at runtime; check with `mypy`/`pyright`; Pydantic validates at runtime. `X | None` syntax from 3.10 (**Ex.**).
- **Modules**: imports cached in `sys.modules`; `if __name__ == "__main__":`; circular imports fixed by restructuring or local imports. Virtual environments isolate dependencies; `pyproject.toml` for metadata (**Ex.**).
- **Testing**: `pytest` fixtures and `parametrize`, `unittest.mock.patch`, `monkeypatch`. Test concurrency bugs with many threads and repeated runs.

---
## 10. Interview coding patterns in Python
```python
from collections import Counter, defaultdict, deque
Counter(words).most_common(3)
groups = defaultdict(list)
for w in words: groups[''.join(sorted(w))].append(w)       # anagram groups
list(dict.fromkeys(items))                                 # dedupe, keep order
def lru_demo():
    from collections import OrderedDict
    class LRU:
        def __init__(s, cap): s.cap, s.d = cap, OrderedDict()
        def get(s, k):
            if k not in s.d: return -1
            s.d.move_to_end(k); return s.d[k]
        def put(s, k, v):
            s.d[k] = v; s.d.move_to_end(k)
            if len(s.d) > s.cap: s.d.popitem(last=False)
```
**"What is the output?" classics (Ex.).** `f(1), f(2)` with a shared default list; `0.1 + 0.2 == 0.3` is False (use `math.isclose`); `[lambda: i ...]` late binding; `(1, [2]) ` tuple mutation; `bool("0")` is True; `-7 // 2` is -4 and `7 % -3` is -2 (floor division and modulo follow the divisor's sign).

**Quick question checklist.** list vs tuple; mutable default arg; `is` vs `==`; shallow vs deep copy; `*args/**kwargs`; decorators and `wraps`; generator vs list; LEGB and closures; MRO and diamond; `__eq__`/`__hash__`; GIL and threading vs multiprocessing vs asyncio; context managers; dataclass vs namedtuple vs Pydantic; how Python manages memory; debugging a slow service (reproduce, profile, find the blocking call or hot loop, fix, re-measure).
