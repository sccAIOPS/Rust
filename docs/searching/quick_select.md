# Quick Select

## 1. Overview

Quick Select (also known as Hoare's selection algorithm) is a selection algorithm to find the k-th smallest element in an unordered list. It is related to QuickSort—using the same partitioning approach—but only recurses into one side of the partition, giving it linear average-case complexity.

Developed by Tony Hoare in 1961 (the same inventor of QuickSort), Quick Select is the algorithm of choice when you need to find order statistics without fully sorting the data.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ of $n$ elements and an integer $k$ (where $1 \leq k \leq n$), find the element that would be at position $k-1$ if the array were sorted (the k-th smallest element).

### 2.2 Mathematical Model

**Partitioning:**

Given pivot value $p$, partition array into:
- $A_L$: Elements < $p$
- $A_E$: Elements = $p$
- $A_R$: Elements > $p$

**Selection Logic:**
- If $k \leq |A_L|$: k-th smallest is in $A_L$
- If $k \leq |A_L| + |A_E|$: k-th smallest is $p$
- Otherwise: k-th smallest is $(k - |A_L| - |A_E|)$-th smallest in $A_R$

### 2.3 Expected Complexity Analysis

Let $T(n)$ be expected comparisons for array of size $n$.

With random pivot, probability of pivot landing at position $i$ is $1/n$.

$$T(n) = n - 1 + \frac{1}{n} \sum_{i=0}^{n-1} T(\max(i, n-1-i))$$

Solving this recurrence yields $T(n) = O(n)$.

**Intuition:** On average, each partition eliminates a constant fraction of elements.

## 3. Algorithm Description

### 3.1 Intuition

Imagine finding the median salary in a company:
1. Pick a random employee's salary as a "pivot"
2. Split employees into "earns less" and "earns more" groups
3. If "earns less" has exactly k-1 people, pivot is the k-th lowest
4. Otherwise, search only the relevant group

Unlike sorting, we only care about one side!

### 3.2 Pseudocode

```
QUICK-SELECT(A, left, right, k):
    if left = right:
        return A[left]
    
    // Partition around a pivot
    pivot_index ← PARTITION(A, left, right)
    
    // Position of pivot in sorted array
    if k = pivot_index:
        return A[k]
    else if k < pivot_index:
        return QUICK-SELECT(A, left, pivot_index - 1, k)
    else:
        return QUICK-SELECT(A, pivot_index + 1, right, k)

PARTITION(A, left, right, pivot_index):
    pivot_value ← A[pivot_index]
    SWAP(A[pivot_index], A[right])  // Move pivot to end
    store_index ← left
    
    for i ← left to right - 1:
        if A[i] < pivot_value:
            SWAP(A[store_index], A[i])
            store_index ← store_index + 1
    
    SWAP(A[right], A[store_index])  // Move pivot to final place
    return store_index
```

### 3.3 Step-by-Step Example

**Array:** `[3, 2, 1, 5, 4]`  
**Find:** 3rd smallest (k = 2, 0-indexed)

**Step 1: First partition (pivot_index = 2, pivot = 1)**

| i | A[i] | Compare | Action |
|---|------|---------|--------|
| - | - | Swap 1 to end | `[3, 2, 4, 5, 1]` |
| 0 | 3 | 3 < 1? No | - |
| 1 | 2 | 2 < 1? No | - |
| 2 | 4 | 4 < 1? No | - |
| 3 | 5 | 5 < 1? No | - |
| - | - | Swap pivot back | `[1, 2, 4, 5, 3]` |

pivot_index = 0, but we need k = 2, so search right side.

**Step 2: Partition [2, 4, 5, 3] (indices 1-4), pivot_index = 2 (value = 4)**

After partitioning: `[1, 2, 3, 4, 5]`
pivot_index = 3

We need k = 2, search left side [1, 2, 3]

**Step 3: Partition [2, 3] (indices 1-2), pivot = 2**

After: `[1, 2, 3, 4, 5]`
pivot_index = 1

We need k = 2, search right of pivot = index 2

**Result:** A[2] = 3 ✓ (3rd smallest)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | When |
|------|------------|------|
| Best | $O(n)$ | Pivot is exactly at k |
| Average | $O(n)$ | Random pivot selection |
| Worst | $O(n^2)$ | Always worst partition (e.g., sorted array, always pick min/max) |

**Average Case Derivation:**

Each level does $O(n)$ work, but search space shrinks by expected factor of 2:
$$T(n) = n + n/2 + n/4 + ... = 2n = O(n)$$

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Auxiliary (iterative) | $O(1)$ |
| Stack (recursive) | $O(\log n)$ average, $O(n)$ worst |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
fn partition(list: &mut [i32], left: usize, right: usize, pivot_index: usize) -> usize {
    let pivot_value = list[pivot_index];
    list.swap(pivot_index, right); // Move pivot to end
    let mut store_index = left;
    for i in left..right {
        if list[i] < pivot_value {
            list.swap(store_index, i);
            store_index += 1;
        }
    }
    list.swap(right, store_index); // Move pivot to its final place
    store_index
}

pub fn quick_select(list: &mut [i32], left: usize, right: usize, index: usize) -> i32 {
    if left == right {
        return list[left];
    }
    let mut pivot_index = left + (right - left) / 2;
    pivot_index = partition(list, left, right, pivot_index);
    
    match index {
        x if x == pivot_index => list[index],
        x if x < pivot_index => quick_select(list, left, pivot_index - 1, index),
        _ => quick_select(list, pivot_index + 1, right, index),
    }
}
```

**Design Choices:**

1. **In-place mutation:** Modifies input array (common for selection algorithms)
2. **Fixed to `i32`:** Not generic (could be improved)
3. **Middle pivot:** Uses `left + (right - left) / 2` (reasonable default)
4. **0-indexed k:** Returns element at index k (not k-th smallest)

**Potential Improvements:**

```rust
// Generic version
pub fn quick_select_generic<T: Ord + Copy>(
    list: &mut [T], 
    left: usize, 
    right: usize, 
    k: usize
) -> T
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Single element | Returns immediately |
| k = 0 | Returns minimum |
| k = n-1 | Returns maximum |
| All equal elements | Works correctly (pivot never moves) |
| Already sorted | Works but may hit O(n²) with bad pivot |

### 5.3 Pivot Selection Strategies

| Strategy | Quality | Implementation |
|----------|---------|----------------|
| First element | Poor for sorted | `pivot = left` |
| Middle element | Better | `pivot = (left + right) / 2` |
| Random | Good average | `pivot = rand(left, right)` |
| Median of three | Very good | `median(A[left], A[mid], A[right])` |
| Median of medians | Guaranteed O(n) | Complex, rarely needed |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Finding Median:** k = n/2 gives the median
2. **Percentile Calculation:** k = n*p/100 for p-th percentile
3. **Top-K Selection:** Run quick_select(k), then take first k elements
4. **Database Query Optimization:** ORDER BY ... LIMIT k
5. **Machine Learning:** Finding threshold values, outlier detection
6. **Image Processing:** Median filtering for noise reduction
7. **Streaming Quantiles:** Maintaining approximate order statistics

### 6.2 Example: Finding Median

```rust
fn find_median(arr: &mut [i32]) -> f64 {
    let n = arr.len();
    if n % 2 == 1 {
        quick_select(arr, 0, n - 1, n / 2) as f64
    } else {
        let a = quick_select(arr, 0, n - 1, n / 2 - 1);
        let b = quick_select(arr, 0, n - 1, n / 2);
        (a + b) as f64 / 2.0
    }
}
```

## 7. Comparison with Alternatives

| Method | Time | Space | In-Place | Notes |
|--------|------|-------|----------|-------|
| Full Sort | $O(n \log n)$ | $O(\log n)$–$O(n)$ | Depends | Overkill for single k |
| Quick Select | $O(n)$ avg | $O(\log n)$ | ✓ | Best for single k |
| Heap-based | $O(n \log k)$ | $O(k)$ | ✗ | Better for small k |
| Counting Sort | $O(n + r)$ | $O(r)$ | ✗ | Only for integers in range |

## 8. Variants

### 8.1 IntroSelect (Hybrid)

Falls back to median-of-medians after too many bad partitions:

```rust
fn introselect(arr: &mut [i32], k: usize, max_depth: usize) -> i32 {
    if max_depth == 0 {
        // Fall back to guaranteed O(n) algorithm
        median_of_medians_select(arr, k)
    } else {
        // Normal quick select with depth tracking
        quick_select_with_depth(arr, k, max_depth - 1)
    }
}
```

### 8.2 Iterative Quick Select

```rust
pub fn quick_select_iterative(list: &mut [i32], mut k: usize) -> i32 {
    let mut left = 0;
    let mut right = list.len() - 1;
    
    while left < right {
        let pivot_index = partition(list, left, right, left + (right - left) / 2);
        if k == pivot_index {
            return list[k];
        } else if k < pivot_index {
            right = pivot_index - 1;
        } else {
            left = pivot_index + 1;
        }
    }
    list[left]
}
```

### 8.3 Median of Medians (Guaranteed Linear)

Achieves worst-case $O(n)$ by choosing a guaranteed good pivot:

1. Divide array into groups of 5
2. Find median of each group
3. Recursively find median of medians
4. Use this as pivot

This guarantees at least 30% of elements are eliminated each partition.

## 9. References

1. Hoare, C. A. R. (1961). "Algorithm 65: Find." Communications of the ACM.
2. Blum, M., et al. (1973). "Time bounds for selection." Journal of Computer and System Sciences.
3. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Chapter 9.
4. Musser, D. R. (1997). "Introspective Sorting and Selection Algorithms." Software: Practice and Experience.

---

**Implementation:** [`src/searching/quick_select.rs`](../../src/searching/quick_select.rs)  
**See Also:** [Kth Smallest](kth_smallest.md), [Kth Smallest (Heap)](kth_smallest_heap.md)
