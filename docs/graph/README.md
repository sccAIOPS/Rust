# Graph Algorithms

This directory contains comprehensive documentation for all graph algorithms implemented in TheAlgorithms/Rust repository.

## Overview

Graph algorithms are fundamental to computer science and are used to solve problems involving networks, relationships, and connectivity. This collection includes traversal algorithms, shortest path algorithms, minimum spanning tree algorithms, and advanced graph theory concepts.

## Algorithm Categories

### Graph Traversal
| Algorithm | File | Description |
|-----------|------|-------------|
| [Breadth-First Search](breadth_first_search.md) | `breadth_first_search.rs` | Level-by-level graph exploration |
| [Depth-First Search](depth_first_search.md) | `depth_first_search.rs` | Deep exploration using stack-based traversal |
| [Lee BFS](lee_breadth_first_search.md) | `lee_breadth_first_search.rs` | Shortest path in a maze/grid |

### Shortest Path Algorithms
| Algorithm | File | Description |
|-----------|------|-------------|
| [Dijkstra's Algorithm](dijkstra.md) | `dijkstra.rs` | Single-source shortest path (non-negative weights) |
| [Bellman-Ford](bellman_ford.md) | `bellman_ford.rs` | Single-source shortest path (handles negative weights) |
| [Floyd-Warshall](floyd_warshall.md) | `floyd_warshall.rs` | All-pairs shortest paths |
| [A* Search](astar.md) | `astar.rs` | Heuristic-guided shortest path search |

### Minimum Spanning Tree
| Algorithm | File | Description |
|-----------|------|-------------|
| [Prim's Algorithm](prim.md) | `prim.rs` | Greedy MST construction |
| [Kruskal's Algorithm](minimum_spanning_tree.md) | `minimum_spanning_tree.rs` | Union-Find based MST |

### Graph Ordering & Cycles
| Algorithm | File | Description |
|-----------|------|-------------|
| [Topological Sort](topological_sort.md) | `topological_sort.rs` | Linear ordering of DAG vertices |
| [Detect Cycle](detect_cycle.md) | `detect_cycle.rs` | Cycle detection in directed/undirected graphs |
| [Eulerian Path](eulerian_path.md) | `eulerian_path.rs` | Path visiting every edge exactly once |

### Strongly Connected Components
| Algorithm | File | Description |
|-----------|------|-------------|
| [Tarjan's SCC](tarjans_scc.md) | `tarjans_ssc.rs` | Single DFS-based SCC algorithm |
| [Kosaraju's SCC](kosaraju.md) | `kosaraju.rs` | Two-pass DFS SCC algorithm |
| [Strongly Connected Components](strongly_connected_components.md) | `strongly_connected_components.rs` | Optimized Tarjan's implementation |

### Network Flow
| Algorithm | File | Description |
|-----------|------|-------------|
| [Ford-Fulkerson](ford_fulkerson.md) | `ford_fulkerson.rs` | Maximum flow using augmenting paths |
| [Dinic's Algorithm](dinic_maxflow.md) | `dinic_maxflow.rs` | Efficient max flow with level graphs |
| [Bipartite Matching](bipartite_matching.md) | `bipartite_matching.rs` | Maximum matching in bipartite graphs |

### Tree Algorithms
| Algorithm | File | Description |
|-----------|------|-------------|
| [Lowest Common Ancestor](lowest_common_ancestor.md) | `lowest_common_ancestor.rs` | LCA using Sparse Table & Tarjan's offline |
| [Heavy-Light Decomposition](heavy_light_decomposition.md) | `heavy_light_decomposition.rs` | Tree path decomposition |
| [Centroid Decomposition](centroid_decomposition.md) | `centroid_decomposition.rs` | Divide-and-conquer on trees |
| [Prüfer Code](prufer_code.md) | `prufer_code.rs` | Tree encoding/decoding |

### Data Structures
| Algorithm | File | Description |
|-----------|------|-------------|
| [Disjoint Set Union](disjoint_set_union.md) | `disjoint_set_union.rs` | Union-Find with path compression |

### Miscellaneous
| Algorithm | File | Description |
|-----------|------|-------------|
| [2-SAT](two_satisfiability.md) | `two_satisfiability.rs` | Boolean satisfiability solver |
| [Graph Enumeration](graph_enumeration.md) | `graph_enumeration.rs` | Graph vertex renumbering |
| [Decremental Connectivity](decremental_connectivity.md) | `decremental_connectivity.rs` | Dynamic connectivity with deletions |
| [DFS Tic-Tac-Toe](depth_first_search_tic_tac_toe.md) | `depth_first_search_tic_tac_toe.rs` | Game tree search with minimax |

## Complexity Overview

| Algorithm | Time Complexity | Space Complexity |
|-----------|-----------------|------------------|
| BFS | O(V + E) | O(V) |
| DFS | O(V + E) | O(V) |
| Dijkstra | O(E log V) | O(V) |
| Bellman-Ford | O(V × E) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| A* Search | O(E log V)* | O(V) |
| Prim's MST | O(E log V) | O(V) |
| Kruskal's MST | O(E log E) | O(V) |
| Topological Sort | O(V + E) | O(V) |
| Tarjan's SCC | O(V + E) | O(V) |
| Kosaraju's SCC | O(V + E) | O(V) |
| Ford-Fulkerson | O(E × max_flow) | O(V²) |
| Dinic's | O(V² × E) | O(V²) |

*A* complexity depends on the heuristic quality

## Graph Representations

The implementations in this repository use several graph representations:

### Adjacency List (BTreeMap-based)
```rust
type Graph<V, E> = BTreeMap<V, BTreeMap<V, E>>;
```
Used by: Dijkstra, Bellman-Ford, Floyd-Warshall, A*, Prim

### Custom Structs
```rust
pub struct Graph {
    nodes: Vec<Node>,
    edges: Vec<Edge>,
}
```
Used by: BFS, DFS

### Adjacency List (Vec-based)
```rust
type Graph = Vec<Vec<usize>>;
```
Used by: Tarjan's SCC, Kosaraju, HLD, Centroid Decomposition

## Usage Guidelines

1. **Choosing the Right Algorithm**:
   - For unweighted graphs: BFS/DFS
   - For weighted graphs (non-negative): Dijkstra
   - For graphs with negative edges: Bellman-Ford
   - For all-pairs shortest paths: Floyd-Warshall
   - For heuristic-guided search: A*

2. **Performance Considerations**:
   - Sparse graphs (E ≈ V): Use adjacency lists
   - Dense graphs (E ≈ V²): Consider adjacency matrices
   - Large graphs: Prefer iterative over recursive implementations

3. **Common Pitfalls**:
   - Dijkstra fails with negative edge weights
   - BFS only finds shortest paths in unweighted graphs
   - DFS stack overflow on very deep graphs

## References

- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS)
- Sedgewick, R. "Algorithms in C/C++/Java"
- Skiena, S. "The Algorithm Design Manual"
