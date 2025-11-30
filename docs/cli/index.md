---
layout: default
title: CLI Reference
nav_order: 2
has_children: true
---

# CLI Reference

Command-line interface for Context Bridge. Use `context-bridge` or the short alias `cb`.

---

## Command Reference

| Category | Command | Description |
|----------|---------|-------------|
| **Setup** | `init` | Initialize configuration |
| | `onboard` | Interactive setup wizard |
| | `install` | Install MCP server in client |
| | `uninstall` | Remove MCP server from client |
| **Project** | `project create <name>` | Create a new project |
| | `project list` | List all projects |
| | `project switch <name>` | Switch active project |
| | `project info [name]` | Show project details |
| | `project delete <name>` | Delete a project |
| **Repository** | `repo add <name> <path>` | Add repository to project |
| | `repo list` | List repositories in project |
| | `repo remove <name>` | Remove repository |
| | `repo validate` | Validate repository paths |
| **Agent** | `agent list` | List available adapters |
| | `agent install <name>` | Install an adapter |
| | `agent uninstall <name>` | Uninstall an adapter |
| | `agent doctor` | Check agent health |
| **Storage** | `storage info` | Show storage configuration |
| | `storage stats` | Display statistics |
| | `storage clear --cache` | Clear cached data |
| | `storage export <file>` | Export storage to file |
| | `storage migrate` | Run database migrations |
| | `storage doctor` | Check storage health |
| **Utility** | `doctor` | System health check |
| | `serve` | Start MCP server |
| | `ui` / `dashboard` | Launch TUI dashboard |
| | `test-query` | Test repository queries |
| | `config show` | Display configuration |
| | `config set <key> <value>` | Set configuration value |
| | `config edit` | Open config in editor |

{: .note }
> Use `cb` as a shorthand for `context-bridge`. Both are identical.

---

## Global Options

| Option | Description |
|--------|-------------|
| `--help`, `-h` | Display help for command |
| `--json` | Output in JSON format |
| `--version`, `-v` | Display version |

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CONTEXT_BRIDGE_VERBOSE` | Enable debug logging | `0` |
| `CONTEXT_BRIDGE_CONFIG` | Config file path | `~/.context-bridge/config.yaml` |
| `CONTEXT_BRIDGE_STORAGE_PATH` | Database path | `~/.context-bridge/storage.db` |
| `CONTEXT_BRIDGE_AGENT_TYPE` | Agent adapter | `claude` |
| `CONTEXT_BRIDGE_AGENT_TIMEOUT` | Timeout in ms | `300000` |

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Invalid arguments |
| `3` | Configuration error |
| `4` | Storage error |

---

## Detailed Pages

- [Project Commands](./project-commands.md) - Create and manage projects
- [Repository Commands](./repo-commands.md) - Add and manage repositories
- [Agent Commands](./agent-commands.md) - Configure agent adapters
- [Storage Commands](./storage-commands.md) - Database and cache management
- [Utility Commands](./utility-commands.md) - Setup and diagnostics
