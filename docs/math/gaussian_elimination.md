# Gaussian Elimination

## 1. Overview

**Gaussian elimination** is a method for solving systems of linear equations by transforming the augmented matrix into row echelon form. It's fundamental to linear algebra and numerical analysis.

**File**: `src/math/gaussian_elimination.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Solve the system $Ax = b$ where:
- $A$ is an $n \times n$ matrix of coefficients
- $x$ is the unknown vector
- $b$ is the constant vector

### 2.2 Matrix Operations

Three elementary row operations:
1. **Swap**: Exchange two rows
2. **Scale**: Multiply a row by a non-zero constant
3. **Add**: Add a multiple of one row to another

### 2.3 Row Echelon Form

A matrix is in row echelon form if:
- All zero rows are at the bottom
- Leading coefficient (pivot) of each row is to the right of the pivot above

## 3. Algorithm Description

### 3.1 Two Phases

1. **Forward elimination**: Transform to row echelon form
2. **Back substitution**: Solve starting from bottom row

### 3.2 Pseudocode

```
function gaussian_elimination(A, b):
    # Create augmented matrix [A|b]
    M ← augment(A, b)
    n ← number of rows
    
    # Forward elimination
    for i from 0 to n-1:
        # Find pivot
        max_row ← argmax(|M[j,i]| for j from i to n-1)
        swap rows i and max_row
        
        # Eliminate below
        for j from i+1 to n-1:
            factor ← M[j,i] / M[i,i]
            for k from i to n:
                M[j,k] ← M[j,k] - factor × M[i,k]
    
    # Back substitution
    x ← new array of size n
    for i from n-1 down to 0:
        x[i] ← (M[i,n] - sum(M[i,j] × x[j] for j from i+1 to n-1)) / M[i,i]
    
    return x
```

### 3.3 Step-by-Step Example

Solve:
$$\begin{cases} 2x + y - z = 8 \\ -3x - y + 2z = -11 \\ -2x + y + 2z = -3 \end{cases}$$

**Augmented matrix**:
$$\left[\begin{array}{ccc|c} 2 & 1 & -1 & 8 \\ -3 & -1 & 2 & -11 \\ -2 & 1 & 2 & -3 \end{array}\right]$$

**Forward elimination**:

$R_2 \leftarrow R_2 + \frac{3}{2}R_1$:
$$\left[\begin{array}{ccc|c} 2 & 1 & -1 & 8 \\ 0 & 0.5 & 0.5 & 1 \\ -2 & 1 & 2 & -3 \end{array}\right]$$

$R_3 \leftarrow R_3 + R_1$:
$$\left[\begin{array}{ccc|c} 2 & 1 & -1 & 8 \\ 0 & 0.5 & 0.5 & 1 \\ 0 & 2 & 1 & 5 \end{array}\right]$$

$R_3 \leftarrow R_3 - 4R_2$:
$$\left[\begin{array}{ccc|c} 2 & 1 & -1 & 8 \\ 0 & 0.5 & 0.5 & 1 \\ 0 & 0 & -1 & 1 \end{array}\right]$$

**Back substitution**:
- $-z = 1 \Rightarrow z = -1$
- $0.5y + 0.5(-1) = 1 \Rightarrow y = 3$
- $2x + 3 - (-1) = 8 \Rightarrow x = 2$

**Solution**: $(x, y, z) = (2, 3, -1)$

## 4. Complexity Analysis

### 4.1 Time Complexity

$$O(n^3)$$

- Forward elimination: $\sum_{k=1}^{n} (n-k)^2 \approx \frac{n^3}{3}$
- Back substitution: $O(n^2)$

### 4.2 Space Complexity

$O(n^2)$ for the augmented matrix (can be done in-place).

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn gaussian_elimination(matrix: &mut [Vec<f32>]) -> Vec<f32> {
    let size = matrix.len();
    assert_eq!(size, matrix[0].len() - 1);

    // Forward elimination
    for i in 0..size - 1 {
        for j in i..size - 1 {
            echelon(matrix, i, j);
        }
    }

    // Back substitution (from bottom up)
    for i in (1..size).rev() {
        eliminate(matrix, i);
    }

    // Extract solution
    let mut result: Vec<f32> = vec![0f32; size];
    for i in 0..size {
        result[i] = matrix[i][size] / matrix[i][i];
    }
    result
}

fn echelon(matrix: &mut [Vec<f32>], i: usize, j: usize) {
    let size = matrix.len();
    if matrix[i][i] != 0f32 {
        let factor = matrix[j + 1][i] / matrix[i][i];
        for k in i..=size {
            matrix[j + 1][k] -= factor * matrix[i][k];
        }
    }
}
```

### 5.2 Numerical Stability

**Partial pivoting**: At each step, swap with the row having the largest absolute pivot value:

```rust
fn find_pivot(matrix: &[Vec<f32>], col: usize) -> usize {
    (col..matrix.len())
        .max_by(|&a, &b| 
            matrix[a][col].abs().partial_cmp(&matrix[b][col].abs()).unwrap()
        )
        .unwrap()
}
```

### 5.3 Edge Cases

| Situation | Detection | Meaning |
|-----------|-----------|---------|
| Zero pivot | matrix[i][i] = 0 | Singular or needs pivoting |
| All zeros in row | All coefficients = 0 | Infinitely many solutions |
| Inconsistent | 0 = non-zero | No solution |

## 6. Applications

### 6.1 Direct Uses

1. **Solving linear systems**: $Ax = b$
2. **Computing determinants**: Product of pivots
3. **Finding inverse**: Solve $AX = I$
4. **LU decomposition**: Factor into $A = LU$

### 6.2 Engineering Applications

- Circuit analysis
- Structural engineering
- Machine learning (least squares)
- Computer graphics (transformations)

## 7. Related Algorithms

| Algorithm | Use Case | Complexity |
|-----------|----------|------------|
| LU Decomposition | Multiple RHS | O(n³) once, O(n²) per solve |
| Cholesky | Symmetric positive definite | O(n³/3) |
| QR Decomposition | Least squares | O(n³) |
| Iterative methods | Sparse systems | Varies |

## 8. References

1. Golub, G. H. & Van Loan, C. F. "Matrix Computations"
2. [Wikipedia: Gaussian Elimination](https://en.wikipedia.org/wiki/Gaussian_elimination)
