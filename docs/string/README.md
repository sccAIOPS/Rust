# String Algorithms

This module contains implementations of various string algorithms for pattern matching, text processing, distance computation, and string property analysis.

## Overview

String algorithms are fundamental in computer science for text processing, searching, and analysis. This collection includes:

- **Pattern Matching**: Efficient algorithms for finding substrings
- **Suffix Structures**: Data structures for suffix-based operations
- **Distance Metrics**: Algorithms for measuring string similarity
- **String Properties**: Utilities for analyzing string characteristics
- **Text Transforms**: Algorithms for encoding and transforming text

## Algorithms

### Pattern Matching Algorithms

| Algorithm | Time Complexity | Space Complexity | Description |
|-----------|-----------------|------------------|-------------|
| [Knuth-Morris-Pratt](knuth_morris_pratt.md) | O(n + m) | O(m) | Linear pattern matching with prefix table |
| [Boyer-Moore](boyer_moore_search.md) | O(n/m) best, O(nm) worst | O(k) | Fast pattern search using bad character rule |
| [Rabin-Karp](rabin_karp.md) | O(n + m) avg, O(nm) worst | O(1) | Rolling hash-based pattern matching |
| [Z Algorithm](z_algorithm.md) | O(n + m) | O(n + m) | Z-array based pattern matching |
| [Aho-Corasick](aho_corasick.md) | O(n + m + z) | O(m × Σ) | Multi-pattern matching automaton |

### Suffix Structures

| Algorithm | Time Complexity | Space Complexity | Description |
|-----------|-----------------|------------------|-------------|
| [Suffix Array](suffix_array.md) | O(n log²n) | O(n) | Sorted array of all suffixes |
| [Suffix Array (Manber-Myers)](suffix_array_manber_myers.md) | O(n log n) | O(n) | Optimized suffix array construction |
| [Suffix Tree](suffix_tree.md) | O(n²) | O(n²) | Compressed trie of all suffixes |
| [Manacher's Algorithm](manacher.md) | O(n) | O(n) | Longest palindromic substring |
| [Shortest Palindrome](shortest_palindrome.md) | O(n) | O(n) | Find shortest palindrome by prefix addition |

### Distance Metrics

| Algorithm | Time Complexity | Space Complexity | Description |
|-----------|-----------------|------------------|-------------|
| [Levenshtein Distance](levenshtein_distance.md) | O(nm) | O(n) or O(nm) | Edit distance (insert, delete, substitute) |
| [Hamming Distance](hamming_distance.md) | O(n) | O(1) | Count differing positions |
| [Jaro-Winkler Distance](jaro_winkler_distance.md) | O(n × m) | O(n + m) | Similarity metric for short strings |

### String Property Checks

| Algorithm | Time Complexity | Space Complexity | Description |
|-----------|-----------------|------------------|-------------|
| [Palindrome Check](palindrome.md) | O(n) | O(1) | Check if string is palindrome |
| [Anagram Check](anagram.md) | O(n) | O(k) | Check if strings are anagrams |
| [Isogram Check](isogram.md) | O(n) | O(k) | Check if all letters appear once |
| [Lipogram Check](lipogram.md) | O(n) | O(k) | Check for missing letters |
| [Pangram Check](pangram.md) | O(n) | O(k) | Check if all alphabet letters present |
| [Isomorphism Check](isomorphism.md) | O(n) | O(k) | Check if strings have same character mapping |

### Text Transforms & Utilities

| Algorithm | Time Complexity | Space Complexity | Description |
|-----------|-----------------|------------------|-------------|
| [String Reverse](reverse.md) | O(n) | O(n) | Reverse a string |
| [Run-Length Encoding](run_length_encoding.md) | O(n) | O(n) | Compress consecutive characters |
| [Burrows-Wheeler Transform](burrows_wheeler_transform.md) | O(n² log n) | O(n²) | Text transformation for compression |
| [Duval Algorithm](duval_algorithm.md) | O(n) | O(n) | Factorization into Lyndon words |
| [Autocomplete (Trie)](autocomplete_using_trie.md) | O(m) query | O(n × m) | Prefix-based word completion |

## When to Use Which Algorithm

### Pattern Matching Selection Guide

```
Single pattern, guaranteed O(n+m)?
├─ Yes → Knuth-Morris-Pratt
└─ No
   └─ Multiple patterns?
      ├─ Yes → Aho-Corasick
      └─ No
         └─ Need practical speed (average case)?
            ├─ Yes → Boyer-Moore (best for long patterns)
            └─ No → Rabin-Karp (good for multiple pattern lengths)
```

### String Similarity Selection Guide

```
Strings same length?
├─ Yes → Hamming Distance (simplest)
└─ No
   └─ Need edit operations count?
      ├─ Yes → Levenshtein Distance
      └─ No → Jaro-Winkler (for names/short strings)
```

## Complexity Summary

| Category | Best Algorithm | Use Case |
|----------|---------------|----------|
| Single Pattern Search | Boyer-Moore | Long patterns in text |
| Multiple Pattern Search | Aho-Corasick | Dictionary matching |
| Edit Distance | Levenshtein | Spell checking |
| Prefix Matching | Trie/Autocomplete | Search suggestions |
| Palindrome Finding | Manacher | Longest palindrome substring |

## Common Patterns in Rust Implementation

### Character Handling

```rust
// Convert to chars for Unicode support
let chars: Vec<char> = text.chars().collect();

// Case-insensitive comparison
c.to_ascii_lowercase()
```

### HashMaps for Character Counting

```rust
use std::collections::HashMap;

fn char_count(s: &str) -> HashMap<char, usize> {
    let mut counts = HashMap::new();
    for c in s.chars() {
        *counts.entry(c).or_insert(0) += 1;
    }
    counts
}
```

### Rolling Hash Pattern

```rust
fn update_hash(old_hash: usize, old_char: u8, new_char: u8, radix_pow: usize, modulo: usize) -> usize {
    let mut new_hash = old_hash;
    new_hash = (new_hash + modulo - (old_char as usize * radix_pow % modulo)) % modulo;
    new_hash = (new_hash * RADIX + new_char as usize) % modulo;
    new_hash
}
```

## References

- Cormen, T. H., et al. "Introduction to Algorithms" - Chapters on String Matching
- Gusfield, D. "Algorithms on Strings, Trees, and Sequences"
- Sedgewick, R. & Wayne, K. "Algorithms" - String Processing
