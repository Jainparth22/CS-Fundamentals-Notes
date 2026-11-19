# Python: 25 Most-Asked Questions (with answers)

Questions are the ones that recur across public Python interview lists (see `README.md` for sources). Answers are written for revision; verify details against the official Python docs.

**Q1. List vs tuple?**
Lists are mutable, tuples are immutable. Tuples are hashable when their items are, so they work as dict keys; they are slightly smaller and faster. Use a tuple for a fixed record and a list for a changing collection.

**Q2. Which types are mutable and which are immutable?**
Mutable: list, dict, set, bytearray, most user classes. Immutable: int, float, bool, str, tuple, frozenset, bytes. Immutable does not mean the contents of a tuple's mutable items cannot change.

**Q3. What is the mutable default argument problem?**
A default like `def f(x, items=[])` is created once when the function is defined, so every call shares the same list. Use `None` as the default and create a new list inside the function.

**Q4. How does a dict work internally?**
It is a hash table with open addressing and a compact array that preserves insertion order (3.7+). Keys must be hashable; lookup, insert and delete are O(1) on average and resizing is amortised.

**Q5. Shallow copy vs deep copy?**
A shallow copy duplicates the outer container but shares the inner objects; a deep copy recursively duplicates everything (`copy.deepcopy`). Use deep copy for nested mutable structures you will change independently.

**Q6. `==` vs `is`?**
`==` compares values through `__eq__`; `is` compares object identity. Use `is` for `None`, `True` and `False`. Small-int and string caching can make `is` look like it works by accident.

**Q7. What are `*args` and `**kwargs`?**
`*args` collects extra positional arguments into a tuple; `**kwargs` collects extra keyword arguments into a dict. They also unpack when calling: `f(*lst, **d)`.

**Q8. What is a list comprehension and how does a generator expression differ?**
A comprehension builds a list eagerly in one expression. A generator expression `(x for x in data)` is lazy and uses constant memory, but can be consumed only once.

**Q9. Iterable vs iterator?**
An iterable has `__iter__` and can produce an iterator (list, str, dict). An iterator has `__next__` and keeps state, raising `StopIteration` when finished. A `for` loop calls `iter()` then `next()` repeatedly.

**Q10. Explain scope (LEGB) and closures.**
Names resolve Local, Enclosing, Global, then Built-in. A closure is an inner function that remembers variables from its enclosing scope; use `nonlocal` to rebind them. Beware late binding in loops (`lambda: i`).

**Q11. What is a decorator and how do you write one?**
A function that takes a function and returns a wrapped function, applied with `@name`. Always use `functools.wraps` to preserve metadata. A decorator with arguments is a function returning a decorator.

**Q12. What are generators and how does `yield` work?**
A function with `yield` returns a generator; each `next()` runs to the next `yield` and pauses, keeping local state. They give lazy, memory-efficient pipelines and support `send()` and `yield from`.

**Q13. What is the GIL?**
A lock in CPython allowing one thread to run Python bytecode at a time, protecting reference counting. It limits CPU-bound threading but not I/O-bound work. Use multiprocessing or native extensions for CPU work.

**Q14. Threading, multiprocessing or asyncio: when?**
Threads for blocking I/O with simple code, multiprocessing for CPU-bound work, asyncio for very many concurrent I/O tasks with async libraries. Never block the event loop with sync calls.

**Q15. How does Python manage memory?**
A private heap with reference counting (objects freed at zero) plus a generational cyclic garbage collector for reference cycles. `weakref` avoids keeping objects alive; `tracemalloc` helps find leaks.

**Q16. `@classmethod` vs `@staticmethod` vs instance method?**
Instance methods receive `self`; class methods receive `cls` (alternative constructors, class-level behaviour); static methods receive nothing and are just namespaced functions.

**Q17. What are dunder (magic) methods?**
Special methods Python calls implicitly: `__init__`, `__repr__`, `__str__`, `__eq__`, `__hash__`, `__len__`, `__iter__`, `__getitem__`, `__call__`, `__enter__/__exit__`, operator methods like `__add__`.

**Q18. How does MRO work with multiple inheritance (the diamond problem)?**
Python uses C3 linearization to produce one consistent order (`Class.__mro__`) where each class appears once and parents follow children. `super()` follows that order, so a shared base runs once.

**Q19. What are abstract base classes?**
Classes using `abc.ABC` and `@abstractmethod` that cannot be instantiated until subclasses implement the abstract methods. They define contracts; `typing.Protocol` gives structural (duck-typed) contracts.

**Q20. What does `@property` do, and what are descriptors?**
`@property` exposes a method as an attribute with controlled get/set. It is built on descriptors: objects defining `__get__`/`__set__`; methods, classmethods and properties are all descriptors.

**Q21. How do context managers work?**
`with` calls `__enter__` and guarantees `__exit__` runs, even on exceptions, for cleanup (files, locks, transactions). Write one with a class or `@contextlib.contextmanager`.

**Q22. What is the contract between `__eq__` and `__hash__`?**
Equal objects must have equal hashes, and the hash must not change while the object is in a set or dict. Defining `__eq__` without `__hash__` makes instances unhashable.

**Q23. How does `functools.lru_cache` work and where can it hurt?**
It memoises results keyed by arguments (must be hashable) with a least-recently-used bound. Pitfalls: unbounded caches leak memory, methods hold references to `self`, and cached results can go stale.

**Q24. dataclass vs namedtuple vs Pydantic?**
`namedtuple` is an immutable lightweight tuple; `@dataclass` generates boilerplate for mutable (or frozen) classes with no validation; Pydantic validates and coerces data at runtime and serialises models (used in FastAPI).

**Q25. A Python service is slow or misbehaving in production. How do you debug it?**
Reproduce and measure first: logs and metrics, then profile (`cProfile`, `py-spy`), check I/O waits, DB queries, blocking calls in async code, memory growth (`tracemalloc`), and concurrency (GIL, lock contention). Fix the biggest bottleneck, then re-measure.
