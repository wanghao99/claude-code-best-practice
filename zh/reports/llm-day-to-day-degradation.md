# LLM 日常性能劣化：迷思与现实

> 译自 [reports/llm-day-to-day-degradation.md](../../reports/llm-day-to-day-degradation.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

已部署的 LLM 在模型权重冻结的情况下，性能还会逐日变化吗？本文深入分析已被证实的成因、基础设施 bug 与心理因素。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<table width="100%">
<tr>
<td width="50%"><a href="https://x.com/nicksdot/status/2029520949176049704"><img src="../../reports/assets/llm-degradation.png" alt="Twitter 用户报告 Claude 质量逐日下降" width="100%" /></a></td>
<td width="50%"><a href="https://x.com/levelsio/status/2029369159893569680"><img src="../../reports/assets/llm-degradation-2.png" alt="Twitter 用户报告 Claude 质量逐日下降" width="100%" /></a></td>
</tr>
</table>

---
---

# Claude Code Ops 4.6 分析（High Reasoning）

当 Anthropic 发布 Opus 4.6 这样的模型时，**模型权重**——数十亿个学习到的参数——就冻结了。训练极其昂贵（数百万美元、数周算力），没有人会一夜之间重训模型。

但权重只是这个庞大系统中的一层。研究揭示了至少 **7 种独立机制**，它们可以在模型权重冻结的情况下，引起真实存在或被感知到的质量变化。

| 问题 | 答案 |
|----------|--------|
| 模型权重在发布后会变吗？ | **不会**——所有提供商均已确认 |
| 模型会逐日表现不同吗？ | **会**——已证实存在 ±8-14% 的波动 |
| 这是蓄意的"削弱（nerfing）"吗？ | **不是**——没有蓄意降质的证据 |
| 基础设施 bug 是真的吗？ | **是**——Anthropic 确认了 3 个 bug，最多影响 16% 的请求 |
| 有一部分是心理因素吗？ | **是**——确认偏误与蜜月效应确实存在 |
| 系统提示词/后训练会变吗？ | **会**——多家提供商都有记录 |
| 用户应该相信自己的感知吗？ | **部分**——真实成因存在，但感知会放大它们 |

---

## 完整的推理栈

模型权重是冻结的，但**其上的九个层面**都能独立影响你的体验：

```
┌──────────────────────────────────────────────┐
│  YOUR SESSION CONTEXT                        │  ← Degrades within session
│  (accumulated errors, long conversations)    │
├──────────────────────────────────────────────┤
│  SYSTEM PROMPT                               │  ← Updated regularly
│  (safety rules, behavior instructions)       │
├──────────────────────────────────────────────┤
│  POST-TRAINING (RLHF / Fine-tuning)         │  ← Can be updated quietly
│  (instruction following, safety alignment)   │
├──────────────────────────────────────────────┤
│  SAMPLING PARAMETERS                         │  ← Can be tuned server-side
│  (temperature, top-p, top-k)                 │
├──────────────────────────────────────────────┤
│  SPECULATIVE DECODING                        │  ← Draft model quality varies
│  (draft model predictions + verification)    │
├──────────────────────────────────────────────┤
│  MoE ROUTING / BATCH COMPOSITION             │  ← ±8-14% variance proven
│  (which experts activate per request)        │
├──────────────────────────────────────────────┤
│  HARDWARE ROUTING                            │  ← TPU vs GPU vs Trainium
│  (which cluster serves your request)         │
├──────────────────────────────────────────────┤
│  QUANTIZATION LEVEL                          │  ← May vary under load
│  (FP16 vs INT8 vs INT4 precision)            │
├──────────────────────────────────────────────┤
│  COMPILER & RUNTIME                          │  ← XLA bugs proven real
│  (XLA:TPU, CUDA, hardware-specific code)     │
├──────────────────────────────────────────────┤
│  MODEL WEIGHTS (FROZEN)                      │  ← These DON'T change
│  (billions of learned parameters)            │
└──────────────────────────────────────────────┘
```

（图中各层注释依次为：会话内劣化 / 定期更新 / 可能被悄悄更新 / 可在服务端调整 / 草稿模型质量不定 / 已证实 ±8-14% 波动 / 由哪个集群服务你的请求 / 负载下可能变化 / XLA bug 已证实为真 / 权重不变。）

关键心智模型：**权重冻结 ≠ 行为冻结。** 这就好比说"同一台发动机 = 同样的驾驶体验"，却无视轮胎、路况、油品和司机疲劳。

---

## 已证实的成因：基础设施 bug

### Anthropic 2025 年 9 月的事后复盘

2025 年 9 月，Anthropic 发布了一份详细的事后复盘（postmortem），披露了 **三个独立的基础设施 bug**，导致 2025 年 8 月至 9 月间 Claude 的质量下降。官方声明：

> "我们绝不会因为需求量、时段或服务器负载而降低模型质量。用户报告的问题完全由基础设施 bug 引起。"

### Bug 1 — 上下文窗口路由错误

Sonnet 4 的请求被错误路由到配置为 1M token 上下文窗口的服务器，而不是标准服务器。

- **时间线**：8 月 5 日引入，8 月 29 日一次负载均衡变更后恶化
- **峰值影响**：最严重的小时（8 月 31 日）有 16% 的 Sonnet 4 请求受影响
- **用户影响**：约 30% 的 Claude Code 用户至少遇到过一条劣化消息
- **隐蔽细节**：路由是"粘性"的——一旦命中一台坏服务器，后续请求会继续发往那里
- **修复**：9 月 4–18 日（在各平台陆续推出）

### Bug 2 — TPU 输出损坏

TPU 服务器上的一处配置错误在 token 生成阶段引发错误，给本应极少出现的 token 赋予了高概率。

- **症状**：英文回复中间冒出泰文或中文字符、明显的代码语法错误
- **受影响模型**：Opus 4.1 与 Opus 4（8 月 25–28 日）、Sonnet 4（8 月 25 日–9 月 2 日）
- **范围**：仅 Claude API；第三方平台不受影响
- **修复**：9 月 2 日回滚

### Bug 3 — XLA:TPU 编译器错误编译（最棘手）

一个为修复精度问题而做的代码改动，意外触发了 Google XLA:TPU 中**一个潜藏的编译器 bug**。

- **根因**：近似版 top-k 运算（用于挑选最可能的后续 token）"有时会返回完全错误的结果，但只发生在特定批大小和模型配置下"
- **为什么难找**：它的行为会随之前或之后运行了哪些运算而改变，还会受调试工具是否启用影响
- **潜伏数月**：2024 年 12 月的一个临时规避方案一直在无意中掩盖着这个更深的 bug
- **受影响**：确认 Haiku 3.5；疑似影响部分 Sonnet 4 和 Opus 3 请求
- **解决**：从近似 top-k 切换为精确 top-k；接受"轻微的效率影响"，因为"模型质量不容妥协"

### 为什么当时很难检测出来

Anthropic 自己的自动化评估没有捕获用户报告的劣化，"部分原因是 Claude 往往能从孤立的失误中恢复得很好"。每个 bug 在不同平台、以不同比例产生不同症状，形成了"一团混乱的报告，无法指向任何单一原因"。

关键背景：Claude 运行在**三种不同的硬件平台**上（AWS Trainium、NVIDIA GPU、Google TPU），各有不同的故障模式、编译器和精度行为。你的请求在不同日子可能落在不同硬件上。

---

## 已证实的成因：MoE 路由波动

现代大模型普遍采用**混合专家（Mixture-of-Experts，MoE）**架构：每个输入只激活模型参数的一个子集（"专家"），由一个学习出来的路由器决定用哪些专家。

Scale AI 的研究揭示了一个关键发现：

> "稀疏 MoE 与批量推理的组合会带来不可预测的结果：一个 batch 的组成可能决定你的查询被路由给哪个专家，而同 batch 中其他用户查询的混合方式不是确定性的。"

### 各提供商的逐日分数波动

| 提供商 | 逐日分数波动 |
|----------|--------------------------|
| OpenAI（GPT-4 系列） | ±10–12% |
| Anthropic（Claude 系列） | ±8–11% |
| Google（Gemini 系列） | ±9–14% |

一个具体例子：同一个模型在某天的**越狱抵抗测试中拿到 77%，第二天只有 63%**。同一模型、同一权重、同一测试——仅基础设施就带来 14 个百分点的摆动。

这意味着：即使零 bug、零变更，同一个模型也可能仅因请求的批处理与路由方式不同，在不同日子产出质量明显不同的输出。当日与日之间的噪声达到 10–15% 时，A/B 测试无法可靠地分辨出 5% 的质量信号。

---

## 已证实的成因：系统提示词与后训练更新

### 系统提示词变更

模型权重不变，但包裹这些权重的**系统提示词**可以随时更新。对 Claude 系统提示词演化的分析显示了几十次迭代，其中"热修（hot-fix）"——为压制不良行为而追加的简短指令——在持续地添加和移除。

Claude 3.7 的系统提示词包含多条针对常见 LLM"坑"的热修指令；Claude 4.0 的系统提示词把它们全部移除，相关行为改由后训练阶段的强化学习解决。

### 后训练理论

对无法解释的质量变化，最可信的理论是：公司可以在不改基础模型权重的情况下更新**微调与 RLHF**（基于人类反馈的强化学习）。这样一来，"模型没变"在技术上依然成立，但行为已经通过更新的安全护栏和指令遵循调整而改变。

---

## 已证实的成因：静默换模型

OpenAI 已被多次记录在用户不知情的情况下更换其交互的模型：

- 一夜之间移除模型选择器，把用户从 GPT-4o 切到 GPT-5
- 把 GPT-4o 变成需要在设置里手动开启的隐藏"遗留模型"，应用内没有任何通知
- 一个"自动切换器" bug 把用户路由到错误的模型
- Plus 订阅用户报告模型在未经同意的情况下切换到"受限版本"

Sam Altman 承认这次上线"比我们希望的要颠簸一些"。Reddit 上的帖子收到数千点赞，称新模型是"灾难"和"降级"。

这说明行业里换模型**确实会发生**——有时是有意的（产品决策），有时是意外（路由 bug）。

---

## 促成因素

### 负载下的量化

为了低成本服务数百万用户，公司可能部署模型的**量化（quantized）**版本——把精度从 FP16 降到 INT8 或 INT4。这能把内存占用降低 2–4 倍并加速推理，但会引入细微的质量损失。提供商是否会在负载下动态切换量化级别尚有争议，但这种技术能力确实存在，且在 vLLM、TensorRT 等服务框架中有充分文档。

### 投机解码

现代服务栈用更小的"草稿"模型向前预测多个 token，再由真正的模型验证。理论上这保持相同的输出分布，但实践中接受率随领域和上下文变化。开箱即用的草稿模型在某些场景表现良好，但常常在领域专属任务或超长上下文上吃力。

### 上下文窗口污染

在长编码会话中，早先的错误会在上下文里累积。模型看到自己的错误，可能延续它们。这是单次会话内"Claude 变笨了"最常见的原因——不是模型劣化，而是上下文被污染。

**实用提示**：感觉质量不对时，用 `/compact` 或开一个全新会话。这是你能做的单个最有效的动作。

---

## 斯坦福研究——以及它为什么复杂

斯坦福与 UC Berkeley 2023 年的那项里程碑研究（Chen、Zaharia、Zou）——"How is ChatGPT's Behavior Changing Over Time?"——常被当作 LLM 劣化的证据引用。其标题级发现：

> 在"这个数是质数吗？请一步步想"的任务上，GPT-4 的准确率从 2023 年 3 月到 6 月间**从 97.6% 跌到 2.4%**。

### 研究证明了什么

- "同一个" LLM 服务的行为**可以在短期内大幅变化**
- 不同能力可能朝相反方向移动（GPT-4 数学变差，GPT-3.5 变好）
- 代码生成质量下降（GPT-4 可执行代码比例：52% → 10%）
- 这项研究创造了 **"LLM drift（LLM 漂移）"** 这个词

### 方法论批评

- 3 月版本使用 **temperature 0.0**，而 6 月版本使用 **temperature 1.0**——一个会增加随机性的根本性混杂变量
- 每个任务只有 **500 次查询**——不足以支撑确定的统计结论
- 那些"数学题"实际上是是非题，变化的是模型的猜测模式，而非数学能力
- 这些变化更可能反映有意的**后训练安全更新**，而非劣化

这项研究证明了一件重要的事——LLM 行为会随时间变化——但机制大概率是有意更新，而不是无意劣化。

---

## 心理层面

### 确认偏误

一旦有人发推说"Claude 今天很蠢"，你就会开始注意到每一个错误。在没人抱怨的日子里，同样的错误你会一带而过。社交媒体会放大这种效应。

### 蜜月效应

用户对新模型有一段初始蜜月期，随后逐渐发现其局限。模型没变——期望的爬升速度超过了能力应有的水平。

### 任务难度波动

你的任务每天都在变。一整天都在啃难题，会感觉模型变差了，其实并没有。

### "周末 Claude"迷思

尽管许多用户相信存在星期几的规律，严谨分析**没有找到**按星期几划分的质量模式的稳定证据。一篇题为 "AI is Dumber on Mondays" 的分析一无所获。

### LLM 的随机本质

LLM 是概率性的。同一提示词每次可能产生不同输出。运气不好时，你可能连续拿到好几个糟糕回复——纯粹是随机性，不是劣化。

---

## 结论

用户描述的现象**真实存在，但归因错误**：

- **正确的是**：他们某些天的体验确实变差了
- **错误的是**：模型被蓄意"削弱"了

实际成因是以下几项的组合：

1. **基础设施 bug**——Anthropic 的事后复盘已证明（最多影响 16% 的请求）
2. **MoE 路由波动**——Scale AI 实测 ±8-14% 的质量摆动，且发生在零变更情况下
3. **系统提示词与后训练更新**——多家提供商均有记录
4. **硬件异构**——TPU、GPU、Trainium 各有不同的故障模式
5. **上下文污染**——长会话劣化会话内质量
6. **确认偏误**——社交媒体放大感知到的模式
7. **随机波动**——同一模型、同一提示词，每次输出都不同

测量问题很严重：±8-14% 的逐日波动意味着你无法把真实的 5% 质量变化从噪声里分辨出来。这正是"全是你的错觉"和"他们削了模型"两个阵营都自信满满的原因——信噪比让任何人都无法仅凭个人经验作出判断。

---

## 参考来源

- [Anthropic: A Postmortem of Three Recent Issues](https://www.anthropic.com/engineering/a-postmortem-of-three-recent-issues) — Official postmortem detailing three infrastructure bugs (September 2025)
- [Anthropic Reveals Three Infrastructure Bugs — InfoQ](https://www.infoq.com/news/2025/10/anthropic-infrastructure-bugs/) — Technical analysis of the postmortem
- [How is ChatGPT's Behavior Changing Over Time? — Stanford/UC Berkeley](https://arxiv.org/abs/2307.09009) — Landmark study on LLM drift (2023)
- [The Truth About ChatGPT's Degrading Capabilities — TechTalks](https://bdtechtalks.com/2023/07/24/chatgpt-capabilities-degrading-study/) — Methodological critique of the Stanford study
- [LLMs Are Getting Dumber and We Have No Idea Why — Ignorance.ai](https://www.ignorance.ai/p/llms-are-getting-dumber-and-we-have) — Five theories for perceived degradation
- [When Claude Forgets How to Code — Robert Matsuoka](https://hyperdev.matsuoka.com/p/when-claude-forgets-how-to-code) — Analysis of Claude quality fluctuations and infrastructure causes
- [Smoothing Out LLM Variance — Scale AI](https://scale.com/blog/smoothing-out-llm-variance) — Measured ±8-14% day-to-day variance across providers
- [What We Can Learn from Anthropic's System Prompt Updates — PromptLayer](https://blog.promptlayer.com/what-we-can-learn-from-anthropics-system-prompt-updates/) — System prompt evolution analysis
- [Claude's System Prompt Changes Reveal Anthropic's Priorities — Drew Breunig](https://www.dbreunig.com/2025/06/03/comparing-system-prompts-across-claude-versions.html) — Hot-fix patterns in system prompts
- [Complaints About Secretly Switching Models — OpenAI Forum](https://community.openai.com/t/complaints-about-secretly-switching-models/1360150) — Documented silent model swaps
- [Speculative Decoding — BentoML LLM Inference Handbook](https://bentoml.com/llm/inference-optimization/speculative-decoding) — How draft models affect serving
- [A Visual Guide to Mixture of Experts — Maarten Grootendorst](https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-mixture-of-experts) — MoE architecture and routing explained

---
---

# Codex 5.3：高置信推理与发现

### 报告范围

本节解释为什么会出现这样一个短窗口：Claude 的输出质量下滑，而 Codex 5.3 在编码任务上感觉稳定或更强。重点不是长期的模型质量排名，而是真实服务条件下短周期生产行为。

报告日期：2026 年 3 月 5 日。

### 观察到的模式

被报告的模式是：

1. 模型质量在一段时间内可接受。
2. 质量看起来连续几天下滑。
3. 质量回到接近此前的基线。

这种形状通常是服务栈或发布（rollout）模式，而不是基础模型能力的永久变化。永久性的能力下降在没有显式回滚或修复的情况下，通常不会恢复得这么快。

### 高置信结论：为什么在劣化窗口里 Codex 5.3 可能看起来更强

在另一家提供商的劣化期内，Codex 5.3 可能明显显得更强，有若干可以同时成立的技术原因：

1. 产品目标契合。Codex 5.3 针对代码生成与 agentic 编码工作流优化，因此即便原始模型强度相当，工具编排、仓库级推理和面向代码的指令微调也能带来更好的编码结果。
2. 推理策略差异。各提供商独立调整延迟、推理深度和解码默认值。同一天里，一家更保守的策略可能比另一家激进提速的策略看起来"更聪明"。
3. 服务路径分离。即使两家提供商都托管最先进的模型，它们的路由层、编译器/运行时栈和发布流水线各不相同。一方的故障并不意味着另一方同步劣化。
4. 发布与回滚时机。如果一家正处于发布中段而另一家稳定，用户就会看到很大的暂时性质量分化，而模型权重没有任何长期变化。
5. 会话级污染效应。在长编码对话中，错误累积会放大感知到的下滑。竞品助手感觉更好，可能只是因为出错的会话被重置了，或者它的工具循环恢复得更快。

### 详细发现

对于"Claude 感觉弱了大概四天，然后又回来了"这样的报告，最可能的解释是：

1. 某个提供商侧的事故、路由问题、解码/运行时 bug 或发布回退，影响了一部分请求。
2. 问题持续得足够久，在真实工作流中被反复注意到。
3. 问题被修复或回滚。
4. 感知质量迅速回升。

同一时期，Codex 5.3 可能感觉明显更好，因为它不共享同一条事故路径，也因为编码任务优化放大了实际结果上的差距。

### 针对该模式的假设排序

| 假设 | 可能性 | 理由 |
|------------|------------|-----------|
| 提供商事故加回滚 | 高 | 与"多天下滑后快速恢复"最匹配 |
| 服务配置变更（采样/延迟/推理预算） | 高 | 不重训模型却突然改变行为的常见来源 |
| 静默别名或快照迁移 | 中高 | 无用户可见动作也能改变行为 |
| 仅提示词漂移与上下文污染 | 中 | 能劣化单个会话，但不太可能单独解释大范围的多日报告 |
| 基础模型永久劣化 | 低 | 与质量快速回升矛盾 |

### 什么能证实或证伪这个发现

要把高置信推断变成硬证据，需要对同一任务集跨天采集请求级遥测：

1. 请求时刻的确切模型标识符与快照/别名。
2. 提供商暴露的任何后端指纹或发布标记。
3. 解码参数（temperature、top_p、top_k、max tokens）。
4. 延迟、超时与错误率轨迹。
5. 在固定编码基准提示集上的结构化质量评分。
6. 失败点的会话长度与 token 上下文深度。

如果质量下降与事故窗口、配置变更或后端指纹变化相关，则事故/配置假设成立。如果不存在这类变化且劣化只出现在长会话中，上下文污染就成为首要解释。

### 实用工程建议

为降低生产环境的逐日波动：

1. 有条件就固定模型快照，不用浮动别名。
2. 保存请求元数据（模型 ID、参数、延迟、错误、响应质量标签）。
3. 每天跑一套固定的编码任务金丝雀套件，出现回退即告警。
4. 连续多轮失败后，重置或压缩长会话。
5. 为事故窗口保留备用的提供商/模型路径。
6. 在内部仪表盘上把"模型质量"与"服务可靠性"分开呈现。

### 最终结论

在 Claude 的短暂劣化窗口里，Codex 5.3 显得更好，是现代 LLM 运维中技术上说得通、也预料得到的结果。最强的解释不是模型永久性崩坏，而是一家提供商的服务路径暂时劣化，叠加编码专属优化，以及同期另一家提供商的稳定运行。
