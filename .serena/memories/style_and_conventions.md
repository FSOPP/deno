# Code Style & Conventions

## Rust

### General
- **Edition**: 2024
- **Toolchain**: Rust 1.92.0 (see `rust-toolchain.toml`)
- **Copyright header**: Every file starts with `// Copyright 2018-2026 the Deno authors. MIT license.`
- **Module system**: `mod` declarations in `main.rs`/`lib.rs`, one module per file or directory
- **Error handling**: Use `anyhow` for general errors, `thiserror` for typed errors, `deno_error` for Deno-specific
- **Async**: Tokio-based, use `async/await`. All I/O is async.
- **Formatting**: `rustfmt` (included in toolchain)
- **Linting**: `clippy` with custom disallowed methods/types per crate

### Clippy Rules (cli/clippy.toml)
- Do NOT use `reqwest::Client::new` — use `HttpClient` via `HttpClientProvider`
- Do NOT use `std::process::exit` — use `deno_runtime::exit`
- Do NOT use `clap::Arg::env` — resolve env vars after loading `.env`
- Do NOT use `tokio::signal::*` — use `deno_signals` crate instead
- Do NOT use `reqwest::Client` type — use `crate::http_util::HttpClient`
- Do NOT use `sys_traits::impls::RealSys` — use `crate::sys::CliSys`

### Clippy Rules (runtime/clippy.toml)
- Do NOT use `std::fs::*`, `std::path::Path::*` filesystem ops — use `FileSystem` or `NodeFs` trait
- Do NOT use `std::env::current_dir`, `set_current_dir`, `temp_dir` — use `FileSystem` trait
- Do NOT use `std::process::exit` — use `deno_runtime::exit`

### Naming
- Snake_case for functions and variables
- CamelCase for types and traits
- SCREAMING_SNAKE_CASE for constants
- Module names match file names

### Patterns
- **Ops**: Rust functions exposed to JavaScript (in `ext/` dirs), registered via `deno_core` macros
- **Extensions**: Collections of ops + JS code providing functionality
- **Workers**: JavaScript execution contexts (main worker, web workers)
- **Resources**: Managed objects passed between Rust and JS (files, sockets, etc.)
- **Factory pattern**: `cli/factory.rs` creates runtime components

## TypeScript/JavaScript

### General
- **Formatting**: dprint (via `./tools/format.js`, uses `.dprint.json` config)
- **Linting**: dlint (via `./tools/lint.js`, uses `.dlint.json` config)
- **Copyright header**: `// Copyright 2018-2026 the Deno authors. MIT license.`
- Test scripts run in Deno with `--allow-all --config=tests/config/deno.json`

## Spec Test Conventions
- Located in `tests/specs/`
- Each test: a directory with `__test__.jsonc` file
- Schema: `tests/specs/schema.json`
- Output assertions use: `[WILDCARD]`, `[WILDLINE]`, `[WILDCHAR]`, `[WILDCHARS(n)]`, `[UNORDERED_START]`/`[UNORDERED_END]`
- Expected output in `.out` files or inline in `__test__.jsonc`
