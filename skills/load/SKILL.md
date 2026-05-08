---
name: load
version: 1.0.0
description: Load a cross-agent session handoff from `C:\Users\11817\.agents\session-handoff.md` and continue the work. Use when starting a new session in Claude Code or Codex after `/save` was run.
---

# /load — Resume Cross-Agent Session

Read the handoff file and resume where the previous session left off.

## Step 1: Check for Handoff

```bash
cat "$HOME/.agents/session-handoff.md" 2>/dev/null || echo "NO_HANDOFF"
```

If `NO_HANDOFF` — reply:

> 没有找到手交班文件。你是从 Codex 切过来的吗？描述一下你在做什么，我直接开始。

Stop there.

## Step 2: Parse and Present

Read the handoff file content. Present a concise summary to the user:

```
从手交班文件读到了：

<Current Task 内容>

Next Steps:
1. ...
2. ...

开始执行 Next Steps 吗？
```

## Step 3: Resume

If user confirms, start working on the Next Steps in order. Reference the Decisions and Changed Files sections as needed.

## Rules

- Don't re-do work already listed as done in Changed Files.
- If the handoff is more than 24 hours old, mention that and ask if anything changed since.
- Once work resumes, you can overwrite the handoff by running `/save` again at any pause point.
