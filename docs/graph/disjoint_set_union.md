# Disjoint Set Union (Union-Find)

## 1. Overview

Disjoint Set Union (DSU), also known as Union-Find, is a data structure that tracks a partition of elements into disjoint sets. It supports two primary operations: **Union** (merge two sets) and **Find** (determine which set an element belongs to). With path compression and union by rank/size, both operations achieve nearly constant amortized time.

## 2. Mathematical Foundation

### 2.1 Partition

A partition of set $S$ is a collection of non-empty, mutually disjoint subsets whose union is $S$.

### 2.2 Key Operations

- **MakeSet(x):** Create singleton set {x}
- **Find(x):** Return representative (root) of set containing x
- **Union(x, y):** Merge sets containing x and y

### 2.3 Inverse Ackermann Function

With both optimizations, time complexity is $O(\alpha(n))$ per operation, where $\alpha$ is the inverse Ackermann function. For all practical purposes, $\alpha(n) \leq 4$.

## 3. Optimizations

### 3.1 Path Compression

During Find, make every node on the path point directly to the root.

```
Before: 1 → 2 → 3 → 4 (root)
After:  1 → 4
        2 → 4
        3 → 4
```

### 3.2 Union by Rank/Size

Always attach smaller tree under root of larger tree.

- **By Rank:** Rank approximates tree height
- **By Size:** Size is actual number of elements

## 4. Algorithm

### 4.1 Pseudocode

```
MAKE-SET(x):
    parent[x] = x
    rank[x] = 0

FIND(x):
    if parent[x] ≠ x:
        parent[x] = FIND(parent[x])  // Path compression
    return parent[x]

UNION(x, y):
    root_x = FIND(x)
    root_y = FIND(y)
    
    if root_x = root_y:
        return  // Already in same set
    
    // Union by rank
    if rank[root_x] < rank[root_y]:
        parent[root_x] = root_y
    else if rank[root_x] > rank[root_y]:
        parent[root_y] = root_x
    else:
        parent[root_y] = root_x
        rank[root_x] += 1
```

## 5. Example

```
Initial: {0}, {1}, {2}, {3}, {4}

Union(0, 1):
    0       2   3   4
    |
    1

Union(2, 3):
    0       2       4
    |       |
    1       3

Union(0, 2):
        0           4
       /|
      1 2
        |
        3

Union(3, 4):
          0
       /  |  \
      1   2   4
          |
          3
```

## 6. Complexity

| Operation | Time (with optimizations) |
|-----------|---------------------------|
| MakeSet | O(1) |
| Find | O(α(n)) amortized |
| Union | O(α(n)) amortized |
| m operations | O(m · α(n)) |

Where $\alpha(n) \leq 4$ for all practical n.

## 7. Implementation

```rust
pub struct DisjointSetUnion {
    parent: Vec<usize>,
    rank: Vec<usize>,
    size: Vec<usize>,
}

impl DisjointSetUnion {
    pub fn new(n: usize) -> Self {
        DisjointSetUnion {
            parent: (0..n).collect(),
            rank: vec![0; n],
            size: vec![1; n],
        }
    }

    /// Find with path compression
    pub fn find(&mut self, x: usize) -> usize {
        if self.parent[x] != x {
            self.parent[x] = self.find(self.parent[x]);
        }
        self.parent[x]
    }

    /// Union by rank
    pub fn union(&mut self, x: usize, y: usize) -> bool {
        let root_x = self.find(x);
        let root_y = self.find(y);

        if root_x == root_y {
            return false;  // Already in same set
        }

        // Union by rank
        match self.rank[root_x].cmp(&self.rank[root_y]) {
            std::cmp::Ordering::Less => {
                self.parent[root_x] = root_y;
                self.size[root_y] += self.size[root_x];
            }
            std::cmp::Ordering::Greater => {
                self.parent[root_y] = root_x;
                self.size[root_x] += self.size[root_y];
            }
            std::cmp::Ordering::Equal => {
                self.parent[root_y] = root_x;
                self.size[root_x] += self.size[root_y];
                self.rank[root_x] += 1;
            }
        }
        true
    }

    /// Check if two elements are in the same set
    pub fn connected(&mut self, x: usize, y: usize) -> bool {
        self.find(x) == self.find(y)
    }

    /// Get size of set containing x
    pub fn set_size(&mut self, x: usize) -> usize {
        let root = self.find(x);
        self.size[root]
    }

    /// Count number of disjoint sets
    pub fn count_sets(&mut self) -> usize {
        (0..self.parent.len())
            .filter(|&x| self.find(x) == x)
            .count()
    }
}
```

## 8. Immutable Find (for &self)

```rust
impl DisjointSetUnion {
    /// Find without path compression (for &self)
    pub fn find_immutable(&self, mut x: usize) -> usize {
        while self.parent[x] != x {
            x = self.parent[x];
        }
        x
    }
}
```

## 9. Applications

### 9.1 Graph Algorithms
- **Kruskal's MST:** Detect cycles during edge addition
- **Connected components:** Track connectivity
- **Cycle detection:** Check if adding edge creates cycle

### 9.2 Other Applications
- **Image processing:** Connected component labeling
- **Network connectivity:** Dynamic connectivity queries
- **Percolation:** Physical simulation
- **Least Common Ancestor:** Offline LCA queries
- **Equivalence classes:** Group equivalent items

## 10. Kruskal's MST with DSU

```rust
pub fn kruskal_mst(n: usize, mut edges: Vec<(usize, usize, i64)>) -> (i64, Vec<(usize, usize)>) {
    // Sort edges by weight
    edges.sort_by_key(|e| e.2);
    
    let mut dsu = DisjointSetUnion::new(n);
    let mut mst_weight = 0;
    let mut mst_edges = vec![];
    
    for (u, v, w) in edges {
        if dsu.union(u, v) {
            mst_weight += w;
            mst_edges.push((u, v));
            
            if mst_edges.len() == n - 1 {
                break;
            }
        }
    }
    
    (mst_weight, mst_edges)
}
```

## 11. Variants

### 11.1 Weighted DSU

Track additional information (e.g., distance to root):

```rust
pub struct WeightedDSU {
    parent: Vec<usize>,
    rank: Vec<usize>,
    diff: Vec<i64>,  // diff[x] = value[x] - value[parent[x]]
}
```

### 11.2 DSU with Rollback

For problems requiring undo:

```rust
pub struct RollbackDSU {
    parent: Vec<usize>,
    rank: Vec<usize>,
    history: Vec<(usize, usize, usize)>,  // (node, old_parent, old_rank)
}
```

### 11.3 Persistent DSU

Use persistent arrays for versioning.

## 12. Edge Cases

| Case | Handling |
|------|----------|
| Self-union | union(x, x) returns false |
| Single element | Forms singleton set |
| All connected | Single set with n elements |
| No unions | n separate sets |

## 13. Common Pitfalls

1. **Forgetting path compression:** Leads to O(n) find
2. **Union without finding roots:** Must find roots first
3. **0 vs 1 indexing:** Be consistent
4. **Mutability:** Find with compression needs &mut self

## 14. References

- Tarjan, R. E. (1975). "Efficiency of a Good But Not Linear Set Union Algorithm"
- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 21
- Fredman, M. L.; Saks, M. E. (1989). "The cell probe complexity of dynamic data structures"
