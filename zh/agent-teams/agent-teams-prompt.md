# Agent Teams 主提示词

> 译自 [agent-teams/agent-teams-prompt.md](../../agent-teams/agent-teams-prompt.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

创建一个 agent team，构建一个时间编排（orchestration）workflow，
以可视化 SVG 卡片的形式显示迪拜当前时间。该 workflow 遵循
Command → Agent → Skill 架构模式：

- 由一个 command 负责编排流程并处理用户交互
- 由一个 agent 使用预加载的 skill 获取迪拜的实时当前时间
- 由一个 skill 根据获取到的数据创建可视化 SVG 时间卡片

**重要**：所有文件必须创建在 `agent-teams/.claude/` 内，
而不是仓库根目录的 `.claude/` 目录。这样可以让 agent team 的
输出保持自包含，并可通过 `cd agent-teams && claude` 直接运行。
不要引用或复制现有的 weather workflow——所有内容全部从零构建。

分配以下 teammates（队友）：

1. **Command Architect** — 设计并实现 `/time-orchestrator`
   command，文件为 `agent-teams/.claude/commands/time-orchestrator.md`。该 command 应：
   - 通过 Agent 工具（而不是 bash）调用 time-agent，获取
     阿联酋迪拜（Asia/Dubai 时区，UTC+4）的当前时间
   - 通过 Skill 工具调用 time-svg-creator skill，根据
     获取到的时间数据渲染 SVG 卡片
   - 在 frontmatter 中使用 model: haiku
   - 包含关键要求：顺序执行的流程、正确的工具使用
     （agent 用 Agent 工具、skill 用 Skill 工具）以及输出总结
   通过共享任务列表与其他 teammates 协调，就组件之间
   传递的数据契约（{time, timezone, formatted}）达成一致。

2. **Agent Engineer** — 设计并实现 `time-agent`，文件为
   `agent-teams/.claude/agents/time-agent.md`；并实现其预加载的 `time-fetcher`
   skill，文件为 `agent-teams/.claude/skills/time-fetcher/SKILL.md`。该 agent 应：
   - 使用 Bash 获取迪拜当前时间（Asia/Dubai，UTC+4），
     命令为 `TZ='Asia/Dubai' date '+%Y-%m-%d %H:%M:%S %Z'`
   - 向 command 返回时间值、时区名称和格式化字符串
   - frontmatter 使用：tools (Bash)、model: haiku、color: blue、maxTurns: 3
   - 通过 `skills:` 字段预加载 time-fetcher skill
   time-fetcher skill（`agent-teams/.claude/skills/time-fetcher/SKILL.md`）
   应包含获取迪拜时间的 bash 命令和期望的输出格式，
   并设置 user-invocable: false，因为它是仅限 agent 使用的领域知识。
   将商定的数据契约发布到共享任务列表，让 Command
   Architect 和 Skill Designer 能够对齐接口。

3. **Skill Designer** — 设计并实现 `time-svg-creator`
   skill，文件为 `agent-teams/.claude/skills/time-svg-creator/SKILL.md`，并附带辅助
   文件 `reference.md`（SVG 模板 + 输出模板）和 `examples.md`
   （输入/输出示例对）。该 skill 应：
   - 从调用上下文接收时间值、时区和格式化字符串
   - 为迪拜创建一个自包含的 SVG 时间卡片，显示当前时间
   - 将 SVG 写入 `agent-teams/output/dubai-time.svg`
   - 将 markdown 总结写入 `agent-teams/output/output.md`
   - 使用提供的精确时间——绝不重新获取
   - 模板保存在 reference.md（带占位符的 SVG 标记、markdown
     输出模板），示例对保存在 examples.md
   同时创建用于存放输出文件的 `agent-teams/output/` 目录。

三位 teammates 都应在共享任务列表中创建任务，
协调数据契约：agent 返回 {time, timezone, formatted}，
command 通过上下文传递它，skill 消费它。
由于各组件相互独立，三位可以全部并行启动——
它们只需要就数据接口达成一致，不需要等待
彼此的实现。
