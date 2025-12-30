# ROT13

## 1. Overview

**ROT13** ("rotate by 13 places") is a special case of the Caesar cipher where the shift is exactly 13. Its key property is that it is its own inverse—applying ROT13 twice returns the original text.

### Historical Context
- **Origin**: Usenet newsgroups (1980s)
- **Purpose**: Hide spoilers, punchlines, offensive content
- **Present**: Cultural artifact, simple obfuscation

⚠️ **Warning**: ROT13 is NOT encryption. It provides zero security and is trivially reversible.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: Text string
**Output**: ROT13-transformed text

### 2.2 Mathematical Model

For letter position $p$ (A=0, B=1, ..., Z=25):

$$ROT13(p) = (p + 13) \mod 26$$

### 2.3 Self-Inverse Property

Since 13 + 13 = 26 ≡ 0 (mod 26):

$$ROT13(ROT13(p)) = (p + 13 + 13) \mod 26 = p$$

This means encryption and decryption are **identical operations**.

### 2.4 Alphabet Mapping

```
Input:  A B C D E F G H I J K L M | N O P Q R S T U V W X Y Z
Output: N O P Q R S T U V W X Y Z | A B C D E F G H I J K L M
```

The alphabet splits exactly in half.

## 3. Algorithm Description

### 3.1 Intuition

ROT13 swaps:
- A ↔ N
- B ↔ O
- C ↔ P
- ... and so on

It's like splitting the alphabet deck in half and swapping the halves.

### 3.2 Pseudocode

```
FUNCTION rot13(text):
    result ← ""
    FOR each character c in text:
        IF c is between 'A' and 'Z':
            result ← result + ((c - 'A' + 13) MOD 26) + 'A'
        ELSE IF c is between 'a' and 'z':
            result ← result + ((c - 'a' + 13) MOD 26) + 'a'
        ELSE:
            result ← result + c
    RETURN result
```

### 3.3 Example

**Input**: "Hello, World!"

| Char | Type | Operation | Result |
|------|------|-----------|--------|
| H | Upper | (7+13) mod 26 = 20 | U |
| e | Lower | (4+13) mod 26 = 17 | r |
| l | Lower | (11+13) mod 26 = 24 | y |
| l | Lower | (11+13) mod 26 = 24 | y |
| o | Lower | (14+13) mod 26 = 1 | b |
| , | Other | Pass through | , |
| (space) | Other | Pass through | (space) |
| W | Upper | (22+13) mod 26 = 9 | J |
| o | Lower | (14+13) mod 26 = 1 | b |
| r | Lower | (17+13) mod 26 = 4 | e |
| l | Lower | (11+13) mod 26 = 24 | y |
| d | Lower | (3+13) mod 26 = 16 | q |
| ! | Other | Pass through | ! |

**Output**: "Uryyb, Jbeyq!"

**Verify** (apply ROT13 again): "Hello, World!" ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Single character | O(1) |
| Full text | **O(n)** |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| No key needed | O(0) |
| Output string | O(n) |
| Total | **O(n)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn rot13(text: &str) -> String {
    text.chars()
        .map(|c| match c {
            'A'..='M' | 'a'..='m' => ((c as u8) + 13) as char,
            'N'..='Z' | 'n'..='z' => ((c as u8) - 13) as char,
            _ => c,
        })
        .collect()
}
```

**Alternative implementation** (branch-free):
```rust
pub fn rot13_branchless(text: &str) -> String {
    text.chars()
        .map(|c| {
            if c.is_ascii_alphabetic() {
                let base = if c.is_ascii_uppercase() { b'A' } else { b'a' };
                (((c as u8 - base) + 13) % 26 + base) as char
            } else {
                c
            }
        })
        .collect()
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty string | Returns empty |
| Only spaces | Returns only spaces |
| Numbers | Pass through unchanged |
| Unicode | ASCII letters only |
| Mixed case | Preserves case |

## 6. Security Analysis

### 6.1 Security Level

**None.** ROT13 provides:
- 0 bits of security
- Instantly reversible
- No key management needed (or possible)

### 6.2 Why ROT13 Is Not Encryption

| Property | ROT13 | Real Encryption |
|----------|-------|-----------------|
| Key | None | Required |
| Strength | 0 bits | 128+ bits |
| Reversibility | Trivial | Requires key |
| Purpose | Obfuscation | Security |

### 6.3 Common Misconception

> "I encrypted the password with ROT13"

**This is wrong.** ROT13 provides no protection against:
- Automated scanning
- Any attacker
- Literally anyone who knows what ROT13 is

## 7. Cultural Significance

### 7.1 Usenet Culture

ROT13 became standard for:
- **Spoilers**: "The killer is Xrire!" (decode to see)
- **Jokes**: Hide punchlines
- **NSFW content**: Prevent accidental viewing
- **Self-referential humor**: "How do you decrypt ROT13? ROT13!"

### 7.2 Famous Examples

**Classic Usenet joke**:
```
Q: How many ROT13 strands does it take to screw in a lightbulb?
A: Gjb. Bar gb ubyq gur ohyo, bar gb ebgngr gur fgeaq.
```
(Decoded: "Two. One to hold the bulb, one to rotate the strand.")

### 7.3 ROT13 Word Coincidences

Some English words become other words:
- `HELLO` ↔ `URYYB`
- `ABJURER` ↔ `NOWHERE`
- `CHECHEN` ↔ `PURPURA`
- `TANG` ↔ `GNAT`

## 8. Variants

### 8.1 ROT5 (Digits)

Rotate digits by 5:
```
0 1 2 3 4 | 5 6 7 8 9
5 6 7 8 9 | 0 1 2 3 4
```

### 8.2 ROT18 (ROT13 + ROT5)

Combines letter and digit rotation:
- Letters: ROT13
- Digits: ROT5

### 8.3 ROT47

Uses ASCII characters 33-126 (94 printable chars):
- Shift of 47 (self-inverse since 94/2 = 47)
- Handles more character types

## 9. Implementation Variations

### 9.1 Lookup Table

```rust
const ROT13_TABLE: [u8; 256] = /* precomputed */;

pub fn rot13_table(text: &str) -> String {
    text.bytes()
        .map(|b| ROT13_TABLE[b as usize] as char)
        .collect()
}
```

### 9.2 SIMD (Theoretical)

```
// Vectorized ROT13 for 16 bytes at once
mask_alpha = identify alphabetic bytes
offset = select(mask_alpha, 13, 0)
result = (input + offset) mod 26 (with case handling)
```

## 10. Real-World Applications

### 10.1 Legitimate Uses

| Use Case | Example |
|----------|---------|
| Spoiler tags | Reddit, forums |
| Puzzle games | Geocaching |
| Email subjects | "[ROT13] Movie ending..." |
| Test data | Simple transformation |

### 10.2 What NOT to Use It For

| Bad Use | Better Alternative |
|---------|-------------------|
| Password storage | bcrypt, Argon2 |
| Secure communication | TLS, encryption |
| Data protection | AES, ChaCha20 |
| Access control | Proper authentication |

## 11. Fun Facts

1. **Double ROT13** is jokingly called "military-grade encryption"
2. **ROT26** is the "unbreakable" variant (it's the identity function)
3. Some email clients have ROT13 built-in
4. `tr 'A-Za-z' 'N-ZA-Mn-za-m'` - Unix one-liner for ROT13

## 12. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Caesar cipher | ROT13 is Caesar with k=13 |
| ROT47 | Extended to ASCII printable |
| Atbash | Z→A, Y→B, etc. |
| Vigenère | Uses multiple shifts |

## 13. References

- RFC 4648 Appendix (mentions ROT13)
- Usenet FAQ archives
- Wikipedia: ROT13
- [Implementation](../../src/ciphers/rot13.rs)
