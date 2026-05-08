---
name: save
version: 1.0.0
description: Quick cross-agent session handoff. Capture current task, changed files, decisions, and next steps to `C:\Users\11817\.agents\session-handoff.md` so another agent can pick up seamlessly. Use when quota is running low, before switching tools, or at any natural pause point. Costs < 500 tokens, runs in seconds.
---

# /save — Cross-Agent Session Handoff

Save the current session state so another agent (Codex ↔ Claude) can continue.

## Step 1: Gather State (parallel, fast)

Run these in parallel:

```bash
git diff --stat 2>/dev/null
git diff --cached --stat 2>/dev/null
git status --short 2>/dev/null
```

## Step 2: Summarize from Context

Extract from the conversation, keep it brief (2-4 bullets each):

- **Current Task**: What are we doing right now?
- **Decisions**: What was decided and why?
- **Next Steps**: Ordered list of what to do next.

If the user typed `/save <note>`, include `<note>` in the Current Task.

## Step 3: Build Handoff

Fill this template with what you gathered:

```markdown
# Session Handoff
**Time**: <ISO timestamp>
**Source**: <Claude Code | Codex>

## Current Task
<1-2 lines>

## Changed Files
<git status summary, or "none">

## Decisions
- <decision 1>
- <decision 2>

## Next Steps
1. <step 1>
2. <step 2>
3. <step 3>
```

## Step 4: Write and Report

```bash
mkdir -p "$HOME/.agents"
```

Write the handoff to `C:\Users\11817\.agents\session-handoff.md`.

After writing, reply with ONE line:

> 已保存到 `~/.agents/session-handoff.md`。下个工具运行 `/load` 即可继续。

## Rules

- Keep it under 10 seconds. Don't read files or run tests.
- If not in a git repo, skip git commands, note "no git repo" in Changed Files.
- The handoff file is overwritten each time — it's a single-session snapshot, not a log.
