# Is Power of Two

## 1. Overview

The **Is Power of Two** algorithm determines whether a given positive integer is a power of two (i.e., can be expressed as $2^k$ for some non-negative integer $k$). This is one of the most elegant applications of bit manipulation, using a single bitwise operation to solve what would otherwise require iterative division or logarithms.

### Historical Context

This technique has been part of programming folklore since the early days of computing. It was popularized in Henry S. Warren Jr.'s seminal book *Hacker's Delight* (2002), which collected numerous bit manipulation tricks used by assembly programmers.

### Key Insight

Powers of two have a unique property in binary representation: they have exactly **one bit set to 1**. This property enables an O(1) solution using the formula `n & (n - 1) == 0`.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: A non-negative integer $n$  
**Output**: `true` if $\exists k \in \mathbb{Z}_{\geq 0}$ such that $n = 2^k$, otherwise `false`

The powers of two sequence: $1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, ...$

### 2.2 Mathematical Model

#### Binary Representation of Powers of Two

For any power of two $2^k$, the binary representation has exactly one `1` bit at position $k$:

| $n$ | Binary | $2^k$ |
|-----|--------|-------|
| 1 | `0001` | $2^0$ |
| 2 | `0010` | $2^1$ |
| 4 | `0100` | $2^2$ |
| 8 | `1000` | $2^3$ |
| 16 | `10000` | $2^4$ |

#### The n & (n-1) Trick

When we subtract 1 from a power of two:
- The single `1` bit becomes `0`
- All bits to the right become `1`

```
n     = 2^k = 100...00  (single 1 followed by k zeros)
n - 1 =       011...11  (k ones)
n & (n-1) =   000...00  (all zeros)
```

For non-powers of two, at least one bit remains set:
```
n = 6     = 110
n - 1 = 5 = 101
n & (n-1) = 100 ≠ 0
```

### 2.3 Correctness Proof

**Theorem**: For $n > 0$, $n$ is a power of two if and only if $n \land (n-1) = 0$.

**Proof**:

$(\Rightarrow)$ If $n = 2^k$ for some $k \geq 0$:
- $n$ has binary representation with a single `1` at position $k$
- $n - 1$ has all bits below position $k$ set to `1` and position $k$ set to `0`
- $n \land (n-1) = 0$ since no bit positions overlap

$(\Leftarrow)$ If $n \land (n-1) = 0$ and $n > 0$:
- Let $n$ have its rightmost `1` bit at position $j$
- After subtracting 1, position $j$ becomes `0` and all positions $< j$ become `1`
- For $n \land (n-1) = 0$, there can be no `1` bits at positions $> j$
- Therefore $n$ has exactly one `1` bit, meaning $n = 2^j$ $\square$

## 3. Algorithm Description

### 3.1 Intuition

Think of the operation `n - 1` as "borrowing" from the rightmost `1` bit. If that's the only `1` bit (power of two), the entire number becomes zero after the AND operation. If there are other `1` bits, they survive the operation.

### 3.2 Pseudocode

```
function is_power_of_two(n):
    if n < 0:
        return ERROR("number must not be negative")
    
    return (n & (n - 1)) == 0
```

### 3.3 Step-by-Step Example

**Example 1**: Check if 16 is a power of two

```
n = 16 = 10000 (binary)

Step 1: Compute n - 1
n - 1 = 15 = 01111 (binary)

Step 2: Compute n & (n - 1)
  10000
& 01111
-------
  00000 = 0

Step 3: Check if result is 0
0 == 0 → true

Result: 16 IS a power of two ✓
```

**Example 2**: Check if 12 is a power of two

```
n = 12 = 1100 (binary)

Step 1: Compute n - 1
n - 1 = 11 = 1011 (binary)

Step 2: Compute n & (n - 1)
  1100
& 1011
------
  1000 = 8

Step 3: Check if result is 0
8 ≠ 0 → false

Result: 12 is NOT a power of two ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| All cases | $O(1)$ | Fixed number of bitwise operations |

The algorithm performs exactly:
- 1 comparison (`n < 0`)
- 1 subtraction (`n - 1`)
- 1 bitwise AND (`&`)
- 1 equality comparison (`== 0`)

All operations are O(1) regardless of the input value.

### 4.2 Space Complexity

| Metric | Complexity |
|--------|------------|
| Auxiliary space | $O(1)$ |
| Stack space | $O(1)$ |

Only a constant amount of extra space is used for the intermediate computation.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn is_power_of_two(number: i32) -> Result<bool, String> {
    if number < 0 {
        return Err("number must not be negative".to_string());
    }

    let num = number as u32;
    Ok(num & num.wrapping_sub(1) == 0)
}
```

**Key Implementation Details**:
- Uses `wrapping_sub` to handle edge cases safely
- Converts to unsigned for cleaner bit manipulation
- Returns `Result` for proper error handling

### 5.2 Edge Cases

| Input | Expected Output | Reasoning |
|-------|-----------------|-----------|
| 0 | `true` | Convention: `0 & (-1) = 0` (debatable) |
| 1 | `true` | $2^0 = 1$ |
| 2 | `true` | $2^1 = 2$ |
| 3 | `false` | Not a power of two |
| -1 | `Error` | Negative numbers invalid |
| -8 | `Error` | Negative numbers invalid |
| $2^{30}$ | `true` | Largest power of 2 in i32 |

**Note on Zero**: The implementation treats 0 as a power of two (`0 & (-1) = 0`). Some implementations exclude 0 by checking `n > 0 && (n & (n-1)) == 0`.

### 5.3 Alternative Implementations

**Using built-in count_ones:**
```rust
fn is_power_of_two_popcount(n: u32) -> bool {
    n > 0 && n.count_ones() == 1
}
```

**Using logarithm:**
```rust
fn is_power_of_two_log(n: u32) -> bool {
    n > 0 && (1 << (n.trailing_zeros())) == n
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Memory Allocation**
   - Memory allocators often require power-of-two alignment
   - Page sizes (4KB, 2MB, 1GB) are powers of two
   - Checking allocation size validity

2. **Hash Table Sizing**
   - Power-of-two sizes allow modulo via bitwise AND: `hash & (size - 1)`
   - Much faster than division-based modulo

3. **Buffer Management**
   - Ring buffers use power-of-two sizes for efficient wrap-around
   - DMA transfers often require aligned buffers

4. **Graphics Programming**
   - Texture dimensions (OpenGL historically required power-of-two)
   - Mipmap levels follow power-of-two sizing

5. **Audio Processing**
   - FFT sizes are typically powers of two
   - Buffer sizes for real-time audio

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| [Previous Power of Two](previous_power_of_two.md) | Find largest $2^k \leq n$ |
| [Count Set Bits](counting_bits.md) | Alternative check: popcount == 1 |
| [Highest Set Bit](highest_set_bit.md) | Related bit position operation |

## 7. Comparison of Methods

| Method | Time | Pros | Cons |
|--------|------|------|------|
| `n & (n-1) == 0` | O(1) | Fastest, no branching | Requires understanding |
| `count_ones() == 1` | O(1) | Readable, uses POPCNT | Slightly slower on some CPUs |
| Repeated division | O(log n) | Intuitive | Much slower |
| Logarithm | O(1) | Mathematical | Floating-point overhead |

## 8. Common Pitfalls

### Pitfall 1: Forgetting Zero
```rust
// Bug: Returns true for 0
fn is_power_of_two_buggy(n: u32) -> bool {
    n & (n - 1) == 0  // 0 & u32::MAX == 0
}

// Fix: Explicitly handle zero if needed
fn is_power_of_two_fixed(n: u32) -> bool {
    n > 0 && (n & (n - 1)) == 0
}
```

### Pitfall 2: Signed Integer Overflow
```rust
// Bug: i32::MIN - 1 overflows
fn is_power_of_two_buggy(n: i32) -> bool {
    (n & (n - 1)) == 0  // Panics on debug for negative
}

// Fix: Use wrapping_sub or convert to unsigned
fn is_power_of_two_fixed(n: i32) -> bool {
    if n <= 0 { return false; }
    let n = n as u32;
    n & (n - 1) == 0
}
```

## 9. References

1. Warren, H. S. (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley. Chapter 2.
2. Anderson, S. E. (2005). *Bit Twiddling Hacks*. Stanford University.
3. Rust Documentation: [`std::primitive::u32::is_power_of_two`](https://doc.rust-lang.org/std/primitive.u32.html#method.is_power_of_two)
