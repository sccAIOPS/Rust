# Snail (Spiral Matrix Traversal)

## 1. Overview

The Snail problem traverses a 2D matrix in a spiral (clockwise) pattern, starting from the top-left corner. While not a traditional DP problem, it demonstrates careful boundary manipulation and directional iteration.

**File**: `src/dynamic_programming/snail.rs`

## 2. Problem Definition

Given an $n \times m$ matrix, return elements in spiral order:

```
Input:           Output:
1  2  3  4       [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]
5  6  7  8   →   
9  10 11 12      Spiral: → ↓ ← ↑ (repeat)
```

## 3. Algorithm Description

### 3.1 Layer-by-Layer Approach

Process the matrix in concentric rectangular layers:

```
Layer 0: Outer boundary
Layer 1: Inner boundary
...
```

### 3.2 Pseudocode

```
FUNCTION snail(matrix)
    result ← []
    IF matrix is empty THEN RETURN result
    
    top ← 0, bottom ← rows-1
    left ← 0, right ← cols-1
    
    WHILE top ≤ bottom AND left ≤ right DO
        // Right: traverse top row
        FOR col ← left TO right DO
            result.append(matrix[top][col])
        END FOR
        top ← top + 1
        
        // Down: traverse right column
        FOR row ← top TO bottom DO
            result.append(matrix[row][right])
        END FOR
        right ← right - 1
        
        // Left: traverse bottom row (if exists)
        IF top ≤ bottom THEN
            FOR col ← right DOWNTO left DO
                result.append(matrix[bottom][col])
            END FOR
            bottom ← bottom - 1
        END IF
        
        // Up: traverse left column (if exists)
        IF left ≤ right THEN
            FOR row ← bottom DOWNTO top DO
                result.append(matrix[row][left])
            END FOR
            left ← left + 1
        END IF
    END WHILE
    
    RETURN result
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**:
```
1  2  3
4  5  6
7  8  9
```

**Execution**:
```
Initial: top=0, bottom=2, left=0, right=2

Round 1:
  Right (row 0): 1, 2, 3       top → 1
  Down (col 2): 6, 9           right → 1
  Left (row 2): 8, 7           bottom → 1
  Up (col 0): 4                left → 1

Round 2:
  Right (row 1, col 1): 5      top → 2
  (Loop ends: top > bottom)

Result: [1, 2, 3, 6, 9, 8, 7, 4, 5]
```

**Visual**:
```
→ → → 
    ↓
← ← ↓
↑   ↓
↑ ← ←
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(m × n)** - Visit each cell exactly once

### 4.2 Space Complexity
- **O(m × n)** for output (O(1) extra space)

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn snail<T: Clone>(matrix: &[Vec<T>]) -> Vec<T> {
    if matrix.is_empty() || matrix[0].is_empty() {
        return vec![];
    }
    
    let mut result = Vec::new();
    let (mut top, mut bottom) = (0i32, matrix.len() as i32 - 1);
    let (mut left, mut right) = (0i32, matrix[0].len() as i32 - 1);
    
    while top <= bottom && left <= right {
        // Right
        for col in left..=right {
            result.push(matrix[top as usize][col as usize].clone());
        }
        top += 1;
        
        // Down
        for row in top..=bottom {
            result.push(matrix[row as usize][right as usize].clone());
        }
        right -= 1;
        
        // Left
        if top <= bottom {
            for col in (left..=right).rev() {
                result.push(matrix[bottom as usize][col as usize].clone());
            }
            bottom -= 1;
        }
        
        // Up
        if left <= right {
            for row in (top..=bottom).rev() {
                result.push(matrix[row as usize][left as usize].clone());
            }
            left += 1;
        }
    }
    
    result
}
```

### 5.2 Direction-Based Approach

Alternative using direction vectors:

```rust
const DIRECTIONS: [(i32, i32); 4] = [(0, 1), (1, 0), (0, -1), (-1, 0)];

fn snail_with_directions<T: Clone + Default>(matrix: &[Vec<T>]) -> Vec<T> {
    let rows = matrix.len();
    let cols = matrix[0].len();
    let mut visited = vec![vec![false; cols]; rows];
    let mut result = Vec::with_capacity(rows * cols);
    
    let (mut r, mut c, mut dir) = (0i32, 0i32, 0);
    
    for _ in 0..(rows * cols) {
        result.push(matrix[r as usize][c as usize].clone());
        visited[r as usize][c as usize] = true;
        
        let (nr, nc) = (r + DIRECTIONS[dir].0, c + DIRECTIONS[dir].1);
        
        if nr < 0 || nr >= rows as i32 || nc < 0 || nc >= cols as i32 
           || visited[nr as usize][nc as usize] {
            dir = (dir + 1) % 4;
        }
        
        r += DIRECTIONS[dir].0;
        c += DIRECTIONS[dir].1;
    }
    
    result
}
```

### 5.3 Edge Cases

| Case | Input | Output |
|------|-------|--------|
| Empty | [] | [] |
| Single element | [[1]] | [1] |
| Single row | [[1,2,3]] | [1,2,3] |
| Single column | [[1],[2],[3]] | [1,2,3] |
| 2×2 | [[1,2],[3,4]] | [1,2,4,3] |

## 6. Variants

### 6.1 Counter-Clockwise Spiral

Reverse direction order: → ↓ ← ↑ becomes → ↑ ← ↓

```rust
// Change direction order to: Right, Up, Left, Down
```

### 6.2 Spiral Matrix II (Generate)

Fill matrix with 1 to n² in spiral order:

```rust
fn generate_spiral(n: usize) -> Vec<Vec<i32>> {
    let mut matrix = vec![vec![0; n]; n];
    let mut num = 1;
    // ... spiral fill logic
    matrix
}
```

### 6.3 Spiral Matrix III (From Center)

Start from given position, expand outward.

### 6.4 Anti-Diagonal Traversal

Different pattern but similar technique:
```
1 2 3     Output: [1, 2, 4, 7, 5, 3, 6, 8, 9]
4 5 6
7 8 9
```

## 7. Applications

1. **Image Processing**: Spiral scan patterns
2. **Cache Optimization**: Space-filling curves
3. **Data Visualization**: Spiral graphs
4. **Game Development**: Procedural generation

## 8. Related Problems

| Problem | Description |
|---------|-------------|
| Spiral Matrix II | Generate n×n spiral |
| Spiral Matrix III | Spiral from any starting point |
| Diagonal Traverse | Diagonal zigzag pattern |
| Rotate Image | 90° rotation |

## 9. Debugging Tips

1. **Print boundaries** at each iteration
2. **Visualize** with colors for each direction
3. **Test rectangular** (non-square) matrices
4. **Check single row/column** separately

## 10. Performance Comparison

| Method | Time | Space | Readability |
|--------|------|-------|-------------|
| Boundary shrinking | O(mn) | O(1) | Good |
| Direction vectors | O(mn) | O(mn) | Medium |
| Recursive layers | O(mn) | O(min(m,n)) | Complex |

## 11. Mathematical Note

Number of complete spirals in n×n matrix:
- Layers = ⌈n/2⌉
- Elements per layer decreases by 8 each time

## 12. References

1. [LeetCode Problem 54](https://leetcode.com/problems/spiral-matrix/)
2. [LeetCode Problem 59](https://leetcode.com/problems/spiral-matrix-ii/)
3. [LeetCode Problem 885](https://leetcode.com/problems/spiral-matrix-iii/)
