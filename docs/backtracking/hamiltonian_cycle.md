# Hamiltonian Cycle

## 1. Overview

A **Hamiltonian Cycle** (or Hamiltonian circuit) is a cycle in a graph that visits every vertex exactly once and returns to the starting vertex. The problem of finding such a cycle is one of the classic NP-complete problems in graph theory.

Named after Sir William Rowan Hamilton, who invented a puzzle (the Icosian game) based on finding Hamiltonian cycles on a dodecahedron in 1857.

### Historical Context
- **1857**: Hamilton's Icosian game introduced the concept
- **1859**: Thomas Kirkman independently studied related problems
- **1972**: Karp proved Hamiltonian Cycle is NP-complete
- **Modern**: Applications in routing, genome assembly, circuit design

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a graph $G = (V, E)$, find a cycle $C = (v_1, v_2, ..., v_n, v_1)$ such that:

1. Every vertex $v \in V$ appears exactly once in $v_1, v_2, ..., v_n$
2. $(v_i, v_{i+1}) \in E$ for all $i \in [1, n-1]$
3. $(v_n, v_1) \in E$ (closes the cycle)

### 2.2 Mathematical Model

**Input**: 
- Adjacency matrix $A$ where $A[i][j] = \text{true}$ if edge $(i, j)$ exists
- Starting vertex $s$

**Output**: A Hamiltonian cycle starting from $s$, or `None` if none exists

**Related Concepts**:
- **Hamiltonian Path**: Visits all vertices exactly once (no return)
- **Eulerian Cycle**: Visits all edges exactly once
- **Traveling Salesman**: Shortest Hamiltonian cycle (weighted graphs)

### 2.3 Existence Conditions

| Condition | Guarantees Hamiltonian Cycle? |
|-----------|------------------------------|
| Complete graph $K_n$ | Yes (for $n \geq 3$) |
| Dirac's condition: $\deg(v) \geq n/2$ for all $v$ | Yes |
| Ore's condition: $\deg(u) + \deg(v) \geq n$ for non-adjacent $u, v$ | Yes |
| Bipartite with unequal partitions | No |
| Graph with cut vertex | No |

## 3. Algorithm Description

### 3.1 Intuition

The backtracking approach builds the cycle incrementally:
1. Start from a given vertex, mark it as visited
2. Find an unvisited adjacent vertex, move there, mark visited
3. Repeat until all vertices are visited
4. Check if there's an edge back to the start
5. If stuck (no valid neighbors), backtrack and try another path

### 3.2 Pseudocode

```
function find_hamiltonian_cycle(adjacency_matrix, start_vertex):
    graph = new Graph(adjacency_matrix)
    path = array of size n, initialized to None
    visited = array of size n, initialized to false
    
    path[0] = start_vertex
    visited[start_vertex] = true
    
    if hamiltonian_cycle_util(path, visited, 1):
        path.append(start_vertex)  // Close the cycle
        return path
    else:
        return None

function hamiltonian_cycle_util(path, visited, pos):
    if pos == n:
        // All vertices visited, check for edge back to start
        return adjacency_matrix[path[pos-1]][path[0]]
    
    for v in 0 to n-1:
        if is_safe(v, visited, path, pos):
            path[pos] = v
            visited[v] = true
            
            if hamiltonian_cycle_util(path, visited, pos + 1):
                return true
            
            // Backtrack
            path[pos] = None
            visited[v] = false
    
    return false

function is_safe(v, visited, path, pos):
    // Check edge exists from previous vertex
    if not adjacency_matrix[path[pos-1]][v]:
        return false
    // Check vertex not already visited
    return not visited[v]
```

### 3.3 Step-by-Step Example

For a square graph (4 vertices):

```
Graph:  0 --- 1
        |     |
        3 --- 2

Starting from vertex 0:

hamiltonian_cycle_util(path=[0,_,_,_], pos=1):
├─ v=1: is_safe? edge(0,1)=T, visited[1]=F → SAFE
│  path=[0,1,_,_], visited=[T,T,F,F]
│  hamiltonian_cycle_util(pos=2):
│  ├─ v=2: is_safe? edge(1,2)=T, visited[2]=F → SAFE
│  │  path=[0,1,2,_], visited=[T,T,T,F]
│  │  hamiltonian_cycle_util(pos=3):
│  │  └─ v=3: is_safe? edge(2,3)=T, visited[3]=F → SAFE
│  │     path=[0,1,2,3], visited=[T,T,T,T]
│  │     hamiltonian_cycle_util(pos=4):
│  │     pos == n, check edge(3,0)=T → SUCCESS!
│  │     return true
│  ...

Result: [0, 1, 2, 3, 0]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Worst Case**: $O(N!)$
  - At each vertex, we try up to $N-1$ unvisited vertices
  - Depth of recursion is $N$
  - Total: $(N-1)!$ paths to explore ≈ $O(N!)$

- **Best Case**: $O(N)$ when a Hamiltonian cycle is found quickly

- **Average Case**: Depends heavily on graph density
  - Dense graphs: More choices but also more constraints
  - Sparse graphs: Fewer choices, faster pruning

### 4.2 Space Complexity

- **Path Array**: $O(N)$
- **Visited Array**: $O(N)$
- **Recursion Stack**: $O(N)$

**Total**: $O(N)$ auxiliary space

### 4.3 Comparison with Related Problems

| Problem | Time | Deterministic? |
|---------|------|----------------|
| Hamiltonian Cycle (decision) | $O(N!)$ backtracking | Yes |
| Hamiltonian Cycle (randomized) | Better average | No |
| Eulerian Cycle | $O(E)$ | Yes |
| TSP (exact) | $O(N^2 \cdot 2^N)$ DP | Yes |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Debug, PartialEq, Eq)]
pub enum FindHamiltonianCycleError {
    EmptyAdjacencyMatrix,
    ImproperAdjacencyMatrix,
    StartOutOfBound,
}

// Using Option<usize> for path elements
path: Vec<Option<usize>>  // None = not yet assigned

// Boolean adjacency matrix
adjacency_matrix: Vec<Vec<bool>>
```

**Key Patterns**:
- **Custom error enum**: Comprehensive error handling
- **Option type**: Clear distinction between unassigned and assigned
- **Graph struct**: Encapsulates adjacency matrix and operations
- **Builder pattern**: `Graph::new()` validates input

### 5.2 Edge Cases

| Case | Graph | Start | Result |
|------|-------|-------|--------|
| Empty matrix | `[]` | 0 | `Err(EmptyAdjacencyMatrix)` |
| Non-square | irregular | 0 | `Err(ImproperAdjacencyMatrix)` |
| Start out of bounds | valid | n | `Err(StartOutOfBound)` |
| Single vertex with self-loop | `[[true]]` | 0 | `Ok(Some([0, 0]))` |
| Single vertex no loop | `[[false]]` | 0 | `Ok(None)` |
| Complete graph K₄ | complete | 0 | `Ok(Some([...]))` |
| Path graph (no cycle) | path | 0 | `Ok(None)` |
| Tree (no cycle) | tree | 0 | `Ok(None)` |

### 5.3 Potential Optimizations

1. **Degree-based pruning**: Vertices with degree < 2 can't be in a cycle
2. **Warnsdorff's heuristic**: Prefer vertices with fewer unvisited neighbors
3. **Randomized selection**: Avoid worst-case systematic exploration
4. **Held-Karp algorithm**: DP approach for TSP gives Hamiltonian cycle in $O(n^2 \cdot 2^n)$

```rust
// Warnsdorff's heuristic
fn next_vertex_warnsdorff(&self, current: usize, visited: &[bool]) -> Option<usize> {
    let unvisited_neighbors = (0..self.num_vertices())
        .filter(|&v| !visited[v] && self.adjacency_matrix[current][v])
        .collect::<Vec<_>>();
    
    // Choose neighbor with minimum degree among unvisited
    unvisited_neighbors.into_iter()
        .min_by_key(|&v| self.count_unvisited_neighbors(v, visited))
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Circuit Board Design**: Visit all connection points exactly once
2. **Genome Assembly**: Order DNA fragments into complete sequence
3. **Route Planning**: Delivery routes visiting all locations
4. **Network Design**: Optimal ring topology construction
5. **Puzzle Games**: Knight's tour, Snake game paths

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|-------------|
| Traveling Salesman | Weighted Hamiltonian cycle |
| Knight's Tour | Hamiltonian path on knight's graph |
| Eulerian Cycle | All edges vs. all vertices |
| Depth-First Search | Foundation for backtracking |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn is_safe(&self, v: usize, visited: &[bool], path: &[Option<usize>], pos: usize) -> bool {
    // Check edge from previous vertex
    if !self.adjacency_matrix[path[pos - 1].unwrap()][v] {
        return false;
    }
    // Check not already visited
    !visited[v]
}

fn hamiltonian_cycle_util(
    &self,
    path: &mut [Option<usize>],
    visited: &mut [bool],
    pos: usize,
) -> bool {
    if pos == self.num_vertices() {
        // Check edge back to start
        return self.adjacency_matrix[path[pos - 1].unwrap()][path[0].unwrap()];
    }
    
    for v in 0..self.num_vertices() {
        if self.is_safe(v, visited, path, pos) {
            path[pos] = Some(v);
            visited[v] = true;
            if self.hamiltonian_cycle_util(path, visited, pos + 1) {
                return true;
            }
            path[pos] = None;
            visited[v] = false;
        }
    }
    false
}
```

**Analysis**:
- ✅ Correctly checks edge existence and visited status
- ✅ Properly closes the cycle by checking return edge
- ✅ Clean backtracking with state restoration
- ✅ Uses `Option<usize>` for clear path representation
- ⚠️ Could use degree heuristics for better average performance

### 7.2 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| `is_safe` | O(1) | ✅ Array lookups only |
| `hamiltonian_cycle_util` | O(N!) | ✅ N choices at N-1 levels |
| Total | O(N!) | ✅ |

## 8. Testing

### 8.1 Test Cases Covered

```rust
test_complete_graph: K₄ → Some([0,1,2,3,0])
test_directed_graph_with_cycle: 5 vertices → cycle exists
test_undirected_graph_with_cycle: 5 vertices → cycle exists
test_directed_graph_no_cycle: 5 vertices → None
test_undirected_graph_no_cycle: 5 vertices → None
test_triangle_graph: K₃ directed → Some([1,2,0,1])
test_tree_graph: 5 vertices → None (trees have no cycles)
test_empty_graph: [] → Err(EmptyAdjacencyMatrix)
test_improper_graph: non-square → Err(ImproperAdjacencyMatrix)
test_start_out_of_bound: → Err(StartOutOfBound)
test_complex_directed_graph: 6 vertices → cycle exists
single_node_self_loop: [[true]] → Some([0,0])
single_node: [[false]] → None
```

### 8.2 Test Coverage Analysis

- ✅ Complete graphs
- ✅ Directed graphs
- ✅ Undirected graphs
- ✅ Trees (no cycle possible)
- ✅ Various starting vertices
- ✅ Error cases
- ✅ Single vertex with/without self-loop

## 9. Theoretical Background

### 9.1 NP-Completeness

The **Hamiltonian Cycle problem** is NP-complete:
- **In NP**: A proposed cycle can be verified in O(N)
- **NP-hard**: Reduction from 3-SAT or Vertex Cover

**Implications**:
- No known polynomial-time algorithm
- Unlikely to find one (unless P = NP)
- Heuristics and approximations are important

### 9.2 Sufficient Conditions (Guarantee Existence)

**Dirac's Theorem (1952)**:
If $|V| \geq 3$ and every vertex has degree $\geq |V|/2$, then $G$ has a Hamiltonian cycle.

**Ore's Theorem (1960)**:
If $|V| \geq 3$ and for every pair of non-adjacent vertices $u, v$: $\deg(u) + \deg(v) \geq |V|$, then $G$ has a Hamiltonian cycle.

### 9.3 Necessary Conditions (Must Be Satisfied)

1. Graph must be connected
2. No cut vertices (articulation points) if cycle must pass through all
3. For bipartite graphs: equal partition sizes

## 10. References

1. Karp, R. M. (1972). "Reducibility Among Combinatorial Problems". *Complexity of Computer Computations*.
2. Garey, M. R., & Johnson, D. S. (1979). *Computers and Intractability*.
3. Dirac, G. A. (1952). "Some theorems on abstract graphs". *Proceedings of the London Mathematical Society*.
4. Ore, O. (1960). "Note on Hamilton circuits". *American Mathematical Monthly*.
5. [Wikipedia: Hamiltonian Path Problem](https://en.wikipedia.org/wiki/Hamiltonian_path_problem)
