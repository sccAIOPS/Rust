# Fast Factorial (Borwein's Algorithm)

## 1. Overview

The Fast Factorial algorithm, developed by **Peter Borwein in 1985**, is an asymptotically faster method for computing the factorial of large numbers compared to the naive iterative approach. This algorithm leverages prime factorization and binary representation to reduce the complexity of factorial computation from linear to logarithmic in the number of multiplications.

The factorial function $n!$ is fundamental in mathematics, appearing in combinatorics, probability theory, calculus, and many areas of computer science. While small factorials can be computed quickly, calculating factorials of large numbers (e.g., $1000!$ or $10000!$) requires efficient algorithms due to the explosive growth of the result.

**Historical Context**: Borwein's algorithm was published in the paper "The Complexity of Computing the Factorial" in the *Journal of Algorithms* (1985). It was one of the first algorithms to achieve sub-quadratic complexity for factorial computation by exploiting the structure of prime factorization.

**Key Insight**: Instead of multiplying all numbers from 1 to $n$, the algorithm:
1. Finds all primes $p \leq n$
2. Computes the exact power of each prime in $n!$ using Legendre's formula
3. Groups multiplications using binary representation to minimize operations

## 2. Mathematical Foundation

### 2.1 Problem Definition

Compute the factorial of a non-negative integer $n$:

$$n! = \prod_{i=1}^{n} i = 1 \times 2 \times 3 \times \cdots \times n$$

With the convention that $0! = 1$ and $1! = 1$.

**Formal Definition**:
- **Input**: Non-negative integer $n \in \mathbb{N}$
- **Output**: $n!$ as a big integer (arbitrary precision)
- **Constraint**: Must handle very large $n$ (e.g., $n \geq 10000$) efficiently

### 2.2 Mathematical Model

#### 2.2.1 Prime Factorization of Factorials

By the Fundamental Theorem of Arithmetic, every factorial can be uniquely expressed as a product of prime powers:

$$n! = \prod_{p \text{ prime}, p \leq n} p^{e_p(n)}$$

where $e_p(n)$ is the exponent of prime $p$ in the factorization of $n!$.

#### 2.2.2 Legendre's Formula

The exponent $e_p(n)$ is given by **Legendre's formula**:

$$e_p(n) = \sum_{i=1}^{\infty} \left\lfloor \frac{n}{p^i} \right\rfloor = \left\lfloor \frac{n}{p} \right\rfloor + \left\lfloor \frac{n}{p^2} \right\rfloor + \left\lfloor \frac{n}{p^3} \right\rfloor + \cdots$$

This sum is finite because $\left\lfloor \frac{n}{p^i} \right\rfloor = 0$ for $p^i > n$.

**Example**: For $n = 10$ and $p = 2$:

$$e_2(10) = \left\lfloor \frac{10}{2} \right\rfloor + \left\lfloor \frac{10}{4} \right\rfloor + \left\lfloor \frac{10}{8} \right\rfloor = 5 + 2 + 1 = 8$$

Indeed, $10! = 3628800 = 2^8 \times 3^4 \times 5^2 \times 7$.

#### 2.2.3 Binary Exponentiation Trick

The key optimization is representing each exponent $e_p(n)$ in binary:

$$e_p(n) = \sum_{k=0}^{m} b_k \cdot 2^k$$

where $b_k \in \{0, 1\}$ are the binary digits.

This allows computing $p^{e_p(n)}$ efficiently:

$$p^{e_p(n)} = \prod_{k=0}^{m} (p^{2^k})^{b_k}$$

By grouping primes with the same binary digit pattern, we minimize multiplications.

### 2.3 Correctness Proof

**Lemma 1**: Legendre's formula correctly computes $e_p(n)$.

*Proof*: The number of multiples of $p^i$ in $\{1, 2, \ldots, n\}$ is $\left\lfloor \frac{n}{p^i} \right\rfloor$. Each such multiple contributes at least $i$ factors of $p$, but we've already counted $(i-1)$ in previous terms. The sum telescopes to give the total exponent.

**Lemma 2**: Binary grouping reduces multiplication count.

*Proof*: Instead of $\pi(n)$ individual exponentiations (where $\pi(n)$ is the prime counting function), we group primes by binary digits. For each bit position $k$, we compute:

$$a_k = \prod_{p : \text{bit } k \text{ of } e_p(n) \text{ is 1}} p$$

Then raise $a_k$ to the power $2^k$ and multiply all results. This requires only $O(\log n)$ exponentiations.

**Theorem (Correctness)**: The algorithm correctly computes $n!$.

*Proof*: By construction, we compute:

$$\prod_{k=0}^{m} a_k^{2^k} = \prod_{k=0}^{m} \left(\prod_{p : b_k(p) = 1} p\right)^{2^k} = \prod_p p^{\sum_k b_k(p) \cdot 2^k} = \prod_p p^{e_p(n)} = n!$$

where $b_k(p)$ is the $k$-th bit of $e_p(n)$.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm works in four phases:

1. **Prime Generation**: Find all primes $p \leq n$ using the Sieve of Eratosthenes
2. **Exponent Calculation**: For each prime, compute its exponent in $n!$ using Legendre's formula
3. **Binary Grouping**: Group primes by their exponent's binary representation
4. **Exponential Multiplication**: Raise each group to appropriate powers of 2 and multiply

**Why it's faster**:
- Traditional method: $O(n)$ multiplications
- This method: $O(\log n)$ groups, each requiring multiplication and exponentiation
- Overall complexity: $O(\log \log n \cdot M(n \log n))$ where $M(n)$ is multiplication complexity

### 3.2 Pseudocode

```
function fast_factorial(n):
    if n < 2:
        return 1
    
    // Phase 1: Generate all primes ≤ n
    primes = sieve_of_eratosthenes(n)
    
    // Phase 2: Compute exponent for each prime
    p_indices = {}
    for each prime p in primes:
        p_indices[p] = index(p, n)  // Legendre's formula
    
    // Phase 3: Determine maximum number of bits needed
    max_bits = ceil(log2(max(p_indices.values())))
    
    // Phase 4: Initialize accumulator array
    a = array of 1's with length max_bits
    
    // Phase 5: Group primes by binary representation
    for each (p, exponent) in p_indices:
        for bit_position from 0 to max_bits - 1:
            if bit bit_position of exponent is 1:
                a[bit_position] *= p
    
    // Phase 6: Compute final result
    result = 1
    for i from 0 to max_bits - 1:
        result *= a[i]^(2^i)
    
    return result

function index(p, n):
    // Legendre's formula implementation
    total = 0
    power = 1
    quotient = n / p
    
    while quotient > 0:
        total += quotient
        power += 1
        quotient = n / p^power
    
    return total
```

### 3.3 Step-by-Step Example

**Example**: Compute $6! = 720$

**Step 1: Find primes** ≤ 6
- Primes: $[2, 3, 5]$

**Step 2: Compute exponents**

For $p = 2$:
$$e_2(6) = \left\lfloor \frac{6}{2} \right\rfloor + \left\lfloor \frac{6}{4} \right\rfloor = 3 + 1 = 4 = 100_2$$

For $p = 3$:
$$e_3(6) = \left\lfloor \frac{6}{3} \right\rfloor = 2 = 10_2$$

For $p = 5$:
$$e_5(6) = \left\lfloor \frac{6}{5} \right\rfloor = 1 = 1_2$$

**Step 3: Map primes to exponents**
- $\{2 \to 4, 3 \to 2, 5 \to 1\}$

**Step 4: Determine max bits**
- Maximum exponent: 4
- $\lceil \log_2(4) \rceil + 1 = 3$ bits needed

**Step 5: Initialize array**
- $a = [1, 1, 1]$ (3 elements)

**Step 6: Group by binary digits**

| Prime | Exponent | Binary | Bit 0 | Bit 1 | Bit 2 |
|-------|----------|--------|-------|-------|-------|
| 2     | 4        | 100    | 0     | 0     | 1     |
| 3     | 2        | 010    | 0     | 1     | 0     |
| 5     | 1        | 001    | 1     | 0     | 0     |

- Bit 0 is set for prime 5: $a[0] = 1 \times 5 = 5$
- Bit 1 is set for prime 3: $a[1] = 1 \times 3 = 3$
- Bit 2 is set for prime 2: $a[2] = 1 \times 2 = 2$

Result: $a = [5, 3, 2]$

**Step 7: Compute final result**

$$\text{result} = a[0]^{2^0} \times a[1]^{2^1} \times a[2]^{2^2} = 5^1 \times 3^2 \times 2^4 = 5 \times 9 \times 16 = 720$$

**Verification**: $6! = 1 \times 2 \times 3 \times 4 \times 5 \times 6 = 720$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

**Overall Complexity**: $O(\log \log n \cdot M(n \log n))$

where $M(n)$ is the complexity of multiplying two $n$-digit numbers.

**Component Breakdown**:

1. **Sieve of Eratosthenes**: $O(n \log \log n)$
   - Generates all primes up to $n$

2. **Exponent Calculation**: $O(\pi(n) \cdot \log n)$
   - For each of $\pi(n) \approx \frac{n}{\ln n}$ primes
   - Legendre's formula requires $O(\log_p n) = O(\log n)$ divisions

3. **Binary Grouping**: $O(\pi(n) \cdot \log n)$
   - For each prime, check $O(\log n)$ bits
   - Multiply prime into appropriate group

4. **Final Exponentiation**: $O(\log n \cdot M(n \log n))$
   - Raise $O(\log n)$ groups to powers of 2
   - Each group product can have $O(n \log n)$ bits (size of $n!$)
   - Dominant term in overall complexity

**Comparison with Naive Approach**:
- Naive: $O(n \cdot M(\log(n!))) = O(n \cdot M(n \log n))$
- Borwein: $O(\log \log n \cdot M(n \log n))$
- **Speedup**: Factor of $\frac{n}{\log \log n}$ for large $n$

**Practical Performance**:
- For $n = 1000$: ~100x faster than naive
- For $n = 10000$: ~1000x faster than naive
- For $n = 100000$: ~10000x faster than naive

### 4.2 Space Complexity

**Auxiliary Space**: $O(\pi(n) + \log n)$

1. **Prime Storage**: $O(\pi(n)) \approx O(n / \log n)$ primes
2. **Exponent Map**: $O(\pi(n))$ entries
3. **Accumulator Array**: $O(\log n)$ big integers
4. **Intermediate Results**: $O(n \log n)$ bits for the result

**Total Space**: $O(n / \log n)$ integers plus $O(n \log n)$ bits for the final result.

**Note**: The result $n!$ itself requires $\Theta(n \log n)$ bits, so any correct algorithm must use at least this much space.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Big Integer Library**:
```rust
use num_bigint::BigUint;
use num_traits::One;
```
- `BigUint`: Arbitrary-precision unsigned integers
- Essential for factorials > 20! (exceeds u64 range)

**Collections**:
```rust
use std::collections::BTreeMap;
```
- `BTreeMap` for ordered prime-to-exponent mapping
- Ensures deterministic iteration order

**Bit Manipulation**:
```rust
let max_bits = p_indices[&2].next_power_of_two().ilog2() + 1;
```
- `next_power_of_two()`: Efficiently rounds up to power of 2
- `ilog2()`: Integer logarithm base 2 (stable in Rust 1.67+)

**Exponentiation**:
```rust
a_i.pow(2u32.pow(i as u32))
```
- `BigUint::pow()` for arbitrary precision
- Nested power: $2^{2^i}$ for binary exponentiation

**Functional Style**:
```rust
a.into_iter()
    .enumerate()
    .map(|(i, a_i)| a_i.pow(2u32.pow(i as u32)))
    .product()
```
- Iterator chain for clean, expressive code
- `product()` trait for multiplication reduction

### 5.2 Edge Cases

| Edge Case | Handling | Result |
|-----------|----------|--------|
| **n = 0** | Early return | $0! = 1$ |
| **n = 1** | Early return | $1! = 1$ |
| **n = 2** | First computation | $2! = 2$ |
| **Large prime exponent** | Binary grouping handles automatically | Correct result |
| **Power-of-2 exponents** | Optimal case (fewer bits set) | Faster computation |
| **Very large n** | Limited by memory for storing primes | Graceful degradation |

**Special Considerations**:

1. **Overflow Safety**: All arithmetic in `BigUint`, no overflow possible
2. **Memory Limits**: For extremely large $n$ (> 1 million), prime sieve may exhaust memory
3. **Bit Operations**: Bit checking must account for all exponents up to $e_2(n)$ (largest)

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Combinatorics and Probability**
- **Binomial Coefficients**: $\binom{n}{k} = \frac{n!}{k!(n-k)!}$
- **Permutations**: Computing arrangements ($n!$ orderings)
- **Stirling Numbers**: Recurrence relations involving factorials
- **Probability Distributions**: Poisson, multinomial distributions

**2. Cryptography**
- **RSA Key Generation**: Computing $\phi(n)$ involves factorials in some variants
- **Combinatorial Cryptanalysis**: Estimating search space sizes
- **Hash Functions**: Some constructions use factorial-based mixing

**3. Symbolic Mathematics**
- **Computer Algebra Systems (CAS)**: Sage, SymPy, Mathematica
- **Taylor Series Expansion**: $e^x = \sum_{n=0}^{\infty} \frac{x^n}{n!}$
- **Gamma Function**: $\Gamma(n) = (n-1)!$ for positive integers
- **Special Functions**: Bessel functions, hypergeometric functions

**4. Scientific Computing**
- **Stirling's Approximation Verification**: Comparing $n!$ with $\sqrt{2\pi n}\left(\frac{n}{e}\right)^n$
- **Number Theory Research**: Distribution of prime factors
- **Sequence Analysis**: OEIS (Online Encyclopedia of Integer Sequences)

**5. Benchmarking and Testing**
- **Big Integer Library Testing**: Factorial is a standard benchmark
- **Performance Profiling**: Stress testing arithmetic operations
- **Algorithm Verification**: Known results for validation

### 6.2 Related Algorithms

**Factorial Computation Methods**:

1. **Naive Iterative**: $O(n)$ multiplications
   ```
   result = 1
   for i from 2 to n:
       result *= i
   ```
   - Simplest implementation
   - Practical for small $n$ (< 100)

2. **Divide and Conquer**: $O(\log n)$ levels, $O(n)$ work
   ```
   factorial(1, n) = factorial(1, n/2) * factorial(n/2+1, n)
   ```
   - Better cache locality
   - Easier to parallelize

3. **Prime Swing Factorial** (Luschny): Even faster than Borwein
   - Uses "swing numbers" and prime factorization
   - Asymptotically optimal
   - More complex implementation

4. **Binary Split Method**: For computing $\sum \frac{1}{n!}$ (e.g., $e$)
   - Efficient for rational arithmetic
   - Minimizes GCD computations

**Complementary Algorithms**:

- **Sieve of Eratosthenes**: Prerequisite for Borwein's algorithm
- **Fast Exponentiation**: Used in final multiplication phase
- **GCD/LCM**: Often combined with factorial in number theory
- **Logarithmic Factorial**: $\log(n!) = \sum_{i=1}^{n} \log i$ for approximations

**When to Use Each**:
- **Small $n$ (< 20)**: Use lookup table
- **Medium $n$ (20-1000)**: Naive or divide-and-conquer
- **Large $n$ (1000-100000)**: Borwein's algorithm
- **Very large $n$ (> 100000)**: Prime swing or specialized methods

## 7. References

### Primary Source
1. **Borwein, Peter** (1985). "The Complexity of Computing the Factorial". *Journal of Algorithms*, 6(3), 376-380. 
   - DOI: [10.1016/0196-6774(85)90006-9](https://doi.org/10.1016/0196-6774(85)90006-9)
   - Original paper describing this algorithm

### Textbooks
2. **Knuth, Donald E.** (1997). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (3rd ed.). 
   - Section 4.5.4: "Factoring into Primes"
   - Section 4.6.3: "Evaluation of Powers"

3. **Crandall, Richard; Pomerance, Carl** (2005). *Prime Numbers: A Computational Perspective* (2nd ed.). Springer.
   - Chapter 5: "Exponentiating and Computing Orders"

### Advanced References
4. **Luschny, Peter** (2008-present). "Factorial Algorithms"
   - [Homepage](http://www.luschny.de/math/factorial/FastFactorialFunctions.htm)
   - Comprehensive survey of factorial algorithms
   - Includes even faster "Prime Swing" method

5. **Shoup, Victor** (2005). *A Computational Introduction to Number Theory and Algebra*. Cambridge University Press.
   - Chapter 15: "Faster Integer Arithmetic"

### Implementation References
- **GMP (GNU Multiple Precision Library)**: `mpz_fac_ui()` implementation
- **FLINT (Fast Library for Number Theory)**: `fmpz_fac_ui()`
- **Rust `num-bigint` crate**: Reference big integer implementation

### Mathematical Foundations
6. **Legendre, Adrien-Marie** (1808). *Essai sur la théorie des nombres*
   - Original formulation of Legendre's formula

7. **OEIS Sequence A000142**: Factorial numbers $n!$
   - [Link](https://oeis.org/A000142)
   - Extensive references and properties

### Online Resources
- [Rosetta Code: Factorial](https://rosettacode.org/wiki/Factorial)
- [MathWorld: Factorial](https://mathworld.wolfram.com/Factorial.html)
- [Wikipedia: Factorial](https://en.wikipedia.org/wiki/Factorial)
