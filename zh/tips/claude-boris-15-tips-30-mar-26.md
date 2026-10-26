# Claude Code 的 15 个隐藏与被低估的功能——来自 Boris Cherny

> 译自 [tips/claude-boris-15-tips-30-mar-26.md](../../tips/claude-boris-15-tips-30-mar-26.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 创造者 Boris Cherny（[@bcherny](https://x.com/bcherny)）于 2026 年 3 月 30 日分享的技巧整理。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 背景

Boris 分享了一大批他最喜欢的 Claude Code 隐藏功能和被低估的功能，重点是他用得最多的那些。

<a href="https://x.com/bcherny/status/2038454336355999749"><img src="../../tips/assets/boris-26-3-30/0.png" alt="Boris Cherny 开场推文" width="50%" /></a>

---

## 1/ Claude Code 有手机 App

你知道 Claude Code 有手机 App 吗？Boris 的大量代码是在 iOS App 上写的——不用打开笔记本电脑就能改代码，非常方便。

- 下载 iOS/Android 版 Claude App
- 进入左侧的 **Code** 标签页
- 可以直接在手机上审查改动、批准 PR、编写代码

<a href="https://x.com/bcherny/status/2038454337811386436"><img src="../../tips/assets/boris-26-3-30/1.png" alt="Claude Code 手机 App" width="50%" /></a>

---

## 2/ 在手机/Web/桌面与终端之间迁移会话

运行 `claude --teleport` 或 `/teleport`，可以把云端会话接到你的机器上继续。或者运行 `/remote-control`，用手机/Web 控制本地正在运行的会话。

- **Teleport**：把云端会话拉到本地终端
- **Remote Control**：让你从任何设备控制本地会话
- Boris 在他的 `/config` 里开启了 **"Enable Remote Control for all sessions"**（为所有会话启用远程控制）

<a href="https://x.com/bcherny/status/2038454339933548804"><img src="../../tips/assets/boris-26-3-30/2.png" alt="Teleport 与 Remote Control" width="50%" /></a>

---

## 3/ /loop 和 /schedule——两个最强大的功能

用它们可以让 Claude 按固定间隔自动运行，单次最长可持续一周。Boris 在本地跑着一堆 loop：

- `/loop 5m /babysit` —— 自动处理 code review、自动 rebase，并把 PR 护送到生产环境
- `/loop 30m /slack-feedback` —— 每 30 分钟自动为 Slack 反馈创建 PR
- `/loop /post-merge-sweeper` —— 创建 PR 来处理他漏掉的 code review 评论
- `/loop 1h /pr-pruner` —— 关闭过期和不再需要的 PR
- ……还有很多！

试着把各种 workflow（工作流）改造成 Skill 加 loop 的组合，非常强大。

<a href="https://x.com/bcherny/status/2038454341884154269"><img src="../../tips/assets/boris-26-3-30/3.png" alt="/loop 与 /schedule" width="50%" /></a>

---

## 4/ 用 hook（钩子）确定性地执行逻辑

用 hook 在 Agent 生命周期中执行逻辑。例如：

- 每次启动 Claude 时**动态加载**上下文（`SessionStart`）
- **记录模型运行的每一条 bash 命令**（`PreToolUse`）
- 把权限提示**转发到 WhatsApp**，由你批准或拒绝（`PermissionRequest`）
- 每当 Claude 停下来就**推它一把**让它继续（`Stop`）

<a href="https://x.com/bcherny/status/2038454343519932844"><img src="../../tips/assets/boris-26-3-30/4.png" alt="使用 hook" width="50%" /></a>

---

## 5/ Cowork Dispatch

Boris 每天都用 Dispatch 来跟进 Slack 和邮件、管理文件，并在人不在电脑前时操作笔记本电脑。不写代码的时候，他就在 dispatch。

- Dispatch 是 Claude Desktop 应用的**安全远程控制**
- 经你许可后，它可以使用你的 MCP、浏览器和电脑
- 可以把它理解为：从任何地方把非编码任务委托给 Claude

<a href="https://x.com/bcherny/status/2038454345419936040"><img src="../../tips/assets/boris-26-3-30/5.png" alt="Cowork Dispatch" width="50%" /></a>

---

## 6/ 做前端工作时使用 Chrome 扩展

使用 Claude Code 最重要的一条技巧：**给 Claude 一种验证自己输出的方式。**做到这一点后，Claude 会不断迭代，直到结果足够好。

- 可以类比为：让某人建网站却不许他用浏览器——结果大概率不好看
- 给 Claude 一个浏览器，它就会一边写代码一边迭代，直到页面好看为止
- Boris 每次写 Web 代码都会用 Chrome 扩展——它往往比其他类似的 MCP 更稳定可靠

<a href="https://x.com/bcherny/status/2038454347156398333"><img src="../../tips/assets/boris-26-3-30/6.png" alt="用于前端的 Chrome 扩展" width="50%" /></a>

---

## 7/ 用 Claude 桌面应用自动启动并测试 Web 服务器

同理，桌面应用内置了让 Claude **自动运行你的 Web 服务器、甚至在内置浏览器中测试它**的能力。

- 在 CLI 或 VSCode 中可以用 Chrome 扩展搭出类似的用法
- 或者直接用桌面应用获得一体化体验

<a href="https://x.com/bcherny/status/2038454348804714642"><img src="../../tips/assets/boris-26-3-30/7.png" alt="桌面应用测试 Web 服务器" width="50%" /></a>

---

## 8/ Fork 你的会话

经常有人问怎么 fork 一个已有会话。两种方式：

1. 在会话里运行 `/branch`
2. 在 CLI 里运行 `claude --resume <session-id> --fork-session`

`/branch` 会创建一个分支对话——你随即进入该分支。要恢复原会话，使用 `claude -r <original-session-id>`。

<a href="https://x.com/bcherny/status/2038454350214041740"><img src="../../tips/assets/boris-26-3-30/8.png" alt="Fork 你的会话" width="50%" /></a>

---

## 9/ 用 /btw 处理穿插提问

Boris 经常在 Agent 工作时用它回答一些小问题。`/btw` 让你提出一个穿插问题，而不打断 Agent 当前任务。

示例：
```
/btw how do I spell dachshund?
> dachshund — German for "badger dog" (dachs + badger, hund + dog).
↑/↓ to scroll · Space, Enter, or Escape to dismiss
```

<a href="https://x.com/bcherny/status/2038454351849787485"><img src="../../tips/assets/boris-26-3-30/9.png" alt="/btw 处理穿插提问" width="50%" /></a>

---

## 10/ 使用 Git worktree

Claude Code 内置了对 git worktree（工作树）的深度支持。要在同一个仓库里做大量并行工作，worktree 是必不可少的。Boris **随时都有几十个 Claude 在跑**，靠的就是这个。

- 用 `claude -w` 在一个 worktree 中启动新会话
- 或在 Claude 桌面应用中勾选 **"worktree"** 复选框
- 对于使用非 git 版本控制的用户，可以用 `WorktreeCreate` hook 加入自己的 worktree 创建逻辑

<a href="https://x.com/bcherny/status/2038454353787519164"><img src="../../tips/assets/boris-26-3-30/10.png" alt="Git worktree" width="50%" /></a>

---

## 11/ 用 /batch 扇出超大规模变更集

`/batch` 会先向你提问，然后让 Claude 把工作扇出到任意数量的 **worktree Agent**（几十个、几百个、甚至几千个）来完成。

- 适合大型代码迁移和其他可并行化的工作
- 每个 worktree Agent 在自己的代码库副本上独立工作

<a href="https://x.com/bcherny/status/2038454355469484142"><img src="../../tips/assets/boris-26-3-30/11.png" alt="/batch 应对超大规模变更集" width="50%" /></a>

---

## 12/ 用 --bare 将 SDK 启动速度提升至多 10 倍

默认情况下，运行 `claude -p`（或 TypeScript/Python SDK）时，Claude 会搜索本地的 CLAUDE.md、settings 和 MCP。但对非交互式使用，大多数时候你都想通过 `--system-prompt`、`--mcp-config`、`--settings` 等显式指定要加载的内容。

- 这是 SDK 最初构建时的一个设计疏漏
- 未来版本会把默认值翻转为 `--bare`
- 目前可以用这个标志主动开启，获得**最多 10 倍的启动加速**

```bash
claude -p "summarize this codebase" \
    --output-format=stream-json \
    --verbose \
    --bare
```

<a href="https://x.com/bcherny/status/2038454357088457168"><img src="../../tips/assets/boris-26-3-30/12.png" alt="--bare 标志加速 SDK 启动" width="50%" /></a>

---

## 13/ 用 --add-dir 让 Claude 访问更多文件夹

跨多个仓库工作时，Boris 通常在一个仓库里启动 Claude，再用 `--add-dir`（或 `/add-dir`）让 Claude 看到另一个仓库。

- 这不只是告诉 Claude 这个仓库的存在，还会**授予它**在该仓库中工作的**权限**
- 也可以在团队的 `settings.json` 中加入 `"additionalDirectories"`，让 Claude Code 启动时总是加载额外的文件夹

<a href="https://x.com/bcherny/status/2038454359047156203"><img src="../../tips/assets/boris-26-3-30/13.png" alt="--add-dir 跨仓库使用" width="50%" /></a>

---

## 14/ 用 --agent 为 Claude Code 定制系统提示词与工具

自定义 Agent 是一个常被忽视的强大原语。用法很简单：在 `.claude/agents/` 里定义一个新 Agent，然后运行：

```bash
claude --agent=<your agent's name>
```

- Agent 可以有受限的工具集、自定义描述和指定模型
- 非常适合创建只读 Agent、专门的审查 Agent 或领域专用工具

<a href="https://x.com/bcherny/status/2038454360418787764"><img src="../../tips/assets/boris-26-3-30/14.png" alt="--agent 定制系统提示词" width="50%" /></a>

---

## 15/ 用 /voice 开启语音输入

趣闻：Boris 的大部分编码是靠对 Claude 说话完成的，而不是打字。

- 在 CLI 中运行 `/voice`，然后按住空格键说话
- 在桌面端按语音按钮
- 或在 iOS 设置中开启听写

<a href="https://x.com/bcherny/status/2038454362226467112"><img src="../../tips/assets/boris-26-3-30/15.png" alt="/voice 语音输入" width="50%" /></a>

---

## 来源

- [Boris Cherny (@bcherny) on X — 2026 年 3 月 30 日](https://x.com/bcherny/status/2038454336355999749)
