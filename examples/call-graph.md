# Example: call graph & impact analysis

## Why this beats the normal way

| Task | Normal way | This tool |
|---|---|---|
| See who calls whom | open every file, read every body, draw it by hand | `get_call_graph` → Mermaid graph with line-level edges |
| Estimate the blast radius of a change | "I think it's used here and here..." then the full test run fails | `get_impact` → direct + transitive dependents, affected tests, before you edit |
| Review what a PR structurally changed | scroll through the whole diff | `get_diff_summary` → declaration-level + / ~ / − changes |

## 1. `get_call_graph` (Mermaid output)

```
graph TD
  A["main src/main.rs:3"] --> B["run_cli src/cli.rs:7"]
  B --> C["Db::open src/storage/db.rs:18"]
  B --> D["Note::new src/models/note.rs:12"]
  D --> E["Tag::new src/models/tag.rs:9"]
  B --> F["Db::save src/storage/db.rs:41"]
  F --> G["Db::load src/storage/db.rs:58"]
  G --> D
```

## 2. Blast radius — `get_impact`

```
get_impact(symbol="Db::save", depth=3)

Direct dependents (2):
  - src/cli.rs:44   run_cli
  - tests/integration.rs:12  test_save_and_load_roundtrip

Transitive dependents (1 more at depth 2):
  - src/main.rs:3   main  (via run_cli)

Affected files (2):
  - src/cli.rs
  - src/storage/db.rs

Risk: medium — public API, 3 call sites, covered by 1 test
```

## 3. Structural PR diff — `get_diff_summary`

```
since_ref: HEAD~2
  src/models/note.rs   +1 ~2
    + pub fn archive(&self) -> bool          (added)
    ~ pub fn add_tag(&mut self, tag: &str)   (changed: now returns Result<(), TagError>)
  src/cli.rs           ~1
    ~ run_cli  (adapted to new add_tag signature)
```
