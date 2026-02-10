# Deno Technical Documentation

Comprehensive architecture and design documentation for the [Deno](https://github.com/denoland/deno) runtime — a secure JavaScript, TypeScript, and WebAssembly runtime built on V8, Rust, and Tokio.

**Version:** 2.6.8  
**Rust Toolchain:** 1.92.0 (edition 2024)  
**Generated:** February 2026

---

## Table of Contents

| Document | Description |
|----------|-------------|
| [Architecture](01-architecture.md) | High-Level & Low-Level Design with component and sequence diagrams |
| [Codebase Structure & Standards](02-codebase-structure.md) | Project layout, naming conventions, dependency landscape |
| [API Contracts](03-api-contracts.md) | CLI interface, runtime ops, LSP protocol, FFI/NAPI boundaries |
| [Data Models & Schemas](04-data-models.md) | Core types, relationships, serialization boundaries |
| [Business Logic & Context](05-business-logic.md) | Design decisions, patterns, feature flags, technical debt |
| [Security & Compliance](06-security.md) | Permission model, cryptography, TLS, sandboxing |
| [Testing & Validation](07-testing.md) | Test suites, patterns, templates, coverage map |

---

## Quick Reference

### Build Commands

```bash
cargo build --bin deno          # Debug build
cargo build --release            # Optimized release build
cargo build --profile=release-lite  # Faster compile, similar perf
```

### Test Commands

```bash
cargo test                       # All Rust tests
cargo test -p <crate>            # Single crate
cargo test specs                 # Spec (integration) tests
```

### Code Quality

```bash
./tools/format.js               # Format (dprint)
./tools/lint.js                  # Lint (clippy + dlint)
```

---

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────┐
│                   CLI (cli/)                         │
│  Flag parsing → Factory → Module Loader → Worker    │
├─────────────────────────────────────────────────────┤
│               Runtime (runtime/)                     │
│    Main Worker │ Web Worker │ Permissions │ Snapshot │
├─────────────────────────────────────────────────────┤
│             Extensions (ext/)                        │
│  web │ fetch │ net │ fs │ crypto │ node │ ...        │
├─────────────────────────────────────────────────────┤
│          Shared Libraries (libs/)                    │
│  config │ resolver │ npm_cache │ node_resolver │ ... │
├─────────────────────────────────────────────────────┤
│              External Crates                         │
│  deno_core (V8) │ tokio │ hyper │ rustls │ ...      │
└─────────────────────────────────────────────────────┘
```
