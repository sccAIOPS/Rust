# Baconian Cipher

## 1. Overview

The **Baconian Cipher** is a steganographic method of hiding messages, attributed to Sir Francis Bacon in 1605. Rather than encrypting by substitution or transposition, it encodes each letter as a sequence of two symbols (traditionally 'a' and 'b'), which can then be hidden in a carrier text using two different typefaces.

### Historical Context
- **1605**: Francis Bacon publishes "Of the Advancement of Learning"
- **1623**: Described in "De Augmentis Scientiarum"
- **Use**: Historical diplomatic communications
- **Present**: Educational tool, steganography concepts

### Classification

The Baconian cipher is both:
- **Substitution cipher**: Letters → binary-like codes
- **Steganographic system**: Hiding messages in plain sight

## 2. Mathematical Foundation

### 2.1 Encoding Scheme

Each letter is represented by a 5-bit binary-like sequence using 'A' and 'B' (or 0 and 1):

**Original Bacon Alphabet (24 letters, I=J, U=V)**:
```
A = AAAAA    N = ABBAB
B = AAAAB    O = ABBBA
C = AAABA    P = ABBBB
D = AAABB    Q = BAAAA
E = AABAA    R = BAAAB
F = AABAB    S = BAABA
G = AABBA    T = BAABB
H = AABBB    U/V = BABAA
I/J = ABAAA   W = BABAB
K = ABAAB    X = BABBA
L = ABABA    Y = BABBB
M = ABABB    Z = BBAAA
```

**Modern 26-letter Version**:
Uses all 32 combinations of 5-bit codes.

### 2.2 Mathematical Model

$$encode(letter) = binary(position - 1, 5\ bits)$$
$$decode(code) = alphabet[binary\_to\_int(code) + 1]$$

Where position is 1-26 for A-Z.

## 3. Algorithm Description

### 3.1 Encoding Process

1. Convert each letter to its 5-character A/B sequence
2. Concatenate all sequences

### 3.2 Steganographic Hiding

**Method 1: Two Typefaces**
- 'A' bits → regular typeface
- 'B' bits → italic (or bold)

**Method 2: Case Alternation**
- 'A' bits → lowercase
- 'B' bits → uppercase

**Method 3: Spacing/Formatting**
- 'A' bits → normal spacing
- 'B' bits → extra space

### 3.3 Pseudocode

```
FUNCTION bacon_encode(plaintext):
    result ← ""
    FOR each letter c in plaintext:
        c ← uppercase(c)
        IF c is alphabetic:
            index ← position of c in alphabet (0-25)
            FOR i FROM 4 DOWNTO 0:
                IF (index >> i) AND 1 = 1:
                    result ← result + 'B'
                ELSE:
                    result ← result + 'A'
    RETURN result

FUNCTION bacon_decode(ciphertext):
    plaintext ← ""
    FOR i FROM 0 TO len(ciphertext) - 1 STEP 5:
        code ← ciphertext[i:i+5]
        value ← 0
        FOR j FROM 0 TO 4:
            IF code[j] = 'B':
                value ← value + (1 << (4 - j))
        IF value < 26:
            plaintext ← plaintext + chr(ord('A') + value)
    RETURN plaintext
```

### 3.4 Step-by-Step Example

**Message**: "HI"

| Letter | Position | Binary | Bacon Code |
|--------|----------|--------|------------|
| H | 8 | 00111 | AABBB |
| I | 9 | 01000 | ABAAA |

**Encoded**: "AABBBABAAA"

**Hiding Example** (using case):
```
Cover text: "THeRe is A piG iN thE GaRDeN"
           "AABBB ABAAA" (ignoring spaces)
           
Read uppercase as B, lowercase as A:
T=B, H=B... → BABBB BABBA = wrong, let me redo:

Actually hide in: "sometext with hidden message"
If we mark each letter position:
s=A, o=A, m=A, e=B, t=B, e=B, x=B, t=A...
```

Better example:
```
Message: "A" = AAAAA

Cover: "hello" (all lowercase = 5 A's)
Encodes: AAAAA = 'A'

Message: "B" = AAAAB

Cover: "hellO" (4 lowercase + 1 uppercase)
Encodes: AAAAB = 'B'
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Encoding | O(n) |
| Decoding | O(n) |
| Hiding | O(5n) = O(n) |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Encoded message | O(5n) |
| Cover text | O(5n) minimum |
| **Expansion ratio** | **5:1** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn bacon_encode(plaintext: &str) -> String {
    plaintext
        .chars()
        .filter(|c| c.is_ascii_alphabetic())
        .map(|c| {
            let idx = c.to_ascii_uppercase() as u8 - b'A';
            (0..5)
                .map(|i| if (idx >> (4 - i)) & 1 == 1 { 'B' } else { 'A' })
                .collect::<String>()
        })
        .collect()
}

pub fn bacon_decode(ciphertext: &str) -> String {
    ciphertext
        .chars()
        .filter(|&c| c == 'A' || c == 'B' || c == 'a' || c == 'b')
        .map(|c| c.to_ascii_uppercase())
        .collect::<Vec<_>>()
        .chunks(5)
        .filter_map(|chunk| {
            if chunk.len() == 5 {
                let value: u8 = chunk.iter().enumerate().fold(0, |acc, (i, &c)| {
                    acc + if c == 'B' { 1 << (4 - i) } else { 0 }
                });
                if value < 26 {
                    Some((b'A' + value) as char)
                } else {
                    None
                }
            } else {
                None
            }
        })
        .collect()
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Non-alphabetic input | Skip |
| Mixed case A/B | Normalize to uppercase |
| Code > 25 | Invalid (skip or error) |
| Incomplete group | Discard (need 5 symbols) |

## 6. Security Analysis

### 6.1 Security Properties

| Property | Assessment |
|----------|------------|
| Key required | No (standard encoding) |
| Obscurity | Relies on hidden medium |
| Frequency analysis | Easy if encoding known |
| Pattern detection | Obvious once suspected |

### 6.2 Weaknesses

1. **Fixed encoding**: No key variation
2. **5:1 expansion**: Large overhead detectable
3. **Binary pattern**: Only 32 codes for 26 letters
4. **Historical**: Well-documented in cryptography texts

### 6.3 Steganographic Security

The security lies in:
- **Detection difficulty**: Is there a hidden message?
- **Carrier innocence**: Cover text appears normal
- **Awareness**: Requires knowing to look for Bacon cipher

## 7. Variants

### 7.1 24-Letter vs 26-Letter

**Original (24 letters)**:
- I = J = ABAAA
- U = V = BABAA
- Only 24 unique codes used

**Modern (26 letters)**:
- Each letter distinct
- Uses 26 of 32 possible codes

### 7.2 Alternative Symbols

| Symbol Set | 'A' equivalent | 'B' equivalent |
|------------|----------------|----------------|
| Binary | 0 | 1 |
| Typography | Roman | Italic |
| Case | lowercase | UPPERCASE |
| Font | serif | sans-serif |
| Image | light pixel | dark pixel |

### 7.3 Extended Baconian

Include digits and punctuation:
- 6 bits per symbol (64 possibilities)
- Or 7 bits (128 possibilities for ASCII)

## 8. Historical Usage

### 8.1 Bacon's Original Application

Bacon proposed using two typefaces in printed books:
```
Normal: This is a sample text with hidden
        message encoded in the typography

With hidden message (exaggerated):
This IS a sAMple Text WITH HIDden
     BB   ABB     BBBB BBBA
```

### 8.2 Claimed Historical Uses

| Claim | Status |
|-------|--------|
| Shakespeare authorship | Conspiracy theory (debunked) |
| Diplomatic communications | Some historical evidence |
| Secret societies | Unverified claims |

## 9. Modern Applications

### 9.1 Educational Use

| Purpose | Application |
|---------|-------------|
| Binary encoding | Introduction to binary |
| Steganography | Hiding vs encryption concepts |
| Information theory | Bits per character |

### 9.2 Digital Steganography

Bacon principles applied to:
- **Image steganography**: LSB encoding
- **Text steganography**: Whitespace, Unicode
- **Audio steganography**: Amplitude variations

## 10. Implementation Variations

### 10.1 Hiding in Text

```rust
pub fn hide_in_text(message: &str, cover: &str) -> Option<String> {
    let bacon = bacon_encode(message);
    let cover_letters: Vec<char> = cover.chars()
        .filter(|c| c.is_ascii_alphabetic())
        .collect();
    
    if cover_letters.len() < bacon.len() {
        return None; // Not enough cover text
    }
    
    let mut result = String::new();
    let mut bacon_idx = 0;
    
    for c in cover.chars() {
        if c.is_ascii_alphabetic() && bacon_idx < bacon.len() {
            let bacon_char = bacon.chars().nth(bacon_idx).unwrap();
            result.push(if bacon_char == 'B' {
                c.to_ascii_uppercase()
            } else {
                c.to_ascii_lowercase()
            });
            bacon_idx += 1;
        } else {
            result.push(c);
        }
    }
    
    Some(result)
}
```

### 10.2 Extracting from Text

```rust
pub fn extract_from_text(stego_text: &str) -> String {
    let bacon: String = stego_text
        .chars()
        .filter(|c| c.is_ascii_alphabetic())
        .map(|c| if c.is_ascii_uppercase() { 'B' } else { 'A' })
        .collect();
    
    bacon_decode(&bacon)
}
```

## 11. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Binary encoding | Bacon is 5-bit binary |
| ASCII | 7-bit character encoding |
| Morse Code | Variable-length binary |
| LSB Steganography | Similar hiding principle |

## 12. Fun Facts

1. Some claim Bacon hid messages proving he wrote Shakespeare's works
2. The cipher is one of the earliest binary encoding schemes
3. 5 bits can encode 32 values—Bacon used only 24 originally
4. The name "biliteral cipher" refers to using two symbols

## 13. References

- Bacon, F. (1623). De Augmentis Scientiarum
- Kahn, D. (1996). The Codebreakers
- Singh, S. (1999). The Code Book
- [Implementation](../../src/ciphers/baconian_cipher.rs)
