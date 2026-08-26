---
title: Linux/macOS 命令行快速切换代理
slug: proxy-switch
description: "在 Linux/macOS 终端中定义 proxy 与 noproxy 函数，一条命令快速开启或关闭代理环境变量。"
date: 2026-08-26 11:00:00
updated: 2026-08-26 11:00:00
categories:
  - linux
tags:
  - linux
  - macos
  - proxy
  - shell
---

# Linux/macOS 命令行快速切换代理

在终端中使用代理时，手动设置和清理环境变量比较繁琐。可以在 shell 配置文件中定义两个函数，一条命令开启代理、一条命令关闭代理。

这种方式只影响终端里的命令行程序（curl、wget、git、npm 等），不影响图形界面应用。

## 1. 配置方法

把以下内容添加到 `~/.bashrc`（bash）或 `~/.zshrc`（zsh）：

```bash
# 开启代理
proxy() {
  export https_proxy=http://user:pass@host:port
  export http_proxy=http://user:pass@host:port
  export all_proxy=socks5://user:pass@host:port
  echo "代理已开启"
}

# 关闭代理
noproxy() {
  unset http_proxy
  unset https_proxy
  unset all_proxy
  echo "代理已关闭"
}
```

注意：把 `user:pass@host:port` 替换为实际的代理地址。如果代理不需要认证，可以省略 `user:pass@`。

## 2. 使用方法

加载配置（新开的终端会自动加载，无需重复执行）：

```bash
# bash
source ~/.bashrc

# zsh
source ~/.zshrc
```

开启代理：

```bash
proxy
```

查看当前代理环境变量：

```bash
env | grep -i proxy
```

验证网络是否走代理：

```bash
curl -I https://www.google.com
```

关闭代理：

```bash
noproxy
```

## 3. 环境变量说明

| 变量 | 作用 |
| --- | --- |
| `http_proxy` | HTTP 请求使用的代理 |
| `https_proxy` | HTTPS 请求使用的代理 |
| `all_proxy` | 所有协议（包括 SOCKS）使用的代理 |

大部分命令行工具会自动读取这些环境变量。少数工具只识别大写形式（`HTTP_PROXY`、`HTTPS_PROXY`），如果遇到个别命令不走代理，可以在函数中同时导出大写变量。

## 4. 补充配置

### 本地地址不走代理

```bash
export no_proxy=localhost,127.0.0.1
```

### git 单独配置代理（可选）

开启：

```bash
git config --global http.proxy http://user:pass@host:port
git config --global https.proxy http://user:pass@host:port
```

关闭：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```
