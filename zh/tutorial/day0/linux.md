# Linux 安装

> 译自 [tutorial/day0/linux.md](../../../tutorial/day0/linux.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

[返回 Day 0](README.md)

## 前置条件

需要 **Node.js v18 或更高**以及 **npm**。

## 第 1 步：安装 Node.js

### 方案 A：通过 nodejs.org 下载页 + fnm（推荐）

**fnm**（Fast Node Manager）是 Node.js 官方推荐的版本管理器。它快、轻量，之后需要时可以方便地切换 Node 版本。

1. 打开浏览器进入 [nodejs.org/en/download](https://nodejs.org/en/download)。

2. 你会看到一排下拉框，写着 **"Get Node.js® vXX.XX.X (LTS) for __ using __ with __"**。按下面设置：

   | 下拉框 | 选择 |
   |----------|--------|
   | Version | **vXX.XX.X (LTS)**——保持默认 LTS 版本，不用改 |
   | OS | **Linux** |
   | Package Manager | **fnm**（在 "Recommended (Official)" 一栏） |
   | Package Format | **npm**——保持默认 |

3. 页面会给出确切命令。打开终端复制粘贴运行，大致长这样：

   ```bash
   # Step 1 — Install fnm
   curl -fsSL https://fnm.vercel.app/install | bash

   # Step 2 — Restart your terminal or reload your shell profile
   source ~/.bashrc   # or: source ~/.zshrc (if you use zsh)

   # Step 3 — Install Node.js
   fnm install 24  # The page will show the exact version number
   ```

   > 版本号可能与上面不同——始终以网站显示的为准。

4. **关闭并重新打开终端**（或运行上面的 `source` 命令），让 `fnm`、`node`、`npm` 生效。

> **为什么用 fnm？** 它在 Node.js 下载页的 "Recommended (Official)" 分类里。和 nvm 一样，它把 Node 装在你的用户目录下，npm 全局安装永远不需要 `sudo`——而 fnm 明显更快（Rust 写的），且在 Windows、macOS、Linux 上行为一致。

### 方案 B：用发行版的包管理器

更快，但可能装到旧版 Node.js。**装完记得检查版本**——低于 v18 就改用方案 A。

**Ubuntu / Debian：**

```bash
sudo apt update
sudo apt install -y nodejs npm

# Check the version
node --version   # Must be v18 or higher
```

**Fedora：**

```bash
sudo dnf install -y nodejs npm
```

**Arch Linux：**

```bash
sudo pacman -S nodejs npm
```

### 方案 C：NodeSource（通过 apt 装最新 LTS，不用 nvm）

适合想要最新 LTS 又不想用 nvm 的 Ubuntu/Debian 用户：

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

## 第 2 步：验证 Node.js

```bash
node --version
npm --version
```

两个都应打印版本号。`node --version` 必须是 v18.x 或更高。

## 第 3 步：安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

> **权限报错？**
> - 用了 **fnm** 或 **nvm** 的不该遇到。检查它是否生效（`which node` 应指向用户目录里的路径，而不是 `/usr/...`）。
> - 系统安装的，要么用 `sudo npm install -g @anthropic-ai/claude-code`，要么修 npm 全局目录权限：
>   ```bash
>   mkdir -p ~/.npm-global
>   npm config set prefix '~/.npm-global'
>   echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
>   source ~/.bashrc
>   ```

## 第 4 步：验证 Claude Code

```bash
claude --version
```

能看到 Claude Code 版本号就成功了。现在回到 [README.md](README.md) 完成登录设置。

---

## 备注

- **WSL（Windows 的 Linux 子系统）：** 本指南在 WSL 里同样适用，直接在 WSL 终端里按步骤做即可。
- **PATH 问题：** 装完后找不到 `claude` 命令时，确认 npm 的全局 bin 在 PATH 里。运行 `npm config get prefix`——该路径下的 `bin/` 子目录需要在 PATH 中。
