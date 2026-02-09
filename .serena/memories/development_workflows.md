# Development Workflows & Design Patterns

## Adding a New CLI Subcommand
1. Define command structure in `cli/args/flags.rs`
2. Add handler in `cli/tools/<command_name>.rs` or `cli/tools/<command_name>/mod.rs`
3. Wire it up in `cli/main.rs`
4. Add spec tests in `tests/specs/<command_name>/`

## Adding/Modifying an Extension
1. Navigate to `ext/<name>/`
2. Rust code provides ops (operations) exposed to JavaScript
3. JavaScript code provides higher-level APIs
4. Register in `runtime/worker.rs` if new extension
5. Add tests in the extension's directory

## Adding a Spec Test
1. Create directory in `tests/specs/` with descriptive name
2. Add `__test__.jsonc` (see `tests/specs/schema.json` for schema)
3. Add input files and `.out` files for expected output
4. Run with: `cargo test specs`

## Key Design Patterns
- **Ops pattern**: Rust → JS bridge functions defined in ext/ crates
- **Extension model**: Groups of ops + JS glue code, registered in runtime
- **Worker model**: Main Worker + Web Workers = isolated JS execution contexts
- **Resource table**: Objects (files, sockets) tracked between Rust/JS via resource IDs
- **Factory pattern**: `cli/factory.rs` creates runtime components on demand
- **Permission system**: All access gated by explicit permissions (--allow-*)
- **Module graph**: Modules tracked as a graph for dependency resolution
- **Snapshot**: V8 snapshots used for fast startup

## Architecture Flow
```
CLI (cli/main.rs)
  → Flag parsing (cli/args/flags.rs)
  → Factory (cli/factory.rs) creates runtime services
  → Module loader (cli/module_loader.rs) loads user code
  → Runtime (runtime/worker.rs) executes JS
    → Extensions (ext/*) provide native ops
      → V8 (via deno_core) runs JavaScript
```

## LSP Integration
- Full Language Server Protocol implementation in `cli/lsp/`
- Supports VS Code and other LSP-compatible editors
- Provides diagnostics, completion, hover, etc.

## NPM Compatibility
- NPM packages supported via `ext/node/` and `libs/npm_cache/`, `libs/npm_installer/`
- Node.js module resolution in `libs/node_resolver/`
- `package.json` support via `libs/package_json/`

## Nix Dev Shell
- `flake.nix` provides reproducible dev environment
- Pre-fetches rusty_v8 binary
- Requires: protobuf, cmake, glib, pkg-config, etc.
