# CLI 启动标志最佳实践

> 译自 [best-practice/claude-cli-startup-flags.md](../../best-practice/claude-cli-startup-flags.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。
> 原文最后更新 2026-03-02。

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

从终端启动 Claude Code 时可用的启动标志（flag）、顶层子命令和启动环境变量参考。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 目录

1. [会话管理](#会话管理)
2. [模型与配置](#模型与配置)
3. [权限与安全](#权限与安全)
4. [输出与格式](#输出与格式)
5. [系统提示词](#系统提示词)
6. [Agent 与 Subagent](#agent-与-subagent)
7. [MCP 与 Plugin](#mcp-与-plugin)
8. [目录与工作区](#目录与工作区)
9. [预算与限制](#预算与限制)
10. [集成](#集成)
11. [初始化与维护](#初始化与维护)
12. [调试与诊断](#调试与诊断)
13. [设置覆盖](#设置覆盖)
14. [版本与帮助](#版本与帮助)
15. [子命令](#子命令)
16. [环境变量](#环境变量)

---

## 会话管理

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--continue` | `-c` | 继续当前目录中最近一次对话 |
| `--resume` | `-r` | 按 ID 或名称恢复指定会话，或显示交互式选择器 |
| `--from-pr <NUMBER\|URL>` | | 恢复与指定 GitHub PR 关联的会话 |
| `--fork-session` | | 恢复时创建新的会话 ID（与 `--resume` 或 `--continue` 配合使用） |
| `--session-id <UUID>` | | 使用指定的会话 ID（必须是有效的 UUID） |
| `--no-session-persistence` | | 禁用会话持久化（仅 print 模式） |
| `--remote` | | 在 claude.ai 上创建新的 Web 会话 |
| `--teleport` | | 在本地终端恢复 Web 会话 |

---

## 模型与配置

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--model <NAME>` | | 通过别名（`sonnet`、`opus`、`haiku`）或完整模型 ID 设置模型 |
| `--fallback-model <NAME>` | | 默认模型过载时的自动回退模型（仅 print 模式） |
| `--betas <LIST>` | | API 请求中包含的 beta 请求头（仅限 API key 用户） |

---

## 权限与安全

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--dangerously-skip-permissions` | | 跳过所有权限提示。务必极其谨慎地使用 |
| `--allow-dangerously-skip-permissions` | | 将跳过权限启用为可选项，但不激活 |
| `--permission-mode <MODE>` | | 以指定的权限模式启动：`default`、`plan`、`acceptEdits`、`bypassPermissions` |
| `--allowedTools <TOOLS>` | | 无需提示即可执行的工具（权限规则语法） |
| `--disallowedTools <TOOLS>` | | 从模型上下文中完全移除的工具 |
| `--tools <TOOLS>` | | 限制 Claude 可使用的内置工具（用 `""` 禁用全部） |
| `--permission-prompt-tool <TOOL>` | | 指定在非交互模式下处理权限提示的 MCP 工具 |

---

## 输出与格式

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--print` | `-p` | 输出响应而不进入交互模式（headless/SDK 模式） |
| `--output-format <FORMAT>` | | 输出格式：`text`、`json`、`stream-json` |
| `--input-format <FORMAT>` | | 输入格式：`text`、`stream-json` |
| `--json-schema <SCHEMA>` | | 获取符合 schema 的已校验 JSON（仅 print 模式） |
| `--include-partial-messages` | | 包含部分流式事件（需要 `--print` 和 `--output-format=stream-json`） |
| `--verbose` | | 启用详细日志，输出完整的逐轮内容 |

---

## 系统提示词

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--system-prompt <TEXT>` | | 用自定义文本替换整个系统提示词 |
| `--system-prompt-file <PATH>` | | 从文件加载系统提示词并替换默认值（仅 print 模式） |
| `--append-system-prompt <TEXT>` | | 在默认系统提示词后追加自定义文本 |
| `--append-system-prompt-file <PATH>` | | 将文件内容追加到默认提示词（仅 print 模式） |

---

## Agent 与 Subagent

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--agent <NAME>` | | 为当前会话指定一个 agent（代理） |
| `--agents <JSON>` | | 通过 JSON 动态定义自定义 subagent |
| `--teammate-mode <MODE>` | | 设置 agent 团队的显示方式：`auto`、`in-process`、`tmux` |

---

## MCP 与 Plugin

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--mcp-config <PATH\|JSON>` | | 从 JSON 文件或字符串加载 MCP 服务器 |
| `--strict-mcp-config` | | 只使用来自 `--mcp-config` 的 MCP 服务器，忽略其他所有服务器 |
| `--plugin-dir <PATH>` | | 仅为本会话从指定目录加载 plugin（可重复使用） |

---

## 目录与工作区

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--add-dir <PATH>` | | 为 Claude 添加可访问的额外工作目录 |
| `--worktree` | `-w` | 在隔离的 git worktree（工作树）中启动 Claude（从 HEAD 分出） |

---

## 预算与限制

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--max-budget-usd <AMOUNT>` | | API 调用花费达到该美元金额上限后停止（仅 print 模式） |
| `--max-turns <NUMBER>` | | 限制 agentic 轮数（仅 print 模式） |

---

## 集成

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--chrome` | | 启用 Chrome 浏览器集成以进行 Web 自动化 |
| `--no-chrome` | | 为本会话禁用 Chrome 浏览器集成 |
| `--ide` | | 启动时自动连接 IDE（仅在恰好有一个有效 IDE 时） |

---

## 初始化与维护

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--init` | | 运行初始化 hook 并启动交互模式 |
| `--init-only` | | 运行初始化 hook 后退出（不进入交互会话） |
| `--maintenance` | | 运行维护 hook 后退出 |

---

## 调试与诊断

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--debug <CATEGORIES>` | | 启用调试模式，可按类别过滤（例如 `"api,hooks"`） |

---

## 设置覆盖

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--settings <PATH\|JSON>` | | 要加载的设置 JSON 文件路径或 JSON 字符串 |
| `--setting-sources <LIST>` | | 逗号分隔的加载来源列表：`user`、`project`、`local` |
| `--disable-slash-commands` | | 为本会话禁用所有 skill 和 slash command |

---

## 版本与帮助

| 标志 | 简写 | 说明 |
|------|-------|-------------|
| `--version` | `-v` | 输出版本号 |
| `--help` | `-h` | 显示帮助信息 |

---

## 子命令

这些是按 `claude <subcommand>` 形式运行的顶层命令：

| 子命令 | 说明 |
|------------|-------------|
| `claude` | 启动交互式 REPL |
| `claude "query"` | 以初始提示词启动 REPL |
| `claude agents` | 列出已配置的 agent |
| `claude auth` | 管理 Claude Code 身份认证 |
| `claude doctor` | 从命令行运行诊断 |
| `claude install` | 安装或切换 Claude Code 原生构建 |
| `claude mcp` | 配置 MCP 服务器（`add`、`remove`、`list`、`get`、`enable`） |
| `claude plugin` | 管理 Claude Code plugin |
| `claude remote-control` | 管理远程控制会话 |
| `claude setup-token` | 为订阅使用创建长期有效的 token |
| `claude update` / `claude upgrade` | 更新到最新版本 |

---

## 环境变量

以下环境变量仅在启动时生效，需在启动 Claude Code 前于 shell 中设置（无法通过 `settings.json` 配置）：

| 变量 | 说明 |
|----------|-------------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 启用实验性 agent 团队。也可通过环境变量配置——参见[设置参考](../../best-practice/claude-settings.md#environment-variables) |
| `CLAUDE_CODE_TMPDIR` | 覆盖内部文件使用的临时目录。也可通过 `env` 键配置——参见[设置参考](../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | 启用额外目录 CLAUDE.md 的加载。也可通过 `env` 键配置——参见[设置参考](../../best-practice/claude-settings.md#environment-variables-via-env) |
| `DISABLE_AUTOUPDATER=1` | 禁用自动更新。也可通过 `env` 键配置——参见[设置参考](../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_CODE_EFFORT_LEVEL` | 控制思考深度——参见[设置参考](../../best-practice/claude-settings.md#environment-variables-via-env) |
| `USE_BUILTIN_RIPGREP=0` | 使用系统 ripgrep 替代内置版本（Alpine Linux） |
| `CLAUDE_CODE_SIMPLE` | 启用简单模式（仅 Bash + Edit 工具）。也可通过 `env` 键配置——参见[设置参考](../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_BASH_NO_LOGIN=1` | 为 BashTool 跳过登录 shell |
| `CCR_FORCE_BUNDLE=1` | 使用 `claude --remote` 时强制打包/上传本地仓库。也可通过 `env` 键配置——参见[设置参考](../../best-practice/claude-settings.md#environment-variables-via-env) |

对于可通过 `settings.json` 中 `"env"` 键配置的环境变量（包括 `MAX_THINKING_TOKENS`、`CLAUDE_CODE_SHELL`、`CLAUDE_CODE_ENABLE_TASKS`、`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`、`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` 等），参见 [Claude 设置参考](../../best-practice/claude-settings.md#environment-variables-via-env)。

---

## 参考来源

- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Headless Mode](https://code.claude.com/docs/en/headless)
- [Claude Code Setup](https://code.claude.com/docs/en/setup)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code Common Workflows](https://code.claude.com/docs/en/common-workflows)
