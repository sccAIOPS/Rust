# Tanh Activation Function

## 1. Overview

**Hyperbolic tangent (tanh)** is a smooth, zero-centered activation function that maps inputs to the range (-1, 1). It was widely used before ReLU became dominant.

**File**: `src/math/tanh.rs`

## 2. Mathematical Foundation

### 2.1 Definition

$$\tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}} = \frac{2}{1 + e^{-2x}} - 1$$

Alternative using sigmoid:
$$\tanh(x) = 2\sigma(2x) - 1$$

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | (-1, 1) |
| Zero-centered | Yes |
| tanh(0) | 0 |
| Saturation | For large |x| |
| Monotonic | Yes |

### 2.3 Derivative

$$\frac{d}{dx}\tanh(x) = 1 - \tanh^2(x) = \text{sech}^2(x)$$

Maximum gradient: $\tanh'(0) = 1$

## 3. Implementation

```rust
use std::f32::consts::E;

pub fn tanh(array: &mut Vec<f32>) -> &mut Vec<f32> {
    for value in &mut *array {
        *value = (2. / (1. + E.powf(-2. * *value))) - 1.;
    }
    array
}
```

### 3.1 Alternative Using Standard Library

```rust
pub fn tanh_std(x: f64) -> f64 {
    x.tanh()  // Rust std library
}
```

### 3.2 Usage Example

```rust
let mut input = vec![1.0, 0.5, -1.0, 0.0, 0.3];
tanh(&mut input);
// Result: [0.7616, 0.4621, -0.7616, 0.0, 0.2913]
```

## 4. Complexity

- **Time**: O(n)
- **Space**: O(1) - in-place modification

## 5. Comparison with Other Activations

| Activation | Range | Zero-centered | Vanishing Gradient |
|------------|-------|---------------|-------------------|
| tanh | (-1, 1) | Yes | Yes, at extremes |
| Sigmoid | (0, 1) | No | Yes, at extremes |
| ReLU | [0, ∞) | No | No (for x > 0) |
| Leaky ReLU | (-∞, ∞) | No | No |

## 6. Vanishing Gradient Problem

For large |x|:
- $\tanh(x) \to \pm 1$
- $\tanh'(x) \to 0$

This causes slow learning in deep networks with saturated neurons.

## 7. Applications

1. **RNNs/LSTMs**: Gate activations in recurrent networks
2. **Feature normalization**: Maps to [-1, 1] range
3. **Autoencoders**: Symmetric output around zero
4. **Time series**: When outputs should be normalized

## 8. When to Use

**Prefer tanh when:**
- Zero-centered output is beneficial
- Output should be bounded symmetrically
- Working with RNN architectures

**Prefer ReLU when:**
- Training deep networks
- Computational efficiency matters
- Sparse activations are desired

## 9. Related Functions

- [Sigmoid](sigmoid.md) - Related activation: $\tanh(x) = 2\sigma(2x) - 1$
- [ReLU](relu.md) - Modern alternative
- [Softmax](softmax.md) - Multi-class output

## 10. References

1. LeCun, Y., et al. (1998). "Efficient BackProp"
2. [Wikipedia: Hyperbolic functions](https://en.wikipedia.org/wiki/Hyperbolic_functions)
