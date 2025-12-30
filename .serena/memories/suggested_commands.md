# Suggested Commands for Development

## Essential Commands (Run before submitting PR)

### Run All Tests
```bash
cargo test
```

### Format Code
```bash
cargo fmt
```

### Run Linter (Clippy)
```bash
cargo clippy --all -- -D warnings
```

### Full CI Check (all three)
```bash
cargo fmt --all -- --check && cargo clippy --all --all-targets -- -D warnings && cargo test
```

## Build Commands

### Build (Debug)
```bash
cargo build
```

### Build (Release)
```bash
cargo build --release
```

### Build with all features
```bash
cargo build --all-features
```

## Testing Commands

### Run specific test
```bash
cargo test test_name
```

### Run tests for a specific module
```bash
cargo test sorting::
```

### Run tests with output
```bash
cargo test -- --nocapture
```

### Run ignored tests
```bash
cargo test -- --ignored
```

## Documentation

### Generate and open documentation
```bash
cargo doc --open
```

## Utility Commands (Linux)

### Find files
```bash
find . -name "*.rs" | head -20
```

### Search in files
```bash
grep -r "pattern" src/
```

### Git operations
```bash
git status
git diff
git add -A
git commit -m "message"
git push
```
