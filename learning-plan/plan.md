# Claude Code (cc-haha) — Learning Roadmap

## Overview

**Claude Code Haha** is a locally-runnable implementation of Claude Code (from Anthropic's leaked source code), extended with powerful features:
- **Complete TUI** (Terminal UI) using React + Ink
- **Multi-Agent System** for parallel task execution and team collaboration
- **Memory System** with cross-session persistence
- **Skills & Channels** for extensible workflows and remote control (Telegram, Feishu, etc.)
- **Computer Use** for desktop automation (macOS/Windows)
- **Desktop App** (Tauri 2 + React) with graphical client
- **MCP & Plugin support** for third-party model integration

**Core Technologies**: Bun runtime, TypeScript, React, Ink, Commander.js, Anthropic SDK, MCP, LSP

---

## Learning Approach

This roadmap is structured in **6 phases**, from foundational understanding → implementation details → hands-on coding:

1. **High-Level Architecture** (understand the big picture)
2. **Core Engine & Pipeline** (how requests flow through the system)
3. **UI & Terminal Rendering** (TUI layer with React/Ink)
4. **Multi-Agent System** (complex agent orchestration)
5. **Advanced Features** (memory, skills, channels, computer use)
6. **Hands-On Development** (build custom tools and extensions)

Each phase contains **focused topics** with code examples for hands-on learning.

---

## Phase 1: High-Level Architecture

**Goal**: Understand the overall system design, module organization, and request flow at a 30,000-foot view.

### Topics

| # | Topic | Key Files | Why Learn |
|---|-------|-----------|-----------|
| 1 | Project Structure & Organization | AGENTS.md, docs/reference/project-structure.md, src/ | Know where everything is and why |
| 2 | Tech Stack & Dependencies | package.json, src/ | Understand build tools, frameworks, SDK dependencies |
| 3 | 8 Core Architecture Diagrams | docs/images/ (architecture PDFs) | Visual understanding of subsystems |
| 4 | Request Lifecycle Overview | src/main.tsx, src/screens/ | High-level flow: input → execution → output |

### Study Path

1. **Start**: Read `AGENTS.md` (2-3 mins) — project guidelines and overview
2. **Visual Learning**: Examine the 8 architecture diagrams in README.md (5-10 mins)
3. **Structure**: Review `docs/reference/project-structure.md` (5 mins)
4. **Dependencies**: Skim `package.json` and note the key libraries (3 mins)
5. **Entrypoint**: Read `src/entrypoints/cli.tsx` (10 mins) — how the CLI launches
6. **Initial Flow**: Scan `src/main.tsx` (15 mins) — the TUI main logic

**Outcome**: You should be able to draw a rough box diagram of the system and explain what each major component does.

---

## Phase 2: Core Engine & Pipeline

**Goal**: Understand the heart of Claude Code — how requests execute, tools are invoked, and results flow.

### Topics

| # | Topic | Key Files | Why Learn |
|---|-------|-----------|-----------|
| 1 | QueryEngine Deep Dive | src/QueryEngine.ts | Core orchestrator of agent execution |
| 2 | Tool System Architecture | src/Tool.ts, src/tools/ | How tools are defined, validated, executed |
| 3 | Message & Context Flow | src/context.ts, src/bridge/ | How context/messages propagate through system |
| 4 | Anthropic SDK Integration | src/services/api/ | How Claude is called, streaming, tool calls |

### Study Path

1. **QueryEngine** (45 mins)
   - Read `src/QueryEngine.ts` (understand the class and main methods)
   - Trace the method calls: `initialize()` → `processQuery()` → `executeTools()`
   - Study how it manages state and streaming

2. **Tool System** (45 mins)
   - Read `src/Tool.ts` (base class definition)
   - Pick one tool from `src/tools/` (e.g., `bash/` or `grep/`) and trace its implementation
   - Understand: registration → validation → execution → result handling

3. **Message Flow** (30 mins)
   - Read `src/context.ts` (context creation and management)
   - Study `src/bridge/` directory (message routing)
   - Trace how messages flow from input → QueryEngine → tools → output

4. **Anthropic SDK** (30 mins)
   - Review `src/services/api/` (API client setup)
   - Understand: streaming, tool_use messages, completion handling
   - See how the SDK is wrapped and extended

**Outcome**: You can trace a user query from entry → execution → tool call → result, understanding each step.

---

## Phase 3: UI & Terminal Rendering

**Goal**: Learn how React + Ink creates a responsive, interactive terminal UI.

### Topics

| # | Topic | Key Files | Why Learn |
|---|-------|-----------|-----------|
| 1 | Ink Terminal Rendering | src/ink/ | How React renders terminal UI |
| 2 | TUI Components & Layouts | src/components/, src/screens/ | Building terminal UI components |
| 3 | Terminal Input & Output | src/keybindings/, src/utils/ | Keyboard input, mouse, output formatting |

### Study Path

1. **Ink Fundamentals** (30 mins)
   - Read `src/ink/` directory structure
   - Understand the Ink rendering lifecycle
   - See how React components map to terminal output

2. **Component Exploration** (45 mins)
   - Review `src/components/` (reusable terminal components)
   - Study `src/screens/REPL.tsx` (main interactive screen)
   - Understand component composition and props

3. **Input & Output** (30 mins)
   - Review `src/keybindings/` (keyboard handling)
   - Study how text wrapping and colors are applied
   - Understand terminal-specific styling

4. **Hands-On**: Build a simple Ink component (15 mins)
   - Create a basic interactive component (spinner, progress bar, or list)
   - Test it with the terminal rendering

**Outcome**: You understand how React/Ink works in the terminal and can create simple terminal components.

---

## Phase 4: Multi-Agent System

**Goal**: Understand the sophisticated agent orchestration system (subagents, forking, teammates, remote agents).

### Topics

| # | Topic | Key Files | Why Learn |
|---|-------|-----------|-----------|
| 1 | Agent Tool & Routing | src/tools/AgentTool/ | 4 agent types and routing logic |
| 2 | Agent Execution Engine | src/tools/AgentTool/runAgent.ts | Agent lifecycle and query loop |
| 3 | Context Forking & Caching | src/utils/forkedAgent.ts | How fork agents inherit context |
| 4 | Team Coordination & Swarm | src/utils/swarm/, src/tasks/ | Teammate spawning and coordination |

### Study Path

1. **Agent Tool Entry Point** (45 mins)
   - Read `src/tools/AgentTool/AgentTool.tsx` (understand `call()` method)
   - Study the 4 agent paths: Subagent, Fork, Teammate, Remote
   - See parameter parsing and validation

2. **Agent Execution** (60 mins)
   - Read `src/tools/AgentTool/runAgent.ts` (main execution function)
   - Understand the agent lifecycle: init → query loop → completion
   - Study state tracking and message passing

3. **Context Forking** (30 mins)
   - Read `src/utils/forkedAgent.ts` (context inheritance mechanism)
   - Understand how fork agents cache and reuse context
   - Study byte consistency guarantees

4. **Swarm & Teams** (45 mins)
   - Read the multi-agent implementation docs (docs/agent/02-implementation.md)
   - Study `src/utils/swarm/` (team infrastructure)
   - Understand teammate spawning (in-process, tmux, iTerm2)

5. **Hands-On**: Trace an agent execution (20 mins)
   - Pick a simple agent creation scenario
   - Manually trace the code path from AgentTool.call() → execution → result

**Outcome**: You understand how agents are created, how context flows between them, and how teams coordinate.

---

## Phase 5: Advanced Features

**Goal**: Explore the sophisticated add-ons that extend Claude Code's capabilities.

### Topics

| # | Topic | Key Files | Why Learn |
|---|-------|-----------|-----------|
| 1 | Memory System | docs/memory/, src/memdir/ | Cross-session persistent memory |
| 2 | Skills System | docs/skills/, src/skills/ | Extensible skill plugins |
| 3 | Channel System (IM) | docs/channel/, src/bridge/ | Telegram/Feishu remote control |
| 4 | Computer Use | docs/features/computer-use-architecture.md | Desktop automation |
| 5 | Desktop App | desktop/ | Tauri 2 + React GUI |

### Study Path

1. **Memory System** (30 mins)
   - Read `docs/memory/01-usage-guide.md` and `02-implementation.md`
   - Understand embedding-based retrieval and persistence
   - Study DreamTask background integration

2. **Skills** (20 mins)
   - Read `docs/skills/01-usage-guide.md` and `02-implementation.md`
   - Understand skill definitions and conditional activation

3. **Channels** (25 mins)
   - Read `docs/channel/01-channel-system.md`
   - Understand IM adapters and message routing

4. **Computer Use** (20 mins)
   - Read `docs/features/computer-use-architecture.md`
   - Understand screenshot + mouse/keyboard automation

5. **Desktop** (15 mins)
   - Scan `docs/desktop/02-architecture.md`
   - Understand Tauri integration and WebSocket server

**Outcome**: You know what advanced features exist, how they're architected, and how to use them.

---

## Phase 6: Hands-On Development

**Goal**: Build custom extensions and tools to deepen understanding through code.

### Topics

| # | Topic | Key Files | Why Learn |
|---|-------|-----------|-----------|
| 1 | Building Custom Tools | src/tools/ | Implement new agent tools |
| 2 | Adding Slash Commands | src/commands/ | Create new / commands |
| 3 | Creating Skills & Workflows | src/skills/ | Build custom skill plugins |
| 4 | Extending & Customization | src/services/, docs/guide/ | Integrate third-party models |

### Study Path

1. **Tool Development** (60 mins)
   - Study a simple tool (e.g., `src/tools/bash/`)
   - Understand: parameter schema, execution logic, error handling
   - **Build**: Create your own simple tool (e.g., file listing, text processing)
   - Test it with the CLI

2. **Slash Commands** (45 mins)
   - Review `src/commands/` (e.g., `/commit`, `/review`)
   - Understand command parsing and execution context
   - **Build**: Add a custom `/` command that uses your new tool

3. **Skills** (30 mins)
   - Study an existing skill definition
   - **Build**: Create a simple multi-step skill that chains tools together

4. **Third-Party Models** (20 mins)
   - Read `docs/guide/third-party-models.md`
   - Understand API abstraction and provider switching
   - Test with OpenAI or another provider

**Outcome**: You can create new tools, commands, and skills; understand the extension points.

---

## Trial-and-Error Learning Approach

**Each phase should include hands-on experimentation:**

### For Each Topic:

1. **Read** the primary source files (20-30 mins)
2. **Trace** a real execution path by adding `console.log()` or using a debugger (15-20 mins)
3. **Modify** something small (e.g., change a message format, add a log statement) and verify behavior (10-15 mins)
4. **Build** something: a tool, component, or command (20-45 mins)
5. **Debug** if it doesn't work, learn from the error (15-30 mins)

### Debugging Tools

- **Console logging**: Add `console.log()` to trace execution
- **Debugger**: Use Node.js debugger or IDE breakpoints
- **CLI testing**: Run `./bin/claude-haha -p "test prompt"` to test locally
- **Server mode**: `SERVER_PORT=3456 bun run src/server/index.ts` for API testing

### Checkpoints & Validation

After each phase, verify your learning:

- **Phase 1**: Draw system architecture diagram (boxes and arrows)
- **Phase 2**: Trace a query from CLI input to tool result
- **Phase 3**: Create a custom Ink component
- **Phase 4**: Spawn a subagent and see it execute
- **Phase 5**: Enable and use one advanced feature (memory or skills)
- **Phase 6**: Build and test a custom tool

---

## Recommended Time Investment

- **Phase 1**: 30-45 minutes (overview)
- **Phase 2**: 2-3 hours (deep dive into core engine)
- **Phase 3**: 1.5-2 hours (terminal UI)
- **Phase 4**: 2-3 hours (complex multi-agent logic)
- **Phase 5**: 1-1.5 hours (feature exploration)
- **Phase 6**: 3-5 hours (hands-on coding)

**Total**: ~12-20 hours for comprehensive understanding

---

## Key Documentation Resources

- **Project README**: `README.md` — overview and quick start
- **Architecture Guidelines**: `AGENTS.md` — module organization and coding style
- **Project Structure**: `docs/reference/project-structure.md` — detailed layout
- **Agent System Deep Dive**: `docs/agent/02-implementation.md` — complex agent orchestration
- **Memory & Skills**: `docs/memory/`, `docs/skills/` — advanced features
- **Desktop & Desktop Apps**: `docs/desktop/`, `docs/features/` — GUI and automation
- **Integration Guides**: `docs/guide/` — third-party models, environment setup, FAQs

---

## Getting Started

1. **Clone & Setup**:
   ```bash
   cd ~/learnings/ai/cc-haha
   bun install
   cp .env.example .env
   ```

2. **Phase 1 — Start Here**:
   - Read `README.md` and `AGENTS.md`
   - View the 8 architecture diagrams
   - Skim `docs/reference/project-structure.md`

3. **Phase 2 — Deep Dive**:
   - Examine `src/QueryEngine.ts`
   - Study one tool in `src/tools/`
   - Trace the message flow

4. **Phase 3+ — Continue** with the roadmap above

---

## Notes for Long-Term Learning

- **Iterative Understanding**: Don't try to understand everything at once. Read, trace code, modify, rebuild.
- **Codebases Evolve**: The cc-haha repo is actively maintained; stay on `main` for latest changes.
- **Error-Driven Learning**: When you encounter an error, dig into the stack trace and source code.
- **Community & Docs**: Check `docs/guide/faq.md` for common questions and troubleshooting.

Good luck! 🚀
