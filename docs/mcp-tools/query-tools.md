---
layout: default
title: Query Tools
parent: MCP Tools
nav_order: 2
---

# Query Tools

Query tools gather information from repositories by spawning read-only sub-agents. Results are cached to improve performance for repeated queries.

---

## query_repo

Query a specific repository using natural language.

### Description

Spawns a read-only sub-agent in the specified repository to answer questions about the codebase. The sub-agent can read files, search code, and analyze structure, but cannot make modifications.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `repo` | string | Yes | Name of the repository to query |
| `query` | string | Yes | Natural language question or request |

### Example Request

```json
{
  "tool": "query_repo",
  "arguments": {
    "repo": "api",
    "query": "How is authentication implemented? What middleware is used?"
  }
}
```

### Example Response

```json
{
  "success": true,
  "repo": "api",
  "query": "How is authentication implemented? What middleware is used?",
  "result": "Authentication is implemented using JWT tokens with the following components:\n\n1. **Middleware** (`src/middleware/auth.ts`):\n   - `authenticateToken` - Validates JWT from Authorization header\n   - `requireRole` - Role-based access control\n\n2. **Token Generation** (`src/services/auth.service.ts`):\n   - Uses `jsonwebtoken` library\n   - Tokens expire after 24 hours\n   - Refresh tokens stored in Redis\n\n3. **Routes** (`src/routes/auth.routes.ts`):\n   - POST /auth/login - User login\n   - POST /auth/refresh - Token refresh\n   - POST /auth/logout - Invalidate tokens",
  "cached": false,
  "executionTime": 4523,
  "subAgentId": "sa_abc123"
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether the query completed successfully |
| `repo` | string | Repository that was queried |
| `query` | string | Original query (for reference) |
| `result` | string | Sub-agent's response |
| `cached` | boolean | Whether result came from cache |
| `executionTime` | number | Time in milliseconds (0 if cached) |
| `subAgentId` | string | Identifier for debugging |

### Sub-Agent Tools

The query sub-agent has access to read-only tools:

| Tool | Description |
|------|-------------|
| `Read` | Read file contents |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents |
| `Bash` | Execute read-only shell commands |

### Errors

| Code | Message | Cause |
|------|---------|-------|
| `REPO_NOT_FOUND` | Repository 'xyz' not found | Repo not in current project |
| `AGENT_TIMEOUT` | Query timed out after 300000ms | Sub-agent exceeded time limit |
| `AGENT_ERROR` | Sub-agent encountered an error | Internal sub-agent failure |

### Notes

- Queries are cached for 1 hour (see [Caching](#query-caching) below)
- Complex queries may take 10-30 seconds
- Use specific, focused questions for best results

---

## query_all_repos

Query all repositories in the current project in parallel.

### Description

Spawns read-only sub-agents in all project repositories simultaneously. Useful for cross-repository questions like finding shared patterns, understanding integrations, or locating code across the project.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Natural language question or request |

### Example Request

```json
{
  "tool": "query_all_repos",
  "arguments": {
    "query": "Where is the User model defined and how is it used?"
  }
}
```

### Example Response

```json
{
  "success": true,
  "query": "Where is the User model defined and how is it used?",
  "results": [
    {
      "repo": "api",
      "success": true,
      "result": "The User model is defined in `src/models/user.model.ts`:\n\n```typescript\nexport interface User {\n  id: string;\n  email: string;\n  name: string;\n  role: 'admin' | 'user';\n  createdAt: Date;\n}\n```\n\nUsed in:\n- `src/services/user.service.ts` - CRUD operations\n- `src/controllers/user.controller.ts` - HTTP handlers\n- `src/middleware/auth.ts` - Token payload typing",
      "cached": false,
      "executionTime": 3250
    },
    {
      "repo": "frontend",
      "success": true,
      "result": "The frontend imports the User type from `@shared/types`:\n\n```typescript\nimport { User } from '@shared/types';\n```\n\nUsed in:\n- `src/contexts/AuthContext.tsx` - Current user state\n- `src/components/UserProfile.tsx` - Display user info\n- `src/hooks/useUser.ts` - User data fetching",
      "cached": true,
      "executionTime": 0
    },
    {
      "repo": "shared",
      "success": false,
      "error": {
        "code": "PATH_INVALID",
        "message": "Repository path does not exist"
      }
    }
  ],
  "summary": {
    "total": 3,
    "succeeded": 2,
    "failed": 1,
    "cached": 1,
    "totalExecutionTime": 3250
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether at least one query succeeded |
| `query` | string | Original query |
| `results` | array | Results from each repository |
| `results[].repo` | string | Repository name |
| `results[].success` | boolean | Whether this query succeeded |
| `results[].result` | string | Sub-agent response (if successful) |
| `results[].error` | object | Error details (if failed) |
| `results[].cached` | boolean | Whether result was cached |
| `results[].executionTime` | number | Execution time in ms |
| `summary.total` | number | Total repositories queried |
| `summary.succeeded` | number | Successful queries |
| `summary.failed` | number | Failed queries |
| `summary.cached` | number | Results from cache |
| `summary.totalExecutionTime` | number | Sum of execution times |

### Notes

- Queries run in parallel across all repositories
- Failed repositories don't stop other queries
- Total time approximately equals slowest query (not sum)
- Each repository result is cached independently

---

## get_repo_structure

Get a high-level overview of a repository's structure.

### Description

Provides a structural summary of a repository including directory layout, key files, and technology stack. Useful for understanding an unfamiliar codebase before diving into specific queries.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `repo` | string | Yes | Name of the repository |

### Example Request

```json
{
  "tool": "get_repo_structure",
  "arguments": {
    "repo": "api"
  }
}
```

### Example Response

```json
{
  "success": true,
  "repo": "api",
  "structure": {
    "name": "ecommerce-api",
    "type": "Node.js Application",
    "rootFiles": [
      "package.json",
      "tsconfig.json",
      ".env.example",
      "README.md"
    ],
    "directories": [
      {
        "name": "src",
        "description": "Source code",
        "children": [
          { "name": "controllers", "fileCount": 8 },
          { "name": "services", "fileCount": 10 },
          { "name": "models", "fileCount": 6 },
          { "name": "middleware", "fileCount": 4 },
          { "name": "routes", "fileCount": 7 },
          { "name": "utils", "fileCount": 12 }
        ]
      },
      {
        "name": "tests",
        "description": "Test files",
        "children": [
          { "name": "unit", "fileCount": 24 },
          { "name": "integration", "fileCount": 8 }
        ]
      },
      {
        "name": "config",
        "description": "Configuration files",
        "fileCount": 3
      }
    ],
    "technologies": [
      "TypeScript",
      "Express.js",
      "PostgreSQL (Prisma)",
      "Redis",
      "Jest"
    ],
    "entryPoint": "src/index.ts",
    "scripts": {
      "start": "node dist/index.js",
      "dev": "ts-node-dev src/index.ts",
      "build": "tsc",
      "test": "jest"
    }
  },
  "cached": true,
  "executionTime": 0
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether retrieval succeeded |
| `repo` | string | Repository name |
| `structure.name` | string | Repository/package name |
| `structure.type` | string | Detected project type |
| `structure.rootFiles` | array | Important root-level files |
| `structure.directories` | array | Directory structure |
| `structure.technologies` | array | Detected tech stack |
| `structure.entryPoint` | string | Main entry file |
| `structure.scripts` | object | Available npm/package scripts |
| `cached` | boolean | Whether result was cached |
| `executionTime` | number | Time in milliseconds |

### Notes

- Structure is analyzed on first request and cached
- Detection based on config files (package.json, etc.)
- Deep directory trees are summarized
- Use before `query_repo` to understand codebase organization

---

## Query Caching

Query tools implement intelligent caching to improve performance and reduce redundant sub-agent operations.

### Cache Mechanism

1. **Key Generation**: Queries are normalized and hashed
2. **Storage**: Results stored in SQLite database
3. **Retrieval**: Cache checked before spawning sub-agent
4. **Expiration**: Entries expire after 1 hour (TTL: 3600 seconds)

### Cache Key Structure

Cache keys are generated using SHA-256 hash:

```
SHA-256(
  normalize(query) +
  repo_id +
  project_id
)
```

**Normalization steps:**
- Convert to lowercase
- Remove extra whitespace
- Trim leading/trailing spaces
- Remove punctuation variations

### Cache Key Examples

```
Query: "How is authentication implemented?"
Repo: "api"
Project: "ecommerce"

Normalized: "how is authentication implemented"
Key: SHA-256("how is authentication implementedapiecommerce")
Result: "a7f3b2c1d4e5f6..."
```

### Cache Behavior Matrix

| Scenario | Cache Hit? | Result |
|----------|------------|--------|
| Exact same query, same repo | Yes | Cached result |
| Same query, different case | Yes | Cached result |
| Same query, extra spaces | Yes | Cached result |
| Same query, different repo | No | New sub-agent |
| Similar but different query | No | New sub-agent |
| Same query after 1 hour | No | New sub-agent |

### TTL Configuration

The default TTL is 1 hour (3600 seconds). This balances:
- **Freshness**: Code changes within an hour are reflected
- **Performance**: Repeated queries are fast
- **Cost**: Reduces sub-agent spawning

### Forcing Fresh Results

To bypass the cache:

```bash
# Clear all cached queries
context-bridge storage clear --cache

# Clear cache for specific project
context-bridge storage clear --cache --project ecommerce
```

### Cache Storage

Cache entries are stored in SQLite:

```sql
-- Simplified schema
CREATE TABLE query_cache (
  cache_key TEXT PRIMARY KEY,
  query TEXT,
  repo TEXT,
  project TEXT,
  result TEXT,
  created_at TIMESTAMP,
  expires_at TIMESTAMP
);
```

### Monitoring Cache Performance

```bash
# View cache statistics
context-bridge storage stats

# Output includes:
#   Cache entries: 47
#   Cache hit rate: 73%
#   Cache size: 2.3 MB
#   Oldest entry: 45 minutes ago
```

---

## Best Practices

### Writing Effective Queries

**Good queries:**
- "How is user authentication implemented?"
- "What API endpoints handle order processing?"
- "Show me the database schema for products"

**Less effective queries:**
- "Tell me everything about the code" (too broad)
- "Fix the bug" (requires write access)
- "Is this good code?" (subjective)

### Query Strategies

1. **Start with structure**: Use `get_repo_structure` first
2. **Be specific**: Ask about particular features or files
3. **Iterate**: Follow up with more detailed questions
4. **Use parallel queries**: `query_all_repos` for cross-cutting concerns

### Performance Tips

- Leverage caching by using consistent query phrasing
- Use `query_all_repos` instead of multiple `query_repo` calls
- Check `cached` field to understand response source
- Monitor execution times for optimization

---

## See Also

- [Context Tools](./context-tools.md) - Project and repository management
- [Implementation Tools](./implementation-tools.md) - Write-capable tools
- [Storage Commands](../cli/storage-commands.md) - Cache management
