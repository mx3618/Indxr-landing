# Vecto-Indxr

> AI codebase indexer & analysis engine — powering structured search, impact analysis, and design tooling for AI agents.

This is the public landing page for the Vecto-Indxr project. The source code repository is private; this page is the public face for links, activity, and releases.

## What it does

- **Structural code indexing** — builds a declaration-level index of a codebase (files, symbols, signatures, call edges) for fast AI-agent navigation
- **Semantic & structural search** — find files, symbols, callers, and patterns across 60+ languages
- **Impact analysis** — blast radius, call graphs, type flow, circular-dependency detection
- **Design & experiment tooling** — architecture drafts, design reviews, experiment tracking, wiki, dataset pipeline
- **MCP server** — exposes 180+ tools to AI agents (Claude, Cursor, Windsurf, Codex, Qwen, ...)
- **CLI + watch mode** — `indxr index`, `indxr serve`, `indxr doctor`, git structural diffing, changelog drafting

## Activity

- GitHub profile: [github.com/mx3618](https://github.com/mx3618)
- Development, experiments, and benchmarks are tracked privately; milestones and release notes are summarized below.

## Highlights

- 430-page PDF inspection: 52,897 ms → 7,531 ms (7.0x, rayon parallel per-page)
- Call-graph fingerprint keying: ~294,000x speedup on cache-key step
- 6,000+ test suite, 0 clippy errors

## Tech

Rust 1.85+ · MCP (Model Context Protocol) · rayon · tree-sitter
