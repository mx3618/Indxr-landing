# GLM-5.3 × Vecto-Indxr — Demo & Token Grant Plan

> Why this project is the strongest vehicle for the GLM-5.3 community moment: it is **built with GLM models, for GLM-class coding agents**, and it directly amplifies every headline strength of GLM-5.3.

## 1. Alignment — GLM-5.3's strengths × what we amplify

| GLM-5.3 headline strength | Our demo uses it to | Expected result |
|---|---|---|
| **Terminal-Bench 3.0 #1** (28.3%, top published) | Agent fixes cross-file bugs in a 440K-line repo via terminal + MCP | measurable task-completion delta vs no-index baseline |
| **Token efficiency** (34.5% at ~75K out tokens vs 23.4% at 96K for 5.2) | surgical-context queries replace full-file dumps | tokens-per-task reduction with the same or better results |
| **1M-token context** | whole-repo planning: the index makes 1M context spend count | one-shot repo-scale refactor/onboarding task |
| **Cyber defense** (CyberGym 84.5%, exploit scores 2x GLM-5.2) | `safety_check` + `security_reach` vulnerability triage pipeline | "sink → reachability proof → fix" in one agent run |
| **Agentic coding** (DeepSWE 66.9%) | agents built with our MCP tools do more of the loop autonomously | 6,025-test suite stays green through the whole demo |
| **Dogfooding** | this project itself was built largely with GLM-5.2/5.3 | "a tool for coding agents, written by coding agents" |

## 2. The demo (what gets recorded & posted)

**Title:** *"GLM-5.3 found a reachable vulnerability in my 440K-line repo in 90 seconds"*

1. **Setup (30s):** GLM-5.3 in a terminal, connected to our 180-tool MCP server over a 433-file / 440K-line Rust codebase
2. **The hunt (60s):** `safety_check` flags a command-execution sink → `security_reach` walks the reverse call graph → 2 attacker-reachable paths proven → GLM-5.3 verdicts severity and writes the fix
3. **The proof (30s):** `cargo test` — 6,025 passed / 0 failed; diff shown
4. **The point (30s):** token comparison — full-file dump vs surgical context (real counts from the run)

**Format:** 30-60s vertical clip (thread hook) + 2-3 min full video + thread with receipts (5 posts).

## 3. Token grant plan (500M–1B tokens) — what we will build

The grant is not for show — it funds the experiments below, each producing a public artifact that keeps the community moment alive:

| # | Experiment | Public artifact |
|---|---|---|
| E1 | **Index-vs-dump benchmark**: N=30 repo tasks (find/fix/refactor), same GLM-5.3 effort level, with vs without structural index — tokens, time, pass rate | open dataset + blog post |
| E2 | **1M-context repo agent**: one-shot multi-file refactor tasks on a 440K-line repo | video series |
| E3 | **Cyber triage pipeline**: GLM-5.3 + `safety_check`/`security_reach` on a synthetic vuln corpus (9 positives / 7 negatives, recall 1.0 / FP 0.0 baseline already recorded) | open benchmark |
| E4 | **Effort-level study**: low/high/max reasoning effort × task difficulty, with token accounting | data table + thread |
| E5 | **GLM-5.3-native docs**: agent onboarding a newcomer into a 440K repo using only index queries | tutorial |

Every result is recorded in the same tracked store used for [test-results.md](examples/test-results.md) — timestamped, seq-ordered, queryable via MCP. No claims without receipts.

## 4. Why this beats a typical demo submission

| Typical demo | This project |
|---|---|
| A wrapper around one API call | 180 MCP tools, 6,025 tests, 11,402-function repo, recorded benchmarks |
| Demo dies after the event | token grant funds 5 public experiments; landing page is the living artifact |
| "It works on my machine" | every claim carries a timestamped record (see examples/) |
| Generic GLM wrapper | amplifies GLM-5.3's exact strengths (terminal, tokens, 1M context, cyber) |
| Built *with* GLM, sometimes | **built largely by GLM-5.2/5.3, about GLM-class agents** — dogfooding at scale |

## 5. Materials already public

- Landing: [github.com/mx3618/Indxr-landing](https://github.com/mx3618/Indxr-landing)
- Real test results: `examples/test-results.md` (6,025 passed / 0 failed; PDF 7.0x; cache-key ~147,000x; security recall 1.0)
- Verbatim evidence: `examples/recorded-evidence.md` (raw JSON lines, timestamps, seq ids)
- Live tool runs: `examples/live-tool-tests.md` (8 tools, measured against 440K lines, with normal-way-vs-tool comparison)
