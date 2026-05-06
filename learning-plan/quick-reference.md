# Claude Code (cc-haha) — Quick Reference Guide

## System Architecture at a Glance

```
┌─────────────────────────────────────────────┐
│  User Input (CLI / Desktop UI)              │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│  entrypoints/cli.tsx → main.tsx             │
│  (CLI parsing + TUI initialization)         │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│  QueryEngine (src/QueryEngine.ts)           │
│  - Request orchestration                    │
│  - Tool routing & execution                 │
│  - Stream management                        │
└────────────┬────────────────────────────────┘
             │
      ┌──────┴──────┐
      │ Tool System │
      │ (src/tools/)│
      └──────┬──────┘
             │
    ┌────────┼────────────┐
    │        │            │
    ▼        ▼            ▼
  Bash     Grep         Edit
  (Shell) (Search)      (File)
    │        │            │
    └────────┼────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│  Anthropic API (Claude LLM)                 │
│  - API calls with tool_use                  │
│  - Streaming completion                     │
└─────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│  Response Processing                        │
│  - Parse results                            │
│  - Format output                            │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│  Ink TUI Rendering (src/components/)        │
│  - Terminal output + interactive UI         │
│  - React component tree → terminal          │
└─────────────────────────────────────────────┘
```

---

## Core Modules Overview

| Module | File(s) | Purpose |
|--------|---------|---------|
| **CLI Entrypoint** | `src/entrypoints/cli.tsx` | Launches the CLI, parses args |
| **Main TUI Logic** | `src/main.tsx` | Commander.js + React setup |
| **Query Engine** | `src/QueryEngine.ts` | Core execution orchestrator |
| **Tool System** | `src/Tool.ts`, `src/tools/` | Tool definitions and execution |
| **Ink Rendering** | `src/ink/` | Terminal UI rendering engine |
| **Components** | `src/components/` | Reusable terminal UI components |
| **Services** | `src/services/` | API, MCP, OAuth, OAuth, plugins |
| **Context** | `src/context.ts`, `src/bridge/` | Message routing & context flow |
| **Agent Tool** | `src/tools/AgentTool/` | Multi-agent orchestration |
| **State** | `src/state/` | Global state management |
| **Utils** | `src/utils/` | Helper functions (400+ files) |

---

## Key Files by Learning Goal

### Understanding Architecture
- `AGENTS.md` — project guidelines
- `docs/images/01-overall-architecture.png` — system diagram
- `docs/reference/project-structure.md` — directory structure
- `README.md` — feature overview

### Learning Query Execution
- `src/entrypoints/cli.tsx` — entry point
- `src/main.tsx` — TUI setup
- `src/QueryEngine.ts` — core engine
- `src/Tool.ts` — tool interface

### Building TUI
- `src/ink/` — Ink setup and rendering
- `src/components/` — terminal components
- `src/screens/REPL.tsx` — main screen
- `src/keybindings/` — input handling

### Multi-Agent System
- `src/tools/AgentTool/AgentTool.tsx` — agent entry point
- `src/tools/AgentTool/runAgent.ts` — execution logic
- `src/utils/forkedAgent.ts` — context forking
- `src/utils/swarm/` — team coordination
- `docs/agent/02-implementation.md` — deep dive

### Advanced Features
- `src/memdir/` — memory system
- `src/skills/` — skill plugins
- `src/bridge/` — channel adapters
- `docs/memory/`, `docs/skills/`, `docs/channel/` — docs

### Desktop App
- `desktop/` — Tauri + React frontend
- `src/server/` — API/WebSocket backend
- `desktop/src-tauri/` — Tauri glue

---

## Development Commands

```bash
# Run CLI locally
./bin/claude-haha

# Run CLI with prompt (headless mode)
./bin/claude-haha -p "your prompt here"

# Start API server (for desktop dev)
SERVER_PORT=3456 bun run src/server/index.ts

# Dev desktop frontend
cd desktop && bun run dev

# Build desktop frontend
cd desktop && bun run build

# Run desktop tests
cd desktop && bun run test

# Build docs locally
bun run docs:dev

# Help with CLI options
./bin/claude-haha --help
```

---

## How to Read the Code

### 1. Follow the Entrypoint
Start with `src/entrypoints/cli.tsx` → `src/main.tsx` → understand the flow

### 2. Trace Tool Execution
Pick a simple tool (e.g., `src/tools/bash/Bash.tsx`):
- See parameter schema
- Understand execution method
- Study result formatting

### 3. Find the Query Engine
`src/QueryEngine.ts` is the orchestrator:
- `initialize()` — setup
- `processQuery()` — main loop
- `executeTools()` — tool invocation
- `handleCompletion()` — result handling

### 4. Understand Context
`src/context.ts` defines context structure:
- Messages array
- Tool registry
- Execution state
- Cached data

### 5. Study Multi-Agent
Complex but powerful:
- `src/tools/AgentTool/AgentTool.tsx` — entry
- `src/tools/AgentTool/runAgent.ts` — execution
- `src/utils/swarm/` — team infrastructure

---

## Common Code Patterns

### Creating a Tool
```typescript
// src/tools/MyTool/MyTool.tsx
export default class MyTool extends Tool {
  name = 'my_tool'
  description = 'Does something useful'
  
  schema = z.object({
    param1: z.string(),
  })
  
  async call(input: { param1: string }) {
    // Your implementation
    return { result: 'output' }
  }
}
```

### Building a Component
```typescript
// src/components/MyComponent.tsx
import React from 'react'
import { Box, Text } from 'ink'

export const MyComponent: React.FC<{ title: string }> = ({ title }) => (
  <Box>
    <Text>{title}</Text>
  </Box>
)
```

### Spawning an Agent
```typescript
// Inside a tool or command
await agentTool.call({
  subagent_type: 'subagent',
  name: 'my_agent',
  instructions: 'Do something',
  model_override: 'claude-3-5-sonnet-20241022',
})
```

---

## Debug Tips

### 1. Add Logging
```typescript
console.log('Debug:', { variable: value })
```

### 2. Use Node Debugger
```bash
node --inspect-brk ./bin/claude-haha
```
Then open `chrome://inspect` in Chrome

### 3. Test with Print Mode
```bash
./bin/claude-haha -p "test prompt" --print
```

### 4. Check Environment Variables
```bash
cat .env
```

### 5. Read Error Stack Traces
Usually shows the failing file and line number

---

## Learning Checkpoints

After each phase, verify:

✓ **Phase 1**: Can you draw the system architecture?
✓ **Phase 2**: Can you trace a query from input to output?
✓ **Phase 3**: Can you create a simple Ink component?
✓ **Phase 4**: Can you spawn a subagent and see it work?
✓ **Phase 5**: Can you enable one advanced feature?
✓ **Phase 6**: Can you build and test a custom tool?

---

## Useful Resources

| Resource | Link |
|----------|------|
| Project README | `README.md` |
| Architecture Diagrams | `docs/images/` |
| Bun Runtime | https://bun.sh |
| React Docs | https://react.dev |
| Ink (Terminal UI) | https://github.com/vadimdemedes/ink |
| Anthropic SDK | https://github.com/anthropics/anthropic-sdk-python |
| MCP Protocol | https://modelcontextprotocol.io |

---

**Happy coding! 🚀**

Start with Phase 1 to build mental models, then dive into Phase 2-3 for hands-on learning.
