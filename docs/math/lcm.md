# Least Common Multiple (LCM)

## 1. Overview

The **Least Common Multiple (LCM)** of two or more integers is the smallest positive integer that is divisible by all of them. The LCM is fundamental in problems involving periodicity, scheduling, and fraction arithmetic.

**File**: `src/math/lcm_of_n_numbers.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given integers $a_1, a_2, \ldots, a_n$, find the smallest positive integer $m$ such that:
$$a_i | m \text{ for all } i \in \{1, 2, \ldots, n\}$$

Formally: $\text{lcm}(a_1, \ldots, a_n) = \min\{m \in \mathbb{Z}^+ : a_i | m \text{ for all } i\}$

### 2.2 Relationship with GCD

For two numbers, LCM and GCD are related by:

$$\text{lcm}(a, b) = \frac{|a \cdot b|}{\gcd(a, b)}$$

**Proof**: Let $d = \gcd(a, b)$, $a = d \cdot a'$, $b = d \cdot b'$ where $\gcd(a', b') = 1$.
Any common multiple of $a$ and $b$ must be divisible by $d \cdot a' \cdot b' = \frac{ab}{d}$.
Conversely, $\frac{ab}{d} = a \cdot b' = b \cdot a'$ is divisible by both $a$ and $b$.

### 2.3 Mathematical Properties

1. **Commutativity**: $\text{lcm}(a, b) = \text{lcm}(b, a)$
2. **Associativity**: $\text{lcm}(a, \text{lcm}(b, c)) = \text{lcm}(\text{lcm}(a, b), c)$
3. **Identity**: $\text{lcm}(a, 1) = |a|$
4. **Absorption**: $\text{lcm}(a, a) = |a|$
5. **Distributivity**: $\text{lcm}(ka, kb) = k \cdot \text{lcm}(a, b)$ for $k > 0$
6. **Divisibility**: If $a | b$, then $\text{lcm}(a, b) = |b|$

### 2.4 Prime Factorization View

If $a = \prod p_i^{\alpha_i}$ and $b = \prod p_i^{\beta_i}$, then:
$$\text{lcm}(a, b) = \prod p_i^{\max(\alpha_i, \beta_i)}$$

Compare with:
$$\gcd(a, b) = \prod p_i^{\min(\alpha_i, \beta_i)}$$

## 3. Algorithm Description

### 3.1 Intuition

For multiple numbers, we compute LCM iteratively using the two-number formula:
$$\text{lcm}(a_1, a_2, \ldots, a_n) = \text{lcm}(a_1, \text{lcm}(a_2, \ldots, a_n))$$

### 3.2 Pseudocode

```
function lcm(nums[]):
    if length(nums) = 1:
        return nums[0]
    
    a ← nums[0]
    b ← lcm(nums[1:])
    
    return a × b / gcd(a, b)

function gcd(a, b):
    if b = 0:
        return a
    return gcd(b, a mod b)
```

### 3.3 Step-by-Step Example

Calculate $\text{lcm}(12, 18, 24)$:

**Step 1**: Compute $\text{lcm}(18, 24)$
- $\gcd(18, 24) = 6$
- $\text{lcm}(18, 24) = \frac{18 \times 24}{6} = \frac{432}{6} = 72$

**Step 2**: Compute $\text{lcm}(12, 72)$
- $\gcd(12, 72) = 12$
- $\text{lcm}(12, 72) = \frac{12 \times 72}{12} = 72$

**Result**: $\text{lcm}(12, 18, 24) = 72$

**Verification**:
- $72 / 12 = 6$ ✓
- $72 / 18 = 4$ ✓
- $72 / 24 = 3$ ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

For $n$ numbers, each with maximum value $M$:

| Operation | Complexity |
|-----------|------------|
| Single GCD | O(log M) |
| LCM of 2 numbers | O(log M) |
| LCM of n numbers | O(n log M) |

### 4.2 Space Complexity

| Implementation | Space |
|----------------|-------|
| Recursive | O(n) - stack depth |
| Iterative | O(1) |

### 4.3 Overflow Considerations

The intermediate product $a \times b$ can overflow. Safe computation:
```rust
// Instead of: (a * b) / gcd(a, b)
// Use: (a / gcd(a, b)) * b
```

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn lcm(nums: &[usize]) -> usize {
    if nums.len() == 1 {
        return nums[0];
    }
    let a = nums[0];
    let b = lcm(&nums[1..]);
    a * b / gcd_of_two_numbers(a, b)
}

fn gcd_of_two_numbers(a: usize, b: usize) -> usize {
    if b == 0 {
        return a;
    }
    gcd_of_two_numbers(b, a % b)
}
```

**Key patterns**:
- Slice recursion with `&nums[1..]`
- Recursive GCD helper function
- Uses unsigned integers (no negative handling needed)

### 5.2 Edge Cases

| Input | Output | Notes |
|-------|--------|-------|
| `[n]` | `n` | Single element |
| `[a, 0]` | Undefined | Division by zero with GCD formula |
| `[0, 0]` | Undefined | $\text{lcm}(0, 0)$ is undefined |
| `[a, 1]` | `a` | LCM with 1 |
| `[a, a]` | `a` | Idempotence |

### 5.3 Overflow-Safe Implementation

```rust
pub fn lcm_safe(a: u64, b: u64) -> Option<u64> {
    if a == 0 || b == 0 {
        return Some(0);
    }
    let g = gcd(a, b);
    (a / g).checked_mul(b)
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Task Scheduling**: Finding when periodic tasks align
   ```rust
   // Tasks repeat every 3, 5, and 7 minutes
   // They align every lcm(3, 5, 7) = 105 minutes
   ```

2. **Clock Synchronization**: Finding common tick intervals

3. **Fraction Addition**: Finding common denominators
   ```rust
   // To add 1/6 + 1/8 + 1/12:
   // LCD = lcm(6, 8, 12) = 24
   // = 4/24 + 3/24 + 2/24 = 9/24
   ```

4. **Display Refresh Rates**: Finding common refresh rates

5. **Game Development**: Animation frame synchronization

### 6.2 Mathematical Applications

| Application | Description |
|-------------|-------------|
| Chinese Remainder Theorem | Product of coprime moduli |
| Cryptography | Key period calculation |
| Number Theory | Studying periodicity |

## 7. Related Algorithms

| Algorithm | Relation |
|-----------|----------|
| [GCD](gcd.md) | Core building block: $\text{lcm}(a,b) \times \gcd(a,b) = |ab|$ |
| [Extended Euclidean](extended_euclidean.md) | Alternative GCD computation |
| [Prime Factorization](prime_factors.md) | Can compute LCM via max exponents |

## 8. Variants and Optimizations

### 8.1 Iterative Implementation

```rust
pub fn lcm_iterative(nums: &[usize]) -> usize {
    nums.iter().fold(1, |acc, &x| {
        acc * x / gcd(acc, x)
    })
}
```

### 8.2 Parallel Computation

For large arrays, LCM can be computed in parallel using divide-and-conquer:

```rust
// Pseudocode for parallel LCM
fn parallel_lcm(nums: &[usize]) -> usize {
    if nums.len() <= THRESHOLD {
        return lcm_sequential(nums);
    }
    let mid = nums.len() / 2;
    let (left, right) = rayon::join(
        || parallel_lcm(&nums[..mid]),
        || parallel_lcm(&nums[mid..])
    );
    lcm_two(left, right)
}
```

### 8.3 LCM with BigIntegers

For cryptographic applications with large numbers:

```rust
use num_bigint::BigUint;

fn big_lcm(a: &BigUint, b: &BigUint) -> BigUint {
    let g = a.gcd(b);
    (a / &g) * b
}
```

## 9. Complexity Comparison Table

| Method | Time | Space | Overflow Risk |
|--------|------|-------|---------------|
| Via GCD | O(n log M) | O(1) | Low if careful |
| Prime factorization | O(n√M) | O(log M) | Low |
| Binary method | O(n log²M) | O(1) | Low |

## 10. References

1. Knuth, D. E. "The Art of Computer Programming, Vol. 2", Section 4.5.2
2. Cormen, T. H. et al. "Introduction to Algorithms", Chapter 31
3. Hardy, G. H. & Wright, E. M. "An Introduction to the Theory of Numbers"
4. [Wikipedia: Least Common Multiple](https://en.wikipedia.org/wiki/Least_common_multiple)
