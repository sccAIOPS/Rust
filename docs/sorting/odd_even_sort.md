# Odd-Even Sort

## 1. Overview

Odd-Even Sort (also called Brick Sort or Odd-Even Transposition Sort) is a parallel sorting algorithm based on Bubble Sort. It compares adjacent elements in alternating odd-even indexed pairs, making it suitable for parallel processing.

### Key Characteristics
- **Type**: Comparison-based, exchange sort
- **In-place**: Yes
- **Stable**: Yes
- **Parallel**: Designed for parallel execution

## 2. Mathematical Foundation

### 2.1 Alternating Comparison Pattern

The algorithm alternates between two phases:
1. **Odd phase**: Compare pairs (1,2), (3,4), (5,6), ...
2. **Even phase**: Compare pairs (0,1), (2,3), (4,5), ...

This allows all odd or all even comparisons to happen simultaneously in parallel.

### 2.2 Convergence

After n phases (for n elements), the array is guaranteed to be sorted.

## 3. Algorithm Description

### 3.1 Pseudocode

```
ODD_EVEN_SORT(A)
    n ← length(A)
    sorted ← false
    
    while not sorted do
        sorted ← true
        
        // Odd phase: compare (1,2), (3,4), (5,6), ...
        for i ← 1 to n - 2 step 2 do
            if A[i] > A[i + 1] then
                swap(A[i], A[i + 1])
                sorted ← false
        
        // Even phase: compare (0,1), (2,3), (4,5), ...
        for i ← 0 to n - 2 step 2 do
            if A[i] > A[i + 1] then
                swap(A[i], A[i + 1])
                sorted ← false
```

### 3.2 Step-by-Step Example

Sorting `[5, 3, 2, 8, 1, 4]`:

```
Initial: [5, 3, 2, 8, 1, 4]
         (0)(1)(2)(3)(4)(5)

Pass 1:
  Odd phase (1-2, 3-4):
    Compare (3,2): swap → [5, 2, 3, 8, 1, 4]
    Compare (8,1): swap → [5, 2, 3, 1, 8, 4]
  
  Even phase (0-1, 2-3, 4-5):
    Compare (5,2): swap → [2, 5, 3, 1, 8, 4]
    Compare (3,1): swap → [2, 5, 1, 3, 8, 4]
    Compare (8,4): swap → [2, 5, 1, 3, 4, 8]

Pass 2:
  Odd phase:
    Compare (5,1): swap → [2, 1, 5, 3, 4, 8]
    Compare (3,4): no swap
  
  Even phase:
    Compare (2,1): swap → [1, 2, 5, 3, 4, 8]
    Compare (5,3): swap → [1, 2, 3, 5, 4, 8]
    Compare (4,8): no swap

Pass 3:
  Odd phase:
    Compare (2,3): no swap
    Compare (5,4): swap → [1, 2, 3, 4, 5, 8]
  
  Even phase:
    All pairs in order → sorted!

Final: [1, 2, 3, 4, 5, 8]
```

## 4. Complexity Analysis

| Case | Sequential | Parallel |
|------|------------|----------|
| **Best** | O(n) | O(n) |
| **Average** | O(n²) | O(n) |
| **Worst** | O(n²) | O(n) |
| **Space** | O(1) | O(1) |

### 4.1 Parallel Speedup

With n/2 processors:
- Each phase: O(1) parallel time
- Number of phases: O(n)
- Total parallel time: O(n)

## 5. Implementation

```rust
pub fn odd_even_sort<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    if n <= 1 {
        return;
    }

    let mut sorted = false;

    while !sorted {
        sorted = true;

        // Odd phase
        for i in (1..n - 1).step_by(2) {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                sorted = false;
            }
        }

        // Even phase
        for i in (0..n - 1).step_by(2) {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                sorted = false;
            }
        }
    }
}
```

### 5.1 Parallel Implementation (with Rayon)

```rust
use rayon::prelude::*;

pub fn odd_even_sort_parallel<T: Ord + Send>(arr: &mut [T]) {
    let n = arr.len();
    
    for _ in 0..n {
        // Odd phase - parallel
        arr.par_chunks_mut(2)
            .skip(1)  // Start from index 1
            .for_each(|chunk| {
                if chunk.len() == 2 && chunk[0] > chunk[1] {
                    chunk.swap(0, 1);
                }
            });
        
        // Even phase - parallel
        arr.par_chunks_mut(2)
            .for_each(|chunk| {
                if chunk.len() == 2 && chunk[0] > chunk[1] {
                    chunk.swap(0, 1);
                }
            });
    }
}
```

## 6. Comparison with Related Algorithms

| Algorithm | Sequential | Parallel | In-place |
|-----------|------------|----------|----------|
| Bubble Sort | O(n²) | O(n²) | Yes |
| Odd-Even Sort | O(n²) | O(n) | Yes |
| Bitonic Sort | O(n log² n) | O(log² n) | Yes |

## 7. Applications

1. **Parallel computing**: SIMD/MIMD architectures
2. **Sorting networks**: Hardware implementation
3. **Distributed systems**: When communication is expensive
4. **Teaching**: Parallel algorithm concepts

## 8. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| Simple parallelization | Independent comparisons per phase |
| In-place | No extra memory needed |
| Stable | Preserves equal element order |
| Predictable | Fixed number of phases |

### Limitations
| Limitation | Description |
|------------|-------------|
| O(n²) sequential | Not efficient without parallelism |
| Fixed passes | Cannot terminate early efficiently |
| Synchronization | Requires barrier between phases |

## 9. Sorting Network Representation

```
Inputs:  a     b     c     d     e     f
         │     │     │     │     │     │
Odd:     │     ├─────┤     ├─────┤     │
         │     │     │     │     │     │
Even:    ├─────┤     ├─────┤     ├─────┤
         │     │     │     │     │     │
Odd:     │     ├─────┤     ├─────┤     │
         │     │     │     │     │     │
... (repeat n times)
         │     │     │     │     │     │
Outputs: 1     2     3     4     5     6
```

## 10. References

1. Habermann, N. (1972). "Parallel Neighbor Sort (or the Glory of the Induction Principle)".
2. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*.

## 11. Source Code

**Implementation**: [src/sorting/odd_even_sort.rs](../../src/sorting/odd_even_sort.rs)
