# Binary Insertion Sort

## 1. Overview

Binary Insertion Sort is an optimization of Insertion Sort that uses binary search to find the correct position for each element, reducing the number of comparisons. However, the number of shifts remains the same, so the overall time complexity is still O(n²).

### Key Characteristics
- **Type**: Comparison-based, insertion sort variant
- **In-place**: Yes
- **Stable**: Yes
- **Optimization**: Reduces comparisons from O(n) to O(log n) per element

## 2. Mathematical Foundation

### 2.1 Comparison Reduction

Standard Insertion Sort:
- For element at position i: up to i comparisons
- Total: $\sum_{i=1}^{n-1} i = \frac{n(n-1)}{2} = O(n²)$ comparisons

Binary Insertion Sort:
- For element at position i: $\lceil \log_2 i \rceil$ comparisons
- Total: $\sum_{i=1}^{n-1} \log_2 i = O(n \log n)$ comparisons

### 2.2 Shift Operations (Unchanged)

Both algorithms require the same number of shifts:
- Worst case: $O(n²)$ shifts
- This dominates the time complexity

## 3. Algorithm Description

### 3.1 Pseudocode

```
BINARY_INSERTION_SORT(A)
    n ← length(A)
    
    for i ← 1 to n - 1 do
        key ← A[i]
        
        // Binary search for insertion point
        pos ← BINARY_SEARCH(A, 0, i - 1, key)
        
        // Shift elements to make room
        for j ← i down to pos + 1 do
            A[j] ← A[j - 1]
        
        A[pos] ← key

BINARY_SEARCH(A, low, high, key)
    while low ≤ high do
        mid ← (low + high) / 2
        if key < A[mid] then
            high ← mid - 1
        else
            low ← mid + 1
    return low
```

### 3.2 Step-by-Step Example

Sorting `[5, 2, 4, 6, 1, 3]`:

```
Initial: [5, 2, 4, 6, 1, 3]

i = 1: key = 2
  Binary search in [5]: pos = 0
  Shift: [5, 5, 4, 6, 1, 3]
  Insert: [2, 5, 4, 6, 1, 3]

i = 2: key = 4
  Binary search in [2, 5]: 
    mid=0, 4 > 2 → low=1
    mid=1, 4 < 5 → high=0
    return 1
  Shift: [2, 5, 5, 6, 1, 3]
  Insert: [2, 4, 5, 6, 1, 3]

i = 3: key = 6
  Binary search in [2, 4, 5]: pos = 3 (already in place)
  No shift needed
  [2, 4, 5, 6, 1, 3]

i = 4: key = 1
  Binary search in [2, 4, 5, 6]: pos = 0
  Shift all: [2, 2, 4, 5, 6, 3]
  Insert: [1, 2, 4, 5, 6, 3]

i = 5: key = 3
  Binary search in [1, 2, 4, 5, 6]:
    mid=2, 3 < 4 → high=1
    mid=0, 3 > 1 → low=1
    mid=1, 3 > 2 → low=2
    return 2
  Shift: [1, 2, 4, 4, 5, 6]
  Insert: [1, 2, 3, 4, 5, 6]

Final: [1, 2, 3, 4, 5, 6]
```

## 4. Complexity Analysis

| Operation | Standard Insertion | Binary Insertion |
|-----------|-------------------|------------------|
| Comparisons | O(n²) | O(n log n) |
| Shifts | O(n²) | O(n²) |
| **Total Time** | **O(n²)** | **O(n²)** |
| Space | O(1) | O(1) |

## 5. Implementation

```rust
pub fn binary_insertion_sort<T: Ord + Copy>(arr: &mut [T]) {
    for i in 1..arr.len() {
        let key = arr[i];
        
        // Binary search for insertion position
        let pos = binary_search_pos(arr, 0, i, key);
        
        // Shift elements
        for j in (pos..i).rev() {
            arr[j + 1] = arr[j];
        }
        
        arr[pos] = key;
    }
}

fn binary_search_pos<T: Ord>(arr: &[T], mut low: usize, high: usize, key: T) -> usize {
    let mut high = high;
    
    while low < high {
        let mid = low + (high - low) / 2;
        if arr[mid] <= key {
            low = mid + 1;
        } else {
            high = mid;
        }
    }
    
    low
}
```

### 5.1 Using slice::rotate_right

```rust
pub fn binary_insertion_sort_rotate<T: Ord>(arr: &mut [T]) {
    for i in 1..arr.len() {
        let pos = arr[..i]
            .binary_search(&arr[i])
            .unwrap_or_else(|pos| pos);
        
        arr[pos..=i].rotate_right(1);
    }
}
```

## 6. Comparison: Standard vs Binary Insertion Sort

### 6.1 Performance Characteristics

| Aspect | Standard | Binary |
|--------|----------|--------|
| Comparisons/element | O(i) | O(log i) |
| Shifts/element | O(i) | O(i) |
| Cache behavior | Good | Similar |
| Branch prediction | Better | Worse |

### 6.2 When Binary Insertion Helps

- **Expensive comparisons**: Custom objects, strings
- **Cheap moves**: Small elements, array indices
- **Linked lists**: Shifts become O(1) if using pointers

### 6.3 When Standard is Better

- **Primitive types**: Comparisons are cheap
- **Nearly sorted data**: Standard terminates early
- **Branch prediction**: Linear scan is predictable

## 7. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| Fewer comparisons | O(n log n) vs O(n²) |
| Stable | Preserves equal element order |
| In-place | O(1) extra space |
| Good for expensive comparisons | Strings, complex objects |

### Limitations
| Limitation | Description |
|------------|-------------|
| Still O(n²) shifts | Dominant factor for arrays |
| More complex code | Binary search logic |
| Not adaptive | Doesn't terminate early |
| Worse for sorted input | Standard can be O(n) |

## 8. Applications

1. **String sorting**: When comparison is expensive
2. **Sorting by key function**: Complex key extraction
3. **Small to medium arrays**: When simplicity matters
4. **Educational**: Understanding algorithm optimization

## 9. Variations

### 9.1 Exponential Search Variant

For nearly sorted arrays, use exponential search:
1. First find range using exponential growth
2. Then binary search within range

### 9.2 Hybrid with Merge Sort

Use binary insertion sort for small subarrays in merge sort.

## 10. References

1. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*, Section 5.2.1.
2. Cormen, T. H. (2009). *Introduction to Algorithms*, Exercise 2.3-6.
3. Sedgewick, R. (2011). *Algorithms*, 4th Edition.

## 11. Source Code

**Implementation**: [src/sorting/binary_insertion_sort.rs](../../src/sorting/binary_insertion_sort.rs)
