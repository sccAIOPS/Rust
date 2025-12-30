# Integer Partition

## 1. Overview

The Integer Partition problem counts the number of ways to write a positive integer as a sum of positive integers, where order doesn't matter. This is a fundamental problem in combinatorics and number theory.

**File**: `src/dynamic_programming/integer_partition.rs`

## 2. Mathematical Foundation

### 2.1 Definition

A partition of integer $n$ is a multiset of positive integers that sum to $n$.

Example: Partitions of 4:
- 4
- 3 + 1
- 2 + 2
- 2 + 1 + 1
- 1 + 1 + 1 + 1

Total: $p(4) = 5$

### 2.2 Notation

- $p(n)$ = number of partitions of $n$
- $p(n, k)$ = partitions of $n$ using parts ≤ $k$
- $p(n)$ = $p(n, n)$

### 2.3 Generating Function

$$\sum_{n=0}^{\infty} p(n) x^n = \prod_{k=1}^{\infty} \frac{1}{1-x^k}$$

### 2.4 Recurrence

$$p(n, k) = p(n, k-1) + p(n-k, k)$$

Interpretation:
- $p(n, k-1)$: partitions not using $k$
- $p(n-k, k)$: partitions using at least one $k$

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION partition_count(n)
    dp[0..n+1][0..n+1] ← 0
    
    // Base case: one way to partition 0
    FOR k ← 0 TO n DO
        dp[0][k] ← 1
    END FOR
    
    FOR i ← 1 TO n DO
        FOR k ← 1 TO n DO
            dp[i][k] ← dp[i][k-1]
            IF i >= k THEN
                dp[i][k] ← dp[i][k] + dp[i-k][k]
            END IF
        END FOR
    END FOR
    
    RETURN dp[n][n]
END FUNCTION
```

### 3.2 Space-Optimized Version

```
FUNCTION partition_count_optimized(n)
    dp[0..n+1] ← 0
    dp[0] ← 1
    
    FOR k ← 1 TO n DO
        FOR i ← k TO n DO
            dp[i] ← dp[i] + dp[i-k]
        END FOR
    END FOR
    
    RETURN dp[n]
END FUNCTION
```

### 3.3 Step-by-Step Example (n=5)

**Using 2D DP**:
```
     k=0  k=1  k=2  k=3  k=4  k=5
n=0   1    1    1    1    1    1   (one way to partition 0)
n=1   0    1    1    1    1    1   (1)
n=2   0    1    2    2    2    2   (2, 1+1)
n=3   0    1    2    3    3    3   (3, 2+1, 1+1+1)
n=4   0    1    3    4    5    5   (4, 3+1, 2+2, 2+1+1, 1+1+1+1)
n=5   0    1    3    5    6    7   

p(5) = 7: {5, 4+1, 3+2, 3+1+1, 2+2+1, 2+1+1+1, 1+1+1+1+1}
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n²)** for DP approach

### 4.2 Space Complexity
- **O(n²)** with 2D table
- **O(n)** with space optimization

### 4.3 Hardy-Ramanujan-Rademacher Formula

For large $n$, there's an O(n^(1/2 + ε)) exact formula:

$$p(n) \sim \frac{1}{4n\sqrt{3}} \exp\left(\pi\sqrt{\frac{2n}{3}}\right)$$

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn partition(num: u64) -> u64 {
    let n = num as usize;
    let mut dp = vec![0u64; n + 1];
    dp[0] = 1;
    
    for k in 1..=n {
        for i in k..=n {
            dp[i] += dp[i - k];
        }
    }
    
    dp[n]
}
```

### 5.2 Edge Cases

| n | p(n) | Partitions |
|---|------|------------|
| 0 | 1 | {} (empty partition) |
| 1 | 1 | {1} |
| 2 | 2 | {2, 1+1} |
| 5 | 7 | See above |
| 10 | 42 | |
| 100 | 190,569,292 | |

### 5.3 Overflow Handling

Partition numbers grow fast:
- $p(100) \approx 1.9 \times 10^8$
- $p(500) \approx 2.3 \times 10^{21}$
- $p(1000) \approx 2.4 \times 10^{31}$

Use `u128` or arbitrary precision for large $n$.

## 6. Restricted Partitions

### 6.1 Distinct Parts

Each part appears at most once:

```rust
fn distinct_partition(n: usize) -> u64 {
    let mut dp = vec![0u64; n + 1];
    dp[0] = 1;
    
    for k in 1..=n {
        for i in (k..=n).rev() {  // Reverse to ensure each k used once
            dp[i] += dp[i - k];
        }
    }
    
    dp[n]
}
```

### 6.2 Odd Parts Only

```rust
fn odd_partition(n: usize) -> u64 {
    let mut dp = vec![0u64; n + 1];
    dp[0] = 1;
    
    for k in (1..=n).step_by(2) {
        for i in k..=n {
            dp[i] += dp[i - k];
        }
    }
    
    dp[n]
}
```

**Euler's Identity**: Partitions into distinct parts = Partitions into odd parts!

### 6.3 At Most k Parts

$$p(n, \leq k \text{ parts}) = p(n, k)$$ (by conjugate symmetry)

## 7. Partition Function Properties

| Property | Formula |
|----------|---------|
| Conjugate | Transpose Young diagram |
| Self-conjugate | Equal to distinct odd parts |
| Rank | Largest part minus number of parts |
| Crank | Largest part minus # parts > largest |

## 8. Applications

1. **Combinatorics**: Counting problems
2. **Physics**: Bosonic string theory, statistical mechanics
3. **Number Theory**: Modular forms, q-series
4. **Chemistry**: Molecular orbital configurations
5. **Computer Science**: Bin packing analysis

## 9. Generating All Partitions

```rust
fn generate_partitions(n: usize) -> Vec<Vec<usize>> {
    let mut result = Vec::new();
    let mut current = Vec::new();
    
    fn backtrack(remaining: usize, max_part: usize, 
                 current: &mut Vec<usize>, result: &mut Vec<Vec<usize>>) {
        if remaining == 0 {
            result.push(current.clone());
            return;
        }
        
        for part in (1..=remaining.min(max_part)).rev() {
            current.push(part);
            backtrack(remaining - part, part, current, result);
            current.pop();
        }
    }
    
    backtrack(n, n, &mut current, &mut result);
    result
}
```

## 10. Young Diagrams

Visual representation of partitions:

```
Partition 5 = 3 + 2:

□ □ □
□ □

Partition 5 = 2 + 2 + 1:

□ □
□ □
□
```

Conjugate partition: Transpose the diagram!

## 11. Famous Results

### Euler's Pentagonal Theorem

$$\prod_{n=1}^{\infty}(1-x^n) = \sum_{k=-\infty}^{\infty}(-1)^k x^{k(3k-1)/2}$$

### Ramanujan's Congruences

- $p(5n + 4) \equiv 0 \pmod{5}$
- $p(7n + 5) \equiv 0 \pmod{7}$
- $p(11n + 6) \equiv 0 \pmod{11}$

## 12. References

1. [OEIS A000041](https://oeis.org/A000041) - Partition numbers
2. Hardy, G.H. & Wright, E.M. "An Introduction to the Theory of Numbers"
3. Andrews, G.E. "The Theory of Partitions"
4. [Wikipedia - Partition (number theory)](https://en.wikipedia.org/wiki/Partition_(number_theory))
