# RPI 工作流

> 译自 [development-workflows/rpi/rpi-workflow.md](../../../development-workflows/rpi/rpi-workflow.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

**RPI** = **R**esearch（研究）→ **P**lan（计划）→ **I**mplement（实现）

一套在每个阶段都设有验证关卡的系统性开发工作流。避免在不可行的特性上浪费精力，并确保文档完整。

<table width="100%">
<tr>
<td><a href="../../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 概述

![RPI Workflow](../../../development-workflows/rpi/rpi-workflow.svg)

---

## 安装

将 `.claude` 文件夹（包含 `agents/` 和 `commands/rpi/`）复制到你的仓库根目录，然后创建 `rpi/plans` 目录。

---

## 工作流示例

### 特性：用户认证

**Step 1：描述**
```
User: "Add OAuth2 authentication with Google and GitHub providers"

1. Claude generates plan
   → Output: rpi/plans/oauth2-authentication.md
2. Create feature folder: rpi/oauth2-authentication/
3. Copy the plan into the feature folder
4. Rename the plan to REQUEST.md
   → Final: rpi/oauth2-authentication/REQUEST.md
```

**Step 2：研究**
```bash
/rpi:research rpi/oauth2-authentication/REQUEST.md
```
输出：
- `research/RESEARCH.md`，含分析内容
- 结论：**GO**（可行，与策略一致）

**Step 3：计划**
```bash
/rpi:plan oauth2-authentication
```
输出：
- `plan/pm.md` — 用户故事与验收标准
- `plan/ux.md` — 登录 UI 流程
- `plan/eng.md` — 技术架构
- `plan/PLAN.md` — 3 个阶段，15 项任务

**Step 4：实现**
```bash
/rpi:implement oauth2-authentication
```
进度：
- 阶段 1：后端基础 → PASS
- 阶段 2：前端集成 → PASS
- 阶段 3：测试与打磨 → PASS

结果：特性完成，可提交 PR。

---

## 特性文件夹结构

所有特性工作都放在 `rpi/{feature-slug}/` 中：

```
rpi/{feature-slug}/
├── REQUEST.md              # Step 1: Initial feature description
├── research/
│   └── RESEARCH.md         # Step 2: GO/NO-GO analysis
├── plan/
│   ├── PLAN.md             # Step 3: Implementation roadmap
│   ├── pm.md               # Product requirements
│   ├── ux.md               # UX design
│   └── eng.md              # Technical specification
└── implement/
    └── IMPLEMENT.md        # Step 4: Implementation record
```

---

## Agent 与 Command

| Command | 使用的 Agent |
|---------|-------------|
| `/rpi:research` | requirement-parser, product-manager, Explore, senior-software-engineer, technical-cto-advisor, documentation-analyst-writer |
| `/rpi:plan` | senior-software-engineer, product-manager, ux-designer, documentation-analyst-writer |
| `/rpi:implement` | Explore, senior-software-engineer, code-reviewer |
