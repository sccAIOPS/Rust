# Linear Search

## 1. Overview

Linear Search (also known as Sequential Search) is the simplest searching algorithm. It sequentially checks each element of the array until a match is found or the entire array has been traversed. While not optimal for sorted data, it is the only option for unsorted collections and remains relevant in many practical scenarios.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ of $n$ elements (not necessarily sorted) and a target value $x$, find the smallest index $i$ such that $A[i] = x$, or determine that no such index exists.

### 2.2 Mathematical Model

**Input:**
- An array $A$ of $n$ comparable elements (any order)
- A target element $x$

**Output:**
- Index $i$ where $A[i] = x$ (first occurrence), or indication that $x \notin A$

**Preconditions:**
- Elements must support equality comparison (`Eq` trait; `Ord` used in implementation for consistency)

### 2.3 Correctness Proof

**Loop Invariant:** $x \notin A[0..i)$ (target not in elements checked so far)

**Initialization:** Before the loop, $i = 0$, so $A[0..0)$ is empty. Trivially, $x$ is not in an empty set.

**Maintenance:** If $A[i] = x$, we return $i$ (correct). Otherwise, $x \neq A[i]$, and by the invariant, $x \notin A[0..i)$, so $x \notin A[0..i+1)$.

**Termination:** When $i = n$, by the invariant, $x \notin A[0..n)$, so $x$ is not in the array.

## 3. Algorithm Description

### 3.1 Intuition

Linear search mimics how humans naturally search:
1. Start from the beginning
2. Check each item one by one
3. Stop when found or when all items checked

Think of finding a book on an unsorted bookshelf—you examine each book in sequence.

### 3.2 Pseudocode

```
LINEAR-SEARCH(A, x):
    for i ← 0 to n-1:
        if A[i] = x:
            return i
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `["cat", "dog", "bird", "fish", "rabbit"]`  
**Target:** `"fish"`

| Step | Index | Element | Comparison | Result |
|------|-------|---------|------------|--------|
| 1 | 0 | "cat" | "cat" ≠ "fish" | Continue |
| 2 | 1 | "dog" | "dog" ≠ "fish" | Continue |
| 3 | 2 | "bird" | "bird" ≠ "fish" | Continue |
| 4 | 3 | "fish" | "fish" = "fish" | **Found at index 3** |

**Target:** `"elephant"` (not in array)

| Step | Index | Element | Comparison | Result |
|------|-------|---------|------------|--------|
| 1-5 | 0-4 | all | all ≠ "elephant" | Continue |
| 6 | - | - | End of array | **Not found** |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | When |
|------|------------|------|
| Best | $O(1)$ | Target is first element |
| Average | $O(n)$ | Target at random position or not present |
| Worst | $O(n)$ | Target is last element or not present |

**Expected Comparisons:**
- If element is present (uniform probability): $(n + 1) / 2$
- If element may be absent: $n$ comparisons worst case

### 4.2 Space Complexity

| Aspect | Complexity | Notes |
|--------|------------|-------|
| Auxiliary Space | $O(1)$ | Only loop counter |
| Total Space | $O(1)$ | No additional allocation |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn linear_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    for (i, data) in arr.iter().enumerate() {
        if item == data {
            return Some(i);
        }
    }
    None
}
```

**Design Choices:**

1. **Iterator with `enumerate`**: Idiomatic Rust, avoids manual indexing
2. **Early return**: Returns immediately on match
3. **`Ord` bound**: Technically only needs `Eq`, but `Ord` allows consistency with other search functions
4. **Reference comparison**: `item == data` compares references to avoid copying

**Alternative Implementation (more generic):**
```rust
pub fn linear_search_eq<T: Eq>(item: &T, arr: &[T]) -> Option<usize> {
    arr.iter().position(|x| x == item)
}
```

### 5.2 Edge Cases

| Case | Input | Expected | Handled |
|------|-------|----------|---------|
| Empty array | `[]`, any target | `None` | ✓ |
| Single element (found) | `[5]`, target=5 | `Some(0)` | ✓ |
| Single element (not found) | `[5]`, target=3 | `None` | ✓ |
| Duplicates | `[1,2,2,3]`, target=2 | `Some(1)` (first) | ✓ |
| All same elements | `[7,7,7]`, target=7 | `Some(0)` | ✓ |
| Target at end | `[1,2,3]`, target=3 | `Some(2)` | ✓ |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Small Collections**: For arrays with < 10-20 elements, linear search often outperforms binary search due to cache efficiency
2. **Unsorted Data**: When data cannot be sorted (e.g., streaming data)
3. **Single-Use Search**: When you'll only search once (sorting overhead not worth it)
4. **Linked Lists**: No random access makes binary search impossible
5. **Finding First Match**: When multiple criteria or complex predicates are involved
6. **Cache-Friendly Access**: Sequential memory access pattern

### 6.2 When to Prefer Linear Search

| Scenario | Why Linear Search |
|----------|-------------------|
| $n < 10$ | Sorting overhead exceeds search savings |
| Unsorted data, one search | $O(n)$ search < $O(n \log n)$ sort |
| Linked list | Binary search needs $O(n)$ random access |
| Finding all occurrences | Must scan entire array anyway |
| Complex predicates | `arr.iter().find(|x| x.matches_complex_criteria())` |

### 6.3 Comparison with Binary Search

| Aspect | Linear Search | Binary Search |
|--------|---------------|---------------|
| Prerequisite | None | Sorted array |
| Time Complexity | $O(n)$ | $O(\log n)$ |
| Break-even point | ~10 elements | >10 elements |
| Memory access | Sequential | Random |
| Cache performance | Excellent | Good |

## 7. Optimizations

### 7.1 Sentinel Search

Eliminates bounds checking by placing target at the end:

```rust
fn sentinel_search<T: Eq + Clone>(item: &T, arr: &mut Vec<T>) -> Option<usize> {
    let n = arr.len();
    if n == 0 { return None; }
    
    let last = arr[n-1].clone();
    arr[n-1] = item.clone();  // Sentinel
    
    let mut i = 0;
    while &arr[i] != item {
        i += 1;
    }
    
    arr[n-1] = last;  // Restore
    
    if i < n - 1 || &arr[n-1] == item {
        Some(i)
    } else {
        None
    }
}
```

**Benefit:** Removes one comparison per iteration (bounds check).

### 7.2 Move-to-Front Heuristic

For repeated searches with temporal locality:

```rust
fn mtf_search<T: Eq>(item: &T, arr: &mut [T]) -> Option<usize> {
    for i in 0..arr.len() {
        if &arr[i] == item {
            arr[..=i].rotate_right(1);  // Move to front
            return Some(0);
        }
    }
    None
}
```

**Benefit:** Frequently accessed elements move to front, improving average case.

## 8. References

1. Knuth, D. E. (1998). "The Art of Computer Programming, Volume 3: Sorting and Searching" (2nd ed.). Section 6.1.
2. Sedgewick, R. & Wayne, K. (2011). "Algorithms" (4th ed.). Addison-Wesley. Chapter 1.
3. Bentley, J. L. (2000). "Programming Pearls" (2nd ed.). Addison-Wesley.

---

**Implementation:** [`src/searching/linear_search.rs`](../../src/searching/linear_search.rs)  
**See Also:** [Binary Search](binary_search.md), [Jump Search](jump_search.md)
