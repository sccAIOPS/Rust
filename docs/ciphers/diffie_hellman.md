# Diffie-Hellman Key Exchange

## 1. Overview

**Diffie-Hellman** is a key exchange protocol that allows two parties to establish a shared secret over an insecure channel. Published in 1976 by Whitfield Diffie and Martin Hellman, it was the first practical method for establishing a shared secret over an unprotected communications channel.

### Historical Context
- **1976**: Published in "New Directions in Cryptography"
- **1977**: RSA algorithm developed (enabled actual encryption)
- **2015**: Logjam attack reveals weaknesses in 1024-bit DH
- **Present**: Foundation for TLS, SSH, IPsec key exchange

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Goal**: Two parties (Alice and Bob) want to agree on a shared secret key without a prior secret.

**Given** (public knowledge):
- Prime $p$ (large)
- Generator $g$ (primitive root mod $p$)

**Produce**: Shared secret $s$ known only to Alice and Bob

### 2.2 Mathematical Model

#### Discrete Logarithm Problem (DLP)

**Easy**: Given $g$, $a$, $p$: compute $g^a \mod p$
**Hard**: Given $g$, $g^a \mod p$, $p$: find $a$

This asymmetry is the foundation of DH security.

#### Protocol Steps

1. **Alice**: Generate private $a$, compute $A = g^a \mod p$, send $A$
2. **Bob**: Generate private $b$, compute $B = g^b \mod p$, send $B$
3. **Alice**: Compute $s = B^a \mod p = g^{ab} \mod p$
4. **Bob**: Compute $s = A^b \mod p = g^{ab} \mod p$

Both arrive at the same secret $s = g^{ab} \mod p$!

### 2.3 Group Theory Foundation

DH operates in the multiplicative group $\mathbb{Z}_p^*$:
- Elements: $\{1, 2, ..., p-1\}$
- Operation: Multiplication modulo $p$
- Generator: $g$ such that $\{g^1, g^2, ..., g^{p-1}\} = \mathbb{Z}_p^*$

## 3. Algorithm Description

### 3.1 Intuition

**Paint mixing analogy**:
1. Alice and Bob agree on a common paint (yellow)
2. Each adds their secret color (Alice: red, Bob: blue)
3. They exchange mixed paints (orange, green)
4. Each adds their secret to received paint
5. Both get the same final color!

An observer sees yellow, orange, and green, but cannot derive the final color without knowing one secret color.

### 3.2 Pseudocode

```
SETUP:
    p ← LargePrime()          // Public
    g ← PrimitiveRoot(p)       // Public

ALICE:
    a ← RandomInteger(2, p-2)  // Private
    A ← ModExp(g, a, p)        // Send to Bob

BOB:
    b ← RandomInteger(2, p-2)  // Private
    B ← ModExp(g, b, p)        // Send to Alice

SHARED SECRET:
    Alice: s ← ModExp(B, a, p) = g^(ab) mod p
    Bob:   s ← ModExp(A, b, p) = g^(ab) mod p
```

### 3.3 Step-by-Step Example

**Public parameters**: $p = 23$, $g = 5$

| Step | Alice | Bob |
|------|-------|-----|
| Choose private | $a = 6$ | $b = 15$ |
| Compute public | $A = 5^6 \mod 23 = 8$ | $B = 5^{15} \mod 23 = 19$ |
| Exchange | Alice receives $B = 19$ | Bob receives $A = 8$ |
| Compute secret | $s = 19^6 \mod 23 = 2$ | $s = 8^{15} \mod 23 = 2$ |

**Shared secret**: $s = 2$

**Verification**: $5^{6 \times 15} \mod 23 = 5^{90} \mod 23 = 2$ ✓

## 4. Implementation Notes

### 4.1 RFC 3526 Standard Primes

The implementation uses standard MODP groups from RFC 3526:

| Group | Bits | Use Case |
|-------|------|----------|
| Group 5 | 1536 | Legacy (minimum) |
| Group 14 | 2048 | General use |
| Group 15 | 3072 | Enhanced security |
| Group 16 | 4096 | High security |
| Group 17 | 6144 | Long-term |
| Group 18 | 8192 | Maximum |

```rust
/// Get RFC 3526 prime for given bit size
pub fn rfc3526_prime(bits: u32) -> Option<BigUint> {
    match bits {
        1536 => Some(MODP_1536_PRIME.clone()),
        2048 => Some(MODP_2048_PRIME.clone()),
        // ... more groups
    }
}
```

### 4.2 Rust-Specific Considerations

```rust
pub struct DiffieHellman {
    prime: BigUint,       // The prime modulus p
    generator: BigUint,   // The generator g
    private_key: BigUint, // Private exponent
    public_key: BigUint,  // g^private mod p
}
```

**Key features**:
- Uses `num-bigint` for arbitrary precision
- Cryptographically secure random number generation required (external)
- Immutable public parameters after construction

### 4.3 Edge Cases

| Case | Handling |
|------|----------|
| Private key = 0 | Invalid (identity element) |
| Private key = 1 | Weak (public = generator) |
| Private key ≥ p-1 | Should be rejected |
| Non-prime p | Security compromised |
| g not generator | Reduced security |

## 5. Complexity Analysis

### 5.1 Time Complexity

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Public key generation | O(k³) | k-bit modular exponentiation |
| Shared secret computation | O(k³) | Same as above |
| Parameter verification | O(k⁴) | Primality testing |

### 5.2 Space Complexity

| Component | Space |
|-----------|-------|
| Prime p | O(k) bits |
| Private key | O(k) bits |
| Public key | O(k) bits |
| Total per party | O(k) bits |

## 6. Security Analysis

### 6.1 Attack Vectors

| Attack | Description | Mitigation |
|--------|-------------|------------|
| Man-in-the-Middle | Attacker intercepts and replaces public keys | Authenticate public keys |
| Logjam | Precomputation for weak groups | Use ≥2048-bit primes |
| Small subgroup | Confining shared secret to small subgroup | Validate received values |
| Pohlig-Hellman | Attack on smooth group order | Use safe primes |

### 6.2 Recommended Parameters (2024)

| Security Level | Prime Size | Notes |
|----------------|------------|-------|
| 80 bits | 1024 bits | Deprecated |
| 112 bits | 2048 bits | Minimum recommended |
| 128 bits | 3072 bits | Medium-term security |
| 192 bits | 7680 bits | Long-term security |

### 6.3 Comparison with ECDH

| Property | DH | ECDH |
|----------|-----|------|
| Key size (128-bit security) | 3072 bits | 256 bits |
| Speed | Slower | Faster |
| Standardization | Wide | Growing |
| Implementation complexity | Simpler | More complex |

## 7. Real-World Applications

### 7.1 Protocol Usage

| Protocol | DH Variant | Purpose |
|----------|------------|---------|
| TLS 1.2 | DHE, ECDHE | Perfect Forward Secrecy |
| TLS 1.3 | ECDHE only | Key establishment |
| SSH | diffie-hellman-group14 | Key exchange |
| IPsec/IKE | MODP groups | VPN key negotiation |
| Signal Protocol | X3DH | Asynchronous key agreement |

### 7.2 Perfect Forward Secrecy

DH enables **ephemeral key exchange**:
- Generate new key pair for each session
- Compromise of long-term keys doesn't reveal past sessions
- Critical for modern TLS deployments

## 8. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| ECDH | Elliptic curve variant (smaller keys) |
| RSA Key Exchange | Alternative (no forward secrecy) |
| ElGamal | Encryption based on DH problem |
| DSA | Signatures based on DH problem |
| X25519 | Modern ECDH curve |

## 9. References

- Diffie, W., & Hellman, M. (1976). New Directions in Cryptography
- RFC 3526: More Modular Exponential (MODP) Diffie-Hellman groups
- RFC 7919: Negotiated Finite Field Diffie-Hellman Ephemeral Parameters
- NIST SP 800-56A: Recommendation for Pair-Wise Key Establishment Schemes
- [Implementation](../../src/ciphers/diffie_hellman.rs)
