# Decimal to Binary Conversion

## 1. Overview

Decimal to Binary conversion transforms numbers from base-10 (decimal) representation to base-2 (binary) representation. This is a fundamental operation in computer science, as binary is the native language of digital computers.

**Historical Context**: The binary numeral system was formally introduced to Western mathematics by Gottfried Leibniz in 1679. Its practical application in computing became essential with the development of electronic computers in the 1940s, where electrical circuits naturally represent two states (on/off, high/low voltage).

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a non-negative decimal integer $D$, convert it to its binary representation $B = b_{n-1}b_{n-2}...b_1b_0$ where each $b_i \in \{0, 1\}$.

**Formal Definition**:

Find the unique sequence of bits $b_i$ such that:
$$D = \sum_{i=0}^{n-1} b_i \times 2^i$$

where $b_i \in \{0, 1\}$ and $n = \lfloor \log_2(D) \rfloor + 1$ (for $D > 0$).

### 2.2 Mathematical Model

**Input Specifications**:
- A non-negative integer $D \geq 0$ in decimal format
- Typically bounded by machine word size (e.g., `u64` supports $0 \leq D \leq 2^{64} - 1$)

**Output Specifications**:
- A string containing only characters '0' and '1'
- No leading zeros (except for the number 0 itself)
- Bit length: $n = \lfloor \log_2(D) \rfloor + 1$ for $D > 0$, or $n = 1$ for $D = 0$

**Key Properties**:
1. **Uniqueness**: Every decimal number has exactly one binary representation (without leading zeros)
2. **Bit Count**: A number $D$ requires $\lceil \log_2(D+1) \rceil$ bits to represent
3. **Modulo-2 Property**: The least significant bit (LSB) is $D \bmod 2$
4. **Division Property**: Dividing by 2 shifts bits right by one position

### 2.3 Correctness Proof

**Algorithm Invariant**: After $k$ iterations, the last $k$ bits of the result represent the binary form of the last $k$ bits of the original number.

**Loop Invariant**:
- Let $D_k$ be the value after $k$ divisions by 2
- Let $B_k$ be the first $k$ bits collected (in reverse order)
- Then: $D = D_k \times 2^k + B_k$

**Termination**: The algorithm terminates when $D_k = 0$, which occurs after $\lfloor \log_2(D) \rfloor + 1$ iterations.

**Correctness**: Upon termination, all bits have been collected, and reversing them produces the correct binary representation.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm uses repeated division by 2 (the base we're converting to). At each step:
1. The remainder ($D \bmod 2$) gives the next bit (from right to left)
2. The quotient ($\lfloor D / 2 \rfloor$) becomes the new number to process

This works because dividing by 2 in decimal is equivalent to a right shift in binary, which reveals each bit from least significant to most significant.

**Example**: Converting 13 to binary
- 13 ÷ 2 = 6 remainder **1** (LSB)
- 6 ÷ 2 = 3 remainder **0**
- 3 ÷ 2 = 1 remainder **1**
- 1 ÷ 2 = 0 remainder **1** (MSB)
- Result (reversed): 1101

### 3.2 Pseudocode

```
FUNCTION decimal_to_binary(decimal_number):
    IF decimal_number = 0 THEN
        RETURN "0"
    END IF
    
    binary_string ← empty_string
    num ← decimal_number
    
    WHILE num > 0 DO
        bit ← num MOD 2           // Get least significant bit
        APPEND bit to binary_string
        num ← num DIV 2            // Integer division (right shift)
    END WHILE
    
    RETURN reverse(binary_string)
END FUNCTION
```

**Line-by-line annotations**:
- Lines 2-4: Handle special case of zero
- Line 6: Initialize result accumulator
- Line 7: Working copy of input
- Line 9: Continue while there are bits to process
- Line 10: Extract least significant bit using modulo 2
- Line 11: Append bit to result (bits are collected in reverse order)
- Line 12: Remove processed bit by dividing by 2 (equivalent to right shift)
- Line 15: Reverse string since bits were collected LSB-first

### 3.3 Step-by-Step Example

**Input**: 92 (decimal)

| Iteration | num | num mod 2 (bit) | num div 2 | Binary String (reversed) |
|-----------|-----|-----------------|-----------|--------------------------|
| Init      | 92  | -               | -         | ""                       |
| 1         | 92  | 0               | 46        | "0"                      |
| 2         | 46  | 0               | 23        | "00"                     |
| 3         | 23  | 1               | 11        | "001"                    |
| 4         | 11  | 1               | 5         | "0011"                   |
| 5         | 5   | 1               | 2         | "00111"                  |
| 6         | 2   | 0               | 1         | "001110"                 |
| 7         | 1   | 1               | 0         | "0011101"                |

**Reverse**: "1011100"

**Verification**: 
- $1×2^6 + 0×2^5 + 1×2^4 + 1×2^3 + 1×2^2 + 0×2^1 + 0×2^0$
- $= 64 + 0 + 16 + 8 + 4 + 0 + 0 = 92$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Best Case**: $O(\log n)$ where $n$ is the input decimal number
  - Even for small numbers, we need at least $\log_2(n)$ iterations
  
- **Average Case**: $O(\log n)$
  - The number of bits in $n$ determines iterations
  
- **Worst Case**: $O(\log n)$
  - Processing the maximum value still only requires $\log_2(n)$ divisions

**Derivation**: 
- After each iteration, $num$ is halved: $num_{i+1} = \lfloor num_i / 2 \rfloor$
- Starting from $n$, we reach 0 after $k$ iterations where $n / 2^k < 1$
- Solving: $k > \log_2(n)$
- Therefore, $T(n) = O(\log n)$

**Bit Length Relationship**: For a number requiring $b$ bits, $b = \lfloor \log_2(n) \rfloor + 1$, so $T(n) = O(b)$ where $b$ is the output length.

### 4.2 Space Complexity

- **Auxiliary Space (during computation)**: $O(\log n)$
  - The binary string grows to length $\lceil \log_2(n+1) \rceil$
  - String building operations may use additional temporary space
  
- **Output Space**: $O(\log n)$
  - The result string has length $\lfloor \log_2(n) \rfloor + 1$

- **Total Space**: $O(\log n)$
  - Dominated by the output string size

**Note**: Some implementations may use $O(\log^2 n)$ space if string concatenation creates intermediate copies, but efficient implementations (like Rust's `String`) use amortized $O(\log n)$.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Function Signature**:
```rust
pub fn decimal_to_binary(base_num: u64) -> String
```
- Takes `u64` by value (cheap to copy, 8 bytes)
- Returns owned `String` (caller receives ownership)
- Input range: $[0, 2^{64} - 1]$ or $[0, 18,446,744,073,709,551,615]$

**Rust Idioms**:
```rust
let mut binary_num = String::new();
loop {
    let bit = (num % 2).to_string();
    binary_num.push_str(&bit);
    num /= 2;
    if num == 0 {
        break;
    }
}
binary_num.chars().rev().collect()
```

**Optimization Opportunities**:
1. **Use `push` instead of `push_str`**:
   ```rust
   binary_num.push(if num % 2 == 0 { '0' } else { '1' });
   ```
   Avoids string-to-string conversion overhead

2. **Use bit manipulation**:
   ```rust
   binary_num.push(char::from(b'0' + (num & 1) as u8));
   num >>= 1;  // Right shift instead of division
   ```

3. **Pre-allocate capacity**:
   ```rust
   let mut binary_num = String::with_capacity(64);
   ```

**Iterator Usage**:
- `.chars().rev().collect()`: Efficiently reverses the string
- Returns a new `String` without modifying the original

### 5.2 Edge Cases

1. **Zero**: "0" (special case, single bit)
2. **One**: "1" (smallest non-zero)
3. **Powers of Two**: 
   - $2^0 = 1$ → "1"
   - $2^1 = 2$ → "10"
   - $2^2 = 4$ → "100"
   - $2^{10} = 1024$ → "10000000000"
4. **Maximum u64**: $2^{64} - 1$ → 64 ones: "1111...1111"
5. **All Patterns**:
   - Sequential numbers produce various bit patterns
   - $2^n - 1$ produces $n$ ones
   - $2^n$ produces "1" followed by $n$ zeros

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Low-Level Programming**:
   - Assembly language instruction encoding
   - Machine code generation in compilers
   - Firmware development for embedded systems

2. **Network Programming**:
   - Creating binary protocol messages
   - IPv4 address representation and manipulation
   - Network packet construction

3. **Bitwise Operations Preparation**:
   - Visualizing bit patterns before masking
   - Debugging bitwise algorithms
   - Understanding flag combinations

4. **Digital Logic Design**:
   - Truth table generation
   - Logic circuit simulation
   - Hardware description languages (VHDL/Verilog)

5. **Data Encoding**:
   - QR code generation
   - Barcode encoding
   - Error-correcting codes

6. **Educational Tools**:
   - Teaching binary arithmetic
   - Computer architecture simulators
   - Number system converters

7. **Cryptography**:
   - Key generation visualization
   - Bit-level encryption preparation
   - Hash function implementation

### 6.2 Related Algorithms

**Inverse Operations**:
- **Binary to Decimal**: Direct inverse (weighted sum of powers of 2)
- **Time Complexity Comparison**: Binary→Decimal is also $O(\log n)$

**Base Conversion Variants**:
- **Decimal to Octal**: Similar algorithm with modulo/division by 8
- **Decimal to Hexadecimal**: Modulo/division by 16 with A-F mapping
- **Decimal to Arbitrary Base**: Generalized version for any base $b$

**Optimization Techniques**:
| Method | Time | Space | Pros | Cons |
|--------|------|-------|------|------|
| Repeated Division | $O(\log n)$ | $O(\log n)$ | Simple, clear | Moderate performance |
| Bit Manipulation | $O(\log n)$ | $O(\log n)$ | Faster | Less readable |
| Lookup Table | $O(1)$ | $O(2^k)$ | Fastest for small $k$ | Only for bounded input |

**When to Use Each**:
- **Repeated Division**: Educational purposes, clarity preferred
- **Bit Manipulation**: Performance-critical code
- **Built-in Functions**: Production code (`format!("{:b}", n)` in Rust)
- **Lookup Tables**: Small, fixed-size inputs (e.g., 8-bit values)

**Related Problems**:
- **Gray Code Generation**: Similar bit manipulation
- **Hamming Weight**: Count of '1' bits
- **Bit Reversal**: Used in FFT algorithms

## 7. References

1. **Knuth, Donald E.** (1997). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (3rd ed.). Addison-Wesley. Section 4.1: Positional Number Systems.

2. **Warren, Henry S.** (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley. Chapter 2: Basics - includes optimization techniques.

3. **Wikipedia**: [Binary Number](https://en.wikipedia.org/wiki/Binary_number)

4. **Wikipedia**: [Radix/Base Conversion](https://en.wikipedia.org/wiki/Radix)

5. **Rust Documentation**: [String API](https://doc.rust-lang.org/std/string/struct.String.html)

6. **Rust Documentation**: [format! macro](https://doc.rust-lang.org/std/macro.format.html) - includes binary formatting

7. **ISO/IEC 80000-13:2008**: International standard for quantities and units in information science

8. **Sedgewick, Robert & Wayne, Kevin** (2011). *Algorithms* (4th ed.). Addison-Wesley. Section on elementary algorithms.
