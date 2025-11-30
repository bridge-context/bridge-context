---
layout: default
title: MCP Tools
nav_order: 3
has_children: true
---

# MCP Tools

Complete reference for the Context Bridge MCP (Model Context Protocol) tools. These tools enable AI assistants to interact with multi-repository projects through intelligent sub-agents.

---

## Overview

Context Bridge exposes 9 MCP tools that AI assistants can use to work across multiple repositories. Tools are organized into three categories based on their function and capabilities.

---

## Tool Summary

| Tool | Category | Description |
|------|----------|-------------|
| `list_projects` | Context | List all available project contexts |
| `switch_project` | Context | Switch to a different project context |
| `get_project_info` | Context | Get details about the current project |
| `list_repos` | Context | List repositories in the current project |
| `query_repo` | Query | Query a specific repository with natural language |
| `query_all_repos` | Query | Query all repositories in parallel |
| `get_repo_structure` | Query | Get high-level overview of a repository |
| `implement_in_repo` | Implementation | Implement a task in a repository via sub-agent |
| `run_in_repo` | Implementation | Run a shell command in a repository |

---

## Tool Categories

### Context Tools (4)

Context tools manage project state and provide information about available repositories. These are lightweight, synchronous operations that don't spawn sub-agents.

- **[list_projects](./context-tools#list_projects)** - Enumerate all configured projects
- **[switch_project](./context-tools#switch_project)** - Change the active project context
- **[get_project_info](./context-tools#get_project_info)** - Retrieve project details and statistics
- **[list_repos](./context-tools#list_repos)** - List repositories within a project

### Query Tools (3)

Query tools gather information from repositories by spawning read-only sub-agents. Results are cached to improve performance for repeated queries.

- **[query_repo](./query-tools#query_repo)** - Natural language query against a single repository
- **[query_all_repos](./query-tools#query_all_repos)** - Parallel queries across all project repositories
- **[get_repo_structure](./query-tools#get_repo_structure)** - High-level repository structure and overview

### Implementation Tools (2)

Implementation tools make changes to repositories by spawning sub-agents with write capabilities. These are the most powerful tools and should be used carefully.

- **[implement_in_repo](./implementation-tools#implement_in_repo)** - Implement tasks via sub-agent with write access
- **[run_in_repo](./implementation-tools#run_in_repo)** - Execute shell commands in repository context

---

## How Sub-Agents Work

Context Bridge uses sub-agents to perform repository operations. When you invoke a query or implementation tool, the server:

1. **Spawns a sub-process** - Launches the configured AI CLI tool (Claude, Codex, Aider)
2. **Sets the working directory** - Changes to the target repository root
3. **Injects the task** - Passes your query or implementation request
4. **Restricts tool access** - Limits available tools based on operation type
5. **Captures output** - Collects and returns the sub-agent's response
6. **Stores results** - Saves to SQLite for observability and caching

### Sub-Agent Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AI Assistant                          │
│                  (MCP Client)                           │
└─────────────────────┬───────────────────────────────────┘
                      │ MCP Protocol
                      ▼
┌─────────────────────────────────────────────────────────┐
│               Context Bridge Server                      │
│                   (MCP Server)                          │
├─────────────────────┬───────────────────────────────────┤
│                     │                                    │
│    ┌────────────────┼────────────────┐                  │
│    ▼                ▼                ▼                  │
│ ┌──────┐       ┌──────┐        ┌──────┐                │
│ │ Sub  │       │ Sub  │        │ Sub  │                │
│ │Agent │       │Agent │        │Agent │                │
│ │Repo A│       │Repo B│        │Repo C│                │
│ └──────┘       └──────┘        └──────┘                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Tool Restrictions

Sub-agents have different tool access depending on the operation type:

### Read-Only Operations (Queries)

Query operations spawn sub-agents with restricted, read-only access:

| Tool | Description |
|------|-------------|
| `Read` | Read file contents |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents |
| `Bash` | Execute shell commands (read-only) |

These restrictions prevent accidental modifications during information gathering.

### Write Operations (Implementation)

Implementation operations spawn sub-agents with full write access:

| Tool | Description |
|------|-------------|
| `Read` | Read file contents |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents |
| `Bash` | Execute shell commands |
| `Edit` | Modify existing files |
| `Write` | Create new files |
| `MultiEdit` | Multiple edits in single operation |

---

## Query Caching

Query tools implement intelligent caching to improve performance and reduce redundant sub-agent operations.

### How Caching Works

1. **Cache Key Generation** - Queries are normalized and hashed using SHA-256
2. **TTL Enforcement** - Cache entries expire after 1 hour (3600 seconds)
3. **Repository Scoping** - Cache keys include repository identifier
4. **Result Storage** - Cached results stored in SQLite database

### Cache Key Structure

```
SHA-256(normalize(query) + repo_id + project_id)
```

### Cache Behavior

| Scenario | Behavior |
|----------|----------|
| Identical query, same repo | Returns cached result |
| Similar query, different repo | Cache miss, new sub-agent |
| Cache expired (>1 hour) | Cache miss, new sub-agent |
| Implementation operation | Never cached (always fresh) |

### Clearing the Cache

```bash
# Clear all cached query results
context-bridge storage clear --cache

# Clear cache for specific project
context-bridge storage clear --cache --project my-project
```

---

## Error Handling

MCP tools return structured errors that AI assistants can understand and recover from.

### Error Categories

| Category | Description | Recovery |
|----------|-------------|----------|
| `PROJECT_NOT_FOUND` | Specified project doesn't exist | List projects, use valid name |
| `REPO_NOT_FOUND` | Repository not in project | List repos, use valid name |
| `AGENT_TIMEOUT` | Sub-agent exceeded time limit | Simplify task, increase timeout |
| `AGENT_ERROR` | Sub-agent encountered error | Review error message, retry |
| `PATH_INVALID` | Repository path doesn't exist | Validate repos, fix paths |
| `PERMISSION_DENIED` | Insufficient access rights | Check file permissions |

### Error Response Format

```json
{
  "error": {
    "code": "REPO_NOT_FOUND",
    "message": "Repository 'backend' not found in project 'ecommerce'",
    "details": {
      "project": "ecommerce",
      "requested_repo": "backend",
      "available_repos": ["api", "frontend", "shared"]
    }
  }
}
```

### Timeout Configuration

Sub-agents have a default timeout of 5 minutes (300,000ms). This can be configured:

```bash
# Set via environment variable
export CONTEXT_BRIDGE_AGENT_TIMEOUT=600000

# Set via configuration
context-bridge config set agent.timeout 600000
```

---

## SQLite Storage

All tool operations are stored in SQLite for observability and debugging.

### Stored Data

| Data | Purpose |
|------|---------|
| Query results | Cache and replay |
| Sub-agent logs | Debugging and auditing |
| Execution times | Performance monitoring |
| Error details | Issue diagnosis |

### Viewing Stored Data

```bash
# View storage statistics
context-bridge storage stats

# View with project filter
context-bridge storage stats --project my-project

# Export for analysis
context-bridge storage export ./cb-data.json
```

---

## In This Section

- [Context Tools](./context-tools) - Project and repository management tools
- [Query Tools](./query-tools) - Read-only repository query tools
- [Implementation Tools](./implementation-tools) - Write-capable implementation tools

---

## See Also

- [CLI Reference](../cli) - Command-line interface documentation
- [Configuration](../getting-started/configuration) - Server and agent configuration
- [Agent Commands](../cli/agent-commands) - Manage agent adapters
