# Fast Power (Binary Exponentiation)

## 1. Overview

**Fast power** (binary exponentiation, exponentiation by squaring) computes $a^n \bmod m$ in O(log n) time by exploiting the binary representation of the exponent.

**File**: `src/math/fast_power.rs`

## 2. Mathematical Foundation

### 2.1 Key Insight

Express the exponent in binary:
$$a^n = a^{\sum_{i} b_i \cdot 2^i} = \prod_{i: b_i = 1} a^{2^i}$$

### 2.2 Recurrence

$$a^n = \begin{cases} 
1 & \text{if } n = 0 \\
(a^{n/2})^2 & \text{if } n \text{ is even} \\
a \cdot (a^{(n-1)/2})^2 & \text{if } n \text{ is odd}
\end{cases}$$

### 2.3 Example: $2^{13}$

Binary of 13 = 1101₂

| Bit | Position | Power | Include? |
|-----|----------|-------|----------|
| 1 | 0 | $2^1 = 2$ | Yes |
| 0 | 1 | $2^2 = 4$ | No |
| 1 | 2 | $2^4 = 16$ | Yes |
| 1 | 3 | $2^8 = 256$ | Yes |

$2^{13} = 2^1 \cdot 2^4 \cdot 2^8 = 2 \cdot 16 \cdot 256 = 8192$

## 3. Implementation

```rust
pub fn fast_power(mut base: usize, mut power: usize, modulus: usize) -> usize {
    assert!(base >= 1);

    let mut res = 1;
    while power > 0 {
        if power & 1 == 1 {
            res = (res * base) % modulus;
        }
        base = (base * base) % modulus;
        power >>= 1;
    }
    res
}
```

### 3.1 Step-by-Step Trace

Computing $2^{10} \bmod 1000$:

| Iteration | power | base | res | Action |
|-----------|-------|------|-----|--------|
| 0 | 10 (1010₂) | 2 | 1 | power even, skip multiply |
| 1 | 5 (101₂) | 4 | 4 | power odd, res = 1 × 4 |
| 2 | 2 (10₂) | 16 | 4 | power even, skip multiply |
| 3 | 1 (1₂) | 256 | 24 | power odd, res = 4 × 256 mod 1000 |
| 4 | 0 | - | **24** | Done |

Result: $2^{10} \bmod 1000 = 1024 \bmod 1000 = 24$ ✓

### 3.2 Usage Example

```rust
const MOD: usize = 1000000007;

assert_eq!(fast_power(2, 1, MOD), 2);
assert_eq!(fast_power(2, 2, MOD), 4);
assert_eq!(fast_power(2, 4, MOD), 16);
assert_eq!(fast_power(3, 4, MOD), 81);
assert_eq!(fast_power(2, 100, MOD), 976371285);
```

## 4. Complexity

- **Time**: O(log n)
- **Space**: O(1)

### 4.1 Comparison

| Method | Time | Operations for n=1000 |
|--------|------|----------------------|
| Naive | O(n) | ~1000 multiplications |
| Binary exp | O(log n) | ~10 multiplications |

## 5. Algorithm Variants

### 5.1 Without Modulus

```rust
fn fast_power_no_mod(mut base: u64, mut power: u32) -> u64 {
    let mut result = 1;
    while power > 0 {
        if power & 1 == 1 {
            result *= base;
        }
        base *= base;
        power >>= 1;
    }
    result
}
```

### 5.2 Recursive Version

```rust
fn fast_power_recursive(base: u64, power: u32, modulus: u64) -> u64 {
    if power == 0 { return 1; }
    let half = fast_power_recursive(base, power / 2, modulus);
    let half_squared = (half * half) % modulus;
    if power % 2 == 1 {
        (half_squared * base) % modulus
    } else {
        half_squared
    }
}
```

## 6. Applications

1. **Cryptography**: RSA encryption/decryption
2. **Modular arithmetic**: Computing inverses via Fermat's little theorem
3. **Competitive programming**: Large exponentiation problems
4. **Primality testing**: Miller-Rabin test
5. **Matrix exponentiation**: Fibonacci in O(log n)

## 7. Fermat's Little Theorem Application

For prime $p$ and $a$ not divisible by $p$:
$$a^{-1} \equiv a^{p-2} \pmod{p}$$

```rust
fn mod_inverse(a: usize, p: usize) -> usize {
    fast_power(a, p - 2, p)
}
```

## 8. Related Algorithms

- [Modular Exponentiation](modular_exponential.md) - With negative exponents
- [Miller-Rabin](miller_rabin.md) - Uses fast power for primality testing
- [RSA](../ciphers/) - Uses modular exponentiation

## 9. References

1. [Wikipedia: Exponentiation by squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring)
2. [CP-Algorithms: Binary Exponentiation](https://cp-algorithms.com/algebra/binary-exp.html)
3. Knuth, D. E. "The Art of Computer Programming, Vol. 2"
