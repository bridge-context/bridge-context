---
layout: default
title: Project Commands
parent: CLI Reference
nav_order: 1
---

# Project Commands

Commands for creating and managing projects. A project groups related repositories together for multi-repo AI workflows.

---

## project create

Create a new project.

### Syntax

```bash
context-bridge project create <name> [description]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Unique project name (alphanumeric, hyphens, underscores) |
| `description` | No | Human-readable project description |

### Options

| Option | Description |
|--------|-------------|
| `--help` | Display help for this command |

### Examples

```bash
# Create a project with description
context-bridge project create ecommerce "E-commerce platform with microservices"

# Create a project without description
context-bridge project create my-app

# Using the short alias
cb project create mobile-app "iOS and Android applications"
```

### Output

```
✓ Created project: ecommerce
  Description: E-commerce platform with microservices

Next steps:
  Add repositories with: context-bridge repo add <name> <path>
```

---

## project list

List all configured projects.

### Syntax

```bash
context-bridge project list [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# List all projects
context-bridge project list

# JSON output for scripting
context-bridge project list --json
```

### Output

```
Projects:
─────────
* ecommerce (active)
    Description: E-commerce platform with microservices
    Repositories: 3

  mobile-app
    Description: iOS and Android applications
    Repositories: 2

  internal-tools
    Repositories: 1
```

### JSON Output

```json
{
  "projects": [
    {
      "name": "ecommerce",
      "description": "E-commerce platform with microservices",
      "active": true,
      "repoCount": 3
    },
    {
      "name": "mobile-app",
      "description": "iOS and Android applications",
      "active": false,
      "repoCount": 2
    }
  ]
}
```

---

## project switch

Switch the active project context.

### Syntax

```bash
context-bridge project switch <name>
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Name of the project to switch to |

### Options

| Option | Description |
|--------|-------------|
| `--help` | Display help for this command |

### Examples

```bash
# Switch to a different project
context-bridge project switch mobile-app

# Verify the switch
context-bridge project info
```

### Output

```
✓ Switched to project: mobile-app
  Repositories: 2
```

### Notes

- The active project determines which repositories are used by default
- Repository commands operate on the active project
- The MCP server uses the active project for queries

---

## project info

Display detailed information about a project.

### Syntax

```bash
context-bridge project info [name] [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | No | Project name (defaults to active project) |

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Show info for active project
context-bridge project info

# Show info for specific project
context-bridge project info ecommerce

# JSON output
context-bridge project info ecommerce --json
```

### Output

```
Project: ecommerce
──────────────────
Description: E-commerce platform with microservices
Status: Active

Repositories:
  api
    Path: /Users/dev/projects/ecommerce-api
    Status: ✓ Valid

  frontend
    Path: /Users/dev/projects/ecommerce-web
    Status: ✓ Valid

  shared
    Path: /Users/dev/projects/ecommerce-shared
    Status: ✓ Valid

Statistics:
  Total repositories: 3
  Valid paths: 3
  Last accessed: 2024-01-15 10:30:00
```

### JSON Output

```json
{
  "name": "ecommerce",
  "description": "E-commerce platform with microservices",
  "active": true,
  "repos": [
    {
      "name": "api",
      "path": "/Users/dev/projects/ecommerce-api",
      "valid": true
    },
    {
      "name": "frontend",
      "path": "/Users/dev/projects/ecommerce-web",
      "valid": true
    }
  ],
  "stats": {
    "repoCount": 3,
    "validPaths": 3,
    "lastAccessed": "2024-01-15T10:30:00Z"
  }
}
```

---

## project delete

Delete a project and its configuration.

### Syntax

```bash
context-bridge project delete <name> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Name of the project to delete |

### Options

| Option | Description |
|--------|-------------|
| `--force`, `-f` | Skip confirmation prompt |
| `--help` | Display help for this command |

### Examples

```bash
# Delete with confirmation prompt
context-bridge project delete old-project

# Force delete without confirmation
context-bridge project delete old-project --force
```

### Output

```
⚠ Delete project "old-project"?
  This will remove the project configuration.
  Repository files will NOT be deleted.

  Confirm deletion? [y/N] y

✓ Deleted project: old-project
```

### Notes

- This command only removes the project from Context Bridge configuration
- Actual repository files on disk are **not** deleted
- Cached data for the project may be removed (use `storage clear --project` to ensure cleanup)
- Cannot delete the active project unless it's the only project

---

## See Also

- [Repository Commands](./repo-commands) - Add and manage repositories within projects
- [Configuration](../getting-started/configuration) - Project configuration file format
