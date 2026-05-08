# cross-agent-handoff

> 跨 AI 编码工具的一键会话交接 — Codex、Claude Code、Gemini/Antigravity 之间无缝切换。

## 解决什么问题

用 Codex 写到一半额度耗尽，或者 Claude Code 的上下文快满了——想在另一个工具里接着干，但会话状态全丢了。

`/save` 10 秒内把当前任务、改动文件、决策、下一步写到一个共享文件，切到另一个工具 `/load` 立刻继续。

## 安装

```bash
git clone https://github.com/k9ight000/cross-agent-handoff.git ~/Desktop/cross-agent-handoff
```

然后将 skills 目录 symlink 到各工具的 skills 目录：

```bash
# Claude Code
ln -s ~/Desktop/cross-agent-handoff/skills/save ~/.claude/skills/save
ln -s ~/Desktop/cross-agent-handoff/skills/load ~/.claude/skills/load

# Codex
ln -s ~/Desktop/cross-agent-handoff/skills/save ~/.codex/skills/save
ln -s ~/Desktop/cross-agent-handoff/skills/load ~/.codex/skills/load

# Google Antigravity
ln -s ~/Desktop/cross-agent-handoff/skills/save ~/.antigravity/skills/save
ln -s ~/Desktop/cross-agent-handoff/skills/load ~/.antigravity/skills/load

# Gemini / Google IDE
ln -s ~/Desktop/cross-agent-handoff/skills/save ~/.gemini/skills/save
ln -s ~/Desktop/cross-agent-handoff/skills/load ~/.gemini/skills/load
```

再把启动自动检查策略写入跨工具配置 `~/.agents/AGENTS.md`：

```markdown
## Session Handoff Policy

At session startup, check for a handoff file:
`C:\Users\<user>\.agents\session-handoff.md`

If the file exists and was written within the last 24 hours:
- Read it and present a brief summary
- Ask whether to continue from the Next Steps listed
```

## 用法

```
写代码中... 感觉到额度快不够了
    ↓
/save          →  存档到 ~/.agents/session-handoff.md
    ↓
切到另一个工具
    ↓
/load          →  自动读取、展示摘要、确认后继续干活
```

**关键**：`/save` 要在额度还活着的时候跑。如果已经耗尽没来得及保存 — Codex 的会话 JSONL 文件在 `~/.codex/sessions/` 里，另一个工具可以直接读取来重建上下文。

## 手交班文件格式

`~/.agents/session-handoff.md`：

```markdown
# Session Handoff
**Time**: 2026-05-09T01:40+08:00
**Source**: Claude Code

## Current Task
<1-2 lines>

## Changed Files
<git status summary>

## Decisions
- <decision + why>

## Next Steps
1. <step 1>
2. <step 2>
```

## 设计原则

- **快**：不读文件、不跑测试、不联网，控制在 10 秒 / 500 tokens 内
- **简**：一个共享文件，覆盖写入，不是日志系统
- **跨工具**：只依赖文件系统，不依赖任何工具的专有 API
- **兜底**：即使忘了 /save，Codex 的会话 JSONL 也是可读的

## License

MIT
