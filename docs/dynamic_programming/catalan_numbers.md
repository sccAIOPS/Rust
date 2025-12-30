# Catalan Numbers

## 1. Overview

Catalan numbers form a sequence of natural numbers with many applications in combinatorics. The n-th Catalan number counts objects like valid parentheses sequences, binary trees with n+1 leaves, and paths that don't cross the diagonal.

**File**: `src/dynamic_programming/catalan_numbers.rs`

## 2. Mathematical Foundation

### 2.1 Definition

The n-th Catalan number is:

$$C_n = \frac{1}{n+1}\binom{2n}{n} = \frac{(2n)!}{(n+1)!n!}$$

### 2.2 Recurrence

$$C_0 = 1$$
$$C_n = \sum_{i=0}^{n-1} C_i \cdot C_{n-1-i}$$

Or equivalently:
$$C_{n+1} = \frac{2(2n+1)}{n+2} C_n$$

### 2.3 First 10 Values

| n | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|---|
| $C_n$ | 1 | 1 | 2 | 5 | 14 | 42 | 132 | 429 | 1430 | 4862 |

## 3. Algorithm Description

### 3.1 DP Approach

Build up Catalan numbers using the recurrence relation.

### 3.2 Pseudocode

```
FUNCTION catalan(n)
    dp[0..n] ← 0
    dp[0] ← 1
    
    FOR i ← 1 TO n DO
        FOR j ← 0 TO i-1 DO
            dp[i] ← dp[i] + dp[j] * dp[i-1-j]
        END FOR
    END FOR
    
    RETURN dp[n]
END FUNCTION
```

### 3.3 Step-by-Step Example (n=4)

```
dp[0] = 1 (base case)

dp[1] = dp[0] * dp[0] = 1 * 1 = 1

dp[2] = dp[0] * dp[1] + dp[1] * dp[0]
      = 1 * 1 + 1 * 1 = 2

dp[3] = dp[0] * dp[2] + dp[1] * dp[1] + dp[2] * dp[0]
      = 1 * 2 + 1 * 1 + 2 * 1 = 5

dp[4] = dp[0] * dp[3] + dp[1] * dp[2] + dp[2] * dp[1] + dp[3] * dp[0]
      = 1 * 5 + 1 * 2 + 2 * 1 + 5 * 1 = 14
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n²)** for computing C_n using DP
- O(1) for O(n log n) formula with modular arithmetic

### 4.2 Space Complexity
- **O(n)** for storing previous values

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn catalan(num: u128) -> u128 {
    if num <= 1 {
        return 1;
    }
    let mut catalan_num = vec![0u128; (num + 1) as usize];
    catalan_num[0] = 1;
    catalan_num[1] = 1;

    for i in 2..=num as usize {
        for j in 0..i {
            catalan_num[i] += catalan_num[j] * catalan_num[i - 1 - j];
        }
    }
    catalan_num[num as usize]
}
```

### 5.2 Overflow Handling

**Warning**: Catalan numbers grow rapidly!

| n | $C_n$ | Bits needed |
|---|-------|-------------|
| 10 | 16,796 | 15 |
| 20 | 6,564,120,420 | 33 |
| 30 | ~5.9 × 10¹⁵ | 53 |
| 36 | > u64::MAX | 65+ |

Implementation uses `u128` to handle larger values.

## 6. Applications

### 6.1 Classic Catalan Objects

| Object | $C_n$ counts |
|--------|--------------|
| Parentheses | Valid sequences of n pairs |
| Binary Trees | Trees with n+1 leaves |
| Mountain Ranges | Paths with n ups and n downs |
| Non-crossing Partitions | Partitions of {1,...,n} |
| Polygon Triangulations | Ways to triangulate (n+2)-gon |
| Dyck Paths | Paths from (0,0) to (2n,0) above x-axis |

### 6.2 Real-World Uses

1. **Compiler Design**: Expression tree generation
2. **Data Structures**: BST enumeration
3. **Algorithm Analysis**: Recursion tree counting

## 7. Bijection Examples

### 7.1 Parentheses ↔ Binary Trees (n=3)

```
((()))  ↔  Tree: right-skewed
(()())  ↔  Tree: balanced
(())()  ↔  Tree: left-right
()(())  ↔  Tree: right-left
()()()  ↔  Tree: linear
```

All 5 = $C_3$ objects!

## 8. Alternative Formulas

```
Formula 1: C_n = binomial(2n, n) / (n + 1)
Formula 2: C_n = binomial(2n, n) - binomial(2n, n+1)
Formula 3: C_{n+1} = (2 * (2n + 1) * C_n) / (n + 2)
```

## 9. Related Sequences

| Sequence | Definition |
|----------|------------|
| Motzkin | Paths with steps up, down, horizontal |
| Narayana | Catalan refinement by peak count |
| Ballot | Generalized voting problem |

## 10. References

1. [OEIS A000108](https://oeis.org/A000108)
2. [Wikipedia - Catalan number](https://en.wikipedia.org/wiki/Catalan_number)
3. Stanley, R.P. "Catalan Numbers"
