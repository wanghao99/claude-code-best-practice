# 定制 Claude Code 的 12 种方式——Boris 的提示（2026-02-12）

> 译自 [tips/claude-boris-12-tips-12-feb-26.md](../../tips/claude-boris-12-tips-12-feb-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 创造者 Boris Cherny（[@bcherny](https://x.com/bcherny)）于 2026 年 2 月 12 日分享的定制技巧汇总。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 背景

Boris Cherny 强调，可定制性是工程师最爱 Claude Code 的原因之一——hook、plugin、LSP、MCP、Skill、effort、自定义 agent、状态栏、输出风格等等。他分享了开发者和团队定制配置的 12 种实用方式。

<a href="https://x.com/bcherny/status/2021699851499798911"><img src="../../tips/assets/boris-26-2-12/0.webp" alt="Boris Cherny 开场推文" width="50%" /></a>

---

## 1/ 配置你的终端

为获得最佳 Claude Code 体验配置终端：

- **主题**：运行 `/config` 设置浅色/深色模式
- **通知**：为 iTerm2 启用通知，或使用自定义通知 hook
- **换行**：如果在 IDE 终端、Apple Terminal、Warp 或 Alacritty 里使用 Claude Code，运行 `/terminal-setup` 启用 shift+enter 换行（不用再输入 `\`）
- **Vim 模式**：运行 `/vim`

<a href="https://x.com/bcherny/status/2021699859359883608"><img src="../../tips/assets/boris-26-2-12/1.webp" alt="配置你的终端" width="50%" /></a>

---

## 2/ 调整 Effort 等级

运行 `/model` 选择你偏好的 effort 等级：

- **Low** —— token 更少、响应更快
- **Medium** —— 均衡表现
- **High** —— token 更多、智能更强

Boris 的偏好：所有事都用 High。

<a href="https://x.com/bcherny/status/2021699860869902424"><img src="../../tips/assets/boris-26-2-12/2.webp" alt="调整 effort 等级" width="50%" /></a>

---

## 3/ 安装 plugin、MCP 与 Skill

Plugin 让你可以安装 LSP（每种主流语言都有）、MCP、Skill、agent 和自定义 hook。

从 Anthropic 官方 plugin 市场安装，或为公司创建你自己的市场。把 `settings.json` 提交进代码库，即可为团队自动添加这些市场。

运行 `/plugin` 开始。

<a href="https://x.com/bcherny/status/2021699862522364149"><img src="../../tips/assets/boris-26-2-12/3.webp" alt="安装 plugin、MCP 与 Skill" width="50%" /></a>

---

## 4/ 创建自定义 agent

把 `.md` 文件放进 `.claude/agents` 即可创建自定义 agent。每个 agent 可以有自己的名称、颜色、工具集、预先允许和预先禁用的工具、权限模式和模型。

还可以通过 `settings.json` 中的 `"agent"` 字段或 `--agent` 标志，为主会话设置默认 agent。

运行 `/agents` 开始。

<a href="https://x.com/bcherny/status/2021700144039903699"><img src="../../tips/assets/boris-26-2-12/4.webp" alt="创建自定义 agent" width="50%" /></a>

---

## 5/ 预先批准常见权限

Claude Code 的权限系统结合了提示注入检测、静态分析、沙箱和人工监督。

开箱即用时会预先批准一小组安全命令。要预先批准更多，运行 `/permissions` 并添加到允许和阻止列表。把这些提交进团队的 `settings.json`。

支持完整的通配符语法——例如 `Bash(bun run *)` 或 `Edit(/docs/**)`。

<a href="https://x.com/bcherny/status/2021700332292911228"><img src="../../tips/assets/boris-26-2-12/5.webp" alt="预先批准常见权限" width="50%" /></a>

---

## 6/ 启用沙箱

选择加入 Claude Code 的开源沙箱运行时，在减少权限提示的同时提升安全性。

运行 `/sandbox` 启用。沙箱在你自己的机器上运行，支持文件和网络隔离。

<a href="https://x.com/bcherny/status/2021700506465579443"><img src="../../tips/assets/boris-26-2-12/6.webp" alt="启用沙箱" width="50%" /></a>

---

## 7/ 添加状态栏

自定义状态栏就显示在输入框下方，展示模型、目录、剩余上下文、费用，以及你工作时想看到的任何其他信息。

每个团队成员可以有不同的状态栏。用 `/statusline` 让 Claude 根据你的 `.bashrc`/`.zshrc` 生成一个。

<a href="https://x.com/bcherny/status/2021700784019452195"><img src="../../tips/assets/boris-26-2-12/7.webp" alt="添加状态栏" width="50%" /></a>

---

## 8/ 自定义你的按键绑定

Claude Code 里的每个按键绑定都可以自定义。运行 `/keybindings` 重新映射任意按键。设置支持热重载，立刻就能感受效果。

<a href="https://x.com/bcherny/status/2021700883873165435"><img src="../../tips/assets/boris-26-2-12/8.webp" alt="自定义你的按键绑定" width="50%" /></a>

---

## 9/ 设置 hook

Hook 让你确定性地接入 Claude 的生命周期：

- 自动把权限请求转发到 Slack 或 Opus
- 在 Claude 到达一轮结束处时提醒它继续（甚至可以启动一个 agent，或用一个提示词判断 Claude 是否应该继续）
- 预处理或后处理工具调用，例如添加你自己的日志

让 Claude 加一个 hook 即可上手。

<a href="https://x.com/bcherny/status/2021701059253874861"><img src="../../tips/assets/boris-26-2-12/9.webp" alt="设置 hook" width="50%" /></a>

---

## 10/ 自定义你的转圈动词

自定义你的转圈动词（spinner verbs），在默认列表之外增删，换成你自己的动词。把 `settings.json` 提交进版本控制，与团队共享。

<a href="https://x.com/bcherny/status/2021701145023197516"><img src="../../tips/assets/boris-26-2-12/10.webp" alt="自定义转圈动词" width="50%" /></a>

---

## 11/ 使用输出风格

运行 `/config` 设置一种输出风格，让 Claude 用不同的语气或格式回复。

- **Explanatory（解释型）**—— 熟悉新代码库时推荐，让 Claude 边工作边解释框架和代码模式
- **Learning（学习型）**—— 让 Claude 以教练方式带你做代码改动
- **Custom（自定义）**—— 创建自定义输出风格，调整 Claude 的语气

<a href="https://x.com/bcherny/status/2021701379409273093"><img src="../../tips/assets/boris-26-2-12/11.webp" alt="使用输出风格" width="50%" /></a>

---

## 12/ 定制一切！

Claude Code 开箱即用就很好，但当你真的做定制时，把 `settings.json` 提交进 git，团队也能受益。配置支持多个层级：

- 针对你的代码库
- 针对某个子目录
- 仅针对你自己
- 通过企业级策略

共有 37 项设置和 84 个环境变量（用 `settings.json` 里的 `"env"` 字段，避免写包装脚本），你想要的任何行为大概率都能配置出来。

<a href="https://x.com/bcherny/status/2021701636075458648"><img src="../../tips/assets/boris-26-2-12/12.webp" alt="定制一切" width="50%" /></a>

---

## 来源

- [Boris Cherny (@bcherny) on X —— 2026 年 2 月 12 日](https://x.com/bcherny)
- [Claude Code 终端配置文档](https://code.claude.com/docs/en/terminal)
- [Claude Code 插件与发现文档](https://code.claude.com/docs/en/discover-plugins)
- [Claude Code sub-agent 文档](https://code.claude.com/docs/en/sub-agents)
- [Claude Code 权限文档](https://code.claude.com/docs/en/permissions)
- [Claude Code 沙箱文档](https://code.claude.com/docs/en/sandbox)
- [Claude Code 状态栏文档](https://code.claude.com/docs/en/statusline)
- [Claude Code 键盘快捷键文档](https://code.claude.com/docs/en/keybindings)
- [Claude Code Hook 参考](https://code.claude.com/docs/en/hooks)
- [Claude Code 输出风格文档](https://code.claude.com/docs/en/output-styles)
- [Claude Code 设置文档](https://code.claude.com/docs/en/settings)
