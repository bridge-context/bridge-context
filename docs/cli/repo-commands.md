---
layout: default
title: Repository Commands
parent: CLI Reference
nav_order: 2
---

# Repository Commands

Commands for managing repositories within the active project. Repositories are the codebases that AI sub-agents can query and modify.

---

## repo add

Add a repository to the current project.

### Syntax

```bash
context-bridge repo add <name> <path> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Unique name for the repository within the project |
| `path` | Yes | Absolute or relative path to the repository root |

### Options

| Option | Description |
|--------|-------------|
| `--description`, `-d` | Optional description for the repository |
| `--help` | Display help for this command |

### Examples

```bash
# Add a repository with absolute path
context-bridge repo add api /Users/dev/projects/ecommerce-api

# Add with relative path (resolved from current directory)
context-bridge repo add frontend ../ecommerce-web

# Add with description
context-bridge repo add shared ~/projects/shared-lib -d "Shared types and utilities"

# Using the short alias
cb repo add backend ./backend-service
```

### Output

```
✓ Added repository: api
  Path: /Users/dev/projects/ecommerce-api
  Project: ecommerce
```

### Path Resolution

- Relative paths are resolved from the current working directory
- Paths are stored as absolute paths in configuration
- Home directory (`~`) expansion is supported

### Validation

The command validates that:
- The path exists and is a directory
- The repository name is unique within the project
- The path is accessible (readable)

---

## repo list

List all repositories in the current project.

### Syntax

```bash
context-bridge repo list [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# List repositories in active project
context-bridge repo list

# JSON output for scripting
context-bridge repo list --json
```

### Output

```
Repositories in "ecommerce":
────────────────────────────
  api
    Path: /Users/dev/projects/ecommerce-api
    Description: REST API backend (Node.js/Express)
    Status: ✓ Valid

  frontend
    Path: /Users/dev/projects/ecommerce-web
    Description: React frontend application
    Status: ✓ Valid

  shared
    Path: /Users/dev/projects/ecommerce-shared
    Description: Shared types and utilities
    Status: ⚠ Path not found

Total: 3 repositories (2 valid, 1 invalid)
```

### JSON Output

```json
{
  "project": "ecommerce",
  "repos": [
    {
      "name": "api",
      "path": "/Users/dev/projects/ecommerce-api",
      "description": "REST API backend (Node.js/Express)",
      "valid": true
    },
    {
      "name": "frontend",
      "path": "/Users/dev/projects/ecommerce-web",
      "description": "React frontend application",
      "valid": true
    },
    {
      "name": "shared",
      "path": "/Users/dev/projects/ecommerce-shared",
      "description": "Shared types and utilities",
      "valid": false
    }
  ],
  "summary": {
    "total": 3,
    "valid": 2,
    "invalid": 1
  }
}
```

---

## repo remove

Remove a repository from the current project.

### Syntax

```bash
context-bridge repo remove <name> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Name of the repository to remove |

### Options

| Option | Description |
|--------|-------------|
| `--force`, `-f` | Skip confirmation prompt |
| `--help` | Display help for this command |

### Examples

```bash
# Remove with confirmation
context-bridge repo remove old-service

# Force remove without confirmation
context-bridge repo remove old-service --force

# Using the short alias
cb repo remove deprecated-lib -f
```

### Output

```
⚠ Remove repository "old-service" from project "ecommerce"?
  Path: /Users/dev/projects/old-service

  This only removes the reference. Files will NOT be deleted.

  Confirm? [y/N] y

✓ Removed repository: old-service
```

### Notes

- This command only removes the repository reference from the project configuration
- Actual files on disk are **not** deleted
- Cached data for the repository is preserved (use `storage clear` to clean up)

---

## repo validate

Validate all repository paths in the current project.

### Syntax

```bash
context-bridge repo validate [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--fix` | Attempt to fix common issues interactively |
| `--help` | Display help for this command |

### Examples

```bash
# Validate all repositories
context-bridge repo validate

# JSON output for CI/CD pipelines
context-bridge repo validate --json

# Interactive fix mode
context-bridge repo validate --fix
```

### Output

```
Validating repositories in "ecommerce":
───────────────────────────────────────

  api
    Path: /Users/dev/projects/ecommerce-api
    ✓ Path exists
    ✓ Directory is readable
    ✓ Contains .git directory

  frontend
    Path: /Users/dev/projects/ecommerce-web
    ✓ Path exists
    ✓ Directory is readable
    ✓ Contains .git directory

  shared
    Path: /Users/dev/projects/ecommerce-shared
    ✗ Path does not exist

Summary:
  Valid: 2
  Invalid: 1

⚠ Some repositories have issues. Use --fix to resolve interactively.
```

### JSON Output

```json
{
  "project": "ecommerce",
  "results": [
    {
      "name": "api",
      "path": "/Users/dev/projects/ecommerce-api",
      "valid": true,
      "checks": {
        "exists": true,
        "readable": true,
        "isGitRepo": true
      }
    },
    {
      "name": "shared",
      "path": "/Users/dev/projects/ecommerce-shared",
      "valid": false,
      "checks": {
        "exists": false,
        "readable": false,
        "isGitRepo": false
      },
      "error": "Path does not exist"
    }
  ],
  "summary": {
    "total": 3,
    "valid": 2,
    "invalid": 1
  },
  "success": false
}
```

### Fix Mode

When using `--fix`, the command interactively helps resolve issues:

```
Fixing repository issues:
─────────────────────────

  shared: Path does not exist
    Current path: /Users/dev/projects/ecommerce-shared

    Options:
    [1] Enter new path
    [2] Remove repository from project
    [3] Skip

    Choose option: 1
    New path: /Users/dev/libs/shared
    ✓ Updated path for "shared"
```

### Validation Checks

| Check | Description |
|-------|-------------|
| Path exists | The directory exists on the filesystem |
| Readable | The directory is accessible with read permissions |
| Git repository | Contains a `.git` directory (optional) |

---

## See Also

- [Project Commands](./project-commands.md) - Create and manage projects
- [Configuration](../getting-started/configuration.md) - Repository configuration options
