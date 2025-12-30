# Exchange Sort

## 1. Overview

Exchange Sort is a simple comparison-based sorting algorithm that compares each element with every other element and swaps them if they are in the wrong order. It's similar to Bubble Sort but uses a different comparison pattern.

### Key Characteristics
- **Type**: Comparison-based, exchange sort
- **In-place**: Yes
- **Stable**: No (in typical implementation)
- **Simple**: Very easy to understand and implement

## 2. Mathematical Foundation

### 2.1 Comparison Pattern

Unlike Bubble Sort which compares adjacent elements, Exchange Sort compares:
- Element at position i with all elements at positions j > i

Total comparisons: $\frac{n(n-1)}{2}$ (always)

### 2.2 Invariant

After iteration i, the element at position i is in its correct final position.

## 3. Algorithm Description

### 3.1 Pseudocode

```
EXCHANGE_SORT(A)
    n ← length(A)
    
    for i ← 0 to n - 2 do
        for j ← i + 1 to n - 1 do
            if A[i] > A[j] then
                swap(A[i], A[j])
```

### 3.2 Step-by-Step Example

Sorting `[64, 34, 25, 12, 22]`:

```
Initial: [64, 34, 25, 12, 22]

i = 0:
  Compare 64 with 34: swap → [34, 64, 25, 12, 22]
  Compare 34 with 25: swap → [25, 64, 34, 12, 22]
  Compare 25 with 12: swap → [12, 64, 34, 25, 22]
  Compare 12 with 22: no swap
  After i=0: [12, 64, 34, 25, 22]  ← 12 is in place

i = 1:
  Compare 64 with 34: swap → [12, 34, 64, 25, 22]
  Compare 34 with 25: swap → [12, 25, 64, 34, 22]
  Compare 25 with 22: swap → [12, 22, 64, 34, 25]
  After i=1: [12, 22, 64, 34, 25]  ← 22 is in place

i = 2:
  Compare 64 with 34: swap → [12, 22, 34, 64, 25]
  Compare 34 with 25: swap → [12, 22, 25, 64, 34]
  After i=2: [12, 22, 25, 64, 34]  ← 25 is in place

i = 3:
  Compare 64 with 34: swap → [12, 22, 25, 34, 64]
  After i=3: [12, 22, 25, 34, 64]  ← 34 is in place

Final: [12, 22, 25, 34, 64]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Best** | O(n²) |
| **Average** | O(n²) |
| **Worst** | O(n²) |
| **Space** | O(1) |

### 4.1 Comparison Count

Always performs exactly $\frac{n(n-1)}{2}$ comparisons.

| n | Comparisons |
|---|-------------|
| 10 | 45 |
| 100 | 4,950 |
| 1000 | 499,500 |

## 5. Implementation

```rust
pub fn exchange_sort<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    
    for i in 0..n.saturating_sub(1) {
        for j in (i + 1)..n {
            if arr[i] > arr[j] {
                arr.swap(i, j);
            }
        }
    }
}
```

### 5.1 With Minimum Finding (Selection Sort Variant)

```rust
pub fn exchange_sort_optimized<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    
    for i in 0..n.saturating_sub(1) {
        let mut min_idx = i;
        for j in (i + 1)..n {
            if arr[j] < arr[min_idx] {
                min_idx = j;
            }
        }
        if min_idx != i {
            arr.swap(i, min_idx);
        }
    }
}
```

## 6. Comparison with Similar Algorithms

| Algorithm | Best | Average | Worst | Adaptive |
|-----------|------|---------|-------|----------|
| Exchange Sort | O(n²) | O(n²) | O(n²) | No |
| Bubble Sort | O(n) | O(n²) | O(n²) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | Yes |

### 6.1 Differences from Bubble Sort

| Aspect | Exchange Sort | Bubble Sort |
|--------|--------------|-------------|
| Comparison pattern | i vs all j > i | Adjacent pairs |
| Early termination | No | Possible |
| Swaps per pass | Multiple | One element bubbles |
| Adaptive | No | Yes |

### 6.2 Differences from Selection Sort

| Aspect | Exchange Sort | Selection Sort |
|--------|--------------|----------------|
| Swaps | O(n²) worst | O(n) |
| Finding minimum | Via swaps | Via comparisons |
| Memory writes | More | Fewer |

## 7. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| Simple | Easy to understand and implement |
| In-place | O(1) extra space |
| No recursion | Simple control flow |

### Limitations
| Limitation | Description |
|------------|-------------|
| O(n²) always | No best case optimization |
| Not adaptive | Doesn't benefit from sorted input |
| Many swaps | More writes than Selection Sort |
| Not stable | Equal elements may be reordered |

## 8. Applications

- **Teaching**: Introduction to sorting concepts
- **Small datasets**: When simplicity matters more than efficiency
- **Embedded systems**: When code size is critical

## 9. References

1. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*.
2. Cormen, T. H. (2009). *Introduction to Algorithms*.

## 10. Source Code

**Implementation**: [src/sorting/exchange_sort.rs](../../src/sorting/exchange_sort.rs)
