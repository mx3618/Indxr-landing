# Example: auto-drafted changelog

`generate_changelog` reads git history, classifies commits by conventional-commit type, and renders a changelog — no hand-writing. Section structure below matches the project's real CHANGELOG.md.

## v0.80.1 (2026-08-18) — single-file changelog history

### Changed
- **`prepare_release`** — the apply path no longer creates `changelog/changelog-<ver>.md`; it prepends the generated draft as the newest `## <version>` section of the root `CHANGELOG.md`.
- **`generate_changelog`** — `next_steps` now points at `CHANGELOG.md` instead of the removed folder.
- **`plan_track`** — `steps` accepts strings or `{id,name,status}` objects; the `step` action can define/extend a task's step list (merge by name).

### Verification
- `prepare_release` 12/12 and `generate_changelog` 12/12 tests pass; full lib run shows only the 16 known pre-existing failures (14 pdf vector_grid + 2 accuracy-suite, both filed as work items). fmt clean; no new clippy warnings in touched files.

## v0.80.0 (2026-08-18) — built-in work tracking (`plan_track`)

Work-item tracking moves into the binary. The `_plan/` folder system is retired; all work now lives on the task board.

### Added
- **`plan_track` tool (187th tool)** — one command for the whole work lifecycle: `next` / `add` / `start` / `step` / `status` / `done` / `history` / `check` / `claim` / `import`.
- **Anti-skip done gate** — `done` is refused unless: every step is done or skipped-with-reason, checklist ticked, every metric `after ≥ baseline` (regression blocks) and `after ≥ target` or a recorded reason, and git evidence shows ≥1 commit per step with the `Task: <id>` trailer.
- **Multi-agent claiming** — `owns` path-prefix collision gate + git-tag claiming (`claim-<task-id>`).

### Removed
- `_plan/` folder system (plan/pending/done + INDEX + scripts). Historical reports archived under `docs/plans-archive/`.

---

**Why not hand-write release notes?** Hand-written notes drift from reality. This tool derives them from the commit history itself — the same history that backs the test results in [test-results.md](test-results.md) and the benchmarks in [recorded-evidence.md](recorded-evidence.md).
