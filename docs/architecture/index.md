---
layout: default
title: Architecture
nav_order: 4
has_children: true
---

# Architecture

Technical deep-dive into Context Bridge's internal architecture, design patterns, and implementation details.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              AI Assistant                                    │
│                            (MCP Client)                                      │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │ MCP Protocol (JSON-RPC)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Context Bridge Server                                │
│                            (MCP Server)                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐  │
│  │   Tool Layer     │  │   Tool Layer     │  │      Tool Layer          │  │
│  │  (Context Tools) │  │  (Query Tools)   │  │  (Implementation Tools)  │  │
│  └────────┬─────────┘  └────────┬─────────┘  └────────────┬─────────────┘  │
│           │                     │                          │                │
│           ▼                     ▼                          ▼                │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         Agent Layer                                    │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │ │
│  │  │  Registry   │→ │   Factory   │→ │  Executor   │→ │   Adapter   │   │ │
│  │  │ (Discovery) │  │ (Creation)  │  │ (Spawning)  │  │  (Claude)   │   │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│           │                                                                 │
│           ▼                                                                 │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                        Storage Layer                                   │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │ │
│  │  │   Adapter   │→ │   Schema    │→ │   Cache     │→ │   Events    │   │ │
│  │  │  (SQLite)   │  │  (Drizzle)  │  │  (Query)    │  │  (Updates)  │   │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Core Components

| Component | Purpose | Key Technologies |
|-----------|---------|------------------|
| [Agent System](./agent-system) | Spawn and manage AI sub-agents | Execa, Adapter Pattern |
| [Storage Layer](./storage-layer) | Persist data, cache queries | SQLite, Drizzle ORM |
| [TUI Dashboard](./tui-dashboard) | Real-time monitoring interface | Ink, React, Zustand |

---

## Directory Structure

```
context-bridge/
├── src/
│   ├── core/                    # Core interfaces and types
│   │   ├── interfaces/          # AgentAdapter, StorageAdapter
│   │   ├── types/               # Shared type definitions
│   │   └── events/              # Event emitter and types
│   │
│   ├── adapters/                # Interface implementations
│   │   ├── agent/               # Agent adapters (Claude, Codex, Aider)
│   │   └── storage/             # Storage adapters (SQLite)
│   │
│   ├── agent/                   # Agent system
│   │   ├── registry.ts          # Adapter discovery
│   │   ├── factory.ts           # Adapter creation
│   │   └── executor.ts          # Sub-agent spawning
│   │
│   ├── storage/                 # Storage system
│   │   ├── schema.ts            # Drizzle schema definitions
│   │   ├── migrations/          # Database migrations
│   │   └── cache.ts             # Query caching logic
│   │
│   ├── tools/                   # MCP tool implementations
│   │   ├── context/             # Context management tools
│   │   ├── query/               # Query tools
│   │   └── implementation/      # Implementation tools
│   │
│   ├── tui/                     # Terminal UI
│   │   ├── app.tsx              # Main Ink application
│   │   ├── screens/             # Screen components
│   │   ├── components/          # Reusable UI components
│   │   └── store/               # Zustand state management
│   │
│   ├── cli/                     # Command-line interface
│   │   ├── commands/            # Commander.js commands
│   │   └── index.ts             # CLI entry point
│   │
│   └── server/                  # MCP server
│       └── index.ts             # Server entry point
│
├── config/                      # Configuration files
└── docs/                        # Documentation
```

---

## Design Principles

### Adapter Pattern

Context Bridge uses the adapter pattern extensively to enable pluggable implementations:

```typescript
// Core interface (stable contract)
interface AgentAdapter {
  execute(options: AgentExecutionOptions): Promise<AgentResult>;
  getCapabilities(): AgentCapabilities;
}

// Implementations (swappable)
class ClaudeAdapter implements AgentAdapter { /* ... */ }
class CodexAdapter implements AgentAdapter { /* ... */ }
class AiderAdapter implements AgentAdapter { /* ... */ }
```

**Benefits:**
- Add new AI providers without modifying core code
- Swap storage backends (SQLite → PostgreSQL)
- Test with mock implementations

### Clean Architecture

The codebase follows clean architecture principles with clear separation of concerns:

```
┌─────────────────────────────────────────────┐
│              UI Layer                        │
│         (CLI, TUI, MCP Server)              │
├─────────────────────────────────────────────┤
│            Application Layer                 │
│    (Tools, Commands, Business Logic)        │
├─────────────────────────────────────────────┤
│             Domain Layer                     │
│      (Interfaces, Types, Events)            │
├─────────────────────────────────────────────┤
│          Infrastructure Layer                │
│    (Adapters, Database, File System)        │
└─────────────────────────────────────────────┘
```

**Dependency Rule:** Inner layers never depend on outer layers. Interfaces are defined in the domain layer; implementations live in infrastructure.

### Event-Driven Updates

Real-time updates flow through a typed EventEmitter system:

```typescript
// Typed event definitions
interface StorageEvents {
  'session:created': (session: Session) => void;
  'query:executed': (query: QueryResult) => void;
  'cache:hit': (key: string) => void;
  'cache:miss': (key: string) => void;
}

// Usage
storage.on('query:executed', (query) => {
  dashboard.refresh();
});
```

**Benefits:**
- Loose coupling between components
- Real-time TUI updates
- Easy to add new event handlers

---

## Security Considerations

### Tool Restrictions

Sub-agents have restricted tool access based on operation type:

| Operation | Allowed Tools | Restricted Tools |
|-----------|--------------|------------------|
| Query | Read, Glob, Grep, Bash (limited) | Edit, Write, MultiEdit |
| Implementation | All tools | None |

### Path Validation

All repository paths are validated before operations:

- Paths must be absolute
- Paths must exist on the filesystem
- Paths cannot escape the repository root (no `../`)

### Process Isolation

Sub-agents run in isolated processes:

- Separate process per sub-agent (via Execa)
- Working directory locked to repository root
- Configurable timeout (default: 5 minutes)
- Resource limits enforced

### Credential Handling

- No credentials stored in the database
- AI API keys inherited from environment
- Configuration files use YAML (no secrets in config)

---

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Runtime | Node.js 18+ | JavaScript runtime |
| Language | TypeScript (strict mode) | Type safety |
| MCP | @modelcontextprotocol/sdk | MCP protocol implementation |
| Database | SQLite + better-sqlite3 | Embedded database |
| ORM | Drizzle ORM | Type-safe database access |
| Process | Execa | Sub-agent process spawning |
| CLI | Commander.js | Command-line parsing |
| TUI | Ink (React for terminals) | Terminal UI framework |
| State | Zustand | Lightweight state management |
| Validation | Zod | Runtime type validation |
| Testing | Vitest | Unit and integration tests |

---

## In This Section

- [Agent System](./agent-system) - Sub-agent spawning, adapters, and tool restrictions
- [Storage Layer](./storage-layer) - Database schema, caching, and event system
- [TUI Dashboard](./tui-dashboard) - Terminal UI architecture and state management

---

## See Also

- [Configuration Reference](../getting-started/configuration) - Configuration options
- [MCP Tools](../mcp-tools) - Tool documentation
- [CLI Reference](../cli) - Command-line interface
