# Jaro-Winkler Distance

## 1. Overview

The Jaro-Winkler distance is a string similarity metric designed for short strings, particularly useful for comparing names. Developed by William Winkler (1990) as an extension of Matthew Jaro's original algorithm (1989), it gives more favorable ratings to strings that match from the beginning—a common characteristic of spelling errors in names.

## 2. Mathematical Foundation

### 2.1 Jaro Similarity

The base Jaro similarity for strings $s_1$ and $s_2$ is:

$$sim_j = \begin{cases}
0 & \text{if } m = 0 \\
\frac{1}{3}\left(\frac{m}{|s_1|} + \frac{m}{|s_2|} + \frac{m - t}{m}\right) & \text{otherwise}
\end{cases}$$

Where:
- $m$ = number of matching characters
- $t$ = number of transpositions / 2

### 2.2 Matching Characters

Two characters are **matching** if they are the same and within the **match window**:

$$\text{match window} = \left\lfloor\frac{\max(|s_1|, |s_2|)}{2}\right\rfloor - 1$$

A character can only be matched once.

### 2.3 Transpositions

A **transposition** occurs when matched characters appear in different order. The transposition count $t$ equals half the number of matched characters that are not in the same sequence.

### 2.4 Jaro-Winkler Extension

$$sim_w = sim_j + \ell \cdot p \cdot (1 - sim_j)$$

Where:
- $\ell$ = length of common prefix (up to 4 characters)
- $p$ = scaling factor (default 0.1)

The common prefix bonus boosts similarity for strings with matching beginnings.

### 2.5 Distance Conversion

$$d_w = 1 - sim_w$$

The Jaro-Winkler distance is simply $1$ minus the similarity.

## 3. Algorithm Description

### 3.1 Intuition

1. **Match Phase:** Find characters that appear in both strings within the match window
2. **Transposition Phase:** Count how many matched characters are in different relative positions
3. **Jaro Phase:** Compute base similarity from matches and transpositions
4. **Winkler Phase:** Add bonus for common prefix

### 3.2 Pseudocode

```
function JARO_WINKLER(s1, s2, winkler_boost=true):
    if s1 = s2:
        return 1.0  // Perfect match
    
    len1 = len(s1)
    len2 = len(s2)
    
    if len1 = 0 or len2 = 0:
        return 0.0
    
    // Calculate match window
    match_distance = max(len1, len2) / 2 - 1
    
    s1_matches = array of false, size len1
    s2_matches = array of false, size len2
    
    matches = 0
    transpositions = 0
    
    // Find matching characters
    for i = 0 to len1 - 1:
        start = max(0, i - match_distance)
        end = min(i + match_distance + 1, len2)
        
        for j = start to end - 1:
            if s2_matches[j] or s1[i] ≠ s2[j]:
                continue
            s1_matches[i] = true
            s2_matches[j] = true
            matches += 1
            break
    
    if matches = 0:
        return 0.0
    
    // Count transpositions
    k = 0
    for i = 0 to len1 - 1:
        if not s1_matches[i]:
            continue
        while not s2_matches[k]:
            k += 1
        if s1[i] ≠ s2[k]:
            transpositions += 1
        k += 1
    
    // Jaro similarity
    jaro = (matches/len1 + matches/len2 + 
            (matches - transpositions/2)/matches) / 3
    
    // Winkler modification
    if winkler_boost:
        prefix = 0
        while prefix < min(4, len1, len2) and s1[prefix] = s2[prefix]:
            prefix += 1
        jaro = jaro + prefix * 0.1 * (1 - jaro)
    
    return jaro
```

### 3.3 Step-by-Step Example

**Strings:** "MARTHA" vs "MARHTA"

**Step 1: Match Window**
$$\left\lfloor\frac{\max(6, 6)}{2}\right\rfloor - 1 = 2$$

**Step 2: Find Matches**

| i | s1[i] | Window | Match in s2 | Position |
|---|-------|--------|-------------|----------|
| 0 | M | [0,2] | M | 0 |
| 1 | A | [0,3] | A | 1 |
| 2 | R | [0,4] | R | 2 |
| 3 | T | [1,5] | T | 4 |
| 4 | H | [2,5] | H | 3 |
| 5 | A | [3,5] | A | 5 |

All 6 characters match: $m = 6$

**Step 3: Count Transpositions**
- Matched positions in s1: [0, 1, 2, 3, 4, 5]
- Characters at those positions: M, A, R, T, H, A
- Matched positions in s2: [0, 1, 2, 4, 3, 5]
- Characters in match order: M, A, R, H, T, A
- Mismatches: (T,H) and (H,T) = 2 transpositions → $t = 2/2 = 1$

**Step 4: Jaro Similarity**
$$sim_j = \frac{1}{3}\left(\frac{6}{6} + \frac{6}{6} + \frac{6-1}{6}\right) = \frac{1}{3}(1 + 1 + 0.833) = 0.944$$

**Step 5: Winkler Bonus**
Common prefix: "MAR" ($\ell = 3$)
$$sim_w = 0.944 + 3 \times 0.1 \times (1 - 0.944) = 0.944 + 0.0168 = 0.961$$

## 4. Complexity Analysis

### 4.1 Time Complexity

$$O(n \cdot m)$$

Where $n = |s_1|$ and $m = |s_2|$.

For each character in $s_1$, we search within the match window in $s_2$.

### 4.2 Space Complexity

$$O(n + m)$$

Boolean arrays to track matched characters.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn jaro_winkler_distance(str_a: &str, str_b: &str) -> f64 {
    let vec_a: Vec<char> = str_a.chars().collect();
    let vec_b: Vec<char> = str_b.chars().collect();
    let len_a = vec_a.len();
    let len_b = vec_b.len();

    // Handle edge cases
    if len_a == 0 || len_b == 0 {
        return if len_a == 0 && len_b == 0 { 1.0 } else { 0.0 };
    }

    if vec_a == vec_b {
        return 1.0;
    }

    // Calculate match window
    let match_range = std::cmp::max(len_a, len_b) / 2;
    let match_range = if match_range > 0 { match_range - 1 } else { 0 };

    let mut matches = 0.0;
    let mut transpositions = 0.0;
    let mut b_match: Vec<bool> = vec![false; len_b];
    let mut a_match: Vec<bool> = vec![false; len_a];

    // Find matching characters
    for i in 0..len_a {
        let start = if i > match_range { i - match_range } else { 0 };
        let end = std::cmp::min(i + match_range + 1, len_b);

        for j in start..end {
            if b_match[j] || vec_a[i] != vec_b[j] {
                continue;
            }
            a_match[i] = true;
            b_match[j] = true;
            matches += 1.0;
            break;
        }
    }

    if matches == 0.0 {
        return 0.0;
    }

    // Count transpositions
    let mut k = 0;
    for i in 0..len_a {
        if !a_match[i] {
            continue;
        }
        while !b_match[k] {
            k += 1;
        }
        if vec_a[i] != vec_b[k] {
            transpositions += 1.0;
        }
        k += 1;
    }

    // Calculate Jaro similarity
    let jaro = (matches / len_a as f64 
              + matches / len_b as f64 
              + (matches - transpositions / 2.0) / matches) / 3.0;

    // Calculate common prefix (up to 4)
    let mut common_prefix = 0;
    for i in 0..std::cmp::min(4, std::cmp::min(len_a, len_b)) {
        if vec_a[i] == vec_b[i] {
            common_prefix += 1;
        } else {
            break;
        }
    }

    // Winkler modification
    jaro + common_prefix as f64 * 0.1 * (1.0 - jaro)
}
```

### 5.2 Implementation Considerations

**Unicode Handling:**
```rust
// Convert to Vec<char> for proper Unicode handling
let vec_a: Vec<char> = str_a.chars().collect();
```

**Scaling Factor Choice:**
- Standard $p = 0.1$
- Higher values favor prefix matches more
- Should not exceed $0.25$ (would allow similarity > 1)

### 5.3 Edge Cases

| s1 | s2 | Similarity |
|----|------|------------|
| "" | "" | 1.0 |
| "a" | "" | 0.0 |
| "a" | "a" | 1.0 |
| "abc" | "xyz" | 0.0 |
| "abc" | "abc" | 1.0 |
| "ab" | "ba" | 0.833 (transposition) |

## 6. Real-World Applications

### 6.1 Primary Use Cases

1. **Record Linkage:**
   - Matching names across databases
   - De-duplication in CRM systems

2. **Spell Checking:**
   - Suggesting corrections for misspelled names
   - Fuzzy search in autocomplete

3. **Data Quality:**
   - Identifying near-duplicates
   - Address matching

4. **Search Engines:**
   - Fuzzy name matching
   - "Did you mean?" suggestions

### 6.2 Example Applications

**Name Matching System:**
```rust
fn find_best_match(query: &str, names: &[&str]) -> Option<(&str, f64)> {
    names
        .iter()
        .map(|name| (*name, jaro_winkler_distance(query, name)))
        .max_by(|(_, a), (_, b)| a.partial_cmp(b).unwrap())
        .filter(|(_, sim)| *sim > 0.8)
}
```

**Threshold-Based Matching:**
```rust
fn are_similar_names(name1: &str, name2: &str) -> bool {
    jaro_winkler_distance(
        &name1.to_lowercase(),
        &name2.to_lowercase()
    ) > 0.85
}
```

## 7. Comparison with Other Metrics

| Metric | Best For | Prefix Sensitive | Complexity |
|--------|----------|------------------|------------|
| **Jaro-Winkler** | Short strings, names | Yes | O(nm) |
| Levenshtein | General editing | No | O(nm) |
| Hamming | Fixed length | No | O(n) |
| Soundex | Phonetic matching | N/A | O(n) |

## 8. Variations

### 8.1 Jaro Similarity Only

For cases where prefix bias is not desired:

```rust
fn jaro_similarity(s1: &str, s2: &str) -> f64 {
    // Same as above but skip Winkler modification
    jaro // Return Jaro score directly
}
```

### 8.2 Case-Insensitive Version

```rust
fn jaro_winkler_case_insensitive(s1: &str, s2: &str) -> f64 {
    jaro_winkler_distance(
        &s1.to_lowercase(),
        &s2.to_lowercase()
    )
}
```

### 8.3 Custom Prefix Length

```rust
fn jaro_winkler_custom(s1: &str, s2: &str, 
                        max_prefix: usize, 
                        scaling: f64) -> f64 {
    // Allow custom prefix length and scaling factor
}
```

## 9. Similarity Interpretation Guide

| Score Range | Interpretation |
|-------------|----------------|
| 0.95 - 1.00 | Excellent match (likely same entity) |
| 0.90 - 0.95 | Strong match (probable match) |
| 0.85 - 0.90 | Good match (possible match) |
| 0.70 - 0.85 | Moderate similarity |
| < 0.70 | Poor match (likely different) |

## 10. References

1. Jaro, M. A. (1989). "Advances in Record-Linkage Methodology as Applied to Matching the 1985 Census of Tampa, Florida". *Journal of the American Statistical Association*.
2. Winkler, W. E. (1990). "String Comparator Metrics and Enhanced Decision Rules in the Fellegi-Sunter Model of Record Linkage". *Proceedings of the Section on Survey Research Methods*.
3. Cohen, W., Ravikumar, P., & Fienberg, S. (2003). "A Comparison of String Metrics for Matching Names and Records".

## Implementation

See: [src/string/jaro_winkler_distance.rs](../../src/string/jaro_winkler_distance.rs)
