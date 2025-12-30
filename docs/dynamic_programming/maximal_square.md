# Maximal Square

## 1. Overview

The Maximal Square problem finds the largest square containing only 1s in a binary matrix. This demonstrates how local decisions propagate to global optimum in grid DP.

**File**: `src/dynamic_programming/maximal_square.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given binary matrix $M$ of size $m \times n$ where $M[i][j] \in \{0, 1\}$.

Find: Maximum area of a square submatrix containing only 1s.

### 2.2 Key Insight

A cell can be bottom-right corner of a square of side $k$ only if:
- The cell above has a square of at least side $k-1$
- The cell to the left has a square of at least side $k-1$
- The cell diagonally above-left has a square of at least side $k-1$

### 2.3 Recurrence

Let $dp[i][j]$ = side length of largest square with $(i,j)$ as bottom-right corner.

$$dp[i][j] = \begin{cases}
0 & \text{if } M[i][j] = 0 \\
1 + \min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) & \text{if } M[i][j] = 1
\end{cases}$$

**Answer**: $\max_{i,j} dp[i][j]^2$

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION maximal_square(matrix)
    m ← rows(matrix)
    n ← cols(matrix)
    max_side ← 0
    dp[0..m][0..n]
    
    FOR i ← 0 TO m-1 DO
        FOR j ← 0 TO n-1 DO
            IF matrix[i][j] = 1 THEN
                IF i = 0 OR j = 0 THEN
                    dp[i][j] ← 1
                ELSE
                    dp[i][j] ← 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
                END IF
                max_side ← max(max_side, dp[i][j])
            ELSE
                dp[i][j] ← 0
            END IF
        END FOR
    END FOR
    
    RETURN max_side * max_side
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**:
```
[1, 0, 1, 0, 0]
[1, 0, 1, 1, 1]
[1, 1, 1, 1, 1]
[1, 0, 0, 1, 0]
```

**DP Table**:
```
[1, 0, 1, 0, 0]    First row/col: copy if 1
[1, 0, 1, 1, 1]
[1, 1, 1, 2, 2]    dp[2][3] = 1 + min(1,1,1) = 2
[1, 0, 0, 1, 0]    dp[2][4] = 1 + min(2,1,1) = 2
```

**Finding max_side**:
- dp[2][3] = 2 (square from (1,2) to (2,3))
- dp[2][4] = 2 (square from (1,3) to (2,4))

**Result**: $2^2 = 4$

**Visual**:
```
[1, 0, 1, 0, 0]
[1, 0,{1, 1},1]   ← 2×2 square
[1, 1,{1, 1},1]
[1, 0, 0, 1, 0]
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(m × n)** - Single pass through matrix

### 4.2 Space Complexity
- **O(m × n)** with separate DP table
- **O(n)** with rolling array
- **O(1)** with in-place modification

## 5. Implementation Notes

### 5.1 Rust Implementation (In-Place)

```rust
pub fn maximal_square(matrix: &mut Vec<Vec<i32>>) -> i32 {
    if matrix.is_empty() || matrix[0].is_empty() {
        return 0;
    }
    
    let rows = matrix.len();
    let cols = matrix[0].len();
    let mut max_side = 0;
    
    for i in 0..rows {
        for j in 0..cols {
            if matrix[i][j] == 1 {
                if i == 0 || j == 0 {
                    // Edge case: first row/col
                    max_side = max_side.max(1);
                } else {
                    let min_neighbor = matrix[i-1][j]
                        .min(matrix[i][j-1])
                        .min(matrix[i-1][j-1]);
                    matrix[i][j] = 1 + min_neighbor;
                    max_side = max_side.max(matrix[i][j]);
                }
            }
        }
    }
    
    max_side * max_side
}
```

### 5.2 Space-Optimized (O(n))

```rust
pub fn maximal_square_optimized(matrix: &[Vec<i32>]) -> i32 {
    let cols = matrix[0].len();
    let mut dp = vec![0; cols + 1];
    let mut max_side = 0;
    let mut prev;
    
    for row in matrix {
        prev = 0;
        for j in 1..=cols {
            let temp = dp[j];
            if row[j-1] == 1 {
                dp[j] = 1 + dp[j].min(dp[j-1]).min(prev);
                max_side = max_side.max(dp[j]);
            } else {
                dp[j] = 0;
            }
            prev = temp;
        }
    }
    
    max_side * max_side
}
```

### 5.3 Edge Cases

| Case | Result |
|------|--------|
| Empty matrix | 0 |
| All zeros | 0 |
| All ones | min(m,n)² |
| Single 1 | 1 |
| Single row/col | 1 if any 1 exists |

## 6. Why Min of Three?

Consider wanting a 3×3 square at position (i,j):

```
[?][?][?]
[?][?][?]
[?][?][X] ← (i,j)
```

For this to be valid:
- Top neighbor must have 2×2 (covers top-right region)
- Left neighbor must have 2×2 (covers bottom-left region)
- Diagonal must have 2×2 (covers top-left region)

The **minimum** determines the bottleneck!

## 7. Related Problems

| Problem | Difference |
|---------|------------|
| Maximal Rectangle | Find largest rectangle, not square |
| Count Square Submatrices | Count all squares, not just largest |
| Largest Plus Sign | Find largest + shape |

## 8. Applications

1. **Image Processing**: Finding square regions of interest
2. **Urban Planning**: Identifying buildable square plots
3. **Game Development**: Collision detection, terrain analysis
4. **Data Storage**: Block allocation in storage systems

## 9. Extensions

### 9.1 Maximal Rectangle

Use histogram method:
```rust
// For each row, compute histogram heights
// Apply largest rectangle in histogram
// Time: O(m × n), Space: O(n)
```

### 9.2 Count All Squares

Sum all dp values instead of taking max:
```rust
let total_squares: i32 = dp.iter().flatten().sum();
```

## 10. Visualization of DP Propagation

```
Input:      After row 1:   After row 2:   After row 3:
1 1 1       1 1 1          1 1 1          1 1 1
1 1 1   →   1 2 2      →   1 2 2      →   1 2 2
1 1 1       · · ·          1 2 3          1 2 3

The "3" at bottom-right proves a 3×3 square exists!
```

## 11. References

1. [LeetCode Problem 221](https://leetcode.com/problems/maximal-square/)
2. [GeeksforGeeks - Maximum size square sub-matrix](https://www.geeksforgeeks.org/maximum-size-sub-matrix-with-all-1s-in-a-binary-matrix/)
