---
title: Linux/macOS 安装 NVM 与 Node.js
slug: nvm-node-install
description: "在 Linux/macOS 上安装 nvm，并用它安装、切换和管理多个 Node.js 版本。"
date: 2026-08-26 10:30:00
updated: 2026-08-26 10:30:00
categories:
  - node
tags:
  - nvm
  - node
  - linux
  - macos
---

# Linux/macOS 安装 NVM 与 Node.js

NVM（Node Version Manager）是 Node.js 的版本管理工具，可以在同一台机器上安装、切换多个 Node.js 版本。本文以 nvm v0.40.7 为例，介绍从安装到日常使用的完整流程。

## 1. 安装前准备

### Linux

安装脚本会使用 `curl` 或 `wget` 下载，并使用 `git` 克隆 nvm 仓库，建议先安装：

```bash
# Debian/Ubuntu
sudo apt update && sudo apt install -y curl git

# CentOS/RHEL
sudo yum install -y curl git
```

如果下载不到官方预编译的 Node.js 二进制（例如特殊架构），会从源码编译，建议安装编译工具：

```bash
# Debian/Ubuntu
sudo apt install -y build-essential libssl-dev
```

### macOS

需要先安装 Xcode Command Line Tools，否则安装脚本无法正常工作：

```bash
xcode-select --install
```

macOS 10.15 之后默认 shell 是 zsh。如果 `~/.zshrc` 不存在，先创建再运行安装脚本：

```bash
touch ~/.zshrc
```

## 2. 安装 NVM

使用官方安装脚本（推荐）：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.7/install.sh | bash
```

或者使用 wget：

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.7/install.sh | bash
```

脚本会完成两件事：

1. 将 nvm 仓库克隆到 `~/.nvm`
2. 自动把加载 nvm 的配置写入 `~/.bashrc`、`~/.bash_profile`、`~/.zshrc` 或 `~/.profile`

安装完成后，重新打开终端，或手动加载配置：

```bash
# bash
source ~/.bashrc

# zsh
source ~/.zshrc
```

验证是否安装成功：

```bash
command -v nvm
```

输出 `nvm` 即表示安装成功。注意：nvm 是一个 shell 函数，不是可执行文件，`which nvm` 无法检测。

如果 `command -v nvm` 没有输出，检查 `~/.bashrc` 或 `~/.zshrc` 是否包含以下配置，没有则手动添加：

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

## 3. 安装 Node.js

安装最新的 LTS 版本（推荐，目前为 Node.js 24）：

```bash
nvm install --lts
```

安装最新版本（目前为 Node.js 26，2026 年 10 月进入 LTS）：

```bash
nvm install node
```

安装指定版本：

```bash
nvm install 24
nvm install 24.19.0
```

安装完成后确认版本：

```bash
node -v
npm -v
```

说明：

- 第一次安装的版本会自动成为默认版本
- 安装 Node.js 时会自动带上对应版本的 npm
- 使用 nvm 后，全局安装 npm 包不需要 `sudo`：

    ```bash
    npm install -g yarn
    ```

## 4. 常用命令

| 命令                     | 说明                         |
| ------------------------ | ---------------------------- |
| `nvm install --lts`    | 安装最新 LTS 版本            |
| `nvm install node`     | 安装最新版本                 |
| `nvm install 24`       | 安装指定主版本               |
| `nvm use 24`           | 切换当前 shell 使用的版本    |
| `nvm alias default 24` | 设置默认版本                 |
| `nvm ls`               | 查看已安装版本               |
| `nvm ls-remote`        | 查看远程可用版本             |
| `nvm current`          | 查看当前使用的版本           |
| `nvm uninstall 24`     | 卸载指定版本                 |
| `nvm which 24`         | 查看指定版本的可执行文件路径 |
| `nvm run 24 --version` | 用指定版本直接运行命令       |

## 5. 项目级版本管理（.nvmrc）

在项目根目录创建 `.nvmrc` 文件，写入需要的版本：

```text
24
```

切换到项目目录后执行：

```bash
nvm use
```

nvm 会自动读取 `.nvmrc` 并切换版本。

## 6. 常见问题

### nvm: command not found

- 重新打开终端，或执行 `source ~/.bashrc` / `source ~/.zshrc`
- macOS 默认 shell 是 zsh，确认 `~/.zshrc` 存在并包含 nvm 配置
- 如果安装脚本写入了错误的配置文件，可以指定 `PROFILE` 重新安装：

```bash
PROFILE=~/.zshrc curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.7/install.sh | bash
```

### 不建议使用 Homebrew 安装 nvm

官方不支持通过 Homebrew 安装 nvm。如果已经通过 Homebrew 安装，先卸载，再使用官方安装脚本：

```bash
brew uninstall nvm
```

### 配置国内 npm 镜像（可选）

```bash
npm config set registry https://registry.npmmirror.com
```

验证：

```bash
npm config get registry
```

## 参考链接

- nvm 官方仓库：https://github.com/nvm-sh/nvm
- Node.js 官网：https://nodejs.org
