# Rightmost Set Bit (Least Significant Set Bit)

## 1. Overview

The Rightmost Set Bit algorithm finds the position of the least significant bit (LSB) that is set to 1 in a given positive integer. This is a fundamental bit manipulation operation used in various algorithms including Fenwick trees, priority queues, and low-level system programming.

The position is 1-indexed, where position 1 represents the least significant bit.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer $n$, find the smallest $k$ such that bit $(k-1)$ is set to 1.

### 2.2 Mathematical Model

**Input:** A positive integer $n > 0$

**Output:** The 1-based position of the rightmost set bit

**Key Formula:** 
$$\text{rightmost\_bit} = n \land (-n)$$

This isolates the rightmost set bit using two's complement arithmetic.

**Examples:**
| Input $n$ | Binary | Rightmost Set Bit | Position |
|-----------|--------|-------------------|----------|
| 1 | 1 | 1 | 1 |
| 2 | 10 | 10 | 2 |
| 5 | 101 | 1 | 1 |
| 12 | 1100 | 100 | 3 |
| 18 | 10010 | 10 | 2 |

### 2.3 Why $n \land (-n)$ Works

In two's complement representation:
- $-n$ is computed as $\sim n + 1$ (flip bits and add 1)
- All bits to the right of the rightmost 1 are 0 in both $n$ and $-n$
- The rightmost 1 bit is 1 in both $n$ and $-n$
- All bits to the left differ between $n$ and $-n$

Therefore, $n \land (-n)$ isolates exactly the rightmost set bit.

## 3. Algorithm Description

### 3.1 Intuition

1. Use the two's complement trick to isolate the rightmost set bit
2. Find the position by computing $\log_2$ of the isolated bit
3. Add 1 for 1-based indexing

### 3.2 Pseudocode

```
function index_of_rightmost_set_bit(num):
    if num <= 0:
        return ERROR
    
    rightmost_bit = num AND (-num)  // Isolate the bit
    position = log2(rightmost_bit) + 1  // Convert to 1-based index
    
    return position
```

### 3.3 Step-by-Step Example

**Input:** `num = 18` (binary: `10010`)

| Step | Operation | Value | Binary |
|------|-----------|-------|--------|
| 1 | num | 18 | 10010 |
| 2 | -num | -18 | ...101110 (two's complement) |
| 3 | num & -num | 2 | 00010 |
| 4 | trailing_zeros | 1 | - |
| 5 | position | 2 | 1 + 1 |

**Result:** Position `2`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| All | $O(1)$ | Constant time bitwise operations |

### 4.2 Space Complexity

- **Auxiliary Space:** $O(1)$
- **Total Space:** $O(1)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn index_of_rightmost_set_bit(num: i32) -> Result<u32, String> {
    if num <= 0 {
        return Err("input must be a positive integer".to_string());
    }
    
    // Isolate the rightmost set bit
    let rightmost_bit = num & -num;
    
    // Use trailing_zeros which gives 0-based position
    let position = rightmost_bit.trailing_zeros() + 1;
    
    Ok(position)
}
```

**Key patterns:**
- Uses `Result` for error handling
- `trailing_zeros()` is a built-in method that counts trailing zeros
- Two's complement works naturally with signed integers

### 5.2 Alternative Using Logarithm

```rust
pub fn index_of_rightmost_set_bit_log(num: i32) -> Result<u32, String> {
    if num <= 0 {
        return Err("input must be a positive integer".to_string());
    }
    
    let rightmost_bit = num & -num;
    let position = (rightmost_bit as f64).log2() as u32 + 1;
    
    Ok(position)
}
```

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | `Err` | No bits set |
| -5 | `Err` | Negative numbers invalid |
| 1 | `Ok(1)` | Only bit 0 set |
| 16 | `Ok(5)` | Power of 2 |
| 7 | `Ok(1)` | Odd number |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Fenwick Trees:** Index calculations for update/query operations
2. **Binary Indexed Operations:** Finding lowest differing bit
3. **Memory Alignment:** Computing alignment requirements
4. **Set Iteration:** Iterating through set bits efficiently
5. **Priority Scheduling:** Round-robin scheduling implementations

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| `highest_set_bit` | MSB instead of LSB |
| `count_trailing_zeros` | Equivalent to position - 1 |
| `is_power_of_two` | True if only one bit set |
| Fenwick Tree | Uses LSB for tree traversal |

### 6.3 Bit Isolation Variants

| Operation | Formula | Purpose |
|-----------|---------|---------|
| Isolate rightmost 1 | `n & -n` | Get the LSB value |
| Clear rightmost 1 | `n & (n-1)` | Remove the LSB |
| Set rightmost 0 | `n | (n+1)` | Set the lowest 0 bit |

## 7. References

1. Warren, H. "Hacker's Delight." Addison-Wesley, 2012. Chapter 2.
2. Fenwick, P. "A New Data Structure for Cumulative Frequency Tables."
3. Intel. "BSF - Bit Scan Forward" instruction documentation.
