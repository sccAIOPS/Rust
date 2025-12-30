# SHA-3 (Keccak)

## 1. Overview

**SHA-3** is a cryptographic hash function standardized by NIST in 2015, based on the Keccak algorithm. Unlike SHA-2, it uses a completely different internal structure called a **sponge construction**, providing a backup if SHA-2 ever becomes compromised.

### Historical Context
- **2007**: NIST announces SHA-3 competition
- **2012**: Keccak selected as winner (51 submissions)
- **2015**: FIPS 202 standardizes SHA-3
- **Present**: Used alongside SHA-2 for cryptographic diversity

### Variants

| Variant | Output Size | Security Level |
|---------|-------------|----------------|
| SHA3-224 | 224 bits | 112 bits |
| SHA3-256 | 256 bits | 128 bits |
| SHA3-384 | 384 bits | 192 bits |
| SHA3-512 | 512 bits | 256 bits |

## 2. Mathematical Foundation

### 2.1 Sponge Construction

SHA-3 uses a **sponge function** with:
- **State**: 1600 bits (5 × 5 × 64-bit array)
- **Rate (r)**: Bits processed per block
- **Capacity (c)**: Security parameter (1600 - r)

```
State = Rate (r) + Capacity (c) = 1600 bits

SHA3-256: r = 1088, c = 512
SHA3-512: r = 576, c = 1024
```

### 2.2 Two Phases

**Absorbing Phase**:
1. Initialize state to zeros
2. XOR input blocks with rate portion
3. Apply permutation function $f$
4. Repeat until all input absorbed

**Squeezing Phase**:
1. Extract rate bits as output
2. Apply permutation if more bits needed
3. Repeat until desired output length

### 2.3 State Representation

3D array $A[x][y][z]$ where:
- $x, y \in \{0, 1, 2, 3, 4\}$
- $z \in \{0, 1, ..., 63\}$

**Lane**: $A[x][y][*]$ - 64-bit word
**Column**: $A[x][*][z]$ - 5 bits
**Row**: $A[*][y][z]$ - 5 bits

## 3. Algorithm Description

### 3.1 Keccak-f Permutation

The core permutation $f = Keccak\text{-}f[1600]$ consists of 24 rounds, each with 5 steps:

#### θ (Theta) - Column Parity Mix
```
C[x] = A[x,0] ⊕ A[x,1] ⊕ A[x,2] ⊕ A[x,3] ⊕ A[x,4]
D[x] = C[x-1] ⊕ ROT(C[x+1], 1)
A[x,y] = A[x,y] ⊕ D[x]
```

#### ρ (Rho) - Lane Rotation
Rotate each lane by a specific amount (0 to 63 bits).

#### π (Pi) - Lane Permutation
```
A'[y, 2x + 3y] = A[x, y]
```

#### χ (Chi) - Non-linear Step
```
A'[x,y] = A[x,y] ⊕ ((¬A[x+1,y]) ∧ A[x+2,y])
```

#### ι (Iota) - Round Constant Addition
```
A[0,0] = A[0,0] ⊕ RC[round]
```

### 3.2 Pseudocode

```
FUNCTION SHA3-256(message):
    // Parameters for SHA3-256
    r ← 1088                // Rate bits
    c ← 512                 // Capacity bits
    d ← 256                 // Output bits
    
    // Pad message
    P ← message || 0x06 || 0x00...00 || 0x80
    // Padding: 0x06 + zeros + 0x80 to make length multiple of r
    
    // Initialize state
    S ← 1600 zero bits
    
    // Absorbing phase
    FOR each r-bit block B in P:
        S[0..r-1] ← S[0..r-1] ⊕ B
        S ← Keccak-f[1600](S)
    
    // Squeezing phase
    Z ← empty
    WHILE |Z| < d:
        Z ← Z || S[0..r-1]
        IF |Z| < d:
            S ← Keccak-f[1600](S)
    
    RETURN Z[0..d-1]
```

### 3.3 Step-by-Step Example

**Input**: "abc" (24 bits)

1. **Padding**: 
   - Add domain separator (0x06)
   - Pad to rate boundary (1088 bits for SHA3-256)
   - Set final bit (0x80)

2. **Absorb**: XOR padded message into state, apply Keccak-f

3. **Squeeze**: Extract 256 bits from state

**SHA3-256("abc")**:
```
3a985da74fe225b2 045c172d6bd390bd 
855f086e3e9d525b 46bfe24511431532
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Per round | O(1) | Fixed 1600-bit state |
| Per block | O(24) | 24 rounds of Keccak-f |
| Total | **O(n)** | n = message length |

**Detailed per-block operations**:
- θ: 5 × 64 XORs + rotations
- ρ: 25 rotations
- π: 25 moves
- χ: 25 × 5 AND/XOR operations
- ι: 1 XOR

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| State | 1600 bits (200 bytes) |
| Intermediate | O(1) |
| Total | **O(1)** constant |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
/// Keccak state: 5x5 array of 64-bit lanes
type State = [[u64; 5]; 5];

/// Apply one round of Keccak-f
fn keccak_round(state: &mut State, rc: u64) {
    // θ step
    let mut c = [0u64; 5];
    for x in 0..5 {
        c[x] = state[x][0] ^ state[x][1] ^ state[x][2] 
             ^ state[x][3] ^ state[x][4];
    }
    
    let mut d = [0u64; 5];
    for x in 0..5 {
        d[x] = c[(x + 4) % 5] ^ c[(x + 1) % 5].rotate_left(1);
    }
    
    for x in 0..5 {
        for y in 0..5 {
            state[x][y] ^= d[x];
        }
    }
    
    // ρ and π steps combined...
    // χ step...
    // ι step...
}
```

**Implementation highlights**:
- State as `[[u64; 5]; 5]` for efficient access
- Lane-interleaved representation
- Rotation offsets precomputed
- Round constants hardcoded

### 5.2 Padding Details

SHA-3 uses multi-rate padding:
```
pad(M) = M || 0x06 || 0x00* || 0x80
```

Where:
- `0x06` = domain separator for SHA-3
- `0x00*` = zero padding
- `0x80` = final bit ensuring distinct padding

**Note**: SHAKE uses `0x1F` as domain separator.

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty message | Valid input, produces hash |
| Message = rate length | Needs additional padding block |
| Non-byte-aligned | Implementation typically requires byte-alignment |

## 6. Security Analysis

### 6.1 Security Properties

| Property | SHA3-256 | SHA3-512 |
|----------|----------|----------|
| Collision resistance | 128 bits | 256 bits |
| Pre-image resistance | 256 bits | 512 bits |
| Second pre-image | 256 bits | 512 bits |

### 6.2 Advantages over SHA-2

| Property | SHA-3 | SHA-2 |
|----------|-------|-------|
| Length extension attacks | Immune | Vulnerable |
| Design origin | Public competition | NSA |
| Internal structure | Sponge | Merkle-Damgård |
| Cryptographic diversity | Yes | Same family as SHA-1 |

### 6.3 Known Issues

| Issue | Status |
|-------|--------|
| Cryptographic breaks | None known |
| Reduced-round attacks | Academic interest only |
| Side-channel | Implementation-dependent |

## 7. Related Functions

### 7.1 SHA-3 Family

| Function | Description |
|----------|-------------|
| SHA3-224/256/384/512 | Fixed-output hash |
| SHAKE128/256 | Extendable-output (XOF) |
| RawSHAKE | Underlying Keccak |
| cSHAKE | Customizable SHAKE |
| KMAC | Keyed MAC |
| TupleHash | Hash of tuples |
| ParallelHash | Parallel hashing |

### 7.2 Comparison

| Function | Output | Speed | Use Case |
|----------|--------|-------|----------|
| SHA-256 | 256 bits | Medium | General, Bitcoin |
| **SHA3-256** | 256 bits | Medium | Diversity, new systems |
| BLAKE2b | 512 bits | Fast | Performance-critical |
| BLAKE3 | Variable | Fastest | Modern applications |

## 8. Real-World Applications

### 8.1 Use Cases

| Domain | Application |
|--------|-------------|
| Ethereum | Address generation, signatures |
| Digital Signatures | Ed448 uses SHAKE256 |
| Random Number Generation | SHAKE as DRBG |
| Post-quantum crypto | Hash-based signatures |
| File integrity | Modern systems |

### 8.2 Standard Adoption

- FIPS 202 (US Government)
- ISO/IEC 10118-3
- IETF protocols (growing)
- Cryptocurrency (Ethereum)

## 9. References

- FIPS 202: SHA-3 Standard
- The Keccak Reference (Bertoni et al.)
- NIST SP 800-185: SHA-3 Derived Functions
- [Implementation](../../src/ciphers/sha3.rs)
