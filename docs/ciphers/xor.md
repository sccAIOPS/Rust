# XOR Cipher

## 1. Overview

The **XOR Cipher** uses the exclusive-or (XOR) operation to combine plaintext with a key. It is the simplest form of stream cipher and forms the basis for many modern encryption algorithms. When used with a truly random key of equal length to the message (never reused), it becomes the theoretically unbreakable **One-Time Pad**.

### Historical Context
- **1917**: Vernam invents the one-time pad
- **1949**: Shannon proves OTP security
- **Present**: XOR is fundamental operation in AES, ChaCha, etc.

### Security Warning

| Configuration | Security |
|---------------|----------|
| Repeating key | **Broken** - trivially attackable |
| Random key, reused | **Broken** - XOR of plaintexts |
| One-time pad | **Unbreakable** - theoretically perfect |

## 2. Mathematical Foundation

### 2.1 XOR Operation

**Truth Table**:
| A | B | A ⊕ B |
|---|---|-------|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

### 2.2 Key Properties

**Self-inverse**: $A \oplus A = 0$
**Identity**: $A \oplus 0 = A$
**Commutative**: $A \oplus B = B \oplus A$
**Associative**: $(A \oplus B) \oplus C = A \oplus (B \oplus C)$

### 2.3 Encryption/Decryption

$$C = P \oplus K$$
$$P = C \oplus K$$

Both operations are identical! XOR is its own inverse.

### 2.4 Why XOR Works

From properties:
$$C \oplus K = (P \oplus K) \oplus K = P \oplus (K \oplus K) = P \oplus 0 = P$$

## 3. Algorithm Description

### 3.1 Simple XOR Cipher

**With single-byte key**:
```
FUNCTION xor_cipher(data, key):
    result ← []
    FOR i FROM 0 TO len(data) - 1:
        result[i] ← data[i] XOR key
    RETURN result
```

**With multi-byte key (repeating)**:
```
FUNCTION xor_cipher_repeating(data, key):
    result ← []
    FOR i FROM 0 TO len(data) - 1:
        result[i] ← data[i] XOR key[i MOD len(key)]
    RETURN result
```

### 3.2 Step-by-Step Example

**Plaintext**: "HI" = [0x48, 0x49]
**Key**: 0x2A

| Byte | Hex | Binary | Key Binary | XOR Result | Result Hex |
|------|-----|--------|------------|------------|------------|
| H | 0x48 | 01001000 | 00101010 | 01100010 | 0x62 |
| I | 0x49 | 01001001 | 00101010 | 01100011 | 0x63 |

**Ciphertext**: [0x62, 0x63] = "bc"

**Decrypt** (same operation):
| Byte | Hex | Binary | Key Binary | XOR Result | Result Hex |
|------|-----|--------|------------|------------|------------|
| b | 0x62 | 01100010 | 00101010 | 01001000 | 0x48 |
| c | 0x63 | 01100011 | 00101010 | 01001001 | 0x49 |

**Recovered**: [0x48, 0x49] = "HI" ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| XOR per byte | O(1) |
| Full message | **O(n)** |

Where n = message length in bytes.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Key | O(k) or O(1) for single byte |
| Output | O(n) |
| **Total** | **O(n)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Single-byte key**:
```rust
pub fn xor_cipher(data: &[u8], key: u8) -> Vec<u8> {
    data.iter().map(|&byte| byte ^ key).collect()
}
```

**Multi-byte key**:
```rust
pub fn xor_cipher_repeating(data: &[u8], key: &[u8]) -> Vec<u8> {
    if key.is_empty() {
        return data.to_vec();
    }
    data.iter()
        .enumerate()
        .map(|(i, &byte)| byte ^ key[i % key.len()])
        .collect()
}
```

**String version**:
```rust
pub fn xor_string(text: &str, key: u8) -> String {
    text.bytes()
        .map(|b| (b ^ key) as char)
        .collect()
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty data | Return empty |
| Empty key | Return original (or error) |
| Key = 0 | No change (identity) |
| Key = 0xFF | Bit flip all |

## 6. Security Analysis

### 6.1 Attacks on Single-Byte XOR

**Frequency Analysis**:
1. XOR preserves character relationships
2. Most common byte in ciphertext likely = most common plaintext byte ⊕ key
3. In English: most common is space (0x20) or 'e' (0x65)

**Brute Force**:
- Only 256 possible keys
- Try all, look for readable text

### 6.2 Attacks on Repeating Key XOR

**Kasiski-like Analysis**:
1. Find key length using index of coincidence
2. Split ciphertext into key-length groups
3. Each group is single-byte XOR - solve independently

**Known Plaintext**:
If any plaintext known: $K = P \oplus C$

### 6.3 The Key Reuse Problem

If same key encrypts two messages:
$$C_1 = P_1 \oplus K$$
$$C_2 = P_2 \oplus K$$
$$C_1 \oplus C_2 = P_1 \oplus P_2$$

The key cancels out! Attacker gets XOR of plaintexts.

### 6.4 One-Time Pad Security

**Requirements**:
1. Key truly random (not pseudo-random)
2. Key as long as message
3. Key used only once
4. Key kept secret

**Shannon's Theorem**: OTP provides perfect secrecy - ciphertext reveals nothing about plaintext.

## 7. Practical Applications

### 7.1 Where XOR Is Used

| Application | How XOR Is Used |
|-------------|-----------------|
| AES | XOR in AddRoundKey |
| ChaCha20 | Plaintext XOR keystream |
| RC4 | XOR-based stream cipher |
| Hash functions | XOR in compression |
| Error detection | Parity bits |
| RAID | XOR for redundancy |

### 7.2 When Simple XOR Is Acceptable

| Use Case | Acceptability |
|----------|---------------|
| Obfuscation (non-security) | OK |
| CTF/Puzzles | OK |
| Learning | OK |
| Any real security | **NEVER** |

## 8. Breaking XOR Cipher

### 8.1 Single-Byte Key Cracking

```rust
pub fn crack_single_byte_xor(ciphertext: &[u8]) -> (u8, String, f64) {
    let mut best_key = 0u8;
    let mut best_score = f64::MIN;
    let mut best_text = String::new();
    
    for key in 0..=255u8 {
        let decrypted: String = ciphertext
            .iter()
            .map(|&b| (b ^ key) as char)
            .collect();
        
        let score = english_score(&decrypted);
        
        if score > best_score {
            best_score = score;
            best_key = key;
            best_text = decrypted;
        }
    }
    
    (best_key, best_text, best_score)
}

fn english_score(text: &str) -> f64 {
    // Score based on character frequency
    let english_freq: HashMap<char, f64> = /* letter frequencies */;
    text.chars()
        .filter_map(|c| english_freq.get(&c.to_ascii_lowercase()))
        .sum()
}
```

### 8.2 Repeating Key Detection

```rust
pub fn find_key_length(ciphertext: &[u8], max_len: usize) -> usize {
    (2..=max_len)
        .min_by_key(|&len| {
            // Calculate average Hamming distance between blocks
            let blocks: Vec<_> = ciphertext.chunks(len).collect();
            let distances: Vec<_> = blocks.windows(2)
                .map(|w| hamming_distance(w[0], w[1]))
                .collect();
            let avg = distances.iter().sum::<u32>() / distances.len() as u32;
            avg * 1000 / len as u32  // Normalize by key length
        })
        .unwrap_or(2)
}
```

## 9. XOR Properties in Cryptography

### 9.1 Diffusion Through XOR

XOR provides:
- Bit-level mixing
- Reversibility
- No information loss

### 9.2 Linear Operation

XOR is linear over GF(2):
$$f(a \oplus b) = f(a) \oplus f(b)$$

This is why modern ciphers combine XOR with non-linear operations (S-boxes).

## 10. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| One-Time Pad | XOR with perfect key |
| Stream ciphers | XOR with generated keystream |
| Block ciphers | XOR in various components |
| LFSR | XOR-based key generation |

## 11. Educational Value

### 11.1 Concepts Demonstrated

| Concept | XOR Example |
|---------|-------------|
| Symmetric encryption | Same operation encrypts/decrypts |
| Key importance | Weak key = weak cipher |
| Mathematical security | OTP proof |
| Stream cipher principle | XOR with keystream |

### 11.2 Common Mistakes

| Mistake | Problem |
|---------|---------|
| Reusing key | Reveals XOR of plaintexts |
| Short key | Easy to brute force |
| Predictable key | No security |
| Thinking XOR = secure | It's not (alone) |

## 12. References

- Shannon, C. (1949). Communication Theory of Secrecy Systems
- Schneier, B. (1996). Applied Cryptography
- NIST: Block Cipher Modes of Operation
- [Implementation](../../src/ciphers/xor.rs)
