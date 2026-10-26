# MCP Server 最佳实践

> 译自 [best-practice/claude-mcp.md](../../best-practice/claude-mcp.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。
> 原文最后更新 2026-03-02，徽章标注为已实施（Implemented）。

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../.mcp.json)

MCP（Model Context Protocol，模型上下文协议）服务器为 Claude Code 扩展了与外部工具、数据库和 API 的连接能力。本指南介绍日常使用的推荐服务器以及配置最佳实践。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 日常使用的 MCP 服务器

> *“曾一口气装了 15 个 MCP 服务器，以为越多越好。结果日常只用到 4 个。”* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)（682 赞）

| MCP 服务器 | 用途 | 资源 |
|------------|-------------|-----------|
| [**Context7**](https://github.com/upstash/context7) | 获取最新库文档到上下文，防止因过时训练数据产生虚构 API | [Reddit：“目前为止最好的编程类 MCP”](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | 浏览器自动化——自主实现、测试并验证 UI 功能。截图、导航、表单测试 | [Reddit：前端开发必备](https://reddit.com/r/mcp/comments/1m59pk0/) · [文档](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | 将 Claude 连接到你真实的 Chrome 浏览器——检查控制台、网络、DOM。调试用户实际看到的内容 | [Reddit：调试“游戏规则改变者”](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [对比报告](../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | 为任意 GitHub 仓库获取结构化的 wiki 式文档——架构、API 面、关系 | [Reddit：“把它和 Context7 一起放在网关后面”](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | 根据提示词生成架构图、流程图和系统设计，以手绘风格的 Excalidraw 草图呈现 | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

调研（Context7/DeepWiki）-> 调试（Playwright/Chrome）-> 绘图（Excalidraw）

---

## 配置

MCP 服务器在项目根目录的 `.mcp.json`（项目作用域）或 `~/.claude.json`（用户作用域）中配置。

### 服务器类型

| 类型 | 传输方式 | 示例 |
|------|-----------|---------|
| **stdio** | 启动一个本地进程 | `npx`、`python`、二进制文件 |
| **http** | 连接到一个远程 URL | HTTP/SSE 端点 |

### `.mcp.json` 示例

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

对密钥使用环境变量展开，而不是把 API 密钥提交到 `.mcp.json`：

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

### MCP 服务器相关设置

`.claude/settings.json` 中的以下设置控制 MCP 服务器的审批：

| 键 | 类型 | 说明 |
|-----|------|-------------|
| `enableAllProjectMcpServers` | boolean | 自动批准所有 `.mcp.json` 服务器，无需提示 |
| `enabledMcpjsonServers` | array | 要自动批准的指定服务器名称白名单 |
| `disabledMcpjsonServers` | array | 要拒绝的指定服务器名称黑名单 |

### MCP 工具的权限规则

MCP 工具在权限规则中遵循 `mcp__<server>__<tool>` 命名约定：

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

---

## MCP 作用域

MCP 服务器可以在三个层级定义：

| 作用域 | 位置 | 用途 |
|-------|----------|---------|
| **Project（项目）** | `.mcp.json`（仓库根目录） | 团队共享的服务器，提交到 git |
| **User（用户）** | `~/.claude.json`（`mcpServers` 键） | 跨所有项目的个人服务器 |
| **Subagent（子代理）** | Agent frontmatter（`mcpServers` 字段） | 限定于特定 subagent 的服务器 |

优先级：Subagent > Project > User

---

## 参考来源

- [MCP Servers — Claude Code Docs](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [5 MCPs that have genuinely made me 10x faster — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [MCP Server Overload Discussion — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)
