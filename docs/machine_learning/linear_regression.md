# Linear Regression

## 1. Overview

**Linear Regression** is one of the most fundamental and widely used supervised machine learning algorithms. It models the relationship between a dependent variable (target) and one or more independent variables (features) by fitting a linear equation to observed data.

The implementation in this repository performs **Simple Linear Regression**, which finds the best-fit line through a set of 2D data points by minimizing the sum of squared residuals.

### Historical Context

Linear regression was first introduced by **Francis Galton** in the 1880s while studying the relationship between heights of parents and children. The term "regression" comes from his observation that children's heights "regressed" toward the population mean. The mathematical foundation was later formalized by **Karl Pearson** and others.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a dataset of $n$ points $(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)$, find the line $y = a + bx$ that best fits the data, where:
- $a$ is the y-intercept
- $b$ is the slope

### 2.2 Mathematical Model

**Input**: Vector of $(x, y)$ coordinate pairs

**Output**: Parameters $(a, b)$ defining the line $y = a + bx$

**Objective**: Minimize the sum of squared residuals (SSR):

$$\text{SSR} = \sum_{i=1}^{n} (y_i - (a + bx_i))^2$$

### 2.3 Derivation

The optimal parameters are derived using the **Ordinary Least Squares (OLS)** method:

**Pearson Correlation Coefficient** ($r$):
$$r = \frac{\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum_{i=1}^{n}(x_i - \bar{x})^2} \cdot \sqrt{\sum_{i=1}^{n}(y_i - \bar{y})^2}} = \frac{\text{Cov}(X,Y)}{\sigma_X \cdot \sigma_Y}$$

**Slope** ($b$):
$$b = r \cdot \frac{\sigma_Y}{\sigma_X}$$

**Intercept** ($a$):
$$a = \bar{y} - b \cdot \bar{x}$$

Where:
- $\bar{x}, \bar{y}$ are the means of $x$ and $y$ values
- $\sigma_X, \sigma_Y$ are the standard deviations

### 2.4 Correctness Proof

The OLS estimators are the **Best Linear Unbiased Estimators (BLUE)** under the Gauss-Markov assumptions:
1. Linear relationship between variables
2. Errors have zero mean
3. Homoscedasticity (constant variance of errors)
4. No autocorrelation of errors

## 3. Algorithm Description

### 3.1 Intuition

Linear regression finds the line that minimizes the vertical distances between the data points and the fitted line. Imagine stretching a rubber band through a scatter plot - the band naturally settles at the position that minimizes total "tension" (squared distances).

### 3.2 Pseudocode

```
FUNCTION linear_regression(data_points)
    INPUT: Array of (x, y) coordinate pairs
    OUTPUT: Tuple (intercept a, slope b) or None if empty
    
    IF data_points is empty THEN
        RETURN None
    
    n ← length of data_points
    
    // Calculate means
    mean_x ← sum(x for all points) / n
    mean_y ← sum(y for all points) / n
    
    // Calculate covariance and standard deviations
    covariance ← 0
    variance_x ← 0
    variance_y ← 0
    
    FOR each (x, y) in data_points DO
        covariance ← covariance + (x - mean_x) * (y - mean_y)
        variance_x ← variance_x + (x - mean_x)²
        variance_y ← variance_y + (y - mean_y)²
    
    std_x ← √variance_x
    std_y ← √variance_y
    
    // Calculate Pearson correlation coefficient
    r ← covariance / (std_x * std_y)
    
    // Calculate slope and intercept
    b ← r * (std_y / std_x)
    a ← mean_y - b * mean_x
    
    RETURN (a, b)
```

### 3.3 Step-by-Step Example

**Input**: `[(0, 0), (1, 1), (2, 2)]`

| Step | Calculation | Result |
|------|-------------|--------|
| 1 | $\bar{x} = (0 + 1 + 2)/3$ | $1.0$ |
| 2 | $\bar{y} = (0 + 1 + 2)/3$ | $1.0$ |
| 3 | $\text{Cov} = (0-1)(0-1) + (1-1)(1-1) + (2-1)(2-1)$ | $2.0$ |
| 4 | $\sigma_X^2 = (0-1)^2 + (1-1)^2 + (2-1)^2$ | $2.0$ |
| 5 | $\sigma_Y^2 = 2.0$ | $2.0$ |
| 6 | $r = 2.0 / (\sqrt{2} \cdot \sqrt{2})$ | $1.0$ |
| 7 | $b = 1.0 \cdot (\sqrt{2} / \sqrt{2})$ | $1.0$ |
| 8 | $a = 1.0 - 1.0 \cdot 1.0$ | $0.0$ |

**Output**: Line equation $y = 0 + 1 \cdot x = x$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| All Cases | $O(n)$ | Single pass to compute means, second pass for statistics |

**Derivation**: The algorithm performs two passes over the data:
1. First fold: Calculate sum of x and y values → $O(n)$
2. Second pass: Calculate covariance and variances → $O(n)$
3. Final calculations: Constant time → $O(1)$

**Total**: $O(n) + O(n) + O(1) = O(n)$

### 4.2 Space Complexity

| Type | Complexity | Explanation |
|------|------------|-------------|
| Auxiliary Space | $O(1)$ | Only scalar variables used |
| Input Space | $O(n)$ | Stores $n$ data points |

**Note**: The implementation consumes the input `Vec<(f64, f64)>` by ownership, not borrowing.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn linear_regression(data_points: Vec<(f64, f64)>) -> Option<(f64, f64)>
```

**Ownership Pattern**: 
- Takes ownership of `data_points` (could be optimized to borrow `&[(f64, f64)]`)
- Returns `Option` to handle empty input gracefully

**Iterator Usage**:
- Uses `fold` for mean calculation (functional style)
- Traditional `for` loop for variance calculation (more readable)

**Numerical Considerations**:
- Uses `powi(2)` for squaring (integer exponent is faster than `powf`)
- Division by zero not explicitly handled (produces `NaN`/`Inf`)

### 5.2 Edge Cases

| Case | Behavior | Note |
|------|----------|------|
| Empty input | Returns `None` | Handled explicitly |
| Single point | May produce `NaN` | Division by zero in std_dev |
| All same x values | Undefined slope | Vertical line case |
| All same y values | Slope = 0 | Horizontal line |
| Perfect collinear | Exact fit | $r = \pm 1$ |

### 5.3 Potential Improvements

1. **Borrow input**: Change to `&[(f64, f64)]` to avoid consuming the data
2. **Numerical stability**: Use Welford's online algorithm for variance
3. **Return more statistics**: Include $R^2$, standard error, p-values
4. **Handle edge cases**: Return `None` for single point or zero variance

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

| Domain | Application | Example |
|--------|-------------|---------|
| Performance Analysis | Predict execution time from input size | `time = a + b * n` |
| Resource Planning | Estimate server costs | `cost = a + b * users` |
| Quality Metrics | Correlate code complexity with bugs | `bugs = a + b * cyclomatic_complexity` |
| A/B Testing | Model conversion rate trends | `conversions = a + b * time` |

### 6.2 Industry Applications

- **Finance**: Stock price prediction, risk assessment
- **Marketing**: Sales forecasting from advertising spend
- **Healthcare**: Drug dosage calculation based on patient weight
- **Real Estate**: House price estimation from square footage

### 6.3 Related Algorithms

| Algorithm | When to Use |
|-----------|-------------|
| **Multiple Linear Regression** | Multiple independent variables |
| **Polynomial Regression** | Non-linear relationships |
| **Ridge/Lasso Regression** | When regularization is needed |
| **Logistic Regression** | Binary classification problems |

## 7. References

### Academic Papers
- Galton, F. (1886). "Regression Towards Mediocrity in Hereditary Stature"
- Stigler, S. M. (1981). "Gauss and the Invention of Least Squares"

### Books
- Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*, Chapter 3
- Hastie, T., Tibshirani, R., & Friedman, J. (2009). *The Elements of Statistical Learning*

### Implementation Reference
- Source: [src/machine_learning/linear_regression.rs](../../src/machine_learning/linear_regression.rs)
