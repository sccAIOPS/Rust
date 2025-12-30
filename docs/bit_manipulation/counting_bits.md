# Count Set Bits (Population Count)

## 1. Overview

The **Count Set Bits** algorithm, also known as **population count** or **popcount**, counts the number of `1` bits in the binary representation of an integer. This implementation uses **Brian Kernighan's algorithm**, which efficiently clears the least significant set bit in each iteration.

### Historical Context

The term "population count" originated from the 1960s when early computers needed to count bits for various applications. Brian Kernighan's elegant algorithm was popularized through *The C Programming Language* by Kernighan and Ritchie.

### Key Insight

The expression `n & (n - 1)` clears the rightmost set bit. By repeatedly applying this operation until the number becomes zero, we count exactly how many set bits existed.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: A non-negative integer $n$  
**Output**: The Hamming weight $H(n)$, i.e., the number of 1-bits in the binary representation of $n$

Formally: $H(n) = \sum_{i=0}^{w-1} b_i$ where $n = \sum_{i=0}^{w-1} b_i \cdot 2^i$ and $w$ is the word size.

### 2.2 Mathematical Model

#### Hamming Weight Examples

| $n$ (decimal) | Binary | Hamming Weight |
|---------------|--------|----------------|
| 0 | `0000` | 0 |
| 1 | `0001` | 1 |
| 5 | `0101` | 2 |
| 7 | `0111` | 3 |
| 15 | `1111` | 4 |
| 170 | `10101010` | 4 |
| 255 | `11111111` | 8 |

#### The n & (n-1) Trick

This operation clears the rightmost set bit:

```
n     = ...xxxxx1000...  (rightmost 1 at some position)
n - 1 = ...xxxxx0111...  (that 1 becomes 0, all lower bits become 1)
n & (n-1) = ...xxxxx0000...  (rightmost 1 is cleared)
```

**Key Property**: The operation `n &= (n - 1)` reduces the popcount by exactly 1.

### 2.3 Correctness Proof

**Theorem**: Brian Kernighan's algorithm correctly computes the population count.

**Proof by loop invariant**:

Let `count` be the loop counter and `n` the current value.

**Loop Invariant**: At the start of each iteration, `count` equals the number of bits cleared so far.

**Initialization**: `count = 0`, and 0 bits have been cleared.

**Maintenance**: 
- The operation `n &= (n - 1)` clears exactly one bit (the rightmost set bit)
- We increment `count` by 1
- Therefore, `count` still equals the number of bits cleared

**Termination**:
- The loop terminates when `n = 0` (no more bits to clear)
- All original set bits have been counted
- `count` equals the original popcount $\square$

## 3. Algorithm Description

### 3.1 Intuition

Instead of checking each bit position, we directly jump to the set bits. Each iteration of `n &= (n - 1)` eliminates exactly one `1` bit, so we only iterate as many times as there are set bits.

### 3.2 Pseudocode

**Brian Kernighan's Algorithm**:
```
function count_set_bits(n):
    count = 0
    while n > 0:
        n = n & (n - 1)  // Clear rightmost set bit
        count = count + 1
    return count
```

**Naive Approach (for comparison)**:
```
function count_set_bits_naive(n):
    count = 0
    while n > 0:
        if n & 1 == 1:
            count = count + 1
        n = n >> 1
    return count
```

### 3.3 Step-by-Step Example

**Example**: Count set bits in 13

```
n = 13 = 1101 (binary), expected popcount = 3

Iteration 1:
  n = 1101
  n-1 = 1100
  n & (n-1) = 1100
  count = 1

Iteration 2:
  n = 1100
  n-1 = 1011
  n & (n-1) = 1000
  count = 2

Iteration 3:
  n = 1000
  n-1 = 0111
  n & (n-1) = 0000
  count = 3

Loop terminates: n = 0

Result: 3 set bits ✓
```

**Visualization**:
```
1101 (13) → 1100 (12) → 1000 (8) → 0000 (0)
  ↓           ↓           ↓
clear bit0  clear bit2  clear bit3
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| Best case | $O(1)$ | When $n = 0$ or has 1 bit set |
| Worst case | $O(w)$ | When all $w$ bits are set (e.g., $n = 2^w - 1$) |
| Average case | $O(k)$ | Where $k$ is the number of set bits |

Brian Kernighan's algorithm is **output-sensitive**: its runtime depends on the number of set bits, not the word size.

### 4.2 Space Complexity

| Metric | Complexity |
|--------|------------|
| Auxiliary space | $O(1)$ |
| Stack space | $O(1)$ |

### 4.3 Comparison with Naive Approach

| Method | Time Complexity | Iterations for n=8 | Iterations for n=15 |
|--------|-----------------|---------------------|---------------------|
| Kernighan's | $O(k)$ | 1 | 4 |
| Naive (shift) | $O(w)$ | 4 | 4 |

Where $k$ = popcount, $w$ = word size (e.g., 32 or 64).

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn count_set_bits(mut n: usize) -> usize {
    let mut count = 0;
    while n > 0 {
        n &= n - 1;  // Clear the least significant set bit
        count += 1;
    }
    count
}
```

**Key Implementation Details**:
- Uses `usize` for platform-appropriate sizing
- In-place modification with `mut`
- Simple, readable code

### 5.2 Edge Cases

| Input | Expected Output | Reasoning |
|-------|-----------------|-----------|
| 0 | 0 | No bits set |
| 1 | 1 | Single bit: `0001` |
| `usize::MAX` | 32 or 64 | All bits set (platform-dependent) |
| Power of 2 | 1 | Single bit set (e.g., 16 = `10000`) |
| 0b10101010 | 4 | Alternating pattern |
| 0b11011011 | 6 | Mixed pattern |

### 5.3 Alternative Implementations

**Using Built-in (Optimal)**:
```rust
fn popcount_builtin(n: u32) -> u32 {
    n.count_ones()  // Compiles to POPCNT instruction
}
```

**Parallel Bit Count (SWAR)**:
```rust
fn popcount_swar(mut n: u32) -> u32 {
    n = n - ((n >> 1) & 0x55555555);
    n = (n & 0x33333333) + ((n >> 2) & 0x33333333);
    n = (n + (n >> 4)) & 0x0F0F0F0F;
    n = n + (n >> 8);
    n = n + (n >> 16);
    n & 0x3F
}
```

**Lookup Table**:
```rust
const POPCOUNT_TABLE: [u8; 256] = /* precomputed */;

fn popcount_table(n: u32) -> u32 {
    POPCOUNT_TABLE[(n & 0xFF) as usize] as u32
        + POPCOUNT_TABLE[((n >> 8) & 0xFF) as usize] as u32
        + POPCOUNT_TABLE[((n >> 16) & 0xFF) as usize] as u32
        + POPCOUNT_TABLE[((n >> 24) & 0xFF) as usize] as u32
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Cryptography**
   - Hamming weight is used in side-channel analysis
   - Bit counting in error-correcting codes

2. **Compression Algorithms**
   - Succinct data structures
   - Rank queries in compressed bitmaps

3. **Chess Engines**
   - Bitboards represent piece positions
   - Counting attacks, mobility evaluation

4. **Database Systems**
   - Bitmap indexes
   - Cardinality estimation

5. **Network Programming**
   - Counting subnet bits in CIDR notation
   - Analyzing packet flags

6. **Machine Learning**
   - Binary neural networks
   - Hamming distance calculation

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| [Is Power of Two](is_power_of_two.md) | Power of 2 has popcount = 1 |
| [Hamming Distance](../string/hamming_distance.md) | popcount(a XOR b) |
| [Find Unique Number](find_unique_number.md) | Uses XOR properties |

## 7. Hardware Support

Modern CPUs provide dedicated popcount instructions:

| Architecture | Instruction | Latency |
|--------------|-------------|---------|
| x86 (SSE4.2+) | `POPCNT` | 1-3 cycles |
| ARM (NEON) | `VCNT` | 1-2 cycles |
| RISC-V (Zbb) | `CPOP` | 1 cycle |

Rust's `count_ones()` uses the optimal instruction when available.

## 8. Comparison of Methods

| Method | Time | Space | Best For |
|--------|------|-------|----------|
| Kernighan's | O(k) | O(1) | Sparse bits (few 1s) |
| Built-in POPCNT | O(1) | O(1) | All cases (if available) |
| SWAR (parallel) | O(1) | O(1) | No hardware support |
| Lookup table | O(w/8) | O(256) | Byte-oriented processing |
| Naive shift | O(w) | O(1) | Educational only |

### Performance Benchmark (Typical)

| Method | 1 bit set | 16 bits set | 32 bits set |
|--------|-----------|-------------|-------------|
| Kernighan's | ~5 ns | ~35 ns | ~65 ns |
| POPCNT | ~1 ns | ~1 ns | ~1 ns |
| SWAR | ~3 ns | ~3 ns | ~3 ns |

## 9. Common Pitfalls

### Pitfall 1: Integer Overflow in n-1
```rust
// Safe in Rust due to panic on overflow in debug mode
// But be careful in languages with silent overflow
let n: u32 = 0;
// n - 1 would underflow to u32::MAX in release mode without checks
```

### Pitfall 2: Using Wrong Type Size
```rust
// Bug: Counts only 32 bits of a 64-bit number
fn buggy_popcount(n: u64) -> u32 {
    (n as u32).count_ones()  // Truncates!
}

// Fix: Use correct type
fn correct_popcount(n: u64) -> u32 {
    n.count_ones()
}
```

### Pitfall 3: Not Using Hardware Intrinsics
```rust
// Avoid: Manual implementation when hardware support exists
fn slow_popcount(n: u32) -> u32 {
    // Kernighan's algorithm - unnecessary if POPCNT available
}

// Prefer: Built-in that uses hardware
fn fast_popcount(n: u32) -> u32 {
    n.count_ones()  // Uses POPCNT on supported CPUs
}
```

## 10. References

1. Warren, H. S. (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley. Chapter 5.
2. Kernighan, B. W., & Ritchie, D. M. (1988). *The C Programming Language* (2nd ed.).
3. Anderson, S. E. (2005). *Bit Twiddling Hacks*. Stanford University.
4. Rust Documentation: [`std::primitive::usize::count_ones`](https://doc.rust-lang.org/std/primitive.usize.html#method.count_ones)
