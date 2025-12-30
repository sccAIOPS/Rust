# Manacher's Algorithm

## 1. Overview

Manacher's algorithm is a linear-time algorithm for finding the longest palindromic substring in a given string. Invented by Glenn Manacher in 1975, it cleverly exploits the symmetric property of palindromes to avoid redundant comparisons.

The algorithm transforms the problem by inserting separators between characters, allowing uniform handling of both odd and even length palindromes.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a string $S$ of length $n$, find the longest substring that reads the same forwards and backwards.

**Formal:** Find indices $i, j$ such that:
- $S[i..j]$ is a palindrome
- $j - i$ is maximized

### 2.2 Mathematical Model

**Transformation:**
Insert separator characters (e.g., '#') between all characters and at boundaries:
- "abba" → "#a#b#b#a#"
- "aba" → "#a#b#a#"

This ensures all palindromes have odd length in the transformed string.

**Palindrome Radius:**
For position $i$ in transformed string, let $P[i]$ = radius of longest palindrome centered at $i$.

**Key Property:**
If palindrome centered at $i$ extends to $r$ ($r = i + P[i]$), and we're computing $P[j]$ where $j < r$, we can use symmetry:
- Mirror position: $j' = 2c - j$ where $c$ is center of known palindrome
- If palindrome at $j'$ fits within bounds: $P[j] = P[j']$
- Otherwise: $P[j] \geq r - j$, then expand

### 2.3 Correctness

**Invariant:** At position $i$, we maintain the rightmost palindrome boundary $r$ and its center $c$.

**Theorem:** Each character is involved in at most one expansion operation, yielding O(n) total.

## 3. Algorithm Description

### 3.1 Intuition

1. Transform input by inserting separators
2. For each position, compute palindrome radius
3. Use previously computed radii to skip comparisons
4. Maintain the palindrome that extends furthest to the right
5. Extract the longest palindrome from the radius array

### 3.2 Pseudocode

```
function MANACHER(s):
    if len(s) <= 1:
        return s
    
    // Transform: "abc" → "#a#b#c#"
    chars = []
    for c in s:
        chars.append('#')
        chars.append(c)
    chars.append('#')
    
    n = len(chars)
    P = [1] * n  // Palindrome radius (including center)
    
    center = 0       // Center of rightmost palindrome
    right = 0        // Right edge of rightmost palindrome
    
    for i = 0 to n - 1:
        // Step 1: Initialize P[i] using symmetry
        if right > i and i > center:
            mirror = 2 * center - i
            P[i] = min(right - i, P[mirror])
        
        // Step 2: Attempt expansion
        radius = (P[i] - 1) / 2 + 1
        while i - radius >= 0 and i + radius < n and chars[i - radius] == chars[i + radius]:
            P[i] += 2
            radius += 1
        
        // Step 3: Update center and right if we extended past right
        if i + P[i] / 2 > right:
            center = i
            right = i + P[i] / 2
            if right >= n - 1:
                break  // Optimization: reached end
    
    // Step 4: Find maximum and extract result
    max_idx = argmax(P)
    max_radius = (P[max_idx] - 1) / 2
    
    return chars[max_idx - max_radius : max_idx + max_radius + 1]
           .filter(c != '#')
           .join("")
```

### 3.3 Step-by-Step Example

**Input:** "babad"

**Transformed:** "#b#a#b#a#d#"

**Computing P array:**

| i | char | P[i] | center | right | Explanation |
|---|------|------|--------|-------|-------------|
| 0 | # | 1 | 0 | 0 | Expand: no match |
| 1 | b | 3 | 1 | 2 | Expand: #b# matches |
| 2 | # | 1 | 1 | 2 | Mirror: P[0]=1, right=2, i=2 |
| 3 | a | 7 | 3 | 6 | Expand: #a#b#a# |
| 4 | # | 1 | 3 | 6 | Mirror: P[2]=1 |
| 5 | b | 3 | 3 | 6 | Mirror: P[1]=3, but right-i=1 |
| ... | | | | | |

**Result:** Maximum P[i] = 7 at position 3
Palindrome: "aba" (or "bab" with different tie-breaking)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Transformation | $O(n)$ |
| Main loop iterations | $O(n)$ |
| Total expansions | $O(n)$ amortized |
| **Total** | $O(n)$ |

**Amortized Analysis:**
The right boundary $r$ only increases. Each expansion operation moves $r$ forward by at least 1. Since $r$ can increase at most $2n$ times (transformed length), total expansions are $O(n)$.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Transformed string | $O(n)$ |
| P array | $O(n)$ |
| **Total** | $O(n)$ |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn manacher(s: String) -> String {
    let l = s.len();
    if l <= 1 {
        return s;
    }

    // Insert dummy characters for uniform odd/even handling
    let mut chars: Vec<char> = Vec::with_capacity(s.len() * 2 + 1);
    for c in s.chars() {
        chars.push('#');
        chars.push(c);
    }
    chars.push('#');

    let mut length_of_palindrome = vec![1usize; chars.len()];
    let mut current_center: usize = 0;
    let mut right_from_current_center: usize = 0;

    for i in 0..chars.len() {
        if right_from_current_center > i && i > current_center {
            // Use symmetry to initialize
            length_of_palindrome[i] = std::cmp::min(
                right_from_current_center - i,
                length_of_palindrome[2 * current_center - i],
            );
            // Update center if we might extend past right
            if length_of_palindrome[i] + i >= right_from_current_center {
                current_center = i;
                right_from_current_center = length_of_palindrome[i] + i;
            }
        }
        
        // Expand around center
        let mut radius = (length_of_palindrome[i] - 1) / 2 + 1;
        while i >= radius && i + radius < chars.len() 
              && chars[i - radius] == chars[i + radius] {
            length_of_palindrome[i] += 2;
            radius += 1;
        }
    }

    // Find and return longest palindrome
    let center_of_max = length_of_palindrome
        .iter()
        .enumerate()
        .max_by_key(|(_, &v)| v)
        .map(|(i, _)| i)
        .unwrap();
    
    let radius_of_max = (length_of_palindrome[center_of_max] - 1) / 2;
    chars[(center_of_max - radius_of_max)..=(center_of_max + radius_of_max)]
        .iter()
        .collect::<String>()
        .replace('#', "")
}
```

### 5.2 Edge Cases

| Input | Output |
|-------|--------|
| "" | "" |
| "a" | "a" |
| "ac" | "a" or "c" |
| "aa" | "aa" |
| "aba" | "aba" |
| "abba" | "abba" |
| "abcde" | Any single character |

### 5.3 Common Pitfalls

1. **Off-by-one errors:** Careful with radius vs. length calculations
2. **Integer overflow:** Use `usize` with care for subtraction
3. **Break condition:** Early termination when reaching string end
4. **Tie-breaking:** Multiple palindromes of same length

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Text Analysis:**
   - Finding palindromic DNA sequences
   - Text compression (palindromes have special structure)

2. **Interview Problems:**
   - Classic algorithm question
   - Foundation for related problems

3. **Natural Language Processing:**
   - Word play detection
   - Pattern finding in text

4. **Data Validation:**
   - Checking data integrity in symmetric formats
   - Finding mirrored patterns

### 6.2 Related Problems

**All Palindromic Substrings:**
```rust
fn count_palindromes(s: &str) -> usize {
    // Manacher gives P array
    // Count = sum((P[i] - 1) / 2) for all i
    let p = compute_manacher(s);
    p.iter().map(|&x| x / 2).sum()
}
```

**Palindrome Partitioning:**
Use Manacher as preprocessing to check palindrome in O(1).

## 7. Comparison with Other Approaches

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | $O(n^3)$ | $O(1)$ | Check all substrings |
| Expand Around Center | $O(n^2)$ | $O(1)$ | Simple, common |
| Dynamic Programming | $O(n^2)$ | $O(n^2)$ | dp[i][j] = is palindrome |
| **Manacher** | $O(n)$ | $O(n)$ | Optimal |
| Suffix Array + LCP | $O(n)$ | $O(n)$ | More complex |

## 8. References

1. Manacher, G. (1975). "A New Linear-Time 'On-Line' Algorithm for Finding the Smallest Initial Palindrome of a String". *JACM*.
2. Gusfield, D. "Algorithms on Strings, Trees, and Sequences", Section 9.2.
3. Jeuring, J. (1994). "The Derivation of On-Line Algorithms, with an Application to Finding Palindromes". *Algorithmica*.

## Implementation

See: [src/string/manacher.rs](../../src/string/manacher.rs)
