# Salsa20

## 1. Overview

**Salsa20** is a stream cipher designed by Daniel J. Bernstein in 2005 and submitted to the eSTREAM project. It generates a pseudorandom keystream that is XORed with plaintext to produce ciphertext.

### Historical Context
- **2005**: Salsa20 published
- **2008**: Selected for eSTREAM portfolio
- **2008**: ChaCha variant introduced (improved diffusion)
- **Present**: Foundation for ChaCha20 in TLS 1.3

### Variants

| Variant | Rounds | Speed | Security |
|---------|--------|-------|----------|
| Salsa20/8 | 8 | Fastest | Marginal |
| Salsa20/12 | 12 | Fast | Good |
| **Salsa20/20** | 20 | Standard | Full |

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- 256-bit key $K$ (or 128-bit expanded)
- 64-bit nonce $N$
- 64-bit stream position

**Output**: 64-byte keystream block

### 2.2 Quarter Round

The fundamental operation on four words $(a, b, c, d)$:

$$b = b \oplus ((a + d) \lll 7)$$
$$c = c \oplus ((b + a) \lll 9)$$
$$d = d \oplus ((c + b) \lll 13)$$
$$a = a \oplus ((d + c) \lll 18)$$

**Key difference from ChaCha**: Uses different rotation amounts.

### 2.3 State Matrix

```
     0   1   2   3
   ┌───┬───┬───┬───┐
 0 │ C │ K │ K │ K │  C=constant, K=key
   ├───┼───┼───┼───┤
 1 │ K │ C │ N │ N │  N=nonce
   ├───┼───┼───┼───┤
 2 │ P │ P │ C │ K │  P=position
   ├───┼───┼───┼───┤
 3 │ K │ K │ K │ C │  
   └───┴───┴───┴───┘
```

**Constants** ("expand 32-byte k"):
- Position 0: 0x61707865
- Position 5: 0x3320646e
- Position 10: 0x79622d32
- Position 15: 0x6b206574

## 3. Algorithm Description

### 3.1 Intuition

Salsa20 scrambles a 64-byte state matrix:
1. Initialize with key, nonce, position, constants
2. Apply column rounds (mix columns)
3. Apply row rounds (mix rows)
4. Add initial state back (makes it one-way)
5. Output as keystream

### 3.2 Round Structure

**Column Round** (operates on columns):
```
QuarterRound(x[0], x[4], x[8],  x[12])
QuarterRound(x[5], x[9], x[13], x[1])
QuarterRound(x[10], x[14], x[2], x[6])
QuarterRound(x[15], x[3], x[7], x[11])
```

**Row Round** (operates on rows):
```
QuarterRound(x[0], x[1], x[2], x[3])
QuarterRound(x[5], x[6], x[7], x[4])
QuarterRound(x[10], x[11], x[8], x[9])
QuarterRound(x[15], x[12], x[13], x[14])
```

**Double Round** = Column Round + Row Round

### 3.3 Pseudocode

```
FUNCTION Salsa20_hash(input[16]):
    x ← copy(input)
    
    // 20 rounds = 10 double rounds
    FOR i FROM 1 TO 10:
        // Column round
        x[4]  ^= rotl(x[0]  + x[12], 7)
        x[8]  ^= rotl(x[4]  + x[0],  9)
        x[12] ^= rotl(x[8]  + x[4],  13)
        x[0]  ^= rotl(x[12] + x[8],  18)
        // ... remaining columns
        
        // Row round
        x[1]  ^= rotl(x[0]  + x[3],  7)
        x[2]  ^= rotl(x[1]  + x[0],  9)
        x[3]  ^= rotl(x[2]  + x[1],  13)
        x[0]  ^= rotl(x[3]  + x[2],  18)
        // ... remaining rows
    
    // Add input (feedforward)
    FOR i FROM 0 TO 15:
        x[i] ← x[i] + input[i]
    
    RETURN x

FUNCTION Salsa20_expand(key, nonce, position):
    // Setup input block
    input[0]  ← 0x61707865  // "expa"
    input[1..4] ← key[0..3]
    input[5]  ← 0x3320646e  // "nd 3"
    input[6..7] ← nonce
    input[8..9] ← position
    input[10] ← 0x79622d32  // "2-by"
    input[11..14] ← key[4..7]
    input[15] ← 0x6b206574  // "te k"
    
    RETURN Salsa20_hash(input)

FUNCTION Salsa20_encrypt(key, nonce, plaintext):
    ciphertext ← []
    position ← 0
    
    WHILE plaintext remaining:
        block ← Salsa20_expand(key, nonce, position)
        ciphertext.append(plaintext ⊕ block)
        position ← position + 1
    
    RETURN ciphertext
```

### 3.4 Example

**Key**: 32 zero bytes
**Nonce**: 8 zero bytes
**Position**: 0

**First block output (hex, first 64 bytes)**:
```
9a97f65b9b4c721b 960a672145fca8d4
e32e67f9111ea979 ce9c4826806aeee6
3de9c0da2bd7f91e bcb2639bf989c625
1b29bf38d39a9bdc e7c55f4b2ac12a39
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Quarter round | O(1) - 8 operations |
| Double round | O(1) - 4 quarter rounds × 2 |
| Per block | O(1) - 10 double rounds |
| Total | **O(n)** where n = message length |

**Operations per 64-byte block**:
- 10 double rounds × 8 quarter rounds = 80 QRs
- 80 × 8 operations = 640 ARX operations
- ~10 operations/byte

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| State | 64 bytes |
| Working copy | 64 bytes |
| Total | **O(1)** constant |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
/// Salsa20 quarter round
#[inline(always)]
fn quarter_round(y: &mut [u32; 16], a: usize, b: usize, c: usize, d: usize) {
    y[b] ^= y[a].wrapping_add(y[d]).rotate_left(7);
    y[c] ^= y[b].wrapping_add(y[a]).rotate_left(9);
    y[d] ^= y[c].wrapping_add(y[b]).rotate_left(13);
    y[a] ^= y[d].wrapping_add(y[c]).rotate_left(18);
}

/// Double round (column + row)
fn double_round(x: &mut [u32; 16]) {
    // Column round
    quarter_round(x, 0, 4, 8, 12);
    quarter_round(x, 5, 9, 13, 1);
    quarter_round(x, 10, 14, 2, 6);
    quarter_round(x, 15, 3, 7, 11);
    
    // Row round
    quarter_round(x, 0, 1, 2, 3);
    quarter_round(x, 5, 6, 7, 4);
    quarter_round(x, 10, 11, 8, 9);
    quarter_round(x, 15, 12, 13, 14);
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty message | Returns empty ciphertext |
| Position overflow | Wraps at 2^64 (security issue) |
| Nonce reuse | BREAKS SECURITY |
| 128-bit key | Expanded with different constants |

### 5.3 Salsa20 vs ChaCha20

| Property | Salsa20 | ChaCha20 |
|----------|---------|----------|
| Rotations | 7, 9, 13, 18 | 16, 12, 8, 7 |
| State layout | Diagonal constants | Row constants |
| Diffusion | Good | Better |
| Security margin | Good | Better |
| Standardization | eSTREAM | IETF (TLS 1.3) |

## 6. Security Analysis

### 6.1 Security Level

- **Key size**: 256 bits
- **Nonce size**: 64 bits
- **Position size**: 64 bits
- **Effective security**: 256 bits

### 6.2 Best Known Attacks

| Variant | Attack | Complexity |
|---------|--------|------------|
| Salsa20/8 | Differential | 2^251 |
| Salsa20/12 | None practical | >2^256 |
| Salsa20/20 | None | Secure |

### 6.3 Critical Requirements

| Requirement | Consequence if Violated |
|-------------|------------------------|
| Unique nonce per key | Plaintext XOR recovery |
| Limit: 2^70 bytes/key | Birthday bound concerns |
| Authenticate | Use NaCl secretbox |

## 7. Real-World Applications

### 7.1 Use Cases

| Domain | Application |
|--------|-------------|
| NaCl library | crypto_stream_salsa20 |
| File encryption | Various tools |
| Key derivation | XSalsa20 + Poly1305 |
| Random generation | Salsa20-based CSPRNG |

### 7.2 Related Constructions

| Name | Description |
|------|-------------|
| XSalsa20 | Extended nonce (192 bits) |
| Salsa20/12 | Reduced rounds |
| HSalsa20 | Hash function variant |
| NaCl secretbox | Salsa20 + Poly1305 |

## 8. XSalsa20 Variant

### 8.1 Motivation

64-bit nonce is too short for random generation:
- Birthday bound: 2^32 messages before collision
- XSalsa20 uses 192-bit nonce: safe for random

### 8.2 Construction

```
HSalsa20: Key derivation using Salsa20 core
    Input: Key (256 bits), Nonce prefix (128 bits)
    Output: Subkey (256 bits)

XSalsa20:
    subkey ← HSalsa20(key, nonce[0:16])
    keystream ← Salsa20(subkey, nonce[16:24], position)
```

## 9. Performance

### 9.1 Benchmarks

| Platform | Cycles/byte |
|----------|-------------|
| x86-64 (SIMD) | ~2-4 |
| ARM (NEON) | ~4-6 |
| 32-bit | ~8-12 |

### 9.2 Optimization Opportunities

1. **SIMD**: 4 blocks in parallel
2. **Unrolling**: Remove loop overhead
3. **Pre-computation**: For fixed key
4. **Caching**: State between blocks

## 10. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| ChaCha20 | Improved variant (preferred) |
| XSalsa20 | Extended nonce version |
| Poly1305 | Companion MAC |
| BLAKE | Related ARX design |

## 11. References

- Bernstein, D.J. (2005). The Salsa20 family of stream ciphers
- eSTREAM Portfolio (2008)
- NaCl: Networking and Cryptography library
- [Implementation](../../src/ciphers/salsa.rs)
