# Quick Sort

## 1. Overview

Quick Sort is a highly efficient, comparison-based, divide-and-conquer sorting algorithm. Developed by Tony Hoare in 1959, it remains one of the most widely used sorting algorithms due to its excellent average-case performance and in-place sorting capability.

### Key Characteristics
- **Type**: Comparison-based, divide-and-conquer
- **In-place**: Yes (with O(log n) stack space)
- **Stable**: No
- **Adaptive**: Partially (performance varies with input)

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ of $n$ comparable elements, rearrange the elements such that:
$$A[0] \leq A[1] \leq A[2] \leq \cdots \leq A[n-1]$$

### 2.2 Mathematical Model

**Input**: Array $A$ of $n$ elements with a total ordering relation $\leq$

**Output**: Permutation $A'$ of $A$ such that $A'[i] \leq A'[j]$ for all $0 \leq i < j < n$

**Recurrence Relation** (for time complexity):
$$T(n) = T(k) + T(n-k-1) + \Theta(n)$$

Where $k$ is the number of elements smaller than the pivot.

### 2.3 Correctness Proof

**Loop Invariant**: After partitioning around pivot $p$:
1. All elements in $A[lo..i-1]$ are $\leq p$
2. All elements in $A[i+1..hi]$ are $\geq p$
3. $A[i] = p$

**Proof by Induction**:
- **Base case**: Array of size 0 or 1 is trivially sorted
- **Inductive step**: Assuming sub-arrays are correctly sorted, the concatenation of sorted left partition + pivot + sorted right partition produces a sorted array

## 3. Algorithm Description

### 3.1 Intuition

Quick Sort works by:
1. Selecting a "pivot" element from the array
2. Partitioning other elements into two sub-arrays:
   - Elements less than the pivot
   - Elements greater than the pivot
3. Recursively sorting the sub-arrays

The key insight is that after partitioning, the pivot is in its final sorted position.

### 3.2 Pseudocode

```
QUICKSORT(A, lo, hi)
    if lo < hi then
        pivot_index ← PARTITION(A, lo, hi)
        QUICKSORT(A, lo, pivot_index - 1)
        QUICKSORT(A, pivot_index + 1, hi)

PARTITION(A, lo, hi)
    pivot ← A[hi]           // Choose last element as pivot
    i ← lo
    j ← hi - 1
    
    while true do
        while A[i] < pivot do
            i ← i + 1
        while j > lo and A[j] > pivot do
            j ← j - 1
        if i ≥ j then
            break
        swap A[i] and A[j]
    
    swap A[i] and A[hi]     // Place pivot in correct position
    return i
```

### 3.3 Step-by-Step Example

Sorting array `[10, 8, 4, 3, 1, 9, 2, 7, 5, 6]`:

```
Initial: [10, 8, 4, 3, 1, 9, 2, 7, 5, 6]
                                      ^ pivot = 6

Step 1: Partition around 6
        [5, 2, 4, 3, 1, 6, 9, 7, 8, 10]
                       ^ pivot in final position

Step 2: Recursively sort [5, 2, 4, 3, 1] with pivot 1
        [1, 2, 4, 3, 5]
         ^

Step 3: Recursively sort [2, 4, 3, 5] with pivot 5
        [2, 4, 3, 5]
                  ^

... continue until fully sorted

Final:  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Condition |
|------|------------|-----------|
| **Best** | $O(n \log n)$ | Pivot always divides array in half |
| **Average** | $O(n \log n)$ | Random pivot selection |
| **Worst** | $O(n^2)$ | Already sorted or reverse sorted (with last-element pivot) |

**Derivation of Average Case**:

For a random pivot, the expected number of comparisons:
$$C(n) = n - 1 + \frac{1}{n}\sum_{k=0}^{n-1}(C(k) + C(n-1-k))$$

Solving this recurrence:
$$C(n) \approx 2n \ln n \approx 1.39n \log_2 n$$

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | $O(\log n)$ | Stack space for recursion (average) |
| **Worst case** | $O(n)$ | Degenerate partitioning |

The implementation uses tail-call optimization to ensure O(log n) space by always recursing on the smaller partition first.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn quick_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    if len > 1 {
        _quick_sort(arr, 0, len - 1);
    }
}
```

**Key Implementation Details**:

1. **Generic over `Ord` trait**: Works with any totally ordered type
2. **In-place sorting**: Uses `swap` for element exchange
3. **Tail-call optimization**: Recurses on smaller partition first
4. **Bounds checking**: Handles empty and single-element arrays

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Returns immediately |
| Single element | Returns immediately |
| Already sorted | Works but may be O(n²) with last-pivot |
| All equal elements | Handled by the equal-element swap logic |
| Two elements | Single comparison and potential swap |

### 5.3 Potential Pitfalls

1. **Worst-case on sorted data**: Use randomized pivot or median-of-three
2. **Stack overflow on large arrays**: Tail-call optimization prevents this
3. **Not stable**: Equal elements may be reordered

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Standard Library Implementations**
   - C++ `std::sort` uses Introsort (Quick Sort + Heap Sort + Insertion Sort)
   - Java's `Arrays.sort()` for primitives

2. **Database Systems**
   - In-memory sorting for query results
   - External merge sort preprocessing

3. **File Systems**
   - Directory listing sorting
   - File search result ordering

4. **Graphics and Gaming**
   - Z-buffer sorting for rendering
   - Spatial partitioning (BSP trees)

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Merge Sort](merge_sort.md) | Also divide-and-conquer | Guaranteed O(n log n), stable |
| [Heap Sort](heap_sort.md) | Also O(n log n) worst case | Memory constrained |
| [Intro Sort](intro_sort.md) | Hybrid with Quick Sort | General purpose |
| [Quick Sort 3-Way](quick_sort_3_ways.md) | Variant of Quick Sort | Many duplicate keys |

### 6.3 Comparison with Alternatives

```
Quick Sort vs Merge Sort:
┌────────────────┬────────────┬────────────┐
│ Criterion      │ Quick Sort │ Merge Sort │
├────────────────┼────────────┼────────────┤
│ Average Time   │ O(n log n) │ O(n log n) │
│ Worst Time     │ O(n²)      │ O(n log n) │
│ Space          │ O(log n)   │ O(n)       │
│ Stable         │ No         │ Yes        │
│ Cache-friendly │ Yes        │ Less so    │
└────────────────┴────────────┴────────────┘
```

## 7. Optimizations

### 7.1 Pivot Selection Strategies

1. **Last element** (current implementation): Simple but vulnerable to sorted input
2. **Median-of-three**: Choose median of first, middle, last elements
3. **Random**: Provides probabilistic guarantees
4. **Ninther**: Median of medians for large arrays

### 7.2 Small Array Optimization

Switch to insertion sort for small subarrays (typically n < 10-20):
```rust
if hi - lo < INSERTION_THRESHOLD {
    insertion_sort(&mut arr[lo..=hi]);
    return;
}
```

### 7.3 Three-Way Partitioning

For arrays with many duplicate keys, use [Dutch National Flag partitioning](dutch_national_flag_sort.md) (see [Quick Sort 3-Way](quick_sort_3_ways.md)).

## 8. References

1. Hoare, C. A. R. (1962). "Quicksort". *The Computer Journal*. 5 (1): 10–16.
2. Sedgewick, R. (1978). "Implementing Quicksort programs". *Communications of the ACM*. 21 (10): 847–857.
3. Bentley, J. L.; McIlroy, M. D. (1993). "Engineering a sort function". *Software: Practice and Experience*. 23 (11): 1249–1265.
4. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 7.

## 9. Source Code

**Implementation**: [src/sorting/quick_sort.rs](../../src/sorting/quick_sort.rs)

```rust
pub fn partition<T: PartialOrd>(arr: &mut [T], lo: usize, hi: usize) -> usize {
    let pivot = hi;
    let mut i = lo;
    let mut j = hi - 1;

    loop {
        while arr[i] < arr[pivot] {
            i += 1;
        }
        while j > 0 && arr[j] > arr[pivot] {
            j -= 1;
        }
        if j == 0 || i >= j {
            break;
        } else if arr[i] == arr[j] {
            i += 1;
            j -= 1;
        } else {
            arr.swap(i, j);
        }
    }
    arr.swap(i, pivot);
    i
}

pub fn quick_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    if len > 1 {
        _quick_sort(arr, 0, len - 1);
    }
}
```
