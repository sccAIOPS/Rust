# Number Theory Algorithms

Number theory is a branch of pure mathematics devoted to the study of integers and integer-valued functions. This module contains implementations of fundamental number-theoretic algorithms with applications in cryptography, computer science, and mathematical research.

## Overview

Number theory algorithms form the backbone of modern cryptography and are essential in many computational problems. The algorithms in this module focus on divisibility, prime numbers, and multiplicative functions.

## Algorithms

| Algorithm | Description | Time Complexity | Space Complexity |
|-----------|-------------|-----------------|------------------|
| [Euler's Totient](euler_totient.md) | Counts integers coprime to n | O(√n) | O(1) |
| [Compute Totient](compute_totient.md) | Sieve-based totient for range [1,n] | O(n log log n) | O(n) |
| [Kth Factor](kth_factor.md) | Finds the kth smallest divisor of n | O(n) | O(k) |

## Key Concepts

### Euler's Totient Function φ(n)

The totient function counts integers in [1, n] that are coprime to n:

$$\phi(n) = n \prod_{p|n}\left(1 - \frac{1}{p}\right)$$

**Properties:**
- φ(1) = 1
- φ(p) = p - 1 for prime p
- φ(mn) = φ(m)φ(n) when gcd(m,n) = 1 (multiplicative)

### Divisor Functions

- **τ(n):** Number of positive divisors of n
- **σ(n):** Sum of positive divisors of n
- **d | n:** d divides n (n mod d = 0)

## Applications

### Cryptography
- **RSA Algorithm:** Uses φ(n) where n = pq for key generation
- **Euler's Theorem:** a^φ(n) ≡ 1 (mod n) enables efficient modular exponentiation

### Competitive Programming
- Counting coprime pairs
- Finding divisors efficiently
- Modular arithmetic problems

### Mathematical Research
- Studying prime distribution
- Analyzing multiplicative functions
- Number-theoretic transforms

## Usage Examples

### Computing Single Totient Value

```rust
use the_algorithms_rust::number_theory::euler_totient;

// φ(12) = 4 (coprime numbers: 1, 5, 7, 11)
let phi_12 = euler_totient(12);
assert_eq!(phi_12, 4);

// φ(prime) = prime - 1
let phi_13 = euler_totient(13);
assert_eq!(phi_13, 12);
```

### Computing Totient for Range

```rust
use the_algorithms_rust::number_theory::compute_totient;

// Get φ(1), φ(2), ..., φ(10)
let totients = compute_totient(10);
// [1, 1, 2, 2, 4, 2, 6, 4, 6, 4]
```

### Finding Kth Factor

```rust
use the_algorithms_rust::number_theory::kth_factor;

// Factors of 12: [1, 2, 3, 4, 6, 12]
let third_factor = kth_factor(12, 3);
assert_eq!(third_factor, 3);

// 7 is prime, only 2 factors: [1, 7]
let second_factor = kth_factor(7, 2);
assert_eq!(second_factor, 7);
```

## Algorithm Selection Guide

| Scenario | Recommended Algorithm |
|----------|----------------------|
| Single φ(n) query | `euler_totient` |
| Multiple φ queries for small range | `compute_totient` |
| Finding specific divisor | `kth_factor` |
| Checking primality | Use `math/prime_check` |
| Finding all prime factors | Use `math/prime_factors` |

## Complexity Summary

```
┌─────────────────────────────────────────────────────┐
│               Time Complexity Chart                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  euler_totient(n)     │████████░░░░░░░│  O(√n)     │
│  compute_totient(n)   │████████████░░░│  O(n log log n) │
│  kth_factor(n, k)     │███████████████│  O(n)      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## Related Modules

- [`math/`](../math/README.md) - General mathematical algorithms
- [`ciphers/`](../ciphers/README.md) - Cryptographic algorithms using number theory
- [`big_integer/`](../big_integer/README.md) - Large number operations

## References

1. Hardy, G.H. & Wright, E.M. "An Introduction to the Theory of Numbers"
2. Rosen, K.H. "Elementary Number Theory and Its Applications"
3. [OEIS A000010](https://oeis.org/A000010) - Euler's totient sequence
4. [CP-Algorithms](https://cp-algorithms.com/algebra/phi-function.html) - Euler's phi function
