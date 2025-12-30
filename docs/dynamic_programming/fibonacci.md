# Fibonacci Sequence

## 1. Overview

The Fibonacci sequence is one of the most famous sequences in mathematics, appearing in nature, art, and computer science. Each number is the sum of the two preceding ones, starting from 0 and 1.

This implementation provides **8 different methods** to compute Fibonacci numbers, each with distinct trade-offs between time complexity, space complexity, and practical performance.

**File**: `src/dynamic_programming/fibonacci.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Compute $F(n)$, the $n$-th Fibonacci number.

### 2.2 Mathematical Model

**Classical Definition** (used by most functions):
$$F(0) = 0, \quad F(1) = 1$$
$$F(n) = F(n-1) + F(n-2) \quad \text{for } n \geq 2$$

**Combinatorial Definition** (used by `fibonacci` and `recursive_fibonacci`):
$$F(0) = F(1) = 1$$
$$F(n) = F(n-1) + F(n-2) \quad \text{for } n \geq 2$$

**Closed-Form (Binet's Formula)**:
$$F(n) = \frac{\phi^n - \psi^n}{\sqrt{5}}$$
where $\phi = \frac{1 + \sqrt{5}}{2}$ (golden ratio) and $\psi = \frac{1 - \sqrt{5}}{2}$

### 2.3 Key Properties

1. **Cassini's Identity**: $F(n-1) \cdot F(n+1) - F(n)^2 = (-1)^n$
2. **Addition Formula**: $F(m+n) = F(m) \cdot F(n+1) + F(m-1) \cdot F(n)$
3. **GCD Property**: $\gcd(F(m), F(n)) = F(\gcd(m, n))$
4. **Divisibility**: $F(n) | F(mn)$ for all positive integers $m, n$

### 2.4 Matrix Representation

$$\begin{pmatrix} F(n+1) \\ F(n) \end{pmatrix} = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^n \begin{pmatrix} 1 \\ 0 \end{pmatrix}$$

This enables $O(\log n)$ computation via matrix exponentiation.

## 3. Algorithm Description

### 3.1 Available Implementations

| Function | Approach | Time | Space | Definition |
|----------|----------|------|-------|------------|
| `fibonacci` | Iterative | O(n) | O(1) | Combinatorial |
| `recursive_fibonacci` | Tail-recursive | O(n) | O(n)* | Combinatorial |
| `classical_fibonacci` | Fast doubling | O(log n) | O(log n) | Classical |
| `logarithmic_fibonacci` | Fast doubling | O(log n) | O(log n) | Classical |
| `memoized_fibonacci` | Memoization | O(n) | O(n) | Classical |
| `matrix_fibonacci` | Matrix exp. | O(n)** | O(n) | Classical |
| `binary_lifting_fibonacci` | Binary lifting | O(log n) | O(1) | Classical |
| `nth_fibonacci_number_modulo_m` | Pisano period | O(m²) | O(m²) | Classical |

*Stack space for recursion; **Current impl is O(n), can be O(log n)

### 3.2 Iterative Approach (Recommended for General Use)

```
FUNCTION fibonacci(n)
    a ← 0
    b ← 1
    FOR i ← 0 TO n-1 DO
        c ← a + b
        a ← b
        b ← c
    END FOR
    RETURN b
END FUNCTION
```

### 3.3 Binary Lifting (Fast Doubling) Approach

Based on these identities:
- $F(2k) = F(k) \cdot (2F(k+1) - F(k))$
- $F(2k+1) = F(k+1)^2 + F(k)^2$

```
FUNCTION binary_lifting_fibonacci(n)
    state ← (0, 1)  // (F(0), F(1))
    FOR i ← (bits of n from MSB to LSB) DO
        // Double: compute F(2k), F(2k+1) from F(k), F(k+1)
        state ← (state.0 * (2*state.1 - state.0),
                 state.0² + state.1²)
        IF bit i of n is 1 THEN
            state ← (state.1, state.0 + state.1)
        END IF
    END FOR
    RETURN state.0
END FUNCTION
```

### 3.4 Step-by-Step Example

Computing `fibonacci(5)` (iterative):

| Step | a | b | c |
|------|---|---|---|
| Init | 0 | 1 | - |
| i=0  | 1 | 1 | 1 |
| i=1  | 1 | 2 | 2 |
| i=2  | 2 | 3 | 3 |
| i=3  | 3 | 5 | 5 |
| i=4  | 5 | 8 | 8 |

Result: `b = 8` ✓ (Note: combinatorial definition, so F(5) = 8)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Method | Best | Average | Worst |
|--------|------|---------|-------|
| Iterative | O(n) | O(n) | O(n) |
| Tail-recursive | O(n) | O(n) | O(n) |
| Fast doubling | O(log n) | O(log n) | O(log n) |
| Memoization | O(1)* | O(n) | O(n) |
| Matrix power | O(log n)** | O(log n)** | O(log n)** |

*With warm cache; **With proper implementation

### 4.2 Space Complexity

| Method | Auxiliary Space | Notes |
|--------|----------------|-------|
| Iterative | O(1) | Best for memory |
| Tail-recursive | O(n) | Stack frames |
| Fast doubling | O(log n) | Recursion depth |
| Memoization | O(n) | HashMap storage |
| Matrix power | O(n) | Matrix storage |
| Binary lifting | O(1) | Best for large n |

### 4.3 Derivation

**Iterative**: Constant work per iteration, n iterations → O(n)

**Fast Doubling**: Each recursion halves the problem size:
$$T(n) = T(n/2) + O(1) \Rightarrow T(n) = O(\log n)$$

### 4.4 Overflow Warning

⚠️ **Warning**: `u128` overflows at n=186 for classical definition.

```rust
fibonacci(185) // OK: 127127879743834334146972278486287885163
fibonacci(186) // OVERFLOW!
```

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Type Selection**:
```rust
// Uses u128 for maximum range before overflow
pub fn fibonacci(n: u32) -> u128
```

**Tail Recursion**:
```rust
fn _recursive_fibonacci(n: u32, previous: u128, current: u128) -> u128 {
    if n == 0 { current }
    else { _recursive_fibonacci(n - 1, current, current + previous) }
}
```

Note: Rust doesn't guarantee tail-call optimization, so this may still use O(n) stack space.

**HashMap Memoization**:
```rust
fn _memoized_fibonacci(n: u32, cache: &mut HashMap<u32, u128>) -> u128 {
    if let Some(&f) = cache.get(&n) {
        return f;
    }
    // Compute and cache...
}
```

### 5.2 Edge Cases

| Case | Classical | Combinatorial |
|------|-----------|---------------|
| n=0 | 0 | 1 |
| n=1 | 1 | 1 |
| n=2 | 1 | 2 |
| n=186 | overflow | overflow |

### 5.3 Special Functions

**Pisano Period** (`nth_fibonacci_number_modulo_m`):
- Computes F(n) mod m efficiently
- Uses the periodicity of Fibonacci sequence modulo m
- Pisano period π(m) ≤ 6m

**Sum of Fibonacci** (`last_digit_of_the_sum_of_nth_fibonacci_number`):
- Uses: $\sum_{i=0}^{n} F(i) = F(n+2) - 1$
- Pisano period of 10 is 60

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Algorithm Analysis**: Analyzing recursive algorithm complexity
2. **Data Structures**: Fibonacci heaps, search trees
3. **Dynamic Programming Education**: Foundation for understanding DP
4. **Pseudo-Random Number Generation**: Linear feedback shift registers

### 6.2 Nature and Science

1. **Phyllotaxis**: Leaf arrangement in plants
2. **Population Dynamics**: Rabbit population model (original problem)
3. **Financial Markets**: Fibonacci retracement levels
4. **Art and Architecture**: Golden ratio aesthetics

### 6.3 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Lucas Numbers | $L(n) = F(n-1) + F(n+1)$ |
| Tribonacci | 3-term recurrence |
| Matrix Exponentiation | General linear recurrence |

## 7. Performance Comparison

```
Benchmark: Computing F(100,000)

Method                    Time
──────────────────────────────────
binary_lifting           ~1 μs
logarithmic_fibonacci    ~2 μs  
classical_fibonacci      ~3 μs
fibonacci (iterative)    ~50 μs
memoized_fibonacci       ~100 μs
```

### Recommendation Matrix

| Use Case | Recommended Method |
|----------|-------------------|
| Single small n (<1000) | `fibonacci` (iterative) |
| Single large n | `binary_lifting_fibonacci` |
| Multiple queries | `memoized_fibonacci` |
| Modular arithmetic | `nth_fibonacci_number_modulo_m` |
| Educational | `recursive_fibonacci` |

## 8. References

1. Knuth, D. E. (1997). *The Art of Computer Programming, Vol. 1*
2. [OEIS A000045 - Fibonacci numbers](https://oeis.org/A000045)
3. [Wikipedia - Fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number)
4. [Fast Doubling Method](https://funloop.org/post/2017-04-14-computing-fibonacci-numbers.html)
