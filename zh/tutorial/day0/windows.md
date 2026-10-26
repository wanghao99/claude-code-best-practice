# Windows 安装

> 译自 [tutorial/day0/windows.md](../../../tutorial/day0/windows.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

[返回 Day 0](README.md)

---

**Node.js**
- 打开 [nodejs.org](https://nodejs.org)
- 点击 **"Download Node.js (LTS)"** 按钮——下载 `.msi` 安装包
- 运行 `.msi` 文件，在向导中一路点 **Next**
- 保持默认选项，点 **Install**，等待完成

**验证 Node.js**
- 打开一个**新**终端（PowerShell 或 Windows Terminal），运行：
  ```powershell
  node --version
  npm --version
  ```

**Claude Code**
- ```powershell
  npm install -g @anthropic-ai/claude-code
  ```
- 如果遇到权限错误，以**管理员身份**运行终端（右键 > 以管理员身份运行）

**验证**
- ```powershell
  claude --version
  ```

---

现在回到 [README.md](README.md) 完成登录设置。
