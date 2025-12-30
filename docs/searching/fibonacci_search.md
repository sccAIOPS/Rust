# Fibonacci Search

## 1. Overview

Fibonacci Search is a comparison-based searching algorithm for sorted arrays that uses Fibonacci numbers to divide the search space. Unlike binary search which divides by 2, Fibonacci search divides in a ratio approaching the golden ratio (~1.618), and crucially, only requires addition and subtraction—no multiplication or division.

Developed by Jack Kiefer in 1953, this algorithm was particularly valuable for early computers and remains useful in systems where multiplication/division is expensive or unavailable.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sorted array $A[0..n-1]$ and target $x$, find index $i$ such that $A[i] = x$, using Fibonacci numbers to determine probe positions.

### 2.2 Fibonacci Numbers

The Fibonacci sequence: $F_0 = 0, F_1 = 1, F_k = F_{k-1} + F_{k-2}$

$$0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, ...$$

**Key Property:** $F_k = F_{k-1} + F_{k-2}$, allowing division without actual division.

### 2.3 Algorithm Principle

1. Find smallest Fibonacci number $F_k \geq n$
2. Use $F_{k-2}$ as the offset for comparison
3. Based on comparison, eliminate either $F_{k-2}$ or $F_{k-1}$ elements
4. Decrement Fibonacci indices and repeat

**Why Fibonacci?**
- $F_k = F_{k-1} + F_{k-2}$ means we can compute offsets using only addition/subtraction
- The ratio $F_{k-1}/F_k$ approaches $1/\phi \approx 0.618$, close to optimal

### 2.4 Comparison with Binary Search

| Aspect | Binary Search | Fibonacci Search |
|--------|---------------|------------------|
| Division ratio | 1:1 | $F_{k-2}:F_{k-1}$ (~0.382:0.618) |
| Operations | Requires division | Only +/- operations |
| Comparisons | $\log_2 n$ | $\log_\phi n \approx 1.44 \log_2 n$ |
| Memory access | Less predictable | More sequential |

## 3. Algorithm Description

### 3.1 Intuition

Imagine you have a ruler marked only at Fibonacci positions (1, 1, 2, 3, 5, 8, 13, ...):
1. Find the Fibonacci mark that covers your search range
2. Use the Fibonacci sequence to determine where to look
3. Based on comparison, shift to smaller Fibonacci numbers
4. The sequence guarantees you'll find the target or prove it's absent

### 3.2 Pseudocode

```
FIBONACCI-SEARCH(A, x):
    n ← length(A)
    if n = 0:
        return NOT_FOUND
    
    // Initialize Fibonacci numbers
    fib0 ← 0       // F(k-2)
    fib1 ← 1       // F(k-1)
    fib2 ← 1       // F(k)
    
    // Find smallest Fibonacci ≥ n
    while fib2 < n:
        fib0 ← fib1
        fib1 ← fib2
        fib2 ← fib0 + fib1
    
    // Offset for eliminated range
    offset ← -1
    
    while fib2 > 1:
        // Get valid index
        index ← min(offset + fib0, n - 1)
        
        if A[index] < x:
            fib2 ← fib1
            fib1 ← fib0
            fib0 ← fib2 - fib1
            offset ← index
        else if A[index] > x:
            fib2 ← fib0
            fib1 ← fib1 - fib0
            fib0 ← fib2 - fib1
        else:
            return index
    
    // Check last element
    if fib1 = 1 AND A[n-1] = x:
        return n - 1
    
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `[10, 22, 35, 40, 45, 50, 80, 82, 85, 90, 100]` (n = 11)  
**Target:** `85`

**Initialization:**

| F(k-2) | F(k-1) | F(k) | Condition |
|--------|--------|------|-----------|
| 0 | 1 | 1 | 1 < 11 |
| 1 | 1 | 2 | 2 < 11 |
| 1 | 2 | 3 | 3 < 11 |
| 2 | 3 | 5 | 5 < 11 |
| 3 | 5 | 8 | 8 < 11 |
| 5 | 8 | 13 | 13 ≥ 11 ✓ |

**Search Phase:**

| Step | fib2 | fib1 | fib0 | offset | index | A[index] | Action |
|------|------|------|------|--------|-------|----------|--------|
| 1 | 13 | 8 | 5 | -1 | min(-1+5,10)=4 | 45 | 45<85, offset=4 |
| 2 | 8 | 5 | 3 | 4 | min(4+3,10)=7 | 82 | 82<85, offset=7 |
| 3 | 5 | 3 | 2 | 7 | min(7+2,10)=9 | 90 | 90>85 |
| 4 | 2 | 1 | 1 | 7 | min(7+1,10)=8 | 85 | **Found!** |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Comparisons |
|------|------------|-------------|
| Best | $O(1)$ | 1 |
| Average | $O(\log n)$ | $\log_\phi n \approx 1.44 \log_2 n$ |
| Worst | $O(\log n)$ | $\log_\phi n$ |

**Derivation:**
- Each iteration reduces search space by at least $F_{k-2}/F_k$ fraction
- $F_k/F_{k-2} \rightarrow \phi^2 \approx 2.618$
- Iterations needed: $\log_{\phi^2} n = \frac{\log n}{2 \log \phi} \approx 1.04 \log_2 n$

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Variables | $O(1)$ |
| Total | $O(1)$ |

Only stores: `fib0`, `fib1`, `fib2`, `offset`, `index`

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cmp::min;
use std::cmp::Ordering;

pub fn fibonacci_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    let len = arr.len();
    if len == 0 {
        return None;
    }
    let mut start = -1;  // Note: signed integer for offset

    // Initialize Fibonacci numbers
    let mut f0 = 0;
    let mut f1 = 1;
    let mut f2 = 1;
    
    // Find smallest Fibonacci ≥ len
    while f2 < len {
        f0 = f1;
        f1 = f2;
        f2 = f0 + f1;
    }
    
    // Search
    while f2 > 1 {
        let index = min((f0 as isize + start) as usize, len - 1);
        match item.cmp(&arr[index]) {
            Ordering::Less => {
                f2 = f0;
                f1 -= f0;
                f0 = f2 - f1;
            }
            Ordering::Equal => return Some(index),
            Ordering::Greater => {
                f2 = f1;
                f1 = f0;
                f0 = f2 - f1;
                start = index as isize;
            }
        }
    }
    
    // Check if last element matches
    if (f1 != 0) && (&arr[len - 1] == item) {
        return Some(len - 1);
    }
    None
}
```

**Implementation Details:**

1. **Signed offset:** Uses `isize` for `start` to handle negative offset (-1 initial)
2. **Generic type:** Works with any `Ord` type
3. **Bounds protection:** `min((f0 as isize + start) as usize, len - 1)`
4. **Final check:** Handles edge case where target is at last position

### 5.2 Edge Cases

| Case | Behavior |
|------|----------|
| Empty array | Returns `None` immediately |
| Single element | Works correctly |
| Target at last position | Caught by final `f1 != 0` check |
| Target not present | Returns `None` after loop |
| Array size = Fibonacci number | Optimal performance |

### 5.3 Potential Issues

1. **Signed/unsigned conversion:** The `start = -1` and subsequent conversions are error-prone
2. **Only ascending order:** Doesn't support descending arrays (unlike binary_search)

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Embedded Systems:** Where multiplication/division is expensive or unavailable
2. **Hardware Implementations:** FPGA/ASIC where adders are cheaper than multipliers
3. **Memory-Constrained Systems:** Constant space requirement
4. **Magnetic Tape Search:** Sequential access benefits from Fibonacci's pattern
5. **Educational:** Understanding non-binary search strategies

### 6.2 When to Use Fibonacci Search

| Scenario | Recommendation |
|----------|---------------|
| Division is expensive | ✓ Fibonacci Search |
| General purpose | Binary Search |
| Sequential access pattern preferred | ✓ Fibonacci Search |
| Very large arrays | Binary Search (fewer comparisons) |
| Hardware without multiplier | ✓ Fibonacci Search |

### 6.3 Historical Context

Fibonacci search was more important in early computing when:
- Division operations took many more cycles than addition
- Memory access patterns mattered more
- Specialized hardware lacked multiplication units

Modern CPUs have made division fast, reducing Fibonacci search's advantages.

## 7. Comparison with Other Algorithms

| Algorithm | Operations | Comparisons | Memory Pattern |
|-----------|------------|-------------|----------------|
| Binary | +, -, × or / | $\log_2 n$ | Random |
| Fibonacci | +, - only | $1.44\log_2 n$ | More sequential |
| Interpolation | +, -, ×, / | $O(\log\log n)$* | Random |
| Jump | +, -, √ | $O(\sqrt{n})$ | Sequential jumps |

*For uniformly distributed data

## 8. Variants and Optimizations

### 8.1 Two-Way Fibonacci Search

Searches from both ends simultaneously:

```rust
fn two_way_fibonacci_search<T: Ord>(item: &T, arr: &[T]) -> Option<usize> {
    // Search from start and end, meeting in middle
    // Uses Fibonacci from both directions
}
```

### 8.2 Parallel Fibonacci Search

For very large arrays, different Fibonacci ranges can be searched in parallel.

### 8.3 Pre-computed Fibonacci Table

```rust
const FIB: [usize; 47] = [
    0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610,
    987, 1597, 2584, 4181, 6765, 10946, 17711, 28657, 46368, 75025,
    121393, 196418, 317811, 514229, 832040, 1346269, 2178309, 3524578,
    5702887, 9227465, 14930352, 24157817, 39088169, 63245986, 102334155,
    165580141, 267914296, 433494437, 701408733, 1134903170, 1836311903
];
// Covers arrays up to ~1.8 billion elements
```

## 9. Mathematical Connection to Golden Ratio

The ratio $F_{k+1}/F_k$ converges to $\phi = \frac{1 + \sqrt{5}}{2} \approx 1.618$

This means Fibonacci search divides the array in the golden ratio, which is:
- Asymptotically optimal for comparison-based search with certain cost functions
- Related to Golden Section Search for optimization

**Binet's Formula:**
$$F_n = \frac{\phi^n - \psi^n}{\sqrt{5}}$$

where $\phi = \frac{1+\sqrt{5}}{2}$ and $\psi = \frac{1-\sqrt{5}}{2}$

## 10. References

1. Kiefer, J. (1953). "Sequential minimax search for a maximum." Proceedings of the American Mathematical Society.
2. Knuth, D. E. (1998). "The Art of Computer Programming, Vol. 3: Sorting and Searching." Section 6.2.1.
3. Ferguson, D. E. (1960). "Fibonaccian searching." Communications of the ACM.
4. Overholt, K. J. (1973). "Efficiency of the Fibonacci search method." BIT Numerical Mathematics.

---

**Implementation:** [`src/searching/fibonacci_search.rs`](../../src/searching/fibonacci_search.rs)  
**See Also:** [Binary Search](binary_search.md), [Exponential Search](exponential_search.md), [Jump Search](jump_search.md)
