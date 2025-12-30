# Smith-Waterman Algorithm (Local Sequence Alignment)

## 1. Overview

The Smith-Waterman algorithm performs local sequence alignment to find the best matching subsequences between two sequences. It's fundamental in bioinformatics for identifying similar regions in DNA, RNA, or protein sequences.

**File**: `src/dynamic_programming/smith_waterman.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- Two sequences $A[1..m]$ and $B[1..n]$
- Match score $M$ (positive)
- Mismatch penalty $X$ (negative)
- Gap penalty $G$ (negative)

Find: Highest-scoring local alignment between subsequences of A and B.

### 2.2 Recurrence

Let $H[i][j]$ = maximum alignment score ending at $A[i]$ and $B[j]$.

$$H[i][j] = \max \begin{cases}
0 & \text{(restart alignment)} \\
H[i-1][j-1] + s(A[i], B[j]) & \text{(match/mismatch)} \\
H[i-1][j] + G & \text{(gap in B)} \\
H[i][j-1] + G & \text{(gap in A)}
\end{cases}$$

where $s(a, b) = M$ if $a = b$, else $X$.

### 2.3 Key Difference from Needleman-Wunsch

| Aspect | Needleman-Wunsch | Smith-Waterman |
|--------|------------------|----------------|
| Alignment | Global | Local |
| Initialization | First row/col with gaps | All zeros |
| Minimum score | Can be negative | Clamped to 0 |
| Traceback start | Bottom-right | Maximum cell |
| Traceback end | Top-left | Any cell with 0 |

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION smith_waterman(A, B, match, mismatch, gap)
    m ← len(A)
    n ← len(B)
    H[0..m][0..n] ← 0  // Initialize all to 0
    
    max_score ← 0
    max_pos ← (0, 0)
    
    FOR i ← 1 TO m DO
        FOR j ← 1 TO n DO
            score_diag ← H[i-1][j-1] + (match IF A[i]=B[j] ELSE mismatch)
            score_up ← H[i-1][j] + gap
            score_left ← H[i][j-1] + gap
            
            H[i][j] ← max(0, score_diag, score_up, score_left)
            
            IF H[i][j] > max_score THEN
                max_score ← H[i][j]
                max_pos ← (i, j)
            END IF
        END FOR
    END FOR
    
    RETURN (max_score, traceback(H, A, B, max_pos))
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**:
- A = "ACACACTA"
- B = "AGCACACA"
- Match = +2, Mismatch = -1, Gap = -1

**Scoring Matrix** (partial):
```
      -  A  G  C  A  C  A  C  A
   -  0  0  0  0  0  0  0  0  0
   A  0  2  1  0  2  1  2  1  2
   C  0  1  1  3  2  4  3  4  3
   A  0  2  1  2  5  4  6  5  6
   C  0  1  1  3  4  7  6  8  7
   A  0  2  1  2  5  6  9  8 10
   C  0  1  1  3  4  7  8 11 10
   T  0  0  0  2  3  6  7 10 10
   A  0  2  1  1  4  5  8  9 12 ← MAX
```

**Maximum Score**: 12 at position (8, 8)

**Traceback**:
```
Starting from H[8][8] = 12, trace back:
A: A-CACACTA
B: AGCACACA

Aligned region:
A: CACACTA
   |||||||
B: CACACAA
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(m × n)** for matrix filling
- **O(m + n)** for traceback

### 4.2 Space Complexity
- **O(m × n)** for full matrix
- **O(min(m, n))** with space optimization (score only)

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn smith_waterman(
    query: &str,
    reference: &str,
    match_score: i32,
    mismatch_penalty: i32,
    gap_penalty: i32,
) -> (i32, String, String) {
    let query: Vec<char> = query.chars().collect();
    let reference: Vec<char> = reference.chars().collect();
    let m = query.len();
    let n = reference.len();
    
    let mut matrix = vec![vec![0i32; n + 1]; m + 1];
    let mut max_score = 0;
    let mut max_pos = (0, 0);
    
    for i in 1..=m {
        for j in 1..=n {
            let match_mismatch = if query[i-1] == reference[j-1] {
                matrix[i-1][j-1] + match_score
            } else {
                matrix[i-1][j-1] + mismatch_penalty
            };
            
            matrix[i][j] = 0
                .max(match_mismatch)
                .max(matrix[i-1][j] + gap_penalty)
                .max(matrix[i][j-1] + gap_penalty);
            
            if matrix[i][j] > max_score {
                max_score = matrix[i][j];
                max_pos = (i, j);
            }
        }
    }
    
    // Traceback
    let (align_a, align_b) = traceback(&matrix, &query, &reference, 
                                        max_pos, match_score, 
                                        mismatch_penalty, gap_penalty);
    
    (max_score, align_a, align_b)
}
```

### 5.2 Traceback

```rust
fn traceback(
    matrix: &[Vec<i32>],
    query: &[char],
    reference: &[char],
    start: (usize, usize),
    match_score: i32,
    mismatch: i32,
    gap: i32,
) -> (String, String) {
    let (mut i, mut j) = start;
    let mut align_a = String::new();
    let mut align_b = String::new();
    
    while i > 0 && j > 0 && matrix[i][j] > 0 {
        let current = matrix[i][j];
        let diag_score = if query[i-1] == reference[j-1] { match_score } else { mismatch };
        
        if current == matrix[i-1][j-1] + diag_score {
            align_a.push(query[i-1]);
            align_b.push(reference[j-1]);
            i -= 1;
            j -= 1;
        } else if current == matrix[i-1][j] + gap {
            align_a.push(query[i-1]);
            align_b.push('-');
            i -= 1;
        } else {
            align_a.push('-');
            align_b.push(reference[j-1]);
            j -= 1;
        }
    }
    
    (align_a.chars().rev().collect(), align_b.chars().rev().collect())
}
```

### 5.3 Scoring Matrices

For proteins, use substitution matrices (BLOSUM62, PAM250):

```rust
fn blosum62_score(a: char, b: char) -> i32 {
    // Lookup in BLOSUM62 matrix
    BLOSUM62[(a as usize, b as usize)]
}
```

## 6. Applications

1. **Genomics**: Finding conserved regions in DNA
2. **Proteomics**: Identifying protein domains
3. **Evolution**: Detecting homologous sequences
4. **Drug Discovery**: Finding similar binding sites
5. **Forensics**: DNA matching

## 7. Variants and Extensions

### 7.1 Affine Gap Penalties

Separate penalties for gap opening vs extension:

$$Gap\_cost = G_o + k \cdot G_e$$

where $G_o$ = opening penalty, $G_e$ = extension penalty, $k$ = gap length.

### 7.2 SIMD Acceleration

Modern implementations use vectorization:
- SSE/AVX for parallel score computation
- 10-100x speedup for long sequences

### 7.3 Semi-Global Alignment

Hybrid between global and local:
- Free gaps at ends of one sequence
- Useful for read mapping

## 8. Comparison with BLAST

| Aspect | Smith-Waterman | BLAST |
|--------|----------------|-------|
| Type | Exact | Heuristic |
| Speed | O(mn) | Sub-quadratic |
| Sensitivity | Optimal | May miss weak matches |
| Use Case | Short sequences, verification | Database search |

## 9. Related Algorithms

| Algorithm | Purpose |
|-----------|---------|
| Needleman-Wunsch | Global alignment |
| BLAST | Fast database search |
| FASTA | Fast similarity search |
| Multiple Sequence Alignment | Align >2 sequences |

## 10. Practical Considerations

### 10.1 Parameter Selection

| Sequence Type | Match | Mismatch | Gap |
|---------------|-------|----------|-----|
| DNA (simple) | +2 | -1 | -1 |
| DNA (sensitive) | +1 | -3 | -5 |
| Protein | BLOSUM62 | BLOSUM62 | -11/-1 (affine) |

### 10.2 Memory Optimization

For very long sequences:
- Use Hirschberg's algorithm for linear space
- Compute in blocks
- Store only necessary rows

## 11. References

1. Smith, T.F. & Waterman, M.S. (1981). "Identification of Common Molecular Subsequences"
2. [NCBI BLAST](https://blast.ncbi.nlm.nih.gov/)
3. [Wikipedia - Smith-Waterman](https://en.wikipedia.org/wiki/Smith%E2%80%93Waterman_algorithm)
4. Durbin et al. "Biological Sequence Analysis"
