# Insertion Sort

## 1. Overview

Insertion Sort is a simple, intuitive comparison-based sorting algorithm that builds the final sorted array one element at a time. It is highly efficient for small datasets and nearly sorted arrays, and serves as the foundation for more complex algorithms like Tim Sort and Shell Sort.

### Key Characteristics
- **Type**: Comparison-based, incremental
- **In-place**: Yes
- **Stable**: Yes
- **Adaptive**: Yes (O(n) for nearly sorted data)

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, sort by maintaining a sorted prefix and repeatedly inserting the next element into its correct position.

### 2.2 Loop Invariant

At the start of each iteration $i$:
- $A[0..i-1]$ contains the first $i$ elements of the original array
- $A[0..i-1]$ is sorted

### 2.3 Inversions

The number of swaps equals the number of **inversions** (pairs $(i, j)$ where $i < j$ but $A[i] > A[j]$).

For a random array: Expected inversions = $\frac{n(n-1)}{4}$

## 3. Algorithm Description

### 3.1 Intuition

Think of sorting playing cards in your hand:
1. Start with one card (trivially sorted)
2. Pick the next card
3. Insert it into the correct position among the sorted cards
4. Repeat until all cards are sorted

### 3.2 Pseudocode

```
INSERTION_SORT(A)
    for i ← 1 to length(A) - 1 do
        key ← A[i]
        j ← i - 1
        
        // Shift elements greater than key
        while j ≥ 0 and A[j] > key do
            A[j + 1] ← A[j]
            j ← j - 1
        
        A[j + 1] ← key
```

### 3.3 Step-by-Step Example

Sorting array `[5, 2, 4, 6, 1, 3]`:

```
Initial: [5, 2, 4, 6, 1, 3]
         ^sorted^ ^unsorted^

i=1, key=2:
  [5, 2, 4, 6, 1, 3]  → [5, 5, 4, 6, 1, 3] → [2, 5, 4, 6, 1, 3]
   ^                       shift 5            insert 2

i=2, key=4:
  [2, 5, 4, 6, 1, 3]  → [2, 5, 5, 6, 1, 3] → [2, 4, 5, 6, 1, 3]
      ^                    shift 5            insert 4

i=3, key=6:
  [2, 4, 5, 6, 1, 3]  → No shifts needed (6 > 5)
         ^

i=4, key=1:
  [2, 4, 5, 6, 1, 3]  → shift all → [1, 2, 4, 5, 6, 3]
            ^

i=5, key=3:
  [1, 2, 4, 5, 6, 3]  → [1, 2, 3, 4, 5, 6]
               ^

Final: [1, 2, 3, 4, 5, 6]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Condition |
|------|------------|-----------|
| **Best** | O(n) | Already sorted |
| **Average** | O(n²) | Random order |
| **Worst** | O(n²) | Reverse sorted |

**Comparison count**:
- Best: $n - 1$
- Worst: $\frac{n(n-1)}{2}$

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | O(1) | Only one temporary variable |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn insertion_sort<T: Ord + Copy>(arr: &mut [T]) {
    for i in 1..arr.len() {
        let mut j = i;
        let cur = arr[i];

        while j > 0 && cur < arr[j - 1] {
            arr[j] = arr[j - 1];
            j -= 1;
        }

        arr[j] = cur;
    }
}
```

**Key Implementation Details**:

1. **`T: Ord + Copy`**: Elements must be orderable and copyable
2. **Shift optimization**: Shifts elements instead of swapping (fewer memory operations)
3. **Single temp variable**: Only one element stored temporarily

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Loop doesn't execute |
| Single element | Loop doesn't execute |
| Already sorted | O(n) - only comparisons, no shifts |
| All equal | O(n) - no shifts needed |

## 6. Real-World Applications

### 6.1 When to Use Insertion Sort

1. **Small arrays (n ≤ 10-50)**
   - Lower overhead than divide-and-conquer algorithms
   - Used as base case in Quick Sort, Merge Sort, Tim Sort

2. **Nearly sorted data**
   - O(n) when only a few elements are out of place
   - Real-time data that's incrementally updated

3. **Online sorting**
   - Elements arrive one at a time
   - Maintaining a sorted list during insertions

4. **Memory-constrained environments**
   - Only O(1) extra space needed

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Binary Insertion Sort](binary_insertion_sort.md) | Uses binary search for position | Reduce comparisons |
| [Shell Sort](shell_sort.md) | Generalized gap-based insertion | Larger arrays |
| [Tim Sort](tim_sort.md) | Uses insertion for small runs | Standard library sort |

## 7. Variants

### 7.1 Binary Insertion Sort

Use binary search to find insertion position:
- Comparisons: O(n log n)
- Shifts: Still O(n²)

### 7.2 Shell Sort

Insertion sort with decreasing gap sequences:
- Starts with large gaps, reducing to 1
- Enables long-range element movement

### 7.3 Library Sort (Gapped Insertion Sort)

Leaves gaps in the array for faster insertions.

## 8. Stability

Insertion Sort is **stable** because:
- Elements are shifted only when strictly greater
- Equal elements maintain their relative order

```
Example: [(2,'a'), (3,'b'), (2,'c')]
After sort: [(2,'a'), (2,'c'), (3,'b')]
             ^'a' stays before 'c'^
```

## 9. References

1. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 2.
2. Knuth, D. E. "The Art of Computer Programming, Vol. 3".
3. Sedgewick, R. "Algorithms in C++".

## 10. Source Code

**Implementation**: [src/sorting/insertion_sort.rs](../../src/sorting/insertion_sort.rs)

```rust
pub fn insertion_sort<T: Ord + Copy>(arr: &mut [T]) {
    for i in 1..arr.len() {
        let mut j = i;
        let cur = arr[i];

        while j > 0 && cur < arr[j - 1] {
            arr[j] = arr[j - 1];
            j -= 1;
        }

        arr[j] = cur;
    }
}
```
