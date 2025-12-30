# Swap Odd and Even Bits

## 1. Overview

This algorithm swaps all odd-positioned bits with adjacent even-positioned bits in a 32-bit integer. Bit positions are 0-indexed from the right (least significant bit). So bit 0 swaps with bit 1, bit 2 swaps with bit 3, and so on.

This operation has applications in graphics processing, data encoding, and bit-level permutations.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a 32-bit unsigned integer $n$, produce an integer $m$ where:
- Bit $2k$ of $n$ becomes bit $2k+1$ of $m$
- Bit $2k+1$ of $n$ becomes bit $2k$ of $m$
- For all $k \in [0, 15]$

### 2.2 Mathematical Model

**Input:** A 32-bit unsigned integer $n$

**Output:** Integer $m$ with odd and even bits swapped

**Visual Representation:**
```
Position: 7  6  5  4  3  2  1  0
Input:    b7 b6 b5 b4 b3 b2 b1 b0
Output:   b6 b7 b4 b5 b2 b3 b0 b1
```

### 2.3 Key Masks

| Mask | Hex | Binary Pattern | Purpose |
|------|-----|----------------|---------|
| Even bits | 0xAAAAAAAA | 10101010... | Selects bits 1,3,5,7,... |
| Odd bits | 0x55555555 | 01010101... | Selects bits 0,2,4,6,... |

### 2.4 Algorithm Formula

$$m = ((n \land \text{0xAAAAAAAA}) >> 1) \lor ((n \land \text{0x55555555}) << 1)$$

## 3. Algorithm Description

### 3.1 Intuition

1. Extract all even-positioned bits (using mask 0xAAAAAAAA)
2. Extract all odd-positioned bits (using mask 0x55555555)
3. Shift even bits right by 1 (moving them to odd positions)
4. Shift odd bits left by 1 (moving them to even positions)
5. Combine the results with OR

### 3.2 Pseudocode

```
function swap_odd_even_bits(num):
    // 0xAAAAAAAA has 1s at even positions (1,3,5,7...)
    even_bits = num AND 0xAAAAAAAA
    
    // 0x55555555 has 1s at odd positions (0,2,4,6...)
    odd_bits = num AND 0x55555555
    
    // Shift even bits right (to odd positions)
    // Shift odd bits left (to even positions)
    return (even_bits >> 1) OR (odd_bits << 1)
```

### 3.3 Step-by-Step Example

**Input:** `num = 23` (binary: `00010111`)

| Step | Operation | Value | Binary (8 bits) |
|------|-----------|-------|-----------------|
| 1 | num | 23 | 00010111 |
| 2 | 0xAA (mask) | 170 | 10101010 |
| 3 | even_bits = num & 0xAA | 2 | 00000010 |
| 4 | 0x55 (mask) | 85 | 01010101 |
| 5 | odd_bits = num & 0x55 | 21 | 00010101 |
| 6 | even_bits >> 1 | 1 | 00000001 |
| 7 | odd_bits << 1 | 42 | 00101010 |
| 8 | result = 1 | 42 | 43 | 00101011 |

**Result:** `43`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| All | $O(1)$ | Fixed number of bitwise operations |

### 4.2 Space Complexity

- **Auxiliary Space:** $O(1)$
- **Total Space:** $O(1)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn swap_odd_even_bits(num: u32) -> u32 {
    // 0xAAAAAAAA selects even-positioned bits (1, 3, 5, ...)
    let even_bits = num & 0xAAAAAAAA;
    
    // 0x55555555 selects odd-positioned bits (0, 2, 4, ...)
    let odd_bits = num & 0x55555555;
    
    // Swap by shifting and combining
    (even_bits >> 1) | (odd_bits << 1)
}
```

**Key patterns:**
- Uses `u32` for unsigned operations
- Hexadecimal masks for clarity
- Single expression possible: `((num & 0xAAAAAAAA) >> 1) | ((num & 0x55555555) << 1)`

### 5.2 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | 0 | All zeros unchanged |
| 1 | 2 | 01 → 10 |
| 2 | 1 | 10 → 01 |
| 3 | 3 | 11 → 11 (palindrome) |
| 0xFFFFFFFF | 0xFFFFFFFF | All ones (palindrome) |
| 0xAAAAAAAA | 0x55555555 | Pattern swap |
| 0x55555555 | 0xAAAAAAAA | Pattern swap |

### 5.3 Involution Property

This operation is its own inverse:
```rust
assert_eq!(swap_odd_even_bits(swap_odd_even_bits(n)), n);
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Graphics Processing:** Pixel data manipulation
2. **Data Encoding:** Custom encoding schemes
3. **Checksum Algorithms:** Bit permutation steps
4. **Network Protocols:** Bit reordering for transmission
5. **Cryptography:** Permutation operations in block ciphers

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| `reverse_bits` | Complete bit reversal |
| Bit permutation | General case |
| Byte swap | At byte granularity |
| Interleave bits | Related transformation |

### 6.3 Generalizations

| Operation | Description |
|-----------|-------------|
| Swap nibbles | Swap 4-bit groups |
| Swap bytes | Swap 8-bit groups |
| Bit interleave | Interleave bits from two numbers |

## 7. References

1. Warren, H. "Hacker's Delight." Addison-Wesley, 2012. Chapter 7.
2. Anderson, S. "Bit Twiddling Hacks." Stanford Graphics.
3. Knuth, D. "The Art of Computer Programming, Vol. 4A."
