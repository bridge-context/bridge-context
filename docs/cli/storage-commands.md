---
layout: default
title: Storage Commands
parent: CLI Reference
nav_order: 4
---

# Storage Commands

Commands for managing the storage backend. Context Bridge uses SQLite for persistent session tracking, caching, and metadata storage.

---

## storage info

Display storage configuration and status.

### Syntax

```bash
context-bridge storage info [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Show storage info
context-bridge storage info

# JSON output
context-bridge storage info --json
```

### Output

```
Storage Configuration
═════════════════════

  Adapter: sqlite
  Database: /Users/dev/.context-bridge/storage.db
  Status: ✓ Connected

  Database Info:
    Size: 2.4 MB
    Tables: 5
    Schema version: 3

  Location:
    Config: ~/.context-bridge/app-config.yaml
    Database: ~/.context-bridge/storage.db
```

### JSON Output

```json
{
  "adapter": "sqlite",
  "path": "/Users/dev/.context-bridge/storage.db",
  "connected": true,
  "info": {
    "size": 2516582,
    "sizeHuman": "2.4 MB",
    "tables": 5,
    "schemaVersion": 3
  },
  "config": {
    "path": "~/.context-bridge/app-config.yaml"
  }
}
```

---

## storage stats

Display storage statistics for sessions, cache, and queries.

### Syntax

```bash
context-bridge storage stats [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--project <name>` | Show stats for a specific project |
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Show overall storage stats
context-bridge storage stats

# Show stats for a specific project
context-bridge storage stats --project ecommerce

# JSON output
context-bridge storage stats --json
```

### Output

```
Storage Statistics
══════════════════

Overall:
  Total sessions: 156
  Total queries: 1,247
  Cache entries: 89
  Database size: 2.4 MB

By Project:
  ecommerce
    Sessions: 78
    Queries: 623
    Cache size: 1.1 MB
    Last accessed: 2024-01-15 10:30:00

  mobile-app
    Sessions: 45
    Queries: 412
    Cache size: 0.8 MB
    Last accessed: 2024-01-14 16:45:00

  internal-tools
    Sessions: 33
    Queries: 212
    Cache size: 0.5 MB
    Last accessed: 2024-01-10 09:15:00
```

### Project-Specific Output

```
Storage Statistics: ecommerce
═════════════════════════════

Sessions:
  Total: 78
  This week: 12
  Today: 3

Queries:
  Total: 623
  Successful: 601 (96.5%)
  Failed: 22 (3.5%)
  Average duration: 2.3s

Cache:
  Entries: 34
  Size: 1.1 MB
  Hit rate: 78.4%

Repositories:
  api: 234 queries
  frontend: 287 queries
  shared: 102 queries
```

### JSON Output

```json
{
  "overall": {
    "sessions": 156,
    "queries": 1247,
    "cacheEntries": 89,
    "databaseSize": 2516582,
    "databaseSizeHuman": "2.4 MB"
  },
  "projects": [
    {
      "name": "ecommerce",
      "sessions": 78,
      "queries": 623,
      "cacheSize": 1153434,
      "cacheSizeHuman": "1.1 MB",
      "lastAccessed": "2024-01-15T10:30:00Z"
    }
  ]
}
```

---

## storage clear

Clear cached data or project-specific storage.

### Syntax

```bash
context-bridge storage clear <target> [options]
```

### Targets

| Target | Description |
|--------|-------------|
| `--cache` | Clear all cached query results |
| `--project <name>` | Clear data for a specific project |
| `--all` | Clear all storage data (use with caution) |

### Options

| Option | Description |
|--------|-------------|
| `--force`, `-f` | Skip confirmation prompt |
| `--help` | Display help for this command |

### Examples

```bash
# Clear cache
context-bridge storage clear --cache

# Clear project data
context-bridge storage clear --project old-project

# Clear all data (with confirmation)
context-bridge storage clear --all

# Force clear without confirmation
context-bridge storage clear --cache --force
```

### Cache Clear Output

```
⚠ Clear all cached data?

  This will remove:
    - 89 cache entries
    - ~1.5 MB of cached query results

  Session history will be preserved.

  Confirm? [y/N] y

Clearing cache...
  ✓ Removed 89 cache entries
  ✓ Freed 1.5 MB

✓ Cache cleared successfully
```

### Project Clear Output

```
⚠ Clear storage for project "old-project"?

  This will remove:
    - 33 sessions
    - 212 query results
    - 12 cache entries

  Project configuration will be preserved.

  Confirm? [y/N] y

Clearing project data...
  ✓ Removed sessions
  ✓ Removed query history
  ✓ Removed cache entries

✓ Project "old-project" data cleared
```

---

## storage export

Export storage data to a file.

### Syntax

```bash
context-bridge storage export <file> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `file` | Yes | Output file path (.json or .db) |

### Options

| Option | Description |
|--------|-------------|
| `--format` | Export format: `json` or `sqlite` (default: auto-detect from extension) |
| `--project <name>` | Export only data for a specific project |
| `--include-cache` | Include cache data in export (default: false) |
| `--help` | Display help for this command |

### Examples

```bash
# Export as JSON
context-bridge storage export backup.json

# Export as SQLite database copy
context-bridge storage export backup.db

# Export specific project
context-bridge storage export ecommerce-backup.json --project ecommerce

# Include cache in export
context-bridge storage export full-backup.json --include-cache
```

### Output

```
Exporting storage data...
─────────────────────────

  Format: JSON
  Output: /Users/dev/backups/context-bridge-backup.json

  Exporting:
    ✓ Projects: 3
    ✓ Sessions: 156
    ✓ Queries: 1,247
    ✓ Metadata: included

  File size: 4.2 MB

✓ Export completed: /Users/dev/backups/context-bridge-backup.json
```

### Export Formats

| Format | Extension | Use Case |
|--------|-----------|----------|
| JSON | `.json` | Human-readable, version control friendly |
| SQLite | `.db` | Complete database copy, faster for large datasets |

---

## storage migrate

Run database migrations to update the schema.

### Syntax

```bash
context-bridge storage migrate [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--dry-run` | Show migrations that would run without applying |
| `--force`, `-f` | Force migration even if schema appears up-to-date |
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Run pending migrations
context-bridge storage migrate

# Preview migrations without applying
context-bridge storage migrate --dry-run

# Force migration
context-bridge storage migrate --force
```

### Output

```
Database Migration
══════════════════

  Current schema version: 2
  Target schema version: 3

  Pending migrations:
    - 003_add_cache_metadata.sql
    - 003_add_query_stats.sql

  Creating backup...
    ✓ Backup saved: ~/.context-bridge/storage.db.bak

  Running migrations...
    ✓ 003_add_cache_metadata.sql
    ✓ 003_add_query_stats.sql

  Verifying schema...
    ✓ Schema version: 3
    ✓ All tables valid

✓ Migration completed successfully
```

### Dry Run Output

```
Database Migration (Dry Run)
════════════════════════════

  Current schema version: 2
  Target schema version: 3

  Migrations that would run:
    1. 003_add_cache_metadata.sql
       - Adds metadata column to cache table
       - Adds index on cache_key

    2. 003_add_query_stats.sql
       - Creates query_stats table
       - Migrates existing query duration data

  Estimated changes:
    - New columns: 1
    - New tables: 1
    - New indexes: 2

  No changes made (dry run mode)
```

---

## storage doctor

Check storage health and diagnose issues.

### Syntax

```bash
context-bridge storage doctor [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--repair` | Attempt to repair detected issues |
| `--json` | Output in JSON format |
| `--help` | Display help for this command |

### Examples

```bash
# Run storage health check
context-bridge storage doctor

# Attempt repairs
context-bridge storage doctor --repair

# JSON output
context-bridge storage doctor --json
```

### Output

```
Storage Health Check
════════════════════

  Database Connection
    ✓ Connection established
    ✓ Read/write access confirmed

  Schema Integrity
    ✓ Schema version: 3
    ✓ All required tables present
    ✓ Foreign key constraints valid

  Data Integrity
    ✓ No orphaned records
    ✓ No corrupted entries
    ⚠ 3 stale cache entries (older than 30 days)

  Performance
    ✓ Indexes present and valid
    ✓ No long-running queries detected
    ✓ Database size within limits

Summary:
  3 checks passed, 1 warning

  Recommendation: Run "storage clear --cache" to remove stale entries
```

### Repair Mode Output

```
Storage Health Check (Repair Mode)
══════════════════════════════════

  Database Connection
    ✓ Connection established

  Schema Integrity
    ✓ Schema valid

  Data Integrity
    ⚠ Found 3 stale cache entries
    → Removing stale entries...
    ✓ Removed 3 entries

    ⚠ Found 1 orphaned session record
    → Cleaning orphaned records...
    ✓ Removed 1 record

  Performance
    ✓ All checks passed

Summary:
  Repairs completed successfully
  - Removed 3 stale cache entries
  - Removed 1 orphaned record
```

### JSON Output

```json
{
  "checks": [
    {
      "name": "Database Connection",
      "status": "pass"
    },
    {
      "name": "Schema Integrity",
      "status": "pass",
      "details": {
        "version": 3,
        "tables": ["projects", "repos", "sessions", "queries", "cache"]
      }
    },
    {
      "name": "Data Integrity",
      "status": "warning",
      "issues": [
        {
          "type": "stale_cache",
          "count": 3,
          "recommendation": "Run storage clear --cache"
        }
      ]
    },
    {
      "name": "Performance",
      "status": "pass"
    }
  ],
  "summary": {
    "passed": 3,
    "warnings": 1,
    "failed": 0,
    "success": true
  }
}
```

---

## See Also

- [Configuration](../getting-started/configuration) - Storage configuration options
- [Utility Commands](./utility-commands) - The `doctor` command for overall health checks
