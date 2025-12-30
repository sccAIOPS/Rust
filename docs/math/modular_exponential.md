# Modular Exponentiation

## 1. Overview

**Modular exponentiation** computes $a^b \mod m$ efficiently using the square-and-multiply method (binary exponentiation). This is a cornerstone of cryptographic algorithms like RSA and Diffie-Hellman.

**File**: `src/math/modular_exponential.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given integers $a$ (base), $b$ (exponent), and $m$ (modulus), compute:
$$a^b \mod m$$

### 2.2 Properties of Modular Arithmetic

1. $(a \cdot b) \mod m = ((a \mod m) \cdot (b \mod m)) \mod m$
2. $(a^b) \mod m = ((a \mod m)^b) \mod m$

### 2.3 Binary Exponentiation

Express $b$ in binary: $b = \sum_{i=0}^{k} b_i \cdot 2^i$

Then:
$$a^b = a^{\sum b_i \cdot 2^i} = \prod_{i: b_i=1} a^{2^i}$$

Each $a^{2^i} = (a^{2^{i-1}})^2$ can be computed by squaring.

## 3. Algorithm Description

### 3.1 Intuition

1. Break exponent into binary representation
2. For each bit position, square the base
3. Multiply into result when bit is 1
4. Take modulo at each step to keep numbers small

### 3.2 Pseudocode

```
function mod_pow(base, power, modulus):
    if modulus = 1:
        return 0
    
    result ← 1
    base ← base mod modulus
    
    while power > 0:
        if power is odd:
            result ← (result × base) mod modulus
        power ← power / 2
        base ← (base × base) mod modulus
    
    return result
```

### 3.3 Step-by-Step Example

Compute $3^{13} \mod 7$:

$13 = 1101_2$ (binary)

| Step | power | base | result | Operation |
|------|-------|------|--------|-----------|
| Init | 13 | 3 | 1 | - |
| 1 | 13 (odd) | 3 | 3 | result = 1×3 mod 7 |
| 2 | 6 | 2 | 3 | base = 3² mod 7 = 2 |
| 3 | 6 (even) | 2 | 3 | Skip |
| 4 | 3 | 4 | 3 | base = 2² mod 7 = 4 |
| 5 | 3 (odd) | 4 | 5 | result = 3×4 mod 7 = 5 |
| 6 | 1 | 2 | 5 | base = 4² mod 7 = 2 |
| 7 | 1 (odd) | 2 | **3** | result = 5×2 mod 7 = 3 |

**Verification**: $3^{13} = 1594323$, $1594323 \mod 7 = 3$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

$$O(\log b)$$

- Number of iterations = number of bits in $b$
- Each iteration: O(1) multiplications and modulo operations

### 4.2 Space Complexity

O(1) - only need a few variables.

### 4.3 Comparison with Naive Method

| Method | Time | Multiplications |
|--------|------|-----------------|
| Naive ($a \cdot a \cdot \ldots$) | O(b) | b - 1 |
| Binary exponentiation | O(log b) | ≤ 2 log b |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn modular_exponential(base: i64, mut power: i64, modulus: i64) -> i64 {
    if modulus == 1 {
        return 0;
    }

    // Handle negative exponents
    let mut base = if power < 0 {
        mod_inverse(base, modulus)
    } else {
        base % modulus
    };

    let mut result = 1;
    power = power.abs();

    while power > 0 {
        if power & 1 == 1 {
            result = (result * base) % modulus;
        }
        power >>= 1;
        base = (base * base) % modulus;
    }
    result
}
```

**Key features**:
- Handles negative exponents via modular inverse
- Uses bit operations for efficiency
- Handles modulus = 1 edge case

### 5.2 Overflow Prevention

For large numbers, use 128-bit intermediate results:

```rust
fn mod_pow_safe(base: u64, power: u64, modulus: u64) -> u64 {
    let mut result: u128 = 1;
    let mut base: u128 = (base % modulus) as u128;
    let modulus: u128 = modulus as u128;
    let mut power = power;
    
    while power > 0 {
        if power & 1 == 1 {
            result = (result * base) % modulus;
        }
        power >>= 1;
        base = (base * base) % modulus;
    }
    result as u64
}
```

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| (a, 0, m) | 1 mod m = 1 | Any number to power 0 |
| (a, b, 1) | 0 | Everything mod 1 is 0 |
| (0, 0, m) | 1 | Convention: 0⁰ = 1 |
| (a, -b, m) | Needs inverse | Requires gcd(a,m) = 1 |

## 6. Applications

### 6.1 Cryptography

1. **RSA Encryption**: $c = m^e \mod n$
2. **RSA Decryption**: $m = c^d \mod n$
3. **Diffie-Hellman**: $g^a \mod p$

### 6.2 Number Theory

- Fermat primality test: $a^{n-1} \mod n$
- Miller-Rabin test
- Computing multiplicative order

### 6.3 Competitive Programming

- Matrix exponentiation for Fibonacci
- Polynomial evaluation
- Fast computation of large powers

## 7. Extended: Modular Inverse

For negative exponents, we need the modular inverse:

```rust
pub fn mod_inverse(b: i64, m: i64) -> i64 {
    let (gcd, x, _) = gcd_extended(b, m);
    if gcd == 1 {
        (x % m + m) % m
    } else {
        panic!("Inverse does not exist");
    }
}
```

## 8. Related Algorithms

| Algorithm | Relation |
|-----------|----------|
| [Fast Power](fast_power.md) | Same technique, different interface |
| [Extended Euclidean](extended_euclidean.md) | For modular inverse |
| [Miller-Rabin](miller_rabin.md) | Uses modular exponentiation |
| [Chinese Remainder Theorem](chinese_remainder_theorem.md) | Optimizes large moduli |

## 9. References

1. Knuth, D. E. "The Art of Computer Programming, Vol. 2", Section 4.6.3
2. Menezes, A. et al. "Handbook of Applied Cryptography", Chapter 2
