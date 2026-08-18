# Example: generated INDEX.md

`indxr index` walks a workspace and writes a navigable structural map. The header format is:

```markdown
> Generated: 2026-08-18 06:32:04 UTC | Workspace: cargo | Members: 1 | Files: 433 | Lines: 440845
```

The full file contains a directory tree, an API surface listing, and a hotspot report. This is an annotated excerpt for a small toy project:

## Why this beats the normal way

| Task | Normal way | `INDEX.md` |
|---|---|---|
| Orient in an unfamiliar repo | open files one by one, build a mental map | tree + API surface + hotspots on one page |
| Find the public API of a module | read every file, note every `pub` | `## API Surface` table with signatures |
| Find what's complex / risky | vibes, or long review meetings | `## Hotspots` ranked by cyclomatic complexity |
| Onboard a new dev / agent | "start here, then read that..." | the index *is* the map — point at it |

## Directory Structure

```
notes-app/
  src/ (9 files)
    models/ (2 files)
      note.rs
      tag.rs
    storage/
      db.rs
    main.rs
    cli.rs
  tests/
    integration.rs
  README.md
  Cargo.toml
```

## API Surface (public)

| Declaration | Kind | File | Signature |
|---|---|---|---|
| `Note` | struct | `src/models/note.rs:4` | `pub struct Note { id: u64, title: String, body: String, tags: Vec<String> }` |
| `Note::new` | fn | `src/models/note.rs:12` | `pub fn new(title: impl Into<String>) -> Self` |
| `Note::add_tag` | method | `src/models/note.rs:28` | `pub fn add_tag(&mut self, tag: &str)` |
| `Tag` | struct | `src/models/tag.rs:3` | `pub struct Tag { name: String, color: Option<String> }` |
| `Db` | struct | `src/storage/db.rs:6` | `pub struct Db { path: PathBuf, conn: SqliteConn }` |
| `Db::open` | fn | `src/storage/db.rs:18` | `pub fn open(path: impl AsRef<Path>) -> Result<Self, DbError>` |
| `Db::save` | method | `src/storage/db.rs:41` | `pub fn save(&self, note: &Note) -> Result<(), DbError>` |
| `main` | fn | `src/main.rs:3` | `pub fn main() -> std::process::ExitCode` |
| `run_cli` | fn | `src/cli.rs:7` | `pub fn run_cli() -> Result<(), CliError>` |

## Hotspots (complexity)

| Symbol | File:line | Complexity | Body lines |
|---|---|---|---|
| `Db::save` | `src/storage/db.rs:41` | 12 | 46 |
| `run_cli` | `src/cli.rs:7` | 9 | 88 |
| `Db::open` | `src/storage/db.rs:18` | 6 | 31 |

## Stats line (verbatim shape)

```
> Generated: 2026-08-18 06:32:04 UTC | Workspace: cargo | Members: 1 | Files: 433 | Lines: 440845
```
