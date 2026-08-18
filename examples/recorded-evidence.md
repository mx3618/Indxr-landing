# Recorded evidence — verbatim excerpts

Excerpts are copied **verbatim** from the project's tracking store and verified through the vecto-indxr MCP server (`track_metric list` / `benchmark_list`, 2026-08-18). Every entry carries its own timestamp and sequence id.

## 1. `benchmarks.jsonl` — performance before/after pairs

```json
{"name":"call_graph.fingerprint_keying_us","value":1472.4,"unit":"us","timestamp":"2026-08-16T21:17:15.142358836+00:00","seq":18,"label":"old-compute_fingerprint_refs-per-call-110K-edges","metadata":null}
{"name":"call_graph.fingerprint_keying_us","value":0.01,"unit":"us","timestamp":"2026-08-16T21:17:20.187953214+00:00","seq":19,"label":"new-memoized-search-fingerprint-per-call","metadata":null}
{"name":"pdf_extract_430p_ms","value":52897.0,"unit":"ms","timestamp":"2026-08-16T22:08:19.863078548+00:00","seq":20,"label":"before-rayon","metadata":null}
{"name":"pdf_extract_430p_ms","value":7531.0,"unit":"ms","timestamp":"2026-08-16T22:08:19.926451320+00:00","seq":21,"label":"after-rayon","metadata":null}
{"name":"run_tests.duration_ms","value":55505.0,"unit":"ms","timestamp":"2026-08-18T06:54:24.495200838+00:00","seq":22,"label":null,"metadata":null}
{"label":"soak186-q8-before","name":"soak186-q8-bench","seq":14,"timestamp":"2026-08-15T10:57:28.215630311+00:00","unit":"ms","value":120.0}
{"label":"soak186-q8-after","name":"soak186-q8-bench","seq":15,"timestamp":"2026-08-15T10:57:30.414191266+00:00","unit":"ms","value":80.0}
```

## 2. `metrics.jsonl` — counters (verbatim fields)

```json
{"id":"mt_000004","name":"sec_recall","tags":["security","benchmark-L1","TPR-9-of-9"],"timestamp":"2026-08-08T18:32:08.093565840+00:00","value":1.0}
{"id":"mt_000003","name":"sec_fp_rate","tags":["security","benchmark-L1","corpus-9pos-7neg"],"timestamp":"2026-08-08T18:32:08.048007234+00:00","value":0.0}
{"id":"mt_000007","name":"sec_scan_time_ms","tags":["security","measured-live","scan_patterns-focus-security","median-3-runs"],"timestamp":"2026-08-08T18:34:14.427474+00:00","value":7845.0}
{"id":"mt_000012","name":"q8_soak_ok_rate","tags":[],"timestamp":"2026-08-11T19:00:31.027415494+00:00","value":100.0}
{"id":"mt_000018","name":"soak186-q8-latency_ms","tags":[],"timestamp":"2026-08-15T14:55:28.807828670+00:00","value":12.5}
{"id":"mt_000020","name":"decimal_digits.speedup_u64max_x","value":5.65,"tags":["cache","serialization","optimization"],"timestamp":"2026-08-16T20:47:06.018511399+00:00"}
```

## 3. CHANGELOG — per-version Verification section (v0.80.1)

> - `prepare_release` 12/12 and `generate_changelog` 12/12 tests pass; full lib run shows only the 16 known pre-existing failures (14 pdf vector_grid + 2 accuracy-suite, both filed as work items). fmt clean; no new clippy warnings in touched files.

## 4. INDEX.md header (structural index stats)

```markdown
> Generated: 2026-08-18 06:32:04 UTC | Workspace: cargo | Members: 1 | Files: 433 | Lines: 440845
```

## 5. Session record (full-suite outcome)

> Full suite 6025 passed / 0 failed / 10 ignored; clippy 0 errors, no new warnings. (session record, 2026-08-17)

## Traceability

- `seq` ids in `benchmarks.jsonl` are monotonically increasing — later entries supersede earlier ones with the same metric name
- Perf claims map to real commits: `9f89334b` (rayon PDF parallelization), `e4afe7a9` (call-graph fingerprint memoization)
- All timestamps are UTC
