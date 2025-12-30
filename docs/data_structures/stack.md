# Stack Using Singly Linked List

## 1. Overview

A Stack is a linear data structure following the Last-In-First-Out (LIFO) principle: the most recently added element is the first to be removed. This implementation uses a singly linked list where the head of the list serves as the top of the stack, enabling O(1) operations.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Design a collection supporting:
- **Push**: Add element to the top
- **Pop**: Remove and return element from the top
- **Peek/Top**: View top element without removal
- **isEmpty**: Check if stack is empty

### 2.2 Mathematical Model

**Stack as Sequence**:
$$S = \langle a_0, a_1, ..., a_{n-1} \rangle$$

where $a_{n-1}$ is the top.

**Operations**:
- $\text{push}(S, x) = \langle a_0, ..., a_{n-1}, x \rangle$
- $\text{pop}(S) = \langle a_0, ..., a_{n-2} \rangle$ (returns $a_{n-1}$)

**LIFO Property**: $\text{pop}(\text{push}(S, x)) = x$

### 2.3 Invariants

1. Last element added is first removed
2. Only the top element is accessible
3. Underlying list head = stack top

## 3. Algorithm Description

### 3.1 Intuition

Think of a stack of plates: you can only add or remove plates from the top. The linked list head naturally serves as the "top" because insertion and deletion at the head are O(1).

### 3.2 Structure Visualization

```
Stack (top on left):

Push(10), Push(20), Push(30):

Top
 ↓
[30|→]──→[20|→]──→[10|/]

Pop() returns 30:

Top
 ↓
[20|→]──→[10|/]

Push(40):

Top
 ↓
[40|→]──→[20|→]──→[10|/]
```

### 3.3 Pseudocode

```
PUSH(stack, value):
    new_node = Node(value)
    new_node.next = stack.top
    stack.top = new_node
    stack.size++

POP(stack):
    if stack.top is None:
        error "Stack underflow"
    value = stack.top.data
    stack.top = stack.top.next
    stack.size--
    return value

PEEK(stack):
    if stack.top is None:
        error "Stack empty"
    return stack.top.data

IS_EMPTY(stack):
    return stack.top is None
```

### 3.4 Step-by-Step Example

**Expression evaluation: 3 + 4 × 2 (postfix: 3 4 2 × +)**

```
Initial: Stack = []

Read 3: push(3)
Stack: [3]

Read 4: push(4)
Stack: [4] → [3]

Read 2: push(2)
Stack: [2] → [4] → [3]

Read ×: pop() = 2, pop() = 4
        push(4 × 2 = 8)
Stack: [8] → [3]

Read +: pop() = 8, pop() = 3
        push(3 + 8 = 11)
Stack: [11]

Result: pop() = 11
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Linked List | Array-based |
|-----------|-------------|-------------|
| Push      | O(1)        | O(1) amortized |
| Pop       | O(1)        | O(1)        |
| Peek      | O(1)        | O(1)        |
| isEmpty   | O(1)        | O(1)        |

### 4.2 Space Complexity

- **Linked List**: O(n) with pointer overhead per element
- **Array**: O(n) but more cache-friendly

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
type Link<T> = Option<Box<Node<T>>>;

struct Node<T> {
    data: T,
    next: Link<T>,
}

pub struct Stack<T> {
    top: Link<T>,
    length: usize,
}

impl<T> Stack<T> {
    pub fn new() -> Self {
        Stack { top: None, length: 0 }
    }
    
    pub fn push(&mut self, data: T) {
        let new_node = Box::new(Node {
            data,
            next: self.top.take(),
        });
        self.top = Some(new_node);
        self.length += 1;
    }
    
    pub fn pop(&mut self) -> Option<T> {
        self.top.take().map(|node| {
            self.top = node.next;
            self.length -= 1;
            node.data
        })
    }
    
    pub fn peek(&self) -> Option<&T> {
        self.top.as_ref().map(|node| &node.data)
    }
}
```

**Key Design Patterns**:
- `Option::take()` to transfer ownership
- `Option::map()` for clean transformations
- `as_ref()` for borrowing without ownership transfer

### 5.2 Iterator Implementation

```rust
impl<T> Stack<T> {
    pub fn iter(&self) -> impl Iterator<Item = &T> {
        StackIter { next: self.top.as_deref() }
    }
}

struct StackIter<'a, T> {
    next: Option<&'a Node<T>>,
}

impl<'a, T> Iterator for StackIter<'a, T> {
    type Item = &'a T;
    
    fn next(&mut self) -> Option<Self::Item> {
        self.next.map(|node| {
            self.next = node.next.as_deref();
            &node.data
        })
    }
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Pop from empty | Return None |
| Peek on empty | Return None |
| Single element | After pop, top = None |
| Push after empty | Creates new top |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Function Call Stack**: Runtime stack for function calls/returns
2. **Undo/Redo**: Editor operations stored on stack
3. **Expression Evaluation**: Postfix/prefix expression parsing
4. **Bracket Matching**: Validate parentheses in code
5. **DFS (Depth-First Search)**: Graph/tree traversal
6. **Backtracking**: Puzzle solving, path finding
7. **Browser History**: Back button implementation

### 6.2 Expression Parsing Example

**Infix to Postfix Conversion** (using operator stack):
```
Input: A + B × C
        
Token   | Action              | Output    | Stack
--------|---------------------|-----------|-------
A       | Output              | A         | 
+       | Push                | A         | +
B       | Output              | A B       | +
×       | × > +, push         | A B       | + ×
C       | Output              | A B C     | + ×
(end)   | Pop all             | A B C × + | 

Result: A B C × +
```

### 6.3 Stack vs Queue

| Aspect | Stack (LIFO) | Queue (FIFO) |
|--------|--------------|--------------|
| Order | Last in, first out | First in, first out |
| Use case | DFS, undo, recursion | BFS, scheduling |
| Analogy | Stack of plates | Line at store |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Stack Overflow**: Unlimited growth without bounds
2. **Memory Leaks**: In non-GC languages
3. **Empty Stack Operations**: Always check before pop/peek
4. **Linked vs Array**: Choosing wrong implementation

### 7.2 Linked List vs Vec for Stack

| Aspect | Linked List | Vec |
|--------|-------------|-----|
| Push | O(1), allocates | O(1) amortized |
| Pop | O(1), deallocates | O(1), no dealloc |
| Memory | Pointer overhead | Contiguous |
| Cache | Poor locality | Good locality |
| Best for | Unknown size, many ops | Known size, cache matters |

### 7.3 Standard Library

```rust
// Vec as a stack (recommended for most cases)
let mut stack: Vec<i32> = Vec::new();
stack.push(1);
stack.push(2);
let top = stack.pop();  // Some(2)
let peek = stack.last();  // Some(&1)

// For production code, Vec is usually better than
// a linked list due to cache efficiency
```

### 7.4 When to Use Linked List Implementation

- Teaching/educational purposes
- When each element is large (reduces copying)
- When you need to split/merge stacks efficiently
- Memory fragmentation concerns with large arrays

## 8. References

- Knuth, D. E. (1997). *The Art of Computer Programming, Volume 1: Fundamental Algorithms* (3rd ed.). Section 2.2.1.
- Sedgewick, R. (1988). *Algorithms* (2nd ed.). Addison-Wesley. Chapter 4.
- "Learning Rust With Entirely Too Many Linked Lists". https://rust-unofficial.github.io/too-many-lists/
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Section 10.1.
