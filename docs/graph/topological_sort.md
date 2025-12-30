# Topological Sort

## 1. Overview

Topological sorting is a linear ordering of vertices in a Directed Acyclic Graph (DAG) such that for every directed edge $(u, v)$, vertex $u$ comes before $v$ in the ordering. It's fundamental for dependency resolution, task scheduling, and build systems.

This implementation uses **Kahn's Algorithm** (BFS-based approach).

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input:** A directed acyclic graph $G = (V, E)$

**Output:** A linear ordering of vertices such that if $(u, v) \in E$, then $u$ appears before $v$

**Existence:** A topological ordering exists if and only if the graph is acyclic (DAG).

### 2.2 Properties

- A DAG has at least one vertex with no incoming edges (source)
- A DAG has at least one vertex with no outgoing edges (sink)
- Multiple valid orderings may exist

## 3. Algorithm (Kahn's Algorithm)

### 3.1 Pseudocode

```
TOPOLOGICAL-SORT(G):
    // Calculate in-degrees
    in_degree[v] ← 0 for all v
    for each edge (u, v):
        in_degree[v] += 1
    
    // Initialize queue with sources
    queue ← vertices with in_degree = 0
    result ← []
    
    while queue not empty:
        u ← queue.dequeue()
        result.append(u)
        
        for each neighbor v of u:
            in_degree[v] -= 1
            if in_degree[v] = 0:
                queue.enqueue(v)
    
    if |result| ≠ |V|:
        return "Cycle detected"
    return result
```

### 3.2 Example

```
    1 → 2 → 3
    ↓       ↓
    4 → 5 → 6
```

**Execution:**
1. Initial in-degrees: {1:0, 2:1, 3:1, 4:1, 5:1, 6:2}
2. Queue: [1]
3. Process 1: result=[1], queue=[2,4]
4. Process 2: result=[1,2], queue=[4,3]
5. Process 4: result=[1,2,4], queue=[3,5]
6. Continue until: result=[1,2,4,3,5,6]

## 4. Complexity Analysis

| Aspect | Complexity |
|--------|------------|
| Time | $O(V + E)$ |
| Space | $O(V)$ |

## 5. Implementation

```rust
pub fn topological_sort<Node: Hash + Eq + Copy>(
    edges: &Vec<(Node, Node)>,
) -> Result<Vec<Node>, TopoligicalSortError> {
    let mut edges_by_source: HashMap<Node, Vec<Node>> = HashMap::default();
    let mut incoming_edges_count: HashMap<Node, usize> = HashMap::default();
    
    // Build graph and count in-degrees
    for (source, destination) in edges {
        incoming_edges_count.entry(*source).or_insert(0);
        edges_by_source.entry(*source).or_default().push(*destination);
        *incoming_edges_count.entry(*destination).or_insert(0) += 1;
    }

    // Find sources (in-degree = 0)
    let mut queue = VecDeque::default();
    for (node, count) in &incoming_edges_count {
        if *count == 0 {
            queue.push_back(*node);
        }
    }
    
    // Process queue
    let mut sorted = Vec::default();
    while let Some(node) = queue.pop_back() {
        sorted.push(node);
        incoming_edges_count.remove(&node);
        
        for neighbour in edges_by_source.get(&node).unwrap_or(&vec![]) {
            if let Some(count) = incoming_edges_count.get_mut(neighbour) {
                *count -= 1;
                if *count == 0 {
                    incoming_edges_count.remove(neighbour);
                    queue.push_front(*neighbour);
                }
            }
        }
    }
    
    if incoming_edges_count.is_empty() {
        Ok(sorted)
    } else {
        Err(TopoligicalSortError::CycleDetected)
    }
}
```

## 6. Applications

1. **Build Systems:** Compiling dependencies in order (Make, Maven)
2. **Package Managers:** Installing packages with dependencies
3. **Course Prerequisites:** Planning course sequences
4. **Task Scheduling:** Project planning, workflow execution
5. **Spreadsheet Recalculation:** Cell dependency order
6. **Symbol Resolution:** Compiler linking order

## 7. Related Algorithms

| Algorithm | Purpose |
|-----------|---------|
| DFS-based TopSort | Alternative using finish times |
| Cycle Detection | Check if topological sort exists |
| SCC (Tarjan/Kosaraju) | Condensation graph is a DAG |

## 8. Edge Cases

| Case | Result |
|------|--------|
| Empty graph | Empty list |
| Single vertex | [vertex] |
| Cycle present | Error |
| Multiple valid orderings | One valid ordering returned |

## 9. References

- Kahn, A. B. (1962). "Topological sorting of large networks"
- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 22
