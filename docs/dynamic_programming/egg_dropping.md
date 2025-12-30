# Egg Dropping Problem

## 1. Overview

The Egg Dropping puzzle determines the minimum number of trials needed to find the critical floor in a building, from which an egg will break when dropped. Given `e` eggs and `f` floors, find the strategy that minimizes worst-case trials.

**File**: `src/dynamic_programming/egg_dropping.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- $e$ eggs
- $f$ floors
- Eggs break from floor $k$ and above (unknown $k$)

Find minimum trials to determine $k$ in the worst case.

### 2.2 Recurrence

Let $dp[e][f]$ = minimum trials for $e$ eggs and $f$ floors.

$$dp[e][f] = 1 + \min_{1 \leq x \leq f} \max(dp[e-1][x-1], dp[e][f-x])$$

Where:
- $dp[e-1][x-1]$: egg breaks, check floors below
- $dp[e][f-x]$: egg survives, check floors above

### 2.3 Base Cases

- $dp[e][0] = 0$: No floors, no trials needed
- $dp[e][1] = 1$: One floor, one trial needed
- $dp[1][f] = f$: One egg, must try each floor from bottom

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION egg_drop(eggs, floors)
    IF eggs = 0 THEN RETURN None
    IF eggs = 1 OR floors ≤ 1 THEN RETURN floors
    
    dp[1..eggs][0..floors] ← 0
    
    // Base cases
    FOR i ← 1 TO eggs DO dp[i][1] ← 1
    FOR j ← 1 TO floors DO dp[1][j] ← j
    
    // Fill table
    FOR i ← 2 TO eggs DO
        FOR j ← 2 TO floors DO
            dp[i][j] ← MIN over k in [1,j] of:
                1 + MAX(dp[i-1][k-1], dp[i][j-k])
        END FOR
    END FOR
    
    RETURN dp[eggs][floors]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: 2 eggs, 10 floors

Strategy: Drop from floor 4, then 7, then 9, then 10...

| Eggs | Floors | Min Trials |
|------|--------|------------|
| 2 | 1 | 1 |
| 2 | 2 | 2 |
| 2 | 3 | 2 |
| 2 | 6 | 3 |
| 2 | 10 | **4** |
| 2 | 36 | 8 |

**Optimal Strategy for 2 eggs, 10 floors**:
1. Drop from floor 4
2. If breaks: linear search floors 1-3 (worst: 4 trials)
3. If survives: drop from floor 7 (worst: 4 trials)

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(e × f²)** for the basic DP
- Can be optimized to **O(e × f)** using binary search

### 4.2 Space Complexity
- **O(e × f)** for the DP table

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn egg_drop(eggs: usize, floors: usize) -> Option<usize> {
    if eggs == 0 { return None; }
    if eggs == 1 || floors <= 1 { return Some(floors); }

    let mut dp = vec![vec![0; floors + 1]; eggs + 1];

    (1..=eggs).for_each(|i| { dp[i][1] = 1; });
    (1..=floors).for_each(|j| { dp[1][j] = j; });

    (2..=eggs).for_each(|i| {
        (2..=floors).for_each(|j| {
            dp[i][j] = (1..=j)
                .map(|k| 1 + dp[i-1][k-1].max(dp[i][j-k]))
                .min()
                .unwrap();
        });
    });

    Some(dp[eggs][floors])
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| 0 eggs | None |
| 0 floors | 0 |
| 1 egg, any floors | floors |
| Many eggs, 1 floor | 1 |

## 6. Applications

1. **Software Testing**: Minimizing test iterations
2. **Quality Control**: Finding failure thresholds
3. **Binary Search Extensions**: Non-uniform search spaces

## 7. Key Insight: Risk vs Information Trade-off

```
High floor drop:
  ├─ Egg breaks → Much info lost, few eggs remain
  └─ Egg survives → Much info gained

Low floor drop:
  ├─ Egg breaks → Little info lost, eggs preserved
  └─ Egg survives → Little info gained

Optimal: Balance information gain with egg preservation
```

## 8. References

1. [Wikipedia - Egg dropping puzzle](https://en.wikipedia.org/wiki/Egg_dropping_puzzle)
2. [Google Interview Question Analysis](https://www.geeksforgeeks.org/egg-dropping-puzzle-dp-11/)
