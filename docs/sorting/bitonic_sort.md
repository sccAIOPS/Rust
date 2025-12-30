# Bitonic Sort

## 1. Overview

Bitonic Sort is a parallel sorting algorithm that works by first creating a bitonic sequence (a sequence that first monotonically increases, then decreases, or vice versa) and then repeatedly merging to produce a sorted sequence. It's designed for parallel processing and sorting networks.

### Key Characteristics
- **Type**: Comparison-based, parallel sort
- **In-place**: Yes
- **Stable**: No
- **Parallel**: Highly parallelizable
- **Constraint**: Array size must be power of 2

## 2. Mathematical Foundation

### 2.1 Bitonic Sequence

A sequence is **bitonic** if:
1. It monotonically increases then decreases, OR
2. It monotonically decreases then increases, OR
3. It can be cyclically rotated to satisfy (1) or (2)

Examples:
- `[1, 3, 5, 7, 6, 4, 2]` - increases then decreases ✓
- `[7, 6, 4, 2, 1, 3, 5]` - decreases then increases ✓
- `[4, 5, 6, 7, 3, 2, 1]` - bitonic (rotate to get valid sequence) ✓

### 2.2 Bitonic Merge

Key theorem: If we compare-exchange elements at distance d in a bitonic sequence, we get two bitonic sequences (one in each half).

## 3. Algorithm Description

### 3.1 Pseudocode

```
BITONIC_SORT(A, low, count, direction)
    if count > 1 then
        k ← count / 2
        
        // Sort first half ascending
        BITONIC_SORT(A, low, k, ASCENDING)
        
        // Sort second half descending
        BITONIC_SORT(A, low + k, k, DESCENDING)
        
        // Merge the bitonic sequence
        BITONIC_MERGE(A, low, count, direction)

BITONIC_MERGE(A, low, count, direction)
    if count > 1 then
        k ← count / 2
        
        for i ← low to low + k - 1 do
            COMPARE_EXCHANGE(A, i, i + k, direction)
        
        BITONIC_MERGE(A, low, k, direction)
        BITONIC_MERGE(A, low + k, k, direction)

COMPARE_EXCHANGE(A, i, j, direction)
    if direction = ASCENDING then
        if A[i] > A[j] then swap(A[i], A[j])
    else
        if A[i] < A[j] then swap(A[i], A[j])
```

### 3.2 Step-by-Step Example

Sorting `[3, 7, 4, 8, 6, 2, 1, 5]`:

```
Initial: [3, 7, 4, 8, 6, 2, 1, 5]
n = 8 (power of 2) ✓

Step 1: Create bitonic sequences of size 2
  [3,7]↑ [4,8]↓ [6,2]↑ [1,5]↓
  → [3,7] [8,4] [2,6] [5,1]

Step 2: Create bitonic sequences of size 4
  [3,7,8,4]↑ [2,6,5,1]↓
  Merge ascending: [3,4,7,8]
  Merge descending: [6,5,2,1]
  → [3,4,7,8,6,5,2,1]

Step 3: Merge entire bitonic sequence (ascending)
  Compare pairs at distance 4: (3,6),(4,5),(7,2),(8,1)
  → [3,4,2,1,6,5,7,8]
  
  Recurse on halves...
  
Final: [1, 2, 3, 4, 5, 6, 7, 8]
```

## 4. Complexity Analysis

| Case | Sequential | Parallel |
|------|------------|----------|
| **Time** | O(n log² n) | O(log² n) |
| **Space** | O(1) or O(n) | O(n) |
| **Comparisons** | O(n log² n) | - |

### 4.1 Parallel Efficiency

- Depth (parallel time): O(log² n)
- Work (total comparisons): O(n log² n)
- Can use n/2 processors effectively

## 5. Implementation

```rust
pub fn bitonic_sort<T: Ord>(arr: &mut [T], ascending: bool) {
    let n = arr.len();
    if n <= 1 {
        return;
    }

    // Ensure power of 2 (pad if necessary in real impl)
    assert!(n.is_power_of_two(), "Length must be power of 2");

    bitonic_sort_recursive(arr, 0, n, ascending);
}

fn bitonic_sort_recursive<T: Ord>(
    arr: &mut [T],
    low: usize,
    count: usize,
    ascending: bool,
) {
    if count > 1 {
        let k = count / 2;

        // Sort first half ascending, second half descending
        bitonic_sort_recursive(arr, low, k, true);
        bitonic_sort_recursive(arr, low + k, k, false);

        // Merge the bitonic sequence
        bitonic_merge(arr, low, count, ascending);
    }
}

fn bitonic_merge<T: Ord>(
    arr: &mut [T],
    low: usize,
    count: usize,
    ascending: bool,
) {
    if count > 1 {
        let k = count / 2;

        for i in low..(low + k) {
            compare_and_swap(arr, i, i + k, ascending);
        }

        bitonic_merge(arr, low, k, ascending);
        bitonic_merge(arr, low + k, k, ascending);
    }
}

fn compare_and_swap<T: Ord>(arr: &mut [T], i: usize, j: usize, ascending: bool) {
    if ascending == (arr[i] > arr[j]) {
        arr.swap(i, j);
    }
}
```

## 6. Sorting Networks

Bitonic Sort can be represented as a sorting network:

```
Inputs:  a  b  c  d  e  f  g  h
         │  │  │  │  │  │  │  │
Level 1: ├──┤  ├──┤  ├──┤  ├──┤
         │  │  │  │  │  │  │  │
Level 2: ├─────┤  ├─────┤
         │  │  │  │  │  │  │  │
         ... (more levels)
         │  │  │  │  │  │  │  │
Outputs: 1  2  3  4  5  6  7  8
```

Each `├──┤` represents a comparator.

## 7. Applications

1. **GPU Sorting**: Excellent for CUDA/OpenCL implementations
2. **FPGA**: Hardware sorting implementations
3. **Parallel databases**: Distributed sorting
4. **Sorting networks**: Hardware design

## 8. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| Highly parallel | O(log² n) parallel time |
| Fixed pattern | Same comparisons regardless of data |
| Hardware-friendly | Easy to implement in circuits |

### Limitations
| Limitation | Description |
|------------|-------------|
| Power of 2 | Requires padding for other sizes |
| Not adaptive | Same work for all inputs |
| O(n log² n) | Worse than O(n log n) algorithms |

## 9. References

1. Batcher, K. E. (1968). "Sorting Networks and Their Applications".
2. Cormen, T. H. (2009). *Introduction to Algorithms*, Chapter 27.

## 10. Source Code

**Implementation**: [src/sorting/bitonic_sort.rs](../../src/sorting/bitonic_sort.rs)
