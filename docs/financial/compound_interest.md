# Compound Interest

## 1. Overview

Compound Interest is one of the most powerful concepts in finance, representing the process where interest earned on an investment is reinvested to generate additional earnings over time. Often described as "interest on interest," compound interest causes wealth to grow exponentially rather than linearly.

Albert Einstein allegedly called compound interest "the eighth wonder of the world," stating "He who understands it, earns it; he who doesn't, pays it." While this quote's attribution is debatable, the sentiment captures the transformative power of compounding.

**Key Insight**: The frequency of compounding (annual, quarterly, monthly, daily) significantly impacts the final amount, even with the same nominal interest rate.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a principal amount $P$, an annual interest rate $r$, the number of times interest is compounded per year $n$, and the investment time period in years $t$, calculate the future value $A$ of the investment.

**Standard Compound Interest Formula**:
$$A = P \left(1 + \frac{r}{n}\right)^{nt}$$

Where:
- $A$ = Final amount (principal + interest)
- $P$ = Principal amount (initial investment)
- $r$ = Annual nominal interest rate (as decimal, e.g., 0.05 for 5%)
- $n$ = Number of times interest is compounded per year
- $t$ = Time period in years

### 2.2 Mathematical Model

**Input Specifications**:
- `principal`: $P \in \mathbb{R}^+$ (positive real number)
- `rate`: $r \in \mathbb{R}^+$ (typically $0 < r < 1$)
- `comp_per_year`: $n \in \mathbb{N}^+$ (positive integer)
  - $n = 1$: Annual compounding
  - $n = 2$: Semi-annual
  - $n = 4$: Quarterly
  - $n = 12$: Monthly
  - $n = 365$: Daily
- `years`: $t \in \mathbb{R}^+$ (positive real number, supports fractional years)

**Output Specification**:
- Single positive real number representing final amount

**Constraints**:
- $P > 0$ (positive principal)
- $r \geq 0$ (non-negative interest rate)
- $n \geq 1$ (at least annual compounding)
- $t \geq 0$ (non-negative time period)

### 2.3 Key Mathematical Properties

**1. Growth Factor per Period**:
$$\text{Growth Factor} = 1 + \frac{r}{n}$$

**2. Total Number of Compounding Periods**:
$$\text{Total Periods} = n \times t$$

**3. Compound Interest Earned**:
$$I = A - P = P\left[\left(1 + \frac{r}{n}\right)^{nt} - 1\right]$$

**4. Effective Annual Rate (EAR)**:
The actual annual rate accounting for compounding:
$$\text{EAR} = \left(1 + \frac{r}{n}\right)^n - 1$$

Example: 12% compounded monthly
$$\text{EAR} = \left(1 + \frac{0.12}{12}\right)^{12} - 1 = 1.01^{12} - 1 \approx 0.1268 = 12.68\%$$

**5. Continuous Compounding Limit**:
As $n \to \infty$:
$$A = \lim_{n \to \infty} P \left(1 + \frac{r}{n}\right)^{nt} = Pe^{rt}$$

This is the theoretical maximum from compounding.

**6. Doubling Time (Rule of 72)**:
Approximate time to double investment:
$$t_{\text{double}} \approx \frac{72}{r \times 100}$$

Example: At 8% annual rate, doubling time $\approx 72/8 = 9$ years

### 2.4 Compounding Frequency Comparison

For $P = \$1000$, $r = 12\%$, $t = 1$ year:

| Frequency | $n$ | Formula | Amount |
|-----------|-----|---------|---------|
| Annual | 1 | $1000(1.12)^1$ | $1,120.00 |
| Semi-annual | 2 | $1000(1.06)^2$ | $1,123.60 |
| Quarterly | 4 | $1000(1.03)^4$ | $1,125.51 |
| Monthly | 12 | $1000(1.01)^{12}$ | $1,126.83 |
| Daily | 365 | $1000(1.000329)^{365}$ | $1,127.47 |
| Continuous | $\infty$ | $1000e^{0.12}$ | $1,127.50 |

**Observation**: Diminishing returns from increasing compounding frequency.

## 3. Algorithm Description

### 3.1 Intuition

Compound interest works by repeatedly applying the same growth rate to an ever-increasing base:

**Year 1**: Earn interest on principal
**Year 2**: Earn interest on (principal + year 1 interest)
**Year 3**: Earn interest on (principal + year 1 interest + year 2 interest)
**...and so on**

The more frequently interest compounds within a year, the more "interest on interest" you earn, though the effect diminishes as frequency increases.

**Visual Mental Model**:
```
$1000 at 10% annual, compounded annually:
Year 0: $1000
Year 1: $1000 × 1.10 = $1100
Year 2: $1100 × 1.10 = $1210
Year 3: $1210 × 1.10 = $1331

vs. Simple Interest (no compounding):
Year 1: $1000 + $100 = $1100
Year 2: $1000 + $200 = $1200
Year 3: $1000 + $300 = $1300
```

### 3.2 Pseudocode

```
Algorithm: Compound Interest
Input: principal P, annual_rate r, comp_per_year n, years t
Output: final_amount A

1. rate_per_period ← r / n
2. growth_factor ← 1 + rate_per_period
3. total_periods ← n × t
4. final_amount ← P × (growth_factor)^total_periods
5. Return final_amount
```

**Alternative Formulation** (matching implementation):
```
Algorithm: Compound Interest (Implementation Style)
Input: principal, rate, comp_per_year, years
Output: final_amount

1. base ← 1.0 + (rate / comp_per_year)
2. exponent ← comp_per_year × years
3. multiplier ← base^exponent
4. final_amount ← principal × multiplier
5. Return final_amount
```

### 3.3 Step-by-Step Example

**Scenario**: Savings account with quarterly compounding

**Given**:
- Principal: $P = \$1,000$
- Annual interest rate: $r = 5\%$ = 0.05
- Compounding frequency: $n = 4$ (quarterly)
- Time period: $t = 2$ years

**Calculation**:

**Step 1**: Calculate rate per period
$$\text{Rate per period} = \frac{0.05}{4} = 0.0125 \text{ (1.25\% per quarter)}$$

**Step 2**: Calculate growth factor
$$\text{Growth factor} = 1 + 0.0125 = 1.0125$$

**Step 3**: Calculate total periods
$$\text{Total periods} = 4 \times 2 = 8 \text{ quarters}$$

**Step 4**: Apply compound formula
$$A = 1000 \times (1.0125)^8$$

**Detailed Period-by-Period Growth**:

| Period | Beginning Balance | Interest Earned | Ending Balance |
|--------|-------------------|-----------------|----------------|
| Q1 (0-3m) | $1,000.00 | $12.50 | $1,012.50 |
| Q2 (3-6m) | $1,012.50 | $12.66 | $1,025.16 |
| Q3 (6-9m) | $1,025.16 | $12.81 | $1,037.97 |
| Q4 (9-12m) | $1,037.97 | $12.97 | $1,050.95 |
| Q5 (12-15m) | $1,050.95 | $13.14 | $1,064.08 |
| Q6 (15-18m) | $1,064.08 | $13.30 | $1,077.38 |
| Q7 (18-21m) | $1,077.38 | $13.47 | $1,090.85 |
| Q8 (21-24m) | $1,090.85 | $13.64 | **$1,104.49** |

**Mathematical Calculation**:
$$A = 1000 \times (1.0125)^8 = 1000 \times 1.104486 = 1104.486$$

**Result**: $A \approx \$1,104.49$

**Interest Earned**: $1,104.49 - $1,000.00 = **$104.49**

**Comparison with Simple Interest**:
Simple interest: $1000 \times (1 + 0.05 \times 2) = $1,100.00$
Compound advantage: $1,104.49 - $1,100.00 = **$4.49 extra** (4.5% more interest)

### 3.4 Fractional Years Example

**Given**: $P = $5,000$, $r = 6\%$, $n = 12$ (monthly), $t = 1.5$ years

**Calculation**:
$$A = 5000 \times \left(1 + \frac{0.06}{12}\right)^{12 \times 1.5}$$
$$A = 5000 \times (1.005)^{18}$$
$$A = 5000 \times 1.09393$$
$$A \approx \$5,469.66$$

## 4. Complexity Analysis

### 4.1 Time Complexity

**Best, Average, and Worst Case**: $O(\log(nt))$

**Derivation**:
- Computing $(1 + r/n)^{nt}$ using `powf()` or `powi()`
- Modern implementations use binary exponentiation
- Binary exponentiation: $O(\log k)$ multiplications for $x^k$
- Here $k = nt$, so $O(\log(nt))$

**Practical Considerations**:
- $nt$ is typically small in financial applications
- Most CPUs have hardware floating-point exponentiation
- Effective constant time: $O(1)$ for typical inputs

**Operations Count**:
1. Division: $r / n$ → $O(1)$
2. Addition: $1 + (r/n)$ → $O(1)$
3. Multiplication: $n \times t$ → $O(1)$
4. Exponentiation: $(base)^{exponent}$ → $O(\log(nt))$
5. Multiplication: $P \times result$ → $O(1)$

**Total**: $O(\log(nt))$, effectively $O(1)$ for financial applications

### 4.2 Space Complexity

**Auxiliary Space**: $O(1)$

**Breakdown**:
- Intermediate variables (base, exponent): $O(1)$
- No arrays or complex data structures
- Stack space for function call: $O(1)$

**Total Space**: $O(1)$ - constant space algorithm

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Function Signature**:
```rust
pub fn compound_interest(
    principal: f64, 
    rate: f64, 
    comp_per_year: u32, 
    years: f64
) -> f64
```

**Type Choices**:
- `principal`, `rate`, `years`: `f64` for floating-point precision
- `comp_per_year`: `u32` for non-negative integer compounding frequency
  - Natural semantic fit (can't compound -5 times per year)
  - Saves memory vs. `i32` or `f64`
  - Safe conversion to `f64` via `as f64`

**Floating-Point Power Function**:
```rust
principal * (1.00 + rate / comp_per_year as f64).powf(comp_per_year as f64 * years)
```
- Uses `.powf()` method (floating-point exponentiation)
- Necessary because exponent `comp_per_year * years` may be non-integer
- Alternative: `.powi()` requires integer exponent (less flexible)

**Type Casting**:
- `comp_per_year as f64`: Safe conversion from `u32` to `f64`
- All `u32` values exactly representable in `f64` (up to $2^{53}$)
- No precision loss for reasonable compounding frequencies

**Numerical Considerations**:
- `f64` provides ~15-17 decimal digits of precision
- Sufficient for financial calculations at human scales
- For extreme precision, consider `rust_decimal` or `bigdecimal` crates

**Style Notes**:
- `1.00` used instead of `1.0` for consistency (financial context)
- Parentheses clarify order of operations
- Single-expression function body (concise)

### 5.2 Edge Cases

**1. Zero Interest Rate**:
```rust
let result = compound_interest(1000.0, 0.0, 4, 2.0);
// Result: 1000.0 (no growth)
// (1 + 0/4)^(4*2) = 1^8 = 1
```

**2. Zero Time Period**:
```rust
let result = compound_interest(1000.0, 0.05, 4, 0.0);
// Result: 1000.0 (no time for compounding)
// (1.0125)^0 = 1
```

**3. Annual Compounding (n=1)**:
```rust
let result = compound_interest(1000.0, 0.10, 1, 5.0);
// Result: 1610.51
// Simplifies to: 1000 × (1.10)^5
```

**4. Very Frequent Compounding**:
```rust
let result = compound_interest(1000.0, 0.12, 365, 1.0);
// Daily compounding
// Result: ≈ 1127.47
// Approaches continuous: 1000 × e^0.12 ≈ 1127.50
```

**5. Fractional Years**:
```rust
let result = compound_interest(1000.0, 0.08, 12, 0.5);
// 6 months with monthly compounding
// Result: ≈ 1040.81
```

**6. Very Long Time Periods**:
```rust
let result = compound_interest(100.0, 0.05, 1, 100.0);
// Result: ≈ 13,150.13
// Shows exponential growth over century
```

**7. High Interest Rates**:
```rust
let result = compound_interest(1000.0, 0.50, 12, 1.0);
// 50% annual rate
// Result: ≈ 1,646.61
// Much higher than nominal due to compounding
```

**8. Negative Input Handling** (undefined, no validation):
```rust
// Current implementation doesn't validate:
compound_interest(-1000.0, 0.05, 4, 2.0);  // Negative principal
compound_interest(1000.0, -0.05, 4, 2.0);  // Negative rate (decay)
compound_interest(1000.0, 0.05, 4, -2.0);  // Negative time (past value)
// Consider adding validation in production code
```

**9. Overflow/Underflow**:
```rust
// Extremely large values may overflow
let result = compound_interest(1e100, 0.50, 12, 100.0);
// May return f64::INFINITY

// Extremely small values may underflow
let result = compound_interest(1e-200, 0.01, 1, 1.0);
// May lose precision or underflow to 0
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. SaaS Revenue Growth Modeling**
- **Context**: Projecting future revenue with monthly user growth
- **Model**: Each month compounds previous growth
- **Example**:
  ```rust
  let initial_mrr = 10_000.0;  // Monthly Recurring Revenue
  let monthly_growth = 0.05;   // 5% month-over-month
  let years = 3.0;
  let projected_mrr = compound_interest(initial_mrr, monthly_growth * 12.0, 12, years);
  // Note: This adapts the formula; rate parameter is annual equivalent
  ```

**2. Technical Debt Accumulation**
- **Context**: Cost of debt grows if not addressed
- **Metaphor**: Debt "compounds" as codebas grows
- **Model**: Each sprint, debt becomes harder to fix (grows by rate)

**3. Performance Optimization Gains**
- **Context**: Iterative performance improvements compound
- **Example**: Each optimization provides 10% speedup
  ```rust
  // After 5 optimizations at 10% each (not additive!)
  let original_time = 1000.0;  // ms
  let speedup_rate = 0.10;
  let iterations = 5.0;
  let final_time = original_time * (1.0 - speedup_rate).powf(iterations);
  // Result: ≈ 590ms (not 500ms with simple addition)
  ```

**4. User Base Growth Projections**
- **Context**: Viral/network effects cause exponential user growth
- **Model**: Each period, new users bring more users
- **Application**: Capacity planning, infrastructure scaling

**5. License Cost Projections**
- **Context**: Enterprise software licenses with annual increases
- **Example**:
  ```rust
  let initial_cost = 50_000.0;
  let annual_increase = 0.05;  // 5% per year
  let years = 5.0;
  let future_cost = compound_interest(initial_cost, annual_increase, 1, years);
  // Budget planning for 5-year contract
  ```

### 6.2 Financial Industry Applications

**Personal Finance**:
1. **Savings Accounts**: Calculate growth of deposits
2. **Retirement Planning**: Project 401(k)/IRA balances
3. **College Savings**: 529 plan projections
4. **Mortgage Calculations**: Compound interest on loan balances
5. **Credit Cards**: Compound interest on unpaid balances (debt trap)

**Investment Management**:
1. **Portfolio Growth**: Project investment returns
2. **Dividend Reinvestment**: DRIP (Dividend Reinvestment Plan) modeling
3. **Bond Pricing**: Compound interest in yield calculations
4. **Certificate of Deposit (CD)**: Interest accrual calculations

**Banking**:
1. **Interest Rate Products**: Savings, money market accounts
2. **APY Calculations**: Annual Percentage Yield disclosures
3. **Loan Amortization**: Interest component of payments
4. **Fixed Income Products**: Bond coupon reinvestment

**Corporate Finance**:
1. **Cost of Capital**: WACC with compounding returns
2. **Investment Analysis**: Growing future cash flows
3. **Pension Fund Management**: Liability projections
4. **Lease Accounting**: Present value calculations

### 6.3 Educational Context

**Financial Literacy**:
- Teaching the power of starting early (time value)
- Demonstrating retirement savings necessity
- Explaining credit card debt dangers
- Illustrating inflation's erosive effect

**Mathematical Concepts**:
- Exponential growth vs. linear growth
- Limits and continuous functions ($e^{rt}$)
- Logarithms (solving for time or rate)
- Geometric series and progressions

### 6.4 Related Algorithms and Formulas

**1. Simple Interest**:
$$A = P(1 + rt)$$
- Linear growth, not exponential
- Used for short-term loans, some bonds
- Compound interest reduces to simple when $n=1$, $t$ small

**2. Continuous Compounding**:
$$A = Pe^{rt}$$
- Limit as $n \to \infty$
- Used in Black-Scholes, quantitative finance
- Implementation:
  ```rust
  fn continuous_compound(principal: f64, rate: f64, years: f64) -> f64 {
      principal * (rate * years).exp()
  }
  ```

**3. Effective Annual Rate (EAR)**:
$$\text{EAR} = \left(1 + \frac{r}{n}\right)^n - 1$$
- Converts nominal rate to actual annual return
- Enables comparison across different compounding frequencies

**4. Annual Percentage Yield (APY)**:
- Same formula as EAR
- Used in consumer finance for transparency
- Required disclosure by regulation (Truth in Savings Act)

**5. Present Value (Inverse)**:
$$P = \frac{A}{\left(1 + \frac{r}{n}\right)^{nt}}$$
- Discounts future value to present
- Inverse operation of compound interest

**6. Solving for Rate (IRR-like)**:
$$r = n\left[\left(\frac{A}{P}\right)^{1/(nt)} - 1\right]$$
- Iterative or logarithmic solution
- Implementation:
  ```rust
  fn solve_for_rate(principal: f64, final_amount: f64, comp_per_year: u32, years: f64) -> f64 {
      let n = comp_per_year as f64;
      let exponent = 1.0 / (n * years);
      n * ((final_amount / principal).powf(exponent) - 1.0)
  }
  ```

**7. Solving for Time**:
$$t = \frac{\ln(A/P)}{n \cdot \ln(1 + r/n)}$$
- Uses logarithms
- Answers: "How long to reach target?"

**8. Rule of 72 (Approximation)**:
$$t_{\text{double}} \approx \frac{72}{100r}$$
- Quick mental math for doubling time
- Works well for rates 6-10%

**When to Use Compound Interest**:
- **Multi-period investments**: Standard application
- **Regular compounding**: Savings accounts, bonds
- **Exponential growth modeling**: Population, viral spread
- **Long time horizons**: Retirement, college savings

**When to Use Alternatives**:
- **Simple Interest**: Short-term loans, T-bills
- **Continuous Compounding**: Theoretical models, options pricing
- **Annuity Formulas**: Regular deposits/withdrawals
- **Amortization**: Loans with payments

## 7. References

### Foundational Mathematics

1. **Euler, L. (1748)**. *Introductio in analysin infinitorum*.
   - Early work on exponential functions and compound interest

2. **Fisher, I. (1930)**. *The Theory of Interest*. Macmillan.
   - Mathematical foundations of interest theory

### Finance Textbooks

3. **Bodie, Z., Kane, A., & Marcus, A. J. (2021)**. *Investments* (12th ed.). McGraw-Hill.
   - Chapter 5: Time Value of Money

4. **Ross, S. A., Westerfield, R. W., & Jaffe, J. (2019)**. *Corporate Finance* (12th ed.). McGraw-Hill.
   - Comprehensive treatment of time value concepts

5. **Brealey, R. A., Myers, S. C., & Allen, F. (2020)**. *Principles of Corporate Finance* (13th ed.). McGraw-Hill.
   - Chapter 2: How to Calculate Present Values

### Practical Guides

6. **CFA Institute**: Level I Curriculum - Quantitative Methods
   - Time value of money, compound interest calculations

7. **Brigham, E. F., & Houston, J. F. (2021)**. *Fundamentals of Financial Management* (15th ed.). Cengage.
   - Practical applications and examples

### Regulatory and Standards

8. **Federal Reserve**: Truth in Savings Act (Regulation DD)
   - APY calculation and disclosure requirements

9. **FDIC**: [Interest Rates: Nominal and Effective](https://www.fdic.gov/)
   - Consumer education on compound interest

### Online Resources

10. **Investopedia**: [Compound Interest](https://www.investopedia.com/terms/c/compoundinterest.asp)
    - Comprehensive guide with calculator

11. **Khan Academy**: [Compound Interest Introduction](https://www.khanacademy.org/economics-finance-domain/core-finance/interest-tutorial/compound-interest-tutorial/v/introduction-to-compound-interest)
    - Video tutorials and practice problems

12. **Wikipedia**: [Compound Interest](https://en.wikipedia.org/wiki/Compound_interest)
    - History, formulas, examples

### Software and Calculators

13. **Excel**: `FV()` function - Future Value with compound interest
14. **Python**: `numpy.fv()` - NumPy financial functions
15. **R**: `fv()` in FinCal package
16. **Online Calculators**: Bankrate, Calculator.net, Investor.gov

### Historical and Cultural

17. **Mandelbrot, B. B., & Hudson, R. L. (2004)**. *The (Mis)Behavior of Markets*. Basic Books.
    - Discusses limitations of compound return assumptions

18. **Shiller, R. J. (2015)**. *Irrational Exuberance* (3rd ed.). Princeton University Press.
    - Context on investment returns and expectations
