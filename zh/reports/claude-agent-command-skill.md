# Agents vs Commands vs Skills — 什么时候用哪个

> 译自 [reports/claude-agent-command-skill.md](../../reports/claude-agent-command-skill.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。
> Agent、Command、Skill 是 Claude Code 的三个机制名，本文保留英文原文，不译为"代理/命令/技能"。

对比 Claude Code 的三种扩展机制：subagent（子代理）、command（命令）、skill（技能）。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

![斜杠菜单中展示 time-skill、time-command、time-agent](../../reports/assets/agent-command-skill-1.jpg)

---

## 一览表

| | Agent | Command | Skill |
|---|---|---|---|
| **存放位置** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **上下文** | 独立的 subagent 进程 | 内联（主对话） | 内联（主对话） |
| **用户可直接调用** | 不出现在 `/` 菜单 — 由 Claude 或通过 Agent 工具调用 | 可以 — `/command-name` | 可以 — `/skill-name`（除非 `user-invocable: false`） |
| **Claude 自动调用** | 可以 — 通过 `description` 字段触发 | 不可以 | 可以 — 通过 `description` 字段触发（除非 `disable-model-invocation: true`） |
| **接收参数** | 通过 `prompt` 参数 | `$ARGUMENTS`、`$0`、`$1` | `$ARGUMENTS`、`$0`、`$1` |
| **动态注入上下文** | 不支持 | 支持 — `` !`command` `` | 支持 — `` !`command` `` |
| **独立上下文窗口** | 有 — 完全隔离 | 无 — 共享主上下文 | 无 — 共享主上下文（除非 `context: fork`） |
| **模型覆盖** | `model:` frontmatter | `model:` frontmatter | `model:` frontmatter |
| **工具限制** | `tools:` / `disallowedTools:` | `allowed-tools:` | `allowed-tools:` |
| **Hooks** | `hooks:` frontmatter | — | `hooks:` frontmatter |
| **持久记忆** | `memory:` frontmatter（user/project/local） | — | — |
| **可预载 skill** | 可以 — `skills:` frontmatter | — | — |
| **MCP 服务器** | `mcpServers:` frontmatter | — | — |

---

## 分别在什么时候用

### 用 Agent 的场景：

- 任务是**自主且多步骤的** — agent 需要在没有持续指导的情况下探索、决策和执行
- 你需要**上下文隔离** — 这部分工作不应该污染主对话窗口
- agent 需要**跨会话的持久记忆**（例如一个会积累代码模式的 code reviewer）
- 你想通过 skill **预载领域知识**，又不弄乱主上下文
- 任务适合**后台运行**或在独立 **git worktree** 中运行
- 你需要**工具限制**或**不同的权限模式**（如 `acceptEdits`、`plan`）

**示例**：`weather-agent` — 用预载的 `weather-fetcher` skill 自主获取天气数据，在受限工具的独立上下文中运行。

### 用 Command 的场景：

- 你需要一个**用户主动触发的入口** — 一个由用户显式启动的工作流
- 这个工作流要负责**编排**其他 agent 或 skill
- 你想**保持上下文精简** — command 的内容在用户触发前不会注入会话上下文

**示例**：`weather-orchestrator` — 用户触发它，它询问摄氏/华氏偏好，调用 agent，再调用 SVG skill。

### 用 Skill 的场景：

- 你希望 **Claude 根据用户意图自动调用** — skill 的 description 会注入会话上下文用于语义匹配
- 任务是一个**可复用的流程**，可以从多个地方调用（command、agent 或 Claude 本身）
- 你需要**agent 预载** — 把领域知识在启动时就固化进某个特定 agent

**示例**：`weather-svg-creator` — 用户要天气卡片时 Claude 自动调用；也可以从 command 中调用。

---

## Command → Agent → Skill 架构

本仓库演示了一个分层编排模式：

```
用户触发 /command
    ↓
Command 编排整个工作流
    ↓
Command 调用 Agent（独立上下文，自主执行）
    ↓
Agent 使用预载的 Skill（领域知识）
    ↓
Command 调用 Skill（内联，生成输出）
```

**具体例子** — 天气系统：

```
/weather-orchestrator（command — 入口，询问摄氏/华氏）
    ↓
weather-agent（agent — 自主获取温度）
    ├── weather-fetcher（agent skill — 预载的 API 调用说明）
    ↓
weather-svg-creator（skill — 内联生成 SVG）
```

---

## Frontmatter 对比

### Agent Frontmatter

```yaml
---
name: my-agent
description: Use this agent PROACTIVELY when...
tools: Read, Write, Edit, Bash
model: sonnet
maxTurns: 10
permissionMode: acceptEdits
memory: user
skills:
  - my-skill
---
```

### Command Frontmatter

```yaml
---
description: Do something useful
argument-hint: [issue-number]
allowed-tools: Read, Edit, Bash(gh *)
model: sonnet
---
```

### Skill Frontmatter

```yaml
---
name: my-skill
description: Do something when the user asks for...
argument-hint: [file-path]
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Grep, Glob
model: sonnet
context: fork
agent: general-purpose
---
```

---

## 关键区别

### 自动调用

| 机制 | Claude 能自动调用吗 | 如何阻止 |
|-----------|------------------------|----------------|
| Agent | 能 — 通过 `description`（写上 "PROACTIVELY" 可以鼓励触发） | 删除或弱化 description |
| Command | 不能 — 只能用户通过 `/` 触发 | 不适用 |
| Skill | 能 — 通过 `description` | 设置 `disable-model-invocation: true` |

### 在 `/` 菜单中的可见性

| 机制 | 出现在 `/` 菜单？ | 如何隐藏 |
|-----------|---------------------|-------------|
| Agent | 否 | 不适用 |
| Command | 是 — 始终可见 | 无法隐藏 |
| Skill | 默认可见 | 设置 `user-invocable: false` |

### 上下文隔离

| 机制 | 在独立上下文运行？ | 如何配置 |
|-----------|---------------------|-----------------|
| Agent | 始终如此 | 内置行为 |
| Command | 从不 | 不适用 |
| Skill | 可选 | 设置 `context: fork` |

---

## 实例演算："现在几点了？"

本仓库为同一个任务定义了全部三种机制 — 显示 PKT（巴基斯坦标准时间）的当前时间。当用户输入**"现在几点了？"**而未显式调用任何 `/` 命令时：

| 机制 | 会触发吗 | 原因 |
|-----------|--------------|---------------|
| `time-command` | 不会 | Command **永远不会被自动调用**。用户必须显式输入 `/time-command` 才能运行。Command 没有自动发现路径 — 严格由用户触发。 |
| `time-agent` | **可能** | agent 的 `description` 写着 *"Use this agent to display the current time in Pakistan Standard Time"*。Claude 将其与用户意图匹配后，可能通过 Agent 工具生成它。但 agent 运行在**独立上下文窗口**中，对这么简单的任务来说过重了。 |
| `time-skill` | **最可能** | skill 的 `description` 写着 *"Display the current time in Pakistan Standard Time (PKT, UTC+5). Use when the user asks for the current time, Pakistan time, or PKT."* Claude 匹配后通过 Skill 工具调用。它**内联**运行、零上下文开销，是最高效的匹配。 |

### 决议顺序

多个机制匹配同一意图时，Claude 倾向于选择能满足请求的**最轻量选项**：

```
1. Skill（内联，无上下文开销）          ← 首选
2. Agent（独立上下文，自主执行）        ← skill 不可用或任务复杂时使用
3. Command（不会自动触发 — 需显式 /）  ← 仅当用户输入 /time-command
```

### 如果给 skill 设置了 `disable-model-invocation: true` 会怎样？

那 Claude 就**不能**自动调用这个 skill 了。agent 成为唯一可自动调用的选项，于是 Claude 会改为生成 `time-agent` — 代价是为一条一行 bash 命令开一个独立上下文窗口。

### 如果 skill 和 agent 都禁用了自动调用呢？

那就**什么都不会自动触发**。Claude 会退回到自身通用知识，大概率直接执行 `TZ='Asia/Karachi' date` — 完全不经过任何扩展机制。用户必须显式输入 `/time-command` 或 `/time-skill` 才能使用它们。

![用户问"现在几点了"时 Claude 自动调用 time-skill](../../reports/assets/agent-command-skill-2.png)

---

## 参考来源

- [Claude Code Skills — 官方文档](https://code.claude.com/docs/en/skills)
- [Claude Code Sub-agents — 官方文档](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Slash Commands — 官方文档](https://code.claude.com/docs/en/slash-commands)
- [Skills 最佳实践](../../best-practice/claude-skills.md)（英文原文）
- [Commands 最佳实践](../../best-practice/claude-commands.md)（英文原文）
- [Sub-agents 最佳实践](../../best-practice/claude-subagents.md)（英文原文）
