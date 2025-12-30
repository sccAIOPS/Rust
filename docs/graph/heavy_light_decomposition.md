# Heavy-Light Decomposition (HLD)

## 1. Overview

Heavy-Light Decomposition (HLD) decomposes a tree into disjoint paths (chains), enabling efficient path and subtree queries. It reduces tree path queries to a logarithmic number of contiguous range queries, which can be handled by segment trees or other range query structures.

The technique was introduced by Sleator and Tarjan in 1983.

## 2. Mathematical Foundation

### 2.1 Key Definitions

- **Heavy Edge:** Edge to child with largest subtree
- **Light Edge:** All other edges
- **Heavy Chain:** Maximal path of heavy edges
- **Chain Head:** First node in a chain (closest to root)

### 2.2 Properties

1. Each node belongs to exactly one heavy chain
2. At most $O(\log n)$ light edges on any root-to-leaf path
3. Therefore, at most $O(\log n)$ chains on any path

### 2.3 Why O(log n) Chains?

When traversing a light edge to child, subtree size at least halves. Thus maximum $\log n$ light edges on any path.

## 3. Algorithm

### 3.1 Construction Steps

1. **DFS 1:** Compute subtree sizes, identify heavy children
2. **DFS 2:** Assign positions in linear array, grouping chains contiguously
3. **Build Data Structure:** Segment tree over linear array

### 3.2 Query Steps

1. While nodes are in different chains:
   - Query chain of deeper node from head to node
   - Move to parent of chain head
2. Query remaining path in same chain

### 3.3 Pseudocode

```
// DFS 1: Compute subtree sizes and heavy children
DFS1(v, parent):
    size[v] = 1
    heavy_child[v] = -1
    max_child_size = 0
    
    for each child c of v:
        if c ≠ parent:
            DFS1(c, v)
            size[v] += size[c]
            if size[c] > max_child_size:
                max_child_size = size[c]
                heavy_child[v] = c

// DFS 2: Assign positions
DFS2(v, parent, chain_head):
    pos[v] = current_pos++
    head[v] = chain_head
    
    if heavy_child[v] ≠ -1:
        DFS2(heavy_child[v], v, chain_head)  // Continue chain
    
    for each child c of v:
        if c ≠ parent and c ≠ heavy_child[v]:
            DFS2(c, v, c)  // Start new chain

// Path Query
QUERY_PATH(u, v):
    result = identity
    while head[u] ≠ head[v]:
        if depth[head[u]] < depth[head[v]]:
            swap(u, v)
        result = combine(result, query_segment(pos[head[u]], pos[u]))
        u = parent[head[u]]
    
    if depth[u] > depth[v]:
        swap(u, v)
    result = combine(result, query_segment(pos[u], pos[v]))
    return result
```

## 4. Example

```
Tree:
        1
       /|\
      2 3 4
     /|   |\
    5 6   7 8
   /|
  9 10

Subtree sizes: 1→10, 2→5, 3→1, 4→3, 5→3, 6→1, 7→1, 8→1, 9→1, 10→1

Heavy edges: 1→2, 2→5, 5→9 (largest subtrees)
             4→7 (tie, pick first)

Chains: [1,2,5,9], [6], [10], [3], [4,7], [8]

Linear array positions:
1→0, 2→1, 5→2, 9→3, 6→4, 10→5, 3→6, 4→7, 7→8, 8→9
```

## 5. Complexity

| Operation | Time |
|-----------|------|
| Preprocessing | O(n) |
| Path Query | O(log²n) with segment tree |
| Path Update | O(log²n) with segment tree |
| Subtree Query | O(log n) |

## 6. Implementation

```rust
pub struct HLD {
    n: usize,
    parent: Vec<usize>,
    depth: Vec<usize>,
    heavy: Vec<Option<usize>>,
    head: Vec<usize>,
    pos: Vec<usize>,
    // Segment tree for range queries
    tree: Vec<i64>,
}

impl HLD {
    pub fn new(adj: &[Vec<usize>], root: usize) -> Self {
        let n = adj.len();
        let mut hld = HLD {
            n,
            parent: vec![usize::MAX; n],
            depth: vec![0; n],
            heavy: vec![None; n],
            head: vec![0; n],
            pos: vec![0; n],
            tree: vec![0; 4 * n],
        };
        
        hld.dfs_size(adj, root, usize::MAX);
        hld.dfs_decompose(adj, root, usize::MAX, root, &mut 0);
        hld
    }

    fn dfs_size(&mut self, adj: &[Vec<usize>], v: usize, p: usize) -> usize {
        self.parent[v] = p;
        let mut size = 1;
        let mut max_child_size = 0;
        
        for &u in &adj[v] {
            if u != p {
                self.depth[u] = self.depth[v] + 1;
                let child_size = self.dfs_size(adj, u, v);
                size += child_size;
                
                if child_size > max_child_size {
                    max_child_size = child_size;
                    self.heavy[v] = Some(u);
                }
            }
        }
        size
    }

    fn dfs_decompose(
        &mut self, 
        adj: &[Vec<usize>], 
        v: usize, 
        p: usize, 
        h: usize,
        cur_pos: &mut usize,
    ) {
        self.head[v] = h;
        self.pos[v] = *cur_pos;
        *cur_pos += 1;
        
        // Visit heavy child first (continues chain)
        if let Some(heavy) = self.heavy[v] {
            self.dfs_decompose(adj, heavy, v, h, cur_pos);
        }
        
        // Visit light children (start new chains)
        for &u in &adj[v] {
            if u != p && Some(u) != self.heavy[v] {
                self.dfs_decompose(adj, u, v, u, cur_pos);
            }
        }
    }

    pub fn query_path(&self, mut u: usize, mut v: usize) -> i64 {
        let mut result = 0;
        
        while self.head[u] != self.head[v] {
            // Move deeper chain head up
            if self.depth[self.head[u]] < self.depth[self.head[v]] {
                std::mem::swap(&mut u, &mut v);
            }
            
            // Query from head[u] to u
            result += self.query_range(self.pos[self.head[u]], self.pos[u]);
            u = self.parent[self.head[u]];
        }
        
        // u and v in same chain
        if self.depth[u] > self.depth[v] {
            std::mem::swap(&mut u, &mut v);
        }
        result + self.query_range(self.pos[u], self.pos[v])
    }

    pub fn update_node(&mut self, v: usize, value: i64) {
        self.update_range(self.pos[v], self.pos[v], value);
    }

    // Segment tree operations (simplified)
    fn query_range(&self, l: usize, r: usize) -> i64 {
        // Implementation depends on query type
        self.tree[l..=r].iter().sum()
    }

    fn update_range(&mut self, pos: usize, _: usize, value: i64) {
        self.tree[pos] = value;
    }
}
```

## 7. Supported Operations

| Operation | Method |
|-----------|--------|
| Path sum/max/min | Query segment tree on path |
| Path update | Update segment tree on path |
| Subtree query | Query contiguous range [pos[v], pos[v]+size[v]-1] |
| LCA | Byproduct of path decomposition |

## 8. Applications

1. **Path queries:** Sum/max/min on tree paths
2. **Path updates:** Add value to all nodes on path
3. **Subtree queries:** Aggregate over subtree
4. **LCA queries:** Natural byproduct
5. **Tree edge queries:** Map edges to child nodes
6. **Dynamic connectivity:** With link-cut trees

## 9. Comparison with Other Techniques

| Technique | Path Query | Update | LCA | Space |
|-----------|------------|--------|-----|-------|
| HLD + Segment Tree | O(log²n) | O(log²n) | O(log n) | O(n) |
| Euler Tour + RMQ | O(log n) | O(log n) | O(1) | O(n) |
| Link-Cut Trees | O(log n) amort. | O(log n) amort. | O(log n) | O(n) |
| Centroid Decomp. | O(log n) | O(log n) | - | O(n log n) |

## 10. Edge Cases

| Case | Handling |
|------|----------|
| Single node | One chain of length 1 |
| Linear tree (path) | One heavy chain |
| Star graph | Root chain + n-1 single chains |
| Query same node | Return node value |

## 11. Common Pitfalls

1. **Parent of root:** Use sentinel value (usize::MAX)
2. **Chain ordering:** Heavy child must be visited first in DFS2
3. **Position indexing:** Ensure segment tree is properly indexed
4. **Edge vs Node:** For edge queries, map edge (u,v) to deeper node

## 12. References

- Sleator, D. D.; Tarjan, R. E. (1983). "A Data Structure for Dynamic Trees"
- Competitive programming resources on HLD
- CP-Algorithms: Heavy-Light Decomposition
