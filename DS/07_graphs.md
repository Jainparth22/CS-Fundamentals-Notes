# Graphs

> **Core idea:** Nodes (vertices) connected by edges. BFS = shortest path unweighted, DFS = connectivity/cycles/ordering, Dijkstra = weighted shortest path, Union-Find = dynamic connectivity.

---

## 1. Graph Representations

### Adjacency List (most common)
```cpp
int n = 5;
vector<vector<int>> adj(n);     // unweighted
adj[0].push_back(1);
adj[1].push_back(2);

vector<vector<pair<int,int>>> wadj(n); // weighted: {neighbor, weight}
wadj[0].push_back({1, 7});
```

### Adjacency Matrix
```cpp
vector<vector<int>> mat(n, vector<int>(n, 0));
mat[u][v] = 1;  // or weight
// Good for dense graphs, O(1) edge lookup, O(n^2) space
```

### Edge List
```cpp
vector<tuple<int,int,int>> edges; // {weight, u, v} — good for Kruskal
```

---

## 2. BFS — Shortest Path (Unweighted)

```cpp
vector<int> bfs(int src, int n, vector<vector<int>>& adj) {
    vector<int> dist(n, -1);
    queue<int> q;
    dist[src] = 0;
    q.push(src);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }
    return dist;
}
```

**Multi-source BFS:** Push all sources into queue initially with dist = 0. Used in: Rotting Oranges, 0-1 Matrix.

---

## 3. DFS — Connectivity & Cycle Detection

```cpp
vector<bool> visited;

void dfs(int u, vector<vector<int>>& adj) {
    visited[u] = true;
    for (int v : adj[u]) {
        if (!visited[v]) dfs(v, adj);
    }
}

int countComponents(int n, vector<vector<int>>& adj) {
    visited.assign(n, false);
    int count = 0;
    for (int i = 0; i < n; i++)
        if (!visited[i]) { dfs(i, adj); count++; }
    return count;
}
```

### Cycle Detection (Directed Graph)
```cpp
// 0 = unvisited, 1 = in-progress, 2 = done
vector<int> state;
bool hasCycle;

void dfs(int u, vector<vector<int>>& adj) {
    state[u] = 1;  // in-progress
    for (int v : adj[u]) {
        if (state[v] == 1) { hasCycle = true; return; }
        if (state[v] == 0) dfs(v, adj);
    }
    state[u] = 2;  // done
}
```

---

## 4. Topological Sort

Only for Directed Acyclic Graphs (DAGs).

### Kahn's Algorithm (BFS) — preferred for interview
```cpp
vector<int> topoSort(int n, vector<vector<int>>& adj) {
    vector<int> indegree(n, 0);
    for (int u = 0; u < n; u++)
        for (int v : adj[u]) indegree[v]++;

    queue<int> q;
    for (int i = 0; i < n; i++) if (indegree[i] == 0) q.push(i);

    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u])
            if (--indegree[v] == 0) q.push(v);
    }
    return (order.size() == n) ? order : vector<int>{}; // empty = cycle
}
```

---

## 5. Dijkstra's Algorithm — O((V+E) log V)

```cpp
vector<int> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    dist[src] = 0;
    pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;  // outdated entry
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

**Pitfall:** Only works with non-negative edge weights. Use Bellman-Ford for negative edges.

---

## 6. Union-Find (Disjoint Set Union)

```cpp
struct DSU {
    vector<int> parent, rank_;
    int components;

    DSU(int n) : parent(n), rank_(n, 0), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }

    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (rank_[a] < rank_[b]) swap(a, b);
        parent[b] = a;
        if (rank_[a] == rank_[b]) rank_[a]++;
        components--;
        return true;
    }
};
```

**Complexity:** α(n) ≈ O(1) amortized with path compression + union by rank.

---

## 7. Minimum Spanning Tree

### Kruskal's — sort edges, greedily add if no cycle (use DSU)
```cpp
int kruskal(int n, vector<tuple<int,int,int>>& edges) { // {weight, u, v}
    sort(edges.begin(), edges.end());
    DSU dsu(n);
    int total = 0;
    for (auto [w, u, v] : edges)
        if (dsu.unite(u, v)) total += w;
    return total;
}
```

---

## 8. Practice Problems

| Problem | Technique | LC # |
|---------|-----------|------|
| Number of Islands | DFS/BFS flood fill | 200 |
| Course Schedule | Topo sort (cycle detection) | 207 |
| Course Schedule II | Topo sort (Kahn's) | 210 |
| Redundant Connection | DSU cycle detection | 684 |
| Network Delay Time | Dijkstra | 743 |
| Min Cost to Connect Points | Kruskal / Prim | 1584 |
| Word Ladder | BFS on word graph | 127 |
| Cheapest Flights K Stops | Bellman-Ford / BFS with states | 787 |
| Pacific Atlantic Water Flow | Reverse DFS from both oceans | 417 |
