# Parentheses Generator

## 1. Overview

The **Parentheses Generator** (also known as Generate Parentheses or Balanced Brackets) generates all possible combinations of well-formed (valid) parentheses given a number of pairs.

A combination is "well-formed" if every opening parenthesis `(` has a corresponding closing parenthesis `)` and they are properly nested.

### Historical Context
- **1758**: Euler studied related Catalan number problems
- **1838**: Catalan formally derived the formula for balanced parentheses
- **Modern**: Fundamental in compiler design, expression parsing, and combinatorics

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given $n$ pairs of parentheses, generate all valid (well-formed) combinations.

A string of parentheses is **valid** if:
1. It contains exactly $n$ opening and $n$ closing parentheses
2. At any prefix, the number of `)` never exceeds the number of `(`
3. The total length is $2n$

### 2.2 Mathematical Model

**Input**: Integer $n \geq 0$ (number of pairs)

**Output**: All valid parentheses strings of length $2n$

**Count**: The number of valid combinations is the **n-th Catalan number**:

$$C_n = \frac{1}{n+1}\binom{2n}{n} = \frac{(2n)!}{(n+1)!n!}$$

### 2.3 Catalan Number Sequence

| n | $C_n$ | Valid Combinations |
|---|-------|-------------------|
| 0 | 1 | `""` (empty) |
| 1 | 1 | `()` |
| 2 | 2 | `(())`, `()()` |
| 3 | 5 | `((()))`, `(()())`, `(())()`, `()(())`, `()()()` |
| 4 | 14 | 14 combinations |
| 5 | 42 | 42 combinations |

### 2.4 Recursive Definition

$$C_n = \sum_{i=0}^{n-1} C_i \cdot C_{n-1-i}$$

This reflects the structure: `(` + inner + `)` + rest.

## 3. Algorithm Description

### 3.1 Intuition

The backtracking approach builds strings character by character:
1. Start with an empty string
2. At each step, decide whether to add `(` or `)`
3. We can add `(` if we haven't used all $n$ opening parentheses
4. We can add `)` only if it doesn't exceed the count of `(`
5. When string length reaches $2n$, we have a valid combination

### 3.2 Pseudocode

```
function generate_parentheses(n):
    if n == 0:
        return []
    
    result = []
    generate("", 0, 0, n, result)
    return result

function generate(current, open_count, close_count, n, result):
    // Base case: complete valid combination
    if length(current) == 2 * n:
        result.add(current)
        return
    
    // Can add opening parenthesis?
    if open_count < n:
        generate(current + "(", open_count + 1, close_count, n, result)
    
    // Can add closing parenthesis?
    if close_count < open_count:
        generate(current + ")", open_count, close_count + 1, n, result)
```

### 3.3 Step-by-Step Example

For `n = 2`:

```
generate("", 0, 0):
└─ add '(': generate("(", 1, 0):
   ├─ add '(': generate("((", 2, 0):
   │  └─ add ')': generate("(()", 2, 1):
   │     └─ add ')': generate("(())", 2, 2) → OUTPUT "(())"
   │
   └─ add ')': generate("()", 1, 1):
      └─ add '(': generate("()(", 2, 1):
         └─ add ')': generate("()()", 2, 2) → OUTPUT "()()"

Result: ["(())", "()()"]
```

### 3.4 Visual Decision Tree

```
                    ""
                    |
                   "("
                  /   \
               "(("   "()"
               /        \
            "(()"      "()(
             |           |
           "(())"     "()()"
              ↓          ↓
           VALID      VALID
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Number of Results**: $C_n$ (n-th Catalan number)
- **String Operations**: Each string has length $2n$

**Total**: $O\left(\frac{4^n}{\sqrt{n}}\right)$ (asymptotic bound on Catalan numbers)

More precisely: $O(C_n \cdot n) = O\left(\frac{4^n}{n^{3/2}}\right)$

### 4.2 Space Complexity

- **Recursion Stack**: $O(2n) = O(n)$ depth
- **Current String**: $O(n)$
- **Result Storage**: $O(C_n \cdot n)$

**Auxiliary Space**: $O(n)$

### 4.3 Why No Pruning Needed?

Unlike other backtracking problems, every path we explore leads to a valid solution:
- We only add `(` when `open_count < n`
- We only add `)` when `close_count < open_count`

This means **no backtracking** in the traditional sense—every leaf is a valid result.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn generate_parentheses(n: usize) -> Vec<String> {
    let mut result = Vec::new();
    if n > 0 {
        generate("", 0, 0, n, &mut result);
    }
    result
}

fn generate(
    current: &str,
    open_count: usize,
    close_count: usize,
    n: usize,
    result: &mut Vec<String>,
) {
    if current.len() == (n * 2) {
        result.push(current.to_string());
        return;
    }
    // ... recursive calls with string concatenation
}
```

**Key Patterns**:
- **String slices**: `&str` for immutable passing
- **String concatenation**: Creates new strings at each level
- **Zero check**: Returns empty vector for `n = 0`
- **usize counters**: Natural fit for counting

### 5.2 Edge Cases

| Case | n | Result |
|------|---|--------|
| Zero pairs | 0 | `[]` (empty vector) |
| One pair | 1 | `["()"]` |
| Two pairs | 2 | `["(())", "()()"]` |
| Large n | 10+ | Many combinations (Catalan numbers grow fast) |

### 5.3 Potential Optimizations

1. **StringBuilder pattern**: Avoid string allocation at each level

```rust
fn generate_with_buffer(
    buffer: &mut String,
    open_count: usize,
    close_count: usize,
    n: usize,
    result: &mut Vec<String>,
) {
    if buffer.len() == n * 2 {
        result.push(buffer.clone());
        return;
    }
    
    if open_count < n {
        buffer.push('(');
        generate_with_buffer(buffer, open_count + 1, close_count, n, result);
        buffer.pop();  // Backtrack
    }
    
    if close_count < open_count {
        buffer.push(')');
        generate_with_buffer(buffer, open_count, close_count + 1, n, result);
        buffer.pop();  // Backtrack
    }
}
```

2. **Iterative approach**: Using a stack instead of recursion

3. **Closure-based approach**: Generate lazily

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Compiler Design**: Valid expression structures
2. **Code Formatting**: Balanced braces/brackets
3. **HTML/XML Validation**: Tag matching
4. **Mathematical Expressions**: Valid formula generation
5. **Test Data Generation**: Creating valid input for parsers

### 6.2 Related Problems

| Problem | Relationship |
|---------|-------------|
| Valid Parentheses (checking) | Uses stack, O(n) |
| Longest Valid Parentheses | DP or stack |
| Different Bracket Types | `()[]{}` - same principle |
| Score of Parentheses | `(()) = 2, ()() = 2` |
| Remove Invalid Parentheses | BFS/backtracking |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn generate(
    current: &str,
    open_count: usize,
    close_count: usize,
    n: usize,
    result: &mut Vec<String>,
) {
    if current.len() == (n * 2) {
        result.push(current.to_string());
        return;
    }

    if open_count < n {
        let new_str = current.to_string() + "(";
        generate(&new_str, open_count + 1, close_count, n, result);
    }

    if close_count < open_count {
        let new_str = current.to_string() + ")";
        generate(&new_str, open_count, close_count + 1, n, result);
    }
}
```

**Analysis**:
- ✅ Correct logic for valid parentheses
- ✅ Clean termination condition
- ✅ Handles edge case (n=0)
- ⚠️ Creates new string at each level (could use mutable buffer)
- ⚠️ `current.to_string()` creates allocation even when not needed

### 7.2 String Allocation Analysis

For `n = 3`:
- 5 results (Catalan number)
- Each result requires path of 6 steps
- Current implementation: ~30 string allocations
- Optimized: 6 allocations (just the results)

### 7.3 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| Results count | $C_n$ | ✅ Catalan number |
| Per-result work | $O(n)$ | ✅ String of length 2n |
| Total | $O(C_n \cdot n)$ | ✅ |
| Space (stack) | $O(n)$ | ✅ Max depth 2n |

## 8. Testing

### 8.1 Test Cases Covered

```rust
test_generate_parentheses_0: (0) → []
test_generate_parentheses_1: (1) → ["()"]
test_generate_parentheses_2: (2) → ["(())", "()()"]
test_generate_parentheses_3: (3) → ["((()))", "(()())", "(())()", "()(())", "()()()"]
test_generate_parentheses_4: (4) → [14 combinations]
```

### 8.2 Result Validation

All results are:
1. Length 2n
2. Equal count of `(` and `)`
3. Valid (no prefix has more `)` than `(`)

```rust
fn is_valid(s: &str) -> bool {
    let mut balance = 0;
    for c in s.chars() {
        match c {
            '(' => balance += 1,
            ')' => balance -= 1,
            _ => return false,
        }
        if balance < 0 { return false; }
    }
    balance == 0
}
```

## 9. Mathematical Connections

### 9.1 Dyck Words

Valid parentheses strings are called **Dyck words**. They have a bijection with:
- Paths from (0,0) to (2n,0) that don't go below x-axis
- Binary trees with n nodes
- Ways to triangulate a convex polygon
- Non-crossing partitions

### 9.2 Catalan Number Applications

The n-th Catalan number counts:
- Valid parentheses with n pairs
- Full binary trees with n+1 leaves
- Ways to cut a convex polygon into triangles
- Paths in a grid that don't cross the diagonal
- Different ways to completely parenthesize a product

### 9.3 Generating Function

$$C(x) = \sum_{n=0}^{\infty} C_n x^n = \frac{1 - \sqrt{1-4x}}{2x}$$

## 10. References

1. Stanley, R. P. (2015). *Catalan Numbers*. Cambridge University Press.
2. Knuth, D. E. (1997). *The Art of Computer Programming, Vol. 1*. Section 2.2.1.
3. Ruskey, F. (2003). *Combinatorial Generation*. Chapter 4.
4. [OEIS A000108](https://oeis.org/A000108) - Catalan numbers
5. [LeetCode Problem 22: Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)
