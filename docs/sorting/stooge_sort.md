# Stooge Sort

## 1. Overview

Stooge Sort is a recursive sorting algorithm with a remarkably bad time complexity of O(n^{log₃ 2.7}) ≈ O(n^{2.71}). It works by recursively sorting the first 2/3 and last 2/3 of the array. Named after The Three Stooges, it's primarily used for educational purposes.

### Key Characteristics
- **Type**: Comparison-based, recursive
- **In-place**: Yes
- **Stable**: No
- **Practical**: No (worse than O(n²))

## 2. Mathematical Foundation

### 2.1 Recurrence Relation

The algorithm divides the problem into three overlapping subproblems of size 2n/3:

$$T(n) = 3T(2n/3) + O(1)$$

Using the Master Theorem:
- $a = 3$, $b = 3/2$
- $\log_b a = \log_{1.5} 3 \approx 2.71$

Therefore: $T(n) = O(n^{\log_{1.5} 3}) = O(n^{2.7095...})$

### 2.2 Why It Works

The algorithm ensures:
1. After sorting first 2/3: smallest element is in first 1/3
2. After sorting last 2/3: largest element is in last 1/3
3. After sorting first 2/3 again: middle elements are ordered correctly

## 3. Algorithm Description

### 3.1 Pseudocode

```
STOOGE_SORT(A, low, high)
    if A[low] > A[high] then
        swap(A[low], A[high])
    
    if high - low + 1 > 2 then
        t ← (high - low + 1) / 3
        
        STOOGE_SORT(A, low, high - t)      // First 2/3
        STOOGE_SORT(A, low + t, high)      // Last 2/3
        STOOGE_SORT(A, low, high - t)      // First 2/3 again
```

### 3.2 Step-by-Step Example

Sorting `[2, 4, 5, 3, 1]`:

```
Initial: [2, 4, 5, 3, 1]
n = 5, t = 5/3 = 1

Level 0: STOOGE(0, 4)
  Compare A[0]=2 and A[4]=1: swap → [1, 4, 5, 3, 2]
  
  Level 1: STOOGE(0, 3) - first 2/3
    [1, 4, 5, 3] → ... → [1, 3, 4, 5]
  
  Level 1: STOOGE(1, 4) - last 2/3
    [3, 4, 5, 2] → ... → [2, 3, 4, 5]
  
  Level 1: STOOGE(0, 3) - first 2/3 again
    [1, 2, 3, 4] → ... → [1, 2, 3, 4]

Final: [1, 2, 3, 4, 5]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Time** | O(n^{2.71}) |
| **Space** | O(log n) - recursion stack |
| **Comparisons** | O(n^{2.71}) |

### 4.1 Comparison Table

| Algorithm | Time Complexity |
|-----------|----------------|
| Bubble Sort | O(n²) |
| Stooge Sort | O(n^{2.71}) |
| Slow Sort | O(n^{log n}) |
| Bogo Sort | O(n·n!) |

## 5. Implementation

```rust
pub fn stooge_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    if len < 2 {
        return;
    }
    _stooge_sort(arr, 0, len - 1);
}

fn _stooge_sort<T: Ord>(arr: &mut [T], low: usize, high: usize) {
    if arr[low] > arr[high] {
        arr.swap(low, high);
    }

    if high - low + 1 > 2 {
        let t = (high - low + 1) / 3;
        
        _stooge_sort(arr, low, high - t);      // First 2/3
        _stooge_sort(arr, low + t, high);      // Last 2/3
        _stooge_sort(arr, low, high - t);      // First 2/3 again
    }
}
```

## 6. Proof of Correctness

**Base Case**: Arrays of size 1 or 2 are correctly sorted by the initial comparison and swap.

**Inductive Step**: For arrays of size n > 2:
1. After first recursive call on [0, 2n/3): The minimum of [0, 2n/3) is in [0, n/3)
2. After second recursive call on [n/3, n): The maximum of [n/3, n) is in [2n/3, n)
3. After third recursive call on [0, 2n/3): The entire array is sorted

## 7. Educational Value

Stooge Sort demonstrates:
1. **Not all recursive algorithms are efficient**: Despite elegant structure
2. **Overlapping subproblems can be harmful**: Unlike dynamic programming
3. **Recurrence analysis**: Non-trivial Master Theorem application
4. **Correctness vs. efficiency**: Algorithm is correct but impractical

## 8. References

1. Cormen, T. H. (2009). *Introduction to Algorithms* (Problem 7-3).
2. "Stooge sort". *Wikipedia*.

## 9. Source Code

**Implementation**: [src/sorting/stooge_sort.rs](../../src/sorting/stooge_sort.rs)
