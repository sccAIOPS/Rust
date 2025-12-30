# Shortest Palindrome

## 1. Overview

The shortest palindrome problem asks: given a string, find the shortest palindrome that can be formed by adding characters only to the front of the string. This is equivalent to finding the longest palindromic prefix and prepending the reverse of the remaining suffix.

The implementation uses the KMP failure function to efficiently find this longest palindromic prefix.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a string $S$ of length $n$, find the shortest string $P$ such that:
1. $P$ is a palindrome
2. $S$ is a suffix of $P$
3. $|P|$ is minimized

### 2.2 Mathematical Model

**Key Insight:**
If $S[0..k]$ is the longest palindromic prefix of $S$, then:
- We need to prepend $reverse(S[k+1..n-1])$ to make $S$ palindromic
- Result: $reverse(S[k+1..n-1]) + S$

**Finding Longest Palindromic Prefix:**
Consider string $T = S + \$ + reverse(S)$ where \$ is a separator.

The longest proper prefix of $T$ that is also a suffix gives us the longest palindromic prefix of $S$.

Why? If $S[0..k] = reverse(S[0..k])$, then this prefix appears as both:
- Prefix of $S$ (at start of $T$)
- Suffix of $reverse(S)$ (at end of $T$)

### 2.3 Correctness

**Theorem:** The KMP prefix function on $S + \$ + reverse(S)$ correctly identifies the longest palindromic prefix.

*Proof:* The separator \$ ensures we don't match across the boundary. The prefix function finds the longest prefix that matches a suffix, which by construction corresponds to a palindromic prefix of $S$.

## 3. Algorithm Description

### 3.1 Intuition

1. Compute the suffix table (KMP failure function) for the original string
2. Match the original against its reverse using this table
3. The final match length gives the longest palindromic prefix
4. Prepend the remaining characters (reversed) to form the shortest palindrome

### 3.2 Pseudocode

```
function SHORTEST_PALINDROME(s):
    if s is empty:
        return ""
    
    original = s.chars()
    suffix_table = COMPUTE_SUFFIX(original)
    
    reversed = s.reverse().chars()
    
    // Find how much of original matches end of reversed
    prefix_match = COMPUTE_PREFIX_MATCH(original, reversed, suffix_table)
    
    // prefix_match[n-1] = length of longest palindromic prefix
    palindrome_prefix_len = prefix_match[len(s) - 1]
    
    // Prepend reverse of suffix that's not part of palindrome
    result = reversed + original[palindrome_prefix_len:]
    return result

function COMPUTE_SUFFIX(chars):
    n = len(chars)
    suffix = [0] * n
    
    for i = 1 to n - 1:
        j = suffix[i - 1]
        while j > 0 and chars[j] != chars[i]:
            j = suffix[j - 1]
        suffix[i] = j + (1 if chars[j] == chars[i] else 0)
    
    return suffix

function COMPUTE_PREFIX_MATCH(original, reversed, suffix):
    n = len(original)
    match_table = [0] * n
    
    match_table[0] = 1 if original[0] == reversed[0] else 0
    
    for i = 1 to n - 1:
        j = match_table[i - 1]
        while j > 0 and reversed[i] != original[j]:
            j = suffix[j - 1]
        match_table[i] = j + (1 if reversed[i] == original[j] else 0)
    
    return match_table
```

### 3.3 Step-by-Step Example

**Input:** "aacecaaa"

**Step 1: Compute suffix table for "aacecaaa"**
| i | char | suffix[i] |
|---|------|-----------|
| 0 | a | 0 |
| 1 | a | 1 |
| 2 | c | 0 |
| 3 | e | 0 |
| 4 | c | 0 |
| 5 | a | 1 |
| 6 | a | 2 |
| 7 | a | 2 |

**Step 2: Compute prefix match against reversed "aaacecaa"**
| i | reversed[i] | match[i] |
|---|-------------|----------|
| 0 | a | 1 |
| 1 | a | 2 |
| 2 | a | 2 |
| 3 | c | 0→1 |
| 4 | e | 0 |
| 5 | c | 0 |
| 6 | a | 1 |
| 7 | a | 7 |

**Step 3: Result**
- Longest palindromic prefix length = 7 ("aacecaa")
- Characters to prepend: "a" (reverse of "a")
- Result: "a" + "aacecaaa" = "aaacecaaa"

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Compute suffix table | $O(n)$ |
| Compute prefix match | $O(n)$ |
| String concatenation | $O(n)$ |
| **Total** | $O(n)$ |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Original char array | $O(n)$ |
| Reversed char array | $O(n)$ |
| Suffix table | $O(n)$ |
| Match table | $O(n)$ |
| **Total** | $O(n)$ |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn shortest_palindrome(s: &str) -> String {
    if s.is_empty() {
        return "".to_string();
    }

    let original_chars: Vec<char> = s.chars().collect();
    let suffix_table = compute_suffix(&original_chars);

    let mut reversed_chars: Vec<char> = s.chars().rev().collect();
    let prefix_match = compute_prefix_match(
        &original_chars, 
        &reversed_chars, 
        &suffix_table
    );

    reversed_chars.append(
        &mut original_chars[prefix_match[original_chars.len() - 1]..].to_vec()
    );
    reversed_chars.iter().collect()
}
```

### 5.2 Key Functions

**Suffix Table (KMP Failure Function):**
```rust
pub fn compute_suffix(chars: &[char]) -> Vec<usize> {
    let mut suffix = vec![0; chars.len()];
    for i in 1..chars.len() {
        let mut j = suffix[i - 1];
        while j > 0 && chars[j] != chars[i] {
            j = suffix[j - 1];
        }
        suffix[i] = j + (chars[j] == chars[i]) as usize;
    }
    suffix
}
```

**Prefix Match Computation:**
```rust
pub fn compute_prefix_match(
    original: &[char], 
    reversed: &[char], 
    suffix: &[usize]
) -> Vec<usize> {
    let mut match_table = vec![0; original.len()];
    match_table[0] = usize::from(original[0] == reversed[0]);
    
    for i in 1..original.len() {
        let mut j = match_table[i - 1];
        while j > 0 && reversed[i] != original[j] {
            j = suffix[j - 1];
        }
        match_table[i] = j + usize::from(reversed[i] == original[j]);
    }
    match_table
}
```

### 5.3 Edge Cases

| Input | Output | Explanation |
|-------|--------|-------------|
| "" | "" | Empty string |
| "a" | "a" | Single char is palindrome |
| "ab" | "bab" | Prepend "b" |
| "aba" | "aba" | Already palindrome |
| "abcd" | "dcbabcd" | No palindromic prefix > 1 |
| "aacecaaa" | "aaacecaaa" | Long palindromic prefix |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Data Transformation:**
   - Converting sequences to symmetric forms
   - Data padding for symmetric algorithms

2. **String Processing:**
   - Text formatting
   - Creating symmetric patterns

3. **Competitive Programming:**
   - Common interview/contest problem
   - Foundation for palindrome problems

### 6.2 Related Problems

**Shortest Palindrome by Appending:**
Similar problem but appending to end instead of prepending.
Solution: Find longest palindromic suffix instead.

**Minimum Insertions for Palindrome:**
Insert anywhere to make palindrome.
Solution: Length - LCS(s, reverse(s))

## 7. Alternative Approaches

### 7.1 Rolling Hash Approach

```rust
fn shortest_palindrome_hash(s: &str) -> String {
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    
    let mut forward_hash = 0u64;
    let mut backward_hash = 0u64;
    let mut power = 1u64;
    let base = 31u64;
    let modulo = 1_000_000_007u64;
    
    let mut best_len = 0;
    
    for i in 0..n {
        let c = chars[i] as u64 - 'a' as u64 + 1;
        
        forward_hash = (forward_hash * base + c) % modulo;
        backward_hash = (backward_hash + c * power) % modulo;
        power = (power * base) % modulo;
        
        if forward_hash == backward_hash {
            best_len = i + 1;
        }
    }
    
    let suffix: String = chars[best_len..].iter().rev().collect();
    suffix + s
}
```

**Trade-off:** Simpler but has small probability of hash collision.

### 7.2 Manacher-Based Approach

Use Manacher's algorithm to find palindromic prefixes directly.

## 8. Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | $O(n^2)$ | $O(n)$ | Check each prefix |
| KMP-based | $O(n)$ | $O(n)$ | Current implementation |
| Rolling Hash | $O(n)$ | $O(1)$ | Small collision risk |
| Z-Algorithm | $O(n)$ | $O(n)$ | Alternative linear |

## 9. References

1. Knuth, D. E., Morris, J. H., & Pratt, V. R. (1977). "Fast Pattern Matching in Strings".
2. LeetCode Problem 214: Shortest Palindrome
3. Gusfield, D. "Algorithms on Strings, Trees, and Sequences".

## Implementation

See: [src/string/shortest_palindrome.rs](../../src/string/shortest_palindrome.rs)
