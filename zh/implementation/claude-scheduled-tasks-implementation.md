# Scheduled Tasks 实现指南

> 译自 [implementation/claude-scheduled-tasks-implementation.md](../../implementation/claude-scheduled-tasks-implementation.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_10%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#loop-demo"><img src="../../!/tags/implemented-hd.svg" alt="Implemented"></a>

`/loop` skill 用于按 cron 间隔调度循环任务。下面是 `/loop 1m "tell current time"` 的演示 — 一个每分钟触发一次的简单循环任务。

---

## Loop 演示

### 1. 调度任务

<p align="center">
  <img src="../../implementation/assets/impl-loop-1.png" alt="/loop 1m tell current time — scheduling and cron setup" width="100%">
</p>

`/loop 1m "tell current time"` 会解析时间间隔（`1m` → 每 1 分钟），创建 cron 任务并确认调度。要点：

- Cron 的最小粒度是 **1 分钟** — `1m` 对应 `*/1 * * * *`
- 循环任务 **3 天后自动过期**
- 任务是 **会话级** 的 — 只存在于内存中，Claude 退出即停止
- 随时可用 `cron cancel <job-id>` 取消

---

### 2. Loop 运行效果

<p align="center">
  <img src="../../implementation/assets/impl-loop-2.png" alt="Recurring task firing every minute" width="100%">
</p>

该任务每分钟触发一次，执行 `date` 并报告当前时间。每次迭代都会异步触发 **UserPromptSubmit** 和 **Stop** hook — 也就是本仓库通篇用于声音通知的同一套 hook 系统。

---

## ![How to Use](../../!/tags/how-to-use.svg)

```bash
$ claude
> /loop 1m "tell current time"
> /loop 5m /simplify
> /loop 10m "check deploy status"
```

---

## ![How to Implement](../../!/tags/how-to-implement.svg)

`/loop` 是 Claude Code 内置 skill — 无需任何配置。它底层使用 cron 工具（`CronCreate`、`CronList`、`CronDelete`）管理循环调度。
