---
layout: default
title: Quick Start
parent: Getting Started
nav_order: 2
---

# Quick Start

Get up and running with Context Bridge in under 5 minutes.

---

## Step 1: Install & Initialize

```bash
# Clone and build
git clone https://github.com/bridge-context/context-bridge.git
cd context-bridge
npm install && npm run build

# Initialize configuration
npm run cb init
```

This creates the configuration directory:

```
~/.context-bridge/
├── config.yaml       # Projects and repositories
├── app-config.yaml   # Agent and storage settings
└── storage.db        # SQLite database
```

---

## Step 2: Set Up Your Project

```bash
# Create a project
npm run cb project create myapp "My full-stack application"

# Add your repositories
npm run cb repo add api ~/projects/myapp-api
npm run cb repo add web ~/projects/myapp-web
npm run cb repo add shared ~/projects/myapp-shared

# Verify setup
npm run cb project list
```

**Output:**

```
Projects:
─────────
myapp
  Description: My full-stack application
  Repositories:
    - api: ~/projects/myapp-api
    - web: ~/projects/myapp-web
    - shared: ~/projects/myapp-shared
```

---

## Step 3: Install MCP Server

```bash
npm run cb install
```

**Restart Claude Code** to load the MCP server.

<details>
<summary><strong>Manual Installation</strong></summary>

Add to your Claude Code config (`~/.claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "context-bridge": {
      "command": "node",
      "args": ["/path/to/context-bridge/dist/mcp-server.js"]
    }
  }
}
```

</details>

---

## Step 4: Start Using

Once installed, use natural language in Claude Code:

| Task | Example Prompt |
|------|----------------|
| **Query single repo** | "How does authentication work in the api repo?" |
| **Query all repos** | "Search all repos for how user data is validated" |
| **Cross-repo analysis** | "How does the frontend consume the API?" |
| **Implement changes** | "Add email validation to user registration in the api repo" |
| **Get structure** | "Show me the structure of the web repo" |

### Common Workflows

```
"Rename the User model to Account across all repos"
"Review recent changes in api for security issues"
"Generate API documentation for all endpoints"
```

---

## Next Steps

- [Configuration Reference](./configuration.md) - Customize agent behavior and storage
- [Multi-Repo Workflow Guide](../guides/multi-repo-workflow.md) - Advanced workflows
- [MCP Tools Reference](../mcp-tools/) - Complete tools documentation
