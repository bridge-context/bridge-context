---
layout: default
title: CLI Reference
nav_order: 2
has_children: true
---

# CLI Reference

Complete reference for the Context Bridge command-line interface.

---

## Usage

Context Bridge provides two equivalent command entry points:

```bash
# Full command
context-bridge <command> [options]

# Short alias
cb <command> [options]
```

Both entry points are identical in functionality. Use whichever you prefer.

---

## Command Categories

| Category | Description |
|----------|-------------|
| [Project Commands](./project-commands) | Create, list, and manage projects |
| [Repository Commands](./repo-commands) | Add, remove, and validate repositories |
| [Agent Commands](./agent-commands) | Manage agent adapters |
| [Storage Commands](./storage-commands) | Database and cache management |
| [Utility Commands](./utility-commands) | Setup, configuration, and diagnostics |

---

## Quick Reference

### Project Management

```bash
project create <name> [description]   # Create a new project
project list                          # List all projects
project switch <name>                 # Switch active project
project info [name]                   # Show project details
project delete <name>                 # Delete a project
```

### Repository Management

```bash
repo add <name> <path>                # Add repository to current project
repo list                             # List repositories in current project
repo remove <name>                    # Remove repository from project
repo validate                         # Validate all repository paths
```

### Agent Management

```bash
agent list                            # List available agent adapters
agent install <name>                  # Install an agent adapter
agent uninstall <name>                # Uninstall an agent adapter
agent doctor                          # Check agent health
```

### Storage Management

```bash
storage info                          # Show storage configuration
storage stats [--project <name>]      # Display storage statistics
storage clear --cache                 # Clear cached data
storage clear --project <name>        # Clear project data
storage export <file>                 # Export storage to file
storage migrate                       # Run database migrations
storage doctor                        # Check storage health
```

### Utilities

```bash
init                                  # Initialize configuration
onboard                               # Interactive setup wizard
doctor                                # System health check
install                               # Install MCP server in client
uninstall                             # Remove MCP server from client
serve                                 # Start the MCP server
ui / dashboard                        # Launch TUI dashboard
test-query                            # Test repository queries
config show                           # Display configuration
config set <key> <value>              # Set configuration value
config edit                           # Open config in editor
config migrate                        # Migrate configuration format
```

---

## Global Options

These options are available for all commands:

| Option | Description |
|--------|-------------|
| `--help`, `-h` | Display help for the command |
| `--json` | Output in JSON format (machine-readable) |
| `--version`, `-v` | Display version information |

### Examples

```bash
# Get help for any command
context-bridge project --help
context-bridge repo add --help

# JSON output for scripting
context-bridge project list --json
context-bridge storage stats --json
```

---

## Environment Variables

Context Bridge behavior can be customized using environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `CONTEXT_BRIDGE_VERBOSE` | Enable debug logging when set to `1` | `0` |
| `CONTEXT_BRIDGE_STORAGE_ADAPTER` | Override storage adapter type | `sqlite` |
| `CONTEXT_BRIDGE_STORAGE_PATH` | Override database file path | `~/.context-bridge/storage.db` |
| `CONTEXT_BRIDGE_CONFIG` | Override configuration file path | `~/.context-bridge/config.yaml` |
| `CONTEXT_BRIDGE_AGENT_TYPE` | Override agent adapter type | `claude` |
| `CONTEXT_BRIDGE_AGENT_TIMEOUT` | Override agent timeout (ms) | `300000` |

### Example Usage

```bash
# Enable verbose debug logging
CONTEXT_BRIDGE_VERBOSE=1 context-bridge serve

# Use a custom storage location
CONTEXT_BRIDGE_STORAGE_PATH=/tmp/cb-test.db context-bridge serve

# Override multiple settings
CONTEXT_BRIDGE_VERBOSE=1 \
CONTEXT_BRIDGE_STORAGE_PATH=/custom/path.db \
context-bridge doctor
```

### Shell Configuration

Add to your `.bashrc`, `.zshrc`, or shell profile for persistent settings:

```bash
# Context Bridge environment configuration
export CONTEXT_BRIDGE_VERBOSE=0
export CONTEXT_BRIDGE_AGENT_TIMEOUT=300000
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Invalid arguments or options |
| `3` | Configuration error |
| `4` | Storage/database error |

---

## In This Section

- [Project Commands](./project-commands) - Create and manage projects
- [Repository Commands](./repo-commands) - Add and manage repositories
- [Agent Commands](./agent-commands) - Configure agent adapters
- [Storage Commands](./storage-commands) - Database and cache management
- [Utility Commands](./utility-commands) - Setup and diagnostics
