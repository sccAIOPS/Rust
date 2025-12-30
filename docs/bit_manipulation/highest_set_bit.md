# Highest Set Bit (Most Significant Bit)

## 1. Overview

The Highest Set Bit algorithm finds the position of the most significant bit (MSB) that is set to 1 in a given positive integer. This is a fundamental bit manipulation operation used in logarithm calculations, priority queues, and various optimization techniques.

The position is 0-indexed, where position 0 represents the least significant bit (rightmost).

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer $n$, find the position $k$ such that:
- Bit $k$ is set to 1
- All bits at positions $> k$ are 0

Mathematically: $k = \lfloor \log_2(n) \rfloor$

### 2.2 Mathematical Model

**Input:** A non-negative integer $n$

**Output:** 
- $\lfloor \log_2(n) \rfloor$ for $n > 0$
- `None` for $n = 0$

**Examples:**
| Input $n$ | Binary | MSB Position |
|-----------|--------|--------------|
| 1 | 1 | 0 |
| 2 | 10 | 1 |
| 3 | 11 | 1 |
| 8 | 1000 | 3 |
| 18 | 10010 | 4 |

### 2.3 Key Properties

1. **Range:** For $n$-bit integers, position ∈ $[0, n-1]$
2. **Uniqueness:** Every positive integer has exactly one MSB position
3. **Relationship:** $2^{\text{MSB}(n)} \leq n < 2^{\text{MSB}(n)+1}$

## 3. Algorithm Description

### 3.1 Intuition

Repeatedly right-shift the number until it becomes zero, counting the shifts. The count minus one gives the MSB position.

### 3.2 Pseudocode

```
function find_highest_set_bit(num):
    if num == 0:
        return None
    
    position = 0
    n = num
    
    while n > 0:
        n = n >> 1  // right shift (divide by 2)
        position = position + 1
    
    return position - 1
```

### 3.3 Step-by-Step Example

**Input:** `num = 18` (binary: `10010`)

| Iteration | n (decimal) | n (binary) | position |
|-----------|-------------|------------|----------|
| Start | 18 | 10010 | 0 |
| 1 | 9 | 1001 | 1 |
| 2 | 4 | 100 | 2 |
| 3 | 2 | 10 | 3 |
| 4 | 1 | 1 | 4 |
| 5 | 0 | 0 | 5 |

**Result:** `position - 1 = 5 - 1 = 4`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| Best | $O(1)$ | $n = 0$ |
| Average | $O(\log n)$ | Shifts proportional to bit width |
| Worst | $O(\log n)$ | Maximum shifts for large $n$ |

### 4.2 Space Complexity

- **Auxiliary Space:** $O(1)$
- **Total Space:** $O(1)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn find_highest_set_bit(num: usize) -> Option<usize> {
    if num == 0 {
        return None;
    }
    
    let mut position = 0;
    let mut n = num;
    
    while n > 0 {
        n >>= 1;
        position += 1;
    }
    
    Some(position - 1)
}
```

**Key patterns:**
- Returns `Option<usize>` to handle the zero case
- Uses `usize` for natural indexing
- Efficient bit shifting operations

### 5.2 Alternative: Using Built-in Functions

Rust provides `leading_zeros()` which can compute this more efficiently:

```rust
pub fn find_highest_set_bit_fast(num: usize) -> Option<usize> {
    if num == 0 {
        return None;
    }
    Some((usize::BITS - 1 - num.leading_zeros()) as usize)
}
```

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| 0 | `None` | No bits set |
| 1 | `Some(0)` | Only bit 0 set |
| 2 | `Some(1)` | Position 1 |
| `usize::MAX` | `Some(63)` | All bits set (64-bit) |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Integer Logarithm:** Computing $\lfloor \log_2(n) \rfloor$
2. **Priority Queues:** Finding highest priority element
3. **Binary Search Trees:** Estimating tree depth
4. **Compression:** Determining minimum bits needed
5. **Graphics:** Mipmap level selection

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| `count_leading_zeros` | Inverse calculation |
| `find_previous_power_of_two` | Uses MSB position |
| `rightmost_set_bit` | LSB instead of MSB |

## 7. References

1. Warren, H. "Hacker's Delight." Addison-Wesley, 2012.
2. Intel. "LZCNT - Count the Number of Leading Zero Bits."
3. ARM. "CLZ (Count Leading Zeros) instruction."
