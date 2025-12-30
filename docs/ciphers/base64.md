# Base64 Encoding

## 1. Overview

**Base64** is a binary-to-text encoding scheme that represents binary data using 64 ASCII characters. It's designed to safely transmit binary data through text-only channels like email or URLs.

### Historical Context
- **1987**: Privacy Enhanced Mail (PEM) uses Base64
- **1992**: MIME standard includes Base64
- **1996**: Base64 for URLs described
- **Present**: Ubiquitous in web, email, data URIs

### Classification

Base64 is:
- **NOT encryption**: No security, fully reversible
- **Encoding scheme**: Binary → printable text
- **Lossless**: Perfect reconstruction

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: Binary data (bytes)
**Output**: ASCII string using 64 characters

### 2.2 The 64-Character Alphabet

**Standard Base64 (RFC 4648)**:
```
Index  Char    Index  Char    Index  Char    Index  Char
0      A       16     Q       32     g       48     w
1      B       17     R       33     h       49     x
2      C       18     S       34     i       50     y
3      D       19     T       35     j       51     z
4      E       20     U       36     k       52     0
5      F       21     V       37     l       53     1
6      G       22     W       38     m       54     2
7      H       23     X       39     n       55     3
8      I       24     Y       40     o       56     4
9      J       25     Z       41     p       57     5
10     K       26     a       42     q       58     6
11     L       27     b       43     r       59     7
12     M       28     c       44     s       60     8
13     N       29     d       45     t       61     9
14     O       30     e       46     u       62     +
15     P       31     f       47     v       63     /

Padding: =
```

### 2.3 Mathematical Model

**Encoding** converts 3 bytes (24 bits) → 4 Base64 characters (4 × 6 bits):

$$byte_0, byte_1, byte_2 \rightarrow char_0, char_1, char_2, char_3$$

Where each character represents 6 bits:
$$char_i = alphabet[\lfloor bits_{6i:6i+5} \rfloor]$$

### 2.4 Bit Manipulation

```
Input:  [aaaaaabb] [bbbbcccc] [ccdddddd]
         byte 0     byte 1     byte 2

Output: [00aaaaaa] [00bbbbbb] [00cccccc] [00dddddd]
         char 0     char 1     char 2     char 3
```

**Character extraction**:
- char₀ = byte₀ >> 2
- char₁ = ((byte₀ & 0x03) << 4) | (byte₁ >> 4)
- char₂ = ((byte₁ & 0x0F) << 2) | (byte₂ >> 6)
- char₃ = byte₂ & 0x3F

## 3. Algorithm Description

### 3.1 Encoding Pseudocode

```
FUNCTION base64_encode(data):
    alphabet ← "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    result ← ""
    
    // Process 3 bytes at a time
    FOR i FROM 0 TO len(data) - 1 STEP 3:
        // Get up to 3 bytes
        b0 ← data[i]
        b1 ← data[i+1] IF exists ELSE 0
        b2 ← data[i+2] IF exists ELSE 0
        
        // Convert to 4 6-bit values
        c0 ← b0 >> 2
        c1 ← ((b0 & 0x03) << 4) | (b1 >> 4)
        c2 ← ((b1 & 0x0F) << 2) | (b2 >> 6)
        c3 ← b2 & 0x3F
        
        // Encode
        result ← result + alphabet[c0]
        result ← result + alphabet[c1]
        result ← result + (alphabet[c2] IF i+1 < len(data) ELSE '=')
        result ← result + (alphabet[c3] IF i+2 < len(data) ELSE '=')
    
    RETURN result
```

### 3.2 Decoding Pseudocode

```
FUNCTION base64_decode(encoded):
    // Build reverse lookup
    decode_table ← {'A':0, 'B':1, ..., '/':63}
    
    // Remove padding
    encoded ← remove trailing '=' from encoded
    
    result ← []
    FOR i FROM 0 TO len(encoded) - 1 STEP 4:
        // Get 4 characters (or fewer at end)
        c0 ← decode_table[encoded[i]]
        c1 ← decode_table[encoded[i+1]]
        c2 ← decode_table[encoded[i+2]] IF exists
        c3 ← decode_table[encoded[i+3]] IF exists
        
        // Convert back to bytes
        b0 ← (c0 << 2) | (c1 >> 4)
        result.append(b0)
        
        IF c2 exists:
            b1 ← ((c1 & 0x0F) << 4) | (c2 >> 2)
            result.append(b1)
        
        IF c3 exists:
            b2 ← ((c2 & 0x03) << 6) | c3
            result.append(b2)
    
    RETURN result
```

### 3.3 Step-by-Step Example

**Input**: "Man" (ASCII: 77, 97, 110)

**Step 1**: Convert to binary
```
M: 01001101
a: 01100001
n: 01101110
```

**Step 2**: Regroup into 6-bit chunks
```
010011 010110 000101 101110
  19     22      5     46
```

**Step 3**: Look up in alphabet
```
19 → T
22 → W
5  → F
46 → u
```

**Output**: "TWFu"

### 3.4 Padding Example

**Input**: "Ma" (2 bytes, not multiple of 3)

```
M: 01001101
a: 01100001
   (padding zeros)

Regrouped: 010011 010110 0001[00]
              19     22    4

Output: TWE=
       (last char is padding because only 2 bytes)
```

**Input**: "M" (1 byte)

```
M: 01001101
   (padding zeros)

Regrouped: 010011 01[0000]
              19     16

Output: TQ==
       (2 padding chars because only 1 byte)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Encoding | O(n) |
| Decoding | O(n) |

Where n = input size in bytes.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Lookup table | O(64) = O(1) |
| Output | O(4n/3) ≈ O(n) |

### 4.3 Expansion Ratio

$$\text{Output size} = \lceil\frac{4 \times \text{Input size}}{3}\rceil$$

**Overhead**: ~33% increase in size

| Input bytes | Output chars | Padding |
|-------------|--------------|---------|
| 1 | 4 (2 data + 2 pad) | == |
| 2 | 4 (3 data + 1 pad) | = |
| 3 | 4 | none |
| n | ⌈4n/3⌉ | 0-2 |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
const ALPHABET: &[u8; 64] = b"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

pub fn base64_encode(data: &[u8]) -> String {
    let mut result = String::new();
    
    for chunk in data.chunks(3) {
        let b0 = chunk[0];
        let b1 = chunk.get(1).copied().unwrap_or(0);
        let b2 = chunk.get(2).copied().unwrap_or(0);
        
        result.push(ALPHABET[(b0 >> 2) as usize] as char);
        result.push(ALPHABET[(((b0 & 0x03) << 4) | (b1 >> 4)) as usize] as char);
        
        if chunk.len() > 1 {
            result.push(ALPHABET[(((b1 & 0x0F) << 2) | (b2 >> 6)) as usize] as char);
        } else {
            result.push('=');
        }
        
        if chunk.len() > 2 {
            result.push(ALPHABET[(b2 & 0x3F) as usize] as char);
        } else {
            result.push('=');
        }
    }
    
    result
}

pub fn base64_decode(encoded: &str) -> Vec<u8> {
    let decode_char = |c: char| -> u8 {
        match c {
            'A'..='Z' => c as u8 - b'A',
            'a'..='z' => c as u8 - b'a' + 26,
            '0'..='9' => c as u8 - b'0' + 52,
            '+' => 62,
            '/' => 63,
            _ => 0,
        }
    };
    
    let chars: Vec<char> = encoded.chars().filter(|&c| c != '=').collect();
    let mut result = Vec::new();
    
    for chunk in chars.chunks(4) {
        let c0 = decode_char(chunk[0]);
        let c1 = decode_char(chunk[1]);
        
        result.push((c0 << 2) | (c1 >> 4));
        
        if chunk.len() > 2 {
            let c2 = decode_char(chunk[2]);
            result.push(((c1 & 0x0F) << 4) | (c2 >> 2));
            
            if chunk.len() > 3 {
                let c3 = decode_char(chunk[3]);
                result.push(((c2 & 0x03) << 6) | c3);
            }
        }
    }
    
    result
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty input | Return empty |
| Non-Base64 chars | Skip or error |
| Missing padding | Some decoders accept |
| Extra padding | Invalid |
| Whitespace | Often allowed (strip) |

## 6. Variants

### 6.1 URL-Safe Base64

Replace characters problematic in URLs:
- `+` → `-`
- `/` → `_`
- Padding often omitted

### 6.2 Base64url (RFC 4648)

```
Standard:  ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
URL-safe:  ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_
```

### 6.3 Other Base Encodings

| Encoding | Characters | Efficiency |
|----------|------------|------------|
| Base16 (Hex) | 16 | 50% |
| Base32 | 32 | 62.5% |
| **Base64** | 64 | 75% |
| Base85 | 85 | 80% |

### 6.4 Base85 (Ascii85)

More efficient but less common:
- Uses chars 33-117 (excluding some)
- 4 bytes → 5 characters (vs 4 bytes → 5.33 chars in Base64)
- Used in PDF, PostScript

## 7. Applications

### 7.1 Common Use Cases

| Application | Example |
|-------------|---------|
| Email (MIME) | Attachments |
| Data URIs | `data:image/png;base64,...` |
| JSON/XML | Binary data in text formats |
| HTTP Auth | Basic Authentication header |
| JWT | JSON Web Tokens |
| Cookies | Binary data storage |

### 7.2 Example: Data URI

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJ
AAAA...
```

### 7.3 Example: Basic Auth

```
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
                     (username:password)
```

## 8. Security Considerations

### 8.1 NOT Encryption

⚠️ **Base64 provides NO security**:
- Trivially reversible
- No key required
- Constant-time decode

### 8.2 Common Mistakes

| Mistake | Reality |
|---------|---------|
| "Base64 encoded password" | Password visible to anyone |
| "Encrypted with Base64" | Not encryption |
| Base64 for obfuscation | Trivially reversed |

### 8.3 Legitimate Security Uses

| Use | Why OK |
|-----|--------|
| Encoding ciphertext | Transport encrypted data safely |
| Key material transport | After encryption |
| Hash output | Compact representation |

## 9. Performance Optimizations

### 9.1 SIMD Encoding

Modern implementations use SIMD for parallel processing:
- Process 12 bytes → 16 chars at once
- 4-6x speedup on modern CPUs

### 9.2 Lookup Table vs Computation

```rust
// Lookup (faster)
result = ALPHABET[index];

// Computation (slower but branchless)
result = if index < 26 { 
    b'A' + index 
} else if index < 52 { 
    b'a' + index - 26 
} ...
```

## 10. Related Encodings

| Encoding | Relationship |
|----------|--------------|
| Hex | Similar concept, 16 chars |
| Base32 | 32-char alphabet |
| Base85 | More efficient |
| URL encoding | Different purpose (escape) |

## 11. References

- RFC 4648: The Base16, Base32, and Base64 Data Encodings
- RFC 2045: MIME Part One (includes Base64)
- RFC 7515: JSON Web Signature (uses Base64url)
- [Implementation](../../src/ciphers/base64.rs)
