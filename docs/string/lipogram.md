# Lipogram Detection

## 1. Overview

A **lipogram** is a constrained writing form in which a particular letter or group of letters is deliberately avoided. The term comes from the Greek *leipográmmatos* meaning "leaving out a letter." Lipogram detection checks whether a text successfully avoids specified forbidden characters.

## 2. Mathematical Foundation

### 2.1 Definition

A string $S$ is a **lipogram** with respect to forbidden set $F$ if and only if:
$$\forall c \in F: c \notin S$$

Or equivalently:
$$S \cap F = \emptyset$$

### 2.2 Properties

1. **Empty String:** Always a valid lipogram (vacuously avoids all letters)
2. **Empty Forbidden Set:** Every string is trivially a lipogram
3. **Full Alphabet Forbidden:** Only non-alphabetic strings qualify
4. **Composition:** If $S$ avoids $F$ and $T$ avoids $F$, then $S + T$ avoids $F$

### 2.3 Lipogram Types

| Type | Description | Example |
|------|-------------|---------|
| **E-lipogram** | Avoids 'e' | "Gadsby" (entire novel) |
| **A-lipogram** | Avoids 'a' | More difficult in English |
| **Univocalic** | Uses only one vowel | "Persevere ye perfect men" |
| **Pangrammatic lipogram** | Uses all letters except one | Almost-pangrams |

## 3. Algorithm Description

### 3.1 Basic Algorithm

```
function IS_LIPOGRAM(text, forbidden):
    forbidden_set = set(forbidden)
    for c in text:
        if lowercase(c) in forbidden_set:
            return false
    return true
```

### 3.2 Find Missing Letters

```
function GET_MISSING_LETTERS(text):
    alphabet = {'a', 'b', ..., 'z'}
    used = set()
    for c in text:
        if c is alphabetic:
            used.add(lowercase(c))
    return alphabet - used
```

### 3.3 Bit Manipulation Approach

```
function IS_LIPOGRAM_BITS(text, forbidden):
    forbidden_mask = 0
    for c in forbidden:
        forbidden_mask |= (1 << (c - 'a'))
    
    for c in text:
        if c is alphabetic:
            bit = 1 << (lowercase(c) - 'a')
            if forbidden_mask AND bit ≠ 0:
                return false
    return true
```

### 3.4 Step-by-Step Example

**Text:** "The quick brown fox jumps over a lazy dog"
**Forbidden:** {'e'}

| Character | Lowercase | In Forbidden? |
|-----------|-----------|---------------|
| T | t | No ✓ |
| h | h | No ✓ |
| e | e | **Yes ✗** |

**Result:** Not a lipogram (contains 'e')

**Text:** "A dog ran across grass"
**Forbidden:** {'e'}

All characters checked, none match forbidden set.
**Result:** Valid lipogram

## 4. Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Set-based | $O(n + |F|)$ | $O(|F|)$ |
| Bit manipulation | $O(n + |F|)$ | $O(1)$ |

Where $n$ is text length and $|F|$ is forbidden set size.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
use std::collections::HashSet;

/// Checks if text is a lipogram avoiding the given character
pub fn is_lipogram(text: &str, forbidden: char) -> bool {
    let forbidden_lower = forbidden.to_ascii_lowercase();
    
    text.chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .all(|c| c != forbidden_lower)
}

/// Returns the set of letters missing from the text
pub fn get_missing_letters(text: &str) -> HashSet<char> {
    let alphabet: HashSet<char> = ('a'..='z').collect();
    let used: HashSet<char> = text
        .chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    alphabet.difference(&used).copied().collect()
}
```

### 5.2 Multiple Forbidden Characters

```rust
pub fn is_lipogram_multi(text: &str, forbidden: &[char]) -> bool {
    let forbidden_set: HashSet<char> = forbidden
        .iter()
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    text.chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .all(|c| !forbidden_set.contains(&c))
}
```

### 5.3 Bit Manipulation Version

```rust
pub fn is_lipogram_bits(text: &str, forbidden: &[char]) -> bool {
    // Build forbidden mask
    let mut forbidden_mask: u32 = 0;
    for &c in forbidden {
        if c.is_ascii_alphabetic() {
            forbidden_mask |= 1 << (c.to_ascii_lowercase() as u8 - b'a');
        }
    }
    
    // Check text
    for c in text.chars() {
        if c.is_ascii_alphabetic() {
            let bit = 1 << (c.to_ascii_lowercase() as u8 - b'a');
            if forbidden_mask & bit != 0 {
                return false;
            }
        }
    }
    
    true
}
```

### 5.4 Detailed Analysis

```rust
#[derive(Debug)]
pub struct LipogramAnalysis {
    pub is_valid: bool,
    pub missing_letters: HashSet<char>,
    pub violations: Vec<(usize, char)>,  // (position, character)
}

pub fn analyze_lipogram(text: &str, forbidden: &[char]) -> LipogramAnalysis {
    let forbidden_set: HashSet<char> = forbidden
        .iter()
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    let mut violations = Vec::new();
    let mut used = HashSet::new();
    
    for (i, c) in text.chars().enumerate() {
        if c.is_alphabetic() {
            let lower = c.to_ascii_lowercase();
            used.insert(lower);
            if forbidden_set.contains(&lower) {
                violations.push((i, c));
            }
        }
    }
    
    let alphabet: HashSet<char> = ('a'..='z').collect();
    let missing = alphabet.difference(&used).copied().collect();
    
    LipogramAnalysis {
        is_valid: violations.is_empty(),
        missing_letters: missing,
        violations,
    }
}
```

### 5.5 Edge Cases

| Text | Forbidden | Result |
|------|-----------|--------|
| "" | 'e' | true (vacuously) |
| "abc" | 'd' | true |
| "abcd" | 'd' | false |
| "ABC" | 'a' | false (case-insensitive) |
| "123" | 'e' | true (no letters) |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Creative Writing:**
   - Constrained writing exercises
   - Literary puzzles
   - Poetry with restrictions

2. **Language Learning:**
   - Vocabulary expansion exercises
   - Writing challenges

3. **Text Analysis:**
   - Detecting missing characters
   - Character frequency analysis

4. **Cryptography:**
   - Analyzing cipher outputs
   - Detecting biased character distributions

### 6.2 Famous Lipograms

| Work | Author | Avoided |
|------|--------|---------|
| "Gadsby" | E.V. Wright | 'e' |
| "La Disparition" | Georges Perec | 'e' |
| "A Void" | Translation of above | 'e' |
| "Eunoia" | Christian Bök | Uses only one vowel per chapter |

### 6.3 Example Applications

**Find Hardest Letter to Avoid:**
```rust
fn hardest_letter_to_avoid(texts: &[&str]) -> char {
    let mut letter_counts: [usize; 26] = [0; 26];
    
    for text in texts {
        for c in text.chars() {
            if c.is_ascii_alphabetic() {
                letter_counts[(c.to_ascii_lowercase() as u8 - b'a') as usize] += 1;
            }
        }
    }
    
    let max_idx = letter_counts
        .iter()
        .enumerate()
        .max_by_key(|(_, &count)| count)
        .map(|(i, _)| i)
        .unwrap();
    
    (b'a' + max_idx as u8) as char
}
```

## 7. Variations

### 7.1 Inverse Lipogram (Must Use Only Specified Letters)

```rust
pub fn uses_only(text: &str, allowed: &[char]) -> bool {
    let allowed_set: HashSet<char> = allowed
        .iter()
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    text.chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .all(|c| allowed_set.contains(&c))
}
```

### 7.2 Lipogram Score (How Close to Valid)

```rust
pub fn lipogram_score(text: &str, forbidden: &[char]) -> f64 {
    let forbidden_set: HashSet<char> = forbidden
        .iter()
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    let letters: Vec<char> = text
        .chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    if letters.is_empty() {
        return 1.0;
    }
    
    let violations = letters
        .iter()
        .filter(|c| forbidden_set.contains(c))
        .count();
    
    1.0 - (violations as f64 / letters.len() as f64)
}
```

### 7.3 Generate Lipogram Suggestion

```rust
pub fn suggest_replacements(text: &str, forbidden: char) -> Vec<(usize, String)> {
    // Returns positions and suggested replacement words
    // (Would require a dictionary/thesaurus in practice)
    todo!("Implement with word replacement suggestions")
}
```

## 8. Related Concepts

| Concept | Description |
|---------|-------------|
| **Pangram** | Uses every letter at least once |
| **Isogram** | Uses each letter at most once |
| **Univocalic** | Uses only one vowel |
| **Palindrome** | Reads same forwards and backwards |

## 9. References

1. Perec, G. (1969). "La Disparition" - Famous e-lipogram novel
2. Wright, E.V. (1939). "Gadsby" - 50,000 word novel without 'e'
3. Borgmann, D.A. (1965). "Language on Vacation"
4. Wikipedia: "Lipogram"

## Implementation

See: [src/string/lipogram.rs](../../src/string/lipogram.rs)
