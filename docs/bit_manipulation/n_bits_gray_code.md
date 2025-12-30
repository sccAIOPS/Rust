# N-Bits Gray Code

## 1. Overview

Gray code (also known as reflected binary code) is a binary numeral system where two successive values differ in only one bit. This property makes Gray code useful in digital systems to prevent spurious output from switches and in error correction.

The algorithm generates all $2^n$ Gray code sequences for a given bit width $n$.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer $n$, generate all $2^n$ Gray code sequences of length $n$ bits, where consecutive codes differ by exactly one bit.

### 2.2 Mathematical Model

**Input:** Positive integer $n$ (number of bits)

**Output:** A sequence of $2^n$ binary strings, each of length $n$

**Gray Code Formula:**
$$G(i) = i \oplus (i >> 1)$$

Where $\oplus$ is XOR and $>>$ is right shift.

### 2.3 Key Properties

1. **Single-bit change:** Adjacent codes differ by exactly 1 bit
2. **Cyclic:** First and last codes also differ by 1 bit
3. **Reflection:** $n$-bit code can be built from $(n-1)$-bit code by reflection
4. **Bijective:** One-to-one mapping with binary numbers

**Example (3-bit Gray Code):**
| Decimal | Binary | Gray |
|---------|--------|------|
| 0 | 000 | 000 |
| 1 | 001 | 001 |
| 2 | 010 | 011 |
| 3 | 011 | 010 |
| 4 | 100 | 110 |
| 5 | 101 | 111 |
| 6 | 110 | 101 |
| 7 | 111 | 100 |

### 2.4 Reflection Construction

The $n$-bit Gray code is constructed from $(n-1)$-bit Gray code:
1. Prefix the $(n-1)$-bit codes with 0
2. Reverse the $(n-1)$-bit codes and prefix with 1
3. Concatenate both sequences

## 3. Algorithm Description

### 3.1 Intuition

For each integer $i$ from 0 to $2^n - 1$:
1. Compute Gray code using XOR formula: $G(i) = i \oplus (i >> 1)$
2. Convert to binary string with proper padding

### 3.2 Pseudocode

```
function generate_gray_code(n):
    if n == 0:
        return ERROR
    
    num_codes = 2^n
    result = []
    
    for i = 0 to num_codes - 1:
        gray = i XOR (i >> 1)
        code = format_binary(gray, n)  // n-bit binary string
        result.append(code)
    
    return result
```

### 3.3 Step-by-Step Example

**Input:** `n = 2`

| i | Binary(i) | i >> 1 | i XOR (i >> 1) | Gray Code |
|---|-----------|--------|----------------|-----------|
| 0 | 00 | 00 | 00 | "00" |
| 1 | 01 | 00 | 01 | "01" |
| 2 | 10 | 01 | 11 | "11" |
| 3 | 11 | 01 | 10 | "10" |

**Output:** `["00", "01", "11", "10"]`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| All | $O(2^n \cdot n)$ | Generate $2^n$ codes, each of length $n$ |

### 4.2 Space Complexity

- **Output Space:** $O(2^n \cdot n)$ for storing all codes
- **Auxiliary Space:** $O(n)$ for temporary string building

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Debug, PartialEq)]
pub enum GrayCodeError {
    ZeroBitCount,
}

pub fn generate_gray_code(n: usize) -> Result<Vec<String>, GrayCodeError> {
    if n == 0 {
        return Err(GrayCodeError::ZeroBitCount);
    }
    
    let num_codes = 1 << n;  // 2^n
    let mut result = Vec::with_capacity(num_codes);
    
    for i in 0..num_codes {
        let gray = i ^ (i >> 1);
        let gray_code = (0..n)
            .rev()
            .map(|bit| if gray & (1 << bit) != 0 { '1' } else { '0' })
            .collect::<String>();
        result.push(gray_code);
    }
    
    Ok(result)
}
```

**Key patterns:**
- Custom error type for zero bit count
- Pre-allocated vector with `with_capacity`
- Bit-by-bit string construction for proper padding

### 5.2 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | `Err(ZeroBitCount)` | Invalid input |
| 1 | `["0", "1"]` | Simplest Gray code |
| 2 | `["00", "01", "11", "10"]` | 4 codes |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Rotary Encoders:** Shaft position encoding without glitches
2. **Error Correction:** Karnaugh maps in logic minimization
3. **Genetic Algorithms:** Mutation operations
4. **ADC Design:** Analog-to-digital converter encoding
5. **Tower of Hanoi:** Optimal solution follows Gray code pattern

### 6.2 Variations

| Variant | Description |
|---------|-------------|
| Balanced Gray Code | Equal number of 0→1 and 1→0 transitions |
| n-ary Gray Code | More than 2 symbols |
| Beckett-Gray Code | For stage lighting control |

### 6.3 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Binary to Gray | Convert binary to Gray: `g = b ^ (b >> 1)` |
| Gray to Binary | Convert Gray to binary: iterative XOR |
| Hamiltonian Path | Gray code is a Hamiltonian path on hypercube |

## 7. References

1. Gray, F. "Pulse Code Communication." U.S. Patent 2,632,058 (1953).
2. Knuth, D. "The Art of Computer Programming, Vol. 4A." Addison-Wesley.
3. Savage, C. "A Survey of Combinatorial Gray Codes." SIAM Review.
