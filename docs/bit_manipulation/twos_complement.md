# Two's Complement

## 1. Overview

Two's complement is the most common method of representing signed integers in computers. This algorithm converts a non-positive integer to its two's complement binary representation, demonstrating how negative numbers are encoded in binary.

Two's complement was adopted because it simplifies hardware design—the same circuitry can perform addition on both positive and negative numbers.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a non-positive integer $n \leq 0$, produce its two's complement binary representation.

### 2.2 Mathematical Model

For an $m$-bit representation:
- **Range:** $[-2^{m-1}, 2^{m-1} - 1]$
- **Negative number $n$:** Represented as $2^m + n$ (or equivalently $2^m - |n|$)

**Two's Complement Formula:**
$$\text{twos\_complement}(n) = 2^m - |n|$$

Where $m$ is the minimum bits needed: $m = \lfloor \log_2(|n|) \rfloor + 2$

### 2.3 Key Properties

1. **Sign bit:** The MSB indicates sign (1 = negative, 0 = non-negative)
2. **Negation:** $-n = \sim n + 1$ (flip bits and add 1)
3. **Single zero:** Only one representation of zero (unlike one's complement)
4. **Asymmetric range:** One more negative than positive numbers

### 2.4 Examples

| Decimal | Two's Complement |
|---------|------------------|
| 0 | 0 |
| -1 | 11 |
| -2 | 110 |
| -5 | 1011 |
| -17 | 101111 |

## 3. Algorithm Description

### 3.1 Intuition

For a negative number $n$:
1. Determine how many bits are needed for $|n|$
2. Compute $2^{\text{bits}} - |n|$ to get the magnitude in two's complement
3. Prefix with '1' to indicate negative

### 3.2 Pseudocode

```
function twos_complement(number):
    if number > 0:
        return ERROR  // Only non-positive allowed
    
    if number == 0:
        return "0b0"
    
    // Calculate bits needed (excluding sign)
    bits = floor(log2(abs(number))) + 1
    
    // Two's complement value
    complement = abs(number) - 2^bits
    
    // Format with leading 1 for sign
    return "0b1" + format_binary(abs(complement), bits)
```

### 3.3 Step-by-Step Example

**Input:** `n = -5`

| Step | Operation | Value |
|------|-----------|-------|
| 1 | abs(n) | 5 |
| 2 | Binary of 5 | 101 |
| 3 | bits needed | 3 |
| 4 | 2^3 | 8 |
| 5 | 5 - 8 | -3 |
| 6 | abs(-3) | 3 |
| 7 | Binary of 3 | 011 |
| 8 | Add sign bit | 1011 |

**Result:** `0b1011`

**Verification:** In 4-bit two's complement:
- 1011 = -(2^3) + 0×2^2 + 1×2^1 + 1×2^0 = -8 + 2 + 1 = -5 ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| All | $O(\log n)$ | Binary string formatting |

### 4.2 Space Complexity

- **Auxiliary Space:** $O(\log n)$ for output string
- **Total Space:** $O(\log n)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn twos_complement(number: i32) -> Result<String, String> {
    if number > 0 {
        return Err("input must be a negative integer".to_string());
    }
    
    if number == 0 {
        return Ok("0b0".to_string());
    }
    
    // Bits needed for absolute value
    let binary_number_length = format!("{:b}", number.abs()).len();
    
    // Two's complement calculation
    let twos_complement_value = 
        (number.abs() as i64) - (1_i64 << binary_number_length);
    
    // Format binary string
    let mut twos_complement_str = 
        format!("{:b}", twos_complement_value.abs());
    
    // Pad with zeros if needed
    let padding = binary_number_length.saturating_sub(twos_complement_str.len());
    if padding > 0 {
        twos_complement_str = format!("{}{}", "0".repeat(padding), twos_complement_str);
    }
    
    Ok(format!("0b1{twos_complement_str}"))
}
```

**Key patterns:**
- Uses `Result` for error handling
- `i64` intermediate to avoid overflow
- Manual padding for correct bit width

### 5.2 Alternative: Direct Bit Method

```rust
pub fn twos_complement_direct(n: i32) -> String {
    if n >= 0 {
        return format!("0b{:b}", n);
    }
    // Rust's {:b} already uses two's complement internally
    format!("0b{:b}", n as u32)
}
```

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | `0b0` | Zero case |
| 1 | `Err` | Positive not allowed |
| -1 | `0b11` | All 1s (in minimum bits) |
| -128 | `0b10000000` | Power of 2 |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **CPU Architecture:** Standard signed integer representation
2. **Arithmetic Operations:** Subtraction via addition
3. **Compiler Design:** Integer type implementation
4. **Embedded Systems:** Low-level bit manipulation
5. **Digital Signal Processing:** Audio/video processing

### 6.2 Comparison with Other Representations

| Method | -5 in 8-bit | Zero | Issues |
|--------|-------------|------|--------|
| Sign-magnitude | 10000101 | +0, -0 | Two zeros |
| One's complement | 11111010 | +0, -0 | Two zeros |
| **Two's complement** | **11111011** | 0 | None |

### 6.3 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| One's complement | Flip all bits (no +1) |
| Sign-magnitude | Separate sign bit |
| Bitwise negation | Part of two's complement |

## 7. References

1. Wikipedia. "Two's Complement." https://en.wikipedia.org/wiki/Two%27s_complement
2. Patterson, D. & Hennessy, J. "Computer Organization and Design."
3. IEEE. "IEEE Standard for Floating-Point Arithmetic." IEEE 754-2019.
4. Warren, H. "Hacker's Delight." Addison-Wesley, 2012.
