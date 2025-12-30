# Suffix Array

## 1. Overview

A suffix array is a sorted array of all suffixes of a string. It provides a space-efficient alternative to suffix trees while supporting many of the same operations, including pattern matching, finding longest common substrings, and various string analysis tasks.

First introduced by Manber and Myers in 1990, suffix arrays have become fundamental in text processing, bioinformatics, and data compression.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a string $S$ of length $n$, construct an array $SA$ where $SA[i]$ is the starting index of the $i$-th lexicographically smallest suffix of $S$.

**Formal Definition:**

$$SA[i] = j \text{ such that } S[j..n-1] \text{ is the } i\text{-th smallest suffix}$$

### 2.2 Mathematical Model

**Suffixes of "banana":**
| Index | Suffix |
|-------|--------|
| 0 | banana |
| 1 | anana |
| 2 | nana |
| 3 | ana |
| 4 | na |
| 5 | a |

**Sorted suffixes:**
| Rank | Index | Suffix |
|------|-------|--------|
| 0 | 5 | a |
| 1 | 3 | ana |
| 2 | 1 | anana |
| 3 | 0 | banana |
| 4 | 4 | na |
| 5 | 2 | nana |

**Suffix Array:** $SA = [5, 3, 1, 0, 4, 2]$

### 2.3 Properties

1. **Uniqueness:** Each position appears exactly once in SA
2. **Lexicographic ordering:** $S[SA[i]..] < S[SA[i+1]..]$ for all $i < n-1$
3. **Pattern matching:** Binary search finds pattern in $O(m \log n)$

## 3. Algorithm Description

### 3.1 Intuition

The basic construction algorithm:
1. Generate all suffixes with their starting indices
2. Sort suffixes lexicographically
3. Extract starting indices to form the suffix array

The optimized approach uses **doubling technique**:
1. Initially, rank suffixes by first character
2. Iteratively double comparison length: 1 → 2 → 4 → 8 → ...
3. At step $k$, compare pairs $(rank[i], rank[i + 2^{k-1}])$

### 3.2 Pseudocode

**Naive Construction:**
```
function BUILD_SUFFIX_ARRAY_NAIVE(s):
    n = len(s)
    suffixes = [(i, s[i:]) for i in range(n)]
    suffixes.sort(by=lambda x: x[1])
    return [idx for (idx, _) in suffixes]
```

**Doubling Algorithm (O(n log²n)):**
```
function BUILD_SUFFIX_ARRAY(s):
    n = len(s)
    
    // Initialize with single character ranks
    suffixes = array of (index, (rank0, rank1))
    for i = 0 to n-1:
        suffixes[i].index = i
        suffixes[i].rank = (ord(s[i]) - ord('a'), 
                           ord(s[i+1]) - ord('a') if i+1 < n else -1)
    
    sort(suffixes by rank)
    
    // Double comparison length
    k = 4
    while k < 2n:
        // Assign new ranks
        rank = 0
        prev_rank = suffixes[0].rank
        suffixes[0].rank.0 = rank
        ind[suffixes[0].index] = 0
        
        for i = 1 to n-1:
            if suffixes[i].rank == prev_rank:
                // Same rank as previous
            else:
                rank += 1
            prev_rank = suffixes[i].rank
            suffixes[i].rank.0 = rank
            ind[suffixes[i].index] = i
        
        // Update second component
        for i = 0 to n-1:
            next = suffixes[i].index + k/2
            suffixes[i].rank.1 = suffixes[ind[next]].rank.0 if next < n else -1
        
        sort(suffixes by rank)
        k *= 2
    
    return [s.index for s in suffixes]
```

### 3.3 Step-by-Step Example

**String:** "banana"

**Initial (k=2):**
| i | Suffix | char[i] | char[i+1] | Rank |
|---|--------|---------|-----------|------|
| 0 | banana | b(1) | a(0) | (1,0) |
| 1 | anana | a(0) | n(13) | (0,13) |
| 2 | nana | n(13) | a(0) | (13,0) |
| 3 | ana | a(0) | n(13) | (0,13) |
| 4 | na | n(13) | a(0) | (13,0) |
| 5 | a | a(0) | -1 | (0,-1) |

**After sorting:**
| Rank | i | Suffix |
|------|---|--------|
| 0 | 5 | a |
| 1 | 1 | anana |
| 1 | 3 | ana |
| 2 | 0 | banana |
| 3 | 2 | nana |
| 3 | 4 | na |

**Continue doubling until all ranks unique...**

**Final SA:** [5, 3, 1, 0, 4, 2]

## 4. Complexity Analysis

### 4.1 Time Complexity

| Algorithm | Time | Space |
|-----------|------|-------|
| Naive (sort all suffixes) | $O(n^2 \log n)$ | $O(n^2)$ |
| Doubling technique | $O(n \log^2 n)$ | $O(n)$ |
| Manber-Myers (radix sort) | $O(n \log n)$ | $O(n)$ |
| DC3/Skew | $O(n)$ | $O(n)$ |
| SA-IS | $O(n)$ | $O(n)$ |

**Current Implementation:** $O(n \log^2 n)$ with comparison sort.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Suffix array | $O(n)$ |
| Rank arrays | $O(n)$ |
| Index array | $O(n)$ |
| **Total** | $O(n)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Clone)]
struct Suffix {
    index: usize,
    rank: (i32, i32),
}

impl Suffix {
    fn cmp(&self, b: &Self) -> Ordering {
        match self.rank.0.cmp(&b.rank.0) {
            Ordering::Equal => {
                if self.rank.1 < b.rank.1 { Ordering::Less }
                else { Ordering::Greater }
            }
            o => o
        }
    }
}
```

**Implementation Details:**
- Uses tuple comparison for two-key sorting
- `i32` for ranks to handle -1 sentinel
- Clone derives for sorting operations
- Custom comparison function

**Potential Optimizations:**
```rust
// Use radix sort instead of comparison sort
fn radix_sort_suffixes(suffixes: &mut [Suffix]) {
    // Sort by second rank, then first rank
    // O(n) instead of O(n log n)
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty string | `[]` |
| Single character | `[0]` |
| All same characters | `[n-1, n-2, ..., 0]` |
| Sorted string | `[0, 1, 2, ..., n-1]` |
| Reverse sorted | `[n-1, n-2, ..., 0]` |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Pattern Matching:**
   - Binary search in O(m log n)
   - Multiple patterns with single construction

2. **Data Compression:**
   - Burrows-Wheeler Transform uses SA
   - bzip2 compression algorithm

3. **Bioinformatics:**
   - Genome sequence indexing
   - DNA/protein alignment (Bowtie, BWA)

4. **Plagiarism Detection:**
   - Finding common substrings
   - Document similarity

5. **Full-Text Search:**
   - Database indexing
   - Search engine backends

### 6.2 Pattern Search Using Suffix Array

```rust
fn search_pattern(sa: &[usize], text: &str, pattern: &str) -> Option<usize> {
    let (mut lo, mut hi) = (0, sa.len());
    
    while lo < hi {
        let mid = (lo + hi) / 2;
        let suffix = &text[sa[mid]..];
        
        if suffix < pattern {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    
    if text[sa[lo]..].starts_with(pattern) {
        Some(sa[lo])
    } else {
        None
    }
}
```

### 6.3 Related Data Structures

| Structure | Pros | Cons |
|-----------|------|------|
| **Suffix Tree** | O(m) pattern search | O(n) space overhead |
| **Suffix Array** | Cache-friendly | O(m log n) search |
| **FM-Index** | Compressed | Complex implementation |
| **Enhanced SA + LCP** | O(m) search | Additional O(n) for LCP |

## 7. LCP Array Extension

The **Longest Common Prefix (LCP) array** enhances suffix arrays:

$$LCP[i] = \text{length of longest common prefix of } SA[i] \text{ and } SA[i-1]$$

**Example for "banana":**
| i | SA[i] | Suffix | LCP[i] |
|---|-------|--------|--------|
| 0 | 5 | a | 0 |
| 1 | 3 | ana | 1 (a) |
| 2 | 1 | anana | 3 (ana) |
| 3 | 0 | banana | 0 |
| 4 | 4 | na | 0 |
| 5 | 2 | nana | 2 (na) |

**Applications:**
- O(m + log n) pattern matching
- Finding longest repeated substring: max(LCP)
- Computing number of distinct substrings

## 8. References

1. Manber, U., & Myers, G. (1990). "Suffix Arrays: A New Method for On-line String Searches". *SIAM Journal on Computing*.
2. Kärkkäinen, J., & Sanders, P. (2003). "Simple Linear Work Suffix Array Construction". *ICALP*.
3. Gusfield, D. "Algorithms on Strings, Trees, and Sequences", Chapter 9.

## Implementation

See: [src/string/suffix_array.rs](../../src/string/suffix_array.rs)
