# Reverse Bits

## 1. Overview

The Reverse Bits algorithm reverses the bit order of a 32-bit unsigned integer. The least significant bit becomes the most significant bit and vice versa. This operation is fundamental in digital signal processing, cryptography, and network protocols.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a 32-bit unsigned integer $n$, produce an integer $m$ where:
- Bit $i$ of $n$ becomes bit $(31-i)$ of $m$
- For all $i \in [0, 31]$

### 2.2 Mathematical Model

**Input:** A 32-bit unsigned integer $n = \sum_{i=0}^{31} b_i \cdot 2^i$

**Output:** $m = \sum_{i=0}^{31} b_i \cdot 2^{31-i}$

**Visual Representation:**
```
Input:  b31 b30 b29 ... b2 b1 b0
Output: b0  b1  b2  ... b29 b30 b31
```

### 2.3 Key Properties

1. **Involution:** Reversing twice returns the original: $\text{reverse}(\text{reverse}(n)) = n$
2. **Preserves bit count:** Same number of 1-bits in input and output
3. **Palindrome invariant:** Bit-palindromes are fixed points

## 3. Algorithm Description

### 3.1 Intuition

Process all 32 bits from right to left:
1. Extract the rightmost bit of the input
2. Append it to the result (which is being built left to right)
3. Shift input right and result left
4. Repeat 32 times

### 3.2 Pseudocode

```
function reverse_bits(n):
    result = 0
    
    for i = 0 to 31:
        result = result << 1          // Make room for next bit
        result = result | (n & 1)     // Add rightmost bit of n
        n = n >> 1                    // Move to next bit of n
    
    return result
```

### 3.3 Step-by-Step Example

**Input:** `n = 13` (binary: `00000000000000000000000000001101`)

| Step | n (last 4 bits) | result (first 4 bits) | Action |
|------|-----------------|----------------------|--------|
| 0 | 1101 | 0000 | Extract 1, shift |
| 1 | 0110 | 0001 | Extract 0, shift |
| 2 | 0011 | 0010 | Extract 1, shift |
| 3 | 0001 | 0101 | Extract 1, shift |
| ... | ... | ... | Continue 28 more times |

**Output:** `2952790016` (binary: `10110000000000000000000000000000`)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| All | $O(1)$ | Always exactly 32 iterations |

Note: Though the loop runs 32 times, this is a constant, making it $O(1)$.

### 4.2 Space Complexity

- **Auxiliary Space:** $O(1)$
- **Total Space:** $O(1)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn reverse_bits(n: u32) -> u32 {
    let mut result: u32 = 0;
    let mut num = n;
    
    for _ in 0..32 {
        result <<= 1;        // Shift result left
        result |= num & 1;   // Add LSB of num
        num >>= 1;           // Shift num right
    }
    
    result
}
```

**Key patterns:**
- Uses `u32` for unsigned 32-bit operations
- Bitwise operations: `<<`, `>>`, `|`, `&`
- Explicit loop count for clarity

### 5.2 Alternative: Using Built-in

Rust provides a built-in method:

```rust
pub fn reverse_bits_builtin(n: u32) -> u32 {
    n.reverse_bits()
}
```

### 5.3 Optimized Version (Divide and Conquer)

```rust
pub fn reverse_bits_fast(mut n: u32) -> u32 {
    n = ((n & 0xFFFF0000) >> 16) | ((n & 0x0000FFFF) << 16);
    n = ((n & 0xFF00FF00) >> 8)  | ((n & 0x00FF00FF) << 8);
    n = ((n & 0xF0F0F0F0) >> 4)  | ((n & 0x0F0F0F0F) << 4);
    n = ((n & 0xCCCCCCCC) >> 2)  | ((n & 0x33333333) << 2);
    n = ((n & 0xAAAAAAAA) >> 1)  | ((n & 0x55555555) << 1);
    n
}
```

This version runs in $O(\log w)$ where $w$ is the bit width.

### 5.4 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | 0 | All zeros remain zeros |
| 1 | 2147483648 | $2^{31}$ |
| 0xFFFFFFFF | 0xFFFFFFFF | All ones (palindrome) |
| 43261596 | 964176192 | Standard test case |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **FFT Algorithms:** Bit-reversal permutation in Cooley-Tukey FFT
2. **Network Protocols:** Byte/bit ordering conversions
3. **Cryptography:** Permutation operations in ciphers
4. **Image Processing:** Mirroring operations
5. **Checksum Calculations:** CRC computations

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| `reverse_bytes` | Reverse at byte level |
| `swap_odd_even_bits` | Partial bit reordering |
| FFT bit-reversal | Uses this as subroutine |

## 7. References

1. Warren, H. "Hacker's Delight." Addison-Wesley, 2012. Chapter 7.
2. Cooley, J. & Tukey, J. "An Algorithm for the Machine Calculation of Complex Fourier Series."
3. Intel. "BSWAP - Byte Swap" instruction documentation.
