# Compute Totient (Sieve Method)

## 1. Overview

The Compute Totient algorithm efficiently calculates Euler's totient function φ(i) for all integers from 1 to n using a sieve-based approach. This is significantly more efficient than computing each φ(i) individually when you need totient values for many numbers.

This algorithm is based on the Sieve of Eratosthenes technique, adapted to compute totient values instead of marking primes. It leverages the multiplicative property of the totient function and processes numbers in a systematic manner.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer n, compute an array where the i-th element is φ(i) for all i from 1 to n.

**Formal Definition:**
$$\text{Output} = [\phi(1), \phi(2), \phi(3), \ldots, \phi(n)]$$

### 2.2 Mathematical Model

**Input:**
- A positive integer n where n ≥ 1

**Output:**
- A vector of n integers where the i-th element (0-indexed) is φ(i+1)

**Key Mathematical Properties Used:**

1. **For prime p:** φ(p) = p - 1

2. **For any integer m and prime p dividing m:**
   $$\phi(m) = \phi\left(\frac{m}{p}\right) \cdot \frac{p-1}{1} \text{ if } p \nmid \frac{m}{p}$$
   $$\phi(m) = \phi\left(\frac{m}{p}\right) \cdot p \text{ if } p | \frac{m}{p}$$

3. **Simplified update rule:** When processing prime p and its multiples:
   $$\phi(kp) = \frac{\phi(kp)}{p} \cdot (p-1) = \phi(kp) \cdot \frac{p-1}{p}$$

### 2.3 Correctness Proof

**Theorem:** The sieve correctly computes all totient values.

**Proof:**
1. Initialize φ(i) = i for all i (trivially true for i = 1).
2. For each prime p (detected when φ(p) = p after previous iterations):
   - Set φ(p) = p - 1 (correct by prime property)
   - For multiples kp: update φ(kp) = φ(kp) × (p-1)/p
3. Each composite number m is visited once for each of its prime factors.
4. After processing all primes ≤ n, the product formula has been applied:
   $$\phi(m) = m \cdot \prod_{p|m}\frac{p-1}{p}$$

**Invariant:** After processing prime p, all numbers whose smallest unprocessed prime factor is > p have correct totient values accounting for all prime factors ≤ p.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm works like a sieve:

1. Start with φ(i) = i for all i (as if all numbers were coprime to everything)
2. For each number p starting from 2:
   - If φ(p) = p, then p is prime (wasn't modified by any smaller prime)
   - Set φ(p) = p - 1 (prime property)
   - For all multiples of p, "knock off" the factor (p-1)/p from their totient values
3. After processing all numbers up to n, all totient values are correct

### 3.2 Pseudocode

```
COMPUTE_TOTIENT(n):
    // Initialize phi array
    phi ← array of size n+1
    FOR i ← 0 TO n DO
        phi[i] ← i
    END FOR
    
    // Sieve process
    FOR p ← 2 TO n DO
        IF phi[p] = p THEN
            // p is prime (not modified by any smaller prime)
            phi[p] ← p - 1
            
            // Update all multiples of p
            FOR i ← 2*p TO n STEP p DO
                phi[i] ← (phi[i] / p) * (p - 1)
            END FOR
        END IF
    END FOR
    
    RETURN phi[1..n]  // Return elements from index 1 to n
```

### 3.3 Step-by-Step Example

**Example: compute_totient(12)**

**Initialization:** phi = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]

**Processing:**

| p | Condition | Action | phi array after |
|---|-----------|--------|-----------------|
| 2 | phi[2]=2 (prime) | phi[2]=1; update 4,6,8,10,12 | [0,1,1,3,2,5,3,7,4,9,5,11,6] |
| 3 | phi[3]=3 (prime) | phi[3]=2; update 6,9,12 | [0,1,1,2,2,5,2,7,4,6,5,11,4] |
| 4 | phi[4]=2≠4 | skip | unchanged |
| 5 | phi[5]=5 (prime) | phi[5]=4; update 10 | [0,1,1,2,2,4,2,7,4,6,4,11,4] |
| 6 | phi[6]=2≠6 | skip | unchanged |
| 7 | phi[7]=7 (prime) | phi[7]=6; no multiples ≤12 | [0,1,1,2,2,4,2,6,4,6,4,11,4] |
| 8-10 | not prime | skip | unchanged |
| 11 | phi[11]=11 (prime) | phi[11]=10; no multiples ≤12 | [0,1,1,2,2,4,2,6,4,6,4,10,4] |
| 12 | phi[12]=4≠12 | skip | unchanged |

**Result:** [1, 1, 2, 2, 4, 2, 6, 4, 6, 4, 10, 4] (indices 1-12)

**Verification:**
- φ(1) = 1 ✓
- φ(6) = 2 ({1, 5} are coprime to 6) ✓
- φ(12) = 4 ({1, 5, 7, 11} are coprime to 12) ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| All cases | O(n log log n) | Same as Sieve of Eratosthenes |

**Derivation:**
- Outer loop: O(n) iterations
- Inner loop for prime p: processes n/p multiples
- Total work: $\sum_{p \text{ prime}, p \leq n} \frac{n}{p} = n \cdot \sum_{p \leq n} \frac{1}{p} \approx n \cdot \ln(\ln n)$
- By Mertens' theorem: O(n log log n)

### 4.2 Space Complexity

- **Auxiliary Space:** O(n) - Array to store all totient values
- **Stack Space:** O(1) - Iterative algorithm

**Trade-off:** This algorithm uses more space than computing single totient values but is much faster for computing multiple values.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::vec;

pub fn compute_totient(n: i32) -> vec::Vec<i32> {
    let mut phi: Vec<i32> = Vec::new();

    // Initialize phi[i] = i
    for i in 0..=n {
        phi.push(i);
    }

    // Compute Phi values using sieve
    for p in 2..=n {
        if phi[p as usize] == p {
            // p is prime
            phi[p as usize] = p - 1;
            
            // Update multiples of p
            for i in ((2 * p)..=n).step_by(p as usize) {
                phi[i as usize] = (phi[i as usize] / p) * (p - 1);
            }
        }
    }

    phi[1..].to_vec()
}
```

**Key Points:**
- Uses `Vec<i32>` for dynamic array allocation
- `step_by()` iterator for efficient multiples enumeration
- Returns slice from index 1 to exclude the 0th element
- Integer division by p before multiplication by (p-1) to avoid overflow

**Potential Improvements:**
```rust
// Pre-allocate with capacity for efficiency
let mut phi: Vec<i32> = (0..=n).collect();

// Use usize throughout to avoid casting
pub fn compute_totient(n: usize) -> Vec<usize>
```

### 5.2 Edge Cases

| Input | Output | Explanation |
|-------|--------|-------------|
| 1 | [1] | φ(1) = 1 |
| 2 | [1, 1] | φ(1) = 1, φ(2) = 1 |
| 0 | [] | Empty result (no positive integers ≤ 0) |
| Large n | Vector of n elements | Memory consideration |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Precomputation for Competitive Programming:**
   - Many problems require multiple totient queries
   - Precompute once, answer queries in O(1)

2. **Cryptographic Key Generation:**
   - When generating multiple RSA key pairs
   - Analyzing totient patterns for security research

3. **Number Theory Research:**
   - Computing totient summatory function: $\Phi(n) = \sum_{k=1}^{n} \phi(k)$
   - Studying distribution of totient values

4. **Reduced Residue Systems:**
   - Finding all primitive roots modulo n
   - Group theory computations in modular arithmetic

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Use |
|-----------|--------------|-------------|
| euler_totient | Single value computation | One-off queries, large n |
| Sieve of Eratosthenes | Same technique | Finding primes |
| Linear Sieve | O(n) prime sieve | When O(n) is needed |
| Mobius Function Sieve | Similar sieve pattern | Computing μ(n) for all n |

### 6.3 Performance Comparison

| Method | Time for n=10⁶ | Time for n=10⁷ |
|--------|---------------|----------------|
| Individual φ(i) calls | ~10s | ~100s |
| Sieve (this algorithm) | ~50ms | ~500ms |

## 7. References

1. **Books:**
   - Bach, E. & Shallit, J. "Algorithmic Number Theory"
   - Crandall, R. & Pomerance, C. "Prime Numbers: A Computational Perspective"

2. **Online Resources:**
   - [CP-Algorithms: Euler's Totient Function](https://cp-algorithms.com/algebra/phi-function.html)
   - [OEIS A000010](https://oeis.org/A000010) - Sequence of totient values

3. **Related Implementations:**
   - [`euler_totient.rs`](euler_totient.md) - Single value computation
   - [`sieve_of_eratosthenes.rs`](../math/sieve_of_eratosthenes.md) - Prime sieve

4. **Academic Papers:**
   - Mertens, F. (1874). "Ein Beitrag zur analytischen Zahlentheorie"
