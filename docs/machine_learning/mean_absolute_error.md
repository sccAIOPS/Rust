# Mean Absolute Error Loss (MAE)

## 1. Overview

**Mean Absolute Error (MAE)** is a robust loss function for regression that measures the average of absolute differences between predicted and actual values. Unlike MSE, it treats all errors equally regardless of magnitude.

### Why MAE?

MAE is preferred when:
- Outliers should not dominate the loss
- Interpretability matters (same units as target)
- Median prediction is desired (MAE minimizer is the median)

## 2. Mathematical Foundation

### 2.1 Definition

For predictions $\hat{y} = [\hat{y}_1, \ldots, \hat{y}_n]$ and actual values $y = [y_1, \ldots, y_n]$:

$$\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |\hat{y}_i - y_i|$$

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | $[0, +\infty)$ |
| Perfect prediction | $0$ |
| Units | Same as target |
| Gradient | $\frac{\partial}{\partial \hat{y}_i} = \frac{\text{sign}(\hat{y}_i - y_i)}{n}$ |

### 2.3 Comparison with MSE

| Aspect | MAE | MSE |
|--------|-----|-----|
| Outlier sensitivity | Low | High |
| Gradient at origin | Undefined | Zero |
| Optimization | Harder (non-smooth) | Easier (smooth) |
| Optimal predictor | Median | Mean |

## 3. Algorithm

### 3.1 Pseudocode

```
FUNCTION mae_loss(predicted, actual)
    total_loss ← 0
    FOR i = 0 TO length(predicted) - 1 DO
        diff ← predicted[i] - actual[i]
        total_loss ← total_loss + |diff|
    RETURN total_loss / length(predicted)
```

### 3.2 Example

**Predicted**: `[1.0, 2.0, 3.0, 4.0]`  
**Actual**: `[1.0, 3.0, 3.5, 4.5]`

| $i$ | $\hat{y}_i$ | $y_i$ | $|\hat{y}_i - y_i|$ |
|-----|-------------|-------|---------------------|
| 0 | 1.0 | 1.0 | 0.0 |
| 1 | 2.0 | 3.0 | 1.0 |
| 2 | 3.0 | 3.5 | 0.5 |
| 3 | 4.0 | 4.5 | 0.5 |

$$\text{MAE} = \frac{0 + 1 + 0.5 + 0.5}{4} = 0.5$$

## 4. Complexity

- **Time**: $O(n)$ - single pass
- **Space**: $O(1)$ - constant

## 5. Use Cases

| Application | Why MAE |
|-------------|---------|
| Financial forecasting | Robust to extreme market events |
| Demand prediction | Outlier-resistant |
| Energy consumption | Interpretable error metric |
| Any regression with outliers | Prevents outlier domination |

## 6. Limitations

⚠️ **Non-differentiable at zero**: Gradient undefined when $\hat{y} = y$

⚠️ **Slower convergence**: Constant gradient magnitude regardless of error size

⚠️ **Multiple optima**: Can have many equivalent solutions

## 7. Visual Comparison

```
Loss
│      
│    ╱╲        MSE (quadratic)
│   ╱  ╲
│  ╱ ╲╱ ╲      MAE (V-shape)
│ ╱      ╲
└──────────→ Error
```

## 8. References

- Source: [src/machine_learning/loss_function/mean_absolute_error_loss.rs](../../src/machine_learning/loss_function/mean_absolute_error_loss.rs)
- Related: [Mean Squared Error](mean_squared_error.md), [Huber Loss](huber_loss.md)
