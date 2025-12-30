# Duval Algorithm (Lyndon Factorization)

## 1. Overview

The **Duval Algorithm** computes the Lyndon factorization of a string in linear time. A **Lyndon word** is a non-empty string that is strictly smaller (lexicographically) than all of its proper rotations. Every string has a unique factorization into a non-increasing sequence of Lyndon words, called its Lyndon factorization.

## 2. Mathematical Foundation

### 2.1 Definitions

**Lyndon Word:** A string $w$ is a Lyndon word if:
$$w < w[i:] + w[:i] \quad \forall i \in (0, |w|)$$

Equivalently, $w$ is the lexicographically smallest rotation of itself.

**Lyndon Factorization:** Every string $S$ has a unique factorization:
$$S = L_1 L_2 ... L_k$$

Where each $L_i$ is a Lyndon word and $L_1 \geq L_2 \geq ... \geq L_k$ (lexicographically).

### 2.2 Properties

1. **Uniqueness:** The Lyndon factorization is unique
2. **Single Characters:** Every single character is a Lyndon word
3. **Concatenation:** If $u < v$ lexicographically, then $uv$ is Lyndon iff $uv$ is primitive
4. **Primitive:** A Lyndon word is always primitive (not a repetition of a smaller string)

### 2.3 Examples of Lyndon Words

| String | Lyndon? | Reason |
|--------|---------|--------|
| "a" | Yes | Single character |
| "ab" | Yes | "ab" < "ba" |
| "aab" | Yes | Smallest rotation |
| "ba" | No | "ab" < "ba" |
| "aba" | No | "aab" < "aba" |
| "abc" | Yes | Smallest rotation |

## 3. Algorithm Description

### 3.1 Duval Algorithm

```
function DUVAL(s):
    n = len(s)
    factorization = []
    i = 0
    
    while i < n:
        j = i + 1
        k = i
        
        while j < n and s[k] <= s[j]:
            if s[k] < s[j]:
                k = i
            else:
                k = k + 1
            j = j + 1
        
        // Output Lyndon words of length (j - k)
        while i <= k:
            factorization.append(s[i : i + (j - k)])
            i = i + (j - k)
    
    return factorization
```

### 3.2 Intuition

The algorithm works by:
1. Finding the longest prefix that is (almost) a Lyndon word
2. This prefix can be factored into identical Lyndon words
3. Each word has length `j - k` where `j` is current position and `k` tracks matching

### 3.3 Step-by-Step Example

**String:** "abbabaab"

| Step | i | j | k | s[k] vs s[j] | Action |
|------|---|---|---|--------------|--------|
| 1 | 0 | 1 | 0 | a < b | k = 0 |
| 2 | 0 | 2 | 0 | a < b | k = 0 |
| 3 | 0 | 3 | 0 | a = a | k = 1 |
| 4 | 0 | 4 | 1 | b = b | k = 2 |
| 5 | 0 | 5 | 2 | a < a | j ends |
| Output | — | — | — | — | "abb" (i=0, len=3) |
| 6 | 3 | 4 | 3 | a = a | k = 4 |
| 7 | 3 | 5 | 4 | b = a | j ends |
| Output | — | — | — | — | "ab" (i=3, len=2) |
| ... | | | | | Continue |

**Result:** ["abb", "ab", "aab"] (Note: verify manually)

## 4. Complexity Analysis

| Metric | Complexity |
|--------|------------|
| Time | $O(n)$ |
| Space | $O(1)$ auxiliary (plus output) |

The algorithm is optimal—each character is processed at most twice.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn duval_algorithm(s: &str) -> Vec<String> {
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    let mut factorization = Vec::new();
    let mut i = 0;
    
    while i < n {
        let mut j = i + 1;
        let mut k = i;
        
        while j < n && chars[k] <= chars[j] {
            if chars[k] < chars[j] {
                k = i;
            } else {
                k += 1;
            }
            j += 1;
        }
        
        // Output Lyndon words of length (j - k)
        let word_len = j - k;
        while i <= k {
            factorization.push(chars[i..i + word_len].iter().collect());
            i += word_len;
        }
    }
    
    factorization
}
```

### 5.2 Finding Minimum Rotation

A key application is finding the lexicographically smallest rotation:

```rust
pub fn min_rotation(s: &str) -> String {
    // Concatenate string with itself
    let doubled = format!("{}{}", s, s);
    let chars: Vec<char> = doubled.chars().collect();
    let n = s.len();
    
    let mut i = 0;
    let mut ans = 0;
    
    while i < n {
        let mut j = i + 1;
        let mut k = i;
        
        while j < 2 * n && chars[k] <= chars[j] {
            if chars[k] < chars[j] {
                k = i;
            } else {
                k += 1;
            }
            j += 1;
        }
        
        while i <= k {
            if i < n {
                ans = i;
            }
            i += j - k;
        }
    }
    
    format!("{}{}", &s[ans..], &s[..ans])
}
```

### 5.3 Check if Lyndon Word

```rust
pub fn is_lyndon(s: &str) -> bool {
    if s.is_empty() {
        return false;
    }
    
    let factorization = duval_algorithm(s);
    factorization.len() == 1 && factorization[0] == s
}
```

### 5.4 Lyndon Word Generator

```rust
pub fn generate_lyndon_words(alphabet: &[char], max_len: usize) -> Vec<String> {
    let mut result = Vec::new();
    let k = alphabet.len();
    
    fn generate(
        alphabet: &[char],
        w: &mut Vec<usize>,
        last_start: usize,
        max_len: usize,
        result: &mut Vec<String>,
    ) {
        if !w.is_empty() {
            let word: String = w.iter().map(|&i| alphabet[i]).collect();
            result.push(word);
        }
        
        if w.len() < max_len {
            let start = w.get(last_start).copied().unwrap_or(0);
            for i in start..alphabet.len() {
                w.push(i);
                let new_start = if w.len() > 1 && w[w.len() - 1] > w[w.len() - 2] {
                    w.len() - 1
                } else {
                    last_start + 1
                };
                generate(alphabet, w, new_start, max_len, result);
                w.pop();
            }
        }
    }
    
    let mut w = Vec::new();
    generate(alphabet, &mut w, 0, max_len, &mut result);
    result
}
```

### 5.5 Edge Cases

| Input | Factorization |
|-------|---------------|
| "" | [] |
| "a" | ["a"] |
| "ab" | ["ab"] |
| "ba" | ["b", "a"] |
| "aaa" | ["a", "a", "a"] |
| "abc" | ["abc"] |

## 6. Real-World Applications

### 6.1 Use Cases

1. **String Comparison:**
   - Lexicographically smallest rotation
   - Circular string comparison

2. **Combinatorics:**
   - Counting Lyndon words
   - Necklace enumeration

3. **Data Compression:**
   - Burrows-Wheeler Transform optimization
   - String preprocessing

4. **Pattern Matching:**
   - Suffix array construction
   - String periodicity

### 6.2 Necklace Comparison

Comparing circular strings (necklaces):

```rust
fn necklace_equal(s1: &str, s2: &str) -> bool {
    if s1.len() != s2.len() {
        return false;
    }
    min_rotation(s1) == min_rotation(s2)
}
```

### 6.3 Standard Form

Finding canonical form of a circular string:

```rust
fn canonical_form(s: &str) -> String {
    min_rotation(s)
}
```

## 7. Related Concepts

### 7.1 Connection to Other Algorithms

| Algorithm | Relation |
|-----------|----------|
| **Suffix Array** | Can use Lyndon factorization |
| **BWT** | Lyndon words appear in theory |
| **KMP** | Similar failure function concept |
| **Z-algorithm** | Related string structure |

### 7.2 Counting Lyndon Words

Number of Lyndon words of length $n$ over alphabet of size $k$:

$$L(n, k) = \frac{1}{n} \sum_{d|n} \mu(d) k^{n/d}$$

Where $\mu$ is the Möbius function.

## 8. Variations

### 8.1 Maximum Rotation (Booth's Algorithm)

```rust
pub fn booth_algorithm(s: &str) -> usize {
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    
    let mut f = vec![-1i32; 2 * n];
    let mut k = 0usize;
    
    for j in 1..2 * n {
        let sj = chars[j % n];
        let mut i = f[j - k - 1];
        
        while i != -1 && sj != chars[(k + i as usize + 1) % n] {
            if sj < chars[(k + i as usize + 1) % n] {
                k = j - i as usize - 1;
            }
            i = f[i as usize];
        }
        
        if sj != chars[(k + i as usize + 1) % n] {
            if sj < chars[k % n] {
                k = j;
            }
            f[j - k] = -1;
        } else {
            f[j - k] = i + 1;
        }
    }
    
    k
}
```

## 9. References

1. Duval, J.-P. (1983). "Factorizing Words over an Ordered Alphabet". *Journal of Algorithms*.
2. Lothaire, M. (2002). "Algebraic Combinatorics on Words", Cambridge University Press.
3. Booth, K. S. (1980). "Lexicographically Least Circular Substrings". *Information Processing Letters*.

## Implementation

See: [src/string/duval_algorithm.rs](../../src/string/duval_algorithm.rs)
