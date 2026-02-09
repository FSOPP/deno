# Suggested Commands

## System: Linux
Standard unix commands are available: `git`, `ls`, `cd`, `grep`, `find`, `cat`, `head`, `tail`, etc.

## Building

```bash
# Quick check for compilation errors (no binary)
cargo check

# Check a specific crate
cargo check -p deno_runtime

# Build debug binary
cargo build
# or faster (only the main binary):
cargo build --bin deno

# Build release (slow, optimized, LTO)
cargo build --release

# Build release-lite (faster compile, similar perf)
cargo build --profile=release-lite

# Run your dev build
./target/debug/deno eval 'console.log("Hello")'
./target/debug/deno run path/to/file.ts
./target/debug/deno run --allow-net --allow-read script.ts
```

## Testing

```bash
# Run all tests (slow!)
cargo test

# Filter by test name
cargo test <nameOfTest>

# Run tests for a specific crate
cargo test -p deno_core

# Run CLI integration tests only
cargo test --bin deno

# Run spec tests only
cargo test specs

# Run a specific spec test
cargo test spec::test_name
```

## Code Quality

```bash
# Format code (uses dprint)
./tools/format.js

# Check formatting only
./tools/format.js --check

# Lint code (runs clippy for Rust + dlint for JS/TS)
./tools/lint.js

# Lint Rust only
./tools/lint.js --rs

# Lint JS/TS only
./tools/lint.js --js

# Combined format + lint
./tools/format.js && ./tools/lint.js
```

## Debugging

```bash
# Verbose logging
DENO_LOG=debug ./target/debug/deno run script.ts
DENO_LOG=deno_core=debug ./target/debug/deno run script.ts

# Full backtrace on crash
RUST_BACKTRACE=1 ./target/debug/deno run script.ts
RUST_BACKTRACE=full ./target/debug/deno run script.ts

# V8 inspector
./target/debug/deno run --inspect-brk script.ts

# LLDB debugging
lldb ./target/debug/deno
```

## Dependencies

```bash
cargo update              # Update Cargo deps
cargo upgrade             # Upgrade to latest compatible (needs cargo-edit)
cargo outdated            # Check for outdated deps
```

## Heap Profiling

```bash
# Build with dhat profiling
cargo build --profile=release-with-debug --features=dhat-heap
# Run the executable — produces dhat-heap.json
# Open at https://nnethercote.github.io/dh_view/dh_view.html
```

## Git

```bash
git status
git diff
git log --oneline -20
git branch
```
