# Find Previous Power of Two

## 1. Overview

This algorithm finds the largest power of two that is less than or equal to a given non-negative integer. It's a fundamental bit manipulation technique used in memory allocation, hash table sizing, and binary tree operations.

The algorithm uses bit shifting operations to efficiently compute the result without costly division or logarithm operations.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a non-negative integer $n$, find the largest $k$ such that $2^k \leq n$.

Equivalently, find:
$$\text{result} = 2^{\lfloor \log_2(n) \rfloor}$$

### 2.2 Mathematical Model

**Input:** Integer $n \geq 0$

**Output:** $2^{\lfloor \log_2(n) \rfloor}$ for $n > 0$, or $0$ for $n = 0$

**Examples:**
| Input $n$ | $\lfloor \log_2(n) \rfloor$ | Output |
|-----------|----------------------------|--------|
| 1 | 0 | 1 |
| 5 | 2 | 4 |
| 8 | 3 | 8 |
| 15 | 3 | 8 |
| 16 | 4 | 16 |

### 2.3 Key Properties

1. **Idempotent on powers of 2:** If $n = 2^k$, then result $= n$
2. **Monotonic:** If $a \leq b$, then $f(a) \leq f(b)$
3. **Binary representation:** The result has exactly one bit set (the highest bit of $n$)

## 3. Algorithm Description

### 3.1 Intuition

Starting from 1, we double a counter (left-shift by 1) until it exceeds the input number. The value just before exceeding is the answer.

### 3.2 Pseudocode

```
function find_previous_power_of_two(n):
    if n < 0:
        return ERROR
    if n == 0:
        return 0
    
    power = 1
    while power <= n:
        power = power << 1  // multiply by 2
    
    return power >> 1  // divide by 2
```

### 3.3 Step-by-Step Example

**Input:** `n = 13`

| Iteration | power | power ≤ n? | Action |
|-----------|-------|------------|--------|
| 0 | 1 | 1 ≤ 13 ✓ | shift left |
| 1 | 2 | 2 ≤ 13 ✓ | shift left |
| 2 | 4 | 4 ≤ 13 ✓ | shift left |
| 3 | 8 | 8 ≤ 13 ✓ | shift left |
| 4 | 16 | 16 ≤ 13 ✗ | exit loop |

**Result:** `16 >> 1 = 8`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| Best | $O(1)$ | $n = 0$ |
| Average | $O(\log n)$ | Loop runs $\lceil \log_2(n) \rceil$ times |
| Worst | $O(\log n)$ | Maximum iterations for large $n$ |

### 4.2 Space Complexity

- **Auxiliary Space:** $O(1)$
- **Total Space:** $O(1)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn find_previous_power_of_two(number: i32) -> Result<u32, String> {
    if number < 0 {
        return Err("Input must be a non-negative integer".to_string());
    }
    
    let number = number as u32;
    if number == 0 {
        return Ok(0);
    }
    
    let mut power = 1u32;
    while power <= number {
        power <<= 1;
    }
    
    Ok(if number > 1 { power >> 1 } else { 1 })
}
```

**Key patterns:**
- Uses `Result` for error handling
- Type conversion from `i32` to `u32` for bit operations
- Handles edge case of `n = 1` explicitly

### 5.2 Alternative Implementation

Using leading zeros (more efficient on modern CPUs):

```rust
pub fn find_previous_power_of_two_fast(n: u32) -> u32 {
    if n == 0 { return 0; }
    1 << (31 - n.leading_zeros())
}
```

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | 0 | Special case |
| 1 | 1 | $2^0 = 1$ |
| 2 | 2 | Exact power of 2 |
| 3 | 2 | Between powers |
| -5 | Error | Negative input |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Memory Allocators:** Finding aligned memory blocks
2. **Hash Tables:** Sizing tables to powers of 2 for fast modulo
3. **Graphics:** Texture dimensions often require power-of-2 sizes
4. **Buffer Management:** Allocating power-of-2 sized buffers

### 6.2 Related Algorithms

| Algorithm | Description |
|-----------|-------------|
| `is_power_of_two` | Check if $n$ is a power of 2 |
| Next Power of Two | Find smallest $2^k \geq n$ |
| `highest_set_bit` | Find position of MSB |

## 7. References

1. Warren, H. "Hacker's Delight." Addison-Wesley, 2012.
2. Anderson, S. "Bit Twiddling Hacks." Stanford Graphics.
3. Intel. "Intel 64 and IA-32 Architectures Software Developer's Manual."
