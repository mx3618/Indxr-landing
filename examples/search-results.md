# Example: search results

Output shapes produced by the search & analysis tools (illustrative data).

## 1. Symbol lookup — `lookup_symbol` / `find`

```
name: "Note::add_tag"
  -> declared in src/models/note.rs:28
     pub fn add_tag(&mut self, tag: &str)
```

## 2. Callers — `get_callers`

```
get_callers(symbol="Note::add_tag")
  -> 3 call sites
     1. src/cli.rs:44   note.add_tag("work")
     2. src/cli.rs:52   note.add_tag("personal")
     3. src/storage/db.rs:67   note.add_tag(tag.name)
```

## 3. Type flow — `get_type_flow`

```
get_type_flow(type_name="Note", direction="both")

Producers (return Note):
  - src/models/note.rs:12  Note::new(title) -> Self
  - src/storage/db.rs:58   Db::load(id) -> Result<Note, DbError>

Consumers (accept Note):
  - src/storage/db.rs:41   Db::save(&self, note: &Note)
  - src/cli.rs:30          render(note: &Note)
```

## 4. Semantic search — `find` (concept query)

```
find(query="persist notes to disk", mode="relevant")
  -> 1. src/storage/db.rs        (score 92)  Db, save, open, SqliteConn
  -> 2. src/models/note.rs       (score 71)  Note, Tag
  -> 3. tests/integration.rs     (score 55)  roundtrip test
```

## 5. Related tests — `get_related_tests`

```
get_related_tests(symbol="Db::save")
  -> tests/integration.rs:12  test_save_and_load_roundtrip   (confidence: body_reference)
  -> src/storage/db.rs:99     #[cfg(test)] mod tests         (same_file_only)
```
