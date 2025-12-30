# Miller-Rabin Primality Test

## 1. Overview

The **Miller-Rabin primality test** is a probabilistic algorithm that determines whether a given number is likely to be prime. It's widely used in cryptographic applications due to its efficiency with large numbers.

**Historical Context**: Developed by Gary L. Miller (1976) as a deterministic test under the Extended Riemann Hypothesis, later converted to a probabilistic test by Michael O. Rabin (1980).

**File**: `src/math/miller_rabin.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive odd integer $n > 2$, determine whether $n$ is prime or composite.

### 2.2 Theoretical Basis

**Fermat's Little Theorem**: If $p$ is prime and $\gcd(a, p) = 1$, then:
$$a^{p-1} \equiv 1 \pmod{p}$$

**Key Insight**: For odd prime $p$, write $p - 1 = 2^s \cdot d$ where $d$ is odd. Then for any $a$ coprime to $p$:

Either:
$$a^d \equiv 1 \pmod{p}$$

Or there exists $r \in \{0, 1, \ldots, s-1\}$ such that:
$$a^{2^r \cdot d} \equiv -1 \pmod{p}$$

**Definition**: A number $a$ is called a **witness** for the compositeness of $n$ if $n$ fails the Miller-Rabin test with base $a$.

### 2.3 Witness Theorem

**Theorem**: If $n$ is an odd composite, then at least $\frac{3}{4}$ of all bases $a \in \{1, 2, \ldots, n-1\}$ are witnesses for $n$.

**Corollary**: The probability of error after $k$ independent tests is at most $\left(\frac{1}{4}\right)^k$.

## 3. Algorithm Description

### 3.1 Intuition

1. Write $n - 1 = 2^s \cdot d$ (factor out all powers of 2)
2. For each test base $a$:
   - Compute $x = a^d \mod n$
   - If $x = 1$ or $x = n-1$, then $n$ passes this round
   - Square $x$ repeatedly up to $s-1$ times
   - If we ever get $n-1$, then $n$ passes
   - If we never get $n-1$ (and didn't start at 1), $n$ is composite
3. If $n$ passes all rounds, it's probably prime

### 3.2 Pseudocode

```
function miller_rabin(n, bases[]):
    if n ≤ 4:
        return (n = 2) or (n = 3)
    
    # Write n-1 = 2^s · d
    s ← trailing_zeros(n - 1)
    d ← (n - 1) >> s
    
    for a in bases:
        if not check_witness(n, a, d, s):
            return a  # Found witness, n is composite
    
    return 0  # Probably prime

function check_witness(n, a, d, s):
    x ← modular_power(a, d, n)
    
    if x = 1 or x = n-1:
        return true  # Passes this round
    
    for _ in 1 to s-1:
        x ← (x × x) mod n
        if x = n-1:
            return true
    
    return false  # a is a witness
```

### 3.3 Step-by-Step Example

Test if $n = 221 = 13 \times 17$ is prime using base $a = 174$:

1. $n - 1 = 220 = 2^2 \times 55$, so $s = 2$, $d = 55$

2. Compute $x = 174^{55} \mod 221$:
   - $174^{55} \equiv 47 \pmod{221}$
   - $47 \neq 1$ and $47 \neq 220$

3. Square: $x = 47^2 \mod 221 = 2209 \mod 221 = 220$
   - $220 = n - 1$ ✓

4. **Result**: 221 passes for base 174 (but it's actually composite!)

Now test with base $a = 137$:
- $137^{55} \equiv 188 \pmod{221}$
- $188^2 \equiv 205 \pmod{221}$
- $205 \neq 1$ and $205 \neq 220$

**Result**: 137 is a witness; 221 is composite.

## 4. Complexity Analysis

### 4.1 Time Complexity

For a single test with base $a$:
$$O(\log^2 n \cdot \log \log n \cdot \log \log \log n)$$

Simplified: $O(\log^3 n)$ using schoolbook multiplication, or $O(\log^2 n)$ with FFT.

For $k$ tests:
$$O(k \log^3 n)$$

### 4.2 Space Complexity

O(1) - only need to store a few big integers.

### 4.3 Error Probability

| Rounds (k) | Error Probability |
|------------|-------------------|
| 1 | ≤ 1/4 |
| 10 | ≤ 10⁻⁶ |
| 20 | ≤ 10⁻¹² |
| 40 | ≤ 10⁻²⁴ |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
fn modulo_power(mut base: u64, mut power: u64, modulo: u64) -> u64 {
    base %= modulo;
    if base == 0 { return 0; }
    let mut ans: u128 = 1;
    let mut bbase: u128 = base as u128;
    while power > 0 {
        if (power % 2) == 1 {
            ans = (ans * bbase) % (modulo as u128);
        }
        bbase = (bbase * bbase) % (modulo as u128);
        power /= 2;
    }
    ans as u64
}

fn check_prime_base(number: u64, base: u64, two_power: u64, odd_power: u64) -> bool {
    let mut x: u128 = modulo_power(base, odd_power, number) as u128;
    let bnumber: u128 = number as u128;
    if x == 1 || x == (bnumber - 1) {
        return true;
    }
    for _ in 1..two_power {
        x = (x * x) % bnumber;
        if x == (bnumber - 1) {
            return true;
        }
    }
    false
}

pub fn miller_rabin(number: u64, bases: &[u64]) -> u64 {
    // Returns 0 for probable prime, witness otherwise
    if number <= 4 {
        match number {
            0 => panic!("0 is invalid"),
            2 | 3 => return 0,
            _ => return number,
        }
    }
    if bases.contains(&number) {
        return 0;
    }
    let two_power: u64 = (number - 1).trailing_zeros() as u64;
    let odd_power = (number - 1) >> two_power;
    for base in bases {
        if !check_prime_base(number, *base, two_power, odd_power) {
            return *base;
        }
    }
    0
}
```

### 5.2 Deterministic Bases

For **deterministic** results (no false positives):

| Range | Sufficient Bases |
|-------|------------------|
| n < 2,047 | {2} |
| n < 1,373,653 | {2, 3} |
| n < 9,080,191 | {31, 73} |
| n < 25,326,001 | {2, 3, 5} |
| n < 3,215,031,751 | {2, 3, 5, 7} |
| n < 4,759,123,141 | {2, 7, 61} |
| n < 2⁶⁴ | {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37} |

### 5.3 Edge Cases

| Input | Handling |
|-------|----------|
| 0 | Invalid (panic/error) |
| 1 | Not prime |
| 2, 3 | Prime |
| Even > 2 | Composite |
| Base in list | Prime |

## 6. Real-World Applications

### 6.1 Use Cases

1. **RSA Key Generation**: Finding large primes (1024-4096 bits)
2. **Digital Signatures**: DSA, ECDSA prime generation
3. **Primality Certificates**: As part of proving primality
4. **Cryptographic Protocols**: Generating safe primes

### 6.2 Comparison with Other Tests

| Test | Type | Time | Use Case |
|------|------|------|----------|
| Trial Division | Deterministic | O(√n) | Small numbers |
| Miller-Rabin | Probabilistic | O(k log³n) | General purpose |
| AKS | Deterministic | O(log⁶n) | Theoretical |
| BPSW | Probabilistic | O(log³n) | No known counterexamples |

## 7. BigInteger Support

The implementation includes `big_miller_rabin` for arbitrary precision:

```rust
pub fn big_miller_rabin(number_ref: &BigUint, bases: &[u64]) -> u64 {
    // Same algorithm, but using BigUint operations
    // Essential for cryptographic applications
}
```

## 8. Related Algorithms

| Algorithm | Relation |
|-----------|----------|
| [Prime Check](prime_check.md) | Trial division alternative |
| [Sieve of Eratosthenes](sieve_of_eratosthenes.md) | Pre-generate small primes |
| [Pollard's Rho](pollard_rho.md) | Factorization (uses Miller-Rabin) |
| [Modular Exponentiation](modular_exponential.md) | Core subroutine |

## 9. Security Considerations

1. **Random Base Selection**: For cryptographic use, bases should be randomly chosen
2. **Timing Attacks**: Constant-time implementation may be needed
3. **Strong Primes**: May need additional tests for "safe prime" properties

## 10. References

1. Miller, G. L. "Riemann's Hypothesis and Tests for Primality" (1976)
2. Rabin, M. O. "Probabilistic Algorithm for Testing Primality" (1980)
3. Menezes, A. et al. "Handbook of Applied Cryptography", Chapter 4
4. [Wikipedia: Miller-Rabin Primality Test](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test)
