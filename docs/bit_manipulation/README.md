# Bit Manipulation Algorithms

This directory contains comprehensive documentation for bit manipulation algorithms implemented in the **TheAlgorithms/Rust** repository.

## Overview

**Bit manipulation** is the act of algorithmically manipulating bits using bitwise operations. These operations are fundamental to computer science and are extremely efficient since they operate directly on the binary representation of numbers at the hardware level.

### Key Characteristics

- **O(1) complexity**: Most operations complete in constant time
- **Memory efficient**: Operate on existing data without extra allocations
- **Hardware-level efficiency**: Map directly to CPU instructions
- **Foundation for optimization**: Many algorithms use bit tricks for speedups

### Fundamental Bitwise Operators

| Operator | Symbol | Description | Example |
|----------|--------|-------------|---------|
| AND | `&` | Sets bit to 1 if both bits are 1 | `5 & 3 = 1` |
| OR | `\|` | Sets bit to 1 if either bit is 1 | `5 \| 3 = 7` |
| XOR | `^` | Sets bit to 1 if bits are different | `5 ^ 3 = 6` |
| NOT | `!` | Inverts all bits | `!5 = -6` (signed) |
| Left Shift | `<<` | Shifts bits left, filling with 0 | `5 << 1 = 10` |
| Right Shift | `>>` | Shifts bits right | `5 >> 1 = 2` |

### Essential Bit Manipulation Tricks

```rust
// Check if number is odd/even
let is_odd = (n & 1) == 1;

// Multiply/divide by power of 2
let multiply_by_8 = n << 3;
let divide_by_4 = n >> 2;

// Check if power of 2
let is_power_of_two = n > 0 && (n & (n - 1)) == 0;

// Clear lowest set bit
let cleared = n & (n - 1);

// Isolate lowest set bit
let lowest = n & (-n);

// Swap without temp variable
a ^= b; b ^= a; a ^= b;
```

## Algorithm Index

| # | Algorithm | File | Description | Complexity |
|---|-----------|------|-------------|------------|
| 1 | [Is Power of Two](is_power_of_two.md) | `is_power_of_two.rs` | Check if number is power of 2 | $O(1)$ |
| 2 | [Count Trailing Zeros](count_trailing_zeros.md) | `binary_count_trailing_zeros.rs` | Count trailing zero bits | $O(1)$ |
| 3 | [Count Set Bits](counting_bits.md) | `counting_bits.rs` | Count number of 1-bits (popcount) | $O(k)$ |
| 4 | [Find Missing Number](find_missing_number.md) | `find_missing_number.rs` | Find missing element using XOR | $O(n)$ |
| 5 | [Find Unique Number](find_unique_number.md) | `find_unique_number.rs` | Find non-duplicate element using XOR | $O(n)$ |
| 6 | [Highest Set Bit](highest_set_bit.md) | `highest_set_bit.rs` | Find position of MSB | $O(\log n)$ |
| 7 | [Rightmost Set Bit](rightmost_set_bit.md) | `rightmost_set_bit.rs` | Find position of LSB | $O(1)$ |
| 8 | [Reverse Bits](reverse_bits.md) | `reverse_bits.rs` | Reverse all bits in integer | $O(w)$ |
| 9 | [Gray Code](gray_code.md) | `n_bits_gray_code.rs` | Generate Gray code sequence | $O(2^n)$ |
| 10 | [Two's Complement](twos_complement.md) | `twos_complement.rs` | Convert to two's complement | $O(\log n)$ |
| 11 | [Swap Odd-Even Bits](swap_odd_even_bits.md) | `swap_odd_even_bits.rs` | Swap adjacent bit pairs | $O(1)$ |
| 12 | [Sum Without Operators](sum_of_two_integers.md) | `sum_of_two_integers.rs` | Add integers using only bits | $O(w)$ |
| 13 | [BCD Encoding](binary_coded_decimal.md) | `binary_coded_decimal.rs` | Convert to Binary Coded Decimal | $O(d)$ |
| 14 | [Previous Power of Two](previous_power_of_two.md) | `find_previous_power_of_two.rs` | Find largest power of 2 ≤ n | $O(\log n)$ |

*Where: $k$ = number of set bits, $w$ = word size (32/64), $n$ = input value, $d$ = number of digits*

## Classification by Use Case

### Power of Two Operations
- **Is Power of Two**: Determine if a number is $2^k$
- **Previous Power of Two**: Find largest $2^k \leq n$
- **Highest Set Bit**: Position of most significant 1-bit

### Bit Position Operations
- **Count Trailing Zeros**: Position of rightmost 1-bit
- **Rightmost Set Bit**: Index of least significant 1-bit
- **Highest Set Bit**: Index of most significant 1-bit

### Bit Counting
- **Count Set Bits**: Population count (Hamming weight)
- **Count Trailing Zeros**: Number of trailing 0-bits

### XOR-Based Problems
- **Find Missing Number**: Exploit XOR cancellation property
- **Find Unique Number**: Single element among pairs

### Bit Transformation
- **Reverse Bits**: Mirror the bit pattern
- **Swap Odd-Even Bits**: Exchange adjacent bits
- **Gray Code**: Generate reflected binary code

### Number Representation
- **Two's Complement**: Signed integer representation
- **BCD Encoding**: Decimal digit encoding

### Arithmetic Without Operators
- **Sum Without Operators**: Implement addition with XOR/AND

## Complexity Comparison

| Algorithm | Time | Space | Key Insight |
|-----------|------|-------|-------------|
| Is Power of Two | $O(1)$ | $O(1)$ | `n & (n-1) == 0` |
| Count Trailing Zeros | $O(1)$ | $O(1)$ | Built-in or `n & -n` |
| Count Set Bits | $O(k)$ | $O(1)$ | Brian Kernighan's trick |
| Find Missing Number | $O(n)$ | $O(1)$ | XOR all elements + indices |
| Find Unique Number | $O(n)$ | $O(1)$ | XOR all elements |
| Highest Set Bit | $O(\log n)$ | $O(1)$ | Right shift until zero |
| Rightmost Set Bit | $O(1)$ | $O(1)$ | `n & -n` isolates LSB |
| Reverse Bits | $O(w)$ | $O(1)$ | Iterative extraction |
| Gray Code | $O(2^n)$ | $O(2^n)$ | Formula: `i ^ (i >> 1)` |
| Two's Complement | $O(\log n)$ | $O(1)$ | Flip bits and add 1 |
| Swap Odd-Even Bits | $O(1)$ | $O(1)$ | Mask and shift |
| Sum Without Operators | $O(w)$ | $O(1)$ | XOR + carry propagation |
| BCD Encoding | $O(d)$ | $O(d)$ | 4 bits per digit |
| Previous Power of Two | $O(\log n)$ | $O(1)$ | Shift until exceed |

## Important XOR Properties

XOR (`^`) is fundamental to many bit manipulation algorithms:

| Property | Expression | Explanation |
|----------|------------|-------------|
| Self-inverse | `a ^ a = 0` | Any number XORed with itself is 0 |
| Identity | `a ^ 0 = a` | XOR with 0 returns the original |
| Commutative | `a ^ b = b ^ a` | Order doesn't matter |
| Associative | `(a ^ b) ^ c = a ^ (b ^ c)` | Grouping doesn't matter |
| Cancellation | `a ^ b ^ b = a` | Pairs cancel out |

### XOR Applications

```rust
// Find single non-duplicate in array where all others appear twice
fn find_single(arr: &[i32]) -> i32 {
    arr.iter().fold(0, |acc, &x| acc ^ x)
}

// Swap without temporary
fn swap(a: &mut i32, b: &mut i32) {
    *a ^= *b;
    *b ^= *a;
    *a ^= *b;
}
```

## Common Bit Masks

| Mask | Binary (8-bit) | Purpose |
|------|----------------|---------|
| `0x55555555` | `01010101...` | Even bits (positions 0, 2, 4, ...) |
| `0xAAAAAAAA` | `10101010...` | Odd bits (positions 1, 3, 5, ...) |
| `0x33333333` | `00110011...` | Pairs of even bits |
| `0xCCCCCCCC` | `11001100...` | Pairs of odd bits |
| `0x0F0F0F0F` | `00001111...` | Lower nibble of each byte |
| `0xF0F0F0F0` | `11110000...` | Upper nibble of each byte |

## When to Use Bit Manipulation

### Good Use Cases
- Checking/setting flags in configuration
- Memory-efficient set operations (bit vectors)
- Cryptographic algorithms
- Low-level hardware programming
- Performance-critical code
- Embedded systems with limited resources

### Avoid When
- Code readability is paramount
- The optimization provides negligible benefit
- Higher-level abstractions are clearer

## Rust-Specific Considerations

### Built-in Methods
Rust provides optimized methods that compile to single CPU instructions:

```rust
let n: u32 = 42;
n.count_ones();        // Population count
n.count_zeros();       // Count zero bits
n.leading_zeros();     // Leading zero count
n.trailing_zeros();    // Trailing zero count
n.rotate_left(k);      // Circular left rotation
n.rotate_right(k);     // Circular right rotation
n.reverse_bits();      // Bit reversal
n.swap_bytes();        // Byte reversal
```

### Overflow Handling
```rust
// Wrapping arithmetic (wraps on overflow)
let result = a.wrapping_add(b);
let result = a.wrapping_sub(b);
let result = a.wrapping_neg();

// Checked arithmetic (returns Option)
let result = a.checked_add(b);

// Saturating arithmetic (clamps to min/max)
let result = a.saturating_add(b);
```

## Real-World Applications

1. **Graphics Programming**: Alpha blending, color manipulation
2. **Networking**: IP address manipulation, subnet masks
3. **Cryptography**: Block ciphers, hash functions
4. **Compression**: Huffman coding, bit packing
5. **Game Development**: Collision detection, state flags
6. **Databases**: Bitmap indexes, bloom filters
7. **Operating Systems**: Permission flags, memory management

## References

1. Warren, H. S. (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley.
2. Knuth, D. E. (2011). *The Art of Computer Programming, Vol. 4A*. Addison-Wesley.
3. Anderson, S. E. (2005). *Bit Twiddling Hacks*. Stanford University.
4. Intel® 64 and IA-32 Architectures Software Developer's Manual.
