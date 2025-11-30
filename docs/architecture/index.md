---
layout: default
title: Architecture
nav_order: 4
has_children: true
---

# Architecture

Technical overview of Context Bridge's layered architecture.

---

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    AI Assistant (MCP Client)                │
└─────────────────────────────┬───────────────────────────────┘
                              │ MCP Protocol
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     MCP Server Layer                        │
│              (Context, Query, Implementation Tools)         │
├─────────────────────────────────────────────────────────────┤
│                    Agent Abstraction                        │
│         (Registry → Factory → Executor → Adapters)          │
├─────────────────────────────────────────────────────────────┤
│                     Storage Layer                           │
│              (SQLite + Drizzle ORM + Cache)                 │
├─────────────────────────────────────────────────────────────┤
│                     CLI Interface                           │
│                     (Commander.js)                          │
├─────────────────────────────────────────────────────────────┤
│                     TUI Dashboard                           │
│                   (Ink/React + Zustand)                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Layer Summary

| Layer | Technology | Purpose |
|-------|------------|---------|
| **MCP Server** | @modelcontextprotocol/sdk | Exposes 9 tools via MCP protocol |
| **Agent Abstraction** | Execa, Adapter Pattern | Spawn and manage AI sub-agents |
| **Storage** | SQLite, Drizzle ORM | Persist data, cache queries |
| **CLI** | Commander.js | Command-line interface |
| **TUI** | Ink, React, Zustand | Real-time monitoring dashboard |

---

## Data Flow

```
User Query → MCP Tool → Agent Factory → Sub-Agent Spawn → Result → Cache → Response
```

1. **AI assistant** receives user request
2. **MCP tool** invoked via protocol
3. **Agent factory** creates appropriate adapter
4. **Sub-agent** spawns in repository directory
5. **Results** captured and stored in SQLite
6. **Response** returned to AI assistant

---

## Key Design Patterns

| Pattern | Usage |
|---------|-------|
| **Adapter** | Pluggable AI backends (Claude, Codex, Aider) |
| **Factory** | Dynamic adapter instantiation |
| **Event-Driven** | Real-time TUI updates via typed EventEmitter |
| **Clean Architecture** | Dependency inversion, layered separation |

---

## Detailed Pages

- [Agent System](./agent-system.md) - Sub-agent spawning and adapters
- [Storage Layer](./storage-layer.md) - Database schema and caching
- [TUI Dashboard](./tui-dashboard.md) - Terminal UI architecture

---

## See Also

- [MCP Tools](../mcp-tools/) - Tool documentation
- [CLI Reference](../cli/) - Command-line interface
- [Configuration](../getting-started/configuration.md) - Configuration options
