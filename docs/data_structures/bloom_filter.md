# Bloom Filter

## 1. Overview

A Bloom Filter is a space-efficient probabilistic data structure for set membership testing. It can tell you "definitely not in set" or "probably in set" but never gives false negatives. It trades perfect accuracy for dramatic space savings, using only a few bits per element.

Invented by Burton Howard Bloom in 1970.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a set $S$ of $n$ elements, support:
- **Insert(x)**: Add $x$ to the set
- **Query(x)**: Check if $x$ is in the set

**Guarantees**:
- If $x \in S$: Always returns true (no false negatives)
- If $x \notin S$: May return true (false positive) with probability $p$

### 2.2 Mathematical Model

**Structure**:
- Bit array of $m$ bits, initially all 0
- $k$ independent hash functions $h_1, h_2, ..., h_k$, each mapping to $[0, m-1]$

**Insert(x)**: Set bits $h_1(x), h_2(x), ..., h_k(x)$ to 1

**Query(x)**: Return true iff all bits $h_1(x), h_2(x), ..., h_k(x)$ are 1

### 2.3 False Positive Probability

After inserting $n$ elements:

**Probability a specific bit is 0**:
$$\left(1 - \frac{1}{m}\right)^{kn} \approx e^{-kn/m}$$

**False positive probability**:
$$p = \left(1 - e^{-kn/m}\right)^k$$

### 2.4 Optimal Parameters

For target false positive rate $p$ and $n$ elements:

**Optimal number of bits**:
$$m = -\frac{n \ln p}{(\ln 2)^2} \approx -1.44 \cdot n \cdot \log_2 p$$

**Optimal number of hash functions**:
$$k = \frac{m}{n} \ln 2 \approx 0.693 \cdot \frac{m}{n}$$

**Space per element**: 
For 1% false positive: ~9.6 bits/element
For 0.1% false positive: ~14.4 bits/element

## 3. Algorithm Description

### 3.1 Intuition

Imagine a board with $m$ light switches. To "add" an item, you flip $k$ specific switches determined by hashing. To check if an item was added, you check if all $k$ switches are on. If any is off, definitely not added. If all are on, probably added (but maybe other items flipped those switches).

### 3.2 Structure Visualization

```
Bloom Filter (m=10, k=3):
Initial: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

Insert "apple" (hashes to positions 1, 4, 7):
        [0, 1, 0, 0, 1, 0, 0, 1, 0, 0]

Insert "banana" (hashes to positions 2, 4, 9):
        [0, 1, 1, 0, 1, 0, 0, 1, 0, 1]

Query "apple" (1, 4, 7): all 1 → probably in set ✓
Query "cherry" (3, 5, 8): bit 3=0 → definitely NOT in set ✓
Query "grape" (1, 2, 4): all 1 → FALSE POSITIVE! (not actually added)
```

### 3.3 Pseudocode

```
CREATE(m, k):
    bits = array of m bits, all 0
    hash_functions = generate k hash functions
    return BloomFilter(bits, hash_functions)

INSERT(filter, element):
    for i from 1 to k:
        index = hash_i(element) mod m
        filter.bits[index] = 1

QUERY(filter, element):
    for i from 1 to k:
        index = hash_i(element) mod m
        if filter.bits[index] == 0:
            return false  // Definitely not in set
    return true  // Probably in set
```

### 3.4 Generating Multiple Hash Functions

**Double Hashing Technique** (Kirsch-Mitzenmacher):
$$h_i(x) = h_1(x) + i \cdot h_2(x) \mod m$$

Only need two base hash functions to generate $k$ hashes.

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time |
|-----------|------|
| Insert | O(k) |
| Query | O(k) |

Where $k$ is typically $O(1)$ (small constant, e.g., 7-10).

### 4.2 Space Complexity

- **Fixed**: O(m) bits
- **Per Element**: O(1) - no storage of elements themselves!
- **Typical**: 8-15 bits per element for low false positive rates

### 4.3 Comparison

| Structure | Space | False Positives | False Negatives |
|-----------|-------|-----------------|-----------------|
| Hash Set | O(n × element_size) | No | No |
| Bloom Filter | O(n) bits | Yes | No |
| Cuckoo Filter | O(n) bits | Yes | Possible with delete |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

pub struct BloomFilter {
    bits: Vec<bool>,
    num_hashes: usize,
    num_bits: usize,
}

impl BloomFilter {
    pub fn new(expected_elements: usize, false_positive_rate: f64) -> Self {
        // Calculate optimal parameters
        let num_bits = Self::optimal_num_bits(expected_elements, false_positive_rate);
        let num_hashes = Self::optimal_num_hashes(num_bits, expected_elements);
        
        BloomFilter {
            bits: vec![false; num_bits],
            num_hashes,
            num_bits,
        }
    }
    
    fn optimal_num_bits(n: usize, p: f64) -> usize {
        let m = -(n as f64 * p.ln()) / (2.0_f64.ln().powi(2));
        m.ceil() as usize
    }
    
    fn optimal_num_hashes(m: usize, n: usize) -> usize {
        let k = (m as f64 / n as f64) * 2.0_f64.ln();
        k.ceil() as usize
    }
    
    fn hash_indices<T: Hash>(&self, item: &T) -> Vec<usize> {
        let mut hasher1 = DefaultHasher::new();
        let mut hasher2 = DefaultHasher::new();
        
        item.hash(&mut hasher1);
        let h1 = hasher1.finish();
        
        // Use different seed for second hash
        hasher2.write_u64(h1);
        item.hash(&mut hasher2);
        let h2 = hasher2.finish();
        
        (0..self.num_hashes)
            .map(|i| {
                (h1.wrapping_add((i as u64).wrapping_mul(h2)) % self.num_bits as u64) as usize
            })
            .collect()
    }
    
    pub fn insert<T: Hash>(&mut self, item: &T) {
        for index in self.hash_indices(item) {
            self.bits[index] = true;
        }
    }
    
    pub fn contains<T: Hash>(&self, item: &T) -> bool {
        self.hash_indices(item).iter().all(|&i| self.bits[i])
    }
}
```

**Key Design Patterns**:
- Automatic parameter calculation
- Double hashing for multiple hash functions
- `Hash` trait for generic types
- `Vec<bool>` (or use `bitvec` crate for efficiency)

### 5.2 Using Bit Vector

```rust
// More memory-efficient using bitvec crate
use bitvec::prelude::*;

pub struct EfficientBloomFilter {
    bits: BitVec,
    num_hashes: usize,
}

impl EfficientBloomFilter {
    pub fn insert<T: Hash>(&mut self, item: &T) {
        for index in self.hash_indices(item) {
            self.bits.set(index, true);
        }
    }
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Zero elements | Creates minimal filter |
| All bits set | Every query returns true |
| Very low FP rate | Many bits and hashes needed |
| Empty filter | All queries return false |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Databases**: Avoid disk lookups for non-existent keys
2. **Web Browsers**: Safe browsing (malicious URL detection)
3. **Caches**: Check before expensive cache lookup
4. **Spell Checkers**: Quick dictionary membership
5. **Network Routers**: Packet filtering
6. **Cryptocurrency**: Bitcoin SPV clients

### 6.2 Case Study: Google Bigtable

Bloom filters used to avoid unnecessary disk reads:
- Each tablet has a Bloom filter
- Query filter before reading from disk
- Saves millions of disk seeks

### 6.3 Variants

| Variant | Description |
|---------|-------------|
| **Counting Bloom Filter** | Support deletion using counters |
| **Scalable Bloom Filter** | Grows dynamically |
| **Cuckoo Filter** | Supports deletion, often better |
| **Quotient Filter** | Cache-friendly, supports merging |
| **Compressed Bloom Filter** | For transmission |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Undersizing**: Too few bits = high false positive rate
2. **No Deletion**: Standard Bloom filters don't support deletion
3. **Parameter Calculation**: Must know expected n in advance
4. **Hash Independence**: Poor hash functions increase FP rate

### 7.2 Counting Bloom Filter

Replace bits with counters to support deletion:
```rust
struct CountingBloomFilter {
    counts: Vec<u8>,  // 4-bit counters common
    // ...
}

fn delete<T: Hash>(&mut self, item: &T) {
    for index in self.hash_indices(item) {
        if self.counts[index] > 0 {
            self.counts[index] -= 1;
        }
    }
}
```

**Caveat**: Counter overflow possible, uses more space

### 7.3 When NOT to Use Bloom Filters

- Need exact membership (no false positives allowed)
- Need to enumerate elements
- Need deletion (use Cuckoo filter instead)
- Small sets (hash set is simpler)
- Need to retrieve values (use hash map)

### 7.4 Union and Intersection

```rust
// Union: OR the bit arrays
fn union(a: &BloomFilter, b: &BloomFilter) -> BloomFilter {
    // bits = a.bits | b.bits
}

// Intersection estimate (not exact)
// Can estimate |A ∩ B| from fill rates
```

## 8. References

- Bloom, B. H. (1970). "Space/Time Trade-offs in Hash Coding with Allowable Errors". *Communications of the ACM*.
- Broder, A., & Mitzenmacher, M. (2004). "Network Applications of Bloom Filters: A Survey". *Internet Mathematics*.
- Kirsch, A., & Mitzenmacher, M. (2006). "Less Hashing, Same Performance: Building a Better Bloom Filter". *ESA 2006*.
- Fan, B., et al. (2014). "Cuckoo Filter: Practically Better Than Bloom". *CoNEXT 2014*.
