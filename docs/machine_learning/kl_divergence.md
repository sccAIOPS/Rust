# KL Divergence Loss

## 1. Overview

**Kullback-Leibler (KL) Divergence** measures how one probability distribution diverges from a second, reference distribution. In machine learning, it quantifies the information lost when using the predicted distribution to approximate the actual distribution.

### Why KL Divergence?

KL divergence is essential for:
- Variational autoencoders (VAE regularization)
- Knowledge distillation
- Comparing probability distributions
- Information-theoretic analysis

## 2. Mathematical Foundation

### 2.1 Definition

For discrete probability distributions $P$ (actual) and $Q$ (predicted):

$$D_{KL}(P \| Q) = \sum_{i} P(i) \log\frac{P(i)}{Q(i)}$$

Or equivalently:
$$D_{KL}(P \| Q) = -\sum_{i} P(i) \log Q(i) + \sum_{i} P(i) \log P(i)$$
$$= H(P, Q) - H(P)$$

Where:
- $H(P, Q)$ = Cross-entropy
- $H(P)$ = Entropy of P

### 2.2 Properties

| Property | Value |
|----------|-------|
| Range | $[0, +\infty)$ |
| $D_{KL}(P \| P)$ | $0$ |
| Symmetric? | **No**: $D_{KL}(P \| Q) \neq D_{KL}(Q \| P)$ |
| Triangle inequality? | **No** |

### 2.3 Asymmetry Intuition

```
P = [0.9, 0.1]    Q = [0.5, 0.5]

D_KL(P||Q) = 0.9 log(0.9/0.5) + 0.1 log(0.1/0.5) = 0.368
D_KL(Q||P) = 0.5 log(0.5/0.9) + 0.5 log(0.5/0.1) = 0.804

Different values!
```

### 2.4 Forward vs Reverse KL

| Mode | Formula | Behavior |
|------|---------|----------|
| Forward KL | $D_{KL}(P \| Q)$ | Mean-seeking (covers all modes) |
| Reverse KL | $D_{KL}(Q \| P)$ | Mode-seeking (may miss modes) |

## 3. Algorithm

### 3.1 Pseudocode

```
FUNCTION kl_divergence(actual, predicted)
    ε ← 1e-5  // Prevent log(0)
    loss ← 0
    
    FOR i = 0 TO length(actual) - 1 DO
        a ← actual[i] + ε
        p ← predicted[i] + ε
        loss ← loss + a × log(a / p)
    
    RETURN loss
```

### 3.2 Example

**Actual**: `[1.346, 1.337, 1.247]` (unnormalized)  
**Predicted**: `[1.034, 1.082, 1.117]`

| $i$ | $a_i$ | $p_i$ | $a_i \log(a_i/p_i)$ |
|-----|-------|-------|---------------------|
| 0 | 1.346 | 1.034 | $1.346 \times 0.264 = 0.355$ |
| 1 | 1.337 | 1.082 | $1.337 \times 0.211 = 0.282$ |
| 2 | 1.247 | 1.117 | $1.247 \times 0.110 = 0.138$ |

$$D_{KL} = 0.355 + 0.282 + 0.138 = 0.775$$

## 4. Complexity

- **Time**: $O(n)$
- **Space**: $O(1)$

## 5. Use Cases

| Application | Usage |
|-------------|-------|
| **VAE** | Regularize latent space: $D_{KL}(q(z|x) \| p(z))$ |
| **Knowledge Distillation** | Match student to teacher outputs |
| **Language Models** | Measure distribution shift |
| **Reinforcement Learning** | Policy gradient regularization (PPO) |
| **Information Theory** | Measure coding inefficiency |

## 6. Implementation Notes

### Numerical Stability

The implementation adds $\epsilon = 10^{-5}$ to prevent:
- $\log(0) = -\infty$ when $p_i = 0$
- Division by zero in $p_i/q_i$

### Rust-Specific

```rust
pub fn kld_loss(actual: &[f64], predicted: &[f64]) -> f64
```

- Uses iterators with `zip` and `map`
- Adds epsilon internally for stability
- Returns raw sum (not averaged)

### Potential Issues

⚠️ **Not normalized**: Returns sum, not mean  
⚠️ **Assumes same length**: No validation  
⚠️ **Unnormalized inputs**: Works but interpretation changes

## 7. Variants

### Symmetric KL (Jensen-Shannon Divergence)

$$D_{JS}(P \| Q) = \frac{1}{2}D_{KL}(P \| M) + \frac{1}{2}D_{KL}(Q \| M)$$

Where $M = \frac{1}{2}(P + Q)$

- Symmetric: $D_{JS}(P \| Q) = D_{JS}(Q \| P)$
- Bounded: $[0, \log 2]$

### Cross-Entropy Loss

$$H(P, Q) = -\sum_i P(i) \log Q(i) = D_{KL}(P \| Q) + H(P)$$

Since $H(P)$ is constant during training, minimizing cross-entropy = minimizing KL divergence.

## 8. Visualization

### Distribution Comparison

```
Actual P        Predicted Q      KL Divergence
   │ █                │ █        Measures "surprise"
   │ █ █              │ █ █      when using Q
   │ █ █ █            │ █ █ █    to code P
   └─────→            └─────→
```

### Forward vs Reverse KL Behavior

```
True Distribution P (bimodal)
      ╱╲    ╱╲
     ╱  ╲  ╱  ╲
────╱────╲╱────╲────

Forward KL: Q covers both modes (spread out)
      ╱────────╲
     ╱          ╲
────╱────────────╲──

Reverse KL: Q picks one mode (concentrated)
      ╱╲
     ╱  ╲
────╱────╲──────────
```

## 9. References

- Source: [src/machine_learning/loss_function/kl_divergence_loss.rs](../../src/machine_learning/loss_function/kl_divergence_loss.rs)
- Related: [Negative Log Likelihood](negative_log_likelihood.md)
- Kullback, S., & Leibler, R. A. (1951). "On Information and Sufficiency"
- Cover, T. M., & Thomas, J. A. (2006). *Elements of Information Theory*
