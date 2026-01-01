# NPV Sensitivity Analysis

## 1. Overview

NPV Sensitivity Analysis is a critical financial tool that evaluates how the Net Present Value of an investment changes across different discount rates. This technique addresses one of the key limitations of standard NPV analysis: the assumption of a known, fixed discount rate.

In practice, the appropriate discount rate is often uncertain and may vary due to:
- Changes in market interest rates
- Company's cost of capital fluctuations
- Risk perception adjustments
- Economic condition changes

Sensitivity analysis provides decision-makers with a range of possible outcomes, helping them understand the robustness of an investment decision under varying conditions.

**Historical Context**: Sensitivity analysis emerged in operations research during the 1950s and became a standard practice in corporate finance by the 1970s, particularly for capital-intensive projects with long time horizons.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a series of cash flows $CF_0, CF_1, \ldots, CF_n$ and a set of discount rates $r_1, r_2, \ldots, r_m$, compute the NPV for each discount rate to analyze how sensitive the project's value is to changes in the discount rate.

**Formal Definition**:
$$\text{NPV}(r_i) = \sum_{t=0}^{n} \frac{CF_t}{(1 + r_i)^t} \quad \forall i \in \{1, 2, \ldots, m\}$$

Output: Vector $[\text{NPV}(r_1), \text{NPV}(r_2), \ldots, \text{NPV}(r_m)]$

### 2.2 Mathematical Model

**Input Specifications**:
- `cash_flows`: Vector $CF \in \mathbb{R}^{n+1}$ representing cash flows for each period
- `discount_rates`: Vector $R = [r_1, r_2, \ldots, r_m] \in \mathbb{R}^m$ where $r_i > -1$

**Output Specification**:
- Vector $NPV \in \mathbb{R}^m$ where $NPV_i = \text{NPV}(CF, r_i)$

**Constraints**:
- $n \geq 0$ (at least one cash flow)
- $m \geq 1$ (at least one discount rate)
- $r_i > -1$ for all $i$ (discount rates must be greater than -100%)

**Key Mathematical Properties**:

1. **Monotonicity**: For typical investment cash flows (negative initial, positive subsequent):
   $$r_i < r_j \implies \text{NPV}(r_i) > \text{NPV}(r_j)$$
   
2. **Continuity**: NPV is a continuous function of the discount rate

3. **Differentiability**: 
   $$\frac{d\text{NPV}}{dr} = -\sum_{t=1}^{n} \frac{t \cdot CF_t}{(1 + r)^{t+1}}$$
   
4. **Convexity**: For most cash flow patterns, NPV is a convex function of $r$

### 2.3 Sensitivity Metrics

**NPV Sensitivity Slope**:
$$\text{Sensitivity} = \frac{\Delta \text{NPV}}{\Delta r} = \frac{\text{NPV}(r_2) - \text{NPV}(r_1)}{r_2 - r_1}$$

**Interpretation**:
- Large negative slope: Project highly sensitive to rate changes (higher risk)
- Small negative slope: Project relatively insensitive (lower risk)

## 3. Algorithm Description

### 3.1 Intuition

The algorithm computes multiple NPV calculations in parallel, each with a different discount rate. This creates a profile showing how the project's value changes with the cost of capital.

Think of it as answering: "What would this investment be worth if our required return was X%, Y%, or Z%?"

The results help identify:
- **Break-even rate**: Where NPV crosses zero (Internal Rate of Return)
- **Safety margin**: How much rates can increase before NPV becomes negative
- **Value volatility**: How stable the investment value is to market changes

### 3.2 Pseudocode

```
Algorithm: NPV Sensitivity Analysis
Input: cash_flows[0..n], discount_rates[0..m-1]
Output: npv_values[0..m-1]

1. Initialize result vector npv_values of size m
2. For each i from 0 to m-1:
3.     current_rate ← discount_rates[i]
4.     npv ← 0
5.     For each t from 0 to n:
6.         npv ← npv + cash_flows[t] / (1 + current_rate)^t
7.     npv_values[i] ← npv
8. Return npv_values
```

**Optimized Functional Version**:
```
Algorithm: NPV Sensitivity (Functional)
Input: cash_flows[0..n], discount_rates[0..m-1]
Output: npv_values[0..m-1]

1. For each rate in discount_rates:
2.     Map cash_flows to present values using rate
3.     Sum present values to get NPV for this rate
4.     Collect NPVs into result vector
5. Return result vector
```

### 3.3 Step-by-Step Example

**Scenario**: Evaluating a 3-year project across different discount rates

**Given**:
- Cash flows: [-1000, 400, 400, 400]
- Discount rates: [5%, 10%, 15%, 20%]

**Detailed Calculation**:

**Rate 1: r = 0.05 (5%)**

| Period | Cash Flow | Discount Factor | Present Value |
|--------|-----------|-----------------|---------------|
| 0 | -1000 | 1.000 | -1000.00 |
| 1 | 400 | 1.050 | 380.95 |
| 2 | 400 | 1.103 | 362.81 |
| 3 | 400 | 1.158 | 345.54 |
| **NPV** | | | **89.30** |

**Rate 2: r = 0.10 (10%)**

| Period | Cash Flow | Discount Factor | Present Value |
|--------|-----------|-----------------|---------------|
| 0 | -1000 | 1.000 | -1000.00 |
| 1 | 400 | 1.100 | 363.64 |
| 2 | 400 | 1.210 | 330.58 |
| 3 | 400 | 1.331 | 300.53 |
| **NPV** | | | **-5.26** |

**Rate 3: r = 0.15 (15%)**

| Period | Cash Flow | Discount Factor | Present Value |
|--------|-----------|-----------------|---------------|
| 0 | -1000 | 1.000 | -1000.00 |
| 1 | 400 | 1.150 | 347.83 |
| 2 | 400 | 1.323 | 302.46 |
| 3 | 400 | 1.521 | 263.01 |
| **NPV** | | | **-86.71** |

**Rate 4: r = 0.20 (20%)**

| Period | Cash Flow | Discount Factor | Present Value |
|--------|-----------|-----------------|---------------|
| 0 | -1000 | 1.000 | -1000.00 |
| 1 | 400 | 1.200 | 333.33 |
| 2 | 400 | 1.440 | 277.78 |
| 3 | 400 | 1.728 | 231.48 |
| **NPV** | | | **-157.41** |

**Summary Results**:

| Discount Rate | NPV | Decision |
|---------------|-----|----------|
| 5% | 89.30 | Accept ✓ |
| 10% | -5.26 | Marginal ≈ |
| 15% | -86.71 | Reject ✗ |
| 20% | -157.41 | Reject ✗ |

**Analysis**:
- **Break-even rate**: Approximately 9.8% (between 5% and 10%)
- **Sensitivity**: High - small rate changes significantly impact decision
- **Recommendation**: Only accept if confident discount rate ≤ 10%

## 4. Complexity Analysis

### 4.1 Time Complexity

**Best, Average, and Worst Case**: $O(m \times n)$

**Derivation**:
- Outer loop: $m$ iterations (one per discount rate)
- Inner loop: $n+1$ iterations (one per cash flow)
- Per iteration: 
  - Power operation: $O(\log t)$ using binary exponentiation
  - Division and addition: $O(1)$
- Total: $O(m \times n \times \log n)$

**Practical Complexity**: $O(m \times n)$
- For typical financial applications: $n < 50$, $m < 20$
- Power operations are highly optimized in modern processors
- Effective linear time per rate-cashflow pair

### 4.2 Space Complexity

**Auxiliary Space**: $O(m)$

**Breakdown**:
- Output vector of NPVs: $O(m)$
- Iterator overhead: $O(1)$
- Temporary computation variables: $O(1)$

**Total Space**: $O(n + m)$ including inputs

**Memory Access Pattern**:
- Sequential reads of cash flows for each rate
- Good cache locality for cash_flows array
- Output vector built incrementally

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Ownership and Borrowing**:
```rust
pub fn npv_sensitivity(cash_flows: &[f64], discount_rates: &[f64]) -> Vec<f64>
```
- Both inputs borrowed as slices (immutable references)
- Returns owned `Vec<f64>` with NPV results
- No lifetime annotations needed (inputs not stored in output)

**Iterator Chain Pattern**:
```rust
discount_rates
    .iter()
    .cloned()  // Clone f64 values (cheap, Copy trait)
    .map(|rate| {
        // Inner NPV calculation
        cash_flows
            .iter()
            .enumerate()
            .map(|(t, &cf)| cf / (1.0 + rate).powi(t as i32))
            .sum()
    })
    .collect()  // Collect into Vec<f64>
```

**Nested Iterator Composition**:
- Outer iterator: processes each discount rate
- Inner iterator: calculates NPV for current rate
- Zero-cost abstraction: compiles to efficient loops
- Lazy evaluation: only computes when collecting

**Type Inference**:
- Return type `Vec<f64>` guides `collect()`
- Intermediate types inferred automatically
- Explicit types unnecessary in most cases

**Floating-Point Considerations**:
- Uses `f64` for financial precision
- `powi()` for integer exponentiation (faster than `powf()`)
- Results accurate to ~15 significant digits

### 5.2 Edge Cases

1. **Empty Cash Flows**:
   ```rust
   let result = npv_sensitivity(&[], &[0.05, 0.10]);
   // Result: [0.0, 0.0] - all NPVs are zero
   ```

2. **Empty Discount Rates**:
   ```rust
   let result = npv_sensitivity(&[-1000.0, 500.0, 600.0], &[]);
   // Result: [] - empty vector
   ```

3. **Single Rate (Degenerate Case)**:
   ```rust
   let result = npv_sensitivity(&[-1000.0, 500.0, 600.0], &[0.10]);
   // Result: [5.79] - reduces to single NPV calculation
   ```

4. **Zero Discount Rate**:
   ```rust
   let result = npv_sensitivity(&[-1000.0, 400.0, 400.0, 400.0], &[0.0]);
   // Result: [200.0] - simple sum of cash flows
   ```

5. **Negative Discount Rate** (rare but valid):
   ```rust
   let result = npv_sensitivity(&[-100.0, 50.0], &[-0.10]);
   // Result: [5.0] - represents deflation
   ```

6. **Very Large Rate Range**:
   ```rust
   let rates: Vec<f64> = (0..100).map(|i| i as f64 / 100.0).collect();
   let result = npv_sensitivity(&[-1000.0, 400.0, 400.0, 400.0], &rates);
   // Creates NPV profile from 0% to 99%
   ```

7. **High Precision Requirements**:
   - For extreme sensitivity analysis, consider using `BigDecimal` or similar
   - Floating-point errors compound across many calculations

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Cloud Cost Sensitivity Analysis**
- **Context**: Evaluating cloud migration with uncertain future costs
- **Example**:
  ```rust
  let migration_costs = vec![-500_000.0, 120_000.0, 120_000.0, 120_000.0, 120_000.0];
  let rate_scenarios = vec![0.08, 0.10, 0.12, 0.15]; // Different WACC scenarios
  let npvs = npv_sensitivity(&migration_costs, &rate_scenarios);
  ```
- **Use**: Determine if migration makes sense under different cost of capital assumptions

**2. Product Lifecycle NPV Range**
- **Context**: SaaS product with uncertain market conditions
- **Rates Represent**: Different market risk premiums
  - Bull market: 8%
  - Base case: 12%
  - Bear market: 18%
- **Output**: Range of expected product values

**3. Infrastructure Investment Decision**
- **Context**: Choosing between on-premise vs. cloud infrastructure
- **Cash Flows**: Total cost of ownership over 5 years
- **Rate Scenarios**: 
  - Low risk: 6% (stable business)
  - Medium risk: 10% (growth phase)
  - High risk: 15% (startup)
- **Decision**: Choose option with positive NPV across all scenarios

**4. Open Source Contribution ROI**
- **Context**: Deciding whether to contribute to/maintain open source
- **Cash Flows**:
  - Development time costs (negative)
  - Reduced external dependencies costs (positive)
  - Community contributions value (positive)
- **Rate Scenarios**: Different opportunity cost assumptions
- **Insight**: Shows value stability across different valuation approaches

### 6.2 Industry Applications

**Corporate Finance**:
- Capital budgeting under interest rate uncertainty
- Merger & acquisition valuations with varying WACC
- Comparing projects with different risk profiles

**Real Estate Development**:
- Property investment across economic cycles
- Different financing cost scenarios
- Risk-adjusted project evaluation

**Energy Sector**:
- Long-term renewable energy projects
- Sensitivity to regulatory changes (reflected in rates)
- Commodity price uncertainty analysis

**Pharmaceutical R&D**:
- Drug development with long approval timelines
- High initial costs, uncertain future revenues
- Risk-adjusted discount rates for different trial phases

**Venture Capital**:
- Startup valuations at different hurdle rates
- Stage-specific discount rates (seed vs. Series A)
- Exit scenario modeling

### 6.3 Visualization and Interpretation

**NPV Profile Chart**:
```
NPV
 |
500 |         ●
    |       /
 0  |-----●-------- IRR (break-even rate)
    |    /
-500|  ●
    |
    +------------------ Discount Rate
    5%  10%  15%  20%
```

**Decision Matrix**:
- Flat slope: Robust project, insensitive to rate changes
- Steep slope: Risky project, highly rate-dependent
- Multiple zero-crossings: Complex cash flow pattern (caution)

### 6.4 Related Algorithms

**Complementary Techniques**:

1. **Monte Carlo Simulation**:
   - Extends to probability distributions of rates
   - Generates distribution of NPV outcomes
   - More sophisticated than discrete scenarios

2. **Scenario Analysis**:
   - Combines rate sensitivity with cash flow variations
   - Creates best/base/worst case scenarios
   - NPV sensitivity is one dimension

3. **Break-Even Analysis**:
   - Solves for rate where NPV = 0 (IRR)
   - Identifies the maximum acceptable rate
   - Uses interpolation or root-finding

4. **Duration Analysis**:
   - Measures weighted average time to cash flows
   - First-order approximation of rate sensitivity
   - $\text{Duration} = -\frac{1}{\text{PV}} \frac{d\text{PV}}{dr}$

5. **Tornado Diagrams**:
   - Shows sensitivity to multiple variables simultaneously
   - Discount rate is typically one key variable
   - Ranks variables by impact magnitude

**When to Use NPV Sensitivity vs. Alternatives**:
- **NPV Sensitivity**: Quick, deterministic, easy to communicate
- **Monte Carlo**: When probability distributions are known, complex dependencies
- **Scenario Analysis**: When multiple factors vary together (e.g., recession scenario)
- **Real Options**: When project has flexibility to adapt based on new information

## 7. References

### Academic Literature

1. **Hertz, D. B. (1964)**. "Risk Analysis in Capital Investment". *Harvard Business Review*, 42(1), 95-106.
   - Pioneering work on sensitivity analysis in capital budgeting

2. **Brealey, R. A., Myers, S. C., & Allen, F. (2020)**. *Principles of Corporate Finance* (13th ed.). McGraw-Hill Education.
   - Chapter 10: Project Analysis - Comprehensive treatment of sensitivity analysis

3. **Copeland, T. E., & Antikarov, V. (2001)**. *Real Options: A Practitioner's Guide*. Texere.
   - Modern approaches to valuation under uncertainty

4. **Saltelli, A., et al. (2008)**. *Global Sensitivity Analysis: The Primer*. Wiley.
   - Mathematical foundations of sensitivity analysis

### Practical Guides

5. **CFA Institute**: Level II Curriculum - Corporate Finance
   - Practical applications in investment analysis

6. **Project Management Institute (PMI)**: *Practice Standard for Project Risk Management*
   - Industry best practices for financial risk analysis

### Online Resources

7. **Investopedia**: [Sensitivity Analysis](https://www.investopedia.com/terms/s/sensitivityanalysis.asp)

8. **Corporate Finance Institute**: [NPV Sensitivity Analysis Tutorial](https://corporatefinanceinstitute.com/resources/financial-modeling/sensitivity-analysis/)

### Software Tools

9. **Excel**: Data Tables and Goal Seek for sensitivity analysis
10. **Python**: NumPy, pandas, matplotlib for NPV profiling
11. **R**: Financial modeling packages (FinCal, tidyquant)

### Standards and Frameworks

12. **FASB ASC 820**: Fair Value Measurement - Sensitivity disclosure requirements
13. **Basel III**: Risk sensitivity requirements for financial institutions
