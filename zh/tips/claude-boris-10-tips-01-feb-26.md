# 使用 Claude Code 的 10 条提示——来自 Claude Code 团队（2026-02-01）

> 译自 [tips/claude-boris-10-tips-01-feb-26.md](../../tips/claude-boris-10-tips-01-feb-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 创造者 Boris Cherny（[@bcherny](https://x.com/bcherny)）于 2026 年 2 月 1 日分享的团队技巧汇总。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 背景

Boris 分享了直接来自 Claude Code 团队的使用技巧。团队使用 Claude 的方式与 Boris 个人的用法并不相同。记住：使用 Claude Code 没有唯一正确的方式——每个人的配置都不一样。你应当自己实验，看看什么对你有效。

<a href="https://x.com/bcherny/status/2017742741636321619"><img src="../../tips/assets/boris-26-2-1/0.png" alt="Boris Cherny 开场推文" width="50%" /></a>

---

## 1/ 做更多并行

同时开出 3–5 个 git worktree，每个并行运行自己的 Claude 会话。这是公认的最大一项生产力解锁，也是团队的头号技巧。Boris 个人使用多个 git checkout，但 Claude Code 团队多数人更偏好 worktree——这正是 `@amorisscode` 在 Claude Desktop 应用里为 worktree 做出原生支持的原因。

还有人给 worktree 命名并设置 shell 别名（`2a`、`2b`、`2c`），一键就能在它们之间跳转。另有人专门留一个只用来读日志和跑 BigQuery 的"分析"worktree。

参见：[Worktrees 文档](https://code.claude.com/docs/en/common...)

<a href="https://x.com/bcherny/status/2017742743125299476"><img src="../../tips/assets/boris-26-2-1/1.png" alt="做更多并行" width="50%" /></a>

---

## 2/ 每个复杂任务都从 plan mode 开始

把精力倾注到计划上，让 Claude 一次做对实现。

有一个人让一个 Claude 写计划，再开第二个 Claude 以资深工程师（staff engineer）的身份评审它。

另一位说，一旦事情跑偏，就切回 plan mode 重新规划，不要硬推下去。他们还会明确告诉 Claude 在验证步骤也进入 plan mode，而不只是构建的时候。

<a href="https://x.com/bcherny/status/2017742745365057733"><img src="../../tips/assets/boris-26-2-1/2.png" alt="每个复杂任务都从 plan mode 开始" width="50%" /></a>

---

## 3/ 投入维护你的 CLAUDE.md

每次纠错之后，以这句话收尾："更新你的 CLAUDE.md，别再犯这个错。"Claude 为自己写规则的能力好得出奇。

随着时间推移，毫不留情地编辑你的 `CLAUDE.md`。持续迭代，直到 Claude 的错误率出现可度量的下降。

有位工程师让 Claude 为每个任务/项目维护一个笔记目录，每个 PR 之后更新，然后让 `CLAUDE.md` 指向它。

<a href="https://x.com/bcherny/status/2017742747067945390"><img src="../../tips/assets/boris-26-2-1/3.png" alt="投入维护你的 CLAUDE.md" width="50%" /></a>

---

## 4/ 创建你自己的 Skill 并提交进 Git

在每个项目之间复用。来自团队的技巧：

- 一天要做一次以上的事，就把它做成 skill 或 command
- 建一个 `/techdebt` slash command，每个会话结束时运行，找出并消灭重复代码
- 建一个 slash command，把 7 天的 Slack、GDrive、Asana 和 GitHub 同步成一份上下文汇总
- 构建分析工程师式的 agent：编写 dbt model、评审代码、在 dev 环境测试改动

参见：[Extend Claude with Skills —— Claude Code 文档](https://code.claude.com/docs/en/skills)

<a href="https://x.com/bcherny/status/2017742748984742078"><img src="../../tips/assets/boris-26-2-1/4.png" alt="创建你自己的 Skill" width="50%" /></a>

---

## 5/ Claude 自己就能修掉大部分 Bug

团队的做法是这样的：

启用 Slack MCP，把一个 Slack bug 讨论串粘给 Claude，只说一句"修掉"。完全不需要切换上下文。

或者，直接说"去把失败的 CI 测试修好"。不要微管理它怎么做。

把 Claude 指向 docker logs 来排查分布式系统——它在这方面的能力强得出奇。

<a href="https://x.com/bcherny/status/2017742750473720121"><img src="../../tips/assets/boris-26-2-1/5.png" alt="Claude 自己修掉大部分 Bug" width="50%" /></a>

---

## 6/ 提升你的提示词水平

a. **挑战 Claude。**对它说"就这些改动狠狠拷问我，我没通过你的测试就不许建 PR"。让 Claude 当你的评审者。或者说"向我证明这能用"，让 Claude 对比 main 分支与你的功能分支之间的行为差异。

b. **在一次平庸的修复之后**，说："结合你现在知道的一切，推翻它，实现那个优雅的方案。"

c. **写详细的规格说明**，在交接工作之前减少歧义。你说得越具体，产出就越好。

<a href="https://x.com/bcherny/status/2017742752566632544"><img src="../../tips/assets/boris-26-2-1/6.png" alt="提升你的提示词水平" width="50%" /></a>

---

## 7/ 终端与环境配置

团队钟爱 Ghostty。多人喜欢它的同步渲染、24-bit 颜色和完善的 Unicode 支持。

为了更轻松地驾驭多个 Claude，用 `/statusline` 定制状态栏，让它始终显示上下文用量和当前 git 分支。很多人还会给终端标签页配色、命名，有时用 tmux——每个标签页对应一个任务/worktree。

用语音听写。你的语速是打字速度的 3 倍，提示词也会因此详细得多。（macOS 上连按两次 fn）

参见：[终端配置文档](https://code.claude.com/docs/en/termin...)

<a href="https://x.com/bcherny/status/2017742753971769626"><img src="../../tips/assets/boris-26-2-1/7.png" alt="终端与环境配置" width="50%" /></a>

---

## 8/ 使用 subagent

a. 在任何希望 Claude 为问题投入更多算力的请求后面加上"使用 subagent"。

b. 把独立任务交给 subagent 处理，保持主 Agent 的上下文窗口干净、专注。

c. 通过 hook 把权限请求转给 Opus 4.5——让它扫描攻击并自动批准安全项。参见：[Hooks 文档](https://code.claude.com/docs/en/hooks#...)

<a href="https://x.com/bcherny/status/2017742755737555434"><img src="../../tips/assets/boris-26-2-1/8.png" alt="使用 subagent" width="50%" /></a>

---

## 9/ 用 Claude 做数据与分析

让 Claude Code 用 "bq" CLI 即时拉取和分析指标。团队在代码库里提交了一个 BigQuery skill，所有人都直接在 Claude Code 里用它做分析查询。Boris 个人已经 6 个多月没手写过一行 SQL。

任何有 CLI、MCP 或 API 的数据库都适用这一招。

<a href="https://x.com/bcherny/status/2017742757666902374"><img src="../../tips/assets/boris-26-2-1/9.png" alt="用 Claude 做数据与分析" width="50%" /></a>

---

## 10/ 用 Claude 学习

团队用 Claude Code 学习的几个技巧：

a. 在 `/config` 中启用 "Explanatory"（解释型）或 "Learning"（学习型）输出风格，让 Claude 解释每次改动背后的"为什么"。

b. 让 Claude 生成一份讲解陌生代码的可视化 HTML 演示文稿。它做的幻灯片出奇地好。

c. 让 Claude 为新协议和新代码库画 ASCII 图，帮助你理解。

d. 构建一个间隔重复（spaced-repetition）学习 skill：你讲解自己的理解，Claude 追问补齐缺口，并存储结果。

<a href="https://x.com/bcherny/status/2017742759218794768"><img src="../../tips/assets/boris-26-2-1/10.png" alt="用 Claude 学习" width="50%" /></a>

---

## 来源

- [Boris Cherny (@bcherny) on X —— 2026 年 2 月 1 日](https://x.com/bcherny/status/2017742741636321619)
