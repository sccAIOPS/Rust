# Sum of Two Integers (Bitwise Addition)

## 1. Overview

This algorithm adds two integers without using the `+` or `-` operators. It relies solely on bitwise operations (XOR and AND) to simulate the addition process, demonstrating how binary arithmetic works at the hardware level.

This technique mirrors how arithmetic logic units (ALUs) in CPUs perform addition.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given two integers $a$ and $b$, compute their sum $a + b$ using only bitwise operations.

### 2.2 Mathematical Model

**Input:** Two integers $a$ and $b$

**Output:** $a + b$

**Key Insight:** Binary addition can be decomposed into:
1. **Sum without carry:** $a \oplus b$ (XOR gives sum where no carry occurs)
2. **Carry bits:** $(a \land b) << 1$ (AND identifies carry positions, shift propagates)

### 2.3 Why This Works

For each bit position:
| $a_i$ | $b_i$ | Sum bit | Carry |
|-------|-------|---------|-------|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

- XOR ($\oplus$) produces the sum bit
- AND ($\land$) produces the carry bit
- The carry must be added to the next position (hence left shift)

### 2.4 Convergence

The algorithm terminates because:
- Each iteration, the carry moves left (higher bit positions)
- Eventually, the carry becomes 0 (no more carries to propagate)
- For $n$-bit integers, at most $n$ iterations

## 3. Algorithm Description

### 3.1 Intuition

Simulate how you add binary numbers by hand:
1. Add corresponding bits (XOR)
2. Calculate carries (AND + shift)
3. Add the carries to the result
4. Repeat until no more carries

### 3.2 Pseudocode

```
function add_two_integers(a, b):
    while b != 0:
        sum_without_carry = a XOR b
        carry = (a AND b) << 1
        a = sum_without_carry
        b = carry
    
    return a
```

### 3.3 Step-by-Step Example

**Input:** `a = 5` (101), `b = 3` (011)

| Iteration | a (binary) | b (binary) | XOR (sum) | AND<<1 (carry) |
|-----------|------------|------------|-----------|----------------|
| 1 | 101 | 011 | 110 | 010 |
| 2 | 110 | 010 | 100 | 100 |
| 3 | 100 | 100 | 000 | 1000 |
| 4 | 000 | 1000 | 1000 | 0000 |

**Result:** `a = 8` (1000), which equals `5 + 3`

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| Best | $O(1)$ | No carries (e.g., $a = 0$) |
| Average | $O(\log(\max(a,b)))$ | Carry propagation |
| Worst | $O(w)$ | Where $w$ is the bit width |

### 4.2 Space Complexity

- **Auxiliary Space:** $O(1)$
- **Total Space:** $O(1)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn add_two_integers(mut a: isize, mut b: isize) -> isize {
    let mut carry;
    
    while b != 0 {
        let sum = a ^ b;           // Sum without carry
        carry = (a & b) << 1;      // Carry bits shifted left
        a = sum;
        b = carry;
    }
    
    a
}
```

**Key patterns:**
- Uses `isize` for signed integer support
- Mutable parameters to avoid extra variables
- Works correctly with negative numbers (two's complement)

### 5.2 Handling Negative Numbers

The algorithm works for negative numbers because:
- Two's complement representation is used
- XOR and AND operations work correctly on all bit patterns
- The sign bit is handled like any other bit

### 5.3 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| (0, 0) | 0 | Zero + Zero |
| (0, 42) | 42 | Zero + Any |
| (-1, -1) | -2 | Negative + Negative |
| (-10, 6) | -4 | Mixed signs |
| (MAX, 1) | Overflow | Platform-dependent |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Hardware Design:** Understanding ALU operations
2. **Embedded Systems:** Systems without hardware multiplication
3. **Interview Questions:** Classic bit manipulation problem
4. **Education:** Teaching binary arithmetic fundamentals
5. **Cryptography:** Low-level operations in cipher implementations

### 6.2 Related Operations

| Operation | Implementation |
|-----------|----------------|
| Subtraction | `add(a, add(~b, 1))` (add with negation) |
| Increment | `add(a, 1)` |
| Decrement | `add(a, -1)` |
| Negate | `add(~a, 1)` |

### 6.3 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Half Adder | Single-bit version |
| Full Adder | With carry-in |
| Carry-Lookahead Adder | Optimized hardware version |

## 7. References

1. Patterson, D. & Hennessy, J. "Computer Organization and Design."
2. Warren, H. "Hacker's Delight." Addison-Wesley, 2012.
3. Mano, M. "Digital Logic and Computer Design."
