# Hinge Loss

## 1. Overview

**Hinge Loss** is the loss function used for training Support Vector Machines (SVMs) and other maximum-margin classifiers. It only penalizes predictions that are on the wrong side of the decision boundary or within the margin.

### Why Hinge Loss?

Hinge loss is designed for:
- Maximum-margin classification
- Binary classification with labels $\{-1, +1\}$
- Situations where correct-but-uncertain predictions shouldn't be penalized

## 2. Mathematical Foundation

### 2.1 Definition

For true labels $y \in \{-1, +1\}$ and raw predictions (scores) $\hat{y} \in \mathbb{R}$:

$$\text{Hinge Loss} = \frac{1}{n} \sum_{i=1}^{n} \max(0, 1 - y_i \cdot \hat{y}_i)$$

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | $[0, +\infty)$ |
| Perfect (confident) prediction | $0$ |
| Loss = 0 when | $y_i \cdot \hat{y}_i \geq 1$ |
| Gradient | $-y_i$ if margin violated, else $0$ |

### 2.3 Margin Interpretation

```
y·ŷ:    -∞    -1     0     1    +∞
         │     │     │     │     │
Loss:    ███████████████─────────
              ↑       ↑     ↑
          Wrong   Margin  Correct
          side    zone    confident
```

- **$y \cdot \hat{y} < 0$**: Wrong side of boundary (high loss)
- **$0 \leq y \cdot \hat{y} < 1$**: Within margin (some loss)
- **$y \cdot \hat{y} \geq 1$**: Correct and confident (zero loss)

## 3. Algorithm

### 3.1 Pseudocode

```
FUNCTION hinge_loss(y_true, y_pred)
    // y_true, y_pred ∈ {-1, +1} or continuous scores
    total_loss ← 0
    FOR i = 0 TO length(y_pred) - 1 DO
        margin ← y_true[i] × y_pred[i]
        loss ← max(0, 1 - margin)
        total_loss ← total_loss + loss
    RETURN total_loss / length(y_pred)
```

### 3.2 Example

**Predictions**: `[-1.0, 1.0, 1.0]`  
**True labels**: `[-1.0, -1.0, 1.0]`

| $i$ | $\hat{y}_i$ | $y_i$ | $y_i \cdot \hat{y}_i$ | $\max(0, 1 - \cdot)$ |
|-----|-------------|-------|----------------------|----------------------|
| 0 | -1.0 | -1.0 | 1.0 | 0.0 |
| 1 | 1.0 | -1.0 | -1.0 | 2.0 |
| 2 | 1.0 | 1.0 | 1.0 | 0.0 |

$$\text{Hinge} = \frac{0 + 2 + 0}{3} = 0.\overline{6}$$

## 4. Complexity

- **Time**: $O(n)$
- **Space**: $O(1)$

## 5. Use Cases

| Application | Why Hinge Loss |
|-------------|---------------|
| SVM classification | Maximum margin optimization |
| Text classification | Robust binary decisions |
| Image classification | When margin matters |
| One-vs-Rest multi-class | Combined with softmax alternatives |

## 6. Variants

### Squared Hinge Loss

$$\text{Squared Hinge} = \frac{1}{n} \sum_{i=1}^{n} [\max(0, 1 - y_i \cdot \hat{y}_i)]^2$$

- Differentiable at margin boundary
- Penalizes violations more heavily

### Multi-class Hinge (Weston-Watkins)

For multi-class with $C$ classes:
$$L = \sum_{j \neq y_i} \max(0, 1 + s_j - s_{y_i})$$

## 7. Comparison with Other Classification Losses

```
Loss
│
│ ╲           
│  ╲ ╲        ← Hinge (linear after margin)
│   ╲  ╲
│    ╲───────  ← Log loss (always positive)
│     ╲
└─────────→ y·ŷ
   -1  0  1  2
```

| Loss | Zero loss when | Gradient behavior |
|------|----------------|-------------------|
| Hinge | $y\hat{y} \geq 1$ | Constant or zero |
| Log loss | Never (asymptotic) | Always non-zero |
| Squared hinge | $y\hat{y} \geq 1$ | Smooth |

## 8. References

- Source: [src/machine_learning/loss_function/hinge_loss.rs](../../src/machine_learning/loss_function/hinge_loss.rs)
- Related: [Negative Log Likelihood](negative_log_likelihood.md)
- Cortes, C., & Vapnik, V. (1995). "Support-vector networks"
