---
title: jdk配置
slug: jdk-config
date: 2020-06-03 16:55:58
updated: 2020-06-04 12:41:35
tags:
  - jdk
  - linux
---

1. 下载源文件 Oracle JDK
2. 解压文件,复制到`/opt/`
3. 配置.修改`/etc/profile`或者`~/.bashrc`,末尾添加:
    ```bash
    export JAVA_HOME=/opt/jdk
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$JAVA_HOME/bin:$PATH
    ```
4. 重启或执行如下命令后生效：

```bash
source /etc/profile 或 source ~/.bashrc
```
