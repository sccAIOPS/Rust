# Segment Tree

## 1. Overview

A Segment Tree is a data structure that allows efficient range queries and point/range updates on an array. It's a full binary tree where each node stores aggregate information (sum, min, max, GCD, etc.) about a segment of the array, enabling O(log n) operations for both queries and updates.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, support:
- **Build**: Construct the tree from array
- **Query(l, r)**: Compute aggregate over $A[l..r]$
- **Update(i, v)**: Set $A[i] = v$ (point update)
- **Update(l, r, v)**: Modify range (range update with lazy propagation)

### 2.2 Mathematical Model

**Tree Structure**:
- Node at index $i$ has children at $2i+1$ and $2i+2$ (0-indexed)
- Leaf nodes represent individual array elements
- Internal nodes represent merged information from children

**For Range $[l, r]$ at Node**:
$$\text{node.value} = f(A[l], A[l+1], ..., A[r])$$

where $f$ is an associative operation (sum, min, max, AND, OR, GCD, etc.).

**Tree Size**: For $n$ elements, need at most $4n$ nodes.

### 2.3 Key Properties

1. **Complete Coverage**: Every array element appears in exactly one leaf
2. **Hierarchical Merging**: Parent = merge(left_child, right_child)
3. **Decomposition**: Any query range can be decomposed into O(log n) tree nodes

## 3. Algorithm Description

### 3.1 Intuition

Imagine organizing a tournament bracket where each match (node) records the combined score of its subtournament. To find the total score for games 3-7, you don't sum all games individually—you find the minimal set of completed subtournaments that cover that range.

### 3.2 Structure Visualization

```
Array: [1, 3, 5, 7, 9, 11]
Segment Tree for SUM:

                [36]               <- sum of [0..5]
              /      \
          [9]          [27]        <- [0..2], [3..5]
         /   \        /    \
       [4]   [5]   [16]    [11]    <- [0..1], [2], [3..4], [5]
       / \         /  \
     [1] [3]     [7]  [9]          <- [0], [1], [3], [4]

Query(1, 4): sum of A[1..4] = 3 + 5 + 7 + 9 = 24
Decomposition: [3] + [5] + [16] = 24
```

### 3.3 Pseudocode

```
BUILD(arr, tree, node, start, end):
    if start == end:
        tree[node] = arr[start]
    else:
        mid = (start + end) / 2
        BUILD(arr, tree, 2*node+1, start, mid)
        BUILD(arr, tree, 2*node+2, mid+1, end)
        tree[node] = merge(tree[2*node+1], tree[2*node+2])

QUERY(tree, node, start, end, l, r):
    if r < start or end < l:
        return IDENTITY  // No overlap
    if l <= start and end <= r:
        return tree[node]  // Total overlap
    mid = (start + end) / 2
    left_val = QUERY(tree, 2*node+1, start, mid, l, r)
    right_val = QUERY(tree, 2*node+2, mid+1, end, l, r)
    return merge(left_val, right_val)

UPDATE(tree, node, start, end, idx, val):
    if start == end:
        tree[node] = val
    else:
        mid = (start + end) / 2
        if idx <= mid:
            UPDATE(tree, 2*node+1, start, mid, idx, val)
        else:
            UPDATE(tree, 2*node+2, mid+1, end, idx, val)
        tree[node] = merge(tree[2*node+1], tree[2*node+2])
```

### 3.4 Step-by-Step Example

**Query(2, 5) on the tree above**:

```
Start at root [36] covering [0..5]
Query range [2..5]

1. Root [0..5]: partial overlap with [2..5]
   - Go left to [0..2]: partial overlap
     - Go left to [0..1]: no overlap → return 0
     - Go right to [2]: total overlap → return 5
     - Return: 0 + 5 = 5
   - Go right to [3..5]: total overlap → return 27
   - Return: 5 + 27 = 32

Result: A[2] + A[3] + A[4] + A[5] = 5 + 7 + 9 + 11 = 32 ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time | Notes |
|-----------|------|-------|
| Build | O(n) | Visit each node once |
| Point Query | O(log n) | Traverse root to leaf |
| Range Query | O(log n) | At most 4 nodes per level |
| Point Update | O(log n) | Traverse root to leaf |
| Range Update | O(log n) | With lazy propagation |

### 4.2 Space Complexity

- **Storage**: O(n) to O(4n) depending on implementation
- **Exact**: For $n$ leaves, need $2n - 1$ nodes (but $4n$ for easy indexing)
- **Recursion**: O(log n) stack space

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct SegmentTree<T, F>
where
    F: Fn(&T, &T) -> T,
{
    size: usize,
    tree: Vec<T>,
    merge: F,
    identity: T,
}

impl<T: Clone, F: Fn(&T, &T) -> T> SegmentTree<T, F> {
    pub fn from_vec(arr: Vec<T>, identity: T, merge: F) -> Self {
        let size = arr.len();
        let mut tree = vec![identity.clone(); 4 * size];
        
        if size > 0 {
            Self::build(&arr, &mut tree, &merge, 0, 0, size - 1);
        }
        
        SegmentTree { size, tree, merge, identity }
    }
    
    pub fn query(&self, l: usize, r: usize) -> T {
        self.query_recursive(0, 0, self.size - 1, l, r)
    }
    
    pub fn update(&mut self, idx: usize, val: T) {
        self.update_recursive(0, 0, self.size - 1, idx, val);
    }
}
```

**Key Design Patterns**:
- Generic over element type `T` and merge function `F`
- Closure-based merge for flexibility
- Identity element for empty ranges
- `4 * size` allocation for safe indexing

### 5.2 Iterative Implementation

```rust
// Bottom-up segment tree (more cache-friendly)
pub struct IterativeSegTree<T> {
    n: usize,
    tree: Vec<T>,
}

impl<T: Clone + Default> IterativeSegTree<T>
where
    T: std::ops::Add<Output = T>,
{
    pub fn query(&self, mut l: usize, mut r: usize) -> T {
        l += self.n;
        r += self.n;
        let mut result = T::default();
        
        while l <= r {
            if l % 2 == 1 {
                result = result + self.tree[l].clone();
                l += 1;
            }
            if r % 2 == 0 {
                result = result + self.tree[r].clone();
                r -= 1;
            }
            l /= 2;
            r /= 2;
        }
        result
    }
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Return identity |
| Single element | Leaf is root |
| Query out of bounds | Validate or return identity |
| l > r | Return identity |
| Update out of bounds | Validate or no-op |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Range Sum Queries**: Database aggregations
2. **Range Minimum/Maximum**: Stock price analysis
3. **Interval Scheduling**: Calendar overlap detection
4. **Computational Geometry**: Line sweep algorithms
5. **String Matching**: LCP queries with suffix arrays
6. **Graphics**: Visibility determination

### 6.2 Variants

| Variant | Description | Use Case |
|---------|-------------|----------|
| **Lazy Propagation** | Defer range updates | Range add/set |
| **Persistent** | Keep history of versions | Undo, time travel |
| **2D Segment Tree** | Tree of trees | Matrix range queries |
| **Dynamic** | Implicit nodes | Large/sparse ranges |
| **Merge Sort Tree** | Store sorted lists | Kth smallest in range |

### 6.3 Common Merge Functions

| Operation | Identity | Merge |
|-----------|----------|-------|
| Sum | 0 | a + b |
| Min | ∞ | min(a, b) |
| Max | -∞ | max(a, b) |
| GCD | 0 | gcd(a, b) |
| AND | all 1s | a & b |
| OR | 0 | a \| b |
| XOR | 0 | a ^ b |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Off-by-One**: Inclusive vs exclusive ranges
2. **Wrong Tree Size**: Need 4n for safe indexing
3. **Missing Base Case**: Leaf nodes need direct assignment
4. **Non-Associative Operations**: Only associative ops work
5. **Lazy Propagation Bugs**: Must push before query/update

### 7.2 Optimization Opportunities

**Memory-Efficient Indexing**:
```rust
// For n = 2^k, can use exactly 2n nodes
let n = arr.len().next_power_of_two();
let mut tree = vec![identity; 2 * n];
```

**Iterative Build**:
```rust
// Build bottom-up (no recursion)
for i in 0..n {
    tree[n + i] = arr[i];
}
for i in (1..n).rev() {
    tree[i] = merge(&tree[2*i], &tree[2*i+1]);
}
```

**Fenwick Tree Alternative**: For sum/XOR only, Fenwick tree uses O(n) space and simpler code.

### 7.3 Recursive vs Iterative

| Aspect | Recursive | Iterative |
|--------|-----------|-----------|
| Code clarity | Clearer | More complex |
| Cache efficiency | Worse | Better |
| Stack usage | O(log n) | O(1) |
| Flexibility | More (lazy prop) | Less |

## 8. References

- Bentley, J. L. (1977). "Solutions to Klee's Rectangle Problems". Unpublished manuscript.
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press.
- Laaksonen, A. (2017). *Competitive Programmer's Handbook*. Chapter 9.
- CP-Algorithms: https://cp-algorithms.com/data_structures/segment_tree.html
