# Claude 高级 Tool Use 模式

> 译自 [reports/claude-advanced-tool-use.md](../../reports/claude-advanced-tool-use.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

API 层面的特性（现已 GA），可降低 token 消耗与延迟，并提升工具准确率。随 Opus/Sonnet 4.6 一同发布。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 目录

1. [概述](#overview)
2. [Programmatic Tool Calling（程序化工具调用，PTC）](#programmatic-tool-calling-ptc)
3. [Web Search/Fetch 动态过滤](#dynamic-filtering-for-web-searchfetch)
4. [Tool Search Tool（工具搜索工具）](#tool-search-tool)
5. [Tool Use Examples（工具使用示例）](#tool-use-examples)
6. [与 Claude Code 的关联](#claude-code-relevance)

---

## 概述

| 特性 | 解决的问题 | Token 节省 | 可用性 |
|---------|---------------|---------------|--------------|
| Programmatic Tool Calling | 多步 Agent 循环在往返中消耗大量 token | 降低约 37% | API、Foundry（GA） |
| Dynamic Filtering | Web search/fetch 结果中无关内容撑大上下文 | 输入 token 减少约 24% | API、Foundry（GA） |
| Tool Search Tool | 过多工具定义撑大上下文 | 降低约 85% | API、Foundry（GA） |
| Tool Use Examples | 仅靠 schema 无法表达使用模式 | 准确率 72% → 90% | API、Foundry（GA） |

所有特性自 2026 年 2 月 18 日起**正式可用（GA）**。

**分层策略** —— 从最大的瓶颈入手：
- 工具定义导致上下文膨胀 → Tool Search Tool
- 大量中间结果 → Programmatic Tool Calling
- Web 搜索噪声 → Dynamic Filtering
- 参数错误 → Tool Use Examples

---

## Programmatic Tool Calling（程序化工具调用，PTC）

<img src="../../reports/assets/programmatic-tool-calling-diagram.svg" alt="PTC 示意图：传统工具调用 vs 程序化工具调用" width="100%" />

### 范式转变

**之前（传统工具调用）：**
```
User prompt → Claude → Tool call 1 → Response 1 → Claude → Tool call 2 → Response 2 → Claude → Tool call 3 → Response 3 → Claude → Final answer
```
每次工具调用都需要一次完整的模型往返。3 个工具 = 3 次推理。

**之后（Programmatic Tool Calling）：**
```
User prompt → Claude → writes Python script → Script calls Tool 1, Tool 2, Tool 3 internally → stdout → Claude → Final answer
```
Claude 编写代码来编排所有工具。只有最终的 `stdout` 进入上下文窗口。3 个工具 = 1 次推理。

### 工作原理

1. 你在定义工具时指定 `allowed_callers: ["code_execution_20250825"]`
2. Claude 编写 Python，在沙箱内以异步函数的形式调用这些工具
3. 当工具函数被调用时，沙箱暂停，API 返回一个 `tool_use` 块
4. 你提供工具结果——它交给**正在运行的代码**，而不是进入 Claude 的上下文
5. 代码恢复运行，处理结果，需要时调用更多工具
6. 只有最终执行的 `stdout` 会到达 Claude

### 关键配置

```json
{
  "tools": [
    {
      "type": "code_execution_20250825",
      "name": "code_execution"
    },
    {
      "name": "query_database",
      "description": "Execute a SQL query. Returns rows as JSON objects with fields: id (str), name (str), revenue (float).",
      "input_schema": {
        "type": "object",
        "properties": {
          "sql": { "type": "string", "description": "SQL query to execute" }
        },
        "required": ["sql"]
      },
      "allowed_callers": ["code_execution_20250825"]
    }
  ]
}
```

### `allowed_callers` 字段

| 取值 | 行为 |
|-------|----------|
| `["direct"]` | 仅传统工具调用（省略时的默认值） |
| `["code_execution_20250825"]` | 只能从 Python 沙箱内调用 |
| `["direct", "code_execution_20250825"]` | 两种模式均可用 |

**建议：** 每个工具只选一种模式，不要两者都开。这能给 Claude 更明确的指引。

### 响应中的 `caller` 字段

每个 tool use 块都包含一个 `caller` 字段，让你知道它是如何被调用的：

```json
// Direct (traditional)
{ "caller": { "type": "direct" } }

// Programmatic (from code execution)
{ "caller": { "type": "code_execution_20250825", "tool_id": "srvtoolu_abc123" } }
```

### 高级模式

**批处理** —— 一次推理处理 N 个条目：
```python
regions = ["West", "East", "Central", "North", "South"]
results = {}
for region in regions:
    data = await query_database(f"SELECT SUM(revenue) FROM sales WHERE region='{region}'")
    results[region] = data[0]["revenue"]

top = max(results.items(), key=lambda x: x[1])
print(f"Top region: {top[0]} with ${top[1]:,}")
```

**提前终止** —— 一旦满足成功条件立即停止：
```python
endpoints = ["us-east", "eu-west", "apac"]
for endpoint in endpoints:
    status = await check_health(endpoint)
    if status == "healthy":
        print(f"Found healthy endpoint: {endpoint}")
        break
```

**条件化工具选择：**
```python
file_info = await get_file_info(path)
if file_info["size"] < 10000:
    content = await read_full_file(path)
else:
    content = await read_file_summary(path)
print(content)
```

**数据过滤** —— 减少 Claude 看到的内容：
```python
logs = await fetch_logs(server_id)
errors = [log for log in logs if "ERROR" in log]
print(f"Found {len(errors)} errors")
for error in errors[-10:]:
    print(error)
```

### 模型兼容性

| 模型 | 是否支持 |
|-------|-----------|
| Claude Opus 4.6 | 支持 |
| Claude Sonnet 4.6 | 支持 |
| Claude Sonnet 4.5 | 支持 |
| Claude Opus 4.5 | 支持 |

### 约束

| 约束 | 详情 |
|-----------|--------|
| **不支持 Bedrock/Vertex** | 仅限 API 与 Foundry |
| **不支持 MCP 工具** | MCP connector 工具无法被程序化调用 |
| **不支持 web search/fetch** | PTC 不支持 Web 工具 |
| **不支持结构化输出** | 与 `strict: true` 工具不兼容 |
| **不支持强制工具选择** | `tool_choice` 无法强制走 PTC |
| **容器生命周期** | 约 4.5 分钟后过期 |
| **ZDR** | 不在零数据保留（Zero Data Retention）覆盖范围内 |
| **工具结果为字符串** | 需校验外部结果，防范代码注入风险 |

### 何时使用 PTC

| 适用场景 | 不太适合的场景 |
|----------------|------------|
| 处理需要聚合的大型数据集 | 响应简单的单次工具调用 |
| 3 次以上连续依赖的工具调用 | 需要即时用户反馈的工具 |
| 在结果到达 Claude 之前先过滤/转换 | 极快的操作（开销大于收益） |
| 跨大量条目的并行操作 | |
| 基于中间结果的条件逻辑 | |

### Token 效率

- 程序化调用的工具结果**不会加入 Claude 的上下文** —— 只有最终 `stdout` 会
- 中间处理发生在代码中，而不是消耗模型 token
- 以程序化方式调用 10 个工具 ≈ 10 次直接调用所需 token 的 1/10

---

## Web Search/Fetch 动态过滤

### 问题

Web 搜索与抓取工具会把完整的 HTML 页面倒进 Claude 的上下文窗口。这些内容大多无关——导航、广告、样板文字。Claude 随后要在全部内容上进行推理，浪费 token 并降低准确率。

### 解决方案

Claude 现在**会编写并执行 Python 代码来过滤 Web 结果**，之后它们才进入上下文窗口。Claude 不再对原始 HTML 进行推理，而是在沙箱中过滤、解析并只提取相关内容。

### 工作原理

**之前：**
```
Query → Search results → Fetch full HTML × N pages → All content enters context → Claude reasons over everything
```

**之后：**
```
Query → Search results → Claude writes filtering code → Code extracts relevant content only → Filtered results enter context
```

### API 配置

使用带 beta header 的新版本工具类型：

```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 4096,
  "tools": [
    {
      "type": "web_search_20260209",
      "name": "web_search"
    },
    {
      "type": "web_fetch_20260209",
      "name": "web_fetch"
    }
  ]
}
```

**必需 header：** `anthropic-beta: code-execution-web-tools-2026-02-09`

在 Sonnet 4.6 与 Opus 4.6 上使用新版工具类型时**默认启用**。

### 基准测试结果

**BrowseComp**（在网站上查找特定信息）：

| 模型 | 无过滤 | 有过滤 | 提升 |
|-------|-------------------|----------------|-------------|
| Sonnet 4.6 | 33.3% | **46.6%** | +13.3 个百分点 |
| Opus 4.6 | 45.3% | **61.6%** | +16.3 个百分点 |

**DeepsearchQA**（多步研究，F1 分数）：

| 模型 | 无过滤 | 有过滤 | 提升 |
|-------|-------------------|----------------|-------------|
| Sonnet 4.6 | 52.6% | **59.4%** | +6.8 个百分点 |
| Opus 4.6 | 69.8% | **77.3%** | +7.5 个百分点 |

**Token 效率：** 输入 token 平均减少 24%。Sonnet 4.6 成本下降；Opus 4.6 因过滤代码更复杂，成本可能略有上升。

### 使用场景

- 筛查技术文档
- 跨多个来源核实引用
- 交叉核对搜索结果
- 多步研究查询
- 从大页面中找出埋藏的特定数据点

---

## Tool Search Tool（工具搜索工具）

### 问题

一次性加载全部工具定义会浪费上下文。如果你有 50 个 MCP 工具，每个约 1.5K token，用户还没提问就先占掉 75K token。

### 解决方案

给低频使用的工具标记 `defer_loading: true`。它们不会进入初始上下文。Claude 通过 Tool Search Tool 按需发现它们。

### 配置

```json
{
  "tools": [
    {
      "type": "mcp_toolset",
      "mcp_server_name": "google-drive",
      "default_config": { "defer_loading": true },
      "configs": {
        "search_files": { "defer_loading": false }
      }
    }
  ]
}
```

### 最佳实践

- 让最常用的 3-5 个工具始终加载，其余全部延迟
- 编写清晰、有描述性的工具名与描述（搜索依赖它们）
- 在系统提示中记录可用能力

### 何时使用

- 工具定义占用超过 10K token
- 可用工具超过 10 个
- 多个 MCP 服务器
- 选项过多导致工具选择准确率下降

### Token 节省

工具定义 token 减少约 85%（Anthropic 基准测试中从 77K 降至 8.7K）。

### Claude Code 中的对应实现

Claude Code 有 **MCP tool search auto 模式**（自 v2.1.7 起默认启用）。当 MCP 工具描述超过上下文的 10% 时，这些工具会被延迟加载并通过 `MCPSearch` 发现。可用 `ENABLE_TOOL_SEARCH=auto:N` 配置阈值，其中 N 为上下文百分比（0-100）。

---

## Tool Use Examples（工具使用示例）

### 问题

JSON schema 定义了结构，但无法表达：
- 何时应包含可选参数
- 哪些参数组合是合理的
- 格式约定（日期格式、ID 模式）
- 嵌套结构的用法

### 解决方案

在工具定义中添加 `input_examples` —— 超越 schema 的具体使用模式。

### 配置

```json
{
  "name": "create_ticket",
  "description": "Create a support ticket",
  "input_schema": {
    "type": "object",
    "properties": {
      "title": { "type": "string" },
      "priority": { "type": "string", "enum": ["low", "medium", "high", "critical"] },
      "assignee": { "type": "string" },
      "labels": { "type": "array", "items": { "type": "string" } }
    },
    "required": ["title"]
  },
  "input_examples": [
    {
      "title": "Login page returns 500 error",
      "priority": "critical",
      "assignee": "oncall-team",
      "labels": ["bug", "auth", "production"]
    },
    {
      "title": "Add dark mode support",
      "priority": "low",
      "labels": ["feature-request", "ui"]
    },
    {
      "title": "Update API docs for v2 endpoints"
    }
  ]
}
```

### 最佳实践

- 使用**真实感的数据**，而不是 "example_value" 这类占位字符串
- 展示**多样性**：最小、部分、完整三种规格
- 保持简洁：**每个工具 1-5 个示例**
- 聚焦于消除歧义——目标是行为层面的清晰，而非 schema 完整性
- 展示参数关联（例如 `priority: "critical"` 通常伴随 `assignee`）

### 结果

在 Anthropic 基准测试中，复杂参数处理准确率从 72% 提升到 90%。

---

## 与 Claude Code 的关联

### 哪些内容直接适用于 Claude Code 用户

| 特性 | Claude Code 中的状态 | 行动 |
|---------|-------------------|--------|
| Tool Search | 自 v2.1.7 起内置为 MCPSearch auto 模式 | MCP 工具较多时调整 `ENABLE_TOOL_SEARCH=auto:N` |
| Dynamic Filtering | CLI 中不可用（API 层 Web 工具） | 与做 Web 研究的 Agent SDK 用户相关 |
| PTC | CLI 中不可用 | 与构建自定义 Agent 的 Agent SDK 用户相关 |
| Tool Use Examples | CLI 中不可配置 | 与自定义 MCP 服务器作者相关 |

### 面向 Agent SDK 开发者

如果你正在用 `@anthropic-ai/claude-agent-sdk` 构建 Agent，PTC 可以立即上手：

1. 在 tools 数组中加入 `code_execution_20250825`
2. 给能从批处理/过滤中受益的工具设置 `allowed_callers`
3. 实现工具结果循环（暂停 → 提供结果 → 恢复）
4. 让工具返回结构化数据（JSON），便于程序化解析

### 面向 MCP 服务器作者

如果你在构建自定义 MCP 服务器，Tool Use Examples 可以改善 Claude 使用你工具的方式：
- 在工具 schema 中添加 `input_examples`
- 在描述中清楚记录返回格式（PTC 需要解析它们）

---

## 参考来源

- [Anthropic Engineering: Advanced Tool Use](https://www.anthropic.com/engineering/advanced-tool-use)
- [Programmatic Tool Calling Documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling)
- [Code Execution Tool Documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool)
- [Improved Web Search with Dynamic Filtering](https://claude.com/blog/improved-web-search-with-dynamic-filtering)
