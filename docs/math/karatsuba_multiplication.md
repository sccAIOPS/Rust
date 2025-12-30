# Karatsuba Multiplication

## 1. Overview

**Karatsuba multiplication** is a divide-and-conquer algorithm for multiplying large numbers. It was the first algorithm to demonstrate that multiplication can be done in sub-quadratic time, achieving $O(n^{1.585})$ complexity.

**Historical Context**: Discovered by Anatoly Karatsuba in 1960, disproving Kolmogorov's conjecture that $O(n^2)$ was optimal.

**File**: `src/math/karatsuba_multiplication.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given two $n$-digit numbers $x$ and $y$, compute their product $x \times y$.

### 2.2 Key Insight

For numbers split in half:
- $x = x_1 \cdot B^m + x_0$ (high and low parts)
- $y = y_1 \cdot B^m + y_0$

Standard expansion requires 4 multiplications:
$$xy = x_1 y_1 \cdot B^{2m} + (x_1 y_0 + x_0 y_1) \cdot B^m + x_0 y_0$$

**Karatsuba's trick**: Reduce to 3 multiplications:
$$z_0 = x_0 \cdot y_0$$
$$z_2 = x_1 \cdot y_1$$
$$z_1 = (x_0 + x_1)(y_0 + y_1) - z_0 - z_2$$

Then: $xy = z_2 \cdot B^{2m} + z_1 \cdot B^m + z_0$

### 2.3 Algebraic Verification

$(x_0 + x_1)(y_0 + y_1) = x_0 y_0 + x_0 y_1 + x_1 y_0 + x_1 y_1$

Therefore:
$(x_0 + x_1)(y_0 + y_1) - z_0 - z_2 = x_0 y_1 + x_1 y_0 = z_1$ ✓

## 3. Algorithm Description

### 3.1 Intuition

1. If numbers are small, use naive multiplication
2. Otherwise, split each number into high and low halves
3. Recursively compute three products
4. Combine results with appropriate shifts

### 3.2 Pseudocode

```
function karatsuba(x, y):
    if x < 10 or y < 10:
        return x × y
    
    n ← max(digits(x), digits(y))
    m ← ceil(n / 2)
    
    # Split: x = x1 × 10^m + x0
    x1, x0 ← split(x, m)
    y1, y0 ← split(y, m)
    
    # Three recursive multiplications
    z0 ← karatsuba(x0, y0)
    z2 ← karatsuba(x1, y1)
    z1 ← karatsuba(x0 + x1, y0 + y1) - z0 - z2
    
    return z2 × 10^(2m) + z1 × 10^m + z0
```

### 3.3 Step-by-Step Example

Multiply $1234 \times 5678$:

**Step 1**: Split (m = 2)
- $x = 1234 = 12 \cdot 100 + 34$, so $x_1 = 12$, $x_0 = 34$
- $y = 5678 = 56 \cdot 100 + 78$, so $y_1 = 56$, $y_0 = 78$

**Step 2**: Compute products
- $z_0 = 34 \times 78 = 2652$
- $z_2 = 12 \times 56 = 672$
- $z_1 = (34 + 12) \times (78 + 56) - z_0 - z_2$
      $= 46 \times 134 - 2652 - 672$
      $= 6164 - 3324 = 2840$

**Step 3**: Combine
$= 672 \cdot 10000 + 2840 \cdot 100 + 2652$
$= 6720000 + 284000 + 2652$
$= 7006652$

**Verification**: $1234 \times 5678 = 7006652$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

$$T(n) = 3T(n/2) + O(n)$$

By Master theorem: $T(n) = O(n^{\log_2 3}) = O(n^{1.585})$

### 4.2 Space Complexity

$O(n)$ for storing intermediate results.

### 4.3 Comparison

| Method | Time | n = 1000 digits |
|--------|------|-----------------|
| Schoolbook | O(n²) | 1,000,000 ops |
| Karatsuba | O(n^1.585) | ~63,000 ops |
| FFT-based | O(n log n) | ~10,000 ops |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
use std::cmp::max;
const TEN: i128 = 10;

pub fn multiply(num1: i128, num2: i128) -> i128 {
    _multiply(num1, num2)
}

fn _multiply(num1: i128, num2: i128) -> i128 {
    // Base case: small numbers
    if num1 < 10 || num2 < 10 {
        return num1 * num2;
    }
    
    let mut num1_str = num1.to_string();
    let mut num2_str = num2.to_string();

    let n = max(num1_str.len(), num2_str.len());
    num1_str = normalize(num1_str, n);
    num2_str = normalize(num2_str, n);

    let a = &num1_str[0..n / 2];
    let b = &num1_str[n / 2..];
    let c = &num2_str[0..n / 2];
    let d = &num2_str[n / 2..];

    let ac = _multiply(a.parse().unwrap(), c.parse().unwrap());
    let bd = _multiply(b.parse().unwrap(), d.parse().unwrap());
    let a_b: i128 = a.parse::<i128>().unwrap() + b.parse::<i128>().unwrap();
    let c_d: i128 = c.parse::<i128>().unwrap() + d.parse::<i128>().unwrap();
    let ad_bc = _multiply(a_b, c_d) - (ac + bd);

    let m = n / 2 + n % 2;
    (TEN.pow(2 * m as u32) * ac) + (TEN.pow(m as u32) * ad_bc) + bd
}

fn normalize(mut a: String, n: usize) -> String {
    let padding = n.saturating_sub(a.len());
    a.insert_str(0, &"0".repeat(padding));
    a
}
```

### 5.2 Key Design Choices

1. **String representation**: For splitting by digits
2. **Base 10**: Human-readable; base 2^32 more efficient
3. **Padding**: Ensures equal-length operands

### 5.3 Edge Cases

| Input | Handling |
|-------|----------|
| n₁ or n₂ < 10 | Base case: naive multiplication |
| Different lengths | Pad shorter number with zeros |
| Zero inputs | Works correctly |

## 6. Applications

### 6.1 Use Cases

1. **Cryptography**: RSA key operations (1024+ bit numbers)
2. **Arbitrary precision arithmetic**: BigInteger libraries
3. **Scientific computing**: High-precision calculations
4. **Competitive programming**: Large number multiplication

### 6.2 When to Use

| Number Size | Best Algorithm |
|-------------|----------------|
| < 64 bits | Hardware multiply |
| 64-500 bits | Karatsuba |
| > 500 bits | Toom-Cook or FFT |

## 7. Variants and Extensions

### 7.1 Toom-Cook (Toom-3)

Split into 3 parts: $T(n) = 5T(n/3) + O(n) = O(n^{1.465})$

### 7.2 Schönhage-Strassen

FFT-based: $O(n \log n \log \log n)$

### 7.3 Harvey-van der Hoeven (2019)

Achieves $O(n \log n)$ - theoretically optimal!

## 8. Related Algorithms

| Algorithm | Complexity | Notes |
|-----------|------------|-------|
| Schoolbook | O(n²) | Simple, cache-friendly for small n |
| Karatsuba | O(n^1.585) | Good balance of simplicity/speed |
| [FFT Multiplication](fast_fourier_transform.md) | O(n log n) | Best for very large n |
| Toom-Cook | O(n^1.465) | Between Karatsuba and FFT |

## 9. Optimization Tips

1. **Choose good threshold**: Switch to naive at ~20-100 digits
2. **Use base 2^32**: More efficient than decimal
3. **Minimize allocations**: Reuse buffers
4. **Consider Toom-3**: For medium-large numbers

## 10. References

1. Karatsuba, A. & Ofman, Y. "Multiplication of Multidigit Numbers on Automata" (1962)
2. Knuth, D. E. "The Art of Computer Programming, Vol. 2", Section 4.3.3
3. [Wikipedia: Karatsuba Algorithm](https://en.wikipedia.org/wiki/Karatsuba_algorithm)
