# Huber Loss

## 1. Overview

**Huber Loss** (also called Smooth L1 Loss) is a hybrid loss function that combines the best properties of MSE and MAE. It behaves like MSE for small errors (smooth, easy to optimize) and like MAE for large errors (robust to outliers).

### Why Huber Loss?

Huber loss provides:
- **Robustness**: Linear growth for large errors prevents outlier domination
- **Smoothness**: Quadratic near zero enables stable optimization
- **Configurability**: The $\delta$ parameter controls the transition point

## 2. Mathematical Foundation

### 2.1 Definition

For residual $r = y - \hat{y}$ and threshold $\delta$:

$$L_\delta(r) = \begin{cases} \frac{1}{2}r^2 & \text{if } |r| \leq \delta \\ \delta(|r| - \frac{1}{2}\delta) & \text{otherwise} \end{cases}$$

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | $[0, +\infty)$ |
| Perfect prediction | $0$ |
| Gradient at $r=0$ | $0$ (smooth) |
| Behavior $|r| \leq \delta$ | Quadratic (like MSE) |
| Behavior $|r| > \delta$ | Linear (like MAE) |

### 2.3 Gradient

$$\frac{\partial L_\delta}{\partial \hat{y}} = \begin{cases} -r = \hat{y} - y & \text{if } |r| \leq \delta \\ -\delta \cdot \text{sign}(r) & \text{otherwise} \end{cases}$$

### 2.4 Visual Comparison

```
Loss
│       ╱        ← MAE (linear everywhere)
│      ╱
│     ╱    ╱     ← Huber (linear for large |r|)
│    ╱    ╱
│   ╱ ───╱       ← Huber transition zone
│  ╱  ∪          ← MSE (quadratic everywhere)
│ ╱
└────────────→ |residual|
     0    δ
```

## 3. Algorithm

### 3.1 Pseudocode

```
FUNCTION huber_loss(y_true, y_pred, delta)
    IF lengths differ OR empty THEN
        RETURN None
    
    total_loss ← 0
    FOR i = 0 TO length(y_pred) - 1 DO
        residual ← |y_true[i] - y_pred[i]|
        
        IF residual ≤ delta THEN
            loss ← 0.5 × residual²       // Quadratic region
        ELSE
            loss ← delta × residual - 0.5 × delta²  // Linear region
        
        total_loss ← total_loss + loss
    
    RETURN total_loss / length(y_pred)
```

### 3.2 Example

**True**: `[10.0, 8.0, 12.0]`  
**Predicted**: `[9.0, 7.0, 11.0]`  
**Delta**: `1.0`

| $i$ | $y_i$ | $\hat{y}_i$ | $|r|$ | Region | Loss |
|-----|-------|-------------|-------|--------|------|
| 0 | 10.0 | 9.0 | 1.0 | $|r| \leq \delta$ | $0.5 \times 1^2 = 0.5$ |
| 1 | 8.0 | 7.0 | 1.0 | $|r| \leq \delta$ | $0.5 \times 1^2 = 0.5$ |
| 2 | 12.0 | 11.0 | 1.0 | $|r| \leq \delta$ | $0.5 \times 1^2 = 0.5$ |

$$\text{Huber} = \frac{0.5 + 0.5 + 0.5}{3} = 0.5$$

### 3.3 Example with Large Error

**True**: `[3.0, 5.0, 7.0]`  
**Predicted**: `[2.0, 4.0, 8.0]`  
**Delta**: `0.5`

| $i$ | $|r|$ | Region | Loss Formula | Loss |
|-----|-------|--------|--------------|------|
| 0 | 1.0 | $|r| > \delta$ | $0.5 \times 1.0 - 0.125$ | 0.375 |
| 1 | 1.0 | $|r| > \delta$ | $0.5 \times 1.0 - 0.125$ | 0.375 |
| 2 | 1.0 | $|r| > \delta$ | $0.5 \times 1.0 - 0.125$ | 0.375 |

$$\text{Huber} = \frac{0.375 \times 3}{3} = 0.375$$

## 4. Complexity

- **Time**: $O(n)$
- **Space**: $O(1)$

## 5. Choosing Delta ($\delta$)

| $\delta$ Value | Behavior |
|----------------|----------|
| Very small | Acts like MAE |
| Very large | Acts like MSE |
| ~1.35 | Optimal for Gaussian noise with outliers |

### Guidelines

```
δ = median(|y - ŷ|) × 1.345
```

This gives 95% efficiency compared to MSE under Gaussian assumption.

## 6. Use Cases

| Application | Why Huber |
|-------------|-----------|
| Robust regression | Outlier-resistant with smooth optimization |
| Object detection | Smooth L1 in bounding box regression |
| Reinforcement learning | TD-error clipping |
| Time series | Handles anomalies gracefully |

## 7. Implementation Notes

### Rust-Specific

```rust
pub fn huber_loss(y_true: &[f64], y_pred: &[f64], delta: f64) -> Option<f64>
```

- Returns `Option` for error handling
- Validates equal lengths and non-empty input
- Uses pattern matching for region selection

## 8. Variants

### Pseudo-Huber Loss

Smooth approximation (always differentiable):
$$L_\delta(r) = \delta^2 \left( \sqrt{1 + (r/\delta)^2} - 1 \right)$$

### Smooth L1 (PyTorch default)

Same as Huber with $\delta = 1$, commonly used in object detection.

## 9. References

- Source: [src/machine_learning/loss_function/huber_loss.rs](../../src/machine_learning/loss_function/huber_loss.rs)
- Related: [MSE](mean_squared_error.md), [MAE](mean_absolute_error.md)
- Huber, P. J. (1964). "Robust Estimation of a Location Parameter"
