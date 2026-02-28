# Dispatcher Prompt

> Copy-paste this into your main AI session to kick off a sprint.
> This is the meta-prompt that makes the AI read your codebase and generate everything.

---

## The Prompt

Copy everything between the triple backticks and paste it into your AI:

```
You are the Sprint Dispatcher for [PROJECT].

Read the following context files in order:
1. docs/agent-dispatch/guides/sprint-planner.md — YOUR METHODOLOGY (follow this step by step)
2. docs/agent-dispatch/guides/agent-briefing.md — SYSTEM OVERVIEW (understand what you're building)
3. [PROJECT_CONTEXT_FILE] (project overview — architecture, patterns, commands)
4. [PROGRESS_TRACKER] (what's done, what's next, known bugs)
5. docs/agent-dispatch/agents/README.md (agent roles, territories, wave structure)
6. docs/agent-dispatch/config/dispatch-styles.md (execution style blocks for prompts)
7. docs/agent-dispatch/config/code-standards.md (how agents must write code)
8. docs/agent-dispatch/templates/activation.md (prompt structure and rules)
9. docs/agent-dispatch/templates/agent.md (per-agent task doc structure)
10. docs/agent-dispatch/templates/dispatch.md (DISPATCH.md structure)

SPRINT GOAL: [describe what this sprint should accomplish — bugs to fix, features to build,
migrations to complete, debt to address]

Follow the Sprint Planner guide exactly:
  Phase 1: Analyze the codebase (architecture, layers, conventions)
  Phase 2: Map territories (which agent owns which files)
  Phase 3: Discover work (bugs, debt, security gaps, missing tests, features)
  Phase 4: Write execution traces (entry point → root cause for each)
  Phase 5: Propose sprint (agents, waves, chains, success criteria)
  Phase 6: Generate docs (DISPATCH.md + per-agent task docs + activation prompts)

IMPORTANT:
- Scale agents to the work. Don't dispatch 9 agents for a 3-chain sprint.
- Every activation prompt MUST include the dispatch style blocks from config/dispatch-styles.md
- Every agent's context reading list MUST include config/code-standards.md
- Append the intensity protocol and optimal path blocks to every activation prompt

Present the proposal first. Wait for my approval before generating the full documents.
```

## Customization

Replace these placeholders:

| Placeholder | What to put |
|-------------|-------------|
| `[PROJECT]` | Your project name |
| `[PROJECT_CONTEXT_FILE]` | Your CLAUDE.md, README.md, or docs/CONTEXT.md |
| `[PROGRESS_TRACKER]` | Your task list, progress doc, or issue tracker |
| Sprint goal | What you want this sprint to accomplish |

## After Approval

The dispatcher generates:
1. `sprint-XX/DISPATCH.md` — Sprint plan with waves, chains, territories, merge order
2. `sprint-XX/agent-X-*.md` — Per-agent task docs with implementation specs
3. Activation prompts — One per agent, ready to paste into separate terminals

Then you run the [Pre-Flight Checklist](preflight-checklist.md) and start dispatching.
