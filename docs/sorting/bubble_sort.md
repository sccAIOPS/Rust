# Bubble Sort

## 1. Overview

Bubble Sort is one of the simplest comparison-based sorting algorithms. It repeatedly steps through the list, compares adjacent elements, and swaps them if they are in the wrong order. The pass through the list is repeated until the list is sorted.

### Key Characteristics
- **Type**: Comparison-based, exchange sort
- **In-place**: Yes
- **Stable**: Yes
- **Adaptive**: Yes (with early termination optimization)

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, sort by repeatedly swapping adjacent elements that are out of order until no swaps are needed.

### 2.2 Loop Invariant

After pass $i$:
- The $i$ largest elements are in their final positions at the end of the array
- $A[n-i..n-1]$ is sorted and contains the $i$ largest elements

### 2.3 Bubble Property

In each pass, the largest unsorted element "bubbles up" to its correct position, like a bubble rising to the surface.

## 3. Algorithm Description

### 3.1 Intuition

Imagine bubbles of different sizes in water—larger bubbles rise faster. Similarly, larger elements bubble toward the end of the array with each pass.

### 3.2 Pseudocode

```
BUBBLE_SORT(A)
    n ← length(A)
    sorted ← false
    
    while not sorted do
        sorted ← true
        for i ← 0 to n - 2 do
            if A[i] > A[i + 1] then
                swap A[i] and A[i + 1]
                sorted ← false
        n ← n - 1    // Optimization: shrink unsorted region
```

### 3.3 Step-by-Step Example

Sorting array `[5, 3, 8, 4, 2]`:

```
Pass 1:
[5, 3, 8, 4, 2] → [3, 5, 8, 4, 2]  (swap 5,3)
[3, 5, 8, 4, 2] → [3, 5, 8, 4, 2]  (5 < 8, no swap)
[3, 5, 8, 4, 2] → [3, 5, 4, 8, 2]  (swap 8,4)
[3, 5, 4, 8, 2] → [3, 5, 4, 2, 8]  (swap 8,2)
                            sorted: ^

Pass 2:
[3, 5, 4, 2, 8] → [3, 5, 4, 2, 8]  (no swap)
[3, 5, 4, 2, 8] → [3, 4, 5, 2, 8]  (swap 5,4)
[3, 4, 5, 2, 8] → [3, 4, 2, 5, 8]  (swap 5,2)
                         sorted: ^

Pass 3:
[3, 4, 2, 5, 8] → [3, 4, 2, 5, 8]  (no swap)
[3, 4, 2, 5, 8] → [3, 2, 4, 5, 8]  (swap 4,2)
                      sorted: ^

Pass 4:
[3, 2, 4, 5, 8] → [2, 3, 4, 5, 8]  (swap 3,2)
                   sorted: ^

Final: [2, 3, 4, 5, 8]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Condition |
|------|------------|-----------|
| **Best** | O(n) | Already sorted (with early termination) |
| **Average** | O(n²) | Random order |
| **Worst** | O(n²) | Reverse sorted |

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | O(1) | Only temp variable for swap |

### 4.3 Comparison and Swap Counts

| Case | Comparisons | Swaps |
|------|-------------|-------|
| Best | n - 1 | 0 |
| Worst | n(n-1)/2 | n(n-1)/2 |
| Average | n(n-1)/2 | n(n-1)/4 |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn bubble_sort<T: Ord>(arr: &mut [T]) {
    if arr.is_empty() {
        return;
    }
    let mut sorted = false;
    let mut n = arr.len();
    while !sorted {
        sorted = true;
        for i in 0..n - 1 {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                sorted = false;
            }
        }
        n -= 1;
    }
}
```

**Key Implementation Details**:

1. **Early termination**: `sorted` flag stops when no swaps occur
2. **Shrinking boundary**: `n -= 1` reduces comparisons each pass
3. **Empty array check**: Prevents underflow on `n - 1`

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Returns immediately |
| Single element | Returns immediately (n-1 = 0) |
| Already sorted | Single pass, O(n) |
| Reverse sorted | Maximum passes needed |

## 6. Real-World Applications

### 6.1 When to Use Bubble Sort

1. **Educational purposes**
   - Teaching basic sorting concepts
   - Demonstrating algorithm analysis

2. **Nearly sorted data with few swaps needed**
   - With early termination optimization
   
3. **Very small arrays**
   - Simple code might be preferable

### 6.2 When NOT to Use

- Large datasets
- Performance-critical applications
- When other O(n²) sorts like Insertion Sort are available

### 6.3 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Cocktail Shaker Sort](cocktail_shaker_sort.md) | Bidirectional bubble | Elements at both ends |
| [Comb Sort](comb_sort.md) | Bubble with gaps | Larger arrays |
| [Insertion Sort](insertion_sort.md) | Better O(n²) sort | Almost always |

## 7. Variants

### 7.1 Cocktail Shaker Sort

Bidirectional bubble sort that alternates directions:
```rust
pub fn cocktail_shaker_sort<T: Ord>(arr: &mut [T]) {
    loop {
        let mut swapped = false;
        // Forward pass
        for i in 0..arr.len() - 1 {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                swapped = true;
            }
        }
        if !swapped { break; }
        
        swapped = false;
        // Backward pass
        for i in (0..arr.len() - 1).rev() {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                swapped = true;
            }
        }
        if !swapped { break; }
    }
}
```

### 7.2 Odd-Even Sort

Parallel-friendly variant that alternates between odd and even indexed pairs.

## 8. Stability

Bubble Sort is **stable** because:
- Only adjacent elements are swapped
- Swap only occurs when `A[i] > A[i+1]`, not `≥`
- Equal elements are never swapped

```
Example: [(3,'a'), (3,'b'), (1,'c')]
Pass 1: [(3,'a'), (1,'c'), (3,'b')]  ← (3,'b') swapped with (1,'c')
Pass 2: [(1,'c'), (3,'a'), (3,'b')]  ← (3,'a') stays before (3,'b')

Final order of equal elements preserved!
```

## 9. Historical Note

Bubble Sort's simplicity made it a common teaching example, but its inefficiency led to Donald Knuth's famous statement:

> "The bubble sort seems to have nothing to recommend it, except a catchy name and the fact that it leads to some interesting theoretical problems."
> — Donald Knuth, The Art of Computer Programming

## 10. References

1. Knuth, D. E. (1998). "The Art of Computer Programming, Vol. 3: Sorting and Searching".
2. Cormen, T. H., et al. "Introduction to Algorithms".
3. Astrachan, O. (2003). "Bubble sort: An archaeological algorithmic analysis". SIGCSE Bulletin.

## 11. Source Code

**Implementation**: [src/sorting/bubble_sort.rs](../../src/sorting/bubble_sort.rs)

```rust
pub fn bubble_sort<T: Ord>(arr: &mut [T]) {
    if arr.is_empty() {
        return;
    }
    let mut sorted = false;
    let mut n = arr.len();
    while !sorted {
        sorted = true;
        for i in 0..n - 1 {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                sorted = false;
            }
        }
        n -= 1;
    }
}
```
