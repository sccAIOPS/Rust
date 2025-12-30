# Logistic Regression

## 1. Overview

**Logistic Regression** is a fundamental supervised machine learning algorithm used for binary classification problems. Despite its name containing "regression," it is primarily used for classification by modeling the probability that an instance belongs to a particular class.

The implementation in this repository uses **gradient descent optimization** to find optimal weights for the logistic function, making it capable of learning from multiple features.

### Historical Context

Logistic regression was developed by statistician **David Cox** in 1958. The logistic function itself was earlier discovered by **Pierre François Verhulst** in the 1830s-1840s for modeling population growth. The algorithm became a cornerstone of modern machine learning and deep learning, as neurons in neural networks are essentially logistic regression units.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a labeled dataset $\{(\mathbf{x}_i, y_i)\}_{i=1}^{n}$ where:
- $\mathbf{x}_i \in \mathbb{R}^d$ is a feature vector
- $y_i \in \{0, 1\}$ is the binary label

Find parameters $\boldsymbol{\theta} = (\theta_0, \theta_1, \ldots, \theta_d)$ such that:

$$P(y = 1 | \mathbf{x}) = \sigma(\boldsymbol{\theta}^T \mathbf{x}) = \frac{1}{1 + e^{-\boldsymbol{\theta}^T \mathbf{x}}}$$

### 2.2 Mathematical Model

**Sigmoid (Logistic) Function**:
$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

**Linear Combination**:
$$z = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \ldots + \theta_d x_d = \boldsymbol{\theta}^T \mathbf{x}$$

**Decision Rule**:
$$\hat{y} = \begin{cases} 1 & \text{if } \sigma(z) \geq 0.5 \\ 0 & \text{otherwise} \end{cases}$$

### 2.3 Loss Function

The **Binary Cross-Entropy Loss** (Log Loss) is used:

$$J(\boldsymbol{\theta}) = -\frac{1}{n} \sum_{i=1}^{n} \left[ y_i \log(\hat{y}_i) + (1 - y_i) \log(1 - \hat{y}_i) \right]$$

### 2.4 Gradient Derivation

The gradient of the loss function with respect to parameters:

$$\frac{\partial J}{\partial \theta_j} = \frac{1}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i) \cdot x_{ij}$$

Where $\hat{y}_i = \sigma(\boldsymbol{\theta}^T \mathbf{x}_i)$

## 3. Algorithm Description

### 3.1 Intuition

Logistic regression finds a decision boundary that separates two classes. The sigmoid function "squashes" any real number into a probability between 0 and 1. The algorithm adjusts the boundary (parameters) iteratively using gradient descent to minimize classification errors.

```
     1 ┤     ╭──────────────
       │    ╱
σ(z)   │   ╱
       │  ╱
     0 ┤─╱───────────────────
       └────────────────────→ z
              Decision boundary at z = 0
```

### 3.2 Pseudocode

```
FUNCTION logistic_regression(data_points, iterations, learning_rate)
    INPUT: 
        data_points: Array of (features[], label) pairs
        iterations: Number of gradient descent iterations
        learning_rate: Step size for optimization
    OUTPUT: Weight vector θ or None if empty
    
    IF data_points is empty THEN
        RETURN None
    
    num_features ← length of first feature vector + 1  // +1 for bias
    params ← [0, 0, ..., 0]  // Initialize weights to zero
    
    FOR iter = 1 TO iterations DO
        gradients ← compute_gradient(params, data_points)
        FOR j = 0 TO num_features - 1 DO
            params[j] ← params[j] - learning_rate × gradients[j]
    
    RETURN params

FUNCTION compute_gradient(params, data_points)
    gradients ← [0, 0, ..., 0]
    
    FOR each (features, y) in data_points DO
        // Compute linear combination
        z ← params[0] + Σ(params[j+1] × features[j])
        
        // Apply sigmoid
        prediction ← 1 / (1 + e^(-z))
        
        // Accumulate gradient for bias
        gradients[0] ← gradients[0] + (prediction - y)
        
        // Accumulate gradient for each feature
        FOR j = 0 TO length(features) - 1 DO
            gradients[j+1] ← gradients[j+1] + (prediction - y) × features[j]
    
    RETURN gradients
```

### 3.3 Step-by-Step Example

**Input**: 
- Data: `[([0], 0), ([1], 0), ([2], 0), ([3], 1), ([4], 1), ([5], 1)]`
- Iterations: 100, Learning rate: 0.05

**Initial**: $\theta_0 = 0, \theta_1 = 0$

| Iteration | $\theta_0$ | $\theta_1$ | Decision Boundary |
|-----------|------------|------------|-------------------|
| 0 | 0.0 | 0.0 | $x = 0$ |
| 10 | -1.2 | 0.5 | $x \approx 2.4$ |
| 50 | -5.8 | 2.3 | $x \approx 2.5$ |
| 100 | -8.9 | 3.6 | $x \approx 2.5$ |
| Final | ≈-17.6 | ≈7.1 | $x \approx 2.5$ |

The decision boundary converges to $x \approx 2.5$ (between classes 0 and 1).

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| Per Iteration | $O(n \cdot d)$ | Iterate over $n$ samples, $d$ features each |
| Total | $O(k \cdot n \cdot d)$ | For $k$ iterations |

Where:
- $n$ = number of training samples
- $d$ = number of features
- $k$ = number of iterations

### 4.2 Space Complexity

| Type | Complexity | Explanation |
|------|------------|-------------|
| Parameters | $O(d)$ | Weight vector |
| Gradients | $O(d)$ | Gradient vector |
| Input Storage | $O(n \cdot d)$ | Training data |
| **Total Auxiliary** | $O(d)$ | Excluding input |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn logistic_regression(
    data_points: Vec<(Vec<f64>, f64)>,
    iterations: usize,
    learning_rate: f64,
) -> Option<Vec<f64>>
```

**Module Integration**:
- Uses `gradient_descent` from `super::optimization`
- Cleanly separates optimization logic from model logic

**Closure Usage**:
```rust
let derivative_fn = |params: &[f64]| derivative(params, &data_points);
```
Creates a closure that captures the data points for use in gradient descent.

**Numerical Stability**:
- Uses `std::f64::consts::E` for Euler's number
- Potential overflow with very large/small z values (sigmoid saturation)

### 5.2 Edge Cases

| Case | Behavior | Recommendation |
|------|----------|----------------|
| Empty input | Returns `None` | Handled correctly |
| Single sample | Works but poor generalization | Need more data |
| Linearly separable | Weights grow unbounded | Add regularization |
| All same label | $\theta_0 \to \pm\infty$, others $\to 0$ | Check data quality |
| Missing features | Crashes (panic) | Validate input |

### 5.3 Potential Improvements

1. **Regularization**: Add L2 penalty to prevent overfitting
   ```rust
   gradient[j] += lambda * params[j];  // L2 regularization
   ```

2. **Batch Processing**: Implement mini-batch gradient descent for large datasets

3. **Early Stopping**: Converge when loss change falls below threshold

4. **Feature Scaling**: Normalize inputs for faster convergence

5. **Numerical Stability**: Use log-sum-exp trick for sigmoid

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

| Domain | Application | Example |
|--------|-------------|---------|
| **Spam Detection** | Classify emails | Features: word frequencies, sender reputation |
| **Code Review** | Predict defect-prone commits | Features: lines changed, complexity metrics |
| **User Churn** | Predict user abandonment | Features: activity metrics, time since last login |
| **Security** | Malware detection | Features: API calls, file characteristics |

### 6.2 Industry Applications

- **Healthcare**: Disease diagnosis (benign vs. malignant tumors)
- **Finance**: Credit approval, fraud detection
- **Marketing**: Customer conversion prediction
- **NLP**: Sentiment analysis (positive/negative)

### 6.3 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| **Perceptron** | Logistic regression without probabilistic output |
| **Neural Networks** | Composition of logistic units |
| **Softmax Regression** | Multi-class extension |
| **SVM** | Different loss function (hinge loss) |
| **Naive Bayes** | Generative vs. discriminative approach |

## 7. Visualization

### Decision Boundary

```
y (label)
│
1 ┼ ● ● ●     ← Class 1 (y=1)
│    ↑
│    │ Decision Boundary
│    ↓
0 ┼ ○ ○ ○     ← Class 0 (y=0)
└──┼──┼──┼──┼──→ x (feature)
   0  1  2  3
```

### Sigmoid Function Properties

| Property | Value |
|----------|-------|
| $\sigma(0)$ | $0.5$ |
| $\sigma(\infty)$ | $1.0$ |
| $\sigma(-\infty)$ | $0.0$ |
| $\sigma'(z)$ | $\sigma(z)(1-\sigma(z))$ |

## 8. References

### Academic Papers
- Cox, D. R. (1958). "The Regression Analysis of Binary Sequences"
- Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Chapter 4

### Online Resources
- [Stanford CS229 Lecture Notes](http://cs229.stanford.edu/notes/cs229-notes1.pdf)
- [Scikit-learn Logistic Regression](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression)

### Implementation Reference
- Source: [src/machine_learning/logistic_regression.rs](../../src/machine_learning/logistic_regression.rs)
- Dependencies: [src/machine_learning/optimization/gradient_descent.rs](../../src/machine_learning/optimization/gradient_descent.rs)
