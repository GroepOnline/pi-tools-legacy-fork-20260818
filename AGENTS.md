# To Clankers

**GroepOnline/pi-tools** is a maintained fork of [fff](https://github.com/dmtrKovalenko/fff): a typo-resistant, SIMD-accelerated file search toolkit. Our product is the pi extension `@groeponline/pi-fff`; the repo carries the full upstream workspace (MCP server, fff.nvim, Node/Bun SDKs, C library, Python bindings, Rust crates).

## Development Commands

Prefer Makefile commands over cargo/bun/npm where they exist.

- `make build` - build everything (Rust crates + native libs)
- `make lint` - Rust linting and analysis
- `make format` - format all code
- `make test` - Rust unit/integration tests
- `make test-node` - Node SDK tests (requires built `libfff_c`)
- `bun test packages/pi-fff/test/` - pi extension tests (no native lib needed)
- `npm run check:ci` in `packages/` - oxlint + oxfmt checks

> `cargo` is intentionally not installed on joep's laptop. Build Rust on the runner: `ssh chef@chef-runner-01-1 'cd <checkout> && export PATH=$HOME/.cargo/bin:$PATH && cargo build --release'`.

## Coding rules

- **Reduce comment size.** Every comment is a concise 1-2 liner; max 4 lines only for genuinely unintuitive concepts.
- No module-level or top-of-file comment blocks.
- No doc comments on private structs/functions.
- Keep structs/functions private when they can be.
- Utility functions go at the end of the file.
- Never change top-level Rust, Lua, C, or bun APIs.

## Architecture

Everything performance-critical lives in Rust; everything host-specific lives in the wrappers.

- `crates/fff-core` - index, background watcher, frecency/history LMDB dbs, scoring
- `crates/fff-grep` - SIMD content search; `crates/fff-query-parser` - query constraints
- `crates/fff-c` - C FFI (consumed by Node/Bun/Python); `crates/fff-mcp` - MCP server
- `crates/fff-nvim` + `lua/` - Neovim plugin
- `packages/fff-node`, `packages/fff-bun` - SDKs (published as `@groeponline/*`)
- `packages/pi-fff` - the pi extension (`@groeponline/pi-fff`), our main product

Databases: frecency (LMDB, file access patterns) and query history (LMDB, past searches). Both can be shared with fff.nvim; pi-fff defaults to `~/.pi/agent/fff/`.

## Fork-specific rules

- Keep the fork delta small and rebaseable: one clean commit on top of upstream `main`.
- The `@groeponline` scope is ours; `@ff-labs/fff-bin-*` platform packages stay upstream-scoped (we consume them, we don't publish them).
- pi-fff source imports must match the `@groeponline` deps — a scope rename that leaves `@ff-labs` imports behind is a runtime bug.
- Update lockfiles (`packages/bun.lock`, `packages/package-lock.json`) when touching package names or deps.
- CI: builds/publishes only on v* tags + workflow_dispatch; test matrix is Linux-only on push/PR.

## Working with Rust

- Prefer struct methods over free functions.
- More than 2 impls in a file -> split the file.
- Be careful with mutex/rwlock: check with a human before introducing long-held locks.

## Working with pi-fff (TypeScript)

- Validate user inputs; document public function types.
- Reuse existing helpers (`loadSdk`, `buildQuery`, `AuxFinderPool`).
- Never break tool registration or the `@`-autocomplete provider flow.
- When adding config options, update `packages/pi-fff/README.md` and the main `README.md`.
