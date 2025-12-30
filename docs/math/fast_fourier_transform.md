# Fast Fourier Transform (FFT)

## 1. Overview

The **Fast Fourier Transform (FFT)** is an algorithm to compute the Discrete Fourier Transform (DFT) in $O(n \log n)$ time instead of $O(n^2)$. It's one of the most important algorithms in computational science, with applications in signal processing, polynomial multiplication, and image processing.

**Historical Context**: Discovered by Cooley and Tukey in 1965, though Gauss had described a similar method in 1805.

**File**: `src/math/fast_fourier_transform.rs`

## 2. Mathematical Foundation

### 2.1 Discrete Fourier Transform

For a sequence $x_0, x_1, \ldots, x_{n-1}$, the DFT is:

$$X_k = \sum_{j=0}^{n-1} x_j \cdot e^{-2\pi i \cdot jk/n} = \sum_{j=0}^{n-1} x_j \cdot \omega_n^{jk}$$

where $\omega_n = e^{-2\pi i/n}$ is the $n$-th root of unity.

### 2.2 Inverse DFT

$$x_j = \frac{1}{n} \sum_{k=0}^{n-1} X_k \cdot e^{2\pi i \cdot jk/n}$$

### 2.3 Key Properties of Roots of Unity

1. $\omega_n^n = 1$
2. $\omega_n^{n/2} = -1$
3. $\omega_{2n}^{2k} = \omega_n^k$ (Halving lemma)
4. $\omega_n^{k+n/2} = -\omega_n^k$ (Cancellation lemma)

### 2.4 Cooley-Tukey Decomposition

Split into even and odd indices:
$$X_k = \sum_{j=0}^{n/2-1} x_{2j} \omega_n^{2jk} + \sum_{j=0}^{n/2-1} x_{2j+1} \omega_n^{(2j+1)k}$$

$$= E_k + \omega_n^k \cdot O_k$$

where $E_k$ and $O_k$ are DFTs of even and odd elements.

## 3. Algorithm Description

### 3.1 Intuition

1. If $n = 1$, DFT is the identity
2. Otherwise:
   - Separate even-indexed and odd-indexed elements
   - Recursively compute DFT of each half
   - Combine using "butterfly" operations

### 3.2 Pseudocode (Recursive)

```
function fft(x):
    n ← length(x)
    if n = 1:
        return x
    
    ω ← e^(-2πi/n)
    even ← fft(x[0], x[2], x[4], ...)
    odd ← fft(x[1], x[3], x[5], ...)
    
    for k from 0 to n/2 - 1:
        t ← ω^k × odd[k]
        X[k] ← even[k] + t
        X[k + n/2] ← even[k] - t
    
    return X
```

### 3.3 Iterative (Cooley-Tukey)

```
function fft_iterative(x):
    n ← length(x)
    x ← bit_reverse_copy(x)
    
    for s from 1 to log(n):
        m ← 2^s
        ωm ← e^(-2πi/m)
        for k from 0 to n-1 step m:
            ω ← 1
            for j from 0 to m/2 - 1:
                t ← ω × x[k + j + m/2]
                u ← x[k + j]
                x[k + j] ← u + t
                x[k + j + m/2] ← u - t
                ω ← ω × ωm
    
    return x
```

### 3.4 Butterfly Operation

The core operation combines two elements:

```
       u ────┬──── u + t
             │
    twiddle  ×
             │
       v ────┴──── u - t
```

## 4. Complexity Analysis

### 4.1 Time Complexity

$$T(n) = 2T(n/2) + O(n) = O(n \log n)$$

### 4.2 Space Complexity

- In-place: O(1) extra (iterative)
- Recursive: O(log n) for stack

### 4.3 Comparison

| Method | Time | Multiplications |
|--------|------|-----------------|
| Naive DFT | O(n²) | n² |
| FFT | O(n log n) | (n/2) log n |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
#[derive(Clone, Copy, Debug)]
pub struct Complex64 {
    pub re: f64,
    pub im: f64,
}

impl Complex64 {
    pub fn new(re: f64, im: f64) -> Self {
        Self { re, im }
    }
}

pub fn fast_fourier_transform(input: &[f64], input_permutation: &[usize]) -> Vec<Complex64> {
    let n = input.len();
    let mut result = Vec::with_capacity(n);
    
    // Bit-reverse permutation
    for position in input_permutation {
        result.push(Complex64::new(input[*position], 0.0));
    }
    
    let mut segment_length = 1;
    while segment_length < n {
        segment_length <<= 1;
        let angle = -2.0 * std::f64::consts::PI / (segment_length as f64);
        let wlen = Complex64::new(angle.cos(), angle.sin());
        
        for segment_start in (0..n).step_by(segment_length) {
            let mut w = Complex64::new(1.0, 0.0);
            for i in 0..(segment_length / 2) {
                let a = result[segment_start + i];
                let b = result[segment_start + i + segment_length / 2] * w;
                result[segment_start + i] = a + b;
                result[segment_start + i + segment_length / 2] = a - b;
                w *= wlen;
            }
        }
    }
    result
}
```

### 5.2 Bit-Reversal Permutation

```rust
pub fn fast_fourier_transform_input_permutation(length: usize) -> Vec<usize> {
    let mut result: Vec<usize> = (0..length).collect();
    let mut reverse = 0;
    
    for position in 1..length {
        let mut bit = length >> 1;
        while bit & reverse != 0 {
            reverse ^= bit;
            bit >>= 1;
        }
        reverse ^= bit;
        if position < reverse {
            result.swap(position, reverse);
        }
    }
    result
}
```

### 5.3 Requirements and Edge Cases

| Requirement | Handling |
|-------------|----------|
| n must be power of 2 | Pad with zeros |
| Floating point precision | Use f64 |
| Numeric stability | Careful ordering |

## 6. Applications

### 6.1 Polynomial Multiplication

Multiply two polynomials of degree $n-1$ in $O(n \log n)$:

1. Evaluate both at $n$ points (FFT)
2. Multiply point-wise: O(n)
3. Interpolate result (inverse FFT)

```rust
fn multiply_polynomials(a: &[f64], b: &[f64]) -> Vec<f64> {
    let n = (a.len() + b.len() - 1).next_power_of_two();
    // ... pad, FFT, multiply, inverse FFT
}
```

### 6.2 Signal Processing

- Frequency analysis
- Filtering
- Convolution
- Spectral analysis

### 6.3 Other Applications

- Image compression (JPEG)
- Audio processing (MP3)
- Fast string matching
- Large integer multiplication

## 7. Variants

### 7.1 Number Theoretic Transform (NTT)

Uses modular arithmetic instead of complex numbers:
- Avoids floating-point errors
- Used in cryptography
- Requires prime modulus

### 7.2 Inverse FFT

Same algorithm with:
- $\omega_n^{-1}$ instead of $\omega_n$
- Divide final result by $n$

## 8. Related Algorithms

| Algorithm | Relation |
|-----------|----------|
| [Karatsuba Multiplication](karatsuba_multiplication.md) | Faster for smaller inputs |
| DFT | What FFT computes |
| NTT | Integer variant |
| DCT | Used in JPEG |

## 9. Performance Tips

1. **Power of 2 length**: Required for standard FFT
2. **Cache efficiency**: Use iterative implementation
3. **SIMD optimization**: Vectorize butterfly operations
4. **Precompute twiddle factors**: Avoid recomputing $\omega_n^k$

## 10. References

1. Cooley, J. W. & Tukey, J. W. "An Algorithm for the Machine Calculation of Complex Fourier Series" (1965)
2. Cormen, T. H. et al. "Introduction to Algorithms", Chapter 30
3. [Wikipedia: Fast Fourier Transform](https://en.wikipedia.org/wiki/Fast_Fourier_transform)
