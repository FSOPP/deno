# Codebase Structure

## Top-Level Layout
```
cli/        — Main CLI binary ("deno" crate). Entry point, subcommands, tools, LSP, module loading.
runtime/    — JavaScript runtime assembly (deno_runtime crate). Workers, permissions, snapshots.
ext/        — Extensions providing native ops to JS (fs, net, crypto, http, etc.)
libs/       — Shared workspace libraries (node_resolver, npm_cache, config, etc.)
tests/      — All test types (specs, unit, integration, WPT, node_compat)
tools/      — Dev scripts: lint.js, format.js, build helpers
```

## cli/ (Main Binary)
- `main.rs` — Entry point, module declarations
- `args/flags.rs` — CLI flag parsing and structure
- `args/mod.rs` — CLI argument configuration, resolution
- `factory.rs` — Service factory for creating runtime components
- `module_loader.rs` — Module loading and resolution
- `resolver.rs` — Module and import resolution
- `file_fetcher.rs` — Fetching remote/local files
- `graph_util.rs` — Module graph utilities
- `graph_container.rs` — Module graph container management
- `type_checker.rs` — TypeScript type checking integration
- `worker.rs` — CLI worker setup
- `npm.rs` — NPM package support
- `node.rs` — Node.js compatibility layer
- `jsr.rs` — JSR (JavaScript Registry) integration
- `http_util.rs` — HTTP client utilities
- `task_runner.rs` — Task runner implementation
- **tools/** — CLI subcommands: fmt, lint, test, bench, doc, compile, publish, etc.
  - `tools/fmt.rs` — Format command
  - `tools/test/` — Test runner (directory)
  - `tools/lint/` — Lint tool (directory)
  - `tools/bench/` — Benchmark tool (directory)
  - `tools/doc.rs` — Doc generation
  - `tools/compile.rs` — Compile to standalone binary
  - `tools/publish/` — Package publishing
  - `tools/pm/` — Package management
  - `tools/jupyter/` — Jupyter kernel
  - `tools/repl/` — REPL
  - `tools/run/` — Run subcommand
  - `tools/serve.rs` — Serve command
  - `tools/upgrade.rs` — Upgrade command
  - `tools/init/` — Init project scaffold
  - `tools/installer/` — Script installer
  - `tools/coverage/` — Coverage reporting
  - `tools/info.rs` — Info/diagnostic command
  - `tools/check.rs` — Type check command
  - `tools/clean.rs` — Cache clean
  - `tools/deploy.rs` — Deploy command
  - `tools/bundle/` — Bundle tool
- **lsp/** — Language Server Protocol implementation
- **cache/** — Caching subsystem
- **ops/** — CLI-specific ops
- **standalone/** — Standalone binary compilation support
- **tsc/** — TypeScript compiler integration
- **util/** — Shared CLI utilities
- **js/** — CLI JavaScript runtime code
- **lib/** — CLI library crate (deno_lib)
- **rt/** — CLI runtime helpers
- **snapshot/** — Snapshot generation (deno_snapshots)
- **schemas/** — JSON schemas
- **bench/** — Benchmarks

## runtime/ (deno_runtime crate)
- `lib.rs` — Runtime library root
- `worker.rs` — Main worker initialization (registers all extensions)
- `web_worker.rs` — Web Worker support
- `permissions.rs` — Permission system
- `worker_bootstrap.rs` — Worker bootstrap data
- `js.rs` — JavaScript runtime code
- `snapshot.rs` — V8 snapshot generation
- `snapshot_info.rs` — Snapshot metadata
- `fmt_errors.rs` — Error formatting
- `transpile.rs` — Transpilation
- `code_cache.rs` — Code caching
- `coverage.rs` — Code coverage
- `tokio_util.rs` — Tokio utilities
- `shared.rs` — Shared types
- **ops/** — Runtime ops
- **js/** — Runtime JavaScript code
- **permissions/** — Permissions crate
- **features/** — Feature flags crate
- **subprocess_windows/** — Windows subprocess handling

## ext/ (Extensions)
Each provides native ops exposed to JavaScript:
- `broadcast_channel/`, `bundle/`, `cache/`, `console/`
- `cron/`, `crypto/`, `fetch/`, `ffi/`
- `fs/`, `http/`, `image/`, `io/`
- `kv/`, `napi/`, `net/`, `node/`
- `os/`, `process/`, `rt_helper/`, `signals/`
- `telemetry/`, `tls/`, `url/`, `web/`
- `webgpu/`, `webidl/`, `websocket/`, `webstorage/`

## libs/ (Shared Libraries)
- `config/` — Deno configuration (deno.json) parsing
- `crypto/` — Crypto provider
- `inspector_server/` — V8 inspector server
- `maybe_sync/` — Conditional Send/Sync wrappers
- `node_resolver/` — Node.js module resolution
- `node_shim/` — Node.js shim
- `npm_cache/` — NPM package cache
- `npm_installer/` — NPM package installer
- `package_json/` — package.json parsing
- `resolver/` — Module resolution (deno_resolver)
- `typescript_go_client/` — TypeScript Go client integration

## tests/
- `specs/` — Main spec (integration) tests with `__test__.jsonc`
- `unit/` — Deno unit tests (JS)
- `unit_node/` — Node.js compatibility unit tests
- `integration/` — Rust integration tests
- `testdata/` — Test fixtures
- `node_compat/` — Node.js compatibility tests
- `wpt/` — Web Platform Tests
- `napi/` — N-API tests
- `ffi/` — FFI tests
- `registry/` — Registry mock tests
- `bench_util/` — Benchmark utilities
