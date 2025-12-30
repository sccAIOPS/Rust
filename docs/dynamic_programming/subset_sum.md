# Subset Sum Problem

## 1. Overview

The Subset Sum problem determines whether there exists a subset of a given array that sums to a target value. This is a classic NP-complete problem solvable in pseudo-polynomial time using dynamic programming.

**File**: `src/dynamic_programming/subset_sum.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- Array $A = \{a_1, a_2, \ldots, a_n\}$
- Target sum $S$

Determine if there exists $I \subseteq \{1, 2, \ldots, n\}$ such that:
$$\sum_{i \in I} a_i = S$$

### 2.2 Recurrence

Let $dp[i][j]$ = true if sum $j$ is achievable using first $i$ elements.

$$dp[i][j] = \begin{cases}
\text{true} & \text{if } j = 0 \\
\text{false} & \text{if } i = 0 \text{ and } j > 0 \\
dp[i-1][j] & \text{if } a_i > j \\
dp[i-1][j] \vee dp[i-1][j-a_i] & \text{otherwise}
\end{cases}$$

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION is_sum_subset(arr, target)
    IF target = 0 THEN RETURN true
    IF arr is empty THEN RETURN false
    
    n ← length(arr)
    dp[0..n][0..target] ← false
    
    FOR i ← 0 TO n DO
        dp[i][0] ← true  // Sum 0 always achievable
    END FOR
    
    FOR i ← 1 TO n DO
        FOR j ← 1 TO target DO
            IF arr[i-1] > j THEN
                dp[i][j] ← dp[i-1][j]
            ELSE
                dp[i][j] ← dp[i-1][j] OR dp[i-1][j - arr[i-1]]
            END IF
        END FOR
    END FOR
    
    RETURN dp[n][target]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: arr = [3, 34, 4, 12, 5, 2], target = 9

**DP Table** (partial):

| Elem\\Sum | 0 | 1 | 2 | 3 | 4 | 5 | ... | 9 |
|-----------|---|---|---|---|---|---|-----|---|
| {} | T | F | F | F | F | F | ... | F |
| {3} | T | F | F | T | F | F | ... | F |
| {3,34} | T | F | F | T | F | F | ... | F |
| {3,34,4} | T | F | F | T | T | F | ... | F |
| {3,34,4,12} | T | F | F | T | T | F | ... | F |
| {3,34,4,12,5} | T | F | F | T | T | T | ... | **T** |

**Result**: true (subset {4, 5} or {3, 4, 2})

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n × sum)** - pseudo-polynomial

### 4.2 Space Complexity
- **O(n × sum)** - can be optimized to O(sum)

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn is_sum_subset(arr: &[i32], required_sum: i32) -> bool {
    if required_sum == 0 { return true; }
    if arr.is_empty() && required_sum > 0 { return false; }

    let n = arr.len();
    let mut dp = vec![vec![false; required_sum as usize + 1]; n + 1];

    for i in 0..=n {
        dp[i][0] = true;
    }

    for i in 1..=n {
        for j in 1..=required_sum as usize {
            if arr[i - 1] > j as i32 {
                dp[i][j] = dp[i - 1][j];
            } else {
                dp[i][j] = dp[i-1][j] || dp[i-1][j - arr[i-1] as usize];
            }
        }
    }

    dp[n][required_sum as usize]
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Target = 0 | true |
| Empty array, target > 0 | false |
| Single element = target | true |
| All elements > target | false |

## 6. NP-Completeness

The subset sum problem is:
- **NP-complete** in the strong sense
- **Pseudo-polynomial** solvable when sum is polynomial in input size
- Basis for cryptographic systems

## 7. Applications

1. **Cryptography**: Basis for some public-key systems
2. **Load Balancing**: Partitioning tasks
3. **Resource Allocation**: Budget matching

## 8. Variants

| Variant | Description |
|---------|-------------|
| Count subsets | Number of subsets with target sum |
| Partition problem | Split into two equal-sum halves |
| Bounded subset sum | Each element has limited count |

## 9. References

1. [Wikipedia - Subset sum problem](https://en.wikipedia.org/wiki/Subset_sum_problem)
2. Garey, M. R., & Johnson, D. S. (1979). *Computers and Intractability*
