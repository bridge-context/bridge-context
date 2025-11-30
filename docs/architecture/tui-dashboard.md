---
layout: default
title: TUI Dashboard
parent: Architecture
nav_order: 3
---

# TUI Dashboard

Deep dive into Context Bridge's Terminal User Interface (TUI) dashboard architecture, built with Ink (React for terminals), Zustand for state management, and React 18.

---

## Overview

The TUI dashboard provides real-time monitoring of Context Bridge operations directly in the terminal. It's built using modern React patterns adapted for terminal rendering.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TUI Dashboard                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │   Ink        │ →  │   Zustand    │ →  │     Components       │  │
│  │              │    │              │    │                      │  │
│  │  • React     │    │  • Global    │    │  • Screens           │  │
│  │  • Terminal  │    │    state     │    │  • Navigation        │  │
│  │  • Flexbox   │    │  • Actions   │    │  • Tables            │  │
│  │              │    │  • Selectors │    │  • Panels            │  │
│  └──────────────┘    └──────────────┘    └──────────────────────┘  │
│         │                   │                      │                │
│         └───────────────────┼──────────────────────┘                │
│                             ▼                                       │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                    Storage Events                              │ │
│  │  Real-time subscription to query:executed, session:created,   │ │
│  │  execution:completed, cache:hit/miss events                   │ │
│  └────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Technology Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Ink | 4.x | React renderer for terminals |
| React | 18.x | Component framework |
| Zustand | 4.x | Lightweight state management |
| TypeScript | 5.x | Type safety |

### Why Ink?

Ink brings React's component model to terminal applications:

- **Familiar React patterns** - hooks, components, JSX
- **Flexbox layout** - terminal-friendly layout system
- **Built-in components** - Text, Box, Spacer, etc.
- **Keyboard input handling** - useInput hook
- **Performance optimized** - efficient terminal rendering

### Why Zustand?

Zustand provides simple, performant state management:

- **Minimal boilerplate** - no providers, reducers, or actions
- **React 18 compatible** - concurrent rendering support
- **TypeScript friendly** - excellent type inference
- **Selective subscriptions** - components only re-render on used state changes

---

## Screen Architecture

The dashboard is organized into 4 main screens:

```
┌─────────────────────────────────────────────────────────────────────┐
│  [1] Projects  │  [2] Activity  │  [3] Metrics  │  [4] Settings    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         Active Screen                                │
│                                                                      │
│                                                                      │
│                                                                      │
│                                                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
│  Status Bar: Active Project • Session ID • Cache Hits              │
└─────────────────────────────────────────────────────────────────────┘
```

### 1. Projects Screen

Overview of all configured projects and their repositories.

```
┌─ Projects ──────────────────────────────────────────────────────────┐
│                                                                      │
│  ┌─ Project List ─────────────────┐  ┌─ Repository Details ───────┐ │
│  │                                │  │                            │ │
│  │  > ecommerce (active)          │  │  Name: frontend            │ │
│  │    mobile-app                  │  │  Path: /home/dev/frontend  │ │
│  │    data-platform               │  │  Queries: 42               │ │
│  │                                │  │  Last: 5 minutes ago       │ │
│  │                                │  │                            │ │
│  │                                │  │  Recent Queries:           │ │
│  │                                │  │  • Find auth components    │ │
│  │                                │  │  • List API endpoints      │ │
│  │                                │  │                            │ │
│  └────────────────────────────────┘  └────────────────────────────┘ │
│                                                                      │
│  Repos: frontend, backend, shared                                   │
└─────────────────────────────────────────────────────────────────────┘
```

**Features:**
- Project list with active indicator
- Repository details panel
- Recent query history per repo
- Query and execution statistics

### 2. Activity Screen

Real-time feed of all MCP operations and sub-agent executions.

```
┌─ Activity ──────────────────────────────────────────────────────────┐
│                                                                      │
│  Time       Tool              Repo        Duration    Status        │
│  ─────────────────────────────────────────────────────────────────  │
│  12:34:56   query_repo        frontend    1.2s        ✓ Success     │
│  12:34:52   get_repo_struct   backend     0.8s        ✓ Cached      │
│  12:34:48   implement_in_rep  frontend    15.3s       ✓ Success     │
│  12:34:30   query_all_repos   *           8.2s        ✓ Success     │
│  12:34:25   list_repos        -           0.01s       ✓ Success     │
│  12:34:20   switch_project    -           0.01s       ✓ Success     │
│                                                                      │
│  ┌─ Details ──────────────────────────────────────────────────────┐ │
│  │ Tool: query_repo                                               │ │
│  │ Query: "Find all authentication-related components"            │ │
│  │ Result: Found 8 components in src/auth/...                     │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Features:**
- Live activity feed (most recent first)
- Color-coded status indicators
- Expandable detail view
- Filter by tool type or repository

### 3. Metrics Screen

Performance statistics and cache analytics.

```
┌─ Metrics ───────────────────────────────────────────────────────────┐
│                                                                      │
│  ┌─ Session Stats ────────────────┐  ┌─ Cache Performance ────────┐ │
│  │                                │  │                            │ │
│  │  Active Sessions:    3         │  │  Hit Rate:     78.5%       │ │
│  │  Total Queries:      142       │  │  Total Hits:   112         │ │
│  │  Total Executions:   89        │  │  Total Misses: 30          │ │
│  │  Avg Query Time:     1.8s      │  │  Cache Size:   2.4 MB      │ │
│  │  Avg Exec Time:      12.4s     │  │  Entries:      86          │ │
│  │                                │  │                            │ │
│  └────────────────────────────────┘  └────────────────────────────┘ │
│                                                                      │
│  ┌─ Response Time Distribution ───────────────────────────────────┐ │
│  │                                                                 │ │
│  │  < 1s   ████████████████████████░░░░░░░░░░░░░░  42 (47%)       │ │
│  │  1-5s   ████████████████░░░░░░░░░░░░░░░░░░░░░░  28 (31%)       │ │
│  │  5-15s  ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  14 (16%)       │ │
│  │  > 15s  ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   5 (6%)        │ │
│  │                                                                 │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Features:**
- Session statistics
- Cache hit rate visualization
- Response time distribution
- Query vs implementation breakdown

### 4. Settings Screen

View and modify configuration.

```
┌─ Settings ──────────────────────────────────────────────────────────┐
│                                                                      │
│  ┌─ Agent Configuration ──────────────────────────────────────────┐ │
│  │                                                                 │ │
│  │  Adapter:        claude                                        │ │
│  │  Model:          claude-sonnet-4-20250514                            │ │
│  │  Timeout:        300,000ms (5 min)                             │ │
│  │  Max Tokens:     8,192                                         │ │
│  │                                                                 │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌─ Storage Configuration ────────────────────────────────────────┐ │
│  │                                                                 │ │
│  │  Adapter:        sqlite                                        │ │
│  │  Path:           ~/.context-bridge/storage.db                  │ │
│  │  Database Size:  4.2 MB                                        │ │
│  │  Schema Version: 3                                             │ │
│  │                                                                 │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  [Enter] Edit  │  [c] Clear Cache  │  [r] Refresh                   │
└─────────────────────────────────────────────────────────────────────┘
```

**Features:**
- Current configuration display
- Storage statistics
- Cache management actions
- Direct configuration editing

---

## Keyboard Navigation

The dashboard supports comprehensive keyboard navigation:

### Global Keys

| Key | Action |
|-----|--------|
| `1` | Switch to Projects screen |
| `2` | Switch to Activity screen |
| `3` | Switch to Metrics screen |
| `4` | Switch to Settings screen |
| `Tab` | Cycle through panels |
| `q` | Quit dashboard |
| `?` | Show help overlay |
| `r` | Refresh current screen |

### Navigation Keys

| Key | Action |
|-----|--------|
| `↑` / `k` | Move selection up |
| `↓` / `j` | Move selection down |
| `Enter` | Select / expand item |
| `Escape` | Close detail view / cancel |
| `Home` | Jump to first item |
| `End` | Jump to last item |
| `Page Up` | Scroll up one page |
| `Page Down` | Scroll down one page |

### Keyboard Handler

```typescript
import { useInput, useApp } from 'ink';

function App() {
  const { exit } = useApp();
  const { setScreen, currentScreen } = useStore();

  useInput((input, key) => {
    // Screen switching
    if (input === '1') setScreen('projects');
    if (input === '2') setScreen('activity');
    if (input === '3') setScreen('metrics');
    if (input === '4') setScreen('settings');

    // Quit
    if (input === 'q') exit();

    // Help overlay
    if (input === '?') toggleHelp();

    // Vim-style navigation
    if (input === 'j' || key.downArrow) moveDown();
    if (input === 'k' || key.upArrow) moveUp();

    // Tab navigation
    if (key.tab) cyclePanel();
  });
}
```

---

## Mouse Support

The dashboard supports mouse input for compatible terminals:

```typescript
import { useMouse } from 'ink';

function ClickableItem({ onClick, children }) {
  const { isHovered } = useMouse();

  return (
    <Box
      onClick={onClick}
      borderStyle={isHovered ? 'double' : 'single'}
    >
      {children}
    </Box>
  );
}
```

**Supported interactions:**
- Click to select items
- Click tabs to switch screens
- Hover highlights (if terminal supports)
- Scroll wheel for lists

---

## Real-Time Event Subscription

The dashboard subscribes to storage events for live updates:

```typescript
// store/useActivityStore.ts
import { useEffect } from 'react';
import { storage } from '../storage';

export function useActivitySubscription() {
  const addActivity = useStore((state) => state.addActivity);
  const updateMetrics = useStore((state) => state.updateMetrics);

  useEffect(() => {
    // Subscribe to storage events
    const subscriptions = [
      storage.on('query:executed', (log) => {
        addActivity({
          type: 'query',
          timestamp: Date.now(),
          data: log,
        });
      }),

      storage.on('execution:completed', (log) => {
        addActivity({
          type: 'execution',
          timestamp: Date.now(),
          data: log,
        });
      }),

      storage.on('cache:hit', () => {
        updateMetrics({ cacheHits: (prev) => prev + 1 });
      }),

      storage.on('cache:miss', () => {
        updateMetrics({ cacheMisses: (prev) => prev + 1 });
      }),

      storage.on('session:created', () => {
        updateMetrics({ activeSessions: (prev) => prev + 1 });
      }),
    ];

    // Cleanup on unmount
    return () => {
      subscriptions.forEach((unsubscribe) => unsubscribe());
    };
  }, [addActivity, updateMetrics]);
}
```

---

## Component Architecture

### Directory Structure

```
src/tui/
├── app.tsx                 # Root application component
├── index.tsx               # Entry point
│
├── screens/
│   ├── ProjectsScreen.tsx  # Projects view
│   ├── ActivityScreen.tsx  # Activity feed
│   ├── MetricsScreen.tsx   # Statistics dashboard
│   └── SettingsScreen.tsx  # Configuration view
│
├── components/
│   ├── Layout/
│   │   ├── Header.tsx      # App header with tabs
│   │   ├── Footer.tsx      # Status bar
│   │   └── Panel.tsx       # Bordered panel container
│   │
│   ├── Navigation/
│   │   ├── TabBar.tsx      # Screen tabs
│   │   └── Breadcrumb.tsx  # Navigation breadcrumb
│   │
│   ├── Data/
│   │   ├── Table.tsx       # Data table component
│   │   ├── List.tsx        # Selectable list
│   │   └── DetailView.tsx  # Expandable detail panel
│   │
│   └── Feedback/
│       ├── Spinner.tsx     # Loading indicator
│       ├── Badge.tsx       # Status badge
│       └── ProgressBar.tsx # Progress visualization
│
└── store/
    ├── index.ts            # Store exports
    ├── useAppStore.ts      # Global app state
    ├── useProjectStore.ts  # Project-specific state
    ├── useActivityStore.ts # Activity feed state
    └── useMetricsStore.ts  # Metrics state
```

### Core Components

#### Panel Component

```tsx
import { Box, Text } from 'ink';

interface PanelProps {
  title: string;
  focused?: boolean;
  children: React.ReactNode;
}

function Panel({ title, focused, children }: PanelProps) {
  return (
    <Box
      flexDirection="column"
      borderStyle={focused ? 'double' : 'single'}
      borderColor={focused ? 'cyan' : 'gray'}
      padding={1}
    >
      <Box marginBottom={1}>
        <Text bold color={focused ? 'cyan' : 'white'}>
          {title}
        </Text>
      </Box>
      {children}
    </Box>
  );
}
```

#### Table Component

```tsx
interface TableProps<T> {
  data: T[];
  columns: Column<T>[];
  selectedIndex?: number;
  onSelect?: (item: T) => void;
}

function Table<T>({ data, columns, selectedIndex, onSelect }: TableProps<T>) {
  return (
    <Box flexDirection="column">
      {/* Header */}
      <Box>
        {columns.map((col) => (
          <Box key={col.key} width={col.width}>
            <Text bold>{col.header}</Text>
          </Box>
        ))}
      </Box>

      {/* Rows */}
      {data.map((row, index) => (
        <Box
          key={index}
          backgroundColor={index === selectedIndex ? 'blue' : undefined}
        >
          {columns.map((col) => (
            <Box key={col.key} width={col.width}>
              <Text>{col.render(row)}</Text>
            </Box>
          ))}
        </Box>
      ))}
    </Box>
  );
}
```

### Zustand Store Pattern

```typescript
// store/useAppStore.ts
import { create } from 'zustand';

type Screen = 'projects' | 'activity' | 'metrics' | 'settings';

interface AppState {
  // State
  currentScreen: Screen;
  showHelp: boolean;
  activeProject: string | null;

  // Actions
  setScreen: (screen: Screen) => void;
  toggleHelp: () => void;
  setActiveProject: (project: string | null) => void;
}

export const useAppStore = create<AppState>((set) => ({
  // Initial state
  currentScreen: 'projects',
  showHelp: false,
  activeProject: null,

  // Actions
  setScreen: (screen) => set({ currentScreen: screen }),
  toggleHelp: () => set((state) => ({ showHelp: !state.showHelp })),
  setActiveProject: (project) => set({ activeProject: project }),
}));
```

---

## Launching the Dashboard

```bash
# Launch the TUI dashboard
context-bridge ui

# Or use the alias
context-bridge dashboard

# Or short form
cb ui
```

---

## See Also

- [Architecture Overview](./) - High-level system architecture
- [Storage Layer](./storage-layer.md) - Event system details
- [Utility Commands](../cli/utility-commands.md) - Dashboard CLI options
