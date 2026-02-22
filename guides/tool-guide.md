# Tool Guide

> Choose and configure AI coding agents for Agent Dispatch sprints

Agent Dispatch is tool-agnostic. The framework coordinates through git branches, worktrees, and completion reports — not through any specific agent's internals. Any agent that can read files, edit code, and run commands works.

This guide helps you pick the right tool for your team and configure it for sprint operation.

---

## Capability Matrix

| Capability | Claude Code | Codex CLI | Cursor | Windsurf | Aider | Continue | OpenCode | Qwen Coder |
|---|---|---|---|---|---|---|---|---|
| Sub-agent spawning | Yes — native (Task tool) | No | No | No | No | No | No | No |
| Multi-file editing | Yes | Yes | Yes (Composer) | Yes (Cascade) | Yes | Yes | Yes | Yes |
| Autonomous operation | Yes | Yes (--full-auto) | Partial | Partial | Yes (--yes) | Partial | Yes | Yes |
| Terminal access | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Context window | 200K | 200K | 128K* | 128K* | Varies | Varies | Varies | 128K |
| Git worktree aware | Yes | Yes | Yes (open folder) | Yes (open folder) | Yes | Yes (open folder) | Yes | Yes |
| Completion reports | Auto | Manual | Manual | Manual | Manual | Manual | Manual | Manual |
| Cost model | API usage | API usage | Subscription | Subscription | API usage | Free + API | API usage | Free / API |

*Context window depends on configured model.

**"Worktree aware"** means the tool can operate inside a git worktree directory without special configuration. All tools on this list can do this because worktrees look like ordinary project directories.

---

## Recommendations

**Best for Agent Dispatch:**

- **Full autonomous sprints** — Claude Code. Native sub-agents via the Task tool, 200K context, writes completion reports without prompting, runs builds and tests autonomously.
- **IDE-integrated sprints** — Cursor or Windsurf. Familiar environment for developers who prefer visual editing. Composer (Cursor) and Cascade (Windsurf) handle multi-file changes within a chain.
- **Budget-conscious teams** — Aider or OpenCode. Open-source, bring-your-own API key. Aider's `--auto-commits` pairs well with the sprint's commit-per-chain model.
- **Mixed teams** — Different tools per agent role works fine. The framework doesn't care which tool edits a branch. BACKEND on Claude Code, FRONTEND on Cursor, QA on Aider — all valid.

**Weakest fit:**

- Tools that require approval for every file edit are better suited for supervised, single-agent use than for autonomous parallel sprints.
- Tools without terminal access cannot run build and test commands, which are required for chain verification.

---

## Per-Tool Setup

### Claude Code

**Install:**

```bash
npm install -g @anthropic-ai/claude-code
```

**Per-worktree launch:**

```bash
cd /path/to/project-backend
claude --dangerously-skip-permissions
```

`--dangerously-skip-permissions` enables fully autonomous operation: the agent edits files, runs commands, and writes completion reports without pausing for approval on each action. Use this flag for sprint operation. Without it, the agent prompts for confirmation on tool use, which breaks autonomous execution.

**Configuration for Agent Dispatch:**

- Each worktree is a separate Claude Code session. Open one terminal per worktree, launch `claude` in each.
- CLAUDE.md is read automatically on session start. Put project context there so every agent reads it without explicit prompting.
- Sub-agents: Add the Team Mode block from `TEMPLATE-ACTIVATION.md` to the activation prompt. The agent uses the Task tool to spawn helpers for independent subtasks within its territory.
- Completion reports: Claude Code writes files autonomously. The agent will produce the report at the path specified in the activation prompt without further intervention.
- Context files: Front-load context in the activation prompt. List files to read explicitly — the agent reads them before touching any code.

**Activation prompt tips:**

- List context files by absolute path for maximum reliability.
- CLAUDE.md is the best place for project-level context that should be available to every agent automatically.
- For sub-agent spawning: "Use the Task tool to spawn a '[name]' sub-agent for [subtask]. Sub-agent must respect your territory boundaries."
- Build and test commands must be explicit — do not assume the agent knows your toolchain.

**Strengths:** Native sub-agent support, largest context window (200K), autonomous file editing and command execution, writes completion reports without prompting, tool use (read/write/grep/bash in one session).

**Limitations:** API cost scales with context length and sub-agent count. Requires an Anthropic API key.

---

### Codex CLI (OpenAI)

**Install:**

```bash
npm install -g @openai/codex
```

**Per-worktree launch:**

```bash
cd /path/to/project-backend
codex --full-auto
```

`--full-auto` enables autonomous operation. Codex executes in a sandboxed environment, which provides a safety layer but means some filesystem operations require explicit path specification.

**Configuration for Agent Dispatch:**

- Each worktree is a separate Codex session in a separate terminal.
- No native sub-agents — the operator must manually dispatch sub-tasks if work needs to be split.
- The sandbox environment means Codex is well-suited for agents that need safe execution of potentially risky commands.

**Activation prompt tips:**

- Be explicit about file paths. Codex performs better with absolute paths than relative ones.
- Include build and test commands in the prompt. State them as: "After each chain, run: `[command]`."
- Request a structured completion report explicitly: "When all chains are complete, write a completion report to `[absolute path]` with this structure: [structure]."
- Codex responds well to step-by-step chain instructions. Break each chain into numbered sub-steps.

**Strengths:** GPT-4.1 model, sandboxed execution for safety, good code generation across many languages.

**Limitations:** No sub-agents. Sandbox may restrict some filesystem operations. API cost. Completion reports require explicit prompting.

---

### Cursor

**Setup:**

- Install Cursor from cursor.sh.
- Open each worktree folder as a separate Cursor window: File → Open Folder → select the worktree directory.
- Enable Agent mode in settings for autonomous multi-step operation.
- Use Composer (Cmd+I / Ctrl+I) for multi-file edits within a chain.

**Configuration for Agent Dispatch:**

- One Cursor window = one agent. Each window operates on a separate worktree with its own branch.
- Composer mode handles multi-file changes across a chain. Use it when a chain requires coordinated edits across multiple files.
- Agent mode allows autonomous file creation and terminal command execution, but Cursor will still surface some confirmations depending on the operation type.
- For sprint use, configure Cursor to auto-accept changes in settings where possible.

**Activation prompt tips:**

- Paste the activation prompt into the Composer chat input.
- Use `@file` mentions to reference context files: `@CLAUDE.md`, `@docs/agent-dispatch/sprint-01/agent-backend.md`.
- Reference files by relative path within the worktree. Cursor's context is scoped to the open workspace folder.
- The Composer context window is smaller than Claude Code's. Keep activation prompts focused — reference the task doc by file rather than inlining all chains.

**Strengths:** IDE integration with visual diffs and inline editing. Composer for multi-file chains. Familiar environment for developers. Strong code completion in addition to agent mode.

**Limitations:** Partial autonomy — some operations surface approval dialogs. Subscription model (no per-API-call cost, but requires Cursor subscription). No sub-agents. Context window depends on configured model (128K default).

---

### Windsurf

**Setup:**

- Install Windsurf from codeium.com/windsurf.
- Open each worktree folder as a separate Windsurf workspace.
- Use Cascade mode for multi-step, multi-file operations.

**Configuration for Agent Dispatch:**

- One Windsurf workspace = one agent.
- Cascade mode chains multiple actions together — well-aligned with Agent Dispatch's chain execution model. A Cascade session can execute one full chain (read → edit → build → test) before moving to the next.
- Terminal access is available in-editor for running build and test commands.

**Activation prompt tips:**

- Paste the activation prompt into the Cascade chat panel.
- Windsurf's Cascade performs well when chains are structured as sequences: "First do X. When X is verified, do Y."
- Use explicit file references. Cascade resolves relative paths from the workspace root.
- For the completion report: "When done, write this exact file structure to `[relative path]`."

**Strengths:** IDE integration. Cascade for multi-step chain execution. Good at following structured file references. Terminal access in-editor.

**Limitations:** Subscription model. Less autonomous than CLI tools (some steps require confirmation). No sub-agents. Context window depends on configured model.

---

### Aider

**Install:**

```bash
pip install aider-chat
```

**Per-worktree launch:**

```bash
cd /path/to/project-backend
aider --yes --auto-commits --model claude-sonnet-4-6
```

`--yes` auto-accepts all suggestions. `--auto-commits` writes a git commit after each change, which aligns with Agent Dispatch's commit-per-chain model. `--model` sets the LLM backend — Aider supports Anthropic, OpenAI, Gemini, local models, and more.

**Configuration for Agent Dispatch:**

- Pre-load context files with `--read`:

```bash
aider --yes --auto-commits \
  --read docs/agent-dispatch/sprint-01/agent-backend.md \
  --read docs/agent-dispatch/agents/README.md \
  --model claude-sonnet-4-6
```

- Aider's `--auto-commits` pairs naturally with Agent Dispatch's "commit after each chain" protocol. Each chain produces a commit, making the sprint history clean and reviewable.
- No native sub-agents. If a chain needs to be split, the operator opens two Aider sessions in the same worktree and coordinates manually.

**Activation prompt tips:**

- Use `/add` commands to load files into context before running a chain:

```
/add src/handler.go src/service.go
```

- Paste the full activation prompt chain-by-chain. Aider works well with sequential instructions per chain.
- For the completion report, instruct explicitly: "Create the file `docs/agent-dispatch/sprint-01/agent-backend-completion.md` with this content: [template]."
- Aider responds well to diff-style instructions: "In `service.go`, change `function X` to do Y instead of Z."

**Strengths:** Open-source, supports many LLM backends (Anthropic, OpenAI, Gemini, local). Git-native with `--auto-commits`. Good multi-file editing. Cheap — bring your own API key at whatever tier fits your budget.

**Limitations:** No sub-agents. Context window is bounded by the configured model. Text-only terminal interface (no visual diff). Completion reports require explicit prompting.

---

### Continue

**Setup:**

- Install the Continue extension in VS Code (continue.dev).
- Open each worktree folder as a separate VS Code window: `code /path/to/project-backend`.
- Configure your LLM provider and model in Continue's settings (`.continue/config.json`).

**Configuration for Agent Dispatch:**

- One VS Code window with Continue = one agent.
- Continue's chat panel handles the activation prompt. Terminal panel in the same window runs builds and tests.
- Continue integrates with VS Code's file explorer — use `@file` mentions to pull context into the chat.
- More interactive than autonomous by default. Best suited for agents that benefit from operator oversight during chain execution.

**Activation prompt tips:**

- Paste the activation prompt into Continue's chat input.
- Use `@file` to reference task documents and context files:

```
@docs/agent-dispatch/sprint-01/agent-backend.md
@CLAUDE.md
```

- Continue works well with iterative instructions: give Chain 1 first, confirm verification, then give Chain 2.
- For completion reports: instruct explicitly with the target file path and expected structure.

**Strengths:** Free extension, IDE-integrated, supports many models (configure any backend), VS Code ecosystem (debugging, source control, terminal all in one window).

**Limitations:** More interactive than autonomous — less suited for fully unattended sprint execution. No sub-agents. Autonomy depends on configured model and Continue's agent mode settings. Context window depends on configured model.

---

### OpenCode

**Install:**

```bash
go install github.com/opencode-ai/opencode@latest
```

**Per-worktree launch:**

```bash
cd /path/to/project-backend
opencode
```

Configure your LLM provider in OpenCode's config file. OpenCode supports Anthropic, OpenAI, and other backends.

**Configuration for Agent Dispatch:**

- Terminal-based, works naturally with git worktrees — the working directory is the agent's territory.
- Multi-file editing is a core feature. OpenCode can coordinate changes across multiple files in a single operation.
- Supports multiple LLM backends, making it flexible for teams with existing API agreements.

**Activation prompt tips:**

- Paste the full activation prompt at session start. OpenCode handles long prompts well.
- Include explicit absolute paths for context files — OpenCode performs better with absolute paths than relative ones in some configurations:

```
Read these files first:
1. /full/path/to/CLAUDE.md
2. /full/path/to/docs/agent-dispatch/sprint-01/agent-backend.md
```

- Build and test commands must be stated explicitly.
- For completion reports: specify the full absolute path and expected file structure.

**Strengths:** Open-source. Multi-model support. Good multi-file editing. Terminal-based workflow pairs well with CLI-heavy sprint operations. No subscription — API key only.

**Limitations:** No sub-agents. Newer tool with a smaller community than Claude Code or Aider. Completion reports require explicit prompting.

---

### Qwen Coder

**Per-worktree launch:**

```bash
cd /path/to/project-backend
qwen-coder
```

**Configuration for Agent Dispatch:**

- Qwen Coder performs best with explicit, step-by-step instructions and absolute file paths.
- Territory boundaries should be stated in precise terms — list exact directory paths rather than describing them semantically.
- Chain instructions should be broken into numbered sub-steps rather than described as a single task.

**Activation prompt tips:**

Front-load with absolute file paths:

```
Read these files first:
1. /full/absolute/path/to/CLAUDE.md
2. /full/absolute/path/to/docs/agent-dispatch/sprint-01/agent-backend.md
3. /full/absolute/path/to/docs/agent-dispatch/agents/README.md
```

For territory boundaries, be exact:

```
TERRITORY:
- CAN modify: /full/path/to/project/src/handlers/, /full/path/to/project/src/services/
- CANNOT modify: /full/path/to/project/src/store/, /full/path/to/project/frontend/
```

For each chain, break it into steps:

```
Chain 1 [P1]: Fix webhook timeout
  Step 1. Read /full/path/to/store/subscription.go
  Step 2. Find the function that holds a mutex during notificationService.Send()
  Step 3. Move the mutex release to before the notification call
  Step 4. Run: go build ./...
  Step 5. Run: go test ./... -race
  Step 6. Confirm no DATA RACE output
```

**Strengths:** Strong code generation, good with explicit step-by-step instructions, works well with long structured prompts.

**Limitations:** Needs more explicit file paths and step-by-step instructions than other tools — activation prompts require more preparation. No sub-agents. Context window 128K.

---

## Migration Notes

### Switching Tools Mid-Sprint

Worktrees are tool-agnostic. The git branch does not know or care which tool edited it.

To switch tools mid-sprint:

1. Have the current tool commit its progress: "Commit all current changes with message 'wip: switching tools mid-sprint'"
2. Close the current tool session.
3. Open the new tool in the same worktree directory.
4. Resume with: "Read `docs/agent-dispatch/sprint-01/agent-backend.md` and `git log --oneline -5` to understand what has been completed. Continue from Chain [N]."

Uncommitted changes are preserved in the working tree regardless of which tool made them. Committed changes are in git history.

### Using Multiple Tools in One Sprint

Running different tools for different agents is valid and sometimes advantageous:

- BACKEND: Claude Code (sub-agents for complex handler chains)
- FRONTEND: Cursor (IDE for visual component work)
- QA: Aider (cheap, thorough scanning with auto-commits)
- DATA: Claude Code (careful, race-detector-verified changes)

The framework coordinates through git branches and completion reports. The only requirements are that each tool can read files, edit code, and commit to git. Tool interop is not needed.

---

## Automation Scripts

For teams that prefer shell-based automation:

**`dispatch.sh`** — Create worktrees and launch agents:

```bash
#!/usr/bin/env bash
# Usage: ./dispatch.sh sprint-01
SPRINT=$1
PROJECT=$(basename "$(pwd)")
PARENT=$(dirname "$(pwd)")

for agent in backend frontend infra services qa data; do
  git branch "$SPRINT/$agent" main 2>/dev/null || true
  git worktree add "$PARENT/${PROJECT}-${agent}" "$SPRINT/$agent"
  # Launch agent in new terminal — adjust for your terminal emulator
  osascript -e "tell app \"Terminal\" to do script \"cd $PARENT/${PROJECT}-${agent} && claude --dangerously-skip-permissions\""
done
```

**`monitor.sh`** — Poll for completion report creation:

```bash
#!/usr/bin/env bash
# Usage: ./monitor.sh sprint-01
SPRINT=$1
PARENT=$(dirname "$(pwd)")
PROJECT=$(basename "$(pwd)")

for agent in backend frontend infra services qa data; do
  REPORT="$PARENT/${PROJECT}-${agent}/docs/agent-dispatch/${SPRINT}/agent-${agent}-completion.md"
  if [ -f "$REPORT" ]; then
    echo "[DONE] $agent — $(head -3 "$REPORT" | tail -1)"
  else
    LAST_COMMIT=$(git -C "$PARENT/${PROJECT}-${agent}" log --oneline -1 2>/dev/null || echo "no commits")
    echo "[WORKING] $agent — last commit: $LAST_COMMIT"
  fi
done
```

**`merge.sh`** — Sequential merge with build+test validation:

```bash
#!/usr/bin/env bash
# Usage: ./merge.sh sprint-01
SPRINT=$1
BUILD_CMD=${BUILD_CMD:-"go build ./..."}
TEST_CMD=${TEST_CMD:-"go test ./..."}

git checkout main

for agent in data design backend services frontend infra qa lead; do
  echo "Merging $SPRINT/$agent..."
  git merge "$SPRINT/$agent" --no-ff -m "Sprint: $agent"
  if ! eval "$BUILD_CMD" && eval "$TEST_CMD"; then
    echo "MERGE FAILED at $agent — build or tests broken. Resolve before continuing."
    exit 1
  fi
  echo "PASS — $agent merged cleanly"
done

echo "All agents merged. Sprint complete."
```

---

## Related Documents

- [OPERATORS-GUIDE.md](operators-guide.md) — Full sprint tutorial, Section 9 for agent-specific tips
- [TEMPLATE-ACTIVATION.md](../templates/activation.md) — Activation prompt templates for all agent roles
- [README.md](../README.md) — Framework overview and agent compatibility summary
- [METHODOLOGY.md](../core/methodology.md) — Execution traces, chain execution, priority levels
- [CUSTOMIZATION.md](customization.md) — Adapting the framework for your project
