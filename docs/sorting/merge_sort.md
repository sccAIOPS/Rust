# Merge Sort

## 1. Overview

Merge Sort is a stable, comparison-based, divide-and-conquer sorting algorithm. Invented by John von Neumann in 1945, it guarantees O(n log n) time complexity in all cases, making it highly predictable and suitable for applications requiring consistent performance.

### Key Characteristics
- **Type**: Comparison-based, divide-and-conquer
- **In-place**: No (requires O(n) auxiliary space)
- **Stable**: Yes
- **Adaptive**: No (same performance regardless of input order)

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ of $n$ comparable elements, produce a sorted permutation while preserving the relative order of equal elements (stability).

### 2.2 Mathematical Model

**Divide-and-Conquer Paradigm**:
1. **Divide**: Split array into two halves
2. **Conquer**: Recursively sort each half
3. **Combine**: Merge sorted halves

**Recurrence Relation**:
$$T(n) = 2T(n/2) + \Theta(n)$$

By the Master Theorem (Case 2): $T(n) = \Theta(n \log n)$

### 2.3 Correctness Proof

**Merge Correctness**:
- **Invariant**: At each step of merging, the next smallest unprocessed element is correctly placed
- **Termination**: Loop terminates when all elements are processed (finite input)

**Sort Correctness** (by strong induction):
- **Base case**: Array of size ≤ 1 is sorted
- **Inductive step**: If merge correctly combines two sorted arrays, and recursive calls correctly sort sub-arrays, then the result is sorted

## 3. Algorithm Description

### 3.1 Intuition

Merge Sort works by:
1. Recursively dividing the array into smaller subarrays until each contains one element
2. Merging adjacent sorted subarrays by comparing their elements
3. The merge operation is the key—it combines two sorted arrays into one sorted array

The brilliance lies in the merge step: comparing the front elements of two sorted arrays always yields the minimum.

### 3.2 Pseudocode

**Top-Down (Recursive)**:
```
MERGE_SORT(A, left, right)
    if left < right then
        mid ← (left + right) / 2
        MERGE_SORT(A, left, mid)
        MERGE_SORT(A, mid + 1, right)
        MERGE(A, left, mid, right)

MERGE(A, left, mid, right)
    L ← A[left..mid]     // Copy left half
    R ← A[mid+1..right]  // Copy right half
    i ← 0, j ← 0, k ← left
    
    while i < |L| and j < |R| do
        if L[i] ≤ R[j] then
            A[k] ← L[i]
            i ← i + 1
        else
            A[k] ← R[j]
            j ← j + 1
        k ← k + 1
    
    // Copy remaining elements
    while i < |L| do
        A[k] ← L[i]
        i ← i + 1
        k ← k + 1
    while j < |R| do
        A[k] ← R[j]
        j ← j + 1
        k ← k + 1
```

**Bottom-Up (Iterative)**:
```
BOTTOM_UP_MERGE_SORT(A)
    n ← length(A)
    size ← 1
    while size < n do
        left ← 0
        while left < n - size do
            mid ← left + size - 1
            right ← min(left + 2*size - 1, n - 1)
            MERGE(A, left, mid, right)
            left ← left + 2*size
        size ← size * 2
```

### 3.3 Step-by-Step Example

Sorting array `[38, 27, 43, 3, 9, 82, 10]`:

```
Level 0: [38, 27, 43, 3, 9, 82, 10]
                    │
         ┌──────────┴──────────┐
Level 1: [38, 27, 43, 3]    [9, 82, 10]
              │                  │
         ┌────┴────┐        ┌────┴────┐
Level 2: [38, 27] [43, 3]   [9, 82] [10]
            │        │         │      │
         ┌──┴──┐  ┌──┴──┐   ┌──┴──┐   │
Level 3: [38][27][43][3]   [9][82]  [10]

Merge back up:
Level 2: [27, 38] [3, 43]   [9, 82] [10]
Level 1: [3, 27, 38, 43]    [9, 10, 82]
Level 0: [3, 9, 10, 27, 38, 43, 82]
```

**Merge operation detail** `[27, 38]` + `[3, 43]`:
```
L: [27, 38]    R: [3, 43]    Result: []
    ^              ^

Compare 27 and 3: 3 < 27
L: [27, 38]    R: [3, 43]    Result: [3]
    ^                 ^

Compare 27 and 43: 27 < 43
L: [27, 38]    R: [3, 43]    Result: [3, 27]
        ^             ^

Compare 38 and 43: 38 < 43
L: [27, 38]    R: [3, 43]    Result: [3, 27, 38]
           ^          ^

L exhausted, copy remaining R:
Result: [3, 27, 38, 43]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Notes |
|------|------------|-------|
| **Best** | $O(n \log n)$ | Always divides and merges |
| **Average** | $O(n \log n)$ | Same as best |
| **Worst** | $O(n \log n)$ | Same as best |

**Derivation**:
- Dividing takes $O(1)$ per level
- $\log n$ levels of recursion
- Each level processes all $n$ elements during merge
- Total: $n \times \log n = O(n \log n)$

**Exact comparison count**: $n \log_2 n - n + 1$ (nearly optimal)

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | $O(n)$ | Temporary arrays for merging |
| **Stack** | $O(\log n)$ | Recursion depth |
| **Total** | $O(n)$ | Dominated by auxiliary space |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
fn merge<T: Ord + Copy>(arr: &mut [T], mid: usize) {
    // Create temporary vectors to support the merge.
    let left_half = arr[..mid].to_vec();
    let right_half = arr[mid..].to_vec();

    let mut l = 0;
    let mut r = 0;

    for v in arr {
        if r == right_half.len() || (l < left_half.len() && left_half[l] < right_half[r]) {
            *v = left_half[l];
            l += 1;
        } else {
            *v = right_half[r];
            r += 1;
        }
    }
}

pub fn top_down_merge_sort<T: Ord + Copy>(arr: &mut [T]) {
    if arr.len() > 1 {
        let mid = arr.len() / 2;
        top_down_merge_sort(&mut arr[..mid]);
        top_down_merge_sort(&mut arr[mid..]);
        merge(arr, mid);
    }
}
```

**Key Implementation Details**:

1. **`T: Ord + Copy`**: Elements must be orderable and copyable
2. **Slice-based recursion**: Uses Rust's slice syntax for clean subdivision
3. **In-place iteration**: Iterates over mutable slice directly

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Returns immediately |
| Single element | Returns immediately |
| Two elements | Single comparison and potential swap |
| All equal | Works correctly, maintains stability |
| Already sorted | Still O(n log n), but cache-friendly |

### 5.3 Top-Down vs Bottom-Up

| Aspect | Top-Down | Bottom-Up |
|--------|----------|-----------|
| Implementation | Recursive | Iterative |
| Cache behavior | Potentially worse | Better locality |
| Stack usage | O(log n) | O(1) |
| Simplicity | More intuitive | More complex indexing |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **External Sorting**
   - Sorting data larger than memory
   - Database external merge sort
   - Hadoop/MapReduce shuffle phase

2. **Linked List Sorting**
   - Merge sort is ideal for linked lists (O(1) merge space)
   - No random access needed

3. **Stable Sorting Requirements**
   - Sorting by multiple keys
   - Preserving original order for equal elements

4. **Parallel Processing**
   - Natural parallelization (independent subproblems)
   - Used in parallel merge sort implementations

5. **Inversion Counting**
   - Count inversions in O(n log n) using merge sort

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Quick Sort](quick_sort.md) | Also divide-and-conquer | In-memory, not stable needed |
| [Tim Sort](tim_sort.md) | Hybrid with merge sort | Real-world data with runs |
| [Heap Sort](heap_sort.md) | Also O(n log n) guaranteed | Memory constrained |
| [Insertion Sort](insertion_sort.md) | Used in optimized merge sort | Small subarrays |

### 6.3 Merge Sort Variants

1. **Natural Merge Sort**: Exploits existing runs in input
2. **Timsort**: Adaptive merge sort used in Python/Java
3. **Parallel Merge Sort**: Divides work across processors
4. **Polyphase Merge Sort**: For tape-based external sorting

## 7. Optimizations

### 7.1 Insertion Sort for Small Arrays

```rust
const CUTOFF: usize = 7;
if arr.len() <= CUTOFF {
    insertion_sort(arr);
    return;
}
```

### 7.2 Skip Merge if Already Sorted

```rust
// After sorting both halves
if arr[mid - 1] <= arr[mid] {
    return; // Already in order, skip merge
}
```

### 7.3 Alternating Auxiliary Arrays

Avoid copying by alternating which array is source/destination:

```rust
fn sort_to(src: &[T], dst: &mut [T]) {
    // Sort from src to dst
}
```

### 7.4 Natural Merge Sort

```rust
fn find_run<T: Ord>(arr: &[T]) -> usize {
    // Find length of ascending or descending run
    let mut i = 1;
    while i < arr.len() && arr[i-1] <= arr[i] {
        i += 1;
    }
    i
}
```

## 8. Stability Analysis

Merge sort is **stable** because:
1. When elements are equal, we take from the left array first
2. This preserves the original relative order

```rust
// Stability: use <= not < for left array preference
if left_half[l] <= right_half[r] {  // Take from left when equal
    *v = left_half[l];
    l += 1;
} else {
    *v = right_half[r];
    r += 1;
}
```

## 9. References

1. von Neumann, J. (1945). "First Draft of a Report on the EDVAC".
2. Knuth, D. E. (1998). "The Art of Computer Programming, Vol. 3: Sorting and Searching".
3. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 2.
4. Sedgewick, R. (1998). "Algorithms in C++".

## 10. Source Code

**Implementation**: [src/sorting/merge_sort.rs](../../src/sorting/merge_sort.rs)

```rust
pub fn top_down_merge_sort<T: Ord + Copy>(arr: &mut [T]) {
    if arr.len() > 1 {
        let mid = arr.len() / 2;
        top_down_merge_sort(&mut arr[..mid]);
        top_down_merge_sort(&mut arr[mid..]);
        merge(arr, mid);
    }
}

pub fn bottom_up_merge_sort<T: Copy + Ord>(a: &mut [T]) {
    if a.len() > 1 {
        let len: usize = a.len();
        let mut sub_array_size: usize = 1;
        while sub_array_size < len {
            let mut start_index: usize = 0;
            while len - start_index > sub_array_size {
                let end_idx = std::cmp::min(start_index + 2 * sub_array_size, len);
                merge(&mut a[start_index..end_idx], sub_array_size);
                start_index = end_idx;
            }
            sub_array_size *= 2;
        }
    }
}
```
