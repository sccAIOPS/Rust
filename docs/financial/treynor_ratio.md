# Treynor Ratio

## 1. Overview

The Treynor Ratio, also known as the reward-to-volatility ratio, is a performance metric that measures the risk-adjusted return of an investment portfolio or trading strategy. Named after Jack L. Treynor (one of the developers of the Capital Asset Pricing Model), it evaluates how much excess return is generated per unit of systematic (market) risk.

Unlike the Sharpe Ratio which uses total volatility (standard deviation), the Treynor Ratio specifically uses **beta** ($\beta$) as its risk measure. This makes it particularly useful for well-diversified portfolios where unsystematic (idiosyncratic) risk has been largely eliminated.

**Key Innovation**: The Treynor Ratio focuses on systematic risk (beta), recognizing that diversified investors should only be concerned with non-diversifiable market risk.

**Historical Context**: Introduced by Jack L. Treynor in 1965, it was one of the first risk-adjusted performance measures and remains fundamental in modern portfolio theory.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a portfolio's return $R_p$, the risk-free rate $R_f$, and the portfolio's beta $\beta_p$, calculate the excess return per unit of systematic risk.

**Formal Definition**:
$$\text{Treynor Ratio} = \frac{R_p - R_f}{\beta_p}$$

Where:
- $R_p$ = Portfolio return (or expected return)
- $R_f$ = Risk-free rate (e.g., Treasury bill rate)
- $\beta_p$ = Portfolio beta (systematic risk measure)

### 2.2 Mathematical Model

**Input Specifications**:
- `portfolio_return`: $R_p \in \mathbb{R}$ (typically as decimal, e.g., 0.12 for 12%)
- `risk_free_rate`: $R_f \in \mathbb{R}$ (typically $0 \leq R_f < R_p$)
- `beta`: $\beta_p \in \mathbb{R}$ (typically $\beta > 0$, but can be negative)

**Output Specification**:
- Real number representing excess return per unit of beta
- `NaN` if $\beta = 0$ (undefined)

**Constraints**:
- $\beta \neq 0$ (division by zero undefined)
- For meaningful interpretation: $\beta > 0$ (positive market exposure)

**Special Cases**:
- $\beta = 0$: Treynor Ratio undefined (returns `NaN`)
- $\beta < 0$: Negative beta (inverse market correlation) - valid but rare
- $R_p < R_f$: Negative excess return (underperformance)

### 2.3 Key Mathematical Concepts

#### 2.3.1 Beta (β)

**Definition**: Measure of systematic risk relative to the market.

$$\beta_p = \frac{\text{Cov}(R_p, R_m)}{\text{Var}(R_m)} = \frac{\sigma_{p,m}}{\sigma_m^2}$$

Where:
- $R_m$ = Market return
- $\text{Cov}(R_p, R_m)$ = Covariance between portfolio and market
- $\text{Var}(R_m) = \sigma_m^2$ = Variance of market returns

**Interpretation**:
- $\beta = 1.0$: Moves in line with market (average risk)
- $\beta > 1.0$: More volatile than market (high risk)
- $0 < \beta < 1.0$: Less volatile than market (low risk)
- $\beta = 0$: No correlation with market (risk-free asset)
- $\beta < 0$: Moves opposite to market (hedge)

**Examples**:
- U.S. Treasury Bonds: $\beta \approx 0$
- S&P 500 Index Fund: $\beta = 1.0$ (by definition)
- Tech Growth Stocks: $\beta \approx 1.3-1.8$
- Utility Stocks: $\beta \approx 0.3-0.7$
- Gold: $\beta \approx -0.1$ to $0.2$

#### 2.3.2 Excess Return

**Risk Premium**:
$$\text{Excess Return} = R_p - R_f$$

This represents the additional return earned for taking on market risk beyond the risk-free rate.

**Capital Asset Pricing Model (CAPM)** Context:
$$E[R_p] = R_f + \beta_p(E[R_m] - R_f)$$

The Treynor Ratio can be seen as:
$$\text{Treynor Ratio} = \frac{R_p - R_f}{\beta_p}$$

Comparing actual excess return per unit beta to what CAPM predicts.

### 2.4 Mathematical Properties

**1. Scale Invariance**:
Multiplying returns by constant doesn't change ranking:
$$\frac{k(R_p - R_f)}{k\beta_p} = \frac{R_p - R_f}{\beta_p}$$

**2. Non-Linearity**:
Not additive across portfolios:
$$\text{TR}(P_1 + P_2) \neq \text{TR}(P_1) + \text{TR}(P_2)$$

**3. Comparison Property**:
Higher Treynor Ratio indicates better risk-adjusted performance (for $\beta > 0$):
$$\text{TR}_A > \text{TR}_B \implies A \text{ outperforms } B \text{ on risk-adjusted basis}$$

**4. Relationship to Sharpe Ratio**:
$$\text{Treynor Ratio} = \frac{R_p - R_f}{\beta_p}$$
$$\text{Sharpe Ratio} = \frac{R_p - R_f}{\sigma_p}$$

Connection:
$$\text{Treynor} = \frac{\text{Sharpe}}{\rho_{p,m}}$$

Where $\rho_{p,m}$ is correlation between portfolio and market.

## 3. Algorithm Description

### 3.1 Intuition

The Treynor Ratio answers: "How much extra return do I get for each unit of market risk I take?"

Think of it as **bang for your buck** in terms of systematic risk:
- Higher ratio = More efficient use of market risk
- Lower ratio = Less compensation for market exposure

**Why Beta?**: For well-diversified portfolios, unsystematic risk can be diversified away, so only systematic (beta) risk matters. Treynor recognizes this by focusing solely on beta.

**Practical Interpretation**:
- Portfolio A: 12% return, beta = 1.2, risk-free = 2%
  - Treynor = (0.12 - 0.02) / 1.2 = 0.0833
- Portfolio B: 10% return, beta = 0.8, risk-free = 2%
  - Treynor = (0.10 - 0.02) / 0.8 = 0.100

Despite lower absolute return, Portfolio B has better risk-adjusted performance.

### 3.2 Pseudocode

```
Algorithm: Treynor Ratio
Input: portfolio_return, risk_free_rate, beta
Output: treynor_ratio or NaN

1. If beta == 0:
2.     Return NaN  // Undefined
3. 
4. excess_return ← portfolio_return - risk_free_rate
5. treynor_ratio ← excess_return / beta
6. Return treynor_ratio
```

**Alternative with Error Handling**:
```
Algorithm: Treynor Ratio (Robust)
Input: portfolio_return, risk_free_rate, beta
Output: treynor_ratio or Error

1. If beta == 0:
2.     Return Error("Beta cannot be zero")
3. If |beta| < EPSILON:
4.     Return Error("Beta too close to zero")
5. 
6. excess_return ← portfolio_return - risk_free_rate
7. treynor_ratio ← excess_return / beta
8. Return Ok(treynor_ratio)
```

### 3.3 Step-by-Step Examples

#### Example 1: Well-Diversified Portfolio

**Scenario**: Evaluating a diversified equity portfolio

**Given**:
- Portfolio return: $R_p = 10\%$ = 0.10
- Risk-free rate: $R_f = 5\%$ = 0.05
- Beta: $\beta = 1.50$

**Calculation**:

```
Step 1: Calculate excess return
  Excess = 0.10 - 0.05 = 0.05 (5%)

Step 2: Check beta
  beta = 1.50 ≠ 0 ✓ (valid)

Step 3: Divide excess return by beta
  Treynor = 0.05 / 1.50 = 0.03333

Result: Treynor Ratio = 0.0333 or 3.33%
```

**Interpretation**: Portfolio earns 3.33% excess return per unit of systematic risk. For every 1.0 of beta exposure, it generates 3.33% above the risk-free rate.

---

#### Example 2: Conservative Portfolio

**Scenario**: Low-beta defensive portfolio

**Given**:
- Portfolio return: $R_p = 8\%$ = 0.08
- Risk-free rate: $R_f = 5\%$ = 0.05
- Beta: $\beta = 0.60$

**Calculation**:

```
Step 1: Excess return
  Excess = 0.08 - 0.05 = 0.03 (3%)

Step 2: Treynor calculation
  Treynor = 0.03 / 0.60 = 0.05

Result: Treynor Ratio = 0.05 or 5.0%
```

**Interpretation**: Despite lower absolute return (8% vs. 10% in Example 1), this portfolio has a **better** Treynor Ratio (5% vs. 3.33%), indicating superior risk-adjusted performance.

---

#### Example 3: Market Index Fund

**Scenario**: S&P 500 index fund

**Given**:
- Portfolio return: $R_p = 12\%$ = 0.12
- Risk-free rate: $R_f = 3\%$ = 0.03
- Beta: $\beta = 1.0$ (by definition)

**Calculation**:

```
Excess = 0.12 - 0.03 = 0.09
Treynor = 0.09 / 1.0 = 0.09

Result: Treynor Ratio = 0.09 or 9%
```

**Interpretation**: The market index earns 9% excess return per unit of beta. This serves as a benchmark for actively managed portfolios.

---

#### Example 4: Zero Beta (Edge Case)

**Scenario**: Risk-free asset or market-neutral strategy

**Given**:
- Portfolio return: $R_p = 5\%$ = 0.05
- Risk-free rate: $R_f = 5\%$ = 0.05
- Beta: $\beta = 0.0$

**Calculation**:

```
Excess = 0.05 - 0.05 = 0.00
Treynor = 0.00 / 0.00 = NaN (undefined)

Result: NaN
```

**Interpretation**: Treynor Ratio is undefined for zero-beta portfolios. The metric is not meaningful when there's no systematic risk.

---

#### Example 5: Negative Beta (Hedge)

**Scenario**: Portfolio with negative market correlation

**Given**:
- Portfolio return: $R_p = 6\%$ = 0.06
- Risk-free rate: $R_f = 3\%$ = 0.03
- Beta: $\beta = -0.50$

**Calculation**:

```
Excess = 0.06 - 0.03 = 0.03
Treynor = 0.03 / (-0.50) = -0.06

Result: Treynor Ratio = -0.06
```

**Interpretation**: Negative Treynor Ratio due to negative beta. Interpretation is tricky: the portfolio provides positive absolute return while hedging market risk. Traditional Treynor comparison doesn't apply cleanly to negative beta assets.

---

#### Example 6: Underperforming Portfolio

**Scenario**: Portfolio failing to beat risk-free rate

**Given**:
- Portfolio return: $R_p = 4\%$ = 0.04
- Risk-free rate: $R_f = 5\%$ = 0.05
- Beta: $\beta = 1.20$

**Calculation**:

```
Excess = 0.04 - 0.05 = -0.01 (-1%)
Treynor = -0.01 / 1.20 = -0.00833

Result: Treynor Ratio = -0.00833
```

**Interpretation**: Negative Treynor Ratio indicates the portfolio underperformed the risk-free rate. Taking market risk resulted in worse outcomes than holding risk-free assets.

## 4. Complexity Analysis

### 4.1 Time Complexity

**All Cases**: $O(1)$ - Constant time

**Operations**:
1. Check if beta == 0: $O(1)$
2. Subtraction: $O(1)$
3. Division: $O(1)$
4. Return: $O(1)$

**Total**: 4 constant-time operations = $O(1)$

### 4.2 Space Complexity

**Space**: $O(1)$ - Constant space

**Memory Usage**:
- Input parameters: 3 × f64 = 24 bytes
- Intermediate result: 1 × f64 = 8 bytes
- No data structures
- No recursion

**Total**: ~32 bytes (negligible)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Function Signature**:
```rust
pub fn treynor_ratio(portfolio_return: f64, risk_free_rate: f64, beta: f64) -> f64
```

**Return Type - f64**:
- Returns `f64::NAN` for beta = 0 case
- Idiomatic for mathematical functions with undefined regions
- Caller must check `.is_nan()` if needed

**NaN Handling**:
```rust
if beta == 0.0 {
    f64::NAN
} else {
    (portfolio_return - risk_free_rate) / beta
}
```

**Key Points**:
- Direct comparison `beta == 0.0` is safe here
- No epsilon comparison needed (exact zero check)
- NaN propagates through subsequent calculations

**Alternative Robust Implementation**:
```rust
pub fn treynor_ratio(portfolio_return: f64, risk_free_rate: f64, beta: f64) -> f64 {
    if beta == 0.0 || !beta.is_finite() {
        f64::NAN
    } else {
        let excess = portfolio_return - risk_free_rate;
        if !excess.is_finite() {
            f64::NAN
        } else {
            excess / beta
        }
    }
}
```

**With Result Type** (Production):
```rust
pub enum TreynorError {
    ZeroBeta,
    InvalidInput,
}

pub fn treynor_ratio(
    portfolio_return: f64, 
    risk_free_rate: f64, 
    beta: f64
) -> Result<f64, TreynorError> {
    if !portfolio_return.is_finite() || !risk_free_rate.is_finite() {
        return Err(TreynorError::InvalidInput);
    }
    if beta == 0.0 {
        return Err(TreynorError::ZeroBeta);
    }
    if !beta.is_finite() {
        return Err(TreynorError::InvalidInput);
    }
    Ok((portfolio_return - risk_free_rate) / beta)
}
```

**Type Safety**:
- All parameters `f64` (no integer conversions)
- Return type `f64` (maintains precision)
- No unsafe code

**Numerical Stability**:
- Subtraction before division (good practice)
- Single division operation (minimizes error)
- No intermediate squaring or complex operations

### 5.2 Edge Cases

**1. Zero Beta**:
```rust
assert!(treynor_ratio(0.10, 0.05, 0.0).is_nan());
// Returns NaN (undefined)
```

**2. Negative Beta**:
```rust
let tr = treynor_ratio(0.08, 0.03, -0.50);
assert_eq!(tr, -0.10);
// Valid but needs careful interpretation
```

**3. Very Small Beta**:
```rust
let tr = treynor_ratio(0.10, 0.05, 0.001);
assert_eq!(tr, 50.0);
// Very large Treynor (beta close to zero)
```

**4. Negative Excess Return**:
```rust
let tr = treynor_ratio(0.03, 0.05, 1.5);
assert_eq!(tr, -0.01333);
// Negative ratio (underperformance)
```

**5. Equal Returns**:
```rust
let tr = treynor_ratio(0.05, 0.05, 1.0);
assert_eq!(tr, 0.0);
// Zero excess return (matching risk-free)
```

**6. NaN/Infinity Propagation**:
```rust
// NaN input
assert!(treynor_ratio(f64::NAN, 0.05, 1.0).is_nan());

// Infinity input
assert!(treynor_ratio(f64::INFINITY, 0.05, 1.0).is_infinite());

// Infinity beta
assert!(treynor_ratio(0.10, 0.05, f64::INFINITY) == 0.0);
```

**7. Very Large Beta**:
```rust
let tr = treynor_ratio(0.15, 0.05, 100.0);
assert_eq!(tr, 0.001);
// Small Treynor (high beta relative to excess return)
```

**8. Floating-Point Precision**:
```rust
// Subtractive cancellation
let tr = treynor_ratio(0.050001, 0.05, 1.0);
// Result may have precision issues
```

## 6. Real-World Applications

### 6.1 Portfolio Management

**1. Portfolio Performance Evaluation**
- Compare multiple portfolio managers
- Adjust for systematic risk differences
- Identify skill vs. luck
- Compensation and incentive design

**Example**:
```rust
struct Portfolio {
    name: String,
    return_pct: f64,
    beta: f64,
}

fn evaluate_managers(portfolios: &[Portfolio], rf: f64) -> Vec<(String, f64)> {
    portfolios
        .iter()
        .map(|p| {
            let tr = treynor_ratio(p.return_pct, rf, p.beta);
            (p.name.clone(), tr)
        })
        .collect()
}

// Usage
let portfolios = vec![
    Portfolio { name: "Growth Fund".to_string(), return_pct: 0.15, beta: 1.5 },
    Portfolio { name: "Value Fund".to_string(), return_pct: 0.11, beta: 0.9 },
    Portfolio { name: "Balanced".to_string(), return_pct: 0.10, beta: 1.0 },
];
let rankings = evaluate_managers(&portfolios, 0.03);
// Compare Treynor Ratios to rank skill
```

**2. Asset Allocation Decisions**
- Determine optimal portfolio weights
- Identify efficient combinations
- Rebalancing triggers

**3. Hedge Fund Analysis**
- Evaluate market-neutral strategies
- Long/short equity performance
- Risk-adjusted alpha generation

**4. Pension Fund Management**
- Trustee reporting
- Manager selection and retention
- Risk budget allocation

### 6.2 Institutional Investment

**1. Mutual Fund Ratings**
- Morningstar and Lipper use risk-adjusted metrics
- Fund comparison within categories
- Performance persistence analysis

**2. Endowment/Foundation Management**
- Long-term performance tracking
- Comparing investment committees
- Strategic asset allocation reviews

**3. Insurance Company Investment**
- Matching liabilities with assets
- Systematic risk exposure limits
- Regulatory capital requirements

**4. Wealth Management**
- High-net-worth client reporting
- Customized portfolio strategies
- Tax-efficient risk management

### 6.3 Academic Research Applications

**1. Market Efficiency Studies**
- Testing CAPM predictions
- Anomaly investigation (value, momentum, size)
- Factor model evaluation

**2. Manager Skill Attribution**
- Separating alpha from beta
- Luck vs. skill debates
- Persistence of performance

**3. Behavioral Finance**
- Risk perception vs. systematic risk
- Investor preferences for risk-adjusted returns
- Herding and momentum effects

### 6.4 Comparison with Other Metrics

**Treynor vs. Sharpe Ratio**:

| Aspect | Treynor Ratio | Sharpe Ratio |
|--------|---------------|--------------|
| **Risk Measure** | Beta (systematic) | Std Dev (total) |
| **Best For** | Diversified portfolios | Any portfolio/asset |
| **Focus** | Market risk only | All volatility |
| **Use Case** | Part of diversified portfolio | Standalone evaluation |
| **Interpretation** | Reward per market risk | Reward per total risk |

**When to Use Treynor**:
- Portfolio is well-diversified
- Part of larger portfolio
- Comparing sub-portfolios
- Institutional context

**When to Use Sharpe**:
- Individual investments
- Concentrated positions
- Total portfolio evaluation
- Retail investor context

**Treynor vs. Jensen's Alpha**:

**Jensen's Alpha**:
$$\alpha = R_p - [R_f + \beta_p(R_m - R_f)]$$

- Absolute excess return
- Treynor: Relative (ratio) measure
- Both use beta as risk measure
- Alpha useful for attribution

**Treynor vs. Information Ratio**:

**Information Ratio**:
$$\text{IR} = \frac{R_p - R_b}{\sigma_{p-b}}$$

Where $R_b$ = Benchmark return, $\sigma_{p-b}$ = Tracking error

- IR: Active management skill
- Treynor: Total portfolio efficiency
- Different denominators and purposes

**Treynor vs. Sortino Ratio**:

**Sortino Ratio**:
$$\text{Sortino} = \frac{R_p - R_f}{\sigma_{\text{downside}}}$$

- Sortino: Downside risk only
- Treynor: Systematic risk
- Different risk philosophies

### 6.5 Practical Considerations

**Limitations**:

1. **Beta Estimation**: 
   - Historical beta may not predict future beta
   - Different lookback periods yield different betas
   - Beta instability in small caps/illiquid assets

2. **Diversification Assumption**:
   - Assumes unsystematic risk is diversified away
   - Not suitable for concentrated portfolios
   - Misleading for individual securities

3. **Market Benchmark**:
   - Treynor depends on chosen market index
   - Global vs. local market debate
   - Factor exposures beyond market beta

4. **Negative Beta Interpretation**:
   - Ranking breaks down for $\beta < 0$
   - Hedge assets need different evaluation
   - Sign of ratio can mislead

5. **Non-Normal Returns**:
   - Assumes returns are normally distributed
   - Fat tails and skewness not captured
   - Crisis periods may distort

**Best Practices**:

1. **Use with Other Metrics**: Combine with Sharpe, Alpha, Information Ratio
2. **Consistent Time Periods**: Compare ratios over same period
3. **Appropriate Benchmark**: Match to investment universe
4. **Consider Qualitative Factors**: Strategy, team, process
5. **Regular Updates**: Recalculate with rolling windows

## 7. References

### Foundational Papers

1. **Treynor, J. L. (1965)**. "How to Rate Management of Investment Funds". *Harvard Business Review*, 43(1), 63-75.
   - Original introduction of the Treynor Ratio

2. **Sharpe, W. F. (1966)**. "Mutual Fund Performance". *Journal of Business*, 39(1), 119-138.
   - Introduces Sharpe Ratio, comparison to Treynor

3. **Jensen, M. C. (1968)**. "The Performance of Mutual Funds in the Period 1945-1964". *Journal of Finance*, 23(2), 389-416.
   - Jensen's Alpha and risk-adjusted performance

### Modern Portfolio Theory

4. **Sharpe, W. F. (1964)**. "Capital Asset Prices: A Theory of Market Equilibrium under Conditions of Risk". *Journal of Finance*, 19(3), 425-442.
   - CAPM foundation for beta

5. **Lintner, J. (1965)**. "The Valuation of Risk Assets and the Selection of Risky Investments in Stock Portfolios and Capital Budgets". *Review of Economics and Statistics*, 47(1), 13-37.
   - Independent development of CAPM

### Textbooks

6. **Bodie, Z., Kane, A., & Marcus, A. J. (2021)**. *Investments* (12th ed.). McGraw-Hill.
   - Chapter 24: Portfolio Performance Evaluation

7. **Reilly, F. K., & Brown, K. C. (2019)**. *Investment Analysis and Portfolio Management* (11th ed.). Cengage.
   - Comprehensive coverage of performance metrics

8. **Elton, E. J., Gruber, M. J., Brown, S. J., & Goetzmann, W. N. (2014)**. *Modern Portfolio Theory and Investment Analysis* (9th ed.). Wiley.
   - Theoretical foundations

### Empirical Studies

9. **Grinblatt, M., & Titman, S. (1989)**. "Portfolio Performance Evaluation: Old Issues and New Insights". *Review of Financial Studies*, 2(3), 393-421.
   - Critical analysis of performance measures

10. **Jobson, J. D., & Korkie, B. M. (1981)**. "Performance Hypothesis Testing with the Sharpe and Treynor Measures". *Journal of Finance*, 36(4), 889-908.
    - Statistical properties of ratios

### Practice and Standards

11. **CFA Institute**: Level III Curriculum - Portfolio Management
    - Risk-adjusted performance measurement

12. **GIPS (Global Investment Performance Standards)**
    - Performance presentation standards
    - Risk-adjusted return guidelines

### Online Resources

13. **Investopedia**: [Treynor Ratio](https://www.investopedia.com/terms/t/treynorratio.asp)

14. **Corporate Finance Institute**: [Treynor Ratio](https://corporatefinanceinstitute.com/resources/portfolio-management/treynor-ratio/)

15. **CFA Institute Research Foundation**: Publications on performance measurement

### Software and Tools

16. **Bloomberg Terminal**: Portfolio analytics and Treynor calculations
17. **Morningstar Direct**: Fund analysis and risk-adjusted metrics
18. **FactSet**: Portfolio attribution and performance analytics
19. **Python**: `pyfolio` library for portfolio analytics
