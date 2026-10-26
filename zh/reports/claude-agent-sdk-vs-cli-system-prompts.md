# Claude Agent SDK 与 Claude CLI 对比：系统提示词与输出一致性

> 译自 [reports/claude-agent-sdk-vs-cli-system-prompts.md](../../reports/claude-agent-sdk-vs-cli-system-prompts.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

![SDK 与 CLI 系统提示词对比图](../../reports/assets/sdk-vs-cli-diagram.svg)

---

## 执行摘要

把同一条消息（例如"挪威的首都是哪里？"）分别发给 **Claude Agent SDK** 和 **Claude CLI（Claude Code）**时，随消息一同发出的系统提示词从根本上不同。CLI 采用**模块化系统提示词架构**（约 269 个基础 token，另按启用功能有条件地加载额外上下文），而 SDK 默认使用极简提示词。**两者之间不存在输出完全一致的保证**——即使配置完全相同——原因在于缺少 seed 参数以及 Claude 架构固有的非确定性。

---

## 1. 系统提示词对比

### Claude CLI（Claude Code）

Claude CLI 采用**模块化系统提示词架构**：基础提示词约 269 个 token，其余上下文按条件加载：

| 组件 | 描述 | 加载方式 |
|-----------|-------------|---------|
| **基础系统提示词** | 核心指令与行为规范 | 始终加载（约 269 token） |
| **工具说明** | 18 个以上内置工具（Write、Read、Edit、Bash、TodoWrite 等） | 始终加载 |
| **编码准则** | 代码风格、格式规则、安全实践 | 始终加载 |
| **安全规则** | 拒绝规则、注入防御、危害防范 | 始终加载 |
| **回复风格** | 语气、详略程度、解释深度、emoji 用法 | 始终加载 |
| **环境上下文** | 工作目录、git 状态、平台信息 | 始终加载 |
| **项目上下文** | CLAUDE.md 内容、设置、hook 配置 | 条件加载 |
| **Subagent 提示词** | Plan mode、Explore agent、Task agent | 条件加载 |
| **安全审查** | 扩展安全指令（约 2,610 token） | 条件加载 |

**关键特性：**
- **模块化架构**，110 多个系统提示词字符串按条件加载
- 基础提示词不大（约 269 token），总量随启用的功能变化
- 包含大量的安全与注入防御层
- 自动加载工作目录中的 CLAUDE.md 文件
- 交互模式下上下文在整个会话中保持

### Claude Agent SDK

Agent SDK 默认使用**极简系统提示词**，只包含：

| 组件 | 描述 | token 影响 |
|-----------|-------------|--------------|
| **必要的工具说明** | 仅有显式提供的工具 | 极小 |
| **基础安全** | 极简的安全指令 | 极小 |

**关键特性：**
- 默认没有编码准则或风格偏好
- 默认没有项目上下文，除非显式配置
- 没有详细的工具描述
- 需要显式配置才能对齐 CLI 行为

---

## 2. 每个接口实际发送什么

### 示例："挪威的首都是哪里？"

#### 通过 Claude CLI

```
System Prompt: [modular, ~269+ base tokens]
├── Base system prompt (~269 tokens)
├── Tool instructions (Write, Read, Edit, Bash, Grep, Glob, etc.)
├── Git safety protocols
├── Code reference guidelines
├── Professional objectivity instructions
├── Security and injection defense rules
├── Environment context (OS, directory, date)
├── CLAUDE.md content (if present) [conditional]
├── MCP tool descriptions (if configured) [conditional]
├── Plan/Explore mode prompts [conditional]
└── Session/conversation context

User Message: "What is the capital of Norway?"
```

#### 通过 Claude Agent SDK（默认）

```
System Prompt: [minimal]
├── Essential tool instructions (if any tools provided)
└── Basic operational context

User Message: "What is the capital of Norway?"
```

#### 通过 Agent SDK（使用 `claude_code` 预设）

```typescript
const response = await query({
  prompt: "What is the capital of Norway?",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code"
    }
  }
});
```

```
System Prompt: [modular, matches CLI]
├── Full Claude Code system prompt
├── Tool instructions
├── Coding guidelines
└── Safety rules

// NOTE: Still does NOT include CLAUDE.md unless settingSources is configured
```

---

## 3. 自定义方式

### Claude CLI 的自定义

| 方式 | 命令 | 效果 |
|--------|---------|--------|
| **追加提示词** | `claude -p "..." --append-system-prompt "..."` | 在保留默认值的基础上追加指令 |
| **替换提示词** | `claude -p "..." --system-prompt "..."` | 完全替换系统提示词 |
| **项目上下文** | CLAUDE.md 文件 | 自动加载、持续生效 |
| **输出样式** | `/output-style [name]` | 应用预定义的回复风格 |

### Agent SDK 的自定义

| 方式 | 配置 | 效果 |
|--------|---------------|--------|
| **自定义提示词** | `systemPrompt: "..."` | 完全替换默认值（丢失工具） |
| **预设 + 追加** | `systemPrompt: { type: "preset", preset: "claude_code", append: "..." }` | 保留 CLI 功能 + 自定义指令 |
| **CLAUDE.md 加载** | `settingSources: ["project"]` | 加载项目级指令 |
| **输出样式** | `settingSources: ["user"]` 或 `settingSources: ["project"]` | 加载已保存的输出样式 |

### 配置对比表

| 特性 | CLI 默认 | SDK 默认 | SDK 加预设 |
|---------|-------------|-------------|-----------------|
| 工具说明 | 完整 | 极简 | 完整 |
| 编码准则 | 有 | 无 | 有 |
| 安全规则 | 有 | 基础 | 有 |
| CLAUDE.md 自动加载 | 有 | 无 | 无* |
| 项目上下文 | 自动 | 无 | 无* |

*需要显式配置 `settingSources: ["project"]`

---

## 4. 输出一致性保证

### 关键结论：不保证确定性

**Claude Messages API 不提供用于复现的 seed 参数。** 这是架构层面的根本限制。

### 阻止输出完全一致的因素

| 因素 | 描述 | 可控吗？ |
|--------|-------------|---------------|
| **系统提示词不同** | CLI 与 SDK 默认值不同 | 可控（通过配置） |
| **浮点运算** | 并行硬件的行为差异 | 不可控 |
| **MoE 路由** | 混合专家（Mixture-of-Experts）架构的路由变化 | 不可控 |
| **批处理与调度** | 云基础设施差异 | 不可控 |
| **数值精度** | 推理引擎差异 | 不可控 |
| **模型快照** | 版本更新/变更 | 不可控 |

### 温度与采样

即使设置 `temperature=0.0`（贪心解码）：
- 也**不**保证完全确定
- 基础设施因素仍可能造成细微差异
- 已知问题：[Claude CLI 对相同输入产生非确定性输出](https://github.com/anthropics/claude-code/issues/3370)

---

## 5. 达到最大一致性

要让 SDK 与 CLI 的输出**尽可能接近**一致：

### Agent SDK 配置

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

// Option 1: Use claude_code preset
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  // Match CLI system prompt as closely as possible
  system: "Your exact system prompt matching CLI",
  messages: [
    { role: "user", content: "What is the capital of Norway?" }
  ],
  // Use greedy decoding for maximum consistency
  temperature: 0
});

// Option 2: With Agent SDK query function
import { query } from "@anthropic-ai/agent-sdk";

for await (const message of query({
  prompt: "What is the capital of Norway?",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code"
    },
    temperature: 0,
    model: "claude-sonnet-4-20250514",
    // Load project context like CLI does
    settingSources: ["project"]
  }
})) {
  // Process response
}
```

### CLI 配置

```bash
# Match the SDK configuration as closely as possible
claude -p "What is the capital of Norway?" \
  --model claude-sonnet-4-20250514 \
  --temperature 0
```

### 仍然没有保证

即便配置完全对齐：
- 同一配置多次运行，输出可能不同
- SDK 与 CLI 之间，输出可能不同
- 不存在强制复现的 seed 参数

---

## 6. 实践影响

### 各接口的适用场景

| 使用场景 | 推荐接口 | 原因 |
|----------|----------------------|--------|
| 交互式开发 | Claude CLI | 完整工具套件、项目上下文 |
| 程序化集成 | Agent SDK | 细粒度控制、便于嵌入 |
| 稳定的 API 响应 | Agent SDK + 自定义提示词 | 对系统提示词控制更强 |
| 批处理 | Agent SDK | 更适合自动化流水线 |
| 一次性任务 | Claude CLI | 上手更快、上下文立即可用 |

### 设计建议

1. **不要依赖比特级完美的可复现性**
   - 让应用对输出的细微差异保持健壮
   - 使用结构化输出与校验

2. **对一致性有要求的生产流水线：**
   - 尽可能缓存结果
   - 使用结构化输出加 JSON schema 校验
   - 结合确定性逻辑与校验
   - 考虑多次生成后取共识

3. **在 SDK 中对齐 CLI 行为：**
   ```typescript
   systemPrompt: {
     type: "preset",
     preset: "claude_code",
     append: "Your additional instructions"
   },
   settingSources: ["project", "user"]
   ```

---

## 7. 系统提示词的 token 影响

| 配置 | 架构 | 说明 |
|---------------|-------------|-------|
| SDK（极简） | 极简默认 | 只有必要的工具说明 |
| SDK（claude_code 预设） | 模块化（约 269+ 基础） | 对齐 CLI，随功能变化 |
| CLI（默认） | 模块化（约 269+ 基础） | 额外上下文按条件加载 |
| CLI（含 MCP 工具） | 模块化 + MCP | MCP 工具描述带来可观的 token 增量 |

**注意：** Claude Code 采用模块化架构，包含 110 多个系统提示词字符串。基础提示词约 269 个 token，各组件从 18 到 2,610 个 token 不等，取决于启用的功能。

**含义：** SDK 的极简默认值给你的实际任务留出更多上下文，但代价是失去 Claude Code 的完整能力。

---

## 8. 汇总表

| 方面 | Claude CLI | Agent SDK（默认） | Agent SDK（预设） |
|--------|------------|--------------------|--------------------|
| **系统提示词** | 模块化（约 269+ 基础） | 极简 | 模块化（对齐 CLI） |
| **内置工具** | 18 个以上 | 仅显式提供 | 18 个以上 |
| **CLAUDE.md 自动加载** | 是 | 否 | 否（需配置） |
| **编码准则** | 是 | 否 | 是 |
| **安全规则** | 完整 | 基础 | 完整 |
| **温度控制** | 是 | 是 | 是 |
| **确定性保证** | 无 | 无 | 无 |
| **输出完全一致？** | 不适用 | 否（相对 CLI） | 更接近，但仍否 |

---

## 9. 结论

**问：同一条消息在 SDK 与 CLI 中分别附带什么系统提示词？**

CLI 采用**模块化系统提示词架构**：约 269 个 token 的基础提示词加 110 多个按条件加载的组件（工具说明、编码准则、安全规则、项目上下文）。SDK 使用**极简默认值**，只含必要的工具说明，但可以通过 `claude_code` 预设配置成与 CLI 一致的行为。

**问：有输出完全一致的保证吗？**

**没有。** 即使系统提示词一致、输入一致、`temperature=0`，也无法保证输出完全一致，原因包括：
- Claude 的 API 缺少 seed 参数
- 浮点运算差异
- 基础设施层面的非确定性
- 模型架构（Mixture-of-Experts）的路由变化

**建议：** 把系统设计成对输出差异健壮，而不是依赖确定性行为。对一致性要求高的应用，使用结构化输出、缓存与校验层。

---

## 参考来源

- [Modifying System Prompts - Agent SDK](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/sdk#modifying-system-prompts)
- [Claude Code CLI Reference](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/cli)
- [Claude Code Headless Mode](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/headless)
- [Claude Code Best Practices - Anthropic Engineering](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Claude Messages API Reference](https://docs.anthropic.com/en/api/messages)
- [GitHub Issue #3370: Non-deterministic output](https://github.com/anthropics/claude-code/issues/3370)
- [Claude Code System Prompts Repository](https://github.com/Piebald-AI/claude-code-system-prompts) - Analysis of modular prompt architecture
- [Why Deterministic Output from LLMs is Nearly Impossible](https://unstract.com/blog/understanding-why-deterministic-output-from-llms-is-nearly-impossible/)

---

*本报告由 Claude Code 使用 Opus 4.5 模型于 2026 年 2 月 3 日生成。*
