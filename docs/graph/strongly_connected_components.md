# Strongly Connected Components (Optimized Tarjan's)

## 1. Overview

This implementation provides an optimized version of Tarjan's SCC algorithm with efficient state management and reverse topological ordering of components. It uses bit manipulation for tracking visited/in-stack states efficiently.

## 2. Implementation Details

### 2.1 State Encoding

The implementation cleverly encodes multiple states in a single `u64`:
- **MSB (bit 63):** Indicates whether vertex is still in stack (NOT_DONE)
- **Remaining bits:** Discovery time of the vertex

```rust
const NOT_DONE: u64 = 1 << 63;

// Check if vertex is in stack
fn is_in_stack(vertex_state: u64) -> bool {
    vertex_state != 0 && (vertex_state & NOT_DONE) != 0
}

// Check if vertex is unvisited
fn is_unvisited(vertex_state: u64) -> bool {
    vertex_state == NOT_DONE
}

// Extract discovery time
fn get_discover_time(vertex_state: u64) -> u64 {
    vertex_state ^ NOT_DONE
}
```

### 2.2 Algorithm Structure

```rust
pub struct StronglyConnectedComponents {
    pub component: Vec<usize>,      // SCC number for each vertex
    pub state: Vec<u64>,            // Encoded state (discovery time + in-stack)
    pub num_components: usize,      // Total SCC count
    stack: Vec<usize>,              // Current DFS stack
    current_time: usize,            // Discovery time counter
}
```

### 2.3 DFS Function

```rust
fn dfs(&mut self, v: usize, adj: &[Vec<usize>]) -> u64 {
    let mut min_disc = self.current_time as u64;
    self.state[v] ^= min_disc;  // Mark as visited with discovery time
    self.current_time += 1;
    self.stack.push(v);

    for &u in adj[v].iter() {
        if is_unvisited(self.state[u]) {
            min_disc = std::cmp::min(self.dfs(u, adj), min_disc);
        } else if is_in_stack(self.state[u]) {
            min_disc = std::cmp::min(get_discover_time(self.state[u]), min_disc);
        }
    }

    // Root of SCC found
    if min_disc == get_discover_time(self.state[v]) {
        self.num_components += 1;
        loop {
            let u = self.stack.pop().unwrap();
            self.component[u] = self.num_components;
            set_done(&mut self.state[u]);
            if u == v { break; }
        }
    }

    min_disc
}
```

## 3. Key Features

### 3.1 Vertex Numbering

The implementation assumes vertices are numbered from **1 to n** (not 0 to n-1). This is handled by:
- Allocating arrays of size `n + 1`
- Skipping index 0 during processing

### 3.2 Output Format

After `find_components()`:
- `component[v]`: The SCC number that vertex v belongs to
- `state[v]`: The discovery time (MSB cleared)
- `num_components`: Total number of SCCs

**Note:** Components are numbered in **reverse topological order** of the condensation graph.

## 4. Usage Example

```rust
let mut sccs = StronglyConnectedComponents::new(5);
let adj = vec![
    vec![],           // Index 0 (unused)
    vec![2, 4],       // Vertex 1 -> 2, 4
    vec![3, 4],       // Vertex 2 -> 3, 4
    vec![5],          // Vertex 3 -> 5
    vec![5],          // Vertex 4 -> 5
    vec![]            // Vertex 5 -> (none)
];

sccs.find_components(&adj);
// sccs.component gives SCC assignment for each vertex
// sccs.num_components gives total number of SCCs
```

## 5. Complexity

| Operation | Time | Space |
|-----------|------|-------|
| find_components | O(V + E) | O(V) |

## 6. Comparison with Basic Tarjan's

| Aspect | This Implementation | Basic Tarjan's |
|--------|---------------------|----------------|
| State storage | Single u64 per vertex | Separate arrays |
| Memory | More compact | More arrays |
| Vertex indices | 1-based | Usually 0-based |
| Output | Component numbers | Component lists |

## 7. Applications

- Same as standard Tarjan's algorithm
- Particularly useful when component membership (not list) is needed
- Efficient for large graphs due to compact state representation

## 8. References

- Tarjan, R. E. (1972). "Depth-first search and linear graph algorithms"
- Implementation optimizations for competitive programming
