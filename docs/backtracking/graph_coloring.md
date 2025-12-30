# Graph Coloring

## 1. Overview

**Graph Coloring** is the problem of assigning colors (labels) to vertices of a graph such that no two adjacent vertices share the same color. The goal is typically to find a valid coloring using a given number of colors, or to find the minimum number of colors needed (the **chromatic number**).

This implementation generates all possible valid colorings of a graph given a maximum number of colors.

### Historical Context
- **1852**: Four Color Problem proposed (maps)
- **1879**: Kempe's flawed "proof" introduced key techniques
- **1976**: Four Color Theorem proved by Appel and Haken (computer-assisted)
- **Modern**: Applications in scheduling, register allocation, frequency assignment

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an undirected graph $G = (V, E)$ and $k$ colors, assign a color $c_v \in \{0, 1, ..., k-1\}$ to each vertex $v \in V$ such that:

$$\forall (u, v) \in E: c_u \neq c_v$$

### 2.2 Mathematical Model

**Input**: 
- Adjacency matrix $A$ where $A[i][j] = \text{true}$ if edge $(i, j)$ exists
- Number of colors $k$

**Output**: All valid colorings, each as a vector of vertex colors

**Chromatic Number**: Minimum $k$ for which a valid coloring exists: $\chi(G)$

### 2.3 Special Graph Chromatic Numbers

| Graph Type | Chromatic Number |
|------------|------------------|
| Empty graph (no edges) | 1 |
| Tree | 2 |
| Cycle (even) | 2 |
| Cycle (odd) | 3 |
| Complete graph $K_n$ | $n$ |
| Bipartite graph | 2 |
| Planar graph | ≤ 4 (Four Color Theorem) |

## 3. Algorithm Description

### 3.1 Intuition

The backtracking approach assigns colors vertex by vertex:
1. Start with vertex 0
2. Try each color from 0 to k-1
3. If the color is valid (no adjacent vertex has it), assign and proceed
4. If all vertices are colored, record the solution
5. Backtrack by unassigning the color and trying the next

### 3.2 Pseudocode

```
function generate_colorings(adjacency_matrix, num_colors):
    graph = new GraphColoring(adjacency_matrix)
    return graph.find_solutions(num_colors)

function find_colorings(vertex, num_colors):
    if vertex == num_vertices:
        record current coloring as solution
        return
    
    for color in 0 to num_colors - 1:
        if is_color_valid(vertex, color):
            vertex_colors[vertex] = color
            find_colorings(vertex + 1, num_colors)
            vertex_colors[vertex] = UNASSIGNED  // backtrack

function is_color_valid(vertex, color):
    for each neighbor of vertex:
        if vertex_colors[neighbor] == color:
            return false
    return true
```

### 3.3 Step-by-Step Example

For a triangle graph (K₃) with 3 colors:

```
Graph:    0 --- 1
           \   /
            \ /
             2

find_colorings(vertex=0):
├─ color=0: colors=[0, _, _]
│  find_colorings(vertex=1):
│  ├─ color=0: INVALID (adjacent to 0)
│  ├─ color=1: colors=[0, 1, _]
│  │  find_colorings(vertex=2):
│  │  ├─ color=0: INVALID (adjacent to 0)
│  │  ├─ color=1: INVALID (adjacent to 1)
│  │  └─ color=2: colors=[0, 1, 2] → SOLUTION
│  └─ color=2: colors=[0, 2, _]
│     find_colorings(vertex=2):
│     └─ color=1: colors=[0, 2, 1] → SOLUTION
├─ color=1: ... (generates [1, 0, 2], [1, 2, 0])
└─ color=2: ... (generates [2, 0, 1], [2, 1, 0])

Total: 6 solutions (3! permutations of colors)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Worst Case**: $O(k^V)$ where $V$ is the number of vertices
  - At each of $V$ vertices, try up to $k$ colors
  - Forms a tree of height $V$ with branching factor up to $k$

- **With Pruning**: Much better in practice
  - Invalid colorings are rejected early
  - Dense graphs have more constraints → more pruning

- **Per-Node Work**: $O(V)$ to check if color is valid

**Total**: $O(k^V \times V)$ worst case

### 4.2 Space Complexity

- **Color Assignment**: $O(V)$
- **Recursion Stack**: $O(V)$
- **Solutions Storage**: $O(S \times V)$ where $S$ is the number of solutions

### 4.3 Solution Count

For a graph with chromatic number $\chi(G)$:
- If $k < \chi(G)$: 0 solutions
- If $k = \chi(G)$: Depends on graph structure
- If $k > \chi(G)$: Generally many solutions (includes permutations)

For complete graph $K_n$ with $k=n$ colors: $n!$ solutions

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Debug, PartialEq, Eq)]
pub enum GraphColoringError {
    EmptyAdjacencyMatrix,
    ImproperAdjacencyMatrix,
}

struct GraphColoring {
    adjacency_matrix: Vec<Vec<bool>>,
    vertex_colors: Vec<usize>,    // usize::MAX = unassigned
    solutions: Vec<Vec<usize>>,
}
```

**Key Patterns**:
- **Custom error enum**: Clear error discrimination
- **Sentinel value**: `usize::MAX` for unassigned vertices
- **Builder pattern**: `new()` → `find_solutions()` workflow
- **Memory efficiency**: Uses `std::mem::take` for solutions

### 5.2 Edge Cases

| Case | Adjacency Matrix | Colors | Result |
|------|------------------|--------|--------|
| Empty matrix | `[]` | any | `Err(EmptyAdjacencyMatrix)` |
| Non-square matrix | `[[T], [T,F]]` | any | `Err(ImproperAdjacencyMatrix)` |
| Single vertex | `[[F]]` | 1 | `Ok(Some([[0]]))` |
| Complete K₃ | triangle | 2 | `Ok(None)` |
| Complete K₃ | triangle | 3 | `Ok(Some([6 colorings]))` |
| Disconnected | no edges | 2 | `Ok(Some([8 colorings]))` |
| Zero colors | any | 0 | `Ok(None)` |

### 5.3 Potential Optimizations

1. **Degree ordering**: Color high-degree vertices first (more constraints)
2. **MRV heuristic**: Color vertex with fewest legal colors
3. **Forward checking**: Maintain domains for unassigned vertices
4. **Symmetry breaking**: Fix color of first vertex to reduce solutions

```rust
// Symmetry breaking: first vertex always gets color 0
fn find_colorings_symmetric(&mut self, num_colors: usize) {
    self.vertex_colors[0] = 0;
    self.find_colorings_helper(1, num_colors);
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Register Allocation**: Assign variables to registers (interference graph)
2. **Scheduling**: Time slots for exams/meetings without conflicts
3. **Frequency Assignment**: Radio/cellular frequencies
4. **Map Coloring**: Regions without adjacent same colors
5. **Pattern Matching**: Constraint satisfaction in databases

### 6.2 Related Algorithms

| Algorithm | Use Case |
|-----------|----------|
| N-Queens | Special case of graph coloring |
| Sudoku | Graph coloring on Sudoku graph |
| SAT Solving | Graph coloring reduces to SAT |
| Maximum Clique | Related to chromatic number |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn is_color_valid(&self, vertex: usize, color: usize) -> bool {
    for neighbor in 0..self.num_vertices() {
        // Check both directions for undirected graph
        if (self.adjacency_matrix[vertex][neighbor] 
            || self.adjacency_matrix[neighbor][vertex])
            && self.vertex_colors[neighbor] == color
        {
            return false;
        }
    }
    true
}
```

**Analysis**:
- ✅ Correctly handles both directed and undirected graphs
- ✅ Uses `usize::MAX` as unassigned sentinel
- ✅ Generates all solutions, not just one
- ⚠️ Checks all vertices, not just neighbors (O(V) instead of O(degree))
- ⚠️ Could use adjacency list for sparse graphs

### 7.2 Directed Graph Support

The implementation treats directed edges bidirectionally:
```rust
if (self.adjacency_matrix[vertex][neighbor] || self.adjacency_matrix[neighbor][vertex])
```

This means an edge in either direction creates a coloring constraint.

### 7.3 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| `is_color_valid` | O(V) | ✅ Iterates all vertices |
| `find_colorings` | O(k^V) | ✅ k choices at V levels |
| Total | O(k^V × V) | ✅ |

## 8. Testing

### 8.1 Test Cases Covered

```rust
test_complete_graph_with_3_colors: 4 vertices, 3 colors → 6 solutions
test_linear_graph_with_2_colors: path graph → 2 solutions
test_incomplete_graph_with_insufficient_colors: K₃ with 1 color → None
test_empty_graph: [] → Err(EmptyAdjacencyMatrix)
test_non_square_matrix: → Err(ImproperAdjacencyMatrix)
test_single_vertex_graph: 1 vertex → [[0]]
test_bipartite_graph_with_2_colors: cycle₄ → 2 solutions
test_large_graph_with_3_colors: 10 vertices → 6 solutions
test_disconnected_graph: 3 vertices, no edges → 8 solutions
test_no_valid_coloring: K₃ with 2 colors → None
test_more_colors_than_nodes: 2 vertices, 3 colors → 6 solutions
test_no_coloring_with_zero_colors: → None
test_complete_graph_with_3_vertices_and_3_colors: K₃ → 6 solutions
test_directed_graph_with_3_colors: → 6 solutions
test_directed_graph_no_valid_coloring: → None
test_large_directed_graph_with_3_colors: 10 vertices → 6 solutions
```

### 8.2 Test Coverage Analysis

- ✅ Various graph sizes
- ✅ Complete graphs
- ✅ Path/linear graphs
- ✅ Bipartite graphs
- ✅ Disconnected graphs
- ✅ Directed graphs
- ✅ Error cases (empty, non-square)
- ✅ Edge cases (single vertex, zero colors)

## 9. Theoretical Background

### 9.1 NP-Completeness

**k-Colorability** (for fixed k ≥ 3) is NP-complete:
- **In NP**: Verify a coloring in O(E) time
- **NP-hard**: Reduction from 3-SAT

Special cases:
- 2-colorability is in P (bipartite testing)
- 3-colorability is NP-complete

### 9.2 Graph Coloring Variants

| Variant | Description |
|---------|-------------|
| Vertex coloring | This implementation |
| Edge coloring | Color edges, not vertices |
| List coloring | Each vertex has allowed color list |
| Fractional coloring | Probabilistic relaxation |

### 9.3 Chromatic Polynomial

The **chromatic polynomial** $P(G, k)$ counts the number of proper $k$-colorings:
- For empty graph $E_n$: $P(E_n, k) = k^n$
- For complete graph $K_n$: $P(K_n, k) = k(k-1)(k-2)...(k-n+1)$
- For tree $T_n$: $P(T_n, k) = k(k-1)^{n-1}$

## 10. References

1. Garey, M. R., & Johnson, D. S. (1979). *Computers and Intractability: A Guide to NP-Completeness*. 
2. Jensen, T. R., & Toft, B. (1995). *Graph Coloring Problems*. Wiley.
3. Appel, K., & Haken, W. (1977). "Every planar map is four colorable". *Illinois Journal of Mathematics*.
4. Brelaz, D. (1979). "New methods to color the vertices of a graph". *Communications of the ACM*.
