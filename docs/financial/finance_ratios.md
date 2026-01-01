# Financial Ratios

## 1. Overview

Financial Ratios are quantitative metrics derived from financial statements that provide insights into a company's performance, profitability, liquidity, efficiency, and financial health. They are essential tools for investors, analysts, creditors, and management to evaluate and compare businesses.

This module implements four fundamental financial ratios:
1. **Return on Investment (ROI)** - Profitability measure
2. **Debt-to-Equity Ratio** - Leverage/solvency measure
3. **Gross Profit Margin** - Profitability margin
4. **Earnings Per Share (EPS)** - Per-share profitability

These ratios represent the building blocks of financial analysis and are used universally across industries for decision-making and performance evaluation.

**Historical Context**: Financial ratio analysis became systematized in the early 20th century with the work of analysts like Alexander Wall (1919), who pioneered comparative ratio analysis for credit evaluation.

## 2. Mathematical Foundation

### 2.1 Return on Investment (ROI)

**Definition**: Measures the gain or loss generated on an investment relative to its cost.

**Formula**:
$$\text{ROI} = \frac{\text{Gain} - \text{Cost}}{\text{Cost}} = \frac{\text{Net Profit}}{\text{Investment}}$$

**Alternative Expression**:
$$\text{ROI} = \frac{\text{Gain}}{\text{Cost}} - 1$$

**Input**:
- `gain`: Total return/final value (f64)
- `cost`: Initial investment (f64)

**Output**: Ratio (typically expressed as decimal or percentage)

**Interpretation**:
- ROI > 0: Profitable investment
- ROI = 0: Break-even
- ROI < 0: Loss

**Example**: 
- Invest $1,000, gain $1,200
- ROI = (1200 - 1000) / 1000 = 0.20 = 20%

---

### 2.2 Debt-to-Equity Ratio

**Definition**: Measures a company's financial leverage by comparing total debt to shareholder equity.

**Formula**:
$$\text{D/E Ratio} = \frac{\text{Total Debt}}{\text{Shareholder Equity}}$$

**Input**:
- `debt`: Total liabilities (f64)
- `equity`: Total shareholder equity (f64)

**Output**: Ratio (pure number, no units)

**Interpretation**:
- D/E < 1: More equity than debt (conservative)
- D/E = 1: Equal debt and equity
- D/E > 1: More debt than equity (leveraged)
- D/E > 2: Highly leveraged (risky)

**Industry Variations**:
- Technology: Typically low (0.0-0.5)
- Utilities: Typically high (1.0-2.0+)
- Financial services: Very high (5.0-15.0+)

**Example**:
- Total debt: $300K
- Total equity: $150K
- D/E = 300 / 150 = 2.0

---

### 2.3 Gross Profit Margin

**Definition**: Percentage of revenue remaining after subtracting cost of goods sold (COGS).

**Formula**:
$$\text{Gross Profit Margin} = \frac{\text{Revenue} - \text{COGS}}{\text{Revenue}} = \frac{\text{Gross Profit}}{\text{Revenue}}$$

**Alternative Form**:
$$\text{GPM} = 1 - \frac{\text{COGS}}{\text{Revenue}}$$

**Input**:
- `revenue`: Total sales (f64)
- `cost`: Cost of goods sold (f64)

**Output**: Ratio (typically expressed as percentage)

**Interpretation**:
- GPM = 0.20: For every $1 of sales, $0.20 remains after direct costs
- Higher GPM: Better pricing power, lower production costs
- Lower GPM: Competitive pricing pressure, higher costs

**Industry Benchmarks**:
- Software/SaaS: 80-95%
- Retail: 20-40%
- Manufacturing: 25-35%
- Grocery: 20-25%

**Example**:
- Revenue: $1,000
- COGS: $800
- GPM = (1000 - 800) / 1000 = 0.20 = 20%

---

### 2.4 Earnings Per Share (EPS)

**Definition**: Portion of a company's profit allocated to each outstanding share of common stock.

**Formula**:
$$\text{EPS} = \frac{\text{Net Income} - \text{Preferred Dividends}}{\text{Weighted Average Shares Outstanding}}$$

**Input**:
- `net_income`: Total earnings after all expenses and taxes (f64)
- `pref_dividend`: Dividends paid to preferred shareholders (f64)
- `share_avg`: Weighted average common shares outstanding (f64)

**Output**: Per-share value (in currency units per share)

**Interpretation**:
- Higher EPS: More profitable per share
- Growing EPS: Company expanding earnings
- Used in P/E ratio: Price / EPS

**Types**:
1. **Basic EPS**: Uses actual shares outstanding
2. **Diluted EPS**: Includes potential shares from options, convertibles
3. **This implementation**: Basic EPS

**Example**:
- Net income: $350K
- Preferred dividends: $50K
- Average shares: 25K
- EPS = (350 - 50) / 25 = $12 per share

## 3. Algorithm Description

### 3.1 Intuition

All four ratios follow the same basic pattern:
1. Take two financial values
2. Perform arithmetic operation (division or subtraction+division)
3. Return ratio

These are simple arithmetic formulas with profound analytical implications. The key is:
- **What you measure**: Selecting the right ratios for analysis
- **How you interpret**: Understanding industry context and trends
- **Comparative analysis**: Benchmarking against peers and history

### 3.2 Pseudocode

```
Algorithm: Return on Investment
Input: gain, cost
Output: roi

1. net_profit ← gain - cost
2. roi ← net_profit / cost
3. Return roi
```

```
Algorithm: Debt-to-Equity Ratio
Input: debt, equity
Output: d_e_ratio

1. d_e_ratio ← debt / equity
2. Return d_e_ratio
```

```
Algorithm: Gross Profit Margin
Input: revenue, cost
Output: gpm

1. gross_profit ← revenue - cost
2. gpm ← gross_profit / revenue
3. Return gpm
```

```
Algorithm: Earnings Per Share
Input: net_income, pref_dividend, share_avg
Output: eps

1. earnings_available ← net_income - pref_dividend
2. eps ← earnings_available / share_avg
3. Return eps
```

### 3.3 Step-by-Step Examples

#### Example 1: Return on Investment

**Scenario**: Stock investment analysis

**Given**:
- Purchase price (cost): $1,000
- Sale price (gain): $1,200

**Calculation**:
```
Step 1: Calculate net profit
  Net profit = 1200 - 1000 = 200

Step 2: Divide by cost
  ROI = 200 / 1000 = 0.20

Result: ROI = 0.20 = 20%
```

**Interpretation**: Earned 20% return on investment.

---

#### Example 2: Debt-to-Equity Ratio

**Scenario**: Company financial structure analysis

**Given**:
- Total debt: $300,000
- Total equity: $150,000

**Calculation**:
```
Step 1: Divide debt by equity
  D/E = 300,000 / 150,000 = 2.0

Result: D/E Ratio = 2.0
```

**Interpretation**: Company has $2 of debt for every $1 of equity. Moderately leveraged position.

---

#### Example 3: Gross Profit Margin

**Scenario**: Retail business profitability

**Given**:
- Revenue: $1,000,000
- Cost of Goods Sold: $800,000

**Calculation**:
```
Step 1: Calculate gross profit
  Gross profit = 1,000,000 - 800,000 = 200,000

Step 2: Divide by revenue
  GPM = 200,000 / 1,000,000 = 0.20

Result: GPM = 0.20 = 20%
```

**Interpretation**: Retains 20¢ of each sales dollar after direct costs. Typical for retail.

---

#### Example 4: Earnings Per Share

**Scenario**: Public company profitability per share

**Given**:
- Net income: $350,000
- Preferred dividends: $50,000
- Weighted average shares: 25,000

**Calculation**:
```
Step 1: Calculate earnings available to common shareholders
  Earnings = 350,000 - 50,000 = 300,000

Step 2: Divide by shares outstanding
  EPS = 300,000 / 25,000 = 12.00

Result: EPS = $12.00 per share
```

**Interpretation**: Each common share earned $12. If stock price is $120, P/E ratio = 10.

## 4. Complexity Analysis

### 4.1 Time Complexity

**All Ratios**: $O(1)$ - Constant time

**Derivation**:
- Fixed number of arithmetic operations
- No loops or recursion
- No data structure traversal

**Operations per ratio**:
- ROI: 2 operations (subtraction, division)
- D/E: 1 operation (division)
- GPM: 2 operations (subtraction, division)
- EPS: 2 operations (subtraction, division)

### 4.2 Space Complexity

**All Ratios**: $O(1)$ - Constant space

**Breakdown**:
- No additional data structures
- Only local variables for computation
- No recursion (no stack usage)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Function Signatures**:
```rust
pub fn return_on_investment(gain: f64, cost: f64) -> f64
pub fn debt_to_equity(debt: f64, equity: f64) -> f64
pub fn gross_profit_margin(revenue: f64, cost: f64) -> f64
pub fn earnings_per_sale(net_income: f64, pref_dividend: f64, share_avg: f64) -> f64
```

**Type Choices**:
- All parameters and returns: `f64`
  - Financial precision (~15-17 significant digits)
  - Handles large corporate financial figures
  - IEEE 754 standard compliance

**Simple Arithmetic**:
```rust
pub fn return_on_investment(gain: f64, cost: f64) -> f64 {
    (gain - cost) / cost
}
```
- Single expression functions
- Inline expansion by compiler
- No overhead

**No Error Handling**:
- Current implementation doesn't validate inputs
- Potential issues:
  - Division by zero (cost=0, equity=0, share_avg=0)
  - Negative values (may be valid or invalid depending on context)
  - NaN/Infinity propagation

**Production-Ready Version**:
```rust
pub fn return_on_investment(gain: f64, cost: f64) -> Result<f64, &'static str> {
    if cost == 0.0 {
        return Err("Cost cannot be zero");
    }
    if !cost.is_finite() || !gain.is_finite() {
        return Err("Inputs must be finite numbers");
    }
    Ok((gain - cost) / cost)
}
```

**Naming Note**:
- `earnings_per_sale` should be `earnings_per_share`
- Appears to be typo in implementation
- Semantically correct: calculates EPS, not earnings per sale

### 5.2 Edge Cases

**1. Division by Zero**:
```rust
// These will return f64::INFINITY or f64::NAN
debt_to_equity(300.0, 0.0);        // inf
gross_profit_margin(1000.0, 0.0);  // 1.0 (valid: 100% margin)
earnings_per_sale(100.0, 0.0, 0.0); // nan (0/0)
```

**2. Negative Values**:
```rust
// Negative gain (loss)
return_on_investment(800.0, 1000.0);  // -0.20 (20% loss) ✓ Valid

// Negative cost (error in accounting)
return_on_investment(1200.0, -1000.0);  // -0.20 (nonsensical)

// Negative equity (bankruptcy situation)
debt_to_equity(500.0, -100.0);  // -5.0 (valid but concerning)

// Negative revenue (returns > sales)
gross_profit_margin(-100.0, 50.0);  // -1.5 (unusual but possible)
```

**3. Zero Cost/Revenue**:
```rust
// Free investment
return_on_investment(1000.0, 0.0);  // inf (infinite return)

// Zero revenue
gross_profit_margin(0.0, 500.0);  // nan (loss with no sales)
```

**4. Equal Values**:
```rust
return_on_investment(1000.0, 1000.0);  // 0.0 (break-even) ✓
debt_to_equity(100.0, 100.0);          // 1.0 (balanced) ✓
gross_profit_margin(1000.0, 1000.0);   // 0.0 (no margin) ✓
```

**5. Very Large Numbers**:
```rust
// Corporate scale
let revenue = 1e12;  // $1 trillion
let cost = 8e11;     // $800 billion
gross_profit_margin(revenue, cost);  // 0.20 ✓ Handles large values
```

**6. Very Small Numbers**:
```rust
// Micro-investments
return_on_investment(1.10, 1.00);  // 0.10 (10% return) ✓
```

**7. Floating-Point Precision**:
```rust
// Precision issues
let gain = 1000.0 + 1e-15;
let cost = 1000.0;
return_on_investment(gain, cost);  // May be 0.0 due to precision
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Return on Investment (ROI)**

**Cloud Migration ROI**:
```rust
let initial_cost = 500_000.0;  // Migration cost
let annual_savings = 200_000.0;
let years = 3.0;
let total_gain = initial_cost + (annual_savings * years);
let roi = return_on_investment(total_gain, initial_cost);
// roi = 1.2 = 120% return over 3 years
```

**Training Program ROI**:
```rust
let training_cost = 10_000.0;
let productivity_gain_value = 15_000.0;
let roi = return_on_investment(productivity_gain_value, training_cost);
// roi = 0.50 = 50% return
```

**2. Debt-to-Equity (Software Company)**

**Startup Financing Structure**:
```rust
let venture_debt = 2_000_000.0;
let equity_raised = 10_000_000.0;
let d_e = debt_to_equity(venture_debt, equity_raised);
// d_e = 0.2 (conservative leverage for tech)
```

**SaaS Company Analysis**:
```rust
// Well-funded SaaS with minimal debt
let debt = 500_000.0;
let equity = 50_000_000.0;
let d_e = debt_to_equity(debt, equity);
// d_e = 0.01 (very low leverage, typical for SaaS)
```

**3. Gross Profit Margin (Software Products)**

**SaaS Platform**:
```rust
let mrr = 100_000.0;         // Monthly Recurring Revenue
let hosting_cost = 10_000.0; // Cloud hosting
let support = 5_000.0;       // Customer support
let cogs = hosting_cost + support;
let gpm = gross_profit_margin(mrr, cogs);
// gpm = 0.85 = 85% (excellent SaaS margin)
```

**Mobile App Revenue**:
```rust
let app_revenue = 50_000.0;
let app_store_fee = 15_000.0;  // 30% to platform
let server_costs = 5_000.0;
let cogs = app_store_fee + server_costs;
let gpm = gross_profit_margin(app_revenue, cogs);
// gpm = 0.60 = 60%
```

**4. Earnings Per Share (Public Tech Company)**

**Quarterly EPS Calculation**:
```rust
let net_income = 5_000_000.0;
let pref_div = 0.0;  // No preferred stock
let shares = 10_000_000.0;
let eps = earnings_per_sale(net_income, pref_div, shares);
// eps = $0.50 per share (quarterly)
// Annual EPS = $2.00
```

### 6.2 Industry Applications by Ratio

#### Return on Investment

**Real Estate**:
- Property investment returns
- Renovation ROI
- Rental property analysis

**Marketing**:
- Campaign ROI
- Customer acquisition cost vs. lifetime value
- Channel performance

**Manufacturing**:
- Equipment purchase ROI
- Process improvement returns
- Automation investment

**Healthcare**:
- Medical equipment ROI
- IT system implementations
- Facility upgrades

---

#### Debt-to-Equity Ratio

**Banking & Finance**:
- Credit risk assessment
- Loan underwriting
- Portfolio risk management

**Investment Analysis**:
- Stock screening
- Credit rating agencies
- Distress prediction

**Corporate Strategy**:
- Capital structure optimization
- Merger & acquisition evaluation
- Refinancing decisions

---

#### Gross Profit Margin

**Retail**:
- Pricing strategy
- Vendor negotiation
- Product mix optimization

**Manufacturing**:
- Production efficiency
- Cost control
- Pricing power assessment

**Services**:
- Service delivery efficiency
- Capacity utilization
- Billing rate optimization

**E-commerce**:
- Marketplace fee impact
- Shipping cost management
- Product category profitability

---

#### Earnings Per Share

**Public Markets**:
- Stock valuation (P/E ratio)
- Quarter-over-quarter comparisons
- Analyst estimates vs. actuals

**Executive Compensation**:
- Performance-based bonuses
- Stock option exercise decisions
- Long-term incentive plans

**Investor Relations**:
- Earnings calls
- Shareholder communications
- Growth trajectory narratives

### 6.3 Integrated Financial Analysis

**Complete Company Analysis**:
```rust
struct CompanyFinancials {
    revenue: f64,
    cogs: f64,
    operating_expenses: f64,
    interest_expense: f64,
    taxes: f64,
    debt: f64,
    equity: f64,
    shares_outstanding: f64,
    preferred_dividends: f64,
}

impl CompanyFinancials {
    fn net_income(&self) -> f64 {
        self.revenue - self.cogs - self.operating_expenses 
            - self.interest_expense - self.taxes
    }
    
    fn analyze(&self) -> FinancialRatios {
        let gpm = gross_profit_margin(self.revenue, self.cogs);
        let d_e = debt_to_equity(self.debt, self.equity);
        let eps = earnings_per_sale(
            self.net_income(), 
            self.preferred_dividends, 
            self.shares_outstanding
        );
        
        FinancialRatios { gpm, d_e, eps }
    }
}
```

### 6.4 Related Metrics and Ratios

**Profitability Ratios**:
1. **Net Profit Margin**: Net Income / Revenue
2. **Operating Margin**: Operating Income / Revenue
3. **EBITDA Margin**: EBITDA / Revenue
4. **Return on Assets (ROA)**: Net Income / Total Assets
5. **Return on Equity (ROE)**: Net Income / Equity

**Leverage Ratios**:
1. **Debt Ratio**: Total Debt / Total Assets
2. **Equity Ratio**: Total Equity / Total Assets
3. **Interest Coverage**: EBIT / Interest Expense
4. **Debt Service Coverage**: Operating Income / Debt Service

**Liquidity Ratios**:
1. **Current Ratio**: Current Assets / Current Liabilities
2. **Quick Ratio**: (Current Assets - Inventory) / Current Liabilities
3. **Cash Ratio**: Cash / Current Liabilities

**Efficiency Ratios**:
1. **Asset Turnover**: Revenue / Total Assets
2. **Inventory Turnover**: COGS / Average Inventory
3. **Receivables Turnover**: Revenue / Accounts Receivable

**Market Ratios**:
1. **Price-to-Earnings (P/E)**: Stock Price / EPS
2. **Price-to-Book (P/B)**: Stock Price / Book Value per Share
3. **Dividend Yield**: Annual Dividend / Stock Price
4. **Payout Ratio**: Dividends / Net Income

**DuPont Analysis** (ROE Decomposition):
$$\text{ROE} = \text{Net Margin} \times \text{Asset Turnover} \times \text{Equity Multiplier}$$
$$\text{ROE} = \frac{\text{Net Income}}{\text{Revenue}} \times \frac{\text{Revenue}}{\text{Assets}} \times \frac{\text{Assets}}{\text{Equity}}$$

## 7. References

### Foundational Texts

1. **Graham, B., & Dodd, D. L. (1934)**. *Security Analysis*. McGraw-Hill.
   - Classic text establishing fundamental analysis

2. **Brigham, E. F., & Ehrhardt, M. C. (2020)**. *Financial Management: Theory & Practice* (16th ed.). Cengage.
   - Comprehensive coverage of financial ratios

3. **Ross, S. A., Westerfield, R. W., & Jaffe, J. (2019)**. *Corporate Finance* (12th ed.). McGraw-Hill.
   - Modern corporate finance perspective

### Analytical Framework

4. **Penman, S. H. (2012)**. *Financial Statement Analysis and Security Valuation* (5th ed.). McGraw-Hill.
   - Connecting ratios to valuation

5. **Palepu, K. G., Healy, P. M., & Peek, E. (2019)**. *Business Analysis and Valuation: IFRS Edition* (5th ed.). Cengage.
   - Framework for ratio analysis

### Standards and Practice

6. **CFA Institute**: Level I Curriculum - Financial Reporting and Analysis
   - Industry-standard ratio definitions and usage

7. **Financial Accounting Standards Board (FASB)**: Generally Accepted Accounting Principles (GAAP)
   - Accounting standards underlying ratio inputs

8. **International Financial Reporting Standards (IFRS)**
   - Global accounting standards

### Industry Research

9. **Damodaran, A. (2012)**. *Investment Valuation* (3rd ed.). Wiley.
   - Valuation applications of financial ratios

10. **Wahlen, J., Baginski, S., & Bradshaw, M. (2017)**. *Financial Reporting, Financial Statement Analysis and Valuation* (9th ed.). Cengage.
    - Practical analysis techniques

### Online Resources

11. **Investopedia**: Financial Ratio sections
    - [ROI](https://www.investopedia.com/terms/r/returnoninvestment.asp)
    - [Debt-to-Equity](https://www.investopedia.com/terms/d/debtequityratio.asp)
    - [Gross Margin](https://www.investopedia.com/terms/g/grossmargin.asp)
    - [EPS](https://www.investopedia.com/terms/e/eps.asp)

12. **Corporate Finance Institute**: [Financial Ratios](https://corporatefinanceinstitute.com/resources/accounting/financial-ratios/)

### Databases and Tools

13. **Bloomberg Terminal**: Financial ratio calculations and comparisons
14. **Yahoo Finance**: Public company ratio displays
15. **SEC EDGAR**: Primary financial statement source
16. **Morningstar**: Investment research and ratio analysis

### Historical Perspective

17. **Horrigan, J. O. (1968)**. "A Short History of Financial Ratio Analysis". *The Accounting Review*, 43(2), 284-294.
    - Historical development of ratio analysis

18. **Wall, A. (1919)**. *Study of Credit Barometrics*. Federal Reserve Board.
    - Pioneer work in systematic ratio analysis
