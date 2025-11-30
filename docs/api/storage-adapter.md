---
layout: default
title: StorageAdapter
parent: API Reference
nav_order: 2
---

# StorageAdapter Interface

The `StorageAdapter` interface defines the contract for data persistence in Context Bridge. All storage backends must implement this interface for managing projects, repositories, sessions, and cached data.

---

## Interface Overview

The StorageAdapter provides methods organized into these categories:

| Category | Description |
|----------|-------------|
| **Lifecycle** | Connection management and health checks |
| **Projects** | Project CRUD operations |
| **Repositories** | Repository management within projects |
| **Sessions** | Session tracking and management |
| **MCP Calls** | MCP tool invocation logging |
| **Sub-Agents** | Sub-agent execution logging |
| **Cache** | Query result caching |
| **Statistics** | Storage metrics and analytics |

---

## Entity Types

### Project

Represents a Context Bridge project containing multiple repositories:

```typescript
interface Project {
  /**
   * Unique identifier (UUID)
   */
  id: string;

  /**
   * Project name (unique)
   */
  name: string;

  /**
   * Optional description
   */
  description?: string;

  /**
   * Creation timestamp (Unix ms)
   */
  createdAt: number;

  /**
   * Last update timestamp (Unix ms)
   */
  updatedAt: number;
}
```

### Repository

Represents a code repository within a project:

```typescript
interface Repository {
  /**
   * Unique identifier (UUID)
   */
  id: string;

  /**
   * Parent project ID
   */
  projectId: string;

  /**
   * Repository name (unique within project)
   */
  name: string;

  /**
   * Absolute filesystem path
   */
  path: string;

  /**
   * Optional description
   */
  description?: string;

  /**
   * Creation timestamp (Unix ms)
   */
  createdAt: number;
}
```

### Session

Represents an active or historical user session:

```typescript
interface Session {
  /**
   * Unique identifier (UUID)
   */
  id: string;

  /**
   * Associated project ID
   */
  projectId: string;

  /**
   * Session creation timestamp (Unix ms)
   */
  createdAt: number;

  /**
   * Last activity timestamp (Unix ms)
   */
  lastActivity: number;

  /**
   * Session status
   */
  status: 'active' | 'stale' | 'closed';

  /**
   * Additional metadata (JSON)
   */
  metadata?: Record<string, unknown>;
}
```

### MCPCall

Logs an MCP tool invocation:

```typescript
interface MCPCall {
  /**
   * Unique identifier (UUID)
   */
  id: string;

  /**
   * Associated session ID
   */
  sessionId: string;

  /**
   * Name of the MCP tool called
   */
  toolName: string;

  /**
   * Parameters passed to the tool (JSON)
   */
  parameters: Record<string, unknown>;

  /**
   * Result returned by the tool (JSON)
   */
  result?: Record<string, unknown>;

  /**
   * Execution duration in milliseconds
   */
  durationMs: number;

  /**
   * Invocation timestamp (Unix ms)
   */
  createdAt: number;

  /**
   * Error message if call failed
   */
  error?: string;
}
```

### SubAgentExecution

Logs a sub-agent process execution:

```typescript
interface SubAgentExecution {
  /**
   * Unique identifier (UUID)
   */
  id: string;

  /**
   * Associated session ID
   */
  sessionId: string;

  /**
   * Target repository ID
   */
  repoId: string;

  /**
   * Prompt sent to sub-agent
   */
  prompt: string;

  /**
   * Sub-agent output
   */
  result: string;

  /**
   * Tool restriction level
   */
  toolRestriction: 'read-only' | 'write';

  /**
   * Execution duration in milliseconds
   */
  durationMs: number;

  /**
   * Process exit code
   */
  exitCode: number;

  /**
   * Execution timestamp (Unix ms)
   */
  createdAt: number;

  /**
   * Error message if execution failed
   */
  error?: string;
}
```

### QueryHistory

Stores query execution history with cache status:

```typescript
interface QueryHistory {
  /**
   * Unique identifier (UUID)
   */
  id: string;

  /**
   * Associated session ID
   */
  sessionId: string;

  /**
   * Target repository ID
   */
  repoId: string;

  /**
   * Original query text
   */
  query: string;

  /**
   * Query result
   */
  result: string;

  /**
   * Whether result was from cache
   */
  cached: boolean;

  /**
   * Execution duration in milliseconds
   */
  durationMs: number;

  /**
   * Execution timestamp (Unix ms)
   */
  createdAt: number;
}
```

### StorageStats

Storage statistics and metrics:

```typescript
interface StorageStats {
  /**
   * Session statistics
   */
  sessions: {
    total: number;
    active: number;
    stale: number;
    closed: number;
  };

  /**
   * Query statistics
   */
  queries: {
    total: number;
    cached: number;
    avgDurationMs: number;
  };

  /**
   * Execution statistics
   */
  executions: {
    total: number;
    successful: number;
    failed: number;
    avgDurationMs: number;
  };

  /**
   * Cache statistics
   */
  cache: {
    entries: number;
    hitRate: number;
    sizeBytes: number;
  };

  /**
   * Database statistics
   */
  database: {
    sizeBytes: number;
    schemaVersion: number;
  };
}
```

---

## Filter Types

### SessionFilter

Filter options for session queries:

```typescript
interface SessionFilter {
  /**
   * Filter by project ID
   */
  projectId?: string;

  /**
   * Filter by status
   */
  status?: 'active' | 'stale' | 'closed';

  /**
   * Filter by creation date (after)
   */
  createdAfter?: number;

  /**
   * Filter by creation date (before)
   */
  createdBefore?: number;

  /**
   * Maximum results to return
   */
  limit?: number;

  /**
   * Offset for pagination
   */
  offset?: number;
}
```

### MCPCallFilter

Filter options for MCP call queries:

```typescript
interface MCPCallFilter {
  /**
   * Filter by session ID
   */
  sessionId?: string;

  /**
   * Filter by tool name
   */
  toolName?: string;

  /**
   * Filter by success/failure
   */
  success?: boolean;

  /**
   * Filter by date (after)
   */
  createdAfter?: number;

  /**
   * Filter by date (before)
   */
  createdBefore?: number;

  /**
   * Maximum results to return
   */
  limit?: number;

  /**
   * Offset for pagination
   */
  offset?: number;
}
```

### SubAgentFilter

Filter options for sub-agent execution queries:

```typescript
interface SubAgentFilter {
  /**
   * Filter by session ID
   */
  sessionId?: string;

  /**
   * Filter by repository ID
   */
  repoId?: string;

  /**
   * Filter by tool restriction
   */
  toolRestriction?: 'read-only' | 'write';

  /**
   * Filter by success (exit code 0)
   */
  success?: boolean;

  /**
   * Filter by date (after)
   */
  createdAfter?: number;

  /**
   * Filter by date (before)
   */
  createdBefore?: number;

  /**
   * Maximum results to return
   */
  limit?: number;

  /**
   * Offset for pagination
   */
  offset?: number;
}
```

---

## Method Categories

### Lifecycle Methods

```typescript
interface StorageAdapter {
  /**
   * Initialize the storage connection
   * Called once at startup
   */
  connect(): Promise<void>;

  /**
   * Close the storage connection
   * Called on graceful shutdown
   */
  disconnect(): Promise<void>;

  /**
   * Check if storage is connected and healthy
   * Used for health checks and diagnostics
   */
  healthCheck(): Promise<boolean>;
}
```

**Example:**

```typescript
const storage = new SQLiteAdapter({ path: './data.db' });

// Initialize
await storage.connect();

// Check health
const healthy = await storage.healthCheck();
console.log('Storage healthy:', healthy);

// Cleanup
await storage.disconnect();
```

---

### Project Methods

```typescript
interface StorageAdapter {
  /**
   * Create a new project
   */
  createProject(data: {
    name: string;
    description?: string;
  }): Promise<Project>;

  /**
   * Get a project by ID
   */
  getProject(id: string): Promise<Project | null>;

  /**
   * Get a project by name
   */
  getProjectByName(name: string): Promise<Project | null>;

  /**
   * List all projects
   */
  listProjects(): Promise<Project[]>;

  /**
   * Update a project
   */
  updateProject(id: string, data: {
    name?: string;
    description?: string;
  }): Promise<Project>;

  /**
   * Delete a project and all associated data
   */
  deleteProject(id: string): Promise<void>;
}
```

---

### Repository Methods

```typescript
interface StorageAdapter {
  /**
   * Add a repository to a project
   */
  addRepository(data: {
    projectId: string;
    name: string;
    path: string;
    description?: string;
  }): Promise<Repository>;

  /**
   * Get a repository by ID
   */
  getRepository(id: string): Promise<Repository | null>;

  /**
   * List repositories in a project
   */
  listRepositories(projectId: string): Promise<Repository[]>;

  /**
   * Update a repository
   */
  updateRepository(id: string, data: {
    name?: string;
    path?: string;
    description?: string;
  }): Promise<Repository>;

  /**
   * Remove a repository from a project
   */
  removeRepository(id: string): Promise<void>;
}
```

---

### Session Methods

```typescript
interface StorageAdapter {
  /**
   * Create a new session
   */
  createSession(data: {
    projectId: string;
    metadata?: Record<string, unknown>;
  }): Promise<Session>;

  /**
   * Get a session by ID
   */
  getSession(id: string): Promise<Session | null>;

  /**
   * List sessions with optional filtering
   */
  listSessions(filter?: SessionFilter): Promise<Session[]>;

  /**
   * Update session (activity, status, metadata)
   */
  updateSession(id: string, data: {
    lastActivity?: number;
    status?: 'active' | 'stale' | 'closed';
    metadata?: Record<string, unknown>;
  }): Promise<Session>;

  /**
   * Get or create an active session for a project
   */
  resolveSession(projectId: string): Promise<Session>;

  /**
   * Mark stale sessions (no activity > 24 hours)
   */
  markStaleSessions(): Promise<number>;
}
```

---

### MCP Call Methods

```typescript
interface StorageAdapter {
  /**
   * Log an MCP tool call
   */
  logMCPCall(data: {
    sessionId: string;
    toolName: string;
    parameters: Record<string, unknown>;
    result?: Record<string, unknown>;
    durationMs: number;
    error?: string;
  }): Promise<MCPCall>;

  /**
   * Get MCP call history with filtering
   */
  getMCPCalls(filter?: MCPCallFilter): Promise<MCPCall[]>;
}
```

---

### Sub-Agent Execution Methods

```typescript
interface StorageAdapter {
  /**
   * Log a sub-agent execution
   */
  logSubAgentExecution(data: {
    sessionId: string;
    repoId: string;
    prompt: string;
    result: string;
    toolRestriction: 'read-only' | 'write';
    durationMs: number;
    exitCode: number;
    error?: string;
  }): Promise<SubAgentExecution>;

  /**
   * Get sub-agent execution history with filtering
   */
  getSubAgentExecutions(filter?: SubAgentFilter): Promise<SubAgentExecution[]>;
}
```

---

### Cache Methods

```typescript
interface StorageAdapter {
  /**
   * Get a cached query result
   */
  getCached(key: string): Promise<string | null>;

  /**
   * Set a cached query result with TTL
   */
  setCached(key: string, value: string, ttlMs: number): Promise<void>;

  /**
   * Invalidate cache entries matching a pattern
   * Returns number of entries invalidated
   */
  invalidateCache(pattern?: string): Promise<number>;

  /**
   * Clear all cache entries
   */
  clearCache(): Promise<void>;
}
```

**Example:**

```typescript
// Cache a query result for 1 hour
const cacheKey = 'query:repo-id:hash';
await storage.setCached(cacheKey, JSON.stringify(result), 3600000);

// Retrieve cached result
const cached = await storage.getCached(cacheKey);
if (cached) {
  return JSON.parse(cached);
}

// Invalidate cache for a specific repo
await storage.invalidateCache('query:repo-id:*');
```

---

### Statistics Methods

```typescript
interface StorageAdapter {
  /**
   * Get storage statistics
   */
  getStats(projectId?: string): Promise<StorageStats>;
}
```

**Example:**

```typescript
const stats = await storage.getStats();

console.log('Total sessions:', stats.sessions.total);
console.log('Cache hit rate:', `${(stats.cache.hitRate * 100).toFixed(1)}%`);
console.log('Database size:', `${(stats.database.sizeBytes / 1024 / 1024).toFixed(2)} MB`);
```

---

## Built-in SQLiteAdapter

Context Bridge includes a SQLite-based storage adapter as the default implementation.

### Features

| Feature | Description |
|---------|-------------|
| **WAL Mode** | Write-Ahead Logging for concurrent access |
| **Drizzle ORM** | Type-safe database queries |
| **Auto-Migrations** | Automatic schema updates |
| **JSON Support** | JSON1 extension for metadata |
| **Connection Pool** | Efficient connection management |

### Configuration

```typescript
import { SQLiteAdapter } from '@agent-context-bridge/cli';

const storage = new SQLiteAdapter({
  /**
   * Path to SQLite database file
   */
  path: '~/.context-bridge/storage.db',

  /**
   * Enable WAL mode (recommended)
   */
  walMode: true,

  /**
   * Enable verbose logging
   */
  verbose: false,
});
```

### Usage

```typescript
// Initialize
await storage.connect();

// Create a project
const project = await storage.createProject({
  name: 'my-project',
  description: 'A multi-repo project',
});

// Add repositories
await storage.addRepository({
  projectId: project.id,
  name: 'api',
  path: '/path/to/api',
});

await storage.addRepository({
  projectId: project.id,
  name: 'frontend',
  path: '/path/to/frontend',
});

// Get or create a session
const session = await storage.resolveSession(project.id);

// Log an MCP call
await storage.logMCPCall({
  sessionId: session.id,
  toolName: 'query_repo',
  parameters: { repo: 'api', query: 'Find all API endpoints' },
  durationMs: 1500,
});

// Get statistics
const stats = await storage.getStats(project.id);
console.log('Project stats:', stats);

// Cleanup
await storage.disconnect();
```

### Database Location

The SQLite database is stored at:

| Platform | Default Path |
|----------|--------------|
| macOS | `~/.context-bridge/storage.db` |
| Linux | `~/.context-bridge/storage.db` |
| Windows | `%USERPROFILE%\.context-bridge\storage.db` |

Override with the `CONTEXT_BRIDGE_STORAGE_PATH` environment variable.

---

## See Also

- [AgentAdapter](./agent-adapter) - Agent adapter interface
- [Storage Layer Architecture](../architecture/storage-layer) - Storage system internals
- [Storage Commands](../cli/storage-commands) - CLI storage management
