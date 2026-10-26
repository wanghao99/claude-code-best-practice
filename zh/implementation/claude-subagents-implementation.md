# Subagents 实现指南

> 译自 [implementation/claude-subagents-implementation.md](../../implementation/claude-subagents-implementation.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_02%2C_2026_07%3A59_PM_PKT-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#weather-agent"><img src="../../!/tags/implemented-hd.svg" alt="Implemented"></a>

天气 agent 在本仓库中作为 **Command → Agent → Skill** 架构模式的示例实现，演示了两种不同的 skill 模式。

---

## Weather Agent

**文件**：[`.claude/agents/weather-agent.md`](../../.claude/agents/weather-agent.md)

```yaml
---
name: weather-agent
description: Use this agent PROACTIVELY when you need to fetch weather data for
  Dubai, UAE. This agent fetches real-time temperature from Open-Meteo
  using its preloaded weather-fetcher skill.
allowedTools:
  - "Read"
  - "Skill"
model: sonnet
color: green
maxTurns: 5
permissionMode: acceptEdits
memory: project
skills:
  - weather-fetcher
---

# Weather Agent

You are a specialized weather agent that fetches weather data for Dubai,
UAE.

## Your Task

Execute the weather workflow by following the instructions from your preloaded
skill:

1. **Fetch**: Follow the `weather-fetcher` skill instructions to fetch the
   current temperature
2. **Report**: Return the temperature value and unit to the caller
3. **Memory**: Update your agent memory with the reading details for
   historical tracking

...
```

该 agent 预载了一个 skill（`weather-fetcher`），提供从 Open-Meteo 取数的指令。它把温度值和单位返回给调用它的 command。

---

## ![How to Use](../../!/tags/how-to-use.svg)

```bash
$ claude
> what is the weather in dubai?
```

---

## ![How to Implement](../../!/tags/how-to-implement.svg)

你可以用 `/agents` 命令创建 agent：
```bash
$ claude
> /agents
```

或者让 Claude 帮你创建 — 它会在 `.claude/agents/<name>.md` 中生成带 YAML frontmatter 和正文的 markdown 文件。

---

<a href="https://github.com/shanraisshan/claude-code-best-practice#orchestration-workflow"><img src="../../!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

天气 agent 是 Command → Agent → Skill 编排模式中的 **Agent**。它接收 `/weather-orchestrator` command 下发的 workflow，用预载的 skill（`weather-fetcher`）取回温度，随后 command 再调用独立的 `weather-svg-creator` skill 生成可视化输出。

<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.svg" alt="Command Skill Agent Architecture Flow" width="100%">
</p>

| 组件 | 角色 | 本仓库实现 |
|------|------|-----------|
| **Command** | 入口，负责用户交互 | [`/weather-orchestrator`](../../.claude/commands/weather-orchestrator.md) |
| **Agent** | 用预载 skill 取数据（agent skill） | [`weather-agent`](../../.claude/agents/weather-agent.md) + [`weather-fetcher`](../../.claude/skills/weather-fetcher/SKILL.md) |
| **Skill** | 独立生成输出（skill） | [`weather-svg-creator`](../../.claude/skills/weather-svg-creator/SKILL.md) |
