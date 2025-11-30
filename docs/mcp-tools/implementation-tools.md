---
layout: default
title: Implementation Tools
parent: MCP Tools
nav_order: 3
---

# Implementation Tools

Implementation tools make changes to repositories by spawning sub-agents with write capabilities. These are the most powerful tools and should be used with appropriate care.

---

## implement_in_repo

Implement a task in a repository via sub-agent.

### Description

Spawns a sub-agent with full write access to implement changes in the specified repository. The sub-agent can read, create, edit, and delete files to complete the requested task.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `repo` | string | Yes | Name of the repository to modify |
| `task` | string | Yes | Description of the implementation task |
| `context` | string | No | Additional context from other repos or planning |

### Example Request

```json
{
  "tool": "implement_in_repo",
  "arguments": {
    "repo": "api",
    "task": "Add a new endpoint POST /api/products that creates a product with name, price, and description fields. Include input validation and proper error handling."
  }
}
```

### Example Response

```json
{
  "success": true,
  "repo": "api",
  "task": "Add a new endpoint POST /api/products...",
  "result": {
    "summary": "Successfully implemented POST /api/products endpoint",
    "changes": [
      {
        "file": "src/routes/products.routes.ts",
        "action": "modified",
        "description": "Added POST /products route handler"
      },
      {
        "file": "src/controllers/products.controller.ts",
        "action": "modified",
        "description": "Added createProduct controller method"
      },
      {
        "file": "src/services/products.service.ts",
        "action": "modified",
        "description": "Added create method with database insertion"
      },
      {
        "file": "src/validators/product.validator.ts",
        "action": "created",
        "description": "New validation schema for product creation"
      }
    ],
    "notes": "Used Zod for input validation. Added proper 400/500 error responses. Consider adding authentication middleware for production use."
  },
  "executionTime": 45230,
  "subAgentId": "sa_impl_xyz789"
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether implementation completed |
| `repo` | string | Repository that was modified |
| `task` | string | Original task description |
| `result.summary` | string | Brief summary of what was done |
| `result.changes` | array | List of file changes |
| `result.changes[].file` | string | File path |
| `result.changes[].action` | string | created, modified, deleted |
| `result.changes[].description` | string | What changed |
| `result.notes` | string | Additional context or recommendations |
| `executionTime` | number | Time in milliseconds |
| `subAgentId` | string | Identifier for debugging |

### Sub-Agent Tools

Implementation sub-agents have full write access:

| Tool | Description |
|------|-------------|
| `Read` | Read file contents |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents |
| `Bash` | Execute shell commands |
| `Edit` | Modify existing files |
| `Write` | Create new files |
| `MultiEdit` | Multiple edits in single operation |

### Using Context Parameter

The `context` parameter provides additional information from other repositories or planning phases:

```json
{
  "tool": "implement_in_repo",
  "arguments": {
    "repo": "frontend",
    "task": "Add a ProductForm component that submits to the new products API",
    "context": "The API endpoint is POST /api/products with body: { name: string, price: number, description: string }. Returns 201 with created product or 400 for validation errors. See api repo for full schema."
  }
}
```

### Timeout Configuration

Implementation tasks have a **5-minute default timeout** (300,000ms). For complex implementations:

```bash
# Increase timeout via environment variable
export CONTEXT_BRIDGE_AGENT_TIMEOUT=600000  # 10 minutes

# Or via configuration
context-bridge config set agent.timeout 600000
```

### Errors

| Code | Message | Cause |
|------|---------|-------|
| `REPO_NOT_FOUND` | Repository 'xyz' not found | Repo not in current project |
| `AGENT_TIMEOUT` | Implementation timed out | Task too complex or timeout too short |
| `AGENT_ERROR` | Sub-agent encountered error | Internal failure |
| `PERMISSION_DENIED` | Cannot write to repository | File system permissions |

### Notes

- Implementation results are **never cached** (always fresh execution)
- Complex tasks may require multiple iterations
- Review changes before committing to version control
- Use `context` to share information across repositories

---

## run_in_repo

Run a shell command in a repository's context.

### Description

Executes a shell command with the working directory set to the repository root. Useful for running tests, builds, linters, or other development tools.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `repo` | string | Yes | Name of the repository |
| `command` | string | Yes | Shell command to execute |

### Example Request

```json
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "npm test"
  }
}
```

### Example Response

```json
{
  "success": true,
  "repo": "api",
  "command": "npm test",
  "result": {
    "exitCode": 0,
    "stdout": " PASS  tests/unit/user.service.test.ts\n PASS  tests/unit/auth.service.test.ts\n PASS  tests/integration/auth.test.ts\n\nTest Suites: 3 passed, 3 total\nTests:       24 passed, 24 total\nSnapshots:   0 total\nTime:        4.523 s",
    "stderr": "",
    "workingDirectory": "/Users/dev/projects/ecommerce-api"
  },
  "executionTime": 5234
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether command executed (not exit code) |
| `repo` | string | Repository where command ran |
| `command` | string | Command that was executed |
| `result.exitCode` | number | Process exit code (0 = success) |
| `result.stdout` | string | Standard output |
| `result.stderr` | string | Standard error output |
| `result.workingDirectory` | string | Directory command ran in |
| `executionTime` | number | Time in milliseconds |

### Common Use Cases

#### Running Tests

```json
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "npm test -- --coverage"
  }
}
```

#### Building the Project

```json
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "frontend",
    "command": "npm run build"
  }
}
```

#### Linting

```json
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "npm run lint"
  }
}
```

#### Checking Git Status

```json
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "git status --short"
  }
}
```

#### Installing Dependencies

```json
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "shared",
    "command": "npm install"
  }
}
```

### Failed Command Example

```json
{
  "success": true,
  "repo": "api",
  "command": "npm test",
  "result": {
    "exitCode": 1,
    "stdout": " FAIL  tests/unit/user.service.test.ts\n  ● UserService › should validate email\n\n    expect(received).toBe(expected)\n\n    Expected: true\n    Received: false\n\nTest Suites: 1 failed, 2 passed, 3 total\nTests:       1 failed, 23 passed, 24 total",
    "stderr": "",
    "workingDirectory": "/Users/dev/projects/ecommerce-api"
  },
  "executionTime": 4891
}
```

**Note:** `success: true` indicates the command executed, not that it passed. Check `exitCode` for actual success.

### Timeout

Commands timeout after 5 minutes by default. Configure with:

```bash
export CONTEXT_BRIDGE_AGENT_TIMEOUT=600000  # 10 minutes
```

### Security Considerations

- Commands run with the same permissions as the Context Bridge server
- Be cautious with commands that modify state
- Avoid commands that require interactive input
- Don't execute untrusted commands

### Errors

| Code | Message | Cause |
|------|---------|-------|
| `REPO_NOT_FOUND` | Repository 'xyz' not found | Repo not in current project |
| `COMMAND_TIMEOUT` | Command timed out | Exceeded timeout limit |
| `COMMAND_FAILED` | Command execution failed | Unable to spawn process |
| `PATH_INVALID` | Repository path invalid | Directory doesn't exist |

---

## Workflow Examples

### Implement and Verify

A common pattern: implement a feature, then verify with tests.

```json
// Step 1: Implement the feature
{
  "tool": "implement_in_repo",
  "arguments": {
    "repo": "api",
    "task": "Add password reset endpoint at POST /api/auth/reset-password"
  }
}

// Step 2: Run existing tests to verify no breakage
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "npm test"
  }
}

// Step 3: Run specific tests for new feature
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "npm test -- --grep 'reset-password'"
  }
}
```

### Cross-Repository Implementation

When a feature spans multiple repositories:

```json
// Step 1: Query API repo to understand existing patterns
{
  "tool": "query_repo",
  "arguments": {
    "repo": "api",
    "query": "How are API endpoints structured? What patterns are used for request validation?"
  }
}

// Step 2: Implement API changes
{
  "tool": "implement_in_repo",
  "arguments": {
    "repo": "api",
    "task": "Add GET /api/users/:id/activity endpoint returning user activity history"
  }
}

// Step 3: Implement frontend with context from API
{
  "tool": "implement_in_repo",
  "arguments": {
    "repo": "frontend",
    "task": "Add UserActivity component that displays user activity timeline",
    "context": "The API endpoint is GET /api/users/:id/activity returning { activities: [{ type: string, timestamp: string, description: string }] }"
  }
}

// Step 4: Verify both repos
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "npm test"
  }
}

{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "frontend",
    "command": "npm test"
  }
}
```

### Feature Branch Workflow

```json
// Step 1: Create feature branch
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "git checkout -b feature/user-preferences"
  }
}

// Step 2: Implement feature
{
  "tool": "implement_in_repo",
  "arguments": {
    "repo": "api",
    "task": "Add user preferences API with endpoints for get/update preferences"
  }
}

// Step 3: Run tests
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "npm test"
  }
}

// Step 4: Check changes
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "git diff --stat"
  }
}

// Step 5: Stage and commit
{
  "tool": "run_in_repo",
  "arguments": {
    "repo": "api",
    "command": "git add -A && git commit -m 'feat: add user preferences API'"
  }
}
```

---

## Best Practices

### Task Description Guidelines

**Be specific and detailed:**
```
Good: "Add POST /api/products endpoint with fields: name (string, required),
      price (number, required), description (string, optional). Include
      Zod validation, proper error responses, and database insertion."

Bad:  "Add products endpoint"
```

**Include acceptance criteria:**
```
Good: "Implement user search with:
      - Full-text search on name and email
      - Pagination (page, limit params)
      - Results include id, name, email, createdAt
      - Return 400 for invalid pagination params"

Bad:  "Add user search"
```

### Using Context Effectively

**Include relevant information from other repos:**
```json
{
  "context": "The shared types package exports UserPreferences interface with: theme: 'light' | 'dark', notifications: boolean, language: string. The frontend expects the API to match this schema exactly."
}
```

**Share API contracts:**
```json
{
  "context": "API contract from api repo: POST /orders returns { id, items, total, status, createdAt }. Status can be: pending, processing, shipped, delivered."
}
```

### Error Recovery

When implementation fails:

1. **Check the error message** - Often contains specific guidance
2. **Query first** - Understand existing patterns before retrying
3. **Simplify the task** - Break into smaller pieces
4. **Provide more context** - Include relevant code snippets
5. **Increase timeout** - For complex implementations

### Safety Practices

1. **Work on branches** - Not directly on main/master
2. **Review changes** - Check git diff before committing
3. **Run tests** - Verify implementation doesn't break existing code
4. **Commit incrementally** - Small, focused commits
5. **Use version control** - Easy rollback if needed

---

## Timeout Reference

| Operation Type | Default Timeout | Configuration |
|----------------|-----------------|---------------|
| Implementation | 5 minutes | `CONTEXT_BRIDGE_AGENT_TIMEOUT` |
| Shell command | 5 minutes | `CONTEXT_BRIDGE_AGENT_TIMEOUT` |
| Query | 5 minutes | `CONTEXT_BRIDGE_AGENT_TIMEOUT` |

### Adjusting Timeouts

```bash
# Environment variable (in milliseconds)
export CONTEXT_BRIDGE_AGENT_TIMEOUT=600000  # 10 minutes

# Configuration file
context-bridge config set agent.timeout 600000

# Per-session (restart server after)
CONTEXT_BRIDGE_AGENT_TIMEOUT=900000 context-bridge serve
```

---

## See Also

- [Context Tools](./context-tools.md) - Project and repository management
- [Query Tools](./query-tools.md) - Read-only repository queries
- [Agent Commands](../cli/agent-commands.md) - Configure agent adapters
- [Configuration](../getting-started/configuration.md) - Timeout settings
