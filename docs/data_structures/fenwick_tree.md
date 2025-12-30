# Fenwick Tree (Binary Indexed Tree)

## 1. Overview

A Fenwick Tree (also known as Binary Indexed Tree or BIT) is a data structure that efficiently supports prefix sum queries and point updates in O(log n) time. It's more memory-efficient than a segment tree (using only O(n) space) and has simpler implementation, though it's less versatile.

Invented by Peter Fenwick in 1994.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[1..n]$ (1-indexed), support:
- **Update(i, delta)**: Add `delta` to $A[i]$
- **Prefix Sum(i)**: Compute $\sum_{j=1}^{i} A[j]$
- **Range Sum(l, r)**: Compute $\sum_{j=l}^{r} A[j]$ = PrefixSum(r) - PrefixSum(l-1)

### 2.2 Mathematical Model

**Key Insight**: Use binary representation of indices to determine responsibilities.

For index $i$, define $g(i) = i \& (-i)$ (lowest set bit of $i$).

**Tree Structure**:
- Node $i$ stores sum of elements in range $[i - g(i) + 1, i]$
- Range length = $g(i)$ = lowest set bit of $i$

**Example** (n=8):
| Index i | Binary | g(i) | Responsible for |
|---------|--------|------|-----------------|
| 1 | 001 | 1 | [1,1] |
| 2 | 010 | 2 | [1,2] |
| 3 | 011 | 1 | [3,3] |
| 4 | 100 | 4 | [1,4] |
| 5 | 101 | 1 | [5,5] |
| 6 | 110 | 2 | [5,6] |
| 7 | 111 | 1 | [7,7] |
| 8 | 1000 | 8 | [1,8] |

### 2.3 Navigation

**Query (going down)**: Remove lowest set bit: `i -= i & (-i)`
**Update (going up)**: Add lowest set bit: `i += i & (-i)`

## 3. Algorithm Description

### 3.1 Intuition

Think of a Fenwick tree as a clever partial sum caching system. Instead of storing every prefix sum (wasteful) or no prefix sums (slow queries), it stores strategic partial sums that allow reconstructing any prefix sum in O(log n) additions.

### 3.2 Structure Visualization

```
Array A: [_, 1, 3, 2, 5, 1, 4, 2, 3]  (1-indexed, A[0] unused)

Fenwick Tree BIT:
Index:    1    2    3    4    5    6    7    8
Binary:  001  010  011  100  101  110  111  1000
BIT[i]:   1    4    2   11    1    5    2   21

BIT[1] = A[1] = 1
BIT[2] = A[1] + A[2] = 4
BIT[3] = A[3] = 2
BIT[4] = A[1..4] = 11
BIT[5] = A[5] = 1
BIT[6] = A[5] + A[6] = 5
BIT[7] = A[7] = 2
BIT[8] = A[1..8] = 21

Query PrefixSum(7):
  7 (111) → BIT[7] = 2
  6 (110) → BIT[6] = 5
  4 (100) → BIT[4] = 11
  0 (000) → done
  Sum = 2 + 5 + 11 = 18 = A[1..7] ✓
```

### 3.3 Pseudocode

```
// Get lowest set bit
LSB(i):
    return i & (-i)

// Prefix sum query [1..i]
QUERY(BIT, i):
    sum = 0
    while i > 0:
        sum += BIT[i]
        i -= LSB(i)
    return sum

// Point update: add delta to position i
UPDATE(BIT, i, delta):
    while i <= n:
        BIT[i] += delta
        i += LSB(i)

// Range sum [l..r]
RANGE_SUM(BIT, l, r):
    return QUERY(BIT, r) - QUERY(BIT, l - 1)

// Build from array
BUILD(arr):
    BIT = copy of arr (1-indexed)
    for i from 1 to n:
        j = i + LSB(i)
        if j <= n:
            BIT[j] += BIT[i]
    return BIT
```

### 3.4 Step-by-Step Example

**Update A[3] += 5, then Query PrefixSum(6)**:

```
Initial BIT: [_, 1, 4, 2, 11, 1, 5, 2, 21]

Update(3, 5):
  i = 3 (011): BIT[3] += 5 → BIT[3] = 7
  i = 3 + 1 = 4 (100): BIT[4] += 5 → BIT[4] = 16
  i = 4 + 4 = 8 (1000): BIT[8] += 5 → BIT[8] = 26
  i = 8 + 8 = 16 > n, done

Updated BIT: [_, 1, 4, 7, 16, 1, 5, 2, 26]

Query(6):
  i = 6 (110): sum = BIT[6] = 5
  i = 6 - 2 = 4 (100): sum += BIT[4] = 5 + 16 = 21
  i = 4 - 4 = 0: done
  
Result: 21 = (1 + 3 + 7 + 5 + 1 + 4) = 21 ✓
(Original was 16, +5 to A[3] gives 21)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time |
|-----------|------|
| Build | O(n) |
| Point Update | O(log n) |
| Prefix Query | O(log n) |
| Range Query | O(log n) |

**Why O(log n)**: Each step removes/adds the lowest set bit. At most log n bits in any index.

### 4.2 Space Complexity

- **Storage**: O(n) - just one array
- **Auxiliary**: O(1) - no recursion needed

### 4.3 Comparison with Segment Tree

| Aspect | Fenwick Tree | Segment Tree |
|--------|--------------|--------------|
| Space | O(n) | O(2n) to O(4n) |
| Code complexity | Simple | More complex |
| Query types | Prefix sums | Any range |
| Update types | Point only* | Point or range |
| Cache efficiency | Better | Worse |

*Range updates possible with modifications

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct FenwickTree {
    tree: Vec<i64>,
}

impl FenwickTree {
    pub fn with_len(n: usize) -> Self {
        FenwickTree { tree: vec![0; n + 1] }
    }
    
    pub fn from_vec(arr: &[i64]) -> Self {
        let n = arr.len();
        let mut tree = vec![0; n + 1];
        
        // Copy array (1-indexed)
        for i in 0..n {
            tree[i + 1] = arr[i];
        }
        
        // Build tree
        for i in 1..=n {
            let j = i + (i & i.wrapping_neg());
            if j <= n {
                tree[j] += tree[i];
            }
        }
        
        FenwickTree { tree }
    }
    
    pub fn update(&mut self, mut i: usize, delta: i64) {
        i += 1;  // Convert to 1-indexed
        while i < self.tree.len() {
            self.tree[i] += delta;
            i += i & i.wrapping_neg();
        }
    }
    
    pub fn prefix_sum(&self, mut i: usize) -> i64 {
        i += 1;  // Convert to 1-indexed
        let mut sum = 0;
        while i > 0 {
            sum += self.tree[i];
            i -= i & i.wrapping_neg();
        }
        sum
    }
    
    pub fn range_sum(&self, l: usize, r: usize) -> i64 {
        if l == 0 {
            self.prefix_sum(r)
        } else {
            self.prefix_sum(r) - self.prefix_sum(l - 1)
        }
    }
}
```

**Key Design Patterns**:
- 1-indexed internally (tree[0] unused)
- `wrapping_neg()` for safe `-i` operation
- Convert 0-indexed user API to 1-indexed internal
- O(n) build instead of O(n log n) individual updates

### 5.2 Finding Lowest Set Bit

```rust
// Method 1: Two's complement trick
fn lsb(i: usize) -> usize {
    i & i.wrapping_neg()
}

// Method 2: XOR trick
fn lsb_xor(i: usize) -> usize {
    i & (i ^ (i - 1))
}

// Method 3: Trailing zeros
fn lsb_tz(i: usize) -> usize {
    1 << i.trailing_zeros()
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty tree | prefix_sum returns 0 |
| Single element | Works correctly |
| Query index 0 | Return 0 (empty prefix) |
| Update at 0 | Shift to 1-indexed |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Cumulative Frequency Tables**: Count inversions
2. **Range Sum Queries**: Financial running totals
3. **Order Statistics**: Kth smallest with coordinate compression
4. **2D Range Queries**: 2D Fenwick tree for matrix sums
5. **Competitive Programming**: Fast and simple

### 6.2 Counting Inversions

```rust
fn count_inversions(arr: &[i32]) -> i64 {
    let n = arr.len();
    let mut sorted: Vec<_> = arr.iter().cloned().enumerate().collect();
    sorted.sort_by_key(|&(_, v)| v);
    
    let mut bit = FenwickTree::with_len(n);
    let mut inversions = 0i64;
    
    for (rank, &(original_idx, _)) in sorted.iter().enumerate() {
        // Count elements to the right that were processed earlier
        inversions += bit.prefix_sum(n - 1) - bit.prefix_sum(original_idx);
        bit.update(original_idx, 1);
    }
    
    inversions
}
```

### 6.3 Variants

| Variant | Description |
|---------|-------------|
| **Range Update + Point Query** | Store differences |
| **Range Update + Range Query** | Use two BITs |
| **2D Fenwick Tree** | Nested structure for matrices |
| **Fenwick Tree on Ranges** | For maximum/minimum |

### 6.4 Range Update, Point Query

Store differences: $D[i] = A[i] - A[i-1]$

```rust
// Range add [l, r] by val
fn range_add(&mut self, l: usize, r: usize, val: i64) {
    self.update(l, val);
    if r + 1 < self.len() {
        self.update(r + 1, -val);
    }
}

// Point query
fn point_query(&self, i: usize) -> i64 {
    self.prefix_sum(i)
}
```

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **0 vs 1 Indexing**: BIT is inherently 1-indexed
2. **Negative Numbers**: Two's complement works, but be careful
3. **Integer Overflow**: Use appropriate type (i64)
4. **Off-by-One in Range**: `range_sum(l, r)` = `prefix(r) - prefix(l-1)`
5. **Building Inefficiently**: Use O(n) build, not O(n log n)

### 7.2 Optimization Opportunities

**Batch Updates**: Build from scratch is faster than n individual updates

**SIMD**: Modern CPUs can parallelize the bit operations

**Cache Efficiency**: Already excellent due to simple array structure

### 7.3 When to Use Fenwick vs Segment Tree

**Use Fenwick Tree**:
- Prefix/range sums only
- Memory is constrained
- Simpler code preferred
- No range updates needed

**Use Segment Tree**:
- Arbitrary range queries (min, max, GCD)
- Range updates with lazy propagation
- Non-invertible operations

## 8. References

- Fenwick, P. M. (1994). "A New Data Structure for Cumulative Frequency Tables". *Software: Practice and Experience*.
- Halim, S. & Halim, F. (2013). *Competitive Programming 3*. Chapter 2.4.4.
- TopCoder Tutorial: "Binary Indexed Trees".
- CP-Algorithms: https://cp-algorithms.com/data_structures/fenwick.html
