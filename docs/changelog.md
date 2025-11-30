---
layout: default
title: Changelog
nav_order: 8
---

# Changelog

All notable changes to Context Bridge are documented here.

---

## v0.4.0 (November 2025)

**TUI Dashboard Release**

A major release introducing a real-time terminal dashboard for monitoring and managing Context Bridge sessions.

### New Features

#### TUI Dashboard
- **Ink-based terminal UI** - Beautiful, responsive terminal interface built with React for terminals
- **Event-driven updates** - Real-time updates as sessions and queries execute
- **4 integrated screens:**
  - **Dashboard** - Overview of active projects, sessions, and system status
  - **Sessions** - Detailed session management and history
  - **Queries** - Query execution history and performance metrics
  - **Settings** - Configuration management and preferences
- **Zustand state management** - Lightweight, performant state handling
- **Keyboard navigation** - Full keyboard shortcuts for efficient navigation

#### Improvements
- Enhanced event system for real-time synchronization
- Improved storage layer performance
- Better error handling and user feedback
- Updated documentation with TUI guide

### Technical Details
- 106/118 tasks completed (90% of Spec 004)
- New dependencies: Ink, Zustand
- Refactored event emitter for typed events

---

## v0.1.0 (January 2025)

**Data Storage & Agent Abstraction Release**

A foundational release introducing persistent storage and pluggable agent architecture.

### New Features

#### Data Storage Layer
- **SQLite database** - Embedded database using better-sqlite3
- **Drizzle ORM** - Type-safe schema definitions and queries
- **Session management** - Track and persist AI sessions
- **Query caching** - Intelligent caching with TTL support
- **Event system** - Storage events for real-time updates

#### Agent Abstraction
- **AgentAdapter interface** - Pluggable architecture for AI providers
- **Claude adapter** - Production-ready Claude Code integration
- **Agent registry** - Automatic discovery and validation
- **npm installation** - Simple global installation with `npm install -g context-bridge`
- **Migration utilities** - Smooth upgrade from legacy configurations

#### Configuration
- **New config format** - Cleaner YAML structure for projects and repos
- **Environment overrides** - Override settings via environment variables
- **Validation** - Zod-powered configuration validation

### Technical Details
- 106/120 tasks completed (88% of Spec 002)
- 51/76 tasks completed (67% of Spec 001)
- New storage schema with migrations
- Comprehensive test coverage

---

## v0.0.1 (January 2025)

**Initial Release**

The first public release of Context Bridge, establishing core multi-repo coordination capabilities.

### Features

#### Multi-Repository Support
- **Project abstraction** - Group related repositories
- **Repository management** - Add, remove, and configure repos
- **Context switching** - Seamless navigation between repos

#### Query Tools
- **query_repo** - Query a specific repository using natural language
- **query_all_repos** - Query all repositories in parallel
- **get_repo_structure** - Get high-level repository overview

#### Implementation Tools
- **implement_in_repo** - Spawn sub-agent to make changes
- **run_in_repo** - Execute commands in repository context

#### MCP Integration
- **Full MCP protocol support** - Compatible with Claude and other MCP clients
- **Tool registration** - Automatic tool discovery and registration
- **JSON-RPC communication** - Standard MCP protocol implementation

### Technical Details
- Core CLI commands implemented
- MCP server with 9 tools
- Basic test coverage
- Initial documentation

---

## Upgrade Guide

### Upgrading to v0.4.0

1. **Update the package:**
   ```bash
   npm update -g context-bridge
   ```

2. **Verify installation:**
   ```bash
   context-bridge doctor
   ```

3. **Launch TUI dashboard:**
   ```bash
   context-bridge tui
   ```

No configuration changes required. The TUI dashboard is a new feature that works with existing configurations.

### Upgrading to v0.1.0

1. **Update the package:**
   ```bash
   npm update -g context-bridge
   ```

2. **Migrate configuration (if needed):**
   ```bash
   context-bridge migrate
   ```

3. **Initialize storage:**
   ```bash
   context-bridge storage init
   ```

The storage layer is automatically initialized on first use if not explicitly created.

### Upgrading from Legacy Versions

If upgrading from pre-0.0.1 development versions:

1. **Backup your configuration:**
   ```bash
   cp ~/.context-bridge/config.yaml ~/.context-bridge/config.yaml.backup
   ```

2. **Clean install:**
   ```bash
   npm uninstall -g context-bridge
   npm install -g context-bridge
   ```

3. **Re-initialize:**
   ```bash
   context-bridge init
   ```

4. **Import existing projects:**
   ```bash
   context-bridge project add <name> --repos <repo-paths>
   ```

---

## Deprecation Notices

### v0.1.0 Deprecations

- **Legacy config format** - The old configuration format is deprecated. Use `context-bridge migrate` to upgrade.
- **Direct Claude CLI calls** - Use the agent abstraction layer instead of direct CLI calls.

---

## See Also

- [Roadmap](./roadmap) - Future development plans
- [Getting Started](./getting-started) - Installation and setup
- [Architecture](./architecture) - System design and patterns
