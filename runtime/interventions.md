# Intervention Catalog

> Copy-paste messages for every operator intervention. When you need to correct an agent, find the intervention here, paste the message, and verify.

## How to Use This Catalog

1. Detect the issue (monitoring dashboard, terminal output, git status)
2. Find the matching intervention below
3. Copy the MESSAGE template
4. Replace all `[bracketed]` values with specifics
5. Paste into the agent's terminal
6. Wait for the agent to respond
7. VERIFY the intervention worked

**Escalation ladder:** 5 min observe → 15 min intervene → 30 min reassign. See [REACTIONS.md](reactions.md) for decision trees.

---

## Territory Violations

### INT-01: Agent Modifying Wrong File

**DETECT:** `git diff --name-only` shows files outside agent's territory.

**MESSAGE:**
```
STOP. You modified [FILE_PATH] which is outside your territory.

Your territory: [TERRITORY_LIST]
The file you modified belongs to [OTHER_AGENT].

Revert now: git checkout -- [FILE_PATH]
Continue with your remaining chains within your territory.
```

**VERIFY:** `git diff --name-only` shows only files within the agent's territory.
**SEVERITY:** YELLOW

---

### INT-02: Agent Creating Files in Wrong Directory

**DETECT:** New file created outside territory (check `git status` for untracked files).

**MESSAGE:**
```
HOLD. You created [FILE_PATH] which is outside your territory.

Your territory: [TERRITORY_LIST]
Delete this file: rm [FILE_PATH]

If you need this file for your chain, document it as a blocker:
"Need [FILE_PATH] created — assign to [CORRECT_AGENT]."
Continue with your chains.
```

**VERIFY:** File is removed. No untracked files outside territory.
**SEVERITY:** YELLOW

---

### INT-03: Agent Reading-Only but Documenting as Modified

**DETECT:** Agent's completion report claims changes to files it hasn't actually modified (or vice versa).

**MESSAGE:**
```
Your completion report lists [FILE] as modified, but git shows no changes to it.
OR: git shows changes to [FILE] but your report doesn't mention it.

Update your completion report to accurately reflect:
- Files actually modified (check: git diff --name-only)
- Lines changed per file
- Reason for each change
```

**VERIFY:** Completion report matches `git diff --name-only` exactly.
**SEVERITY:** GREEN

---

## Chain Execution Violations

### INT-04: Agent Skipping Chains Out of Order

**DETECT:** Agent starts Chain 3 before completing Chain 1 or Chain 2.

**MESSAGE:**
```
STOP. You started Chain [N] before completing Chain [N-1].

Chain execution protocol: Complete each chain fully before starting the next.
Order: P0 → P1 → P2 → P3. Within same priority: sequential order.

Go back to Chain [N-1]. Complete it (trace → fix → verify → document).
Then proceed to Chain [N].
```

**VERIFY:** Agent resumes the correct chain and completes it before moving on.
**SEVERITY:** YELLOW

---

### INT-05: Agent Not Verifying Before Moving On

**DETECT:** Agent marks a chain as complete but didn't run build/tests or verify the fix.

**MESSAGE:**
```
HOLD. You marked Chain [N] as complete but did not verify.

Before moving to Chain [N+1], you MUST:
1. Run build: [BUILD_COMMAND]
2. Run tests: [TEST_COMMAND]
3. Confirm the original signal is resolved: [VERIFICATION_STEP]

Do this now. Do not start Chain [N+1] until Chain [N] is fully verified.
```

**VERIFY:** Agent runs build + tests, reports results.
**SEVERITY:** YELLOW

---

### INT-06: Agent Context-Switching Mid-Chain

**DETECT:** Agent starts working on an unrelated task before finishing the current chain.

**MESSAGE:**
```
STOP. You're context-switching mid-chain.

Current chain: Chain [N] — [CHAIN_TITLE]
You started working on: [UNRELATED_WORK]

Revert the unrelated changes. Return to Chain [N].
Complete it fully (fix → verify → document) before doing anything else.

Exception: If you discovered a P0 issue, document it and flag for operator.
```

**VERIFY:** Agent reverts unrelated work and resumes the current chain.
**SEVERITY:** YELLOW

---

## Agent Stuck Protocols

### INT-07: Agent Stuck — Wrong Root Cause

**DETECT:** Agent is repeatedly editing the wrong file/function. Changes don't fix the signal.

**MESSAGE:**
```
STOP iterating. The root cause is not where you're looking.

You've been modifying [WRONG_FILE]. The actual issue is:
File: [CORRECT_FILE]
Line: [LINE_NUMBER]
Issue: [DESCRIPTION]

Revert your recent changes: git checkout -- [WRONG_FILE]
Read [CORRECT_FILE] starting at line [LINE_NUMBER].
The fix is: [SPECIFIC_GUIDANCE]
```

**VERIFY:** Agent reads the correct file and makes a targeted fix.
**SEVERITY:** YELLOW

---

### INT-08: Agent Stuck — Needs Information

**DETECT:** Agent asks questions or makes assumptions about unknown context.

**MESSAGE:**
```
Here's the information you need:

[ANSWER TO AGENT'S QUESTION]

Specifically:
- [FACT 1]
- [FACT 2]
- [FACT 3]

Continue with Chain [N] using this information.
```

**VERIFY:** Agent proceeds with the correct information. No more guessing.
**SEVERITY:** GREEN

---

### INT-09: Agent Stuck — Fundamentally Blocked

**DETECT:** Agent can't make progress even after corrections. Chain requires work outside agent's capability or territory.

**MESSAGE:**
```
PARK Chain [N]. This chain is blocked and cannot be completed in this sprint.

Document in your completion report:
- Chain [N]: PARKED
- Reason: [BLOCKER_DESCRIPTION]
- Needed: [WHAT_WOULD_UNBLOCK_IT]

Move to Chain [N+1]. If all remaining chains are complete, write your
completion report and commit.
```

**VERIFY:** Agent parks the chain, documents the blocker, moves to next chain.
**SEVERITY:** YELLOW

---

## P0 Discovery Routing

### INT-10: P0 Found — Fix Now (Same Agent)

**DETECT:** Agent reports a P0 discovery that falls within their territory.

**MESSAGE:**
```
P0 ACKNOWLEDGED. This is now your highest priority.

PAUSE your current chain (Chain [N]).
NEW PRIORITY: Fix the P0 issue you discovered.

P0 Chain: [TITLE]
Vector: [TRACE_FROM_P0_REPORT]
Fix: [GUIDANCE]
Verify: [VERIFICATION]

After fixing, resume Chain [N] from where you left off.
```

**VERIFY:** P0 is fixed and verified. Agent resumes original chain.
**SEVERITY:** RED

---

### INT-11: P0 Found — Route to Different Agent

**DETECT:** Agent reports a P0 discovery in another agent's territory.

**MESSAGE to discovering agent:**
```
P0 ACKNOWLEDGED. You correctly flagged this.
This is in [OTHER_AGENT]'s territory. Do NOT fix it.

Document it in your completion report under P0 DISCOVERIES.
Continue with your assigned chains.
```

**MESSAGE to responsible agent:**
```
P0 ESCALATION from [DISCOVERING_AGENT].

NEW P1 CHAIN (execute before your remaining P2/P3 chains):
Title: [P0_TITLE]
Vector: [TRACE]
Signal: [DESCRIPTION]
Fix: [GUIDANCE]
Verify: [VERIFICATION]
```

**VERIFY:** Discovering agent continues their work. Responsible agent addresses the P0.
**SEVERITY:** RED

---

### INT-12: P0 Found — Stop Sprint

**DETECT:** P0 is severe enough that continuing the sprint would make things worse (data corruption, security breach active).

**MESSAGE (broadcast to all agents):**
```
SPRINT PAUSED. P0 Critical.

ALL AGENTS: Stop your current work immediately.
1. Commit everything you have right now
2. Write a partial completion report with current state
3. Do NOT start new chains

Reason: [P0_DESCRIPTION]

I will resume the sprint after the P0 is resolved.
Stand by for further instructions.
```

**VERIFY:** All agents commit and stop. Sprint health → RED.
**SEVERITY:** RED

---

## Inter-Agent Conflict

### INT-13: Merge Conflict Between Agents

**DETECT:** Two agents modified the same file (discovered during merge or pre-merge scan).

**MESSAGE to later-merge-order agent:**
```
Your changes to [FILE] conflict with [EARLIER_AGENT]'s changes.

[EARLIER_AGENT] has merge priority (earlier in merge order).
Their changes will be kept. You need to redo your change to [FILE]
so it works with [EARLIER_AGENT]'s version.

After [EARLIER_AGENT]'s branch is merged, rebase your branch:
git rebase main

Then re-apply your fix to [FILE] in a way that's compatible.
```

**VERIFY:** Later agent's branch rebases cleanly and the file works with both changes.
**SEVERITY:** YELLOW

---

### INT-14: Agent Depends on Another Agent's Uncommitted Work

**DETECT:** Agent B needs a change that Agent A is still working on.

**MESSAGE to blocked agent:**
```
Chain [N] is blocked waiting on [OTHER_AGENT]'s Chain [M].

PARK Chain [N]. Move to your next chain.
I'll notify you when [OTHER_AGENT]'s work is available.

Document: "Chain [N]: PARKED — waiting on [OTHER_AGENT] Chain [M]."
```

**MESSAGE to blocking agent (after they finish):**
```
[BLOCKED_AGENT] is waiting on your Chain [M] output.
Please ensure Chain [M] is committed before moving on.
```

**VERIFY:** Blocked agent moves to next chain. Blocking agent prioritizes the needed chain.
**SEVERITY:** YELLOW

---

### INT-15: Duplicate Work Detected

**DETECT:** Two agents are working on the same bug or making the same change.

**MESSAGE to lower-priority agent:**
```
STOP. [OTHER_AGENT] is already working on [TASK_DESCRIPTION].

This chain overlaps with their Chain [M]. Your work is redundant.
Revert your changes: git checkout -- [FILES]

Skip Chain [N] — mark it as "Duplicate of [OTHER_AGENT] Chain [M]" in your report.
Move to Chain [N+1].
```

**VERIFY:** One agent continues, the other moves on. No duplicate work.
**SEVERITY:** YELLOW

---

## Agent Crash/Recovery

### INT-16: Agent Disconnected — Resume

**DETECT:** Agent terminal closed or session died. Branch has committed work.

**MESSAGE (paste into new terminal in same worktree):**
```
You are [CODENAME] agent RESUMING Sprint [XX].
Branch: sprint-[XX]/[AGENT_NAME]

Your previous session ended unexpectedly. Here's your state:

COMPLETED chains:
- Chain 1: COMPLETE — [SUMMARY]
- Chain 2: COMPLETE — [SUMMARY]

RESUME from: Chain [N] — [CHAIN_TITLE]
Last known state: [TRACING/FIXING/VERIFYING]
[DETAILS OF WHERE THEY LEFT OFF]

TERRITORY: [SAME AS ORIGINAL]
PROTOCOLS: [SAME AS ORIGINAL]

Continue from where you left off. Do not redo completed chains.
```

**VERIFY:** Agent picks up from the correct chain without redoing work.
**SEVERITY:** YELLOW

---

### INT-17: Agent Disconnected — Lost Uncommitted Work

**DETECT:** Agent terminal died AND had uncommitted changes that are now lost.

**MESSAGE:**
```
You are [CODENAME] agent RESTARTING Sprint [XX].
Branch: sprint-[XX]/[AGENT_NAME]

Your previous session ended and uncommitted work was lost.

COMPLETED chains (already committed):
- Chain 1: COMPLETE — [SUMMARY]

REDO Chain [N]: [CHAIN_TITLE]
Your previous attempt was [DESCRIPTION_OF_WHAT_WAS_LOST].
[INCLUDE THE ORIGINAL CHAIN DETAILS]

TERRITORY: [SAME AS ORIGINAL]
PROTOCOLS: [SAME AS ORIGINAL]
```

**VERIFY:** Agent redoes the lost work. Build + tests pass.
**SEVERITY:** RED

---

### INT-18: Agent Unresponsive — Force Restart

**DETECT:** Agent is running but not responding to messages. Appears hung.

**MESSAGE:**
```
[First, kill the agent process and start fresh]
[Paste INT-16 or INT-17 recovery prompt depending on commit state]
```

**Pre-restart steps:**
1. Check committed state: `git -C /path/to/worktree log --oneline -5`
2. Check uncommitted state: `git -C /path/to/worktree diff --stat`
3. If uncommitted work exists, try to commit it: `git -C /path/to/worktree add . && git -C /path/to/worktree commit -m "WIP: auto-save before restart"`
4. Kill and restart the agent tool
5. Use INT-16 (if work was saved) or INT-17 (if work was lost)

**VERIFY:** New session starts cleanly. Agent resumes correct chain.
**SEVERITY:** RED

---

## Output Quality

### INT-19: Agent Produced Incomplete Fix

**DETECT:** Agent's fix addresses part of the chain but misses edge cases or related code paths.

**MESSAGE:**
```
Your fix for Chain [N] is partially correct but incomplete.

What you fixed: [CORRECT_PART]
What you missed: [MISSING_PART]

Specifically:
- [MISSING_EDGE_CASE_1]
- [MISSING_CODE_PATH_2]
- [MISSING_VALIDATION_3]

Complete the fix before marking Chain [N] as done.
```

**VERIFY:** Agent extends the fix to cover missing cases. Full verification passes.
**SEVERITY:** GREEN

---

### INT-20: Agent Introduced New Bug

**DETECT:** Agent's change fixes the original issue but introduces a new bug (tests fail, new error appears).

**MESSAGE:**
```
Your fix for Chain [N] introduced a regression.

Original issue: [FIXED ✓]
New issue: [DESCRIPTION OF NEW BUG]
Evidence: [TEST FAILURE / ERROR MESSAGE]

The problem is likely at [FILE:LINE]: [GUIDANCE]
Fix the regression without reverting the original fix.
Then re-verify both the original chain and the new issue.
```

**VERIFY:** Original fix still works AND regression is resolved. All tests pass.
**SEVERITY:** YELLOW

---

### INT-21: Agent Over-Engineered the Fix

**DETECT:** Agent added unnecessary abstraction, refactoring, or changes beyond the chain's scope.

**MESSAGE:**
```
Your fix for Chain [N] is correct but over-engineered.

The chain required: [MINIMAL_FIX_DESCRIPTION]
You also added: [UNNECESSARY_ADDITIONS]

Revert the unnecessary changes:
git checkout -- [FILES_WITH_UNNECESSARY_CHANGES]

Keep only the minimal fix. This sprint isn't about refactoring.
Document suggested improvements in your completion report under
"Suggested follow-up" for a future sprint.
```

**VERIFY:** Only the minimal fix remains. No unnecessary abstractions added.
**SEVERITY:** GREEN

---

## Scope Management

### INT-22: Agent Adding Unrequested Features

**DETECT:** Agent is implementing features or improvements not in their assigned chains.

**MESSAGE:**
```
STOP. You're adding [FEATURE/IMPROVEMENT] which is not in your chains.

Your assigned chains: [LIST]
What you're working on: [UNASSIGNED_WORK]

Revert: git checkout -- [FILES]
Return to your assigned chains. If you believe this improvement is
valuable, document it in your completion report under "Suggested
follow-up" — don't implement it now.
```

**VERIFY:** Agent reverts unassigned work and returns to their chains.
**SEVERITY:** YELLOW

---

### INT-23: Agent Adding Unnecessary Dependencies

**DETECT:** Agent installs new packages/dependencies not justified by their chains.

**MESSAGE:**
```
HOLD. You added [DEPENDENCY] but your chains don't require it.

Revert: [COMMAND TO REMOVE DEPENDENCY]
(e.g., npm uninstall [package], go mod tidy, pip uninstall [package])

If you believe this dependency is necessary for your chain, explain
why in your completion report. Otherwise, use existing tools.
```

**VERIFY:** Dependency is removed. `[dependency file]` matches pre-sprint state for unrelated entries.
**SEVERITY:** YELLOW

---

### INT-24: Agent Modifying Shared Config Files

**DETECT:** Agent edits shared config (package.json, go.mod, Makefile, .env.example) without justification.

**MESSAGE:**
```
HOLD. [CONFIG_FILE] is a shared configuration file.

Changes to shared configs must be justified and documented.
What did you change? Why was it necessary for your chain?

If justified: document the change and reasoning in your completion report.
If not justified: git checkout -- [CONFIG_FILE]

Note: LEAD has final authority on shared config changes during merge.
```

**VERIFY:** Change is either justified (documented) or reverted.
**SEVERITY:** YELLOW

---

## Quick Reference

| ID | Category | Intervention | Severity |
|----|----------|-------------|----------|
| INT-01 | Territory | Wrong file modified | YELLOW |
| INT-02 | Territory | File created in wrong dir | YELLOW |
| INT-03 | Territory | Report doesn't match changes | GREEN |
| INT-04 | Chain | Chains out of order | YELLOW |
| INT-05 | Chain | No verification | YELLOW |
| INT-06 | Chain | Context-switching mid-chain | YELLOW |
| INT-07 | Stuck | Wrong root cause | YELLOW |
| INT-08 | Stuck | Needs information | GREEN |
| INT-09 | Stuck | Fundamentally blocked | YELLOW |
| INT-10 | P0 | Fix now (same agent) | RED |
| INT-11 | P0 | Route to different agent | RED |
| INT-12 | P0 | Stop sprint | RED |
| INT-13 | Conflict | Merge conflict | YELLOW |
| INT-14 | Conflict | Cross-agent dependency | YELLOW |
| INT-15 | Conflict | Duplicate work | YELLOW |
| INT-16 | Crash | Resume (committed work) | YELLOW |
| INT-17 | Crash | Restart (lost work) | RED |
| INT-18 | Crash | Force restart (hung) | RED |
| INT-19 | Quality | Incomplete fix | GREEN |
| INT-20 | Quality | New bug introduced | YELLOW |
| INT-21 | Quality | Over-engineered | GREEN |
| INT-22 | Scope | Unrequested features | YELLOW |
| INT-23 | Scope | Unnecessary dependencies | YELLOW |
| INT-24 | Scope | Shared config modified | YELLOW |

---

**Related Documents:**
- [REACTIONS.md](reactions.md) — Decision trees for runtime events
- [STATUS-TRACKING.md](status-tracking.md) — Sprint monitoring methodology
- [OPERATORS-GUIDE.md](../guides/operators-guide.md) — Full operator tutorial
- [TEMPLATE-ACTIVATION.md](../templates/activation.md) — Activation prompt templates
