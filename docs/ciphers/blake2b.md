# BLAKE2b

## 1. Overview

**BLAKE2b** is a cryptographic hash function optimized for 64-bit platforms, designed to be faster than MD5 while providing security comparable to SHA-3. It's an improved version of BLAKE, a SHA-3 finalist, standardized in RFC 7693.

### Historical Context
- **2008**: BLAKE submitted to SHA-3 competition
- **2012**: BLAKE2 released (optimized, not standardized)
- **2015**: RFC 7693 standardizes BLAKE2
- **Present**: Widely used in Argon2, libsodium, WireGuard

### Variants

| Variant | Word Size | Max Output | Optimized For |
|---------|-----------|------------|---------------|
| BLAKE2b | 64 bits | 512 bits | 64-bit platforms |
| BLAKE2s | 32 bits | 256 bits | 32-bit platforms |
| BLAKE2bp | 64 bits | 512 bits | Parallel (4 ways) |
| BLAKE2sp | 32 bits | 256 bits | Parallel (8 ways) |

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- Message $m$ (0 to 2^128 - 1 bytes)
- Key $k$ (0 to 64 bytes, optional)
- Output length $nn$ (1 to 64 bytes)

**Output**: Hash of length $nn$ bytes

### 2.2 Core Primitives

#### Mixing Function G

The core operation mixing four words:

```
G(a, b, c, d, x, y):
    a = a + b + x
    d = (d ⊕ a) >>> 32
    c = c + d
    b = (b ⊕ c) >>> 24
    a = a + b + y
    d = (d ⊕ a) >>> 16
    c = c + d
    b = (b ⊕ c) >>> 63
```

#### SIGMA Permutation

Message word scheduling over 12 rounds:
```
σ[0]  = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
σ[1]  = [14, 10, 4, 8, 9, 15, 13, 6, 1, 12, 0, 2, 11, 7, 5, 3]
σ[2]  = [11, 8, 12, 0, 5, 2, 15, 13, 10, 14, 3, 6, 7, 1, 9, 4]
...
```

### 2.3 Initialization Vector

Derived from fractional part of π:
```
IV[0] = 0x6A09E667F3BCC908
IV[1] = 0xBB67AE8584CAA73B
IV[2] = 0x3C6EF372FE94F82B
IV[3] = 0xA54FF53A5F1D36F1
IV[4] = 0x510E527FADE682D1
IV[5] = 0x9B05688C2B3E6C1F
IV[6] = 0x1F83D9ABFB41BD6B
IV[7] = 0x5BE0CD19137E2179
```

## 3. Algorithm Description

### 3.1 Intuition

BLAKE2 is like a high-speed blender that:
1. Sets up initial state based on parameters
2. Processes message in 128-byte blocks
3. Uses ARX (Add-Rotate-XOR) operations for speed
4. Applies different message schedules per round

The design prioritizes both security AND performance.

### 3.2 Pseudocode

```
FUNCTION BLAKE2b(message, key, output_length):
    // Initialize state
    h[0..7] ← IV[0..7]
    h[0] ← h[0] ⊕ 0x01010000 ⊕ (key_length << 8) ⊕ output_length
    
    // If keyed, prepend padded key as first block
    IF key_length > 0:
        block ← key || zeros(128 - key_length)
        message ← block || message
    
    // Process all complete blocks
    bytes_compressed ← 0
    WHILE remaining bytes ≥ 128:
        bytes_compressed += 128
        Compress(h, block, bytes_compressed, false)
    
    // Process final block (with padding)
    bytes_compressed += remaining_bytes
    block ← remaining_bytes || zeros(128 - remaining)
    Compress(h, block, bytes_compressed, true)  // last = true
    
    RETURN first output_length bytes of h

FUNCTION Compress(h, block, t, last):
    // Initialize working vector
    v[0..7] ← h[0..7]
    v[8..11] ← IV[0..3]
    v[12] ← IV[4] ⊕ (t mod 2^64)      // Low bits of counter
    v[13] ← IV[5] ⊕ (t >> 64)         // High bits of counter
    v[14] ← IF last THEN IV[6] ⊕ 0xFF..FF ELSE IV[6]
    v[15] ← IV[7]
    
    // Parse block as 16 words
    m[0..15] ← block as 64-bit words
    
    // 12 rounds of mixing
    FOR round FROM 0 TO 11:
        σ ← SIGMA[round mod 10]
        // Column step
        G(v[0], v[4], v[8],  v[12], m[σ[0]],  m[σ[1]])
        G(v[1], v[5], v[9],  v[13], m[σ[2]],  m[σ[3]])
        G(v[2], v[6], v[10], v[14], m[σ[4]],  m[σ[5]])
        G(v[3], v[7], v[11], v[15], m[σ[6]],  m[σ[7]])
        // Diagonal step
        G(v[0], v[5], v[10], v[15], m[σ[8]],  m[σ[9]])
        G(v[1], v[6], v[11], v[12], m[σ[10]], m[σ[11]])
        G(v[2], v[7], v[8],  v[13], m[σ[12]], m[σ[13]])
        G(v[3], v[4], v[9],  v[14], m[σ[14]], m[σ[15]])
    
    // Update hash state
    FOR i FROM 0 TO 7:
        h[i] ← h[i] ⊕ v[i] ⊕ v[i + 8]
```

### 3.3 Example

**BLAKE2b-256("abc")**:
```
bddd813c634239723171ef3fee98579b
94964e3bb1cb3e427262c8c068d52319
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Per round | O(1) | 16 G-function calls |
| Per block | O(12) | 12 rounds |
| Total | **O(n)** | n = message length |

**Operations per block**:
- 12 rounds × 8 G-calls × 12 operations = 1152 operations
- Highly parallelizable (8 G-calls can run in parallel)

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| State (h) | 64 bytes |
| Working vector (v) | 128 bytes |
| Message block (m) | 128 bytes |
| Total | **O(1)** constant |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
/// BLAKE2b initialization vector
const IV: [u64; 8] = [
    0x6a09e667f3bcc908, 0xbb67ae8584caa73b,
    0x3c6ef372fe94f82b, 0xa54ff53a5f1d36f1,
    0x510e527fade682d1, 0x9b05688c2b3e6c1f,
    0x1f83d9abfb41bd6b, 0x5be0cd19137e2179,
];

/// G mixing function
#[inline(always)]
fn g(v: &mut [u64; 16], a: usize, b: usize, c: usize, d: usize, x: u64, y: u64) {
    v[a] = v[a].wrapping_add(v[b]).wrapping_add(x);
    v[d] = (v[d] ^ v[a]).rotate_right(32);
    v[c] = v[c].wrapping_add(v[d]);
    v[b] = (v[b] ^ v[c]).rotate_right(24);
    v[a] = v[a].wrapping_add(v[b]).wrapping_add(y);
    v[d] = (v[d] ^ v[a]).rotate_right(16);
    v[c] = v[c].wrapping_add(v[d]);
    v[b] = (v[b] ^ v[c]).rotate_right(63);
}
```

**Key optimizations**:
- `#[inline(always)]` for G function
- Use `wrapping_add` for modular arithmetic
- `rotate_right` for rotations
- SIMD potential for parallel G calls

### 5.2 Parameter Block

```
Offset  Field           Bytes
0       Digest length   1
1       Key length      1
2       Fanout          1
3       Depth           1
4-7     Leaf length     4
8-15    Node offset     8
16      Node depth      1
17      Inner length    1
18-31   Reserved        14
32-47   Salt            16
48-63   Personalization 16
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty message, no key | Valid, produces hash |
| Key only | Acts as MAC |
| Zero output length | Invalid (min 1) |
| Output > 64 bytes | Invalid (max 64) |
| Non-multiple of 128 | Padded with zeros |

## 6. Security Analysis

### 6.1 Security Levels

| Output Size | Collision | Pre-image |
|-------------|-----------|-----------|
| 256 bits | 128 bits | 256 bits |
| 384 bits | 192 bits | 384 bits |
| 512 bits | 256 bits | 512 bits |

### 6.2 Advantages

| Feature | Benefit |
|---------|---------|
| No length extension | Unlike MD5/SHA-256 |
| Single-pass keyed hashing | Efficient MAC |
| Parallelizable | BLAKE2bp/sp variants |
| Personalization | Domain separation built-in |
| Salt support | Additional entropy |

### 6.3 Known Attacks

| Attack | Status |
|--------|--------|
| Cryptographic break | None known |
| Reduced-round attacks | Up to 2.5 rounds (academic) |
| Practical attacks | None |

## 7. Performance Comparison

### 7.1 Speed Benchmarks (64-bit)

| Algorithm | Cycles/byte | Relative |
|-----------|-------------|----------|
| MD5 | ~5 | 1.0x |
| **BLAKE2b** | ~3 | 1.7x faster |
| SHA-256 | ~15 | 0.3x slower |
| SHA-512 | ~10 | 0.5x slower |
| SHA3-256 | ~12 | 0.4x slower |

### 7.2 Why So Fast?

1. **ARX design**: Only add, rotate, XOR (fast on all CPUs)
2. **No S-boxes**: No table lookups (cache-timing safe)
3. **64-bit words**: Optimized for modern CPUs
4. **Parallelism**: 8 G-calls per round can be parallel
5. **Fewer rounds**: 12 vs SHA-256's 64

## 8. Real-World Applications

### 8.1 Use Cases

| Domain | Application |
|--------|-------------|
| Password hashing | Argon2 uses BLAKE2b |
| VPN | WireGuard uses BLAKE2s |
| Cryptographic libraries | libsodium default hash |
| File integrity | IPFS content addressing |
| Signatures | RAR5 authentication |

### 8.2 When to Use BLAKE2b

| Scenario | Recommendation |
|----------|----------------|
| Maximum speed | BLAKE2b |
| 32-bit platforms | BLAKE2s |
| Regulatory compliance | SHA-256/SHA-3 |
| Password hashing | Argon2 (uses BLAKE2b) |
| Variable-length output | BLAKE2X |

## 9. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| BLAKE | SHA-3 finalist predecessor |
| BLAKE2s | 32-bit optimized variant |
| BLAKE3 | Newer, faster (tree hashing) |
| ChaCha | Same ARX core design |
| Argon2 | Uses BLAKE2b internally |

## 10. References

- RFC 7693: The BLAKE2 Cryptographic Hash and MAC
- BLAKE2 Paper (Aumasson et al., 2013)
- FIPS 180-4: Secure Hash Standard (comparison)
- [Implementation](../../src/ciphers/blake2b.rs)
