# Shell Sort

## 1. Overview

Shell Sort is a generalization of insertion sort that allows the exchange of items that are far apart. The algorithm starts by sorting pairs of elements far apart from each other, then progressively reducing the gap between elements to be compared.

### Key Characteristics
- **Type**: Comparison-based, diminishing increment sort
- **In-place**: Yes
- **Stable**: No
- **Adaptive**: Partially

## 2. Mathematical Foundation

### 2.1 Gap Sequences

The performance of Shell Sort depends heavily on the gap sequence used:

| Sequence | Formula | Complexity |
|----------|---------|------------|
| Shell (1959) | $n/2^k$ | O(n²) |
| Hibbard (1963) | $2^k - 1$ | O(n^{3/2}) |
| Sedgewick (1986) | $4^k + 3 \cdot 2^{k-1} + 1$ | O(n^{4/3}) |
| Tokuda (1992) | $\lceil(9(9/4)^k - 4)/5\rceil$ | Unknown |

## 3. Algorithm Description

### 3.1 Pseudocode

```
SHELL_SORT(A)
    n ← length(A)
    gap ← n / 2
    
    while gap > 0 do
        for i ← gap to n - 1 do
            temp ← A[i]
            j ← i
            while j ≥ gap and A[j - gap] > temp do
                A[j] ← A[j - gap]
                j ← j - gap
            A[j] ← temp
        gap ← gap / 2
```

### 3.2 Step-by-Step Example

Sorting `[35, 33, 42, 10, 14, 19, 27, 44]`:

```
Initial: [35, 33, 42, 10, 14, 19, 27, 44]

Gap = 4:
Compare pairs (35,14), (33,19), (42,27), (10,44)
Result: [14, 19, 27, 10, 35, 33, 42, 44]

Gap = 2:
Compare pairs at distance 2
Result: [14, 10, 27, 19, 35, 33, 42, 44]

Gap = 1:
Standard insertion sort
Result: [10, 14, 19, 27, 33, 35, 42, 44]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Best** | O(n log n) |
| **Average** | O(n^{1.25}) to O(n^{1.5}) |
| **Worst** | O(n²) (Shell's sequence) |

## 5. Implementation

```rust
pub fn shell_sort<T: Ord + Copy>(values: &mut [T]) {
    fn insertion<T: Ord + Copy>(values: &mut [T], start: usize, gap: usize) {
        for i in ((start + gap)..values.len()).step_by(gap) {
            let val_current = values[i];
            let mut pos = i;
            while pos >= gap && values[pos - gap] > val_current {
                values[pos] = values[pos - gap];
                pos -= gap;
            }
            values[pos] = val_current;
        }
    }

    let mut count_sublist = values.len() / 2;
    while count_sublist > 0 {
        for pos_start in 0..count_sublist {
            insertion(values, pos_start, count_sublist);
        }
        count_sublist /= 2;
    }
}
```

## 6. References

1. Shell, D. L. (1959). "A High-Speed Sorting Procedure". *Communications of the ACM*.
2. Sedgewick, R. (1996). "Analysis of Shellsort and Related Algorithms".

## 7. Source Code

**Implementation**: [src/sorting/shell_sort.rs](../../src/sorting/shell_sort.rs)
