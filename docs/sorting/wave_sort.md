# Wave Sort

## 1. Overview

Wave Sort arranges array elements in a wave-like pattern where elements at even indices are greater than or equal to their adjacent elements. The pattern looks like: `a[0] >= a[1] <= a[2] >= a[3] <= a[4] ...`

### Key Characteristics
- **Type**: Rearrangement algorithm
- **In-place**: Yes
- **Stable**: No
- **Output**: Not fully sorted, but wave-like pattern

## 2. Mathematical Foundation

### 2.1 Wave Pattern Definition

For array A of length n, wave pattern satisfies:
- $A[0] \geq A[1]$
- $A[1] \leq A[2]$
- $A[2] \geq A[3]$
- ...

General rule:
- If i is even: $A[i] \geq A[i-1]$ and $A[i] \geq A[i+1]$
- If i is odd: $A[i] \leq A[i-1]$ and $A[i] \leq A[i+1]$

### 2.2 Non-Uniqueness

Multiple valid wave arrangements exist for the same array:
- `[3, 1, 2]` and `[2, 1, 3]` are both valid for `{1, 2, 3}`

## 3. Algorithm Description

### 3.1 Method 1: Sort and Swap (Simple)

```
WAVE_SORT_SIMPLE(A)
    sort(A)
    for i ← 0 to length(A) - 2 step 2 do
        swap(A[i], A[i + 1])
```

### 3.2 Method 2: Linear Time (Optimal)

```
WAVE_SORT_LINEAR(A)
    n ← length(A)
    
    for i ← 0 to n - 1 step 2 do
        // Check left neighbor
        if i > 0 and A[i] < A[i - 1] then
            swap(A[i], A[i - 1])
        
        // Check right neighbor
        if i < n - 1 and A[i] < A[i + 1] then
            swap(A[i], A[i + 1])
```

### 3.3 Step-by-Step Example

Sorting `[1, 2, 3, 4, 5, 6]` into wave form:

**Method 1 (Sort + Swap):**
```
After sort: [1, 2, 3, 4, 5, 6]
Swap pairs: (1,2), (3,4), (5,6)
Result: [2, 1, 4, 3, 6, 5]
         ↑  ↓  ↑  ↓  ↑  ↓   (wave pattern)
```

**Method 2 (Linear):**
```
Initial: [1, 2, 3, 4, 5, 6]

i = 0: Check A[0]=1 vs A[1]=2
       1 < 2, swap → [2, 1, 3, 4, 5, 6]

i = 2: Check A[2]=3 vs A[1]=1 → OK (3 > 1)
       Check A[2]=3 vs A[3]=4
       3 < 4, swap → [2, 1, 4, 3, 5, 6]

i = 4: Check A[4]=5 vs A[3]=3 → OK (5 > 3)
       Check A[4]=5 vs A[5]=6
       5 < 6, swap → [2, 1, 4, 3, 6, 5]

Result: [2, 1, 4, 3, 6, 5]
```

## 4. Complexity Analysis

| Method | Time | Space |
|--------|------|-------|
| Sort + Swap | O(n log n) | O(1) or O(n) |
| Linear | O(n) | O(1) |

## 5. Implementation

```rust
pub fn wave_sort<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    
    for i in (0..n).step_by(2) {
        // Check left neighbor
        if i > 0 && arr[i] < arr[i - 1] {
            arr.swap(i, i - 1);
        }
        
        // Check right neighbor
        if i + 1 < n && arr[i] < arr[i + 1] {
            arr.swap(i, i + 1);
        }
    }
}
```

### 5.1 Verification Function

```rust
pub fn is_wave_sorted<T: Ord>(arr: &[T]) -> bool {
    for i in (0..arr.len()).step_by(2) {
        if i > 0 && arr[i] < arr[i - 1] {
            return false;
        }
        if i + 1 < arr.len() && arr[i] < arr[i + 1] {
            return false;
        }
    }
    true
}
```

## 6. Comparison with Similar Algorithms

| Algorithm | Output Pattern | Time |
|-----------|---------------|------|
| Wave Sort | a ≥ b ≤ c ≥ d | O(n) |
| Wiggle Sort | a < b > c < d | O(n) |
| Regular Sort | a ≤ b ≤ c ≤ d | O(n log n) |

## 7. Applications

1. **Signal processing**: Creating alternating patterns
2. **Graphics**: Generating wave-like visual effects
3. **Game development**: Terrain generation
4. **Data visualization**: Creating visual interest

## 8. Variations

### 8.1 Inverted Wave
Pattern: `a[0] <= a[1] >= a[2] <= a[3] ...`

```rust
pub fn inverted_wave_sort<T: Ord>(arr: &mut [T]) {
    for i in (1..arr.len()).step_by(2) {
        if i > 0 && arr[i] < arr[i - 1] {
            arr.swap(i, i - 1);
        }
        if i + 1 < arr.len() && arr[i] < arr[i + 1] {
            arr.swap(i, i + 1);
        }
    }
}
```

### 8.2 Strict Wave
Pattern with strict inequalities: `a[0] > a[1] < a[2] > a[3] ...`
(Only possible if all elements are distinct)

## 9. References

1. GeeksforGeeks. "Sort an array in wave form".
2. Programming interview problems and solutions.

## 10. Source Code

**Implementation**: [src/sorting/wave_sort.rs](../../src/sorting/wave_sort.rs)
