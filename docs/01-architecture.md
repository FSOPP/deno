# Architecture Document: Deno Runtime

## 1. Overview

Deno is a modern JavaScript/TypeScript/WebAssembly runtime with secure defaults. It is built on three foundational technologies:

- **V8** — Google's JavaScript engine (via `deno_core` / `rusty_v8`)
- **Rust** — Systems programming language providing memory safety and performance
- **Tokio** — Asynchronous runtime for Rust

Deno provides a single binary (`deno`) that serves as both a runtime and a complete developer toolchain, including a formatter, linter, type checker, test runner, bundler, and LSP server.

## 2. High-Level Design

### 2.1 System Context Diagram

```plantuml
@startuml
!theme plain
title System Context Diagram - Deno Runtime

actor "Developer" as dev
actor "CI/CD System" as ci

rectangle "Deno Runtime" as deno {
}

cloud "npm Registry" as npm
cloud "JSR (JavaScript Registry)" as jsr
cloud "Remote URLs (HTTP/HTTPS)" as remote
database "Local File System" as fs
rectangle "V8 Engine" as v8
rectangle "Editor / IDE" as editor

dev --> deno : executes scripts, uses tools
ci --> deno : build, test, lint
deno --> npm : resolves npm packages
deno --> jsr : resolves JSR packages
deno --> remote : fetches remote modules
deno --> fs : reads/writes files
deno --> v8 : JavaScript execution
editor --> deno : LSP protocol

@enduml
```

### 2.2 Component Diagram — Architectural Layers

```plantuml
@startuml
!theme plain
title Component Architecture - Deno Runtime

package "CLI Layer (cli/)" #LightBlue {
  [Argument Parser\n(args/flags.rs)] as ArgParser
  [Service Factory\n(factory.rs)] as Factory
  [Module Loader\n(module_loader.rs)] as ModLoader
  [Type Checker\n(type_checker.rs)] as TypeChecker
  [LSP Server\n(lsp/)] as LSP
  [Subcommands\n(tools/)] as Tools
  [File Fetcher\n(file_fetcher.rs)] as Fetcher
  [Graph Utilities\n(graph_util.rs)] as GraphUtil
}

package "Runtime Layer (runtime/)" #LightGreen {
  [Main Worker\n(worker.rs)] as MainWorker
  [Web Worker\n(web_worker.rs)] as WebWorker
  [Permissions\n(permissions/)] as Permissions
  [V8 Snapshot\n(snapshot.rs)] as Snapshot
  [Bootstrap\n(worker_bootstrap.rs)] as Bootstrap
  [Transpiler\n(transpile.rs)] as Transpiler
}

package "Extensions Layer (ext/)" #LightYellow {
  [Web APIs\n(web, url, fetch)] as WebAPIs
  [I/O APIs\n(fs, io, net, http)] as IOAPIs
  [Crypto & TLS\n(crypto, tls)] as CryptoAPIs
  [Node Compat\n(node, napi)] as NodeAPIs
  [Runtime APIs\n(os, process, signals)] as RuntimeAPIs
  [Storage APIs\n(kv, cache, webstorage)] as StorageAPIs
  [Special APIs\n(ffi, webgpu, cron)] as SpecialAPIs
}

package "Shared Libraries (libs/)" #Lavender {
  [Config Parser\n(config/)] as Config
  [Module Resolver\n(resolver/)] as Resolver
  [NPM Cache\n(npm_cache/)] as NpmCache
  [NPM Installer\n(npm_installer/)] as NpmInstaller
  [Node Resolver\n(node_resolver/)] as NodeResolver
  [Package JSON\n(package_json/)] as PkgJson
}

package "External Crates" #LightGray {
  [deno_core\n(V8 bindings, ops)] as Core
  [tokio\n(async runtime)] as Tokio
  [deno_ast\n(AST parsing)] as AST
  [deno_graph\n(module graph)] as Graph
  [hyper / reqwest\n(HTTP)] as HTTP
  [rustls\n(TLS)] as Rustls
}

ArgParser --> Factory : parsed flags
Factory --> ModLoader : creates
Factory --> TypeChecker : creates
Factory --> Fetcher : creates
ModLoader --> MainWorker : provides modules
Tools --> Factory : uses
LSP --> Factory : uses

MainWorker --> WebAPIs : registers
MainWorker --> IOAPIs : registers
MainWorker --> CryptoAPIs : registers
MainWorker --> NodeAPIs : registers
MainWorker --> RuntimeAPIs : registers
MainWorker --> StorageAPIs : registers
MainWorker --> SpecialAPIs : registers
MainWorker --> Permissions : enforces
MainWorker --> Snapshot : loads
MainWorker --> Bootstrap : init

Resolver --> Config : reads
Resolver --> NodeResolver : delegates
NpmInstaller --> NpmCache : uses

MainWorker --> Core : binds V8
Core --> Tokio : async I/O
GraphUtil --> Graph : builds
Fetcher --> HTTP : HTTP requests
CryptoAPIs --> Rustls : TLS ops
ModLoader --> AST : transpile

@enduml
```

### 2.3 Key Design Decisions

| Decision | Rationale | Alternatives Considered | Source |
|----------|-----------|------------------------|--------|
| Single binary with embedded tools | Developer experience — no separate installs for fmt, lint, test, etc. | Separate binaries per tool | `cli/main.rs`, `cli/tools/` |
| V8 snapshots for fast startup | Avoid re-initializing built-in APIs on every startup | No snapshots (lazy init) | `runtime/snapshot.rs` |
| Permission-based security model | Secure by default — no file/net/env access unless explicitly granted | No sandbox, capability-based | `runtime/permissions/lib.rs` |
| Ops bridge pattern (Rust ↔ JS) | Type-safe, efficient function calls between Rust and JavaScript | Message passing, FFI | `ext/*/` (all `#[op2]` macros) |
| Tokio-based async runtime | Industry-standard async Rust runtime with excellent ecosystem | async-std, smol | `Cargo.toml` workspace deps |
| Factory pattern for service creation | Lazy initialization of only needed services per subcommand | Eager global initialization | `cli/factory.rs` |
| Module graph for dependency resolution | Correct ordering, deduplication, and cycle detection | Linear loading | `cli/graph_util.rs`, `deno_graph` |
| Node.js / npm compatibility | Ecosystem adoption — run npm packages and Node.js code | Pure Deno ecosystem only | `ext/node/`, `libs/node_resolver/` |

## 3. Low-Level Design

### 3.1 Execution Flow — Running a Script

```plantuml
@startuml
!theme plain
title Sequence Diagram - deno run script.ts

participant "CLI\n(main.rs)" as CLI
participant "Flag Parser\n(args/flags.rs)" as Flags
participant "Factory\n(factory.rs)" as Factory
participant "Module Loader\n(module_loader.rs)" as Loader
participant "File Fetcher\n(file_fetcher.rs)" as Fetcher
participant "Graph Builder\n(graph_util.rs)" as Graph
participant "Type Checker\n(type_checker.rs)" as TC
participant "Main Worker\n(runtime/worker.rs)" as Worker
participant "V8 Engine\n(deno_core)" as V8
participant "Extensions\n(ext/*)" as Ext
participant "Permissions\n(permissions/)" as Perm

CLI -> Flags : parse argv
Flags --> CLI : Arc<Flags>
CLI -> Factory : new(flags)
Factory -> Loader : create_module_loader()
Factory -> Fetcher : create_file_fetcher()

CLI -> Worker : create(options)
Worker -> Ext : register extensions
Worker -> Perm : initialize permissions
Worker -> V8 : create isolate

CLI -> Loader : load("script.ts")
Loader -> Fetcher : fetch(specifier)
Fetcher --> Loader : source code
Loader -> Graph : build module graph
Graph --> Loader : resolved graph

alt Type checking enabled
  Loader -> TC : check(graph)
  TC --> Loader : diagnostics
end

Loader --> Worker : compiled modules
Worker -> V8 : execute_main_module()
V8 -> Ext : op calls (via #[op2])
Ext -> Perm : permission checks
Perm --> Ext : allow/deny
Ext --> V8 : results
V8 --> Worker : completion
Worker --> CLI : exit code

@enduml
```

### 3.2 Extension Registration Order

Extensions are registered in a specific order in `runtime/worker.rs` (function `common_extensions`). The order matters because extensions may depend on ops from earlier extensions.

```plantuml
@startuml
!theme plain
title Extension Registration Order (runtime/worker.rs)

rectangle "1. Telemetry" as T #LightCyan
rectangle "2. WebIDL" as WIDL #LightYellow
rectangle "3. Web\n(base Web APIs)" as Web #LightYellow
rectangle "4. WebGPU" as WGPU #LightYellow
rectangle "5. Image" as Img #LightYellow
rectangle "6. Fetch" as Fetch #LightGreen
rectangle "7. Cache" as Cache #LightGreen
rectangle "8. WebSocket" as WS #LightGreen
rectangle "9. WebStorage" as WSto #LightGreen
rectangle "10. Crypto" as Crypto #Orange
rectangle "11. FFI" as FFI #Pink
rectangle "12. Net" as Net #LightGreen
rectangle "13. TLS" as TLS #Orange
rectangle "14. KV" as KV #Lavender
rectangle "15. Cron" as Cron #Lavender
rectangle "16. NAPI" as NAPI #Pink
rectangle "17. HTTP" as HTTP #LightGreen
rectangle "18. I/O" as IO #LightBlue
rectangle "19. FS" as FS #LightBlue
rectangle "20. OS" as OS #LightBlue
rectangle "21. Process" as Proc #LightBlue
rectangle "22. Node\n(compat)" as Node #Pink
rectangle "23. Runtime Ops" as RT #Gray
rectangle "24. Worker Host" as WH #Gray
rectangle "25. FS Events" as FSE #Gray
rectangle "26. Permissions" as Perm #Red
rectangle "27. TTY" as TTY #Gray
rectangle "28. HTTP Runtime" as HTTPR #Gray
rectangle "29. Bundle Runtime" as BR #Gray
rectangle "30. Bootstrap" as Boot #Gray

T -[hidden]down-> WIDL
WIDL -[hidden]down-> Web
Web -[hidden]down-> WGPU
WGPU -[hidden]down-> Img
Img -[hidden]down-> Fetch
Fetch -[hidden]down-> Cache
Cache -[hidden]down-> WS
WS -[hidden]down-> WSto
WSto -[hidden]down-> Crypto

@enduml
```

### 3.3 Module Resolution Architecture

```plantuml
@startuml
!theme plain
title Module Resolution Flow

start

:User imports module\n(import "specifier");

if (Specifier type?) then (file:// or relative)
  :Local file system read;
else if (Specifier type?) then (https:// or http://)
  :Remote fetch via FileFetcher;
  :Cache in DENO_DIR;
else if (Specifier type?) then (npm:)
  :NPM package resolution;
  :npm_installer → npm_cache;
  :Node module resolver;
else if (Specifier type?) then (jsr:)
  :JSR registry lookup;
  :Resolve to HTTPS URL;
  :Fetch and cache;
else if (Specifier type?) then (node:)
  :Node built-in module;
  :ext/node polyfill;
else
  :Unknown scheme → error;
  stop
endif

:Parse AST (deno_ast);
:Transpile TS → JS if needed;
:Add to Module Graph;
:Resolve transitive deps;
:Return compiled module to V8;

stop

@enduml
```

### 3.4 Worker Architecture

```plantuml
@startuml
!theme plain
title Worker Architecture

package "Main Worker" #LightBlue {
  [V8 Isolate\n(single-threaded)] as MainIsolate
  [Event Loop\n(tokio runtime)] as MainLoop
  [Op Table\n(registered ops)] as MainOps
  [Resource Table\n(open files, sockets)] as MainRes
  [Module Map\n(loaded modules)] as MainMod
}

package "Web Worker (spawned)" #LightGreen {
  [V8 Isolate\n(own thread)] as WorkerIsolate
  [Event Loop\n(tokio runtime)] as WorkerLoop
  [Op Table\n(subset of ops)] as WorkerOps
  [Resource Table\n(own resources)] as WorkerRes
  [Module Map\n(own modules)] as WorkerMod
}

MainIsolate --> MainLoop : drives
MainLoop --> MainOps : dispatches
MainOps --> MainRes : accesses
MainIsolate --> MainMod : loads

WorkerIsolate --> WorkerLoop : drives
WorkerLoop --> WorkerOps : dispatches
WorkerOps --> WorkerRes : accesses
WorkerIsolate --> WorkerMod : loads

MainIsolate ..> WorkerIsolate : postMessage\n(structured clone)

note right of MainIsolate
  Main worker has full permissions
  and access to all registered extensions.
  Web workers may have restricted permissions.
end note

@enduml
```

### 3.5 CLI Subcommand Architecture

The CLI binary exposes numerous subcommands, all dispatched from `run_subcommand()` in `cli/main.rs`:

```plantuml
@startuml
!theme plain
title CLI Subcommand Dispatch

rectangle "deno" as entry

package "Script Execution" #LightBlue {
  [run] as run
  [eval] as eval
  [repl] as repl
  [serve] as serve
  [x] as x
}

package "Development Tools" #LightGreen {
  [fmt] as fmt
  [lint] as lint
  [test] as test
  [bench] as bench
  [check] as check
  [doc] as doc
  [coverage] as coverage
}

package "Package Management" #LightYellow {
  [add] as add
  [remove] as remove
  [install] as install
  [uninstall] as uninstall
  [outdated] as outdated
  [audit] as audit
}

package "Build & Deploy" #Orange {
  [compile] as compile
  [bundle] as bundle
  [deploy] as deploy
  [publish] as publish
}

package "Info & Utilities" #Lavender {
  [info] as info
  [init] as init
  [cache] as cache
  [clean] as clean
  [upgrade] as upgrade
  [lsp] as lsp
  [jupyter] as jupyter
  [completions] as completions
}

entry --> run
entry --> eval
entry --> repl
entry --> serve
entry --> x
entry --> fmt
entry --> lint
entry --> test
entry --> bench
entry --> check
entry --> doc
entry --> coverage
entry --> add
entry --> remove
entry --> install
entry --> uninstall
entry --> outdated
entry --> audit
entry --> compile
entry --> bundle
entry --> deploy
entry --> publish
entry --> info
entry --> init
entry --> cache
entry --> clean
entry --> upgrade
entry --> lsp
entry --> jupyter
entry --> completions

@enduml
```

## 4. Internal Crate Dependency Graph

```plantuml
@startuml
!theme plain
title Internal Crate Dependency Graph

' CLI layer
[deno (cli)] as cli #LightBlue
[deno_lib (cli/lib)] as lib #LightBlue
[deno_snapshots (cli/snapshot)] as snap #LightBlue

' Runtime layer
[deno_runtime (runtime)] as rt #LightGreen
[deno_permissions (runtime/permissions)] as perms #LightGreen
[deno_features (runtime/features)] as feat #LightGreen

' Extension crates
[deno_web] as web #LightYellow
[deno_fetch] as fetch #LightYellow
[deno_net] as net #LightYellow
[deno_http] as http #LightYellow
[deno_fs] as fs #LightYellow
[deno_io] as io #LightYellow
[deno_crypto] as crypto #LightYellow
[deno_node] as node #Pink
[deno_napi] as napi #Pink
[deno_ffi] as ffi #LightYellow
[deno_kv] as kv #LightYellow
[deno_tls] as tls #LightYellow
[deno_url] as url #LightYellow
[deno_webidl] as webidl #LightYellow
[deno_websocket] as ws #LightYellow
[deno_webstorage] as wsto #LightYellow
[deno_cache] as cache #LightYellow
[deno_telemetry] as tel #LightYellow
[deno_os] as os #LightYellow
[deno_process] as proc #LightYellow
[deno_signals] as sig #LightYellow
[deno_webgpu] as webgpu #LightYellow
[deno_image] as image #LightYellow

' Library crates
[deno_config] as config #Lavender
[deno_resolver] as resolver #Lavender
[deno_npm_cache] as npm_cache #Lavender
[deno_npm_installer] as npm_inst #Lavender
[node_resolver] as node_res #Lavender
[deno_package_json] as pkg_json #Lavender

' External
[deno_core] as core #LightGray

' Deps: CLI
cli --> lib
cli --> rt
cli --> snap
lib --> rt
snap --> rt

' Deps: Runtime
rt --> perms
rt --> feat
rt --> web
rt --> fetch
rt --> net
rt --> http
rt --> fs
rt --> io
rt --> crypto
rt --> node
rt --> napi
rt --> ffi
rt --> kv
rt --> tls
rt --> url
rt --> webidl
rt --> ws
rt --> wsto
rt --> cache
rt --> tel
rt --> os
rt --> proc
rt --> sig
rt --> webgpu
rt --> image

' Deps: Libs
cli --> config
cli --> resolver
cli --> npm_cache
cli --> npm_inst
cli --> node_res
cli --> pkg_json

' Deps: Core
rt --> core
web --> core
fetch --> core
net --> core
fs --> core

@enduml
```

## 5. LSP Architecture

```plantuml
@startuml
!theme plain
title LSP Server Architecture (cli/lsp/)

actor "Editor / IDE" as editor

package "LSP Server (cli/lsp/)" #LightBlue {
  [Language Server\n(language_server.rs)] as LS
  [Diagnostics\n(diagnostics.rs)] as Diag
  [Completions\n(completions.rs)] as Comp
  [Code Lens\n(code_lens.rs)] as CL
  [Semantic Tokens\n(semantic_tokens.rs)] as ST
  [Document Manager\n(documents.rs)] as Docs
  [Config\n(config.rs)] as LSPConfig
  [TSC Integration\n(tsc.rs)] as TSC
  [Analysis\n(analysis.rs)] as Analysis
  [Registries\n(registries.rs)] as Reg
  [Resolver\n(resolver.rs)] as Res
  [Performance\n(performance.rs)] as Perf
}

editor <--> LS : JSON-RPC\n(stdio transport)
LS --> Diag : request diagnostics
LS --> Comp : completion requests
LS --> CL : code lens requests
LS --> ST : semantic token requests
LS --> Docs : document sync
LS --> LSPConfig : workspace config
LS --> TSC : TypeScript analysis
LS --> Analysis : code analysis
LS --> Reg : registry completions
LS --> Res : module resolution
LS --> Perf : performance tracking

@enduml
```

## 6. References

- [cli/main.rs](cli/main.rs) — CLI entry point and subcommand dispatch
- [cli/factory.rs](cli/factory.rs) — Service factory for runtime components
- [cli/module_loader.rs](cli/module_loader.rs) — Module loading and resolution
- [cli/args/flags.rs](cli/args/flags.rs) — CLI flag definitions
- [runtime/worker.rs](runtime/worker.rs) — Main worker and extension registration (`common_extensions` at line 1039)
- [runtime/web_worker.rs](runtime/web_worker.rs) — Web Worker implementation
- [runtime/permissions/lib.rs](runtime/permissions/lib.rs) — Permission system (`Permissions` struct at line 3342)
- [runtime/snapshot.rs](runtime/snapshot.rs) — V8 snapshot generation
- [cli/lsp/language_server.rs](cli/lsp/language_server.rs) — LSP server implementation

## Appendix

### Glossary

| Term | Definition |
|------|------------|
| **Op** | A Rust function registered with deno_core that can be called from JavaScript. Defined via `#[op2]` macro. |
| **Extension** | A collection of ops and optional JavaScript code that provides a set of APIs (e.g., `deno_fetch`). |
| **Worker** | A JavaScript execution context with its own V8 isolate, event loop, and resource table. |
| **Resource** | A Rust object (file, socket, etc.) tracked in the resource table, accessible from JS via resource IDs. |
| **Module Graph** | A directed graph of module dependencies, used for correct load ordering and deduplication. |
| **Snapshot** | A serialized V8 heap state that enables fast startup by skipping initialization of built-in APIs. |
| **Specifier** | A module identifier (URL, file path, npm: specifier, jsr: specifier, etc.). |
| **DENO_DIR** | The local cache directory where Deno stores downloaded modules, compiled code, and type check info. |
| **JSR** | JavaScript Registry — Deno's native package registry (jsr.io). |
| **NAPI** | Node-API — compatibility layer for native Node.js addons. |
| **FFI** | Foreign Function Interface — allows calling native shared libraries from JavaScript. |
