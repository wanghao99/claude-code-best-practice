# Agent Teams 实现指南

> 译自 [implementation/claude-agent-teams-implementation.md](../../implementation/claude-agent-teams-implementation.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_12%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#time-orchestration"><img src="../../!/tags/implemented-hd.svg" alt="Implemented"></a>

<p align="center">
  <img src="../../implementation/assets/impl-agent-teams.png" alt="Agent Teams in action — split pane mode with tmux" width="100%">
</p>

Agent Teams 会拉起**多个相互独立的 Claude Code 会话**，通过共享任务清单协作。与 subagent（同一会话内隔离的上下文分叉）不同，每个队友都有自己完整的上下文窗口，并自动加载 CLAUDE.md、MCP 服务器和 skill。

---

## ![How to Use](../../!/tags/how-to-use.svg)

时间编排 workflow 完全由一个 agent team 构建而成。运行成品：

```bash
cd agent-teams
claude
/time-orchestrator
```

这会触发 **Command → Agent → Skill** 流水线：agent 获取迪拜当前时间，skill 把 SVG 时间卡片渲染到 `agent-teams/output/dubai-time.svg`。

---

## ![How to Implement](../../!/tags/how-to-implement.svg)

你可以用 agent teams 复刻一份天气编排 workflow — 本例中的时间编排 workflow 就完全由一个 agent team 构建。

### 1. 安装 [iTerm2](https://iterm2.com/) 和 tmux

```bash
brew install --cask iterm2
brew install tmux
```

### 2. 启动 iTerm2 → tmux → Claude

```bash
tmux new -s dev
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

### 3. 用团队结构的提示词引导

<a id="time-orchestration"></a>

把这段提示词粘贴给 Claude，用 agent teams 引导出一个完整的时间编排器 workflow：

主提示词：**[agent-teams-prompt.md](../../agent-teams/agent-teams-prompt.md)**

### 团队协作流程

```
┌──────────────────────────────────────────────────────────────┐
│                         LEAD (You)                           │
│       "Create an agent team to build time orchestration"     │
└──────────────────────────┬───────────────────────────────────┘
                           │ spawns team (all parallel)
              ┌────────────┼────────────┐
              ▼            ▼            ▼
   ┌────────────────┐ ┌──────────┐ ┌──────────────┐
   │ Command        │ │ Agent    │ │ Skill        │
   │ Architect      │ │ Engineer │ │ Designer     │
   │                │ │          │ │              │
   │ agent-teams/   │ │ agent-   │ │ agent-teams/ │
   │ .claude/       │ │ teams/   │ │ .claude/     │
   │ commands/      │ │ .claude/ │ │ skills/      │
   │ time-          │ │ agents/  │ │ time-svg-    │
   │ orchestrator.md│ │ time-    │ │ creator/     │
   │                │ │ agent.md │ │              │
   └───────┬────────┘ └────┬─────┘ └──────┬───────┘
           │               │              │
           ▼               ▼              ▼
   ┌──────────────────────────────────────────────────┐
   │            Shared Task List                      │
   │  ☐ Agree on data contract: {time, tz, formatted} │
   │  ☐ Command uses Agent tool (not bash)            │
   │  ☐ Agent preloads time-fetcher skill             │
   │  ☐ Skill reads time from context (no re-fetch)   │
   │  ☐ All files inside agent-teams/.claude/         │
   └──────────────────────────────────────────────────┘
                       │
                       ▼
          ┌──────────────────────────────┐
          │  cd agent-teams && claude    │
          │    /time-orchestrator        │
          │   Command → Agent → Skill    │
          └──────────────────────────────┘
```
