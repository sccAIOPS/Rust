# Repository Copilot Standards

## Overview
This is **TheAlgorithms/Rust** - an educational algorithms library with 21 categories and 200+ algorithm implementations.

## Active Agents
The following agents have been generated based on codebase analysis:

| Agent | Role | Primary Use Case |
|-------|------|------------------|
| `@Senior-Architecture_Analyst` | Architecture | Extract UML diagrams, C4 models, module structure |
| `@Senior-Design_Pattern_Analyst` | Patterns | Identify design patterns, data structures, Rust idioms |
| `@Principal-Algorithm_Analyst` | Analysis | Deep algorithm analysis, complexity verification, pitfalls |
| `@Senior-Feature_Planner` | Planning | BDD specs, requirements, SOLID/KISS compliance |
| `@Principal-System_Designer` | Design | High-level and low-level architecture design |
| `@Senior-Rust_Implementer` | Implementation | TDD-based algorithm implementation |
| `@Senior-Code_Reviewer` | Review | Code quality, style, security review |
| `@Senior-QA_Tester` | Testing | Unit, integration, property-based, performance testing |

## Topology
Using **Star Topology** structure.
- Reason: Single-module educational library with consistent patterns
- Central Hub: This copilot-instructions.md orchestrates all agents
- Agents operate independently on their specific domains

## Agent Workflow Chains

### Chain 1: Documentation & Analysis
```
@Senior-Architecture_Analyst → @Senior-Design_Pattern_Analyst → @Principal-Algorithm_Analyst
```
Use for: Understanding existing codebase, generating documentation, identifying improvements

### Chain 2: Feature Development (Full Cycle)
```
@Senior-Feature_Planner → @Principal-System_Designer → @Senior-Rust_Implementer → @Senior-Code_Reviewer → @Senior-QA_Tester
```
Use for: Adding new algorithms or data structures

### Chain 3: Quick Implementation
```
@Senior-Rust_Implementer → @Senior-Code_Reviewer
```
Use for: Simple algorithm additions with clear requirements

### Chain 4: Quality Assurance
```
@Senior-Code_Reviewer → @Senior-QA_Tester
```
Use for: Reviewing and testing existing implementations

## Workflow Rules

### 1. File Operations
- Always reference existing file paths when answering
- Check `src/lib.rs` for module structure
- Verify algorithm doesn't exist before creating

### 2. Code Standards
- Run `cargo fmt` before committing
- Run `cargo clippy --all -- -D warnings` 
- Run `cargo test` to verify all tests pass
- Follow snake_case naming (no acronyms: `depth_first_search` not `DFS`)

### 3. Dependencies
- Check `Cargo.toml` for existing dependencies
- Prefer standard library over external crates
- New dependencies require justification

### 4. Testing
- Every algorithm needs tests in `#[cfg(test)] mod tests`
- Include edge cases: empty, single element, sorted, reverse
- Use quickcheck for property-based tests when applicable
- Mark slow tests (>300ms) with `#[ignore]`

## Project Structure Reference
```
src/
├── lib.rs                    # Main exports
├── backtracking/            # N-Queens, Sudoku, etc.
├── big_integer/             # Large number operations
├── bit_manipulation/        # Bitwise algorithms
├── ciphers/                 # Cryptographic algorithms
├── compression/             # Data compression
├── conversions/             # Number base conversions
├── data_structures/         # Trees, Graphs, Heaps, etc.
├── dynamic_programming/     # DP algorithms
├── financial/               # Financial calculations
├── general/                 # General utilities
├── geometry/                # Geometric algorithms
├── graph/                   # Graph algorithms
├── greedy/                  # Greedy algorithms
├── machine_learning/        # ML algorithms
├── math/                    # Mathematical algorithms
├── navigation/              # Navigation algorithms
├── number_theory/           # Number theory
├── searching/               # Search algorithms
├── signal_analysis/         # Signal processing
├── sorting/                 # 30+ sorting algorithms
└── string/                  # String algorithms
```

## Quick Start Examples

### Extract Architecture
```
@Senior-Architecture_Analyst generate C4 container diagram for this repository
```

### Analyze Algorithm
```
@Principal-Algorithm_Analyst analyze the quick_sort implementation for pitfalls and optimization opportunities
```

### Plan New Feature
```
@Senior-Feature_Planner create a feature plan for implementing Tim Sort algorithm
```

### Implement Algorithm
```
@Senior-Rust_Implementer implement the jump_search algorithm in the searching module using TDD
```

### Review Code
```
@Senior-Code_Reviewer review the recent changes to the AVL tree implementation
```

### Test Feature
```
@Senior-QA_Tester create comprehensive tests for the binary_search_tree module
```

## Quality Gates
Before any PR is merged:
- [ ] `cargo fmt --check` passes
- [ ] `cargo clippy --all -- -D warnings` passes
- [ ] `cargo test` passes
- [ ] Documentation is complete
- [ ] Tests cover edge cases
