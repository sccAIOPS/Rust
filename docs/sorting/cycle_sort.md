# Cycle Sort

## 1. Overview

Cycle Sort is an in-place, unstable sorting algorithm that is optimal in terms of the total number of writes to the original array. It is based on the idea that a permutation can be factored into cycles, and each element is moved directly to its final position.

### Key Characteristics
- **Type**: Comparison-based, selection sort variant
- **In-place**: Yes
- **Stable**: No
- **Optimal writes**: Yes (minimizes write operations)

## 2. Mathematical Foundation

### 2.1 Cycle Decomposition

Any permutation can be decomposed into disjoint cycles:
- Original: `[3, 1, 4, 2]` → Permutation from `[1, 2, 3, 4]`
- Cycles: `(1 → 3 → 4 → 2 → 1)` or in 0-indexed: `(0 → 2 → 3 → 1 → 0)`

### 2.2 Write Optimality

- Each element is written at most once to its final position
- Number of writes = $n - c$ where $c$ = number of cycles (including fixed points)

## 3. Algorithm Description

### 3.1 Pseudocode

```
CYCLE_SORT(A)
    n ← length(A)
    writes ← 0
    
    for cycle_start ← 0 to n - 2 do
        item ← A[cycle_start]
        
        // Find position where item should go
        pos ← cycle_start
        for i ← cycle_start + 1 to n - 1 do
            if A[i] < item then
                pos ← pos + 1
        
        // If item is already in correct position, continue
        if pos = cycle_start then continue
        
        // Handle duplicates
        while item = A[pos] do
            pos ← pos + 1
        
        // Put item in its correct position
        if pos ≠ cycle_start then
            swap(item, A[pos])
            writes ← writes + 1
        
        // Rotate the rest of the cycle
        while pos ≠ cycle_start do
            pos ← cycle_start
            for i ← cycle_start + 1 to n - 1 do
                if A[i] < item then
                    pos ← pos + 1
            
            while item = A[pos] do
                pos ← pos + 1
            
            if item ≠ A[pos] then
                swap(item, A[pos])
                writes ← writes + 1
    
    return writes
```

### 3.2 Step-by-Step Example

Sorting `[4, 3, 2, 1]`:

```
Initial: [4, 3, 2, 1]

Cycle starting at index 0:
  item = 4
  Count elements < 4: 3 elements → pos = 3
  Swap: 4 ↔ 1 → [1, 3, 2, 4]
  
  Continue cycle with item = 1:
  Count elements < 1: 0 → pos = 0
  Cycle complete!

Cycle starting at index 1:
  item = 3
  Count elements < 3: 1 element → pos = 2
  Swap: 3 ↔ 2 → [1, 2, 3, 4]
  
  Continue cycle with item = 2:
  Count elements < 2: 1 → pos = 2... wait, adjust
  Actually: pos becomes 1, cycle complete

Final: [1, 2, 3, 4]
Total writes: 2 (optimal!)
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Time** | O(n²) |
| **Space** | O(1) |
| **Writes** | O(n) - optimal |

### 4.1 Write Analysis

| Algorithm | Writes (worst) |
|-----------|---------------|
| Cycle Sort | n - c |
| Selection Sort | n - 1 |
| Insertion Sort | O(n²) |
| Bubble Sort | O(n²) |

## 5. Implementation

```rust
pub fn cycle_sort<T: Ord>(arr: &mut [T]) {
    let len = arr.len();

    for cycle_start in 0..len.saturating_sub(1) {
        let mut pos = cycle_start;

        // Find position for arr[cycle_start]
        for i in (cycle_start + 1)..len {
            if arr[i] < arr[cycle_start] {
                pos += 1;
            }
        }

        // If already in correct position
        if pos == cycle_start {
            continue;
        }

        // Handle duplicates
        while arr[pos] == arr[cycle_start] {
            pos += 1;
        }

        // Put element in correct position
        arr.swap(cycle_start, pos);

        // Rotate rest of the cycle
        while pos != cycle_start {
            pos = cycle_start;

            for i in (cycle_start + 1)..len {
                if arr[i] < arr[cycle_start] {
                    pos += 1;
                }
            }

            while arr[pos] == arr[cycle_start] {
                pos += 1;
            }

            arr.swap(cycle_start, pos);
        }
    }
}
```

## 6. Use Cases

Cycle Sort is ideal when:
- Write operations are expensive (flash memory, EEPROM)
- Number of writes must be minimized
- Memory endurance is a concern

## 7. References

1. Haddon, B. K. (1990). "Cycle Sort: A Linear Sorting Method". *The Computer Journal*.
2. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*.

## 8. Source Code

**Implementation**: [src/sorting/cycle_sort.rs](../../src/sorting/cycle_sort.rs)
