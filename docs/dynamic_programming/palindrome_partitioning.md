# Palindrome Partitioning (Minimum Cuts)

## 1. Overview

The Palindrome Partitioning problem finds the minimum number of cuts needed to partition a string into palindromic substrings. Every string can be partitioned into single characters, but we seek the optimal solution.

**File**: `src/dynamic_programming/palindrome_partitioning.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given string $s$ of length $n$, find minimum $k$ such that $s$ can be split into $k+1$ palindromic substrings.

$$\min\{k : s = p_1 p_2 \ldots p_{k+1}, \text{ each } p_i \text{ is palindrome}\}$$

### 2.2 Recurrence

Let $cuts[i]$ = minimum cuts for $s[0..i]$

$$cuts[i] = \begin{cases}
0 & \text{if } s[0..i] \text{ is palindrome} \\
\min_{j<i, s[j+1..i] \text{ is palindrome}} (cuts[j] + 1) & \text{otherwise}
\end{cases}$$

## 3. Algorithm Description

### 3.1 Two-Phase Approach

**Phase 1**: Precompute palindrome table
**Phase 2**: DP for minimum cuts

### 3.2 Pseudocode

```
FUNCTION min_cuts(s)
    n ← len(s)
    // Phase 1: Build palindrome table
    is_palindrome[0..n][0..n] ← false
    FOR len ← 1 TO n DO
        FOR i ← 0 TO n-len DO
            j ← i + len - 1
            IF len = 1 THEN
                is_palindrome[i][j] ← true
            ELSE IF len = 2 THEN
                is_palindrome[i][j] ← (s[i] = s[j])
            ELSE
                is_palindrome[i][j] ← (s[i] = s[j]) AND is_palindrome[i+1][j-1]
            END IF
        END FOR
    END FOR
    
    // Phase 2: Compute minimum cuts
    cuts[0..n] ← i  // worst case: n-1 cuts
    FOR i ← 0 TO n-1 DO
        IF is_palindrome[0][i] THEN
            cuts[i] ← 0
        ELSE
            FOR j ← 0 TO i-1 DO
                IF is_palindrome[j+1][i] THEN
                    cuts[i] ← min(cuts[i], cuts[j] + 1)
                END IF
            END FOR
        END IF
    END FOR
    
    RETURN cuts[n-1]
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**: s = "aab"

**Phase 1: Palindrome Table**
```
    0   1   2
   'a' 'a' 'b'
0   T   T   F
1       T   F
2           T

T = True (is palindrome)
- s[0..0] = "a" ✓
- s[1..1] = "a" ✓
- s[2..2] = "b" ✓
- s[0..1] = "aa" ✓
- s[1..2] = "ab" ✗
- s[0..2] = "aab" ✗
```

**Phase 2: Minimum Cuts**
```
cuts[0] = 0 (single char "a" is palindrome)
cuts[1] = 0 (s[0..1] = "aa" is palindrome)
cuts[2]: s[0..2] = "aab" not palindrome
  - j=0: s[1..2] = "ab" not palindrome
  - j=1: s[2..2] = "b" is palindrome
         cuts[2] = min(2, cuts[1] + 1) = 1
```

**Result**: 1 cut → "aa" | "b"

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n²)** - Two passes, each O(n²)

### 4.2 Space Complexity
- **O(n²)** for palindrome table
- Can be optimized to O(n) with Manacher's algorithm

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn min_palindrome_partitions(s: &str) -> usize {
    let s: Vec<char> = s.chars().collect();
    let n = s.len();
    if n == 0 { return 0; }

    // Palindrome check table
    let mut is_palindrome = vec![vec![false; n]; n];
    
    // All single chars are palindromes
    for i in 0..n {
        is_palindrome[i][i] = true;
    }
    
    // Check length 2 to n
    for len in 2..=n {
        for i in 0..=n-len {
            let j = i + len - 1;
            is_palindrome[i][j] = s[i] == s[j] 
                && (len == 2 || is_palindrome[i+1][j-1]);
        }
    }

    // DP for minimum cuts
    let mut cuts = vec![0; n];
    for i in 0..n {
        if is_palindrome[0][i] {
            cuts[i] = 0;
        } else {
            cuts[i] = i; // worst case
            for j in 0..i {
                if is_palindrome[j+1][i] {
                    cuts[i] = cuts[i].min(cuts[j] + 1);
                }
            }
        }
    }
    
    cuts[n-1]
}
```

### 5.2 Edge Cases

| Input | Output | Reason |
|-------|--------|--------|
| "" | 0 | Empty string |
| "a" | 0 | Single char is palindrome |
| "aa" | 0 | Already palindrome |
| "ab" | 1 | "a" \| "b" |
| "aab" | 1 | "aa" \| "b" |
| "abc" | 2 | "a" \| "b" \| "c" |

## 6. Optimizations

### 6.1 Expand Around Center

Instead of O(n²) palindrome table, use center expansion:

```rust
fn expand(s: &[char], mut left: i32, mut right: i32) -> (usize, usize) {
    while left >= 0 && right < s.len() as i32 
          && s[left as usize] == s[right as usize] {
        left -= 1;
        right += 1;
    }
    ((left + 1) as usize, (right - 1) as usize)
}
```

### 6.2 Manacher's Algorithm

O(n) palindrome detection for ultimate optimization.

## 7. Applications

1. **Text Compression**: Breaking text into palindromic chunks
2. **Bioinformatics**: DNA sequence analysis
3. **Natural Language**: Identifying repeated structures

## 8. Related Problems

| Problem | Description |
|---------|-------------|
| Palindrome Partitioning II | Return all partitions |
| Longest Palindromic Substring | Find longest single palindrome |
| Shortest Palindrome | Min chars to add to front |

## 9. Variant: Return All Partitions

Instead of count, return actual partitions using backtracking:

```
"aab" → [["a","a","b"], ["aa","b"]]
```

## 10. References

1. [LeetCode Problem 132](https://leetcode.com/problems/palindrome-partitioning-ii/)
2. Manacher, G. (1975). "A New Linear-Time Algorithm for Finding the Smallest Initial Palindrome of a String"
