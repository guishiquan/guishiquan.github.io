---
title: 服务器安装GUI
slug: server-gui
description: "在 CentOS 服务器上安装 GNOME 图形界面，并设置为默认图形启动。"
date: 2018-02-20 10:12:21
updated: 2020-06-04 12:41:36
tags:
  - linux
  - centos
---

## Centos

1. `yum -y groupinstall "X Window System"`
2. `yum grouplist` 查看组件
3. `yum groupinstall GNOME Desktop Environment -y` 安装gnome桌面
4. `systemctl set-default graphical.target` 设置为桌面环境启动服务器
5. `reboot`重启

PS：安装GUI后出现黑色背景,无标题栏等可能是应为使用了 yum update .因此没事不要乱更新.
