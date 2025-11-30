---
layout: default
title: Agent Commands
parent: CLI Reference
nav_order: 3
---

# Agent Commands

Commands for managing agent adapters. Agents are the AI CLI tools that Context Bridge uses to spawn sub-agents for repository queries and implementations.

---

## agent list

List all available agent adapters.

### Syntax

```bash
context-bridge agent list [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# List all agents
context-bridge agent list

# JSON output
context-bridge agent list --json
```

### Output

```
Available Agent Adapters:
─────────────────────────

  claude (active)
    Status: ✓ Installed
    Version: 1.0.24
    Description: Anthropic Claude Code CLI adapter

  codex
    Status: ✓ Installed
    Version: 0.8.2
    Description: OpenAI Codex CLI adapter

  aider
    Status: ✗ Not installed
    Description: Aider AI pair programming adapter

Current configuration:
  Agent: claude
  Timeout: 300000ms
```

### JSON Output

```json
{
  "agents": [
    {
      "name": "claude",
      "installed": true,
      "active": true,
      "version": "1.0.24",
      "description": "Anthropic Claude Code CLI adapter"
    },
    {
      "name": "codex",
      "installed": true,
      "active": false,
      "version": "0.8.2",
      "description": "OpenAI Codex CLI adapter"
    },
    {
      "name": "aider",
      "installed": false,
      "active": false,
      "description": "Aider AI pair programming adapter"
    }
  ],
  "current": {
    "agent": "claude",
    "timeout": 300000
  }
}
```

---

## agent install

Install an agent adapter.

### Syntax

```bash
context-bridge agent install <name> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Name of the agent adapter to install |

### Options

| Option | Description |
|--------|-------------|
| `--help` | Display help for this command |

### Available Agents

| Name | Description |
|------|-------------|
| `claude` | Anthropic Claude Code CLI |
| `codex` | OpenAI Codex CLI |
| `aider` | Aider AI pair programming |

### Examples

```bash
# Install the Claude adapter
context-bridge agent install claude

# Install Aider adapter
context-bridge agent install aider
```

### Output

```
Installing agent adapter: claude
────────────────────────────────

  Checking prerequisites...
    ✓ Node.js >= 18
    ✓ npm available

  Installing claude-code CLI...
    ✓ Package installed globally

  Configuring adapter...
    ✓ Configuration updated

✓ Agent "claude" installed successfully

To use this agent:
  context-bridge config set agent.type claude
```

### Notes

- Some agents may require additional configuration (API keys, etc.)
- Use `agent doctor` after installation to verify functionality
- The install command may require administrator privileges for global npm installs

---

## agent uninstall

Uninstall an agent adapter.

### Syntax

```bash
context-bridge agent uninstall <name> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Name of the agent adapter to uninstall |

### Options

| Option | Description |
|--------|-------------|
| `--force`, `-f` | Skip confirmation prompt |
| `--help` | Display help for this command |

### Examples

```bash
# Uninstall an agent
context-bridge agent uninstall aider

# Force uninstall without confirmation
context-bridge agent uninstall aider --force
```

### Output

```
⚠ Uninstall agent adapter "aider"?

  This will:
  - Remove the CLI tool (if installed globally)
  - Clear adapter configuration

  Confirm? [y/N] y

Uninstalling agent adapter: aider
─────────────────────────────────
  ✓ Removed global package
  ✓ Cleared configuration

✓ Agent "aider" uninstalled
```

### Notes

- Cannot uninstall the currently active agent
- Switch to a different agent first with `config set agent.type <name>`
- Global npm packages are removed; local configurations are preserved

---

## agent doctor

Check agent health and diagnose issues.

### Syntax

```bash
context-bridge agent doctor [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Run agent health check
context-bridge agent doctor

# JSON output for automation
context-bridge agent doctor --json
```

### Output

```
Agent Health Check
══════════════════

Current Agent: claude
─────────────────────

  CLI Installation
    ✓ claude-code command found
    ✓ Version: 1.0.24

  Configuration
    ✓ Agent type: claude
    ✓ Timeout: 300000ms
    ✓ Model: claude-sonnet-4-20250514

  Connectivity
    ✓ API authentication valid
    ✓ Test query successful

  Permissions
    ✓ Can spawn subprocesses
    ✓ Can access repositories

Summary:
  All checks passed ✓
```

### Failed Check Output

```
Agent Health Check
══════════════════

Current Agent: claude
─────────────────────

  CLI Installation
    ✓ claude-code command found
    ✓ Version: 1.0.24

  Configuration
    ✓ Agent type: claude
    ✓ Timeout: 300000ms
    ✓ Model: claude-sonnet-4-20250514

  Connectivity
    ✗ API authentication failed
      Error: Invalid API key

      Suggestion: Set ANTHROPIC_API_KEY environment variable
      or configure via: claude-code config set apiKey <key>

  Permissions
    ⚠ Skipped (connectivity check failed)

Summary:
  1 check failed, 1 skipped

  Run with suggested fixes and try again.
```

### JSON Output

```json
{
  "agent": "claude",
  "checks": [
    {
      "name": "CLI Installation",
      "status": "pass",
      "details": {
        "command": "claude-code",
        "version": "1.0.24"
      }
    },
    {
      "name": "Configuration",
      "status": "pass",
      "details": {
        "type": "claude",
        "timeout": 300000,
        "model": "claude-sonnet-4-20250514"
      }
    },
    {
      "name": "Connectivity",
      "status": "fail",
      "error": "Invalid API key",
      "suggestion": "Set ANTHROPIC_API_KEY environment variable"
    },
    {
      "name": "Permissions",
      "status": "skip",
      "reason": "Connectivity check failed"
    }
  ],
  "summary": {
    "passed": 2,
    "failed": 1,
    "skipped": 1,
    "success": false
  }
}
```

### Checks Performed

| Check | Description |
|-------|-------------|
| CLI Installation | Verifies the agent CLI tool is installed and accessible |
| Configuration | Validates agent settings in app-config.yaml |
| Connectivity | Tests API authentication and connectivity |
| Permissions | Ensures the agent can spawn processes and access repos |

---

## See Also

- [Configuration](../getting-started/configuration) - Agent configuration options
- [Utility Commands](./utility-commands) - The `doctor` command for overall health checks
