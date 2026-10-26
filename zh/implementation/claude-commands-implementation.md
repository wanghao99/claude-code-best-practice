# Commands 实现指南

> 译自 [implementation/claude-commands-implementation.md](../../implementation/claude-commands-implementation.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_02%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#weather-orchestrator"><img src="../../!/tags/implemented-hd.svg" alt="Implemented"></a>

天气编排器 command 在本仓库中作为 **Command → Agent → Skill** 架构模式的入口实现，演示了 command 如何编排多步骤 workflow。

---

## Weather Orchestrator

**文件**：[`.claude/commands/weather-orchestrator.md`](../../.claude/commands/weather-orchestrator.md)

```yaml
---
description: Fetch weather data for Dubai and create an SVG weather card
model: haiku
---

# Weather Orchestrator Command

Fetch the current temperature for Dubai, UAE and create a visual SVG weather card.

## Workflow

### Step 1: Ask User Preference
Use the AskUserQuestion tool to ask the user whether they want the temperature
in Celsius or Fahrenheit.

### Step 2: Fetch Weather Data
Use the Agent tool to invoke the weather agent:
- subagent_type: weather-agent
- prompt: Fetch the current temperature for Dubai, UAE in [unit]...

### Step 3: Create SVG Weather Card
Use the Skill tool to invoke the weather-svg-creator skill:
- skill: weather-svg-creator

...
```

这个 command 负责编排整个 workflow：先询问用户偏好的温度单位，再通过 Agent tool 调用 `weather-agent`，最后通过 Skill tool 调用 `weather-svg-creator` skill。

---

## ![How to Use](../../!/tags/how-to-use.svg)

```bash
$ claude
> /weather-orchestrator
```

---

## ![How to Implement](../../!/tags/how-to-implement.svg)

让 Claude 帮你创建 — 它会在 `.claude/commands/<name>.md` 中生成带 YAML frontmatter 和正文的 markdown 文件。

---

<a href="https://github.com/shanraisshan/claude-code-best-practice#orchestration-workflow"><img src="../../!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

天气编排器是 Command → Agent → Skill 编排模式中的 **Command**。它作为入口，负责用户交互（温度单位偏好），把取数工作委派给 `weather-agent`，再调用 `weather-svg-creator` skill 生成可视化输出。

<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.svg" alt="Command Skill Agent Architecture Flow" width="100%">
</p>

| 组件 | 角色 | 本仓库实现 |
|------|------|-----------|
| **Command** | 入口，负责用户交互 | [`/weather-orchestrator`](../../.claude/commands/weather-orchestrator.md) |
| **Agent** | 用预载 skill 取数据（agent skill） | [`weather-agent`](../../.claude/agents/weather-agent.md) + [`weather-fetcher`](../../.claude/skills/weather-fetcher/SKILL.md) |
| **Skill** | 独立生成输出（skill） | [`weather-svg-creator`](../../.claude/skills/weather-svg-creator/SKILL.md) |
