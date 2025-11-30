---
layout: default
title: MCP Tools
nav_order: 3
has_children: true
---

# MCP Tools

Context Bridge exposes 9 MCP tools for AI assistants to work across multiple repositories.

---

## Tool Reference

| Tool | Description | Use Case |
|------|-------------|----------|
| `list_projects` | List all project contexts | Discover available projects |
| `switch_project` | Switch to a different project | Change working context |
| `get_project_info` | Get current project details | View repos and settings |
| `list_repos` | List repositories in project | See available repos |
| `query_repo` | Query a repo with natural language | Get info from one repo |
| `query_all_repos` | Query all repos in parallel | Cross-repo analysis |
| `get_repo_structure` | Get high-level repo overview | Understand codebase structure |
| `implement_in_repo` | Implement a task via sub-agent | Make changes in a repo |
| `run_in_repo` | Run shell command in repo | Execute tests, builds, etc. |

---

## Tool Categories

| Category | Tools | Description |
|----------|-------|-------------|
| **Context** | 4 tools | Manage project state (lightweight, no sub-agents) |
| **Query** | 3 tools | Gather information via read-only sub-agents |
| **Implementation** | 2 tools | Make changes via write-capable sub-agents |

---

## Sub-Agent Tool Access

| Operation | Available Tools |
|-----------|-----------------|
| **Query** (read-only) | `Read`, `Glob`, `Grep`, `Bash` |
| **Implementation** (write) | All tools including `Edit`, `Write`, `MultiEdit` |

---

## Detailed Pages

- [Context Tools](./context-tools.md) - `list_projects`, `switch_project`, `get_project_info`, `list_repos`
- [Query Tools](./query-tools.md) - `query_repo`, `query_all_repos`, `get_repo_structure`
- [Implementation Tools](./implementation-tools.md) - `implement_in_repo`, `run_in_repo`

---

## See Also

- [CLI Reference](../cli/) - Command-line interface
- [Configuration](../getting-started/configuration.md) - Server and agent settings
