# cross-agent-handoff

Compact cross-agent handoff skills for Codex, Claude Code, Gemini, and Antigravity.

The goal is not to serialize an entire project into the prompt. The goal is to write a small, reliable map: project identity, exact file anchors, decisions, current state, unknowns, and next safe steps.

## Skills

- `/save` or "交班": writes `C:\Users\11817\.agents\session-handoff.md`
- `/load` or "接班": reads that file, validates the workspace, restores only targeted context, then continues safely

## Why v1.1 Exists

The original version was very token-efficient, but too sparse. A short handoff can cause drift when the next agent:

- resumes in the wrong repo or branch
- misses the project/component boundary
- translates or paraphrases exact identifiers
- treats unverified assumptions as facts
- skips risks that should require confirmation

Version 1.1 keeps the handoff compact while adding:

- project anchors: workspace, repo root, branch, HEAD, key files
- current state split into done, verified, not verified, blocked
- a context rehydration pack with 3-7 files/searches to read first
- explicit unknowns and watch-outs
- load-time validation before editing

## Handoff File

Shared path:

```text
C:\Users\11817\.agents\session-handoff.md
```

Core sections:

```markdown
# Session Handoff
**Time**: <ISO timestamp with timezone>
**Source**: <agent/tool>
**Workspace**: <cwd>
**Repo Root**: <git root>
**Branch / HEAD**: <branch> @ <sha>
**Handoff Profile**: <compact | expanded>
**User Language**: <reply language>

## Current Task
## Project Anchors
## Changed Files
## Decisions
## Current State
## Next Steps
## Context Rehydration Pack
## Watch Out
```

## Install

Clone the source repo:

```bash
git clone https://github.com/k9ight000/cross-agent-handoff.git ~/Desktop/cross-agent-handoff
```

Then copy or symlink `skills/save` and `skills/load` into each agent's skills directory:

```text
~/.agents/skills/save
~/.agents/skills/load
~/.codex/skills/save
~/.codex/skills/load
~/.claude/skills/save
~/.claude/skills/load
~/.antigravity/skills/save
~/.antigravity/skills/load
~/.gemini/skills/save
~/.gemini/skills/load
```

## Usage Loop

```text
working in one agent
  -> /save
  -> switch to another agent
  -> /load or "接班"
  -> agent validates repo/branch/files
  -> agent reads only targeted context
  -> continue
```

## Design Rules

- Compact by default: 50-120 lines.
- Expanded only when needed: up to about 200 lines.
- Never include secrets, cookies, tokens, full logs, full diffs, or private page text.
- Preserve exact identifiers. Do not translate filenames, commands, APIs, selectors, or error strings.
- Treat the handoff as a map, not proof. The receiving agent verifies anchors before acting.
