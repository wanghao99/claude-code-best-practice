# Code Review 与测试时计算——来自 Boris Cherny 的技巧

> 译自 [tips/claude-boris-2-tips-10-mar-26.md](../../tips/claude-boris-2-tips-10-mar-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 创造者 Boris Cherny（[@bcherny](https://x.com/bcherny)）于 2026 年 3 月 10 日分享的洞见整理。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1/ Code Review 登场

Claude Code 新功能：**Code Review（代码审查）**。一个 Agent 团队会对每个 PR 做深度审查。

- 优先为 Anthropic 自己的团队打造——今年每位工程师的代码产出提升了 **200%**，而审查成了瓶颈
- Boris 用了几周，发现它能抓住许多他原本注意不到的真实 bug
- PR 打开时，Claude 会派出一队 Agent 去搜寻 bug

<a href="https://x.com/bcherny/status/2031089411820228645"><img src="../../tips/assets/boris-26-3-10/0.png" alt="Boris Cherny 发布 Code Review" width="50%" /></a>

---

## 2/ 测试时计算与多个上下文窗口

粗略地说，往一个编码问题上投入的 token 越多，结果就越好。Boris 把这称为**测试时计算（test time compute）**。

- 使用**彼此独立的上下文窗口**会让结果更好——这正是 subagent（子代理）能奏效的原因，也是一个 Agent 可能引入 bug、而另一个 Agent（用完全相同的模型）却能发现它们的原因
- 类比工程团队：如果 Boris 引入了 bug，同事审查这段代码时可能比他自己更可靠地发现它
- 极限情况下，Agent 也许终将写出完美无 bug 的代码——在那之前，**多个互不相关的上下文窗口**往往是好办法

<a href="https://x.com/bcherny/status/2031151689219321886"><img src="../../tips/assets/boris-26-3-10/1.png" alt="Boris Cherny 谈测试时计算" width="50%" /></a>

---

## 来源

- [Boris Cherny (@bcherny) on X —— 2026 年 3 月 10 日](https://x.com/bcherny)
