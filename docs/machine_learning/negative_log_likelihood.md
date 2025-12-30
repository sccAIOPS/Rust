# Negative Log Likelihood Loss

## 1. Overview

**Negative Log Likelihood (NLL)** is a fundamental loss function for binary and multi-class classification. It measures how well a probabilistic model's predictions match the actual labels, penalizing confident but wrong predictions heavily.

### Why NLL?

NLL (also called Log Loss or Cross-Entropy for binary classification) is the standard loss because:
- Directly maximizes the likelihood of correct predictions
- Produces well-calibrated probability estimates
- Has strong theoretical foundations in information theory
- Provides strong gradients for confident wrong predictions

## 2. Mathematical Foundation

### 2.1 Binary Classification Definition

For true labels $y \in \{0, 1\}$ and predicted probabilities $\hat{y} \in (0, 1)$:

$$\text{NLL} = -\frac{1}{n} \sum_{i=1}^{n} \left[ y_i \log(\hat{y}_i) + (1-y_i) \log(1-\hat{y}_i) \right]$$

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | $[0, +\infty)$ |
| Perfect prediction | $0$ (when $\hat{y} = y$) |
| Confident wrong | $\to +\infty$ |
| Valid inputs | $y, \hat{y} \in [0, 1]$ |

### 2.3 Gradient

$$\frac{\partial L}{\partial \hat{y}_i} = -\frac{y_i}{\hat{y}_i} + \frac{1-y_i}{1-\hat{y}_i}$$

For logistic regression with sigmoid: $\nabla = \hat{y} - y$ (elegant!)

### 2.4 Information-Theoretic View

$$\text{NLL} = H(P, Q) = H(P) + D_{KL}(P \| Q)$$

- $H(P)$ = Entropy of true distribution (constant)
- $D_{KL}$ = KL divergence from predicted to true
- Minimizing NLL = Minimizing KL divergence

## 3. Algorithm

### 3.1 Pseudocode

```
FUNCTION negative_log_likelihood(y_true, y_pred)
    // Validate inputs
    IF lengths differ THEN RETURN Error
    IF empty THEN RETURN Error
    IF any value ∉ [0, 1] THEN RETURN Error
    
    total_loss ← 0
    FOR i = 0 TO length(y_pred) - 1 DO
        loss ← -y_true[i] × log(y_pred[i]) 
               - (1 - y_true[i]) × log(1 - y_pred[i])
        total_loss ← total_loss + loss
    
    RETURN total_loss / length(y_pred)
```

### 3.2 Example

**True**: `[1.0, 0.0, 1.0]`  
**Predicted**: `[0.9, 0.1, 0.8]`

| $i$ | $y_i$ | $\hat{y}_i$ | $-y\log\hat{y}$ | $-(1-y)\log(1-\hat{y})$ | Loss |
|-----|-------|-------------|-----------------|------------------------|------|
| 0 | 1.0 | 0.9 | $-\log(0.9) = 0.105$ | $0$ | 0.105 |
| 1 | 0.0 | 0.1 | $0$ | $-\log(0.9) = 0.105$ | 0.105 |
| 2 | 1.0 | 0.8 | $-\log(0.8) = 0.223$ | $0$ | 0.223 |

$$\text{NLL} = \frac{0.105 + 0.105 + 0.223}{3} = 0.145$$

### 3.3 Penalty for Wrong Confidence

| $\hat{y}$ when $y=1$ | Loss |
|----------------------|------|
| 0.99 | 0.01 |
| 0.9 | 0.11 |
| 0.5 | 0.69 |
| 0.1 | 2.30 |
| 0.01 | 4.61 |

Being confidently wrong is **exponentially** penalized!

## 4. Complexity

- **Time**: $O(n)$
- **Space**: $O(1)$

## 5. Implementation Notes

### Rust-Specific

```rust
pub fn neg_log_likelihood(
    y_true: &[f64],
    y_pred: &[f64],
) -> Result<f64, NegativeLogLikelihoodLossError>
```

**Robust Error Handling**:
- `InputsHaveDifferentLength`: When arrays don't match
- `EmptyInputs`: When arrays are empty
- `InvalidValues`: When values outside $[0, 1]$

**Validation Function**:
```rust
fn are_all_values_in_range(values: &[f64]) -> bool {
    values.iter().all(|&x| (0.0..=1.0).contains(&x))
}
```

### Numerical Stability

⚠️ **Current implementation may produce Inf**:
- `log(0)` when $\hat{y} = 0$ or $\hat{y} = 1$ exactly
- Consider clamping: `y_pred.clamp(epsilon, 1.0 - epsilon)`

## 6. Use Cases

| Application | Why NLL |
|-------------|---------|
| **Logistic Regression** | Standard binary classification loss |
| **Neural Networks** | Output layer with softmax |
| **Medical Diagnosis** | Well-calibrated probabilities |
| **Fraud Detection** | Interpretable probability scores |

## 7. Variants

### Multi-class Cross-Entropy

For $C$ classes with one-hot labels:
$$L = -\sum_{c=1}^{C} y_c \log(\hat{y}_c)$$

### Label Smoothing

Replace hard labels with soft:
$$y_\text{smooth} = (1-\epsilon)y + \frac{\epsilon}{C}$$

Prevents overconfidence.

### Focal Loss

$$L = -\alpha(1-\hat{y})^\gamma \log(\hat{y})$$

Down-weights easy examples for imbalanced data.

## 8. Visualization

### Loss vs Predicted Probability

```
Loss
│
8 ┤ ●                          ● 
  │  ╲                        ╱
4 ┤   ╲                      ╱
  │    ╲                    ╱
2 ┤     ╲──              ──╱
  │        ╲────────────╱
0 ┤              ●
  └──┬────┬────┬────┬────┬────→ ŷ
     0   0.2  0.5  0.8   1
     
     When y=1: Loss decreases as ŷ→1
     When y=0: Loss decreases as ŷ→0
```

### Comparison with Other Losses

| Loss | Gradient at $\hat{y}=0.5$ | Behavior |
|------|---------------------------|----------|
| NLL | Strong (2.0) | Good gradient everywhere |
| Hinge | 0 or 1 | Sparse gradients |
| MSE | Moderate (0.5) | Weaker gradient |

## 9. Common Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| Loss = Inf | $\hat{y}$ exactly 0 or 1 | Clamp predictions |
| Slow convergence | Poor calibration | Add label smoothing |
| Overconfidence | Model too certain | Regularization, dropout |

## 10. References

- Source: [src/machine_learning/loss_function/negative_log_likelihood.rs](../../src/machine_learning/loss_function/negative_log_likelihood.rs)
- Related: [KL Divergence](kl_divergence.md), [Hinge Loss](hinge_loss.md)
- Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Chapter 4
- [Towards Data Science: NLL Explanation](https://towardsdatascience.com/cross-entropy-negative-log-likelihood-and-all-that-jazz-47a95bd2e81)
