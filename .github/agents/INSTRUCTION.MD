# Agent Usage Guide

## Overview
This directory contains specialized GitHub Copilot agents for TheAlgorithms/Rust repository. Each agent is configured for a specific role in the software development lifecycle.

## Available Agents

### 1. @Senior-Architecture_Analyst
**Purpose**: Extract and document software architecture

**Capabilities**:
- Generate C4 model diagrams (Context, Container, Component)
- Create PlantUML class diagrams for data structures
- Generate sequence diagrams for algorithm flows
- Document module dependencies and relationships

**Example Usage**:
```
@Senior-Architecture_Analyst generate a C4 component diagram for the data_structures module
```

---

### 2. @Senior-Design_Pattern_Analyst
**Purpose**: Identify and document design patterns and data structures

**Capabilities**:
- Identify Rust design patterns (Newtype, Builder, Strategy, etc.)
- Classify data structure implementations
- Document algorithmic paradigms (Divide & Conquer, DP, Greedy)
- Analyze generic programming patterns

**Example Usage**:
```
@Senior-Design_Pattern_Analyst identify all design patterns used in the sorting module
```

---

### 3. @Principal-Algorithm_Analyst
**Purpose**: Deep algorithm analysis and optimization recommendations

**Capabilities**:
- Verify time/space complexity claims
- Identify algorithmic pitfalls and edge cases
- Provide optimization recommendations
- Generate comprehensive analysis reports

**Example Usage**:
```
@Principal-Algorithm_Analyst analyze the heap_sort implementation for potential issues and optimizations
```

---

### 4. @Senior-Feature_Planner
**Purpose**: Plan new features using BDD and SOLID principles

**Capabilities**:
- Create BDD specifications (Gherkin format)
- Define functional and non-functional requirements
- Apply SOLID and KISS principle checklists
- Generate feature request templates

**Example Usage**:
```
@Senior-Feature_Planner create a feature plan for adding a red-black tree implementation
```

**Input Template**:
```markdown
## Feature Request: [Algorithm Name]
- **Category**: [sorting/searching/data_structures/etc.]
- **Description**: [What it does]
- **Complexity**: Time O(?), Space O(?)
- **Priority**: [Critical/High/Medium/Low]
```

---

### 5. @Principal-System_Designer
**Purpose**: Design system and software architecture

**Capabilities**:
- Design high-level module architecture
- Create low-level component specifications
- Define interfaces and API contracts
- Generate implementation guides for developers

**Example Usage**:
```
@Principal-System_Designer design the architecture for a new graph traversal module
```

---

### 6. @Senior-Rust_Implementer
**Purpose**: Implement algorithms following TDD principles

**Capabilities**:
- Write tests first (TDD approach)
- Implement idiomatic Rust code
- Integrate with existing module structure
- Follow project coding standards

**Example Usage**:
```
@Senior-Rust_Implementer implement the jump_search algorithm using TDD
```

**Workflow**:
1. Write failing tests
2. Implement minimal code to pass
3. Refactor while keeping tests green
4. Document and integrate

---

### 7. @Senior-Code_Reviewer
**Purpose**: Review code for quality, style, and security

**Capabilities**:
- Run automated quality checks (fmt, clippy, test)
- Review for correctness and edge cases
- Check security concerns (especially for ciphers)
- Enforce project coding standards

**Example Usage**:
```
@Senior-Code_Reviewer review the new AVL tree implementation for code quality
```

**Review Levels**:
- 🔴 **Critical**: Must fix before merge
- 🟡 **Quality**: Should fix
- 🟢 **Suggestion**: Nice to have

---

### 8. @Senior-QA_Tester
**Purpose**: Comprehensive testing and quality assurance

**Capabilities**:
- Write unit tests with edge cases
- Create property-based tests (QuickCheck)
- Design integration tests
- Set up performance benchmarks

**Example Usage**:
```
@Senior-QA_Tester create comprehensive tests for the merge_sort implementation
```

**Test Types**:
- Unit tests: Basic functionality
- Edge case tests: Empty, single, sorted, etc.
- Property tests: Invariants with QuickCheck
- Performance tests: Benchmarks (marked `#[ignore]`)

---

## Workflow Chains

### Full Feature Development Cycle
```
1. @Senior-Feature_Planner    → Define requirements
2. @Principal-System_Designer → Design architecture  
3. @Senior-Rust_Implementer   → Implement with TDD
4. @Senior-Code_Reviewer      → Review implementation
5. @Senior-QA_Tester          → Comprehensive testing
```

### Documentation Cycle
```
1. @Senior-Architecture_Analyst    → Extract architecture
2. @Senior-Design_Pattern_Analyst  → Identify patterns
3. @Principal-Algorithm_Analyst    → Analyze algorithms
```

### Quick Fix Cycle
```
1. @Senior-Rust_Implementer → Quick implementation
2. @Senior-Code_Reviewer    → Fast review
```

---

## Project Standards Reference

### Naming Conventions
- Functions: `snake_case`
- Types: `PascalCase`
- Constants: `SCREAMING_SNAKE_CASE`
- **No acronyms**: Use `depth_first_search` not `DFS`

### File Structure
```rust
// src/category/algorithm_name.rs

//! Module documentation

/// Function documentation
pub fn algorithm_name<T: Ord>(input: &mut [T]) {
    // Implementation
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_basic() {
        // Test
    }
}
```

### Quality Commands
```bash
cargo fmt        # Format code
cargo clippy --all -- -D warnings  # Lint
cargo test       # Run tests
```

---

## Tips for Best Results

1. **Be Specific**: Include file names and function names when asking about existing code
2. **Provide Context**: Mention the algorithm category and expected complexity
3. **Chain Agents**: Use multiple agents for complex tasks
4. **Review Output**: Agents may need refinement; iterate as needed
5. **Follow Standards**: Agents are configured for this specific project's conventions
