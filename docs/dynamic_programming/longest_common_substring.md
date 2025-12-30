# Longest Common Substring

## 1. Overview

The Longest Common Substring problem finds the longest contiguous sequence of characters that appears in both strings. Unlike LCS (subsequence), substrings must be contiguous in both input strings.

**File**: `src/dynamic_programming/longest_common_substring.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given two strings $S_1$ and $S_2$, find the longest string $S$ that is a contiguous substring of both $S_1$ and $S_2$.

### 2.2 Substring vs Subsequence

| Type | Contiguous | Example from "ABCDE" |
|------|------------|---------------------|
| Substring | Yes | "BCD", "ABC", "E" |
| Subsequence | No | "ACE", "BD", "ABCDE" |

### 2.3 Mathematical Model

Let $L[i][j]$ be the length of the longest common substring **ending at** position $i$ in $S_1$ and position $j$ in $S_2$.

**Recurrence**:
$$L[i][j] = \begin{cases}
0 & \text{if } i = 0 \text{ or } j = 0 \\
L[i-1][j-1] + 1 & \text{if } S_1[i] = S_2[j] \\
0 & \text{otherwise}
\end{cases}$$

**Result**: $\max_{i,j} L[i][j]$

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION longest_common_substring(s1, s2)
    m ← length(s1)
    n ← length(s2)
    
    L[0..m][0..n] ← 0
    max_len ← 0
    
    FOR i ← 1 TO m DO
        FOR j ← 1 TO n DO
            IF s1[i-1] = s2[j-1] THEN
                L[i][j] ← L[i-1][j-1] + 1
                max_len ← MAX(max_len, L[i][j])
            ELSE
                L[i][j] ← 0  // Reset on mismatch
            END IF
        END FOR
    END FOR
    
    RETURN max_len
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: s1 = "ABAB", s2 = "BABA"

**DP Table**:

|   | ε | B | A | B | A |
|---|---|---|---|---|---|
| ε | 0 | 0 | 0 | 0 | 0 |
| A | 0 | 0 | 1 | 0 | 1 |
| B | 0 | 1 | 0 | 2 | 0 |
| A | 0 | 0 | 2 | 0 | 3 |
| B | 0 | 1 | 0 | 3 | 0 |

**Result**: Maximum value is 3, corresponding to substring "ABA" or "BAB"

### 3.3 Key Difference from LCS

```
LCS Table (carries forward):     Substring Table (resets):
     B  A  B  A                      B  A  B  A
  ┌──┬──┬──┬──┬──┐                ┌──┬──┬──┬──┬──┐
A │0 │0 │1 │1 │1 │             A │0 │0 │1 │0 │1 │
B │0 │1 │1 │2 │2 │             B │0 │1 │0 │2 │0 │
A │0 │1 │2 │2 │3 │             A │0 │0 │2 │0 │3 │
B │0 │1 │2 │3 │3 │             B │0 │1 │0 │3 │0 │
  └──┴──┴──┴──┴──┘                └──┴──┴──┴──┴──┘
   Values grow               Values reset on mismatch
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **O(n × m)** where n and m are string lengths
- Single pass through the DP table

### 4.2 Space Complexity

- **O(n × m)** for full DP table
- Can be optimized to **O(min(n, m))** using rolling array

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn longest_common_substring(s1: &str, s2: &str) -> usize {
    let mut substr_len = vec![vec![0; s2.len() + 1]; s1.len() + 1];
    let mut max_len = 0;

    s1.as_bytes().iter().enumerate().for_each(|(i, &c1)| {
        s2.as_bytes().iter().enumerate().for_each(|(j, &c2)| {
            if c1 == c2 {
                substr_len[i + 1][j + 1] = substr_len[i][j] + 1;
                max_len = max_len.max(substr_len[i + 1][j + 1]);
            }
        });
    });

    max_len
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty strings | 0 |
| One empty | 0 |
| Identical strings | length of string |
| No common chars | 0 |
| Case sensitive | "ABC" vs "abc" = 0 |

## 6. Applications

1. **Plagiarism Detection**: Finding copied text passages
2. **DNA Analysis**: Finding common gene segments
3. **File Comparison**: Identifying identical file sections
4. **Spell Checking**: Finding similar word parts

## 7. References

1. [Wikipedia - Longest common substring problem](https://en.wikipedia.org/wiki/Longest_common_substring_problem)
2. Gusfield, D. (1997). *Algorithms on Strings, Trees, and Sequences*
