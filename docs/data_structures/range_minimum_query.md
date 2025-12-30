# Range Minimum Query (RMQ)

## 1. Overview

Range Minimum Query (RMQ) is a data structure problem: preprocess an array to answer queries asking for the minimum element in any subarray. Various solutions offer different trade-offs between preprocessing time, query time, and space. The Sparse Table achieves O(n log n) preprocessing with O(1) queries.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, preprocess it to answer:
- **Query(l, r)**: Return $\min(A[l], A[l+1], ..., A[r])$

Optionally also return the index of the minimum.

### 2.2 Mathematical Model

**Sparse Table Approach**:

Precompute $M[i][j]$ = index of minimum in $A[i..i+2^j-1]$

For any range $[l, r]$:
$$k = \lfloor \log_2(r - l + 1) \rfloor$$
$$\text{RMQ}(l, r) = \min(A[M[l][k]], A[M[r-2^k+1][k]])$$

**Key Property**: Overlapping ranges are OK for min/max (idempotent operations).

### 2.3 Why O(1) Query Works

For range $[l, r]$ of length $n$:
- Let $k = \lfloor \log_2 n \rfloor$
- Range $[l, l+2^k-1]$ covers at least half
- Range $[r-2^k+1, r]$ covers at least half
- Together they cover entire $[l, r]$
- Min of overlapping mins = min of range

## 3. Algorithm Description

### 3.1 Intuition

Think of it as precomputing answers for "powers of two" sized ranges. Any query range can be covered by at most two overlapping power-of-two ranges, and since we're finding minimum, overlap doesn't matter.

### 3.2 Structure Visualization

```
Array A: [2, 4, 3, 1, 6, 7, 8, 9, 1, 7]
Indices:  0  1  2  3  4  5  6  7  8  9

Sparse Table M[i][j] = index of min in A[i..i+2^j-1]:

j=0 (length 1):  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]  // M[i][0] = i
j=1 (length 2):  [0, 2, 3, 3, 4, 5, 6, 8, 8, _]
j=2 (length 4):  [3, 3, 3, 3, 4, 8, 8, 8, _, _]
j=3 (length 8):  [3, 3, 8, _, _, _, _, _, _, _]

Query RMQ(1, 6):
  Length = 6, k = floor(log2(6)) = 2 (covers length 4)
  Left:  M[1][2] = 3 → A[3] = 1
  Right: M[6-4+1][2] = M[3][2] = 3 → A[3] = 1
  Answer: min(1, 1) = 1 at index 3 ✓
```

### 3.3 Pseudocode

```
BUILD_SPARSE_TABLE(A):
    n = len(A)
    LOG = precompute_logs(n)
    K = floor(log2(n)) + 1
    M = array[n][K]
    
    // Initialize for length 1
    for i from 0 to n-1:
        M[i][0] = i
    
    // Build for larger lengths
    for j from 1 to K-1:
        for i from 0 to n - 2^j:
            left = M[i][j-1]
            right = M[i + 2^(j-1)][j-1]
            if A[left] <= A[right]:
                M[i][j] = left
            else:
                M[i][j] = right
    
    return M, LOG

QUERY(A, M, LOG, l, r):
    length = r - l + 1
    k = LOG[length]
    left = M[l][k]
    right = M[r - 2^k + 1][k]
    if A[left] <= A[right]:
        return left
    else:
        return right
```

### 3.4 Step-by-Step Example

**Build for A = [5, 2, 4, 7, 6, 3, 1, 2]**:

```
j=0: M[i][0] = i
     [0, 1, 2, 3, 4, 5, 6, 7]

j=1: M[i][1] = argmin(A[M[i][0]], A[M[i+1][0]])
     M[0][1] = argmin(A[0]=5, A[1]=2) = 1
     M[1][1] = argmin(A[1]=2, A[2]=4) = 1
     M[2][1] = argmin(A[2]=4, A[3]=7) = 2
     ...
     [1, 1, 2, 4, 5, 6, 6, _]

j=2: M[i][2] = argmin(A[M[i][1]], A[M[i+2][1]])
     M[0][2] = argmin(A[1]=2, A[2]=4) = 1
     M[1][2] = argmin(A[1]=2, A[4]=6) = 1
     M[2][2] = argmin(A[2]=4, A[6]=1) = 6
     ...
     [1, 1, 6, 6, 6, _, _, _]

j=3: M[i][3]
     M[0][3] = argmin(A[1]=2, A[6]=1) = 6
     [6, _, _, _, _, _, _, _]

Query RMQ(2, 5):
  length = 4, k = 2
  M[2][2] = 6, A[6] = 1
  M[5-4+1][2] = M[2][2] = 6, A[6] = 1
  Wait, that's wrong range. Let me recalculate.
  
  Actually for range [2,5], we need:
  M[2][2] covers [2,5] ✓
  M[5-3][2] = M[2][2] covers [2,5] (same!)
  Answer: A[6]? No wait, 6 is not in [2,5].
  
  Let me recalculate j=2 more carefully:
  M[2][2] = min of [2,3,4,5] = min(4,7,6,3) = 3 at index 5
  
So M[2][2] = 5, A[5] = 3, which is correct min of [2,5]
```

## 4. Complexity Analysis

### 4.1 Time/Space Trade-offs

| Method | Preprocess | Query | Space |
|--------|------------|-------|-------|
| No preprocessing | - | O(n) | O(1) |
| Precompute all | O(n²) | O(1) | O(n²) |
| Sparse Table | O(n log n) | O(1) | O(n log n) |
| Segment Tree | O(n) | O(log n) | O(n) |
| Block decomposition | O(n) | O(√n) | O(√n) |
| Fischer-Heun | O(n) | O(1) | O(n) |

### 4.2 Sparse Table Complexity

- **Preprocessing**: O(n log n) time and space
- **Query**: O(1) time

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct RangeMinimumQuery<T> {
    data: Vec<T>,
    sparse_table: Vec<Vec<usize>>,
    log_table: Vec<usize>,
}

impl<T: Ord + Clone> RangeMinimumQuery<T> {
    pub fn new(data: Vec<T>) -> Self {
        let n = data.len();
        if n == 0 {
            return RangeMinimumQuery {
                data,
                sparse_table: vec![],
                log_table: vec![],
            };
        }
        
        // Precompute log values
        let mut log_table = vec![0; n + 1];
        for i in 2..=n {
            log_table[i] = log_table[i / 2] + 1;
        }
        
        let k = log_table[n] + 1;
        let mut sparse_table = vec![vec![0; k]; n];
        
        // Initialize for length 1
        for i in 0..n {
            sparse_table[i][0] = i;
        }
        
        // Build sparse table
        for j in 1..k {
            let len = 1 << j;
            for i in 0..=n - len {
                let left = sparse_table[i][j - 1];
                let right = sparse_table[i + (1 << (j - 1))][j - 1];
                sparse_table[i][j] = if data[left] <= data[right] {
                    left
                } else {
                    right
                };
            }
        }
        
        RangeMinimumQuery { data, sparse_table, log_table }
    }
    
    pub fn query(&self, l: usize, r: usize) -> Option<usize> {
        if l > r || r >= self.data.len() {
            return None;
        }
        
        let len = r - l + 1;
        let k = self.log_table[len];
        let left = self.sparse_table[l][k];
        let right = self.sparse_table[r - (1 << k) + 1][k];
        
        Some(if self.data[left] <= self.data[right] {
            left
        } else {
            right
        })
    }
    
    pub fn query_value(&self, l: usize, r: usize) -> Option<&T> {
        self.query(l, r).map(|idx| &self.data[idx])
    }
}
```

**Key Design Patterns**:
- Store indices (not values) for flexibility
- Precompute log table to avoid floating point
- `Option` return for invalid queries
- Generic over `T: Ord`

### 5.2 Log Table Precomputation

```rust
// O(n) precomputation of floor(log2(i))
fn build_log_table(n: usize) -> Vec<usize> {
    let mut log = vec![0; n + 1];
    for i in 2..=n {
        log[i] = log[i / 2] + 1;
    }
    log
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Return None |
| Single element | Return that element |
| l > r | Return None or swap |
| l == r | Return single element |
| Ties | Return leftmost (or rightmost) |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Lowest Common Ancestor**: LCA reduces to RMQ
2. **Suffix Arrays**: LCP queries use RMQ
3. **Computational Geometry**: Nearest smaller element
4. **Stock Analysis**: Minimum price in date range
5. **Signal Processing**: Peak/trough detection

### 6.2 LCA to RMQ Reduction

Given a tree, preprocess for LCA queries:
1. Euler tour of tree, recording depths
2. RMQ on depth array gives LCA

```
Tree:       Euler Tour: [A, B, A, C, D, C, E, C, A]
    A       Depths:     [0, 1, 0, 1, 2, 1, 2, 1, 0]
   /|\
  B C       LCA(B, E) = node at min depth in tour between B and E
    |       = A (depth 0)
   D E
```

### 6.3 Variants

| Variant | Description |
|---------|-------------|
| **Range Maximum Query** | Same structure, compare max |
| **2D RMQ** | Sparse table in 2D, O(1) query |
| **Dynamic RMQ** | Segment tree for updates |
| **RMQ with updates** | Use segment tree instead |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Off-by-One**: `r - (1 << k) + 1` not `r - (1 << k)`
2. **Integer Overflow**: `1 << k` for large k
3. **Log Calculation**: Use integer log, not float
4. **Boundary Checks**: Ensure indices in bounds
5. **Non-Idempotent Ops**: Sparse table only works for idempotent (min, max, gcd, and, or)

### 7.2 When Not to Use Sparse Table

**Use Segment Tree Instead When**:
- Need point/range updates
- Operation isn't idempotent (e.g., sum)
- Memory is very constrained

**Use Fischer-Heun When**:
- Need O(n) space with O(1) query
- Worth the implementation complexity

### 7.3 Fischer-Heun Optimization

Achieves O(n) preprocessing and O(1) query:
1. Divide into blocks of size (log n) / 2
2. Use sparse table on block minima
3. Precompute all answers within blocks (only O(√n) unique block types)

Complex but theoretically optimal.

### 7.4 Comparison Summary

| Use Case | Best Structure |
|----------|----------------|
| Static array, many queries | Sparse Table |
| Need updates | Segment Tree |
| Memory constrained | Segment Tree |
| Theoretically optimal | Fischer-Heun |
| Simple implementation | Sparse Table |

## 8. References

- Bender, M. A., & Farach-Colton, M. (2000). "The LCA Problem Revisited". *LATIN 2000*.
- Fischer, J., & Heun, V. (2006). "Theoretical and Practical Improvements on the RMQ-Problem". *CPM 2006*.
- Berkman, O., & Vishkin, U. (1993). "Recursive Star-Tree Parallel Data Structure". *SIAM J. Computing*.
- CP-Algorithms: https://cp-algorithms.com/data_structures/sparse-table.html
