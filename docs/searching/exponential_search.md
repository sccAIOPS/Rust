# Exponential Search

## 1. Overview

Exponential Search (also called Doubling Search or Galloping Search) is an algorithm for searching sorted arrays that combines the best aspects of linear and binary search. It first finds a range where the target element may exist by exponentially increasing the search position, then performs binary search within that range.

Particularly useful for unbounded or infinite arrays, this algorithm was developed for scenarios where the array size is unknown or when the target is likely near the beginning.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sorted array $A[0..n-1]$ (possibly unbounded) and target $x$, find index $i$ such that $A[i] = x$.

### 2.2 Mathematical Model

**Two-Phase Approach:**

1. **Range Finding Phase:** Find smallest $k$ such that $A[2^k] \geq x$
2. **Binary Search Phase:** Search in range $A[2^{k-1}..2^k]$

**Complexity Analysis:**

If target is at position $i$:
- Range finding: $O(\log i)$ comparisons to find $k$ where $2^{k-1} < i \leq 2^k$
- Binary search: $O(\log(2^k - 2^{k-1})) = O(\log 2^{k-1}) = O(k-1) = O(\log i)$
- Total: $O(\log i)$

### 2.3 Key Insight

Unlike binary search which always takes $O(\log n)$ regardless of target position, exponential search takes $O(\log i)$ where $i$ is the target's position. This is optimal when:
- The target is near the beginning
- The array size is unknown
- The array is effectively infinite (generator-based)

## 3. Algorithm Description

### 3.1 Intuition

Think of searching for a house number on a very long street:
1. Check house #1
2. Check house #2, #4, #8, #16, ... (doubling each time)
3. Once you've gone past the target, you know it's between your last two positions
4. Do a detailed search in that range

This is more efficient than always starting from the middle if the target is near the beginning.

### 3.2 Pseudocode

```
EXPONENTIAL-SEARCH(A, x):
    n ← length(A)
    if n = 0:
        return NOT_FOUND
    
    // Phase 1: Find range by exponential jumps
    bound ← 1
    while bound < n AND A[bound] ≤ x:
        bound ← bound × 2
    
    // Ensure bound doesn't exceed array length
    if bound > n:
        bound ← n
    
    // Phase 2: Binary search in range [bound/2, bound)
    left ← bound / 2
    right ← bound
    
    while left < right:
        mid ← left + (right - left) / 2
        if A[mid] = x:
            return mid
        else if A[mid] < x:
            left ← mid + 1
        else:
            right ← mid
    
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `[1, 3, 5, 7, 9, 11, 13, 15, 17, 19]` (n = 10)  
**Target:** `7`

**Phase 1: Range Finding**

| Step | bound | A[bound] | Comparison | Action |
|------|-------|----------|------------|--------|
| 1 | 1 | 3 | 3 ≤ 7 | bound = 2 |
| 2 | 2 | 5 | 5 ≤ 7 | bound = 4 |
| 3 | 4 | 9 | 9 > 7 | Stop, range is [2, 4) |

**Phase 2: Binary Search in [2, 4)**

| Step | left | right | mid | A[mid] | Action |
|------|------|-------|-----|--------|--------|
| 1 | 2 | 4 | 3 | 7 | 7 = 7, **Found!** |

**Total comparisons:** 3 (range finding) + 1 (binary search) = 4

**Comparison with Binary Search:**
- Binary search: $\log_2(10) \approx 4$ comparisons
- Exponential search: $\log_2(4) \times 2 \approx 4$ comparisons

**For target at position 1:**
- Exponential: ~2 comparisons
- Binary: ~4 comparisons

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | When |
|------|------------|------|
| Best | $O(1)$ | Target at position 0 or 1 |
| Average | $O(\log i)$ | Target at position $i$ |
| Worst | $O(\log n)$ | Target at end or not present |

**Key Property:** Performance is $O(\log i)$ where $i$ is the target's position, making it adaptive.

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Auxiliary | $O(1)$ |
| Total | $O(1)$ |

### 4.3 Comparison Table

| Algorithm | Complexity | Bounded Array | Unbounded Array |
|-----------|------------|---------------|-----------------|
| Linear | $O(n)$ | ✓ | ✓ |
| Binary | $O(\log n)$ | ✓ | ✗ (needs size) |
| Exponential | $O(\log i)$ | ✓ | ✓ |
| Interpolation | $O(\log \log n)$* | ✓ | ✗ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cmp::Ordering;

pub fn exponential_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    let len = arr.len();
    if len == 0 {
        return None;
    }
    
    // Phase 1: Find range
    let mut upper = 1;
    while (upper < len) && (&arr[upper] <= item) {
        upper *= 2;
    }
    if upper > len {
        upper = len
    }

    // Phase 2: Binary search in [lower, upper)
    let mut lower = upper / 2;
    while lower < upper {
        let mid = lower + (upper - lower) / 2;
        match item.cmp(&arr[mid]) {
            Ordering::Less => upper = mid,
            Ordering::Equal => return Some(mid),
            Ordering::Greater => lower = mid + 1,
        }
    }
    None
}
```

**Implementation Quality:**
- ✓ Generic over `Ord` types
- ✓ Handles empty arrays
- ✓ Prevents overflow with `upper > len` check
- ✓ Efficient binary search implementation

**Minor Issue:** The check `&arr[upper] <= item` uses `<=`, meaning if target equals `arr[upper]`, we continue doubling unnecessarily. Consider `< item` for slight improvement.

### 5.2 Edge Cases

| Case | Behavior |
|------|----------|
| Empty array | Returns `None` immediately |
| Single element (found) | Works (upper=1, binary search finds it) |
| Single element (not found) | Returns `None` |
| Target at position 0 | Found in binary search phase |
| Target at position 1 | Found quickly (upper=2, binary search in [0,2)) |
| Target beyond array | upper caps at len, returns `None` |

### 5.3 Optimization for Position 0

The current implementation might miss position 0 in the range-finding loop. Add:

```rust
if &arr[0] == item {
    return Some(0);
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Unbounded Sorted Lists:** Searching in infinite sequences (e.g., prime numbers)
2. **Log File Analysis:** When searching recent entries (likely near start)
3. **Autocomplete Suggestion:** Finding completions in a large sorted dictionary
4. **Database Query Optimization:** When index position is uncertain
5. **Network Protocols:** Finding sequence numbers in sliding windows
6. **Cache Lines:** Searching for recently accessed items (temporal locality)

### 6.2 When to Use Exponential Search

| Scenario | Recommendation |
|----------|---------------|
| Array size unknown | ✓ Exponential Search |
| Target likely near start | ✓ Exponential Search |
| Uniformly distributed | Interpolation or Binary Search |
| Small arrays | Binary Search |
| Known target region | Direct Binary Search on subrange |

### 6.3 Real-World Example: Git Bisect

Git's `bisect` command uses exponential search concepts:
- Start with recent commits
- Double the range until the bug-introducing commit is bracketed
- Binary search to find exact commit

## 7. Variants

### 7.1 Galloping Search (in Merge Operations)

Used in TimSort for merging sorted runs:

```rust
fn gallop_left<T: Ord>(key: &T, arr: &[T], base: usize) -> usize {
    let mut offset = 1;
    while base + offset < arr.len() && arr[base + offset] < *key {
        offset *= 2;
    }
    // Binary search in [base + offset/2, base + offset)
}
```

### 7.2 One-Sided Binary Search

Variation that only expands in one direction:

```rust
fn one_sided_binary_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    let mut step = 1;
    let mut prev = 0;
    
    // Expand exponentially
    while step < arr.len() && arr[step] < *item {
        prev = step;
        step = step.saturating_mul(2);
    }
    
    // Binary search in [prev, min(step, len))
    let end = step.min(arr.len());
    binary_search_in_range(item, arr, prev, end)
}
```

### 7.3 Fibonacci Search Connection

Fibonacci search is similar but uses Fibonacci numbers instead of powers of 2:
- Exponential: 1, 2, 4, 8, 16, 32, ...
- Fibonacci: 1, 1, 2, 3, 5, 8, 13, 21, ...

Fibonacci avoids expensive multiplication/division on some architectures.

## 8. Performance Benchmarks

For an array of 1,000,000 elements:

| Target Position | Binary Search | Exponential Search |
|-----------------|---------------|--------------------|
| 0 | ~20 comparisons | ~2 comparisons |
| 10 | ~20 comparisons | ~8 comparisons |
| 1000 | ~20 comparisons | ~20 comparisons |
| 500,000 | ~20 comparisons | ~38 comparisons |
| 999,999 | ~20 comparisons | ~40 comparisons |

**Takeaway:** Exponential search shines when targets are near the beginning.

## 9. References

1. Bentley, J. L., & Yao, A. C. C. (1976). "An almost optimal algorithm for unbounded searching." Information Processing Letters.
2. Baeza-Yates, R., & Salinger, A. (2005). "Fast intersection algorithms for sorted sequences." Algorithms and Applications.
3. Peters, T. (2002). "TimSort" (Python's sorting algorithm using galloping search).

---

**Implementation:** [`src/searching/exponential_search.rs`](../../src/searching/exponential_search.rs)  
**See Also:** [Binary Search](binary_search.md), [Jump Search](jump_search.md), [Fibonacci Search](fibonacci_search.md)
