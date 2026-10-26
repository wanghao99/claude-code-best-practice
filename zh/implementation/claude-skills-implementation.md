# Skills 实现指南

> 译自 [implementation/claude-skills-implementation.md](../../implementation/claude-skills-implementation.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_02%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#weather-svg-creator"><img src="../../!/tags/implemented-hd.svg" alt="Implemented"></a>

本仓库实现了两个 skill，作为 **Command → Agent → Skill** 架构模式的一部分，演示了两种不同的 skill 调用模式：**agent skill**（预载）和 **skill**（直接调用）。

---

## Weather SVG Creator（Skill）

**文件**：[`.claude/skills/weather-svg-creator/SKILL.md`](../../.claude/skills/weather-svg-creator/SKILL.md)

```yaml
---
name: weather-svg-creator
description: Creates an SVG weather card showing the current temperature for
  Dubai. Writes the SVG to orchestration-workflow/weather.svg and updates
  orchestration-workflow/output.md.
---

# Weather SVG Creator Skill

This skill creates a visual SVG weather card and writes the output files.

## Task
Create an SVG weather card displaying the temperature for Dubai, UAE,
and write it along with a summary to output files.

## Instructions
You will receive the temperature value and unit (Celsius or Fahrenheit)
from the calling context.

### 1. Create SVG Weather Card
Generate a clean SVG weather card...

### 2. Write SVG File
Write the SVG content to `orchestration-workflow/weather.svg`.

### 3. Write Output Summary
Write to `orchestration-workflow/output.md`...

...
```

这是一个 **skill** — 由 command 通过 Skill tool 直接调用。它从会话上下文拿到温度数据，生成 SVG 天气卡片和输出摘要。

---

## Weather Fetcher（Agent Skill）

**文件**：[`.claude/skills/weather-fetcher/SKILL.md`](../../.claude/skills/weather-fetcher/SKILL.md)

```yaml
---
name: weather-fetcher
description: Instructions for fetching current weather temperature data
  for Dubai, UAE from Open-Meteo API
user-invocable: false
---

# Weather Fetcher Skill

This skill provides instructions for fetching current weather data.

## Task
Fetch the current temperature for Dubai, UAE in the requested unit
(Celsius or Fahrenheit).

## Instructions
1. Fetch Weather Data: Use the WebFetch tool to get current weather data
   - Celsius URL: https://api.open-meteo.com/v1/forecast?latitude=25.2048&longitude=55.2708&current=temperature_2m&temperature_unit=celsius
   - Fahrenheit URL: https://api.open-meteo.com/v1/forecast?latitude=25.2048&longitude=55.2708&current=temperature_2m&temperature_unit=fahrenheit
2. Extract Temperature: From the JSON response, extract `current.temperature_2m`
3. Return Result: Return the temperature value and unit clearly.

...
```

这是一个 **agent skill** — 通过 `skills:` frontmatter 字段在启动时预载进 `weather-agent`。它不直接被调用，而是作为领域知识注入 agent 的上下文。注意 `user-invocable: false` 会让它不出现在 `/` 命令菜单里。

---

## 两种 Skill 模式

| 模式 | 调用方式 | 示例 | 关键区别 |
|---------|-----------|---------|----------------|
| **Skill** | `Skill(skill: "name")` | `weather-svg-creator` | 通过 Skill tool 直接调用 |
| **Agent Skill** | 通过 `skills:` 字段预载 | `weather-fetcher` | 启动时注入 agent 上下文 |

---

## ![How to Use](../../!/tags/how-to-use.svg)

**Skill** — 用斜杠命令直接调用：
```bash
$ claude
> /weather-svg-creator
```

---

## ![How to Implement](../../!/tags/how-to-implement.svg)

让 Claude 帮你创建 — 它会在 `.claude/skills/my-skill/SKILL.md` 中生成带 YAML frontmatter 和正文的 markdown 文件。

# My Skill

Instructions for what the skill does.
```
