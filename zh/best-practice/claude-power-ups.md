# Power-ups 最佳实践

> 译自 [best-practice/claude-power-ups.md](../../best-practice/claude-power-ups.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。
> 原文最后更新 2026-04-02。

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2002%2C_2026-white?style=flat&labelColor=555)

互动课程，用动画演示讲解 Claude Code 的功能。每节课讲一个大多数人都会错过的 Claude Code 能力。v2.1.90 引入。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 用法

```bash
claude
/powerup
```

---

## Power-ups（10 节）

<p align="center">
  <img src="../../best-practice/assets/claude-power-ups/powerup-menu.png" alt="Power-ups 菜单，显示 10 节课程" width="700">
</p>

| # | 课程 | 主题 |
|---|----------|--------|
| 1 | 和你的代码库对话 | `@` 文件引用、行号引用 |
| 2 | 用模式掌控方向 | `shift+tab`、plan、auto |
| 3 | 撤销任何操作 | `/rewind`、`Esc-Esc` |
| 4 | 后台运行 | tasks、`/tasks` |
| 5 | 教 Claude 你的规则 | `CLAUDE.md`、`/memory` |
| 6 | 用工具扩展 | MCP、`/mcp` |
| 7 | 自动化你的工作流 | skills、hooks |
| 8 | 复制多个自己 | subagents、`/agents` |
| 9 | 随时随地写代码 | `/remote-control`、`/teleport` |
| 10 | 调节模型 | `/model`、`/effort` |

---

## 示例：调节模型

最后一节课用动画演示模型切换和思考强度控制。

<p align="center">
  <img src="../../best-practice/assets/claude-power-ups/dial-the-model-1.png" alt="调节模型 — 演示深度思考" width="700">
</p>

<p align="center">
  <img src="../../best-practice/assets/claude-power-ups/dial-the-model-2.png" alt="调节模型 — 演示展示假设" width="700">
</p>

<p align="center">
  <img src="../../best-practice/assets/claude-power-ups/dial-the-model-3.png" alt="调节模型 — 演示将强度设为 high" width="700">
</p>

---

## 参考来源

- [Changelog — v2.1.90](https://code.claude.com/docs/en/changelog)
