# Caesar Cipher

## 1. Overview

The **Caesar Cipher** is one of the oldest and simplest encryption techniques, named after Julius Caesar who reportedly used it to communicate with his generals. It is a type of substitution cipher where each letter is replaced by another letter a fixed number of positions down the alphabet.

### Historical Context
- **~100 BCE**: Used by Julius Caesar (shift of 3)
- **Renaissance**: Simple cryptanalysis techniques developed
- **Present**: Educational tool, basis for understanding substitution ciphers

⚠️ **Warning**: The Caesar cipher provides NO real security. Use only for educational purposes or simple obfuscation.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- Plaintext message $P$
- Shift value $k$ (0 to 25)

**Output**: Ciphertext $C$

### 2.2 Mathematical Model

For alphabet position $p$ (A=0, B=1, ..., Z=25):

**Encryption**: $E_k(p) = (p + k) \mod 26$

**Decryption**: $D_k(c) = (c - k) \mod 26$

### 2.3 Group Theory Perspective

The Caesar cipher is an element of $\mathbb{Z}_{26}$:
- Encryption: Addition in the cyclic group
- Key space: 26 possible shifts
- Identity: Shift of 0

## 3. Algorithm Description

### 3.1 Intuition

Imagine the alphabet as a circular wheel:
```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓  (shift=3)
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
```

Each letter "slides" forward by the key amount, wrapping around at Z.

### 3.2 Pseudocode

```
FUNCTION caesar_encrypt(plaintext, shift):
    ciphertext ← ""
    FOR each character c in plaintext:
        IF c is a letter:
            base ← 'A' if c is uppercase else 'a'
            shifted ← ((c - base + shift) MOD 26) + base
            ciphertext ← ciphertext + shifted
        ELSE:
            ciphertext ← ciphertext + c  // Keep non-letters
    RETURN ciphertext

FUNCTION caesar_decrypt(ciphertext, shift):
    RETURN caesar_encrypt(ciphertext, 26 - shift)
```

### 3.3 Step-by-Step Example

**Plaintext**: "HELLO"
**Shift**: 3

| Letter | Position | + Shift | mod 26 | Result |
|--------|----------|---------|--------|--------|
| H | 7 | 10 | 10 | K |
| E | 4 | 7 | 7 | H |
| L | 11 | 14 | 14 | O |
| L | 11 | 14 | 14 | O |
| O | 14 | 17 | 17 | R |

**Ciphertext**: "KHOOR"

### 3.4 Full Alphabet Reference (Shift=3)

```
Plain:  ABCDEFGHIJKLMNOPQRSTUVWXYZ
Cipher: DEFGHIJKLMNOPQRSTUVWXYZABC
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Single character | O(1) |
| Encryption/Decryption | **O(n)** |

Where n = message length

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Key | O(1) - single number |
| Output | O(n) - same as input |
| Total | **O(n)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn caesar_encrypt(text: &str, shift: u8) -> String {
    let shift = shift % 26;  // Normalize shift
    
    text.chars()
        .map(|c| {
            if c.is_ascii_alphabetic() {
                let base = if c.is_ascii_uppercase() { b'A' } else { b'a' };
                let shifted = (c as u8 - base + shift) % 26 + base;
                shifted as char
            } else {
                c  // Non-alphabetic characters unchanged
            }
        })
        .collect()
}

pub fn caesar_decrypt(text: &str, shift: u8) -> String {
    caesar_encrypt(text, 26 - (shift % 26))
}
```

**Key implementation details**:
- Handle both uppercase and lowercase
- Preserve non-alphabetic characters
- Normalize shift to 0-25 range
- Decryption is encryption with complementary shift

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Shift = 0 | No change |
| Shift = 26 | No change (equivalent to 0) |
| Shift > 26 | Modulo 26 |
| Non-letters | Pass through unchanged |
| Empty string | Returns empty string |
| Unicode | ASCII letters only typically |

## 6. Security Analysis

### 6.1 Key Space Analysis

- **Total keys**: 26 (actually 25 meaningful, since 0 is identity)
- **Bits of security**: ~4.7 bits (log₂(26) ≈ 4.7)

### 6.2 Attack Methods

| Attack | Complexity | Description |
|--------|------------|-------------|
| Brute force | O(26) | Try all shifts |
| Frequency analysis | O(n) | Match letter frequencies |
| Known plaintext | O(1) | Single letter reveals key |

### 6.3 Cryptanalysis Example

**Ciphertext**: "WKLV LV D WHVW"

**Frequency analysis**:
- W appears 3 times
- Most common English letter is 'E'
- If W→E, shift = 22 (W-E=22, or equivalently encrypt shift=4)

**Brute force** (showing meaningful results):
```
Shift 0:  WKLV LV D WHVW
Shift 1:  VJKU KU C VGUV
Shift 2:  UIJT JT B UFTU
Shift 3:  THIS IS A TEST  ← Clear text!
```

## 7. Variants

### 7.1 ROT13

Special case where shift = 13:
- Self-inverse: ROT13(ROT13(x)) = x
- Used for spoiler hiding, not security
- `ROT13("HELLO") = "URYYB"`

### 7.2 ROT47

- Uses ASCII 33-126 (94 characters)
- Shift of 47 (self-inverse)
- Handles numbers and symbols

### 7.3 Generalized Caesar

- Arbitrary alphabet (not just A-Z)
- Different modulus for different character sets
- Example: Base64 Caesar with 64-character alphabet

## 8. Educational Value

### 8.1 Concepts Demonstrated

| Concept | Caesar Example |
|---------|----------------|
| Symmetric encryption | Same key encrypts/decrypts |
| Modular arithmetic | Wrapping at alphabet end |
| Key space | Brute force feasibility |
| Frequency analysis | Pattern preservation |
| Known-plaintext attack | Single letter reveals all |

### 8.2 Why Caesar Fails

1. **Too few keys**: 26 is trivially searchable
2. **Frequency preservation**: Letter frequencies unchanged
3. **Patterns preserved**: Same letters → same ciphertext
4. **No diffusion**: One letter doesn't affect others

## 9. Real-World Applications

### 9.1 Legitimate Uses

| Use | Purpose |
|-----|---------|
| Education | Teaching cryptography basics |
| Puzzles/Games | Simple encryption challenges |
| ROT13 | Spoiler/NSFW hiding on forums |
| Obfuscation | Very weak hiding (not security) |

### 9.2 Historical Significance

- First documented cipher in Western history
- Introduced concept of systematic encryption
- Foundation for understanding substitution ciphers

## 10. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| ROT13 | Caesar with shift=13 |
| Vigenère | Multiple Caesar ciphers |
| Affine cipher | Caesar + multiplication |
| Substitution cipher | Generalization (arbitrary mapping) |

## 11. References

- Singh, S. (1999). The Code Book
- Kahn, D. (1996). The Codebreakers
- Suetonius. The Twelve Caesars (historical reference)
- [Implementation](../../src/ciphers/caesar.rs)
