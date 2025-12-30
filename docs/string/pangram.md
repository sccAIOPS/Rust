# Pangram Detection

## 1. Overview

A **pangram** (from Greek *pan gramma*, "every letter") is a sentence that contains every letter of the alphabet at least once. The most famous English pangram is "The quick brown fox jumps over the lazy dog." Pangram detection verifies whether a given text qualifies as a pangram for a specified alphabet.

## 2. Mathematical Foundation

### 2.1 Definition

A string $S$ is a **pangram** over alphabet $\Sigma$ if and only if:
$$\forall c \in \Sigma: c \in S$$

For English: $\Sigma = \{a, b, c, ..., z\}$ (26 letters)

### 2.2 Properties

1. **Minimum Length:** A pangram must have at least $|\Sigma|$ alphabetic characters
2. **Perfect Pangram:** Uses each letter exactly once (length = 26 for English)
3. **Short Pangram:** Shortest possible pangrams minimize total length

### 2.3 Types

| Type | Description | Example |
|------|-------------|---------|
| **Standard** | Every letter at least once | "The quick brown fox jumps over a lazy dog" |
| **Perfect** | Every letter exactly once | "Mr Jock, TV quiz PhD, bags few lynx" (35 chars) |
| **Self-enumerating** | Describes its own letter count | Complex constructions |
| **Lipogrammatic** | Pangram missing one letter | Near-pangrams |

## 3. Algorithm Description

### 3.1 Set-Based Approach

```
function IS_PANGRAM(text):
    alphabet = {'a', 'b', ..., 'z'}
    found = empty set
    
    for c in text:
        if c is alphabetic:
            found.add(lowercase(c))
            if |found| = 26:
                return true  // Early exit
    
    return |found| = 26
```

### 3.2 Bit Manipulation Approach

```
function IS_PANGRAM_BITS(text):
    seen = 0  // 26-bit integer
    
    for c in text:
        if c is alphabetic:
            bit = 1 << (lowercase(c) - 'a')
            seen = seen OR bit
            if seen = 0x3FFFFFF:  // All 26 bits set
                return true
    
    return seen = 0x3FFFFFF
```

### 3.3 Counting Approach

```
function IS_PANGRAM_COUNT(text):
    counts = array of 0s, size 26
    unique = 0
    
    for c in text:
        if c is alphabetic:
            idx = lowercase(c) - 'a'
            if counts[idx] = 0:
                unique += 1
                if unique = 26:
                    return true
            counts[idx] += 1
    
    return unique = 26
```

### 3.4 Step-by-Step Example

**Text:** "Pack my box with five dozen liquor jugs"

| Iteration | Character | Found Set | Count |
|-----------|-----------|-----------|-------|
| 1-4 | P,a,c,k | {p,a,c,k} | 4 |
| 5-6 | m,y | +{m,y} | 6 |
| 7-9 | b,o,x | +{b,o,x} | 9 |
| 10-13 | w,i,t,h | +{w,i,t,h} | 13 |
| 14-17 | f,i,v,e | +{f,v,e} | 16 |
| 18-22 | d,o,z,e,n | +{d,z,n} | 19 |
| 23-28 | l,i,q,u,o,r | +{l,q,u,r} | 23 |
| 29-32 | j,u,g,s | +{j,g,s} | 26 ✓ |

**Result:** Is Pangram ✓

## 4. Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Set | $O(n)$ | $O(|\Sigma|)$ = $O(26)$ = $O(1)$ |
| Bits | $O(n)$ | $O(1)$ |
| Counting | $O(n)$ | $O(|\Sigma|)$ = $O(1)$ |

All approaches are $O(n)$ time with constant space for fixed alphabet.

## 5. Implementation Notes

### 5.1 Rust Implementation - HashSet

```rust
use std::collections::HashSet;

pub fn is_pangram(text: &str) -> bool {
    let alphabet: HashSet<char> = ('a'..='z').collect();
    
    let found: HashSet<char> = text
        .chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    alphabet.is_subset(&found)
}
```

### 5.2 Rust Implementation - Bit Manipulation

```rust
pub fn is_pangram_bits(text: &str) -> bool {
    const ALL_LETTERS: u32 = (1 << 26) - 1;  // 0x3FFFFFF
    
    let mut seen: u32 = 0;
    
    for c in text.chars() {
        if c.is_ascii_alphabetic() {
            let bit = 1 << (c.to_ascii_lowercase() as u8 - b'a');
            seen |= bit;
            if seen == ALL_LETTERS {
                return true;  // Early exit
            }
        }
    }
    
    seen == ALL_LETTERS
}
```

### 5.3 Get Missing Letters

```rust
pub fn missing_letters(text: &str) -> Vec<char> {
    let mut seen: u32 = 0;
    
    for c in text.chars() {
        if c.is_ascii_alphabetic() {
            seen |= 1 << (c.to_ascii_lowercase() as u8 - b'a');
        }
    }
    
    ('a'..='z')
        .filter(|&c| seen & (1 << (c as u8 - b'a')) == 0)
        .collect()
}
```

### 5.4 Perfect Pangram Check

```rust
pub fn is_perfect_pangram(text: &str) -> bool {
    let mut counts = [0u8; 26];
    
    for c in text.chars() {
        if c.is_ascii_alphabetic() {
            let idx = (c.to_ascii_lowercase() as u8 - b'a') as usize;
            counts[idx] += 1;
            if counts[idx] > 1 {
                return false;  // Letter repeated
            }
        }
    }
    
    counts.iter().all(|&c| c == 1)
}
```

### 5.5 Edge Cases

| Input | is_pangram | Reason |
|-------|------------|--------|
| "" | false | No letters |
| "abcdefghijklmnopqrstuvwxyz" | true | Minimal pangram |
| "ABCDEFGHIJKLMNOPQRSTUVWXYZ" | true | Case-insensitive |
| "abc...xyz" (missing 'q') | false | Missing letter |
| "the quick brown fox..." | true | Classic pangram |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Typography:**
   - Font preview sentences
   - Testing character rendering

2. **Keyboard Testing:**
   - Verify all keys work
   - Touch typing practice

3. **Data Quality:**
   - Ensure all characters supported
   - Character encoding verification

4. **Games & Puzzles:**
   - Word games
   - Crossword validation

### 6.2 Famous Pangrams

| Pangram | Length | Notes |
|---------|--------|-------|
| "The quick brown fox jumps over the lazy dog" | 35 | Most famous |
| "Pack my box with five dozen liquor jugs" | 32 | Short |
| "Sphinx of black quartz, judge my vow" | 29 | Shorter |
| "How vexingly quick daft zebras jump!" | 30 | Fun |
| "Mr Jock, TV quiz PhD, bags few lynx" | 26 | Near-perfect |

### 6.3 Example Applications

**Font Preview Generator:**
```rust
fn get_pangram_for_preview() -> &'static str {
    "The quick brown fox jumps over the lazy dog"
}

fn get_short_pangram() -> &'static str {
    "Pack my box with five dozen liquor jugs"
}
```

**Text Quality Check:**
```rust
fn analyze_text_coverage(text: &str) -> (usize, Vec<char>) {
    let mut seen: u32 = 0;
    
    for c in text.chars() {
        if c.is_ascii_alphabetic() {
            seen |= 1 << (c.to_ascii_lowercase() as u8 - b'a');
        }
    }
    
    let count = seen.count_ones() as usize;
    let missing: Vec<char> = ('a'..='z')
        .filter(|&c| seen & (1 << (c as u8 - b'a')) == 0)
        .collect();
    
    (count, missing)
}
```

## 7. Variations

### 7.1 Custom Alphabet Pangram

```rust
pub fn is_pangram_custom(text: &str, alphabet: &[char]) -> bool {
    let required: HashSet<char> = alphabet
        .iter()
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    let found: HashSet<char> = text
        .chars()
        .filter(|c| c.is_alphabetic())
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    required.is_subset(&found)
}
```

### 7.2 Pangram Score (Percentage Coverage)

```rust
pub fn pangram_score(text: &str) -> f64 {
    let mut seen: u32 = 0;
    
    for c in text.chars() {
        if c.is_ascii_alphabetic() {
            seen |= 1 << (c.to_ascii_lowercase() as u8 - b'a');
        }
    }
    
    seen.count_ones() as f64 / 26.0
}
```

### 7.3 Find Shortest Pangram Substring

```rust
pub fn shortest_pangram_window(text: &str) -> Option<(usize, usize)> {
    let chars: Vec<char> = text
        .chars()
        .map(|c| c.to_ascii_lowercase())
        .collect();
    
    const ALL: u32 = (1 << 26) - 1;
    let mut counts = [0u32; 26];
    let mut seen: u32 = 0;
    let mut unique = 0;
    
    let mut best: Option<(usize, usize)> = None;
    let mut left = 0;
    
    for right in 0..chars.len() {
        if chars[right].is_ascii_alphabetic() {
            let idx = (chars[right] as u8 - b'a') as usize;
            if counts[idx] == 0 {
                unique += 1;
                seen |= 1 << idx;
            }
            counts[idx] += 1;
        }
        
        // Try to shrink window
        while unique == 26 {
            if best.is_none() || right - left < best.unwrap().1 - best.unwrap().0 {
                best = Some((left, right));
            }
            
            if chars[left].is_ascii_alphabetic() {
                let idx = (chars[left] as u8 - b'a') as usize;
                counts[idx] -= 1;
                if counts[idx] == 0 {
                    unique -= 1;
                }
            }
            left += 1;
        }
    }
    
    best
}
```

## 8. Comparison with Related Concepts

| Concept | Definition |
|---------|------------|
| **Pangram** | Contains all letters (at least once) |
| **Perfect Pangram** | Contains all letters (exactly once) |
| **Lipogram** | Missing specific letter(s) |
| **Isogram** | No repeated letters |

## 9. References

1. "Word Ways: The Journal of Recreational Linguistics"
2. Wikipedia: "Pangram"
3. Borgmann, D.A. (1965). "Language on Vacation"

## Implementation

See: [src/string/pangram.rs](../../src/string/pangram.rs)
