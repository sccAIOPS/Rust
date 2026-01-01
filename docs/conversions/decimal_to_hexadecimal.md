# Decimal to Hexadecimal Conversion

## 1. Overview

Decimal to Hexadecimal conversion transforms numbers from base-10 (decimal) representation to base-16 (hexadecimal) representation. Hexadecimal is extensively used in computing as a compact, human-readable way to represent binary data - each hex digit corresponds to exactly 4 bits.

**Historical Context**: While hexadecimal notation existed in earlier mathematics, it became fundamental to computing in the 1960s. The IBM System/360 mainframe (1964) standardized hexadecimal for representing memory addresses and machine instructions. Today, hex notation is ubiquitous in programming, from memory addresses to color codes (#FF5733) to cryptographic hashes.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a non-negative decimal integer $D$, convert it to its hexadecimal representation $H = h_{n-1}h_{n-2}...h_1h_0$ where each $h_i \in \{0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F\}$.

**Formal Definition**:

Find the unique sequence of hex digits $h_i$ such that:
$$D = \sum_{i=0}^{n-1} v(h_i) \times 16^i$$

where:
- $v(h_i) = h_i$ for $h_i \in \{0..9\}$
- $v(h_i) = h_i - 55$ for $h_i \in \{A..F\}$ (ASCII 'A' = 65, value = 10)

### 2.2 Mathematical Model

**Input Specifications**:
- A non-negative integer $D \geq 0$ in decimal format
- Typically bounded by machine word size (e.g., `u64` supports $0 \leq D \leq 2^{64} - 1$)

**Output Specifications**:
- A string containing characters from {'0'-'9', 'A'-'F'}
- No leading zeros (except for 0 itself)
- Uppercase letters by convention
- Length: $n = \lfloor \log_{16}(D) \rfloor + 1$ for $D > 0$, or $n = 1$ for $D = 0$

**Key Properties**:
1. **Uniqueness**: Every decimal number has exactly one hex representation (without leading zeros)
2. **Digit Count**: A number $D$ requires $\lceil \log_{16}(D+1) \rceil$ hex digits
3. **Modulo-16 Property**: The least significant digit is $D \bmod 16$
4. **Division Property**: Dividing by 16 shifts hex digits right by one position
5. **Compactness**: Hex uses 4× fewer digits than binary for the same value

### 2.3 Correctness Proof

**Algorithm Invariant**: After $k$ iterations, the last $k$ hex digits of the result represent the hex form of the last $k$ digits of the original number.

**Loop Invariant**:
- Let $D_k$ be the value after $k$ divisions by 16
- Let $H_k$ be the first $k$ hex digits collected (in reverse order)
- Then: $D = D_k \times 16^k + H_k$

**Termination**: The algorithm terminates when $D_k = 0$, which occurs after $\lfloor \log_{16}(D) \rfloor + 1$ iterations.

**Correctness**: Upon termination, all hex digits have been collected, and the result correctly represents $D$ in base 16.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm uses repeated division by 16 (the target base). At each step:
1. The remainder ($D \bmod 16$) gives the next hex digit (0-9 or A-F)
2. The quotient ($\lfloor D / 16 \rfloor$) becomes the new number to process

The remainders are collected from least significant to most significant digit, then the string is built in reverse (or digits are prepended).

**Digit Mapping**:
- Remainder 0-9 → '0'-'9' (ASCII: add 48 or add '0')
- Remainder 10-15 → 'A'-'F' (ASCII: subtract 10, add 65 or add 'A')

**Example**: Converting 255 to hex
- 255 ÷ 16 = 15 remainder **15** ('F')
- 15 ÷ 16 = 0 remainder **15** ('F')
- Result: "FF"

### 3.2 Pseudocode

```
FUNCTION decimal_to_hexadecimal(decimal_number):
    IF decimal_number = 0 THEN
        RETURN "0"
    END IF
    
    hex_string ← empty_string
    num ← decimal_number
    
    WHILE num > 0 DO
        remainder ← num MOD 16
        
        IF remainder < 10 THEN
            hex_char ← remainder + '0'    // '0' to '9'
        ELSE
            hex_char ← (remainder - 10) + 'A'  // 'A' to 'F'
        END IF
        
        PREPEND hex_char to hex_string
        num ← num DIV 16
    END WHILE
    
    RETURN hex_string
END FUNCTION
```

**Line-by-line annotations**:
- Lines 2-4: Handle special case of zero
- Line 6: Initialize result string
- Line 7: Working copy of input
- Line 9: Loop while digits remain
- Line 10: Extract least significant hex digit
- Lines 12-16: Map remainder to hex character
  - 0-9 → '0'-'9': Add ASCII value of '0' (48)
  - 10-15 → 'A'-'F': Subtract 10, add ASCII value of 'A' (65)
- Line 18: Prepend digit to result (building string in correct order)
- Line 19: Remove processed digit

### 3.3 Step-by-Step Example

**Input**: 123456 (decimal)

| Iteration | num    | num mod 16 | Hex Digit | num div 16 | Hex String |
|-----------|--------|------------|-----------|------------|------------|
| Init      | 123456 | -          | -         | -          | ""         |
| 1         | 123456 | 0          | '0'       | 7716       | "0"        |
| 2         | 7716   | 4          | '4'       | 482        | "40"       |
| 3         | 482    | 2          | '2'       | 30         | "240"      |
| 4         | 30     | 14         | 'E'       | 1          | "E240"     |
| 5         | 1      | 1          | '1'       | 0          | "1E240"    |

**Result**: "1E240"

**Verification**: 
- $1×16^4 + 14×16^3 + 2×16^2 + 4×16^1 + 0×16^0$
- $= 65536 + 57344 + 512 + 64 + 0 = 123456$ ✓

**Alternative Computation (powers of 16)**:
```
123456 = 1×65536 + 14×4096 + 2×256 + 4×16 + 0×1
       = 1×16⁴ + E×16³ + 2×16² + 4×16¹ + 0×16⁰
       = 1E240₁₆
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Best Case**: $O(\log_{16} n)$ where $n$ is the input decimal number
  - Minimum iterations needed to represent any non-zero number
  
- **Average Case**: $O(\log_{16} n)$ ≈ $O(\log n / 4)$ in binary terms
  - Each iteration reduces the number by a factor of 16
  
- **Worst Case**: $O(\log_{16} n)$
  - Maximum value requires most iterations

**Derivation**: 
- After each iteration, $num_{i+1} = \lfloor num_i / 16 \rfloor$
- Starting from $n$, we reach 0 after $k$ iterations where $n / 16^k < 1$
- Solving: $k > \log_{16}(n)$
- Therefore, $T(n) = O(\log_{16} n)$

**Relationship to Output Length**: 
- Output has $d = \lfloor \log_{16}(n) \rfloor + 1$ digits
- Time complexity: $T(n) = O(d)$ where $d$ is the number of hex digits

**Comparison to Binary Conversion**:
- Binary: $O(\log_2 n)$ iterations (many)
- Hexadecimal: $O(\log_{16} n) = O(\log_2 n / 4)$ iterations (4× fewer)

### 4.2 Space Complexity

- **Auxiliary Space (during computation)**: $O(\log_{16} n)$
  - String grows to length $\lceil \log_{16}(n+1) \rceil$
  - String operations may require temporary buffers
  
- **Output Space**: $O(\log_{16} n)$
  - Result string length is $\lfloor \log_{16}(n) \rfloor + 1$

- **Total Space**: $O(\log_{16} n)$
  - Dominated by the output string

**Note**: The implementation uses `insert(0, char)` which may have $O(n)$ cost per insertion in some string implementations, but `String::with_capacity()` pre-allocation can mitigate this.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Function Signature**:
```rust
pub fn decimal_to_hexadecimal(base_num: u64) -> String
```
- Takes `u64` by value (8 bytes, cheap to copy)
- Returns owned `String`
- Input range: $[0, 2^{64} - 1]$ (18,446,744,073,709,551,615)
- Output: Up to 16 hex digits for max `u64`

**Character Conversion**:
```rust
let hex_char = if remainder < 10 {
    (remainder as u8 + b'0') as char    // '0'-'9'
} else {
    (remainder as u8 - 10 + b'A') as char  // 'A'-'F'
};
```

**String Building Strategy**:
```rust
hexadecimal_num.insert(0, hex_char);  // Prepend (may be O(n))
```

**Optimization Opportunities**:

1. **Use `Vec<u8>` and reverse**:
   ```rust
   let mut digits = Vec::with_capacity(16);
   loop {
       let remainder = num % 16;
       digits.push(if remainder < 10 { b'0' + remainder as u8 } 
                   else { b'A' + (remainder - 10) as u8 });
       num /= 16;
       if num == 0 { break; }
   }
   digits.reverse();
   String::from_utf8(digits).unwrap()
   ```

2. **Use format! macro** (most idiomatic):
   ```rust
   format!("{:X}", base_num)  // Uppercase
   format!("{:x}", base_num)  // Lowercase
   ```

3. **Pre-allocate capacity**:
   ```rust
   let mut hex = String::with_capacity(16);  // Max hex digits for u64
   ```

**Bitwise Alternative**:
```rust
let mut hex = String::new();
let mut num = base_num;
loop {
    let nibble = (num & 0xF) as u8;  // Extract 4 bits
    let hex_char = if nibble < 10 { b'0' + nibble } 
                   else { b'A' + (nibble - 10) };
    hex.insert(0, hex_char as char);
    num >>= 4;  // Shift right by 4 bits
    if num == 0 { break; }
}
```

### 5.2 Edge Cases

1. **Zero**: "0" (special case)
2. **Single Digit Decimal**: 
   - 0-9 → "0"-"9"
   - No conversion needed
3. **Single Digit Hex**:
   - 10 → "A"
   - 15 → "F"
4. **Powers of 16**:
   - $16^0 = 1$ → "1"
   - $16^1 = 16$ → "10"
   - $16^2 = 256$ → "100"
   - $16^3 = 4096$ → "1000"
5. **Common Values**:
   - 255 → "FF" (byte maximum)
   - 256 → "100"
   - 65535 → "FFFF" (16-bit maximum)
6. **Maximum u64**: 
   - $2^{64} - 1$ = 18,446,744,073,709,551,615
   - → "FFFFFFFFFFFFFFFF" (16 F's)
7. **Boundary Values**:
   - 2,147,483,647 → "7FFFFFFF" (max i32)
   - 2,147,483,648 → "80000000"

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Memory Address Display**:
   - Debugger output formatting
   - Memory dump utilities
   - Core dump analysis
   - Example: Address 1048576 → "0x100000"

2. **Color Code Generation**:
   - Web development (CSS hex colors)
   - Image editors
   - Graphics APIs
   - Example: RGB(255, 87, 51) → "#FF5733"

3. **Checksum and Hash Display**:
   - MD5, SHA-256 hash formatting
   - File integrity verification
   - Cryptographic key representation
   - Example: Hash display in 32-character hex string

4. **Hardware Programming**:
   - Device register configuration
   - Embedded systems firmware
   - Memory-mapped I/O
   - Example: Configure register at 0x40021000

5. **Network Protocol Development**:
   - MAC address formatting
   - IPv6 address display
   - Packet hex dumps
   - Example: MAC "00:1A:2B:3C:4D:5E"

6. **File Format Processing**:
   - Magic number identification
   - Binary file analysis
   - Executable headers
   - Example: ELF magic "7F454C46"

7. **Assembly Programming**:
   - Machine code display
   - Opcode representation
   - Assembly listing generation

8. **Data Serialization**:
   - UUID generation and display
   - Binary data in JSON/XML
   - URL encoding for binary data

### 6.2 Related Algorithms

**Inverse Operation**:
- **Hexadecimal to Decimal**: Parse hex string to integer
- Both have $O(\log_{16} n)$ time complexity

**Related Base Conversions**:
| Conversion | Algorithm | Complexity | Notes |
|------------|-----------|------------|-------|
| Decimal → Binary | Repeated ÷2 | $O(\log_2 n)$ | 4× more iterations |
| Decimal → Octal | Repeated ÷8 | $O(\log_8 n)$ | Base 8 |
| Decimal → Hex | Repeated ÷16 | $O(\log_{16} n)$ | Most compact |
| Binary → Hex | Group by 4 | $O(n)$ | Direct mapping |

**Optimization Techniques**:
| Method | Time | Space | Pros | Cons |
|--------|------|-------|------|------|
| Repeated Division | $O(\log_{16} n)$ | $O(\log_{16} n)$ | Clear, educational | Moderate speed |
| Bit Shifting | $O(\log_{16} n)$ | $O(\log_{16} n)$ | Faster | Less readable |
| format! Macro | $O(\log_{16} n)$ | $O(\log_{16} n)$ | Idiomatic Rust | Less control |
| Lookup Table | $O(1)$ per nibble | $O(16)$ | Fastest | Fixed input size only |

**When to Use Each**:
- **format!("{:X}", n)**: Production Rust code (idiomatic, fast, tested)
- **Manual conversion**: Educational purposes, custom requirements
- **Bit shifting**: Performance-critical embedded systems
- **Lookup tables**: Fixed-size binary data conversion

**Related Problems**:
1. **RGB to Hex Color**: Combine three decimal→hex conversions
2. **UUID Formatting**: Convert 128-bit number with hyphens
3. **Base64 Encoding**: Different encoding scheme for binary data
4. **Gray Code**: Alternative binary encoding

**Conversion Chains**:
```
Common workflows:
1. Decimal → Hex → Display to user
2. Binary data → Hex → Human readable
3. Integer → Hex → Protocol message
4. Memory address → Hex → Debugger output
```

## 7. References

1. **Knuth, Donald E.** (1997). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (3rd ed.). Addison-Wesley. Section 4.1: Positional Number Systems.

2. **Warren, Henry S.** (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley. Chapter 2: Basics.

3. **Wikipedia**: [Hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal)

4. **RFC 3986**: Uniform Resource Identifier (URI) - Percent-encoding using hexadecimal

5. **W3C CSS Color Module Level 3**: Hexadecimal color notation standard

6. **Intel® 64 and IA-32 Architectures Software Developer's Manual**: Assembly language hex notation

7. **Rust Documentation**: [format! macro](https://doc.rust-lang.org/std/macro.format.html)

8. **Rust Documentation**: [std::fmt formatting traits](https://doc.rust-lang.org/std/fmt/)

9. **ISO/IEC 9899:2018**: C Programming Language Standard - Hexadecimal constants and notation

10. **ARM Architecture Reference Manual**: Hardware register specification in hexadecimal

11. **Sedgewick, Robert & Wayne, Kevin** (2011). *Algorithms* (4th ed.). Addison-Wesley.
