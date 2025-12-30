# Matrix Chain Multiplication

## 1. Overview

The Matrix Chain Multiplication problem finds the optimal way to parenthesize a sequence of matrices to minimize the total number of scalar multiplications. This is a classic interval DP problem.

**File**: `src/dynamic_programming/matrix_chain_multiply.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given matrices $A_1, A_2, \ldots, A_n$ with dimensions $d_0 \times d_1, d_1 \times d_2, \ldots, d_{n-1} \times d_n$, find the parenthesization that minimizes scalar multiplications.

### 2.2 Key Insight

Multiplying a $p \times q$ matrix by a $q \times r$ matrix requires $p \cdot q \cdot r$ scalar multiplications.

### 2.3 Recurrence

Let $m[i][j]$ = minimum cost to multiply matrices $A_i$ through $A_j$.

$$m[i][j] = \begin{cases}
0 & \text{if } i = j \\
\min_{i \leq k < j}(m[i][k] + m[k+1][j] + d_{i-1} \cdot d_k \cdot d_j) & \text{otherwise}
\end{cases}$$

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION matrix_chain_multiply(dimensions)
    n ← length(dimensions) - 1  // Number of matrices
    m[1..n][1..n] ← 0
    
    FOR len ← 2 TO n DO
        FOR i ← 1 TO n - len + 1 DO
            j ← i + len - 1
            m[i][j] ← ∞
            FOR k ← i TO j - 1 DO
                cost ← m[i][k] + m[k+1][j] + d[i-1] × d[k] × d[j]
                m[i][j] ← MIN(m[i][j], cost)
            END FOR
        END FOR
    END FOR
    
    RETURN m[1][n]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: dimensions = [10, 30, 5, 60] → Matrices: 10×30, 30×5, 5×60

| Parenthesization | Computation |
|-----------------|-------------|
| (A₁A₂)A₃ | (10×30×5) + (10×5×60) = 1500 + 3000 = 4500 |
| A₁(A₂A₃) | (30×5×60) + (10×30×60) = 9000 + 18000 = 27000 |

**Optimal**: (A₁A₂)A₃ with cost **4500**

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n³)**: Three nested loops

### 4.2 Space Complexity
- **O(n²)**: 2D DP table

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn matrix_chain_multiply(
    dimensions: Vec<usize>
) -> Result<usize, MatrixChainMultiplicationError> {
    if dimensions.is_empty() {
        return Err(MatrixChainMultiplicationError::EmptyDimensions);
    }
    if dimensions.len() == 1 {
        return Err(MatrixChainMultiplicationError::InsufficientDimensions);
    }

    let mut dp = vec![vec![0; dimensions.len()]; dimensions.len()];

    (2..dimensions.len()).for_each(|chain_len| {
        (0..dimensions.len() - chain_len).for_each(|start| {
            let end = start + chain_len;
            dp[start][end] = (start + 1..end)
                .map(|split| {
                    dp[start][split] + dp[split][end] 
                    + dimensions[start] * dimensions[split] * dimensions[end]
                })
                .min()
                .unwrap_or(usize::MAX);
        });
    });

    Ok(dp[0][dimensions.len() - 1])
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty dimensions | Error |
| Single dimension | Error |
| Two dimensions (one matrix) | 0 |

## 6. Applications

1. **Linear Algebra Libraries**: Optimizing chain multiplications
2. **Machine Learning**: Neural network computations
3. **Computer Graphics**: Transformation matrix chains

## 7. Visualization

```
Matrices: A₁(10×30), A₂(30×5), A₃(5×60)

         Optimal: ((A₁ × A₂) × A₃)
         
         Step 1: A₁ × A₂ = 10×30×5 = 1500 ops
                 Result: 10×5 matrix
         
         Step 2: Result × A₃ = 10×5×60 = 3000 ops
         
         Total: 4500 operations
```

## 8. References

1. Cormen, T. H. et al. (2009). *Introduction to Algorithms*, Chapter 15.2
2. [Wikipedia - Matrix chain multiplication](https://en.wikipedia.org/wiki/Matrix_chain_multiplication)
