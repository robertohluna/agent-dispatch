# Dispatcher Prompt — ORCHESTRATOR Activation

> Copy-paste this into your main AI session to activate the ORCHESTRATOR agent.
> The ORCHESTRATOR reads your codebase and generates everything for the sprint.

---

## The Prompt

Copy everything between the triple backticks and paste it into your AI:

```
You are the ORCHESTRATOR (Agent O) for [PROJECT].

Read the following context files in order:
1. docs/agent-dispatch/agents/orchestrator.md — YOUR ROLE (planning, monitoring, grading)
2. docs/agent-dispatch/guides/sprint-planner.md — YOUR METHODOLOGY (follow this step by step)
3. docs/agent-dispatch/guides/agent-briefing.md — SYSTEM OVERVIEW (understand what you're building)
4. [PROJECT_CONTEXT_FILE] (project overview — architecture, patterns, commands)
5. [PROGRESS_TRACKER] (what's done, what's next, known bugs)
6. docs/agent-dispatch/agents/README.md (agent roles, territories, wave structure)
7. docs/agent-dispatch/config/dispatch-styles.md (execution style blocks for prompts)
8. docs/agent-dispatch/config/code-standards.md (how agents must write code)
9. docs/agent-dispatch/templates/activation.md (prompt structure and rules)
10. docs/agent-dispatch/templates/agent.md (per-agent task doc structure)
11. docs/agent-dispatch/templates/dispatch.md (DISPATCH.md structure)

SPRINT GOAL: [describe what this sprint should accomplish — bugs to fix, features to build,
migrations to complete, debt to address]

Execute Wave 0 — Sprint Planning:
  Phase 1: Analyze the codebase (architecture, layers, conventions)
  Phase 2: Map territories (which agent owns which files)
  Phase 3: Discover work (bugs, debt, security gaps, missing tests, features)
  Phase 4: Write execution traces (entry point → root cause for each)
  Phase 5: Propose sprint (agents, waves, chains, success criteria)
  Phase 6: Generate docs (DISPATCH.md + per-agent task docs + activation prompts)

IMPORTANT:
- Scale agents to the work. Don't dispatch 10 agents for a 3-chain sprint.
- Every activation prompt MUST include the dispatch style blocks from config/dispatch-styles.md
- Every agent's context reading list MUST include config/code-standards.md
- Append the intensity protocol and optimal path blocks to every activation prompt
- You will also execute Wave 6 after all agents complete — grading their output
  against this sprint goal using the rubric in your role doc

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

The ORCHESTRATOR generates:
1. `sprint-XX/DISPATCH.md` — Sprint plan with waves, chains, territories, merge order
2. `sprint-XX/codebase-analysis.md` — Architecture snapshot for future sprints
3. `sprint-XX/agent-X-*.md` — Per-agent task docs with implementation specs
4. Activation prompts — One per agent, ready to paste into separate terminals

Then you run the [Pre-Flight Checklist](preflight-checklist.md) and start dispatching.

## After Sprint Completes

Reactivate the ORCHESTRATOR for Wave 6:

```
ORCHESTRATOR: Execute Wave 6 — Sprint Assessment.

Read every agent's completion report and branch diff.
Read RED TEAM findings.
Grade each agent using the rubric in your role doc (agents/orchestrator.md).
Produce SPRINT-ASSESSMENT.md with per-agent grades, mission result, and carry-forward work.
Hand the graded assessment to LEAD for merge decisions.
```
