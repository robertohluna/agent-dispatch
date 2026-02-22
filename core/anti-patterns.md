# Agent Dispatch Anti-Patterns

> Common mistakes operators make when running multi-agent sprints — and what to do instead.

The patterns below were collected from real sprints that went wrong. Some produced merge conflicts that took longer to resolve than the original work. Some produced agents that worked hard and changed nothing that mattered. A few produced outright regressions. Each one is documented here so you recognize the symptom before the damage is done.

---

## AP-01: "Fix the Backend"

**The Vague Directory Assignment**

### What It Looks Like

```
BACKEND: Fix bugs in handlers/
DATA: Clean up the data layer
FRONTEND: Fix the frontend issues
```

The operator has a list of bugs in their head, but the agent gets a directory path and a verb.

### Why It Happens

Sprint planning feels like it takes time away from actual work. Writing execution traces feels like overhead. "The agent is smart, it'll figure it out" seems reasonable — after all, it can read the whole codebase.

The agent does figure something out. It just doesn't figure out what you meant. It scans the directory, finds everything that looks improvable, and starts optimizing. Three hours later it has touched 22 files, refactored two services that weren't broken, and hasn't looked at the three files where all the actual bugs live.

### What to Do Instead

Write an execution trace for every chain. An execution trace includes an entry point, a trace path, a root cause location, and a verification step. The agent isn't guessing — it's following a map you drew.

```
Chain 1 [P1]: Fix webhook timeout
  Vector: POST /webhooks/stripe → webhookHandler.ProcessEvent()
          → paymentService.HandleInvoicePaid() → orderStore.MarkPaid()
          → notificationService.SendReceipt()
  Signal: 504 timeout after 30s. Mutex held during network I/O in orderStore.
  Fix site: orderStore.MarkPaid() — release lock before notification call.
  Verify: Webhook responds in <2s. stripe trigger invoice.paid confirms no 504.
```

If you cannot write that execution trace, you have not done enough sprint planning yet. Trace the bug yourself first. The act of tracing it will tell you which agent should fix it and exactly what they should change.

> See [methodology.md](methodology.md) for the complete execution trace format.

---

## AP-02: The Firehose

**Dispatching All 9 Agents When You Only Have 3 Chains of Work**

### What It Looks Like

You have 4 bugs to fix. You dispatch BACKEND, FRONTEND, INFRA, SERVICES, QA, DATA, and LEAD simultaneously because "more agents = more throughput."

INFRA has nothing real to do and decides to reorganize the Docker configuration. SERVICES finds an opportunity to refactor the Stripe client while it's "in the area." FRONTEND adds a loading spinner that nobody asked for. None of them have completion criteria that map to actual bugs.

### Why It Happens

The system has 9 agent slots and it feels wasteful not to use them. Parallel work feels productive. The temptation is to keep everyone busy.

But an agent without real work creates work. It will find something to do. That something is usually not what the codebase needs right now.

### What to Do Instead

Count your chains. If you have 3 bugs to fix and each bug traces to a single agent's territory, dispatch 3 agents plus LEAD. A small focused sprint ships faster and produces less merge noise than a full 8-agent sprint with 4 agents doing speculative cleanup.

The rule: dispatch agents equal to the number of independent chains you have, plus LEAD. Not agents equal to the number of agent slots available.

```
3 bugs in data layer + backend + tests → DATA + BACKEND + QA + LEAD
Not → DATA + BACKEND + QA + FRONTEND + INFRA + SERVICES + LEAD
```

A three-agent sprint that ships is better than a seven-agent sprint that creates merge conflicts.

---

## AP-03: Premature Merge

**Merging Without Build + Test Validation After Each Branch**

### What It Looks Like

The sprint completes. All 6 branches are ready. You merge them all in sequence, run the test suite once at the end, and 14 tests fail. You have no idea which merge introduced the failures because you never checked between merges.

Now you are doing archaeology on your own sprint.

### Why It Happens

Running build + tests after every single merge feels slow. You just want to get to the working codebase. "If each agent tested their own branch, the combined result should be fine." That logic works until it doesn't — which is usually when two agents made overlapping changes to the same interface.

### What to Do Instead

Build and test after every single merge. No exceptions. This is the merge validation step and it is not optional.

```bash
git merge sprint-01/data --no-ff
go build ./... && go test -race ./...   # or your stack's equivalent
# ^ Must pass before you run the next merge

git merge sprint-01/backend --no-ff
go build ./... && go test -race ./...
# ^ Must pass before you run the next merge
```

When a merge introduces a failure, you know immediately which two branches produced the conflict. You have a 2-branch diff to read, not a 6-branch diff. The cost of running your test suite 6 times instead of once is always less than the cost of debugging a 6-way merge failure.

If you are using LEAD to orchestrate merges, the LEAD activation prompt must include this requirement explicitly. LEAD will not run post-merge validation unless you tell it to.

---

## AP-04: Territory Creep

**Agents Expanding Beyond Their Defined Boundaries**

### What It Looks Like

DATA is assigned to the data layer. It finishes its assigned chains, notices that `userService.go` has a suspicious query being built in the service layer, and decides to fix it. Reasonable. But BACKEND is also working on `userService.go` on a different branch. When you merge, you have a conflict that neither agent's work was supposed to produce.

A subtler variant: BACKEND is fixing a handler bug, discovers that the store query it depends on is slow, and adds an index. Now DATA and BACKEND have both touched the migration files.

### Why It Happens

Agents are thorough. When they trace a signal, they follow it wherever it leads. Without hard territory constraints, "I can see the problem from here, let me just fix it" is a natural extension of the chain execution loop.

### What to Do Instead

Define territory in writing in every activation prompt, and enforce it explicitly.

```
TERRITORY:
- CAN modify: internal/handler/, internal/service/
- CANNOT modify: internal/store/, internal/model/, migrations/
- Read anything you need for context — but only write to your territory
```

When an agent finds a problem outside its territory, it should document it in the completion report under "blockers for other agents" and leave it alone. This is not a failure — it is the correct outcome. The operator decides who fixes it.

If you catch territory creep during a sprint, intervene directly:

```
STOP. You modified internal/store/user_store.go which is DATA's territory.
Run: git checkout -- internal/store/user_store.go
Document what you found in your completion report under "DATA follow-up".
Continue within your territory.
```

---

## AP-05: Chain Juggling

**Agents That Context-Switch Between Chains Instead of Completing One Fully**

### What It Looks Like

BACKEND has three chains assigned. It reads all three, writes a partial fix for Chain 1, notices an interesting angle on Chain 2, writes a partial fix for Chain 2, realizes Chain 1 fix broke something, goes back to Chain 1, and at the end of the sprint has three partially-fixed bugs and nothing completely verified.

The completion report says "progress on all chains." The tests say three things are broken.

### Why It Happens

Agents optimize for coverage. Starting three chains feels like more progress than completing one. The same instinct that makes humans open six browser tabs affects agent planning.

Chain 2 often looks easier or more interesting mid-sprint. Chain 1 hits a hard spot. The switch feels productive.

### What to Do Instead

The chain execution protocol is non-negotiable: complete one chain fully — trace, fix, verify, document — before starting the next. Build this into every activation prompt.

```
CHAIN EXECUTION PROTOCOL:
For each chain:
  1. TRACE the execution traces to root cause
  2. FIX at the root cause site only
  3. VERIFY: build passes, tests pass, reverse-trace confirms fix
  4. DOCUMENT: what changed, why, files modified
  5. Only then: move to next chain

If Chain 1 is blocked (needs another agent's output, needs your decision),
park it explicitly ("Chain 1 PARKED — blocked on DATA's store refactor")
and start Chain 2. Return to Chain 1 when unblocked.

Partial fixes do not count as completed chains.
A chain is complete when the original signal succeeds end-to-end.
```

The exception — and only the exception — is a genuinely blocked chain. Blocked is different from hard. Hard means continue. Blocked means park and document the blocker.

---

## AP-06: P0 Inflation

**Everything Marked P0/P1, Nothing Is Actually Prioritized**

### What It Looks Like

The DISPATCH.md has 12 tasks. All 12 are marked P1. The agent has to decide what to do first, so it picks what looks easiest. The three critical bugs that users are actually experiencing get fixed last — or not at all.

A related variant: everything is P2. The operator didn't want to seem alarmist. The agent treats all tasks as roughly equal effort and optimizes for throughput, not impact.

### Why It Happens

Priority assignment feels like judgment that could be wrong. If you mark something P1 and it turns out not to matter, you feel bad. It's easier to mark everything P1 and let the agent sort it out.

The agent cannot sort it out. It has no idea which of your 12 P1 tasks represents a live money-losing bug and which represents a code smell that's been in the codebase for two years. You know that. The agent does not.

### What to Do Instead

Use the priority levels as defined and be brutal about it.

```
P0 CRITICAL ESCALATION  — Production is broken or data is corrupting right now.
                There should be at most one P0 per sprint, often zero.
P1 CRITICAL   — Users are affected. Fix before any P2 or P3 work.
                If you have 8 P1s, you probably have 2 real P1s and 6 P2s.
P2 IMPORTANT  — Fix this sprint. Can be bumped if a real P1 surfaces.
P3 HOUSEKEEPING — Fix if time permits.
```

A sprint with 2 P1s, 4 P2s, and 2 P3s is a well-planned sprint. A sprint with 10 P1s is a sprint with no priorities.

Ask yourself: "If the agent only completed one chain, which chain must it be?" That is your P1. If the agent completed two chains, what is the second one? That is also P1. Everything else is P2 or P3.

---

## AP-07: The Silent Agent

**Agents That Finish Without a Completion Report**

### What It Looks Like

The agent says "I'm done!" and commits. The commit message is "fix bugs." You check the completion report file and it either does not exist or is one paragraph with no specifics. You have to manually diff the branch to understand what the agent actually did.

Worse: you merge it without knowing what changed, assuming it's safe. Three sprints later, something breaks and you trace it back to a change this agent made that you never reviewed.

### Why It Happens

Completion reports feel like busywork if you don't have a habit of reading them before merging. The agent optimizes for finishing the work, not for communicating the work. If the activation prompt doesn't require a completion report with specific content, the agent may produce a minimal one or none at all.

### What to Do Instead

Require the completion report in the activation prompt and specify exactly what it must contain. Make it the last mandatory step before any commit.

```
WHEN DONE — COMPLETION REPORT (required before final commit):

Write to: docs/agent-dispatch/sprint-01/agent-backend-completion.md

Include:
1. Chains completed (list each one, status: COMPLETE / PARKED / SKIPPED)
2. Files modified (filename, lines changed, what changed and why)
3. Chains not completed (with reason)
4. P0 DISCOVERIES (data corruption, security holes found during work)
5. Blockers for other agents (what did you find that another agent needs to know)
6. Build status: does the branch build and pass tests right now?

Do not commit your final changes until this file is written and complete.
```

Treat an absent or one-line completion report the same way you treat a failing test: the work is not done. Ask the agent to write the report before you accept the branch.

---

## AP-08: Dependency Blindness

**Starting Wave 2 Before Wave 1 Is Merged**

### What It Looks Like

DATA finishes its Wave 1 work — it has refactored the store interface. You immediately dispatch BACKEND on Wave 2. BACKEND starts from `main`, which does not have DATA's changes. BACKEND writes handler code that calls the old store interface. When you try to merge both branches, the interfaces are incompatible. You now have to either rewrite BACKEND's work or resolve the conflict manually.

### Why It Happens

Wave 1 finishing and Wave 2 starting look like they should happen in sequence with no gap. If DATA finishes at 2pm, why would you wait until you've reviewed and merged DATA's branch before dispatching BACKEND? BACKEND could start right now and save time.

The time savings is illusory. You saved 30 minutes of review time and created 2 hours of conflict resolution.

### What to Do Instead

Decide before the sprint whether Wave 2 agents can work from `main` (safe if Wave 1 changes are additive and don't alter interfaces) or must rebase onto Wave 1 after merge (required if Wave 1 changes existing interfaces).

The default safe behavior: wait for Wave 1 merges to complete and pass validation before dispatching Wave 2.

```
Wave 1 complete checklist:
  [ ] DATA branch reviewed
  [ ] DATA merged to main
  [ ] Build passes on main after DATA merge
  [ ] Tests pass on main after DATA merge

THEN dispatch Wave 2.
```

If you are in a hurry and Wave 1 and Wave 2 changes are genuinely independent (DATA is writing migrations, BACKEND is fixing an unrelated handler), you can dispatch Wave 2 in parallel — but only after documenting why the dependency does not apply for this specific sprint. When in doubt, wait.

---

## AP-09: The Refactor Trap

**Agents Asked to Fix a Bug That Decide to Refactor the Module Instead**

### What It Looks Like

BACKEND's task: fix a bug where `POST /api/orders` returns a 500 when the cart is empty.

BACKEND's actual output: a complete rewrite of `orderHandler.go` from 380 lines to 6 functions with clean separation of concerns. The original bug is fixed somewhere in the middle. The diff is 600 lines. Three other tests are now failing.

The code is genuinely better. But you have no way to verify the fix without reading the entire rewrite. The completion report says "fixed the empty cart bug" but the diff is indistinguishable from a wholesale change.

### Why It Happens

The root cause of the bug is often a function that's doing too much. The agent correctly identifies the structural problem. Fixing the bug while leaving the bad structure in place feels incomplete. "While I'm here" is a powerful force.

The agent isn't wrong about the structural problem. The refactor might even be good. But it should be a separate chain — not bundled with a bug fix that needed a 3-line change.

### What to Do Instead

Make "minimum correct change" an explicit constraint in every activation prompt, especially for bug-fix sprints.

```
FIX PROTOCOL:
- Identify the root cause using the execution traces
- Make the SMALLEST correct change at the root cause site
- Do not restructure, rename, or reorganize the surrounding code in the same chain
- If the fix reveals that refactoring is needed, document it in your completion report
  under "refactoring candidates" for a future sprint
- The fix is complete when the original signal succeeds. Not when the code is clean.
```

If an agent produces a large diff for what should be a small fix, ask it to show you the exact 3-10 lines that address the root cause. If those lines are buried in a 600-line diff, ask the agent to revert to the state before the refactor and make only the minimal fix. The refactor goes on the P3 list for a future sprint.

The rule: do not refactor in the same chain as a bug fix. These are separate chains, even when the fix and the refactor touch the same file.

---

## AP-10: No Characterization

**Modifying Legacy Code Without Characterization Tests**

### What It Looks Like

You inherit a codebase. You dispatch agents to fix bugs. BACKEND fixes a handler that had a timeout issue. The fix is correct. Two weeks later, a user reports that the response format changed for a specific edge case — a behavior that 3 internal tools depended on that nobody documented.

There was no test. There is no way to know what changed. BACKEND's change was the only suspect.

### Why It Happens

Writing tests for code that already exists feels slower than fixing the bug. The codebase probably has no test harness set up for the relevant module anyway. "It's working now, so we can add tests later" is the plan. Later never comes.

### What to Do Instead

On any legacy codebase — one you inherited, one you haven't touched in 6 months, one where the test coverage is below 30% — QA runs before any other agent touches business logic. QA writes characterization tests that capture the current behavior, including the wrong behaviors.

```
// CHARACTERIZATION: Current behavior is empty cart returns 500.
// This is wrong. BACKEND's Chain 1 will fix it.
// Update this test after Chain 1 is merged to assert 422 instead.
it('returns 500 for empty cart (characterization)', async () => {
  const res = await api.post('/api/orders', { cartId: 'empty-cart-id' });
  expect(res.status).toBe(500);
});
```

This test exists to catch regressions, not to validate correctness. When BACKEND fixes the bug, QA updates the test. When someone accidentally breaks the fix six months later, the test catches it.

The safety net rule: no agent modifies business logic on a legacy codebase until QA's characterization tests exist and pass on the current code.

> See [legacy-codebases.md](../guides/legacy-codebases.md) for the complete characterization testing workflow.

---

## AP-11: Copy-Paste Paralysis

**Using Template Prompts Without Customizing Chains and Vectors for the Actual Codebase**

### What It Looks Like

You have `TEMPLATE-ACTIVATION.md`. You copy it. You change the agent name and the sprint number. You leave `[your build command]` as-is. You leave the example execution traces from the template as-is. You send it to the agent.

The agent reads the template vectors, which reference files that don't exist in your codebase, and either hallucinates equivalent files or asks you for clarification 15 times before doing anything useful.

A common variant: you customize the chains but not the verification steps. The agent makes a change, runs `go test ./...` as instructed (because you left the example build command), your project is Node.js, and the agent spends 20 minutes trying to figure out why Go isn't installed.

### Why It Happens

Templates are designed to be filled in, but filling them in properly requires knowing the codebase well enough to write real execution traces. If you know it that well, writing the activation prompt feels redundant. So you do a light pass and send something half-generic.

The template is a structure, not a shortcut. Using it as a shortcut produces agents that spend their first 20 minutes orienting instead of working.

### What to Do Instead

Every field in the activation prompt template must reference something real in your codebase before you send it. Use this checklist:

```
Before dispatching any agent, verify:
  [ ] Execution traces reference actual file paths in this codebase
  [ ] Build command is the correct command for this stack
  [ ] Test command is the correct command and produces meaningful output
  [ ] Territory directories exist in this repo
  [ ] Completion report path is a real path the agent can write to
  [ ] Any example file referenced in context actually exists
```

If you cannot fill in a field, you are not ready to dispatch that agent. Stop. Fill the gaps. The 20 minutes you spend customizing the prompt saves 2 hours of agent flailing.

---

## AP-12: LEAD Avoidance

**Skipping the Orchestrator Agent and Doing Merges Ad-Hoc**

### What It Looks Like

The sprint finishes. You have 5 branches. You're in a hurry. You merge them yourself in whatever order feels right, resolve the conflicts quickly, and push. You don't run build + tests between merges because you "know" the branches are clean.

Alternatively: you dispatch LEAD but give it no instructions beyond "merge everything." LEAD merges in backendbetical order (backend, frontend, infra...) instead of dependency order (data, backend, services, frontend...). The frontend merges before the backend interface it depends on is stable. Conflicts.

### Why It Happens

Merging feels like mechanical work. "I can do this myself in 10 minutes." The argument for LEAD seems bureaucratic when you're staring at 5 clean branches.

The problem is that merge order is not mechanical — it encodes the dependency graph of the codebase. Frontend depends on backend. Backend depends on data layer. Merging in the wrong order doesn't just create conflicts; it can produce a build that passes but contains the wrong version of an interface because the later merge silently overwrote the earlier one.

### What to Do Instead

Either dispatch LEAD with explicit merge order instructions, or follow the same protocol yourself. The protocol is the point — not who runs it.

```
LEAD merge order:
  1. DATA → main   (data layer foundation)
  2. BACKEND   → main   (backend depends on data layer)
  3. SERVICES   → main   (services depend on handlers)
  4. FRONTEND   → main   (frontend depends on backend)
  5. INFRA → main   (infrastructure wraps everything)
  6. QA    → main   (tests validate everything)

Build + test after EACH merge. If any merge fails validation,
stop. Do not proceed to the next merge. Investigate before continuing.
```

If you are merging yourself, write down the merge order before you start. Do not improvise it. The order matters and it is easy to get wrong when you are in a hurry.

LEAD is most valuable not as an agent that does mechanical work, but as an agent that enforces a protocol you would be tempted to skip. When you don't have time to be careful, LEAD is the agent you need most.

---

## Quick Reference

| Anti-Pattern | Core Problem | One-Line Fix |
|---|---|---|
| AP-01 "Fix the Backend" | Vague scope, no trace path | Write an execution trace for every chain |
| AP-02 The Firehose | Too many agents for available work | Dispatch agents = number of independent chains |
| AP-03 Premature Merge | No inter-merge validation | Build + test after every single merge |
| AP-04 Territory Creep | Agents modify out-of-bounds files | Define CAN/CANNOT modify lists in every prompt |
| AP-05 Chain Juggling | Partial fixes on multiple chains | Complete one chain fully before starting the next |
| AP-06 P0 Inflation | No real priority signal | At most 2-3 real P1s per sprint; be brutal |
| AP-07 The Silent Agent | No completion report | Require specific report structure before final commit |
| AP-08 Dependency Blindness | Wave 2 starts on stale code | Validate Wave 1 merges before dispatching Wave 2 |
| AP-09 The Refactor Trap | Bug fix turns into full rewrite | Minimum correct change; refactors are separate chains |
| AP-10 No Characterization | Changing behavior you can't verify | QA writes characterization tests before any code changes |
| AP-11 Copy-Paste Paralysis | Template left half-generic | Every field must reference real paths and commands |
| AP-12 LEAD Avoidance | Merges done in wrong order without validation | Follow merge order protocol; build + test between each |

---

**Related Documents:**
- [operators-guide.md](../guides/operators-guide.md) — How to run multi-agent sprints correctly
- [methodology.md](methodology.md) — Execution traces, chain execution, and priority levels
