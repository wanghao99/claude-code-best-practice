# Claude Code：全局与项目级功能对比

> 译自 [reports/claude-global-vs-project-settings.md](../../reports/claude-global-vs-project-settings.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

全面对比 Claude Code 中哪些功能仅限全局（`~/.claude/`），哪些同时具备全局与项目级（`.claude/`）两种形态。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 目录

1. [概述](#overview)
2. [仅全局功能](#global-only-features)
3. [双作用域功能](#dual-scope-features)
4. [设置优先级](#settings-precedence)
5. [目录结构对比](#directory-structure-comparison)
6. [Tasks 系统](#tasks-system)
7. [Agent Teams](#agent-teams)
8. [设计原则](#design-principles)
9. [参考来源](#sources)

---

## 概述

Claude Code 采用**作用域层级**：有些功能同时存在于全局（`~/.claude/`）与项目（`.claude/`）两级，另一些则只在全局存在。设计原则是：属于*个人状态*或*跨项目协调*的东西放在全局；属于*可团队共享的项目配置*的东西可以放在项目级。

- `~/.claude/` 是你的**用户级主目录**（全局，覆盖所有项目）
- 仓库内的 `.claude/` 是你的**项目级主目录**（只作用于该项目）

---

## 仅全局功能

这些功能**只**存在于 `~/.claude/` 下，无法限定到某个项目：

| 功能 | 位置 | 用途 |
|---------|----------|---------|
| **Tasks** | `~/.claude/tasks/` | 跨会话、跨 agent 的持久任务清单 |
| **Agent Teams** | `~/.claude/teams/` | 多 agent 协同配置（实验性，2026 年 2 月） |
| **Auto Memory（自动记忆）** | `~/.claude/projects/<hash>/memory/` | Claude 按项目自我写入的经验（个人专属，绝不共享） |
| **凭据与 OAuth** | 系统钥匙串 + `~/.claude.json` | API key、OAuth token（绝不放进项目文件） |
| **Keybindings（键位绑定）** | `~/.claude/keybindings.json` | 自定义键盘快捷键 |
| **MCP 用户级服务器** | `~/.claude.json`（`mcpServers` 键） | 在所有项目中生效的个人 MCP 服务器 |
| **偏好与缓存** | `~/.claude.json` | 主题、模型、输出样式、会话状态 |

---

## 双作用域功能

这些功能两级都存在，且**项目级优先于**全局：

| 功能 | 全局（`~/.claude/`） | 项目（`.claude/`） | 优先级 |
|---------|----------------------|---------------------|------------|
| **CLAUDE.md** | `~/.claude/CLAUDE.md` | `./CLAUDE.md` 或 `.claude/CLAUDE.md` | 项目覆盖全局 |
| **Settings** | `~/.claude/settings.json` | `.claude/settings.json` + `.claude/settings.local.json` | 项目 > 全局 |
| **Rules** | `~/.claude/rules/*.md` | `.claude/rules/*.md` | 项目覆盖 |
| **Agents/Subagents** | `~/.claude/agents/*.md` | `.claude/agents/*.md` | 项目覆盖 |
| **Commands** | `~/.claude/commands/*.md` | `.claude/commands/*.md` | 两级都可用 |
| **Skills** | `~/.claude/skills/` | `.claude/skills/` | 两级都可用 |
| **Hooks** | `~/.claude/hooks/` | `.claude/hooks/` | 两级都执行 |
| **MCP 服务器** | `~/.claude.json`（user 作用域） | `.mcp.json`（project 作用域） | 三个作用域：local > project > user |

---

## 设置优先级

用户可写的设置按以下覆盖顺序生效（从高到低）：

| 优先级 | 位置 | 作用域 | 版本控制 | 用途 |
|----------|----------|-------|-----------------|---------|
| 1 | 命令行标志 | 会话 | 不适用 | 单次会话覆盖 |
| 2 | `.claude/settings.local.json` | 项目 | 否（git 忽略） | 个人项目专属 |
| 3 | `.claude/settings.json` | 项目 | 是（提交入库） | 团队共享设置 |
| 4 | `~/.claude/settings.local.json` | 用户 | 不适用 | 个人全局覆盖 |
| 5 | `~/.claude/settings.json` | 用户 | 不适用 | 全局个人设置 |

策略层：`managed-settings.json` 由组织强制下发，本地文件无法覆盖。

**重要**：`deny` 规则拥有最高的安全优先级，不能被低优先级的 allow/ask 规则覆盖。

---

## 目录结构对比

### 全局作用域（`~/.claude/`）

```
~/.claude/
├── settings.json              # User-level settings (all projects)
├── settings.local.json        # Personal overrides
├── CLAUDE.md                  # User memory (all projects)
├── agents/                    # User subagents (available to all projects)
│   └── *.md
├── rules/                     # User-level modular rules
│   └── *.md
├── commands/                  # User-level commands
│   └── *.md
├── skills/                    # User-level skills
│   └── */SKILL.md
├── tasks/                     # GLOBAL-ONLY: Task lists
│   └── {task-list-id}/
├── teams/                     # GLOBAL-ONLY: Agent team configs
│   └── {team-name}/
│       └── config.json
├── projects/                  # GLOBAL-ONLY: Per-project auto-memory
│   └── {project-hash}/
│       └── memory/
│           ├── MEMORY.md
│           └── *.md
├── keybindings.json           # GLOBAL-ONLY: Keyboard shortcuts
└── hooks/                     # User-level hooks
    ├── scripts/
    └── config/

~/.claude.json                 # GLOBAL-ONLY: MCP servers, OAuth, preferences, caches
```

### 项目作用域（`.claude/`）

```
.claude/
├── settings.json              # Team-shared settings
├── settings.local.json        # Personal project overrides (git-ignored)
├── CLAUDE.md                  # Project memory (alternative to ./CLAUDE.md)
├── agents/                    # Project subagents
│   └── *.md
├── rules/                     # Project-level modular rules
│   └── *.md
├── commands/                  # Custom slash commands
│   └── *.md
├── skills/                    # Custom skills
│   └── {skill-name}/
│       ├── SKILL.md
│       └── supporting-files/
├── hooks/                     # Project-level hooks
│   ├── scripts/
│   └── config/
└── plugins/                   # Installed plugins

.mcp.json                      # Project-scoped MCP servers (repo root)
```

---

## Tasks 系统

随 **Claude Code v2.1.16**（2026 年 1 月 22 日）引入，取代已废弃的 TodoWrite 系统。

### 存储

任务存储在本地文件系统的 `~/.claude/tasks/`（不是云端数据库）。这让任务状态可审计、可版本控制、可从崩溃中恢复。

### 工具

| 工具 | 用途 |
|------|---------|
| **TaskCreate** | 用 `subject`、`description` 和 `activeForm` 创建新任务 |
| **TaskGet** | 按 ID 获取某个任务的完整详情 |
| **TaskUpdate** | 修改状态、设置 owner、添加依赖或删除 |
| **TaskList** | 列出所有任务及其当前状态 |

### 任务生命周期

```
pending  →  in_progress  →  completed
```

### 依赖管理

任务可以通过 `addBlockedBy`/`addBlocks` 阻塞其他任务，形成依赖图，防止过早执行。

### 多会话协作

```bash
CLAUDE_CODE_TASK_LIST_ID=my-project-tasks claude
```

共享同一 ID 的所有会话都会实时看到任务更新，从而支持并行工作流与会话恢复。

### 与旧 Todos 的关键差异

| 方面 | 旧 Todos | 新 Tasks |
|---------|-----------|-----------|
| 作用域 | 单个会话 | 跨会话、跨 agent |
| 依赖 | 无 | 完整依赖图 |
| 存储 | 仅内存 | 文件系统（`~/.claude/tasks/`） |
| 持久化 | 会话结束即丢失 | 重启和崩溃后仍在 |
| 多会话 | 不支持 | 通过 `CLAUDE_CODE_TASK_LIST_ID` |

---

## Agent Teams

于 **2026 年 2 月 5 日**公布，为实验性功能。Agent Teams 允许多个 Claude Code 会话围绕共享工作协同。

### 启用

```json
// In ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 配置

团队配置位于 `~/.claude/teams/{team-name}/`，支持以下模式：

| 模式 | 描述 | 要求 |
|------|-------------|--------------|
| **进程内**（默认） | 所有队友都在你的终端里运行 | 无 |
| **分屏（split panes）** | 每个队友有自己的窗格 | tmux 或 iTerm2（不支持 VS Code 终端） |

---

## 设计原则

"仅全局 vs 双作用域"的划分遵循清晰的模式：

| 类别 | 作用域 | 理由 |
|----------|-------|-----------|
| **协调状态**（tasks、teams） | 仅全局 | 需要在任何单个项目之外持久存在 |
| **安全状态**（凭据、OAuth） | 仅全局 | 防止被误提交进版本控制 |
| **个人学习**（auto-memory） | 仅全局 | 属于用户个人，不适合团队共享 |
| **输入偏好**（keybindings） | 仅全局 | 用户肌肉记忆，与项目无关 |
| **配置**（settings、rules、agents） | 两级都有 | 团队需要共享项目专属行为 |
| **工作流定义**（commands、skills） | 两级都有 | 可以个人使用，也可以团队共享 |

自动记忆（`~/.claude/projects/<hash>/memory/`）是一个值得注意的混合体：它*关于*某个具体项目，却*存储*在全局，因为它代表的是个人学习，而不是可团队共享的配置。

---

## 参考来源

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Orchestrate Teams of Claude Code Sessions](https://code.claude.com/docs/en/agent-teams)
- [What are Tasks in Claude Code - ClaudeLog](https://claudelog.com/faqs/what-are-tasks-in-claude-code/)
- [Claude Code Task Management - ClaudeFast](https://claudefa.st/blog/guide/development/task-management)
- [Claude Code Tasks Update - VentureBeat](https://venturebeat.com/orchestration/claude-codes-tasks-update-lets-agents-work-longer-and-coordinate-across)
- [Where Are Claude Code Global Settings - ClaudeLog](https://claudelog.com/faqs/where-are-claude-code-global-settings/)
- [Claude Opus 4.6 Agent Teams - VentureBeat](https://venturebeat.com/technology/anthropics-claude-opus-4-6-brings-1m-token-context-and-agent-teams-to-take)
- [How to Set Up Claude Code Agent Teams (Full Walkthrough) - r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1qz8tyy/how_to_set_up_claude_code_agent_teams_full/)
- [Anthropic replaced Claude Code's old 'Todos' with Tasks - r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/comments/1qkjznp/anthropic_replaced_claude_codes_old_todos_with/)
