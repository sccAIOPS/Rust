# Anagram Detection

## 1. Overview

An **anagram** is a word or phrase formed by rearranging the letters of another, using all original letters exactly once. Anagram detection is fundamental in word games, cryptography, and text analysis. The algorithm determines whether two strings are anagrams of each other.

## 2. Mathematical Foundation

### 2.1 Definition

Two strings $S$ and $T$ are **anagrams** if and only if:
$$\forall c \in \Sigma: \text{count}(c, S) = \text{count}(c, T)$$

Where $\Sigma$ is the alphabet and $\text{count}(c, X)$ is the frequency of character $c$ in string $X$.

### 2.2 Properties

1. **Reflexive:** Every string is an anagram of itself
2. **Symmetric:** If $S$ is anagram of $T$, then $T$ is anagram of $S$
3. **Transitive:** If $S \sim T$ and $T \sim U$, then $S \sim U$
4. **Length Invariant:** $|S| = |T|$ (necessary but not sufficient)

### 2.3 Equivalence Classes

Anagram relationship defines equivalence classes on strings. All anagrams of a string form one equivalence class. The **canonical form** (sorted string) uniquely identifies each class.

## 3. Algorithm Description

### 3.1 Approach 1: Sorting

Sort both strings and compare.

```
function IS_ANAGRAM_SORT(s, t):
    if len(s) ≠ len(t):
        return false
    return sort(s) = sort(t)
```

**Time:** $O(n \log n)$ **Space:** $O(n)$

### 3.2 Approach 2: Character Counting

Count character frequencies and compare.

```
function IS_ANAGRAM_COUNT(s, t):
    if len(s) ≠ len(t):
        return false
    
    count = array of 0s, size 26 (or larger for Unicode)
    
    for i = 0 to len(s) - 1:
        count[s[i]] += 1
        count[t[i]] -= 1
    
    for c in count:
        if c ≠ 0:
            return false
    
    return true
```

**Time:** $O(n)$ **Space:** $O(1)$ for fixed alphabet

### 3.3 Approach 3: Prime Product (Mathematical)

Assign each letter a unique prime and compare products.

```
function IS_ANAGRAM_PRIME(s, t):
    if len(s) ≠ len(t):
        return false
    
    primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 
              43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101]
    
    product_s = 1
    product_t = 1
    
    for i = 0 to len(s) - 1:
        product_s *= primes[index(s[i])]
        product_t *= primes[index(t[i])]
    
    return product_s = product_t
```

**Time:** $O(n)$ **Space:** $O(1)$
**Caveat:** Overflow for long strings; use BigInt if needed.

### 3.4 Step-by-Step Example

**Strings:** "listen" vs "silent"

**Method: Character Count**

| Step | Char s | Char t | Count after operation |
|------|--------|--------|----------------------|
| 0 | l(+1) | s(-1) | l:1, s:-1 |
| 1 | i(+1) | i(-1) | l:1, s:-1, i:0 |
| 2 | s(+1) | l(-1) | l:0, s:0, i:0 |
| 3 | t(+1) | e(-1) | l:0, s:0, i:0, t:1, e:-1 |
| 4 | e(+1) | n(-1) | t:1, e:0, n:-1 |
| 5 | n(+1) | t(-1) | t:0, n:0 |

All counts are 0 → **Anagrams**

## 4. Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Sorting | $O(n \log n)$ | $O(n)$ | Simple, universal |
| Counting | $O(n)$ | $O(|\Sigma|)$ | Optimal for small alphabet |
| Prime | $O(n)$ | $O(1)$ | Risk of overflow |

## 5. Implementation Notes

### 5.1 Rust Implementation - Sorting Approach

```rust
pub fn is_anagram_sort(s: &str, t: &str) -> bool {
    let mut s_chars: Vec<char> = s.chars().collect();
    let mut t_chars: Vec<char> = t.chars().collect();
    
    s_chars.sort_unstable();
    t_chars.sort_unstable();
    
    s_chars == t_chars
}
```

### 5.2 Rust Implementation - Counting Approach

```rust
use std::collections::HashMap;

pub fn is_anagram(s: &str, t: &str) -> bool {
    if s.len() != t.len() {
        return false;
    }
    
    let mut char_count: HashMap<char, i32> = HashMap::new();
    
    for (c_s, c_t) in s.chars().zip(t.chars()) {
        *char_count.entry(c_s).or_insert(0) += 1;
        *char_count.entry(c_t).or_insert(0) -= 1;
    }
    
    char_count.values().all(|&count| count == 0)
}
```

### 5.3 ASCII-Only Optimization

```rust
pub fn is_anagram_ascii(s: &str, t: &str) -> bool {
    if s.len() != t.len() {
        return false;
    }
    
    let mut counts = [0i32; 26];
    
    for (c_s, c_t) in s.bytes().zip(t.bytes()) {
        if c_s.is_ascii_alphabetic() && c_t.is_ascii_alphabetic() {
            counts[(c_s.to_ascii_lowercase() - b'a') as usize] += 1;
            counts[(c_t.to_ascii_lowercase() - b'a') as usize] -= 1;
        }
    }
    
    counts.iter().all(|&c| c == 0)
}
```

### 5.4 Case-Insensitive with Space Handling

```rust
pub fn is_anagram_clean(s: &str, t: &str) -> bool {
    fn normalize(s: &str) -> Vec<char> {
        s.chars()
            .filter(|c| c.is_alphabetic())
            .map(|c| c.to_ascii_lowercase())
            .collect()
    }
    
    let s_clean = normalize(s);
    let t_clean = normalize(t);
    
    if s_clean.len() != t_clean.len() {
        return false;
    }
    
    let mut counts: HashMap<char, i32> = HashMap::new();
    for (&cs, &ct) in s_clean.iter().zip(t_clean.iter()) {
        *counts.entry(cs).or_insert(0) += 1;
        *counts.entry(ct).or_insert(0) -= 1;
    }
    
    counts.values().all(|&v| v == 0)
}
```

### 5.5 Edge Cases

| s | t | Result | Reason |
|---|---|--------|--------|
| "" | "" | true | Empty strings |
| "a" | "" | false | Different lengths |
| "a" | "a" | true | Same string |
| "ab" | "ba" | true | Simple swap |
| "ab" | "ab" | true | Identical |
| "aab" | "bba" | false | Different counts |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Word Games:**
   - Scrabble cheating detection
   - Crossword puzzle solvers
   - Word scramble games

2. **Cryptanalysis:**
   - Transposition cipher breaking
   - Pattern recognition in encrypted text

3. **Search & Indexing:**
   - Anagram-based search indexes
   - Grouping related words

4. **Data Processing:**
   - Detecting rearranged data
   - Plagiarism detection assistance

### 6.2 Example Applications

**Find All Anagrams in Dictionary:**
```rust
use std::collections::HashMap;

fn group_anagrams(words: Vec<&str>) -> HashMap<String, Vec<&str>> {
    let mut groups: HashMap<String, Vec<&str>> = HashMap::new();
    
    for word in words {
        let mut key: Vec<char> = word.to_lowercase().chars().collect();
        key.sort_unstable();
        let key: String = key.into_iter().collect();
        
        groups.entry(key).or_default().push(word);
    }
    
    groups
}
```

**Find Anagrams of a Word:**
```rust
fn find_anagrams<'a>(word: &str, dictionary: &[&'a str]) -> Vec<&'a str> {
    let mut sorted: Vec<char> = word.to_lowercase().chars().collect();
    sorted.sort_unstable();
    
    dictionary
        .iter()
        .filter(|&w| {
            if w.len() != word.len() { return false; }
            let mut w_sorted: Vec<char> = w.to_lowercase().chars().collect();
            w_sorted.sort_unstable();
            w_sorted == sorted
        })
        .copied()
        .collect()
}
```

## 7. Variations

### 7.1 Find All Anagram Substrings

Find all starting indices where an anagram of `pattern` begins in `text`:

```rust
fn find_anagram_indices(text: &str, pattern: &str) -> Vec<usize> {
    let text: Vec<char> = text.chars().collect();
    let pattern: Vec<char> = pattern.chars().collect();
    let n = text.len();
    let m = pattern.len();
    
    if m > n { return vec![]; }
    
    let mut result = vec![];
    let mut pattern_count = [0i32; 26];
    let mut window_count = [0i32; 26];
    
    // Count pattern characters
    for &c in &pattern {
        pattern_count[(c as u8 - b'a') as usize] += 1;
    }
    
    // Initial window
    for i in 0..m {
        window_count[(text[i] as u8 - b'a') as usize] += 1;
    }
    
    if window_count == pattern_count {
        result.push(0);
    }
    
    // Slide window
    for i in m..n {
        window_count[(text[i] as u8 - b'a') as usize] += 1;
        window_count[(text[i - m] as u8 - b'a') as usize] -= 1;
        
        if window_count == pattern_count {
            result.push(i - m + 1);
        }
    }
    
    result
}
```

### 7.2 K-Anagram (Allow K Differences)

```rust
fn is_k_anagram(s: &str, t: &str, k: usize) -> bool {
    if s.len() != t.len() { return false; }
    
    let mut counts: HashMap<char, i32> = HashMap::new();
    
    for c in s.chars() {
        *counts.entry(c).or_insert(0) += 1;
    }
    for c in t.chars() {
        *counts.entry(c).or_insert(0) -= 1;
    }
    
    let diff: i32 = counts.values().filter(|&&v| v > 0).sum();
    diff as usize <= k
}
```

## 8. Performance Comparison

| Method | n=10 | n=100 | n=1000 | n=10000 |
|--------|------|-------|--------|---------|
| Sort | ~50ns | ~1μs | ~15μs | ~200μs |
| HashMap | ~100ns | ~500ns | ~5μs | ~50μs |
| Array | ~30ns | ~200ns | ~2μs | ~20μs |

Array-based counting is fastest for ASCII strings.

## 9. References

1. Knuth, D.E. "The Art of Computer Programming, Vol. 3: Sorting and Searching".
2. "Programming Pearls" by Jon Bentley, Chapter 2.
3. Skiena, S.S. "The Algorithm Design Manual", String Problems.

## Implementation

See: [src/string/anagram.rs](../../src/string/anagram.rs)
