---
layout: default
title: Context Bridge
---

# Context Bridge

**MCP server for multi-repo AI development workflows**

Context Bridge enables AI assistants to work seamlessly across multiple repositories, spawning intelligent sub-agents to query, analyze, and implement changes across your entire codebase.

---

## Key Features

- **Multi-Repository Support** - Work across multiple repositories in a single AI session
- **Intelligent Sub-Agents** - Spawns specialized agents to query, analyze, and implement changes
- **Pluggable Agent Adapters** - Support for Claude, Codex, Aider, and more
- **SQLite Storage** - Persistent session tracking and intelligent caching
- **TUI Dashboard** - Real-time monitoring with a beautiful terminal interface
- **MCP Protocol** - Built on the Model Context Protocol for seamless AI integration

---

## Current Status

**Version:** v0.4.0
**Status:** Production Ready
**Tests:** 293+ tests passing

---

## Requirements

- **Node.js** >= 18
- **AI CLI Tool** - Claude Code, Codex, Aider, or compatible MCP client

---

## Documentation

- [Getting Started](./getting-started) - Installation and quick start guide
- [CLI Reference](./cli) - Complete command-line interface documentation
- [MCP Tools](./mcp-tools) - MCP tools reference for AI assistants
- [Guides](./guides) - Multi-repo workflows, custom adapters, and troubleshooting
- [Architecture](./architecture) - System internals and design
- [API Reference](./api) - TypeScript API for extensions and custom adapters
- [Roadmap](./roadmap) - Development roadmap and future plans
- [Changelog](./changelog) - Version history and upgrade guides

---

## Quick Start

```bash
# Install Context Bridge
npm install -g context-bridge

# Initialize in your project
context-bridge init

# Start the MCP server
context-bridge serve
```

---

## License

MIT License - See [LICENSE](https://github.com/bridge-context/context-bridge/blob/main/LICENSE) for details.
