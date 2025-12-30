# Binary Search

## 1. Overview

Binary Search is a fundamental divide-and-conquer algorithm for finding the position of a target value within a sorted array. First described by John Mauchly in 1946, it remains one of the most efficient searching algorithms for sorted data, serving as a building block for more complex algorithms.

The implementation in this repository handles both ascending and descending sorted arrays, automatically detecting the sort order.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sorted array $A[0..n-1]$ of $n$ elements and a target value $x$, find an index $i$ such that $A[i] = x$, or determine that no such index exists.

### 2.2 Mathematical Model

**Input:**
- A sorted array $A$ of $n$ comparable elements
- A target element $x$

**Output:**
- Index $i$ where $A[i] = x$, or indication that $x \notin A$

**Preconditions:**
- Array must be sorted (ascending or descending)
- Elements must support total ordering (`Ord` trait in Rust)

### 2.3 Correctness Proof

**Loop Invariant:** If $x$ exists in the array, then $x \in A[left..right)$

**Initialization:** Initially, $left = 0$ and $right = n$, so $x \in A[0..n)$ if $x$ exists.

**Maintenance:** At each iteration:
- If $A[mid] = x$: Found, return $mid$
- If $A[mid] < x$ (ascending): $x$ cannot be in $A[left..mid]$, so update $left = mid + 1$
- If $A[mid] > x$ (ascending): $x$ cannot be in $A[mid+1..right)$, so update $right = mid$

**Termination:** The interval $[left, right)$ shrinks by at least half each iteration. When $left \geq right$, the interval is empty, confirming $x \notin A$.

## 3. Algorithm Description

### 3.1 Intuition

Imagine searching for a word in a dictionary:
1. Open to the middle page
2. If the target word comes before the current page, search the left half
3. If it comes after, search the right half
4. Repeat until found or no pages remain

### 3.2 Pseudocode

```
BINARY-SEARCH(A, x):
    is_ascending ← A[0] < A[n-1]
    left ← 0
    right ← n
    
    while left < right:
        mid ← left + (right - left) / 2    // Prevents overflow
        
        if A[mid] = x:
            return mid
        else if (is_ascending AND A[mid] < x) OR (NOT is_ascending AND A[mid] > x):
            left ← mid + 1
        else:
            right ← mid
    
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `[1, 3, 5, 7, 9, 11, 13]` (ascending)  
**Target:** `7`

| Step | left | right | mid | A[mid] | Comparison | Action |
|------|------|-------|-----|--------|------------|--------|
| 1 | 0 | 7 | 3 | 7 | 7 = 7 | **Found at index 3** |

**Array:** `[1, 3, 5, 7, 9, 11, 13]`  
**Target:** `6`

| Step | left | right | mid | A[mid] | Comparison | Action |
|------|------|-------|-----|--------|------------|--------|
| 1 | 0 | 7 | 3 | 7 | 6 < 7 | right = 3 |
| 2 | 0 | 3 | 1 | 3 | 6 > 3 | left = 2 |
| 3 | 2 | 3 | 2 | 5 | 6 > 5 | left = 3 |
| 4 | 3 | 3 | - | - | left ≥ right | **Not found** |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| Best | $O(1)$ | Target is at the middle position |
| Average | $O(\log n)$ | Each comparison halves the search space |
| Worst | $O(\log n)$ | Target at boundary or not present |

**Derivation:**

After $k$ iterations, the search space size is $\frac{n}{2^k}$.

The algorithm terminates when $\frac{n}{2^k} \leq 1$, i.e., $k \geq \log_2 n$.

Therefore, maximum iterations = $\lfloor \log_2 n \rfloor + 1 = O(\log n)$.

### 4.2 Space Complexity

| Aspect | Complexity | Notes |
|--------|------------|-------|
| Auxiliary Space | $O(1)$ | Only uses `left`, `right`, `mid` variables |
| Stack Space | $O(1)$ | Iterative implementation |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn binary_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize>
```

**Key Design Decisions:**

1. **Generic over `Ord`**: Works with any totally ordered type
2. **Returns `Option<usize>`**: Idiomatic Rust for possibly-missing values
3. **Takes `&T` and `&[T]`**: Borrows data, avoiding unnecessary copies
4. **Handles both sort orders**: Auto-detects ascending/descending via `is_asc_arr`

**Overflow Prevention:**
```rust
let mid = left + (right - left) / 2;  // Safe
// vs.
let mid = (left + right) / 2;  // Can overflow!
```

### 5.2 Edge Cases

| Case | Input | Expected Output | Handled |
|------|-------|-----------------|---------|
| Empty array | `[]` | `None` | ✓ |
| Single element (found) | `[5]`, target=5 | `Some(0)` | ✓ |
| Single element (not found) | `[5]`, target=3 | `None` | ✓ |
| First element | `[1,2,3]`, target=1 | `Some(0)` | ✓ |
| Last element | `[1,2,3]`, target=3 | `Some(2)` | ✓ |
| Descending order | `[5,4,3,2,1]`, target=3 | `Some(2)` | ✓ |
| With gaps | `[1,3,8,11]`, target=5 | `None` | ✓ |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Database Indexing**: B-trees use binary search within nodes
2. **Version Control**: Git's `bisect` command uses binary search to find bugs
3. **Autocomplete**: Finding word suggestions in sorted dictionaries
4. **IP Routing**: Finding the correct route in sorted routing tables
5. **Game Development**: Collision detection with sorted spatial partitions

### 6.2 Related Algorithms

| Algorithm | When to Prefer |
|-----------|---------------|
| Linear Search | Unsorted data or very small arrays (n < 10) |
| Interpolation Search | Uniformly distributed numeric data |
| Exponential Search | Unbounded or very large arrays |
| Ternary Search | Finding extrema in unimodal functions |

## 7. References

1. Knuth, D. E. (1998). "The Art of Computer Programming, Volume 3: Sorting and Searching" (2nd ed.). Addison-Wesley. Section 6.2.1.
2. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Chapter 2.
3. Bentley, J. L. (1986). "Programming Pearls". Addison-Wesley. Column 4.

---

**Implementation:** [`src/searching/binary_search.rs`](../../src/searching/binary_search.rs)  
**See Also:** [Binary Search (Recursive)](binary_search_recursive.md), [Exponential Search](exponential_search.md)
