# Squash 合并与 PR 体量分布——来自 Boris Cherny 的技巧

> 译自 [tips/claude-boris-2-tips-25-mar-26.md](../../tips/claude-boris-2-tips-25-mar-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 创造者 Boris Cherny（[@bcherny](https://x.com/bcherny)）于 2026 年 3 月 25 日分享的洞见整理。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1/ 单日 266 次贡献——永远用 Squash

Boris 分享了他的 GitHub 贡献图：**3 月 24 日当天 266 次贡献**——来自 **141 个 PR，全部采用 squash 合并**，每个 PR 的中位数为 **118 行**。

- Squash 合并把分支上的所有提交合并为目标分支上的单个提交——保持历史干净、线性
- 一个 PR 等于一个提交，便于整体回滚整个功能，也让 `git bisect` 更简单
- 在 AI 辅助的高节奏 workflow（工作流，141 个 PR/天）下，squash 是务实之选——分支内那些"修 lint""试试这个"的散碎提交都是噪声

<a href="https://x.com/bcherny/status/2038552880018538749"><img src="../../tips/assets/boris-26-3-25/1.png" alt="Boris Cherny——266 次贡献，全部 squash" width="50%" /></a>

---

## 2/ PR 体量分布——保持 PR 小巧

Boris 分享了这 141 个 PR 的体量分布，合计 **45,032 行变更**（新增 + 删除）：

| 指标 | 行数（增+删） | 含义 |
|--------|---------------:|---------|
| **p50** | **118** | PR 体量中位数——一半的 PR 不超过 118 行 |
| p90 | 498 | 90% 的 PR 少于 500 行 |
| **p99** | **2,978** | 只有约 1 个 PR 超过约 3K 行 |
| min | 2 | 最小的 PR——一次 2 行的快速修复 |
| max | 10,459 | 最大的单个 PR——很可能是迁移或生成的代码 |

- **中位数 118 行**意味着即便一天 141 个 PR，大多数 PR 仍然聚焦、可审查
- 分布严重右偏——偶发的大 PR 不可避免（批量重命名、迁移），但常态是紧凑的
- 小 PR 降低合并冲突风险、更容易审查，并与 squash 合并完美搭配，实现干净的回滚

<a href="https://x.com/bcherny/status/2038552880018538749"><img src="../../tips/assets/boris-26-3-25/2.png" alt="Boris Cherny——PR 体量分布表" width="50%" /></a>

---

## 来源

- [Boris Cherny (@bcherny) on X —— 2026 年 3 月 25 日](https://x.com/bcherny)
