# AGENTS.md — apus

> Swift-native code intelligence for AI coding agents: Xcode Index Store +
> SwiftSyntax knowledge graph, exposed over MCP.

## Purpose

Apus is a purpose-built Swift/Xcode code intelligence tool that replaces
tree-sitter with native Apple tooling for richer analysis. It builds a
knowledge graph of a Swift codebase (symbol resolution, conformances,
extensions, call graphs) and serves it through the Model Context Protocol
(MCP) so coding assistants can search, navigate, and reason about Swift
structure the way the compiler does.

MCP tools: `search`, `lookup`, `context`, `impact`, `conformances`,
`extensions`.

## Required

- macOS 14+, Xcode installed, Swift 6.0+.
- The target project must have been built in Xcode at least once — Apus reads
  the Index Store that Xcode produces during builds.
- Run `apus index` after building, and re-index when code changes
  significantly. The MCP server reads the persisted graph and starts instantly.
- Graph databases live at `~/.apus/<project-hash>/graph.db`.

## Layout

- **ApusCore** — Graph types (`GraphNode`, `GraphEdge`, `KnowledgeGraph`
  protocol), SQLite persistence (GRDB + FTS5), HybridGraph
- **ApusProject** — SPM / Xcode / XcodeGen project parsers
- **ApusIndexStore** — C API wrappers for Xcode's Index Store
  (`libIndexStore.dylib` via CIndexStore system library)
- **ApusSyntax** — SwiftSyntax-based source file parsing
- **ApusAnalysis** — Codebase analysis, plan generation, graph export
  (DOT/Mermaid/JSON), web explorer
- **ApusMCP** — MCP server exposing graph via tools (search, lookup, context,
  impact, conformances, extensions)
- **ApusCLI** — `apus index`, `apus query`, `apus analyze`, `apus plan`,
  `apus serve`, `apus export`, `apus explore`, `apus checkpoint`, `apus update`
- `Sources/`, `Tests/`, `Package.swift` — SPM package layout

Indexing pipeline: Project Discovery → Index Store → SwiftSyntax → Graph
Merge → SQLite Persistence. The MCP server loads the persisted graph into
memory and serves queries over stdio.

## Commands

```bash
swift build                          # Build all targets
swift test --filter ApusMCPTests     # Run MCP server tests
swift test --filter ApusCoreTests    # Run core graph tests
swift test                           # Run all tests

apus index [path]                    # Index a Swift project (after Xcode build)
apus index . --include-tests         # Include test targets
apus index . --force --verbose       # Force full re-index
apus query <search> --path           # Query the knowledge graph
apus query "parse" --kind function   # Filter by kind
apus query "Model" --json            # JSON output for scripting
apus serve --path                    # Start MCP server on stdio

apus analyze .                       # Full analysis report to stdout
apus analyze . --sections overview,hotspots
apus analyze . --output report.md    # sections: overview, architecture,
                                     # typesystem, api, dependencies,
                                     # hotspots, patterns

apus export . --format dot|mermaid|json
apus explore .                       # Interactive web explorer (Cytoscape.js)
apus checkpoint diff|save|list .     # Graph metrics snapshots
apus plan .                          # Implementation plan (context + tasks)
apus update                          # Rebuild release binary in place
```

MCP client config (`.claude/settings.json` or project-level):

```json
{
  "mcpServers": {
    "apus": {
      "command": "apus",
      "args": ["serve", "--path", "/path/to/your/project"]
    }
  }
}
```

Install: `swift build -c release` then copy `.build/release/apus` onto PATH.

## Conventions

- Index after building the project in Xcode; the Index Store must be populated.
- `apus index` automatically saves a checkpoint with a one-line summary of what
  changed since the previous index.
- Checkpoints capture node/edge/file counts, public API surface, and breakdowns
  by symbol kind, target, and edge type.
- Exports can be filtered by target or node kind
  (`--target ApusCore --exclude-kind file`).
- Architecture diagram and full CLI surface live in `README.md`.

## Related

- `README.md` — full product and CLI documentation
- `CLAUDE.md` — Claude Code guidance (content absorbed into this file)
- [Model Context Protocol](https://modelcontextprotocol.io)
