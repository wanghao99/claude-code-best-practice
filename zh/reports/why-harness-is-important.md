# 为什么是 Harness——Claude Code 设计解析

> 译自 [reports/why-harness-is-important.md](../../reports/why-harness-is-important.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

解释 Claude Code 的各项功能为什么不是"换了马甲的提示词"，以及 harness 为什么才是把玩具级输出和生产级工程工作真正区分开来的东西。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 执行摘要

资深 Claude Code 用户中流行一种归约：*"skill、command、subagent、hook——它们最终都会变成发给模型的提示词，所以光有一条强提示词就等价了。"*

在模型最终推理调用的那一层，这在技术上是事实：模型看到的只有 token。

而在其他所有层面——真正发生软件工程的地方——**这种归约就站不住脚了。** Harness 不是一套提示词投递系统，而是一套**提示词构建系统、确定性执行系统和上下文架构系统**——这些能力无法用更强的措辞替代。

本报告说明这种归约在哪里成立、在哪里失效，以及为什么把"模型看到什么"与"系统做了什么"混为一谈，会让实践者偏离那些真正给 Claude Code 带来杠杆的功能。

---

## 听起来有道理的归约

对于**一次性的原子任务**——比如"给我写一个递归计算斐波那契数列的函数"——harness 对输出质量没有任何贡献。把相同的 token 交给相同的模型，无论它们是经由 skill、command 还是裸提示词送达，得到的输出分布都一样。

在这个狭窄的范围内，归约成立：

> 输出质量 ≈ 提示词质量

这正是 Claude Code 相比普通聊天机器人几乎没有附加价值的范围——也是这种归约默认假定的范围，而它恰恰不是真实工程工作所处的范围。

---

## 归约在哪里失效

harness 有十项架构能力，作用在提示词无法触及的层面上。

| # | 能力 | 它做什么 | 为什么提示词复制不了 |
|---|------------|--------------|-------------------------------|
| 1 | **上下文隔离** | Subagent 在各自独立的上下文窗口中运行 | 提示词只能填满一个窗口。N 个并行 subagent 带来约 N 倍的有效上下文。 |
| 2 | **Harness 强制的工具限制** | `allowed-tools` / `disallowedTools` 在模型能够使用工具之前就将其屏蔽 | 提示词里的指令只是建议，模型可以无视；拒绝规则无法被无视。 |
| 3 | **懒加载的规则与记忆** | `paths:` frontmatter 和子目录中的 `CLAUDE.md` 只在 Claude 触碰匹配路径时才加载 | 提示词是静态的——无法根据运行时正在读哪些文件来条件加载。 |
| 4 | **Hook：确定性代码执行** | 生命周期事件（PreToolUse、PostToolUse、Stop 等）触发 shell 命令执行，并且可以**阻断**工具调用 | 提示词无法拦截自己的工具调用；hook 不管模型"想不想"都会执行。 |
| 5 | **模型路由** | `model: haiku` 或 `model: opus` 把调用路由到不同的模型端点 | 提示词中的任何 token 都改变不了由哪个模型应答。 |
| 6 | **并行能力** | 多个 subagent 并发执行 | 提示词是串行的；由 harness 负责调度并行进程并收集结果。 |
| 7 | **跨会话持久化** | 记忆系统和设置层级在会话之间持久保存 | 提示词随会话结束而消失。 |
| 8 | **模块化系统提示词** | CLI 根据启用的功能，有条件地加载 110 多个系统提示词片段 | 用户无法手工编写或替换 CLI 内部的提示词片段。 |
| 9 | **Skill 预加载** | `skills:` 字段把 skill 的完整内容注入 subagent 的起始上下文 | 用户无法替另一个 agent 预先塞入上下文——只有 harness 加载器能做到。 |
| 10 | **权限分类** | `auto` 权限模式用后台分类器预先批准或阻断工具调用 | 提示词无法为自身增加一层执行前的安全防线。 |

每一行都是"更强的措辞"绝对无法替代的一个维度。

---

## "提示词"一词的两种含义

这种归约玩了一个偷换概念的游戏。"提示词"这个词被用来指代两种截然不同的东西：

| 含义 | 谁控制 | 规模 |
|---------|-----------------|------|
| (a) 用户敲进去的内容 | 用户 | 约 6–60 个 token |
| (b) 推理时模型实际看到的内容 | harness | 约 5,000–50,000+ 个 token |

在聊天机器人里，(a) 和 (b) 是同一个东西。
在 Claude Code 里，二者天差地别。

harness 的职责恰恰是让 (b) 比 (a) 丰富得多：

```
User types: "write a recursive flatten function"   ← (a) ~6 tokens

What the model actually sees at inference:         ← (b) ~15,000 tokens
  ├── CLAUDE.md (project conventions)
  ├── Matching .claude/rules/*.md (loaded via paths: frontmatter)
  ├── Modular system prompt fragments
  ├── Tool definitions
  ├── Environment context (cwd, git status, platform)
  ├── Prior turn history
  ├── Files read by the model via Read/Grep tools
  └── User's 6-token request
```

**输出质量是 (b) 的函数，不是 (a) 的函数。** (b) 由 harness 构建。"光靠一条强提示词"无法重现 (b)，因为其中大部分根本不是用户写的。

---

## 即便只看输出质量，harness 也在做功

设想同一句提示——"写一个递归的 flatten 函数"——放在三种环境里：

| 环境 | 模型看到什么 | 典型结果 |
|-------------|---------------------|----------------|
| 聊天机器人，无工具 | 那句话 | 教科书式递归，风格泛泛 |
| Claude Code，未读取文件 | 那句话 + CLAUDE.md | 符合项目声明的规范 |
| Claude Code，agentic 循环 | 那句话 + CLAUDE.md + 读取相邻文件 + 跑测试 | 符合真实代码库的模式、通过测试、处理好既有代码已经处理的边界情况 |

同一个模型，同一句用户提示，**三种不同的输出质量。** 差别来自 harness——具体说，来自它组装出的有效上下文，以及它使能的迭代循环。

对于非平凡任务，输出质量是以下因素的函数：

```
Output quality = f(effective context, model capability, iteration loop)
```

用户只控制*有效上下文*的一小片（自己敲的提示词），其余部分——以及整个迭代循环——都由 harness 控制。

---

## 这种归约对在哪里（又错在哪里）

| 说法 | 判定 |
|-------|---------|
| "推理时，模型只看到 token。" | 正确 |
| "Skill、command 和 subagent 提示词都会向某个上下文贡献 token。" | 正确 |
| "对于真空中的原子任务，提示词质量主导输出质量。" | 正确 |
| "因此强提示词等价于使用各项功能。" | 错误 |
| "因此 harness 对输出质量无关紧要。" | 在真实工程任务上错误 |

前三句是准确的观察。跳到第四句的推理才是失败之处：它把模型与包裹模型的系统混为一谈，也把原子任务与真实工程工作混为一谈。

---

## 正确的心智模型

> **提示词控制的是要求模型做什么。**
> **harness 控制的是系统在模型触及不到的层面做什么**——在 token 到达之前、在 token 产出之后、跨会话、跨上下文、跨进程。

各项功能不是"多了几个步骤的提示词"，而是 **harness 层面的原语**——确定性执行、上下文架构与基础设施路由——它们作用在模型没有发言权的层面。

一个有用的类比：

| 层 | 聊天机器人 | Claude Code |
|-------|---------|-------------|
| 菜谱 | 用户的消息 | 用户的消息 + harness 组装的上下文 |
| 厨房 | 没有——只有一个学生 | 工具、hook、记忆、并行工人、生命周期事件 |

你可以写出世上最好的菜谱，但没有厨房，就做不成规模。

---

## 给实践者的要点

1. **对原子问题，提示词质量几乎就是一切。** harness 无关紧要，只需这个的话用聊天机器人就够了。
2. **对真实代码库工作，harness 在默默负重。** 推理时的有效提示词大部分由 harness 构建，而非用户敲入。
3. **把功能用在提示词原则上做不到的事情上：** 确定性（hook）、隔离（subagent）、懒加载（带 `paths:` 的 rules）、持久化（memory）、路由（按 agent 的 `model:`）与并行。
4. **强提示词必要但不充分。** 功能提供提示词无法提供的确定性、隔离与组合能力，二者互补，不能互相替代。

---

## 参考来源

- [Agents vs Commands vs Skills](claude-agent-command-skill.md) — 展示了各功能的上下文隔离、模型覆盖与工具限制
- [Claude Agent SDK vs CLI System Prompts](claude-agent-sdk-vs-cli-system-prompts.md) — 记录了 110 多个模块化系统提示词片段
- [Claude Agent Memory](claude-agent-memory.md) — 通过 `memory:` 作用域实现跨会话持久化
- [Claude Memory Best Practice](../../best-practice/claude-memory.md) — 懒加载的子目录 `CLAUDE.md` 文件
- [Claude Subagents Best Practice](../../best-practice/claude-subagents.md) — harness 强制能力的 frontmatter 参考
- [Claude Settings Best Practice](../../best-practice/claude-settings.md) — 权限规则求值与 `auto` 模式分类器
- [Orchestration Workflow](../../orchestration-workflow/orchestration-workflow.md) — 证明该归约失效的具体演示
