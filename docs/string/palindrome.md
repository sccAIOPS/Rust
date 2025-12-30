# Palindrome Detection

## 1. Overview

A **palindrome** is a sequence that reads the same forwards and backwards. Palindrome detection is fundamental in string algorithms, appearing in computational biology (DNA sequence analysis), natural language processing, and interview problems. This document covers both simple palindrome checking and finding the longest palindromic substring.

## 2. Mathematical Foundation

### 2.1 Definition

A string $S$ of length $n$ is a **palindrome** if:
$$S_i = S_{n-1-i} \quad \forall i \in [0, \lfloor n/2 \rfloor)$$

Equivalently: $S = S^R$ where $S^R$ is the reverse of $S$.

### 2.2 Properties

1. **Empty String:** $\epsilon$ is a palindrome (vacuously true)
2. **Single Character:** Any single character is a palindrome
3. **Composition:** If $S$ is a palindrome, so is any character $c$ where $cSc$
4. **Subpalindromes:** A palindrome of length $n$ contains $n$ palindromic substrings (including single characters)

### 2.3 Longest Palindromic Substring

Finding the longest palindromic substring (LPS) has several approaches:
- **Brute Force:** $O(n^3)$
- **Dynamic Programming:** $O(n^2)$ time, $O(n^2)$ space
- **Expand Around Center:** $O(n^2)$ time, $O(1)$ space
- **Manacher's Algorithm:** $O(n)$ time and space

## 3. Algorithm Description

### 3.1 Simple Palindrome Check

**Approach:** Compare string with its reverse.

```
function IS_PALINDROME(s):
    n = len(s)
    for i = 0 to n/2 - 1:
        if s[i] ≠ s[n - 1 - i]:
            return false
    return true
```

### 3.2 Longest Palindromic Substring (Expand Around Center)

```
function LONGEST_PALINDROME_SUBSTRING(s):
    if len(s) < 2:
        return s
    
    start = 0
    max_len = 1
    
    for i = 0 to len(s) - 1:
        // Odd-length palindromes (center at i)
        len1 = expand_around_center(s, i, i)
        // Even-length palindromes (center between i and i+1)
        len2 = expand_around_center(s, i, i + 1)
        
        len = max(len1, len2)
        if len > max_len:
            max_len = len
            start = i - (len - 1) / 2
    
    return s[start : start + max_len]

function EXPAND_AROUND_CENTER(s, left, right):
    while left >= 0 and right < len(s) and s[left] = s[right]:
        left -= 1
        right += 1
    return right - left - 1
```

### 3.3 Step-by-Step Example

**String:** "babad"

| Center | Type | Expansion | Palindrome | Length |
|--------|------|-----------|------------|--------|
| 0 | odd | b | "b" | 1 |
| 0-1 | even | ba | - | 0 |
| 1 | odd | aba | "aba" | 3 |
| 1-2 | even | ab | - | 0 |
| 2 | odd | bab | "bab" | 3 |
| 2-3 | even | ba | - | 0 |
| 3 | odd | a | "a" | 1 |
| 3-4 | even | ad | - | 0 |
| 4 | odd | d | "d" | 1 |

**Result:** "bab" or "aba" (length 3)

## 4. Complexity Analysis

### 4.1 Simple Palindrome Check

| Metric | Complexity |
|--------|------------|
| Time | $O(n)$ |
| Space | $O(1)$ using two pointers |

### 4.2 Expand Around Center

| Metric | Complexity |
|--------|------------|
| Time | $O(n^2)$ worst case |
| Space | $O(1)$ |

### 4.3 Dynamic Programming

| Metric | Complexity |
|--------|------------|
| Time | $O(n^2)$ |
| Space | $O(n^2)$ |

For $O(n)$ solution, see [Manacher's Algorithm](manacher.md).

## 5. Implementation Notes

### 5.1 Rust Implementation - Simple Check

```rust
pub fn is_palindrome(s: &str) -> bool {
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    
    for i in 0..n / 2 {
        if chars[i] != chars[n - 1 - i] {
            return false;
        }
    }
    true
}

// Using iterators (more idiomatic)
pub fn is_palindrome_iter(s: &str) -> bool {
    s.chars().eq(s.chars().rev())
}
```

### 5.2 Rust Implementation - Longest Palindromic Substring

```rust
pub fn longest_palindromic_substring(s: &str) -> String {
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    
    if n < 2 {
        return s.to_string();
    }
    
    let mut start = 0;
    let mut max_len = 1;
    
    for i in 0..n {
        // Odd length palindromes
        let len1 = expand_around_center(&chars, i as isize, i as isize);
        // Even length palindromes
        let len2 = expand_around_center(&chars, i as isize, (i + 1) as isize);
        
        let len = len1.max(len2);
        if len > max_len {
            max_len = len;
            start = i - (len - 1) / 2;
        }
    }
    
    chars[start..start + max_len].iter().collect()
}

fn expand_around_center(chars: &[char], mut left: isize, mut right: isize) -> usize {
    let n = chars.len() as isize;
    
    while left >= 0 && right < n && chars[left as usize] == chars[right as usize] {
        left -= 1;
        right += 1;
    }
    
    (right - left - 1) as usize
}
```

### 5.3 Case-Insensitive and Alphanumeric Only

```rust
pub fn is_palindrome_clean(s: &str) -> bool {
    let cleaned: Vec<char> = s
        .chars()
        .filter(|c| c.is_alphanumeric())
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    cleaned.iter().eq(cleaned.iter().rev())
}
```

### 5.4 Edge Cases

| Input | is_palindrome | Notes |
|-------|---------------|-------|
| "" | true | Empty string |
| "a" | true | Single char |
| "aa" | true | Even length |
| "aba" | true | Odd length |
| "ab" | false | Simple false case |
| "A man a plan..." | true | With cleaning |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Computational Biology:**
   - DNA palindrome detection (restriction enzyme sites)
   - RNA secondary structure prediction

2. **Natural Language Processing:**
   - Word games and puzzles
   - Linguistic analysis

3. **Data Validation:**
   - ISBN check digits
   - VIN verification

4. **Algorithm Problems:**
   - Interview questions
   - Competitive programming

### 6.2 Example Applications

**DNA Palindrome (Complementary):**
```rust
fn is_dna_palindrome(seq: &str) -> bool {
    let complement: String = seq.chars().rev().map(|c| match c {
        'A' => 'T', 'T' => 'A',
        'C' => 'G', 'G' => 'C',
        _ => c
    }).collect();
    seq == complement
}
```

## 7. Variations

### 7.1 Count All Palindromic Substrings

```rust
pub fn count_palindromic_substrings(s: &str) -> usize {
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    let mut count = 0;
    
    for i in 0..n {
        // Count odd-length palindromes
        count += count_from_center(&chars, i as isize, i as isize);
        // Count even-length palindromes
        count += count_from_center(&chars, i as isize, (i + 1) as isize);
    }
    
    count
}

fn count_from_center(chars: &[char], mut left: isize, mut right: isize) -> usize {
    let n = chars.len() as isize;
    let mut count = 0;
    
    while left >= 0 && right < n && chars[left as usize] == chars[right as usize] {
        count += 1;
        left -= 1;
        right += 1;
    }
    
    count
}
```

### 7.2 Minimum Insertions for Palindrome

See [Shortest Palindrome](shortest_palindrome.md) for related problem.

## 8. Related Algorithms

| Algorithm | Purpose | Complexity |
|-----------|---------|------------|
| [Manacher](manacher.md) | LPS in O(n) | O(n) |
| [Shortest Palindrome](shortest_palindrome.md) | Min chars to prepend | O(n) |
| [Palindrome Partitioning](../dynamic_programming/palindrome_partitioning.md) | Min cuts | O(n²) |

## 9. References

1. Manacher, G. (1975). "A New Linear-Time 'On-Line' Algorithm for Finding the Smallest Initial Palindrome of a String".
2. Gusfield, D. "Algorithms on Strings, Trees and Sequences", Chapter 9.
3. Cormen, T.H., et al. "Introduction to Algorithms", String Matching chapter.

## Implementation

See: [src/string/palindrome.rs](../../src/string/palindrome.rs)
