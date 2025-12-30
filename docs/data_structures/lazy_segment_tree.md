# Lazy Segment Tree

## 1. Overview

A Lazy Segment Tree extends the standard segment tree with lazy propagation, enabling efficient range updates in O(log n) time. Instead of immediately updating all affected nodes, it defers (lazily propagates) updates until they're actually needed, making range modifications as efficient as point updates.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, support:
- **Range Query(l, r)**: Compute aggregate over $A[l..r]$
- **Range Update(l, r, v)**: Apply modification to all elements in $A[l..r]$

### 2.2 Mathematical Model

**Lazy Tag**: Each node stores a pending operation to be applied to its subtree.

**Composition Property**: If applying update $u_1$ then $u_2$, must be able to combine: $u_2 \circ u_1$

**Supported Operations**:
| Update Type | Composition | Query Affected |
|-------------|-------------|----------------|
| Range Add | $u_2 + u_1$ | Sum: add $v \times len$ |
| Range Set | $u_2$ (overwrites) | Depends on value |
| Range XOR | $u_2 \oplus u_1$ | Various |

### 2.3 Key Invariant

A node's value is always correct assuming all lazy tags from root to that node have been applied.

## 3. Algorithm Description

### 3.1 Intuition

Imagine a manager who receives orders to give raises to departments. Instead of immediately updating every employee's salary, they post a notice on each department door: "All employees get +$100". Only when someone actually needs their salary (query) does the manager walk through and apply all pending notices.

### 3.2 Lazy Propagation Flow

```
Range Update [2,5] with +10:

Before:                        After (with lazy tags):
       [36]                           [36]              lazy=0
      /    \                         /    \
   [9]      [27]                  [9]      [27+40]      lazy=+10
   / \      /  \                  / \      /    \
 [4] [5] [16] [11]             [4] [5] [16+20] [11+10]
                                        lazy=+10  lazy=+10
                                        
Note: Values in brackets show actual stored values + what lazy would add
Actual tree[node] isn't updated yet - lazy tag remembers the pending update
```

### 3.3 Push Down Operation

```
PUSH_DOWN(node, start, end):
    if lazy[node] != IDENTITY:
        mid = (start + end) / 2
        left_len = mid - start + 1
        right_len = end - mid
        
        // Apply lazy to children
        tree[left] += lazy[node] * left_len  // For sum
        tree[right] += lazy[node] * right_len
        
        // Pass lazy tag to children
        lazy[left] += lazy[node]
        lazy[right] += lazy[node]
        
        // Clear current lazy
        lazy[node] = IDENTITY
```

### 3.4 Pseudocode

```
RANGE_UPDATE(node, start, end, l, r, val):
    if r < start or end < l:
        return  // No overlap
    
    if l <= start and end <= r:
        // Total overlap: apply lazy update
        tree[node] += val * (end - start + 1)
        lazy[node] += val
        return
    
    // Partial overlap: push down first
    PUSH_DOWN(node, start, end)
    
    mid = (start + end) / 2
    RANGE_UPDATE(left, start, mid, l, r, val)
    RANGE_UPDATE(right, mid+1, end, l, r, val)
    tree[node] = tree[left] + tree[right]

RANGE_QUERY(node, start, end, l, r):
    if r < start or end < l:
        return 0  // No overlap
    
    if l <= start and end <= r:
        return tree[node]  // Total overlap
    
    // Partial overlap: push down first
    PUSH_DOWN(node, start, end)
    
    mid = (start + end) / 2
    return RANGE_QUERY(left, start, mid, l, r) +
           RANGE_QUERY(right, mid+1, end, l, r)
```

### 3.5 Step-by-Step Example

**Array: [1, 3, 5, 7, 9, 11], Range Add +10 to [1,4], then Query [2,5]**

```
Initial tree (sum):
          [36]
         /    \
      [9]      [27]
      / \      /  \
    [4] [5] [16] [11]

Range Update [1,4] +10:
1. At root [0,5]: partial overlap, recurse
2. At [0,2]: partial overlap
   - At [0,1]: partial overlap  
     - [0]: no overlap
     - [1]: total overlap → tree += 10, lazy = 10
   - [2]: total overlap → tree += 10, lazy = 10
3. At [3,5]: partial overlap
   - [3,4]: total overlap → tree += 20, lazy = 10
   - [5]: no overlap

Tree after update:
          [76]           (36 + 40 for 4 elements)
         /    \
      [29]     [47]
      / \      /   \
   [4] [13+L] [36+L] [11]     L = lazy tag 10

Query [2,5]:
1. At root: partial, push down... (but root has no lazy)
2. At [0,2]: partial
   - Push down: [1] gets lazy applied
   - Query [2]: returns 15 (5 + 10)
3. At [3,5]: partial  
   - Push down [3,4]: values updated
   - Query [3,4]: returns 36
   - Query [5]: returns 11
4. Total: 15 + 36 + 11 = 62

Verify: (3+10) + (5+10) + (7+10) + (9+10) + 11 = 13 + 15 + 17 + 19 + 11 = 75
Wait, let me recalculate: A[2..5] = 5+7+9+11 + 10*3 (indices 2,3,4 got +10)
= 5+10 + 7+10 + 9+10 + 11 = 62 ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time |
|-----------|------|
| Build | O(n) |
| Range Query | O(log n) |
| Range Update | O(log n) |
| Point Query | O(log n) |
| Point Update | O(log n) |

**Key Insight**: Each node is visited at most twice per operation (once going down, once coming up).

### 4.2 Space Complexity

- **Storage**: O(n) for tree + O(n) for lazy array = O(n)
- **Recursion**: O(log n) stack space

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct LazySegmentTree<T> {
    len: usize,
    tree: Vec<T>,
    lazy: Vec<T>,
}

impl LazySegmentTree<i64> {
    pub fn new(arr: &[i64]) -> Self {
        let len = arr.len();
        let mut st = LazySegmentTree {
            len,
            tree: vec![0; 4 * len],
            lazy: vec![0; 4 * len],
        };
        if len > 0 {
            st.build(arr, 0, 0, len - 1);
        }
        st
    }
    
    fn push_down(&mut self, node: usize, start: usize, end: usize) {
        if self.lazy[node] != 0 {
            let mid = (start + end) / 2;
            let left = 2 * node + 1;
            let right = 2 * node + 2;
            
            // Apply to left child
            self.tree[left] += self.lazy[node] * (mid - start + 1) as i64;
            self.lazy[left] += self.lazy[node];
            
            // Apply to right child
            self.tree[right] += self.lazy[node] * (end - mid) as i64;
            self.lazy[right] += self.lazy[node];
            
            self.lazy[node] = 0;
        }
    }
    
    pub fn range_update(&mut self, l: usize, r: usize, val: i64) {
        self.update_recursive(0, 0, self.len - 1, l, r, val);
    }
    
    pub fn range_query(&mut self, l: usize, r: usize) -> i64 {
        self.query_recursive(0, 0, self.len - 1, l, r)
    }
}
```

**Key Design Patterns**:
- Separate `tree` and `lazy` vectors
- `push_down` before recursing on partial overlap
- Identity element is 0 for addition
- `&mut self` for query (due to lazy propagation)

### 5.2 Different Update Types

**Range Set (Assign)**:
```rust
fn push_down_set(&mut self, node: usize, start: usize, end: usize) {
    if self.lazy[node].is_some() {
        let val = self.lazy[node].unwrap();
        let mid = (start + end) / 2;
        
        self.tree[left] = val * (mid - start + 1) as i64;
        self.tree[right] = val * (end - mid) as i64;
        self.lazy[left] = Some(val);
        self.lazy[right] = Some(val);
        self.lazy[node] = None;
    }
}
```

**Range Add + Range Set** (need priority):
```rust
struct LazyTag {
    set: Option<i64>,  // Applied first if present
    add: i64,          // Applied after set
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty range (l > r) | Return identity, no update |
| Single element range | Works like point update |
| Update then query same range | Push down ensures correctness |
| Overlapping updates | Composition must be correct |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Range Updates in Databases**: Bulk salary increases
2. **Game Development**: AOE (area of effect) damage
3. **Network Flow**: Capacity updates on paths
4. **Scheduling**: Time slot modifications
5. **Image Processing**: Region-based filters
6. **Financial Systems**: Portfolio-wide adjustments

### 6.2 Supported Query/Update Combinations

| Query | Update | Works? | Notes |
|-------|--------|--------|-------|
| Sum | Add | ✓ | Classic case |
| Sum | Set | ✓ | Need Option for lazy |
| Min | Add | ✓ | min unchanged by constant add |
| Min | Set | ✓ | Straightforward |
| Max | Add | ✓ | Same as min |
| GCD | Add | ✗ | GCD not preserved under addition |

### 6.3 Beats (Segment Tree Beats)

For complex updates like "set $A[i] = \min(A[i], v)$" for range:
- Track min/max/second-min in each node
- Conditionally propagate based on values
- Enables operations standard lazy can't handle

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Forgetting Push Down**: Must push before partial overlap recursion
2. **Wrong Update Formula**: Sum needs `val * length`
3. **Query Mutation**: Query must also push down (mutates tree)
4. **Composition Order**: `new_lazy = compose(new_update, old_lazy)`
5. **Identity Confusion**: Different for add (0) vs set (None)

### 7.2 Optimization Opportunities

**Avoid Unnecessary Push**:
```rust
fn query(&mut self, node: usize, ...) -> i64 {
    if /* no overlap */ { return 0; }
    if /* total overlap */ { 
        return self.tree[node];  // No push needed!
    }
    self.push_down(node, start, end);
    // recurse...
}
```

**Iterative Implementation**: Possible but complex, usually recursive is fine.

**Memory Layout**: Consider storing `(value, lazy)` pairs for better cache locality.

### 7.3 Debugging Tips

1. Print tree state after each operation
2. Verify invariant: node value correct if all ancestor lazy applied
3. Test with small arrays, verify against brute force
4. Check edge cases: single element, entire array, empty ranges

## 8. References

- Laaksonen, A. (2017). *Competitive Programmer's Handbook*. Chapter 28.
- CP-Algorithms: https://cp-algorithms.com/data_structures/segment_tree.html
- Ji, R. "Segment Tree Beats". *2016 China National Olympiad in Informatics*.
- Eppstein, D. "Lazy Propagation in Segment Trees". *Algorithm Design Manual*.
