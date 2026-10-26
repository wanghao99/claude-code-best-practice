# Commands 最佳实践

> 译自 [best-practice/claude-commands.md](../../best-practice/claude-commands.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。
> 原文同步至 Claude Code v2.1.310，最后更新 2026-10-25。

![Last Updated](https://img.shields.io/badge/Last_Updated-Oct%2025%2C_2026%2011%3A07%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.310-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../implementation/claude-commands-implementation.md)

Claude Code command（命令）— frontmatter 字段与官方内置斜杠命令。

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
| `name` | string | 否 | 显示在 `/` 菜单中的命令名。默认用目录名 |
| `description` | string | 推荐 | 命令做什么。显示在自动补全中，并供 Claude 自动发现使用。省略时用 markdown 内容的第一个非空行。把关键用例写在最前面：列表中 `description` 和 `when_to_use` 合计在 1,536 字符处截断 |
| `when_to_use` | string | 否 | 补充说明 Claude 何时该调用 — 触发短语或示例请求。在列表中追加到 `description` 后，计入 1,536 字符上限 |
| `argument-hint` | string | 否 | 自动补全时显示的参数提示（如 `[issue-number]`、`[filename] [format]`） |
| `arguments` | string/list | 否 | 命名位置参数，用于命令内容中的 `$name` 替换。接受空格分隔字符串或 YAML 列表 — 名称按顺序对应参数位置 |
| `disable-model-invocation` | boolean | 否 | 设为 `true` 阻止 Claude 自动调用。同时阻止预载进 subagent，以及以该 skill 为提示词的会话级定时任务触发。默认：`false` |
| `user-invocable` | boolean | 否 | 设为 `false` 使其不出现在 `/` 菜单 — 命令变为纯背景知识。你输入 `/name` 时 Claude Code 也不会执行。默认：`true` |
| `paths` | string/list | 否 | 限制激活条件的 glob 模式。接受逗号分隔字符串或 YAML 列表。设置后，只有处理匹配文件时 Claude 才自动加载。模式相对于 Claude 当前工作目录，目录之外的文件不匹配 |
| `allowed-tools` | string/list | 否 | 调用该 skill 的轮次内 Claude 无需确认即可使用的工具。授权在你发送下一条消息时清除。接受空格/逗号分隔字符串或 YAML 列表 |
| `disallowed-tools` | string/list | 否 | 命令激活期间从 Claude 可用工具池移除的工具。用于绝不该调用某些工具的自主 skill。接受空格/逗号分隔字符串或 YAML 列表。限制在下一条消息时清除。只要还有其他工具可用，就无法移除 `EndConversation` |
| `model` | string | 否 | 命令运行时使用的模型（如 `haiku`、`sonnet`、`opus`）。仅当前轮次生效；下一条消息恢复会话模型。接受与 `/model` 相同的值，或 `inherit` 保持当前模型。被组织 `availableModels` 白名单排除、或 auto 模式不支持的值会被忽略，会话保持当前模型 |
| `effort` | string | 否 | skill 激活期间的思考强度，覆盖会话设置。可选：`low`、`medium`、`high`、`xhigh`、`max`；可用档位取决于模型 |
| `context` | string | 否 | 设为 `fork` 在 fork 出的 subagent 上下文中运行 |
| `agent` | string | 否 | `context: fork` 时使用的 subagent 类型（默认：`general-purpose`） |
| `background` | boolean | 否 | 仅在 `context: fork` 时生效。设为 `false` 在调用轮次内等待 fork 出的 subagent 返回结果，而不是后台运行。默认：`true`。需要 v2.1.218+ |
| `shell` | string | 否 | `` !`command` `` 块使用的 shell — `bash`（默认）或 `powershell`。无 Git Bash 的 Windows 默认开启 PowerShell 工具，有 Git Bash 的 Windows 对 claude.ai 和 Console 账户开启；Bedrock、Agent Platform、Foundry 以及 macOS、Linux、WSL 需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `metadata` | object | 否 | 自由格式 YAML 键值对，供你自己的工具从 `SKILL.md` 读取。Claude Code 不处理其内容，非 map 的值会被丢弃。不要复用保留字段名（如 `paths`）作为键 |
| `license` | string | 否 | 按 [Agent Skills](https://agentskills.io) 规范声明的许可证。Claude Code 接受该字段但不处理 |
| `compatibility` | string | 否 | 按 [Agent Skills](https://agentskills.io) 规范声明的环境要求，如目标产品或系统前置条件。最多 500 字符。Claude Code 接受该字段但不处理 |
| `hooks` | object | 否 | skill 被调用时 Claude Code 注册并在会话其余时间持续运行的钩子 |

---

## 官方内置命令（94 个）

| # | 命令 | 分类 | 说明 |
|---|---------|-----|-------------|
| 1 | `/design-login` | 认证 | 为 `/design-sync` 用 claude.ai 账号授权设计系统访问。登录独立版 Claude Design 站点 claude.ai/design，该站点 2026 年 12 月 14 日关闭 |
| 2 | `/login` | 认证 | 登录 Anthropic 账户 |
| 3 | `/logout` | 认证 | 退出 Anthropic 账户 |
| 4 | `/setup-bedrock` | 认证 | 通过交互式向导配置 Amazon Bedrock 认证、区域和模型绑定。仅设置 `CLAUDE_CODE_USE_BEDROCK=1` 时可见。首次使用 Bedrock 的用户也可从登录页进入该向导 |
| 5 | `/setup-vertex` | 认证 | 通过交互式向导配置 Google Cloud Agent Platform 认证、项目、区域和模型绑定。仅设置 `CLAUDE_CODE_USE_VERTEX=1` 时可见。首次用户也可从登录页进入 |
| 6 | `/upgrade` | 认证 | 打开升级页面切换到更高套餐。浏览器打开失败时，显示登录提示而不打印 URL |
| 7 | `/auto-mode-setup` | 配置 | 从项目和近期会话起草 `autoMode.environment` 条目，确认后保存到用户设置。Pro、Max、Team 套餐需要 v2.1.228+（原生 Windows 需 v2.1.233+）。Enterprise 和 Claude Console API key 需要 v2.1.283+。命令不存在时参考 `/auto-mode-setup` 返回 'Unknown command' 的排查文档 |
| 8 | `/color [color\|default]` | 配置 | 设置当前会话的输入框颜色。可用颜色：`red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink`、`cyan`。`default` 重置。不带参数则随机选色。Remote Control 连接时颜色同步到 claude.ai/code。非交互模式（`-p`）也可用 |
| 9 | `/config [key=value ...]` | 配置 | 打开设置界面调整主题、模型、输出风格等。v2.1.181 起可传一个或多个 `key=value` 直接设置而不打开界面，如 `/config theme=dark` 或 `/config model=sonnet`。`key=value` 形式无法打开需要在面板确认的设置（如 `autoContinueAtUsageLimit`），但可以关闭它们。非交互（`-p`）模式和 Claude 移动端 Remote Control 也可用。运行 `/config --help` 查看可设置的键。别名：`/settings` |
| 10 | `/focus [on\|off]` | 配置 | 切换专注视图：只显示你最近一条提示、单行工具调用摘要（含编辑的 diff 统计）和最终回复。工具调用摘要还会统计本轮启动的 subagent 数量，把已完成的后台任务通知合并为一个计数。选择跨会话保留；可在设置中用 `viewMode` 覆盖。仅全屏渲染下可用。Remote Control 客户端执行 `/focus [on\|off]` 只对当前会话生效，不改变保存的选择；需要 v2.1.281+。VS Code 扩展有独立的 Focus 视图开关，与 `viewMode` 无关 |
| 11 | `/import [codex\|gemini\|cursor] [--dry-run] [--yes]` | 配置 | 把机器上其他编码 agent 的配置（目前支持 OpenAI Codex、Google Gemini CLI、Cursor）迁入 Claude Code，包括指令文件、MCP 服务器、命令、subagent 和 skill。非交互模式（`-p`）下列出发现的内容并给出确认导入的命令。`--dry-run` 预览不写入；`--yes` 跳过交互选择。从 Cursor 导入需要 v2.1.265+。Bedrock、Agent Platform (Vertex)、Foundry、Claude Platform on AWS、Claude apps 网关或关闭 feature flag 获取时不可用 |
| 12 | `/keybindings` | 配置 | 打开或创建快捷键配置文件 |
| 13 | `/output-style [style]` | 配置 | 列出输出风格或切换，如 `/output-style concise`。需要 v2.1.269+ |
| 14 | `/permissions` | 配置 | 管理工具权限的允许、询问和拒绝规则。打开交互对话框，可按作用域查看规则、增删规则、管理工作目录、查看近期 auto mode 拒绝记录。对话框的 Auto mode 标签页还可查看和编辑 auto mode 分类器规则。Claude 正在回复时运行，对话框立即打开，改动从本轮 Claude 的下一个工具调用起生效。v2.1.234 之前会排队到本轮结束。别名：`/allowed-tools` |
| 15 | `/powerup` | 配置 | 通过带动画演示的快速互动课程了解 Claude Code 功能 |
| 16 | `/privacy-settings` | 配置 | 查看和更新隐私设置。仅 Pro 和 Max 套餐可用 |
| 17 | `/radio` | 配置 | 在浏览器打开 Claude FM lo-fi 电台。无浏览器时打印流地址 |
| 18 | `/sandbox` | 配置 | 切换沙箱模式。仅支持的平台上可用 |
| 19 | `/scroll-speed` | 配置 | 交互式调整滚轮速度。仅全屏渲染下可用。JetBrains 终端不可用 |
| 20 | `/statusline` | 配置 | 配置状态栏。描述你想要的效果，或不带参数时从 shell 提示符自动配置 |
| 21 | `/stickers` | 配置 | 订购 Claude Code 贴纸 |
| 22 | `/terminal-setup` | 配置 | 配置终端的 Shift+Enter 等快捷键。仅在需要的终端中可见，如 VS Code、Cursor、Devin Desktop、Alacritty、Zed。Apple 终端中启用 Option+Enter 换行并关闭提示音。iTerm2 中开启剪贴板访问使 `/copy` 可用 |
| 23 | `/theme` | 配置 | 更换配色主题。含明暗两版、色盲友好（daltonized）主题、使用终端调色板的 ANSI 主题、跟随终端明暗模式的"Auto (match terminal)"、以及从 `~/.claude/themes/` 或插件加载的自定义主题。选"New custom theme…"可自建 |
| 24 | `/tui [default\|fullscreen]` | 配置 | 设置终端 UI 渲染器并保持对话重新启动。`fullscreen` 启用无闪烁的备用屏幕渲染器。不带参数时打印当前渲染器 |
| 25 | `/voice [hold\|tap\|off]` | 配置 | 切换语音听写，或以指定模式启用。需要 Claude.ai 账户 |
| 26 | `/autocompact [auto\|<tokens>]` | 上下文 | 设置自动压缩阈值：上下文窗口占用到多少时自动压缩。传具体大小如 `500k`，或 `auto` 恢复为模型默认值。值保存到用户设置并应用到当前会话。不带参数时打开显示当前阈值的对话框。需要 v2.1.221+ |
| 27 | `/context [all]` | 上下文 | 用彩色网格可视化当前上下文占用。对上下文消耗大的工具、记忆膨胀和容量告警给出优化建议。对话超出上下文窗口时，输出包含超限幅度和可用哪个命令释放空间的警告。全屏模式下 `/context` 折叠逐项明细以保持网格可见。传 `all` 展开 |
| 28 | `/cost` | 上下文 | `/usage` 的别名 |
| 29 | `/insights` | 上下文 | 生成 HTML 报告，分析本机的 Claude Code 会话，包括项目领域、交互模式和摩擦点。云会话不可用 |
| 30 | `/rate-limit-options` | 上下文 | 显示 claude.ai 用量限制拦截请求时的继续工作方式：等待并在限制重置后自动继续、添加用量额度、或升级套餐。本机终端触达限制时 Claude Code 也会自行打开此菜单。需要 claude.ai 订阅。等待继续功能需要 v2.1.234+ |
| 31 | `/stats` | 上下文 | `/usage` 的别名。打开 Stats 标签页 |
| 32 | `/status` | 上下文 | 打开设置界面的 Status 标签页，显示版本、模型、账户和连接状态。后台会话中 `Session kind` 一行显示 `background job · attached` 或 `background job · unattended`（取决于是否连接了终端），其他会话显示 `interactive`。v2.1.221 之前无此行。Claude 回复中也可用 |
| 33 | `/usage` | 上下文 | 显示会话成本、套餐用量限制和活动统计。Pro、Max、Team、Enterprise 套餐下包含占用套餐额度的明细。`/cost` 和 `/stats` 是别名 |
| 34 | `/usage-credits` | 上下文 | 配置用量额度以便触达限制后继续工作。Team 和 Enterprise 套餐中无账单权限的成员从 CLI 向管理员发送额度申请而不打开浏览器（v2.1.248+）。无浏览器可开时（如 SSH）打印 URL（v2.1.205+）。前身为 `/extra-usage` |
| 35 | `/bug [report]` | 调试 | 报告 bug 或分享对话。你选择包含多少会话历史，发送前在授权确认页确认。第三方供应商或无 Anthropic 凭据时，报告写入本地 `~/.claude/feedback-bundles/`。VS Code 扩展中打开扩展自己的反馈对话框（需要 v2.1.229+）。Claude 回复中运行时对话框立即打开。v2.1.232 之前排队到本轮结束。v2.1.212 之前 `/bug` 和 `/share` 是 `/feedback` 的别名。别名：`/share` |
| 36 | `/feedback [report]` | 调试 | 发送关于 Claude Code 的产品反馈。打开与 `/bug` 相同的对话框，同样有授权步骤、发送规则和回复中行为。不带参数且有 Claude 起草的反馈时，打开草稿队列供你审阅、编辑、发送或丢弃；队列里也可新写一份。带参数时直接打开对话框 |
| 37 | `/heapdump` | 调试 | 将 JavaScript 堆快照和内存明细写到 `~/Desktop` 用于诊断内存占用（Linux 无桌面目录时写主目录）。不显示在命令菜单中，需完整输入。只分享 `-diagnostics.json` 文件，`.heapsnapshot` 包含你的对话和凭据。报告内存增长类 bug 时有用 |
| 38 | `/help` | 调试 | 显示帮助和可用命令 |
| 39 | `/release-notes` | 调试 | 以交互式版本选择器查看 changelog。选具体版本看发布说明，或选择显示全部。说明显示在记录中但不进入 Claude 可见的对话 |
| 40 | `/tasks` | 调试 | 查看和管理当前会话的后台工作，包括已完成的 subagent。也可用 `/bashes` |
| 41 | `/copy [N]` | 导出 | 复制最近一条助手回复到剪贴板。传数字 `N` 复制倒数第 N 条：`/copy 2` 复制倒数第二条。存在代码块或引用块时，显示交互选择器挑选单个块或整条回复。选择器中按 `w` 将所选写入文件而非剪贴板，SSH 场景下有用 |
| 42 | `/export [filename]` | 导出 | 将当前对话导出为纯文本。带文件名时直接写入该文件；不带时打开对话框选择复制到剪贴板或保存为文件 |
| 43 | `/agents` | 扩展 | 打印提示：让 Claude 创建或管理 subagent，或直接编辑 `.claude/agents/` 或 `~/.claude/agents/`。v2.1.197 及之前打开创建和管理 subagent 配置的交互界面 |
| 44 | `/chrome` | 扩展 | 配置 Claude in Chrome 设置 |
| 45 | `/hooks` | 扩展 | 查看钩子配置 |
| 46 | `/ide` | 扩展 | 管理 IDE 集成并显示状态 |
| 47 | `/mcp [reconnect (<server>\|all)\|enable\|disable [<server>\|all]]` | 扩展 | 管理 MCP 服务器连接和 OAuth 认证。不带参数打开交互列表；`reconnect <server>` 重连单个断开的服务器，`reconnect all` 重试所有失败或待认证的服务器；`enable`/`disable` 加服务器名或 `all` 直接改变连接状态而不打开对话框。非交互（`-p`）模式下不带参数运行时打印服务器状态的文本摘要 |
| 48 | `/plugin [subcommand]` | 扩展 | 管理 Claude Code 插件。不带参数打开插件菜单，或传 `list`、`install`、`enable`、`disable` 等子命令直接执行 |
| 49 | `/reload-plugins [--force]` | 扩展 | 重载所有活跃插件以应用未生效的改动，无需重启。报告各类组件的重载数量并标记加载错误。重载会改变已加载的 MCP 工具并使提示缓存失效时，命令会警告并跳过，除非传 `--force`。非交互模式（`-p`）、Agent SDK 和桌面应用也可用，桌面应用中仅对直接输入会话的命令生效且不应用插件 MCP 服务器变更；需要 v2.1.260+ |
| 50 | `/reload-skills` | 扩展 | 重新扫描 skill 和命令目录，使会话期间在磁盘上新增或修改的 skill 无需重启即可用。报告可用 skill 总数和增删数量 |
| 51 | `/skill-doctor` | 扩展 | 显示每个 skill 的上下文开销和使用频率，帮你找出可关闭的 skill。需要 v2.1.252+ 和 feature flag 获取 |
| 52 | `/skills` | 扩展 | 列出可用 skill。输入按名称、描述或来源过滤。按 `t` 按令牌数排序。按 `Space` 或 `Enter` 切换 skill 可见性；`Esc` 保存关闭。插件 skill、`disable-model-invocation: true` 的 skill 和被托管 `skillOverrides` 条目覆盖的 skill 无法切换 |
| 53 | `/memory` | 记忆 | 编辑 `CLAUDE.md` 文件、启用或禁用自动记忆、查看自动记忆条目 |
| 54 | `/advisor [model\|off]` | 模型 | 启用或禁用 advisor 工具，它会在任务的关键节点咨询第二个模型。接受模型名（`fable`、`opus`、`sonnet`）或完整模型 ID（`fable` 需要 Fable 5 权限）；不带参数打开选择器。非交互（`-p`）和 Remote Control 模式下不带参数时打印当前 advisor 而非打开选择器；需要 v2.1.260。`off` 关闭 |
| 55 | `/effort [level\|auto\|status\|ultracode [on\|off]]` | 模型 | 设置思考强度：`low` 到 `xhigh`、`max` 或 `auto`；`status` 打印当前值。`ultracode` 或 `ultracode on` 在当前强度下为本会话开启 ultracode，`ultracode off` 关闭；`ultracode` 键会持久保存。`max` 仅会话内有效。`on`/`off` 参数和保持当前强度需要 v2.1.284+。v2.1.284 之前 `/effort ultracode` 将会话设为 `xhigh`，`/effort ultracode off` 报 `Invalid argument` 错误。Claude 回复中运行时，确认缓存警告（如有）后新强度应用于本轮下一个请求。v2.1.242 之前由 feature flag 决定回复中执行还是排队到本轮结束，且不获取 feature flag 的会话总是排队。`-p` 模式可用 |
| 56 | `/fast [on\|off]` | 模型 | 切换快速模式。Claude 回复中运行时无需等待本轮结束即切换，但进行中的本轮按原速度完成。v2.1.242 之前由 feature flag 决定回复中执行还是排队，且不获取 feature flag 的会话总是排队 |
| 57 | `/model [model]` | 模型 | 切换模型并保存为新会话默认值。支持的模型可用左右方向键调整强度档位。不带参数打开选择器；在某行按 `s` 仅当前会话切换。确认后立即生效，不等当前回复结束。v2.1.242 之前由 feature flag 决定回复中执行还是排队，且不获取 feature flag 的会话总是排队。非交互（`-p`）模式带模型参数也可用，仅当前会话生效且不保存为默认；需要 v2.1.205+ |
| 58 | `/passes` | 模型 | 与朋友分享一周免费 Claude Code。仅符合条件的账户可见 |
| 59 | `/plan [description]` | 模型 | 从输入框直接进入 plan 模式。可传描述直接带着任务进入，如 `/plan fix the auth bug` |
| 60 | `/add-dir <path>` | 项目 | 为当前会话添加可访问文件的工作目录。输入部分路径显示匹配建议；按 `Tab` 接受。大部分 `.claude/` 配置不会从新增目录发现。多数网络路径（如 `\\server\share`）无法添加。添加成功后运行 `DirectoryAdded` 钩子。Claude 回复中运行时立即请求确认目录；确认后本轮 Claude 的下一个工具调用即可访问。v2.1.234 之前排队到本轮结束 |
| 61 | `/diff` | 项目 | 审查工作区的改动，包括 Claude 目前为止的编辑 |
| 62 | `/init` | 项目 | 初始化项目并生成 `CLAUDE.md` 指南。设置 `CLAUDE_CODE_NEW_INIT=1` 使用交互式流程，还会带你过一遍 skill、钩子和个人记忆文件。`/init` 发现 OpenAI Codex 或 Google Gemini CLI 配置时，会建议通过 `/import` 迁移 |
| 63 | `/review [low\|medium\|high\|xhigh\|max\|ultra] [--fix] [--comment] [--max-findings n\|all\|default] [pr#\|branch\|path]` | 项目 | `/code-review` 的别名：审查当前 diff，或你传入的 PR 号、分支、路径。接受相同强度档位和标志。未给强度时沿用你上次输入的 `low` 到 `max` 档位。深度云审查用 `/code-review ultra`。v2.1.223 之前 `/review` 是独立命令，按编号对 GitHub PR 做单遍只读审查；v2.1.186 到 v2.1.201 期间运行与 `/code-review medium` 相同的引擎 |
| 64 | `/security-review` | 项目 | 分析当前分支的待提交改动中的安全漏洞。审查你的分支与 origin 默认分支之间的 diff，识别注入、认证问题、数据暴露等风险。需要 `origin` 远程 |
| 65 | `/team-onboarding` | 项目 | 从你的 Claude Code 使用历史生成团队入职指南。Claude 分析近 30 天的会话、命令和 MCP 服务器使用，产出 markdown 指南。Team 和 Enterprise 套餐的 claude.ai 订阅者还会获得队友可直接在 Claude Code 中打开的分享链接 |
| 66 | `/ultrareview [PR or branch]` | 项目 | 在云沙箱中运行深度多 agent 代码审查。传 PR 引用审查该 PR，或传基准分支/提交改变对比基准。推荐写法是 `/code-review ultra`，`/ultrareview` 是别名。Pro 和 Max 每月 3 次免费，之后需要用量额度 |
| 67 | `/autofix-pr [prompt]` | 远程 | 生成一个云会话，盯着当前分支的 PR，在 CI 失败或评审人留评论时推送修复。通过 `gh pr view` 从当前分支检测打开的 PR；要盯别的 PR，先检出其分支。默认云会话被指示修复所有 CI 失败和评审评论；可传提示改变指令，如 `/autofix-pr only fix lint and type errors`。需要 `gh` CLI 和云会话权限 |
| 68 | `/desktop` | 远程 | 在 Claude Code 桌面应用中继续当前会话。需要 macOS 或 x64 Windows 和 Claude 订阅。别名：`/app` |
| 69 | `/install-github-app` | 远程 | 为仓库安装 Claude GitHub App，可选一并配置 GitHub Actions workflow 和密钥。仅支持 github.com 仓库。git 远程在 gitlab.com 或 bitbucket.org 时，命令打印提示后退出 |
| 70 | `/install-slack-app` | 远程 | 安装 Claude Slack 应用。打开浏览器完成 OAuth 流程 |
| 71 | `/mobile` | 远程 | 显示二维码下载 Claude 移动应用。别名：`/ios`、`/android` |
| 72 | `/remote-control` | 远程 | 让本会话可从 claude.ai 远程控制。未登录时运行会提示 Remote Control 需要 claude.ai 订阅并说明登录方式。别名：`/rc` |
| 73 | `/remote-env` | 远程 | 选择从 CLI 启动的云会话的默认云环境 |
| 74 | `/schedule [description]` | 远程 | 创建、更新、列出或运行云端定时任务（前身为 routines）。Claude 以对话方式带你完成设置。可询问某个定时任务的近期运行。别名：`/routines` |
| 75 | `/teleport` | 远程 | 把云会话拉到本终端。打开选择器，然后拉取分支和对话。也可用 `/tp`。需要 claude.ai 订阅 |
| 76 | `/web-setup` | 远程 | 用本地 `gh` CLI 凭据为云会话连接你的 GitHub 账户 |
| 77 | `/artifacts` | 会话 | 列出你拥有或分享给你的 artifact，然后将其附加到会话、在浏览器打开或复制链接。artifact 可用的地方即可用。需要 v2.1.208+；用 `Enter` 附加需要 v2.1.216+ |
| 78 | `/background [prompt]` | 会话 | 将当前会话转为后台 agent 运行并释放本终端。可传提示在脱离前再发一条指令。用 `claude agents` 监控。要把对话复制到新的后台会话并保留当前会话运行，用 `/fork`。别名：`/bg` |
| 79 | `/branch [name]` | 会话 | 在当前位置创建对话分支，尝试另一个方向而不丢失当前对话。切换进入分支并保留原对话，可用 `/resume` 返回。要以独立后台会话运行副本而不是切换进去，用 `/fork`；要把支线任务交给结果回报到本对话的 subagent，用 `/subtask` |
| 80 | `/btw [question]` | 会话 | 不增加对话内容地询问关于当前会话的支线问题。不带问题时显示最近的支线问题供回看；还没问过则打印用法 |
| 81 | `/cd <path>` | 会话 | 保持对话不变，将本会话移到新工作目录。输入部分路径显示匹配建议；按 `Tab` 接受。新目录会应用哪些配置、`/cd` 与 `/add-dir` 的区别见权限指南 |
| 82 | `/clear [name]` | 会话 | 以空上下文开始新对话。可传名字给上一段对话打标签，便于 `/resume` 选择器识别。想继续同一段对话但释放上下文，改用 `/compact`。用 `/resume` 恢复上一段对话，或从 rewind 菜单还原。别名：`/reset`、`/new` |
| 83 | `/compact [instructions]` | 会话 | 通过总结至今的对话来释放上下文。可传聚焦指令影响摘要内容 |
| 84 | `/exit` | 会话 | 退出 CLI。在已附加的后台会话中则是脱离，会话继续运行。别名：`/quit` |
| 85 | `/fork [prompt]` | 会话 | 把当前对话复制到新的后台会话，并在这里继续工作。传提示则副本立即开始执行；不传则在 agent 视图等待第一条提示。Claude Code 会指示副本改代码前创建自己的 worktree（需要 v2.1.221+）。要把支线任务交给结果回到本对话的 subagent，用 `/subtask`；要自己切换进副本，用 `/branch`。agent 视图关闭时，`/fork` 改为启动 fork 型 subagent |
| 86 | `/goal [condition\|clear]` | 会话 | 设定目标 — Claude 跨轮次持续工作直到条件满足。不带参数时显示当前或最近达成的目标。`clear`、`stop`、`off`、`reset`、`none` 或 `cancel` 提前移除生效中的目标 |
| 87 | `/list-agents` | 会话 | 列出 Claude 可以通信的 subagent、agent team 队友和其他 Claude Code 会话，以及各自的名称。也可用 `/peers`。需要 v2.1.224+；更早版本报 `Unknown command: /list-agents`。队友行和显示本会话名称的首行需要 v2.1.239+。仅在启用跨会话消息的地方可用 |
| 88 | `/recap` | 会话 | 按需生成当前会话的一行摘要 |
| 89 | `/rename [name]` | 会话 | 重命名当前会话并显示在输入框上。不带名字时从对话历史自动生成。非交互模式（`-p`）也可用。Claude Code 会把名字中的控制字符和不可见字符替换为空格，长度上限 200 字符（需要 v2.1.221+）。移除不可见字符后为空则拒绝：`That name is empty once invisible characters are removed. Usage: /rename <name>` |
| 90 | `/resume [session]` | 会话 | 按 ID 或名字恢复对话，或打开会话选择器。后台会话在选择器中带 `bg` 标记。恢复运行中的后台会话会把本终端附加到它，并把当前对话转入后台 — 在空输入框按 `←` 返回 agent 视图，那里也列出你离开的对话。v2.1.285 之前会拒绝并提示用 `claude attach` 打开或先停止。别名：`/continue` |
| 91 | `/rewind` | 会话 | 把对话和/或代码回退到之前的某个点，或从选中的消息开始总结。见 checkpointing。别名：`/checkpoint`、`/undo` |
| 92 | `/stop` | 会话 | 停止你附加的后台会话（或你以 peek 回复发送的目标）；对话记录和 worktree 保留。只脱离不停止用 `/exit` 或按 `←` |
| 93 | `/subtask <task>` | 会话 | 生成 fork 型 subagent：继承完整对话的后台 subagent，你继续干活的同时它处理任务，完成后结果回到本对话。要把对话复制到独立后台会话，用 `/fork`。agent 视图关闭时 `/subtask` 不可用，`/fork` 会代替它生成 fork 型 subagent |
| 94 | `/workflows` | 会话 | 打开 workflow 进度视图，查看、暂停、恢复或保存运行中和已完成的 workflow |

`/debug` 等内置 skill 也会出现在斜杠命令菜单中，但它们不是内置命令。

---

## 参考来源

- [Claude Code Commands 官方文档](https://code.claude.com/docs/en/commands)
- [Claude Code Interactive Mode 官方文档](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
