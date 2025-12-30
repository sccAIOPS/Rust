# Ternary Search for Min/Max (Recursive)

## 1. Overview

The recursive implementation of Ternary Search Min/Max provides an elegant, functionally-styled approach to finding extrema of unimodal functions. This version directly mirrors the mathematical definition of the algorithm, making the logic easier to follow and verify.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a unimodal function $f: [a, b] \rightarrow \mathbb{R}$ and precision $\epsilon$, find the value $f(x^*)$ where $x^*$ is the extremum (maximum or minimum).

### 2.2 Recursive Definition

**For Maximum:**

$$
\text{search\_max}(f, a, b, \epsilon) = 
\begin{cases}
f(a) & \text{if } |b - a| < \epsilon \\
\text{search\_max}(f, m_1, b, \epsilon) & \text{if } f(m_1) < f(m_2) \\
\text{search\_max}(f, a, m_2, \epsilon) & \text{if } f(m_1) > f(m_2) \\
\text{search\_max}(f, m_1, m_2, \epsilon) & \text{otherwise}
\end{cases}
$$

where $m_1 = a + (b-a)/3$ and $m_2 = b - (b-a)/3$

**For Minimum:**

$$
\text{search\_min}(f, a, b, \epsilon) = 
\begin{cases}
f(a) & \text{if } |b - a| < \epsilon \\
\text{search\_min}(f, a, m_2, \epsilon) & \text{if } f(m_1) < f(m_2) \\
\text{search\_min}(f, m_1, b, \epsilon) & \text{if } f(m_1) > f(m_2) \\
\text{search\_min}(f, m_1, m_2, \epsilon) & \text{otherwise}
\end{cases}
$$

### 2.3 Recurrence Analysis

**Time Recurrence:**
$$T(n) = T(2n/3) + O(1)$$

where $n$ represents the interval size relative to precision.

**Recursion Depth:**
$$d = \log_{3/2}\left(\frac{b - a}{\epsilon}\right)$$

## 3. Algorithm Description

### 3.1 Intuition

The recursive version thinks of the problem as:
- **Base case:** Interval is small enough—return the function value
- **Recursive case:** Determine which portion contains the extremum, recurse on that portion

This mirrors how we might explain the algorithm mathematically.

### 3.2 Pseudocode

**Finding Maximum:**
```
TERNARY-SEARCH-MAX-REC(f, start, end, precision):
    if |end - start| < precision:
        return f(start)
    
    mid1 ← start + (end - start) / 3
    mid2 ← end - (end - start) / 3
    
    r1 ← f(mid1)
    r2 ← f(mid2)
    
    if r1 < r2:
        return TERNARY-SEARCH-MAX-REC(f, mid1, end, precision)
    else if r1 > r2:
        return TERNARY-SEARCH-MAX-REC(f, start, mid2, precision)
    else:
        return TERNARY-SEARCH-MAX-REC(f, mid1, mid2, precision)
```

### 3.3 Call Tree Example

**Function:** $f(x) = -x^2 + 3x + 5$ (maximum at $x = 1.5$, $f(1.5) = 7.25$)  
**Interval:** $[-10, 10]$  
**Precision:** $0.01$

```
ternary_search_max_rec(f, -10, 10, 0.01)
│ mid1 = -3.33, mid2 = 3.33
│ f(-3.33) = -6.11, f(3.33) = 4.44
│ f(m1) < f(m2) → recurse on [mid1, end]
│
└─► ternary_search_max_rec(f, -3.33, 10, 0.01)
    │ mid1 = 1.11, mid2 = 5.56
    │ f(1.11) = 7.10, f(5.56) = -6.58
    │ f(m1) > f(m2) → recurse on [start, mid2]
    │
    └─► ternary_search_max_rec(f, -3.33, 5.56, 0.01)
        │ mid1 = -0.37, mid2 = 2.59
        │ f(-0.37) = 3.76, f(2.59) = 6.94
        │ f(m1) < f(m2) → recurse on [mid1, end]
        │
        └─► ... (continues until precision met)
            │
            └─► return f(1.5) ≈ 7.25 ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Aspect | Complexity |
|--------|------------|
| Recursive calls | $O(\log((b-a)/\epsilon))$ |
| Function evaluations per call | 2 |
| **Total** | $O(\log((b-a)/\epsilon))$ |

### 4.2 Space Complexity

| Aspect | Complexity | Notes |
|--------|------------|-------|
| Stack frames | $O(\log((b-a)/\epsilon))$ | One frame per recursion |
| Per frame | $O(1)$ | Only stores mid1, mid2, r1, r2 |
| **Total** | $O(\log((b-a)/\epsilon))$ | Dominated by recursion depth |

**Stack Frame Contents:**
- `f`: Function pointer (shared)
- `start`, `end`: f32 (8 bytes)
- `absolute_precision`: f32 (4 bytes)
- `mid1`, `mid2`, `r1`, `r2`: f32 (16 bytes)
- Return address: ~8 bytes

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn ternary_search_max_rec(
    f: fn(f32) -> f32,
    start: f32,
    end: f32,
    absolute_precision: f32,
) -> f32 {
    if (end - start).abs() >= absolute_precision {
        let mid1 = start + (end - start) / 3.0;
        let mid2 = end - (end - start) / 3.0;

        let r1 = f(mid1);
        let r2 = f(mid2);

        if r1 < r2 {
            return ternary_search_max_rec(f, mid1, end, absolute_precision);
        } else if r1 > r2 {
            return ternary_search_max_rec(f, start, mid2, absolute_precision);
        }
        return ternary_search_max_rec(f, mid1, mid2, absolute_precision);
    }
    f(start)
}

pub fn ternary_search_min_rec(
    f: fn(f32) -> f32,
    start: f32,
    end: f32,
    absolute_precision: f32,
) -> f32 {
    if (end - start).abs() >= absolute_precision {
        let mid1 = start + (end - start) / 3.0;
        let mid2 = end - (end - start) / 3.0;

        let r1 = f(mid1);
        let r2 = f(mid2);

        if r1 < r2 {
            return ternary_search_min_rec(f, start, mid2, absolute_precision);
        } else if r1 > r2 {
            return ternary_search_min_rec(f, mid1, end, absolute_precision);
        }
        return ternary_search_min_rec(f, mid1, mid2, absolute_precision);
    }
    f(start)
}
```

**Key Observations:**

1. **Tail recursion potential:** Each branch ends with a recursive call, making it a candidate for tail-call optimization (though Rust doesn't guarantee TCO)

2. **Immutable parameters:** Unlike iterative version, doesn't mutate `start`/`end`

3. **Return value:** Returns $f(x^*)$, not $x^*$

### 5.2 Comparison with Iterative

| Aspect | Recursive | Iterative |
|--------|-----------|-----------|
| Code clarity | More elegant | More verbose |
| Space | $O(\log n)$ stack | $O(1)$ |
| Performance | Slight overhead | Marginally faster |
| Debugging | Clearer call stack | Requires breakpoints |

### 5.3 Edge Cases

| Case | Behavior |
|------|----------|
| Interval < precision | Returns f(start) immediately |
| start > end | Still works (abs in condition) |
| Precision = 0 | Infinite recursion! |
| f(m1) = f(m2) | Correctly narrows to middle |

**Critical Bug Risk:** Setting `absolute_precision = 0.0` causes infinite recursion.

## 6. Real-World Applications

### 6.1 When to Use Recursive Version

1. **Functional programming context:** Matches functional paradigm
2. **Clear mathematical correspondence:** When proving correctness matters
3. **Educational settings:** Easier to teach and understand
4. **Prototyping:** Faster to write correctly

### 6.2 Example Applications

**1. Neural Network Learning Rate Finder:**
```rust
fn validation_loss(lr: f32) -> f32 {
    // Train model with given learning rate
    // Return validation loss (unimodal in practice)
    simulate_training(lr)
}

let best_loss = ternary_search_min_rec(
    validation_loss,
    1e-6,   // min lr
    1.0,    // max lr
    1e-5    // precision
);
```

**2. Physical System Equilibrium:**
```rust
fn potential_energy(position: f32) -> f32 {
    // Mechanical system potential energy
    0.5 * SPRING_CONSTANT * position.powi(2) + GRAVITY * position
}

let equilibrium_energy = ternary_search_min_rec(
    potential_energy,
    -10.0,
    10.0,
    0.001
);
```

## 7. Recursion Depth Considerations

**For typical use cases:**

| Interval | Precision | Max Depth |
|----------|-----------|-----------|
| $[-10, 10]$ | $0.001$ | ~23 |
| $[-1000, 1000]$ | $0.0001$ | ~38 |
| $[-10^9, 10^9]$ | $0.000001$ | ~75 |

Rust's default stack size (1-8 MB) easily handles these depths.

**Stack Overflow Risk:**
```rust
// DANGEROUS: effectively precision = 0
ternary_search_max_rec(f, 0.0, 1.0, f32::EPSILON)  // ~150+ levels
```

## 8. Variants

### 8.1 Returning (x*, f(x*))

```rust
fn ternary_search_max_rec_full(
    f: fn(f32) -> f32,
    start: f32,
    end: f32,
    precision: f32,
) -> (f32, f32) {
    if (end - start).abs() < precision {
        return (start, f(start));
    }
    
    let mid1 = start + (end - start) / 3.0;
    let mid2 = end - (end - start) / 3.0;
    let r1 = f(mid1);
    let r2 = f(mid2);
    
    if r1 < r2 {
        ternary_search_max_rec_full(f, mid1, end, precision)
    } else if r1 > r2 {
        ternary_search_max_rec_full(f, start, mid2, precision)
    } else {
        ternary_search_max_rec_full(f, mid1, mid2, precision)
    }
}
```

### 8.2 Generic Version

```rust
fn ternary_search_extremum_rec<F>(
    f: F,
    start: f64,
    end: f64,
    precision: f64,
    find_max: bool,
) -> f64
where
    F: Fn(f64) -> f64,
{
    if (end - start).abs() < precision {
        return f(start);
    }
    
    let mid1 = start + (end - start) / 3.0;
    let mid2 = end - (end - start) / 3.0;
    let r1 = f(mid1);
    let r2 = f(mid2);
    
    let go_right = if find_max { r1 < r2 } else { r1 > r2 };
    
    if go_right {
        ternary_search_extremum_rec(f, mid1, end, precision, find_max)
    } else if (find_max && r1 > r2) || (!find_max && r1 < r2) {
        ternary_search_extremum_rec(f, start, mid2, precision, find_max)
    } else {
        ternary_search_extremum_rec(f, mid1, mid2, precision, find_max)
    }
}
```

## 9. References

1. Brent, R. P. (1973). "Algorithms for Minimization without Derivatives." Prentice-Hall. Chapter 5.
2. Nocedal, J., & Wright, S. J. (2006). "Numerical Optimization" (2nd ed.). Springer. Chapter 3.
3. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Chapter 4.

---

**Implementation:** [`src/searching/ternary_search_min_max_recursive.rs`](../../src/searching/ternary_search_min_max_recursive.rs)  
**See Also:** [Ternary Search Min/Max (Iterative)](ternary_search_min_max.md), [Ternary Search (Recursive)](ternary_search_recursive.md)
