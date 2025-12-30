# Minimum Cost Path

## 1. Overview

The Minimum Cost Path problem finds the path with minimum total cost from top-left to bottom-right in a grid, moving only right or down. This is a classic grid DP problem demonstrating optimal substructure.

**File**: `src/dynamic_programming/minimum_cost_path.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an $m \times n$ matrix $cost$ where $cost[i][j]$ is the cost to enter cell $(i,j)$.

Find: $\min \sum_{(i,j) \in path} cost[i][j]$

where path goes from $(0,0)$ to $(m-1, n-1)$ using only right and down moves.

### 2.2 Recurrence

Let $dp[i][j]$ = minimum cost to reach cell $(i,j)$

$$dp[i][j] = cost[i][j] + \min(dp[i-1][j], dp[i][j-1])$$

**Boundary conditions**:
- $dp[0][0] = cost[0][0]$
- $dp[0][j] = dp[0][j-1] + cost[0][j]$ (first row)
- $dp[i][0] = dp[i-1][0] + cost[i][0]$ (first column)

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION min_cost_path(cost)
    m ← rows(cost)
    n ← cols(cost)
    dp[0..m][0..n]
    
    // Base case
    dp[0][0] ← cost[0][0]
    
    // First row
    FOR j ← 1 TO n-1 DO
        dp[0][j] ← dp[0][j-1] + cost[0][j]
    END FOR
    
    // First column
    FOR i ← 1 TO m-1 DO
        dp[i][0] ← dp[i-1][0] + cost[i][0]
    END FOR
    
    // Fill rest
    FOR i ← 1 TO m-1 DO
        FOR j ← 1 TO n-1 DO
            dp[i][j] ← cost[i][j] + min(dp[i-1][j], dp[i][j-1])
        END FOR
    END FOR
    
    RETURN dp[m-1][n-1]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**:
```
cost = [[1, 2, 3],
        [4, 8, 2],
        [1, 5, 3]]
```

**DP Table Construction**:
```
Initial:     After first row/col:    Final:
[1][?][?]    [1][3][6]               [1][3][ 6]
[?][?][?]    [5][?][?]               [5][11][8]
[?][?][?]    [6][?][?]               [6][11][11]

dp[1][1] = cost[1][1] + min(dp[0][1], dp[1][0])
         = 8 + min(3, 5) = 11

dp[1][2] = cost[1][2] + min(dp[0][2], dp[1][1])
         = 2 + min(6, 11) = 8

dp[2][1] = cost[2][1] + min(dp[1][1], dp[2][0])
         = 5 + min(11, 6) = 11

dp[2][2] = cost[2][2] + min(dp[1][2], dp[2][1])
         = 3 + min(8, 11) = 11
```

**Result**: 11

**Optimal Path**: (0,0)→(0,1)→(0,2)→(1,2)→(2,2) = 1+2+3+2+3 = 11

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(m × n)** - Visit each cell once

### 4.2 Space Complexity
- **O(m × n)** with 2D DP table
- **O(n)** with space optimization (single row)

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn minimum_cost_path(cost: Vec<Vec<usize>>) -> Result<usize, &'static str> {
    if cost.is_empty() || cost[0].is_empty() {
        return Err("The grid is empty");
    }
    
    let rows = cost.len();
    let cols = cost[0].len();
    let mut dp = vec![vec![0; cols]; rows];
    
    dp[0][0] = cost[0][0];
    
    // First row
    for j in 1..cols {
        dp[0][j] = dp[0][j-1] + cost[0][j];
    }
    
    // First column
    for i in 1..rows {
        dp[i][0] = dp[i-1][0] + cost[i][0];
    }
    
    // Rest of grid
    for i in 1..rows {
        for j in 1..cols {
            dp[i][j] = cost[i][j] + dp[i-1][j].min(dp[i][j-1]);
        }
    }
    
    Ok(dp[rows-1][cols-1])
}
```

### 5.2 Space-Optimized Version

```rust
pub fn minimum_cost_path_optimized(cost: Vec<Vec<usize>>) -> usize {
    let cols = cost[0].len();
    let mut dp = vec![0; cols];
    
    dp[0] = cost[0][0];
    for j in 1..cols {
        dp[j] = dp[j-1] + cost[0][j];
    }
    
    for row in cost.iter().skip(1) {
        dp[0] += row[0];
        for j in 1..cols {
            dp[j] = row[j] + dp[j].min(dp[j-1]);
        }
    }
    
    dp[cols-1]
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty grid | Return error |
| Single cell | Return that cell's cost |
| Single row | Sum all cells |
| Single column | Sum all cells |

## 6. Path Reconstruction

To get actual path, track decisions:

```rust
fn reconstruct_path(dp: &[Vec<usize>], cost: &[Vec<usize>]) -> Vec<(usize, usize)> {
    let (mut i, mut j) = (dp.len() - 1, dp[0].len() - 1);
    let mut path = vec![(i, j)];
    
    while i > 0 || j > 0 {
        if i == 0 {
            j -= 1;
        } else if j == 0 {
            i -= 1;
        } else if dp[i-1][j] < dp[i][j-1] {
            i -= 1;
        } else {
            j -= 1;
        }
        path.push((i, j));
    }
    
    path.reverse();
    path
}
```

## 7. Variants

### 7.1 Extended Movement

| Variant | Allowed Moves |
|---------|---------------|
| 4-Direction | Up, Down, Left, Right (use Dijkstra) |
| 8-Direction | Include diagonals |
| Knight moves | Chess knight pattern |

### 7.2 Maximum Cost Path

Same recurrence but with `max` instead of `min`:

$$dp[i][j] = cost[i][j] + \max(dp[i-1][j], dp[i][j-1])$$

### 7.3 Obstacles

Some cells blocked (infinite cost):

```rust
if cost[i][j] == BLOCKED {
    dp[i][j] = usize::MAX;
} else {
    // normal calculation
}
```

## 8. Applications

1. **Robotics**: Path planning in grid environments
2. **Image Processing**: Seam carving for resizing
3. **Sequence Alignment**: DNA/protein alignment
4. **Game AI**: Movement cost calculation

## 9. Visual Representation

```
Start →[1]→[2]→[3]
        ↓       ↓
       [4] [8]→[2]
        ↓       ↓
       [1]→[5] [3]← End

Optimal: 1→2→3→2→3 = 11
```

## 10. References

1. [LeetCode Problem 64](https://leetcode.com/problems/minimum-path-sum/)
2. Cormen, T.H. "Introduction to Algorithms" - Dynamic Programming chapter
