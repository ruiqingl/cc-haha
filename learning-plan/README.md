# Claude Code (cc-haha) Learning Resources

This folder contains a **complete learning roadmap** for understanding how Claude Code was designed and implemented.

## 📚 Files in This Folder

### 1. **plan.md** — Main Learning Roadmap ⭐
The comprehensive 6-phase learning plan covering:
- **Phase 1**: High-Level Architecture (30-45 min)
- **Phase 2**: Core Engine & Pipeline (2-3 hours)
- **Phase 3**: UI & Terminal Rendering (1.5-2 hours)
- **Phase 4**: Multi-Agent System (2-3 hours)
- **Phase 5**: Advanced Features (1-1.5 hours)
- **Phase 6**: Hands-On Development (3-5 hours)

**Total time investment**: ~12-20 hours for comprehensive understanding

**Start here if**: You want a structured learning path with checkpoints and hands-on exercises.

---

### 2. **quick-reference.md** — At-a-Glance Guide
One-page reference covering:
- System architecture diagram
- Core modules overview
- Key files by learning goal
- Development commands
- Common code patterns
- Debug tips
- Learning checkpoints

**Use this when**: You need a quick reminder or want to know "where's the code for X?"

---

### 3. **code-structure-guide.md** — Deep Dive Documentation
Detailed explanation of:
- Entry points and initialization
- Core execution pipeline (QueryEngine)
- Tool system architecture
- Multi-agent system (4 agent types)
- Terminal UI (Ink + React)
- Services layer (API, MCP)
- Message flow and context
- State management
- Advanced systems (Memory, Skills, Channels)
- Common execution patterns
- File organization summary

**Use this when**: You're studying a specific subsystem and need to understand how it works internally.

---

## 🎯 How to Use These Resources

### For First-Time Learners
1. Read **plan.md** — Phase 1 section only (overview in 30-45 mins)
2. Skim **quick-reference.md** to get familiar with key files
3. Start Phase 2 when ready, following the study path in **plan.md**

### For Focused Learning
Use **quick-reference.md** to find the right files, then use **code-structure-guide.md** to understand the subsystem.

Example: "How do agents work?"
1. Read quick-reference.md section "Multi-Agent System"
2. Review code-structure-guide.md section "Multi-Agent System"
3. Then read the actual code in `src/tools/AgentTool/`

### For Hands-On Development
Follow **Phase 6** in **plan.md** and use **code-structure-guide.md** section "Common Code Patterns" for examples.

---

## 📖 Key Documentation in the Repo

Beyond these learning files, the main repository has:

| Document | Location | Purpose |
|----------|----------|---------|
| Project Overview | `README.md` | Feature list, tech stack, quick start |
| Guidelines | `AGENTS.md` | Module organization, coding style |
| Architecture Images | `docs/images/` | 8 core system diagrams |
| Project Structure | `docs/reference/project-structure.md` | Detailed directory layout |
| Agent Deep Dive | `docs/agent/02-implementation.md` | Complex multi-agent system |
| Memory System | `docs/memory/` | Cross-session persistence |
| Skills | `docs/skills/` | Extensible workflows |
| Desktop App | `docs/desktop/` | Tauri UI implementation |
| FAQs | `docs/guide/faq.md` | Common questions and fixes |

---

## 🚀 Quick Start

1. **Read plan.md Phase 1** (30 mins)
   - Understand system overview
   - Learn module organization

2. **Skim the repo structure**
   ```bash
   cd ~/learnings/ai/cc-haha
   ls -la src/
   cat AGENTS.md
   ```

3. **Follow Phase 2** from plan.md
   - Study QueryEngine.ts
   - Trace a tool execution
   - Understand message flow

4. **Continue phases** as your understanding grows

---

## 💡 Learning Tips

- **Read Code**: Start with `src/entrypoints/cli.tsx` → `src/main.tsx` → `src/QueryEngine.ts`
- **Trace Execution**: Add `console.log()` and run `./bin/claude-haha -p "test"`
- **Build Something**: Create a simple tool or component to solidify understanding
- **Reference**: Keep quick-reference.md open while coding
- **Debug**: Use code-structure-guide.md to understand architecture when confused

---

## 📊 Learning Progress Tracker

Use this to track your learning:

```markdown
Phase 1: High-Level Architecture
  [ ] Read AGENTS.md and README.md
  [ ] View 8 architecture diagrams
  [ ] Draw system architecture sketch
  ✓ COMPLETE when: Can explain the 5 core modules

Phase 2: Core Engine & Pipeline
  [ ] Study QueryEngine.ts (main methods)
  [ ] Trace one tool (e.g., Bash)
  [ ] Understand message flow
  [ ] Trace a complete query execution
  ✓ COMPLETE when: Can draw query lifecycle diagram

Phase 3: UI & Terminal Rendering
  [ ] Study Ink setup
  [ ] Review component structure
  [ ] Build a simple component
  ✓ COMPLETE when: Can create Ink component

Phase 4: Multi-Agent System
  [ ] Understand AgentTool routing
  [ ] Study runAgent.ts
  [ ] Learn context forking
  [ ] Understand team coordination
  ✓ COMPLETE when: Can spawn and trace an agent

Phase 5: Advanced Features
  [ ] Explore memory system
  [ ] Learn skills system
  [ ] Review channel adapters
  ✓ COMPLETE when: Can explain one advanced feature

Phase 6: Hands-On Development
  [ ] Build custom tool
  [ ] Add slash command
  [ ] Create skill
  ✓ COMPLETE when: Successfully run custom tool
```

---

## ❓ FAQ

**Q: How long will this take?**
A: ~12-20 hours for comprehensive understanding. Phases 1-3 are foundational (4-5 hours), Phases 4-6 are deep-dive (8-15 hours).

**Q: Can I skip phases?**
A: Phase 1 is essential. Phase 2 provides critical context. Phases 3-6 can be done in any order, but sequentially is recommended.

**Q: What if I get stuck?**
A: Check FAQs in `docs/guide/faq.md`, review code-structure-guide.md for that subsystem, or trace with console.log().

**Q: Where's the best place to start reading code?**
A: `src/entrypoints/cli.tsx` → `src/main.tsx` → `src/QueryEngine.ts`

**Q: How do I run the code?**
A: `./bin/claude-haha` for interactive mode, or `./bin/claude-haha -p "prompt"` for headless.

---

## 🔗 Related Resources

- **Bun Runtime**: https://bun.sh
- **React**: https://react.dev
- **Ink (Terminal UI)**: https://github.com/vadimdemedes/ink
- **Anthropic SDK**: https://sdk.anthropic.com
- **MCP Protocol**: https://modelcontextprotocol.io
- **TypeScript**: https://www.typescriptlang.org

---

## 📝 Notes

- These learning materials are organized by **learning goal**, not code file order
- Each phase builds on previous knowledge
- Hands-on coding (Phase 6) is crucial for deep understanding
- The codebase is large (~40 src directories); focus on one topic at a time
- Regular commits to git help track your learning progress

---

**Happy learning! 🎓**

Start with **plan.md** and take it one phase at a time. You'll understand Claude Code's architecture and implementation in no time!
