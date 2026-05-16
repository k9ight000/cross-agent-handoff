---
name: save
version: 1.1.0
description: Use when the user says /save, handoff, switch agents/tools, context or quota is low, or a coding session needs a compact reliable state snapshot before another agent continues.
---

# /save - Cross-Agent Session Handoff

## Goal

Write a compact map that another agent can resume from after minimal verification. The handoff is not the whole project. It must preserve the project identity, exact file anchors, decisions, unknowns, and safe next steps so the next agent does not fill gaps with guesses.

## Failure This Prevents

Sparse summaries save tokens but create drift:

- The next agent continues in the wrong repo, branch, or tool context.
- Exact identifiers get translated or paraphrased incorrectly.
- A decision made in conversation is lost because it is not tied to files or evidence.
- Unverified assumptions are treated as facts.
- Risky next actions happen without the user's confirmation.

## Step 1: Fast Inspection

Use the current shell equivalent. Run in parallel when the environment supports it.

```bash
pwd
git rev-parse --show-toplevel 2>/dev/null
git branch --show-current 2>/dev/null
git rev-parse --short HEAD 2>/dev/null
git log -1 --oneline 2>/dev/null
git status --short 2>/dev/null
git diff --stat 2>/dev/null
git diff --cached --stat 2>/dev/null
```

On PowerShell, use `2>$null` instead of `2>/dev/null`.

Do not read the whole project. Use files already seen in the conversation. If the task would be ambiguous without a file anchor, read only the smallest metadata needed, such as filenames, headings, or a narrow `rg` hit.

## Step 2: Choose Detail Level

Default to **compact**: 50-120 lines, usually under 1200 tokens.

Use **expanded** only when needed: 120-200 lines for multi-file refactors, unresolved bugs, failed attempts, hidden setup, project handoff after a long session, or anything where a sparse note would be misleading.

Never include full diffs, full logs, full stack traces, cookies, tokens, session IDs, private page text, or secrets. Record the conclusion and the path to re-check instead.

## Step 3: Build the Handoff

Use this exact structure. Keep path names, commands, API names, environment variables, branch names, and error strings exact. Do not translate identifiers.

```markdown
# Session Handoff
**Time**: <ISO timestamp with timezone>
**Source**: <Codex | Claude Code | Gemini | Antigravity | other>
**Workspace**: <current working directory>
**Repo Root**: <git root, or "no git repo">
**Branch / HEAD**: <branch> @ <short sha, or "unknown">
**Handoff Profile**: <compact | expanded>
**User Language**: <language to use when replying to the user>

## Current Task
<1-3 lines. Include the user's latest instruction and what "done" means.>

## Project Anchors
- Product/component: <what project or subsystem this is>
- Key files or entry points: <3-7 exact paths if known>
- Key commands/env: <commands already used or needed next>
- External context: <links, tickets, docs, or "none">

## Changed Files
<git status summary. For each important file: exact path - what changed or why it matters. If many files changed, group them by area.>

## Decisions
- <decision> - reason: <why>; confidence: <high|medium|low>; evidence: <file/command/conversation>

## Current State
- Done: <what is complete>
- Verified: <commands/checks run and result>
- Not verified: <what still needs proof>
- Blocked: <blockers, or "none">

## Next Steps
1. <first safe step. If context is incomplete, make the first step a targeted verification/read.>
2. <next action>
3. <next action>

## Context Rehydration Pack
- Read first: <3-7 exact files/sections and why. Use "none" only for tiny tasks.>
- Search first: <rg queries or symbols to locate relevant code, if useful>
- Commands to re-run: <short verification commands, if useful>
- Unknowns to resolve: <facts the next agent must not guess>

## Watch Out
- <risks, constraints, fragile assumptions, high-risk actions requiring confirmation>
- Do not include or expose secrets, tokens, cookies, session IDs, or private page content.
- Treat this handoff as a map, not proof. Verify anchors before acting.
```

## Step 4: Quality Check Before Writing

Before writing, make sure all are true:

- The task is the current conversation, not an older task.
- Workspace, repo root, branch, and HEAD are present or explicitly unknown.
- Changed files say why they matter, not just their names.
- Next steps are concrete and ordered.
- Unknowns and not-verified items are clearly labeled.
- Risky actions say they require user confirmation.
- Sensitive data is redacted.

## Step 5: Write and Report

Write UTF-8 text to:

`C:\Users\11817\.agents\session-handoff.md`

Create `C:\Users\11817\.agents` if needed. After writing, re-read the file briefly to verify it is readable and is about the current task.

If the target path is blocked by permissions, stop and tell the user the exact command needed. Do not write to a hidden substitute path.

After success, reply in one short line:

`已保存到 C:\Users\11817\.agents\session-handoff.md；下个工具运行 /load 或说“接班”即可继续。`
