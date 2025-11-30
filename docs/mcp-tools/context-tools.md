---
layout: default
title: Context Tools
parent: MCP Tools
nav_order: 1
---

# Context Tools

Context tools manage project state and provide information about available repositories. These are lightweight, synchronous operations that don't spawn sub-agents.

---

## list_projects

List all available project contexts.

### Description

Returns a list of all projects configured in Context Bridge. Use this to discover available projects before switching context or to verify project setup.

### Parameters

This tool takes no parameters.

### Example Request

```json
{
  "tool": "list_projects",
  "arguments": {}
}
```

### Example Response

```json
{
  "projects": [
    {
      "name": "ecommerce",
      "description": "E-commerce platform with microservices",
      "active": true,
      "repoCount": 3,
      "createdAt": "2024-01-10T08:00:00Z",
      "lastAccessed": "2024-01-15T14:30:00Z"
    },
    {
      "name": "mobile-app",
      "description": "iOS and Android applications",
      "active": false,
      "repoCount": 2,
      "createdAt": "2024-01-12T10:00:00Z",
      "lastAccessed": "2024-01-14T09:15:00Z"
    },
    {
      "name": "internal-tools",
      "description": null,
      "active": false,
      "repoCount": 1,
      "createdAt": "2024-01-08T16:00:00Z",
      "lastAccessed": "2024-01-08T16:30:00Z"
    }
  ],
  "count": 3,
  "activeProject": "ecommerce"
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `projects` | array | List of project objects |
| `projects[].name` | string | Unique project identifier |
| `projects[].description` | string\|null | Human-readable description |
| `projects[].active` | boolean | Whether this is the current project |
| `projects[].repoCount` | number | Number of repositories in project |
| `projects[].createdAt` | string | ISO 8601 creation timestamp |
| `projects[].lastAccessed` | string | ISO 8601 last access timestamp |
| `count` | number | Total number of projects |
| `activeProject` | string | Name of the currently active project |

### Notes

- Projects are returned in order of last access (most recent first)
- The `active` flag indicates the current working context
- Empty projects (no repositories) are included in the list

---

## switch_project

Switch to a different project context.

### Description

Changes the active project context. All subsequent repository operations will use the new project's repositories until another switch is performed.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `project` | string | Yes | Name of the project to switch to |

### Example Request

```json
{
  "tool": "switch_project",
  "arguments": {
    "project": "mobile-app"
  }
}
```

### Example Response

```json
{
  "success": true,
  "previousProject": "ecommerce",
  "currentProject": "mobile-app",
  "repos": [
    {
      "name": "ios-app",
      "path": "/Users/dev/projects/mobile/ios"
    },
    {
      "name": "android-app",
      "path": "/Users/dev/projects/mobile/android"
    }
  ],
  "repoCount": 2
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether the switch was successful |
| `previousProject` | string | Name of the previously active project |
| `currentProject` | string | Name of the now-active project |
| `repos` | array | List of repositories in the new project |
| `repos[].name` | string | Repository name |
| `repos[].path` | string | Absolute path to repository |
| `repoCount` | number | Number of repositories in new project |

### Errors

| Code | Message | Cause |
|------|---------|-------|
| `PROJECT_NOT_FOUND` | Project 'xyz' not found | Specified project doesn't exist |

### Error Response Example

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "Project 'unknown-project' not found",
    "details": {
      "requested": "unknown-project",
      "available": ["ecommerce", "mobile-app", "internal-tools"]
    }
  }
}
```

### Notes

- Switching projects is instantaneous and doesn't affect running sub-agents
- The switch persists across server restarts
- Use `list_projects` to see available project names

---

## get_project_info

Get detailed information about the current project.

### Description

Returns comprehensive information about the active project, including all repositories, their validation status, and project statistics.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `project` | string | No | Project name (defaults to active project) |

### Example Request

```json
{
  "tool": "get_project_info",
  "arguments": {}
}
```

### Example Response

```json
{
  "name": "ecommerce",
  "description": "E-commerce platform with microservices",
  "active": true,
  "repos": [
    {
      "name": "api",
      "path": "/Users/dev/projects/ecommerce-api",
      "description": "REST API backend (Node.js/Express)",
      "valid": true,
      "stats": {
        "files": 234,
        "directories": 45,
        "languages": ["TypeScript", "JavaScript", "JSON"]
      }
    },
    {
      "name": "frontend",
      "path": "/Users/dev/projects/ecommerce-web",
      "description": "React frontend application",
      "valid": true,
      "stats": {
        "files": 156,
        "directories": 32,
        "languages": ["TypeScript", "CSS", "HTML"]
      }
    },
    {
      "name": "shared",
      "path": "/Users/dev/projects/ecommerce-shared",
      "description": "Shared types and utilities",
      "valid": false,
      "error": "Path does not exist"
    }
  ],
  "stats": {
    "repoCount": 3,
    "validRepos": 2,
    "invalidRepos": 1,
    "totalFiles": 390,
    "createdAt": "2024-01-10T08:00:00Z",
    "lastAccessed": "2024-01-15T14:30:00Z",
    "queryCount": 47,
    "implementationCount": 12
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Project identifier |
| `description` | string\|null | Project description |
| `active` | boolean | Whether this is the active project |
| `repos` | array | List of repository details |
| `repos[].name` | string | Repository name |
| `repos[].path` | string | Absolute path |
| `repos[].description` | string\|null | Repository description |
| `repos[].valid` | boolean | Whether path is accessible |
| `repos[].stats` | object | Repository statistics (if valid) |
| `repos[].error` | string | Error message (if invalid) |
| `stats` | object | Project-level statistics |
| `stats.repoCount` | number | Total repositories |
| `stats.validRepos` | number | Accessible repositories |
| `stats.invalidRepos` | number | Inaccessible repositories |
| `stats.queryCount` | number | Total queries executed |
| `stats.implementationCount` | number | Total implementations executed |

### Request with Project Parameter

```json
{
  "tool": "get_project_info",
  "arguments": {
    "project": "mobile-app"
  }
}
```

### Notes

- Repository statistics are gathered lazily and cached
- Invalid repositories are included with error information
- Use this to verify project health before operations

---

## list_repos

List all repositories in the current project.

### Description

Returns a list of repositories configured in the current (or specified) project. This is a simpler alternative to `get_project_info` when you only need repository names and paths.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `project` | string | No | Project name (defaults to active project) |

### Example Request

```json
{
  "tool": "list_repos",
  "arguments": {}
}
```

### Example Response

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
  "count": 3,
  "validCount": 2
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `project` | string | Name of the project |
| `repos` | array | List of repositories |
| `repos[].name` | string | Repository identifier |
| `repos[].path` | string | Absolute filesystem path |
| `repos[].description` | string\|null | Optional description |
| `repos[].valid` | boolean | Whether path exists and is accessible |
| `count` | number | Total number of repositories |
| `validCount` | number | Number of valid repositories |

### Request with Project Parameter

```json
{
  "tool": "list_repos",
  "arguments": {
    "project": "internal-tools"
  }
}
```

### Example Response (Specific Project)

```json
{
  "project": "internal-tools",
  "repos": [
    {
      "name": "admin-dashboard",
      "path": "/Users/dev/projects/admin",
      "description": "Internal admin dashboard",
      "valid": true
    }
  ],
  "count": 1,
  "validCount": 1
}
```

### Errors

| Code | Message | Cause |
|------|---------|-------|
| `PROJECT_NOT_FOUND` | Project 'xyz' not found | Specified project doesn't exist |
| `NO_ACTIVE_PROJECT` | No active project | No project selected and none specified |

### Notes

- Use this tool before `query_repo` or `implement_in_repo` to get valid repository names
- Invalid repositories are included but marked with `valid: false`
- Repository names are case-sensitive

---

## Usage Patterns

### Discovering and Switching Projects

```
1. Call list_projects to see available projects
2. Note the project with the repos you need
3. Call switch_project to change context
4. Call list_repos to verify available repositories
```

### Verifying Project Health

```
1. Call get_project_info for comprehensive details
2. Check repos[].valid for accessibility issues
3. Review stats for usage patterns
4. Address any invalid repositories via CLI
```

### Common Workflow

```json
// Step 1: List projects
{ "tool": "list_projects" }

// Step 2: Switch to target project
{ "tool": "switch_project", "arguments": { "project": "ecommerce" } }

// Step 3: List available repos
{ "tool": "list_repos" }

// Step 4: Proceed with queries or implementations
{ "tool": "query_repo", "arguments": { "repo": "api", "query": "..." } }
```

---

## See Also

- [Query Tools](./query-tools.md) - Query repository contents
- [Implementation Tools](./implementation-tools.md) - Implement changes in repositories
- [Project Commands](../cli/project-commands.md) - CLI project management
