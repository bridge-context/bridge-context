---
layout: default
title: Context Bridge
---

# Context Bridge

[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D18.0.0-brightgreen)](https://nodejs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-purple)](https://modelcontextprotocol.io/)
[![Tests](https://img.shields.io/badge/Tests-293%2B%20passing-success)](https://github.com/bridge-context/context-bridge)

**Supercharge your AI coding assistant with multi-repo awareness.**

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Claude / AI IDE                             │
│                          │                                      │
│                          ▼                                      │
│              ┌───────────────────────┐                          │
│              │    Context Bridge     │                          │
│              │      MCP Server       │                          │
│              └───────────────────────┘                          │
│                          │                                      │
│          ┌───────────────┼───────────────┐                      │
│          ▼               ▼               ▼                      │
│    ┌──────────┐    ┌──────────┐    ┌──────────┐                 │
│    │Sub-Agent │    │Sub-Agent │    │Sub-Agent │   (parallel)    │
│    │  Repo A  │    │  Repo B  │    │  Repo C  │                 │
│    └──────────┘    └──────────┘    └──────────┘                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Why Context Bridge?

| Challenge | Solution |
|-----------|----------|
| AI assistants can only see one repo at a time | Query and modify multiple repos in a single conversation |
| Microservices require context-switching | Parallel sub-agents gather info from all services at once |
| Repeated queries waste API tokens | Intelligent caching (1-hour TTL) reduces costs |
| No visibility into AI operations | TUI dashboard with real-time monitoring and metrics |
| Locked into one AI tool | Pluggable adapters for Claude, Codex, Aider, and more |

---

## Key Features

- **Multi-Repo Projects** - Group related repositories into unified projects
- **Natural Language Queries** - Ask questions across your entire codebase
- **Parallel Sub-Agents** - Concurrent queries for fast multi-repo analysis
- **Cross-Repo Implementation** - Make coordinated changes across services
- **TUI Dashboard** - Real-time monitoring with beautiful terminal UI
- **Query Caching** - SHA-256 hashed cache (~80% cost reduction)
- **Persistent Storage** - SQLite database for sessions and metrics
- **Pluggable Agents** - Support for Claude, Codex, Aider, and custom adapters

---

## Quick Example

```bash
# Initialize and set up a project
cb init
cb project create myapp "My microservices app"
cb repo add api ~/code/api
cb repo add web ~/code/web
cb install  # Add to Claude Code

# In Claude, ask:
# "How does authentication work across all repos?"
# "Add rate limiting to the API and update the frontend to handle it"
```

---

## Documentation

| Section | Description |
|---------|-------------|
| [Getting Started](./getting-started/) | Installation and quick start guide |
| [CLI Reference](./cli/) | Complete command-line interface documentation |
| [MCP Tools](./mcp-tools/) | MCP tools reference for AI assistants |
| [Guides](./guides/) | Multi-repo workflows, custom adapters, troubleshooting |
| [Architecture](./architecture/) | System internals and design |
| [API Reference](./api/) | TypeScript API for extensions and custom adapters |
| [Roadmap](./roadmap.md) | Development roadmap and future plans |
| [Changelog](./changelog.md) | Version history and upgrade guides |

---

## Requirements

- **Node.js** >= 18.0.0
- **AI CLI Tool** - Claude Code, Codex, Aider, or compatible MCP client

---

## License

MIT License - See [LICENSE](https://github.com/bridge-context/context-bridge/blob/main/LICENSE) for details.
