# Code Style and Conventions

## Naming Conventions

### General Rules
- **Do NOT use acronyms**: Use full names (e.g., `depth_first_search` NOT `DFS`)
- **snake_case** for function and variable names
- **PascalCase** for types, traits, and structs
- **SCREAMING_SNAKE_CASE** for constants

### Module and File Naming
- Module names in snake_case
- One algorithm per file
- File name matches the primary exported function/struct

## Project Structure Pattern

```
src/
  category_name/
    mod.rs              # Module exports
    algorithm_name.rs   # Algorithm implementation + tests
```

### mod.rs Pattern
```rust
mod my_algorithm;

pub use self::my_algorithm::my_algorithm;
```

### Algorithm File Pattern
```rust
pub fn my_algorithm() {
    // Implementation
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_name() {
        // Test implementation
    }
}
```

## Code Style

### Function Signatures
- Use generics with trait bounds (e.g., `T: Ord`, `T: PartialOrd`)
- Prefer slices (`&[T]`, `&mut [T]`) over Vec references when possible
- Use `pub` for exported functions, private helpers can be named with `_` prefix (e.g., `_quick_sort`)

### Testing Best Practices
1. Tests go in a `#[cfg(test)] mod tests` block at the end of each file
2. Import from parent: `use super::*;`
3. Test multiple scenarios: empty, single element, sorted, reverse-sorted, random
4. For sorting: verify both `is_sorted()` and `have_same_elements()`
5. **Important**: If a test takes > 300ms, add `#[ignore]` attribute

### Clippy Configuration
The project uses very strict Clippy settings (pedantic, restriction, nursery, cargo categories as warnings) with many specific lints allowed. Check `Cargo.toml` `[lints.clippy]` section for details.

## Type Annotations
- Use type annotations where clarity is needed
- Leverage Rust's type inference when obvious
- Generic bounds should be minimal but sufficient

## Comments and Documentation
- Doc comments (`///`) for public APIs
- Inline comments for complex logic
- Link to algorithm explanations when helpful
