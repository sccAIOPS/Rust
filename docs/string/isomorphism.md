# String Isomorphism

## 1. Overview

Two strings are **isomorphic** if there exists a one-to-one mapping between characters such that replacing each character in one string with its mapped character yields the other string. String isomorphism is useful in pattern matching, cryptanalysis, and word game solving.

## 2. Mathematical Foundation

### 2.1 Definition

Strings $S$ and $T$ are **isomorphic** if there exists a bijection $f: \Sigma_S \to \Sigma_T$ such that:
$$\forall i: f(S_i) = T_i$$

Where $\Sigma_S$ and $\Sigma_T$ are the sets of characters in $S$ and $T$ respectively.

### 2.2 Properties

1. **Equal Length Required:** $|S| = |T|$
2. **One-to-One:** Different characters in $S$ must map to different characters in $T$
3. **Onto:** Each character in $T$ must be mapped from some character in $S$
4. **Order Preserved:** Character positions are preserved by the mapping
5. **Symmetric:** If $S \sim T$, then $T \sim S$ (using inverse mapping)
6. **Transitive:** If $S \sim T$ and $T \sim U$, then $S \sim U$ (compose mappings)

### 2.3 Canonical Form

Each equivalence class of isomorphic strings has a **canonical form**:
- Replace first unique character with 'a'
- Replace second unique character with 'b'
- Continue through alphabet

Example: "paper" → "abacr" → canonical: "abacb"

## 3. Algorithm Description

### 3.1 Two-Map Approach

Maintain two mappings and verify consistency.

```
function IS_ISOMORPHIC(s, t):
    if len(s) ≠ len(t):
        return false
    
    s_to_t = empty map
    t_to_s = empty map
    
    for i = 0 to len(s) - 1:
        c_s = s[i]
        c_t = t[i]
        
        // Check s → t mapping
        if c_s in s_to_t:
            if s_to_t[c_s] ≠ c_t:
                return false
        else:
            s_to_t[c_s] = c_t
        
        // Check t → s mapping (ensures bijection)
        if c_t in t_to_s:
            if t_to_s[c_t] ≠ c_s:
                return false
        else:
            t_to_s[c_t] = c_s
    
    return true
```

### 3.2 Pattern Encoding Approach

Convert both strings to canonical form and compare.

```
function TO_PATTERN(s):
    pattern = empty array
    mapping = empty map
    next_id = 0
    
    for c in s:
        if c not in mapping:
            mapping[c] = next_id
            next_id += 1
        pattern.append(mapping[c])
    
    return pattern

function IS_ISOMORPHIC_PATTERN(s, t):
    return TO_PATTERN(s) = TO_PATTERN(t)
```

### 3.3 Step-by-Step Example

**Strings:** "egg" vs "add"

**Two-Map Method:**

| i | s[i] | t[i] | s_to_t | t_to_s | Valid? |
|---|------|------|--------|--------|--------|
| 0 | e | a | {e→a} | {a→e} | ✓ |
| 1 | g | d | {e→a, g→d} | {a→e, d→g} | ✓ |
| 2 | g | d | check g→d ✓ | check d→g ✓ | ✓ |

**Result:** Isomorphic ✓

**Non-Isomorphic Example:** "foo" vs "bar"

| i | s[i] | t[i] | s_to_t | t_to_s | Valid? |
|---|------|------|--------|--------|--------|
| 0 | f | b | {f→b} | {b→f} | ✓ |
| 1 | o | a | {f→b, o→a} | {b→f, a→o} | ✓ |
| 2 | o | r | o→a exists, but a≠r | — | ✗ |

**Result:** Not Isomorphic ✗

## 4. Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Two-Map | $O(n)$ | $O(|\Sigma|)$ |
| Pattern | $O(n)$ | $O(|\Sigma|)$ |

Where $|\Sigma|$ is the alphabet size (constant for ASCII).

## 5. Implementation Notes

### 5.1 Rust Implementation - HashMap

```rust
use std::collections::HashMap;

pub fn is_isomorphic(s: &str, t: &str) -> bool {
    if s.len() != t.len() {
        return false;
    }
    
    let mut s_to_t: HashMap<char, char> = HashMap::new();
    let mut t_to_s: HashMap<char, char> = HashMap::new();
    
    for (c_s, c_t) in s.chars().zip(t.chars()) {
        // Check s → t mapping
        if let Some(&mapped) = s_to_t.get(&c_s) {
            if mapped != c_t {
                return false;
            }
        } else {
            s_to_t.insert(c_s, c_t);
        }
        
        // Check t → s mapping (ensures bijection)
        if let Some(&mapped) = t_to_s.get(&c_t) {
            if mapped != c_s {
                return false;
            }
        } else {
            t_to_s.insert(c_t, c_s);
        }
    }
    
    true
}
```

### 5.2 Pattern-Based Implementation

```rust
pub fn is_isomorphic_pattern(s: &str, t: &str) -> bool {
    fn to_pattern(s: &str) -> Vec<usize> {
        let mut map: HashMap<char, usize> = HashMap::new();
        let mut next_id = 0;
        
        s.chars()
            .map(|c| {
                *map.entry(c).or_insert_with(|| {
                    let id = next_id;
                    next_id += 1;
                    id
                })
            })
            .collect()
    }
    
    to_pattern(s) == to_pattern(t)
}
```

### 5.3 ASCII-Optimized Version

```rust
pub fn is_isomorphic_ascii(s: &str, t: &str) -> bool {
    if s.len() != t.len() {
        return false;
    }
    
    let mut s_to_t: [Option<u8>; 256] = [None; 256];
    let mut t_to_s: [Option<u8>; 256] = [None; 256];
    
    for (c_s, c_t) in s.bytes().zip(t.bytes()) {
        let s_idx = c_s as usize;
        let t_idx = c_t as usize;
        
        match (s_to_t[s_idx], t_to_s[t_idx]) {
            (Some(mapped), _) if mapped != c_t => return false,
            (_, Some(mapped)) if mapped != c_s => return false,
            _ => {
                s_to_t[s_idx] = Some(c_t);
                t_to_s[t_idx] = Some(c_s);
            }
        }
    }
    
    true
}
```

### 5.4 Get Isomorphism Mapping

```rust
pub fn get_mapping(s: &str, t: &str) -> Option<HashMap<char, char>> {
    if !is_isomorphic(s, t) {
        return None;
    }
    
    let mapping: HashMap<char, char> = s.chars()
        .zip(t.chars())
        .collect();
    
    Some(mapping)
}
```

### 5.5 Edge Cases

| s | t | Result | Reason |
|---|---|--------|--------|
| "" | "" | true | Empty strings |
| "a" | "b" | true | Single char |
| "ab" | "aa" | false | Two chars → one char |
| "aa" | "ab" | false | One char → two chars |
| "abc" | "def" | true | 1-to-1 mapping |
| "aba" | "xyx" | true | Pattern match |
| "aba" | "xyz" | false | Pattern mismatch |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Cryptanalysis:**
   - Substitution cipher breaking
   - Pattern recognition in encrypted text

2. **Word Games:**
   - Finding words with same pattern
   - Crossword puzzle solving

3. **Natural Language Processing:**
   - Pattern matching in text
   - Anonymization pattern detection

4. **Compiler Design:**
   - Pattern matching in parsers
   - Variable renaming detection

### 6.2 Example Applications

**Find Pattern Matches:**
```rust
fn find_pattern_matches<'a>(pattern: &str, words: &[&'a str]) -> Vec<&'a str> {
    words
        .iter()
        .filter(|&&word| is_isomorphic(pattern, word))
        .copied()
        .collect()
}

// Example: find_pattern_matches("aba", &["eye", "add", "noon"])
// Returns: ["eye", "noon"]
```

**Group Isomorphic Strings:**
```rust
fn group_isomorphic(words: Vec<&str>) -> HashMap<Vec<usize>, Vec<&str>> {
    let mut groups: HashMap<Vec<usize>, Vec<&str>> = HashMap::new();
    
    for word in words {
        let pattern = to_pattern(word);
        groups.entry(pattern).or_default().push(word);
    }
    
    groups
}
```

**Decode Substitution Cipher:**
```rust
fn decode_with_mapping(encrypted: &str, mapping: &HashMap<char, char>) -> String {
    encrypted
        .chars()
        .map(|c| *mapping.get(&c).unwrap_or(&c))
        .collect()
}
```

## 7. Variations

### 7.1 Case-Insensitive Isomorphism

```rust
pub fn is_isomorphic_ignore_case(s: &str, t: &str) -> bool {
    is_isomorphic(
        &s.to_lowercase(),
        &t.to_lowercase()
    )
}
```

### 7.2 Partial Isomorphism

Check if strings are isomorphic ignoring specific characters:

```rust
pub fn is_isomorphic_filtered(s: &str, t: &str, ignore: &HashSet<char>) -> bool {
    let s_filtered: String = s.chars().filter(|c| !ignore.contains(c)).collect();
    let t_filtered: String = t.chars().filter(|c| !ignore.contains(c)).collect();
    is_isomorphic(&s_filtered, &t_filtered)
}
```

### 7.3 Word Pattern Matching

Match pattern like "abba" with words like "dog cat cat dog":

```rust
pub fn word_pattern(pattern: &str, words: &str) -> bool {
    let pattern_chars: Vec<char> = pattern.chars().collect();
    let word_list: Vec<&str> = words.split_whitespace().collect();
    
    if pattern_chars.len() != word_list.len() {
        return false;
    }
    
    let mut char_to_word: HashMap<char, &str> = HashMap::new();
    let mut word_to_char: HashMap<&str, char> = HashMap::new();
    
    for (&p, &w) in pattern_chars.iter().zip(word_list.iter()) {
        if let Some(&mapped) = char_to_word.get(&p) {
            if mapped != w { return false; }
        } else {
            char_to_word.insert(p, w);
        }
        
        if let Some(&mapped) = word_to_char.get(w) {
            if mapped != p { return false; }
        } else {
            word_to_char.insert(w, p);
        }
    }
    
    true
}
```

## 8. Related Problems

| Problem | Description |
|---------|-------------|
| **Anagram** | Same letters, different arrangement |
| **Isomorphism** | Same structure, different characters |
| **Word Pattern** | Pattern matching with words |
| **Group Anagrams** | Cluster by character multiset |
| **Group Isomorphic** | Cluster by character pattern |

## 9. References

1. "Introduction to Algorithms" - Cormen et al., Pattern Matching chapter
2. "Programming Pearls" - Jon Bentley
3. LeetCode Problem 205: Isomorphic Strings

## Implementation

See: [src/string/isomorphism.rs](../../src/string/isomorphism.rs)
