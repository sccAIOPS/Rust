# Binary Coded Decimal (BCD)

## 1. Overview

Binary Coded Decimal (BCD) is a class of binary encodings of decimal numbers where each decimal digit is represented by a fixed number of binary digits, usually four. This encoding allows decimal numbers to be stored and processed in binary form while maintaining a direct correspondence with their decimal representation.

BCD was invented in the early days of computing and is still used in applications where decimal precision is crucial, such as financial calculations and digital displays.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a decimal integer $n$, convert it to its BCD representation where each decimal digit (0-9) is encoded as a 4-bit binary value.

### 2.2 Mathematical Model

**Input:** An integer $n \geq 0$

**Output:** A binary string where each 4-bit group represents one decimal digit

**Encoding Table:**
| Decimal | BCD (4-bit) |
|---------|-------------|
| 0 | 0000 |
| 1 | 0001 |
| 2 | 0010 |
| 3 | 0011 |
| 4 | 0100 |
| 5 | 0101 |
| 6 | 0110 |
| 7 | 0111 |
| 8 | 1000 |
| 9 | 1001 |

### 2.3 Key Properties

1. **Self-complementing:** Some BCD variants (like Excess-3) are self-complementing
2. **Waste:** 6 out of 16 possible 4-bit combinations are unused (1010-1111)
3. **No rounding errors:** Exact decimal representation without floating-point issues

## 3. Algorithm Description

### 3.1 Intuition

The algorithm processes each decimal digit independently:
1. Extract each digit from the number
2. Convert each digit to its 4-bit binary equivalent
3. Concatenate all 4-bit groups

### 3.2 Pseudocode

```
function binary_coded_decimal(number):
    if number < 0:
        return BCD(0)
    
    result = ""
    for each digit d in number (left to right):
        binary_digit = format(d, "04b")  // 4-bit binary
        result = result + binary_digit
    
    return "0b" + result
```

### 3.3 Step-by-Step Example

**Input:** `n = 987`

| Step | Digit | Binary (4-bit) | Cumulative Result |
|------|-------|----------------|-------------------|
| 1 | 9 | 1001 | 1001 |
| 2 | 8 | 1000 | 10011000 |
| 3 | 7 | 0111 | 100110000111 |

**Output:** `0b100110000111`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| Best | $O(d)$ | Process $d$ digits |
| Average | $O(d)$ | Process $d$ digits |
| Worst | $O(d)$ | Process $d$ digits |

Where $d = \lfloor \log_{10}(n) \rfloor + 1$ is the number of decimal digits.

### 4.2 Space Complexity

- **Auxiliary Space:** $O(d)$ for the output string
- **Total Space:** $O(d)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn binary_coded_decimal(number: i32) -> String {
    let num = if number < 0 { 0 } else { number };
    let digits = num.to_string();
    
    let bcd = digits.chars().fold(String::new(), |mut acc, digit| {
        let digit_value = digit.to_digit(10).unwrap();
        write!(acc, "{digit_value:04b}").unwrap();
        acc
    });
    
    format!("0b{bcd}")
}
```

**Key patterns:**
- Uses `fold` for efficient string building
- Format specifier `{:04b}` ensures 4-bit padding
- Negative numbers are treated as 0

### 5.2 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | `0b0000` | Single digit |
| -5 | `0b0000` | Negative treated as 0 |
| 10 | `0b00010000` | Two digits |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Financial Systems:** BCD prevents rounding errors in currency calculations
2. **Digital Displays:** Seven-segment displays often use BCD input
3. **Legacy Systems:** COBOL and mainframe applications use packed BCD
4. **Real-Time Clocks:** Many RTCs store time in BCD format

### 6.2 Related Algorithms

| Algorithm | Use Case |
|-----------|----------|
| Packed BCD | Two digits per byte (more efficient) |
| Excess-3 BCD | Self-complementing variant |
| Gray Code | Related binary encoding |

## 7. References

1. IBM. "Packed Decimal Format." IBM Documentation.
2. IEEE. "Decimal Floating-Point Arithmetic." IEEE 754-2008.
3. Patterson, D. & Hennessy, J. "Computer Organization and Design."
