# Centroid Decomposition

## 1. Overview

Centroid Decomposition recursively decomposes a tree by repeatedly finding and removing centroids, creating a new tree of logarithmic height. This enables efficient divide-and-conquer algorithms for path queries, achieving O(n log n) preprocessing and O(log n) queries for many problems.

## 2. Mathematical Foundation

### 2.1 Centroid Definition

A **centroid** of a tree is a node whose removal splits the tree into subtrees, each with at most $n/2$ nodes.

### 2.2 Key Theorem

**Every tree has at least one centroid**, and it can be found in O(n) time.

### 2.3 Centroid Tree Properties

1. Height is O(log n) because each subtree has ≤ n/2 nodes
2. Every path in original tree passes through the LCA in centroid tree
3. Enables divide-and-conquer approach

## 3. Algorithm

### 3.1 Construction Steps

1. Find centroid of current tree
2. Remove centroid (mark as processed)
3. Recursively decompose each subtree
4. Record centroid as parent of subtree centroids

### 3.2 Pseudocode

```
BUILD-CENTROID-TREE(tree):
    return DECOMPOSE(tree, 0, -1)  // Start from node 0, no parent

DECOMPOSE(tree, root, centroid_parent):
    centroid = FIND-CENTROID(tree, root)
    centroid_tree_parent[centroid] = centroid_parent
    removed[centroid] = true
    
    for each neighbor u of centroid:
        if not removed[u]:
            DECOMPOSE(tree, u, centroid)
    
    return centroid

FIND-CENTROID(tree, root):
    n = count_nodes(tree, root)
    return find_centroid_dfs(tree, root, -1, n)

find_centroid_dfs(tree, v, parent, n):
    size[v] = 1
    is_centroid = true
    
    for each neighbor u of v:
        if u ≠ parent and not removed[u]:
            result = find_centroid_dfs(tree, u, v, n)
            if result ≠ -1:
                return result
            size[v] += size[u]
            if size[u] > n/2:
                is_centroid = false
    
    if n - size[v] > n/2:
        is_centroid = false
    
    if is_centroid:
        return v
    return -1
```

## 4. Example

```
Original tree:
      1
     /|\
    2 3 4
   /|   |\
  5 6   7 8
 /
9

Step 1: Find centroid (node 1 or 2, depends on tie-breaking)
        Assume centroid = 1, mark removed

Step 2: Decompose subtrees rooted at 2, 3, 4
        Centroid of {2,5,6,9} = 2 or 5
        Centroid of {3} = 3
        Centroid of {4,7,8} = 4

Centroid Tree (one possibility):
           1
         / | \
        2  3  4
       /\     |\
      5  6    7 8
      |
      9
```

## 5. Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Construction | O(n log n) | O(n) |
| Tree height | O(log n) | - |
| Path query | O(log n) × query_cost | - |

## 6. Implementation

```rust
pub struct CentroidDecomposition {
    n: usize,
    adj: Vec<Vec<usize>>,
    removed: Vec<bool>,
    subtree_size: Vec<usize>,
    centroid_parent: Vec<Option<usize>>,
    centroid_depth: Vec<usize>,
}

impl CentroidDecomposition {
    pub fn new(adj: Vec<Vec<usize>>) -> Self {
        let n = adj.len();
        let mut cd = CentroidDecomposition {
            n,
            adj,
            removed: vec![false; n],
            subtree_size: vec![0; n],
            centroid_parent: vec![None; n],
            centroid_depth: vec![0; n],
        };
        cd.decompose(0, None, 0);
        cd
    }

    fn get_subtree_size(&mut self, v: usize, parent: Option<usize>) -> usize {
        self.subtree_size[v] = 1;
        for i in 0..self.adj[v].len() {
            let u = self.adj[v][i];
            if !self.removed[u] && Some(u) != parent {
                self.subtree_size[v] += self.get_subtree_size(u, Some(v));
            }
        }
        self.subtree_size[v]
    }

    fn find_centroid(&self, v: usize, parent: Option<usize>, tree_size: usize) -> usize {
        for &u in &self.adj[v] {
            if !self.removed[u] && Some(u) != parent {
                if self.subtree_size[u] > tree_size / 2 {
                    return self.find_centroid(u, Some(v), tree_size);
                }
            }
        }
        v
    }

    fn decompose(&mut self, v: usize, parent: Option<usize>, depth: usize) -> usize {
        let tree_size = self.get_subtree_size(v, None);
        let centroid = self.find_centroid(v, None, tree_size);
        
        self.removed[centroid] = true;
        self.centroid_parent[centroid] = parent;
        self.centroid_depth[centroid] = depth;
        
        for i in 0..self.adj[centroid].len() {
            let u = self.adj[centroid][i];
            if !self.removed[u] {
                self.decompose(u, Some(centroid), depth + 1);
            }
        }
        
        centroid
    }

    /// Process all paths through a centroid (template for queries)
    pub fn process_paths_through(&self, centroid: usize) {
        // For each subtree of centroid:
        // 1. Compute distances/values from centroid to all nodes
        // 2. Combine with values from other subtrees
        // This is problem-specific
    }
}
```

## 7. Query Template: Distance Queries

```rust
impl CentroidDecomposition {
    /// Find all nodes at distance exactly k from a given node
    pub fn nodes_at_distance(&self, v: usize, k: usize, dist_from: &[Vec<usize>]) -> Vec<usize> {
        let mut result = vec![];
        let mut current = v;
        
        // Walk up centroid tree
        while let Some(c) = self.centroid_parent[current].or(if current == 0 { Some(current) } else { None }) {
            let dist_to_centroid = dist_from[c][v];
            
            if dist_to_centroid == k {
                result.push(c);
            } else if dist_to_centroid < k {
                // Check nodes at distance (k - dist_to_centroid) from centroid
                // in OTHER subtrees (not containing v)
            }
            
            if current == c { break; }
            current = c;
        }
        
        result
    }
}
```

## 8. Applications

1. **Distance queries:** Count pairs at distance k
2. **Path queries:** Sum/max on paths
3. **Point update queries:** Update node values
4. **Closest marked node:** Find nearest special node
5. **Tree coloring:** Color queries after updates
6. **Xor paths:** Count paths with specific xor

## 9. Problem Types

| Problem Type | Centroid Usage |
|--------------|----------------|
| Count paths of length k | For each centroid, count paths through it |
| Sum of path lengths | Combine distances from centroid to subtrees |
| Max path value | Maintain max values in subtrees |
| Dynamic updates | Update O(log n) ancestors in centroid tree |

## 10. Comparison with Other Techniques

| Technique | Path Query | Update | Construction |
|-----------|------------|--------|--------------|
| Centroid Decomp. | O(log n) | O(log n) | O(n log n) |
| Heavy-Light Decomp. | O(log²n) | O(log²n) | O(n) |
| Mo's Algorithm | O(n√n) | - | O(1) |
| Tree DP | O(n) per query | - | - |

## 11. Edge Cases

| Case | Handling |
|------|----------|
| Single node | Node is its own centroid |
| Linear tree (path) | Middle node is centroid |
| Star graph | Center is centroid |
| Disconnected | Apply to each component |

## 12. Common Pitfalls

1. **Forgetting to mark removed:** Must mark centroid as removed before recursing
2. **Incorrect size computation:** Recompute sizes for each centroid search
3. **Parent direction:** Centroid tree parent ≠ original tree parent
4. **Counting paths twice:** When counting paths through centroid, avoid double-counting

## 13. Implementation Tips

1. **Precompute distances:** Store dist[centroid][node] for O(1) distance queries
2. **Use DFS order:** Assign DFS timestamps for efficient subtree operations
3. **Batch updates:** Process updates in batches for efficiency

## 14. References

- Megiddo, N.; Cole, R. (1990). "Optimal partition algorithms"
- Centroid decomposition in competitive programming
- CP-Algorithms: Centroid Decomposition
