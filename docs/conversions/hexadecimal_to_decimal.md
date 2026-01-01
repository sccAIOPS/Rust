# Hexadecimal to Decimal Conversion

## 1. Overview

Hexadecimal to Decimal conversion translates numbers from base-16 (hexadecimal) representation to base-10 (decimal) representation. Hexadecimal is widely used in computing due to its compact representation of binary data - each hex digit represents exactly 4 bits.

**Historical Context**: The hexadecimal system gained prominence in the 1960s with the advent of byte-oriented computing architectures. IBM's System/360, introduced in 1964, popularized hexadecimal notation for memory addresses and machine code. Today, it's ubiquitous in programming, particularly for memory addresses, color codes, and low-level data representation.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a hexadecimal string $H = h_{n-1}h_{n-2}...h_1h_0$ where each $h_i \in \{0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F\}$, convert it to its decimal equivalent $D$.

**Formal Definition**:
$$D = \sum_{i=0}^{n-1} v(h_i) \times 16^i$$

where $v(h_i)$ is the decimal value of hex digit $h_i$:
- $v(h_i) = h_i$ for $h_i \in \{0..9\}$
- $v(h_i) = 10 + (h_i - \text{'A'})$ for $h_i \in \{A..F\}$

### 2.2 Mathematical Model

**Input Specifications**:
- A string containing characters from: {'0'-'9', 'A'-'F', 'a'-'f'}
- Case-insensitive (both uppercase and lowercase accepted)
- No prefix (e.g., "0x") - pure hex digits
- Maximum length bounded by target integer type (16 hex digits for `u64`)

**Output Specifications**:
- A non-negative integer in base-10
- `Result<u64, &'static str>` in Rust (error on invalid input or overflow)

**Key Properties**:
1. **Positional Value**: Each position represents a power of 16
2. **Digit Mapping**: 0-9 map directly, A-F (or a-f) map to 10-15
3. **Compactness**: One hex digit = 4 binary bits = 2^4 values
4. **Range**: For $n$ hex digits, range is $[0, 16^n - 1]$

### 2.3 Correctness Proof

**Invariant**: After processing $k$ hex digits from right to left, the accumulated value equals the decimal representation of those $k$ digits.

**Termination**: The algorithm terminates after processing all $n$ digits.

**Correctness Theorem**: 
For a valid hexadecimal string of length $n$, the algorithm produces:
$$\text{result} = \sum_{i=0}^{n-1} v(h_i) \times 16^i$$

This follows from the definition of positional notation in base 16.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm leverages Rust's built-in `from_str_radix` function, which implements the positional notation formula efficiently. The manual approach would process digits from right to left (or left to right with running multiplication), converting each hex digit to its decimal value and accumulating powers of 16.

**Conversion Table**:
| Hex | Decimal | Binary |
|-----|---------|--------|
| 0   | 0       | 0000   |
| 1   | 1       | 0001   |
| 9   | 9       | 1001   |
| A/a | 10      | 1010   |
| B/b | 11      | 1011   |
| F/f | 15      | 1111   |

**Example**: "2B3" (hex) to decimal
- 3 × 16⁰ = 3 × 1 = 3
- B × 16¹ = 11 × 16 = 176
- 2 × 16² = 2 × 256 = 512
- Sum: 512 + 176 + 3 = 691

### 3.2 Pseudocode

```
FUNCTION hexadecimal_to_decimal(hex_string):
    // Input validation
    IF hex_string is empty THEN
        RETURN Error("Empty input")
    END IF
    
    // Validate all characters are valid hex digits
    FOR each char in hex_string DO
        IF char NOT IN {'0'..'9', 'A'..'F', 'a'..'f'} THEN
            RETURN Error("Input was not a hexadecimal number")
        END IF
    END FOR
    
    // Conversion using radix 16
    TRY
        result ← parse_integer(hex_string, radix=16)
        RETURN Success(result)
    CATCH overflow_error
        RETURN Error("Failed to convert hexadecimal to decimal")
    END TRY
END FUNCTION
```

**Line-by-line annotations**:
- Lines 2-4: Reject empty input
- Lines 6-11: Validate each character is a valid hex digit (0-9, A-F, case-insensitive)
- Lines 13-18: Use built-in radix conversion with error handling for overflow

**Alternative Manual Implementation**:
```
FUNCTION hex_to_decimal_manual(hex_string):
    result ← 0
    power ← 1  // Represents 16^i
    
    FOR each digit in reverse(hex_string) DO
        IF digit IN '0'..'9' THEN
            value ← digit - '0'
        ELSE IF digit IN 'A'..'F' THEN
            value ← digit - 'A' + 10
        ELSE IF digit IN 'a'..'f' THEN
            value ← digit - 'a' + 10
        ELSE
            RETURN Error("Invalid character")
        END IF
        
        result ← result + value × power
        power ← power × 16
    END FOR
    
    RETURN result
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**: "1267A" (hexadecimal)

**Method 1: Right to Left**
| Step | Digit | Value | Position | Power (16^i) | Contribution | Running Sum |
|------|-------|-------|----------|--------------|--------------|-------------|
| 1    | A     | 10    | 0        | 1            | 10 × 1 = 10  | 10          |
| 2    | 7     | 7     | 1        | 16           | 7 × 16 = 112 | 122         |
| 3    | 6     | 6     | 2        | 256          | 6 × 256 = 1536 | 1658      |
| 4    | 2     | 2     | 3        | 4096         | 2 × 4096 = 8192 | 9850     |
| 5    | 1     | 1     | 4        | 65536        | 1 × 65536 = 65536 | 75386 |

**Result**: 75,386 (decimal)

**Verification**: 
- $1×16^4 + 2×16^3 + 6×16^2 + 7×16^1 + 10×16^0$
- $= 65536 + 8192 + 1536 + 112 + 10 = 75386$ ✓

**Method 2: Left to Right (Horner's Method)**
```
result = 0
result = result × 16 + 1 = 1
result = result × 16 + 2 = 18
result = result × 16 + 6 = 294
result = result × 16 + 7 = 4711
result = result × 16 + 10 = 75386
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Best Case**: $O(n)$ where $n$ is the length of the hex string
  - Must validate every character
  
- **Average Case**: $O(n)$
  - Character validation: $O(n)$
  - Radix conversion: $O(n)$
  
- **Worst Case**: $O(n)$
  - All characters must be processed

**Derivation**: 
- Input validation loop: $n$ iterations
- `from_str_radix` implementation: $n$ digit conversions
- Each digit operation: $O(1)$
- Total: $T(n) = O(n) + O(n) = O(n)$

### 4.2 Space Complexity

- **Auxiliary Space**: $O(1)$
  - Fixed number of variables (result accumulator, power multiplier)
  - No data structures growing with input size
  
- **Total Space**: $O(n)$
  - Input string: $O(n)$
  - Algorithm workspace: $O(1)$
  - Total: $O(n)$

**Note**: The `from_str_radix` implementation in Rust's standard library is highly optimized and uses constant extra space.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Function Signature**:
```rust
pub fn hexadecimal_to_decimal(hexadecimal_str: &str) -> Result<u64, &'static str>
```

**Type Choices**:
- Input: `&str` (borrowed string slice) - no ownership transfer
- Output: `Result<u64, &'static str>` - explicit error handling
- Return type alternatives:
  - `Ok(u64)`: Successful conversion
  - `Err(&'static str)`: Error with descriptive message

**Error Handling Strategy**:
```rust
match u64::from_str_radix(hexadecimal_str, 16) {
    Ok(decimal) => Ok(decimal),
    Err(_e) => Err("Failed to convert octal to hexadecimal"),  // Note: error message has bug
}
```

**Input Validation**:
```rust
for hexadecimal_str in hexadecimal_str.chars() {
    if !hexadecimal_str.is_ascii_hexdigit() {
        return Err("Input was not a hexadecimal number");
    }
}
```

**Standard Library Usage**:
- `char::is_ascii_hexdigit()`: Efficient ASCII hex validation
- `u64::from_str_radix(s, 16)`: Standard radix conversion
- Built-in overflow detection

**Performance Considerations**:
1. **Early validation**: Catches errors before expensive conversion
2. **Zero-copy**: `&str` avoids unnecessary string copies
3. **Optimized parsing**: `from_str_radix` is highly optimized in stdlib

### 5.2 Edge Cases

1. **Empty String**: Error "Empty input"
2. **Single Digit**: 
   - "0" → 0
   - "F" → 15
3. **Leading Zeros**: "00FF" → 255 (valid, leading zeros ignored)
4. **Case Insensitivity**: 
   - "abc" → 2748
   - "ABC" → 2748
   - "AbC" → 2748
5. **Invalid Characters**:
   - "0xABC" → Error (prefix not allowed)
   - "12G4" → Error ('G' not hex)
   - "12 34" → Error (space not allowed)
6. **Maximum u64**: 
   - "FFFFFFFFFFFFFFFF" → 18,446,744,073,709,551,615
   - 16 hex digits maximum
7. **Overflow**: 
   - "10000000000000000" (17 digits) → Error
8. **Boundary Values**:
   - "7FFFFFFF" → 2,147,483,647 (max i32)
   - "80000000" → 2,147,483,648 (i32 overflow point)

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Memory Address Interpretation**:
   - Debugger address display
   - Memory dump analysis
   - Pointer arithmetic visualization
   - Example: "0x7FFF5FBF" → stack address

2. **Color Code Processing**:
   - Web development (CSS hex colors)
   - Image processing
   - Graphics programming
   - Example: "#FF5733" → RGB(255, 87, 51)

3. **Network Programming**:
   - MAC address parsing ("00:1A:2B:3C:4D:5E")
   - IPv6 address handling
   - Protocol hex dumps interpretation

4. **Cryptography**:
   - Hash value display and processing
   - Key representation (RSA, AES keys in hex)
   - Digital signature verification
   - Example: SHA-256 hash interpretation

5. **File Format Parsing**:
   - Binary file magic numbers
   - Executable file headers (ELF, PE)
   - Firmware binary analysis
   - Example: "7F454C46" → ELF magic number

6. **Embedded Systems**:
   - Hardware register configuration
   - Sensor data interpretation
   - Bootloader communication
   - Flash memory programming

7. **Data Serialization**:
   - UUID parsing ("550e8400-e29b-41d4-a716-446655440000")
   - Binary data in JSON/XML
   - Database BLOB representation

### 6.2 Related Algorithms

**Direct Conversions**:
| Conversion | Relationship | Efficiency |
|------------|--------------|------------|
| Hex → Binary | 1 hex = 4 bits | O(n), simple mapping |
| Hex → Octal | Group by 3 bits | O(n), via binary |
| Hex → Decimal | Positional notation | O(n), this algorithm |
| Decimal → Hex | Repeated division by 16 | O(log n) |

**Conversion Path Efficiency**:
```
Hex → Binary: Direct (4-bit chunks)
Hex → Decimal: Direct (positional sum)
Hex → Octal: Hex → Binary → Octal (3-bit chunks)
```

**Related Utilities**:
1. **Color Space Conversions**: Hex → RGB → HSL/HSV
2. **UUID Parsing**: Hex string → 128-bit value
3. **Checksum Verification**: Hex checksum → numeric comparison
4. **Base64 Encoding**: Often combined with hex for data transmission

**When to Use Each**:
- **Hex → Decimal**: When displaying to users, arithmetic operations
- **Hex → Binary**: Bit-level analysis, logic operations
- **Hex → ASCII**: Protocol message interpretation
- **Keep as Hex**: Memory addresses, color codes (human-readable)

**Performance Comparison**:
| Method | Time | Space | Use Case |
|--------|------|-------|----------|
| Standard Library | O(n) | O(1) | Production code |
| Manual Loop | O(n) | O(1) | Educational |
| Lookup Table | O(n) | O(16) | When caching digit values |
| Horner's Method | O(n) | O(1) | Educational/optimization |

## 7. References

1. **Knuth, Donald E.** (1997). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (3rd ed.). Addison-Wesley. Section 4.1: Positional Number Systems.

2. **Wikipedia**: [Hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal)

3. **RFC 3986**: Uniform Resource Identifier (URI): Generic Syntax - defines hex encoding in URLs

4. **W3C CSS Color Module**: Defines hex color notation standard

5. **Intel® 64 and IA-32 Architectures Software Developer Manuals**: Use of hexadecimal in assembly programming

6. **Rust Documentation**: [u64::from_str_radix](https://doc.rust-lang.org/std/primitive.u64.html#method.from_str_radix)

7. **RFC 4122**: A Universally Unique IDentifier (UUID) URN Namespace - hex representation

8. **ISO/IEC 9899**: C Programming Language Standard - hexadecimal notation conventions

9. **Warren, Henry S.** (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley. Chapter on base conversions.
