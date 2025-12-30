# Breadth-First Search (BFS)

## 1. Overview

Breadth-First Search (BFS) is a fundamental graph traversal algorithm that explores vertices in layers, visiting all neighbors at the current depth before moving to vertices at the next depth level. It was invented by Konrad Zuse in 1945 and later reinvented by Edward F. Moore in 1959.

BFS is the foundation for many graph algorithms and is particularly useful for finding shortest paths in unweighted graphs.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a graph $G = (V, E)$ and a source vertex $s \in V$, BFS systematically explores the edges of $G$ to discover every vertex reachable from $s$.

**Input:**
- A graph $G = (V, E)$ (directed or undirected)
- A source vertex $s \in V$
- A target vertex $t \in V$ (optional)

**Output:**
- The shortest path from $s$ to $t$ (in terms of number of edges)
- Or the discovery order of all reachable vertices

### 2.2 Mathematical Model

BFS computes a **breadth-first tree** with the following properties:
- The path from $s$ to any vertex $v$ in the tree is the shortest path in $G$
- For any vertex $v$ reachable from $s$: $\delta(s, v) = \text{depth of } v \text{ in BFS tree}$

Where $\delta(s, v)$ is the shortest-path distance (minimum number of edges).

### 2.3 Correctness Proof

**Lemma (Shortest Path Property):** For any edge $(u, v) \in E$:
$$\delta(s, v) \leq \delta(s, u) + 1$$

**Theorem:** BFS correctly computes shortest paths.

**Proof:** By induction on the distance from $s$:
- **Base case:** $\delta(s, s) = 0$ ✓
- **Inductive step:** Assume BFS correctly computes distances for all vertices at distance $\leq k$. For any vertex $v$ at distance $k+1$, there exists a vertex $u$ at distance $k$ such that $(u, v) \in E$. Since BFS explores vertices in order of distance, $u$ is discovered before $v$, and $v$ is discovered when exploring $u$'s neighbors.

## 3. Algorithm Description

### 3.1 Intuition

Imagine dropping a stone in water—the ripples spread outward in concentric circles. BFS works similarly:
1. Start at the source vertex (the center)
2. Visit all immediate neighbors (first ring)
3. Visit all neighbors of neighbors (second ring)
4. Continue until the target is found or all vertices are visited

### 3.2 Pseudocode

```
BFS(G, source, target):
    visited ← empty set
    history ← empty list
    queue ← empty queue
    
    visited.add(source)
    queue.enqueue(source)
    
    while queue is not empty:
        current ← queue.dequeue()
        history.append(current)
        
        if current == target:
            return history  // Found target
        
        for each neighbor in G.neighbors(current):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.enqueue(neighbor)
    
    return None  // Target not reachable
```

### 3.3 Step-by-Step Example

Consider this graph:
```
    1 --- 2
    |     |
    3 --- 4 --- 5
```

BFS from vertex 1 to vertex 5:

| Step | Queue | Visited | Current | Action |
|------|-------|---------|---------|--------|
| 0 | [1] | {1} | - | Initialize |
| 1 | [2, 3] | {1, 2, 3} | 1 | Process 1, add neighbors |
| 2 | [3, 4] | {1, 2, 3, 4} | 2 | Process 2, add 4 |
| 3 | [4] | {1, 2, 3, 4} | 3 | Process 3, 4 already visited |
| 4 | [5] | {1, 2, 3, 4, 5} | 4 | Process 4, add 5 |
| 5 | [] | {1, 2, 3, 4, 5} | 5 | **Found target!** |

**Result:** Path history = [1, 2, 3, 4, 5], shortest path length = 2

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| Best | $O(1)$ | Target is the source |
| Average | $O(V + E)$ | Each vertex and edge visited once |
| Worst | $O(V + E)$ | Target not found, full traversal |

**Derivation:**
- Each vertex is enqueued and dequeued at most once: $O(V)$
- Each edge is examined at most twice (once from each endpoint): $O(E)$
- Total: $O(V + E)$

### 4.2 Space Complexity

| Component | Space | Notes |
|-----------|-------|-------|
| Visited set | $O(V)$ | Stores all visited vertices |
| Queue | $O(V)$ | At most $V$ vertices in queue |
| History | $O(V)$ | Stores traversal order |
| **Total** | $O(V)$ | |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::HashSet;
use std::collections::VecDeque;

pub fn breadth_first_search(graph: &Graph, root: Node, target: Node) -> Option<Vec<u32>> {
    let mut visited: HashSet<Node> = HashSet::new();
    let mut history: Vec<u32> = Vec::new();
    let mut queue = VecDeque::new();

    visited.insert(root);
    queue.push_back(root);
    
    while let Some(currentnode) = queue.pop_front() {
        history.push(currentnode.value());
        
        if currentnode == target {
            return Some(history);
        }
        
        for neighbor in currentnode.neighbors(graph) {
            if visited.insert(neighbor) {
                queue.push_back(neighbor);
            }
        }
    }
    None
}
```

**Key Rust Patterns:**
- `HashSet::insert()` returns `false` if element already exists—used for atomic check-and-insert
- `VecDeque` provides O(1) push/pop from both ends
- Pattern matching with `while let Some(...)` for clean queue processing

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty graph | Returns `None` |
| Source = Target | Returns `Some(vec![source])` |
| Disconnected graph | Returns `None` if target unreachable |
| Self-loops | Handled correctly (visited check) |
| Parallel edges | First discovered path used |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Social Networks**
   - Finding degrees of separation between users
   - Friend recommendation (friends of friends)
   - Detecting communities

2. **Web Crawlers**
   - Discovering linked pages level by level
   - Building site maps
   - Link analysis

3. **GPS Navigation**
   - Shortest path in unweighted road networks
   - Finding nearest facility (gas station, restaurant)

4. **Network Broadcasting**
   - Packet routing in networks
   - Minimum hop routing protocols

5. **Puzzle Solving**
   - Solving sliding puzzles
   - Finding minimum moves in games
   - State space exploration

### 6.2 Related Algorithms

| Algorithm | Difference from BFS |
|-----------|---------------------|
| DFS | Explores depth-first, not level-by-level |
| Dijkstra | Handles weighted edges |
| A* | Uses heuristic for guided search |
| Bidirectional BFS | Searches from both ends simultaneously |
| Lee Algorithm | BFS specialized for maze/grid problems |

## 7. Performance Optimizations

### 7.1 Potential Improvements

1. **Bidirectional BFS**: Search from both source and target, meeting in the middle
   - Reduces complexity from $O(b^d)$ to $O(b^{d/2})$ where $b$ is branching factor

2. **Early termination**: Return immediately when target is found (already implemented)

3. **Memory optimization**: Use bit vector instead of HashSet for dense integer vertices

### 7.2 Common Pitfalls

| Issue | Impact | Solution |
|-------|--------|----------|
| Not marking visited before enqueue | Duplicates in queue | Mark visited when adding to queue |
| Using Vec instead of VecDeque | O(n) dequeue operations | Use VecDeque or swap-remove |
| Storing full path in queue | Memory explosion | Store parent pointers separately |

## 8. References

- Moore, E. F. (1959). "The shortest path through a maze"
- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS), Chapter 22
- Sedgewick, R. "Algorithms in C++", Part 5: Graph Algorithms
