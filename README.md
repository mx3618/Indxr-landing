# Vecto-Indxr

> AI codebase indexer & analysis engine — structured search, impact analysis, and design tooling for AI agents.

Vecto-Indxr builds a **declaration-level structural index** of a codebase — files, symbols, signatures, call edges — and exposes it to AI agents through a **MCP server (180+ tools)** and a **CLI**. It answers questions like *"what tests cover `parse_files`?"*, *"who calls this function?"*, or *"how would renaming this symbol break my code?"* in milliseconds.

The project is **developed largely with GLM-5.2 and GLM-5.3** — it is built by the same kind of coding agents it serves. The source repository is **private**. This page is the public face: features, activity, and runnable-looking examples of what the tool produces.

---

## Table of contents

- [Feature overview](#feature-overview)
- [Quick tour (examples)](#quick-tour-examples)
- [Benchmarks](#benchmarks)
- [CLI & MCP surface](#cli--mcp-surface)
- [Tech stack](#tech-stack)
- [Activity](#activity)
- [FAQ](#faq)

---

## Feature overview

### 1. Structural indexing (`indxr index`)
- Walks a workspace and writes `INDEX.md` — a navigable map of every file, symbol, signature, and its relationships
- 60+ languages via tree-sitter (Rust, Python, TypeScript, Go, Java, C/C++, ...)
- Incremental re-indexing with per-file fingerprints (only changed files are re-parsed)
- Cached analysis results keyed by fast content hashes (~294,000x speedup on the cache-key step)

### 2. Search & navigation
| Tool | Answers |
|------|---------|
| `find` | "where is `Scheduler` defined?" / fuzzy file matching |
| `get_callers` / `get_call_graph` | "who calls this? what is the call graph?" |
| `get_type_flow` | "who produces / consumes this type?" |
| `search_semantic` | "find code *about* caching" (concept, not keyword) |
| `get_impact` | blast radius of changing a symbol |

### 3. Analysis & health
- Complexity hotspots, dead code, duplicate functions, circular dependencies
- Test coverage audit (`check_test_coverage`), related-test discovery
- Performance anti-pattern scanning (clones, `unwrap`, locks in hot paths)
- Structural diff of PRs: `+ added`, `- removed`, `~ changed` at declaration level

### 4. Design & experiment tooling
- Architecture design drafts with component graphs, constraint checking, cost estimation
- Experiment tracking with hypothesis → method → result → lesson lifecycle
- Codebase knowledge wiki, session journal, metrics & baselines
- Dataset pipeline: collect → validate → redact → export (SFT/DPO/COT/...)

### 5. MCP server (`indxr serve`)
- 180+ tools across 11 categories: search, reading, analysis, design, experiment, learning, wiki, profile, external, execution, workspace
- JSON-RPC over stdio & HTTP, streaming results, adaptive result compression
- Git integration: structured `git status`, safe `git commit`/`restore` with dry-run defaults

### 6. CLI commands
`indxr index` · `serve` · `query` · `symbol` · `doctor` · `watch` · `flow` · `wiki` · `init` · `dev` · `default`

---

## Why it's different from the normal way

| Task | Normal way (grep + opening files) | Vecto-Indxr |
|------|-----------------------------------|-------------|
| Find where a symbol is defined | `grep -rn` across the repo, open every hit | `find` → exact declaration + signature, instant |
| Understand a function | open the file, read the body, chase callers by hand | `explain_symbol` → signature + docs + relationships **without reading source** |
| Who calls this? | grep + manually open every candidate file | `get_callers` → exact call sites with file:line |
| What tests cover this? | guess from names, grep test files | `get_related_tests` → 3-layer discovery (defining file → names → body refs) |
| Will my change break something? | mental model, then run the full suite | `get_impact` → blast radius: affected files + tests *before* editing |
| How healthy is the repo? | vibes, broken CI, slow reviews | `get_health` → complexity, hotspots, doc coverage, test counts |
| Keep the agent's context small | dump whole files into the prompt | surgical lookups — millisecond, token-efficient |
| Navigate a 440K-line repo | Ctrl+F and hope | one structural index, every query is a lookup |

The same questions, answered in **milliseconds instead of minutes** — and for AI agents, measured in **tokens saved, not just time saved**.

## Quick tour (examples)

Real output shapes produced by the tool (data is illustrative):

- [`examples/index.md`](examples/index.md) — what a generated `INDEX.md` looks like
- [`examples/search-results.md`](examples/search-results.md) — symbol lookup + callers + type flow
- [`examples/call-graph.md`](examples/call-graph.md) — Mermaid call graph from `get_call_graph`
- [`examples/mcp-tools.md`](examples/mcp-tools.md) — sample MCP requests/responses
- [`examples/changelog.md`](examples/changelog.md) — auto-drafted changelog from git history
- [`examples/test-results.md`](examples/test-results.md) — **real recorded test results** (suite, perf, security)
- [`examples/recorded-evidence.md`](examples/recorded-evidence.md) — verbatim excerpts from the tracking store
- [`examples/live-tool-tests.md`](examples/live-tool-tests.md) — **live runs** of find / search / analysis tools against a 440K-line codebase

---

## Benchmarks (recorded, real)

Every number below comes from the project's own tracking store (`benchmarks.jsonl` / `metrics.jsonl`), verified through the vecto-indxr MCP server on 2026-08-18. Full tables + verbatim excerpts:

- **[test-results.md](examples/test-results.md)** — test suite, perf, security, reliability
- **[recorded-evidence.md](examples/recorded-evidence.md)** — raw JSON lines from the tracking files

| Metric | Before | After | Speedup |
|--------|--------|-------|---------|
| Test suite | — | **6,025 passed / 0 failed / 10 ignored** | — |
| Linting | — | clippy 0 errors, 0 new warnings | — |
| PDF inspection (430 pages) | 52,897 ms | 7,531 ms | **7.0x** |
| Call-graph cache keying | 1,472.4 µs | 0.01 µs | **~147,000x** |
| Soak test Q8 round-trip | 120 ms | 80 ms | **1.5x** |
| Security scan recall | — | **1.0 (9/9)** | — |
| Security false-positive rate | — | **0.0** | — |

---

## Tech stack

Rust 1.85+ · MCP (Model Context Protocol) · tree-sitter · rayon · tokio · serde

## Built with

This project is developed largely using **GLM-5.2 and GLM-5.3** — the codebase, experiments, and benchmarks on this page were produced with Z.ai models driving the development workflow.

---

## Activity

- GitHub profile: [github.com/mx3618](https://github.com/mx3618)
- Development happens in a private repository; milestones and release notes are summarized in [examples/changelog.md](examples/changelog.md)

---

## FAQ

**Can I use it?**
The source is private. Contact the maintainer via the GitHub profile for access.

**Is this a public API demo?**
The `examples/` folder shows the *output formats* the tool produces — not the implementation.

**Why private?**
The project is under active development; the maintainer prefers to release a stable, documented version first.
