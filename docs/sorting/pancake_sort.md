# Pancake Sort

## 1. Overview

Pancake Sort is a sorting algorithm that sorts a sequence by a series of "pancake flips". A flip reverses the order of the first k elements. The goal is to sort the array using only this flip operation.

### Key Characteristics
- **Type**: Comparison-based, reversal sort
- **In-place**: Yes
- **Stable**: No
- **Unique**: Only uses prefix reversals

## 2. Mathematical Foundation

### 2.1 The Pancake Problem

Given a stack of n pancakes of different sizes, sort them using only spatula flips (prefix reversals).

**Pancake Number P(n)**: The minimum number of flips required to sort any stack of n pancakes.

Known bounds:
- Lower bound: $\frac{15n}{14}$
- Upper bound: $\frac{18n}{11}$
- Exact values: P(1)=0, P(2)=1, P(3)=3, P(4)=4, ...

### 2.2 Algorithm Strategy

1. Find the maximum element
2. Flip it to the top (if not already there)
3. Flip the entire array to move it to the bottom
4. Repeat for the remaining unsorted portion

## 3. Algorithm Description

### 3.1 Pseudocode

```
PANCAKE_SORT(A)
    n ← length(A)
    
    for current_size ← n down to 2 do
        // Find index of maximum in unsorted portion
        max_idx ← 0
        for i ← 0 to current_size - 1 do
            if A[i] > A[max_idx] then
                max_idx ← i
        
        if max_idx ≠ current_size - 1 then
            // Flip maximum to top (if not already there)
            if max_idx > 0 then
                FLIP(A, 0, max_idx)
            
            // Flip maximum to its final position
            FLIP(A, 0, current_size - 1)

FLIP(A, start, end)
    while start < end do
        swap(A[start], A[end])
        start ← start + 1
        end ← end - 1
```

### 3.2 Step-by-Step Example

Sorting `[3, 2, 4, 1]`:

```
Initial: [3, 2, 4, 1]
current_size = 4

Step 1: Find max (4 at index 2)
  Flip[0..2]: [4, 2, 3, 1]
  Flip[0..3]: [1, 3, 2, 4]  ← 4 is now in place

Step 2: current_size = 3
  Find max in [1, 3, 2]: 3 at index 1
  Flip[0..1]: [3, 1, 2, 4]
  Flip[0..2]: [2, 1, 3, 4]  ← 3 is now in place

Step 3: current_size = 2
  Find max in [2, 1]: 2 at index 0
  Already at top, so just flip
  Flip[0..1]: [1, 2, 3, 4]  ← 2 is now in place

Final: [1, 2, 3, 4]
Total flips: 5
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Time** | O(n²) |
| **Space** | O(1) |
| **Flips** | At most 2n - 3 |

### 4.1 Flip Count Analysis

- Each element (except minimum) needs at most 2 flips
- Total: at most $2(n-1) - 1 = 2n - 3$ flips
- Best case: 0 flips (already sorted)

## 5. Implementation

```rust
fn flip<T>(arr: &mut [T], k: usize) {
    let mut start = 0;
    let mut end = k;
    while start < end {
        arr.swap(start, end);
        start += 1;
        end -= 1;
    }
}

pub fn pancake_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    if len < 2 {
        return;
    }

    for current_size in (1..len).rev() {
        // Find index of maximum element
        let max_idx = arr[..=current_size]
            .iter()
            .enumerate()
            .max_by(|(_, a), (_, b)| a.cmp(b))
            .map(|(i, _)| i)
            .unwrap();

        if max_idx != current_size {
            // Flip max to top if not already there
            if max_idx > 0 {
                flip(arr, max_idx);
            }
            // Flip max to its final position
            flip(arr, current_size);
        }
    }
}
```

## 6. Variants

### 6.1 Burnt Pancake Problem

Each pancake has a "burnt" side that should face down:
- Need to sort by size AND orientation
- Requires more flips

### 6.2 Signed Permutation

Related to genome rearrangement in computational biology.

## 7. Applications

- **Parallel computing**: Prefix reversal is useful for certain parallel architectures
- **Computational biology**: Genome rearrangement analysis
- **Theoretical CS**: Combinatorial optimization problems

## 8. References

1. Gates, W.; Papadimitriou, C. (1979). "Bounds for Sorting by Prefix Reversal". *Discrete Mathematics*.
2. Chitturi, B. et al. (2009). "An (18/11)n Upper Bound for Sorting by Prefix Reversals".

## 9. Source Code

**Implementation**: [src/sorting/pancake_sort.rs](../../src/sorting/pancake_sort.rs)
