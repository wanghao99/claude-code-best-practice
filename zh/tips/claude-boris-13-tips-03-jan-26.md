# 我如何使用 Claude Code——Boris 的 13 条提示（2026-01-03）

> 译自 [tips/claude-boris-13-tips-03-jan-26.md](../../tips/claude-boris-13-tips-03-jan-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 创造者 Boris Cherny（[@bcherny](https://x.com/bcherny)）于 2026 年 1 月 3 日分享的个人配置技巧汇总。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 背景

Boris 分享了他个人的 Claude Code 配置，并提到它"出奇地原味（vanilla）"——Claude Code 开箱即用就很好，所以他没做多少定制。使用方式没有唯一正确答案：团队刻意把它做成你想怎么用、怎么定制、怎么魔改都行。Claude Code 团队里每个人的用法都大不相同。

<a href="https://x.com/bcherny/status/2007179832300581177"><img src="../../tips/assets/boris-26-1-3/0.png" alt="Boris Cherny 开场推文" width="50%" /></a>

---

## 1/ 并行运行 5 个 Claude

在终端里并行运行 5 个 Claude。给标签页编号 1–5，再借助系统通知得知哪个 Claude 需要输入。

参见：[终端配置文档](https://code.claude.com/docs/en/terminal)

<a href="https://x.com/bcherny/status/2007179833990885678"><img src="../../tips/assets/boris-26-1-3/1.png" alt="并行运行 5 个 Claude" width="50%" /></a>

---

## 2/ 用 claude.ai/code 获得更高并行度

在 claude.ai/code 上并行运行 5–10 个 Claude，与本地 Claude 并行。用 `claude.ai/code` 把本地会话交接给网页会话，在 Chrome 里手动发起会话，并在两端之间来回"传送"。

<a href="https://x.com/bcherny/status/2007179836704600237"><img src="../../tips/assets/boris-26-1-3/2.png" alt="claude.ai/code 并行" width="50%" /></a>

---

## 3/ 事事都用开启思考的 Opus

什么事都用带思考（thinking）的 Opus 4.5。这是 Boris 用过的最好的编程模型——虽然它比 Sonnet 更大更慢，但由于你需要的引导更少、它的工具使用更强，最终几乎总是比用小模型更快。

<a href="https://x.com/bcherny/status/2007179838864666847"><img src="../../tips/assets/boris-26-1-3/3.png" alt="带思考的 Opus" width="50%" /></a>

---

## 4/ 与团队共享同一份 CLAUDE.md

为仓库共享一份 `CLAUDE.md`。把它提交进 git，让全团队每周多次共同维护。任何时候 Claude 做错了事，就补进 `CLAUDE.md`，让 Claude 知道下次别再这样。

<a href="https://x.com/bcherny/status/2007179840848597422"><img src="../../tips/assets/boris-26-1-3/4.png" alt="共享的 CLAUDE.md" width="50%" /></a>

---

## 5/ 在 PR 上 @claude 来更新 CLAUDE.md

代码评审时，在同事的 PR 上标记 `@claude`，让它把要补充的内容加进 `CLAUDE.md`，作为 PR 的一部分。为此使用 Claude Code 的 GitHub action（[install-@hub-action](https://github.com/apps/claude)）——这是 Boris 版本的"复利式工程（Compounding Engineering）"。

<a href="https://x.com/bcherny/status/2007179842928947333"><img src="../../tips/assets/boris-26-1-3/5.png" alt="在 PR 上标记 @claude" width="50%" /></a>

---

## 6/ 大多数会话从 plan mode 开始

大多数会话从 plan mode（计划模式）开始（shift+tab 按两次）。如果目标是提交一个 Pull Request，就用 plan mode 与 Claude 来回打磨，直到你满意它的计划。然后切入自动接受编辑模式，Claude 通常能一次做完。一个好计划真的非常重要。

<a href="https://x.com/bcherny/status/2007179845336527000"><img src="../../tips/assets/boris-26-1-3/6.png" alt="plan mode" width="50%" /></a>

---

## 7/ 用 slash command 承载内循环工作流

每天要重复多次的每个"内循环（inner loop）"工作流都做成 slash command（斜杠命令）。这能省去反复输入提示词，也让 Claude 自己能用这些工作流。Command 提交进 git，存放在 `.claude/commands/`。

示例：`/commit-push-pr` —— 提交、推送并开一个 PR。

<a href="https://x.com/bcherny/status/2007179847949500714"><img src="../../tips/assets/boris-26-1-3/7.png" alt="slash command" width="50%" /></a>

---

## 8/ 用 subagent 自动化常见工作流

经常使用几个 subagent：`code-simplifier` 在 Claude 做完工作后简化代码，`verify-app` 带有对 Claude Code 做端到端测试的详细说明，等等。把 subagent 看作把最常见工作流自动化——与 slash command 类似。

Subagent 存放在 `.claude/agents/`。

<a href="https://x.com/bcherny/status/2007179850139000872"><img src="../../tips/assets/boris-26-1-3/8.png" alt="subagent" width="50%" /></a>

---

## 9/ 用 PostToolUse hook 自动格式化代码

用 `PostToolUse` hook 格式化 Claude 的代码。Claude 通常开箱就能生成格式良好的代码，hook 负责搞定最后那 10%，避免之后在 CI 里出现格式错误。

```json
"PostToolUse": [
  {
    "matcher": "Write|Edit",
    "hooks": [
      {
        "type": "command",
        "command": "bun run format || true"
      }
    ]
  }
]
```

<a href="https://x.com/bcherny/status/2007179852047335529"><img src="../../tips/assets/boris-26-1-3/9.png" alt="用于格式化的 PostToolUse hook" width="50%" /></a>

---

## 10/ 预先放行权限，而不是 --dangerously-skip-permissions

不要用 `--dangerously-skip-permissions`。改用 `/permissions` 预先放行那些你确认在自己环境中安全的常见 bash 命令，避免不必要的权限提示。这些配置大部分提交进 `.claude/settings.json` 并与团队共享。

<a href="https://x.com/bcherny/status/2007179854077407667"><img src="../../tips/assets/boris-26-1-3/10.png" alt="预先放行权限" width="50%" /></a>

---

## 11/ 通过 MCP 让 Claude 使用你的所有工具

Claude Code 会用上你的所有工具。它经常在 Slack 里搜索和发帖（通过 MCP server），运行 BigQuery 查询来回答数据分析问题（使用 `bq` CLI），从 Sentry 抓取错误日志等。Slack 的 MCP 配置提交进 `.mcp.json` 并与团队共享。

<a href="https://x.com/bcherny/status/2007179856266789204"><img src="../../tips/assets/boris-26-1-3/11.png" alt="MCP 工具" width="50%" /></a>

---

## 12/ 用后台 agent 验证长时间运行的任务

对非常长时间运行的任务，可以 (a) 提示 Claude 在完成时用一个后台 agent 验证自己的工作，(b) 用 agent 的 Stop hook 更确定性地做到这一点，或 (c) 使用 ralph-wiggum 插件（最初由 @GeoffreyHuntley 构思）。

<a href="https://x.com/bcherny/status/2007179858435281082"><img src="../../tips/assets/boris-26-1-3/12.png" alt="长时间任务的验证" width="50%" /></a>

---

## 13/ 给 Claude 一种验证自己工作的方式

这大概是从 Claude Code 获得出色结果最重要的一点——给 Claude 一种验证自己工作的方式。只要 Claude 有了这条反馈回路，最终结果的质量就能翻 2–3 倍。

Claude 会对 Boris 落地的每一个改动做测试。

<a href="https://x.com/bcherny/status/2007179861115511237"><img src="../../tips/assets/boris-26-1-3/13.png" alt="给 Claude 验证的方式" width="50%" /></a>

---

## 来源

- [Boris Cherny (@bcherny) on X —— 2026 年 1 月 3 日](https://x.com/bcherny/status/2007179832300581177)
