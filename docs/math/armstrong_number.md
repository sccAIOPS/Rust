# Armstrong Numbers (Narcissistic Numbers)

## 1. Overview

An **Armstrong number** (narcissistic number) is a number that equals the sum of its digits raised to the power of the number of digits. These numbers are named after Michael F. Armstrong.

**File**: `src/math/armstrong_number.rs`

## 2. Mathematical Foundation

### 2.1 Definition

A number with $d$ digits $(a_1 a_2 ... a_d)_{10}$ is Armstrong if:

$$n = \sum_{i=1}^{d} a_i^d$$

### 2.2 Examples

| Number | Digits | Calculation | Armstrong? |
|--------|--------|-------------|------------|
| 1 | 1 | 1¹ = 1 | ✓ |
| 153 | 3 | 1³ + 5³ + 3³ = 1 + 125 + 27 = 153 | ✓ |
| 370 | 3 | 3³ + 7³ + 0³ = 27 + 343 + 0 = 370 | ✓ |
| 9474 | 4 | 9⁴ + 4⁴ + 7⁴ + 4⁴ = 6561 + 256 + 2401 + 256 = 9474 | ✓ |
| 105 | 3 | 1³ + 0³ + 5³ = 1 + 0 + 125 = 126 ≠ 105 | ✗ |

### 2.3 All Armstrong Numbers (Base 10)

There are exactly **88** Armstrong numbers in base 10:
```
0, 1, 2, 3, 4, 5, 6, 7, 8, 9,         (1 digit)
153, 370, 371, 407,                    (3 digits)
1634, 8208, 9474,                      (4 digits)
54748, 92727, 93084,                   (5 digits)
548834,                                (6 digits)
...
115132219018763992565095597973971522401  (39 digits, largest)
```

## 3. Implementation

```rust
pub fn is_armstrong_number(number: u32) -> bool {
    let mut digits: Vec<u32> = Vec::new();
    let mut num: u32 = number;

    loop {
        digits.push(num % 10);
        num /= 10;
        if num == 0 {
            break;
        }
    }

    let sum_nth_power_of_digits: u32 = digits
        .iter()
        .map(|digit| digit.pow(digits.len() as u32))
        .sum();
    
    sum_nth_power_of_digits == number
}
```

### 3.1 Usage Example

```rust
assert!(is_armstrong_number(1));       // 1¹ = 1
assert!(is_armstrong_number(153));     // 1³ + 5³ + 3³ = 153
assert!(is_armstrong_number(9474));    // 9⁴ + 4⁴ + 7⁴ + 4⁴ = 9474
assert!(!is_armstrong_number(15));     // No 2-digit Armstrong numbers
assert!(!is_armstrong_number(105));    // 1³ + 0³ + 5³ = 126 ≠ 105
```

## 4. Algorithm

### 4.1 Steps

1. Extract all digits of the number
2. Count the number of digits (d)
3. Compute sum of each digit raised to power d
4. Compare sum with original number

### 4.2 Pseudocode

```
function is_armstrong(n):
    digits = []
    temp = n
    while temp > 0:
        digits.append(temp % 10)
        temp = temp / 10
    
    d = length(digits)
    sum = 0
    for digit in digits:
        sum += digit^d
    
    return sum == n
```

## 5. Complexity

- **Time**: O(d) where d = number of digits = O(log n)
- **Space**: O(d) for storing digits

## 6. Interesting Properties

### 6.1 No 2-Digit Armstrong Numbers

Proof: For a 2-digit number $ab$:
- Maximum: 9² + 9² = 162 (but 99 is the largest 2-digit number)
- Maximum sum for numbers 10-99 is always < 162
- But 9² + 9² = 162 > 99, and smaller digits give smaller sums

### 6.2 Pluperfect Digital Invariants

The generalized form where the sum of k-th powers equals the number (k not necessarily equal to digit count).

## 7. Related Concepts

| Concept | Definition |
|---------|------------|
| Armstrong (Narcissistic) | Sum of d-th powers = number |
| Happy Number | Repeated digit² sum → 1 |
| Kaprekar Number | n² splits to parts summing to n |
| Automorphic Number | n² ends with n |

## 8. Applications

1. **Number theory**: Studying special number properties
2. **Programming exercises**: Common coding interview question
3. **Recreational mathematics**: Mathematical curiosities
4. **Digit manipulation**: Teaching digit extraction algorithms

## 9. References

1. [Wikipedia: Narcissistic number](https://en.wikipedia.org/wiki/Narcissistic_number)
2. [OEIS A005188](https://oeis.org/A005188) - Armstrong numbers
3. Hardy, G. H. & Wright, E. M. "An Introduction to the Theory of Numbers"
