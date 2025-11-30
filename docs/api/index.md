---
layout: default
title: API Reference
nav_order: 5
has_children: true
---

# API Reference

Complete TypeScript API reference for Context Bridge's core interfaces, types, and exports.

---

## Quick Links

| Interface | Description |
|-----------|-------------|
| [AgentAdapter](./agent-adapter) | Interface for AI agent implementations |
| [StorageAdapter](./storage-adapter) | Interface for data persistence backends |

---

## Package Exports

Context Bridge exports its core types and classes from the `@agent-context-bridge/cli` package.

### Agent Types

```typescript
import {
  AgentAdapter,
  AgentExecutionOptions,
  AgentResult,
  InstallationStatus,
} from '@agent-context-bridge/cli/agent/types';
```

| Export | Type | Description |
|--------|------|-------------|
| `AgentAdapter` | Interface | Core agent adapter interface |
| `AgentExecutionOptions` | Interface | Options passed to execute() |
| `AgentResult` | Interface | Result returned from execute() |
| `InstallationStatus` | Interface | CLI installation validation result |

### Base Adapter

```typescript
import { BaseAdapter } from '@agent-context-bridge/cli/agent/adapters/base';
```

| Export | Type | Description |
|--------|------|-------------|
| `BaseAdapter` | Class | Abstract base class with utility methods |

### Agent Errors

```typescript
import {
  AgentError,
  AgentTimeoutError,
  AgentNotFoundError,
} from '@agent-context-bridge/cli/agent/errors';
```

| Export | Type | Description |
|--------|------|-------------|
| `AgentError` | Class | Base error class for agent operations |
| `AgentTimeoutError` | Class | Thrown when agent execution times out |
| `AgentNotFoundError` | Class | Thrown when agent CLI is not installed |

---

## Creating Extensions

Context Bridge supports custom adapters for both agents and storage backends.

### Custom Agent Adapters

Extend `BaseAdapter` to create your own agent adapter:

```typescript
import {
  BaseAdapter,
  AgentAdapter,
  AgentExecutionOptions,
  AgentResult,
} from '@agent-context-bridge/cli';

export class MyAdapter extends BaseAdapter implements AgentAdapter {
  readonly name = 'my-adapter';
  readonly version = '1.0.0';

  async execute(options: AgentExecutionOptions): Promise<AgentResult> {
    // Build command and execute
    const args = this.buildCommand(options);
    // ... spawn process and capture output
    return this.parseOutput(output);
  }

  async validateInstallation(): Promise<InstallationStatus> {
    // Check if CLI tool is installed
  }

  parseOutput(output: string): string {
    // Parse CLI output into structured result
  }

  buildCommand(options: AgentExecutionOptions): string[] {
    // Build CLI arguments from options
  }

  mapTools(tools: string[]): string[] {
    // Map Context Bridge tools to adapter-specific names
  }
}
```

### Adapter Discovery

Place custom adapters in one of these locations:

| Location | Priority | Scope |
|----------|----------|-------|
| `~/.context-bridge/adapters/` | User | All projects |
| `./.context-bridge/adapters/` | Project | Current project only |

Adapters are loaded at startup and validated against the interface.

---

## Type Safety with Zod Schemas

Context Bridge uses [Zod](https://zod.dev) for runtime type validation, ensuring type safety at both compile time and runtime.

### Schema Validation

All external inputs are validated using Zod schemas:

```typescript
import { z } from 'zod';

// Agent execution options schema
const AgentExecutionOptionsSchema = z.object({
  prompt: z.string().min(1),
  workingDirectory: z.string(),
  context: z.string().optional(),
  allowedTools: z.enum(['read-only', 'write']),
  timeout: z.number().positive(),
  model: z.string().optional(),
  maxTokens: z.number().positive().optional(),
  sessionId: z.string().uuid(),
});

// Type inference from schema
type AgentExecutionOptions = z.infer<typeof AgentExecutionOptionsSchema>;
```

### Validation Patterns

```typescript
// Validate input and get typed result
const options = AgentExecutionOptionsSchema.parse(rawInput);

// Safe parsing with error handling
const result = AgentExecutionOptionsSchema.safeParse(rawInput);
if (!result.success) {
  console.error('Validation errors:', result.error.issues);
}
```

### Key Schemas

| Schema | Purpose |
|--------|---------|
| `AgentExecutionOptionsSchema` | Validate agent execution inputs |
| `AgentResultSchema` | Validate agent execution outputs |
| `ProjectConfigSchema` | Validate project configuration |
| `StorageConfigSchema` | Validate storage configuration |

---

## In This Section

- [AgentAdapter](./agent-adapter) - Agent adapter interface reference
- [StorageAdapter](./storage-adapter) - Storage adapter interface reference

---

## See Also

- [Architecture Overview](../architecture) - System design and patterns
- [Custom Adapters Guide](../guides/custom-adapters) - Step-by-step adapter creation
- [Configuration](../getting-started/configuration) - Configuration options
