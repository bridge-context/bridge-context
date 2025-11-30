---
layout: default
title: Utility Commands
parent: CLI Reference
nav_order: 5
---

# Utility Commands

General utility commands for setup, configuration, diagnostics, and running the Context Bridge server.

---

## init

Initialize Context Bridge configuration.

### Syntax

```bash
context-bridge init [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--force`, `-f` | Overwrite existing configuration |
| `--help` | Display help for this command |

### Examples

```bash
# Initialize configuration
context-bridge init

# Force reinitialize (overwrites existing config)
context-bridge init --force
```

### Output

```
Initializing Context Bridge...
══════════════════════════════

  Creating configuration directory...
    ✓ Created ~/.context-bridge/

  Creating configuration files...
    ✓ Created config.yaml
    ✓ Created app-config.yaml

  Initializing storage...
    ✓ Created storage.db
    ✓ Applied schema migrations

✓ Initialization complete!

Next steps:
  1. Create a project:     context-bridge project create <name>
  2. Add repositories:     context-bridge repo add <name> <path>
  3. Install MCP server:   context-bridge install
```

### Created Files

| File | Purpose |
|------|---------|
| `~/.context-bridge/config.yaml` | Projects and repositories |
| `~/.context-bridge/app-config.yaml` | Agent and storage settings |
| `~/.context-bridge/storage.db` | SQLite database |

---

## onboard

Interactive setup wizard for new users.

### Syntax

```bash
context-bridge onboard [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--help` | Display help for this command |

### Examples

```bash
# Start interactive onboarding
context-bridge onboard
```

### Output

```
Welcome to Context Bridge!
══════════════════════════

This wizard will help you set up Context Bridge for your development workflow.

Step 1: Configuration
─────────────────────
  ✓ Configuration files created

Step 2: Create Your First Project
─────────────────────────────────
  ? Project name: ecommerce
  ? Description: My e-commerce platform
  ✓ Project created

Step 3: Add Repositories
────────────────────────
  ? Add a repository? (Y/n) Y
  ? Repository name: api
  ? Repository path: ~/projects/ecommerce-api
  ✓ Repository added

  ? Add another repository? (Y/n) Y
  ? Repository name: frontend
  ? Repository path: ~/projects/ecommerce-web
  ✓ Repository added

  ? Add another repository? (Y/n) n

Step 4: Agent Configuration
───────────────────────────
  ? Select agent adapter:
    > claude (recommended)
      codex
      aider
  ✓ Agent configured: claude

Step 5: Install MCP Server
──────────────────────────
  ? Install in Claude Desktop? (Y/n) Y
  ✓ MCP server installed

Setup Complete!
───────────────
  Project: ecommerce (2 repositories)
  Agent: claude
  MCP: Installed in Claude Desktop

  Restart Claude Desktop to start using Context Bridge.
```

---

## doctor

Run a comprehensive system health check.

### Syntax

```bash
context-bridge doctor [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Run health check
context-bridge doctor

# JSON output for automation
context-bridge doctor --json
```

### Output

```
Context Bridge Health Check
═══════════════════════════

  System Requirements
    ✓ Node.js version: v20.10.0 (required: >= 18)
    ✓ npm available: 10.2.3

  Configuration
    ✓ Config directory: ~/.context-bridge/
    ✓ config.yaml: Valid
    ✓ app-config.yaml: Valid

  Storage
    ✓ SQLite database: Connected
    ✓ Schema version: 3 (current)
    ✓ Database size: 2.4 MB

  Agent
    ✓ Agent type: claude
    ✓ CLI installed: claude-code v1.0.24
    ✓ API connectivity: OK

  Projects
    ✓ Projects configured: 3
    ✓ Repositories: 8 total, 8 valid

Summary:
  All systems operational ✓
```

### With Issues

```
Context Bridge Health Check
═══════════════════════════

  System Requirements
    ✓ Node.js version: v20.10.0

  Configuration
    ✓ config.yaml: Valid
    ✗ app-config.yaml: Parse error at line 12
      Error: Invalid YAML syntax

  Storage
    ✓ SQLite database: Connected

  Agent
    ⚠ CLI installed: claude-code not found
      Suggestion: Run "npm install -g @anthropic-ai/claude-code"

  Projects
    ⚠ 1 repository has invalid path
      - ecommerce/shared: Path does not exist

Summary:
  1 error, 2 warnings

  Fix the errors above and run "context-bridge doctor" again.
```

---

## install

Install the MCP server in your AI client.

### Syntax

```bash
context-bridge install [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--client <name>` | Target client: `claude-desktop`, `cursor`, `auto` (default: `auto`) |
| `--help` | Display help for this command |

### Examples

```bash
# Auto-detect and install
context-bridge install

# Install for specific client
context-bridge install --client claude-desktop
context-bridge install --client cursor
```

### Output

```
Installing Context Bridge MCP Server
════════════════════════════════════

  Detecting AI clients...
    ✓ Found: Claude Desktop

  Installing in Claude Desktop...
    ✓ Configuration updated
    ✓ MCP server registered

✓ Installation complete!

  Please restart Claude Desktop to load the MCP server.

  Verify installation:
    1. Open Claude Desktop
    2. Check for "context-bridge" in MCP servers
    3. Try: "What projects do I have in Context Bridge?"
```

---

## uninstall

Remove the MCP server from your AI client.

### Syntax

```bash
context-bridge uninstall [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--client <name>` | Target client: `claude-desktop`, `cursor`, `auto` (default: `auto`) |
| `--help` | Display help for this command |

### Examples

```bash
# Auto-detect and uninstall
context-bridge uninstall

# Uninstall from specific client
context-bridge uninstall --client claude-desktop
```

### Output

```
Uninstalling Context Bridge MCP Server
══════════════════════════════════════

  ✓ Removed from Claude Desktop configuration

  Please restart Claude Desktop to complete uninstallation.

  Note: Your configuration and data are preserved in ~/.context-bridge/
```

---

## serve

Start the MCP server.

### Syntax

```bash
context-bridge serve [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--port <port>` | Server port (for HTTP mode) |
| `--stdio` | Use stdio transport (default for MCP) |
| `--help` | Display help for this command |

### Examples

```bash
# Start server (stdio mode for MCP clients)
context-bridge serve

# Start with HTTP transport
context-bridge serve --port 3000
```

### Output (stdio mode)

```
Context Bridge MCP Server
═════════════════════════
  Version: 0.4.0
  Transport: stdio
  Project: ecommerce (3 repos)

  Waiting for MCP client connection...
```

### Notes

- The server runs in stdio mode by default for MCP client compatibility
- Use `--port` for HTTP mode when testing or debugging
- The server loads the active project on startup

---

## ui / dashboard

Launch the TUI (Terminal User Interface) dashboard.

### Syntax

```bash
context-bridge ui [options]
context-bridge dashboard [options]
```

Both commands are aliases and function identically.

### Options

| Option | Description |
|--------|-------------|
| `--help` | Display help for this command |

### Examples

```bash
# Launch dashboard
context-bridge ui

# Alternative command
context-bridge dashboard
```

### Features

The TUI dashboard provides:

- **Real-time monitoring** of agent activity
- **Session history** and query logs
- **Project overview** with repository status
- **Performance metrics** and statistics
- **Interactive navigation** with keyboard shortcuts

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `q` | Quit dashboard |
| `Tab` | Switch panels |
| `↑/↓` | Navigate lists |
| `Enter` | View details |
| `r` | Refresh data |
| `?` | Show help |

---

## test-query

Test repository queries without an MCP client.

### Syntax

```bash
context-bridge test-query [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--repo <name>` | Target repository |
| `--query <text>` | Query to execute |
| `--project <name>` | Project context (default: active project) |
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Interactive query mode
context-bridge test-query

# Query specific repository
context-bridge test-query --repo api --query "How does authentication work?"

# Query with specific project
context-bridge test-query --project ecommerce --repo frontend --query "List all components"
```

### Interactive Mode Output

```
Context Bridge Test Query
═════════════════════════

  Project: ecommerce
  Available repos: api, frontend, shared

  ? Select repository: api
  ? Enter query: How does authentication work?

  Executing query...

  ─────────────────────────────────────────────────────────────────

  The authentication system uses JWT tokens with the following flow:

  1. User submits credentials to POST /auth/login
  2. Server validates against the users table
  3. On success, generates a JWT with user ID and roles
  4. Token is returned and stored client-side

  Key files:
  - src/auth/controller.ts - Login/logout handlers
  - src/auth/middleware.ts - JWT verification middleware
  - src/auth/service.ts - Token generation and validation

  ─────────────────────────────────────────────────────────────────

  Query completed in 2.3s
```

---

## config show

Display current configuration.

### Syntax

```bash
context-bridge config show [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Show all configuration
context-bridge config show

# JSON output
context-bridge config show --json
```

### Output

```
Context Bridge Configuration
════════════════════════════

Agent:
  type: claude
  timeout: 300000
  model: claude-sonnet-4-20250514
  maxTokens: 8192

Storage:
  adapter: sqlite
  path: ~/.context-bridge/storage.db

Projects:
  Active: ecommerce
  Total: 3

Files:
  Config: ~/.context-bridge/config.yaml
  App Config: ~/.context-bridge/app-config.yaml
```

---

## config set

Set a configuration value.

### Syntax

```bash
context-bridge config set <key> <value> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `key` | Yes | Configuration key (dot notation) |
| `value` | Yes | Value to set |

### Options

| Option | Description |
|--------|-------------|
| `--help` | Display help for this command |

### Available Keys

| Key | Type | Description |
|-----|------|-------------|
| `agent.type` | string | Agent adapter type |
| `agent.timeout` | number | Agent timeout in milliseconds |
| `agent.model` | string | Model for sub-agents |
| `agent.maxTokens` | number | Maximum response tokens |
| `storage.adapter` | string | Storage adapter type |
| `storage.path` | string | Database file path |

### Examples

```bash
# Set agent type
context-bridge config set agent.type claude

# Set timeout
context-bridge config set agent.timeout 600000

# Set storage path
context-bridge config set storage.path ~/custom/storage.db
```

### Output

```
✓ Set agent.timeout = 600000

  Configuration updated in ~/.context-bridge/app-config.yaml
```

---

## config edit

Open configuration in your default editor.

### Syntax

```bash
context-bridge config edit [file] [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `file` | No | Config file to edit: `config`, `app-config` (default: `config`) |

### Options

| Option | Description |
|--------|-------------|
| `--help` | Display help for this command |

### Examples

```bash
# Edit project configuration
context-bridge config edit

# Edit app configuration
context-bridge config edit app-config
```

### Output

```
Opening ~/.context-bridge/config.yaml in editor...

  Editor: vim (from $EDITOR)

  After editing, validate with: context-bridge doctor
```

### Editor Selection

The editor is selected in this order:
1. `$VISUAL` environment variable
2. `$EDITOR` environment variable
3. System default (usually `vi` or `notepad`)

---

## config migrate

Migrate configuration to the latest format.

### Syntax

```bash
context-bridge config migrate [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--dry-run` | Show changes without applying |
| `--backup` | Create backup before migrating (default: true) |
| `--help` | Display help for this command |

### Examples

```bash
# Migrate configuration
context-bridge config migrate

# Preview changes
context-bridge config migrate --dry-run
```

### Output

```
Configuration Migration
═══════════════════════

  Current version: 1
  Target version: 2

  Changes:
    - config.yaml: Restructure repos format
    - app-config.yaml: Add new agent options

  Creating backup...
    ✓ Backup: ~/.context-bridge/config.yaml.bak
    ✓ Backup: ~/.context-bridge/app-config.yaml.bak

  Migrating...
    ✓ config.yaml migrated
    ✓ app-config.yaml migrated

✓ Migration complete

  Validate with: context-bridge doctor
```

---

## See Also

- [Project Commands](./project-commands) - Manage projects
- [Repository Commands](./repo-commands) - Manage repositories
- [Agent Commands](./agent-commands) - Configure agents
- [Storage Commands](./storage-commands) - Manage storage
- [Configuration](../getting-started/configuration) - Configuration reference
