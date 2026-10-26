# 视频 1：从 Vibe Coding 到 Agentic Engineering — Claude Code 的 Workflows

> 译自 [!/video-presentation-transcript/1-video-workflow.md](../../../!/video-presentation-transcript/1-video-workflow.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

**总时长：约 5 分钟**

---

## 开场 — 问题所在（0:00 – 0:45）

- "如果你刚开始用 Claude Code，多半在做 vibe coding——输入提示词、拿到结果、再来一次。这能用，但你只发挥了 Claude Code 一小部分的能力。"
- "这个仓库是一套精心整理的最佳实践合集，带你从 vibe coding 走向 agentic engineering——Claude 不只是回应你，它替你运行 workflow。"
- "在第一支视频里，我讲基础：**Command、Agent 和 Skill**——以及它们如何串成可复用的 workflow。"

---

## 第一部分 — 临时拼凑的做法（0:45 – 2:00）

**演示：vibe coding 方式**

- 打开一个全新的 Claude Code 终端
- 输入：*"迪拜天气怎么样？把结果写到输出文件里，并做一张 SVG 卡片。"*
- 展示结果——它能跑，但指出：
  - SVG 的设计每次都不一样（颜色、布局、字体都是随机的）
  - 你得坐在旁边盯着它干活
  - 明天再跑一次，会得到一张长得完全不同的卡片
- **再开一个终端，把同一段提示词再跑一遍**
  - 把两张 SVG 并排展示——它们长得不一样
- "这就是 vibe coding 的问题。它能成功一次。但不可复现。这不是一个你能信任的 workflow。"

---

## 第二部分 — Workflow 的做法（2:00 – 3:15）

**演示：`/weather-orchestrator` 命令**

- "现在给你看同一个任务，换成 workflow 的做法。"
- 输入：`/weather-orchestrator`
- 逐步讲解屏幕上发生的事：
  1. 它**问你**要摄氏还是华氏（结构化的用户交互）
  2. 它**生成一个 weather-agent** 去取温度（你能在终端里看到绿色的 agent）
  3. 它**调用一个 skill** 来创建 SVG 卡片
  4. 输出：`orchestration-workflow/weather.svg` + `orchestration-workflow/output.md`
- "再跑一次——同样的 SVG 布局、同样的文件结构、同样干净的结果。每一次都一样。"
- "你可以发起之后就走开。它自主运行。"

---

## 第三部分 — 原理解析：Command → Agent → Skill（3:15 – 4:30）

**讲解三个构建单元**

### Command（`.claude/commands/`）

- "command 是入口——像一个脚本。它是一个 markdown 文件，告诉 Claude *该按哪些步骤走*。"
- "我们的 `weather-orchestrator` 是指挥。它问用户一个问题，调用一个 agent，再调用一个 skill。"
- Command 放在 `.claude/commands/`，以 `/slash-command` 的形式出现

### Agent（`.claude/agents/`）

- "agent 是一个专职工人。我们的 `weather-agent` 只做一件事：取温度。"
- "它**预置了一个 skill**，叫 `weather-fetcher`——这个 skill 在启动时注入 agent 的上下文，所以它确切知道该调哪个 API、怎么解析响应。"
- Agent 有自己的工具、模型和权限。它们是隔离的工人。

### Skill（`.claude/skills/`）

- "skill 是一组可复用的指令。把它想成一份菜谱。"
- "我们这里有两种 skill 用法："
  - **Agent skill**（预置）：`weather-fetcher` 固化在 agent 里——它是领域知识
  - **被调用的 skill**：`weather-svg-creator` 通过 Skill 工具独立调用——它创建 SVG 卡片
- Skill 既可以是背景知识，也可以是独立动作

### 流程图（可选，在屏幕上展示）

```
/weather-orchestrator (Command)
    → AskUser: C° or F°?
    → weather-agent (Agent + weather-fetcher skill)
    → weather-svg-creator (Skill)
    → Output: weather.svg + output.md
```

---

## 第四部分 — 为什么这很重要 / 收尾（4:30 – 5:00）

- "vibe coding 和 agentic engineering 的区别就是**结构**。"
  - Vibe coding：你输入、你祈祷、你得到点东西。
  - Agentic engineering：你把 workflow 定义一次，它每次都按同样的方式运行。
- "Command、Agent 和 Skill 是三个构建单元。理解了它们，你就能搭任何 workflow。"
- "这个仓库还有更多模式——hook、多 agent 团队、CLAUDE.md 配置——后续视频会讲。"
- "仓库链接在简介里。点个 star，clone 下来，开始搭你自己的 workflow。"

---

## 速查表

| 概念 | 位置 | 用途 |
|---------|----------|---------|
| Command | `.claude/commands/` | 入口、编排、`/slash-command` |
| Agent | `.claude/agents/` | 拥有独立工具和模型的专职工人 |
| Skill | `.claude/skills/` | 可复用指令（预置或被调用） |
