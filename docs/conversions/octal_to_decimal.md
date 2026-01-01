# Octal to Decimal Conversion

## 1. Overview

Octal to Decimal conversion transforms numbers from base-8 (octal) representation to base-10 (decimal) representation. While less common than hexadecimal today, octal notation has historical significance in computing and remains useful in specific contexts like Unix file permissions.

**Historical Context**: Octal was widely used in early computing, particularly in systems like the PDP-8 and PDP-11 minicomputers (1960s-1970s). It naturally fit 12-bit, 24-bit, and 36-bit word sizes (divisible by 3). Unix file permissions (rwxrwxrwx = 777₈) preserve this legacy.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an octal string $O = o_{n-1}o_{n-2}...o_1o_0$ where each $o_i \in \{0,1,2,3,4,5,6,7\}$, convert it to its decimal equivalent $D$.

**Formal Definition**:
$$D = \sum_{i=0}^{n-1} o_i \times 8^i$$

where each $o_i$ represents a digit value from 0 to 7.

### 2.2 Mathematical Model

**Input Specifications**:
- String containing only digits '0' through '7'
- Whitespace allowed (trimmed during processing)
- Maximum length bounded by target type (21 octal digits for `u64`)

**Output Specifications**:
- `Result<u64, &'static str>`
- Success: Non-negative decimal integer
- Error cases: Empty, invalid digit (8 or 9), overflow

**Key Properties**:
1. **Position Value**: Each position represents power of 8
2. **Digit Range**: Only 0-7 valid (8 and 9 are invalid)
3. **Binary Relationship**: 1 octal digit = 3 binary bits
4. **Range**: For $n$ octal digits, range is $[0, 8^n - 1]$

### 2.3 Correctness Proof

**Invariant**: After processing $k$ digits, accumulated value equals decimal representation of those $k$ octal digits.

**Termination**: Algorithm terminates after processing all validated digits.

**Correctness**: Using Rust's `from_str_radix` with base 8 correctly implements:
$$\sum_{i=0}^{n-1} o_i \times 8^i$$

## 3. Algorithm Description

### 3.1 Intuition

Convert octal to decimal using positional notation where each digit's value is multiplied by the appropriate power of 8, based on its position.

**Conversion Table**:
| Octal | Decimal | Binary |
|-------|---------|--------|
| 0     | 0       | 000    |
| 1     | 1       | 001    |
| 7     | 7       | 111    |

**Example**: "123" (octal)
- 3 × 8⁰ = 3 × 1 = 3
- 2 × 8¹ = 2 × 8 = 16
- 1 × 8² = 1 × 64 = 64
- Sum: 64 + 16 + 3 = 83

### 3.2 Pseudocode

```
FUNCTION octal_to_decimal(octal_string):
    trimmed ← TRIM(octal_string)
    
    IF trimmed is empty THEN
        RETURN Error("Empty")
    END IF
    
    FOR each char in trimmed DO
        IF char NOT IN {'0'..'7'} THEN
            RETURN Error("Non-octal Value")
        END IF
    END FOR
    
    TRY
        result ← parse_with_radix(trimmed, 8)
        RETURN Success(result)
    CATCH overflow
        RETURN Error("Conversion error")
    END TRY
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**: "12345" (octal)

| Digit | Position | Value | Power (8^i) | Contribution |
|-------|----------|-------|-------------|--------------|
| 5     | 0        | 5     | 1           | 5            |
| 4     | 1        | 4     | 8           | 32           |
| 3     | 2        | 3     | 64          | 192          |
| 2     | 3        | 2     | 512         | 1024         |
| 1     | 4        | 1     | 4096        | 4096         |

**Sum**: 4096 + 1024 + 192 + 32 + 5 = **5349**

## 4. Complexity Analysis

### 4.1 Time Complexity

- **All Cases**: $O(n)$ where $n$ is string length
- Validation loop: $O(n)$
- Radix conversion: $O(n)$

### 4.2 Space Complexity

- **Auxiliary Space**: $O(1)$
- **Total Space**: $O(n)$ for input string

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn octal_to_decimal(octal_str: &str) -> Result<u64, &'static str>
```

**Key Features**:
- Input trimming: `octal_str.trim()`
- Validation: `chars().all(|c| ('0'..='7').contains(&c))`
- Conversion: `u64::from_str_radix(octal_str, 8)`
- Error types: Empty, Non-octal, Conversion error

### 5.2 Edge Cases

1. **Empty/Whitespace**: Error
2. **Invalid digits**: "89", "18" → Error
3. **Leading zeros**: "0123" → 83
4. **Max u64**: "1777777777777777777777" (21 digits)

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Unix File Permissions**: chmod 755, 644
2. **Legacy System Maintenance**: PDP-11, VAX systems
3. **Assembly Programming**: Some instruction encodings
4. **Escape Sequences**: C/C++ octal literals (\077)
5. **Network Protocols**: Historical protocols using octal

### 6.2 Related Algorithms

- **Octal to Binary**: Group 3 bits at a time
- **Binary to Octal**: Reverse grouping
- **Octal to Hexadecimal**: Via binary intermediate

## 7. References

1. **Knuth, Donald E.** (1997). *The Art of Computer Programming, Vol 2*
2. **Wikipedia**: [Octal](https://en.wikipedia.org/wiki/Octal)
3. **Unix chmod man page**: File permission octal notation
4. **DEC PDP-11 Architecture Handbook**: Historical octal usage
