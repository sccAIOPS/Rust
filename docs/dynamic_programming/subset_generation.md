# Subset Generation

## 1. Overview

Subset Generation enumerates all possible subsets (power set) of a given set. While primarily a backtracking/recursion technique, it demonstrates fundamental concepts used in many DP problems involving subset enumeration.

**File**: `src/dynamic_programming/subset_generation.rs`

## 2. Mathematical Foundation

### 2.1 Power Set Definition

For a set $S$ with $n$ elements, the power set $\mathcal{P}(S)$ contains all $2^n$ subsets:

$$|\mathcal{P}(S)| = 2^n$$

### 2.2 Example

$S = \{1, 2, 3\}$

$\mathcal{P}(S) = \{\emptyset, \{1\}, \{2\}, \{3\}, \{1,2\}, \{1,3\}, \{2,3\}, \{1,2,3\}\}$

## 3. Algorithm Description

### 3.1 Recursive Backtracking

For each element, choose to include or exclude it:

```
FUNCTION generate_subsets(set, index, current, result)
    IF index = len(set) THEN
        result.append(copy of current)
        RETURN
    END IF
    
    // Exclude element at index
    generate_subsets(set, index + 1, current, result)
    
    // Include element at index
    current.append(set[index])
    generate_subsets(set, index + 1, current, result)
    current.pop()  // backtrack
END FUNCTION
```

### 3.2 Iterative (Bitmask)

Each subset corresponds to a binary number:

```
FUNCTION generate_subsets_bitmask(set)
    n ← len(set)
    result ← []
    
    FOR mask ← 0 TO 2^n - 1 DO
        subset ← []
        FOR i ← 0 TO n-1 DO
            IF bit i is set in mask THEN
                subset.append(set[i])
            END IF
        END FOR
        result.append(subset)
    END FOR
    
    RETURN result
END FUNCTION
```

### 3.3 Step-by-Step Example (Recursive)

**Input**: [1, 2, 3]

```
Call tree:
generate([1,2,3], 0, [])
├─ exclude 1: generate([1,2,3], 1, [])
│  ├─ exclude 2: generate([1,2,3], 2, [])
│  │  ├─ exclude 3: → []
│  │  └─ include 3: → [3]
│  └─ include 2: generate([1,2,3], 2, [2])
│     ├─ exclude 3: → [2]
│     └─ include 3: → [2,3]
└─ include 1: generate([1,2,3], 1, [1])
   ├─ exclude 2: generate([1,2,3], 2, [1])
   │  ├─ exclude 3: → [1]
   │  └─ include 3: → [1,3]
   └─ include 2: generate([1,2,3], 2, [1,2])
      ├─ exclude 3: → [1,2]
      └─ include 3: → [1,2,3]
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n × 2^n)** - Generate 2^n subsets, each up to n elements

### 4.2 Space Complexity
- **O(n × 2^n)** for storing all subsets
- **O(n)** for recursion stack (if not storing)

## 5. Implementation Notes

### 5.1 Rust Implementation (Recursive)

```rust
pub fn generate_subsets<T: Clone>(set: &[T]) -> Vec<Vec<T>> {
    let mut result = Vec::new();
    let mut current = Vec::new();
    backtrack(set, 0, &mut current, &mut result);
    result
}

fn backtrack<T: Clone>(
    set: &[T],
    index: usize,
    current: &mut Vec<T>,
    result: &mut Vec<Vec<T>>,
) {
    if index == set.len() {
        result.push(current.clone());
        return;
    }
    
    // Exclude
    backtrack(set, index + 1, current, result);
    
    // Include
    current.push(set[index].clone());
    backtrack(set, index + 1, current, result);
    current.pop();
}
```

### 5.2 Rust Implementation (Bitmask)

```rust
pub fn generate_subsets_bitmask<T: Clone>(set: &[T]) -> Vec<Vec<T>> {
    let n = set.len();
    let mut result = Vec::with_capacity(1 << n);
    
    for mask in 0..(1 << n) {
        let mut subset = Vec::new();
        for i in 0..n {
            if (mask & (1 << i)) != 0 {
                subset.push(set[i].clone());
            }
        }
        result.push(subset);
    }
    
    result
}
```

### 5.3 Iterator-Based

```rust
pub fn subset_iter<T: Clone>(set: &[T]) -> impl Iterator<Item = Vec<T>> + '_ {
    (0..(1 << set.len())).map(move |mask| {
        set.iter()
            .enumerate()
            .filter(|(i, _)| (mask & (1 << i)) != 0)
            .map(|(_, v)| v.clone())
            .collect()
    })
}
```

### 5.4 Edge Cases

| Input | Output |
|-------|--------|
| [] | [[]] (just empty set) |
| [1] | [[], [1]] |
| [1,1] | [[], [1], [1], [1,1]] (duplicates if not handled) |

## 6. Handling Duplicates

For sets with duplicate elements:

```rust
pub fn subsets_with_dup<T: Clone + Ord>(mut set: Vec<T>) -> Vec<Vec<T>> {
    set.sort();  // Required for duplicate handling
    let mut result = Vec::new();
    let mut current = Vec::new();
    backtrack_dup(&set, 0, &mut current, &mut result);
    result
}

fn backtrack_dup<T: Clone + Eq>(
    set: &[T],
    start: usize,
    current: &mut Vec<T>,
    result: &mut Vec<Vec<T>>,
) {
    result.push(current.clone());
    
    for i in start..set.len() {
        // Skip duplicates
        if i > start && set[i] == set[i - 1] {
            continue;
        }
        current.push(set[i].clone());
        backtrack_dup(set, i + 1, current, result);
        current.pop();
    }
}
```

## 7. Variants

### 7.1 Subsets of Size K

```rust
fn subsets_of_size_k<T: Clone>(set: &[T], k: usize) -> Vec<Vec<T>> {
    // Combinations C(n, k)
}
```

### 7.2 Subsets with Sum Constraint

```rust
fn subsets_with_sum(set: &[i32], target: i32) -> Vec<Vec<i32>> {
    // Filter by sum == target
}
```

### 7.3 Lexicographically Ordered

Generate subsets in sorted order using careful ordering.

## 8. Applications

1. **Combinatorics**: Counting problems
2. **Testing**: Generate all test cases
3. **Optimization**: Brute-force search space
4. **Database**: Query powerset for analysis
5. **Genetics**: Gene combination analysis

## 9. Relationship to DP

Subset generation is foundation for:
- Bitmask DP (state compression)
- Subset Sum problem
- Knapsack variations
- Traveling Salesman (visit subsets of cities)

## 10. Memory Optimization

For large sets, don't store all subsets:

```rust
// Lazy iterator approach
fn process_subsets<T: Clone, F>(set: &[T], mut processor: F)
where
    F: FnMut(&[T]),
{
    let mut current = Vec::new();
    fn backtrack<T: Clone, F: FnMut(&[T])>(
        set: &[T], index: usize, current: &mut Vec<T>, processor: &mut F
    ) {
        if index == set.len() {
            processor(current);
            return;
        }
        backtrack(set, index + 1, current, processor);
        current.push(set[index].clone());
        backtrack(set, index + 1, current, processor);
        current.pop();
    }
    backtrack(set, 0, &mut current, &mut processor);
}
```

## 11. Comparison of Approaches

| Approach | Time | Space | Best For |
|----------|------|-------|----------|
| Recursive | O(n·2^n) | O(n) stack | Clear logic |
| Bitmask | O(n·2^n) | O(1) extra | Small n |
| Iterator | O(n·2^n) | O(n) per | Lazy evaluation |

## 12. Practical Limits

| n | Subsets | Approx Size |
|---|---------|-------------|
| 10 | 1,024 | 1 KB |
| 20 | 1,048,576 | 1 MB |
| 25 | 33,554,432 | 33 MB |
| 30 | 1,073,741,824 | 1 GB |

For n > 25, consider:
- Streaming/lazy evaluation
- Pruning (branch and bound)
- Approximate methods

## 13. References

1. Knuth, D.E. "The Art of Computer Programming, Vol 4A"
2. [LeetCode Problem 78](https://leetcode.com/problems/subsets/)
3. [LeetCode Problem 90](https://leetcode.com/problems/subsets-ii/) (with duplicates)
