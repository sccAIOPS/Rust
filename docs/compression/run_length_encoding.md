# Run-Length Encoding (RLE)

## 1. Overview

Run-Length Encoding (RLE) is one of the simplest and oldest data compression techniques. It was first patented by Golomb in 1966, though the concept predates the patent. RLE works by replacing consecutive sequences of the same data value (called "runs") with a single value and a count.

RLE is particularly effective for data that contains many such runs, such as:
- Simple graphic images (like icons, line drawings, and animations)
- Fax transmissions
- PCX image format
- BMP files (as an optional compression scheme)
- TIFF files

The algorithm is a form of **lossless compression**, meaning the original data can be perfectly reconstructed from the compressed data.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an input sequence $S = s_1, s_2, ..., s_n$ where $s_i$ are elements from an alphabet $\Sigma$, find a compressed representation $C$ such that:

1. $|C| \leq |S|$ for inputs with repeated consecutive elements
2. The original sequence $S$ can be perfectly reconstructed from $C$

### 2.2 Mathematical Model

**Input Specification:**
- A string $S$ of length $n$ over alphabet $\Sigma$

**Output Specification (Encoding):**
- A sequence of pairs $(c_i, k_i)$ where:
  - $c_i \in \Sigma$ is a character
  - $k_i \in \mathbb{Z}^+$ is the run length (count)

**Compression Property:**

For a string with $r$ runs, the encoded output has $r$ pairs. The compression ratio is:

$$\text{Compression Ratio} = \frac{\text{Original Size}}{\text{Compressed Size}} = \frac{n}{2r}$$

(assuming each pair takes 2 units of storage)

**Effectiveness Condition:**

RLE achieves compression when:
$$r < \frac{n}{2}$$

This means the average run length must be greater than 2.

### 2.3 Correctness Proof

**Claim:** The RLE encoding and decoding functions are inverses.

**Proof:**

Let $\text{encode}: \Sigma^* \rightarrow (\Sigma \times \mathbb{Z}^+)^*$ and $\text{decode}: (\Sigma \times \mathbb{Z}^+)^* \rightarrow \Sigma^*$

For any string $S = s_1 s_2 ... s_n$:

1. **Encoding preserves information:** Each maximal run of character $c$ with length $k$ is uniquely represented as $(c, k)$.

2. **Decoding reconstructs runs:** Each pair $(c, k)$ expands to exactly $k$ copies of $c$.

3. **Composition:** $\text{decode}(\text{encode}(S)) = S$ because:
   - The partition of $S$ into maximal runs is unique
   - Each run is encoded and decoded without loss
   - The order of runs is preserved

Therefore, RLE is **lossless** and **invertible**. ∎

## 3. Algorithm Description

### 3.1 Intuition

**Encoding:**
Walk through the string from left to right. When you see the same character repeated, keep counting. When a different character appears (or you reach the end), output the character and its count, then start counting the new character.

**Decoding:**
For each (character, count) pair, output the character repeated `count` times.

### 3.2 Pseudocode

**Encoding:**
```
function RLE_ENCODE(text):
    if text is empty:
        return empty list
    
    encoded ← empty list
    count ← 1
    
    for i from 0 to length(text) - 1:
        if i + 1 < length(text) AND text[i] = text[i + 1]:
            count ← count + 1
        else:
            append (text[i], count) to encoded
            count ← 1
    
    return encoded
```

**Decoding:**
```
function RLE_DECODE(encoded):
    result ← empty string
    
    for each (char, count) in encoded:
        append char repeated count times to result
    
    return result
```

### 3.3 Step-by-Step Example

**Encoding Example:**

Input: `"AAAABBBCCDAA"`

| Step | i | Current Char | Next Char | Count | Action | Encoded |
|------|---|--------------|-----------|-------|--------|---------|
| 1 | 0 | A | A | 2 | Continue | - |
| 2 | 1 | A | A | 3 | Continue | - |
| 3 | 2 | A | A | 4 | Continue | - |
| 4 | 3 | A | B | 4 | Output | [('A', 4)] |
| 5 | 4 | B | B | 2 | Continue | - |
| 6 | 5 | B | B | 3 | Continue | - |
| 7 | 6 | B | C | 3 | Output | [('A', 4), ('B', 3)] |
| 8 | 7 | C | C | 2 | Continue | - |
| 9 | 8 | C | D | 2 | Output | [('A', 4), ('B', 3), ('C', 2)] |
| 10 | 9 | D | A | 1 | Output | [('A', 4), ('B', 3), ('C', 2), ('D', 1)] |
| 11 | 10 | A | A | 2 | Continue | - |
| 12 | 11 | A | - | 2 | Output | [('A', 4), ('B', 3), ('C', 2), ('D', 1), ('A', 2)] |

**Decoding Example:**

Input: `[('A', 4), ('B', 3), ('C', 2), ('D', 1), ('A', 2)]`

| Pair | Output Segment | Cumulative Result |
|------|----------------|-------------------|
| ('A', 4) | "AAAA" | "AAAA" |
| ('B', 3) | "BBB" | "AAAABBB" |
| ('C', 2) | "CC" | "AAAABBBCC" |
| ('D', 1) | "D" | "AAAABBBCCD" |
| ('A', 2) | "AA" | "AAAABBBCCDAA" |

## 4. Complexity Analysis

### 4.1 Time Complexity

**Encoding:**
- **Best Case:** $O(n)$ — Single pass through the input
- **Average Case:** $O(n)$ — Always processes each character once
- **Worst Case:** $O(n)$ — No character comparisons shortcut processing

**Decoding:**
- **Best Case:** $O(m)$ where $m$ is output length (minimum when all counts are 1)
- **Average Case:** $O(m)$ — Linear in output size
- **Worst Case:** $O(m)$ — When decompressing highly compressed data

**Derivation:**

For encoding, we iterate through $n$ characters exactly once, performing $O(1)$ operations per character. Thus: $T(n) = n \cdot O(1) = O(n)$

For decoding with $r$ pairs producing output of length $m$:
$$T(r, m) = \sum_{i=1}^{r} k_i = m = O(m)$$

### 4.2 Space Complexity

**Encoding:**
- **Auxiliary Space:** $O(r)$ where $r$ is the number of runs
- **Best Case:** $O(1)$ when the entire string is one run
- **Worst Case:** $O(n)$ when no characters repeat (each char is its own run)

**Decoding:**
- **Auxiliary Space:** $O(m)$ for the output string
- This is unavoidable as we must store the decompressed result

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Ownership and Borrowing:**
```rust
pub fn run_length_encode(text: &str) -> Vec<(char, i32)>
```
- Takes a string slice (`&str`) to avoid ownership transfer
- Returns owned `Vec` since the caller needs the encoded data

**Iterator Usage:**
```rust
let res = encoded
    .iter()
    .map(|x| (x.0).to_string().repeat(x.1 as usize))
    .collect::<String>();
```
- Uses iterators for idiomatic decoding
- `collect::<String>()` efficiently concatenates results

**Unicode Considerations:**
- Uses `chars()` iterator for proper Unicode handling
- Note: Current implementation uses `nth()` which is $O(n)$ for each access
- A more efficient approach would convert to `Vec<char>` first

### 5.2 Edge Cases

| Case | Input | Expected Output (Encode) |
|------|-------|--------------------------|
| Empty string | `""` | `[]` |
| Single character | `"A"` | `[('A', 1)]` |
| All same | `"AAAA"` | `[('A', 4)]` |
| All different | `"ABCD"` | `[('A',1), ('B',1), ('C',1), ('D',1)]` |
| Unicode | `"日日本"` | `[('日', 2), ('本', 1)]` |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**Image Compression:**
- **Fax machines:** CCITT Group 3/4 uses RLE for bi-level images
- **BMP format:** Supports RLE4 and RLE8 compression modes
- **PCX format:** Uses RLE as its primary compression
- **TGA format:** Optional RLE compression

**Data Transmission:**
- Run-length limited (RLL) encoding in hard disk drives
- Efficient transmission of sparse data matrices

**Gaming:**
- Tile-based game maps with repeated terrain
- Sprite sheet compression for simple animations

**Bioinformatics:**
- DNA sequence representation (sequences often have repeated bases)

### 6.2 Related Algorithms

| Algorithm | When to Use |
|-----------|-------------|
| **RLE** | Data with many consecutive repeats |
| **Huffman Coding** | Variable symbol frequencies, no positional patterns |
| **LZ77/LZ78** | Repeated patterns (not just consecutive) |
| **Burrows-Wheeler + MTF + RLE** | General text compression pipeline |

**Compression Pipeline:**

RLE is often used as part of a compression pipeline:

```
Input → BWT → MTF → RLE → Entropy Coding → Compressed
```

The Burrows-Wheeler Transform groups similar characters, MTF converts to small integers, RLE compresses the runs of zeros, and finally entropy coding (like Huffman) compresses the result.

## 7. References

1. **Golomb, S. W.** (1966). "Run-length encodings". IEEE Transactions on Information Theory. IT-12 (3): 399–401.

2. **Salomon, D.** (2007). *Data Compression: The Complete Reference* (4th ed.). Springer. ISBN 978-1-84628-602-5.

3. **Wikipedia**: [Run-length encoding](https://en.wikipedia.org/wiki/Run-length_encoding)

4. **Sayood, K.** (2017). *Introduction to Data Compression* (5th ed.). Morgan Kaufmann.

5. **Implementation**: [TheAlgorithms/Rust - run_length_encoding.rs](https://github.com/TheAlgorithms/Rust/blob/master/src/compression/run_length_encoding.rs)
