# 构建 Claude Code 的经验：我们如何使用 Skill——Thariq（2026-03-17）

> 译自 [tips/claude-thariq-tips-17-mar-26.md](../../tips/claude-thariq-tips-17-mar-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Thariq（[@trq212](https://x.com/trq212)）于 2026 年 3 月 17 日分享的一份综合指南，讲 Anthropic 内部如何使用 Skill。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 背景

Skill 已成为 Claude Code 中使用最广泛的扩展点之一。它们灵活、易做、易于分发。但这种灵活性也让人难以判断怎样用效果最好。Thariq 分享了 Anthropic 大规模使用 Skill 的经验——目前有数百个 Skill 在活跃使用中。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/1.png" alt="Thariq 开场推文" width="50%" /></a>

---

## Skill 是什么？

一个常见的误解是 Skill "只是一些 markdown 文件"，但最有意思的地方在于它们是**文件夹**，可以包含脚本、素材、数据等——Agent 可以去发现、探索并操作这些东西。Skill 还有非常丰富的配置选项，包括注册动态 hook。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/2.png" alt="Skill 是什么？" width="50%" /></a>

---

## Skill 的类型

团队在给所有 Skill 编目之后，发现它们自然聚成了 9 类。最好的 Skill 干净地落在某一类里；让人困惑的那些则横跨好几类。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/3.png" alt="Skill 类型九宫格" width="50%" /></a>

---

### 1/ 库与 API 参考

解释如何正确使用某个库、CLI 或 SDK 的 Skill。可以是内部库，也可以是 Claude Code 偶尔处理不好的公共库。它们通常包含一个参考代码片段文件夹，外加一份写脚本时要避开的坑（gotcha）清单。

**示例**：billing-lib、internal-platform-cli、frontend-design

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/4.png" alt="库与 API 参考" width="50%" /></a>

---

### 2/ 产品验证

描述如何测试或验证你的代码是否正常工作的 Skill。它们常与 Playwright、tmux 等外部工具搭配使用。验证类 Skill 对确保 Claude 的输出正确极其有用。让一位工程师专门花一周，把你们的验证 Skill 打磨到优秀，是值得的。

**示例**：signup-flow-driver、checkout-verifier、tmux-cli-driver

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/5.png" alt="产品验证" width="50%" /></a>

---

### 3/ 数据获取与分析

连接你的数据和监控体系的 Skill。可能包含带凭据取数的库、具体的看板 ID 等，以及常见工作流或取数方式的说明。

**示例**：funnel-query、cohort-compare、grafana

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/6.png" alt="数据获取与分析" width="50%" /></a>

---

### 4/ 业务流程与团队自动化

把重复性工作流自动化成一条命令的 Skill。它们的说明通常相当简单，但可能对其他 Skill 或 MCP 有较复杂的依赖。把以往结果保存到日志文件里，能帮助模型保持一致，并复盘这个工作流的历次执行。

**示例**：standup-post、create-\<ticket-system\>-ticket、weekly-recap

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/7.png" alt="业务流程与团队自动化" width="50%" /></a>

---

### 5/ 代码脚手架与模板

为代码库中某个特定功能生成框架样板代码的 Skill。可以把这些 Skill 与可组合的脚本结合使用。当你的脚手架带有无法纯用代码覆盖的自然语言要求时，它们尤其有用。

**示例**：new-\<framework\>-workflow、new-migration、create-app

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/8.png" alt="代码脚手架与模板" width="50%" /></a>

---

### 6/ 代码质量与评审

在你的组织内部强制执行代码质量并帮助评审代码的 Skill。可以包含确定性脚本或工具，以获得最大健壮性。你可能希望把这些 Skill 作为 hook 的一部分自动运行，或在 GitHub Action 里运行。

**示例**：adversarial-review、code-style、testing-practices

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/9.png" alt="代码质量与评审" width="50%" /></a>

---

### 7/ CI/CD 与部署

帮助你在代码库里拉取、推送和部署代码的 Skill。这些 Skill 可能引用其他 Skill 来收集数据。

**示例**：babysit-pr、deploy-\<service\>、cherry-pick-prod

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/10.png" alt="CI/CD 与部署" width="50%" /></a>

---

### 8/ Runbook（操作手册）

接收一个症状（比如 Slack 讨论串、告警或错误签名），走完一趟多工具排查，并产出结构化报告的 Skill。

**示例**：\<service\>-debugging、oncall-runner、log-correlator

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/11.png" alt="Runbook" width="50%" /></a>

---

### 9/ 基础设施运维

执行例行维护和运维流程的 Skill——其中一些涉及受益于护栏（guardrail）保护的破坏性操作。它们让工程师在关键操作中更容易遵循最佳实践。

**示例**：\<resource\>-orphans、dependency-management、cost-investigation

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/12.png" alt="基础设施运维" width="50%" /></a>

---

## 制作 Skill 的技巧

9 条编写高效 Skill 的最佳实践，外加分发与度量方面的建议。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/13.png" alt="制作 Skill 的技巧九宫格" width="50%" /></a>

---

### 技巧 1：不要陈述显而易见的内容

Claude Code 对你的代码库了解很多，Claude 对编程也了解很多，包括许多默认偏好。如果你发布的是一个以知识为主的 Skill，尽量聚焦那些能把 Claude 推出常规思路的信息。前端设计 Skill 就是个好例子——它是通过与客户反复迭代、改进 Claude 的设计品味而打磨出来的，避开 Inter 字体、紫色渐变这类老套模式。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/14.png" alt="不要陈述显而易见的内容" width="50%" /></a>

---

### 技巧 2：建一个"坑"章节

任何 Skill 中信号密度最高的内容是"坑（Gotchas）"章节。这些章节应该从 Claude 使用你的 Skill 时经常撞上的失败点积累而来。理想情况下，你会随着时间推移不断更新 Skill，把这些坑收录进去。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/15.png" alt="建一个坑章节" width="50%" /></a>

---

### 技巧 3：利用文件系统与渐进式披露

Skill 是一个文件夹，而不只是一个 markdown 文件。你应当把整个文件系统当作一种上下文工程（context engineering）和渐进式披露（progressive disclosure）的手段。告诉 Claude 你的 Skill 里有哪些文件，它会在合适的时机去读。最简单的形式是指向其他 markdown 文件——例如把详细的函数签名和用法示例拆到 `references/api.md`。你可以有分别放参考资料、脚本、示例等的多个文件夹。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/16.png" alt="渐进式披露" width="50%" /></a>

---

### 技巧 4：避免把 Claude 钉死在轨道上

Claude 通常会尽量遵守你的指示，而 Skill 又高度可复用，所以要小心别写得过于具体。给 Claude 需要的信息，但给它因地制宜的灵活度。与其给出一步到位的处方式指示，不如给出目标和约束。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/17.png" alt="避免把 Claude 钉死在轨道上" width="50%" /></a>

---

### 技巧 5：想清楚初始化设置

有些 Skill 可能需要用户提供上下文来完成设置。一个好模式是把设置信息存在 Skill 目录的 `config.json` 文件里。如果配置尚未就绪，agent 可以转而向用户询问。你可以指示 Claude 使用 AskUserQuestion 工具提出结构化的多选题。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/18.png" alt="想清楚初始化设置" width="50%" /></a>

---

### 技巧 6：description 字段是写给模型看的

Claude Code 启动会话时，会构建一份清单，列出每个可用 Skill 及其 description。Claude 扫描这份清单来判断"这个请求有没有对应的 Skill"。也就是说，description 字段不是摘要——它描述的是这个 Skill **何时触发**。它是写给模型看的。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/19.png" alt="description 即触发条件" width="50%" /></a>

---

### 技巧 7：记忆与数据存储

有些 Skill 可以通过在内部存储数据来获得某种形式的记忆。数据可以存在最简单的只追加文本日志或 JSON 文件里，也可以复杂到 SQLite 数据库。存在 Skill 目录里的数据在升级 Skill 时可能被删掉，所以请用 `${CLAUDE_PLUGIN_DATA}` 作为每个插件的稳定存储目录。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/20.png" alt="记忆与数据存储" width="50%" /></a>

---

### 技巧 8：存储脚本并生成代码

你能给 Claude 的最强大工具之一就是代码。给 Claude 脚本和库，让它把轮次花在组合编排上——决定下一步做什么，而不是重新构造样板。Claude 还能现场生成脚本，把这些功能组合起来做更高级的分析。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/21.png" alt="存储脚本并生成代码" width="50%" /></a>

---

### 技巧 9：按需 hook

Skill 可以包含只在 Skill 被调用时才激活、并持续到会话结束的 hook。用它来承载那些你不想一直运行、但有时极其有用的强势（opinionated）hook。

**示例**：
- `/careful` —— 通过针对 Bash 的 PreToolUse matcher 阻止 rm -rf、DROP TABLE、force-push、kubectl delete
- `/freeze` —— 阻止不在特定目录内的任何 Edit/Write

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/22.png" alt="按需 hook" width="50%" /></a>

---

## 分发 Skill

与团队共享 Skill 的两种方式：
- **提交进你的仓库**（放在 `.claude/skills` 下）——适合在数量不多的仓库间协作的小团队
- **做成 plugin**，搭一个 Claude Code plugin 市场，让用户上传和安装插件

每个提交进仓库的 Skill 都会给模型的上下文增加一点负担。规模扩大之后，内部 plugin 市场让你能分发 Skill，并让团队自己决定装哪些。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/23.png" alt="分发 Skill" width="50%" /></a>

---

## 管理一个市场

没有一个中央团队来决定哪些 Skill 可以进入市场。相反，要试着自然地发现最有用的 Skill。把它们上传到 GitHub 的一个沙箱文件夹，在 Slack 或其他论坛里指给大家。一旦某个 Skill 有了人气（由 Skill 所有者自己判断），就可以提 PR 把它移进市场。发布前做筛选很重要，可以避免重复的 Skill。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/24.png" alt="管理一个市场" width="50%" /></a>

---

## 组合 Skill

你可能希望 Skill 之间相互依赖。例如，一个文件上传 Skill 负责上传文件，还有一个 CSV 生成 Skill 先生成 CSV 再调用前者上传。这种依赖管理还没有原生内建在市场或 Skill 里，但你只要按名字引用其他 Skill，模型就会在它们已安装的情况下调用它们。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/25.png" alt="组合 Skill" width="50%" /></a>

---

## 度量 Skill

要了解一个 Skill 用得怎么样，可以用 PreToolUse hook 记录公司内部的 Skill 使用情况。这样就能发现哪些 Skill 很受欢迎，哪些相比预期触发不足。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/26.png" alt="度量 Skill" width="50%" /></a>

---

## 结语

Skill 是 Agent 极其强大、灵活的工具，但现在还早，大家都在摸索最佳用法。与其把这篇当作权威指南，不如当作一堆经过验证、确实有效的技巧杂锦。理解 Skill 最好的方式就是上手、实验、看什么对你有效。我们大多数 Skill 起初都只有几行字和一条坑，正因为 Claude 不断撞上新的边界情况、大家持续补充，它们才越来越好。

<a href="https://x.com/trq212/status/2033949937936085378"><img src="../../tips/assets/thariq-26-3-17/27.png" alt="结语" width="50%" /></a>

---

## 来源

- [Thariq (@trq212) on X —— 2026 年 3 月 17 日](https://x.com/trq212/status/2033949937936085378)
- [Skilljar —— Agent Skills 课程](https://code.claude.com/docs/en/skills)
- [Skill Creator](https://code.claude.com/docs/en/skills)
