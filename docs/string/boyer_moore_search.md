# Boyer-Moore String Search Algorithm

## 1. Overview

The Boyer-Moore algorithm, developed by Robert S. Boyer and J Strother Moore in 1977, is one of the most efficient string searching algorithms for practical use. Unlike naive approaches that scan left-to-right, Boyer-Moore compares the pattern from right-to-left and uses two heuristics to skip portions of the text.

The algorithm achieves sublinear time in the best case by potentially skipping large sections of the text without examining every character.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- A text string $T$ of length $n$
- A pattern string $P$ of length $m$

Find: All starting positions where $P$ occurs in $T$.

### 2.2 Mathematical Model

**Bad Character Rule:**

For a character $c$ at position $i$ in the text that causes a mismatch, define:
$$\text{bad\_char}[c] = \max\{j : P[j] = c, j < m-1\}$$

If $c$ doesn't appear in $P$, set $\text{bad\_char}[c] = -1$.

**Shift Amount:** When mismatch occurs at pattern position $j$ with text character $c$:
$$\text{shift} = \max(1, j - \text{bad\_char}[c])$$

**Good Suffix Rule (not implemented in current version):**

When a suffix of the pattern matches but the preceding character doesn't, shift to align with another occurrence of that suffix or a prefix that matches a suffix of the suffix.

### 2.3 Correctness Proof

**Theorem:** The bad character rule never skips a valid match.

*Proof:* If text character $c$ at position $i$ mismatches pattern position $j$, any alignment where $P[k] = c$ for $k > \text{bad\_char}[c]$ would also mismatch at some position since $c$ doesn't appear in $P[bad\_char[c]+1..m-1]$.

## 3. Algorithm Description

### 3.1 Intuition

1. Align the pattern with the beginning of the text
2. Compare characters from right-to-left within the pattern
3. On mismatch:
   - Use bad character table to determine how far to shift
   - Shift pattern to align the mismatched text character with its rightmost occurrence in the pattern
4. On full match, record position and shift

The right-to-left comparison allows larger jumps because when we find a mismatch with character $c$ that doesn't exist in the pattern, we can skip the entire pattern length.

### 3.2 Pseudocode

```
function BOYER_MOORE_SEARCH(text, pattern):
    if text is empty or pattern is empty or len(pattern) > len(text):
        return []
    
    bad_char_table = BUILD_BAD_CHAR_TABLE(pattern)
    positions = []
    shift = 0
    
    while shift <= len(text) - len(pattern):
        j = len(pattern) - 1
        
        // Compare right-to-left
        while j >= 0 and pattern[j] = text[shift + j]:
            j = j - 1
        
        if j < 0:
            // Full match found
            positions.append(shift)
            shift = shift + CALC_MATCH_SHIFT(...)
        else:
            // Mismatch - use bad character rule
            shift = shift + CALC_MISMATCH_SHIFT(j, shift, text, bad_char_table)
    
    return positions

function BUILD_BAD_CHAR_TABLE(pattern):
    table = empty hashmap
    for i = 0 to len(pattern) - 1:
        table[pattern[i]] = i
    return table

function CALC_MISMATCH_SHIFT(mismatch_index, shift, text, bad_char_table):
    mismatch_char = text[shift + mismatch_index]
    bad_char_shift = bad_char_table.get(mismatch_char, -1)
    return max(1, mismatch_index - bad_char_shift)
```

### 3.3 Step-by-Step Example

**Text:** "HERE IS A SIMPLE EXAMPLE"  
**Pattern:** "EXAMPLE"

**Bad Character Table:**
| Char | Position |
|------|----------|
| E | 6 |
| X | 1 |
| A | 3 |
| M | 4 |
| P | 5 |
| L | 2 |

**Search Process:**

```
Step 1: Align at position 0
Text:    H E R E   I S   A   S I M P L E   E X A M P L E
Pattern: E X A M P L E
                     ↑ Compare from right: E ≠ ' ' (space)
         Space not in pattern → shift by 7

Step 2: Shift to position 7
Text:    H E R E   I S   A   S I M P L E   E X A M P L E
Pattern:               E X A M P L E
                                   ↑ E ≠ S
         S not in pattern → shift by 7

Step 3: Continue shifting...
...

Final: Match found at position 17
Text:    H E R E   I S   A   S I M P L E   E X A M P L E
Pattern:                                   E X A M P L E
                                           ✓ Full match!
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Condition |
|------|------------|-----------|
| **Best** | $O(n/m)$ | Pattern not in text, large alphabet |
| **Average** | $O(n/m)$ to $O(n)$ | Typical text |
| **Worst** | $O(nm)$ | Repeating characters (e.g., "aaa...a" in "aaa...a") |

**Best Case Derivation:**
With large alphabet and non-matching pattern, each comparison typically causes a shift of $m$ positions, yielding $n/m$ comparisons.

**Worst Case Derivation:**
When text and pattern consist of same repeated character, every position must be checked: $O(nm)$.

**Note:** Adding the Good Suffix Rule (not in current implementation) reduces worst case to $O(n + m)$.

### 4.2 Space Complexity

| Component | Space | Description |
|-----------|-------|-------------|
| Bad character table | $O(k)$ | k = alphabet size (unique chars in pattern) |
| Character vectors | $O(n + m)$ | Text and pattern as char vectors |
| **Total** | $O(n + m)$ | With HashMap implementation |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// HashMap for bad character table
use std::collections::HashMap;

fn build_bad_char_table(pat: &[char]) -> HashMap<char, isize> {
    let mut bad_char_table = HashMap::new();
    for (i, &ch) in pat.iter().enumerate() {
        bad_char_table.insert(ch, i as isize);
    }
    bad_char_table
}
```

**Implementation Choices:**
- Uses `isize` for indices to handle negative values cleanly
- `HashMap` provides O(1) lookups but with hashing overhead
- Current implementation only uses Bad Character Rule (simpler, good enough for most cases)

**Potential Optimizations:**
```rust
// For ASCII-only, use fixed array instead of HashMap
let mut bad_char_table = [-1isize; 256];
for (i, &ch) in pat.iter().enumerate() {
    bad_char_table[ch as usize] = i as isize;
}
```

### 5.2 Edge Cases

| Case | Handling | Result |
|------|----------|--------|
| Empty text | Early return | `[]` |
| Empty pattern | Early return | `[]` |
| Pattern longer than text | Early return | `[]` |
| Single character pattern | Degrades to linear scan | All positions |
| Overlapping matches | Found correctly | All match positions |
| Case sensitivity | Exact match only | Case-sensitive |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Text Editors (Vim, Emacs, VS Code):**
   - Fast search in large files
   - Best for longer search terms

2. **Grep and similar tools:**
   - File content searching
   - Log analysis

3. **Intrusion Detection Systems:**
   - Network packet inspection
   - Malware signature matching

4. **Database Systems:**
   - LIKE clause optimization
   - Full-text search indexing

5. **Compilers:**
   - Lexical analysis
   - Token recognition

### 6.2 Related Algorithms

| Algorithm | Comparison |
|-----------|------------|
| **KMP** | More consistent O(n+m), but slower in practice for long patterns |
| **Rabin-Karp** | Better for multiple patterns, uses hashing |
| **Sunday** | Variant using next character after window |
| **Horspool** | Simplified Boyer-Moore with only bad character rule |

## 7. Performance Comparison

| Pattern Length | Boyer-Moore | KMP | Naive |
|---------------|-------------|-----|-------|
| Short (< 5) | Similar | Similar | Acceptable |
| Medium (5-20) | 2-5x faster | Baseline | 2-10x slower |
| Long (> 20) | 5-10x faster | Baseline | Much slower |

**When to Choose Boyer-Moore:**
- ✅ Long patterns
- ✅ Large alphabets (natural language)
- ✅ Pattern rarely found in text
- ❌ Very short patterns (overhead not worth it)
- ❌ Small alphabets (DNA, binary)

## 8. References

1. Boyer, R. S., & Moore, J. S. (1977). "A Fast String Searching Algorithm". *Communications of the ACM*, 20(10), 762-772.
2. Horspool, R. N. (1980). "Practical Fast Searching in Strings". *Software: Practice and Experience*, 10(6), 501-506.
3. Gusfield, D. "Algorithms on Strings, Trees, and Sequences", Chapter 2.

## Implementation

See: [src/string/boyer_moore_search.rs](../../src/string/boyer_moore_search.rs)
