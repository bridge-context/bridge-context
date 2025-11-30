---
layout: default
title: Storage Layer
parent: Architecture
nav_order: 2
---

# Storage Layer

Deep dive into Context Bridge's storage layer architecture, including the adapter interface, database schema, caching mechanism, and event system.

---

## Overview

The storage layer provides persistent data storage for Context Bridge, handling session management, query caching, and execution logging. It uses the adapter pattern to support different storage backends.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Storage Layer                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │   Adapter    │ →  │   Schema     │ →  │     Cache            │  │
│  │              │    │              │    │                      │  │
│  │  • SQLite    │    │  • Drizzle   │    │  • Query results     │  │
│  │  • WAL mode  │    │  • Migrations│    │  • TTL enforcement   │  │
│  │  • better-   │    │  • Types     │    │  • Key generation    │  │
│  │    sqlite3   │    │              │    │                      │  │
│  └──────────────┘    └──────────────┘    └──────────────────────┘  │
│         │                                          │                │
│         ▼                                          ▼                │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                     Event System                               │ │
│  │  • session:created  • query:executed  • cache:hit/miss        │ │
│  └────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## StorageAdapter Interface

All storage implementations must implement this interface:

```typescript
interface StorageAdapter {
  /**
   * Initialize the storage connection
   */
  connect(): Promise<void>;

  /**
   * Close the storage connection
   */
  disconnect(): Promise<void>;

  /**
   * Check if storage is connected and healthy
   */
  isHealthy(): Promise<boolean>;

  // Session Management
  createSession(data: SessionCreate): Promise<Session>;
  getSession(id: string): Promise<Session | null>;
  updateSession(id: string, data: SessionUpdate): Promise<Session>;
  listSessions(filter?: SessionFilter): Promise<Session[]>;

  // Query History
  logQuery(data: QueryLogCreate): Promise<QueryLog>;
  getQueryHistory(filter?: QueryFilter): Promise<QueryLog[]>;

  // Sub-Agent Execution Logging
  logExecution(data: ExecutionLogCreate): Promise<ExecutionLog>;
  getExecutionHistory(filter?: ExecutionFilter): Promise<ExecutionLog[]>;

  // Cache Operations
  getCached(key: string): Promise<CacheEntry | null>;
  setCached(key: string, value: string, ttl: number): Promise<void>;
  invalidateCache(pattern?: string): Promise<number>;

  // Statistics
  getStats(projectId?: string): Promise<StorageStats>;

  // Event Emitter
  on<E extends keyof StorageEvents>(
    event: E,
    handler: StorageEvents[E]
  ): void;
}
```

---

## Database Schema

Context Bridge uses SQLite with Drizzle ORM for type-safe database access.

### Entity Relationship Diagram

```
┌─────────────┐       ┌─────────────┐       ┌─────────────────────┐
│  projects   │       │    repos    │       │      sessions       │
├─────────────┤       ├─────────────┤       ├─────────────────────┤
│ id (PK)     │←──┐   │ id (PK)     │   ┌──→│ id (PK)             │
│ name        │   │   │ project_id  │───┤   │ project_id (FK)     │
│ description │   │   │ name        │   │   │ created_at          │
│ created_at  │   │   │ path        │   │   │ last_activity       │
│ updated_at  │   │   │ description │   │   │ status              │
└─────────────┘   │   │ created_at  │   │   │ metadata            │
                  │   └─────────────┘   │   └─────────────────────┘
                  │                     │             │
                  │   ┌─────────────────┴─────────────┘
                  │   │
                  │   ▼
┌─────────────────┴───────────────┐     ┌─────────────────────────┐
│          mcp_calls              │     │   sub_agent_executions  │
├─────────────────────────────────┤     ├─────────────────────────┤
│ id (PK)                         │     │ id (PK)                 │
│ session_id (FK)                 │     │ session_id (FK)         │
│ tool_name                       │     │ repo_id (FK)            │
│ parameters                      │     │ prompt                  │
│ result                          │     │ result                  │
│ duration_ms                     │     │ tool_restriction        │
│ created_at                      │     │ duration_ms             │
│ error                           │     │ exit_code               │
└─────────────────────────────────┘     │ created_at              │
                                        │ error                   │
┌─────────────────────────────────┐     └─────────────────────────┘
│         query_history           │
├─────────────────────────────────┤     ┌─────────────────────────┐
│ id (PK)                         │     │        settings         │
│ session_id (FK)                 │     ├─────────────────────────┤
│ repo_id (FK)                    │     │ key (PK)                │
│ query                           │     │ value                   │
│ result                          │     │ updated_at              │
│ cached                          │     └─────────────────────────┘
│ duration_ms                     │
│ created_at                      │
└─────────────────────────────────┘
```

### Table Definitions

#### projects

Stores project configurations.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `name` | TEXT | Unique project name |
| `description` | TEXT | Optional description |
| `created_at` | INTEGER | Unix timestamp |
| `updated_at` | INTEGER | Unix timestamp |

#### repos

Stores repository configurations within projects.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `project_id` | TEXT | Foreign key to projects |
| `name` | TEXT | Repository name (unique within project) |
| `path` | TEXT | Absolute filesystem path |
| `description` | TEXT | Optional description |
| `created_at` | INTEGER | Unix timestamp |

#### sessions

Tracks user sessions for observability.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `project_id` | TEXT | Foreign key to projects |
| `created_at` | INTEGER | Session start timestamp |
| `last_activity` | INTEGER | Last activity timestamp |
| `status` | TEXT | 'active', 'stale', 'closed' |
| `metadata` | TEXT | JSON metadata blob |

#### mcp_calls

Logs all MCP tool invocations.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `session_id` | TEXT | Foreign key to sessions |
| `tool_name` | TEXT | Name of the MCP tool called |
| `parameters` | TEXT | JSON parameters passed |
| `result` | TEXT | JSON result returned |
| `duration_ms` | INTEGER | Execution time |
| `created_at` | INTEGER | Unix timestamp |
| `error` | TEXT | Error message if failed |

#### sub_agent_executions

Logs sub-agent process executions.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `session_id` | TEXT | Foreign key to sessions |
| `repo_id` | TEXT | Foreign key to repos |
| `prompt` | TEXT | Prompt sent to sub-agent |
| `result` | TEXT | Sub-agent output |
| `tool_restriction` | TEXT | 'read-only' or 'write' |
| `duration_ms` | INTEGER | Execution time |
| `exit_code` | INTEGER | Process exit code |
| `created_at` | INTEGER | Unix timestamp |
| `error` | TEXT | Error message if failed |

#### query_history

Stores query execution history with cache status.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `session_id` | TEXT | Foreign key to sessions |
| `repo_id` | TEXT | Foreign key to repos |
| `query` | TEXT | Original query text |
| `result` | TEXT | Query result |
| `cached` | INTEGER | 1 if result was from cache |
| `duration_ms` | INTEGER | Execution time |
| `created_at` | INTEGER | Unix timestamp |

#### settings

Stores application settings as key-value pairs.

| Column | Type | Description |
|--------|------|-------------|
| `key` | TEXT | Primary key (setting name) |
| `value` | TEXT | Setting value (JSON) |
| `updated_at` | INTEGER | Unix timestamp |

---

## SQLite Adapter

The default storage implementation uses SQLite.

### Features

| Feature | Implementation |
|---------|----------------|
| WAL Mode | Write-Ahead Logging for concurrent reads |
| Connection Pool | Single connection with better-sqlite3 |
| Auto-Migrations | Drizzle Kit migrations |
| Synchronous Writes | Immediate durability |
| JSON Support | JSON1 extension for metadata |

### WAL Mode

Write-Ahead Logging enables concurrent reads during writes:

```typescript
// Enabled automatically on connection
db.pragma('journal_mode = WAL');
db.pragma('synchronous = NORMAL');
db.pragma('foreign_keys = ON');
```

**Benefits:**
- Concurrent readers don't block writers
- Readers don't block writers
- Better crash recovery
- Improved write performance

### Auto-Migrations

Drizzle Kit handles schema migrations automatically:

```typescript
// Migration on startup
import { migrate } from 'drizzle-orm/better-sqlite3/migrator';

await migrate(db, { migrationsFolder: './migrations' });
```

Migration files are stored in `src/storage/migrations/` and run automatically when the database is opened.

### Drizzle ORM Usage

```typescript
// Type-safe queries
const sessions = await db
  .select()
  .from(sessionsTable)
  .where(eq(sessionsTable.projectId, projectId))
  .orderBy(desc(sessionsTable.lastActivity));

// Type-safe inserts
const [session] = await db
  .insert(sessionsTable)
  .values({
    id: uuid(),
    projectId,
    createdAt: Date.now(),
    lastActivity: Date.now(),
    status: 'active',
  })
  .returning();
```

---

## Query Caching

Query results are cached to avoid redundant sub-agent executions.

### Cache Key Generation

```typescript
function generateCacheKey(query: string, repoId: string, projectId: string): string {
  const normalized = normalizeQuery(query);
  const input = `${normalized}:${repoId}:${projectId}`;
  return crypto.createHash('sha256').update(input).digest('hex');
}

function normalizeQuery(query: string): string {
  return query
    .toLowerCase()
    .trim()
    .replace(/\s+/g, ' ');  // Collapse whitespace
}
```

### Cache Entry Structure

```typescript
interface CacheEntry {
  key: string;           // SHA-256 hash
  value: string;         // Cached result (JSON)
  createdAt: number;     // Unix timestamp
  expiresAt: number;     // TTL expiration
  hitCount: number;      // Access counter
}
```

### TTL Enforcement

```typescript
const CACHE_TTL = 3600000; // 1 hour in milliseconds

async getCached(key: string): Promise<CacheEntry | null> {
  const entry = await db.query.cache.findFirst({
    where: eq(cacheTable.key, key)
  });

  if (!entry) return null;

  // Check TTL
  if (Date.now() > entry.expiresAt) {
    await this.invalidateCache(key);
    return null;
  }

  // Update hit count
  await db.update(cacheTable)
    .set({ hitCount: entry.hitCount + 1 })
    .where(eq(cacheTable.key, key));

  return entry;
}
```

### Cache Behavior

| Scenario | Behavior |
|----------|----------|
| Query, cache miss | Execute sub-agent, cache result |
| Query, cache hit | Return cached result |
| Query, cache expired | Execute sub-agent, update cache |
| Implementation | Never cached (always fresh) |
| Manual clear | Invalidate matching entries |

---

## Event System

The storage layer emits events for real-time updates.

### StorageEvents Type

```typescript
interface StorageEvents {
  // Session events
  'session:created': (session: Session) => void;
  'session:updated': (session: Session) => void;
  'session:closed': (sessionId: string) => void;

  // Query events
  'query:executed': (log: QueryLog) => void;

  // Execution events
  'execution:started': (data: { sessionId: string; repoId: string }) => void;
  'execution:completed': (log: ExecutionLog) => void;
  'execution:failed': (data: { sessionId: string; error: string }) => void;

  // Cache events
  'cache:hit': (key: string) => void;
  'cache:miss': (key: string) => void;
  'cache:invalidated': (pattern: string, count: number) => void;

  // MCP events
  'mcp:call': (log: McpCallLog) => void;
}
```

### Event Usage

```typescript
// Emit events from storage operations
async createSession(data: SessionCreate): Promise<Session> {
  const session = await this.insertSession(data);
  this.emit('session:created', session);
  return session;
}

// Subscribe to events (e.g., from TUI)
storage.on('query:executed', (log) => {
  dashboard.updateQueryList(log);
});

storage.on('cache:hit', (key) => {
  metrics.incrementCacheHits();
});
```

### Event-Driven TUI Updates

The TUI dashboard subscribes to storage events for real-time updates:

```typescript
// In TUI store
useEffect(() => {
  const unsubscribe = [
    storage.on('session:created', () => refreshSessions()),
    storage.on('query:executed', () => refreshQueries()),
    storage.on('execution:completed', () => refreshActivity()),
  ];

  return () => unsubscribe.forEach(fn => fn());
}, []);
```

---

## Session Management

Sessions track user activity and provide context for queries.

### Session Lifecycle

```
1. Session Created
   └─ When first MCP call received
   └─ Status: 'active'

2. Session Active
   └─ Tracks MCP calls, queries, executions
   └─ last_activity updated on each operation

3. Session Stale
   └─ No activity for 24 hours
   └─ Status: 'stale'
   └─ Can be resumed if new activity

4. Session Closed
   └─ Explicitly closed or garbage collected
   └─ Status: 'closed'
```

### 24-Hour Staleness

Sessions are marked stale after 24 hours of inactivity:

```typescript
const STALE_THRESHOLD = 24 * 60 * 60 * 1000; // 24 hours

async markStaleSessions(): Promise<number> {
  const staleTime = Date.now() - STALE_THRESHOLD;

  const result = await db.update(sessionsTable)
    .set({ status: 'stale' })
    .where(
      and(
        eq(sessionsTable.status, 'active'),
        lt(sessionsTable.lastActivity, staleTime)
      )
    );

  return result.changes;
}
```

### Session Resolution

When an MCP call is received, the session is resolved:

```typescript
async resolveSession(projectId: string): Promise<Session> {
  // Try to find active session for this project
  const existing = await this.getActiveSession(projectId);

  if (existing) {
    // Update last activity
    return this.updateSession(existing.id, {
      lastActivity: Date.now()
    });
  }

  // Create new session
  return this.createSession({
    projectId,
    status: 'active'
  });
}
```

---

## Storage Statistics

The storage layer provides statistics for monitoring:

```typescript
interface StorageStats {
  sessions: {
    total: number;
    active: number;
    stale: number;
  };
  queries: {
    total: number;
    cached: number;
    avgDuration: number;
  };
  executions: {
    total: number;
    successful: number;
    failed: number;
    avgDuration: number;
  };
  cache: {
    entries: number;
    hitRate: number;
    size: number;  // bytes
  };
  database: {
    size: number;  // bytes
    version: number;
  };
}
```

---

## See Also

- [Architecture Overview](./index) - High-level system architecture
- [Agent System](./agent-system) - Sub-agent spawning and execution
- [Storage Commands](../cli/storage-commands) - CLI storage management
- [Configuration](../getting-started/configuration) - Storage configuration options
