# Levenshtein Distance (Edit Distance)

## 1. Overview

The Levenshtein distance, named after Vladimir Levenshtein who introduced it in 1965, measures the minimum number of single-character edits (insertions, deletions, or substitutions) required to change one string into another. It is one of the most fundamental string metrics used in spell checking, DNA analysis, and natural language processing.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given two strings $S$ of length $m$ and $T$ of length $n$, find the minimum number of edit operations to transform $S$ into $T$.

**Edit Operations:**
1. **Insert:** Add a character
2. **Delete:** Remove a character  
3. **Substitute:** Replace one character with another

### 2.2 Mathematical Model

**Recursive Definition:**

$$lev(i, j) = \begin{cases}
\max(i, j) & \text{if } \min(i, j) = 0 \\
\min \begin{cases}
lev(i-1, j) + 1 & \text{(deletion)} \\
lev(i, j-1) + 1 & \text{(insertion)} \\
lev(i-1, j-1) + [S_i \neq T_j] & \text{(match/substitute)}
\end{cases} & \text{otherwise}
\end{cases}$$

Where $[S_i \neq T_j]$ is 1 if the characters differ, 0 if they match.

### 2.3 Properties

1. **Metric Properties:**
   - $lev(S, T) \geq 0$ (non-negativity)
   - $lev(S, T) = 0 \iff S = T$ (identity)
   - $lev(S, T) = lev(T, S)$ (symmetry)
   - $lev(S, U) \leq lev(S, T) + lev(T, U)$ (triangle inequality)

2. **Bounds:**
   - $lev(S, T) \geq ||S| - |T||$ (at least length difference)
   - $lev(S, T) \leq \max(|S|, |T|)$ (at most replace all)

## 3. Algorithm Description

### 3.1 Intuition

Build a matrix where cell $(i, j)$ represents the edit distance between prefixes $S[0..i-1]$ and $T[0..j-1]$. Fill the matrix row by row, using previously computed values.

### 3.2 Pseudocode

**Naive (O(nm) space):**
```
function LEVENSHTEIN_NAIVE(s1, s2):
    m = len(s1)
    n = len(s2)
    
    // Initialize matrix
    D = matrix of size (m+1) × (n+1)
    
    for i = 0 to m:
        D[i][0] = i  // Delete all chars from s1
    for j = 0 to n:
        D[0][j] = j  // Insert all chars of s2
    
    // Fill matrix
    for i = 1 to m:
        for j = 1 to n:
            cost = 0 if s1[i-1] == s2[j-1] else 1
            D[i][j] = min(
                D[i-1][j] + 1,      // Deletion
                D[i][j-1] + 1,      // Insertion
                D[i-1][j-1] + cost  // Substitution/Match
            )
    
    return D[m][n]
```

**Optimized (O(n) space):**
```
function LEVENSHTEIN_OPTIMIZED(s1, s2):
    if s1 is empty: return len(s2)
    
    m = len(s1)
    n = len(s2)
    
    // Only need previous row
    prev_dist = [0, 1, 2, ..., m]
    
    for j = 1 to n:
        prev_substitution = prev_dist[0]
        prev_dist[0] = j  // Distance from empty s1 prefix
        
        for i = 1 to m:
            deletion = prev_dist[i-1] + 1
            insertion = prev_dist[i] + 1
            substitution = prev_substitution + (0 if s1[i-1] == s2[j-1] else 1)
            
            prev_substitution = prev_dist[i]  // Save before overwriting
            prev_dist[i] = min(deletion, insertion, substitution)
    
    return prev_dist[m]
```

### 3.3 Step-by-Step Example

**Strings:** "kitten" → "sitting"

**DP Matrix:**

|   |   | s | i | t | t | i | n | g |
|---|---|---|---|---|---|---|---|---|
|   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| k | 1 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| i | 2 | 2 | 1 | 2 | 3 | 4 | 5 | 6 |
| t | 3 | 3 | 2 | 1 | 2 | 3 | 4 | 5 |
| t | 4 | 4 | 3 | 2 | 1 | 2 | 3 | 4 |
| e | 5 | 5 | 4 | 3 | 2 | 2 | 3 | 4 |
| n | 6 | 6 | 5 | 4 | 3 | 3 | 2 | 3 |

**Result:** 3 (substitute k→s, substitute e→i, insert g)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Implementation | Time | Space |
|----------------|------|-------|
| Naive recursive | $O(3^{m+n})$ | $O(m+n)$ stack |
| Memoized | $O(mn)$ | $O(mn)$ |
| DP (matrix) | $O(mn)$ | $O(mn)$ |
| **DP (optimized)** | $O(mn)$ | $O(\min(m,n))$ |

### 4.2 Space Complexity

The optimized implementation only needs the previous row:
- Original: $O(mn)$ for full matrix
- Optimized: $O(n)$ using two rows (or one row with careful updates)
- Current implementation: $O(n)$ with single row

## 5. Implementation Notes

### 5.1 Rust Implementation

**Naive Version:**
```rust
pub fn naive_levenshtein_distance(string1: &str, string2: &str) -> usize {
    let distance_matrix: Vec<Vec<usize>> = (0..=string1.len())
        .map(|i| {
            (0..=string2.len())
                .map(|j| {
                    if i == 0 { j }
                    else if j == 0 { i }
                    else { 0 }
                })
                .collect()
        })
        .collect();

    let updated_matrix = (1..=string1.len()).fold(distance_matrix, |matrix, i| {
        (1..=string2.len()).fold(matrix, |mut inner_matrix, j| {
            let cost = usize::from(
                string1.as_bytes()[i - 1] != string2.as_bytes()[j - 1]
            );
            inner_matrix[i][j] = (inner_matrix[i - 1][j - 1] + cost)
                .min(inner_matrix[i][j - 1] + 1)
                .min(inner_matrix[i - 1][j] + 1);
            inner_matrix
        })
    });

    updated_matrix[string1.len()][string2.len()]
}
```

**Optimized Version:**
```rust
pub fn optimized_levenshtein_distance(string1: &str, string2: &str) -> usize {
    if string1.is_empty() {
        return string2.len();
    }
    
    let l1 = string1.len();
    let mut prev_dist: Vec<usize> = (0..=l1).collect();

    for (row, c2) in string2.chars().enumerate() {
        let mut prev_substitution_cost = prev_dist[0];
        prev_dist[0] = row + 1;

        for (col, c1) in string1.chars().enumerate() {
            let deletion_cost = prev_dist[col] + 1;
            let insertion_cost = prev_dist[col + 1] + 1;
            let substitution_cost = if c1 == c2 {
                prev_substitution_cost
            } else {
                prev_substitution_cost + 1
            };
            
            prev_substitution_cost = prev_dist[col + 1];
            prev_dist[col + 1] = _min3(deletion_cost, insertion_cost, substitution_cost);
        }
    }
    prev_dist[l1]
}
```

### 5.2 Edge Cases

| Case | Distance |
|------|----------|
| Both empty | 0 |
| One empty | Length of other |
| Identical strings | 0 |
| No common chars | max(m, n) |
| One is prefix of other | Length difference |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Spell Checking:**
   - Suggest corrections within edit distance threshold
   - Auto-correct functionality

2. **DNA Sequence Alignment:**
   - Compare genetic sequences
   - Find mutations

3. **Plagiarism Detection:**
   - Fuzzy document comparison
   - Near-duplicate detection

4. **Search Engines:**
   - "Did you mean?" suggestions
   - Fuzzy search

5. **Data Deduplication:**
   - Record linkage
   - Fuzzy matching in databases

### 6.2 Variations

| Variant | Difference |
|---------|------------|
| **Damerau-Levenshtein** | Adds transposition operation |
| **Hamming Distance** | Only substitutions, same-length strings |
| **LCS Distance** | Only insertions and deletions |
| **Weighted Edit Distance** | Different costs per operation |

## 7. Optimizations and Extensions

### 7.1 Early Termination

```rust
fn levenshtein_bounded(s1: &str, s2: &str, max_dist: usize) -> Option<usize> {
    // Return None if distance > max_dist
    // Use diagonal band optimization
}
```

### 7.2 Diagonal Band Optimization

When maximum distance $k$ is known, only compute cells within $k$ diagonals:
- Time: $O(kn)$ instead of $O(mn)$
- Useful for approximate matching

### 7.3 Edit Sequence Recovery

```rust
fn edit_operations(s1: &str, s2: &str) -> Vec<EditOp> {
    // Build full matrix
    // Backtrack to find actual operations
}
```

## 8. References

1. Levenshtein, V. I. (1966). "Binary Codes Capable of Correcting Deletions, Insertions and Reversals". *Soviet Physics Doklady*.
2. Wagner, R. A., & Fischer, M. J. (1974). "The String-to-String Correction Problem". *JACM*.
3. Navarro, G. (2001). "A Guided Tour to Approximate String Matching". *ACM Computing Surveys*.

## Implementation

See: [src/string/levenshtein_distance.rs](../../src/string/levenshtein_distance.rs)
