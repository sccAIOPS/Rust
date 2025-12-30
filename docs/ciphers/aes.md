# AES (Advanced Encryption Standard)

## 1. Overview

The **Advanced Encryption Standard (AES)**, also known as Rijndael, is a symmetric block cipher established by the U.S. National Institute of Standards and Technology (NIST) in 2001. It replaced DES as the federal standard for encrypting sensitive data and is now the most widely used symmetric encryption algorithm worldwide.

### Historical Context
- **1997**: NIST announced a competition to replace DES
- **1998**: 15 algorithms submitted for consideration
- **2000**: Rijndael (by Joan Daemen and Vincent Rijmen) selected as winner
- **2001**: Published as FIPS 197

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- Plaintext block $P$ of 128 bits
- Key $K$ of 128, 192, or 256 bits

Produce:
- Ciphertext block $C$ of 128 bits

Such that decryption with the same key recovers the original plaintext.

### 2.2 Mathematical Model

AES operates on a $4 \times 4$ column-major order matrix of bytes called the **state**:

$$
\text{State} = \begin{bmatrix}
s_{0,0} & s_{0,1} & s_{0,2} & s_{0,3} \\
s_{1,0} & s_{1,1} & s_{1,2} & s_{1,3} \\
s_{2,0} & s_{2,1} & s_{2,2} & s_{2,3} \\
s_{3,0} & s_{3,1} & s_{3,2} & s_{3,3}
\end{bmatrix}
$$

Operations are performed in the Galois Field $GF(2^8)$ with the irreducible polynomial:
$$m(x) = x^8 + x^4 + x^3 + x + 1$$

### 2.3 Key Properties

| Key Size | Block Size | Rounds |
|----------|------------|--------|
| 128 bits | 128 bits | 10 |
| 192 bits | 128 bits | 12 |
| 256 bits | 128 bits | 14 |

## 3. Algorithm Description

### 3.1 Intuition

AES encrypts data through multiple rounds of four operations:
1. **SubBytes**: Non-linear substitution using S-box
2. **ShiftRows**: Cyclic shifting of rows
3. **MixColumns**: Column mixing using matrix multiplication
4. **AddRoundKey**: XOR with round key

Each operation provides specific security properties:
- SubBytes → Confusion (non-linearity)
- ShiftRows → Diffusion (spreading)
- MixColumns → Diffusion (mixing)
- AddRoundKey → Key dependency

### 3.2 Pseudocode

```
FUNCTION AES_Encrypt(plaintext, key):
    state ← plaintext_to_state(plaintext)
    round_keys ← KeyExpansion(key)
    
    // Initial round
    state ← AddRoundKey(state, round_keys[0])
    
    // Main rounds
    FOR round = 1 TO Nr-1:
        state ← SubBytes(state)
        state ← ShiftRows(state)
        state ← MixColumns(state)
        state ← AddRoundKey(state, round_keys[round])
    
    // Final round (no MixColumns)
    state ← SubBytes(state)
    state ← ShiftRows(state)
    state ← AddRoundKey(state, round_keys[Nr])
    
    RETURN state_to_ciphertext(state)
```

### 3.3 Step-by-Step Example

For AES-128 with key `2b7e151628aed2a6abf7158809cf4f3c`:

**Input plaintext**: `3243f6a8885a308d313198a2e0370734`

**Round 0 (AddRoundKey only)**:
```
State XOR Key:
32 88 31 e0     2b 28 ab 09     19 a0 9a e9
43 5a 31 37  ⊕  7e ae f7 cf  =  3d f4 c6 f8
f6 30 98 07     15 d2 15 4f     e3 e2 8d 48
a8 8d a2 34     16 a6 88 3c     be 2b 2a 08
```

**Rounds 1-9**: SubBytes → ShiftRows → MixColumns → AddRoundKey

**Round 10 (Final)**: SubBytes → ShiftRows → AddRoundKey

**Output ciphertext**: `3925841d02dc09fbdc118597196a0b32`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Key Expansion | O(k) | k = key size in words |
| Single Block Encryption | O(1) | Fixed operations per block |
| n-Block Encryption | O(n) | Linear in plaintext size |

**Per-round operations**:
- SubBytes: 16 table lookups = O(1)
- ShiftRows: 12 byte moves = O(1)
- MixColumns: 16 GF multiplications = O(1)
- AddRoundKey: 16 XOR operations = O(1)

**Total**: O(n) where n is the number of bytes to encrypt

### 4.2 Space Complexity

| Component | Space | Notes |
|-----------|-------|-------|
| S-box | 256 bytes | Precomputed lookup table |
| Inverse S-box | 256 bytes | For decryption |
| Round constants | 256 bytes | RCON values |
| GF multiplication tables | 4 KB | Optional optimization |
| Expanded key | 176-240 bytes | Depends on key size |
| State | 16 bytes | Current processing block |

**Total**: O(1) auxiliary space (tables are constant size)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Type aliases for clarity
type Byte = u8;
type Word = u32;
type AesWord = [Byte; 4];

// Key enum for type-safe key sizes
pub enum AesKey {
    AesKey128([Byte; 16]),
    AesKey192([Byte; 24]),
    AesKey256([Byte; 32]),
}
```

**Key Rust patterns used**:
- Pattern matching for key size selection
- Iterator chains for byte transformations
- Const arrays for lookup tables
- `chunks_mut` for in-place block operations

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty input | Returns empty Vec |
| Non-16-byte aligned input | Zero-padded to block boundary |
| Invalid key size | Compile-time enforcement via enum |

### 5.3 Potential Pitfalls

1. **No padding scheme**: The implementation uses zero-padding which doesn't support arbitrary decryption
2. **ECB mode only**: Each block encrypted independently (not recommended for production)
3. **No authentication**: Vulnerable to bit-flipping attacks without MAC

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

| Domain | Application |
|--------|-------------|
| Web Security | TLS/SSL encryption |
| Storage | Full-disk encryption (BitLocker, FileVault) |
| Databases | Transparent Data Encryption (TDE) |
| Communications | VPN tunnels, secure messaging |
| Hardware | Secure boot, HSMs, smart cards |

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| DES | Predecessor (64-bit blocks, now deprecated) |
| 3DES | Triple-DES, slower but backward compatible |
| ChaCha20 | Alternative stream cipher, often faster in software |
| AES-GCM | AES with Galois Counter Mode (authenticated) |
| AES-CTR | AES in Counter mode (stream cipher behavior) |

## 7. Security Analysis

### 7.1 Known Attack Resistance

| Attack Type | Status |
|-------------|--------|
| Brute Force | Secure (2^128 - 2^256 operations) |
| Differential Cryptanalysis | Designed to resist |
| Linear Cryptanalysis | Designed to resist |
| Related-Key Attack | Theoretical weaknesses in AES-256 |
| Side-Channel | Implementation dependent |

### 7.2 Best Practices

1. **Use authenticated modes** (GCM, CCM) for production
2. **Never reuse nonces** in CTR/GCM modes
3. **Use constant-time implementations** to prevent timing attacks
4. **Key derivation**: Use PBKDF2/Argon2 for password-based keys

## 8. References

- FIPS 197: Advanced Encryption Standard (AES)
- Daemen, J., & Rijmen, V. (2002). *The Design of Rijndael*
- NIST SP 800-38A: Recommendation for Block Cipher Modes of Operation
- [Implementation](../../src/ciphers/aes.rs)
