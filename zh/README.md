# 中文翻译总索引（按顺序读）

> 译文本目录基于 2026-10-26 的 main 分支（commit 29f6dd2），除机器生成内容外全部翻译完成，共 53 篇。
> 想跟进原文更新：仓库根目录执行 `git pull`；已翻译篇章若原文有实质改动，需按每篇译文头部标注的 commit 重新核对。

**用法：从第 1 篇开始，按编号读下一篇即可。** 编号即推荐阅读顺序，从零基础到进阶排列。每篇开头都有"译自"链接可跳回英文原文对照。

## 阶段 0：装好并跑通（tutorial，5 篇）

| 编号 | 篇目 | 说明 |
|---|---|---|
| 1 | [Day 0 — Claude Code 安装](tutorial/day0/README.md) | 安装总览：装 Node.js，装 Claude Code，登录 |
| 2 | [Day 0 — macOS 安装](tutorial/day0/mac.md) | macOS 详细步骤（fnm 装 Node、npm 全局安装、浏览器登录） |
| 3 | [Day 0 — Windows 安装](tutorial/day0/windows.md) | 仅 Windows 用户需要 |
| 4 | [Day 0 — Linux 安装](tutorial/day0/linux.md) | 仅 Linux 用户需要 |
| 5 | [Day 1 — 与 Claude Code 的第一次对话](tutorial/day1/README.md) | 第一课：用"问路人、雇专员、培训手册"三层比喻理解 Prompt / Agent / Skill |

## 阶段 1：三原语入门（4 篇）

先读三篇短参考，知道 agent / command / skill 的文件长什么样、放在哪里；再读对比篇，知道什么时候用哪个。

| 编号 | 篇目 | 说明 |
|---|---|---|
| 6 | [Subagents 最佳实践](best-practice/claude-subagents.md) | subagent（子代理）：frontmatter 字段、写法、调用方式 |
| 7 | [Commands 最佳实践](best-practice/claude-commands.md) | slash 命令 + 94 个内置命令速查 |
| 8 | [Skills 最佳实践](best-practice/claude-skills.md) | Skill 参考手册 + 内置 skill 列表 |
| 9 | [Agents vs Commands vs Skills — 什么时候用哪个](reports/claude-agent-command-skill.md) | 三者机制对比表，同一个任务三种实现的实例演算 |

## 阶段 2：核心机制（5 篇）

| 编号 | 篇目 | 说明 |
|---|---|---|
| 10 | [Claude 记忆系统](best-practice/claude-memory.md) | CLAUDE.md 分层记忆、rules、自动压缩 |
| 11 | [MCP Server 最佳实践](best-practice/claude-mcp.md) | MCP 是什么、怎么配、常见 server |
| 12 | [Settings（settings.json 配置）最佳实践](best-practice/claude-settings.md) | 设置参考手册，1400+ 行，当字典查，不必通读 |
| 13 | [Power-ups 最佳实践](best-practice/claude-power-ups.md) | 进阶增强能力概览 |
| 14 | [CLI 启动标志最佳实践](best-practice/claude-cli-startup-flags.md) | 命令行启动参数参考 |

## 阶段 3：编排与实战（10 篇）

| 编号 | 篇目 | 说明 |
|---|---|---|
| 15 | [编排工作流（Orchestration Workflow）](orchestration-workflow/orchestration-workflow.md) | `/weather-orchestrator` 完整示例：一个 Command 调度多个 Agent 和 Skill 协作 |
| 16 | [Subagents 实现指南](implementation/claude-subagents-implementation.md) | 仓库自带 subagent 的落地实现 |
| 17 | [Commands 实现指南](implementation/claude-commands-implementation.md) | 仓库自带 command 的落地实现 |
| 18 | [Skills 实现指南](implementation/claude-skills-implementation.md) | 仓库自带 skill 的落地实现 |
| 19 | [Goal 实现指南](implementation/claude-goal-implementation.md) | goal（目标）机制的落地实现 |
| 20 | [Agent Teams 实现指南](implementation/claude-agent-teams-implementation.md) | 多 agent 团队协作机制的落地实现 |
| 21 | [Scheduled Tasks 实现指南](implementation/claude-scheduled-tasks-implementation.md) | 定时任务机制的落地实现 |
| 22 | [Agent Teams 主提示词](agent-teams/agent-teams-prompt.md) | agent teams 功能的完整提示词，相当于设计文档 |
| 23 | [RPI 工作流](development-workflows/rpi/rpi-workflow.md) | 树莓派 7x24 无人值守自动开发工作流 |
| 24 | [跨模型（Claude Code + Codex）工作流](development-workflows/cross-model-workflow/cross-model-workflow.md) | Claude Code 与 Codex 跨模型协作 |

## 阶段 4：机制深挖（reports，11 篇）

| 编号 | 篇目 | 说明 |
|---|---|---|
| 25 | [为什么是 Harness——Claude Code 设计解析](reports/why-harness-is-important.md) | 为什么模型外围的工具脚手架（harness）决定 agent 上限 |
| 26 | [Agent Memory frontmatter（subagent 持久记忆）](reports/claude-agent-memory.md) | Claude Code 自动记忆机制深挖 |
| 27 | [全局与项目级功能对比](reports/claude-global-vs-project-settings.md) | 全局与项目级设置的区别（含 Tasks 系统） |
| 28 | [Agent SDK 与 CLI 对比](reports/claude-agent-sdk-vs-cli-system-prompts.md) | 两者系统提示词与输出一致性差异 |
| 29 | [Claude 高级 Tool Use 模式](reports/claude-advanced-tool-use.md) | 高级工具使用模式 |
| 30 | [用量、速率限制与 Extra Usage](reports/claude-usage-and-rate-limits.md) | 用量与限额机制 |
| 31 | [Spinner（加载动画）里的单词与小贴士](reports/claude-spinner-verbs-and-tips.md) | 加载动词彩蛋与杂项技巧 |
| 32 | [LLM 日常性能劣化：迷思与现实](reports/llm-day-to-day-degradation.md) | "模型变笨了"的实测记录 |
| 33 | [理解大型 Monorepo 中的 Claude Skills 发现机制](reports/claude-skills-for-larger-mono-repos.md) | 大 monorepo 的 skill 组织 |
| 34 | [浏览器自动化 MCP 全面对比报告](reports/claude-in-chrome-v-chrome-devtools-mcp.md) | Claude in Chrome 与 Chrome DevTools MCP 选型 |
| 35 | [学习之旅——天气播报员 Agent 重设计方案](reports/learning-journey-weather-reporter-redesign.md) | 仓库示例 agent 的迭代重设计全过程 |

## 阶段 5：作者与团队技巧（tips，9 篇，按发布时间）

| 编号 | 篇目 | 说明 |
|---|---|---|
| 36 | [How I Use Claude Code — 13 Tips](tips/claude-boris-13-tips-03-jan-26.md) | 作者"朴素用法"13 条，最经典的入门技巧帖 |
| 37 | [10 Tips — From the Claude Code Team](tips/claude-boris-10-tips-01-feb-26.md) | 团队 10 条技巧 |
| 38 | [12 Ways to Customize Claude Code](tips/claude-boris-12-tips-12-feb-26.md) | 12 种自定义方式 |
| 39 | [Code Review & Test Time Compute](tips/claude-boris-2-tips-10-mar-26.md) | 代码审查与测试时算力 |
| 40 | [How We Use Skills — Thariq](tips/claude-thariq-tips-17-mar-26.md) | 团队成员讲他们自己怎么用 Skills |
| 41 | [Squash Merging & PR Size Distribution](tips/claude-boris-2-tips-25-mar-26.md) | squash 合并与 PR 体积分布 |
| 42 | [15 Hidden & Under-Utilized Features](tips/claude-boris-15-tips-30-mar-26.md) | 15 个隐藏和被低估的功能 |
| 43 | [6 Tips for Getting More Out of Opus 4.7](tips/claude-boris-6-tips-16-apr-26.md) | 针对 Opus 4.7 的 6 个技巧 |
| 44 | [Session Management & 1M Context — Thariq](tips/claude-thariq-tips-16-apr-26.md) | 会话管理与 1M 上下文 |

## 阶段 6：长视频访谈与演讲（videos，8 篇，按发布时间，均为完整逐字稿）

| 编号 | 篇目 | 说明 |
|---|---|---|
| 45 | [造 Claude Code 的工程师亲述它的秘密 — Every](videos/claude-cat-every-29-oct-25.md) | Anthropic 内部 power user 怎么用 Claude Code |
| 46 | [Boris Cherny 谈什么成就了他的职业生涯 — Ryan Peterman](videos/claude-boris-ryan-peterman-15-dec-25.md) | Meta/Instagram 经历、Claude Code 起源、vibe coding 之辩 |
| 47 | [走进 Claude Code：与创造者 Boris Cherny 对话 — Y Combinator](videos/claude-boris-y-combinator-17-feb-26.md) | YC 访谈：产品演化与设计取舍 |
| 48 | [Claude Code 掌门人：编程被解决之后会发生什么 — Lenny's Podcast](videos/claude-boris-lennys-podcast-19-feb-26.md) | 产品哲学：工程师角色会怎么变 |
| 49 | [与 Boris Cherny 一起打造 Claude Code — The Pragmatic Engineer](videos/claude-boris-pragmatic-engineer-04-mar-26.md) | 工程视角的深度长访谈 |
| 50 | [我们在 Research-Plan-Implement 上搞错的一切 — MLOps Community](videos/claude-dex-mlops-community-24-mar-26.md) | Dex 讲"研究-计划-实施"模式的踩坑复盘 |
| 51 | [完整走查：AI 编码的工作流 — Matt Pocock](videos/claude-matt-pocock-24-apr-26.md) | 大会工作坊：聪明区/笨区、垂直切片、Ralph 循环、深模块 |
| 52 | [从 Vibe Coding 到 Agentic Engineering — Andrej Karpathy](videos/claude-karpathy-ai-engineer-02-may-26.md) | Karpathy 谈 agent 工程化 |
| 53 | [视频 1：从 Vibe Coding 到 Agentic Engineering](!/video-presentation-transcript/1-video-workflow.md) | 仓库作者自制 5 分钟演示视频的讲稿，主题与第 6-9 篇呼应，读完第 9 篇即可提前看 |

## 未翻译的范围（不需要看）

- `changelog/`（12 篇）— 机器生成的仓库审计日志
- `.claude/`、`.codex/`、`agent-teams/.claude/`、`development-workflows/rpi/.claude/` — 仓库自身的运行配置（agent、command、skill、hook、rules 定义；这些文件怎么写，读第 6-24 篇的指南即可，正文里被引用的文件译文直接链接到英文原文件）
- `orchestration-workflow/output.md`、`agent-teams/output/output.md` — 命令运行的示例输出
- `presentation/` — 幻灯片素材，无 markdown 正文
- 仓库根 `CLAUDE.md`、`README.md` — 英文 README 的内容已吸收进本索引

## 已知问题

- [第 30 篇](reports/claude-usage-and-rate-limits.md)正文中有一条指向 `claude-cli-startup-flags.md` 的链接，英文原版即指向不存在的位置（该文件实际在 `best-practice/` 目录），译文按原样保留。
