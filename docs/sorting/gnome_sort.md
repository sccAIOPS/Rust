# Gnome Sort

## 1. Overview

Gnome Sort (also called Stupid Sort) is a simple sorting algorithm similar to Insertion Sort. It works by comparing adjacent elements and swapping them if they are in the wrong order, moving backward when a swap is made and forward when no swap is needed.

### Key Characteristics
- **Type**: Comparison-based, exchange sort
- **In-place**: Yes
- **Stable**: Yes
- **Simple**: Very easy to implement

## 2. Mathematical Foundation

### 2.1 Garden Gnome Analogy

The algorithm is named after the behavior of a garden gnome sorting flower pots:
1. Look at the pot next to you and the one before
2. If they're in order, move forward
3. If not, swap them and move backward
4. Repeat until you reach the end

## 3. Algorithm Description

### 3.1 Pseudocode

```
GNOME_SORT(A)
    pos ← 0
    n ← length(A)
    
    while pos < n do
        if pos = 0 or A[pos] ≥ A[pos - 1] then
            pos ← pos + 1
        else
            swap(A[pos], A[pos - 1])
            pos ← pos - 1
```

### 3.2 Step-by-Step Example

Sorting `[34, 2, 10, -9]`:

```
Initial: [34, 2, 10, -9], pos = 0

pos = 0: Move forward
pos = 1: 2 < 34, swap → [2, 34, 10, -9], pos = 0
pos = 0: Move forward
pos = 1: 34 ≥ 2, move forward
pos = 2: 10 < 34, swap → [2, 10, 34, -9], pos = 1
pos = 1: 10 ≥ 2, move forward
pos = 2: 34 ≥ 10, move forward
pos = 3: -9 < 34, swap → [2, 10, -9, 34], pos = 2
pos = 2: -9 < 10, swap → [2, -9, 10, 34], pos = 1
pos = 1: -9 < 2, swap → [-9, 2, 10, 34], pos = 0
pos = 0: Move forward
...
pos = 4: Done

Final: [-9, 2, 10, 34]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Best** | O(n) - already sorted |
| **Average** | O(n²) |
| **Worst** | O(n²) - reverse sorted |
| **Space** | O(1) |

## 5. Implementation

```rust
pub fn gnome_sort<T: Ord>(arr: &mut [T]) {
    let mut pos = 0;
    let len = arr.len();

    while pos < len {
        if pos == 0 || arr[pos] >= arr[pos - 1] {
            pos += 1;
        } else {
            arr.swap(pos, pos - 1);
            pos -= 1;
        }
    }
}
```

### 5.1 Optimized Version (with teleportation)

```rust
pub fn gnome_sort_optimized<T: Ord>(arr: &mut [T]) {
    let len = arr.len();
    
    for i in 1..len {
        let mut pos = i;
        while pos > 0 && arr[pos] < arr[pos - 1] {
            arr.swap(pos, pos - 1);
            pos -= 1;
        }
    }
}
```

## 6. Comparison with Insertion Sort

| Aspect | Gnome Sort | Insertion Sort |
|--------|------------|----------------|
| Comparisons | More | Fewer |
| Swaps | Same | Same |
| Code Simplicity | Simpler | Slightly complex |
| Uses binary search | No | Possible |

## 7. References

1. Hamid Sarbazi-Azad (2000). "Stupid Sort: A new sorting algorithm".
2. *The Art of Computer Programming*, Vol. 3.

## 8. Source Code

**Implementation**: [src/sorting/gnome_sort.rs](../../src/sorting/gnome_sort.rs)
