# Bead Sort (Gravity Sort)

## 1. Overview

Bead Sort, also known as Gravity Sort, is a natural sorting algorithm that works by simulating gravity on beads arranged on vertical poles. Beads "fall" due to gravity, resulting in a sorted arrangement. It only works with positive integers.

### Key Characteristics
- **Type**: Physical/Natural simulation sort
- **In-place**: Depends on implementation
- **Stable**: N/A (only sorts positive integers)
- **Unique**: Simulates physical phenomenon

## 2. Mathematical Foundation

### 2.1 Physical Model

Imagine an abacus:
- Rows represent numbers (each bead = 1)
- Columns are vertical poles
- Gravity pulls beads down

Example for `[3, 1, 4, 2]`:
```
Before gravity:    After gravity:
  1 2 3 4            1 2 3 4
  o o o .            o o o o   ← 4
  o . . .            o o o .   ← 3
  o o o o            o o . .   ← 2
  o o . .            o . . .   ← 1
```

### 2.2 Mathematical Formulation

For array A with n elements:
- Create a grid of $n \times max(A)$
- Count beads falling in each column
- Read row sums for sorted result

## 3. Algorithm Description

### 3.1 Pseudocode

```
BEAD_SORT(A)
    n ← length(A)
    max ← maximum(A)
    
    // Create bead grid
    grid ← 2D array [n][max] initialized to 0
    
    // Place beads
    for i ← 0 to n - 1 do
        for j ← 0 to A[i] - 1 do
            grid[i][j] ← 1
    
    // Apply gravity (count beads per column)
    for col ← 0 to max - 1 do
        count ← 0
        for row ← 0 to n - 1 do
            count ← count + grid[row][col]
            grid[row][col] ← 0
        
        // Drop beads
        for row ← n - 1 down to n - count do
            grid[row][col] ← 1
    
    // Read result
    for i ← 0 to n - 1 do
        A[i] ← sum of grid[i]
```

### 3.2 Step-by-Step Example

Sorting `[4, 2, 3, 1]`:

```
Initial array: [4, 2, 3, 1]

Step 1: Create bead representation
Row 0: oooo  (4 beads)
Row 1: oo    (2 beads)
Row 2: ooo   (3 beads)
Row 3: o     (1 bead)

As grid:
Col:  0 1 2 3
Row 0: 1 1 1 1
Row 1: 1 1 0 0
Row 2: 1 1 1 0
Row 3: 1 0 0 0

Step 2: Apply gravity (beads fall down)
Count per column: [4, 3, 2, 1]

After gravity:
Col:  0 1 2 3
Row 0: 1 1 1 1  → 4
Row 1: 1 1 1 0  → 3
Row 2: 1 1 0 0  → 2
Row 3: 1 0 0 0  → 1

Result: [4, 3, 2, 1] (descending)
Reverse for ascending: [1, 2, 3, 4]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Time** | O(n × m) where m = max(arr) |
| **Space** | O(n × m) |

### 4.1 Hardware Implementation

With parallel hardware:
- Time: O(1) for gravity simulation
- Space: O(n × m) physical beads

This makes it theoretically the fastest sorting algorithm with appropriate hardware!

## 5. Implementation

```rust
pub fn bead_sort(arr: &mut [usize]) {
    if arr.is_empty() {
        return;
    }

    let max = *arr.iter().max().unwrap();
    
    // Create bead grid
    let mut beads = vec![vec![0u8; max]; arr.len()];
    
    // Place beads
    for (i, &val) in arr.iter().enumerate() {
        for j in 0..val {
            beads[i][j] = 1;
        }
    }
    
    // Apply gravity
    for col in 0..max {
        let mut count = 0;
        for row in 0..arr.len() {
            count += beads[row][col] as usize;
            beads[row][col] = 0;
        }
        
        // Drop beads to bottom
        for row in (arr.len() - count)..arr.len() {
            beads[row][col] = 1;
        }
    }
    
    // Read sorted values (descending order)
    for (i, row) in beads.iter().enumerate() {
        arr[i] = row.iter().map(|&b| b as usize).sum();
    }
    
    // Reverse for ascending order
    arr.reverse();
}
```

## 6. Limitations

| Limitation | Impact |
|------------|--------|
| Positive integers only | Cannot sort negatives or floats |
| Memory intensive | O(n × max) space |
| Large values | Impractical for large numbers |
| Not comparison-based | Limited theoretical interest |

## 7. Physical Implementations

### 7.1 Abacus
Traditional counting tool naturally performs bead sort.

### 7.2 Spaghetti Sort
Similar concept using spaghetti strands of different lengths:
1. Hold spaghetti vertically
2. Lower onto table
3. Pick tallest repeatedly

## 8. Applications

- **Education**: Understanding physical analogies in CS
- **Specialized hardware**: Parallel sorting with custom chips
- **Small integer domains**: When max value is bounded

## 9. References

1. Arulanandham, J. J. (2002). "Implementing Bead Sort: A Physical Simulation of the Gravity Sort Algorithm".
2. Guan, D. J. (2020). "Analysis of Bead Sort Algorithm".

## 10. Source Code

**Implementation**: [src/sorting/bead_sort.rs](../../src/sorting/bead_sort.rs)
