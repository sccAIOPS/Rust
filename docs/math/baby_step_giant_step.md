# Baby-Step Giant-Step Algorithm

## 1. Overview

The **Baby-Step Giant-Step (BSGS)** algorithm solves the discrete logarithm problem (DLP) in O(√n) time and space. It finds $x$ such that $a^x \equiv b \pmod{n}$.

**File**: `src/math/baby_step_giant_step.rs`

## 2. Mathematical Foundation

### 2.1 Problem Statement

Given $a$, $b$, and $n$ where gcd(a, n) = 1, find $x$ such that:
$$a^x \equiv b \pmod{n}$$

### 2.2 Key Insight

Express $x = im - j$ where $m = \lceil\sqrt{n}\rceil$ and $0 \leq i, j < m$.

Then: $a^{im - j} \equiv b \pmod{n}$

Rearranging: $a^{im} \equiv b \cdot a^j \pmod{n}$

### 2.3 Algorithm

1. **Baby steps**: Compute and store $b \cdot a^j \pmod{n}$ for $j = 0, 1, ..., m-1$
2. **Giant steps**: Compute $a^{im} \pmod{n}$ for $i = 1, 2, ...$ until match found

## 3. Implementation

```rust
use std::collections::HashMap;

pub fn baby_step_giant_step(a: usize, b: usize, n: usize) -> Option<usize> {
    if greatest_common_divisor_stein(a as u64, n as u64) != 1 {
        return None;
    }

    let mut h_map = HashMap::new();
    let m = (n as f64).sqrt().ceil() as usize;
    
    // Baby step: compute b * a^j for j = 0..m
    let mut step = 1;
    for i in 0..m {
        h_map.insert((step * b) % n, i);
        step = (step * a) % n;
    }
    
    // Giant step: find i where a^(im) matches
    let giant_step = step;  // a^m
    for i in (m..=n).step_by(m) {
        if let Some(v) = h_map.get(&step) {
            return Some(i - v);
        }
        step = (step * giant_step) % n;
    }
    None
}
```

### 3.1 Usage Example

```rust
// Find x: 5^x ≡ 3 (mod 11)
assert_eq!(baby_step_giant_step(5, 3, 11), Some(2));
// Verify: 5² = 25 ≡ 3 (mod 11) ✓

// Find x: 3^x ≡ 83 (mod 100)
assert_eq!(baby_step_giant_step(3, 83, 100), Some(9));
// Verify: 3⁹ = 19683 ≡ 83 (mod 100) ✓

// No solution when gcd(a, n) ≠ 1
assert!(baby_step_giant_step(2, 1, 84).is_none());
```

## 4. Algorithm Trace

Finding x where $3^x \equiv 13 \pmod{17}$:

**Setup**: m = ⌈√17⌉ = 5

**Baby steps** (store b · aʲ mod n):
| j | 13 · 3ʲ mod 17 |
|---|----------------|
| 0 | 13 |
| 1 | 39 mod 17 = 5 |
| 2 | 15 |
| 3 | 11 |
| 4 | 16 |

**Giant steps** (compute aⁱᵐ mod n):
| i | 3⁵ⁱ mod 17 | Match? |
|---|------------|--------|
| 5 | 3⁵ = 243 mod 17 = 5 | Yes! j=1 |

**Result**: x = 5 - 1 = 4

**Verify**: 3⁴ = 81 ≡ 13 (mod 17) ✓

## 5. Complexity

- **Time**: O(√n)
- **Space**: O(√n) for the hash table

### 5.1 Comparison with Naive

| Method | Time | Space |
|--------|------|-------|
| Brute force | O(n) | O(1) |
| BSGS | O(√n) | O(√n) |

For n = 10⁹: naive needs ~10⁹ ops, BSGS needs ~31,623 ops.

## 6. Prerequisites

- gcd(a, n) = 1 (otherwise no solution)
- n should be reasonably small (due to √n space requirement)

## 7. Variants

### 7.1 Pohlig-Hellman

When n has known factorization, solve DLP in each prime power subgroup and combine with CRT.

### 7.2 Pollard's Rho

O(√n) time but O(1) space, probabilistic algorithm.

## 8. Applications

1. **Cryptanalysis**: Breaking Diffie-Hellman key exchange
2. **Order finding**: Determining order of elements in groups
3. **Index calculus**: Building block for more advanced DLP algorithms
4. **Elliptic curve cryptography**: Computing discrete logs on curves

## 9. Security Implications

The existence of sub-exponential DLP algorithms means:
- Small groups are insecure
- Recommended key sizes: 2048+ bits for finite fields
- Elliptic curves provide better security per bit

## 10. Related Algorithms

- [Modular Exponentiation](modular_exponential.md) - Computing aˣ mod n
- [Fast Power](fast_power.md) - Efficient exponentiation
- [Quadratic Residue](quadratic_residue.md) - Related cryptographic problem
- [Elliptic Curve](elliptic_curve.md) - DLP on elliptic curves

## 11. References

1. [Wikipedia: Baby-step giant-step](https://en.wikipedia.org/wiki/Baby-step_giant-step)
2. Shanks, D. (1971). "Class number, a theory of factorization, and genera"
3. Shoup, V. "A Computational Introduction to Number Theory and Algebra"
