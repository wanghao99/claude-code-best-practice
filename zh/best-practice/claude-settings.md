# Settings（settings.json 配置）最佳实践

> 译自 [best-practice/claude-settings.md](../../best-practice/claude-settings.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

![Last Updated](https://img.shields.io/badge/Last_Updated-Oct%2013%2C%202026%2010%3A53%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.296-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../.claude/settings.json)

Claude Code `settings.json` 全部配置项的完整指南。截至 v2.1.296，Claude Code 暴露了 **244+ 个设置项**和 **396+ 个环境变量**（用 `settings.json` 的 `"env"` 字段设置环境变量，可以省去写包装脚本）。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 目录

1. [Settings 层级](#settings-层级)
2. [核心配置](#核心配置)
3. [权限](#权限)
4. [Hooks](#hooks)
5. [MCP 服务器](#mcp-服务器)
6. [沙箱](#沙箱)
7. [插件](#插件)
8. [模型配置](#模型配置)
9. [显示与 UX](#显示与-ux)
10. [AWS 与云凭据](#aws-与云凭据)
11. [环境变量（通过 env）](#环境变量通过-env)
12. [常用命令](#常用命令)

---

## Settings 层级

设置按优先级从高到低生效：

| 优先级 | 位置 | 作用范围 | 是否共享 | 用途 |
|----------|----------|-------|---------|---------|
| 1 | 托管设置（Managed settings） | 组织 | 是（由 IT 部署） | 不可被覆盖的安全策略 |
| 2 | 命令行参数 | 单次会话 | 不适用 | 临时单会话覆盖 |
| 3 | `.claude/settings.local.json` | 项目 | 否（git 忽略） | 个人项目专属 |
| 4 | `.claude/settings.json` | 项目 | 是（提交到仓库） | 团队共享设置 |
| 5 | `~/.claude/settings.json` | 用户 | 不适用 | 全局个人默认值 |

**托管设置（Managed settings）**由组织强制执行，任何其他层级（包括命令行参数）都无法覆盖。分发方式：
- **服务器托管**设置（远程下发）
- **MDM 描述文件** — macOS plist，位于 `com.anthropic.claudecode`
- **注册表策略** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode`（管理员级）和 `HKCU\SOFTWARE\Policies\ClaudeCode`（用户级，策略优先级最低）
- **文件** — `managed-settings.json` 和 `managed-mcp.json`（macOS：`/Library/Application Support/ClaudeCode/`，Linux/WSL：`/etc/claude-code/`，Windows：`C:\Program Files\ClaudeCode\`）
- **Drop-in 目录** — 与 `managed-settings.json` 同级的 `managed-settings.d/`，用于放独立的策略片段（v2.1.83）。遵循 systemd 惯例：先以 `managed-settings.json` 作为基础合并，再把 `managed-settings.d/` 里所有 `*.json` 按字母序逐个合并到上层。标量值后写的覆盖先写的；数组做拼接去重；对象做深合并。以 `.` 开头的隐藏文件会被忽略。用数字前缀控制合并顺序（如 `10-telemetry.json`、`20-security.json`）

在托管层内部，优先级为：服务器托管 > MDM/操作系统级策略 > 基于文件的（`managed-settings.d/*.json` + `managed-settings.json`）> HKCU 注册表（仅 Windows）。对大多数键来说只有一个托管来源生效；各层来源之间不做合并。基于文件的这一层内部，drop-in 文件和基础文件会合并到一起。

**例外 —— 管理员来源联合（union）键**：以下键只要*任意*管理员控制的托管来源设置了就会生效，而不只看优先最高的那个来源：`sandbox.network.allowManagedDomainsOnly`、`sandbox.network.allowedDomains`（当 `allowManagedDomainsOnly` 已设置时）、`sandbox.filesystem.allowManagedReadPathsOnly`、`sandbox.filesystem.allowRead`（当 `allowManagedReadPathsOnly` 已设置时）、`allowAllClaudeAiMcps`、`sandbox.bwrapPath`、`sandbox.socatPath`、`forceRemoteSettingsRefresh`，以及 `env`（它在所有管理员来源之间**按键合并**，而不是整体取优先最高来源的对象）。

> **注意**：自 v2.1.75 起，已废弃的 Windows 回退路径 `C:\ProgramData\ClaudeCode\managed-settings.json` 已被移除。请改用 `C:\Program Files\ClaudeCode\managed-settings.json`。

> **注意（v2.1.126）**：`/config` 现在把修改持久化写入 `~/.claude/settings.json`，而不是只保存在内存里。通过交互式 Config UI 做的编辑在重启后仍然保留。

**仅限托管层的策略键**：

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `parentSettingsBehavior` | string | `"first-wins"` | 控制当存在管理员部署的托管层时，由嵌入宿主进程（SDK 父进程）以编程方式提供的托管设置是否生效。`"first-wins"`：父进程提供的设置被丢弃，只有管理员层生效。`"merge"`：父进程提供的设置在管理员层之下生效，并会被过滤——只能收紧策略、不能放宽。需要 v2.1.133+ |
| `policyHelper` | object | - | 管理员部署的可执行文件，在启动时动态计算托管设置。对象结构：`{path: string, timeoutMs?: number, refreshIntervalMs?: number}` — `path` 指向 helper 二进制；`timeoutMs` 限制等待时间（省略或 `0` 表示不限时）；`refreshIntervalMs` 控制在会话开始时重新运行（省略表示只在启动时运行一次，`0` 表示禁用，否则必须 ≥ `60000`）。只从 MDM 或系统级 `managed-settings.json` 文件读取（绝不从用户/项目设置读取）。**配置后，`policyHelper` 的输出是唯一生效的托管来源——该次运行中其他托管层来源（服务器托管、MDM、文件）全部被忽略。**需要 v2.1.136+ |
| `requiredMinimumVersion` | string | - | **（仅托管层）** 低于此版本下限时 Claude Code 拒绝启动。CLI 报错退出并提示用户升级。与 `minimumVersion`（控制自动更新的版本下限）互补——这个键是在启动时强制执行。示例：`"2.1.163"` |
| `requiredMaximumVersion` | string | - | **（仅托管层）** 超过此版本上限时 Claude Code 拒绝启动。版本太新时 CLI 报错退出。配合 `requiredMinimumVersion` 可在托管环境中钉住特定版本区间。示例：`"2.1.165"` |
| `browserExternalPageTools` | string | - | **（仅托管层）** 设为 `"disabled"` 阻止 Claude 在桌面应用 Browser 面板中使用读取或操作外部页面的工具。用户仍可自行导航到外部站点，本地开发服务器预览不受影响 |
| `disableBrowserExternalNavigation` | boolean | - | **（仅托管层）** 设为 `true`（只认 JSON 布尔值——字符串 `"true"` 会被静默忽略）阻止 Claude 把桌面应用 Browser 面板导航到外部 URL。只有 JSON 布尔 `true` 生效 |
| `disableMobileSimulatorTools` | boolean | - | **（仅托管层）** 设为 `true`（只认 JSON 布尔值——格式错误时会记录警告）移除 Claude 的移动模拟器工具。只有 JSON 布尔 `true` 生效 |

**重要**：
- `deny` 规则安全优先级最高，不能被低优先级的 allow/ask 规则覆盖。
- 即使本地文件写了不同的值，托管设置仍可能锁定或覆盖本地行为。
- 数组型设置（如 `permissions.allow`）在各作用域之间**拼接并去重**——所有层级的条目会合并而不是替换。**例外**：`fallbackModel`、`availableModels`、`modelPicker`、`modelSettings` **不**合并——由定义了它们的最高优先级设置文件整体提供。对 `availableModels` 而言，托管来源的值原样生效，低层级作用域无法扩充。

---

## 核心配置

### 常规设置

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `$schema` | string | - | JSON Schema URL，用于 IDE 校验和自动补全（如 `"https://json.schemastore.org/claude-code-settings.json"`） |
| `model` | string | `"default"` | 覆盖默认模型。接受别名（`sonnet`、`opus`、`haiku`）或完整模型 ID |
| `agent` | string | - | 为主会话设置默认 agent。取值是 `.claude/agents/` 中的 agent 名称。也可通过 `--agent` CLI 参数设置 |
| `language` | string | `"english"` | Claude 的首选回复语言。同时决定语音听写语言和**自动生成的会话标题**语言（v2.1.121；自 v2.1.176 起，未设置语言时标题跟随对话语言） |
| `claudeMdExcludes` | array | - | Glob 模式或绝对路径，指定加载[记忆](https://code.claude.com/docs/en/memory)时要跳过的 `CLAUDE.md` 文件。模式按绝对文件路径匹配。只作用于用户、项目和本地记忆；托管策略文件不能被排除。示例：`["**/vendor/**/CLAUDE.md"]` |
| `claudeMd` | string | - | **（仅托管层）** 以组织托管[记忆](https://code.claude.com/docs/en/memory)形式注入的 CLAUDE.md 风格指令。只在托管或策略设置中生效；在用户、项目和本地设置中被忽略。示例：`"Always run make lint before committing."` |
| `cleanupPeriodDays` | number | `30` | 启动清理扫掠的保留天数下限（最小 1）。不活跃会话记录和孤立的 subagent worktree 会被删除；自 v2.1.117 起扫掠还覆盖 `~/.claude/tasks/`、`~/.claude/shell-snapshots/`、`~/.claude/backups/`。设为 `0` 会校验报错被拒绝。要在非交互模式（`-p`）下禁用会话记录写入，用 `--no-session-persistence` 或 SDK 选项 `persistSession: false` |
| `autoUpdatesChannel` | string | `"latest"` | 发布渠道：`"stable"` 或 `"latest"` |
| `minimumVersion` | string | - | 阻止自动更新器降级到指定版本以下。切换到 stable 渠道并选择留在当前版本直到 stable 追上时会自动设置。与 `autoUpdatesChannel` 配合使用 |
| `alwaysThinkingEnabled` | boolean | - | 扩展思考**默认开启**。设为 `false` 可对所有会话禁用。设为 `true` 是空操作。**注意**：对 Claude Opus 5.5、Sonnet 5.5、Haiku 5.5、Fable 无效——这些模型无法禁用思考 |
| `thinkingBudgetTokens` | number | - | 为每次回复的扩展思考设置固定 token 预算。设置后，思考 token 数被限制为该值。未设置时，Claude Code 根据模型和 effort 等级使用动态预算。与 `alwaysThinkingEnabled` 配合使用 *（不在官方 settings 页面——未验证）* |
| `skipWebFetchPreflight` | boolean | `false` | 跳过 WebFetch 的域名安全检查（该检查会在抓取前把请求的主机名发送到 `api.anthropic.com`）。在阻止了到 Anthropic 出站流量的环境（如出口受限的 Bedrock、Vertex AI、Foundry 部署）中设为 `true` |
| `availableModels` | array | - | 限制用户可通过 `/model`、`--model`、Config 工具或 `ANTHROPIC_MODEL` 选择的模型。不影响 Default 选项。自 v2.1.172 起，也约束 subagent 派发的模型选择器和 `advisorModel` 选择器。配合 `enforceAvailableModels: true` 可进一步约束 Default 模型选项。示例：`["sonnet", "haiku"]` |
| `enforceAvailableModels` | boolean | `false` | 为 `true` 时，`availableModels` 白名单也约束 Default 模型选项——用户即使通过 Default 槽位也无法选择白名单之外的模型。不设此开关时，`availableModels` 不限制 Default 选项。与 `availableModels` 配合实现完整模型锁定（v2.1.175） |
| `fastModePerSessionOptIn` | boolean | `false` | 要求用户每个会话单独选择加入 fast mode |
| `fastMode` | boolean | `false` | 为所有会话启用 fast mode。为 `true` 时，Claude Code 默认使用更快的模型档位。用户也可以用 `/fast` 按会话切换。与 `fastModePerSessionOptIn` 配对 |
| `defaultShell` | string | `"bash"` | 输入框 `!` 命令的默认 shell。接受 `"bash"`（默认）或 `"powershell"`。设 `"powershell"` 后 Windows 上交互式 `!` 命令走 PowerShell。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`（v2.1.84）。**v2.1.120**：当 PowerShell 可用时，即使没装 Git for Windows，它也会作为 Windows 的回退 shell。**v2.1.126**：启用 PowerShell 时，它被当作主 shell 而不是默认 Bash。PowerShell 7 检测现在还覆盖 Microsoft Store 安装、不在 PATH 上的 MSI 安装和 `.NET` 全局工具安装 |
| `includeGitInstructions` | boolean | `true` | 在 Claude 的系统提示中包含内置的提交与 PR 工作流指令以及 git 状态快照。设置了 `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` 环境变量时，该变量优先于本设置 |
| `voice` | object | - | 语音听写配置。对象含三个字段：`enabled`（boolean——按键说话开关）、`mode`（string——`"hold"` 按住说话或 `"tap"` 点按切换）、`autoSubmit`（boolean——听写结束后立即提交转写文本）。运行 `/voice` 时自动写入。需要 Claude.ai 账号（v2.1.118 扩展了结构） |
| `voiceEnabled` | boolean | - | **已废弃** — `voice.enabled` 的旧别名。请改用 `voice` 对象以获得 `mode` 和 `autoSubmit` 控制 |
| `showClearContextOnPlanAccept` | boolean | `false` | 在计划接受界面显示“清除上下文”选项。设为 `true` 恢复该选项（自 v2.1.81 起默认隐藏） |
| `viewMode` | string | - | 启动时的默认记录视图模式：`"default"`、`"verbose"` 或 `"focus"`。设置后覆盖 `/focus` 粘性切换的选择 |
| `disableDeepLinkRegistration` | string | - | 设为 `"disable"` 阻止 Claude Code 启动时向操作系统注册 `claude-cli://` 协议处理器。Deep link 允许外部工具通过 `claude-cli://open?q=...` 打开带预填提示的 Claude Code 会话。`q` 参数支持用 URL 编码换行符（`%0A`）的多行提示。适用于协议处理器注册受限或被单独管理的环境 |
| `showThinkingSummaries` | boolean | `false` | 在交互式会话中显示扩展思考摘要。未设置或为 `false` 时（交互模式默认），思考块被 API 脱敏、显示为折叠占位。脱敏只改变你看到的内容，不改变模型生成的内容——要降低思考开销，请调低预算或禁用思考。非交互模式（`-p`）和 SDK 调用方无论本设置如何都会收到摘要 |
| `disableSkillShellExecution` | boolean | `false` | 禁止来自用户、项目、插件或附加目录来源的 skill 与自定义命令中 `` !`...` `` 和 `` ```! `` 块的内联 shell 执行。命令会被替换为 `[shell command execution disabled by policy]` 而不执行。内置和托管 skill 不受影响（v2.1.91） |
| `skillListingMaxDescChars` | number | `1536` | Claude 每轮看到的 [skill 列表](https://code.claude.com/docs/en/skills)中，单个 skill 的 `description` 与 `when_to_use` 合计文本的字符上限。超出会被截断（v2.1.105） |
| `skillListingBudgetFraction` | number | `0.01` | 预留给 Claude 每轮看到的 [skill 列表](https://code.claude.com/docs/en/skills)的上下文窗口比例（`0.01` = 1%）。列表超出预算时，最少使用的 skill 描述会折叠成仅名称——Claude 仍能调用但看不到用途（v2.1.105） |
| `forceRemoteSettingsRefresh` | boolean | `false` | **（仅托管层）** 阻塞 CLI 启动直到远程托管设置刷新拉取完成。拉取失败则 CLI 退出（fail-closed）。用于策略必须在任何会话开始前保持最新的企业环境（v2.1.92） |
| `wslInheritsWindowsSettings` | boolean | `false` | **（仅 Windows 托管设置）** 为 `true` 时，WSL 上的 Claude Code 除 `/etc/claude-code` 外还读取 Windows 策略链（HKLM 注册表 + `C:\Program Files\ClaudeCode\managed-settings.json`），且 Windows 来源优先。只在 HKLM 注册表键或 `C:\Program Files\ClaudeCode\managed-settings.json` 中设置时生效（这两处都需要 Windows 管理员权限写入）。若要让 HKCU 策略也应用到 WSL，该开关必须额外设置在 HKCU 自身中。对原生 Windows 无效（v2.1.118） |
| `tui` | string | `"default"` | 渲染模式：`"fullscreen"` 或 `"default"`。通过 `/tui fullscreen` 设置为无闪烁的备用屏幕渲染（v2.1.110） |
| `awaySummaryEnabled` | boolean | `true` | 用户离开后返回时生成“离开摘要”（空闲会话回顾）。设为 `false` 退出。与 `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` 环境变量配对（v2.1.110） |
| `autoCompactEnabled` | boolean | `true` | 上下文接近模型上限时自动压缩对话。设为 `false` 禁用自动压缩、手动管理上下文。也可用 `DISABLE_AUTO_COMPACT` 环境变量禁用 |
| `autoCompactWindow` | number | 按模型调优 | 触发自动压缩前上下文窗口可填充到的程度，以 token 计（100,000–1,000,000）。未设置时，Claude Code 使用为你的模型调优的窗口。用 `/autocompact` 命令（写入用户设置）或 `--autocompact` CLI 参数设置。与 `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 环境变量配对（v2.1.221+） |
| `skillOverrides` | object | - | 按 skill 名称设置的逐个可见性覆盖。取值为 `"on"`（完整）、`"name-only"`（可见但不自动展示描述）、`"user-invocable-only"`（对模型发现隐藏但仍可斜杠调用）或 `"off"`（完全隐藏）。示例：`{"legacy-context": "name-only", "deploy": "off"}`（v2.1.129） |
| `disableRemoteControl` | boolean | `false` | 禁用 [Remote Control](https://code.claude.com/docs/en/remote-control)：屏蔽 `claude remote-control`、`--remote-control` 参数、自动启动和会话内开关。通常放在托管设置中做按设备 MDM 强制，但任意作用域都有效（v2.1.128） |
| `agentPushNotifEnabled` | boolean | `false` | 当 Claude 决定推送时（如任务完成），向 [Remote Control](https://code.claude.com/docs/en/remote-control) 发送主动推送通知。在 `/config` 中显示为 **Push when Claude decides** |
| `inputNeededNotifEnabled` | boolean | `false` | 当权限确认或问题等待用户输入时，向 [Remote Control](https://code.claude.com/docs/en/remote-control) 发送推送通知。在 `/config` 中显示为 **Push when actions required** |
| `remoteControlAtStartup` | boolean/null | - | 启动时自动连接 [Remote Control](https://code.claude.com/docs/en/remote-control)。`true` 总是自动连接，`false` 从不自动连接，未设置则用组织默认。**作用域例外**：项目或本地设置中的 `false` 即使面对托管 `true` 也生效（用户和项目设置可以选择退出）。只有用户设置、`--settings` 和托管设置能设为 `true`（v2.1.119+） |
| `disableAgentView` | boolean | `false` | 设为 `true` 关闭[后台 agent 与 agent 视图](https://code.claude.com/docs/en/agent-view)：`claude agents`、`--bg`、`/background` 和按需 supervisor。任意作用域可设，但通常放在托管设置。等价于把 `CLAUDE_CODE_DISABLE_AGENT_VIEW` 环境变量设为 `1` |
| `disableWorkflows` | boolean | `false` | 设为 `true` 禁用[动态 workflow](https://code.claude.com/docs/en/workflows)（`/workflows`）和内置 workflow 斜杠命令。任意作用域可设。等价于 `CLAUDE_CODE_DISABLE_WORKFLOWS` 环境变量。Workflow 于 v2.1.154 引入 |
| `workflowKeywordTriggerEnabled` | boolean | `true` | 在提示中输入 "ultracode" 一词是否触发[动态 workflow](https://code.claude.com/docs/en/workflows)。设为 `false` 则要求显式 `/workflows` 调用。Ultracode、`/workflows` 和已保存的 workflow 命令不受影响。在 `/config` 中显示为 **Ultracode keyword trigger**（v2.1.157；触发关键词在 v2.1.160 从 workflow 改名为 ultracode） |
| `ultracode` | boolean | - | 为 `true` 时，harness 默认为每个实质性任务编写并运行 workflow，不计 token 成本地最大化彻底程度。持久化布尔值——可写入任意设置文件（默认未设置）。也可用 `/effort ultracode` 或 `--settings` 按会话激活（v2.1.154） |
| `dynamicWorkflowSize` | string | - | [动态 workflow](https://code.claude.com/docs/en/workflows) 中生成的 agent 数量的建议值。取值：`"small"`、`"medium"`、`"large"`。设置后，workflow harness 在按任务伸缩前以此作为默认机群规模。通过 `/config` 设置为 **Workflow size**（v2.1.202；取值在 v2.1.205 正式化）*（不在官方文档——未验证；v2.1.219 已被 `workflowSizeGuideline` 取代）* |
| `workflowSizeGuideline` | string | `"medium"` | 动态 workflow 机群规模的建议值，可从任意设置文件设置。取值：`"small"`、`"medium"`（v2.1.219 起的默认）、`"large"`、`"unrestricted"`。默认机群规模现在会显示在运行中 workflow 的状态行里。请用这个键代替 `dynamicWorkflowSize`——它能从托管或用户设置传播（v2.1.219） |
| `disableBundledSkills` | boolean | `false` | 对模型隐藏 Claude Code 的内置能力（bundled skills）。为 `true` 时模型无法调用内置 skill。与 `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` 环境变量配对。适用于要求严格仅插件自定义的场景（v2.1.169） |
| `disableArtifact` | boolean | `false` | 禁用 Artifact 网页发布工具。为 `true` 时 Claude 无法创建或发布 web artifact。任意作用域可设 |
| `enableArtifact` | boolean | - | **（v2.1.196+）** 只能用于**禁用** Artifact 网页发布工具。设为 `false` 时从任意作用域禁用——一旦禁用没有任何东西能重新开启。设为 `true` 等价于未设置、本身无作用。`disableArtifact: true` 有同样的禁用效果 |
| `feedbackSurveyRate` | number | - | 符合条件的会话弹出质量调查的概率（0–1）。企业管理员可控制调查出现频率。示例：`0.05` = 5% 的符合条件的会话 |
| `advisorModel` | string | - | 服务端 advisor 工具使用的模型。接受模型别名（`opus`、`sonnet`）或完整模型 ID。未设置时 advisor 使用会话模型。需要 v2.1.98+。**v2.1.210+**：设 `"fable"` 不再附加 advisor——Fable 5 暂时无法用于 advisor 选择器；请改用 `"opus"` 或 `"sonnet"` |
| `respondToBashCommands` | boolean | `true` | `!` shell 命令完成后 Claude 是否自动接着响应。设为 `false` 禁用 `!` bash 命令结束后的自动跟进响应（v2.1.186） |
| `askUserQuestionTimeout` | string | `"never"` | 未应答的 AskUserQuestion 对话框过多久后自动继续（不等用户）。取值：`"60s"`、`"5m"`、`"10m"`、`"never"`（不自动继续——默认）。通过 `/config` 设置为 **Question auto-continue timeout**。与 `CLAUDE_AFK_TIMEOUT_MS` 环境变量配对；该环境变量只在本设置设为时长时才生效。只从用户和托管设置生效——项目与本地设置的值被忽略（v2.1.200） |
| `theme` | string | `"dark"` | UI 颜色主题。取值：`"auto"`（跟随系统）、`"dark"`、`"light"`、`"dark-daltonized"`、`"light-daltonized"`、`"dark-ansi"`、`"light-ansi"`、`"custom:<slug>"`、`"custom:<plugin>:<slug>"`。插件提供的主题用 `custom:` 前缀 |
| `verbose` | boolean | `false` | 显示完整工具输出而不是截断摘要。等价于以 `--verbose` 运行；跨会话保持详细视图 |
| `switchModelsOnFlag` | boolean | `true` | 请求被安全分类器标记时自动切换到回退模型。为 `false` 时被标记的请求直接阻止而不是改道（v2.1.170） |
| `processWrapper` | string | - | 用于后台进程的企业启动器命令（如注入凭据的包装器）。只从托管设置、用户设置（`~/.claude/settings.json`）或 `--settings` 生效；从项目和本地设置忽略，防止不可信仓库劫持后台进程启动（v2.1.210） |
| `remote.defaultEnvironmentId` | string | - | 通过 `--remote` 启动远程会话或后台 agent 但未显式给出环境 ID 时使用的默认环境 ID。设置后 Claude Code 自动选择该环境而不是提示。只从用户和托管设置生效（v2.1.200+） |
| `crossSessionInbound` | string | - | 控制本会话如何处理来自你其他 Claude 会话的消息（经 Remote Control）。取值：`"accept"`（接收其他会话的消息）、`"hold"`（排队入站消息但不处理）、`"refuse"`（拒绝会话间流量）。作用域例外：项目或本地设置中更严格的值即使面对托管设置也生效（`accept` < `hold` < `refuse`）。在 `/config` 中显示为 **Messages from your other sessions**（v2.1.224） |
| `dialogExpiry` | string | `"5m"` | 来自其他会话的入站对话框请求排队多久后自动过期。接受 `"60s"`、`"5m"`、`"10m"` 或 `"never"`（不过期）。作用域：用户或托管。`crossSessionInbound: "hold"` 的会话恢复时，超过此时长的排队请求被丢弃。环境变量覆盖：`CLAUDE_CODE_USER_DIALOG_TIMEOUT_MS`。覆盖远程客户端对话框、挂起的跨会话消息和 Fable 同意提示。在 `/config` 中显示为 **Dialog expiry**（v2.1.246） |
| `autoContinueAtUsageLimit` | boolean | - | 为 `true` 时，用量限制（API 速率限制、每日上限）重置后 Claude Code 自动继续会话，而不是等待用户输入。在 `/config` 中显示为 **Auto-continue at usage limit**（v2.1.234） |
| `feedbackDrafts` | string | `"notify"` | 控制后台 bug 报告草稿排队。取值：`"notify"`（默认——排队草稿并弹出通知）、`"quiet"`（静默排队不通知）、`"off"`（完全禁用）。作用域：用户或托管。控制 `SendFeedback` 工具的开关。环境变量禁用：`CLAUDE_CODE_SEND_FEEDBACK=0`。在 `/config` 中显示为 **Feedback drafts**（v2.1.247） |
| `desktopSessionCleanupPeriodDays` | number | - | 桌面会话数据的清理天数下限（独立于 `cleanupPeriodDays` 控制的记录清理）。超过该时限的不活跃桌面会话 artifact 会在启动清理扫掠中被删除（v2.1.248） |
| `bashOutputMaxChars` | number | - | 覆盖 Claude 从 Bash 或 PowerShell 工具响应中捕获的最大字符数。设置后优先于 `BASH_MAX_OUTPUT_LENGTH` 环境变量。用于调高输出很多命令的捕获上限，或调紧以控制上下文用量。需要 v2.1.261+ |
| `syncClaudeAiSkills` | boolean | - | 把 claude.ai 的 skill 同步到本 Claude Code 会话。作用域：用户、本地、托管或 `--settings`。需要 v2.1.275+ |
| `syncClaudeAiPlugins` | boolean | - | 把 claude.ai 的插件同步到本 Claude Code 会话。作用域：用户、本地、托管或 `--settings`。需要 v2.1.275+ |
| `promptSuggestionEnabled` | boolean | `true` | 在输入框中根据上下文显示提示建议。设为 `false` 禁用提示自动补全建议 |

**示例：**
```json
{
  "model": "opus",
  "agent": "code-reviewer",
  "language": "japanese",
  "cleanupPeriodDays": 60,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true
}
```

### 计划与记忆目录

把计划和自动记忆文件存到自定义位置。

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | `/plan` 输出存放目录 |
| `autoMemoryDirectory` | string | - | 自动记忆存储的自定义目录。接受 `~/` 展开路径。可写入任意设置文件；项目和本地值只在获得工作区信任后生效 |
| `autoMemoryEnabled` | boolean | `true` | 启用[自动记忆](https://code.claude.com/docs/en/memory)。为 `false` 时 Claude 不读写自动记忆目录。也可在会话中用 `/memory` 切换，或用 `CLAUDE_CODE_DISABLE_AUTO_MEMORY` 环境变量禁用 |
| `fileCheckpointingEnabled` | boolean | `true` | 每次编辑前对文件做快照，便于用 `/rewind` 恢复。设为 `false` 跳过检查点、节省磁盘。也可用 `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` 环境变量禁用 |

**示例：**
```json
{
  "plansDirectory": "./my-plans"
}
```

**使用场景**：适合把计划产物与 Claude 内部文件分开管理，或把计划放到团队共享位置。

### Worktree 设置

配置 `--worktree` 如何创建和管理 git worktree。适合在大型 monorepo 中减少磁盘占用和启动时间。

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | 从主仓库符号链接到每个 worktree 的目录，避免大目录在磁盘上重复 |
| `worktree.sparsePaths` | array | `[]` | 通过 git sparse-checkout（cone 模式）在每个 worktree 中检出的目录。只有列出的路径会写入磁盘 |
| `worktree.baseRef` | string | `"fresh"` | 新 worktree 从哪个 ref 拉分支。`"fresh"` 从 `origin/<默认分支>` 拉出与远端一致的干净树；`"head"` 从当前本地 `HEAD` 拉，包括已跟踪未提交的改动（v2.1.133） |
| `worktree.bgIsolation` | string | `"worktree"` | [后台会话](https://code.claude.com/docs/en/agent-view)的隔离模式。`"worktree"`（默认）在调用 `EnterWorktree` 前阻止对主检出区的 `Edit`/`Write`；`"none"` 允许后台任务直接编辑工作副本（v2.1.143） |

**示例：**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### 署名设置

自定义 git 提交和 pull request 的署名消息。

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `attribution.commit` | string | Co-authored-by | Git 提交署名（支持 trailer） |
| `attribution.pr` | string | 生成消息 | PR 描述署名 |
| `attribution.sessionUrl` | boolean | `true` | 在 web 会话或 Remote Control 会话生成的提交和 PR 中包含指向 claude.ai 会话 URL 的链接。设为 `false` 省略链接。对本地 CLI 会话无效（v2.1.183） |
| `prUrlTemplate` | string | - | URL 模板，控制提交署名中的 "PR" 徽章如何链接到 PR 界面。支持仓库主机、owner、仓库和 PR 号占位符。适用于默认 `https://github.com/...` URL 不适用的自建 GitLab/Bitbucket/GitHub Enterprise（v2.1.119） |
| `includeCoAuthoredBy` | boolean | `true` | **已废弃** - 请改用 `attribution` |

**示例：**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**注意**：设为空字符串（`""`）可完全隐藏署名。

### 认证辅助

用于动态生成认证 token 的脚本。

| 键 | 类型 | 说明 |
|-----|------|-------------|
| `apiKeyHelper` | string | 输出认证 token 的 shell 脚本路径（同时作为 `X-Api-Key` 和 `Authorization: Bearer` 头发送） |
| `forceLoginMethod` | string | 限制登录方式为 `"claudeai"`、`"console"` 或 `"gateway"` 账号。组织管理的 Claude gateway 部署用 `"gateway"`。**（仅托管层）** |
| `forceLoginOrgUUID` | string \| array | 要求登录账号属于指定组织。接受单个 UUID 字符串（登录时会预选该组织）或 UUID 数组（接受任一列出组织、不预选）。设在托管设置时，认证账号不属于列出组织则登录失败；空数组 fail-closed，报配置错误并阻止登录 |
| `forceLoginGatewayUrl` | string | **（仅托管层）** `forceLoginMethod` 为 `"gateway"` 时，在登录界面预填 Claude gateway URL。免去用户手动输入。通常与 `forceLoginMethod: "gateway"` 一起设置 |
| `gcpAuthRefresh` | string | GCP 应用默认凭据（ADC）过期或加载失败时刷新它的自定义脚本。由 Claude Code 在重试认证前运行。适用于 ADC 短时效、需要组织专属 helper 续期的场景。示例：`"gcloud auth application-default login"` |

**示例：**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### 公司公告

在启动时向用户展示自定义公告（随机轮换）。

| 键 | 类型 | 说明 |
|-----|------|-------------|
| `companyAnnouncements` | array | 启动时显示的字符串数组 |

**示例：**
```json
{
  "companyAnnouncements": [
    "Welcome to Acme Corp!",
    "Remember to run tests before committing!",
    "Check the wiki for coding standards"
  ]
}
```

---

## 权限

控制 Claude 可以执行哪些工具和操作。

### 权限结构

```json
{
  "permissions": {
    "allow": [],
    "ask": [],
    "deny": [],
    "additionalDirectories": [],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### 权限键

| 键 | 类型 | 说明 |
|-----|------|-------------|
| `permissions.allow` | array | 允许使用工具而不弹确认的规则 |
| `permissions.ask` | array | 需要用户确认的规则 |
| `permissions.deny` | array | 阻止使用工具的规则（优先级最高） |
| `permissions.additionalDirectories` | array | Claude 可访问的额外目录 |
| `permissions.defaultMode` | string | 默认权限模式。有效值：`"default"`、`"manual"`（`"default"` 的别名，v2.1.200）、`"acceptEdits"`、`"dontAsk"`、`"bypassPermissions"`、`"auto"`、`"plan"`。Remote 环境只认 `acceptEdits` 和 `plan`（v2.1.70+）。**注意（v2.1.142）**：在项目（`.claude/settings.json`）或本地设置中 `"auto"` 被忽略——仓库不能给自己授予 auto 模式；请用 `~/.claude/settings.json` |
| `permissions.disableBypassPermissionsMode` | string | 阻止激活 bypass 模式 |
| `permissions.skipDangerousModePermissionPrompt` | boolean | 跳过通过 `--dangerously-skip-permissions` 或 `defaultMode: "bypassPermissions"` 进入 bypass 权限模式前的确认提示。在项目设置（`.claude/settings.json`）中被忽略，防止不可信仓库自动绕过提示 |
| `allowManagedPermissionRulesOnly` | boolean | **（仅托管层）** 只有托管权限规则生效；用户/项目的 `allow`、`ask`、`deny` 规则被忽略 |
| `permissions.blockReadsOutsideWorkingDirectories` | boolean | **（作用域例外：任意作用域）** 为 `true` 时，Claude Code 阻止所有针对当前工作目录（工作目录 + `additionalDirectories`）之外文件的 Read 操作。任意作用域的 `true`——包括项目和本地设置——即使面对托管 `false` 也生效。适合在共享或容器化环境中把读取限制在项目树内（v2.1.257+） |
| `autoMode` | object | 自定义 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 分类器阻止和放行的内容。含 `environment`（受信基础设施描述）、`allow`（阻止规则的例外）、`soft_deny`（阻止规则）、`hard_deny`（无条件阻止规则——不能被 `allow` 例外或 `$defaults` 哨兵覆盖，v2.1.136）——全部是自然语言字符串数组。还接受 `classifyAllShell`（boolean，默认 `false`）：为 `true` 时所有 Bash/PowerShell 命令都经过 auto-mode 分类器，而不只是任意代码执行模式，覆盖更严但分类器调用更多（v2.1.193）。**不从共享项目设置**（`.claude/settings.json`）**或本地设置**（`.claude/settings.local.json`）**读取**，防止仓库注入（v2.1.207 移除了本地设置作用域）。只在用户和托管设置中可用；请在 `~/.claude/settings.json` 中配置。设置 `allow` 或 `soft_deny` 会**整体替换**该部分的默认列表，除非在数组里包含字面量 `"$defaults"`——该哨兵在对应位置继承内置规则，自定义条目追加在其前后（v2.1.118）。自定义前先运行 `claude auto-mode defaults` 查看内置规则 |
| `disableAutoMode` | string | 设为 `"disable"` 阻止激活 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)。从 `Shift+Tab` 循环中移除 `auto`，并在启动时拒绝 `--permission-mode auto`。任意设置层级可设；放在托管设置中最有用，用户无法覆盖 |
| `useAutoModeDuringPlan` | boolean | auto mode 可用时，plan mode 是否采用 auto mode 语义。默认：`true`。不从共享项目设置（`.claude/settings.json`）读取。在 `/config` 中显示为 "Use auto mode during plan" |

### 权限模式

| 模式 | 行为 |
|------|----------|
| `"default"` | 标准权限检查，弹确认 |
| `"manual"` | v2.1.200 引入的 `"default"` 别名——同样的标准权限检查加确认。改名是为了在 CLI、VS Code 和 JetBrains 界面统一表述。凡接受 `"default"` 之处均接受 |
| `"acceptEdits"` | 自动接受文件编辑**和常见文件系统命令**（`mkdir`、`touch`、`mv`、`cp` 等），限工作目录或 `additionalDirectories` 内的路径。**v2.1.160**：写入授予代码执行的构建工具配置文件（`.npmrc`、`.yarnrc*`、`bunfig.toml`、`.bazelrc`、`.pre-commit-config.yaml`、`.devcontainer/` 等）以及写入 shell 启动文件（`.zshenv`、`.zlogin`、`.bash_login`）和 `~/.config/git/` 前总是弹确认 |
| `"dontAsk"` | 除通过 `/permissions` 或 `permissions.allow` 规则预先批准的工具外，其余自动拒绝。MCP 服务器清单中标记 `requiresUserInteraction` 的工具和组织设为 ask 的连接器工具即使你显式允许也**被拒绝** |
| `"bypassPermissions"` | 跳过所有权限检查（危险）。所有基于路径的确认都被跳过——写 `.git`、`.config/git`、`.claude`、`.vscode`、`.idea`、`.husky`、`.cargo`、`.devcontainer`、`.yarn`、`.mvn` 不再确认（**v2.1.121** 豁免了 `.claude/commands/`、`.claude/agents/`、`.claude/skills/`、`.claude/worktrees/`；**v2.1.126** 移除了其余所有基于路径的确认）。三类操作仍然确认：(1) 针对文件系统根或家目录的删除（`rm -rf /`、`rm -rf ~`，包括用 `$(...)`、反引号替换或 `<(...)` 进程替换的命令），(2) MCP 服务器清单中标记 `requiresUserInteraction` 的工具，(3) 组织设为 ask 的连接器工具。显式 `permissions.ask` 规则仍然确认——被绕过的只是内置权限检查 |
| `"auto"` | 自动批准工具调用，辅以后台安全检查验证操作与你的请求一致。研究预览。分类器自动批准只读和文件编辑；其余全部走安全检查。连续 3 次或累计 20 次阻止后退回弹确认。自 v2.1.111 起进入默认 `Shift+Tab` 权限模式循环（`--enable-auto-mode` 参数已在 v2.1.111 移除——用 `--permission-mode auto` 进入）。用 `autoMode` 设置配置 |
| `"plan"` | 只读探索模式。自 v2.1.136 起，即使存在匹配的 `Edit(...)` allow 规则也阻止文件写入——plan mode 现在覆盖显式 allow 规则以维持只读保证 |

### 工具权限语法

| 工具 | 语法 | 示例 |
|------|--------|----------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`、`Bash(* install)`、`Bash(git * main)` |
| `PowerShell` | `PowerShell(cmd *)` | `PowerShell(Get-ChildItem *)`、`PowerShell(git commit *)` — 形式与 Bash 相同；常见别名会规范化（`gci`/`ls`/`dir` → `Get-ChildItem`），并解析 PowerShell AST，`|`/`;`/`&&`/`||` 链的每个子命令都要匹配 |
| `Read` | `Read(path pattern)` | `Read(.env)`、`Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`、`Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`、`Write(./docs/**)` — **只对 deny 和 ask 规则有效**；allow 规则解析时接受但从不查询（见下方说明） |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` — **只对 deny 和 ask 规则有效**；allow 规则解析时接受但从不查询（见下方说明） |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | 全局网页搜索 |
| `Task` | `Task(agent-name)` | `Task(Explore)`、`Task(my-agent)` — `Agent` 的旧别名；新配置请用 `Agent(name)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`、`Agent(*)` — 权限限定在 subagent 派发 |
| `Skill` | `Skill(skill-name)` 或 `Skill(prefix *)` | `Skill(weather-fetcher)`、`Skill(weather *)` 匹配 `weather-fetcher`/`weather-svg-creator`（v2.1.139）*（不在官方权限文档——未验证）* |
| `MCP` | `mcp__server__tool` | `mcp__memory__*`、`mcp__github__*` — 官方文档只给出双下划线形式；`MCP(server:tool)` 简写 *（不在官方权限文档——未验证）* |
| `Tool` | `Tool(param:value)` | `Agent(model:opus)`、`Agent(isolation:worktree)`、`Bash(run_in_background:true)` — 按工具输入参数匹配 **deny 和 ask 规则**；值位置支持 `*` 通配。**allow 规则不使用此语法**——对单个参数值的 allow 无法建立整体安全性，所以 allow 规则继续用各工具自己的说明符语法。匹配工具的主内容字段（如 `Bash(command:rm *)`）同样被禁止并触发启动警告（v2.1.178） |
| `Cd` | `Cd(path pattern)` | `Cd(/home/*)`、`Cd(~/projects/*)` — 控制 `/cd` 命令可导航到哪些目录。**白名单模式**：加入任意一条 `Cd` allow 规则就把 `/cd` 切换到白名单模式——未显式允许的路径全部拒绝。裸 `Cd` deny 会完全禁用 `/cd`。**通配语义与 Read/Edit 不同**：`*` 恰好匹配一个路径段；`**` 跨段。结尾的 `/**` 也匹配具名根目录本身。这些规则不遵循 gitignore 风格匹配 |

> **v2.1.210**：allow 规则中的 `Write(path)`、`NotebookEdit(path)`、`Glob(path)` 解析时被接受但**从不查询**——Claude Code 的 allow 规则求值只检查 `Edit(path)` 和 `Read(path)`。启动警告会标记这些条目并推荐正确替代。在 **deny** 和 **ask** 规则中，`Write(path)`、`NotebookEdit(path)`、`Glob(path)` 继续正常工作。

**求值顺序**：规则按顺序求值：先 deny，再 ask，再 allow。第一条匹配的规则生效。

**Deny 规则通配（v2.1.166）**：在 deny 规则中，工具名位置用 `"*"` 匹配所有工具——等价于全局拒绝。例如 deny 数组里的 `"*"` 阻止所有工具调用。这样可以完全锁定访问，再为特定操作开 allow/ask 例外。

**Allow 规则通配限制**：allow 规则中，工具名通配只在字面 `mcp__<server>__` 前缀**之后**被接受——server 段不能含通配。未锚定的 allow 通配如 `"*"`、`"B*"`、`"mcp__*"` 会**被跳过并给出启动警告**，不自动批准任何东西。例如 `"allow": ["mcp__github__*"]` 有效，而 `"allow": ["mcp__*"]` 无效。要白名单化请用 `"deny": ["*"]` 加具体 allow 规则。

**Read/Edit 路径模式**：`Read`、`Edit`、`Write` 的权限规则支持 gitignore 风格模式，四种前缀：

| 前缀 | 含义 | 示例 |
|--------|---------|---------|
| `//` | 从文件系统根开始的绝对路径 | `Read(//Users/alice/file)` |
| `~/` | 相对家目录 | `Read(~/.zshrc)` |
| `/` | 相对项目根 | `Edit(/src/**)` |
| `./` 或无前缀 | 相对路径（当前目录） | `Read(.env)`、`Read(*.ts)` |

**符号链接解析**：权限规则同时检查符号链接路径和解析后的目标。**Allow** 规则只在符号链接*及其目标*都匹配时适用——允许目录内的符号链接如果指向目录外，仍会弹确认。**Deny** 规则在符号链接*或目标任一*匹配时适用——指向被拒文件的符号链接本身也被拒。

**Bash 通配要点**：
- `*` 可出现在**任意位置**：前缀（`Bash(* install)`）、后缀（`Bash(npm *)`）、中间（`Bash(git * main)`）
- **词边界**：`Bash(ls *)`（`*` 前有空格）匹配 `ls -la` 但不匹配 `lsof`；`Bash(ls*)`（无空格）两者都匹配
- `Bash(*)` 等价于 `Bash`（匹配所有 bash 命令）
- 权限规则支持输出重定向：`Bash(python:*)` 匹配 `python script.py > output.txt`
- `:*` 后缀语法（如 `Bash(npm:*)`）是尾部通配的等价写法——**未废弃**但只在结尾被识别（如 `Bash(git:* push)` 中的冒号按字面处理）。权限对话框写的是空格形式
- **复合命令**：shell 操作符（`&&`、`||`、`;`、`|`、`|&`、`&` 和换行）会拆分命令，每个子命令必须独立匹配——`Bash(safe-cmd *)` **不**授权 `safe-cmd && other-cmd`
- **进程包装器**：`timeout`、`time`、`nice`、`nohup`、`stdbuf` 在匹配前被剥离（所以 `Bash(npm test *)` 也匹配 `timeout 30 npm test`）；裸 `xargs`（无参数）同样被剥离。执行包装器 `watch`、`setsid`、`ionice`、`flock` 和带 `-exec`/`-delete` 的 `find` 总是弹确认，前缀规则无法批准

**示例：**
```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__memory__*"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(git push *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)",
      "Bash(curl *)"
    ],
    "additionalDirectories": ["../shared-libs/"]
  }
}
```

---

## Hooks

Hook 配置（事件、属性、matcher、退出码、环境变量和 HTTP hook）维护在一个独立仓库：

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — 完整 hook 参考，含声音通知系统、全部 33 个 hook 事件（包括 `PostToolBatch`、`PostToolUseFailure`、`UserPromptExpansion`、`StopFailure`、`CwdChanged`、`DirectoryAdded`、`FileChanged`、`WorktreeCreate`、`WorktreeRemove`、`PostCompact`、`Elicitation`、`ElicitationResult`、`InstructionsLoaded`、`MessageDisplay`、`PermissionDenied`、`TaskCreated` 等）、HTTP hook、`mcp_tool` 处理器类型、`onFailure: "block"` 选项（v2.1.295+）、matcher 模式、退出码和环境变量。

Hook 相关设置键（`hooks`、`disableAllHooks`（同时禁用任何自定义状态行）、`allowManagedHooksOnly`、`allowedHttpHookUrls`、`httpHookAllowedEnvVars`）在该仓库文档中说明。

官方 hook 参考见 [Claude Code Hooks 文档](https://code.claude.com/docs/en/hooks)。

---

## MCP 服务器

配置 Model Context Protocol 服务器以扩展能力。

> **OAuth（v2.1.111）**：通过 OAuth 认证的 MCP 服务器遵循 [RFC 9728](https://datatracker.ietf.org/doc/rfc9728/) 做受保护资源元数据发现。合规服务器在 `/.well-known/oauth-protected-resource` 下暴露授权端点，Claude Code 自动完成 OAuth 流程——符合规范的服务器不需要手写 `apiKeyHelper` 或 `headersHelper` 脚本。

> **保留服务器名（v2.1.128+）**：`workspace`、`Claude Browser`、`Claude Preview` 是保留的 MCP 服务器名。使用这些名字的用户定义服务器在加载时被跳过并向会话日志记录警告。请重命名已有的同名服务器以避免冲突。

> **`.mcp.json` 热重载（v2.1.139）**：`/mcp` 的 Reconnect 操作现在会在重连前从磁盘重新读取 `.mcp.json`，新增或编辑服务器不再需要重启会话。Claude Code 还会向 stdio 启动的 MCP 服务器环境注入 `CLAUDE_PROJECT_DIR`（v2.1.139），让服务器可以相对项目根解析路径。

> **按服务器超时下限（v2.1.162）**：单个服务器的 `timeout` 值低于 1000ms 时被忽略，改用全局 `MCP_TOOL_TIMEOUT` 默认值。≥ 1000ms 的值照常生效。

### MCP 设置

| 键 | 类型 | 作用域 | 说明 |
|-----|------|-------|-------------|
| `enableAllProjectMcpServers` | boolean | 任意 | 自动批准所有 `.mcp.json` 服务器。**安全说明（v2.1.196）**：`.mcp.json` 服务器不再自批准——现在必须通过 `enableAllProjectMcpServers: true` 或 `enabledMcpjsonServers` 显式选择加入 |
| `enabledMcpjsonServers` | array | 任意 | 白名单指定服务器名 |
| `disabledMcpjsonServers` | array | 任意 | 黑名单指定服务器名 |
| `allowedMcpServers` | array | 任意 | 按名称/命令/URL 匹配的白名单。条目跨设置文件合并 |
| `deniedMcpServers` | array | 任意 | 按匹配的黑名单。条目跨设置文件合并 |
| `allowManagedMcpServersOnly` | boolean | 仅托管 | 只允许托管白名单中显式列出的 MCP 服务器 |
| `channelsEnabled` | boolean | 仅托管 | 允许 Team 和 Enterprise 用户使用 [channels](https://code.claude.com/docs/en/channels)。未设置或为 `false` 时，无论 `--channels` 参数如何都阻止 channel 消息投递 |
| `allowedChannelPlugins` | array | 仅托管 | 允许推送消息的 channel 插件白名单。设置后替换默认 Anthropic 白名单。未定义 = 回退默认，空数组 = 阻止所有 channel 插件。需要 `channelsEnabled: true`。每个条目是含 `marketplace` 和 `plugin` 字段的对象（v2.1.84） |
| `managedMcpServers` | object | 仅托管 | 管理员部署的 MCP 服务器定义，优先级高于所有其他作用域（包括用户和项目的 `.mcp.json` 条目）。结构与 `mcpServers` 相同。存在时托管 MCP 服务器无条件加载——用户无法禁用。需要 v2.1.259+ |
| `allowAllClaudeAiMcps` | boolean | 仅托管 | 在 `managed-mcp.json` 之外同时加载 claude.ai 云端 MCP 连接器。启用后，除管理员部署的托管 MCP 服务器外，claude.ai 托管的 MCP 连接器也可用 |
| `disableClaudeAiConnectors` | boolean | 任意 | 禁止自动拉取 claude.ai MCP 连接器。为 `true` 时不加载 claude.ai 云端连接器。**收紧值例外**：任意作用域（包括项目设置）的 `true` 即使面对托管 `false` 也生效——低优先级作用域可以退出但无法加入（v2.1.182） |
| `allowClaudeInChromeWithManagedMcp` | boolean | `false` | **（仅托管，限设备来源）** 允许 Claude in Chrome 访问托管 MCP 服务器。为 `true` 时，基于 Chrome 的 Claude 实例获得对 `managedMcpServers` 中管理员部署 MCP 服务器的访问。需要 v2.1.282+ |
| `allowedProviders` | array | 仅托管 | 限制可用的 API 提供商（Anthropic、Bedrock、Vertex、Foundry）。每个条目是一个提供商标识。设置后，Claude Code 拒绝对列表外提供商启动会话。需要 v2.1.285+。与 `forceLoginMethod` 配对做提供商强制 |

### MCP 服务器匹配（托管设置）

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem"] },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" }
  ]
}
```

> **`${VAR}` 插值（v2.1.219）**：`allowedMcpServers` 和 `deniedMcpServers` 条目可含 `${VAR}` 占位符。Claude Code 在加载时从启动环境和托管设置的 `env` 块解析。用于在托管设置中引用按环境而定的服务器名或 URL 而无需硬编码（如 `{ "serverUrl": "https://${MCP_HOST}/*" }`）。

### 按服务器加载工具（`alwaysLoad`，v2.1.121）

默认情况下 MCP 工具定义是延迟的（按需通过工具搜索加载进上下文）。在 `.mcp.json`（或内联 `mcpServers`）的单个 MCP 服务器条目上设 `alwaysLoad: true` 可豁免该服务器的延迟加载——该服务器的每个工具都会在会话启动时预载，不受 `ENABLE_TOOL_SEARCH` 影响。适用于所有服务器类型；需要 Claude Code v2.1.121+。只对每轮都需要的一小组工具有必要——每个预载工具都占用本可用于对话的上下文。

```json
{
  "mcpServers": {
    "always-on-server": {
      "type": "http",
      "url": "https://mcp.example.com",
      "alwaysLoad": true
    }
  }
}
```

MCP 服务器也可以在工具的 `_meta` 对象中加 `"anthropic/alwaysLoad": true` 把单个工具标记为始终加载——适用于只有一部分工具需要绕过延迟加载的服务器。

**示例：**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## 沙箱

配置 bash 命令沙箱以提升安全。

### 沙箱设置

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `sandbox.enabled` | boolean | `false` | 启用 bash 沙箱 |
| `sandbox.failIfUnavailable` | boolean | `false` | 沙箱已启用但无法启动时报错退出，而不是以非沙箱方式运行。适合要求严格沙箱的企业策略（v2.1.83） |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | 沙箱内自动批准 bash。自 v2.1.139 起，shell 展开形式（`$VAR`、`$(cmd)`）被正确识别，含变量替换的命令不再在沙箱自动批准启用时退回弹确认 |
| `sandbox.excludedCommands` | array | `[]` | 在沙箱外运行的命令 |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | 允许 `dangerouslyDisableSandbox`。设为 `false` 后该逃生口被完全禁用，所有命令必须在沙箱运行（或列入 `excludedCommands`）。适合要求严格沙箱的企业策略 |
| `sandbox.filesystem.disabled` | boolean | `false` | 跳过文件系统隔离但保留网络出口控制。为 `true` 时，沙箱不限制文件访问，但网络出口仍限制在 `sandbox.network.allowedDomains`。**只从用户设置、托管设置或 `--settings` 生效**——在 `.claude/settings.json` 和 `.claude/settings.local.json` 中被忽略（v2.1.216） |
| `sandbox.ignoreViolations` | object | `{}` | 命令模式到路径数组的映射——抑制违规警告 *（在 JSON schema 中，不在官方 settings 页面）* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | **（仅 Linux 和 WSL2）** 为无特权 Docker 环境启用较弱沙箱（降低安全性） |
| `sandbox.network.allowUnixSockets` | array | `[]` | **（仅 macOS）** 沙箱内可访问的特定 Unix socket 路径。在 Linux 和 WSL2 上被忽略（seccomp 过滤器无法检查 socket 路径）；请改用 `allowAllUnixSockets` |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | 允许所有 Unix socket（覆盖 `allowUnixSockets`）。在 Linux 和 WSL2 上这是允许 Unix socket 的唯一方式，因为它跳过了否则会阻止 `socket(AF_UNIX, ...)` 调用的 seccomp 过滤器 |
| `sandbox.network.allowLocalBinding` | boolean | `false` | 允许绑定 localhost 端口（macOS） |
| `sandbox.network.allowedDomains` | array | `[]` | 沙箱的网络域名白名单 |
| `sandbox.network.strictAllowlist` | boolean | `false` | 为 `true` 时，对**不在** `allowedDomains` 的主机直接拒绝网络访问而不弹确认。默认未列出主机会弹确认；本设置把白名单变成硬阻止。有效白名单还包括所有 `WebFetch(domain:...)` allow 规则。只作用于沙箱内 bash 命令的出站连接——不影响 Claude Code 自身的 API 和 WebFetch 调用。**只从用户设置、托管设置或 `--settings` 生效**——在 `.claude/settings.json` 和 `.claude/settings.local.json` 中被忽略（v2.1.219） |
| `sandbox.network.deniedDomains` | array | `[]` | bash 沙箱的网络域名黑名单。优先于 `allowedDomains` 中的通配。支持 glob 模式（如 `"*.example.com"`）（v2.1.113） |
| `sandbox.network.httpProxyPort` | number | - | HTTP 代理端口 1-65535（自定义代理） |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 代理端口 1-65535（自定义代理） |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | 只允许托管白名单（托管设置）中的域名 |
| `sandbox.network.allowMachLookup` | array | `[]` | （仅 macOS）沙箱可查找的额外 XPC/Mach 服务名。支持单个尾部 `*` 前缀匹配。供通过 XPC 通信的工具使用，如 iOS 模拟器或 Playwright。示例：`["com.apple.coresimulator.*"]` |
| `sandbox.filesystem.allowWrite` | array | `[]` | 沙箱内命令可写入的额外路径。数组跨所有设置作用域合并。也与 `Edit(...)` allow 权限规则的路径合并。前缀：`/`（绝对）、`~/`（家）、`./` 或无（项目设置中相对项目，用户设置中相对 `~/.claude`）。旧的绝对路径前缀 `//` 仍可用。**注意**：这与 [Read/Edit 权限规则](#工具权限语法)不同——那边 `//` 表绝对、`/` 表相对项目 |
| `sandbox.filesystem.denyWrite` | array | `[]` | 沙箱内命令不可写入的路径。数组跨所有设置作用域合并。也与 `Edit(...)` deny 权限规则的路径合并。路径前缀约定与 `allowWrite` 相同 |
| `sandbox.filesystem.denyRead` | array | `[]` | 沙箱内命令不可读取的路径。数组跨所有设置作用域合并。也与 `Read(...)` deny 权限规则的路径合并。路径前缀约定与 `allowWrite` 相同 |
| `sandbox.filesystem.allowRead` | array | `[]` | 在 `denyRead` 区域内重新允许读取的路径。覆盖匹配的 `denyRead` glob 条目，但精确路径的 `denyRead` 规则仍阻止精确匹配——`allowRead` 并非压过所有 `denyRead` 模式。数组跨所有设置作用域合并。路径前缀约定与 `allowWrite` 相同 |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **（仅托管层）** 只尊重托管设置中的 `allowRead` 路径。用户、项目和本地设置的 `allowRead` 条目被忽略 |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | （仅 macOS）允许访问系统 TLS 信任（`com.apple.trustd.agent`）；降低安全性 |
| `sandbox.bwrapPath` | string | - | **（仅托管层，Linux/WSL2）** bubblewrap（`bwrap`）二进制的绝对路径。覆盖自动 `PATH` 检测。只从托管设置生效，不从用户或项目设置。示例：`/opt/admin/bwrap`（v2.1.133） |
| `sandbox.socatPath` | string | - | **（仅托管层，Linux/WSL2）** 沙箱网络代理所用 `socat` 二进制的绝对路径。覆盖自动 `PATH` 检测。只从托管设置生效。示例：`/opt/admin/socat`（v2.1.133） |
| `sandbox.allowAppleEvents` | boolean | `false` | **（仅 macOS）** 允许沙箱内命令发送 Apple Events 的选择加入。供使用 `open`、`osascript` 或依赖 Apple Events IPC 的浏览器认证流程的工具使用。**警告**：启用会失去代码执行隔离——Apple Events 可用于在其他应用中执行代码（v2.1.181） |
| `sandbox.credentials` | object | — | 细粒度控制阻止哪些凭据文件和环境变量进入沙箱子进程环境。对象含两个数组：`files`（凭据文件条目数组——见下方子键）和 `envVars`（`{name: string, mode: string, injectHosts?: string[]}` 条目数组——`mode` 为 `"deny"`（默认，剥除变量）或 `"mask"`（替换为占位符，只从用户设置/托管/`--settings` 生效；同一变量两种模式同时出现时 `deny` 优先），`injectHosts` 只对列出的主机暴露真实值）。单个无效条目被剥除并警告；有效子集照常执行（v2.1.187；条目对象结构自 v2.1.191；逐条 `mode` 和 `injectHosts` 自 v2.1.199） |
| `sandbox.credentials.files[].path` | string | — | 要保护的凭据文件的绝对路径或 `~/` 前缀路径 |
| `sandbox.credentials.files[].mode` | string | `"deny"` | `"deny"` 阻止沙箱读取该文件；`"mask"` 拦截读取并替换为占位值，命令仍能运行但无法外泄原始凭据。**`mask` 模式只从用户设置、托管设置或 `--settings` 生效**（v2.1.187 deny；v2.1.221 mask） |
| `sandbox.credentials.files[].extract` | string | — | 含单个捕获组的正则，用于在文件中定位要遮蔽的凭据值。`mode` 为 `"mask"` 且文件不是知名格式时必填。示例：`"oauth_token:\\s*(\\S+)"`（v2.1.221） |
| `sandbox.credentials.files[].injectHosts` | array | — | 即使 `mode` 为 `"mask"`，也通过 TLS 终止代理接收真实凭据值的主机名。需要已配置 `sandbox.network.tlsTerminate`（v2.1.199） |
| `sandbox.credentials.files[].onExtractNoMatch` | string | `"warn"` | `extract` 正则未匹配到内容时的行为：`"warn"`（记录警告、允许读取）、`"deny"`（阻止读取）或 `"error"`（中止沙箱命令）（v2.1.221） |
| `sandbox.credentials.files[].maskDuplicates` | boolean | `false` | 为 `true` 时，同一沙箱命令读取的其他文件和环境变量中出现的相同凭据值也一并遮蔽（v2.1.221） |
| `sandbox.credentials.files[].decode` | string | - | 凭据提取的解码策略。设为 `"jwt"` 启用 JWT 感知遮蔽——文件按 JWT 解码并遮蔽各个 claim。配合 `maskClaims` 指定保护哪些 claim（v2.1.224） |
| `sandbox.credentials.files[].maskClaims` | array | - | `decode` 为 `"jwt"` 时，需要在沙箱命令输出中遮蔽取值的 JWT claim 名列表。示例：`["access_token", "id_token"]`（v2.1.224） |
| `sandbox.credentials.awsPairs` | object | - | 面向沙箱命令的 AWS 凭据对遮蔽。把 AWS access key ID 和对应 secret access key 作为关联对遮蔽。对象含用于凭据遮蔽环境下 AWS SigV4 重签名的凭据对条目。需要相关凭据文件为 `mask` 模式（v2.1.224） |
| `sandbox.credentials.sigv4` | object | - | 面向沙箱 HTTP 请求的 AWS SigV4 重签名配置。配置后，TLS 终止代理在凭据遮蔽之后用 AWS 凭据对出站请求重签名——沙箱命令无需直接接触原始凭据即可发起经过认证的 AWS API 调用。需要 `sandbox.network.tlsTerminate`（v2.1.224） |
| `sandbox.network.tlsTerminate` | object | — | 沙箱 bash 命令的 TLS 终止配置。设置后，Claude Code 充当沙箱出站 HTTPS 的 TLS 中间人，实现凭据遮蔽（`sandbox.credentials` 的 `mask` 模式）。设为 `{}` 为会话生成临时证书颁发机构，或设 `caCertPath` 和 `caKeyPath` 提供自己的 CA。**只从用户设置、托管设置或 `--settings` 生效**——在 `.claude/settings.json` 和 `.claude/settings.local.json` 中被忽略。实验性（v2.1.199） |
| `sandbox.credentials.allowPlaintextInject` | boolean | `false` | 允许 `mask` 凭据替换也应用于**纯 HTTP 请求**（上游身份未验证、凭据明文传输）。为 `false`（默认）时，经 `injectHosts` 的凭据注入只限 TLS 终止的 HTTPS 连接，防止凭据经未加密通道发送。只在明文暴露可接受的受信本地环境使用（v2.1.199） |

**示例：**
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker", "gh"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

---

## 插件

配置 Claude Code 插件和插件市场。

### 插件设置

| 键 | 类型 | 作用域 | 说明 |
|-----|------|-------|-------------|
| `enabledPlugins` | object | 任意 | 启用/禁用特定插件 |
| `extraKnownMarketplaces` | object | 任意 | 添加自定义插件市场。在项目/本地设置中受信任门控。别名：`additionalMarketplaces`。支持 `autoUpdate` 字段。取最高优先级设置文件中的条目整体生效 |
| `strictKnownMarketplaces` | array | 仅托管 | 允许的市场来源对象数组。只有匹配条目的市场被允许；**空数组会阻止包括官方市场在内的一切**。每个条目是 `hostPattern`、`pathPattern` 或 `skills-dir` 来源说明。别名：`allowedMarketplaces`。**Owner 通配条目（v2.1.223）**：接受 `"owner/*"` 匹配组织的全部仓库 |
| `strictPluginOnlyCustomization` | boolean \| array | 仅托管 | 阻止来自用户和项目来源的 skill、agent、hook 和 MCP 服务器，使其只能来自插件或托管设置。`true` 锁定全部四个面；数组如 `["skills", "hooks"]` 只锁定具名项 |
| `pluginSuggestionMarketplaces` | array | 仅托管 | 市场名白名单，其中的插件可在会话中作为情境化安装建议出现。限制哪些市场能弹“你可能想要这个插件”提示（v2.1.152） |
| `skippedMarketplaces` | array | 任意 | 用户拒绝安装的市场 *（在 JSON schema 中，不在官方 settings 页面）* |
| `skippedPlugins` | array | 任意 | 用户拒绝安装的插件 *（在 JSON schema 中，不在官方 settings 页面）* |
| `pluginConfigs` | object | 托管 / 用户 / --settings | 按插件配置的 MCP 服务器（以 `plugin@marketplace` 为键）。自 v2.1.207 起不再从项目级 `.claude/settings.json` 或 `.claude/settings.local.json` 读取；只有用户、`--settings` 和托管设置生效 |
| `blockedMarketplaces` | array | 仅托管 | 阻止特定插件市场。每个条目可按来源字符串、`hostPattern` 或 `pathPattern` 匹配——自 v2.1.119 起，`hostPattern` 和 `pathPattern` 匹配器在任何下载触盘前正确强制执行，被阻止的市场不会落盘 |
| `pluginTrustMessage` | string | 仅托管 | 提示用户信任插件时显示的自定义消息 |
| `disableSideloadFlags` | boolean | 仅托管 | 拒绝 `--plugin-dir`、`--plugin-url`、`--agents`、`--mcp-config` 启动参数。为 `true` 时，用户无法通过启动时传 sideload 参数绕过 `strictKnownMarketplaces`。用于托管环境强制仅市场分发插件（v2.1.193） |
| `disableCommandPluginSources` | boolean | 仅托管 | 阻止 `command` 类型市场来源（通过本地命令动态解析插件目录）。为 `true` 时，即使插件配置里存在 `command` 类型来源条目也被忽略。用于阻止使用 command 来源的插件在托管环境加载（v2.1.229+） |
| `prependPlugins` | array | 托管（无托管层时为用户） | 无论用户设置如何，注入到插件列表最前面的插件。随 Claude Mods 引入（约 v2.1.287） |
| `appendPlugins` | array | 托管（无托管层时为用户） | 注入到用户插件之后、列表末尾的插件。随 Claude Mods 引入（约 v2.1.287） |

**市场来源类型**：`github`、`git`、`directory`、`settings`、`url`、`npm`、`file`、`archive`、`command`。用 `source: 'settings'` 直接在设置里内联声明一小批插件，无需搭建托管市场仓库。用 `source: 'archive'` 做基于 zip 的插件安装（支持 SHA-256 钉住，v2.1.224）。用 `source: 'command'` 做动态插件目录解析——本地命令打印插件目录路径，每个会话重新解析；`mode: "link"` 原地使用而不复制（v2.1.229）。注意：`hostPattern` 是 `blockedMarketplaces` 的*匹配器*字段，不是来源类型。

**Owner 通配条目（v2.1.223）**：`strictKnownMarketplaces` 和 `blockedMarketplaces` 现在接受 `"owner/*"` 通配条目，匹配特定组织或用户的全部仓库。

**示例：**
```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "experimental@acme-tools": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "inline-tools": {
      "source": {
        "source": "settings",
        "name": "inline-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": { "source": "github", "repo": "acme-corp/code-formatter" }
          }
        ]
      }
    }
  }
}
```

---

## 模型配置

### 模型别名

| 别名 | 说明 |
|-------|-------------|
| `"default"` | 为你的账号类型推荐的模型 |
| `"sonnet"` | 最新 Sonnet 模型（v2.1.284 起 Anthropic API 上为 **Claude Sonnet 5.5**；Bedrock、Vertex、Agent Platform、Foundry 上为 Claude Sonnet 4.5）。默认 `medium` effort。原生 1M token 上下文 |
| `"opus"` | 最新 Opus 模型（v2.1.280 起 Anthropic API 和多数提供商为 **Claude Opus 5.5**；Foundry 上为 Opus 4.6）。默认 `medium` effort。原生 1M token 上下文。支持 `ultracode` effort |
| `"haiku"` | 快速 Haiku 模型（v2.1.293 起 Anthropic API 上为 **Claude Haiku 5.5**；其他云提供商为 Haiku 4.5）。默认 `medium` effort |
| `"sonnet[1m]"` | 1M token 上下文的 Sonnet |
| `"opus[1m]"` | 1M token 上下文的 Opus（自 v2.1.75 起为 Max、Team、Enterprise 默认） |
| `"opusplan"` | 规划用 Opus，执行用 Sonnet |
| `"fable"` | Claude Fable 5.1（需要 v2.1.257+）— 长程推理模型。Anthropic API 和多数提供商。Claude 应用网关上的 `fable` 解析为 Fable 5。默认含 1M 上下文；`[1m]` 后缀被自动剥离（v2.1.173） |
| `"best"` | 账号可用则 Fable，否则 Opus |

**示例：**
```json
{
  "model": "opus"
}
```

> **注意（v2.1.144）**：`/model` 会把你的选择保存为新会话的**持久默认**。在 `/model` 选择器里按 `s` 只对当前会话生效。`model` 设置和 `ANTHROPIC_MODEL` 也控制持久默认。

### 模型覆盖

把 Anthropic 模型 ID 映射为 Bedrock、Vertex 或 Foundry 部署的提供商专属模型 ID。

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `effortLevel` | string | - | 跨会话持久化 effort 等级。接受 `"low"`、`"medium"`、`"high"`、`"xhigh"`（Fable 5、Opus 5、Sonnet 5、Opus 4.7、Opus 4.8，v2.1.111）。**`"max"` 和 `"ultracode"` 仅限单会话，在此不接受**——用 `/effort` 或 `--settings` 设置单会话，不要写进 `settings.json`。运行 `/effort <level>` 时自动写入。所有支持 effort 的模型默认 effort 为 `high`，唯独 Opus 4.7 默认 `xhigh`。不支持的等级回落到当前模型支持的 最高等级。**v2.1.251 之前 `/effort` 写 `effortLevel`；自 v2.1.251 起 `/effort` 通过 `modelSettings` 按模型保存默认值**——想给不同模型不同 effort 默认就用 `modelSettings`。注意：Opus 5.5 及之后的 5.x 模型忽略用户设置中的顶层 `effortLevel` |
| `modelSettings` | object | - | 按模型保存的 effort 等级和配置，以模型别名或 ID 为键。每个条目含 `effortLevel`（以及 `maxEffortLevel`、`autoCompactWindow` 字段）。自 v2.1.251 起 `/effort` 写入此键，可为不同模型设不同默认 effort。按模型逐个解析（不是整体对象）。需要 v2.1.251+。示例：`{"opus": {"effortLevel": "xhigh"}, "sonnet": {"effortLevel": "high"}}` |
| `modelPicker` | object | - | 定制 `/model` 选择器的行（带标签、有序）。设置后覆盖默认模型列表。**跨设置文件不合并**——由定义它的最高优先级文件整体提供。示例：`[{"id": "opus", "label": "Opus 5 (reasoning)"}, {"id": "sonnet", "label": "Sonnet 5 (balanced)"}]`（v2.1.242+） |
| `modelPricing` | object | - | **（仅托管层）** 合同约定的按模型费率和折扣系数，应用于用量成本报表。让组织在使用协商价而非公开牌价时也能准确归因成本（v2.1.243+） |
| `fallbackModel` | array | - | 主模型不可用（如限流或容量问题）时按序尝试的最多 3 个回退模型 ID。每个条目是模型 ID 或别名；`"default"` 展开为账号默认。Claude Code 先试主模型；失败后按序试每个回退，首个成功即停止。**与多数数组设置不同，此键跨设置文件不合并**——由定义它的最高优先级文件提供整条链；超过 3 个的条目（去重后）被静默忽略（v2.1.166） |
| `modelOverrides` | object | - | 把模型选择器条目映射为提供商专属 ID（如 Bedrock 推理 profile ARN）。每个键是模型选择器条目名，每个值是提供商模型 ID |
| `maxEffortLevel` | string | 无上限 | **（作用域例外：任意作用域）** 用户通过 `/effort`、`--effort` 或 `effortLevel` 能设置的最高 effort 等级。接受 `low`、`medium`、`high`、`xhigh`、`max`。设置后超过上限的等级被静默钳制。用户、`--settings` 或托管设置中的 `false` 即使项目设 `true` 也保持。用于托管部署的成本控制（v2.1.267+） |
| `deniedModels` | array | - | **（仅托管层）** 用户不能选择的模型 ID 或别名列表。与 `availableModels` 白名单互补——`deniedModels` 是黑名单思路，无需维护完整白名单即可阻止特定新模型。条目按 `availableModelsMatch` 规则匹配（v2.1.283+） |
| `availableModelsMatch` | string | `"prefix"` | **（仅托管层）** `availableModels` 和 `deniedModels` 条目的匹配规则。`"prefix"`（默认）匹配以条目开头的任意模型 ID；`"exact"` 要求精确匹配。需要精确钉住时用 `"exact"`（如只阻止 `claude-opus-5-5` 而不阻止未来 Opus 5.x）。需要 v2.1.283+ |

**示例：**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### Effort 等级

`/model` 命令提供 **effort 等级**控制，调整模型每次回复投入的推理量。在 `/model` 界面用 ← → 方向键切换 effort 等级。

| Effort 等级 | 说明 |
|-------------|-------------|
| Ultracode | 仅限单会话：在 xhigh 推理深度之外触发 ultracode workflow。经 `/effort ultracode` 激活——**不是 settings.json 中合法的 `effortLevel` 值**（v2.1.203+） |
| Max | 仅限单会话：支持的模型（Fable 5、Opus 5、Sonnet 5、Opus 4.8、Opus 4.7、Opus 4.6、Sonnet 4.6）上的最大推理深度。**不是 settings.json 中合法的 `effortLevel` 值** |
| XHigh | 加强的高推理深度。可用于 Fable 5.1、Opus 5.5、Sonnet 5.5、Haiku 5.5、Opus 4.7、Opus 4.8（v2.1.111）。所有套餐的 Opus 4.7 默认此级；其他支持的 4.x 模型默认 `high` |
| High（除 Opus 4.7 外 4.x effort 模型默认） | 完整推理深度，最适合复杂任务 |
| Medium | 平衡推理，适合日常任务 |
| Low | 最少推理，响应最快 |

**用法：**
1. 运行 `/effort low`、`/effort medium`、`/effort high` 直接设置（v2.1.76+）
2. 或运行 `/model` → 选模型 → 用 **← →** 方向键调整
3. 设置通过 `settings.json` 的 `effortLevel` 键持久化。**自 v2.1.243 起 `/effort` 通过 `modelSettings` 按模型保存默认值**——全局 `effortLevel` 仍是 `modelSettings` 无对应条目时的回退

**注意**：Effort 等级适用于 Opus 4.6、Sonnet 4.6、Opus 4.7、Opus 4.8 和所有 5.x 模型。默认值在 v2.1.68 从 High 改为 Medium，又在 v2.1.94 为 API key、Bedrock/Vertex/Foundry、Team、Enterprise 用户改回 **High**。v2.1.117 中，Pro/Max 订阅用户在 Opus 4.6 和 Sonnet 4.6 上的默认也从 `medium` 提到 `high`，各档位统一到 `high`。v2.1.111 引入 **`xhigh`**（当时仅 Opus 4.7）并在所有套餐的 Opus 4.7 上设为默认。**v2.1.154** 加入 **Opus 4.8**；支持 `xhigh` 但默认 `high`。**Claude 5.x 模型（Opus 5.5、Sonnet 5.5、Haiku 5.5）默认 `medium` effort**——低于旧模型的 `high` 默认。自 v2.1.75 起，Opus 4.6 的 1M 上下文窗口在 Max、Team、Enterprise 套餐默认可用。

**Effort 环境变量传播**：在 skill 文件内用 `${CLAUDE_EFFORT}` 引用当前 effort 等级（v2.1.120）。自 v2.1.133 起，同样的 `$CLAUDE_EFFORT` 变量也注入 Bash 工具子进程和 hook 处理器的环境，shell 脚本和 hook 命令无需另读配置文件即可按当前 effort 档位调整行为。

### 模型环境变量

通过 `env` 键配置：

```json
{
  "env": {
    "ANTHROPIC_MODEL": "sonnet",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "custom-haiku-model",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "custom-sonnet-model",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "custom-opus-model",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku",
    "MAX_THINKING_TOKENS": "10000"
  }
}
```

---

## 显示与 UX

### 显示设置

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `statusLine` | object | - | 自定义状态行配置 |
| `outputStyle` | string | `"default"` | 输出风格（如 `"Explanatory"`） |
| `spinnerTipsEnabled` | boolean | `true` | 等待时显示提示 |
| `spinnerVerbs` | object | - | 自定义 spinner 动词，含 `mode`（"append" 或 "replace"）和 `verbs` 数组 |
| `spinnerTipsOverride` | object | - | 自定义 spinner 提示，含 `tips`（字符串数组）和可选 `excludeDefault`（boolean）。`excludeDefault` 为 `true` 时只显示自定义提示；为 `false` 或缺省时与内置提示合并。自 v2.1.121 起，`excludeDefault: true` 也抑制按时间的 spinner 提示 |
| `respectGitignore` | boolean | `true` | 文件选择器尊重 .gitignore |
| `prefersReducedMotion` | boolean | `false` | 减少 UI 动画和动态效果 |
| `axScreenReader` | boolean | `false` | 启用适合屏幕阅读器的输出模式。为 `true` 时，Claude 输出不含装饰性制表符、颜色和其他终端 UI 元素的纯平文本。在 `/config` 中显示为 **Screen reader mode**。也可用 `--ax-screen-reader` CLI 参数（v2.1.181） |
| `syntaxHighlightingDisabled` | boolean | `false` | 禁用 diff、代码块和文件预览中的语法高亮。与 `CLAUDE_CODE_SYNTAX_HIGHLIGHT` 环境变量不同——那只管 diff 输出 |
| `fileSuggestion` | object | - | 自定义文件建议命令（见下方文件建议配置） |
| `autoScrollEnabled` | boolean | `true` | 全屏模式下自动滚动对话。设为 `false` 禁用自动滚动（v2.1.110）。v2.1.119 之前版本存在 `~/.claude.json` |
| `editorMode` | string | `"normal"` | 输入框的按键绑定模式：`"normal"` 或 `"vim"`。在 `/config` 中显示为 **Editor mode**。v2.1.119 之前版本存在 `~/.claude.json` |
| `vimInsertModeRemaps` | object | - | vim 插入模式下应用的自定义按键重映射。对象的键必须是恰好两个可打印字符（输入序列），值是替换序列——唯一有效目标值是 `<Esc>`。只在 `editorMode` 为 `"vim"` 时生效。示例：`{"jk": "<Esc>", "jj": "<Esc>"}`（v2.1.208） |
| `showTurnDuration` | boolean | `true` | 回复后显示轮次耗时消息（如 "Cooked for 1m 6s"）。v2.1.119 之前版本存在 `~/.claude.json` |
| `teammateMode` | string | `"in-process"` | [agent team](https://code.claude.com/docs/en/agent-teams) 队友的显示方式：`"auto"`（tmux 或 iTerm2 中选分屏，否则 in-process）、`"in-process"`（v2.1.179 起默认）、`"tmux"`、`"iterm2"`（无视自动检测强制 iTerm2 分屏，v2.1.186）。见[选择显示模式](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode)。v2.1.119 之前版本存在 `~/.claude.json` |
| `terminalProgressBarEnabled` | boolean | `true` | 在支持的终端（ConEmu、Ghostty 1.2.0+、iTerm2 3.6.6+）中显示终端进度条。在 `/config` 中显示为 **Terminal progress bar**。v2.1.119 之前版本存在 `~/.claude.json` |
| `preferredNotifChannel` | string | `"auto"` | 任务完成和权限确认的通知方式。取值：`"auto"`、`"terminal_bell"`、`"iterm2"`、`"iterm2_with_bell"`、`"kitty"`、`"ghostty"`、`"notifications_disabled"`。默认 `"auto"` 在 iTerm2、Ghostty、Kitty 发桌面通知，其他终端不做任何事。设 `"terminal_bell"` 在任意终端响铃。在 `/config` 中显示为 **Notifications**。见[获取终端响铃或通知](https://code.claude.com/docs/en/terminal-config#get-a-terminal-bell-or-notification) |
| `wheelScrollAccelerationEnabled` | boolean | `true` | 全屏模式下禁用鼠标滚轮加速。设为 `false` 改用固定步进滚动，而不是操作系统级加速曲线（v2.1.174） |
| `footerLinksRegexes` | array | - | 对**轮次输出**（工具结果、文件内容、抓取页面、Claude 回复）匹配的对象数组，匹配项在底栏显示为链接徽章。每个条目为 `{type, pattern, url, label}`，其中 `pattern` 是含命名捕获组的正则，`url`/`label` 可引用这些组。匹配到的模式在聊天界面底部生成可点击徽章。每轮最多 5 个徽章；URL 最长 2048 字符；允许的协议：`http`、`https`、`vscode`、`cursor`、`windsurf`、`zed`、`jetbrains`、`idea`、`slack`、`linear`、`notion`、`figma`、`vscode-insiders`。仅限用户/`--settings`/托管（v2.1.176） |
| `emojiCompletionEnabled` | boolean | `true` | 在提示输入中启用 emoji 短代码自动补全（如输入 `:tada:` 会补全出对应的庆祝 emoji）。设为 `false` 禁用。需要 v2.1.217+ |
| `keybindingFlavor` | string | - | 删词的键盘快捷键风格。取值：`"readline"`（Bash/GNU readline 风格 Ctrl+W——向前删到空白边界）、`"classic"`（默认删词行为）。未设置时用 classic 行为（v2.1.236） |
| `spellcheck` | object | - | 提示输入中的拼写检查下划线。作用域：用户或托管。对象含 `enabled`（boolean）、`checker`（string——`"aspell"`、`"hunspell"`、`"ispell"` 或 `"auto"`）、`language`（string——如 `"en_US"`）、`color`（string——下划线颜色）。通过 `/config` 设置。需要 v2.1.235+ |
| `promptCacheTtl` | string | - | 覆盖主对话的提示缓存 TTL。接受 `"5m"` 或 `"1h"`。优先级：`FORCE_PROMPT_CACHING_5M` > `CLAUDE_CODE_PROMPT_CACHE_TTL` > 本键 > `ENABLE_PROMPT_CACHING_1H`。与 `subagentPromptCacheTtl` 配对（v2.1.243+） |
| `subagentPromptCacheTtl` | string | - | 覆盖 subagent、workflow 和后台/辅助请求的提示缓存 TTL。接受 `"5m"` 或 `"1h"`。优先级：`FORCE_PROMPT_CACHING_5M` > `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` > 本键 > `ENABLE_PROMPT_CACHING_1H`。与 `promptCacheTtl` 配对独立控制（v2.1.243+） |
| `timeFormat` | string | `"auto"` | UI 时间显示格式。取值：`"auto"`、`"12-hour"`、`"24-hour"`、`"24-hour-utc"`，或 `strftime` 格式串。默认 `"auto"` 跟随系统区域。需要 v2.1.257+ |
| `timeZone` | string | - | 时间显示用的 IANA 时区名（如 `"America/New_York"`、`"UTC"`）。未设置时用系统时区。需要 v2.1.257+ |
| `maxProseWidth` | number | - | 散文输出最大宽度（字符，最小 40）。设置后 Claude 把长文本折行到该宽度内。需要 v2.1.282+ |
| `subagentStatusLine` | object | - | subagent 运行时显示的自定义状态行。对象含 `type`（设为 `"command"`）和 `command`（shell 命令）。受 `disableAllHooks`/`allowManagedHooksOnly` 门控。需要 v2.1.214+ |
| `bashEditDiffEnabled` | boolean | - | Bash 命令编辑文件时显示 diff 预览。作用域：用户或托管。环境变量覆盖：`CLAUDE_CODE_BASH_EDIT_DIFF`。需要 v2.1.269+ |

### 全局配置设置（`~/.claude.json`）

这些 IDE 相关偏好存在 `~/.claude.json`，**不是** `settings.json`。

> **v2.1.119 迁移说明**：自 v2.1.119 起，`autoScrollEnabled`、`editorMode`、`showTurnDuration`、`teammateMode`、`terminalProgressBarEnabled` 移入 `settings.json`，见上方显示设置表。更早版本存在这里。

| 键 | 类型 | 默认值 | 说明 |
|-----|------|---------|-------------|
| `autoConnectIde` | boolean | `false` | Claude Code 从外部终端启动时自动连接运行中的 IDE。在 VS Code 或 JetBrains 终端之外运行时，`/config` 中显示为 **Auto-connect to IDE (external terminal)** |
| `autoInstallIdeExtension` | boolean | `true` | 从 VS Code 终端运行时自动安装 Claude Code IDE 扩展。在 `/config` 中显示为 **Auto-install IDE extension**。也可用 `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` 环境变量禁用 |
| `externalEditorContext` | boolean | `false` | 用 `Ctrl+G` 打开外部编辑器时，把 Claude 上一次回复以 `#` 注释形式预置为上下文。设为 `true` 启用 |
| `teammateDefaultModel` | string | `null` | **v2.1.234 已移除。** 此前用于设置 lead 派发 [agent-team](https://code.claude.com/docs/en/agent-teams) 队友时的默认模型。现在队友默认继承 lead 的模型 |
| `diffTool` | string | - | 查看文件 diff 时调用的外部 diff 工具命令。设置后，Claude Code 以两个文件路径为参数启动该命令，而不是渲染内置 diff 视图 |
| `permissionExplainerEnabled` | boolean | `true` | **v2.1.257 已移除。** 此前在权限确认旁显示 AI 生成的自然语言解释。该设置现在是空操作；权限解释器已从 UI 移除 |

### 工作区与团队

| 键 | 类型 | 说明 |
|-----|------|-------------|
| `sshConfigs` | object[] | 在 Desktop 中显示为下拉列表的 SSH 连接定义。每个条目必须含 `id`、`name`、`sshHost`；可选 `sshPort`、`sshIdentityFile`、`startDirectory`。只从托管和用户设置生效——项目设置值被忽略 |

**字段参考：**

| 字段 | 必填 | 说明 |
|-------|----------|-------------|
| `id` | 是 | SSH 连接条目的唯一标识 |
| `name` | 是 | Desktop 下拉中显示的名称 |
| `sshHost` | 是 | SSH 主机（如 `user@dev.example.com` 或 `dev.example.com`） |
| `sshPort` | 否 | SSH 端口号 |
| `sshIdentityFile` | 否 | SSH 身份文件（私钥）路径 |
| `startDirectory` | 否 | 连接后的初始工作目录 |

**示例：**
```json
{
  "sshConfigs": [
    {
      "id": "dev-vm",
      "name": "Dev VM",
      "sshHost": "user@dev.example.com",
      "sshPort": 22,
      "sshIdentityFile": "~/.ssh/id_ed25519",
      "startDirectory": "/home/user/project"
    }
  ]
}
```

### 状态行配置

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2,
    "refreshInterval": 5
  }
}
```

| 字段 | 说明 |
|-------|-------------|
| `type` | 设为 `"command"` 运行 shell 脚本 |
| `command` | 生成状态行输出的 shell 命令或脚本路径 |
| `padding` | 加到状态行内容的额外水平间距（字符）。默认 `0`。控制在界面内置间距之外的相对缩进 |
| `refreshInterval` | 除事件驱动更新外，每 N 秒重新运行命令。最小 `1`。适合状态行显示时间相关数据（如时钟）或主会话空闲时后台 subagent 改变 git 状态的场景。不设置则只在事件时运行（v2.1.97） |
| `hideVimModeIndicator` | 为 `true` 时从状态区隐藏内置 vim 模式指示器，让自定义状态行命令自行渲染。只在 `editorMode` 为 `"vim"` 时有效 |

**状态行输入字段：**

状态行命令通过 stdin 接收 JSON 对象。完整 JSON schema 和示例见[状态行文档](https://code.claude.com/docs/en/statusline)。

| 字段 | 说明 |
|-------|-------------|
| `model.id`、`model.display_name` | 当前模型标识和显示名 |
| `cwd`、`workspace.current_dir` | 当前工作目录（两者同值；推荐 `workspace.current_dir`） |
| `workspace.project_dir` | Claude Code 启动时的目录（工作目录变化后可能与 `cwd` 不同） |
| `workspace.added_dirs` | 通过 `/add-dir` 或 `--add-dir` 添加的额外目录 |
| `workspace.git_worktree` | 在 `git worktree add` 创建的链接 worktree 内时的 worktree 名。主工作树中不存在（v2.1.97） |
| `cost.total_cost_usd` | 会话总费用（美元） |
| `cost.total_duration_ms` | 会话开始以来的总耗时（毫秒） |
| `cost.total_api_duration_ms` | 等待 API 响应的总时间（毫秒） |
| `cost.total_lines_added`、`cost.total_lines_removed` | 会话中改动的代码行数 |
| `context_window.total_input_tokens`、`context_window.total_output_tokens` | 会话累计 token 数 |
| `context_window.context_window_size` | 最大上下文窗口（token，默认 200000，扩展上下文 1000000） |
| `context_window.used_percentage` | 预计算的上下文窗口已用百分比 |
| `context_window.remaining_percentage` | 预计算的上下文窗口剩余百分比 |
| `context_window.current_usage` | 最近一次 API 调用的 token 数（输入、输出、缓存 token） |
| `exceeds_200k_tokens` | 最近 API 响应的总 token 是否超过 200k（固定阈值） |
| `rate_limits.five_hour.used_percentage` | 5 小时限额使用百分比（v2.1.80+） |
| `rate_limits.five_hour.resets_at` | 5 小时限额重置时间戳（Unix 秒） |
| `rate_limits.seven_day.used_percentage` | 7 天限额使用百分比 |
| `rate_limits.seven_day.resets_at` | 7 天限额重置时间戳（Unix 秒） |
| `session_id` | 唯一会话标识 |
| `session_name` | 用 `--name` 或 `/rename` 设置的自定义会话名。未设置时不存在 |
| `transcript_path` | 对话记录文件路径 |
| `version` | Claude Code 版本 |
| `output_style.name` | 当前输出风格名称 |
| `vim.mode` | 启用 vim 模式时的当前 vim 模式（`NORMAL` 或 `INSERT`） |
| `agent.name` | 使用 `--agent` 参数或 agent 设置运行时的 agent 名 |
| `effort.level` | 当前推理 effort（`low`、`medium`、`high`、`xhigh`、`max`）。反映会话实时值，包括会话中 `/effort` 修改。当前模型不支持 effort 参数时不存在（v2.1.121） |
| `thinking.enabled` | 会话是否启用扩展思考（v2.1.121） |
| `worktree.name` | 活动 worktree 名（仅在 `--worktree` 会话期间存在） |
| `worktree.path` | worktree 目录绝对路径 |
| `worktree.branch` | worktree 的 git 分支名。基于 hook 的 worktree 无此字段 |
| `worktree.original_cwd` | 进入 worktree 前的目录 |
| `worktree.original_branch` | 进入 worktree 前检出的 git 分支。基于 hook 的 worktree 无此字段 |
| `github` | 检测到时当前分支的 GitHub 仓库和 PR 信息——仓库标识及关联 PR（v2.1.145） |

### 文件建议配置

文件建议脚本通过 stdin 接收 JSON 对象（如 `{"query": "src/comp"}`），必须输出最多 15 个文件路径（每行一个）。

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**示例：**
```json
{
  "statusLine": {
    "type": "command",
    "command": "git branch --show-current 2>/dev/null || echo 'no-branch'"
  },
  "spinnerTipsEnabled": true,
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Cooking", "Brewing", "Crafting", "Conjuring"]
  },
  "spinnerTipsOverride": {
    "tips": ["Use /compact at ~50% context", "Start with plan mode for complex tasks"],
    "excludeDefault": true
  }
}
```

---

## AWS 与云凭据

### AWS 设置

| 键 | 类型 | 说明 |
|-----|------|-------------|
| `awsAuthRefresh` | string | 刷新 AWS 认证的脚本（修改 `.aws` 目录） |
| `awsCredentialExport` | string | 输出含 AWS 凭据 JSON 的脚本 |

**示例：**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| 键 | 类型 | 说明 |
|-----|------|-------------|
| `otelHeadersHelper` | string | 动态生成 OpenTelemetry 头的脚本 |

**示例：**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## 环境变量（通过 env）

为所有 Claude Code 会话设置环境变量。

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### 常用环境变量

| 变量 | 说明 |
|----------|-------------|
| `ANTHROPIC_API_KEY` | 认证用 API key |
| `ANTHROPIC_AUTH_TOKEN` | OAuth token |
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude.ai 认证的 OAuth access token。SDK 和自动化环境中 `/login` 的替代。优先于钥匙串存储的凭据 |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | Claude.ai 认证的 OAuth refresh token。设置后 `claude auth login` 直接交换该 token 而不打开浏览器。需要 `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | refresh token 被授予时带的作用域，空格分隔（如 `"user:profile user:inference user:sessions:claude_code"`）。设置 `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` 时必填 |
| `ANTHROPIC_WORKSPACE_ID` | [工作负载身份联合](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation)的工作区 ID。当联合规则限定多个工作区时设置，让 token 交换知道目标工作区（v2.1.141） |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `ANTHROPIC_BEDROCK_BASE_URL` | 覆盖 Bedrock 端点 URL |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | 覆盖 Bedrock Mantle 端点 URL。见 [Mantle 端点](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock 服务档位：`default`、`flex`、`priority`。作为 `X-Amzn-Bedrock-Service-Tier` 头随每个请求发送。见 [Amazon Bedrock 服务档位](https://code.claude.com/docs/en/amazon-bedrock#service-tiers)（v2.1.122） |
| `ANTHROPIC_AWS_API_KEY` | Claude Platform on AWS 的工作区 API key |
| `ANTHROPIC_AWS_BASE_URL` | 覆盖 Claude Platform on AWS 端点 URL |
| `ANTHROPIC_AWS_WORKSPACE_ID` | Claude Platform on AWS 必需的工作区 ID |
| `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` | 由嵌入 Claude Code 并代管模型提供商路由的宿主平台设置。设置后，`settings.json` 中提供商选择/端点/认证类环境变量（如 `CLAUDE_CODE_USE_BEDROCK`、`ANTHROPIC_BASE_URL`、`ANTHROPIC_API_KEY`）被忽略，用户设置无法覆盖宿主路由。Bedrock/Vertex/Foundry 的自动遥测退出也被跳过，遥测遵循标准 `DISABLE_TELEMETRY` 退出（v2.1.126） |
| `ANTHROPIC_VERTEX_BASE_URL` | 覆盖 Vertex AI 端点 URL |
| `ANTHROPIC_BETAS` | 逗号分隔的 Anthropic beta 头值 |
| `ANTHROPIC_VERTEX_PROJECT_ID` | Vertex AI 的 GCP 项目 ID |
| `GCLOUD_PROJECT` | Vertex AI 请求的 GCP 项目 ID（覆盖 `ANTHROPIC_VERTEX_PROJECT_ID`） |
| `GOOGLE_APPLICATION_CREDENTIALS` | Vertex AI 认证用的 GCP 服务账号凭据文件路径 |
| `GOOGLE_CLOUD_PROJECT` | Vertex AI 请求的 GCP 项目 ID（覆盖 `ANTHROPIC_VERTEX_PROJECT_ID`） |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | 添加到 `/model` 选择器的自定义条目模型 ID。让非标准或网关专属模型可选，而不替换内置别名 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | `/model` 选择器中自定义模型条目的显示名。未设置时默认为模型 ID |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | `/model` 选择器中自定义模型条目的显示描述。未设置时默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES` | 覆盖自定义模型条目的能力检测。逗号分隔值（如 `effort,thinking`）。自定义模型支持自动检测无法确认的功能时必填。见[模型配置](https://code.claude.com/docs/en/model-config#customize-pinned-model-display-and-capabilities) |
| `ANTHROPIC_MODEL` | 使用的模型名。接受别名（`sonnet`、`opus`、`haiku`）或完整模型 ID。覆盖 `model` 设置 |
| `INIT_PROMPT` | 会话初始化时注入的自定义系统提示 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 用自定义模型 ID 覆盖 Haiku 模型别名（如第三方部署） |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上钉定模型时，自定义 `/model` 选择器中 Haiku 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Haiku 条目描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | 覆盖钉定 Haiku 模型的能力检测。逗号分隔值（如 `effort,thinking`）。钉定模型支持自动检测无法确认的功能时必填 |
| `CLAUDE_PID` | 只读。在每个派生子进程（Bash 工具、hook、状态行命令、MCP stdio 服务器）中设为 Claude Code 进程的 PID。供脚本向父 Claude Code 进程发信号或识别 Claude Code 血统。v2.1.214 引入 |
| `CLAUDECODE` | 在 Claude Code 派生的 shell 环境（Bash 工具、tmux 会话）中设为 `1`。hook 和状态行命令中不设。用于检测脚本是否运行在 Claude Code shell 内 |
| `CLAUDE_CODE_CHILD_SESSION` | 在 Claude Code 经 Bash、PowerShell、Monitor 工具派生的子进程、hook 命令和状态行命令中设为 `1`。stdio MCP 服务器子进程不设。与 `CLAUDECODE` 不同，它只由 Claude Code 自身的派生路径设置（IDE 扩展不设），因此能可靠区分嵌套 `claude` 会话与 IDE 集成终端里启动的顶层 `claude`。嵌套交互式 TUI 会话自动排除在 `--resume`、`--continue`、上箭头历史和 `claude agents` 之外。非交互 `claude -p` 会话仍会持久化。设 `CLAUDE_CODE_FORCE_SESSION_PERSISTENCE=1` 可覆盖该排除（v2.1.172） |
| `CLAUDE_CODE_FORCE_SESSION_PERSISTENCE` | 设为 `1` 覆盖对嵌套交互式 TUI 会话的自动排除（`--resume`、`--continue`、上箭头历史、`claude agents`）。默认嵌套会话（`CLAUDE_CODE_CHILD_SESSION=1`）被排除以防污染历史。想跟踪嵌套会话时设置此变量强制持久化 |
| `CLAUDE_CODE_SESSION_ID` | 只读。在 Bash 和 PowerShell 工具子进程中自动设为当前会话 ID。与传给 hook 的 `session_id` 字段一致。`/clear` 时更新。用于把脚本和外部工具与启动它们的 Claude Code 会话关联（v2.1.132）。`--resume` 时也注入 stdio MCP 服务器环境 |
| `AI_AGENT` | 由 Claude Code 在子进程环境（Bash 工具、hook、MCP stdio 服务器）自动设置。标识父进程是 AI agent 的通用标志——适合会针对任何 AI agent 调整行为的工具，而不用逐个检查 `CLAUDECODE` 这类 agent 专属变量 *（在 v2.1.120 changelog，尚未进入官方 env-vars 页）* |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | 设为 `1`，组织状态检查因网络错误失败时仍允许 fast mode。适用于企业代理阻止状态端点的场景 |
| `CLAUDE_CODE_USE_BEDROCK` | 使用 AWS Bedrock（`1` 启用） |
| `CLAUDE_CODE_USE_VERTEX` | 使用 Google Vertex AI（`1` 启用） |
| `CLAUDE_CODE_USE_FOUNDRY` | 使用 Microsoft Foundry（`1` 启用） |
| `CLAUDE_CODE_USE_MANTLE` | 使用 Bedrock [Mantle 端点](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint)（`1` 启用） |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | 设为 `1` 在 Windows 启用 PowerShell 工具（选择加入预览）。启用后 Claude 可原生运行 PowerShell 命令而非经 Git Bash。仅支持原生 Windows，不支持 WSL（v2.1.84） |
| `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` | 设为 `1` 后，Claude Code 为工具调用、hook 和状态行命令启动 PowerShell 时不再传 `-ExecutionPolicy Bypass`，改为尊重机器的有效执行策略。默认 Claude Code 在进程作用域绕过执行策略，让 `.ps1` 脚本和模块导入在默认 Restricted 的 Windows 上可用。绝不覆盖组策略 `MachinePolicy`/`UserPolicy`（v2.1.143） |
| `CLAUDE_CODE_REMOTE` | 只读。Claude Code 作为云会话运行时自动设为 `true`。在 hook 或安装脚本中读取它可判断是否处于云环境 |
| `CLAUDE_CODE_REMOTE_SESSION_ID` | 只读。云会话中自动设为当前会话 ID。读取它可构造回到会话记录的链接 |
| `CLAUDE_CODE_BRIDGE_SESSION_ID` | 只读。会话有活动 Remote Control 连接时，在 Bash 工具和 hook 命令子进程中自动设置。值为 `session_` 形式的会话 ID（与会话 `claude.ai/code` URL 中的标识一致）。连接结束后移除。让脚本能链回运行它们的会话。云会话中请改读 `CLAUDE_CODE_REMOTE_SESSION_ID`（v2.1.199） |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | 自动生成的 Remote Control 会话名前缀。默认为主机名 |
| `CLAUDE_CLIENT_PRESENCE_FILE` | 文件路径；该文件存在时表示有活动客户端，抑制 Remote Control 的移动端推送。适合桌面客户端常驻、不想要手机 ping 的环境 |
| `CLAUDE_CODE_DISABLE_NOTIFICATION_PRESENCE_CHECK` | 设为 `1`，用户正在会话中输入时仍发送推送。默认检测到活跃用户时抑制通知（v2.1.193） |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 启用/禁用遥测（`0` 或 `1`） |
| `DISABLE_ERROR_REPORTING` | 禁用错误上报（`1` 禁用） |
| `DISABLE_AUTOUPDATER` | 设为 `1` 禁用对 npm registry 的自动更新检查。也可作为仅启动时生效的变量——见 [CLI 启动参数](claude-cli-startup-flags.md#environment-variables) |
| `DISABLE_UPDATES` | 设为 `1` 完全阻断所有更新路径——自动检查、通知和手动 `claude update`。比只禁用后台检查的 `DISABLE_AUTOUPDATER` 更严格。用于必须显式重新启用前阻断一切更新的环境 |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | 设为 `1`，新版本可用时让 Claude Code 后台运行包管理器的升级命令。适用于 Homebrew 和 WinGet 安装。其他包管理器仍只显示升级命令而不执行。见[自动更新](https://code.claude.com/docs/en/setup#auto-updates)（v2.1.129） |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | 设为 `1`，当 `ANTHROPIC_BASE_URL` 指向 Anthropic 兼容网关（如 LiteLLM、Kong 或内部代理）时，从网关的 `/v1/models` 端点填充 `/model` 选择器。默认关闭，因为共享 API key 背后的网关会暴露该 key 可访问的每个模型。发现的模型仍受 `availableModels` 白名单过滤（v2.1.129，从此前的自动发现改为选择加入） |
| `DISABLE_TELEMETRY` | 禁用遥测（`1` 禁用） |
| `DO_NOT_TRACK` | 标准退出变量；设为 `1` 退出遥测收集。被 `DISABLE_TELEMETRY` 认可 |
| `MCP_TIMEOUT` | MCP 启动超时（ms） |
| `MCP_CONNECT_TIMEOUT_MS` | MCP 服务器 TCP/Unix 连接阶段超时（ms，默认 `5000`）。与 `MCP_TIMEOUT`（启动）和 `MCP_TOOL_TIMEOUT`（单次调用）不同 |
| `MCP_SERVER_CONNECTION_BATCH_SIZE` | 启动时并行连接的本地（stdio）MCP 服务器数。用于在服务器很多的机器上限制启动负载 |
| `MCP_REMOTE_SERVER_CONNECTION_BATCH_SIZE` | 启动时并行连接的远程（HTTP/SSE）MCP 服务器数。与本地批量分开，以考虑网络延迟 |
| `CLAUDE_CODE_MCP_ALLOWLIST_ENV` | 只用安全基线环境启动 stdio MCP 服务器，剥除大多数继承的环境变量，防止凭据泄入不可信服务器进程 |
| `MAX_MCP_OUTPUT_TOKENS` | MCP 最大输出 token（默认 25000）。输出超 10,000 token 时显示警告 |
| `API_TIMEOUT_MS` | API 请求超时（ms，默认 600000） |
| `API_FORCE_IDLE_TIMEOUT` | 覆盖流式连接的 5 分钟空闲超时。设 `0` 完全禁用空闲超时，`1` 对所有连接强制，或不设置用默认（在频繁卡顿的慢速或不可靠网关上自动启用）。适用于慢速 API 网关（v2.1.169） |
| `CLAUDE_CODE_CONNECT_TIMEOUT_MS` | **v2.1.186 已移除。** 此变量是空操作。请改用 `API_TIMEOUT_MS`。此前控制流式 API 请求的连接、TLS 和响应头阶段超时 |
| `CLAUDE_AFK_TIMEOUT_MS` | `askUserQuestionTimeout` 设为时长时，未应答的 AskUserQuestion 对话框多少毫秒后自动继续。自 v2.1.200 起默认 `"never"`（不自动继续），由 `askUserQuestionTimeout` 控制；此环境变量只在时长生效时适用。设 `0` 立即关闭对话框。想让问题保持打开请用 `askUserQuestionTimeout: "never"`（v2.1.198；默认值 v2.1.200 改变） |
| `CLAUDE_AFK_COUNTDOWN_MS` | 未应答 AskUserQuestion 对话框自动继续前多少毫秒出现屏上倒计时（默认 `20000` / 20 秒）（v2.1.198） |
| `BASH_MAX_TIMEOUT_MS` | Bash 命令超时 |
| `BASH_MAX_OUTPUT_LENGTH` | Bash 最大输出长度 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 自动压缩阈值百分比（1-100）。默认约 95%。调低（如 `50`）更早触发压缩。超过 95% 的值无效。用 `/context` 监控当前用量。示例：`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | 覆盖 Claude Code 为当前模型认定的上下文窗口大小。只在同时设置 `DISABLE_COMPACT` 时生效。用于经 `ANTHROPIC_BASE_URL` 路由到上下文窗口与其名称内置大小不符的模型 |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 设为 `1` 后，每条 Bash 或 PowerShell 命令之后**回到原工作目录**。默认命令内的目录变更会延续到下一条命令——此变量让每次调用后恢复原 cwd |

| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 禁用后台任务（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP` | 设为 `1` 禁用对空闲后台 shell 命令的内存压力自动回收。未设置时，Claude Code 会在内存压力下自动回收空闲后台 shell 以释放资源 |
| `CLAUDE_CODE_DISABLE_BG_EXIT_HANDOFF` | 设为 `1`，supervisor 停止、重启或更新某后台会话进程时，一并停止该会话正在运行的后台 shell 命令、动态 workflow 和后台 subagent。默认它们被移交给会话的下一个进程（v2.1.198） |
| `CLAUDE_CODE_DISABLE_ADVISOR_TOOL` | 设为 `1` 禁用 advisor 工具和 `/advisor` 命令。等价于不使用 advisor 的环境变量形式。与 `advisorModel` 配对做 advisor 配置（最低 v2.1.98） |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | 设为 `1` 关闭后台 agent 与 agent 视图（`claude agents`、`--bg`、`/background`、按需 supervisor）。`disableAgentView` 设置的环境变量等价物 |
| `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS` | 设为 `1` 禁用内置 Explore 和 Plan subagent。Claude 改用搜索工具或 general-purpose subagent 探索。Plan mode 直接读文件而不是启动 Explore 和 Plan agent（v2.1.198） |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | 启用实验性 agent teams 功能（`1` 启用）。允许在会话内生成协同的 subagent 团队。也可作为仅启动时生效的变量——见 [CLI 启动参数](claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_DISABLE_WORKFLOWS` | 设为 `1` 禁用[动态 workflow](https://code.claude.com/docs/en/workflows)（`/workflows`）和内置 workflow 斜杠命令。`disableWorkflows` 设置的环境变量等价物 |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | **自 v2.1.207 起，auto mode 在 Bedrock、Vertex AI、Foundry 上默认可用——不再需要此选择加入。** 此前（v2.1.158–v2.1.206）：设 `1` 使 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 在这些提供商上可用。要关闭 auto mode 请用设置中的 `disableAutoMode: "disable"` |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` | 设为 `1` 对模型隐藏 Claude Code 的内置能力（bundled skills）。`disableBundledSkills` 设置的环境变量等价物（v2.1.169） |
| `CLAUDE_CODE_DISABLE_ARTIFACT` | 设为 `1` 禁用 [Artifact](https://code.claude.com/docs/en/artifacts) 工具（把会话输出发布为 claude.ai 私有网页）。等价于 `disableArtifact` 设置 |
| `CLAUDE_CODE_ARTIFACT_AUTO_OPEN` | 设为 `0` 阻止新 artifact 发布时 Claude Code 自动打开浏览器。重新发布已有 artifact 无论此设置如何都不打开浏览器 |
| `ENABLE_TOOL_SEARCH` | MCP 工具搜索阈值（如 `auto:5`） |
| `ENABLE_PROMPT_CACHING_1H` | 选择加入 1 小时提示缓存 TTL。替代已废弃的 `ENABLE_PROMPT_CACHING_1H_BEDROCK` |
| `FORCE_PROMPT_CACHING_5M` | 强制 5 分钟提示缓存 TTL |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | 退出离开摘要/空闲会话回顾。设 `0` 禁用。与 `awaySummaryEnabled` 设置配对（v2.1.110） |
| `DISABLE_PROMPT_CACHING` | 禁用所有提示缓存（`1` 禁用） |
| `DISABLE_PROMPT_CACHING_HAIKU` | 禁用 Haiku 提示缓存 |
| `DISABLE_PROMPT_CACHING_SONNET` | 禁用 Sonnet 提示缓存 |
| `DISABLE_PROMPT_CACHING_OPUS` | 禁用 Opus 提示缓存 |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | **已废弃。** 在 Bedrock 上请求 1 小时缓存 TTL（`1` 启用）。已被 `ENABLE_PROMPT_CACHING_1H` 取代 |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | 禁用实验性 beta 功能（`1` 禁用） |
| `CLAUDE_CODE_SHELL` | 覆盖自动 shell 检测 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | 覆盖默认文件读取 token 上限 |
| `CLAUDE_CODE_GLOB_HIDDEN` | 设为 `false` 让 Claude 调用 Glob 工具时从结果中排除点文件。默认包含。不影响 `@` 文件自动补全、`ls`、Grep 或 Read |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | 设为 `false` 让 Glob 工具尊重 `.gitignore` 模式。默认 Glob 返回所有匹配文件（含 gitignored）。不影响 `@` 文件自动补全——那有独立的 `respectGitignore` 设置 |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Glob 文件发现超时（秒） |
| `CLAUDE_CODE_ENABLE_TASKS` | 控制会话用结构化 Task 工具（`TaskCreate`、`TaskUpdate`、`TaskGet`、`TaskList`）还是旧版 `TodoWrite` 工具。自 v2.1.142 起 Task 工具是所有模式的默认。设 `0` 回退 `TodoWrite` |
| `CLAUDE_CODE_SIMPLE` | 设为 `1` 以最小系统提示且只有 Bash、文件读取和文件编辑工具运行。也可作为仅启动时生效的变量——见 [CLI 启动参数](claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | 空闲多久（ms）后 SDK 模式自动退出 |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | 禁用自适应思考（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_THINKING` | 强制禁用扩展思考（`1` 禁用） |
| `DISABLE_INTERLEAVED_THINKING` | 阻止发送 interleaved-thinking beta 头（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | 通过自动压缩把所有 1M 上下文窗口模型限制在 200K 上下文。自 v2.1.223 起适用于所有 1M 模型（不只一个）。设 `1` 在整个部署中禁用 1M 上下文 |
| `CLAUDE_CODE_ACCOUNT_UUID` | 覆盖认证用账号 UUID |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | 禁用 git 相关系统提示指令 |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | 设 `0` 从系统提示中省略 Claude Code 署名块 |
| `CLAUDE_CODE_NEW_INIT` | 设为 `true` 让 `/init` 运行交互式安装流程。先问生成哪些文件（CLAUDE.md、skill、hook）再探索代码库。不设则 `/init` 自动生成 CLAUDE.md |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | 一个或多个只读插件种子目录的路径，Unix 用 `:`、Windows 用 `;` 分隔。把预置插件打进容器镜像。Claude Code 启动时从这些目录注册市场并使用预缓存插件而无需重新 clone |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | 启用 Claude.ai MCP 服务器 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 设置 effort 等级：`low`、`medium`、`high`、`xhigh`（Opus 4.7 和 4.8，v2.1.111）、`max`（仅 Opus 4.6）或 `auto`（用模型默认）。优先于 `/effort` 和 `effortLevel` 设置。也可作为仅启动时生效的变量——见 [CLI 启动参数](claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_EFFORT` | 只读。注入 Bash 工具子进程和 hook 处理器，携带当前 effort 等级，让 shell 脚本和 hook 适应当前档位（`CLAUDE_CODE_EFFORT_LEVEL` 的伴生变量；v2.1.133）。skill 文件内用 `${CLAUDE_EFFORT}` *（在 changelog，不在官方 env-vars 页——只读，不可由用户配置）* |
| `CLAUDE_CODE_ALWAYS_ENABLE_EFFORT` | 设为 `1` 在所有模型上强制启用 effort 参数，包括通常不支持 effort 等级选择的模型。让 `/effort` 和 `effortLevel` 设置在标准 effort 能力集之外的模型上生效（v2.1.154） |
| `CLAUDE_CODE_MAX_TURNS` | 停止前的最大 agent 轮数 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 等价于同时设置 `DISABLE_AUTOUPDATER`、`DISABLE_FEEDBACK_COMMAND`、`DISABLE_ERROR_REPORTING`、`DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | 跳过首次运行设置安装流程 *（不在官方文档——未验证）* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | 覆盖提示缓存行为 *（不在官方文档——未验证）* |
| `CLAUDE_CODE_DISABLE_TOOLS` | 逗号分隔的要禁用工具列表 *（不在官方文档——未验证）* |
| `CLAUDE_CODE_DISABLE_MCP` | 禁用所有 MCP 服务器（`1` 禁用） *（不在官方文档——未验证）* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | 每次回复最大输出 token。默认 32,000（v2.1.77 起 Opus 4.6 为 64,000）。上限 64,000（v2.1.77 起 Opus 4.6 和 Sonnet 4.6 为 128,000） |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | 完全禁用 fast mode（`1` 禁用） |
| `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` | **v2.1.160 已移除** — 该环境变量现为空操作。Fast mode 无论此变量如何都跑默认模型。此前（v2.1.142–v2.1.159）把 [fast mode](https://code.claude.com/docs/en/fast-mode) 钉定到 Claude Opus 4.6 |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | 设为 `1` 禁用流式请求中途失败时的非流式回退。流式错误改为传播到重试层。适用于代理或网关导致回退产生重复工具执行的场景（v2.1.83） |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | 中止卡顿流的流式空闲看门狗。自 v2.1.163 起默认启用；设 `0` 禁用 |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | 在 Anthropic API 上默认启用（v2.1.139+）；设 `0` 退出 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 禁用自动记忆（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | 禁用 `/rewind` 的文件检查点（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | 禁用附件处理（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | 阻止加载 CLAUDE.md 文件（`1` 禁用） |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | 启动时从 `--add-dir` 指定的附加目录加载 CLAUDE.md 记忆文件（`1` 启用）。也可作为仅启动时生效的变量——见 [CLI 启动参数](claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS` | 跳过从系统级托管 skill 目录加载 skill（`1` 禁用） |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | 上次会话在轮次中途中断时自动续跑（`1` 启用） |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN_MAX_AGE_MS` | 重启时会被自动续跑的中断轮次的最大时长（ms）。超过的跳过。只在 `CLAUDE_CODE_RESUME_INTERRUPTED_TURN=1` 时适用 |
| `CLAUDE_CODE_RESUME_PROMPT` | 自动续跑中断轮次时在开头注入的自定义提示。默认为内置续接消息 |
| `CLAUDE_CODE_SAFE_MODE` | 设为 `1` 启用安全模式，限制潜在破坏性操作并施加更严格的工具防护 |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | 设为 `1` 跳过把提示历史和会话记录写入磁盘。带此变量启动的会话不出现在 `--resume`、`--continue` 或上箭头历史。适合一次性脚本会话 |
| `CLAUDE_CODE_USER_EMAIL` | 同步提供认证用用户邮箱 |
| `CLAUDE_CODE_ORGANIZATION_UUID` | 同步提供认证用组织 UUID |
| `CLAUDE_CONFIG_DIR` | 自定义配置目录（覆盖默认 `~/.claude`） |
| `CLAUDE_CODE_TMPDIR` | 覆盖内部临时文件所用临时目录。Claude Code 在该路径后追加 `/claude/`。默认：Unix/macOS 为 `/tmp`，Windows 为 `os.tmpdir()` |
| `ANTHROPIC_CUSTOM_HEADERS` | API 请求的自定义头（`Name: Value` 格式，多个头换行分隔） |
| `CLAUDE_CODE_EXTRA_BODY` | 合并进每个 API 请求体顶层的 JSON 对象。用于注入厂商专属字段（如自定义网关的路由提示） |
| `CLAUDE_CODE_PROPAGATE_TRACEPARENT` | 设为 `1` 经自定义代理路由时透传 W3C `traceparent` 头，把 Claude Code 追踪链接到你的上游遥测 |
| `ANTHROPIC_FOUNDRY_API_KEY` | Microsoft Foundry 认证用 API key |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry 资源的基础 URL |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Foundry 资源名 |
| `ANTHROPIC_FOUNDRY_AUTH_TOKEN` | Foundry 的 Azure 认证 token（服务主体认证的替代，v2.1.203+） |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock 认证用 API key |
| `ANTHROPIC_SMALL_FAST_MODEL` | **已废弃** — 请改用 `ANTHROPIC_DEFAULT_HAIKU_MODEL` |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | 已废弃 Haiku 级模型覆盖的 AWS 区域 |
| `CLAUDE_CODE_SHELL_PREFIX` | 前置到 bash 命令的命令前缀 |
| `BASH_DEFAULT_TIMEOUT_MS` | bash 命令默认超时（ms） |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | 跳过 Bedrock 的 AWS 认证（`1` 跳过） |
| `CLAUDE_CODE_AWS_CHAIN_RESOLVE_TIMEOUT_MS` | Bedrock 上 AWS 凭据链解析的超时（ms）。凭据来源缺失的环境下调低（如 `5000`）可快速失败。默认 `60000`（60 秒）（v2.1.207） |
| `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_GUARD` | 设为 `1` 禁用拒绝意外 MIME 类型响应的 Bedrock content-type 防护。网关返回非标准 content type 但响应体有效时使用（v2.1.208） |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | 跳过 Foundry 的 Azure 认证（`1` 跳过） |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | 跳过 Bedrock Mantle 的 AWS 认证（如使用 LLM 网关时） |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | 跳过 Vertex 的 Google 认证（`1` 跳过） |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | 允许代理执行 DNS 解析 |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper` 的凭据刷新间隔（ms） |
| `CLAUDE_CODE_CLIENT_CERT` | mTLS 客户端证书路径 |
| `CLAUDE_CODE_CLIENT_KEY` | mTLS 客户端私钥路径 |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | 加密 mTLS 密钥的口令 |
| `CLAUDE_CODE_CERT_STORE` | TLS 连接的 CA 证书来源列表，逗号分隔：`bundled`（随 Claude Code 附带的 Mozilla CA 集）和/或 `system`（操作系统信任库）。默认 `bundled,system`。系统集成需要原生二进制发行版；Node.js 运行时上无论此值如何都只用 bundled 集（v2.1.101） |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | 插件市场 git clone 超时（ms，默认 120000） |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | 设为 `1` 让 GitHub `owner/repo` 简写插件来源通过 HTTPS 而非 SSH clone。适用于插件安装/更新和 `/plugin marketplace add`/`update`。适合没有配置 `github.com` SSH key 的 CI runner 或容器（v2.1.141） |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | 覆盖插件根目录 |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | 跳过自动添加官方市场（`1` 禁用） |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | 等待插件安装完成再首次查询（`1` 启用） |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | 同步插件安装的超时（ms） |
| `CLAUDE_CODE_SYNC_SKILLS` | 设为 `1` 等待 skill 同步完成再首次查询，确保会话开始前 skill 已就绪 |
| `CLAUDE_CODE_SYNC_SKILLS_INSTALL_TIMEOUT_MS` | 同步 skill 同步中安装阶段的超时（ms） |
| `CLAUDE_CODE_SYNC_SKILLS_WAIT_TIMEOUT_MS` | `CLAUDE_CODE_SYNC_SKILLS=1` 时等待全部 skill 同步完成的超时（ms） |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | 设为 `1`，`git pull` 失败时保留现有市场缓存而不是清空重 clone。适合重 clone 也会同样失败的离线或隔离环境 |
| `CLAUDE_CODE_ENABLE_BACKGROUND_PLUGIN_REFRESH` | 后台安装完成后在会话轮次边界刷新插件状态（`1` 启用）。不设置时新装插件在下一个会话生效 |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | 在 UI 中隐藏邮箱/组织信息 *（不在官方文档——未验证）* |
| `CLAUDE_CODE_DISABLE_CRON` | 禁用计划/cron 任务（`1` 禁用） |
| `DISABLE_INSTALLATION_CHECKS` | 禁用安装警告 |
| `DISABLE_FEEDBACK_COMMAND` | 禁用 `/feedback` 命令。也接受旧名 `DISABLE_BUG_COMMAND` |
| `DISABLE_DOCTOR_COMMAND` | 隐藏 `/doctor` 命令（`1` 禁用） |
| `DISABLE_LOGIN_COMMAND` | 隐藏 `/login` 命令（`1` 禁用） |
| `DISABLE_LOGOUT_COMMAND` | 隐藏 `/logout` 命令（`1` 禁用） |
| `DISABLE_UPGRADE_COMMAND` | 隐藏 `/upgrade` 命令（`1` 禁用） |
| `DISABLE_EXTRA_USAGE_COMMAND` | 隐藏 `/extra-usage` 命令——v2.1.144 已改名 `/usage-credits`，但此环境变量名不变（`1` 禁用） |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | 隐藏 `/install-github-app` 命令（`1` 禁用） |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | 禁用花式文本和非必要模型调用 *（不在官方文档——未验证）* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | 覆盖调试日志文件路径。尽管名字如此，这是文件路径不是目录。需另行通过 `--debug`、`/debug` 或 `DEBUG` 环境变量启用调试模式；只设此变量不开启日志。用 `--debug-file` 可同时启用调试模式和设置路径。默认 `~/.claude/debug/<session-id>.txt` |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | 最低调试日志级别 |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | 强制长任务自动转后台（`1` 启用） |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | 阻止把 Opus 4.0/4.1 重映射到更新的模型（`1` 禁用） |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | 对所有主模型而不只是 default 触发回退模型（`1` 启用） |
| `CCR_FORCE_BUNDLE` | 设为 `1` 强制 `claude --remote` 即使可访问 GitHub 也打包并上传本地仓库。也可作为仅启动时生效的变量——见 [CLI 启动参数](claude-cli-startup-flags.md#environment-variables) |
| `CLAUDE_CODE_GIT_BASH_PATH` | 仅 Windows：Git Bash 可执行文件（`bash.exe`）路径。Git Bash 已安装但不在 PATH 时使用 |
| `DISABLE_COST_WARNINGS` | 禁用费用警告消息 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | subagent 默认模型（如 `haiku`、`sonnet`）。**自 v2.1.238 起（破坏性变更）**：从覆盖改为默认——显式设置模型的 agent 定义和传显式模型的 subagent 工具调用优先于此变量 |
| `CLAUDE_CODE_FORWARD_SUBAGENT_TEXT` | 设为 `1` 把 subagent 流式文本实时转发到父会话。默认 subagent 输出缓冲到完成。自 v2.1.219 起，转发也适用于深度 2+ 的嵌套 subagent（v2.1.211） |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | 设为 `1` 从子进程环境（Bash 工具、hook、MCP stdio 服务器）剥除 Anthropic 和云厂商凭据。子进程不应继承 API key 时用于纵深防御（v2.1.83） |
| `CLAUDE_CODE_SCRIPT_CAPS` | JSON 对象，限制 `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` 设置时特定脚本每会话可被调用的次数。键是按命令文本匹配的子串；值是整数调用上限。如 `{"deploy.sh": 2}` 允许 `deploy.sh` 最多调用两次。按子串匹配；经 `xargs` 或 `find -exec` 的运行时分叉不被检测——这是纵深防御控制 |
| `CLAUDE_CODE_PERFORCE_MODE` | 设为 `1` 启用 Perforce 感知的写保护。设置后，目标文件缺少 owner-write 位时（Perforce 在 `p4 edit` 打开前会清除已同步文件的该位），Edit、Write、NotebookEdit 失败并提示 `p4 edit <file>`。防止 Claude Code 绕过 Perforce 变更跟踪（v2.1.98） |
| `CLAUDE_CODE_PROCESS_WRAPPER` | 启动时包裹 Claude Code 进程。指定包装器可执行文件和参数；Claude Code 作为最后一个参数追加。适用于性能分析工具、沙箱或追踪（如 `CLAUDE_CODE_PROCESS_WRAPPER="strace -e trace=file"`） |
| `CLAUDE_CODE_MAX_RETRIES` | 覆盖 API 请求重试次数（默认 10） |
| `CLAUDE_CODE_RETRY_WATCHDOG` | 把非容量类 API 错误的重试数提高到 300。标准重试上限（`CLAUDE_CODE_MAX_RETRIES`，默认 10）对容量错误仍然适用（v2.1.199） |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | 最大并行只读工具数（默认 10） |
| `CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION` | 每会话允许的最大网页搜索数（默认 `200`）。防止长 agent 会话中工具失控。自 v2.1.290 起，配额还按 `CLAUDE_CODE_WEB_SEARCH_REFILLS_PER_HOUR` 速率每小时补充 |
| `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION` | 每会话可生成的最大 subagent 数（默认 `200`）。防止深度嵌套 agent 工作流耗尽资源 |
| `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` | 并发运行的最大 subagent 数（默认 `20`）。限制并行 agent 扇出；超出的 agent 排队等空位（v2.1.217） |
| `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` | subagent 最大嵌套深度（v2.1.219 起默认 `3`）。达到深度上限的 subagent 不能再派生 subagent（v2.1.217） |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | SDK 模式下禁用内置 subagent 类型（`1` 禁用） |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | SDK 模式下跳过 MCP 工具的 `mcp__<server>__` 前缀（`1` 启用） |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` | 后台 subagent 的卡顿超时（ms，默认 600000 / 10 分钟）。超时未产出任何输出则杀死 subagent |
| `MCP_CONNECTION_NONBLOCKING` | `-p` 模式下设 `true` 完全跳过 MCP 连接等待。把 `--mcp-config` 服务器连接限制在 5 秒，而不是被最慢的服务器阻塞 |
| `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` | 慢速 MCP 工具调用自动转后台的毫秒数（默认 `120000` / 2 分钟）。超过阈值的 MCP 调用继续在后台运行而不阻塞 agent 轮次 |
| `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` | MCP 工具调用的空闲超时（ms）。窗口内无流式响应则取消调用。适用于网关或 MCP 服务器静默挂起——与总耗时限制 `MCP_TOOL_TIMEOUT` 不同（v2.1.187） |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hook 超时（ms，取代硬性 1.5 秒限制） |
| `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` | Stop hook 阻止会话结束的最大次数（默认 `8`）。达到后无论 hook 退出码如何会话都结束 |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | 禁用反馈调查提示（`1` 禁用） |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | 设为 `1`，在到 Anthropic 的非必要流量被阻止时，把会话质量调查路由到你自己的 OpenTelemetry collector。调查评分只作为 OTEL 事件发往你配置的 collector——不向 Anthropic 发送任何调查数据。在设置了 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`、`DISABLE_TELEMETRY` 或 `DO_NOT_TRACK` 时适用；否则无效。`CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` 和组织产品反馈策略优先（v2.1.136） |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | 禁用终端标题更新（`1` 禁用） |
| `CLAUDE_CODE_TMUX_TRUECOLOR` | 设为 `1` 允许 tmux 内 24 位真彩色输出。默认 `$TMUX` 设置时 Claude Code 钳制到 256 色，因为 tmux 除非配置否则不透传真彩色转义序列。在 `~/.tmux.conf` 加 `set -ga terminal-overrides ',*:Tc'` 后再设此变量 |
| `CLAUDE_CODE_NO_FLICKER` | 设为 `1` 启用无闪烁备用屏幕渲染。消除全屏重绘的视觉闪烁（v2.1.88） |
| `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT` | 设为 `1` 全屏渲染时每帧重绘整个屏幕。部分重绘在特殊终端模拟器产生视觉伪影时使用 |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | 设为 `1` 禁用全屏渲染，用经典主屏渲染器。对话保留在终端原生回滚区，`Cmd+f` 和 tmux 复制模式照常工作。优先于 `CLAUDE_CODE_NO_FLICKER` 和 `tui` 设置。也可用 `/tui default` 切换（v2.1.132） |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | 设为 `1` 强制启用 DEC 私有模式 2026 同步输出（终端支持但未被自动检测到时）。适用于实现了 BSU/ESU 但不响应能力探测的模拟器，如 Emacs `eat`。tmux 下无效（v2.1.129） |
| `CLAUDE_CODE_SCROLL_SPEED` | 全屏渲染的鼠标滚轮滚动倍数。调大更快，调小更精细 |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | 设为 `1` 禁用全屏渲染的虚拟滚动，渲染记录中每条消息。全屏模式滚动出现空白区域（本应有消息）时使用 |
| `CLAUDE_CODE_DISABLE_MOUSE` | 设为 `1` 禁用全屏渲染的鼠标跟踪。鼠标事件干扰终端复用器或无障碍工具时有用 |
| `CLAUDE_CODE_DISABLE_MOUSE_CLICKS` | 设为 `1` 禁用全屏渲染的鼠标点击交互但保留滚轮。区分点击抑制与完整鼠标跟踪禁用（`CLAUDE_CODE_DISABLE_MOUSE`）。终端选择或外部无障碍工具与应用内点击冲突时有用 |
| `CLAUDE_CODE_HIDE_CWD` | 设为 `1` 在 Claude Code 启动 logo 横幅中隐藏当前工作目录。适合录屏、演示或共享会话中避免 CWD 路径泄露主机或项目布局信息（v2.1.119） |
| `CLAUDE_CODE_ACCESSIBILITY` | 设为 `1` 为屏幕阅读器和无障碍工具保持原生终端光标可见 |
| `CLAUDE_AX_SCREEN_READER` | 设为 `1` 渲染屏幕阅读器友好输出：无装饰边框和动画的纯平文本。设 `0` 即使 `axScreenReader` 设置为 `true` 也强制关闭。`--ax-screen-reader` CLI 参数优先（v2.1.181+） |
| `CLAUDE_CODE_NATIVE_CURSOR` | 设为 `1` 在输入光标位置显示终端自带光标，替代 Claude Code 的自定义光标字符 |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | 设 `0` 禁用 diff 输出中的语法高亮 |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | 跳过自动安装 IDE 扩展（`1` 跳过） |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | 覆盖自动 IDE 连接行为 |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | 覆盖连接用的 IDE 主机地址 |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | 跳过 IDE 锁文件校验（`1` 跳过） |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | OTel 头 helper 脚本的防抖间隔（ms） |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | OpenTelemetry flush 超时（ms） |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | OpenTelemetry 关闭超时（ms） |
| `CLAUDE_ENABLE_BYTE_WATCHDOG` | 设 `1` 强制启用字节级流式空闲看门狗，或 `0` 强制禁用。未设置时，Anthropic API 连接默认启用。字节看门狗在 `CLAUDE_STREAM_IDLE_TIMEOUT_MS`（最少 5 分钟）设定的时间内线上无字节到达时中止连接，独立于事件级看门狗 |
| `CLAUDE_ENABLE_BYTE_WATCHDOG_BEDROCK` | 设 `1` 专为 Bedrock 连接启用字节级流式空闲看门狗。默认字节看门狗在 Bedrock 上不活动。配合 `CLAUDE_STREAM_IDLE_TIMEOUT_MS` 配置超时 |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | 流式空闲看门狗超时（ms）。两个看门狗：**字节级**（默认且最小 `300000` / 5 分钟，线上无字节到达即中止）和**事件级**（默认 `90000` / 90 秒，无下限，无 SSE 事件到达即中止）。字节看门狗在 Anthropic API 连接上默认启用；用 `CLAUDE_ENABLE_BYTE_WATCHDOG` 控制。长时工具或慢网络导致过早超时报错时调高事件超时 |
| `OTEL_LOG_TOOL_DETAILS` | 设 `1` 在 OpenTelemetry 事件中包含 `tool_parameters`。默认出于隐私省略（v2.1.85） |
| `CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH` | OpenTelemetry 事件负载的最大内容长度（字节，默认 `61440` / 60 KB）。在作为 OTel 事件发出前截断过大的工具输入、输出或消息体，防止负载超限 |
| `OTEL_LOG_RAW_API_BODIES` | 设 `1` 把完整 API 请求和响应体作为 OpenTelemetry 日志事件发出。默认出于隐私和负载大小省略。适合在网关或代理处调试 |
| `OTEL_RESOURCE_ATTRIBUTES` | 逗号分隔的 `key=value` 对，作为资源属性加到 Claude Code 发出的所有 OpenTelemetry 指标数据点上。用于附加环境或部署标签（如 `environment=production,team=platform`），出现在每个指标上便于 collector 过滤（v2.1.162） |
| `OTEL_LOG_USER_PROMPTS` | 设 `1` 在 OpenTelemetry LLM 请求 span 中包含 `user_system_prompt` 字段。默认出于隐私省略——用户提示可能含敏感数据，只有在自控 OTel collector 且有策略时才选择加入 |
| `OTEL_LOG_ASSISTANT_RESPONSES` | 设 `1` 在 OpenTelemetry 日志事件中包含模型回复文本。默认出于隐私和负载大小省略。只在自控 OTel collector 且有模型输出处理策略时使用 |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | 指标和日志的 OpenTelemetry collector 端点 URL。见[监控](https://code.claude.com/docs/en/monitoring-usage) |
| `OTEL_EXPORTER_OTLP_HEADERS` | OpenTelemetry 导出器头（`Name=Value` 格式，逗号分隔），用于向 collector 认证 |
| `OTEL_LOG_TOOL_CONTENT` | 设 `1` 把完整工具输入输出作为 OpenTelemetry 日志事件发出。默认出于隐私省略 |
| `OTEL_METRICS_EXPORTER` | OpenTelemetry 指标导出器类型（如 `otlp`）。见[监控](https://code.claude.com/docs/en/monitoring-usage) |
| `OTEL_TRACES_EXPORTER` | OpenTelemetry 追踪导出器类型（如 `otlp`）。见[监控](https://code.claude.com/docs/en/monitoring-usage) |
| `OTEL_METRICS_INCLUDE_ENTRYPOINT` | 设 `1` 把会话入口（交互式 vs `-p` vs SDK）作为标签加到所有 OpenTelemetry 指标数据点。便于按 Claude Code 的调用方式拆分指标（v2.1.161） |
| `OTEL_ATTRIBUTE_VALUE_LENGTH_LIMIT` | OpenTelemetry 属性值的最大长度（字节）。超限截断。用于限制大属性值 span 的内存和负载 |
| `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` | 设 `1` 把已认证账号 UUID 作为标签加到所有指标数据点 |
| `OTEL_METRICS_INCLUDE_SESSION_ID` | 设 `1` 把会话 ID 作为标签加到所有指标数据点 |
| `OTEL_METRICS_INCLUDE_VERSION` | 设 `1` 把 Claude Code 版本作为标签加到所有指标数据点 |
| `OTEL_METRICS_INCLUDE_RESOURCE_ATTRIBUTES` | 设 `1` 把 `OTEL_RESOURCE_ATTRIBUTES` 的键值对提升为每个指标数据点的维度 |
| `CLAUDE_CODE_OTEL_DIAG_STDERR` | 设 `1` 把 OpenTelemetry SDK 诊断输出打到 stderr。用于排查 collector 连通性或导出器配置问题 |
| `CLAUDE_CODE_FORK_SUBAGENT` | 设 `1` 在外部构建（非 Anthropic 签名发行版）上启用分叉 subagent。分叉 subagent 在隔离的子进程运行，不共享主 agent 的上下文 |
| `CLAUDE_CODE_MCP_SERVER_NAME` | MCP 服务器名，作为环境变量传给 `headersHelper` 脚本以便生成服务器专属认证头 *（在 v2.1.85 changelog，尚未进入官方 env-vars 页）* |
| `CLAUDE_CODE_MCP_SERVER_URL` | MCP 服务器 URL，与 `CLAUDE_CODE_MCP_SERVER_NAME` 一起传给 `headersHelper` 脚本 *（在 v2.1.85 changelog，尚未进入官方 env-vars 页）* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 覆盖 Opus 模型别名（如 `claude-opus-4-6[1m]`） |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上钉定模型时，自定义 `/model` 选择器中 Opus 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Opus 条目描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | 覆盖钉定 Opus 模型的能力检测。逗号分隔值（如 `effort,thinking`）。钉定模型支持自动检测无法确认的功能时必填 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 覆盖 Sonnet 模型别名（如 `claude-sonnet-4-6`） |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上钉定模型时，自定义 `/model` 选择器中 Sonnet 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Sonnet 条目描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | 覆盖钉定 Sonnet 模型的能力检测。逗号分隔值（如 `effort,thinking`）。钉定模型支持自动检测无法确认的功能时必填 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL` | 覆盖 Fable 模型别名（如 `claude-fable-5`） |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上钉定模型时，自定义 `/model` 选择器中 Fable 条目标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Fable 条目描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_SUPPORTED_CAPABILITIES` | 覆盖钉定 Fable 模型的能力检测。逗号分隔值（如 `effort,thinking`）。钉定模型支持自动检测无法确认的功能时必填 |
| `MAX_THINKING_TOKENS` | 每次回复的最大扩展思考 token。设 `0` 在 Anthropic API 上完全禁用扩展思考（等价于 `--thinking disabled`）。只在使用固定思考预算时适用——自适应思考模型（Opus 4.7+）上由 effort 等级控制思考深度 |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 设置自动压缩计算所用上下文容量（token）。默认为模型上下文窗口（200K 标准，1M 扩展模型）。在 1M 模型上设较低值（如 `500000`）可按 500K 处理。上限为实际上下文窗口。`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 按此值的百分比应用。设置此值会把压缩阈值与状态行 `used_percentage` 解耦 |
| `DISABLE_AUTO_COMPACT` | 禁用自动上下文压缩（`1` 禁用）。手动 `/compact` 仍可用 |
| `DISABLE_COMPACT` | 禁用所有压缩——自动和手动（`1` 禁用） |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | 启用提示建议 |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | 会话要求 plan mode |
| `CLAUDE_CODE_TEAM_NAME` | agent teams 的团队名 |
| `CLAUDE_CODE_TASK_LIST_ID` | 任务集成的任务列表 ID |
| `CLAUDE_ENV_FILE` | 自定义环境文件路径 |
| `FORCE_AUTOUPDATE_PLUGINS` | 强制插件自动更新（`1` 启用） |
| `HTTP_PROXY` | 网络请求的 HTTP 代理 URL |
| `HTTPS_PROXY` | 网络请求的 HTTPS 代理 URL |
| `NO_PROXY` | 绕过代理的主机列表，逗号分隔 |
| `MCP_TOOL_TIMEOUT` | MCP 工具执行超时（ms） |
| `MCP_CLIENT_SECRET` | MCP OAuth 客户端密钥 |
| `MCP_OAUTH_CALLBACK_PORT` | MCP OAuth 回调端口 |
| `IS_DEMO` | 启用演示模式 |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | 斜杠命令工具输出的字符预算 |
| `TASK_MAX_OUTPUT_LENGTH` | 后台任务结果的最大输出长度（字节）。超限截断 |
| `MAX_STRUCTURED_OUTPUT_RETRIES` | 结构化输出（schema 约束的工具调用）校验失败时的最大重试数。默认 `3` |
| `DEBUG` | 设为 `claude:*` 或其他调试命名空间以启用详细调试日志。多数用途等价于 `--debug` 参数 |
| `FORCE_HYPERLINK` | 设 `1` 强制 OSC 8 超链接渲染，即使终端未显式宣告支持。终端支持超链接但未被自动检测到时使用 |
| `DISABLE_GROWTHBOOK` | 设 `1` 禁用 GrowthBook 特性开关客户端，阻止到特性开关 CDN 的出站调用。适合隔离或防火墙受限环境 |
| `DISABLE_PROMPT_CACHING_FABLE` | 设 `1` 专为 Fable 模型请求禁用提示缓存 |
| `CLAUDE_CODE_FORCE_STRIKETHROUGH` | 设 `1` 强制启用 diff 输出中删除行的删除线渲染，即使终端未确认支持 |
| `CLAUDE_DISABLE_ADOPT` | 设 `1` 阻止 Claude Code 收养可能运行在同一终端的既有 `claude` 进程。用于进程收养会导致意外行为的环境 |
| `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS` | 等待后台任务产出多久（ms）后才打印等待指示。控制 Claude Code 静默多久后才告知用户后台任务仍在运行 |
| `CLAUDE_CODE_TEAM_TEARDOWN_PARK_TIMEOUT_MS` | 停靠队友时 agent team 拆卸的超时（ms）。控制 lead agent 等待队员完成拆卸多久后才继续 |
| `CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT` | 以 `--simple` / `CLAUDE_CODE_SIMPLE=1` 模式运行时注入的自定义系统提示 |
| `CLAUDE_CODE_ENABLE_APPEND_SUBAGENT_PROMPT` | 设 `1` 把父级系统提示追加到每个 subagent 的系统提示。未设置时 subagent 只收到自己的提示 |
| `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK` | 设 `1` 跳过判断 fast mode 可用性的组织级检查。组织状态端点不可达且希望 fast mode 始终视为可用的环境使用 |
| `CLAUDE_CODE_USE_ANTHROPIC_AWS` | 设 `1` 用 Claude Platform on AWS 发 API 请求。需要 `ANTHROPIC_AWS_API_KEY` 和 `ANTHROPIC_AWS_WORKSPACE_ID` |
| `CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH` | 设 `1` 跳过 Claude Platform on AWS 的认证。凭据由外部注入的环境使用 |
| `CLAUDE_CODE_SKIP_AWS_CRED_CACHE` | 设 `1` 禁用 AWS 凭据缓存。每次请求强制全新解析凭据。凭据频繁轮换时有用 |
| `CLAUDE_CODE_USE_NATIVE_FILE_SEARCH` | 设 `1` 让 Glob 工具用操作系统原生文件搜索——macOS（Spotlight/mdfind）和 Windows（Windows Search）——替代内置搜索。配置了原生索引的大型仓库上可能更快 |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Claude 3.5 Haiku 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_3_5_SONNET` | Claude 3.5 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Claude 3.7 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Claude 4.0 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Claude 4.0 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Claude 4.1 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_5_OPUS` | Claude 4.5 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_5_SONNET` | Claude 4.5 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_6_OPUS` | Claude 4.6 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_6_SONNET` | Claude 4.6 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_7_OPUS` | Claude 4.7 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_8_OPUS` | Claude 4.8 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_5_OPUS` | Claude 5 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_5_SONNET` | Claude 5 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_FABLE_5` | Claude Fable 5 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_HAIKU_4_5` | Claude Haiku 4.5 的 Vertex AI 区域覆盖 |
| `CLAUDE_BYTE_STREAM_IDLE_TIMEOUT_MS` | 字节级流式空闲看门狗中止卡顿连接前的超时（ms）。`CLAUDE_ENABLE_BYTE_WATCHDOG` 活动时适用。最小 300,000（5 分钟） |
| `CLAUDE_CODE_DISABLE_ADMIN_ENV_UNION` | 设 `1` 禁用 `env` 在多个管理员托管来源之间的按键联合行为。设置后只用优先级最高的托管来源的 `env` 块（v2.1.223） |
| `CLAUDE_CODE_ENABLE_OPUS_4_7_FAST_MODE` | **v2.1.142 已移除。** 此前在 Opus 4.7 上启用 fast mode。该环境变量现为空操作 |
| `MCP_DISCOVERY_CACHE` | MCP 服务器发现结果的本地缓存文件路径。设置后 Claude Code 启动时从该文件读取发现结果而不是查询发现端点，降低多 MCP 服务器环境的延迟 |
| `USE_BUILTIN_RIPGREP` | 设 `0` 让 Grep 工具用系统 `rg` 二进制而不是 Claude Code 自带的 ripgrep。默认用自带二进制。也可作为仅启动时生效的变量——见 [CLI 启动参数](claude-cli-startup-flags.md#environment-variables) |
| `ANTHROPIC_BEDROCK_REGION_PREFIX` | Bedrock 跨区域推理 profile ID 的区域前缀。设置后 Claude Code 自动把该值前置于 Bedrock 模型 ID 以构造跨区域推理 profile ARN（v2.1.224） |
| `ANTHROPIC_DEFAULT_MODEL` | 任何设置文件都没设 `model` 键时应用的默认模型名。与总是覆盖模型的 `ANTHROPIC_MODEL` 不同，此变量只是没有设置文件指定模型时的最后回退（v2.1.236+） |
| `CLAUDE_CODE_PROJECT_DIR_NAME` | 会话记录目录名中使用的当前项目短显示名。设置后覆盖自动推导的项目名（通常是仓库或目录名）。多个项目共用目录名时有用（v2.1.234+） |
| `CLAUDE_CODE_TOOL_MEMORY_LIMIT` | Linux 上沙箱工具进程的内存限制（cgroup 强制）。防止失控构建耗尽全部内存。接受字节数或人类可读大小（如 `"2G"`）。只在 Linux 沙箱活动时强制（v2.1.233） |
| `CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS` | WebFetch 响应的缓存 TTL（ms）。默认 `900000`（15 分钟）。设置后抓取页面缓存该时长。设 `0` 保持默认——**不**禁用缓存（v2.1.233） |
| `CLAUDE_CODE_ENABLE_TODO_TOOLS` | 设 `1` 启用旧版 `TodoWrite`/`TodoRead`/`TodoDone` 工具，作为默认 `TaskCreate`/`TaskUpdate`/`TaskGet` 任务管理工具的替代（v2.1.234） |
| `CLAUDE_CODE_WORKFLOW_PREFIX_STAGGER_MS` | 动态 workflow 中启动同前缀 agent 的间隔延迟（ms）。多个 agent 共享同一标签前缀时，此延迟防止同时启动压垮下游服务。默认 `0`（无间隔）（v2.1.229） |
| `CLAUDE_CODE_DISABLE_STRUCTURED_OUTPUTS` | 设 `1` 禁用模型响应的结构化输出格式。可改善与不支持结构化输出功能的旧模型版本或自定义 API 代理的兼容性（v2.1.288） |
| `CLAUDE_CODE_WEB_SEARCH_REFILLS_PER_HOUR` | 每小时 WebSearch 配额补充次数（默认 `100`）。控制可补充配额的回充速率。需要 v2.1.290+ |
| `CLAUDE_CODE_OVERLOADED_RETRY_BASE_DELAY_MS` | API 返回过载错误时首次重试的基础延迟（ms）。后续重试以此为基础指数退避（v2.1.292） |
| `CLAUDE_CODE_OVERLOADED_RETRY_MAX_DELAY_MS` | 过载错误重试的最大延迟上限（ms）。防止指数退避无限增长（v2.1.296） |
| `CLAUDE_CODE_RETRY_WATCHDOG_MAX_WAIT_MS` | 重试看门狗放弃卡顿请求前的最大总等待时间（ms）。防止无限挂起的重试（v2.1.295） |
| `CLAUDE_CODE_WORKFLOW_SUBAGENT_MODEL` | 专为 workflow subagent 覆盖模型。对动态 workflow 中启动的 agent 优先于 `CLAUDE_CODE_SUBAGENT_MODEL`。接受模型别名或完整 ID（v2.1.296） |
| `MCP_PROTOCOL_NEGOTIATION` | 控制 MCP 协议版本协商。设 `"legacy"` 让 stdio MCP 服务器和连接器退出 2026-07-28 协议版本（自 v2.1.292 起默认协商）（v2.1.292） |

---

## 常用命令

| 命令 | 说明 |
|---------|-------------|
| `/model` | 切换模型并调整 effort 等级。保存为持久默认；按 `s` 只对当前会话生效 |
| `/effort` | 直接设置 effort 等级：`low`、`medium`、`high`、`xhigh`（Fable 5、Opus 5、Sonnet 5、Opus 4.7、4.8）、`max`（仅限单会话；所有支持 effort 的模型可用）、`ultracode`（仅限单会话；启用 ultracode workflow，v2.1.203+）（v2.1.76+） |
| `/config` | 交互式配置 UI；也接受 `key=value` 语法做提示式设置：`/config model=sonnet`（v2.1.181） |
| `/autocompact` | 设置自动压缩窗口大小（token）。把 `autoCompactWindow` 写入用户设置。用 `--autocompact <tokens>` CLI 参数可按会话达到同样效果（v2.1.221+） |
| `/memory` | 查看/编辑所有记忆文件 |
| `/agents` | 管理 subagent |
| `/mcp` | 管理 MCP 服务器 |
| `/hooks` | 查看已配置的 hook |
| `/plugin` | 管理插件 |
| `claude plugin tag` | 在市场中为分发标记插件版本。在市场仓库中运行，带插件名和版本号（v2.1.118） |
| `claude plugin prune` | 移除市场来源已不存在的插件（如市场被删或 `extraKnownMarketplaces` 条目被移除）。清理本地缓存并禁用孤儿插件（v2.1.121） |
| `claude plugin details <plugin>` | 显示插件的组件清单（命令、agent、skill、hook）及其增加的每会话上下文 token 成本。适合在托管环境启用插件前审计 token 开销（v2.1.139） |
| `/keybindings` | 配置自定义键盘快捷键 |
| `/skills` | 查看和管理 skill |
| `/permissions` | 查看和管理权限规则 |
| `/usage-credits` | 查看剩余用量配额和限制。v2.1.144 从 `/extra-usage` 改名（旧名仍可用） |
| `claude gateway` | 管理组织托管部署的 Claude Gateway 连接。需要托管设置中的 `forceLoginMethod: "gateway"`（v2.1.195） |
| `claude auto-mode reset` | 重置当前会话的 auto-mode 分类。弹确认；传 `--yes` 跳过（v2.1.212） |
| `/fork` | 把当前会话上下文分叉到新的隔离 subagent 会话（v2.1.212） |
| `/subtask` | 在独立上下文中启动隔离子任务。子任务独立运行，完成后返回结果（v2.1.212） |
| `--doctor` | 诊断配置问题 |
| `--debug` | 调试模式，含 hook 执行细节 |

---

## 速查：完整示例

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "advisorModel": "opus",
  "agent": "code-reviewer",
  "language": "english",
  "cleanupPeriodDays": 30,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "viewMode": "default",
  "tui": "fullscreen",
  "awaySummaryEnabled": false,
  "includeGitInstructions": true,
  "defaultShell": "bash",
  "plansDirectory": "./plans",
  "claudeMdExcludes": ["**/vendor/**/CLAUDE.md"],
  "effortLevel": "high",
  "autoCompactWindow": 800000,
  "skillListingMaxDescChars": 1536,
  "skillListingBudgetFraction": 0.01,
  "disableAgentView": false,
  "disableWorkflows": false,
  "workflowKeywordTriggerEnabled": true,
  "workflowSizeGuideline": "medium",
  "syntaxHighlightingDisabled": false,
  "emojiCompletionEnabled": true,

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"],
    "baseRef": "fresh",
    "bgIsolation": "worktree"
  },

  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "classifyAllShell": false,
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ],
    "soft_deny": ["$defaults", "Never run terraform apply"],
    "hard_deny": ["Never run rm -rf on directories outside the project"]
  },

  "permissions": {
    "allow": [
      "Edit(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__memory__*",
      "Agent(*)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../shared/"],
    "defaultMode": "acceptEdits"
  },

  "enableAllProjectMcpServers": true,

  "mcpServers": {
    "always-on-server": {
      "type": "http",
      "url": "https://mcp.example.com",
      "alwaysLoad": true
    }
  },

  "sshConfigs": [
    {
      "id": "dev-vm",
      "name": "Dev VM",
      "sshHost": "user@dev.example.com"
    }
  ],

  "sandbox": {
    "enabled": true,
    "excludedCommands": ["git", "docker"],
    "filesystem": {
      "denyRead": ["./secrets/"],
      "denyWrite": ["./.env"]
    }
  },

  "attribution": {
    "commit": "Generated with Claude Code",
    "pr": ""
  },
  "prUrlTemplate": "https://gitlab.example.com/{owner}/{repo}/-/merge_requests/{number}",

  "statusLine": {
    "type": "command",
    "command": "git branch --show-current"
  },

  "spinnerTipsEnabled": true,
  "spinnerTipsOverride": {
    "tips": ["Custom tip 1", "Custom tip 2"],
    "excludeDefault": false
  },
  "prefersReducedMotion": false,
  "preferredNotifChannel": "terminal_bell",

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium",
    "ANTHROPIC_BEDROCK_SERVICE_TIER": "priority"
  }
}
```

---

## 来源

- [Claude Code Settings Reference](https://code.claude.com/docs/en/settings-reference) — 权威键索引（约 180 个键，含 Scope/Type/Default 列）
- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Model Configuration](https://code.claude.com/docs/en/model-config)
- [Claude Code Status Line Reference](https://code.claude.com/docs/en/statusline)
- [Claude Code Environment Variables](https://code.claude.com/docs/en/env-vars)
- [Claude Code Settings JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub Settings Examples](https://github.com/feiskyer/claude-code-settings)
- [Claude Code Permissions Reference](https://code.claude.com/docs/en/permissions)
- [Claude Code Sandbox Reference](https://code.claude.com/docs/en/sandboxing)








