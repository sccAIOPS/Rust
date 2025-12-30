# Selection Sort

## 1. Overview

Selection Sort is a simple comparison-based sorting algorithm that divides the input into a sorted and unsorted region. It repeatedly selects the minimum element from the unsorted region and moves it to the end of the sorted region.

### Key Characteristics
- **Type**: Comparison-based, selection-based
- **In-place**: Yes
- **Stable**: No (standard implementation)
- **Adaptive**: No (always O(n²))

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, sort by repeatedly finding the minimum element in the unsorted portion and placing it at the beginning.

### 2.2 Loop Invariant

At iteration $i$:
- $A[0..i-1]$ contains the $i$ smallest elements in sorted order
- $A[i..n-1]$ contains the remaining unsorted elements

### 2.3 Comparison Count

Selection Sort always makes exactly:
$$\text{Comparisons} = \frac{n(n-1)}{2} = \binom{n}{2}$$

regardless of input order.

## 3. Algorithm Description

### 3.1 Intuition

Selection Sort mimics how humans might sort:
1. Scan through all elements to find the smallest
2. Put it in the first position
3. Repeat for the remaining elements

### 3.2 Pseudocode

```
SELECTION_SORT(A)
    n ← length(A)
    for i ← 0 to n - 2 do
        min_index ← i
        for j ← i + 1 to n - 1 do
            if A[j] < A[min_index] then
                min_index ← j
        swap A[i] and A[min_index]
```

### 3.3 Step-by-Step Example

Sorting array `[64, 25, 12, 22, 11]`:

```
Initial: [64, 25, 12, 22, 11]

i=0: Find minimum in [64, 25, 12, 22, 11]
     Minimum: 11 at index 4
     Swap A[0] and A[4]
     [11, 25, 12, 22, 64]
     ^sorted^

i=1: Find minimum in [25, 12, 22, 64]
     Minimum: 12 at index 2
     Swap A[1] and A[2]
     [11, 12, 25, 22, 64]
     ^sorted^

i=2: Find minimum in [25, 22, 64]
     Minimum: 22 at index 3
     Swap A[2] and A[3]
     [11, 12, 22, 25, 64]
     ^sorted^

i=3: Find minimum in [25, 64]
     Minimum: 25 at index 3
     No swap needed (already in place)
     [11, 12, 22, 25, 64]

Final: [11, 12, 22, 25, 64]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Notes |
|------|------------|-------|
| **Best** | O(n²) | Always scans entire unsorted portion |
| **Average** | O(n²) | Same as best |
| **Worst** | O(n²) | Same as best |

**Key insight**: Selection Sort makes the same number of comparisons regardless of input order.

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | O(1) | Only indices and temp for swap |

### 4.3 Swap Count

Selection Sort minimizes swaps:
- **Swaps**: Exactly $n - 1$ (at most)
- **Optimal for write-expensive memory**: Flash memory, EEPROM

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn selection_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    for left in 0..len {
        let mut smallest = left;
        for right in (left + 1)..len {
            if arr[right] < arr[smallest] {
                smallest = right;
            }
        }
        arr.swap(smallest, left);
    }
}
```

**Key Implementation Details**:

1. **`T: Ord`**: Only requires ordering (no Copy needed)
2. **Swap-based**: Uses efficient swap instead of assignments
3. **No early termination**: Always completes all iterations

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Loop doesn't execute |
| Single element | Loop doesn't execute (for 0..0 is empty) |
| Already sorted | Still O(n²) comparisons, 0 swaps |
| All equal | Still O(n²) comparisons |

## 6. Real-World Applications

### 6.1 When to Use Selection Sort

1. **Memory with expensive writes**
   - Flash memory
   - EEPROM
   - Minimizes write cycles

2. **Small arrays**
   - Simple implementation
   - Predictable performance

3. **When stability not required**
   - Simple implementation preferred over stable alternatives

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Bubble Sort](bubble_sort.md) | Both O(n²) | Bubble for nearly sorted |
| [Insertion Sort](insertion_sort.md) | Both simple | Insertion for nearly sorted |
| [Heap Sort](heap_sort.md) | Selection-based | Large arrays |

## 7. Why Selection Sort is Not Stable

```
Example: [(5,'a'), (3,'b'), (5,'c'), (2,'d')]

After finding minimum 2:
[(2,'d'), (3,'b'), (5,'c'), (5,'a')]
                           ^swapped^
                           
Now (5,'a') is after (5,'c'), but originally it was before!
```

### Stable Selection Sort

Can be made stable by shifting instead of swapping:
```rust
fn stable_selection_sort<T: Ord + Clone>(arr: &mut [T]) {
    for i in 0..arr.len() {
        let min_idx = (i..arr.len()).min_by_key(|&j| &arr[j]).unwrap();
        let min_val = arr[min_idx].clone();
        // Shift elements right instead of swapping
        for j in (i..min_idx).rev() {
            arr[j + 1] = arr[j].clone();
        }
        arr[i] = min_val;
    }
}
```

## 8. Comparison with Other O(n²) Algorithms

| Algorithm | Comparisons | Swaps | Adaptive | Stable |
|-----------|-------------|-------|----------|--------|
| Selection Sort | n²/2 | n-1 | No | No |
| [Bubble Sort](bubble_sort.md) | n²/2 | n²/2 | Yes | Yes |
| [Insertion Sort](insertion_sort.md) | n²/4 avg | n²/4 avg | Yes | Yes |

**Selection Sort wins on**: Swap count
**Insertion Sort wins on**: Adaptive behavior, average comparisons

## 9. Variants

### 9.1 Double Selection Sort (Cocktail Selection)

Find both minimum and maximum in each pass:
```rust
fn double_selection_sort<T: Ord>(arr: &mut [T]) {
    let mut left = 0;
    let mut right = arr.len() - 1;
    
    while left < right {
        let (min_idx, max_idx) = find_min_max(arr, left, right);
        // Handle edge cases and swap
        left += 1;
        right -= 1;
    }
}
```

### 9.2 Bingo Sort

Optimized for many duplicate values (see [Bingo Sort](bingo_sort.md)).

## 10. References

1. Knuth, D. E. "The Art of Computer Programming, Vol. 3".
2. Cormen, T. H., et al. "Introduction to Algorithms".
3. Sedgewick, R. "Algorithms in C++".

## 11. Source Code

**Implementation**: [src/sorting/selection_sort.rs](../../src/sorting/selection_sort.rs)

```rust
pub fn selection_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    for left in 0..len {
        let mut smallest = left;
        for right in (left + 1)..len {
            if arr[right] < arr[smallest] {
                smallest = right;
            }
        }
        arr.swap(smallest, left);
    }
}
```
