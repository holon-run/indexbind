# Agent Notes

## Project Status

- Public project name is now `indexbind`.
- GitHub repository is `https://github.com/holon-run/indexbind` (migrated from `jolestar/indexbind`).
- Local directory path is still the old one:
  - `/Users/jolestar/opensource/src/github.com/jolestar/inkdex`
- If the local directory is renamed to `.../indexbind`, start a new Codex session instead of trying to reuse this one.

## Current Shape

- Rust workspace crates:
  - `crates/indexbind-core`
  - `crates/indexbind-build`
  - `crates/indexbind-node`
- npm package name: `indexbind`
- Retrieval model is document-first.
- Artifact is SQLite single-file.
- Node runtime uses the Rust core through NAPI.

## Phase Status

- Phase 1: complete
- Phase 2: complete
- Phase 3: complete
  - optional reranking exists
  - default reranker kind is `embedding-v1`
  - `heuristic-v1` remains available
- Phase 4: mostly complete
  - native loader fallback/error handling improved
  - `inspect` CLI command exists
  - minimal benchmark fixtures exist
  - incremental build cache and fresh export flow exist
  - published prebuilt native packages exist for macOS arm64, macOS x64, and Linux x64 (glibc)

## Important Commands

- Build native addon, TS wrapper, and public CLI:
  - `npm run build`
- Build the native addon and TS CLI without wasm:
  - `npm run build:cli`
- TypeScript and Rust checks:
  - `npm run check`
- Rust tests:
  - `cargo test --workspace`
- Build an artifact:
  - `npm run build:cli && node dist/cli.js build ./docs ./index.sqlite`
- Inspect an artifact:
  - `npm run build:cli && node dist/cli.js inspect ./index.sqlite`
- Run bundled retrieval regression fixture:
  - `npm run benchmark:basic`

## Benchmark Fixture

- Fixture path:
  - `fixtures/benchmark/basic`
- Purpose:
  - minimal retrieval regression baseline
  - fixed corpus + fixed queries + expected top hit

## Naming and Packaging Notes

- Old `inkdex` naming was fully renamed in source/config/docs to `indexbind`.
- `origin` already points to:
  - `https://github.com/holon-run/indexbind.git`
- `package.json` repository/homepage/bugs fields point to `holon-run/indexbind`.
- Native loader expects files/packages under `indexbind` names:
  - local dev file: `native/indexbind.<platform>.node`
  - prebuilt package pattern: `@indexbind/native-<platform>`

## Known Follow-up Work

- Decide whether to rename the local directory on disk from `inkdex` to `indexbind`.
- If/when local directory is renamed:
  - open a new session from the new path
  - rebuild native output with `npm run build`
- Finish Phase 4 publishing work:
  - install matrix validation on Linux/Windows/macOS
  - widen prebuilt coverage, especially Windows and remaining Linux targets
  - publish-oriented README cleanup
  - package `exports/files` cleanup before release

## Recent Commits

- `a89a857` Rename project to Indexbind
- `545e3bc` Add benchmark fixtures and regression command
- `ef29738` Improve native loading and artifact inspection
- `1d7de38` Add embedding-based document reranker
- `2b5809a` Implement Phase 3 optional document reranking
- `a0ccb93` Implement Phase 2 local embeddings and hybrid fusion
- `9635741` Implement Phase 1 document-first retrieval core
