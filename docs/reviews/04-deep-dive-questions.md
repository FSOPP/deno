# Deep Dive Questions for Design Validation

The following 5 questions are designed to probe the most ambiguous and highest-risk areas of the documentation. Each question targets a specific design gap that must be resolved to validate the architecture.

---

## Question 1: Permission Canonicalization and Symlink Attack Surface

**Context:** The `ReadDescriptor` and `WriteDescriptor` types both contain a `canonicalized: bool` field (verified at `runtime/permissions/lib.rs:278` and `350`). Doc 06 documents the permission state machine but never mentions canonicalization.

**Question:**

> At what point in the permission check pipeline is path canonicalization performed? Is it done:
> (a) at descriptor creation time (CLI flag parsing),
> (b) at check time (each op call), or
> (c) lazily on first access?
>
> For scenario (a): an attacker could create a symlink **after** flag parsing.
> For scenario (b): this adds overhead to every FS op.
> For scenario (c): there's a TOCTOU window.
>
> Additionally: if `--allow-read=/tmp/safe` is granted and `/tmp/safe` is a symlink to `/etc/passwd`, does the permission check resolve the symlink before comparison? The `canonicalized` field suggests conditional behavior — under what conditions is `canonicalized` set to `true`?

**Why it matters:** This is a potential sandbox escape vector. The answer determines whether Deno's file system permissions can be bypassed through symlink manipulation.

---

## Question 2: `--ignore-*` Flag Semantics and Interaction with `--deny-*`

**Context:** Doc 04 lists `ignore_env` and `ignore_read` fields in `PermissionsOptions`. Doc 06 shows `Ignored` as a permission state. But no document explains the full interaction matrix.

**Question:**

> What is the complete interaction matrix for `--allow-*`, `--deny-*`, and `--ignore-*` flags?
>
> Specifically:
> 1. `--ignore-read=/etc` + `--deny-read=/etc` → Does deny override ignore?
> 2. `--ignore-read` (no path) + `--allow-read=/tmp` → What happens for reads outside `/tmp`? Are they ignored (return `NotFound`) or prompted?
> 3. Can `--ignore-*` be used to mask the existence of sensitive files from sandboxed code, or does it only affect the error type returned?
> 4. Is there precedence: `deny > ignore > allow`, or `deny > allow > ignore`?
>
> The `Ignored` state returning `NotFound` instead of `PermissionDenied` has **information security implications**: malicious code could distinguish between "permission denied" and "file not found" to probe the sandbox boundaries.

**Why it matters:** The `--ignore-*` feature is undocumented in the API reference (Doc 03) and its interaction semantics are unclear. If the precedence is wrong, it could either leak information or silently mask permission-denied errors.

---

## Question 3: FFI Callback Safety and Resource Lifecycle

**Context:** Doc 03 Section 5 documents the FFI boundary but omits discussion of `op_ffi_unsafe_callback_create`, `op_ffi_unsafe_callback_close`, and `op_ffi_unsafe_callback_ref` (verified in `ext/ffi/lib.rs:78-80`).

**Question:**

> How does the FFI callback lifecycle interact with V8's garbage collector?
>
> Specifically:
> 1. When `Deno.UnsafeCallback` is created via `op_ffi_unsafe_callback_create`, what prevents the callback from being called after the JS function has been GC'd?
> 2. If native code retains a function pointer from `Deno.UnsafeCallback` beyond the callback's explicit lifetime, what happens? Is there a guard, or is this undefined behavior?
> 3. The `op_ffi_unsafe_callback_ref` op suggests reference counting — does the ref count prevent GC, and is there a leak risk if `close()` is never called?
> 4. For `Deno.UnsafeFnPointer`, what prevents use-after-free if the backing library is unloaded (`lib.close()`) while JS still holds a reference to the pointer?
>
> These are fundamental memory safety questions that affect whether FFI can cause segfaults or memory corruption in the Deno process.

**Why it matters:** FFI explicitly operates **outside** the Rust safety guarantees and V8 sandbox. A memory corruption through FFI could compromise the entire process, including the permission system. This deserves explicit documentation of the safety contract.

---

## Question 4: Web Worker Permission Inheritance and Escalation

**Context:** Doc 01 Section 3.4 (Worker Architecture) notes "Main worker has full permissions and access to all registered extensions. Web workers may have restricted permissions." But the permission delegation model is not documented anywhere.

**Question:**

> How are permissions delegated from the main worker to a web worker?
>
> 1. Does a web worker inherit the main worker's permissions by default, or does it start with `Prompt` state?
> 2. Can a web worker's permissions be **more** restrictive than the main worker's? Can they be **less** restrictive (escalation)?
> 3. The `CreateWebWorkerCb` callback in `WorkerOptions` — does it receive an explicit `PermissionsOptions`, or does it clone the parent's `PermissionsContainer`?
> 4. If `--allow-all` is passed, does every web worker also get `--allow-all`?
> 5. Are there spec tests (in `tests/specs/worker/`) that verify permission escalation is impossible?
>
> The answer affects whether a malicious library imported into the main thread can spawn workers with escalated permissions.

**Why it matters:** Worker-based permission escalation is a common attack pattern in browser-like runtimes. The documentation must clarify that workers cannot exceed their parent's permissions.

---

## Question 5: Spec Test Coverage of Unstable Features

**Context:** Doc 05 Section 4 documents 26 unstable feature flags (8 CLI-level, 18 runtime-level). Doc 07 documents the spec test infrastructure. But there's no explicit mapping between unstable features and their test coverage.

**Question:**

> For each of the 26 unstable feature flags, does there exist:
> 1. At least one spec test that enables the flag and validates the feature works?
> 2. At least one spec test that validates the feature is **disabled** by default (negative test)?
> 3. Is there a mechanism in the test infrastructure to automatically generate tests for new unstable features?
>
> Specifically:
> - `--unstable-unsafe-proto`: Is there a test that confirms `__proto__` is NOT accessible when the flag is off?
> - `--unstable-ffi`: Are FFI tests gated behind this flag, and do they verify the permission check still applies even with the unstable flag?
> - `--unstable-temporal`: The Temporal API is complex — is there WPT coverage for it?
> - `--unstable-vsock`: Is this feature tested on CI environments that lack vsock support?
>
> The concern is that unstable features, by nature, receive less testing rigor, but they can still be security-relevant (see `--unstable-unsafe-proto`).

**Why it matters:** Unstable features are where bugs are most likely to exist. Without a clear testing strategy, regressions could ship in stable releases if an unstable flag is promoted to stable without adequate test coverage.
