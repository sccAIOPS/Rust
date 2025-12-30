# Task Completion Checklist

## Before Submitting Any Changes

### Required Steps (in order)

1. **Run Tests**
   ```bash
   cargo test
   ```
   - All tests must pass
   - If a test takes > 300ms, add `#[ignore]` attribute or optimize

2. **Format Code**
   ```bash
   cargo fmt
   ```
   - Ensures consistent formatting

3. **Run Clippy**
   ```bash
   cargo clippy --all -- -D warnings
   ```
   - Must have zero warnings

## Adding a New Algorithm

1. Create `src/category_name/algorithm_name.rs`
2. Implement the algorithm with `pub` visibility
3. Add comprehensive tests in `#[cfg(test)] mod tests` block
4. Export in `src/category_name/mod.rs`:
   ```rust
   mod algorithm_name;
   pub use self::algorithm_name::function_name;
   ```
5. If creating a new category, add `pub mod category_name;` to `src/lib.rs`
6. Add entry to `DIRECTORY.md` with correct link
7. Run full CI check:
   ```bash
   cargo fmt && cargo clippy --all -- -D warnings && cargo test
   ```

## Pull Request Checklist

From the PR template:
- [ ] Ran `cargo clippy --all -- -D warnings` with latest rust nightly
- [ ] Ran `cargo fmt`
- [ ] Ran `cargo test` and all tests passed
- [ ] Added algorithm to corresponding `mod.rs` file
- [ ] Added algorithm to `DIRECTORY.md` with correct link
- [ ] Followed `CONTRIBUTING.md` guidelines
- [ ] Tests don't take > 300ms (use `#[ignore]` if needed)

## CI Pipeline

The GitHub Actions workflow checks:
1. `cargo fmt --all -- --check` (formatting)
2. `cargo clippy --all --all-targets -- -D warnings` (linting)
3. `cargo test` (testing)
