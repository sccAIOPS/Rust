# Adam Optimizer

## 1. Overview

**Adam (Adaptive Moment Estimation)** is one of the most popular optimization algorithms in deep learning. It combines the benefits of two other extensions of stochastic gradient descent: **AdaGrad** (adaptive learning rates) and **RMSprop** (momentum with squared gradients).

This implementation provides a stateful optimizer that maintains first and second moment estimates of gradients, enabling efficient training with minimal hyperparameter tuning.

### Historical Context

Adam was introduced by **Diederik P. Kingma** and **Jimmy Ba** in their 2015 paper "Adam: A Method for Stochastic Optimization." It has since become the default optimizer for many deep learning applications due to its robustness and fast convergence.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a stochastic objective function $f(\boldsymbol{\theta})$ with parameters $\boldsymbol{\theta}$, iteratively update $\boldsymbol{\theta}$ to minimize $\mathbb{E}[f(\boldsymbol{\theta})]$.

### 2.2 Mathematical Model

**Hyperparameters**:
- $\alpha$ (learning rate): Default $0.001$
- $\beta_1$ (first moment decay): Default $0.9$
- $\beta_2$ (second moment decay): Default $0.999$
- $\epsilon$ (numerical stability): Default $10^{-8}$

**State Variables**:
- $\mathbf{m}_t$: First moment estimate (mean of gradients)
- $\mathbf{v}_t$: Second moment estimate (mean of squared gradients)
- $t$: Time step (iteration count)

### 2.3 Update Equations

**Initialize**: $\mathbf{m}_0 = \mathbf{0}$, $\mathbf{v}_0 = \mathbf{0}$, $t = 0$

**At each step** $t$:

1. **Get gradient**: $\mathbf{g}_t = \nabla_\theta f(\boldsymbol{\theta}_{t-1})$

2. **Update biased first moment**:
   $$\mathbf{m}_t = \beta_1 \cdot \mathbf{m}_{t-1} + (1 - \beta_1) \cdot \mathbf{g}_t$$

3. **Update biased second moment**:
   $$\mathbf{v}_t = \beta_2 \cdot \mathbf{v}_{t-1} + (1 - \beta_2) \cdot \mathbf{g}_t^2$$

4. **Bias correction**:
   $$\hat{\mathbf{m}}_t = \frac{\mathbf{m}_t}{1 - \beta_1^t}$$
   $$\hat{\mathbf{v}}_t = \frac{\mathbf{v}_t}{1 - \beta_2^t}$$

5. **Parameter update**:
   $$\boldsymbol{\theta}_t = \boldsymbol{\theta}_{t-1} - \alpha \cdot \frac{\hat{\mathbf{m}}_t}{\sqrt{\hat{\mathbf{v}}_t} + \epsilon}$$

### 2.4 Why Bias Correction?

At early iterations, $\mathbf{m}_t$ and $\mathbf{v}_t$ are biased toward zero (initialized at 0). Bias correction compensates:

| Step $t$ | $1 - \beta_1^t$ | $\hat{m}_t / m_t$ |
|----------|-----------------|-------------------|
| 1 | $0.1$ | $10\times$ |
| 5 | $0.41$ | $2.4\times$ |
| 10 | $0.65$ | $1.5\times$ |
| 100 | $\approx 1$ | $\approx 1\times$ |

## 3. Algorithm Description

### 3.1 Intuition

Adam behaves like a ball rolling down a hill with friction:
- **Momentum ($\mathbf{m}$)**: The ball accumulates velocity in consistent directions
- **Adaptive rate ($\mathbf{v}$)**: The ball slows down on steep slopes and speeds up on flat terrain
- **Bias correction**: Compensates for the ball starting from rest

### 3.2 Pseudocode

```
CLASS Adam
    CONSTRUCTOR(learning_rate=0.001, betas=(0.9, 0.999), epsilon=1e-8, params_len)
        α ← learning_rate
        β₁, β₂ ← betas
        ε ← epsilon
        m ← [0, 0, ..., 0]   // First moment, length = params_len
        v ← [0, 0, ..., 0]   // Second moment, length = params_len
        t ← 0                 // Time step
    
    FUNCTION step(gradients)
        INPUT: gradients - array of gradient values
        OUTPUT: parameter updates (to be subtracted from params)
        
        t ← t + 1
        updates ← [0, 0, ..., 0]
        
        FOR i = 0 TO length(gradients) - 1 DO
            // Update biased moments
            m[i] ← β₁ × m[i] + (1 - β₁) × gradients[i]
            v[i] ← β₂ × v[i] + (1 - β₂) × gradients[i]²
            
            // Bias-corrected moments
            m̂ ← m[i] / (1 - β₁^t)
            v̂ ← v[i] / (1 - β₂^t)
            
            // Compute update
            updates[i] ← α × m̂ / (√v̂ + ε)
        
        RETURN updates  // Subtract from parameters
```

### 3.3 Step-by-Step Example

**Setup**: 2 parameters, defaults ($\alpha=0.001$, $\beta_1=0.9$, $\beta_2=0.999$)

**Gradients**: $\mathbf{g} = [2.0, -4.0]$

| Step | Quantity | Param 0 | Param 1 |
|------|----------|---------|---------|
| **t=1** | | | |
| | $g$ | $2.0$ | $-4.0$ |
| | $m = 0.1g$ | $0.2$ | $-0.4$ |
| | $v = 0.001g^2$ | $0.004$ | $0.016$ |
| | $\hat{m} = m/0.1$ | $2.0$ | $-4.0$ |
| | $\hat{v} = v/0.001$ | $4.0$ | $16.0$ |
| | Update | $-0.001 \times 2/(\sqrt{4}+\epsilon)$ | $0.001 \times 4/(\sqrt{16}+\epsilon)$ |
| | | $\approx -0.001$ | $\approx 0.001$ |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity | Explanation |
|-----------|------------|-------------|
| Per step | $O(n)$ | Linear in number of parameters |
| Memory update | $O(n)$ | Update $\mathbf{m}$ and $\mathbf{v}$ |
| Total per epoch | $O(n \cdot B)$ | $B$ batches |

### 4.2 Space Complexity

| Type | Complexity | Explanation |
|------|------------|-------------|
| First moment $\mathbf{m}$ | $O(n)$ | Same size as parameters |
| Second moment $\mathbf{v}$ | $O(n)$ | Same size as parameters |
| **Total** | $O(2n)$ | 2× model size |

**Comparison**:
| Optimizer | Memory Overhead |
|-----------|-----------------|
| SGD | $O(1)$ |
| Momentum | $O(n)$ |
| AdaGrad | $O(n)$ |
| Adam | $O(2n)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct Adam {
    learning_rate: f64,
    betas: (f64, f64),
    epsilon: f64,
    m: Vec<f64>,
    v: Vec<f64>,
    t: usize,
}
```

**Design Patterns**:

1. **Builder-like Constructor**:
   ```rust
   Adam::new(
       learning_rate: Option<f64>,  // None → default 0.001
       betas: Option<(f64, f64)>,   // None → default (0.9, 0.999)
       epsilon: Option<f64>,        // None → default 1e-8
       params_len: usize,
   )
   ```

2. **Stateful Optimizer**:
   - Maintains moment vectors across calls
   - Time step incremented each `step()`

3. **Returns Deltas** (not updated params):
   ```rust
   let deltas = optimizer.step(&gradients);
   for (param, delta) in params.iter_mut().zip(deltas) {
       *param -= delta;  // Note: subtract (deltas are positive for descent)
   }
   ```

### 5.2 Edge Cases

| Case | Behavior | Note |
|------|----------|------|
| Empty gradients | Returns `[]` | Works correctly |
| Zero gradients | Updates tend to zero | Expected |
| Very large gradients | May need clipping | Add externally |
| `t` overflow | After $2^{64}$ steps | Not practical |

### 5.3 Current Implementation Notes

**Issue**: The current implementation returns `model_params` starting from zero, not accumulated:
```rust
model_params[i] -= self.learning_rate * m_hat / (v_hat.sqrt() + self.epsilon);
```

This returns the delta, which should be subtracted from actual parameters externally.

### 5.4 Potential Improvements

1. **Weight Decay (AdamW)**:
   ```rust
   params[i] -= weight_decay * params[i];
   ```

2. **AMSGrad** (monotonic v):
   ```rust
   v_max[i] = v_max[i].max(v[i]);
   // Use v_max instead of v for update
   ```

3. **Gradient Clipping**:
   ```rust
   fn step_with_clip(&mut self, gradients: &[f64], max_norm: f64) -> Vec<f64>
   ```

## 6. Hyperparameter Tuning

### 6.1 Default Values

| Parameter | Default | Range | Effect |
|-----------|---------|-------|--------|
| $\alpha$ | $0.001$ | $10^{-5}$ to $0.1$ | Step size |
| $\beta_1$ | $0.9$ | $0.8$ to $0.99$ | Gradient smoothing |
| $\beta_2$ | $0.999$ | $0.9$ to $0.9999$ | Squared gradient smoothing |
| $\epsilon$ | $10^{-8}$ | $10^{-10}$ to $10^{-6}$ | Numerical stability |

### 6.2 Tuning Guidelines

```
Problem: Loss not decreasing
→ Increase α (learning rate)

Problem: Loss oscillating
→ Decrease α or increase β₁

Problem: Slow convergence late in training
→ Decrease α with scheduling

Problem: NaN values
→ Increase ε or add gradient clipping
```

## 7. Real-World Applications

### 7.1 Deep Learning Use Cases

| Domain | Application | Why Adam |
|--------|-------------|----------|
| **NLP** | Transformer training | Handles varying gradient magnitudes |
| **Computer Vision** | CNN training | Fast convergence |
| **Reinforcement Learning** | Policy optimization | Stable with noisy gradients |
| **GANs** | Generator/Discriminator | Handles non-stationary objectives |

### 7.2 When to Use Adam

✅ **Use Adam when**:
- First attempt at a new problem
- Limited time for hyperparameter tuning
- Sparse gradients (NLP, embeddings)
- Non-stationary objectives

❌ **Consider alternatives when**:
- Fine-tuning pretrained models (SGD often better)
- Achieving absolute best performance (SGD + scheduler)
- Memory constrained (SGD uses less)

### 7.3 Related Optimizers

| Optimizer | Relationship |
|-----------|--------------|
| [Gradient Descent](gradient_descent.md) | Foundation |
| [Momentum](momentum.md) | First moment only |
| **AdaGrad** | Per-parameter rates (no moment) |
| **RMSprop** | Running average of squared gradients |
| **AdamW** | Adam with decoupled weight decay |
| **LAMB** | Layer-wise adaptive for large batches |

## 8. Visualization

### Adam vs Other Optimizers

```
Loss vs Iterations (Log Scale)
    │
10² │●═══════════════════════════════════  SGD
    │  ╲
10¹ │   ╲●════════════════════════════════  Momentum
    │    ╲
10⁰ │     ╲●══════════════════════════════  RMSprop
    │      ╲
10⁻¹│       ╲●════════════════════════════  Adam
    └─────────────────────────────────────→
         Iterations
```

### Adaptive Learning Rate Effect

```
Parameter Space with Contours
    │       
    │   ╭───SGD (zigzag)
    │  ╱    
    │ ╱ ╭───Adam (direct path)
    │╱ ╱
    ●═════●  Start → Minimum
```

## 9. Common Issues

| Problem | Symptom | Solution |
|---------|---------|----------|
| **Sharp minima** | Poor generalization | Lower $\alpha$, use SGD for fine-tuning |
| **Slow late convergence** | Loss plateaus | Learning rate warmup/decay |
| **Memory issues** | OOM errors | Gradient checkpointing, smaller batch |
| **Numerical instability** | NaN/Inf | Increase $\epsilon$, gradient clipping |

## 10. References

### Academic Papers
- Kingma, D. P., & Ba, J. (2015). "Adam: A Method for Stochastic Optimization." [arXiv:1412.6980](https://arxiv.org/abs/1412.6980)
- Loshchilov, I., & Hutter, F. (2019). "Decoupled Weight Decay Regularization" (AdamW)
- Reddi, S. J., et al. (2018). "On the Convergence of Adam and Beyond" (AMSGrad)

### Books
- Goodfellow, I., et al. (2016). *Deep Learning*, Chapter 8

### Online Resources
- [PyTorch Adam Documentation](https://pytorch.org/docs/stable/generated/torch.optim.Adam.html)
- [Sebastian Ruder's Optimizer Overview](https://ruder.io/optimizing-gradient-descent/)

### Implementation Reference
- Source: [src/machine_learning/optimization/adam.rs](../../src/machine_learning/optimization/adam.rs)
