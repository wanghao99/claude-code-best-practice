# Day 0 — Claude Code 安装

> 译自 [tutorial/day0/README.md](../../../tutorial/day0/README.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

本指南带你在这台机器上安装 Claude Code 并完成登录认证，之后就可以开始使用了。

## 第 1 步：安装 Claude Code

按操作系统选择指南：

| 系统 | 指南 |
|----|-------|
| Windows | [windows.md](windows.md) |
| Linux | [linux.md](linux.md) |
| macOS | [mac.md](mac.md) |

跟着对应系统的指南做完，回到本页做登录认证。

---

## 第 2 步：验证安装

按系统指南操作完后，确认一切正常：

```bash
node --version    # 应显示 v18.x 或更高
claude --version  # 应显示已安装的 Claude Code 版本
```

---

## 第 3 步：登录

<img src="../../../tutorial/day0/assets/login.png" alt="Claude Code 登录界面" width="50%">

在终端运行 `claude`。首次启动时会让你选择登录方式。

### 方式 1：订阅账户（Claude Pro / Max）

- 选择 **Claude.ai account**
- 浏览器打开 — 登录并授权
- 回到终端，你已登录

### 方式 2a：API Key（团队邀请）

团队管理员从 Anthropic 控制台邀请你。

- 你会收到一封**邀请邮件** — 接受并创建 Anthropic 账户
- 在终端运行 `claude`
- 选择 **Anthropic API Key**
- 你的密钥在控制台**自动生成** — 无需手动配置
- Claude Code 立即可用

### 方式 2b：API Key（你已有密钥）

如果别人把密钥分享给你（通过 Slack、邮件等）或你自己创建过：

- 在终端运行 `claude`
- 选择 **Anthropic API Key**
- 粘贴密钥（以 `sk-ant-` 开头）
- 密钥**永久保存** — 之后不会再问

---
