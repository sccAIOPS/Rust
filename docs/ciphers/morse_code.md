# Morse Code

## 1. Overview

**Morse Code** is a character encoding scheme that represents letters, numbers, and punctuation as sequences of short and long signals (dots and dashes). Developed in the 1830s-1840s by Samuel Morse and Alfred Vail, it was designed for telegraph communication.

### Historical Context
- **1837**: Morse demonstrates electrical telegraph
- **1844**: First telegraph message "What hath God wrought"
- **1851**: International Morse Code standardized
- **1999**: Maritime Morse requirement dropped
- **Present**: Amateur radio, emergency signaling

### Classification

Morse Code is:
- **Not a cipher**: No secret key
- **Character encoding**: Like ASCII, but for signaling
- **Variable-length code**: Efficient for common letters

## 2. Mathematical Foundation

### 2.1 Encoding Scheme

**Basic elements**:
- **Dot (dit)**: Short signal, written as `.`
- **Dash (dah)**: Long signal (3× dot length), written as `-`

### 2.2 Timing Rules

| Element | Duration |
|---------|----------|
| Dot | 1 unit |
| Dash | 3 units |
| Intra-character gap | 1 unit |
| Inter-character gap | 3 units |
| Inter-word gap | 7 units |

### 2.3 Code Table

**Letters**:
```
A: .-      N: -.
B: -...    O: ---
C: -.-.    P: .--.
D: -..     Q: --.-
E: .       R: .-.
F: ..-.    S: ...
G: --.     T: -
H: ....    U: ..-
I: ..      V: ...-
J: .---    W: .--
K: -.-     X: -..-
L: .-..    Y: -.--
M: --      Z: --..
```

**Digits**:
```
0: -----   5: .....
1: .----   6: -....
2: ..---   7: --...
3: ...--   8: ---..
4: ....-   9: ----.
```

### 2.4 Information Theory

Morse Code uses variable-length encoding:
- Common letters (E, T) are short
- Rare letters (Q, Z) are long

This is a form of **entropy coding** predating Huffman by ~100 years.

## 3. Algorithm Description

### 3.1 Encoding Pseudocode

```
FUNCTION morse_encode(text):
    morse_dict ← load_morse_dictionary()
    result ← []
    
    FOR each character c in uppercase(text):
        IF c = ' ':
            result.append('/')  // Word separator
        ELSE IF c in morse_dict:
            result.append(morse_dict[c])
    
    RETURN join(result, ' ')  // Space between characters

FUNCTION morse_decode(morse):
    reverse_dict ← invert(morse_dictionary)
    result ← ""
    
    words ← split(morse, '/')
    FOR each word in words:
        characters ← split(word, ' ')
        FOR each code in characters:
            IF code in reverse_dict:
                result ← result + reverse_dict[code]
        result ← result + ' '
    
    RETURN trim(result)
```

### 3.2 Step-by-Step Example

**Text**: "HELLO WORLD"

| Char | Morse Code |
|------|------------|
| H | .... |
| E | . |
| L | .-.. |
| L | .-.. |
| O | --- |
| (space) | / |
| W | .-- |
| O | --- |
| R | .-. |
| L | .-.. |
| D | -.. |

**Morse**: `.... . .-.. .-.. --- / .-- --- .-. .-.. -..`

### 3.3 Decoding Example

**Morse**: `... --- ...`

| Code | Character |
|------|-----------|
| ... | S |
| --- | O |
| ... | S |

**Text**: "SOS"

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Encoding | O(n) with hash map lookup |
| Decoding | O(m) where m = morse length |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Dictionary | O(1) - fixed 36+ entries |
| Output | O(n) to O(4n) - variable expansion |

### 4.3 Average Code Length

| Letter | Frequency | Length | Contribution |
|--------|-----------|--------|--------------|
| E | 12.7% | 1 | 0.127 |
| T | 9.1% | 1 | 0.091 |
| A | 8.2% | 2 | 0.164 |
| ... | ... | ... | ... |

Average ≈ 3.5 signals per character (English text)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::HashMap;

lazy_static! {
    static ref MORSE_TABLE: HashMap<char, &'static str> = {
        let mut m = HashMap::new();
        m.insert('A', ".-");
        m.insert('B', "-...");
        m.insert('C', "-.-.");
        // ... complete alphabet
        m.insert('0', "-----");
        m.insert('1', ".----");
        // ... complete digits
        m
    };
    
    static ref REVERSE_TABLE: HashMap<&'static str, char> = {
        MORSE_TABLE.iter().map(|(&c, &m)| (m, c)).collect()
    };
}

pub fn encode(text: &str) -> String {
    text.to_uppercase()
        .chars()
        .filter_map(|c| {
            if c == ' ' {
                Some("/".to_string())
            } else {
                MORSE_TABLE.get(&c).map(|s| s.to_string())
            }
        })
        .collect::<Vec<_>>()
        .join(" ")
}

pub fn decode(morse: &str) -> String {
    morse
        .split('/')
        .map(|word| {
            word.split_whitespace()
                .filter_map(|code| REVERSE_TABLE.get(code))
                .collect::<String>()
        })
        .collect::<Vec<_>>()
        .join(" ")
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Unknown characters | Skip or placeholder |
| Empty input | Return empty |
| Multiple spaces | Preserve or normalize |
| Lowercase input | Convert to uppercase |
| Invalid Morse | Skip invalid sequences |

## 6. Variants and Extensions

### 6.1 International vs American Morse

| Feature | American | International |
|---------|----------|---------------|
| Character set | Limited | Comprehensive |
| Standardization | Historical | ITU standard |
| Usage | Obsolete | Current |

### 6.2 Prosigns (Procedure Signs)

| Prosign | Meaning | Code |
|---------|---------|------|
| AR | End of message | .-.-. |
| BT | Break | -...- |
| SK | End of contact | ...-.- |
| SOS | Distress | ...---... |

### 6.3 Q Codes

Standardized three-letter codes:
- QTH: Location
- QSL: Acknowledgment
- QRM: Interference

## 7. Physical Transmission

### 7.1 Sound/Light Signals

```
DOT:  ▮
DASH: ▮▮▮

"SOS" timing:
▮ ▮ ▮   ▮▮▮ ▮▮▮ ▮▮▮   ▮ ▮ ▮
  S         O           S
```

### 7.2 Transmission Speed

Measured in **WPM** (words per minute):
- Standard word: "PARIS" (50 dots worth)
- 5 WPM: Beginner
- 20 WPM: Proficient
- 40+ WPM: Expert

## 8. Applications

### 8.1 Historical Uses

| Era | Application |
|-----|-------------|
| 1840s-1900s | Telegraph communication |
| 1900s-1990s | Maritime distress (SOS) |
| WWI/WWII | Military communication |
| 1900s-present | Amateur radio |

### 8.2 Modern Uses

| Application | Context |
|-------------|---------|
| Amateur radio | Preferred for weak signals |
| Aviation | Some navigation beacons |
| Emergency | SOS still recognized |
| Education | Learning tool |
| Accessibility | Alternative input method |

## 9. Information Theory Perspective

### 9.1 Comparison to Binary

| Encoding | Bits per char | Notes |
|----------|---------------|-------|
| ASCII | 7 fixed | Uniform |
| Morse | ~3.5 avg | Variable |
| Huffman | ~4.5 avg | Optimal for given frequencies |

### 9.2 Prefix-Free Property

Morse Code is **NOT** prefix-free:
- E = `.`
- I = `..`
- S = `...`

This requires explicit delimiters (spaces between characters).

Compare to Huffman codes which are prefix-free (no delimiters needed).

## 10. Decoding Ambiguity

### 10.1 Without Delimiters

```
".-" could be:
- A (as single character)
- ET (E + T)

"-.." could be:
- D (as single character)
- TE (T + E)
- TEE (T + E + E)
```

### 10.2 Example Ambiguity

`.....` without spaces could be:
- EEEEE
- EEI + others
- H + E
- S + EE
- 5 (digit)

This is why character spacing is critical!

## 11. Learning Morse Code

### 11.1 Mnemonics

| Letter | Mnemonic | Code |
|--------|----------|------|
| A | a-LERT | .- |
| B | BOO-mer-rang-it | -... |
| C | CO-ca CO-la | -.-. |
| M | MMMM (hum) | -- |
| O | OH MY GOD | --- |

### 11.2 Koch Method

1. Start with two characters at full speed
2. Add one new character when 90% accurate
3. Repeat until complete alphabet learned

## 12. Related Encodings

| Encoding | Relationship |
|----------|--------------|
| ASCII | Fixed-length digital encoding |
| Huffman | Optimal variable-length |
| Binary | Two symbols like Morse |
| Braille | Tactile character encoding |

## 13. Fun Facts

1. **SOS** was chosen because it's easy to send: `...---...`
2. **E** and **T** are single signals because they're most common
3. The word "Morse" in Morse Code: `-- --- .-. ... .`
4. Experienced operators can "hear" words, not individual letters
5. The longest letter is **J**: `.---` (4 signals)

## 14. References

- ITU-R M.1677: International Morse Code
- ARRL Handbook for Radio Communications
- [Implementation](../../src/ciphers/morse_code.rs)
