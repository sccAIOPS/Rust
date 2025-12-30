# Heap (Binary Heap)

## 1. Overview

A Heap is a specialized tree-based data structure that satisfies the heap property: in a max-heap, each parent is greater than or equal to its children; in a min-heap, each parent is less than or equal to its children. The heap is the foundation for the heap sort algorithm and is commonly used to implement priority queues.

The binary heap was introduced by J. W. J. Williams in 1964 for the heap sort algorithm.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Design a data structure supporting:
- **Insert**: Add element while maintaining heap property
- **Extract-Min/Max**: Remove and return the extremum
- **Peek**: Return the extremum without removal
- **Heapify**: Convert an arbitrary array to a heap

### 2.2 Mathematical Model

**Complete Binary Tree**: A heap is a complete binary tree, meaning all levels are fully filled except possibly the last, which is filled from left to right.

**Heap Property**:
- **Max-Heap**: $\forall i > 0: A[\text{parent}(i)] \geq A[i]$
- **Min-Heap**: $\forall i > 0: A[\text{parent}(i)] \leq A[i]$

**Array Representation** (0-indexed):
- Parent of node $i$: $\text{parent}(i) = \lfloor (i-1)/2 \rfloor$
- Left child of node $i$: $\text{left}(i) = 2i + 1$
- Right child of node $i$: $\text{right}(i) = 2i + 2$

**Height**: For $n$ elements, height $h = \lfloor \log_2 n \rfloor$

### 2.3 Key Properties

1. The root contains the maximum (max-heap) or minimum (min-heap) element
2. No ordering relationship between siblings
3. Subtrees are also heaps

## 3. Algorithm Description

### 3.1 Intuition

A heap is like a tournament bracket. The winner (maximum/minimum) is always at the top. When you remove the winner, you run a mini-tournament to find the next best element. When adding a new competitor, they bubble up until they find their proper ranking.

### 3.2 Array Layout

```
Tree View:           Array: [50, 30, 40, 10, 20, 35, 38]
       50                    0   1   2   3   4   5   6
      /  \
    30    40
   /  \   / \
  10  20 35  38
  
Index relationships:
- Parent of 5 (35): (5-1)/2 = 2 (40) ✓
- Children of 1 (30): 2*1+1=3 (10), 2*1+2=4 (20) ✓
```

### 3.3 Pseudocode

```
PUSH(heap, value):
    heap.append(value)
    SIFT_UP(heap, heap.length - 1)

SIFT_UP(heap, index):
    while index > 0:
        parent = (index - 1) / 2
        if compare(heap[index], heap[parent]):  // index should be above parent
            swap(heap[index], heap[parent])
            index = parent
        else:
            break

POP(heap):
    if heap is empty:
        return None
    result = heap[0]
    heap[0] = heap[last]
    heap.remove_last()
    if heap is not empty:
        SIFT_DOWN(heap, 0)
    return result

SIFT_DOWN(heap, index):
    while true:
        best = index
        left = 2 * index + 1
        right = 2 * index + 2
        
        if left < heap.length and compare(heap[left], heap[best]):
            best = left
        if right < heap.length and compare(heap[right], heap[best]):
            best = right
        
        if best == index:
            break
        
        swap(heap[index], heap[best])
        index = best

BUILD_HEAP(array):
    // Start from last non-leaf node
    for i from (n/2 - 1) downto 0:
        SIFT_DOWN(array, i)
```

### 3.4 Step-by-Step Example

**Build Max-Heap from [4, 10, 3, 5, 1]**:

```
Initial array: [4, 10, 3, 5, 1]
Tree view:
       4
      / \
    10   3
   / \
  5   1

Start at index 1 (last non-leaf = n/2 - 1 = 1)

Sift-down index 1: 10 > 5, 10 > 1, no change needed
Sift-down index 0: 4 vs children (10, 3)
  - 10 > 4, swap 4 and 10
  - Now at index 1: 4 vs children (5, 1)
  - 5 > 4, swap 4 and 5

Final: [10, 5, 3, 4, 1]
       10
      /  \
     5    3
    / \
   4   1
```

**Pop from Max-Heap [10, 5, 3, 4, 1]**:

```
1. Return 10, move last (1) to root
   [1, 5, 3, 4]
       1
      / \
     5   3
    /
   4

2. Sift-down 1: 5 > 3 > 1, swap 1 and 5
   [5, 1, 3, 4]
       5
      / \
     1   3
    /
   4

3. Sift-down 1: 4 > 1, swap 1 and 4
   [5, 4, 3, 1]
       5
      / \
     4   3
    /
   1

Result: 10, heap is now [5, 4, 3, 1]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time |
|-----------|------|
| Push      | O(log n) |
| Pop       | O(log n) |
| Peek      | O(1) |
| Build Heap | O(n) |

**Build Heap is O(n), not O(n log n)**:

Proof: Sift-down at height $h$ costs $O(h)$. Number of nodes at height $h$ is $\leq \lceil n/2^{h+1} \rceil$.

$$T(n) = \sum_{h=0}^{\lfloor \log n \rfloor} \lceil n/2^{h+1} \rceil \cdot O(h) = O(n \sum_{h=0}^{\infty} h/2^h) = O(n)$$

### 4.2 Space Complexity

- **Storage**: O(n) for n elements
- **Auxiliary**: O(1) - heaps are in-place
- **Per Operation**: O(1) extra space (iterative) or O(log n) (recursive sift)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct Heap<T> {
    items: Vec<T>,
    comparator: fn(&T, &T) -> bool,
}

impl<T> Heap<T> {
    pub fn new_min() -> Self where T: Ord {
        Self { items: vec![], comparator: |a, b| a < b }
    }
    
    pub fn new_max() -> Self where T: Ord {
        Self { items: vec![], comparator: |a, b| a > b }
    }
    
    pub fn new(comparator: fn(&T, &T) -> bool) -> Self {
        Self { items: vec![], comparator }
    }
}
```

**Key Design Patterns**:
- `Vec<T>` for dynamic array storage
- Custom comparator function for flexibility
- Separate constructors for min/max heaps
- Generic over `T` with no trait bounds (comparator handles comparison)

**Index Calculations**:
```rust
fn parent(i: usize) -> usize { (i - 1) / 2 }
fn left_child(i: usize) -> usize { 2 * i + 1 }
fn right_child(i: usize) -> usize { 2 * i + 2 }
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Pop from empty | Return None |
| Single element | Pop returns it, sift-down is no-op |
| All equal elements | Valid heap, any arrangement works |
| Push duplicate | Allowed, treated as separate element |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Priority Queues**: Task scheduling, event-driven simulation
2. **Heap Sort**: O(n log n) in-place sorting
3. **Graph Algorithms**: Dijkstra's, Prim's MST
4. **Operating Systems**: Process scheduling
5. **Median Finding**: Two heaps (min and max)
6. **K Largest/Smallest**: Maintain heap of size k

### 6.2 Heap Sort

```rust
pub fn heap_sort<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    // Build max-heap
    for i in (0..n/2).rev() {
        sift_down(arr, i, n);
    }
    // Extract elements
    for i in (1..n).rev() {
        arr.swap(0, i);
        sift_down(arr, 0, i);
    }
}
```

### 6.3 Variants

| Variant | Description | Use Case |
|---------|-------------|----------|
| **Binary Heap** | Standard, 2 children | General purpose |
| **d-ary Heap** | d children per node | Decrease-key heavy |
| **Binomial Heap** | Forest of binomial trees | Mergeable |
| **Fibonacci Heap** | Amortized O(1) decrease-key | Dijkstra's algorithm |
| **Pairing Heap** | Simpler than Fibonacci | Practical alternative |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Off-by-One in Index**: 0-indexed vs 1-indexed formulas differ
2. **Forgetting to Sift**: After swap, must continue sifting
3. **Wrong Comparator**: Min vs max heap confusion
4. **Empty Heap Check**: Pop on empty heap

### 7.2 Optimization Opportunities

**Floyd's Heap Construction**:
Build heap bottom-up (sift-down from n/2-1 to 0) is O(n).
Top-down (repeated insert) is O(n log n).

**d-ary Heap**:
- Push: O(log_d n) - fewer levels
- Pop: O(d log_d n) - more comparisons per level
- Best d depends on push/pop ratio

**Cache Optimization**:
- Use array (cache-friendly) not pointers
- Consider memory layout for large elements

### 7.3 Standard Library

Rust's `std::collections::BinaryHeap` is a max-heap:

```rust
use std::collections::BinaryHeap;

let mut heap = BinaryHeap::new();
heap.push(3);
heap.push(1);
heap.push(4);
assert_eq!(heap.pop(), Some(4));

// For min-heap, use Reverse wrapper:
use std::cmp::Reverse;
let mut min_heap = BinaryHeap::new();
min_heap.push(Reverse(3));
```

## 8. References

- Williams, J. W. J. (1964). "Algorithm 232: Heapsort". *Communications of the ACM*.
- Floyd, R. W. (1964). "Algorithm 245: Treesort 3". *Communications of the ACM*.
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 6.
- Sedgewick, R. (1988). *Algorithms* (2nd ed.). Addison-Wesley. Chapter 11.
