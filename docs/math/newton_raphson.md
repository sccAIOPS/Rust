# Newton-Raphson Method

## 1. Overview

The **Newton-Raphson method** (also called Newton's method) is an iterative algorithm for finding roots of real-valued functions. It uses the function's derivative to rapidly converge to a solution, achieving quadratic convergence near the root.

**File**: `src/math/newton_raphson.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a differentiable function $f: \mathbb{R} \to \mathbb{R}$, find $x^*$ such that $f(x^*) = 0$.

### 2.2 Derivation

Using Taylor series expansion around current estimate $x_n$:
$$f(x) \approx f(x_n) + f'(x_n)(x - x_n)$$

Setting $f(x) = 0$ and solving:
$$x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}$$

### 2.3 Geometric Interpretation

At each step:
1. Draw tangent line to curve at $(x_n, f(x_n))$
2. Find where tangent intersects x-axis
3. Use intersection as next estimate

### 2.4 Convergence

For a simple root (where $f'(x^*) \neq 0$):
- **Quadratic convergence**: $|x_{n+1} - x^*| \leq C|x_n - x^*|^2$
- Number of correct digits approximately doubles each iteration

## 3. Algorithm Description

### 3.1 Pseudocode

```
function newton_raphson(f, f', x0, iterations):
    x ← x0
    for i from 1 to iterations:
        x ← x - f(x) / f'(x)
    return x
```

### 3.2 Step-by-Step Example

Find root of $f(x) = x^2 - 2$ (i.e., compute $\sqrt{2}$):
- $f'(x) = 2x$
- Update: $x_{n+1} = x_n - \frac{x_n^2 - 2}{2x_n} = \frac{x_n + 2/x_n}{2}$

| n | xₙ | f(xₙ) | x_{n+1} |
|---|-----|-------|---------|
| 0 | 1.0 | -1.0 | 1.5 |
| 1 | 1.5 | 0.25 | 1.41667 |
| 2 | 1.41667 | 0.00694 | 1.41422 |
| 3 | 1.41422 | 0.00001 | **1.41421356...** |

After only 3 iterations, we have 8 digits of accuracy!

## 4. Complexity Analysis

### 4.1 Convergence Rate

| Type | Rate | Error Bound |
|------|------|-------------|
| Quadratic | $e_{n+1} \leq Ce_n^2$ | Digits double per iteration |

### 4.2 Number of Iterations

To achieve error $\epsilon$ starting from error $e_0$:
$$n \approx \log_2 \log_2(e_0/\epsilon)$$

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn find_root(
    f: fn(f64) -> f64, 
    fd: fn(f64) -> f64, 
    guess: f64, 
    iterations: i32
) -> f64 {
    let mut result = guess;
    for _ in 0..iterations {
        result = iteration(f, fd, result);
    }
    result
}

pub fn iteration(
    f: fn(f64) -> f64, 
    fd: fn(f64) -> f64, 
    guess: f64
) -> f64 {
    guess - f(guess) / fd(guess)
}
```

### 5.2 Stopping Criteria

```rust
fn newton_with_tolerance(f: fn(f64) -> f64, fd: fn(f64) -> f64, 
                         x0: f64, tol: f64, max_iter: i32) -> Option<f64> {
    let mut x = x0;
    for _ in 0..max_iter {
        let fx = f(x);
        if fx.abs() < tol {
            return Some(x);
        }
        let dx = fx / fd(x);
        if dx.abs() < tol {
            return Some(x);
        }
        x -= dx;
    }
    None  // Failed to converge
}
```

### 5.3 Edge Cases

| Situation | Problem | Solution |
|-----------|---------|----------|
| f'(x) = 0 | Division by zero | Perturb or use modified Newton |
| Poor initial guess | May diverge | Use bracketing method first |
| Multiple roots | Slow convergence | Use modified Newton |

## 6. Applications

### 6.1 Computing Square Roots

$f(x) = x^2 - a$, so:
$$x_{n+1} = \frac{1}{2}\left(x_n + \frac{a}{x_n}\right)$$

(This is the Babylonian method)

### 6.2 Computing Reciprocals (No Division)

$f(x) = 1/x - a$, so:
$$x_{n+1} = x_n(2 - ax_n)$$

### 6.3 Optimization

Finding minima: solve $f'(x) = 0$
$$x_{n+1} = x_n - \frac{f'(x_n)}{f''(x_n)}$$

## 7. Related Algorithms

| Algorithm | Convergence | Requires |
|-----------|-------------|----------|
| Bisection | Linear | Bracketing |
| Newton | Quadratic | Derivative |
| Secant | Superlinear (~1.618) | Two points |
| Halley | Cubic | First & second derivative |

## 8. References

1. Press, W. H. et al. "Numerical Recipes", Chapter 9
2. [Wikipedia: Newton's Method](https://en.wikipedia.org/wiki/Newton%27s_method)
