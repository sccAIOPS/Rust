# Simpson's Integration

## 1. Overview

**Simpson's rule** approximates the definite integral using quadratic polynomials to fit the function over subintervals. It provides higher accuracy than the trapezoidal rule with comparable computational cost.

**File**: `src/math/simpsons_integration.rs`

## 2. Mathematical Foundation

### 2.1 Definition (Composite Simpson's Rule)

For function $f(x)$ over $[a, b]$ with $n$ subintervals:

$$\int_a^b f(x) \, dx \approx \sum_{i=0}^{n-1} \frac{h}{6}[f(x_i) + 4f(x_i + h/2) + f(x_{i+1})]$$

Where $h = \frac{b - a}{n}$

### 2.2 Simple Simpson's Rule

For a single interval (3 points):

$$\int_a^b f(x) \, dx \approx \frac{b-a}{6}[f(a) + 4f(\frac{a+b}{2}) + f(b)]$$

### 2.3 Geometric Interpretation

Simpson's rule fits a parabola through three points (left, middle, right) of each subinterval, then computes the exact area under that parabola.

## 3. Visual Representation

```
f(x) │     
     │     ●        parabola fit
     │   ╱   ╲●     through 3 points
     │  ●     │
     │  │     │
    ─┼──┴─────┴──► x
     a  mid    b
```

## 4. Implementation

```rust
pub fn simpsons_integration<F>(f: F, a: f64, b: f64, n: usize) -> f64
where
    F: Fn(f64) -> f64,
{
    let h = (b - a) / n as f64;
    (0..n)
        .map(|i| {
            let x0 = a + i as f64 * h;
            let x1 = x0 + h / 2.0;
            let x2 = x0 + h;
            (h / 6.0) * (f(x0) + 4.0 * f(x1) + f(x2))
        })
        .sum()
}
```

### 4.1 Usage Example

```rust
// Integrate x² from 0 to 1 (exact: 1/3)
let result = simpsons_integration(|x| x.powi(2), 0.0, 1.0, 100);
// Result: 0.333333... (error < 1e-6)

// Integrate sin(x) from 0 to π (exact: 2)
let result = simpsons_integration(|x| x.sin(), 0.0, std::f64::consts::PI, 100);
// Result: ≈ 2.0
```

## 5. Complexity

- **Time**: O(n)
- **Space**: O(1)
- **Function evaluations**: 2n + 1

## 6. Error Analysis

### 6.1 Error Bound

$$|E| \leq \frac{(b-a)^5}{2880n^4} \max_{a \leq x \leq b} |f^{(4)}(x)|$$

### 6.2 Key Properties

| Property | Value |
|----------|-------|
| Convergence order | O(h⁴) |
| Exact for | Polynomials up to degree 3 |
| Doubling n | Reduces error by factor of 16 |

### 6.3 Comparison with Trapezoidal

| Metric | Trapezoidal | Simpson's |
|--------|-------------|-----------|
| Error order | O(h²) | O(h⁴) |
| Function evals | n+1 | 2n+1 |
| Exact for | Degree 1 | Degree 3 |
| Practical accuracy | Good | Excellent |

## 7. When to Use

### 7.1 Simpson's Excels

- Smooth functions with continuous derivatives
- Polynomial-like behavior
- When high accuracy is needed with few evaluations

### 7.2 Consider Alternatives

- Highly oscillatory functions → Adaptive methods
- Functions with singularities → Special quadrature
- Very high precision needed → Gaussian quadrature

## 8. Variants

### 8.1 Simpson's 3/8 Rule

Uses 4 points per interval (cubic fit):
$$\int_a^b f(x) \, dx \approx \frac{3h}{8}[f(x_0) + 3f(x_1) + 3f(x_2) + f(x_3)]$$

### 8.2 Adaptive Simpson's

```rust
fn adaptive_simpsons<F>(f: &F, a: f64, b: f64, tol: f64) -> f64 
where F: Fn(f64) -> f64 
{
    let whole = simpsons_single(f, a, b);
    let mid = (a + b) / 2.0;
    let left = simpsons_single(f, a, mid);
    let right = simpsons_single(f, mid, b);
    
    if (left + right - whole).abs() < 15.0 * tol {
        left + right + (left + right - whole) / 15.0
    } else {
        adaptive_simpsons(f, a, mid, tol/2.0) + 
        adaptive_simpsons(f, mid, b, tol/2.0)
    }
}
```

## 9. Applications

1. **Scientific computing**: Numerical integration
2. **Physics simulations**: Computing physical quantities
3. **Statistics**: Distribution function calculations
4. **Engineering**: Area/volume computations
5. **Finance**: Option pricing integrals

## 10. Related Methods

- [Trapezoidal Integration](trapezoidal_integration.md) - Simpler, less accurate
- Gaussian quadrature - Higher accuracy, fixed points
- Monte Carlo integration - High-dimensional problems
- Romberg integration - Richardson extrapolation

## 11. References

1. [Wikipedia: Simpson's rule](https://en.wikipedia.org/wiki/Simpson%27s_rule)
2. Press, W. H., et al. "Numerical Recipes"
3. Burden, R. L. & Faires, J. D. "Numerical Analysis"
