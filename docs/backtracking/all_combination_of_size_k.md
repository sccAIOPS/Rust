# Combinations (All Combinations of Size K)

## 1. Overview

**Combinations** are selections of items from a collection where order does not matter. This algorithm generates all possible combinations of size $k$ from a set of $n$ elements (specifically, integers $0$ to $n-1$).

Unlike permutations where order matters (AB ≠ BA), in combinations they are the same (AB = BA).

### Historical Context
- **1654**: Pascal and Fermat established combinatorial foundations
- **1713**: Bernoulli's *Ars Conjectandi* formalized combinatorics
- **Modern**: Core concept in probability, statistics, and algorithm design

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given integers $n$ and $k$ where $0 \leq k \leq n$, generate all subsets of $\{0, 1, 2, ..., n-1\}$ with exactly $k$ elements.

The number of such combinations is the **binomial coefficient**:

$$C(n, k) = \binom{n}{k} = \frac{n!}{k!(n-k)!}$$

### 2.2 Mathematical Model

**Input**: 
- $n$ - Upper bound (range is $0$ to $n-1$)
- $k$ - Size of each combination

**Output**: All $\binom{n}{k}$ combinations, each as a sorted vector

**Properties**:
- Elements within each combination are in ascending order
- No duplicate elements within a combination
- All combinations are distinct

### 2.3 Examples

| n | k | C(n,k) | Combinations |
|---|---|--------|--------------|
| 4 | 2 | 6 | `{0,1}, {0,2}, {0,3}, {1,2}, {1,3}, {2,3}` |
| 5 | 3 | 10 | `{0,1,2}, {0,1,3}, {0,1,4}, {0,2,3}, ...` |
| 3 | 3 | 1 | `{0,1,2}` |
| 3 | 0 | 1 | `{}` (empty set) |

### 2.4 Pascal's Triangle Relationship

$$\binom{n}{k} = \binom{n-1}{k-1} + \binom{n-1}{k}$$

This recurrence reflects the algorithm's structure: for each element, we either include it or exclude it.

## 3. Algorithm Description

### 3.1 Intuition

The backtracking approach builds combinations incrementally:
1. Start with an empty combination and position 0
2. At each step, choose the next element from remaining candidates
3. Candidates must be greater than the last chosen element (maintains sorted order)
4. When $k$ elements are chosen, record the combination
5. Backtrack by removing the last element and trying the next candidate

The key optimization: only consider elements from `start` to `n-k+index` (upper bound ensures enough elements remain).

### 3.2 Pseudocode

```
function generate_all_combinations(n, k):
    if n == 0 and k > 0:
        return Error(InvalidZeroRange)
    if k > n:
        return Error(KGreaterThanN)
    
    combinations = []
    current = array of size k
    backtrack(0, n, k, 0, current, combinations)
    return combinations

function backtrack(start, n, k, index, current, combinations):
    if index == k:
        combinations.add(copy(current))
        return
    
    // Upper bound: need (k - index) more elements
    // Last valid start position: n - (k - index) = n - k + index
    for num in start to (n - k + index):
        current[index] = num
        backtrack(num + 1, n, k, index + 1, current, combinations)
```

### 3.3 Step-by-Step Example

For `n=4, k=2`:

```
backtrack(start=0, index=0):
├─ num=0: current=[0,_]
│  backtrack(start=1, index=1):
│  ├─ num=1: current=[0,1] → OUTPUT
│  ├─ num=2: current=[0,2] → OUTPUT
│  └─ num=3: current=[0,3] → OUTPUT
│
├─ num=1: current=[1,_]
│  backtrack(start=2, index=1):
│  ├─ num=2: current=[1,2] → OUTPUT
│  └─ num=3: current=[1,3] → OUTPUT
│
└─ num=2: current=[2,_]
   backtrack(start=3, index=1):
   └─ num=3: current=[2,3] → OUTPUT

Result: [0,1], [0,2], [0,3], [1,2], [1,3], [2,3]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Total Combinations**: $\binom{n}{k}$
- **Work per Combination**: $O(k)$ to copy the combination
- **Total**: $O\left(\binom{n}{k} \times k\right)$

For fixed $k$: $O(n^k)$ since $\binom{n}{k} = O(n^k / k!)$

**Maximum value**: When $k = n/2$, $\binom{n}{n/2} \approx \frac{2^n}{\sqrt{n}}$

### 4.2 Space Complexity

- **Current Combination**: $O(k)$
- **Recursion Stack**: $O(k)$ depth
- **Output Storage**: $O\left(\binom{n}{k} \times k\right)$

**Auxiliary Space**: $O(k)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Error handling with custom enum
#[derive(Debug, PartialEq)]
pub enum CombinationError {
    KGreaterThanN,
    InvalidZeroRange,
}

// Pre-allocated combination vector
let mut current = vec![0; k];  // Avoids repeated allocations

// Result type for error handling
pub fn generate_all_combinations(n: usize, k: usize) 
    -> Result<Vec<Vec<usize>>, CombinationError>
```

**Key Patterns**:
- **Custom error type**: Clear error discrimination
- **Pre-allocation**: `vec![0; k]` allocated once, indices updated
- **Result type**: Idiomatic Rust error handling
- **Clone on output**: Only when adding to results

### 5.2 Edge Cases

| Case | n | k | Result |
|------|---|---|--------|
| Empty set, 0 elements | 0 | 0 | `Ok([[]])` (one empty combination) |
| Empty set, k > 0 | 0 | 1 | `Err(InvalidZeroRange)` |
| k > n | 3 | 4 | `Err(KGreaterThanN)` |
| k = n | 3 | 3 | `Ok([[0, 1, 2]])` |
| k = 1 | 5 | 1 | `Ok([[0], [1], [2], [3], [4]])` |

### 5.3 Potential Optimizations

1. **Iterative approach**: Avoid recursion stack overhead
2. **Bit manipulation**: For small n, use bitmask enumeration
3. **Iterator-based**: Lazy generation without storing all combinations
4. **Gray code order**: Generate combinations with minimal change between consecutive ones

```rust
// Bitmask approach for small n
fn combinations_bitmask(n: usize, k: usize) -> Vec<Vec<usize>> {
    (0..(1 << n))
        .filter(|mask| mask.count_ones() as usize == k)
        .map(|mask| (0..n).filter(|i| mask & (1 << i) != 0).collect())
        .collect()
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Feature Selection**: Choosing k features from n candidates
2. **Test Combinations**: Pairwise or n-wise testing
3. **Team Formation**: Selecting k members from n candidates
4. **Portfolio Selection**: Choosing k assets from n options
5. **A/B Testing**: Selecting variant combinations

### 6.2 Related Algorithms

| Algorithm | Description |
|-----------|-------------|
| Permutations | Ordered arrangements (more results) |
| Power Set | All subsets (2^n combinations) |
| Next Combination | Lexicographic iteration |
| Multiset Combinations | With repeated elements |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn backtrack(
    start: usize,
    n: usize,
    k: usize,
    index: usize,
    current: &mut Vec<usize>,
    combinations: &mut Vec<Vec<usize>>,
) {
    if index == k {
        combinations.push(current.clone());
        return;
    }

    for num in start..=(n - k + index) {
        current[index] = num;
        backtrack(num + 1, n, k, index + 1, current, combinations);
    }
}
```

**Analysis**:
- ✅ Efficient upper bound calculation `n - k + index`
- ✅ Pre-allocated current vector (indices updated, not pushed)
- ✅ Clean termination condition
- ✅ Proper input validation with custom error types
- ⚠️ Uses `clone()` for results; could use arena or indices

### 7.2 Upper Bound Optimization

The loop bound `start..=(n - k + index)` is crucial:
- We need to choose `k - index` more elements
- Last element we can choose is `n - 1`
- So we can start choosing at most at position `n - (k - index) = n - k + index`

This prevents exploring paths that can't lead to valid combinations.

### 7.3 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| `backtrack` calls | $O(\binom{n}{k})$ | ✅ One call per combination |
| Per-call work | $O(k)$ | ✅ Clone of size k |
| Total | $O(\binom{n}{k} \times k)$ | ✅ |

## 8. Testing

### 8.1 Test Cases Covered

```rust
test_generate_4_2: (4, 2) → 6 combinations
test_generate_4_3: (4, 3) → 4 combinations
test_generate_5_3: (5, 3) → 10 combinations
test_generate_5_1: (5, 1) → 5 combinations
test_empty: (0, 0) → [[]]
test_generate_n_eq_k: (3, 3) → [[0, 1, 2]]
test_generate_k_greater_than_n: (3, 4) → Err(KGreaterThanN)
test_zero_range_with_nonzero_k: (0, 1) → Err(InvalidZeroRange)
```

### 8.2 Test Coverage Analysis

- ✅ Standard cases (various n and k)
- ✅ Edge case: k = 0 (handled via n = k = 0)
- ✅ Edge case: k = n
- ✅ Edge case: k = 1
- ✅ Error case: k > n
- ✅ Error case: n = 0 with k > 0

## 9. Mathematical Properties

### 9.1 Combinatorial Identities

The implementation implicitly relies on these identities:

1. **Symmetry**: $\binom{n}{k} = \binom{n}{n-k}$
2. **Pascal's Rule**: $\binom{n}{k} = \binom{n-1}{k-1} + \binom{n-1}{k}$
3. **Sum**: $\sum_{k=0}^{n} \binom{n}{k} = 2^n$

### 9.2 Lexicographic Order

The output is in **lexicographic order**:
- `[0,1] < [0,2] < [0,3] < [1,2] < [1,3] < [2,3]`

This is a natural consequence of the algorithm's structure (iterating from smaller to larger values).

## 10. References

1. Knuth, D. E. (2011). *The Art of Computer Programming, Volume 4A*. Addison-Wesley.
2. Ruskey, F. (2003). *Combinatorial Generation*. University of Victoria.
3. Graham, R. L., Knuth, D. E., & Patashnik, O. (1994). *Concrete Mathematics*. Addison-Wesley.
4. [OEIS A007318](https://oeis.org/A007318) - Pascal's Triangle
