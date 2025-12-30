# Decremental Connectivity

## 1. Overview

Decremental connectivity is the problem of maintaining connected components in a graph under edge deletions. Unlike incremental connectivity (which can be solved efficiently with Union-Find), decremental connectivity is more challenging and typically requires O(log n) or O(log² n) amortized time per operation.

## 2. Problem Definition

### 2.1 Operations

- **Delete(u, v):** Remove edge (u, v) from the graph
- **Connected(u, v):** Query if u and v are in the same connected component

### 2.2 Challenge

Union-Find handles additions well but doesn't support deletions. When an edge is deleted, we need to determine if the component splits and update accordingly.

## 3. Approaches

### 3.1 Naive Approach

Rebuild connected components after each deletion:
- Time per deletion: O(V + E)
- Time per query: O(1)
- Total for m deletions: O(m(V + E))

### 3.2 Link-Cut Trees

Use link-cut trees for dynamic forests:
- Time per operation: O(log n) amortized
- Complex implementation

### 3.3 Euler Tour Trees (ET-Trees)

Maintain spanning forest using Euler tour representation:
- Time per deletion: O(log² n) amortized
- Practical for moderate-sized graphs

### 3.4 Even-Shiloach Algorithm (BFS-based)

For unweighted graphs with bounded deletions:
- Maintain BFS tree from each vertex
- Time: O(V × E) total for all deletions

## 4. Even-Shiloach Algorithm

### 4.1 Key Idea

Maintain a spanning tree rooted at source. When an edge is deleted:
1. If edge not in spanning tree → no change
2. If edge in spanning tree → search for replacement edge

### 4.2 Pseudocode

```
INITIALIZE(G, source):
    BFS to compute distances from source
    For each vertex v: level[v] = distance from source
    spanning_tree_edges = BFS tree edges

DELETE(u, v):
    if (u, v) not in spanning_tree:
        return  // Connectivity unchanged
    
    // Assume level[v] = level[u] + 1 (v is child)
    // Search for replacement edge
    queue = [v]
    while queue not empty:
        x = queue.pop()
        
        for each neighbor y of x:
            if level[y] = level[x] - 1:
                // Found replacement: edge (x, y)
                add (x, y) to spanning_tree
                return
        
        // No replacement at this level, demote x
        level[x] += 1
        if level[x] > n:
            // x disconnected from source
            mark x as unreachable
        else:
            // Re-examine x's neighbors
            for each neighbor y of x:
                if level[y] = level[x] - 1:
                    queue.push(x)
                    break
```

## 5. Complexity Analysis

| Algorithm | Deletion | Query | Total (m ops) |
|-----------|----------|-------|---------------|
| Naive | O(V + E) | O(1) | O(m(V + E)) |
| Even-Shiloach | O(V + E) amortized total | O(1) | O(VE) |
| Link-Cut Trees | O(log n) | O(log n) | O(m log n) |
| ET-Trees | O(log² n) | O(log n) | O(m log² n) |

## 6. Implementation (Simplified Even-Shiloach)

```rust
use std::collections::{HashMap, HashSet, VecDeque};

pub struct DecrementalConnectivity {
    n: usize,
    adj: Vec<HashSet<usize>>,
    level: Vec<usize>,
    tree_parent: Vec<Option<usize>>,
    source: usize,
}

impl DecrementalConnectivity {
    pub fn new(n: usize, edges: &[(usize, usize)], source: usize) -> Self {
        let mut adj = vec![HashSet::new(); n];
        for &(u, v) in edges {
            adj[u].insert(v);
            adj[v].insert(u);
        }
        
        // BFS to initialize levels and spanning tree
        let mut level = vec![usize::MAX; n];
        let mut tree_parent = vec![None; n];
        let mut queue = VecDeque::new();
        
        level[source] = 0;
        queue.push_back(source);
        
        while let Some(u) = queue.pop_front() {
            for &v in &adj[u] {
                if level[v] == usize::MAX {
                    level[v] = level[u] + 1;
                    tree_parent[v] = Some(u);
                    queue.push_back(v);
                }
            }
        }
        
        DecrementalConnectivity {
            n,
            adj,
            level,
            tree_parent,
            source,
        }
    }

    pub fn delete(&mut self, u: usize, v: usize) {
        // Remove edge from adjacency
        self.adj[u].remove(&v);
        self.adj[v].remove(&u);
        
        // Check if edge is in spanning tree
        let (parent, child) = if self.tree_parent[v] == Some(u) {
            (u, v)
        } else if self.tree_parent[u] == Some(v) {
            (v, u)
        } else {
            return;  // Not a tree edge
        };
        
        // Try to find replacement edge
        self.tree_parent[child] = None;
        
        if let Some(replacement) = self.find_replacement(child) {
            self.tree_parent[child] = Some(replacement);
        } else {
            // Subtree rooted at child is disconnected
            self.disconnect_subtree(child);
        }
    }

    fn find_replacement(&self, v: usize) -> Option<usize> {
        // Look for edge to vertex at level[v] - 1
        let target_level = self.level[v].saturating_sub(1);
        
        for &u in &self.adj[v] {
            if self.level[u] == target_level {
                return Some(u);
            }
        }
        
        None
    }

    fn disconnect_subtree(&mut self, root: usize) {
        // Mark all vertices in subtree as disconnected
        let mut queue = VecDeque::new();
        queue.push_back(root);
        
        while let Some(v) = queue.pop_front() {
            self.level[v] = usize::MAX;
            
            // Find children in tree
            for &u in &self.adj[v] {
                if self.tree_parent[u] == Some(v) {
                    queue.push_back(u);
                }
            }
        }
    }

    pub fn connected(&self, u: usize, v: usize) -> bool {
        // Both must be reachable from source
        self.level[u] != usize::MAX && self.level[v] != usize::MAX
    }

    pub fn distance(&self, v: usize) -> Option<usize> {
        if self.level[v] == usize::MAX {
            None
        } else {
            Some(self.level[v])
        }
    }
}
```

## 7. Full Decremental Connectivity (Multi-Source)

```rust
pub struct FullDecrementalConnectivity {
    n: usize,
    adj: Vec<HashSet<usize>>,
    component: Vec<usize>,
}

impl FullDecrementalConnectivity {
    pub fn new(n: usize, edges: &[(usize, usize)]) -> Self {
        let mut adj = vec![HashSet::new(); n];
        for &(u, v) in edges {
            adj[u].insert(v);
            adj[v].insert(u);
        }
        
        // Find initial components via DFS/BFS
        let component = Self::find_components(n, &adj);
        
        FullDecrementalConnectivity { n, adj, component }
    }

    fn find_components(n: usize, adj: &[HashSet<usize>]) -> Vec<usize> {
        let mut component = vec![usize::MAX; n];
        let mut comp_id = 0;
        
        for start in 0..n {
            if component[start] != usize::MAX {
                continue;
            }
            
            let mut stack = vec![start];
            while let Some(v) = stack.pop() {
                if component[v] != usize::MAX {
                    continue;
                }
                component[v] = comp_id;
                
                for &u in &adj[v] {
                    if component[u] == usize::MAX {
                        stack.push(u);
                    }
                }
            }
            comp_id += 1;
        }
        
        component
    }

    pub fn delete(&mut self, u: usize, v: usize) {
        if !self.adj[u].contains(&v) {
            return;
        }
        
        self.adj[u].remove(&v);
        self.adj[v].remove(&u);
        
        // Check if deletion splits component
        if !self.still_connected(u, v) {
            // Assign new component to smaller side
            self.split_component(u, v);
        }
    }

    fn still_connected(&self, u: usize, v: usize) -> bool {
        // BFS from u to check if v is reachable
        let mut visited = HashSet::new();
        let mut queue = VecDeque::new();
        queue.push_back(u);
        visited.insert(u);
        
        while let Some(x) = queue.pop_front() {
            if x == v {
                return true;
            }
            for &y in &self.adj[x] {
                if !visited.contains(&y) {
                    visited.insert(y);
                    queue.push_back(y);
                }
            }
        }
        
        false
    }

    fn split_component(&mut self, u: usize, v: usize) {
        // Find vertices reachable from v (smaller search)
        let new_comp = self.component.iter().max().unwrap_or(&0) + 1;
        
        let mut queue = VecDeque::new();
        queue.push_back(v);
        
        while let Some(x) = queue.pop_front() {
            if self.component[x] == new_comp {
                continue;
            }
            self.component[x] = new_comp;
            
            for &y in &self.adj[x] {
                if self.component[y] != new_comp {
                    queue.push_back(y);
                }
            }
        }
    }

    pub fn connected(&self, u: usize, v: usize) -> bool {
        self.component[u] == self.component[v]
    }
}
```

## 8. Applications

1. **Network reliability:** Analyze connectivity under failures
2. **Social networks:** Track community splits
3. **Road networks:** Handle road closures
4. **Game AI:** Dynamic obstacle handling
5. **Incremental SAT:** Clause deletion

## 9. Related Problems

| Problem | Best Known |
|---------|------------|
| Incremental connectivity | O(α(n)) per op (DSU) |
| Decremental connectivity | O(log² n) per op |
| Fully dynamic connectivity | O(log² n) per op |
| Offline connectivity | O(m α(n)) total |

## 10. Edge Cases

| Case | Handling |
|------|----------|
| Delete non-existent edge | No change |
| Self-loops | Typically not allowed |
| Multi-edges | Track edge count |
| Isolated vertices | Own component |

## 11. Common Pitfalls

1. **Edge tracking:** Must know which edges exist
2. **Tree vs non-tree edges:** Only tree edge deletions may split
3. **Component reassignment:** Update all affected vertices
4. **Query after delete:** Ensure consistency

## 12. References

- Even, S.; Shiloach, Y. (1981). "An On-Line Edge-Deletion Problem"
- Henzinger, M. R.; King, V. (1999). "Randomized Fully Dynamic Graph Algorithms"
- Holm, J.; de Lichtenberg, K.; Thorup, M. (2001). "Poly-logarithmic deterministic fully-dynamic algorithms"
