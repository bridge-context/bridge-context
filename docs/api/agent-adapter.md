---
layout: default
title: AgentAdapter
parent: API Reference
nav_order: 1
---

# AgentAdapter Interface

The `AgentAdapter` interface defines the contract for AI agent implementations in Context Bridge. All agent adapters (Claude, Codex, Aider, custom) must implement this interface.

---

## Interface Definition

```typescript
interface AgentAdapter {
  /**
   * Unique identifier for this adapter (used in configuration)
   */
  readonly name: string;

  /**
   * Semantic version of the adapter
   */
  readonly version: string;

  /**
   * Execute a task in the given repository
   */
  execute(options: AgentExecutionOptions): Promise<AgentResult>;

  /**
   * Validate the CLI tool is installed and accessible
   */
  validateInstallation(): Promise<InstallationStatus>;

  /**
   * Parse raw CLI output into structured format
   */
  parseOutput(output: string): string;

  /**
   * Build CLI command arguments from execution options
   */
  buildCommand(options: AgentExecutionOptions): string[];

  /**
   * Map Context Bridge tools to adapter-specific tool names
   */
  mapTools(tools: string[]): string[];
}
```

---

## Properties

### name

```typescript
readonly name: string;
```

Unique identifier for the adapter. Used in configuration files to select the adapter.

**Examples:** `'claude'`, `'codex'`, `'aider'`, `'my-custom-adapter'`

**Requirements:**
- Must be unique across all registered adapters
- Should be lowercase with hyphens (kebab-case)
- Used as the `agent.type` value in configuration

### version

```typescript
readonly version: string;
```

Semantic version string for the adapter implementation.

**Examples:** `'1.0.0'`, `'2.1.3'`, `'0.4.0-beta.1'`

**Requirements:**
- Should follow [Semantic Versioning](https://semver.org/)
- Displayed in logs and diagnostics
- Used for compatibility checking

---

## Methods

### execute()

```typescript
execute(options: AgentExecutionOptions): Promise<AgentResult>;
```

Execute a task in the specified repository using the underlying AI CLI tool.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `options` | `AgentExecutionOptions` | Execution configuration (see below) |

**Returns:** `Promise<AgentResult>` - Structured execution result

**Throws:**
- `AgentTimeoutError` - When execution exceeds timeout
- `AgentNotFoundError` - When CLI tool is not installed
- `AgentError` - For other execution failures

**Example:**

```typescript
const result = await adapter.execute({
  prompt: 'Find all TODO comments in the codebase',
  workingDirectory: '/path/to/repo',
  allowedTools: 'read-only',
  timeout: 300000,
  sessionId: 'session-uuid',
});

if (result.success) {
  console.log('Output:', result.output);
} else {
  console.error('Error:', result.error);
}
```

---

### validateInstallation()

```typescript
validateInstallation(): Promise<InstallationStatus>;
```

Check if the underlying CLI tool is installed and accessible.

**Returns:** `Promise<InstallationStatus>` - Installation validation result

**Example:**

```typescript
const status = await adapter.validateInstallation();

if (status.installed) {
  console.log(`Found ${adapter.name} version ${status.version}`);
} else {
  console.error(`Installation error: ${status.error}`);
  console.log(`Install with: ${status.installCommand}`);
}
```

---

### parseOutput()

```typescript
parseOutput(output: string): string;
```

Parse raw CLI output into a cleaned, structured format.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `output` | `string` | Raw stdout from CLI process |

**Returns:** `string` - Cleaned and formatted output

**Responsibilities:**
- Remove ANSI escape codes
- Extract relevant content from verbose output
- Normalize line endings
- Handle adapter-specific output formats

**Example:**

```typescript
const rawOutput = '\x1b[32m[INFO]\x1b[0m Task completed\nResult: Success';
const cleaned = adapter.parseOutput(rawOutput);
// Returns: "Task completed\nResult: Success"
```

---

### buildCommand()

```typescript
buildCommand(options: AgentExecutionOptions): string[];
```

Build CLI command arguments from execution options.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `options` | `AgentExecutionOptions` | Execution configuration |

**Returns:** `string[]` - Array of CLI arguments

**Example:**

```typescript
const args = adapter.buildCommand({
  prompt: 'List all TypeScript files',
  workingDirectory: '/path/to/repo',
  allowedTools: 'read-only',
  timeout: 300000,
  model: 'claude-sonnet-4-20250514',
  sessionId: 'session-uuid',
});

// Returns for Claude adapter:
// ['--print', '--output-format', 'text', '--model', 'claude-sonnet-4-20250514',
//  '--allowedTools', 'Read,Glob,Grep,Bash', 'List all TypeScript files']
```

---

### mapTools()

```typescript
mapTools(tools: string[]): string[];
```

Map Context Bridge standard tool names to adapter-specific tool names.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `tools` | `string[]` | Context Bridge tool names |

**Returns:** `string[]` - Adapter-specific tool names

**Standard Tools:**

| Context Bridge | Purpose |
|----------------|---------|
| `Read` | Read file contents |
| `Write` | Create/overwrite files |
| `Edit` | Modify existing files |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents |
| `Bash` | Execute shell commands |
| `MultiEdit` | Multiple file edits |

**Example:**

```typescript
// For an adapter using different tool names
const mapped = adapter.mapTools(['Read', 'Glob', 'Grep']);
// Returns: ['file_read', 'file_search', 'content_search']
```

---

## Related Types

### AgentExecutionOptions

Configuration passed to the `execute()` method:

```typescript
interface AgentExecutionOptions {
  /**
   * The task or query to execute
   */
  prompt: string;

  /**
   * Working directory (repository path)
   */
  workingDirectory: string;

  /**
   * Additional context to inject
   */
  context?: string;

  /**
   * Tool restriction level
   */
  allowedTools: 'read-only' | 'write';

  /**
   * Execution timeout in milliseconds
   */
  timeout: number;

  /**
   * Model identifier (adapter-specific)
   */
  model?: string;

  /**
   * Maximum tokens for response
   */
  maxTokens?: number;

  /**
   * Session ID for logging and tracking
   */
  sessionId: string;
}
```

### AgentResult

Result returned from the `execute()` method:

```typescript
interface AgentResult {
  /**
   * Whether execution completed successfully
   */
  success: boolean;

  /**
   * The agent's response output
   */
  output: string;

  /**
   * Execution duration in milliseconds
   */
  duration: number;

  /**
   * Error message if execution failed
   */
  error?: string;

  /**
   * Process exit code
   */
  exitCode: number;

  /**
   * Files modified during execution (write operations only)
   */
  modifiedFiles?: string[];
}
```

### InstallationStatus

Result from the `validateInstallation()` method:

```typescript
interface InstallationStatus {
  /**
   * Whether the CLI tool is installed
   */
  installed: boolean;

  /**
   * Version string if installed
   */
  version?: string;

  /**
   * Error message if not installed
   */
  error?: string;

  /**
   * Command to install the CLI tool
   */
  installCommand?: string;
}
```

---

## Zod Validation Schema

Context Bridge validates all inputs using Zod schemas:

```typescript
import { z } from 'zod';

export const AgentExecutionOptionsSchema = z.object({
  prompt: z.string().min(1, 'Prompt is required'),
  workingDirectory: z.string().min(1, 'Working directory is required'),
  context: z.string().optional(),
  allowedTools: z.enum(['read-only', 'write']),
  timeout: z.number().positive('Timeout must be positive'),
  model: z.string().optional(),
  maxTokens: z.number().positive().optional(),
  sessionId: z.string().uuid('Session ID must be a valid UUID'),
});

export const AgentResultSchema = z.object({
  success: z.boolean(),
  output: z.string(),
  duration: z.number().nonnegative(),
  error: z.string().optional(),
  exitCode: z.number().int(),
  modifiedFiles: z.array(z.string()).optional(),
});

export const InstallationStatusSchema = z.object({
  installed: z.boolean(),
  version: z.string().optional(),
  error: z.string().optional(),
  installCommand: z.string().optional(),
});
```

---

## Example Implementation

Complete example of a custom adapter implementation:

```typescript
import { execa } from 'execa';
import {
  BaseAdapter,
  AgentAdapter,
  AgentExecutionOptions,
  AgentResult,
  InstallationStatus,
} from '@agent-context-bridge/cli';

export class CustomAdapter extends BaseAdapter implements AgentAdapter {
  readonly name = 'custom-ai';
  readonly version = '1.0.0';

  async execute(options: AgentExecutionOptions): Promise<AgentResult> {
    const startTime = Date.now();

    // Validate installation
    const status = await this.validateInstallation();
    if (!status.installed) {
      return {
        success: false,
        output: '',
        error: status.error,
        exitCode: 1,
        duration: Date.now() - startTime,
      };
    }

    // Build and execute command
    const args = this.buildCommand(options);

    try {
      const result = await execa('custom-ai', args, {
        cwd: options.workingDirectory,
        timeout: options.timeout,
        reject: false,
      });

      return {
        success: result.exitCode === 0,
        output: this.parseOutput(result.stdout),
        error: result.exitCode !== 0 ? result.stderr : undefined,
        exitCode: result.exitCode ?? 1,
        duration: Date.now() - startTime,
      };
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

  async validateInstallation(): Promise<InstallationStatus> {
    try {
      const result = await execa('custom-ai', ['--version'], { reject: false });
      if (result.exitCode === 0) {
        return {
          installed: true,
          version: result.stdout.trim(),
        };
      }
      return {
        installed: false,
        error: 'custom-ai CLI not found',
        installCommand: 'npm install -g custom-ai-cli',
      };
    } catch {
      return {
        installed: false,
        error: 'custom-ai CLI not found',
        installCommand: 'npm install -g custom-ai-cli',
      };
    }
  }

  parseOutput(output: string): string {
    // Remove ANSI codes and clean output
    return output.replace(/\x1b\[[0-9;]*m/g, '').trim();
  }

  buildCommand(options: AgentExecutionOptions): string[] {
    const args = ['run', '--non-interactive'];

    if (options.model) {
      args.push('--model', options.model);
    }

    const tools = this.mapTools(
      options.allowedTools === 'read-only'
        ? ['Read', 'Glob', 'Grep', 'Bash']
        : ['Read', 'Glob', 'Grep', 'Bash', 'Write', 'Edit', 'MultiEdit']
    );
    args.push('--tools', tools.join(','));

    args.push('--prompt', options.prompt);

    return args;
  }

  mapTools(tools: string[]): string[] {
    const toolMap: Record<string, string> = {
      'Read': 'file_read',
      'Write': 'file_write',
      'Edit': 'file_edit',
      'Glob': 'file_glob',
      'Grep': 'file_grep',
      'Bash': 'shell_exec',
      'MultiEdit': 'file_multi_edit',
    };

    return tools.map(tool => toolMap[tool] || tool);
  }
}

export default CustomAdapter;
```

---

## See Also

- [StorageAdapter](./storage-adapter) - Storage adapter interface
- [Custom Adapters Guide](../guides/custom-adapters) - Step-by-step adapter creation
- [Agent System Architecture](../architecture/agent-system) - Agent system internals
