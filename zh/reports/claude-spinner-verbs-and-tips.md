# Claude Code：Spinner（加载动画）里的单词与小贴士

> 译自 [reports/claude-spinner-verbs-and-tips.md](../../reports/claude-spinner-verbs-and-tips.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

Claude Code 的 spinner 下方轮换显示的单词与小贴士，提取自 `~/.local/share/claude/versions/2.1.121`。单词是产品原始输出的英文文字游戏，保留原文；小贴士文本译为中文，命令与配置项保持原样。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Spinner 单词

| # | 动词 | # | 动词 | # | 动词 | # | 动词 |
|---:|---|---:|---|---:|---|---:|---|
| 1 | Accomplishing | 48 | Discombobulating | 95 | Levitating | 142 | Sketching |
| 2 | Actioning | 49 | Doing | 96 | Lollygagging | 143 | Slithering |
| 3 | Actualizing | 50 | Doodling | 97 | Manifesting | 144 | Smooshing |
| 4 | Architecting | 51 | Drizzling | 98 | Marinating | 145 | Sock-hopping |
| 5 | Baking | 52 | Ebbing | 99 | Meandering | 146 | Spelunking |
| 6 | Beaming | 53 | Effecting | 100 | Metamorphosing | 147 | Spinning |
| 7 | Beboppin' | 54 | Elucidating | 101 | Misting | 148 | Sprouting |
| 8 | Befuddling | 55 | Embellishing | 102 | Moonwalking | 149 | Stewing |
| 9 | Billowing | 56 | Enchanting | 103 | Moseying | 150 | Sublimating |
| 10 | Blanching | 57 | Envisioning | 104 | Mulling | 151 | Swirling |
| 11 | Bloviating | 58 | Evaporating | 105 | Mustering | 152 | Swooping |
| 12 | Boogieing | 59 | Fermenting | 106 | Musing | 153 | Symbioting |
| 13 | Boondoggling | 60 | Fiddle-faddling | 107 | Nebulizing | 154 | Synthesizing |
| 14 | Booping | 61 | Finagling | 108 | Nesting | 155 | Tempering |
| 15 | Bootstrapping | 62 | Flambéing | 109 | Newspapering | 156 | Thinking |
| 16 | Brewing | 63 | Flibbertigibbeting | 110 | Noodling | 157 | Thundering |
| 17 | Bunning | 64 | Flowing | 111 | Nucleating | 158 | Tinkering |
| 18 | Burrowing | 65 | Flummoxing | 112 | Orbiting | 159 | Tomfoolering |
| 19 | Calculating | 66 | Fluttering | 113 | Orchestrating | 160 | Topsy-turvying |
| 20 | Canoodling | 67 | Forging | 114 | Osmosing | 161 | Transfiguring |
| 21 | Caramelizing | 68 | Forming | 115 | Perambulating | 162 | Transmuting |
| 22 | Cascading | 69 | Frolicking | 116 | Percolating | 163 | Twisting |
| 23 | Catapulting | 70 | Frosting | 117 | Perusing | 164 | Undulating |
| 24 | Cerebrating | 71 | Gallivanting | 118 | Philosophising | 165 | Unfurling |
| 25 | Channeling | 72 | Galloping | 119 | Photosynthesizing | 166 | Unravelling |
| 26 | Channelling | 73 | Garnishing | 120 | Pollinating | 167 | Vibing |
| 27 | Choreographing | 74 | Generating | 121 | Pondering | 168 | Waddling |
| 28 | Churning | 75 | Gesticulating | 122 | Pontificating | 169 | Wandering |
| 29 | Clauding | 76 | Germinating | 123 | Pouncing | 170 | Warping |
| 30 | Coalescing | 77 | Gitifying | 124 | Precipitating | 171 | Whatchamacalliting |
| 31 | Cogitating | 78 | Grooving | 125 | Prestidigitating | 172 | Whirlpooling |
| 32 | Combobulating | 79 | Gusting | 126 | Processing | 173 | Whirring |
| 33 | Composing | 80 | Harmonizing | 127 | Proofing | 174 | Whisking |
| 34 | Computing | 81 | Hashing | 128 | Propagating | 175 | Wibbling |
| 35 | Concocting | 82 | Hatching | 129 | Puttering | 176 | Working |
| 36 | Considering | 83 | Herding | 130 | Puzzling | 177 | Wrangling |
| 37 | Contemplating | 84 | Honking | 131 | Quantumizing | 178 | Zesting |
| 38 | Cooking | 85 | Hullaballooing | 132 | Razzle-dazzling | 179 | Zigzagging |
| 39 | Crafting | 86 | Hyperspacing | 133 | Razzmatazzing | | |
| 40 | Creating | 87 | Ideating | 134 | Recombobulating | | |
| 41 | Crunching | 88 | Imagining | 135 | Reticulating | | |
| 42 | Crystallizing | 89 | Improvising | 136 | Roosting | | |
| 43 | Cultivating | 90 | Incubating | 137 | Ruminating | | |
| 44 | Deciphering | 91 | Inferring | 138 | Sautéing | | |
| 45 | Deliberating | 92 | Infusing | 139 | Scampering | | |
| 46 | Determining | 93 | Ionizing | 140 | Schlepping | | |
| 47 | Dilly-dallying | 94 | Jitterbugging | 141 | Scurrying | | |

## 小贴士

| ID | 文本 |
|---|---|
| new-user-warmup | 从小功能或 bug 修复入手，让 Claude 先提出计划，再核验它建议的改动 |
| default-permission-mode-config | 用 /config 更改默认权限模式（包括 Plan Mode） |
| git-worktrees | 用 git worktree 并行运行多个 Claude 会话。 |
| color-when-multi-clauding | 同时开着多个 Claude 会话？用 /color 和 /rename 让它们一眼可辨。 |
| memory-command | 用 /memory 查看和管理 Claude 记忆 |
| theme-command | 用 /theme 更换配色主题 |
| colorterm-truecolor | 试试设置环境变量 COLORTERM=truecolor，获得更丰富的颜色 |
| powershell-tool-env | 设置 CLAUDE_CODE_USE_POWERSHELL_TOOL=1 启用 PowerShell 工具（预览版） |
| status-line | 用 /statusline 配置自定义状态栏，显示在输入框下方 |
| prompt-queue | Claude 工作时按 Enter，可以把后续消息排进队列。 |
| enter-to-steer-in-relatime | 在 Claude 工作时发消息，实时调整它的方向 |
| todo-list | 处理复杂任务时让 Claude 创建 todo 清单，跟踪进度、保持正轨 |
| ide-upsell-external-terminal | 把 Claude 连接到你的 IDE · /ide |
| install-github-app | 运行 /install-github-app，直接在 GitHub issue 和 PR 里 @claude |
| install-slack-app | 运行 /install-slack-app，在 Slack 里使用 Claude |
| permissions | 用 /permissions 预先批准或预先拒绝 bash、edit 和 MCP 工具 |
| drag-and-drop-images | 你知道可以直接把图片文件拖进终端吗？ |
| paste-images-mac | 在 Claude Code 里粘贴图片用 control+v（不是 cmd+v！） |
| double-esc | 连按两次 esc，把对话回退到之前的某个时间点 |
| double-esc-code-restore | 连按两次 esc，把代码和/或对话回退到之前的某个时间点 |
| continue | 运行 claude --continue 或 claude --resume 恢复对话 |
| rename-conversation | 用 /rename 给对话命名，之后在 /resume 里轻松找到 |
| custom-commands | 在项目的 .claude/skills/ 下添加 .md 文件即可创建 skill；放到 ~/.claude/skills/ 则在任何项目都可用 |
| custom-agents | 用 /agents 优化特定任务，例如 Software Architect、Code Writer、Code Reviewer |
| agent-flag | 用 --agent <agent_name> 直接与某个 subagent 开始对话 |
| desktop-app | 用 Claude 桌面应用在本地或远程运行 Claude Code：clau.de/desktop |
| web-app | 任务在云端跑，你继续在本地写代码 · clau.de/web |
| voice-mode | 用 /voice 启用按住说话的语音输入 |
| no-flicker | 试试无闪烁渲染，现已支持鼠标 · /tui fullscreen |
| team-artifacts | 从 team-discovery 状态中呈现团队 artifact 建议 |
| plan-mode-for-complex-tasks | 动手改之前，先用 Plan Mode 为复杂请求做准备。按 &lt;cycle-mode key&gt; |
| terminal-setup | 运行 /terminal-setup 启用便捷的终端集成，比如 Option+Enter 换行等 |
| shift-enter | 按 Option+Enter（Apple Terminal）或 Shift+Enter 发送多行消息 |
| shift-enter-setup | 运行 /terminal-setup 启用 Option+Enter（Apple Terminal）或 Shift+Enter 换行 |
| vscode-command-install | 打开命令面板（Cmd+Shift+P），运行 "Shell Command: Install '&lt;editor&gt;' command in PATH" 启用 IDE 集成 |
| shift-tab | 按 &lt;cycle-mode key&gt; 切换聊天模式 |
| image-paste | 用 &lt;image-paste key&gt; 粘贴图片 |
| desktop-shortcut | 用 &lt;suggested shortcut&gt; 在 Claude Code Desktop 中继续当前会话 |
| remote-control | 通过远程控制把本会话配对到你的手机 |
| push-notif | 长任务完成时在手机上收到通知——在 &lt;settings menu&gt; 中启用推送通知 |
| opusplan-mode-reminder | 你的默认模型设置是 Opus Plan Mode。按 &lt;cycle-mode key&gt; |
| frontend-design-plugin | 在写 HTML/CSS？安装 frontend-design 插件 |
