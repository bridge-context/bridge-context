---
layout: default
title: Getting Started
nav_order: 1
has_children: true
---

# Getting Started

Welcome to Context Bridge! This section will help you install, configure, and start using Context Bridge with your AI assistant.

---

## Overview

Context Bridge is an MCP (Model Context Protocol) server that enables AI assistants to work seamlessly across multiple repositories. It spawns intelligent sub-agents to query, analyze, and implement changes across your entire codebase.

---

## In This Section

### [Installation](./installation.md)
Complete installation guide including:
- Prerequisites (Node.js, AI CLI tools)
- Installing from source
- Global CLI setup
- MCP client configuration

### [Quick Start](./quick-start.md)
Get up and running in 5 minutes:
- Initialize configuration
- Create your first project
- Add repositories
- Start using with Claude

### [Configuration](./configuration.md)
Detailed configuration reference:
- Project and repository setup
- Agent adapter options
- Storage configuration
- Environment variable overrides

---

## Quick Installation

```bash
# Clone and build
git clone https://github.com/bridge-context/context-bridge.git
cd context-bridge
npm install && npm run build

# Link globally
npm link

# Verify installation
context-bridge doctor

# Initialize and install
context-bridge init
context-bridge install
```

---

## Need Help?

- Check the [Configuration](./configuration.md) guide for troubleshooting
- Visit our [GitHub Issues](https://github.com/bridge-context/context-bridge/issues) for support
