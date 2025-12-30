# Cross-Entropy Loss

## 1. Overview

**Cross-entropy loss** (log loss) measures the difference between two probability distributions. It's the standard loss function for classification tasks in machine learning.

**File**: `src/math/cross_entropy_loss.rs`

## 2. Mathematical Foundation

### 2.1 Definition

For true distribution $p$ (one-hot) and predicted distribution $q$:

$$H(p, q) = -\sum_{i} p_i \log(q_i)$$

For one-hot encoding with true class $c$:
$$L = -\log(q_c)$$

### 2.2 Relationship to KL Divergence

$$H(p, q) = H(p) + D_{KL}(p \| q)$$

Where $H(p)$ is entropy and $D_{KL}$ is Kullback-Leibler divergence.

### 2.3 Binary Cross-Entropy

For binary classification with label $y \in \{0, 1\}$ and prediction $\hat{y}$:

$$L = -[y \log(\hat{y}) + (1-y) \log(1-\hat{y})]$$

## 3. Implementation

```rust
pub fn cross_entropy_loss(actual: &[f64], predicted: &[f64]) -> f64 {
    let mut loss: Vec<f64> = Vec::new();
    for (a, p) in actual.iter().zip(predicted.iter()) {
        loss.push(-a * p.ln());
    }
    loss.iter().sum()
}
```

### 3.1 Numerically Stable Version

```rust
pub fn cross_entropy_stable(actual: &[f64], predicted: &[f64]) -> f64 {
    let epsilon = 1e-15;  // Prevent log(0)
    actual.iter()
        .zip(predicted.iter())
        .map(|(&a, &p)| -a * p.clamp(epsilon, 1.0 - epsilon).ln())
        .sum()
}
```

## 4. Complexity

- **Time**: O(n) where n = number of classes
- **Space**: O(1)

## 5. Gradient

For softmax output + cross-entropy loss:

$$\frac{\partial L}{\partial z_i} = q_i - p_i$$

This simple gradient is why softmax + cross-entropy is commonly used together.

## 6. Applications

1. **Multi-class classification**: Image classification, NLP
2. **Binary classification**: Spam detection, medical diagnosis
3. **Neural network training**: Standard classification loss

## 7. Comparison with Other Losses

| Loss | Use Case | Properties |
|------|----------|------------|
| Cross-entropy | Classification | Probabilistic, penalizes confident wrong predictions |
| MSE | Regression | Sensitive to outliers |
| [Huber](huber_loss.md) | Robust regression | Less sensitive to outliers |

## 8. References

1. Murphy, K. P. "Machine Learning: A Probabilistic Perspective"
2. [Wikipedia: Cross-entropy](https://en.wikipedia.org/wiki/Cross_entropy)
