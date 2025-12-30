# Combinations

## 1. Overview

**Combinations** counts the number of ways to choose $k$ items from $n$ items without regard to order. This is one of the fundamental concepts in combinatorics.

**File**: `src/math/combinations.rs`

## 2. Mathematical Foundation

### 2.1 Definition

The number of combinations of $n$ items taken $k$ at a time:

$$C(n, k) = \binom{n}{k} = \frac{n!}{k!(n-k)!}$$

### 2.2 Computational Formula

To avoid large factorial computations:

$$\binom{n}{k} = \frac{n \cdot (n-1) \cdot ... \cdot (n-k+1)}{k!} = \prod_{i=0}^{k-1} \frac{n-i}{i+1}$$

### 2.3 Properties

| Property | Formula |
|----------|---------|
| Symmetry | $\binom{n}{k} = \binom{n}{n-k}$ |
| Pascal's identity | $\binom{n}{k} = \binom{n-1}{k-1} + \binom{n-1}{k}$ |
| Sum of row | $\sum_{k=0}^{n} \binom{n}{k} = 2^n$ |
| Choose 0 | $\binom{n}{0} = 1$ |
| Choose all | $\binom{n}{n} = 1$ |

### 2.4 Examples

| n | k | C(n,k) | Interpretation |
|---|---|--------|----------------|
| 5 | 2 | 10 | Ways to pick 2 from 5 |
| 10 | 5 | 252 | Ways to pick 5 from 10 |
| 6 | 3 | 20 | Ways to pick 3 from 6 |
| 20 | 5 | 15504 | Ways to pick 5 from 20 |

## 3. Implementation

```rust
pub fn combinations(n: i64, k: i64) -> i64 {
    if n < 0 || k < 0 {
        panic!("Please insert positive values");
    }

    let mut res: i64 = 1;
    for i in 0..k {
        res *= n - i;
        res /= i + 1;
    }

    res
}
```

### 3.1 Key Implementation Detail

The division `res /= i + 1` is performed in each iteration to keep intermediate values small and avoid overflow. This works because the partial product is always divisible.

### 3.2 Usage Example

```rust
assert_eq!(combinations(10, 5), 252);
assert_eq!(combinations(6, 3), 20);
assert_eq!(combinations(20, 5), 15504);
```

## 4. Complexity

- **Time**: O(k)
- **Space**: O(1)

## 5. Algorithm Trace

Computing C(10, 3):

| i | n - i | i + 1 | res (before ÷) | res (after ÷) |
|---|-------|-------|----------------|---------------|
| 0 | 10 | 1 | 10 | 10 |
| 1 | 9 | 2 | 90 | 45 |
| 2 | 8 | 3 | 360 | 120 |

Result: 120 ✓

Verification: $\frac{10!}{3! \cdot 7!} = \frac{10 \times 9 \times 8}{6} = 120$

## 6. Combinations vs Permutations

| Concept | Order matters? | Formula | Example (n=4, k=2) |
|---------|---------------|---------|-------------------|
| Permutation | Yes | $\frac{n!}{(n-k)!}$ | 12 (AB, BA, AC, CA, ...) |
| Combination | No | $\frac{n!}{k!(n-k)!}$ | 6 (AB, AC, AD, BC, BD, CD) |

Relationship: $C(n,k) = \frac{P(n,k)}{k!}$

## 7. Optimization for Large Values

### 7.1 Use Symmetry

```rust
fn combinations_optimized(n: i64, k: i64) -> i64 {
    let k = k.min(n - k);  // C(n,k) = C(n, n-k)
    combinations(n, k)
}
```

### 7.2 BigInt for Very Large Values

See [Binomial Coefficient](binomial_coefficient.md) for arbitrary precision implementation.

## 8. Modular Combinations

For combinations mod prime p, use Lucas' theorem or:

```rust
fn combinations_mod(n: i64, k: i64, p: i64) -> i64 {
    // Precompute factorials and inverse factorials mod p
    // C(n,k) = n! * (k!)^(-1) * ((n-k)!)^(-1) mod p
}
```

## 9. Applications

1. **Probability**: Calculating odds in games
2. **Statistics**: Binomial distribution
3. **Cryptography**: Key generation
4. **Algorithms**: Counting paths, subsets
5. **Lottery**: Odds of winning
6. **Computer science**: Subset enumeration

## 10. Related Functions

- [Binomial Coefficient](binomial_coefficient.md) - BigInt version
- [Factorial](factorial.md) - Used in formula
- [Pascal's Triangle](pascal_triangle.md) - Visualizes combinations

## 11. References

1. [Wikipedia: Combination](https://en.wikipedia.org/wiki/Combination)
2. [Wikipedia: Binomial coefficient](https://en.wikipedia.org/wiki/Binomial_coefficient)
3. Graham, R. L., et al. "Concrete Mathematics"
