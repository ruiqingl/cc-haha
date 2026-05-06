# Claude Code — Code Structure Deep Dive

## Entry Points & Initialization

### CLI Entrypoint: `src/entrypoints/cli.tsx`
**What it does**: Launches the CLI application
- Parses command-line arguments (Commander.js)
- Sets up environment
- Calls `main.tsx` to start the TUI

```
cli.tsx → parseArgs() → main.tsx
```

### Main TUI Logic: `src/main.tsx`
**What it does**: Orchestrates React + Ink rendering
- Initializes React rendering
- Sets up the REPL screen
- Manages TUI lifecycle

```
main.tsx → React.render() → REPL component
```

### Recovery CLI: `src/localRecoveryCli.ts`
**What it does**: Fallback mode if TUI fails
- Used when `CLAUDE_CODE_FORCE_RECOVERY_CLI=1`
- Basic CLI interface without fancy TUI

---

## Core Execution Pipeline

### QueryEngine: `src/QueryEngine.ts`
**The heart of everything — orchestrates request execution**

Key responsibilities:
- Manages conversation history and context
- Executes the query loop (agentic reasoning)
- Routes to tools and processes results
- Handles streaming and completions

Main methods:
```typescript
class QueryEngine {
  initialize()        // Setup phase
  processQuery()      // Main agentic loop
  executeTools()      // Tool invocation
  handleCompletion()  // Result processing
  streamText()        // Streaming output
}
```

**Data Flow**:
```
Input Query
    ↓
parseQuery() — Extract intent and parameters
    ↓
buildPrompt() — Create system/user messages
    ↓
callAPI() — Send to Claude (streaming)
    ↓
parseToolUse() — Detect tool_use blocks
    ↓
executeTools() — Invoke requested tools
    ↓
appendResults() — Add tool results to context
    ↓
Loop or Complete? — If more steps needed, loop back
    ↓
formatOutput() — Prepare response
    ↓
renderUI() — Display via Ink TUI
```

---

### Tool System: `src/Tool.ts` + `src/tools/`

**Tool Base Class** (`Tool.ts`):
```typescript
abstract class Tool {
  name: string                  // Tool identifier (e.g., 'bash')
  description: string           // Human-readable description
  schema: ZodSchema            // Parameters schema (validation)
  
  async call(input: T): Promise<Result>  // Execute the tool
}
```

**Tool Registry**:
- Tools are auto-discovered from `src/tools/` directory
- Each tool is a class extending `Tool`
- Schema is used for validation and Claude's tool_use format

**Common Tools**:
- `Bash` — Execute shell commands
- `Grep` — Search files and content
- `Edit` — Read/write files with diff support
- `AgentTool` — Spawn sub-agents or teammates
- `Think` — Internal reasoning (no execution)

**Tool Execution Flow**:
```
Tool Call Request (from Claude)
    ↓
Validate Parameters (against schema)
    ↓
Execute Tool Logic
    ↓
Format Result
    ↓
Return to QueryEngine
    ↓
Append to Context
```

---

## Multi-Agent System

### AgentTool: `src/tools/AgentTool/AgentTool.tsx`

**Entry point for all agent creation**

Routes to 4 different agent types based on parameters:

```typescript
AgentTool.call(input)
  │
  ├─ IF team_name + name ────→ spawnTeammate()    [Path 1]
  │
  ├─ IF run_in_background ──→ registerAsyncAgent() [Path 2]
  │
  ├─ IF fork enabled ────────→ buildForkedContext() [Path 3]
  │
  └─ DEFAULT ────────────────→ runAgent()          [Path 4]
```

### Agent Types

| Type | Execution | Context | Use Case |
|------|-----------|---------|----------|
| **Subagent** | Synchronous, in-process | Independent | Child tasks, parallel work |
| **Fork** | Synchronous, in-process | Inherited (cached) | Quick variants of current task |
| **Teammate** | Async, separate process | Independent + mailbox | Long-running tasks, team collab |
| **Remote** | Remote execution | Sent to CCR | Execution on other machines |

### Agent Execution: `src/tools/AgentTool/runAgent.ts`

Main agent execution function:

```typescript
async function runAgent(config: AgentConfig): Promise<AgentResult> {
  // 1. Initialize agent context
  const context = buildAgentContext(config)
  
  // 2. Create QueryEngine for this agent
  const engine = new QueryEngine(context)
  
  // 3. Run query loop (same as main QueryEngine)
  const result = await engine.processQuery(config.instructions)
  
  // 4. Return results to parent
  return result
}
```

**Agent Lifecycle**:
```
Spawn Agent
    ↓
Initialize Context
    ↓
Create QueryEngine
    ↓
Query Loop (same as main)
    ↓
Track Progress + Update UI
    ↓
Parent Queries Results
    ↓
Return Output
    ↓
Cleanup
```

### Context Forking: `src/utils/forkedAgent.ts`

How fork agents inherit context:

```typescript
function buildForkedContext(parentContext: Context): Context {
  // 1. Copy parent messages array
  const forkedMessages = [...parentContext.messages]
  
  // 2. Cache parent's tool registry
  const toolRegistry = parentContext.tools  // Reused, not copied
  
  // 3. Snapshot important state
  const forkedContext = {
    messages: forkedMessages,
    tools: toolRegistry,           // Shared reference
    workingDir: parentContext.workingDir,
    environment: parentContext.environment,
  }
  
  return forkedContext
}
```

**Byte Consistency**: Fork ensures that the same context state produces identical results (deterministic).

### Team Coordination: `src/utils/swarm/`

How teammates communicate:

```typescript
// Teammate 1
await mailbox.send('teammate-2', {
  type: 'request',
  data: { task: 'analyze results' }
})

// Teammate 2
const message = await mailbox.receive('teammate-1')
const result = await processMessage(message)

await mailbox.send('teammate-1', {
  type: 'response',
  data: result
})
```

---

## Terminal UI (Ink)

### Ink Setup: `src/ink/`

How React renders to terminal:

```typescript
// src/ink/index.ts
import { render } from 'ink'
import React from 'react'

// Create React component tree
const app = React.createElement(REPL, { initialPrompt })

// Render to terminal (not DOM!)
render(app)
```

**Key Difference**: No DOM, renders directly to terminal output.

### REPL Component: `src/screens/REPL.tsx`

**Main interactive screen**:
- Displays conversation history
- Shows current input buffer
- Renders tool outputs
- Handles user input

```typescript
const REPL: React.FC = () => {
  const [messages, setMessages] = useState([])
  const [input, setInput] = useState('')
  
  return (
    <Box flexDirection="column">
      <ConversationHistory messages={messages} />
      <InputBuffer value={input} onChange={setInput} />
      <StatusBar status="ready" />
    </Box>
  )
}
```

### Component Patterns

**Text Output**:
```typescript
<Box>
  <Text color="green">✓ Success</Text>
</Box>
```

**Layout**:
```typescript
<Box flexDirection="column" width={80}>
  <Box>Left</Box>
  <Box marginLeft={10}>Right</Box>
</Box>
```

**Conditional Rendering**:
```typescript
{isLoading && <Spinner />}
{hasError && <ErrorBox error={error} />}
```

---

## Services Layer

### API Client: `src/services/api/`

Wraps Anthropic SDK:

```typescript
class AnthropicClient {
  async createCompletion(config) {
    // 1. Build request
    const request = this.buildRequest(config)
    
    // 2. Call Anthropic API
    const response = await this.client.messages.create(request)
    
    // 3. Handle streaming
    if (response.stream) {
      for await (const chunk of response) {
        yield chunk
      }
    }
    
    return response
  }
}
```

### MCP Integration: `src/services/mcp/`

Model Context Protocol servers:

```typescript
class MCPManager {
  async loadServers() {
    // 1. Read MCP config
    const config = readMCPConfig()
    
    // 2. Start each server
    for (const serverConfig of config.servers) {
      await this.startServer(serverConfig)
    }
    
    // 3. Register their tools
    this.registerServerTools()
  }
}
```

---

## Message Flow & Context

### Context Structure: `src/context.ts`

```typescript
interface Context {
  // Conversation history
  messages: Message[]
  
  // Tool registry
  tools: Map<string, Tool>
  
  // Execution state
  workingDir: string
  environment: Record<string, string>
  
  // Permissions tracking
  permissions: PermissionCache
  
  // Session data
  sessionId: string
  createdAt: Date
  
  // Advanced features
  memory?: MemoryManager
  skills?: SkillRegistry
}

interface Message {
  role: 'user' | 'assistant' | 'tool'
  content: string | ContentBlock[]
  timestamp?: Date
}
```

### Message Routing: `src/bridge/`

How messages flow between components:

```
User Input
    ↓ (entrypoints/cli.tsx)
QueryEngine.processQuery()
    ↓ (Tool invocation detected)
ToolRouter.route()
    ↓ (Execute tool)
Tool.call()
    ↓ (Result returned)
QueryEngine.appendResult()
    ↓ (UI update)
REPL Component (Ink)
    ↓ (Terminal output)
User sees result
```

---

## State Management

### Global State: `src/state/`

Reactive state for UI updates:

```typescript
// Observable pattern
const stateStore = {
  messages$: observable(initialMessages),
  loading$: observable(false),
  error$: observable(null),
  
  addMessage: (msg) => stateStore.messages$.next([...stateStore.messages, msg]),
  setLoading: (val) => stateStore.loading$.next(val),
}
```

### Hooks: `src/hooks/`

React hooks for state management:

```typescript
function useMessages() {
  const [messages, setMessages] = useState([])
  
  useEffect(() => {
    const subscription = stateStore.messages$.subscribe(setMessages)
    return () => subscription.unsubscribe()
  }, [])
  
  return messages
}
```

---

## Advanced Systems

### Memory System: `src/memdir/`

Cross-session persistent memory:

```
User Query
    ↓
Embed Query → Vector
    ↓
Search Memory Database
    ↓
Find Similar Past Interactions
    ↓
Inject into Context (System Prompt)
    ↓
Enhanced Query Execution
```

### Skills System: `src/skills/`

Composable workflows:

```typescript
interface Skill {
  name: string
  trigger: (context) => boolean  // When to activate
  steps: SkillStep[]              // What to do
}

// Multi-step skill
const analyticsSkill = {
  name: 'analyze_code',
  trigger: (ctx) => ctx.query.includes('analyze'),
  steps: [
    { type: 'tool', name: 'grep', params: {...} },
    { type: 'tool', name: 'bash', params: {...} },
    { type: 'agent', name: 'summarizer' },
  ]
}
```

### Channel System: `src/bridge/channels/`

Remote control via IM:

```
Telegram Message
    ↓
TelegramAdapter.parse()
    ↓
Extract Prompt + Metadata
    ↓
QueryEngine.processQuery()
    ↓
Format Result for Telegram
    ↓
Send Back to Telegram
```

---

## Common Execution Patterns

### Pattern 1: Simple Tool Execution
```
User Input → QueryEngine → Tool.call() → Result → UI Display
```

### Pattern 2: Multi-Step Reasoning
```
User Query → QueryEngine Loop (multiple iterations):
  • Claude thinks about next step
  • Decides tool to call
  • Executes tool
  • Returns result
  → Loop until task complete
```

### Pattern 3: Agent Spawning
```
Current Agent → AgentTool.call() → New Agent Context → QueryEngine Loop → Results → Return to Parent
```

### Pattern 4: Team Collaboration
```
Leader Agent
  ├→ Spawn Teammate1 → QueryEngine Loop
  ├→ Spawn Teammate2 → QueryEngine Loop
  └→ Wait + Collect Results → Process via Mailbox
```

---

## File Organization Summary

```
src/
├── entrypoints/cli.tsx       ← Start here: entry point
├── main.tsx                  ← TUI orchestration
├── QueryEngine.ts            ← Core execution engine
├── Tool.ts                   ← Tool base class
├── context.ts                ← Context/message management
├── commands.ts               ← Slash commands
│
├── tools/                    ← Tool implementations
│   ├── AgentTool/           ← Multi-agent spawning
│   ├── bash/                ← Shell execution
│   ├── grep/                ← File search
│   └── edit/                ← File editing
│
├── components/               ← UI components
├── screens/                  ← Full screens (REPL)
├── ink/                      ← Ink rendering setup
├── keybindings/              ← Input handling
│
├── services/                 ← API, MCP, OAuth
├── bridge/                   ← Message routing & IM
├── state/                    ← Global state
├── hooks/                    ← React hooks
│
├── memdir/                   ← Memory system
├── skills/                   ← Skills system
├── utils/                    ← Helper functions (400+ files)
└── constants/                ← Constants & config
```

---

**This guide should help you navigate and understand the codebase structure!**
