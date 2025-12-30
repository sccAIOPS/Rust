# Vigenère Cipher

## 1. Overview

The **Vigenère cipher** is a polyalphabetic substitution cipher that uses a keyword to shift letters by varying amounts. First described by Giovan Battista Bellaso in 1553 and later misattributed to Blaise de Vigenère, it was considered "le chiffre indéchiffrable" (the indecipherable cipher) for three centuries.

### Historical Context
- **1553**: Bellaso publishes the cipher
- **1586**: Vigenère publishes related work (gets credit)
- **1854**: Babbage/Kasiski break the cipher
- **Present**: Educational tool for understanding polyalphabetic ciphers

⚠️ **Warning**: The Vigenère cipher is broken and provides no real security.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- Plaintext $P = p_0, p_1, ..., p_{n-1}$
- Keyword $K = k_0, k_1, ..., k_{m-1}$

**Output**: Ciphertext $C = c_0, c_1, ..., c_{n-1}$

### 2.2 Mathematical Model

**Encryption**: $c_i = (p_i + k_{i \mod m}) \mod 26$

**Decryption**: $p_i = (c_i - k_{i \mod m}) \mod 26$

### 2.3 Tabula Recta

The classical tool for Vigenère encryption:

```
    A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
A   A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
B   B C D E F G H I J K L M N O P Q R S T U V W X Y Z A
C   C D E F G H I J K L M N O P Q R S T U V W X Y Z A B
D   D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
...
Z   Z A B C D E F G H I J K L M N O P Q R S T U V W X Y
```

## 3. Algorithm Description

### 3.1 Intuition

The Vigenère cipher applies a different Caesar shift to each letter:
- First letter: shifted by first key letter
- Second letter: shifted by second key letter
- When key exhausted: wrap around to start

This disguises letter frequencies that doom simple substitution.

### 3.2 Pseudocode

```
FUNCTION vigenere_encrypt(plaintext, key):
    ciphertext ← ""
    key_index ← 0
    
    FOR each character c in plaintext:
        IF c is a letter:
            shift ← key[key_index MOD len(key)] - 'A'
            IF c is uppercase:
                encrypted ← ((c - 'A' + shift) MOD 26) + 'A'
            ELSE:
                encrypted ← ((c - 'a' + shift) MOD 26) + 'a'
            ciphertext ← ciphertext + encrypted
            key_index ← key_index + 1
        ELSE:
            ciphertext ← ciphertext + c  // Non-letters unchanged
    
    RETURN ciphertext
```

### 3.3 Step-by-Step Example

**Plaintext**: "ATTACKATDAWN"
**Key**: "LEMON"

| Plain | A | T | T | A | C | K | A | T | D | A | W | N |
|-------|---|---|---|---|---|---|---|---|---|---|---|---|
| Key | L | E | M | O | N | L | E | M | O | N | L | E |
| Shift | 11| 4 | 12| 14| 13| 11| 4 | 12| 14| 13| 11| 4 |
| Cipher| L | X | F | O | P | V | E | F | R | N | H | R |

**Ciphertext**: "LXFOPVEFRNHR"

### 3.4 Decryption Example

**Ciphertext**: "LXFOPVEFRNHR"
**Key**: "LEMON"

| Cipher | L | X | F | O | P | V | E | F | R | N | H | R |
|--------|---|---|---|---|---|---|---|---|---|---|---|---|
| Key | L | E | M | O | N | L | E | M | O | N | L | E |
| Shift | -11| -4 | -12| -14| -13| -11| -4 | -12| -14| -13| -11| -4 |
| Plain | A | T | T | A | C | K | A | T | D | A | W | N |

**Plaintext**: "ATTACKATDAWN"

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Encryption | **O(n)** |
| Decryption | **O(n)** |

Where n = plaintext length

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Key | O(m) |
| Output | O(n) |
| Total | **O(n + m)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn vigenere_encrypt(text: &str, key: &str) -> String {
    let key: Vec<u8> = key.to_uppercase()
        .chars()
        .filter(|c| c.is_ascii_alphabetic())
        .map(|c| c as u8 - b'A')
        .collect();
    
    if key.is_empty() {
        return text.to_string();
    }
    
    let mut key_index = 0;
    text.chars()
        .map(|c| {
            if c.is_ascii_alphabetic() {
                let base = if c.is_ascii_uppercase() { b'A' } else { b'a' };
                let shift = key[key_index % key.len()];
                key_index += 1;
                ((c as u8 - base + shift) % 26 + base) as char
            } else {
                c
            }
        })
        .collect()
}

pub fn vigenere_decrypt(text: &str, key: &str) -> String {
    let key: Vec<u8> = key.to_uppercase()
        .chars()
        .filter(|c| c.is_ascii_alphabetic())
        .map(|c| 26 - (c as u8 - b'A'))  // Inverse shift
        .collect();
    
    // Use encrypt with inverse key
    vigenere_encrypt_with_shifts(text, &key)
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty key | Return plaintext unchanged |
| Key with numbers | Filter out (use letters only) |
| Repeating key letters | Valid (reduces security) |
| Key longer than message | Only use needed portion |
| Non-letters in text | Skip, don't advance key index |

## 6. Security Analysis

### 6.1 Key Space

| Key Length | Possible Keys |
|------------|---------------|
| 1 | 26 (= Caesar cipher) |
| 2 | 676 |
| 3 | 17,576 |
| m | 26^m |

### 6.2 Breaking the Cipher

#### Kasiski Examination (1863)

1. Find repeated sequences in ciphertext
2. Distances between repeats suggest key length multiples
3. GCD of distances likely reveals key length

**Example**:
```
Ciphertext: VVHQWVVHQWVVHQW...
            ↑     ↑     ↑
Position:   0     6     12
Distance: 6, 6 → Key length likely 6 or factor of 6
```

#### Index of Coincidence

Expected IC for English: ~0.0667
For random text: ~0.0385

By grouping ciphertext at positions 0, m, 2m, ... and calculating IC:
- Correct key length → IC ≈ 0.0667
- Wrong key length → IC ≈ 0.0385

#### Frequency Analysis per Position

Once key length m is known:
1. Split ciphertext into m groups
2. Each group is simple Caesar cipher
3. Frequency analysis breaks each group

### 6.3 Known Attacks Summary

| Attack | Requirement | Effectiveness |
|--------|-------------|---------------|
| Kasiski | Sufficient ciphertext | Finds key length |
| IC Analysis | ~25 chars/key position | Finds key length |
| Frequency | Key length known | Breaks completely |
| Known plaintext | Any known text | Instant break |

## 7. Variants

### 7.1 Autokey Cipher

Uses plaintext as part of key:
```
Key = keyword + plaintext
```
Harder to break but still vulnerable.

### 7.2 Running Key Cipher

Uses long text (e.g., book) as key:
- Key as long as message
- Still breakable with enough ciphertext

### 7.3 One-Time Pad

True random key, same length as message:
- **Theoretically unbreakable** if:
  - Key is truly random
  - Key used only once
  - Key as long as message
  - Key kept secret

## 8. Educational Value

### 8.1 Concepts Demonstrated

| Concept | How Vigenère Shows It |
|---------|----------------------|
| Polyalphabetic substitution | Multiple alphabets |
| Key length sensitivity | Longer = harder |
| Statistical attacks | Kasiski, IC |
| Evolution of cryptanalysis | 300 years to break |

### 8.2 Common Student Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Advancing key on non-letters | Only advance on letters |
| Case sensitivity in key | Normalize key to uppercase |
| Modular arithmetic errors | Use `%` carefully with negatives |

## 9. Real-World Applications

### 9.1 Historical

| Era | Use |
|-----|-----|
| 16th-19th century | Diplomatic communications |
| American Civil War | Confederate cipher |
| Pre-WW1 | Various military uses |

### 9.2 Modern Uses

- **Educational**: Teaching cryptography
- **Puzzles**: Escape rooms, CTF competitions
- **Cultural**: Crosswords, games

## 10. Comparison with Modern Ciphers

| Property | Vigenère | AES |
|----------|----------|-----|
| Key size | Variable | 128-256 bits |
| Security | Broken | Secure |
| Speed | Very fast | Fast |
| Purpose | Historical | Production |

## 11. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Caesar | Single-key version |
| Autokey | Self-keying variant |
| One-Time Pad | Perfected version |
| Enigma | Mechanical polyalphabetic |

## 12. References

- Kasiski, F.W. (1863). Die Geheimschriften und die Dechiffrir-kunst
- Singh, S. (1999). The Code Book
- Kahn, D. (1996). The Codebreakers
- [Implementation](../../src/ciphers/vigenere.rs)
