# Floyd's Cycle Detection Algorithm

## 1. Overview

Floyd's Cycle Detection Algorithm (also known as the "Tortoise and Hare" algorithm) detects cycles in sequences using only O(1) space. Given a function f and starting point x₀, it determines whether the sequence x₀, f(x₀), f(f(x₀)), ... eventually repeats, and if so, finds where the cycle begins and its length.

Invented by Robert W. Floyd in 1967.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- A function $f: S \rightarrow S$ on a finite set $S$
- A starting element $x_0 \in S$

Find:
- **μ (mu)**: The smallest index where the cycle begins
- **λ (lambda)**: The length of the cycle

The sequence is: $x_0, x_1 = f(x_0), x_2 = f(x_1), ...$

### 2.2 Mathematical Model

**Sequence Structure** (ρ-shaped):
```
x₀ → x₁ → x₂ → ... → x_μ → x_{μ+1} → ... → x_{μ+λ-1}
                       ↑___________________________|
                       
μ = "tail" length (index where cycle starts)
λ = cycle length
```

**Cycle Property**: For all $i \geq μ$: $x_i = x_{i+λ}$

**Meeting Point**: If tortoise (slow) and hare (fast) start together and hare moves twice as fast, they meet inside the cycle at some point $x_ν$ where $ν = μ + k$ for some $k < λ$.

### 2.3 Key Theorem

**Theorem**: When tortoise and hare first meet, they are at position $ν$ where $ν ≡ 0 \pmod{λ}$ relative to the cycle start.

**Proof**: 
- At meeting time t, tortoise is at position t, hare at position 2t
- Both are in cycle: $t ≥ μ$ and $2t ≥ μ$
- Same position means: $2t - t ≡ 0 \pmod{λ}$, so $t ≡ 0 \pmod{λ}$
- Distance from cycle start: $(t - μ) \bmod λ$

**Finding μ**: After meeting at position ν, restart tortoise at x₀. Both move one step at a time. They meet at x_μ (the cycle start).

## 3. Algorithm Description

### 3.1 Intuition

Imagine a circular track with a tail. A tortoise and hare start at the same point. The hare runs twice as fast. If there's no cycle, the hare falls off the end. If there's a cycle, the hare eventually laps the tortoise inside the cycle.

### 3.2 Visualization

```
Sequence: 1 → 2 → 3 → 4 → 5 → 6 → 3 (cycles back)
                     ↑___________↓
μ = 2 (cycle starts at index 2, value 3)
λ = 4 (cycle length: 3 → 4 → 5 → 6 → 3)

Phase 1 - Find meeting point:
Step  Tortoise  Hare
 0      1        1
 1      2        3
 2      3        5
 3      4        3
 4      5        5    ← Meet!

Phase 2 - Find cycle start (μ):
Restart tortoise at start, both move at same speed:
Step  Tortoise  Hare(from meeting)
 0      1        5
 1      2        6
 2      3        3    ← Meet at cycle start!

μ = 2

Phase 3 - Find cycle length (λ):
Keep hare still, move tortoise until back:
  3 → 4 → 5 → 6 → 3  (4 steps)
λ = 4
```

### 3.3 Pseudocode

```
DETECT_CYCLE(f, x0):
    // Phase 1: Find meeting point
    tortoise = f(x0)
    hare = f(f(x0))
    
    while tortoise != hare:
        if hare reaches end:
            return NO_CYCLE
        tortoise = f(tortoise)
        hare = f(f(hare))
    
    // Phase 2: Find start of cycle (μ)
    mu = 0
    tortoise = x0
    while tortoise != hare:
        tortoise = f(tortoise)
        hare = f(hare)
        mu++
    
    // Phase 3: Find length of cycle (λ)
    lambda = 1
    hare = f(tortoise)
    while tortoise != hare:
        hare = f(hare)
        lambda++
    
    return (mu, lambda)
```

### 3.4 Step-by-Step Example

**Linked List Cycle Detection**:

```
List: 1 → 2 → 3 → 4 → 5 → 3 (back to node 3)

Phase 1:
  T=1, H=1 (start)
  T=2, H=3 (step 1)
  T=3, H=5 (step 2)
  T=4, H=3 (step 3)
  T=5, H=5 (step 4) - MEET!

Phase 2: Find where cycle starts
  T=1, H=5
  T=2, H=3
  T=3, H=4
  Wait, that's wrong. Let me recalculate...
  
  Actually, moving both one step from meeting point (5):
  T=1, H=5
  T=2, H=3
  T=3, H=4
  T=4, H=5
  Hmm, they don't meet at 3...

Let me reconsider: the sequence is values, not positions.
f(1)=2, f(2)=3, f(3)=4, f(4)=5, f(5)=3

Phase 1:
  T=f(1)=2, H=f(f(1))=3
  T=f(2)=3, H=f(f(3))=f(4)=5
  T=f(3)=4, H=f(f(5))=f(3)=4 - MEET at 4!

Phase 2:
  T=1, H=4
  T=2, H=5
  T=3, H=3 - MEET at 3! (μ=2 steps)
  
Phase 3:
  Starting at 3, count until back to 3:
  3 → 4 → 5 → 3 (λ=3 steps)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Phase | Time |
|-------|------|
| Find meeting | O(μ + λ) |
| Find μ | O(μ) |
| Find λ | O(λ) |
| **Total** | **O(μ + λ)** |

### 4.2 Space Complexity

- **O(1)** - only two pointers needed!

This is the key advantage over hash-based detection (O(n) space).

### 4.3 Comparison

| Method | Time | Space |
|--------|------|-------|
| Floyd's | O(μ + λ) | O(1) |
| Hash Set | O(μ + λ) | O(μ + λ) |
| Brent's | O(μ + λ) | O(1) |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
/// Detects cycle in an iterable sequence
/// Returns Some((mu, lambda)) if cycle found, None otherwise
pub fn floyd<T, F>(x0: T, f: F) -> Option<(usize, usize)>
where
    T: Clone + PartialEq,
    F: Fn(&T) -> Option<T>,
{
    // Phase 1: Find meeting point
    let mut tortoise = f(&x0)?;
    let mut hare = f(&f(&x0)?)?;
    
    while tortoise != hare {
        tortoise = f(&tortoise)?;
        hare = f(&f(&hare)?)?;
    }
    
    // Phase 2: Find start of cycle (mu)
    let mut mu = 0;
    tortoise = x0.clone();
    while tortoise != hare {
        tortoise = f(&tortoise)?;
        hare = f(&hare)?;
        mu += 1;
    }
    
    // Phase 3: Find length of cycle (lambda)
    let mut lambda = 1;
    hare = f(&tortoise)?;
    while tortoise != hare {
        hare = f(&hare)?;
        lambda += 1;
    }
    
    Some((mu, lambda))
}
```

### 5.2 Linked List Version

```rust
type Link<T> = Option<Rc<RefCell<Node<T>>>>;

struct Node<T> {
    val: T,
    next: Link<T>,
}

pub fn has_cycle<T>(head: Link<T>) -> bool {
    let mut slow = head.clone();
    let mut fast = head.clone();
    
    while fast.is_some() {
        slow = slow.and_then(|n| n.borrow().next.clone());
        fast = fast
            .and_then(|n| n.borrow().next.clone())
            .and_then(|n| n.borrow().next.clone());
        
        if let (Some(s), Some(f)) = (&slow, &fast) {
            if Rc::ptr_eq(s, f) {
                return true;
            }
        }
    }
    
    false
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| No cycle | Return None when hare reaches end |
| Single element cycle | Works correctly (λ = 1) |
| Entire sequence is cycle | μ = 0 |
| Empty input | Return None |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Linked List Cycle Detection**: Classic interview problem
2. **Finding Duplicate**: In array where each value is valid index
3. **Random Number Generators**: Detect period
4. **State Machine Analysis**: Find loops
5. **Memory Leak Detection**: Circular references
6. **Pollard's Rho**: Integer factorization

### 6.2 Finding Duplicate Number

Given array where values are in [1, n] and there's exactly one duplicate:

```rust
pub fn find_duplicate(nums: &[i32]) -> i32 {
    // Treat array as linked list: nums[i] points to nums[nums[i]]
    let f = |i: usize| nums[i] as usize;
    
    // Phase 1: Find meeting point
    let mut slow = f(0);
    let mut fast = f(f(0));
    while slow != fast {
        slow = f(slow);
        fast = f(f(fast));
    }
    
    // Phase 2: Find cycle start (the duplicate)
    slow = 0;
    while slow != fast {
        slow = f(slow);
        fast = f(fast);
    }
    
    slow as i32
}
```

### 6.3 Brent's Algorithm (Optimization)

Alternative that's often faster in practice:

```rust
pub fn brent<T, F>(x0: T, f: F) -> Option<(usize, usize)>
where
    T: Clone + PartialEq,
    F: Fn(&T) -> T,
{
    // Find lambda (cycle length) first
    let mut power = 1;
    let mut lambda = 1;
    let mut tortoise = x0.clone();
    let mut hare = f(&x0);
    
    while tortoise != hare {
        if power == lambda {
            tortoise = hare.clone();
            power *= 2;
            lambda = 0;
        }
        hare = f(&hare);
        lambda += 1;
    }
    
    // Find mu (cycle start)
    let mut mu = 0;
    tortoise = x0.clone();
    hare = x0.clone();
    
    // Move hare lambda steps ahead
    for _ in 0..lambda {
        hare = f(&hare);
    }
    
    // Move both until they meet
    while tortoise != hare {
        tortoise = f(&tortoise);
        hare = f(&hare);
        mu += 1;
    }
    
    Some((mu, lambda))
}
```

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Off-by-One**: Initial positions matter (tortoise = f(x0), not x0)
2. **Null Checks**: In linked lists, check before dereferencing
3. **Equality**: Must compare values, not just positions
4. **Termination**: Handle no-cycle case properly

### 7.2 When Floyd's Might Not Work

- Elements not comparable for equality
- Function f is expensive to compute (Brent's might be better)
- Need to find all cycles (different approach needed)

### 7.3 Optimization Tips

**Early Termination**: If you only need to know IF there's a cycle (not where), stop at Phase 1.

**Brent's Algorithm**: Often faster because:
- Fewer function evaluations
- Better cache behavior
- Finds λ directly

## 8. References

- Floyd, R. W. (1967). "Non-deterministic Algorithms". *JACM*.
- Knuth, D. E. (1997). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (3rd ed.). Section 3.1.
- Brent, R. P. (1980). "An Improved Monte Carlo Factorization Algorithm". *BIT Numerical Mathematics*.
- Pollard, J. M. (1975). "A Monte Carlo Method for Factorization". *BIT Numerical Mathematics*.
