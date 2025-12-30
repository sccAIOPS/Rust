# Factorial

## 1. Overview

The **factorial** of a non-negative integer $n$, denoted $n!$, is the product of all positive integers less than or equal to $n$. Factorial is fundamental in combinatorics, probability theory, and calculus.

**File**: `src/math/factorial.rs`

## 2. Mathematical Foundation

### 2.1 Definition

$$n! = \begin{cases} 1 & \text{if } n = 0 \\ n \times (n-1)! & \text{if } n > 0 \end{cases}$$

Equivalently: $n! = 1 \times 2 \times 3 \times \ldots \times n = \prod_{k=1}^{n} k$

### 2.2 Key Properties

1. **Recurrence**: $n! = n \cdot (n-1)!$
2. **Base case**: $0! = 1$ (empty product)
3. **Growth**: $n! \sim \sqrt{2\pi n}\left(\frac{n}{e}\right)^n$ (Stirling's approximation)
4. **Factorial bound**: $\left(\frac{n}{e}\right)^n < n! < n^n$

### 2.3 Factorial Values

| n | n! |
|---|-----|
| 0 | 1 |
| 1 | 1 |
| 5 | 120 |
| 10 | 3,628,800 |
| 20 | 2,432,902,008,176,640,000 |
| 100 | 9.33 × 10¹⁵⁷ |

**Note**: $20!$ is the largest factorial that fits in a 64-bit unsigned integer.

## 3. Algorithm Description

### 3.1 Implementations

**Iterative** (preferred):
```
function factorial(n):
    return product(1, 2, 3, ..., n)
```

**Recursive**:
```
function factorial(n):
    if n ≤ 1: return 1
    return n × factorial(n-1)
```

### 3.2 Step-by-Step Example

Compute $5!$:

| Step | Accumulator | Multiply by |
|------|-------------|-------------|
| 0 | 1 | - |
| 1 | 1 | 1 |
| 2 | 2 | 2 |
| 3 | 6 | 3 |
| 4 | 24 | 4 |
| 5 | **120** | 5 |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Implementation | Complexity |
|----------------|------------|
| Iterative | O(n) |
| Recursive | O(n) |
| With BigInt | O(n² log n) due to multiplication costs |

### 4.2 Space Complexity

| Implementation | Space |
|----------------|-------|
| Iterative | O(1) |
| Recursive | O(n) stack |
| BigInt | O(n log n) for result |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
// Iterative (recommended)
pub fn factorial(number: u64) -> u64 {
    if number == 0 || number == 1 {
        1
    } else {
        (2..=number).product()
    }
}

// Recursive
pub fn factorial_recursive(n: u64) -> u64 {
    if n == 0 || n == 1 {
        1
    } else {
        n * factorial_recursive(n - 1)
    }
}

// BigInt for large factorials
pub fn factorial_bigmath(num: u32) -> BigUint {
    let mut result: BigUint = One::one();
    for i in 1..=num {
        result *= i;
    }
    result
}
```

### 5.2 Overflow Considerations

| Type | Max n | Max n! |
|------|-------|--------|
| u32 | 12 | 479,001,600 |
| u64 | 20 | 2,432,902,008,176,640,000 |
| u128 | 34 | ~2.95 × 10³⁸ |
| BigUint | Unlimited | Memory-bounded |

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | 1 | By convention |
| 1 | 1 | Base case |
| n > 20 (u64) | Overflow | Use BigUint |

## 6. Applications

### 6.1 Combinatorics

- **Permutations**: $P(n,k) = \frac{n!}{(n-k)!}$
- **Combinations**: $C(n,k) = \frac{n!}{k!(n-k)!}$

### 6.2 Probability

- Multinomial distributions
- Poisson distribution: $P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}$

### 6.3 Calculus

- Taylor series: $e^x = \sum_{n=0}^{\infty} \frac{x^n}{n!}$
- Gamma function: $\Gamma(n+1) = n!$

## 7. Optimizations

### 7.1 Memoization

```rust
fn factorial_memoized(n: u64, cache: &mut HashMap<u64, u64>) -> u64 {
    if let Some(&result) = cache.get(&n) {
        return result;
    }
    let result = if n <= 1 { 1 } else { n * factorial_memoized(n - 1, cache) };
    cache.insert(n, result);
    result
}
```

### 7.2 Lookup Table

For small $n$, precompute:
```rust
const FACTORIALS: [u64; 21] = [
    1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800,
    39916800, 479001600, 6227020800, 87178291200, 1307674368000,
    20922789888000, 355687428096000, 6402373705728000,
    121645100408832000, 2432902008176640000
];
```

## 8. Related Functions

| Function | Definition | Relation to Factorial |
|----------|------------|----------------------|
| [Binomial Coefficient](binomial_coefficient.md) | $\binom{n}{k}$ | Uses factorial |
| [Combinations](combinations.md) | $C(n,k)$ | Uses factorial |
| Gamma Function | $\Gamma(z)$ | $\Gamma(n+1) = n!$ |
| Double Factorial | $n!!$ | Product of same-parity integers |

## 9. References

1. Graham, R. L. et al. "Concrete Mathematics"
2. Knuth, D. E. "The Art of Computer Programming, Vol. 1"
