# Eulerian Path and Circuit

## 1. Overview

An **Eulerian path** traverses every edge of a graph exactly once. An **Eulerian circuit** is an Eulerian path that starts and ends at the same vertex. The problem was first solved by Leonhard Euler in 1736 for the famous Seven Bridges of Königsberg problem.

## 2. Mathematical Foundation

### 2.1 Existence Conditions

**Undirected Graph:**
| Type | Condition |
|------|-----------|
| Eulerian Circuit | All vertices have even degree |
| Eulerian Path | Exactly 0 or 2 vertices have odd degree |

**Directed Graph:**
| Type | Condition |
|------|-----------|
| Eulerian Circuit | in-degree = out-degree for all vertices |
| Eulerian Path | At most one vertex with out-degree - in-degree = 1 (start) and at most one with in-degree - out-degree = 1 (end) |

### 2.2 Graph Connectivity

Additionally, the graph must be connected (ignoring isolated vertices).

## 3. Hierholzer's Algorithm

### 3.1 Idea

1. Start at any vertex (or odd-degree vertex for path)
2. Follow edges, removing them as visited
3. When stuck, backtrack and splice in sub-circuits

### 3.2 Pseudocode

```
HIERHOLZER(G, start):
    path ← []
    stack ← [start]
    
    while stack not empty:
        v ← stack.top()
        if v has unvisited edges:
            u ← any neighbor with unvisited edge
            mark edge (v, u) as visited
            stack.push(u)
        else:
            path.append(stack.pop())
    
    return path.reversed()
```

## 4. Example

```
Undirected:
0 —— 1
|  × |
3 —— 2

Degrees: 0(3), 1(3), 2(3), 3(3) - all odd!
No Eulerian circuit (but has Eulerian path: 0-1-2-3-0-2-1-3)
```

```
Directed:
0 → 1 → 2
↑       ↓
← ← 3 ← ←

All vertices: in-degree = out-degree = 1
Eulerian circuit: 0 → 1 → 2 → 3 → 0
```

## 5. Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Check existence | O(V + E) | O(1) |
| Find path/circuit | O(E) | O(E) |

## 6. Implementation

```rust
use std::collections::LinkedList;

pub struct Graph {
    adj: Vec<LinkedList<usize>>,  // Using LinkedList for O(1) edge removal
    num_edges: usize,
}

impl Graph {
    pub fn new(n: usize) -> Self {
        Graph {
            adj: vec![LinkedList::new(); n],
            num_edges: 0,
        }
    }

    pub fn add_edge(&mut self, u: usize, v: usize) {
        self.adj[u].push_back(v);
        self.adj[v].push_back(u);  // Undirected
        self.num_edges += 1;
    }

    fn remove_edge(&mut self, u: usize, v: usize) {
        // Remove v from u's list
        let mut cursor = self.adj[u].cursor_front_mut();
        while let Some(&val) = cursor.current() {
            if val == v {
                cursor.remove_current();
                break;
            }
            cursor.move_next();
        }
        // Remove u from v's list
        let mut cursor = self.adj[v].cursor_front_mut();
        while let Some(&val) = cursor.current() {
            if val == u {
                cursor.remove_current();
                break;
            }
            cursor.move_next();
        }
    }

    pub fn has_eulerian_circuit(&self) -> bool {
        self.adj.iter().all(|neighbors| neighbors.len() % 2 == 0)
    }

    pub fn has_eulerian_path(&self) -> bool {
        let odd_count = self.adj.iter()
            .filter(|neighbors| neighbors.len() % 2 == 1)
            .count();
        odd_count == 0 || odd_count == 2
    }

    pub fn find_eulerian_path(&mut self) -> Option<Vec<usize>> {
        if !self.has_eulerian_path() {
            return None;
        }

        // Find starting vertex (odd degree or any if circuit)
        let start = self.adj.iter()
            .position(|neighbors| neighbors.len() % 2 == 1)
            .unwrap_or(0);

        let mut path = Vec::new();
        let mut stack = vec![start];

        while let Some(&v) = stack.last() {
            if self.adj[v].is_empty() {
                path.push(stack.pop().unwrap());
            } else {
                let u = self.adj[v].front().copied().unwrap();
                self.remove_edge(v, u);
                stack.push(u);
            }
        }

        path.reverse();
        
        // Verify all edges used
        if path.len() == self.num_edges + 1 {
            Some(path)
        } else {
            None  // Graph was not connected
        }
    }
}
```

### 6.1 Directed Graph Version

```rust
pub struct DiGraph {
    adj: Vec<Vec<usize>>,
    in_degree: Vec<usize>,
    out_degree: Vec<usize>,
}

impl DiGraph {
    pub fn find_eulerian_path(&mut self) -> Option<Vec<usize>> {
        // Find start (out > in) and end (in > out)
        let mut start = 0;
        let mut end = 0;
        let mut start_nodes = 0;
        let mut end_nodes = 0;

        for v in 0..self.adj.len() {
            let diff = self.out_degree[v] as i32 - self.in_degree[v] as i32;
            match diff {
                1 => { start = v; start_nodes += 1; }
                -1 => { end = v; end_nodes += 1; }
                0 => {}
                _ => return None,
            }
        }

        // Valid: either circuit (0,0) or path (1,1)
        if !((start_nodes == 0 && end_nodes == 0) || 
             (start_nodes == 1 && end_nodes == 1)) {
            return None;
        }

        let mut path = Vec::new();
        let mut stack = vec![start];
        let mut edge_ptr = vec![0; self.adj.len()];

        while let Some(&v) = stack.last() {
            if edge_ptr[v] < self.adj[v].len() {
                let u = self.adj[v][edge_ptr[v]];
                edge_ptr[v] += 1;
                stack.push(u);
            } else {
                path.push(stack.pop().unwrap());
            }
        }

        path.reverse();
        Some(path)
    }
}
```

## 7. Applications

1. **Route planning:** Traverse all roads exactly once
2. **DNA sequencing:** Reconstruct sequence from fragments
3. **Drawing puzzles:** Draw figure without lifting pen
4. **Circuit design:** Testing all connections
5. **Snow plow routing:** Cover all streets efficiently

## 8. Related Problems

| Problem | Description |
|---------|-------------|
| Hamiltonian Path | Visit all vertices exactly once (NP-complete) |
| Chinese Postman | Minimum cost to traverse all edges |
| de Bruijn Sequence | Eulerian path in de Bruijn graph |
| Seven Bridges | Original Eulerian problem |

## 9. Variations

### 9.1 Fleury's Algorithm

Simpler but slower O(E²):
- Always choose non-bridge edge if possible
- Use Tarjan's bridge algorithm or DFS to check

### 9.2 Chinese Postman Problem

When no Eulerian path exists, find minimum-cost edge duplications to create one.

## 10. Edge Cases

| Case | Behavior |
|------|----------|
| Empty graph | Trivial path/circuit |
| Single vertex | Valid circuit |
| Disconnected | No Eulerian path |
| All even degrees | Eulerian circuit exists |
| Two odd degrees | Eulerian path (not circuit) |

## 11. Common Pitfalls

1. **Connectivity check:** Graph must be connected
2. **Edge removal:** Use efficient data structure
3. **Directed vs undirected:** Different existence conditions
4. **Multi-edges:** Handle duplicate edges correctly
5. **Self-loops:** Count as adding 2 to degree

## 12. References

- Hierholzer, C. (1873). "Über die Möglichkeit, einen Linienzug ohne Wiederholung und ohne Unterbrechung zu umfahren"
- Euler, L. (1736). "Solutio problematis ad geometriam situs pertinentis"
- Fleury's algorithm (1883)
