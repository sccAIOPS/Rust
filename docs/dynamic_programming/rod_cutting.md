# Rod Cutting Problem

## 1. Overview

The Rod Cutting problem finds the maximum profit obtainable by cutting a rod into pieces and selling them, given a price list for different lengths. This is a classic example of optimal substructure in dynamic programming.

**File**: `src/dynamic_programming/rod_cutting.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- Rod of length $n$
- Price array $p[1..n]$ where $p[i]$ is the price for length $i$

Find the maximum revenue $r_n$ obtainable by cutting the rod and selling pieces.

### 2.2 Recurrence

$$r_n = \max_{1 \leq i \leq n}(p[i] + r_{n-i})$$

With base case: $r_0 = 0$

### 2.3 Optimal Substructure

If an optimal solution cuts the rod into pieces of lengths $i_1, i_2, \ldots, i_k$, then:
$$r_n = p[i_1] + p[i_2] + \ldots + p[i_k]$$

And removing any piece $i_j$ leaves an optimal solution for the remaining length.

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION rod_cut(prices)
    n ← length(prices)
    IF n = 0 THEN RETURN 0
    
    max_profit[0..n] ← 0
    
    FOR length ← 1 TO n DO
        max_profit[length] ← prices[length-1]  // No cut
        FOR cut ← 1 TO length-1 DO
            current ← prices[cut-1] + max_profit[length-cut]
            max_profit[length] ← MAX(max_profit[length], current)
        END FOR
    END FOR
    
    RETURN max_profit[n]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: prices = [1, 5, 8, 9, 10, 17, 17, 20]

| Length | Best Split | Max Profit |
|--------|------------|------------|
| 1 | 1 | 1 |
| 2 | 2 | 5 |
| 3 | 3 | 8 |
| 4 | 2+2 | 10 |
| 5 | 2+3 | 13 |
| 6 | 6 | 17 |
| 7 | 2+2+3 or 1+6 | 18 |
| 8 | 2+6 | **22** |

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n²)**: Two nested loops

### 4.2 Space Complexity
- **O(n)**: Single DP array

## 5. Implementation Notes

### 5.1 Rust Implementation (Functional Style)

```rust
pub fn rod_cut(prices: &[usize]) -> usize {
    if prices.is_empty() { return 0; }

    (1..=prices.len()).fold(
        vec![0; prices.len() + 1],
        |mut max_profit, rod_length| {
            max_profit[rod_length] = (1..=rod_length)
                .map(|cut| prices[cut-1] + max_profit[rod_length-cut])
                .fold(prices[rod_length-1], |max, curr| max.max(curr));
            max_profit
        }
    )[prices.len()]
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty prices | 0 |
| Single length | prices[0] |
| All zeros | 0 |
| Greedy fails | DP finds optimal |

## 6. Why Greedy Fails

**Example**: prices = [2, 5, 7, 8] for length 4
- Greedy (max price/length): 4×p[1]=8 or 2×p[2]=10
- But p[2]+p[2]=5+5=**10** is optimal (same as greedy here)

**Counter-example**: prices = [3, 5, 8, 9]
- Greedy: p[4]=9
- Optimal: p[2]+p[2]=**10**

## 7. Applications

1. **Manufacturing**: Cutting raw materials optimally
2. **Resource Allocation**: Dividing resources for max value
3. **Stock Cutting**: Fabric, lumber, metal cutting

## 8. Visualization

```
Rod length 8, prices = [1, 5, 8, 9, 10, 17, 17, 20]

Option 1: [────────8────────] = 20
Option 2: [──2──][────6────] = 5 + 17 = 22 ✓ Best
Option 3: [─1─][─1─][───6───] = 1 + 1 + 17 = 19
Option 4: [──2──][──2──][──2──][──2──] = 20
```

## 9. References

1. Cormen, T. H. et al. (2009). *Introduction to Algorithms*, Chapter 15
2. [Wikipedia - Cutting stock problem](https://en.wikipedia.org/wiki/Cutting_stock_problem)
