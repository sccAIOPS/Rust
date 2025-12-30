# Prime Check

## 1. Overview

**Prime checking** (primality testing) determines whether a given number is prime. A prime number is a natural number greater than 1 that has no positive divisors other than 1 and itself.

**File**: `src/math/prime_check.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer $n$, determine whether $n$ is prime.

**Definition**: $n$ is prime iff $n > 1$ and:
$$\forall d \in \mathbb{Z}^+, 1 < d < n \implies d \nmid n$$

### 2.2 Key Theorem

**Theorem**: If $n$ is composite, then $n$ has a factor $d$ where $1 < d \leq \sqrt{n}$.

**Proof**: If $n = ab$ with $1 < a \leq b < n$, then $a^2 \leq ab = n$, so $a \leq \sqrt{n}$.

### 2.3 Trial Division Optimization

Only need to check:
1. Divisibility by 2
2. Odd numbers from 3 to $\sqrt{n}$

Further optimization: Check only numbers of form $6k \pm 1$ (all primes > 3 have this form).

## 3. Algorithm Description

### 3.1 Intuition

Test divisibility starting from smallest potential factors up to the square root of $n$. If any factor is found, $n$ is composite; otherwise, it's prime.

### 3.2 Pseudocode

```
function is_prime(n):
    if n < 2:
        return false
    if n < 4:
        return true
    if n is even:
        return false
    
    for i from 3 to √n step 2:
        if n mod i = 0:
            return false
    
    return true
```

### 3.3 Step-by-Step Example

Check if $n = 97$ is prime:

| Step | Divisor | Check | Result |
|------|---------|-------|--------|
| 1 | 2 | 97 is odd | Continue |
| 2 | 3 | 97 mod 3 = 1 | Continue |
| 3 | 5 | 97 mod 5 = 2 | Continue |
| 4 | 7 | 97 mod 7 = 6 | Continue |
| 5 | 9 | 97 mod 9 = 7 | Continue |
| 6 | √97 ≈ 9.8 | Loop ends | **Prime** |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Notes |
|------|------------|-------|
| Best | O(1) | n < 4 or n is even |
| Average | O(√n) | Must check up to √n |
| Worst | O(√n) | n is prime |

### 4.2 Space Complexity

O(1) - only constant extra space needed.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn prime_check(num: usize) -> bool {
    if (num > 1) & (num < 4) {
        return true;
    } else if (num < 2) || (num.is_multiple_of(2)) {
        return false;
    }

    let stop: usize = (num as f64).sqrt() as usize + 1;
    for i in (3..stop).step_by(2) {
        if num.is_multiple_of(i) {
            return false;
        }
    }
    true
}
```

**Key patterns**:
- Uses `is_multiple_of` trait method
- Computes square root once
- Steps by 2 to skip even numbers
- The `+1` handles floating-point precision issues

### 5.2 Edge Cases

| Input | Output | Reason |
|-------|--------|--------|
| 0 | false | 0 is not prime |
| 1 | false | 1 is not prime by definition |
| 2 | true | Smallest and only even prime |
| 3 | true | Prime |
| 4 | false | 4 = 2² |
| Large prime | true | e.g., 2003 |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Cryptography**: Generating keys for RSA, DSA
2. **Hash Tables**: Prime table sizes for better distribution
3. **Random Number Generation**: Prime moduli for LCGs
4. **Mathematical Research**: Finding new primes

### 6.2 Limitations

Trial division is impractical for very large numbers. For cryptographic applications, use:
- **Miller-Rabin** for probabilistic testing
- **AKS** for deterministic polynomial-time testing

## 7. Related Algorithms

| Algorithm | Use Case | Complexity |
|-----------|----------|------------|
| [Sieve of Eratosthenes](sieve_of_eratosthenes.md) | Find all primes up to n | O(n log log n) |
| [Miller-Rabin](miller_rabin.md) | Large number primality | O(k log³n) |
| [Prime Factorization](prime_factors.md) | Find all prime factors | O(√n) |

## 8. References

1. Hardy, G. H. & Wright, E. M. "An Introduction to the Theory of Numbers"
2. [Wikipedia: Primality Test](https://en.wikipedia.org/wiki/Primality_test)
