---
name: indexbind
version: 1.0.1
description: Use when an agent needs to install or use indexbind from Node, browsers, Web Workers, or Cloudflare Workers. This skill helps choose the right package, CLI, artifact, and entrypoint, and points to the live markdown docs for details.
---

# Indexbind

Use this skill when the task is about using `indexbind` from a host application or environment.

## Usage examples

- "Use `indexbind` to add local search to a docs folder."
- "Help me choose between `indexbind`, `indexbind/web`, and `indexbind/cloudflare`."
- "Show me how to build a SQLite artifact for Node and a bundle for Workers."

## Install

Install the package:

```bash
npm install indexbind
```

Optional global install when the goal is using `indexbind` as a shell command from arbitrary directories:

```bash
npm install -g indexbind
```

Then use either:

- `npx indexbind ...` for local installs and per-project workflows
- `indexbind ...` after a global install
- `import ... from 'indexbind'` or `indexbind/build` for programmatic usage

Platform notes:
- native prebuilds exist for macOS arm64, macOS x64, and Linux x64 (glibc)
- Windows usage should go through WSL

Install and packaging docs:
- `https://indexbind.holon.run/guides/getting-started.md`
- `https://indexbind.holon.run/reference/packaging.md`

## Choose the right interface

- Index a local docs folder or local knowledge-base directory from the shell:
  use `npx indexbind ...`
- Local Node querying over a built SQLite artifact:
  use `indexbind`
- Programmatic build, incremental cache update, inspect, or benchmark:
  use `indexbind/build`
- Mixed local knowledge bases that need host-defined document classification, metadata, or directory weighting:
  normalize documents in the host first, then pass them to `indexbind/build`
- A mostly-default local docs or knowledge-base directory that only needs light host policy:
  use `indexbind.build.js` and `indexbind.search.js` beside that directory’s `.indexbind/`
- Browser or standard worker querying over a canonical bundle:
  use `indexbind/web`
- Cloudflare Worker querying:
  use `indexbind/cloudflare`
- Shell-driven build/update/export/inspect flows:
  use `npx indexbind ...`

API docs:
- `https://indexbind.holon.run/reference/api.md`
- `https://indexbind.holon.run/reference/cli.md`

## Choose the artifact

- Local directory indexing for later Node queries:
  build a native SQLite artifact
- Local directory indexing for browser or worker delivery:
  build a canonical bundle
- Node runtime:
  use a native SQLite artifact
- Browser, Web Worker, Cloudflare Worker:
  use a canonical bundle
- Repeated rebuilds over a stable corpus:
  use the build cache, then export fresh artifacts or bundles

Concepts:
- `https://indexbind.holon.run/concepts/runtime-model.md`
- `https://indexbind.holon.run/concepts/canonical-bundles.md`

## Common commands

Typical CLI commands:

- `npx indexbind build ./docs`
- `npx indexbind build-bundle ./docs`
- `npx indexbind update-cache ./docs --git-diff`
- `npx indexbind build [input-dir] [output-file] [--backend <hashing|model-id>]`
- `npx indexbind build-bundle [input-dir] [output-dir] [--backend <hashing|model-id>]`
- `npx indexbind update-cache [input-dir] [cache-file] [--git-diff] [--git-base <rev>] [--backend <hashing|model-id>]`
- `npx indexbind export-artifact <output-file> [--cache-file <path>]`
- `npx indexbind export-bundle <output-dir> [--cache-file <path>]`
- `npx indexbind inspect <artifact-file>`
- `npx indexbind search <artifact-file> <query>`
- `npx indexbind benchmark <artifact-file> <queries-json>`

Use `indexbind/build` instead when the host already has documents in memory or wants tighter control from code.

## Index-scoped conventions

When one indexed root only needs a small amount of host-specific behavior, place convention files beside that root:

```text
docs/
  indexbind.build.js
  indexbind.search.js
  .indexbind/
```

Use `indexbind.build.js` when the default directory scanner is already correct and you only need to:

- skip a few files from indexing
- derive `canonicalUrl`
- inject metadata such as `is_default_searchable`, `source_root`, `content_kind`, or `directory_weight`
- normalize `title` or `summary`

Use `indexbind.search.js` when CLI or Node search should automatically apply:

- a default search profile
- a metadata filter
- score adjustment defaults
- lightweight query rewrite or alias expansion

These convention files are index-scoped, not repo-global:

- if you index `./docs`, the files live in `./docs/`
- they affect only that indexed root
- there is no repo-root fallback

## Common APIs

Use these APIs when the host already has documents or wants tighter control:

- `openIndex(...)` from `indexbind`
- `buildFromDirectory(...)` from `indexbind/build`
- `buildCanonicalBundle(...)` from `indexbind/build`
- `buildCanonicalBundleFromDirectory(...)` from `indexbind/build`
- `updateBuildCache(...)` from `indexbind/build`
- `updateBuildCacheFromDirectory(...)` from `indexbind/build`
- `exportArtifactFromBuildCache(...)` from `indexbind/build`
- `exportCanonicalBundleFromBuildCache(...)` from `indexbind/build`
- `inspectArtifact(...)` from `indexbind/build`
- `benchmarkArtifact(...)` from `indexbind/build`
- `openWebIndex(...)` from `indexbind/web`
- `openWebIndex(...)` from `indexbind/cloudflare`

Docs:
- `https://indexbind.holon.run/reference/api.md`
- `https://indexbind.holon.run/guides/adoption-examples.md`
- `https://indexbind.holon.run/reference/cli.md`

## Cloudflare rule

Inside Cloudflare Workers:

- prefer `indexbind/cloudflare`
- if bundle files are not directly exposed as public URLs, pass a custom `fetch` to `openWebIndex(...)`
- use the host asset loader such as `ASSETS.fetch(...)` rather than monkey-patching global fetch

Docs:
- `https://indexbind.holon.run/guides/web-and-cloudflare.md`
- `https://indexbind.holon.run/reference/api.md`

## Read in this order when unsure

1. `https://indexbind.holon.run/guides/getting-started.md`
2. `https://indexbind.holon.run/reference/api.md`
3. `https://indexbind.holon.run/reference/cli.md`
4. `https://indexbind.holon.run/guides/web-and-cloudflare.md`
