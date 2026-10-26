# Claude Code：用量、速率限制与 Extra Usage（额外用量）

> 译自 [reports/claude-usage-and-rate-limits.md](../../reports/claude-usage-and-rate-limits.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

理解 Claude Code 的用量限制如何运作，以及撞到限制之后如何继续工作。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 概述

Claude Code 的订阅方案（Pro、Max 5x、Max 20x）采用滚动窗口重置的用量限制。三个内置斜杠命令帮你监控和管理用量：

| Command | 描述 | 适用对象 |
|---------|-------------|--------------|
| `/usage` | 查看方案限额与速率限制状态 | Pro、Max 5x、Max 20x |
| `/extra-usage` | 撞到限制时配置按量付费的溢出用量 | Pro、Max 5x、Max 20x |
| `/cost` | 显示当前会话的 token 用量与花费 | API key 用户 |

---

## `/usage` — 查看你的限额

显示当前方案的用量限制和速率限制状态。适合在撞到限制之前查看自己还剩多少额度。

---

## `/extra-usage` — 超过限制后继续工作

`/extra-usage` 命令用于配置**按量付费的溢出计费（pay-as-you-go overflow billing）**，这样当你撞到方案的速率限制时，Claude Code 会无缝继续工作，而不是把你直接挡住。

### 工作原理

1. 你撞到了方案的速率限制（限制每 5 小时重置一次）
2. 如果已启用 extra usage 且账户有可用资金，Claude Code 会继续运行，不会中断
3. 溢出的 token 按**标准 API 费率**计费，与订阅费分开结算

### 配置方法

CLI 中的 `/extra-usage` 命令会引导你完成配置。你也可以在 claude.ai 网页端的 **Settings > Usage** 中配置：

1. 启用 extra usage
2. 添加支付方式
3. 设置**月度消费上限**（或选择不设上限）
4. 可选：添加**预付资金**，并在余额低于阈值时自动充值

### 关键细节

| 细节 | 数值 |
|--------|-------|
| 每日划扣上限 | $2,000/天 |
| 计费方式 | 独立于订阅，按标准 API 费率 |
| 限制重置窗口 | 每 5 小时 |

### 已知问题

截至 2026 年 2 月，`/extra-usage` CLI 命令尚[无文档](https://github.com/anthropics/claude-code/issues/12396)，可能只打开一个登录窗口，却没有清晰的配置选项。目前通过 **claude.ai 网页界面**配置是更可靠的路径。

---

## `/cost` — 会话花费（API 用户）

对于使用 API key 认证（而非订阅方案）的用户，`/cost` 显示：

- 当前会话的总花费
- API 时长与墙钟时间
- token 用量明细
- 做出的代码改动

该命令与 Pro/Max 订阅用户无关。

---

## Fast Mode 与 Extra Usage

Fast mode（`/fast`）使用 Claude Opus 4.6，输出更快。它与 extra usage 之间有一层特殊的计费关系：

- Fast mode 的用量**从第一个 token 起就计入 extra usage**
- 即使你的订阅方案还有剩余额度，也是如此
- Fast mode 不消耗方案自带的速率限制

也就是说，要使用 `/fast`，必须先启用 extra usage 并确保有资金。

---

## CLI 启动标志

两个启动标志与用量预算相关（仅限 API key 用户、print 模式）：

| 标志 | 描述 |
|------|-------------|
| `--max-budget-usd <AMOUNT>` | API 调用花费达到该美元金额后停止 |
| `--max-turns <NUMBER>` | 限制 agentic 轮数 |

完整列表见 [CLI 启动标志参考](claude-cli-startup-flags.md)。

---

## 参考来源

- [Extra usage for paid Claude plans — Claude Help Center](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)
- [Using Claude Code with your Pro or Max plan — Claude Help Center](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan)
- [/extra-usage slash command is undocumented — GitHub Issue #12396](https://github.com/anthropics/claude-code/issues/12396)
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
