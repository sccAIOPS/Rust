# Burrows-Wheeler Transform (BWT)

## 1. Overview

The Burrows-Wheeler Transform (BWT), also known as **block-sorting compression**, was invented by Michael Burrows and David Wheeler in 1994 while working at DEC Systems Research Center. The algorithm was published in their technical report "A Block-sorting Lossless Data Compression Algorithm."

BWT is not a compression algorithm itself but a **reversible transformation** that rearranges a string to group similar characters together. This clustering makes the output highly amenable to compression by subsequent algorithms like Move-to-Front (MTF) transform and Run-Length Encoding (RLE).

The transform is the foundation of the **bzip2** compression utility, which achieves compression ratios superior to many other popular algorithms. What makes BWT remarkable is that this powerful rearrangement is completely **reversible** without storing any additional data beyond a single index.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a string $S$ of length $n$, the Burrows-Wheeler Transform produces:
1. A permuted string $S'$ of the same length
2. An index $I$ indicating the position of the original string in a sorted list of rotations

The transformation must be **invertible**: given $(S', I)$, we can uniquely recover $S$.

### 2.2 Mathematical Model

**Rotation Matrix:**

For a string $S = s_0 s_1 ... s_{n-1}$, define the rotation $R_k(S)$ as:
$$R_k(S) = s_k s_{k+1} ... s_{n-1} s_0 s_1 ... s_{k-1}$$

The **rotation matrix** $M$ is formed by all $n$ rotations of $S$:

$$M = \begin{bmatrix} R_0(S) \\ R_1(S) \\ \vdots \\ R_{n-1}(S) \end{bmatrix}$$

**Sorted Rotation Matrix:**

Let $M'$ be the matrix $M$ with rows sorted lexicographically.

**BWT Output:**
- $S'$ = last column of $M'$ (concatenated)
- $I$ = row index of original string $S$ in $M'$

**Key Property (Last-First Mapping):**

The $i$-th occurrence of character $c$ in the last column corresponds to the $i$-th occurrence of $c$ in the first column. This property enables efficient reversal.

### 2.3 Correctness Proof

**Claim:** BWT is invertible—the original string can be uniquely recovered from $(S', I)$.

**Proof:**

1. **First column recovery:** Sorting $S'$ gives the first column of $M'$ (since $M'$ is sorted by first character).

2. **Last-First (LF) property:** For any character $c$, if it appears as the $k$-th occurrence in the last column (row $i$), then it appears as the $k$-th occurrence in the first column (row $j$), and row $j$'s last character precedes row $i$'s first character in the original string.

3. **Reconstruction:** Starting from index $I$:
   - The character at position $I$ in the last column is the last character of $S$
   - Use LF-mapping to find the preceding character
   - Repeat $n$ times to recover $S$

**Alternative proof (constructive):**

The iterative method repeatedly:
1. Prepends the BWT string as a new first column
2. Sorts the resulting strings

After $n$ iterations, all rotations are reconstructed. Row $I$ contains the original string. ∎

## 3. Algorithm Description

### 3.1 Intuition

**Forward Transform:**
1. Generate all rotations of the input string
2. Sort these rotations lexicographically
3. Take the last character from each sorted rotation
4. Remember which row contains the original string

**Why it works for compression:**
- Sorting groups rotations with similar prefixes
- Rotations with similar prefixes often end with similar characters
- This creates "runs" of similar characters in the output

**Inverse Transform:**
The key insight is that we only need the last column and the index. By repeatedly prepending and sorting, we reconstruct all rotations.

### 3.2 Pseudocode

**Forward Transform:**
```
function BWT_TRANSFORM(S):
    n ← length(S)
    
    // Generate all rotations
    rotations ← empty list
    for i from 0 to n-1:
        rotation ← S[i..n] + S[0..i]
        append rotation to rotations
    
    // Sort rotations lexicographically
    sort(rotations)
    
    // Find index of original string
    idx ← index of S in rotations
    
    // Build BWT string from last characters
    bwt_string ← ""
    for each rotation in rotations:
        bwt_string ← bwt_string + last_char(rotation)
    
    return (bwt_string, idx)
```

**Inverse Transform:**
```
function REVERSE_BWT(bwt_string, idx):
    n ← length(bwt_string)
    table ← array of n empty strings
    
    // Iteratively build the rotation table
    for iteration from 1 to n:
        // Prepend BWT characters to each row
        for i from 0 to n-1:
            table[i] ← bwt_string[i] + table[i]
        
        // Sort the table
        sort(table)
    
    return table[idx]
```

### 3.3 Step-by-Step Example

**Forward Transform Example:**

Input: `"^BANANA"`

**Step 1: Generate all rotations**

| Index | Rotation |
|-------|----------|
| 0 | `^BANANA` |
| 1 | `BANANA^` |
| 2 | `ANANA^B` |
| 3 | `NANA^BA` |
| 4 | `ANA^BAN` |
| 5 | `NA^BANA` |
| 6 | `A^BANAN` |

**Step 2: Sort rotations lexicographically**

| Sorted Index | Rotation | Last Char |
|--------------|----------|-----------|
| 0 | `A^BANAN` | N |
| 1 | `ANA^BAN` | N |
| 2 | `ANANA^B` | B |
| 3 | `BANANA^` | ^ |
| 4 | `NA^BANA` | A |
| 5 | `NANA^BA` | A |
| 6 | `^BANANA` | A |

**Step 3: Extract results**
- BWT string (last column): `BNN^AAA`
- Original string `^BANANA` is at index **6**

**Inverse Transform Example:**

Input: BWT string = `"BNN^AAA"`, index = 6

**Iteration 1:**
| Prepend | After Sort |
|---------|------------|
| B | A |
| N | A |
| N | A |
| ^ | B |
| A | N |
| A | N |
| A | ^ |

**Iteration 2:**
| Prepend | After Sort |
|---------|------------|
| BA | A^ |
| NA | AN |
| NA | AN |
| ^B | BA |
| AN | NA |
| AN | NA |
| A^ | ^B |

**Continue for 7 iterations...**

**Final Table (after 7 iterations):**

| Index | Rotation |
|-------|----------|
| 0 | `A^BANAN` |
| 1 | `ANA^BAN` |
| 2 | `ANANA^B` |
| 3 | `BANANA^` |
| 4 | `NA^BANA` |
| 5 | `NANA^BA` |
| 6 | `^BANANA` |

Result: `table[6]` = `"^BANANA"` ✓

## 4. Complexity Analysis

### 4.1 Time Complexity

**Forward Transform (Naive Implementation):**

| Operation | Complexity |
|-----------|------------|
| Generate rotations | $O(n^2)$ |
| Sort rotations | $O(n^2 \log n)$ |
| Build BWT string | $O(n)$ |
| **Total** | $O(n^2 \log n)$ |

**Derivation:** 
- Generating $n$ rotations, each of length $n$: $O(n^2)$
- Sorting $n$ strings of length $n$: $O(n \cdot n \log n) = O(n^2 \log n)$

**Optimized Forward Transform:**
Using suffix arrays, the transform can be computed in:
- $O(n)$ with DC3/Skew algorithm
- $O(n \log n)$ with simpler suffix array construction

**Inverse Transform (Naive Implementation):**

| Operation | Complexity |
|-----------|------------|
| Per iteration (prepend + sort) | $O(n^2 \log n)$ |
| Number of iterations | $n$ |
| **Total** | $O(n^3 \log n)$ |

**Optimized Inverse Transform:**
Using the LF-mapping property: $O(n)$

### 4.2 Space Complexity

**Forward Transform:**
- **Naive:** $O(n^2)$ — storing all rotations explicitly
- **Optimized:** $O(n)$ — using suffix array indices

**Inverse Transform:**
- **Naive:** $O(n^2)$ — storing the reconstruction table
- **Optimized:** $O(n)$ — using LF-mapping arrays

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Result Structure:**
```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct BwtResult {
    pub bwt_string: String,
    pub idx_original_string: usize,
}
```
Using a struct makes the return value self-documenting and type-safe.

**String Rotation:**
```rust
pub fn all_rotations(s: &str) -> Vec<String> {
    (0..s.len())
        .map(|i| format!("{}{}", &s[i..], &s[..i]))
        .collect()
}
```
- Uses string slicing which is $O(n)$ per rotation
- `format!` creates owned strings, required for storage

**Assertions for Safety:**
```rust
assert!(!s.is_empty(), "Input string must not be empty");
assert!(idx_original_string < bwt_string.len(), "Index must be less than BWT string length");
```
Panics clearly communicate precondition violations.

**Character Iteration:**
```rust
let bwt_chars: Vec<char> = bwt_string.chars().collect();
```
Converting to `Vec<char>` enables $O(1)$ indexing for Unicode strings.

### 5.2 Edge Cases

| Case | Input | BWT String | Index |
|------|-------|------------|-------|
| Single char | `"A"` | `"A"` | 0 |
| Repeated chars | `"AAAA"` | `"AAAA"` | 0 |
| Two chars | `"AB"` | `"BA"` | 1 |
| Palindrome | `"ABA"` | `"BAA"` | 1 |

**Important Considerations:**
- Empty string input should panic or return an error
- The algorithm assumes valid UTF-8 strings
- For binary data, use byte arrays instead of strings

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**bzip2 Compression:**
```
Input → BWT → MTF → RLE → Huffman → Output
```
bzip2 achieves 10-15% better compression than gzip on text files.

**Bioinformatics:**
- **FM-index:** Data structure for substring search in genomic data
- **Bowtie/BWA:** DNA sequence alignment tools use BWT
- Enables searching in compressed space

**Full-Text Search:**
- BWT enables efficient substring queries
- Used in bioinformatics databases with terabytes of sequence data

**Data Deduplication:**
- BWT helps identify repeated patterns
- Used in backup systems and storage optimization

### 6.2 Related Algorithms

| Algorithm | Relationship | Use Case |
|-----------|--------------|----------|
| **Suffix Array** | Efficient BWT construction | Large-scale text processing |
| **Move-to-Front** | Post-BWT transform | Converts clustering to small integers |
| **Run-Length Encoding** | Compresses MTF output | Exploits zero runs |
| **Huffman Coding** | Final entropy coding | Optimal prefix codes |
| **FM-index** | BWT-based search structure | Pattern matching |

**Complete bzip2 Pipeline:**

```mermaid
flowchart LR
    A[Input] --> B[Block Sorting/BWT]
    B --> C[Move-to-Front]
    C --> D[Run-Length Encoding]
    D --> E[Huffman Coding]
    E --> F[Compressed Output]
```

### 6.3 Compression Effectiveness

**Why BWT Improves Compression:**

Consider the string `"ABRACADABRA"`:
- Contains patterns: `ABRA` appears twice
- BWT output: `"RDARCAAAABB"`
- Notice the clustering of `A`s and `B`s

After MTF, repeated characters become runs of zeros, which RLE compresses efficiently.

**Typical Compression Ratios:**

| File Type | gzip | bzip2 (BWT-based) |
|-----------|------|-------------------|
| English text | 3:1 | 4:1 |
| Source code | 4:1 | 5:1 |
| DNA sequences | 4:1 | 8:1 |

## 7. References

1. **Burrows, M. & Wheeler, D. J.** (1994). "A Block-sorting Lossless Data Compression Algorithm". Digital Equipment Corporation Technical Report 124.

2. **Manzini, G.** (2001). "An analysis of the Burrows-Wheeler transform". Journal of the ACM, 48(3), 407-430.

3. **Ferragina, P. & Manzini, G.** (2000). "Opportunistic data structures with applications". FOCS 2000. (FM-index)

4. **Wikipedia**: [Burrows-Wheeler Transform](https://en.wikipedia.org/wiki/Burrows%E2%80%93Wheeler_transform)

5. **Seward, J.** (1996). "bzip2 and libbzip2". [https://sourceware.org/bzip2/](https://sourceware.org/bzip2/)

6. **Implementation**: [TheAlgorithms/Rust - burrows_wheeler_transform.rs](https://github.com/TheAlgorithms/Rust/blob/master/src/compression/burrows_wheeler_transform.rs)
