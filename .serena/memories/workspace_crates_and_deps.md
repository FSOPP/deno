# Key Workspace Crates & Dependencies

## Workspace Members (Cargo workspace)
The workspace contains ~50+ crates. Key top-level crates:

| Crate | Path | Purpose |
|-------|------|---------|
| `deno` | `cli/` | Main CLI binary (v2.6.8) |
| `deno_runtime` | `runtime/` | JS runtime assembly |
| `deno_lib` | `cli/lib/` | CLI library |
| `deno_snapshots` | `cli/snapshot/` | V8 snapshot generation |
| `deno_permissions` | `runtime/permissions/` | Permission system |
| `deno_features` | `runtime/features/` | Feature flags |

## Extension Crates (ext/)
Each `deno_*` crate provides ops (native functions) to JavaScript:
- `deno_web`, `deno_fetch`, `deno_net`, `deno_http` — Web APIs
- `deno_fs`, `deno_io` — File system & I/O
- `deno_crypto` — Cryptography
- `deno_node` — Node.js compatibility
- `deno_napi` — N-API compatibility
- `deno_ffi` — Foreign Function Interface
- `deno_kv` — Key-Value store
- `deno_cache` — Cache API
- `deno_websocket`, `deno_webstorage` — WebSocket/WebStorage
- `deno_tls` — TLS support
- `deno_url` — URL parsing
- `deno_webidl` — WebIDL bindings
- `deno_webgpu` — WebGPU support
- `deno_image` — Image processing
- `deno_cron` — Cron scheduling
- `deno_telemetry` — OpenTelemetry
- `deno_signals` — Signal handling
- `deno_process` — Process management
- `deno_os` — OS info

## Library Crates (libs/)
| Crate | Path | Purpose |
|-------|------|---------|
| `deno_config` | `libs/config/` | deno.json parsing |
| `node_resolver` | `libs/node_resolver/` | Node module resolution |
| `deno_npm_cache` | `libs/npm_cache/` | NPM cache management |
| `deno_npm_installer` | `libs/npm_installer/` | NPM installer |
| `deno_resolver` | `libs/resolver/` | Module resolution |
| `deno_package_json` | `libs/package_json/` | package.json parsing |
| `deno_crypto_provider` | `libs/crypto/` | Crypto provider |
| `deno_inspector_server` | `libs/inspector_server/` | V8 inspector |
| `deno_maybe_sync` | `libs/maybe_sync/` | Send/Sync wrappers |
| `node_shim` | `libs/node_shim/` | Node.js shim |
| `deno_typescript_go_client_rust` | `libs/typescript_go_client/` | TS Go client |

## Key External Dependencies
- `deno_core` 0.385.0 — Core V8 bindings and op framework
- `deno_ast` 0.53.0 — AST parsing/transpiling
- `deno_graph` 0.107.0 — Module graph
- `tokio` 1.47.1 — Async runtime
- `hyper` 1.6.0 — HTTP
- `rustls` 0.23.28 — TLS
- `rusqlite` 0.37.0 — SQLite
- `serde` 1.0.149 — Serialization
- `clap` 4.5.56 — CLI argument parsing
- `reqwest` 0.12.5 — HTTP client

## Build Profiles
- `dev` — Default debug (with select packages at higher opt-level for perf)
- `release` — Full optimization, LTO, opt-level 'z' (size), codegen-units=1
- `release-lite` — Faster compile, thin LTO, codegen-units=128
- `release-with-debug` — Release-lite + debug symbols
