# ChaCha20

## 1. Overview

**ChaCha20** is a stream cipher designed by Daniel J. Bernstein as a variant of Salsa20. It offers the same security margins with improved diffusion per round, making it the preferred choice for modern cryptographic protocols.

### Historical Context
- **2005**: Salsa20 published
- **2008**: ChaCha variant introduced
- **2014**: Google adopts ChaCha20-Poly1305 for TLS
- **2015**: RFC 7539 standardizes ChaCha20-Poly1305
- **Present**: Default cipher in TLS 1.3 for mobile devices

### Why ChaCha Over AES?

| Property | ChaCha20 | AES |
|----------|----------|-----|
| Software speed (no AES-NI) | Very fast | Slow |
| Timing attacks | Immune | Requires care |
| Mobile performance | Excellent | Needs hardware |
| SIMD optimization | Natural | Complex |

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- 256-bit key $K$
- 96-bit nonce $N$
- 32-bit counter $C$

**Output**: Keystream of 64-byte blocks

**Encryption**: $Ciphertext = Plaintext \oplus Keystream$

### 2.2 Quarter Round

The core operation on four 32-bit words:

$$\text{QR}(a, b, c, d):$$
$$a = a + b; \quad d = (d \oplus a) \lll 16$$
$$c = c + d; \quad b = (b \oplus c) \lll 12$$
$$a = a + b; \quad d = (d \oplus a) \lll 8$$
$$c = c + d; \quad b = (b \oplus c) \lll 7$$

**Key properties**:
- ARX design (Add-Rotate-XOR only)
- Invertible (for proof of correctness)
- No S-boxes (constant-time)

### 2.3 State Matrix

16 32-bit words arranged as 4×4 matrix:

```
     0   1   2   3
   ┌───┬───┬───┬───┐
 0 │ C │ C │ C │ C │  Constants "expand 32-byte k"
   ├───┼───┼───┼───┤
 1 │ K │ K │ K │ K │  Key (words 0-3)
   ├───┼───┼───┼───┤
 2 │ K │ K │ K │ K │  Key (words 4-7)
   ├───┼───┼───┼───┤
 3 │ B │ N │ N │ N │  Block counter + Nonce
   └───┴───┴───┴───┘
```

**Constants** (ASCII): `"expand 32-byte k"`
```
0x61707865, 0x3320646e, 0x79622d32, 0x6b206574
```

## 3. Algorithm Description

### 3.1 Intuition

ChaCha20 is like a shuffle machine:
1. Set up initial state from key, nonce, counter
2. Apply 20 rounds of shuffling (quarter rounds)
3. Add initial state to shuffled state
4. Output 64 bytes of keystream
5. Increment counter, repeat for more keystream

The alternating column and diagonal rounds ensure rapid diffusion.

### 3.2 Round Structure

```
Column Round:
    QR(0, 4,  8, 12)    QR(1, 5,  9, 13)
    QR(2, 6, 10, 14)    QR(3, 7, 11, 15)

Diagonal Round:
    QR(0, 5, 10, 15)    QR(1, 6, 11, 12)
    QR(2, 7,  8, 13)    QR(3, 4,  9, 14)
```

**ChaCha20**: 10 column rounds + 10 diagonal rounds = 20 rounds

### 3.3 Pseudocode

```
FUNCTION ChaCha20_block(key, counter, nonce):
    // Initialize state
    state[0..3]   ← CONSTANTS
    state[4..11]  ← key[0..7]           // 256 bits
    state[12]     ← counter              // 32 bits
    state[13..15] ← nonce[0..2]          // 96 bits
    
    working ← copy(state)
    
    // 20 rounds (10 double-rounds)
    FOR i FROM 0 TO 9:
        // Column round
        QuarterRound(working, 0, 4,  8, 12)
        QuarterRound(working, 1, 5,  9, 13)
        QuarterRound(working, 2, 6, 10, 14)
        QuarterRound(working, 3, 7, 11, 15)
        // Diagonal round
        QuarterRound(working, 0, 5, 10, 15)
        QuarterRound(working, 1, 6, 11, 12)
        QuarterRound(working, 2, 7,  8, 13)
        QuarterRound(working, 3, 4,  9, 14)
    
    // Add initial state
    FOR i FROM 0 TO 15:
        working[i] ← working[i] + state[i]
    
    RETURN working as 64 bytes (little-endian)

FUNCTION ChaCha20_encrypt(key, nonce, plaintext):
    counter ← 1
    ciphertext ← []
    
    FOR each 64-byte block in plaintext:
        keystream ← ChaCha20_block(key, counter, nonce)
        ciphertext ← ciphertext || (block ⊕ keystream)
        counter ← counter + 1
    
    RETURN ciphertext
```

### 3.4 Example

**Key**: All zeros (32 bytes)
**Nonce**: All zeros (12 bytes)
**Counter**: 1

**First keystream block (first 64 bytes)**:
```
76b8e0ada0f13d90 405d6ae55386bd28
bdd219b8a08ded1a a836efcc8b770dc7
da41597c5157488d 7724e03fb8d84a37
6a43b8f41518a11c c387b669b2ee6586
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Quarter round | O(1) | 16 operations |
| Per block | O(1) | 80 quarter rounds |
| Total | **O(n)** | n = message length |

**Operations per 64-byte block**:
- 20 rounds × 4 quarter rounds = 80 QRs
- 80 × 16 = 1280 ARX operations
- ~20 operations/byte

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| State | 64 bytes |
| Working copy | 64 bytes |
| Total | **O(1)** constant |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
/// ChaCha20 quarter round
macro_rules! quarter_round {
    ($a:expr, $b:expr, $c:expr, $d:expr) => {
        $a = $a.wrapping_add($b); $d = ($d ^ $a).rotate_left(16);
        $c = $c.wrapping_add($d); $b = ($b ^ $c).rotate_left(12);
        $a = $a.wrapping_add($b); $d = ($d ^ $a).rotate_left(8);
        $c = $c.wrapping_add($d); $b = ($b ^ $c).rotate_left(7);
    };
}
```

**Implementation features**:
- Macro for quarter round (inlines efficiently)
- `wrapping_add` for modular arithmetic
- `rotate_left` for rotations
- Little-endian byte conversion
- SIMD-friendly structure

### 5.2 Variants

| Variant | Nonce | Counter | Max Data |
|---------|-------|---------|----------|
| Original ChaCha20 | 64 bits | 64 bits | 2^70 bytes |
| IETF ChaCha20 | 96 bits | 32 bits | 256 GB |
| XChaCha20 | 192 bits | 64 bits | 2^70 bytes |

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty plaintext | Valid, returns empty |
| Counter overflow | Implementation-dependent |
| Nonce reuse | CRITICAL: Breaks security |
| Partial block | XOR only needed bytes |

## 6. Security Analysis

### 6.1 Security Level

- **Key size**: 256 bits
- **Effective security**: 256 bits
- **Best known attack**: Exhaustive key search

### 6.2 Critical Requirements

| Requirement | Consequence if Violated |
|-------------|------------------------|
| Never reuse nonce | Full plaintext recovery |
| Random/unique nonce | Collision → key recovery |
| Authenticate ciphertext | Use ChaCha20-Poly1305 |

### 6.3 Nonce Uniqueness Strategies

| Strategy | Pros | Cons |
|----------|------|------|
| Counter | Simple, guaranteed unique | State required |
| Random (96-bit) | Stateless | 2^48 limit before collision |
| Random (192-bit XChaCha) | Safe for random | Slightly slower |

### 6.4 Known Attacks

| Attack | Rounds Broken | Status |
|--------|--------------|--------|
| Differential | 6/20 | Academic |
| Linear | 7/20 | Academic |
| Full ChaCha20 | None | Secure |

## 7. ChaCha20-Poly1305

### 7.1 AEAD Construction

**Poly1305** provides authentication:
```
Encrypt:
    keystream ← ChaCha20(key, nonce, counter=0)[0:32]  // Poly1305 key
    ciphertext ← ChaCha20_encrypt(key, nonce, plaintext, counter=1)
    tag ← Poly1305(keystream, AAD || ciphertext)
    Return (ciphertext, tag)

Decrypt:
    Verify tag first!
    plaintext ← ChaCha20_decrypt(key, nonce, ciphertext)
    Return plaintext
```

### 7.2 Usage in Protocols

| Protocol | Configuration |
|----------|---------------|
| TLS 1.3 | AEAD_CHACHA20_POLY1305 |
| WireGuard | ChaCha20-Poly1305 |
| SSH | chacha20-poly1305@openssh.com |
| NaCl/libsodium | crypto_secretbox |

## 8. Real-World Applications

### 8.1 Use Cases

| Domain | Application |
|--------|-------------|
| Mobile encryption | TLS on ARM devices |
| VPN | WireGuard, IPsec |
| Secure messaging | Signal Protocol |
| Disk encryption | Some implementations |
| Random generation | ChaCha-based CSPRNG |

### 8.2 When to Use ChaCha20

| Scenario | Recommendation |
|----------|----------------|
| No AES hardware | ChaCha20 |
| Mobile devices | ChaCha20 |
| AES-NI available | AES-GCM (similar speed) |
| Side-channel concerns | ChaCha20 |
| Key agility needed | ChaCha20 (faster key setup) |

## 9. Comparison with AES

| Property | ChaCha20 | AES-256-GCM |
|----------|----------|-------------|
| Key size | 256 bits | 256 bits |
| Block size | 64 bytes | 16 bytes |
| Software speed | ~3 cycles/byte | ~15 cycles/byte |
| Hardware speed | Similar | Similar |
| Side-channel safety | Built-in | Needs care |
| Parallelizable | Yes | Yes (CTR/GCM) |

## 10. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Salsa20 | Predecessor (different rotation) |
| XSalsa20/XChaCha20 | Extended nonce variants |
| Poly1305 | Companion MAC |
| BLAKE2 | Same ARX core |

## 11. References

- RFC 7539: ChaCha20 and Poly1305 for IETF Protocols
- RFC 8439: ChaCha20 and Poly1305 (updates 7539)
- Bernstein, D.J. (2008). ChaCha, a variant of Salsa20
- [Implementation](../../src/ciphers/chacha.rs)
