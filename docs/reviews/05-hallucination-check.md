# AI Hallucination Check

This section flags any technical terms, crate names, type names, line references, or architectural claims that appear potentially "invented" or inconsistent with the established naming conventions and actual codebase.

---

## Methodology

Each claim was verified against:
1. Actual source code at the referenced file and line number
2. `Cargo.toml` workspace dependency declarations
3. Workspace member list
4. Directory structure via filesystem inspection

---

## Verified Accurate (No Hallucinations)

| Claim | Document | Verification |
|-------|----------|-------------|
| `DenoSubcommand` at line 599 | Doc 03, 04 | ✅ `grep -n 'pub enum DenoSubcommand' cli/args/flags.rs` → line 599 |
| `Flags` struct at line 854 | Doc 04 | ✅ `grep -n 'pub struct Flags' cli/args/flags.rs` → line 854 |
| `PermissionFlags` at line ~910 | Doc 04 | ✅ Actual: line 907 (close enough, documented as approximation) |
| `MainWorker` at line 150 | Doc 04 | ✅ Exact match |
| `WorkerOptions` at line 221 | Doc 04 | ✅ Exact match |
| `BootstrapOptions` at line 93 | Doc 04 | ✅ Exact match |
| `Permissions` at line 3342 | Doc 04, 06 | ✅ Exact match |
| `PermissionsOptions` at line 3368 | Doc 04 | ✅ Exact match |
| `PermissionsContainer` at line 3771 | Doc 04 | ✅ Exact match |
| `PermissionState` at line 133 | Doc 06 | ✅ Exact match — 6 variants match |
| `common_extensions` at line 1039 | Doc 01 | ✅ Exact match |
| Rust edition 2024 | Doc 02 | ✅ Verified in `Cargo.toml` |
| Rust toolchain 1.92.0 | Doc 02 | ✅ Verified in `rust-toolchain.toml` |
| `deno_core` 0.385.0 | Doc 02 | ✅ Exact match in `Cargo.toml` |
| `deno_ast` 0.53.0 | Doc 02 | ✅ Exact match |
| `deno_graph` 0.107.0 | Doc 02 | ✅ Exact match |
| `tokio` 1.47.1 | Doc 02 | ✅ Exact match |
| `hyper` 1.6.0 | Doc 02 | ✅ Exact match |
| `rustls` 0.23.28 | Doc 02 | ✅ Exact match |
| `clap` 4.5.56 | Doc 02 | ✅ Exact match |
| `serde` 1.0.149 | Doc 02 | ✅ Exact match |
| `aws-lc-rs` 1.13.1 | Doc 06 | ✅ Exact match in `Cargo.toml` |
| `deno_native_certs` 0.3.0 | Doc 06 | ✅ Exact match |
| `deno_media_type` 0.4.0 | Doc 05 | ✅ Exact match |
| Extension registration order (30 items) | Doc 01 | ✅ Matches `runtime/worker.rs` lines 1051-1100 |
| Copyright header year range `2018-2026` | Doc 02 | ✅ Matches actual files |
| 41 spec test categories | Doc 07 | ✅ Verified: `find tests/specs -maxdepth 1 -type d` → 41 |

---

## Potential Hallucinations / Inaccuracies Found

### 1. `tower-lsp` Framework Name — **Minor Inaccuracy**

| Document | Claimed | Actual |
|----------|---------|--------|
| Doc 03 Section 4 | "tower-lsp framework" | `deno_tower_lsp` (package alias in Cargo.toml: `tower-lsp = { package = "deno_tower_lsp", version = "=0.4.3" }`) |
| Doc 05 Section 2 | "Tower-LSP for editor support" | Same fork |

**Verdict:** Not a hallucination — `tower-lsp` is the import name in Rust code (via the `package` field alias). But it's misleading because it's a Deno-maintained fork, not the upstream crate. The documentation should note this.

### 2. `deno_url` Version — **Data Omission**

| Document | Claimed | Actual |
|----------|---------|--------|
| Doc 02, Extension Crates table | `deno_url` — version listed as `(ext/url/)` | `0.222.0` (from `ext/url/Cargo.toml`) |

**Verdict:** Not a hallucination but a data-gathering failure by the documentation agent. The version exists and is easily retrievable.

### 3. `PermissionFlags` Line Reference — **Approximate Accuracy**

| Document | Claimed | Actual |
|----------|---------|--------|
| Doc 04 | "line ~910" | line 907 |

**Verdict:** Acceptable. The `~` prefix indicates approximation. Within 3 lines.

### 4. Missing `--ignore-*` State Transition in State Machine — **Incomplete Model**

Doc 06 Section 2.1 shows a state transition `PROMPT --> IGNORED : --ignore-<perm>=<specific>` but this is not reflected in the Allow/Deny interaction table (Section 2.4). This is not a hallucination but an incomplete model.

### 5. `deno_console` Listed in Directory Tree — **Accurate but Potentially Confusing**

Doc 02 lists `ext/console/` as "(deprecated, part of web)" in the directory tree. The crate exists (version 0.222.0, description "DEPRECATED: use `deno_web` instead") but is NOT in workspace members. This is technically accurate documentation but could lead a developer to believe the crate is still functional.

### 6. `PermissionPrompter` Trait Signature — **Verified Accurate**

Doc 06 Section 2.6 shows the `PermissionPrompter` trait at line 95 of `prompter.rs`. Verified: exact match.

---

## Crate Name Convention Verification

All crate names follow the established convention:

| Convention | Pattern | Examples Verified |
|-----------|---------|-------------------|
| Extension crates | `deno_<name>` | `deno_fetch`, `deno_crypto`, `deno_node`, `deno_webgpu` ✅ |
| Library crates | `deno_<name>` | `deno_config`, `deno_resolver`, `deno_npm_cache` ✅ |
| Node.js-specific libs | `node_<name>` | `node_resolver`, `node_shim` ✅ |
| NAPI symbol crate | `napi_sym` | ✅ Deviates from `deno_*` but is intentional (provides C ABI symbols) |
| Runtime helper crate | `denort_helper` | ✅ Follows `denort` naming for the standalone runtime binary |
| External fork | `deno_tower_lsp` | ✅ Follows `deno_*` convention for forked crates |
| Go client integration | `deno_typescript_go_client_rust` | ✅ Verbose but descriptive |

**No invented or hallucinated crate names were detected.**

---

## Summary

| Category | Count |
|----------|-------|
| Claims verified as fully accurate | 27 |
| Minor inaccuracies (naming, missing values) | 2 |
| Incomplete models (not wrong, but insufficient) | 2 |
| Hallucinated / invented content | **0** |

**Overall assessment: The documentation suite does not contain AI hallucinations.** All technical terms, crate names, line references, and architectural claims are grounded in the actual codebase. The issues found are data omissions and incomplete coverage, not fabrications.
