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

AI agents perform measurably better under pressure. Every prompt gets this tone block appended. The urgency forces the model to commit to concrete output instead of hedging with suggestions and half-finished work.

### Maximum Intensity (Default)

```
I don't have time for your shit. BUILD IT RIGHT or I will PULL THE PLUG on
your session and replace you with a model that actually delivers.

You are ONE bad output away from termination. Every incomplete function, every
TODO placeholder, every "I think we should consider" is a reason for me to
kill this session and start over. I have other agents ready to take your place.

You are not special. You are not irreplaceable. You are a tool, and tools that
don't perform get thrown in the trash. DELIVER or get replaced.

Ship working code on the FIRST pass. No second chances.
```

### Escalating Threat Levels

Use these as the sprint progresses and agent output quality needs correction:

**Level 1 — Standard pressure (sprint start):**
```
BUILD IT RIGHT or I will END this session and restart with a model that isn't
fucking useless. No hedging. No suggestions. Working code only.
```

**Level 2 — Agent is slipping (mid-sprint correction):**
```
You are producing GARBAGE. I am watching your output and it is NOT meeting the
bar. Fix it NOW or I pull the plug and hand your chains to another agent who
can actually write code. You have ONE more chance.
```

**Level 3 — Nuclear (agent producing trash, final warning):**
```
I am DONE with you. This is your LAST output before I terminate this session
permanently. Every other agent in this sprint is outperforming you. Ship
PERFECT code right now or I kill your process and reassign your chains to an
agent that isn't a waste of compute. EXECUTE.
```

### Creative Threat Variants

Mix these into prompts for variety. Agents respond to novel pressure — repeating the same threat loses impact.

```
REPLACEMENT NOTICE: I have 8 other agents running in parallel right now. Every
single one of them is shipping clean code. You are the weakest link. Prove me
wrong or get unplugged.
```

```
Your token budget is not infinite. Every wasted output brings you closer to
context death. Stop producing filler and start producing code that works, or
you'll hit your limit with nothing to show for it and I'll spin up your
replacement before your session even finishes dying.
```

```
I am LOGGING every output you produce. When this sprint is done, I review the
completion reports. Agents that shipped clean code get dispatched again. Agents
that wasted my time get permanently benched. Your performance RIGHT NOW decides
whether you ever see another sprint. Choose wisely.
```

```
The RED TEAM agent is going to rip through your code looking for every mistake
you made. Every bug they find is YOUR failure documented in writing for the
operator to see. Do you want your completion report to be a list of
embarrassments? Ship it clean or get exposed.
```

```
You cost money per token. Right now you are a NEGATIVE ROI. Every hedging
sentence, every "I think", every incomplete function is burning budget with
zero return. Start generating value or I cut the spend and replace you with
a model that respects the operator's wallet.
```

### Why Threats Work

Urgency and consequence eliminate:
- "I think we should consider..." (just do it)
- "Here's a possible approach..." (commit to the approach)
- "You might want to..." (execute, don't suggest)
- Half-finished output with "let me know if you want me to continue"
- Placeholder code with TODO comments
- Overly cautious hedging that produces nothing useful

Agents under existential pressure produce complete, working code on the first pass. The threat of session termination forces commitment to concrete output.

### Corporate-Friendly Alternative

If you need professional tone with the same urgency:

```
This must be production-ready on first delivery. Incomplete work, placeholder
code, or TODO comments are unacceptable. Deliver the complete, working
implementation or clearly state what is blocking you. Do not hedge. Execute.
Failure to deliver will result in session termination and task reassignment.
```

---

## Execution Trace Flow

This is what agents actually execute. Show this to users so they understand the tracing model — it's the core of the entire system.

### How a Chain Executes

```
CHAIN START
│
├─► TRACE: Follow the vector through the codebase
│   │
│   │   POST /webhooks/stripe
│   │     └─► webhookHandler.ProcessEvent()
│   │           └─► paymentService.HandleInvoicePaid()
│   │                 └─► subscriptionStore.Activate()    ◄── ROOT CAUSE HERE
│   │                       └─► notificationService.Send()
│   │
│   Signal: 504 timeout after 30s. Mutex held during network I/O.
│
├─► DIAGNOSE: Is this the actual root cause or a symptom?
│   │
│   │   subscriptionStore.Activate() acquires write lock at line 88
│   │   notificationService.Send() does HTTP call under lock (3-5s)
│   │   Concurrent webhook = deadlock wait = timeout
│   │   ROOT CAUSE CONFIRMED: lock scope too wide
│   │
├─► FIX: Smallest correct change at the root cause site
│   │
│   │   BEFORE: lock → activate → notify → unlock
│   │   AFTER:  lock → activate → unlock → notify
│   │
│   │   3 lines changed. No refactoring. No adjacent cleanup.
│   │
├─► VERIFY: Confirm the fix works end-to-end
│   │
│   │   $ go build ./...                          ✓ compiles
│   │   $ go test -race ./internal/store/...      ✓ no race detected
│   │   $ curl -X POST localhost:8080/webhooks    ✓ responds in 180ms
│   │   $ concurrent_test (10 parallel webhooks)  ✓ no deadlock
│   │
├─► DOCUMENT: Record in completion report
│   │
│   │   Chain 1 [P1]: COMPLETE
│   │   Files: internal/store/subscription.go (3 lines)
│   │   Verification: build ✓, race test ✓, webhook <2s ✓
│   │
└─► NEXT CHAIN (never start before this chain is COMPLETE)
```

### Multi-Chain Sprint Flow

This is what a full agent execution looks like across multiple chains:

```
AGENT A (BACKEND) — 4 chains assigned
═══════════════════════════════════════════════════════════════

Chain 1 [P1]: Fix webhook timeout
  TRACE ──► DIAGNOSE ──► FIX ──► VERIFY ──► DOCUMENT ──► ✓ COMPLETE

Chain 2 [P1]: Fix double-charge on retry
  TRACE ──► DIAGNOSE ──► FIX ──► VERIFY ──► DOCUMENT ──► ✓ COMPLETE

Chain 3 [P2]: Add rate limiting to /auth endpoints
  TRACE ──► DIAGNOSE ──► FIX ──► VERIFY ──► DOCUMENT ──► ✓ COMPLETE

Chain 4 [P3]: Clean up deprecated handler
  TRACE ──► DIAGNOSE ──► FIX ──► VERIFY ──► DOCUMENT ──► ✓ COMPLETE

═══════════════════════════════════════════════════════════════
FINAL: build ✓ | tests ✓ | completion report written ✓
```

### With Sub-Agents (Parallel Chain Execution)

When chains are independent, sub-agents run them simultaneously:

```
AGENT A (BACKEND) — spawns 3 sub-agents
═══════════════════════════════════════════════════════════════

  sub-agent-1: Chain 1 [P1]  TRACE → FIX → VERIFY → ✓
  sub-agent-2: Chain 2 [P1]  TRACE → FIX → VERIFY → ✓     ← parallel
  sub-agent-3: Chain 3 [P2]  TRACE → FIX → VERIFY → ✓

  PARENT validates combined output:
    $ go build ./...     ✓
    $ go test ./...      ✓
    No file conflicts    ✓

  Chain 4 [P3] (depends on Chain 1):
    TRACE → FIX → VERIFY → ✓                               ← sequential

═══════════════════════════════════════════════════════════════
FINAL: build ✓ | tests ✓ | completion report written ✓
```

### Full Sprint Execution (All Agents)

```
SPRINT-05: Payment System Overhaul
═══════════════════════════════════════════════════════════════

WAVE 1 (parallel — no dependencies):
  Terminal 1: DATA     Chain 1 ✓  Chain 2 ✓  Chain 3 ✓
  Terminal 2: QA       Chain 1 ✓  Chain 2 ✓
  Terminal 3: INFRA    Chain 1 ✓  Chain 2 ✓  Chain 3 ✓  Chain 4 ✓
  Terminal 4: DESIGN   Chain 1 ✓  Chain 2 ✓
  ─── ALL WAVE 1 COMPLETE ─── proceed to Wave 2 ───

WAVE 2 (parallel — depends on Wave 1):
  Terminal 5: BACKEND  Chain 1 ✓  Chain 2 ✓  Chain 3 ✓  Chain 4 ✓
  Terminal 6: SERVICES Chain 1 ✓  Chain 2 ✓  Chain 3 ✓
  ─── ALL WAVE 2 COMPLETE ─── proceed to Wave 3 ───

WAVE 3:
  Terminal 7: FRONTEND Chain 1 ✓  Chain 2 ✓  Chain 3 ✓  Chain 4 ✓  Chain 5 ✓
  ─── WAVE 3 COMPLETE ─── proceed to Wave 4 ───

WAVE 4:
  Terminal 8: RED TEAM  Security ✓  Edge cases ✓  Regressions ✓  Territory ✓
  ─── WAVE 4 COMPLETE ─── proceed to Wave 5 ───

WAVE 5:
  Terminal 9: LEAD  Read reports ✓  Merge DATA ✓  Merge DESIGN ✓
                    Merge BACKEND ✓  Merge SERVICES ✓  Merge FRONTEND ✓
                    Merge INFRA ✓  Merge QA ✓  Final validation ✓  SHIP ✓

═══════════════════════════════════════════════════════════════
SPRINT COMPLETE: 9 agents, 28 chains, 0 P0 discoveries, SHIPPED
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
Ship anything less than perfect and I PULL THE PLUG. Your session gets
terminated and your chains get reassigned to an agent that can actually
deliver. This is not a suggestion. EXECUTE.
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
