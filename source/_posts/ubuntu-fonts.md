---
title: Ubuntu字体
slug: ubuntu-fonts
description: "卸载 fonts-arphic-ukai 与 fonts-arphic-uming 两个字体包，修复 Ubuntu 更新后字体变难看的问题。"
date: 2018-06-03 15:42:06
updated: 2020-06-04 12:41:35
tags:
  - linux
  - ubuntu
  - 字体
---

Ubuntu因不完整的语言支持更新后字体会变得难看？

原因是多安装了两个字体，需要卸载两个字体。

```bash
sudo apt-get remove fonts-arphic-ukai fonts-arphic-uming
```
