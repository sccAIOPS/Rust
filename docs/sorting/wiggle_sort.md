# Wiggle Sort

## 1. Overview

Wiggle Sort rearranges array elements such that they follow an alternating less-than/greater-than pattern: `a[0] < a[1] > a[2] < a[3] > a[4] ...`. It's similar to Wave Sort but with the opposite comparison direction.

### Key Characteristics
- **Type**: Rearrangement algorithm
- **In-place**: Yes
- **Stable**: No
- **Output**: Alternating pattern, not fully sorted

## 2. Mathematical Foundation

### 2.1 Wiggle Pattern Definition

For array A of length n:
- $A[0] < A[1]$
- $A[1] > A[2]$
- $A[2] < A[3]$
- ...

General rule for index i:
- If i is odd: $A[i-1] < A[i] > A[i+1]$ (local maximum)
- If i is even: $A[i-1] > A[i] < A[i+1]$ (local minimum)

### 2.2 Wiggle Sort II Variant

A harder variant where:
- $nums[0] < nums[1] > nums[2] < nums[3]...$
- No two adjacent elements are equal
- Requires median finding for O(n) solution

## 3. Algorithm Description

### 3.1 Pseudocode (Simple Swap Method)

```
WIGGLE_SORT(A)
    n ← length(A)
    
    for i ← 0 to n - 2 do
        if i is even then
            // Even index: should be local minimum
            if A[i] > A[i + 1] then
                swap(A[i], A[i + 1])
        else
            // Odd index: should be local maximum
            if A[i] < A[i + 1] then
                swap(A[i], A[i + 1])
```

### 3.2 Step-by-Step Example

Sorting `[3, 5, 2, 1, 6, 4]` into wiggle form:

```
Initial: [3, 5, 2, 1, 6, 4]

i = 0 (even): Should be < next
  3 < 5? Yes → no swap
  [3, 5, 2, 1, 6, 4]

i = 1 (odd): Should be > next
  5 > 2? Yes → no swap
  [3, 5, 2, 1, 6, 4]

i = 2 (even): Should be < next
  2 < 1? No → swap
  [3, 5, 1, 2, 6, 4]

i = 3 (odd): Should be > next
  2 > 6? No → swap
  [3, 5, 1, 6, 2, 4]

i = 4 (even): Should be < next
  2 < 4? Yes → no swap
  [3, 5, 1, 6, 2, 4]

Result: [3, 5, 1, 6, 2, 4]
Check:   < > < > <    ✓
```

## 4. Complexity Analysis

| Method | Time | Space |
|--------|------|-------|
| Simple Swap | O(n) | O(1) |
| Sort + Interleave | O(n log n) | O(n) |

## 5. Implementation

### 5.1 Simple O(n) Solution

```rust
pub fn wiggle_sort<T: Ord>(arr: &mut [T]) {
    for i in 0..arr.len().saturating_sub(1) {
        if i % 2 == 0 {
            // Even index: should be local minimum
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
            }
        } else {
            // Odd index: should be local maximum
            if arr[i] < arr[i + 1] {
                arr.swap(i, i + 1);
            }
        }
    }
}
```

### 5.2 Verification Function

```rust
pub fn is_wiggle_sorted<T: Ord>(arr: &[T]) -> bool {
    for i in 0..arr.len().saturating_sub(1) {
        if i % 2 == 0 {
            if arr[i] > arr[i + 1] {
                return false;
            }
        } else {
            if arr[i] < arr[i + 1] {
                return false;
            }
        }
    }
    true
}
```

### 5.3 Wiggle Sort II (No Equal Adjacent)

```rust
// For arrays where nums[0] < nums[1] > nums[2] < ...
// with no equal adjacent elements
pub fn wiggle_sort_ii<T: Ord + Clone>(arr: &mut [T]) {
    let mut sorted = arr.to_vec();
    sorted.sort();
    
    let n = arr.len();
    let mid = (n + 1) / 2;
    
    // Interleave smaller and larger halves
    let mut j = mid;
    let mut k = n;
    
    for i in 0..n {
        if i % 2 == 0 {
            j -= 1;
            arr[i] = sorted[j].clone();
        } else {
            k -= 1;
            arr[i] = sorted[k].clone();
        }
    }
}
```

## 6. Proof of Correctness

**Claim**: The simple swap algorithm produces a valid wiggle sequence.

**Proof**:
- At each step i, we ensure the relationship between A[i] and A[i+1]
- When we fix A[i] and A[i+1], we might break A[i-1] and A[i]
- But since we alternate the comparison direction, fixing A[i] vs A[i+1] actually helps maintain A[i-1] vs A[i]

For i even (should be <):
- If we swap because A[i] > A[i+1], then A[i] becomes smaller
- This maintains A[i-1] > A[i] since A[i] got smaller

## 7. Comparison: Wiggle vs Wave

| Aspect | Wiggle Sort | Wave Sort |
|--------|-------------|-----------|
| Pattern | < > < > | ≥ ≤ ≥ ≤ |
| Even indices | Local minima | Local maxima |
| Odd indices | Local maxima | Local minima |
| First element | Smaller | Larger |

## 8. Applications

1. **Data visualization**: Creating zigzag patterns
2. **Signal processing**: Alternating high/low signals
3. **Interview problems**: Common coding challenge
4. **Graphics**: Saw-tooth patterns

## 9. Variations

### 9.1 Descending Wiggle
Pattern: `a[0] > a[1] < a[2] > a[3] ...`

### 9.2 Circular Wiggle
Last element also participates: includes `a[n-1]` vs `a[0]`

## 10. References

1. LeetCode Problem 324: "Wiggle Sort II"
2. LeetCode Problem 280: "Wiggle Sort"
3. Programming Interviews Exposed (2018)

## 11. Source Code

**Implementation**: [src/sorting/wiggle_sort.rs](../../src/sorting/wiggle_sort.rs)
