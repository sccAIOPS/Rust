# Kth Factor of N

## 1. Overview

The Kth Factor algorithm finds the kth smallest factor (divisor) of a given number n. A factor of n is any positive integer that divides n evenly (with no remainder). This problem appears frequently in competitive programming and has applications in number theory and factorization problems.

This straightforward approach iterates through potential divisors and collects factors until the kth one is found or determines that fewer than k factors exist.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given two positive integers n and k, find the kth smallest positive divisor of n, or return -1 if n has fewer than k divisors.

**Formal Definition:**
Let $D(n) = \{d : d | n, d > 0\}$ be the set of positive divisors of n.
Let $d_1 < d_2 < \ldots < d_{|D(n)|}$ be the divisors in sorted order.

$$\text{kth\_factor}(n, k) = \begin{cases} d_k & \text{if } k \leq |D(n)| \\ -1 & \text{otherwise} \end{cases}$$

### 2.2 Mathematical Model

**Input:**
- n: A positive integer (the number to factorize)
- k: A positive integer (the index of the desired factor)

**Output:**
- The kth smallest factor of n, or -1 if it doesn't exist

**Key Properties:**

1. **Divisor count:** The number of divisors τ(n) (tau function) for $n = p_1^{a_1} \cdot p_2^{a_2} \cdots p_m^{a_m}$ is:
   $$\tau(n) = (a_1 + 1)(a_2 + 1) \cdots (a_m + 1)$$

2. **Divisor bounds:**
   - Minimum divisors: 2 (for primes > 1)
   - Maximum divisors for n: approximately $n^{1/\ln(\ln(n))}$ (highly composite numbers)

3. **Divisor symmetry:** If d divides n, then n/d also divides n.

### 2.3 Correctness Proof

**Theorem:** The algorithm correctly finds the kth factor.

**Proof:**
1. We iterate i from 1 to n in ascending order.
2. For each i, we check if i divides n (i.e., n mod i = 0).
3. If i divides n, it's a factor, and we add it to our list.
4. Factors are discovered in ascending order since we iterate from 1.
5. When we've collected k factors, the kth one is returned.
6. If we reach n+1 without finding k factors, we return -1.

**Invariant:** After iteration i, the factors list contains all divisors of n that are ≤ i, in sorted order.

## 3. Algorithm Description

### 3.1 Intuition

The algorithm uses a simple linear scan:

1. Start from 1 (the smallest possible factor)
2. For each number i up to n, check if it divides n
3. If it does, add it to the list of factors
4. Once we have k factors, return the kth one
5. If we finish scanning without finding k factors, return -1

The key optimization is early termination: we return as soon as we find the kth factor, without computing all divisors.

### 3.2 Pseudocode

```
KTH_FACTOR(n, k):
    factors ← empty list
    
    FOR i ← 1 TO n DO
        IF n MOD i = 0 THEN
            APPEND i TO factors
        END IF
        
        IF LENGTH(factors) = k THEN
            RETURN factors[k-1]  // Return kth factor (0-indexed)
        END IF
    END FOR
    
    RETURN -1  // Fewer than k factors exist
```

### 3.3 Step-by-Step Example

**Example 1: kth_factor(12, 3)**

| i | n % i | Is factor? | Factors list | Check |
|---|-------|------------|--------------|-------|
| 1 | 12 % 1 = 0 | Yes | [1] | len < 3 |
| 2 | 12 % 2 = 0 | Yes | [1, 2] | len < 3 |
| 3 | 12 % 3 = 0 | Yes | [1, 2, 3] | len = 3 ✓ |

**Result:** Return factors[2] = 3 ✓

All factors of 12: {1, 2, 3, 4, 6, 12}, so the 3rd factor is 3.

---

**Example 2: kth_factor(7, 2)**

| i | n % i | Is factor? | Factors list | Check |
|---|-------|------------|--------------|-------|
| 1 | 7 % 1 = 0 | Yes | [1] | len < 2 |
| 2 | 7 % 2 = 1 | No | [1] | - |
| 3 | 7 % 3 = 1 | No | [1] | - |
| 4 | 7 % 4 = 3 | No | [1] | - |
| 5 | 7 % 5 = 2 | No | [1] | - |
| 6 | 7 % 6 = 1 | No | [1] | - |
| 7 | 7 % 7 = 0 | Yes | [1, 7] | len = 2 ✓ |

**Result:** Return factors[1] = 7 ✓

---

**Example 3: kth_factor(4, 4)**

Factors of 4: {1, 2, 4} → only 3 factors exist.

**Result:** Return -1 (k=4 > τ(4)=3)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Description |
|------|------------|-------------|
| Best | O(k) | First k numbers are all factors (e.g., n=2^m) |
| Average | O(n) | Depends on factor distribution |
| Worst | O(n) | Need to scan all numbers up to n |

**Derivation:**
- The loop runs at most n iterations.
- Each iteration performs O(1) work (modulo operation, comparison, vector push).
- Early termination provides speedup when k is small and n has many small factors.

**Note:** An optimized O(√n) algorithm exists by using the divisor pairing property (d and n/d are both divisors).

### 4.2 Space Complexity

- **Auxiliary Space:** O(min(k, τ(n))) - Vector to store factors found so far
- **Stack Space:** O(1) - Iterative algorithm

In the worst case, this is O(τ(n)), but typically much smaller due to early termination.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn kth_factor(n: i32, k: i32) -> i32 {
    let mut factors: Vec<i32> = Vec::new();
    let k = (k as usize) - 1;  // Convert to 0-based index
    
    for i in 1..=n {
        if n % i == 0 {
            factors.push(i);
        }
        if let Some(number) = factors.get(k) {
            return *number;
        }
    }
    -1
}
```

**Key Points:**
- Uses `Vec<i32>` for dynamic factor storage
- Converts k to 0-based index for vector access
- `get(k)` returns `Option`, safely handling out-of-bounds
- Early return pattern for efficiency
- Returns -1 as sentinel for "not found"

**Alternative Approach (Space-Optimized):**
```rust
pub fn kth_factor_space_optimized(n: i32, k: i32) -> i32 {
    let mut count = 0;
    for i in 1..=n {
        if n % i == 0 {
            count += 1;
            if count == k {
                return i;
            }
        }
    }
    -1
}
```

**O(√n) Optimized Version:**
```rust
pub fn kth_factor_sqrt(n: i32, k: i32) -> i32 {
    let mut factors = Vec::new();
    let sqrt_n = (n as f64).sqrt() as i32;
    
    for i in 1..=sqrt_n {
        if n % i == 0 {
            factors.push(i);
        }
    }
    
    let len = factors.len();
    if k as usize <= len {
        return factors[k as usize - 1];
    }
    
    // Check larger factors in reverse
    let mut count = len as i32;
    for i in (1..=sqrt_n).rev() {
        if n % i == 0 && n / i != i {
            count += 1;
            if count == k {
                return n / i;
            }
        }
    }
    -1
}
```

### 5.2 Edge Cases

| Input | Output | Explanation |
|-------|--------|-------------|
| (n, 1) | 1 | 1 is always the first factor |
| (n, τ(n)) | n | n is always the last factor |
| (n, τ(n)+1) | -1 | More factors requested than exist |
| (1, 1) | 1 | 1 has only one factor |
| (1, 2) | -1 | 1 has only one factor |
| (prime, 2) | prime | Primes have exactly 2 factors |
| (prime, 3) | -1 | Primes have exactly 2 factors |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Factorization Problems:**
   - Finding specific divisors for mathematical computations
   - Generating divisor sequences

2. **Grid and Matrix Operations:**
   - Finding dimensions that evenly divide a total size
   - Layout calculations for UI grids

3. **Load Balancing:**
   - Distributing n items across k workers where k divides n
   - Finding valid partition sizes

4. **Number Theory Research:**
   - Studying divisor distributions
   - Analyzing properties of divisor functions

5. **Competitive Programming:**
   - LeetCode Problem 1492: "The kth Factor of n"
   - Common in interview questions

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Use |
|-----------|--------------|-------------|
| Prime Factorization | Finds prime divisors | Need prime factors specifically |
| Divisor Count τ(n) | Counts total divisors | Only need count, not values |
| Divisor Sum σ(n) | Sums all divisors | Perfect/amicable number checks |
| GCD | Related to common divisors | Finding shared factors |

### 6.3 Optimization Strategies

1. **√n Optimization:** Use divisor pairing to reduce from O(n) to O(√n)
2. **Prime Factorization:** For very large n, factorize first, then generate divisors
3. **Precomputation:** Use sieve for multiple queries on range [1, N]

## 7. References

1. **Books:**
   - Rosen, K.H. "Elementary Number Theory and Its Applications"
   - Niven, I., Zuckerman, H.S. "An Introduction to the Theory of Numbers"

2. **Online Resources:**
   - [LeetCode 1492](https://leetcode.com/problems/the-kth-factor-of-n/) - Original problem
   - [OEIS A000005](https://oeis.org/A000005) - Divisor count sequence

3. **Related Implementations:**
   - [`prime_factors.rs`](../math/prime_factors.md) - Prime factorization
   - [`gcd.rs`](../math/greatest_common_divisor.md) - GCD algorithms

4. **Academic Resources:**
   - Apostol, T.M. "Introduction to Analytic Number Theory" - Chapter on divisor functions
