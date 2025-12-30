# Trapezoidal Integration

## 1. Overview

**Trapezoidal integration** (trapezoidal rule) approximates the definite integral of a function by dividing the area under the curve into trapezoids and summing their areas.

**File**: `src/math/trapezoidal_integration.rs`

## 2. Mathematical Foundation

### 2.1 Definition

For a function $f(x)$ over interval $[a, b]$ divided into $n$ subintervals:

$$\int_a^b f(x) \, dx \approx \frac{h}{2} \sum_{i=0}^{n-1} [f(x_i) + f(x_{i+1})]$$

Where $h = \frac{b - a}{n}$ and $x_i = a + ih$

### 2.2 Simplified Formula

$$\int_a^b f(x) \, dx \approx \frac{h}{2}[f(a) + 2f(x_1) + 2f(x_2) + ... + 2f(x_{n-1}) + f(b)]$$

### 2.3 Single Trapezoid

For one trapezoid from $a$ to $b$:
$$\text{Area} = \frac{(f(a) + f(b)) \cdot (b - a)}{2}$$

## 3. Visual Representation

```
f(x) │     
     │   ●────●
     │  ╱│    │╲
     │ ╱ │    │ ╲●
     │╱  │    │  │╲
    ─┼───┴────┴──┴──► x
     a   x₁   x₂   b
     
Each trapezoid approximates the area under the curve
```

## 4. Implementation

```rust
pub fn trapezoidal_integral<F>(a: f64, b: f64, f: F, precision: u32) -> f64
where
    F: Fn(f64) -> f64,
{
    let delta = (b - a) / precision as f64;

    (0..precision)
        .map(|trapezoid| {
            let left_side = a + (delta * trapezoid as f64);
            let right_side = left_side + delta;

            0.5 * (f(left_side) + f(right_side)) * delta
        })
        .sum()
}
```

### 4.1 Usage Example

```rust
// Integrate x² from 0 to 1 (exact answer: 1/3)
let result = trapezoidal_integral(0.0, 1.0, |x| x.powi(2), 1000);
// Result: ≈ 0.33333... (within 0.0001 of exact)

// Higher precision
let result = trapezoidal_integral(0.0, 1.0, |x| x.powi(2), 10000);
// Result: ≈ 0.333333... (within 0.00001 of exact)
```

## 5. Complexity

- **Time**: O(n) where n = number of subdivisions
- **Space**: O(1)

## 6. Error Analysis

### 6.1 Error Bound

$$|E| \leq \frac{(b-a)^3}{12n^2} \max_{a \leq x \leq b} |f''(x)|$$

### 6.2 Convergence Rate

- Error is O(h²) = O(1/n²)
- Doubling n reduces error by factor of 4

### 6.3 When It Works Well

| Function Type | Accuracy |
|--------------|----------|
| Linear | Exact |
| Smooth, low curvature | Excellent |
| Polynomial | Good |
| Highly oscillatory | Poor (needs many subdivisions) |
| Discontinuous | Poor |

## 7. Comparison with Other Methods

| Method | Error Order | Function Evaluations |
|--------|-------------|---------------------|
| Left/Right Riemann | O(h) | n |
| Trapezoidal | O(h²) | n+1 |
| [Simpson's](simpsons_integration.md) | O(h⁴) | 2n+1 |
| Gaussian quadrature | Very high | Varies |

## 8. Adaptive Version

```rust
// Adaptive trapezoidal rule (concept)
fn adaptive_trapezoidal<F>(a: f64, b: f64, f: &F, tol: f64) -> f64 
where F: Fn(f64) -> f64 
{
    let coarse = trapezoidal_integral(a, b, f, 1);
    let fine = trapezoidal_integral(a, b, f, 2);
    
    if (coarse - fine).abs() < tol {
        fine
    } else {
        let mid = (a + b) / 2.0;
        adaptive_trapezoidal(a, mid, f, tol/2.0) + 
        adaptive_trapezoidal(mid, b, f, tol/2.0)
    }
}
```

## 9. Applications

1. **Scientific computing**: Numerical integration
2. **Physics**: Calculating work, area, volume
3. **Signal processing**: Area under signals
4. **Statistics**: Probability calculations
5. **Economics**: Consumer/producer surplus

## 10. References

1. [Wikipedia: Trapezoidal rule](https://en.wikipedia.org/wiki/Trapezoidal_rule)
2. Press, W. H., et al. "Numerical Recipes"
3. Burden, R. L. & Faires, J. D. "Numerical Analysis"
