# Comb Sort

## 1. Overview

Comb Sort is an improvement over Bubble Sort that uses a gap larger than 1. The gap starts with a large value and shrinks by a factor (shrink factor) in each iteration until it reaches 1. It eliminates "turtles" - small values near the end of the list.

### Key Characteristics
- **Type**: Comparison-based, exchange sort
- **In-place**: Yes
- **Stable**: No
- **Shrink Factor**: 1.3 (optimal empirically determined)

## 2. Mathematical Foundation

### 2.1 Shrink Factor

The optimal shrink factor was empirically determined to be approximately **1.3**:
- Gap decreases as: $gap_k = \lfloor gap_{k-1} / 1.3 \rfloor$
- Initial gap: $gap_0 = n$

### 2.2 Turtle Problem

In Bubble Sort, small elements at the end ("turtles") move slowly:
- Each pass moves a turtle only one position left
- Large gaps in Comb Sort move turtles quickly

## 3. Algorithm Description

### 3.1 Pseudocode

```
COMB_SORT(A)
    n ← length(A)
    gap ← n
    shrink ← 1.3
    sorted ← false
    
    while not sorted do
        gap ← floor(gap / shrink)
        if gap ≤ 1 then
            gap ← 1
            sorted ← true
        
        for i ← 0 to n - gap - 1 do
            if A[i] > A[i + gap] then
                swap(A[i], A[i + gap])
                sorted ← false
```

### 3.2 Step-by-Step Example

Sorting `[8, 4, 1, 56, 3, -44, 23, -6, 28, 0]`:

```
Initial: [8, 4, 1, 56, 3, -44, 23, -6, 28, 0]
n = 10, initial gap = 10

Gap = 7 (10/1.3):
Compare (8,-6), (4,28), (1,0)
Result: [-6, 4, 0, 56, 3, -44, 23, 8, 28, 1]

Gap = 5:
Compare pairs at distance 5
Result: [-44, 4, 0, 56, 3, -6, 23, 8, 28, 1]

... continue until gap = 1
Final: [-44, -6, 0, 1, 3, 4, 8, 23, 28, 56]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Best** | O(n log n) |
| **Average** | O(n²/2^p) where p = number of increments |
| **Worst** | O(n²) |
| **Space** | O(1) |

## 5. Implementation

```rust
pub fn comb_sort<T: Ord>(arr: &mut [T]) {
    let mut gap = arr.len();
    let shrink = 1.3;
    let mut sorted = false;

    while !sorted {
        gap = (gap as f64 / shrink).floor() as usize;
        if gap <= 1 {
            gap = 1;
            sorted = true;
        }

        for i in 0..arr.len() - gap {
            if arr[i] > arr[i + gap] {
                arr.swap(i, i + gap);
                sorted = false;
            }
        }
    }
}
```

## 6. Comparison with Bubble Sort

| Aspect | Bubble Sort | Comb Sort |
|--------|-------------|-----------|
| Average Case | O(n²) | O(n²/2^p) |
| Gap | Always 1 | Shrinking |
| Turtle Problem | Severe | Eliminated |
| Passes Needed | ~n | ~log n |

## 7. References

1. Lacey, S.; Box, R. (1991). "A Fast, Easy Sort". *Byte Magazine*.
2. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*.

## 8. Source Code

**Implementation**: [src/sorting/comb_sort.rs](../../src/sorting/comb_sort.rs)
