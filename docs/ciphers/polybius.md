# Polybius Square

## 1. Overview

The **Polybius Square** is a substitution cipher that converts each letter into a pair of numbers representing its position in a 5×5 grid. Named after the Greek historian Polybius who described it around 200 BCE, it was used for long-distance communication using torches.

### Historical Context
- **~200 BCE**: Polybius describes the torch signaling system
- **15th century**: Used in diplomatic ciphers
- **WWII**: Modified versions used in various ciphers (ADFGX/ADFGVX)
- **Present**: Educational tool, basis for more complex ciphers

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: Plaintext with 25 distinct letters (I/J combined)
**Output**: Numeric pairs (row, column)

### 2.2 The Grid

Standard 5×5 Polybius Square:

```
    1   2   3   4   5
  ┌───┬───┬───┬───┬───┐
1 │ A │ B │ C │ D │ E │
  ├───┼───┼───┼───┼───┤
2 │ F │ G │ H │ I/J│ K │
  ├───┼───┼───┼───┼───┤
3 │ L │ M │ N │ O │ P │
  ├───┼───┼───┼───┼───┤
4 │ Q │ R │ S │ T │ U │
  ├───┼───┼───┼───┼───┤
5 │ V │ W │ X │ Y │ Z │
  └───┴───┴───┴───┴───┘
```

### 2.3 Encoding Function

For letter at row $r$, column $c$:
$$encode(letter) = (r, c)$$

**Example**: H is at row 2, column 3 → "23"

### 2.4 Alternative: Keyed Square

Using keyword "ZEBRA":
```
    1   2   3   4   5
  ┌───┬───┬───┬───┬───┐
1 │ Z │ E │ B │ R │ A │  ← Key letters first
  ├───┼───┼───┼───┼───┤
2 │ C │ D │ F │ G │ H │  ← Remaining alphabet
  ├───┼───┼───┼───┼───┤
3 │ I/J│ K │ L │ M │ N │
  ├───┼───┼───┼───┼───┤
4 │ O │ P │ Q │ S │ T │
  ├───┼───┼───┼───┼───┤
5 │ U │ V │ W │ X │ Y │
  └───┴───┴───┴───┴───┘
```

## 3. Algorithm Description

### 3.1 Encoding Pseudocode

```
FUNCTION polybius_encode(plaintext, grid):
    ciphertext ← ""
    
    FOR each character c in plaintext:
        IF c is a letter:
            c ← uppercase(c)
            IF c = 'J':
                c ← 'I'  // I and J share position
            
            row, col ← find_position(c, grid)
            ciphertext ← ciphertext + str(row) + str(col)
        ELSE:
            // Handle non-letters (implementation choice)
            ciphertext ← ciphertext + c
    
    RETURN ciphertext

FUNCTION polybius_decode(ciphertext, grid):
    plaintext ← ""
    
    FOR i FROM 0 TO len(ciphertext) - 1 STEP 2:
        row ← int(ciphertext[i])
        col ← int(ciphertext[i + 1])
        plaintext ← plaintext + grid[row][col]
    
    RETURN plaintext
```

### 3.2 Step-by-Step Example

**Plaintext**: "HELLO"
**Grid**: Standard Polybius Square

| Letter | Row | Column | Encoding |
|--------|-----|--------|----------|
| H | 2 | 3 | 23 |
| E | 1 | 5 | 15 |
| L | 3 | 1 | 31 |
| L | 3 | 1 | 31 |
| O | 3 | 4 | 34 |

**Ciphertext**: "2315313134"

### 3.3 Decoding Example

**Ciphertext**: "2315313134"

| Pair | Row | Column | Letter |
|------|-----|--------|--------|
| 23 | 2 | 3 | H |
| 15 | 1 | 5 | E |
| 31 | 3 | 1 | L |
| 31 | 3 | 1 | L |
| 34 | 3 | 4 | O |

**Plaintext**: "HELLO"

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Grid lookup | O(1) with hash map |
| Encoding | **O(n)** |
| Decoding | **O(n)** |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Grid | O(25) = O(1) |
| Output | O(2n) for encoding |
| **Total** | **O(n)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
const STANDARD_GRID: [[char; 5]; 5] = [
    ['A', 'B', 'C', 'D', 'E'],
    ['F', 'G', 'H', 'I', 'K'],
    ['L', 'M', 'N', 'O', 'P'],
    ['Q', 'R', 'S', 'T', 'U'],
    ['V', 'W', 'X', 'Y', 'Z'],
];

pub fn polybius_encode(plaintext: &str) -> String {
    plaintext
        .chars()
        .filter_map(|c| {
            let c = c.to_ascii_uppercase();
            let c = if c == 'J' { 'I' } else { c };
            
            for (row, row_chars) in STANDARD_GRID.iter().enumerate() {
                if let Some(col) = row_chars.iter().position(|&x| x == c) {
                    return Some(format!("{}{}", row + 1, col + 1));
                }
            }
            None
        })
        .collect()
}

pub fn polybius_decode(ciphertext: &str) -> String {
    ciphertext
        .chars()
        .collect::<Vec<_>>()
        .chunks(2)
        .filter_map(|chunk| {
            if chunk.len() == 2 {
                let row = chunk[0].to_digit(10)? as usize - 1;
                let col = chunk[1].to_digit(10)? as usize - 1;
                Some(STANDARD_GRID.get(row)?.get(col)?)
            } else {
                None
            }
        })
        .collect()
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Letter 'J' | Convert to 'I' |
| Non-letters | Skip or preserve |
| Lowercase | Convert to uppercase |
| Invalid digit pairs | Skip or error |
| Odd-length ciphertext | Invalid input |

## 6. Security Analysis

### 6.1 Key Space

| Configuration | Keys |
|---------------|------|
| Standard grid | 1 |
| Keyed grid | 25!/25 ≈ 1.5×10²⁵ |
| With key phrase | Depends on phrase |

### 6.2 Cryptanalysis

| Attack | Effectiveness |
|--------|---------------|
| Frequency analysis | Effective (pairs correspond to letters) |
| Pattern analysis | Number patterns reveal structure |
| Known plaintext | Instantly breaks standard grid |

### 6.3 Weaknesses

1. **Expansion**: 2:1 expansion ratio
2. **Pattern preservation**: Same letter → same pair
3. **Limited alphabet**: Only 25 letters
4. **Standard grid**: Trivially reversed if known

## 7. Variants and Extensions

### 7.1 ADFGX Cipher (WWI)

Replace digits with letters A, D, F, G, X:
```
    A   D   F   G   X
A │ b │ t │ a │ l │ p │
D │ d │ h │ o │ z │ k │
F │ q │ f │ v │ s │ n │
G │ g │ i │ c │ u │ x │
X │ m │ r │ e │ w │ y │
```

Then apply columnar transposition.

### 7.2 ADFGVX Cipher

6×6 grid including digits:
- Letters: A-Z (I and J separate)
- Digits: 0-9
- Uses ADFGVX as coordinates

### 7.3 Bifid Cipher

1. Encode with Polybius
2. Write row digits, then column digits
3. Re-pair and decode

**Example**: "GO"
- G = 22, O = 34
- Rows: 2, 3 | Cols: 2, 4
- Recombine: 23, 24 = H, I
- Ciphertext: "HI"

### 7.4 Trifid Cipher

3D version with 27 positions (+ space).

## 8. Historical Signaling System

### 8.1 Torch Method (Polybius)

```
Left torches: Row number (1-5)
Right torches: Column number (1-5)

Example: Letter "H" (row 2, col 3)
- Raise 2 torches on left
- Raise 3 torches on right
```

### 8.2 Advantages

| Property | Benefit |
|----------|---------|
| Simple signals | Easy to transmit |
| Limited vocabulary | Only 10 states (1-5) per side |
| Error checking | Visual confirmation |

## 9. Modern Applications

### 9.1 Use Cases

| Application | Purpose |
|-------------|---------|
| Education | Teaching substitution ciphers |
| Puzzles | Escape rooms, CTF |
| Steganography | Hiding in numeric data |
| Building blocks | Component of complex ciphers |

### 9.2 As Building Block

Polybius encoding is used in:
- Nihilist cipher
- VIC cipher (Soviet)
- Various fractionating ciphers

## 10. Educational Value

### 10.1 Concepts Demonstrated

| Concept | Example |
|---------|---------|
| Substitution cipher | Letter → number pair |
| 2D coordinate systems | (row, col) addressing |
| Fractionation | Breaking letters into components |
| Keyed permutation | Custom grid ordering |

### 10.2 Programming Concepts

- 2D array indexing
- Character encoding
- Hash map alternatives
- Modular arithmetic (for extensions)

## 11. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Nihilist cipher | Polybius + modular addition |
| Bifid cipher | Polybius + recombination |
| ADFGX/ADFGVX | Polybius + transposition |
| Tap code | Polybius variant for knocking |

## 12. Tap Code Variant

Prison "knock code":
```
    1    2    3    4    5
1 │ A  │ B  │ C/K│ D  │ E  │
2 │ F  │ G  │ H  │ I  │ J  │
3 │ L  │ M  │ N  │ O  │ P  │
4 │ Q  │ R  │ S  │ T  │ U  │
5 │ V  │ W  │ X  │ Y  │ Z  │

"H" = knock-knock, knock-knock-knock (2,3)
```

## 13. References

- Polybius. The Histories, Book X
- Kahn, D. (1996). The Codebreakers
- Singh, S. (1999). The Code Book
- [Implementation](../../src/ciphers/polybius.rs)
