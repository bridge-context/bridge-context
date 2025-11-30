---
layout: default
title: Installation
parent: Getting Started
nav_order: 1
---

# Installation

This guide covers installing Context Bridge from source and configuring it for use with your AI assistant.

---

## Prerequisites

### Node.js

Context Bridge requires **Node.js 18 or later**.

```bash
# Check your Node.js version
node --version
```

If you need to install or update Node.js, visit [nodejs.org](https://nodejs.org/) or use a version manager like [nvm](https://github.com/nvm-sh/nvm).

### AI CLI Tool

Context Bridge works with any MCP-compatible AI client. Supported options include:

- **[Claude Desktop](https://claude.ai/download)** - Anthropic's desktop application
- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** - CLI-based Claude assistant
- **[Cursor](https://cursor.sh/)** - AI-powered code editor
- **[Codex](https://github.com/openai/codex)** - OpenAI's coding assistant
- **[Aider](https://aider.chat/)** - AI pair programming in your terminal

---

## Install from Source

### 1. Clone the Repository

```bash
git clone https://github.com/bridge-context/context-bridge.git
cd context-bridge
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Build the Project

```bash
npm run build
```

This compiles the TypeScript source and prepares the CLI for use.

---

## Link Globally

To use `context-bridge` as a global command:

```bash
npm link
```

This creates a symlink in your global `node_modules`, making the `context-bridge` command available system-wide.

---

## Verify Installation

Run the doctor command to verify your installation:

```bash
context-bridge doctor
```

This checks:
- Node.js version compatibility
- Configuration file presence
- Storage database connectivity
- Agent adapter availability

A successful output looks like:

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

### Claude Desktop

The easiest way to install Context Bridge in Claude Desktop:

```bash
context-bridge install
```

This automatically adds the MCP server configuration to your Claude Desktop settings.

**Manual installation:**

1. Open Claude Desktop settings
2. Navigate to the MCP servers section
3. Add a new server with the following configuration:

```json
{
  "mcpServers": {
    "context-bridge": {
      "command": "node",
      "args": ["/path/to/context-bridge/dist/index.js"],
      "env": {}
    }
  }
}
```

Replace `/path/to/context-bridge` with your actual installation path.

### Cursor

For Cursor, add the MCP server in your workspace settings:

1. Open Cursor settings (`Cmd+,` or `Ctrl+,`)
2. Search for "MCP" or navigate to AI settings
3. Add the Context Bridge server:

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

---

## Next Steps

- [Quick Start](./quick-start) - Set up your first project in 5 minutes
- [Configuration](./configuration) - Learn about configuration options
