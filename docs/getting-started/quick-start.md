---
layout: default
title: Quick Start
parent: Getting Started
nav_order: 2
---

# Quick Start

Get up and running with Context Bridge in 5 minutes.

---

## 1. Initialize Configuration

Start by initializing your Context Bridge configuration:

```bash
context-bridge init
```

This creates the configuration directory and default files:

```
~/.context-bridge/
├── config.yaml       # Projects and repositories
├── app-config.yaml   # Agent and storage settings
└── storage.db        # SQLite database
```

---

## 2. Create a Project

A project groups related repositories together. Create your first project:

```bash
context-bridge project create my-app "My full-stack application"
```

**Example:**

```bash
context-bridge project create ecommerce "E-commerce platform with API and frontend"
```

---

## 3. Add Repositories

Add repositories to your project:

```bash
context-bridge repo add <name> <path>
```

**Example - adding multiple repos:**

```bash
# Add backend API repository
context-bridge repo add api ~/projects/ecommerce-api

# Add frontend application
context-bridge repo add frontend ~/projects/ecommerce-web

# Add shared library
context-bridge repo add shared ~/projects/ecommerce-shared
```

**Verify your setup:**

```bash
context-bridge project list
```

Output:

```
Projects:
─────────
ecommerce
  Description: E-commerce platform with API and frontend
  Repositories:
    - api: ~/projects/ecommerce-api
    - frontend: ~/projects/ecommerce-web
    - shared: ~/projects/ecommerce-shared
```

---

## 4. Install in Claude Desktop

Install the MCP server in Claude Desktop:

```bash
context-bridge install
```

**Restart Claude Desktop** to load the new MCP server.

---

## 5. Example Usage with Claude

Once installed, you can use Context Bridge directly in your Claude conversations.

### List Your Projects

Ask Claude:

> "What projects do I have configured in Context Bridge?"

Claude will use the `list_projects` tool to show your available projects.

### Query a Repository

> "In my ecommerce project, how does the authentication system work in the API repo?"

Claude spawns a sub-agent to analyze the `api` repository and returns a detailed explanation.

### Query Across All Repositories

> "Search all repos in ecommerce for how user data is validated"

Claude queries all repositories in parallel, synthesizing findings from `api`, `frontend`, and `shared`.

### Implement Changes

> "Add email validation to the user registration in the API repo"

Claude spawns an implementation agent that makes the changes directly in your codebase.

### Get Repository Structure

> "Show me the structure of the frontend repo"

Claude returns a high-level overview of the repository's architecture and key files.

---

## Common Workflows

### Cross-Repository Refactoring

> "Rename the User model to Account across all repos in the ecommerce project"

### Understanding Dependencies

> "How does the frontend consume the API? Show me the integration points"

### Code Review Assistance

> "Review the recent changes in the api repo for security issues"

### Documentation Generation

> "Generate API documentation for all endpoints in the api repo"

---

## Next Steps

- [Configuration Reference](./configuration) - Customize agent behavior and storage
- [Usage Guide](../usage) - Advanced workflows and best practices
- [API Reference](../api) - Complete MCP tools documentation
