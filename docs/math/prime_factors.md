# Prime Factorization

## 1. Overview

**Prime factorization** decomposes a positive integer into a product of prime numbers. This fundamental operation in number theory has applications in cryptography, computational algebra, and algorithm design.

**File**: `src/math/prime_factors.rs`

## 2. Mathematical Foundation

### 2.1 Fundamental Theorem of Arithmetic

Every integer $n > 1$ can be uniquely represented as:
$$n = p_1^{e_1} \cdot p_2^{e_2} \cdot \ldots \cdot p_k^{e_k}$$

where $p_1 < p_2 < \ldots < p_k$ are primes and $e_i \geq 1$.

### 2.2 Problem Definition

Given a positive integer $n$, find all prime factors with their multiplicities, in increasing order.

**Output format**: List of primes with repetition: $[p_1, p_1, \ldots, p_2, p_2, \ldots]$

## 3. Algorithm Description

### 3.1 Trial Division

The simplest method: divide by each potential factor starting from 2.

### 3.2 Pseudocode

```
function prime_factors(n):
    factors ← []
    i ← 2
    
    while i × i ≤ n:
        while n mod i = 0:
            factors.append(i)
            n ← n / i
        if i = 2:
            i ← 3
        else:
            i ← i + 2
    
    if n > 1:
        factors.append(n)  # n is prime
    
    return factors
```

### 3.3 Step-by-Step Example

Factorize $n = 2560$:

| Step | n | i | Check | Action |
|------|---|---|-------|--------|
| 1 | 2560 | 2 | 2560 % 2 = 0 | factors = [2], n = 1280 |
| 2 | 1280 | 2 | 1280 % 2 = 0 | factors = [2,2], n = 640 |
| 3 | 640 | 2 | 640 % 2 = 0 | factors = [2,2,2], n = 320 |
| ... | ... | 2 | Continue... | ... |
| 9 | 5 | 2 | 5 % 2 ≠ 0 | i = 3 |
| 10 | 5 | 3 | 3² > 5 | Exit loop |
| 11 | 5 | - | n > 1 | factors.append(5) |

**Result**: [2, 2, 2, 2, 2, 2, 2, 2, 2, 5]

Verification: $2^9 \times 5 = 512 \times 5 = 2560$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Example |
|------|------------|---------|
| Best | O(log n) | n is a power of 2 |
| Average | O(√n) | Random composite |
| Worst | O(√n) | n is prime |

### 4.2 Space Complexity

O(log n) - maximum number of prime factors.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn prime_factors(n: u64) -> Vec<u64> {
    let mut i = 2;
    let mut n = n;
    let mut factors = Vec::new();
    
    while i * i <= n {
        if n.is_multiple_of(i) {
            n /= i;
            factors.push(i);
        } else {
            if i != 2 {
                i += 1;
            }
            i += 1;
        }
    }
    if n > 1 {
        factors.push(n);
    }
    factors
}
```

**Key patterns**:
- Uses `is_multiple_of` trait
- Skips even numbers after 2
- Handles remaining prime factor

### 5.2 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | [] | Convention |
| 1 | [] | 1 has no prime factors |
| 2 | [2] | Smallest prime |
| Prime p | [p] | Single factor |
| p² | [p, p] | Perfect square of prime |

## 6. Optimizations

### 6.1 Wheel Factorization

Skip multiples of 2, 3:
```rust
// Check 2, 3, then 6k±1
let wheel = [1, 2, 2, 4, 2, 4, 2, 4, 6, 2, 6];
```

### 6.2 Pre-computed Primes

```rust
fn factorize_with_primes(mut n: u64, primes: &[u64]) -> Vec<u64> {
    let mut factors = Vec::new();
    for &p in primes {
        if p * p > n { break; }
        while n % p == 0 {
            factors.push(p);
            n /= p;
        }
    }
    if n > 1 { factors.push(n); }
    factors
}
```

### 6.3 Using Linear Sieve

For multiple factorizations, precompute SPF:
```rust
// O(log n) per factorization after O(N) precomputation
fn factorize_fast(mut n: usize, spf: &[usize]) -> Vec<usize> {
    let mut factors = Vec::new();
    while n > 1 {
        factors.push(spf[n]);
        n /= spf[n];
    }
    factors
}
```

## 7. Applications

### 7.1 Computing Number-Theoretic Functions

```rust
// Number of divisors: d(n) = Π(e_i + 1)
fn num_divisors(factors: &[u64]) -> u64 {
    let mut count = 1;
    let mut prev = 0;
    let mut exp = 0;
    for &f in factors {
        if f == prev {
            exp += 1;
        } else {
            count *= exp + 1;
            prev = f;
            exp = 1;
        }
    }
    count * (exp + 1)
}
```

### 7.2 Use Cases

1. **Cryptanalysis**: Breaking RSA when factors are found
2. **Computing GCD/LCM**: Alternative to Euclidean algorithm
3. **Simplifying fractions**: Finding common factors
4. **Modular arithmetic**: Computing totient φ(n)

## 8. Related Algorithms

| Algorithm | Use Case | Complexity |
|-----------|----------|------------|
| Trial Division | Small numbers | O(√n) |
| [Pollard's Rho](pollard_rho.md) | Large semi-primes | O(n^1/4) |
| [Linear Sieve](linear_sieve.md) | Multiple factorizations | O(n) preprocess, O(log n) query |
| Quadratic Sieve | Very large numbers | Sub-exponential |

## 9. References

1. Hardy, G. H. & Wright, E. M. "An Introduction to the Theory of Numbers"
2. Crandall, R. & Pomerance, C. "Prime Numbers: A Computational Perspective"
