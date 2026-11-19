# DSA Notes E: Design paradigms, DP, strings, NP, interview method (concepts 15 to 19)

Compiled notes, not yet assessed (M0). **Est.** = sourced, **Ex.** = my example. Reminder from the research: DP, greedy, backtracking, union-find and bit tricks are **not named** on the Microsoft page or Google's 2018 PDF; they are standard algorithms content (MIT 6.006 covers DP) and widely reported in interviews, but that last claim is unofficial and unverified here.

---
## 15. Divide and conquer, greedy, backtracking
**ELI15.** Divide and conquer: split a big job, solve parts, combine. Greedy: always grab the best-looking choice right now and never look back. Backtracking: try a choice, go deeper, and undo it if it fails (a maze with chalk).

**Divide and conquer.** Pattern: divide, conquer recursively, combine. Cost via master theorem (concept 4). Examples: merge sort, quicksort, binary search, closest pair, fast exponentiation (`x^n` in O(log n)), maximum subarray (also Kadane).

**Greedy.** Works only when the problem has the **greedy-choice property** and **optimal substructure**. Proof tools: **exchange argument** (swap an optimal solution toward the greedy choice without loss) and "greedy stays ahead".
- Classic correct greedy: interval scheduling (sort by end time), activity selection, fractional knapsack (by value/weight), Huffman coding, Dijkstra, Kruskal/Prim, jump game, gas station, task scheduling by deadlines.
- Classic greedy failure: coin change with coins [1,3,4] and amount 6: greedy gives 4+1+1 (3 coins) but 3+3 uses 2. Use DP. Greedy coin change is correct for some systems (for example standard currency denominations) but not in general.

**Backtracking.** Build solutions incrementally; when a partial solution cannot succeed, abandon it ("prune"). Template:
```python
def subsets(nums):
    res, path = [], []
    def go(i):
        if i == len(nums): res.append(path[:]); return     # copy the path
        go(i + 1)                                         # skip
        path.append(nums[i]); go(i + 1); path.pop()       # take, then undo
    go(0); return res
```
```cpp
void go(int i, vector<int>& a, vector<int>& path, vector<vector<int>>& res){ if(i==(int)a.size()){ res.push_back(path); return;} go(i+1,a,path,res); path.push_back(a[i]); go(i+1,a,path,res); path.pop_back(); }
```
Problems: permutations, combinations, subsets (2^n), N-Queens, Sudoku, word search, generate parentheses, partition into palindromes. Complexity is exponential in general (e.g. permutations n!); pruning and ordering choices reduce it. Handle duplicates by sorting and skipping equal siblings.

**Interview / expert.** Identify the **decision tree** of the problem, the state, the constraints that allow pruning. Memoising a backtracking search turns it into DP. For greedy, always try to find a counterexample before trusting it.

**Pitfalls.** Appending `path` without copying (all results alias one list); forgetting to undo state; trusting a greedy that was not proved.

---
## 16. Dynamic programming
**ELI15.** DP is "don't redo work you already did". Like writing answers to small sums on sticky notes so you can reuse them instead of recomputing.

**Foundations.** DP applies when a problem has **overlapping subproblems** and **optimal substructure**. Two implementations:
- **Top-down (memoisation):** recursion plus a cache.
- **Bottom-up (tabulation):** fill a table in dependency order; allows space optimisation.
Recipe: (1) define the **state** (what do the parameters mean), (2) write the **recurrence**, (3) set **base cases**, (4) decide **order of evaluation**, (5) read the answer, (6) analyse time = states x transition cost, then optimise space.

**Worked example: coin change (minimum coins).** `dp[a]` = fewest coins to make amount `a`; `dp[0] = 0`; `dp[a] = min(dp[a-c] + 1)` over coins `c <= a`. Coins [1,3,4], amount 6: dp = [0,1,2,1,1,2,2], so answer 2. Time O(amount * coins), space O(amount).
```python
def coin_change(coins, amount):
    INF = float('inf'); dp = [0] + [INF] * amount
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]: dp[a] = dp[a - c] + 1
    return dp[amount] if dp[amount] != INF else -1
```
```cpp
int coinChange(vector<int>& coins,int amt){ vector<int> dp(amt+1, amt+1); dp[0]=0; for(int a=1;a<=amt;++a) for(int c:coins) if(c<=a) dp[a]=min(dp[a],dp[a-c]+1); return dp[amt]>amt?-1:dp[amt]; }
```
**Pattern families (Ex.).**
- 1D: climbing stairs, house robber, longest increasing subsequence (O(n^2) DP, O(n log n) with patience sorting/binary search), jump game, word break.
- 2D grid: unique paths, minimum path sum.
- Two sequences: longest common subsequence, edit distance (`dp[i][j]` over prefixes).
- Knapsack: 0/1 knapsack `dp[i][w]`, unbounded, subset sum, partition equal subset.
- Intervals/trees/bitmask: matrix-chain multiplication, burst balloons, DP on trees, travelling-salesman bitmask DP O(2^n * n^2).
- Strings: palindromic substrings, longest palindromic subsequence.

**Interview / expert.** Space optimisation: rolling arrays when a row depends only on the previous row. **Pseudo-polynomial**: knapsack DP is O(n*W), polynomial in the numeric value `W` but exponential in its bit length, so the problem remains NP-hard (concept 18). To reconstruct the solution, store parent choices. Memoisation in Python: `functools.lru_cache`; recursion depth limits apply. A DP vs greedy test: if local choices can be wrong and need comparison across options, use DP.

**Pitfalls.** Wrong state definition (missing a parameter), wrong iteration order, off-by-one in base cases, forgetting INF handling, mutable default in memo dict.

---
## 17. Strings, intervals, bits, basic counting
**Strings.** Immutable in Python (concatenating in a loop is O(n^2); use `''.join`). Techniques: frequency arrays (26 letters), sliding window, two pointers (palindromes, expand around centre O(n^2)), hashing, tries, DP (edit distance, LCS). Pattern matching: naive O(n*m); **KMP** O(n+m) using the failure (prefix) function; Rabin-Karp uses a rolling hash (expected O(n+m)); Z-algorithm. C++ `std::string` is mutable; `s += c` is amortised O(1).
**Intervals.** Sort by start; merge overlapping (compare next start with current end); insert interval; minimum meeting rooms (sort + min-heap of end times, or sweep line of +1/-1 events); non-overlapping intervals (greedy by end).
**Bits.** `x & 1` parity; `x & (x-1)` clears the lowest set bit (count bits, power-of-two test `x > 0 and x & (x-1) == 0`); `x ^ x = 0` and `x ^ 0 = x` (find the unique element); shifts as multiply/divide by 2 (arithmetic shift for negatives differs by language; Python ints are unbounded, C++ shifting a negative signed value has pitfalls). Bitmask subsets enumerate `2^n` sets.
**Counting and probability (Google's PDF says interviewers sometimes ask basic discrete math, **Est.**).** Permutations `n!`, combinations `C(n,k)`, inclusion-exclusion, expected value basics, pigeonhole; reservoir sampling (pick one uniformly from a stream); shuffle (Fisher-Yates).
**Pitfalls.** Off-by-one in interval touching (`[1,2]` and `[2,3]` overlap or not: ask); signed shift/overflow in C++; Unicode vs bytes in Python strings.

---
## 18. NP-completeness: recognising hard problems
**ELI15.** Some problems are easy to *check* but seem impossible to *solve* quickly, like a huge Sudoku: a finished grid is verified in seconds, finding one may take forever.

**Foundations.** **P** = decision problems solvable in polynomial time. **NP** = problems whose "yes" answers can be *verified* in polynomial time. **NP-hard** = at least as hard as every problem in NP. **NP-complete** = in NP and NP-hard. A **polynomial-time reduction** from A to B turns A-instances into B-instances; if B were fast, A would be. Whether P = NP is unsolved.
Classic NP-complete problems: SAT/3-SAT (the first, Cook-Levin), clique, vertex cover, Hamiltonian cycle, subset sum, partition, knapsack (decision version), graph colouring, travelling salesman (decision version). Google's PDF tells candidates to recognise famous NP-complete problems "in disguise" such as travelling salesman and knapsack and to know what NP-complete means (**Est.**).

**What to do when you meet one.** Do not hunt for a polynomial algorithm. Options: exponential search with pruning/DP (bitmask TSP O(2^n n^2), pseudo-polynomial knapsack DP), approximation algorithms (vertex cover 2-approximation, metric TSP 1.5 with Christofides), heuristics (greedy, local search), exploit small input sizes or special structure (trees, bounded width).
**Interview / expert.** Typical flag: "find the best subset/ordering" with n around 20 or less suggests bitmask/backtracking; large n with exactness required suggests the problem is special-case solvable or you misread it. Distinguish "NP-hard" from "exponential time algorithm exists" and from "undecidable".

---
## 19. Interview method: clarify, brute force, optimise, code, test, communicate
Both official sources agree on the process (Microsoft page; Google 2018 PDF, **Est.**):
1. **Clarify**: ask questions, state assumptions (input size, duplicates, negatives, empty input, sorted?, memory limits, return format). Problems are often deliberately underspecified.
2. **Examples**: work one small and one edge example by hand.
3. **Brute force first**: state it and its complexity; "get to the simplest solution quickly".
4. **Optimise** aloud: find the bottleneck; ask which data structure removes it (hash map for lookup, sorting for order, heap for extremes, two pointers/sliding window for scans, DP for overlap, graph model for relationships). Discuss time/space trade-offs.
5. **Code real code** in your strongest language (no pseudocode); clean names, small helper functions. If you forget a library signature, say so and assume a sensible one.
6. **Test your own code** with examples, then edge cases (empty, one element, duplicates, negatives, large values, cycles), by tracing, without prompting.
7. **State final complexity** (time and space) and mention improvements.
8. **Think aloud** and use interviewer hints; Google's guide stresses explaining thought process, Microsoft's stresses clarifying and testing.

**Pattern recognition cheat sheet (Ex.).**
| Signal in the problem | Try |
|---|---|
| sorted input, find a value or boundary | binary search |
| pairs/triplets summing to a target | sort + two pointers, or hash map |
| contiguous subarray/substring with a condition | sliding window or prefix sums |
| next greater/smaller element | monotonic stack |
| top-K / k-th / streaming extremes | heap |
| shortest path unweighted / levels | BFS |
| connectivity, grouping, cycles | DFS or union-find |
| dependencies, ordering | topological sort |
| count ways / min-max over choices with overlap | DP |
| generate all combinations/arrangements | backtracking |
| frequency, duplicates, lookup | hash map/set |
| intervals | sort by start or end |

**Pitfalls.** Silent coding; jumping to the optimal solution without explaining; skipping tests; ignoring constraints; unreadable code. Practice on paper or in a plain editor, with the language you will use in the interview.
