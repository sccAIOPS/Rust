# Perfect Numbers

## 1. Overview

A **perfect number** is a positive integer equal to the sum of its proper divisors (excluding itself). The study of perfect numbers dates back to ancient Greek mathematics.

**File**: `src/math/perfect_numbers.rs`

## 2. Mathematical Foundation

### 2.1 Definition

A number $n$ is perfect if:
$$\sigma(n) - n = n$$

Or equivalently:
$$\sigma(n) = 2n$$

Where $\sigma(n)$ is the sum of all divisors of $n$.

### 2.2 Examples

| Number | Proper Divisors | Sum |
|--------|-----------------|-----|
| 6 | 1, 2, 3 | 6 ✓ |
| 28 | 1, 2, 4, 7, 14 | 28 ✓ |
| 12 | 1, 2, 3, 4, 6 | 16 ✗ |
| 496 | 1, 2, 4, 8, 16, 31, 62, 124, 248 | 496 ✓ |
| 8128 | (16 divisors) | 8128 ✓ |

### 2.3 Euclid-Euler Theorem

An even number is perfect if and only if it has the form:
$$2^{p-1}(2^p - 1)$$

Where $2^p - 1$ is a **Mersenne prime**.

| p | Mersenne Prime | Perfect Number |
|---|----------------|----------------|
| 2 | 3 | 6 |
| 3 | 7 | 28 |
| 5 | 31 | 496 |
| 7 | 127 | 8128 |

### 2.4 Open Problems

- Are there any **odd perfect numbers**? (Unknown, none found)
- Are there infinitely many even perfect numbers? (Unknown)

## 3. Implementation

```rust
pub fn is_perfect_number(num: usize) -> bool {
    let mut sum = 0;

    for i in 1..num - 1 {
        if num.is_multiple_of(i) {
            sum += i;
        }
    }

    num == sum
}

pub fn perfect_numbers(max: usize) -> Vec<usize> {
    let mut result: Vec<usize> = Vec::new();

    for i in 1..=max {
        if is_perfect_number(i) {
            result.push(i);
        }
    }

    result
}
```

### 3.1 Usage Example

```rust
assert!(is_perfect_number(6));
assert!(is_perfect_number(28));
assert!(is_perfect_number(496));
assert!(!is_perfect_number(12));

let perfects = perfect_numbers(1000);
// Result: [6, 28, 496]
```

## 4. Complexity

### Current Implementation
- **is_perfect_number**: O(n) - iterates through all potential divisors
- **perfect_numbers**: O(n × max) - checks each number up to max

### Optimized Approach
- Check divisors up to √n: O(√n)
- Use Mersenne prime formula: O(1) for verification

## 5. Related Number Types

| Type | Definition | Examples |
|------|------------|----------|
| Perfect | σ(n) = 2n | 6, 28, 496 |
| Deficient | σ(n) < 2n | 1, 2, 4, 8, 9 |
| Abundant | σ(n) > 2n | 12, 18, 20, 24 |
| Amicable | σ(a) = a + b, σ(b) = a + b | (220, 284) |

## 6. Applications

1. **Number theory research**: Studying divisor functions
2. **Cryptography**: Connection to Mersenne primes
3. **Mathematical education**: Illustrating divisibility
4. **Historical mathematics**: One of the oldest studied number properties

## 7. Optimization Opportunities

```rust
// Optimized: check divisors up to √n
pub fn is_perfect_optimized(num: usize) -> bool {
    if num <= 1 { return false; }
    
    let mut sum = 1;
    let sqrt_n = (num as f64).sqrt() as usize;
    
    for i in 2..=sqrt_n {
        if num % i == 0 {
            sum += i;
            if i != num / i {
                sum += num / i;
            }
        }
    }
    
    sum == num
}
```

## 8. References

1. [Wikipedia: Perfect number](https://en.wikipedia.org/wiki/Perfect_number)
2. [OEIS A000396](https://oeis.org/A000396) - Sequence of perfect numbers
3. Euclid, "Elements" Book IX, Proposition 36
