# Union-Find (Disjoint Set Union)

## 1. Overview

Union-Find (also called Disjoint Set Union or DSU) is a data structure that tracks elements partitioned into disjoint (non-overlapping) sets. It provides near-constant time operations for uniting sets and determining if elements are in the same set, making it ideal for connectivity problems and Kruskal's MST algorithm.

Developed through contributions by Bernard Galler, Michael Fischer, Robert Tarjan, and others from the 1960s-1980s.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Maintain a partition of elements $\{0, 1, ..., n-1\}$ into disjoint sets, supporting:
- **MakeSet(x)**: Create singleton set $\{x\}$
- **Find(x)**: Return representative of set containing $x$
- **Union(x, y)**: Merge sets containing $x$ and $y$

### 2.2 Mathematical Model

**Equivalence Relation**: Union-Find maintains an equivalence relation with properties:
- Reflexive: $x \sim x$
- Symmetric: $x \sim y \Rightarrow y \sim x$
- Transitive: $x \sim y \land y \sim z \Rightarrow x \sim z$

**Representation**: Forest of trees where each tree is a set, and the root is the representative.

**Complexity with Path Compression + Union by Rank**:
$$\alpha(n) \text{ per operation}$$

where $\alpha$ is the inverse Ackermann function, effectively $\leq 4$ for all practical $n$.

### 2.3 Inverse Ackermann Function

The Ackermann function $A(m, n)$ grows astronomically fast:
- $A(4, 2) = 2^{2^{2^{65536}}} - 3$

Its inverse $\alpha(n)$ grows incredibly slowly:
- $\alpha(n) \leq 4$ for $n < 10^{80}$ (more atoms than in observable universe)

## 3. Algorithm Description

### 3.1 Intuition

Think of each element as a person in a social network. Initially, everyone is their own group. When two people become friends (union), their entire groups merge. Finding if two people are connected (same set) means checking if they share the same group leader.

### 3.2 Tree Representation

```
Initial: MakeSet for 0,1,2,3,4
[0] [1] [2] [3] [4]  (each is its own parent)

Union(0,1):        Union(2,3):
  1                    3
  |                    |
  0                    2

Union(1,3):
    3
   /|\
  1 2
  |
  0

Find(0) → follows path to root → returns 3
```

### 3.3 Optimizations

**Path Compression** (in Find):
```
Before Find(0):        After Find(0):
    3                      3
    |                    / | \
    1                   0  1  2
    |
    0
```

**Union by Rank**: Always attach smaller tree under larger tree's root.

### 3.4 Pseudocode

```
MAKE_SET(x):
    parent[x] = x
    rank[x] = 0

FIND(x):
    if parent[x] != x:
        parent[x] = FIND(parent[x])  // Path compression
    return parent[x]

UNION(x, y):
    root_x = FIND(x)
    root_y = FIND(y)
    
    if root_x == root_y:
        return  // Already in same set
    
    // Union by rank
    if rank[root_x] < rank[root_y]:
        parent[root_x] = root_y
    else if rank[root_x] > rank[root_y]:
        parent[root_y] = root_x
    else:
        parent[root_y] = root_x
        rank[root_x]++
```

### 3.5 Step-by-Step Example

**Kruskal's MST setup**:

```
Elements: {A, B, C, D, E}
Initial: parent = [A, B, C, D, E], rank = [0, 0, 0, 0, 0]

Union(A, B):
  Find(A) = A, Find(B) = B
  rank[A] == rank[B], so parent[B] = A, rank[A]++
  parent = [A, A, C, D, E], rank = [1, 0, 0, 0, 0]
  Tree:  A
         |
         B

Union(C, D):
  parent[D] = C, rank[C]++
  parent = [A, A, C, C, E], rank = [1, 0, 1, 0, 0]
  Trees: A    C
         |    |
         B    D

Union(B, D):
  Find(B) = A, Find(D) = C
  rank[A] == rank[C], so parent[C] = A, rank[A]++
  parent = [A, A, A, C, E], rank = [2, 0, 1, 0, 0]
  Tree:    A
          /|\
         B C
           |
           D

Find(D) with path compression:
  Follow: D → C → A
  Compress: parent[D] = A, parent[C] = A
  parent = [A, A, A, A, E]
  Tree:    A
         /|\ \
        B C D (flattened!)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Without Optimization | With Both Optimizations |
|-----------|---------------------|------------------------|
| MakeSet   | O(1)                | O(1)                   |
| Find      | O(n)                | O(α(n)) ≈ O(1)         |
| Union     | O(n)                | O(α(n)) ≈ O(1)         |

**For m operations on n elements**: O(m · α(n)) ≈ O(m)

### 4.2 Space Complexity

- **Storage**: O(n) for parent and rank arrays
- **Auxiliary**: O(1) iterative, O(log n) recursive (without path compression)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct UnionFind {
    parent: Vec<usize>,
    rank: Vec<usize>,
    count: usize,  // Number of disjoint sets
}

impl UnionFind {
    pub fn new(n: usize) -> Self {
        UnionFind {
            parent: (0..n).collect(),
            rank: vec![0; n],
            count: n,
        }
    }
    
    pub fn find(&mut self, x: usize) -> usize {
        if self.parent[x] != x {
            self.parent[x] = self.find(self.parent[x]);  // Path compression
        }
        self.parent[x]
    }
    
    pub fn union(&mut self, x: usize, y: usize) -> bool {
        let root_x = self.find(x);
        let root_y = self.find(y);
        
        if root_x == root_y {
            return false;  // Already connected
        }
        
        // Union by rank
        match self.rank[root_x].cmp(&self.rank[root_y]) {
            std::cmp::Ordering::Less => self.parent[root_x] = root_y,
            std::cmp::Ordering::Greater => self.parent[root_y] = root_x,
            std::cmp::Ordering::Equal => {
                self.parent[root_y] = root_x;
                self.rank[root_x] += 1;
            }
        }
        self.count -= 1;
        true
    }
    
    pub fn connected(&mut self, x: usize, y: usize) -> bool {
        self.find(x) == self.find(y)
    }
}
```

**Key Design Patterns**:
- `Vec<usize>` for parent/rank arrays
- `&mut self` for find (due to path compression mutation)
- Return `bool` from union to indicate if merge happened
- Track `count` for number of components

### 5.2 Iterative Find (No Recursion)

```rust
pub fn find_iterative(&mut self, mut x: usize) -> usize {
    // Find root
    let mut root = x;
    while self.parent[root] != root {
        root = self.parent[root];
    }
    
    // Path compression
    while self.parent[x] != root {
        let next = self.parent[x];
        self.parent[x] = root;
        x = next;
    }
    
    root
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Union same element | Find returns same root, no-op |
| Single element | Is its own set |
| All elements same set | Tree of height O(log n) |
| Find after union | Path compression flattens |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Kruskal's MST**: Check if adding edge creates cycle
2. **Network Connectivity**: Are two computers connected?
3. **Image Processing**: Connected component labeling
4. **Social Networks**: Friend groups, mutual friends
5. **Percolation**: Does water flow from top to bottom?
6. **Equivalence Classes**: Type unification in compilers

### 6.2 Kruskal's Algorithm Integration

```rust
fn kruskal(n: usize, edges: &mut [(usize, usize, i32)]) -> Vec<(usize, usize, i32)> {
    edges.sort_by_key(|e| e.2);  // Sort by weight
    let mut uf = UnionFind::new(n);
    let mut mst = Vec::new();
    
    for &(u, v, w) in edges.iter() {
        if uf.union(u, v) {  // If u and v weren't connected
            mst.push((u, v, w));
            if mst.len() == n - 1 {
                break;
            }
        }
    }
    mst
}
```

### 6.3 Variants

| Variant | Modification |
|---------|-------------|
| **Weighted Union-Find** | Track component sizes |
| **Union-Find with Rollback** | Undo unions (for offline queries) |
| **Persistent Union-Find** | Immutable versions |
| **Link-Cut Trees** | Dynamic connectivity with O(log n) |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Forgetting Path Compression**: Degrades to O(n) per operation
2. **Wrong Union Direction**: Without rank, can create long chains
3. **Mutable Find**: Requires `&mut self` which can be awkward
4. **Index Out of Bounds**: Ensure x, y < n

### 7.2 Union by Size vs Rank

**Union by Rank**: Attach tree with smaller rank under larger
- Rank is upper bound on height
- Slightly simpler to implement

**Union by Size**: Attach smaller tree under larger
- Can track actual component sizes
- More useful metadata

```rust
// Union by size
pub fn union_by_size(&mut self, x: usize, y: usize) {
    let root_x = self.find(x);
    let root_y = self.find(y);
    
    if root_x != root_y {
        if self.size[root_x] < self.size[root_y] {
            self.parent[root_x] = root_y;
            self.size[root_y] += self.size[root_x];
        } else {
            self.parent[root_y] = root_x;
            self.size[root_x] += self.size[root_y];
        }
    }
}
```

### 7.3 Without Path Compression

If `find` cannot mutate (e.g., concurrent access):
- Use "path halving": `parent[x] = parent[parent[x]]` during find
- Still O(log n) per operation with union by rank

## 8. References

- Tarjan, R. E. (1975). "Efficiency of a Good But Not Linear Set Union Algorithm". *JACM*.
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 21.
- Sedgewick, R. & Wayne, K. (2011). *Algorithms* (4th ed.). Chapter 1.5.
- Galil, Z., & Italiano, G. F. (1991). "Data Structures and Algorithms for Disjoint Set Union Problems". *ACM Computing Surveys*.
