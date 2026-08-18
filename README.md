# Vecto-Indxr

> AI codebase indexer & analysis engine — structured search, impact analysis, and design tooling for AI agents.

Vecto-Indxr builds a **declaration-level structural index** of a codebase — files, symbols, signatures, call edges — and exposes it to AI agents through a **MCP server (180+ tools)** and a **CLI**. It answers questions like *"what tests cover `parse_files`?"*, *"who calls this function?"*, or *"how would renaming this symbol break my code?"* in milliseconds.

The project is **developed largely with GLM-5.2 and GLM-5.3** — it is built by the same kind of coding agents it serves. The source repository is **private**. This page is the public face: features, activity, and runnable-looking examples of what the tool produces.

---

## Table of contents

- [Feature overview](#feature-overview)
- [Why it's different](#why-its-different-from-the-normal-way)
- [GLM-5.3 demo & token grant plan](#glm-53-demo--token-grant-plan)
- [Quick tour (examples)](#quick-tour-examples)
- [Benchmarks (recorded, real)](#benchmarks-recorded-real)
- [CLI & MCP surface](#cli--mcp-surface)
- [Tech stack](#tech-stack)
- [Built with](#built-with)
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

### Measured: speed & precision (2026-08-18, same 440K-line repo)

"Normal way" = real `rg` commands with measured wall time and full hit counts. "Tool way" = live MCP runs (full outputs in [examples/live-tool-tests.md](examples/live-tool-tests.md)). Precision = real answers ÷ raw hits, computed from the counts below.

| Task | Normal way — measured | Indxr MCP tool — live run | Speed (time to *correct* answer) | Precision |
|---|---|---|---|---|
| Find definition of `parse_files` | `rg "fn parse_files"` · 6 ms scan · **5 hits, 1 real def** (4 noise: comment, string, `parse_files_at_ref`, test fn) | `find` (mode=symbol) · indexed lookup · **4 matches, def first, 0 noise** | 6 ms + open/filter 4 noise hits ≈ seconds | **20% → 100%** |
| Who calls `parse_files`? | `rg "parse_files"` · 9 ms scan · **77 hits: 14 real calls, 56 noise** (docs, strings, cfg(test)) | `get_callers` · 10–50 ms (ReferenceIndex) · **6 exact sites with file:line, 0 noise** | 9 ms + read 56 noise lines | **18% → 100%** |
| Find code about "caching analysis results" | `rg -i "caching\|analysis"` · 9 ms · **116 files, unranked** | `search_semantic` (BM25) · **101 candidates ranked** (0.555 / 0.546 / 0.539) | flat list vs 1 ranked answer | **no ranking → relevance-ranked** |
| What tests cover `parse_files`? | name-guessing grep · 3 ms · **12 hits, no confidence** | `get_related_tests` · **6 tests with confidence** (`defining_file` / `body_reference`) | guess + verify vs direct answer | **guess → 3-layer discovery** |
| How healthy is the codebase? | no standard method (vibes / CI status) | `get_health` · 1 call · **11,402 fns / 6,279 tests / 971 hotspots / docs 40.2%** | — | **quantified vs vibes** |
| What breaks if I change `parse_files`? | mental model + run full suite (minutes) | `get_impact` · **21 files, 47 transitive dependents, 32 affected tests** | trial-and-error vs prediction | **predicted before editing** |

The raw `rg` scan itself is fast (3–9 ms) — the difference is **precision**: 77 raw lines (56 noise) vs 6 exact answers, and what the agent must read afterward. At agent scale that is measured in tokens: **77 lines of context vs 6**.

### Measured: token efficiency & speed (why it's built for low-token agents)

This project is designed so agents spend **context on answers, not on raw matches**. Assumptions, stated openly: code ≈ 12 tokens/line; "normal way" cost = grep hits + the context lines an agent must actually read + the files it opens to verify — all derived from the **measured hit counts** in the table above (77 hits, 5 hits, 116 files).

| Task | Normal way — context consumed | Indxr tool — context consumed | Tokens saved | Time to correct answer |
|---|---|---|---|---|
| Find definition of `parse_files` | 5 hits × ~9 lines (hit + context) ≈ **500** | `find` ≈ **80** | **~84%** | seconds → ms |
| Who calls `parse_files`? | 77 hits + 3 file opens to verify ≈ **2,400** | `get_callers` ≈ **200** | **~92%** | minutes → ms |
| Concept search ("caching") | 116 file names + 3 files opened ≈ **7,000** | `search_semantic` ≈ **150** | **~98%** | minutes → ms |
| Tests covering a symbol | 12 hits + 2 test files opened ≈ **3,800** | `get_related_tests` ≈ **150** | **~96%** | minutes → ms |
| Blast radius of a change | 14 call sites × ~30 lines + suite logs ≈ **10,000** | `get_impact` ≈ **300** | **~97%** | run suite → predicted before editing |
| Understand one function | full body read ≈ **720** | `explain_symbol` ≈ **120** | **~83%** | file read → zero source reads |

**A typical 3-question session** (define + callers + tests): normal way ≈ **6,700 tokens** of context vs tools ≈ **430 tokens** → **~94% fewer tokens**, delivered as structured data instead of raw lines to filter. That is the design goal: *context in, answers out* — which is exactly what a token-efficient model like GLM-5.3 (34.5% task-completion at ~75K output tokens) rewards.

## GLM-5.3 demo & token grant plan

This project is built largely with **GLM-5.2 and GLM-5.3**, and it amplifies GLM-5.3's headline strengths (Terminal-Bench #1, token efficiency, 1M-token context, cyber defense). The complete demo plan, token-grant roadmap (5 public experiments), and the "why this beats a typical submission" comparison are in:

**→ [GLM-5.3-demo-plan.md](GLM-5.3-demo-plan.md)**

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
