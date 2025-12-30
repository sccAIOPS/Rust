# Pascal's Triangle

## 1. Overview

**Pascal's triangle** is a triangular array where each number is the sum of the two numbers directly above it. It reveals binomial coefficients and has numerous applications in combinatorics and algebra.

**File**: `src/math/pascal_triangle.rs`

## 2. Mathematical Foundation

### 2.1 Definition

Row $n$, position $k$:
$$P(n, k) = \binom{n}{k} = \frac{n!}{k!(n-k)!}$$

### 2.2 Recurrence Relation

$$\binom{n}{k} = \binom{n-1}{k-1} + \binom{n-1}{k}$$

### 2.3 Structure

```
Row 0:           1
Row 1:          1 1
Row 2:         1 2 1
Row 3:        1 3 3 1
Row 4:       1 4 6 4 1
Row 5:      1 5 10 10 5 1
```

### 2.4 Properties

| Property | Description |
|----------|-------------|
| Symmetry | $\binom{n}{k} = \binom{n}{n-k}$ |
| Row sum | $2^n$ |
| Edges | Always 1 |
| Powers of 11 | Rows give digits: 1, 11, 121, 1331, ... |

## 3. Algorithm

### 3.1 Direct Calculation Method

For each row, use the formula:
$$C(n, k) = C(n, k-1) \times \frac{n-k+1}{k}$$

### 3.2 Pseudocode

```
function pascal_triangle(num_rows):
    result = []
    for i from 1 to num_rows:
        row = [1]
        coeff = 1
        for k from 1 to i-1:
            coeff = coeff * (i - k) / k
            row.append(coeff)
        result.append(row)
    return result
```

## 4. Implementation

```rust
pub fn pascal_triangle(num_rows: i32) -> Vec<Vec<i32>> {
    let mut ans: Vec<Vec<i32>> = vec![];

    for i in 1..=num_rows {
        let mut vec: Vec<i32> = vec![1];

        let mut res: i32 = 1;
        for k in 1..i {
            res *= i - k;
            res /= k;
            vec.push(res);
        }
        ans.push(vec);
    }

    ans
}
```

### 4.1 Usage Example

```rust
let triangle = pascal_triangle(5);
// Result:
// [
//   [1],
//   [1, 1],
//   [1, 2, 1],
//   [1, 3, 3, 1],
//   [1, 4, 6, 4, 1]
// ]
```

## 5. Complexity

- **Time**: O(n²) - filling n rows with up to n elements each
- **Space**: O(n²) - storing the entire triangle

## 6. Applications

1. **Binomial coefficients**: $(a + b)^n$ expansion
2. **Combinatorics**: Counting combinations
3. **Probability**: Binomial distribution coefficients
4. **Number patterns**: Fibonacci numbers (diagonal sums)
5. **Polynomial interpolation**: Newton's forward differences

## 7. Hidden Patterns

### 7.1 Fibonacci Sequence

Diagonal sums yield Fibonacci numbers:
```
        1                 → 1
       1 1                → 1
      1 2 1               → 2
     1 3 3 1              → 3
    1 4 6 4 1             → 5
   1 5 10 10 5 1          → 8
```

### 7.2 Powers of 2

Each row sums to a power of 2:
- Row 0: 1 = 2⁰
- Row 3: 1+3+3+1 = 8 = 2³
- Row n: sum = 2ⁿ

## 8. Related Algorithms

- [Binomial Coefficient](binomial_coefficient.md) - Single coefficient calculation
- [Factorial](factorial.md) - Used in direct formula
- Catalan numbers - Can be derived from Pascal's triangle

## 9. References

1. [Wikipedia: Pascal's triangle](https://en.wikipedia.org/wiki/Pascal%27s_triangle)
2. Graham, R. L., et al. "Concrete Mathematics"
