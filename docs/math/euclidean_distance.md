# Euclidean Distance

## 1. Overview

**Euclidean distance** is the "ordinary" straight-line distance between two points in Euclidean space. It's the most common distance metric used in mathematics, physics, and machine learning.

**File**: `src/math/euclidean_distance.rs`

## 2. Mathematical Foundation

### 2.1 Definition

For two points $\mathbf{p} = (p_1, p_2, ..., p_n)$ and $\mathbf{q} = (q_1, q_2, ..., q_n)$ in n-dimensional space:

$$d(\mathbf{p}, \mathbf{q}) = \sqrt{\sum_{i=1}^{n} (p_i - q_i)^2}$$

### 2.2 Special Cases

**2D (Pythagorean distance)**:
$$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

**3D**:
$$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2 + (z_2 - z_1)^2}$$

### 2.3 Properties

| Property | Description |
|----------|-------------|
| Non-negativity | $d(p, q) \geq 0$ |
| Identity | $d(p, q) = 0 \iff p = q$ |
| Symmetry | $d(p, q) = d(q, p)$ |
| Triangle inequality | $d(p, r) \leq d(p, q) + d(q, r)$ |

## 3. Implementation

```rust
pub fn euclidean_distance(vector_1: &Vector, vector_2: &Vector) -> f64 {
    let squared_sum: f64 = vector_1
        .iter()
        .zip(vector_2.iter())
        .map(|(&a, &b)| (a - b).powi(2))
        .sum();

    squared_sum.sqrt()
}

type Vector = Vec<f64>;
```

### 3.1 Usage Example

```rust
// 2D vectors
let vec1 = vec![1.0, 2.0];
let vec2 = vec![4.0, 6.0];
let dist = euclidean_distance(&vec1, &vec2);
// Result: 5.0 (3² + 4² = 25, √25 = 5)

// 4D vectors
let vec1 = vec![1.0, 2.0, 3.0, 4.0];
let vec2 = vec![5.0, 6.0, 7.0, 8.0];
let dist = euclidean_distance(&vec1, &vec2);
// Result: 8.0
```

## 4. Complexity

- **Time**: O(n) where n = dimensions
- **Space**: O(1)

## 5. Comparison with Other Distance Metrics

| Metric | Formula | Use Case |
|--------|---------|----------|
| Euclidean (L²) | $\sqrt{\sum(p_i - q_i)^2}$ | Default choice, geometric distance |
| Manhattan (L¹) | $\sum|p_i - q_i|$ | Grid-based movement |
| Chebyshev (L∞) | $\max|p_i - q_i|$ | King's move in chess |
| Minkowski | $(\sum|p_i - q_i|^p)^{1/p}$ | Generalized norm |
| Cosine | $1 - \frac{\mathbf{p} \cdot \mathbf{q}}{|\mathbf{p}||\mathbf{q}|}$ | Text similarity, direction |

## 6. Visual Comparison

```
        B
        │
    L∞ ─┼───┐
        │   │
    L² ─┼───●
        │  ╱│
        │ ╱ │
        │╱  │
    A───●───┴─── L¹
```

- L¹ (Manhattan): Follow grid lines
- L² (Euclidean): Direct line
- L∞ (Chebyshev): Maximum single dimension

## 7. Optimization: Squared Distance

For comparisons, avoid the square root:

```rust
pub fn euclidean_distance_squared(v1: &[f64], v2: &[f64]) -> f64 {
    v1.iter()
        .zip(v2.iter())
        .map(|(&a, &b)| (a - b).powi(2))
        .sum()
}

// Useful for: finding nearest neighbor, comparisons
// if dist_squared(a, b) < dist_squared(a, c)
// then dist(a, b) < dist(a, c)
```

## 8. Applications

1. **Machine learning**: k-NN, k-means clustering
2. **Computer vision**: Image similarity
3. **Recommendation systems**: User/item similarity
4. **Physics**: Distance calculations
5. **GPS/Navigation**: Point-to-point distance
6. **Pattern recognition**: Feature matching

## 9. Numerical Considerations

### 9.1 Overflow Prevention

For very large values, normalize first or use:
```rust
fn safe_euclidean(v1: &[f64], v2: &[f64]) -> f64 {
    let max_diff = v1.iter()
        .zip(v2.iter())
        .map(|(&a, &b)| (a - b).abs())
        .fold(0.0_f64, f64::max);
    
    if max_diff == 0.0 { return 0.0; }
    
    let scaled_sum: f64 = v1.iter()
        .zip(v2.iter())
        .map(|(&a, &b)| ((a - b) / max_diff).powi(2))
        .sum();
    
    max_diff * scaled_sum.sqrt()
}
```

## 10. References

1. [Wikipedia: Euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance)
2. [Wikipedia: Distance](https://en.wikipedia.org/wiki/Distance)
3. Duda, R. O., et al. "Pattern Classification"
