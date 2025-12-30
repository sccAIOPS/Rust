# Transposition Cipher

## 1. Overview

The **Transposition Cipher** rearranges the positions of characters in the plaintext without changing the characters themselves. Unlike substitution ciphers that replace characters, transposition ciphers permute them according to a defined pattern.

### Historical Context
- **Ancient times**: Spartan scytale (5th century BCE)
- **WWI**: Double transposition used by German military
- **WWII**: Combined with substitution in rotor machines
- **Present**: Educational; component of modern ciphers

### Variants

| Type | Description |
|------|-------------|
| Columnar | Write in rows, read in columns |
| Rail Fence | Zigzag pattern |
| Route | Follow specific path through grid |
| Grille | Use mask to select positions |

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- Plaintext $P = p_0, p_1, ..., p_{n-1}$
- Key (determines permutation)

**Output**: Ciphertext $C$ (same characters, different order)

### 2.2 Mathematical Model

A transposition cipher is a **permutation** $\pi$ of positions:
$$c_i = p_{\pi(i)}$$

For columnar transposition with key length $k$:
- Arrange text in $\lceil n/k \rceil$ rows of $k$ columns
- Read columns in order determined by key

## 3. Columnar Transposition

### 3.1 Algorithm Description

**Encryption**:
1. Write plaintext in rows of key length
2. Number columns based on alphabetical order of key
3. Read columns in numerical order

**Decryption**:
1. Calculate number of rows needed
2. Fill columns in key order
3. Read row by row

### 3.2 Pseudocode

```
FUNCTION columnar_encrypt(plaintext, key):
    // Remove spaces, convert to uppercase
    text ← normalize(plaintext)
    key_order ← get_column_order(key)
    cols ← len(key)
    rows ← ceil(len(text) / cols)
    
    // Pad if necessary
    text ← text + 'X' * (rows * cols - len(text))
    
    // Create grid
    grid ← empty 2D array [rows][cols]
    FOR i FROM 0 TO len(text) - 1:
        grid[i / cols][i MOD cols] ← text[i]
    
    // Read columns in key order
    ciphertext ← ""
    FOR col_num FROM 1 TO cols:
        col_index ← position of col_num in key_order
        FOR row FROM 0 TO rows - 1:
            ciphertext ← ciphertext + grid[row][col_index]
    
    RETURN ciphertext

FUNCTION get_column_order(key):
    // Assign numbers based on alphabetical order
    sorted_key ← sort(enumerate(key), by character)
    order ← array of len(key)
    FOR i, (original_pos, char) in enumerate(sorted_key):
        order[original_pos] ← i + 1
    RETURN order
```

### 3.3 Step-by-Step Example

**Plaintext**: "ATTACK AT DAWN" → "ATTACKATDAWN"
**Key**: "ZEBRA"

**Step 1**: Determine column order from key
```
Key:    Z  E  B  R  A
Order:  5  2  1  4  3
```

**Step 2**: Write plaintext in grid
```
Column: 5  2  1  4  3
        A  T  T  A  C
        K  A  T  D  A
        W  N  X  X  X  (padded with X)
```

**Step 3**: Read columns in order (1, 2, 3, 4, 5)
```
Column 1 (B): T T X
Column 2 (E): T A N
Column 3 (A): C A X
Column 4 (R): A D X
Column 5 (Z): A K W
```

**Ciphertext**: "TTXTANCAXADXAKW"

### 3.4 Decryption Example

**Ciphertext**: "TTXTANCAXADXAKW"
**Key**: "ZEBRA" (order: 5, 2, 1, 4, 3)

**Step 1**: Calculate dimensions
- Key length: 5
- Ciphertext length: 15
- Rows: 15 / 5 = 3

**Step 2**: Fill columns in key order
```
Column 1: T T X
Column 2: T A N
Column 3: C A X
Column 4: A D X
Column 5: A K W
```

**Step 3**: Read row by row
```
Row 1: A(col5) T(col2) T(col1) A(col4) C(col3) = ATTAC
Row 2: K(col5) A(col2) T(col1) D(col4) A(col3) = KATDA
Row 3: W(col5) N(col2) X(col1) X(col4) X(col3) = WNXXX
```

**Plaintext**: "ATTACKATDAWNXXX" → "ATTACK AT DAWN"

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Key order calculation | O(k log k) |
| Grid construction | O(n) |
| Column reading | O(n) |
| **Total** | **O(n + k log k)** |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Grid | O(n) |
| Key order | O(k) |
| **Total** | **O(n)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn transposition_encrypt(plaintext: &str, key: &str) -> String {
    let text: Vec<char> = plaintext
        .chars()
        .filter(|c| c.is_ascii_alphabetic())
        .map(|c| c.to_ascii_uppercase())
        .collect();
    
    if text.is_empty() || key.is_empty() {
        return String::new();
    }
    
    let key_order = get_key_order(key);
    let cols = key.len();
    let rows = (text.len() + cols - 1) / cols;
    
    // Pad text
    let mut padded = text;
    while padded.len() < rows * cols {
        padded.push('X');
    }
    
    // Build ciphertext by reading columns
    let mut ciphertext = String::new();
    for target_order in 1..=cols {
        let col = key_order.iter().position(|&o| o == target_order).unwrap();
        for row in 0..rows {
            ciphertext.push(padded[row * cols + col]);
        }
    }
    
    ciphertext
}

fn get_key_order(key: &str) -> Vec<usize> {
    let mut indexed: Vec<_> = key.chars().enumerate().collect();
    indexed.sort_by_key(|&(_, c)| c);
    
    let mut order = vec![0; key.len()];
    for (rank, (original_pos, _)) in indexed.into_iter().enumerate() {
        order[original_pos] = rank + 1;
    }
    order
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty plaintext | Return empty string |
| Empty key | Return empty string |
| Key with duplicates | Use stable sort (first occurrence wins) |
| Plaintext < key length | Pad with X |
| Non-alphabetic chars | Remove or preserve (implementation choice) |

## 6. Security Analysis

### 6.1 Key Space

For key length $k$ with unique letters:
- Number of permutations: $k!$
- Example: 10-letter key → 3,628,800 permutations

### 6.2 Cryptanalysis

| Attack | Method |
|--------|--------|
| Anagramming | Try rearranging columns |
| Frequency analysis | Letter frequencies unchanged |
| Probable word | Find likely words, deduce structure |
| Multiple messages | Compare patterns |

### 6.3 Weaknesses

1. **Frequency preserved**: Same letter distribution
2. **Digraph analysis**: Common pairs (TH, HE) may appear
3. **Pattern recognition**: Repeated key structure
4. **Known plaintext**: Devastating if any text known

## 7. Double Transposition

### 7.1 Description

Apply transposition twice with different keys:
1. First transposition with key₁
2. Second transposition with key₂

### 7.2 Security Improvement

- Breaks simple anagramming attacks
- Used in WWII by German military
- Still vulnerable to sophisticated cryptanalysis

## 8. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Rail Fence | Simple transposition pattern |
| Route Cipher | Read grid in specific path |
| Scytale | Ancient Greek transposition |
| Grille | Use template for positions |

## 9. Modern Relevance

### 9.1 In Modern Cryptography

Transposition principles appear in:
- **AES ShiftRows**: Row-wise byte permutation
- **DES P-boxes**: Bit permutation
- **Block cipher modes**: Data arrangement

### 9.2 Combined with Substitution

Modern ciphers combine:
- **Confusion** (substitution): Obscure key-ciphertext relationship
- **Diffusion** (transposition): Spread plaintext influence

This is Shannon's principle of product ciphers.

## 10. Educational Value

### 10.1 Concepts Demonstrated

| Concept | Transposition Example |
|---------|----------------------|
| Permutation | Character rearrangement |
| Diffusion | Spreading information |
| Key-based encryption | Different keys → different order |
| Padding | Handling variable-length input |

### 10.2 Common Implementation Mistakes

| Mistake | Solution |
|---------|----------|
| Off-by-one in grid indexing | Careful bounds checking |
| Wrong column order | Test with known examples |
| Forgetting padding | Always check text length |
| Key duplicate handling | Define consistent behavior |

## 11. References

- Kahn, D. (1996). The Codebreakers
- Singh, S. (1999). The Code Book
- Schneier, B. (1996). Applied Cryptography
- [Implementation](../../src/ciphers/transposition.rs)
