# Isogram Detection

## 1. Overview

An **isogram** (also known as a **non-pattern word** or **heterogram**) is a word or phrase in which no letter of the alphabet occurs more than once. The term was coined by Dmitri Borgmann in his book "Language on Vacation" (1965). Isogram detection is a simple but useful string property check.

## 2. Mathematical Foundation

### 2.1 Definition

A string $S$ is an **isogram** if and only if:
$$\forall c \in S: \text{count}(c, S) = 1$$

Or equivalently:
$$|\{c : c \in S\}| = |S|$$

The number of unique characters equals the string length.

### 2.2 Properties

1. **Length Limit:** For English alphabet, maximum isogram length is 26
2. **Empty String:** Vacuously an isogram
3. **Case Consideration:** Typically case-insensitive ("A" = "a")
4. **Space/Punctuation:** Usually ignored in traditional isograms

### 2.3 Types of Isograms

| Order | Definition | Example |
|-------|------------|---------|
| **First-order** | Each letter appears exactly once | "uncopyrightable" |
| **Second-order** | Each letter appears exactly twice | "intestines" |
| **Third-order** | Each letter appears exactly three times | "deeded" (if allowing 'd' 3x, 'e' 3x) |

## 3. Algorithm Description

### 3.1 Approach 1: Set-Based

```
function IS_ISOGRAM_SET(s):
    seen = empty set
    for c in s:
        if c is not alphabetic:
            continue
        c_lower = lowercase(c)
        if c_lower in seen:
            return false
        seen.add(c_lower)
    return true
```

### 3.2 Approach 2: Sorting

```
function IS_ISOGRAM_SORT(s):
    letters = extract_alphabetic_lowercase(s)
    sort(letters)
    for i = 1 to len(letters) - 1:
        if letters[i] = letters[i-1]:
            return false
    return true
```

### 3.3 Approach 3: Bit Manipulation (ASCII only)

```
function IS_ISOGRAM_BITS(s):
    seen = 0  // 26-bit integer
    for c in s:
        if c is not alphabetic:
            continue
        bit = 1 << (lowercase(c) - 'a')
        if seen AND bit ≠ 0:
            return false
        seen = seen OR bit
    return true
```

### 3.4 Step-by-Step Example

**String:** "Algorithm"

| Char | Lowercase | Seen Set | Action |
|------|-----------|----------|--------|
| A | a | {} | Add 'a' |
| l | l | {a} | Add 'l' |
| g | g | {a,l} | Add 'g' |
| o | o | {a,l,g} | Add 'o' |
| r | r | {a,l,g,o} | Add 'r' |
| i | i | {a,l,g,o,r} | Add 'i' |
| t | t | {a,l,g,o,r,i} | Add 't' |
| h | h | {a,l,g,o,r,i,t} | Add 'h' |
| m | m | {a,l,g,o,r,i,t,h} | Add 'm' |

All unique → **Is Isogram: true**

**String:** "Hello"

| Char | Lowercase | Seen Set | Action |
|------|-----------|----------|--------|
| H | h | {} | Add 'h' |
| e | e | {h} | Add 'e' |
| l | l | {h,e} | Add 'l' |
| l | l | {h,e,l} | 'l' already seen! |

Duplicate found → **Is Isogram: false**

## 4. Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Set | $O(n)$ | $O(\min(n, |\Sigma|))$ |
| Sort | $O(n \log n)$ | $O(n)$ |
| Bits | $O(n)$ | $O(1)$ |

Where $|\Sigma| = 26$ for English alphabet.

## 5. Implementation Notes

### 5.1 Rust Implementation - HashSet

```rust
use std::collections::HashSet;

pub fn is_isogram(s: &str) -> bool {
    let mut seen = HashSet::new();
    
    for c in s.chars() {
        if c.is_alphabetic() {
            let lower = c.to_ascii_lowercase();
            if !seen.insert(lower) {
                return false;
            }
        }
    }
    
    true
}
```

### 5.2 Rust Implementation - Bit Manipulation

```rust
pub fn is_isogram_bits(s: &str) -> bool {
    let mut seen: u32 = 0;
    
    for c in s.chars() {
        if c.is_ascii_alphabetic() {
            let bit = 1 << (c.to_ascii_lowercase() as u8 - b'a');
            if seen & bit != 0 {
                return false;
            }
            seen |= bit;
        }
    }
    
    true
}
```

### 5.3 Rust Implementation - Functional Style

```rust
pub fn is_isogram_functional(s: &str) -> bool {
    let letters: Vec<char> = s
        .chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    let unique: HashSet<char> = letters.iter().copied().collect();
    
    letters.len() == unique.len()
}
```

### 5.4 N-th Order Isogram Check

```rust
use std::collections::HashMap;

pub fn is_nth_order_isogram(s: &str, order: usize) -> bool {
    let mut counts: HashMap<char, usize> = HashMap::new();
    
    for c in s.chars() {
        if c.is_alphabetic() {
            *counts.entry(c.to_ascii_lowercase()).or_insert(0) += 1;
        }
    }
    
    counts.values().all(|&count| count == order)
}
```

### 5.5 Edge Cases

| Input | Result | Reason |
|-------|--------|--------|
| "" | true | Vacuously true |
| "a" | true | Single letter |
| "aA" | false | 'a' appears twice (case-insensitive) |
| "a b" | true | Spaces ignored |
| "ab-cd" | true | Hyphens ignored |
| "alphabet" | false | 'a' appears twice |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Word Games:**
   - Finding valid isogram words
   - Constraint in word puzzles

2. **Password Validation:**
   - Requiring unique characters
   - Entropy measurement

3. **Linguistic Research:**
   - Analyzing language patterns
   - Finding longest isograms

4. **Educational Tools:**
   - Teaching string algorithms
   - Vocabulary exercises

### 6.2 Example Applications

**Find All Isograms in Dictionary:**
```rust
fn find_isograms<'a>(dictionary: &[&'a str]) -> Vec<&'a str> {
    dictionary
        .iter()
        .filter(|word| is_isogram(word))
        .copied()
        .collect()
}
```

**Longest Isogram:**
```rust
fn longest_isogram<'a>(dictionary: &[&'a str]) -> Option<&'a str> {
    dictionary
        .iter()
        .filter(|word| is_isogram(word))
        .max_by_key(|word| word.len())
        .copied()
}
```

## 7. Famous Isograms

| Word | Length | Notes |
|------|--------|-------|
| "uncopyrightables" | 17 | Longest common English isogram |
| "dermatoglyphics" | 15 | Study of fingerprints |
| "misconjugatedly" | 15 | Grammatical term |
| "subdermatoglyphic" | 17 | Variant spelling |
| "ambidextrously" | 14 | Using both hands |

## 8. Variations

### 8.1 Strict Isogram (No Non-Letters)

```rust
pub fn is_strict_isogram(s: &str) -> bool {
    if !s.chars().all(|c| c.is_ascii_alphabetic()) {
        return false;
    }
    is_isogram(s)
}
```

### 8.2 Case-Sensitive Isogram

```rust
pub fn is_isogram_case_sensitive(s: &str) -> bool {
    let mut seen = HashSet::new();
    
    for c in s.chars() {
        if c.is_alphabetic() && !seen.insert(c) {
            return false;
        }
    }
    
    true
}
```

### 8.3 Isogram Substring

Find longest isogram substring:

```rust
pub fn longest_isogram_substring(s: &str) -> &str {
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    let mut best_start = 0;
    let mut best_len = 0;
    
    for start in 0..n {
        let mut seen: u32 = 0;
        for end in start..n {
            if !chars[end].is_ascii_alphabetic() {
                break;
            }
            let bit = 1 << (chars[end].to_ascii_lowercase() as u8 - b'a');
            if seen & bit != 0 {
                break;
            }
            seen |= bit;
            if end - start + 1 > best_len {
                best_len = end - start + 1;
                best_start = start;
            }
        }
    }
    
    &s[best_start..best_start + best_len]
}
```

## 9. References

1. Borgmann, D. A. (1965). "Language on Vacation: An Olio of Orthographical Oddities".
2. Eckler, A. R. (1996). "Making the Alphabet Dance".
3. Wikipedia: "Isogram".

## Implementation

See: [src/string/isogram.rs](../../src/string/isogram.rs)
