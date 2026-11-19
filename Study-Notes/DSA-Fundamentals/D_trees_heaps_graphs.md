# DSA Notes D: Trees, heaps, graphs (concepts 11 to 14)

Compiled notes, not yet assessed (M0). **Est.** = sourced, **Ex.** = my example.

---
## 11. Trees: traversals, BST, tries
**ELI15.** A tree is a family tree or folder structure: one root, each node has children, no loops.

**Foundations.** Binary tree: each node has at most 2 children. Height `h`; a balanced tree has `h = O(log n)`, a degenerate (linked-list-like) tree has `h = n`. Types: full, complete, perfect, balanced, BST.
**Traversals** (Google's PDF names BFS/DFS and inorder/preorder/postorder, **Est.**):
- DFS: preorder (node, L, R), inorder (L, node, R), postorder (L, R, node). BFS = level order using a queue. All O(n) time; DFS space O(h), BFS space O(width).
```python
def inorder(t):                      # recursive, O(h) stack
    return inorder(t.left) + [t.val] + inorder(t.right) if t else []
from collections import deque
def levels(root):
    q, out = deque([root] if root else []), []
    while q:
        out.append([n.val for n in q])
        q = deque(c for n in q for c in (n.left, n.right) if c)
    return out
```
```cpp
struct T{int v;T*l=nullptr;T*r=nullptr;};
void inorder(T* t, vector<int>& o){ if(!t) return; inorder(t->l,o); o.push_back(t->v); inorder(t->r,o); }
```

**BST.** For every node, left subtree < node < right subtree. Inorder traversal of a BST is sorted. Search/insert/delete O(h): O(log n) if balanced, O(n) worst. Delete: leaf (remove), one child (splice), two children (replace with inorder successor). Validate with bounds `(lo, hi)`, not just by comparing a node to its children.
**Balanced trees (concept level).** AVL (height difference of subtrees at most 1; rotations) and red-black trees keep height O(log n); C++ `std::map`/`set` are typically red-black trees (implementation detail). B-trees/B+ trees are used in databases for disk-friendly lookup. Python has no built-in balanced BST.
**Trie (prefix tree).** Each edge is a character; insert, search, prefix-search cost O(L) for a word of length L, independent of the number of words. Used for autocomplete and dictionary problems. Memory is the cost. Microsoft's page names "trees and tries" (**Est.**).

**Common problems.** Max depth, diameter, lowest common ancestor (BST: by value; general: recursion returns found nodes), path sum, serialise/deserialise, level-order, right-side view, kth smallest in BST (inorder), construct from preorder + inorder.

**Interview / expert.** Recursion depth equals tree height (skewed trees can overflow the stack); use an iterative traversal with a stack, or Morris traversal for O(1) space. Segment trees and Fenwick (BIT) trees answer range queries with updates in O(log n); treat as advanced.

**Pitfalls.** Assuming a tree is balanced; validating BST with only parent-child checks; forgetting the null base case; modifying a tree while traversing.

---
## 12. Heaps and priority queues
**ELI15.** A priority queue is a hospital ER line: the most urgent person goes next, not the first to arrive. A heap is a clever tree-in-an-array that always keeps the most urgent at the top.

**Foundations.** A **binary heap** is a complete binary tree stored in an array where each parent <= its children (min-heap). Index math: parent `(i-1)//2`, children `2i+1`, `2i+2`. Operations: `push` (add at end, sift up) O(log n); `pop` min (swap root with last, sift down) O(log n); `peek` O(1); **build-heap** from n items is O(n) (not O(n log n)); heap sort is O(n log n).
- Python: `heapq` is a **min-heap** on a list (`heappush`, `heappop`, `heapify`); for max-heap push negatives.
- C++: `std::priority_queue` is a **max-heap** by default; min-heap with `priority_queue<int, vector<int>, greater<int>>`.
```python
import heapq
h = []; [heapq.heappush(h, x) for x in (5, 1, 3)]; heapq.heappop(h)   # 1
def top_k(nums, k):                      # k largest in O(n log k)
    h = []
    for x in nums:
        heapq.heappush(h, x)
        if len(h) > k: heapq.heappop(h)
    return h
```
```cpp
priority_queue<int, vector<int>, greater<int>> pq; pq.push(5); pq.push(1); int m = pq.top(); pq.pop();
```
**Patterns.** Top-K (size-k heap), K-th largest, merge K sorted lists (heap of k heads: O(N log k)), running median (max-heap + min-heap), scheduling/meeting rooms, Dijkstra, Huffman coding. Google's PDF says to know heaps' applications and O() characteristics (**Est.**).

**Interview / expert.** Searching an arbitrary element in a heap is O(n). Decrease-key is not supported by `heapq`/`priority_queue`; Dijkstra uses lazy deletion (push duplicates, skip stale entries). Python tuples compare element by element, so ties can fall through to comparing payloads that are not comparable; add a counter as tiebreaker.

**Pitfalls.** Using `heapq` as a max-heap without negating; heapifying then assuming the list is sorted (it is not); confusing heap with BST.

---
## 13. Graphs: representation, BFS, DFS, components, cycles, topological sort
**ELI15.** A graph is dots (vertices) joined by lines (edges): cities and roads, people and friendships, tasks and prerequisites.

**Foundations.** `G = (V, E)`; directed or undirected; weighted or not; cyclic or acyclic; connected. **Representations** (Google's PDF names three: objects and pointers, matrix, adjacency list, **Est.**):
| Representation | Space | Edge check | Iterate neighbours | Good for |
|---|---|---|---|---|
| Adjacency list | O(V+E) | O(deg) | O(deg) | sparse graphs (default) |
| Adjacency matrix | O(V^2) | O(1) | O(V) | dense graphs, Floyd-Warshall |
| Edge list | O(E) | O(E) | O(E) | Kruskal, Bellman-Ford |

**BFS and DFS: O(V + E)** with an adjacency list.
```python
from collections import deque
def bfs(adj, s):                          # shortest path in edges, unweighted
    dist = {s: 0}; q = deque([s])
    while q:
        u = q.popleft()
        for v in adj[u]:
            if v not in dist: dist[v] = dist[u] + 1; q.append(v)
    return dist
def dfs(adj, u, seen):
    seen.add(u)
    for v in adj[u]:
        if v not in seen: dfs(adj, v, seen)
```
```cpp
vector<int> bfs(const vector<vector<int>>& g,int s){ vector<int> d(g.size(),-1); queue<int> q; d[s]=0; q.push(s); while(!q.empty()){ int u=q.front(); q.pop(); for(int v:g[u]) if(d[v]<0){ d[v]=d[u]+1; q.push(v);} } return d; }
```
**Why BFS gives shortest paths in unweighted graphs:** it explores in order of increasing distance (queue = FIFO by level); DFS goes deep first and may reach a vertex by a long path first.

**Applications.** Connected components (run search from each unvisited vertex); grid problems (number of islands, shortest path in a maze; neighbours are 4 or 8 cells, mark visited); cycle detection (undirected: a visited neighbour that is not the parent; directed: DFS with three colours or a recursion stack); bipartite check (2-colour with BFS); flood fill; multi-source BFS (rotting oranges); word ladder.
**Topological sort** (DAG only): order vertices so every edge goes from earlier to later. Kahn's algorithm: repeatedly remove vertices with indegree 0 (O(V+E)); or DFS postorder reversed. If not all vertices are removed, there is a cycle (course schedule problem).

**Interview / expert.** Many problems become graph problems "in disguise" (Google's PDF: modelling a problem as a graph often yields a standard algorithm, **Est.**). Choose BFS for shortest unweighted paths and level structure, DFS for connectivity, cycles, ordering, backtracking. Recursion depth can reach V in DFS; use an explicit stack for large graphs. Implicit graphs (states as vertices) cover puzzles and word transformations.

**Pitfalls.** Marking visited when popping instead of when pushing (blows up the queue and time); forgetting disconnected components; treating undirected edges as directed (add both directions); using BFS for weighted shortest paths.

---
## 14. Shortest paths, spanning trees, union-find
**ELI15.** Shortest path is Google Maps. A minimum spanning tree connects all towns with the least total road. Union-find answers "are these two people in the same friend group?" fast.

**Shortest paths.**
| Algorithm | Handles | Time (with typical structure) |
|---|---|---|
| BFS | unweighted | O(V+E) |
| 0-1 BFS (deque) | weights 0 or 1 | O(V+E) |
| Dijkstra | non-negative weights, single source | O((V+E) log V) with a binary heap |
| Bellman-Ford | negative edges, detects negative cycles | O(V*E) |
| Floyd-Warshall | all pairs | O(V^3) |
| Topological-order DP | DAG | O(V+E) |
Dijkstra idea (greedy): repeatedly finalise the closest unfinalised vertex; it is correct only with non-negative weights because a later edge can never shorten an already finalised path.
```python
import heapq
def dijkstra(adj, s):                     # adj[u] = [(v, w), ...]
    dist = {s: 0}; h = [(0, s)]
    while h:
        d, u = heapq.heappop(h)
        if d > dist.get(u, float('inf')): continue     # stale entry
        for v, w in adj[u]:
            nd = d + w
            if nd < dist.get(v, float('inf')): dist[v] = nd; heapq.heappush(h, (nd, v))
    return dist
```
**Minimum spanning tree** (connected, undirected, weighted; V-1 edges, no cycle, min total weight): **Kruskal** sorts edges and adds the next edge that joins two components (union-find): O(E log E). **Prim** grows one tree using a heap: O(E log V). The cut property justifies both greedy choices.

**Union-find (disjoint set union).** Operations `find(x)` and `union(a, b)`. With **path compression** and **union by rank/size**, amortised cost per operation is O(alpha(n)), the inverse Ackermann function, effectively constant.
```python
class DSU:
    def __init__(s, n): s.p = list(range(n)); s.sz = [1]*n
    def find(s, x):
        while s.p[x] != x: s.p[x] = s.p[s.p[x]]; x = s.p[x]    # path halving
        return x
    def union(s, a, b):
        a, b = s.find(a), s.find(b)
        if a == b: return False
        if s.sz[a] < s.sz[b]: a, b = b, a
        s.p[b] = a; s.sz[a] += s.sz[b]; return True
```
Uses: connected components, cycle detection in undirected graphs, Kruskal, accounts merge, redundant connection.

**Interview / expert.** State why Dijkstra fails with negative edges and when to switch to Bellman-Ford. For grids with uniform cost use BFS; for A* add a heuristic (admissible and consistent). Know the difference between shortest path (weights) and fewest edges. Network-flow and strongly-connected components are advanced extras (Stanford CS161 lists flow as a possible topic, snippet only).

**Pitfalls.** Dijkstra without skipping stale heap entries (still correct but slower); negative weights; integer infinity overflow in C++ (`INT_MAX + w`); using union-find on directed connectivity.
