# Codebase Structure

## Root Files
- `Cargo.toml` - Project configuration and dependencies
- `clippy.toml` - Clippy-specific configuration (allows duplicate crate: glam)
- `README.md` - Project introduction and links
- `CONTRIBUTING.md` - Contribution guidelines
- `DIRECTORY.md` - Algorithm directory with links (auto-generated)
- `LICENSE` - Project license

## Source Directory (`src/`)

### Main Entry Point
- `src/lib.rs` - Library root, exports all algorithm modules

### Algorithm Modules (21 categories)

| Module | Description | Example Algorithms |
|--------|-------------|-------------------|
| `backtracking/` | Backtracking algorithms | N-Queens, Sudoku, Hamiltonian Cycle |
| `big_integer/` | Big number operations | Fast factorial, Poly1305 |
| `bit_manipulation/` | Bitwise operations | Gray code, Two's complement |
| `ciphers/` | Cryptography | AES, RSA, SHA256, ChaCha |
| `compression/` | Data compression | Burrows-Wheeler, RLE |
| `conversions/` | Type/base conversions | Binary/Hex/Octal conversions |
| `data_structures/` | Data structures | AVL Tree, B-Tree, Graph, Heap |
| `dynamic_programming/` | DP algorithms | (various) |
| `financial/` | Financial algorithms | (various) |
| `general/` | General algorithms | (various) |
| `geometry/` | Geometry algorithms | (various) |
| `graph/` | Graph algorithms | (various) |
| `greedy/` | Greedy algorithms | (various) |
| `machine_learning/` | ML algorithms | (various) |
| `math/` | Mathematical algorithms | (various) |
| `navigation/` | Navigation algorithms | (various) |
| `number_theory/` | Number theory | (various) |
| `searching/` | Search algorithms | (various) |
| `signal_analysis/` | Signal processing | (various) |
| `sorting/` | Sorting algorithms | 30+ algorithms |
| `string/` | String algorithms | (various) |

### Module Structure Pattern
Each category follows this pattern:
```
category/
  mod.rs           # Module declarations and re-exports
  algorithm1.rs    # Algorithm implementation + tests
  algorithm2.rs    # Algorithm implementation + tests
  ...
```

## GitHub Configuration (`.github/`)

### Workflows
- `build.yml` - Main CI: fmt, clippy, test
- `code_ql.yml` - CodeQL security analysis
- `directory_workflow.yml` - Auto-update DIRECTORY.md
- `stale.yml` - Stale issue/PR management
- `upload_coverage_report.yml` - Code coverage

### Other
- `pull_request_template.md` - PR template with checklist
- `CODEOWNERS` - Code ownership rules
- `dependabot.yml` - Dependency updates

## Git Hooks (`git_hooks/`)
- `pre-commit` - Pre-commit hook script
