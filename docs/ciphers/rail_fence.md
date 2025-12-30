# Rail Fence Cipher

## 1. Overview

The **Rail Fence Cipher** (also known as Zigzag Cipher) is a transposition cipher that writes plaintext in a zigzag pattern across multiple "rails" and then reads off each rail in sequence to produce ciphertext.

### Historical Context
- **Ancient**: Used in various forms throughout history
- **Civil War**: Employed by both Union and Confederate forces
- **Present**: Educational tool, puzzle/CTF challenges

⚠️ **Warning**: Rail Fence provides minimal security. Use only for educational purposes.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: 
- Plaintext of length $n$
- Number of rails $r$ (key)

**Output**: Ciphertext (transposed text)

### 2.2 Pattern Analysis

For $r$ rails, the pattern repeats every $2(r-1)$ characters.

**Cycle length**: $c = 2(r-1)$

**Position mapping**: Character at position $i$ goes to rail:
$$rail(i) = \begin{cases} 
i \mod c & \text{if } i \mod c < r \\
c - (i \mod c) & \text{otherwise}
\end{cases}$$

## 3. Algorithm Description

### 3.1 Intuition

Imagine multiple horizontal rails:
1. Write letters diagonally down
2. When hitting bottom, write diagonally up
3. Repeat until text exhausted
4. Read each rail left to right

### 3.2 Visual Example

**Plaintext**: "WEAREDISCOVERED"
**Rails**: 3

```
Rail 0: W . . . E . . . C . . . R . .
Rail 1: . E . R . D . S . O . E . E .
Rail 2: . . A . . . I . . . V . . . D

Reading: WECR + ERDSOEEE + AIVD = "WECRERDSOEEEAIVD"
```

### 3.3 Pseudocode

```
FUNCTION rail_fence_encrypt(plaintext, rails):
    IF rails ≤ 1 OR rails ≥ len(plaintext):
        RETURN plaintext
    
    // Create rails
    fence ← array of 'rails' empty strings
    
    // Track position and direction
    rail ← 0
    direction ← 1  // 1 = down, -1 = up
    
    FOR each character c in plaintext:
        fence[rail] ← fence[rail] + c
        
        // Change direction at top or bottom
        IF rail = 0:
            direction ← 1
        ELSE IF rail = rails - 1:
            direction ← -1
        
        rail ← rail + direction
    
    RETURN concatenate(fence)

FUNCTION rail_fence_decrypt(ciphertext, rails):
    IF rails ≤ 1 OR rails ≥ len(ciphertext):
        RETURN ciphertext
    
    n ← len(ciphertext)
    
    // Calculate characters per rail
    rail_lengths ← calculate_rail_lengths(n, rails)
    
    // Split ciphertext into rails
    fence ← split ciphertext according to rail_lengths
    
    // Read in zigzag order
    plaintext ← ""
    rail_indices ← [0] * rails
    rail ← 0
    direction ← 1
    
    FOR i FROM 0 TO n - 1:
        plaintext ← plaintext + fence[rail][rail_indices[rail]]
        rail_indices[rail] ← rail_indices[rail] + 1
        
        IF rail = 0:
            direction ← 1
        ELSE IF rail = rails - 1:
            direction ← -1
        
        rail ← rail + direction
    
    RETURN plaintext
```

### 3.4 Step-by-Step Example

**Plaintext**: "ATTACKATDAWN"
**Rails**: 3

**Step 1**: Create zigzag pattern
```
Position:  0 1 2 3 4 5 6 7 8 9 10 11
Character: A T T A C K A T D A W  N
Rail:      0 1 2 1 0 1 2 1 0 1 2  1

Rail 0: A . . . C . . . D . .  .  → ACD
Rail 1: . T . A . K . T . A .  N  → TAKTACN
Rail 2: . . T . . . A . . . W  .  → TAW
```

**Step 2**: Read rails in order
```
Ciphertext = "ACD" + "TAKTACN" + "TAW" = "ACDTAKTACNTAW"
```

Wait, let me recalculate:
```
Position:  0 1 2 3 4 5 6 7 8 9 10 11
Character: A T T A C K A T D A W  N

Rail pattern: 0 1 2 1 0 1 2 1 0 1 2  1

Rail 0: A(0), C(4), D(8)        → ACD
Rail 1: T(1), A(3), K(5), T(7), A(9), N(11) → TAKTAN
Rail 2: T(2), A(6), W(10)      → TAW
```

**Ciphertext**: "ACDTAKTANTAW"

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Encryption | **O(n)** |
| Decryption | **O(n)** |

Single pass through text for both operations.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Rail arrays | O(n) total |
| Indices | O(r) |
| **Total** | **O(n)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn rail_fence_encrypt(text: &str, rails: usize) -> String {
    if rails <= 1 || rails >= text.len() {
        return text.to_string();
    }
    
    let chars: Vec<char> = text.chars().collect();
    let mut fence: Vec<Vec<char>> = vec![Vec::new(); rails];
    
    let mut rail = 0;
    let mut direction = 1i32;
    
    for &c in &chars {
        fence[rail].push(c);
        
        if rail == 0 {
            direction = 1;
        } else if rail == rails - 1 {
            direction = -1;
        }
        
        rail = (rail as i32 + direction) as usize;
    }
    
    fence.into_iter().flatten().collect()
}

pub fn rail_fence_decrypt(cipher: &str, rails: usize) -> String {
    if rails <= 1 || rails >= cipher.len() {
        return cipher.to_string();
    }
    
    let n = cipher.len();
    
    // Calculate how many chars per rail
    let mut rail_lens = vec![0; rails];
    let mut rail = 0;
    let mut direction = 1i32;
    
    for _ in 0..n {
        rail_lens[rail] += 1;
        if rail == 0 {
            direction = 1;
        } else if rail == rails - 1 {
            direction = -1;
        }
        rail = (rail as i32 + direction) as usize;
    }
    
    // Split ciphertext into rails
    let chars: Vec<char> = cipher.chars().collect();
    let mut fence: Vec<Vec<char>> = Vec::new();
    let mut pos = 0;
    for &len in &rail_lens {
        fence.push(chars[pos..pos + len].to_vec());
        pos += len;
    }
    
    // Read in zigzag order
    let mut result = String::new();
    let mut indices = vec![0; rails];
    rail = 0;
    direction = 1;
    
    for _ in 0..n {
        result.push(fence[rail][indices[rail]]);
        indices[rail] += 1;
        
        if rail == 0 {
            direction = 1;
        } else if rail == rails - 1 {
            direction = -1;
        }
        rail = (rail as i32 + direction) as usize;
    }
    
    result
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Rails = 1 | No transformation |
| Rails ≥ text length | No transformation |
| Rails = 2 | Simplest zigzag |
| Empty text | Return empty |

### 5.3 Rail Length Formula

For text length $n$ and $r$ rails:
```
cycle = 2 * (r - 1)
full_cycles = n / cycle
remainder = n % cycle

Top rail (0): full_cycles + (1 if remainder > 0 else 0)
Bottom rail (r-1): full_cycles + (1 if remainder >= r else 0)
Middle rails: 2 * full_cycles + (contribution from remainder)
```

## 6. Security Analysis

### 6.1 Key Space

| Rails | Permutations |
|-------|--------------|
| 2 | 1 |
| 3 | 1 |
| n | 1 per rail count |

**Total keys**: Only $n-2$ meaningful keys (2 to n-1 rails)

### 6.2 Cryptanalysis Methods

| Attack | Effectiveness |
|--------|---------------|
| Brute force | Trivial (try all rail counts) |
| Pattern recognition | Visible in ciphertext |
| Known plaintext | Instantly reveals rails |

### 6.3 Why It's Weak

1. **Very few keys**: Only text_length - 2 options
2. **Predictable pattern**: Zigzag is easily recognized
3. **Letter frequency**: Unchanged
4. **No substitution**: Original letters preserved

## 7. Variants

### 7.1 Rail Fence with Offset

Start from different position in the zigzag:
- Adds another parameter (0 to cycle-1)
- Slightly increases key space
- Still easily broken

### 7.2 Multiple Rail Fence

Apply rail fence multiple times with different rail counts:
```
Round 1: 3 rails
Round 2: 5 rails
Round 3: 4 rails
```

### 7.3 Combined with Substitution

Rail Fence + Caesar/Vigenère:
1. Apply substitution
2. Apply transposition
3. Better than either alone

## 8. Visualization

### 8.1 Pattern for 4 Rails

```
Text: ABCDEFGHIJKLMNOP (16 chars)
Cycle length: 2*(4-1) = 6

Rail 0: A . . . . . G . . . . . M . . .
Rail 1: . B . . . F . H . . . L . N . .
Rail 2: . . C . E . . . I . K . . . O .
Rail 3: . . . D . . . . . J . . . . . P

Pattern indices per rail:
Rail 0: 0, 6, 12     (every 6)
Rail 1: 1, 5, 7, 11, 13  (1, +4, +2, +4, +2...)
Rail 2: 2, 4, 8, 10, 14  (2, +2, +4, +2, +4...)
Rail 3: 3, 9, 15     (every 6, offset 3)
```

## 9. Real-World Applications

### 9.1 Historical Uses

| Context | Usage |
|---------|-------|
| Civil War | Quick field encryption |
| WWI | Combined with other methods |
| Escape room puzzles | Common cipher challenge |

### 9.2 Modern Uses

- **CTF competitions**: Entry-level challenge
- **Education**: Teaching transposition concepts
- **Puzzles**: Crosswords, geocaching

## 10. Educational Value

### 10.1 Concepts Demonstrated

| Concept | Rail Fence Example |
|---------|-------------------|
| Transposition | Characters rearranged |
| Pattern-based encryption | Zigzag structure |
| Key as parameter | Rail count |
| Inverse operations | Decrypt reverses encrypt |

### 10.2 Programming Concepts

- Iterator patterns
- State machines (direction tracking)
- Two-pass algorithms (decrypt)
- Index calculation

## 11. Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| Columnar Transposition | Different arrangement |
| Route Cipher | Arbitrary path through grid |
| Scytale | Ancient transposition |

## 12. References

- Kahn, D. (1996). The Codebreakers
- Singh, S. (1999). The Code Book
- [Implementation](../../src/ciphers/rail_fence.rs)
