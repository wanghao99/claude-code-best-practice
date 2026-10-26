# Claude 记忆系统

> 译自 [best-practice/claude-memory.md](../../best-practice/claude-memory.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

通过 CLAUDE.md 文件实现持久上下文 — 怎么写，以及在 monorepo 中怎么加载。

<table width="100%">
<tr>
<td><a href="../../">← 返回 Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1. 写好 CLAUDE.md

一份结构良好的 CLAUDE.md 是提升 Claude Code 在你项目中输出质量的最有效手段。Humanlayer 有一篇优秀指南，讲该写什么、怎么组织、常见坑。

- [Humanlayer - Writing a good Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## 2. 大型 Monorepo 中的 CLAUDE.md

在 monorepo 中使用 Claude Code 时，理解 CLAUDE.md 如何被加载进上下文，对有效组织项目指令至关重要。

<p align="center">
  <a href="https://x.com/bcherny/status/2016339448863355206"><img src="../../best-practice/assets/claude-memory/claude-memory-monorepo.jpg" alt="CLAUDE.md 在 monorepo 中的加载" width="600"></a>
</p>

### 两种加载机制

Claude Code 用两种不同的机制加载 CLAUDE.md：

#### 祖先加载（沿目录树向上）

启动 Claude Code 时，它从当前工作目录**向上**走到文件系统根目录，沿途发现的所有 CLAUDE.md 都会加载。这些文件在**启动时立即加载**。

#### 后代加载（沿目录树向下）

当前工作目录下子目录中的 CLAUDE.md **不会在启动时加载**。只有当 Claude 在会话中读取那些子目录里的文件时才会包含进来。这就是**懒加载（lazy loading）**。

### 示例 Monorepo 结构

一个典型的 monorepo，不同组件放在不同目录：

```
/mymonorepo/
├── CLAUDE.md          # 根级指令（所有组件共享）
├── frontend/
│   └── CLAUDE.md      # 前端专属指令
├── backend/
│   └── CLAUDE.md      # 后端专属指令
└── api/
    └── CLAUDE.md      # API 专属指令
```

### 场景 1：从根目录启动 Claude Code

在 `/mymonorepo/` 下运行时：

```bash
cd /mymonorepo
claude
```

| 文件 | 启动时加载？ | 原因 |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | 是 | 它是你的当前工作目录 |
| `/mymonorepo/frontend/CLAUDE.md` | 否 | 只有读/改 `frontend/` 里的文件时才加载 |
| `/mymonorepo/backend/CLAUDE.md` | 否 | 只有读/改 `backend/` 里的文件时才加载 |
| `/mymonorepo/api/CLAUDE.md` | 否 | 只有读/改 `api/` 里的文件时才加载 |

### 场景 2：从组件目录启动 Claude Code

在 `/mymonorepo/frontend/` 下运行时：

```bash
cd /mymonorepo/frontend
claude
```

| 文件 | 启动时加载？ | 原因 |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | 是 | 它是祖先目录 |
| `/mymonorepo/frontend/CLAUDE.md` | 是 | 它是你的当前工作目录 |
| `/mymonorepo/backend/CLAUDE.md` | 否 | 目录树的另一个分支 |
| `/mymonorepo/api/CLAUDE.md` | 否 | 目录树的另一个分支 |

### 关键要点

1. **祖先总是启动时加载** — Claude 沿目录树向上走，加载找到的所有 CLAUDE.md。这保证你始终能用上根级的、全仓库的指令。

2. **后代懒加载** — 子目录的 CLAUDE.md 只有在你操作那些子目录里的文件时才加载。避免无关上下文撑爆会话。

3. **兄弟分支永不加载** — 在 `frontend/` 工作时，`backend/CLAUDE.md` 和 `api/CLAUDE.md` 不会进入上下文。

4. **全局 CLAUDE.md** — 还可以在主目录放 `~/.claude/CLAUDE.md`，它对**所有** Claude Code 会话生效，与项目无关。

### 为什么这个设计适合 Monorepo

- **共享指令向下传播** — 根级 CLAUDE.md 存放全仓库的约定、编码规范和通用模式。

- **组件专属指令保持隔离** — 前端开发者的上下文不会被后端专属指令弄乱，反之亦然。

- **上下文经过优化** — 后代 CLAUDE.md 懒加载，Claude Code 避免在启动时装入可能高达数百 KB 的无关指令。

### 最佳实践

1. **共享约定放根 CLAUDE.md** — 编码规范、提交信息格式、PR 模板等全仓库准则。

2. **组件专属指令放组件 CLAUDE.md** — 该组件特有的框架模式、架构、测试约定。

3. **个人偏好用 CLAUDE.local.md** — 不想与团队共享的指令放这里，并加入 `.gitignore`。

---

## 参考来源

- [Claude Code 文档 - How Claude Looks Up Memories](https://code.claude.com/docs/en/memory#how-claude-looks-up-memories)
- [Boris Cherny 在 X 上对 CLAUDE.md 加载机制的澄清](https://x.com/bcherny/status/2016339448863355206)
- [Humanlayer - Writing a good Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
