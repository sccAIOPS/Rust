# Kth Smallest Element

## 1. Overview

The Kth Smallest algorithm finds the k-th order statistic in an array—the element that would be at position k if the array were sorted. This implementation uses a partition-based approach similar to Quick Select, with average-case linear time complexity.

This is a fundamental algorithm in computer science with applications ranging from finding medians to database query optimization.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ of $n$ elements and an integer $k$ (where $1 \leq k \leq n$), find the k-th smallest element (1-indexed).

**Formal Definition:**
Find $x \in A$ such that $|\{a \in A : a < x\}| < k$ and $|\{a \in A : a \leq x\}| \geq k$

### 2.2 Order Statistics

The k-th order statistic of a set is the k-th smallest element:
- 1st order statistic: minimum
- n-th order statistic: maximum
- ⌊(n+1)/2⌋-th order statistic: lower median
- ⌈(n+1)/2⌉-th order statistic: upper median

### 2.3 Reduction Property

After partitioning around pivot at position $p$:
- k-th smallest is in left partition if $k < p + 1$
- k-th smallest is the pivot if $k = p + 1$
- k-th smallest is $(k - p - 1)$-th smallest in right partition if $k > p + 1$

## 3. Algorithm Description

### 3.1 Intuition

The algorithm repeatedly narrows down the search:
1. Partition the array around a pivot
2. The pivot lands in its final sorted position
3. If that's position k, we're done
4. Otherwise, search only the relevant partition

Unlike sorting, we discard half the array at each step!

### 3.2 Pseudocode

```
KTH-SMALLEST(A, k):
    if A is empty:
        return NOT_FOUND
    return _KTH-SMALLEST(A, k, 0, n-1)

_KTH-SMALLEST(A, k, lo, hi):
    if lo = hi:
        return A[lo]
    
    pivot ← PARTITION(A, lo, hi)
    i ← pivot - lo + 1  // Position of pivot in subarray
    
    if k = i:
        return A[pivot]
    else if k < i:
        return _KTH-SMALLEST(A, k, lo, pivot - 1)
    else:
        return _KTH-SMALLEST(A, k - i, pivot + 1, hi)
```

### 3.3 Step-by-Step Example

**Array:** `[9, 17, 3, 16, 13, 10, 1, 5, 7, 12, 4, 8, 9, 0]`  
**Find:** 6th smallest (k = 6)

**Sorted would be:** `[0, 1, 3, 4, 5, 7, 8, 9, 9, 10, 12, 13, 16, 17]`  
**6th smallest:** 7

**Execution:**

| Step | Subarray | Pivot pos | Pivot value | k | i | Action |
|------|----------|-----------|-------------|---|---|--------|
| 1 | [0..13] | 7 | 5 | 6 | 8 | k<i, search left |
| 2 | [0..6] | 3 | 3 | 6 | 4 | k>i, search right, k=6-4=2 |
| 3 | [4..6] | 5 | 7 | 2 | 2 | k=i, **Found: 7** |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Scenario |
|------|------------|----------|
| Best | $O(n)$ | Pivot always at k |
| Average | $O(n)$ | Random partitions |
| Worst | $O(n^2)$ | Always worst partition |

**Average Case Recurrence:**
$$T(n) = T(n/2) + O(n) = O(n)$$

The work done is: $n + n/2 + n/4 + ... \leq 2n$

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Input mutation | In-place |
| Stack (recursive) | $O(\log n)$ average, $O(n)$ worst |
| Auxiliary | $O(1)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use crate::sorting::partition;
use std::cmp::{Ordering, PartialOrd};

pub fn kth_smallest<T>(input: &mut [T], k: usize) -> Option<T>
where
    T: PartialOrd + Copy,
{
    if input.is_empty() {
        return None;
    }
    let kth = _kth_smallest(input, k, 0, input.len() - 1);
    Some(kth)
}

fn _kth_smallest<T>(input: &mut [T], k: usize, lo: usize, hi: usize) -> T
where
    T: PartialOrd + Copy,
{
    if lo == hi {
        return input[lo];
    }

    let pivot = partition(input, lo, hi);
    let i = pivot - lo + 1;

    match k.cmp(&i) {
        Ordering::Equal => input[pivot],
        Ordering::Less => _kth_smallest(input, k, lo, pivot - 1),
        Ordering::Greater => _kth_smallest(input, k - i, pivot + 1, hi),
    }
}
```

**Key Design Decisions:**

1. **Reuses `partition` from sorting module:** Code reuse, consistent behavior
2. **Generic over `PartialOrd + Copy`:** Works with any copyable, comparable type
3. **1-indexed k:** k=1 returns minimum (user-friendly)
4. **Returns `Option<T>`:** Handles empty array gracefully
5. **Mutates input:** Standard for selection algorithms

### 5.2 Dependency on Partition

The implementation imports `partition` from the sorting module. This should be the Lomuto or Hoare partition scheme:

```rust
// Expected partition interface
pub fn partition<T: PartialOrd>(arr: &mut [T], lo: usize, hi: usize) -> usize
```

### 5.3 Edge Cases

| Case | Input | k | Result |
|------|-------|---|--------|
| Empty | `[]` | any | `None` |
| Single | `[5]` | 1 | `Some(5)` |
| k=1 | `[3,1,2]` | 1 | `Some(1)` (min) |
| k=n | `[3,1,2]` | 3 | `Some(3)` (max) |
| Duplicates | `[2,2,2]` | 2 | `Some(2)` |
| k out of range | `[1,2]` | 5 | Undefined (may panic) |

**Important:** The implementation doesn't validate `k <= n`. Consider adding:

```rust
if k == 0 || k > input.len() {
    return None;
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Median Finding:**
   ```rust
   let median = kth_smallest(&mut data, (data.len() + 1) / 2);
   ```

2. **Percentile Calculation:**
   ```rust
   let p90 = kth_smallest(&mut data, data.len() * 90 / 100);
   ```

3. **Top-K Elements:**
   ```rust
   let threshold = kth_smallest(&mut data, data.len() - k + 1);
   let top_k: Vec<_> = data.iter().filter(|&x| x >= threshold).collect();
   ```

4. **Outlier Detection:**
   ```rust
   let q1 = kth_smallest(&mut data.clone(), data.len() / 4);
   let q3 = kth_smallest(&mut data.clone(), 3 * data.len() / 4);
   let iqr = q3 - q1;
   // Outliers are outside [q1 - 1.5*iqr, q3 + 1.5*iqr]
   ```

5. **Database LIMIT Queries:**
   - `SELECT * FROM table ORDER BY col LIMIT k`
   - Only need k-th smallest, not full sort

### 6.2 When to Use This vs. Alternatives

| Scenario | Recommendation |
|----------|---------------|
| Single k query, large n | ✓ kth_smallest |
| Multiple k values | Sort once, index directly |
| Streaming data | Heap-based approach |
| Need original array intact | Use kth_smallest_heap |
| k very small (k << n) | Heap-based may be better |

## 7. Comparison with kth_smallest_heap

| Aspect | kth_smallest (partition) | kth_smallest_heap |
|--------|--------------------------|-------------------|
| Time (average) | $O(n)$ | $O(n \log k)$ |
| Time (worst) | $O(n^2)$ | $O(n \log k)$ |
| Space | $O(1)$ + stack | $O(k)$ |
| Mutates input | Yes | No |
| Streaming support | No | Yes |
| Multiple queries | Must re-run | Heap reusable |

## 8. Variants and Optimizations

### 8.1 Median of Medians

Guarantees $O(n)$ worst case by choosing a provably good pivot:

```rust
fn kth_smallest_guaranteed<T: Ord + Copy>(arr: &mut [T], k: usize) -> T {
    if arr.len() <= 5 {
        arr.sort();
        return arr[k - 1];
    }
    
    // Divide into groups of 5, find medians
    let medians: Vec<T> = arr.chunks(5)
        .map(|chunk| {
            let mut c = chunk.to_vec();
            c.sort();
            c[c.len() / 2]
        })
        .collect();
    
    // Recursively find median of medians
    let pivot = kth_smallest_guaranteed(&mut medians.clone(), (medians.len() + 1) / 2);
    
    // Partition around this pivot and recurse
    // ...
}
```

### 8.2 Iterative Version

```rust
pub fn kth_smallest_iterative<T: PartialOrd + Copy>(
    input: &mut [T], 
    mut k: usize
) -> Option<T> {
    if input.is_empty() || k == 0 || k > input.len() {
        return None;
    }
    
    let mut lo = 0;
    let mut hi = input.len() - 1;
    
    loop {
        if lo == hi {
            return Some(input[lo]);
        }
        
        let pivot = partition(input, lo, hi);
        let i = pivot - lo + 1;
        
        if k == i {
            return Some(input[pivot]);
        } else if k < i {
            hi = pivot - 1;
        } else {
            k -= i;
            lo = pivot + 1;
        }
    }
}
```

### 8.3 Floyd-Rivest Algorithm

A more sophisticated selection algorithm with better constants:
- Expected comparisons: $n + \min(k, n-k) + O(n^{1/2})$
- Faster in practice for large arrays

## 9. References

1. Hoare, C. A. R. (1961). "Algorithm 65: Find." Communications of the ACM.
2. Blum, M., Floyd, R. W., Pratt, V., Rivest, R. L., & Tarjan, R. E. (1973). "Time bounds for selection." Journal of Computer and System Sciences.
3. Floyd, R. W., & Rivest, R. L. (1975). "Expected time bounds for selection." Communications of the ACM.
4. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Chapter 9.

---

**Implementation:** [`src/searching/kth_smallest.rs`](../../src/searching/kth_smallest.rs)  
**See Also:** [Quick Select](quick_select.md), [Kth Smallest (Heap)](kth_smallest_heap.md)
