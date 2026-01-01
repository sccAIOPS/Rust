# Conversion Algorithms

## Overview

This category contains algorithms for converting between different numeral systems, color spaces, and measurement units. These conversions are fundamental operations in computer science, enabling translation between human-readable representations and machine formats, as well as between different standards and conventions.

## Categories

### Number Base Conversions

Algorithms for converting numbers between different positional notation systems.

#### Binary Conversions
- **[Binary to Decimal](binary_to_decimal.md)** - Convert base-2 to base-10
- **[Decimal to Binary](decimal_to_binary.md)** - Convert base-10 to base-2

#### Hexadecimal Conversions
- **[Hexadecimal to Decimal](hexadecimal_to_decimal.md)** - Convert base-16 to base-10
- **[Decimal to Hexadecimal](decimal_to_hexadecimal.md)** - Convert base-10 to base-16

#### Octal Conversions
- **[Octal to Decimal](octal_to_decimal.md)** - Convert base-8 to base-10

### Color Space Conversions

Algorithms for converting between different color representation models.

- **[RGB to CMYK](rgb_cmyk_conversion.md)** - Convert additive (screen) color model to subtractive (print) color model

### Unit Conversions

Algorithms for converting between different measurement systems.

- **[Length Conversion](length_conversion.md)** - Convert between metric and imperial length units

## Quick Reference

### Number Base Conversion Matrix

| From / To | Binary | Octal | Decimal | Hexadecimal |
|-----------|--------|-------|---------|-------------|
| **Binary** | - | Group by 3 | [→](binary_to_decimal.md) | Group by 4 |
| **Octal** | Expand to 3 bits | - | [→](octal_to_decimal.md) | Via binary |
| **Decimal** | [→](decimal_to_binary.md) | Divide by 8 | - | [→](decimal_to_hexadecimal.md) |
| **Hexadecimal** | Expand to 4 bits | Via binary | [→](hexadecimal_to_decimal.md) | - |

### Complexity Summary

| Algorithm | Time | Space | Key Feature |
|-----------|------|-------|-------------|
| Binary ↔ Decimal | O(n) | O(1)* | Bit-by-bit processing |
| Hex ↔ Decimal | O(n) | O(1)* | Radix 16 conversion |
| Octal ↔ Decimal | O(n) | O(1)* | Radix 8 conversion |
| RGB → CMYK | O(1) | O(1) | Fixed calculation |
| Length Conversion | O(1) | O(1) | Hub-and-spoke pattern |

*Auxiliary space only; output size is O(log n) for the value

## Common Use Cases

### Software Development
1. **Memory Addresses** - Hexadecimal representation
2. **Bitwise Operations** - Binary visualization
3. **File Permissions** - Octal notation (Unix)
4. **Color Codes** - RGB hex values (#RRGGBB)
5. **International Software** - Unit conversions

### Low-Level Programming
1. **Assembly Language** - Binary/hex for instructions
2. **Hardware Registers** - Hex for configuration
3. **Debugging** - Memory dumps in hex
4. **Protocol Development** - Hex for packet data

### Design and Graphics
1. **Web Development** - RGB color codes
2. **Print Design** - CMYK color conversion
3. **Image Processing** - Color space transformations

### Engineering and Science
1. **CAD Software** - Unit conversions
2. **International Collaboration** - Metric/Imperial conversion
3. **Scientific Computing** - Standardized units

## Implementation Notes

### Rust-Specific Features

**Type Safety**:
```rust
// Number conversions use Options/Results for safety
pub fn binary_to_decimal(binary: &str) -> Option<u128>
pub fn hexadecimal_to_decimal(hex: &str) -> Result<u64, &'static str>
```

**Zero-Copy String Processing**:
```rust
// Use &str for input (borrowed, no allocation)
pub fn conversion(input: &str) -> String
```

**Overflow Protection**:
```rust
// Use checked arithmetic to prevent panics
if let Some(sum) = num.checked_add(&idx_val) {
    num = sum;
} else {
    return None;  // Overflow detected
}
```

### Performance Considerations

**Binary/Octal/Hex to Decimal**:
- Use built-in `from_str_radix` when possible (optimized)
- Process bits/digits from right to left
- Watch for overflow with large inputs

**Decimal to Binary/Octal/Hex**:
- Use `format!` macro for production code
- Manual implementation for educational purposes
- Consider pre-allocating string capacity

**Color Conversions**:
- Use `f64` for intermediate calculations
- Pattern matching for special cases (pure black)
- Truncate vs. round for final values

**Unit Conversions**:
- Hub-and-spoke pattern minimizes conversion factors
- Compile-time constant lookup
- Optimal for any-to-any conversion

## Best Practices

### Number Base Conversions

1. **Input Validation**: Always validate input strings
2. **Overflow Handling**: Use `Option`/`Result` types
3. **Case Insensitivity**: Accept both 'A'-'F' and 'a'-'f' for hex
4. **Leading Zeros**: Handle gracefully
5. **Empty Input**: Return appropriate error

### Color Conversions

1. **Gamut Awareness**: Simple RGB→CMYK doesn't handle out-of-gamut colors
2. **ICC Profiles**: Use for professional color work
3. **Floating Point**: Use adequate precision (f64 recommended)
4. **Special Cases**: Handle black, white, and grays explicitly

### Unit Conversions

1. **Precision**: Use `f64` to minimize rounding errors
2. **Standard Factors**: Use exact conversion factors from standards
3. **Transitivity**: Ensure A→B→C equals A→C
4. **Zero Identity**: 0 in any unit should convert to 0

## Testing Strategies

### Number Conversions
```rust
#[test]
fn test_edge_cases() {
    assert_eq!(binary_to_decimal("0"), Some(0));
    assert_eq!(binary_to_decimal("1111111111"), Some(1023));
    assert!(binary_to_decimal("").is_none());
}
```

### Color Conversions
```rust
#[test]
fn test_pure_colors() {
    assert_eq!(rgb_to_cmyk((255, 255, 255)), (0, 0, 0, 0));  // White
    assert_eq!(rgb_to_cmyk((0, 0, 0)), (0, 0, 0, 100));      // Black
}
```

### Unit Conversions
```rust
#[test]
fn test_round_trip() {
    let original = 100.0;
    let converted = length_conversion(original, Meter, Foot);
    let back = length_conversion(converted, Foot, Meter);
    assert!((original - back).abs() < 0.0001);
}
```

## Further Reading

### Number Systems
- Knuth, Donald E. *The Art of Computer Programming, Volume 2*
- Warren, Henry S. *Hacker's Delight*

### Color Science
- Sharma, Gaurav. *Digital Color Imaging Handbook*
- Hunt, R.W.G. *The Reproduction of Colour*

### Standards
- ISO/IEC 80000: Quantities and units
- NIST Special Publication 811: Guide for SI Units
- ICC Color Management Specifications

### Rust Resources
- Rust Standard Library Documentation
- Rust by Example: String handling
- The Rust Programming Language: Error handling

## Related Categories

- **[Ciphers](../ciphers/README.md)** - Base64 encoding (related to base conversion)
- **[String Algorithms](../string/README.md)** - String parsing techniques
- **[Math](../math/README.md)** - Number theory and arithmetic
- **[Bit Manipulation](../bit_manipulation/README.md)** - Low-level binary operations

## Contributing

When adding new conversion algorithms:

1. **Follow the template** in existing algorithm docs
2. **Include mathematical foundation** with formal definitions
3. **Provide complexity analysis** with derivations
4. **Document edge cases** thoroughly
5. **Add real-world use cases** from your experience
6. **Include Rust-specific considerations**
7. **Write comprehensive tests** covering edge cases

## Algorithm Status

✅ **Implemented and Documented**:
- Binary to Decimal
- Decimal to Binary
- Hexadecimal to Decimal
- Decimal to Hexadecimal
- Octal to Decimal
- RGB to CMYK
- Length Conversion

📋 **Additional Conversions in Codebase** (not yet documented):
- Binary to Hexadecimal
- Binary to Octal
- Hexadecimal to Binary
- Hexadecimal to Octal
- Octal to Binary
- Octal to Hexadecimal
- Decimal to Octal

🔄 **Potential Future Additions**:
- CMYK to RGB
- RGB to HSL/HSV
- Temperature conversions (Celsius, Fahrenheit, Kelvin)
- Mass unit conversions
- Volume unit conversions
- Time unit conversions
- Currency conversions (requires exchange rate data)

---

**Note**: All complexity analysis assumes the value to be converted is $n$. For string-based conversions, $n$ refers to the length of the string or the number of digits in the result.
