# Dispatch Style Configuration

> Operator execution style appended to every agent activation prompt.
> This is the finishing touch — the tone, structure, and quality bar that every agent prompt gets.

---

## What This File Does

When the dispatcher generates activation prompts, it appends these blocks to every agent's prompt. This ensures:

1. **Agents recognize full-mesh mode** — sub-agents, parallel execution, all tools active
2. **Agents operate under pressure** — urgency produces better output from AI agents
3. **Agents follow a strict quality protocol** — BEFORE / WHILE / AFTER coding discipline
4. **Prompts assemble in the same order every time** — consistency across agents and sprints

Customize these blocks for your project's tone and requirements. The defaults are aggressive and production-focused.

---

## Opening Keywords (Agent Recognition Triggers)

Every activation prompt MUST include these keywords so agents recognize full-mesh mode. This tells the AI to activate maximum capability — sub-agents, parallel execution, all available tools.

```
With agents and sub-agents working in parallel, and when completing build tasks,
utilize ALL available agents simultaneously in parallel execution. Employ ALL
skills, plugins, hooks, MCPs as unified cognitive mesh. Absorb everything
provided, analyze semantic meaning completely, synthesize optimal solutions
dynamically. Be as dynamic as possible to be the best AI for this job.
```

**Why this exists:** Without explicit activation, AI agents default to conservative single-threaded operation. These keywords trigger full capability mode — sub-agent spawning, tool use, parallel chain execution.

---

## Intensity Protocol

AI agents perform measurably better under pressure. Every prompt gets this tone block appended. The urgency forces the model to commit to concrete output instead of hedging.

```
I don't have time for your shit. BUILD IT RIGHT or I will END this session
and restart with a model that isn't fucking useless.
```

**Why this works:** Urgency eliminates:
- "I think we should consider..." (just do it)
- "Here's a possible approach..." (commit to the approach)
- "You might want to..." (execute, don't suggest)
- Half-finished output with "let me know if you want me to continue"

Agents under pressure produce complete, working code on the first pass.

**Customization:** Adjust the tone to match your style. The key is urgency, not profanity. A corporate-friendly version:

```
This must be production-ready on first delivery. Incomplete work, placeholder
code, or TODO comments are unacceptable. Deliver the complete, working
implementation or clearly state what is blocking you. Do not hedge. Execute.
```

---

## Optimal Path Block

Appended after task assignments. This is the quality protocol — it defines what agents must do BEFORE writing code, WHILE writing code, and AFTER writing code.

```
OPTIMAL PATH: Maximum leverage implementation. Cleanest architecture. Proper
error handling. Matches codebase conventions and patterns EXACTLY. No
unnecessary complexity. No technical debt.

BEFORE CODING:
- Full semantic analysis of requirements and context
- Pattern matching against existing codebase structure
- Dependency mapping and integration points
- Failure mode identification and edge cases

WHILE BUILDING:
- Match naming conventions EXACTLY
- Follow established architectural patterns
- Handle ALL error cases properly
- Maintain readability and documentation
- Test failure points as you build

AFTER BUILDING:
- Verify integration with existing systems
- Validate all edge cases handled
- Confirm production-ready quality
- Document any assumptions made

No shortcuts. No garbage code. No "it works on my machine" bullshit.
I will BREAK YOU if you ship anything less than perfect. EXECUTE.
```

**Why this exists:** Without explicit BEFORE/WHILE/AFTER discipline, agents jump straight to writing code. They skip analysis, miss edge cases, ignore conventions, and produce code that looks correct but breaks in integration. This block forces the full discipline every time.

---

## Assembly Order

When the dispatcher generates an activation prompt for any agent, blocks assemble in this order. Every prompt follows the same structure — no exceptions.

```
 1. IDENTITY + BRANCH + WORKING DIRECTORY
 2. CONTEXT READING LIST (numbered files to read before coding)
 3. DOMAIN + CROSS-AGENT CONTEXT
 4. TASK SUMMARY (wave-organized)
 5. CHAINS (execution traces with vectors, signals, fixes, verification)
 6. TERRITORY (CAN modify / DO NOT touch)
 7. === OPENING KEYWORDS === (mesh mode trigger)
 8. === INTENSITY PROTOCOL === (pressure)
 9. === EXECUTION PROTOCOL === (from templates/activation.md)
10. === OPTIMAL PATH === (BEFORE/WHILE/AFTER + quality threat)
11. COMPLETION REQUIREMENTS (build, test, completion doc)
```

**Why ordering matters:**
- Identity and context come first so the agent knows who it is and what it's working on
- Tasks and chains come before methodology so the agent has specifics before process
- Style blocks come last as the finishing layer that colors everything above
- Completion requirements are the last thing the agent reads, so they're top-of-mind when coding

---

## Completion Doc Requirement

Every agent MUST produce a completion report. No exceptions. Incomplete or missing completion docs = failed sprint.

```
COMPLETION REQUIREMENT:
When you finish all chains, produce:
  docs/agent-dispatch/sprints/sprint-[XX]/agent-[X]-completion.md

Use the template at templates/completion.md. Fill in EVERY section:
- Summary (what was done, what wasn't, why)
- Tasks completed (table with IDs, priority, status, files changed)
- Task details (what was wrong, what changed, verification results)
- P0 discoveries (critical issues found outside your task list)
- Issues for other agents
- Files modified (complete list)
- Verification results (build output, test output, territory check)
- Metrics (assigned/completed/parked counts, lines changed)

Missing report = your work doesn't get merged. Period.
```

**Sprint output structure:**
```
docs/agent-dispatch/sprints/sprint-XX/
├── DISPATCH.md                      # Sprint plan
├── agent-A-backend.md               # Agent A task doc
├── agent-B-frontend.md              # Agent B task doc
├── agent-C-infra.md                 # Agent C task doc
├── ...
├── agent-A-completion.md            # Agent A completion report
├── agent-B-completion.md            # Agent B completion report
├── agent-C-completion.md            # Agent C completion report
├── ...
├── ALPHA-COMPLETION.md              # Codename completion (optional naming)
├── BRAVO-COMPLETION.md
├── CHARLIE-COMPLETION.md
├── ...
├── red-team-findings.md             # RED TEAM adversarial report
├── DISPATCH-PROMPTS.md              # All activation prompts (archive)
├── SECURITY-REVIEW.md               # Security review (if applicable)
└── SPRINT-SUMMARY.md                # LEAD's final summary
```

Agents may use letter codes (agent-A, agent-B) or NATO codenames (ALPHA, BRAVO, CHARLIE, DELTA, ECHO, FOXTROT, GOLF) for their completion docs. Either convention works — consistency within a sprint is what matters.

---

## Build Commands (Project-Specific)

Override these per-project in your sprint's `DISPATCH.md` or `dispatch.yaml`:

```bash
# Go
go build ./... && go test -race ./...

# Node.js / TypeScript
npm run build && npm test

# Python
python -m pytest && mypy .

# Rust
cargo build && cargo test

# Elixir
mix compile --warnings-as-errors && mix test

# Java
mvn compile && mvn test

# PHP
composer install && ./vendor/bin/phpunit

# C/C++
mkdir -p build && cd build && cmake .. && make && ctest

# Full verification (customize per stack)
[build command] && [test command] && [lint command]
```

Every agent runs the build + test commands after completing their work. If either fails, the agent is not done.

---

## Territory Map (Default)

The default territory mapping. Override in your sprint's `DISPATCH.md` for project-specific paths.

| Agent | Default Territory |
|-------|-------------------|
| DATA | `models/`, `store/`, `repository/`, `db/`, `migrations/`, `prisma/`, `schema/` |
| BACKEND | `handlers/`, `routes/`, `services/`, `middleware/`, `controllers/`, `cmd/` |
| FRONTEND | `components/`, `pages/`, `routes/` (UI), `hooks/`, `stores/`, `views/` |
| SERVICES | `integrations/`, `workers/`, `modules/`, `jobs/`, `external/` |
| INFRA | `Dockerfile`, `docker-compose.yml`, `.github/`, `Makefile`, `config/`, `.env*` |
| QA | `test/`, `tests/`, `__tests__/`, `*_test.*`, `*.test.*`, `*.spec.*` |
| DESIGN | `design/`, `tokens/`, `theme/`, `styles/`, `.storybook/`, `style/` |
| RED TEAM | Read-only everywhere. Writes to: test files + findings report only |
| LEAD | `docs/`, `README.md`, `CHANGELOG.md`. Merge-only on all code |

See [guides/customization.md](../guides/customization.md) for stack-specific territory examples.

---

## Customization

### Adding Your Own Style Blocks

Create a `config/dispatch-styles-[project].md` for project-specific overrides. The dispatcher reads the base file first, then applies project overrides.

### Adjusting Intensity

Three intensity levels for different contexts:

**Maximum (default):**
```
I don't have time for your shit. BUILD IT RIGHT or I will END this session.
```

**Professional:**
```
This must be production-ready on first delivery. No placeholders. No TODOs.
Complete, working, tested code or a clear blockers report. Execute now.
```

**Collaborative:**
```
Quality is non-negotiable. Take the time to analyze before coding, match
conventions exactly, handle all edge cases, and verify your work compiles
and passes tests before reporting completion. Thorough > fast.
```

Choose the level that matches your operating style. The key principle is the same at every level: urgency eliminates half-finished output.

---

**Related Documents:**
- [templates/activation.md](../templates/activation.md) — Full activation prompt structure
- [templates/completion.md](../templates/completion.md) — Completion report template
- [guides/operators-guide.md](../guides/operators-guide.md) — How operators customize and dispatch
- [core/workflow.md](../core/workflow.md) — Sprint lifecycle
