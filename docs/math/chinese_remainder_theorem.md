# Chinese Remainder Theorem

## 1. Overview

The **Chinese Remainder Theorem (CRT)** provides a method to find a number that simultaneously satisfies multiple modular congruences. Given remainders when divided by coprime moduli, CRT finds the unique solution modulo the product of all moduli.

**File**: `src/math/chinese_remainder_theorem.rs`

## 2. Mathematical Foundation

### 2.1 Problem Statement

Find $x$ such that:
$$\begin{cases}
x \equiv a_1 \pmod{n_1} \\
x \equiv a_2 \pmod{n_2} \\
\vdots \\
x \equiv a_k \pmod{n_k}
\end{cases}$$

Where $n_1, n_2, ..., n_k$ are pairwise coprime.

### 2.2 Theorem

If $n_1, n_2, ..., n_k$ are pairwise coprime, then there exists a unique solution modulo $N = n_1 \cdot n_2 \cdot ... \cdot n_k$.

### 2.3 Construction

$$x = \sum_{i=1}^{k} a_i \cdot N_i \cdot y_i \pmod{N}$$

Where:
- $N_i = N / n_i$
- $y_i = N_i^{-1} \pmod{n_i}$ (modular inverse)

### 2.4 Example

Solve:
- $x \equiv 2 \pmod{3}$
- $x \equiv 3 \pmod{5}$
- $x \equiv 2 \pmod{7}$

| Step | Value |
|------|-------|
| N = 3 × 5 × 7 | 105 |
| N₁ = 105/3 = 35 | 35⁻¹ ≡ 2 (mod 3) |
| N₂ = 105/5 = 21 | 21⁻¹ ≡ 1 (mod 5) |
| N₃ = 105/7 = 15 | 15⁻¹ ≡ 1 (mod 7) |
| x = 2(35)(2) + 3(21)(1) + 2(15)(1) | = 140 + 63 + 30 = 233 |
| x mod 105 | = 23 |

Verification: 23 ≡ 2 (mod 3), 23 ≡ 3 (mod 5), 23 ≡ 2 (mod 7) ✓

## 3. Implementation

```rust
use super::extended_euclidean_algorithm;

fn mod_inv(x: i32, n: i32) -> Option<i32> {
    let (g, x, _) = extended_euclidean_algorithm(x, n);
    if g == 1 {
        Some((x % n + n) % n)
    } else {
        None
    }
}

pub fn chinese_remainder_theorem(residues: &[i32], modulli: &[i32]) -> Option<i32> {
    let prod = modulli.iter().product::<i32>();

    let mut sum = 0;

    for (&residue, &modulus) in residues.iter().zip(modulli) {
        let p = prod / modulus;
        sum += residue * mod_inv(p, modulus)? * p
    }
    Some(sum % prod)
}
```

### 3.1 Usage Example

```rust
// x ≡ 3 (mod 2), x ≡ 5 (mod 3), x ≡ 7 (mod 1)
let result = chinese_remainder_theorem(&[3, 5, 7], &[2, 3, 1]);
// Result: Some(5)

// x ≡ 1 (mod 3), x ≡ 4 (mod 5), x ≡ 6 (mod 7)  
let result = chinese_remainder_theorem(&[1, 4, 6], &[3, 5, 7]);
// Result: Some(34)

// Non-coprime moduli (no solution)
let result = chinese_remainder_theorem(&[1, 4, 6], &[1, 2, 0]);
// Result: None
```

## 4. Algorithm

### 4.1 Pseudocode

```
function CRT(residues, moduli):
    N = product(moduli)
    x = 0
    
    for i in 0..k:
        Ni = N / moduli[i]
        yi = modular_inverse(Ni, moduli[i])
        if yi is None: return None  // Not coprime
        x += residues[i] * Ni * yi
    
    return x mod N
```

## 5. Complexity

- **Time**: O(k × log(max_modulus)) for k congruences
- **Space**: O(1)

The log factor comes from computing modular inverses via the [Extended Euclidean Algorithm](extended_euclidean.md).

## 6. Prerequisites

For CRT to have a solution:
- All moduli must be pairwise coprime: gcd(nᵢ, nⱼ) = 1 for i ≠ j
- Uses [Extended Euclidean Algorithm](extended_euclidean.md) for modular inverse

## 7. Applications

1. **Cryptography**: RSA key generation, secret sharing
2. **Large integer arithmetic**: Representing large numbers as residues
3. **Parallel computing**: Splitting computations by moduli
4. **Hash functions**: Combining multiple hash values
5. **Error correction**: Reed-Solomon codes
6. **Calendar calculations**: Finding dates satisfying multiple cycles

## 8. Historical Context

The theorem is named after its discovery in Chinese mathematics:
- First appeared in "Sunzi Suanjing" (~3rd century AD)
- Problem: "There are things of an unknown number which when divided by 3 leave 2, by 5 leave 3, and by 7 leave 2. What is the number?"
- Answer: 23 (and 23 + 105k for any integer k)

## 9. Generalization

For non-coprime moduli, a solution exists if and only if:
$$a_i \equiv a_j \pmod{\gcd(n_i, n_j)}$$

for all pairs $(i, j)$.

## 10. Related Algorithms

- [Extended Euclidean Algorithm](extended_euclidean.md) - For computing modular inverse
- [GCD](gcd.md) - For checking coprimality
- [Modular Exponentiation](modular_exponential.md) - Often used together

## 11. References

1. [Wikipedia: Chinese remainder theorem](https://en.wikipedia.org/wiki/Chinese_remainder_theorem)
2. Knuth, D. E. "The Art of Computer Programming, Vol. 2: Seminumerical Algorithms"
3. Shoup, V. "A Computational Introduction to Number Theory and Algebra"
