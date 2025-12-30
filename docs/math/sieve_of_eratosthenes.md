# Sieve of Eratosthenes

## 1. Overview

The **Sieve of Eratosthenes** is an ancient algorithm for finding all prime numbers up to a specified limit. It is one of the most efficient ways to find small primes and serves as the basis for more advanced sieves.

**Historical Context**: Attributed to the Greek mathematician Eratosthenes of Cyrene (c. 276-194 BCE), who also calculated the Earth's circumference.

**File**: `src/math/sieve_of_eratosthenes.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer $n$, find all prime numbers $p$ where $2 \leq p \leq n$.

### 2.2 Key Insight

If $n$ is composite, it must have a prime factor $p \leq \sqrt{n}$.

**Corollary**: To find all primes up to $n$, we only need to sieve out multiples of primes up to $\sqrt{n}$.

### 2.3 Algorithm Foundation

For each prime $p$:
- Mark all multiples $p^2, p^2+p, p^2+2p, \ldots$ as composite
- Start from $p^2$ because smaller multiples were already marked by smaller primes

## 3. Algorithm Description

### 3.1 Intuition

1. Create a list of consecutive integers from 2 to $n$
2. Start with the first prime (2)
3. Cross out all multiples of the current prime
4. Move to the next uncrossed number (it's the next prime)
5. Repeat until you've processed all numbers up to $\sqrt{n}$
6. All remaining uncrossed numbers are prime

### 3.2 Pseudocode

```
function sieve_of_eratosthenes(n):
    is_prime[0..n] ← true
    is_prime[0] ← false
    is_prime[1] ← false
    
    for p from 2 to √n:
        if is_prime[p]:
            for multiple from p² to n step p:
                is_prime[multiple] ← false
    
    return [i for i in 2..n if is_prime[i]]
```

### 3.3 Step-by-Step Example

Find all primes up to 30:

```
Initial: 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30

p=2: Cross out 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30
     2 3 _ 5 _ 7 _ 9 _  11 _  13 _  15 _  17 _  19 _  21 _  23 _  25 _  27 _  29 _

p=3: Cross out 9, 15, 21, 27
     2 3 _ 5 _ 7 _ _ _  11 _  13 _  _  _  17 _  19 _  _  _  23 _  25 _  _  _  29 _

p=5: Cross out 25
     2 3 _ 5 _ 7 _ _ _  11 _  13 _  _  _  17 _  19 _  _  _  23 _  _  _  _  _  29 _

√30 ≈ 5.5, so we stop.

Primes: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29
```

### 3.4 Visual Representation

```
Step 0: [2][3][4][5][6][7][8][9][10]...[30]
Step 1: [2][3][ ][5][ ][7][ ][9][  ]...[ ]  (multiples of 2)
Step 2: [2][3][ ][5][ ][7][ ][ ][  ]...[ ]  (multiples of 3)
Step 3: [2][3][ ][5][ ][7][ ][ ][  ]...[ ]  (multiples of 5)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

$$T(n) = O(n \log \log n)$$

**Derivation**:
The inner loop runs $\frac{n}{p}$ times for each prime $p$. Total work:
$$\sum_{p \leq n, p \text{ prime}} \frac{n}{p} = n \sum_{p \leq n} \frac{1}{p}$$

By Mertens' theorem: $\sum_{p \leq n} \frac{1}{p} \approx \log \log n$

### 4.2 Space Complexity

O(n) - for the boolean array.

### 4.3 Cache Efficiency

The standard sieve has poor cache locality for large $n$. Segmented sieve improves this.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn sieve_of_eratosthenes(num: usize) -> Vec<usize> {
    let mut result: Vec<usize> = Vec::new();
    if num >= 2 {
        let mut sieve: Vec<bool> = vec![true; num + 1];
        sieve[0] = false;
        sieve[1] = false;

        let end: usize = (num as f64).sqrt() as usize;

        for start in 2..=end {
            if sieve[start] {
                result.push(start);
                for i in (start * start..=num).step_by(start) {
                    sieve[i] = false;
                }
            }
        }

        // Collect remaining primes beyond √num
        for (i, &is_prime) in sieve.iter().enumerate().skip(end + 1) {
            if is_prime {
                result.push(i);
            }
        }
    }
    result
}
```

**Key optimizations**:
- Start crossing at $p^2$, not $2p$
- Use `step_by` for efficient iteration
- Collect primes in two phases

### 5.2 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | `[]` | No primes |
| 1 | `[]` | No primes |
| 2 | `[2]` | Smallest prime |
| 10 | `[2,3,5,7]` | Standard test |

### 5.3 Memory Optimizations

**Bit Array**: Use 1 bit per number instead of 1 byte:
```rust
// Space: n/8 bytes instead of n bytes
let mut sieve = vec![0u64; (n + 63) / 64];
```

**Odd-only sieve**: Store only odd numbers:
```rust
// Space: n/16 bytes
// sieve[i] represents (2i + 3)
```

## 6. Real-World Applications

### 6.1 Use Cases

1. **Cryptography**: Generating prime candidates for RSA
2. **Number Theory Research**: Studying prime distribution
3. **Competitive Programming**: Precompute primes for multiple queries
4. **Hash Functions**: Finding prime table sizes

### 6.2 Industry Applications

| Application | Why Sieve? |
|-------------|------------|
| Primality testing | Pre-compute small primes for trial division |
| Factorization | Fast lookup of small factors |
| Random sampling | Generate uniformly random primes |

## 7. Variants

### 7.1 Segmented Sieve

For finding primes in range $[L, R]$ when $R$ is large:

```rust
// Only needs O(√R) space
fn segmented_sieve(l: usize, r: usize) -> Vec<usize> {
    let small_primes = sieve_of_eratosthenes((r as f64).sqrt() as usize);
    let mut is_prime = vec![true; r - l + 1];
    
    for p in small_primes {
        let start = ((l + p - 1) / p) * p;
        for i in (start..=r).step_by(p) {
            if i > p {
                is_prime[i - l] = false;
            }
        }
    }
    // ... collect primes
}
```

### 7.2 Comparison with Other Sieves

| Sieve | Time | Space | Best For |
|-------|------|-------|----------|
| Eratosthenes | O(n log log n) | O(n) | General use |
| [Linear Sieve](linear_sieve.md) | O(n) | O(n) | Need factorization |
| Sundaram | O(n log n) | O(n) | Only odd primes |
| Atkin | O(n) | O(n) | Theoretical interest |

## 8. Related Algorithms

| Algorithm | Relation |
|-----------|----------|
| [Linear Sieve](linear_sieve.md) | O(n) time, includes factorization |
| [Prime Check](prime_check.md) | Single number primality |
| [Miller-Rabin](miller_rabin.md) | Large number primality |

## 9. Performance Benchmarks

```
Benchmark: Generate all primes up to n
┌───────────┬─────────────┬──────────────┬───────────────┐
│ n         │ Primes      │ Time (ms)    │ Memory (MB)   │
├───────────┼─────────────┼──────────────┼───────────────┤
│ 10⁶       │ 78,498      │ ~10          │ ~1            │
│ 10⁷       │ 664,579     │ ~100         │ ~10           │
│ 10⁸       │ 5,761,455   │ ~1,000       │ ~100          │
│ 10⁹       │ 50,847,534  │ ~15,000      │ ~1,000        │
└───────────┴─────────────┴──────────────┴───────────────┘
```

## 10. References

1. Eratosthenes of Cyrene (c. 276-194 BCE)
2. Crandall, R. & Pomerance, C. "Prime Numbers: A Computational Perspective"
3. [Wikipedia: Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)
