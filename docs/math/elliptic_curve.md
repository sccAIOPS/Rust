# Elliptic Curves

## 1. Overview

**Elliptic curves** are algebraic structures fundamental to modern cryptography. This implementation defines elliptic curves over prime fields and provides group operations on curve points.

**File**: `src/math/elliptic_curve.rs`

## 2. Mathematical Foundation

### 2.1 Definition

An elliptic curve over a field F is defined by the Weierstrass equation:

$$E: y^2 = x^3 + Ax + B$$

Where the **discriminant** must be non-zero:
$$\Delta = -16(4A^3 + 27B^2) \neq 0$$

### 2.2 Points on the Curve

Points form an **abelian group** with:
- Affine points $(x, y)$ satisfying the curve equation
- Point at infinity $\mathcal{O}$ (identity element)

### 2.3 Group Law

For points $P = (x_1, y_1)$ and $Q = (x_2, y_2)$:

**Point addition** ($P \neq Q$):
$$\lambda = \frac{y_2 - y_1}{x_2 - x_1}$$

**Point doubling** ($P = Q$):
$$\lambda = \frac{3x_1^2 + A}{2y_1}$$

**Result** $R = P + Q = (x_3, y_3)$:
$$x_3 = \lambda^2 - x_1 - x_2$$
$$y_3 = \lambda(x_1 - x_3) - y_1$$

## 3. Implementation

### 3.1 Curve Structure

```rust
pub struct EllipticCurve<F, const A: i64, const B: i64> {
    infinity: bool,
    x: F,
    y: F,
}

impl<F: Field, const A: i64, const B: i64> EllipticCurve<F, A, B> {
    pub fn infinity() -> Self {
        Self {
            infinity: true,
            x: F::ZERO,
            y: F::ZERO,
        }
    }

    pub fn new(x: impl Into<F>, y: impl Into<F>) -> Option<Self> {
        let x = x.into();
        let y = y.into();
        if Self::contains(x, y) {
            Some(Self { infinity: false, x, y })
        } else {
            None
        }
    }

    fn contains(x: F, y: F) -> bool {
        y * y == x * x * x + x.integer_mul(A) + F::ONE.integer_mul(B)
    }
}
```

### 3.2 Usage Example

```rust
use the_algorithms_rust::math::{EllipticCurve, PrimeField};

// Define curve y² = x³ + x over F_7
type E = EllipticCurve<PrimeField<7>, 1, 0>;

// Create point (0, 0) on curve
let P = E::new(0, 0).expect("not on curve");

// Point at infinity (identity)
let O = E::infinity();

// Point doubling
let Q = P + P;
assert_eq!(Q, E::infinity());  // Order of P is 2
```

## 4. Group Operations

### 4.1 Addition

```rust
impl<F: Field, const A: i64, const B: i64> Add for EllipticCurve<F, A, B> {
    type Output = Self;

    fn add(self, other: Self) -> Self {
        if self.is_infinity() { return other; }
        if other.is_infinity() { return self; }
        
        if self.x == other.x && self.y == -other.y {
            return Self::infinity();
        }
        
        let lambda = if self == other {
            // Point doubling
            (F::THREE * self.x * self.x + F::from(A)) / (F::TWO * self.y)
        } else {
            // Point addition
            (other.y - self.y) / (other.x - self.x)
        };
        
        let x3 = lambda * lambda - self.x - other.x;
        let y3 = lambda * (self.x - x3) - self.y;
        
        Self { infinity: false, x: x3, y: y3 }
    }
}
```

### 4.2 Negation

```rust
impl<F: Field, const A: i64, const B: i64> Neg for EllipticCurve<F, A, B> {
    fn neg(self) -> Self {
        if self.is_infinity() { self }
        else { Self { infinity: false, x: self.x, y: -self.y } }
    }
}
```

## 5. Properties

### 5.1 Curve Invariants

| Property | Constraint |
|----------|-----------|
| Characteristic | ≠ 2, 3 (simplified Weierstrass form) |
| Discriminant | Δ ≠ 0 (curve is smooth) |
| Group order | #E(F_p) ≈ p (Hasse's theorem) |

### 5.2 Hasse's Theorem

For curve over $\mathbb{F}_p$:
$$|p + 1 - \#E(\mathbb{F}_p)| \leq 2\sqrt{p}$$

## 6. Example Curves

### 6.1 Small Curve: y² = x³ + x (mod 7)

Points: $\mathcal{O}$, (0,0), (2,1), (2,6), (4,2), (4,5), (5,1), (5,6)

Order: 8 points

### 6.2 secp256k1 (Bitcoin)

$$y^2 = x^3 + 7 \pmod{p}$$

Where $p = 2^{256} - 2^{32} - 977$

## 7. Complexity

| Operation | Time |
|-----------|------|
| Point addition | O(log p) field ops |
| Point doubling | O(log p) field ops |
| Scalar multiplication (nP) | O(log n × log p) |
| Point enumeration | O(p) |

## 8. Applications

1. **ECDSA**: Digital signatures (Bitcoin, TLS)
2. **ECDH**: Key exchange
3. **EdDSA**: Ed25519 signatures
4. **Pairing-based crypto**: ZK-proofs, IBE
5. **Isogeny-based crypto**: Post-quantum candidates

## 9. Security

### 9.1 Elliptic Curve Discrete Logarithm Problem (ECDLP)

Given $P$ and $Q = nP$, find $n$.

- Best known: O(√p) using Pollard's rho
- Provides ~128-bit security with 256-bit curves

### 9.2 Comparison with RSA

| Security Level | RSA Key | ECC Key |
|---------------|---------|---------|
| 80-bit | 1024 bits | 160 bits |
| 128-bit | 3072 bits | 256 bits |
| 256-bit | 15360 bits | 512 bits |

## 10. Related Algorithms

- [Quadratic Residue](quadratic_residue.md) - Point decompression
- [Baby-Step Giant-Step](baby_step_giant_step.md) - Solving ECDLP
- [Modular Exponentiation](modular_exponential.md) - Field arithmetic

## 11. References

1. [Wikipedia: Elliptic curve](https://en.wikipedia.org/wiki/Elliptic_curve)
2. Washington, L. C. "Elliptic Curves: Number Theory and Cryptography"
3. Silverman, J. H. "The Arithmetic of Elliptic Curves"
4. [NIST: Recommended Elliptic Curves](https://csrc.nist.gov/publications/detail/fips/186/4/final)
