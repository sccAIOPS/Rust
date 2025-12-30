# Matrix Operations

## 1. Overview

A comprehensive **matrix operations** library implementing basic linear algebra operations including creation, indexing, arithmetic operations, and transformations for generic numeric types.

**File**: `src/math/matrix_ops.rs`

## 2. Mathematical Foundation

### 2.1 Matrix Definition

An $m \times n$ matrix $A$ is a rectangular array:

$$A = \begin{bmatrix} 
a_{11} & a_{12} & \cdots & a_{1n} \\
a_{21} & a_{22} & \cdots & a_{2n} \\
\vdots & \vdots & \ddots & \vdots \\
a_{m1} & a_{m2} & \cdots & a_{mn}
\end{bmatrix}$$

### 2.2 Operations

| Operation | Definition |
|-----------|------------|
| Addition | $(A + B)_{ij} = a_{ij} + b_{ij}$ |
| Scalar multiplication | $(cA)_{ij} = c \cdot a_{ij}$ |
| Matrix multiplication | $(AB)_{ij} = \sum_k a_{ik} b_{kj}$ |
| Transpose | $(A^T)_{ij} = a_{ji}$ |

## 3. Implementation

### 3.1 Matrix Structure

```rust
pub struct Matrix<T: MatrixElement> {
    data: Vec<T>,
    rows: usize,
    cols: usize,
}
```

Uses row-major storage: element at (i, j) stored at index `cols * i + j`.

### 3.2 Matrix Creation

```rust
// From vector with dimensions
let m = Matrix::new(vec![1, 2, 3, 4], 2, 2);

// Using macro
let m = matrix![
    [1, 2, 3],
    [4, 5, 6]
];

// Zero matrix
let zero = Matrix::<i32>::zero(3, 3);

// Identity matrix
let identity = Matrix::<i32>::identity(3);
```

### 3.3 Indexing

```rust
impl<T: MatrixElement> Index<[usize; 2]> for Matrix<T> {
    fn index(&self, index: [usize; 2]) -> &Self::Output {
        let [i, j] = index;
        &self.data[(self.cols * i) + j]
    }
}
```

Usage:
```rust
let val = matrix[[i, j]];     // Read
matrix[[i, j]] = new_val;     // Write
```

### 3.4 Addition

```rust
impl<T: MatrixElement> Add<&Matrix<T>> for &Matrix<T> {
    fn add(self, rhs: &Matrix<T>) -> Self::Output {
        // Requires identical dimensions
        let mut result = Matrix::zero(self.rows, self.cols);
        for i in 0..self.rows {
            for j in 0..self.cols {
                result[[i, j]] = self[[i, j]] + rhs[[i, j]];
            }
        }
        result
    }
}
```

### 3.5 Transpose

```rust
pub fn transpose(&self) -> Self {
    let mut result = Matrix::zero(self.cols, self.rows);
    for i in 0..self.rows {
        for j in 0..self.cols {
            result[[i, j]] = self[[j, i]];
        }
    }
    result
}
```

## 4. Supported Types

The library uses a trait bound for matrix elements:

```rust
pub trait MatrixElement:
    Add<Output = Self> + Sub<Output = Self> + Mul<Output = Self> 
    + AddAssign + Copy + From<u8>
{}

// Implemented for:
matrix_element_type_def!(i16, i32, i64, i128, u8, u16, u32, u128, f32, f64);
```

## 5. Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Creation | O(mn) | O(mn) |
| Indexing | O(1) | O(1) |
| Addition | O(mn) | O(mn) |
| Subtraction | O(mn) | O(mn) |
| Multiplication | O(mnp) | O(mp) |
| Transpose | O(mn) | O(mn) |
| Scalar mult | O(mn) | O(mn) |

## 6. Usage Examples

### 6.1 Basic Operations

```rust
let a = matrix![[1, 2], [3, 4]];
let b = matrix![[5, 6], [7, 8]];

// Addition
let sum = &a + &b;  // [[6, 8], [10, 12]]

// Subtraction  
let diff = &a - &b;  // [[-4, -4], [-4, -4]]

// Scalar multiplication
let scaled = 2 * &a;  // [[2, 4], [6, 8]]

// Matrix multiplication
let product = &a * &b;  // [[19, 22], [43, 50]]

// Transpose
let transposed = a.transpose();  // [[1, 3], [2, 4]]
```

### 6.2 Special Matrices

```rust
// Identity matrix
let i3 = Matrix::<i32>::identity(3);
// [[1, 0, 0], [0, 1, 0], [0, 0, 1]]

// Zero matrix
let zero = Matrix::<f64>::zero(2, 3);
// [[0, 0, 0], [0, 0, 0]]
```

## 7. Properties

### 7.1 Matrix Multiplication

- Not commutative: $AB \neq BA$ in general
- Associative: $(AB)C = A(BC)$
- Distributive: $A(B + C) = AB + AC$

### 7.2 Identity Matrix

- $AI = IA = A$
- $I^n = I$

### 7.3 Transpose

- $(A^T)^T = A$
- $(A + B)^T = A^T + B^T$
- $(AB)^T = B^T A^T$

## 8. Applications

1. **Graphics**: 3D transformations, projections
2. **Machine learning**: Neural network weights
3. **Physics**: Systems of equations, rotations
4. **Graph theory**: Adjacency matrices
5. **Economics**: Input-output models
6. **Statistics**: Covariance matrices

## 9. Related Algorithms

- [Gaussian Elimination](gaussian_elimination.md) - Solving linear systems
- Matrix exponentiation - Fibonacci in O(log n)
- Eigenvalue decomposition

## 10. References

1. [Wikipedia: Matrix (mathematics)](https://en.wikipedia.org/wiki/Matrix_(mathematics))
2. Strang, G. "Linear Algebra and Its Applications"
3. Golub, G. H. & Van Loan, C. F. "Matrix Computations"
