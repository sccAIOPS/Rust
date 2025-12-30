# Intro Sort (Introspective Sort)

## 1. Overview

Intro Sort is a hybrid sorting algorithm that combines Quick Sort, Heap Sort, and Insertion Sort to achieve excellent average-case performance while guaranteeing O(n log n) worst-case complexity. Developed by David Musser in 1997, it is used as the default sorting algorithm in many C++ standard library implementations.

### Key Characteristics
- **Type**: Hybrid (Quick Sort + Heap Sort + Insertion Sort)
- **In-place**: Yes
- **Stable**: No
- **Adaptive**: Partially adaptive through Quick Sort

## 2. Mathematical Foundation

### 2.1 Problem Definition

Design a sorting algorithm that:
- Has O(n log n) average-case performance (like Quick Sort)
- Has O(n log n) worst-case guarantee (like Heap Sort)
- Is efficient for small arrays (like Insertion Sort)

### 2.2 Depth Limit

The key innovation is a **depth limit** that determines when to switch from Quick Sort to Heap Sort:

$$\text{max\_depth} = 2 \times \lfloor \log_2 n \rfloor$$

If recursion depth exceeds this limit, the algorithm switches to Heap Sort.

## 3. Algorithm Description

### 3.1 Intuition

Intro Sort makes intelligent decisions about which algorithm to use:
1. **Small arrays (n ≤ 16)**: Use Insertion Sort
2. **Normal recursion**: Use Quick Sort
3. **Excessive recursion**: Switch to Heap Sort

This avoids Quick Sort's O(n²) worst case while maintaining its excellent average performance.

### 3.2 Pseudocode

```
INTROSORT(A)
    max_depth ← 2 × floor(log₂(length(A)))
    INTROSORT_RECURSIVE(A, 0, length(A) - 1, max_depth)

INTROSORT_RECURSIVE(A, lo, hi, depth_limit)
    length ← hi - lo + 1
    
    if length ≤ 16 then
        INSERTION_SORT(A[lo..hi])
    else if depth_limit = 0 then
        HEAP_SORT(A[lo..hi])
    else
        pivot ← PARTITION(A, lo, hi)
        INTROSORT_RECURSIVE(A, lo, pivot - 1, depth_limit - 1)
        INTROSORT_RECURSIVE(A, pivot + 1, hi, depth_limit - 1)
```

### 3.3 Step-by-Step Example

Sorting array `[67, 34, 29, 15, 21, 9, 99]`:

```
n = 7, max_depth = 2 × floor(log₂(7)) = 2 × 2 = 4

Since n = 7 < 16:
  Use Insertion Sort directly

Result: [9, 15, 21, 29, 34, 67, 99]
```

For larger arrays:
```
Array: [64, 32, 16, 48, 80, 96, 8, 24, 40, 56, 72, 88, 4, 12, 20, 28, 36]
n = 17, max_depth = 2 × 4 = 8

depth=8: Quick Sort partition around pivot
         Continue recursively...
         
If one branch recurses too deep (depth reaches 0):
         Switch to Heap Sort for that subarray
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Algorithm Used |
|------|------------|----------------|
| **Best** | O(n log n) | Quick Sort dominates |
| **Average** | O(n log n) | Quick Sort dominates |
| **Worst** | O(n log n) | Heap Sort fallback |

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | O(log n) | Quick Sort recursion stack |
| **Total** | O(log n) | All three algorithms use O(log n) or less |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
fn insertion_sort<T: Ord>(arr: &mut [T]) {
    for i in 1..arr.len() {
        let mut j = i;
        while j > 0 && arr[j] < arr[j - 1] {
            arr.swap(j, j - 1);
            j -= 1;
        }
    }
}

fn heap_sort<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    // Build max-heap
    for i in (0..n / 2).rev() {
        heapify(arr, n, i);
    }
    // Extract elements
    for i in (0..n).rev() {
        arr.swap(0, i);
        heapify(arr, i, 0);
    }
}

pub fn intro_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    let max_depth = (2.0 * (len as f64).log2()) as usize + 1;
    intro_sort_recursive(arr, max_depth);
}

fn intro_sort_recursive<T: Ord>(arr: &mut [T], max_depth: usize) {
    let len = arr.len();

    if len <= 16 {
        insertion_sort(arr);
    } else if max_depth == 0 {
        heap_sort(arr);
    } else {
        let pivot = partition(arr);
        intro_sort_recursive(&mut arr[..pivot], max_depth - 1);
        intro_sort_recursive(&mut arr[pivot + 1..], max_depth - 1);
    }
}
```

### 5.2 Edge Cases

| Case | Algorithm Used |
|------|----------------|
| Empty array | Returns immediately |
| Single element | Returns immediately |
| n ≤ 16 | Insertion Sort |
| Sorted input | Quick Sort (good pivot selection helps) |
| Degenerate pivot selection | Heap Sort kicks in |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **C++ Standard Library**
   - `std::sort()` in most implementations (GCC, MSVC, Clang)
   
2. **General-Purpose Sorting**
   - When you need reliable O(n log n) without knowing input characteristics
   
3. **System Programming**
   - Kernel sorting routines
   - Database engines

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Quick Sort](quick_sort.md) | Base algorithm | When data is known to be random |
| [Heap Sort](heap_sort.md) | Fallback algorithm | When space is critical |
| [Tim Sort](tim_sort.md) | Alternative hybrid | When stability needed |

## 7. Why Not Just Use Heap Sort Always?

While Heap Sort guarantees O(n log n), Quick Sort is faster in practice due to:
1. Better cache locality
2. Lower constant factors
3. In-place partitioning efficiency

Intro Sort gets the best of both worlds.

## 8. References

1. Musser, D. R. (1997). "Introspective Sorting and Selection Algorithms". *Software: Practice and Experience*. 27 (8): 983–993.
2. ISO/IEC 14882:2017. "Programming Languages — C++".

## 9. Source Code

**Implementation**: [src/sorting/intro_sort.rs](../../src/sorting/intro_sort.rs)

```rust
pub fn intro_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    let max_depth = (2.0 * len as f64).log2() as usize + 1;

    fn intro_sort_recursive<T: Ord>(arr: &mut [T], max_depth: usize) {
        let len = arr.len();

        if len <= 16 {
            insertion_sort(arr);
        } else if max_depth == 0 {
            heap_sort(arr);
        } else {
            let pivot = partition(arr);
            intro_sort_recursive(&mut arr[..pivot], max_depth - 1);
            intro_sort_recursive(&mut arr[pivot + 1..], max_depth - 1);
        }
    }

    intro_sort_recursive(arr, max_depth);
}
```
