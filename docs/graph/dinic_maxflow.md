# Dinic's Maximum Flow Algorithm

## 1. Overview

Dinic's algorithm (also known as Dinitz's algorithm) computes maximum flow in O(V²E) time. It improves upon Edmonds-Karp by finding multiple augmenting paths in each phase using the concept of **blocking flows** on **level graphs**. Published by Yefim Dinitz in 1970.

## 2. Mathematical Foundation

### 2.1 Level Graph

A **level graph** $L_G$ is constructed from the residual graph where:
- $level(s) = 0$ (source)
- $level(v) = $ shortest path distance from $s$ to $v$
- Only edges $(u, v)$ where $level(v) = level(u) + 1$ are kept

### 2.2 Blocking Flow

A **blocking flow** saturates at least one edge on every path from $s$ to $t$ in the level graph.

### 2.3 Key Theorem

After at most $O(V)$ phases of finding blocking flows, maximum flow is achieved.

## 3. Algorithm

### 3.1 Phases

1. **Build Level Graph:** BFS from source
2. **Find Blocking Flow:** DFS with optimization
3. **Repeat** until sink is unreachable

### 3.2 Pseudocode

```
DINIC(G, s, t):
    flow ← 0
    while BFS-LEVEL-GRAPH(s, t) succeeds:
        while blocking_flow ← DFS-BLOCKING-FLOW(s, t, ∞) > 0:
            flow += blocking_flow
    return flow

BFS-LEVEL-GRAPH(s, t):
    level[all] ← -1
    level[s] ← 0
    queue ← [s]
    while queue not empty:
        u ← queue.dequeue()
        for each edge (u, v) with residual > 0:
            if level[v] = -1:
                level[v] ← level[u] + 1
                queue.enqueue(v)
    return level[t] ≠ -1

DFS-BLOCKING-FLOW(u, t, pushed):
    if u = t:
        return pushed
    for edges (u, v) starting from iter[u]:
        if level[v] = level[u] + 1 and residual(u,v) > 0:
            d ← DFS(v, t, min(pushed, residual))
            if d > 0:
                update flows
                return d
        iter[u]++  // Optimization: skip saturated edges
    return 0
```

## 4. Example

```
    10      10
s ────→ a ────→ t
│       ↑       ↑
│   4   │   10  │
└───→ b ┴───────┘
    10
```

**Phase 1:** Level graph with levels s(0), a(1), b(1), t(2)
- Blocking flow: 20 (paths s→a→t: 10, s→b→t: 10)

**Maximum Flow: 20**

## 5. Complexity

| Metric | Complexity |
|--------|------------|
| Time (general) | O(V²E) |
| Time (unit capacity) | O(E√E) |
| Time (bipartite) | O(E√V) |
| Space | O(V + E) |

## 6. Implementation

```rust
use std::collections::VecDeque;

#[derive(Clone)]
pub struct Edge {
    pub to: usize,
    pub cap: i64,
    pub flow: i64,
}

pub struct Dinic {
    graph: Vec<Vec<usize>>,  // Adjacency list (indices into edges)
    edges: Vec<Edge>,
    level: Vec<i32>,
    iter: Vec<usize>,
}

impl Dinic {
    pub fn new(n: usize) -> Self {
        Dinic {
            graph: vec![vec![]; n],
            edges: vec![],
            level: vec![0; n],
            iter: vec![0; n],
        }
    }

    pub fn add_edge(&mut self, from: usize, to: usize, cap: i64) {
        self.graph[from].push(self.edges.len());
        self.edges.push(Edge { to, cap, flow: 0 });
        
        self.graph[to].push(self.edges.len());
        self.edges.push(Edge { to: from, cap: 0, flow: 0 });  // Reverse edge
    }

    fn bfs(&mut self, s: usize, t: usize) -> bool {
        self.level.fill(-1);
        self.level[s] = 0;
        
        let mut queue = VecDeque::new();
        queue.push_back(s);
        
        while let Some(u) = queue.pop_front() {
            for &idx in &self.graph[u] {
                let e = &self.edges[idx];
                if self.level[e.to] < 0 && e.cap > e.flow {
                    self.level[e.to] = self.level[u] + 1;
                    queue.push_back(e.to);
                }
            }
        }
        
        self.level[t] >= 0
    }

    fn dfs(&mut self, u: usize, t: usize, pushed: i64) -> i64 {
        if u == t || pushed == 0 {
            return pushed;
        }
        
        while self.iter[u] < self.graph[u].len() {
            let idx = self.graph[u][self.iter[u]];
            let e = &self.edges[idx];
            
            if self.level[e.to] == self.level[u] + 1 && e.cap > e.flow {
                let to = e.to;
                let residual = e.cap - e.flow;
                let d = self.dfs(to, t, pushed.min(residual));
                
                if d > 0 {
                    self.edges[idx].flow += d;
                    self.edges[idx ^ 1].flow -= d;  // Reverse edge at idx ^ 1
                    return d;
                }
            }
            self.iter[u] += 1;
        }
        0
    }

    pub fn max_flow(&mut self, s: usize, t: usize) -> i64 {
        let mut flow = 0;
        
        while self.bfs(s, t) {
            self.iter.fill(0);
            loop {
                let pushed = self.dfs(s, t, i64::MAX);
                if pushed == 0 { break; }
                flow += pushed;
            }
        }
        
        flow
    }
}
```

## 7. Key Optimizations

### 7.1 Edge Pointer Optimization

```rust
iter: Vec<usize>  // Current edge pointer for each vertex
```

Instead of iterating all edges each DFS call, maintain a pointer to skip already-processed edges.

### 7.2 XOR Trick for Reverse Edges

```rust
self.edges[idx ^ 1]  // Reverse edge
```

If edges are added in pairs (forward, reverse), the reverse edge index is `idx ^ 1`.

### 7.3 Early Termination

Stop DFS immediately when:
- Reach sink (return flow)
- No more flow possible (return 0)

## 8. Dinic's vs Other Algorithms

| Algorithm | Time | Best for |
|-----------|------|----------|
| Ford-Fulkerson | O(Ef) | Small flows |
| Edmonds-Karp | O(VE²) | General |
| Dinic | O(V²E) | General |
| Dinic | O(E√V) | Bipartite matching |
| Push-Relabel | O(V³) or O(V²√E) | Dense graphs |

## 9. Applications

1. **Bipartite matching:** O(E√V) with Dinic
2. **Minimum cut:** Finding bottleneck
3. **Circulation problems:** With demands
4. **Project scheduling:** Resource allocation
5. **Network reliability:** Maximum disjoint paths

## 10. Special Cases

| Graph Type | Complexity |
|------------|------------|
| Unit capacity | O(E√E) |
| Bipartite | O(E√V) |
| Unit network | O(E√V) |
| Planar | O(V log V) with special techniques |

## 11. Common Pitfalls

1. **XOR indexing:** Only works if edges added in pairs
2. **Integer overflow:** Use i64 for large capacities
3. **Forgetting iter reset:** Must reset in each BFS phase
4. **Self-loops:** Handle or disallow

## 12. References

- Dinitz, Y. (1970). "Algorithm for solution of a problem of maximum flow in networks with power estimation"
- Even, S.; Tarjan, R. E. (1975). "Network Flow and Testing Graph Connectivity"
- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 26
