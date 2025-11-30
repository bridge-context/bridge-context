# Context Bridge

[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D18.0.0-brightgreen)](https://nodejs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-purple)](https://modelcontextprotocol.io/)
[![Tests](https://img.shields.io/badge/Tests-293%2B%20passing-success)](https://github.com/bridge-context/context-bridge)

**Supercharge your AI coding assistant with multi-repo awareness.**

Context Bridge is an MCP (Model Context Protocol) server that enables AI assistants like Claude to work across multiple repositories simultaneously. It spawns intelligent sub-agents to query, analyze, and implement changes across your entire codebase - perfect for microservices, monorepos, and distributed architectures.

```
┌─────────────────────────────────────────────────────────────────┐
│                         Claude / AI IDE                         │
│                              │                                  │
│                              ▼                                  │
│                     ┌─────────────────┐                         │
│                     │ Context Bridge  │                         │
│                     │   (MCP Server)  │                         │
│                     └────────┬────────┘                         │
│              ┌───────────────┼───────────────┐                  │
│              ▼               ▼               ▼                  │
│        ┌──────────┐   ┌──────────┐   ┌──────────┐               │
│        │ api-gw   │   │ user-svc │   │ order-svc│               │
│        │ sub-agent│   │ sub-agent│   │ sub-agent│               │
│        └──────────┘   └──────────┘   └──────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

## Why Context Bridge?

| Challenge | Solution |
|-----------|----------|
| AI assistants can only see one repo at a time | Query and modify multiple repos in a single conversation |
| Microservices require context-switching | Parallel sub-agents gather info from all services at once |
| Repeated queries waste API tokens | Intelligent caching (1-hour TTL) reduces costs |
| No visibility into AI operations | TUI dashboard with real-time monitoring and metrics |
| Locked into one AI tool | Pluggable adapters for Claude, Codex, Aider, and more |

## Key Features

- **Multi-Repo Projects** - Organize repositories into logical project groups
- **Natural Language Queries** - Ask questions across your entire codebase
- **Parallel Sub-Agents** - Query all repos simultaneously for faster results
- **Cross-Repo Implementation** - Implement features spanning multiple services
- **TUI Dashboard** - Real-time terminal UI for monitoring agent activity
- **Query Caching** - SHA-256 keyed cache reduces API costs by up to 80%
- **Persistent Storage** - SQLite-backed tracking of all operations
- **Pluggable Agents** - Built-in Claude adapter + npm-installable community adapters

## Quick Start

### 1. Install & Initialize

```bash
# Clone and build
git clone https://github.com/bridge-context/context-bridge.git
cd context-bridge && npm install && npm run build

# Initialize (creates ~/.context-bridge/config.yaml)
npx context-bridge init
```

> **Prerequisites:** Node.js >= 18.0.0 and [Claude CLI](https://docs.anthropic.com/en/docs/claude-cli) (or another supported AI CLI)

### 2. Set Up Your Project

```bash
# Create a project and add your repos
cb project create my-platform "My microservices platform"
cb repo add api-gateway /path/to/api-gateway
cb repo add user-service /path/to/user-service
cb repo add order-service /path/to/order-service
```

### 3. Install MCP Server

```bash
# Auto-install for Claude Desktop / Claude Code
cb install
```

<details>
<summary>Manual installation</summary>

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "context-bridge": {
      "command": "node",
      "args": ["/path/to/context-bridge/dist/server.js"]
    }
  }
}
```
</details>

### 4. Start Using

Now Claude can work across all your repos:

```
You: "What authentication methods are used across all my services?"
You: "Implement a new health check endpoint in the user-service"
You: "Run tests in the order-service"
```

## CLI Reference

All commands support the short alias `cb` (e.g., `cb project list`).

| Category | Command | Description |
|----------|---------|-------------|
| **Setup** | `init` | Initialize configuration |
| | `install` / `uninstall` | Add/remove MCP server from Claude Desktop |
| | `doctor` | Check system health |
| | `ui` / `dashboard` | Launch TUI dashboard |
| **Projects** | `project create <name>` | Create a new project |
| | `project list` | List all projects |
| | `project switch <name>` | Switch active project |
| **Repos** | `repo add <name> <path>` | Add repo to current project |
| | `repo list` | List repos in project |
| | `repo remove <name>` | Remove repo from project |
| **Agents** | `agent list` | List available adapters |
| | `agent install <name>` | Install npm adapter |
| | `agent doctor` | Validate agent setup |
| **Storage** | `storage info` | Show storage stats |
| | `storage stats` | Detailed metrics |
| | `storage clear --cache` | Clear query cache |
| | `storage export <file>` | Export data to JSON |
| **Config** | `config show` | Display configuration |
| | `config set <key> <value>` | Set config value |
| | `config edit` | Open in editor |

<details>
<summary>Full CLI documentation</summary>

See the [CLI Reference](./docs/cli/index.md) for complete command documentation.
</details>

## Storage & Analytics

Context Bridge automatically tracks all operations with zero configuration:

| What's Tracked | Details |
|----------------|---------|
| **Sessions** | Groups related operations by server startup |
| **MCP Calls** | All tool invocations with args, results, timing |
| **Sub-Agent Executions** | Full transcripts, token usage, cost estimates |
| **Query Cache** | SHA-256 keyed, 1-hour TTL, ~80% cost reduction |

**Storage location:** `~/.context-bridge/storage.db` (SQLite with WAL mode)

```bash
# View statistics
cb storage stats

# Export analytics
cb storage export metrics.json

# Clear cache to force fresh queries
cb storage clear --cache
```

## MCP Tools

Context Bridge exposes 9 tools via the Model Context Protocol:

| Tool | Description |
|------|-------------|
| `list_projects` | List all project contexts |
| `switch_project` | Switch active project |
| `get_project_info` | Get project details and repos |
| `list_repos` | List repos in current project |
| `query_repo` | Query a repo with natural language (spawns sub-agent) |
| `query_all_repos` | Query all repos in parallel |
| `get_repo_structure` | Get high-level repo overview |
| `implement_in_repo` | Implement changes via sub-agent |
| `run_in_repo` | Run commands (tests, builds, etc.) |

## Configuration

Config file: `~/.context-bridge/config.yaml`

```yaml
projects:
  my-platform:
    description: 'E-commerce platform'
    repos:
      - name: api-gateway
        path: /path/to/api-gateway
      - name: user-service
        path: /path/to/user-service

current_project: my-platform

agent:
  type: claude      # Built-in: claude | Community: codex, aider
  timeout: 300000   # 5 minutes default (in ms)
```

<details>
<summary>Custom adapters</summary>

```yaml
# Use a local custom adapter
agent:
  path: /path/to/custom-adapter.js
  timeout: 600000
```

See [Custom Adapters Guide](./docs/guides/custom-adapters.md) for building your own.
</details>

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    Context Bridge                         │
├──────────────────────────────────────────────────────────┤
│  MCP Server Layer     │  Exposes tools via MCP protocol  │
│  Agent Abstraction    │  Pluggable adapters (Claude,     │
│                       │  Codex, Aider, custom)           │
│  Storage Layer        │  SQLite + Drizzle ORM            │
│  CLI Interface        │  Commander.js based              │
│  TUI Dashboard        │  Ink (React for terminals)       │
└──────────────────────────────────────────────────────────┘
```

**Sub-agent Tool Restrictions:**
- **Read queries:** `Read`, `Glob`, `Grep`, `Bash`
- **Write operations:** + `Edit`, `Write`, `MultiEdit`

## Security

> **Only install adapters from trusted sources.** Custom adapters execute with full system access.

- Built-in Claude adapter is maintained by the project team
- Community npm adapters should be from verified publishers
- Use `cb agent doctor` to validate installations
- Review adapter source code before installing

## Use Cases

| Scenario | Example Prompts |
|----------|-----------------|
| **Microservices** | "How do the services communicate?" / "Add tracing headers to all calls" |
| **Monorepos** | "Which packages depend on shared-utils?" / "Update logging format everywhere" |
| **Cross-repo Refactoring** | "Rename UserDTO to UserResponse in all services" |
| **Code Auditing** | "Find deprecated API usages" / "Check auth code for vulnerabilities" |

## Development

```bash
npm run build      # Build project
npm run dev        # Watch mode
npm test           # Run tests
npm run lint       # Lint code

# Debug mode (logs to ~/.context-bridge/debug.log)
CONTEXT_BRIDGE_VERBOSE=1 cb serve
```

## Contributing

Contributions welcome! See [DEVELOPMENT.md](./DEVELOPMENT.md) for setup instructions.

For AI coding agents, see [AGENTS.md](./AGENTS.md) for codebase guidance.

## License

MIT - see [LICENSE](./LICENSE)

## Links

- [Documentation](./docs/index.md) - Full documentation
- [Roadmap](./ROADMAP.md) - Current progress and planned features
- [Model Context Protocol](https://modelcontextprotocol.io/) - MCP specification
- [Claude CLI](https://docs.anthropic.com/en/docs/claude-cli) - Default AI CLI tool
- [MCP SDK](https://github.com/modelcontextprotocol/typescript-sdk) - TypeScript SDK
