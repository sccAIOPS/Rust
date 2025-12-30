# ReLU Activation Function

## 1. Overview

**ReLU (Rectified Linear Unit)** is the most popular activation function in deep learning. It's computationally efficient and helps mitigate the vanishing gradient problem.

**File**: `src/math/relu.rs`

## 2. Mathematical Foundation

### 2.1 Definition

$$\text{ReLU}(x) = \max(0, x) = \begin{cases} x & \text{if } x > 0 \\ 0 & \text{otherwise} \end{cases}$$

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | [0, ∞) |
| Domain | ℝ |
| ReLU(0) | 0 |

### 2.3 Derivative

$$\text{ReLU}'(x) = \begin{cases} 1 & \text{if } x > 0 \\ 0 & \text{if } x < 0 \\ \text{undefined} & \text{if } x = 0 \end{cases}$$

In practice, derivative at 0 is often set to 0 or 0.5.

## 3. Implementation

```rust
pub fn relu(array: &mut Vec<f32>) -> &mut Vec<f32> {
    for value in &mut *array {
        if value <= &mut 0. {
            *value = 0.;
        }
    }
    array
}
```

## 4. Complexity

- **Time**: O(n)
- **Space**: O(1)

## 5. Advantages

1. **Computationally efficient**: Just a comparison
2. **No vanishing gradient** (for positive inputs)
3. **Sparse activation**: ~50% of neurons output 0
4. **Biological plausibility**: One-sided firing

## 6. Limitations

| Issue | Description | Solution |
|-------|-------------|----------|
| Dying ReLU | Neurons stuck at 0 | Leaky ReLU, ELU |
| Unbounded | Can explode | Batch normalization |
| Not zero-centered | - | PReLU, ELU |

## 7. Variants

| Variant | Formula | Notes |
|---------|---------|-------|
| [Leaky ReLU](leaky_relu.md) | max(αx, x) | Prevents dying |
| PReLU | max(αx, x) | α learned |
| ELU | x if x>0, α(eˣ-1) | Smooth |
| GELU | x·Φ(x) | Used in transformers |

## 8. References

1. Nair, V. & Hinton, G. "Rectified Linear Units Improve Restricted Boltzmann Machines" (2010)
