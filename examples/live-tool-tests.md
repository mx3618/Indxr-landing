# Live tool tests — real outputs

**What this is:** every output below is a **real, live run** of the find / search / analysis tools through the vecto-indxr MCP server on 2026-08-18, executed against the project's own codebase (433 files, 440K lines, 11,402 functions). Nothing is fabricated or hand-edited — results are trimmed to top matches only.

## At a glance — normal way vs this project's tools

| Task | Normal way | This tool | Result you see below |
|---|---|---|---|
| Find a symbol | `grep -rn "parse_files"` → open hits | `find` (mode=symbol) | §1 — declaration + signature, 4 matches |
| Find code *about* a concept | guess keywords, multiple greps | `search_semantic` (BM25) | §2 — 101 ranked candidates |
| Assess codebase health | vibes / CI status | `get_health` | §3 — 11,402 functions, 6,279 tests, hotspots |
| Who calls this function? | grep + open every file | `get_callers` | §4 — 6 exact call sites with file:line |
| What tests exist for it? | guess names, grep test dir | `get_related_tests` | §5 — 6 tests, ranked with confidence |
| See the call structure | read every function body | `get_call_graph` | §6 — Mermaid graph, line-level edges |
| What breaks if I change it? | mental model + full test run | `get_impact` | §7 — 21 files, 47 transitive, 32 tests |
| Understand a function | open file, read whole body | `explain_symbol` | §8 — interface + docs, no source read |

## 1. `find` — symbol lookup (mode: symbol)

```
find(query="parse_files", mode="symbol")
  -> 4 matches. Top results:

  src/indexer.rs:258
    pub fn parse_files(files: &[&FileEntry], cache: &Cache, registry: &ParserRegistry)
    -> (Vec<ParseResult>, usize)
    doc: "Parse a batch of files in parallel via rayon, with incremental cache lookup..."

  src/indexer.rs:306  parse_files_sequential  (test baseline)
  src/wiki/generate.rs:842  parse_files_at_ref  (diff-time re-parse)
```

## 2. `search_semantic` — concept search (BM25)

```
search_semantic(query="caching analysis results", algorithm=bm25)
  -> 101 candidates. Top 3:

  1. src/semantic/mod.rs:2863  results: Vec<SemanticMatch>        score 0.555
  2. src/mcp/helpers.rs:5689   test_detect_result_type_hotspot_analysis  score 0.546
  3. src/mcp/tools/pdf.rs:2618 test_pdf_search_cached_results_identical  score 0.539
```

## 3. `get_health` — codebase health

```
get_health(detail=summary)
  -> health 69/100 (fair)
  -> 11,402 functions | 6,279 tests | 971 hotspots (8.5%)
  -> docs coverage 40.2% | public API: 1,465
  -> complexity: avg 3.4 / median 1 / max 202 / p90 8

Hottest files:
  src/pdf_inspector/extractor/xobjects.rs      avg complexity 24.0 (max 115)
  src/mcp/tools/experiments/transfer.rs        avg complexity 19.9
  src/mcp/tools/dispatch.rs                    avg complexity 15.7
```

## 4. `get_callers` — who calls it

```
get_callers(symbol="parse_files")
  -> 6 call sites:
     src/commands/default.rs:229  run           (call)
     src/indexer.rs:460           build_index   (call)
     src/indexer.rs:1776          test_parallel_index_same_result
     src/indexer.rs:1808          test_parallel_index_empty
     src/indexer.rs:1830          test_parallel_index_single_file
     src/indexer.rs:1886          test_parallel_same_result
```

## 5. `get_related_tests` — test discovery

```
get_related_tests(symbol="parse_files")
  -> 6 tests:
     src/indexer.rs:306  parse_files_sequential          (defining_file)
     src/indexer.rs:1776 test_parallel_index_same_result (body_reference)
     src/indexer.rs:1808 test_parallel_index_empty       (body_reference)
     src/indexer.rs:1830 test_parallel_index_single_file (body_reference)
     ...
```

## 6. `get_call_graph` — Mermaid output (trimmed to 8 nodes)

```
graph TD
  n0["parse_files"]
  n0 -->|L282| n2["collect"]
  n0 -->|L287| n2
  n0 -->|L289| n5["parse_one_file"]
  n0 -->|L289| n1["cache"]
  n0 -->|L289| n6["registry"]
  n0 -->|L289| n3["file_entry"]
  n0 -->|L293| n4["len"]
```
(20 nodes / 13 edges total for this symbol; 5 method-call edges excluded as unresolved — reported by the tool itself)

## 7. `get_impact` — blast radius (depth 2)

```
get_impact(symbol="parse_files", depth=2)
  -> blast radius: 21 files
  -> 8 direct dependents + 47 transitive
  -> 32 affected tests

Direct:
  src/commands/default.rs:229  run
  src/indexer.rs:460           build_index
  src/indexer.rs:1776+         (4 test fns)

Transitive (depth 2):
  src/commands/dev.rs:128  run_diff
  src/commands/mod.rs:72   dispatch
  src/external_index.rs:327 add_external_repo
  src/indexer.rs:609       build_workspace_index
  ...

Affected tests include: test_build_index_incremental_modified,
test_build_index_excludes_large_files, test_build_index_empty_directory, ...
```

## 8. `explain_symbol` — interface + doc without reading source

```
explain_symbol(name="parse_files", detail=summary)
  -> src/indexer.rs:258  pub fn parse_files(
        files: &[&FileEntry], cache: &Cache, registry: &ParserRegistry
     ) -> (Vec<ParseResult>, usize)
  -> doc: parallel parse via rayon with incremental cache lookup;
     thread-local tree-sitter Parser per worker; returns (results, failed);
     order NOT guaranteed (work-stealing).
```

## Takeaway

These are the answers a coding agent gets **in milliseconds** — the index is built once (`indxr index`), and every query above is a lookup against it, not a file read. The same tools powered the 6,025-test suite and the benchmarks in [test-results.md](test-results.md).
