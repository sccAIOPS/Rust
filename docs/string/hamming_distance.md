# Hamming Distance

## 1. Overview

The Hamming distance, introduced by Richard Hamming in 1950, measures the number of positions at which corresponding symbols of two strings of equal length differ. It is one of the simplest and most efficient string distance metrics, widely used in coding theory, cryptography, and error detection.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given two strings $S$ and $T$ of equal length $n$, the Hamming distance is:

$$d_H(S, T) = |\{i : S_i \neq T_i, 0 \leq i < n\}|$$

**Requirement:** Strings must have the same length.

### 2.2 Mathematical Properties

1. **Metric Space Properties:**
   - $d_H(S, T) \geq 0$ (non-negativity)
   - $d_H(S, T) = 0 \iff S = T$ (identity)
   - $d_H(S, T) = d_H(T, S)$ (symmetry)
   - $d_H(S, U) \leq d_H(S, T) + d_H(T, U)$ (triangle inequality)

2. **Bounds:**
   - $0 \leq d_H(S, T) \leq n$
   - $d_H(S, T) = n \iff$ no character matches

3. **Relation to Other Metrics:**
   - Hamming distance = Levenshtein distance when only substitutions allowed
   - For binary strings: $d_H(S, T) = popcount(S \oplus T)$

### 2.3 Error Detection/Correction

In coding theory:
- **Error Detection:** Can detect up to $d-1$ errors if minimum Hamming distance between codewords is $d$
- **Error Correction:** Can correct up to $\lfloor(d-1)/2\rfloor$ errors

## 3. Algorithm Description

### 3.1 Intuition

Simply iterate through both strings simultaneously and count positions where characters differ.

### 3.2 Pseudocode

```
function HAMMING_DISTANCE(s1, s2):
    if len(s1) ≠ len(s2):
        return ERROR("Strings must have equal length")
    
    distance = 0
    for i = 0 to len(s1) - 1:
        if s1[i] ≠ s2[i]:
            distance += 1
    
    return distance
```

### 3.3 Step-by-Step Example

**Strings:** "karolin" vs "kathrin"

| Position | s1 | s2 | Match? |
|----------|----|----|--------|
| 0 | k | k | ✓ |
| 1 | a | a | ✓ |
| 2 | r | t | ✗ |
| 3 | o | h | ✗ |
| 4 | l | r | ✗ |
| 5 | i | i | ✓ |
| 6 | n | n | ✓ |

**Hamming Distance:** 3

## 4. Complexity Analysis

### 4.1 Time Complexity

$$O(n)$$

Single pass through both strings.

### 4.2 Space Complexity

$$O(1)$$

Only a counter variable needed.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
#[derive(Debug, PartialEq)]
pub enum HammingDistanceError {
    InputStringsHaveDifferentLength,
}

pub fn hamming_distance(string_a: &str, string_b: &str) 
    -> Result<usize, HammingDistanceError> 
{
    if string_a.len() != string_b.len() {
        return Err(HammingDistanceError::InputStringsHaveDifferentLength);
    }

    let distance = string_a
        .chars()
        .zip(string_b.chars())
        .filter(|(a, b)| a != b)
        .count();

    Ok(distance)
}
```

**Implementation Features:**
- Uses Rust's Result type for error handling
- Functional style with iterator chains
- Unicode-safe with `.chars()` iterator
- Clean, idiomatic Rust

### 5.2 Binary String Optimization

For binary strings or byte arrays:

```rust
fn hamming_distance_bytes(a: &[u8], b: &[u8]) -> usize {
    a.iter()
        .zip(b.iter())
        .map(|(x, y)| (x ^ y).count_ones() as usize)
        .sum()
}

// Or for u64 values directly
fn hamming_distance_u64(a: u64, b: u64) -> u32 {
    (a ^ b).count_ones()
}
```

### 5.3 Edge Cases

| Input | Result |
|-------|--------|
| "", "" | Ok(0) |
| "a", "" | Err(DifferentLength) |
| "a", "a" | Ok(0) |
| "a", "b" | Ok(1) |
| Same strings | Ok(0) |
| No common chars | Ok(n) |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Error Detection & Correction:**
   - Network transmission verification
   - Memory error detection (ECC)
   - QR codes and barcodes

2. **Cryptography:**
   - Measuring key strength
   - Avalanche effect testing

3. **Bioinformatics:**
   - SNP detection in DNA
   - Sequence similarity for same-length sequences

4. **Data Deduplication:**
   - Near-duplicate detection for fixed-size hashes
   - Similarity hashing (SimHash, MinHash)

5. **Machine Learning:**
   - Feature comparison
   - Perceptual hashing

### 6.2 Common Use Cases

**Spell Checking (Fixed Length):**
```rust
fn suggest_corrections(word: &str, dictionary: &[&str]) -> Vec<&str> {
    dictionary
        .iter()
        .filter(|w| w.len() == word.len())
        .filter(|w| hamming_distance(word, w).unwrap() <= 1)
        .copied()
        .collect()
}
```

**Hash Similarity:**
```rust
fn similar_hashes(h1: u64, h2: u64, threshold: u32) -> bool {
    (h1 ^ h2).count_ones() <= threshold
}
```

## 7. Comparison with Other Metrics

| Metric | Equal Length Required | Operations | Time |
|--------|----------------------|------------|------|
| **Hamming** | Yes | Substitution only | O(n) |
| Levenshtein | No | Insert, Delete, Substitute | O(mn) |
| Damerau | No | + Transposition | O(mn) |
| Jaro-Winkler | No | Transposition-based | O(mn) |

## 8. Variations

### 8.1 Weighted Hamming Distance

```rust
fn weighted_hamming(s1: &str, s2: &str, weights: &[f64]) -> f64 {
    s1.chars()
        .zip(s2.chars())
        .zip(weights.iter())
        .filter(|((a, b), _)| a != b)
        .map(|(_, w)| w)
        .sum()
}
```

### 8.2 Generalized Hamming Distance

For alphabets with similarity scores:

```rust
fn generalized_hamming(s1: &str, s2: &str, 
                       similarity: impl Fn(char, char) -> f64) -> f64 {
    s1.chars()
        .zip(s2.chars())
        .map(|(a, b)| 1.0 - similarity(a, b))
        .sum()
}
```

## 9. References

1. Hamming, R. W. (1950). "Error Detecting and Error Correcting Codes". *Bell System Technical Journal*.
2. MacKay, D. J. C. "Information Theory, Inference, and Learning Algorithms", Chapter 1.
3. Peterson, W. W., & Weldon, E. J. "Error-Correcting Codes".

## Implementation

See: [src/string/hamming_distance.rs](../../src/string/hamming_distance.rs)
