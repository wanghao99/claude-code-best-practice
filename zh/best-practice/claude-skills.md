# Skills 最佳实践

> 译自 [best-practice/claude-skills.md](../../best-practice/claude-skills.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。
> 原文同步至 Claude Code v2.1.310，最后更新 2026-10-26。

![Last Updated](https://img.shields.io/badge/Last_Updated-Oct%2026%2C_2026%2010%3A04%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.310-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../implementation/claude-skills-implementation.md)

Claude Code skill（技能）— frontmatter 字段与官方内置 skill。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter 字段（20 个）

| 字段 | 类型 | 必填 | 说明 |
|-------|------|----------|-------------|
| `name` | string | 否 | 显示名称和 `/slash-command` 标识符。省略时默认用目录名 |
| `description` | string | 推荐 | 这个 skill 做什么。显示在自动补全中，并供 Claude 自动发现使用 |
| `when_to_use` | string | 否 | 补充说明 Claude 何时该调用这个 skill — 触发短语和示例请求。在 skill 列表中追加到 `description` 后面，计入 1,536 字符上限 |
| `argument-hint` | string | 否 | 自动补全时显示的参数提示（如 `[issue-number]`、`[filename]`） |
| `arguments` | string/list | 否 | 命名位置参数，用于 skill 内容中的 `$name` 替换。接受空格分隔字符串或 YAML 列表 — 名称按顺序对应参数位置 |
| `disable-model-invocation` | boolean | 否 | 设为 `true` 阻止 Claude 自动调用这个 skill |
| `user-invocable` | boolean | 否 | 设为 `false` 使其不出现在 `/` 菜单 — skill 变为纯背景知识，供 agent 预载使用 |
| `allowed-tools` | string/list | 否 | skill 激活期间无需权限确认即可使用的工具 |
| `disallowed-tools` | string/list | 否 | skill 激活期间从 Claude 可用工具池中移除的工具（例如为后台循环屏蔽 `AskUserQuestion`）。接受空格/逗号分隔字符串或 YAML 列表 — 限制在下一条消息时清除 |
| `model` | string | 否 | skill 运行时使用的模型（如 `haiku`、`sonnet`、`opus`） |
| `effort` | string | 否 | 调用时覆盖模型的思考强度（`low`、`medium`、`high`、`xhigh`、`max`） |
| `context` | string | 否 | 设为 `fork` 时在隔离的 subagent 上下文中运行 |
| `agent` | string | 否 | `context: fork` 时的 subagent 类型（默认：`general-purpose`） |
| `background` | boolean | 否 | 仅在 `context: fork` 时生效。设为 `false` 时在调用轮次内等待 fork 出的 subagent 返回结果，而不是后台运行（默认：`true`）。需要 v2.1.218+ |
| `hooks` | object | 否 | 作用于该 skill 的生命周期钩子 |
| `paths` | string/list | 否 | 限制 skill 自动激活条件的 glob 模式。接受逗号分隔字符串或 YAML 列表 — 只有处理匹配文件时 Claude 才加载该 skill |
| `shell` | string | 否 | `` !`command` `` 块使用的 shell — `bash`（默认）或 `powershell`。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `metadata` | YAML map | 否 | 自由格式的 YAML 键值对（如授权、目录字段），供你自己的工具从 `SKILL.md` 读取。Claude Code 不处理其内容；非 map 的值会被丢弃。不要复用 frontmatter 字段名（如 `paths`）作为键 |
| `license` | string | 否 | skill 的许可证。[Agent Skills](https://agentskills.io) 规范的一部分。Claude Code 接受该字段但不处理 |
| `compatibility` | string | 否 | skill 的环境要求（最多 500 字符），如目标产品或系统前置条件。[Agent Skills](https://agentskills.io) 规范的一部分。Claude Code 接受该字段但不处理 |

---

## 官方内置 Skill（25 个）

| # | Skill | 说明 |
|---|-------|-------------|
| 1 | `code-review` | 按选定强度审查当前 diff 中的正确性 bug（low/medium：更少但置信度高的发现；high 到 max：覆盖更广）— `--comment` 将发现以内联评论发到 PR |
| 2 | `batch` | 批量对多个文件执行命令 |
| 3 | `debug` | 调试失败的命令或代码问题 |
| 4 | `loop` | 按固定间隔循环执行一个提示或斜杠命令（最长 3 天） |
| 5 | `claude-api` | 用 Claude API 或 Anthropic SDK 构建应用 — 在 `anthropic` / `@anthropic-ai/sdk` 导入时触发 |
| 6 | `fewer-permission-prompts` | 扫描会话记录中常见的只读 Bash/MCP 调用，向 `.claude/settings.json` 添加推荐白名单以减少权限确认 |
| 7 | `run` | 启动并操作项目应用，在真实应用里看改动效果（而不只是测试）。需要 v2.1.145 |
| 8 | `verify` | 构建并运行应用，确认代码改动确实生效，不退化为只跑测试或类型检查。需要 v2.1.145 |
| 9 | `run-skill-generator` | 教 `/run` 和 `/verify` 如何构建、启动该项目 — 在 `.claude/skills/run-<name>/` 记录每个项目的启动配方。需要 v2.1.145 |
| 10 | `simplify` | 审查已改动代码的可清理之处（复用、简化、效率、抽象层级），四个审查 agent 并行。从 v2.1.154 起**不再**找正确性 bug — 那个用 `/code-review` |
| 11 | `design` | 在同一画布上起草 UI 模型图、页面流、落地页或海报，作为 artboard 发布为 artifact，运行 Claude Design 编辑器的研究预览。账户开通保存时可编辑并发布新版本；否则仅查看并导出 PNG 或 PDF。需要 artifact 可用的会话。Anthropic API 可用；Amazon Bedrock、Google Cloud Agent Platform、Microsoft Foundry 不可用。v2.1.234 引入 |
| 12 | `design-sync` | 把仓库里的 React 设计系统转换并上传到 Claude Design — 可选命名设计系统（如 `/design-sync Acme DS`）。首次同步会验证每个组件，大仓库可能耗时数小时。仅 Anthropic API 可用（Bedrock、Google Cloud Agent Platform、Microsoft Foundry 不可用）。独立版 Claude Design 将于 2026 年 12 月 14 日关闭 |
| 13 | `dataviz` | 设计图表、图形和仪表盘，内置调色板校验器保证可视化无障碍且风格一致 — 任何输出媒介中的图表请求都能触发。v2.1.198 引入 |
| 14 | `doctor` | Claude Code 配置的安装/健康检查。唯一不受 `disableBundledSkills` 影响的内置 skill — 该设置开启时仍可输入调用。v2.1.205 从内置命令重新归类为内置 skill |
| 15 | `review` | 对 GitHub PR 做快速单遍只读审查。需要多 agent 深度审查时改用 `/code-review <level> <pr#>`。v2.1.186 到 v2.1.201 曾使用 `/code-review medium` 引擎；v2.1.202 恢复为快速单遍审查。v2.1.108 起可被 Skill 工具调用 |
| 16 | `security-review` | 审查当前 diff 中的安全漏洞并给出修复建议。传 `--fix` 应用发现，传 `--comment` 以内联评论发到 GitHub PR。v2.1.108 起可被 Skill 工具调用 |
| 17 | `workflow-authoring` | 加载编写动态 workflow 脚本的参考：脚本 API、恢复行为、质量模式和完整示例。写脚本前 Claude 通常会自行加载。仅在启用动态 workflow 时可用。需要 v2.1.248 |
| 18 | `skill-doctor` | 报告哪些已加载的 skill 从未被使用、各自占用多少上下文，便于裁剪。v2.1.261 引入 |
| 19 | `update-config` | 通过 `settings.json` 配置 Claude Code harness — 描述任何设置变更（放行一条命令、设环境变量、加钩子），Claude 编辑对应的设置文件；主题、模型等简单选项用 `/config` 即可 |
| 20 | `slides` | 按你的简报生成一份 Claude Slides artifact 演示文稿（如 `/slides a quarterly review for the sales team`） |
| 21 | `artifact-capabilities` | 加载已发布 artifact 可用的运行时能力参考，如调用连接器或提供文件下载，包括你的账户具备哪些。构建使用这类能力的页面前 Claude 通常会自行加载。artifact 可用的地方即可用 |
| 22 | `artifact-diagramming` | 加载 artifact 的图表绘制指南：何时需要图、画什么、如何写内联 SVG 并保持明暗主题下都清晰。artifact 中需要图时 Claude 通常会自行加载。需要 v2.1.221+ |
| 23 | `claude-in-chrome` | 通过 Claude in Chrome 让 Claude 在你的浏览器里执行任务（测试页面、填表、读控制台日志）。启用 Chrome 集成时可用（如 `claude --chrome`） |
| 24 | `artifact-design` | 加载已发布 artifact 页面的设计指南与基础 — 布局、主题、尺寸和文字风格。构建任何 artifact 前 Claude 通常会自行加载。v2.1.281 引入 |
| 25 | `plugin-authoring` | 制作 mod — 对 Claude 界面或行为本身的修改，如实时面板、输入框上方的条带、状态栏、toast、斜杠命令，或工具调用/提示上的钩子。用于制作、修改或调试 mod 及其钩子模块。需要 v2.1.287+ |

另见：[官方 Skills 仓库](https://github.com/anthropics/skills/tree/main/skills)，收录社区维护的可安装 skill。

---

## 参考来源

- [Claude Code Skills — 官方文档](https://code.claude.com/docs/en/skills)
- [Monorepo 中的 Skill 发现](../../reports/claude-skills-for-larger-mono-repos.md)（英文原文）
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
