# String Reversal

## 1. Overview

String reversal is one of the most fundamental string operations, transforming a string to read backwards. While conceptually simple, efficient implementation requires understanding of string encoding, memory allocation, and iteration patterns. It serves as a building block for more complex algorithms like palindrome detection.

## 2. Mathematical Foundation

### 2.1 Definition

The **reverse** of string $S = s_0s_1...s_{n-1}$ is:
$$S^R = s_{n-1}s_{n-2}...s_1s_0$$

Where $S^R_i = S_{n-1-i}$ for all $i \in [0, n)$.

### 2.2 Properties

1. **Involution:** $(S^R)^R = S$
2. **Length Preservation:** $|S^R| = |S|$
3. **Concatenation:** $(AB)^R = B^RA^R$
4. **Palindrome:** $S$ is palindrome $\iff S = S^R$
5. **Empty String:** $\epsilon^R = \epsilon$

## 3. Algorithm Description

### 3.1 In-Place Two-Pointer

```
function REVERSE_IN_PLACE(s):
    left = 0
    right = len(s) - 1
    
    while left < right:
        swap(s[left], s[right])
        left += 1
        right -= 1
```

### 3.2 Using Extra Space

```
function REVERSE_COPY(s):
    result = empty string
    for i = len(s) - 1 down to 0:
        result.append(s[i])
    return result
```

### 3.3 Recursive Approach

```
function REVERSE_RECURSIVE(s):
    if len(s) <= 1:
        return s
    return s[last] + REVERSE_RECURSIVE(s[1:-1]) + s[first]
```

### 3.4 Step-by-Step Example

**String:** "hello"

**Two-Pointer Method:**

| Step | left | right | Array State |
|------|------|-------|-------------|
| 0 | 0 | 4 | [h,e,l,l,o] |
| 1 | 1 | 3 | [o,e,l,l,h] |
| 2 | 2 | 2 | [o,l,l,e,h] |

**Result:** "olleh"

## 4. Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| In-place | $O(n)$ | $O(1)$ |
| Copy | $O(n)$ | $O(n)$ |
| Recursive | $O(n)$ | $O(n)$ stack |

## 5. Implementation Notes

### 5.1 Rust Implementation - Basic

```rust
pub fn reverse(s: &str) -> String {
    s.chars().rev().collect()
}
```

### 5.2 In-Place on Byte Array

```rust
pub fn reverse_bytes_in_place(bytes: &mut [u8]) {
    let n = bytes.len();
    for i in 0..n / 2 {
        bytes.swap(i, n - 1 - i);
    }
}
```

### 5.3 Unicode-Aware Reversal

```rust
pub fn reverse_graphemes(s: &str) -> String {
    use unicode_segmentation::UnicodeSegmentation;
    s.graphemes(true).rev().collect()
}

// Without external crate (basic chars)
pub fn reverse_chars(s: &str) -> String {
    s.chars().rev().collect()
}
```

### 5.4 Word-Level Reversal

```rust
pub fn reverse_words(s: &str) -> String {
    s.split_whitespace()
        .rev()
        .collect::<Vec<_>>()
        .join(" ")
}

// Reverse each word in place
pub fn reverse_each_word(s: &str) -> String {
    s.split_whitespace()
        .map(|word| word.chars().rev().collect::<String>())
        .collect::<Vec<_>>()
        .join(" ")
}
```

### 5.5 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| "" | "" | Empty string |
| "a" | "a" | Single character |
| "ab" | "ba" | Two characters |
| "Hello 世界" | "界世 olleH" | Unicode (chars) |
| "café" | "éfac" | Diacritics |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Palindrome Detection:**
   - Compare string with its reverse
   - Building block for DNA analysis

2. **Stack Simulation:**
   - LIFO ordering
   - Expression evaluation

3. **Text Processing:**
   - Reverse reading direction
   - Encryption schemes

4. **Algorithm Building Blocks:**
   - Suffix array construction
   - String matching (reverse pattern)

### 6.2 Example Applications

**Palindrome Check:**
```rust
fn is_palindrome(s: &str) -> bool {
    let cleaned: String = s
        .chars()
        .filter(|c| c.is_alphanumeric())
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    cleaned == reverse(&cleaned)
}
```

**Reverse DNS Lookup Format:**
```rust
fn to_reverse_dns(ip: &str) -> String {
    ip.split('.')
        .rev()
        .collect::<Vec<_>>()
        .join(".")
        + ".in-addr.arpa"
}
```

## 7. Variations

### 7.1 Reverse Substring

```rust
pub fn reverse_substring(s: &str, start: usize, end: usize) -> String {
    let chars: Vec<char> = s.chars().collect();
    let mut result: Vec<char> = chars.clone();
    
    let end = end.min(chars.len());
    for i in start..end {
        result[i] = chars[start + end - 1 - i];
    }
    
    result.into_iter().collect()
}
```

### 7.2 Rotate String

```rust
pub fn rotate_left(s: &str, k: usize) -> String {
    if s.is_empty() { return s.to_string(); }
    let k = k % s.len();
    format!("{}{}", &s[k..], &s[..k])
}

pub fn rotate_right(s: &str, k: usize) -> String {
    if s.is_empty() { return s.to_string(); }
    let k = k % s.len();
    format!("{}{}", &s[s.len()-k..], &s[..s.len()-k])
}
```

### 7.3 Reverse with K-Groups

```rust
pub fn reverse_k_group(s: &str, k: usize) -> String {
    s.chars()
        .collect::<Vec<_>>()
        .chunks(k)
        .map(|chunk| chunk.iter().rev().collect::<String>())
        .collect()
}
```

## 8. Unicode Considerations

### 8.1 Grapheme Clusters

Characters like emojis or accented letters may consist of multiple code points:

```rust
// "e\u{0301}" (e + combining accent) should reverse as one unit
// Using unicode-segmentation crate is recommended for correctness
```

### 8.2 Byte vs Char vs Grapheme

| Level | "café" | Reversed |
|-------|--------|----------|
| Bytes | [99,97,102,195,169] | Invalid UTF-8! |
| Chars | ['c','a','f','é'] | "éfac" |
| Graphemes | ["c","a","f","é"] | "éfac" |

## 9. Performance Notes

| String Length | Time (approx) |
|---------------|---------------|
| 100 | ~100ns |
| 10,000 | ~10μs |
| 1,000,000 | ~1ms |

String reversal is memory-bound; consider allocation patterns for performance.

## 10. References

1. Sedgewick, R. "Algorithms", Chapter on String Processing
2. Rust String documentation: https://doc.rust-lang.org/std/string/
3. Unicode Standard Annex #29: Text Segmentation

## Implementation

See: [src/string/reverse.rs](../../src/string/reverse.rs)
