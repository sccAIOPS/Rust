# Depth-First Search (DFS)

## 1. Overview

Depth-First Search (DFS) is a fundamental graph traversal algorithm that explores as far as possible along each branch before backtracking. It was first investigated by French mathematician Charles Pierre Trémaux as a strategy for solving mazes in the 19th century.

DFS forms the basis for many graph algorithms including topological sorting, cycle detection, and strongly connected component algorithms.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a graph $G = (V, E)$ and a source vertex $s \in V$, DFS explores vertices by going as deep as possible before backtracking.

**Input:**
- A graph $G = (V, E)$ (directed or undirected)
- A source vertex $s \in V$
- A target vertex $t \in V$ (optional)

**Output:**
- Whether a path exists from $s$ to $t$
- The traversal order of vertices (discovery order)

### 2.2 Mathematical Model

DFS produces a **depth-first forest** composed of **depth-first trees**. For each vertex $v$, DFS computes:
- **Discovery time** $d[v]$: When $v$ is first discovered
- **Finish time** $f[v]$: When exploration of $v$ is complete

**Edge Classification:**
- **Tree edges**: Edges in the DFS tree
- **Back edges**: $(u, v)$ where $v$ is an ancestor of $u$
- **Forward edges**: $(u, v)$ where $v$ is a descendant of $u$
- **Cross edges**: All other edges

### 2.3 Key Properties

**Parenthesis Theorem:** For any two vertices $u$ and $v$, exactly one of the following holds:
1. $[d[u], f[u]]$ and $[d[v], f[v]]$ are entirely disjoint
2. $[d[u], f[u]]$ is entirely contained within $[d[v], f[v]]$
3. $[d[v], f[v]]$ is entirely contained within $[d[u], f[u]]$

**White-Path Theorem:** Vertex $v$ is a descendant of $u$ in a DFS tree iff at time $d[u]$, $v$ is reachable from $u$ via a path of white (unvisited) vertices.

## 3. Algorithm Description

### 3.1 Intuition

Imagine exploring a maze by always taking the leftmost unexplored path until you hit a dead end, then backtracking to try the next path. This is exactly how DFS works:
1. Start at the source vertex
2. Pick an unvisited neighbor and recursively explore it
3. When no unvisited neighbors remain, backtrack
4. Continue until target is found or all reachable vertices are visited

### 3.2 Pseudocode

**Iterative Version (implemented in Rust):**
```
DFS(G, source, target):
    visited ← empty set
    history ← empty list
    stack ← empty stack
    
    stack.push(source)
    
    while stack is not empty:
        current ← stack.pop()
        history.append(current)
        
        if current == target:
            return history
        
        for each neighbor in G.neighbors(current).reverse():
            if neighbor not in visited:
                visited.add(neighbor)
                stack.push_front(neighbor)  // Add to front for DFS behavior
    
    return None
```

**Recursive Version:**
```
DFS-Recursive(G, v, visited, history, target):
    visited.add(v)
    history.append(v)
    
    if v == target:
        return true
    
    for each neighbor in G.neighbors(v):
        if neighbor not in visited:
            if DFS-Recursive(G, neighbor, visited, history, target):
                return true
    
    return false
```

### 3.3 Step-by-Step Example

Consider this graph:
```
    1 --- 2
    |     |
    3 --- 4 --- 5
```

DFS from vertex 1 to vertex 5:

| Step | Stack | Visited | Current | Action |
|------|-------|---------|---------|--------|
| 0 | [1] | {} | - | Initialize |
| 1 | [2, 3] | {1} | 1 | Process 1, add neighbors |
| 2 | [4, 3] | {1, 2} | 2 | Process 2, add 4 (3 via front) |
| 3 | [5, 3] | {1, 2, 4} | 4 | Process 4, add 5 (3 already queued) |
| 4 | [3] | {1, 2, 4, 5} | 5 | **Found target!** |

**Result:** Path = [1, 2, 4, 5]

Note: The implementation uses `push_front` to get DFS behavior with a deque.

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| Best | $O(1)$ | Target is the source |
| Average | $O(V + E)$ | Each vertex and edge visited once |
| Worst | $O(V + E)$ | Target not found, full traversal |

**Derivation:**
- Each vertex is visited at most once: $O(V)$
- Each edge is examined at most once (for directed) or twice (for undirected): $O(E)$
- Total: $O(V + E)$

### 4.2 Space Complexity

| Component | Space | Notes |
|-----------|-------|-------|
| Visited set | $O(V)$ | Tracks visited vertices |
| Stack/Recursion | $O(V)$ | Worst case: linear graph |
| History | $O(V)$ | Stores traversal order |
| **Total** | $O(V)$ | |

**Warning:** Recursive DFS can cause stack overflow for deep graphs. Use iterative version for graphs with depth > ~10,000.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::HashSet;
use std::collections::VecDeque;

pub fn depth_first_search(graph: &Graph, root: Vertex, objective: Vertex) -> Option<Vec<u32>> {
    let mut visited: HashSet<Vertex> = HashSet::new();
    let mut history: Vec<u32> = Vec::new();
    let mut queue = VecDeque::new();
    queue.push_back(root);

    while let Some(current_vertex) = queue.pop_front() {
        history.push(current_vertex.value());

        if current_vertex == objective {
            return Some(history);
        }

        // Reverse iteration + push_front gives DFS order
        for neighbor in current_vertex.neighbors(graph).into_iter().rev() {
            if visited.insert(neighbor) {
                queue.push_front(neighbor);
            }
        }
    }
    None
}
```

**Implementation Details:**
- Uses `VecDeque` with `push_front` to simulate a stack
- Reverses neighbor iteration to maintain expected DFS order
- `HashSet::insert()` returns `false` for duplicates, enabling atomic check-and-insert

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty graph | Returns `None` |
| Source = Target | Returns immediately with `Some(vec![source])` |
| Disconnected graph | Returns `None` if target unreachable |
| Self-loops | Handled by visited check |
| Cycles | Handled by visited set |
| Very deep graphs | Use iterative version to avoid stack overflow |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Topological Sorting**
   - Build system dependency resolution
   - Task scheduling
   - Course prerequisite ordering

2. **Cycle Detection**
   - Detecting circular dependencies
   - Deadlock detection
   - Reference cycle detection in garbage collection

3. **Strongly Connected Components**
   - Web page classification
   - Social network analysis
   - Circuit analysis

4. **Maze Generation & Solving**
   - Procedural maze generation
   - Path finding in games

5. **Syntax Analysis**
   - Expression tree traversal
   - AST processing in compilers

6. **Puzzle Solving**
   - Sudoku solvers
   - Chess puzzles
   - Constraint satisfaction problems

### 6.2 Related Algorithms

| Algorithm | Relationship to DFS |
|-----------|---------------------|
| BFS | Level-by-level vs depth-first exploration |
| Topological Sort | Uses DFS finish times |
| Tarjan's SCC | DFS with lowlink values |
| Kosaraju's SCC | Two DFS passes |
| Articulation Points | DFS with discovery/low values |
| Bridges in Graph | DFS-based detection |

## 7. DFS vs BFS Comparison

| Aspect | DFS | BFS |
|--------|-----|-----|
| Data Structure | Stack (LIFO) | Queue (FIFO) |
| Path Type | Any path | Shortest path (unweighted) |
| Space | O(h) where h = height | O(w) where w = max width |
| Complete | Yes (finite graphs) | Yes |
| Optimal | No | Yes (unweighted) |
| Memory | Often lower | Often higher |
| Use Case | Connectivity, cycles | Shortest paths, levels |

## 8. Performance Optimizations

### 8.1 Potential Improvements

1. **Iterative vs Recursive**
   - Iterative avoids stack overflow
   - Recursive is cleaner for tree structures

2. **Early Termination**
   - Return immediately when target found (already implemented)

3. **Visited Array vs HashSet**
   - For dense integer vertices, use `Vec<bool>` for O(1) lookup with lower constant

4. **Neighbor Pre-sorting**
   - Pre-sort adjacency lists for deterministic traversal order

### 8.2 Common Pitfalls

| Issue | Impact | Solution |
|-------|--------|----------|
| Stack overflow | Crash on deep graphs | Use iterative version |
| Not handling cycles | Infinite loop | Always track visited vertices |
| Wrong neighbor order | Different path found | Document expected order |
| Modifying graph during traversal | Undefined behavior | Make copy or use immutable reference |

## 9. References

- Tarjan, R. E. (1972). "Depth-first search and linear graph algorithms"
- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS), Chapter 22
- Sedgewick, R. "Algorithms in C++", Part 5: Graph Algorithms
- Trémaux, C. P. (1882). "Algorithm for solving mazes"
