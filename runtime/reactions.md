# Runtime Reactions

> Decision trees for every in-sprint event. When something happens, look it up here.

## When to Use This Guide

You're mid-sprint. Agents are running. Something happens that wasn't in the plan. This guide gives you a decision tree for the 12 most common runtime events.

Each reaction follows the same format:
- **SYMPTOM** — What you observe
- **DIAGNOSIS** — What's actually happening
- **ACTION** — What to do (step by step)
- **VERIFY** — How to confirm the reaction worked

## Escalation Timer Framework

Not every issue needs immediate intervention. Use escalation timers to avoid micromanaging:

| Timer | Trigger | Action |
|-------|---------|--------|
| **5 min** | Agent shows unusual behavior | Note it. Keep watching. Most issues self-resolve. |
| **15 min** | Agent hasn't progressed or issue persists | Intervene. Send a correction message. |
| **30 min** | Agent still stuck after intervention | Reassign chains. Restart agent if needed. |

**Default response:** Wait 5 minutes before intervening. Agents often get unstuck on their own.

---

## R-01: CI Fails on Agent Branch

**SYMPTOM:** CI pipeline fails on an agent's branch. Build errors or test failures in CI.

**DIAGNOSIS:**
- Agent made changes that break the build
- OR agent's branch is behind main and has stale dependencies
- OR CI environment differs from local (missing env vars, different runtime)

**ACTION:**
1. Check CI logs — is it a code error or an environment issue?
2. If code error: message the agent with the exact failure output
   ```
   CI failed on your branch. Error: [paste exact error]
   Fix this before continuing to your next chain.
   ```
3. If environment issue: fix CI config yourself (INFRA territory) or dispatch to INFRA agent
4. If branch is stale: rebase agent's branch onto latest main

**VERIFY:** CI passes on the agent's branch after fix.

---

## R-02: Agent Stuck in Loop

**SYMPTOM:** Agent is repeating the same action (editing, reverting, re-editing the same file). Making no meaningful progress for 15+ minutes.

**DIAGNOSIS:**
- Agent is confused about the root cause
- OR the execution trace in the task doc is wrong/incomplete
- OR the agent is fighting a constraint it can't solve (e.g., needs to modify a file outside territory)

**ACTION:**
1. Read the agent's recent output — what is it trying to do?
2. If trace is wrong: provide the correct trace
   ```
   STOP. The root cause is not in [file]. Look at [correct file:line].
   The actual issue is [description]. Make ONLY that change.
   ```
3. If territory conflict: either expand territory or reassign the chain to the correct agent
4. If fundamentally stuck: park the chain, move to next chain
   ```
   PARK Chain [N]. Move to Chain [N+1]. We'll revisit this chain after the sprint.
   ```

**VERIFY:** Agent makes progress on a chain within 5 minutes of intervention.

---

## R-03: Agent Crosses Territory

**SYMPTOM:** Agent modifies files outside its assigned territory.

**DIAGNOSIS:**
- Agent's chain traced into another agent's territory (common — execution traces cross boundaries)
- OR agent is scope-creeping (fixing things it noticed but weren't assigned)
- OR territory boundaries were unclear in the activation prompt

**ACTION:**
1. Check if the change is related to an assigned chain or is scope creep
2. If chain-related:
   ```
   HOLD. You modified [file] which is in [OTHER-AGENT]'s territory.
   Revert: git checkout -- [file]
   Document this as a blocker in your completion report:
   "Chain [N] requires change to [file] — assign to [OTHER-AGENT]."
   Continue with your remaining chains.
   ```
3. If scope creep:
   ```
   REVERT changes to [file]. That's outside your territory and not in your chains.
   Focus on your assigned chains only.
   ```

**VERIFY:** `git diff --name-only` shows only files within agent's territory.

---

## R-04: Review Feedback Arrives Mid-Sprint

**SYMPTOM:** PR review comments or external feedback arrives while agents are still working.

**DIAGNOSIS:**
- Feedback might affect in-progress chains
- OR feedback is for already-completed chains (needs rework)
- OR feedback is for a different sprint (ignore for now)

**ACTION:**
1. Assess: Does this feedback affect currently running agents?
2. If YES and agent hasn't started the affected chain:
   - Update the chain in the agent's task doc
   - Message the agent: "Chain [N] has been updated. Read the new requirements before starting it."
3. If YES and agent already completed the affected chain:
   - Add a new chain at the end of the agent's task list
   - Message: "New chain added: address review feedback on [topic]. Execute after current chains."
4. If NO — feedback is for a different area: park it for the next sprint

**VERIFY:** Agent acknowledges the updated chain and incorporates feedback.

---

## R-05: Build Breaks on Main

**SYMPTOM:** `main` branch no longer builds or tests fail on `main`.

**DIAGNOSIS:**
- A merge introduced a conflict or regression
- OR an external dependency changed (upstream API, package registry)
- OR a merge happened without post-merge validation

**ACTION:**
1. **STOP all merges immediately.** Do not merge more agent branches until main is fixed.
2. Identify the breaking merge: `git log --oneline main -5` — which was the last merge?
3. Options:
   - **Quick fix:** If the issue is obvious, fix it directly on main
   - **Revert:** `git revert [merge-commit]` — revert the breaking merge, fix the agent's branch, re-merge
   - **Dispatch:** Send the fix to the appropriate agent as a new P1 chain
4. Sprint health → RED until main is green again

**VERIFY:** `[build command] && [test command]` passes on main.

---

## R-06: Agent Reports P0 Discovery

**SYMPTOM:** Agent writes a P0 DISCOVERY in their completion report or flags a critical issue.

**DIAGNOSIS:**
- Agent found a critical issue (security vulnerability, data corruption, race condition) during their work
- This issue was NOT in their assigned chains — they discovered it while tracing

**ACTION:**
1. **Read the P0 report immediately.** Understand the severity.
2. Decide:
   - **Fix now:** Assign a new P1 chain to the appropriate agent (may be a different agent than the discoverer)
   - **Fix later:** If the P0 doesn't affect the current sprint's chains, document it and fix in next sprint
   - **Stop the sprint:** If the P0 is severe enough that continuing would make things worse
3. If fixing now: create a new chain for the responsible agent
   ```
   NEW P1 CHAIN: [P0 discovery title]
   Vector: [trace from the P0 report]
   Signal: [description]
   Fix: [assigned territory]
   ```

**VERIFY:** P0 is either fixed (verify with test) or documented with a clear owner for next sprint.

---

## R-07: Agent Finishes Early

**SYMPTOM:** Agent completes all assigned chains ahead of other agents in the same wave.

**DIAGNOSIS:**
- Chains were simpler than estimated
- OR agent worked faster than expected
- This is an opportunity, not a problem

**ACTION:**
1. Review the completion report — did the agent actually verify all chains, or did it skip verification?
2. If work is genuinely complete:
   - Check if there are PARKED chains from other agents that this agent could pick up
   - Check if there are P3 chains that were deprioritized
   - If nothing to reassign: let the agent rest. Don't make work.
   ```
   Good work. All chains verified. If you have capacity:
   [P3 chain or parked chain from another agent within your territory]
   Otherwise, commit your completion report and you're done.
   ```
3. If verification was skipped:
   ```
   Hold. Your completion report shows Chain 2 has no verification step.
   Go back and verify: [specific verification instruction].
   ```

**VERIFY:** Agent's branch has clean build + test pass. Completion report is thorough.

---

## R-08: Agent Discovers Blocked Chain

**SYMPTOM:** Agent reports a chain is blocked — it depends on another agent's work or needs external input.

**DIAGNOSIS:**
- Chain traces into another agent's territory (cross-dependency)
- OR chain needs information that doesn't exist yet (API spec, design decision)
- OR chain depends on a Wave 1 change that hasn't been merged yet

**ACTION:**
1. Have the agent PARK the blocked chain and move to the next one
   ```
   PARK Chain [N]. Document the blocker:
   "Blocked: needs [specific thing] from [AGENT/PERSON]."
   Continue with Chain [N+1].
   ```
2. Route the blocker:
   - If it's another agent's work: message that agent or add it to their chain list
   - If it's external: get the answer yourself and update the agent
   - If it's a wave dependency: it'll resolve when the earlier wave merges
3. After blocker resolves: have the agent return to the parked chain

**VERIFY:** Agent moves to next chain. Blocker is documented and routed.

---

## R-09: Agent Produces Bad Output

**SYMPTOM:** Agent's changes are wrong — incorrect logic, breaks existing behavior, doesn't match the chain requirement, or makes things worse.

**DIAGNOSIS:**
- Agent misunderstood the execution trace
- OR the trace was wrong/incomplete
- OR agent fixed a symptom instead of the root cause

**ACTION:**
1. Be specific about what's wrong:
   ```
   STOP. Your fix for Chain [N] is incorrect.
   Problem: [what's wrong with the agent's change]
   The actual root cause is at [file:line]: [explanation]
   Revert your change: git checkout -- [files]
   Re-read the vector trace and try again from [specific starting point].
   ```
2. If the agent continues to produce bad output after 2 corrections:
   - Provide the exact fix (specific lines to change)
   - Or reassign the chain to a different agent
3. Document in sprint notes for retrospective

**VERIFY:** Agent's revised fix is correct. Build + tests pass. Chain is properly verified.

---

## R-10: Agent Crashes or Disconnects

**SYMPTOM:** Agent terminal closes, session dies, or agent becomes unresponsive.

**DIAGNOSIS:**
- Context window overflow (too much code read)
- OR API rate limit / timeout
- OR network issue
- OR the tool crashed

**ACTION:**
1. Check the agent's branch — what was committed?
   ```bash
   git -C /path/to/worktree log --oneline -5
   git -C /path/to/worktree status
   ```
2. Assess progress: Which chains were completed? What's in progress?
3. Restart the agent in the same worktree with a recovery prompt:
   ```
   You are [CODENAME] agent resuming Sprint [XX].
   Your branch: sprint-[XX]/[agent-name]

   PREVIOUSLY COMPLETED:
   - Chain 1: [COMPLETE — summarize what was done]
   - Chain 2: [IN PROGRESS — summarize where it left off]

   RESUME FROM: Chain 2, step [X].
   [Include the original territory and protocol sections]
   ```
4. If data was lost (uncommitted changes): re-dispatch from the last clean commit

**VERIFY:** Restarted agent picks up where it left off. No duplicate work.

---

## R-11: Two Agents Modify Same File

**SYMPTOM:** During merge, two branches have changes to the same file.

**DIAGNOSIS:**
- Territory overlap (shouldn't happen if territories are clean)
- OR shared file (package.json, go.mod, requirements.txt) modified by multiple agents
- OR one agent crossed territory

**ACTION:**
1. If shared file (dependency file):
   - Merge both sets of changes (usually additive — both added dependencies)
   - Run `[install command]` to validate
2. If territory overlap:
   - Earlier merge order wins (DATA > DESIGN > BACKEND > SERVICES > etc.)
   - Review the later agent's change — if it's clearly better, keep it instead
3. If one agent crossed territory:
   - Keep the territorial owner's version
   - Check if the crossing agent's change needs to be redone by the correct agent

**VERIFY:** File has correct combined changes. Build + tests pass after merge.

---

## R-12: Sprint Scope Change Mid-Sprint

**SYMPTOM:** New urgent work arrives, priorities shift, or a critical bug is reported while agents are running.

**DIAGNOSIS:**
- External pressure (production incident, stakeholder request)
- OR P0 discovery that changes the sprint focus
- OR original scope was wrong

**ACTION:**
1. **Don't panic.** Finish the current wave before changing scope.
2. Assess the new work:
   - Can it wait for this sprint to finish? → Add to next sprint
   - Is it P0/P1 and can't wait? → Continue below
3. If scope change is needed:
   - Stop dispatching new waves
   - Let current agents finish their in-progress chains (don't interrupt mid-chain)
   - Add new chains to the appropriate agent's task list
   - Or create a new agent for the new work (if it's a new territory)
4. Document the scope change in sprint notes

**VERIFY:** New work is assigned. In-progress work isn't disrupted. Sprint health updated.

---

## Quick Reference

| # | Event | Severity | Default Timer | Key Action |
|---|-------|----------|--------------|------------|
| R-01 | CI fails | YELLOW | 5 min | Send error output to agent |
| R-02 | Agent stuck | YELLOW | 15 min | Provide correct trace or park chain |
| R-03 | Territory crossing | YELLOW | Immediate | Revert + document as blocker |
| R-04 | Review feedback | GREEN | Next chain | Update task doc or add new chain |
| R-05 | Build breaks main | RED | Immediate | Stop merges. Fix or revert. |
| R-06 | P0 discovery | RED | Immediate | Read, decide: fix now or park |
| R-07 | Agent early finish | GREEN | — | Reassign parked chains or P3 work |
| R-08 | Blocked chain | YELLOW | Immediate | Park chain, route blocker |
| R-09 | Bad output | YELLOW | 15 min | Specific correction or reassign |
| R-10 | Agent crash | RED | Immediate | Check commits, restart with context |
| R-11 | Same file conflict | YELLOW | At merge | Earlier merge order wins |
| R-12 | Scope change | YELLOW/RED | End of wave | Finish current chains, then adjust |

**Related Documents:**
- [INTERVENTIONS.md](interventions.md) — Detailed intervention templates (what to type into agent terminals)
- [STATUS-TRACKING.md](status-tracking.md) — How to track sprint status
- [OPERATORS-GUIDE.md](../guides/operators-guide.md) — Full operator tutorial
- [METHODOLOGY.md](../core/methodology.md) — Execution traces and chain execution theory
