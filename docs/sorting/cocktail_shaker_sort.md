# Cocktail Shaker Sort

## 1. Overview

Cocktail Shaker Sort (also known as Bidirectional Bubble Sort or Cocktail Sort) is a variation of Bubble Sort that sorts in both directions on each pass through the list. This helps move both "rabbits" (large values) and "turtles" (small values) efficiently.

### Key Characteristics
- **Type**: Comparison-based, exchange sort
- **In-place**: Yes
- **Stable**: Yes
- **Bidirectional**: Yes

## 2. Mathematical Foundation

### 2.1 Bidirectional Optimization

Traditional Bubble Sort only bubbles up (or down):
- Forward pass: Largest element moves to end
- Backward pass: Smallest element moves to start

This reduces the worst-case constant factor compared to Bubble Sort.

## 3. Algorithm Description

### 3.1 Pseudocode

```
COCKTAIL_SHAKER_SORT(A)
    swapped ← true
    start ← 0
    end ← length(A) - 1
    
    while swapped do
        swapped ← false
        
        // Forward pass (left to right)
        for i ← start to end - 1 do
            if A[i] > A[i + 1] then
                swap(A[i], A[i + 1])
                swapped ← true
        
        if not swapped then break
        
        swapped ← false
        end ← end - 1
        
        // Backward pass (right to left)
        for i ← end - 1 down to start do
            if A[i] > A[i + 1] then
                swap(A[i], A[i + 1])
                swapped ← true
        
        start ← start + 1
```

### 3.2 Step-by-Step Example

Sorting `[5, 1, 4, 2, 8, 0, 2]`:

```
Initial: [5, 1, 4, 2, 8, 0, 2]

Pass 1 Forward: 8 moves to end
[1, 4, 2, 5, 0, 2, 8]

Pass 1 Backward: 0 moves to start
[0, 1, 4, 2, 5, 2, 8]

Pass 2 Forward:
[0, 1, 2, 4, 2, 5, 8]

Pass 2 Backward:
[0, 1, 2, 2, 4, 5, 8]

Final: [0, 1, 2, 2, 4, 5, 8]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Best** | O(n) - already sorted |
| **Average** | O(n²) |
| **Worst** | O(n²) |
| **Space** | O(1) |

## 5. Implementation

```rust
pub fn cocktail_shaker_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();

    if len == 0 {
        return;
    }

    loop {
        let mut swapped = false;

        // Forward pass
        for i in 0..(len - 1).saturating_sub(0) {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                swapped = true;
            }
        }

        if !swapped {
            break;
        }

        swapped = false;

        // Backward pass
        for i in (0..(len - 1)).rev() {
            if arr[i] > arr[i + 1] {
                arr.swap(i, i + 1);
                swapped = true;
            }
        }

        if !swapped {
            break;
        }
    }
}
```

## 6. Advantages Over Bubble Sort

| Aspect | Bubble Sort | Cocktail Shaker |
|--------|-------------|-----------------|
| Direction | Unidirectional | Bidirectional |
| Turtle Problem | Yes | Reduced |
| Passes for nearly sorted | High | Lower |
| Best case | O(n) | O(n) |

## 7. References

1. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*.
2. Astrachan, O. (2003). "Bubble Sort: An Archaeological Algorithmic Analysis".

## 8. Source Code

**Implementation**: [src/sorting/cocktail_shaker_sort.rs](../../src/sorting/cocktail_shaker_sort.rs)
