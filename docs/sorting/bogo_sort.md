# Bogo Sort

## 1. Overview

Bogo Sort (also known as Permutation Sort, Stupid Sort, or Monkey Sort) is a highly inefficient sorting algorithm based on randomly shuffling the array until it happens to be sorted. It serves as an example of what NOT to do and is used for educational purposes.

### Key Characteristics
- **Type**: Random permutation sort
- **In-place**: Yes
- **Stable**: No
- **Practical**: Absolutely not!

## 2. Mathematical Foundation

### 2.1 Probability Analysis

Given n elements:
- Total permutations: $n!$
- Sorted permutations: $1$
- Probability of sorted after one shuffle: $\frac{1}{n!}$

Expected number of shuffles to sort: $n!$

### 2.2 Expected Comparisons

- Each check requires $n - 1$ comparisons
- Expected total comparisons: $(n-1) \cdot n! = O(n \cdot n!)$

## 3. Algorithm Description

### 3.1 Pseudocode

```
BOGO_SORT(A)
    while not IS_SORTED(A) do
        SHUFFLE(A)

IS_SORTED(A)
    for i ← 0 to length(A) - 2 do
        if A[i] > A[i + 1] then
            return false
    return true

SHUFFLE(A)
    for i ← length(A) - 1 down to 1 do
        j ← random(0, i)
        swap(A[i], A[j])
```

### 3.2 Example Run

Sorting `[3, 1, 2]`:

```
Initial: [3, 1, 2] - Not sorted
Shuffle → [1, 3, 2] - Not sorted
Shuffle → [2, 1, 3] - Not sorted
Shuffle → [3, 2, 1] - Not sorted
Shuffle → [2, 3, 1] - Not sorted
... (potentially many more shuffles)
Shuffle → [1, 2, 3] - Sorted! ✓
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Best** | O(n) - already sorted |
| **Average** | O((n+1)!) |
| **Worst** | O(∞) - unbounded |
| **Space** | O(1) |

### 4.1 Comparison with Other Algorithms

| Algorithm | Average Time |
|-----------|-------------|
| Bogo Sort | O((n+1)!) |
| Bubble Sort | O(n²) |
| Quick Sort | O(n log n) |

For n = 10:
- Quick Sort: ~33 comparisons
- Bogo Sort: ~36,288,000 expected shuffles

## 5. Implementation

```rust
use rand::seq::SliceRandom;
use rand::thread_rng;

pub fn bogo_sort<T: Ord>(arr: &mut [T]) {
    let mut rng = thread_rng();
    
    while !is_sorted(arr) {
        arr.shuffle(&mut rng);
    }
}

fn is_sorted<T: Ord>(arr: &[T]) -> bool {
    arr.windows(2).all(|w| w[0] <= w[1])
}
```

### 5.1 Deterministic Variant (Permutation Sort)

```rust
// Generates all permutations and finds the sorted one
// Still O(n!) but guaranteed to terminate
pub fn permutation_sort<T: Ord + Clone>(arr: &mut [T]) {
    let sorted = {
        let mut v = arr.to_vec();
        v.sort();
        v
    };
    
    // Generate permutations until we find sorted one
    // (This is just for illustration - actual implementation
    // would use Heap's algorithm or similar)
}
```

## 6. Variants

### 6.1 Bogo Bogo Sort
Sort recursively using bogo sort on smaller portions - even worse!

### 6.2 Quantum Bogo Sort
Theoretical: Create superposition of all permutations, collapse to sorted state.

### 6.3 Bogus Sort
Check if sorted; if not, randomly swap two elements.

## 7. Educational Value

Bogo Sort teaches:
1. **Algorithm analysis**: Understanding best/average/worst cases
2. **Probability in algorithms**: Expected running time
3. **What makes algorithms efficient**: Contrast with good algorithms
4. **Randomized algorithms**: Not all random algorithms are bad (cf. QuickSort)

## 8. When to "Use" (Never)

- **Demonstration**: Teaching algorithm complexity
- **Humor**: Programming jokes
- **Never**: Any real application

## 9. References

1. Gruber, H.; Holzer, M.; Ruepp, O. (2007). "Sorting the Slow Way: An Analysis of Perversely Awful Randomized Sorting Algorithms".
2. *The Jargon File*: Definition of "bogosort".

## 10. Source Code

**Implementation**: [src/sorting/bogo_sort.rs](../../src/sorting/bogo_sort.rs)
