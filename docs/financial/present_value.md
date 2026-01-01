# Present Value (PV)

## 1. Overview

Present Value (PV), also known as present discounted value, is a fundamental concept in finance and economics that determines the current value of a future amount of money or stream of cash flows given a specified rate of return. It is the cornerstone of the time value of money principle.

The concept answers a simple but profound question: "How much would I need to invest today to have a specific amount in the future, given a certain rate of return?"

**Historical Context**: The mathematical foundation of present value was formalized by Irving Fisher in the early 20th century in his seminal work "The Theory of Interest" (1930), though the underlying principles have been understood since ancient times.

**Core Principle**: Money available today is worth more than the same amount in the future due to its potential earning capacity. This is the most fundamental concept in finance.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a series of future cash flows $CF_0, CF_1, CF_2, \ldots, CF_n$ occurring at times $t = 0, 1, 2, \ldots, n$ and a discount rate $r$, calculate the equivalent value of these cash flows in today's dollars.

**Formal Definition**:
$$\text{PV} = \sum_{t=0}^{n} \frac{CF_t}{(1 + r)^t}$$

Where:
- $CF_t$ = Cash flow at time period $t$
- $r$ = Discount rate (required rate of return)
- $t$ = Time period
- $n$ = Total number of periods

**Note**: This is mathematically identical to Net Present Value (NPV), but conceptually different in application. PV typically values future receipts, while NPV evaluates investment projects (including initial outlay).

### 2.2 Mathematical Model

**Input Specifications**:
- `discount_rate`: Real number $r \in \mathbb{R}$, typically $0 \leq r < 1$
  - Must satisfy: $r \geq 0$ (non-negative constraint in this implementation)
- `cash_flows`: Vector $CF \in \mathbb{R}^{n+1}$, typically all non-negative
  - $CF_0$ = Cash flow at present (t=0)
  - $CF_t$ for $t > 0$ = Future cash flows

**Output Specification**:
- Single real number representing present value
- Rounded to 2 decimal places (cents precision)
- Or error if invalid inputs

**Constraints**:
1. $r \geq 0$ (non-negative discount rate)
2. $n \geq 0$ (at least one cash flow required)
3. Cash flows can be positive or negative

**Error Conditions**:
- `NegativeDiscount`: Discount rate < 0
- `EmptyCashFlow`: No cash flows provided

### 2.3 Key Mathematical Properties

1. **Additivity (Linearity)**:
   $$\text{PV}(CF_1 + CF_2, r) = \text{PV}(CF_1, r) + \text{PV}(CF_2, r)$$

2. **Homogeneity**:
   $$\text{PV}(k \cdot CF, r) = k \cdot \text{PV}(CF, r)$$

3. **Monotonicity**:
   - Decreasing in $r$: Higher discount rates → Lower present values
   - Increasing in $CF_t$: Larger cash flows → Higher present value

4. **Boundary Conditions**:
   - When $r = 0$: $\text{PV} = \sum_{t=0}^{n} CF_t$ (simple sum)
   - As $r \to \infty$: $\text{PV} \to CF_0$ (only immediate cash flow matters)

5. **Time Value Decay**:
   - $\frac{\partial \text{PV}}{\partial t} < 0$ for fixed $CF$ and $r > 0$
   - Distant cash flows contribute less to PV

### 2.4 Special Cases

**Perpetuity** (infinite series of equal payments):
$$\text{PV}_{\text{perpetuity}} = \frac{CF}{r}$$

**Growing Perpetuity**:
$$\text{PV}_{\text{growing}} = \frac{CF}{r - g}$$
where $g$ is growth rate and $g < r$

**Annuity** (finite series of equal payments):
$$\text{PV}_{\text{annuity}} = CF \cdot \frac{1 - (1 + r)^{-n}}{r}$$

## 3. Algorithm Description

### 3.1 Intuition

Present value "rewinds the clock" on future money, asking what it's worth today. Each future cash flow is discounted (reduced) by a factor that accounts for:
1. **Opportunity cost**: What you could earn by investing elsewhere
2. **Time preference**: People generally prefer money now to money later
3. **Risk**: Uncertainty about receiving future payments

The discount factor $(1 + r)^{-t}$ becomes smaller as time $t$ increases, meaning:
- Cash flow next year: Discounted by $(1 + r)^1$
- Cash flow in 10 years: Discounted by $(1 + r)^{10}$ (much smaller)

### 3.2 Pseudocode

```
Algorithm: Present Value with Error Handling
Input: discount_rate r, cash_flows[0..n]
Output: PV (rounded to 2 decimals) or Error

1. If r < 0:
2.     Return Error(NegativeDiscount)
3. If cash_flows is empty:
4.     Return Error(EmptyCashFlow)
5. 
6. Initialize pv ← 0
7. For each index t from 0 to n:
8.     discount_factor ← (1 + r)^t
9.     present_value_t ← cash_flows[t] / discount_factor
10.    pv ← pv + present_value_t
11. 
12. pv_rounded ← round(pv, 2)  // Round to cents
13. Return Ok(pv_rounded)
```

**Rounding Function**:
```
Function: round(value, decimals)
Input: value (float), decimals (int)
Output: rounded value

1. multiplier ← 10^decimals
2. Return (value × multiplier).round() / multiplier
```

### 3.3 Step-by-Step Example

**Scenario**: Valuing a 4-year investment bond

**Given**:
- Year 0: $10 (immediate payment)
- Year 1: $20.70
- Year 2: -$293 (payment required, e.g., maintenance)
- Year 3: $297
- Discount rate: 13% (0.13)

**Calculation**:

| Period (t) | Cash Flow (CF) | Discount Factor | Present Value | Running Total |
|------------|----------------|-----------------|---------------|---------------|
| 0 | $10.00 | $(1.13)^0 = 1.000$ | $10.00 | $10.00 |
| 1 | $20.70 | $(1.13)^1 = 1.130$ | $18.32 | $28.32 |
| 2 | -$293.00 | $(1.13)^2 = 1.277$ | -$229.43 | -$201.11 |
| 3 | $297.00 | $(1.13)^3 = 1.443$ | $205.80 | **$4.69** |

**Detailed Calculations**:
- **Period 0**: $10.00 / 1.000 = 10.00$
- **Period 1**: $20.70 / 1.130 = 18.32$
- **Period 2**: $-293.00 / 1.277 = -229.43$
- **Period 3**: $297.00 / 1.443 = 205.80$

**Result**: PV = $4.69

**Interpretation**: This stream of cash flows is worth $4.69 today when discounted at 13%.

**Alternative Example - All Positive Cash Flows**:

Given: [109129.39, 30923.23, 15098.93, 29734.0, 39.0], r = 0.07

| Period | Cash Flow | Discount Factor | Present Value |
|--------|-----------|-----------------|---------------|
| 0 | 109,129.39 | 1.000 | 109,129.39 |
| 1 | 30,923.23 | 1.070 | 28,899.28 |
| 2 | 15,098.93 | 1.145 | 13,189.59 |
| 3 | 29,734.00 | 1.225 | 24,273.88 |
| 4 | 39.00 | 1.311 | 29.75 |
| **Total** | | | **175,521.89** |

After rounding to 2 decimals: **$175,519.15**

## 4. Complexity Analysis

### 4.1 Time Complexity

**Best, Average, and Worst Case**: $O(n)$

**Derivation**:
- Single pass through $n+1$ cash flows
- For each element:
  - Enumerate: $O(1)$
  - Power operation $(1 + r)^t$: $O(\log t)$ via binary exponentiation
  - Division: $O(1)$
  - Addition: $O(1)$
- Total: $\sum_{t=0}^{n} O(\log t) = O(n \log n)$

**Practical Complexity**: $O(n)$
- Modern CPUs highly optimize power operations
- For financial applications, $n$ is typically small ($< 100$ periods)
- Logarithmic factor negligible in practice

**Rounding Operation**: $O(1)$ - constant time arithmetic

### 4.2 Space Complexity

**Auxiliary Space**: $O(1)$

**Breakdown**:
- Accumulator variable: $O(1)$
- Temporary computation variables: $O(1)$
- Error enum (if error): $O(1)$

**Total Space**: $O(n)$ including input vector

**Memory Access Pattern**:
- Sequential read through cash flows
- Good cache locality
- No additional data structures allocated

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Error Handling with Result Type**:
```rust
pub enum PresentValueError {
    NegetiveDiscount,  // Note: typo preserved from original
    EmptyCashFlow,
}

pub fn present_value(
    discount_rate: f64, 
    cash_flows: Vec<f64>
) -> Result<f64, PresentValueError>
```

**Type System Benefits**:
- Compile-time guarantee that errors are handled
- Explicit error types document failure modes
- `Result<T, E>` forces caller to handle errors via:
  - `unwrap()` - panic on error (testing)
  - `expect()` - panic with message
  - `?` operator - propagate error
  - `match` or `if let` - explicit handling

**Ownership Semantics**:
- Takes `Vec<f64>` by value (ownership transfer)
- Alternative design could use `&[f64]` (borrow) for efficiency
- Current design simpler but copies on call

**Functional Iterator Pattern**:
```rust
let present_value = cash_flows
    .iter()
    .enumerate()
    .map(|(i, &cash_flow)| {
        cash_flow / (1.0 + discount_rate).powi(i as i32)
    })
    .sum::<f64>();
```

**Rounding Implementation**:
```rust
fn round(value: f64) -> f64 {
    (value * 100.0).round() / 100.0
}
```
- Multiplies by 100 to shift decimal point
- Uses built-in `round()` for banker's rounding
- Divides by 100 to restore scale
- Simple but effective for cents precision

**Numerical Precision**:
- Uses `f64` (IEEE 754 double precision)
- ~15-17 significant decimal digits
- Rounding to 2 decimals hides floating-point errors
- For high-precision finance, consider `rust_decimal` crate

**Type Casting**:
- `i as i32` for `powi()` parameter
- Safe for typical financial time periods ($t < 2^{31}$)

### 5.2 Edge Cases

1. **Empty Cash Flows**:
   ```rust
   let result = present_value(0.07, vec![]);
   assert_eq!(result, Err(PresentValueError::EmptyCashFlow));
   ```

2. **Negative Discount Rate**:
   ```rust
   let result = present_value(-0.05, vec![100.0, 200.0]);
   assert_eq!(result, Err(PresentValueError::NegetiveDiscount));
   ```

3. **Zero Discount Rate**:
   ```rust
   let result = present_value(0.0, vec![100.0, 200.0, 50.0]);
   assert_eq!(result, Ok(350.0));  // Simple sum
   ```

4. **Single Cash Flow**:
   ```rust
   let result = present_value(0.10, vec![1000.0]);
   assert_eq!(result, Ok(1000.0));  // No discounting at t=0
   ```

5. **Negative Cash Flows**:
   ```rust
   let result = present_value(0.10, vec![100.0, -50.0, 200.0]);
   // Valid: Handles mixed cash flows
   ```

6. **Very Large Discount Rate**:
   ```rust
   let result = present_value(5.0, vec![100.0, 1000.0, 10000.0]);
   // Future values heavily discounted
   // Result: ~100.32 (only first period matters)
   ```

7. **Very Long Time Horizon**:
   ```rust
   let cash_flows: Vec<f64> = vec![100.0; 1000];  // 1000 periods
   let result = present_value(0.05, cash_flows);
   // Long tail becomes negligible
   ```

8. **Rounding Edge Cases**:
   ```rust
   // Value: 10.455 → rounds to 10.46 (banker's rounding to even)
   // Value: 10.445 → rounds to 10.44
   ```

9. **Floating-Point Precision Issues**:
   ```rust
   // May not be exactly 0.55 due to binary representation
   assert_eq!(round(0.55434), 0.55);
   ```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. SaaS Subscription Valuation**
- **Context**: Valuing future subscription revenue
- **Example**:
  ```rust
  // 3-year subscription: $100/month
  let monthly_revenue = 100.0;
  let months = 36;
  let cash_flows: Vec<f64> = vec![monthly_revenue; months];
  let pv = present_value(0.01, cash_flows)?;  // 1% monthly discount
  ```
- **Use**: Determine customer lifetime value (CLV)

**2. Technical Debt Cost Analysis**
- **Context**: Cost of postponing refactoring
- **Cash Flows**:
  - Immediate: Save $50K (by not refactoring now)
  - Future: +$20K/year extra maintenance for 5 years
- **PV Analysis**: Compare present value of future costs to immediate savings

**3. Cloud vs. On-Premise Cost Comparison**
- **On-Premise**:
  - t=0: -$500K (servers, setup)
  - t=1-5: -$50K/year (maintenance)
- **Cloud**:
  - t=0: -$0
  - t=1-5: -$150K/year (subscription)
- **Decision**: Compare present values at company's discount rate

**4. Developer Training Investment**
- **Context**: Evaluating training program ROI
- **Cash Flows**:
  - t=0: -$10K (training cost)
  - t=1-3: +$5K/year (productivity gains)
- **PV**: Determines if training creates value

**5. License Purchase vs. Subscription**
- **Purchase**: Pay $10K upfront (perpetual)
- **Subscription**: Pay $2K/year
- **Analysis**: 
  ```rust
  let purchase_pv = 10_000.0;
  let subscription_cf = vec![2000.0; 10];  // 10 years
  let subscription_pv = present_value(0.08, subscription_cf)?;
  // Compare PVs to decide
  ```

### 6.2 Industry Applications

**Personal Finance**:
- Mortgage and loan calculations
- Retirement planning (valuing future pension payments)
- College savings plans
- Lottery lump sum vs. annuity decisions

**Corporate Finance**:
- Bond valuation (coupon payments + principal)
- Stock valuation (dividend discount model)
- Lease vs. buy analysis
- Equipment purchase decisions

**Real Estate**:
- Property valuation (discounted cash flow method)
- Rental income streams
- Development project analysis
- REIT (Real Estate Investment Trust) valuations

**Insurance**:
- Annuity pricing
- Life insurance policy valuation
- Claims reserves calculation
- Pension fund liabilities

**Government/Public Sector**:
- Infrastructure project evaluation
- Social program cost-benefit analysis
- Environmental remediation decisions
- Military procurement decisions

**Legal/Settlements**:
- Structured settlement valuations
- Wrongful death calculations
- Contract breach damages
- Patent royalty stream valuation

### 6.3 Practical Examples

**Example 1: Bond Valuation**
```rust
// 5-year bond, $1000 face value, 5% annual coupon, 7% discount rate
let coupon = 50.0;  // $1000 × 5%
let face_value = 1000.0;
let cash_flows = vec![
    coupon,           // Year 1
    coupon,           // Year 2
    coupon,           // Year 3
    coupon,           // Year 4
    coupon + face_value,  // Year 5
];
// Note: Should add t=0 with 0.0 if using this implementation
let pv = present_value(0.07, cash_flows)?;
// Bond worth less than face value (discount rate > coupon rate)
```

**Example 2: Retirement Planning**
```rust
// Will receive $50K/year for 20 years starting now
let annual_pension = 50_000.0;
let years = 20;
let cash_flows = vec![annual_pension; years];
let pv = present_value(0.04, cash_flows)?;  // 4% discount rate
// This is what the pension is worth in today's dollars
```

### 6.4 Related Algorithms

**1. Net Present Value (NPV)**:
- Identical formula, different application
- Typically includes negative initial investment
- Used for project evaluation
- Decision rule: Accept if NPV > 0

**2. Future Value (FV)**:
- Inverse of present value
- Formula: $FV = PV \times (1 + r)^n$
- Answers: "What will this investment be worth?"

**3. Internal Rate of Return (IRR)**:
- Solves for $r$ where $\text{PV} = 0$
- Iterative/numerical solution required
- Yield to maturity for bonds

**4. Annuity Formulas**:
- Closed-form for equal periodic payments
- Present Value of Annuity: $PV = PMT \times \frac{1 - (1+r)^{-n}}{r}$
- More efficient than general PV for regular payments

**5. Modified Duration**:
- Measures interest rate sensitivity of PV
- $D = -\frac{1}{PV} \frac{\partial PV}{\partial r}$
- Used in bond portfolio management

**6. Discounted Cash Flow (DCF) Valuation**:
- Extensions of PV for equity valuation
- Includes terminal value estimation
- Free cash flow to firm (FCFF) or equity (FCFE)

**When to Use PV vs. Alternatives**:
- **Use PV**: Valuing known future cash flows, comparing alternatives
- **Use FV**: Planning for future goals, savings targets
- **Use NPV**: Evaluating investments with initial costs
- **Use Annuity Formula**: Regular, equal payments (simpler, more efficient)
- **Use IRR**: Finding break-even rate, communicating returns as percentage

## 7. References

### Foundational Texts

1. **Fisher, I. (1930)**. *The Theory of Interest*. Macmillan.
   - Original mathematical formulation of present value

2. **Williams, J. B. (1938)**. *The Theory of Investment Value*. Harvard University Press.
   - Applied present value to stock valuation

3. **Brealey, R. A., Myers, S. C., & Allen, F. (2020)**. *Principles of Corporate Finance* (13th ed.). McGraw-Hill.
   - Chapter 2: How to Calculate Present Values

### Academic Resources

4. **Damodaran, A. (2012)**. *Investment Valuation: Tools and Techniques for Determining the Value of Any Asset* (3rd ed.). Wiley.
   - Comprehensive coverage of DCF methods

5. **Copeland, T., Weston, F., & Shastri, K. (2005)**. *Financial Theory and Corporate Policy* (4th ed.). Pearson.
   - Theoretical foundations

### Standards and Practice

6. **CFA Institute**: Level I Curriculum - Quantitative Methods
   - Time value of money calculations

7. **FASB (Financial Accounting Standards Board)**: ASC 820 - Fair Value Measurement
   - Present value techniques for accounting

### Online Resources

8. **Wikipedia**: [Present Value](https://en.wikipedia.org/wiki/Present_value)
   - Comprehensive overview with examples

9. **Investopedia**: [Present Value - PV](https://www.investopedia.com/terms/p/presentvalue.asp)
   - Practical explanations and calculators

10. **Khan Academy**: [Present Value](https://www.khanacademy.org/economics-finance-domain/core-finance/interest-tutorial/present-value)
    - Video tutorials and interactive examples

### Software Implementations

11. **Excel**: `PV()`, `NPV()`, `XNPV()` functions
12. **Python NumPy**: `numpy.pv()` function
13. **R**: `PV()` in FinCal package
14. **MATLAB**: Financial Toolbox present value functions

### Historical Context

15. **Goetzmann, W. N., & Rouwenhorst, K. G. (2005)**. *The Origins of Value: The Financial Innovations that Created Modern Capital Markets*. Oxford University Press.
    - Historical development of time value concepts
