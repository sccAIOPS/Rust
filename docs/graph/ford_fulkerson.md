# Ford-Fulkerson Maximum Flow Algorithm

## 1. Overview

The Ford-Fulkerson algorithm computes the maximum flow in a flow network. It repeatedly finds augmenting paths from source to sink and pushes flow along them until no more augmenting paths exist. The method was published by L. R. Ford Jr. and D. R. Fulkerson in 1956.

## 2. Mathematical Foundation

### 2.1 Flow Network

A **flow network** $G = (V, E)$ is a directed graph where each edge $(u, v)$ has:
- **Capacity** $c(u, v) \geq 0$
- A designated **source** $s$ and **sink** $t$

### 2.2 Flow Properties

A valid flow $f$ satisfies:
1. **Capacity constraint:** $0 \leq f(u, v) \leq c(u, v)$
2. **Conservation:** For $v \neq s, t$: $\sum_u f(u, v) = \sum_w f(v, w)$

### 2.3 Residual Graph

For flow $f$, the **residual capacity** is:
$$c_f(u, v) = c(u, v) - f(u, v) + f(v, u)$$

The residual graph contains edges with positive residual capacity.

### 2.4 Max-Flow Min-Cut Theorem

$$\text{max flow} = \text{min cut capacity}$$

## 3. Algorithm

### 3.1 Pseudocode

```
FORD-FULKERSON(G, s, t):
    Initialize flow f = 0 on all edges
    
    while exists augmenting path p from s to t in residual graph:
        cf ← minimum residual capacity along p
        for each edge (u, v) in p:
            if (u, v) is forward edge:
                f(u, v) += cf
            else:
                f(v, u) -= cf
    
    return total flow into t
```

### 3.2 DFS-based Path Finding

```rust
fn dfs(
    graph: &[Vec<Edge>],
    source: usize,
    sink: usize,
    visited: &mut [bool],
    flow: i64,
) -> i64 {
    if source == sink {
        return flow;
    }
    visited[source] = true;
    
    for edge in &graph[source] {
        if !visited[edge.to] && edge.capacity > edge.flow {
            let residual = edge.capacity - edge.flow;
            let pushed = dfs(graph, edge.to, sink, visited, min(flow, residual));
            
            if pushed > 0 {
                edge.flow += pushed;
                reverse_edge.flow -= pushed;
                return pushed;
            }
        }
    }
    0
}
```

## 4. Example

```
Graph:        Capacities:
s → a → t     s→a: 10, a→t: 5
↓   ↓         s→b: 5, b→t: 10
b → ↗         a→b: 8
```

**Iteration 1:** Path s→a→t, flow = 5
**Iteration 2:** Path s→a→b→t, flow = 5
**Iteration 3:** Path s→b→t, flow = 5

**Maximum Flow = 15**

## 5. Complexity

| Variant | Path Finding | Time Complexity |
|---------|--------------|-----------------|
| DFS | Arbitrary | O(E × f*) where f* is max flow |
| BFS (Edmonds-Karp) | Shortest path | O(V × E²) |

**Warning:** With DFS, complexity depends on max flow value (can be infinite for irrational capacities).

## 6. Implementation

```rust
#[derive(Clone)]
pub struct Edge {
    pub to: usize,
    pub capacity: i64,
    pub flow: i64,
    pub reverse_index: usize,
}

pub struct FlowNetwork {
    pub adj: Vec<Vec<Edge>>,
}

impl FlowNetwork {
    pub fn add_edge(&mut self, from: usize, to: usize, capacity: i64) {
        let from_idx = self.adj[to].len();
        let to_idx = self.adj[from].len();
        
        self.adj[from].push(Edge { to, capacity, flow: 0, reverse_index: from_idx });
        self.adj[to].push(Edge { to: from, capacity: 0, flow: 0, reverse_index: to_idx });
    }

    pub fn max_flow(&mut self, source: usize, sink: usize) -> i64 {
        let mut total_flow = 0;
        
        loop {
            let mut visited = vec![false; self.adj.len()];
            let flow = self.dfs(source, sink, &mut visited, i64::MAX);
            
            if flow == 0 { break; }
            total_flow += flow;
        }
        
        total_flow
    }

    fn dfs(&mut self, u: usize, sink: usize, visited: &mut [bool], flow: i64) -> i64 {
        if u == sink { return flow; }
        visited[u] = true;
        
        for i in 0..self.adj[u].len() {
            let edge = &self.adj[u][i];
            if !visited[edge.to] && edge.capacity > edge.flow {
                let to = edge.to;
                let rev_idx = edge.reverse_index;
                let residual = edge.capacity - edge.flow;
                let pushed = self.dfs(to, sink, visited, flow.min(residual));
                
                if pushed > 0 {
                    self.adj[u][i].flow += pushed;
                    self.adj[to][rev_idx].flow -= pushed;
                    return pushed;
                }
            }
        }
        0
    }
}
```

## 7. Key Implementation Details

### 7.1 Reverse Edges

Every edge needs a reverse edge for flow cancellation:
- Forward edge: capacity = actual capacity, initial flow = 0
- Reverse edge: capacity = 0, allows flow cancellation

### 7.2 Residual Capacity

```rust
residual_capacity = capacity - flow  // For forward edge
residual_capacity = -flow            // For reverse edge (flow can be negative)
```

## 8. Applications

1. **Network routing:** Maximum data throughput
2. **Bipartite matching:** Convert to max flow problem
3. **Minimum cut:** Find bottleneck in network
4. **Image segmentation:** Graph cuts
5. **Project selection:** Maximum profit subset
6. **Circulation problems:** Feasibility checking

## 9. Variants

| Algorithm | Time | Features |
|-----------|------|----------|
| Ford-Fulkerson (DFS) | O(Ef) | Simple but slow |
| Edmonds-Karp (BFS) | O(VE²) | Polynomial time |
| Dinic's | O(V²E) | Blocking flow |
| Push-Relabel | O(V²E) or O(V³) | No augmenting paths |

## 10. Common Pitfalls

1. **Forgetting reverse edges:** Required for correctness
2. **Integer overflow:** Use i64 for large capacities
3. **Incorrect termination:** Stop when no augmenting path exists
4. **DFS performance:** Can be exponential with bad path choices

## 11. References

- Ford, L. R.; Fulkerson, D. R. (1956). "Maximal flow through a network"
- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 26
