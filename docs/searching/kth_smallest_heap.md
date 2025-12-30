# Kth Smallest Element (Heap-based)

## 1. Overview

The heap-based approach to finding the k-th smallest element maintains a max-heap of size k containing the k smallest elements seen so far. Unlike the partition-based approach, this method doesn't mutate the input array and is particularly suited for streaming data or when the original array must be preserved.

This algorithm achieves $O(n \log k)$ time complexity, which is better than $O(n \log n)$ sorting when $k << n$, and offers consistent performance regardless of input distribution.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ and integer $k$ (where $1 \leq k \leq n$), find the k-th smallest element without modifying the input array.

### 2.2 Heap Property

A **max-heap** is a complete binary tree where each node is greater than or equal to its children.

**Key Insight:** If we maintain a max-heap of the k smallest elements:
- The heap's maximum (root) is the k-th smallest element
- Any element larger than the root cannot be in the k smallest

### 2.3 Algorithm Principle

1. Add first k elements to a max-heap
2. For each remaining element:
   - If element < heap root: replace root with element, heapify
   - Otherwise: element cannot be in k smallest, skip
3. Root of heap is the k-th smallest

**Invariant:** The heap always contains the k smallest elements seen so far.

## 3. Algorithm Description

### 3.1 Intuition

Imagine finding the 3 shortest students in a class of 30:
1. Line up the first 3 students by height (tallest visible)
2. For each remaining student:
   - If shorter than the tallest of the 3: swap them out
   - Otherwise: definitely not in the 3 shortest
3. The tallest of your final 3 is the 3rd shortest overall

### 3.2 Pseudocode

```
KTH-SMALLEST-HEAP(A, k):
    if k > length(A):
        return NOT_FOUND
    
    heap ← new MaxHeap()
    
    // Initialize with first k elements
    for i ← 0 to k-1:
        heap.add(A[i])
    
    // Process remaining elements
    for i ← k to n-1:
        if A[i] < heap.peek():
            heap.pop()
            heap.add(A[i])
    
    return heap.peek()  // Root is k-th smallest
```

### 3.3 Step-by-Step Example

**Array:** `[9, 17, 3, 16, 13, 10, 1, 5, 7, 12, 4, 8, 9, 0]`  
**Find:** 6th smallest (k = 6)

**Phase 1: Build initial heap with first 6 elements**

| Step | Element | Heap (max at root) |
|------|---------|-------------------|
| 1 | 9 | [9] |
| 2 | 17 | [17, 9] |
| 3 | 3 | [17, 9, 3] |
| 4 | 16 | [17, 16, 3, 9] |
| 5 | 13 | [17, 16, 3, 9, 13] |
| 6 | 10 | [17, 16, 10, 9, 13, 3] |

**Heap root = 17** (current k-th smallest candidate)

**Phase 2: Process remaining elements**

| Step | Element | Compare with 17 | Action | New Root |
|------|---------|-----------------|--------|----------|
| 7 | 1 | 1 < 17 | Replace 17 with 1 | 16 |
| 8 | 5 | 5 < 16 | Replace 16 with 5 | 13 |
| 9 | 7 | 7 < 13 | Replace 13 with 7 | 10 |
| 10 | 12 | 12 > 10 | Skip | 10 |
| 11 | 4 | 4 < 10 | Replace 10 with 4 | 9 |
| 12 | 8 | 8 < 9 | Replace 9 with 8 | 7 |
| 13 | 9 | 9 > 7 | Skip | 7 |
| 14 | 0 | 0 < 7 | Replace 7 with 0 | 7 |

**Final heap contains:** `[0, 1, 3, 4, 5, 7]` (the 6 smallest)  
**Heap root = 7** ← This is the 6th smallest ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Phase | Operations | Per Operation | Total |
|-------|------------|---------------|-------|
| Build initial heap | k | $O(\log k)$ | $O(k \log k)$ |
| Process remaining | n - k | $O(\log k)$ | $O((n-k) \log k)$ |
| **Total** | - | - | $O(n \log k)$ |

**Key Insight:** When $k$ is small (e.g., top 10 of millions), this is much faster than sorting.

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Heap storage | $O(k)$ |
| Auxiliary | $O(1)$ |
| **Total** | $O(k)$ |

### 4.3 Comparison

| k Value | Heap-based | Partition-based | Full Sort |
|---------|------------|-----------------|-----------|
| $O(1)$ | $O(n)$ | $O(n)$ | $O(n \log n)$ |
| $O(\log n)$ | $O(n \log \log n)$ | $O(n)$ | $O(n \log n)$ |
| $O(\sqrt{n})$ | $O(n \log \sqrt{n})$ | $O(n)$ | $O(n \log n)$ |
| $O(n)$ | $O(n \log n)$ | $O(n)$ | $O(n \log n)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use crate::data_structures::Heap;
use std::cmp::{Ord, Ordering};

pub fn kth_smallest_heap<T>(input: &[T], k: usize) -> Option<T>
where
    T: Ord + Copy,
{
    if input.len() < k {
        return None;
    }

    let mut heap = Heap::new_max();

    // First k elements go to the heap as baseline
    for &val in input.iter().take(k) {
        heap.add(val);
    }

    // Process remaining elements
    for &val in input.iter().skip(k) {
        let cur_big = heap.pop().unwrap();
        match val.cmp(&cur_big) {
            Ordering::Greater => {
                heap.add(cur_big);  // Keep current k-th smallest
            }
            _ => {
                heap.add(val);  // New element is smaller, include it
            }
        }
    }

    heap.pop()
}
```

**Design Decisions:**

1. **Uses custom `Heap` from data_structures:** Project's own max-heap implementation
2. **Non-mutating:** Takes `&[T]`, doesn't modify input
3. **`Ord + Copy` bounds:** More restrictive than partition version (`PartialOrd`)
4. **Returns `Option<T>`:** Handles k > n gracefully

**Implementation Note:**
The current implementation always pops and re-adds for every element after initial k:
```rust
let cur_big = heap.pop().unwrap();
match val.cmp(&cur_big) { ... }
```

More efficient approach (avoid unnecessary pop for larger elements):
```rust
let cur_big = heap.peek().unwrap();
if val < *cur_big {
    heap.pop();
    heap.add(val);
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty input | Returns `None` |
| k > n | Returns `None` |
| k = 1 | Returns minimum |
| k = n | Returns maximum |
| All equal | Returns that value |
| Single element, k=1 | Returns that element |

### 5.3 Using Standard Library

Alternative using `BinaryHeap`:

```rust
use std::collections::BinaryHeap;

pub fn kth_smallest_std<T: Ord + Copy>(input: &[T], k: usize) -> Option<T> {
    if input.len() < k || k == 0 {
        return None;
    }
    
    let mut heap = BinaryHeap::with_capacity(k);
    
    for &val in input.iter().take(k) {
        heap.push(val);
    }
    
    for &val in input.iter().skip(k) {
        if val < *heap.peek().unwrap() {
            heap.pop();
            heap.push(val);
        }
    }
    
    heap.pop()
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Streaming Data:**
   ```rust
   struct StreamingKth<T> {
       heap: BinaryHeap<T>,
       k: usize,
   }
   
   impl<T: Ord> StreamingKth<T> {
       fn process(&mut self, value: T) {
           if self.heap.len() < self.k {
               self.heap.push(value);
           } else if value < *self.heap.peek().unwrap() {
               self.heap.pop();
               self.heap.push(value);
           }
       }
       
       fn get_kth(&self) -> Option<&T> {
           self.heap.peek()
       }
   }
   ```

2. **Real-time Analytics:**
   - Finding top-k products by sales
   - Identifying k lowest latency servers
   - Streaming percentile calculations

3. **Database Systems:**
   - `ORDER BY ... LIMIT k` optimization
   - Approximate percentile queries

4. **Machine Learning:**
   - K-Nearest Neighbors (finding k closest points)
   - Feature selection (k most important features)

### 6.2 When to Prefer Heap-based Approach

| Scenario | Recommendation |
|----------|---------------|
| Input must not be modified | ✓ Heap-based |
| Streaming data | ✓ Heap-based |
| Very small k | ✓ Heap-based |
| Multiple k values needed | Sort once instead |
| Single query, modifiable input | Partition-based |
| Worst-case guarantee needed | ✓ Heap-based (no O(n²)) |

## 7. Variants

### 7.1 Top-K Elements

Return all k smallest elements (not just the k-th):

```rust
pub fn top_k_smallest<T: Ord + Copy>(input: &[T], k: usize) -> Vec<T> {
    if input.len() < k {
        return input.to_vec();
    }
    
    let mut heap = BinaryHeap::with_capacity(k);
    
    for &val in input {
        if heap.len() < k {
            heap.push(val);
        } else if val < *heap.peek().unwrap() {
            heap.pop();
            heap.push(val);
        }
    }
    
    heap.into_sorted_vec()
}
```

### 7.2 K-th Largest (Min-Heap)

For k-th largest, use a min-heap of size k:

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

pub fn kth_largest<T: Ord + Copy>(input: &[T], k: usize) -> Option<T> {
    let mut heap = BinaryHeap::with_capacity(k);
    
    for &val in input {
        if heap.len() < k {
            heap.push(Reverse(val));
        } else if val > heap.peek().unwrap().0 {
            heap.pop();
            heap.push(Reverse(val));
        }
    }
    
    heap.pop().map(|r| r.0)
}
```

### 7.3 Approximate Streaming Quantiles

For very large streams, use reservoir sampling or t-digest for approximate quantiles.

## 8. Performance Considerations

### 8.1 Cache Behavior

- Heap operations have good cache locality (array-based)
- Better than partition for random access patterns

### 8.2 Practical Performance

For n = 1,000,000:

| k | Heap-based | Partition-based |
|---|------------|-----------------|
| 10 | ~3.3M comparisons | ~2M comparisons |
| 100 | ~6.6M comparisons | ~2M comparisons |
| 10,000 | ~13M comparisons | ~2M comparisons |
| 100,000 | ~17M comparisons | ~2M comparisons |

Heap-based has higher comparison count but predictable performance and no input mutation.

## 9. References

1. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Chapter 6 (Heapsort).
2. Williams, J. W. J. (1964). "Algorithm 232: Heapsort." Communications of the ACM.
3. Munro, J. I., & Paterson, M. S. (1980). "Selection and sorting with limited storage." Theoretical Computer Science.

---

**Implementation:** [`src/searching/kth_smallest_heap.rs`](../../src/searching/kth_smallest_heap.rs)  
**See Also:** [Kth Smallest (Partition)](kth_smallest.md), [Quick Select](quick_select.md)
