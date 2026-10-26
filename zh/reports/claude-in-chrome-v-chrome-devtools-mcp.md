# 浏览器自动化 MCP 全面对比报告

> 译自 [reports/claude-in-chrome-v-chrome-devtools-mcp.md](../../reports/claude-in-chrome-v-chrome-devtools-mcp.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 摘要

基于大量调研，我分析了你截图中的两款工具，外加第三个主要竞争者。以下是完整的对比分析，帮助你为自己的工作选择最佳的自动化测试方案。

---

## 1. 三个竞争者

### **A. Chrome DevTools MCP**（你的截图 #1）
- **来源：** Google Chrome 官方团队
- **发布：** 2025 年 9 月公开预览
- **架构：** 基于 Chrome DevTools Protocol (CDP) + Puppeteer 构建
- **Token 占用：** 约 19.0k token（上下文的 9.5%）
- **工具：** 26 个专用工具，分 6 大类

### **B. Claude in Chrome**（你的截图 #2）
- **来源：** Anthropic 官方扩展
- **发布：** Beta 版，正逐步推送到所有付费计划（Pro、Max、Team、Enterprise）
- **架构：** 具备 computer-use（计算机操作）能力的浏览器扩展
- **Token 占用：** 约 15.4k token（上下文的 7.7%）
- **工具：** 16 个工具，包含 computer use 能力

### **C. Playwright MCP**（强有力的替代方案）
- **来源：** Microsoft（官方实现 + 社区实现）
- **架构：** 基于可访问性树的自动化
- **Token 占用：** 约 13.7k token（上下文的 6.8%）
- **工具：** 21 个工具

---

## 2. 详细特性对比

| 特性 | Chrome DevTools MCP | Claude in Chrome | Playwright MCP |
|---------|---------------------|------------------|----------------|
| **主要用途** | 调试与性能分析 | 通用浏览器自动化 | UI 测试与 E2E |
| **浏览器支持** | 仅 Chrome | 仅 Chrome | Chromium、Firefox、WebKit |
| **Token 效率** | 19.0k (9.5%) | 15.4k (7.7%) | 13.7k (6.8%) |
| **元素选择** | CSS/XPath 选择器 | 视觉 + DOM | 可访问性树（语义化） |
| **性能追踪** | 极好 | 不支持 | 有限 |
| **网络检查** | 深度分析 | 基础 | 基础 |
| **控制台日志** | 完整访问 | 完整访问 | 有限 |
| **跨浏览器** | 不支持 | 不支持 | 支持 |
| **CI/CD 集成** | 极好 | 差（需要登录） | 极好 |
| **无头模式** | 支持 | 不支持 | 支持 |
| **认证** | 需要配置 | 使用你的会话 | 需要配置 |
| **定时任务** | 不支持 | 支持 | 不支持 |
| **费用** | 免费 | 需要付费计划 | 免费 |
| **本地安装** | 需要 Node.js | 浏览器扩展 | 需要 Node.js |

---

## 3. 工具明细

### Chrome DevTools MCP（26 个工具）

```
INPUT AUTOMATION (8):     click, drag, fill, fill_form, handle_dialog,
                          hover, press_key, upload_file

NAVIGATION (6):           close_page, list_pages, navigate_page,
                          new_page, select_page, wait_for

EMULATION (2):            emulate, resize_page

PERFORMANCE (3):          performance_analyze_insight,
                          performance_start_trace, performance_stop_trace

NETWORK (2):              get_network_request, list_network_requests

DEBUGGING (5):            evaluate_script, get_console_message,
                          list_console_messages, take_screenshot,
                          take_snapshot
```

### Claude in Chrome（16 个工具）

```
BROWSER CONTROL:          navigate, read_page, find, computer
                          (click, type, scroll)

FORM INTERACTION:         form_input, javascript_tool

MEDIA:                    upload_image, get_page_text, gif_creator

TAB MANAGEMENT:           tabs_context_mcp, tabs_create_mcp

DEVELOPMENT:              read_console_messages, read_network_requests

UTILITIES:                shortcuts_list, shortcuts_execute,
                          resize_window, update_plan
```

### Playwright MCP（21 个工具）

```
NAVIGATION:               navigate, goBack, goForward, reload

INTERACTION:              click, fill, select, hover, press,
                          drag, uploadFile

ELEMENT QUERIES:          getElement, getElements, waitForSelector

ASSERTIONS:               assertVisible, assertText, assertTitle

PAGE STATE:               screenshot, getAccessibilityTree,
                          evaluateScript

BROWSER MGMT:             newPage, closePage
```

---

## 4. 自动化测试使用场景分析

### **Chrome DevTools MCP 最适合：**

**性能测试**
- 录制带 Core Web Vitals 的性能追踪
- 定位渲染瓶颈与布局偏移
- 内存泄漏检测与 CPU 性能分析

**深度调试**
- 网络请求检查（header、payload、时序）
- 控制台错误分析与堆栈追踪
- 实时 DOM 检查

**CI/CD 流水线**
- 支持无头执行
- 稳定的、基于脚本的自动化
- 不依赖认证状态

**理想工作流：**"找出这个页面为什么慢"或"调试这个 API 调用"

---

### **Claude in Chrome 最适合：**

**手动测试辅助**
- 在已登录账号的状态下测试
- 带视觉上下文的探索性测试
- 录制可回放的工作流

**快速验证**
- 设计验证（对比 Figma 与产出）
- 抽查新功能
- 开发过程中读取控制台错误

**周期性浏览器任务**
- 定时自动检查
- 多标签页工作流管理
- 从你录制的操作中学习

**理想工作流：**"检查我的改动看起来是否正常"或"用我的登录态测试这个表单"

---

### **Playwright MCP 最适合：**

**E2E 测试自动化**
- 跨浏览器测试（Chrome、Firefox、Safari）
- 生成可复用的测试脚本
- 生成 Page Object Model

**可靠的 UI 测试**
- 可访问性树 = 不会出现不稳定的选择器
- 确定性交互
- 不易因 UI 变更而失效

**CI/CD 集成**
- 流水线使用无头模式
- 从自然语言生成 Playwright 测试文件
- 与测试管理工具集成

**理想工作流：**"为这个用户流程编写 E2E 测试"或"跨浏览器测试这个功能"

---

## 5. Token 效率分析

| 工具 | Token 占用 | 上下文占比 | 效率评级 |
|------|-------------|--------------|-------------------|
| Playwright MCP | ~13.7k | 6.8% | 最佳（5/5 星） |
| Claude in Chrome | ~15.4k | 7.7% | 良好（4/5 星） |
| Chrome DevTools MCP | ~19.0k | 9.5% | 尚可（3/5 星） |

**影响：** 在 200k token 上下文下：
- Playwright 为你的工作留出 186.3k token
- Claude in Chrome 留出 184.6k token
- Chrome DevTools 留出 181k token

Playwright 与 Chrome DevTools 之间约 5.3k token 的差距，在带大量代码上下文的复杂会话中可能有影响。

---

## 6. 安全性考量

### Chrome DevTools MCP
- 默认使用隔离的浏览器配置文件
- 无云依赖
- 完全本地控制
- 注意远程调试端口安全（使用隔离配置文件）

### Claude in Chrome
- 无缓解措施时**攻击成功率为 23.6%**（有防御措施时降至 11.2%）
- 使用你真实的浏览器会话（存在 cookie 暴露风险）
- 屏蔽金融/成人/盗版站点
- 仍处于 beta 阶段，存在已知漏洞

### Playwright MCP
- 隔离的浏览器上下文
- 无云依赖
- 成熟的安全模型（Microsoft 支持）
- 能安全地处理认证

---

## 7. 安装命令

### Chrome DevTools MCP

```bash
claude mcp add chrome-devtools npx chrome-devtools-mcp@latest
```

### Claude in Chrome

```
Install from Chrome Web Store (requires Pro/Max/Team/Enterprise plan)
```

### Playwright MCP（推荐）

```bash
# First, install browsers
npx playwright install

# Then add to Claude Code (user scope = all projects)
claude mcp add playwright -s user -- npx @playwright/mcp@latest
```

---

## 8. 推荐方案

### **针对你的自动化测试工作流：**

#### 首选工具：Playwright MCP

**用途：** 日常 E2E 测试、跨浏览器验证、生成测试脚本

**理由：**
- Token 占用最低（为你的代码留出更多上下文）
- 跨浏览器支持（Chrome、Firefox、Safari）
- 可访问性树方式 = 更可靠的选择器
- 出色的 CI/CD 集成
- 能生成真正的 Playwright 测试文件
- 免费，无需订阅

#### 次选工具：Chrome DevTools MCP

**用途：** 性能调试、网络分析、Core Web Vitals

**理由：**
- 性能追踪与调试能力无可匹敌
- 深入的网络请求检查
- Google 官方工具，长期支持
- 需要回答"为什么这么慢？"时必不可少

#### 按需使用：Claude in Chrome

**用途：** 登录状态下的快速手动验证、探索性测试、设计验证

**理由：**
- 适合开发过程中的快速视觉检查
- 能读取你的登录状态
- 对"这个看起来对吗？"的验证很有用
- CI/CD 或严肃的测试自动化请跳过它

---

## 9. 推荐配置

```bash
# Install both Playwright and Chrome DevTools MCP
npx playwright install
claude mcp add playwright -s user -- npx @playwright/mcp@latest
claude mcp add chrome-devtools -s user -- npx chrome-devtools-mcp@latest
```

### 建议工作流

```
1. DEVELOP      → Claude Code (terminal)
2. TEST         → Playwright MCP (E2E, cross-browser)
3. DEBUG        → Chrome DevTools MCP (performance, network)
4. VERIFY       → Claude in Chrome (quick visual checks)
5. CI/CD        → Playwright MCP (headless, automated)
```

---

## 10. 最终结论

| 如果你需要… | 就用这个 |
|----------------|----------|
| 跨浏览器 E2E 测试 | **Playwright MCP** |
| 性能分析 | **Chrome DevTools MCP** |
| 网络调试 | **Chrome DevTools MCP** |
| 快速视觉验证 | **Claude in Chrome** |
| CI/CD 自动化 | **Playwright MCP** |
| 测试脚本生成 | **Playwright MCP** |
| 最低 token 占用 | **Playwright MCP** |
| 登录态会话测试 | **Claude in Chrome** |
| 控制台日志调试 | **Chrome DevTools MCP** |

### **一句话推荐：**

**同时安装 Playwright MCP 和 Chrome DevTools MCP。** 以 Playwright 作为主力测试工具（token 效率更高、跨浏览器、更适合 E2E）。需要深入的性能分析或网络调试时使用 Chrome DevTools。Claude in Chrome 只在需要登录会话的快速手动验证时使用。

---

## 参考来源

- [Chrome DevTools MCP - GitHub](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- [Anthropic - Piloting Claude in Chrome](https://claude.com/blog/claude-for-chrome)
- [Claude in Chrome Help Center](https://support.claude.com/en/articles/12012173-getting-started-with-claude-in-chrome)
- [Playwright MCP - GitHub](https://github.com/microsoft/playwright-mcp)
- [Simon Willison - Using Playwright MCP with Claude Code](https://til.simonwillison.net/claude-code/playwright-mcp-claude-code)
- [Testomat.io - Playwright MCP Claude Code](https://testomat.io/blog/playwright-mcp-claude-code/)
- [MCP Integration Guide - Scrapeless](https://www.scrapeless.com/en/blog/mcp-integration-guide)
- [Chrome DevTools MCP Guide - Vladimir Siedykh](https://vladimirsiedykh.com/blog/chrome-devtools-mcp-ai-browser-debugging-complete-guide-2025)
- [Addy Osmani - Give your AI eyes](https://addyosmani.com/blog/devtools-mcp/)

---

*本报告由 Claude Code 使用 Opus 4.5 模型于 2025 年 12 月 19 日生成。*
