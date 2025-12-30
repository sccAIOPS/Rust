# Ternary Search for Min/Max (Iterative)

## 1. Overview

Ternary Search Min/Max is an optimization algorithm for finding the minimum or maximum value of a **unimodal function** on a given interval. Unlike array-based ternary search, this variant evaluates a function at probe points to narrow down the location of an extremum.

This is where ternary search truly shines—binary search cannot solve this problem directly because it requires a sorted sequence, whereas unimodal function optimization only has a single peak or valley.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**For Maximum:**
Given a unimodal function $f: [a, b] \rightarrow \mathbb{R}$ that is strictly increasing on $[a, x^*]$ and strictly decreasing on $[x^*, b]$, find $x^*$ such that $f(x^*) = \max_{x \in [a,b]} f(x)$.

**For Minimum:**
Given a unimodal function $f: [a, b] \rightarrow \mathbb{R}$ that is strictly decreasing on $[a, x^*]$ and strictly increasing on $[x^*, b]$, find $x^*$ such that $f(x^*) = \min_{x \in [a,b]} f(x)$.

### 2.2 Mathematical Model

**Unimodal Function Property:**

For a maximum-finding problem, if we have two points $m_1 < m_2$ in $[a, b]$:
- If $f(m_1) < f(m_2)$: The maximum is in $[m_1, b]$ (can discard $[a, m_1)$)
- If $f(m_1) > f(m_2)$: The maximum is in $[a, m_2]$ (can discard $(m_2, b]$)
- If $f(m_1) = f(m_2)$: The maximum is in $[m_1, m_2]$

**Convergence:**

After $k$ iterations, the interval shrinks to:
$$|b - a| \times \left(\frac{2}{3}\right)^k$$

To achieve precision $\epsilon$:
$$k = \log_{3/2}\left(\frac{b - a}{\epsilon}\right) = O\left(\log\frac{b-a}{\epsilon}\right)$$

### 2.3 Correctness Proof

**Invariant:** The extremum $x^*$ is always within $[\text{start}, \text{end}]$.

**Proof (for maximum):**
- If $f(m_1) < f(m_2)$, the maximum cannot be in $[a, m_1)$ because:
  - If $x^* < m_1$: Since $f$ is increasing on $[a, x^*]$ and decreasing on $[x^*, b]$, and $m_1 < m_2$, we'd have $f(m_1) > f(m_2)$ (contradiction)
- Similar logic for other cases

## 3. Algorithm Description

### 3.1 Intuition

Imagine searching for the peak of a mountain by checking elevations:
1. Pick two points, 1/3 and 2/3 along the current search range
2. Compare elevations at these points
3. The lower elevation tells us which side to eliminate
4. Repeat until the range is small enough

```
Finding Maximum:

      f(m2) ← Higher
        ●
       /|\
      / | \
f(m1)→●  |  \
    /  |  |   \
   /   |  |    \
  /    |  |     \
─┼─────┼──┼──────┼─
 a    m1 m2      b
         
Since f(m2) > f(m1), peak is in [m1, b]
```

### 3.2 Pseudocode

**Finding Maximum:**
```
TERNARY-SEARCH-MAX(f, start, end, precision):
    while |end - start| ≥ precision:
        mid1 ← start + (end - start) / 3
        mid2 ← end - (end - start) / 3
        
        r1 ← f(mid1)
        r2 ← f(mid2)
        
        if r1 < r2:
            start ← mid1      // Maximum is in right 2/3
        else if r1 > r2:
            end ← mid2        // Maximum is in left 2/3
        else:
            start ← mid1      // Maximum is in middle third
            end ← mid2
    
    return f(start)
```

**Finding Minimum:**
```
TERNARY-SEARCH-MIN(f, start, end, precision):
    while |end - start| ≥ precision:
        mid1 ← start + (end - start) / 3
        mid2 ← end - (end - start) / 3
        
        r1 ← f(mid1)
        r2 ← f(mid2)
        
        if r1 < r2:
            end ← mid2        // Minimum is in left 2/3
        else if r1 > r2:
            start ← mid1      // Minimum is in right 2/3
        else:
            start ← mid1      // Minimum is in middle third
            end ← mid2
    
    return f(start)
```

### 3.3 Step-by-Step Example

**Function:** $f(x) = -x^2 - 2x + 3$ (parabola opening downward)  
**Maximum at:** $x = -1$, $f(-1) = 4$  
**Search interval:** $[-10, 10]$  
**Precision:** $0.01$

| Iteration | start | end | mid1 | mid2 | f(mid1) | f(mid2) | Action |
|-----------|-------|-----|------|------|---------|---------|--------|
| 1 | -10 | 10 | -3.33 | 3.33 | -4.44 | -14.44 | f(m1)>f(m2), end=3.33 |
| 2 | -10 | 3.33 | -5.56 | -1.11 | -17.53 | 3.99 | f(m1)<f(m2), start=-5.56 |
| 3 | -5.56 | 3.33 | -2.59 | 0.37 | 0.93 | 2.49 | f(m1)<f(m2), start=-2.59 |
| ... | ... | ... | ... | ... | ... | ... | ... |
| Final | -1.01 | -0.99 | - | - | - | - | Return f(-1) = **4.0** |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Aspect | Complexity |
|--------|------------|
| Iterations | $O(\log((b-a)/\epsilon))$ |
| Function evaluations per iteration | 2 |
| **Total function evaluations** | $O(\log((b-a)/\epsilon))$ |

If the function evaluation costs $O(g)$:
**Total:** $O(g \cdot \log((b-a)/\epsilon))$

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Variables | $O(1)$ |
| Total | $O(1)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn ternary_search_max(
    f: fn(f32) -> f32,
    mut start: f32,
    mut end: f32,
    absolute_precision: f32,
) -> f32 {
    while (start - end).abs() >= absolute_precision {
        let mid1 = start + (end - start) / 3.0;
        let mid2 = end - (end - start) / 3.0;

        let r1 = f(mid1);
        let r2 = f(mid2);

        if r1 < r2 {
            start = mid1;
        } else if r1 > r2 {
            end = mid2;
        } else {
            start = mid1;
            end = mid2;
        }
    }
    f(start)
}

pub fn ternary_search_min(
    f: fn(f32) -> f32,
    mut start: f32,
    mut end: f32,
    absolute_precision: f32,
) -> f32 {
    while (start - end).abs() >= absolute_precision {
        let mid1 = start + (end - start) / 3.0;
        let mid2 = end - (end - start) / 3.0;

        let r1 = f(mid1);
        let r2 = f(mid2);

        if r1 < r2 {
            end = mid2;
        } else if r1 > r2 {
            start = mid1;
        } else {
            start = mid1;
            end = mid2;
        }
    }
    f(start)
}
```

**Design Choices:**

1. **Function pointer:** Takes `fn(f32) -> f32` for the objective function
2. **Mutable bounds:** `start` and `end` are mutated in-place
3. **Absolute precision:** Uses epsilon comparison for floating-point
4. **Returns value:** Returns $f(x^*)$, not $x^*$ itself

**Potential Improvement:**
Return both the optimal point and value:
```rust
fn ternary_search_max(f: fn(f32) -> f32, ...) -> (f32, f32) {
    // ...
    (start, f(start))  // (x*, f(x*))
}
```

### 5.2 Numerical Considerations

1. **Floating-point precision:** Use `absolute_precision` > machine epsilon
2. **Function evaluation cost:** Consider caching if $f$ is expensive
3. **Equal values:** The `r1 == r2` case handles flat regions

### 5.3 Edge Cases

| Case | Behavior |
|------|----------|
| start > end | Works due to abs() comparison |
| Very small interval | Terminates immediately |
| Flat function | Converges to any point (all equal) |
| Linear function | Works, but binary search better |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Machine Learning:**
   - Hyperparameter tuning (learning rate, regularization)
   - Loss function optimization
   - Neural network architecture search

2. **Computer Graphics:**
   - Ray-surface intersection (distance function minimization)
   - Bezier curve extrema
   - Light transport optimization

3. **Game Development:**
   - AI utility function maximization
   - Physics simulation parameter tuning
   - Difficulty curve balancing

4. **Financial Engineering:**
   - Portfolio optimization
   - Option pricing (implied volatility)
   - Risk measure calculation

5. **Signal Processing:**
   - Peak detection in spectra
   - Optimal filter parameters
   - Correlation maximization

### 6.2 Example: Finding Optimal Learning Rate

```rust
fn training_loss(lr: f32) -> f32 {
    // Simulated: loss decreases then increases as lr increases
    // Optimal around lr = 0.01
    let base_loss = 1.0;
    let optimal_lr = 0.01;
    base_loss + 100.0 * (lr - optimal_lr).powi(2)
}

let optimal_loss = ternary_search_min(training_loss, 0.0001, 0.1, 0.00001);
// Returns approximately 1.0 (the minimum loss)
```

## 7. Comparison with Other Optimization Methods

| Method | Use Case | Convergence | Requirements |
|--------|----------|-------------|--------------|
| Ternary Search | Unimodal functions | $O(\log(1/\epsilon))$ | Unimodality |
| Golden Section | Unimodal functions | Slightly better constants | Unimodality |
| Gradient Descent | Differentiable | Depends on curvature | Derivative |
| Newton's Method | Twice differentiable | Quadratic | Second derivative |
| Binary Search | Monotonic | $O(\log n)$ | Monotonicity |

### 7.1 Ternary vs Golden Section Search

Golden section search uses the golden ratio $\phi = \frac{1+\sqrt{5}}{2}$ to reuse one function evaluation per iteration:

| Aspect | Ternary Search | Golden Section |
|--------|----------------|----------------|
| Evaluations/iteration | 2 | 1 (after first) |
| Reduction ratio | 2/3 | ~0.618 |
| Total evaluations | $2\log_{3/2}(n)$ | $\log_\phi(n)$ |

Golden section is more efficient when function evaluation is expensive.

## 8. Limitations

1. **Unimodality requirement:** Fails for multimodal functions
2. **No derivative use:** Slower than gradient methods when derivatives available
3. **Precision limited:** Cannot achieve arbitrary precision with floating-point

## 9. References

1. Kiefer, J. (1953). "Sequential minimax search for a maximum." Proceedings of the American Mathematical Society.
2. Press, W. H., et al. (2007). "Numerical Recipes" (3rd ed.). Cambridge University Press. Section 10.2.
3. Boyd, S., & Vandenberghe, L. (2004). "Convex Optimization." Cambridge University Press.

---

**Implementation:** [`src/searching/ternary_search_min_max.rs`](../../src/searching/ternary_search_min_max.rs)  
**See Also:** [Ternary Search Min/Max (Recursive)](ternary_search_min_max_recursive.md), [Ternary Search](ternary_search.md)
