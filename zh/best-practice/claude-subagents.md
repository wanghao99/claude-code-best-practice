# Subagents 最佳实践

> 译自 [best-practice/claude-subagents.md](../../best-practice/claude-subagents.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。
> 原文同步至 Claude Code v2.1.310，最后更新 2026-10-25。

![Last Updated](https://img.shields.io/badge/Last_Updated-Oct%2025%2C_2026%2011%3A33%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.310-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../implementation/claude-subagents-implementation.md)

Claude Code subagent（子代理）— frontmatter 字段与官方内置 agent 类型。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter 字段（16 个）

| 字段 | 类型 | 必填 | 说明 |
|-------|------|----------|-------------|
| `name` | string | 是 | 唯一标识符，使用小写字母和连字符 |
| `description` | string | 是 | 何时调用。写上 `"PROACTIVELY"` 可让 Claude 自动调用 |
| `tools` | string/list | 否 | 允许使用的工具白名单，逗号分隔（如 `Read, Write, Edit, Bash`）。省略时继承全部工具。支持 `Agent(agent_type)` 语法来限制可生成的子 agent；旧的 `Task(agent_type)` 别名仍然有效 |
| `disallowedTools` | string/list | 否 | 禁用的工具，从继承或指定的列表中移除 |
| `model` | string | 否 | 使用的模型：`sonnet`、`opus`、`haiku`、完整模型 ID（如 `claude-opus-4-6`）或 `inherit`（默认：`inherit`） |
| `permissionMode` | string | 否 | 权限模式：`default`、`acceptEdits`、`auto`、`dontAsk`、`bypassPermissions` 或 `plan` |
| `maxTurns` | integer | 否 | subagent 停止前允许的最大轮数 |
| `skills` | list | 否 | 启动时预载进 agent 上下文的 skill 名称（注入完整内容，而不只是声明可用） |
| `mcpServers` | list | 否 | 该 subagent 专用的 MCP 服务器 — 服务器名字符串或内联 `{name: config}` 对象 |
| `hooks` | object | 否 | 作用于该 subagent 的生命周期钩子。支持所有钩子事件；最常用的是 `PreToolUse`、`PostToolUse` 和 `Stop` |
| `memory` | string | 否 | 持久记忆作用域：`user`、`project` 或 `local` |
| `background` | boolean | 否 | 设为 `true` 时始终作为后台任务运行（默认：`false`） |
| `effort` | string | 否 | 该 subagent 激活时覆盖思考强度：`low`、`medium`、`high`、`xhigh`、`max`（仅 Opus 4.6）。默认继承会话设置 |
| `isolation` | string | 否 | 设为 `"worktree"` 时在临时 git worktree 中运行（无改动则自动清理） |
| `initialPrompt` | string | 否 | 当该 agent 作为主会话 agent 运行时（通过 `--agent` 或 `agent` 设置），自动作为第一条用户消息提交。其中的 command 和 skill 会被处理。前置在任何用户输入之前 |
| `color` | string | 否 | 该 subagent 在任务列表和对话记录中的显示颜色：`red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink` 或 `cyan` |

---

## 官方内置 Agent（5 个）

| # | Agent | 模型 | 工具 | 说明 |
|---|-------|-------|-------|-------------|
| 1 | `general-purpose` | inherit | 全部 | 复杂的多步骤任务 — 研究、代码搜索和自主工作的默认 agent 类型 |
| 2 | `Explore` | haiku | 只读（无 Write、Edit） | 快速的代码库搜索与探索 — 为查找文件、搜索代码、回答代码库问题优化 |
| 3 | `Plan` | inherit | 只读（无 Write、Edit） | plan 模式下的预规划研究 — 在写代码前探索代码库、设计实现方案 |
| 4 | `statusline-setup` | sonnet | Read、Edit | 配置用户的 Claude Code 状态栏 |
| 5 | `claude-code-guide` | haiku | Glob、Grep、Read、WebFetch、WebSearch | 回答关于 Claude Code 功能、Agent SDK 和 Claude API 的问题 |

---

## 参考来源

- [创建自定义 subagent — Claude Code 官方文档](https://code.claude.com/docs/en/sub-agents)
- [CLI 参考 — Claude Code 官方文档](https://code.claude.com/docs/en/cli-reference)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
