# Financial Algorithms

## Overview

This directory contains comprehensive documentation for the financial algorithms implemented in TheAlgorithms/Rust. These algorithms form the foundation of financial analysis, investment decision-making, and portfolio management across corporate finance, investment banking, and asset management.

The financial module covers seven essential algorithms spanning:
- **Time Value of Money** (Present Value, Compound Interest, NPV)
- **Investment Analysis** (NPV Sensitivity, Payback Period)
- **Financial Ratios** (ROI, D/E, Gross Margin, EPS)
- **Portfolio Performance** (Treynor Ratio)

## Algorithms

### 1. [Net Present Value (NPV)](npv.md)

**Category**: Time Value of Money, Capital Budgeting

**Formula**: $\text{NPV} = \sum_{t=0}^{n} \frac{CF_t}{(1 + r)^t}$

**Purpose**: Calculates the present value of a series of future cash flows, fundamental for investment decisions.

**Use Cases**:
- Capital budgeting decisions
- Project evaluation and ranking
- Infrastructure investment analysis
- Cloud migration ROI

**Complexity**: $O(n)$ time, $O(1)$ space

**Key Insight**: Accounts for time value of money; positive NPV creates value.

---

### 2. [NPV Sensitivity Analysis](npv_sensitivity.md)

**Category**: Risk Analysis, Scenario Planning

**Formula**: Computes NPV across multiple discount rates: $[\text{NPV}(r_1), \text{NPV}(r_2), \ldots, \text{NPV}(r_m)]$

**Purpose**: Evaluates how sensitive an investment's value is to changes in discount rates.

**Use Cases**:
- Interest rate risk assessment
- Break-even rate identification (IRR)
- What-if scenario analysis
- Risk-adjusted decision making

**Complexity**: $O(m \times n)$ time where $m$ = number of rates, $n$ = cash flows

**Key Insight**: Shows investment robustness to cost of capital changes.

---

### 3. [Present Value (PV)](present_value.md)

**Category**: Time Value of Money, Valuation

**Formula**: $\text{PV} = \sum_{t=0}^{n} \frac{CF_t}{(1 + r)^t}$

**Purpose**: Determines current value of future cash flows; mathematically identical to NPV but conceptually different application.

**Use Cases**:
- Bond valuation
- Annuity and pension calculations
- Structured settlement valuation
- Lease vs. buy analysis

**Complexity**: $O(n)$ time, $O(1)$ space

**Key Insight**: A dollar today is worth more than a dollar tomorrow.

**Special Feature**: Error handling with `Result<f64, PresentValueError>` for invalid inputs.

---

### 4. [Compound Interest](compound_interest.md)

**Category**: Time Value of Money, Investment Growth

**Formula**: $A = P \left(1 + \frac{r}{n}\right)^{nt}$

**Purpose**: Calculates future value of an investment with periodic compounding.

**Use Cases**:
- Savings account projections
- Retirement planning
- Loan amortization
- Investment growth modeling

**Complexity**: $O(\log(nt))$ time (exponentiation), $O(1)$ space

**Key Parameters**:
- $P$ = Principal
- $r$ = Annual interest rate
- $n$ = Compounding frequency per year
- $t$ = Time in years

**Key Insight**: "Interest on interest" creates exponential growth; compounding frequency matters.

---

### 5. [Payback Period](payback.md)

**Category**: Capital Budgeting, Liquidity Analysis

**Formula**: $T = \min\{t : \sum_{i=0}^{t} CF_i \geq 0\}$

**Purpose**: Measures time required to recover initial investment.

**Use Cases**:
- Quick investment screening
- Liquidity-focused decisions
- High-uncertainty projects
- Small business capital decisions

**Complexity**: 
- Best: $O(1)$ (immediate payback)
- Average/Worst: $O(n)$

**Returns**: `Option<usize>` - `Some(period)` or `None` if never pays back

**Key Insight**: Simple but ignores time value of money and post-payback cash flows.

---

### 6. [Financial Ratios](finance_ratios.md)

**Category**: Financial Analysis, Performance Metrics

**Purpose**: Calculates four fundamental financial ratios for company and investment analysis.

#### 6.1 Return on Investment (ROI)

**Formula**: $\text{ROI} = \frac{\text{Gain} - \text{Cost}}{\text{Cost}}$

**Measures**: Profitability relative to investment

**Interpretation**: 
- ROI > 0: Profitable
- ROI = 0: Break-even
- ROI < 0: Loss

---

#### 6.2 Debt-to-Equity Ratio

**Formula**: $\text{D/E} = \frac{\text{Total Debt}}{\text{Total Equity}}$

**Measures**: Financial leverage

**Interpretation**:
- D/E < 1: Conservative (more equity)
- D/E = 1: Balanced
- D/E > 2: Highly leveraged (risky)

---

#### 6.3 Gross Profit Margin

**Formula**: $\text{GPM} = \frac{\text{Revenue} - \text{COGS}}{\text{Revenue}}$

**Measures**: Profitability after direct costs

**Typical Values**:
- Software: 80-95%
- Retail: 20-40%
- Manufacturing: 25-35%

---

#### 6.4 Earnings Per Share (EPS)

**Formula**: $\text{EPS} = \frac{\text{Net Income} - \text{Preferred Dividends}}{\text{Shares Outstanding}}$

**Measures**: Per-share profitability

**Use**: Foundation for P/E ratio valuation

---

**All Ratios Complexity**: $O(1)$ time, $O(1)$ space

**Key Insight**: Simple arithmetic with profound analytical implications.

---

### 7. [Treynor Ratio](treynor_ratio.md)

**Category**: Portfolio Performance, Risk-Adjusted Returns

**Formula**: $\text{Treynor Ratio} = \frac{R_p - R_f}{\beta_p}$

**Purpose**: Measures excess return per unit of systematic (market) risk.

**Use Cases**:
- Portfolio manager evaluation
- Risk-adjusted performance comparison
- Asset allocation decisions
- Hedge fund analysis

**Parameters**:
- $R_p$ = Portfolio return
- $R_f$ = Risk-free rate
- $\beta_p$ = Portfolio beta (systematic risk)

**Complexity**: $O(1)$ time, $O(1)$ space

**Returns**: `f64` (or `NaN` if beta = 0)

**Key Insight**: Focuses on systematic risk only; appropriate for well-diversified portfolios.

**Comparison**: 
- vs. Sharpe Ratio: Uses beta instead of total volatility
- vs. Jensen's Alpha: Ratio vs. absolute measure

## Algorithm Categories

### Time Value of Money
1. **Present Value** - Current worth of future cash flows
2. **Net Present Value** - Investment project valuation
3. **Compound Interest** - Future value with compounding

### Investment Analysis
1. **NPV Sensitivity** - Risk analysis across scenarios
2. **Payback Period** - Time to recover investment

### Financial Metrics
1. **Financial Ratios** - Performance and leverage metrics
2. **Treynor Ratio** - Risk-adjusted portfolio performance

## Complexity Summary

| Algorithm | Time Complexity | Space Complexity |
|-----------|----------------|------------------|
| NPV | $O(n)$ | $O(1)$ |
| NPV Sensitivity | $O(m \times n)$ | $O(m)$ |
| Present Value | $O(n)$ | $O(1)$ |
| Compound Interest | $O(\log n)$ | $O(1)$ |
| Payback Period | $O(n)$ | $O(1)$ |
| ROI | $O(1)$ | $O(1)$ |
| Debt-to-Equity | $O(1)$ | $O(1)$ |
| Gross Profit Margin | $O(1)$ | $O(1)$ |
| EPS | $O(1)$ | $O(1)$ |
| Treynor Ratio | $O(1)$ | $O(1)$ |

Where:
- $n$ = number of time periods / cash flows
- $m$ = number of discount rates (sensitivity analysis)

## Quick Reference

### When to Use Which Algorithm

**Investment Decision Making**:
- **NPV**: Primary metric for go/no-go decisions
- **Payback**: Quick screening, liquidity concerns
- **NPV Sensitivity**: Uncertainty about discount rate
- **ROI**: Simple return comparison

**Valuation**:
- **Present Value**: Bond pricing, annuity valuation
- **Compound Interest**: Savings growth, loan balances

**Company Analysis**:
- **Gross Profit Margin**: Pricing power, cost structure
- **Debt-to-Equity**: Financial risk, leverage
- **EPS**: Per-share profitability, P/E ratio input

**Portfolio Management**:
- **Treynor Ratio**: Manager performance, systematic risk focus
- **ROI**: Overall return assessment

### Typical Applications by Industry

**Technology/Software**:
- NPV for platform investments
- Payback for tool purchases
- High GPM (80%+) expected
- ROI for development projects

**Finance/Banking**:
- High D/E ratios typical (5-15x)
- Treynor for portfolio evaluation
- EPS as key performance metric
- NPV for branch expansion

**Manufacturing**:
- Payback for equipment (3-7 years)
- Moderate GPM (25-35%)
- NPV for capacity expansion
- D/E typically 1-2x

**Retail**:
- Lower GPM (20-40%)
- Payback for store renovations
- ROI for marketing campaigns
- Inventory turnover critical

## Implementation Notes

### Rust-Specific Features

**Type Safety**:
- All algorithms use `f64` for financial precision
- `Option<T>` for potentially undefined results (payback)
- `Result<T, E>` for validated inputs (present value)

**Performance**:
- Zero-cost abstractions with iterators
- Inline expansion of simple functions
- No heap allocations in hot paths
- Functional programming patterns

**Error Handling**:
```rust
// Present Value with error handling
pub fn present_value(discount_rate: f64, cash_flows: Vec<f64>) 
    -> Result<f64, PresentValueError>

// Payback Period with optional result
pub fn payback(cash_flow: &[f64]) -> Option<usize>

// Treynor Ratio with NaN for undefined
pub fn treynor_ratio(portfolio_return: f64, risk_free_rate: f64, beta: f64) -> f64
```

### Common Patterns

**Borrowed Slices**:
```rust
// Efficient borrowing, no ownership transfer
pub fn npv(cash_flows: &[f64], rate: f64) -> f64
```

**Iterator Chains**:
```rust
cash_flows
    .iter()
    .enumerate()
    .map(|(t, &cf)| cf / (1.0 + rate).powi(t as i32))
    .sum()
```

**Early Returns**:
```rust
for (year, &cf) in cash_flow.iter().enumerate() {
    total += cf;
    if total >= 0.0 {
        return Some(year);  // Early exit
    }
}
None
```

## Testing Considerations

### Edge Cases to Test

1. **Empty Inputs**: `npv(&[], 0.10)` → `0.0`
2. **Zero Values**: `compound_interest(1000.0, 0.0, 4, 2.0)` → `1000.0`
3. **Division by Zero**: `debt_to_equity(100.0, 0.0)` → `inf`
4. **Negative Values**: `return_on_investment(800.0, 1000.0)` → `-0.20`
5. **NaN/Infinity**: `treynor_ratio(0.10, 0.05, 0.0)` → `NaN`
6. **Very Large Numbers**: Test with $10^{12}$ scale values
7. **Precision Issues**: Floating-point comparison tolerances

### Example Test Patterns

```rust
#[test]
fn test_npv_basic() {
    let cash_flows = vec![-1000.0, 300.0, 400.0, 500.0];
    let rate = 0.10;
    let result = npv(&cash_flows, rate);
    assert!((result - (-21.03)).abs() < 0.1);
}

#[test]
fn test_payback_no_recovery() {
    let cash_flows = vec![-1000.0, 100.0, 100.0, 100.0];
    assert_eq!(payback(&cash_flows), None);
}

#[test]
fn test_treynor_zero_beta() {
    assert!(treynor_ratio(0.10, 0.05, 0.0).is_nan());
}
```

## Mathematical Foundations

### Key Principles

**1. Time Value of Money**:
$$PV = \frac{FV}{(1 + r)^t}$$

Foundation for NPV, PV, and compound interest.

**2. Discount Factor**:
$$DF_t = \frac{1}{(1 + r)^t}$$

Converts future values to present equivalents.

**3. Compounding**:
$$A = P(1 + r/n)^{nt}$$

Interest earning interest over time.

**4. Risk-Adjusted Returns**:
$$\text{Excess Return} = R_p - R_f$$

Return above risk-free rate.

**5. Systematic Risk**:
$$\beta = \frac{\text{Cov}(R_p, R_m)}{\text{Var}(R_m)}$$

Market correlation measure.

### Relationships Between Algorithms

```
Present Value ≈ Net Present Value
  ↓
Compound Interest (inverse)
  ↓
Future Value

NPV → IRR (solving NPV = 0)
  ↓
NPV Sensitivity → IRR estimation

Payback Period → Discounted Payback
  ↓
Simple → Time-value adjusted

Financial Ratios → DuPont Analysis
  ↓
ROI/ROE decomposition

Treynor Ratio ↔ Sharpe Ratio
  ↓
Beta vs. Total volatility
```

## Further Reading

### Comprehensive References

1. **Brealey, R. A., Myers, S. C., & Allen, F. (2020)**. *Principles of Corporate Finance* (13th ed.).
   - Chapters 2, 5, 6: Time value, NPV, capital budgeting

2. **Ross, S. A., Westerfield, R. W., & Jaffe, J. (2019)**. *Corporate Finance* (12th ed.).
   - Complete treatment of financial decision-making

3. **Bodie, Z., Kane, A., & Marcus, A. J. (2021)**. *Investments* (12th ed.).
   - Portfolio theory and performance measurement

4. **Damodaran, A. (2012)**. *Investment Valuation* (3rd ed.).
   - Practical valuation techniques

### Online Resources

- **CFA Institute**: Level I, II, III Curricula
- **Investopedia**: Financial calculator and definitions
- **Khan Academy**: Finance and capital markets courses
- **Corporate Finance Institute**: Free financial modeling courses

### Standards

- **FASB GAAP**: U.S. accounting standards
- **IFRS**: International accounting standards
- **GIPS**: Global Investment Performance Standards
- **CFA Institute**: Code of Ethics and Standards

## Related Algorithms

### In This Repository

- **Math Module**: Prime numbers, GCD (for rational calculations)
- **Sorting**: Ranking investments by metrics
- **Data Structures**: Efficient storage of time series data

### Not Yet Implemented

**Potential Extensions**:
1. **Internal Rate of Return (IRR)**: Solve NPV = 0
2. **Modified IRR**: Improved IRR with reinvestment assumptions
3. **Discounted Payback**: Time-value adjusted payback
4. **Profitability Index**: NPV / Initial Investment
5. **Bond Pricing**: Coupon bond valuation
6. **Option Pricing**: Black-Scholes, binomial trees
7. **Portfolio Optimization**: Mean-variance, Markowitz
8. **VaR/CVaR**: Risk measurement
9. **Sharpe Ratio**: Total volatility risk-adjusted returns
10. **Information Ratio**: Active management skill measure

## Contributing

When adding new financial algorithms:

1. **Documentation**: Follow the established template
2. **Mathematical Foundation**: Include formulas and proofs
3. **Real-World Examples**: Provide practical use cases
4. **Edge Cases**: Test boundary conditions thoroughly
5. **References**: Cite academic and practitioner sources
6. **Complexity Analysis**: Document time and space complexity
7. **Rust Idioms**: Use iterators, proper error handling, type safety

## License

All algorithms and documentation in this repository are available under the MIT License. See the main repository [LICENSE](../../LICENSE) file for details.

---

*TheAlgorithms/Rust - Educational implementations of fundamental algorithms*
