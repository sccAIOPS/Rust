# Dijkstra's Algorithm

## 1. Overview

Dijkstra's algorithm is a famous algorithm for finding the shortest paths between nodes in a weighted graph with non-negative edge weights. It was conceived by computer scientist Edsger W. Dijkstra in 1956 and published three years later.

The algorithm is widely used in network routing protocols (OSPF, IS-IS) and as a subroutine in other algorithms.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Single-Source Shortest Path (SSSP) Problem:**

Given a weighted directed graph $G = (V, E, w)$ where:
- $V$ is the set of vertices
- $E$ is the set of edges
- $w: E \rightarrow \mathbb{R}^+$ assigns non-negative weights to edges

Find the shortest path from a source vertex $s$ to all other vertices.

**Input:**
- A graph $G$ with non-negative edge weights
- A source vertex $s \in V$

**Output:**
- For each vertex $v$: the shortest distance $\delta(s, v)$ and predecessor $\pi(v)$

### 2.2 Mathematical Model

**Shortest Path Substructure:**
If $p = \langle v_0, v_1, ..., v_k \rangle$ is a shortest path from $v_0$ to $v_k$, then for any $0 \leq i \leq j \leq k$, the subpath $\langle v_i, ..., v_j \rangle$ is a shortest path from $v_i$ to $v_j$.

**Triangle Inequality:**
For all edges $(u, v) \in E$:
$$\delta(s, v) \leq \delta(s, u) + w(u, v)$$

**Relaxation Operation:**
```
RELAX(u, v, w):
    if d[v] > d[u] + w(u, v):
        d[v] = d[u] + w(u, v)
        π[v] = u
```

### 2.3 Correctness Proof

**Theorem:** Dijkstra's algorithm correctly computes shortest paths when all edge weights are non-negative.

**Proof Sketch (by induction):**
- Let $S$ be the set of vertices whose shortest distances are known
- Initially $S = \{s\}$ with $d[s] = 0$
- At each step, we extract the vertex $u$ with minimum $d[u]$ from the frontier
- Since all edge weights are non-negative, no path through vertices not in $S$ can be shorter
- Therefore, $d[u] = \delta(s, u)$, and we can add $u$ to $S$

**Why Non-Negative Weights Required:**
With negative edges, a later discovery might find a shorter path, invalidating the greedy choice.

## 3. Algorithm Description

### 3.1 Intuition

Imagine water spreading from a source:
1. Water starts at the source with distance 0
2. Water flows along edges, taking time proportional to edge weight
3. When water reaches a vertex, that's the shortest time to reach it
4. From each flooded vertex, water continues to spread

The key insight: once a vertex is "flooded" (extracted from the priority queue), its shortest distance is final.

### 3.2 Pseudocode

```
DIJKSTRA(G, w, s):
    // Initialize
    for each vertex v in V:
        d[v] ← ∞
        π[v] ← NIL
    d[s] ← 0
    
    // Priority queue ordered by d values
    Q ← V  // All vertices with their d values
    
    while Q is not empty:
        u ← EXTRACT-MIN(Q)  // Vertex with smallest d value
        
        for each neighbor v of u:
            // Relaxation
            if d[v] > d[u] + w(u, v):
                d[v] ← d[u] + w(u, v)
                π[v] ← u
                DECREASE-KEY(Q, v, d[v])
    
    return d, π
```

### 3.3 Step-by-Step Example

Consider this weighted graph:
```
        12
    a ----→ c
    |       |
  10|       |32
    ↓   20  ↓
    b ----→ d
        ↖   |
      4  \  |60
          \ ↓
            e
```

Dijkstra from vertex 'a':

| Step | u | d[a] | d[b] | d[c] | d[d] | d[e] | Queue |
|------|---|------|------|------|------|------|-------|
| Init | - | 0 | ∞ | ∞ | ∞ | ∞ | {a,b,c,d,e} |
| 1 | a | **0** | 10 | 12 | ∞ | ∞ | {b,c,d,e} |
| 2 | b | 0 | **10** | 12 | 30 | ∞ | {c,d,e} |
| 3 | c | 0 | 10 | **12** | 30 | ∞ | {d,e} |
| 4 | d | 0 | 10 | 12 | **30** | 90 | {e} |
| 5 | e | 0 | 10 | 12 | 30 | **90** | {} |

**Shortest Paths:**
- a → a: 0
- a → b: 10 (via a→b)
- a → c: 12 (via a→c)
- a → d: 30 (via a→b→d)
- a → e: 90 (via a→b→d→e)

## 4. Complexity Analysis

### 4.1 Time Complexity

The complexity depends on the priority queue implementation:

| Implementation | INSERT | EXTRACT-MIN | DECREASE-KEY | Total |
|----------------|--------|-------------|--------------|-------|
| Array | O(1) | O(V) | O(1) | O(V²) |
| Binary Heap | O(log V) | O(log V) | O(log V) | O((V+E) log V) |
| Fibonacci Heap | O(1)* | O(log V)* | O(1)* | O(V log V + E) |

*Amortized

**For sparse graphs (E ≈ V):** Binary heap gives O(V log V)
**For dense graphs (E ≈ V²):** Array gives O(V²), which is optimal

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Distance array | O(V) |
| Predecessor array | O(V) |
| Priority queue | O(V) |
| Graph | O(V + E) |
| **Total** | O(V + E) |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::{BTreeMap, BTreeSet};

type Graph<V, E> = BTreeMap<V, BTreeMap<V, E>>;

pub fn dijkstra<V: Ord + Copy, E: Ord + Copy + Add<Output = E>>(
    graph: &Graph<V, E>,
    start: V,
) -> BTreeMap<V, Option<(V, E)>> {
    let mut ans = BTreeMap::new();
    let mut prio = BTreeSet::new();

    ans.insert(start, None);

    for (new, weight) in &graph[&start] {
        ans.insert(*new, Some((start, *weight)));
        prio.insert((*weight, *new));
    }

    while let Some((path_weight, vertex)) = prio.pop_first() {
        for (next, weight) in &graph[&vertex] {
            let new_weight = path_weight + *weight;
            match ans.get(next) {
                Some(Some((_, dist_next))) if new_weight >= *dist_next => {}
                Some(None) => {}
                _ => {
                    if let Some(Some((_, prev_weight))) =
                        ans.insert(*next, Some((vertex, new_weight)))
                    {
                        prio.remove(&(prev_weight, *next));
                    }
                    prio.insert((new_weight, *next));
                }
            }
        }
    }
    ans
}
```

**Key Implementation Details:**
- Uses `BTreeSet` as priority queue (ordered set)
- `pop_first()` extracts minimum element
- Generic over vertex type `V` and edge weight type `E`
- Returns `Option<(predecessor, distance)>` for each vertex
- Source vertex mapped to `None` (no predecessor)

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Source not in graph | Panics (could return empty map) |
| Disconnected vertices | Not included in result |
| Self-loops | Correctly ignored (distance 0 < any weight) |
| Negative edges | **Invalid input** - undefined behavior |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Network Routing**
   - OSPF (Open Shortest Path First) protocol
   - IS-IS (Intermediate System to Intermediate System)
   - Traffic engineering in data centers

2. **GPS Navigation**
   - Finding shortest/fastest routes
   - Real-time traffic rerouting

3. **Social Networks**
   - Finding shortest "relationship path"
   - Influence propagation modeling

4. **Games**
   - Pathfinding for NPCs
   - Movement cost calculation

5. **Logistics**
   - Delivery route optimization
   - Supply chain planning

6. **Network Analysis**
   - Centrality measures (closeness centrality)
   - Network diameter calculation

### 6.2 Related Algorithms

| Algorithm | When to Use |
|-----------|-------------|
| BFS | Unweighted graphs |
| Bellman-Ford | Graphs with negative edges |
| A* | Single target with good heuristic |
| Floyd-Warshall | All-pairs shortest paths |
| Johnson's | All-pairs with sparse negative edges |

## 7. Performance Optimizations

### 7.1 Potential Improvements

1. **Bidirectional Dijkstra**
   - Search from both source and target
   - Meet in the middle
   - Reduces search space significantly

2. **A* Algorithm**
   - Use heuristic to guide search
   - Optimal with admissible heuristic

3. **Goal-Directed Search**
   - ALT (A*, Landmarks, Triangle inequality)
   - Contraction Hierarchies

4. **Parallel Implementation**
   - Process multiple vertices simultaneously
   - Δ-stepping algorithm

### 7.2 Common Pitfalls

| Issue | Impact | Solution |
|-------|--------|----------|
| Negative edge weights | Incorrect results | Use Bellman-Ford |
| Not using priority queue | O(V²) always | Use binary heap |
| Revisiting vertices | Wasted computation | Mark as finalized |
| Integer overflow | Wrong distances | Use saturating arithmetic |

## 8. Comparison with Similar Algorithms

| Feature | Dijkstra | Bellman-Ford | Floyd-Warshall |
|---------|----------|--------------|----------------|
| Negative edges | ✗ | ✓ | ✓ |
| Negative cycles | ✗ | Detects | Detects |
| Time | O(E log V) | O(VE) | O(V³) |
| Space | O(V) | O(V) | O(V²) |
| Problem | SSSP | SSSP | APSP |

## 9. References

- Dijkstra, E. W. (1959). "A note on two problems in connexion with graphs"
- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS), Chapter 24
- Fredman, M. L., & Tarjan, R. E. (1987). "Fibonacci heaps and their uses in improved network optimization algorithms"
