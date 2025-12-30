# Cycle Detection in Graphs

## 1. Overview

Cycle detection determines whether a graph contains a cycle—a path that starts and ends at the same vertex. The implementation provides both DFS and BFS-based approaches for both directed and undirected graphs.

## 2. Mathematical Foundation

### 2.1 Definitions

**Cycle in Undirected Graph:** A path $v_0, v_1, ..., v_k$ where $v_0 = v_k$ and $k \geq 3$.

**Cycle in Directed Graph:** A path $v_0, v_1, ..., v_k$ where $v_0 = v_k$ and $k \geq 1$.

### 2.2 Detection Methods

**Undirected Graphs:**
- DFS: Back edge to non-parent vertex indicates cycle
- BFS: Back edge to visited vertex indicates cycle

**Directed Graphs:**
- DFS: Back edge (to vertex in current recursion stack) indicates cycle
- BFS (Kahn's): If topological sort incomplete, cycle exists

## 3. Algorithm Descriptions

### 3.1 DFS for Undirected Graphs

```
DFS-CYCLE-UNDIRECTED(G, v, parent, visited):
    visited[v] ← true
    
    for each neighbor u of v:
        if u = parent:
            continue  // Ignore edge back to parent
        if visited[u]:
            return true  // Back edge found = cycle
        if DFS-CYCLE-UNDIRECTED(G, u, v, visited):
            return true
    
    return false
```

### 3.2 DFS for Directed Graphs

```
DFS-CYCLE-DIRECTED(G, v, visited, in_stack):
    visited[v] ← true
    in_stack[v] ← true
    
    for each neighbor u of v:
        if in_stack[u]:
            return true  // Back edge in current path
        if not visited[u] and DFS-CYCLE-DIRECTED(G, u, visited, in_stack):
            return true
    
    in_stack[v] ← false
    return false
```

### 3.3 BFS for Directed Graphs (Kahn's)

```
BFS-CYCLE-DIRECTED(G):
    in_degree ← compute in-degrees
    queue ← vertices with in_degree = 0
    count ← 0
    
    while queue not empty:
        v ← queue.dequeue()
        count += 1
        for each neighbor u of v:
            in_degree[u] -= 1
            if in_degree[u] = 0:
                queue.enqueue(u)
    
    return count ≠ |V|  // true if cycle exists
```

## 4. Complexity Analysis

| Algorithm | Time | Space |
|-----------|------|-------|
| DFS (Undirected) | O(V + E) | O(V) |
| DFS (Directed) | O(V + E) | O(V) |
| BFS/Kahn's | O(V + E) | O(V) |

## 5. Implementation

```rust
pub trait DetectCycle {
    fn detect_cycle_dfs(&self) -> bool;
    fn detect_cycle_bfs(&self) -> bool;
}

// Undirected graph DFS
fn undirected_graph_detect_cycle_dfs(
    graph: &UndirectedGraph,
    visited: &mut HashSet<&String>,
    parent: Option<&String>,
    u: &String,
) -> bool {
    visited.insert(u);
    for (v, _) in graph.adjacency_table().get(u).unwrap() {
        if matches!(parent, Some(p) if v == p) {
            continue;
        }
        if visited.contains(v) ||
           undirected_graph_detect_cycle_dfs(graph, visited, Some(u), v) {
            return true;
        }
    }
    false
}

// Directed graph DFS
fn directed_graph_detect_cycle_dfs(
    graph: &DirectedGraph,
    visited: &mut HashSet<&String>,
    in_stack: &mut HashSet<&String>,
    u: &String,
) -> bool {
    visited.insert(u);
    in_stack.insert(u);
    
    for (v, _) in graph.adjacency_table().get(u).unwrap() {
        if visited.contains(v) && in_stack.contains(v) {
            return true;
        }
        if !visited.contains(v) &&
           directed_graph_detect_cycle_dfs(graph, visited, in_stack, v) {
            return true;
        }
    }
    
    in_stack.remove(u);
    false
}
```

## 6. Applications

1. **Deadlock Detection:** Resource allocation graphs
2. **Dependency Checking:** Circular dependencies in build systems
3. **Course Prerequisites:** Detecting impossible schedules
4. **Type Checking:** Detecting recursive type definitions
5. **Database Schema:** Detecting circular foreign keys

## 7. Key Differences

| Aspect | Undirected | Directed |
|--------|------------|----------|
| Cycle condition | Back edge to non-parent | Back edge in recursion stack |
| Minimum cycle | 3 vertices | 1 vertex (self-loop) |
| Parent tracking | Required | Not required |
| Stack tracking | Not required | Required |

## 8. Edge Cases

| Case | Undirected | Directed |
|------|------------|----------|
| Self-loop | Depends on definition | Cycle |
| Two vertices, two edges | Cycle | Not necessarily |
| Disconnected components | Check all components | Check all components |

## 9. References

- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 22
- Sedgewick, R. "Algorithms in C++", Chapter 19
