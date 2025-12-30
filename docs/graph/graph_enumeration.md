# Graph Enumeration

## 1. Overview

Graph enumeration involves systematically generating or counting graphs with specific properties. This includes enumerating all graphs on n vertices, all connected graphs, all trees, or graphs with specific structural constraints. Enumeration algorithms are fundamental in combinatorics, chemistry (molecular enumeration), and network analysis.

## 2. Mathematical Foundation

### 2.1 Counting Labeled Graphs

Number of labeled graphs on n vertices:
$$2^{\binom{n}{2}} = 2^{n(n-1)/2}$$

### 2.2 Counting Labeled Trees (Cayley's Formula)

Number of labeled trees on n vertices:
$$n^{n-2}$$

### 2.3 Counting Unlabeled Graphs

Uses Pólya enumeration theorem. No simple closed form exists.

| n | Graphs | Connected | Trees |
|---|--------|-----------|-------|
| 1 | 1 | 1 | 1 |
| 2 | 2 | 1 | 1 |
| 3 | 4 | 2 | 1 |
| 4 | 11 | 6 | 2 |
| 5 | 34 | 21 | 3 |

## 3. Enumeration Methods

### 3.1 Direct Enumeration

Generate all possible edge combinations:

```
ENUMERATE-LABELED-GRAPHS(n):
    num_edges = n*(n-1)/2
    for mask = 0 to 2^num_edges - 1:
        graph = create_graph(n, mask)
        yield graph
```

### 3.2 Recursive Generation

Generate graphs by adding edges one at a time:

```
ENUMERATE-RECURSIVE(current_graph, edge_list, index):
    yield current_graph
    
    for i = index to |edge_list| - 1:
        (u, v) = edge_list[i]
        add_edge(current_graph, u, v)
        ENUMERATE-RECURSIVE(current_graph, edge_list, i + 1)
        remove_edge(current_graph, u, v)
```

### 3.3 Orderly Generation (McKay's Algorithm)

Generate one representative per isomorphism class:
- Use canonical form to avoid duplicates
- Only extend graphs that are canonical

## 4. Example: Enumerating 3-Vertex Graphs

```
Vertices: {0, 1, 2}
Possible edges: (0,1), (0,2), (1,2)

All labeled graphs (2³ = 8):
1. Empty: {}
2. {(0,1)}
3. {(0,2)}
4. {(1,2)}
5. {(0,1), (0,2)}
6. {(0,1), (1,2)}
7. {(0,2), (1,2)}
8. {(0,1), (0,2), (1,2)} = K₃

Unlabeled (4 equivalence classes):
1. Empty
2. Single edge (graphs 2,3,4 are isomorphic)
3. Path P₃ (graphs 5,6,7 are isomorphic)
4. Triangle K₃
```

## 5. Complexity

| Task | Complexity |
|------|------------|
| Enumerate labeled graphs | O(2^(n²/2)) |
| Enumerate trees | O(n^(n-2)) |
| Check isomorphism | GI-complete |
| Canonical form (nauty) | Exponential worst case, fast in practice |

## 6. Implementation

```rust
/// Enumerate all labeled graphs on n vertices
pub fn enumerate_labeled_graphs(n: usize) -> Vec<Vec<(usize, usize)>> {
    let mut graphs = Vec::new();
    
    // Generate all possible edges
    let mut edges = Vec::new();
    for i in 0..n {
        for j in (i + 1)..n {
            edges.push((i, j));
        }
    }
    
    let num_edges = edges.len();
    
    // Iterate through all subsets of edges
    for mask in 0..(1 << num_edges) {
        let mut graph = Vec::new();
        for (i, &edge) in edges.iter().enumerate() {
            if (mask >> i) & 1 == 1 {
                graph.push(edge);
            }
        }
        graphs.push(graph);
    }
    
    graphs
}

/// Enumerate all labeled graphs with exactly k edges
pub fn enumerate_k_edge_graphs(n: usize, k: usize) -> Vec<Vec<(usize, usize)>> {
    let mut graphs = Vec::new();
    let mut edges = Vec::new();
    
    for i in 0..n {
        for j in (i + 1)..n {
            edges.push((i, j));
        }
    }
    
    fn combinations(
        edges: &[(usize, usize)],
        k: usize,
        start: usize,
        current: &mut Vec<(usize, usize)>,
        result: &mut Vec<Vec<(usize, usize)>>,
    ) {
        if current.len() == k {
            result.push(current.clone());
            return;
        }
        if start >= edges.len() || edges.len() - start < k - current.len() {
            return;
        }
        
        // Include edge at start
        current.push(edges[start]);
        combinations(edges, k, start + 1, current, result);
        current.pop();
        
        // Exclude edge at start
        combinations(edges, k, start + 1, current, result);
    }
    
    combinations(&edges, k, 0, &mut Vec::new(), &mut graphs);
    graphs
}

/// Enumerate connected graphs (filter enumeration)
pub fn enumerate_connected_graphs(n: usize) -> Vec<Vec<(usize, usize)>> {
    enumerate_labeled_graphs(n)
        .into_iter()
        .filter(|g| is_connected(n, g))
        .collect()
}

fn is_connected(n: usize, edges: &[(usize, usize)]) -> bool {
    if n <= 1 {
        return true;
    }
    
    let mut adj = vec![vec![]; n];
    for &(u, v) in edges {
        adj[u].push(v);
        adj[v].push(u);
    }
    
    let mut visited = vec![false; n];
    let mut stack = vec![0];
    let mut count = 0;
    
    while let Some(v) = stack.pop() {
        if visited[v] {
            continue;
        }
        visited[v] = true;
        count += 1;
        
        for &u in &adj[v] {
            if !visited[u] {
                stack.push(u);
            }
        }
    }
    
    count == n
}
```

## 7. Tree Enumeration

### 7.1 Using Prüfer Sequences

```rust
/// Enumerate all labeled trees on n vertices
pub fn enumerate_labeled_trees(n: usize) -> Vec<Vec<(usize, usize)>> {
    if n <= 1 {
        return vec![vec![]];
    }
    if n == 2 {
        return vec![vec![(0, 1)]];
    }
    
    let mut trees = Vec::new();
    let len = n - 2;
    
    // Generate all Prüfer sequences
    fn generate_sequences(
        n: usize,
        len: usize,
        current: &mut Vec<usize>,
        result: &mut Vec<Vec<usize>>,
    ) {
        if current.len() == len {
            result.push(current.clone());
            return;
        }
        for v in 0..n {
            current.push(v);
            generate_sequences(n, len, current, result);
            current.pop();
        }
    }
    
    let mut sequences = Vec::new();
    generate_sequences(n, len, &mut Vec::new(), &mut sequences);
    
    // Convert each sequence to tree
    for seq in sequences {
        let edges = prufer_to_tree(&seq, n);
        trees.push(edges);
    }
    
    trees
}

fn prufer_to_tree(sequence: &[usize], n: usize) -> Vec<(usize, usize)> {
    // Implementation from prufer_code.md
    // ... (see Prüfer Code documentation)
    vec![]  // Placeholder
}
```

## 8. Applications

1. **Molecular chemistry:** Enumerate isomers
2. **Network design:** Generate candidate topologies
3. **Algorithm testing:** Generate all inputs
4. **Combinatorics:** Counting problems
5. **Graph theory research:** Verify conjectures

## 9. Special Graph Classes

| Class | Count Formula (n vertices) |
|-------|---------------------------|
| Complete graphs | 1 |
| Paths | n!/2 (labeled) |
| Cycles | (n-1)!/2 (labeled, n≥3) |
| Trees | n^(n-2) (labeled) |
| Forests | Various formulas |
| Bipartite | Complex formula |

## 10. Optimization Techniques

### 10.1 Pruning

Skip invalid partial graphs early:
- If graph cannot become connected, prune
- If graph violates degree constraints, prune

### 10.2 Canonical Forms

Use graph canonical form to avoid isomorphic duplicates:
- Sort adjacency lists
- Use lexicographically smallest representation
- Apply nauty/bliss algorithms

### 10.3 Symmetry Breaking

Add constraints to eliminate symmetric solutions:
- First edge always includes vertex 0
- Edges in lexicographic order

## 11. Edge Cases

| Case | Handling |
|------|----------|
| n = 0 | Empty graph only |
| n = 1 | Single vertex, no edges |
| k > n(n-1)/2 | No such graph exists |
| Dense enumeration | May exhaust memory |

## 12. Common Pitfalls

1. **Exponential growth:** Enumeration is inherently expensive
2. **Memory limits:** Store graphs as needed, not all at once
3. **Duplicate handling:** Use canonical forms or careful generation
4. **Off-by-one:** Careful with n vs n-1 edge counts

## 13. References

- McKay, B. D. (1998). "Isomorph-Free Exhaustive Generation"
- Read, R. C.; Wilson, R. J. "An Atlas of Graphs"
- Pólya, G. (1937). "Kombinatorische Anzahlbestimmungen für Gruppen, Graphen und chemische Verbindungen"
