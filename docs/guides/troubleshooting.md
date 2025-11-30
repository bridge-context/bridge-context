---
layout: default
title: Troubleshooting
parent: Guides
nav_order: 3
---

# Troubleshooting Guide

Diagnose and fix common Context Bridge issues.

---

## Quick Diagnostics

Run the built-in health check to identify issues:

```bash
context-bridge doctor
```

Example output with issues:

```
Context Bridge Health Check
===========================

  System Requirements
    ✓ Node.js version: v20.10.0 (required: >= 18)

  Configuration
    ✓ config.yaml: Valid
    ✗ app-config.yaml: Parse error at line 12
      Error: Invalid YAML syntax

  Storage
    ✓ SQLite database: Connected

  Agent
    ⚠ CLI installed: claude-code not found
      Suggestion: Run "npm install -g @anthropic-ai/claude-code"

  Projects
    ⚠ 1 repository has invalid path
      - ecommerce/shared: Path does not exist

Summary:
  1 error, 2 warnings

  Fix the errors above and run "context-bridge doctor" again.
```

---

## Common Issues

### MCP Not Connecting

**Symptoms:**
- Claude Desktop doesn't show Context Bridge tools
- "MCP server not responding" errors
- Tools list is empty in Claude

**Solutions:**

1. **Verify installation:**
   ```bash
   context-bridge install
   ```

2. **Restart Claude Desktop** after installation

3. **Check MCP configuration:**
   ```bash
   # macOS
   cat ~/Library/Application\ Support/Claude/claude_desktop_config.json

   # Windows
   type %APPDATA%\Claude\claude_desktop_config.json
   ```

   Should contain:
   ```json
   {
     "mcpServers": {
       "context-bridge": {
         "command": "context-bridge",
         "args": ["serve"]
       }
     }
   }
   ```

4. **Test server manually:**
   ```bash
   context-bridge serve
   ```

   If errors appear, address them before retrying.

5. **Check Context Bridge is in PATH:**
   ```bash
   which context-bridge
   ```

---

### Agent Not Found

**Symptoms:**
- "Agent 'claude' not found" error
- "CLI tool not installed" warnings
- Sub-agent execution fails

**Solutions:**

1. **Verify CLI tool is installed:**
   ```bash
   # For Claude
   claude --version

   # For Codex
   codex --version

   # For Aider
   aider --version
   ```

2. **Install the CLI tool:**
   ```bash
   # Claude Code
   npm install -g @anthropic-ai/claude-code

   # OpenAI Codex
   npm install -g @openai/codex

   # Aider
   pip install aider-chat
   ```

3. **Check adapter configuration:**
   ```bash
   context-bridge config show
   ```

   Verify `agent.type` matches an installed CLI.

4. **Verify PATH includes the CLI:**
   ```bash
   echo $PATH
   ```

   The directory containing the CLI must be in PATH.

---

### Repository Not Found

**Symptoms:**
- "Repository 'xyz' not found in project" error
- Queries fail with "repo not found"
- Project shows 0 repositories

**Solutions:**

1. **List current repositories:**
   ```bash
   context-bridge repo list
   ```

2. **Check if you're in the right project:**
   ```bash
   context-bridge project list
   ```

3. **Switch to the correct project:**
   ```bash
   context-bridge project switch my-project
   ```

4. **Re-add the repository if missing:**
   ```bash
   context-bridge repo add my-repo /path/to/repo
   ```

5. **Verify repository path exists:**
   ```bash
   ls -la /path/to/repo
   ```

---

### Query Timeouts

**Symptoms:**
- "Query timed out after 300000ms" error
- Sub-agent doesn't respond
- Long-running queries fail

**Solutions:**

1. **Increase timeout:**
   ```bash
   # Via environment variable
   export CONTEXT_BRIDGE_AGENT_TIMEOUT=600000  # 10 minutes

   # Via configuration
   context-bridge config set agent.timeout 600000
   ```

2. **Simplify the query:**
   - Break complex queries into smaller parts
   - Ask about specific files rather than entire codebase
   - Use `get_repo_structure` first to narrow focus

3. **Check repository size:**
   Large repositories take longer. Consider:
   - Querying specific directories
   - Using more targeted searches
   - Splitting into smaller repos

4. **Check system resources:**
   ```bash
   # CPU and memory
   top

   # Disk space
   df -h
   ```

---

### Storage Issues

**Symptoms:**
- "Database locked" errors
- "Cannot write to storage" errors
- Cache not working

**Solutions:**

1. **Check database file:**
   ```bash
   ls -la ~/.context-bridge/storage.db
   ```

2. **Verify permissions:**
   ```bash
   # Should be readable and writable
   test -r ~/.context-bridge/storage.db && echo "Readable" || echo "Not readable"
   test -w ~/.context-bridge/storage.db && echo "Writable" || echo "Not writable"
   ```

3. **Fix permissions:**
   ```bash
   chmod 644 ~/.context-bridge/storage.db
   ```

4. **Clear and rebuild storage:**
   ```bash
   context-bridge storage clear --all
   context-bridge init
   ```

5. **Check disk space:**
   ```bash
   df -h ~/.context-bridge/
   ```

6. **Use a different storage path:**
   ```bash
   context-bridge config set storage.path /new/path/storage.db
   ```

---

### Configuration Errors

**Symptoms:**
- "Invalid YAML syntax" errors
- "Configuration file not found" errors
- Settings not being applied

**Solutions:**

1. **Validate configuration:**
   ```bash
   context-bridge doctor
   ```

2. **Check YAML syntax:**
   ```bash
   # Install yaml lint
   npm install -g yaml-lint

   # Validate files
   yamllint ~/.context-bridge/config.yaml
   yamllint ~/.context-bridge/app-config.yaml
   ```

3. **Common YAML issues:**
   - Incorrect indentation (use 2 spaces, not tabs)
   - Missing colons after keys
   - Unquoted special characters

4. **Reset to defaults:**
   ```bash
   context-bridge init --force
   ```

5. **Edit configuration manually:**
   ```bash
   context-bridge config edit
   ```

6. **Backup before editing:**
   ```bash
   cp ~/.context-bridge/config.yaml ~/.context-bridge/config.yaml.bak
   ```

---

## Debug Mode

Enable verbose logging to diagnose issues:

```bash
CONTEXT_BRIDGE_VERBOSE=1 context-bridge serve
```

Or for specific commands:

```bash
CONTEXT_BRIDGE_VERBOSE=1 context-bridge test-query \
  --repo api \
  --query "How does auth work?"
```

### What Debug Mode Shows

- Full CLI arguments passed to sub-agents
- Timing information for each operation
- Raw output from sub-agents
- Cache hit/miss information
- Configuration values being used

### Logging Levels

| Level | Environment Variable | Description |
|-------|---------------------|-------------|
| Normal | (default) | Basic operation logging |
| Verbose | `CONTEXT_BRIDGE_VERBOSE=1` | Detailed operation logging |
| Debug | `CONTEXT_BRIDGE_DEBUG=1` | Full debug output |

---

## Log Locations

### Debug Log

```
~/.context-bridge/debug.log
```

Contains recent debug output even when verbose mode is off.

**View recent logs:**
```bash
tail -100 ~/.context-bridge/debug.log
```

**Search for errors:**
```bash
grep -i error ~/.context-bridge/debug.log
```

### Storage Database

```
~/.context-bridge/storage.db
```

SQLite database containing:
- Session history
- Query cache
- Repository metadata

**Inspect database:**
```bash
sqlite3 ~/.context-bridge/storage.db ".tables"
sqlite3 ~/.context-bridge/storage.db "SELECT * FROM sessions LIMIT 5;"
```

### MCP Server Logs

When running via Claude Desktop, logs appear in:

**macOS:**
```
~/Library/Logs/Claude/mcp-context-bridge.log
```

**Windows:**
```
%LOCALAPPDATA%\Claude\Logs\mcp-context-bridge.log
```

---

## FAQ

### Why is my query returning stale results?

Context Bridge caches query results for 1 hour. To get fresh results:

```bash
# Clear all cached queries
context-bridge storage clear --cache

# Clear cache for specific project
context-bridge storage clear --cache --project my-project
```

### How do I see what's in the cache?

```bash
context-bridge storage stats
```

Output:
```
Storage Statistics
==================
  Cache entries: 47
  Cache hit rate: 73%
  Cache size: 2.3 MB
  Oldest entry: 45 minutes ago
```

### Why is Context Bridge slow?

Common causes and fixes:

| Cause | Solution |
|-------|----------|
| Large repositories | Query specific directories |
| Complex queries | Break into smaller queries |
| Slow disk | Use SSD or tmpfs for storage |
| Network issues | Check API connectivity |
| CPU throttling | Close other heavy applications |

### How do I completely reset Context Bridge?

```bash
# Remove all data and configuration
rm -rf ~/.context-bridge

# Reinitialize
context-bridge init
```

### Can I use multiple AI backends?

Yes, but only one at a time. To switch:

```bash
context-bridge config set agent.type codex
```

Then restart the MCP server or Claude Desktop.

### How do I update Context Bridge?

```bash
npm update -g context-bridge
```

Then verify:
```bash
context-bridge --version
context-bridge doctor
```

### Where can I get help?

1. Check this troubleshooting guide
2. Run `context-bridge doctor` for diagnostics
3. Search [GitHub Issues](https://github.com/bridge-context/context-bridge/issues)
4. Create a new issue with debug logs

---

## Getting Help

When reporting issues, include:

1. **Context Bridge version:**
   ```bash
   context-bridge --version
   ```

2. **System information:**
   ```bash
   node --version
   uname -a
   ```

3. **Doctor output:**
   ```bash
   context-bridge doctor --json
   ```

4. **Debug logs:**
   ```bash
   tail -50 ~/.context-bridge/debug.log
   ```

5. **Steps to reproduce the issue**

---

## See Also

- [Quick Start](../getting-started/quick-start) - Initial setup
- [Configuration](../getting-started/configuration) - Configuration reference
- [CLI Reference](../cli) - Command documentation
