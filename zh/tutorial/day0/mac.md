# macOS 安装

> 译自 [tutorial/day0/mac.md](../../../tutorial/day0/mac.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

[返回 Day 0](README.md)

---

**终端**
- 打开"终端"（按 `Cmd + Space`，输入 "Terminal"，回车）

**Homebrew**
- 检查是否已装 Homebrew：
  ```bash
  brew --version
  ```
- 如果提示 "command not found"，先安装 Homebrew：
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```

**Claude Code**
- ```bash
  brew install --cask claude-code
  ```

**验证**
- ```bash
  claude --version
  ```

---

现在回到 [README.md](README.md) 做登录配置。
