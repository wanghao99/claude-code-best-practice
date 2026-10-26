# 理解大型 Monorepo 中的 Claude Skills 发现机制

> 译自 [reports/claude-skills-for-larger-mono-repos.md](../../reports/claude-skills-for-larger-mono-repos.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

在 monorepo 中使用 Claude Code 时，理解 skill 如何被发现并加载进上下文，对有效组织项目专属能力至关重要。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 与 CLAUDE.md 的重要区别

**Skill 的加载行为与 CLAUDE.md 文件不同。** CLAUDE.md 文件会沿目录树向上查找（祖先加载），而 skill 使用另一种发现机制，专注于项目内部的嵌套目录。

## Skill 如何被发现

### 1. 标准 Skill 位置

skill 按作用域从以下固定位置加载：

| 位置 | 路径 | 生效范围 |
|----------|------|------------|
| 企业级（Enterprise） | 托管设置 | 组织内所有用户 |
| 个人级（Personal） | `~/.claude/skills/<skill-name>/SKILL.md` | 你的所有项目 |
| 项目级（Project） | `.claude/skills/<skill-name>/SKILL.md` | 仅当前项目 |
| 插件（Plugin） | `<plugin>/skills/<skill-name>/SKILL.md` | 启用该插件的地方 |

### 2. 从嵌套目录自动发现

当你处理子目录中的文件时，Claude Code 会自动从嵌套的 `.claude/skills/` 目录中发现 skill。例如，如果你正在编辑 `packages/frontend/` 下的文件，Claude Code 也会去 `packages/frontend/.claude/skills/` 里查找 skill。

这支持了各 package 拥有自己 skill 的 monorepo 结构。

## Monorepo 结构示例

设想一个按 package 拆分的典型 monorepo：

```
/mymonorepo/
├── .claude/
│   └── skills/
│       └── shared-conventions/SKILL.md    # Project-level skill
├── packages/
│   ├── frontend/
│   │   ├── .claude/
│   │   │   └── skills/
│   │   │       └── react-patterns/SKILL.md  # Frontend-specific skill
│   │   └── src/
│   │       └── App.tsx
│   ├── backend/
│   │   ├── .claude/
│   │   │   └── skills/
│   │   │       └── api-design/SKILL.md      # Backend-specific skill
│   │   └── src/
│   └── shared/
│       ├── .claude/
│       │   └── skills/
│       │       └── utils-patterns/SKILL.md  # Shared utilities skill
│       └── src/
```

## 场景 1：刚在根目录启动 Claude（还没编辑任何文件）

当你在 `/mymonorepo/` 下运行 Claude Code、且尚未编辑任何文件时：

```bash
cd /mymonorepo
claude
# Just started - no files edited yet
```

| Skill | 在上下文里吗？ | 原因 |
|-------|-------------|--------|
| `shared-conventions` | **是** | 根目录 `.claude/skills/` 中的项目级 skill |
| `react-patterns` | **否** | 未被发现——还没处理过 `packages/frontend/` 下的文件 |
| `api-design` | **否** | 未被发现——还没处理过 `packages/backend/` 下的文件 |
| `utils-patterns` | **否** | 未被发现——还没处理过 `packages/shared/` 下的文件 |

## 场景 2：编辑过某个 package 里的文件之后

当你让 Claude 编辑 `packages/frontend/src/App.tsx` 之后：

| Skill | 在上下文里吗？ | 原因 |
|-------|-------------|--------|
| `shared-conventions` | **是** | 根目录 `.claude/skills/` 中的项目级 skill |
| `react-patterns` | **是** | 编辑 `packages/frontend/` 下文件时被发现 |
| `api-design` | **否** | 仍未被发现——还没处理过 `packages/backend/` 下的文件 |
| `utils-patterns` | **否** | 仍未被发现——还没处理过 `packages/shared/` 下的文件 |

**关键洞察**：嵌套 skill 是在你处理那些目录下的文件时**按需**发现的，不会在会话开始时预加载。

## 关键行为：描述 vs 完整内容

skill 的描述会加载进上下文，让 Claude 知道有哪些可用能力，但**完整 skill 内容只在被调用时加载**。这是一项重要优化：

- **描述**：始终在上下文中（受字符预算约束）
- **完整内容**：skill 被调用时按需加载

> 注意：预加载了 skill 的 subagent 行为不同——完整 skill 内容会在启动时注入。

## 优先级顺序（当 skill 同名时）

当不同层级的 skill 同名时，高优先级位置获胜：

| 优先级 | 位置 | 作用域 |
|----------|----------|-------|
| 1（最高） | 企业级 | 整个组织 |
| 2 | 个人级（`~/.claude/skills/`） | 你的所有项目 |
| 3（最低） | 项目级（`.claude/skills/`） | 仅当前项目 |

插件 skill 使用 `plugin-name:skill-name` 命名空间，因此不会与其他层级冲突。

## 这个设计为什么适合 Monorepo

- **package 专属 skill 保持隔离** —— 在 `packages/frontend/` 工作的前端开发者获得前端专属 skill，后端 skill 不会挤占上下文。

- **自动发现减少配置** —— 不需要显式注册 package 级 skill；处理那些目录下的文件时它们会被自动发现。

- **上下文得到优化** —— 初始只加载 skill 描述，嵌套 skill 按需发现。

- **各团队可以维护自己的 skill** —— 每个 package 团队都能定义自己领域专属的 skill，无需与其他团队协调。

## 字符预算注意事项

skill 描述加载进上下文时受字符预算限制（默认 15,000 字符）。在拥有大量 package 和 skill 的大型 monorepo 中，你可能撞到这个上限。

- 运行 `/context` 查看是否有 skill 被排除的警告
- 设置 `SLASH_COMMAND_TOOL_CHAR_BUDGET` 环境变量调高上限

## 最佳实践

1. **把共享工作流放在根目录 `.claude/skills/`** —— 全仓库的规范、提交工作流与共享模式。

2. **把 package 专属 skill 放在 package 的 `.claude/skills/`** —— 该 package 独有的框架模式、组件规范与测试工具。

3. **危险 skill 使用 `disable-model-invocation: true`** —— 部署类或破坏性 skill 应要求用户显式调用。

4. **保持 skill 描述简洁** —— 描述始终占用上下文（受字符预算限制），冗长的描述浪费上下文空间。

5. **在 skill 名中使用命名空间** —— 考虑用 package 名做前缀（如 `frontend-review`、`backend-deploy`）避免混淆。

## 对比：Skill 与 CLAUDE.md 的加载

| 行为 | CLAUDE.md | Skills |
|----------|-----------|--------|
| 祖先加载（沿目录树向上） | 是 | 否 |
| 嵌套/子目录发现（沿目录树向下） | 是（懒加载） | 是（自动发现） |
| 全局位置 | `~/.claude/CLAUDE.md` | `~/.claude/skills/` |
| 项目位置 | `.claude/` 或仓库根目录 | `.claude/skills/` |
| 内容加载 | 完整内容 | 仅描述（调用时加载完整内容） |

---

## 参考来源

- [Claude Code Documentation - Extend Claude with Skills](https://code.claude.com/docs/en/skills)
- [Claude Code Documentation - Automatic Discovery from Nested Directories](https://code.claude.com/docs/en/skills#automatic-discovery-from-nested-directories)
