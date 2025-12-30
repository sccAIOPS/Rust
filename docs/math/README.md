# Mathematical Algorithms

This directory contains comprehensive documentation for mathematical algorithms implemented in the Rust Algorithms repository.

## Overview

Mathematical algorithms form the backbone of computational science, cryptography, and numerical analysis. This collection includes algorithms ranging from basic number theory to advanced numerical methods and machine learning activation functions.

## Algorithm Categories

### Number Theory
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Greatest Common Divisor (GCD) | `greatest_common_divisor.rs` | [gcd.md](gcd.md) | O(log min(a,b)) |
| Extended Euclidean Algorithm | `extended_euclidean_algorithm.rs` | [extended_euclidean.md](extended_euclidean.md) | O(log min(a,b)) |
| Least Common Multiple (LCM) | `lcm_of_n_numbers.rs` | [lcm.md](lcm.md) | O(n log M) |
| Prime Check | `prime_check.rs` | [prime_check.md](prime_check.md) | O(√n) |
| Prime Numbers | `prime_numbers.rs` | [prime_numbers.md](prime_numbers.md) | O(n√n) |
| Sieve of Eratosthenes | `sieve_of_eratosthenes.rs` | [sieve_of_eratosthenes.md](sieve_of_eratosthenes.md) | O(n log log n) |
| Linear Sieve | `linear_sieve.rs` | [linear_sieve.md](linear_sieve.md) | O(n) |
| Miller-Rabin Primality Test | `miller_rabin.rs` | [miller_rabin.md](miller_rabin.md) | O(k log³n) |
| Pollard's Rho Factorization | `pollard_rho.rs` | [pollard_rho.md](pollard_rho.md) | O(n^(1/4)) |
| Prime Factors | `prime_factors.rs` | [prime_factors.md](prime_factors.md) | O(√n) |
| Chinese Remainder Theorem | `chinese_remainder_theorem.rs` | [chinese_remainder_theorem.md](chinese_remainder_theorem.md) | O(n log M) |
| Euler's Totient Function | *in number_theory* | [euler_totient.md](euler_totient.md) | O(√n) |

### Combinatorics
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Factorial | `factorial.rs` | [factorial.md](factorial.md) | O(n) |
| Binomial Coefficient | `binomial_coefficient.rs` | [binomial_coefficient.md](binomial_coefficient.md) | O(k) |
| Combinations | `combinations.rs` | [combinations.md](combinations.md) | O(k) |
| Pascal's Triangle | `pascal_triangle.rs` | [pascal_triangle.md](pascal_triangle.md) | O(n²) |
| Catalan Numbers | `catalan_numbers.rs` | [catalan_numbers.md](catalan_numbers.md) | O(n) |
| Bell Numbers | `bell_numbers.rs` | [bell_numbers.md](bell_numbers.md) | O(n²) |

### Exponentiation & Modular Arithmetic
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Modular Exponentiation | `modular_exponential.rs` | [modular_exponential.md](modular_exponential.md) | O(log n) |
| Fast Power | `fast_power.rs` | [fast_power.md](fast_power.md) | O(log n) |
| Binary Exponentiation | `binary_exponentiation.rs` | [binary_exponentiation.md](binary_exponentiation.md) | O(log n) |

### Multiplication Algorithms
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Karatsuba Multiplication | `karatsuba_multiplication.rs` | [karatsuba_multiplication.md](karatsuba_multiplication.md) | O(n^1.585) |
| Fast Fourier Transform | `fast_fourier_transform.rs` | [fast_fourier_transform.md](fast_fourier_transform.md) | O(n log n) |

### Numerical Methods
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Newton-Raphson Method | `newton_raphson.rs` | [newton_raphson.md](newton_raphson.md) | Quadratic convergence |
| Square Root | `square_root.rs` | [square_root.md](square_root.md) | O(log n) iterations |
| Gaussian Elimination | `gaussian_elimination.rs` | [gaussian_elimination.md](gaussian_elimination.md) | O(n³) |
| Trapezoidal Integration | `trapezoidal_integration.rs` | [trapezoidal_integration.md](trapezoidal_integration.md) | O(n) |
| Simpson's Integration | `simpsons_integration.rs` | [simpsons_integration.md](simpsons_integration.md) | O(n) |
| Interpolation | `interpolation.rs` | [interpolation.md](interpolation.md) | O(n²) |
| Least Squares Approximation | `least_square_approx.rs` | [least_square_approx.md](least_square_approx.md) | O(n) |

### Matrix Operations
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Matrix Operations | `matrix_ops.rs` | [matrix_ops.md](matrix_ops.md) | O(n²) - O(n³) |

### Activation Functions (Machine Learning)
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Sigmoid | `sigmoid.rs` | [sigmoid.md](sigmoid.md) | O(n) |
| ReLU | `relu.rs` | [relu.md](relu.md) | O(n) |
| Leaky ReLU | `leaky_relu.rs` | [leaky_relu.md](leaky_relu.md) | O(n) |
| Softmax | `softmax.rs` | [softmax.md](softmax.md) | O(n) |
| Tanh | `tanh.rs` | [tanh.md](tanh.md) | O(n) |

### Loss Functions
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Cross Entropy Loss | `cross_entropy_loss.rs` | [cross_entropy_loss.md](cross_entropy_loss.md) | O(n) |
| Huber Loss | `huber_loss.rs` | [huber_loss.md](huber_loss.md) | O(n) |

### Number Properties
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Perfect Numbers | `perfect_numbers.rs` | [perfect_numbers.md](perfect_numbers.md) | O(√n) |
| Armstrong Number | `armstrong_number.rs` | [armstrong_number.md](armstrong_number.md) | O(log n) |
| Amicable Numbers | `amicable_numbers.rs` | [amicable_numbers.md](amicable_numbers.md) | O(√n) |
| Collatz Sequence | `collatz_sequence.rs` | [collatz_sequence.md](collatz_sequence.md) | Unknown |

### Distance & Geometry
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Euclidean Distance | `euclidean_distance.rs` | [euclidean_distance.md](euclidean_distance.md) | O(n) |

### Cryptographic Primitives
| Algorithm | File | Documentation | Complexity |
|-----------|------|---------------|------------|
| Elliptic Curve | `elliptic_curve.rs` | [elliptic_curve.md](elliptic_curve.md) | O(log n) |
| Baby-Step Giant-Step | `baby_step_giant_step.rs` | [baby_step_giant_step.md](baby_step_giant_step.md) | O(√n) |
| Quadratic Residue | `quadratic_residue.rs` | [quadratic_residue.md](quadratic_residue.md) | O(log p) |

## Quick Reference

### Complexity Comparison for Prime Testing

```
┌─────────────────────────┬──────────────────┬─────────────────┐
│ Algorithm               │ Time Complexity  │ Space           │
├─────────────────────────┼──────────────────┼─────────────────┤
│ Trial Division          │ O(√n)            │ O(1)            │
│ Sieve of Eratosthenes   │ O(n log log n)   │ O(n)            │
│ Linear Sieve            │ O(n)             │ O(n)            │
│ Miller-Rabin            │ O(k log³n)       │ O(1)            │
└─────────────────────────┴──────────────────┴─────────────────┘
```

### When to Use Which Algorithm

| Task | Best Algorithm | Notes |
|------|----------------|-------|
| Single prime check | Miller-Rabin | Probabilistic but fast |
| All primes up to n | Sieve of Eratosthenes | Best for n < 10^8 |
| Factorization | Pollard's Rho | For semi-primes |
| Modular inverse | Extended Euclidean | When gcd = 1 |
| Large number multiplication | Karatsuba/FFT | FFT for very large |

## Dependencies

Most algorithms use only the Rust standard library. Some require:
- `num-bigint`: For arbitrary precision integers
- `num-traits`: For numeric trait bounds

## Contributing

When adding new mathematical algorithms:
1. Include comprehensive test cases
2. Document time and space complexity
3. Provide mathematical foundations
4. Add edge case handling
5. Follow the documentation template in [PLAN.md](../PLAN.md)

## References

- Knuth, D. E. "The Art of Computer Programming, Volume 2: Seminumerical Algorithms"
- Cormen, T. H. et al. "Introduction to Algorithms"
- Hardy, G. H. & Wright, E. M. "An Introduction to the Theory of Numbers"
