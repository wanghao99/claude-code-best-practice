# 学习之旅——天气播报员 Agent 重设计方案

> 译自 [reports/learning-journey-weather-reporter-redesign.md](../../reports/learning-journey-weather-reporter-redesign.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

← 返回 [README](../../README.md)

## 概述

从第 7 页开始的所有幻灯片都围绕同一个贯穿案例重新设计：**天气播报员 agent（weather reporter agent）**。叙事线与 TOC 的可见顺序一致（Agents → Skills → Context → CLAUDE.md → Commands+Workflow），让听众先认识天气播报员，再了解他们知道什么、怎么思考、遵循什么规则，最后学会用一条命令触发他们。

---

## 1. 现有章节 → 新章节映射

| 现有章节 | 现有页码 | 动作 | 新位置 |
|---|---|---|---|
| Topic 1: Context | 7-11（分节页在 7） | 移到 Topic 3 | 第 17-21 页 |
| Topic 2: CLAUDE.md | 12-17（分节页在 12） | 移到 Topic 4 | 第 22-27 页 |
| Topic 3: Agents | 18-23（分节页在 18） | 移到 Topic 1 | 第 7-12 页 |
| Topic 4: Skills | 24-29（分节页在 24） | 移到 Topic 2 | 第 13-18 页 |
| Topic 5: Commands | 30-32（分节页在 30） | 与 Workflow 合并为 Topic 5 | 第 28-32 页 |
| Topic 6: Workflow | 33-36（分节页在 33） | 并入 Commands 节 | （不再有独立分节页） |
| 收尾页 | 37 | 保留，更新副标题 | 第 33 页 |

**新总数：33 页**（当前 37 页，减去 Workflow 分节页，再减去并入 Commands 节的 3 页 Workflow 内容页，Commands 节则多出这 3 页）。

等等——重新数一遍：

当前：第 7-37 页 = 31 页。
- Agents 节：6 页（18-23）→ 变为 Topic 1（7-12）
- Skills 节：6 页（24-29）→ 变为 Topic 2（13-18）
- Context 节：5 页（7-11）→ 变为 Topic 3（19-23）
- CLAUDE.md 节：6 页（12-17）→ 变为 Topic 4（24-29）
- Commands+Workflow 合并：3 + 1 分节页 + 4 内容页 = Commands（3）+ Workflow（1 分节页 + 3 内容页）= 7 页 → 变为 Topic 5（30-36）
- 收尾：1 页（37）

**新总数：37 页。**（没有删除任何页；Workflow 分节页成为合并后的 Commands+Workflow 节的一部分——保留为子节，或去掉它的 `data-level`，避免出现第二个分节页。）

**决定**：保留全部 37 页。去掉旧 Workflow 分节页（第 33 页）上的 `data-level`，让它被当作内容页而不是分节页。Commands 节覆盖 30-36 页，原 Workflow 分节页成为 Commands 节内部的视觉"章节头"。

其实更简单的做法：把 Workflow 分节页保留为没有 `data-level` 的内容页，journey 进度条停在 `commands` 层级，节号文本从 "Topic 6" 改成一个普通小标题。

---

## 2. 新的 LEVELS 映射（键与颜色不变）

新章节顺序是：**Agents → Skills → Context → CLAUDE.md → Commands**。`workflow` 层级键不再用于 `data-level`（分节页失去 `data-level`）。`LEVELS` 映射仍保留 `workflow`，供 journey 进度条的历史显示使用，只是没有任何页面触发它。

**修订方案**：既然没有任何页面带 `data-level="workflow"`，就把 `workflow` 层级从 LEVELS 映射中整个去掉。journey 进度条最高到 `commands`（83%）。这没问题——Workflow 一节作为高潮*放在* Commands 节内部呈现，不再是独立主题。

不过，收尾阶段进度条只到 83% 而不是 100%，观感不佳。更好的方案：**把 Commands 与 Workflow 合并成一节，叫 "Commands & Workflow"**，用 `data-level="commands"`。`workflow` 层级保留在 LEVELS 的 100% 位置，并把 `data-level="workflow"` 指派给*原来的* Workflow 分节页——它成为 Commands 节内部的视觉过渡。这样进度条到 workflow 幻灯片时填满 100%。

**最终决定**：LEVELS 里同时保留 `commands`（83%）和 `workflow`（100%）。Commands 分节页标 `data-level="commands"`，Workflow 子节页标 `data-level="workflow"`。journey 刻度保持原样。这与现有结构完全一致——只是内容页重新排序。

---

## 3. 逐页内容大纲

### 第 1-6 页（不变）

第 1 页（标题）、2（Boris GIF）、3（Vibe→Agentic）、4（What is Vibe Coding）、5（Good vs Bad Prompts）、6（TOC——仅更新 goToSlide 目标）。

**第 6 页 TOC 更新：**
- Agents 行：`goToSlide(7)`（原 18）
- Skills 行：`goToSlide(13)`（原 24）
- Context 行：`goToSlide(19)`（原 7）
- CLAUDE.md 行：`goToSlide(25)`（原 12）
- Commands 行：`goToSlide(30)`（原 30——不变）

---

### 第 1 节：Agents（第 7-12 页）——"这个人"

**第 7 页** — 分节页（`data-level="agents"`，Topic 1）
- 标题："Agents — The Weather Reporter"
- 描述："An agent is Claude playing a specific role. Meet the weather reporter — a specialist hired to fetch and report weather data for Dubai."（agent 就是 Claude 扮演一个特定角色。来认识天气播报员——一位受雇为迪拜获取并播报天气数据的专员。）

**第 8 页** — "The Restaurant Kitchen"（餐厅后厨，当前第 19 页）
- 内容：沿用同一个类比（裸提示 = 在任意一家后厨里瞎喊；agent = 一位明确的专员）
- 把 agent 示例整体改成 "weather reporter" 的表述
- 保留对比裸提示 vs 天气 agent 的双栏卡片

**第 9 页** — "Prompting vs. Agent — Side by Side"（裸提示 vs. Agent 并排对比，当前第 20 页）
- 保留表格原样，天气示例已经用得很好。

**第 10 页** — "Agents Get Their Own Brain"（Agent 有自己的大脑，当前第 21 页）
- 保留 Thariq 的提示，并关联到："the weather reporter works in their own brain — all that web fetching stays out of yours."（天气播报员在自己的大脑里工作——那些网络抓取都留在他的脑子里，不进你的。）

**第 11 页** — "How to Create Your Own Agent"（如何创建你自己的 Agent，当前第 22 页）
- 保留 `/agents` 操作指南模式
- 更新代码块，展示真实的 `weather-agent.md` 路径

**第 12 页** — "Agent Config Fields"（Agent 配置字段，当前第 23 页）
- 保留字段行表格，新增一个标注框，展示 `skills: [weather-fetcher]` 字段的上下文。

---

### 第 2 节：Skills（第 13-18 页）——"播报员知道什么"

**第 13 页** — 分节页（`data-level="skills"`，Topic 2）
- 标题："Skills — What the Weather Reporter Knows"
- 描述："Skills are the specific things the reporter has been trained to do. Our reporter has two: fetch the data, and render it as a card."（skill 是播报员受过训练的具体本领。我们的播报员有两项：取数据，把它渲染成卡片。）

**第 14 页** — "The Training Manual"（训练手册，当前第 25 页）
- 重新组织：天气播报员有两个 skill：weather-fetcher（去拿温度）和 weather-svg-creator（生成可视化卡片）。
- 把 "Shayan" 示例替换为天气播报员的这两个 skill。

**第 15 页** — "When to Turn Something Into a Skill"（什么时候该做成 Skill，当前第 26 页）
- 保留 Boris 的提示，把 weather-fetcher 和 weather-svg-creator 加为示例中的两项。

**第 16 页** — "Why Separate Agents and Skills?"（为什么要区分 Agent 和 Skill，当前第 27 页）
- 保留双栏，更新强调点：weather-agent = 这个人，weather-fetcher = 他受过的训练。

**第 17 页** — "How to Create Your Own Skill"（如何创建你自己的 Skill，当前第 28 页）
- 保留。代码块已经是真实的 `weather-fetcher` SKILL.md 内容——正好。

**第 18 页** — "Skill Config Fields"（Skill 配置字段，当前第 29 页）
- 保留。补充说明：weather-fetcher 上设置了 `user-invocable: false`，因为它是 agent 专属。

---

### 第 3 节：Context（第 19-23 页）——"播报员的大脑"

**第 19 页** — 分节页（`data-level="context"`，Topic 3）
- 标题："Context — The Reporter's Brain"
- 描述："Now that you've met the reporter and know their skills, let's understand what they can actually hold in mind at once."（已经认识了播报员和他的技能，现在来理解他一次能装进脑子里的东西有多少。）

**第 20 页** — "Claude's Brain"（Claude 的大脑，当前第 8 页）
- 保留。加一句关联天气播报员："When the weather-agent is dispatched, it gets its own fresh brain — and weather-fetcher is pinned into it at startup."（weather-agent 被派出时会得到一个全新的大脑——weather-fetcher 在启动时就固定在其中。）
- 两张图都保留（context-window.jpeg 留在这里）。

**第 21 页** — "What Loads at Session Start"（会话启动时加载什么，当前第 9 页）
- 保留。关联天气播报员："At startup, Claude knows *about* weather-fetcher (description only). When the command runs, the full skill content is loaded into the agent's brain."（启动时，Claude 只*知道* weather-fetcher 的存在（仅描述）。命令运行时，完整 skill 内容才装进 agent 的大脑。）
- context.jpg 留在这里。

**第 22 页** — "Keep the Brain Clear"（保持大脑清爽，当前第 10 页）
- 保留分支点表格。

**第 23 页** — "How to Manage Your Context"（如何管理你的上下文，当前第 11 页）
- 保留 `/context`、`/compact`、`/clear` 操作指南。

---

### 第 4 节：CLAUDE.md（第 24-29 页）——"随身规则手册"

**第 24 页** — 分节页（`data-level="claude-md"`，Topic 4）
- 标题："CLAUDE.md — The Reporter's Pocket Rulebook"
- 描述："The reporter consults this at the start of every shift — even though their brain resets overnight."（播报员每次开播前都要翻一翻这本手册——尽管他的大脑每晚都会重置。）

**第 25 页** — "The Employee Handbook"（员工手册，当前第 13 页）
- 保留。改为天气播报员表述：CLAUDE.md 是播报员出镜前读的规则手册——"always report in Celsius unless asked, always cite the source"（除非被要求，一律用摄氏度播报；永远注明数据来源）。

**第 26 页** — "How to Create Your CLAUDE.md"（如何创建你的 CLAUDE.md，当前第 14 页）
- 保留 `/init` 操作指南。

**第 27 页** — "Grow CLAUDE.md With Every Mistake"（每个错误都让 CLAUDE.md 成长，当前第 15 页）
- 保留 Boris 的提示。

**第 28 页** — "What Goes in CLAUDE.md"（CLAUDE.md 里放什么，当前第 16 页）
- 保留代码块。加入天气播报员的触点：一条展示天气专属规则的注释。

**第 29 页** — "How CLAUDE.md Loads"（CLAUDE.md 如何加载，当前第 17 页）
- 保留。

---

### 第 5 节：Commands + Workflow（第 30-36 页）——"扳机"

**第 30 页** — 分节页（`data-level="commands"`，Topic 5）
- 标题："Commands — The Trigger"
- 描述："One word kicks off the whole chain. `/weather-orchestrator` → agent → skill → SVG card."（一个词启动整条链路：`/weather-orchestrator` → agent → skill → SVG 卡片。）

**第 31 页** — "Commands — The Entry Point"（Commands——入口，当前第 31 页）
- 保留。介绍得当，已经引用了 weather-orchestrator。

**第 32 页** — "How to Create Your Own Command"（如何创建你自己的 Command，当前第 32 页）
- 保留。代码块已经是 weather-orchestrator.md。

**第 33 页** — Workflow 子节（原第 33 页，`data-level="workflow"`）
- 节号文本从 "Topic 6" 改为 "Putting It All Together"（串起所有部分）
- 保留 `data-level="workflow"`，让进度条填到 100%。
- 标题更新为："Workflow — All Five Pieces Together"
- 描述："Watch the weather reporter example run from one keystroke to SVG card output."（看天气播报员示例从一次按键一路跑到 SVG 卡片输出。）

**第 34 页** — "Command → Agent → Skill"（当前第 34 页）
- 保留代码块流程图，已经非常合适。

**第 35 页** — "Two Ways Skills Are Used"（Skill 的两种用法，当前第 35 页）
- 保留对比预加载 vs 直接调用的双栏。

**第 36 页** — "How to Wire Your Own Workflow"（如何接好你自己的 Workflow，当前第 36 页）
- 保留。已经在用天气工作流作示例。

**第 37 页** — 收尾（当前第 37 页）
- 保留。副标题更新为："Five concepts, one running example"（五个概念，一个贯穿案例）
- 正文更新为呼应天气播报员的叙事线。

---

## 4. 素材复用清单

| 素材 | 当前位置 | 新位置 | 动作 |
|---|---|---|---|
| `context-window.jpeg` | 第 8 页（Claude 的大脑） | 第 20 页（内容相同，仅重编号） | 保留——无需改动 |
| `context.jpg` | 第 9 页（会话启动时加载什么） | 第 21 页（内容相同，仅重编号） | 保留——无需改动 |
| `../../!/claude-jumping.svg` | 第 1 页、页头 | 不变 | 无需处理 |
| `../../!/root/boris-slider.gif` | 第 2 页 | 不变 | 无需处理 |

两张上下文示意图原样保留在原处——包含它们的页面只是重新编号（8→20、9→21）。

---

## 5. 记录事项的影响

### 新的分节页位置与 `data-level` 指派

| 页码 | 主题 | `data-level` |
|---|---|---|
| 7 | Agents | `agents` |
| 13 | Skills | `skills` |
| 19 | Context | `context` |
| 25 | CLAUDE.md | `claude-md` |
| 30 | Commands | `commands` |
| 33 | Workflow 子节 | `workflow` |

### 第 6 页 TOC 的 `goToSlide` 目标

| 行 | 主题 | 旧目标 | 新目标 |
|---|---|---|---|
| 行 1 | Agents | 18 | 7 |
| 行 2 | Skills | 24 | 13 |
| 行 3 | Context | 7 | 19 |
| 行 4 | CLAUDE.md | 12 | 25 |
| 行 5 | Commands | 30 | 30 |

### journey 刻度（不变）

journey 刻度栏从上到下已经是：Workflow、Commands、Skills、Agents、CLAUDE.md、Context。这是叙事顺序的*倒序*（最上 = 最高层级 = 最后达成）。无需改动。

### LEVELS 映射（不变）

全部 6 个层级键（`context`、`claude-md`、`agents`、`skills`、`commands`、`workflow`）保留，不增不减。

---

## 6. 实现方式

HTML 是一个大文件。幻灯片目前的顺序与新叙事线不符。最干净的做法是：

1. 剪切各页 div 并按新顺序粘贴（7-12 = 旧 18-23，13-18 = 旧 24-29，19-23 = 旧 7-11，24-29 = 旧 12-17，30-37 不变）。
2. 按顺序重新编号所有 `data-slide` 属性。
3. 更新各分节页的 `data-level` 属性。
4. 更新分节页的节号文本与 h1。
5. 更新第 6 页 TOC 的 `goToSlide` 目标。
6. 更新 Workflow 分节页（旧第 33 页）的节号文本。
7. 按上述要求做针对性的内容修改，换成天气播报员表述。

总页数：**37**（不变）。

---

## 7. 歧义点——无关键阻塞

所有歧义都已在上方解决，直接进入实现。
