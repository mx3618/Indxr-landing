# Example: MCP tool calls — request/response

Sample MCP (Model Context Protocol) interactions — how an AI agent talks to the server. Shapes are real (from live runs); data is trimmed.

## 1. Find a symbol

```
Request:
  { "method": "tools/call", "params": { "name": "find",
      "arguments": { "query": "parse_files", "mode": "symbol" } } }

Response (trimmed):
  { "status": "ok", "matches": 4,
    "symbols": [
      { "name": "parse_files", "kind": "fn",
        "file": "src/indexer.rs", "line": 258,
        "signature": "pub fn parse_files(files: &[&FileEntry], cache: &Cache, registry: &ParserRegistry) -> (Vec<ParseResult>, usize)",
        "doc_comment": "Parse a batch of files in parallel via rayon, with incremental cache lookup..." },
      ...
    ] }
```

## 2. Explain a symbol (without reading source)

```
Request:
  { "method": "tools/call", "params": { "name": "explain_symbol",
      "arguments": { "name": "parse_files", "detail": "summary" } } }

Response (trimmed):
  { "status": "ok", "name": "parse_files", "kind": "fn",
    "file": "src/indexer.rs", "line": 258,
    "signature": "pub fn parse_files(files: &[&FileEntry], cache: &Cache, registry: &ParserRegistry) -> (Vec<ParseResult>, usize)",
    "doc_comment": "... parallel parse via rayon ... thread-local tree-sitter Parser per worker ...
                  returns (results, failed) ... order NOT guaranteed (work-stealing) ..." }
```

## 3. Health check

```
Request:
  { "method": "tools/call", "params": { "name": "get_health",
      "arguments": { "detail": "summary" } } }

Response (trimmed):
  { "status": "ok", "health_score": 69,
    "total_functions": 11402, "test_count": 6279,
    "high_complexity_count": 971, "documented_pct": 40.2,
    "public_api_count": 1465 }
```

## 4. The envelope (every tool wraps results)

```
{ "status": "ok",
  "next_steps": [ ... ],     // what to call next, with args
  "chain": [ ... ],          // suggested tool sequence
  ... tool-specific fields }
```

Every response carries routing hints — an agent never has to guess the next call. That guidance layer is itself tested (see [test-results.md](test-results.md): routing tests 12/12).
