# Word Break Problem

## 1. Overview

The Word Break problem determines if a string can be segmented into a space-separated sequence of dictionary words. This implementation uses a Trie for efficient dictionary lookup combined with memoized recursion.

**File**: `src/dynamic_programming/word_break.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- String $s$ of length $n$
- Dictionary $D$ of words

Determine if $s$ can be written as $w_1 w_2 \ldots w_k$ where each $w_i \in D$.

### 2.2 Recurrence

Let $canBreak(i)$ = true if $s[i..n]$ can be segmented.

$$canBreak(i) = \bigvee_{j=i+1}^{n} (s[i..j] \in D \land canBreak(j))$$

Base case: $canBreak(n) = true$ (empty suffix)

## 3. Algorithm Description

### 3.1 Approach

1. Build a Trie from dictionary words
2. Use memoization to avoid recomputation
3. For each position, try all possible word endings

### 3.2 Pseudocode

```
FUNCTION word_break(s, dictionary)
    trie ← build_trie(dictionary)
    memo[0..len(s)] ← None
    RETURN search(trie, s, 0, memo)
END FUNCTION

FUNCTION search(trie, s, start, memo)
    IF start = len(s) THEN RETURN true
    IF memo[start] is Some(result) THEN RETURN result
    
    FOR end ← start+1 TO len(s) DO
        IF s[start..end] IN trie AND search(trie, s, end, memo) THEN
            memo[start] ← Some(true)
            RETURN true
        END IF
    END FOR
    
    memo[start] ← Some(false)
    RETURN false
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**: s = "applepenapple", dict = ["apple", "pen"]

```
Position 0: Try "a", "ap", "app", "appl", "apple" ✓
  → Found "apple", recurse from position 5
  
Position 5: Try "p", "pe", "pen" ✓
  → Found "pen", recurse from position 8
  
Position 8: Try "a", "ap", "app", "appl", "apple" ✓
  → Found "apple", recurse from position 13
  
Position 13: Base case (end of string) → true
```

**Result**: true ("apple" + "pen" + "apple")

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n²)** with memoization, where n = string length
- Trie lookup: O(word length)

### 4.2 Space Complexity
- **O(n)** for memoization
- **O(D)** for Trie where D = total dictionary characters

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn word_break(s: &str, word_dict: &[&str]) -> bool {
    let mut trie = Trie::new();
    for &word in word_dict {
        trie.insert(word.chars(), true);
    }

    let mut memo = vec![None; s.len() + 1];
    search(&trie, s, 0, &mut memo)
}

fn search(trie: &Trie<char, bool>, s: &str, start: usize, 
          memo: &mut Vec<Option<bool>>) -> bool {
    if start == s.len() { return true; }
    if let Some(res) = memo[start] { return res; }

    for end in start + 1..=s.len() {
        if trie.get(s[start..end].chars()).is_some() 
           && search(trie, s, end, memo) {
            memo[start] = Some(true);
            return true;
        }
    }

    memo[start] = Some(false);
    false
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty string | true |
| Empty dictionary | false (unless s is empty) |
| Single char match | true |
| No solution | false |

## 6. Applications

1. **NLP**: Tokenization without spaces (Chinese, Japanese)
2. **Spell Checking**: Compound word detection
3. **Search Engines**: Query segmentation

## 7. Variants

| Variant | Description |
|---------|-------------|
| Word Break II | Return all valid segmentations |
| Minimum cuts | Minimum spaces to add |
| Concatenated words | Find words formed by concatenation |

## 8. Why Use Trie?

```
Without Trie (HashSet): O(n) lookup per word
With Trie: O(k) lookup where k = word length

For long strings with many prefix checks,
Trie allows early termination on mismatch.
```

## 9. References

1. [LeetCode Problem 139](https://leetcode.com/problems/word-break/)
2. [Wikipedia - Trie](https://en.wikipedia.org/wiki/Trie)
