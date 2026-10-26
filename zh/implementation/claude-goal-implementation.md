# Goal 实现指南

> 译自 [implementation/claude-goal-implementation.md](../../implementation/claude-goal-implementation.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

![Last Updated](https://img.shields.io/badge/Last_Updated-May_13%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#goal-tips-from-the-community"><img src="../../!/tags/implemented-hd.svg" alt="Implemented"></a>

`/goal` 让你的 agent 跨轮次持续工作，直到满足指定条件 — Claude Code、Codex 和 Hermes Agent 都支持它。社区正在收敛出几个与之搭配的高杠杆提示词技巧。

---

## 来自社区的 Goal 技巧

### 1. 让 agent 自己提出 goal

<p align="center">
  <img src="../../implementation/assets/impl-goal-claude.png" alt="Alex Finn tweet — /goal is the most underrated AI feature of 2026" width="50%">
</p>

> 官宣了。Claude Code 刚刚发布了 /goal
>
> 2026 年最被低估的 AI 功能
>
> 现在 Claude Code、Codex 和 Hermes agent 都支持它
>
> 它能让你的 agent 完成长时间运行的任务，有时一跑就是好几天
>
> 每个人都应该立刻用起这条提示词：
>
> 'Based on what you know about me, my goals, ambitions, and what we've built together already, what are the 3 /goals we can run right now that would run for long time periods and produce the best results?'
>
> 选一个，然后让它给你写一条提示词
>
> 你会得到几个非常强大的 goal 提示词，让你选定的 agent 去完成长时间任务，交付令人惊艳的结果。
>
> 今晚抽出 15 分钟做这件事，以后你会感谢我。

**来源：**[Alex Finn (@AlexFinn) 在 X 上](https://x.com/AlexFinn/status/2053976411296452887)

---

### 2. 让 agent 替你起草 /goal 提示词

<p align="center">
  <img src="../../implementation/assets/impl-goal-codex.png" alt="Meta Alchemist tweet — /goal trick for Codex" width="50%">
</p>

> 想知道 Codex 上最好用的 /goal 技巧吗？
>
> 只要告诉你的 Codex：
>
> "read this session and repo, analyze deeply the exact intent and goals we are looking to achieve here then write me the /goal prompt for this.
>
> make sure to dig into history & docs we have to be 100% clear"
>
> 还可以补充：
>
> "if you are not sure about certain parts or wanna ask me a few questions to clarify certain goals further don't hesitate"
>
> 然后把 Codex 给你的内容原样复制粘贴，把开头改成 /goal
>
> 它就会在那个会话／仓库里做你真正想做的事，不停歇直到完成。

**来源：**[Meta Alchemist (@meta_alchemist) 在 X 上](https://x.com/meta_alchemist/status/2054214497443995694)

---

## ![How to Use](../../!/tags/how-to-use.svg)

```bash
$ claude
> /goal <condition>
> /goal clear
```

`/goal <condition>` 让 Claude 跨轮次持续工作，直到由 Haiku 判定的条件成立。它与 `/loop`（按时间驱动）和 auto 模式（按工具）互为补充。需要 Claude Code v2.1.139+。

完整行为说明见[官方文档](https://code.claude.com/docs/en/goal)。
