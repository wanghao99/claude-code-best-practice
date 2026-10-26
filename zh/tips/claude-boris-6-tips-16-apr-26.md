# 充分发挥 Opus 4.7 的 6 条技巧——来自 Boris Cherny

> 译自 [tips/claude-boris-6-tips-16-apr-26.md](../../tips/claude-boris-6-tips-16-apr-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 创造者 Boris Cherny（[@bcherny](https://x.com/bcherny)）于 2026 年 4 月 16 日分享的技巧推文串——此前几周他一直在内部试用（dogfooding）Opus 4.7。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 背景

经过几周的内部试用，Boris 感觉"生产力爆棚"，于是分享了六种从新模型中获得更多产出的方法——涵盖权限自动化、effort（投入度）调优，以及验证模式。

<a href="https://x.com/bcherny"><img src="../../tips/assets/boris-26-4-16/0.png" alt="Boris Cherny 开场推文——内部试用 Opus 4.7" width="50%" /></a>

---

## 1/ Auto 模式——告别权限提示

Opus 4.7 擅长复杂、长时间运行的任务：深度研究、重构代码、构建复杂功能、迭代直到达到性能基准。过去，面对这类长任务，你要么守着模型全程盯梢，要么使用 `--dangerously-skip-permissions`。

Anthropic 最近推出了更安全的替代方案——**auto 模式（自动模式）**。在该模式下，权限提示会被交给一个基于模型的分类器，由它判断命令是否可以安全运行：

- 安全则自动批准
- 有风险则暂停并询问

这意味着模型运行时不用再盯梢。更进一步，它还意味着你可以并行运行更多个 Claude——只要安全，你就可以把注意力切到下一个 Claude。

Auto 模式目前已向 Max、Teams 和 Enterprise 用户的 Opus 4.7 开放。在 CLI 中按 **Shift+Tab** 可在 `Ask permissions` → `Plan mode`（计划模式）→ `Auto mode` 之间循环切换，也可以在桌面端或 VS Code 的下拉菜单中选择。

<a href="https://x.com/bcherny"><img src="../../tips/assets/boris-26-4-16/1.png" alt="Boris Cherny 谈 auto 模式" width="50%" /></a>

---

## 2/ 新的 /fewer-permission-prompts Skill

Anthropic 发布了新的 `/fewer-permission-prompts` Skill（技能）。它会扫描你的会话历史，找出那些安全却反复触发权限提示的常见 bash 和 MCP 命令，然后推荐一份可加入权限允许列表（allowlist）的命令清单。

用它来调优你的权限配置、避免不必要的权限提示，尤其是在你不用 auto 模式的情况下。

<a href="https://x.com/bcherny"><img src="../../tips/assets/boris-26-4-16/2.png" alt="Boris Cherny 谈 /fewer-permission-prompts Skill" width="50%" /></a>

---

## 3/ Recaps（回顾摘要）

Anthropic 本周早些时候为迎接 Opus 4.7 上线了 **recaps**。Recap 是一段简短摘要，说明 Agent 做了什么、接下来做什么。

过几分钟或几小时后回到一个长时间运行的会话时非常有用：

```
* Cogitated for 6m 27s

* recap: Fixing the post-submit transcript shift bug. The styling-flash
  part is shipped as PR #29869 (auto-merge on, posted to stamps). Next:
  I need a screen recording of the remaining horizontal rewrap on `cc -c`
  to target that separate cause. (disable recaps in /config)
```

如果不需要，可以在 `/config` 中关闭 recaps。

<a href="https://x.com/bcherny"><img src="../../tips/assets/boris-26-4-16/3.png" alt="Boris Cherny 谈 recaps" width="50%" /></a>

---

## 4/ Focus 模式

Boris 一直在用 CLI 里新的 **focus 模式（专注模式）**，它会隐藏所有中间过程，只聚焦最终结果。如今模型已经达到这样的水平：他基本信任它会运行正确的命令、做出正确的修改，自己只看最终结果。

用 `/focus` 开启或关闭。

<a href="https://x.com/bcherny"><img src="../../tips/assets/boris-26-4-16/4.png" alt="Boris Cherny 谈 focus 模式" width="50%" /></a>

---

## 5/ 配置你的 effort 等级

Opus 4.7 使用**自适应思考（adaptive thinking）**，而非思考预算（thinking budget）。想让模型想得更多或更少，就调节 effort。

- **更低的 effort** —— 响应更快、token 消耗更少
- **更高的 effort** —— 最强的智能与能力

滑块提供五档：`low` · `medium` · `high` · `xhigh` · `max` —— 左端偏速度，右端偏智能。

<a href="https://x.com/bcherny"><img src="../../tips/assets/boris-26-4-16/5.png" alt="Boris Cherny 谈 effort 等级" width="50%" /></a>

---

## 6/ 给 Claude 一种验证自己工作的方式

最后，确保 Claude 有办法验证自己的工作。这一点一直很重要——现在 4.7 能让你从 Claude 获得的产出达到以往的 2-3 倍，所以它比以往任何时候都重要。

验证方式因任务而异：

- **后端工作** —— 让 Claude 运行你的服务器/服务做端到端测试
- **前端工作** —— 使用 [Claude Chromium 扩展](https://code.claude.com/docs/en/chrome)，给 Claude 一种控制浏览器的方式
- **桌面应用** —— 使用 Computer Use

Boris 现在的提示词长这样：`Claude do blah blah /go`，其中 `/go` 是一个 Skill，它会：

1. 用 bash、浏览器或 Computer Use 对自己做端到端测试
2. 运行 `/simplify`
3. 创建 PR

对于长时间运行的工作，验证更加重要——当你回到任务前时，你知道代码是能用的。

<a href="https://x.com/bcherny"><img src="../../tips/assets/boris-26-4-16/6.png" alt="Boris Cherny 谈验证" width="50%" /></a>

---

## 来源

- [Boris Cherny (@bcherny) on X —— 2026 年 4 月 16 日](https://x.com/bcherny)
