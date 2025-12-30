# Huber Loss

## 1. Overview

**Huber loss** is a robust loss function that combines the best properties of Mean Squared Error (MSE) and Mean Absolute Error (MAE). It's less sensitive to outliers than MSE while maintaining differentiability everywhere.

**File**: `src/math/huber_loss.rs`

## 2. Mathematical Foundation

### 2.1 Definition

$$L_\delta(y, \hat{y}) = \begin{cases} \frac{1}{2}(y - \hat{y})^2 & \text{if } |y - \hat{y}| \leq \delta \\ \delta |y - \hat{y}| - \frac{1}{2}\delta^2 & \text{otherwise} \end{cases}$$

Where:
- $y$ = actual value
- $\hat{y}$ = predicted value
- $\delta$ = threshold parameter

### 2.2 Derivative

$$\frac{\partial L_\delta}{\partial \hat{y}} = \begin{cases} -(y - \hat{y}) & \text{if } |y - \hat{y}| \leq \delta \\ -\delta \cdot \text{sign}(y - \hat{y}) & \text{otherwise} \end{cases}$$

### 2.3 Comparison with Other Losses

| Property | MSE | MAE | Huber |
|----------|-----|-----|-------|
| Sensitivity to outliers | High | Low | Medium (controlled by δ) |
| Differentiable at 0 | Yes | No | Yes |
| Gradient magnitude | Unbounded | Constant | Bounded |

## 3. Implementation

```rust
pub fn huber_loss(actual: &[f64], predicted: &[f64], delta: f64) -> f64 {
    let mut loss: Vec<f64> = Vec::new();
    for (a, p) in actual.iter().zip(predicted.iter()) {
        if (a - p).abs() <= delta {
            loss.push(0.5 * (a - p).powf(2.));
        } else {
            loss.push(delta * (a - p).abs() - (0.5 * delta));
        }
    }
    loss.iter().sum()
}
```

### 3.1 Usage Example

```rust
let actual = vec![1.0, 2.0, 3.0, 4.0, 5.0];
let predicted = vec![5.0, 7.0, 9.0, 11.0, 13.0];
let delta = 1.0;
let loss = huber_loss(&actual, &predicted, delta);
// Result: 27.5
```

## 4. Complexity

- **Time**: O(n) where n = number of samples
- **Space**: O(n) for loss vector

## 5. Parameter Selection

### 5.1 Effect of δ

| δ Value | Behavior |
|---------|----------|
| Large δ | More like MSE (quadratic everywhere) |
| Small δ | More like MAE (linear for most errors) |
| δ = 1.0 | Common default |

### 5.2 Guidelines

- Start with δ = 1.0
- Increase δ if data has few outliers
- Decrease δ if data has many outliers

## 6. Visual Representation

```
Loss
  │
  │         Huber
  │        ╱     ╲
  │      ╱         ╲
  │    ╱  MSE        ╲
  │  ╱  (quadratic)    ╲
  │╱                     ╲
  ├─────────┼─────────────┼─── Error
       -δ       0       +δ
           (linear)
```

- Quadratic for |error| ≤ δ
- Linear for |error| > δ

## 7. Applications

1. **Robust regression**: When data contains outliers
2. **Reinforcement learning**: Q-learning with function approximation
3. **Object detection**: Bounding box regression (Smooth L1)
4. **Financial modeling**: Predictions with occasional extreme values

## 8. Smooth L1 Loss

**Smooth L1** is Huber loss with δ = 1, commonly used in object detection:

$$L(x) = \begin{cases} 0.5x^2 & \text{if } |x| < 1 \\ |x| - 0.5 & \text{otherwise} \end{cases}$$

## 9. Related Functions

- [Cross-entropy Loss](cross_entropy_loss.md) - Classification loss
- MSE - Mean Squared Error
- MAE - Mean Absolute Error

## 10. References

1. Huber, P. J. (1964). "Robust Estimation of a Location Parameter"
2. Girshick, R. (2015). "Fast R-CNN" (uses Smooth L1)
3. [Wikipedia: Huber loss](https://en.wikipedia.org/wiki/Huber_loss)
