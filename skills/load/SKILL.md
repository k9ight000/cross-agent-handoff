---
name: load
version: 1.1.0
description: Use when the user says /load, resume, continue from handoff, switch agents/tools, 接班, or a new session needs to pick up from C:\Users\11817\.agents\session-handoff.md.
---

# /load - Resume Cross-Agent Session

## Goal

Resume from the shared handoff without pretending it is the whole project. The handoff is a map. Verify the map against the current workspace, then restore only the context needed for the next safe step.

## Step 1: Read the Handoff

Read:

`C:\Users\11817\.agents\session-handoff.md`

Shell equivalents:

```bash
cat "$HOME/.agents/session-handoff.md" 2>/dev/null || echo "NO_HANDOFF"
```

PowerShell:

```powershell
Get-Content "$env:USERPROFILE\.agents\session-handoff.md" -ErrorAction SilentlyContinue
```

If no handoff exists, reply:

`没有找到交班文件。你告诉我当前任务和项目路径，我直接接着做。`

Then stop.

## Step 2: Validate the Map

Before acting, compare the handoff with the current environment:

- Handoff age: if older than 24 hours, mention the date and treat it as potentially stale.
- Workspace and repo root: if the current directory differs, say so and either switch only if obvious, or ask.
- Branch / HEAD: if the current branch or commit differs from the handoff, report the difference before modifying files.
- Changed files: check `git status --short` when inside a repo.
- User language: continue in the user's preferred language, but preserve exact identifiers in their original spelling.

Never translate or paraphrase exact filenames, commands, API names, environment variables, branch names, error strings, selectors, or IDs.

## Step 3: Brief the User

Present a concise briefing:

```markdown
从交班文件读到：
- Current Task: <summary>
- Current State: <done / verified / not verified>
- Next Steps: <first 2-3 steps>
- Watch Out: <risks or confirmations needed>
- Unknowns: <facts that still need checking>
```

If the user explicitly said `/load`, `接班`, or asked you to continue, proceed after the briefing unless the next action is destructive, externally visible, costly, or otherwise high-risk. For high-risk actions, ask for explicit confirmation first.

## Step 4: Targeted Context Rehydration

Read only what the handoff points to, then expand only if evidence requires it.

Default budget:

- Read first: 3 files/sections maximum.
- Expanded budget: up to 7 files/sections for complex handoffs.
- Prefer `rg` for symbols and exact phrases before opening broad files.
- Prefer `git status --short`, `git diff --stat`, and narrow diffs over full logs.

Use this order:

1. Read `Project Anchors` and `Context Rehydration Pack`.
2. Verify repo/branch/status.
3. Open the listed "Read first" files or run the listed searches.
4. Compare what you find against `Decisions`, `Current State`, and `Next Steps`.
5. If something conflicts, say the conflict and resolve it before editing.

Do not read the entire repository, entire chat history, full logs, or full diffs unless the user asks or the handoff cannot be safely interpreted without it.

## Step 5: Resume Work

Start with the first safe `Next Steps` item that is not already complete.

Rules:

- Treat `Verified` as evidence, but re-run checks when the code may have changed.
- Treat `Not verified` and `Unknowns` as tasks to resolve, not as facts.
- Do not redo work already listed as complete unless verification contradicts it.
- Do not perform delete, overwrite, commit, push, submit, send-message, payment, application, or other high-risk actions without explicit user confirmation.
- If you discover the handoff is wrong or stale, update your plan and tell the user briefly.

When you make meaningful progress or hit a new pause point, run `/save` again so the next handoff reflects the new state.
