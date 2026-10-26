# Claude Code：Agent Memory frontmatter（subagent 持久记忆）

> 译自 [reports/claude-agent-memory.md](../../reports/claude-agent-memory.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

为 subagent 提供持久记忆——让 agent 能够跨会话学习、记忆并积累知识。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 概述

`memory` frontmatter 字段随 **Claude Code v2.1.33**（2026 年 2 月）引入，它让每个 subagent 都拥有自己基于 markdown 的持久知识存储。在此之前，每次调用 agent 都要从零开始。

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Write, Edit, Bash
model: sonnet
memory: user
---

You are a code reviewer. As you review code, update your agent memory with
patterns, conventions, and recurring issues you discover.
```

---

## 记忆作用域

| 作用域 | 存储位置 | 是否纳入版本控制 | 是否共享 | 适用场景 |
|-------|-----------------|-------------------|--------|----------|
| `user` | `~/.claude/agent-memory/<agent-name>/` | 否 | 否 | 跨项目知识（推荐默认值） |
| `project` | `.claude/agent-memory/<agent-name>/` | 是 | 是 | 需要团队共享的项目专属知识 |
| `local` | `.claude/agent-memory-local/<agent-name>/` | 否（被 git 忽略） | 否 | 项目内的个人知识 |

这些作用域与设置层级（`~/.claude/settings.json` → `.claude/settings.json` → `.claude/settings.local.json`）一一对应。

---

## 工作原理

1. **启动时**：`MEMORY.md` 的前 200 行会被注入 agent 的系统提示词
2. **工具访问**：`Read`、`Write`、`Edit` 会自动启用，agent 因此可以管理自己的记忆
3. **执行过程中**：agent 可自由读写自己的记忆目录
4. **整理**：当 `MEMORY.md` 超过 200 行时，agent 会把细节转移到按主题拆分的文件中

```
~/.claude/agent-memory/code-reviewer/     # user scope example
├── MEMORY.md                              # Primary file (first 200 lines loaded)
├── react-patterns.md                      # Topic-specific file
└── security-checklist.md                  # Topic-specific file
```

---

## Agent Memory 与其他记忆系统对比

| 系统 | 谁写入 | 谁读取 | 作用范围 |
|--------|-----------|-----------|-------|
| **CLAUDE.md** | 你（手动） | 主 Claude 及所有 agent | 项目 |
| **Auto-memory（自动记忆）** | 主 Claude（自动） | 仅主 Claude | 按项目、按用户 |
| **`/memory` command** | 你（通过编辑器） | 仅主 Claude | 按项目、按用户 |
| **Agent memory** | agent 自身 | 仅该特定 agent | 可配置（user/project/local） |

这些系统是**互补**的——agent 既读取 CLAUDE.md（项目上下文），也读取自己的记忆（agent 专属知识）。

---

## 实战示例

```yaml
---
name: api-developer
description: Implement API endpoints following team conventions
tools: Read, Write, Edit, Bash
model: sonnet
memory: project
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions from your preloaded skills.
As you work, save architectural decisions and patterns to your memory.
```

它把 **skill**（启动时加载的静态知识）与 **memory**（随时间积累的动态知识）结合起来。

---

## 使用技巧

- **提示 agent 使用记忆** —— 写入明确指令：`"Before starting, review your memory. After completing, update your memory with what you learned."`
- 调用 agent 时**要求检查记忆**：`"Review this PR, and check your memory for patterns you've seen before."`
- **选择合适的作用域** —— 跨项目用 `user`，团队共享用 `project`，个人知识用 `local`

---

## 参考来源

- [创建自定义 subagent — Claude Code 文档](https://code.claude.com/docs/en/sub-agents)
- [管理 Claude 的记忆 — Claude Code 文档](https://code.claude.com/docs/en/memory)
- [Claude Code v2.1.33 发布说明](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
