# Binomial Coefficient

## 1. Overview

The **binomial coefficient** $\binom{n}{k}$ (read "n choose k") counts the number of ways to choose $k$ elements from a set of $n$ elements. It's fundamental in combinatorics, probability, and the binomial theorem.

**File**: `src/math/binomial_coefficient.rs`

## 2. Mathematical Foundation

### 2.1 Definition

$$\binom{n}{k} = \frac{n!}{k!(n-k)!}$$

For non-negative integers $n$ and $k$ with $0 \leq k \leq n$.

By convention: $\binom{n}{k} = 0$ when $k > n$ or $k < 0$.

### 2.2 Key Properties

1. **Symmetry**: $\binom{n}{k} = \binom{n}{n-k}$
2. **Pascal's Rule**: $\binom{n}{k} = \binom{n-1}{k-1} + \binom{n-1}{k}$
3. **Sum**: $\sum_{k=0}^{n} \binom{n}{k} = 2^n$
4. **Hockey Stick**: $\sum_{i=r}^{n} \binom{i}{r} = \binom{n+1}{r+1}$

### 2.3 Binomial Theorem

$$(x + y)^n = \sum_{k=0}^{n} \binom{n}{k} x^{n-k} y^k$$

## 3. Algorithm Description

### 3.1 Computation Without Factorial

To avoid large intermediate values:

$$\binom{n}{k} = \frac{n \cdot (n-1) \cdot \ldots \cdot (n-k+1)}{k!} = \prod_{i=0}^{k-1} \frac{n-i}{i+1}$$

### 3.2 Pseudocode

```
function binom(n, k):
    result ← 1
    for i from 0 to k-1:
        result ← result × (n - i)
        result ← result / (i + 1)
    return result
```

### 3.3 Step-by-Step Example

Compute $\binom{10}{3}$:

| i | n - i | i + 1 | result |
|---|-------|-------|--------|
| 0 | 10 | 1 | 1 × 10 / 1 = 10 |
| 1 | 9 | 2 | 10 × 9 / 2 = 45 |
| 2 | 8 | 3 | 45 × 8 / 3 = **120** |

## 4. Complexity Analysis

### 4.1 Time Complexity

O(min(k, n-k)) - iterate through the smaller of k or n-k.

### 4.2 Space Complexity

O(1) for single computation, O(n²) for Pascal's triangle.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
use num_bigint::BigInt;
use num_traits::FromPrimitive;

pub fn binom(n: u64, k: u64) -> BigInt {
    let mut res = BigInt::from_u64(1).unwrap();
    for i in 0..k {
        res = (res * BigInt::from_u64(n - i).unwrap()) 
            / BigInt::from_u64(i + 1).unwrap();
    }
    res
}
```

**Key design**:
- Uses BigInt to handle large results
- Alternates multiplication and division to minimize intermediate values
- Division is exact due to divisibility properties

### 5.2 Integer-Only Version

```rust
pub fn binom_u64(n: u64, k: u64) -> u64 {
    let k = k.min(n - k);  // Use symmetry
    let mut result = 1u64;
    for i in 0..k {
        result = result * (n - i) / (i + 1);
    }
    result
}
```

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| (n, 0) | 1 | Empty selection |
| (n, n) | 1 | Full selection |
| (n, k) where k > n | 0 | Impossible |
| (0, 0) | 1 | Base case |

## 6. Pascal's Triangle

```
            1           n=0
           1 1          n=1
          1 2 1         n=2
         1 3 3 1        n=3
        1 4 6 4 1       n=4
       1 5 10 10 5 1    n=5
```

Each entry is the sum of the two entries above it.

## 7. Applications

### 7.1 Combinatorics

- Counting subsets of size k
- Lattice paths (Catalan numbers)
- Voting paradoxes

### 7.2 Probability

- Binomial distribution: $P(X=k) = \binom{n}{k} p^k (1-p)^{n-k}$
- Hypergeometric distribution

### 7.3 Computer Science

- Generating combinations
- Error-correcting codes
- Algorithm analysis

## 8. Related Algorithms

| Algorithm | Relation |
|-----------|----------|
| [Factorial](factorial.md) | Definition component |
| [Combinations](combinations.md) | Same computation |
| [Pascal's Triangle](pascal_triangle.md) | Visual representation |
| [Catalan Numbers](catalan_numbers.md) | Uses binomial coefficients |

## 9. References

1. Graham, R. L. et al. "Concrete Mathematics"
2. [Wikipedia: Binomial Coefficient](https://en.wikipedia.org/wiki/Binomial_coefficient)
