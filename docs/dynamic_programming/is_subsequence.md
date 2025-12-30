# Is Subsequence

## 1. Overview

The Is Subsequence problem determines whether one string is a subsequence of another. A subsequence maintains relative ordering but doesn't require contiguity.

**File**: `src/dynamic_programming/is_subsequence.rs`

## 2. Mathematical Foundation

### 2.1 Definition

String $s$ is a subsequence of $t$ if:
- $s$ can be obtained by deleting characters from $t$
- Relative order is preserved

Formally, $s[0..m]$ is a subsequence of $t[0..n]$ if there exists indices $0 \leq i_0 < i_1 < \ldots < i_{m-1} < n$ such that $s[j] = t[i_j]$ for all $j$.

### 2.2 Examples

```
s = "ace", t = "abcde" → true
     a c e
     ↓ ↓ ↓
     a b c d e

s = "aec", t = "abcde" → false
     a e c  (e comes before c in s, but after in t)
```

## 3. Algorithm Description

### 3.1 Two-Pointer Approach (Greedy)

```
FUNCTION is_subsequence(s, t)
    i ← 0  // pointer for s
    j ← 0  // pointer for t
    
    WHILE i < len(s) AND j < len(t) DO
        IF s[i] = t[j] THEN
            i ← i + 1
        END IF
        j ← j + 1
    END WHILE
    
    RETURN i = len(s)
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: s = "ace", t = "abcde"

```
Step 1: s[0]='a' vs t[0]='a' → Match! i=1, j=1
Step 2: s[1]='c' vs t[1]='b' → No match, j=2
Step 3: s[1]='c' vs t[2]='c' → Match! i=2, j=3
Step 4: s[2]='e' vs t[3]='d' → No match, j=4
Step 5: s[2]='e' vs t[4]='e' → Match! i=3, j=5
Final: i=3 = len(s) → true
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n)** where n = len(t)
- Single pass through t

### 4.2 Space Complexity
- **O(1)** - only two pointers needed

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn is_subsequence(sub: &str, main_seq: &str) -> bool {
    let mut sub_iter = sub.chars().peekable();
    for main_char in main_seq.chars() {
        if sub_iter.peek() == Some(&main_char) {
            sub_iter.next();
        }
    }
    sub_iter.peek().is_none()
}
```

### 5.2 Key Design Choices

1. **Peekable Iterator**: Allows checking without consuming
2. **Unicode Support**: Uses `chars()` not byte indexing
3. **Early Termination**: Could optimize by checking if sub is consumed

### 5.3 Edge Cases

| Case | s | t | Result |
|------|---|---|--------|
| Empty subsequence | "" | "abc" | true |
| Empty main sequence | "a" | "" | false |
| Both empty | "" | "" | true |
| Equal strings | "abc" | "abc" | true |
| Longer sub | "abcd" | "abc" | false |

## 6. Alternative Approaches

### 6.1 DP Solution (for multiple queries)

Pre-process t to answer many s queries efficiently.

```rust
// Build next[i][c] = first occurrence of c at or after position i
fn build_next_table(t: &str) -> Vec<[Option<usize>; 26]> {
    let t: Vec<char> = t.chars().collect();
    let n = t.len();
    let mut next = vec![[None; 26]; n + 1];
    
    for i in (0..n).rev() {
        next[i] = next[i + 1];
        next[i][(t[i] as u8 - b'a') as usize] = Some(i);
    }
    next
}
```

### 6.2 Follow-Up: Multiple Queries

If checking many s strings against same t:
- Pre-process t: O(26n)
- Each query: O(m) where m = len(s)
- Total: O(26n + k·m) for k queries

## 7. Applications

1. **File Comparison**: Detecting minimal edit paths
2. **DNA Sequencing**: Finding genetic subsequences
3. **Autocomplete**: Fuzzy matching
4. **Version Control**: Finding common ancestors

## 8. Related Problems

| Problem | Difference |
|---------|------------|
| Longest Common Subsequence | Find longest, not just check |
| Edit Distance | Minimum operations to transform |
| Substring Search | Contiguous matching |
| Regular Expression | Pattern matching with wildcards |

## 9. Variations

### 9.1 Count Subsequences

Count how many ways s can be formed from t.

```
s = "ab", t = "aab"
     ↓↓     ↓↓   = 2 ways
     a·b    a·b
```

### 9.2 K-Subsequence

Check if s is subsequence when at most k characters can be skipped.

## 10. References

1. [LeetCode Problem 392](https://leetcode.com/problems/is-subsequence/)
2. [Wikipedia - Subsequence](https://en.wikipedia.org/wiki/Subsequence)
