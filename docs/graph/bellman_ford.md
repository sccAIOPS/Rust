# Bellman-Ford Algorithm

## 1. Overview

The Bellman-Ford algorithm computes shortest paths from a single source vertex to all other vertices in a weighted directed graph. Unlike Dijkstra's algorithm, it can handle graphs with negative edge weights and detect negative weight cycles.

The algorithm was developed by Alfonso Shimbel (1955), Richard Bellman (1958), and Lester Ford Jr. (1956).

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Single-Source Shortest Path with Negative Edges:**

Given a weighted directed graph $G = (V, E, w)$ where:
- $V$ is the set of vertices
- $E$ is the set of edges
- $w: E \rightarrow \mathbb{R}$ assigns (possibly negative) weights to edges

Find the shortest path from source $s$ to all vertices, or detect a negative cycle.

**Input:**
- A graph $G$ with arbitrary edge weights
- A source vertex $s \in V$

**Output:**
- For each vertex $v$: shortest distance $\delta(s, v)$ and predecessor $\pi(v)$
- Or indication that a negative cycle exists

### 2.2 Mathematical Model

**Path-Relaxation Property:**
If $p = \langle v_0, v_1, ..., v_k \rangle$ is a shortest path and we relax edges $(v_0, v_1), (v_1, v_2), ..., (v_{k-1}, v_k)$ in order, then $d[v_k] = \delta(s, v_k)$.

**Key Insight:**
- A shortest path contains at most $|V| - 1$ edges
- After $|V| - 1$ relaxation rounds, all shortest paths are found
- If the $(|V|)$-th round still improves distances, a negative cycle exists

**Negative Cycle Detection:**
A negative cycle reachable from $s$ exists iff after $|V| - 1$ iterations, some edge $(u, v)$ still satisfies:
$$d[v] > d[u] + w(u, v)$$

### 2.3 Correctness Proof

**Theorem:** After $i$ iterations, Bellman-Ford correctly computes all shortest paths of at most $i$ edges.

**Proof by Induction:**
- **Base case ($i = 0$):** $d[s] = 0$ is correct.
- **Inductive step:** Assume correctness for paths of ≤ $i$ edges. For any shortest path $p$ to $v$ with $i + 1$ edges, let $u$ be the predecessor of $v$ on $p$. The subpath to $u$ has $i$ edges, so $d[u]$ is correct. During iteration $i + 1$, relaxing $(u, v)$ sets $d[v]$ correctly.

**Corollary:** Since any simple path has at most $|V| - 1$ edges, $|V| - 1$ iterations suffice.

## 3. Algorithm Description

### 3.1 Intuition

Think of it as "message passing" in a network:
1. Source knows its distance is 0
2. Each round, every vertex tells its neighbors: "I can reach the source with distance $d$"
3. Neighbors update if they receive a better offer
4. After $|V| - 1$ rounds, everyone knows their shortest distance
5. If messages still improve after $|V| - 1$ rounds, there's a "free money" cycle

### 3.2 Pseudocode

```
BELLMAN-FORD(G, w, s):
    // Initialize
    for each vertex v in V:
        d[v] ← ∞
        π[v] ← NIL
    d[s] ← 0
    
    // Main loop: relax all edges |V| - 1 times
    for i ← 1 to |V| - 1:
        for each edge (u, v) in E:
            if d[v] > d[u] + w(u, v):
                d[v] ← d[u] + w(u, v)
                π[v] ← u
    
    // Check for negative cycles
    for each edge (u, v) in E:
        if d[v] > d[u] + w(u, v):
            return "Negative cycle detected"
    
    return d, π
```

### 3.3 Step-by-Step Example

Consider this graph (with a negative edge):
```
        5
    a ----→ b
    |       |
   6|       |-2
    ↓       ↓
    c ----→ d
        8
```

Bellman-Ford from 'a':

**Iteration 1:**
| Edge | d[u] + w | d[v] | Update? |
|------|----------|------|---------|
| (a,b) | 0 + 5 = 5 | ∞ | Yes, d[b] = 5 |
| (a,c) | 0 + 6 = 6 | ∞ | Yes, d[c] = 6 |
| (b,d) | 5 + (-2) = 3 | ∞ | Yes, d[d] = 3 |
| (c,d) | 6 + 8 = 14 | 3 | No |

**Iteration 2:**
| Edge | d[u] + w | d[v] | Update? |
|------|----------|------|---------|
| (a,b) | 0 + 5 = 5 | 5 | No |
| (a,c) | 0 + 6 = 6 | 6 | No |
| (b,d) | 5 + (-2) = 3 | 3 | No |
| (c,d) | 6 + 8 = 14 | 3 | No |

**Iteration 3:** No changes

**Final Result:** d[a]=0, d[b]=5, d[c]=6, d[d]=3

**Negative Cycle Check:** No edge can be relaxed ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| All Cases | $O(V \cdot E)$ | $|V|-1$ iterations, each examining $|E|$ edges |

**Breakdown:**
- Initialization: $O(V)$
- Main loop: $(|V| - 1) \times |E| = O(VE)$
- Negative cycle check: $O(E)$
- Total: $O(VE)$

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Distance array | O(V) |
| Predecessor array | O(V) |
| **Total** | O(V) |

Note: The graph representation is typically O(V + E) additional.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::BTreeMap;
use std::ops::{Add, Neg, Sub};

type Graph<V, E> = BTreeMap<V, BTreeMap<V, E>>;

pub fn bellman_ford<
    V: Ord + Copy,
    E: Ord + Copy + Add<Output = E> + Neg<Output = E> + Sub<Output = E>,
>(
    graph: &Graph<V, E>,
    start: &V,
) -> Option<BTreeMap<V, Option<(V, E)>>> {
    let mut ans: BTreeMap<V, Option<(V, E)>> = BTreeMap::new();
    ans.insert(*start, None);

    // Main loop: |V| - 1 iterations
    for _ in 1..(graph.len()) {
        for (u, edges) in graph {
            let dist_u = match ans.get(u) {
                Some(Some((_, d))) => Some(*d),
                Some(None) => None,
                None => continue,
            };

            for (v, d) in edges {
                // Relaxation logic with careful handling of source
                match ans.get(v) {
                    Some(Some((_, dist))) if /* new path is longer */ => {}
                    Some(None) => { /* check for negative loop through source */ }
                    _ => {
                        // Update with shorter path
                        ans.insert(*v, Some((*u, /* new distance */)));
                    }
                }
            }
        }
    }

    // Negative cycle detection
    for (u, edges) in graph {
        for (v, d) in edges {
            // If still can relax, negative cycle exists
            if /* can_relax */ {
                return None;
            }
        }
    }

    Some(ans)
}
```

**Implementation Notes:**
- Returns `Option` to indicate negative cycle (returns `None`)
- Handles the source vertex specially (has `None` as predecessor)
- Uses generic types with trait bounds for flexible edge weights
- Requires `Neg` trait for negative cycle detection logic

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Single vertex | Returns `{start: None}` |
| Disconnected vertices | Not included in result |
| Zero-weight edges | Handled correctly |
| Negative edges | Correctly handled |
| Negative cycles | Returns `None` |
| Self-loops with negative weight | Detected as negative cycle |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Currency Arbitrage Detection**
   - Exchange rates as edge weights
   - Log transform: $w = -\log(\text{rate})$
   - Negative cycle = arbitrage opportunity

2. **Network Routing (BGP)**
   - Border Gateway Protocol uses path-vector routing
   - Can handle routing policies as negative weights

3. **Constraint Solving**
   - Difference constraints: $x_j - x_i \leq b_{ij}$
   - Model as edges with weight $b_{ij}$
   - Shortest path gives satisfying assignment

4. **Timing Analysis**
   - Circuit delay computation
   - Setup/hold time verification
   - Negative weights for inverse delays

5. **Project Scheduling**
   - PERT/CPM with negative slack
   - Critical path finding

### 6.2 Related Algorithms

| Algorithm | Comparison |
|-----------|------------|
| Dijkstra | Faster but can't handle negative edges |
| Floyd-Warshall | All-pairs, same negative edge capability |
| SPFA | Optimized Bellman-Ford (average case faster) |
| Johnson's | All-pairs using Bellman-Ford + Dijkstra |

## 7. Optimizations

### 7.1 SPFA (Shortest Path Faster Algorithm)

An optimization that only relaxes edges from recently updated vertices:

```
SPFA(G, w, s):
    Initialize d[], π[] as in Bellman-Ford
    queue ← {s}
    in_queue[s] ← true
    
    while queue not empty:
        u ← queue.dequeue()
        in_queue[u] ← false
        
        for each edge (u, v):
            if d[v] > d[u] + w(u, v):
                d[v] ← d[u] + w(u, v)
                π[v] ← u
                if not in_queue[v]:
                    queue.enqueue(v)
                    in_queue[v] ← true
```

**Complexity:** O(VE) worst case, but often O(E) in practice.

### 7.2 Common Pitfalls

| Issue | Impact | Solution |
|-------|--------|----------|
| Not detecting all negative cycles | Silent incorrect results | Check all edges, not just reachable |
| Integer overflow | Wrong distances | Use saturating arithmetic |
| Early termination | Missing optimizations | Check if no updates occurred |
| Infinite loops with neg cycles | Never terminates | Limit iterations to V-1 |

## 8. Bellman-Ford vs Dijkstra

| Feature | Bellman-Ford | Dijkstra |
|---------|--------------|----------|
| Time Complexity | O(VE) | O(E log V) |
| Negative Edges | ✓ | ✗ |
| Negative Cycles | Detects | Undefined behavior |
| Implementation | Simpler | More complex (heap) |
| Parallelizable | Yes (edges) | Limited |

**When to Use Bellman-Ford:**
- Negative edge weights present
- Need to detect negative cycles
- Simpler implementation preferred
- Graph is sparse (E ≈ V)

## 9. References

- Bellman, R. (1958). "On a routing problem"
- Ford, L. R. Jr. (1956). "Network Flow Theory"
- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS), Chapter 24
- Shimbel, A. (1955). "Structure in communication nets"
