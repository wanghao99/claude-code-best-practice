# 编排工作流（Orchestration Workflow）

> 译自 [orchestration-workflow/orchestration-workflow.md](../../orchestration-workflow/orchestration-workflow.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

本文档描述 **Command → Agent（携带 skill）→ Skill** 的编排工作流，并通过一个天气数据获取与 SVG 渲染系统来演示。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 系统概述

天气系统在单个编排工作流中演示了两种不同的 skill 模式：
- **Agent Skills**（预加载）：`weather-fetcher` 在启动时作为领域知识注入到 `weather-agent` 中
- **Skills**（独立）：`weather-svg-creator` 由 command 通过 Skill 工具直接调用

这展示了 **Command → Agent → Skill** 架构模式，其中：
- command 负责编排工作流并处理用户交互
- agent 使用其预加载的 skill 获取数据
- skill 独立创建可视化输出

## 组件概览

| 组件 | 角色 | 示例 |
|-----------|------|---------|
| **Command** | 入口，用户交互 | [`/weather-orchestrator`](../../.claude/commands/weather-orchestrator.md) |
| **Agent** | 使用预加载 skill（agent skill）获取数据 | [`weather-agent`](../../.claude/agents/weather-agent.md) 搭配 [`weather-fetcher`](../../.claude/skills/weather-fetcher/SKILL.md) |
| **Skill** | 独立创建输出（skill） | [`weather-svg-creator`](../../.claude/skills/weather-svg-creator/SKILL.md) |

## 流程图

```
╔══════════════════════════════════════════════════════════════════╗
║              ORCHESTRATION WORKFLOW                              ║
║           Command  →  Agent  →  Skill                            ║
╚══════════════════════════════════════════════════════════════════╝

                         ┌───────────────────┐
                         │  User Interaction │
                         └─────────┬─────────┘
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  /weather-orchestrator — Command (Entry Point)      │
         └─────────────────────────┬───────────────────────────┘
                                   │
                              Step 1
                                   │
                                   ▼
                      ┌────────────────────────┐
                      │  AskUser — C° or F°?   │
                      └────────────┬───────────┘
                                   │
                         Step 2 — Agent tool
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-agent — Agent ● skill: weather-fetcher     │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          Returns: temp + unit
                                   │
                         Step 3 — Skill tool
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-svg-creator — Skill ● SVG card + output    │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          ┌────────┴────────┐
                          │                 │
                          ▼                 ▼
                   ┌────────────┐    ┌────────────┐
                   │weather.svg │    │ output.md  │
                   └────────────┘    └────────────┘
```

## 组件详情

### 1. Command

#### `/weather-orchestrator`（Command）
- **位置**：`.claude/commands/weather-orchestrator.md`
- **用途**：入口——编排工作流并处理用户交互
- **动作**：
  1. 询问用户温度单位偏好（摄氏/华氏）
  2. 通过 Agent 工具调用 weather-agent
  3. 通过 Skill 工具调用 weather-svg-creator
- **模型**：haiku

### 2. 携带预加载 Skill 的 Agent（Agent Skill）

#### `weather-agent`（Agent）
- **位置**：`.claude/agents/weather-agent.md`
- **用途**：使用其预加载的 skill 获取天气数据
- **Skills**：`weather-fetcher`（作为领域知识预加载）
- **可用工具**：Read、Skill
- **模型**：sonnet
- **颜色**：green
- **记忆**：project

该 agent 在启动时将 `weather-fetcher` 预加载到自己的上下文中。它按照该 skill 的指令获取温度，并把值返回给 command。

### 3. Skill

#### `weather-svg-creator`（Skill）
- **位置**：`.claude/skills/weather-svg-creator/SKILL.md`
- **用途**：创建可视化的 SVG 天气卡片并写入输出文件
- **调用方式**：由 command 通过 Skill 工具调用（不预加载到任何 agent 中）
- **输出**：
  - `orchestration-workflow/weather.svg` — SVG 天气卡片
  - `orchestration-workflow/output.md` — 天气摘要

### 4. 预加载 Skill

#### `weather-fetcher`（Skill）
- **位置**：`.claude/skills/weather-fetcher/SKILL.md`
- **用途**：获取实时温度数据的指令
- **数据源**：Open-Meteo API（阿联酋迪拜）
- **输出**：温度值和单位（摄氏或华氏）
- **说明**：这是一个 agent skill——预加载到 `weather-agent` 中，不被直接调用

## 执行流程

1. **用户调用**：用户运行 `/weather-orchestrator` command
2. **用户提示**：command 询问用户偏好的温度单位（摄氏/华氏）
3. **Agent 调用**：command 通过 Agent 工具调用 `weather-agent`
4. **Skill 执行**（在 agent 上下文内）：
   - agent 按照 `weather-fetcher` skill 的指令从 Open-Meteo 获取温度
   - agent 将温度值和单位返回给 command
5. **SVG 创建**：command 通过 Skill 工具调用 `weather-svg-creator`
   - skill 在 `orchestration-workflow/weather.svg` 创建 SVG 天气卡片
   - skill 将摘要写入 `orchestration-workflow/output.md`
6. **结果展示**：向用户展示摘要，包括：
   - 所请求的温度单位
   - 获取到的温度
   - SVG 卡片位置
   - 输出文件位置

## 执行示例

```
Input: /weather-orchestrator
├─ Step 1: Asks: Celsius or Fahrenheit?
│  └─ User: Celsius
├─ Step 2: Agent tool → weather-agent
│  ├─ Preloaded Skill:
│  │  └─ weather-fetcher (domain knowledge)
│  ├─ Fetches from Open-Meteo → 26°C
│  └─ Returns: temperature=26, unit=Celsius
├─ Step 3: Skill tool → /weather-svg-creator
│  ├─ Creates: orchestration-workflow/weather.svg
│  └─ Writes: orchestration-workflow/output.md
└─ Output:
   ├─ Unit: Celsius
   ├─ Temperature: 26°C
   ├─ SVG: orchestration-workflow/weather.svg
   └─ Summary: orchestration-workflow/output.md
```

## 关键设计原则

1. **两种 Skill 模式**：同时演示 agent skills（预加载）和 skills（直接调用）
2. **Command 作为编排者**：command 处理用户交互并协调工作流
3. **Agent 负责数据获取**：agent 使用其预加载的 skill 获取数据，然后返回
4. **Skill 负责输出**：SVG 创建器独立运行，从 command 上下文接收数据
5. **清晰分离**：获取（agent）→ 渲染（skill）——每个组件单一职责

## 架构模式

### Agent Skill（预加载）

```yaml
# In agent definition (.claude/agents/weather-agent.md)
---
name: weather-agent
skills:
  - weather-fetcher    # Preloaded into agent context at startup
---
```

- **Skill 被预加载**：完整 skill 内容在启动时注入 agent 的上下文
- **Agent 使用 skill 知识**：agent 遵循预加载 skill 的指令
- **无动态调用**：skills 是参考材料，不会被单独调用

### Skill（直接调用）

```yaml
# In skill definition (.claude/skills/weather-svg-creator/SKILL.md)
---
name: weather-svg-creator
description: Creates an SVG weather card...
---
```

- **通过 Skill 工具调用**：command 调用 `Skill(skill: "weather-svg-creator")`
- **独立执行**：在 command 的上下文中运行，不在 agent 内部
- **从上下文接收数据**：使用会话中已可用的温度数据
