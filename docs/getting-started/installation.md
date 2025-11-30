---
layout: default
title: Installation
parent: Getting Started
nav_order: 1
---

# Installation

Install Context Bridge from npm or source.

---

## Prerequisites

| Requirement | Version | Notes |
|-------------|---------|-------|
| **Node.js** | >= 18.0.0 | Check with `node --version` |
| **MCP Client** | Any | Claude Desktop, Claude Code, Cursor, Codex, Aider |

---

## Installation Methods

| Method | Command | Best For |
|--------|---------|----------|
| **npm (global)** | `npm install -g context-bridge` | Most users |
| **From source** | Clone + build | Development/contribution |
| **Link globally** | `npm link` | Testing local changes |

<details>
<summary><strong>npm global install</strong></summary>

```bash
npm install -g context-bridge
```

This installs `context-bridge` and `cb` commands globally.

</details>

<details>
<summary><strong>Install from source</strong></summary>

```bash
git clone https://github.com/bridge-context/context-bridge.git
cd context-bridge
npm install
npm run build
```

</details>

<details>
<summary><strong>Link globally (development)</strong></summary>

After building from source:

```bash
npm link
```

Creates a symlink making `context-bridge` and `cb` available system-wide.

</details>

---

## Verify Installation

```bash
# Check version
cb --version

# Run health check
cb doctor
```

Successful `cb doctor` output:

```
Context Bridge Health Check
===========================
✓ Node.js version: v20.10.0
✓ Configuration: ~/.context-bridge/config.yaml
✓ Storage: SQLite connected
✓ Agent: Claude adapter ready

All systems operational!
```

---

## Install in MCP Client

<details>
<summary><strong>Claude Desktop (automatic)</strong></summary>

```bash
cb install
```

Automatically configures Claude Desktop's MCP settings.

</details>

<details>
<summary><strong>Claude Desktop (manual)</strong></summary>

Add to Claude Desktop settings → MCP servers:

```json
{
  "mcpServers": {
    "context-bridge": {
      "command": "context-bridge",
      "args": ["serve"]
    }
  }
}
```

</details>

<details>
<summary><strong>Cursor</strong></summary>

In Cursor settings (`Cmd+,` / `Ctrl+,`) → AI → MCP:

```json
{
  "mcp.servers": {
    "context-bridge": {
      "command": "context-bridge",
      "args": ["serve"]
    }
  }
}
```

</details>

---

## Next Steps

- [Quick Start](./quick-start.md) - Set up your first project in 5 minutes
- [Configuration](./configuration.md) - Configuration options
