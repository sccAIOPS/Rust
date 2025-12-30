# TEA (Tiny Encryption Algorithm)

## 1. Overview

**TEA** (Tiny Encryption Algorithm) is a block cipher notable for its simplicity of description and implementation. Designed by David Wheeler and Roger Needham in 1994, it operates on 64-bit blocks using a 128-bit key.

### Historical Context
- **1994**: TEA published (Wheeler & Needham)
- **1997**: Related-key attack discovered
- **1998**: XTEA (eXtended TEA) published as fix
- **2004**: XXTEA published for variable blocks
- **Present**: Educational use; avoid for production

⚠️ **Warning**: TEA has known vulnerabilities. Use AES or ChaCha20 for production systems.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- 64-bit plaintext block (two 32-bit words)
- 128-bit key (four 32-bit words)

**Output**: 64-bit ciphertext block

### 2.2 Feistel-like Structure

TEA uses a modified Feistel network:
- Two 32-bit halves: $L$ (left/v0) and $R$ (right/v1)
- Key words: $K[0], K[1], K[2], K[3]$
- 64 rounds (32 cycles)

### 2.3 Magic Constant

**Delta**: $\delta = \frac{\sqrt{5} - 1}{2} \times 2^{32} = 0x9E3779B9$

This is derived from the golden ratio, chosen because:
- Irrational number (no patterns)
- Full period when summed modulo 2^32
- Provides "nothing up my sleeve" constant

## 3. Algorithm Description

### 3.1 Intuition

TEA is remarkably simple:
1. Split data into two halves
2. Mix halves 64 times using additions, XORs, and shifts
3. Each round adds delta to a sum variable
4. Uses alternating key words

The many rounds compensate for the simple round function.

### 3.2 Encryption Round Function

Each cycle (2 rounds):
```
sum += delta
v0 += ((v1 << 4) + k0) ^ (v1 + sum) ^ ((v1 >> 5) + k1)
v1 += ((v0 << 4) + k2) ^ (v0 + sum) ^ ((v0 >> 5) + k3)
```

### 3.3 Pseudocode

```
CONSTANTS:
    delta = 0x9E3779B9
    rounds = 32  // 32 cycles = 64 rounds

FUNCTION TEA_encrypt(v[2], key[4]):
    v0, v1 ← v[0], v[1]
    sum ← 0
    
    FOR i FROM 1 TO rounds:
        sum ← sum + delta
        v0 ← v0 + (((v1 << 4) + key[0]) ⊕ (v1 + sum) ⊕ ((v1 >> 5) + key[1]))
        v1 ← v1 + (((v0 << 4) + key[2]) ⊕ (v0 + sum) ⊕ ((v0 >> 5) + key[3]))
    
    RETURN [v0, v1]

FUNCTION TEA_decrypt(v[2], key[4]):
    v0, v1 ← v[0], v[1]
    sum ← delta × rounds  // = 0xC6EF3720 for 32 rounds
    
    FOR i FROM 1 TO rounds:
        v1 ← v1 - (((v0 << 4) + key[2]) ⊕ (v0 + sum) ⊕ ((v0 >> 5) + key[3]))
        v0 ← v0 - (((v1 << 4) + key[0]) ⊕ (v1 + sum) ⊕ ((v1 >> 5) + key[1]))
        sum ← sum - delta
    
    RETURN [v0, v1]
```

### 3.4 Step-by-Step Example

**Plaintext**: `[0x01234567, 0x89ABCDEF]`
**Key**: `[0x00112233, 0x44556677, 0x8899AABB, 0xCCDDEEFF]`

| Cycle | Sum | v0 | v1 |
|-------|-----|-----|-----|
| 0 | 0x00000000 | 0x01234567 | 0x89ABCDEF |
| 1 | 0x9E3779B9 | 0xDE6D4AA9 | 0x3E7E9C7F |
| 2 | 0x3C6EF372 | ... | ... |
| ... | ... | ... | ... |
| 32 | 0xC6EF3720 | Final v0 | Final v1 |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Per cycle | O(1) - 8 operations |
| Per block | O(32) = O(1) |
| Total | **O(n)** where n = message blocks |

**Operations per block**:
- 32 cycles × 8 operations = 256 operations
- ~4 operations/byte

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| State (v0, v1) | 8 bytes |
| Key | 16 bytes |
| Sum | 4 bytes |
| Total | **O(1)** constant |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
const DELTA: u32 = 0x9E3779B9;
const ROUNDS: u32 = 32;

pub fn encrypt(plaintext: &[u32; 2], key: &[u32; 4]) -> [u32; 2] {
    let mut v0 = plaintext[0];
    let mut v1 = plaintext[1];
    let mut sum: u32 = 0;

    for _ in 0..ROUNDS {
        sum = sum.wrapping_add(DELTA);
        v0 = v0.wrapping_add(
            ((v1 << 4).wrapping_add(key[0]))
            ^ (v1.wrapping_add(sum))
            ^ ((v1 >> 5).wrapping_add(key[1]))
        );
        v1 = v1.wrapping_add(
            ((v0 << 4).wrapping_add(key[2]))
            ^ (v0.wrapping_add(sum))
            ^ ((v0 >> 5).wrapping_add(key[3]))
        );
    }

    [v0, v1]
}
```

**Key points**:
- Use `wrapping_add` and `wrapping_sub` for modular arithmetic
- No external dependencies needed
- Entire algorithm in ~20 lines

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| All-zero key | Valid but weak |
| All-zero plaintext | Valid |
| Key with equivalent classes | Vulnerability |
| Non-64-bit blocks | Requires padding/mode |

## 6. Security Analysis

### 6.1 Known Vulnerabilities

| Vulnerability | Impact | Year |
|---------------|--------|------|
| Equivalent keys | 2^126 related keys → same ciphertext | 1997 |
| Related-key attack | Key recovery with 2^23 chosen plaintexts | 1997 |
| Weak key schedule | No key expansion | Design |

### 6.2 Attack Details

**Related-Key Attack**:
- TEA has equivalent keys: different keys producing same ciphertext
- Each key has 3 equivalent keys (2^126 pairs)
- This is a design flaw, not implementation issue

### 6.3 Comparison

| Cipher | Block | Key | Rounds | Status |
|--------|-------|-----|--------|--------|
| TEA | 64 | 128 | 64 | Broken |
| XTEA | 64 | 128 | 64 | Improved |
| XXTEA | Variable | 128 | Variable | Better |
| AES-128 | 128 | 128 | 10 | Secure |

## 7. Variants

### 7.1 XTEA (Block TEA)

**Improvements**:
- Different key schedule
- Key words selected based on sum bits
- Resists related-key attacks

```
v0 += ((v1 << 4) ^ (v1 >> 5)) + v1) ^ (sum + key[sum & 3])
sum += delta
v1 += ((v0 << 4) ^ (v0 >> 5)) + v0) ^ (sum + key[(sum >> 11) & 3])
```

### 7.2 XXTEA (Corrected Block TEA)

**Improvements**:
- Variable-length blocks
- More complex mixing
- Better security margins

### 7.3 Comparison

| Property | TEA | XTEA | XXTEA |
|----------|-----|------|-------|
| Block size | 64 bits | 64 bits | Variable |
| Key schedule | None | Improved | More complex |
| Equivalent keys | Yes | No | No |
| Recommended | No | Legacy | Legacy |

## 8. Real-World Applications

### 8.1 Historical Usage

| Domain | Notes |
|--------|-------|
| Xbox | Used in game saves |
| Some IoT devices | Small code size |
| Educational | Teaching block ciphers |

### 8.2 When TEA Might Be Appropriate

| Scenario | Recommendation |
|----------|----------------|
| Learning | Yes - excellent for understanding ciphers |
| Embedded (severe constraints) | Consider XTEA |
| Any security requirement | Use AES or ChaCha20 |
| New projects | Never |

## 9. Educational Value

### 9.1 Why Study TEA?

1. **Simplicity**: Entire algorithm fits on one page
2. **Illustrates concepts**:
   - Feistel-like structure
   - Key scheduling importance
   - Security analysis
3. **Demonstrates vulnerabilities**: Real-world crypto failures
4. **Easy implementation**: No tables, minimal code

### 9.2 Lessons Learned

| Lesson | TEA Example |
|--------|-------------|
| Simple ≠ Weak | Many rounds compensate |
| Simple = Analyzable | Vulnerabilities found quickly |
| Key schedule matters | No schedule → related keys |
| Standards matter | Competition-vetted ciphers are stronger |

## 10. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| XTEA | Direct successor |
| XXTEA | Block variant |
| RC5 | Similar simplicity goals |
| AES | Modern replacement |
| Simon/Speck | NSA lightweight ciphers |

## 11. References

- Wheeler, D.J. & Needham, R.M. (1994). TEA, a Tiny Encryption Algorithm
- Kelsey, J., Schneier, B., & Wagner, D. (1997). Related-key cryptanalysis of 3-WAY, Biham-DES, CAST, DES-X, NewDES, RC2, and TEA
- Wheeler, D.J. & Needham, R.M. (1998). XTEA
- [Implementation](../../src/ciphers/tea.rs)
