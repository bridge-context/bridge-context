---
layout: default
title: Custom Adapters
parent: Guides
nav_order: 2
---

# Creating Custom Adapters

This guide explains how to create custom agent adapters for Context Bridge, allowing you to integrate any AI CLI tool.

---

## Quick Start

### 1. Copy the Template

Start with the adapter template:

```bash
mkdir -p ~/.context-bridge/adapters
curl -o ~/.context-bridge/adapters/my-adapter.ts \
  https://raw.githubusercontent.com/bridge-context/context-bridge/main/docs/examples/custom-adapter.ts
```

### 2. Implement Required Methods

Edit the template to implement your adapter logic:

```typescript
// ~/.context-bridge/adapters/my-adapter.ts
import { BaseAdapter, AgentExecutionOptions, AgentResult } from 'context-bridge';

export class MyAdapter extends BaseAdapter {
  readonly id = 'my-adapter';
  readonly name = 'My Custom Adapter';

  async execute(options: AgentExecutionOptions): Promise<AgentResult> {
    // Your implementation here
  }
}
```

### 3. Configure Context Bridge

Update your configuration to use the new adapter:

```yaml
# ~/.context-bridge/app-config.yaml
agent:
  type: my-adapter
  timeout: 300000
  options:
    # Your adapter-specific options
```

---

## Required Interface

All adapters must implement the `AgentAdapter` interface:

```typescript
interface AgentAdapter {
  /**
   * Unique identifier for this adapter (used in configuration)
   */
  readonly id: string;

  /**
   * Human-readable name (used in UI and logs)
   */
  readonly name: string;

  /**
   * Execute a task in a repository
   */
  execute(options: AgentExecutionOptions): Promise<AgentResult>;

  /**
   * Validate the CLI tool is installed and accessible
   */
  validateInstallation(): Promise<{ valid: boolean; error?: string }>;

  /**
   * Parse output from the CLI tool into structured result
   */
  parseOutput(stdout: string, stderr: string, exitCode: number): AgentResult;

  /**
   * Build CLI command and arguments
   */
  buildCommand(options: AgentExecutionOptions): { command: string; args: string[] };

  /**
   * Map Context Bridge tools to adapter-specific tool names
   */
  mapTools(tools: ToolRestriction): string[];
}
```

### AgentExecutionOptions

```typescript
interface AgentExecutionOptions {
  prompt: string;              // Task or query to execute
  workingDirectory: string;    // Repository path
  context?: string;            // Additional context
  allowedTools: ToolRestriction; // 'read-only' | 'write'
  timeout: number;             // Timeout in milliseconds
  model?: string;              // Model identifier (optional)
  maxTokens?: number;          // Max response tokens (optional)
  sessionId: string;           // Session ID for logging
}
```

### AgentResult

```typescript
interface AgentResult {
  success: boolean;            // Whether execution succeeded
  output: string;              // Response from the agent
  duration: number;            // Execution time in ms
  error?: string;              // Error message if failed
  exitCode: number;            // Process exit code
  modifiedFiles?: string[];    // Files changed (write operations)
}
```

---

## BaseAdapter Class

Extend `BaseAdapter` to get helpful utilities:

```typescript
import { BaseAdapter } from 'context-bridge';

export class MyAdapter extends BaseAdapter {
  constructor() {
    super();
  }

  // Inherited utilities:
  // - this.log(message) - Logging with adapter prefix
  // - this.getCommandName() - Get CLI command name
}
```

### Inherited Methods

| Method | Description |
|--------|-------------|
| `log(message: string)` | Log with adapter prefix: `[MyAdapter] message` |
| `getCommandName()` | Returns the CLI command name from configuration |

---

## Complete Example Implementation

Here's a complete adapter for a hypothetical "superai" CLI tool:

```typescript
// ~/.context-bridge/adapters/superai-adapter.ts
import { execa } from 'execa';
import {
  BaseAdapter,
  AgentAdapter,
  AgentExecutionOptions,
  AgentResult,
  ToolRestriction,
} from 'context-bridge';

export class SuperAIAdapter extends BaseAdapter implements AgentAdapter {
  readonly id = 'superai';
  readonly name = 'SuperAI';

  /**
   * Execute a task using the SuperAI CLI
   */
  async execute(options: AgentExecutionOptions): Promise<AgentResult> {
    const startTime = Date.now();

    // Validate installation first
    const validation = await this.validateInstallation();
    if (!validation.valid) {
      return {
        success: false,
        output: '',
        error: validation.error,
        exitCode: 1,
        duration: Date.now() - startTime,
      };
    }

    // Build command
    const { command, args } = this.buildCommand(options);

    try {
      // Execute the CLI
      const result = await execa(command, args, {
        cwd: options.workingDirectory,
        timeout: options.timeout,
        env: {
          ...process.env,
          SUPERAI_SESSION: options.sessionId,
        },
        reject: false, // Don't throw on non-zero exit
      });

      return this.parseOutput(result.stdout, result.stderr, result.exitCode);
    } catch (error) {
      return {
        success: false,
        output: '',
        error: error instanceof Error ? error.message : 'Unknown error',
        exitCode: 1,
        duration: Date.now() - startTime,
      };
    }
  }

  /**
   * Validate SuperAI CLI is installed
   */
  async validateInstallation(): Promise<{ valid: boolean; error?: string }> {
    try {
      const result = await execa('superai', ['--version'], { reject: false });
      if (result.exitCode !== 0) {
        return {
          valid: false,
          error: 'SuperAI CLI not found. Install with: npm install -g superai-cli',
        };
      }
      this.log(`Found SuperAI version: ${result.stdout.trim()}`);
      return { valid: true };
    } catch {
      return {
        valid: false,
        error: 'SuperAI CLI not found. Install with: npm install -g superai-cli',
      };
    }
  }

  /**
   * Parse CLI output into structured result
   */
  parseOutput(stdout: string, stderr: string, exitCode: number): AgentResult {
    const success = exitCode === 0;

    // Extract modified files from output (adapter-specific parsing)
    const modifiedFiles = this.extractModifiedFiles(stdout);

    return {
      success,
      output: stdout,
      error: success ? undefined : stderr || 'Command failed',
      exitCode,
      duration: 0, // Will be set by execute()
      modifiedFiles,
    };
  }

  /**
   * Build CLI command and arguments
   */
  buildCommand(options: AgentExecutionOptions): { command: string; args: string[] } {
    const args: string[] = [
      'run',
      '--non-interactive',
      '--output-format', 'text',
    ];

    // Add model if specified
    if (options.model) {
      args.push('--model', options.model);
    }

    // Add max tokens if specified
    if (options.maxTokens) {
      args.push('--max-tokens', String(options.maxTokens));
    }

    // Add tool restrictions
    const tools = this.mapTools(options.allowedTools);
    args.push('--tools', tools.join(','));

    // Add context if provided
    if (options.context) {
      args.push('--context', options.context);
    }

    // Add the prompt
    args.push('--prompt', options.prompt);

    return { command: 'superai', args };
  }

  /**
   * Map Context Bridge tools to SuperAI tool names
   */
  mapTools(restriction: ToolRestriction): string[] {
    // SuperAI uses different tool names than Context Bridge
    const readOnlyTools = [
      'file_read',      // Maps to Read
      'file_search',    // Maps to Glob
      'content_search', // Maps to Grep
      'shell_read',     // Maps to Bash (read-only mode)
    ];

    const writeTools = [
      ...readOnlyTools,
      'file_write',     // Maps to Write
      'file_edit',      // Maps to Edit
      'file_multi',     // Maps to MultiEdit
      'shell_full',     // Maps to Bash (full mode)
    ];

    return restriction === 'read-only' ? readOnlyTools : writeTools;
  }

  /**
   * Extract modified files from CLI output
   */
  private extractModifiedFiles(output: string): string[] {
    const files: string[] = [];

    // Parse SuperAI's output format for modified files
    // Format: "Modified: path/to/file.ts"
    const regex = /Modified:\s+(.+)$/gm;
    let match;

    while ((match = regex.exec(output)) !== null) {
      files.push(match[1].trim());
    }

    return files;
  }
}

// Export for Context Bridge to discover
export default SuperAIAdapter;
```

---

## Tool Mapping Reference

Context Bridge uses these standard tools that must be mapped to your CLI:

| Context Bridge Tool | Purpose | Typical CLI Equivalent |
|---------------------|---------|------------------------|
| `Read` | Read file contents | `file_read`, `cat`, `read` |
| `Write` | Create/overwrite files | `file_write`, `write` |
| `Edit` | Modify existing files | `file_edit`, `patch`, `edit` |
| `Glob` | Find files by pattern | `file_search`, `find`, `glob` |
| `Grep` | Search file contents | `content_search`, `search`, `grep` |
| `Bash` | Execute shell commands | `shell`, `exec`, `run` |
| `MultiEdit` | Multiple file edits | `multi_edit`, `batch_edit` |

### Tool Restrictions

When `allowedTools` is `'read-only'`:
- Only Read, Glob, Grep, and Bash (with restrictions) are available
- Bash should not allow file modifications

When `allowedTools` is `'write'`:
- All tools are available
- Bash has full access

---

## Publishing to npm

Share your adapter with the community by publishing to npm.

### Package Naming Convention

Use the prefix `agent-context-bridge-` for discoverability:

```
agent-context-bridge-superai
agent-context-bridge-gemini
agent-context-bridge-llama
```

### Package Structure

```
agent-context-bridge-superai/
  ├── package.json
  ├── README.md
  ├── src/
  │   └── index.ts
  ├── dist/
  │   └── index.js
  └── tsconfig.json
```

### package.json

```json
{
  "name": "agent-context-bridge-superai",
  "version": "1.0.0",
  "description": "SuperAI adapter for Context Bridge",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "keywords": [
    "context-bridge",
    "context-bridge-adapter",
    "superai",
    "ai",
    "agent"
  ],
  "peerDependencies": {
    "context-bridge": "^0.4.0"
  },
  "dependencies": {
    "execa": "^8.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0"
  }
}
```

### Publishing

```bash
# Build your adapter
npm run build

# Publish to npm
npm publish
```

### Using Published Adapters

Users can install and use your adapter:

```bash
npm install -g agent-context-bridge-superai
```

```yaml
# ~/.context-bridge/app-config.yaml
agent:
  type: superai
  timeout: 300000
  options:
    model: superai-v2
```

---

## Testing and Debugging

### Testing Your Adapter

Create a test file to verify your adapter works:

```typescript
// test-adapter.ts
import { SuperAIAdapter } from './superai-adapter';

async function test() {
  const adapter = new SuperAIAdapter();

  // Test installation check
  const validation = await adapter.validateInstallation();
  console.log('Installation valid:', validation.valid);

  // Test command building
  const { command, args } = adapter.buildCommand({
    prompt: 'List all TypeScript files',
    workingDirectory: '/tmp/test-repo',
    allowedTools: 'read-only',
    timeout: 30000,
    sessionId: 'test-123',
  });
  console.log('Command:', command, args.join(' '));

  // Test tool mapping
  const readTools = adapter.mapTools('read-only');
  const writeTools = adapter.mapTools('write');
  console.log('Read-only tools:', readTools);
  console.log('Write tools:', writeTools);
}

test().catch(console.error);
```

Run the test:

```bash
npx ts-node test-adapter.ts
```

### Debug Mode

Enable verbose logging to debug adapter issues:

```bash
CONTEXT_BRIDGE_VERBOSE=1 context-bridge test-query \
  --repo api \
  --query "List all files"
```

### Common Issues

**Adapter not loading:**
```bash
# Check adapter file is in correct location
ls -la ~/.context-bridge/adapters/

# Verify TypeScript compilation
npx tsc --noEmit ~/.context-bridge/adapters/my-adapter.ts
```

**CLI not found:**
```bash
# Check CLI is installed and in PATH
which superai

# Test CLI directly
superai --version
```

**Timeout issues:**
```yaml
# Increase timeout in configuration
agent:
  timeout: 600000  # 10 minutes
```

**Tool mapping errors:**
```bash
# Enable verbose mode to see tool arguments
CONTEXT_BRIDGE_VERBOSE=1 context-bridge test-query \
  --repo api \
  --query "Read the README file"
```

### Logging Best Practices

Use the inherited `log()` method for consistent output:

```typescript
this.log('Starting execution...');
this.log(`Using model: ${options.model}`);
this.log(`Allowed tools: ${tools.join(', ')}`);
```

Output:
```
[SuperAI] Starting execution...
[SuperAI] Using model: superai-v2
[SuperAI] Allowed tools: file_read, file_search, content_search
```

---

## See Also

- [Agent System Architecture](../architecture/agent-system) - Deep dive on agent internals
- [Configuration Reference](../getting-started/configuration) - Agent configuration options
- [Troubleshooting](./troubleshooting) - Common issues and solutions
