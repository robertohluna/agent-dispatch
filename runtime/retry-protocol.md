# Escalating Retry Protocol

> When an agent fails a chain, don't just send a generic correction.
> Use escalating retries with progressively broader context.
> Inspired by Stripe's minions pattern: max 3 attempts, then move on.

---

## The Problem

When an agent fails a chain (build breaks, tests fail, wrong approach), the default response is a single correction message. This wastes the agent's remaining context on the same narrow view that caused the failure.

## The Solution

Three-tier retry with escalating context. Each retry gives the agent MORE information and a DIFFERENT angle. After 3 attempts, park the chain and move on.

---

## Retry Flow

```
Chain execution attempt
  │
  ├─ Build + test PASS → ✓ Chain COMPLETE, move to next chain
  │
  └─ Build + test FAIL → Enter retry protocol
       │
       ├─ RETRY 1: Focused Fix (narrow context)
       │   └─ PASS → ✓ Chain COMPLETE
       │   └─ FAIL ↓
       │
       ├─ RETRY 2: Rethink (broader context)
       │   └─ PASS → ✓ Chain COMPLETE
       │   └─ FAIL ↓
       │
       └─ RETRY 3: Fresh Approach (full context + all errors)
           └─ PASS → ✓ Chain COMPLETE
           └─ FAIL → ✗ PARK chain, document failure, move to next chain
```

---

## RETRY 1: Focused Fix

**Context level:** Narrow — just the error output
**Approach:** Fix the specific failure

**Message template:**
```
Your chain [CHAIN_TITLE] failed verification.

Error output:
[PASTE EXACT ERROR OUTPUT]

Fix this specific error. Do not change your overall approach —
the approach is likely correct, the implementation has a bug.

After fixing, run:
  [build command]
  [test command]

Both must pass before this chain is complete.
```

**When to use:** First failure. The agent's approach was probably right, just a bug in implementation.

---

## RETRY 2: Rethink

**Context level:** Broader — error output + what was tried + adjacent code
**Approach:** Reconsider the approach, not just the implementation

**Message template:**
```
Your chain [CHAIN_TITLE] has failed twice. Your current approach is not working.

Attempt 1 error:
[PASTE ERROR FROM ATTEMPT 1]

Attempt 2 error:
[PASTE ERROR FROM RETRY 1]

STOP and rethink. Do not apply another incremental fix to the same approach.

Instead:
1. Re-read the original execution trace for this chain
2. Re-read the 3-5 files surrounding the fix site
3. Consider: is the root cause actually where you think it is?
4. Consider: are you matching the codebase's patterns, or fighting them?
5. Try a FUNDAMENTALLY DIFFERENT approach to solving this chain

The definition of insanity is trying the same thing and expecting different results.
Find a different angle.

After implementing the new approach, run:
  [build command]
  [test command]
```

**When to use:** Second failure. The agent is probably in a local minimum — same approach, slightly different implementation, same class of error.

---

## RETRY 3: Fresh Approach

**Context level:** Full — all accumulated errors + related code + architectural context
**Approach:** Start the chain from scratch with maximum context

**Message template:**
```
FINAL ATTEMPT on chain [CHAIN_TITLE]. Two previous approaches have failed.

All accumulated errors:
--- Attempt 1 ---
[ERROR 1]
--- Attempt 2 ---
[ERROR 2]
--- Attempt 3 ---
[ERROR 3]

What has been tried so far:
[BRIEF SUMMARY OF APPROACHES TRIED]

This is your last attempt. If this fails, the chain will be PARKED
and assigned to a different approach in the next sprint iteration.

Instructions:
1. IGNORE your previous work on this chain. Start fresh.
2. Re-read the ENTIRE execution trace from entry point to root cause
3. Read ALL callers and callees of the functions involved
4. Read how SIMILAR fixes were done elsewhere in the codebase
   (grep for similar patterns)
5. Implement a solution that follows the codebase's existing patterns EXACTLY
6. Verify BEFORE committing:
   [build command]
   [test command]

If you cannot make this work, document what you learned about WHY it fails
in your completion report. This information helps the next attempt.
```

**When to use:** Third and final attempt. Give the agent maximum context and a clean slate.

---

## After 3 Retries: PARK the Chain

If all 3 retries fail:

1. **PARK the chain** — mark it as FAILED in the completion report
2. **Document what was tried** — approaches, error messages, hypotheses
3. **Move to the next chain** — do not spend more time on this one
4. **The ORCHESTRATOR will handle it** — either:
   - Reassign to a different agent in a convergence iteration
   - Carry forward to the next sprint with accumulated context
   - Escalate to operator for manual intervention

**Agent message after parking:**
```
Chain [CHAIN_TITLE] has been PARKED after 3 failed attempts.

Move to your next chain immediately. Document what you learned about
this chain's failure in your completion report under "PARKED CHAINS":
- What approaches were tried
- What errors occurred
- Your best hypothesis for why it fails
- What you would try differently

This information will be used to retry with a different approach.
```

---

## Tool Restriction by Retry Level

Optionally restrict tools on retries to prevent the agent from making things worse:

| Retry | Tools Allowed | Rationale |
|-------|--------------|-----------|
| Original attempt | Full tools for category | Normal execution |
| Retry 1 | `Read, Edit, Bash, Grep, Glob` | Fix only — no new files |
| Retry 2 | Full tools for category | Rethink may need new approach |
| Retry 3 | Full tools for category | Fresh start, no restrictions |

---

## Per-Chain Verification (Verify After Every Chain)

Don't wait until all chains are done to verify. Run build + test after EACH chain:

```
CHAIN COMPLETE → run [build command] + [test command]
  │
  ├─ PASS → commit chain, move to next
  │
  └─ FAIL → enter retry protocol (Retry 1 → 2 → 3 → PARK)
```

This catches failures early. A chain that breaks the build in chain 2 of 5 is caught immediately, not discovered at the end when it's harder to diagnose.

**Add to every agent activation prompt:**
```
VERIFICATION PROTOCOL:
After completing EACH chain (not just all chains), run:
  [build command]
  [test command]

If EITHER fails, fix the issue before moving to the next chain.
A chain is not complete until verification passes.
Do not accumulate broken chains — fix as you go.
```

---

## Integration with Interventions

The retry protocol extends the intervention catalog (`runtime/interventions.md`):

| Intervention | When | Message |
|-------------|------|---------|
| INT-RETRY-1 | First build/test failure on a chain | Focused fix (narrow context) |
| INT-RETRY-2 | Second failure on same chain | Rethink (broader context) |
| INT-RETRY-3 | Third failure on same chain | Fresh approach (full context) |
| INT-PARK | After 3 failures | Park chain, move on |

---

## Integration with Convergence Loop

When chains are PARKED, they feed into the convergence loop:

```
Sprint Iteration 1:
  Chain 3 → PARKED after 3 retries
  Chain 5 → PARKED after 3 retries
  All other chains → COMPLETE

ORCHESTRATOR assesses:
  Mission: PARTIALLY ACHIEVED (2 chains parked)

Convergence decision:
  → Auto-generate Iteration 2 targeting parked chains
  → New traces informed by failure analysis from Iteration 1
  → Possibly reassign to different agent or different approach
```

See the convergence loop section in `guides/orchestrator-playbook.md` for details.

---

## Quick Reference

```
Attempt 1: Execute chain → verify → FAIL
Retry 1:   "Fix this error" (narrow) → verify → FAIL
Retry 2:   "Rethink your approach" (broad) → verify → FAIL
Retry 3:   "Start fresh, here's everything" (full) → verify → FAIL
PARK:      Document failure, move to next chain
```
