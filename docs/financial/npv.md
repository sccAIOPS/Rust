# Net Present Value (NPV)

## 1. Overview

Net Present Value (NPV) is a fundamental financial metric used in capital budgeting and investment planning to analyze the profitability of projected investments or projects. NPV calculates the present value of a series of future cash flows by discounting them to today's dollars using a specified discount rate.

The concept was formalized in the 20th century as part of modern financial theory, though the underlying principle of time value of money dates back centuries. NPV is considered one of the most reliable measures for investment decision-making because it accounts for the time value of money and provides an absolute measure of value creation.

**Key Principle**: A dollar today is worth more than a dollar tomorrow due to its earning potential.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a series of cash flows $CF_0, CF_1, CF_2, \ldots, CF_n$ occurring at times $t = 0, 1, 2, \ldots, n$ and a discount rate $r$, calculate the present value of the entire cash flow series.

**Formal Definition**: 
$$\text{NPV} = \sum_{t=0}^{n} \frac{CF_t}{(1 + r)^t}$$

Where:
- $CF_t$ = Cash flow at time period $t$
- $r$ = Discount rate (required rate of return)
- $t$ = Time period
- $n$ = Total number of periods

### 2.2 Mathematical Model

**Input Specifications**:
- `cash_flows`: Vector of real numbers $\mathbb{R}^{n+1}$ representing cash flows for each period
  - $CF_0$ is typically negative (initial investment)
  - Subsequent values can be positive (inflows) or negative (outflows)
- `rate`: Real number $r \in \mathbb{R}^+$, typically $0 < r < 1$ (expressed as decimal, e.g., 0.10 for 10%)

**Output Specification**:
- Single real number representing the net present value in the same currency units as the cash flows

**Constraints**:
- $r > -1$ (discount rate must be greater than -100%)
- $n \geq 0$ (at least one cash flow, though typically $n \geq 1$)

**Key Mathematical Properties**:

1. **Linearity**: $\text{NPV}(aCF_1 + bCF_2, r) = a \cdot \text{NPV}(CF_1, r) + b \cdot \text{NPV}(CF_2, r)$
2. **Monotonicity**: As discount rate increases, NPV decreases (for typical cash flow patterns)
3. **Zero-rate boundary**: When $r = 0$, $\text{NPV} = \sum_{t=0}^{n} CF_t$

### 2.3 Decision Rule

**Investment Decision Criteria**:
- **NPV > 0**: Accept the project (creates value)
- **NPV = 0**: Indifferent (breaks even on required return)
- **NPV < 0**: Reject the project (destroys value)

**Correctness**: The NPV formula is mathematically sound based on the principle of discounting, which is derived from the opportunity cost of capital.

## 3. Algorithm Description

### 3.1 Intuition

NPV works by "bringing back" all future cash flows to the present day using a discount rate. Think of it as answering the question: "What would I need to invest today at rate $r$ to receive these future cash flows?"

Each cash flow is divided by $(1 + r)^t$ where $t$ is the number of periods in the future. This discount factor gets smaller as $t$ increases, meaning distant cash flows are worth less in present value terms.

### 3.2 Pseudocode

```
Algorithm: Net Present Value
Input: cash_flows[0..n], discount_rate r
Output: npv (real number)

1. Initialize npv ← 0
2. For t = 0 to n:
3.     discount_factor ← (1 + r)^t
4.     present_value_t ← cash_flows[t] / discount_factor
5.     npv ← npv + present_value_t
6. Return npv
```

**Optimized Version** (using enumerate):
```
Algorithm: NPV (Functional)
Input: cash_flows[0..n], discount_rate r
Output: npv

1. Map each (index t, value cf) in cash_flows to cf / (1 + r)^t
2. Sum all mapped values
3. Return sum
```

### 3.3 Step-by-Step Example

**Scenario**: Evaluating a project with initial investment and 3 years of returns

**Given**:
- Initial investment: $CF_0 = -1000$ (negative indicates outflow)
- Year 1 return: $CF_1 = 300$
- Year 2 return: $CF_2 = 400$
- Year 3 return: $CF_3 = 500$
- Discount rate: $r = 0.10$ (10%)

**Calculation**:

| Period (t) | Cash Flow | Discount Factor | Present Value | Running NPV |
|------------|-----------|-----------------|---------------|-------------|
| 0 | -1000 | $(1.10)^0 = 1.000$ | -1000.00 | -1000.00 |
| 1 | 300 | $(1.10)^1 = 1.100$ | 272.73 | -727.27 |
| 2 | 400 | $(1.10)^2 = 1.210$ | 330.58 | -396.69 |
| 3 | 500 | $(1.10)^3 = 1.331$ | 375.66 | **-21.03** |

**Result**: NPV = -21.03

**Interpretation**: At a 10% discount rate, this project destroys $21.03 of value and should be rejected.

## 4. Complexity Analysis

### 4.1 Time Complexity

**Best, Average, and Worst Case**: $O(n)$

**Derivation**:
- The algorithm iterates through each cash flow exactly once
- For each iteration:
  - Computing $(1 + r)^t$ using `powi()`: $O(\log t)$ operations
  - Division and addition: $O(1)$ operations
- Total: $\sum_{t=0}^{n} O(\log t) = O(n \log n)$

However, in practice:
- Modern processors optimize power operations
- For typical financial applications, $n$ is small (often $< 50$ periods)
- Effective time complexity is treated as $O(n)$

**Amortized Complexity**: $O(n)$ - each element processed once

### 4.2 Space Complexity

**Auxiliary Space**: $O(1)$

**Explanation**:
- The implementation uses only a constant amount of extra space
- Accumulator variable for sum
- Temporary variables for intermediate calculations
- No additional data structures created

**Total Space**: $O(n)$ for storing input cash flows

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Ownership and Borrowing**:
```rust
pub fn npv(cash_flows: &[f64], rate: f64) -> f64
```
- Takes a borrowed slice `&[f64]` - no ownership transfer
- Immutable borrow sufficient since we only read values
- `rate` copied (f64 implements Copy trait)

**Generic Type Constraints**:
- Uses `f64` for financial precision
- Could be generalized to use generic floating-point types with trait bounds:
  ```rust
  pub fn npv<T: Float>(cash_flows: &[T], rate: T) -> T
  ```

**Iterator Usage**:
```rust
cash_flows
    .iter()
    .enumerate()
    .map(|(t, &cf)| cf / (1.00 + rate).powi(t as i32))
    .sum()
```
- Functional style using iterator chain
- `enumerate()` provides index for time period
- `map()` applies discount formula
- `sum()` aggregates results
- Zero-cost abstraction: compiles to efficient machine code

**Numerical Considerations**:
- Uses `powi()` for integer exponentiation (faster than `powf()`)
- Type cast `t as i32` required for `powi()`
- Floating-point precision: results accurate to ~15 decimal places

### 5.2 Edge Cases

1. **Empty Cash Flows**:
   ```rust
   let cash_flows: Vec<f64> = vec![];
   let npv = npv(&cash_flows, 0.10);
   // Result: 0.0 (sum of empty iterator)
   ```

2. **Single Cash Flow**:
   ```rust
   let cash_flows = vec![1000.0];
   let npv = npv(&cash_flows, 0.10);
   // Result: 1000.0 (no discounting for t=0)
   ```

3. **Zero Discount Rate**:
   ```rust
   let cash_flows = vec![-100.0, 50.0, 50.0, 50.0];
   let npv = npv(&cash_flows, 0.0);
   // Result: 50.0 (simple sum)
   ```

4. **Negative Discount Rate**:
   - Mathematically valid but rare in practice
   - Represents deflation or negative opportunity cost
   - Implementation handles correctly

5. **Very Large/Small Cash Flows**:
   - Risk of floating-point overflow/underflow
   - Consider using normalized values or arbitrary-precision arithmetic for extreme cases

6. **High Discount Rates**:
   ```rust
   let npv = npv(&[-1000.0, 300.0, 400.0, 500.0], 0.50);
   // Distant cash flows have minimal impact
   ```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Infrastructure Investment Decisions**
- **Context**: Cloud migration vs. on-premise infrastructure
- **Example**: 
  - Initial cost: $500K (cloud migration)
  - Annual savings: $150K for 5 years
  - Discount rate: 12% (company's cost of capital)
  - **NPV calculation determines financial viability**

**2. Product Development ROI**
- **Context**: Evaluating new feature development
- **Cash Flows**:
  - Development cost: -$200K (t=0)
  - Increased revenue: +$80K/year for 4 years
  - Maintenance: -$10K/year
- **Decision**: Proceed only if NPV > 0

**3. Technology Stack Upgrades**
- **Context**: Migrating from legacy system
- **Considerations**:
  - Upfront costs (training, migration)
  - Productivity gains (reduced maintenance)
  - Risk reduction (security, compliance)
  - All converted to cash flow equivalents

**4. Open Source vs. Commercial Software**
- **Analysis**:
  - Commercial: High initial cost, lower maintenance
  - Open source: Low initial cost, higher maintenance
  - NPV helps compare total cost of ownership

### 6.2 Industry Applications

**Financial Services**:
- Capital budgeting for new branches
- Equipment purchase decisions
- Merger and acquisition valuations

**Manufacturing**:
- Equipment replacement analysis
- Automation investment decisions
- Capacity expansion projects

**Real Estate**:
- Property investment analysis
- Development project evaluation
- Lease vs. buy decisions

**Energy Sector**:
- Renewable energy project evaluation
- Exploration and drilling decisions
- Plant modernization investments

### 6.3 Related Algorithms

**Variants**:
1. **Adjusted NPV (APV)**: Separates financing effects from operating cash flows
2. **Risk-Adjusted NPV**: Uses probability-weighted cash flows
3. **Modified NPV**: Accounts for reinvestment rate different from discount rate

**Complementary Metrics**:
1. **Internal Rate of Return (IRR)**: Discount rate where NPV = 0
   - Solves: $\sum_{t=0}^{n} \frac{CF_t}{(1 + \text{IRR})^t} = 0$
   - Often used alongside NPV

2. **Payback Period**: Time to recover initial investment
   - Simpler but ignores time value of money beyond payback

3. **Profitability Index (PI)**: Ratio of PV of benefits to PV of costs
   - $\text{PI} = \frac{\text{NPV} + \text{Initial Investment}}{\text{Initial Investment}}$
   - Useful for capital rationing

4. **Modified Internal Rate of Return (MIRR)**: Addresses IRR's reinvestment assumption

**When to Use NPV vs. Alternatives**:
- **Use NPV**: 
  - Comparing mutually exclusive projects
  - Absolute value creation matters
  - Different project scales
- **Use IRR**: 
  - Communicating with stakeholders (percentage easier to understand)
  - Quick comparison screening
- **Use Payback**: 
  - Liquidity concerns paramount
  - Highly uncertain distant cash flows
- **Use PI**: 
  - Capital rationing situations
  - Ranking independent projects

## 7. References

### Academic Literature
1. **Fisher, I. (1930)**. *The Theory of Interest*. Macmillan. 
   - Foundation of present value theory

2. **Brealey, R. A., Myers, S. C., & Allen, F. (2020)**. *Principles of Corporate Finance* (13th ed.). McGraw-Hill Education.
   - Chapter 5: Net Present Value and Other Investment Criteria

3. **Ross, S. A., Westerfield, R. W., & Jaffe, J. (2019)**. *Corporate Finance* (12th ed.). McGraw-Hill Education.
   - Comprehensive treatment of NPV in capital budgeting

### Online Resources
4. **Khan Academy**: [Net Present Value Introduction](https://www.khanacademy.org/economics-finance-domain/core-finance/interest-tutorial/present-value/v/introduction-to-present-value)

5. **Investopedia**: [NPV - Net Present Value](https://www.investopedia.com/terms/n/npv.asp)

### Related Implementations
6. **NumPy Financial**: `numpy.npv()` function
7. **Excel**: `NPV()` function
8. **Python scipy**: Financial functions module

### Standards
9. **CFA Institute**: Level I Curriculum - Corporate Finance section
10. **FASB**: Financial Accounting Standards for present value measurements
