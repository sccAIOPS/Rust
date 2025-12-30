# Interpolation Methods

## 1. Overview

**Interpolation** is the process of estimating unknown values between known data points. This module implements two fundamental interpolation methods: linear interpolation and Lagrange polynomial interpolation.

**File**: `src/math/interpolation.rs`

## 2. Linear Interpolation

### 2.1 Definition

For two points $(x_0, y_0)$ and $(x_1, y_1)$:

$$y = y_0 + (x - x_0) \cdot \frac{y_1 - y_0}{x_1 - x_0}$$

### 2.2 Geometric Interpretation

Linear interpolation draws a straight line between two points and finds the y-value for any x on that line.

```
y │
  │       ●(x₁, y₁)
  │      ╱
  │   ●─╱── y (interpolated)
  │  ╱
  │ ●(x₀, y₀)
  └────────────► x
        x
```

### 2.3 Implementation

```rust
pub fn linear_interpolation(x: f64, point0: (f64, f64), point1: (f64, f64)) -> f64 {
    point0.1 + (x - point0.0) * (point1.1 - point0.1) / (point1.0 - point0.0)
}
```

### 2.4 Usage

```rust
let point1 = (0.0, 0.0);
let point2 = (1.0, 1.0);
let y = linear_interpolation(0.5, point1, point2);
// Result: 0.5
```

## 3. Lagrange Polynomial Interpolation

### 3.1 Definition

For $n+1$ points, the Lagrange polynomial is:

$$L(x) = \sum_{j=0}^{n} y_j \cdot \ell_j(x)$$

Where the basis polynomials are:

$$\ell_j(x) = \prod_{k=0, k \neq j}^{n} \frac{x - x_k}{x_j - x_k}$$

### 3.2 Properties

| Property | Description |
|----------|-------------|
| Uniqueness | Only polynomial of degree ≤ n passing through n+1 points |
| Exactness | $L(x_i) = y_i$ for all given points |
| Degree | At most n for n+1 points |

### 3.3 Implementation

```rust
pub fn lagrange_polynomial_interpolation(x: f64, defined_points: &Vec<(f64, f64)>) -> f64 {
    let mut defined_x_values: Vec<f64> = Vec::new();
    let mut defined_y_values: Vec<f64> = Vec::new();

    for (x, y) in defined_points {
        defined_x_values.push(*x);
        defined_y_values.push(*y);
    }

    let mut sum = 0.0;

    for y_index in 0..defined_y_values.len() {
        let mut numerator = 1.0;
        let mut denominator = 1.0;
        for x_index in 0..defined_x_values.len() {
            if y_index == x_index {
                continue;
            }
            denominator *= defined_x_values[y_index] - defined_x_values[x_index];
            numerator *= x - defined_x_values[x_index];
        }

        sum += numerator / denominator * defined_y_values[y_index];
    }
    sum
}
```

### 3.4 Usage

```rust
// Points from f(x) = x²
let points = vec![(0.0, 0.0), (1.0, 1.0), (2.0, 4.0), (3.0, 9.0)];

let y = lagrange_polynomial_interpolation(1.5, &points);
// Result: 2.25 (which equals 1.5²)
```

## 4. Complexity

| Method | Time | Space |
|--------|------|-------|
| Linear | O(1) | O(1) |
| Lagrange | O(n²) | O(n) |

## 5. Comparison of Methods

| Feature | Linear | Lagrange |
|---------|--------|----------|
| Points needed | 2 | n+1 |
| Accuracy | Low (for curves) | High |
| Smoothness | C⁰ (continuous) | C^∞ (smooth) |
| Computation | Fast | Slower |
| Runge's phenomenon | No | Yes (high degree) |

## 6. Runge's Phenomenon

**Warning**: High-degree polynomial interpolation can oscillate wildly between points (Runge's phenomenon), especially with equally-spaced points.

Solutions:
- Use Chebyshev nodes instead of equally-spaced points
- Use spline interpolation
- Limit polynomial degree

## 7. Other Interpolation Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| Linear | Straight line | Simple, fast |
| Lagrange | Polynomial fit | Exact polynomial through points |
| Newton | Divided differences | Efficient updates |
| Cubic spline | Piecewise cubic | Smooth curves |
| Bezier | Control points | Graphics, CAD |

## 8. Applications

1. **Computer graphics**: Smooth animations, curves
2. **Signal processing**: Resampling, reconstruction
3. **Data analysis**: Filling missing values
4. **Numerical methods**: Function approximation
5. **Physics**: Trajectory prediction

## 9. Related Functions

- [Newton-Raphson](newton_raphson.md) - Root finding using interpolation concepts
- [Gaussian Elimination](gaussian_elimination.md) - Can be used to find interpolating polynomial

## 10. References

1. [Wikipedia: Linear interpolation](https://en.wikipedia.org/wiki/Linear_interpolation)
2. [Wikipedia: Lagrange polynomial](https://en.wikipedia.org/wiki/Lagrange_polynomial)
3. Burden, R. L. & Faires, J. D. "Numerical Analysis"
