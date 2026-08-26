---
title: Manjaro Linux 安装配置 Avahi（mDNS）
slug: LAN_avahi
description: "在 Manjaro Linux 上安装配置 Avahi 与 nss-mdns，实现 .local 域名解析（mDNS）。"
date: 2026-08-26 14:15:00
updated: 2026-08-26 14:15:00
categories:
  - linux
tags:
  - linux
  - manjaro
  - avahi
  - mdns
---

# Manjaro Linux 安装配置 Avahi（mDNS）

在 Manjaro Linux 上安装和配置 mDNS（多播 DNS）服务可以借助 Avahi 来实现，Avahi 是 Linux 系统中 mDNS 的一个常见实现，下面为你详细介绍安装和配置步骤。

## 1. 安装 Avahi

可以使用 `pacman` 包管理器来安装 Avahi，在终端中执行以下命令：

```bash
sudo pacman -S avahi
```

在执行过程中，`pacman` 会自动下载并安装 Avahi 及其依赖项。

## 2. 安装 nss-mdns

`nss-mdns` 用于让系统的名称解析服务支持 `.local` 域名的解析，同样使用 `pacman` 进行安装：

```bash
sudo pacman -S nss-mdns
```

## 3. 配置 nss-mdns

安装完成 `nss-mdns` 后，需要对其进行配置，以确保系统能够正确解析 `.local` 域名。编辑 `/etc/nsswitch.conf` 文件，找到 `hosts` 这一行，将其修改为如下内容：

```text
hosts: files mymachines myhostname mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] dns
```

上述配置中，`mdns_minimal` 表示优先使用 mDNS 进行 `.local` 域名的解析。可以使用以下命令进行编辑：

```bash
sudo nano /etc/nsswitch.conf
```

编辑完成后，按下 `Ctrl + X`，然后按 `Y` 保存更改，最后按 `Enter` 退出编辑器。

## 4. 启动并设置 Avahi 服务开机自启

使用以下命令启动 Avahi 服务：

```bash
sudo systemctl start avahi-daemon.service
```

可以使用以下命令检查服务是否成功启动：

```bash
sudo systemctl status avahi-daemon.service
```

若要让 Avahi 服务在系统开机时自动启动，可以执行以下命令：

```bash
sudo systemctl enable avahi-daemon.service
```

## 5. 测试 mDNS 服务

完成上述步骤后，就可以测试 mDNS 服务是否正常工作。假设局域网内有一台设备的主机名为 `other_device`，可以在终端中使用以下命令进行测试：

```bash
ping other_device.local
```

如果能够正常 ping 通，说明 mDNS 服务已经成功安装并配置。
