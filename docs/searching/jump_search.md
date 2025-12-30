# Jump Search

## 1. Overview

Jump Search (also called Block Search) is a searching algorithm for sorted arrays that improves upon linear search by jumping ahead by fixed steps instead of checking every element. It provides a balance between linear search's simplicity and binary search's efficiency, particularly useful when backward traversal is expensive.

Developed as an optimization for tape-based storage systems where rewinding is costly, Jump Search remains relevant in scenarios involving sequential access media or when minimizing backward movements is important.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sorted array $A[0..n-1]$ and target $x$, find index $i$ such that $A[i] = x$.

### 2.2 Mathematical Model

**Step Size Optimization:**

Let $m$ be the jump size. The algorithm performs:
- At most $\frac{n}{m}$ jumps
- At most $m-1$ linear comparisons within a block

Total comparisons: $\frac{n}{m} + m - 1$

To minimize, take derivative and set to zero:
$$\frac{d}{dm}\left(\frac{n}{m} + m\right) = -\frac{n}{m^2} + 1 = 0$$

Solving: $m = \sqrt{n}$

**Optimal step size: $m = \lfloor\sqrt{n}\rfloor$**

### 2.3 Correctness Proof

**Invariant:** After jumping to position $j$, if $x$ exists in $A$, then $x \in A[\text{prev}..j)$ or $x \in A[j..n)$.

**Two-Phase Correctness:**
1. **Jump Phase:** Stops when $A[\min(j, n-1)] \geq x$, ensuring $x$ (if present) is in $A[\text{prev}..j)$
2. **Linear Phase:** Checks $A[\text{prev}], A[\text{prev}+1], ...$ until $x$ found or $A[i] > x$

## 3. Algorithm Description

### 3.1 Intuition

Imagine searching for a name in a phone book:
1. Flip pages in large chunks (e.g., 50 pages at a time)
2. Once you've gone too far, go back one chunk
3. Search linearly within that chunk

This is faster than checking every page, but doesn't require random access like opening to exact middle.

### 3.2 Pseudocode

```
JUMP-SEARCH(A, x):
    n ← length(A)
    if n = 0:
        return NOT_FOUND
    
    step ← floor(√n)
    prev ← 0
    
    // Jump phase: find the block containing x
    while A[min(n, step) - 1] < x:
        prev ← step
        step ← step + floor(√n)
        if prev ≥ n:
            return NOT_FOUND
    
    // Linear search phase: search within the block
    while A[prev] < x:
        prev ← prev + 1
    
    if A[prev] = x:
        return prev
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]` (n = 16)  
**Target:** `11`  
**Step size:** $\sqrt{16} = 4$

**Jump Phase:**

| Jump | prev | step | Check Index | A[index] | Comparison |
|------|------|------|-------------|----------|------------|
| 1 | 0 | 4 | 3 | 3 | 3 < 11, continue |
| 2 | 4 | 8 | 7 | 7 | 7 < 11, continue |
| 3 | 8 | 12 | 11 | 11 | 11 ≥ 11, stop jumping |

**Linear Phase:**

| Step | prev | A[prev] | Comparison |
|------|------|---------|------------|
| 1 | 8 | 8 | 8 < 11, continue |
| 2 | 9 | 9 | 9 < 11, continue |
| 3 | 10 | 10 | 10 < 11, continue |
| 4 | 11 | 11 | 11 = 11, **found!** |

**Total comparisons:** 3 (jumps) + 4 (linear) = 7

**Binary search would use:** $\log_2(16) = 4$ comparisons

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Comparisons |
|------|------------|-------------|
| Best | $O(1)$ | Target at position $\sqrt{n} - 1$ |
| Average | $O(\sqrt{n})$ | $\approx \sqrt{n}$ |
| Worst | $O(\sqrt{n})$ | $\frac{n}{\sqrt{n}} + \sqrt{n} - 1 = 2\sqrt{n} - 1$ |

**Derivation:**
- Maximum jumps: $\frac{n}{\sqrt{n}} = \sqrt{n}$
- Maximum linear steps within block: $\sqrt{n} - 1$
- Total: $O(\sqrt{n})$

### 4.2 Space Complexity

| Aspect | Complexity | Notes |
|--------|------------|-------|
| Auxiliary Space | $O(1)$ | Only `step`, `prev` variables |
| Total Space | $O(1)$ | Constant extra space |

### 4.3 Comparison with Other Algorithms

| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| Linear Search | $O(n)$ | $O(1)$ | No sorting required |
| Jump Search | $O(\sqrt{n})$ | $O(1)$ | Good for sequential access |
| Binary Search | $O(\log n)$ | $O(1)$ | Requires random access |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cmp::min;

pub fn jump_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    let len = arr.len();
    if len == 0 {
        return None;
    }
    let mut step = (len as f64).sqrt() as usize;
    let mut prev = 0;

    // Jump phase
    while &arr[min(len, step) - 1] < item {
        prev = step;
        step += (len as f64).sqrt() as usize;
        if prev >= len {
            return None;
        }
    }
    
    // Linear search phase
    while &arr[prev] < item {
        prev += 1;
    }
    
    if &arr[prev] == item {
        return Some(prev);
    }
    None
}
```

**Key Implementation Details:**

1. **Step calculation:** Uses `(len as f64).sqrt() as usize` for optimal step size
2. **Bounds checking:** `min(len, step) - 1` prevents out-of-bounds access
3. **Early termination:** Returns `None` if `prev >= len` in jump phase

**Potential Issue:**
The step size is recalculated each iteration. For better performance:
```rust
let step_size = (len as f64).sqrt() as usize;  // Calculate once
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Checked first, returns `None` |
| Single element | Step = 1, works correctly |
| Target at first position | Found in first linear phase iteration |
| Target at last position | Found after max jumps + linear |
| Target not present | Linear phase exits without finding |

### 5.3 Known Limitation

The current implementation only works with **ascending** sorted arrays (unlike binary_search which handles both directions).

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Tape/Sequential Storage:** Where seeking backward is expensive
2. **Linked Lists:** More efficient than binary search (no random access)
3. **Network Searches:** When data arrives in sorted batches
4. **Embedded Systems:** When memory for recursion is limited
5. **Database Cursors:** Forward-only cursor optimization

### 6.2 When to Use Jump Search

| Scenario | Recommendation |
|----------|---------------|
| Random access available | Use Binary Search |
| Sequential access only | Use Jump Search |
| Very small arrays (<16) | Use Linear Search |
| Backward seeks expensive | Use Jump Search |
| Memory constrained | Use Jump Search (no recursion) |

### 6.3 Related Algorithms

| Algorithm | Relationship |
|-----------|-------------|
| Linear Search | Jump search degenerates to this when step = 1 |
| Binary Search | Better for random access |
| Exponential Search | Similar concept, different step growth |
| Block Search | Alternative name for jump search |

## 7. Optimizations

### 7.1 Variable Step Size

For non-uniform distributions:
```rust
fn adaptive_jump_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    // Start with larger steps, decrease as we get closer
    let mut step = arr.len() / 2;
    // ... implementation
}
```

### 7.2 Binary Search in Block

Replace linear search with binary search in the final block:
```rust
// After finding the block [prev, step)
binary_search(item, &arr[prev..min(step, len)])
    .map(|i| prev + i)
```

This changes worst case from $O(\sqrt{n})$ to $O(\sqrt{n} + \log\sqrt{n}) = O(\sqrt{n})$, but improves constants.

## 8. References

1. Baeza-Yates, R., & Salinger, A. (2010). "Experimental analysis of a fast intersection algorithm for sorted sequences." SPIRE 2010.
2. Knuth, D. E. (1998). "The Art of Computer Programming, Volume 3." Section 6.2.1.
3. Shneiderman, B. (1978). "Jump Searching: A Fast Sequential Search Technique." Communications of the ACM.

---

**Implementation:** [`src/searching/jump_search.rs`](../../src/searching/jump_search.rs)  
**See Also:** [Binary Search](binary_search.md), [Exponential Search](exponential_search.md), [Linear Search](linear_search.md)
