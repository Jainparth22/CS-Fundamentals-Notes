# DSA Notes A: Foundations (concepts 1 to 4)

Compiled notes, not yet assessed (mastery M0). Format per concept: ELI15, Foundations, Working knowledge, Interview and expert depth, Pitfalls. Labels: **Est.** = sourced, **Ex.** = my example. Sources are listed in `SOURCES.md`.

---
## 1. What an algorithm is, cost model, input size
**ELI15.** An algorithm is a recipe a computer can follow without thinking. Two recipes can give the same answer but one may take a second and another a year when the input grows. We care about how the effort *grows*.

**Foundations.** An algorithm is a finite, well-defined sequence of steps that maps an input to an output. We measure cost as a function of input size `n` (items, bits, vertices+edges), not in seconds, because seconds depend on the machine. The standard model is the **word-RAM**: each basic operation (add, compare, array index) costs one unit. (MIT 6.006 builds its analysis on a model of computation, **Est.**)

**Working knowledge.**
- Time vs space: count steps and extra memory separately.
- Cases: worst case (default in interviews), average case (needs a distribution), best case (rarely useful).
- "Size" must be defined: for a graph it is V and E; for a number `n` the input size is its bit length, which matters for pseudo-polynomial algorithms (see DP).

**Interview / expert.** State the model out loud when it matters (for example "hash lookups O(1) average"). Python ints are arbitrary precision so `a + b` is not O(1) for huge numbers; C++ `int` overflows silently (undefined behaviour for signed). Real speed also depends on cache locality, which Big-O hides (arrays beat linked lists in practice).

**Pitfalls.** Mixing up input value and input size; treating hash operations as always O(1); ignoring that string concatenation or slicing copies O(len).

---
## 2. Asymptotic analysis (Big-O, Omega, Theta)
**ELI15.** Big-O is a speed limit on growth: "no matter how big the input, the work will not grow faster than this curve (up to a constant)".

**Foundations.** `f(n) = O(g(n))` if there exist constants `c, n0` with `f(n) <= c*g(n)` for all `n >= n0`. `Omega` is a lower bound, `Theta` is both (tight). Drop constants and lower-order terms: `3n + 50 = O(n)`.
Growth order: `1 < log n < sqrt(n) < n < n log n < n^2 < n^3 < 2^n < n!`.

**Working knowledge. Analysing code.**
- Single loop to `n`: O(n). Nested loops: multiply. Loop where the variable doubles or halves: O(log n).
- `for i in range(n): for j in range(i)`: inner runs `0+1+...+(n-1) = n(n-1)/2` times: O(n^2) (**Ex.**).
- Sequential blocks add, the largest dominates. Recursion: write a recurrence (concept 4).
- Space includes recursion stack depth.
- Typical constraints to complexity: n <= 10^5 needs about O(n log n); n <= 5000 allows O(n^2); n <= 20 allows O(2^n) (a rule of thumb, **Ex.**).

**Interview / expert.** Say "worst case" explicitly. Distinguish `O` (upper bound) from `Theta` (tight); saying "binary search is O(n)" is true but useless. Amortised and expected costs are different from worst case. Little-o and little-omega exist but rarely appear.

**Pitfalls.** `O(n)` does not mean "exactly n steps"; `log` base does not matter inside O; two inputs need two variables (`O(n*m)`), not one `n`.

---
## 3. Arrays and dynamic arrays; amortised analysis
**ELI15.** An array is a row of numbered lockers side by side, so you can jump to locker 500 directly. A dynamic array is a row that, when full, moves into a bigger row.

**Foundations.** A static array stores elements contiguously, so `a[i]` is address arithmetic: O(1). Insert or delete in the middle shifts elements: O(n). A **dynamic array** (Python `list`, C++ `std::vector`) keeps spare capacity; when full it allocates a larger block and copies.

**Amortised analysis (key idea).** If capacity doubles when full, copying cost over `n` appends is `1 + 2 + 4 + ... < 2n`, so average cost per append is O(1) even though a single append may cost O(n). Methods: aggregate (above), accounting (each append pays 3 coins, 2 saved for future copying), potential. 

| Operation | Python list (CPython) | C++ vector |
|---|---|---|
| index `a[i]` | O(1) | O(1) |
| append / push_back | O(1) amortised | O(1) amortised |
| pop end | O(1) | O(1) |
| insert/pop at index k | O(n-k) (front: O(n)) | O(n-k) |
| `x in a` | O(n) | `std::find` O(n) |
| sort | O(n log n) | `std::sort` O(n log n) |
Python figures are from the official time-complexity page (**Est.**). Growth factor is an implementation detail: CPython over-allocates by about 12.5% plus a constant; GCC's vector doubles, MSVC uses 1.5x. Any constant factor greater than 1 keeps appends amortised O(1) (**Ex.**, standard result).

```python
a = [3, 1, 4]; a.append(1); a.insert(0, 9); a.pop()       # list
from collections import deque; d = deque(); d.appendleft(1) # O(1) both ends
```
```cpp
std::vector<int> v{3,1,4}; v.push_back(1); v.insert(v.begin(), 9); v.pop_back();
```

**Interview / expert.** `vector` reallocation invalidates iterators and references (cppreference containers page, **Est.**). Python slicing copies (`a[1:]` is O(n)), a frequent hidden cost. Use `deque` for queue behaviour. Pre-size with `vector::reserve`. 2D arrays: row-major locality; `[[0]*m]*n` in Python aliases rows (bug).

**Pitfalls.** Calling insert-at-front O(1); forgetting that amortised is not worst case per operation; off-by-one in bounds.

---
## 4. Recursion, induction, recurrences
**ELI15.** Recursion solves a big problem by solving smaller copies of the same problem, with a stopping rule. Like looking up a word's meaning that uses another word, until you reach words you know.

**Foundations.** A recursive function needs a **base case** and a **recursive step** that moves toward the base. Correctness is proved by **induction**: assume it works for smaller inputs, prove for the current one. Each call uses a stack frame, so depth costs space.

**Recurrences.** Write cost as `T(n)`.
- Linear recursion `T(n) = T(n-1) + O(1)` gives O(n). `T(n) = T(n-1) + n` gives O(n^2).
- Divide and conquer `T(n) = a*T(n/b) + f(n)`. **Master theorem** (compare `f(n)` with `n^(log_b a)`):
  1. if `f` is polynomially smaller: `T = Theta(n^(log_b a))`;
  2. if `f = Theta(n^(log_b a))`: `T = Theta(n^(log_b a) * log n)`;
  3. if `f` is polynomially larger (and regularity holds): `T = Theta(f(n))`.
  - Merge sort `2T(n/2)+n` gives Theta(n log n). Binary search `T(n/2)+1` gives Theta(log n). Naive Fibonacci `T(n-1)+T(n-2)` is exponential (not master-theorem form).
- Recursion tree method: sum the work per level.

```python
def fact(n):               # base case first
    return 1 if n <= 1 else n * fact(n - 1)
```
```cpp
long long fact(int n){ return n <= 1 ? 1 : n * fact(n-1); }
```

**Interview / expert.** Python's default recursion limit is about 1000 (`sys.getrecursionlimit()`), so deep recursion needs an iterative rewrite or a raised limit; C++ risks stack overflow around 10^5 depth depending on frame size. **Tail recursion** is not optimised in CPython. Memoisation turns exponential recursion into polynomial (concept 16). Convert recursion to an explicit stack for DFS on deep graphs.

**Pitfalls.** Missing or wrong base case, recursion that does not shrink, recomputing overlapping subproblems, forgetting stack space in the complexity.
