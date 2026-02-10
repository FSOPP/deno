# Cross-Document Inconsistencies

## 1. Subcommand Count Mismatch (Doc 03 vs. Doc 04)

| Document | Claim | Actual |
|----------|-------|--------|
| **Doc 03** (API Contracts, Section 2.1) | Lists **31 subcommands** in the reference table | `DenoSubcommand` has **36 variants** |
| **Doc 04** (Data Models, Section 2.2) | Lists all **36 variants** correctly | Matches source at `cli/args/flags.rs:599` |

**Missing from Doc 03:**
- `Task(TaskFlags)` — the `deno task` runner
- `Vendor` — the `deno vendor` command
- `ApproveScripts(ApproveScriptsFlags)` — lifecycle script approval
- `JSONReference(JSONReferenceFlags)` — JSON reference output
- `Help(HelpFlags)` — help display variant

**Impact:** A developer using Doc 03 as their CLI reference would miss 5 subcommands. `deno task` is one of the most commonly used subcommands, making this a significant omission.

---

## 2. Permission Flags Incomplete (Doc 03 vs. Doc 04 vs. Doc 06)

| Document | `--ignore-*` flags documented? | `Ignored` permission state? |
|----------|-------------------------------|---------------------------|
| **Doc 03** (Section 2.2, Permission Flags) | **NO** — not listed at all | N/A |
| **Doc 04** (Section 3.5, PermissionsOptions) | **YES** — `ignore_env`, `ignore_read` fields listed | N/A |
| **Doc 06** (Section 2.1, State Machine) | Yes (in diagram) | **YES** — `Ignored` state shown |
| **Actual code** (`cli/args/flags.rs:4613-4616`) | `--ignore-env`, `--ignore-read` exist | `PermissionState::Ignored = 5` at line 133 |

**Impact:** A security auditor reading Doc 03 (the primary API reference) would completely miss the `--ignore-*` permission mechanism, which fundamentally changes the permission model by allowing operations to return fake "not found" responses instead of denying them.

---

## 3. LSP Framework Naming (Doc 03 vs. Actual)

| Document | Claim | Actual |
|----------|-------|--------|
| **Doc 03** (Section 4) | Uses "tower-lsp" framework | `deno_tower_lsp` (version `=0.4.3`, a Deno-maintained fork) |
| **Doc 05** (Section 2, Key Design Decisions) | "Tower-LSP for editor support" | Same fork — `deno_tower_lsp` |

**Impact:** Low. The fork is API-compatible, but someone trying to find the crate on crates.io would find a different crate. The documentation should note this is a fork package (`package = "deno_tower_lsp"`).

---

## 4. Extension Registration vs. Crate Dependency Graph (Doc 01 Section 3.2 vs. Section 4)

Doc 01 Section 3.2 lists the extension registration order as 30 entries. Doc 01 Section 4 (Crate Dependency Graph) shows `deno_runtime` depending on all extension crates. These are **consistent**.

However, the extension registration in Section 3.2 includes runtime-internal ops:
- `ops::runtime::deno_runtime` (23. Runtime Ops)
- `ops::worker_host::deno_worker_host` (24. Worker Host)
- `ops::fs_events::deno_fs_events` (25. FS Events)
- `ops::permissions::deno_permissions` (26. Permissions)
- `ops::tty::deno_tty` (27. TTY)
- `ops::http::deno_http_runtime` (28. HTTP Runtime)
- `ops::bootstrap::deno_bootstrap` (30. Bootstrap)

These are NOT separate crates — they are op modules within `deno_runtime` itself. The Crate Dependency Graph correctly does NOT show them as separate nodes. But the extension list's numbering (calling them extensions alongside actual crate-backed extensions) blurs the architectural boundary.

**Impact:** Medium. A contributor could confuse runtime-internal op modules with actual extension crates.

---

## 5. Doc 02 Crate Inventory vs. Workspace Members

| Crate | In Workspace `members`? | In Doc 02 Inventory? | Notes |
|-------|------------------------|---------------------|-------|
| `denort` (`cli/rt/`) | **YES** | **NO** | Binary crate for the standalone runtime |
| `deno_broadcast_channel` | No | No (listed in directory tree only) | Correctly omitted but confusing |
| `deno_console` | No | Listed in directory tree as deprecated | Correctly marked |
| `deno_url` | Yes | Yes, but **missing version** | Version is `0.222.0` |
| `tests/sqlite_extension_test` | Yes | No | Test crate, reasonable to omit |
| `tests/util/server` | Yes | No | Test infrastructure crate |
| `tests/util/test_macro` | Yes | No | Test infrastructure crate |

**Impact:** The `denort` omission is the most significant — it's a user-facing binary that ships with Deno and should be documented.

---

## 6. Module Resolution in LSP vs. CLI (Doc 01 Section 5 vs. Doc 05 Section 6)

Doc 05 Section 6 documents the Module Resolution Strategy with 8 specifier schemes. Doc 01 Section 5 (LSP Architecture) shows an LSP `resolver.rs` module. These are architecturally separate:

- **CLI resolution**: `libs/resolver/` + `libs/node_resolver/` + `cli/module_loader.rs`
- **LSP resolution**: `cli/lsp/resolver.rs` (wraps CLI resolution but adds document-relative logic)

Neither Doc 01 nor Doc 05 explicitly states that the LSP may resolve modules differently from the CLI (e.g., for unsaved/virtual documents). This could lead to "works in editor but fails at runtime" bugs.

**Impact:** Medium. Should be documented as an architectural decision point.

---

## 7. Supply Chain Security (Doc 06 Section 6 vs. Doc 02 Section 7)

Doc 06 Section 6 covers supply chain security features (lockfile, `--cached-only`, etc.) but makes no mention of the Clippy-enforced patterns documented in Doc 02 Section 7. These are complementary security layers:
- Doc 02's clippy rules prevent **the Deno codebase itself** from bypassing the sandbox
- Doc 06's supply chain features protect **user applications** from dependency attacks

Neither document cross-references the other.

**Impact:** Low-Medium. A security auditor would need to read both documents to get the full picture.

---

## 8. Test Category Mismatch (Doc 07 vs. Actual)

Doc 07 Section 3.2 states "41 categories" and lists them. The actual filesystem has 41 directories, but:

| Category | In Doc 07 List? | In Filesystem? |
|----------|----------------|---------------|
| `future_install_node_modules` | **NO** | **YES** |
| `remove` | **NO** (only mentioned as "add / remove") | **YES** (separate directory) |
| `future` | Yes | Yes |

**Impact:** Low. The `future_install_node_modules` category is a specialized testing category that may be transient.
