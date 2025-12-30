# Coin Change Problem

## 1. Overview

The Coin Change problem finds the minimum number of coins needed to make a given amount. This is the classic "unbounded knapsack" variant where each coin denomination can be used unlimited times.

**File**: `src/dynamic_programming/coin_change.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- Coin denominations $c_1, c_2, \ldots, c_n$
- Target amount $A$

Find the minimum number of coins to make exactly $A$, or determine it's impossible.

### 2.2 Mathematical Model

Let $dp[i]$ = minimum coins needed for amount $i$.

**Recurrence**:
$$dp[i] = \begin{cases}
0 & \text{if } i = 0 \\
\min_{c_j \leq i}(dp[i - c_j] + 1) & \text{if } \exists c_j \leq i \\
\infty & \text{otherwise}
\end{cases}$$

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION coin_change(coins, amount)
    dp[0..amount] ← None (impossible)
    dp[0] ← Some(0)
    
    FOR curr ← 0 TO amount DO
        FOR each coin IN coins WHERE coin ≤ curr DO
            IF dp[curr - coin] is Some(k) THEN
                dp[curr] ← MIN(dp[curr], k + 1)
            END IF
        END FOR
    END FOR
    
    RETURN dp[amount]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: coins = [1, 2, 5], amount = 11

| Amount | Min Coins | Coins Used |
|--------|-----------|------------|
| 0 | 0 | - |
| 1 | 1 | 1 |
| 2 | 1 | 2 |
| 3 | 2 | 1+2 |
| 4 | 2 | 2+2 |
| 5 | 1 | 5 |
| 6 | 2 | 5+1 |
| 7 | 2 | 5+2 |
| 8 | 3 | 5+2+1 |
| 9 | 3 | 5+2+2 |
| 10 | 2 | 5+5 |
| 11 | **3** | **5+5+1** |

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(amount × n)** where n = number of coin denominations

### 4.2 Space Complexity
- **O(amount)** for the DP array

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn coin_change(coins: &[usize], amount: usize) -> Option<usize> {
    let mut min_coins = vec![None; amount + 1];
    min_coins[0] = Some(0);

    (0..=amount).for_each(|curr_amount| {
        coins.iter()
            .filter(|&&coin| curr_amount >= coin)
            .for_each(|&coin| {
                if let Some(prev) = min_coins[curr_amount - coin] {
                    min_coins[curr_amount] = Some(
                        min_coins[curr_amount]
                            .map_or(prev + 1, |curr| curr.min(prev + 1))
                    );
                }
            });
    });

    min_coins[amount]
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Amount = 0 | Some(0) |
| No coins | None |
| Impossible | None |
| Single coin | amount / coin (if divisible) |

### 5.3 Greedy Doesn't Work

**Counter-example**: coins = [1, 12, 20], amount = 24
- Greedy: 20 + 1 + 1 + 1 + 1 = 5 coins
- Optimal: 12 + 12 = **2 coins**

## 6. Applications

1. **Currency Exchange**: Optimal change-making
2. **Resource Allocation**: Minimizing resource units
3. **Covering Problems**: Minimum elements to cover

## 7. Variants

| Variant | Description |
|---------|-------------|
| Coin Change 2 | Count number of ways |
| Limited Coins | Each coin has limited quantity |
| With Constraints | Additional conditions |

## 8. References

1. [LeetCode Problem 322](https://leetcode.com/problems/coin-change/)
2. [Wikipedia - Change-making problem](https://en.wikipedia.org/wiki/Change-making_problem)
