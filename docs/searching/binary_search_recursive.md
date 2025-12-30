# Binary Search (Recursive)

## 1. Overview

The recursive implementation of Binary Search elegantly expresses the divide-and-conquer nature of the algorithm. While functionally equivalent to the iterative version, the recursive approach offers clearer logic at the cost of additional stack space. This implementation also handles both ascending and descending sorted arrays.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sorted array $A[0..n-1]$ and target $x$, find index $i$ such that $A[i] = x$.

### 2.2 Mathematical Model

**Recurrence Relation:**

$$T(n) = T(n/2) + O(1)$$

By the Master Theorem (Case 2): $T(n) = O(\log n)$

**Recursive Definition:**

$$
\text{search}(A, x, l, r) = 
\begin{cases}
\text{None} & \text{if } l \geq r \\
\text{mid} & \text{if } A[\text{mid}] = x \\
\text{search}(A, x, l, \text{mid}) & \text{if } A[\text{mid}] > x \text{ (ascending)} \\
\text{search}(A, x, \text{mid}+1, r) & \text{if } A[\text{mid}] < x \text{ (ascending)}
\end{cases}
$$

where $\text{mid} = l + \lfloor(r - l) / 2\rfloor$

### 2.3 Correctness Proof

**Strong Induction on interval size $(r - l)$:**

**Base Case:** When $l \geq r$, the interval is empty, correctly returning `None`.

**Inductive Step:** Assume correctness for all intervals of size $< k$. For an interval of size $k$:
- If $A[\text{mid}] = x$: Correctly returns `mid`
- Otherwise: Recurses on interval of size $< k/2 < k$, which is correct by induction

**Termination:** Each recursive call reduces interval size by at least half, guaranteeing termination.

## 3. Algorithm Description

### 3.1 Intuition

The recursive approach directly mirrors the mathematical definition:
1. If search space is empty → not found
2. Check the middle element
3. Recurse on the appropriate half

This is the "pure" divide-and-conquer paradigm in action.

### 3.2 Pseudocode

```
BINARY-SEARCH-REC(A, x, left, right):
    if left ≥ right:
        return NOT_FOUND
    
    is_ascending ← A[0] < A[n-1]
    mid ← left + (right - left) / 2
    
    if A[mid] = x:
        return mid
    else if (is_ascending AND A[mid] > x) OR (NOT is_ascending AND A[mid] < x):
        return BINARY-SEARCH-REC(A, x, left, mid)
    else:
        return BINARY-SEARCH-REC(A, x, mid + 1, right)
```

### 3.3 Step-by-Step Example

**Array:** `[2, 4, 6, 8, 10, 12, 14, 16]`  
**Target:** `10`

```
Call Stack Visualization:

search([2,4,6,8,10,12,14,16], 10, 0, 8)
  mid = 4, A[4] = 10
  10 == 10 → Return Some(4) ✓

Target: 5 (not in array)

search([2,4,6,8,10,12,14,16], 5, 0, 8)
│ mid = 4, A[4] = 10
│ 5 < 10 → search left half
└─► search([...], 5, 0, 4)
    │ mid = 2, A[2] = 6
    │ 5 < 6 → search left half
    └─► search([...], 5, 0, 2)
        │ mid = 1, A[1] = 4
        │ 5 > 4 → search right half
        └─► search([...], 5, 2, 2)
            │ left(2) ≥ right(2)
            └─► Return None ✗
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Recurrence |
|------|------------|------------|
| Best | $O(1)$ | Found at first mid |
| Average | $O(\log n)$ | $T(n) = T(n/2) + O(1)$ |
| Worst | $O(\log n)$ | $T(n) = T(n/2) + O(1)$ |

### 4.2 Space Complexity

| Aspect | Complexity | Notes |
|--------|------------|-------|
| Auxiliary Space | $O(1)$ | No additional data structures |
| Stack Space | $O(\log n)$ | Maximum recursion depth |
| **Total** | $O(\log n)$ | Dominated by call stack |

**Stack Frame Analysis:**
- Each recursive call adds one frame to the stack
- Maximum depth = $\lfloor \log_2 n \rfloor + 1$
- Each frame stores: `item`, `arr` (reference), `left`, `right`, return address

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn binary_search_rec<T: Ord>(
    item: &T, 
    arr: &[T], 
    left: usize, 
    right: usize
) -> Option<usize>
```

**Design Choices:**

1. **Explicit bounds:** `left` and `right` passed as parameters for recursion
2. **Slice borrowing:** Uses `&[T]` to avoid copying
3. **Order detection inside function:** Checks `arr[0] < arr[arr.len()-1]` at each call

**Potential Improvement:**
```rust
// Current: Checks order at every recursive call
let is_asc = arr.len() > 1 && arr[0] < arr[arr.len() - 1];

// Better: Pass order as parameter (optimization)
fn binary_search_rec_opt<T: Ord>(
    item: &T, arr: &[T], left: usize, right: usize, is_asc: bool
) -> Option<usize>
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | `arr.is_empty()` check returns `None` |
| Single element | Works correctly, mid = 0 |
| Two elements | Correctly handles mid calculation |
| Descending order | Auto-detected, search direction inverted |

### 5.3 Recursion Depth Considerations

For Rust's default stack size (~1-8 MB depending on platform):

| Array Size | Max Recursion Depth | Stack Safety |
|------------|---------------------|--------------|
| $10^6$ | 20 | ✓ Safe |
| $10^9$ | 30 | ✓ Safe |
| $10^{18}$ | 60 | ✓ Safe |

Binary search's logarithmic depth makes stack overflow virtually impossible in practice.

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Functional Programming**: Preferred in languages like Haskell, ML
2. **Teaching Tool**: Clearer demonstration of divide-and-conquer
3. **Formal Verification**: Easier to prove correct than iterative version
4. **Parallel Processing**: Natural for work-stealing schedulers

### 6.2 Comparison with Iterative Version

| Aspect | Recursive | Iterative |
|--------|-----------|-----------|
| Readability | More elegant | More verbose |
| Space | $O(\log n)$ stack | $O(1)$ |
| Performance | Slightly slower (call overhead) | Faster |
| Debugging | Easier stack traces | Requires more breakpoints |
| Tail-call optimization | Not guaranteed in Rust | N/A |

**Recommendation:** Use iterative for production code where performance matters; recursive for educational purposes or when clarity is paramount.

## 7. References

1. Aho, A. V., Hopcroft, J. E., & Ullman, J. D. (1974). "The Design and Analysis of Computer Algorithms". Addison-Wesley.
2. Sedgewick, R. (1998). "Algorithms in C++". Addison-Wesley. Chapter 12.
3. Graham, R. L., Knuth, D. E., & Patashnik, O. (1994). "Concrete Mathematics". Addison-Wesley. Chapter 1.

---

**Implementation:** [`src/searching/binary_search_recursive.rs`](../../src/searching/binary_search_recursive.rs)  
**See Also:** [Binary Search (Iterative)](binary_search.md), [Ternary Search (Recursive)](ternary_search_recursive.md)
