# Lowest Common Ancestor (LCA)

## 1. Overview

The Lowest Common Ancestor (LCA) of two nodes $u$ and $v$ in a rooted tree is the deepest node that is an ancestor of both $u$ and $v$. LCA has numerous applications in tree algorithms and is a fundamental operation in many tree-based data structures.

## 2. Mathematical Foundation

### 2.1 Definition

For a rooted tree $T$ with root $r$:
$$LCA(u, v) = \text{deepest node } w \text{ such that } w \text{ is ancestor of both } u \text{ and } v$$

### 2.2 Properties

1. $LCA(u, u) = u$
2. $LCA(u, v) = LCA(v, u)$ (symmetric)
3. If $u$ is ancestor of $v$, then $LCA(u, v) = u$
4. $LCA(u, v)$ lies on path from $u$ to root and path from $v$ to root

## 3. Methods Overview

| Method | Preprocessing | Query | Space |
|--------|--------------|-------|-------|
| Naive | O(1) | O(n) | O(n) |
| Binary Lifting | O(n log n) | O(log n) | O(n log n) |
| Euler Tour + RMQ | O(n) | O(1) | O(n) |
| Heavy-Light Decomposition | O(n) | O(log n) | O(n) |

## 4. Binary Lifting Method

### 4.1 Key Idea

Precompute $2^k$-th ancestors for each node. To find LCA:
1. Lift lower node to same depth
2. Binary search for LCA using jump pointers

### 4.2 Pseudocode

```
PREPROCESS(tree):
    // parent[v][k] = 2^k-th ancestor of v
    for each v:
        parent[v][0] = direct_parent(v)
        depth[v] = compute_depth(v)
    
    for k = 1 to LOG:
        for each v:
            if parent[v][k-1] exists:
                parent[v][k] = parent[parent[v][k-1]][k-1]

LCA(u, v):
    // Ensure u is deeper
    if depth[u] < depth[v]:
        swap(u, v)
    
    // Lift u to same depth as v
    diff = depth[u] - depth[v]
    for k = 0 to LOG:
        if (diff >> k) & 1:
            u = parent[u][k]
    
    if u = v:
        return u
    
    // Binary search for LCA
    for k = LOG downto 0:
        if parent[u][k] ≠ parent[v][k]:
            u = parent[u][k]
            v = parent[v][k]
    
    return parent[u][0]
```

## 5. Example

```
        1 (root)
       /|\
      2 3 4
     /|   |
    5 6   7
   /
  8

depth: 1→0, 2→1, 3→1, 4→1, 5→2, 6→2, 7→2, 8→3

Binary Lifting Table (parent[v][k]):
v | k=0 | k=1 | k=2
--|-----|-----|----
1 |  -  |  -  |  -
2 |  1  |  -  |  -
5 |  2  |  1  |  -
8 |  5  |  2  |  1

LCA(8, 6):
1. depth[8]=3, depth[6]=2, lift 8 by 1: 8→5
2. parent[5][0]=2 ≠ parent[6][0]=2? No
3. Answer: parent[5][0] = 2
```

## 6. Complexity

| Operation | Time |
|-----------|------|
| Preprocessing | O(n log n) |
| LCA Query | O(log n) |
| Space | O(n log n) |

## 7. Implementation

```rust
pub struct LCA {
    parent: Vec<Vec<usize>>,
    depth: Vec<usize>,
    log: usize,
}

impl LCA {
    pub fn new(adj: &[Vec<usize>], root: usize) -> Self {
        let n = adj.len();
        let log = (n as f64).log2().ceil() as usize + 1;
        
        let mut parent = vec![vec![usize::MAX; log]; n];
        let mut depth = vec![0; n];
        
        // DFS to compute depths and direct parents
        let mut stack = vec![(root, usize::MAX, 0)];
        while let Some((v, p, d)) = stack.pop() {
            parent[v][0] = p;
            depth[v] = d;
            
            for &u in &adj[v] {
                if u != p {
                    stack.push((u, v, d + 1));
                }
            }
        }
        
        // Binary lifting: compute 2^k ancestors
        for k in 1..log {
            for v in 0..n {
                if parent[v][k-1] != usize::MAX {
                    parent[v][k] = parent[parent[v][k-1]][k-1];
                }
            }
        }
        
        LCA { parent, depth, log }
    }

    fn lift(&self, mut v: usize, dist: usize) -> usize {
        for k in 0..self.log {
            if (dist >> k) & 1 == 1 {
                v = self.parent[v][k];
                if v == usize::MAX { break; }
            }
        }
        v
    }

    pub fn lca(&self, mut u: usize, mut v: usize) -> usize {
        // Ensure u is deeper
        if self.depth[u] < self.depth[v] {
            std::mem::swap(&mut u, &mut v);
        }
        
        // Lift u to same depth as v
        u = self.lift(u, self.depth[u] - self.depth[v]);
        
        if u == v {
            return u;
        }
        
        // Binary search for LCA
        for k in (0..self.log).rev() {
            if self.parent[u][k] != self.parent[v][k] {
                u = self.parent[u][k];
                v = self.parent[v][k];
            }
        }
        
        self.parent[u][0]
    }

    pub fn distance(&self, u: usize, v: usize) -> usize {
        let l = self.lca(u, v);
        self.depth[u] + self.depth[v] - 2 * self.depth[l]
    }
}
```

## 8. Applications

1. **Tree distances:** $dist(u, v) = depth[u] + depth[v] - 2 \times depth[LCA(u,v)]$
2. **Path queries:** Aggregate values on tree paths
3. **Subtree queries:** Check if $u$ is in subtree of $v$
4. **Tree DP:** Computing functions over paths
5. **Graph algorithms:** Used in HLD, centroid decomposition
6. **Tarjan's offline LCA:** Process all queries in one DFS

## 9. Euler Tour + RMQ Method

### 9.1 Idea

1. Do Euler tour recording depth at each visit
2. LCA(u,v) = node with minimum depth between first occurrences of u and v
3. Use Sparse Table for O(1) RMQ

```rust
// Euler tour produces: [1,2,5,8,5,2,6,2,1,3,1,4,7,4,1]
// depths:              [0,1,2,3,2,1,2,1,0,1,0,1,2,1,0]
// first[v] = first occurrence index of v
// LCA(8, 6) = node at min depth in tour[first[8]..=first[6]]
```

This achieves O(n) preprocessing and O(1) queries.

## 10. Edge Cases

| Case | Handling |
|------|----------|
| Same node | LCA(v, v) = v |
| Parent-child | LCA = parent |
| Root query | LCA(root, v) = root |
| Single node tree | LCA = that node |

## 11. Common Pitfalls

1. **Root handling:** Root has no parent (use sentinel)
2. **Log calculation:** Ensure enough bits for tree height
3. **0 vs 1 indexing:** Be consistent
4. **Uninitialized arrays:** Set parent[v][k] = MAX for non-existent ancestors

## 12. References

- Bender, M. A.; Farach-Colton, M. (2000). "The LCA Problem Revisited"
- Tarjan, R. E. (1979). "Applications of Path Compression on Balanced Trees"
- Binary lifting technique in competitive programming
