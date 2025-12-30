# Pigeonhole Sort

## 1. Overview

Pigeonhole Sort is a non-comparison sorting algorithm suitable for sorting elements where the number of elements (n) and the range of possible key values (N) are approximately the same. It's a simplified version of Counting Sort.

### Key Characteristics
- **Type**: Distribution sort, non-comparison
- **In-place**: No (requires auxiliary space)
- **Stable**: Yes (with proper implementation)
- **Best for**: Small range of integer keys

## 2. Mathematical Foundation

### 2.1 Pigeonhole Principle

The Pigeonhole Principle states: If n items are put into m containers where n > m, at least one container must contain more than one item.

In Pigeonhole Sort:
- "Pigeonholes" = containers for each possible key value
- Elements are placed in their corresponding holes
- Holes are emptied in order

### 2.2 Range Calculation

For values in range $[min, max]$:
- Number of pigeonholes: $max - min + 1$
- Hole index for value $v$: $v - min$

## 3. Algorithm Description

### 3.1 Pseudocode

```
PIGEONHOLE_SORT(A)
    n ← length(A)
    min ← minimum(A)
    max ← maximum(A)
    range ← max - min + 1
    
    // Create pigeonholes
    holes ← array of range empty lists
    
    // Distribute elements into holes
    for each element x in A do
        holes[x - min].append(x)
    
    // Collect elements from holes
    index ← 0
    for i ← 0 to range - 1 do
        for each element x in holes[i] do
            A[index] ← x
            index ← index + 1
```

### 3.2 Step-by-Step Example

Sorting `[8, 3, 2, 7, 4, 6, 8]`:

```
Initial: [8, 3, 2, 7, 4, 6, 8]
min = 2, max = 8
range = 8 - 2 + 1 = 7

Create 7 pigeonholes (indices 0-6):

Distribution:
  Hole 0 (value 2): [2]
  Hole 1 (value 3): [3]
  Hole 2 (value 4): [4]
  Hole 3 (value 5): []
  Hole 4 (value 6): [6]
  Hole 5 (value 7): [7]
  Hole 6 (value 8): [8, 8]

Collect from holes in order:
Result: [2, 3, 4, 6, 7, 8, 8]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Time** | O(n + N) where N = range |
| **Space** | O(n + N) |

### 4.1 When to Use

| Condition | Recommendation |
|-----------|---------------|
| N ≈ n | ✓ Excellent choice |
| N >> n | ✗ Use Counting Sort or other |
| N << n | ✓ Good, but Counting Sort may be better |
| Non-integer keys | ✗ Not applicable |

## 5. Implementation

```rust
pub fn pigeonhole_sort(arr: &mut [i32]) {
    if arr.len() <= 1 {
        return;
    }

    let min = *arr.iter().min().unwrap();
    let max = *arr.iter().max().unwrap();
    let range = (max - min + 1) as usize;

    // Create pigeonholes
    let mut holes: Vec<Vec<i32>> = vec![vec![]; range];

    // Distribute elements
    for &val in arr.iter() {
        holes[(val - min) as usize].push(val);
    }

    // Collect elements
    let mut index = 0;
    for hole in holes {
        for val in hole {
            arr[index] = val;
            index += 1;
        }
    }
}
```

### 5.1 Generic Implementation

```rust
pub fn pigeonhole_sort_generic<T, F>(arr: &mut [T], key: F)
where
    T: Clone,
    F: Fn(&T) -> usize,
{
    if arr.len() <= 1 {
        return;
    }

    let keys: Vec<usize> = arr.iter().map(&key).collect();
    let min_key = *keys.iter().min().unwrap();
    let max_key = *keys.iter().max().unwrap();
    let range = max_key - min_key + 1;

    let mut holes: Vec<Vec<T>> = vec![vec![]; range];

    for item in arr.iter() {
        holes[key(item) - min_key].push(item.clone());
    }

    let mut index = 0;
    for hole in holes {
        for item in hole {
            arr[index] = item;
            index += 1;
        }
    }
}
```

## 6. Comparison with Similar Algorithms

| Algorithm | Time | Space | Stability |
|-----------|------|-------|-----------|
| Pigeonhole Sort | O(n + N) | O(n + N) | Yes |
| Counting Sort | O(n + k) | O(k) | Yes |
| Bucket Sort | O(n + k) | O(n + k) | Depends |

### 6.1 Pigeonhole vs Counting Sort

| Aspect | Pigeonhole | Counting Sort |
|--------|------------|---------------|
| Storage | Stores actual elements | Stores counts |
| Implementation | Simpler | More complex |
| Memory | Higher (stores elements) | Lower (just counts) |
| Flexibility | Better for objects | Better for integers |

## 7. Applications

1. **Radix Sort component**: As a stable sort for individual digits
2. **Small range integers**: When range ≈ count
3. **Sorting with satellite data**: When elements have associated data

## 8. Limitations

| Limitation | Description |
|------------|-------------|
| Integer keys only | Cannot directly sort floats/strings |
| Range-dependent | Inefficient for large ranges |
| Memory usage | Needs O(range) pigeonholes |

## 9. References

1. Cormen, T. H. (2009). *Introduction to Algorithms*, Chapter 8.
2. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*.

## 10. Source Code

**Implementation**: [src/sorting/pigeonhole_sort.rs](../../src/sorting/pigeonhole_sort.rs)
