# Floyd-Warshall Algorithm

## 1. Overview

The Floyd-Warshall algorithm is a classic dynamic programming algorithm for finding shortest paths between all pairs of vertices in a weighted graph. It was published by Robert Floyd in 1962, based on a 1959 paper by Bernard Roy and Stephen Warshall's work on transitive closure.

The algorithm handles both positive and negative edge weights (but not negative cycles) and can detect negative cycles.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**All-Pairs Shortest Path (APSP) Problem:**

Given a weighted graph $G = (V, E, w)$, find the shortest path distance $\delta(i, j)$ for all pairs of vertices $i, j \in V$.

**Input:**
- A graph $G$ with $n$ vertices
- Edge weights $w: E \rightarrow \mathbb{R}$ (may be negative)
- No negative cycles

**Output:**
- A matrix $D$ where $D[i][j] = \delta(i, j)$
- Optionally, a predecessor matrix for path reconstruction

### 2.2 Mathematical Model

**Recurrence Relation:**

Let $d^{(k)}_{ij}$ be the shortest path from $i$ to $j$ using only vertices $\{1, 2, ..., k\}$ as intermediate vertices.

$$d^{(k)}_{ij} = \min(d^{(k-1)}_{ij}, d^{(k-1)}_{ik} + d^{(k-1)}_{kj})$$

**Base Case:**
$$d^{(0)}_{ij} = \begin{cases} 0 & \text{if } i = j \\ w(i,j) & \text{if } (i,j) \in E \\ \infty & \text{otherwise} \end{cases}$$

**Final Result:**
$$\delta(i, j) = d^{(n)}_{ij}$$

### 2.3 Correctness Proof

**Proof by Induction on $k$:**

- **Base ($k = 0$):** $d^{(0)}_{ij}$ is correct since paths with no intermediate vertices are direct edges.

- **Inductive Step:** Assume $d^{(k-1)}$ is correct. For $d^{(k)}_{ij}$:
  - Either the shortest path doesn't use vertex $k$: $d^{(k)}_{ij} = d^{(k-1)}_{ij}$
  - Or it uses $k$: $d^{(k)}_{ij} = d^{(k-1)}_{ik} + d^{(k-1)}_{kj}$
  - Taking the minimum gives the correct answer.

**Negative Cycle Detection:**
If $d^{(n)}_{ii} < 0$ for any vertex $i$, a negative cycle exists.

## 3. Algorithm Description

### 3.1 Intuition

Consider a construction company deciding how to connect cities:
1. Initially, only direct routes are known
2. Gradually consider each city as a "hub"
3. For each hub, check if going through it provides a shortcut
4. After considering all hubs, all shortest paths are known

### 3.2 Pseudocode

```
FLOYD-WARSHALL(W):
    n ← W.rows
    D ← W  // Initialize with direct edge weights
    
    // Set diagonal to 0
    for i ← 1 to n:
        D[i][i] ← 0
    
    // Consider each vertex as intermediate
    for k ← 1 to n:
        for i ← 1 to n:
            for j ← 1 to n:
                if D[i][k] + D[k][j] < D[i][j]:
                    D[i][j] ← D[i][k] + D[k][j]
    
    return D
```

### 3.3 Step-by-Step Example

Consider this graph:
```
    1 ----3---→ 2
    |           ↑
    6           2
    |           |
    ↓           |
    3 ----1---→ 4
```

**Initial Distance Matrix ($k = 0$):**
```
      1    2    3    4
  1 [ 0    3    6    ∞ ]
  2 [ ∞    0    ∞    ∞ ]
  3 [ ∞    ∞    0    1 ]
  4 [ ∞    2    ∞    0 ]
```

**After $k = 1$ (considering vertex 1 as intermediate):**
```
      1    2    3    4
  1 [ 0    3    6    ∞ ]
  2 [ ∞    0    ∞    ∞ ]
  3 [ ∞    ∞    0    1 ]
  4 [ ∞    2    ∞    0 ]
```
(No changes - no paths improved through 1)

**After $k = 3$ (considering vertex 3):**
```
      1    2    3    4
  1 [ 0    3    6    7 ]  ← 1→3→4 = 6+1 = 7
  2 [ ∞    0    ∞    ∞ ]
  3 [ ∞    ∞    0    1 ]
  4 [ ∞    2    ∞    0 ]
```

**After $k = 4$ (considering vertex 4):**
```
      1    2    3    4
  1 [ 0    3    6    7 ]
  2 [ ∞    0    ∞    ∞ ]
  3 [ ∞    3    0    1 ]  ← 3→4→2 = 1+2 = 3
  4 [ ∞    2    ∞    0 ]
```

**Final Result:** All shortest paths computed.

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| All Cases | $O(V^3)$ | Three nested loops, each O(V) |

This is optimal for dense graphs where $E = O(V^2)$.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Distance matrix | $O(V^2)$ |
| Predecessor matrix (optional) | $O(V^2)$ |
| **Total** | $O(V^2)$ |

**Space Optimization:** Can update matrix in-place since $d^{(k)}_{ik} = d^{(k-1)}_{ik}$ and $d^{(k)}_{kj} = d^{(k-1)}_{kj}$.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use num_traits::Zero;
use std::collections::BTreeMap;
use std::ops::Add;

type Graph<V, E> = BTreeMap<V, BTreeMap<V, E>>;

pub fn floyd_warshall<V: Ord + Copy, E: Ord + Copy + Add<Output = E> + Zero>(
    graph: &Graph<V, E>,
) -> BTreeMap<V, BTreeMap<V, E>> {
    let mut map: BTreeMap<V, BTreeMap<V, E>> = BTreeMap::new();
    
    // Initialize with direct edges and zeros on diagonal
    for (u, edges) in graph.iter() {
        map.entry(*u).or_default().insert(*u, Zero::zero());
        for (v, weight) in edges.iter() {
            map.entry(*v).or_default().insert(*v, Zero::zero());
            map.entry(*u).or_default().insert(*v, *weight);
        }
    }
    
    let keys = map.keys().copied().collect::<Vec<_>>();
    
    // Main Floyd-Warshall loop
    for &k in &keys {
        for &i in &keys {
            if !map[&i].contains_key(&k) { continue; }
            for &j in &keys {
                if i == j { continue; }
                if !map[&k].contains_key(&j) { continue; }
                
                let through_k = map[&i][&k] + map[&k][&j];
                match map[&i].get(&j) {
                    Some(&direct) if direct <= through_k => {}
                    _ => { map.entry(i).or_default().insert(j, through_k); }
                };
            }
        }
    }
    map
}
```

**Implementation Notes:**
- Uses `BTreeMap` for sparse representation
- `num_traits::Zero` provides generic zero value
- Handles missing edges by skipping (implicit infinity)
- Generic over vertex and edge types

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty graph | Returns empty map |
| Single vertex | Returns `{v: {v: 0}}` |
| Disconnected components | Missing pairs not in result |
| Negative edges | Correctly handled |
| Negative cycles | Diagonal becomes negative (detection) |
| Self-loops | Included if in input |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Network Analysis**
   - Finding network diameter
   - Computing betweenness centrality
   - Detecting network partitions

2. **Transitive Closure**
   - Reachability analysis
   - Dependency resolution
   - Access control (can A reach B?)

3. **Path Problems**
   - Minimum bottleneck path
   - Maximum bandwidth path (with modification)
   - Safest path

4. **Computer Graphics**
   - Scene graph shortest paths
   - Animation blending weights

5. **Bioinformatics**
   - Protein interaction networks
   - Gene regulatory networks

6. **Game Development**
   - Precomputed pathfinding tables
   - Influence mapping

### 6.2 Variants

| Variant | Modification |
|---------|--------------|
| Transitive Closure | Use OR instead of MIN+ADD |
| Maximum Capacity | Use MIN(max operations) |
| Minimum Reliability | Use product instead of sum |
| Path Counting | Add count arrays |

## 7. Path Reconstruction

### 7.1 Predecessor Matrix

```
FLOYD-WARSHALL-PATHS(W):
    // Initialize predecessor matrix
    for i ← 1 to n:
        for j ← 1 to n:
            if i = j or W[i][j] = ∞:
                π[i][j] ← NIL
            else:
                π[i][j] ← i
    
    for k ← 1 to n:
        for i ← 1 to n:
            for j ← 1 to n:
                if D[i][k] + D[k][j] < D[i][j]:
                    D[i][j] ← D[i][k] + D[k][j]
                    π[i][j] ← π[k][j]  // Update predecessor
    
    return D, π
```

### 7.2 Path Extraction

```
GET-PATH(π, i, j):
    if i = j:
        return [i]
    if π[i][j] = NIL:
        return []  // No path
    
    path ← GET-PATH(π, i, π[i][j])
    path.append(j)
    return path
```

## 8. Comparison with Other APSP Algorithms

| Algorithm | Time | Space | Best For |
|-----------|------|-------|----------|
| Floyd-Warshall | O(V³) | O(V²) | Dense graphs |
| Johnson's | O(V² log V + VE) | O(V²) | Sparse graphs |
| V × Dijkstra | O(VE log V) | O(V) | Non-negative, sparse |
| V × Bellman-Ford | O(V²E) | O(V) | Negative edges |

## 9. Common Pitfalls

| Issue | Impact | Solution |
|-------|--------|----------|
| Not initializing diagonal | Wrong self-distances | Set D[i][i] = 0 |
| Wrong loop order | Incorrect results | Must be k → i → j |
| Integer overflow | Wrong distances | Use saturating add |
| Not detecting negative cycles | Silent errors | Check diagonal after |

## 10. References

- Floyd, R. W. (1962). "Algorithm 97: Shortest Path"
- Warshall, S. (1962). "A Theorem on Boolean Matrices"
- Roy, B. (1959). "Transitivité et connexité"
- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS), Chapter 25
