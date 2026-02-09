# Deno Project Overview

## Purpose
Deno is a JavaScript, TypeScript, and WebAssembly runtime with secure defaults.
Built on V8 (JavaScript engine), Rust, and Tokio (async runtime).
Current version: **2.6.8** (as of the codebase state).

## Tech Stack
- **Rust** (edition 2024, toolchain 1.92.0) — core runtime, CLI, extensions
- **TypeScript/JavaScript** — runtime JS APIs, development tooling scripts
- **V8** — JavaScript execution engine (via `rusty_v8` / `deno_core`)
- **Tokio** — async runtime for Rust
- **Cargo workspace** — manages ~50+ crates
- **dprint** — code formatting (TypeScript/JS/JSON/Markdown)
- **dlint** — JavaScript/TypeScript linting
- **clippy** — Rust linting with custom disallowed methods

## License
MIT License. Copyright 2018-2026 the Deno authors.

## Repository
https://github.com/denoland/deno (default branch: `main`)
