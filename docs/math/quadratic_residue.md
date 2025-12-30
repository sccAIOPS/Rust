# Quadratic Residue (Cipolla's Algorithm)

## 1. Overview

**Quadratic residue** problems involve finding square roots modulo a prime. **Cipolla's algorithm** solves $x^2 \equiv a \pmod{p}$ for odd prime $p$ using an extension field.

**File**: `src/math/quadratic_residue.rs`

## 2. Mathematical Foundation

### 2.1 Definitions

A number $a$ is a **quadratic residue** mod $p$ if there exists $x$ such that:
$$x^2 \equiv a \pmod{p}$$

### 2.2 Legendre Symbol

The Legendre symbol $\left(\frac{a}{p}\right)$ is:
$$\left(\frac{a}{p}\right) = \begin{cases}
0 & \text{if } a \equiv 0 \pmod{p} \\
1 & \text{if } a \text{ is a quadratic residue mod } p \\
-1 & \text{if } a \text{ is a quadratic non-residue mod } p
\end{cases}$$

### 2.3 Euler's Criterion

$$\left(\frac{a}{p}\right) \equiv a^{(p-1)/2} \pmod{p}$$

This provides an efficient test for quadratic residuosity.

### 2.4 Cipolla's Algorithm

1. Find $t$ such that $t^2 - a$ is a non-residue mod $p$
2. Define field extension $\mathbb{F}_{p^2}$ with $i^2 = t^2 - a$
3. Compute $(t + i)^{(p+1)/2}$ in this extension
4. The result's real part is the solution

## 3. Implementation

### 3.1 Custom Complex Number Type

```rust
struct CustomComplexNumber {
    real: u64,
    imag: u64,
    f: Rc<CustomFiniteField>,  // Contains modulus and i²
}

impl CustomComplexNumber {
    pub fn mult_other(&mut self, rhs: &Self) {
        let tmp = (self.imag * rhs.real + self.real * rhs.imag) % self.f.modulus;
        self.real = (self.real * rhs.real
            + ((self.imag * rhs.imag) % self.f.modulus) * self.f.i_square)
            % self.f.modulus;
        self.imag = tmp;
    }
}
```

### 3.2 Residue Check

```rust
fn is_residue(x: u64, modulus: u64) -> bool {
    let power = (modulus - 1) >> 1;
    x != 0 && fast_power(x, power, modulus) == 1
}
```

### 3.3 Legendre Symbol

```rust
pub fn legendre_symbol(a: u64, odd_prime: u64) -> i64 {
    let result = fast_power(a, (odd_prime - 1) / 2, odd_prime);
    match result {
        0 => 0,
        1 => 1,
        _ => -1,  // result == p - 1
    }
}
```

## 4. Algorithm

### 4.1 Pseudocode

```
function cipolla(a, p):
    // Step 1: Check if solution exists
    if legendre_symbol(a, p) != 1:
        return None  // No solution
    
    // Step 2: Find non-residue
    t = random in [1, p-1]
    while is_residue(t² - a, p):
        t = random in [1, p-1]
    
    // Step 3: Compute in extension field
    i² = (t² - a) mod p
    result = (t + i)^((p+1)/2) in F_p[i]
    
    return result.real
```

## 5. Complexity

- **Time**: O(log p) for exponentiation (expected O(log p) trials to find t)
- **Space**: O(1)

## 6. Example

Solve $x^2 \equiv 10 \pmod{13}$:

1. **Check residuosity**: $10^{(13-1)/2} = 10^6 \equiv 1 \pmod{13}$ ✓

2. **Find non-residue t²-a**: Try t=2: 2²-10 = -6 ≡ 7 (mod 13)
   - $7^6 \equiv 12 \equiv -1 \pmod{13}$ → 7 is non-residue ✓

3. **Compute in extension**: i² = 7, compute (2+i)⁷ mod 13
   - Result: x = 6 (or x = 7, since -6 ≡ 7)

4. **Verify**: 6² = 36 ≡ 10 (mod 13) ✓

## 7. When Solutions Exist

For odd prime $p$:
- Exactly $(p-1)/2$ quadratic residues in $\{1, 2, ..., p-1\}$
- Exactly $(p-1)/2$ quadratic non-residues
- If $x^2 \equiv a$, then so does $(-x)^2$

## 8. Alternative Methods

| Method | Use Case | Complexity |
|--------|----------|------------|
| Cipolla | General odd primes | O(log p) |
| Tonelli-Shanks | Large primes, p-1 has small factors | O(log² p) |
| Direct formula | p ≡ 3 (mod 4) | O(log p) |

### 8.1 Special Case: p ≡ 3 (mod 4)

$$x \equiv \pm a^{(p+1)/4} \pmod{p}$$

## 9. Applications

1. **Cryptography**: RSA variant schemes
2. **Elliptic curves**: Point decompression
3. **Primality testing**: Part of some primality tests
4. **Number theory**: Studying quadratic forms

## 10. Related Concepts

- [Fast Power](fast_power.md) - Used for exponentiation
- [Miller-Rabin](miller_rabin.md) - Uses quadratic residues
- [Elliptic Curve](elliptic_curve.md) - Requires square roots

## 11. References

1. [Wikipedia: Cipolla's algorithm](https://en.wikipedia.org/wiki/Cipolla%27s_algorithm)
2. [Wikipedia: Quadratic residue](https://en.wikipedia.org/wiki/Quadratic_residue)
3. Cohen, H. "A Course in Computational Algebraic Number Theory"
