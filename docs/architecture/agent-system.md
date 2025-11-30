---
layout: default
title: Agent System
parent: Architecture
nav_order: 1
---

# Agent System

Deep dive into Context Bridge's agent system architecture, including the adapter interface, registry, factory, and executor components.

---

## Overview

The agent system manages AI sub-agents that perform operations in repositories. It follows the adapter pattern to support multiple AI providers (Claude, Codex, Aider) through a unified interface.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Agent System                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │   Registry   │ →  │   Factory    │ →  │     Executor         │  │
│  │              │    │              │    │                      │  │
│  │  • Discovery │    │  • Config    │    │  • Spawn process     │  │
│  │  • Loading   │    │  • Creation  │    │  • Capture output    │  │
│  │  • Validation│    │  • Injection │    │  • Handle timeout    │  │
│  └──────────────┘    └──────────────┘    └──────────────────────┘  │
│         │                   │                      │                │
│         ▼                   ▼                      ▼                │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                     Agent Adapters                             │ │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │ │
│  │  │ Claude  │    │  Codex  │    │  Aider  │    │ Custom  │    │ │
│  │  └─────────┘    └─────────┘    └─────────┘    └─────────┘    │ │
│  └────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## AgentAdapter Interface

All agent adapters implement this core interface:

```typescript
interface AgentAdapter {
  /**
   * Unique identifier for this adapter
   */
  readonly id: string;

  /**
   * Human-readable name
   */
  readonly name: string;

  /**
   * Execute a task in the given repository
   */
  execute(options: AgentExecutionOptions): Promise<AgentResult>;

  /**
   * Get adapter capabilities and supported features
   */
  getCapabilities(): AgentCapabilities;

  /**
   * Check if the adapter's CLI tool is available
   */
  isAvailable(): Promise<boolean>;

  /**
   * Get the version of the underlying CLI tool
   */
  getVersion(): Promise<string | null>;
}
```

---

## Core Types

### AgentExecutionOptions

Configuration passed to the adapter when executing a task:

```typescript
interface AgentExecutionOptions {
  /**
   * The task or query to execute
   */
  prompt: string;

  /**
   * Working directory for the sub-agent
   */
  workingDirectory: string;

  /**
   * Additional context to inject into the prompt
   */
  context?: string;

  /**
   * Tools available to the sub-agent
   */
  allowedTools: ToolRestriction;

  /**
   * Execution timeout in milliseconds
   */
  timeout: number;

  /**
   * Model to use (adapter-specific)
   */
  model?: string;

  /**
   * Maximum tokens for response
   */
  maxTokens?: number;

  /**
   * Session ID for logging
   */
  sessionId: string;
}
```

### AgentResult

Result returned after task execution:

```typescript
interface AgentResult {
  /**
   * Whether execution completed successfully
   */
  success: boolean;

  /**
   * The sub-agent's response output
   */
  output: string;

  /**
   * Execution time in milliseconds
   */
  duration: number;

  /**
   * Error message if execution failed
   */
  error?: string;

  /**
   * Exit code from the sub-agent process
   */
  exitCode: number;

  /**
   * Files modified during execution (implementation tools only)
   */
  modifiedFiles?: string[];
}
```

### AgentCapabilities

Describes what an adapter can do:

```typescript
interface AgentCapabilities {
  /**
   * Supports read-only query operations
   */
  supportsQuery: boolean;

  /**
   * Supports write/implementation operations
   */
  supportsImplementation: boolean;

  /**
   * Supports streaming output
   */
  supportsStreaming: boolean;

  /**
   * Maximum context window size
   */
  maxContextSize: number;

  /**
   * Available models for this adapter
   */
  availableModels: string[];
}
```

---

## Registry

The registry discovers and manages available agent adapters.

### Discovery

```typescript
class AgentRegistry {
  /**
   * Discover all available adapters from multiple sources
   */
  async discover(): Promise<void>;

  /**
   * Get an adapter by ID
   */
  get(id: string): AgentAdapter | undefined;

  /**
   * List all registered adapters
   */
  list(): AgentAdapter[];

  /**
   * Check if an adapter is available (CLI installed)
   */
  async isAvailable(id: string): Promise<boolean>;
}
```

### Adapter Sources

The registry loads adapters from multiple sources:

| Source | Location | Priority |
|--------|----------|----------|
| Built-in | `src/adapters/agent/` | 1 (lowest) |
| User plugins | `~/.context-bridge/adapters/` | 2 |
| Project plugins | `./.context-bridge/adapters/` | 3 (highest) |

### Loading Process

```
1. Scan built-in adapters directory
2. Scan user plugin directory
3. Scan project plugin directory
4. Validate each adapter implements interface
5. Check CLI availability
6. Register available adapters
```

---

## Factory

The factory creates configured adapter instances.

```typescript
class AgentFactory {
  /**
   * Create an adapter instance from configuration
   */
  create(config: AgentConfig): AgentAdapter;

  /**
   * Create the default adapter (Claude)
   */
  createDefault(): AgentAdapter;
}
```

### Configuration-Based Creation

```typescript
// Configuration
const config: AgentConfig = {
  type: 'claude',
  timeout: 300000,
  options: {
    model: 'claude-sonnet-4-20250514',
    maxTokens: 8192
  }
};

// Create adapter
const adapter = factory.create(config);
```

### Dependency Injection

The factory injects dependencies into adapters:

```typescript
// Factory injects:
// - Logger instance
// - Storage adapter reference
// - Event emitter
// - Configuration values
```

---

## Executor

The executor spawns sub-agent processes and manages their lifecycle.

### Process Spawning with Execa

```typescript
class AgentExecutor {
  /**
   * Execute a task using the configured adapter
   */
  async execute(options: AgentExecutionOptions): Promise<AgentResult> {
    const subprocess = execa(this.cliCommand, this.buildArgs(options), {
      cwd: options.workingDirectory,
      timeout: options.timeout,
      env: this.buildEnvironment(options),
      reject: false,  // Handle errors manually
    });

    // Capture output
    const { stdout, stderr, exitCode } = await subprocess;

    return this.parseResult(stdout, stderr, exitCode);
  }
}
```

### Execution Flow

```
1. Validate options and paths
2. Build CLI arguments with tool restrictions
3. Set up environment variables
4. Spawn sub-process with Execa
5. Monitor for timeout
6. Capture stdout/stderr
7. Parse and validate output
8. Log execution to storage
9. Return structured result
```

### Timeout Handling

```typescript
// Default timeout: 5 minutes (300,000ms)
// Configurable via:
// - AgentExecutionOptions.timeout
// - CONTEXT_BRIDGE_AGENT_TIMEOUT environment variable
// - agent.timeout in configuration

try {
  const result = await subprocess;
} catch (error) {
  if (error.timedOut) {
    return {
      success: false,
      error: `Agent timed out after ${timeout}ms`,
      exitCode: 124,  // Standard timeout exit code
    };
  }
}
```

---

## Built-in Claude Adapter

The default adapter uses Anthropic's Claude CLI.

### Implementation Overview

```typescript
class ClaudeAdapter implements AgentAdapter {
  readonly id = 'claude';
  readonly name = 'Claude';

  async execute(options: AgentExecutionOptions): Promise<AgentResult> {
    const args = [
      '--print',           // Output mode
      '--output-format', 'text',
      '--model', options.model ?? 'claude-sonnet-4-20250514',
      '--max-tokens', String(options.maxTokens ?? 8192),
      '--allowedTools', this.formatTools(options.allowedTools),
      options.prompt,
    ];

    // Spawn process
    return this.executor.run('claude', args, options);
  }
}
```

### CLI Arguments

| Argument | Purpose |
|----------|---------|
| `--print` | Enable print mode for scripted output |
| `--output-format text` | Plain text output (not JSON) |
| `--model` | Model identifier |
| `--max-tokens` | Maximum response tokens |
| `--allowedTools` | Comma-separated tool list |

---

## Tool Restrictions

Sub-agents have different tool access based on operation type.

### READ_ONLY_TOOLS

For query operations (read-only):

```typescript
const READ_ONLY_TOOLS = [
  'Read',      // Read file contents
  'Glob',      // Find files by pattern
  'Grep',      // Search file contents
  'Bash',      // Shell commands (restricted)
] as const;
```

**Bash Restrictions in Read-Only Mode:**
- No file modification commands (`rm`, `mv`, `cp` to new locations)
- No package installation (`npm install`, `pip install`)
- No git write operations (`git commit`, `git push`)

### WRITE_TOOLS

For implementation operations (full access):

```typescript
const WRITE_TOOLS = [
  ...READ_ONLY_TOOLS,
  'Edit',      // Modify existing files
  'Write',     // Create new files
  'MultiEdit', // Multiple file edits
] as const;
```

### Tool Restriction Type

```typescript
type ToolRestriction = 'read-only' | 'write';

// Usage in execution options
const options: AgentExecutionOptions = {
  prompt: 'Find all TODO comments',
  allowedTools: 'read-only',  // Only READ_ONLY_TOOLS
  // ...
};
```

### How Restrictions Are Enforced

```typescript
// Tools formatted for Claude CLI
formatTools(restriction: ToolRestriction): string {
  const tools = restriction === 'read-only'
    ? READ_ONLY_TOOLS
    : WRITE_TOOLS;
  return tools.join(',');
}

// Passed as CLI argument
// claude --allowedTools "Read,Glob,Grep,Bash"
```

---

## Adding Custom Adapters

To add support for a new AI provider:

### 1. Create Adapter Class

```typescript
// ~/.context-bridge/adapters/my-adapter.ts
import { AgentAdapter, AgentExecutionOptions, AgentResult } from 'context-bridge';

export class MyAdapter implements AgentAdapter {
  readonly id = 'my-adapter';
  readonly name = 'My Custom Adapter';

  async execute(options: AgentExecutionOptions): Promise<AgentResult> {
    // Implementation
  }

  getCapabilities() {
    return {
      supportsQuery: true,
      supportsImplementation: true,
      supportsStreaming: false,
      maxContextSize: 100000,
      availableModels: ['model-v1', 'model-v2'],
    };
  }

  async isAvailable() {
    // Check if CLI tool is installed
  }

  async getVersion() {
    // Return CLI version
  }
}
```

### 2. Export Adapter

```typescript
// ~/.context-bridge/adapters/index.ts
export { MyAdapter } from './my-adapter';
```

### 3. Configure Usage

```yaml
# ~/.context-bridge/app-config.yaml
agent:
  type: my-adapter
  options:
    # Adapter-specific options
```

---

## See Also

- [Architecture Overview](./) - High-level system architecture
- [Storage Layer](./storage-layer.md) - Data persistence and caching
- [Agent Commands](../cli/agent-commands.md) - CLI agent management
- [Configuration](../getting-started/configuration.md) - Agent configuration options
