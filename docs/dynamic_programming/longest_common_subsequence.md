# Longest Common Subsequence (LCS)

## 1. Overview

The Longest Common Subsequence problem finds the longest sequence that can be derived from two sequences by deleting some elements without changing the order of remaining elements. Unlike substrings, subsequences don't need to be contiguous.

**File**: `src/dynamic_programming/longest_common_subsequence.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given two sequences $X = (x_1, x_2, \ldots, x_m)$ and $Y = (y_1, y_2, \ldots, y_n)$, find a longest sequence $Z$ that is a subsequence of both $X$ and $Y$.

### 2.2 Subsequence Definition

A sequence $Z = (z_1, z_2, \ldots, z_k)$ is a **subsequence** of $X$ if there exists indices $i_1 < i_2 < \ldots < i_k$ such that $z_j = x_{i_j}$ for all $j$.

**Example**: "ACE" is a subsequence of "ABCDE" but "AEC" is not.

### 2.3 Mathematical Model

Let $c[i][j]$ be the length of LCS of $X[1..i]$ and $Y[1..j]$.

**Recurrence Relation**:
$$c[i][j] = \begin{cases}
0 & \text{if } i = 0 \text{ or } j = 0 \\
c[i-1][j-1] + 1 & \text{if } x_i = y_j \\
\max(c[i-1][j], c[i][j-1]) & \text{if } x_i \neq y_j
\end{cases}$$

### 2.4 Optimal Substructure Proof

**Theorem**: LCS exhibits optimal substructure.

**Proof**: Let $Z = (z_1, \ldots, z_k)$ be an LCS of $X$ and $Y$.

1. If $x_m = y_n$, then $z_k = x_m = y_n$ and $Z[1..k-1]$ is an LCS of $X[1..m-1]$ and $Y[1..n-1]$.

2. If $x_m \neq y_n$:
   - If $z_k \neq x_m$, then $Z$ is an LCS of $X[1..m-1]$ and $Y$.
   - If $z_k \neq y_n$, then $Z$ is an LCS of $X$ and $Y[1..n-1]$.

This shows the problem can be decomposed into smaller subproblems. ∎

## 3. Algorithm Description

### 3.1 Intuition

Build a 2D table where each cell $(i, j)$ stores the LCS length of the first $i$ characters of $X$ and first $j$ characters of $Y$. When characters match, extend the previous diagonal solution. When they don't, take the better of excluding one character from either string.

### 3.2 Pseudocode

```
FUNCTION longest_common_subsequence(X, Y)
    m ← length(X)
    n ← length(Y)
    
    // Build LCS length table
    c[0..m][0..n] ← 0
    
    FOR i ← 1 TO m DO
        FOR j ← 1 TO n DO
            IF X[i] = Y[j] THEN
                c[i][j] ← c[i-1][j-1] + 1
            ELSE
                c[i][j] ← MAX(c[i-1][j], c[i][j-1])
            END IF
        END FOR
    END FOR
    
    // Reconstruct LCS
    RETURN reconstruct_lcs(X, Y, c)
END FUNCTION

FUNCTION reconstruct_lcs(X, Y, c)
    lcs ← []
    i ← m, j ← n
    
    WHILE i > 0 AND j > 0 DO
        IF X[i] = Y[j] THEN
            lcs.prepend(X[i])
            i ← i - 1
            j ← j - 1
        ELSE IF c[i-1][j] ≥ c[i][j-1] THEN
            i ← i - 1
        ELSE
            j ← j - 1
        END IF
    END WHILE
    
    RETURN lcs
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**: X = "ABCBDAB", Y = "BDCAB"

**DP Table**:

|   | ε | B | D | C | A | B |
|---|---|---|---|---|---|---|
| ε | 0 | 0 | 0 | 0 | 0 | 0 |
| A | 0 | 0 | 0 | 0 | 1 | 1 |
| B | 0 | 1 | 1 | 1 | 1 | 2 |
| C | 0 | 1 | 1 | 2 | 2 | 2 |
| B | 0 | 1 | 1 | 2 | 2 | 3 |
| D | 0 | 1 | 2 | 2 | 2 | 3 |
| A | 0 | 1 | 2 | 2 | 3 | 3 |
| B | 0 | 1 | 2 | 2 | 3 | 4 |

**Backtracking** (from c[7][5]=4):
- (7,5): B=B → Include B, go to (6,4)
- (6,4): A=A → Include A, go to (5,3)
- (5,3): D≠C, c[4,3]≥c[5,2] → go to (4,3)
- (4,3): B≠C, c[3,3]≥c[4,2] → go to (3,3)
- (3,3): C=C → Include C, go to (2,2)
- (2,2): B≠D, c[1,2]<c[2,1] → go to (2,1)
- (2,1): B=B → Include B, go to (1,0)

**Result**: LCS = "BCAB" (length 4)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Phase | Complexity |
|-------|------------|
| Table filling | O(m × n) |
| Reconstruction | O(m + n) |
| **Total** | **O(m × n)** |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| DP Table | O(m × n) |
| Result string | O(min(m, n)) |
| **Total** | **O(m × n)** |

**Space Optimization**: Can reduce to O(min(m, n)) using rolling array (loses backtracking).

### 4.3 Proof of Time Complexity

The algorithm performs:
- Two nested loops: $O(m \times n)$ iterations
- Each iteration does $O(1)$ work (comparison, array access)
- Total: $O(m \times n)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Unicode Handling**:
```rust
let first_seq_chars = first_seq.chars().collect::<Vec<char>>();
```

The implementation properly handles Unicode by converting strings to character vectors, ensuring multi-byte characters are handled correctly.

**Functional Style**:
```rust
(1..=first_seq_len).for_each(|i| {
    (1..=second_seq_len).for_each(|j| {
        lcs_lengths[i][j] = if first_seq_chars[i-1] == second_seq_chars[j-1] {
            lcs_lengths[i-1][j-1] + 1
        } else {
            lcs_lengths[i-1][j].max(lcs_lengths[i][j-1])
        };
    });
});
```

### 5.2 Important Note: Non-Symmetry

⚠️ **The LCS function is not symmetric!**

```rust
longest_common_subsequence("hello, world!", "world, hello!")  // "hello!"
longest_common_subsequence("world, hello!", "hello, world!")  // "world!"
```

Both results are valid LCS of the same length, but the reconstruction path differs based on input order.

### 5.3 Edge Cases

| Case | Result |
|------|--------|
| Empty strings | "" |
| One empty | "" |
| Identical strings | The string itself |
| No common chars | "" |
| Single char match | That character |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Diff Tools**: `diff`, `git diff` use LCS-based algorithms
2. **Merge Conflicts**: 3-way merge uses LCS
3. **Version Control**: Tracking file changes
4. **Code Plagiarism Detection**: Comparing code similarity

### 6.2 Industry Applications

| Industry | Application |
|----------|-------------|
| Bioinformatics | DNA/Protein sequence alignment |
| Text Processing | Document comparison |
| Speech Recognition | Error correction |
| Data Compression | Delta encoding |

### 6.3 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Longest Common Substring | Contiguous version |
| Edit Distance | Related DP structure |
| Shortest Common Supersequence | SCS = m + n - LCS |
| Longest Increasing Subsequence | Single sequence version |

## 7. Variants and Extensions

### All LCS Enumeration

Multiple LCS of same length may exist:
```
X = "ABCD", Y = "ACBD"
LCS options: "ABD", "ACD" (both length 3)
```

### LCS of Multiple Sequences

The k-sequence LCS problem:
- Time: O(n^k) for k sequences of length n
- NP-hard for arbitrary k

### Weighted LCS

Each character has a weight; maximize total weight instead of length.

## 8. Visualization

```
String X:  A B C B D A B
String Y:  B D C A B

DP Table Progression:
────────────────────────

     B  D  C  A  B
  ┌──┬──┬──┬──┬──┬──┐
  │ 0│ 0│ 0│ 0│ 0│ 0│
A ├──┼──┼──┼──┼──┼──┤
  │ 0│ 0│ 0│ 0│⬊1│ 1│  A matches A
B ├──┼──┼──┼──┼──┼──┤
  │ 0│⬊1│ 1│ 1│ 1│⬊2│  B matches B
C ├──┼──┼──┼──┼──┼──┤
  │ 0│ 1│ 1│⬊2│ 2│ 2│  C matches C
B ├──┼──┼──┼──┼──┼──┤
  │ 0│⬊1│ 1│ 2│ 2│⬊3│  B matches B
D ├──┼──┼──┼──┼──┼──┤
  │ 0│ 1│⬊2│ 2│ 2│ 3│  D matches D
A ├──┼──┼──┼──┼──┼──┤
  │ 0│ 1│ 2│ 2│⬊3│ 3│  A matches A
B ├──┼──┼──┼──┼──┼──┤
  │ 0│⬊1│ 2│ 2│ 3│⬊4│  B matches B
  └──┴──┴──┴──┴──┴──┘

⬊ = diagonal move (character match)
LCS Length = 4, LCS = "BCAB"
```

## 9. References

1. Cormen, T. H. et al. (2009). *Introduction to Algorithms*, Chapter 15
2. [Wikipedia - Longest common subsequence problem](https://en.wikipedia.org/wiki/Longest_common_subsequence_problem)
3. Hirschberg, D. S. (1975). A linear space algorithm for computing maximal common subsequences
