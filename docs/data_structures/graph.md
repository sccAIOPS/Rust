# Graph

## 1. Overview

A Graph is a fundamental data structure consisting of vertices (nodes) connected by edges. Graphs model relationships and are used extensively in computer science for networks, social connections, maps, dependencies, and countless other applications. This document covers graph representation and basic operations.

## 2. Mathematical Foundation

### 2.1 Definition

A graph $G = (V, E)$ consists of:
- $V$: Set of vertices (nodes)
- $E$: Set of edges, where each edge connects two vertices

**Types**:
- **Undirected**: Edge $(u, v) = (v, u)$
- **Directed (Digraph)**: Edge $(u, v) \neq (v, u)$
- **Weighted**: Each edge has an associated weight $w(u, v)$
- **Unweighted**: All edges have implicit weight 1

### 2.2 Terminology

| Term | Definition |
|------|------------|
| Adjacent | Two vertices connected by an edge |
| Degree | Number of edges incident to a vertex |
| In-degree | (Directed) Number of incoming edges |
| Out-degree | (Directed) Number of outgoing edges |
| Path | Sequence of vertices connected by edges |
| Cycle | Path that starts and ends at same vertex |
| Connected | Path exists between any two vertices |
| DAG | Directed Acyclic Graph |

### 2.3 Properties

- **Simple Graph**: No self-loops or multi-edges
- **Complete Graph $K_n$**: Every pair of vertices connected
- **Bipartite**: Vertices can be split into two sets with edges only between sets
- **Tree**: Connected graph with no cycles, $|E| = |V| - 1$

## 3. Representations

### 3.1 Adjacency Matrix

2D array where `matrix[i][j]` indicates edge from $i$ to $j$.

```
Graph:          Adjacency Matrix:
  0 --- 1         0  1  2  3
  |     |      0 [0, 1, 1, 0]
  2 --- 3      1 [1, 0, 0, 1]
               2 [1, 0, 0, 1]
               3 [0, 1, 1, 0]
```

**Pros**: O(1) edge lookup, simple  
**Cons**: O(V²) space, inefficient for sparse graphs

### 3.2 Adjacency List

Array of lists where `adj[i]` contains neighbors of vertex $i$.

```
Graph:          Adjacency List:
  0 --- 1       0: [1, 2]
  |     |       1: [0, 3]
  2 --- 3       2: [0, 3]
                3: [1, 2]
```

**Pros**: O(V + E) space, efficient for sparse graphs  
**Cons**: O(degree) edge lookup

### 3.3 Edge List

List of (source, destination, weight) tuples.

```
Graph:          Edge List:
  0 --5-- 1     [(0, 1, 5), (0, 2, 3), (1, 3, 2), (2, 3, 4)]
  |       |
 3|       |2
  |       |
  2 --4-- 3
```

**Pros**: Simple, good for edge-centric algorithms (Kruskal's)  
**Cons**: O(E) for edge lookup, O(E) for neighbor enumeration

### 3.4 Comparison

| Operation | Adj. Matrix | Adj. List | Edge List |
|-----------|-------------|-----------|-----------|
| Space | O(V²) | O(V + E) | O(E) |
| Add edge | O(1) | O(1) | O(1) |
| Remove edge | O(1) | O(degree) | O(E) |
| Check edge | O(1) | O(degree) | O(E) |
| List neighbors | O(V) | O(degree) | O(E) |
| List all edges | O(V²) | O(V + E) | O(E) |

## 4. Algorithm Description

### 4.1 Pseudocode

```
// Adjacency List Representation
ADD_VERTEX(graph):
    graph.adj.append(empty_list)
    return new_vertex_id

ADD_EDGE(graph, u, v, weight):
    graph.adj[u].append((v, weight))
    if undirected:
        graph.adj[v].append((u, weight))

REMOVE_EDGE(graph, u, v):
    graph.adj[u].remove(v)
    if undirected:
        graph.adj[v].remove(u)

NEIGHBORS(graph, u):
    return graph.adj[u]

HAS_EDGE(graph, u, v):
    return v in graph.adj[u]
```

### 4.2 Basic Traversals

**Breadth-First Search (BFS)**:
```
BFS(graph, start):
    visited = set()
    queue = [start]
    visited.add(start)
    
    while queue not empty:
        u = queue.pop_front()
        process(u)
        for v in neighbors(u):
            if v not in visited:
                visited.add(v)
                queue.push_back(v)
```

**Depth-First Search (DFS)**:
```
DFS(graph, start):
    visited = set()
    stack = [start]
    
    while stack not empty:
        u = stack.pop()
        if u not in visited:
            visited.add(u)
            process(u)
            for v in neighbors(u):
                if v not in visited:
                    stack.push(v)
```

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::BTreeMap;

pub type Vertex = usize;
pub type Weight = i32;

#[derive(Debug)]
pub struct DirectedGraph {
    adjacency_list: BTreeMap<Vertex, Vec<(Vertex, Weight)>>,
}

impl DirectedGraph {
    pub fn new() -> Self {
        DirectedGraph {
            adjacency_list: BTreeMap::new(),
        }
    }
    
    pub fn add_vertex(&mut self, v: Vertex) {
        self.adjacency_list.entry(v).or_insert_with(Vec::new);
    }
    
    pub fn add_edge(&mut self, u: Vertex, v: Vertex, weight: Weight) {
        self.add_vertex(u);
        self.add_vertex(v);
        self.adjacency_list.get_mut(&u).unwrap().push((v, weight));
    }
    
    pub fn neighbors(&self, u: Vertex) -> impl Iterator<Item = &(Vertex, Weight)> {
        self.adjacency_list.get(&u).into_iter().flatten()
    }
    
    pub fn vertices(&self) -> impl Iterator<Item = &Vertex> {
        self.adjacency_list.keys()
    }
}

#[derive(Debug)]
pub struct UndirectedGraph {
    adjacency_list: BTreeMap<Vertex, Vec<(Vertex, Weight)>>,
}

impl UndirectedGraph {
    pub fn add_edge(&mut self, u: Vertex, v: Vertex, weight: Weight) {
        self.adjacency_list.entry(u).or_default().push((v, weight));
        self.adjacency_list.entry(v).or_default().push((u, weight));
    }
}
```

**Key Design Patterns**:
- `BTreeMap` for sorted vertex iteration
- `Vec<(Vertex, Weight)>` for adjacency list
- Separate types for directed vs undirected
- Iterator-based neighbor access

### 5.2 Generic Graph Trait

```rust
pub trait Graph {
    fn vertices(&self) -> Vec<Vertex>;
    fn edges(&self) -> Vec<(Vertex, Vertex, Weight)>;
    fn neighbors(&self, v: Vertex) -> Vec<(Vertex, Weight)>;
    fn add_edge(&mut self, u: Vertex, v: Vertex, w: Weight);
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Self-loop | Depends on graph type (allow or reject) |
| Parallel edges | Depends on multigraph support |
| Non-existent vertex | Auto-create or error |
| Negative weights | Valid for some algorithms, not others |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Social Networks**: Friends, followers
2. **Maps/Navigation**: Roads, cities
3. **Web**: Pages, hyperlinks
4. **Dependencies**: Package managers, build systems
5. **Networks**: Computer networks, routing
6. **Recommendations**: Similar items, users

### 6.2 Common Graph Algorithms

| Algorithm | Purpose | Complexity |
|-----------|---------|------------|
| BFS | Shortest path (unweighted) | O(V + E) |
| DFS | Traversal, cycles | O(V + E) |
| Dijkstra | Shortest path (weighted) | O((V + E) log V) |
| Bellman-Ford | Shortest path (negative weights) | O(VE) |
| Floyd-Warshall | All-pairs shortest path | O(V³) |
| Kruskal/Prim | Minimum spanning tree | O(E log E) |
| Topological Sort | DAG ordering | O(V + E) |
| Kosaraju/Tarjan | Strongly connected components | O(V + E) |

### 6.3 Choosing Representation

| Use Case | Best Representation |
|----------|---------------------|
| Dense graph (E ≈ V²) | Adjacency Matrix |
| Sparse graph | Adjacency List |
| Edge-centric algorithms | Edge List |
| Frequent edge queries | Adjacency Matrix |
| Memory constrained | Adjacency List |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Wrong Representation**: Adjacency matrix for sparse graph wastes memory
2. **Directed vs Undirected**: Forgetting to add both directions
3. **Self-Loops**: Can cause infinite loops in traversals
4. **Disconnected Components**: Algorithms may miss vertices
5. **Integer Overflow**: Sum of weights can overflow

### 7.2 Optimization Opportunities

**Compressed Sparse Row (CSR)**:
For static graphs, store edges in contiguous arrays:
```rust
struct CSRGraph {
    row_ptr: Vec<usize>,    // Start of each vertex's edges
    col_idx: Vec<Vertex>,   // Destination vertices
    values: Vec<Weight>,    // Edge weights
}
```

**Benefits**: Cache-friendly, memory-efficient, fast iteration

**Implicit Graphs**: Don't store edges explicitly
```rust
// Grid graph: neighbors are adjacent cells
fn neighbors(v: (usize, usize), rows: usize, cols: usize) -> Vec<(usize, usize)> {
    // Compute neighbors on-the-fly
}
```

### 7.3 Memory Considerations

For graph with V vertices, E edges:

| Representation | Memory |
|----------------|--------|
| Adjacency Matrix | 4V² bytes (i32) |
| Adjacency List | ~16E bytes |
| Edge List | ~12E bytes |
| CSR | ~8E + 4V bytes |

## 8. References

- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapters 22-26.
- Sedgewick, R. (2002). *Algorithms in C++, Part 5: Graph Algorithms*.
- Even, S. (2011). *Graph Algorithms* (2nd ed.). Cambridge University Press.
- Skiena, S. S. (2008). *The Algorithm Design Manual* (2nd ed.). Chapter 5.
