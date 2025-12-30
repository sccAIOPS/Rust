# Ternary Search (Recursive)

## 1. Overview

The recursive implementation of Ternary Search expresses the divide-and-conquer nature of the algorithm in its purest form. This version divides the search space into three parts recursively, providing cleaner code at the cost of stack space.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sorted array $A[0..n-1]$ and target $x$, find index $i$ such that $A[i] = x$ using recursive three-way partitioning.

### 2.2 Mathematical Model

**Recurrence Relation:**

$$T(n) = T(2n/3) + O(1)$$

By the Master Theorem: $T(n) = O(\log n)$

**Recursive Definition:**

$$
\text{search}(A, x, l, r) = 
\begin{cases}
\text{None} & \text{if } l > r \text{ or } A \text{ is empty} \\
m_1 & \text{if } A[m_1] = x \\
m_2 & \text{if } A[m_2] = x \\
\text{search}(A, x, l, m_1-1) & \text{if } x < A[m_1] \\
\text{search}(A, x, m_2+1, r) & \text{if } x > A[m_2] \\
\text{search}(A, x, m_1+1, m_2-1) & \text{otherwise}
\end{cases}
$$

where:
- $m_1 = l + (r - l) / 3$
- $m_2 = r - (r - l) / 3$

## 3. Algorithm Description

### 3.1 Intuition

The recursive approach mirrors the mathematical definition:
1. Base case: empty range means not found
2. Check both midpoints
3. Recurse on the appropriate third of the array

### 3.2 Pseudocode

```
TERNARY-SEARCH-REC(A, x, start, end):
    if A is empty:
        return NOT_FOUND
    
    if end ≥ start:
        mid1 ← start + (end - start) / 3
        mid2 ← end - (end - start) / 3
        
        if x < A[mid1]:
            return TERNARY-SEARCH-REC(A, x, start, mid1 - 1)
        if x = A[mid1]:
            return mid1
        
        if x > A[mid2]:
            return TERNARY-SEARCH-REC(A, x, mid2 + 1, end)
        if x = A[mid2]:
            return mid2
        
        return TERNARY-SEARCH-REC(A, x, mid1 + 1, mid2 - 1)
    
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `[1, 2, 3, 4, 5, 6, 7, 8, 9]` (n = 9)  
**Target:** `6`

```
Call Tree:

ternary_search_rec([1..9], 6, 0, 8)
│ mid1 = 0 + (8-0)/3 = 2  → A[2] = 3
│ mid2 = 8 - (8-0)/3 = 6  → A[6] = 7
│ 3 < 6 < 7 → search middle third
│
└─► ternary_search_rec([1..9], 6, 3, 5)
    │ mid1 = 3 + (5-3)/3 = 3  → A[3] = 4
    │ mid2 = 5 - (5-3)/3 = 5  → A[5] = 6
    │ A[mid2] = 6 = target
    │
    └─► Return Some(5) ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Comparisons |
|------|------------|-------------|
| Best | $O(1)$ | 1-2 (found at midpoint) |
| Average | $O(\log n)$ | ~$2\log_3 n$ |
| Worst | $O(\log n)$ | $2\lceil\log_3 n\rceil$ |

### 4.2 Space Complexity

| Aspect | Complexity | Notes |
|--------|------------|-------|
| Auxiliary | $O(1)$ | No extra data structures |
| Stack Space | $O(\log n)$ | Recursion depth |
| **Total** | $O(\log n)$ | Dominated by call stack |

**Stack Frame Analysis:**
- Maximum depth: $\lceil\log_3 n\rceil$
- Each frame: `target`, `list` (reference), `start`, `end`, return address

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cmp::Ordering;

pub fn ternary_search_rec<T: Ord>(
    target: &T,
    list: &[T],
    start: usize,
    end: usize,
) -> Option<usize> {
    if list.is_empty() {
        return None;
    }

    if end >= start {
        let mid1: usize = start + (end - start) / 3;
        let mid2: usize = end - (end - start) / 3;

        match target.cmp(&list[mid1]) {
            Ordering::Less => return ternary_search_rec(target, list, start, mid1 - 1),
            Ordering::Equal => return Some(mid1),
            Ordering::Greater => match target.cmp(&list[mid2]) {
                Ordering::Greater => return ternary_search_rec(target, list, mid2 + 1, end),
                Ordering::Equal => return Some(mid2),
                Ordering::Less => return ternary_search_rec(target, list, mid1 + 1, mid2 - 1),
            },
        }
    }

    None
}
```

**Design Choices:**

1. **Explicit bounds:** `start` and `end` passed as parameters
2. **Pattern matching:** Clean handling of comparison results
3. **Early returns:** Each branch explicitly returns

**Potential Issues:**

1. **Underflow risk:** `mid1 - 1` can underflow when `mid1 = 0`
   - Mitigated by the `start >= end` condition check

2. **Only ascending order:** Unlike the iterative version, doesn't handle descending arrays

### 5.2 Edge Cases

| Case | Input | Behavior |
|------|-------|----------|
| Empty list | `[], 0, 0` | Returns `None` |
| Invalid range | `start > end` | Returns `None` |
| Single element (found) | `[1], 0, 0, target=1` | Returns `Some(0)` |
| End out of bounds | `[1,2,3], 0, 3` | Works (tests show this) |

### 5.3 Comparison with Iterative

| Aspect | Recursive | Iterative |
|--------|-----------|-----------|
| Code clarity | More elegant | More verbose |
| Space | $O(\log n)$ | $O(1)$ |
| Sort order support | Ascending only | Both directions |
| Performance | Slight call overhead | Faster |

## 6. Real-World Applications

### 6.1 When to Use Recursive Version

1. **Educational purposes:** Clearer demonstration of algorithm logic
2. **Functional programming style:** Matches recursive problem definition
3. **Small to medium arrays:** Stack overflow not a concern
4. **Prototyping:** Faster to write and debug

### 6.2 When to Prefer Iterative

1. **Production code:** Better performance
2. **Very large arrays:** Avoid stack overflow risk
3. **Embedded systems:** Limited stack space
4. **Performance-critical paths:** Avoid function call overhead

## 7. Recursion Depth Analysis

For Rust's default stack size (~1-8 MB):

| Array Size | Max Recursion Depth | Safety |
|------------|---------------------|--------|
| $10^3$ | ~7 | ✓ Safe |
| $10^6$ | ~13 | ✓ Safe |
| $10^9$ | ~19 | ✓ Safe |
| $10^{18}$ | ~38 | ✓ Safe |

The logarithmic depth makes stack overflow highly unlikely.

## 8. Variants

### 8.1 Tail-Recursive Version

```rust
pub fn ternary_search_tail_rec<T: Ord>(
    target: &T,
    list: &[T],
    start: usize,
    end: usize,
) -> Option<usize> {
    if list.is_empty() || start > end {
        return None;
    }
    
    let mid1 = start + (end - start) / 3;
    let mid2 = end - (end - start) / 3;
    
    if &list[mid1] == target {
        Some(mid1)
    } else if &list[mid2] == target {
        Some(mid2)
    } else if target < &list[mid1] {
        ternary_search_tail_rec(target, list, start, mid1.saturating_sub(1))
    } else if target > &list[mid2] {
        ternary_search_tail_rec(target, list, mid2 + 1, end)
    } else {
        ternary_search_tail_rec(target, list, mid1 + 1, mid2.saturating_sub(1))
    }
}
```

Note: Rust doesn't guarantee tail-call optimization, but this style is cleaner.

## 9. References

1. Aho, A. V., Hopcroft, J. E., & Ullman, J. D. (1974). "The Design and Analysis of Computer Algorithms." Addison-Wesley.
2. Manber, U. (1989). "Introduction to Algorithms: A Creative Approach." Addison-Wesley.

---

**Implementation:** [`src/searching/ternary_search_recursive.rs`](../../src/searching/ternary_search_recursive.rs)  
**See Also:** [Ternary Search (Iterative)](ternary_search.md), [Binary Search (Recursive)](binary_search_recursive.md)
