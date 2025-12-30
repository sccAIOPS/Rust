# Count Trailing Zeros

## 1. Overview

The **Count Trailing Zeros** algorithm determines the number of consecutive zero bits at the least significant end of a binary number. This operation is also known as finding the **count trailing zeros (CTZ)** or **number of trailing zeros (NTZ)**. It's closely related to finding the position of the rightmost set bit.

### Historical Context

The trailing zeros count became especially important with the introduction of the `TZCNT` (trailing zero count) instruction in modern x86 processors (BMI1 instruction set). Earlier processors used the `BSF` (bit scan forward) instruction for similar purposes.

### Key Insight

Trailing zeros indicate the highest power of 2 that divides the number. For example, if a number has 3 trailing zeros, it's divisible by $2^3 = 8$.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: A non-negative integer $n$  
**Output**: The number of trailing zero bits in the binary representation of $n$

Formally, for $n > 0$: Find the largest $k$ such that $2^k$ divides $n$, i.e., $n \mod 2^k = 0$.

For $n = 0$: Return 0 (or the word size, depending on convention).

### 2.2 Mathematical Model

#### Binary Structure Analysis

For any integer $n > 0$, we can write:
$$n = m \cdot 2^k$$

where $m$ is odd and $k$ is the number of trailing zeros.

| $n$ | Binary | Trailing Zeros | Factorization |
|-----|--------|----------------|---------------|
| 1 | `0001` | 0 | $1 \cdot 2^0$ |
| 2 | `0010` | 1 | $1 \cdot 2^1$ |
| 4 | `0100` | 2 | $1 \cdot 2^2$ |
| 6 | `0110` | 1 | $3 \cdot 2^1$ |
| 12 | `1100` | 2 | $3 \cdot 2^2$ |
| 16 | `10000` | 4 | $1 \cdot 2^4$ |
| 24 | `11000` | 3 | $3 \cdot 2^3$ |

#### Relationship to Rightmost Set Bit

The trailing zeros count equals the bit position (0-indexed) of the rightmost set bit:
$$\text{ctz}(n) = \lfloor \log_2(n \land -n) \rfloor$$

where $n \land -n$ isolates the rightmost set bit.

### 2.3 Correctness Proof

**Theorem**: For $n > 0$, `n.trailing_zeros()` returns the position of the rightmost 1-bit.

**Proof by construction**:
1. Let $n$ have binary representation $b_{w-1}b_{w-2}...b_k b_{k-1}...b_0$
2. If $b_i = 1$ and $b_j = 0$ for all $j < i$, then $i$ is the position of rightmost 1-bit
3. The number of trailing zeros is exactly $i$
4. This equals $\lfloor \log_2(2^i) \rfloor = i$ $\square$

## 3. Algorithm Description

### 3.1 Intuition

Imagine reading the binary number from right to left, counting zeros until you hit the first one. That count is the number of trailing zeros.

### 3.2 Pseudocode

**Method 1: Using Built-in (Optimal)**
```
function count_trailing_zeros(n):
    if n == 0:
        return 0  // or word_size, depending on convention
    return n.trailing_zeros()  // Hardware instruction
```

**Method 2: Using n & -n Trick**
```
function count_trailing_zeros_bitwise(n):
    if n == 0:
        return 0
    
    rightmost_bit = n & (-n)  // Isolate rightmost set bit
    return log2(rightmost_bit)  // Position of that bit
```

**Method 3: Iterative**
```
function count_trailing_zeros_iterative(n):
    if n == 0:
        return 0
    
    count = 0
    while (n & 1) == 0:
        count += 1
        n >>= 1
    return count
```

### 3.3 Step-by-Step Example

**Example**: Count trailing zeros of 36

```
n = 36 = 100100 (binary)

Method 1: Built-in
36.trailing_zeros() = 2

Method 2: n & -n trick
Step 1: Compute -n (two's complement)
-36 in binary (assuming 8 bits): 11011100

Step 2: n & (-n)
  00100100
& 11011100
----------
  00000100 = 4

Step 3: log2(4) = 2

Result: 2 trailing zeros ✓
```

**Verification**: $36 = 9 \times 4 = 9 \times 2^2$, so 36 has 2 trailing zeros.

## 4. Complexity Analysis

### 4.1 Time Complexity

| Method | Complexity | Explanation |
|--------|------------|-------------|
| Built-in (`trailing_zeros()`) | $O(1)$ | Single CPU instruction (TZCNT/BSF) |
| `n & -n` + log2 | $O(1)$ | Constant operations |
| Iterative | $O(k)$ | Where $k$ is the number of trailing zeros |

### 4.2 Space Complexity

| Metric | Complexity |
|--------|------------|
| Auxiliary space | $O(1)$ |
| Stack space | $O(1)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
/// Primary implementation using built-in
pub fn binary_count_trailing_zeros(num: u64) -> u32 {
    if num == 0 {
        return 0;
    }
    num.trailing_zeros()
}

/// Alternative using bit manipulation
pub fn binary_count_trailing_zeros_bitwise(num: u64) -> u32 {
    if num == 0 {
        return 0;
    }
    let rightmost_set_bit = num & (num.wrapping_neg());
    63 - rightmost_set_bit.leading_zeros()
}
```

**Key Implementation Details**:
- Uses `wrapping_neg()` for safe two's complement negation
- `trailing_zeros()` compiles to efficient CPU instruction
- Handles zero specially (returns 0 in this implementation)

### 5.2 Edge Cases

| Input | Expected Output | Reasoning |
|-------|-----------------|-----------|
| 0 | 0 | No set bits (convention varies) |
| 1 | 0 | `1` = `...0001`, rightmost bit is position 0 |
| 2 | 1 | `2` = `...0010` |
| 16 | 4 | `16` = `...10000` |
| 25 | 0 | `25` = `11001`, odd number |
| $2^{32}$ | 32 | Single bit at position 32 |

### 5.3 Convention for Zero

Different systems handle zero differently:

| Convention | `ctz(0)` Result | Used By |
|------------|-----------------|---------|
| Return 0 | 0 | This implementation |
| Return word size | 32 or 64 | GCC `__builtin_ctz` (undefined behavior) |
| Return word size (defined) | 32 or 64 | TZCNT instruction |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Finding Divisibility by Powers of 2**
   ```rust
   // Check if n is divisible by 8
   let divisible_by_8 = n.trailing_zeros() >= 3;
   ```

2. **Binary GCD Algorithm (Stein's Algorithm)**
   - Remove common factors of 2 from both numbers
   - Uses trailing zeros to determine shift amount

3. **Memory Alignment Checking**
   ```rust
   // Check if address is 16-byte aligned
   let is_aligned = (address & 0xF) == 0;
   // Or: address.trailing_zeros() >= 4
   ```

4. **Efficient Modulo for Powers of 2**
   - If `n` has `k` trailing zeros, `n` divides evenly by `2^k`

5. **Position Finding in Bit Vectors**
   - Find the first set bit in a bitmap
   - Used in memory allocators and schedulers

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| [Rightmost Set Bit](rightmost_set_bit.md) | `n & -n` isolates it |
| [Count Set Bits](counting_bits.md) | Related counting operation |
| [Is Power of Two](is_power_of_two.md) | Power of 2 has (word_size - 1) trailing zeros |

## 7. Hardware Support

Modern CPUs provide dedicated instructions:

| Architecture | Instruction | Behavior for 0 |
|--------------|-------------|----------------|
| x86 (BMI1) | `TZCNT` | Returns operand size |
| x86 (Legacy) | `BSF` | Undefined (flags set) |
| ARM | `RBIT` + `CLZ` | Equivalent operation |
| RISC-V (Zbb) | `CTZ` | Returns XLEN |

Rust's `trailing_zeros()` uses the optimal instruction for each platform.

## 8. Comparison of Methods

| Method | Time | Portability | Notes |
|--------|------|-------------|-------|
| `trailing_zeros()` | O(1) | High | Best choice |
| `n & -n` + leading_zeros | O(1) | High | Good alternative |
| De Bruijn sequence | O(1) | High | Table lookup |
| Iterative | O(k) | High | Simple but slow |
| Binary search | O(log w) | High | Middle ground |

### De Bruijn Method

```rust
const DEBRUIJN: u32 = 0x077CB531;
const LOOKUP: [u32; 32] = [
    0, 1, 28, 2, 29, 14, 24, 3, 30, 22, 20, 15, 25, 17, 4, 8,
    31, 27, 13, 23, 21, 19, 16, 7, 26, 12, 18, 6, 11, 5, 10, 9
];

fn ctz_debruijn(n: u32) -> u32 {
    if n == 0 { return 32; }
    let idx = (((n & n.wrapping_neg()).wrapping_mul(DEBRUIJN)) >> 27) as usize;
    LOOKUP[idx]
}
```

## 9. Common Pitfalls

### Pitfall 1: Undefined Behavior for Zero
```rust
// Some C/C++ builtins have undefined behavior for 0
// Rust's trailing_zeros() is well-defined: returns bit width
let zeros = 0u32.trailing_zeros(); // Returns 32
```

### Pitfall 2: Confusing with Leading Zeros
```rust
let n: u32 = 8;  // Binary: 00000000000000000000000000001000
n.trailing_zeros()  // 3 (zeros after the 1)
n.leading_zeros()   // 28 (zeros before the 1)
```

### Pitfall 3: Sign Extension in Negation
```rust
// Use wrapping_neg() to avoid surprises
let n: u64 = 1;
let neg = n.wrapping_neg();  // Safe
// let neg = -n;  // Works but less explicit
```

## 10. References

1. Warren, H. S. (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley. Chapter 5.
2. Anderson, S. E. (2005). *Bit Twiddling Hacks*. Stanford University.
3. Intel® 64 and IA-32 Architectures Software Developer's Manual, Vol. 2.
4. Rust Documentation: [`std::primitive::u64::trailing_zeros`](https://doc.rust-lang.org/std/primitive.u64.html#method.trailing_zeros)
