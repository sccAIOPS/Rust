# Prim's Algorithm

## 1. Overview

Prim's algorithm is a greedy algorithm that finds a Minimum Spanning Tree (MST) for a weighted undirected graph. It was developed by Czech mathematician Vojtěch Jarník in 1930 and independently by Robert C. Prim in 1957 and Edsger Dijkstra in 1959.

The algorithm grows the MST one edge at a time, always adding the cheapest edge that connects the tree to a new vertex.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Minimum Spanning Tree Problem:**

Given a connected, undirected graph $G = (V, E)$ with edge weights $w: E \rightarrow \mathbb{R}$, find a spanning tree $T \subseteq E$ that minimizes:
$$\sum_{e \in T} w(e)$$

**Spanning Tree Properties:**
- Connects all vertices (spanning)
- Contains exactly $|V| - 1$ edges
- Contains no cycles (tree)

**Input:**
- Connected undirected graph $G = (V, E)$
- Edge weight function $w$

**Output:**
- MST edges with total minimum weight

### 2.2 Mathematical Model

**Cut Property:**
For any cut $(S, V \setminus S)$ of the graph, the minimum weight edge crossing the cut is in some MST.

**Prim's Invariant:**
At each step, the algorithm maintains a tree $T$ that is a subtree of some MST.

**Greedy Choice:**
Always select the minimum weight edge connecting $T$ to a vertex not in $T$.

### 2.3 Correctness Proof

**Theorem:** Prim's algorithm produces a minimum spanning tree.

**Proof (by Cut Property):**
1. Let $T_k$ be the tree after $k$ iterations
2. Let $e = (u, v)$ be the edge added in iteration $k+1$
3. Consider the cut $(T_k, V \setminus T_k)$
4. Edge $e$ is the minimum weight edge crossing this cut
5. By the Cut Property, $e$ belongs to some MST
6. Since $T_k$ is a subtree of an MST and $e$ is in an MST, $T_{k+1}$ is also a subtree of an MST
7. By induction, the final tree is an MST

## 3. Algorithm Description

### 3.1 Intuition

Imagine building a road network to connect cities:
1. Start from any city
2. Always build the cheapest road that connects a new city to your network
3. Repeat until all cities are connected

### 3.2 Pseudocode

```
PRIM(G, w, r):
    // Initialize
    for each vertex v in V:
        key[v] ← ∞
        π[v] ← NIL
        in_mst[v] ← false
    key[r] ← 0
    
    Q ← priority queue of all vertices by key
    
    while Q is not empty:
        u ← EXTRACT-MIN(Q)
        in_mst[u] ← true
        
        for each neighbor v of u:
            if not in_mst[v] and w(u,v) < key[v]:
                π[v] ← u
                key[v] ← w(u,v)
                DECREASE-KEY(Q, v, key[v])
    
    return {(v, π[v]) : v ∈ V and π[v] ≠ NIL}
```

### 3.3 Step-by-Step Example

Consider this weighted graph:
```
        6
    a-------b
    |\     /|
   7| \2  / |5
    |  \ /  |
    c---d---e
      5   3
```

Prim's algorithm starting from 'a':

| Step | MST Vertices | Next Edge | Weight | Total |
|------|--------------|-----------|--------|-------|
| 0 | {a} | - | - | 0 |
| 1 | {a, d} | (a, d) | 2 | 2 |
| 2 | {a, d, e} | (d, e) | 3 | 5 |
| 3 | {a, d, e, b} | (e, b) | 5 | 10 |
| 4 | {a, d, e, b, c} | (d, c) | 5 | 15 |

**Final MST:**
```
    a
     \
     2\
       \
    c---d---e---b
      5   3   5
```

Total weight: 2 + 3 + 5 + 5 = 15

## 4. Complexity Analysis

### 4.1 Time Complexity

| Implementation | Time |
|----------------|------|
| Adjacency matrix + Array | $O(V^2)$ |
| Binary Heap + Adjacency List | $O(E \log V)$ |
| Fibonacci Heap | $O(E + V \log V)$ |

**Analysis for Binary Heap:**
- Each vertex extracted once: $V \times O(\log V)$
- Each edge may trigger DECREASE-KEY: $E \times O(\log V)$
- Total: $O((V + E) \log V) = O(E \log V)$

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Priority queue | O(V) |
| Key array | O(V) |
| Parent array | O(V) |
| MST edges | O(V) |
| **Total** | O(V + E) |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cmp::Reverse;
use std::collections::{BTreeMap, BinaryHeap};

type Graph<V, E> = BTreeMap<V, BTreeMap<V, E>>;

pub fn prim_with_start<V: Ord + Copy, E: Ord + Add + Copy>(
    graph: &Graph<V, E>,
    start: V,
) -> Graph<V, E> {
    let mut mst: Graph<V, E> = Graph::new();
    let mut prio = BinaryHeap::new();

    mst.insert(start, BTreeMap::new());

    // Add edges from start vertex
    for (v, c) in &graph[&start] {
        prio.push(Reverse((*c, v, start)));
    }

    while let Some(Reverse((dist, t, prev))) = prio.pop() {
        // Skip if vertex already in MST
        if mst.contains_key(t) {
            continue;
        }

        // Add edge to MST
        add_edge(&mut mst, prev, *t, dist);

        // Add new edges to frontier
        for (v, c) in &graph[t] {
            if !mst.contains_key(v) {
                prio.push(Reverse((*c, v, *t)));
            }
        }
    }

    mst
}
```

**Implementation Details:**
- Uses `BinaryHeap` with `Reverse` for min-heap behavior
- Stores `(weight, destination, source)` tuples
- Uses lazy deletion (skip vertices already in MST)
- Returns the MST as a graph structure

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty graph | Returns empty MST |
| Single vertex | Returns graph with one vertex |
| Disconnected graph | Returns MST of component containing start |
| Multiple MSTs | Returns one valid MST |
| Negative weights | Handled correctly |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Network Design**
   - Designing minimum cost computer networks
   - Laying fiber optic cables
   - Electrical grid planning

2. **Clustering**
   - Single-link hierarchical clustering
   - Image segmentation
   - Taxonomy construction

3. **Approximation Algorithms**
   - Traveling Salesman Problem (2-approximation)
   - Steiner Tree Problem

4. **Circuit Design**
   - Printed circuit board routing
   - VLSI design

5. **Transportation**
   - Road network planning
   - Pipeline layout

### 6.2 Related Algorithms

| Algorithm | Comparison |
|-----------|------------|
| Kruskal's | Edge-centric vs vertex-centric |
| Borůvka's | Parallel-friendly MST |
| Reverse-Delete | Starts with all edges, removes |

## 7. Prim's vs Kruskal's

| Aspect | Prim's | Kruskal's |
|--------|--------|-----------|
| Approach | Grow single tree | Merge forest |
| Best for | Dense graphs | Sparse graphs |
| Data Structure | Priority queue | Union-Find |
| Starting point | Requires start vertex | No start needed |
| Time (binary heap) | O(E log V) | O(E log E) |
| Edge sorting | Not needed | Required |
| Connected graph | Required | Not required |

**Choose Prim's when:**
- Graph is dense
- Starting point is natural
- Memory for edge list is limited

**Choose Kruskal's when:**
- Graph is sparse
- Edges are already sorted
- Multiple MSTs for components needed

## 8. Performance Optimizations

### 8.1 Implementation Choices

1. **Lazy vs Eager Deletion**
   - Lazy: Skip vertices already in MST (simpler)
   - Eager: Use indexed priority queue with DECREASE-KEY

2. **Dense Graphs**
   - Use adjacency matrix + array for O(V²)
   - Simpler than heap for dense graphs

3. **Parallel Implementation**
   - Use Borůvka's algorithm phases
   - Process vertex components in parallel

### 8.2 Common Pitfalls

| Issue | Impact | Solution |
|-------|--------|----------|
| Not handling disconnected graphs | Incomplete MST | Check if all vertices reached |
| Integer overflow | Wrong MST | Use appropriate weight type |
| Duplicate edges in heap | Wasted memory | Use indexed heap |
| Directed graphs | Wrong semantics | Ensure undirected edges |

## 9. Correctness Verification

### 9.1 MST Properties to Check

1. **Connectivity:** All vertices reachable
2. **Acyclicity:** No cycles exist
3. **Edge count:** Exactly $|V| - 1$ edges
4. **Optimality:** No cheaper alternative

### 9.2 Test Cases

```rust
#[test]
fn test_prim_basic() {
    let mut graph = BTreeMap::new();
    add_edge(&mut graph, 'a', 'b', 6);
    add_edge(&mut graph, 'a', 'c', 7);
    add_edge(&mut graph, 'a', 'd', 2);
    add_edge(&mut graph, 'b', 'd', 5);
    add_edge(&mut graph, 'b', 'e', 5);
    add_edge(&mut graph, 'c', 'd', 5);
    add_edge(&mut graph, 'd', 'e', 3);

    let mst = prim(&graph);
    let total_weight: i32 = mst.values()
        .flat_map(|edges| edges.values())
        .sum::<i32>() / 2;  // Each edge counted twice
    
    assert_eq!(total_weight, 15);  // 2 + 3 + 5 + 5
}
```

## 10. References

- Jarník, V. (1930). "O jistém problému minimálním"
- Prim, R. C. (1957). "Shortest connection networks and some generalizations"
- Dijkstra, E. W. (1959). "A note on two problems in connexion with graphs"
- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS), Chapter 23
