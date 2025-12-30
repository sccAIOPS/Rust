# Sigmoid Activation Function

## 1. Overview

The **sigmoid function** (logistic function) is a smooth, S-shaped curve that maps any real number to a value between 0 and 1. It's one of the earliest activation functions used in neural networks.

**File**: `src/math/sigmoid.rs`

## 2. Mathematical Foundation

### 2.1 Definition

$$\sigma(x) = \frac{1}{1 + e^{-x}}$$

### 2.2 Key Properties

| Property | Value |
|----------|-------|
| Range | (0, 1) |
| Domain | ℝ |
| σ(0) | 0.5 |
| Symmetry | σ(-x) = 1 - σ(x) |

### 2.3 Derivative

$$\sigma'(x) = \sigma(x)(1 - \sigma(x))$$

This makes backpropagation efficient since $\sigma'$ depends only on $\sigma$.

## 3. Implementation

```rust
use std::f32::consts::E;

pub fn sigmoid(array: &mut Vec<f32>) -> &mut Vec<f32> {
    for value in &mut *array {
        *value = 1. / (1. + E.powf(-1. * *value));
    }
    array
}
```

## 4. Complexity

- **Time**: O(n) for n elements
- **Space**: O(1) (in-place modification)

## 5. Applications

1. **Binary classification**: Output layer for probability
2. **Logistic regression**: Core function
3. **Gates in LSTMs**: Control information flow

## 6. Limitations

| Issue | Description |
|-------|-------------|
| Vanishing gradients | σ'(x) → 0 for large |x| |
| Not zero-centered | Outputs always positive |
| Computationally expensive | Requires exp() |

## 7. Related Functions

- [ReLU](relu.md): Simpler, avoids vanishing gradients
- [Softmax](softmax.md): Multi-class generalization
- [Tanh](tanh.md): Zero-centered alternative

## 8. References

1. [Wikipedia: Sigmoid Function](https://en.wikipedia.org/wiki/Sigmoid_function)
