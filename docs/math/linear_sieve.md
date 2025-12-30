# Linear Sieve

## 1. Overview

The **Linear Sieve** (also known as Euler's Sieve or the Linear Sieve of Eratosthenes) generates all prime numbers up to $n$ in exactly $O(n)$ time. Unlike the classic Sieve of Eratosthenes, it visits each composite number exactly once and can be extended to compute multiplicative functions.

**File**: `src/math/linear_sieve.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a positive integer $n$:
1. Find all primes up to $n$
2. For each number $i \leq n$, find its smallest prime factor (SPF)

### 2.2 Key Insight

Every composite number $m$ can be uniquely represented as:
$$m = \text{spf}(m) \cdot k$$

where $\text{spf}(m)$ is the smallest prime factor of $m$, and $\text{spf}(m) \leq \text{spf}(k)$.

### 2.3 Algorithm Invariant

When processing number $i$:
- For each prime $p \leq \text{spf}(i)$, mark $p \cdot i$ as composite with $\text{spf}(p \cdot i) = p$
- Stop when $p > \text{spf}(i)$ because then $p \cdot i$ would be marked by a smaller $i'$ later

## 3. Algorithm Description

### 3.1 Intuition

1. For each number $i$ from 2 to $n$:
   - If $i$ has no recorded SPF, it's prime
   - For each prime $p$ seen so far (in order):
     - Mark $p \cdot i$ with SPF = $p$
     - Stop if $p$ divides $i$ (key optimization!)

The stopping condition ensures each composite is marked exactly once.

### 3.2 Pseudocode

```
function linear_sieve(n):
    spf[2..n] ← 0  # smallest prime factor
    primes ← []
    
    for i from 2 to n:
        if spf[i] = 0:  # i is prime
            spf[i] ← i
            primes.append(i)
        
        for p in primes:
            if p > spf[i] or p × i > n:
                break
            spf[p × i] ← p
    
    return primes, spf
```

### 3.3 Step-by-Step Example

Compute for $n = 12$:

| i | spf[i] before | Action | Marked |
|---|---------------|--------|--------|
| 2 | 0 | Prime! spf[2]=2 | spf[4]=2 |
| 3 | 0 | Prime! spf[3]=3 | spf[6]=2, spf[9]=3 |
| 4 | 2 | - | spf[8]=2 (stop: 2\|4) |
| 5 | 0 | Prime! spf[5]=5 | spf[10]=2 (stop: 2<5) |
| 6 | 2 | - | spf[12]=2 (stop: 2\|6) |
| 7 | 0 | Prime! spf[7]=7 | (14>12, stop) |
| 8-12 | - | All already have spf | No new marks |

**Result**: 
- Primes: [2, 3, 5, 7, 11]
- SPF: [_, _, 2, 3, 2, 5, 2, 7, 2, 3, 2, 11, 2]

## 4. Complexity Analysis

### 4.1 Time Complexity

$$O(n)$$

**Proof**: Each composite $m$ is visited exactly once when $i = m / \text{spf}(m)$.

### 4.2 Space Complexity

$O(n)$ for the SPF array.

### 4.3 Comparison with Sieve of Eratosthenes

| Aspect | Eratosthenes | Linear Sieve |
|--------|--------------|--------------|
| Time | O(n log log n) | O(n) |
| Space | O(n) | O(n) |
| Cache efficiency | Better | Worse |
| Practical speed | Often faster | Slower due to cache misses |
| Factorization | Not included | O(log n) per number |
| Multiplicative functions | Extra work | Natural extension |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub struct LinearSieve {
    max_number: usize,
    pub primes: Vec<usize>,
    pub minimum_prime_factor: Vec<usize>,
}

impl LinearSieve {
    pub fn prepare(&mut self, max_number: usize) -> Result<(), &'static str> {
        if max_number <= 1 {
            return Err("Sieve size should be more than 1");
        }
        self.max_number = max_number;
        self.minimum_prime_factor.resize(max_number + 1, 0);
        
        for i in 2..=max_number {
            if self.minimum_prime_factor[i] == 0 {
                self.minimum_prime_factor[i] = i;
                self.primes.push(i);
            }
            for p in self.primes.iter() {
                let mlt = (*p) * i;
                if *p > self.minimum_prime_factor[i] || mlt > max_number {
                    break;
                }
                self.minimum_prime_factor[mlt] = *p;
            }
        }
        Ok(())
    }

    pub fn factorize(&self, mut number: usize) -> Result<Vec<usize>, &'static str> {
        if number > self.max_number {
            return Err("Number too big");
        }
        let mut result: Vec<usize> = Vec::new();
        while number > 1 {
            result.push(self.minimum_prime_factor[number]);
            number /= self.minimum_prime_factor[number];
        }
        Ok(result)
    }
}
```

### 5.2 Key Features

1. **Struct-based**: Allows reuse after initialization
2. **Error handling**: Returns `Result` for invalid inputs
3. **Factorization**: O(log n) per number using SPF array

### 5.3 Edge Cases

| Input | Handling |
|-------|----------|
| n ≤ 1 | Error |
| Double initialization | Error |
| Factorize(0) | Error |
| Factorize(n > max) | Error |

## 6. Applications

### 6.1 Fast Factorization

```rust
// After sieve.prepare(1_000_000):
let factors = sieve.factorize(720).unwrap();
// [2, 2, 2, 2, 3, 3, 5] in O(log 720) time
```

### 6.2 Computing Multiplicative Functions

The linear sieve can compute any multiplicative function $f$ during sieving:

```rust
// Example: Euler's totient function φ(n)
// When p is new prime: φ(p) = p - 1
// When i % p == 0: φ(p·i) = p · φ(i)
// When i % p != 0: φ(p·i) = φ(p) · φ(i) = (p-1) · φ(i)
```

### 6.3 Common Multiplicative Functions

| Function | Base Case (prime p) | Multiplicative Property |
|----------|---------------------|-------------------------|
| φ(n) (Euler's totient) | p - 1 | φ(ab) = φ(a)φ(b) if gcd(a,b)=1 |
| μ(n) (Möbius) | -1 | μ(ab) = μ(a)μ(b) if gcd(a,b)=1 |
| σ(n) (sum of divisors) | p + 1 | σ(ab) = σ(a)σ(b) if gcd(a,b)=1 |
| d(n) (number of divisors) | 2 | d(ab) = d(a)d(b) if gcd(a,b)=1 |

## 7. When to Use

### Use Linear Sieve When:
- You need factorization of many numbers
- Computing multiplicative functions
- Memory is not the bottleneck
- Theoretical O(n) guarantee matters

### Use Eratosthenes When:
- Only need primes (not factorization)
- Cache efficiency matters (large n)
- Simpler code is preferred

## 8. Related Algorithms

| Algorithm | Relation |
|-----------|----------|
| [Sieve of Eratosthenes](sieve_of_eratosthenes.md) | Classic alternative |
| [Prime Factorization](prime_factors.md) | Single-number factorization |
| [Pollard's Rho](pollard_rho.md) | Large number factorization |

## 9. Extended Example: Euler's Totient

```rust
// Computing φ(n) for all n ≤ N using linear sieve
fn compute_totient(n: usize) -> Vec<usize> {
    let mut phi = vec![0; n + 1];
    let mut primes = Vec::new();
    let mut spf = vec![0; n + 1];
    
    phi[1] = 1;
    
    for i in 2..=n {
        if spf[i] == 0 {
            spf[i] = i;
            phi[i] = i - 1;  // φ(p) = p - 1
            primes.push(i);
        }
        for &p in &primes {
            if p > spf[i] || p * i > n { break; }
            spf[p * i] = p;
            if i % p == 0 {
                // φ(p·i) = p · φ(i) when p | i
                phi[p * i] = p * phi[i];
            } else {
                // φ(p·i) = (p-1) · φ(i) when p ∤ i
                phi[p * i] = (p - 1) * phi[i];
            }
        }
    }
    phi
}
```

## 10. References

1. Gries, D. & Misra, J. "A Linear Sieve Algorithm for Finding Prime Numbers" (1978)
2. Knuth, D. E. "The Art of Computer Programming, Vol. 2"
3. [Wikipedia: Sieve of Eratosthenes - Linear Sieve](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes#Euler's_sieve)
