# Detailed Findings by Module

## Module 01 — Architecture

| Aspect | Assessment |
|--------|-----------|
| **Strengths** | Excellent plantUML diagrams covering system context, component architecture, execution flow, extension registration, module resolution, worker architecture, LSP architecture, and crate dependency graph. Line references (e.g., `common_extensions` at line 1039 in `runtime/worker.rs`) are **verified accurate**. Extension registration order in Section 3.2 matches actual code exactly. |
| **Weaknesses / Gaps** | (1) Missing **error-handling and panic recovery** paths in the execution flow — what happens when V8 panics, when module resolution fails, or when the event loop encounters an unhandled rejection? (2) Worker Architecture (Section 3.4) states "Main worker has full permissions" but doesn't describe permission **delegation** to Web Workers — this is a security-relevant design decision. (3) The Crate Dependency Graph (Section 4) does not show `deno_core`'s dependency on individual ext crates beyond `web`, `fetch`, `net`, `fs` — the actual `deno_core` relationship is only to the runtime, not directly to extensions. |
| **Recommendation** | Add an "Error Propagation" section showing: (a) V8 exception → Rust Result → CLI exit code flow, (b) permission denial error propagation, (c) unhandled rejection behavior. Add web worker permission inheritance model to Section 3.4. |

---

## Module 02 — Codebase Structure

| Aspect | Assessment |
|--------|-----------|
| **Strengths** | Comprehensive crate inventory with verified version numbers. Clippy enforcement rules are well-documented and match actual `clippy.toml` files. Build profile documentation is accurate. |
| **Weaknesses / Gaps** | (1) **Missing crate: `denort`** (`cli/rt/`, version 2.6.8) — a workspace member binary crate not listed in Section 5 CLI Crates. (2) **Missing version for `deno_url`** — listed as `(ext/url/)` instead of `0.222.0`. (3) Directory tree in Section 1 lists `ext/console/` as "(deprecated, part of web)" but does not note it is **excluded from workspace members**. (4) Section 5 omits `deno_broadcast_channel` (`ext/broadcast_channel/`, version 0.216.0) — while not a workspace member, it's an existing crate in the directory tree that could confuse developers. (5) Missing `deno_console` (0.222.0) — exists as a crate but excluded from workspace, creating potential confusion. (6) Missing `sqlite_extension_test` (workspace member at `tests/sqlite_extension_test/`). |
| **Recommendation** | (a) Add `denort` to the CLI Crates table. (b) Fill in `deno_url` version. (c) Add a "Non-Member / Deprecated Crates" subsection to explicitly list `ext/console` and `ext/broadcast_channel` as excluded/orphaned. (d) Document the `cli/rt/` (denort) binary purpose. |

---

## Module 03 — API Contracts & Interfaces

| Aspect | Assessment |
|--------|-----------|
| **Strengths** | Strong subcommand reference table with 31 subcommands mapped to handler locations. Permission flag table is well-structured. Op pattern example with `#[op2]` is realistic and educational. LSP method table with line-number references is valuable. FFI type table is complete. |
| **Weaknesses / Gaps** | (1) **Missing subcommands**: `Task(TaskFlags)`, `ApproveScripts(ApproveScriptsFlags)`, `JSONReference(JSONReferenceFlags)`, `Help(HelpFlags)`, `Vendor` are in the actual enum but NOT in the subcommand reference table. Doc lists 31 but the actual enum has 36 variants. (2) **Missing `--ignore-*` flags**: `--ignore-read` and `--ignore-env` are real CLI flags (verified in `cli/args/flags.rs:4613-4616`) but completely absent from Section 2.2 (Permission Flags). (3) **FFI section lacks security context**: No mention that FFI operations require `--allow-ffi` permission, no discussion of memory safety risks, no mention of `op_ffi_unsafe_callback_*` ops. (4) **NAPI section lacks serialization cost analysis**: The architecture diagram is clean, but there's no discussion of the performance overhead of napi_* function calls, V8 handle management, or GC interaction. (5) **Missing `--allow-import` / `--deny-import`** flag clarification — it's listed in Section 2.2 but not connected to how it interacts with the Module Resolution in Doc 05. |
| **Recommendation** | (a) Add remaining 5 subcommands to the reference table. (b) Add `--ignore-read` and `--ignore-env` to Section 2.2. (c) Add a "Safety Considerations" subsection to both FFI and NAPI sections covering memory safety, serialization costs, and permission requirements. (d) Cross-reference `--allow-import` with module resolution schemes. |

---

## Module 04 — Data Models & Schemas

| Aspect | Assessment |
|--------|-----------|
| **Strengths** | All struct field tables are verified accurate against source: `Flags` at line 854, `DenoSubcommand` at line 599, `MainWorker` at line 150, `WorkerOptions` at line 221, `BootstrapOptions` at line 93, `Permissions` at line 3342, `PermissionsOptions` at line 3368, `PermissionsContainer` at line 3771 — **all line numbers match**. Relationship diagram correctly shows the data flow from `Flags` → `Factory` → `WorkerOptions` → `MainWorker`. Serialization boundaries table is informative. |
| **Weaknesses / Gaps** | (1) `DenoSubcommand` tree in Section 2.2 correctly lists all 36 variants, but this **contradicts** Doc 03 which lists only 31. The documentation itself is internally inconsistent. (2) `PermissionsOptions` field table (Section 3.5) lists `ignore_env` and `ignore_read` fields, but Doc 03 omits the corresponding CLI flags. (3) `Permissions` struct fields match exactly, but the `UnaryPermission` type is not explained — its internal structure (allow list, deny list, prompt flag) would be valuable. (4) Missing `WorkerExecutionMode` enum values — referenced in `BootstrapOptions` but not expanded. |
| **Recommendation** | (a) Reconcile subcommand count with Doc 03. (b) Expand `UnaryPermission<T>` internal structure. (c) Document `WorkerExecutionMode` variants. |

---

## Module 05 — Business Logic & Context

| Aspect | Assessment |
|--------|-----------|
| **Strengths** | Excellent design decision table with clear rationale. Design patterns section is well-mapped to code locations. Unstable feature flags table is comprehensive and matches `runtime/features/data.rs`. Module resolution strategy table covers all 8 specifier schemes. Environment variable documentation is thorough. |
| **Weaknesses / Gaps** | (1) **Cargo Feature Flags table** lists `module_specifier` feature on `deno_media_type` — while the crate exists (verified: version 0.4.0), this feature is not typically something a developer would toggle; the table mixes internal and user-facing features. (2) The **Module Resolution Strategy** in Section 6 does not discuss how the LSP (Doc 01, Section 5) resolves modules differently — the LSP has its own `resolver.rs` that may diverge from the CLI's resolution. (3) Missing discussion of **import map resolution** as a strategy — it's mentioned in the CLI flags but not in the module resolution table. (4) The execution flow diagram (Section 5) omits `x` (remote execution) flow and doesn't show how `deno task` dispatches to `deno_task_shell`. |
| **Recommendation** | (a) Split Cargo features into "user-facing" and "internal". (b) Add import map row to module resolution table. (c) Document LSP's module resolution divergence. (d) Add `task` and `x` to execution flow. |

---

## Module 06 — Security & Compliance

| Aspect | Assessment |
|--------|-----------|
| **Strengths** | Permission State Machine diagram is accurate — all 6 states (`Granted`, `GrantedPartial`, `Prompt`, `Denied`, `DeniedPartial`, `Ignored`) match `PermissionState` enum at `runtime/permissions/lib.rs:133`. Permission checking flow sequence diagram accurately shows the PermissionsContainer → Permissions → PermissionPrompter chain. Clippy enforcement rules correctly document the sandboxing approach. V8 isolation model is well-explained. |
| **Weaknesses / Gaps** | (1) **Allow/Deny interaction table (Section 2.4) is incomplete**: Does not cover `--ignore-*` flag interactions. What happens with `--allow-read --ignore-read=/secret`? (2) **No TOCTOU analysis**: Permission checks happen at op call time, but between the check and the actual file system operation, conditions may change. No documentation of mitigations. (3) **No symlink discussion**: The `ReadDescriptor` and `WriteDescriptor` have a `canonicalized` field (verified at line 278), but the documentation never explains when/how canonicalization is applied or what attacks it mitigates. (4) **"Fail-open" risk**: When `--allow-all` / `-A` is used, ALL permission checks return `Granted` — this is documented, but there's no risk statement or recommended mitigation (e.g., CI linting for `-A` usage). (5) **Supply Chain Security (Section 6)** is minimal by modern standards: no mention of SBOM generation, cargo-vet/cargo-audit integration, dependency vendoring (`deno vendor` exists but isn't linked), or signed releases. (6) **Missing**: `--unsafely-ignore-certificate-errors` risk discussion is minimal — only "DANGEROUS - for development only" note in the flow diagram, no operational guidance. (7) **Section 4.2 ("What is NOT Sandboxed")** omits important items: disk space exhaustion, file descriptor exhaustion, and DNS resolution (which happens before `--allow-net` checks). |
| **Recommendation** | (a) Add `--ignore-*` to the interaction table. (b) Add a "Known Limitations" section covering TOCTOU, symlinks, and resource exhaustion. (c) Add an SBOM / supply chain hardening roadmap. (d) Expand `-A` risk guidance. (e) Add DNS pre-check gap to "What is NOT Sandboxed". |

---

## Module 07 — Testing & Validation

| Aspect | Assessment |
|--------|-----------|
| **Strengths** | Accurate multi-layered test architecture. Spec test schema documentation matches `tests/specs/schema.json`. Output assertion wildcards (`[WILDCARD]`, `[WILDLINE]`, etc.) are well-documented. Running commands are practical and correct. |
| **Weaknesses / Gaps** | (1) **Coverage Map (Section 7) uses vague "Comprehensive" / "Extensive" labels** instead of actual coverage percentages. For a security-first runtime, FFI and Security Sandbox should have explicit coverage targets (e.g., "100% branch coverage for permission checks"). (2) **Spec test schema documentation omits several fields**: `canonicalizedTempDir`, `symlinkedTempDir`, `base`, `repeat`, `ignore`, and `variants` — all present in the actual schema but not in Doc 07's Section 3.4 "Step Properties" table. (3) Coverage Map omits **FFI tests** (`tests/ffi/`) and **NAPI tests** (`tests/napi/`) from the table — these are listed in Section 1 but not in Section 7's coverage assessment. (4) **No mention of fuzz testing** — critical for a runtime that parses URLs, JavaScript, and TypeScript. (5) The "41 categories" claim is correct but Doc 07 lists categories selectively, missing `future_install_node_modules`. (6) No discussion of how **unstable features** are tested — are `--unstable-*` flags covered by spec tests? There's no mapping. |
| **Recommendation** | (a) Add numeric coverage targets for high-risk areas (FFI, permissions, crypto). (b) Complete the step properties table with all schema fields. (c) Add FFI and NAPI to coverage map. (d) Document unstable feature testing strategy. (e) Discuss fuzz testing plans. |
