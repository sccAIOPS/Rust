# Euler's Totient Function

## 1. Overview

Euler's Totient Function, denoted as φ(n) (phi of n), is a fundamental concept in number theory. It counts the number of positive integers up to n that are relatively prime (coprime) to n. Two numbers are relatively prime if their greatest common divisor (GCD) is 1.

Named after the Swiss mathematician Leonhard Euler, this function was first introduced in 1763. It plays a crucial role in modular arithmetic, particularly in RSA cryptography, and has numerous applications in abstract algebra and number theory.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer n, compute φ(n), where:

$$\phi(n) = |\{k : 1 \leq k \leq n, \gcd(k, n) = 1\}|$$

In other words, φ(n) is the count of integers k in the range [1, n] such that gcd(k, n) = 1.

### 2.2 Mathematical Model

**Input:**
- A positive integer n where n ≥ 1

**Output:**
- φ(n): The count of integers in [1, n] that are coprime to n

**Key Properties:**

1. **Multiplicativity:** For coprime integers m and n:
   $$\phi(mn) = \phi(m) \cdot \phi(n)$$

2. **For prime p:**
   $$\phi(p) = p - 1$$

3. **For prime power $p^k$:**
   $$\phi(p^k) = p^k - p^{k-1} = p^{k-1}(p - 1)$$

4. **Product formula:** For any positive integer n with prime factorization $n = p_1^{a_1} \cdot p_2^{a_2} \cdots p_k^{a_k}$:
   $$\phi(n) = n \prod_{p|n}\left(1 - \frac{1}{p}\right) = n \cdot \frac{p_1-1}{p_1} \cdot \frac{p_2-1}{p_2} \cdots \frac{p_k-1}{p_k}$$

### 2.3 Correctness Proof

**Theorem:** The product formula correctly computes φ(n).

**Proof Sketch:**
1. By the inclusion-exclusion principle, we count integers not divisible by any prime factor of n.
2. For each prime factor p, the integers divisible by p in [1, n] number n/p.
3. Using inclusion-exclusion and simplifying, we get:
   $$\phi(n) = n \prod_{p|n}\left(1 - \frac{1}{p}\right)$$

The formula can also be derived from the multiplicativity of φ and the formula for prime powers.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm finds all prime factors of n and applies the product formula iteratively:

1. Start with result = n
2. For each prime factor p found:
   - Divide out all occurrences of p from n
   - Update: result = result - result/p (equivalent to result × (1 - 1/p))
3. If a prime factor larger than √n remains, apply the formula once more

### 3.2 Pseudocode

```
EULER_TOTIENT(n):
    result ← n
    num ← n
    p ← 2
    
    // Find all prime factors up to √num
    WHILE p * p ≤ num DO
        IF num is divisible by p THEN
            // p is a prime factor
            // Remove all factors of p
            WHILE num is divisible by p DO
                num ← num / p
            END WHILE
            // Apply formula: result = result * (1 - 1/p)
            result ← result - result / p
        END IF
        p ← p + 1
    END WHILE
    
    // If num > 1, then it's a remaining prime factor
    IF num > 1 THEN
        result ← result - result / num
    END IF
    
    RETURN result
```

### 3.3 Step-by-Step Example

**Example: φ(12)**

Initial: n = 12, result = 12, num = 12

1. **p = 2:** 12 is divisible by 2
   - Divide out 2s: num = 12 → 6 → 3
   - Update result: result = 12 - 12/2 = 12 - 6 = 6
   
2. **p = 3:** 3 × 3 = 9 > 3, but we continue checking
   - Actually p² = 9 > 3, but let's check: 3 is divisible by 3
   - Wait, let's redo: After p=2, num=3, p increments to 3
   - p = 3: p² = 9 > num = 3, so while loop exits

3. **Post-loop check:** num = 3 > 1, so 3 is a prime factor
   - Update result: result = 6 - 6/3 = 6 - 2 = 4

**Result: φ(12) = 4**

Verification: Numbers coprime to 12 in [1,12]: {1, 5, 7, 11} → 4 numbers ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| Best | O(√n) | n is prime (only one iteration needed) |
| Average | O(√n) | Loop runs at most √n times |
| Worst | O(√n) | Same as average |

**Derivation:**
- The outer while loop runs while p² ≤ n, so at most O(√n) iterations.
- The inner while loop (dividing out factors) runs at most O(log n) times total across all prime factors.
- Overall: O(√n + log n) = O(√n)

### 4.2 Space Complexity

- **Auxiliary Space:** O(1) - Only a constant number of variables used
- **Stack Space:** O(1) - Iterative algorithm, no recursion

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn euler_totient(n: u64) -> u64 {
    let mut result = n;
    let mut num = n;
    let mut p = 2;

    while p * p <= num {
        if num % p == 0 {
            while num % p == 0 {
                num /= p;
            }
            result -= result / p;
        }
        p += 1;
    }

    if num > 1 {
        result -= result / num;
    }

    result
}
```

**Key Points:**
- Uses `u64` for large number support
- The `is_multiple_of` trait method can be used for clarity (available via trait extension)
- Integer division automatically floors, which is correct for this algorithm
- No heap allocations needed

### 5.2 Edge Cases

| Input | Output | Explanation |
|-------|--------|-------------|
| 1 | 1 | Only 1 is coprime to 1 |
| 2 | 1 | Only 1 is coprime to 2 |
| Prime p | p-1 | All numbers except p are coprime |
| p^k | p^(k-1)(p-1) | Prime power formula |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **RSA Cryptography:**
   - Computing the private key requires φ(n) where n = pq
   - The decryption exponent d satisfies: e × d ≡ 1 (mod φ(n))

2. **Modular Exponentiation Optimization:**
   - Euler's theorem: a^φ(n) ≡ 1 (mod n) when gcd(a,n) = 1
   - Used to reduce large exponents in modular arithmetic

3. **Hash Function Design:**
   - Determining cycle lengths in multiplicative hash functions
   - Ensuring uniform distribution properties

4. **Group Theory Applications:**
   - Computing the order of elements in multiplicative groups
   - Analyzing cyclic group structures

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Use |
|-----------|--------------|-------------|
| Compute Totient (Sieve) | Computes φ for all numbers up to n | Multiple queries for small n |
| GCD (Euclidean) | φ counts numbers with gcd = 1 | When checking coprimality |
| Sieve of Eratosthenes | Finding prime factors | Pre-computing primes |
| Modular Exponentiation | Uses Euler's theorem | Large power computations |

## 7. References

1. **Original Work:**
   - Euler, L. (1763). "Theoremata arithmetica nova methodo demonstrata"

2. **Books:**
   - Hardy, G.H. & Wright, E.M. "An Introduction to the Theory of Numbers"
   - Rosen, K.H. "Elementary Number Theory and Its Applications"

3. **Online Resources:**
   - [OEIS A000010](https://oeis.org/A000010) - Euler's totient function sequence
   - [Wikipedia: Euler's totient function](https://en.wikipedia.org/wiki/Euler%27s_totient_function)

4. **Related Implementations:**
   - [`compute_totient.rs`](compute_totient.md) - Sieve-based approach for multiple values
