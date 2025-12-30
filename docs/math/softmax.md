# Softmax Function

## 1. Overview

The **softmax function** converts a vector of real numbers into a probability distribution. It's the standard activation function for multi-class classification output layers.

**File**: `src/math/softmax.rs`

## 2. Mathematical Foundation

### 2.1 Definition

For a vector $\mathbf{x} = (x_1, \ldots, x_n)$:

$$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_{j=1}^{n} e^{x_j}}$$

### 2.2 Properties

1. **Probability**: All outputs in (0, 1) and sum to 1
2. **Order preserving**: argmax unchanged
3. **Smooth approximation** of argmax

### 2.3 Temperature Scaling

$$\text{softmax}_T(x_i) = \frac{e^{x_i/T}}{\sum_j e^{x_j/T}}$$

- T → 0: Approaches one-hot (hard)
- T → ∞: Approaches uniform

## 3. Implementation

```rust
use std::f32::consts::E;

pub fn softmax(array: Vec<f32>) -> Vec<f32> {
    let mut softmax_array = array;

    for value in &mut softmax_array {
        *value = E.powf(*value);
    }

    let sum: f32 = softmax_array.iter().sum();

    for value in &mut softmax_array {
        *value /= sum;
    }

    softmax_array
}
```

### 3.1 Numerically Stable Version

```rust
pub fn softmax_stable(x: &[f64]) -> Vec<f64> {
    let max_val = x.iter().cloned().fold(f64::NEG_INFINITY, f64::max);
    let exp: Vec<f64> = x.iter().map(|&xi| (xi - max_val).exp()).collect();
    let sum: f64 = exp.iter().sum();
    exp.iter().map(|&e| e / sum).collect()
}
```

Subtracting max prevents overflow when $e^{x_i}$ is large.

## 4. Complexity

- **Time**: O(n)
- **Space**: O(n) or O(1) in-place

## 5. Applications

1. **Multi-class classification**: Final layer activation
2. **Attention mechanisms**: Computing attention weights
3. **Language models**: Next token probability
4. **Reinforcement learning**: Action probability (softmax policy)

## 6. Derivative (Jacobian)

$$\frac{\partial \text{softmax}_i}{\partial x_j} = \text{softmax}_i \cdot (\delta_{ij} - \text{softmax}_j)$$

Where $\delta_{ij}$ is the Kronecker delta.

## 7. Related Functions

| Function | Use Case |
|----------|----------|
| [Sigmoid](sigmoid.md) | Binary classification |
| Log-softmax | Numerical stability with NLL |
| Sparsemax | Sparse outputs |

## 8. References

1. Goodfellow, I. et al. "Deep Learning", Chapter 6
2. [Wikipedia: Softmax Function](https://en.wikipedia.org/wiki/Softmax_function)
