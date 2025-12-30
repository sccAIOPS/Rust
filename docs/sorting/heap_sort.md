# Heap Sort

## 1. Overview

Heap Sort is a comparison-based sorting algorithm that uses a binary heap data structure. It achieves O(n log n) time complexity in all cases and requires only O(1) auxiliary space, making it an excellent choice for memory-constrained environments.

### Key Characteristics
- **Type**: Comparison-based, selection sort variant
- **In-place**: Yes
- **Stable**: No
- **Adaptive**: No

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, sort it using the heap property to repeatedly extract the maximum (or minimum) element.

### 2.2 Binary Heap Properties

A **binary heap** is a complete binary tree satisfying the heap property:
- **Max-Heap**: Parent ≥ Children: $A[parent(i)] \geq A[i]$
- **Min-Heap**: Parent ≤ Children: $A[parent(i)] \leq A[i]$

**Array Representation**:
For element at index $i$:
- Parent: $\lfloor (i-1)/2 \rfloor$
- Left child: $2i + 1$
- Right child: $2i + 2$

### 2.3 Mathematical Model

**Build Heap Time Complexity**:
$$\sum_{h=0}^{\lfloor \log n \rfloor} \lceil \frac{n}{2^{h+1}} \rceil \cdot O(h) = O(n)$$

**Heap Sort Total**:
- Build heap: $O(n)$
- Extract max $n$ times: $n \times O(\log n) = O(n \log n)$

## 3. Algorithm Description

### 3.1 Intuition

Heap Sort works in two phases:
1. **Build Heap**: Convert the array into a max-heap (for ascending sort)
2. **Extract Max**: Repeatedly extract the maximum element and place it at the end

The beauty is that after building the heap, the maximum is always at the root (index 0), and extraction is efficient.

### 3.2 Pseudocode

```
HEAPSORT(A, ascending)
    n ← length(A)
    
    // Phase 1: Build heap
    BUILD_HEAP(A, ascending)
    
    // Phase 2: Extract elements one by one
    for i ← n-1 down to 1 do
        swap A[0] and A[i]
        HEAPIFY(A[0..i-1], 0, ascending)

BUILD_HEAP(A, is_max_heap)
    n ← length(A)
    // Start from last non-leaf node
    for i ← (n-1)/2 down to 0 do
        HEAPIFY(A, i, is_max_heap)

HEAPIFY(A, i, is_max_heap)
    n ← length(A)
    largest ← i
    left ← 2*i + 1
    right ← 2*i + 2
    
    // Compare with children based on heap type
    if is_max_heap then
        if left < n and A[left] > A[largest] then
            largest ← left
        if right < n and A[right] > A[largest] then
            largest ← right
    else  // min-heap
        if left < n and A[left] < A[largest] then
            largest ← left
        if right < n and A[right] < A[largest] then
            largest ← right
    
    if largest ≠ i then
        swap A[i] and A[largest]
        HEAPIFY(A, largest, is_max_heap)
```

### 3.3 Step-by-Step Example

Sorting array `[4, 10, 3, 5, 1]` (ascending):

```
Initial array as binary tree:
        4
       / \
      10  3
     / \
    5   1

Step 1: Build max-heap (start from last non-leaf)
Index 1 (value 10): No change needed (10 > 5 and 10 > 1)
        4
       / \
      10  3
     / \
    5   1

Index 0 (value 4): 4 < 10, swap
        10
       / \
      4   3
     / \
    5   1

After heapify at index 1: 4 < 5, swap
        10
       / \
      5   3
     / \
    4   1

Max-heap built: [10, 5, 3, 4, 1]

Step 2: Extract max repeatedly
Extract 10, swap with last: [1, 5, 3, 4, 10]
Heapify [1, 5, 3, 4]:
        1           5
       / \   →     / \
      5   3       4   3
     /           /
    4           1
Result: [5, 4, 3, 1, | 10]

Extract 5: [1, 4, 3, | 5, 10]
Heapify: [4, 1, 3, | 5, 10]

Extract 4: [3, 1, | 4, 5, 10]

Extract 3: [1, | 3, 4, 5, 10]

Final: [1, 3, 4, 5, 10]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Notes |
|------|------------|-------|
| **Best** | $O(n \log n)$ | All cases same |
| **Average** | $O(n \log n)$ | All cases same |
| **Worst** | $O(n \log n)$ | Guaranteed |

**Breakdown**:
- Build heap: $O(n)$ (not $O(n \log n)$ as might be expected)
- $n-1$ extract-max operations: $(n-1) \times O(\log n)$
- Total: $O(n) + O(n \log n) = O(n \log n)$

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | $O(1)$ | In-place sorting |
| **Stack** | $O(\log n)$ | Recursive heapify |
| **Iterative** | $O(1)$ | If heapify is iterative |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cmp::Ordering;

fn build_heap<T: Ord>(arr: &mut [T], is_max_heap: bool) {
    let mut i = (arr.len() - 1) / 2;
    while i > 0 {
        heapify(arr, i, is_max_heap);
        i -= 1;
    }
    heapify(arr, 0, is_max_heap);
}

fn heapify<T: Ord>(arr: &mut [T], i: usize, is_max_heap: bool) {
    let comparator: fn(&T, &T) -> Ordering = if is_max_heap {
        |a, b| a.cmp(b)
    } else {
        |a, b| b.cmp(a)
    };

    let mut idx = i;
    let l = 2 * i + 1;
    let r = 2 * i + 2;

    if l < arr.len() && comparator(&arr[l], &arr[idx]) == Ordering::Greater {
        idx = l;
    }
    if r < arr.len() && comparator(&arr[r], &arr[idx]) == Ordering::Greater {
        idx = r;
    }

    if idx != i {
        arr.swap(i, idx);
        heapify(arr, idx, is_max_heap);
    }
}

pub fn heap_sort<T: Ord>(arr: &mut [T], ascending: bool) {
    if arr.len() <= 1 {
        return;
    }

    build_heap(arr, ascending);

    let mut end = arr.len() - 1;
    while end > 0 {
        arr.swap(0, end);
        heapify(&mut arr[..end], 0, ascending);
        end -= 1;
    }
}
```

**Key Implementation Details**:

1. **Bidirectional sorting**: Supports both ascending and descending
2. **Function pointer for comparison**: Clean abstraction for heap type
3. **Slice manipulation**: Uses `&mut arr[..end]` to shrink heap

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Returns immediately |
| Single element | Returns immediately |
| Two elements | Single comparison |
| All equal | Works correctly |
| Already sorted | Still O(n log n) |

### 5.3 Why Heap Sort is Not Stable

```
Example: [(3, 'a'), (1, 'b'), (3, 'c')]

After heap operations, the relative order of (3, 'a') and (3, 'c')
may change because heap operations don't preserve original positions.
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Priority Queues**
   - Task scheduling (OS kernels)
   - Event-driven simulation
   - Dijkstra's shortest path

2. **Selection Problems**
   - Finding k largest/smallest elements
   - Running median calculation
   - Top-k queries

3. **Memory-Constrained Systems**
   - Embedded systems
   - Real-time systems requiring predictable performance
   - When O(n) extra space is unacceptable

4. **External Sorting**
   - K-way merge using min-heap
   - Sorting streaming data

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Quick Sort](quick_sort.md) | Both O(n log n) avg | Better cache locality |
| [Merge Sort](merge_sort.md) | Both guaranteed O(n log n) | Stability needed |
| [Intro Sort](intro_sort.md) | Uses heap sort as fallback | General purpose |
| [Selection Sort](selection_sort.md) | Similar selection approach | Very small arrays |

### 6.3 Heap Sort vs Other O(n log n) Sorts

```
                    Quick Sort   Merge Sort   Heap Sort
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Worst case          O(n²)        O(n log n)   O(n log n)
Space               O(log n)     O(n)         O(1)
Stable              No           Yes          No
Cache-friendly      Yes          Moderate     No
Constant factor     Low          Medium       High
```

## 7. Optimizations

### 7.1 Bottom-Up Heapify (Floyd's Method)

More efficient sift-down instead of sift-up:
```rust
fn build_heap_optimized<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    // Start from the last internal node
    for i in (0..n/2).rev() {
        sift_down(arr, i, n);
    }
}
```

### 7.2 Ternary Heap

Using a 3-ary heap instead of binary:
- Shallower tree: $\log_3 n$ vs $\log_2 n$
- More comparisons per level
- Better cache performance in some cases

### 7.3 Smooth Sort

Heap sort variant with better performance on nearly sorted data:
- Uses Leonardo heaps
- Adaptive to existing order
- Still O(1) space

### 7.4 Weak Heap Sort

Uses a weak heap structure:
- Fewer comparisons than binary heap sort
- ~$n \log n - 0.9n$ comparisons

## 8. Heap Data Structure Details

### 8.1 Array Representation Visualization

```
Array:  [16, 14, 10, 8, 7, 9, 3, 2, 4, 1]
Index:   0   1   2  3  4  5  6  7  8  9

Tree structure:
                16(0)
               /    \
           14(1)    10(2)
           /  \      /  \
        8(3) 7(4)  9(5) 3(6)
        / \   |
      2(7)4(8)1(9)
```

### 8.2 Index Calculations

```rust
fn parent(i: usize) -> usize { (i - 1) / 2 }
fn left_child(i: usize) -> usize { 2 * i + 1 }
fn right_child(i: usize) -> usize { 2 * i + 2 }
```

## 9. References

1. Williams, J. W. J. (1964). "Algorithm 232: Heapsort". *Communications of the ACM*. 7 (6): 347–348.
2. Floyd, R. W. (1964). "Algorithm 245: Treesort 3". *Communications of the ACM*. 7 (12): 701.
3. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 6.
4. Sedgewick, R. (1998). "Algorithms in C++".

## 10. Source Code

**Implementation**: [src/sorting/heap_sort.rs](../../src/sorting/heap_sort.rs)

```rust
pub fn heap_sort<T: Ord>(arr: &mut [T], ascending: bool) {
    if arr.len() <= 1 {
        return;
    }

    // Build heap based on the order
    build_heap(arr, ascending);

    let mut end = arr.len() - 1;
    while end > 0 {
        arr.swap(0, end);
        heapify(&mut arr[..end], 0, ascending);
        end -= 1;
    }
}
```
