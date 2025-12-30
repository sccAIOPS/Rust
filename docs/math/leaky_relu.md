# Leaky ReLU

## 1. Overview

**Leaky Rectified Linear Unit (Leaky ReLU)** is a modified ReLU that allows small negative values when input is negative, preventing "dead neurons" during neural network training.

**File**: `src/math/leaky_relu.rs`

## 2. Mathematical Foundation

### 2.1 Definition

$$f(x) = \begin{cases} x & \text{if } x \geq 0 \\ \alpha x & \text{if } x < 0 \end{cases}$$

Where $\alpha$ is a small positive constant (typically 0.01).

### 2.2 Derivative

$$f'(x) = \begin{cases} 1 & \text{if } x > 0 \\ \alpha & \text{if } x < 0 \end{cases}$$

Key advantage: Non-zero gradient for negative inputs allows continued learning.

### 2.3 Comparison with ReLU

| Property | ReLU | Leaky ReLU |
|----------|------|------------|
| Negative output | 0 | αx |
| Gradient at x < 0 | 0 | α |
| Dead neurons | Yes | No |
| Parameter | None | α |

## 3. Implementation

```rust
pub fn leaky_relu(vector: &Vec<f64>, alpha: f64) -> Vec<f64> {
    let mut _vector = vector.to_owned();

    for value in &mut _vector {
        if value < &mut 0. {
            *value *= alpha;
        }
    }

    _vector
}
```

### 3.1 Usage Example

```rust
let input = vec![-10.0, 2.0, -3.0, 4.0, -5.0, 10.0, 0.05];
let alpha = 0.01;
let output = leaky_relu(&input, alpha);
// Result: [-0.1, 2.0, -0.03, 4.0, -0.05, 10.0, 0.05]
```

## 4. Complexity

- **Time**: O(n)
- **Space**: O(n) - creates new vector

## 5. Variants

### 5.1 Parametric ReLU (PReLU)

$\alpha$ is learned during training rather than fixed:

```rust
// PReLU: α is a trainable parameter per channel
fn prelu(x: f64, alpha: f64) -> f64 {
    if x >= 0.0 { x } else { alpha * x }
}
```

### 5.2 Exponential Linear Unit (ELU)

$$f(x) = \begin{cases} x & \text{if } x \geq 0 \\ \alpha(e^x - 1) & \text{if } x < 0 \end{cases}$$

## 6. Applications

1. **Deep neural networks**: Prevents dying ReLU problem
2. **GANs**: Commonly used in discriminator networks
3. **Object detection**: YOLO and other architectures

## 7. Related Functions

- [ReLU](relu.md) - Original rectified linear unit
- [Sigmoid](sigmoid.md) - Smooth activation function
- [Softmax](softmax.md) - Multi-class output activation

## 8. References

1. Maas, A. L., et al. (2013). "Rectifier Nonlinearities Improve Neural Network Acoustic Models"
2. [Wikipedia: Rectifier (neural networks)](https://en.wikipedia.org/wiki/Rectifier_(neural_networks))
