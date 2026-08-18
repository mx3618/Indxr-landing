# Real recorded test results

All figures below are read from the project's own tracking records — `benchmarks.jsonl`, `metrics.jsonl`, the changelog, and session records. Nothing is extrapolated. Raw excerpts are in [recorded-evidence.md](recorded-evidence.md).

## Test suite

| Result | Value | Recorded |
|---|---|---|
| Full suite | **6,025 passed / 0 failed / 10 ignored** | session record, 2026-08-17 |
| Clippy | **0 errors, no new warnings** | session record, 2026-08-17 |
| Latest full-suite duration | **55,505 ms** | `benchmarks.jsonl` seq 22, 2026-08-18 |
| Full-suite duration (earlier) | 92,016 ms → 46,117 ms | `benchmarks.jsonl` seq 12-13, 2026-08-14 |
| `prepare_release` | 12/12 tests pass | CHANGELOG v0.80.1 |
| `generate_changelog` | 12/12 tests pass | CHANGELOG v0.80.1 |

## Performance (measured, before/after)

| Metric | Before | After | Speedup | Evidence |
|---|---|---|---|---|
| PDF inspection, 430 pages | 52,897 ms | 7,531 ms | **7.0x** | `benchmarks.jsonl` seq 20-21, commit `9f89334b` (rayon per-page parallel) |
| Call-graph cache keying | 1,472.4 µs | 0.01 µs | **~147,000x** | `benchmarks.jsonl` seq 18-19, commit `e4afe7a9` (memoized fingerprint) |
| Soak test Q8 round-trip | 120 ms | 80 ms | **1.5x** | `benchmarks.jsonl` seq 14-15, 2026-08-15 |
| Soak test Q8 throughput | 1,109 tps | 1,200 tps | **+8.2%** | `benchmarks.jsonl` seq 16-17, 2026-08-15 |
| Decimal digits serializer (u64 max) | — | 5.65x | **5.65x** | `metrics.jsonl` mt_000020, 2026-08-16 |

## Security & reliability

| Metric | Value | Recorded |
|---|---|---|
| Security scan recall | **1.0 (9 of 9 positives)** | `metrics.jsonl` mt_000004, 2026-08-08 |
| Security scan false-positive rate | **0.0** | `metrics.jsonl` mt_000003, 2026-08-08 |
| Full security scan time (median of 3) | 7,845 ms | `metrics.jsonl` mt_000007, 2026-08-08 |
| Soak test Q8 ok rate | **100%** | `metrics.jsonl` mt_000012, 2026-08-11 |
| Soak test 186 queries, median latency | 12.5 ms | `metrics.jsonl` mt_000018, 2026-08-15 |
| Soak test 186 queries, throughput | 99.7% | `metrics.jsonl` mt_000019, 2026-08-15 |
| Codebase health score | 69/100 | `metrics.jsonl` mt_000013, 2026-08-12 |

## How these are tracked

- `benchmarks.jsonl` — every run is appended with a timestamp and seq id; before/after pairs share the metric name with labels
- `metrics.jsonl` — counters with tags and timestamps
- CHANGELOG.md — per-version Verification sections
- Session records (STATE.md) — full-suite and clippy outcomes
