# Find Unique Number

## 1. Overview

The **Find Unique Number** algorithm identifies the single element that appears exactly once in an array where every other element appears exactly twice. Using XOR bit manipulation, it achieves O(n) time complexity with O(1) space—a remarkably elegant solution.

### Historical Context

This problem, often called "Single Number," became a popular interview question due to its elegant XOR solution. It demonstrates how understanding fundamental bit properties can lead to optimal algorithms that seem almost magical.

### Key Insight

XOR's self-inverse property (`a ^ a = 0`) means paired elements cancel each other out. When we XOR all elements together, only the unique (unpaired) element remains.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: An array of $n$ integers where every element except one appears exactly twice  
**Output**: The element that appears only once

**Constraints**:
- All elements except one appear exactly twice
- The unique element appears exactly once
- Array is non-empty

### 2.2 Mathematical Model

#### XOR Properties Revisited

| Property | Expression | Application |
|----------|------------|-------------|
| Self-inverse | $a \oplus a = 0$ | Pairs cancel |
| Identity | $a \oplus 0 = a$ | Zero doesn't affect result |
| Commutative | $a \oplus b = b \oplus a$ | Order independence |
| Associative | $(a \oplus b) \oplus c = a \oplus (b \oplus c)$ | Grouping independence |

#### The Cancellation Chain

Given array $A = [a_1, a_1, a_2, a_2, ..., a_k, a_k, u]$ where $u$ is unique:

$$\bigoplus_{i=1}^{n} A[i] = (a_1 \oplus a_1) \oplus (a_2 \oplus a_2) \oplus ... \oplus (a_k \oplus a_k) \oplus u$$
$$= 0 \oplus 0 \oplus ... \oplus 0 \oplus u = u$$

### 2.3 Correctness Proof

**Theorem**: XORing all elements of an array where each element except one appears twice yields the unique element.

**Proof**:

Let $A$ be the array with elements $\{x_1, x_1, x_2, x_2, ..., x_k, x_k, u\}$ (in any order).

By commutativity and associativity of XOR, we can reorder:
$$\bigoplus A = (x_1 \oplus x_1) \oplus (x_2 \oplus x_2) \oplus ... \oplus (x_k \oplus x_k) \oplus u$$

By self-inverse property:
$$= 0 \oplus 0 \oplus ... \oplus 0 \oplus u$$

By identity property:
$$= u$$ $\square$

## 3. Algorithm Description

### 3.1 Intuition

Think of XOR as a "toggle" operation. Each number "toggles" certain bits. When you see the same number twice, it toggles the same bits twice, returning them to their original state (effectively canceling out). The unique number toggles its bits only once, so those bits remain in the final result.

### 3.2 Pseudocode

```
function find_unique_number(arr):
    if arr is empty:
        return ERROR("input list must not be empty")
    
    result = 0
    for each num in arr:
        result = result XOR num
    
    return result
```

### 3.3 Step-by-Step Example

**Example**: Find unique in `[4, 5, 4, 6, 6]`

```
Array: [4, 5, 4, 6, 6]
Expected unique: 5

Binary representations:
4 = 100
5 = 101
6 = 110

XOR progression:
Start:  result = 0 = 000

XOR 4:  000 ^ 100 = 100 (4)
XOR 5:  100 ^ 101 = 001 (1)
XOR 4:  001 ^ 100 = 101 (5)
XOR 6:  101 ^ 110 = 011 (3)
XOR 6:  011 ^ 110 = 101 (5)

Result: 5 ✓
```

**Grouped View**:
```
(4 ^ 4) ^ (6 ^ 6) ^ 5
= 0 ^ 0 ^ 5
= 5 ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| All cases | $O(n)$ | Single pass through array |

### 4.2 Space Complexity

| Metric | Complexity |
|--------|------------|
| Auxiliary space | $O(1)$ |
| Stack space | $O(1)$ |

This is optimal—we must examine each element at least once (lower bound $\Omega(n)$), and we use no extra data structures.

### 4.3 Comparison with Alternatives

| Method | Time | Space | Notes |
|--------|------|-------|-------|
| XOR | O(n) | O(1) | Optimal |
| Hash Map | O(n) | O(n) | Count occurrences |
| Sorting | O(n log n) | O(1) | Check neighbors |
| Brute Force | O(n²) | O(1) | Check each pair |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn find_unique_number(arr: &[i32]) -> Result<i32, String> {
    if arr.is_empty() {
        return Err("input list must not be empty".to_string());
    }

    let result = arr.iter().fold(0, |acc, &num| acc ^ num);
    Ok(result)
}
```

**Key Implementation Details**:
- Uses `fold` for functional, concise implementation
- Returns `Result` for proper error handling
- Works with signed integers (XOR handles negative numbers correctly)

### 5.2 Edge Cases

| Input | Expected Output | Reasoning |
|-------|-----------------|-----------|
| `[]` | Error | Empty array |
| `[7]` | 7 | Single element is unique |
| `[1, 1, 2, 2, 3]` | 3 | Standard case |
| `[-1, -1, -2, -2, -3]` | -3 | Negative numbers work |
| `[0, 1, 1]` | 0 | Zero is the unique |
| `[1000, 2000, 1000, 3000, 3000]` | 2000 | Large numbers |

### 5.3 Alternative Implementations

**Explicit Loop**:
```rust
fn find_unique_loop(arr: &[i32]) -> i32 {
    let mut result = 0;
    for &num in arr {
        result ^= num;
    }
    result
}
```

**Using reduce**:
```rust
fn find_unique_reduce(arr: &[i32]) -> Option<i32> {
    arr.iter().copied().reduce(|a, b| a ^ b)
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Error Detection**
   - Finding corrupted bit in parity-based systems
   - Detecting single-bit errors in RAID systems

2. **Data Deduplication**
   - Identifying unique records in paired datasets
   - Finding unpaired transactions

3. **Game Development**
   - Finding odd player in matchmaking
   - Detecting unmatched game events

4. **Testing**
   - Verifying paired operations (open/close, alloc/free)
   - Finding unmatched log entries

5. **Networking**
   - Finding unpaired request/response
   - Detecting lost acknowledgments

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| [Find Missing Number](find_missing_number.md) | Same XOR technique, different setup |
| Single Number II | Extension: one unique among triples |
| Single Number III | Extension: two unique elements |

## 7. Extensions

### Single Number II: One Among Triples

When every element appears three times except one:

```rust
fn single_number_ii(nums: &[i32]) -> i32 {
    let mut ones = 0;
    let mut twos = 0;
    
    for &num in nums {
        ones = (ones ^ num) & !twos;
        twos = (twos ^ num) & !ones;
    }
    
    ones
}
```

### Single Number III: Two Unique Elements

When two elements appear once and others twice:

```rust
fn single_number_iii(nums: &[i32]) -> (i32, i32) {
    // XOR all gives x ^ y where x, y are the unique elements
    let xor_all = nums.iter().fold(0, |acc, &x| acc ^ x);
    
    // Find rightmost set bit (x and y differ here)
    let diff_bit = xor_all & (-xor_all);
    
    let mut x = 0;
    let mut y = 0;
    
    for &num in nums {
        if num & diff_bit != 0 {
            x ^= num;
        } else {
            y ^= num;
        }
    }
    
    (x, y)
}
```

## 8. Mathematical Elegance

### Why XOR is Perfect for This Problem

1. **Involutory**: XOR is its own inverse
2. **Abelian Group**: Forms a group under XOR with 0 as identity
3. **No Information Loss**: Unlike addition (overflow) or multiplication
4. **Bit Independence**: Each bit position operates independently

### Generalization: Finding Element Appearing Odd Times

If one element appears an odd number of times and all others appear an even number of times, XOR still finds it:

```rust
fn find_odd_occurrence(arr: &[i32]) -> i32 {
    arr.iter().fold(0, |acc, &x| acc ^ x)
}

// Works because: x ^ x = 0 (even occurrences cancel)
// Remaining: x ^ x ^ x = x (odd occurrence survives)
```

## 9. Common Pitfalls

### Pitfall 1: Assuming Positive Numbers Only
```rust
// XOR works fine with negative numbers
let arr = [-1, -1, -5];
let unique = find_unique_number(&arr);  // Returns -5 correctly
```

### Pitfall 2: Confusing with "Find Duplicate"
```rust
// "Find unique" (one appears once, rest twice):
// [1, 1, 2, 2, 3] → 3

// "Find duplicate" (one appears twice, rest once):
// [1, 2, 2, 3, 4] → 2 (different problem!)

// XOR finds the duplicate only if n+1 elements from range [1,n]
```

### Pitfall 3: Expecting Order Preservation
```rust
// XOR is commutative - order doesn't matter
let arr1 = [4, 5, 4, 6, 6];
let arr2 = [5, 4, 6, 4, 6];
// Both return 5
```

### Pitfall 4: Empty Array Handling
```rust
// Using reduce without handling empty case
fn buggy(arr: &[i32]) -> i32 {
    arr.iter().copied().reduce(|a, b| a ^ b).unwrap()  // Panics if empty!
}

// Use fold with initial value or proper error handling
fn correct(arr: &[i32]) -> Result<i32, String> {
    if arr.is_empty() {
        return Err("empty array".to_string());
    }
    Ok(arr.iter().fold(0, |acc, &x| acc ^ x))
}
```

## 10. Visualization

```
Array: [2, 3, 2, 4, 4]

Bit positions (8-bit):
2 = 00000010
3 = 00000011
2 = 00000010
4 = 00000100
4 = 00000100

XOR accumulation:
00000000  (start)
00000010  (^ 2)
00000001  (^ 3)
00000011  (^ 2) - note: 2 cancelled
00000111  (^ 4)
00000011  (^ 4) - note: 4 cancelled

Result: 00000011 = 3 ✓

Only the bits of the unique number (3) survive!
```

## 11. References

1. Warren, H. S. (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley.
2. LeetCode Problem 136: Single Number
3. Knuth, D. E. (2011). *The Art of Computer Programming, Vol. 4A*. Addison-Wesley.
