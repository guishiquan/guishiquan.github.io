---
title: pip 离线下载组件与依赖
slug: pip-offline-download
date: 2026-08-26 12:00:00
updated: 2026-08-26 12:00:00
categories:
  - python
tags:
  - pip
  - python
  - offline
---

# pip 离线下载组件与依赖

在无法直接访问外网的目标机器上安装 Python 组件时，可以先用有网的机器执行 `pip download`，把组件和依赖下载到本地目录，再拷贝到目标机器离线安装。本文介绍单个组件、指定版本、跨平台以及 requirements.txt 批量下载的方法。

## 1. 下载单个组件及其依赖

```bash
pip download requests -d ./offline_packages
```

说明：

- `pip download` 只下载不安装
- 默认会同时解析并下载所有依赖
- `-d`（`--dest`）指定下载目录，目录不存在时自动创建

下载指定版本：

```bash
pip download requests==2.32.3 -d ./offline_packages
```

只下载组件本身，不下载依赖：

```bash
pip download requests --no-deps -d ./offline_packages
```

## 2. 批量下载 requirements.txt

requirements.txt 内容示例：

```text
flask==3.0.3
requests==2.32.3
numpy==2.1.3
```

批量下载：

```bash
pip download -r requirements.txt -d ./offline_packages
```

建议在 requirements.txt 中固定版本号，保证下载和离线安装的结果一致。

## 3. 指定平台下载（跨平台）

默认下载的是当前系统的 wheel。如果需要在 Linux 机器上为 Windows 或 macOS 下载组件，需要同时指定平台、Python 版本、解释器和 ABI 参数，并配合 `--only-binary=:all:`。

为 Windows 64 位 + Python 3.10 下载：

```bash
pip download -r requirements.txt -d ./offline_packages \
  --platform win_amd64 \
  --python-version 3.10 \
  --implementation cp \
  --abi cp310 \
  --only-binary=:all:
```

为 Linux x86_64 + Python 3.11 下载：

```bash
pip download -r requirements.txt -d ./offline_packages \
  --platform manylinux2014_x86_64 \
  --python-version 3.11 \
  --implementation cp \
  --abi cp311 \
  --only-binary=:all:
```

为 macOS（Apple Silicon）+ Python 3.10 下载：

```bash
pip download -r requirements.txt -d ./offline_packages \
  --platform macosx_11_0_arm64 \
  --python-version 3.10 \
  --implementation cp \
  --abi cp310 \
  --only-binary=:all:
```

参数说明：

| 参数 | 说明 | 常见值 |
| --- | --- | --- |
| `--platform` | 目标平台标签 | `win_amd64`、`manylinux2014_x86_64`、`macosx_11_0_arm64` |
| `--python-version` | 目标 Python 版本 | `3.10`、`3.11`、`3.12` |
| `--implementation` | Python 解释器 | `cp`（CPython）、`py`（解释器无关） |
| `--abi` | ABI 标签 | `cp310`、`abi3`、`none` |

注意事项：

- 使用 `--platform`、`--python-version`、`--implementation`、`--abi` 时，必须同时加 `--only-binary=:all:` 或 `--no-deps`，否则 pip 会报错
- 纯 Python 包一般提供 `py3-none-any` 通用 wheel，不区分平台
- 没有对应平台 wheel 的包只能下载源码包（tar.gz），离线安装时目标机器需要编译环境

## 4. 离线安装

把整个 `offline_packages` 目录拷贝到目标机器，然后执行：

```bash
pip install --no-index --find-links=./offline_packages -r requirements.txt
```

说明：

- `--no-index`：不访问 PyPI 等索引源
- `--find-links`：从本地目录查找组件

## 5. 常用选项速查

| 选项 | 说明 |
| --- | --- |
| `-d, --dest` | 下载目录 |
| `-r, --requirement` | 从 requirements.txt 批量下载 |
| `--no-deps` | 不下载依赖 |
| `--only-binary=:all:` | 只下载 wheel 二进制包 |
| `--no-binary=:all:` | 只下载源码包 |
| `-i, --index-url` | 指定下载源（如国内镜像） |
| `--no-index` | 不使用索引源 |
| `-f, --find-links` | 从本地目录或链接查找 |
| `--require-hashes` | 按 requirements.txt 中的哈希校验 |

使用国内镜像下载：

```bash
pip download -r requirements.txt -d ./offline_packages -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 6. 常见问题

### 跨平台参数报错

跨平台下载参数（`--platform` 等）必须配合 `--only-binary=:all:`（或 `--no-deps`）使用，否则 pip 会拒绝执行。补上 `--only-binary=:all:` 后重试。

### 下载的包在目标机器上装不上

检查目标机器的系统、Python 版本和位数是否与下载时指定的平台参数一致；如果只下载到源码包，还需要 gcc 等编译工具。

### 需要同时支持多个平台或 ABI

同一个参数可以重复使用多次：

```bash
pip download -r requirements.txt -d ./offline_packages \
  --platform win_amd64 --platform win32 \
  --python-version 3.10 \
  --implementation cp \
  --abi cp310 --abi abi3 --abi none \
  --only-binary=:all:
```

## 参考链接

- pip download 官方文档：https://pip.pypa.io/en/stable/cli/pip_download/
- PEP 425（wheel 兼容性标签）：https://peps.python.org/pep-0425/
