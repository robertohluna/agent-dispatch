<!--
  TEMPLATE: Copy this file to sprint-XX/RETROSPECTIVE.md after each sprint closes.
  Fill in every [bracketed placeholder] with actual values from your completion reports.
  Delete table rows that have no content rather than leaving placeholder text.
  Written by LEAD or the human operator — after the sprint tag is pushed, before next planning.

  Inputs to pull from:
    - sprint-XX/DISPATCH.md          (goals, success criteria, wave assignments)
    - sprint-XX/agent-*-completion.md (chains completed, files changed, P0 discoveries)
    - git log / git diff --stat      (objective metrics)
    - Your own observations as operator during the sprint
-->

# Sprint [XX] Retrospective — [Theme]

> Date: [YYYY-MM-DD]
> Agents dispatched: [N] of 8
> Chains assigned: [N] | Completed: [N] | Parked: [N]
> Duration: [YYYY-MM-DD] → [YYYY-MM-DD]

---

## Results vs. Success Criteria

Pull these rows from `sprint-XX/DISPATCH.md` "Success Criteria" section.

| Criterion | Target | Actual | Met? |
|-----------|--------|--------|------|
| [criterion from DISPATCH.md] | [target value or condition] | [what was measured] | Y / N |
| [criterion] | [target] | [actual] | Y / N |
| [criterion] | [target] | [actual] | Y / N |

**Ship decision:** SHIPPED / NO-SHIP / PARTIAL — [one sentence explaining the outcome]

---

## What Worked

Things that reduced friction, produced good output, or should be repeated next sprint.

- [e.g., Execution traces for DATA were precise — no wasted reads, all root causes found]
- [e.g., Wave 1 had no cross-agent conflicts because territory boundaries were clean]
- [e.g., P0 critical escalation worked correctly — QA flagged the race condition before merge]

---

## What Did Not Work

Things that caused rework, blocked agents, slowed merges, or produced low-quality output.

- [e.g., Chain 3 for BACKEND was under-specified — agent interpreted the vector incorrectly]
- [e.g., FRONTEND depended on an API shape that SERVICES changed mid-sprint, causing a merge conflict]
- [e.g., Two chains were assigned to DATA that belonged in SERVICES's territory]

---

## P0 Discoveries

Critical Escalation signals surfaced during the sprint. Pull from completion reports "P0 Discoveries" sections.

| Discovery | Found By | File | Resolution | Carry to Sprint [XX+1]? |
|-----------|----------|------|------------|------------------------|
| [description] | [AGENT CODENAME] | [path/to/file.ext] | Fixed in sprint / Parked / Deferred | Y / N |
| [description] | [AGENT CODENAME] | [path/to/file.ext] | [resolution] | Y / N |

_None — delete this table if no P0 findings were reported._

---

## Agent Effectiveness

| Agent | Chains Assigned | Chains Completed | Chains Parked | Quality | Notes |
|-------|-----------------|-----------------|---------------|---------|-------|
| BACKEND | [N] | [N] | [N] | HIGH / MED / LOW | [observations] |
| FRONTEND | [N] | [N] | [N] | HIGH / MED / LOW | [observations] |
| INFRA | [N] | [N] | [N] | HIGH / MED / LOW | [observations] |
| SERVICES | [N] | [N] | [N] | HIGH / MED / LOW | [observations] |
| QA | [N] | [N] | [N] | HIGH / MED / LOW | [observations] |
| DATA | [N] | [N] | [N] | HIGH / MED / LOW | [observations] |
| LEAD | [N] | [N] | [N] | HIGH / MED / LOW | [observations] |

Quality definitions: HIGH = chains complete, root causes correct, no rework needed. MED = chains complete but required operator intervention or rework. LOW = chains incomplete, vectors misread, or output not usable.

---

## Execution Methodology Review

Answer each question in one or two sentences based on what actually happened.

- **Execution traces accurate?** Did the traced paths in agent task docs match actual root causes, or did agents need to re-orient after reading the code?
- **Priority levels correct?** Were P1s genuinely more critical than P2s and P3s? Did any P2 turn out to be a P0?
- **Chain execution clean?** Did agents complete one chain before starting the next, or did completion reports show context-switching?
- **Merge validation smooth?** How many merge conflicts occurred? Were they in files expected to have high context density?
- **Context density correct?** Were the files flagged as high-density the actual sites of most changes, or did agents spend most time elsewhere?

---

## Metrics

Pull from git log, git diff --stat, and completion reports.

- Total files modified: [N]
- Total lines changed: +[added] / -[removed]
- Tests added: [N]
- Build failures during merge sequence: [N]
- Merge conflicts resolved: [N]
- P0 discoveries: [N]
- Chains requiring operator intervention: [N]
- Sprint duration (planned vs. actual): [planned] → [actual]

---

## Carry-Forward Items

Parked chains, unresolved P0s, and deferred work. These become input to the next DISPATCH.md.

| Item | Source | Priority | Assigned Sprint | Notes |
|------|--------|----------|----------------|-------|
| [description] | [Agent that parked it] | P1 / P2 / P3 | Sprint [XX+1] | [context for next operator] |
| [description] | [source] | [priority] | Sprint [XX+1] | [notes] |

_None — delete this table if all chains completed and no carry-forward exists._

---

## Process Improvements for Next Sprint

Concrete changes to make before writing the next DISPATCH.md.

- [e.g., Split DATA's territory — query layer and migration layer are too different for one agent]
- [e.g., Add a Wave 0 for reading completion reports before merge — LEAD was blocked waiting]
- [e.g., Write chain vectors at the function level, not the file level — agents need more precision]
- [e.g., Include the build command in every chain's Verify section — one agent skipped it]
