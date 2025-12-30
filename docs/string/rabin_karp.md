# Rabin-Karp Algorithm

## 1. Overview

The Rabin-Karp algorithm, developed by Richard Karp and Michael Rabin in 1987, is a string searching algorithm that uses hashing to find patterns in text. It computes a hash value for the pattern and for each substring of the text, comparing hashes instead of character-by-character.

The algorithm is particularly useful for multiple pattern matching and plagiarism detection due to its rolling hash technique that allows efficient hash updates.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- A text string $T$ of length $n$
- A pattern string $P$ of length $m$

Find: All starting positions where $P$ occurs in $T$.

### 2.2 Mathematical Model

**Polynomial Rolling Hash:**

For a string $S = s_0 s_1 ... s_{m-1}$, define its hash as:
$$H(S) = \left(\sum_{i=0}^{m-1} s_i \cdot d^{m-1-i}\right) \mod q$$

Where:
- $d$ = radix (typically 256 for ASCII)
- $q$ = prime modulus
- $s_i$ = ASCII/Unicode value of character $i$

**Rolling Hash Update:**

Given $H(T[i..i+m-1])$, compute $H(T[i+1..i+m])$:
$$H_{new} = \left((H_{old} - s_i \cdot d^{m-1}) \cdot d + s_{i+m}\right) \mod q$$

This allows O(1) hash updates instead of O(m) recomputation.

### 2.3 Correctness Analysis

**Hash Collision Handling:**

If $H(P) = H(T[i..i+m-1])$:
- **True positive:** Pattern actually matches
- **False positive (spurious hit):** Hash collision, verify with character comparison

**Expected Performance:**

The probability of a false positive is approximately $1/q$ where $q$ is the modulus. With a good prime $q$, expected number of spurious hits is $O(n/q)$.

## 3. Algorithm Description

### 3.1 Intuition

1. Compute hash of the pattern
2. Compute hash of the first m characters of text
3. Slide the window:
   - If hashes match, verify character-by-character
   - Update hash using rolling hash formula
4. Record all verified matches

The rolling hash is like maintaining a "fingerprint" of each window that can be efficiently updated.

### 3.2 Pseudocode

```
CONSTANTS:
    RADIX = 256    // Base for polynomial hash
    MOD = 101      // Prime modulus

function RABIN_KARP(text, pattern):
    if text is empty or pattern is empty or len(pattern) > len(text):
        return []
    
    m = len(pattern)
    n = len(text)
    
    pat_hash = COMPUTE_HASH(pattern)
    text_hash = COMPUTE_HASH(text[0..m])
    
    // Compute RADIX^(m-1) mod MOD
    radix_pow = 1
    for i = 1 to m - 1:
        radix_pow = (radix_pow * RADIX) mod MOD
    
    results = []
    
    for i = 0 to n - m:
        // Compare hashes
        if text_hash == pat_hash:
            // Verify to handle collisions
            if text[i..i+m] == pattern:
                results.append(i)
        
        // Update rolling hash for next window
        if i < n - m:
            text_hash = UPDATE_HASH(text, i, i + m, text_hash, radix_pow)
    
    return results

function COMPUTE_HASH(s):
    hash_val = 0
    for char in s:
        hash_val = (hash_val * RADIX + ord(char)) mod MOD
    return hash_val

function UPDATE_HASH(s, old_idx, new_idx, old_hash, radix_pow):
    new_hash = old_hash
    old_char = s[old_idx]
    new_char = s[new_idx]
    
    // Remove old character contribution
    new_hash = (new_hash + MOD - (ord(old_char) * radix_pow) mod MOD) mod MOD
    // Add new character
    new_hash = (new_hash * RADIX + ord(new_char)) mod MOD
    
    return new_hash
```

### 3.3 Step-by-Step Example

**Text:** "ABABCAB"  
**Pattern:** "ABC"  
**RADIX = 256, MOD = 101**

**Step 1: Compute pattern hash**
```
H("ABC") = (65×256² + 66×256 + 67) mod 101
         = (4259840 + 16896 + 67) mod 101
         = 4276803 mod 101
         = 8
```

**Step 2: Compute initial text hash**
```
H("ABA") = (65×256² + 66×256 + 65) mod 101
         = 4276801 mod 101
         = 6
```

**Step 3: Search with rolling hash**

| Position | Window | Hash | Match? |
|----------|--------|------|--------|
| 0 | ABA | 6 | ✗ (6 ≠ 8) |
| 1 | BAB | Update... | ✗ |
| 2 | ABC | 8 | ✓ Hash match! Verify: "ABC" = "ABC" ✓ |
| 3 | BCA | Update... | ✗ |
| 4 | CAB | Update... | ✗ |

**Rolling Hash Update Example (position 0 → 1):**
```
H_new = ((H_old - ord('A')×256²) × 256 + ord('B')) mod 101
      = ((6 - 65×65536 mod 101) × 256 + 66) mod 101
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Condition |
|------|------------|-----------|
| **Best** | $O(n + m)$ | Few/no spurious hits |
| **Average** | $O(n + m)$ | Good hash function |
| **Worst** | $O(nm)$ | Many spurious hits (bad hash) |

**Analysis:**
- Hash computation: $O(m)$
- Rolling updates: $O(n - m)$ updates, each $O(1)$
- Spurious hit verification: Expected $O(n/q)$ hits, each $O(m)$

With prime $q$, expected time is $O(n + m + nm/q) = O(n + m)$ for large $q$.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Hash values | $O(1)$ |
| radix_pow | $O(1)$ |
| Results | $O(k)$ where k = number of matches |
| **Total** | $O(k)$ |

**Note:** Unlike KMP/Boyer-Moore, no preprocessing table needed!

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
const MOD: usize = 101;
const RADIX: usize = 256;

fn compute_hash(s: &str) -> usize {
    let mut hash_val = 0;
    for &byte in s.as_bytes().iter() {
        hash_val = (hash_val * RADIX + byte as usize) % MOD;
    }
    hash_val
}
```

**Implementation Notes:**
- Uses `as_bytes()` for efficient byte access (ASCII assumption)
- Small MOD (101) may cause more collisions in practice
- Character-by-character verification handles collisions

**Suggested Improvements:**
```rust
// Larger prime for fewer collisions
const MOD: usize = 1_000_000_007;

// Or use u64 for larger values
fn compute_hash_u64(s: &str) -> u64 {
    s.bytes().fold(0u64, |hash, byte| {
        (hash.wrapping_mul(RADIX as u64).wrapping_add(byte as u64)) % MOD
    })
}
```

### 5.2 Edge Cases

| Case | Handling | Result |
|------|----------|--------|
| Empty text | Early return | `[]` |
| Empty pattern | Early return | `[]` |
| Pattern longer than text | Early return | `[]` |
| Hash collision | Character verification | Correct match |
| All same characters | Many hash matches, all verified | Works correctly |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Plagiarism Detection:**
   - Rolling hash enables efficient document comparison
   - MOSS (Measure of Software Similarity)

2. **Version Control (Git):**
   - Content-addressable storage
   - Detecting moved/renamed content

3. **Data Deduplication:**
   - File system deduplication
   - Backup systems

4. **Network Security:**
   - Virus signature detection
   - Pattern matching in packet inspection

5. **Bioinformatics:**
   - Genome sequence matching
   - Finding k-mers in DNA

### 6.2 Multi-Pattern Extension

Rabin-Karp excels at searching for multiple patterns:

```
function MULTI_PATTERN_SEARCH(text, patterns):
    pattern_hashes = {COMPUTE_HASH(p): p for p in patterns}
    
    for each window in text:
        h = window_hash
        if h in pattern_hashes:
            if window == pattern_hashes[h]:
                report match
```

Time: $O(n \cdot m + k)$ where $k$ = number of patterns (vs. $O(n \cdot m \cdot k)$ for naive)

### 6.3 Related Algorithms

| Algorithm | Use Case |
|-----------|----------|
| **KMP** | Single pattern, guaranteed linear |
| **Boyer-Moore** | Single long pattern, practical speed |
| **Aho-Corasick** | Multiple patterns with automaton |
| **Rolling Hash** | Substring matching, duplicate detection |

## 7. Hash Function Considerations

### 7.1 Choosing MOD

| MOD Value | Pros | Cons |
|-----------|------|------|
| Small prime (101) | Fast computation | More collisions |
| Large prime (10⁹+7) | Fewer collisions | Slower modulo |
| Power of 2 | Fast (bitwise) | Poor distribution |

### 7.2 Multiple Hashes

For critical applications, use multiple hash functions:

```rust
fn double_hash(s: &str) -> (usize, usize) {
    let h1 = compute_hash_with_mod(s, 1_000_000_007);
    let h2 = compute_hash_with_mod(s, 1_000_000_009);
    (h1, h2)
}
```

Collision probability: $\approx 1/(q_1 \cdot q_2)$

## 8. References

1. Karp, R. M., & Rabin, M. O. (1987). "Efficient Randomized Pattern-Matching Algorithms". *IBM Journal of Research and Development*, 31(2), 249-260.
2. Cormen, T. H., et al. "Introduction to Algorithms" (3rd ed.), Section 32.2.
3. Sedgewick, R. "Algorithms in C", Chapter 19.

## Implementation

See: [src/string/rabin_karp.rs](../../src/string/rabin_karp.rs)
