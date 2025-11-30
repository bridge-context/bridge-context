---
layout: default
title: Configuration
parent: Getting Started
nav_order: 3
---

# Configuration Reference

Context Bridge uses two configuration files to manage projects and application settings.

---

## Configuration Files

| File | Purpose |
|------|---------|
| `~/.context-bridge/config.yaml` | Projects and repository definitions |
| `~/.context-bridge/app-config.yaml` | Agent adapters and storage settings |

---

## Project Configuration

**Location:** `~/.context-bridge/config.yaml`

This file defines your projects and their associated repositories.

### Structure

```yaml
projects:
  my-project:
    description: "Project description"
    repos:
      repo-name:
        path: /absolute/path/to/repo
        description: "Optional repo description"
```

### Example

```yaml
projects:
  ecommerce:
    description: "E-commerce platform with microservices"
    repos:
      api:
        path: /Users/dev/projects/ecommerce-api
        description: "REST API backend (Node.js/Express)"
      frontend:
        path: /Users/dev/projects/ecommerce-web
        description: "React frontend application"
      shared:
        path: /Users/dev/projects/ecommerce-shared
        description: "Shared types and utilities"

  mobile-app:
    description: "iOS and Android mobile application"
    repos:
      ios:
        path: /Users/dev/projects/mobile-ios
      android:
        path: /Users/dev/projects/mobile-android
      backend:
        path: /Users/dev/projects/mobile-backend
```

### Managing Projects via CLI

```bash
# Create a new project
context-bridge project create <name> [description]

# List all projects
context-bridge project list

# Add a repository to the current project
context-bridge repo add <name> <path>

# Remove a repository
context-bridge repo remove <name>
```

---

## Application Configuration

**Location:** `~/.context-bridge/app-config.yaml`

This file configures the agent adapter and storage backend.

### Structure

```yaml
agent:
  type: claude
  timeout: 300000
  options:
    # Agent-specific options

storage:
  adapter: sqlite
  options:
    # Storage-specific options
```

### Agent Configuration

The agent adapter determines which AI CLI tool Context Bridge uses to spawn sub-agents.

#### Claude Adapter (Default)

```yaml
agent:
  type: claude
  timeout: 300000  # 5 minutes in milliseconds
  options:
    model: claude-sonnet-4-20250514
    maxTokens: 8192
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `type` | string | `claude` | Agent adapter type |
| `timeout` | number | `300000` | Maximum execution time (ms) |
| `model` | string | `claude-sonnet-4-20250514` | Model to use for sub-agents |
| `maxTokens` | number | `8192` | Maximum response tokens |

#### Codex Adapter

```yaml
agent:
  type: codex
  timeout: 300000
  options:
    model: gpt-4
```

#### Aider Adapter

```yaml
agent:
  type: aider
  timeout: 300000
  options:
    model: gpt-4
    editFormat: diff
```

### Storage Configuration

Storage manages session tracking and caching.

#### SQLite Adapter (Default)

```yaml
storage:
  adapter: sqlite
  options:
    path: ~/.context-bridge/storage.db
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `adapter` | string | `sqlite` | Storage adapter type |
| `path` | string | `~/.context-bridge/storage.db` | Database file location |

The SQLite database stores:
- Session history and context
- Query result caching
- Repository metadata

---

## Environment Variable Overrides

Configuration values can be overridden using environment variables:

| Variable | Overrides | Example |
|----------|-----------|---------|
| `CONTEXT_BRIDGE_CONFIG` | Config file path | `/custom/path/config.yaml` |
| `CONTEXT_BRIDGE_AGENT_TYPE` | `agent.type` | `claude` |
| `CONTEXT_BRIDGE_AGENT_TIMEOUT` | `agent.timeout` | `600000` |
| `CONTEXT_BRIDGE_STORAGE_ADAPTER` | `storage.adapter` | `sqlite` |
| `CONTEXT_BRIDGE_STORAGE_PATH` | `storage.options.path` | `/custom/storage.db` |

### Example Usage

```bash
# Use a custom configuration file
CONTEXT_BRIDGE_CONFIG=/path/to/config.yaml context-bridge serve

# Override agent timeout for long-running operations
CONTEXT_BRIDGE_AGENT_TIMEOUT=600000 context-bridge serve

# Use a different storage location
CONTEXT_BRIDGE_STORAGE_PATH=/tmp/context-bridge.db context-bridge serve
```

### Shell Configuration

Add to your `.bashrc`, `.zshrc`, or shell profile:

```bash
# Context Bridge configuration
export CONTEXT_BRIDGE_AGENT_TYPE=claude
export CONTEXT_BRIDGE_AGENT_TIMEOUT=300000
```

---

## Complete Example

### config.yaml

```yaml
projects:
  saas-platform:
    description: "Multi-tenant SaaS application"
    repos:
      api:
        path: /Users/dev/saas/api
        description: "GraphQL API server"
      web:
        path: /Users/dev/saas/web
        description: "Next.js frontend"
      workers:
        path: /Users/dev/saas/workers
        description: "Background job processors"
      infra:
        path: /Users/dev/saas/infrastructure
        description: "Terraform and Kubernetes configs"
```

### app-config.yaml

```yaml
agent:
  type: claude
  timeout: 300000
  options:
    model: claude-sonnet-4-20250514
    maxTokens: 8192

storage:
  adapter: sqlite
  options:
    path: ~/.context-bridge/storage.db
```

---

## Troubleshooting

### Configuration Not Loading

1. Verify file permissions:
   ```bash
   ls -la ~/.context-bridge/
   ```

2. Check YAML syntax:
   ```bash
   context-bridge doctor
   ```

3. Validate with environment override:
   ```bash
   CONTEXT_BRIDGE_CONFIG=~/.context-bridge/config.yaml context-bridge project list
   ```

### Agent Timeout Issues

For large repositories or complex queries, increase the timeout:

```yaml
agent:
  timeout: 600000  # 10 minutes
```

Or via environment variable:

```bash
export CONTEXT_BRIDGE_AGENT_TIMEOUT=600000
```

---

## Next Steps

- [Usage Guide](../usage) - Learn advanced workflows
- [API Reference](../api) - MCP tools documentation
