# Binary to Decimal Conversion

## 1. Overview

Binary to Decimal conversion is a fundamental operation in computer science that translates numbers from base-2 (binary) representation to base-10 (decimal) representation. This conversion is essential for human-readable interpretation of binary data used internally by computers.

**Historical Context**: Binary numeral system has been used since ancient times, but its modern application in computing was popularized by Gottfried Wilhelm Leibniz in the 17th century. The conversion between binary and decimal became crucial with the advent of digital computers in the 1940s.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a binary string $b = b_{n-1}b_{n-2}...b_1b_0$ where each $b_i \in \{0, 1\}$, convert it to its decimal equivalent $D$.

**Formal Definition**:
$$D = \sum_{i=0}^{n-1} b_i \times 2^i$$

where $b_i$ is the bit at position $i$ (0-indexed from right to left).

### 2.2 Mathematical Model

**Input Specifications**:
- A string containing only characters '0' and '1'
- Maximum length typically bounded by the target integer type (e.g., 128 bits for `u128`)

**Output Specifications**:
- A non-negative integer in base-10
- `None` or error if overflow occurs or invalid input

**Key Properties**:
1. **Position Value**: Each bit position represents a power of 2
2. **Weight**: Rightmost bit (LSB) has weight $2^0 = 1$, leftmost bit has weight $2^{n-1}$
3. **Range**: For $n$ bits, the range is $[0, 2^n - 1]$

### 2.3 Correctness Proof

**Invariant**: After processing $k$ bits from right to left, the accumulated value equals the decimal representation of those $k$ bits.

**Termination**: The algorithm terminates after processing all $n$ bits in the input string.

**Correctness Theorem**: 
For a valid binary string of length $n$, the algorithm produces:
$$\text{result} = \sum_{i=0}^{n-1} b_i \times 2^i$$

This follows directly from the loop invariant and the definition of binary representation.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm processes the binary string from right to left (least significant to most significant bit). For each '1' bit encountered, it adds the corresponding power of 2 to the result. The power of 2 doubles with each position as we move left.

**Example**: For binary "1011"
- Position 0 (rightmost): 1 × 2⁰ = 1
- Position 1: 1 × 2¹ = 2
- Position 2: 0 × 2² = 0
- Position 3: 1 × 2³ = 8
- Sum: 1 + 2 + 0 + 8 = 11

### 3.2 Pseudocode

```
FUNCTION binary_to_decimal(binary_string):
    IF length(binary_string) > MAX_BITS THEN
        RETURN None
    END IF
    
    result ← 0
    position_value ← 1  // Represents 2^i
    
    FOR each bit in reverse(binary_string) DO
        IF bit = '1' THEN
            IF result + position_value would overflow THEN
                RETURN None
            END IF
            result ← result + position_value
        ELSE IF bit ≠ '0' THEN
            RETURN None  // Invalid character
        END IF
        
        position_value ← position_value × 2  // Left shift
    END FOR
    
    RETURN result
END FUNCTION
```

**Line-by-line annotations**:
- Lines 1-3: Length validation prevents overflow
- Line 5: Initialize accumulator
- Line 6: Initialize position value (2⁰)
- Line 8: Process bits from LSB to MSB
- Lines 9-13: Add position value if bit is '1', with overflow check
- Lines 14-15: Validate input (only '0' and '1' allowed)
- Line 18: Double the position value for next bit (equivalent to left shift)

### 3.3 Step-by-Step Example

**Input**: "1101" (binary)

| Step | Bit | Position | Position Value (2^i) | Current Sum | Action |
|------|-----|----------|---------------------|-------------|---------|
| Init | -   | -        | 1                   | 0           | Initialize |
| 1    | '1' | 0        | 1                   | 0 + 1 = 1   | Add 1 |
| 2    | '0' | 1        | 2                   | 1           | Skip |
| 3    | '1' | 2        | 4                   | 1 + 4 = 5   | Add 4 |
| 4    | '1' | 3        | 8                   | 5 + 8 = 13  | Add 8 |

**Result**: 13 (decimal)

**Verification**: 
- $1 \times 2^3 + 1 \times 2^2 + 0 \times 2^1 + 1 \times 2^0$
- $= 8 + 4 + 0 + 1 = 13$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Best Case**: $O(n)$ where $n$ is the length of the binary string
  - Must process every character to validate and convert
  
- **Average Case**: $O(n)$
  - Linear scan through the string is always required
  
- **Worst Case**: $O(n)$
  - Even with all '1' bits, we still process each character once

**Derivation**: Each character in the input string is visited exactly once in the loop. Each iteration performs constant-time operations (comparison, addition, shift). Therefore, $T(n) = n \times O(1) = O(n)$.

### 4.2 Space Complexity

- **Auxiliary Space**: $O(1)$
  - Only a fixed number of variables used (result, position_value, loop counter)
  - No additional data structures that grow with input size

- **Total Space**: $O(n)$
  - Input string storage: $O(n)$
  - Algorithm variables: $O(1)$
  - Total: $O(n) + O(1) = O(n)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Ownership and Borrowing**:
```rust
pub fn binary_to_decimal(binary: &str) -> Option<u128>
```
- Takes `&str` (borrowed string slice) - no ownership transfer needed
- Returns `Option<u128>` for safe error handling (None on overflow/invalid input)

**Type Constraints**:
- Uses `u128` to support up to 128-bit binary numbers
- `num_traits::CheckedAdd` trait for overflow-safe arithmetic
- Maximum input length: 128 characters

**Safety Features**:
- `checked_add()`: Returns `None` instead of panicking on overflow
- Left shift operator `<<=`: Efficiently multiplies by 2
- Pattern matching on characters for validation

**Rust Idioms**:
```rust
for bit in binary.chars().rev() {
    match bit {
        '1' => { /* add position value */ }
        '0' => { /* skip */ }
        _ => return None,  // Invalid character
    }
}
```

### 5.2 Edge Cases

1. **Empty String**: Should return error or None
2. **Leading Zeros**: "0000110" → valid, equals 6
3. **All Zeros**: "0000" → valid, equals 0
4. **All Ones**: Maximum value for given bit length
5. **Invalid Characters**: "102", "1a1" → should return None
6. **Overflow**: String length > 128 bits → return None
7. **Maximum 128-bit Value**: 
   - Input: 128 ones
   - Output: 340,282,366,920,938,463,463,374,607,431,768,211,455
8. **129+ bits**: Should return None (overflow)

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Computer Architecture**:
   - Assembly language programming (binary opcodes to human-readable format)
   - CPU instruction decoding
   - Register value interpretation

2. **Network Programming**:
   - IP address manipulation (binary to dotted decimal)
   - Subnet mask calculations
   - Protocol header parsing

3. **Embedded Systems**:
   - Reading sensor data from hardware registers
   - GPIO pin state interpretation
   - Bit-field extraction from control registers

4. **File Format Parsing**:
   - Binary file readers
   - Image format decoders (reading pixel data)
   - Audio/video codec implementations

5. **Cryptography**:
   - Key representation conversion
   - Hash value formatting
   - Binary-to-text encoding schemes

6. **Data Compression**:
   - Huffman coding decoders
   - Bit stream readers
   - Variable-length code interpretation

### 6.2 Related Algorithms

**Direct Variants**:
- **Decimal to Binary**: Inverse operation (repeated division by 2)
- **Binary to Octal**: Group bits by 3 (since 2³ = 8)
- **Binary to Hexadecimal**: Group bits by 4 (since 2⁴ = 16)

**Related Conversions**:
- **Arbitrary Base Conversion**: Generalized version for any base
- **Gray Code to Binary**: Special binary encoding conversion
- **Two's Complement**: Signed binary representation

**When to Use**:
- **Binary to Decimal**: When displaying binary data to users
- **Decimal to Binary**: When preparing data for bitwise operations
- **Direct Hex/Octal**: When working with larger groups of bits efficiently

**Performance Comparison**:
| Conversion | Time | Space | Use Case |
|------------|------|-------|----------|
| Binary→Decimal | O(n) | O(1) | Human display |
| Decimal→Binary | O(log n) | O(log n) | Bitwise prep |
| Binary→Hex | O(n) | O(n) | Compact display |

## 7. References

1. **Knuth, Donald E.** (1997). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (3rd ed.). Addison-Wesley. Section 4.1: Positional Number Systems.

2. **Warren, Henry S.** (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley. Chapter 2: Basics.

3. **Wikipedia**: [Binary Number](https://en.wikipedia.org/wiki/Binary_number)

4. **Wikipedia**: [Positional Notation](https://en.wikipedia.org/wiki/Positional_notation)

5. **Rust Documentation**: [Checked Arithmetic](https://doc.rust-lang.org/std/primitive.u128.html#method.checked_add)

6. **IEEE Standard 754**: For understanding floating-point binary representation

7. **NIST Digital Library of Mathematical Functions**: For numerical computation references
