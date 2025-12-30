# Kosaraju's Strongly Connected Components Algorithm

## 1. Overview

Kosaraju's algorithm finds all Strongly Connected Components (SCCs) in a directed graph using two DFS passes. It was discovered by S. Rao Kosaraju in 1978 (unpublished) and later described by Micha Sharir in 1981.

The algorithm is conceptually simpler than Tarjan's but requires building the transpose graph.

## 2. Mathematical Foundation

### 2.1 Key Insight

If we reverse all edges in a graph, the SCCs remain the same (just with reversed internal edges).

**Theorem:** If we perform DFS on the transpose graph in reverse order of original finish times, each DFS tree corresponds to exactly one SCC.

## 3. Algorithm

### 3.1 Steps

1. **First DFS:** Traverse original graph, record vertices by finish time
2. **Build Transpose:** Reverse all edge directions
3. **Second DFS:** Process vertices in decreasing finish time order on transpose

### 3.2 Pseudocode

```
KOSARAJU(G):
    // First DFS: get finish times
    visited ← [false, ...]
    stack ← []
    for each vertex v:
        if not visited[v]:
            DFS1(v, visited, stack)
    
    // Build transpose graph
    G_T ← transpose(G)
    
    // Second DFS: find SCCs
    visited ← [false, ...]
    sccs ← []
    while stack not empty:
        v ← stack.pop()
        if not visited[v]:
            scc ← []
            DFS2(v, G_T, visited, scc)
            sccs.add(scc)
    
    return sccs

DFS1(v, visited, stack):
    visited[v] ← true
    for each neighbor u of v:
        if not visited[u]:
            DFS1(u, visited, stack)
    stack.push(v)  // Record finish time

DFS2(v, G_T, visited, scc):
    visited[v] ← true
    scc.add(v)
    for each neighbor u in G_T:
        if not visited[u]:
            DFS2(u, G_T, visited, scc)
```

## 4. Example

```
Original:           Transpose:
0 → 1               0 ← 1
↓   ↓               ↓   ↓
2 ← 3               2 → 3
```

**First DFS from 0:** Visit order: 0→1→3→2
**Stack (finish order):** [2, 3, 1, 0]

**Second DFS on transpose:**
- Pop 0: DFS finds {0}
- Pop 1: DFS finds {1, 3, 2}

**SCCs:** {0}, {1, 3, 2}

## 5. Complexity

| Metric | Complexity |
|--------|------------|
| Time | O(V + E) |
| Space | O(V + E) for transpose |

## 6. Implementation

```rust
pub struct Graph {
    vertices: usize,
    adj_list: Vec<Vec<usize>>,
    transpose_adj_list: Vec<Vec<usize>>,
}

impl Graph {
    pub fn add_edge(&mut self, u: usize, v: usize) {
        self.adj_list[u].push(v);
        self.transpose_adj_list[v].push(u);  // Build transpose simultaneously
    }

    pub fn dfs(&self, node: usize, visited: &mut Vec<bool>, stack: &mut Vec<usize>) {
        visited[node] = true;
        for &neighbor in &self.adj_list[node] {
            if !visited[neighbor] {
                self.dfs(neighbor, visited, stack);
            }
        }
        stack.push(node);  // Push after exploring all neighbors
    }

    pub fn dfs_scc(&self, node: usize, visited: &mut Vec<bool>, scc: &mut Vec<usize>) {
        visited[node] = true;
        scc.push(node);
        for &neighbor in &self.transpose_adj_list[node] {
            if !visited[neighbor] {
                self.dfs_scc(neighbor, visited, scc);
            }
        }
    }
}

pub fn kosaraju(graph: &Graph) -> Vec<Vec<usize>> {
    let mut visited = vec![false; graph.vertices];
    let mut stack = Vec::new();

    // First DFS pass
    for i in 0..graph.vertices {
        if !visited[i] {
            graph.dfs(i, &mut visited, &mut stack);
        }
    }

    // Second DFS pass on transpose
    let mut sccs = Vec::new();
    visited = vec![false; graph.vertices];

    while let Some(node) = stack.pop() {
        if !visited[node] {
            let mut scc = Vec::new();
            graph.dfs_scc(node, &mut visited, &mut scc);
            sccs.push(scc);
        }
    }

    sccs
}
```

## 7. Why It Works

1. **Finish time ordering:** Vertices with later finish times can reach vertices with earlier times
2. **Transpose reversal:** In transpose, if SCC A can reach SCC B originally, now B can reach A
3. **Processing order:** By processing in reverse finish time on transpose, we can't escape an SCC before fully exploring it

## 8. Applications

Same as Tarjan's algorithm:
- 2-SAT solving
- Social network analysis
- Compiler optimization
- Web page clustering
- Deadlock detection

## 9. Kosaraju's vs Tarjan's

| Aspect | Kosaraju's | Tarjan's |
|--------|------------|----------|
| Passes | 2 | 1 |
| Transpose graph | Required | Not required |
| Space | O(V + E) extra | O(V) extra |
| Conceptual complexity | Simpler | More complex |
| Output order | Topological | Reverse topological |

## 10. Edge Cases

| Case | Behavior |
|------|----------|
| Empty graph | Empty list of SCCs |
| Single vertex | One SCC per vertex |
| Self-loop | Single-vertex SCC |
| DAG | Each vertex is its own SCC |
| Fully connected | Single SCC |

## 11. References

- Sharir, M. (1981). "A strong-connectivity algorithm and its applications"
- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 22
