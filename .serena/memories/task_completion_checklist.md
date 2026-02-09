# Task Completion Checklist

When a coding task is completed, the following steps should be taken:

## 1. Compilation Check
```bash
cargo check
```
Or for a specific crate:
```bash
cargo check -p <crate_name>
```

## 2. Formatting
```bash
./tools/format.js
```

## 3. Linting
```bash
# Full lint (Rust clippy + JS dlint + copyright check)
./tools/lint.js

# Or separately:
./tools/lint.js --rs    # Rust only
./tools/lint.js --js    # JS/TS only
```

## 4. Testing
Run relevant tests depending on what was changed:
```bash
# If changed specific crate
cargo test -p <crate_name>

# If changed CLI tool
cargo test --bin deno

# If changed/added spec tests
cargo test specs

# Specific test
cargo test <test_name>
```

## 5. Verify Clippy Rules
- cli crate: Make sure not to use disallowed methods/types (see cli/clippy.toml)
- runtime crate: Use FileSystem/NodeFs traits instead of std::fs (see runtime/clippy.toml)

## 6. Copyright Headers
All new files must include:
```
// Copyright 2018-2026 the Deno authors. MIT license.
```

## Quick Combined Check
```bash
cargo check && ./tools/format.js --check && ./tools/lint.js
```
