# Z Algorithm

## 1. Overview

The Z algorithm is a linear-time string matching algorithm that constructs the Z-array for a given string. The Z-array at position $i$ contains the length of the longest substring starting at position $i$ that matches a prefix of the string.

This algorithm is particularly elegant because the same technique used to construct the Z-array can be directly applied to pattern matching by concatenating the pattern and text.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Z-Array Definition:**

For a string $S$ of length $n$, define $Z[i]$ as the length of the longest substring starting at position $i$ that is also a prefix of $S$:

$$Z[i] = \max\{k : S[0..k-1] = S[i..i+k-1]\}$$

By convention, $Z[0]$ is typically undefined or set to 0/n.

### 2.2 Mathematical Model

**Z-Box:** The interval $[i, i + Z[i] - 1]$ where the match occurs.

**Key Observation:**
If we've computed Z-values for positions $1$ to $i-1$, and we know a Z-box $[l, r]$ that extends furthest to the right, we can use this information to accelerate computing $Z[i]$.

**Cases:**
1. If $i > r$: Compute $Z[i]$ naively from position 0
2. If $i \leq r$: 
   - $k = i - l$ (corresponding position in prefix)
   - If $Z[k] < r - i + 1$: $Z[i] = Z[k]$ (fits within known match)
   - Else: Start matching from $r - i + 1$ and extend

### 2.3 Correctness Proof

**Invariant:** The Z-box $[l, r]$ always represents the rightmost interval where $S[l..r] = S[0..r-l]$.

**Theorem:** The Z algorithm correctly computes all Z-values in $O(n)$ time.

*Proof sketch:* Each character is compared at most twice: once when it extends a Z-box to the right, and at most once when starting a new Z-box. The index $r$ only increases, bounding total comparisons.

## 3. Algorithm Description

### 3.1 Intuition

The Z algorithm maintains a "window" of the rightmost Z-box seen so far. When computing $Z[i]$:
- If $i$ is outside this window, compare characters naively
- If $i$ is inside the window, we already know that $S[i..r] = S[k..k+(r-i)]$ where $k = i - l$
  - Use previously computed $Z[k]$ to get a head start
  - Only extend if needed

### 3.2 Pseudocode

```
function Z_ARRAY(s):
    n = len(s)
    if n == 0: return []
    
    Z = [0] * n
    Z[0] = n  // or leave undefined
    
    l = 0, r = 0  // Current Z-box [l, r]
    
    for i = 1 to n - 1:
        if i > r:
            // Outside Z-box: naive computation
            Z[i] = match_length(s, 0, i)
        else:
            k = i - l  // Mirror position
            if Z[k] < r - i + 1:
                // Z-value fits within current Z-box
                Z[i] = Z[k]
            else:
                // Need to extend beyond Z-box
                Z[i] = r - i + 1
                Z[i] += match_from(s, Z[i], i + Z[i])
        
        // Update Z-box if current extends further right
        if i + Z[i] - 1 > r:
            l = i
            r = i + Z[i] - 1
    
    return Z

function MATCH_PATTERN(text, pattern):
    // Concatenate: pattern + sentinel + text
    combined = pattern + "$" + text
    Z = Z_ARRAY(combined)
    
    matches = []
    pat_len = len(pattern)
    
    for i = pat_len + 1 to len(combined) - 1:
        if Z[i] == pat_len:
            matches.append(i - pat_len - 1)  // Position in original text
    
    return matches
```

### 3.3 Step-by-Step Example

**String:** "aabxaab"

**Computing Z-array:**

| i | s[i..] | Z[i] | l | r | Explanation |
|---|--------|------|---|---|-------------|
| 0 | aabxaab | 7 | - | - | Full string (by definition) |
| 1 | abxaab | 1 | 1 | 1 | 'a' matches s[0], 'b'≠'a' |
| 2 | bxaab | 0 | 1 | 1 | 'b'≠'a' |
| 3 | xaab | 0 | 1 | 1 | 'x'≠'a' |
| 4 | aab | 3 | 4 | 6 | "aab" matches prefix |
| 5 | ab | 1 | 4 | 6 | Inside Z-box, k=1, Z[1]=1 |
| 6 | b | 0 | 4 | 6 | Inside Z-box, k=2, Z[2]=0 |

**Result:** Z = [7, 1, 0, 0, 3, 1, 0]

**Pattern Matching Example:**

Text: "xaabxaab", Pattern: "aab"

Combined: "aab$xaabxaab"
Z-array:  [12, 1, 0, 0, 0, 3, 1, 0, 0, 3, 1, 0]

Matches at positions where Z[i] = 3 (pattern length):
- i=5 → text position = 5-3-1 = 1
- i=9 → text position = 9-3-1 = 5

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Z-array construction | $O(n)$ |
| Pattern matching | $O(n + m)$ |

**Proof of Linear Time:**

The key observation is that the right boundary $r$ only increases. Each character position contributes to at most:
1. One comparison when $r$ extends past it
2. Constant work when using cached Z-values

Total comparisons: $O(n)$

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Z-array | $O(n)$ |
| Combined string (pattern matching) | $O(n + m)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
fn calculate_z_value<T: Eq>(
    input_string: &[T],
    pattern: &[T],
    start_index: usize,
    mut z_value: usize,
) -> usize {
    let size = input_string.len();
    let pattern_size = pattern.len();

    while (start_index + z_value) < size && z_value < pattern_size {
        if input_string[start_index + z_value] != pattern[z_value] {
            break;
        }
        z_value += 1;
    }
    z_value
}
```

**Implementation Features:**
- Generic over any type implementing `Eq`
- Works with any indexable slice
- Supports both Z-array construction and pattern matching

**Usage Patterns:**
```rust
// Z-array of a string
let z = z_array("aabxaab".as_bytes());

// Pattern matching
let matches = match_pattern("text".as_bytes(), "pattern".as_bytes());
```

### 5.2 Edge Cases

| Case | Z-array Result |
|------|----------------|
| Empty string | `[]` |
| Single character | `[1]` or `[0]` |
| All same characters "aaaa" | `[4, 3, 2, 1]` |
| No repeating prefix | `[n, 0, 0, ..., 0]` |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Pattern Matching:**
   - Alternative to KMP with similar complexity
   - Simpler implementation in many cases

2. **String Analysis:**
   - Finding repetitive structures
   - Period detection in strings

3. **Bioinformatics:**
   - Genome sequence analysis
   - Finding repeated motifs

4. **Text Compression:**
   - Detecting repetitions for LZ77-style compression
   - Dictionary-based compression

### 6.2 Related Applications

**Finding All Periods:**
A string has period $p$ if $S[i] = S[i + p]$ for all valid $i$.

Using Z-array:
```rust
fn find_periods(s: &str) -> Vec<usize> {
    let z = z_array(s.as_bytes());
    let n = s.len();
    
    (1..n)
        .filter(|&i| i + z[i] >= n)
        .collect()
}
```

### 6.3 Related Algorithms

| Algorithm | Comparison |
|-----------|------------|
| **KMP** | Uses prefix function instead of Z-array |
| **Suffix Array** | More general but more complex |
| **Rolling Hash** | Better for multiple patterns |

**Z-array vs KMP Prefix Function:**

Both can be converted to each other in O(n):
- Z[i] gives match length at position i
- π[i] gives length of longest prefix = suffix ending at i

## 7. Variations and Extensions

### 7.1 Finding Longest Repeated Substring

```rust
fn longest_repeated_substring(s: &str) -> usize {
    let z = z_array(s.as_bytes());
    z.iter().skip(1).max().copied().unwrap_or(0)
}
```

### 7.2 Finding Distinct Substrings

The Z-array can help count distinct substrings by identifying which suffixes share prefixes.

### 7.3 Multiple Pattern Matching

```rust
fn multi_pattern_search(text: &str, patterns: &[&str]) -> Vec<Vec<usize>> {
    patterns.iter()
        .map(|p| match_pattern(text.as_bytes(), p.as_bytes()))
        .collect()
}
```

## 8. References

1. Gusfield, D. "Algorithms on Strings, Trees, and Sequences", Section 1.4.
2. Main, M. G., & Lorentz, R. J. (1984). "An O(n log n) Algorithm for Finding All Repetitions in a String". *Journal of Algorithms*.
3. Crochemore, M., & Rytter, W. "Text Algorithms", Chapter 2.

## Implementation

See: [src/string/z_algorithm.rs](../../src/string/z_algorithm.rs)
