# Kruskal's Minimum Spanning Tree Algorithm

## 1. Overview

Kruskal's algorithm is a greedy algorithm that finds a Minimum Spanning Tree (MST) for a connected weighted undirected graph. It was developed by Joseph Kruskal in 1956. The algorithm builds the MST by repeatedly adding the smallest edge that doesn't create a cycle.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a connected, undirected graph $G = (V, E)$ with edge weights $w: E \rightarrow \mathbb{R}$, find a spanning tree $T \subseteq E$ minimizing $\sum_{e \in T} w(e)$.

### 2.2 Key Properties

**Cut Property:** For any cut of the graph, the minimum weight crossing edge belongs to some MST.

**Cycle Property:** For any cycle, the maximum weight edge does not belong to any MST.

## 3. Algorithm Description

### 3.1 Pseudocode

```
KRUSKAL(G, w):
    A ← ∅  // MST edges
    
    // Sort edges by weight
    sorted_edges ← SORT(E by w)
    
    // Initialize disjoint sets
    for each vertex v in V:
        MAKE-SET(v)
    
    for each edge (u, v) in sorted_edges (ascending):
        if FIND-SET(u) ≠ FIND-SET(v):
            A ← A ∪ {(u, v)}
            UNION(u, v)
            
            if |A| = |V| - 1:
                break
    
    return A
```

### 3.2 Step-by-Step Example

```
        7
    a-------b
    |\     /|
   5| \8  / |9
    |  \ /  |
    c---d---e
      6   4
```

Sorted edges: (d,e)=4, (a,c)=5, (c,d)=6, (a,b)=7, (a,d)=8, (b,e)=9

| Step | Edge | Weight | Action | MST Edges |
|------|------|--------|--------|-----------|
| 1 | (d,e) | 4 | Add (different sets) | {(d,e)} |
| 2 | (a,c) | 5 | Add | {(d,e), (a,c)} |
| 3 | (c,d) | 6 | Add | {(d,e), (a,c), (c,d)} |
| 4 | (a,b) | 7 | Add | {(d,e), (a,c), (c,d), (a,b)} |
| 5 | (a,d) | 8 | Skip (same set) | - |
| 6 | (b,e) | 9 | Skip (same set) | - |

**Total Weight:** 4 + 5 + 6 + 7 = 22

## 4. Complexity Analysis

### Time Complexity
- Sorting edges: $O(E \log E)$
- Union-Find operations: $O(E \cdot \alpha(V))$ where $\alpha$ is inverse Ackermann
- **Total:** $O(E \log E) = O(E \log V)$

### Space Complexity
- Edge list: $O(E)$
- Union-Find: $O(V)$
- **Total:** $O(V + E)$

## 5. Implementation

```rust
use crate::graph::DisjointSetUnion;

pub struct Edge {
    source: usize,
    destination: usize,
    cost: usize,
}

pub fn kruskal(mut edges: Vec<Edge>, num_vertices: usize) -> Option<(usize, Vec<Edge>)> {
    let mut dsu = DisjointSetUnion::new(num_vertices);
    let mut mst_cost: usize = 0;
    let mut mst_edges: Vec<Edge> = Vec::with_capacity(num_vertices - 1);

    // Sort edges by cost
    edges.sort_unstable_by_key(|edge| edge.cost);

    for edge in edges {
        if mst_edges.len() == num_vertices - 1 {
            break;
        }

        // If vertices are in different components, add edge
        if dsu.merge(edge.source, edge.destination) != usize::MAX {
            mst_cost += edge.cost;
            mst_edges.push(edge);
        }
    }

    // Check if MST is complete
    (mst_edges.len() == num_vertices - 1).then_some((mst_cost, mst_edges))
}
```

## 6. Applications

1. **Network Design:** Minimum cost networks
2. **Clustering:** Creating k clusters by removing k-1 largest edges
3. **Image Segmentation:** Graph-based segmentation
4. **Approximation Algorithms:** TSP 2-approximation

## 7. Kruskal's vs Prim's

| Aspect | Kruskal's | Prim's |
|--------|-----------|--------|
| Approach | Edge-centric | Vertex-centric |
| Best for | Sparse graphs | Dense graphs |
| Data Structure | Union-Find | Priority Queue |
| Edge sorting | Required | Not needed |

## 8. References

- Kruskal, J. B. (1956). "On the shortest spanning subtree of a graph"
- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 23
