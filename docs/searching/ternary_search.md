# Ternary Search

## 1. Overview

Ternary Search is a divide-and-conquer algorithm that divides the search space into three parts instead of two (as in binary search). While it has a slightly higher constant factor than binary search for array searching, its primary value lies in finding extrema (maxima/minima) of unimodal functions—a task where binary search cannot be directly applied.

This implementation handles both ascending and descending sorted arrays, automatically detecting the sort order.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**For Array Search:**
Given a sorted array $A[0..n-1]$ and target $x$, find index $i$ such that $A[i] = x$.

**For Function Optimization:**
Given a unimodal function $f$ on interval $[a, b]$, find $x^*$ such that $f(x^*)$ is the maximum (or minimum).

### 2.2 Mathematical Model

**Trisection Process:**

Divide $[l, r]$ into three equal parts using two midpoints:
- $m_1 = l + \frac{r - l}{3}$
- $m_2 = r - \frac{r - l}{3}$

**Decision Logic (for ascending array):**
- If $A[m_1] = x$: Found at $m_1$
- If $A[m_2] = x$: Found at $m_2$
- If $x < A[m_1]$: Search in $[l, m_1 - 1]$
- If $x > A[m_2]$: Search in $[m_2 + 1, r]$
- Otherwise: Search in $[m_1 + 1, m_2 - 1]$

### 2.3 Complexity Comparison

**Binary Search:**
- Reduces search space by factor of 2 each iteration
- $\log_2 n$ comparisons

**Ternary Search:**
- Reduces search space by factor of 3 each iteration
- But requires 2 comparisons per iteration
- Total: $2 \times \log_3 n = 2 \times \frac{\log_2 n}{\log_2 3} \approx 1.26 \times \log_2 n$

**Conclusion:** For array search, binary search is ~26% more efficient.

### 2.4 Correctness Proof

**Loop Invariant:** If $x$ exists in array, then $x \in A[\text{left}..\text{right}]$.

**Termination:** Search space reduces by at least 1/3 each iteration:
$$\text{new size} \leq \frac{2}{3} \times \text{old size}$$

After $k$ iterations: $\text{size} \leq n \times (2/3)^k$

Terminates when size becomes 0 or 1.

## 3. Algorithm Description

### 3.1 Intuition

Imagine dividing a sorted bookshelf into three sections:
1. Check two dividing points
2. If target is before first divider, search left section
3. If target is after second divider, search right section
4. Otherwise, search the middle section

### 3.2 Pseudocode

```
TERNARY-SEARCH(A, x):
    if A is empty:
        return NOT_FOUND
    
    is_ascending ← A[0] < A[n-1]
    left ← 0
    right ← n - 1
    
    while left ≤ right:
        mid1 ← left + (right - left) / 3
        mid2 ← right - (right - left) / 3
        
        // Handle edge case where interval is very small
        if mid1 = mid2 = left:
            if A[left] = x:
                return left
            left ← left + 1
            continue
        
        if A[mid1] = x:
            return mid1
        if A[mid2] = x:
            return mid2
        
        if (is_ascending AND x < A[mid1]) OR (NOT is_ascending AND x > A[mid1]):
            right ← mid1 - 1
        else if (is_ascending AND x > A[mid2]) OR (NOT is_ascending AND x < A[mid2]):
            left ← mid2 + 1
        else:
            left ← mid1 + 1
            right ← mid2 - 1
    
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `[1, 3, 5, 7, 9, 11, 13, 15, 17]` (n = 9)  
**Target:** `11`

| Step | left | right | mid1 | mid2 | A[mid1] | A[mid2] | Action |
|------|------|-------|------|------|---------|---------|--------|
| 1 | 0 | 8 | 2 | 6 | 5 | 13 | 5<11<13, search [3,5] |
| 2 | 3 | 5 | 3 | 5 | 7 | 11 | A[mid2]=11, **Found!** |

**Comparisons:** 4 (2 per iteration × 2 iterations)  
**Binary search:** ~4 comparisons

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Notes |
|------|------------|-------|
| Best | $O(1)$ | Target at first mid1 or mid2 |
| Average | $O(\log n)$ | ~$1.26 \log_2 n$ comparisons |
| Worst | $O(\log n)$ | ~$2 \log_3 n$ comparisons |

**Recurrence:** $T(n) = T(2n/3) + O(1) = O(\log n)$

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Iterative | $O(1)$ |
| Recursive | $O(\log n)$ stack |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn ternary_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    if arr.is_empty() {
        return None;
    }
    let is_asc = is_asc_arr(arr);
    let mut left = 0;
    let mut right = arr.len() - 1;

    while left <= right {
        if match_compare(item, arr, &mut left, &mut right, is_asc) {
            return Some(left);
        }
    }
    None
}
```

**Key Implementation Details:**

1. **Handles both sort orders:** Uses `is_asc_arr()` helper
2. **Modular design:** `match_compare` function encapsulates comparison logic
3. **Edge case handling:** Special case when `mid1 == mid2 == left`
4. **Saturating subtraction:** Uses `saturating_sub(1)` to prevent underflow

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Returns `None` immediately |
| Single element | Special case in `match_compare` |
| Two elements | Handled correctly |
| Three elements | mid1=0, mid2=2 works correctly |
| Descending order | Detected and handled |
| Element not present | Loop exits, returns `None` |

### 5.3 Implementation Quality

**Strengths:**
- Clean separation of concerns
- Handles edge cases thoroughly
- Supports both sort orders

**Potential Issue:**
- More complex than binary search for same performance
- Only justified when extending to unimodal function optimization

## 6. Real-World Applications

### 6.1 Primary Use Case: Unimodal Function Optimization

The real power of ternary search is finding extrema of unimodal functions:

```
Unimodal Function (for maximum):
      /\
     /  \
    /    \
   /      \
  /        \
```

Binary search cannot directly find the peak, but ternary search can.

### 6.2 Software Engineering Use Cases

1. **Machine Learning:** Finding optimal hyperparameters on a smooth loss surface
2. **Graphics:** Finding intersection points on curves
3. **Game Development:** AI decision making on utility curves
4. **Signal Processing:** Finding peaks in smooth signals
5. **Competitive Programming:** Optimization problems with unimodal objective

### 6.3 When NOT to Use for Array Search

| Scenario | Better Alternative |
|----------|-------------------|
| Sorted array lookup | Binary Search |
| Unsorted data | Linear Search or Hash Table |
| Very small arrays | Linear Search |
| Multimodal function | Gradient descent or exhaustive search |

## 7. Comparison with Binary Search

| Aspect | Binary Search | Ternary Search |
|--------|---------------|----------------|
| Comparisons per iteration | 1 | 2 |
| Space reduction | 1/2 | 2/3 |
| Total comparisons | $\log_2 n$ | $2\log_3 n \approx 1.26\log_2 n$ |
| Implementation complexity | Simple | Moderate |
| Array search | ✓ Better | Suboptimal |
| Unimodal optimization | ✗ Cannot | ✓ Excellent |

## 8. References

1. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Problem 9-3.
2. Karp, R. M. (1972). "Reducibility Among Combinatorial Problems." Complexity of Computer Computations.
3. Preparata, F. P., & Shamos, M. I. (1985). "Computational Geometry." Springer-Verlag.

---

**Implementation:** [`src/searching/ternary_search.rs`](../../src/searching/ternary_search.rs)  
**See Also:** [Ternary Search (Recursive)](ternary_search_recursive.md), [Ternary Search Min/Max](ternary_search_min_max.md), [Binary Search](binary_search.md)
