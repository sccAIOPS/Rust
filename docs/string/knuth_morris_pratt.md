# Knuth-Morris-Pratt (KMP) Algorithm

## 1. Overview

The Knuth-Morris-Pratt (KMP) algorithm is a linear-time string matching algorithm developed by Donald Knuth, Vaughan Pratt, and James H. Morris in 1977. It efficiently finds all occurrences of a pattern string within a text string by preprocessing the pattern to avoid redundant comparisons.

The key insight is that when a mismatch occurs, the pattern itself contains sufficient information to determine where the next match could begin, thus bypassing re-examination of previously matched characters.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- A text string $T$ of length $n$
- A pattern string $P$ of length $m$

Find: All positions $i$ where $P$ occurs as a substring of $T$, i.e., $T[i..i+m] = P$

### 2.2 Mathematical Model

**Prefix Function (Failure Function):**

For a pattern $P[0..m-1]$, define $\pi[i]$ as the length of the longest proper prefix of $P[0..i]$ that is also a suffix of $P[0..i]$.

$$\pi[i] = \max\{k : 0 \leq k < i \text{ and } P[0..k-1] = P[i-k+1..i]\}$$

**Properties:**
- $\pi[0] = 0$ (by definition, empty proper prefix)
- $0 \leq \pi[i] < i$ for all $i > 0$
- If $\pi[i] = k$, then $\pi[i+1] \leq k + 1$

### 2.3 Correctness Proof

**Invariant:** At each step, we maintain the longest prefix of $P$ that matches a suffix of $T[0..i]$.

**Theorem:** KMP correctly finds all occurrences of $P$ in $T$.

*Proof:* When a mismatch occurs at position $j$ of the pattern after matching $j$ characters, we know that $T[i-j..i-1] = P[0..j-1]$. The prefix function tells us the longest prefix of $P$ that is also a suffix of $P[0..j-1]$, which means this prefix also matches $T[i-\pi[j-1]..i-1]$. Thus, we can safely shift the pattern.

## 3. Algorithm Description

### 3.1 Intuition

1. **Preprocessing Phase:** Build a "partial match table" (prefix table) from the pattern
2. **Searching Phase:** Scan the text left-to-right, using the table to skip ahead when mismatches occur

When we see a mismatch after matching some characters, instead of restarting from scratch, we use our knowledge of the pattern's internal structure to "slide" intelligently.

### 3.2 Pseudocode

```
function KMP_SEARCH(text, pattern):
    if pattern is empty or text is empty:
        return []
    
    // Build partial match table
    π = BUILD_PREFIX_TABLE(pattern)
    
    matches = []
    match_length = 0
    
    for i = 0 to length(text) - 1:
        // Handle mismatch by following failure links
        while match_length > 0 and text[i] ≠ pattern[match_length]:
            match_length = π[match_length - 1]
        
        // Character matches
        if text[i] = pattern[match_length]:
            match_length = match_length + 1
        
        // Full pattern match found
        if match_length = length(pattern):
            matches.append(i - match_length + 1)
            match_length = π[match_length - 1]
    
    return matches

function BUILD_PREFIX_TABLE(pattern):
    π = [0] * length(pattern)
    
    for i = 1 to length(pattern) - 1:
        length = π[i - 1]
        
        while length > 0 and pattern[length] ≠ pattern[i]:
            length = π[length - 1]
        
        if pattern[length] = pattern[i]:
            π[i] = length + 1
        else:
            π[i] = length
    
    return π
```

### 3.3 Step-by-Step Example

**Text:** "ABABDABACDABABCABAB"  
**Pattern:** "ABABC"

**Step 1: Build Prefix Table**

| i | Pattern[0..i] | Longest Proper Prefix = Suffix | π[i] |
|---|---------------|-------------------------------|------|
| 0 | "A" | "" | 0 |
| 1 | "AB" | "" | 0 |
| 2 | "ABA" | "A" | 1 |
| 3 | "ABAB" | "AB" | 2 |
| 4 | "ABABC" | "" | 0 |

**Step 2: Search**

```
Text:    A B A B D A B A C D A B A B C A B A B
Pattern: A B A B C
         ↑ match at i=0,1,2,3, mismatch at i=4
         
After mismatch: π[3] = 2, so we know "AB" matches
Text:    A B A B D A B A C D A B A B C A B A B
Pattern:     A B A B C
             ↑ continue from here...
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| **Best** | $O(n)$ | Pattern not found, minimal backtracking |
| **Average** | $O(n + m)$ | Linear in combined length |
| **Worst** | $O(n + m)$ | Guaranteed linear time |

**Derivation:**
- Preprocessing: $O(m)$ - each character examined at most twice
- Searching: $O(n)$ - text index $i$ never decreases; match_length increases/decreases bounded by total advances

**Amortized Analysis:** Let $\Phi$ = match_length. Each iteration either:
- Increments $\Phi$ by 1 (at most $n$ times total)
- Decreases $\Phi$ (each decrease requires prior increase)

Total operations: $O(n + m)$

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Prefix table | $O(m)$ |
| Character vectors | $O(n + m)$ |
| **Total** | $O(n + m)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Current implementation converts to Vec<char> for Unicode safety
let text_chars = string.chars().collect::<Vec<char>>();
let pattern_chars = pattern.chars().collect::<Vec<char>>();
```

**Trade-offs:**
- ✅ Correct Unicode handling
- ✅ O(1) random access to characters
- ❌ Additional O(n + m) space for character vectors
- ❌ Initial conversion time

**Alternative for ASCII-only:**
```rust
// More efficient for ASCII strings
let text_bytes = string.as_bytes();
let pattern_bytes = pattern.as_bytes();
```

### 5.2 Edge Cases

| Case | Handling | Return |
|------|----------|--------|
| Empty pattern | Early return | `[]` |
| Empty text | Early return | `[]` |
| Pattern longer than text | Never matches | `[]` |
| Pattern equals text | Single match | `[0]` |
| All characters same | Multiple overlapping matches | All valid positions |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Text Editors:** Find/Replace functionality
   - Efficient for repeated searches with same pattern
   - Foundation for regex engines

2. **Bioinformatics:**
   - DNA sequence matching
   - Finding gene patterns in genomes

3. **Plagiarism Detection:**
   - Finding copied text segments
   - Document similarity analysis

4. **Network Security:**
   - Intrusion detection systems
   - Pattern matching in packet payloads

5. **Data Validation:**
   - Log file analysis
   - Configuration file parsing

### 6.2 Related Algorithms

| Algorithm | When to Prefer |
|-----------|---------------|
| **Boyer-Moore** | Longer patterns, large alphabets |
| **Rabin-Karp** | Multiple patterns of different lengths |
| **Aho-Corasick** | Multiple patterns simultaneously |
| **Z-Algorithm** | Simpler implementation, same complexity |

## 7. References

1. Knuth, D. E., Morris, J. H., & Pratt, V. R. (1977). "Fast Pattern Matching in Strings". *SIAM Journal on Computing*, 6(2), 323-350.
2. Cormen, T. H., et al. "Introduction to Algorithms" (3rd ed.), Chapter 32.
3. Sedgewick, R. & Wayne, K. "Algorithms" (4th ed.), Section 5.3.

## Implementation

See: [src/string/knuth_morris_pratt.rs](../../src/string/knuth_morris_pratt.rs)
