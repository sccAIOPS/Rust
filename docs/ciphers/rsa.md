# RSA (Rivest-Shamir-Adleman)

## 1. Overview

**RSA** is a public-key cryptosystem that enables secure data transmission. Named after its inventors Ron Rivest, Adi Shamir, and Leonard Adleman, who publicly described the algorithm in 1977, it was one of the first practical public-key cryptosystems and remains widely used for secure data transmission.

### Historical Context
- **1976**: Whitfield Diffie and Martin Hellman publish concept of public-key cryptography
- **1977**: RSA algorithm developed at MIT
- **1983**: RSA patented (expired 2000)
- **Present**: Foundational algorithm for digital signatures and key exchange

⚠️ **Warning**: This implementation is for educational purposes only. Production systems should use established libraries with proper key sizes and padding schemes.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Key Generation**:
Given two large prime numbers $p$ and $q$, generate:
- Public key $(n, e)$ for encryption
- Private key $(n, d)$ for decryption

**Encryption/Decryption**:
- Encrypt message $m$ to ciphertext $c$: $c = m^e \mod n$
- Decrypt ciphertext $c$ to message $m$: $m = c^d \mod n$

### 2.2 Mathematical Model

#### Number-Theoretic Foundation

1. **Modulus**: $n = p \times q$
2. **Euler's Totient**: $\phi(n) = (p-1)(q-1)$
3. **Public Exponent**: Choose $e$ such that $1 < e < \phi(n)$ and $\gcd(e, \phi(n)) = 1$
4. **Private Exponent**: $d = e^{-1} \mod \phi(n)$ (modular multiplicative inverse)

#### Key Property

The security relies on the difficulty of the **integer factorization problem**:
Given $n = p \times q$, finding $p$ and $q$ is computationally infeasible for large primes.

### 2.3 Correctness Proof

**Theorem**: For any message $m < n$:
$$m^{ed} \equiv m \pmod{n}$$

**Proof** (using Euler's theorem):
1. By construction: $ed \equiv 1 \pmod{\phi(n)}$
2. Therefore: $ed = k \cdot \phi(n) + 1$ for some integer $k$
3. By Euler's theorem: $m^{\phi(n)} \equiv 1 \pmod{n}$ when $\gcd(m, n) = 1$
4. Thus: $m^{ed} = m^{k \cdot \phi(n) + 1} = (m^{\phi(n)})^k \cdot m \equiv 1^k \cdot m \equiv m \pmod{n}$

## 3. Algorithm Description

### 3.1 Intuition

RSA works like a padlock:
- **Public key** = Open padlock (anyone can lock)
- **Private key** = The key to open it (only owner can unlock)

The mathematical "padlock" is the difficulty of factoring large numbers. Multiplying two primes is easy; finding the original primes from their product is hard.

### 3.2 Pseudocode

```
FUNCTION GenerateKeyPair(p, q):
    n ← p × q
    φ ← (p - 1) × (q - 1)
    
    // Find e coprime to φ
    e ← 2
    WHILE gcd(e, φ) ≠ 1:
        e ← e + 1
    
    // Compute modular inverse
    d ← ModularInverse(e, φ)
    
    RETURN (PublicKey(n, e), PrivateKey(n, d))

FUNCTION Encrypt(message, public_key):
    RETURN ModularExponentiation(message, public_key.e, public_key.n)

FUNCTION Decrypt(ciphertext, private_key):
    RETURN ModularExponentiation(ciphertext, private_key.d, private_key.n)

FUNCTION ModularExponentiation(base, exp, mod):
    result ← 1
    base ← base MOD mod
    WHILE exp > 0:
        IF exp is odd:
            result ← (result × base) MOD mod
        exp ← exp >> 1
        base ← (base × base) MOD mod
    RETURN result
```

### 3.3 Step-by-Step Example

**Given**: $p = 61$, $q = 53$

1. **Compute n**: $n = 61 \times 53 = 3233$

2. **Compute φ(n)**: $\phi(n) = 60 \times 52 = 3120$

3. **Choose e**: First $e$ where $\gcd(e, 3120) = 1$ → $e = 17$

4. **Compute d**: $d = 17^{-1} \mod 3120 = 2753$

**Keys**:
- Public: $(3233, 17)$
- Private: $(3233, 2753)$

**Encrypt message m = 65**:
$$c = 65^{17} \mod 3233 = 2790$$

**Decrypt ciphertext c = 2790**:
$$m = 2790^{2753} \mod 3233 = 65$$

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Key Generation | O(k³) | k = key bits, dominated by primality testing |
| Encryption | O(k²) | Single modular exponentiation |
| Decryption | O(k³) | With CRT optimization: O(k²) |
| Modular Exponentiation | O(log e × k²) | Square-and-multiply |

**Detailed Analysis**:
- GCD computation: O(log min(a,b)) iterations
- Extended Euclidean: O(log n) iterations
- Modular exponentiation: O(log e) multiplications, each O(k²)

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Keys | O(k) bits |
| Intermediate values | O(k) bits |
| Total | O(k) |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
/// Public key containing (n, e)
pub struct PublicKey {
    pub n: u64,  // Modulus
    pub e: u64,  // Public exponent
}

/// Private key containing (n, d)
pub struct PrivateKey {
    pub n: u64,  // Modulus
    pub d: u64,  // Private exponent
}
```

**Implementation details**:
- Uses `u128` for intermediate calculations to prevent overflow
- Extended Euclidean algorithm for modular inverse
- Square-and-multiply for efficient exponentiation

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| p = q | Should be rejected (reduces security) |
| Message ≥ n | Must split message or use hybrid encryption |
| e not coprime to φ | Search continues until valid e found |
| Non-prime inputs | Undefined behavior (caller responsibility) |

### 5.3 Critical Limitations

1. **Small key sizes**: This implementation uses u64, limiting practical key sizes
2. **No padding**: Vulnerable to:
   - Chosen plaintext attacks
   - Malleability attacks
   - Small message attacks
3. **Deterministic**: Same message always produces same ciphertext
4. **No primality testing**: Assumes inputs are prime

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

| Domain | Application |
|--------|-------------|
| TLS/SSL | Key exchange in HTTPS |
| Digital Signatures | Code signing, document signing |
| Secure Email | PGP/GPG encryption |
| Authentication | SSH key authentication |
| Cryptocurrency | Transaction signing |

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| DSA | Alternative signature algorithm |
| ECDSA | Elliptic curve variant (smaller keys) |
| ElGamal | Alternative public-key encryption |
| Diffie-Hellman | Key exchange (RSA can also do this) |
| RSA-OAEP | RSA with Optimal Asymmetric Encryption Padding |

## 7. Security Analysis

### 7.1 Recommended Key Sizes (2024)

| Key Size | Security Level | Use Case |
|----------|---------------|----------|
| 1024 bits | Deprecated | Legacy only |
| 2048 bits | ~112 bits | Minimum for general use |
| 3072 bits | ~128 bits | Medium-term security |
| 4096 bits | ~140 bits | Long-term security |

### 7.2 Known Vulnerabilities

| Attack | Mitigation |
|--------|------------|
| Factorization | Use large primes (≥1024 bits each) |
| Timing attacks | Constant-time implementation |
| Bleichenbacher attack | Use OAEP padding |
| Small exponent attack | Ensure message > n^(1/e) |
| Common modulus attack | Never share n between users |

### 7.3 Production Requirements

1. **Use OAEP padding** (PKCS#1 v2.1)
2. **Minimum 2048-bit keys**
3. **Cryptographically secure random primes**
4. **Constant-time operations**
5. **Chinese Remainder Theorem for decryption**

## 8. References

- Rivest, R., Shamir, A., & Adleman, L. (1978). A Method for Obtaining Digital Signatures and Public-Key Cryptosystems
- PKCS #1: RSA Cryptography Specifications (RFC 8017)
- NIST SP 800-56B: Recommendation for Pair-Wise Key Establishment
- [Implementation](../../src/ciphers/rsa_cipher.rs)
