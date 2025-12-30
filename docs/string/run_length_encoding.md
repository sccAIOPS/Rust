# Run-Length Encoding (RLE)

## 1. Overview

**Run-Length Encoding (RLE)** is a simple form of lossless data compression where consecutive identical elements (runs) are stored as a single data value and count. It's particularly effective for data with many such runs, like simple graphics, fax transmissions, and certain file formats.

## 2. Mathematical Foundation

### 2.1 Definition

Given a string $S = c_1^{r_1}c_2^{r_2}...c_k^{r_k}$ where each $c_i$ is a character and $r_i$ is its run length (consecutive occurrences), the RLE encoding is:

$$\text{RLE}(S) = (c_1, r_1)(c_2, r_2)...(c_k, r_k)$$

### 2.2 Properties

1. **Lossless:** Original data can be perfectly reconstructed
2. **Compression Ratio:** Best for long runs; worst case expands data
3. **Length:** $|RLE(S)| \leq 2|S|$ in worst case (alternating characters)
4. **Worst Case:** "abab" → "a1b1a1b1" (expansion)
5. **Best Case:** "aaaa" → "a4" (75% compression)

### 2.3 Compression Analysis

| Input Pattern | Original Length | Encoded Length | Ratio |
|---------------|-----------------|----------------|-------|
| "aaaa" | 4 | 2 | 50% |
| "aaaabbbb" | 8 | 4 | 50% |
| "abcd" | 4 | 8 | 200% |
| "aabbccdd" | 8 | 8 | 100% |

## 3. Algorithm Description

### 3.1 Encoding

```
function RLE_ENCODE(s):
    if s is empty:
        return ""
    
    result = []
    current_char = s[0]
    count = 1
    
    for i = 1 to len(s) - 1:
        if s[i] = current_char:
            count += 1
        else:
            result.append(current_char + str(count))
            current_char = s[i]
            count = 1
    
    result.append(current_char + str(count))
    return join(result)
```

### 3.2 Decoding

```
function RLE_DECODE(encoded):
    result = []
    i = 0
    
    while i < len(encoded):
        char = encoded[i]
        i += 1
        
        // Parse number
        num_str = ""
        while i < len(encoded) and encoded[i] is digit:
            num_str += encoded[i]
            i += 1
        
        count = parse_int(num_str)
        result.append(char repeated count times)
    
    return join(result)
```

### 3.3 Step-by-Step Example

**Encoding:** "AAABBBCCCCDA"

| Position | Char | Count | Action |
|----------|------|-------|--------|
| 0-2 | A | 3 | Continue |
| 3-5 | B | 3 | Output "A3", start B |
| 6-9 | C | 4 | Output "B3", start C |
| 10 | D | 1 | Output "C4", start D |
| 11 | A | 1 | Output "D1", start A |
| end | — | — | Output "A1" |

**Result:** "A3B3C4D1A1"

**Decoding:** "A3B3C4D1A1"

| Token | Char | Count | Output |
|-------|------|-------|--------|
| A3 | A | 3 | AAA |
| B3 | B | 3 | BBB |
| C4 | C | 4 | CCCC |
| D1 | D | 1 | D |
| A1 | A | 1 | A |

**Result:** "AAABBBCCCCDA"

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Encode | $O(n)$ |
| Decode | $O(n + m)$ |

Where $n$ is input length and $m$ is output length.

### 4.2 Space Complexity

| Operation | Complexity |
|-----------|------------|
| Encode | $O(n)$ worst case |
| Decode | $O(m)$ output size |

## 5. Implementation Notes

### 5.1 Rust Implementation - Basic

```rust
pub fn encode(input: &str) -> String {
    if input.is_empty() {
        return String::new();
    }
    
    let mut result = String::new();
    let mut chars = input.chars().peekable();
    
    while let Some(current) = chars.next() {
        let mut count = 1;
        
        while chars.peek() == Some(&current) {
            chars.next();
            count += 1;
        }
        
        result.push(current);
        result.push_str(&count.to_string());
    }
    
    result
}

pub fn decode(encoded: &str) -> String {
    let mut result = String::new();
    let mut chars = encoded.chars().peekable();
    
    while let Some(c) = chars.next() {
        let mut count_str = String::new();
        
        while let Some(&digit) = chars.peek() {
            if digit.is_ascii_digit() {
                count_str.push(digit);
                chars.next();
            } else {
                break;
            }
        }
        
        let count: usize = count_str.parse().unwrap_or(1);
        result.extend(std::iter::repeat(c).take(count));
    }
    
    result
}
```

### 5.2 Optimized Version with Pre-allocation

```rust
pub fn encode_optimized(input: &str) -> String {
    if input.is_empty() {
        return String::new();
    }
    
    let chars: Vec<char> = input.chars().collect();
    let mut result = String::with_capacity(input.len());
    
    let mut i = 0;
    while i < chars.len() {
        let current = chars[i];
        let mut count = 1;
        
        while i + count < chars.len() && chars[i + count] == current {
            count += 1;
        }
        
        result.push(current);
        result.push_str(&count.to_string());
        i += count;
    }
    
    result
}
```

### 5.3 Byte-Level RLE (for binary data)

```rust
pub fn encode_bytes(input: &[u8]) -> Vec<u8> {
    if input.is_empty() {
        return Vec::new();
    }
    
    let mut result = Vec::new();
    let mut i = 0;
    
    while i < input.len() {
        let current = input[i];
        let mut count: u8 = 1;
        
        while i + (count as usize) < input.len() 
              && input[i + (count as usize)] == current 
              && count < 255 
        {
            count += 1;
        }
        
        result.push(current);
        result.push(count);
        i += count as usize;
    }
    
    result
}

pub fn decode_bytes(encoded: &[u8]) -> Vec<u8> {
    let mut result = Vec::new();
    
    for chunk in encoded.chunks(2) {
        if chunk.len() == 2 {
            result.extend(std::iter::repeat(chunk[0]).take(chunk[1] as usize));
        }
    }
    
    result
}
```

### 5.4 Edge Cases

| Input | Encoded | Notes |
|-------|---------|-------|
| "" | "" | Empty |
| "a" | "a1" | Single char |
| "ab" | "a1b1" | No compression |
| "aaa" | "a3" | Good compression |
| "11" | "12" (ambiguous) | Digits problematic |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Image Compression:**
   - BMP files (simple RLE)
   - Fax transmissions (CCITT)
   - PCX image format

2. **File Formats:**
   - TGA images
   - PackBits (TIFF)
   - PDF content streams

3. **Game Development:**
   - Tile maps
   - Sprite compression

4. **Data Storage:**
   - Simple databases
   - Log compression

### 6.2 Format Variations

| Format | Encoding Style |
|--------|----------------|
| Basic | char + count |
| PackBits | flag byte + data |
| BMP RLE | escape sequences |
| CCITT | bit-level |

### 6.3 Example Applications

**Compress Log Lines:**
```rust
fn compress_repeated_lines(lines: &[&str]) -> Vec<(String, usize)> {
    if lines.is_empty() {
        return Vec::new();
    }
    
    let mut result = Vec::new();
    let mut current = lines[0];
    let mut count = 1;
    
    for &line in &lines[1..] {
        if line == current {
            count += 1;
        } else {
            result.push((current.to_string(), count));
            current = line;
            count = 1;
        }
    }
    result.push((current.to_string(), count));
    
    result
}
```

## 7. Variations

### 7.1 Modified RLE (Omit Count=1)

```rust
pub fn encode_modified(input: &str) -> String {
    let chars: Vec<char> = input.chars().collect();
    let mut result = String::new();
    let mut i = 0;
    
    while i < chars.len() {
        let current = chars[i];
        let mut count = 1;
        
        while i + count < chars.len() && chars[i + count] == current {
            count += 1;
        }
        
        result.push(current);
        if count > 1 {
            result.push_str(&count.to_string());
        }
        i += count;
    }
    
    result
}
```

### 7.2 PackBits Algorithm

```rust
// PackBits: negative count for literal run, positive for repeat
pub fn packbits_encode(input: &[u8]) -> Vec<u8> {
    // Complex implementation with literal and repeat runs
    todo!("Full PackBits implementation")
}
```

### 7.3 Escape-Based RLE

```rust
const ESCAPE: char = '#';

pub fn encode_escaped(input: &str) -> String {
    let mut result = String::new();
    let chars: Vec<char> = input.chars().collect();
    let mut i = 0;
    
    while i < chars.len() {
        let current = chars[i];
        let mut count = 1;
        
        while i + count < chars.len() && chars[i + count] == current {
            count += 1;
        }
        
        if count >= 4 || current == ESCAPE {
            result.push(ESCAPE);
            result.push(current);
            result.push_str(&count.to_string());
            result.push(ESCAPE);
        } else {
            result.extend(std::iter::repeat(current).take(count));
        }
        
        i += count;
    }
    
    result
}
```

## 8. Compression Comparison

| Method | Best For | Ratio (Typical) |
|--------|----------|-----------------|
| RLE | Long runs | 10-90% |
| Huffman | Varied frequency | 40-60% |
| LZ77 | Repeated patterns | 30-70% |
| gzip | General | 20-40% |

RLE is often used as a preprocessing step for other compression algorithms.

## 9. References

1. Salomon, D. "Data Compression: The Complete Reference"
2. Wikipedia: "Run-length encoding"
3. PNG Specification (uses RLE in filters)

## Implementation

See: [src/string/run_length_encoding.rs](../../src/string/run_length_encoding.rs)
