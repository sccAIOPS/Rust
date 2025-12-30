# Mean Squared Error Loss (MSE)

## 1. Overview

**Mean Squared Error (MSE)** is the most commonly used loss function for regression problems. It measures the average of the squared differences between predicted and actual values, penalizing larger errors more heavily than smaller ones.

### Why MSE?

MSE is popular because:
- Differentiable everywhere (smooth gradient for optimization)
- Always non-negative (zero only for perfect predictions)
- Heavily penalizes outliers (due to squaring)
- Corresponds to maximum likelihood estimation under Gaussian noise

## 2. Mathematical Foundation

### 2.1 Definition

For predictions $\hat{y} = [\hat{y}_1, \ldots, \hat{y}_n]$ and actual values $y = [y_1, \ldots, y_n]$:

$$\text{MSE} = \frac{1}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i)^2$$

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | $[0, +\infty)$ |
| Perfect prediction | $0$ |
| Units | Squared units of target |
| Gradient | $\frac{\partial}{\partial \hat{y}_i} = \frac{2(\hat{y}_i - y_i)}{n}$ |

### 2.3 Relationship to Other Metrics

$$\text{RMSE} = \sqrt{\text{MSE}}$$

$$\text{MSE} = \text{Variance} + \text{Bias}^2$$

## 3. Algorithm

### 3.1 Pseudocode

```
FUNCTION mse_loss(predicted, actual)
    total_loss ← 0
    FOR i = 0 TO length(predicted) - 1 DO
        diff ← predicted[i] - actual[i]
        total_loss ← total_loss + diff × diff
    RETURN total_loss / length(predicted)
```

### 3.2 Example

**Predicted**: `[1.0, 2.0, 3.0, 4.0]`  
**Actual**: `[1.0, 3.0, 3.5, 4.5]`

| $i$ | $\hat{y}_i$ | $y_i$ | $(\hat{y}_i - y_i)^2$ |
|-----|-------------|-------|----------------------|
| 0 | 1.0 | 1.0 | 0.0 |
| 1 | 2.0 | 3.0 | 1.0 |
| 2 | 3.0 | 3.5 | 0.25 |
| 3 | 4.0 | 4.5 | 0.25 |

$$\text{MSE} = \frac{0 + 1 + 0.25 + 0.25}{4} = 0.375$$

## 4. Complexity

- **Time**: $O(n)$ - single pass through data
- **Space**: $O(1)$ - only accumulator variable

## 5. Use Cases

| Application | Why MSE |
|-------------|---------|
| Linear regression | Natural choice for continuous targets |
| Neural network regression | Standard output layer loss |
| Time series forecasting | Penalizes large deviations |
| Recommender systems | Predicting ratings |

## 6. Limitations

⚠️ **Sensitive to outliers**: Squaring amplifies large errors

⚠️ **Not in original units**: Harder to interpret than MAE

⚠️ **Assumes Gaussian errors**: May not be appropriate for all distributions

## 7. References

- Source: [src/machine_learning/loss_function/mean_squared_error_loss.rs](../../src/machine_learning/loss_function/mean_squared_error_loss.rs)
- Related: [Mean Absolute Error](mean_absolute_error.md), [Huber Loss](huber_loss.md)
